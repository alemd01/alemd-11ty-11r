---
layout: post
title: Recolección centralizada de logs de sistema, mediante journald.
excerpt: En esta práctica, implementaremos un sistema de recolección de log mediante journald en nuestro escenario de OpenStack.
date: 2023-01-04
updatedDate: 2023-01-04
tags:
  - post
  - ASO
---

Lo primero que tendremos que hacer es instalar el paquete `systemd-journal-remote` en todas las máquinas del escenario. Para ello ejecutaremos el siguiente comando

```txt
usuario@alfa:~$ sudo apt install systemd-journal-remote
[usuario@bravo ~]$ sudo dnf install systemd-journal-remote
ubuntu@charlie:~$ sudo apt install systemd-journal-remote
ubuntu@delta:~$ sudo apt install systemd-journal-remote
```

![ASO-P7.1.png](/img/ASO-P7.1.png)

### Configuración del Servidor Alfa.

Una vez hayamos instalado el paquete en todo el escenario, lo siguiente que realizaremos será configurar el servidor que recogerá los logs.

Ya que no utilizaremos https, desactivaremos esa opción en el fichero:

```txt
usuario@alfa:~$ sudo grep --color -E -- '--listen-http' /lib/systemd/system/systemd-journal-remote.service 
ExecStart=/lib/systemd/systemd-journal-remote --listen-https=-3 --output=/var/log/journal/remote/                                                    
usuario@alfa:~$ sudo sed -i 's/--listen-https=-3/--listen-http=-3/g' /lib/systemd/system/systemd-journal-remote.service                               
usuario@alfa:~$ sudo grep --color -E -- '--listen-http' /lib/systemd/system/systemd-journal-remote.service systemd-journal-remote-250-12.el9_1.86_64                              
ExecStart=/lib/systemd/systemd-journal-remote --listen-http=-3 --output=/var/log/journal/remote/         
```

Ahora habilitamos el servicio:

```txt
usuario@alfa:~$ sudo systemctl enable --now systemd-journal-remote.socket
Created symlink /etc/systemd/system/sockets.target.wants/systemd-journal-remote.socket → /lib/systemd/system/systemd-journal-remote.socket.
usuario@alfa:~$ sudo systemctl enable --now systemd-journal-remote.service
usuario@alfa:~$
---

Si no existe el siguiente directorio, lo creamos:

```txt
usuario@alfa:~$ sudo mkdir -p /var/log/journal/remote/
usuario@alfa:~$ sudo chown systemd-journal-remote:systemd-journal-remote /var/log/journal/remote
```

### Configuración de los clientes Bravo, Charlie y Delta.

Lo primero que haremos será crear un usuario llamado `systemd-journal-upload` con el home `/run/systemd` y en el grupo `systemd-journal-upload`:

Bravo

```txt
[usuario@bravo ~]$ sudo adduser --system --home-dir /run/systemd --no-create-home --user-group systemd-journal-upload
```

Charlie y Delta(son iguales)

```txt
ubuntu@charlie:~$ sudo adduser --system --home /run/systemd --no-create-home --disabled-login --group systemd-journal-upload                                  
[sudo] password for ubuntu:
Adding system user `systemd-journal-upload' (UID 109) ...
Adding new group `systemd-journal-upload' (GID 114) ...
Adding new user `systemd-journal-upload' (UID 109) with group `systemd-journal-upload' ...                                                                    
Not creating home directory `/run/systemd'.


ubuntu@delta:~$ sudo adduser --system --home /run/systemd --no-create-home --disabled-login --group systemd-journal-upload                          
[sudo] password for ubuntu:                                               
Adding system user `systemd-journal-upload' (UID 108) ...                 
Adding new group `systemd-journal-upload' (GID 113) ...                   
Adding new user `systemd-journal-upload' (UID 108) with group `systemd-journal-upload' ...                                                          
Not creating home directory `/run/systemd'.                               
```

Una vez creado los usuarios, tenemos que modificar la configuración del journal-upload para indicar que los clientes envíen los logs a alfa.

```txt
[usuario@bravo ~]$ sudo sed -i 's/# URL=/URL=http:\/\/alfa.alejandro-montes.gonzalonazareno.org:19532/g' /etc/systemd/journal-upload.conf
[usuario@bravo ~]$ sudo grep 'URL=' /etc/systemd/journal-upload.conf
URL=http://alfa.alejandro-montes.gonzalonazareno.org:19532
```

Repetiremos los mismos comandos en charlie y delta.

Una vez realizado esto, reiniciaremos el servicio en todos los clientes:

```txt
[usuario@bravo ~]$ sudo systemctl restart systemd-journal-upload.service
```

### Comprobaciones.

Si en el servidor le hacemos un ls a la ruta donde se guardan los logs de los clientes, se habrán creado 3 archivos distintos:

```txt
usuario@alfa:~$ ls /var/log/journal/remote/
remote-172.16.0.200.journal  remote-192.168.0.3.journal
remote-192.168.0.2.journal
usuario@alfa:~$ 
```

Si queremos comprobarlo con el comando journalctl ejecutamos lo siguiente indicando el archivo con la ip del cliente:

