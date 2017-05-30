# plexvps
Plex + Google Drive + Torrent + Sickrage en un VPS



---------------------
instalar rclone: montar google drive como unidad para que plex lea de ella
---------------------
-Linux installation from precompiled binary
-Fetch and unpack

curl -O https://downloads.rclone.org/rclone-current-linux-amd64.zip
unzip rclone-current-linux-amd64.zip
cd rclone-*-linux-amd64

-Copy binary file

sudo cp rclone /usr/bin/
sudo chown root:root /usr/bin/rclone
sudo chmod 755 /usr/bin/rclone

-Install manpage

sudo mkdir -p /usr/local/share/man/man1
sudo cp rclone.1 /usr/local/share/man/man1/
sudo mandb 

-Run rclone config to setup. See rclone config docs for more details.

rclone config

- Opcion "n", dar nombre e ir a la url para coger id.

* Si queremos hacer copias de un shared folder a otro, hay que crear otra unidad como nueva con otro nombre para hacer copias de uno a otro y aprovechas el bandwith.

- Montar la unidad

mkdir /home/plexcloud
rclone mount --allow-other --allow-non-empty -v plexcloud: /home/plexcloud/ &

------------------------------------
instalar plexmediaserver
------------------------------------
wget downloadwebdeplex
dkpg -i esearchivo

- tunneling: ssh root@ipvps -L 8888:localhost:32400
- acceder en tu pc local a http://localhost:8888
- configurar y luego ya estara disponible en http://ipvps:32400

------------------------------------
instalar plexpy : monitorizar actividad plex y notificaciones
------------------------------------

https://www.htpcbeginner.com/install-plexpy-on-ubuntu/


-----------------------------------------------------------
instalar tranmission: descargas de torrents
-----------------------------------------------------------


apt-get install tranmission-daemon

- Paramos el servicio:

service plexmediaserver stop

- Configurar las preferencias:

nano /var/lib/transmission-daemon/.config/transmission-daemon/settings.json

- Modificar user y pass ( si quieres, por defecto es transmission / transmission ).  rpc-whitelist-enabled a false para que nos deje entrar via web, tambien podemos dejarlo en true y poner nuestra ip en rpc-whitelist. 
    "rpc-password": "passwordquetuquieras", 
    "rpc-username": "transmission", 
    "rpc-whitelist-enabled": false, 

-  Habilitar carpeta de incomplete

"incomplete-dir-enabled": true, 
"incomplete-dir": "/var/lib/transmission-daemon/incomplete",

- Guardamos con Ctrl+X -> Y.
- Creamos el directorio con y le damos el propietario a user de transmission:

mkdir /var/lib/transmission-daemon/incomplete
chown debian-tranmission:debian-tranmission /var/lib/transmission-daemon/incomplete

- Arrancamos de nuevo el servicio

service plexmediaserver start

- Acceder con http://ipvps:9091

-----------------
Copiar autómaticamente descargas completas al drive
------------------

Descargar script y situarlo en /home/rclonemv.sh (por ejemplo)

- Poner en el cron para que se haga cada X minutos:

crontab -e

# Edit this file to introduce tasks to be run by cron.
#
# Each task to run has to be defined through a single line
# indicating with different fields when the task will be run
# and what command to run for the task
#
# To define the time you can provide concrete values for
# minute (m), hour (h), day of month (dom), month (mon),
# and day of week (dow) or use '*' in these fields (for 'any').#
# Notice that tasks will be started based on the cron's system
# daemon's notion of time and timezones.
#
# Output of the crontab jobs (including errors) is sent through
# email to the user the crontab file belongs to (unless redirected).
#
# For example, you can run a backup of all your user accounts
# at 5 a.m every week with:
# 0 5 * * 1 tar -zcf /var/backups/home.tgz /home/
#
# For more information see the manual pages of crontab(5) and cron(8)
#
# m h  dom mon dow   command
*/15 * * * * /home/rclonemv.sh


---------
Copiar de nube a nube o carpeta compartida
---------

- Si nos comparten una carpeta, damos boton derecho en ella y pinchamos en "Añadir a nuestro drive"
- Una vez hecho esto, vamos a duplicar el servidor en rclone para hacer una copia como si estuviesemos copiando de nube a nube

rclone config

- Opcion "n", dar nombre e ir a la url para coger id. Le damos otro nombre para diferenciarlas.

- Para hacer la copia

rclone copy -v --stats 30s drive1:ruta drive2:ruta

_______

XML Plex no transcoding AC3 on iOS

https://forums.plex.tv/discussion/260803/unnecessary-transcoding-of-h264

------
rclone google cloud platform
------
https://docs.google.com/document/d/17sOynlIKO5cgdzir4xmxzKHLGglKGGtT4zNsBklvSnQ/edit

