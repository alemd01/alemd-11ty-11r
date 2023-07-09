---
layout: post
title: Informática forense.
excerpt: Tercer taller de la unidad de Docker.
date: 2023-02-07
updatedDate: 2023-02-07
tags:
  - post
  - SAD
---

La informática forense es el conjunto de técnicas que nos permite obtener la máxima información posible tras un incidente o delito informático.

En esta práctica, realizarás la fase de toma de evidencias y análisis de las mismas sobre una máquina Linux y otra Windows. Supondremos que pillamos al delincuente in fraganti y las máquinas se encontraban encendidas. Opcionalmente, podéis realizar el análisis de un dispositivo Android.

Sobre cada una de las máquinas debes realizar un volcado de memoria y otro de disco duro, tomando las medidas necesarias para certificar posteriormente la cadena de custodia.

Debes tratar de obtener las siguientes informaciones:

### Máquina Windows.

Lo primero que haremos será un volcado de memoria y un volcado de disco sobre los que trabajaremos. Para hacer un volcado de memoria en Windows 7, usaré una herramienta llamada DumpIt y la ejecutaremos:

![SAD-P7.12.png](/img/SAD-P7.12.png)

Pasado unos minutos nos genera la siguiente imagen:

![SAD-P7.13.png](/img/SAD-P7.13.png)

Para analizar esta imagen, usaremos la herramienta Volatility, la instalaremos en nuestro equipo debian, para ello tendremos que instalar python and pip si no lo tenemos instalado y con pip instalaremos distorm3.

```txt
alemd@debian:~$ sudo pip install distorm3
Collecting distorm3
  Downloading distorm3-3.5.2.tar.gz (138 kB)
     |████████████████████████████████| 138 kB 3.3 MB/s 
Building wheels for collected packages: distorm3
  Building wheel for distorm3 (setup.py) ... done
  Created wheel for distorm3: filename=distorm3-3.5.2-cp39-cp39-linux_x86_64.whl size=126327 sha256=9f9b53cd375a137982810e36b7c0caf1b1f6b4f1f96e2627b211e08a4c43917e
  Stored in directory: /root/.cache/pip/wheels/eb/3a/b1/929df023c1328f8c1f94de335b110b1f2347aafa1a55edbf99
Successfully built distorm3
Installing collected packages: distorm3
Successfully installed distorm3-3.5.2

```

También necesitaremos git para copiar el repositorio de volatility.

```txt
alemd@debian:~/2ASIR/SAD/volatility$ git clone https://github.com/volatilityfoundation/volatility3.git
Clonando en 'volatility3'...
remote: Enumerating objects: 30094, done.
remote: Counting objects: 100% (938/938), done.
remote: Compressing objects: 100% (463/463), done.
remote: Total 30094 (delta 561), reused 805 (delta 469), pack-reused 29156
Recibiendo objetos: 100% (30094/30094), 6.07 MiB | 6.70 MiB/s, listo.
Resolviendo deltas: 100% (22742/22742), listo.

```

Ahora me copiaré el dump a mi máquina para trabajar sobre él.



**Por comandos:**

#### 1. Procesos en ejecución.

Usando volatility con un dumpeo de un Windows 7, listo los procesos activos a la hora de hacer el dump:

```txt
python3 vol.py -f USUARIO-PC-20230207-201216.raw windows.pslist.PsList
```

![SAD-P7.14.png](/img/SAD-P7.14.png)

Ahora realizo un escaneo de los procesos:

![SAD-P7.15.png](/img/SAD-P7.15.png)

La diferencia entre psscan y pslist es que pslist muestra todos los procesos que hay en ERPROCESS y psscan busca en memoria procesos ocultos, por eso psscan tarda bastante más.


#### 2. Servicios en ejecución.

Usando volatility y un dump de Windows 7, la sintaxis del comando es la siguiente:

```txt
python3 vol.py -f USUARIO-PC-20230207-201216.raw windows.getservicesids.GetServiceSIDs
```

![SAD-P7.16.png](/img/SAD-P7.16.png)


#### 3. Puertos abiertos.

El comando para ver los puertos abiertos es el siguiente:

```txt
python3 vol.py -f USUARIO-PC-20230207-201216.raw windows.netscan.NetScan
```

![SAD-P7.20.png](/img/SAD-P7.20.png)

#### 4. Conexiones establecidas por la máquina.

Con este comando vemos las conexiones establecidas por la máquina:

```txt
python3 vol.py -f USUARIO-PC-20230207-201216.raw windows.netscan.NetScan
```

![SAD-P7.19.png](/img/SAD-P7.19.png)


#### 5. Sesiones de usuario establecidas remotamente.

El comando es el siguiente:

```txt
python3 vol.py -f USUARIO-PC-20230207-201216.raw windows.sessions.Sessions
```

![SAD-P7.18.png](/img/SAD-P7.18.png)


#### 6. Ficheros transferidos recientemente por NetBios.

#### 7. Contenido de la caché DNS.

#### 8. Variables de entorno.

Para ver las variables de entorno he usado volatility y el dump de Windows 7:

```txt
 python3 vol.py -f USUARIO-PC-20230207-201216.raw windows.envars.Envars
```

![SAD-P7.17.png](/img/SAD-P7.17.png)


**Analizando el Registro de Windows:**

#### 9. Dispositivos USB conectados

#### 10. Redes wifi utilizadas recientemente.

#### 11. Configuración del firewall de nodo.

#### 12. Programas que se ejecutan en el Inicio.

#### 13. Asociación de extensiones de ficheros y aplicaciones.

En el apartado file views, desplegamos file types y by extension y nos aparecen todos los archivos ordenados por carpetas y extensión:

![SAD-P7.20.png](/img/SAD-P7.20.png)


#### 14. Aplicaciones usadas recientemente.

Nos vamos al apartado de Data Artifacts y Run Programs. Como vemos en la captura el último programa ejecutado es el FTK imager que es para hacer la imagen del disco:

![SAD-P7.22.png](/img/SAD-P7.22.png)


#### 15. Ficheros abiertos recientemente.

Nuevamente en el apartado Data Artifacts, y en Recent Documents:

![SAD-P7.23.png](/img/SAD-P7.23.png)


#### 16. Software Instalado.

Seguimos en el apartado Data Artifacts y en Installed Programs:

![SAD-P7.24.png](/img/SAD-P7.24.png)


#### 17. Contraseñas guardadas.



#### 18. Cuentas de Usuario

En este apartado, aparecen todas las cuentas de usuario del sistema y sus contraseñas. El apartado es OS account.

![SAD-P7.24.png](/img/SAD-P7.24.png)

**Con Aplicaciones de terceros:**

#### 19. Historial de navegación y descargas. Cookies.

* --> Data Artifacts --> web search. Veremos las búsquedas realizadas en el navegador.

![SAD-P7.26.png](/img/SAD-P7.26.png)

* --> Data Artifacts --> web History. Aparece el historial.

![SAD-P7.27.png](/img/SAD-P7.27.png)

* --> Data Artifacts --> web Download. Aparecen las descargas realizadas.

![SAD-P7.28.png](/img/SAD-P7.28.png)

* --> Data Artifacts --> web Cookies. Aparecen las cookies guardadas en el navegador.

![SAD-P7.29.png](/img/SAD-P7.29.png)

* --> Data Artifacts --> web Bookmarks. Aparecen los marcadores del navegador.

![SAD-P7.30.png](/img/SAD-P7.30.png)


#### 20. Volúmenes cifrados

**Sobre la imagen del disco:**

#### 21. Archivos con extensión cambiada.

Para verlo vamos a la siguiente ruta --> Analysis Results --> Extension Mismatch Detected.

![SAD-P7.31.png](/img/SAD-P7.31.png)

#### 22. Archivos eliminados.

--> Deleted Files --> File System

![SAD-P7.32.png](/img/SAD-P7.32.png)

#### 23. Archivos Ocultos.



#### 24. Archivos que contienen una cadena determinada.

Arriba a la derecha, tenemos la opción de Keyword Search.

![SAD-P7.33.png](/img/SAD-P7.33.png)


#### 25. Búsqueda de imágenes por ubicación.

En la barra de herramientas, usamos la opción Geolocation.

![SAD-P7.34.png](/img/SAD-P7.34.png)


#### 26. Búsqueda de archivos por autor.

--> Data Artifacts --> Metadata.

![SAD-P7.35.png](/img/SAD-P7.35.png)


### Máquina Linux.

**Intenta realizar las mismas operaciones en una máquina Linux para aquellos apartados que tengan sentido y no se realicen de manera idéntica a Windows.**

---
## **Documento realizado por:**

 ✒️ **Alejandro Montes Delgado** - *2º ASIR*