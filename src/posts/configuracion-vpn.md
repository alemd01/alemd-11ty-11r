---
layout: post
title: "Configurar servidor VPN en OpenVPN y WireGuard."
excerpt: "En esta práctica veremos como configurar servidores VPN."
date: 2023-01-10
tags:
  - post
  - SAD
---

### OpenVPN

#### 1.1. VPN de acceso remoto con OpenVPN y certificados x509. (5 puntos)

**Configura una conexión VPN de acceso remoto entre dos equipos del cloud:**

* **Uno de los dos equipos (el que actuará como servidor) estará conectado a dos redes**
* **Para la autenticación de los extremos se usarán obligatoriamente certificados digitales, que se generarán utilizando openssl y se almacenarán en el directorio /etc/openvpn, junto con  los parámetros Diffie-Helman y el certificado de la propia Autoridad de Certificación.** 
* **Se utilizarán direcciones de la red 10.99.99.0/24 para las direcciones virtuales de la VPN. La dirección 10.99.99.1 se asignará al servidor VPN.**
* **Los ficheros de configuración del servidor y del cliente se crearán en el directorio /etc/openvpn de cada máquina, y se llamarán servidor.conf y cliente.conf respectivamente.** 
* **Tras el establecimiento de la VPN, la máquina cliente debe ser capaz de acceder a una máquina que esté en la otra red a la que está conectado el servidor.** 
**Documenta el proceso detalladamente.**

Antes que nada, tendremos que tener un escenario donde realizaremos la práctica. Crearé dos instancias en OpenStack, y el equipo servidor estará conectado a 2 redes distintas.

![SAD-P6.1.png](/img/SAD-P6.1.png)

Una vez creado el escenario, empezaremos con la práctica.

##### Configuración del servidor.

Lo primero que haremos será instalar openvpn.

![SAD-P6.2.png](/img/SAD-P6.2.png)

Ahora, activaremos el bit de forwarding en el servidor.

```txt
root@servidor-vpn:~$  nano /etc/sysctl.conf
```

editamos la siguiente línea:

```txt
# Uncomment the next line to enable packet forwarding for IPv4
net.ipv4.ip_forward=1
```

Por precaución, copiaremos la configuración que se encuentra en `/usr/share/easy-rsa` a `/etc/openvpn` para evitar que en una próxima actualización del paquete sobreescriba los cambios que hagamos:

```txt
root@servidor-vpn:~$  cp -r /usr/share/easy-rsa /etc/openvpn
```

Ahora dentro del directorio `/etc/openvpn/easy-rsa/`, ejecutaremos el siguiente script para inicializar el directorio PKI:

```txt
root@servidor-vpn:/etc/openvpn/easy-rsa# ./easyrsa init-pki

init-pki complete; you may now create a CA or requests.
Your newly created PKI dir is: /etc/openvpn/easy-rsa/pki



```

A continuación, vamos a generar el certificado de la CA y la clave con la que firmaremos los certificados. Tendremos que introducir una contraseña para el certificado y un nombre para la autoridad certificadora.

```txt
root@servidor-vpn:/etc/openvpn/easy-rsa# ./easyrsa build-ca
Using SSL: openssl OpenSSL 1.1.1n  15 Mar 2022

Enter New CA Key Passphrase: 
Re-Enter New CA Key Passphrase: 
Generating RSA private key, 2048 bit long modulus (2 primes)
..............+++++
................................+++++
e is 65537 (0x010001)
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Common Name (eg: your user, host, or server name) [Easy-RSA CA]:alejandro-montes

CA creation complete and you may now import and sign cert requests.
Your new CA certificate file for publishing is at:
/etc/openvpn/easy-rsa/pki/ca.crt
```

Como indica la salida del comando, el certificado se ha generado en la ruta `/etc/openvpn/easy-rsa/pki/ca.crt` mientras que la clave privada se encuentra en `/etc/openvpn/easy-rsa/pki/private/ca.key`. Ahora tendremos que generar los parámetros Diffie-Hellman, los cuáles se usan para el intercambio de claves entre el servidor de openvpn y los clientes.

![SAD-P6.3.png](/img/SAD-P6.3.png)

El archivo ha sido generado en el directorio `/etc/openvpn/easy-rsa/pki/dh.pem`.

Ahora generaremos el certificado y la clave privada del servidor openvpn:

```txt
root@servidor-vpn:/etc/openvpn/easy-rsa# ./easyrsa build-server-full server nopass
Using SSL: openssl OpenSSL 1.1.1n  15 Mar 2022
Generating a RSA private key
.........................+++++
.......................+++++
writing new private key to '/etc/openvpn/easy-rsa/pki/easy-rsa-22061.NS9Bww/tmp.gmhUC5'
-----
Using configuration from /etc/openvpn/easy-rsa/pki/easy-rsa-22061.NS9Bww/tmp.xcbWer
Enter pass phrase for /etc/openvpn/easy-rsa/pki/private/ca.key:
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
commonName            :ASN.1 12:'server'
Certificate is to be certified until Apr 17 11:03:08 2025 GMT (825 days)

Write out database with 1 new entries
Data Base Updated
```

El certificado ha sido guardado en la ruta `/etc/openvpn/easy-rsa/pki/issued/server.crt` y la clave privada se ha generado en `/etc/openvpn/easy-rsa/pki/private/server.key`. La opción nopass es para deshabilitar la frase de paso.

Una vez hecho esto, generaremos el certificado y la clave privada del cliente de la vpn:

```txt
root@servidor-vpn:/etc/openvpn/easy-rsa# ./easyrsa build-client-full cliente-vpn nopass
Using SSL: openssl OpenSSL 1.1.1n  15 Mar 2022
Generating a RSA private key
.................................+++++
..........+++++
writing new private key to '/etc/openvpn/easy-rsa/pki/easy-rsa-22138.MZyJbQ/tmp.MWGIDo'
-----
Using configuration from /etc/openvpn/easy-rsa/pki/easy-rsa-22138.MZyJbQ/tmp.1G19BJ
Enter pass phrase for /etc/openvpn/easy-rsa/pki/private/ca.key:
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
commonName            :ASN.1 12:'cliente-vpn'
Certificate is to be certified until Apr 17 11:06:42 2025 GMT (825 days)

Write out database with 1 new entries
Data Base Updated


```

El certificado esta generado en `etc/openvpn/easy-rsa/pki/issued/Clientevpn1.crt` y la clave privada en `/etc/openvpn/easy-rsa/pki/private/Clientevpn1.key`.

Ahora pasaremos al cliente los ficheros necesarios. Para ello, primero en el servidor crearemos una carpeta con los ficheros del cliente vpn y posteriormente lo pasaremos al cliente por scp mediante mi máquina local:

```txt
root@servidor-vpn:/etc/openvpn/easy-rsa# mkdir /home/debian/cliente-vpn
root@servidor-vpn:/etc/openvpn/easy-rsa# cp -rp /etc/openvpn/easy-rsa/pki/{ca.crt,issued/cliente-vpn.crt,private/cliente-vpn.key} /home/debian/cliente-vpn/
root@servidor-vpn:/etc/openvpn/easy-rsa# chown debian: /home/debian/cliente-vpn/
alemd@172.22.0.161's password: 
cliente-vpn.key                           100% 1704   201.0KB/s   00:00    
ca.crt                                    100% 1224   138.4KB/s   00:00    
cliente-vpn.crt                           100% 4525   325.1KB/s   00:00   


alemd@debian:~$ scp -r cliente-vpn/ debian@172.22.201.177:
cliente-vpn.key                           100% 1704   125.8KB/s   00:00    
ca.crt                                    100% 1224     5.1KB/s   00:00    
cliente-vpn.crt                           100% 4525   329.6KB/s   00:00  
```

Una vez hecho esto, desde el lado del servidor aún, crearemos el archivo de configuración del tunel que crearemos a partir del fichero de ejemplo:

```txt
root@servidor-vpn:/home/debian# cp /usr/share/doc/openvpn/examples/sample-config-files/server.conf /etc/openvpn/server/servidor.conf
```

Lo editamos y el resultado es el siguiente:

```txt
root@servidor-vpn:/home/debian# nano /etc/openvpn/server/servidor.conf
```

```txt

port 1194
proto udp
dev tun

ca /etc/openvpn/easy-rsa/pki/ca.crt
cert /etc/openvpn/easy-rsa/pki/issued/server.crt
key /etc/openvpn/easy-rsa/pki/private/server.key  # This file should be kept
 secret
dh  /etc/openvpn/easy-rsa/pki/dh.pem

topology subnet

server 10.100.200.0 255.255.255.0
ifconfig-pool-persist /var/log/openvpn/ipp.txt

push "route 192.168.100.0 255.255.255.0"

keepalive 10 120
cipher AES-256-CBC
persist-key
persist-tun
status /var/log/openvpn/openvpn-status.log
verb 3

explicit-exit-notify 1
```

