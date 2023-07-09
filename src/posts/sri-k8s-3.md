---
layout: post
title: Ejercicios de K8s III.
excerpt: Tercer taller de la unidad de Kubernetes.
date: 2023-02-12
updatedDate: 2023-02-12
tags:
  - post
  - SRI
  - Kubernetes
---

### Trabajando con Deployments.

**En este taller vamos a crear un Deployment de una aplicación web. Sigamos los siguientes pasos:**

#### 1. Crea un fichero yaml con la descripción del recurso Deployment, teniendo en cuenta los siguientes aspectos:
  * **Indica nombres distintos para el Deployment y para el contenedor de los Pods que va a controlar.**
  * **El Deployment va a crear 2 réplicas.**
  * **La imagen que debes desplegar es iesgn/test_web:latest.**
  * **Indica de manera adecuada una etiqueta en la especificación del Pod que vas a definir que coincida con el selector del Deployment.**

El yaml creado es el siguiente:

```txt
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-t3   
  labels:
    app: nginx
spec:
  revisionHistoryLimit: 2
  strategy:
    type: RollingUpdate
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: iesgn/test_web:latest
        name: contendor-t3   
        ports:
        - name: http
          containerPort: 80
```

#### 2. Crea el Deployment.

Para crear el deployment usamos el siguiente comando:

```txt
alemd@debian:~/minikube$ kubectl apply -f pod-t3.yaml
deployment.apps/deployment-t3 created
```

#### 3. Comprueba los recursos que se han creado: Deployment, ReplicaSet y Pods.

Para ver todos los recursos usamos el siguiente comando:

```txt
alemd@debian:~/minikube$ kubectl get all
NAME                                 READY   STATUS    RESTARTS   AGE
pod/deployment-t3-6cb4d64f6f-64mzx   1/1     Running   0          61s
pod/deployment-t3-6cb4d64f6f-m4448   1/1     Running   0          61s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   11d

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/deployment-t3   2/2     2            2           61s

NAME                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/deployment-t3-6cb4d64f6f   2         2         2       61s
```

#### 4. Obtén información detallada del Deployment creado.

Usamos el siguiente comando para ver información detallada del deployment:

```txt
alemd@debian:~/minikube$ kubectl describe deployment deployment-t3
Name:                   deployment-t3
Namespace:              default
CreationTimestamp:      Fri, 10 Feb 2023 18:33:15 +0100
Labels:                 app=nginx
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=nginx
Replicas:               2 desired | 2 updated | 2 total | 2 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx
  Containers:
   contendor-t3:
    Image:        iesgn/test_web:latest
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   deployment-t3-6cb4d64f6f (2/2 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  2m30s  deployment-controller  Scaled up replica set deployment-t3-6cb4d64f6f to 2
```

#### 5. Crea un una redirección utilizando el port-forward para acceder a la aplicación, sabiendo que la aplicación ofrece el servicio en el puerto 80, y accede a la aplicación con un navegador web.

Para realizar el port-forward con un deployment usamos el siguiente comando:

```txt
alemd@debian:~/minikube$ kubectl port-forward deployment/deployment-t3 8088:80
Forwarding from 127.0.0.1:8088 -> 80
Forwarding from [::1]:8088 -> 80
```

Accedemos a la página web por el puerto 8088:

![SRI-K8S-T3.1.png](/img/SRI-K8S-T3.1.png)

#### 6. Accede a los logs del despliegue para comprobar el acceso que has hecho en el punto anterior.

A continuación, muestro los logs del deployment.

```txt
alemd@debian:~/minikube$ kubectl logs deployments/deployment-t3
Found 2 pods, using pod/deployment-t3-6cb4d64f6f-m4448
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 10.244.0.14. Set the 'ServerName' directive globally to suppress this message
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 10.244.0.14. Set the 'ServerName' directive globally to suppress this message
[Fri Feb 10 17:33:26.060609 2023] [mpm_event:notice] [pid 1:tid 140502009566336] AH00489: Apache/2.4.46 (Unix) configured -- resuming normal operations
[Fri Feb 10 17:33:26.196721 2023] [core:notice] [pid 1:tid 140502009566336] AH00094: Command line: 'httpd -D FOREGROUND'
127.0.0.1 - - [10/Feb/2023:17:38:47 +0000] "GET / HTTP/1.1" 200 2884
127.0.0.1 - - [10/Feb/2023:17:38:47 +0000] "GET /favicon.ico HTTP/1.1" 404 196

```

