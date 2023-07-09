---
layout: post
title: "Práctica: Kubernetes."
excerpt: Realización de la práctica de kubernetes.
date: 2023-02-27
updatedDate: 2023-02-27
tags:
  - post
  - SRI
---

En IAW has creado dos imágenes de dos aplicaciones: bookmedik (php) y polls (python django). Elige una de ellas y despliégala en kubernetes. Para ello vamos a hacer dos ejercicios:

### Ejercicio1: Despliegue en minikube.

**Escribe los ficheros yaml que te posibilitan desplegar la aplicación en minikube. Recuerda que la base de datos debe tener un volumen para hacerla persistente. Debes crear ficheros para los deployments, services, ingress, volúmenes,…**

**Despliega la aplicación en minikube.**

Lo primero que haremos será crear el ConfigMap y un Secret para guardar variables de entorno:

```txt
alemd@debian:~/minikube/bookmedik$ kubectl create cm cm-mariadb --from-literal=mysql_usuario=bookmedik  --from-literal=basededatos=bookmedik
configmap/cm-mariadb created
alemd@debian:~/minikube/bookmedik$ kubectl create secret generic secret-mariadb --from-literal=password=bookmedik --from-literal=rootpass=root
```

Ahora creo el volúmen persistente de la base de datos:

```txt
alemd@debian:~/minikube/bookmedik$ nano pvc-bookmedik.yaml

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
    name: pvc-bookmedik
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi

```

Desplegamos la base de datos:

```txt
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mariadb
  labels:
    app: mariadb
    tier: backend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mariadb
      tier: backend
  template:
    metadata:
      labels:
        app: mariadb
        tier: backend
    spec:
      volumes:
        - name: volumen-mariadb
          persistentVolumeClaim:
            claimName: pvc-bookmedik
      containers:
        - name: contenedor-mariadb
          image: mariadb
          env:
            - name: MARIADB_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: secret-mariadb
                  key: rootpass
            - name: MARIADB_DATABASE
              valueFrom:
                configMapKeyRef:
                  name: cm-mariadb
                  key: basededatos
            - name: MARIADB_USER
              valueFrom:
                configMapKeyRef:
                  name: cm-mariadb
                  key: mysql_usuario
            - name: MARIADB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: secret-mariadb
                  key: password
          ports:
            - name: mariadb-server
              containerPort: 3306
          volumeMounts:
            - mountPath: "/var/lib/mysql"
              name: volumen-mariadb
```

Creamos también el servicio de mariadb:

```txt
apiVersion: v1
kind: Service
metadata:
  name: mariadb
  labels:
    app: mariadb
    tier: backend
spec:
  type: ClusterIP
  ports:
  - port: 3306
    targetPort: mariadb-server
  selector:
    app: mariadb
    tier: backend

```

Lo desplegamos:

```txt
alemd@debian:~/minikube/bookmedik$ kubectl apply -f .
deployment.apps/mariadb created
persistentvolumeclaim/pvc-bookmedik created
service/mariadb created
```

Ahora crearemos el despliegue de bookmedik:

```txt
apiVersion: apps/v1
kind: Deployment
metadata:
  name: bookmedik
  labels:
    app: bookmedik
    tier: frontend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: bookmedik
      tier: frontend
  template:
    metadata:
      labels:
        app: bookmedik
        tier: frontend
    spec:
      containers:
      - name: contenedor-bookmedik
        image: alemd01/bookmedik:v1
        env:
          - name: USUARIO_BOOKMEDIK
            valueFrom:
              configMapKeyRef:
                name: cm-mariadb
                key: mysql_usuario
          - name: CONTRA_BOOKMEDIK
            valueFrom:
              secretKeyRef:
                name: secret-mariadb
                key: password
          - name: DATABASE_HOST
            value: mariadb
          - name: NOMBRE_DB
            valueFrom:
              configMapKeyRef:
                name: cm-mariadb
                key: basededatos
        ports:
          - name: http-server
            containerPort: 80
```

Ahora, por último, crearemos el servicio de bookmedik:

```txt
apiVersion: v1
kind: Service
metadata:
  name: bookmedik
  labels:
    app: bookmedik
    tier: frontend
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: http-server
  selector:
    app: bookmedik
    tier: frontend

```

Lanzamos todo nuevamente:

```txt
alemd@debian:~/minikube/bookmedik$ kubectl apply -f .
deployment.apps/bookmedik created
service/bookmedik created
deployment.apps/mariadb unchanged
service/mariadb unchanged
persistentvolumeclaim/pvc-bookmedik unchanged
```

Comprobamos que se ha creado todo correctamente:

![SRI-K8S-P.2.png](/img/SRI-K8S-P.2.png)

Entramos a través de la IP a bookmedik.

![SRI-K8S-P.1.png](/img/SRI-K8S-P.1.png)

Para facilitarnos la tarea procederé a crear un ingress para poder acceder a través de nombre:

```txt
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: bookmedik-ing
spec:
  rules:
  - host: www.amontes-bookmedik.org
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: bookmedik
            port:
              number: 80

```

Lo levantamos:

```txt
alemd@debian:~/minikube/bookmedik$ kubectl apply -f bookmedik-ingress.yaml 
ingress.networking.k8s.io/bookmedik-ing created
```

Lo añadimos al /etc/hosts y accedemos a través de la dirección:

![SRI-K8S-P.3.png](/img/SRI-K8S-P.3.png)

Comprobamos nuevamente que se loguea correctamente:

![SRI-K8S-P.4.png](/img/SRI-K8S-P.4.png)


### Ejercicio2: Despliegue en otra distribución de kubernetes.

**Instala un clúster de kubernetes (más de un nodo). Tienes distintas opciones para construir un cluster de kubernetes: Alternativas para instalación simple de k8s.**

**Realiza el despliegue de la aplicación en el nuevo clúster. Es posible que no tenga instalado un ingress controller, por lo que no va a funcionar el ingress (puedes buscar como hacer la instalación: por ejemplo el nginx controller).**

**Escala la aplicación y ejecuta kubectl get pods -o wide para ver cómo se ejecutan en los distintos nodos del clúster.**

En mi caso por variar un poco, crearé el cluster con Kind. Tendremos que usar una máquina donde hemos instalado previamente Docker, yo usaré una máquina limpia de proxmox.

```txt
usuario@kind:~$ docker --version
Docker version 20.10.5+dfsg1, build 55c4c88
```

Como podemos comprobar, ya tenemos instalado docker, entonces, lo primero que haremos para instalar kind, es bajar el binario, darle permisos de ejecución y moverlo a un directorio accesible desde $PATH:

```txt
usuario@kind:~$ curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.17.0/kind-linux-amd64
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--  100    98  100    98    0     0    426      0 --:--:-- --:--:-- --:--:--   426
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
  1 6766k    1  128k    0     0   196k      0  0:00:34 --:--:--  0:00:34  100 6766k  100 6766k    0     0  3954k      0  0:00:01  0:00:01 --:--:-- 6100 6766k  100 6766k    0     0  3941k      0  0:00:01  0:00:01 --:--:-- 6239k
usuario@kind:~$ chmod +x ./kind
usuario@kind:~$ sudo mv ./kind /usr/local/bin/kind
```

Verificamos que está instalado correctamente:

```txt
usuario@kind:~$ kind version
kind v0.17.0 go1.19.2 linux/amd64
```

Ahora procederemos a la creación del cluster, para ello crearemos un yaml con los nodos que tendrá el cluster:

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
```

En este fichero indicamos que tenemos un nodo controlador y dos nodos "worker". Ahora crearemos el cluster:

```txt
usuario@kind:~$ kind create cluster --config=config.yaml
Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.25.3) 🖼
 ✓ Preparing nodes 📦 📦 📦  
 ✓ Writing configuration 📜 
 ✓ Starting control-plane 🕹️ 
 ✓ Installing CNI 🔌 
 ✓ Installing StorageClass 💾 
 ✓ Joining worker nodes 🚜 
Set kubectl context to "kind-kind"
You can now use your cluster with:

kubectl cluster-info --context kind-kind

