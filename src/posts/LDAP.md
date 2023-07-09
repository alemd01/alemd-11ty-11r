---
layout: post
title: Instalación y configuración inicial de OpenLDAP.
excerpt: Veremos la instalación y la configuración inicial de OpenLDAP.
date: 2023-01-19
updatedDate: 2023-01-19
tags:
  - post
  - ASO
---

### Instalación y configuración del servidor LDAP.


Lo primero que debemos de hacer es asegurarnos de que tenemos el FQND configurado correctamente. En nuestro caso, en el escenario de OpenStack está configurado correctamente ya que tenemos instalado un servidor DNS. Muestro el hostname de Alfa:

```txt
usuario@alfa:~$ hostname -f
alfa.alejandro-montes.gonzalonazareno.org
```

Una vez comprobado de que el FQDN está configurado correctamente, comenzaremos con la instalación:

```txt
usuario@alfa:~$ sudo apt install slapd
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  ldap-utils libltdl7 libodbc1
Suggested packages:
  libsasl2-modules-gssapi-mit | libsasl2-modules-gssapi-heimdal
  libmyodbc odbc-postgresql tdsodbc unixodbc-bin
The following NEW packages will be installed:
  ldap-utils libltdl7 libodbc1 slapd
0 upgraded, 4 newly installed, 0 to remove and 1 not upgraded.
Need to get 2270 kB of archives.
After this operation, 6400 kB of additional disk space will be used.
Do you want to continue? [Y/n] y

```

Nos pedirá una contraseña para el usuario administrador de LDAP. Ahora instalaremos el siguiente paquete:

```txt
usuario@alfa:~$ sudo apt install ldap-utils
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
ldap-utils is already the newest version (2.4.57+dfsg-3+deb11u1).
ldap-utils set to manually installed.
0 upgraded, 0 newly installed, 0 to remove and 1 not upgraded.
```

No se me ha instalado ldap-utils debido a que se me ha instalado como dependencia. Para verificar que está instalado, haremos una búsqueda con las credenciales de la instalación:

```txt
usuario@alfa:~$ ldapsearch -x -D "cn=admin,dc=alejandro-montes,dc=gonzalonazareno,dc=org" -b "dc=alejandro-montes,dc=gonzalonazareno,dc=org" -W
Enter LDAP Password:
# extended LDIF
#
# LDAPv3
# base <dc=alejandro-montes,dc=gonzalonazareno,dc=org> with scope subtree
# filter: (objectclass=*)
# requesting: ALL
#

# alejandro-montes.gonzalonazareno.org
dn: dc=alejandro-montes,dc=gonzalonazareno,dc=org
objectClass: top
objectClass: dcObject
objectClass: organization
o: alejandro-montes.gonzalonazareno.org
dc: alejandro-montes

# search result
search: 2
result: 0 Success

# numResponses: 2
# numEntries: 1
```

Ahora, vamos a crear dos objetos, para ello crearemos el siguiente fichero:

```txt
usuario@alfa:~$ nano unidades.ldif
```

Y ponemos lo siguiente:

```txt
dn: ou=Personas,dc=alejandro-montes,dc=gonzalonazareno,dc=org
objectClass: organizationalUnit
ou: Personas

dn: ou=Grupos,dc=alejandro-montes,dc=gonzalonazareno,dc=org
objectClass: organizationalUnit
ou: Grupos
```

Ejecutamos el siguiente comando para incluirlo en nuestro árbol de directorios:

```txt
usuario@alfa:~$ ldapadd -x -D "cn=admin,dc=alejandro-montes,dc=gonzalonazareno,dc=org" -f unidades.ldif -W
Enter LDAP Password:
adding new entry "ou=Personas,dc=alejandro-montes,dc=gonzalonazareno,dc=org"

adding new entry "ou=Grupos,dc=alejandro-montes,dc=gonzalonazareno,dc=org"
```

