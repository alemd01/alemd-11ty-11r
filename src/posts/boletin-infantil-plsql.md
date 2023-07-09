---
layout: post
title: "Boletín individual PL/SQL"
excerpt: "En esta práctica vamos a realizar un boletín de ejercicios de PLSQL."
date: 2022-12-04
tags:
  - post
  - GBD
---

### Hacer un procedimiento que muestre el nombre y el salario del empleado cuyo código es 7082

```plsql
CREATE OR REPLACE PROCEDURE MUESTRANOMBREYSALARIO
IS
    V_ENAME EMP.ENAME%TYPE;
    V_SAL EMP.SAL%TYPE;
BEGIN
    SELECT ENAME, SAL INTO V_ENAME, V_SAL
    FROM EMP WHERE EMPNO = 7782;
    
    DBMS_OUTPUT.PUT_LINE('Nombre: '|| v_ename);
    DBMS_OUTPUT.PUT_LINE('Salario: '|| v_sal);
END;
/

```

![BD-P3.1.png](/img/BD-P3.1.png)

### Hacer un procedimiento que reciba como parámetro un código de empleado y devuelva su nombre

```plsql
CREATE OR REPLACE PROCEDURE MUESTRA_NOMBRE (P_EMPNO EMP.EMPNO%TYPE)
IS
    V_NOMBRE EMP.ENAME%TYPE;
BEGIN
    SELECT ENAME INTO V_NOMBRE 
    FROM EMP WHERE EMPNO=P_EMPNO;

    DBMS_OUTPUT.PUT_LINE('Nombre: '|| V_NOMBRE);
END;
/
```
![BD-P3.2.png](/img/BD-P3.2.png)

### Hacer un procedimiento que devuelva los nombres de los tres empleados más antiguos

```plsql
CREATE OR REPLACE PROCEDURE TRES_MAS_ANTIGUOS
AS
BEGIN
  FOR emp IN (SELECT ENAME
              FROM EMP
              ORDER BY HIREDATE ASC
              FETCH FIRST 3 ROWS ONLY)
  LOOP
    DBMS_OUTPUT.PUT_LINE(emp.ENAME);
  END LOOP;
END;
/
```
![BD-P3.3.png](/img/BD-P3.3.png)

### Hacer un procedimiento que reciba el nombre de un tablespace y muestre los nombres de los usuarios que lo tienen como tablespace por defecto (Vista DBA_USERS)

```plsql
CREATE OR REPLACE PROCEDURE MUESTRA_TABLESPACE (P_TABLESPACE VARCHAR2)
IS
    CURSOR C_NOMBRE IS
    SELECT USERNAME
    FROM DBA_USERS
    WHERE DEFAULT_TABLESPACE=P_TABLESPACE;
BEGIN
    FOR R_NAME IN C_NOMBRE
    LOOP
        DBMS_OUTPUT.PUT_LINE(CHR(9)||'Nombre: '|| R_NAME.USERNAME);
    END LOOP;
END;
/
```
![BD-P3.4.png](/img/BD-P3.4.png)

### Modificar el procedimiento anterior para que haga lo mismo pero devolviendo el número de usuarios que tienen ese tablespace como tablespace por defecto. Nota: Hay que convertir el procedimiento en función

```plsql

CREATE OR REPLACE FUNCTION F_CUENTA_USUARIOS (P_TABLESPACE VARCHAR2)
 RETURN NUMBER
IS
  V_RESULTADO NUMBER;
BEGIN
  SELECT COUNT(USERNAME) INTO V_RESULTADO
  FROM DBA_USERS WHERE DEFAULT_TABLESPACE=P_TABLESPACE;
  RETURN V_RESULTADO;
END;
/
```
![BD-P3.5.png](/img/BD-P3.5.png)

### Hacer un procedimiento llamado mostrar_usuarios_por_tablespace que muestre por pantalla un listado de los tablespaces existentes con la lista de usuarios de cada uno y el número de los mismos, así: (Vistas DBA_TABLESPACES y DBA_USERS)

