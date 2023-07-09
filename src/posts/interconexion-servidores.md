---
layout: post
title: "Interconexión de Servidores de Bases de datos"
excerpt: En este articulo, veremos como interconectar servidores de base de datos.
date: 2022-11-16
updatedDate: 2022-11-16
tags:
  - post
  - GBD
---

### Realizar un enlace entre dos servidores de bases de datos ORACLE, explicando la configuración necesaria en ambos extremos y demostrando su funcionamiento.

En el escenario, tenemos 2 servidores Oracle, uno creado en mi propio sistema debian y otro en una máquina virtual debian. Ambos se conectan a través de una red virtual. Lo primero que haremos, es crear un usuario en cada servidor con los privilegios suficientes para poder realizar la interconexión. (Solo muestro la creación de un usuario para no repetir información.)


![BD-P2.2.png](/img/BD-P2.2.png)


He añadido como ejemplo el esquema SCOTT para poder visualizar algunas tablas. Vamos a configurar el archivo `tnsnames.ora`:

```txt
sudo nano /opt/oracle/product/19c/dbhome_1/network/admin/tnsnames.ora
```

```txt
ORCLCDB =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = 192.168.122.97)(PORT = 1521))
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = ORCLCDB)
    )
  )
```

Una vez modificado el fichero tnsnames.ora en ambos servidores, probaremos que ambos servidores se ven correctamente con el `tnsping`.

![BD-P2.4.png](/img/BD-P2.4.png)

A continuación, crearemos el enlace de ambas bases de datos, la sintaxis del comando para realizar el enlace es la siguiente:

```txt
CREATE DATABASE LINK nombreenlace 
CONNECT to usuario IDENTIFIED BY contraseña 
using 'alias-conexion-tnsora';
```

![BD-P2.3.png](/img/BD-P2.3.png)


Ahora, verificaremos el correcto funcionamiento del enlace que hemos creado anteriormente. Hacemos una consulta donde muestra el DEPTNO de un servidor y el DNAME de otro.

![BD-P2.5.png](/img/BD-P2.5.png)

---

### Realizar un enlace entre dos servidores de bases de datos Postgres, explicando la configuración necesaria en ambos extremos y demostrando su funcionamiento.

Lo primero que haré será crear la base de datos en ambos servidores.

```txt
usuario@PostgreSQL:~$ su - postgres
Contraseña: 
postgres@PostgreSQL:~$ psql
psql (11.17 (Debian 11.17-0+deb10u1))
Digite «help» para obtener ayuda.

postgres=# CREATE DATABASE SERVIDOR1;
CREATE DATABASE
```

Una vez creado las bases de datos en los 2 servidores, crearemos un usuario para cada base de datos y le otorgaremos los privilegios correspondientes.

```txt
postgres=# CREATE USER alejandro1 WITH PASSWORD 'usuario';
CREATE ROLE
postgres=# GRANT ALL PRIVILEGES ON DATABASE SERVIDOR1 to alejandro1;
GRANT
```

Nos conectaremos a la base de datos con el usuario que hemos creado para comprobar que funciona correctamente.

```txt
postgres@PostgreSQL:~$ psql -h localhost -U alejandro1 -d servidor1;
Contraseña para usuario alejandro1: 
psql (11.17 (Debian 11.17-0+deb10u1))
conexión SSL (protocolo: TLSv1.3, cifrado: TLS_AES_256_GCM_SHA384, bits: 256, compresión: desactivado)
Digite «help» para obtener ayuda.

servidor1=> 

```

En las bases de datos, introduciré algunas tablas y registros para mostrar el funcionamiento del enlace posteriormente. He añadido el esquema scott en ambas bases de datos como hice en el anterior ejercicio.

Una vez que hemos creado las tablas y hemos insertados los registros, modificaremos el fichero de configuración de postgres para que acepte conexiones de forma remota. El fichero que tenemos que editar es el siguiente:

```txt
usuario@PostgreSQL:~$ sudo nano /etc/postgresql/11/main/postgresql.conf
```
Editamos la línea que empieza por:

```txt
listen_addresses = 'localhost'
``` 
Y lo cambiamos por lo siguiente:

```txt
listen_addresses = '*'
```
Ahora, editaremos un segundo fichero que se ubica en:

```txt
usuario@PostgreSQL:~$ sudo nano /etc/postgresql/11/main/pg_hba.conf 
```