Ahora crearemos un fichero para crear un grupo:

```txt
dn: cn=prueba,ou=Grupos,dc=alejandro-montes,dc=gonzalonazareno,dc=org
objectClass: top
objectClass: posixGroup
gidNumber: 2001
cn: prueba
```

lo añadimos con el siguiente comando:

```txt
usuario@alfa:~$ ldapadd -x -D 'cn=admin,dc=alejandro-montes,dc=gonzalonazareno,dc=org' -W -f grupos.ldif
```

Con el siguiente comando generaremos una contraseña cifrada para el usuario:

```txt
root@alfa:/home/usuario# slappasswd
New password: 
Re-enter new password: 
{SSHA}L53zECZUj4SOYLkP8kELnX8wM7oN04FM

```

Ahora crearemos el siguiente fichero para crear un usuario:

```txt
usuario@alfa:~$ nano usuarios.ldif
```

Añadimos lo siguiente:

```txt
dn: uid=usuario1, ou=Grupos, dc=alejandro-montes, dc=gonzalonazareno, dc=org
objectClass: top
objectClass: posixAccount
objectClass: inetOrgPerson
objectClass: person
cn: usuario1
uid: usuario1
uidNumber: 2001
gidNumber: 2001
homeDirectory: /nfs/users/usuario1
loginShell: /bin/bash
userPassword: {SSHA}L53zECZUj4SOYLkP8kELnX8wM7oN04FM
sn: usuario1
mail: pruebausuario@prueba.com
givenName: usuario1

```

Ejecutamos el siguiente comando para añadir el usuario a LDAP:

```txt
usuario@alfa:~$ ldapadd -x -D "cn=admin,dc=alejandro-montes,dc=gonzalonazareno,dc=org" -W -f usuario.ldif
Enter LDAP Password: 
adding new entry "uid=usuario, ou=Grupos, dc=alejandro-montes, dc=gonzalonazareno, dc=org"

```

Si hacemos un ldapsearch, podemos ver que el usuario creado está en el grupo `Grupos`.

```txt
# usuario1, Grupos, alejandro-montes.gonzalonazareno.org
dn: uid=usuario1,ou=Grupos,dc=alejandro-montes,dc=gonzalonazareno,dc=org
objectClass: top
objectClass: posixAccount
objectClass: inetOrgPerson
objectClass: person
cn: usuario1
uid: usuario1
uidNumber: 2000
gidNumber: 2000
homeDirectory: /nfs/users/pruebausuario
loginShell: /bin/bash
userPassword:: dXN1YXJpbw==
sn: apellido
mail: pruebausuario@prueba.com
givenName: usuario1


```

### Configuración NFS en el servidor(Alfa).

Lo primero que haré será crear el directorio donde se guardarán el home de todos los usuarios.

```txt
root@alfa:/home/usuario# mkdir /nfs
root@alfa:/home/usuario# mkdir /nfs/users

```

Creamos el directorio para el home del usuario de prueba y le cambiamos de dueño al directorio.

```txt
root@alfa:/home/usuario# mkdir /nfs/users/prueba
root@alfa:/home/usuario# chown 2000:2000 /nfs/users/prueba/

```

Instalamos los paquetes necesarios para nfs, en mi caso ya están instalados:

```txt
root@alfa:/home/usuario# apt install nfs-kernel-server nfs-common
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
nfs-common is already the newest version (1:1.3.4-6).
nfs-kernel-server is already the newest version (1:1.3.4-6).
0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.

```

Añadimos la siguiente línea al fichero exports para indicar a nfs el directorio que vamos a compartir.

```txt
/nfs/users *(rw,fsid=0,subtree_check)
```

### Instalación de NSS, PAM y NSCD.

Ahora instalaremos los siguientes paquetes para que el servidor LDAP sea capaz de identificarse, resolver nombres de usuarios y grupos, y cachear la resolución de nombres. Para ellos instalaremos los siguientes paquetes:

```txt
root@alfa:/home/usuario# apt install libpam-ldapd nscd libnss-ldap
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  nslcd nslcd-utils
Suggested packages:
  kstart
The following NEW packages will be installed:
  libnss-ldap libpam-ldapd nscd nslcd nslcd-utils
0 upgraded, 5 newly installed, 0 to remove and 0 not upgraded.
Need to get 710 kB of archives.
After this operation, 1361 kB of additional disk space will be used.
Do you want to continue? [Y/n] y
```

Ahora tendremos que poner la dirección del servidor LDAP, en nuestro caso es el localhost.

![ASO-LDAP-1.png](/img/ASO-LDAP-1.png)

En este apartado, tenemos que indicar el dominio separado en vez de por un punto, ponemos al principio `dc=` y una coma, así tendremos que tener el dominio completo.

![ASO-LDAP-2.png](/img/ASO-LDAP-2.png)

Ahora nos aparece la configuración de nss, le damos siguiente ya que nos pide la dirección del servidor ldap.

![ASO-LDAP-3.png](/img/ASO-LDAP-3.png)

Volvemos a indicar el dominio pero esta vez será para la configuración de NSS:

![ASO-LDAP-4.png](/img/ASO-LDAP-4.png)

Indicamos la versión 3:

![ASO-LDAP-5.png](/img/ASO-LDAP-5.png)

Ahora tenemos que indicar la cuenta administrador, en mi caso es admin.

![ASO-LDAP-6.png](/img/ASO-LDAP-6.png)

Indicamos la contraseña del usuario y pulsamos siguiente, en la siguiente ventana pulsaremos `<Ok>` y se terminará de instalar. Para finalizar la configuración, editaremos el siguiente fichero para que el sistema use LDAP para la resolución de nombres.

```txt
root@alfa:/home/usuario# nano /etc/nsswitch.conf 

```

Editamos las 4 primeras líneas no comentadas y ponemos lo siguiente:

```txt
passwd:         files ldap
group:          files ldap
shadow:         files ldap
gshadow:        files ldap
```

Como prueba, muestro el uuid del usuario y nos loguearemos desde el propio servidor con el usuario creado.

```txt
root@alfa:/home/usuario# id usuario1
uid=2001(usuario1) gid=2001(prueba) groups=2001(prueba)
root@alfa:/home/usuario# login
alfa login: usuario1
Password: 
Linux alfa 5.10.0-20-amd64 #1 SMP Debian 5.10.158-2 (2022-12-13) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Thu Feb  2 16:45:31 UTC 2023 on pts/2
usuario1@alfa:~$ 

```

### Configuración y prueba de cliente LDAP en Ubuntu(Delta).

En el cliente(Delta), después de actualizarlo, lo primero que haremos será instalar los paquetes necesarios:

```txt
ubuntu@delta:~$ sudo apt install ldap-utils
[sudo] password for ubuntu: 
Reading package lists... Done
Building dependency tree       
Reading state information... Done
Suggested packages:
  libsasl2-modules-gssapi-mit | libsasl2-modules-gssapi-heimdal
The following NEW packages will be installed:
  ldap-utils
0 upgraded, 1 newly installed, 0 to remove and 11 not upgraded.
Need to get 121 kB of archives.
After this operation, 745 kB of additional disk space will be used.
Get:1 http://archive.ubuntu.com/ubuntu focal-updates/main amd64 ldap-utils amd64 2.4.49+dfsg-2ubuntu1.9 [121 kB]

```

Una vez se hayan instalados las dependencias, editaremos el fichero de configuración para el cliente:

```txt
ubuntu@delta:~$ sudo nano /etc/ldap/ldap.conf

```

Y añadimos lo siguiente:

```txt
BASE    dc=alejandro-montes,dc=gonzalonazareno,dc=org
URI     ldap://alfa.alejandro-montes.gonzalonazareno.org

```

