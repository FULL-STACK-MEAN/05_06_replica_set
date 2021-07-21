# Rollback

Set de operaciones que se han persistido y registrado en el oplog del primario mientras este no 
era capaz de comunicarse con el resto de miembros por una partición de red hasta que cualquiera
de esos miembros adquiere la condición de primario.

Para no provocar inconsistencias en las colecciones, cuando el miembro en estado Rollback se recupera
las operaciones que no estén en el oplog del nuevo primario se quedan pendientes de actualizar
de manera manual por los DBA.

# Write concern

Write concern es un sistema de reconocimiento de escritura en los miembros de un replica set
que garantiza que una operación de escritura se registre en un determinado número de miembros. Es un valor
que se establece como opción en las operaciones de escritura.

Sintaxis (en el objeto de opciones de cualquier op de escrictura)

{
    writeConcern: {
        w: <Entero con el número de miembros del cluster que reconocerán la escritura ó 'majority'>,
        j: <boleana si para considerar la escritura se tiene en cuenta si está en el journal>,
        wtimeout: <entero en milisegundos>
    }
}

w: - numero de miembros en los que al menos debe escribirse la operación
   - 'majority' el menor de los valores:
        - Mayoría de los miembros con voto i/árbitros
        - Mayoría de todos los miembros con datos y voto.

Con el writeConcern.w seteado a 'majority' hacemos desaparecer la posibilidad de rollback.

El reconocimiento de escritura tiene un coste en términos de velocidad de las operaciones
de escritura.