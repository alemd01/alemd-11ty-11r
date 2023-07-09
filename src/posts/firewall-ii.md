---
layout: post
title: "Cortafuegos II: Perimetral con nftables."
excerpt: En este post veremos la segunda práctica de cortafuegos.
date: 2023-02-21
updatedDate: 2023-02-21
tags:
  - post
  - SAD
---

### Despliegue del escenario.

Antes que nada, lo primero que haremos será desplegar la receta heat escenario2.yaml en OpenStack. Para que funcione tendremos que cambiar el ID de la imagen que usa por uno actualizado con debian 11. El id que he usado yo es el siguiente `6d992898-7e4f-44b9-a681-6dcf32d24a1f`, este ID se puede encontrar en el apartado de Computación de OpenStack, en Imágenes, seleccionamos la imagen que vamos a usar para las máquinas del escenario, en este caso Debian 11 y copiamos el ID. 

### Configuración de rutas por defecto.

En lan configuramos las rutas por defecto:

```txt
debian@lan:~$ sudo ip route del default
debian@lan:~$ sudo ip route add default via 192.168.100.2
```

### Preparación del escenario con nftables

#### Crear las tablas filter y nat.

```txt
root@router-fw:/home/debian# nft add table inet filter
root@router-fw:/home/debian# nft add table inet nat
```

#### Crear las cadenas de filter.

```txt
root@router-fw:/home/debian#  nft add chain inet filter input { type filter hook input priority 0 \; counter \; policy accept \; }
root@router-fw:/home/debian#  nft add chain inet filter output { type filter hook output priority 0 \; counter \; policy accept \; }
root@router-fw:/home/debian#  nft add chain inet filter forward { type filter hook forward priority 0 \; counter \; policy accept \; }
```

#### Crear las cadenas de nat. 

```txt
root@router-fw:/home/debian#  nft add chain inet nat prerouting { type nat hook prerouting priority 0 \; }
root@router-fw:/home/debian#  nft add chain inet nat postrouting { type nat hook postrouting priority 100 \; }
```

#### Tráfico ssh entrante al firewall.

```txt
root@router-fw:/home/debian#  nft add rule inet filter input ip saddr 172.29.0.0/16 tcp dport 22 ct state new,established counter accept
root@router-fw:/home/debian#  nft add rule inet filter output ip daddr 172.29.0.0/16 tcp sport 22 ct state established counter accept
```

#### Políticas por defecto.

```txt
root@router-fw:/home/debian#  nft chain inet filter input { policy drop \; }
root@router-fw:/home/debian#  nft chain inet filter output { policy drop \; }
root@router-fw:/home/debian#  nft chain inet filter forward { policy drop \; }
```

Llegados a este punto, no podremos ni hacer ping a la lo ni a internet ni realizar un ssh ya que está todo restringido:

```txt
root@router-fw:/home/debian# ping 127.0.0.1
PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
^C
--- 127.0.0.1 ping statistics ---
3 packets transmitted, 0 received, 100% packet loss, time 2045ms

root@router-fw:/home/debian# ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
^C
--- 8.8.8.8 ping statistics ---
4 packets transmitted, 0 received, 100% packet loss, time 3073ms

root@router-fw:/home/debian# exit
exit
debian@router-fw:~$ ssh debian@192.168.100.10
^C
```


#### Activamos el bit de forwarding.

```txt
root@router-fw:/home/debian# echo 1 > /proc/sys/net/ipv4/ip_forward
```

#### SNAT.

```txt
root@router-fw:/home/debian#  nft add rule inet nat postrouting oifname "ens3" ip saddr 192.168.100.0/24 counter masquerade
```

#### Permitimos el ssh desde el cortafuego a la LAN.

```txt
root@router-fw:/home/debian#  nft add rule inet filter output oifname "ens4" ip daddr 192.168.100.0/24 tcp dport 22 ct state new,established counter accept
root@router-fw:/home/debian#  nft add rule inet filter input iifname "ens4" ip saddr 192.168.100.0/24 tcp sport 22 ct state established counter accept
```

Ahora probamos que el ssh a la máquina lan sí funciona:

