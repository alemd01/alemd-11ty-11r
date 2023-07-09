---
layout: post
title: Movimiento de datos
excerpt: En este post veremos como realizar exportaciones e importaciones y trabajaremos con ellas.
date: 2023-02-25
updatedDate: 2023-02-25
tags:
  - post
  - GBD
---

### Realiza una exportación del esquema de SCOTT usando Oracle Data Pump con las siguientes condiciones:

* **Exporta tanto la estructura de las tablas como los datos de las mismas.**
* **Excluye la tabla BONUS y los departamentos con menos de dos empleados.**
* **Realiza una estimación previa del tamaño necesario para el fichero de exportación.**
* **Programa la operación para dentro de 2 minutos.**
* **Genera un archivo de log en el directorio raíz.**

Lo primero que haremos será crear un directorio donde se guardará la exportación:

```txt
alemd@debian:~$ sudo mkdir /opt/oracle/export
```

Ahora nos conectamos con un usuario con privilegios y creamos un directorio objeto en la base de datos, le damos permiso de lectura y escritura en el directorio y le damos privilegios para que pueda exportar:

```txt
SQL> CREATE DIRECTORY BD_EXPORT AS '/opt/oracle/export';

Directorio creado.

SQL> GRANT READ,WRITE ON DIRECTORY BD_EXPORT TO SCOTT;

Concesion terminada correctamente.


SQL> GRANT  DATAPUMP_EXP_FULL_DATABASE TO SCOTT;

Concesion terminada correctamente.

```

Ejecutamos el siguiente comando para ver la estimación del tamaño del fichero de exportación:

```txt
alemd@debian:~$ expdp scott/TIGER schemas=scott exclude=table:\"IN \(\'BONUS\'\)\" directory=BD_EXPORT ESTIMATE_ONLY=YES

Export: Release 19.0.0.0.0 - Production on Sun Feb 26 16:23:01 2023
Version 19.3.0.0.0

Copyright (c) 1982, 2019, Oracle and/or its affiliates.  All rights reserved.

Connected to: Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production

Advertencia: Las operaciones de Oracle Data Pump no se necesitan normalmente cuando se conecta a la raiz o al elemento inicial de una base de datos del contenedor.

Iniciando "SCOTT"."SYS_EXPORT_SCHEMA_01":  scott/******** schemas=scott exclude=table:"IN ('BONUS')" directory=BD_EXPORT ESTIMATE_ONLY=YES 
Estimacion en curso mediante el metodo BLOCKS...
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/TABLE_DATA
.  estimado "SCOTT"."EMP"                                 896 MB
.  estimado "SCOTT"."DEPT"                                 64 KB
Estimacion total mediante el metodo BLOCKS: 896.0 MB
El trabajo "SCOTT"."SYS_EXPORT_SCHEMA_01" ha terminado correctamente en Dom Feb 26 16:23:10 2023 elapsed 0 00:00:08


```


Creamos un script para realizar la exportación y lo ejecutamos:

```txt
echo "La exportación se realizará en 2 minutos..."
sleep 120
expdp scott/TIGER schemas=scott exclude=table:\"IN \(\'BONUS\'\)\" directory=BD_EXPORT dumpfile=SCOTT.dmp logfile=SCOTT.log

```
Lo ejecutamos:

