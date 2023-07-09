---
layout: post
title: "Criptografía II: Integridad, firmas y autenticación"
excerpt: "En esta práctica realizaremos la 2ª práctica de criptografía."
date: 2022-12-08
tags:
  - post
  - SAD
---

### Tarea 1: Firmas electrónicas

#### 1. Manda un documento y la firma electrónica del mismo a un compañero. Verifica la firma que tu has recibido.

Para generar la firma de un documento, usamos el siguiente comando:

```txt
gpg --detach-sign prueba.txt
```

Le enviamos al compañero el documento con la firma y saldrá el error del siguiente apratado.

#### 2. ¿Qué significa el mensaje que aparece en el momento de verificar la firma?

```txt
 gpg: Firma correcta de "Pepe D <josedom24@gmail.com>" [desconocido]
 gpg: ATENCIÓN: ¡Esta clave no está certificada por una firma de confianza!
 gpg:          No hay indicios de que la firma pertenezca al propietario.
 Huellas dactilares de la clave primaria: E8DD 5DA9 3B88 F08A DA1D  26BF 5141 3DDB 0C99 55FC
```

Este mensaje sale porque no tenemos un anillo de confianza. El error, nos hace referencia a que nuestra clave no está en ningun anillo de confianza.

#### 3. Vamos a crear un anillo de confianza entre los miembros de nuestra clase, para ello.

**Tu clave pública debe estar en un servidor de claves.**

Tengo la clave subida al servidor de ubuntu.

**Escribe tu fingerprint en un papel y dárselo a tu compañero, para que puede descargarse tu clave pública.**

Mi fingerprint es:

```txt
A24B1921B28D5F7BA7320497E29C0E02DE5CF68C
```

**Te debes bajar al menos tres claves públicas de compañeros. Firma estas claves.**

Clave de Alfonso.

```txt
gpg --keyserver keyserver.ubuntu.com --recv-keys DA5E8263079216E8
gpg --sign-key DA5E8263079216E8
```

Clave de Paco.

```txt
gpg --keyserver gpg.mit.edu --recv-keys D5DF96E1BBD7DB81
gpg --sign-key 7C499EE49A10C836480FCC9CD5DF96E1BBD7DB81
```

Clave de Óscar.

```txt
gpg --keyserver keyserver.ubuntu.com --recv-keys A8544395EB1796857BC8C1309E4B2C4247B5AA59
gpg --sign-key A8544395EB1796857BC8C1309E4B2C4247B5AA59
```


**Tu te debes asegurar que tu clave pública es firmada por al menos tres compañeros de la clase.**

Mi clave pública ha sido firmada por Alfonso, Paco y Óscar.

**Una vez que firmes una clave se la tendrás que devolver a su dueño, para que otra persona se la firme.**

Exporto la clave de Alfonso y se la envío.

```txt
gpg --export -a DA5E8263079216E8 > clave_alfonso.asc
```

Exporto la clave de Paco y se la envío

```txt
gpg --export -a 7C499EE49A10C836480FCC9CD5DF96E1BBD7DB81 > paco.key
```

Exporto la clave de Óscar y se la envío

```txt
gpg --export -a A8544395EB1796857BC8C1309E4B2C4247B5AA59 > oscar.key
```

**Cuando tengas las tres firmas sube la clave al servidor de claves y rellena tus datos en la tabla Claves públicas PGP 2020-2021**

Subido.

**Asegurate que te vuelves a bajar las claves públicas de tus compañeros que tengan las tres firmas.**

Me he tenido que bajar manualmente las claves, ya que desde la terminal no se actualizaba.

#### 4. Muestra las firmas que tiene tu clave pública.

```txt
gpg --list-sig
```

![SAD-P4.1.png](/img/SAD-P4.1.png)

#### 5. Comprueba que ya puedes verificar sin “problemas” una firma recibida por una persona en la que confías.

Alfonso me ha enviado un documento con su firma. Muestro que se ha verificado correctamente.

![SAD-P4.2.png](/img/SAD-P4.2.png)

#### 6. Comprueba que puedes verificar con confianza una firma de una persona en las que no confías, pero sin embargo si confía otra persona en la que tu tienes confianza total.

---

### Tarea 2: Correo seguro con evolution/thunderbird

#### 1. Configura el cliente de correo evolution con tu cuenta de correo habitual

He iniciado sesión con la cuenta que uso normalmente.

![SAD-P4.3.png](/img/SAD-P4.3.png)


#### 2. Añade a la cuenta las opciones de seguridad para poder enviar correos firmados con tu clave privada o cifrar los mensajes para otros destinatarios

