---
layout: post
title: "Migrar CentOS a otra distribución."
excerpt: "En este articulo, veremos como realizar una migración de CentOS a otra distribución."
date: 2022-11-18
tags:
  - post
  - ASO
---

Debido al anuncio del fin de soporte por parte de Red Hat de Centos8 el pasado 31 de diciembre de 2021, y teniendo en cuentas que el fin de vida de centos 7 está programada para el 30 de junio de 2024. Han salido múltiples distribuciones que cubren el hueco dejado por esta distribución tan extendida y tan usada en el ámbito de servidores.

En la presente práctica, analiza posibles versiones candidatas y opciones desplegadas para la migración de tus servidores CentOS.

El espectro es amplio:

* Cambiar el rumbo a una nueva distribución, debian, opensuse, slakware, etc.

* Soluciones aportadas por Red Hat: Red Hat Enterprise Linux, CentOS Stream.

* Solución aportada por Oracle Linux.

Nuevas distribuciones surgidas para paliar el hueco dejado:

* AlmaLinux.
* Rocky Linux.
* VZLinux.
* euroLinux.

### Analiza el desencadenante de la retirada de centOS 8 del mercado. ¿Qué opinión tienes al respecto?

Al final, creo que la comunidad ha convertido algo que en un principio era malo, en algo muy positivo, ya que gracias a la descontinuación de CentOS han aparecido muchas distribuciones y buenas para usar.

CentOS 8 se ha convertido en el nuevo CentOS Stream, este cambio supone un fin a una gran trayectoria o legado, aunque a grandes rasgos nada o poco ha cambiado o vaya a cambiar.

### Crea una cuenta en Red Hat y descárgate la iso de Red Hat Enterprise Linux (REL) y evalúa el producto. Comenta el procedimiento de alta.

Lo primero que debemos de hacer es acceder a la página oficial de [Red Hat](https://www.redhat.com/es/technologies/linux-platforms/enterprise-linux/server/trial?sc_cid=7013a000002pdRFAAY&gclid=Cj0KCQiAg_KbBhDLARIsANx7wAz-LbYyFuxRlM2fU6aFLoXtvGOLRanBnxNm6uBHRNATDa0XvyGdkggaApVJEALw_wcB&gclsrc=aw.ds)
 para iniciar el registro. Para el registro debemos de poner nuestra información personal y en el apartado de la organización no dejaba ponerlo en blanco y he puesto Gonzalo Nazareno.

Una vez hecho el registro, se descargará automáticamente. Cuando se haya descargado la iso, Detallaré por encima la instalación de RHEL. Una vez descargada la iso la insertaremos en una máquina virtual. Al arrancar la máquina virtual, aparecerá el típico instalador de linux.

![aso5.3.png](/img/aso5.3.png)

Simplemente tenemos que elegir el destino de la instalación, añadir una contraseña de root y comenzará la instalación.

![aso5.4.png](/img/aso5.4.png)


#### Evaluación del producto.

Red Hat Enterprise Linux es la version comercial de Red Hat basada en Fedora que a su vez está basada en el anterior Red Hat linux. Mientras que las versiones de fedora salen cada 6 meses aproximadamente, las de RHEL suelen hacerlo cada 18 o 24 meses. Cada una de estas versiones cuenta con una serie de servicios de valor añadido sobre la base de los que basa su negocio ya sea como soporte, formación, consultoría, certificación etc.

Cada versión lanzada de RHEL cuenta por el momento con soporte durante al menos 10 años desde la fecha de lanzamiento de la GA (version que acaba en 0).

En cuanto a la interfaz gráfica, podemos ver una interfaz gnome, bastante familiar ya que es la interfaz gráfica más usada o de las más usada en linux, la podemos ver comumente en debian.

El gestor de paquetes de RHEL es yum y el administrador de paquetes RPM, no apt o dpkg como sería en el caso de Debian respectivamente. RHEL tiene muy pocos paquetes disponibles en su repositorio, si se quiere instalar algún paquete como el libreoffice por ejemplo, se debe de instalar desde repositorios de terceros o descargando directamente el paquete. Los paquetes de RHEL son RPM.

Como conclusión propia, me parece la distribución de linux idónea para instalar en una empresa grande y con un buen desarrollo tecnológico aunque no debemos de pasar por alto que es un sistema operativo caro ya que no te "venden" únicamente el sistema operativo si no que te dan soportes, asesorías, formación y demás. Otro aspecto a tener en cuenta es que al ser un sistema operativo pensado para servidores pueden ser demasiado estables y quizás necesitemos que nuestros paquetes se actualicen más a menudo.

### Descarga la iso de CentOS Stream y evalúa el producto.

Descargaremos la iso de la página oficial de [CentOS](https://www.centos.org/centos-stream/#tab-3). No pondré los pasos de la instalación ya que es exactamente igual que RHEL. Esperaremos a que se instale.

![aso5.5.png](/img/aso5.5.png)

#### Evaluación del producto.

CentOS ya no es una distribución estable como RHEL, se ha convertido en una distribución de lanzamiento contínuo entonces para entornos de producción como servidores de empresas es inviable, la distribución de CentOS a pasado mas a un entorno de desarrollo o también a nivel usuario.

Independientemente de que se actualice continuamente, el SO está basado en Red Hat por lo que es muy similar a RHEL.

### Descarga iso de una de las otras distribuciones candidatas, indica criterios para la elección de la nueva distribución y evalúa el producto.

La distribución que he elegido para migrar desde CentOS es Rocky Linux, ya que es una distribución muy estable y no recibe tantas actualizaciones como CentOS Stream por ejemplo, que estaría más enfocado a un entorno de desarrollo.

#### Evaluación del producto.

En cuanto a sistema operativo es muy similar al resto de sistemas operativos que hemos visto anteriormente. Rocky Linux es un punto intermedio entre CentOS Stream y RHEL es decir, no es un sistema para un entorno de desarrollo ni tampoco es tan estable como RHEL.

### Instala CentOS 7, y evalúa la herramientas que ofrecen la distribución del punto 3.

A continuación, realizaré una migración de CentOS 8 a Rocky Linux 8. Detallaré todos los pasos que he seguido para realizar de forma correcta la migración.


En nuestro servidor CentOS 8, como los repositorios de Centos 8 no funcionan, he pasado al servidor el script por correo.

Cambiamos los permisos del script con el siguiente comando:

```txt
[root@localhost usuario]# chmod u+x migrate2rocky.sh
```

Ejecutamos el script:

```txt
[root@localhost usuario]# ./migrate2rocky.sh -r
```

![aso5.6.png](/img/aso5.6.png)

Esperamos que el script se termine de ejecutar y se habrá actualizado correctamente.

![aso5.7.png](/img/aso5.7.png)

como podemos apreciar el sistema se ha migrado correctamente a rocky linux.

## **Documento realizado por:**

 ✒️ **Alejandro Montes Delgado** - *2º ASIR*