Buscamos la siguiente línea:

```txt
host    all             all             127.0.0.1/32            md5
```

Y ponemos lo siguiente:

```txt
host    all             all              192.168.122.0/24       md5
```
Podemos cambiar la dirección de red por `all` y así permitirá el trafico desde cualquier red.

Una vez modificados ambos ficheros reiniciaremos el servicio para que se apliquen los cambios.

```txt
usuario@PostgreSQL:~$ sudo systemctl restart postgresql
```

Esta sería la configuración para un servidor, tenemos que hacer exactamente los mismos pasos en el otro servidor, cambiando los nombres necesarios.

Instalaremos el paquete `postgresql-contrib`, ejecuntando el siguiente comando:

```txt
usuario@PostgreSQL:~$ sudo apt install postgresql-contrib
```

Nos cambiaremos al usuario postgres y nos conectaremos a la base de datos que hemos creado anteriormente.

```txt
usuario@PostgreSQL:~$ su - postgres
Contraseña: 
postgres@PostgreSQL:~$ psql -d servidor1
psql (11.17 (Debian 11.17-0+deb10u1))
Digite «help» para obtener ayuda.

servidor1=# 
```

Creamos la extension con la que nos conectaremos a la base de datos remota con el siguiente comando: 

```txt
servidor1=# CREATE EXTENSION dblink;
CREATE EXTENSION
```

Ahora probaré a hacer una consulta del DNAME y DEPTNO del servidor2 desde el servidor1:

```txt
SELECT * FROM dblink('dbname=servidor2 host=192.168.122.218 user=alejandro2 password=usuario', 'select DNAME, DEPTNO FROM DEPT') as DEPT (DNAME VARCHAR, DEPTNO NUMERIC);
```
![BD-P2.8.png](/img/BD-P2.8.png)

Repetiremos el proceso en el servidor 2 ya que el dblink es unidireccional y si queremos conectar el servidor2 con el servidor1 debemos repetir los pasos que hemos hecho anteriormente en el servidor1.

Ahora probaré a hacer una consulta del DNAME y DEPTNO del servidor1 desde el servidor2:

```txt
SELECT * FROM dblink('dbname=servidor1 host=192.168.122.98 user=alejandro1 password=usuario', 'select DNAME, DEPTNO FROM DEPT') as DEPT (DNAME VARCHAR, DEPTNO NUMERIC);
```
![BD-P2.9.png](/img/BD-P2.9.png)

---

### Realizar un enlace entre un servidor ORACLE y otro Postgres o MySQL empleando Heterogeneus Services, explicando la configuración necesaria en ambos extremos y demostrando su funcionamiento.


#### Oracle a PostgreSQL.

El primer paso que haremos en la máquina oracle consistirá en instalar la paquetéria para crear dicho enlace:

```txt
usuario@oracle:~$ sudo apt install odbc-postgresql unixodbc               
```

![BD-P2.10.png](/img/BD-P2.10.png)

Cuando ya esté instalado, procederemos a realizar la configuración en el servidor oracle. Modificaremos el archivo odbcinst.ini:

```txt
usuario@oracle:~$ sudo nano /etc/odbc.ini
---

Tenemos que dejar el siguiente contenido:

```txt
[PostgreSQL ANSI]
Description=PostgreSQL ODBC driver (ANSI version)
Driver=psqlodbca.so
Setup=libodbcpsqlS.so
Debug=0
CommLog=1
UsageCount=1

[PostgreSQL Unicode]
Description=PostgreSQL ODBC driver (Unicode version)
Driver=psqlodbcw.so
Setup=libodbcpsqlS.so
Debug=0
CommLog=1
UsageCount=1
```

Ahora editaremos otro fichero para terminar de configurar el servidor oracle:

```txt
usuario@oracle:~$ sudo nano /etc/odbc.ini 
```

```txt
[PSQLA]
Debug = 0
CommLog = 0
ReadOnly = 1
Driver = PostgreSQL ANSI   
Servername = 192.168.122.218
Username = alejandro2
Password = usuario
Port = 5432
Database = servidor2
Trace = 0
TraceFile = /tmp/sql.log

[PSQLU]
Debug = 0
CommLog = 0
ReadOnly = 0
Driver = PostgreSQL Unicode
Servername = 192.168.122.218
Username = alejandro2
Password = usuario
Port = 5432
Database = servidor2
Trace = 0
TraceFile = /tmp/sql.log

