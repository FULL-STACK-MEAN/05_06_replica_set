# Archivos de configuración de servidores

Desde CLI podemos iniciar un servidor con el comando (por ejemplo):

mongod --replSet clusterGetafe --dbpath server1 --port 27101

Pero como alternativa podemos usar archivos de configuración que tienen
las siguientes características

- extension .conf
- se escriben en yaml
- se ejecutan con la opción --config

mongod --config <archivo>.conf (en Unix se levanta sin terminal)

## apagar un servidor desde la shell de mongo

use admin

db.shutdownServer()