```txt
debian@router-fw:~$ ssh debian@192.168.100.10
Linux lan 5.10.0-18-amd64 #1 SMP Debian 5.10.140-1 (2022-09-02) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Wed Feb 22 18:17:21 2023 from 192.168.100.2
debian@lan:~$ 
```
#### Permitimos tráfico para la interfaz loopback.

```txt
root@router-fw:/home/debian#  nft add rule inet filter output oifname "lo" counter accept
root@router-fw:/home/debian#  nft add rule inet filter input iifname "lo" counter accept
```

Probamos que vuelve a funcionar el ping a la lo:

```txt
root@router-fw:/home/debian# ping 127.0.0.1
PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.122 ms
64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.546 ms
64 bytes from 127.0.0.1: icmp_seq=3 ttl=64 time=0.077 ms
^C
--- 127.0.0.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2005ms
rtt min/avg/max/mdev = 0.077/0.248/0.546/0.211 ms
```

#### Peticiones y respuestas protocolo ICMP.

Para permitir que a la máquina router-fw le hagan ping desde fuera ejecutamos lo siguiente:

```txt
root@router-fw:/home/debian#  nft add rule inet filter input iifname "ens3" icmp type echo-request counter accept
root@router-fw:/home/debian#  nft add rule inet filter output oifname "ens3" icmp type echo-reply counter accept
```

Probamos que funciona:

```txt
alemd@debian:~$ ping 172.22.200.17
PING 172.22.200.17 (172.22.200.17) 56(84) bytes of data.
64 bytes from 172.22.200.17: icmp_seq=1 ttl=61 time=310 ms
64 bytes from 172.22.200.17: icmp_seq=2 ttl=61 time=332 ms
64 bytes from 172.22.200.17: icmp_seq=3 ttl=61 time=355 ms
64 bytes from 172.22.200.17: icmp_seq=4 ttl=61 time=174 ms
^C
--- 172.22.200.17 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3004ms
rtt min/avg/max/mdev = 173.704/292.667/355.466/70.578 ms
```

Ahora permito que puedan hacer ping desde el cortafuegos a la lan:

```txt
root@router-fw:/home/debian#  nft add rule inet filter output oifname "ens4" icmp type echo-request counter accept
root@router-fw:/home/debian#  nft add rule inet filter input iifname "ens4" icmp type echo-reply counter accept
```

Pruebo que funciona:

```txt
root@router-fw:/home/debian# ping 192.168.100.10
PING 192.168.100.10 (192.168.100.10) 56(84) bytes of data.
64 bytes from 192.168.100.10: icmp_seq=1 ttl=64 time=4.31 ms
64 bytes from 192.168.100.10: icmp_seq=2 ttl=64 time=5.72 ms
64 bytes from 192.168.100.10: icmp_seq=3 ttl=64 time=2.02 ms
64 bytes from 192.168.100.10: icmp_seq=4 ttl=64 time=6.16 ms
^C
--- 192.168.100.10 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3007ms
rtt min/avg/max/mdev = 2.022/4.554/6.164/1.614 ms
```

#### Reglas forward.

##### Permitir hacer ping desde la LAN.

Configuramos las siguientes reglas para que pueda haber tráfico icmp desde la lan a internet:

```txt
root@router-fw:/home/debian#  nft add rule inet filter forward iifname "ens4" oifname "ens3" ip saddr 192.168.100.0/24 icmp type echo-request counter accept
root@router-fw:/home/debian#  nft add rule inet filter forward iifname "ens3" oifname "ens4" ip daddr 192.168.100.0/24 icmp type echo-reply counter accept
```

Realizamos un ping a 1.1.1.1 para probar que funciona:

```txt
debian@lan:~$ ping 1.1.1.1
PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.
64 bytes from 1.1.1.1: icmp_seq=1 ttl=51 time=45.5 ms
64 bytes from 1.1.1.1: icmp_seq=2 ttl=51 time=44.0 ms
64 bytes from 1.1.1.1: icmp_seq=3 ttl=51 time=46.1 ms
64 bytes from 1.1.1.1: icmp_seq=4 ttl=51 time=45.4 ms
64 bytes from 1.1.1.1: icmp_seq=5 ttl=51 time=46.0 ms
^C
--- 1.1.1.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4008ms
rtt min/avg/max/mdev = 44.020/45.400/46.062/0.744 ms
```