Ahora que hemos terminado de crear el fichero de configuración, habilitaremos el servicio:

```txt
root@servidor-vpn:/home/debian# systemctl enable --now openvpn-server@servidor
Created symlink /etc/systemd/system/multi-user.target.wants/openvpn-server@servidor.service → /lib/systemd/system/openvpn-server@.service.
```

Muestro el estado del servicio:

```txt
root@servidor-vpn:/home/debian# systemctl status openvpn-server@servidor

● openvpn-server@servidor.service - OpenVPN service for servidor
     Loaded: loaded (/lib/systemd/system/openvpn-server@.service; enabled; vendor preset: enabled)
     Active: active (running) since Fri 2023-01-13 11:37:51 UTC; 1min 8s ago
       Docs: man:openvpn(8)
             https://community.openvpn.net/openvpn/wiki/Openvpn24ManPage
             https://community.openvpn.net/openvpn/wiki/HOWTO
   Main PID: 22260 (openvpn)
     Status: "Initialization Sequence Completed"
      Tasks: 1 (limit: 527)
     Memory: 988.0K
        CPU: 14ms
     CGroup: /system.slice/system-openvpn\x2dserver.slice/openvpn-server@servidor.service
             └─22260 /usr/sbin/openvpn --status /run/openvpn-server/status-servidor.log --status-version 2 --suppress-timestamps --config servidor.conf

Jan 13 11:37:51 servidor-vpn openvpn[22260]: net_iface_up: set tun0 up
Jan 13 11:37:51 servidor-vpn openvpn[22260]: net_addr_v4_add: 10.100.200.1/24 dev tun0
Jan 13 11:37:51 servidor-vpn openvpn[22260]: Could not determine IPv4/IPv6 protocol. Using AF_INET
Jan 13 11:37:51 servidor-vpn openvpn[22260]: Socket Buffers: R=[212992->212992] S=[212992->212992]
Jan 13 11:37:51 servidor-vpn openvpn[22260]: UDPv4 link local (bound): [AF_INET][undef]:1194
Jan 13 11:37:51 servidor-vpn openvpn[22260]: UDPv4 link remote: [AF_UNSPEC]
Jan 13 11:37:51 servidor-vpn openvpn[22260]: MULTI: multi_init called, r=256 v=256
Jan 13 11:37:51 servidor-vpn openvpn[22260]: IFCONFIG POOL IPv4: base=10.100.200.2 size=252
Jan 13 11:37:51 servidor-vpn openvpn[22260]: IFCONFIG POOL LIST
Jan 13 11:37:51 servidor-vpn openvpn[22260]: Initialization Sequence Completed

```

##### Configuración del cliente VPN.

Ahora tomaremos el control del cliente e instalaremos openvpn:

```txt
debian@cliente-vpn:~$ sudo apt install openvpn
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  easy-rsa libccid liblzo2-2 libpcsclite1 libpkcs11-helper1 libusb-1.0-0
  opensc opensc-pkcs11 pcscd
Suggested packages:
  pcmciautils openvpn-systemd-resolved
The following NEW packages will be installed:
  easy-rsa libccid liblzo2-2 libpcsclite1 libpkcs11-helper1 libusb-1.0-0
  opensc opensc-pkcs11 openvpn pcscd
0 upgraded, 10 newly installed, 0 to remove and 0 not upgraded.
Need to get 2551 kB of archives.
After this operation, 7667 kB of additional disk space will be used.
Do you want to continue? [Y/n] y
```

Movemos el contenido del directorio que nos hemos pasado anteriormente por scp:

```txt
root@cliente-vpn:/home/debian# mv cliente-vpn/* /etc/openvpn/client/
```

Le cambiamos el propietario a root a los archivos que hemos movido:

```txt
root@cliente-vpn:/home/debian# chown root: /etc/openvpn/client/*
```

Creamos el siguiente archivo:

```txt
root@cliente-vpn:/home/debian# nano /etc/openvpn/client/cliente.conf
```

Le añadimos lo siguiente:

```txt
client
dev tun
proto udp

remote 10.0.0.15 1194
resolv-retry infinite
nobind
persist-key
persist-tun

ca /etc/openvpn/client/ca.crt
cert /etc/openvpn/client/cliente-vpn.crt
key /etc/openvpn/client/cliente-vpn.key

remote-cert-tls server
cipher AES-256-CBC
verb 3

```

Ahora miramos si el servicio está funcionando correctamente:

```txt
root@cliente-vpn:/etc/openvpn/client# systemctl status openvpn-client@cliente
● openvpn-client@cliente.service - OpenVPN tunnel for cliente
     Loaded: loaded (/lib/systemd/system/openvpn-client@.service; enabled; vendor preset: enabled)
     Active: active (running) since Fri 2023-01-13 19:44:09 UTC; 15s ago
       Docs: man:openvpn(8)
             https://community.openvpn.net/openvpn/wiki/Openvpn24ManPage
             https://community.openvpn.net/openvpn/wiki/HOWTO
   Main PID: 22514 (openvpn)
     Status: "Initialization Sequence Completed"
      Tasks: 1 (limit: 527)
     Memory: 1004.0K
        CPU: 22ms
     CGroup: /system.slice/system-openvpn\x2dclient.slice/openvpn-client@cliente.service
             └─22514 /usr/sbin/openvpn --suppress-timestamps --nobind --config cliente.conf

Jan 13 19:44:09 cliente-vpn openvpn[22514]: net_route_v4_best_gw query: dst 0.0.0.0
Jan 13 19:44:09 cliente-vpn openvpn[22514]: net_route_v4_best_gw result: via 10.0.0.1 dev ens3
Jan 13 19:44:09 cliente-vpn openvpn[22514]: ROUTE_GATEWAY 10.0.0.1/255.255.255.0 IFACE=ens3 HWADDR=fa:16:3e:7a:78:88
Jan 13 19:44:10 cliente-vpn openvpn[22514]: TUN/TAP device tun0 opened
Jan 13 19:44:10 cliente-vpn openvpn[22514]: net_iface_mtu_set: mtu 1500 for tun0
Jan 13 19:44:10 cliente-vpn openvpn[22514]: net_iface_up: set tun0 up
Jan 13 19:44:10 cliente-vpn openvpn[22514]: net_addr_v4_add: 10.100.200.2/24 dev tun0
Jan 13 19:44:10 cliente-vpn openvpn[22514]: net_route_v4_add: 192.168.100.0/24 via 10.100.200.1 dev [NULL] table 0 metric -1
Jan 13 19:44:10 cliente-vpn openvpn[22514]: WARNING: this configuration may cache passwords in memory -- use the auth-nocache option to prevent >
Jan 13 19:44:10 cliente-vpn openvpn[22514]: Initialization Sequence Completed
```

Como podemos ver, el servicio está activo y el servidor le ha asignado una ip correctamente. Ahora mostraré las interfaces de red del cliente y veremos que se ha creado un tun0 con la dirección ip de la vpn:

```txt
root@cliente-vpn:/etc/openvpn/client# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc pfifo_fast state UP group default qlen 1000
    link/ether fa:16:3e:7a:78:88 brd ff:ff:ff:ff:ff:ff
    altname enp0s3
    inet 10.0.0.186/24 brd 10.0.0.255 scope global dynamic ens3
       valid_lft 68878sec preferred_lft 68878sec
    inet6 fe80::f816:3eff:fe7a:7888/64 scope link 
       valid_lft forever preferred_lft forever
3: tun0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN group default qlen 500
    link/none 
    inet 10.100.200.2/24 scope global tun0
       valid_lft forever preferred_lft forever
    inet6 fe80::e7f5:181c:a293:e455/64 scope link stable-privacy 
       valid_lft forever preferred_lft forever
```

Ahora muestro las interfaces de red del servidor:

```txt
root@servidor-vpn:/home/debian# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc pfifo_fast state UP group default qlen 1000
    link/ether fa:16:3e:b6:d0:79 brd ff:ff:ff:ff:ff:ff
    altname enp0s3
    inet 192.168.100.66/24 brd 192.168.100.255 scope global dynamic ens3
       valid_lft 78208sec preferred_lft 78208sec
    inet6 fe80::f816:3eff:feb6:d079/64 scope link 
       valid_lft forever preferred_lft forever
3: ens4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc pfifo_fast state UP group default qlen 1000
    link/ether fa:16:3e:23:26:50 brd ff:ff:ff:ff:ff:ff
    altname enp0s4
    inet 10.0.0.15/24 brd 10.0.0.255 scope global dynamic ens4
       valid_lft 83174sec preferred_lft 83174sec
    inet6 fe80::f816:3eff:fe23:2650/64 scope link 
       valid_lft forever preferred_lft forever
4: tun0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN group default qlen 500
    link/none 
    inet 10.100.200.1/24 scope global tun0
       valid_lft forever preferred_lft forever
    inet6 fe80::c656:fa09:7f9c:ffb4/64 scope link stable-privacy 
       valid_lft forever preferred_lft forever
```

