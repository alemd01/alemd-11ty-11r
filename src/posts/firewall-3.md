---
layout: post
title: "Cortafuegos III: Perimetral sobre escenario."
excerpt: En este post veremos la tercera práctica de cortafuegos.
date: 2023-03-01
updatedDate: 2023-03-01
tags:
  - post
  - SAD
---

**Sobre el escenario creado en el módulo de servicios con las máquinas Alfa (Router), Bravo (DMZ), Charlie y Delta (LAN) y empleando iptables o nftables, configura un cortafuegos perimetral en la máquina Alfa de forma que el escenario siga funcionando completamente teniendo en cuenta los siguientes puntos:**
* **Política por defecto DROP para las cadenas INPUT, FORWARD y OUTPUT.**
* **Se pueden usar las extensiones que creamos adecuadas, pero al menos debe implementarse seguimiento de la conexión.**
* **Debemos implementar que el cortafuegos funcione después de un reinicio de la máquina.**
* **Debes indicar pruebas de funcionamiento de todas las reglas.**
* **El cortafuego debe cumplir al menos estas reglas:**

### La máquina Alfa tiene un servidor ssh escuchando por el puerto 22, pero al acceder desde el exterior habrá que conectar al puerto 2222.

Antes que nada permitiremos el tráfico ssh a alfa para que no se interrumpa la conexión al aplicar las políticas drop:

```txt
root@alfa:/home/usuario# iptables -t nat -A PREROUTING -p tcp --dport 2222 -i ens3 -j DNAT --to 10.0.0.148:22
root@alfa:/home/usuario# iptables -A INPUT -i ens3 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
root@alfa:/home/usuario# iptables -A OUTPUT -o ens3 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT
```

Pruebo que funciona la conexión ssh desde el exterior a través del puerto 22:

```txt
alemd@debian:~$ ssh -A usuario@172.22.200.13 -p 2222
Linux alfa 5.10.0-21-amd64 #1 SMP Debian 5.10.162-1 (2023-01-21) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sun Mar  5 09:20:39 2023 from 172.29.0.70
usuario@alfa:~$ 
```

Ahora pongo las políticas por defecto para input, output y forward:

```txt
root@alfa:/home/usuario# iptables -P INPUT DROP
root@alfa:/home/usuario# iptables -P OUTPUT DROP
root@alfa:/home/usuario# iptables -P FORWARD DROP
```

### Desde Delta y Bravo se debe permitir la conexión ssh por el puerto 22 a la máquina Alfa.

Primero para poder trabajar correctamente sobre el escenario, añado reglas para que desde alfa se pueda conectar por ssh tanto a la red interna(charlie y delta) como a la red dmz(bravo):

```txt
root@alfa:/home/usuario# iptables -A OUTPUT -d 192.168.0.0/24 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
root@alfa:/home/usuario# iptables -A INPUT -s 192.168.0.0/24 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT
root@alfa:/home/usuario# iptables -A OUTPUT -d 172.16.0.0/16 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
root@alfa:/home/usuario# iptables -A INPUT -s 172.16.0.0/16 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT
```

Ahora, añado las reglas para que delta pueda conectarse por ssh a alga:

```txt
root@alfa:/home/usuario# iptables -A OUTPUT -d 192.168.0.3/24 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT
root@alfa:/home/usuario# iptables -A INPUT -s 192.168.0.3/24 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
```

Lo mismo pero las reglas para Bravo:

```txt
root@alfa:/home/usuario# iptables -A OUTPUT -d 172.16.0.200/16 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT
root@alfa:/home/usuario# iptables -A INPUT -s 172.16.0.200/16 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
```

Realizo ssh desde delta a alfa:

```txt
ubuntu@delta:~$ ssh usuario@192.168.0.1
Linux alfa 5.10.0-21-amd64 #1 SMP Debian 5.10.162-1 (2023-01-21) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sun Mar  5 09:59:07 2023 from 172.29.0.70
usuario@alfa:~$ 

```

Realizo ssh desde bravo a alfa:

```txt
[usuario@bravo ~]$ ssh usuario@172.16.0.1
Linux alfa 5.10.0-21-amd64 #1 SMP Debian 5.10.162-1 (2023-01-21) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sun Mar  5 10:01:17 2023 from 192.168.0.3
usuario@alfa:~$ 

```

### La máquina Alfa debe tener permitido el tráfico para la interfaz loopback.

```txt
root@alfa:/home/usuario# iptables -A INPUT -i lo -p icmp -j ACCEPT
root@alfa:/home/usuario# iptables -A OUTPUT -o lo -p icmp -j ACCEPT
```

ping a la loopback:

```txt
root@alfa:/home/usuario# ping 127.0.0.1
PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.143 ms
64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.106 ms
64 bytes from 127.0.0.1: icmp_seq=3 ttl=64 time=0.067 ms
64 bytes from 127.0.0.1: icmp_seq=4 ttl=64 time=0.096 ms
^C
--- 127.0.0.1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3056ms
rtt min/avg/max/mdev = 0.067/0.103/0.143/0.027 ms
root@alfa:/home/usuario# 

```

### A la máquina Alfa se le puede hacer ping desde la DMZ, pero desde la LAN se le debe rechazar la conexión (REJECT) y desde el exterior se rechazará de manera silenciosa.

Las reglas para que desde la red DMZ puedan hacer ping a bravo son las siguientes:

```txt
root@alfa:/home/usuario# iptables -A INPUT -s 172.16.0.0/16 -p icmp -m icmp --icmp-type echo-request -j ACCEPT
root@alfa:/home/usuario# iptables -A OUTPUT -d 172.16.0.0/16 -p icmp -m icmp --icmp-type echo-reply -j ACCEPT
```


Ping desde la red dmz a alfa:

```txt
[usuario@bravo ~]$ ping 172.16.0.1
PING 172.16.0.1 (172.16.0.1) 56(84) bytes of data.
64 bytes from 172.16.0.1: icmp_seq=1 ttl=64 time=0.395 ms
64 bytes from 172.16.0.1: icmp_seq=2 ttl=64 time=0.881 ms
64 bytes from 172.16.0.1: icmp_seq=3 ttl=64 time=0.697 ms
64 bytes from 172.16.0.1: icmp_seq=4 ttl=64 time=0.929 ms
^C
--- 172.16.0.1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3059ms
rtt min/avg/max/mdev = 0.395/0.725/0.929/0.209 ms
```

Las reglas usadas para rechazar la conexión desde la red LAN son las siguientes:

```txt
root@alfa:/home/usuario# iptables -A INPUT -s 192.168.0.0/24 -p icmp -m icmp --icmp-type echo-request -j REJECT
root@alfa:/home/usuario# iptables -A OUTPUT -d 192.168.0.0/24 -p icmp -m icmp --icmp-type echo-reply -j REJECT
```

Prueba de que no funciona el ping desde la lan:

```txt
ubuntu@delta:~$ ping 192.168.0.1
PING 192.168.0.1 (192.168.0.1) 56(84) bytes of data.
^C
--- 192.168.0.1 ping statistics ---
12 packets transmitted, 0 received, 100% packet loss, time 11247ms
```

Con rechazar de manera silenciosa nos referimos a que se rechaza directamente con las políticas drop que hemos puesto al principio. Para ello pruebo que desde el exterior tampoco podemos hacer ping a alfa:

```txt
alemd@debian:~$ ping 172.22.200.13
PING 172.22.200.13 (172.22.200.13) 56(84) bytes of data.
^C
--- 172.22.200.13 ping statistics ---
4 packets transmitted, 0 received, 100% packet loss, time 3073ms

```

### La máquina Alfa puede hacer ping a la LAN, la DMZ y al exterior.

Con estas reglas aceptamos que alga haga ping a todo:

```txt
root@alfa:/home/usuario# iptables -A OUTPUT -p icmp -m icmp --icmp-type echo-request -j ACCEPT
root@alfa:/home/usuario# iptables -A INPUT -p icmp -m icmp --icmp-type echo-reply -j ACCEPT
```

