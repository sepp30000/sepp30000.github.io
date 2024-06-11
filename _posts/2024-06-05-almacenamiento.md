---
layout: post
title:  "Sistema de almacenamiento"
date:   2024-06-04 02:33:18 +0200
---

## Sistema de almacenamiento

Terminada la parte de la red, llega la parte del almacenamiento. La empresa se le ha ofrecido una solución de almacenamiento interna por diversas razones:

- Tener una backup interno de los que se suba en la nube. Aunque la empresa tiene contratada una solución de almacenamiento en la nube, quiere poder realizar copias de seguridad de sus datos cuando ellos quieran.

- Las fotocopiadoras multifunción que reparan los técnicos utilizan para el escaneo diferentes opciones de almacenamiento (correo, ftp, pendrive...) y quieren realizar pruebas con ellas puesto que dependiendo donde vayan los equipos pueden usar opciones diferentes.

Con estás razones expuestas por el cliente se le ofrecen dos opciones:

- La creación de un servidor web **NGINX** que ofrezca un **WebDav** alojado en el servidor.

- Un servidor FTPS alojado en el servidor.

Estos dos servicios serán creados en docker con el objetivo de que puedan ser movidos al servidor de backup en caso de fallo.

### Instalación de Docker

El primer paso será la instalación de Docker en el servidor. Esto permite la creación de contenedores donde se podrán alojar los diferentes servicios que se van a ir alojando dentro del servidor.

**Esta instalación se realizará en Ubuntu Server 24.04**

#### Conexión por ssh

```bash
ssh sepp@192.168.11.11
```

#### Añadir clave oficial y repositorio oficial de **Docker**
 
```bash
# Añadir clave GPG oficial de docker:
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

```bash
# Añadir repositorio oficial a APT
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
```

#### Instalación de Docker

```bash
# Docker y sus plugins incluido compose
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

#### Inicio de servicio

```bash
sudo systemctl start docker
sudo systemctl enable docker
```

#### Hacer que no pida sudo cada vez que trabajamos con docker

```bash
sudo usermod -aG docker sepp
newgrp docker
```

#### Probar funcionamiento

```bash
docker run hello-world
```