Ahora añadiremos un cliente conectado a la red de la vpn. Añadiremos una ruta estática por defecto para que use el servidor:

```txt
ip route del default
ip route add default via 192.168.11.10
```

**Atención he vuelto a hacer el escenario en Vagrant ya que en OpenStack da problemas al cambiar las rutas estáticas de la máquina.**

Hay que remarcar que en el servidor tenemos que añadir una regla de encaminamiento desde la red de la vpn a la red con la que queremos que se vea. La regla es la siguiente:

```txt
root@servidor-vpn:/home/vagrant# sudo iptables -A POSTROUTING -t nat -s 172.111.222.0/24 -o eth2 -j MASQUERADE
```


Muestro como desde el cliente externo a la red vpn realiza ping al cliente de la vpn:

```txt
vagrant@clientevpn:~$ ping 172.111.222.2
PING 172.111.222.2 (172.111.222.2) 56(84) bytes of data.
64 bytes from 172.111.222.2: icmp_seq=1 ttl=63 time=1.70 ms
64 bytes from 172.111.222.2: icmp_seq=2 ttl=63 time=3.05 ms
64 bytes from 172.111.222.2: icmp_seq=3 ttl=63 time=2.35 ms
64 bytes from 172.111.222.2: icmp_seq=4 ttl=63 time=2.78 ms
64 bytes from 172.111.222.2: icmp_seq=5 ttl=63 time=2.72 ms
```


Ahora muestro un traceroute:

```txt
vagrant@clientevpn:~$ traceroute 172.111.222.2
traceroute to 172.111.222.2 (172.111.222.2), 30 hops max, 60 byte packets
 1  10.100.200.1 (10.100.200.1)  0.349 ms  0.267 ms  0.306 ms
 2  172.111.222.2 (172.111.222.2)  2.544 ms  2.526 ms  2.547 ms
```


Ahora desde el cliente de la vpn hago ping y un traceroute al cliente externo a la vpn:

```txt
vagrant@clientevpn:~$ ping 10.100.200.10
PING 10.100.200.10 (10.100.200.10) 56(84) bytes of data.
64 bytes from 10.100.200.10: icmp_seq=1 ttl=63 time=3.63 ms
64 bytes from 10.100.200.10: icmp_seq=2 ttl=63 time=2.50 ms
64 bytes from 10.100.200.10: icmp_seq=3 ttl=63 time=3.00 ms
64 bytes from 10.100.200.10: icmp_seq=4 ttl=63 time=3.08 ms
^C
--- 10.100.200.10 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 2.497/3.053/3.633/0.402 ms
vagrant@clientevpn:~$ traceroute 10.100.200.10
traceroute to 10.100.200.10 (10.100.200.10), 30 hops max, 60 byte packets
 1  172.111.222.1 (172.111.222.1)  1.059 ms  0.955 ms  0.904 ms
 2  10.100.200.10 (10.100.200.10)  1.773 ms  1.640 ms  2.039 ms

```



#### 1.2. VPN sitio a sitio con OpenVPN y certificados x509. (10 puntos)

**Configura una conexión VPN sitio a sitio entre dos equipos del cloud:**

* **Cada equipo estará conectado a dos redes, una de ellas en común.**
* **Para la autenticación de los extremos se usarán obligatoriamente certificados digitales, que se generarán utilizando openssl y se almacenarán en el directorio /etc/openvpn, junto con con los parámetros Diffie-Helman y el certificado de la propia Autoridad de Certificación.**
* **Se utilizarán direcciones de la red 10.99.99.0/24 para las direcciones virtuales de la VPN.**
* **Tras el establecimiento de la VPN, una máquina de cada red detrás de cada servidor VPN debe ser capaz de acceder a una máquina del otro extremo.**
**Documenta el proceso detalladamente.**


En este apartado, necesitaremos dos clientes y dos servidores y tendremos que conectar los servidores a través de una vpn. Adjunto un esquema lógico para que se vea más claro:

![site2site.png](/img/site2site.png)

El escenario será realizado en Vagrant. Adjunto el Vagrantfile con el escenario completo:

```txt
Vagrant.configure("2") do |config|

    config.vm.define :nodo1 do |nodo1|
      nodo1.vm.box = "debian/bullseye64"
      nodo1.vm.hostname = "servidor-vpn"
      nodo1.vm.synced_folder ".", "/vagrant", disabled: true
      nodo1.vm.network :private_network,
      :libvirt__network_name => "net1",
      :libvirt__dhcp_enabled => false,
      :ip => "10.0.0.10",
      :libvirt__forward_mode => "veryisolated"
    end
  config.vm.define :nodo2 do |nodo2|
    nodo2.vm.box = "debian/bullseye64"
    nodo2.vm.hostname = "clientevpn"
    nodo2.vm.synced_folder ".", "/vagrant", disabled: true
    nodo2.vm.network :private_network,
      :libvirt__network_name => "net1",
      :libvirt__dhcp_enabled => false,
      :ip => "10.0.0.11",
      :libvirt__forward_mode => "veryisolated"
    end
   config.vm.define :nodo3 do |nodo3|
     nodo3.vm.box = "debian/bullseye64"
     nodo3.vm.hostname = "servidor2-vpn"
     nodo3.vm.synced_folder ".", "/vagrant", disabled: true      
     nodo3.vm.network :private_network,
      :libvirt__network_name => "net2",
      :libvirt__dhcp_enabled => false,
      :ip => "10.1.2.1",
      :libvirt__forward_mode => "veryisolated"
    end
  config.vm.define :nodo4 do |nodo4|
    nodo4.vm.box = "debian/bullseye64"
    nodo4.vm.hostname = "cliente2vpn"
    nodo4.vm.synced_folder ".", "/vagrant", disabled: true
    nodo4.vm.network :private_network,
      :libvirt__network_name => "net2",
      :libvirt__dhcp_enabled => false,
      :ip => "10.1.2.2",
      :libvirt__forward_mode => "veryisolated"
    end
  end
```

Como podemos ver, son dos servidores totalmente aislados con un cliente cada uno y lo que haremos será configurar una vpn para que ambas redes sean visibles entre sí.

##### Configuración del servidor1.

Lo primero que haremos, será instalar openvpn y configurar un fichero llamado vars, cogeremos de ejemplo un fichero que se encontrará en el mismo directorio:

```txt
vagrant@servidor-vpn:~$ sudo apt install openvpn
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  easy-rsa libccid liblzo2-2 libpcsclite1 libpkcs11-helper1 libusb-1.0-0
  opensc opensc-pkcs11 pcscd
Suggested packages:
  pcmciautils resolvconf openvpn-systemd-resolved
The following NEW packages will be installed:
  easy-rsa libccid liblzo2-2 libpcsclite1 libpkcs11-helper1 libusb-1.0-0
  opensc opensc-pkcs11 openvpn pcscd
0 upgraded, 10 newly installed, 0 to remove and 31 not upgraded.
Need to get 2551 kB of archives.
After this operation, 7667 kB of additional disk space will be used.
Do you want to continue? [Y/n] y
vagrant@servidor-vpn:~$ cd /usr/share/easy-rsa/
vagrant@servidor-vpn:/usr/share/easy-rsa$ sudo su
root@servidor-vpn:/usr/share/easy-rsa# cp vars.example vars
root@servidor-vpn:/usr/share/easy-rsa# nano vars
```

El archivo vars contiene lo siguiente:

```txt
set_var EASYRSA_REQ_COUNTRY     "ES"
set_var EASYRSA_REQ_PROVINCE    "Sevilla"
set_var EASYRSA_REQ_CITY        "Utrera"
set_var EASYRSA_REQ_ORG         "amontes"
set_var EASYRSA_REQ_EMAIL       "aaleemd11@gmail.com"
set_var EASYRSA_REQ_OU          "Practica VPN"
```

Ahora crearemos el directorio donde se almacenaran los documentos de la CA:

```txt
root@servidor-vpn:/usr/share/easy-rsa# ./easyrsa init-pki

Note: using Easy-RSA configuration from: /usr/share/easy-rsa/vars

init-pki complete; you may now create a CA or requests.
Your newly created PKI dir is: /usr/share/easy-rsa/pki


```

Ahora crearemos una clave Diffie-Hellman:

```txt
root@servidor-vpn:/usr/share/easy-rsa# ./easyrsa gen-dh

Note: using Easy-RSA configuration from: /usr/share/easy-rsa/vars
Using SSL: openssl OpenSSL 1.1.1n  15 Mar 2022
Generating DH parameters, 2048 bit long safe prime, generator 2
This is going to take a long time
........................+............................................................+....+..............................+.............................................................................................+..............................................+...........................................................................................................................................................................................................................+...................................

DH parameters of size 2048 created at /usr/share/easy-rsa/pki/dh.pem

```

Una vez hecho esto, crearemos la CA:

```txt
root@servidor-vpn:/usr/share/easy-rsa# ./easyrsa build-ca

Note: using Easy-RSA configuration from: /usr/share/easy-rsa/vars
Using SSL: openssl OpenSSL 1.1.1n  15 Mar 2022

Enter New CA Key Passphrase: 
Re-Enter New CA Key Passphrase: 
Generating RSA private key, 2048 bit long modulus (2 primes)
.....................+++++
......................................+++++
e is 65537 (0x010001)
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Common Name (eg: your user, host, or server name) [Easy-RSA CA]:Alejandro Montes

CA creation complete and you may now import and sign cert requests.
Your new CA certificate file for publishing is at:
/usr/share/easy-rsa/pki/ca.crt
```

Cuando tengamos lista nuestra CA, lo primero que haremos será crear y firmar el certificado que usará nuestra máquina servidor.

```txt
root@servidor-vpn:/usr/share/easy-rsa# ./easyrsa gen-req server

Note: using Easy-RSA configuration from: /usr/share/easy-rsa/vars
Using SSL: openssl OpenSSL 1.1.1n  15 Mar 2022
Generating a RSA private key
..........................+++++
.................................................................................................+++++
writing new private key to '/usr/share/easy-rsa/pki/easy-rsa-613.56pi8H/tmp.zJd9ro'
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Common Name (eg: your user, host, or server name) [server]:Alejandro Montes

Keypair and certificate request completed. Your files are:
req: /usr/share/easy-rsa/pki/reqs/server.req
key: /usr/share/easy-rsa/pki/private/server.key
```

Ahora firmamos el certificado:

```txt
root@servidor-vpn:/usr/share/easy-rsa# ./easyrsa sign-req server server

Note: using Easy-RSA configuration from: /usr/share/easy-rsa/vars
Using SSL: openssl OpenSSL 1.1.1n  15 Mar 2022


You are about to sign the following certificate.
Please check over the details shown below for accuracy. Note that this request
has not been cryptographically verified. Please be sure it came from a trusted
source or that you have verified the request checksum with the sender.

Request subject, to be signed as a server certificate for 825 days:

subject=
    commonName                = Alejandro Montes


Type the word 'yes' to continue, or any other input to abort.
  Confirm request details: yes
Using configuration from /usr/share/easy-rsa/pki/easy-rsa-635.4wZS7o/tmp.l26dgD
Enter pass phrase for /usr/share/easy-rsa/pki/private/ca.key:
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
commonName            :ASN.1 12:'Alejandro Montes'
Certificate is to be certified until Apr 29 09:05:56 2025 GMT (825 days)

Write out database with 1 new entries
Data Base Updated

Certificate created at: /usr/share/easy-rsa/pki/issued/server.crt
```

Tras crear y firmar el certificado, crearemos y firmaremos el certificado que usará el servidor2.:

```txt
root@servidor-vpn:/usr/share/easy-rsa# ./easyrsa gen-req vpn_server2

Note: using Easy-RSA configuration from: /usr/share/easy-rsa/vars
Using SSL: openssl OpenSSL 1.1.1n  15 Mar 2022
Generating a RSA private key
.....+++++
.............................................+++++
writing new private key to '/usr/share/easy-rsa/pki/easy-rsa-698.V9av96/tmp.uT7ClE'
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Common Name (eg: your user, host, or server name) [vpn_server2]:Servidor 2

Keypair and certificate request completed. Your files are:
req: /usr/share/easy-rsa/pki/reqs/vpn_server2.req
key: /usr/share/easy-rsa/pki/private/vpn_server2.key

```

Y lo firmamos:

```txt
root@servidor-vpn:/usr/share/easy-rsa# ./easyrsa sign-req client vpn_server2

Note: using Easy-RSA configuration from: /usr/share/easy-rsa/vars
Using SSL: openssl OpenSSL 1.1.1n  15 Mar 2022


You are about to sign the following certificate.
Please check over the details shown below for accuracy. Note that this request
has not been cryptographically verified. Please be sure it came from a trusted
source or that you have verified the request checksum with the sender.

Request subject, to be signed as a client certificate for 825 days:

subject=
    commonName                = Servidor 2


Type the word 'yes' to continue, or any other input to abort.
  Confirm request details: yes
Using configuration from /usr/share/easy-rsa/pki/easy-rsa-747.Dd9qde/tmp.YOHohQ
Enter pass phrase for /usr/share/easy-rsa/pki/private/ca.key:
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
commonName            :ASN.1 12:'Servidor 2'
Certificate is to be certified until Apr 29 09:10:37 2025 GMT (825 days)

Write out database with 1 new entries
Data Base Updated

Certificate created at: /usr/share/easy-rsa/pki/issued/vpn_server2.crt
```

Ahora copiaremos los ficheros a la ruta `/etc/openvpn` para que funcione el servidor:

```txt
root@servidor-vpn:/usr/share/easy-rsa/pki# cp ca.crt /etc/openvpn/server/
root@servidor-vpn:/usr/share/easy-rsa/pki# cp dh.pem /etc/openvpn/server/
root@servidor-vpn:/usr/share/easy-rsa/pki# cp issued/server.crt /etc/openvpn/server/
root@servidor-vpn:/usr/share/easy-rsa/pki# cp private/server.key /etc/openvpn/server/
```

Ahora pasamos al otro servidor los archivos necesarios para que se conecte al servidor vpn:

```txt
root@servidor-vpn:/usr/share/easy-rsa# scp pki/ca.crt vagrant@192.168.121.252:
ca.crt                                  100% 1224   672.0KB/s   00:00    
root@servidor-vpn:/usr/share/easy-rsa# scp pki/issued/vpn_server2.crt vagrant@192.168.121.252:
vpn_server2.crt                         100% 4527   134.1KB/s   00:00    
root@servidor-vpn:/usr/share/easy-rsa# scp pki/private/vpn_server2.key vagrant@192.168.121.252:
vpn_server2.key    
```

Una vez hayamos pasados los archivos al otro servidor y hayamos copiado los ficheros a su ruta correspondiente, crearemos el fichero de configuración del servidor vpn.


```txt
dev tun
ifconfig 100.0.123.1 100.0.123.2
route 10.1.2.0 255.255.255.0
tls-server
ca ca.crt
cert server.crt
key server.key
dh dh.pem
comp-lzo
keepalive 10 120
log /var/log/openvpn/server.log
verb 3
askpass passwd.txt
```

Creamos en /etc/openvpn/server un fichero llamado passwd.txt con la contraseña de frase de paso que hemos introducido anteriormente.

Iniciamos el servidor vpn:

```txt
root@servidor-vpn:/etc/openvpn/server# systemctl start openvpn-server@servidor
```

Ahora hacemos un status para ver que se ha iniciado correctamente:

```txt
● openvpn-server@servidor.service - OpenVPN service for servidor
     Loaded: loaded (/lib/systemd/system/openvpn-server@.service; disable>
     Active: active (running) since Wed 2023-01-25 12:21:45 UTC; 39s ago
       Docs: man:openvpn(8)
             https://community.openvpn.net/openvpn/wiki/Openvpn24ManPage
             https://community.openvpn.net/openvpn/wiki/HOWTO
   Main PID: 11960 (openvpn)
     Status: "Pre-connection initialization successful"
      Tasks: 1 (limit: 527)
     Memory: 936.0K
        CPU: 27ms
     CGroup: /system.slice/system-openvpn\x2dserver.slice/openvpn-server@>
             └─11960 /usr/sbin/openvpn --status /run/openvpn-server/statu>

Jan 25 12:21:44 servidor-vpn systemd[1]: Starting OpenVPN service for ser>
Jan 25 12:21:45 servidor-vpn openvpn[11960]: WARNING: Compression for rec>
Jan 25 12:21:45 servidor-vpn systemd[1]: Started OpenVPN service for serv>
```

Si no lo hemos hecho antes, activamos el bit de forwarding a 1:

```txt
root@servidor-vpn:/etc/openvpn/server# echo 1 > /proc/sys/net/ipv4/ip_forward
```

##### Configuración del cliente1.

Cambiaremos la ruta por defecto del cliente1:

```txt
vagrant@clientevpn:~$ sudo ip r del default
vagrant@clientevpn:~$ sudo ip r add default via 10.0.0.10
```

