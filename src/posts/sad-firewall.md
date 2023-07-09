---
layout: post
title: "Cortafuegos I: De nodo con iptables."
excerpt: Primera práctica de la unidad de Firewall.
date: 2023-02-11
updatedDate: 2023-02-11
tags:
  - post
  - SAD
---

### Limpieza de las reglas previas.

```txt
root@maquina:/home/debian# iptables -F
root@maquina:/home/debian# iptables -t nat -F
root@maquina:/home/debian# iptables -Z
root@maquina:/home/debian# iptables -t nat -Z
```

### Vamos a permitir ssh.

Cómo estamos conectado a la máquina por ssh, vamos a permitir la conexión ssh desde la red 172.22.0.0/16, antes de cambiar las políticas por defecto a DROP, para no perder la conexión:

```txt
root@maquina:/home/debian# iptables -A INPUT -s 172.22.0.0/16 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
root@maquina:/home/debian# iptables -A OUTPUT -d 172.22.0.0/16 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT
```

### Política por defecto.

```txt
root@maquina:/home/debian# iptables -P INPUT DROP
root@maquina:/home/debian# iptables -P OUTPUT DROP
```

Comprobamos que el equipo no puede acceder a ningún servicio ni de Internet ni de la red local, ya que la política lo impide.

```txt
root@maquina:/home/debian# ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=111 time=478 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=111 time=586 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=111 time=463 ms
^C
--- 8.8.8.8 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 463.406/509.111/586.265/54.865 ms
root@maquina:/home/debian# iptables -P INPUT DROP
root@maquina:/home/debian# iptables -P OUTPUT DROP
root@maquina:/home/debian# ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
^C
--- 8.8.8.8 ping statistics ---
3 packets transmitted, 0 received, 100% packet loss, time 2033ms
```

### Permitimos tráfico para la interfaz loopback.

```txt
root@maquina:/home/debian# iptables -A INPUT -i lo -p icmp -j ACCEPT
root@maquina:/home/debian# iptables -A OUTPUT -o lo -p icmp -j ACCEPT
```

### Peticiones y respuestas protocolo ICMP.

Compruebo que no funciona:

```txt
root@maquina:/home/debian# ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
^C
--- 8.8.8.8 ping statistics ---
11 packets transmitted, 0 received, 100% packet loss, time 10226ms
```

Añado las reglas:

```txt
root@maquina:/home/debian# iptables -A INPUT -i ens3 -p icmp --icmp-type echo-reply -j ACCEPT
root@maquina:/home/debian# iptables -A OUTPUT -o ens3 -p icmp --icmp-type echo-request -j ACCEPT
```

Comprobamos su funcionamiento haciendo ping a una IP pública:

```txt
root@maquina:/home/debian# ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=2 ttl=111 time=312 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=111 time=486 ms
64 bytes from 8.8.8.8: icmp_seq=5 ttl=111 time=302 ms
64 bytes from 8.8.8.8: icmp_seq=6 ttl=111 time=594 ms
^C
--- 8.8.8.8 ping statistics ---
6 packets transmitted, 4 received, 33.3333% packet loss, time 5018ms
rtt min/avg/max/mdev = 302.468/423.514/594.125/122.526 ms
```

### Consultas y respuestas DNS.

```txt
root@maquina:/home/debian# iptables -A OUTPUT -o ens3 -p udp --dport 53 -m state --state NEW,ESTABLISHED -j ACCEPT
root@maquina:/home/debian# iptables -A INPUT -i ens3 -p udp --sport 53 -m state --state ESTABLISHED -j ACCEPT
```

Comprobamos su funcionamiento con una consulta DNS:

```txt
root@maquina:/home/debian# dig @8.8.8.8 www.josedomingo.org

; <<>> DiG 9.16.37-Debian <<>> @8.8.8.8 www.josedomingo.org
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 17100
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;www.josedomingo.org.		IN	A

;; ANSWER SECTION:
www.josedomingo.org.	900	IN	CNAME	endor.josedomingo.org.
endor.josedomingo.org.	900	IN	A	37.187.119.60

;; Query time: 304 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)
;; WHEN: Tue Feb 14 11:51:49 UTC 2023
;; MSG SIZE  rcvd: 84
```

### Tráfico https.


