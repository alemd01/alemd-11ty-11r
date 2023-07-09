---
layout: post
title: LDAPs
excerpt: En este post, veremos como convertir el servidor LDAP en LDAPs.
date: 2023-02-16
updatedDate: 2023-02-16
tags:
  - post
  - ASO
---

Usaremos los certificados proporcionados a principio de curso mediante la página de gestiona del centro. para ello los llevaremos al servidor y una vez en el servidor los moveremos a sus rutas correspondientes:

```txt
usuario@alfa:/etc/ssl$ sudo mv alemd.crt /etc/ssl/certs
usuario@alfa:/etc/ssl$ sudo mv gonzalonazareno.crt /etc/ssl/certs
usuario@alfa:/etc/ssl$ sudo mv alemd.key /etc/ssl/private/
```

Nos tenemos que asegurar de que el dueño de los archivos es root y tiene los privilegios correctos. Ahora tendremos que crear unas acls para que el usuario openldap que es el que se conecta con ldap tenga permisos sobre los ficheros anteriores.

```txt
root@alfa:/etc/ssl# setfacl -m u:openldap:r-x /etc/ssl/private
root@alfa:/etc/ssl# setfacl -m u:openldap:r-x /etc/ssl/certs
root@alfa:/etc/ssl# setfacl -m u:openldap:r-x /etc/ssl/private/alemd.key
```

### Configuración del servidor(Alfa).

Ahora tenemos que incorporar los cambios con un registro ldif y aplicamos la configuración.

```txt
nano ldaps_config.ldif

dn: cn=config
changetype: modify
replace: olcTLSCACertificateFile
olcTLSCACertificateFile: /etc/ssl/certs/gonzalonazareno.crt
-
replace: olcTLSCertificateKeyFile
olcTLSCertificateKeyFile: /etc/ssl/private/alemd.key
-
replace: olcTLSCertificateFile
olcTLSCertificateFile: /etc/ssl/certs/alemd.crt
```

Para aplicar la configuración usamos el siguiente comando:

```txt
usuario@alfa:~$ sudo ldapmodify -Y EXTERNAL -H ldapi:/// -f ldaps_config.ldif 
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
modifying entry "cn=config"
```

Editamos el siguiente fichero para habilitar que ldap use el puerto 636. El fichero que tendremos que editar es `/etc/default/slapd`:

La siguiente línea editada queda así:


```txt
SLAPD_SERVICES="ldap:/// ldapi:/// ldaps:///"
```

Muestro que ya se encuentra escuchando por el puerto 636:

```txt
root@alfa:/home/usuario# netstat -tlnp | egrep slap
tcp        0      0 0.0.0.0:636             0.0.0.0:*               LISTEN      8551/slapd          
tcp        0      0 0.0.0.0:389             0.0.0.0:*               LISTEN      8551/slapd          
tcp6       0      0 :::636                  :::*                    LISTEN      8551/slapd          
tcp6       0      0 :::389                  :::*                    LISTEN      8551/slapd   
```

Copiamos el certificado a la siguiente ruta, que es la ruta donde se ponen los certificados instalados localmente:

```txt
root@alfa:/home/usuario# cp /etc/ssl/certs/gonzalonazareno.crt /usr/local/share/ca-certificates/
```

Ahora actualizamos la lista de los certificados para que ca-certificates realice el enlace simbólico:

```txt
root@alfa:/home/usuario# update-ca-certificates
Updating certificates in /etc/ssl/certs...
rehash: warning: skipping duplicate certificate in gonzalonazareno.crt
1 added, 0 removed; done.
Running hooks in /etc/ca-certificates/update.d...
done.
```

Estos 2 pasos anteriores, lo tenemos que repetir en todos los clientes que queramos que use LDAPs. 

Editamos la siguiente línea en el caso de que no tengamos un certificado válido(no es necesario):

```txt
root@alfa:/home/usuario# nano /etc/ldap/ldap.conf
```

Añadimos lo siguiente:

```txt
TLS_REQCERT     allow
```

Ahora podremos ejecutar peticiones al servidor ldap usando SSL/TLS. Realizo una consulta para mostrar que funciona:

```txt
usuario@alfa:~$ ldapsearch -x -b "dc=alejandro-montes,dc=gonzalonazareno,dc=org" -H ldaps://localhost:636
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
```

