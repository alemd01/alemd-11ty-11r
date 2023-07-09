---
layout: post
title: "Ejercicios gestión de paquetería"
excerpt: Vamos a trabajar con ejercicios sobre paquetería.
date: 2022-10-18
updatedDate: 2022-10-18
tags:
  - post
  - ASO
---


# Trabajo con apt, aptitude, dpkg
#### Prepara una máquina virtual con Debian bullseye, realizar las siguientes acciones:

### 1. Que acciones consigo al realizar apt update y apt upgrade. Explica detalladamente.

Al hacer un apt update, actualizas la lista de paquetes que hay disponible y sus versiones en los repositorios aunque no realiza ninguna instalación.
Después de hacer un apt update, hay que hacer un apt upgrade, permite instalar si no tiene instalado y actualiza versiones de los paquetes ya instalados.

### 2. Lista la relación de paquetes que pueden ser actualizados. ¿Qué información puedes sacar a tenor de lo mostrado en el listado?.

Para mostrar los paquetes que pueden ser instalados, se usa el siguiente comando:

```bash
$ sudo apt list --upgradable
```
Muestro captura del comando ejecutado en mi máquina.
![img](/img/ASO-P3.1.png)

En la línea el resaltado verde apunta al nombre del paquete. oldstable indica que no es de la versión stable de debian(esa máquina es debian buster). Los números siguientes indican la versión del paquete y lo siguiente a la arquitectura del paquete(amd64).

### 3. Indica la versión instalada, candidata así como la prioridad del paquete openssh-client.

Para ver la versión instalada y la candidata de un paquete usamos el siguiente comando: 
```bash
$ sudo apt policy openssh-client
```
En la siguiente captura veremos la ejecución del comando:
![img](/img/ASO-P3.2.png)

Como indica la imagen, la versión instalada es la 1.7.9, la candidata es la misma y la prioridad es 500. 

### 4. ¿Cómo puedes sacar información de un paquete oficial instalado o que no este instalado?

para ver información de un paquete oficial instalado, usariamos dpkg -s nombre_paquete.

Si no está instalado, se puede ver información de un paquete usando apt show nombre_paquete

### 5. Saca toda la información que puedas del paquete openssh-client que tienes actualmente instalado en tu máquina.

Para ver mucha información del paquete podemos usar _dpkg -s openssh-client_.

```txt
Package: openssh-client
Status: install ok installed
Priority: standard
Section: net
Installed-Size: 4298
Maintainer: Debian OpenSSH Maintainers <debian-ssh@lists.debian.org>
Architecture: amd64
Multi-Arch: foreign
Source: openssh
Version: 1:8.4p1-5+deb11u1
Replaces: openssh-sk-helper, ssh, ssh-krb5
Provides: rsh-client, ssh-client
Depends: adduser (>= 3.10), dpkg (>= 1.7.0), passwd, libc6 (>= 2.26), libedit2 (>= 2.11-20080614-0), libfido2-1 (>= 1.5.0), libgssapi-krb5-2 (>= 1.17), libselinux1 (>= 3.1~), libssl1.1 (>= 1.1.1), zlib1g (>= 1:1.1.4)
Recommends: xauth
Suggests: keychain, libpam-ssh, monkeysphere, ssh-askpass
Breaks: openssh-sk-helper
Conflicts: sftp
Conffiles:
 /etc/ssh/ssh_config 8a5bddc82befb71d8ef34cc903d3d077
Description: secure shell (SSH) client, for secure access to remote machines
 This is the portable version of OpenSSH, a free implementation of
 the Secure Shell protocol as specified by the IETF secsh working
 group.
 .
 Ssh (Secure Shell) is a program for logging into a remote machine
 and for executing commands on a remote machine.
 It provides secure encrypted communications between two untrusted
 hosts over an insecure network. X11 connections and arbitrary TCP/IP
 ports can also be forwarded over the secure channel.
 It can be used to provide applications with a secure communication
 channel.
 .
 This package provides the ssh, scp and sftp clients, the ssh-agent
 and ssh-add programs to make public key authentication more convenient,
 and the ssh-keygen, ssh-keyscan, ssh-copy-id and ssh-argv0 utilities.
 .
 In some countries it may be illegal to use any encryption at all
 without a special permit.
 .
 ssh replaces the insecure rsh, rcp and rlogin programs, which are
 obsolete for most purposes.
Homepage: http://www.openssh.com/

```
Como podemos ver, nos muestra una descripción, si está instalado, la prioridad, el directorio del fichero de configuración...

