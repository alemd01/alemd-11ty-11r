---
layout: post
title: Ejercicios de K8s IV.
excerpt: Cuarto taller de la unidad de Kubernetes.
date: 2023-02-13
updatedDate: 2023-02-13
tags:
  - post
  - SRI
  - Kubernetes
---

### Despliegue y acceso de la aplicación GuestBook.

Una vez que tenemos creado el despliegue de la aplicación GuestBook, que realizamos en el anterioor taller, vamos a crear los Services correspondientes para acceder a ella:

**Service para acceder a la aplicación.**

El primer Service que vamos a crear nos va a permitir acceder a la aplicación GuestBook desde el exterior, para ello crea un fichero yaml con la definición del Service a partir de la siguiente plantilla:

```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: guestbook
    labels:
      app: guestbook
      tier: frontend
  spec:
    type: 
    ports:
    - port: 
      targetPort: 
    selector:
      app: guestbook
      tier: frontend
```
Tienes que poner el tipo del Service, el puerto del servicio será el 80 y el nombre del puerto de la aplicación que hemos asignado en el Deployment es http-server.

Realiza los siguientes pasos:

#### 1. Elabora el fichero yaml con la definición del Service, y créalo.

El fichero queda de la siguiente manera:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: guestbook
  labels:
    app: guestbook
    tier: frontend
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: http-server
  selector:
    app: guestbook
```

Creamos el sevicio:

```txt
alemd@debian:~/minikube$ kubectl apply -f service-guestbook.yaml 
service/guestbook created
```

#### 2. Comprueba el puerto que le han asignado al Service para acceder desde el exterior.

Con el siguiente comando vemos la ip de minikube:

```txt
alemd@debian:~/minikube$ minikube ip
192.168.39.178
```

Ahora vemos información de los servicios para ver el puerto:

```txt
alemd@debian:~/minikube$ kubectl get service
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
guestbook    NodePort    10.97.200.110   <none>        80:31272/TCP   4m1s
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP        13d
```

Vemos que tiene el puerto 31272.

#### 3. Accede a la ip del nodo master y al puerto asignado desde un navegador web para ver la aplicación.

![SRI-K8S-T4.1.png](/img/SRI-K8S-T4.1.png)

#### 4. Responde la siguiente pregunta: ¿Por qué aparece el mensaje de error: Waiting for database connection…?

Porque hemos configurado el front-end, falta configurar la base de datos para que funcione.

**Service para acceder a la base de datos.**

A continuación vamos a crear el Service para acceder a la base de datos. Vamos a crear el fichero yaml para su definición a partir de la siguiente plantilla:

```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: redis
    labels:
      app: redis
      tier: backend
  spec:
    type: 
    ports:
    - port: 
      targetPort: 
    selector:
      app: redis
      tier: backend
```
Tienes que poner el tipo del Service, el puerto del servicio será el 6379 y el nombre del puerto de la base de datos que hemos asignado en el Deployment es redis-server. Nota: No cambies el nombre del Service, ya que la aplicación guestbook va a acceder por defecto a la base de datos usando el nombre redis.

Realiza los siguientes pasos:

#### 1. Elabora el fichero yaml con la definición del Service, y créalo.

El fichero yaml queda de la siguiente manera:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis
  labels:
    app: redis
    tier: backend
spec:
  type: ClusterIP
  ports:
  - port: 6379
    targetPort: redis-server
  selector:
    app: redis
    tier: backend
```
```txt
alemd@debian:~/minikube$ kubectl apply -f service-guestbook2.yaml 
service/redis created
```

#### 2. Lista los Services que has creado.

```txt
alemd@debian:~/minikube$ kubectl get services
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
guestbook    NodePort    10.97.200.110   <none>        80:31272/TCP   57m
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP        13d
redis        ClusterIP   10.98.117.225   <none>        6379/TCP       97s
```

#### 3. Accede a la ip del nodo master y al puerto asignado desde un navegador web para ver la aplicación. Comprueba que funciona sin ningún problema.

![SRI-K8S-T4.2.png](/img/SRI-K8S-T4.2.png)

**Acceso a la aplicación usando Ingress**

Vamos a crear el fichero yaml de definición del recurso Ingress para acceder a la aplicación a partir de la siguiente plantilla:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: guestbook
spec:
  rules:
  - host: 
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: 
            port:
              number: 80
