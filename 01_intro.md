# Introducción a Replica Set

- Un replica set o cluster es un grupo de servidores que mantienen el mismo set de datos, para:

    - ALTA DISPONIBLIDAD
    - Incremento en la capacidad de lectura
    - Copias adicionales de los datos para propósitos dedicados
        - reporting
        - recuperación de desastres
        - backup
        - ...

- ¿Es el replica set un sistema de escalado horizontal?

En principio no, porque las operaciones de escritura solo se producen en uno de los miembros, con lo
cual el escalado horizontal solo podría llevarse a cabo con la arquitectura sharding.

Pero, en operaciones de lectura si que podemos aprovechar la distribución del cluster para descargar
al primario de lecturas repartiendo de esta forma la capacidad de procesamiento, lo que se podría considerar
como un escalado horizontal.

