---
layout: post
title: "Práctica 2: Implantación de aplicaciones web Python en docker."
excerpt: Práctica 2 de docker.
date: 2023-02-27
updatedDate: 2023-02-27
tags:
  - post
  - IAW
---

Queremos desplegar en docker la aplicación escrita en python django: tutorial de django, que desplegamos en la tarea Despliegue de aplicaciones python.

Tienes que tener en cuenta los siguientes aspectos:

  * La aplicación debe guardar los datos en una base de datos mariadb.
  * La aplicación se podrá configurar para indicar los parámetros de conexión a la base de datos: usuario, contraseña, host y base de datos.
  * La aplicación deberá tener creado un usuario administrador para el acceso.

### Crea una imagen docker para poder desplegar un contenedor con la aplicación. La imagen la puedes hacer desde una imagen base o desde la imagen oficial de python.

Vamos a crear una red para que ambos contenedores se conecten:

```txt
usuario@debian:~$ docker network create django-net
d57e165b54b222fa43b8ba34e8cdf9c745d99850d45ba512b76b7d53446172f3
```

Creamos el contenedor de mariadb con las siguientes variables:

```txt
usuario@debian:~$ docker run -d --name mariadb -v vol_polls:/var/lib/mysql --network django-net -e MARIADB_ROOT_PASSWORD=root -e MARIADB_USER=django -e MARIADB_PASSWORD=django -e MARIADB_DATABASE=django mariadb
24ceb57a2f2a8426f75f4592d0ff5cb8a654f7735962ec17a40b7c3282625810

```

Ahora modificaremos nuestro django_tutorial ya modificado en anteriores prácticas y haremos que coja los parámetros de la base de datos por variables de entorno. Para ello editaremos lo siguiente en el settings.py:

```txt
import os

BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
```

Y en el apartado DATABASE ponemos lo siguiente:

```txt
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': os.environ.get("BASE_DATOS"),
        'USER': os.environ.get('USUARIO'),
        'PASSWORD': os.environ.get("CONTRA"),
        'HOST': os.environ.get('HOST'),
        'PORT': '3306',
    }
}
```

También modificaremos los siguientes apartados:

```txt
ALLOWED_HOSTS = [os.environ.get("ALLOWED_HOSTS")]
STATIC_ROOT = os.path.join(BASE_DIR, 'static')
STATIC_URL = '/static/'
CSRF_TRUSTED_ORIGINS = ['http://*.alejandro-montes.es','http://*.127.0.0.1','https://*.alejandro-montes.es','https://*.127.0.0.1']
```

Ahora creamos un dockerfile:

```txt
FROM python:3
WORKDIR /usr/src/app
MAINTAINER Alejandro Montes Delgado "aaleemd11@gmail.com"
RUN apt-get install git && pip install --root-user-action=ignore --upgrade pip && pip install --root-user-action=ignore django mysqlclient
RUN git clone https://github.com/alemd01/django_tutorial.git /usr/src/app && mkdir static 
ADD ./django_polls.sh /usr/src/app/
RUN chmod +x /usr/src/app/django_polls.sh
ENV ALLOWED_HOSTS=*
ENV HOST=mariadb
ENV USUARIO=django
ENV CONTRA=django
ENV BASE_DATOS=django
ENV DJANGO_SUPERUSER_PASSWORD=admin
ENV DJANGO_SUPERUSER_USERNAME=admin
ENV DJANGO_SUPERUSER_EMAIL=admin@example.org
CMD ["/usr/src/app/django_polls.sh"]
```

Este dockerfile hace referencia a un script, que es el siguiente:

```txt
#! /bin/sh

python3 manage.py makemigrations
python3 manage.py migrate
python3 manage.py createsuperuser --noinput
python3 manage.py collectstatic --no-input
python3 manage.py runserver 0.0.0.0:8006
```

Ahora contruimos la imagen:

```txt
usuario@debian:~/docker/django$ docker build -t alemd01/django_tutorial:v1 .
Sending build context to Docker daemon   15.3MB
Step 1/16 : FROM python:3
3: Pulling from library/python
1e4aec178e08: Already exists 
6c1024729fee: Already exists 
c3aa11fbc85a: Already exists 
aa54add66b3a: Already exists 
9e3a60c2bce7: Already exists 
3b2123ce9d0d: Already exists 
05df7720fcb8: Already exists 
972ab8743e38: Already exists 
c6510153b089: Already exists 
Digest: sha256:86b2fe5e3cf5b979f4b21849ad340762f4a15129583c8134c6c2dc4a880942e6
Status: Downloaded newer image for python:3
 ---> a41622906035
Step 2/16 : WORKDIR /usr/src/app
 ---> Running in 8ee7c5e5e76c
Removing intermediate container 8ee7c5e5e76c
 ---> 50b45c704853
Step 3/16 : MAINTAINER Alejandro Montes Delgado "aaleemd11@gmail.com"
```

Listamos las imágenes para verificar que se ha creado correctamente:

```txt
usuario@debian:~/docker/django$ docker image ls
REPOSITORY                TAG       IMAGE ID       CREATED          SIZE
alemd01/django_tutorial   v1        936a3553d4ee   12 seconds ago   1GB
python                    3         a41622906035   2 days ago       925MB

```


### Crea un docker-compose para desplegar los contenedores necesarios. Configura los volúmenes que creas necesarios para que la aplicación sea persistente.

Ahora con docker compose crearemos el escenario. El contenido del docker compose es el siguiente:

```txt
version: '3.7'
services:
  django-tutorial:
    container_name: django-tutorial
    image: alemd01/django_tutorial:v1
    restart: always
    environment:
      ALLOWED_HOSTS: "*"
      HOST: bd_mariadb_django
      USUARIO: django
      CONTRA: django
      BASE_DATOS: django
      DJANGO_SUPERUSER_PASSWORD: admin
      DJANGO_SUPERUSER_USERNAME: admin
      DJANGO_SUPERUSER_EMAIL: admin@admin.org
    ports:
      - 8084:8006
    depends_on:
      - db_django
  db_django:
    container_name: bd_mariadb_django
    image: mariadb:latest
    restart: always
    environment:
      MARIADB_ROOT_PASSWORD: root
      MARIADB_DATABASE: django
      MARIADB_USER: django
      MARIADB_PASSWORD: django
    volumes:
      - mariadb_data_django:/var/lib/mysql

volumes:
    mariadb_data_django:

```

Levantamos el escenario:

```txt
usuario@debian:~/docker/django$ docker-compose up -d
Creating network "django_default" with the default driver
Creating volume "django_mariadb_data_django" with default driver
Creating bd_mariadb_django ... done
Creating django-tutorial   ... done
```

Verificamos que se han creado correctamente los contenedores:

![IAW-DOCKER-2.1.png](/img/IAW-DOCKER-2.1.png)

Accedemos a nuestro django a través de internet:

![IAW-DOCKER-2.2.png](/img/IAW-DOCKER-2.2.png)

Nos logueamos en la zona de administración para verificar que funciona:

![IAW-DOCKER-2.3.png](/img/IAW-DOCKER-2.3.png)


### Una vez probada en el entorno de desarrollo, despliega la aplicación en tu VPS usando el docker-compose y configurando el nginx como proxy inveso para acceder por nombre a la aplicación.

Lo primero que haremos será subir la imagen a dockerhub:

```txt
usuario@debian:~/docker/django$ docker push alemd01/django_tutorial:v1
The push refers to repository [docker.io/alemd01/django_tutorial]
8881bf95f7f1: Pushed 
328379340940: Pushed 
8d465b69d17c: Pushed 
ecfce2c62c6a: Pushed 
0dd588580c52: Pushed 
a875c3488142: Mounted from library/python 
dd5a8632e1df: Mounted from library/python 
43281466799e: Mounted from library/python 
17efc79ac0fe: Mounted from library/python 
0b6859e9fff1: Mounted from library/python 
11829b3be9c0: Mounted from library/python 
dc8e1d8b53e9: Mounted from library/python 
9d49e0bc68a4: Mounted from library/python 
8e396a1aad50: Mounted from alemd01/bookmedik 
v1: digest: sha256:c7cc58e1486f4d147198321cc6b0d63a09390e1e1435d97f7906d3ac24c92f5e size: 3264
```

Ahora en nuestro proovedor de dns añadiremos una entrada cname para django:

![IAW-DOCKER-2.4.png](/img/IAW-DOCKER-2.4.png)

Una vez hecho esto, es momento de ir a nuestro vps y lo primero que haremos será generar un certificado para nuestro virtualhost:

```txt
root@nabil:/home/alemd# certbot certonly --standalone -d django.alejandro-montes.es
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator standalone, Installer None
Attempting to parse the version 1.32.2 renewal configuration file found at /etc/letsencrypt/renewal/mail.alejandro-montes.es.conf with version 1.12.0 of Certbot. This might not work.
Requesting a certificate for django.alejandro-montes.es
Performing the following challenges:
http-01 challenge for django.alejandro-montes.es
Waiting for verification...
Cleaning up challenges

IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/django.alejandro-montes.es/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/django.alejandro-montes.es/privkey.pem
   Your certificate will expire on 2023-05-28. To obtain a new or
   tweaked version of this certificate in the future, simply run
   certbot again. To non-interactively renew *all* of your
   certificates, run "certbot renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le

```

Ahora levantaremos el docker compose que es exactamente el mismo que usamos en desarrollo:

```txt
root@nabil:/home/alemd/docker/django# docker-compose up -d
Creating bd_mariadb_django ... done
Creating django-tutorial   ... done
```

Ahora crearemos un proxy inverso a través de un virtualhost de ngninx para poder acceder a la aplicación mediante un nombre:

```txt
server {
        listen 80;
        listen [::]:80;

        server_name django.alejandro-montes.es;

        return 301 https://$host$request_uri;
}

server {
        listen 443 ssl http2;
        listen [::]:443 ssl http2;

        ssl    on;
        ssl_certificate /etc/letsencrypt/live/django.alejandro-montes.es/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/django.alejandro-montes.es/privkey.pem;

        index index.html index.php index.htm index.nginx-debian.html;

        server_name django.alejandro-montes.es;

        location / {
                proxy_pass http://localhost:8084;
                include proxy_params;
        }
}
```

Lo activamos mediante un enlace simbólico:

```txt
root@nabil:/home/alemd/docker/django# ln -s /etc/nginx/sites-available/django /etc/nginx/sites-enabled/django
```

Reiniciamos nginx para que se apliquen los cambios:

```txt
root@nabil:/home/alemd/docker/django# systemctl restart nginx
```

Accedemos a través de internet a la url:

![IAW-DOCKER-2.5.png](/img/IAW-DOCKER-2.5.png)

Entramos en la zona de administración:

**No funciona.**

---
## **Documento realizado por:**

 ✒️ **Alejandro Montes Delgado** - *2º ASIR*