##### Consultas y respuestas DNS desde la LAN.

Usamos las siguientes reglas para que desde la lan haya consultas y respuestas dns:

```txt
root@router-fw:/home/debian#  nft add rule inet filter forward iifname "ens4" oifname "ens3" ip saddr 192.168.100.0/24 udp dport 53 ct state new,established counter accept
root@router-fw:/home/debian#  nft add rule inet filter forward iifname "ens3" oifname "ens4" ip daddr 192.168.100.0/24 udp sport 53 ct state established counter accept
```

Pruebo que funciona con dig:

```txt
debian@lan:~$ dig @8.8.8.8 www.google.es

; <<>> DiG 9.16.33-Debian <<>> @8.8.8.8 www.google.es
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 42599
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;www.google.es.			IN	A

;; ANSWER SECTION:
www.google.es.		300	IN	A	142.250.184.163

;; Query time: 72 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)
;; WHEN: Wed Feb 22 19:20:50 UTC 2023
;; MSG SIZE  rcvd: 58
```

##### Permitimos la navegación web desde la LAN.

```txt
root@router-fw:/home/debian#  nft add rule inet filter forward iifname "ens4" oifname "ens3" ip protocol tcp ip saddr 192.168.100.0/24 tcp dport { 80,443 } ct state new,established counter accept
root@router-fw:/home/debian#  nft add rule inet filter forward iifname "ens3" oifname "ens4" ip protocol tcp ip daddr 192.168.100.0/24 tcp sport { 80,443 } ct state established counter accept
```
Pruebo que funciona con curl:

```txt
debian@lan:~$ curl google.es
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="http://www.google.es/">here</A>.
</BODY></HTML>
```

##### Permitimos el acceso a nuestro servidor web de la LAN desde el exterior.

```txt
root@router-fw:/home/debian#  nft add rule inet filter forward iifname "ens3" oifname "ens4" ip daddr 192.168.100.0/24 tcp dport 80 ct state new,established counter accept
root@router-fw:/home/debian#  nft add rule inet filter forward iifname "ens4" oifname "ens3" ip saddr 192.168.100.0/24 tcp sport 80 ct state established counter accept
```

Necesitamos una regla DNAT:

```txt
root@router-fw:/home/debian# nft add rule inet nat prerouting iifname "ens3" tcp dport 80 counter dnat ip to 192.168.100.10
```

Probamos que funciona, para ello instalaré apache en la lan:


### Permite poder hacer conexiones ssh al exterior desde la máquina cortafuegos.

```txt
root@router-fw:/home/debian# nft add rule inet filter output oifname "ens3" tcp dport 22 ct state new,established counter accept
root@router-fw:/home/debian# nft add rule inet filter input iifname "ens3" tcp sport 22 ct state established counter accept
```

Me conecto a mi host para verificar que funciona:

```txt
root@router-fw:/home/debian# ssh alemd@172.29.0.70
The authenticity of host '172.29.0.70 (172.29.0.70)' can't be established.
ECDSA key fingerprint is SHA256:P4VynMzXNSUoRSzuUrFgWShiT4hYwT/y6DqQ7Sf/ujs.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '172.29.0.70' (ECDSA) to the list of known hosts.
alemd@172.29.0.70's password: 
Linux debian 5.19.0-0.deb11.2-amd64 #1 SMP PREEMPT_DYNAMIC Debian 5.19.11-1~bpo11+1 (2022-10-03) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
You have new mail.
Last login: Tue Feb 14 13:13:33 2023 from 172.22.200.42
alemd@debian:~$ 
```

### Permite hacer consultas DNS desde la máquina cortafuegos sólo al servidor 192.168.202.2. Comprueba que no puedes hacer un dig @1.1.1.1.

```txt
root@router-fw:/home/debian# nft add rule inet filter output ip daddr 192.168.202.2 udp dport 53 ct state new,established counter accept
root@router-fw:/home/debian# nft add rule inet filter input ip saddr 192.168.202.2 udp sport 53 ct state established counter accept
```