```txt
usuario@alfa:~$ sudo journalctl --file /var/log/journal/remote/remote-172.16.0.200.journal
```

Y nos aparecerá esto:

```txt
-- Journal begins at Mon 2022-11-28 21:25:01 UTC, ends at Sun 2023-01-08 23:04:44 UTC. --
Nov 28 21:25:01 localhost kernel: Linux version 5.14.0-162.6.1.el9_1.x86_64 (mockbuild@dal1-prod-builder001.bld.equ.rockylinux.org) (gcc (GCC) 11.3.1>
Nov 28 21:25:01 localhost kernel: The list of certified hardware and cloud instances for Red Hat Enterprise Linux 9 can be viewed at the Red Hat Ecos>
Nov 28 21:25:01 localhost kernel: Command line: BOOT_IMAGE=(hd0,gpt2)/vmlinuz-5.14.0-162.6.1.el9_1.x86_64 root=UUID=4814451b-2177-4679-bda1-e10797d12>
Nov 28 21:25:01 localhost kernel: x86/fpu: Supporting XSAVE feature 0x001: 'x87 floating point registers'
Nov 28 21:25:01 localhost kernel: x86/fpu: Supporting XSAVE feature 0x002: 'SSE registers'
Nov 28 21:25:01 localhost kernel: x86/fpu: Supporting XSAVE feature 0x004: 'AVX registers'
Nov 28 21:25:01 localhost kernel: x86/fpu: Supporting XSAVE feature 0x020: 'AVX-512 opmask'
Nov 28 21:25:01 localhost kernel: x86/fpu: Supporting XSAVE feature 0x040: 'AVX-512 Hi256'
Nov 28 21:25:01 localhost kernel: x86/fpu: Supporting XSAVE feature 0x080: 'AVX-512 ZMM_Hi256'
Nov 28 21:25:01 localhost kernel: x86/fpu: Supporting XSAVE feature 0x200: 'Protection Keys User registers'
Nov 28 21:25:01 localhost kernel: x86/fpu: xstate_offset[2]:  576, xstate_sizes[2]:  256
Nov 28 21:25:01 localhost kernel: x86/fpu: xstate_offset[5]:  832, xstate_sizes[5]:   64
Nov 28 21:25:01 localhost kernel: x86/fpu: xstate_offset[6]:  896, xstate_sizes[6]:  512
Nov 28 21:25:01 localhost kernel: x86/fpu: xstate_offset[7]: 1408, xstate_sizes[7]: 1024
Nov 28 21:25:01 localhost kernel: x86/fpu: xstate_offset[9]: 2432, xstate_sizes[9]:    8
Nov 28 21:25:01 localhost kernel: x86/fpu: Enabled xstate features 0x2e7, context size is 2440 bytes, using 'compacted' format.
Nov 28 21:25:01 localhost kernel: signal: max sigframe size: 3632
Nov 28 21:25:01 localhost kernel: BIOS-provided physical RAM map:
Nov 28 21:25:01 localhost kernel: BIOS-e820: [mem 0x0000000000000000-0x000000000009fbff] usable
Nov 28 21:25:01 localhost kernel: BIOS-e820: [mem 0x000000000009fc00-0x000000000009ffff] reserved
Nov 28 21:25:01 localhost kernel: BIOS-e820: [mem 0x00000000000f0000-0x00000000000fffff] reserved
Nov 28 21:25:01 localhost kernel: BIOS-e820: [mem 0x0000000000100000-0x000000003ffdcfff] usable
Nov 28 21:25:01 localhost kernel: BIOS-e820: [mem 0x000000003ffdd000-0x000000003fffffff] reserved
Nov 28 21:25:01 localhost kernel: BIOS-e820: [mem 0x00000000feffc000-0x00000000feffffff] reserved
Nov 28 21:25:01 localhost kernel: BIOS-e820: [mem 0x00000000fffc0000-0x00000000ffffffff] reserved
Nov 28 21:25:01 localhost kernel: NX (Execute Disable) protection: active
Nov 28 21:25:01 localhost kernel: SMBIOS 2.8 present.
Nov 28 21:25:01 localhost kernel: DMI: OpenStack Foundation OpenStack Nova, BIOS 1.15.0-1 04/01/2014
Nov 28 21:25:01 localhost kernel: Hypervisor detected: KVM
Nov 28 21:25:01 localhost kernel: kvm-clock: Using msrs 4b564d01 and 4b564d00
Nov 28 21:25:01 localhost kernel: kvm-clock: using sched offset of 13231497159309 cycles
Nov 28 21:25:01 localhost kernel: clocksource: kvm-clock: mask: 0xffffffffffffffff max_cycles: 0x1cd42e4dffb, max_idle_ns: 881590591483 ns
Nov 28 21:25:01 localhost kernel: tsc: Detected 2095.082 MHz processor
Nov 28 21:25:01 localhost kernel: e820: update [mem 0x00000000-0x00000fff] usable ==> reserved
Nov 28 21:25:01 localhost kernel: e820: remove [mem 0x000a0000-0x000fffff] usable

```


## **Documento realizado por:**

 ✒️ **Alejandro Montes Delgado** - *2º ASIR*