#### 7. Elimina el Deployment y comprueba que se han borrado todos los recursos creados.

Usamos el deplete deplyment:

```txt
alemd@debian:~/minikube$ kubectl delete deployments.apps deployment-t3
deployment.apps "deployment-t3" deleted
```

### Actualización y desactualización de nuestra aplicación.

**El equipo de desarrollo ha creado una primera versión preliminar de una aplicación web y ha creado una imagen de contenedor con el siguiente nombre: iesgn/test_web:version1.**

**Vamos a desplegar esta primera versión de la aplicación, para ello:**

#### 1. Crea un fichero yaml (puedes usar el de la actividad anterior) para desplegar la imagen: iesgn/test_web:version1.

He usado el mismo fichero que en el anterior ejercicio, no lo vuelvo a poner para no duplicar información, el cambio en el fichero es en container la imagen es `iesgn/test_web:version1`.

#### 2. Crea el Deployment, recuerda realizar una anotación para indicar las características del despliegue.

Creamos el deployment y lo anotamos con los siguientes comandos:

```txt
alemd@debian:~/minikube$ kubectl apply -f deployment-t3.yaml
deployment.apps/deployment-t3 created
alemd@debian:~/minikube$ kubectl annotate deployment/deployment-t3 iesgn/test_web="despliegue version 1"
deployment.apps/deployment-t3 annotated
```

#### 3. Crea una redirección utilizando el port-forward para acceder a la aplicación, sabiendo que la aplicación ofrece el servicio en el puerto 80, y accede a la aplicación con un navegador web.

Adjunto el comando para el port-forward:

```txt
alemd@debian:~/minikube$ kubectl port-forward deployment/deployment-t3 8088:80
Forwarding from 127.0.0.1:8088 -> 80
Forwarding from [::1]:8088 -> 80
Handling connection for 8088
```

![SRI-K8S-T3.2.png](/img/SRI-K8S-T3.2.png)


**Nuestro equipo de desarrollo ha seguido trabajando y ya tiene lista la versión 2 de nuestra aplicación, han creado una imagen que se llama: iesgn/test_web:version2. Vamos a actualizar nuestro despliegue con la nueva versión, para ello:**

#### 4. Realiza la actualización del despliegue utilizando la nueva imagen (no olvides anotar la causa).

Lo primero que haremos será editar nuestro fichero yaml, cambiando `iesgn/test_web:version1` por `iesgn/test_web:version2`. Volvemos a hacer un apply:

```txt
alemd@debian:~/minikube$ kubectl apply -f deployment-t3.yaml
deployment.apps/deployment-t3 configured
```

Lo anotamos:

```txt
alemd@debian:~/minikube$ kubectl annotate deployment/deployment-t3 iesgn/test_web2="despliegue version 2"
deployment.apps/deployment-t3 annotated
```

#### 5. Comprueba los recursos que se han creado: Deployment, ReplicaSet y Pods.

Usamos el siguiente comando:

```txt
alemd@debian:~/minikube$ kubectl get all
NAME                                 READY   STATUS    RESTARTS   AGE
pod/deployment-t3-548d4685f4-6cfxc   1/1     Running   0          4m50s
pod/deployment-t3-548d4685f4-z4tqf   1/1     Running   0          4m39s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   11d

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/deployment-t3   2/2     2            2           12m

NAME                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/deployment-t3-548d4685f4   2         2         2       4m50s
replicaset.apps/deployment-t3-55f8ffb8f5   0         0         0       12m

```

#### 6. Visualiza el historial de actualizaciones.

Para ver el historial de actualizaciones usamos el siguiente comando:

```txt
alemd@debian:~/minikube$ kubectl rollout history deployment/deployment-t3
deployment.apps/deployment-t3 
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

#### 7. Crea una redirección utilizando el port-forward para acceder a la aplicación, sabiendo que la aplicación ofrece el servicio en el puerto 80, y accede a la aplicación con un navegador web.

Creamos la redirección:

```txt
alemd@debian:~/minikube$ kubectl port-forward deployment/deployment-t3 8088:80
Forwarding from 127.0.0.1:8088 -> 80
Forwarding from [::1]:8088 -> 80
```

Accedemos a la página web.

![SRI-K8S-T3.3.png](/img/SRI-K8S-T3.3.png)


**Finalmente después de un trabajo muy duro, el equipo de desarrollo ha creado la imagen iesgn/test_web:version3 con la última versión de nuestra aplicación y la vamos a poner en producción, para ello:**

#### 8. Realiza la actualización del despliegue utilizando la nueva imagen (no olvides anotar la causa).

Actualizamos a la version 3, adjunto todos los comandos:

```txt
alemd@debian:~/minikube$ kubectl set image deployment/deployment-t3 contendor-t3=iesgn/test_web:version3
deployment.apps/deployment-t3 image updated
alemd@debian:~/minikube$ kubectl annotate deployment/deployment-t3 iesgn/change_cause="despliegue version 3"
deployment.apps/deployment-t3 annotated
```

#### 9. Comprueba los recursos que se han creado: Deployment, ReplicaSet y Pods.

Vemos todos los recursos:

```txt
alemd@debian:~/minikube$ kubectl get all
NAME                                 READY   STATUS    RESTARTS   AGE
pod/deployment-t3-56fc8df8d9-lss46   1/1     Running   0          75s
pod/deployment-t3-56fc8df8d9-ts7tv   1/1     Running   0          86s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   11d

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/deployment-t3   2/2     2            2           26m

NAME                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/deployment-t3-548d4685f4   0         0         0       18m
replicaset.apps/deployment-t3-55f8ffb8f5   0         0         0       26m
replicaset.apps/deployment-t3-56fc8df8d9   2         2         2       86s
```

#### 10. Visualiza el historial de actualizaciones.

Muestro el historial de versiones:

```txt
alemd@debian:~/minikube$ kubectl rollout history deployment/deployment-t3
deployment.apps/deployment-t3 
REVISION  CHANGE-CAUSE
2         <none>
3         <none>
4         <none>
```

#### 11. Crea una redirección utilizando el port-forward para acceder a la aplicación, sabiendo que la aplicación ofrece el servicio en el puerto 80, y accede a la aplicación con un navegador web.

```txt
alemd@debian:~/minikube$ kubectl port-forward deployment/deployment-t3 8088:80
Forwarding from 127.0.0.1:8088 -> 80
Forwarding from [::1]:8088 -> 80
```

Muestro la página web:

![SRI-K8S-T3.4.png](/img/SRI-K8S-T3.4.png)

**¡Vaya!, parece que esta versión tiene un fallo, y no se ve de forma adecuada la hoja de estilos, tenemos que volver a la versión anterior:**

#### 12. Ejecuta la instrucción que nos permite hacer un rollback de nuestro despliegue.

Hacemos el rollback:

```txt
alemd@debian:~/minikube$ kubectl set image deployment deployment-t3 contendor-t3=iesgn/test_web:version2
deployment.apps/deployment-t3 image updated
```

#### 13. Comprueba los recursos que se han creado: Deployment, ReplicaSet y Pods.

```txt
alemd@debian:~/minikube$ kubectl get all
NAME                                 READY   STATUS    RESTARTS   AGE
pod/deployment-t3-548d4685f4-4wbkh   1/1     Running   0          3m4s
pod/deployment-t3-548d4685f4-r9492   1/1     Running   0          3m10s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   11d

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/deployment-t3   2/2     2            2           33m