```txt
alemd@debian:~$ ./export.sh 
La exportación se realizará en 2 minutos...

Export: Release 19.0.0.0.0 - Production on Sat Feb 25 21:09:00 2023
Version 19.3.0.0.0

Copyright (c) 1982, 2019, Oracle and/or its affiliates.  All rights reserved.

Connected to: Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production

Advertencia: Las operaciones de Oracle Data Pump no se necesitan normalmente cuando se conecta a la raiz o al elemento inicial de una base de datos del contenedor.

Iniciando "SCOTT"."SYS_EXPORT_SCHEMA_01":  scott/******** schemas=scott exclude=table:"IN ('BONUS')" directory=BD_EXPORT dumpfile=SCOTT.dmp logfile=SCOTT.log 
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/TABLE_DATA
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/INDEX/STATISTICS/INDEX_STATISTICS
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/STATISTICS/TABLE_STATISTICS
Procesando el tipo de objeto SCHEMA_EXPORT/STATISTICS/MARKER
Procesando el tipo de objeto SCHEMA_EXPORT/PRE_SCHEMA/PROCACT_SCHEMA
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/TABLE
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/GRANT/OWNER_GRANT/OBJECT_GRANT
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/COMMENT
Procesando el tipo de objeto SCHEMA_EXPORT/PROCEDURE/PROCEDURE
Procesando el tipo de objeto SCHEMA_EXPORT/PROCEDURE/ALTER_PROCEDURE
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/INDEX/INDEX
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/CONSTRAINT/CONSTRAINT
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/CONSTRAINT/REF_CONSTRAINT
. . "SCOTT"."EMP"                               22.41 KB      14 filas exportadas
. . "SCOTT"."DEPT"                              6.046 KB       5 filas exportadas
La tabla maestra "SCOTT"."SYS_EXPORT_SCHEMA_01" se ha cargado/descargado correctamente
******************************************************************************
El juego de archivos de volcado para SCOTT.SYS_EXPORT_SCHEMA_01 es:
  /opt/oracle/export/SCOTT.dmp
El trabajo "SCOTT"."SYS_EXPORT_SCHEMA_01" ha terminado correctamente en Sab Feb 25 21:09:14 2023 elapsed 0 00:00:13

```

No he podido excluir los departamentos con menos de dos empleados debido a que usando el parámetro query e indicando que para la tabla dept coja los que tienen más de dos empleados, no me funcionaba. Adjunto el parámetro query y su contenido:

```txt
query=DEPT:\"WHERE \(SELECT COUNT\(*\) FROM EMP WHERE EMP.DEPTNO = DEPT.DEPTNO\) \> 1\"
```

### Importa el fichero obtenido anteriormente usando Oracle Data Pump pero en un usuario distinto de la misma base de datos.

Primero le damos permisos de lectura y escritura sobre el directorio donde está almacenada la exportación al usuario en el que la importaremos:

```txt
SQL> GRANT READ, WRITE ON DIRECTORY BD_EXPORT TO alemd;

Concesion terminada correctamente.

```

Antes de realizar la importación, al usuario en el cual la realizaremos, le concederemos privilegios sobre importación:

```txt
SQL> GRANT IMP_FULL_DATABASE TO alemd;

Concesion terminada correctamente.

```

Con el comando impdp importaremos la exportación realizada anteriormente:

```txt
alemd@debian:~$ impdp alemd/usuario schemas=SCOTT directory=BD_EXPORT dumpfile=SCOTT.dmp logfile=SCOTTimport.log 

Import: Release 19.0.0.0.0 - Production on Sun Feb 26 13:37:10 2023
Version 19.3.0.0.0

Copyright (c) 1982, 2019, Oracle and/or its affiliates.  All rights reserved.

Connected to: Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production

Advertencia: Las operaciones de Oracle Data Pump no se necesitan normalmente cuando se conecta a la raiz o al elemento inicial de una base de datos del contenedor.

La tabla maestra "ALEMD"."SYS_IMPORT_SCHEMA_01" se ha cargado/descargado correctamente
Iniciando "ALEMD"."SYS_IMPORT_SCHEMA_01":  alemd/******** schemas=SCOTT directory=BD_EXPORT dumpfile=SCOTT.dmp logfile=SCOTTimport.log 
Procesando el tipo de objeto SCHEMA_EXPORT/ROLE_GRANT
Procesando el tipo de objeto SCHEMA_EXPORT/DEFAULT_ROLE
Procesando el tipo de objeto SCHEMA_EXPORT/PRE_SCHEMA/PROCACT_SCHEMA
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/TABLE
ORA-39151: La tabla "SCOTT"."EMP" existe. Todos los metadados dependientes y los datos se omitiran debido table_exists_action de omitir

ORA-39151: La tabla "SCOTT"."DEPT" existe. Todos los metadados dependientes y los datos se omitiran debido table_exists_action de omitir

Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/TABLE_DATA
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/GRANT/OWNER_GRANT/OBJECT_GRANT
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/FGA_POLICY
ORA-39083: Fallo de creacion del tipo de objeto FGA_POLICY con el error:
ORA-28101: la politica ya existe

El sql que falla es:
BEGIN DBMS_FGA.ADD_POLICY('SCOTT','EMP','POLICY_EJ4','SAL > 2000','','','',TRUE,'INSERT',DBMS_FGA.DB_EXTENDED,DBMS_FGA.ANY_COLUMNS,'SYS'); END;

Procesando el tipo de objeto SCHEMA_EXPORT/PROCEDURE/PROCEDURE
ORA-31684: El tipo de objeto PROCEDURE:"SCOTT"."ADD_10000_EMP" ya existe

ORA-31684: El tipo de objeto PROCEDURE:"SCOTT"."ADD_100000_EMP" ya existe

Procesando el tipo de objeto SCHEMA_EXPORT/PROCEDURE/ALTER_PROCEDURE
ORA-39111: Se ha saltado el tipo de objeto dependiente ALTER_PROCEDURE:"SCOTT"."ADD_100000_EMP", ya existe el tipo de objeto base PROCEDURE:"SCOTT"."ADD_100000_EMP"

ORA-39111: Se ha saltado el tipo de objeto dependiente ALTER_PROCEDURE:"SCOTT"."ADD_10000_EMP", ya existe el tipo de objeto base PROCEDURE:"SCOTT"."ADD_10000_EMP"

Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/INDEX/INDEX
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/CONSTRAINT/CONSTRAINT
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/INDEX/STATISTICS/INDEX_STATISTICS
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/CONSTRAINT/REF_CONSTRAINT
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/STATISTICS/TABLE_STATISTICS
Procesando el tipo de objeto SCHEMA_EXPORT/STATISTICS/MARKER
El trabajo "ALEMD"."SYS_IMPORT_SCHEMA_01" ha terminado con 7 error(es) en Dom Feb 26 13:37:19 2023 elapsed 0 00:00:08

```

Salen errores, porque no es el primer intento con ese usuario y se han duplicado cosas. A continuación nos conectaremos con el usuario y realizaremos una consulta a la tabla DEPT.

```txt
SQL> SELECT DNAME FROM SCOTT.DEPT;

DNAME
--------------
PRUEBA
ACCOUNTING
RESEARCH
SALES
OPERATIONS

```

Tenemos que indicar el esquema al consultar la tabla debido a que hemos realizado la exportación del esquema y al realizar la importación, el esquema se ha importado.

### Realiza una exportación de la estructura de todas las tablas de la base de datos usando el comando expdp de Oracle Data Pump probando al menos cinco de las posibles opciones que ofrece dicho comando y documentándolas adecuadamente.

La exportación la realizo usando el siguiente comando:

```txt
alemd@debian:~$ expdp system/usuario DIRECTORY=BD_EXPORT DUMPFILE=full_export.dmp FULL=YES CONTENT=METADATA_ONLY exclude=table:\"IN \(\'SCOTT.DEPT\'\)\" 

Export: Release 19.0.0.0.0 - Production on Sun Feb 26 16:29:46 2023
Version 19.3.0.0.0

Copyright (c) 1982, 2019, Oracle and/or its affiliates.  All rights reserved.

Connected to: Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production

Advertencia: Las operaciones de Oracle Data Pump no se necesitan normalmente cuando se conecta a la raiz o al elemento inicial de una base de datos del contenedor.

Iniciando "SYSTEM"."SYS_EXPORT_FULL_01":  system/******** DIRECTORY=BD_EXPORT DUMPFILE=full_export.dmp FULL=YES CONTENT=METADATA_ONLY exclude=table:"IN ('SCOTT.DEPT')" 
Procesando el tipo de objeto DATABASE_EXPORT/EARLY_OPTIONS/VIEWS_AS_TABLES/TABLE_DATA
Procesando el tipo de objeto DATABASE_EXPORT/NORMAL_OPTIONS/TABLE_DATA
Procesando el tipo de objeto DATABASE_EXPORT/NORMAL_OPTIONS/VIEWS_AS_TABLES/TABLE_DATA

.
.
.
.
.

La tabla maestra "SYSTEM"."SYS_EXPORT_FULL_01" se ha cargado/descargado correctamente
******************************************************************************
El juego de archivos de volcado para SYSTEM.SYS_EXPORT_FULL_01 es:
  /opt/oracle/export/full_export.dmp
El trabajo "SYSTEM"."SYS_EXPORT_FULL_01" ha terminado correctamente en Dom Feb 26 16:36:18 2023 elapsed 0 00:06:31

```

