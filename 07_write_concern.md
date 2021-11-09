# Reconocimiento de escritura en los replica set

Write concern es un sistema de reconocimiento de escritura en los miembros de un replica set
que garantiza que una operación de escritua se registre en un determinado nº de miembros.

Esta funcionalidad aporta la máxima consistencia temporal de los datos del cluster:
    - Con majority evita rollback
    - Garantiza los datos frescos en lecturas a secundarios

Desventaja
    - Las operaciones de escritura serán más lentas

Es una configuración que se establece:

    - A nivel global (para todo el cluster) Desde mongo 5.0 por defecto se establece majority a nivel global
    - A nivel de colección
    - A nivel de operación

Sintaxis

Objeto que se le pasa a la propiedad writeConcern con las siguientes opciones

writeConcern: {
    w: <Entero con el nº de miembros | 'majority' >,
    j: <boolean>, // se considera la escritura cuando se persiste en el journal
    wtimeout: <entero> // milisegundos de timeout
}

w: nº de miembros (con valor 1 realmente no tendriamos reconocimiento de escritura)
   'majority'   - Mayoría de los miembros con voto i/árbitros
                ó
                - Mayoría de todos los miembros con datos y voto
