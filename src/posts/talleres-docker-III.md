---
layout: post
title: Ejercicios con Docker III.
excerpt: Tercer taller de la unidad de Docker.
date: 2023-02-07
updatedDate: 2023-02-07
tags:
  - post
  - Docker
  - SRI
---
# Taller 3: Creación de imágenes Docker.

## Creación de una imagen a partir de un Dockerfile.

Lo primero que haremos será una cuenta en Docker Hub.

### Crea una página web estática (por ejemplo busca una plantilla HTML5). O simplemente crea un index.html.

Yo usaré una página web estática que creé el curso anterior. Para ello mediante scp pasaré la página estática a la máquina donde uso docker.

### Crea un fichero Dockerfile que permita crear una imagen con un servidor web sirviendo la página. Puedes usar una imagen base debian o ubuntu, o una imagen que tenga ya un servicio web.

```txt
usuario@debian:~/docker/taller3$ nano Dockerfile
```

Añadimos lo siguiente:

```txt
FROM debian:latest
MAINTAINER alemd01 "aaleemd11@gmail.com"
RUN apt update && apt install -y apache2 && apt clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
ADD miresa /var/www/html/
EXPOSE 80
CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
```

### Ejecuta el comando docker que crea la nueva imagen. La imagen se debe llamar /mi_servidor_web:v1.

```txt
usuario@debian:~/docker/taller3$ docker build -t alemd01/mi_servidor_web:v1 .
Sending build context to Docker daemon  3.989MB
Step 1/6 : FROM debian:latest
latest: Pulling from library/debian
699c8a97647f: Pull complete 
Digest: sha256:92277f7108c432febe41beffd367dbb6dac60b9fbfe517c77208e6457eafe22b
Status: Downloaded newer image for debian:latest
 ---> 20473158e8b3
Step 2/6 : MAINTAINER alemd01 "aaleemd11@gmail.com"
 ---> Running in 1bfd4864f4ba
Removing intermediate container 1bfd4864f4ba
 ---> ea9811fb5ce2
Step 3/6 : RUN apt update && apt install -y apache2 && apt clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
 ---> Running in 1f3515239860

```

Ejecuto el contenedor para probarlo.

![IAW-T6-T3.1.png](/img/IAW-T6-T3.1.png)


### Conéctate a Docker Hub y sube la imagen que acabas de crear.

Para conectarnos a Docker Hub, ejecutamos lo siguiente:

```txt
usuario@debian:~/docker/taller3$ docker login -u alemd01
Password: 
WARNING! Your password will be stored unencrypted in /home/usuario/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```

Como contraseña podemos usar un `access token`, lo podemos obtener yendo a los ajustes de nuestro perfil de docker hub y en el apartado de `Security` nos aparecerá para crear un nuevo token de acceso.

Ahora subiremos nuestra imagen:

```txt
usuario@debian:~/docker/taller3$ docker push alemd01/mi_servidor_web:v1
The push refers to repository [docker.io/alemd01/mi_servidor_web]
fb9bbbdf2bba: Pushed 
de157d4f9d2d: Pushed 
8fcfc59d80ac: Mounted from library/debian 
v1: digest: sha256:1f518a3a8db31e1cf556ddeeec920e35545cf5d117dbb58f0462bfb1834a5850 size: 952

```

Como podemos ver se ha subido correctamente:

![IAW-T6-T3.2.png](/img/IAW-T6-T3.2.png)

### Descarga la imagen en otro ordenador donde tengas docker instalado, y crea un contenedor a partir de ella. (Si no tienes otro ordenador con docker instalado, borra la imagen en tu ordenador y bájala de Docker Hub).

Como no tengo otra máquina con docker, borraré la imagen:

```txt
usuario@debian:~/docker/taller3$ docker rm -f mi_servidor_web 
mi_servidor_web
usuario@debian:~/docker/taller3$ docker rmi alemd01/mi_servidor_web:v1
Untagged: alemd01/mi_servidor_web:v1
Untagged: alemd01/mi_servidor_web@sha256:1f518a3a8db31e1cf556ddeeec920e35545cf5d117dbb58f0462bfb1834a5850
Deleted: sha256:b1421e4328a124a2d5d79cbcdaee041a6a5cc540281f3dee34a4a4a2bb9b8c5b
Deleted: sha256:f41cc79ea6eb3e88d31d8e6ea80c3a66847423f757f828801f91fc828105496f
Deleted: sha256:14fedc712e4013f7b8fbac9a90497f599f92ad737dd9c9dc348c3206b46c8356
Deleted: sha256:7a41ce465264e588eb8efe17edccd0f1589b95c8f609f604da9da717f3ee5923
Deleted: sha256:fc066f3915bee0e6cdece071401e09403d26848e875cf5beef9d1f6f8faf9a91
Deleted: sha256:a1cbbce8a51991a52ea414bf88a33fa04adf85d34b64cfcd9a93d6b08727a291
Deleted: sha256:ea9811fb5ce2d965671ba62b2eeebfdc3268a4f71f01a16ff5eb7ced271d02f2

```

Ahora vuelvo a descargar la imagen:

