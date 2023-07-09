---
layout: post
title: Poblar un directorio LDAP desde un fichero CSV.
excerpt: Crear un fichero CSV que incluya información personal de cada uno incluyendo los siguientes datos.
date: 2023-02-14
updatedDate: 2023-02-14
tags:
  - post
  - ASO
---

### Creación del fichero CSV.

Lo primero que tenemos que hacer es crear un fichero CSV con información de mis compañeros. El fichero CSV queda de la siguiente manera:

```txt
Alejandro,Montes Delgado,aaleemd11@gmail.com,alemd,ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCvd5Lg3K1k1WeHYHE3T5aPSEx/duCM87J2LFhgFODU3Z0+1mE5bjDUzvrUTVoHdMu1phOd8IS2vPCL2Ei8PjooLdLrsqHtFN9JYvtBRRhTs5tV3JSGaaFrljYtiITbulVTHBTqssrr4VfJ1jxMF9aSVcPjfVxsNMo01AhK8PAnCiSVBru9BUYgoG+7rCVMeB78qTXyuKNLoFqsuIPmQeV8D0JGKGozS2cikflyEOgYC+UwILUliV0dVtlpPltOOMLKngWF1fyjnfsJykiULI+l20Iid5QtDx5SK6nLRTtrr9MYCZbSJqAKVsxxKIYY5I+qSNsikgNrTnWJVq6z4zr5u4AqazjiDqX+266cDBf7rEq9GA9OZuul4/Kzy+PB9+w5rCgneSP7iL38ehQSjWzfpD1ny8mxCFmwuEwFBpGYhYHNywBYg8f1iMayjBeatFIU8WUCrCct0y1Zlf/Hn+Lzd5xb4aJgWd+mMyMmhZPcTAC6UN03tM2b0s6wjP8DzJc= ubuntu@delta
Angel,Suarez Perez,angelsuarezperez@gmail.com,angel,ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDFEJGJIWb1KviE3kwQRoUV2C4E0d9L0mq9XjSt1kmznXSXMODJBdHwa7GOb9iwmW3OKkTa6RGokB2nKvs+2IBm9HcPYM0WM78wxEazuUvwpHR4mV58gzif/ETM48/ceQcf1fqADW1456Mfk7zFHwgf8vjyNfFCtR6NGve7wb7ojm79a9CJpqZEJ9X+EVxjbKysdvhS3BsNo5L3aJwPsAHE29+lFhDmOOzuEXWTjvFRkC1aFN1iVyxdolNJuh3onWXZDzPI55Q/DHtkpjKZ4cYqBsmwZFjWlum5aWRltgYMZtvoGOBglIxBQWjY4sl7WCiW89b+6zIcG4vWq48qhoO76ROYEPJL4pdkxxUZbjk4xApHLoO56+fXCtoo+fO/zQiHyTElzXBg/v2MmqxUrFQmQCzhbAuKK88rycZVFdIknuc+Raoi3faiX2x5N9AL4PMpEVJaTi7b/n7NMZVTkuVT9U/X9pwABSi1yngxlr2BpTg2FJD5iId15v51nx1ynXE= angelsuarez@charlie
Juan Jesus,Alejo Sillero,juanjesusalejosillero@gmail.com,juanje,ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDzLgcT2qKvflOcKWjUGX0ecoVWN+phHx7dEws3b/rY/xAichGJ6oP8ucD4lVJ8XVrEOaUgqQ1laPK33+u9MmzARx4g17/jKcwepUWdkKVA2++RWG3bsNgxCCkR1Gi7XMAAjwq8/17OjCj+4bvfTPlW5FSjDqaLhfqqeDtKpFJ3wjGG5sjNPC0GU4cRKzggZaR40ld7siaOiMteQ8X6bIggeXw+ULGiUhB4/uoLu0z69AzGgDfoPJuJEx4pPlcnOip/TAuL/pUjTjdUUDTsrZSJegWoLmRwylKvwtX8WojqI2TnTOyLT0IG1oStq4gC4AKOiCqfiBOm25bFfX0lW0uUaR1RjEuGz3jV0vkH3pCiuarNk5KnEQQqUO0x6ZvdCvOlsWYoiDQ6MclGKfUkUzC1uST5khs4xB1zQAZ5795on4SV8STASTwjpxuTuk7v4lTxrm8bTAF4bWiezgOQ0aFr7P0APygX0rbCR1aXoGfSyrrvqOUtkUzpiWwZwOpj0K0= juanjesus@delta.juanjesus.gonzalonazareno.org
Ivan,Pina Castillo,ivanpicas88@gmail.com,ivan,ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCfYm123AV12rRYM+tPkrd0Hrzc9Py32Ov8JVZCnH5zVBj3I/IxE08LUhccSSx9aD0DrW+RdfpmLCSBTgGnbdM9eYlq3jxoBqqye4DQeXLSPyXcp/qRPGPsNO+eGypVhRB+Oq9B+ktrHgzAXQSP1yjmjN57H7GVBnMEJhpCEVXk5vWgMhVNxsDSF6lHrbiaYLtunTtt+fNgrprzXuUqhUwEDRt6/ktwad420J7kmqkB4dQuex3hV+16l1GyNH8AJzNzoinTiLr/jW8Ja0udgIknsxFvZ5Df+ACCrXfIFwvdPTm6Nya0jCm9vFx5yc5O1E07qlbAAn3FiIfS5Udjs6rNZjfFH5GmlpodhcGy4nkCYZvylnEayIa/ak4wA7oDft60hlHBMCHMoyY3ZcIkWGmVkwnTB3xfxfykPeD14zQAlIuMol9RNmPUbDYtbfY64npLPmagUIHSpwwbs1byEBbzzqzG8qcCAPUk3mK6oB5+OKUNJDv+4MM+suj+Y/PnWM8= debian@delta
```

