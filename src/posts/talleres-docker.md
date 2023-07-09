---
layout: post
title: Ejercicios con Docker.
excerpt: Primer taller de la unidad de Docker.
date: 2023-02-05
updatedDate: 2023-02-05
tags:
  - post
  - Docker
  - SRI
---

### Almacenamiento.

**Vamos a trabajar con volúmenes docker:**

#### 1. Crea un volumen docker que se llame miweb.

Creo el volumen y lo muestro.

```txt
usuario@debian:~$ docker volume create miweb
miweb
usuario@debian:~$ docker volume ls
DRIVER    VOLUME NAME
local     miweb
```

#### 2. Crea un contenedor desde la imagen php:7.4-apache donde montes en el directorio /var/www/html (que sabemos que es el DocumentRoot del servidor que nos ofrece esa imagen) el volumen docker que has creado.

Creamos el contenedor y listo los contenedores.

```txt
usuario@debian:~$ docker run -d --name miweb -v miweb:/var/www/html -p 8080:80 php:7.4-apache
Unable to find image 'php:7.4-apache' locally
7.4-apache: Pulling from library/php
a603fa5e3b41: Pull complete 
c428f1a49423: Pull complete 
156740b07ef8: Pull complete 
fb5a4c8af82f: Pull complete 
25f85b498fd5: Pull complete 
9b233e420ac7: Pull complete 
fe42347c4ecf: Pull complete 
d14eb2ed1e17: Pull complete 
66d98f73acb6: Pull complete 
d2c43c5efbc8: Pull complete 
ab590b48ea47: Pull complete 
80692ae2d067: Pull complete 
05e465aaa99a: Pull complete 
Digest: sha256:c9d7e608f73832673479770d66aacc8100011ec751d1905ff63fae3fe2e0ca6d
Status: Downloaded newer image for php:7.4-apache
645843f9d771e6d8ea8322ef4329af791f798d74806b863bd2884515fc58314a
usuario@debian:~$ docker ps
CONTAINER ID   IMAGE            COMMAND                  CREATED          STATUS          PORTS                  NAMES
645843f9d771   php:7.4-apache   "docker-php-entrypoi…"   20 seconds ago   Up 17 seconds   0.0.0.0:8080->80/tcp   miweb
```

#### 3. Utiliza el comando docker cp para copiar un fichero index.html (donde aparece tu nombre) en el directorio /var/www/html.

Lo primero que haremos será crear un index de prueba:

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Prueba</title>
  </head>
  <body>
    <h1>Alejandro Montes Delgado</h1>
    <p>Ejercicio3 de Almacenamiento</p>
  </body>
</html>
```

Ahora usamos el comando docker cp para enviar el index al contenedor:

```txt
usuario@debian:~$ docker cp index.html miweb:/var/www/html
```

#### 4. Accede al contenedor desde el navegador para ver la información ofrecida por el fichero index.html.

![IAW-T6-T1.1.png](/img/IAW-T6-T1.1.png)

#### 5. Borra el contenedor

Borro el contenedor y muestro que se ha borrado:

```txt
usuario@debian:~$ docker rm -f miweb
miweb
usuario@debian:~$ docker ps -a
CONTAINER ID   IMAGE       COMMAND                CREATED       STATUS                    PORTS                  NAMES
51e59b04534c   httpd:2.4   "httpd-foreground"     11 days ago   Exited (255) 4 days ago   0.0.0.0:8080->80/tcp   my-apache-app
5ea92851b3d2   ubuntu      "bash"                 11 days ago   Exited (0) 11 days ago                           contenedor1
76cb08ee11c3   ubuntu      "echo 'Hello world'"   11 days ago   Exited (0) 11 days ago                           priceless_hamilton

```

#### 6. Crea un nuevo contenedor y monta el mismo volumen como en el ejercicio anterior.

Creo de nuevo el contenedor con el mismo volumen y veremos que no se ha borrado la página web ya que al eliminar el contenedor no se elimina el volumen:

```txt
usuario@debian:~$ docker run -d --name miweb2 -v miweb:/var/www/html -p 8080:80 php:7.4-apache
98fd921d25f3a4525b2e9e0bcd26e36330d1d469daf007f33f3dfc0ff181c19c
```

#### Accede al contenedor desde el navegador para ver la información ofrecida por el fichero index.html. ¿Seguía existiendo ese fichero?

![IAW-T6-T1.2.png](/img/IAW-T6-T1.2.png)

**Vamos a trabajar con bind mount:**

#### 7. Crea un directorio en tu host y dentro crea un fichero index.html (donde aparece tu nombre).

```txt
usuario@debian:~$ mkdir web
usuario@debian:~$ cd web
usuario@debian:~/web$ nano index.html 
```

El contenido del index es el siguiente:

```txt
<!DOCTYPE html>
<html>
  <head>
    <title>Prueba</title>
  </head>
  <body>
    <h1>Alejandro Montes Delgado</h1>
    <p>Bind Mount</p>
  </body>
</html>
```

#### 8. Crea un contenedor desde la imagen php:7.4-apache donde montes en el directorio /var/www/html el directorio que has creado por medio de bind mount.

```txt
usuario@debian:~/web$ docker run -d --name miweb -v /home/usuario/web:/var/www/html -p 8080:80 php:7.4-apache
405f87ad2a41df0e6fde514d622c741d08da1d498e35420ef106036d5d1d86e6