[Default]
 Driver = /usr/lib/x86_64-linux-gnu/odbc/liboplodbcS.so

```

Para comprobar que hemos configurado correctamente el fichero `odbcinst.ini` ejecutamos el siguiente comando:

```txt
usuario@oracle:~$ odbcinst -q -d
[PostgreSQL ANSI]
[PostgreSQL Unicode]
```
Para comprobar que hemos configurado correctamente el fichero `odbc.ini` ejecutamos el siguiente comando:

```txt
usuario@oracle:~$ odbcinst -q -s
[PSQLA]
[PSQLU]
[Default]
```
Con este comando probamos la conexión a postgres;

```txt
usuario@oracle:~$ sudo isql -v PSQLU                                      
[sudo] password for usuario:                                              
+---------------------------------------+
| Connected!                            |
|                                       |
| sql-statement                         |
| help [tablename]                      |
| quit                                  |
|                                       |
+---------------------------------------+
SQL> quit
usuario@oracle:~$ sudo isql -v PSQLA
+---------------------------------------+
| Connected!                            |
|                                       |
| sql-statement                         |
| help [tablename]                      |
| quit                                  |
|                                       |
+---------------------------------------+
```

Una vez comprobado que las conexiones funcionan correctamente, vamos a configurar el `heterogeneus services`. Tendremos que crear un fichero dentro de uno de los directorios de oracle y añadiremos lo siguiente:

```txt
usuario@oracle:~$ sudo nano /opt/oracle/product/19c/dbhome_1/hs/admin/initPSQLU.ora
```

Añadimos lo siguiente:

```txt
HS_FDS_CONNECT_INFO = PSQLU
HS_FDS_TRACE_LEVEL = Debug
HS_FDS_SHAREABLE_NAME = /usr/lib/x86_64-linux-gnu/odbc/psqlodbcw.so
HS_LANGUAGE = AMERICAN_AMERICA.WE8ISO8859P1
set ODBCINI=/etc/odbc.ini
```

Ahora modificaremos el `listener.ora` añadiendo lo siguiente:

```txt
usuario@oracle:~$ sudo nano /opt/oracle/product/19c/dbhome_1/network/admin/listener.ora
```

```txt
SID_LIST_LISTENER=
 (SID_LIST=
 (SID_DESC=
 (SID_NAME=PSQLU)
 (ORACLE_HOME=/opt/oracle/product/19c/dbhome_1)
 (PROGRAM=dg4odbc)
 )
 )
```

También modificaremos el `tnsnames.ora` añadiendo lo siguiente:

```txt
usuario@oracle:~$ sudo nano /opt/oracle/product/19c/dbhome_1/network/admin/tnsnames.ora
```
```txt
PSQLU =
 (DESCRIPTION=
   (ADDRESS=(PROTOCOL=tcp)(HOST=localhost)(PORT=1521))
   (CONNECT_DATA=(SID=PSQLU))
 (HS=OK)
 )
```

Una vez tengamo todo configurado guardaremos los cambios y reiniciaremos el servidor:

```txt
usuario@oracle:~$ sudo systemctl restart oracledb_ORCLCDB-19c.service
```

En el servidor postgres, tenemos que permitir conexiones remotas no documentaré nuevamente como se hace ya que está explicado mas arriba.

Cuando hayamos terminado de configurar ambos servidores correctamente, procederenis a realizar la creación del enlace de la base de datos en oracle para que se pueda consultar la de postgres.

```txt
CREATE PUBLIC DATABASE LINK psql1 
CONNECT TO "alejandro2" IDENTIFIED BY "usuario"
USING 'PSQLU'
```

![BD-P2.11.png](/img/BD-P2.11.png)

Ahora hacemos una prueba para comprobar que funciona:

![BD-P2.12.png](/img/BD-P2.12.png)

#### Postgres a Oracle.

Para realizar la conexión desde un servidor Postgres a un servidor oracle, primero, debemos de instalar los siguientes paquetes:

```txt
usuario@PostgreSQL:~$ sudo apt install libaio1 postgresql-server-dev-all build-e
ssential git                                                                    
[sudo] password for usuario:                                                    
Leyendo lista de paquetes... Hecho                                              
Creando árbol de dependencias                                                   
Leyendo la información de estado... Hecho                                       
build-essential ya está en su versión más reciente (12.6).                      
fijado build-essential como instalado manualmente.                              
libaio1 ya está en su versión más reciente (0.3.112-3).                         
Se instalarán los siguientes paquetes adicionales:                              
  binfmt-support clang-7 dctrl-tools git-man lib32gcc1 lib32stdc++6 libc6-i386  
  libclang-common-7-dev libclang1-7 liberror-perl libffi-dev libgc1c2
  libncurses-dev libncurses6 libncursesw6 libobjc-8-dev libobjc4 libomp-7-dev
  libomp5-7 libpq-dev libtinfo-dev libtinfo6 llvm-7 llvm-7-dev llvm-7-runtime
  postgresql-server-dev-11

