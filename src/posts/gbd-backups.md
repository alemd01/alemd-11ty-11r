---
layout: post
title: "Copias de seguridad en distintos SGBD."
excerpt: "En esta práctica veremos las copias de seguridad enfocadas en las bases de datos."
date: 2023-03-01
tags:
  - post
  - GBD
---

### Realiza una copia de seguridad lógica de tu base de datos completa, teniendo en cuenta los siguientes requisitos:

* **La copia debe estar encriptada y comprimida.**
* **Debe realizarse en un conjunto de ficheros con un tamaño máximo de 60 MB.**
* **Programa la operación para que se repita cada día a una hora determinada.**

Para la correcta resolución del ejercicio, he creado un trabajo en el scheduler de oracle que se ejecuta todos los días y realiza una copia de seguridad compimida y encriptada y de máximo de 60MB, para ello, he tenido que crear un procedimiento que crea el trabajo:

```txt
BEGIN
DBMS_SCHEDULER.create_job (
job_name => 'Export_db',
job_type => 'executable',
job_action => 'expdp SYSTEM/usuario directory=BD_EXPORT dumpfile=bkdb_$(date +%d-%m-%Y).dmp encryption=all compression=all filesize=60M encryption_password=usuario',
start_date => SYSDATE,
repeat_interval => 'FREQ=DAILY; BYHOUR=13; BYMINUTE=10',
end_date => NULL,
enabled => TRUE,
comments => 'export_db_DAILY');
END;
/
```

Este procedimiento ejecuta el comando que se encuentra en `job_action` empezando hoy con una frecuencia diaria indicando que la hora del backup es a las 13 y 10. A continuación muestro una consulta de la tabla donde se almacenan todos los trabajos del scheduler que muestra el nombre del trabajo y el intervalo de repetición:

```txt
SQL> SELECT JOB_NAME, REPEAT_INTERVAL FROM ALL_SCHEDULER_JOBS WHERE JOB_NAME = 'EXPORT_DB';

JOB_NAME
--------------------------------------------------------------------------------
REPEAT_INTERVAL
--------------------------------------------------------------------------------
EXPORT_DB
FREQ=DAILY; BYHOUR=13; BYMINUTE=10
```

No funciona, he estado probando pero no se ejecuta todos los días el scheduler. Adjunto el comando que si funciona correctamente y tiene el resto de características:

```txt
expdp SYSTEM/usuario directory=BD_EXPORT dumpfile=bkdb_$(date +%d-%m-%Y).dmp encryption=all compression=all filesize=60M encryption_password=usuario'
```

### Restaura la copia de seguridad lógica creada en el punto anterior.

### Pon tu base de datos en modo ArchiveLog y realiza con RMAN una copia de seguridad física en caliente.

Primero comprobaremos que nuestra base de datos no está en modo archive log, para ello ejecutaremos el siguiente comando:

```txt
SQL> ARCHIVE LOG LIST
Modo log de la base de datos		  Modo de No Archivado
Archivado automatico		 Desactivado
Destino del archivo	       /opt/oracle/product/19c/dbhome_1/dbs/arch
Secuencia de log en linea mas antigua	  139
Secuencia de log actual 	  141
```

Como nos indica el modo log no está activado. Para activarlo tenemos que apagar la base de datos, arrancarla en modo mount, activar el modo archivelog y abrir la base de datos. A continuación dejo la secuencia de comandos completa:

```txt
SQL> shutdown immediate
Base de datos cerrada.
Base de datos desmontada.
Instancia ORACLE cerrada.
SQL> startup mount;
Instancia ORACLE iniciada.

Total System Global Area 3573545736 bytes
Fixed Size		    9141000 bytes
Variable Size		  973078528 bytes
Database Buffers	 2583691264 bytes
Redo Buffers		    7634944 bytes
Base de datos montada.
SQL> alter database archivelog;

Base de datos modificada.

SQL> alter database open;

Base de datos modificada.

```

Ahora volvemos a comprobar que se ha activado correctamente el modo archivelog:

```txt
SQL> archive log list
Modo log de la base de datos		  Modo de Archivado
Archivado automatico		 Activado
Destino del archivo	       /opt/oracle/product/19c/dbhome_1/dbs/arch
Secuencia de log en linea mas antigua	  139
Siguiente secuencia de log para archivar   141
Secuencia de log actual 	  141
```