### 6. Saca toda la información que puedas del paquete openssh-client candidato a actualizar en tu máquina.



### 7. Lista todo el contenido referente al paquete openssh-client actual de tu máquina. Utiliza para ello tanto dpkg como apt.

Para listar el contenido del paquete con dpkg usamos el siguiente comando:

![imagen](/img/ASO-P3.3.png)

Para listar el contenido del paquete con apt usamos el siguiente comando:

![imagen](/img/ASO-P3.4.png)

### 8. Listar el contenido de un paquete sin la necesidad de instalarlo o descargarlo.

```bash
$ sudo apt-file list nombre_paquete
```

### 9. Simula la instalación del paquete openssh-client.

Para simular una instalación con apt, se usa el parámetro -s. A continuación, muestro la ejecución del comando:

![imagen](/img/ASO-P3.5.png)

### 10. ¿Qué comando te informa de los posible bugs que presente un determinado paquete?

El comando que informa sobre los posibles bugs de un paquete es el siguiente:

```bash
$ sudo apt-listbugs -s all nombre_paquete
```

### 11. Después de realizar un apt update && apt upgrade. Si quisieras actualizar únicamente los paquetes que tienen de cadena openssh. ¿Qué procedimiento seguirías?. Realiza esta acción, con las estructuras repetitivas que te ofrece bash, así como con el comando xargs.



### 12. ¿Cómo encontrarías qué paquetes dependen de un paquete específico.

```bash
$ sudo apt-cache depends nombre_paquete
```
Y para dependencias inversas:

```bash
$ sudo apt-cache rdepends nombre_paquete
```

### 13. ¿Cómo procederías para encontrar el paquete al que pertenece un determinado fichero?

```bash
$ apt-file search ruta
```

### 14. ¿Que procedimientos emplearías para liberar la caché en cuanto a descargas de paquetería?

Con `apt autoclean` elimina paquetes de la caché que tiene una versión mas moderna.

### 15. Realiza la instalación del paquete keyboard-configuration pasando previamente los valores de los parámetros de configuración como variables de entorno.

```bash
$ debconf keyboard-configuration
```

### 16. Reconfigura el paquete locales de tu equipo, añadiendo una localización que no exista previamente. Comprueba a modificar las variables de entorno correspondientes para que la sesión del usuario utilice otra localización.

```bash
$ export LANG=en_GB.utf8
$ export LANGUAGE=en_GB.utf8
$ export LC_ALL=en_GB.utf8
```

```bash
$ sudo dpkg-reconfigure locales -f --frontend Noninteractive
```

### 17. Interrumpe la configuración de un paquete y explica los pasos a dar para continuar la instalación.

Probariamos con dpkg --configure -a. Si no funciona, tendríamos que limpiar la caché, apt clean y apt autoclean. Haríamos un apt-get update --fix-missing y apt-get install -f para recuperar las dependencias dañadas.

### 18. Explica la instrucción que utilizarías para hacer una actualización completa de todos los paquetes de tu sistema de manera completamente no interactiva

```bash
$ sudo apt update && sudo apt upgrade -y
```

### 19. Bloquea la actualización de determinados paquetes.

```bash
$ apt mark hold paquete
```

# Trabajo con ficheros .deb

### 1. Descarga un paquete sin instalarlo, es decir, descarga el fichero .deb correspondiente. Indica diferentes formas de hacerlo.

```bash
$ wget url_paquete

$ sudo apt-get download nombrepaquete
```

### 2. ¿Cómo puedes ver el contenido, que no extraerlo, de lo que se instalará en el sistema de un paquete deb?