Pruebo que desde papion funciona:

```txt
root@router-fw:/home/debian# dig @192.168.202.2 www.google.es

; <<>> DiG 9.16.33-Debian <<>> @192.168.202.2 www.google.es
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 56690
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 4, ADDITIONAL: 9

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 9f7eecd40b7f37f141ffcfbd63f7b1e31322ad62a64d5bcb (good)
;; QUESTION SECTION:
;www.google.es.			IN	A

;; ANSWER SECTION:
www.google.es.		300	IN	A	142.250.184.3

;; AUTHORITY SECTION:
google.es.		66802	IN	NS	ns3.google.com.
google.es.		66802	IN	NS	ns1.google.com.
google.es.		66802	IN	NS	ns4.google.com.
google.es.		66802	IN	NS	ns2.google.com.

;; ADDITIONAL SECTION:
ns1.google.com.		63807	IN	A	216.239.32.10
ns2.google.com.		63807	IN	A	216.239.34.10
ns3.google.com.		63807	IN	A	216.239.36.10
ns4.google.com.		63807	IN	A	216.239.38.10
ns1.google.com.		63807	IN	AAAA	2001:4860:4802:32::a
ns2.google.com.		63807	IN	AAAA	2001:4860:4802:34::a
ns3.google.com.		63807	IN	AAAA	2001:4860:4802:36::a
ns4.google.com.		63807	IN	AAAA	2001:4860:4802:38::a

;; Query time: 67 msec
;; SERVER: 192.168.202.2#53(192.168.202.2)
;; WHEN: Thu Feb 23 18:35:15 UTC 2023
;; MSG SIZE  rcvd: 344
```

Y ahora pruebo desde la 1.1.1.1:

```txt
root@router-fw:/home/debian# dig @1.1.1.1 www.google.es

; <<>> DiG 9.16.33-Debian <<>> @1.1.1.1 www.google.es
; (1 server found)
;; global options: +cmd
;; connection timed out; no servers could be reached

```

### Permite que la máquina cortafuegos pueda navegar por internet.

```txt
root@router-fw:/home/debian# nft add rule inet filter output oifname "ens3" ip protocol tcp tcp dport { 80,443 } ct state new,established counter accept
root@router-fw:/home/debian# nft add rule inet filter input iifname "ens3" ip protocol tcp tcp sport { 80,443 } ct state established counter accept
```

Compruebo que funciona:

```txt
root@router-fw:/home/debian# curl google.es
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="http://www.google.es/">here</A>.
</BODY></HTML>
```

### Los equipos de la red local deben poder tener conexión al exterior.

Este paso está realizado anteriormente, no vuelvo a repetirlo para no duplicar información.

### Permitimos el ssh desde el cortafuego a la LAN

Este paso también está realizado anteriormente.

### Permitimos hacer ping desde la LAN a la máquina cortafuegos.

```txt
root@router-fw:/home/debian#  nft add rule inet filter input iifname "ens4" icmp type echo-request counter accept
root@router-fw:/home/debian#  nft add rule inet filter output oifname "ens4" icmp type echo-reply counter accept
```

Probamos a hacer ping desde la lan al firewall:

```txt
debian@lan:~$ ping 192.168.100.2
PING 192.168.100.2 (192.168.100.2) 56(84) bytes of data.
64 bytes from 192.168.100.2: icmp_seq=1 ttl=64 time=2.36 ms
64 bytes from 192.168.100.2: icmp_seq=2 ttl=64 time=2.97 ms
64 bytes from 192.168.100.2: icmp_seq=3 ttl=64 time=2.52 ms
64 bytes from 192.168.100.2: icmp_seq=4 ttl=64 time=4.40 ms
^C
--- 192.168.100.2 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3007ms
rtt min/avg/max/mdev = 2.361/3.061/4.399/0.803 ms

```

### Permite realizar conexiones ssh desde los equipos de la LAN

```txt
root@router-fw:/home/debian# nft add rule inet filter forward iifname "ens4" oifname "ens3" ip saddr 192.168.100.0/24 tcp dport 22 ct state new,established counter accept
root@router-fw:/home/debian# nft add rule inet filter forward iifname "ens3" oifname "ens4" ip daddr 192.168.100.0/24 tcp sport 22 ct state established counter accept
```

