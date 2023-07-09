---
layout: post
title: Servidores Web, Base de Datos y DNS en nuestros escenario de OpenStack 
excerpt: En esta práctica vamos a configurar un servidor DNS, Web y de bases de datos en nuestro escenario de OpenStack.
date: 2023-01-14
updatedDate: 2023-01-14
tags:
  - post
  - SRI
---


### Entrega la configuración DNS de cada máquina.

Para que la configuración sea persistente en todas las máquinas tendremos que hacer lo siguiente:

```txt
usuario@alfa:~$ sudo nano /usr/lib/systemd/resolv.conf 
```

En este fichero, añadiremos la configuración dns para que al reiniciar se mantenga.

* #### Alfa.

```txt
usuario@alfa:~$ cat /etc/resolv.conf

nameserver 192.168.0.2
search alejandro-montes.gonzalonazareno.org
nameserver 172.16.202.2
search openstacklocal
```

* #### Bravo.

```txt
[usuario@bravo ~]$ cat /etc/resolv.conf 

nameserver 192.168.0.2
search alejandro-montes.gonzalonazareno.org
nameserver 172.16.202.2
```

* #### Charlie.

```txt
ubuntu@charlie:~$ cat /etc/resolv.conf 

nameserver 192.168.0.2
search alejandro-montes.gonzalonazareno.org
nameserver 127.0.0.53
options edns0 trust-ad

```

* #### Delta.

```txt
ubuntu@delta:~$ cat /etc/resolv.conf 

nameserver 192.168.0.2
search alejandro-montes.gonzalonazareno.org
nameserver 127.0.0.53
options edns0 trust-ad

```

### Entrega la definición de las vistas y de las zonas.

* #### Configuración de las vistas.

```txt
view externa {
                match-clients { 172.22.0.0/16; 172.29.0.0/16; 192.168.202.2/32; };
                allow-recursion { any; };
                zone "alejandro-montes.gonzalonazareno.org"
                {
                        type master;
                        file "db.externa.alejandro-montesgonzalonazareno.org";
                };
                include "/etc/bind/zones.rfc1918";
                include "/etc/bind/named.conf.default-zones";
};
view dmz {
                match-clients { 172.16.0.0/16; };
                allow-recursion { any; };
                zone "alejandro-montes.gonzalonazareno.org"
                {
                        type master;
                        file "db.dmz.alejandro-montes.gonzalonazareno.org";
                };
                zone "16.172.in-addr.arpa"
                {
                        type master;
                        file "db.16.172";
                };
                zone "0.168.192.in-addr.arpa"
                {
                        type master;
                        file "db.0.168.192";
                };
                include "/etc/bind/zones.rfc1918";
                include "/etc/bind/named.conf.default-zones";
};
view interna {
                match-clients { 192.168.0.0/24; 127.0.0.1; };
                allow-recursion { any; };
                zone "alejandro-montes.gonzalonazareno.org"
                {
                        type master;
                        file "db.interna.alejandro-montes.gonzalonazareno.org";
                };
                zone "0.168.192.in-addr.arpa"
                {
                        type master;
                        file "db.0.168.192";
                };
                zone "16.172.in-addr.arpa"
                {
                        type master;
                        file "db.16.172";
                };
                include "/etc/bind/zones.rfc1918";
                include "/etc/bind/named.conf.default-zones";
};
```

* #### Configuración de la zona externa.

```txt
$TTL    86400
@       IN      SOA     alfa.alejandro-montes.gonzalonazareno.org. root.alejandro-montes.gonzalonazareno.org. (
                              1         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                          86400 )       ; Negative Cache TTL
;
@       IN      NS      alfa.alejandro-montes.gonzalonazareno.org.

$ORIGIN alejandro-montes.gonzalonazareno.org.

alfa                    IN      A       172.22.200.13
dns                     IN      CNAME   alfa
www                     IN      CNAME   alfa

```

* #### Configuración de la zona dmz.

```txt
$TTL    86400
@       IN      SOA     charlie.alejandro-montes.gonzalonazareno.org. root.charlie.alejandro-montes.gonzalonazareno.org. (
                              1         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                          86400 )       ; Negative Cache TTL
;
@       IN      NS      charlie.alejandro-montes.gonzalonazareno.org.

$ORIGIN alejandro-montes.gonzalonazareno.org.

alfa                    IN      A       172.16.0.1
bravo                   IN      A       172.16.0.200
charlie                 IN      A       192.168.0.2
delta                   IN      A       192.168.0.3
www                     IN      CNAME   bravo
bd                      IN      CNAME   delta
dns                     IN      CNAME   charlie
```

* #### Configuración de la zona Interna.