```

Indica un host del tipo www.tunombre.org, indica el nombre del Service que creaste para acceder a la aplicación guestbook y ten en cuenta que el puerto de dicho servicio era el 80.

Realiza los siguientes pasos:

#### 1. Activa el addon ingress en minikube para instalar el Ingress Controller.

Para activar el addon ingress usamos el siguiente comando:

```txt
alemd@debian:~/minikube$ minikube addons enable ingress
💡  ingress is an addon maintained by Kubernetes. For any concerns contact minikube on GitHub.
You can view the list of minikube maintainers at: https://github.com/kubernetes/minikube/blob/master/OWNERS
    ▪ Using image registry.k8s.io/ingress-nginx/controller:v1.5.1
    ▪ Using image registry.k8s.io/ingress-nginx/kube-webhook-certgen:v20220916-gd32f8c343
    ▪ Using image registry.k8s.io/ingress-nginx/kube-webhook-certgen:v20220916-gd32f8c343
🔎  Verifying ingress addon...
🌟  The 'ingress' addon is enabled
```

#### 2. Crea La definición del recurso Ingress con los datos sugeridos, y crea el recurso Ingress.

El fichero yaml queda de la siguiente manera:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: guestbook
spec:
  rules:
  - host: www.alejandro-montes.org
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: guestbook
            port:
              number: 80
```

Y creamos el ingress:

```txt
alemd@debian:~/minikube$ kubectl apply -f service-guestbook3.yaml 
ingress.networking.k8s.io/guestbook created
```

#### 3. Modifica el fichero /etc/hosts de tu ordenador para configurar la resolución estática.

Configuramos el /etc/hosts de nuestro ordenador con la ip de minikube y www.tunombre.org.

#### 4. Accede a la aplicación usando el nombre que has asignado.

![SRI-K8S-T4.3.png](/img/SRI-K8S-T4.3.png)

###  Despliegue y acceso de la Aplicación Lets-Chat.

Let’s Chat es una aplicación web escrita en Node.js que utilizando una base de datos MongoDB nos posibilita la creación de salas de chats.

Vamos a realiza el despliegue y acceso a esta aplicación teniendo en cuenta los siguientes aspectos:

  * La imagen docker que vamos a usar para el despliegue de Let’s Chat es sdelements/lets-chat y para desplegar mongoDB utilizaremos la imagen mongo. Nota: utiliza imagen mongo:4, Let’s Chat es una aplicación antigua y no funciona con las últimas versiones de mongo.
  * Al crear el despliegue de Let’s Chat podemos poner varias replicas, pero el despliegue de la base de datos, sólo creará una replica.
  * El puerto en el que responde la aplicación es el 8080. La base de datos utiliza el puerto 27017.
  * Vamos acceder desde el exterior a la aplicación. Sin embargo, no es necesario acceder desde el exterior a la base de datos.
  * El nombre del Service para acceder a la base de datos debe ser mongo ya que por defecto la aplicación va a conectar a la base de datos usando ese nombre.
  * Queremos acceder a la aplicación usando un nombre del tipo www.chat-tunombre.org.

Realiza los siguientes pasos:

#### 1. Utilizando como modelos los ficheros yaml de la actividad anterior, crea los ficheros necesarios para crear los recursos en tu cluster de Kubernetes para desplegar esta aplicación.

Adjunto los deployments:

  * Deployment de letschat:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: lets-chat
  labels:
    name: letschat
spec:
  replicas: 3 
  selector:
    matchLabels:
      name: lets-chat
  template:
    metadata:
      labels:
        name: lets-chat
    spec:
      containers:
      - name: lets-chat
        image: sdelements/lets-chat
        ports:
          - name: http-server
            containerPort: 8080

```

  * Deployment de mongo:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo
  labels:
    name: mongo
spec:
  replicas: 1
  selector:
    matchLabels:
      name: mongo
  template:
    metadata:
      labels:
        name: mongo
    spec:
      containers:
      - name: mongo
        image: mongo:4
        ports:
          - name: mongo
            containerPort: 27017

```
Adjunto los services:

  * Service de letschat:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: lets-chat
  labels:
    name: lets-chat
    tier: frontend
spec:
  type: NodePort
  ports:
  - port: 8080
    targetPort: http-server 
  selector:
    name: lets-chat

```

  * Service de mongo:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mongo
  labels:
    app: mongo
    tier: backend
spec:
  type: ClusterIP
  ports:
  - port: 27017
    targetPort: mongo 
  selector:
    name: mongo
```

---
## **Documento realizado por:**

 ✒️ **Alejandro Montes Delgado** - *2º ASIR*