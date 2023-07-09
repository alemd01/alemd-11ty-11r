---
layout: post
title: "Ejercicios de manejo de módulos"
excerpt: Vamos a trabajar con ejercicios sobre paquetería.
date: 2022-11-03
updatedDate: 2022-11-03
tags:
  - post
  - ASO
---


### 1. Comprueba los módulos cargados en tu equipo.

Para ver los módulos cargados de mi equipo, usamos el siguiente comando:

```bash
$ lsmod
```
![Imagen no encontrada: /img/ASO-P4.1.png](/img/ASO-P4.1.png "Imagen no encontrada: /img/ASO-P4.1.png")
lsmod, lo que hace es leer el contenido del archivo /proc/modules.
### 2. Cuenta el número de módulos disponibles en el núcleo que estás usando.

```bash
$ ls -R /lib/modules/$(uname -r) | wc -l
```
![ASO-P4.2.png](/img/ASO-P4.2.png)
### 3. Conecta un lápiz USB y observa la salida de la instrucción sudo dmesg.

he ejecutado el siguiente comando:

```bash
$ sudo dmesg -w
```
lo que hace el parametro -w es que se actualiza en el momento el comando, y al añadir el Pen-Drive se han añadido estas líneas:

![ASO-P4.3.png](/img/ASO-P4-3.png)

### 4. Elimina el módulo correspondiente a algún dispotivo no esencial y comprueba qué ocurre. Vuelve a cargarlo.

Vamos a eliminar el modulo usb storage, pero al hacer un lsmod vemos que usb_storage depende de uas, entonces, tenemos que eliminar el modulo uas primero y despues el de usb-storage.
```bash
$ modprobe -r uas
$ modprobe -r usb_storage
```
![ASO-P4.4.png](/img/ASO-P4.4.png)

### 5. Selecciona un módulo que esté en uso en tu equipo y configura el arranque para que no se cargue automáticamente.

Editamos el siguiente fichero:
```bash
$ sudo nano /etc/modprobe.d/blacklist.conf
```

Añadimos lo siguiente:
```txt
blacklist [nombre_modulo]
```

Una vez hecho esto ejecutamos el siguiente comando para actualizar:
```bash
$ sudo update-initramfs -u
```
### 6. Carga el módulo loop, obtén información de qué es y para qué sirve. Lista el contenido de /sys/modules/loop/parameters y configura el equipo para que se puedan cargar como máximo 12 dispositvos loop la próxima vez que se arranque.

```bash
$ sudo modprobe loop
```
![ASO-P4.6.png](/img/ASO-P4.6.png)

![ASO-P4.7.png](/img/ASO-P4.7.png)

editamos el siguiente fichero:

```bash
$ sudo nano /etc/modules
```

añadimos lo siguiente:

```txt
options loop max_loop=12
```

Una vez hecho esto ejecutamos el siguiente comando para actualizar:
```bash
$ sudo update-initramfs -u
```


## **Documento realizado por:**

 ✒️ **Alejandro Montes Delgado** - *2º ASIR*