```txt
root@maquina:/home/debian# iptables -A OUTPUT -o ens3 -p tcp --dport 443 -m state --state NEW,ESTABLISHED -j ACCEPT
root@maquina:/home/debian# iptables -A INPUT -i ens3 -p tcp --sport 443 -m state --state ESTABLISHED -j ACCEPT
```

### Tráfico http/https.

```txt
root@maquina:/home/debian# iptables -A OUTPUT -o ens3 -p tcp -m multiport --dports 80,443 -m state --state NEW,ESTABLISHED -j ACCEPT
root@maquina:/home/debian# iptables -A INPUT -i ens3 -p tcp -m multiport --sports 80,443 -m state --state ESTABLISHED -j ACCEPT
```

### Permitimos el acceso a nuestro servidor web.

```txt
root@maquina:/home/debian# iptables -A INPUT -i ens3 -p tcp --dport 80 -m state --state NEW,ESTABLISHED -j ACCEPT
root@maquina:/home/debian# iptables -A OUTPUT -o ens3 -p tcp --sport 80 -m state --state ESTABLISHED -j ACCEPT
```

### Ejercicios.

#### 1. Permite poder hacer conexiones ssh al exterior.

Añadimos reglas para permitir ssh al exterior:

```txt
root@maquina:/home/debian# iptables -A INPUT -i ens3 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT
root@maquina:/home/debian# iptables -A OUTPUT -o ens3 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT

```

Probamos que funciona el ssh

```txt
root@maquina:/home/debian# ssh alemd@172.22.0.161
The authenticity of host '172.22.0.161 (172.22.0.161)' can't be established.
ECDSA key fingerprint is SHA256:P4VynMzXNSUoRSzuUrFgWShiT4hYwT/y6DqQ7Sf/ujs.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '172.22.0.161' (ECDSA) to the list of known hosts.
alemd@172.22.0.161's password: 
Linux debian 5.19.0-0.deb11.2-amd64 #1 SMP PREEMPT_DYNAMIC Debian 5.19.11-1~bpo11+1 (2022-10-03) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
You have new mail.
alemd@debian:~$ 
```

#### 2. Deniega el acceso a tu servidor web desde una ip concreta.

Instalamos apache y hacemos una web de prueba. Muestro las reglas para denegar el acceso a alfa que es la IP 172.22.200.13 

```txt
root@maquina:/home/debian# iptables -I INPUT 4 ! -s 172.22.200.13 -p tcp --dport 80 -m state --state NEW,ESTABLISHED -j ACCEPT
root@maquina:/home/debian# iptables -I OUTPUT 4 ! -d 172.22.200.13 -p tcp --sport 80 -m state --state ESTABLISHED -j ACCEPT
```

Desde mi máquina, hago un curl.

```txt
alemd@debian:~$ curl 172.22.200.42
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
  <body>
   <h1>PRACTICA FIREWALL</p>
  </body>
</html>
```

Desde alfa hago un curl.

```txt
usuario@alfa:~$ curl 172.22.200.42
curl: (28) Failed to connect to 172.22.200.42 port 80: Connection timed out
```

#### 3. Permite hacer consultas DNS sólo al servidor 192.168.202.2. Comprueba que no puedes hacer un dig @1.1.1.1.

Las reglas introducidas son las siguientes:

```txt
root@maquina:/home/debian# iptables -I INPUT 6 -s 192.168.202.2/32 -p udp --sport 53 -m state --state ESTABLISHED -j ACCEPT
root@maquina:/home/debian# iptables -I OUTPUT 6 -d 192.168.202.2/32 -p udp --dport 53 -m state --state NEW,ESTABLISHED -j ACCEPT
```


Hago una consulta al servidor 192.168.202.2:

```txt
root@maquina:/home/debian# dig @192.168.202.2 www.josedomingo.org

; <<>> DiG 9.16.37-Debian <<>> @192.168.202.2 www.josedomingo.org
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 40487
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 5, ADDITIONAL: 6

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 456870443b553f07e841ab7e63ed310c806bb11f806a708b (good)
;; QUESTION SECTION:
;www.josedomingo.org.		IN	A

;; ANSWER SECTION:
www.josedomingo.org.	900	IN	CNAME	endor.josedomingo.org.
endor.josedomingo.org.	900	IN	A	37.187.119.60

;; AUTHORITY SECTION:
josedomingo.org.	3600	IN	NS	ns1.cdmon.net.
josedomingo.org.	3600	IN	NS	ns2.cdmon.net.
josedomingo.org.	3600	IN	NS	ns5.cdmondns-01.com.
josedomingo.org.	3600	IN	NS	ns4.cdmondns-01.org.
josedomingo.org.	3600	IN	NS	ns3.cdmon.net.

;; ADDITIONAL SECTION:
ns1.cdmon.net.		67034	IN	A	35.189.106.232
ns2.cdmon.net.		67034	IN	A	35.195.57.29
ns3.cdmon.net.		67034	IN	A	35.157.47.125
ns4.cdmondns-01.org.	3600	IN	A	52.58.66.183
ns5.cdmondns-01.com.	67034	IN	A	52.59.146.62

;; Query time: 260 msec
;; SERVER: 192.168.202.2#53(192.168.202.2)
;; WHEN: Wed Feb 15 19:22:52 UTC 2023
;; MSG SIZE  rcvd: 318
```