Para que el cliente use por defecto LDAPs para realizar las conexiones, vamos a modificar el fichero de configuración del cliente y añadir lo siguiente:

```txt
root@delta:/home/ubuntu# nano /etc/ldap/ldap.conf 
```

Modificamos esta línea y queda así:

```txt
URI     ldaps://alfa.alejandro-montes.gonzalonazareno.org
```

Hacemos una consulta al servidor LDAPs desde un cliente ubuntu para verificar que funciona:

![LDAPS.1.png](/img/LDAPS.1.png)

### Configuración de cliente Ubuntu(Delta).

Como hemos visto antes, en el clientes tenemos que pasarnos el certificado de la entidad certificadora y actualizar los certificados locales. Adjunto los comandos usados:

```txt
root@alfa:/home/usuario# scp /usr/local/share/ca-certificates/gonzalonazareno.crt ubuntu@192.168.0.3:
ubuntu@192.168.0.3's password: 
gonzalonazareno.crt                                                                                100% 3634     4.2MB/s   00:00  
root@delta:/home/ubuntu# chown root: gonzalonazareno.crt 
root@delta:/home/ubuntu# mv gonzalonazareno.crt /usr/local/share/ca-certificates/
root@delta:/home/ubuntu# update-ca-certificates 
Updating certificates in /etc/ssl/certs...
rehash: warning: skipping ca-certificates.crt,it does not contain exactly one certificate or CRL
rehash: warning: skipping duplicate certificate in gonzalonazareno.crt
1 added, 0 removed; done.
Running hooks in /etc/ca-certificates/update.d...
done.

```

En el archivo de configuración de LDAP tenemos que indicar en el URI que sea ldaps:

```txt
root@delta:/home/ubuntu# nano /etc/ldap/ldap.conf 

URI     ldaps://alfa.alejandro-montes.gonzalonazareno.org

```

Debemos de hacer lo mismo en el archivo `/etc/ldap.conf`. Una vez hecho esto, ya podemos realizar el login desde Delta:

```txt
root@delta:/home/ubuntu# login usuario1
Password: 
Welcome to Ubuntu 20.04.5 LTS (GNU/Linux 5.10.0-21-amd64 x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage
Last login: Tue Feb  7 08:33:57 UTC 2023 on pts/4
usuario1@delta:~$ ls
prueba	pruebadelta  rafa1.txt
```

Como vemos al hacer el ls siguen las carpetas y los archivos creados anteriormente.

### Configuración de cliente Rocky Linux(Bravo).

Antes que nada, tenemos que acordarnos de mover el certificado y actualizar los certificados locales. Dejo los comandos usados para actualizar los certificados en Rocky Linux:

```txt
usuario@alfa:~$ scp /usr/local/share/ca-certificates/gonzalonazareno.crt usuario@172.16.0.200:
gonzalonazareno.crt                                                                                          100% 3634     1.2MB/s   00:00 

[root@bravo usuario]# chown root: gonzalonazareno.crt
[root@bravo usuario]# mv gonzalonazareno.crt /usr/share/pki/ca-trust-source/anchors/
[root@bravo usuario]# update-ca-trust 
```


Si queremos configurar un cliente de LDAP en rocky linux, lo primero es instalar el paquete del cliente de ldap:

```txt
[root@bravo usuario]# dnf install openldap-clients
Failed to set locale, defaulting to C.UTF-8
Last metadata expiration check: 2:18:55 ago on Sat Feb 18 17:57:07 2023.
Package openldap-clients-2.6.2-3.el9.x86_64 is already installed.
Dependencies resolved.
Nothing to do.
Complete!
```

En mi caso ya está instalado. Ahora editaremos el archivo de configuración PAM. El archivo se encuentra en la siguiente ruta:

```txt
[root@bravo usuario]# nano /etc/pam.d/common-auth
```

Y contiene la siguiente línea:

```txt
auth sufficient pam_ldap.so
```

Ahora nos vamos al archivo de configuración de LDAP:

```txt
[root@bravo usuario]# nano /etc/openldap/ldap.conf
```

y añadimos lo siguiente:

```txt
BASE dc=alejandro-montes,dc=gonzalonazareno,dc=org
URI ldaps://alfa.alejandro-montes.gonzalonazareno.org
```

Por último, para permitir que el usuario LDAP acceda a su home, debemos de editar el siguiente archivo de configuración PAM:

```txt
[root@bravo usuario]# nano /etc/pam.d/common-session
```

Y añadimos lo siguiente:

```txt
session sufficient pam_mkhomedir.so umask=0022 skel=/etc/skel/
```

Realizo una consulta al servidor LDAP desde bravo para probar que funciona:

![LDAPS.2.png](/img/LDAPS.2.png)


Ahora instalaremos los paquetes necesariso para hacer el login con LDAP:

```txt
[root@bravo usuario]# dnf -y install openldap-clients sssd sssd-ldap oddjob-mkhomedir 
Failed to set locale, defaulting to C.UTF-8
Last metadata expiration check: 1:21:38 ago on Sat Feb 18 21:02:59 2023.
Package openldap-clients-2.6.2-3.el9.x86_64 is already installed.
Package sssd-2.7.3-4.el9_1.1.x86_64 is already installed.
Package sssd-ldap-2.7.3-4.el9_1.1.x86_64 is already installed.
Package oddjob-mkhomedir-0.34.7-6.el9.x86_64 is already installed.
Dependencies resolved.
======================================================================================================================================================
 Package                                  Architecture                   Version                                 Repository                      Size
======================================================================================================================================================
Upgrading:
 libipa_hbac                              x86_64                         2.7.3-4.el9_1.3                         baseos                          36 k
 libsss_certmap                           x86_64                         2.7.3-4.el9_1.3                         baseos                          78 k
 libsss_idmap                             x86_64                         2.7.3-4.el9_1.3                         baseos                          42 k
 libsss_nss_idmap                         x86_64                         2.7.3-4.el9_1.3                         baseos                          45 k
 libsss_sudo                              x86_64                         2.7.3-4.el9_1.3                         baseos                          35 k
 sssd                                     x86_64                         2.7.3-4.el9_1.3                         baseos                          27 k
 sssd-ad                                  x86_64                         2.7.3-4.el9_1.3                         baseos                         211 k
 sssd-client                              x86_64                         2.7.3-4.el9_1.3                         baseos                         150 k
 sssd-common                              x86_64                         2.7.3-4.el9_1.3                         baseos                         1.6 M
 sssd-common-pac                          x86_64                         2.7.3-4.el9_1.3                         baseos                          95 k
 sssd-ipa                                 x86_64                         2.7.3-4.el9_1.3                         baseos                         274 k
 sssd-kcm                                 x86_64                         2.7.3-4.el9_1.3                         baseos                         108 k
 sssd-krb5                                x86_64                         2.7.3-4.el9_1.3                         baseos                          73 k
 sssd-krb5-common                         x86_64                         2.7.3-4.el9_1.3                         baseos                          89 k
 sssd-ldap                                x86_64                         2.7.3-4.el9_1.3                         baseos                         156 k
 sssd-nfs-idmap                           x86_64                         2.7.3-4.el9_1.3                         baseos                          39 k
 sssd-proxy                               x86_64                         2.7.3-4.el9_1.3                         baseos                          70 k
```

Ejecutamos el siguiente comando para que mkhomedir cree el directorio home del usuario en el momento en el que inicia sesión:

```txt
[root@bravo usuario]# authselect select sssd with-mkhomedir --force
Backup stored at /var/lib/authselect/backups/2023-02-18-22-25-09.ddiDdO
Profile "sssd" was selected.
The following nsswitch maps are overwritten by the profile:
- passwd
- group
- netgroup
- automount
- services

Make sure that SSSD service is configured and enabled. See SSSD documentation for more information.
 
- with-mkhomedir is selected, make sure pam_oddjob_mkhomedir module
  is present and oddjobd service is enabled and active
  - systemctl enable --now oddjobd.service
```

Editamos el archivo de configuración de SSSD:

```txt
[root@bravo usuario]# nano /etc/sssd/sssd.conf
```

Y añadimos las siguientes líneas al archivo_

```txt
[domain/default]
id_provider = ldap
autofs_provider = ldap
auth_provider = ldap
chpass_provider = ldap
ldap_uri = ldaps://alfa.alejandro-montes.gonzalonazareno.org
ldap_search_base = dc=alejandro-montes,dc=gonzalonazareno,dc=org
ldap_id_use_start_tls = True
ldap_tls_cacertdir = /etc/openldap/certs
cache_credentials = True
ldap_tls_reqcert = allow

[sssd]
services = nss, pam, autofs
domains = default

[nss]
homedir_substring = /nfs/users
```

