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

## Estados (sin configuración adicional):

PRIMARY Todas las operaciones del cluster se dirigen al miembro en estado primario
SECUNDARY Se recibe una réplica de cada operación que se ejecuta en el primario (oplog)

¿Cuando se producen cambios de estado?

El mecanismo automatic failover comprueba permanentemente la conexión entre los miembros (ping cada 10s) y
si detecta que uno de los miembros no está disponible desencadena un proceso de elecciones.

Elecciones => reasignar el primario y reconfigurar el funcionamiento del cluster.

## Configurar la prioridad de los miembros.

rs.reconfig(<configuración>)

1.- Descargar el objeto de configuración

let configuration = rs.conf() // Este devuelve el objeto de configuración del cluster actual

2.- Cambiar la prioridad (en el objeto JavaScript)

configuration.members[2].priority = 2 // Será superior a 1 que es el valor por defecto

3.- Reconfiguramos el cluster con ese objeto

rs.reconfig(configuration)

Cuando pasen 15-20 segundos el miembro con mayor prioridad pasará a ser el PRIMARIO (siempre que esté disponible)

## Añadir un nuevo miembro al cluster

1.- Crear un directorio para el nuevo miembro

server4

2.- Levantamos con el comando

mongod --replSet clusterGetafe --dbpath server4 --port 27104

3.- Añadir a la configuración desde el primario

rs.add({
    host: "localhost:27104",
    priority: 0, // para evitar que cuando se añada se convierta en primario
    votes: 0
})

4.- Volvemos a obtener la configuración del cluster

let configuration = rs.conf()

5.- Modificamos la prioridad y votos del nuevo miembro y renconfiguramos

configuration.members[3].priority = 1;
configuration.members[3].votes = 1;

rs.reconfig(configuration)

## Tolerancia a fallos

- Para determinar la tolerancia a fallos debemos tener en cuenta que no pueden 
existir dos primarios al mismo tiempo.

- Para que se produzcan elecciones y por tanto se elija un nuevo primario, debe haber en los
miembros disponibles mayoria contabilizando para esa mayoría el total de los miembros incluyendo los caidos.

- Toda vez que es necesario tener mayoria para que se produzcan elecciones la tolerancia a fallos aumentará
siempre en los clusters impares.

Tabla de tolerancia a fallos

nº Miembros totales         Mayoría (elecciones)        Tolerancia a fallos
    2                           2                           0
    3                           2                           1
    4                           3                           1
    5                           3                           2
    6                           4                           2
    7                           4                           3
    ...

## Añadir un miembro árbritro

1.- Crear un directorio

server5

2.- Levantamos con el comando

mongod --replSet clusterGetafe --dbpath server5 --port 27105

3.- Añadimos desde el primario como arbitro

rs.addArb("localhost:27105")

- El árbitro no será nunca primario ni secundario

## Operaciones de lectura en secundarios

- Solamente se podrán, por defecto, realizar operaciones sobre el primario.

- MongoDB permite que a los secundarios se les pueda configurar la posibilidad de recibir operaciones
de lectura (nunca de escritura) para que, aunque hallamos perdido la disponibilidad por la tolerancia
a fallos, al menos se mantengan las lecturas.

- También podemos realizar operaciones de lectura en secundarios para los propósitos dedicados.

Para establecer operaciones de lectura cuando el servidor esté en estado secundario:

db.setSecondaryOk() // Se establece a nivel de base de datos