##### Configuración del servidor2.


Ahora configuraremos el servidor 2 que es el otro extremo del túnel vpn. Lo primero que haremos será instalar openvpn y mover los ficheros que pasamos anteriormente por scp a las rutas adecuadas:

```txt
vagrant@servidor2-vpn:~$ sudo apt install openvpn
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  easy-rsa libccid liblzo2-2 libpcsclite1 libpkcs11-helper1 libusb-1.0-0
  opensc opensc-pkcs11 pcscd
Suggested packages:
  pcmciautils resolvconf openvpn-systemd-resolved
The following NEW packages will be installed:
  easy-rsa libccid liblzo2-2 libpcsclite1 libpkcs11-helper1 libusb-1.0-0
  opensc opensc-pkcs11 openvpn pcscd
0 upgraded, 10 newly installed, 0 to remove and 0 not upgraded.
Need to get 2551 kB of archives.
After this operation, 7667 kB of additional disk space will be used.
Do you want to continue? [Y/n] y
vagrant@servidor2-vpn:~$ sudo su
root@servidor2-vpn:/home/vagrant# mv ca.crt /etc/openvpn/client/
root@servidor2-vpn:/home/vagrant# mv vpn_server2.crt /etc/openvpn/client/
root@servidor2-vpn:/home/vagrant# mv vpn_server2.key /etc/openvpn/client/
```

Creamos un archivo de configuración para este servidor:

```txt
root@servidor2-vpn:/home/vagrant# nano /etc/openvpn/client/client.conf
```

```txt
dev tun
remote 192.168.121.6
ifconfig 100.0.123.2 100.0.123.1
route 10.0.0.0 255.255.255.0
tls-client
ca ca.crt
cert vpn_server2.crt
key vpn_server2.key
comp-lzo
keepalive 10 60
verb 3
askpass passwd2.txt
```

Ahora arrancamos el servicio:

```txt
root@servidor2-vpn:/etc/openvpn/client# systemctl start openvpn-client@client
```

Muestro el estado del servicio:

```txt
openvpn-client@client.service - OpenVPN tunnel for client
     Loaded: loaded (/lib/systemd/system/openvpn-client@.service; disable>
     Active: active (running) since Wed 2023-01-25 14:25:56 UTC; 2min 14s>
       Docs: man:openvpn(8)
             https://community.openvpn.net/openvpn/wiki/Openvpn24ManPage
             https://community.openvpn.net/openvpn/wiki/HOWTO
   Main PID: 12472 (openvpn)
     Status: "Pre-connection initialization successful"
      Tasks: 1 (limit: 527)
     Memory: 1.1M
        CPU: 50ms
     CGroup: /system.slice/system-openvpn\x2dclient.slice/openvpn-client@>
             └─12472 /usr/sbin/openvpn --suppress-timestamps --nobind --c>

Jan 25 14:28:06 servidor2-vpn openvpn[12472]: ROUTE_GATEWAY 192.168.121.1>
Jan 25 14:28:06 servidor2-vpn openvpn[12472]: TUN/TAP device tun0 opened
Jan 25 14:28:06 servidor2-vpn openvpn[12472]: net_iface_mtu_set: mtu 1500>
Jan 25 14:28:06 servidor2-vpn openvpn[12472]: net_iface_up: set tun0 up
Jan 25 14:28:06 servidor2-vpn openvpn[12472]: net_addr_ptp_v4_add: 100.0.>
Jan 25 14:28:06 servidor2-vpn openvpn[12472]: net_route_v4_add: 10.0.0.0/>
Jan 25 14:28:06 servidor2-vpn openvpn[12472]: TCP/UDP: Preserving recentl>
Jan 25 14:28:06 servidor2-vpn openvpn[12472]: Socket Buffers: R=[212992->>
Jan 25 14:28:06 servidor2-vpn openvpn[12472]: UDP link local: (not bound)
Jan 25 14:28:06 servidor2-vpn openvpn[12472]: UDP link remote: [AF_INET]1>
```

No debemos olvidarnos de activar el bit de forwarding en este servidor también:

```txt
echo 1 > /proc/sys/net/ipv4/ip_forward
```

##### Configuración cliente 2.

Cambiamos la ruta por defecto del cliente2:

```txt
vagrant@cliente2vpn:~$ sudo ip r del default
vagrant@cliente2vpn:~$ sudo ip r add default via 10.1.2.1
```

##### Comprobaciones.

IPs:

Servidor1-vpn:10.0.0.10,192.168.121.6

Cliente1-vpn:10.0.0.11

Servidor2-vpn:10.1.2.1.1,192.168.121.252

Cliente2-vpn:10.1.2.1.2

Ping y traceroute desde el cliente del servidor 1 al cliente del servidor 2:

```txt
vagrant@clientevpn:~$ ping 10.1.2.2
PING 10.1.2.2 (10.1.2.2) 56(84) bytes of data.
64 bytes from 10.1.2.2: icmp_seq=1 ttl=62 time=3.13 ms
64 bytes from 10.1.2.2: icmp_seq=2 ttl=62 time=3.48 ms
64 bytes from 10.1.2.2: icmp_seq=3 ttl=62 time=3.55 ms
64 bytes from 10.1.2.2: icmp_seq=4 ttl=62 time=3.14 ms
^C
--- 10.1.2.2 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 3.130/3.325/3.549/0.191 ms
vagrant@clientevpn:~$ traceroute 10.1.2.2
traceroute to 10.1.2.2 (10.1.2.2), 30 hops max, 60 byte packets
 1  10.0.0.10 (10.0.0.10)  0.470 ms  0.414 ms  0.469 ms
 2  pool-100-0-123-2.bstnma.fios.verizon.net (100.0.123.2)  1.674 ms  3.207 ms  3.144 ms
 3  10.1.2.2 (10.1.2.2)  3.090 ms  3.072 ms  2.993 ms
```

Ping y traceroute desde el cliente del servidor 2 al cliente del servidor 1:


```txt
vagrant@cliente2vpn:~$ ping 10.0.0.11
PING 10.0.0.11 (10.0.0.11) 56(84) bytes of data.
64 bytes from 10.0.0.11: icmp_seq=1 ttl=62 time=3.55 ms
64 bytes from 10.0.0.11: icmp_seq=2 ttl=62 time=3.91 ms
64 bytes from 10.0.0.11: icmp_seq=3 ttl=62 time=3.89 ms
^C
--- 10.0.0.11 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 3.551/3.782/3.910/0.163 ms
vagrant@cliente2vpn:~$ traceroute 10.0.0.11
traceroute to 10.0.0.11 (10.0.0.11), 30 hops max, 60 byte packets
 1  10.1.2.1 (10.1.2.1)  0.577 ms  0.532 ms  0.476 ms
 2  lo0-100.BSTNMA-VFTTP-306.verizon-gni.net (100.0.123.1)  2.497 ms  2.832 ms  3.371 ms
 3  10.0.0.11 (10.0.0.11)  3.341 ms  3.210 ms  3.121 ms
```

### WireGuard

#### 2.1. VPN de acceso remoto con WireGuard. (5 puntos)

**Monta una VPN de acceso remoto usando Wireguard. Intenta probarla con clientes Windows, Linux y Android. Documenta el proceso adecuadamente y compáralo con el del apartado A.**


Para crear el escenario he usado este Vagrantfile:

```txt
Vagrant.configure("2") do |config|

    config.vm.define :nodo1 do |nodo1|
      nodo1.vm.box = "debian/bullseye64"
      nodo1.vm.hostname = "wireguard-server"
      nodo1.vm.synced_folder ".", "/vagrant", disabled: true
      nodo1.vm.network :private_network,
      :libvirt__network_name => "net1",
      :libvirt__dhcp_enabled => false,
      :ip => "192.168.100.1",
      :libvirt__forward_mode => "veryisolated"
      nodo1.vm.network :private_network,
      :libvirt__network_name => "net-privada",
      :libvirt__dhcp_enabled => false,
      :ip => "192.168.50.1",
      :libvirt__forward_mode => "veryisolated"

    end
    config.vm.define :nodo2 do |nodo2|
      nodo2.vm.box = "debian/bullseye64"
      nodo2.vm.hostname = "wireguard-client"
      nodo2.vm.synced_folder ".", "/vagrant", disabled: true
      nodo2.vm.network :private_network,
      :libvirt__network_name => "net1",
      :libvirt__dhcp_enabled => false,
      :ip => "192.168.100.2",
      :libvirt__forward_mode => "veryisolated"
    end
    config.vm.define :nodo3 do |nodo3|
      nodo3.vm.box = "debian/bullseye64"
      nodo3.vm.hostname = "private-client"
      nodo3.vm.synced_folder ".", "/vagrant", disabled: true
      nodo3.vm.network :private_network,
      :libvirt__network_name => "net-privada",
      :libvirt__dhcp_enabled => false,
      :ip => "192.168.50.10",
      :libvirt__forward_mode => "veryisolated"
    end
  end
```