```txt
usuario@debian:~/docker/taller3$ docker pull alemd01/mi_servidor_web:v1
v1: Pulling from alemd01/mi_servidor_web
699c8a97647f: Already exists 
b70b6b1f6935: Pull complete 
6cb65f0b958b: Pull complete 
Digest: sha256:1f518a3a8db31e1cf556ddeeec920e35545cf5d117dbb58f0462bfb1834a5850
Status: Downloaded newer image for alemd01/mi_servidor_web:v1
docker.io/alemd01/mi_servidor_web:v1

```

Y creamos el contenedor:

```txt
usuario@debian:~/docker/taller3$ docker run -d -p 8080:80 --name mi_servidor_web alemd01/mi_servidor_web:v1
c372915cce45d36f14c433e5ef4f877bc1efc59b87e3bdc788aee2e79b5a71c9

```

### Vamos a hacer una modificación de la página web: haz una modificación al fichero index.html.

```txt
usuario@debian:~/docker/taller3$ nano miresa/index.html 
```

Esta es la línea que he editado:

```html
<h3 class="heading">Utrera en el siglo XX. Docker Taller 3.</h3>
```


### Vuelve a crear una nueva imagen, en esta caso pon ta etiqueta v2. Súbela a Docker Hub.

Construyo la nueva imagen:

```txt
usuario@debian:~/docker/taller3$ docker build -t alemd01/mi_servidor_web:v2 .
Sending build context to Docker daemon   3.99MB
Step 1/6 : FROM debian:latest
 ---> 20473158e8b3
Step 2/6 : MAINTAINER alemd01 "aaleemd11@gmail.com"
 ---> Running in 9c6ce0fe2cd4
Removing intermediate container 9c6ce0fe2cd4
 ---> c5a25059498d
Step 3/6 : RUN apt update && apt install -y apache2 && apt clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
 ---> Running in 5cc2d230d04e
```

La subo a Docker Hub:

```txt
usuario@debian:~/docker/taller3$ docker push alemd01/mi_servidor_web:v2
The push refers to repository [docker.io/alemd01/mi_servidor_web]
c93977927766: Pushed 
2e2c716f2c7f: Pushed 
8fcfc59d80ac: Layer already exists 
v2: digest: sha256:04c70a813837a0fbbc176a499732a7d3fa99dd36f6ed3d60bf56b5b9afbf0b1d size: 952

```

### Por último, baja la nueva imagen en el ordenador donde está corriendo el contenedor. Para hacer la implantación de la nueva versión debes borrar el contenedor y crear uno nuevo desde la nueva versión de la imagen.

Elimino el contenedor y la imagen:

```txt
usuario@debian:~/docker/taller3$ docker rm -f mi_servidor_web 
mi_servidor_web
usuario@debian:~/docker/taller3$ docker rmi alemd01/mi_servidor_web:v2
Untagged: alemd01/mi_servidor_web:v2
Untagged: alemd01/mi_servidor_web@sha256:04c70a813837a0fbbc176a499732a7d3fa99dd36f6ed3d60bf56b5b9afbf0b1d
Deleted: sha256:4c59dc832620ef19a0c7c8cfc2c8c5aab5b252ee5a95682d1dbb021c7644f8b1
Deleted: sha256:9009b939999f9230485edd5be57919a22a076493fb95eb53eea4dde3896f56db
Deleted: sha256:1b5da554f6e3376a9dd6cc76706945a46cc49ba8b1217c19b2f9a44938af005f
Deleted: sha256:d8e4f0fdb9ec151d0b3e8740cf63c34444e411956a3315aadd9e0075bb2b0c0f
Deleted: sha256:291028a34b7b79422480095d3ccef56d931a7d723e7c4c36af9cb0092ecaae4e
Deleted: sha256:329429045bff31241bf29767b5f37a38ec89d9430a12c8f874adc346b46534f9
Deleted: sha256:c5a25059498db80665ae7743cefdd116beee2b9e433aefdd12f1ecf167aedc01
```

Hacemos un pull de la imagen en Docker Hub:

```txt
usuario@debian:~/docker/taller3$ docker pull alemd01/mi_servidor_web:v2
v2: Pulling from alemd01/mi_servidor_web
699c8a97647f: Already exists 
e45857a00bba: Pull complete 
5ec4abd0fb7e: Pull complete 
Digest: sha256:04c70a813837a0fbbc176a499732a7d3fa99dd36f6ed3d60bf56b5b9afbf0b1d
Status: Downloaded newer image for alemd01/mi_servidor_web:v2
docker.io/alemd01/mi_servidor_web:v2

```

Creamos el contenedor a partir de la imagen:

```txt
usuario@debian:~/docker/taller3$ docker run -d -p 8080:80 --name mi_servidor_web alemd01/mi_servidor_web:v2
41549244e7a29291ae4eb1f9ef090ec385fbf4bcb52ee9e29ecbebbd02588f09

```

Accedo mediante mi navegador para comprobar que funciona correctamente:

![IAW-T6-T3.3.png](/img/IAW-T6-T3.3.png)



---
## **Documento realizado por:**

 ✒️ **Alejandro Montes Delgado** - *2º ASIR*