```

Ahora descargaremos los paquetes oficiales de `Oracle Instant Client`:

```txt
postgres@PostgreSQL:~$ wget https://download.oracle.com/otn_software/linux/instantclient/211000/instantclient-basic-linux.x64-21.1.0.0.0.zip
postgres@PostgreSQL:~$ wget https://download.oracle.com/otn_software/linux/instantclient/211000/instantclient-sdk-linux.x64-21.1.0.0.0.zip
postgres@PostgreSQL:~$ wget https://download.oracle.com/otn_software/linux/instantclient/211000/instantclient-sqlplus-linux.x64-21.1.0.0.0.zip

```

Extraemos los archivos .zip con el siguiente comando:

```txt
postgres@PostgreSQL:~$ unzip instantclient-basic-linux.x64-21.1.0.0.0.zip
postgres@PostgreSQL:~$ unzip instantclient-sdk-linux.x64-21.1.0.0.0.zip
postgres@PostgreSQL:~$ unzip instantclient-sqlplus-linux.x64-21.1.0.0.0.zip
```

Tenemos que usar las siguientes variables para que funcione correctamente el cliente de oracle:

```txt
postgres@PostgreSQL:/$ export ORACLE_HOME=/var/lib/postgresql/instantclient_21_1/
postgres@PostgreSQL:/$ export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$ORACLE_HOME
postgres@PostgreSQL:/$ export PATH=$PATH:$ORACLE_HOME
```

como podemos ver, ya nos devuelve la ruta cuando hacemos un which:

```txt
postgres@PostgreSQL:/$ which sqlplus
/var/lib/postgresql/instantclient_21_1/sqlplus
```

Y si probamos a hacer una conexión remota desde posgres funciona correctamente.

![BD-P2.13.png](/img/BD-P2.13.png)

Ahora instalaremos el paquete `oracle_fdw`, para ellos tendremos que clonar el siguiente repositorio de github:

```txt
git clone https://github.com/laurenz/oracle_fdw.git
```

entramos en la carpeta del repositorio clonado y a continuación, compilaremos el paquete. Ejecutamos el siguiente comando:

```txt
postgres@PostgreSQL:~/oracle_fdw$ make                                          
gcc -Wall -Wmissing-prototypes -Wpointer-arith -Wdeclaration-after-statement -Wendif-labels -Wmissing-format-attribute -Wformat-security -fno-strict-aliasing -fwrapv -fexcess-precision=standard -Wno-format-truncation -Wno-stringop-truncation -g -g -O2 -fstack-protector-strong -Wformat -Werror=format-security -fno-omit-frame-pointer -fPIC -I"/var/lib/postgresql/instantclient_21_1/sdk/include" -I"/var/lib/postgresql/instantclient_21_1/oci/include" -I"/var/lib/postgresql/instantclient_21_1/rdbms/public" -I"/var/lib/postgresql/instantclient_21_1/"  -I. -I./ -I/usr/include/postgresql/11/server -I/usr/include/postgresql/internal  -Wdate-time -D_FORTIFY_SOURCE=2 -D_GNU_SOURCE -I/usr/include/libxml2  -I/usr/include/mit-krb5  -c -o oracle_fdw.o oracle_fdw.c                                          
```
Una vez haya terminado, ejecutamos el make install.

```txt
postgres@PostgreSQL:~/oracle_fdw$ sudo make install
[sudo] password for postgres: 
/bin/mkdir -p '/usr/lib/postgresql/11/lib'
/bin/mkdir -p '/usr/share/postgresql/11/extension'
/bin/mkdir -p '/usr/share/postgresql/11/extension'
/bin/mkdir -p '/usr/share/doc/postgresql-doc-11/extension'
/usr/bin/install -c -m 755  oracle_fdw.so '/usr/lib/postgresql/11/lib/oracle_fdw.so'
/usr/bin/install -c -m 644 .//oracle_fdw.control '/usr/share/postgresql/11/extension/'
/usr/bin/install -c -m 644 .//oracle_fdw--1.2.sql .//oracle_fdw--1.0--1.1.sql .//oracle_fdw--1.1--1.2.sql  '/usr/share/postgresql/11/extension/'
/usr/bin/install -c -m 644 .//README.oracle_fdw '/usr/share/doc/postgresql-doc-11/extension/'
```

Con el siguiente comando, generaremos los enlaces necesarios y cargaremos en memorai las nuevas librerías compartidas que hemos introducido:

```txt
postgres@PostgreSQL:~/oracle_fdw$ sudo ldconfig
```

Ya hemos terminado toda la configuración necesaria, ahora crearemos el enlace y empezaremos a usarlo. Para ello, debemos de acceder a nuestra base de datos de postgres.

Al crear la extensión me daba el siguiente fallo:

```txt
servidor1=# CREATE EXTENSION oracle_fdw;
ERROR:  no se pudo cargar la biblioteca «/usr/lib/postgresql/11/lib/oracle_fdw.so»: libclntsh.so.21.1: no se puede abrir el fichero del objeto compartido: No existe el fichero o el directorio
```

Lo he solucionado añadiendo al siguiente fichero lo siguiente:

```txt
postgres@PostgreSQL:~$ sudo nano /etc/ld.so.conf
```txt

añadimos la siguiente ruta:

```txt
/var/lib/postgresql/instantclient_21_1/
```

Y ahora ya podemos crear el enlace correctamente:

```txt
postgres@PostgreSQL:~$ psql -d servidor1
psql (11.17 (Debian 11.17-0+deb10u1))
Digite «help» para obtener ayuda.