Una vez tengamos el fichero, añadiremos el esquema openssh-lpk. Tendremos que crear el siguiente fichero:

```txt
root@alfa:/home/usuario# nano /etc/ldap/schema/openssh-lpk.ldif
```

El contenido es el siguiente:

```txt
dn: cn=openssh-lpk,cn=schema,cn=config
objectClass: olcSchemaConfig
cn: openssh-lpk
olcAttributeTypes: ( 1.3.6.1.4.1.24552.500.1.1.1.13 NAME 'sshPublicKey'
  DESC 'MANDATORY: OpenSSH Public key'
  EQUALITY octetStringMatch
  SYNTAX 1.3.6.1.4.1.1466.115.121.1.40 )
olcObjectClasses: ( 1.3.6.1.4.1.24552.500.1.1.2.0 NAME 'ldapPublicKey' SUP top AUXILIARY
  DESC 'MANDATORY: OpenSSH LPK objectclass'
  MAY ( sshPublicKey $ uid )
  )
```

Importamos el esquema con el siguiente comando:

```txt
root@alfa:/home/usuario# ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/ldap/schema/openssh-lpk.ldif
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
adding new entry "cn=openssh-lpk,cn=schema,cn=config"
```

### Creación de Script en python para poblar el directorio LDAP.

Para crear el script, tenemos que instalar python y los entornos virtuales:

```txt
usuario@alfa:~$ sudo apt install python3-venv
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  python-pip-whl python3.9-venv
The following NEW packages will be installed:
  python-pip-whl python3-venv python3.9-venv
0 upgraded, 3 newly installed, 0 to remove and 0 not upgraded.
Need to get 1954 kB of archives.
After this operation, 2327 kB of additional disk space will be used.
Do you want to continue? [Y/n] y
Get:1 https://deb.debian.org/debian bullseye/main amd64 python-pip-whl all 20.3.4-4+deb11u1 [1948 kB]
Get:2 https://deb.debian.org/debian bullseye/main amd64 python3.9-venv amd64 3.9.2-1 [5396 B]
Get:3 https://deb.debian.org/debian bullseye/main amd64 python3-venv amd64
```