Una vez hayamos terminado, podremos comprobar que funciona:

```txt
ubuntu@delta:~$ ldapsearch -x -b "dc=alejandro-montes,dc=gonzalonazareno,dc=org"
# extended LDIF
#
# LDAPv3
# base <dc=alejandro-montes,dc=gonzalonazareno,dc=org> with scope subtree
# filter: (objectclass=*)
# requesting: ALL
#

# alejandro-montes.gonzalonazareno.org
dn: dc=alejandro-montes,dc=gonzalonazareno,dc=org
objectClass: top
objectClass: dcObject
objectClass: organization
o: alejandro-montes.gonzalonazareno.org
dc: alejandro-montes

# Personas, alejandro-montes.gonzalonazareno.org
dn: ou=Personas,dc=alejandro-montes,dc=gonzalonazareno,dc=org
objectClass: organizationalUnit
ou: Personas

# Grupos, alejandro-montes.gonzalonazareno.org
dn: ou=Grupos,dc=alejandro-montes,dc=gonzalonazareno,dc=org
objectClass: organizationalUnit
ou: Grupos

# prueba, Grupos, alejandro-montes.gonzalonazareno.org
dn: cn=prueba,ou=Grupos,dc=alejandro-montes,dc=gonzalonazareno,dc=org
objectClass: top
objectClass: posixGroup
gidNumber: 2001
cn: prueba

# usuario1, Grupos, alejandro-montes.gonzalonazareno.org
dn: uid=usuario1,ou=Grupos,dc=alejandro-montes,dc=gonzalonazareno,dc=org
objectClass: top
objectClass: posixAccount
objectClass: inetOrgPerson
objectClass: person
cn: usuario1
uid: usuario1
uidNumber: 2001
gidNumber: 2001
homeDirectory: /nfs/users/usuario1
loginShell: /bin/bash
sn: usuario1
mail: pruebausuario@prueba.com
givenName: usuario1

# search result
search: 2
result: 0 Success

# numResponses: 6
# numEntries: 5
```

Ahora instalaremos los paquetes necesarios para poder conectarnos al servidor. Como son exactamente los mismos paquetes que en el servidor, no pondré la configuración ya que es igual solo que poniendo los datos del servidor en vez de local.

```txt
ubuntu@delta:~$ sudo apt install -y libnss-ldap libpam-ldapd nscd
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following additional packages will be installed:
  ldap-auth-client ldap-auth-config nslcd nslcd-utils
Suggested packages:
  kstart
The following NEW packages will be installed:
  ldap-auth-client ldap-auth-config libnss-ldap libpam-ldapd nscd nslcd
  nslcd-utils
0 upgraded, 7 newly installed, 0 to remove and 11 not upgraded.
Need to get 330 kB of archives.
After this operation, 1,285 kB of additional disk space will be used.
Get:1 http://archive.ubuntu.com/ubuntu focal/universe amd64 nslcd amd64 0.9.11-1 [156 kB]
Get:2 http://archive.ubuntu.com/ubuntu focal-updates/universe a
```

Editamos el fichero nsswitch.conf

```txt
ubuntu@delta:~$ sudo nano /etc/nsswitch.conf
```

Añadimos lo siguiente:

```txt
passwd:         files systemd ldap
group:          files systemd ldap
shadow:         files ldap
gshadow:        files ldap
```


Reiniciamos el servicio para que se apliquen los cambios:

```txt
ubuntu@delta:~$ sudo systemctl restart nscd

```

Aún así, todavía no podemos acceder al home del usuario1 ya que no está montado, para ello tenemos que instalar y configurar el cliente NFS.

