---
layout: post
title: Creación de un sistema automatizado de instalación.
excerpt: Creación de un sistema automatizado de instalación. Se deberá configurar el sistema para que se responda automáticamente a todos los item en la instalación. Las diferentes contraseñas deberán codificarse para que no aparezcan en texto plano. Se trabajará con un esquema lvm creando volúmenes lógicos /, home y var. 
date: 2022-10-05
updatedDate: 2022-10-05
tags:
  - post
  - ASO
---
*Instalación automatizada basada en medio de almacenamiento extraíble.*

Usaremos 7z para descomprimir la iso.

```diff-js
7z x /mnt/Archivos/ISOs/debian-10.6.0-amd64-xfce-CD-1.iso
```
![ASO-P1.8.png](/img/ASO-P1.8.png)


descomprimimos el initrd. Damos permisos a la carpeta que vamos a modificar, ponemos el preseed en el initrd, volvemos a comprimir y ponemos los permisos de antes
```diff-js
7z x initrd.gz
chmod +w -R install.amd/
echo preseed.cfg | cpio -H newc -o -A -F install.amd/initrd
gzip install.amd/initrd
chmod -w -R install.amd/
```

Regeneramos el md5sum.txt
```diff-js
chmod +w md5sum.txt
find -follow -type f ! -name md5sum.txt -print0 | xargs -0 md5sum > md5sum.txt
chmod -w md5sum.txt
```

Creamos una nueva iso booteable con el comando _genisoimage_
```diff-js
sudo genisoimage -r -J -b isolinux/isolinux.bin -c isolinux/boot.cat -no-emul-boot -boot-load-size 4 -boot-info-table -o ../debian-10.6.0-amd64-xfce-CD-1.iso 
```

Ya tenemos la iso lista, ahora haremos una instalación desatendida. Ponemos el disco con la iso e iniciamos la máquina. Le damos a _Advanced options_.
![imagen](/img/ASO-P1.9.png)

Ahora le damos a _Automated install_ y empezará la instalación
![imagen](/img/ASO-P1.10.png)

ponemos contraseña de root y del usuario(me da problemas en el preseed y he decidido ponerla antes de la instalación).
![imagen](/img/ASO-P1.11.png)

Ahora empieza la instalación desatendida.
![imagen](/img/ASO-P1.12.png)

Aquí muestro la salida del comando _df -h_
![imagen](/img/ASO-P1.14.png)



