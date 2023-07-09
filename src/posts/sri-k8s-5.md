---
layout: post
title: Ejercicios de K8s V.
excerpt: Quinto taller de la unidad de Kubernetes.
date: 2023-02-16
updatedDate: 2023-02-16
tags:
  - post
  - SRI
  - Kubernetes
---

### Configurando nuestra aplicación Temperaturas.

**En un ejemplo anterior: Ejemplo completo: Desplegando y accediendo a la aplicación Temperaturas habíamos desplegado una aplicación formada por dos microservicios que nos permitía visualizar las temperaturas de municipios.**

**Recordamos que el componente frontend hace peticiones al componente backend utilizando el nombre temperaturas-backend, que es el nombre que asignamos al Service ClusterIP para el acceso al backend.**

**Vamos a cambiar la configuración de la aplicación para indicar otro nombre.**

**Podemos configurar el nombre del servidor backend al que vamos acceder desde el frontend modificando la variable de entorno TEMP_SERVER a la hora de crear el despliegue del frontend.**

**Por defecto el valor de esa variable es:**

`TEMP_SERVER temperaturas-backend:5000`

**Vamos a modificar esta variable en el despliegue del frontend y cambiaremos el nombre del Service del backend para que coincidan, para ello realiza los siguientes pasos:**

#### 1. Crea un recurso ConfigMap con un dato que tenga como clave SERVIDOR_TEMPERATURAS y como contenido servidor-temperaturas:5000.

Creo el recurso ConfigMap:

```txt
alemd@debian:~/minikube$ kubectl create cm temperaturas --from-literal=SERVIDOR_TEMPERATURAS=servidor-temperaturas:5000
configmap/temperaturas created
```

Podemos ver el recurso utilizando el siguiente comando:

```txt
alemd@debian:~/minikube$ kubectl describe cm temperaturas
Name:         temperaturas
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
SERVIDOR_TEMPERATURAS:
----
servidor-temperaturas:5000

BinaryData
====

Events:  <none>
```

#### 2. Modifica el fichero de despliegue del frontend: frontend-deployment.yaml para añadir la modificación de la variable TEMP_SERVER con el valor que hemos guardado en el ConfigMap.

El fichero queda de la siguiente manera:

```txt
apiVersion: apps/v1
kind: Deployment
metadata:
  name: temperaturas-frontend
  labels:
    app: temperaturas
    tier: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: temperaturas
      tier: frontend
  template:
    metadata:
      labels:
        app: temperaturas
        tier: frontend
    spec:
      containers:
      - name: contenedor-temperaturas
        image: iesgn/temperaturas_frontend
        ports:
          - name: http-server
            containerPort: 3000
        env:
          - name: TEMP_SERVER
            valueFrom:
              configMapKeyRef:
                name: temperaturas
                key: SERVIDOR_TEMPERATURAS
```

#### 3. Realiza el despliegue y crea el Service para acceder al frontend.

fichero del frontend service:

```txt
apiVersion: v1
kind: Service
metadata:
  name: temperaturas-frontend
  labels:
    app: temperaturas
    tier: frontend
spec:
  type: NodePort
  ports:
  - port: 3000
    targetPort: http-server
  selector:
    app: temperaturas
    tier: frontend
```

#### 4. Despliega el microservicio backend.

Muestro el fichero del despliegue del backend:

```txt
apiVersion: apps/v1
kind: Deployment
metadata:
  name: temperaturas-backend
  labels:
    app: temperaturas
    tier: backend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: temperaturas
      tier: backend
  template:
    metadata:
      labels:
        app: temperaturas
        tier: backend
    spec:
      containers:
        - name: contendor-servidor-temperaturas
          image: iesgn/temperaturas_backend
          ports:
            - name: api-server
              containerPort: 5000
```

#### 5. Modifica el fichero backend-srv.yaml para cambiar el nombre del Service por servidor-temperaturas y crea el Service.

Muestro el fichero:

```txt
apiVersion: v1
kind: Service
metadata:
  name: servidor-temperaturas
  labels:
    app: temperaturas
    tier: backend
spec:
  type: ClusterIP
  ports:
  - port: 5000
    targetPort: api-server
  selector:
    app: temperaturas
    tier: backend
```

#### 6. Accede a la aplicación usando el puerto asignado al Service NodePort del frontend o creando el recurso Ingress.

![SRI-K8S-V.1.png](/img/SRI-K8S-V.1.png)

### Despliegue y acceso de la aplicación Nextcloud.