```plsql
CREATE OR REPLACE PROCEDURE MOSTRAR_USUARIOS_POR_TABLESPACE
IS
  CURSOR C_TABLESPACE IS
  SELECT TABLESPACE_NAME 
  FROM DBA_TABLESPACES;
  V_CONT NUMBER;
  V_CONT_TOTAL NUMBER :=0;
BEGIN
  FOR R_TABLESPACE IN C_TABLESPACE
  LOOP
    DBMS_OUTPUT.PUT_LINE(' Tablespace: '||R_TABLESPACE.TABLESPACE_NAME);
    MUESTRA_TABLESPACE(R_TABLESPACE.TABLESPACE_NAME);  
    V_CONT := F_CUENTA_USUARIOS(R_TABLESPACE.TABLESPACE_NAME);
    V_CONT_TOTAL := V_CONT_TOTAL+V_CONT;
    DBMS_OUTPUT.PUT_LINE('El total de usuarios en el Tablespace es: '||V_CONT);
  END LOOP;
  DBMS_OUTPUT.PUT_LINE(chr(10)||'El total de usuarios en la base de datos es: '||V_CONT_TOTAL);
END;
/
```
![BD-P3.6.png](/img/BD-P3.6.png)

![BD-P3.6.2.png](/img/BD-P3.6.2.png)


### Hacer un procedimiento llamado mostrar_codigo_fuente  que reciba el nombre de otro procedimiento y muestre su código fuente. (DBA_SOURCE)

Lo he hecho de 2 formas distintas:

Si usamos una consulta con la tabla DBA_SOURCES lo he hecho así.

```plsql
CREATE OR REPLACE PROCEDURE MUESTRA_CODIGO_FUENTE (P_NOMBREPROCEDURE VARCHAR2)
IS
    --CURSOR DONDE ALMACENAMOS EL PROCEDIMIENTO
    CURSOR C_NOMBREPROCEDURE IS
    SELECT TEXT
    FROM DBA_SOURCE
    WHERE NAME=P_NOMBREPROCEDURE;
BEGIN
    --RECORREMOS EL CURSOR PARA MOSTRAR EL PROCEDIMIENTO
    FOR R_NAME IN C_NOMBREPROCEDURE
    LOOP
        DBMS_OUTPUT.PUT_LINE(R_NAME.TEXT);
    END LOOP;
END;
/
```

Muestra el código sin formato y con un salto de línea entre sí. He tenido que usar un cursor porque el código lo dividía en filas y no lo almacenaba en una sola.

Para mostrarlo con formato he usado este procedimiento:

```plsql
PROCEDURE MOSTRAR_CODIGO_FUENTE (P_PROCEDIMIENTO VARCHAR2)
IS
  -- VARIABLE DONDE SE GUARDA EL CODIGO FUENTE
  V_CODIGOFUENTE CLOB;
BEGIN
  -- OBTIENE EL CODIGO FUENTE DEL PROCEDIMIENTO:
  SELECT DBMS_METADATA.GET_DDL('PROCEDURE', P_PROCEDIMIENTO)
  INTO V_CODIGOFUENTE 
  FROM DUAL;
  DBMS_OUTPUT.PUT_LINE(V_CODIGOFUENTE);
END;
/
```
![BD-P3.7.png](/img/BD-P3.7.png)


### Hacer un procedimiento llamado mostrar_privilegios_usuario que reciba el nombre de un usuario y muestre sus privilegios de sistema y sus privilegios sobre objetos. (DBA_SYS_PRIVS y DBA_TAB_PRIVS)