Creamos un entorno virtual y lo activamos:

```txt
usuario@alfa:~$ python3 -m venv ldap
usuario@alfa:~$ source ldap/bin/activate
(ldap) usuario@alfa:~$ 
```

Instalamos con pip python3-ldap y ldap.

```txt
(ldap) usuario@alfa:~$ pip install python3-ldap
Collecting python3-ldap
  Downloading python3_ldap-0.9.8.4-py2.py3-none-any.whl (295 kB)
     |████████████████████████████████| 295 kB 1.8 MB/s 
Collecting pyasn1>=0.1.7
  Downloading pyasn1-0.4.8-py2.py3-none-any.whl (77 kB)
     |████████████████████████████████| 77 kB 1.2 MB/s 
Installing collected packages: pyasn1, python3-ldap
Successfully installed pyasn1-0.4.8 python3-ldap-0.9.8.4
(ldap) usuario@alfa:~$ pip install ldap3==2.6
Collecting ldap3==2.6
  Downloading ldap3-2.6-py2.py3-none-any.whl (393 kB)
     |████████████████████████████████| 393 kB 1.6 MB/s 
Requirement already satisfied: pyasn1>=0.1.8 in ./ldap/lib/python3.9/site-packages (from ldap3==2.6) (0.4.8)
Installing collected packages: ldap3
Successfully installed ldap3-2.6
```

Una vez tengamos instalado el entorno virtual y las dependencias, procederemos a crear el script en python:

```python
#!/usr/bin/env python

import ldap3
from ldap3 import Connection, ALL
from getpass import getpass
from sys import exit

### VARIABLES

# Shell que se le asigna a los usuarios
shell = '/bin/bash'

# Ruta absoluta del directorio que contiene los directorios personales de los usuarios. Terminado en "/"
home_dir = '/home/ldap/'

# El valor inicial para los UID que se asignan al insertar usuarios. 
uid_number = 5000

# El GID que se le asigna a los usuarios. Si no se manda al anadir el usuario da error.
gid = 5000

### VARIABLES

# Leemos el fichero .csv de los usuarios y guardamos cada linea en una lista.
with open('usuarios.csv', 'r') as usuarios:
  usuarios = usuarios.readlines()


### Parametros para la conexion
ldap_ip = 'ldap://alfa.alejandro-montes.gonzalonazareno.org:389'
dominio_base = 'dc=alejandro-montes,dc=gonzalonazareno,dc=org'
user_admin = 'admin' 
contrasena = getpass('Contrasena: ')

# Intenta realizar la conexion.
conn = Connection(ldap_ip, 'cn={},{}'.format(user_admin, dominio_base),contrasena)

# conn.bind() devuelve "True" si se ha establecido la conexion y "False" en caso contrario.

# Si no se establece la conexion imprime por pantalla un error de conexion.
if not conn.bind():
  print('No se ha podido conectar con ldap') 
  if conn.result['description'] == 'invalidCredentials':
    print('Credenciales no validas.')
  # Termina el script.
  exit(0)

# Recorre la lista de usuarios
for user in usuarios:
  # Separa los valores del usuario usando como delimitador ",", y asigna cada valor a la variable correspondiente.
  user = user.split(',')
  cn = user[0]
  sn = user[1]
  mail = user[2]
  uid = user[3]
  ssh = user[4]

  #Anade el usuario.
  conn.add(
    'uid={},ou=Personas,{}'.format(uid, dominio_base),
    object_class = 
      [
      'inetOrgPerson',
      'posixAccount', 
      'ldapPublicKey'
      ],
    attributes =
      {
      'cn': cn,
      'sn': sn,
      'mail': mail,
      'uid': uid,
      'uidNumber': str(uid_number),
      'gidNumber': str(gid),
      'homeDirectory': '{}{}'.format(home_dir,uid),
      'loginShell': shell,
      'sshPublicKey': str(ssh)
      })

  if conn.result['description'] == 'entryAlreadyExists':
    print('El usuario {} ya existe.'.format(uid))

  # Aumenta el contador para asignar un UID diferente a cada usuario (cada vez que ejecutemos el script debemos asegurarnos de ante mano que no existe dicho uid en el directorio ldap, o se solaparian los datos)
  uid_number += 1

#Cierra la conexion.
conn.unbind()
```