Prueba de ping a la LAN, DMZ y exterior:

```txt
root@alfa:/home/usuario# ping 192.168.0.2
PING 192.168.0.2 (192.168.0.2) 56(84) bytes of data.
64 bytes from 192.168.0.2: icmp_seq=1 ttl=64 time=0.232 ms
64 bytes from 192.168.0.2: icmp_seq=2 ttl=64 time=0.101 ms
64 bytes from 192.168.0.2: icmp_seq=3 ttl=64 time=0.106 ms
^C
--- 192.168.0.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2032ms
rtt min/avg/max/mdev = 0.101/0.146/0.232/0.060 ms
root@alfa:/home/usuario# ping 192.168.0.3
PING 192.168.0.3 (192.168.0.3) 56(84) bytes of data.
64 bytes from 192.168.0.3: icmp_seq=1 ttl=64 time=0.103 ms
64 bytes from 192.168.0.3: icmp_seq=2 ttl=64 time=0.105 ms
64 bytes from 192.168.0.3: icmp_seq=3 ttl=64 time=0.099 ms
^C
--- 192.168.0.3 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2041ms
rtt min/avg/max/mdev = 0.099/0.102/0.105/0.002 ms
root@alfa:/home/usuario# ping 172.16.0.200
PING 172.16.0.200 (172.16.0.200) 56(84) bytes of data.
64 bytes from 172.16.0.200: icmp_seq=1 ttl=64 time=0.889 ms
64 bytes from 172.16.0.200: icmp_seq=2 ttl=64 time=0.890 ms
64 bytes from 172.16.0.200: icmp_seq=3 ttl=64 time=1.04 ms
^C

root@alfa:/home/usuario# ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=111 time=40.6 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=111 time=40.4 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=111 time=40.5 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=111 time=40.7 ms
^C
--- 8.8.8.8 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3006ms
rtt min/avg/max/mdev = 40.398/40.544/40.682/0.116 ms
root@alfa:/home/usuario# 

```

### Desde la máquina Bravo se puede hacer ping y conexión ssh a las máquinas de la LAN.

Permitir ping desde bravo a la LAN:

```txt
root@alfa:/home/usuario# iptables -A FORWARD -s 172.16.0.200/16 -d 192.168.0.0/24 -p icmp -m icmp --icmp-type echo-request -j ACCEPT
root@alfa:/home/usuario# iptables -A FORWARD -d 172.16.0.200/16 -s 192.168.0.0/24 -p icmp -m icmp --icmp-type echo-reply -j ACCEPT
```

Pruebo que funciona el ping:

```txt
[usuario@bravo ~]$ ping 192.168.0.2
PING 192.168.0.2 (192.168.0.2) 56(84) bytes of data.
64 bytes from 192.168.0.2: icmp_seq=1 ttl=63 time=0.921 ms
64 bytes from 192.168.0.2: icmp_seq=2 ttl=63 time=0.888 ms
64 bytes from 192.168.0.2: icmp_seq=3 ttl=63 time=0.990 ms
64 bytes from 192.168.0.2: icmp_seq=4 ttl=63 time=0.916 ms
^C
--- 192.168.0.2 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3004ms
rtt min/avg/max/mdev = 0.888/0.928/0.990/0.037 ms
[usuario@bravo ~]$ ping 192.168.0.3
PING 192.168.0.3 (192.168.0.3) 56(84) bytes of data.
64 bytes from 192.168.0.3: icmp_seq=1 ttl=63 time=0.372 ms
64 bytes from 192.168.0.3: icmp_seq=2 ttl=63 time=0.869 ms
64 bytes from 192.168.0.3: icmp_seq=3 ttl=63 time=0.939 ms
^C
--- 192.168.0.3 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2053ms
rtt min/avg/max/mdev = 0.372/0.726/0.939/0.252 ms
[usuario@bravo ~]$ 

```

Permitir ssh desde bravo a la LAN:

```txt
root@alfa:/home/usuario# iptables -A FORWARD -s 172.16.0.200/16 -d 192.168.0.0/24 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
root@alfa:/home/usuario# iptables -A FORWARD -d 172.16.0.200/16 -s 192.168.0.0/24 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT

```

ssh desde bravo a la LAN:

```txt
[usuario@bravo ~]$ ssh ubuntu@192.168.0.2
The authenticity of host '192.168.0.2 (192.168.0.2)' can't be established.
ED25519 key fingerprint is SHA256:z9xxpXd5midX9yWZZYnriMe0GiGsB6EyvICFXTngapc.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.0.2' (ED25519) to the list of known hosts.
ubuntu@192.168.0.2's password: 
Welcome to Ubuntu 20.04.5 LTS (GNU/Linux 5.10.0-21-amd64 x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage
Last login: Tue Feb 14 09:39:47 2023 from 192.168.0.1
ubuntu@charlie:~$ 

```

### Desde cualquier máquina de la LAN se puede conectar por ssh a la máquina Bravo.

Las reglas usadas son las siguientes:

```txt
root@alfa:/home/usuario# iptables -A FORWARD -s 192.168.0.0/24 -d 172.16.0.200/16 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
root@alfa:/home/usuario# iptables -A FORWARD -s 172.16.0.200/16 -d 192.168.0.0/24 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT
```

Me conecto desde charlie a bravo:

```txt
ubuntu@charlie:~$ ssh usuario@172.16.0.200
Last login: Sun Mar  5 12:45:52 2023 from 172.16.0.1
[usuario@bravo ~]$ 

```
Me conecto desde delta a bravo:

```txt
ubuntu@delta:~$ ssh usuario@172.16.0.200
The authenticity of host '172.16.0.200 (172.16.0.200)' can't be established.
ECDSA key fingerprint is SHA256:LB+A+mITUMpM9cur6VRckSO/0uVzUYjPGAcNUopJwZ0.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '172.16.0.200' (ECDSA) to the list of known hosts.
Last login: Sun Mar  5 12:46:44 2023 from 192.168.0.2
[usuario@bravo ~]$ 

```

### Configura la máquina Alfa para que las máquinas de LAN y DMZ puedan acceder al exterior.

Configuración para que la LAN salga al exterior:

```txt
root@alfa:/home/usuario# iptables -t nat -A POSTROUTING -s 192.168.0.0/24 -o ens3 -j MASQUERADE
root@alfa:/home/usuario# iptables -A FORWARD -i br0 -o ens3 -p icmp -m icmp --icmp-type echo-request -j ACCEPT
root@alfa:/home/usuario# iptables -A FORWARD -i ens3 -o br0 -p icmp -m icmp --icmp-type echo-reply -j ACCEPT
```

Prueba de que funciona correctamente desde la LAN:

```txt
ubuntu@delta:~$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=110 time=40.6 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=110 time=41.0 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=110 time=40.9 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=110 time=40.9 ms
^C
--- 8.8.8.8 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 40.614/40.854/41.031/0.152 ms
```

Reglas para que la DMZ salga al exterior:

```txt
root@alfa:/home/usuario# iptables -t nat -A POSTROUTING -s 172.16.0.0/16 -o ens3 -j MASQUERADE
root@alfa:/home/usuario# iptables -A FORWARD -i ens4 -o ens3 -p icmp -m icmp --icmp-type echo-request -j ACCEPT
root@alfa:/home/usuario# iptables -A FORWARD -i ens3 -o ens4 -p icmp -m icmp --icmp-type echo-reply -j ACCEPT
```

Prueba de que funciona correctamente desde la DMZ:

```txt
[usuario@bravo ~]$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=110 time=40.8 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=110 time=41.1 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=110 time=41.7 ms
^C
--- 8.8.8.8 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 40.762/41.189/41.672/0.373 ms

```

### Las máquinas de la LAN pueden hacer ping al exterior y navegar.

Primero para que pueda navegar debe resolver correctamente las direcciones dns, entonces tendremos que permitir el tráfico dns en nuestro escenario:

