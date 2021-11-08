# Despliegue de cluster replica set (en local)

## Cluster de 3 servidores

1.- Crear 3 directorios para alojar los datos en disco.

En la ubicación donde quereamos (preferible donde se ubique nuestra terminal)

mkdir server1
mkdir server2
mkdir server3

2.- Levantar los 3 servidores miembros usando el comando mongod (ejecutable).

Con la siguiente configuración:

mongod --dbpath server1 --port 27101 --replSet clusterGetafe
mongod --dbpath server2 --port 27102 --replSet clusterGetafe
mongod --dbpath server3 --port 27103 --replSet clusterGetafe

3.- Configuración e inicialización del cluster

Conectar con la shell de mongo a uno de los tres miembros (da igual)

mongo --port 27102 // Puede ser cualquier miembro

Usaremos la referencia global rs

rs.initiate({ // el método initiate recibe un objeto de configuración del cluster
    _id: "clusterGetafe",
    members: [
        {_id: 0, host: "localhost:27101"},
        {_id: 1, host: "localhost:27102"},
        {_id: 2, host: "localhost:27103"}
    ]
})

Cuando hayan pasado aprox 15-20seg (proceso elecciones) se habrá determinado
cual es el primario y cuales son los secundarios.

rs.status() // Devuelve información del cluster

## Estados de los miembros

PRIMARY Todas las operaciones del cluster se dirigen a este miembro
SECONDARY Reciben una replica de cada operación que se ejecuta en el primario (oplog)

Podemos desde la shell simular una conexión de cliente y realizar operaciones.

Desde una shell conectada al primario

```
use clinica
db.pacientes.insert({nombre: 'Juan', apellidos: 'Pérez'})
db.pacientes.find()
```

Si intentamos realizar cualquier operación en un secundario, incluso
desde la shell, comprobaremos que no se puede realizar.

```
// Cualquier operación devuelve error
```

Si necesitaramos de manera opcional que un miembro, cuando sea secundario,
tenga permisos de lectura, debemos autorizarlo expresamente.

```
rs.secondaryOk() // desde la shell conectada al secondary que queremos autorizar

db.pacientes.find() // ok

db.pacientes.insert({nombre: "Laura"}) // lo que no podremos nunca es operaciones de escritura

```

¿Cuando se producen cambios de estado? 

Los replica set tienen un mecanismo automatic failover que comprueba permanentemente la
comunicación entre los miembros (2 seg ping). Si se detecta una interrupción de la comunicación
mayor de 10 seg (parametrizable), el sistema entiende que el miembro no esta disponible
y desencadena el proceso de elecciones para elegir un nuevo primario.

## Configurar la prioridad de los miembros

Podemos definir que un determinado miembro tenga prioridad a la hora de
ser el primario. Con esta prioridad y siempre que sea posible un determinado
servidor será siempre el primario.

Se puede configurar al inicio (initiate) o reconfigurando el cluster (reconfig).

Por ejemplo, vamos a reconfigurar para que el miembro server3 (27103) sea el primario:

1.- Descargar el objeto de configuración.

Desde la shell del primario

```
let configuration = rs.conf() // Devuelve el objeto de configuracion
```

2.- Cambiamos la prioridad del miembro (priority)

```
configuration.members[2].priority = 2 // ya que 1 es el valor por defecto
```

3.- Reconfiguramos con nuestro nuevo objeto de configuración

```
rs.reconfig(configuration)
```

Al lanzar reconfig se desencadenan elecciones y como el miembro server3 tiene
prioridad mayor que los otros y está disponible pasará a ser el primario.