Ahora, explicaré todos los parámetros usados:

* DIRECTORY: Con este parámetro indicamos el directorio donde se guardará el dump, como vemos indica BD_EXPORT, esto lo hemos creados desde la terminal de sqlplus con el `CREATE DIRECTORY`, que indicamos un nombre del directorio y la ruta.

* DUMPFILE: Es el nombre del archivo donde se guarda la exportación.

* FULL: Indicamos que la exportación es de la base de datos al completo.

* CONTENT: Con este parámetro, indicamos el contenido. Podemos indicar ALL que es por defecto e indica todo el contenido, DATA_ONLY solo guarda datos y METADATA_ONLY solo guarda metadatos como la estructúras de tablas.

* EXCLUDE: Con este parámetros indicamos lo que queremos excluir, ya sea a través de una expresión regular, una tabla, un esquema...

### Intenta realizar operaciones similares de importación y exportación con las herramientas proporcionadas con MySQL desde línea de comandos, documentando el proceso.

Vamos a realizar una exportación de la base de datos SCOTT y la importaremos en una base de datos llamada import:

```txt
root@debian:/home/usuario# mysqldump -u root SCOTT > mariadbexport.sql
root@debian:/home/usuario# ls
alemd.key  gonzalonazareno.crt	PRACTICA_DOCKER
docker	   mariadbexport.sql
```

Con este comando, hemos exportado la base de datos SCOTT al fichero mariadbexport.sql. Ahora Crearemos una base de datos totalmente limpia llamada import que es donde importaremos el dump.

```txt
MariaDB [(none)]> CREATE DATABASE IMPORT_SCOTT;
Query OK, 1 row affected (0,001 sec)

MariaDB [(none)]> EXIT
Bye
```

Ahora ejecutamos el siguiente comando para importar los datos:

```txt
root@debian:/home/usuario# mysql -u root IMPORT_SCOTT < mariadbexport.sql
```

Nos conectamos a la base de datos y mostramos que se han creado las tablas correctamente:

```txt
root@debian:/home/usuario# mysql
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 35
Server version: 10.5.18-MariaDB-0+deb11u1 Debian 11

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> use IMPORT_SCOTT
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MariaDB [IMPORT_SCOTT]> show tables;
+------------------------+
| Tables_in_IMPORT_SCOTT |
+------------------------+
| DEPT                   |
| EMP                    |
+------------------------+
2 rows in set (0,000 sec)
```

Ahora realizaré una consulta para probar que los datos están insertados correctamente:

```txt
MariaDB [IMPORT_SCOTT]> SELECT * FROM DEPT;
+--------+------------+----------+
| DEPTNO | DNAME      | LOC      |
+--------+------------+----------+
|     10 | ACCOUNTING | NEW YORK |
|     20 | RESEARCH   | DALLAS   |
|     30 | SALES      | CHICAGO  |
|     40 | OPERATIONS | BOSTON   |
+--------+------------+----------+
4 rows in set (0,001 sec)

```

### Intenta realizar operaciones similares de importación y exportación con las herramientas proporcionadas con Postgres desde línea de comandos, documentando el proceso.

En nuestra base de datos postgres, tenemos la base de datos de un aeropuerto, y procederemos a realizar una exportación:

```txt
postgres@PostgreSQL:~$ pg_dump -U postgres aeropuerto > aeropuertodump.sql
postgres@PostgreSQL:~$ ls
11  aeropuertodump.sql	instantclient_21_1  oracle_fdw	PG_11_201809051
```

Ahora lo importaremos en la base de datos import_aeropuerto:

