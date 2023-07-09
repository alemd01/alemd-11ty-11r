### Práctica 8: Movimiento de Datos.

Lo primero que haré será crear una base de datos donde insertaré algunas tablas con los caracteres indicados:

```txt
examen=# CREATE TABLE PRUEBA (
NOMBRE VARCHAR(30),
APELLIDO VARCHAR(30),
FECHA_NACIMIENTO DATE,
SALARIO NUMERIC(8) 
);                                     
CREATE TABLE
examen=# INSERT INTO PRUEBA VALUES('Juan','Perez','2000-05-15',5000);
INSERT 0 1
examen=# INSERT INTO PRUEBA VALUES('Manolo','Diaz','1999-03-11',5000);
INSERT 0 1
examen=# INSERT INTO PRUEBA VALUES('Paco','Fernandez','1999-03-11',1000);
INSERT 0 1
examen=# select * from prueba;
 nombre | apellido  | fecha_nacimiento | salario 
--------+-----------+------------------+---------
 Juan   | Perez     | 2000-05-15       |    5000
 Manolo | Diaz      | 1999-03-11       |    5000
 Paco   | Fernandez | 1999-03-11       |    1000
(3 filas)

```

Ahora con esta función la ejecutamos y nos creará el csv correspondiente de la tabla:

```txt
examen=# CREATE OR REPLACE FUNCTION exportar_tablas_a_csv(
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
            'COPY %I TO %L WITH (FORMAT CSV, DELIMITER ";", HEADER)',
            _nombre_de_la_tabla,
            _ruta || _nombre_de_la_tabla || '.csv'
        );
    END LOOP;
END;
$$ LANGUAGE plpgsql;
CREATE FUNCTION
```

ejecutamos la función:

```txt
examen=# SELECT exportar_tablas_a_csv('prueba', '/var/lib/postgresql/');
 exportar_tablas_a_csv 
-----------------------
 
(1 fila)

```

Muestro el archivo que se ha creado:

```txt
nombre;apellido;fecha_nacimiento;salario
Juan;Perez;2000-05-15;5000
Manolo;Diaz;1999-03-11;5000
Paco;Fernandez;1999-03-11;1000
```

Ahora me paso este csv a nuestro servidor oracle y lo cargaremos con sqlloader. Creo la tabla en oracle donde vamos a cargar los datos:

```txt
SQL> CREATE TABLE prueba (
  2  NOMBRE VARCHAR2(30),
  3  APELLIDO VARCHAR2(30),
  4  FECHA_NACIMIENTO DATE,
  5  SALARIO NUMBER(8)
  6  );

Tabla creada.

```

Muestro el fichero de control que vamos a usar:

```txt
OPTIONS (SKIP=1)
LOAD DATA
INFILE '/home/usuario/oracle/prueba.csv'
INTO TABLE prueba
FIELDS TERMINATED BY ';' OPTIONALLY ENCLOSED BY '"'
TRAILING NULLCOLS
(NOMBRE, APELLIDO, FECHA_NACIMIENTO DATE "YYYY-MM-DD", SALARIO)

```


Muestro la salida del comando:

```txt
usuario@oracle:~/oracle$ sqlldr c##servidor2/usuario control=/home/usuario/oracle/prueba.ctl log=/home/usuario/oracle/prueba.log

SQL*Loader: Release 19.0.0.0.0 - Production on Tue Mar 7 11:17:37 2023
Version 19.3.0.0.0

Copyright (c) 1982, 2019, Oracle and/or its affiliates.  All rights reserved.

Path used:      Conventional
Commit point reached - logical record count 3

Table PRUEBA:
  3 Rows successfully loaded.

Check the log file:
  /home/usuario/oracle/prueba.log
for more information about the load.

```

Como vemos se han añadido 3 filas correctamente, a continuación consulto la tabla:

```txt

SQL> SELECT * FROM prueba;

NOMBRE			       APELLIDO 		      FECHA_NA
------------------------------ ------------------------------ --------
   SALARIO
----------
Juan			       Perez			      15/05/00
      5000

Manolo			       Diaz			      11/03/99
      5000

Paco			       Fernandez		      11/03/99
      1000

```

