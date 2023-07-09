---
layout: post
title: Ejercicios con Docker II.
excerpt: Segundo taller de la unidad de Docker.
date: 2023-02-06
updatedDate: 2023-02-06
tags:
  - post
  - Docker
  - SRI
---

# Taller 2: Escenarios multicontenedor en Docker

## Despliegue de Nextcloud.

**Vamos a desplegar la aplicación nextcloud con una base de datos (puedes elegir mariadb o PostgreSQL) utilizando la aplicación docker-compose. Puedes coger cómo modelo el fichero docker-compose.yml el que hemos estudiado para desplegar WordPress.**

### Instala docker-compose en tu ordenador.

```txt
usuario@debian:~/docker/taller1$ sudo apt install docker-compose
Leyendo lista de paquetes... Hecho
Creando árbol de dependencias... Hecho
Leyendo la información de estado... Hecho
Se instalarán los siguientes paquetes adicionales:
  python3-cached-property python3-docker python3-dockerpty
  python3-docopt python3-texttable python3-websocket
Se instalarán los siguientes paquetes NUEVOS:
  docker-compose python3-cached-property python3-docker
  python3-dockerpty python3-docopt python3-texttable python3-websocket
0 actualizados, 7 nuevos se instalarán, 0 para eliminar y 31 no actualizados.
Se necesita descargar 301 kB de archivos.
Se utilizarán 1.647 kB de espacio de disco adicional después de esta operación.
¿Desea continuar? [S/n] s
```

### Dentro de un directorio crea un fichero docker-compose.yml para realizar el despliegue de nextcloud con una base de datos. Recuerda las variables de entorno y la persistencia de información.

```txt
usuario@debian:~/docker$ mkdir nextcloud
usuario@debian:~/docker$ cd nextcloud/
usuario@debian:~/docker/nextcloud$ nano docker-compose.yml
```

El contenido del fichero es el siguiente:

```txt
version: '3.7'
services:
  nextcloud:
    image: nextcloud:latest
    container_name: nextcloud
    restart: always
    ports:
      - 8080:80
    volumes:
      - nextcloud:/var/www/html
    environment:
      - MYSQL_HOST=mariadb
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
      - MYSQL_PASSWORD=nextcloud
      - MYSQL_ROOT_PASSWORD=nextcloud
    networks:
      - red-taller2
    depends_on:
      - mariadb
  mariadb:
    image: mariadb:latest
    container_name: mariadb
    restart: always
    volumes:
      - mariadb:/var/lib/mysql
    environment:
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
      - MYSQL_PASSWORD=nextcloud
      - MYSQL_ROOT_PASSWORD=nextcloud
    networks:
      - red-taller2
volumes:
  nextcloud:
  mariadb:
networks:
  red-taller2:
    driver: bridge
    driver_opts:
      com.docker.network.driver.mtu: 1420
```

### Levanta el escenario con docker-compose.

```txt
usuario@debian:~/docker/nextcloud$ docker-compose up -d
Creating mariadb ... done
Creating nextcloud ... done
```

### Muestra los contenedores con docker-compose.

```txt
usuario@debian:~/docker/nextcloud$ docker-compose ps
  Name                 Command               State          Ports        
-------------------------------------------------------------------------
mariadb     docker-entrypoint.sh mariadbd    Up      3306/tcp            
nextcloud   /entrypoint.sh apache2-for ...   Up      0.0.0.0:8080->80/tcp

```

### Accede a la aplicación y comprueba que funciona.

![IAW-T6-T2.1.png](/img/IAW-T6-T2.1.png)

### Comprueba el almacenamiento que has definido y que se ha creado una nueva red de tipo bridge.

```txt
usuario@debian:~/docker/nextcloud$ docker volume ls
DRIVER    VOLUME NAME
local     miweb
local     nextcloud_mariadb
local     nextcloud_nextcloud
usuario@debian:~/docker/nextcloud$ docker network ls
NETWORK ID     NAME                    DRIVER    SCOPE
0a056584eb52   bridge                  bridge    local
5a9854477fa0   host                    host      local
50145eea35b5   nextcloud_red-taller2   bridge    local
6819e5224e2d   none                    null      local
8812477bf491   red-taller1             bridge    local
```

### Borra el escenario con docker-compose.

```txt
usuario@debian:~/docker/nextcloud$ docker-compose down --volumes
Stopping nextcloud ... done
Stopping mariadb   ... done
Removing nextcloud ... done
Removing mariadb   ... done
Removing network nextcloud_red-taller2
Removing volume nextcloud_nextcloud
Removing volume nextcloud_mariadb
usuario@debian:~/docker/nextcloud$ docker network ls
NETWORK ID     NAME          DRIVER    SCOPE
0a056584eb52   bridge        bridge    local
5a9854477fa0   host          host      local
6819e5224e2d   none          null      local
8812477bf491   red-taller1   bridge    local
usuario@debian:~/docker/nextcloud$ docker volume ls
DRIVER    VOLUME NAME
local     miweb
usuario@debian:~/docker/nextcloud$ docker ps -a
CONTAINER ID   IMAGE       COMMAND                CREATED       STATUS                    PORTS                  NAMES
51e59b04534c   httpd:2.4   "httpd-foreground"     11 days ago   Exited (255) 4 days ago   0.0.0.0:8080->80/tcp   my-apache-app
5ea92851b3d2   ubuntu      "bash"                 11 days ago   Exited (0) 11 days ago                           contenedor1
76cb08ee11c3   ubuntu      "echo 'Hello world'"   11 days ago   Exited (0) 11 days ago                           priceless_hamilton
```

---
## **Documento realizado por:**

 ✒️ **Alejandro Montes Delgado** - *2º ASIR*