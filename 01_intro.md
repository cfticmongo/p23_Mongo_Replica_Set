# Introducción a Replica Set

Un replica set es un cluster o grupo de servidores que mantienen el mismo set de datos, para:

- ALTA DISPONIBILIDAD.
- Incremento en la capacidad de lectura.
- Copias adicionales de los datos para propósitos dedicados.
    - Reporting.
    - Recuperación de desastres.
    - backup.
    - Distribución geográfica para lecturas.

¿Es el replica set es un sistema de escalado horizontal?

En principio no es un sistema de escalado horizontal, porque las operaciones de escritura
solo se van realizar en uno de los miembros. Para escalado horizontal, MongoDB propone el sharding.

En operaciones de lectura si que podría considerarse un sistema de escalado horizontal.