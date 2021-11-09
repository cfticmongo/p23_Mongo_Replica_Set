# Colección oplog en los miembros de replica set

El oplog es una colección (base de datos "local") de tipo capped collection en la
que se registran las operaciones del cluster replica set, siendo la fuente de sincronización
de los secundarios.

- Es limitada en tamaño (capped) y no se puede modificar.
- Todas las operaciones registradas en el oplog son convertidas
  en idempotentes (el resultado de la operación siempre será el mismo
  con independencia del número de veces que se ejecute).

## Acceder al oplog (primario)

Se encuentra en la base de datos local y la colección se llam oplog.rs

```
use local

db.oplog.rs.find().sort({$natural: -1}) // Ordena en sentido descendente por fecha de inserción

```

Para comprobar creamos una base de datos y añadimos un documento

```
use clusterTest

db.foo.insert({puntuacion: 0})

use local

db.oplog.rs.find({op: "i"}).sort({$natural: -1}).limit(1s)

```

{
        "op" : "i",
        "ns" : "clusterTest.foo",
        "ui" : UUID("b2cdf51e-bcfc-4e37-a1c5-a14034ba105e"),
        "o" : {
                "_id" : ObjectId("618aaf79a2474160a9a17f97"), // Operación que servirá
                "puntuacion" : 0                            // para sincronizar a los secundarios
        },
        "ts" : Timestamp(1636478841, 2),
        "t" : NumberLong(12),
        "v" : NumberLong(2),
        "wall" : ISODate("2021-11-09T17:27:21.313Z")
}

```
use clusterTest

db.foo.updateOne({"_id" : ObjectId("618aaf79a2474160a9a17f97")}, {$inc: {puntuacion: 1}})

use local

db.oplog.rs.find({op: "u"}).sort({$natural: -1}).limit(1s)

```

{
        "op" : "u",
        "ns" : "clusterTest.foo",
        "ui" : UUID("b2cdf51e-bcfc-4e37-a1c5-a14034ba105e"),
        "o" : {
                "$v" : 2,
                "diff" : {
                        "u" : {
                                "puntuacion" : 1 // Convierte la op en idempotente porque si el operador
                        }                         // $inc se ejecuta más de una vez cambiaría los valores
                }
        },
        "o2" : {
                "_id" : ObjectId("618aaf79a2474160a9a17f97")
        },
        "ts" : Timestamp(1636479355, 1),
        "t" : NumberLong(12),
        "v" : NumberLong(2),
        "wall" : ISODate("2021-11-09T17:35:55.072Z")
}

Ocurre lo mismo si la operación afecta a varios documentos

```
use clusterTest

db.foo.insert([
    {a: 1},
    {a: 9},
    {a: -14}
])

use local

db.oplog.rs.find({op: "i"}).sort({$natural: -1}).limit(1s)

```

Comprobamos como de 1 operación en la colección se guardan 3 operaciones idempotentes en el oplog del replicaSet

{
        "op" : "i",
        "ns" : "clusterTest.foo",
        "ui" : UUID("b2cdf51e-bcfc-4e37-a1c5-a14034ba105e"),
        "o" : {
                "_id" : ObjectId("618ab2b7a2474160a9a17f9a"),
                "a" : -14
        },
        "ts" : Timestamp(1636479671, 3),
        "t" : NumberLong(12),
        "v" : NumberLong(2),
        "wall" : ISODate("2021-11-09T17:41:11.565Z")
}
{
        "op" : "i",
        "ns" : "clusterTest.foo",
        "ui" : UUID("b2cdf51e-bcfc-4e37-a1c5-a14034ba105e"),
        "o" : {
                "_id" : ObjectId("618ab2b7a2474160a9a17f99"),
                "a" : 9
        },
        "ts" : Timestamp(1636479671, 2),
        "t" : NumberLong(12),
        "v" : NumberLong(2),
        "wall" : ISODate("2021-11-09T17:41:11.565Z")
}
{
        "op" : "i",
        "ns" : "clusterTest.foo",
        "ui" : UUID("b2cdf51e-bcfc-4e37-a1c5-a14034ba105e"),
        "o" : {
                "_id" : ObjectId("618ab2b7a2474160a9a17f98"),
                "a" : 1
        },
        "ts" : Timestamp(1636479671, 1),
        "t" : NumberLong(12),
        "v" : NumberLong(2),
        "wall" : ISODate("2021-11-09T17:41:11.565Z")
}

Ejemplo de pregunta de la certificacion

En la siguiente colección foo con los siguiente documentos

{a: 1}, {a: 1}, {a: 3}, {a: 4}

¿Cuales de las siguientes operaciones registrarán 4 documentos en la
coleccion oplog? (Varias respuestas)

a) db.foo.insert({puntuacion: 0}) No ok
b) db.foo.update({}, {$inc: {puntuacion: 1}}) No ok
c) db.foo.update({}, {$set: {fecha: new Date()}}, {multi: true}) Ok
d) db.foo.insert([{a: 5}, {b: 6}, {c: 7}, {d: 8}]) Ok
e) Ninguna de las anteriores No ok

## Tamaño del oplog

El tamaño es establecido automáticamente por Mongo al crear el cluster con los siguentes valores:

- Si tiene el motor WiredTiger
    5% espacio disponible en disco con min 50MB y max 50GB

Para operaciones 'normales', este tamaño del oplog proporciona una ventana
de entre 1 y 2 días.

Puede haber 3 escenarios en los que sea aconsejable aumentar el tamaño predeterminado
del oplog:

    - Frecuentes operaciones de actualización a múltiples documentos al mismo tiempo.
    - Frecuentes eliminaciones e inserciones de documentos al mismo tiempo.
    - Frecuentes operaciones de actualización que no modifiquen el tamaño de las
      colecciones pero generen muchos docs en el oplog.