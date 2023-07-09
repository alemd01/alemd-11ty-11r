---
layout: post
title: Ejercicios de K8s.
excerpt: Primer taller de la unidad de Kubernetes.
date: 2023-02-10
updatedDate: 2023-02-10
tags:
  - post
  - SRI
  - Kubernetes
---

**Vamos a crear nuestro primer Pod, y para ellos vamos a desplegar una imagen que nos ofrece un servidor web con una página estática. Para ello realiza los siguientes pasos:**

### Crea un fichero yaml con la descripción del recurso Pod, teniendo en cuenta los siguientes aspectos:
  * #### Indica nombres distintos para el Pod y para el contenedor.
  * #### La imagen que debes desplegar es iesgn/test_web:latest.
  * #### Indica una etiqueta en la descripción del Pod.

El contenido del fichero es el siguiente:

```txt
apiVersion: v1
kind: Pod
metadata:
 name: pod-t1
 labels:
   app: nginx
   service: web
spec:
 containers:
   - image: iesgn/test_web:latest
     name: contenedor-t1
     imagePullPolicy: Always

```

### Crea el Pod.

Para crear el pod usamos el siguiente comando:

```txt
alemd@debian:~/minikube$ kubectl create -f pod-t1.yaml
pod/pod-t1 created
```


### Comprueba que el Pod se ha creado y está corriendo.

Para comprobar que el pod está creado usamos el siguiente comando:

```txt
alemd@debian:~/minikube$ kubectl get pods
NAME     READY   STATUS             RESTARTS   AGE
pod-t1   0/1     ImagePullBackOff   0          2m11s
```

### Obtén información detallada del Pod creado.

Con el siguiente comando vemos información detallada del pod:

```txt
alemd@debian:~/minikube$ kubectl describe pod pod-t1
Name:             pod-t1
Namespace:        default
Priority:         0
Service Account:  default
Node:             minikube/192.168.39.178
Start Time:       Fri, 10 Feb 2023 08:47:38 +0100
Labels:           app=nginx
                  service=web
Annotations:      <none>
Status:           Running
IP:               10.244.0.4
IPs:
  IP:  10.244.0.4
Containers:
  contenedor-t1:
    Container ID:   docker://0dca5570cfa08f80d86a65045f964124dc1c994a7be10885e75a5015dee87904
    Image:          iesgn/test_web:latest
```

### Accede de forma interactiva al Pod y comprueba los ficheros que están en el DocumentRoot (usr/local/apache2/htdocs/).

Los comandos ejecutados son los siguientes:

```txt
alemd@debian:~/minikube$ kubectl exec -it pod-t1 -- /bin/bash
root@pod-t1:/usr/local/apache2# ls
bin  build  cgi-bin  conf  error  htdocs  icons  include  logs	modules
root@pod-t1:/usr/local/apache2# cd htdocs/
root@pod-t1:/usr/local/apache2/htdocs# ls
index.html

```

### Crea una redirección con kubectl port-forward utilizando el puerto de localhost 8888 y sabiendo que el Pod ofrece el servicio en el puerto 80. Accede a la aplicación desde un navegador.

Muestro el comando usado para redireccionar el puerto:

```txt
alemd@debian:~/minikube$ kubectl port-forward pod-t1 8888:80
Forwarding from 127.0.0.1:8888 -> 80
Forwarding from [::1]:8888 -> 80
Handling connection for 8888
```

Muestro una captura accediendo por el puerto 8888.

![SRI-T8-T1.1.png](/img/SRI-T8-T1.1.png)

### Muestra los logs del Pod y comprueba que se visualizan los logs de los accesos que hemos realizado en el punto anterior.

El comando para mostrar los logs el siguiente:

```txt
alemd@debian:~/minikube$ kubectl logs pod-t1 
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 10.244.0.4. Set the 'ServerName' directive globally to suppress this message
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 10.244.0.4. Set the 'ServerName' directive globally to suppress this message
[Fri Feb 10 07:50:08.134095 2023] [mpm_event:notice] [pid 1:tid 140490452923520] AH00489: Apache/2.4.46 (Unix) configured -- resuming normal operations
[Fri Feb 10 07:50:08.135109 2023] [core:notice] [pid 1:tid 140490452923520] AH00094: Command line: 'httpd -D FOREGROUND'
127.0.0.1 - - [10/Feb/2023:07:55:18 +0000] "GET / HTTP/1.1" 200 2884
127.0.0.1 - - [10/Feb/2023:07:55:19 +0000] "GET /favicon.ico HTTP/1.1" 404 196
127.0.0.1 - - [10/Feb/2023:07:57:24 +0000] "GET / HTTP/1.1" 200 2884
127.0.0.1 - - [10/Feb/2023:07:57:24 +0000] "GET /favicon.ico HTTP/1.1" 404 196
```

### Elimina el Pod, y comprueba que ha sido eliminado.

La instrucción usada para eliminar el pod es:

```txt
alemd@debian:~/minikube$ kubectl delete pod pod-t1
pod "pod-t1" deleted
```

Muestro los pods para verificar que se ha borrado:

```txt
alemd@debian:~/minikube$ kubectl get pod
No resources found in default namespace.

```

---
## **Documento realizado por:**

 ✒️ **Alejandro Montes Delgado** - *2º ASIR*