**Basándonos en el Ejemplo completo: Despliegue y acceso a Wordpress + MariaDB vamos a realizar el despliegue de la aplicación NextCloud + MariaDB. Para ello ten en cuenta lo siguiente:**

  * **El despliegue de la base de datos se haría de la misma forma que encontramos en el ejemplo de Wordpress, pero para esta actividad vamos a usar la imagen mariadb:10.5.**

  * **Según la documentación de NextCloud en DockerHub las variables de entorno que tienes que modificar serían: MYSQL_DATABASE, MYSQL_USER, MYSQL_PASSWORD y MYSQL_HOST.**

  * **Al igual que en el ejemplo utiliza un recurso ConfigMap para guardar los valores de configuración no sensibles, y un recurso Secret para los datos sensibles.**

  * **4. Utiliza los ficheros yaml del ejemplo haciendo las modificaciones oportunas.**

Lo primero que haremos será crear los recursos ConfigMap y Secret.

```txt
alemd@debian:~/minikube/nextcloud$ kubectl create cm cm-nextcloud --from-literal=MYSQL_DATABASE=mariadb --from-literal=MYSQL_USER=mariadb
configmap/cm-nextcloud created
alemd@debian:~/minikube/nextcloud$ kubectl create secret generic secret-nextcloud --from-literal=MYSQL_PASSWORD=mariadb
secret/secret-nextcloud created
alemd@debian:~/minikube/nextcloud$ kubectl create cm cm-host --from-literal=MYSQL_HOST=mariadb-service
configmap/cm-host created
```

Ahora, adjunto el deployment de mariadb:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mariadb-deployment
  labels:
    app: nextcloud
    type: database
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nextcloud
      type: database
  template:
    metadata:
      labels:
        app: nextcloud
        type: database
    spec:
      containers:
        - name: contenedor-mariadb
          image: mariadb
          ports:
            - containerPort: 3306
              name: db-port
          env:
            - name: MYSQL_USER
              valueFrom:
                configMapKeyRef:
                  name: cm-nextcloud
                  key: MYSQL_USER
            - name: MYSQL_DATABASE
              valueFrom:
                configMapKeyRef:
                  name: cm-nextcloud
                  key: MYSQL_DATABASE
            - name: MYSQL_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: secret-nextcloud
                  key: MYSQL_PASSWORD
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: secret-nextcloud
                  key: MYSQL_PASSWORD
```

El service de mariadb:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mariadb-service
  labels:
    app: nextcloud
    type: database
spec:
  selector:
    app: nextcloud
    type: database
  ports:
  - port: 3306
    targetPort: db-port
  type: ClusterIP
```

El deployment de nextcloud:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nextcloud-deployment
  labels:
    app: nextcloud
    type: frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nextcloud
      type: frontend
  template:
    metadata:
      labels:
        app: nextcloud
        type: frontend
    spec:
      containers:
        - name: contenedor-nextcloud
          image: nextcloud
          ports:
            - containerPort: 80
              name: http-port
            - containerPort: 443
              name: https-port
          env:
            - name: MYSQL_DATABASE
              valueFrom:
                configMapKeyRef:
                  name: cm-nextcloud
                  key: MYSQL_DATABASE
            - name: MYSQL_USER
              valueFrom:
                configMapKeyRef:
                  name: cm-nextcloud
                  key: MYSQL_USER
            - name: MYSQL_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: secret-nextcloud
                  key: MYSQL_PASSWORD
            - name: MYSQL_HOST
              valueFrom:
                configMapKeyRef:
                  name: cm-host
                  key: MYSQL_HOST
```

El service de nextcloud:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nextcloud-service
  labels:
    app: nextcloud
    type: frontend
spec:
  selector:
    app: nextcloud
    type: frontend
  ports:
  - name: http-sv-port
    port: 80
    targetPort: http-port
  - name: https-sv-port
    port: 443
    targetPort: https-port
  type: NodePort
```

Ahora levantamos los deployments y los services:

```txt
alemd@debian:~/minikube/nextcloud$ kubectl apply -f mariadb-deployment.yaml 
deployment.apps/mariadb-deployment created
alemd@debian:~/minikube/nextcloud$ kubectl apply -f mariadb-service.yaml 
service/mariadb-service created
alemd@debian:~/minikube/nextcloud$ kubectl apply -f nextcloud-deployment.yaml 
deployment.apps/nextcloud-deployment created
alemd@debian:~/minikube/nextcloud$ kubectl apply -f nextcloud-service.yaml
service/nextcloud-service created
```

Ya podemos acceder a nuestro nextcloud:

![SRI-K8S-5.1.png](/img/SRI-K8S-5.1.png)

---
## **Documento realizado por:**

 ✒️ **Alejandro Montes Delgado** - *2º ASIR*