### Práctica 9: Copias de seguridad y Restauración.

Creamos un tablespace y una tabla que contenga ese tablespace:

```txt
SQL> CREATE TABLESPACE EXAMEN_TS DATAFILE 'examen_ts.dbf' SIZE 5M AUTOEXTEND ON;

Tablespace creado.

SQL> CREATE TABLE examen (
  2  NOMBRE VARCHAR2(30),
  3  APELLIDO VARCHAR2(30),
  4  FECHA_NACIMIENTO DATE,
  5  SALARIO NUMBER(8)
  6  ) TABLESPACE EXAMEN_TS;

Tabla creada.

```

Inserto registros en las tablas:

```txt

SQL> INSERT INTO examen VALUES ('Juan','Perez',TO_DATE('2000-05-15','YYYY-MM-DD'), 5000);

1 fila creada.

SQL> INSERT INTO examen VALUES ('Manolo','Diaz',TO_DATE('1999-03-11','YYYY-MM-DD'), 5000);

1 fila creada.

SQL> INSERT INTO examen VALUES ('Paco','Fernandez',TO_DATE('1999-03-11','YYYY-MM-DD'), 1000);

1 fila creada.

```

Ahora consultamos donde se encuentra el tablespace creado:

```txt
SQL> SELECT FILE_NAME FROM DBA_DATA_FILES WHERE TABLESPACE_NAME = 'TS_EXAMEN';

FILE_NAME
--------------------------------------------------------------------------------
/opt/oracle/product/19c/dbhome_1/dbs/examen_ts.dbf

```

Ahora nos conectamos con RMAN y realizamos el backup en caliente para ello primero registramos la base de datos y después ejecutamos el backup:

```txt
RMAN> REGISTER DATABASE;

database registered in recovery catalog
starting full resync of recovery catalog
full resync complete

RMAN> BACKUP DATABASE;

Starting backup at 07-MAR-23
allocated channel: ORA_DISK_1
channel ORA_DISK_1: SID=269 device type=DISK
channel ORA_DISK_1: starting full datafile backup set
channel ORA_DISK_1: specifying datafile(s) in backup set
input datafile file number=00001 name=/opt/oracle/oradata/ORCLCDB/system01.dbf
input datafile file number=00003 name=/opt/oracle/oradata/ORCLCDB/sysaux01.dbf
input datafile file number=00014 name=/opt/oracle/oradata/ORCLCDB/TS_RAMN.dbf
input datafile file number=00004 name=/opt/oracle/oradata/ORCLCDB/undotbs01.dbf
input datafile file number=00007 name=/opt/oracle/oradata/ORCLCDB/users01.dbf
input datafile file number=00013 name=/opt/oracle/product/19c/dbhome_1/dbs/examen_ts.dbf
channel ORA_DISK_1: starting piece 1 at 07-MAR-23
channel ORA_DISK_1: finished piece 1 at 07-MAR-23
piece handle=/opt/oracle/product/19c/dbhome_1/dbs/021mel7j_1_1 tag=TAG20230307T114346 comment=NONE
channel ORA_DISK_1: backup set complete, elapsed time: 00:01:57
channel ORA_DISK_1: starting full datafile backup set
channel ORA_DISK_1: specifying datafile(s) in backup set
input datafile file number=00010 name=/opt/oracle/oradata/ORCLCDB/ORCLPDB1/sysaux01.dbf
input datafile file number=00009 name=/opt/oracle/oradata/ORCLCDB/ORCLPDB1/system01.dbf
input datafile file number=00011 name=/opt/oracle/oradata/ORCLCDB/ORCLPDB1/undotbs01.dbf
input datafile file number=00012 name=/opt/oracle/oradata/ORCLCDB/ORCLPDB1/users01.dbf
channel ORA_DISK_1: starting piece 1 at 07-MAR-23
channel ORA_DISK_1: finished piece 1 at 07-MAR-23
piece handle=/opt/oracle/product/19c/dbhome_1/dbs/041melb9_1_1 tag=TAG20230307T114346 comment=NONE
channel ORA_DISK_1: backup set complete, elapsed time: 00:00:39
channel ORA_DISK_1: starting full datafile backup set
channel ORA_DISK_1: specifying datafile(s) in backup set
input datafile file number=00006 name=/opt/oracle/oradata/ORCLCDB/pdbseed/sysaux01.dbf
input datafile file number=00005 name=/opt/oracle/oradata/ORCLCDB/pdbseed/system01.dbf
input datafile file number=00008 name=/opt/oracle/oradata/ORCLCDB/pdbseed/undotbs01.dbf
channel ORA_DISK_1: starting piece 1 at 07-MAR-23
channel ORA_DISK_1: finished piece 1 at 07-MAR-23
piece handle=/opt/oracle/product/19c/dbhome_1/dbs/051melcg_1_1 tag=TAG20230307T114346 comment=NONE
channel ORA_DISK_1: backup set complete, elapsed time: 00:00:35
Finished backup at 07-MAR-23

Starting Control File and SPFILE Autobackup at 07-MAR-23
piece handle=/opt/oracle/product/19c/dbhome_1/dbs/c-2889820145-20230307-02 comment=NONE
Finished Control File and SPFILE Autobackup at 07-MAR-23

```

