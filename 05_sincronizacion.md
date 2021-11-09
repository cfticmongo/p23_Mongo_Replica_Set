# Sincronización y estados del replica set

## Formas de sincronización

1.- Sincronización inicial (cuando un miembro es incorporado al cluster).

    - El miembro se incorpora vacío.
        - Clonación desde el primario incluyendo todas las bases de datos menos la "local".
        - Clonación de la base de datos "local", que tendrá el oplog actualizado.
        - Sincroniza las últimas operaciones registradas en el oplog.

    Tiene la ventaja
        - Es muy sencillo de implementar.

    Tiene las siguientes desventajas.
        - Proceso lento.
        - Tiene un consumo de procesos y memoria importantes en el primario. Puede provocar
        perder el set de datos frecuentes en memoria.
        - Si el volumen de escrituras en el cluster es muy elevado el secundario podría no
        llegar a alcanzar la sincronización.

    - El miembro se incorpore con un backup del primario (o de un secundario sincronizado)

    Tiene las ventajas
        - Proceso bastante más rápido.
        - Solamente se producirán la clonación de las operaciones desde la fecha del backup.
        - Casi inmediatamente se producirá la clonación de la base de datos "local".
        - Sincroniza las últimas operaciones registradas en el oplog.

    Tiene desventajas.
        - Mayor complejidad porque exige la recuperación del backup.
        - No permite usar la herramienta nativa de backup de MongoDB, ya que esta no 
        contiene la colección oplog.
    
2.- Replicación (proceso convencional por el cual los miembros del cluster sincronizan
las operaciones del primario).

    Los secundarios copian las operaciones del oplog del primario, las implementan en
    sus colecciones y las copian en su propio oplog.

## Estados de los miembros

STARTUP Estado del miembro al ser añadido al cluster

STARTUP2 Una vez añadido si se han lanzado elecciones

RECOVERING Cuando el miembro se está sincronizando

SECONDARY

PRIMARY

ARBITER

DOWN

UNKNOW

REMOVED Se está eliminando

ROLLBACK. Cuando un miembro ha tenido un rollback