##### Configuración del servidor.

Una vez creado el escenario, entraremos al servidor y lo configuraremos, para ello lo primero que haremos será instalar wireguard:


```txt
vagrant@wireguard-server:~$ sudo apt install wireguard
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  wireguard-tools
Suggested packages:
  openresolv | resolvconf
The following NEW packages will be installed:
  wireguard wireguard-tools
0 upgraded, 2 newly installed, 0 to remove and 31 not upgraded.
Need to get 94.3 kB of archives.
After this operation, 344 kB of additional disk space will be used.
Do you want to continue? [Y/n] y
```

Como he tenido problemas en el apartado anterior, lo primero que haremos será activar el bit de forwarding:


```txt
vagrant@wireguard-server:~$ sudo nano /etc/sysctl.conf
```

Y descomentamos la línea:

```txt
net.ipv4.ip_forward=1
```

Creamos un par de claves usando el siguiente comando:

```txt
vagrant@wireguard-server:~$ sudo su
root@wireguard-server:/home/vagrant# cd /etc/wireguard/
root@wireguard-server:/etc/wireguard# wg genkey | tee serverprivatekey | wg pubkey > serverpublickey
```

Creamos el archivo de configuración:

```txt
root@wireguard-server:/etc/wireguard# nano wg0.conf
```

```txt
[Interface]
Address = 10.0.100.1 #IP de la nueva interfaz
PrivateKey = yMILnmrL3ACptes5tfVdGQE7aeMRjX04t1KgOF811W8= #Clave privada generada anteriormente.
ListenPort = 51820 #Puerto de escucha.
```

Levantamos la interfaz creada con el siguiente comando:

```txt
root@wireguard-server:/etc/wireguard# wg-quick up wg0
[#] ip link add wg0 type wireguard
[#] wg setconf wg0 /dev/fd/63
[#] ip -4 address add 10.0.100.1 dev wg0
[#] ip link set mtu 1420 up dev wg0
```

Como vemos con el siguiente comanfo, la interfaz está activa:

```txt
root@wireguard-server:/etc/wireguard# wg
interface: wg0
  public key: bEnc2dZ0wBF6R3GBNmeGWidnyng+w4I2QZNFqFA2emA=
  private key: (hidden)
  listening port: 51820
```

Ahora que el servidor ya está configurado, procederemos a configurar los clientes:


##### Configuración cliente debian.

Instalamos wireguard y creamos el par de claves:

```txt
vagrant@wireguard-client:~$ sudo apt install wireguard
vagrant@wireguard-client:~$ sudo su
cdroot@wireguard-client:/home/vagrant# cd /etc/wireguard/
root@wireguard-client:/etc/wireguard# wg genkey | tee clientprivatekey | wg pubkey > clientpublickey
```

Creamos el fichero de configuración:

```txt
root@wireguard-client:/etc/wireguard# nano wg0.conf

[Interface]
Address = 10.0.100.2
PrivateKey = kDCxFoW5x4U41A826+d1l8k6J1frGKHYk5PsK2A7Y2k=
ListenPort = 51820

[Peer]
PublicKey = bEnc2dZ0wBF6R3GBNmeGWidnyng+w4I2QZNFqFA2emA=
AllowedIPs = 0.0.0.0/0
Endpoint = 192.168.100.1:51820
```

Ahora del lado del servidor editamos la configuración y añadimos un bloque peer:

```txt
root@wireguard-server:/etc/wireguard# nano wg0.conf 


[Interface]
Address = 10.0.100.1
PrivateKey = 6OC4j5vEf+qxlROFk+6V8GSKc+wBJkHpDtLtCFLUG3s=
ListenPort = 51820

[Peer]
Publickey = 3VMhNkz7uu2NdG4kv59RPyZZmmF/00MZvvbnt5tyx38=
AllowedIPs = 10.0.100.2/32
PersistentKeepAlive = 25
```

Desactivamos y activamos la interfaz de wireguard tanto en el cliente como en el servidor:

```txt
root@wireguard-server:/etc/wireguard# wg-quick down wg0
[#] ip link delete dev wg0
root@wireguard-server:/etc/wireguard# wg-quick up wg0
[#] ip link add wg0 type wireguard
[#] wg setconf wg0 /dev/fd/63
[#] ip -4 address add 10.0.100.1 dev wg0
[#] ip link set mtu 1420 up dev wg0
[#] ip -4 route add 10.0.100.2/32 dev wg0
```

```txt
root@wireguard-server:/etc/wireguard# wg
interface: wg0
  public key: B3YLtNXGjfm5H+4DIz/rs55TtJ1T65IB1q7rpS1IqBA=
  private key: (hidden)
  listening port: 51820

peer: 3VMhNkz7uu2NdG4kv59RPyZZmmF/00MZvvbnt5tyx38=
  endpoint: 192.168.100.2:51820
  allowed ips: 10.0.100.2/32
  latest handshake: 15 seconds ago
  transfer: 180 B received, 124 B sent
  persistent keepalive: every 25 seconds
```

Ahora el cliente:

```txt
root@wireguard-client:/etc/wireguard# wg-quick down wg0
[#] ip -4 rule delete table 51820
[#] ip -4 rule delete table main suppress_prefixlength 0
[#] ip link delete dev wg0
[#] nft -f /dev/fd/63
root@wireguard-client:/etc/wireguard# wg-quick up wg0
[#] ip link add wg0 type wireguard
[#] wg setconf wg0 /dev/fd/63
[#] ip -4 address add 10.0.100.2 dev wg0
[#] ip link set mtu 1420 up dev wg0
[#] wg set wg0 fwmark 51820
[#] ip -4 route add 0.0.0.0/0 dev wg0 table 51820
[#] ip -4 rule add not fwmark 51820 table 51820
[#] ip -4 rule add table main suppress_prefixlength 0
[#] sysctl -q net.ipv4.conf.all.src_valid_mark=1
[#] nft -f /dev/fd/63
```

```txt
root@wireguard-client:/etc/wireguard# wg
interface: wg0
  public key: 3VMhNkz7uu2NdG4kv59RPyZZmmF/00MZvvbnt5tyx38=
  private key: (hidden)
  listening port: 51820
  fwmark: 0xca6c

peer: B3YLtNXGjfm5H+4DIz/rs55TtJ1T65IB1q7rpS1IqBA=
  endpoint: 192.168.100.1:51820
  allowed ips: 0.0.0.0/0
  latest handshake: 33 seconds ago
  transfer: 156 B received, 612 B sent
```

##### Cliente Debian.

Para que el cliente interno Debian se vea con la vpn tendremos que cambiarle la ruta por defecto:

```txt
vagrant@private-client:~$ sudo ip r del default
vagrant@private-client:~$ sudo ip r add default via 192.168.50.1
```

##### Comprobaciones.

Ping y traceroute desde el cliente interno (Debian) al cliente vpn:

```txt
vagrant@private-client:~$ ping 10.0.100.2
PING 10.0.100.2 (10.0.100.2) 56(84) bytes of data.
64 bytes from 10.0.100.2: icmp_seq=1 ttl=63 time=1.06 ms
64 bytes from 10.0.100.2: icmp_seq=2 ttl=63 time=2.16 ms
64 bytes from 10.0.100.2: icmp_seq=3 ttl=63 time=2.49 ms
64 bytes from 10.0.100.2: icmp_seq=4 ttl=63 time=2.06 ms
^C
--- 10.0.100.2 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3006ms
rtt min/avg/max/mdev = 1.056/1.939/2.486/0.534 ms
vagrant@private-client:~$ traceroute 10.0.100.2
traceroute to 10.0.100.2 (10.0.100.2), 30 hops max, 60 byte packets
 1  192.168.50.1 (192.168.50.1)  0.641 ms  0.622 ms  0.700 ms
 2  10.0.100.2 (10.0.100.2)  1.899 ms  1.976 ms  2.234 ms
```


Ping y traceroute desde el cliente vpn al cliente interno(Debian):

