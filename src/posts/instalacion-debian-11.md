---
layout: post
title: Instalación de Debian 11 en el equipo de trabajo
excerpt: Hablo sobre la instalación con lvm y los problemas que he tenido.
date: 2022-10-05
updatedDate: 2022-10-05
tags:
  - post
  - ASO
---
He instalado en mi equipo de trabajo debian 11 con lvm, a continuación, explico de forma detallada como lo he instalado, configurado y los problemas e inconvenientes que me han surgido a lo largo de la instalación. En mi equipo tengo 2 discos duros, un HDD de 1 Tb de capacidad y un SSD de 240 Gb, al poner lvm en mi equipo lo he estructurado de la siguiente manera:

El SSD esta particionado por la mitad, y una partición está reservada para windows la otra partición la he usado como volumen físico. En el caso del disco duro es exactamente igual está particionado en 2, 500gb cada partición, una de ellas está reservada para los archivos y la otra la he usado como otro volumen físico. He hecho 2 volúmenes de grupo, uno por disco, para que no interfiera en el rendimiento del ssd. En el ssd puse 2 volúmenes lógicos, uno la raíz con 30 Gb y el otro volumen lógico está el boot al que le di 1Gb.
En el HDD, tengo 2 particiones, una de ellas la he usado como volumen físico, he creado un grupo de volúmenes con este volumen físico y en el grupo de volúmenes he hecho 2 volúmenes lógicos, uno de 100Gb en el que he puesto la raiz y otro volumen lógico de 230Gb en el que he puesto el var.

Una vez instalado el S.O he tenido los siguientes problemas con los drivers: 
La tarjeta Wi-FI no me funcionaba, para solucionarlo hice el siguiente comando:

```diff-js
sudo apt install firmware-atheros
```

por último usé este comando para instalar firmware de linux.

```diff-js
sudo apt-get install firmware-linux firmware-linux-free firmware-linux-nonfree firmware-misc-nonfree
```