Tenemos que irnos a ajustes y configuración de la cuenta, y le damos a cifrado extremo a extremo. Allí importaremos nuestra clave privada y activaremos el cifrado para nuevos mensajes.

![SAD-P4.5.png](/img/SAD-P4.5.png)



#### 3. Envía y recibe varios mensajes con tus compañeros y comprueba el funcionamiento adecuado de GPG.

Aquí muestro como Alfonso me ha enviado un correo cifrado por mi clave pública y lo puedo visualizar correctamente.

![SAD-P4.4.png](/img/SAD-P4.4.png)

Como podemos ver en la siguiente captura le he enviado correctamente un correo cifrado a Paco.

![SAD-P4.7.png](/img/SAD-P4.7.jpg)


#### 4. Correo firmado para Raúl.

Enviado el correo a Raúl, mi correo es alemd01@outlook.com para que lo puedas verificar.

---

### Tarea 3: Integridad de ficheros

#### 1. Para validar el contenido de la imagen CD, solo asegúrese de usar la herramienta apropiada para sumas de verificación. Para cada versión publicada existen archivos de suma de comprobación con algoritmos fuertes (SHA256 y SHA512); debería usar las herramientas sha256sum o sha512sum para trabajar con ellos.

Una vez descargada la ISO de debian, abriremos el fichero [SHA256SUMS](https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/) que se encuentra en el mismo enlace y nos centraremos en el checksum para la imagen `debian-11.5.0-amd64-netinst.iso` ya que esa es la que hemos descargado.

Ejecutaremos el siguiente comando para ver checksum de la ISO:

```txt
alemd@debian:~/Descargas$ sha256sum debian-11.5.0-amd64-netinst.iso
e307d0e583b4a8f7e5b436f8413d4707dd4242b70aea61eb08591dc0378522f3  debian-11.5.0-amd64-netinst.iso
```

Y realizaremos un cat del archivo SHA512SUMS para ver si es el mismo checksum:

```txt
alemd@debian:~/Descargas$ cat SHA256SUMS | grep debian-11.5.0-amd64-netinst.iso
e307d0e583b4a8f7e5b436f8413d4707dd4242b70aea61eb08591dc0378522f3  debian-11.5.0-amd64-netinst.iso
```

Como podemos comprobar, el checksum es el mismo.

#### 2. Verifica que el contenido del hash que has utilizado no ha sido manipulado, usando la firma digital que encontrarás en el repositorio. Puedes encontrar una guía para realizarlo en este artículo: How to verify an authenticity of downloaded Debian ISO images

Primero descargaremos el fichero [SHA256SUMS.sign](https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/SHA256SUMS.sign).

Importaremos la clave pública de debian para poder verificar la firma.

```txt
alemd@debian:~/Descargas$ gpg --keyserver keyring.debian.org --recv DF9B9C49EAA9298432589D76DA87E80D6294BE9B
gpg: clave DA87E80D6294BE9B: clave pública "Debian CD signing key <debian-cd@lists.debian.org>" importada
gpg: Cantidad total procesada: 1
gpg:               importadas: 1
```

Una vez importada la clave pública de debian, procederemos a verificar la firma:

```txt
alemd@debian:~/Descargas$ gpg --verify SHA256SUMS.sign SHA256SUMS
gpg: Firmado el dom 11 sep 2022 01:00:07 CEST
gpg:                usando RSA clave DF9B9C49EAA9298432589D76DA87E80D6294BE9B
gpg: Firma correcta de "Debian CD signing key <debian-cd@lists.debian.org>" [desconocido]
gpg: ATENCIÓN: ¡Esta clave no está certificada por una firma de confianza!
gpg:          No hay indicios de que la firma pertenezca al propietario.
Huellas dactilares de la clave primaria: DF9B 9C49 EAA9 2984 3258  9D76 DA87 E80D 6294 BE9B
```

Indica que se ha verificado correctamente y nos advierte de que la clave no está certificada por una firma de confianza.

---

### Tarea 4: Integridad y autenticidad (apt secure)

#### 1. ¿Qué software utiliza apt secure para realizar la criptografía asimétrica?

El software que  usa la herramienta apt secure es GPG.

#### 2. ¿Para que sirve el comando apt-key? ¿Qué muestra el comando apt-key list?

Este comando, es una herramienta que permite gestionar la lista de claves que utiliza APT para autenticar paquetes. Algunas de las opciones mas usadas son las siguientes: add, del, export, exportall, list y finger.

El comando apt-key list muestra todas las claves de confianza para APT:

```txt
alemd@debian:~/Descargas$ apt-key list
Warning: apt-key is deprecated. Manage keyring files in trusted.gpg.d instead (see apt-key(8)).
/etc/apt/trusted.gpg
--------------------
pub   rsa4096 2021-10-27 [SC] [caduca: 2023-01-20]
      F9A2 1197 6ED6 62F0 0E59  361E 5E3C 45D7 B312 C643
uid        [desconocida] Spotify Public Repository Signing Key <tux@spotify.com>

/etc/apt/trusted.gpg.d/debian-archive-bullseye-automatic.gpg
------------------------------------------------------------
pub   rsa4096 2021-01-17 [SC] [caduca: 2029-01-15]
      1F89 983E 0081 FDE0 18F3  CC96 73A4 F27B 8DD4 7936
uid        [desconocida] Debian Archive Automatic Signing Key (11/bullseye) <ftpmaster@debian.org>
sub   rsa4096 2021-01-17 [S] [caduca: 2029-01-15]

/etc/apt/trusted.gpg.d/debian-archive-bullseye-security-automatic.gpg
---------------------------------------------------------------------
pub   rsa4096 2021-01-17 [SC] [caduca: 2029-01-15]
      AC53 0D52 0F2F 3269 F5E9  8313 A484 4904 4AAD 5C5D
uid        [desconocida] Debian Security Archive Automatic Signing Key (11/bullseye) <ftpmaster@debian.org>
sub   rsa4096 2021-01-17 [S] [caduca: 2029-01-15]

/etc/apt/trusted.gpg.d/debian-archive-bullseye-stable.gpg
---------------------------------------------------------
pub   rsa4096 2021-02-13 [SC] [caduca: 2029-02-11]
      A428 5295 FC7B 1A81 6000  62A9 605C 66F0 0D6C 9793
uid        [desconocida] Debian Stable Release Key (11/bullseye) <debian-release@lists.debian.org>

/etc/apt/trusted.gpg.d/debian-archive-buster-automatic.gpg
----------------------------------------------------------
pub   rsa4096 2019-04-14 [SC] [caduca: 2027-04-12]
      80D1 5823 B7FD 1561 F9F7  BCDD DC30 D7C2 3CBB ABEE
uid        [desconocida] Debian Archive Automatic Signing Key (10/buster) <ftpmaster@debian.org>
sub   rsa4096 2019-04-14 [S] [caduca: 2027-04-12]

/etc/apt/trusted.gpg.d/debian-archive-buster-security-automatic.gpg
-------------------------------------------------------------------
pub   rsa4096 2019-04-14 [SC] [caduca: 2027-04-12]
      5E61 B217 265D A980 7A23  C5FF 4DFA B270 CAA9 6DFA
uid        [desconocida] Debian Security Archive Automatic Signing Key (10/buster) <ftpmaster@debian.org>
sub   rsa4096 2019-04-14 [S] [caduca: 2027-04-12]

/etc/apt/trusted.gpg.d/debian-archive-buster-stable.gpg
-------------------------------------------------------
pub   rsa4096 2019-02-05 [SC] [caduca: 2027-02-03]
      6D33 866E DD8F FA41 C014  3AED DCC9 EFBF 77E1 1517
uid        [desconocida] Debian Stable Release Key (10/buster) <debian-release@lists.debian.org>

/etc/apt/trusted.gpg.d/debian-archive-stretch-automatic.gpg
-----------------------------------------------------------
pub   rsa4096 2017-05-22 [SC] [caduca: 2025-05-20]
      E1CF 20DD FFE4 B89E 8026  58F1 E0B1 1894 F66A EC98
uid        [desconocida] Debian Archive Automatic Signing Key (9/stretch) <ftpmaster@debian.org>
sub   rsa4096 2017-05-22 [S] [caduca: 2025-05-20]

/etc/apt/trusted.gpg.d/debian-archive-stretch-security-automatic.gpg
--------------------------------------------------------------------
pub   rsa4096 2017-05-22 [SC] [caduca: 2025-05-20]
      6ED6 F5CB 5FA6 FB2F 460A  E88E EDA0 D238 8AE2 2BA9
uid        [desconocida] Debian Security Archive Automatic Signing Key (9/stretch) <ftpmaster@debian.org>
sub   rsa4096 2017-05-22 [S] [caduca: 2025-05-20]

/etc/apt/trusted.gpg.d/debian-archive-stretch-stable.gpg
--------------------------------------------------------
pub   rsa4096 2017-05-20 [SC] [caduca: 2025-05-18]
      067E 3C45 6BAE 240A CEE8  8F6F EF0F 382A 1A7B 6500
uid        [desconocida] Debian Stable Release Key (9/stretch) <debian-release@lists.debian.org>

/etc/apt/trusted.gpg.d/microsoft.gpg
------------------------------------
pub   rsa2048 2015-10-28 [SC]
      BC52 8686 B50D 79E3 39D3  721C EB3E 94AD BE12 29CF
uid        [desconocida] Microsoft (Release signing) <gpgsecurity@microsoft.com>

/etc/apt/trusted.gpg.d/spotify-2021-10-27-5E3C45D7B312C643.gpg
--------------------------------------------------------------
pub   rsa4096 2021-10-27 [SC] [caduca: 2023-01-20]
      F9A2 1197 6ED6 62F0 0E59  361E 5E3C 45D7 B312 C643
uid        [desconocida] Spotify Public Repository Signing Key <tux@spotify.com>

/etc/apt/trusted.gpg.d/spotify-2022-11-14-7A3A762FAFD4A51F.gpg
--------------------------------------------------------------
pub   rsa4096 2022-11-14 [SC] [caduca: 2024-02-07]
      E274 09F5 1D1B 6633 7F2D  2F41 7A3A 762F AFD4 A51F
uid        [desconocida] Spotify Public Repository Signing Key <tux@spotify.com>

```

