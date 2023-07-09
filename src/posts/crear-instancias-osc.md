---
layout: post
title: "Creación de instancias desde OSC"
excerpt: Vamos a trabajar con OpenStack Client.
date: 2022-11-03
updatedDate: 2022-11-03
tags:
  - post
  - SRI
---
1.  Instala el cliente de línea de comando de OpenStack cómo se muestra en [Instalación y uso básico de OpenStack client (OSC)](https://github.com/josedom24/curso_openstack_ies/blob/main/modulo1/osc.md).

Primero vamos a crear un entorno virtual donde vamos a trabajar con OpenStack client, para ello instalaremos python y venv si es necesario.

```bash
sudo apt install python3 python3-venv
mkdir venv
cd venv
```
Activamos el entorno virtual con el siguiente comando:

```bash
mkdir osc
python3 -m venv osc
```
Usamos el siguiente comando para instalar en el entorno virtual el cliente de openstack:

```bash
pip install python-openstackclient==5.8.0
```

3.  Descarga de Horizon tu fichero de credenciales, cargálo y ejecuta la instrucción `openstack server list` para visualizar tus instancias.

Una vez descargado el fichero, realizamos lo siguiente:
```bash
source crendenciales.sh
```
tendríamos que insertar la contraseña con la que entramos a openstack y ya podríamos interactuar con el servidor openstack.

5.  Vamos a crear una instancia y la vamos a configurar con [**cloud-init**](https://github.com/josedom24/curso_openstack_ies/blob/main/modulo3/cloudinit.md), para ello crea un fichero `cloud-config.yaml` con el siguiente contenido:

```yaml
    #cloud-config
         package_update: true
         package_upgrade: true
         preserve_hostname: false
         fqdn: test1.gonzalonazareno.org
         hostname: prueba-alemd
         # Crear un usuario y añadir clave pública ssh
         users:
           - name: jose
             sudo: ALL=(ALL) NOPASSWD:ALL
             shell: /bin/bash
             passwd: asdasd
             ssh_authorized_keys:
             # Clave pública de nuestra máquina.
               - ssh-rsa 
           - name: alemd
             sudo: ALL=(ALL) NOPASSWD:ALL
             shell: /bin/bash
             passwd: usuario
           #  ssh_authorized_keys:
             # Clave pública de nuestra máquina.
             #  - ssh-rsa 

         chpasswd:
           list: |
             root:asdasd
           expire: False
```

Modifica el fichero e indica un nombre a la máquina, crea otro usuario y cambia las contraseñas.
    
Este fichero actualizará la paquetería de la instancia, configurará el FQDN de la instancia, creará un nuevo usuario y cambiará la contraseña del usuario `root`.
    
4.  Siguiendo el apartado [Gestión de instancia con OpenStack client (OSC)](https://github.com/josedom24/curso_openstack_ies/blob/main/modulo3/osc_nova.md), visualiza con el OSC los flavors disponibles, imágenes disponibles, claves disponibles, reglas de cortafuegos,…
5.  Crea una nueva instancia usando el comando que encontrarás en la sección [Configuración de instancias con cloud-init](https://github.com/josedom24/curso_openstack_ies/blob/main/modulo1/osc.md).
6.  Añade una IP flotante a la instancia para ello puedes usar los comandos que encuentras en [Gestión de instancia con OpenStack client (OSC)](https://github.com/josedom24/curso_openstack_ies/blob/main/modulo3/osc_nova.md).
7.  Accede por ssh (recuerda usar el usuario que has creado con cloud-init) a la instancia.


## **Documento realizado por:**

 ✒️ **Alejandro Montes Delgado** - *2º ASIR*