Pruebo a realizar un ssh a mi host desde la lan:

```txt
debian@lan:~$ ssh alemd@172.29.0.70
The authenticity of host '172.29.0.70 (172.29.0.70)' can't be established.
ECDSA key fingerprint is SHA256:P4VynMzXNSUoRSzuUrFgWShiT4hYwT/y6DqQ7Sf/ujs.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '172.29.0.70' (ECDSA) to the list of known hosts.
alemd@172.29.0.70's password: 
Linux debian 5.19.0-0.deb11.2-amd64 #1 SMP PREEMPT_DYNAMIC Debian 5.19.11-1~bpo11+1 (2022-10-03) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
You have new mail.
Last login: Thu Feb 23 19:29:53 2023 from 172.22.200.17
alemd@debian:~$ 

```

### Instala un servidor de correos en la máquina de la LAN. Permite el acceso desde el exterior y desde el cortafuego al servidor de correos. Para probarlo puedes ejecutar un telnet al puerto 25 tcp.

Permito el acceso desde el exterior:

```txt
root@router-fw:/home/debian# nft add rule inet filter forward iifname "ens3" oifname "ens4" ip daddr 192.168.100.0/24 tcp dport 25 counter accept
root@router-fw:/home/debian# nft add rule inet filter forward iifname "ens4" oifname "ens3" ip saddr 192.168.100.0/24 tcp sport 25 counter accept
root@router-fw:/home/debian# nft add rule inet nat prerouting iifname "ens3" tcp dport 25 counter dnat ip to 192.168.100.10
```

Ahora instalamos postfix y lo probamos desde mi host con telnet:

```txt
alemd@debian:~$ telnet 172.22.200.17 25
Trying 172.22.200.17...
Connected to 172.22.200.17.
Escape character is '^]'.
220 lan.novalocal ESMTP Postfix (Debian/GNU)
```

Ahora permito el acceso desde el firewall:

```txt
root@router-fw:/home/debian# nft add rule inet filter output ip daddr 192.168.100.10 tcp dport 25 counter accept
root@router-fw:/home/debian# nft add rule inet filter input ip saddr 192.168.100.10 tcp sport 25 counter accept
```

Pruebo que funciona desde el firewall:

```txt
root@router-fw:/home/debian# telnet 192.168.100.10 25
Trying 192.168.100.10...
Connected to 192.168.100.10.
Escape character is '^]'.
220 lan.novalocal ESMTP Postfix (Debian/GNU)
```

### Permite poder hacer conexiones ssh desde exterior a la LAN

```txt
root@router-fw:/home/debian# nft add rule inet filter forward iifname "ens3" oifname "ens4" ip daddr 192.168.100.0/24 tcp dport 22 ct state new,established counter accept
root@router-fw:/home/debian# nft add rule inet filter forward iifname "ens4" oifname "ens3" ip saddr 192.168.100.0/24 tcp sport 22 ct state established counter accept
root@router-fw:/home/debian# nft add rule inet nat prerouting iifname "ens3" tcp dport 22 counter dnat ip to 192.168.100.10
```

Me conecto desde mi host:

```txt
alemd@debian:~$ ssh debian@172.22.200.17
The authenticity of host '172.22.200.17 (172.22.200.17)' can't be established.
ECDSA key fingerprint is SHA256:9RvhywkwaWaGJ74PhMORwkJ7ky7SkDy+eZ5Jjn3agvQ.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '172.22.200.17' (ECDSA) to the list of known hosts.
Linux lan 5.10.0-18-amd64 #1 SMP Debian 5.10.140-1 (2022-09-02) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Thu Feb 23 19:35:07 2023 from 192.168.100.2
debian@lan:~$ 
```

### Modifica la regla anterior, para que al acceder desde el exterior por ssh tengamos que conectar al puerto 2222, aunque el servidor ssh este configurado para acceder por el puerto 22.

```txt
root@router-fw:/home/debian# sudo nft add rule inet nat prerouting iifname "ens3" tcp dport 2222 counter dnat ip to 192.168.100.10:22
```