```txt
$TTL    86400
@       IN      SOA     charlie.alejandro-montes.gonzalonazareno.org. root.charlie.alejandro-montes.gonzalonazareno.org. (
                              1         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                          86400 )       ; Negative Cache TTL
;
@       IN      NS      charlie.alejandro-montes.gonzalonazareno.org.

$ORIGIN alejandro-montes.gonzalonazareno.org.

alfa                    IN      A       192.168.0.1
bravo                   IN      A       172.16.0.200
charlie                 IN      A       192.168.0.2
delta                   IN      A       192.168.0.3
www                     IN      CNAME   bravo
bd                      IN      CNAME   delta
dns                     IN      CNAME   charlie

```

* #### Configuración de la zona DMZ inversa.

```txt
$TTL    86400
@       IN      SOA     charlie.alejandro-montes.gonzalonazareno.org. root.alejandro-montes.gonzalonazareno.org. (
                              1         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                          86400 )       ; Negative Cache TTL
;
@       IN      NS              charlie.alejandro-montes.gonzalonazareno.org.

$ORIGIN 16.172.in-addr.arpa.

200.0                   IN      PTR             bravo.alejandro-montes.gonzalonazareno.org.
```

* #### Configuración de la zona interna inversa.

```txt
$TTL    86400
@       IN      SOA     charlie.alejandro-montes.gonzalonazareno.org. root.alejandro-montes.gonzalonazareno.org. (
                              1         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                          86400 )       ; Negative Cache TTL
;
@       IN      NS              charlie.alejandro-montes.gonzalonazareno.org.

$ORIGIN 0.168.192.in-addr.arpa.

1                       IN      PTR             alfa.alejandro-montes.gonzalonazareno.org.
2                       IN      PTR             charlie.alejandro-montes.gonzalonazareno.org.
3                       IN      PTR             delta.alejandro-montes.gonzalonazareno.org.
```

### Entrega el resultado de las siguientes consultas desde una máquina interna a nuestra red y otro externo:

* #### El servidor DNS con autoridad sobre la zona del dominio tu_nombre.gonzalonazareno.org.

El comando es el siguiente:

```txt
dig ns alejandro-montes.gonzalonazareno.org
```

Desde alfa(Red Interna):

![SRI-P7.4.png](/img/SRI-P7.4.png)

Desde bravo(Red DMZ):

![SRI-P7.2.png](/img/SRI-P7.2.png)

Desde mi portátil:

![SRI-P7.3.png](/img/SRI-P7.3.png)

* #### La dirección IP de alfa.

El comando es el siguiente:

 ```txt
dig alfa.alejandro-montes.gonzalonazareno.org
 ```

Desde alfa(Red Interna):

![SRI-P7.5.png](/img/SRI-P7.5.png)

Desde bravo(Red DMZ):

![SRI-P7.6.png](/img/SRI-P7.6.png)

Desde mi portátil:

![SRI-P7.7.png](/img/SRI-P7.7.png)

* #### Una resolución de www.

El comando es el siguiente:

```txt
dig www.alejandro-montes.gonzalonazareno.org
```

Desde alfa(Red Interna):

![SRI-P7.8.png](/img/SRI-P7.8.png)

Desde bravo(Red DMZ):

![SRI-P7.9.png](/img/SRI-P7.9.png)

Desde mi portátil:

![SRI-P7.10.png](/img/SRI-P7.10.png)

* #### Una resolución de bd.

El comando es el siguiente:

```txt
dig bd.alejandro-montes.gonzalonazareno.org
```

Desde alfa(Red Interna):

![SRI-P7.11.png](/img/SRI-P7.11.png)

Desde bravo(Red DMZ):

![SRI-P7.12.png](/img/SRI-P7.12.png)

Desde mi portátil:

![SRI-P7.13.png](/img/SRI-P7.13.png)

* #### Un resolución inversa de IP fija en cada una de las redes. (Esta consulta sólo funcionará desde una máquina interna).

El comando es el siguiente:

```txt
dig -x 0.0.0.0
```

Desde alfa(Red Interna) a la red interna:

![SRI-P7.14.png](/img/SRI-P7.14.png)

Desde alfa(Red Interna) a la red dmz:

![SRI-P7.15.png](/img/SRI-P7.15.png)

Desde bravo(Red DMZ) a la red interna:

![SRI-P7.16.png](/img/SRI-P7.16.png)

Desde bravo(Red DMZ) a la red dmz:

![SRI-P7.17.png](/img/SRI-P7.17.png)


### Entrega una captura de pantalla accediendo a www.tunombre.gonzalonazareno.org/info.php donde se vea la salida del fichero info.php.

![SRI-P7.18.png](/img/SRI-P7.18.png)

### Entrega una prueba de funcionamiento donde se vea como se realiza una conexión a la base de datos desde bravo.

Desde Bravo, conecto al servidor de bases de datos usando el CNAME `bd` de delta:

![SRI-P7.1.png](/img/SRI-P7.1.png)

---
## **Documento realizado por:**

 ✒️ **Alejandro Montes Delgado** - *2º ASIR*