Ejecutamos el script:

```txt
(ldap) usuario@alfa:~$ nano usuarios.csv
(ldap) usuario@alfa:~$ python3 usuarios.py 
Contrasena:
(ldap) usuario@alfa:~$ 
```

Si hacemos un LDAP search el resultado es el siguiente:

```txt
(ldap) usuario@alfa:~$ ldapsearch -x -D "cn=admin,dc=alejandro-montes,dc=gonzalonazareno,dc=org" -b "dc=alejandro-montes,dc=gonzalonazareno,dc=org" -W

```

![LDAP-III.1.png](/img/LDAP-III.1.png)

### Configurar el sistema para que funcionen los usuarios de LDAP.

En el **cliente** de LDAP editamos la configuración de LDAP y ponemos lo siguiente(Si has seguido los pasos de la práctica anterior, este paso lo tendremos hecho):

```txt
BASE    dc=alejandro-montes,dc=gonzalonazareno,dc=org
URI     ldap://alfa.alejandro-montes.gonzalonazareno.org
```

### Configurar el servicio ssh para que permita acceder a los usuarios del LDAP utilizando las claves públicas que hay allí, en lugar de almacenarlas en .ssh/authorized_keys y que se cree el directorio “home” al vuelo.

Para configurar que el servicio ssh cree el home nada más crearse, tenemos que ejecutar el siguiente comando:

```txt
root@alfa:/home/usuario# echo "session    required        pam_mkhomedir.so" >> /etc/pam.d/common-session
```

Ahora realizaremos un script que encuentre las claves públicas de los usuarios. El script debe de estar en una ruta con los permisos correspondientes para que funcione:

```txt
root@alfa:/home/usuario# nano /opt/sshsearch.sh

#!/bin/bash

ldapsearch -x -u -LLL -o ldif-wrap=no '(&(objectClass=posixAccount)(uid='"$1"'))' 'sshPublicKey' | sed -n 's/^[ \t]*sshPublicKey::[ \t]*\(.*\)/\1/p' | base64 -d
root@alfa:/home/usuario# chmod 755 /opt/sshsearch.sh
```

Si ejecutamos el script vemos que funciona correctamente:

```txt
(ldap) usuario@alfa:~$ source /opt/sshsearch.sh alemd
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCvd5Lg3K1k1WeHYHE3T5aPSEx/duCM87J2LFhgFODU3Z0+1mE5bjDUzvrUTVoHdMu1phOd8IS2vPCL2Ei8PjooLdLrsqHtFN9JYvtBRRhTs5tV3JSGaaFrljYtiITbulVTHBTqssrr4VfJ1jxMF9aSVcPjfVxsNMo01AhK8PAnCiSVBru9BUYgoG+7rCVMeB78qTXyuKNLoFqsuIPmQeV8D0JGKGozS2cikflyEOgYC+UwILUliV0dVtlpPltOOMLKngWF1fyjnfsJykiULI+l20Iid5QtDx5SK6nLRTtrr9MYCZbSJqAKVsxxKIYY5I+qSNsikgNrTnWJVq6z4zr5u4AqazjiDqX+266cDBf7rEq9GA9OZuul4/Kzy+PB9+w5rCgneSP7iL38ehQSjWzfpD1ny8mxCFmwuEwFBpGYhYHNywBYg8f1iMayjBeatFIU8WUCrCct0y1Zlf/Hn+Lzd5xb4aJgWd+mMyMmhZPcTAC6UN03tM2b0s6wjP8DzJc= ubuntu@delta
(ldap) usuario@alfa:~$ source /opt/sshsearch.sh ivan
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCfYm123AV12rRYM+tPkrd0Hrzc9Py32Ov8JVZCnH5zVBj3I/IxE08LUhccSSx9aD0DrW+RdfpmLCSBTgGnbdM9eYlq3jxoBqqye4DQeXLSPyXcp/qRPGPsNO+eGypVhRB+Oq9B+ktrHgzAXQSP1yjmjN57H7GVBnMEJhpCEVXk5vWgMhVNxsDSF6lHrbiaYLtunTtt+fNgrprzXuUqhUwEDRt6/ktwad420J7kmqkB4dQuex3hV+16l1GyNH8AJzNzoinTiLr/jW8Ja0udgIknsxFvZ5Df+ACCrXfIFwvdPTm6Nya0jCm9vFx5yc5O1E07qlbAAn3FiIfS5Udjs6rNZjfFH5GmlpodhcGy4nkCYZvylnEayIa/ak4wA7oDft60hlHBMCHMoyY3ZcIkWGmVkwnTB3xfxfykPeD14zQAlIuMol9RNmPUbDYtbfY64npLPmagUIHSpwwbs1byEBbzzqzG8qcCAPUk3mK6oB5+OKUNJDv+4MM+suj+Y/PnWM8= debian@delta
```