Hago una consulta a través del servidor 1.1.1.1:

```txt
root@maquina:/home/debian# dig @1.1.1.1 www.josedomingo.org

; <<>> DiG 9.16.37-Debian <<>> @1.1.1.1 www.josedomingo.org
; (1 server found)
;; global options: +cmd
;; connection timed out; no servers could be reached
```

#### 4. No permitir el acceso al servidor web de www.josedomingo.org (Tienes que utilizar la ip). ¿Puedes acceder a fp.josedomingo.org?

Las reglas usadas son las siguientes.

```txt
root@maquina:/home/debian# iptables -I OUTPUT 8 ! -d 37.187.119.60/32 -p tcp --dport 80 -m state --state NEW,ESTABLISHED -j ACCEPT
root@maquina:/home/debian# iptables -I INPUT 8 ! -s 37.187.119.60/32 -p tcp --sport 80 -m state --state ESTABLISHED -j ACCEPT
```

Realizo un curl a www.josedomingo.org

```txt
root@maquina:/home/debian# curl www.josedomingo.org
curl: (28) Failed to connect to www.josedomingo.org port 80: Connection timed out
```

Realizo un curl a fp.josedomingo.org.

```txt
root@maquina:/home/debian# curl fp.josedomingo.org
curl: (28) Failed to connect to fp.josedomingo.org port 80: Connection timed out
```

Como vemos a fp.josedomingo.org no se puede acceder ya que tiene la misma ip que www.josedomingo.org.

Si hago un curl a otra ip si funciona:

```txt
root@maquina:/home/debian# curl alejandro-montes.es
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

#### 5. Permite mandar un correo usando nuestro servidor de correo: babuino-smtp. Para probarlo ejecuta un telnet bubuino-smtp.gonzalonazareno.org 25.

Las reglas utilizadas son las siguientes:

```txt
root@maquina:/home/debian# iptables -A OUTPUT -d 192.168.203.3/32 -p tcp --dport 25 -m state --state NEW,ESTABLISHED -j ACCEPT
root@maquina:/home/debian# iptables -A INPUT -s 192.168.203.3/32 -p tcp --sport 25 -m state --state ESTABLISHED -j ACCEPT
```

La ip 192.168.203.3 no se encuentra disponible:

```txt
debian@maquina:~$ ping babuino-smtp.gonzalonazareno.org
PING babuino-smtp.gonzalonazareno.org (192.168.203.3) 56(84) bytes of data.
^C
--- babuino-smtp.gonzalonazareno.org ping statistics ---
5 packets transmitted, 0 received, 100% packet loss, time 4104ms

```

#### 6. Instala un servidor mariadb, y permite los accesos desde la ip de tu cliente. Comprueba que desde otro cliente no se puede acceder.

Las reglas usadas son las siguientes:

```txt
root@maquina:/home/debian# iptables -A INPUT -s 172.22.6.172/16 -p tcp --dport 3306 -m state --state NEW,ESTABLISHED -j ACCEPT
root@maquina:/home/debian# iptables -A OUTPUT -d 172.22.6.172/16 -p tcp --sport 3306 -m state --state ESTABLISHED -j ACCEPT
```

He creado un servidor mariadb habilitando el acceso remoto. Desde un cliente me conecto al usuario FIREWALL.

```txt
usuario@debian:~$ mysql -h 172.22.200.42 -u FIREWALL testing -p
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 34
Server version: 10.5.18-MariaDB-0+deb11u1 Debian 11

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [testing]> 
```

---
## **Documento realizado por:**

 ✒️ **Alejandro Montes Delgado** - *2º ASIR*