Ahora procederemos a crear un catálogo de RMAN, con esta estrategia crearemos un repositorio de información, un tablespace con un usuario y haremos que toda la información para gestionar copias se guarde allí y la maneje el usuario creado. Ahora crearemos el tablespace y el usuario:

```txt
SQL> CREATE TABLESPACE TS_RMAN DATAFILE '/opt/oracle/oradata/ORCLCDB/TS_RAMN.dbf' SIZE 300M;

Tablespace creado.

SQL> CREATE USER USUARIO_RMAN IDENTIFIED BY usuario DEFAULT TABLESPACE TS_RMAN QUOTA UNLIMITED ON TS_RMAN;

Usuario creado.

```

Le damos privilegios al usuario creado:

```txt
SQL> GRANT RECOVERY_CATALOG_OWNER TO USUARIO_RMAN;

Concesion terminada correctamente.

SQL> GRANT CONNECT, RESOURCE TO USUARIO_RMAN;

Concesion terminada correctamente.


```

Ahora nos conectamos con el usuario que hemos creado a RMAN:

```txt
alemd@debian:~$ rman

Recovery Manager: Release 19.0.0.0.0 - Production on Fri Mar 3 17:05:19 2023
Version 19.3.0.0.0

Copyright (c) 1982, 2019, Oracle and/or its affiliates.  All rights reserved.

RMAN> CONNECT CATALOG USUARIO_RMAN

recovery catalog database Password: 
connected to recovery catalog database

RMAN> 


```

Creamos un catálogo en el tablespace que hemos creado anteriormente:

```txt
RMAN> CREATE CATALOG TABLESPACE TS_RMAN;

recovery catalog created

```

Ahora nos saldremos y nos conectaremos con nuestro usuario:

```txt
alemd@debian:~$ rman target=/ catalog USUARIO_RMAN

Recovery Manager: Release 19.0.0.0.0 - Production on Fri Mar 3 17:10:37 2023
Version 19.3.0.0.0

Copyright (c) 1982, 2019, Oracle and/or its affiliates.  All rights reserved.

connected to target database: ORCLCDB (DBID=2889830663)
recovery catalog database Password: 
connected to recovery catalog database

RMAN> 

```

Ahora registramos nuestra base de datos:

```txt
RMAN> REGISTER DATABASE;

database registered in recovery catalog
starting full resync of recovery catalog
full resync complete

```

Ya tendríamos nuestra base de datos registrada y preparada. Ahora nos queda lanzar una copia de seguridad:

```txt
RMAN> BACKUP DATABASE;

Starting backup at 03-MAR-23
allocated channel: ORA_DISK_1
channel ORA_DISK_1: SID=147 device type=DISK
channel ORA_DISK_1: starting full datafile backup set
channel ORA_DISK_1: specifying datafile(s) in backup set
input datafile file number=00004 name=/opt/oracle/oradata/ORCLCDB/undotbs01.dbf
input datafile file number=00007 name=/opt/oracle/oradata/ORCLCDB/users01.dbf
input datafile file number=00003 name=/opt/oracle/oradata/ORCLCDB/sysaux01.dbf
input datafile file number=00001 name=/opt/oracle/oradata/ORCLCDB/system01.dbf
input datafile file number=00019 name=/opt/oracle/oradata/ORCLCDB/TS_RAMN.dbf
input datafile file number=00018 name=/opt/oracle/product/19c/dbhome_1/dbs/index.dbf
input datafile file number=00017 name=/opt/oracle/product/19c/dbhome_1/dbs/ts1.dbf
input datafile file number=00014 name=/opt/oracle/oradata/ORCLCDB/ts2.dbf
input datafile file number=00015 name=/opt/oracle/oradata/ts2.dbf
channel ORA_DISK_1: starting piece 1 at 03-MAR-23
channel ORA_DISK_1: finished piece 1 at 03-MAR-23
piece handle=/opt/oracle/product/19c/dbhome_1/dbs/021m4n1b_1_1 tag=TAG20230303T171314 comment=NONE
channel ORA_DISK_1: backup set complete, elapsed time: 00:00:35
channel ORA_DISK_1: starting full datafile backup set
channel ORA_DISK_1: specifying datafile(s) in backup set
input datafile file number=00010 name=/opt/oracle/oradata/ORCLCDB/ORCLPDB1/sysaux01.dbf
input datafile file number=00009 name=/opt/oracle/oradata/ORCLCDB/ORCLPDB1/system01.dbf
input datafile file number=00011 name=/opt/oracle/oradata/ORCLCDB/ORCLPDB1/undotbs01.dbf
input datafile file number=00012 name=/opt/oracle/oradata/ORCLCDB/ORCLPDB1/users01.dbf
channel ORA_DISK_1: starting piece 1 at 03-MAR-23
channel ORA_DISK_1: finished piece 1 at 03-MAR-23
piece handle=/opt/oracle/product/19c/dbhome_1/dbs/031m4n2f_1_1 tag=TAG20230303T171314 comment=NONE
channel ORA_DISK_1: backup set complete, elapsed time: 00:00:07
channel ORA_DISK_1: starting full datafile backup set
channel ORA_DISK_1: specifying datafile(s) in backup set
input datafile file number=00006 name=/opt/oracle/oradata/ORCLCDB/pdbseed/sysaux01.dbf
input datafile file number=00005 name=/opt/oracle/oradata/ORCLCDB/pdbseed/system01.dbf
input datafile file number=00008 name=/opt/oracle/oradata/ORCLCDB/pdbseed/undotbs01.dbf
channel ORA_DISK_1: starting piece 1 at 03-MAR-23
channel ORA_DISK_1: finished piece 1 at 03-MAR-23
piece handle=/opt/oracle/product/19c/dbhome_1/dbs/041m4n2m_1_1 tag=TAG20230303T171314 comment=NONE
channel ORA_DISK_1: backup set complete, elapsed time: 00:00:07
Finished backup at 03-MAR-23

Starting Control File and SPFILE Autobackup at 03-MAR-23
piece handle=/opt/oracle/product/19c/dbhome_1/dbs/c-2889830663-20230303-01 comment=NONE
Finished Control File and SPFILE Autobackup at 03-MAR-23


```

Y una vez finalizado ya habríamos hecho la copia de seguridad en caliente con RMAN.

### Borra un fichero de datos de un tablespace e intenta recuperar la instancia de la base de datos a partir de la copia de seguridad creada en el punto anterior.

He decidido borrar el fichero del tablespace USERS, para ello ejecutaré una consulta que me mostrará la ruta del fichero de dicho tablespace:

```txt
SQL> SELECT FILE_NAME FROM DBA_DATA_FILES WHERE TABLESPACE_NAME = 'USERS';

FILE_NAME
--------------------------------------------------------------------------------
/opt/oracle/oradata/ORCLCDB/users01.dbf


```

En nuestra terminal, como root, ejecutamos lo siguiente:

```txt
root@debian:~# rm /opt/oracle/oradata/ORCLCDB/users01.dbf
root@debian:~# ls /opt/oracle/oradata/ORCLCDB/
control01.ctl  pdbseed	   redo03.log	 temp01.dbf   undotbs01.dbf
control02.ctl  redo01.log  sysaux01.dbf  ts2.dbf
ORCLPDB1       redo02.log  system01.dbf  TS_RAMN.dbf
```


Ya hemos borrado el fichero del tablespace, para ello me conectaré a un usuario con ese tablespace y veremos que da error:

```txt
SQL> select * from emp;
select * from emp
              *
ERROR en linea 1:
ORA-01116: error al abrir el archivo de base de datos 7
ORA-01110: archivo de datos 7: '/opt/oracle/oradata/ORCLCDB/users01.dbf'
ORA-27041: no se ha podido abrir el archivo
Linux-x86_64 Error: 2: No such file or directory
Additional information: 3

```

Ahora nos conectamos con nuestro usuario a RMAN:

```txt
alemd@debian:~$ rman target=/ catalog USUARIO_RMAN

Recovery Manager: Release 19.0.0.0.0 - Production on Fri Mar 3 17:56:57 2023
Version 19.3.0.0.0

Copyright (c) 1982, 2019, Oracle and/or its affiliates.  All rights reserved.

connected to target database: ORCLCDB (DBID=2889830663)
recovery catalog database Password: 
connected to recovery catalog database

RMAN> 

```

Desactivamos el tablespace:

```txt
RMAN> SQL "ALTER TABLESPACE USERS OFFLINE IMMEDIATE";

sql statement: ALTER TABLESPACE USERS OFFLINE IMMEDIATE

```