Añadimos al fichero sshd_config lo siguiente:

```txt
root@alfa:/home/usuario# nano /etc/ssh/sshd_config

```

```txt
AuthorizedKeysCommand /opt/buscarclave.sh
AuthorizedKeysCommandUser nobody
```

reiniciamos ssh:

```txt
root@alfa:/home/usuario# systemctl restart sshd
```

### Comprobaciones.

Desde el cliente delta, accedo con el usuario `alemd` y creo una carpeta y un usuario:

```txt
ubuntu@delta:~$ ssh alemd@alfa.alejandro-montes.gonzalonazareno.org
The authenticity of host 'alfa.alejandro-montes.gonzalonazareno.org (192.168.0.1)' can't be established.
ECDSA key fingerprint is SHA256:Keks1avOP1EmyzMJEwSoAL+mcSzp7ag4TWkE5gjB4m8.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'alfa.alejandro-montes.gonzalonazareno.org,192.168.0.1' (ECDSA) to the list of known hosts.
Creating directory '/home/ldap/alemd'.
Linux alfa 5.10.0-21-amd64 #1 SMP Debian 5.10.162-1 (2023-01-21) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
alemd@alfa:~$ mkdir prueba
alemd@alfa:~$ touch prueba2
alemd@alfa:~$ ls -l
total 4
drwxr-xr-x 2 alemd 5000 4096 Feb 14 09:42 prueba
-rw-r--r-- 1 alemd 5000    0 Feb 14 09:42 prueba2
```

Desde alfa muestro que en la carpeta /home/ldap se ha creado el usuario alemd y se encuentran los archivos que he creado desde delta.

```txt
root@alfa:/home/ldap# ls
alemd
root@alfa:/home/ldap# cd alemd/
root@alfa:/home/ldap/alemd# ls
prueba	prueba2
root@alfa:/home/ldap/alemd# ls -l
total 4
drwxr-xr-x 2 alemd 5000 4096 Feb 14 09:42 prueba
-rw-r--r-- 1 alemd 5000    0 Feb 14 09:42 prueba2
```

Ahora ivan se conecta a mi servidor y crea un fichero y una carpeta:

![LDAP-III.2.png](/img/LDAP-III.2.png)

Desde el servidor muestro que se ha creado el Home y los ficheros que el ha creado.

```txt
usuario@alfa:/home/ldap$ ls
alemd  ivan
usuario@alfa:/home/ldap$ cd ivan/
usuario@alfa:/home/ldap/ivan$ ls -l
total 4
drwxr-xr-x 2 ivan 5000 4096 Feb 14 09:48 FUNCIONA
-rw-r--r-- 1 ivan 5000    0 Feb 14 09:49 soy-Ivan
```


---
## **Documento realizado por:**

 ✒️ **Alejandro Montes Delgado** - *2º ASIR*