```plsql
CREATE OR REPLACE FUNCTION SYSTEM_PRIVILEGES (P_USERNAME VARCHAR2)
RETURN VARCHAR2
IS
  -- LA VARIABLE TIENE COMO TIPO DE DATO CLOB PORQUE ES UNA CADENA DE CARACTERES MUY GRANDE LO QUE PUEDE LLEGAR A ALMACENAR.
  V_PRIVILEGIOS CLOB;
  -- CREAMOS UN CURSOR QUE ALMACENA LOS PRIVILEGIOS DE LOS USUARIOS SEGÚN EL USUARIO QUE HEMOS PASADO POR PARÁMETRO.
  CURSOR C_PRIVILEGIOS IS
  SELECT PRIVILEGE
  FROM DBA_SYS_PRIVS
  WHERE GRANTEE=P_USERNAME;
BEGIN
  --EN EL FOR LO QUE HACEMOS ES GUARDAR EN LA VARIABLE LOS PRIVILEGIOS SEPARADOS POR UNA COMA.
  FOR R_SYS IN C_PRIVILEGIOS
  LOOP
    V_PRIVILEGIOS := V_PRIVILEGIOS|| R_SYS.PRIVILEGE || ', ';
  END LOOP;
  -- CON EL RTRIM QUITAMOS LA ÚLTIMA COMA Y ESPACIO.
  V_PRIVILEGIOS := RTRIM(V_PRIVILEGIOS, ', ');
  RETURN V_PRIVILEGIOS;
END;
/

-- ESTA FUNCION NO LA COMENTO YA QUE ES LA MISMA QUE LA ANTERIOR CAMBIANDO LA CONSULTA.
CREATE OR REPLACE FUNCTION TABLES_PRIVILEGES (P_USERNAME VARCHAR2)
RETURN VARCHAR2
IS
  V_PRIVILEGIOS CLOB;
  CURSOR C_PRIVILEGIOS IS
  SELECT PRIVILEGE
  FROM DBA_TAB_PRIVS
  WHERE GRANTOR=P_USERNAME;
BEGIN
  FOR R_TAB IN C_PRIVILEGIOS
  LOOP
    V_PRIVILEGIOS := V_PRIVILEGIOS|| R_TAB.PRIVILEGE || ', ';
  END LOOP;
  V_PRIVILEGIOS := RTRIM(V_PRIVILEGIOS, ', ');
  RETURN V_PRIVILEGIOS;
END;
/

-- ESTE PROCEDIMIENTO ES MUY SENCILLO, ALMACENAMOS LAS FUNCIONES EN VARIABLES Y LAS MOSTRAMOS POR PANTALLA.
CREATE OR REPLACE PROCEDURE MOSTRAR_PRIVILEGIOS_USUARIO (P_USERNAME VARCHAR2)
IS
  V_SYS_PRIV CLOB;
  V_TAB_PRIV CLOB;
BEGIN
  V_SYS_PRIV := SYSTEM_PRIVILEGES(P_USERNAME);
  V_TAB_PRIV := TABLES_PRIVILEGES(P_USERNAME);
  DBMS_OUTPUT.PUT_LINE('Los privilegios de sistema del usuario '|| P_USERNAME ||' son: '|| V_SYS_PRIV);
  DBMS_OUTPUT.PUT_LINE('Los privilegios sobre objetos del usuario '|| P_USERNAME ||' son: '|| V_TAB_PRIV);
END;
/
```

![BD-P3.8.png](/img/BD-P3.8.png)


### Realiza un procedimiento llamado listar_comisiones que nos muestre por pantalla un listado de las comisiones de los empleados agrupados según la localidad donde está ubicado su departamento con el siguiente formato:

SIN TERMINAR. NO FUNCIONA.

TOTAL DE COMISIONES POR LOCALIDAD:

TOTAL DE COMISIONES POR DEPARTAMENTO:
SELECT DNAME, NVL(SUM(COMM),0) FROM EMP, DEPT WHERE EMP.DEPTNO=DEPT.DEPTNO AND DEPT.DEPTNO=20 GROUP BY DNAME;

TOTAL DE COMISIONES DE EMPLEADOS:
SELECT ENAME, NVL(COMM,0) AS COMISION FROM EMP WHERE DEPTNO=30 ORDER BY ENAME ASC;


```plsql
--TERMINAR
CREATE OR REPLACE PROCEDURE LISTAR_COMISIONES
IS
BEGIN
  LISTAR_DEPARTAMENTOS;
  LISTAR_LOCALIDAD;
  LISTAR_COMISION;
END;
/

CREATE OR REPLACE PROCEDURE LISTAR_DEPARTAMENTOS (P_DEPTNO OUT DEPT.DEPTNO%TYPE)
IS
  CURSOR C_DEPT IS
  SELECT DNAME, DEPTNO FROM DEPT
  ORDER BY LOC ASC;
BEGIN
  FOR R_DEPT IN C_DEPT LOOP
    DBMS_OUTPUT.PUT_LINE('DEPARTAMENTO: '||R_DEPT.DNAME);
    P_DEPTNO:=R_DEPT.DEPTNO;
    LISTAR_LOCALIDAD(P_DEPTNO);
  END LOOP;
END;
/

CREATE OR REPLACE PROCEDURE LISTAR_LOCALIDAD (P_DEPTNO DEPT.DEPTNO%TYPE)
IS
  CURSOR C_LOC IS
  SELECT LOC FROM DEPT
  ORDER BY LOC ASC;
BEGIN
  FOR R_LOC IN C_LOC LOOP
    DBMS_OUTPUT.PUT_LINE('LOCALIDAD: '||R_LOC.LOC);
    LISTAR_COMISION(P_DEPTNO);
  END LOOP;
END;
/

CREATE OR REPLACE PROCEDURE LISTAR_COMISION (P_DEPTNO DEPT.DEPTNO%TYPE)
IS
  CURSOR C_COMM IS
  SELECT ENAME, NVL(COMM,0) AS "COMISION"
  FROM EMP WHERE DEPTNO=P_DEPTNO
  ORDER BY ENAME ASC;
BEGIN
  FOR R_COMM IN C_COMM LOOP
    DBMS_OUTPUT.PUT_LINE(CHR(9)||R_COMM.ENAME||' ------------- '||R_COMM.COMISION);
  END LOOP;
END;
/
```

