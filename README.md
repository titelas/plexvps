# plexvps
La idea de esta guía es montar un servidor de Plex con Google Drive en un VPS con linux (ubuntu en mi caso). 
Lo acompañaré de algunos extras como: cliente de torrent (transmission), sickrage para descargar series de forma automatica, plexpy para monitorizar nuestro servidor plex y algunos scripts y consideraciones para mejorar la experiencia a la hora del visionado.

## rclone
Utilizaremos rclone para montar Google Drive como  unidad de disco en nuestro sistema de tal forma que Plex pueda leer directamente de ahí el contenido. 

Descargamos la versión más actual de su web y lo descomprimimos.
```
curl -O https://downloads.rclone.org/rclone-current-linux-amd64.zip
unzip rclone-current-linux-amd64.zip
cd rclone-*-linux-amd64
```
Copiamos el binario.
```
sudo cp rclone /usr/bin/
sudo chown root:root /usr/bin/rclone
sudo chmod 755 /usr/bin/rclone
```
Instalamos manpage.
```
sudo mkdir -p /usr/local/share/man/man1
sudo cp rclone.1 /usr/local/share/man/man1/
sudo mandb 
```
Entramos en la configuración.
```
rclone config
```
N: creamos una nueva unidad
7: del tipo Google Drive
Dejamos client id y client secret en blanco
En el siguiente paso le damos a N y nos dará una url que debemos pegar en nuestro navegador (en tu ordenador local), loguearnos con nuestra cuenta de Google Drive que vayamos a utilizar y copiar el token que nos da y pegarlo.

Creamos una carpeta y montamos la unidad en ella.
```
mkdir /home/plexcloud
rclone mount --allow-other --allow-non-empty -v plexcloud: /home/plexcloud/ &
```

Si todo ha ido bien, listado el directorio deberéis ver vuestro contenido del drive.
```
ls /home/plexcloud
```

## Servidor plex
Vamos a instalar el plex media server y configurarlo.

En la sección de Downloads de su web, copiamos el enlace de la versión de linux, en mi caso Ubuntu 64 bits.
Descargamos e instalamos.
```
wget enlacecopiado
dkpg -i archivodescargado
```

Para acceder por primera vez, como entorno gráfico en linux y por tanto no hay navegador, debemos hacer un tunneling por ssh para enlazar nuestro localhost con el VPS.
```
ssh root@ipvps -L 8888:localhost:32400
```

Ahora accedemos en nuestro navegador a http://localhost:8888 y plex debería darnos la bienvenida para proceder a su configuración.
Añadimos las bibliotecas que queramos apuntando al contenido del drive, que detecta como una carpeta más.

En principio Plex debería empezar a escanear todo el contenido de los directorios que le hayamos indicado y una vez acabe ya está disponible par utilizarlo en cualquier dispositivo.

A partir de ahora, podéis acceder a vuestro servidor plex mediante http://ipvps:32400
