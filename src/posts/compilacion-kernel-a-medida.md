---
layout: post
title: Compilar un Kernel linux a medida.
excerpt: En este articulo, veremos como compilar un kernel linux a medida.
date: 2022-11-10
updatedDate: 2020-11-10
tags:
  - post
  - ASO
---

Antes que nada, crearemos un directorio de trabajo, donde descargaremos el kérnel y trabajaremos con él.

```bash
mkdir ~/kernel
cd ~/kernel
```

Instalamos las herramientas de compilación:

```bash
sudo apt install build-essential qtbase5-dev
```
Añadimos al `sources.list` el repositorio de bullseye-backports y con el siguiente comando podremos ver las versiones disponibles del kernel:

```txt
      ~/kernel$ sudo apt policy linux-source
linux-source:
  Instalados: (ninguno)
  Candidato:  5.10.149-2
  Tabla de versión:
     6.0.3-1~bpo11+1 100
        100 http://deb.debian.org/debian bullseye-backports/main amd64 Packages
     5.19.11-1~bpo11+1 100
        100 http://deb.debian.org/debian bullseye-backports/main amd64 Packages
     5.18.16-1~bpo11+1 100
        100 http://deb.debian.org/debian bullseye-backports/main amd64 Packages
     5.10.149-2 500
        500 http://security.debian.org/debian-security bullseye-security/main amd64 Packages
     5.10.140-1 500
        500 http://deb.debian.org/debian bullseye/main amd64 Packages
     5.10.136-1 500
        500 http://security.debian.org/debian-security bullseye-security/main amd64 Packages
     5.10.127-2 500
        500 http://deb.debian.org/debian bullseye/main amd64 Packages
        500 http://security.debian.org/debian-security bullseye-security/main amd64 Packages
     5.10.120-1 500
        500 http://security.debian.org/debian-security bullseye-security/main amd64 Packages
     5.10.113-1 500
        500 http://security.debian.org/debian-security bullseye-security/main amd64 Packages
     5.10.103-1 500
        500 http://security.debian.org/debian-security bullseye-security/main amd64 Packages
```

usaremos la versión del kernel 6.0.3.1. Para descargar el kernel con la versión que vamos a utilizar, usamos el siguiente comando:

```bash
sudo apt install linux-source=6.0.3-1~bpo11+1
```

movemos a nuestro directorio el archivo que se ha descargado. A mí se me ha descargado en */usr/src*:

```bash
sudo mv /usr/src/linux-source-6.0.tar.xz ./
```
Tenemos que desempaquetarlo, para ello usaremos tar:

```bash
tar -xf ./linux-source-6.0.tar.xz
```

Como podemos ver, ya tenemos listo el kernel para trabajar con él.

![ASO-P6.1.png](/img/ASO-P6.1.png)

A la hora de compilar el kernel, necesitamos el archivo .config el cual se van a cargar de forma estática los modulos del kernel. Cogeremos de referencia el `.config` de nuestro kernel. Ejecutaremos el siguiente comando para ello:

```bash
make oldconfig
```
Al ejecutar el make, nos pregunta si queremos incluir componentes adicionales, como la idea es hacer el kernel lo más pequeño posible, le daremos a todo que no. Como podemos ver, en el primer comando muestra los módulos que se han enlazado estáticamente(=y) y el segundo los que se han enlazado dinámicamente(=m).

![ASO-P6.2.png](/img/ASO-P6.2.png)

Como podemos apreciar son un total de unos 5700 módulos enlazados, para seguir quitando módulos, usaremos el siguiente comando para quitar aquellos que no están activos:

```bash
make localmodconfig
```

Cuando se ejecuta el comando, vuelve a preguntar si queremos enlazar algun componente, para seguir descartando componentes, le he seguido dando a no. Volvemos a contar cuantos módulos tenemos:


![ASO-P6.3.png](/img/ASO-P6.3.png)

Ahora, ya tenemos menos de 2000 módulos enlazados, sigue siendo una cantidad alta pero hemos reducido a más de la mitad los módulos enlazados.

Dicho esto, empezaremos nuestra primera compilación, así verificaremos que funciona todo.

```bash
alemd@debian:~/kernel/linux-source-6.0$ make -j4 bindeb-pkg
  SYNC    include/config/auto.conf.cmd
  UPD     include/config/kernel.release
sh ./scripts/package/mkdebian
dpkg-buildpackage -r"fakeroot -u" -a$(cat debian/arch)  -b -nc -uc
dpkg-buildpackage: información: paquete fuente linux-upstream
dpkg-buildpackage: información: versión de las fuentes 6.0.3-1
dpkg-buildpackage: información: distribución de las fuentes bullseye
dpkg-buildpackage: información: fuentes modificadas por alemd <alemd@debian>
dpkg-buildpackage: información: arquitectura del sistema amd64
 dpkg-source --before-build .
 debian/rules binary
make KERNELRELEASE=6.0.3 ARCH=x86 	KBUILD_BUILD_VERSION=1 -f ./Makefile
  SYSHDR  arch/x86/include/generated/uapi/asm/unistd_32.h
  SYSHDR  arch/x86/include/generated/uapi/asm/unistd_64.h
  WRAP    arch/x86/include/generated/uapi/asm/bpf_perf_event.h
  WRAP    arch/x86/include/generated/uapi/asm/errno.h
  HOSTCC  arch/x86/tools/relocs_32.o
  WRAP    arch/x86/include/generated/uapi/asm/fcntl.h
  WRAP    arch/x86/include/generated/uapi/asm/ioctl.h
  WRAP    arch/x86/include/generated/uapi/asm/ioctls.h
  WRAP    arch/x86/include/generated/uapi/asm/ipcbuf.h
  WRAP    arch/x86/include/generated/uapi/asm/param.h
  WRAP    arch/x86/include/generated/uapi/asm/poll.h
  WRAP    arch/x86/include/generated/uapi/asm/resource.h

```

Una vez compilado el kernel, instalaremos el paquete .deb que se ha generado con el siguiente comando:

```bash
sudo dpkg -i ../linux-image-6.0.3_6.0.3-1_amd64.deb
```

Ahora que está instalado el kernel, probaremos que funciona, para ello tendremos que reiniciar nuestra máquina y en *Advanced Options for Debian GNU/Linux* seleccionaremos el nuevo kernel. Probaremos que todo funciona correctamente y volveremos a reiniciar y entraremos con el kernel que usamos habitualmente. Como podemos comprobar después de reiniciar, estamos usando el kernel 6.0.3.

![ASO-P6.4.png](/img/ASO-P6.4.png)

Ahora, realizaremos una nueva compilación y antes que nada, eliminaremos los ficheros residuales de la compilación. Para ello utilizaremos el siguiente comando.

![ASO-P6.5.png](/img/ASO-P6.5.png)

Para esta compilación, no vamos a usar los parametros *oldconfig* ni *localmodconfig*, usaremos *xconfig* que es una aplicación gráfica para instalar los componentes a mano. Primero que nada instalaremos una dependencia:

```bash
sudo apt install pkg-config
```

Ahora procederemos a hacer el make xconfig, nos aparecerá una ventana como la de la siguiente imagen y simplemente tenemos que quitar los módulos que no vayamos a usar en el kernel desenmarcando las casillas. Es un proceso largo y lento pero es sencillo de hacer.

![ASO-P6.6.png](/img/ASO-P6.6.png)


Volvemos a contar cuantos módulos tiene el kernel:

![ASO-P6.7.png](/img/ASO-P6.7.png)



## **Documento realizado por:**

 ✒️ **Alejandro Montes Delgado** - *2º ASIR*