Copiamos el certificado en el siguiente directorio:

```txt
[root@bravo usuario]# cp /usr/share/pki/ca-trust-source/anchors/gonzalonazareno.crt /etc/openldap/certs/
```

Reiniciamos el servicio y lo habilitamos:

```txt
[root@bravo usuario]# chmod 600 /etc/sssd/sssd.conf 
[root@bravo usuario]# systemctl restart sssd
[root@bravo usuario]# systemctl enable sssd
```

Ahora creamos una unidad NFS donde se montará el home de todos los usuarios LDAP:

```txt
[root@bravo usuario]# mkdir /nfs
[root@bravo usuario]# mkdir /nfs/users
[root@bravo usuario]# nano /etc/systemd/system/nfs-users.mount
```

El contenido de la unidad es el siguiente:

```txt
[Unit]
Description=Unidad de montaje NFS
Requires=NetworkManager.service
After=NetworkManager.service
[Mount]
What=172.16.0.1:/nfs/users
Where=/nfs/users
Options=_netdev,auto
Type=nfs
[Install]
WantedBy=multi-user.target
```

Recargamos el demonio y activamos la unidad:

```txt
[root@bravo usuario]# systemctl daemon-reload
[root@bravo usuario]# systemctl start nfs-users.mount
```

Hago un df -h para comprobar que se ha montado correctamente:

```txt
[root@bravo usuario]# df -h
Filesystem                  Size  Used Avail Use% Mounted on
devtmpfs                    4.0M     0  4.0M   0% /dev
tmpfs                       383M     0  383M   0% /dev/shm
tmpfs                       153M   20M  134M  13% /run
/dev/vda5                    29G  1.8G   28G   7% /
/dev/vda2                   994M  345M  649M  35% /boot
/dev/vda1                   100M  7.0M   93M   7% /boot/efi
172.16.0.1:/media/Archivos  2.0G     0  1.9G   0% /home/usuario/archivos
tmpfs                        77M     0   77M   0% /run/user/1000
172.16.0.1:/nfs/users        30G  5.9G   23G  21% /nfs/users
[root@bravo usuario]# 
```

Ahora realizamos un login con el usuario1 y creamos una carpeta:

```txt
[root@bravo ~]# su - usuario1
Last failed login: Sat Feb 18 22:42:09 UTC 2023 on pts/0
There was 1 failed login attempt since the last successful login.
[usuario1@bravo ~]$ ls
prueba  pruebadelta  rafa1.txt
[usuario1@bravo ~]$ mkdir desde-bravo
```

La visualizo desde alfa:

```txt
[usuario@bravo ~]$ exit
logout
Connection to 172.16.0.200 closed.
usuario@alfa:~$ sudo su
root@alfa:/home/usuario# cd
root@alfa:~# cd /nfs/users/usuario1/
root@alfa:/nfs/users/usuario1# ls
desde-bravo  prueba  pruebadelta  rafa1.txt
root@alfa:/nfs/users/usuario1# 
```

Como vemos a creado correctamente la carpeta en el home nfs.

**EXTRA**

Vamos a configurar nuestro servidor LDAP para que podamos loguearnos sin ser root. Esto pasa debido a que el servidor NFS necesita ser root para acceder, para ello le indicaremos un nuevo parámetro:

```txt
root@alfa:~# nano /etc/exports 
```

Editamos la línea de la carpeta NFS que compartimos y añadimos el último parámetro:

```txt
/nfs/users *(rw,fsid=0,subtree_check,no_root_squash)
```

Reiniciamos el servidor nfs:

```txt
root@alfa:~# systemctl restart nfs-kernel-server
```

Pruebo a iniciar sesión desde bravo(RockyLinux):

```txt
[usuario@bravo ~]$ su - usuario1
Password: 
Last login: Sat Feb 18 23:15:01 UTC 2023 on pts/1
[usuario1@bravo ~]$ 
```

Inicio sesión desde delta(Ubuntu):

```txt
ubuntu@delta:~$ su - usuario1
Password: 
usuario1@delta:~$ ls
desde-bravo  prueba  pruebadelta  rafa1.txt
```

---
## **Documento realizado por:**

 ✒️ **Alejandro Montes Delgado** - *2º ASIR*