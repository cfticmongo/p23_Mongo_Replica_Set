# Rollback

Set de operaciones que se han persistido y registrado en el oplog del primario mientras
que este no era capaz de comunicarse con el resto de miembros por una partición de red. Las
operaciones se registrarían hasta que el primario pierda su condición, momento en el cual
los clientes dejarán de enviarle operaciones de escritura.

Para no provocar inconsistencias en los documentos de las colecciones, cuando el miembro se
recupera y puede conectarse al resto de miembros incluyendo el nuevo primario, detectará que hay
operaciones que no fueron enviadas a los otros miembros y, entra en un estado Rollback, envía esas
operaciones a un documento bson en el disco, elimina esas operaciones del oplog, se sincroniza
al primario y se recupera a secundario y posteriormente a primario (en el caso de que tuviera
mayor prioridad).

Con las operaciones en el set de datos en BSON rollback el DBA puede importarlas a una coleccion temporal,
visualizar los documentos y decidir si se integran o no de manera manual.

¿Como se puede evitar el rollback?

- Poniendo el reconocimiento de escritura con el valor 'majority'.