```txt
oot@wireguard-client:/etc/wireguard# ping 192.168.50.10
PING 192.168.50.10 (192.168.50.10) 56(84) bytes of data.
64 bytes from 192.168.50.10: icmp_seq=1 ttl=63 time=2.11 ms
64 bytes from 192.168.50.10: icmp_seq=2 ttl=63 time=2.81 ms
64 bytes from 192.168.50.10: icmp_seq=3 ttl=63 time=2.31 ms
64 bytes from 192.168.50.10: icmp_seq=4 ttl=63 time=1.03 ms
^C
--- 192.168.50.10 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3006ms
rtt min/avg/max/mdev = 1.027/2.062/2.806/0.649 ms
root@wireguard-client:/etc/wireguard# traceroute 192.168.50.10
traceroute to 192.168.50.10 (192.168.50.10), 30 hops max, 60 byte packets
 1  10.0.100.1 (10.0.100.1)  0.905 ms  0.672 ms  0.462 ms
 2  192.168.50.10 (192.168.50.10)  1.202 ms  1.026 ms  0.953 ms
```

##### Cliente Windows.

Para windows, nos descargaremos el programa desde su página oficial, es un ejecutable y se instala de manera muy sencilla, no veo necesario poner los pasos a seguir para la instalación.

Una vez hecho esto, le damos a crear nuevo túnel y nos aparecerá lo siguiente:

![SAD-P6.4.png](/img/SAD-P6.4.png)

Crearemos el archivo de configuración del cliente vpn.

![SAD-P6.5.png](/img/SAD-P6.5.png)

Añadimos al servidor otro peer con los datos del cliente windows. La configuración quedaría así:

```txt
[Interface]
Address = 10.0.100.1
PrivateKey = 6OC4j5vEf+qxlROFk+6V8GSKc+wBJkHpDtLtCFLUG3s=
ListenPort = 51820

[Peer]
Publickey = 3VMhNkz7uu2NdG4kv59RPyZZmmF/00MZvvbnt5tyx38=
AllowedIPs = 10.0.100.2/32
PersistentKeepAlive = 25

[Peer]
Publickey = ARu1pvsyDprc147XdaBNliCeuBl8mtnY65xIYR2UPio=
AllowedIPs = 10.0.100.3/32
PersistentKeepAlive = 25

```

Si  en el cliente pulsamos activate, habilitamos la vpn:

![SAD-P6.6.png](/img/SAD-P6.6.png)

##### Comprobaciones Cliente VPN Windows.

Lo primero que haremos será hacer ping y traceroute desde el cliente windows al cliente interno:

![SAD-P6.7.png](/img/SAD-P6.7.png)

Ahora desde el cliente interno ping y traceroute al cliente vpn windows:

```txt
vagrant@private-client:~$ ping 10.0.100.3
PING 10.0.100.3 (10.0.100.3) 56(84) bytes of data.
64 bytes from 10.0.100.3: icmp_seq=1 ttl=127 time=2.29 ms
64 bytes from 10.0.100.3: icmp_seq=2 ttl=127 time=11.8 ms
64 bytes from 10.0.100.3: icmp_seq=3 ttl=127 time=4.78 ms
64 bytes from 10.0.100.3: icmp_seq=4 ttl=127 time=1.83 ms
^C
--- 10.0.100.3 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 1.834/5.178/11.816/3.992 ms
vagrant@private-client:~$ traceroute 10.0.100.3
traceroute to 10.0.100.3 (10.0.100.3), 30 hops max, 60 byte packets
 1  192.168.50.1 (192.168.50.1)  0.514 ms  0.621 ms  0.571 ms
 2  10.0.100.3 (10.0.100.3)  12.915 ms * *
vagrant@private-client:~$ 
```

Por último desde el otro cliente de la vpn también probaremos que tiene conexión:

```txt
root@wireguard-client:/home/vagrant# ping 10.0.100.3
PING 10.0.100.3 (10.0.100.3) 56(84) bytes of data.
64 bytes from 10.0.100.3: icmp_seq=1 ttl=127 time=2.98 ms
64 bytes from 10.0.100.3: icmp_seq=2 ttl=127 time=11.6 ms
64 bytes from 10.0.100.3: icmp_seq=3 ttl=127 time=3.14 ms
64 bytes from 10.0.100.3: icmp_seq=4 ttl=127 time=1.96 ms
^C
--- 10.0.100.3 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 1.964/4.924/11.618/3.890 ms
root@wireguard-client:/home/vagrant# traceroute 10.0.100.3
traceroute to 10.0.100.3 (10.0.100.3), 30 hops max, 60 byte packets
 1  10.0.100.1 (10.0.100.1)  1.913 ms  1.565 ms  1.451 ms
 2  10.0.100.3 (10.0.100.3)  4.352 ms * *

```


#### 2.2. VPN sitio a sitio con WireGuard. (10 puntos)

**Configura una VPN sitio a sitio usando WireGuard. Documenta el proceso adecuadamente y compáralo con el del apartado B.**

Lo primero que haré será copiar el escenario del apartado B que es openvpn site to site, será exactamente el mismo cambiando openvpn por wireguard. No adjunto el vagrant ya que está más arriba.

Para no olvidarnos lo primero que haremos será cambiar la puerta de enlace de los dos clientes:

```txt
vagrant@clientevpn:~$ sudo ip r del default
vagrant@clientevpn:~$ sudo ip r add default via 10.0.0.10

```

Cliente2:

```txt
vagrant@cliente2vpn:~$ sudo ip r del default
vagrant@cliente2vpn:~$ sudo ip r add default via 10.1.2.1

```

##### Configuración Servidor1.

En el servidor1, instalaremos wireguard y activaremos el bit de forwarding:

```txt
vagrant@servidor-vpn:~$ sudo apt install wireguard
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  wireguard-tools
Suggested packages:
  openresolv | resolvconf
The following NEW packages will be installed:
  wireguard wireguard-tools
0 upgraded, 2 newly installed, 0 to remove and 31 not upgraded.
Need to get 94.3 kB of archives.
After this operation, 344 kB of additional disk space will be used.
Do you want to continue? [Y/n] y
```

Activamos el bit de forwarding:

```txt
root@servidor-vpn:/home/vagrant# nano /etc/sysctl.conf 


#Uncoment
net.ipv4.ip_forward=1
```

Nos movemos al directorio de wireguard y generamos el par de claves:

```txt
root@servidor-vpn:/home/vagrant# cd /etc/wireguard/
root@servidor-vpn:/etc/wireguard# wg genkey | tee serverprivatekey | wg pubkey > serverpublickey
root@servidor-vpn:/etc/wireguard# cat serverpublickey 
mTfMLCkgFZboI9Pi+35malydyOVVFIiAPZvMG0QNhnA=
root@servidor-vpn:/etc/wireguard# cat serverprivatekey 
mC4yoTaz9iVpXW2i4ykl+3u68SqIVbDOS71wQrZ6WG0=

```

Creamos el fichero de configuración:

```txt
root@servidor-vpn:/etc/wireguard# nano wg0.conf

[Interface]
Address = 10.0.100.1
PrivateKey = mC4yoTaz9iVpXW2i4ykl+3u68SqIVbDOS71wQrZ6WG0=
ListenPort = 51820
```

Iniciamos la vpn para probar que funciona:

```txt
root@servidor-vpn:/etc/wireguard# wg-quick up wg0
[#] ip link add wg0 type wireguard
[#] wg setconf wg0 /dev/fd/63
[#] ip -4 address add 10.0.100.1 dev wg0
[#] ip link set mtu 1420 up dev wg0
root@servidor-vpn:/etc/wireguard# wg
interface: wg0
  public key: mTfMLCkgFZboI9Pi+35malydyOVVFIiAPZvMG0QNhnA=
  private key: (hidden)
  listening port: 51820

```

##### Configuración servidor2.

Ahora replicaremos los pasos que hemos hecho en el servidor1, instalaremos wireguard y activaremos el bit de forwarding.

```txt
vagrant@servidor2-vpn:~$ sudo apt install wireguard
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  wireguard-tools
Suggested packages:
  openresolv | resolvconf
The following NEW packages will be installed:
  wireguard wireguard-tools
0 upgraded, 2 newly installed, 0 to remove and 31 not upgraded.
Need to get 94.3 kB of archives.
After this operation, 344 kB of additional disk space will be used.
Do you want to continue? [Y/n] y
```

Activamos el bit de forwarding:

```txt
vagrant@servidor2-vpn:~$ sudo nano /etc/sysctl.conf 

# Uncomment the next line to enable packet forwarding for IPv4
net.ipv4.ip_forward=1

```

Nos conectamos como super usuario, nos movemos al directorio de wireguard y generamos el par de claves:

```txt
root@servidor2-vpn:/home/vagrant# cd /etc/wireguard/
root@servidor2-vpn:/etc/wireguard# wg genkey | tee clientprivatekey | wg pubkey > clientpublickey
root@servidor2-vpn:/etc/wireguard# cat clientpublickey 
E4obRhgP0i+SvbDf8uKfGnZFNjKmqH/E+ms7WiL9Qjw=
root@servidor2-vpn:/etc/wireguard# cat clientprivatekey 
WBGshn38JwiYiFhJnVocCKyGpGjL0wgvMviRIPLmEXE=
```