Not sure what to do next? 😅  Check out https://kind.sigs.k8s.io/docs/user/quick-start/
```

Muestro los cluster y los nodos creados:

```txt
usuario@kind:~$ kind get clusters
kind
usuario@kind:~$ kind get nodes
kind-worker
kind-control-plane
kind-worker2
```

Muestro desde docker los contenedores que se han creado:

![SRI-K8S-P.5.png](/img/SRI-K8S-P.5.png)

Una vez tengamos nuestro cluster en marcha, instalaremos kubectl para interactuar con este. Los comandos usados son los siguientes:

```txt
usuario@kind:~$ curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--  100   138  100   138    0     0    831      0 --:--:-- --:--:-- --:--:--  100   138  100   138    0     0    831      0 --:--:-- --:--:-- --:--:--   826
 16 45.8M   16 7758k    0     0  4804k      0  0:00:09  0:00:01  0:00:08 4 22 45.8M   22 10.2M    0     0  4923k      0  0:00:09  0:00:02  0:00:07 5 53 45.8M   53 24.2M    0     0  7966k      0  0:00:05  0:00:03  0:00:02 1100 45.8M  100 45.8M    0     0  11.3M      0  0:00:04  0:00:04 --:--:-- 15.7M
usuario@kind:~$ sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
usuario@kind:~$ chmod +x kubectl
usuario@kind:~$ sudo mv ./kubectl /usr/local/bin/kubectl
usuario@kind:~$ kubectl version --client
WARNING: This version information is deprecated and will be replaced with the output from kubectl version --short.  Use --output=yaml|json to get the full version.
Client Version: version.Info{Major:"1", Minor:"26", GitVersion:"v1.26.2", GitCommit:"fc04e732bb3e7198d2fa44efa5457c7c6f8c0f5b", GitTreeState:"clean", BuildDate:"2023-02-22T13:39:03Z", GoVersion:"go1.19.6", Compiler:"gc", Platform:"linux/amd64"}
Kustomize Version: v4.5.7

```

Ahora desde kubectl mostraremos que el clúster está creado correctamente para verificar que funciona tanto kubectl como el clúster esté bien montado:

```txt
usuario@kind:~$ kubectl get nodes
NAME                 STATUS   ROLES           AGE   VERSION
kind-control-plane   Ready    control-plane   11m   v1.25.3
kind-worker          Ready    <none>          10m   v1.25.3
kind-worker2         Ready    <none>          10m   v1.25.3
```

Una vez hecho esto, procederé a desplegar la aplicación de bookmedik, para ello lo primero que haremos será crear los ConfigMap y el secret:

```txt
usuario@kind:~$ kubectl create cm cm-mariadb --from-literal=mysql_usuario=bookmedik  --from-literal=basededatos=bookmedik
configmap/cm-mariadb created
usuario@kind:~$ kubectl create secret generic secret-mariadb --from-literal=password=bookmedik --from-literal=rootpass=root
secret/secret-mariadb created
```

Ahora nos pasamos a nuestra máquina los ficheros yaml con los que hemos desplegado bookmedik en el ejercicio anterior:

```txt
alemd@debian:~/minikube$ scp -r bookmedik/ usuario@172.22.7.238:bookmedik
mariadb-deployment.yaml                 100% 1346     4.8KB/s   00:00    
bookmedik-deployment.yaml               100%  997    10.1KB/s   00:00    
bookmedik-service.yaml                  100%  228     3.1KB/s   00:00    
pvc-bookmedik.yaml                      100%  164     1.4KB/s   00:00    
mariadb-srv.yaml                        100%  226     2.6KB/s   00:00    
bookmedik-ingress.yaml                  100%  301     4.2KB/s   00:00   
```

Ahora desplegamos todo:

```txt
usuario@kind:~/bookmedik$ kubectl apply -f .
deployment.apps/bookmedik created
ingress.networking.k8s.io/bookmedik-ing created
service/bookmedik created
deployment.apps/mariadb created
service/mariadb created
persistentvolumeclaim/pvc-bookmedik created
```

Comprobamos que se ha creado correctamente todo:

![SRI-K8S-P.6.png](/img/SRI-K8S-P.6.png)

Ahora hacemos un escalado en el despliegue:

```txt
usuario@kind:~/bookmedik$ kubectl scale deployment/bookmedik --replicas=3
deployment.apps/bookmedik scaled
```

Vemos que los pods están repartidos en distintos nodos:

![SRI-K8S-P.7.png](/img/SRI-K8S-P.7.png)

Por último, y no menos importante, probaré que la aplicación funciona correctamente:


![SRI-K8S-P.8.png](/img/SRI-K8S-P.8.png)


El ingress nginx no funciona, estuve probando diferentes configuraciones pero no he conseguido que funcione.

---
## **Documento realizado por:**

 ✒️ **Alejandro Montes Delgado** - *2º ASIR*