```txt
postgres@PostgreSQL:~$ psql -U postgres -d import_aeropuerto < aeropuertodump.sql 
SET
SET
SET
SET
SET
 set_config 
------------
 
(1 fila)

SET
SET
SET
SET
CREATE FUNCTION
ALTER FUNCTION
CREATE FUNCTION
ALTER FUNCTION
SET
SET
CREATE TABLE
ALTER TABLE
CREATE TABLE
ALTER TABLE
CREATE TABLE
ALTER TABLE
CREATE TABLE
ALTER TABLE
CREATE TABLE
ALTER TABLE
CREATE TABLE
ALTER TABLE
CREATE TABLE
ALTER TABLE
CREATE TABLE
ALTER TABLE
CREATE TABLE
ALTER TABLE
CREATE TABLE
ALTER TABLE
CREATE TABLE
ALTER TABLE
CREATE TABLE
ALTER TABLE
COPY 8
COPY 6
COPY 6
COPY 20
COPY 3
COPY 4
COPY 10
COPY 18
COPY 12
COPY 3
COPY 12
COPY 5
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
CREATE TRIGGER
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
GRANT

```

Nos conectamos a la base de datos y mostramos que se ha importado correctamente:

```txt
postgres@PostgreSQL:~$ psql
psql (11.17 (Debian 11.17-0+deb10u1))
Digite «help» para obtener ayuda.

postgres=# \c import_aeropuerto
Ahora está conectado a la base de datos «import_aeropuerto» con el usuario «postgres».
import_aeropuerto=# \d
              Listado de relaciones
 Esquema |       Nombre        | Tipo  |  Dueño   
---------+---------------------+-------+----------
 public  | aeronaves           | tabla | postgres
 public  | aeropuertos         | tabla | postgres
 public  | auxiliares          | tabla | postgres
 public  | auxiliaresdeviaje   | tabla | postgres
 public  | copilotos           | tabla | postgres
 public  | modelos             | tabla | postgres
 public  | pasajeros           | tabla | postgres
 public  | pasajerosembarcados | tabla | postgres
 public  | personal            | tabla | postgres
 public  | pilotos             | tabla | postgres
 public  | viajes              | tabla | postgres
 public  | vuelos              | tabla | postgres
(12 filas)

```

Hacemos una consulta a la tabla pilotos:

```txt
import_aeropuerto=# select * from pilotos;
 numpasaporte 
--------------
 CVF000126
 DSA000777
 EJK000333
(3 filas)

```

### Exporta los documentos de una colección de MongoDB que cumplan una determinada condición e impórtalos en otra base de datos.

Tenemos una colección con algunos restaurantes de estados unidos. Vamos a exportar los restaurantes que sean del barrio del Bronx y de tipo de comida Americana:

```txt
root@mongodb:/home/vagrant# mongoexport -u admin --db admin --collection restaurants -q "{ \"borough\": \"Bronx\", \"cuisine\": \"American\" }" --out restaurantes_americanos_bronx.json
Enter password for mongo user:

2023-02-26T18:43:11.396+0000	connected to: mongodb://localhost/
2023-02-26T18:43:11.524+0000	exported 822 records
root@mongodb:/home/vagrant# ls
primer-dataset.json  restaurantes_americanos_bronx.json
```

Ahora lo importaremos en una base de datos llamada prueba:

```txt
root@mongodb:/home/vagrant# mongoimport -u alemd --db prueba --collection restaurants --file restaurantes_americanos_bronx.json
Enter password for mongo user:

2023-02-26T19:01:51.176+0000	connected to: mongodb://localhost/
2023-02-26T19:01:51.777+0000	822 document(s) imported successfully. 0 document(s) failed to import.
```

Nos conectamos y haremos una consulta para ver que funciona:

```txt
root@mongodb:/home/vagrant# mongosh -u alemd -p --authenticationDatabase prueba
Enter password: *******
Current Mongosh Log ID:	63fbacdc88222b513ba0b957
Connecting to:		mongodb://<credentials>@127.0.0.1:27017/?directConnection=true&serverSelectionTimeoutMS=2000&authSource=prueba&appName=mongosh+1.7.1
Using MongoDB:		6.0.4
Using Mongosh:		1.7.1

For mongosh info see: https://docs.mongodb.com/mongodb-shell/

Enterprise test> use prueba
switched to db prueba
Enterprise prueba> db.restaurants.findOne()
{
  _id: ObjectId("63fba382a0e597287f628a4a"),
  address: {
    building: '2300',
    coord: [ -73.8786113, 40.8502883 ],
    street: 'Southern Boulevard',
    zipcode: '10460'
  },
  borough: 'Bronx',
  cuisine: 'American',
  grades: [
    {
      date: ISODate("2014-05-28T00:00:00.000Z"),
      grade: 'A',
      score: 11
    },
    { date: ISODate("2013-06-19T00:00:00.000Z"), grade: 'A', score: 4 },
    { date: ISODate("2012-06-15T00:00:00.000Z"), grade: 'A', score: 3 }
  ],
  name: 'Wild Asia',
  restaurant_id: '40357217'
}
```

### SQLLoader es una herramienta que sirve para cargar grandes volúmenes de datos en una instancia de ORACLE. Exportad los datos de una base de datos completa desde Postgres a texto plano con delimitadores y emplead SQLLoader para realizar el proceso de carga de dichos datos a una instancia ORACLE. Debéis documentar todo el proceso, explicando los distintos ficheros de configuración y de log que tiene SQLLoader.

Lo primero que haremos será en postgres exportar la base de datos del aeropuerto como un archivo csv delimitado por coma. Para ello, he creado un procedimiento que me creará por cada tabla de la base de datos un fichero csv con los datos de cada tabla:

```txt
postgres=# CREATE OR REPLACE FUNCTION exportar_tablas_a_csv(
    _nombre_de_la_base_de_datos TEXT,
    _ruta TEXT
)
RETURNS VOID AS $$
DECLARE
    _nombre_de_la_tabla TEXT;
BEGIN
    FOR _nombre_de_la_tabla IN
        SELECT table_name
        FROM information_schema.tables
        WHERE table_schema = 'public'
        AND table_type = 'BASE TABLE'
    LOOP
        EXECUTE format(
            'COPY %I TO %L WITH (FORMAT CSV, DELIMITER ",", HEADER)',
            _nombre_de_la_tabla,
            _ruta || _nombre_de_la_tabla || '.csv'
        );
    END LOOP;
END;
$$ LANGUAGE plpgsql;
CREATE FUNCTION

```

Ahora ejecutamos el procedimiento:

```txt
aeropuerto=# SELECT exportar_tablas_a_csv('SCOTT', '/var/lib/postgresql/');
 exportar_tablas_a_csv 
-----------------------
 
(1 fila)

```

Como vemos, tenemos las tablas emp y dept en ficheros csv:

```txt
postgres@PostgreSQL:~$ ls
11	    dept.csv  instantclient_21_1  PG_11_201809051
aeropuerto  emp.csv   oracle_fdw

```

Ahora creamos los archivos de control que usaremos para importar cada tabla:

```txt
OPTIONS (SKIP=1)
LOAD DATA
INFILE '/home/alemd/oracle/dept.csv'
INTO TABLE DEPT
FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '"'
TRAILING NULLCOLS
(deptno, DNAME, LOC)
```

Ahora muestro el fichero de control de la tabla emp:

```txt
OPTIONS (SKIP=1)
LOAD DATA
INFILE '/home/alemd/oracle/emp.csv'
INTO TABLE emp
FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '"'
TRAILING NULLCOLS
(EMPNO,
ENAME,
JOB,
MGR,
HIREDATE DATE "YYYY-MM-DD HH24:MI:SS",
SAL DECIMAL EXTERNAL (7),
COMM DECIMAL EXTERNAL (7),
DEPTNO)
```

La opción skip=1, hace que no inserte la primera fila, que en el fichero csv sería el nombre de las columnas, con load data indicamos que cargue los datos desde el archivo emp.csv a la tabla emp y está separado por una coma y opcionalmente cerrado por una comilla doble. Por último, indicamos el nombre de las columnas, y si necesita algún formato concreto como una fecha o un número decimal.