Creamos el fichero de configuración del Servidor2.

```txt
root@servidor2-vpn:/etc/wireguard# sudo nano wg0.conf
```

```txt
[Interface]
Address = 10.0.100.2
PrivateKey = WBGshn38JwiYiFhJnVocCKyGpGjL0wgvMviRIPLmEXE=
ListenPort = 51820

[Peer]
PublicKey = mTfMLCkgFZboI9Pi+35malydyOVVFIiAPZvMG0QNhnA=
AllowedIPs = 0.0.0.0/0
Endpoint = 192.168.121.225:51820

```

Ahora modificaremos el servidor1 para añadir un peer con los datos del servidor2:

```txt
root@servidor-vpn:/etc/wireguard# nano wg0.conf
```

El fichero quedará así:

```txt
[Interface]
Address = 10.0.100.1
PrivateKey = mC4yoTaz9iVpXW2i4ykl+3u68SqIVbDOS71wQrZ6WG0=
ListenPort = 51820

[Peer]
Publickey = E4obRhgP0i+SvbDf8uKfGnZFNjKmqH/E+ms7WiL9Qjw=
AllowedIPs = 0.0.0.0/0
PersistentKeepAlive = 25

```

Reiniciamos la interfaz de la vpn del servidor1:

```txt
root@servidor-vpn:/etc/wireguard# wg-quick down wg0
[#] ip link delete dev wg0
root@servidor-vpn:/etc/wireguard# wg-quick up wg0
[#] ip link add wg0 type wireguard
[#] wg setconf wg0 /dev/fd/63
[#] ip -4 address add 10.0.100.1 dev wg0
[#] ip link set mtu 1420 up dev wg0
[#] wg set wg0 fwmark 51820
[#] ip -4 route add 0.0.0.0/0 dev wg0 table 51820
[#] ip -4 rule add not fwmark 51820 table 51820
[#] ip -4 rule add table main suppress_prefixlength 0
[#] sysctl -q net.ipv4.conf.all.src_valid_mark=1
[#] nft -f /dev/fd/63

```

Iniciamos la interfaz del servidor2:

```txt
root@servidor2-vpn:/etc/wireguard# wg-quick down wg0
wg-quick: `wg0' is not a WireGuard interface
root@servidor2-vpn:/etc/wireguard# wg-quick up wg0
[#] ip link add wg0 type wireguard
[#] wg setconf wg0 /dev/fd/63
[#] ip -4 address add 10.0.100.2 dev wg0
[#] ip link set mtu 1420 up dev wg0
[#] wg set wg0 fwmark 51820
[#] ip -4 route add 0.0.0.0/0 dev wg0 table 51820
[#] ip -4 rule add not fwmark 51820 table 51820
[#] ip -4 rule add table main suppress_prefixlength 0
[#] sysctl -q net.ipv4.conf.all.src_valid_mark=1
[#] nft -f /dev/fd/63
```

Muestro el estado de conexión de ambos servidores:

Servidor1:

```txt
root@servidor-vpn:/etc/wireguard# wg
interface: wg0
  public key: mTfMLCkgFZboI9Pi+35malydyOVVFIiAPZvMG0QNhnA=
  private key: (hidden)
  listening port: 51820
  fwmark: 0xca6c

peer: E4obRhgP0i+SvbDf8uKfGnZFNjKmqH/E+ms7WiL9Qjw=
  endpoint: 192.168.121.31:51820
  allowed ips: 0.0.0.0/0
  latest handshake: 8 seconds ago
  transfer: 276 B received, 124 B sent
  persistent keepalive: every 25 seconds
```

Servidor2:

```txt
root@servidor2-vpn:/etc/wireguard# wg-quick up wg0
[#] ip link add wg0 type wireguard
[#] wg setconf wg0 /dev/fd/63
[#] ip -4 address add 10.0.100.2 dev wg0
[#] ip link set mtu 1420 up dev wg0
[#] wg set wg0 fwmark 51820
[#] ip -4 route add 0.0.0.0/0 dev wg0 table 51820
[#] ip -4 rule add not fwmark 51820 table 51820
[#] ip -4 rule add table main suppress_prefixlength 0
[#] sysctl -q net.ipv4.conf.all.src_valid_mark=1
[#] nft -f /dev/fd/63
```

##### Comprobaciones.


Las IP de los equipos son:

Servidor1: 10.0.0.10,192.168.121.225

Cliente1: 10.0.0.11

Servidor2: 10.1.2.1, 192.168.121.31

Cliente2: 10.1.2.2

Desde el cliente del servidor1 (cliente1) hago ping y traceroute al cliente del servidor2 (cliente2):

```txt
vagrant@clientevpn:~$ ping 10.1.2.2
PING 10.1.2.2 (10.1.2.2) 56(84) bytes of data.
64 bytes from 10.1.2.2: icmp_seq=1 ttl=62 time=3.47 ms
64 bytes from 10.1.2.2: icmp_seq=2 ttl=62 time=3.42 ms
64 bytes from 10.1.2.2: icmp_seq=3 ttl=62 time=3.87 ms
64 bytes from 10.1.2.2: icmp_seq=4 ttl=62 time=3.93 ms
^C
--- 10.1.2.2 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 3.424/3.673/3.933/0.228 ms
vagrant@clientevpn:~$ traceroute 10.1.2.2
traceroute to 10.1.2.2 (10.1.2.2), 30 hops max, 60 byte packets
 1  10.0.0.10 (10.0.0.10)  0.626 ms  0.557 ms  0.526 ms
 2  10.0.100.2 (10.0.100.2)  1.445 ms  1.356 ms  2.004 ms
 3  10.1.2.2 (10.1.2.2)  2.759 ms  2.620 ms  2.521 ms

```

Ahora, desde el cliente del servidor2 (cliente2) hago ping y traceroute al cliente del servidor1 (cliente1):


```txt
agrant@cliente2vpn:~$ ping 10.0.0.11
PING 10.0.0.11 (10.0.0.11) 56(84) bytes of data.
64 bytes from 10.0.0.11: icmp_seq=1 ttl=62 time=3.44 ms
64 bytes from 10.0.0.11: icmp_seq=2 ttl=62 time=3.69 ms
64 bytes from 10.0.0.11: icmp_seq=3 ttl=62 time=3.83 ms
64 bytes from 10.0.0.11: icmp_seq=4 ttl=62 time=3.81 ms
^C
--- 10.0.0.11 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3004ms
rtt min/avg/max/mdev = 3.438/3.690/3.830/0.155 ms
vagrant@cliente2vpn:~$ traceroute 10.0.0.11
traceroute to 10.0.0.11 (10.0.0.11), 30 hops max, 60 byte packets
 1  10.1.2.1 (10.1.2.1)  0.451 ms  0.470 ms  0.454 ms
 2  10.0.100.1 (10.0.100.1)  1.640 ms  2.705 ms  2.682 ms
 3  10.0.0.11 (10.0.0.11)  4.315 ms  4.267 ms  4.207 ms

```

### Comparaciones

#### 3.1. Comparación entre OpenVPN y Wireguard acceso remoto.

Wireguard es más rápido que OpenVPN y un 15% mas eficiente a la hora de consumir datos, pero en cambio yo personalmente veo a OpenVPN más estable y seguro. En mi experiencia montando ambos servidores, la configuración de OpenVPN parece más complicada pero no se el motivo me ha dado más dolores de cabeza Wireguard que OpenVPN, ya que OpenVPN prácticamente la monté al 2º intento y Wireguard concretamente la de Acceso remoto me ha tardado bastante en funcionar. También están las diferencias de configuración que no creo que sea necesario nombrar ya que las acabamos de ver.

#### 3.2. Comparación entre OpenVPN y Wireguard site to site.

En la VPN site to site también sigue la misma dinámica que el anterior, Wireguard bastante más rápido y eficiente y OpenVPN más estable y seguro. Al montar los servidores Site to Site, sí me ha gustado más Wireguard la configuración ha sido igual de sencilla que el anterior pero sin saber el motivo ha funcionado a la primera y sin ningún tipo de fallo en la vpn, OpenVPN también ha funcionado correctamente y ha sido más difícil de configurar que Wireguard pero tampoco ha sido algo imposible. 

### Conclusión

Como conclusión si tengo que montar una VPN, según el entorno en que me encuentre elegiría uno u otro, es decir, si tengo que montar una VPN en una empresa elegiría OpenVPN ya que aunque pierde un poco de rendimiento prima la privacidad y la seguridad, y si la VPN es para uso personal, probablemente me decantaría por Wireguard ya que es más fácil de configurar y es más eficiente.

---

## **Documento realizado por:**

 ✒️ **Alejandro Montes Delgado** - *2º ASIR*
