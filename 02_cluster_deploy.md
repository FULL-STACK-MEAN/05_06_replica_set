# Despliegue de cluster MongoDB en local

## Cluster de 3 servidores

1.- Crear 3 directorios para los datos

server1, server2, server3

2.- Utilizar los comandos para levantar los 3 servidores del cluster

mongod --replSet clusterGetafe --dbpath server1 --port 27101
mongod --replSet clusterGetafe --dbpath server2 --port 27102
mongod --replSet clusterGetafe --dbpath server3 --port 27103

3.- Configuración e inicialización del replica set

Conectamos con la shell de mongo a uno de los miembros del cluster

mongo --port 27101 (puede ser a cualquie miembro)

Utilizamos el método initiate sobre la instación rs (replica set) para la config

rs.initiate({
    _id: "clusterGetafe",
    members: [
        {_id: 0, host: "localhost:27101"},
        {_id: 1, host: "localhost:27102"},
        {_id: 2, host: "localhost:27103"}
    ]
})


Aprox en 15 o 20 segundos el cluster estará disponible

rs.status()

Estados (sin configuración adicional):

PRIMARY Todas las operaciones del cluster se dirigen al miembro en estado primario
SECUNDARY Se recibe una réplica de cada operación que se ejecuta en el primario (oplog)

¿Cuando se producen cambios de estado?

El mecanismo automatic failover comprueba permanentemente la conexión entre los miembros (ping cada 10s) y
si detecta que uno de los miembros no está disponible desencadena un proceso de elecciones.

Elecciones => reasignar el primario y reconfigurar el funcionamiento del cluster