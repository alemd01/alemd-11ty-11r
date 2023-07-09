---
layout: post
title: Ejercicios de K8s II.
excerpt: Segundo taller de la unidad de Kubernetes.
date: 2023-02-11
updatedDate: 2023-02-11
tags:
  - post
  - SRI
  - Kubernetes
---

**Como indicamos en el contenido de este módulo, no se va a trabajar directamente con los Pods (realmente tampoco vamos a trabajar directamente con los ReplicaSet, en el siguiente módulo explicaremos los Deployments que serán el recurso con el que trabajaremos). En este ejercicio vamos a crear un ReplicaSet que va a controlar un conjunto de Pods. Para ello, realiza los siguientes pasos:**

### Crea un fichero yaml con la descripción del recurso ReplicaSet, teniendo en cuenta los siguientes aspectos:

  * #### Indica nombres distintos para el ReplicaSet y para el contenedor de los Pods que va a controlar.
  * #### El ReplicaSet va a crear 3 réplicas.
  * #### La imagen que debes desplegar es iesgn/test_web:latest.
  * #### Indica de manera adecuada una etiqueta en la especificación del Pod que vas a definir que coincida con el selector del ReplicaSet.

El fichero es el siguiente:

```txt
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replicaset-t2   
  labels:
    service: web
spec:
  replicas: 3
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
          name: contenedor-t2   
```

### Crea el ReplicaSet.

Usamos el siguiente comando para crear el ReplicaSet:

```txt
alemd@debian:~/minikube$ kubectl apply -f pod-t2.yaml
replicaset.apps/replicaset-t2 created
```

### Comprueba que se ha creado el ReplicaSet y los 3 Pods.

El comando para ver los pods es el siguiente:

```txt
alemd@debian:~/minikube$ kubectl get rs
NAME            DESIRED   CURRENT   READY   AGE
replicaset-t2   3         3         3       4m2s
alemd@debian:~/minikube$ kubectl get pods
NAME                  READY   STATUS    RESTARTS   AGE
replicaset-t2-56w4m   1/1     Running   0          116s
replicaset-t2-9jnsp   1/1     Running   0          116s
replicaset-t2-mbhkx   1/1     Running   0          116s

```

### Obtén información detallada del ReplicaSet creado.

Usamos el siguiente comando para ver información detallada del ReplicaSet:

```txt
alemd@debian:~/minikube$ kubectl describe rs replicaset-t2
Name:         replicaset-t2
Namespace:    default
Selector:     app=nginx
Labels:       service=web
Annotations:  <none>
Replicas:     3 current / 3 desired
Pods Status:  3 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=nginx
  Containers:
   contenedor-t2:
    Image:        iesgn/test_web:latest
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age    From                   Message
  ----    ------            ----   ----                   -------
  Normal  SuccessfulCreate  4m49s  replicaset-controller  Created pod: replicaset-t2-9jnsp
  Normal  SuccessfulCreate  4m49s  replicaset-controller  Created pod: replicaset-t2-56w4m
  Normal  SuccessfulCreate  4m49s  replicaset-controller  Created pod: replicaset-t2-mbhkx
```

### Vamos a probar la tolerancia a fallos: Elimina uno de los 3 Pods, y comprueba que inmediatamente se ha vuelto a crear un nuevo Pod.

Borramos un pod y acto seguido lo mostramos y veremos que se ha creado otro automáticamente:

```txt
alemd@debian:~/minikube$ kubectl delete pod replicaset-t2-56w4m
pod "replicaset-t2-56w4m" deleted
alemd@debian:~/minikube$ kubectl get pods
NAME                  READY   STATUS    RESTARTS   AGE
replicaset-t2-9jnsp   1/1     Running   0          5m38s
replicaset-t2-mbhkx   1/1     Running   0          5m38s
replicaset-t2-zt5dx   1/1     Running   0          10s
```

### Vamos a comprobar la escalabilidad: escala el ReplicaSet para tener 6 Pods de la aplicación.

Cambiamos el número de réplicas y mostramos los pods que tenemos:

```txt
alemd@debian:~/minikube$ kubectl scale rs replicaset-t2 --replicas=6
replicaset.apps/replicaset-t2 scaled
alemd@debian:~/minikube$ kubectl get pods
NAME                  READY   STATUS              RESTARTS   AGE
replicaset-t2-5fgnq   0/1     ContainerCreating   0          13s
replicaset-t2-8fzgx   0/1     ContainerCreating   0          13s
replicaset-t2-9jnsp   1/1     Running             0          7m1s
replicaset-t2-chxc9   0/1     ContainerCreating   0          13s
replicaset-t2-mbhkx   1/1     Running             0          7m1s
replicaset-t2-zt5dx   1/1     Running             0          93s
```

### Elimina el ReplicaSet y comprueba que se han borrado todos los Pods.

Los eliminamos y los mostramos, como vemos tarda un poco ya que al hacer el primer get pods el estado de las réplicas era Terminating.

```txt
alemd@debian:~/minikube$ kubectl delete rs replicaset-t2
replicaset.apps "replicaset-t2" deleted
alemd@debian:~/minikube$ kubectl get pods
NAME                  READY   STATUS        RESTARTS   AGE
replicaset-t2-5fgnq   1/1     Terminating   0          63s
replicaset-t2-8fzgx   1/1     Terminating   0          63s
replicaset-t2-9jnsp   1/1     Terminating   0          7m51s
replicaset-t2-chxc9   1/1     Terminating   0          63s
replicaset-t2-mbhkx   1/1     Terminating   0          7m51s
replicaset-t2-zt5dx   1/1     Terminating   0          2m23s
alemd@debian:~/minikube$ kubectl get pods
No resources found in default namespace.
```

---
## **Documento realizado por:**

 ✒️ **Alejandro Montes Delgado** - *2º ASIR*