Ejecutamos el sql loader para la tabla dept, como vemos se han cargado correctamente los datos.

```txt
alemd@debian:~/oracle$ sqlldr SCOTT/TIGER control=/home/alemd/oracle/dept.ctl log=/home/alemd/oracle/dept.log

SQL*Loader: Release 19.0.0.0.0 - Production on Mon Feb 27 00:15:41 2023
Version 19.3.0.0.0

Copyright (c) 1982, 2019, Oracle and/or its affiliates.  All rights reserved.

Path used:      Conventional
Commit point reached - logical record count 4

Table DEPT:
  4 Rows successfully loaded.

Check the log file:
  /home/alemd/oracle/dept.log
for more information about the load.
```

Ejecutamos el sql loader para la tabla emp:

```txt
alemd@debian:~/oracle$ sqlldr SCOTT/TIGER control=/home/alemd/oracle/emp.ctl log=/home/alemd/oracle/emp.log

SQL*Loader: Release 19.0.0.0.0 - Production on Wed Mar 1 10:39:05 2023
Version 19.3.0.0.0

Copyright (c) 1982, 2019, Oracle and/or its affiliates.  All rights reserved.

Path used:      Conventional
Commit point reached - logical record count 14

Table EMP:
  14 Rows successfully loaded.

Check the log file:
  /home/alemd/oracle/emp.log
for more information about the load.
```


 Ahora veremos el log:

```txt
alemd@debian:~/oracle$ cat dept.log 

SQL*Loader: Release 19.0.0.0.0 - Production on Mon Feb 27 00:15:41 2023
Version 19.3.0.0.0

Copyright (c) 1982, 2019, Oracle and/or its affiliates.  All rights reserved.

Control File:   /home/alemd/oracle/dept.ctl
Data File:      /home/alemd/oracle/dept.csv
  Bad File:     /home/alemd/oracle/dept.bad
  Discard File:  none specified
 
 (Allow all discards)

Number to load: ALL
Number to skip: 1
Errors allowed: 50
Bind array:     250 rows, maximum of 1048576 bytes
Continuation:    none specified
Path used:      Conventional

Table DEPT, loaded from every logical record.
Insert option in effect for this table: INSERT
TRAILING NULLCOLS option in effect

   Column Name                  Position   Len  Term Encl Datatype
------------------------------ ---------- ----- ---- ---- ---------------------
DEPTNO                              FIRST     *   ,  O(") CHARACTER            
DNAME                                NEXT     *   ,  O(") CHARACTER            
LOC                                  NEXT     *   ,  O(") CHARACTER            


Table DEPT:
  4 Rows successfully loaded.
  0 Rows not loaded due to data errors.
  0 Rows not loaded because all WHEN clauses were failed.
  0 Rows not loaded because all fields were null.


Space allocated for bind array:                 193500 bytes(250 rows)
Read   buffer bytes: 1048576

Total logical records skipped:          1
Total logical records read:             4
Total logical records rejected:         0
Total logical records discarded:        0

Run began on Mon Feb 27 00:15:41 2023
Run ended on Mon Feb 27 00:15:41 2023

Elapsed time was:     00:00:00.10
CPU time was:         00:00:00.03

```

Este log indica que en la tabla dept hemos tenido 4 filas cargadas correctamente. Ahora vemos el log de la tabla EMP:

```txt
SQL*Loader: Release 19.0.0.0.0 - Production on Wed Mar 1 10:39:05 2023
Version 19.3.0.0.0

Copyright (c) 1982, 2019, Oracle and/or its affiliates.  All rights reserved.

Control File:   /home/alemd/oracle/emp.ctl
Data File:      /home/alemd/oracle/emp.csv
  Bad File:     /home/alemd/oracle/emp.bad
  Discard File:  none specified
 
 (Allow all discards)

Number to load: ALL
Number to skip: 1
Errors allowed: 50
Bind array:     250 rows, maximum of 1048576 bytes
Continuation:    none specified
Path used:      Conventional

Table EMP, loaded from every logical record.
Insert option in effect for this table: APPEND
TRAILING NULLCOLS option in effect

   Column Name                  Position   Len  Term Encl Datatype
------------------------------ ---------- ----- ---- ---- ---------------------
EMPNO                               FIRST     *   ,  O(") CHARACTER            
ENAME                                NEXT     *   ,  O(") CHARACTER            
JOB                                  NEXT     *   ,  O(") CHARACTER            
MGR                                  NEXT     *   ,  O(") CHARACTER            
HIREDATE                             NEXT     *   ,  O(") DATE YYYY-MM-DD HH24:MI:SS
SAL                                  NEXT     7   ,  O(") CHARACTER            
COMM                                 NEXT     7   ,  O(") CHARACTER            
DEPTNO                               NEXT     *   ,  O(") CHARACTER            


Table EMP:
  14 Rows successfully loaded.
  0 Rows not loaded due to data errors.
  0 Rows not loaded because all WHEN clauses were failed.
  0 Rows not loaded because all fields were null.


Space allocated for bind array:                 392000 bytes(250 rows)
Read   buffer bytes: 1048576

Total logical records skipped:          1
Total logical records read:            14
Total logical records rejected:         0
Total logical records discarded:        0

Run began on Wed Mar 01 10:39:05 2023
Run ended on Wed Mar 01 10:39:05 2023

Elapsed time was:     00:00:00.10
CPU time was:         00:00:00.03
```

Vemos también que se ha insertado correctamente. Por último muestro que se han insertado los registros correctamente:

```txt
SQL> SELECT * FROM DEPT;

    DEPTNO DNAME	  LOC
---------- -------------- -------------
	10 ACCOUNTING	  NEW YORK
	20 RESEARCH	  DALLAS
	30 SALES	  CHICAGO
	40 OPERATIONS	  BOSTON

SQL> SELECT * FROM EMP;

     EMPNO ENAME      JOB	       MGR HIREDATE	   SAL	     COMM
---------- ---------- --------- ---------- -------- ---------- ----------
    DEPTNO
----------
      7839 KING       PRESIDENT 	 1 17/11/81	  5000		0
	10

      7698 BLAKE      MANAGER	      7839 01/05/81	  2850		0
	30

      7782 CLARK      MANAGER	      7839 09/06/81	  2450		0
	10


     EMPNO ENAME      JOB	       MGR HIREDATE	   SAL	     COMM
---------- ---------- --------- ---------- -------- ---------- ----------
    DEPTNO
----------
      7566 JONES      MANAGER	      7839 02/04/81	  2975		0
	20

      7788 SCOTT      ANALYST	      7566 19/04/01	  3000		0
	20

      7902 FORD       ANALYST	      7566 03/12/81	  3000		0
	20


     EMPNO ENAME      JOB	       MGR HIREDATE	   SAL	     COMM
---------- ---------- --------- ---------- -------- ---------- ----------
    DEPTNO
----------
      7369 SMITH      CLERK	      7902 17/12/80	   800		0
	20

      7499 ALLEN      SALESMAN	      7698 20/02/81	  1600	      300
	30

      7521 WARD       SALESMAN	      7698 22/02/81	  1250	      500
	30


     EMPNO ENAME      JOB	       MGR HIREDATE	   SAL	     COMM
---------- ---------- --------- ---------- -------- ---------- ----------
    DEPTNO
----------
      7654 MARTIN     SALESMAN	      7698 28/09/81	  1250	     1400
	30

      7844 TURNER     SALESMAN	      7698 08/09/81	  1500		0
	30

      7876 ADAMS      CLERK	      7788 23/05/01	  1100		0
	20


     EMPNO ENAME      JOB	       MGR HIREDATE	   SAL	     COMM
---------- ---------- --------- ---------- -------- ---------- ----------
    DEPTNO
----------
      7900 JAMES      CLERK	      7698 03/12/81	   950		0
	30

      7934 MILLER     CLERK	      7782 23/01/82	  1300		0
	10


14 filas seleccionadas.

```

El último ejercicio ha sido realizado con la ayuda de mi compañera María Jesús.

---
## **Documento realizado por:**

 ✒️ **Alejandro Montes Delgado** - *2º ASIR*

