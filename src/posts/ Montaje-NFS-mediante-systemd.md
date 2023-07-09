---
layout: post
title: "Montaje Servidor y cliente NFS mediante Systemd."
excerpt: "En esta práctica configuraremos un servidor y un cliente NFS mediante systemd."
date: 2022-12-15
tags:
  - post
  - ASO
---

### Creación del volumen y particionado.

Lo primero que he hecho ha sido añadir un volumen de 2Gb a la máquina `Alfa` de mi escenario en OpenStack. Una vex añadido el volumen, crearemos una partición y le daremos formato ext4.

Para crear la partición he usado el siguiente comando:

```txt
root@alfa:/media/Archivos# fdisk /dev/vdb

Welcome to fdisk (util-linux 2.36.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0x04ba1fd8.

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1): 
First sector (2048-4194303, default 2048): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-4194303, default 4194303): 

Created a new partition 1 of type 'Linux' and of size 2 GiB.

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

Una vez creada la partición con el siguiente comando, le asignaremos el formato ext4.

```txt
root@alfa:/media/Archivos# mkfs.ext4 /dev/vdb1
mke2fs 1.46.2 (28-Feb-2021)
Discarding device blocks: done                            
Creating filesystem with 524032 4k blocks and 131072 inodes
Filesystem UUID: faa25fa0-cffb-4494-b586-bdbe74f845aa
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (8192 blocks): done
Writing superblocks and filesystem accounting information: done 
```

### Creación de la unidad en systemd.

Primero de todo crearemos el directorio donde se montará el volumen en mi caso es el siguiente:

```txt
root@alfa:/media/Archivos# mkdir /media/Archivos
```

Ahora crearemos la unidad de systemd para montar el volumen en el servidor Alfa.

Creamos el siguiente fichero:

```txt
root@alfa:/media/Archivos# nano /etc/systemd/system/media-Archivos.mount
```

El fichero tiene dicho ya que debe de llamarse igual que la ruta donde se monta el volumen cambiando las barras `/` por guiones `-` y es `.mount` porque es una unidad de montaje. Añadiremos lo siguiente:

```txt
[Unit]
Description=Disco montado para practica ASO
[Mount]
What=/dev/vdb1
Where=/media/Archivos
Type=ext4
Options=defaults
[Install]
WantedBy=multi-user.target
```
- Description como su nombre indica es una descripción del volumen montado.
- En What debemos indicar la ruta del volumen que queremos montar.
- En Where es donde lo hemos montado, es decir, la ruta donde se monta el volumen.
- La opción Type es el formato que tendra el volumen.
- En Options indicamos las opciones por defecto.

Una vez hecho esto, la unidad estaría creada. Ahora procederemos a activarlo.

### Activación de la unidad de montaje.

Para activar la unidad de montaje, tenemos que recargar el daemon de systemd antes para que detecto los nuevos cambios y la nueva unidad. Para ello usamos el siguiente comando:

```txt
root@alfa:/media/Archivos# systemctl daemon-reload
```

Ahora ejecutamos el siguiente comando para iniciar la unidad creada:

```txt
root@alfa:/media/Archivos# systemctl start media-Archivos.mount
```

Para habilitar que la unidad se inicie automáticamente al inicio del sistema usamos el siguiente comando:

```txt
root@alfa:/media/Archivos# systemctl enable media-Archivos.mount
Created symlink /etc/systemd/system/multi-user.target.wants/media-Archivos.mount → /etc/systemd/system/media-Archivos.mount.
```

### Configuración servidor NFS.

Lo primero que tenemos que hacer es instalar los paquetes necesarios. Son los siguientes paquetes:

- nfs-kernel-server
- nfs-common

Una vez instalados los paquetes, editaremos el fichero exports.

```txt
root@alfa:/media/Archivos# nano /etc/exports
```

Añadiremos lo siguiente:

```txt
/media/Archivos *(rw,no_root_squash)
```

Activamos el servicio:

```txt
root@alfa:/media/Archivos# service nfs-kernel-server start
```

Ahora comprobamos los directorios exportados:

```txt
root@alfa:/media/Archivos# showmount --exports localhost
Export list for localhost:
/media/Archivos *
```

### Configuración cliente NFS.

El cliente que configuraré será bravo. Lo primero que haremos será instalar el cliente nfs-utils: 

```txt
[usuario@bravo system]$ sudo dnf install nfs-utils
Failed to set locale, defaulting to C.UTF-8
Last metadata expiration check: 1:58:56 ago on Tue Dec 27 15:52:39 2022.
Package nfs-utils-1:2.5.4-15.el9.x86_64 is already installed.
Dependencies resolved.
Nothing to do.
Complete!
```

Ahora crearemos una unidad de montaje del volumen nfs:

```txt
[usuario@bravo system]$ sudo nano /etc/systemd/system/home-usuario-archivos.mount
```

Añadimos lo siguiente:

```txt
[Unit]
Description= cliente NFS
[Mount]
What=172.16.0.1:/media/Archivos
Where=/home/usuario/archivos
Type=nfs
Options=defaults
[Install]
WantedBy=multi-user.target
```

Reiniciamos el demonio de systemctl:

```txt
[usuario@bravo system]$ sudo systemctl daemon-reload
```

Arrancamos la unidad que hemos creado:

```txt
[usuario@bravo system]$ sudo systemctl start home-usuario-archivos.mount 
```

Como podemos ver se ha montado correctamente:

```txt
[usuario@bravo system]$ df -h
Filesystem                  Size  Used Avail Use% Mounted on
devtmpfs                    4.0M     0  4.0M   0% /dev
tmpfs                       383M     0  383M   0% /dev/shm
tmpfs                       153M   14M  140M   9% /run
/dev/vda5                    29G  1.3G   28G   5% /
/dev/vda2                   994M  267M  728M  27% /boot
/dev/vda1                   100M  7.0M   93M   7% /boot/efi
tmpfs                        77M     0   77M   0% /run/user/1000
172.16.0.1:/media/Archivos  2.0G     0  1.9G   0% /home/usuario/archivos
```

Ahora hacemos una prueba para ver que está montado correctamente. Desde el servidor crearemos una carpeta en el volumen:

```txt
usuario@alfa:/media/Archivos$ sudo mkdir prueba
```

Ahora en el cliente listaremos los archivos para ver que se ha creado el directorio.

```txt
[usuario@bravo ~]$ cd archivos/
[usuario@bravo archivos]$ ls
lost+found  prueba
```

Hacemos la prueba del revés para ver que se pueden crear carpetas y ficheros desde el cliente:

```txt
[usuario@bravo archivos]$ sudo mkdir prueba2
[usuario@bravo archivos]$ ls
lost+found  prueba  prueba2
```
Ahora listamos desde el servidor para ver que se ha creado correctamente:

```txt
usuario@alfa:/media/Archivos$ ls
lost+found  prueba  prueba2
```


---
## **Documento realizado por:**

 ✒️ **Alejandro Montes Delgado** - *2º ASIR*
