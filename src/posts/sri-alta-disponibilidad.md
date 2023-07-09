---
layout: post
title: "Práctica: Cluster de Alta Disponibilidad"
excerpt: En esta práctica vamos a configurar un cluster en alta disponibilidad.
date: 2023-03-06
updatedDate: 2023-03-06
tags:
  - post
  - SRI
---

### Cluster de HA activo-pasivo.

#### 1. Utiliza el Vagrantfile la receta ansible del escenario 7: 07-HA-IPFailover-Apache2+DRBD+GFS2 para crear un clúster de alta disponibilidad activo-pasivo. Nota: La receta instala apache2 + php.
#### 2. Comprueba que los recursos están configurados de manera adecuada, configura tu host para que use el servidor DNS y comprueba que puedes acceder de forma adecuada a la página.
#### 3. Instala en los dos nodos un Galera MariaDB.
#### 4. Instala Wordpress en el clúster.

---

Una vez levantado el escenario con vagrant y hayamos pasado la receta de ansible, podemos ver que se ha configurado el cluster y se han activado los servicios:

```txt
root@nodo1:/home/vagrant# pcs status
Cluster name: mycluster
Cluster Summary:
  * Stack: corosync
  * Current DC: nodo1 (version 2.0.5-ba59be7122) - partition with quorum
  * Last updated: Mon Mar  6 11:54:13 2023
  * Last change:  Mon Mar  6 11:34:41 2023 by root via cibadmin on nodo1
  * 2 nodes configured
  * 5 resource instances configured

Node List:
  * Online: [ nodo1 nodo2 ]

Full List of Resources:
  * VirtualIP	(ocf::heartbeat:IPaddr2):	 Started nodo1
  * WebSite	(ocf::heartbeat:apache):	 Started nodo1
  * Clone Set: WebData-clone [WebData] (promotable):
    * Masters: [ nodo1 ]
    * Slaves: [ nodo2 ]
  * WebFS	(ocf::heartbeat:Filesystem):	 Started nodo1

Daemon Status:
  corosync: active/enabled
  pacemaker: active/enabled
  pcsd: active/enabled

```

Configuramos el dns de nuestra máquina con la ip interna que tiene la máquina dns del escenario(no la ip por defecto de vagrant) y comprobamos que podemos acceder a la página:


![SRI-AD.1.png](/img/SRI-AD.1.png)

Si apagamos el nodo1, nos visualizará el nodo2:

![SRI-AD.2.png](/img/SRI-AD.2.png)

Ahora procederemos a instalar en ambos nodos mariadb y ejecutaremos el script mysql_secure_installation. En el nodo 1 instalaremos el cluster, para ello pararemos la base de datos y editaremos el fichero galera:

```txt
root@nodo1:~# systemctl stop mariadb.service
root@nodo1:~# nano /etc/mysql/mariadb.conf.d/60-galera.cnf

[galera]
# Mandatory settings
wsrep_on                 = ON
wsrep_cluster_name       = "MariaDB Galera Cluster"
wsrep_cluster_address    = gcomm://10.1.1.101,10.1.1.102
binlog_format            = row
default_storage_engine   = InnoDB
innodb_autoinc_lock_mode = 2

# Allow server to accept connections on all interfaces.
bind-address = 0.0.0.0
wsrep-NODE-address=10.1.1.101

```



### Cluster de HA activo-activo.

**Siguiendo las instrucciones que encuentras en el escenario 7: 07-HA-IPFailover-Apache2+DRBD+GFS2 convierte el clúster en activo-activo. Es necesario instalar el fencing para que el clúster funcione de manera adecuada. Nota: Tienes que tener en cuenta que se va a formatear de nuevo el drbd, por lo que se va a perder el wordpress. Si quieres puedes guardarlo en otro directorio, para luego recuperarlo.**

**Una vez que el clúster este configurado como activo-activo y WordPress esté funcionado, configura un método de balanceo de carga:**

* **Balanceo por DNS: Podríamos quitar el recurso VirtualIP y hacer un balanceo de carga por DNS como vimos en el escenario 1 (1 punto) o el escenario 2 (2 puntos).**
* **Añadir un balanceador de carga HAProxy (que balancee la carga entre los dos servidores web) (2 puntos).**
* **Podrías instalar un HAProxy en los dos nodos y crear un recurso del clúster para que los controle. Para ello habría que crear un recurso con pacemaker para controlar los balanceadores de carga (el recurso se llama systemd:happroxy). Puedes seguir de base el artículo How to setup highly available Pacemaker/Corosync cluster with HAProxy load balancer (3 puntos).**


---
## **Documento realizado por:**

 ✒️ **Alejandro Montes Delgado** - *2º ASIR*