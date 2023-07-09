---
layout: post
title: "Cifrado asimétrico con gpg y openssl."
excerpt: "En esta práctica vamos a cifrar ficheros utilizando cifrado asimétrico utilizando el programa gpg. Puedes encontrar el resumen de comando en esta chuleta o buscar información en internet."
date: 2022-12-02
tags:
  - post
  - SAD
---

### Tarea 1: Generación de claves

#### Genera un par de claves (pública y privada). ¿En que directorio se guarda las claves de un usuario?

Para generar un par de claves, usaremos el siguiente comando.

```txt
$ gpg --gen-key
```

![SAD-P3.1.png](/img/SAD-P3.1.png)

Las claves públicas y privadas se guardan en `~/.gnupg`

#### Lista las claves públicas que tienes en tu almacén de claves. Explica los distintos datos que nos muestra. ¿Cómo deberías haber generado las claves para indicar, por ejemplo, que tenga un 1 mes de validez?

El comando usado para listar las claves es:

```txt
$ gpg --list-keys
```

![SAD-P3.2.png](/img/SAD-P3.2.png)

Para indicar el tiempo de validez al crear el par de claves, usamos el comando:

```txt
$ gpg --full-gen-key
```

![SAD-P3.4.png](/img/SAD-P3.4.png)

![SAD-P3.5.png](/img/SAD-P3.5.png)


#### Lista las claves privadas de tu almacén de claves.

El comando usado para listar las claves privadas es:

```txt
$ gpg --list-secret-keys
```

![SAD-P3.3.png](/img/SAD-P3.3.png)


---

### Tarea 2: Importar / exportar clave pública

#### Exporta tu clave pública en formato ASCII y guardalo en un archivo nombre_apellido.asc y envíalo al compañero con el que vas a hacer esta práctica.

El comando para generar el archivo con la clave pública en formato ASCII es el siguiente:

```txt
$ gpg --armor --output alejandro_montes.asc --export aaleemd11@gmail.com
```
Le he enviado mi clave pública a mi compañero Juan Jesús.


#### Importa las claves públicas recibidas de vuestro compañero.

El comando para importar la clave pública de Juan Jesús es el siguiente:

![SAD-P3.6.png](/img/SAD-P3.6.png)


#### Comprueba que las claves se han incluido correctamente en vuestro keyring.

Muestro mi anillo de claves y se ve que la clave de Juan Jesús está importada correctamente:

![SAD-P3.8.png](/img/SAD-P3.8.png)


---

### Tarea 3: Cifrado asimétrico con claves públicas

#### Cifraremos un archivo cualquiera y lo remitiremos por email a uno de nuestros compañeros que nos proporcionó su clave pública.

Ciframos un archivo de ejemplo y se lo envío a Iván.

```txt
gpg --encrypt --recipient githubemail1asir@gmail.com para_juanjesus.txt
```

#### Nuestro compañero, a su vez, nos remitirá un archivo cifrado para que nosotros lo descifremos.

El comando usado para desifrar el archivo que nos ha enviado nuestro compañero es el siguiente:

```txt
gpg --decrypt criptoarchivo.txt.gpg
```

![SAD-P3.9.png](/img/SAD-P3.9.png)



#### Tanto nosotros como nuestro compañero comprobaremos que hemos podido descifrar los mensajes recibidos respectivamente.

Nuestro compañero también ha descifrado el archivo correctamente.

![SAD-P3.10.png](/img/SAD-P3.10.png)



#### Por último, enviaremos el documento cifrado a alguien que no estaba en la lista de destinatarios y comprobaremos que este usuario no podrá descifrar este archivo.

No tengo captura pero lo he probado y mi compañero no podía visualizar el contenido ni desencriptarlo.

#### Para terminar, indica los comandos necesarios para borrar las claves públicas y privadas que posees.

```txt
gpg --delete-secret-key "Alejandro Montes Delgado"
```
![SAD-P3.11.png](/img/SAD-P3.11.png)


---

### Tarea 4: Exportar clave a un servidor público de claves PGP

#### Genera la clave de revocación de tu clave pública para utilizarla en caso de que haya problemas.

Para crear una  clave de rovación usamos el siguiente comando:

```txt
gpg --gen-revoke 0549F57086A3E680F1DD7F7DB99701ED0FAD4CEF
```
![SAD-P3.12.png](/img/SAD-P3.12.png)

#### Exporta tu clave pública al servidor pgp.rediris.es

He usado otro servidor de claves públicas debido a que rediris está caido.

```txt
gpg --keyserver pgp.mit.edu --send-keys 0549F57086A3E680F1DD7F7DB99701ED0FAD4CEF
```

![SAD-P3.13.png](/img/SAD-P3.13.png)

Como podemos ver se ha subido correctamente.

![SAD-P3.14.png](/img/SAD-P3.14.png)


#### Borra la clave pública de alguno de tus compañeros de clase e impórtala ahora del servidor público de rediris.

Borro la clave pública de mi compañero:

```txt
gpg --delete-key "Juan Jesus"
```
Para importar la clave de Juan Jesús usamos el siguiente comando:

```txt
gpg --keyserver pgp.rediris.es --recv-keys 63DE4D1C7A7D69E8BB02F2F60D9A9FF1B5EE9093
```

Da error pero porque no funciona correctamente rediris.

![SAD-P3.15.png](/img/SAD-P3.15.png)


---

### Tarea 5: Cifrado asimétrico con openssl

#### Genera un par de claves (pública y privada).

Para generar el par de claves, uso el siguiente comando:

![SAD-P3.16.png](/img/SAD-P3.16.png)

Extraemos la clave pública y la insertamos en un fichero con el siguiente comando:

```txt
openssl rsa -in alejandro-montes-ssl.pem -pubout > alejandro-montes-ssl.pub.pem
```

![SAD-P3.17.png](/img/SAD-P3.17.png)


#### Envía tu clave pública a un compañero.

Le he enviado mi clave a Juanje y él me ha enviado la suya.

#### Utilizando la clave pública cifra un fichero de texto y envíalo a tu compañero.

Este es el fichero que yo he cifrado:

![SAD-P3.18.png](/img/SAD-P3.18.png)

Este es el fichero que ha encriptado Juanje

![SAD-P3.19.png](/img/SAD-P3.19.png)

#### Tu compañero te ha mandado un fichero cifrado, muestra el proceso para el descifrado.

Para descifrarlo usamos el siguiente comando:

```txt
openssl rsautl -decrypt -inkey alejandro-montes-ssl.pem -in criptoarchivossl.txt.enc -out criptoarchivossl.txt
```
![SAD-P3.20.png](/img/SAD-P3.20.png)

Mi compañero Juanje también ha descifrado el mío correctamente.

![SAD-P3.21.png](/img/SAD-P3.21.png)


---
## **Documento realizado por:**

 ✒️ **Alejandro Montes Delgado** - *2º ASIR*