si es un .deb:

```bash
$ dpkg -c nombrepaquete
```
O con tar:
```bash
$ tar -tf nombre_paquete.tar.gz
```

### 3. Sobre el fichero .deb descargado, utiliza el comando ar. ar permite extraer el contenido de una paquete deb. Indica el procedimiento para visualizar con ar el contenido del paquete deb. Con el paquete que has descargado y utilizando el comando ar, descomprime el paquete. ¿Qué información dispones después de la extracción?. Indica la finalidad de lo extraído.

```bash
$ ar -tvx *.deb
```
Se extrae: 

* *debian-binary* que indica la versión del archivo.
* *control.tar.xz* que tiene scritps de preinstalación, postinstalación, preborrado, postborrado, configuración...
* data.tar.xz en este paquete se encuentra todos los archivos que serán extraídos con dpkg.

### 4. Indica el procedimiento para descomprimir lo extraído por ar del punto anterior. ¿Qué información contiene?

tar xvf nombrearchivo.

# Trabajo con repositorios

### 1. Añade a tu fichero sources.list los repositorios de bullseye-backports y sid.

Tenemos que editar el fichero sources.list:
```bash
$ nano /etc/apt/sources.list
```
Añadimos al sources.list lo siguiente:
```bash
deb https://deb.debian.org/debian bullseye-backports main
deb https://deb.debian.org/debian sid main
```

### 2. Configura el sistema APT para que los paquetes de debian bullseye tengan mayor prioridad y por tanto sean los que se instalen por defecto.

Para dar prioridad a los paquetes de debian bullseye, tenemos que crear un archivo llamado `preferences.pref` y debe contener lo siguiente:
```bash
$ sudo nano /etc/apt/preferences.d/preferences.p
```

```txt
Package: *
Pin: release a=stable
Pin-Priority: 1000
```
### 3. Configura el sistema APT para que los paquetes de bullseye-backports tengan mayor prioridad que los de unstable.

Tenemos que hacer lo mismo que antes pero cambiando lo siguiente:
```txt
Pin: release a=stable
```
Por esto:

```txt
Pin: release a=bullseye-backports
```
### 4. ¿Cómo añades la posibilidad de descargar paquetería de la arquitectura i386 en tu sistema. ¿Que comando has empleado?. Lista arquitecturas no nativas. ¿Cómo procederías para desechar la posibilidad de descargar paquetería de la arquitectura i386?

El siguiente comando es para añadir:
```bash
$ sudo dpkg --add-architecture i386
```
Este comando lista las arquitecturas soportadas por nuestro sistema:
```bash
$ dpkg --prin-architectures
```
este comando elimina las arquitecturas:
```bash
$ dpkg --remove-architecture i386
```

### 5. Si quisieras descargar un paquete, ¿cómo puedes saber todas las versiones disponible de dicho paquete?

```bash
$ sudo apt show --all-versions nombrepaquete
```

### 6. Indica el procedimiento para descargar un paquete del repositorio stable.

```bash
$ sudo apt install -t stable nombrepaquete
```

### 7. Indica el procedimiento para descargar un paquete del repositorio de buster-backports.

```bash
$ sudo apt install -t bullseye-backports nombrepaquete
```

### 8. Indica el procedimiento para descargar un paquete del repositorio de sid.

```bash
$ sudo apt install -t sid nombrepaquete
```

### 9. Indica el procedimiento para descargar un paquete de arquitectura i386.

```bash
$ sudo apt install nombrepaquete:i386
```

# Trabajo con directorios

### 1. Que cometidos tienen:

* **/var/lib/apt/lists/**: En este directorio contiene características sobre los repositorios.

* **/var/lib/dpkg/available**: Lista con los paquetes que hay disponibles.

* **/var/lib/dpkg/status**: En este directorio, hay información sobre los paquetes instalados en el sistema.

* **/var/cache/apt/archives/**: Aquí hay los archivos `.deb` que estan instalados.

<br><br><br><br>



## **Documento realizado por:**

 ✒️ **Alejandro Montes Delgado** - *2º ASIR*

