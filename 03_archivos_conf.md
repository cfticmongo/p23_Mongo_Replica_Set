# Archivos de configuración de servidores (standalone, replica o sharding)

Cuando levantamos un servidor mongo podemos ejecutarlo desde la
linea de comandos con:

mongod <opciones...>

Hay una alternativa y es que mongod lea la configuración de un archivo en vez
de pasar las opciones

mongod --config <archivo>.conf

Los archivos deben tener

- Extensión .conf
- Se escriben en yaml

## Para apagar servidores desde la shell

Conectar una shell al servidor

mongo --port 27103

use admin

db.shutdownServer() // Cierra el propio servidor

