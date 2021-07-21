# Colección oplog en replica set

El oplog es una colección (base de datos local) en la que se registran las operaciones en los
cluster replica set, siendo la fuente de sincronización de los secundarios.

- Es una colección limitada en tamaño (capped) y que no se puede modificar.
- Todas las operaciones registradas en el oplog son convertidas en
  idempotentes

## Acceso a el oplog (del primario)

use local

db.oplog.rs.find().sort({$natural: -1}) // Ordena en el sentido de inserción de los documentos descendente

use clusterTest

db.foo.insert({puntuacion: 0})

{ 
  "op" : "i", 
  "ns" : "clusterTest.foo", 
  "ui" : UUID("731bcef5-c6e2-49c9-be1c-5c645f27b13a"), 
  "o" : { "_id" : ObjectId("60f84ddbf25df58fad4f0583"), "puntuacion" : 0 }, 
  "ts" : Timestamp(1626885596, 1), 
  "t" : NumberLong(6), 
  "wall" : ISODate("2021-07-21T16:39:56.274Z"), 
  "v" : NumberLong(2) 
}

db.foo.update({},{$inc:{puntuacion: 1}}) // Donde se ejecuta la operación de idempotencia

{ 
    "op" : "u", 
    "ns" : "clusterTest.foo", 
    "ui" : UUID("731bcef5-c6e2-49c9-be1c-5c645f27b13a"), 
    "o" : { "$v" : 1, "$set" : { "puntuacion" : 1 } }, // idempotente 
    "o2" : { "_id" : ObjectId("60f84ddbf25df58fad4f0583") }, 
    "ts" : Timestamp(1626885635, 1), 
    "t" : NumberLong(6), 
    "wall" : ISODate("2021-07-21T16:40:35.188Z"), 
    "v" : NumberLong(2) 
}

- Una operación de escritura en MongoDb se puede convertir en muchas operaciones en el oplog

db.foo.insert([
    {a: 1},
    {a: 1},
    {a: 3}
])

{ "op" : "i", "ns" : "clusterTest.foo", "ui" : UUID("731bcef5-c6e2-49c9-be1c-5c645f27b13a"), "o" : { "_id" : ObjectId("60f84fd5f25df58fad4f0586"), "a" : 3 }, "ts" : Timestamp(1626886101, 3), "t" : NumberLong(6), "wall" : ISODate("2021-07-21T16:48:21.724Z"), "v" : NumberLong(2) }
{ "op" : "i", "ns" : "clusterTest.foo", "ui" : UUID("731bcef5-c6e2-49c9-be1c-5c645f27b13a"), "o" : { "_id" : ObjectId("60f84fd5f25df58fad4f0585"), "a" : 1 }, "ts" : Timestamp(1626886101, 2), "t" : NumberLong(6), "wall" : ISODate("2021-07-21T16:48:21.724Z"), "v" : NumberLong(2) }
{ "op" : "i", "ns" : "clusterTest.foo", "ui" : UUID("731bcef5-c6e2-49c9-be1c-5c645f27b13a"), "o" : { "_id" : ObjectId("60f84fd5f25df58fad4f0584"), "a" : 1 }, "ts" : Timestamp(1626886101, 1), "t" : NumberLong(6), "wall" : ISODate("2021-07-21T16:48:21.724Z"), "v" : NumberLong(2) }

db.foo.update({}, {$set: {fecha: new Date()}}, {multi: true})

{ "op" : "u", "ns" : "clusterTest.foo", "ui" : UUID("731bcef5-c6e2-49c9-be1c-5c645f27b13a"), "o" : { "$v" : 1, "$set" : { "fecha" : ISODate("2021-07-21T16:50:04.775Z") } }, "o2" : { "_id" : ObjectId("60f84fd5f25df58fad4f0586") }, "ts" : Timestamp(1626886204, 4), "t" : NumberLong(6), "wall" : ISODate("2021-07-21T16:50:04.776Z"), "v" : NumberLong(2) }
{ "op" : "u", "ns" : "clusterTest.foo", "ui" : UUID("731bcef5-c6e2-49c9-be1c-5c645f27b13a"), "o" : { "$v" : 1, "$set" : { "fecha" : ISODate("2021-07-21T16:50:04.775Z") } }, "o2" : { "_id" : ObjectId("60f84fd5f25df58fad4f0585") }, "ts" : Timestamp(1626886204, 3), "t" : NumberLong(6), "wall" : ISODate("2021-07-21T16:50:04.776Z"), "v" : NumberLong(2) }
{ "op" : "u", "ns" : "clusterTest.foo", "ui" : UUID("731bcef5-c6e2-49c9-be1c-5c645f27b13a"), "o" : { "$v" : 1, "$set" : { "fecha" : ISODate("2021-07-21T16:50:04.775Z") } }, "o2" : { "_id" : ObjectId("60f84fd5f25df58fad4f0584") }, "ts" : Timestamp(1626886204, 2), "t" : NumberLong(6), "wall" : ISODate("2021-07-21T16:50:04.776Z"), "v" : NumberLong(2) }
{ "op" : "u", "ns" : "clusterTest.foo", "ui" : UUID("731bcef5-c6e2-49c9-be1c-5c645f27b13a"), "o" : { "$v" : 1, "$set" : { "fecha" : ISODate("2021-07-21T16:50:04.775Z") } }, "o2" : { "_id" : ObjectId("60f84ddbf25df58fad4f0583") }, "ts" : Timestamp(1626886204, 1), "t" : NumberLong(6), "wall" : ISODate("2021-07-21T16:50:04.776Z"), "v" : NumberLong(2) }

Pregunta

En la colección foo con los siguientes documentos
{a: 1},{a: 1},{a: 3},{a: 4}

¿Cuales de las siguientes operaciones registrán 4 documentos en la colección oplog? Varias respuestas
disponibles

a) db.foo.insert({puntuacion: 0})
b) db.foo.update({},{$inc:{puntuacion: 1}})
c) db.foo.update({}, {$set: {fecha: new Date()}})
d) db.foo.update({}, {$set: {fecha: new Date()}}, {multi: true}) ok
e) db.foo.insert([{a: 5},{a: 6},{a: 7},{a: 8}]) ok
f) Ninguna de las anteriores

## Tamaño del oplog

El tamaño es establecido por MongoDB al crear el cluster y los valores serán:

- Si el motor es WiredTiger:
    5% espacio disponible en disco con min 50MB y max 50GB

Con este tamaño se obtiene una ventana de operaciones de ente 1 y 2 días

Puede haber 3 escenarios en los que sea aconsejable aumentar el tamaño predeterminado:

- Actualizaciones a múltiples documentos al mismo tiempo.
- Al mismo tiempo muchas eliminaciones e inserciones.
- Múltiples actualizaciones que no modifican tamaño de las colecciones pero
  si generan muchos docs en el oplog.




