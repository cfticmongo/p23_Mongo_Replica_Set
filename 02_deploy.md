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