Pruebo que funciona:

```txt
alemd@debian:~$ ssh -p 2222 debian@172.22.200.17
Linux lan 5.10.0-18-amd64 #1 SMP Debian 5.10.140-1 (2022-09-02) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Thu Feb 23 19:55:37 2023 from 172.29.0.70
debian@lan:~$ 
```

### Permite hacer consultas DNS desde la LAN sólo al servidor 192.168.202.2. Comprueba que no puedes hacer un dig @1.1.1.1.

```txt
root@router-fw:/home/debian# nft -a list ruleset | grep 53
		ip saddr 172.29.0.0/16 tcp dport 22 ct state established,new counter packets 5318 bytes 372360 accept # handle 8
		ip saddr 192.168.202.2 udp sport 53 ct state established counter packets 55 bytes 19786 accept # handle 31
		ip daddr 192.168.202.2 udp dport 53 ct state established,new counter packets 55 bytes 3712 accept # handle 30
		iifname "ens4" oifname "ens3" ip saddr 192.168.100.0/24 udp dport 53 ct state established,new counter packets 202 bytes 13946 accept # handle 20
		iifname "ens3" oifname "ens4" ip daddr 192.168.100.0/24 udp sport 53 ct state established counter packets 202 bytes 79854 accept # handle 21
```

Como vemos tenemos que eliminar los handle 20 y 21.

```txt
root@router-fw:/home/debian# sudo nft delete rule inet filter forward handle 20 
root@router-fw:/home/debian# sudo nft delete rule inet filter forward handle 21
```

Ahora añadimos las nuevas reglas:

```txt
root@router-fw:/home/debian# sudo nft add rule inet filter forward iifname "ens4" oifname "ens3" ip saddr 192.168.100.0/24 ip daddr 192.168.202.2 udp dport 53 ct state new,established counter accept
root@router-fw:/home/debian# sudo nft add rule inet filter forward iifname "ens3" oifname "ens4" ip saddr 192.168.202.2 ip daddr 192.168.100.0/24 udp sport 53 ct state established counter accept
```

Realizo consultas a papion:

```txt
debian@lan:~$ dig @192.168.202.2 google.es

; <<>> DiG 9.16.33-Debian <<>> @192.168.202.2 google.es
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 45495
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 4, ADDITIONAL: 9

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 7f256c5d9934b3162b2837d563f7c7eb8fff0cd52a78e95c (good)
;; QUESTION SECTION:
;google.es.			IN	A

;; ANSWER SECTION:
google.es.		300	IN	A	142.250.185.3

;; AUTHORITY SECTION:
google.es.		61162	IN	NS	ns2.google.com.
google.es.		61162	IN	NS	ns4.google.com.
google.es.		61162	IN	NS	ns1.google.com.
google.es.		61162	IN	NS	ns3.google.com.

;; ADDITIONAL SECTION:
ns1.google.com.		58167	IN	A	216.239.32.10
ns2.google.com.		58167	IN	A	216.239.34.10
ns3.google.com.		58167	IN	A	216.239.36.10
ns4.google.com.		58167	IN	A	216.239.38.10
ns1.google.com.		58167	IN	AAAA	2001:4860:4802:32::a
ns2.google.com.		58167	IN	AAAA	2001:4860:4802:34::a
ns3.google.com.		58167	IN	AAAA	2001:4860:4802:36::a
ns4.google.com.		58167	IN	AAAA	2001:4860:4802:38::a

;; Query time: 76 msec
;; SERVER: 192.168.202.2#53(192.168.202.2)
;; WHEN: Thu Feb 23 20:09:20 UTC 2023
;; MSG SIZE  rcvd: 340
```

Consulto a la ip 1.1.1.1:

```txt
debian@lan:~$ dig @1.1.1.1 google.es

; <<>> DiG 9.16.33-Debian <<>> @1.1.1.1 google.es
; (1 server found)
;; global options: +cmd
;; connection timed out; no servers could be reached
```

### Permite que los equipos de la LAN puedan navegar por internet.

Este paso se ha realizado anteriormente.

---
## **Documento realizado por:**

 ✒️ **Alejandro Montes Delgado** - *2º ASIR*