```

#### 9. Accede al contenedor desde el navegador para ver la información ofrecida por el fichero index.html.

![IAW-T6-T1.3.png](/img/IAW-T6-T1.3.png)

#### 10. Modifica el contenido del fichero index.html en tu host y comprueba que al refrescar la página ofrecida por el contenedor, el contenido ha cambiado.

Edito el index  y queda de la siguiente manera:

```txt
<!DOCTYPE html>
<html>
  <head>
    <title>Prueba</title>
  </head>
  <body>
    <h1>Alejandro Montes Delgado</h1>
    <p>Bind Mount, esto es una modificacion.</p>
  </body>
</html>
```

Refresco la página y muestro el cambio:

![IAW-T6-T1.4.png](/img/IAW-T6-T1.4.png)

#### 11. Borra el contenedor

```txt
usuario@debian:~$ docker rm -f miweb
miweb
```

#### 12. Crea un nuevo contenedor y monta el mismo directorio como en el ejercicio anterior.

```txt
usuario@debian:~/web$ docker run -d --name miweb-nueva -v /home/usuario/web:/var/www/html -p 8080:80 php:7.4-apache
2074045b34c2643ac2186e117e51b9d7c6fdbec5e341d8ca3599f2fcf7f18f43
```

#### 13. Accede al contenedor desde el navegador para ver la información ofrecida por el fichero index.html. ¿Se sigue viendo el mismo contenido?

Sí, se sigue viendo el mismo contenido.

### Redes

**Despliegue de Nextcloud + mariadb/postgreSQL**

Vamos a desplegar la aplicación nextcloud con una base de datos (puedes elegir mariadb o PostgreSQL) (NOTA: Para que no te de errores utiliza la imagen mariadb:10.5). Te puede servir el ejercicio que hemos realizado para desplegar Wordpress. Para ello sigue los siguientes pasos:

#### 1. Crea una red de tipo bridge.

```txt
usuario@debian:~/web$ docker network create red-taller1
8812477bf491b70d5402f72b3336483c364aa56b9f746c59b6d931a03cf6ac57
usuario@debian:~/web$ docker network ls
NETWORK ID     NAME          DRIVER    SCOPE
0a056584eb52   bridge        bridge    local
5a9854477fa0   host          host      local
6819e5224e2d   none          null      local
8812477bf491   red-taller1   bridge    local

```

#### 2. Crea el contenedor de la base de datos conectado a la red que has creado. La base de datos se debe configurar para crear una base de dato y un usuario. Además el contenedor debe utilizar almacenamiento (volúmenes o bind mount) para guardar la información. Puedes seguir la documentación de mariadb o la de PostgreSQL.

```txt
usuario@debian:~/docker/taller1$ docker run -d --name nextcloud-db -v /home/usuario/docker/taller1/mariadb/:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=nextcloud -e MYSQL_DATABASE=nextcloud -e MYSQL_USER=nextcloud -e MYSQL_PASSWORD=nextcloud --network red-taller1 mariadb:10.5
Unable to find image 'mariadb:10.5' locally
10.5: Pulling from library/mariadb
7608715873ec: Pull complete 
578dfcf52320: Pull complete 
06e052319319: Pull complete 
64a5db893c1d: Pull complete 
a474552678e0: Pull complete 
08cf6eb2006a: Pull complete 
e8af41dcfbcd: Pull complete 
d1ea3aee3f93: Pull complete 
Digest: sha256:fd7c3b8e1f0306819aa90df0fae9573dabb09ef9ee88e4890aa97f9e6ad1b1d1
Status: Downloaded newer image for mariadb:10.5
33ac7a28b4871412b60ccc7306d52e456eb0b81c243d82b076d8e0a9ec6ce08b
```

#### 3. A continuación, siguiendo la documentación de la imagen nextcloud, crea un contenedor conectado a la misma red, e indica las variables adecuadas para que se configure de forma adecuada y realice la conexión a la base de datos. El contenedor también debe ser persistente usando almacenamiento.

```txt
usuario@debian:~/docker/taller1$ mkdir nextcloud
usuario@debian:~/docker/taller1$ docker run -d --name nextcloud -v /home/dusuario/docker/taller1/nextcloud:/var/www/html -e MYSQL_HOST=nextcloud-bd -e MYSQL_DATABASE=nextcloud -e MYSQL_USER=nextcloud -e MYSQL_PASSWORD=nextcloud -e MYSQL_ROOT_PASSWORD=nextcloud -p 8080:80 --network red-taller1 nextcloud
Unable to find image 'nextcloud:latest' locally
latest: Pulling from library/nextcloud
01b5b2efb836: Pull complete 
45244a9928d1: Pull complete 
139d4815e950: Pull complete 
9a420fd884ad: Pull complete 
1de46a46cfcd: Pull complete 
9cc46e699e97: Pull complete 
9a8d67ebc9db: Pull complete 
f9464a3489e8: Pull complete 
5fececf0d356: Pull complete 
6dd28e697cc8: Pull complete 
b38a56ab5f21: Pull complete 
ca91e5a7ae8b: Pull complete 
185f934955ec: Pull complete 
979a87e257ad: Pull complete 
63796aca9c04: Pull complete 
afba53389a1b: Pull complete 
6a79123df1c3: Pull complete 
790f82f8928c: Pull complete 
2bdee9c1e37b: Pull complete 
39b72fdca150: Pull complete 
Digest: sha256:c64895f402b6d8a2d6704bcd17fd26b5c0b69fcaccdd2185dc35e5fbe4f6488c
Status: Downloaded newer image for nextcloud:latest
13e2680e97784f790d300de21017c6381cc39ff75614be0ffa8e534863f5159d
```

#### 4. Accede a la aplicación usando un navegador web.

![IAW-T6-T1.5.png](/img/IAW-T6-T1.5.png)


---
## **Documento realizado por:**

 ✒️ **Alejandro Montes Delgado** - *2º ASIR*