NAME                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/deployment-t3-548d4685f4   2         2         2       25m
replicaset.apps/deployment-t3-55f8ffb8f5   0         0         0       33m
replicaset.apps/deployment-t3-56fc8df8d9   0         0         0       8m35s
```

#### 14. Visualiza el historial de actualizaciones.

```txt
alemd@debian:~/minikube$ kubectl rollout history deployment/deployment-t3
deployment.apps/deployment-t3 
REVISION  CHANGE-CAUSE
3         <none>
4         <none>
5         <none>
```

#### 15. Crea una redirección utilizando el port-forward para acceder a la aplicación, sabiendo que la aplicación ofrece el servicio en el puerto 80, y accede a la aplicación con un navegador web.

```txt
alemd@debian:~/minikube$ kubectl port-forward deployment/deployment-t3 8088:80
Forwarding from 127.0.0.1:8088 -> 80
Forwarding from [::1]:8088 -> 80
Handling connection for 8088
```

![SRI-K8S-T3.5.png](/img/SRI-K8S-T3.5.png)

### Despliegue de la aplicación GuestBook.

**En esta tarea vamos a desplegar una aplicación web que requiere de dos servicios para su ejecución. La aplicación se llama GuestBook y necesita los siguientes servicios:**

* **La aplicación Guestbook es una aplicación web desarrollada en python que es servida en el puerto 5000/tcp. Utilizaremos la imagen iesgn/guestbook.**
* **Esta aplicación guarda la información en una base de datos no relacional redis, que utiliza el puerto 6379/tcp para recibir las conexiones. Usaremos la imagen redis.**

**Por lo tanto si tenemos dos servicios distintos, tendremos dos ficheros yaml para crear dos recursos Deployment, uno para cada servicio. Con esta manera de trabajar podemos obtener las siguientes características:**

 **1. Cada conjunto de Pods creado en cada despliegue ejecutarán un solo proceso para ofrecer el servicio.**
 **2. Cada conjunto de Pods se puede escalar de manera independiente. Esto es importante, si identificamos que al acceder a alguno de los servicios se crea un cuello de botella, podemos escalarlo para tener más Pods ejecutando el servicio.**
 **3. Las actualizaciones de los distintos servicios no interfieren en el resto.**
 **4. Lo estudiaremos en un módulo posterior, pero podremos gestionar el almacenamiento de cada servicio de forma independiente.**

**Por lo tanto para desplegar la aplicaciones tendremos dos ficheros.yaml:**

* **guestbook-deployment.yaml**
* **redis-deployment.yaml**

**Para realizar el despliegue realiza los siguientes pasos:**

#### 1. Usando los ficheros anteriores crea los dos Deployments.

Ejecutamos los siguientes comandos:

```txt
alemd@debian:~/minikube$ kubectl apply -f guestbook-deployment.yaml 
deployment.apps/guestbook created
alemd@debian:~/minikube$ kubectl apply -f redis-deployment.yaml 
deployment.apps/redis created
```

#### 2. Comprueba que los recursos que se han creado: Deployment, ReplicaSet y Pods.

Muestro todos los recursos:

```txt
alemd@debian:~/minikube$ kubectl get all
NAME                             READY   STATUS    RESTARTS   AGE
pod/guestbook-8555449655-46576   1/1     Running   0          2m23s
pod/guestbook-8555449655-7sr82   1/1     Running   0          2m23s
pod/guestbook-8555449655-hrvld   1/1     Running   0          2m23s
pod/redis-6db6c7bfb5-5rn57       1/1     Running   0          2m17s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   11d

NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/guestbook   3/3     3            3           2m24s
deployment.apps/redis       1/1     1            1           2m17s

NAME                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/guestbook-8555449655   3         3         3       2m23s
replicaset.apps/redis-6db6c7bfb5       1         1         1       2m17s
```

#### 3. Crea una redirección utilizando el port-forward para acceder a la aplicación, sabiendo que la aplicación ofrece el servicio en el puerto 5000, y accede a la aplicación con un navegador web.

```txt
alemd@debian:~/minikube$ kubectl port-forward deployment/guestbook 8088:5000
Forwarding from 127.0.0.1:8088 -> 5000
Forwarding from [::1]:8088 -> 5000

```

Muestro la web:

![SRI-K8S-T3.6.png](/img/SRI-K8S-T3.6.png)


**¿Qué aparece en la página principal de la aplicación?. Aparece el siguiente mensaje: Waiting for database connection…. Por lo tanto podemos indicar varias conclusiones:**

**1. Hasta ahora no estamos accediendo de forma “normal” a las aplicaciones. El uso de la opción port-forward es un mecanismo que realmente nos posibilita acceder a la aplicación, pero utilizando un proxy. Deberíamos acceder a las aplicaciones usando una ip y un puerto determinado.**
**2. Parece que tampoco hay acceso entre los Pods de los distintos despliegues. Parece que los Pods de la aplicación guestbook no pueden acceder al Pod donde se está ejecutando la base de datos redis.**

**En el siguiente módulo estudiaremos los recursos que nos ofrece la API de Kubernetes para permitirnos el acceso a las aplicaciones desde el exterior, y para que los distintos Pods de los despliegues puedan acceder entre ellos.**

---
## **Documento realizado por:**

 ✒️ **Alejandro Montes Delgado** - *2º ASIR*