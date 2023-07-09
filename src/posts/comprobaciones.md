## Funcionamiento de los ejercicios

### 3. Añade dos columnas llamadas Distancia y Duración (en formato hh:mi) a la tabla Vuelos y rellénalas adecuadamente. Realiza un trigger que garantice que un avión asignado a un viaje tenga una autonomía superior al menos en un 25% a la distancia a recorrer.

**Pruebas con el update:**

* Update que la autonomía del avión es superior en un 25% superior a la distancia:
```sql
UPDATE VUELOS SET DISTANCIA=800 WHERE CODVUELO='IB-5940';
```
![BD-AMD.1.png](/img/BD-AMD.1.png)

* Update que la autonomía del avión es menor del 25% de la distancia a recorrer.
```sql
UPDATE VUELOS SET DISTANCIA=5000 WHERE CODVUELO='IB-5940';
```

![BD-AMD.2.png](/img/BD-AMD.2.png)

**Pruebas con el insert:**
* Insert que la autonomía del avión es superior en un 25% superior a la distancia:
```sql
INSERT INTO VUELOS VALUES (
    'UX-6013',
    'PMI',
    'MAD',
    'Air Europa',
    'Jueves',
    TO_DATE('10:15', 'HH24:MI'),
    'EJK000333',
    'HYJ000568',
    782,
    TO_DATE('01:30','HH24:MI')
);
```
![BD-AMD.4.png](/img/BD-AMD.4.png)



* Insert que la autonomía del avión es menor del 25% de la distancia a recorrer.
```sql
INSERT INTO VUELOS VALUES (
    'UX-6013',
    'PMI',
    'MAD',
    'Air Europa',
    'Jueves',
    TO_DATE('10:15', 'HH24:MI'),
    'EJK000333',
    'HYJ000568',
    7820,
    TO_DATE('01:30','HH24:MI')
);
```

![BD-AMD.3.png](/img/BD-AMD.3.png)

### 7. Realiza los módulos de programación necesarios para garantizar que una misma compañía no tiene más de dos vuelos con el mismo origen y destino en un mismo día de la semana.

Inserto un vuelo con el mismo origen y destino y en el mismo día de la semana.

```sql
INSERT INTO VUELOS VALUES (
    'CD-6015',
    'PMI',
    'MAD',
    'Air Europa',
    'Jueves',
    TO_DATE('14:15', 'HH24:MI'),
    'CVF000126',
    'FHY000456',
    782,
    TO_DATE('01:30','HH24:MI')
);
```

![BD-AMD.5.png](/img/BD-AMD.5.png)


Si insertamos otro vuelo con el mismo origen y destino y en el mismo día de la semana, se dispara el trigger.

```sql
INSERT INTO VUELOS VALUES (
    'CD-6017',
    'PMI',
    'MAD',
    'Air Europa',
    'Jueves',
    TO_DATE('14:15', 'HH24:MI'),
    'CVF000126',
    'FHY000456',
    782,
    TO_DATE('01:30','HH24:MI')
);
```

![BD-AMD.6.png](/img/BD-AMD.6.png)

El trigger no funciona bien, ya que al eliminar un vuelo no lo detecta y si se vuelve a insertar salta el trigger cuando no debería de hacerlo.

### 8. Realiza los módulos de programación necesarios para garantizar que el número de auxiliares de vuelo en un viaje es de al menos uno por cada 5 pasajeros que realizan ese viaje.

Vamos a ingresar pasajeros hasta que salte la excepción. No dejo los inserts aquí ya que el código sería muy largo, todos los inserts se encuentran en el archivo del ejercicio 8. Dejo aquí el resultado de hacer todos los inserts.

![BD-AMD.7.png](img/BD-AMD.7.png)


He tenido problemas con la tabla temporal, para que se rellene tenemos que actualizar la tabla auxiliares de viaje para que se rellene


### 3. (pl/pgsql) Añade dos columnas llamadas Distancia y Duración (en formato hh:mi) a la tabla Vuelos y rellénalas adecuadamente. Realiza un trigger que garantice que un avión asignado a un viaje tenga una autonomía superior al menos en un 25% a la distancia a recorrer.

El primer insert funciona correctamente, en el segundo salta el error.

```sql
 INSERT INTO VUELOS VALUES (
     'UX-6013',
     'PMI',
     'MAD',
     'Air Europa',
     'Jueves',
     TO_TIMESTAMP('10:15', 'HH24:MI'),
     'EJK000333',
     'HYJ000568',
     TO_TIMESTAMP('01:30','HH24:MI'),
      782
 );

```
Este es el insert el cual hace que se dispare el trigger.
```sql
 INSERT INTO VUELOS VALUES (
     'UX-6013',
     'PMI',
     'MAD',
     'Air Europa',
     'Jueves',
     TO_TIMESTAMP('10:15', 'HH24:MI'),
     'EJK000333',
     'HYJ000568',
     TO_TIMESTAMP('01:30','HH24:MI'),
      7820
 );
```

![BD-AMD.8.png](img/BD-AMD.8.png)