![alt image](https://github.com/sepp30000/sepp30000.github.io/blob/main/_posts/Capturas/Docker-funciona.png?raw=true)

![alt image](https://github.com/sepp30000/sepp30000.github.io/blob/main/_posts/Capturas/docker%20sin%20sudo.png?raw=true)

### WebDAV

Ahora que docker es completamente funcional, es el momento de centrarse en la instalación de los servicios de almacenamiento. El primer servicio instalado será WebDAV. El protocolo WebDAV permite guardar, copiar, editar o compartir archivos de manera rápida. De manera similar a Samba o FTP. WebDAV tiene soporte multiplataforma y muestra los archivos dentro de un directorio como si de un archivo local se tratase.

Para montar este sistema de almacenamiento se usará NGINX, un servidor web muy utilizado y de código abierto. El primer paso será el montaje de la imagen del NGINX.

#### Configuración

**Dockerfile**
```bash
# Imagen base utlizada
FROM debian:10.6-slim

# Argumento de uid y gid usados
ARG UID=${UID:-1000}
ARG GID=${GID:-1000}

# Actualización de los repositorios, instalación de NGINX y utilidades necesarias más la eliminación de las listas de repositorios 
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
                    nginx \
                    nginx-extras \
                    apache2-utils && \
                    rm -rf /var/lib/apt/lists

# Modifica el UID y el GID de la carpeta www-data que almacena el WebDav
RUN usermod -u $UID www-data && groupmod -g $GID www-data


VOLUME /media

# Exposición del puerto 80
EXPOSE 80

# Copia el archivo de configuración creado a dentro del contenedor
COPY webdav.conf /etc/nginx/conf.d/default.conf
RUN rm /etc/nginx/sites-enabled/*

# Mueve el script creado a la raíz y le da permisos de ejecución
COPY entrypoint.sh /
RUN chmod +x entrypoint.sh

# Ejecuta el script y NGINX
CMD /entrypoint.sh && nginx -g "daemon off;"
```

**docker-compose.yml**
```bash
services:
  webdav:
    # Nombre del contenedor
    container_name: webdav
    # Nombre de la imagen o image id los dos funcionan
    image: seppwebdav:latest
    # Exposición de puertos
    ports:
      - 80:80
    # Volumenes creados
    volumes:
      - $HOME/docker/webdav:/media
    # Usuarios y contraseña
    environment:
      - USERNAME=paco
      - PASSWORD=12345
      - UID=1000
      - GID=1000
      - TZ=Europe/Madrid
    labels:
    # Opciones de traefik. Un balanceador de carga y proxy inverso
      - traefik.backend=webdav
      # Aquí si saliera fuera se pondría el dominio
      - traefik.frontend.rule=Host:localhost
      - traefik.docker.network=web
      # Reenvio de puertos
      - traefik.port=80
      # Habilita que lo gestiona trafic
      - traefik.enable=true
      # Medidas de seguridad
      - traefik.http.middlewares.securedheaders.headers.forcestsheader=true
      - traefik.http.middlewares.securedheaders.headers.sslRedirect=true
      - traefik.http.middlewares.securedheaders.headers.STSPreload=true
      - traefik.http.middlewares.securedheaders.headers.ContentTypeNosniff=true
      - traefik.http.middlewares.securedheaders.headers.BrowserXssFilter=true
      - traefik.http.middlewares.securedheaders.headers.STSIncludeSubdomains=true
      - traefik.http.middlewares.securedheaders.headers.stsSeconds=63072000
      - traefik.http.middlewares.securedheaders.headers.frameDeny=true
      - traefik.http.middlewares.securedheaders.headers.browserXssFilter=true
      - traefik.http.middlewares.securedheaders.headers.contentTypeNosniff=true
networks:
  web:
   external: true
```

**entrypoint.sh**

```bash
#!/bin/bash
# Creación del usuario que pedimos en el compose
if [ -n "$USERNAME" ] && [ -n "$PASSWORD" ]
then
        htpasswd -bc /etc/nginx/htpasswd $USERNAME $PASSWORD
else
    echo Using no auth.
        sed -i 's%auth_basic "Restricted";% %g' /etc/nginx/conf.d/default.conf
        sed -i 's%auth_basic_user_file htpasswd;% %g' /etc/nginx/conf.d/default.conf
fi
# Cambio de propietario de /media
mediaowner=$(ls -ld /media | awk '{print $3}')
if [ "$mediaowner" != "www-data" ]
then
    chown -R www-data:www-data /media
fi
```

**webdav.conf**
```bash
dav_ext_lock_zone zone=a:10m;

server {
  set $webdav_root "/media/";
  # Necesitaran usuario y contraseña y la ubicación de esta
  auth_basic "Restricted";
  auth_basic_user_file /etc/nginx/htpasswd;
  dav_ext_lock zone=a;

  location / {

        root                    $webdav_root;
        error_page              599 = @propfind_handler;
        error_page              598 = @delete_handler;
        error_page              597 = @copy_move_handler;
        open_file_cache         off;

        access_log /var/log/nginx/webdav_access.log;
        error_log /var/log/nginx/webdav_error.log debug;

        send_timeout            3600;
        client_body_timeout     3600;
        keepalive_timeout       3600;
        lingering_timeout       3600;
        client_max_body_size    10G;

        if ($request_method = PROPFIND) {
                return 599;
        }

        if ($request_method = PROPPATCH) { 
                add_header      Content-Type 'text/xml';
                return          207 '<?xml version="1.0"?><a:multistatus xmlns:a="DAV:"><a:response><a:propstat><a:status>HTTP/1.1 200 OK</a:status></a:propstat></a:response></a:multistatus>';
        }

        if ($request_method = MKCOL) { 
                rewrite ^(.*[^/])$ $1/ break;
        }

        if ($request_method = DELETE) {
                return 598;
        }

        if ($request_method = COPY) {
                return 597;
        }

        if ($request_method = MOVE) {
                return 597;
        }

        dav_methods             PUT MKCOL;
        dav_ext_methods         OPTIONS LOCK UNLOCK;
        create_full_put_path    on;
        min_delete_depth        0;
        dav_access              user:rw group:rw all:rw;

        autoindex               on;
        autoindex_exact_size    on;
        autoindex_localtime     on;

        if ($request_method = OPTIONS) {
                add_header      Allow 'OPTIONS, GET, HEAD, POST, PUT, MKCOL, MOVE, COPY, DELETE, PROPFIND, PROPPATCH, LOCK, UNLOCK';
                add_header      DAV '1, 2';
                return 200;
        }
  }
  # Location establece directivas para mover, eliminar, copiar archivos
  location @propfind_handler {
        internal;

        open_file_cache off;
        if (!-e $webdav_root/$uri) { 
                return 404;
        }
        root                    $webdav_root;
        dav_ext_methods         PROPFIND;
  }
  location @delete_handler {
        internal;

        open_file_cache off;
        if (-d $webdav_root/$uri) { 
                rewrite ^(.*[^/])$ $1/ break;
        }
        root                    $webdav_root;
        dav_methods             DELETE;
  }
  location @copy_move_handler {
        internal;

        open_file_cache off;
        if (-d $webdav_root/$uri) { 
                more_set_input_headers 'Destination: $http_destination/';
                rewrite ^(.*[^/])$ $1/ break;
        }
        root                    $webdav_root;
        dav_methods             COPY MOVE;
  }
}
```

Con todo esto ya está preparada la configuración estos archivos serán guardados en un repositorio aparte [Repositorio Webdav](https://github.com/sepp30000/WEBDAV) para poder guardarlo por si acaso.

#### Levantar Docker

Ahora que está preparado el entorno se puede levantar el servicio de web, para ello se creará la imagen del contenedor:

```bash
# Se hace dentro de la carpeta que está el dockerfile
docker build -t seppwebdav:latest
```
Y ya se puede levantar el contenedor

```bash
docker compose up -d
```

Y comprobamos que el contenedor está levantado

![alt image](https://github.com/sepp30000/sepp30000.github.io/blob/main/_posts/Capturas/webdav1.png?raw=true)

#### Comprobar funcionamiento

Ya levantado el contenedor, en este caso es en MACOSX, desde el finder nos dirigimos a **ir-conectarse a un servidor**. Desde allí ponemos la dirección de la puerta de enlace de nuestra red (ya que por la redirección de puertos nos mandará al servidor) y ya estaría preparado el WebDAV.

![alt image](https://github.com/sepp30000/sepp30000.github.io/blob/main/_posts/Capturas/webdav3.png?raw=true)

![alt image](https://github.com/sepp30000/sepp30000.github.io/blob/main/_posts/Capturas/webdav4.png?raw=true)

### FTP

Ahora llega el momento del **FTP**, en este caso se va a realizar un FTPs también usando *Docker*. El repositorio se encuentra ubicado en el siguiente [enlace](https://github.com/sepp30000/FTP).

Todo comienza con la creación de los certificados para en la máquina del servidor con el objetivo de que desde la creación de un volumen sean los mismos para el docker como el servidor

```bash
sudo su
mkdir -p /etc/ssl/private
openssl dhparam -out /etc/ssl/private/pure-ftpd-dhparams.pem 2048
openssl req -x509 -nodes -newkey rsa:2048 -sha256 -keyout \
    /etc/ssl/private/pure-ftpd.pem \
    -out /etc/ssl/private/pure-ftpd.pem
chmod 600 /etc/ssl/private/*.pem
```

El siguiente paso será crear el *docker-compose* del servidor FTP y levantarlo

```bash
services:
  ftpd_server:
    image: stilliard/pure-ftpd
    container_name: pure-ftpd
    ports:
    # Redirección de puertos
      - "21:21"
      - "30000-30009:30000-30009"
    volumes: 
      # Creación del volumen del ftp
      - "/home/sepp/ftps/ftp:/home/sepp/"
      # Certificados
      - "/etc/ssl/private/:/etc/ssl/private/"
    environment:
      FTP_USER_NAME: sepp
      FTP_USER_PASS: mipass
      FTP_USER_HOME: /home/sepp

    restart: always
```

```bash
docker compose up -d
```

Ahora desde Filezilla se comprueba el funcionamiento.

![alt image](https://github.com/sepp30000/sepp30000.github.io/blob/main/_posts/Capturas/FTP.png?raw=true)

Nos aparecerá el certificado reconociéndolo. 

![alt image](https://github.com/sepp30000/sepp30000.github.io/blob/main/_posts/Capturas/Certificado.png?raw=true)

Con esto ya tendríamos el FTP habilitado para usarse.