Borramos el tablespace:

```txt
root@oracle:/opt/oracle/oradata/ORCLCDB# rm ts_examen.dbf 

```

Desactivamos el tablespace:

```txt
RMAN> SQL "ALTER TABLESPACE TS_EXAMEN OFFLINE IMMEDIATE";

sql statement: ALTER TABLESPACE TS_EXAMEN OFFLINE IMMEDIATE
```

Restauramos el tablespace:

```txt
RMAN> RESTORE TABLESPACE TS_EXAMEN;

Starting restore at 07-MAR-23
using channel ORA_DISK_1

channel ORA_DISK_1: starting datafile backup set restore
channel ORA_DISK_1: specifying datafile(s) to restore from backup set
channel ORA_DISK_1: restoring datafile 00015 to /opt/oracle/oradata/ORCLCDB/ts_examen.dbf
channel ORA_DISK_1: reading from backup piece /opt/oracle/product/19c/dbhome_1/dbs/081menn7_1_1
channel ORA_DISK_1: piece handle=/opt/oracle/product/19c/dbhome_1/dbs/081menn7_1_1 tag=TAG20230307T122614
channel ORA_DISK_1: restored backup piece 1
channel ORA_DISK_1: restore complete, elapsed time: 00:00:03
Finished restore at 07-MAR-23

RMAN> 
```

Y lo recuperamos:

```txt
RMAN> RECOVER TABLESPACE TS_EXAMEN;

Starting recover at 07-MAR-23
using channel ORA_DISK_1

starting media recovery
media recovery complete, elapsed time: 00:00:01

Finished recover at 07-MAR-23

```

Volvemos a poner online:

```txt
RMAN> SQL "ALTER TABLESPACE TS_EXAMEN ONLINE";

sql statement: ALTER TABLESPACE TS_EXAMEN ONLINE

RMAN> 
```

Probamos que funciona:

```txt
usuario@oracle:~$ sqlplus

SQL*Plus: Release 19.0.0.0.0 - Production on Tue Mar 7 12:35:52 2023
Version 19.3.0.0.0

Copyright (c) 1982, 2019, Oracle.  All rights reserved.

Enter user-name: usuario
Enter password: 
Hora de Ultima Conexion Correcta: Mar Mar 07 2023 12:29:59 +01:00

Conectado a:
Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production
Version 19.3.0.0.0

SQL> SELECT * FROM examen;

NOMBRE			       APELLIDO 		      FECHA_NA
------------------------------ ------------------------------ --------
   SALARIO
----------
Juan			       Perez			      15/05/00
      5000

Manolo			       Diaz			      11/03/99
      5000

Paco			       Fernandez		      11/03/99
      1000


```