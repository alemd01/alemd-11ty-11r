---
layout: post
title: "Ejercicios de modificación de parámetros del kérnel"
date: 2022-11-08
updatedDate: 2022-11-08
tags:
  - post
  - ASO
---


### 1. Deshabilita apparmor en el arranque.

```bash
$ sudo systemctl disable apparmor
```

### 2. Deshabilita si es posible el Kernel Mode Setting (KSM) de la tarjeta gráfica.

Viene desactivado por defecto ya que mi tarjeta gráfica es amd.

### 3. Cambia provisionalmente la swappiness para que la swap de tu equipo se active cuando se use más de un 90% de la RAM.

Con el cat vemos el valor por defecto y con el siguiente comando lo cambiamos:

![img](/img/ASO-P5.1.png)

### 4. Haz que el cambio de la swappiness sea permanente.

Para hacer el cambio permanente, tenemos que editar el fichero `sysctl.conf`:

```bash
$ sudo nano /etc/sysctl.conf
```

añadimos la siguiente línea en el archivo:

```txt
vm.swappiness = 90
```

### 5. Muestra el valor del bit de forward para IPv6.

cat /proc/sys/net/ipv6/conf/all/forwarding
![img](/img/ASO-P5.2.png)

### 6. Deshabilita completamente las Magic Sysrq en el arranque y vuelve a habilitarlas después de reiniciar.

Para deshabilitar las Magic Sysrq usamos el siguiente comando.
```bash
$ sysctl -w kernel.sysrq=0
```

Si lo queremos volver a activar simplemente lo ponemos a 1 de nuevo:

```bash
$ sysctl -w kernel.sysrq=1
```

## **Documento realizado por:**

 ✒️ **Alejandro Montes Delgado** - *2º ASIR*