```txt
root@alfa:/home/usuario# iptables -A OUTPUT -p udp --dport 53 -m state --state NEW,ESTABLISHED -j ACCEPT
root@alfa:/home/usuario# iptables -A INPUT -p udp --sport 53 -m state --state ESTABLISHED -j ACCEPT
root@alfa:/home/usuario# iptables -t nat -A PREROUTING -p udp -i ens3 --dport 53 -j DNAT --to 192.168.0.2
root@alfa:/home/usuario# iptables -A FORWARD -p udp --dport 53 -m state --state NEW,ESTABLISHED -j ACCEPT
root@alfa:/home/usuario# iptables -A FORWARD -p udp --sport 53 -m state --state ESTABLISHED -j ACCEPT
```

En el apartado anterior hemos configurado el ping, ahora configuraremos que pueda navegar en el exterior:

```txt
root@alfa:/home/usuario# iptables -A FORWARD -i br0 -o ens3 -p tcp --dport 80 -m state --state NEW,ESTABLISHED -j ACCEPT
root@alfa:/home/usuario# iptables -A FORWARD -i ens3 -o br0 -p tcp --sport 80 -m state --state ESTABLISHED -j ACCEPT
root@alfa:/home/usuario# iptables -A FORWARD -i br0 -o ens3 -p tcp --dport 443 -m state --state NEW,ESTABLISHED -j ACCEPT
root@alfa:/home/usuario# iptables -A FORWARD -i ens3 -o br0 -p tcp --sport 443 -m state --state ESTABLISHED -j ACCEPT
```

Prueba desde charlie:

```txt
ubuntu@charlie:~$ curl google.es
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="http://www.google.es/">here</A>.
</BODY></HTML>
```

Prueba desde delta:

```txt
ubuntu@delta:~$ curl google.es
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="http://www.google.es/">here</A>.
</BODY></HTML>
```

### La máquina Bravo puede navegar. Instala un servidor web, un servidor ftp y un servidor de correos si no los tienes aún.

Usaré delta para el servidor de correos ya que tiene instalado un servidor de correos.

```txt
root@alfa:/home/usuario# iptables -A FORWARD -i ens4 -o ens3 -p tcp --dport 80 -m state --state NEW,ESTABLISHED -j ACCEPT
root@alfa:/home/usuario# iptables -A FORWARD -i ens3 -o ens4 -p tcp --sport 80 -m state --state ESTABLISHED -j ACCEPT
root@alfa:/home/usuario# iptables -A FORWARD -i ens4 -o ens3 -p tcp --dport 443 -m state --state NEW,ESTABLISHED -j ACCEPT
root@alfa:/home/usuario# iptables -A FORWARD -i ens3 -o ens4 -p tcp --sport 443 -m state --state ESTABLISHED -j ACCEPT
```

Pruebo de que bravo puede navegar:

```txt
[usuario@bravo ~]$ curl google.es
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="http://www.google.es/">here</A>.
</BODY></HTML>
```


### Configura la máquina Alfa para que los servicios web y ftp sean accesibles desde el exterior.

```txt
root@alfa:/home/usuario# iptables -t nat -A PREROUTING -p tcp -i ens3 --dport 80 -j DNAT --to 172.16.0.200
root@alfa:/home/usuario# iptables -A FORWARD -i ens3 -o ens4 -p tcp --dport 80 -m state --state NEW,ESTABLISHED -j ACCEPT
root@alfa:/home/usuario# iptables -A FORWARD -i ens4 -o ens3 -p tcp --sport 80 -m state --state ESTABLISHED -j ACCEPT
root@alfa:/home/usuario# iptables -t nat -A PREROUTING -p tcp -i ens3 --dport 21 -j DNAT --to 172.16.0.200
root@alfa:/home/usuario# iptables -A FORWARD -i ens3 -o ens4 -p tcp --dport 21 -m state --state NEW,ESTABLISHED -j ACCEPT
root@alfa:/home/usuario# iptables -A FORWARD -i ens4 -o ens3 -p tcp --sport 21 -m state --state ESTABLISHED -j ACCEPT
```

Prueba de que es accesible desde el exterior el servidor web:

```txt
alemd@debian:~$ curl 172.22.200.13
<!doctype html>
<html lang="en" class="no-js">
<head>
    <meta charset="utf-8">
    <meta http-equiv="x-ua-compatible" content="ie=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel="canonical" href="https://html5-templates.com/" />
    <title>Bootstrap Template With Sticky Menu</title>
    <meta name="description" content="Simplified Bootstrap template with sticky menu">
    <link href="css/bootstrap.min.css" rel="stylesheet">
    <link href="css/sticky-menu.css" rel="stylesheet">
</head>
<body id="page-top" data-spy="scroll" data-target=".navbar-fixed-top">
    <nav class="navbar navbar-default navbar-fixed-top" role="navigation">
        <div class="container">
            <div class="navbar-header page-scroll">
                <button type="button" class="navbar-toggle" data-toggle="collapse" data-target=".navbar-ex1-collapse">
                    <span class="sr-only">Toggle menu</span>
                    <span class="icon-bar"></span>
                    <span class="icon-bar"></span>
                    <span class="icon-bar"></span>
                </button>
                <a class="navbar-brand page-scroll" href="#page-top">Welcome</a>
            </div>

            <div class="collapse navbar-collapse navbar-ex1-collapse">
                <ul class="nav navbar-nav">
                    <li class="hidden">
                        <a class="page-scroll" href="#page-top"></a>
                    </li>
                    <li>
                        <a class="page-scroll" href="#about">About</a>
                    </li>
                    <li>
                        <a class="page-scroll" href="#whatwedo">What We Do</a>
                    </li>
                    <li>
                        <a class="page-scroll" href="#contact">Contact</a>
                    </li>
                </ul>
            </div>	<!-- .navbar-collapse -->
        </div>		<!-- .container -->
    </nav>
    <!-- Welcome   -->
    <section id="welcome" class="welcome-section">
        <div class="container">
            <div class="row">
                <div class="col-lg-12">
                    <h1>Curso: Introducción Docker</h1>
                    <h2>Aplicación escrita en PHP</h2>
                    <p>Fecha: <strong>
                        05/03/2023                    </strong></p>
                    <a class="btn btn-primary page-scroll" href="#about">Click To Scroll Down!</a>
                </div>
            </div>
        </div>
    </section>

    <!-- About -->
    <section id="about" class="about-section">
        <div class="container">
            <div class="row">
                <div class="col-lg-12">
                    <h1>About</h1>
                </div>
            </div>
        </div>
    </section>

    <!-- What we do Section -->
    <section id="whatwedo" class="whatwedo-section">
        <div class="container">
            <div class="row">
                <div class="col-lg-12">
                    <h1>What We Do</h1>
                </div>
            </div>
        </div>
    </section>

    <!-- Contact Section -->
    <section id="contact" class="contact-section">
        <div class="container">
            <div class="row">
                <div class="col-lg-12">
                    <h1>Contact Section</h1>
                </div>
            </div>
        </div>
    </section>
	
	<a id="back2Top" title="Back to top" href="#">&#10148;</a>
	
    <!-- jQuery -->
    <script src="js/jquery.js"></script>

    <!-- Bootstrap Core JavaScript -->
    <script src="js/bootstrap.min.js"></script>

    <!-- Scrolling Nav JavaScript -->
    <script src="js/jquery.easing.min.js"></script>
    <script src="js/sticky-menu.js"></script>

</body>

</html>
```

### El servidor web y el servidor ftp deben ser accesibles desde la LAN y desde el exterior.
### El servidor de correos sólo debe ser accesible desde la LAN.
### En la máquina Charlie instala un servidor mysql si no lo tiene aún. A este servidor se puede acceder desde la DMZ, pero no desde el exterior.
### Evita ataques DoS por ICMP Flood, limitando el número de peticiones por segundo desde una misma IP.
### Evita ataques DoS por SYN Flood.
### Evita que realicen escaneos de puertos a Alfa.

---
## **Documento realizado por:**

 ✒️ **Alejandro Montes Delgado** - *2º ASIR*