Nota: Los nombres de localidades, departamentos y empleados deben aparecer por orden alfabético.

Si alguno de los departamentos no tiene ningún empleado con comisiones, aparecerá un mensaje informando de ello en lugar de la lista de empleados.

El procedimiento debe gestionar adecuadamente las siguientes excepciones:

    a) La tabla Empleados está vacía.
    b) Alguna comisión es mayor que 10000.

### Realiza un procedimiento que reciba el nombre de una tabla y muestre los nombres de las restricciones que tiene, a qué columna afectan y en qué consisten exactamente. (DBA_TABLES, DBA_CONSTRAINTS, DBA_CONS_COLUMNS)

```plsql
CREATE OR REPLACE PROCEDURE MUESTRA_NOMBRE_RESTRICCIONES (P_TABLA VARCHAR2)
IS
  CURSOR C_NOMBRE IS
  SELECT CONSTRAINT_NAME 
  FROM DBA_CONSTRAINTS 
  WHERE TABLE_NAME=P_TABLA;
  V_COLUMNA VARCHAR2(100);
  V_TIPO_CONSTRAINT VARCHAR2(10);
BEGIN
  DBMS_OUTPUT.PUT_LINE('El nombre de la tabla es: '||P_TABLA);
  FOR R_CONST IN C_NOMBRE
  LOOP
    DBMS_OUTPUT.PUT_LINE('El nombre de la restriccion es: '||R_CONST.CONSTRAINT_NAME);
    SELECT COLUMN_NAME INTO V_COLUMNA FROM DBA_CONS_COLUMNS WHERE TABLE_NAME=P_TABLA AND CONSTRAINT_NAME=R_CONST.CONSTRAINT_NAME;
    DBMS_OUTPUT.PUT_LINE('La columna a la que afecta la restriccion es: '||V_COLUMNA);
    SELECT CONSTRAINT_TYPE INTO V_TIPO_CONSTRAINT FROM DBA_CONSTRAINTS WHERE TABLE_NAME=P_TABLA AND CONSTRAINT_NAME=R_CONST.CONSTRAINT_NAME;
    EXPLICA_CONSTRAINT(V_TIPO_CONSTRAINT);
  END LOOP;
END;
/

CREATE OR REPLACE PROCEDURE EXPLICA_CONSTRAINT (P_TIPO_CONSTRAINT VARCHAR2)
IS

BEGIN
  CASE
    WHEN P_TIPO_CONSTRAINT = 'P' THEN
      DBMS_OUTPUT.PUT_LINE('La restriccion es una clave primaria.'||CHR(10));
    WHEN P_TIPO_CONSTRAINT = 'C' THEN
      DBMS_OUTPUT.PUT_LINE('La restriccion es tipo CHECK y lo que hace es validar.'||CHR(10));
    WHEN P_TIPO_CONSTRAINT = 'R' THEN
      DBMS_OUTPUT.PUT_LINE('La restriccion es una clave foranea o ajena.'||CHR(10));
  END CASE;
END;
/
```

![BD-P3.9.png](/img/BD-P3.9.png)


### Realiza al menos dos de los ejercicios anteriores en Postgres usando PL/pgSQL.

#### Primer ejercicio

```plsql
CREATE OR REPLACE PROCEDURE MUESTRANOMBREYSALARIO()
AS $$
DECLARE
    V_ENAME EMP.ENAME%TYPE;
    V_SAL EMP.SAL%TYPE;
BEGIN
    SELECT ENAME, SAL INTO V_ENAME, V_SAL
    FROM EMP WHERE EMPNO = 7782;
    
    RAISE NOTICE '%','Nombre: '|| v_ename;
    RAISE NOTICE '%','Salario: '|| v_sal;
END;
$$ LANGUAGE plpgsql;
```
![BD-P3.10.png](/img/BD-P3.10.png)


#### Segundo ejercicio

```plsql
CREATE OR REPLACE PROCEDURE MUESTRA_NOMBRE (P_EMPNO EMP.EMPNO%TYPE)
AS $$
DECLARE
    V_NOMBRE EMP.ENAME%TYPE;
BEGIN
    SELECT ENAME INTO V_NOMBRE 
    FROM EMP WHERE EMPNO=P_EMPNO;

    RAISE NOTICE '%','Nombre: '|| V_NOMBRE;
END;
$$ LANGUAGE plpgsql;
```

![BD-P3.11.png](/img/BD-P3.11.png)


---
## **Documento realizado por:**

 ✒️ **Alejandro Montes Delgado** - *2º ASIR*