#### 3. En que fichero se guarda el anillo de claves que guarda la herramienta apt-key?

Se encuentra en `/etc/apt/trusted.gpg`.

#### 4. ¿Qué contiene el archivo Release de un repositorio de paquetes?. ¿Y el archivo Release.gpg?. Puedes ver estos archivos en el repositorio http://ftp.debian.org/debian/dists/Debian10.1/. Estos archivos se descargan cuando hacemos un apt update.

El archivo release contiene una lista de los archivos `Package`, junto con sus hash MD5, SHA1 y SHA256. Con esto, podemos verificar que los ficheros `Packages` no han sido manipulados, comparando el hash que se nos ha facilitado con el hash que nosotros calculamos osbre el fichero que hemos descargado.

El archivo release.gpg Contiene la firma con la que se verifica que no se ha manipulado el paquete.

#### 5. Explica el proceso por el cual el sistema nos asegura que los ficheros que estamos descargando son legítimos.

Primero se descargan los ficheros Release, Release.gpg y el fichero Packages correspondientes. Se comprueba que usando la firma Release.gpg más la clave pública correspondiente el fichero es legítimo, entonces apt ya confía en todos sus checksum. Apt compara el checksum del Package con el que checksum del fichero Release, si ambos coinciden, el fichero se habría descargado.

#### 6. Añade de forma correcta el repositorio de virtualbox añadiendo la clave pública de virtualbox como se indica en la documentación.

Primero descargamos las claves gpg:

```txt
sudo wget -O- https://www.virtualbox.org/download/oracle_vbox_2016.asc | gpg --dearmor | sudo tee /usr/share/keyrings/virtualbox.gpg
```

Añadimos el repositorio al sources.list:

```txt
echo deb [arch=amd64 signed-by=/usr/share/keyrings/virtualbox.gpg] http://download.virtualbox.org/virtualbox/debian bullseye contrib | sudo tee /etc/apt/sources.list.d/virtualbox.list
```

---

### Tarea 5: Autentificación: ejemplo SSH

#### 1. Explica los pasos que se producen entre el cliente y el servidor para que el protocolo cifre la información que se transmite? ¿Para qué se utiliza la criptografía simétrica? ¿Y la asimétrica?

Son 2 pasos los que se realizan antes de que se cifre una comunicación ssh: 

- Conexión:

El cliente establece una conexión TCP con l servidor, y este responde con las versiones del protocolo que soporta. El cliente tiene que usar unas de las versiones que soporta el servidor para continuar. El servidor comparte su clave pública con el cliente para que compruebe si es la máquina ala que quería conectarse. Ahora, ambas partes están preparadas para la negociación de una clave de sesión usando Diffie-Hellman.

- Negociación:

El cliente y el servidor, se ponen de acuerdo en un número primo alto, que servirá como seed value, también se ponen de acuerdo en un tipo de cifrado, generan un número primo por separado, que mantienen en secreto de la otra parte que lo usan como una clave privada de la negociación, intercambian sus claves públicas generadas(no son las mismas que usan para la autenticación). Cada parte usa su clave privada más la clave pública de la otra parte más el número primo compartido original para calcular una clave compartida llamada shared secret. Este shared secret se usa para encriptar toda la comunicación a partir de ahora.

La criptografía simétrica se usa para encriptar conexiones mediante una clave secreta.