servidor1=# CREATE EXTENSION oracle_fdw;
CREATE EXTENSION
```

Una vez creado el enlace crearemos el schema donde importaremos la base de datos de oracle:

```txt
servidor1=# CREATE SCHEMA oracle;
CREATE SCHEMA
```

Ahora que está creado el schema, tendremos que importar las tablas de la base de datos de oracle ya que actualmente el schema está vacío:

```txt
servidor1=# CREATE SERVER oracle FOREIGN DATA WRAPPER oracle_fdw OPTIONS (dbserver '//192.168.122.1/ORCLCDB');
CREATE SERVER
```

También tenemos que mapear nuestro usuario local a un usuario en el otro servidor para que pueda acceder y tenga privilegios sobre las tablas.

```txt
servidor1=# CREATE USER MAPPING FOR alejandro1 SERVER oracle OPTIONS (user 'c##servidor1', password 'usuario');
CREATE USER MAPPING
```

Por último, le daremos privilegios a nuestro usuario del servidor sobre el nuevo esquema y el servidor remoto.

```txt
servidor1=# GRANT ALL PRIVILEGES ON SCHEMA oracle to alejandro1;
GRANT
servidor1=# GRANT ALL PRIVILEGES ON FOREIGN SERVER oracle TO alejandro1;
GRANT
```

Ahora accederemos a la base de datos con el usuario al que le hemos dado privilegios.

```txt
postgres@PostgreSQL:~$ psql -h localhost -U alejandro1 -d servidor1
Contraseña para usuario alejandro1: 
psql (11.17 (Debian 11.17-0+deb10u1))
conexión SSL (protocolo: TLSv1.3, cifrado: TLS_AES_256_GCM_SHA384, bits: 256, compresión: desactivado)
Digite «help» para obtener ayuda.
```

Ahora importaremos el schema que hemos creado anteriormente.

```txt
servidor1=> IMPORT FOREIGN SCHEMA "C##SERVIDOR1" FROM SERVER "oracle" INTO oracle;
IMPORT FOREIGN SCHEMA
```
Vamos a realizar una consulta para demostrar que funciona correctamente.

```txt
 SELECT DEPT.DNAME AS Departamento, EMP.ENAME AS Nombre
 FROM oracle.EMP EMP, DEPT
 WHERE EMP.DEPTNO = DEPT.DEPTNO;
```

![BD-P2.14.png](/img/BD-P2.14.png)

## **Documento realizado por:**

 ✒️ **Alejandro Montes Delgado** - *2º ASIR*