Ahora, restauramos el fichero:

```txt
RMAN> RESTORE TABLESPACE USERS;

Starting restore at 03-MAR-23
allocated channel: ORA_DISK_1
channel ORA_DISK_1: SID=261 device type=DISK

channel ORA_DISK_1: starting datafile backup set restore
channel ORA_DISK_1: specifying datafile(s) to restore from backup set
channel ORA_DISK_1: restoring datafile 00007 to /opt/oracle/oradata/ORCLCDB/users01.dbf
channel ORA_DISK_1: reading from backup piece /opt/oracle/product/19c/dbhome_1/dbs/021m4n1b_1_1
channel ORA_DISK_1: piece handle=/opt/oracle/product/19c/dbhome_1/dbs/021m4n1b_1_1 tag=TAG20230303T171314
channel ORA_DISK_1: restored backup piece 1
channel ORA_DISK_1: restore complete, elapsed time: 00:00:15
Finished restore at 03-MAR-23

```

Lo recuperamos y lo volvemos a activar:

```txt
RMAN> RECOVER TABLESPACE USERS;

Starting recover at 03-MAR-23
using channel ORA_DISK_1

starting media recovery
media recovery complete, elapsed time: 00:00:00

Finished recover at 03-MAR-23

RMAN> SQL "ALTER TABLESPACE USERS ONLINE";

sql statement: ALTER TABLESPACE USERS ONLINE

```

Para verificar que se ha realizado correctamente, me conectaré de nuevo al usuario SCOTT y realizaré la misma consulta:

```txt
alemd@debian:~$ sqlplus SCOTT/TIGER

SQL*Plus: Release 19.0.0.0.0 - Production on Fri Mar 3 18:01:22 2023
Version 19.3.0.0.0

Copyright (c) 1982, 2019, Oracle.  All rights reserved.

Hora de Ultima Conexion Correcta: Vie Mar 03 2023 17:55:53 +01:00

Conectado a:
Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production
Version 19.3.0.0.0

SQL> SELECT * FROM DEPT;

    DEPTNO DNAME	  LOC
---------- -------------- -------------
	10 ACCOUNTING	  NEW YORK
	20 RESEARCH	  DALLAS
	30 SALES	  CHICAGO
	40 OPERATIONS	  BOSTON

```


### Borra un fichero de control e intenta recuperar la base de datos a partir de la copia de seguridad creada en el punto anterior.

Ahora borraremos un fichero de control y procederemos a restaurar la base de datos completa:

```txt
root@debian:/opt/oracle/oradata/ORCLCDB# rm control01.ctl 
root@debian:/opt/oracle/oradata/ORCLCDB# rm control02.ctl 
```

Forzamos el apagado:

```txt
SQL> shutdown abort
Instancia ORACLE cerrada.
```

Le ponemos el modo no montada:

```txt
SQL> startup nomount
Instancia ORACLE iniciada.

Total System Global Area 3573545736 bytes
Fixed Size		    9141000 bytes
Variable Size		  973078528 bytes
Database Buffers	 2583691264 bytes
Redo Buffers		    7634944 bytes
```

Ahora entramos en RMAN:

```txt
alemd@debian:~$ rman target=/ catalog USUARIO_RMAN

Recovery Manager: Release 19.0.0.0.0 - Production on Sat Mar 4 11:52:40 2023
Version 19.3.0.0.0

Copyright (c) 1982, 2019, Oracle and/or its affiliates.  All rights reserved.

connected to target database: ORCLCDB (not mounted)
recovery catalog database Password: 
RMAN-00571: ===========================================================
RMAN-00569: =============== ERROR MESSAGE STACK FOLLOWS ===============
RMAN-00571: ===========================================================
RMAN-00554: initialization of internal recovery manager package failed
RMAN-04004: error from recovery catalog database: ORA-01033: inicializacion o cierre de ORACLE en curso
```

Como vemos se ha borrado el fichero de control donde se encontraba nuestro catálogo. Tenemos que decirle a RMAN que restaure los controlfile desde los ficheros de la copia de seguridad:

```txt
alemd@debian:~$ rman target=/

Recovery Manager: Release 19.0.0.0.0 - Production on Mon Mar 6 17:51:42 2023
Version 19.3.0.0.0

Copyright (c) 1982, 2019, Oracle and/or its affiliates.  All rights reserved.

connected to target database: ORCLCDB (DBID=2889830663, not open)

RMAN> RESTORE CONTROLFILE FROM '/opt/oracle/product/19c/dbhome_1/dbs/c-2889830663-20230303-00';

Starting restore at 06-MAR-23
using channel ORA_DISK_1

channel ORA_DISK_1: restoring control file
channel ORA_DISK_1: restore complete, elapsed time: 00:00:01
output file name=/opt/oracle/oradata/ORCLCDB/control01.ctl
output file name=/opt/oracle/oradata/ORCLCDB/control02.ctl
Finished restore at 06-MAR-23

RMAN> 
```

Restauramos la base de datos con el comando:

```txt
RESTORE DATABASE;
```

Y la recuperamos:

```txt
RECOVER DATABASE;
```

Por último, abrimos la base de datos reseteando los logs:

```txt
SQL> alter database open resetlogs;
Database altered.

```

Y ya estaría finalizada la recuperación. Me conecto con el usuario SCOTT y hago una breve consulta para mostrar que funciona:

```txt
SQL> SELECT * FROM DEPT;

    DEPTNO DNAME	  LOC
---------- -------------- -------------
	10 ACCOUNTING	  NEW YORK
	20 RESEARCH	  DALLAS
	30 SALES	  CHICAGO
	40 OPERATIONS	  BOSTON

```

### Documenta el empleo de las herramientas de copia de seguridad y restauración de Postgres.

Para realizar una copia de seguridad completa ejecutamos el siguiente comando:

```txt
postgres@PostgreSQL:~$ pg_dumpall -U postgres > backup_$(date +"%d-%m-%Y").sql
postgres@PostgreSQL:~$ ls
11	    backup_06-03-2023.sql  emp.csv	       oracle_fdw
aeropuerto  dept.csv		   instantclient_21_1  PG_11_201809051
```

Como vemos, con este comando se guarda el backup con la fecha en la que se realiza el backup. Con este comando podriamos automatizarlo con Cron para que periódicamente se realice una copia de seguridad. Para ello añadiremos la siguiente línea al cron:

```txt
00 10 * * 0 postgres pg_dumpall -U postgres > /var/lib/postgresql/backup/f
ullbackup_$(date +"%d-%m-%Y").sql
```

Esta línea indica que todos los días a las 22:15 el usuario postgres ejecutará la copia de seguridad y se guardará en la ruta indicada.

Para restaurar la base de datos completa usamos el siguiente comando:

```txt
psql -f pg_copia_seguridad.sql postgres
```

### Documenta el empleo de las herramientas de copia de seguridad y restauración de MySQL.

Para hacer un backup de las tablas de una base de datos usamos el siguiente comando:

```txt
root@debian:/home/usuario# mysqldump -u root SCOTT > backup.sql
```

Si queremos realizar un backup de todas las bases de datos usamos el siguiente comando:

```txt
root@debian:/home/usuario# mysqldump -u root --all-databases > backup.sql
```

Ahora simularemos la pérdida de datos en la tabla emp y borraremos todos los registros que tiene dicha tabla:

```txt
MariaDB [SCOTT]> DELETE FROM EMP;
Query OK, 14 rows affected (0,002 sec)
```

Muestro que se han borrado todos los registros:

```txt
MariaDB [SCOTT]> SELECT * FROM EMP;
Empty set (0,001 sec)

```

Ahora restauraremos la copia de seguridad:

```txt
root@debian:/home/usuario# mysql -u root SCOTT < backup.sql 
```

Si volvemos a consultar la tabla emp, veremos que se ha restaurado correctamente:

```txt
MariaDB [SCOTT]> SELECT * FROM EMP;
+-------+--------+-----------+------+---------------------+---------+---------+--------+
| EMPNO | ENAME  | JOB       | MGR  | HIREDATE            | SAL     | COMM    | DEPTNO |
+-------+--------+-----------+------+---------------------+---------+---------+--------+
|  7369 | SMITH  | CLERK     | 7902 | 1980-12-17 00:00:00 |  800.00 |    NULL |     20 |
|  7499 | ALLEN  | SALESMAN  | 7698 | 1981-02-20 00:00:00 | 1600.00 |  300.00 |     30 |
|  7521 | WARD   | SALESMAN  | 7698 | 1981-02-22 00:00:00 | 1250.00 |  500.00 |     30 |
|  7566 | JONES  | MANAGER   | 7839 | 1981-04-02 00:00:00 | 2975.00 |    NULL |     20 |
|  7654 | MARTIN | SALESMAN  | 7698 | 1981-09-28 00:00:00 | 1250.00 | 1400.00 |     30 |
|  7698 | BLAKE  | MANAGER   | 7839 | 1981-05-01 00:00:00 | 2850.00 |    NULL |     30 |
|  7782 | CLARK  | MANAGER   | 7839 | 1981-06-09 00:00:00 | 2450.00 |    NULL |     10 |
|  7788 | SCOTT  | ANALYST   | 7566 | 1982-12-09 00:00:00 | 3000.00 |    NULL |     20 |
|  7839 | KING   | PRESIDENT | NULL | 1981-11-17 00:00:00 | 5000.00 |    NULL |     10 |
|  7844 | TURNER | SALESMAN  | 7698 | 1981-09-08 00:00:00 | 1500.00 |    0.00 |     30 |
|  7876 | ADAMS  | CLERK     | 7788 | 1983-01-12 00:00:00 | 1100.00 |    NULL |     20 |
|  7900 | JAMES  | CLERK     | 7698 | 1981-12-03 00:00:00 |  950.00 |    NULL |     30 |
|  7902 | FORD   | ANALYST   | 7566 | 1981-12-03 00:00:00 | 3000.00 |    NULL |     20 |
|  7934 | MILLER | CLERK     | 7782 | 1982-01-23 00:00:00 | 1300.00 |    NULL |     10 |
+-------+--------+-----------+------+---------------------+---------+---------+--------+
14 rows in set (0,001 sec)

```

### Documenta el empleo de las herramientas de copia de seguridad y restauración de MongoDB.

Para realizar la copia de seguridad usaremos mongodump. El comando usado es el siguiente:

```txt
root@mongodb:/home/vagrant# mongodump --db prueba --out backup_restaurant -u alemd
Enter password for mongo user:

2023-03-06T21:37:53.960+0000	writing prueba.alumno to backup_restaurant/prueba/alumno.bson
2023-03-06T21:37:53.969+0000	writing prueba.prueba1 to backup_restaurant/prueba/prueba1.bson
2023-03-06T21:37:53.977+0000	writing prueba.prueba2 to backup_restaurant/prueba/prueba2.bson
2023-03-06T21:37:54.025+0000	writing prueba.restaurants to backup_restaurant/prueba/restaurants.bson
2023-03-06T21:37:54.039+0000	done dumping prueba.prueba1 (0 documents)
2023-03-06T21:37:54.039+0000	done dumping prueba.prueba2 (0 documents)
2023-03-06T21:37:54.040+0000	done dumping prueba.restaurants (822 documents)
2023-03-06T21:37:54.042+0000	done dumping prueba.alumno (3 documents)
```

Eliminaremos la colección restaurants que tiene 822 documentos:

```txt
Enterprise prueba> db.restaurants.drop();
true
Enterprise prueba> db.restaurants.find();


```

Ahora restauraremos la base de datos y comprobaremos que la colección está nuevamente:

```txt
root@mongodb:/home/vagrant# mongorestore --db prueba backup_restaurant/prueba/restaurants.bson -u alemd
Enter password for mongo user:

2023-03-06T21:46:42.758+0000	checking for collection data in backup_restaurant/prueba/restaurants.bson
2023-03-06T21:46:42.759+0000	reading metadata for prueba.restaurants from backup_restaurant/prueba/restaurants.metadata.json
2023-03-06T21:46:43.039+0000	restoring prueba.restaurants from backup_restaurant/prueba/restaurants.bson
2023-03-06T21:46:43.145+0000	finished restoring prueba.restaurants (822 documents, 0 failures)
2023-03-06T21:46:43.146+0000	no indexes to restore for collection prueba.restaurants
2023-03-06T21:46:43.146+0000	822 document(s) restored successfully. 0 document(s) failed to restore.

```

Comprobamos que se ha restaurado correctamente los documentos, aunque ya nos indica que 822 documentos han sido restaurados correctamente:

```txt
Enterprise prueba> db.restaurants.findOne();
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

Como vemos hemos hecho una consulta de un documento y lo muestra correctamente.

---
## **Documento realizado por:**

 ✒️ **Alejandro Montes Delgado** - *2º ASIR*