La criptografía asimétrica se usa durante el intercambio de claves, se usa en la autenticación del cliente con el servidor.

#### 2. Explica los dos métodos principales de autentificación: por contraseña y utilizando un par de claves públicas y privadas.

- Por Contraseñas:

Es el método más simple, el servidor pide al cliente que introduzca la contraseña del usuario al que intenta conectar y esta contraseña se envía usando la encriptación que previamente se ha negociado para que se mantenga en secreto.

- Por par de claves:

Utiliza dos claves separadas para el cifrado y el descifrado. Estas dos claves se conocen como clave pública y privada, juntas forman el par de claves. La clave pública como su nombre indica se distribuye abiertamente y se comparte con el otro cliente. La clave privada debe de permanecer oculta, para que la conexión sea segura, ningún tercero puede conocerla.

El cliente envía el ID del par de claves con el que se quiere autenticar ante el servidor. El servidor comprueba en el fichero `authorized_keys` del usuario al que quiere conectar el cliente si ese ID coincide con algunas de las claves públicas que tiene en el fichero. Si se encuentra una clave pública que coincide, el servidor genera un número aleatorio y usa esa clave pública para desencriptarlo. El servidor envía este número encriptado al cliente. Si el cliente tiene la clave privada del par de claves, podrá desencriptar este mensaje y revelar el número original. El cliente combina el número desencriptado con la clave compartida de sesión que se estça usando para encriptar la comunicación y calcula el hash MD5 de este valor. El cliente envía al servidor este hash como respuesta. El servidor compara el hash que él mismo generó con el que el cliente le ha enviado. Si los valores son iguales, se prueba que el cliente tiene la clave privada correcta y el cliente se autentica correctamente.

#### 3. En el cliente para que sirve el contenido que se guarda en el fichero ~/.ssh/know_hosts?

En este fichero, se guardan todas las claves públicas a las que el cliente se conecta. Se usa para autenticar los servidores a los que se conecta.

#### 4. ¿Qué significa este mensaje que aparece la primera vez que nos conectamos a un servidor?

```txt
 $ ssh debian@172.22.200.74
 The authenticity of host '172.22.200.74 (172.22.200.74)' can't be established.
 ECDSA key fingerprint is SHA256:7ZoNZPCbQTnDso1meVSNoKszn38ZwUI4i6saebbfL4M.
 Are you sure you want to continue connecting (yes/no)? 
```

Este mensaje es mostrado al ser la primera vez que nos conectamos a ese servidor por ssh y sucede porque en el fichero `know_hosts` no tenemos almacenada la clave pública y no se ha podido verificar la autenticidad del mismo. Si estamos seguro de su autenticidad, le diremos que sí y se guardará su clave pública en el fichero `know_hosts`.

#### 5. En ocasiones cuando estamos trabajando en el cloud, y reutilizamos una ip flotante nos aparece este mensaje:

```txt
 $ ssh debian@172.22.200.74
 @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
 @    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
 @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
 IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
 Someone could be eavesdropping on you right now (man-in-the-middle attack)!
 It is also possible that a host key has just been changed.
 The fingerprint for the ECDSA key sent by the remote host is
 SHA256:W05RrybmcnJxD3fbwJOgSNNWATkVftsQl7EzfeKJgNc.
 Please contact your system administrator.
 Add correct host key in /home/jose/.ssh/known_hosts to get rid of this message.
 Offending ECDSA key in /home/jose/.ssh/known_hosts:103
   remove with:
   ssh-keygen -f "/home/jose/.ssh/known_hosts" -R "172.22.200.74"
 ECDSA host key for 172.22.200.74 has changed and you have requested strict checking.
```

El mensaje nos indica que la clave pública del servidor al cual nos queremos conectar ha cambiado con respecto a la que teníamos guardadada para esa dirección IP en nuestro fichero `know_hosts`.

Esta alerta sale porque así se realizan los ataques Man-In-The-Middle, pero si estamos seguros de que somos nosotros o una máquina que conocemos, borramos la línea correspondiente en nuestro `know_hosts` de la antigua conexión y volvemos a conectar o podemos ejecutar el siguiente comando:

```txt
   ssh-keygen -f "/home/jose/.ssh/known_hosts" -R "<<DIRECCION.IP.DEL.SERVIDOR>>"
```

#### 6. ¿Qué guardamos y para qué sirve el fichero en el servidor ~/.ssh/authorized_keys?

Es un fichero que existe para cada usuario del sistema y guarda las claves públicas de los clientes que entran a ese usuario mediante ssh.

---
## **Documento realizado por:**

 ✒️ **Alejandro Montes Delgado** - *2º ASIR*
