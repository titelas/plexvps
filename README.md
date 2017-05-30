# plexvps
La idea de esta guía es montar un servidor de Plex con Google Drive en un VPS con linux (ubuntu en mi caso). 
Lo acompañaré de algunos extras como: cliente de torrent (transmission), sickrage para descargar series de forma automatica, plexpy para monitorizar nuestro servidor plex y algunos scripts y consideraciones para mejorar la experiencia a la hora del visionado.



## VPS y SSH
En mi caso voy a utilizar un VPS pequeño y barato de Scaleway (https://www.scaleway.com 2.99€) ya que voy a hacer direct play de todo el contenido, pero tened en cuenta que si alguno de vuestros users va a necesitar transcoding, este server no va a poder con ello. Necesitaréis mínimo 4 cores, se recomienda una puntuación mínima de 2000 en passmark para un sólo transcoding a 1080p.

Otras opciones económicas: https://www.kimsufi.com/es/servidores.xml 

Cuando tengamos el vps arrancado, tenemos que entrar por ssh. Seguramente en el panel nos indiquen cómo hacerlo pero lo normal es abrir un sesión en PuTTy (Windows) o Terminal (Mac OS).
```
ssh root@ipvps
```



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
N: creamos una nueva unidad.

7: del tipo Google Drive.

Dejamos client id y client secret en blanco.

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

En la sección de Downloads de su web (https://www.plex.tv/es/downloads/), copiamos el enlace de la versión de linux, en mi caso Ubuntu 64 bits.
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



## Optimizaciones para evitar transcoding

En los clientes/dispositivos que vayáis a utilizar hay que tener en cuenta un par de consideraciones respecto a la calidad de vídeo y los subtítulos para evitar la transcodificación.

Os dejo a ajustar en estas imágenes http://imgur.com/a/HoWyR.


Respecto al servidor, en mi caso detecté que estaba haciendo transcoding el audio ac3 5.1 a dispositivos iOS. 

Encontré una solución en el foro de plex. Hay que incluir los dos XML (iOS.xml y tvOS.xml) en el directorio Profiles.
```
wget iOS.xml tvOS.xml
mkdir /var/lib/plexmediaserver/Application Support/Plex Media Server/Profiles
cp iOS.xml tvOS.xml /var/lib/plexmediaserver/Application Support/Plex Media Server/Profiles
```



## plexpy
Podemos instalar el programa de monitorzación plexpy en nuestro vps para controlar lo que pasa en que cada momento en nuestro servidor y habilitar notificaciones personalizadas de cualquier evento que ocurra.


Instalamos git, clonamos el repositorio de plexpy y lo arrancamos.
```
sudo apt-get install git-core
cd /opt
sudo git clone https://github.com/JonnyWong16/plexpy.git
cd plexpy
python PlexPy.py
```

Ahora vamos crearlo como servicio y hacer que arranque por defecto con el sistema.
```
cd /etc/systemd/system/
sudo nano plexpy.service
```

Copiamos lo siguiente y lo pegamos en el nuevo archivo.
```
[Unit]
Description=PlexPy - Stats for Plex Media Server usage

[Service]
ExecStart=/opt/plexpy/PlexPy.py --quiet --daemon --nolaunch --config /opt/plexpy/config.ini --datadir /opt/plexpy
GuessMainPID=no
Type=forking
User=root
Group=root

[Install]
WantedBy=multi-user.target
``` 

Guardamos con Ctrl + X y Yes (Y).

Ya podremos acceder a la interfaz de plexpy y configurarlo a traves de http://ipvps:8081