```txt
ubuntu@delta:~$ sudo apt install -y nfs-common
[sudo] password for ubuntu: 
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following additional packages will be installed:
  keyutils libevent-2.1-7 libnfsidmap2 libtirpc-common libtirpc3 rpcbind
Suggested packages:
  open-iscsi watchdog
The following NEW packages will be installed:
  keyutils libevent-2.1-7 libnfsidmap2 libtirpc-common libtirpc3
  nfs-common rpcbind
0 upgraded, 7 newly installed, 0 to remove and 11 not upgraded.
Need to get 543 kB of archives.
After this operation, 1,924 kB of additional disk space will be used.
Get:1 http://archive.ubuntu.com/ubuntu focal-updates/main amd64 libtirpc-common all 1.2.5-1ubuntu0.1 [7,712 B]
Get:2 http://archive.ubuntu.com/ubuntu focal-updates/main amd64 libtirpc3 amd64 1.2.5-1ubuntu0.1 [77.9 kB]
Get:3 http://archive.ubuntu.com/ubuntu focal/main amd64 rpcbind amd64 1.2.5-8 [42.8 kB]
Get:4 http://archive.ubuntu.com/ubuntu focal-updates/main amd64 keyutils amd64 1.6-6ubuntu1.1 [44.8 kB]
Get:5 http://archive.ubuntu.com/ubuntu focal/main amd64 libevent-2.1-7 amd64 2.1.11-stable-1 [138 kB]

```

Activamos nfs:

```txt
ubuntu@delta:~$ sudo systemctl start nfs-client.target
ubuntu@delta:~$ sudo systemctl enable nfs-client.target

```

Creamos el directorio donde montaremos el servicio nfs:

```txt
root@delta:/home/ubuntu# mkdir /nfs/users
root@delta:/home/ubuntu# mkdir /nfs/users/usuario1
```

Cambiamos el dueño al usuario1:

```txt
root@delta:/home/ubuntu# chown 2001:2001 /nfs/users/usuario1
```

Ejecutamos lo siguiente para que se cargue automáticamente el módulo nfs:

```txt
root@delta:/home/ubuntu# echo NFS | tee -a /etc/modules
NFS
```

Creamos una nueva unidad de systemd:

```txt
root@delta:/home/ubuntu# nano /etc/systemd/system/nfs-users.mount
```

Añadimos lo siguiente:

```txt
[Unit]
Description=Home centralizado
Requires=network-online.target
After=network-online.target
[Mount]
What=192.168.0.1:/nfs/users
Where=/nfs/users
Options=_netdev,auto
Type=nfs
[Install]
WantedBy=multi-user.target
```

Recargamos el demonio de systemd para que se agregue la nueva unidad y la encendemos:

```txt
root@delta:/home/ubuntu# systemctl daemon-reload
root@delta:/home/ubuntu# systemctl start nfs-users.mount
root@delta:/home/ubuntu# systemctl enable nfs-users.mount
Created symlink /etc/systemd/system/multi-user.target.wants/nfs-users.mount → /etc/systemd/system/nfs-users.mount.

```

### Comprobaciones

Desde el servidor conectado al usuario1, crearemos una carpeta de prueba:

```txt
root@alfa:/nfs/users# login usuario1
Password: 
Linux alfa 5.10.0-20-amd64 #1 SMP Debian 5.10.158-2 (2022-12-13) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Thu Feb  2 19:15:26 UTC 2023 on pts/3
usuario1@alfa:~$ mkdir prueba
usuario1@alfa:~$ ls
prueba
```

Desde el cliente ubuntu nos conectaremos al usuario1 y haremos un ls y crearemos otra carpeta:

```txt
root@delta:/nfs/users# login usuario1
Password: 
Welcome to Ubuntu 20.04.5 LTS (GNU/Linux 5.10.0-20-amd64 x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

Last login: Thu Feb  2 19:14:55 UTC 2023 on pts/4
usuario1@delta:~$ ls
prueba
usuario1@delta:~$ mkdir pruebadelta
usuario1@delta:~$ ls
prueba	pruebadelta
```



---
## **Documento realizado por:**

 ✒️ **Alejandro Montes Delgado** - *2º ASIR*