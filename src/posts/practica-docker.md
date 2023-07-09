---
layout: post
title: "Práctica: Implantación de aplicaciones web PHP en docker."
excerpt: Práctica 1 de docker.
date: 2023-02-16
updatedDate: 2023-02-16
tags:
  - post
  - IAW
---

Imaginemos que el equipo de desarrollo de nuestra empresa ha desarrollado una aplicación PHP que se llama BookMedik (https://github.com/evilnapsis/bookmedik).

Queremos crear una imagen Docker para implantar dicha aplicación.

Tenemos que tener en cuenta los siguientes aspectos:

**Contenedor mariadb**


  * Es necesario que nuestra aplicación guarde su información en un contenedor docker mariadb.
  * El script para generar la base de datos y los registros lo encuentras en el repositorio y se llama schema.sql. Debes crear un usuario con su contraseña en la base de datos. La base de datos se llama bookmedik y se crea al ejecutar el script.
  * Ejecuta el contenedor mariadb y carga los datos del script schema.sql. Para más información.
  * El contenedor mariadb debe tener un volumen para guardar la base de datos.

**Contenedor bookmedik**


  * Vamos a crear tres versiones de la imagen que nos permite implantar la aplicación PHP.
  * La imagen debe crear las variables de entorno necesarias con datos de conexión por defecto.
  * Al crear un contenedor a partir de estas imágenes se ejecutará un script bash que realizará las siguientes tareas:
    * Modifique el fichero core\controller\Database.php para que lea las variables de entorno. Para obtener las variables de entorno en PHP usar la función getenv. Para más información.
    * Inicialice la base de datos con el fichero schema.sql.
    * Ejecute el servidor web.
  * El contenedor que creas debe tener un volumen para guardar los logs del servidor web.
  * La imagen la tienes que crear en tu entorno de desarrollo con el comando docker build.

---

Lo primero que haremos será crear un contenedor con la imagen de mariadb con las variables de entorno que nos han indicado y crearemos una nueva red para la aplicación web.

```txt
usuario@debian:~/docker/practica$ docker network create red_bookmedik
f5873a0d59ee09ca18fed495a0cdae9b9dbd4251e926ee083c77cd906a696b86
usuario@debian:~/docker/practica$ docker run -d --name bd_mariadb -v bookmedik_vol:/var/lib/mysql --network red_bookmedik -e MARIADB_ROOT_PASSWORD=root -e MARIADB_DATABASE=bookmedik -e MARIADB_USER=bookmedik -e MARIADB_PASSWORD=bookmedik mariadb
fcc6779f9140829742d3db89892d38d5103a31902001090fb8901fd1fe690669
```

### Creación de una imagen docker con una aplicación web desde una imagen base.

* **Vamos a crear una imagen que se llame usuario/bookmedik:v1.**

En primer lugar, haremos un fork al repositorio de bookmedik y lo clonaremos a nuestro equipo. Editaremos el fichero schema.sql y quitaremos las siguientes líneas:

```txt
create database bookmedik;
use bookmedik;
```

Ahora, modificaremos el fichero `Database.php` para que se configure con las variables de entorno que indicamos al crear el contenedor.

la ruta es la siguiente:

```txt
usuario@debian:~/docker/practica/bookmedik$ nano core/controller/Database.php
```

El fichero queda de la siguiente manera:

```php
<?php
class Database {
        public static $db;
        public static $con;
        function Database(){
                $this->user=getenv('USUARIO_BOOKMEDIK');$this->pass=getenv('CONTRA_BOOKMEDIK');$this->host=getenv('DATABASE_HOST');$this->ddbb=getenv('NOMBRE_DB');
        }

        function connect(){
                $con = new mysqli($this->host,$this->user,$this->pass,$this->ddbb);
                $con->query("set sql_mode=''");
                return $con;
        }

        public static function getCon(){
                if(self::$con==null && self::$db==null){
                        self::$db = new Database();
                        self::$con = self::$db->connect();
                }
                return self::$con;
        }
}
?>
```

Una vez hecho esto, ya podremos crear el Dockerfile con el que crearemos la imagen.

* **Crea una imagen docker con la aplicación desde una imagen base de debian o ubuntu.**

El contenido del Dockerfile es el siguiente:

```txt
FROM debian:bullseye
MAINTAINER Alejandro Montes "alejandro@alejandro-montes.es"
RUN apt update && apt upgrade -y && apt install apache2 libapache2-mod-php php php-mysql mariadb-client -y && apt clean && rm -rf /var/lib/apt/lists/*
ADD bookmedik /var/www/html/
ADD script.sh /opt/
RUN chmod +x /opt/script.sh && rm /var/www/html/index.html
ENTRYPOINT ["/opt/script.sh"]
```

Como vemos, el Dockerfile ejecuta un script, el contenido del script es el siguiente:

```bash
#! /bin/sh

mysql -u $USUARIO_BOOKMEDIK --password=$CONTRA_BOOKMEDIK -h $DATABASE_HOST $NOMBRE_DB < /var/www/html/schema.sql

/usr/sbin/apache2ctl -D FOREGROUND
```

Una vez hecho esto, ya podemos crear la imagen:

```txt
usuario@debian:~/docker/practica$ docker build -t alemd01/bookmedik:v1 .
Sending build context to Docker daemon   5.78MB
Step 1/7 : FROM debian:bullseye
bullseye: Pulling from library/debian
1e4aec178e08: Pull complete 
Digest: sha256:43ef0c6c3585d5b406caa7a0f232ff5a19c1402aeb415f68bcd1cf9d10180af8
Status: Downloaded newer image for debian:bullseye
 ---> 54e726b437fb
Step 2/7 : MAINTAINER Alejandro Montes "alejandro@alejandro-montes.es"
 ---> Running in 1647cfd1e95c
Removing intermediate container 1647cfd1e95c
 ---> a59b1bdf1432
Step 3/7 : RUN apt update && apt upgrade -y && apt install apache2 libapache2-mod-php php php-mysql mariadb-client -y && apt clean && rm -rf /var/lib/apt/lists/*
 ---> Running in 0f4011dfed2b
```

Como podemos ver la imagen se ha creado correctamente:

```txt
usuario@debian:~/docker/practica$ docker images
REPOSITORY                TAG          IMAGE ID       CREATED          SIZE
alemd01/bookmedik         v1           d30f6251c040   14 minutes ago   302MB
debian                    bullseye     54e726b437fb   10 days ago      124MB
alemd01/mi_servidor_web   v2           4c59dc832620   12 days ago      239MB
alemd01/mi_servidor_web   v1           b1421e4328a1   12 days ago      239MB
nextcloud                 latest       2af5f013b660   2 weeks ago      1.02GB
debian                    latest       20473158e8b3   2 weeks ago      124MB
mariadb                   10.5         817e0c16a766   2 weeks ago      402MB
mariadb                   latest       039bd724508b   2 weeks ago      410MB
httpd                     2.4          6e794a483258   4 weeks ago      145MB
ubuntu                    latest       6b7dfa7e8fdb   2 months ago     77.8MB
php                       7.4-apache   20a3732f422b   3 months ago     453MB
hello-world               latest       feb5d9fea6a5   17 months ago    13.3kB
```

### Despliegue en el entorno de desarrollo.

* **Crea un script con docker-compose que levante el escenario con los dos contenedores.**
* **Recuerda que para acceder a la aplicación: Usuario: admin, contraseña: admin.**

Vamos a crear el fichero docker-compose con la configuración necesaria para levantar los dos contenedores:

```yaml
version: '3.1'
services:
  bookmedik:
    container_name: bookmedik-app
    image: alemd01/bookmedik:v1
    restart: always
    environment:
      USUARIO_BOOKMEDIK: bookmedik
      CONTRA_BOOKMEDIK: bookmedik
      DATABASE_HOST: bd_mariadb
      NOMBRE_DB: bookmedik
    ports:
      - 8081:80
    depends_on:
      - db
  db:
    container_name: bd_mariadb
    image: mariadb
    restart: always
    environment:
      MARIADB_ROOT_PASSWORD: root
      MARIADB_DATABASE: bookmedik
      MARIADB_USER: bookmedik
      MARIADB_PASSWORD: bookmedik
    volumes:
      - mariadb_data:/var/lib/mysql
volumes:
    mariadb_data:
```

Levantamos los contenedores:

```txt
usuario@debian:~/docker/practica$ docker-compose up -d
Creating bd_mariadb ... done
Creating bookmedik-app ... done
```

Muestro que están creados:

```txt
usuario@debian:~/docker/practica$ docker ps
CONTAINER ID   IMAGE                  COMMAND                  CREATED          STATUS          PORTS                  NAMES
5cc92c18f18c   alemd01/bookmedik:v1   "/opt/script.sh"         11 minutes ago   Up 11 minutes   0.0.0.0:8081->80/tcp   bookmedik-app
9045e0b5dd8a   mariadb                "docker-entrypoint.s…"   11 minutes ago   Up 11 minutes   3306/tcp               bd_mariadb
```

Muestro que la aplicación funciona:

![IAW-P7-T1.1.png](/img/IAW-P7-T1.1.png)

### Creación de una imagen docker con una aplicación web desde una imagen PHP.


* **Vamos a crear una imagen que se llame usuario/bookmedik:v2.**
* **Realiza la imagen docker de la aplicación a partir de la imagen oficial PHP que encuentras en docker hub. Lee la documentación de la imagen para configurar una imagen con apache2 y php, además seguramente tengas que instalar alguna extensión de php.**
* **Modifica el fichero docker-compose.yml` para probar esta imagen.**

Adjunto el Dockerfile:

```txt
FROM php:7.4-apache-bullseye
MAINTAINER Alejandro Montes Delgado "aaleemd11@gmail.com"
RUN apt update && apt upgrade -y && docker-php-ext-install mysqli pdo pdo_mysql && apt install mariadb-client -y && apt clean && rm -rf /var/lib/apt/lists/*
ADD bookmedik /var/www/html/
ADD script.sh /opt/
RUN chmod +x /opt/script.sh
ENTRYPOINT ["/opt/script.sh"]
```

Creamos la imagen nueva:

```txt
usuario@debian:~/docker/practica/tarea3$ docker build -t alemd01/bookmedik:v2 .Sending build context to Docker daemon   5.78MB
Step 1/7 : FROM php:7.4-apache-bullseye
7.4-apache-bullseye: Pulling from library/php
Digest: sha256:c9d7e608f73832673479770d66aacc8100011ec751d1905ff63fae3fe2e0ca6d
Status: Downloaded newer image for php:7.4-apache-bullseye
 ---> 20a3732f422b
Step 2/7 : MAINTAINER Alejandro Montes Delgado "aaleemd11@gmail.com"
 ---> Running in 8a5fb4ef6046
Removing intermediate container 8a5fb4ef6046
 ---> 62d2489e4625
Step 3/7 : RUN apt update && apt upgrade -y && docker-php-ext-install mysqli pdo pdo_mysql && apt install mariadb-client -y && apt clean && rm -rf /var/lib/apt/lists/*
 ---> Running in 50204a750e86
```

Ahora en el docker-compose cambiamos la version a v2:

```yaml
version: '3.1'
services:
  bookmedik:
    container_name: bookmedik-app
    image: alemd01/bookmedik:v2
    restart: always
    environment:
      USUARIO_BOOKMEDIK: bookmedik
      CONTRA_BOOKMEDIK: bookmedik
      DATABASE_HOST: bd_mariadb
      NOMBRE_DB: bookmedik
    ports:
      - 8081:80
    depends_on:
      - db
  db:
    container_name: bd_mariadb
    image: mariadb
    restart: always
    environment:
      MARIADB_ROOT_PASSWORD: root
      MARIADB_DATABASE: bookmedik
      MARIADB_USER: bookmedik
      MARIADB_PASSWORD: bookmedik
    volumes:
      - mariadb_data:/var/lib/mysql
volumes:
    mariadb_data:
```

Levantamos los contenedores(tenemos que borrar los contenedores anteriores si no se han borrado):

```txt
usuario@debian:~/docker/practica/tarea3$ docker-compose up -d
Creating bd_mariadb ... done
Creating bookmedik-app ... done
```

Compruebo que los contenedores se han creado correctamente:

```txt
usuario@debian:~/docker/practica/tarea3$ docker ps
CONTAINER ID   IMAGE                  COMMAND                  CREATED              STATUS              PORTS                  NAMES
27288d25383b   alemd01/bookmedik:v2   "/opt/script.sh"         About a minute ago   Up About a minute   0.0.0.0:8081->80/tcp   bookmedik-app
17cb7aba76cd   mariadb                "docker-entrypoint.s…"   About a minute ago   Up About a minute   3306/tcp               bd_mariadb
```

Accedo nuevamente a bookmedik:

![IAW-P7-T1.2.png](/img/IAW-P7-T1.2.png)

### Ejecución de una aplicación PHP en docker con nginx (OPTATIVA).

* **Vamos a crear una imagen que se llame usuario/bookmedik:v3.**
* **En este caso queremos usar un contenedor que utilice nginx para servir la aplicación PHP. Puedes crear la imagen desde una imagen base debian o ubuntu o desde la imagen oficial de nginx.**
* **Vamos a crear otro contenedor que sirva php-fpm.**
* **Para que funcione de forma adecuada el php-fpm tiene que tener acceso al directorio donde se encuentra la aplicación.**
* **Y finalmente nuestro contenedor con la aplicación.**
* **Crea un script con docker compose que levante el escenario con los tres contenedores.**

*SIN HACER*

### Puesta en producción de nuestra aplicación.

* **Elige una de las tres imágenes y súbela a Docker Hub.**

He elegido la versión v2, realizamos un docker push para subir la imagen:

```txt
usuario@debian:~/docker/practica$ docker push alemd01/bookmedik:v2
The push refers to repository [docker.io/alemd01/bookmedik]
b29d0d6e7e6e: Pushed 
1da8c39d9650: Pushed 
57f2b3e3056d: Pushed 
9a664d5dd358: Pushed 
3d33242bf117: Mounted from library/php 
529016396883: Mounted from library/php 
5464bcc3f1c2: Mounted from library/php 
28192e867e79: Mounted from library/php 
d173e78df32e: Mounted from library/php 
0be1ec4fbfdc: Mounted from library/php 
30fa0c430434: Mounted from library/php 
a538c5a6e4e0: Mounted from library/php 
e5d40f64dcb4: Mounted from library/php 
44148371c697: Mounted from library/php 
797a7c0590e0: Mounted from library/php 
f60117696410: Mounted from library/php 
ec4a38999118: Mounted from library/php 
v2: digest: sha256:f12b29054e777e6b27a0672dfed50d4e8b752837c1d355db6a0e6cb8faded808 size: 3872
```

En nuestro proveedor de DNS, registramos un nuevo subdominio para bookmedik:

![IAW-P7-T1.3.png](/img/IAW-P7-T1.3.png)

Ahora en el vps creamos un certificado para ese subdominio:

```txt
root@nabil:~# certbot certonly --standalone -d bookmedik.alejandro-montes.es
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator standalone, Installer None
Attempting to parse the version 1.32.2 renewal configuration file found at /etc/letsencrypt/renewal/mail.alejandro-montes.es.conf with version 1.12.0 of Certbot. This might not work.
Requesting a certificate for bookmedik.alejandro-montes.es
Performing the following challenges:
http-01 challenge for bookmedik.alejandro-montes.es
Waiting for verification...
Cleaning up challenges

IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/bookmedik.alejandro-montes.es/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/bookmedik.alejandro-montes.es/privkey.pem
   Your certificate will expire on 2023-05-25. To obtain a new or
   tweaked version of this certificate in the future, simply run
   certbot again. To non-interactively renew *all* of your
   certificates, run "certbot renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le

```

* **En tu VPS instala Docker y utilizando el docker-compose.yml correspondiente, crea un contenedor en ella de la aplicación.**

Ahora procederemos a la instalación de docker y docker-compose:

```txt
alemd@nabil:~$ sudo apt install docker.io docker-compose
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following packages were automatically installed and are no longer required:
  libevent-2.1-7 libgnutls-dane0 libunbound8
Use 'sudo apt autoremove' to remove them.
The following additional packages will be installed:
  cgroupfs-mount containerd libintl-perl libintl-xs-perl
  libmodule-find-perl libmodule-scandeps-perl libproc-processtable-perl
  libsort-naturally-perl libyaml-0-2 needrestart python3-attr
  python3-cached-property python3-docker python3-dockerpty
  python3-docopt python3-importlib-metadata python3-jsonschema
  python3-more-itertools python3-pyrsistent python3-texttable
  python3-websocket python3-yaml python3-zipp runc tini
Suggested packages:
  containernetworking-plugins docker-doc aufs-tools btrfs-progs
  debootstrap rinse rootlesskit xfsprogs zfs-fuse | zfsutils-linux
  needrestart-session | libnotify-bin iucode-tool python-attr-doc
  python-jsonschema-doc
Recommended packages:
  criu
```

creamos el siguiente docker-compose para levantar el escenario:

```yaml
version: '3.1'
services:
  bookmedik:
    container_name: bookmedik-app
    image: alemd01/bookmedik:v2
    restart: always
    environment:
      USUARIO_BOOKMEDIK: bookmedik
      CONTRA_BOOKMEDIK: bookmedik
      DATABASE_HOST: bd_mariadb
      NOMBRE_DB: bookmedik
    ports:
      - 8088:80
    depends_on:
      - db
  db:
    container_name: bd_mariadb
    image: mariadb
    restart: always
    environment:
      MARIADB_ROOT_PASSWORD: root
      MARIADB_DATABASE: bookmedik
      MARIADB_USER: bookmedik
      MARIADB_PASSWORD: bookmedik
    volumes:
      - mariadb_data:/var/lib/mysql
volumes:
    mariadb_data:

```

Simplemente he cambiado el puerto porque ya está en uso. Ahora levantamos el escenario:

```txt
root@nabil:/home/alemd/docker# docker-compose up -d
Creating network "docker_default" with the default driver
Creating volume "docker_mariadb_data" with default driver
Pulling db (mariadb:)...
latest: Pulling from library/mariadb
10ac4908093d: Pulling fs layer
44779101e748: Downloading [=====================>                         10ac4908093d: Downloading [>                                              10ac4908093d: Downloading [=====================>                             ]   12.8MB/30.43MB
10ac4908093d: Downloading [===============================================10ac4908093d: Download complete
10ac4908093d: Extracting [>                                                  ]  327.7kB/30.43MBd complete

10ac4908093d: Extracting [====>                                              ]  2.621MB/30.43MBd complete

86f876b891f6: Downloading [>                                              10ac4908093d: Extracting [======>                                            ]  3.932MB/30.43MBding [==========>                                    10ac4908093d: Pull complete
44779101e748: Pull complete
a721db3e3f3d: Pull complete
1850a929b84a: Pull complete
c3aa31fb301e: Pull complete
86f876b891f6: Pull complete
fe3602f5cecf: Pull complete
8fe289a9f834: Pull complete
Digest: sha256:dd0f492b6b6e7bb4aa707181b799d4efe42cb3a9f6012ec3dbaf326d402151e8
Status: Downloaded newer image for mariadb:latest
Creating bd_mariadb ... done
Creating bookmedik-app ... done
```

* **Configura el nginx de tu VPS para que haga de proxy inverso y nos permita acceder a la aplicación con https://bookmedik.tudominio.xxx.**

Ahora tendremos que configurar un virtualhost en nuestro vps:

```txt
root@nabil:~# nano /etc/nginx/sites-available/bookmedik
```

El contenido del fichero es el siguiente:

```txt
server {
        listen 80;
        listen [::]:80;

        server_name bookmedik.alejandro-montes.es;

        return 301 https://$host$request_uri;
}

server {
        listen 443 ssl http2;
        listen [::]:443 ssl http2;

        ssl    on;
        ssl_certificate /etc/letsencrypt/live/bookmedik.alejandro-montes.es/fullchain.pem;
        ssl_certificate_key     /etc/letsencrypt/live/bookmedik.alejandro-montes.es/privkey.pem;

        index index.html index.php index.htm index.nginx-debian.html;

        server_name bookmedik.alejandro-montes.es;

        location / {
                proxy_pass http://localhost:8088;
                include proxy_params;
        }

}
```

Una vez creado el virtualhost, tenemos que crear el enlace simbólico:

```txt
root@nabil:~# ln -s /etc/nginx/sites-available/bookmedik /etc/nginx/sites-enabled/bookmedik
```

Lo que queda, es reiniciar nginx y probar que funciona:

```txt
root@nabil:~# systemctl restart nginx
```

![IAW-P7-T1.4.png](/img/IAW-P7-T1.4.png)

### Modificación de la aplicación.

* **En el entorno de desarrollo vamos a hacer una modificación de la aplicación. Por ejemplo modifica el fichero core/app/view/login-view.php y pon tu nombre en la línea `<h4 class="title">Acceder a BookMedik</h4>`.**
* **Vamos a trabajar con la primera imagen que construimos. Vuelve a crear la imagen con la etiqueta v1_2.**
* **Cambia el docker-compose para probar el cambio.**
* **Modifica la aplicación en producción.**

Lo primero que haremos será editar el fichero login-view.php y la línea tiene que quedar de la siguiente forma:

```php
      <h4 class="title">Alejandro Montes delgado</h4>
```

Construimos nuevamente la imagen con la etiqueta v1_2:

```txt
usuario@debian:~/docker/practica/tarea5$ docker build -t alemd01/bookmedik:v1_2 .
Sending build context to Docker daemon  5.782MB
Step 1/7 : FROM debian:bullseye
 ---> 54e726b437fb
Step 2/7 : MAINTAINER Alejandro Montes "alejandro@alejandro-montes.es"
 ---> Using cache
 ---> 6170a85f83e3
Step 3/7 : RUN apt update && apt upgrade -y && apt install apache2 libapache2-mod-php php php-mysql mariadb-client -y && apt clean && rm -rf /var/lib/apt/lists/*
 ---> Using cache
 ---> c2019c7047c1
Step 4/7 : ADD bookmedik /var/www/html/
 ---> 943a8e7040b8
Step 5/7 : ADD script.sh /opt/
 ---> 9aa836a5e135
Step 6/7 : RUN chmod +x /opt/script.sh && rm /var/www/html/index.html
 ---> Running in e0df57437824
Removing intermediate container e0df57437824
 ---> a9a6a4b3f1f6
Step 7/7 : ENTRYPOINT ["/opt/script.sh"]
 ---> Running in eea03e1f5bce
Removing intermediate container eea03e1f5bce
 ---> 9bc356ec059d
Successfully built 9bc356ec059d
Successfully tagged alemd01/bookmedik:v1_2
```

Modificamos el docker compose para aplicar el cambio solo tenemos que cambiar la etiqueta y poner la nueva:

```txt
    image: alemd01/bookmedik:v1_2
```

Levantamos el escenario en desarrollo aún:

```txt
usuario@debian:~/docker/practica/tarea5$ docker-compose up -d
Creating bd_mariadb ... done
Creating bookmedik-app ... done
```

Probamos que funciona:

![IAW-P7-T1.5.png](/img/IAW-P7-T1.5.png)

Una vez comprobado que se han aplicado los cambios correctamente y que accede, lo pasaremos a producción para ello haremos un docker push de la imagen:

```txt
usuario@debian:~/docker/practica/tarea5$ docker push alemd01/bookmedik:v1_2
The push refers to repository [docker.io/alemd01/bookmedik]
6462ef003814: Pushed 
27aea5a6faf2: Pushed 
610af938ab10: Pushed 
61d7b4db50a6: Pushed 
8e396a1aad50: Mounted from library/debian 
v1_2: digest: sha256:21138d2ff768626704f6ef484d256dbddbec7b08b1ce63c75ebabab381adb75f size: 1366
```

Ahora que nuestra imagen está lista, pasaremos a producción y lo que haremos será cambiar en el docker compose la versión. La línea queda igual que la anterior.

```txt
    image: alemd01/bookmedik:v1_2
```

Levantamos el escenario:

```txt
root@nabil:/home/alemd/docker# docker-compose up -d
Pulling bookmedik (alemd01/bookmedik:v1_2)...
v1_2: Pulling from alemd01/bookmedik
1e4aec178e08: Downloading [>                                              1e4aec178e08: Downloading [>                                                  ]  1.081MB/55.05MBfs layer
dcb3cfc7cb33: Waiting
1e4aec178e08: Downloading [=>                                                 ]  2.163MB/55.05MB

1e4aec178e08: Downloading [===>                                               ]  3.783MB/55.05MB
1e4aec178e08: Downloading [======>                                        1e4aec178e08: Downloading [=============>                                 1e4aec178e08: Downloading [====================>                          1e4aec178e08: Downloading [==========================>                        ]  29.04MB/55.05MB
1e4aec178e08: Downloading [=================================>             1e4aec178e08: Downloading [========================================>      1e4aec178e08: Downloading [==============================================>    ]  51.52MB/55.05MB complete
1e4aec178e08: Download complete
1e4aec178e08: Extracting [>                                               1e4aec178e08: Extracting [==>                                                ]  2.785MB/55.05MBding [===============================================1e4aec178e08: Pull complete
b8fab135fbf3: Pull complete
2728d62bb111: Pull complete
dcb3cfc7cb33: Pull complete
71b3df28513f: Pull complete
Digest: sha256:21138d2ff768626704f6ef484d256dbddbec7b08b1ce63c75ebabab381adb75f
Status: Downloaded newer image for alemd01/bookmedik:v1_2
bd_mariadb is up-to-date
Recreating bookmedik-app ... done
```

Accedemos nuevamente a https://bookmedik.alejandro-montes.es y veremos los cambios aplicados:

![IAW-P7-T1.6.png](/img/IAW-P7-T1.6.png)

Muestro que funciona después del login:

![IAW-P7-T1.7.png](/img/IAW-P7-T1.7.png)


---
## **Documento realizado por:**

 ✒️ **Alejandro Montes Delgado** - *2º ASIR*