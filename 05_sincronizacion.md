# Sincronización y estados del replica set

## Formas de sincronización

1.- Sincronización inicial (cuando un miembro se incorpora al cluster)

- El miembro esté vacío
    - Clonación desde el primario incluyendo todas las bases de datos menos base de datos "local".
    - Clonación de la base de datos "local" que ya contiene el oplog actualizado.
    - Sincronización de las últimas operaciones registradas en el oplog.

    Tiene las siguientes desventajas
        - Proceso lento.
        - Tiene un consumo de procesos y memoria importantes en el primario. Puede hacer
          perder el set de datos frecuentes en memoria.
        - Si el volumen de operaciones de escritura es muy grande en el cluster el
          secundario podría no alcanzar la sincronización.

- El miembro se incorpore con un backup del primario.
    - Clonación de las operaciones que falten desde la fecha del backup.
    - Clonación de la base de datos "local" que ya contiene el oplog actualizado.
    - Sincronización de las últimas operaciones registradas en el oplog.

    Una desventaja.
        - El proceso de backup no se puede hacer con la herramienta nativa
        de MongoDB, mongodump, porque está no hace backup del oplog.

2.- Replicación (proceso convencional por el cual los miembros del cluster sincronizan las operaciones)

    Los secundarios copian las operaciones del oplog del primario o de otro secundario que tenga
    operaciones mas recientes y las persite en las correspondientes colecciones y en su oplog.

## Estados de los miembros

STARTUP Al añadir el miembro al cluster

STARTUP2 Una vez añadido el miembro al cluster si se han lanzado nuevas elecciones

RECOVERING Cuando el miembro se está sincronizando

ARBITER

DOWN

UNKNOW

REMOVED

PRIMARY

SECONDARY

ROLLBACK. Estado cuando un miembro ha tenido un rollback.



