---
layout: post
title: Ejercicios de K8s VI.
excerpt: Sexto taller de la unidad de Kubernetes.
date: 2023-02-20
updatedDate: 2023-02-20
tags:
  - post
  - SRI
  - Kubernetes
---

### Desplegando un servidor web persistente.

**Siguiendo la guía explicada en el Ejemplo 2: Gestión dinámica de volúmenes, vamos a crear un servidor web que permita la ejecución de scripts PHP con almacenamiento persistente.**

**Para realizar esta actividad vamos a usar asignación dinámica de volúmenes y puedes usar, como modelos, los ficheros del ejemplo 2.**

**Realiza los siguientes pasos:**

#### Crea un fichero yaml para definir un recurso PersistentVolumenClaim que se llame pvc-webserver y para solicitar un volumen de 2Gb.

El contenido del fichero es el siguiente:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
    name: pvc-webserver
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
```

#### Crea el recurso y comprueba que se ha asociado un volumen de forma dinámica a la solicitud.

Creamos el recurso con el siguiente comando:

```txt
alemd@debian:~/minikube/taller6$ kubectl apply -f pvc-webserver.yaml
persistentvolumeclaim/pvc-webserver created
```

Y lo visualizamos con el siguiente comando:

```txt
alemd@debian:~/minikube/taller6$ kubectl get pv,pvc
NAME                                                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                   STORAGECLASS   REASON   AGE
persistentvolume/pvc-f30ccc20-4f70-48e4-923c-b9d541348327   2Gi        RWO            Delete           Bound    default/pvc-webserver   standard                27s

NAME                                  STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/pvc-webserver   Bound    pvc-f30ccc20-4f70-48e4-923c-b9d541348327   2Gi        RWO            standard       28s
```


#### Crea un fichero yaml para desplegar un servidor web desde la imagen php:7.4-apache, asocia el volumen al Pod que se va a crear e indica el punto de montaje en el DocumentRoot del servidor: /var/www/html.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: servidorweb
  labels:
    app: apache
spec:
  replicas: 1
  selector:
    matchLabels:
      app: apache
  template:
    metadata:
      labels:
        app: apache
    spec:
      volumes:
        - name: volumen-servidorweb
          persistentVolumeClaim:
            claimName: pvc-webserver
      containers:
        - name: contenedor-apache-php
          image: php:7.4-apache
          ports:
            - name: http-server
              containerPort: 80
          volumeMounts:
            - mountPath: "/var/www/html"
              name: volumen-servidorweb
```

#### Despliega el servidor y crea un fichero info.php en /var/www/html, con el siguiente contenido: <?php phpinfo(); ?>.

Desplegamos el servidor:

```txt
alemd@debian:~/minikube/taller6$ kubectl apply -f servidorweb-php.yaml 
deployment.apps/servidorweb created
```

Ahora creamos un service:

```txt
alemd@debian:~/minikube/taller6$ nano web-service.yaml
```

Con el siguiente contenido:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: servicio-servidorweb
spec:
  type: NodePort
  ports:
  - name: service-http
    port: 80
    targetPort: http-server
  selector:
    app: apache
```

Creamos el service:

```txt
alemd@debian:~/minikube/taller6$ kubectl apply -f web-service.yaml
service/servicio-servidorweb created
```

Ahora ejecutamos el siguiente comando para añadir el info.php al servidor web:

```txt
alemd@debian:~/minikube/taller6$ kubectl exec pod/servidorweb-6d46fd9685-vhqjq -- bash -c "echo '<?php phpinfo(); ?>' > /var/www/html/info.php"
```

#### Define y crea un Service NodePort, accede desde un navegador al fichero info.php y comprueba que se visualiza de forma correcta.

El Service NodePort está creado más arriba.

Probamos que funciona accediendo a través de la IP y el puerto:

![SRI-K8S-VI.1.png](/img/SRI-K8S-VI.1.png)

#### Comprobemos la persistencia: elimina el Deployment, vuelve a crearlo y vuelve a acceder desde el navegador al fichero info.php. ¿Se sigue visualizando?

Eliminamos el deployment:

```txt
alemd@debian:~/minikube/taller6$ kubectl delete deployments.apps servidorweb
deployment.apps "servidorweb" deleted
```

Volvemos a crearlo:

```txt
alemd@debian:~/minikube/taller6$ kubectl apply -f servidorweb-php.yaml
deployment.apps/servidorweb created
```

Volvemos a acceder a la página:

![SRI-K8S-VI.2.png](/img/SRI-K8S-VI.2.png)

Como vemos funciona correctamente.

### Haciendo persistente la aplicación GuestBook.

**En este ejercicio vamos a volver a desplegar nuestra aplicación GuestBook, que realizamos en el taller 4 y 5, para añadirle persistencia a la base de datos redis.**

**Por lo tanto necesitaremos solicitar un volumen, que se asociará de forma dinámica.**

**Creando el despliegue de redis para que guarde la información en un directorio.**

**Si estudiamos la documentación de la imagen redis en Docker Hub, para que la información de la base de datos se guarde en un directorio /data del contenedor hay que ejecutar con docker:**

`docker run --name some-redis -d redis redis-server --appendonly yes`

**Es decir, hay que crear el contenedor ejecutando el proceso redis-server con los argumentos --appendonly yes.**

**Por lo tanto tenemos que cambiar el fichero de definición del Deployment de redis, redis-deployment.yaml de la siguiente manera:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
  labels:
    app: redis
    tier: backend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
      tier: backend
  template:
    metadata:
      labels:
        app: redis
        tier: backend
    spec:
      volumes:
        - name: volumen-redis
          persistentVolumeClaim:
            claimName: xxxxxxxxxxxx
      containers:
        - name: contenedor-redis
          image: redis
          command: ["redis-server"]
          args: ["--appendonly", "yes"]
          ports:
            - name: redis-server
              containerPort: 6379
          volumeMounts:
            - mountPath: xxxxxxxxxxxx
              name: volumen-redis
```

**Hemos usado el parámetro command para ejecutar el proceso, y el parámetro args para indicar los argumentos.**

**Nota: Los valores xxxxxxxxxxxx tendrás que rellenarlos durante la realización de la actividad.**

**Pasos a seguir**

**Realiza los siguientes pasos:**

#### 1. Crea un fichero yaml para definir un recurso PersistentVolumenClaim que se llame pvc-redis y para solicitar un volumen de 3Gb.

El contenido del fichero es el siguiente:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
    name: pvc-redis
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
```

#### 2. Crea el recurso y comprueba que se ha asociado un volumen de forma dinámica a la solicitud.

Creamos el recurso:

```txt
alemd@debian:~/minikube/taller6$ kubectl apply -f pvc-redis.yaml 
persistentvolumeclaim/pvc-redis created
```

Ahora comprobamos que se ha asociado el volumen:

```txt
alemd@debian:~/minikube/taller6$ kubectl get pv,pvc
NAME                                                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                   STORAGECLASS   REASON   AGE
persistentvolume/pvc-25b35b29-616c-488d-9f94-0e9e95321da3   3Gi        RWO            Delete           Bound    default/pvc-redis       standard                47s
persistentvolume/pvc-f30ccc20-4f70-48e4-923c-b9d541348327   2Gi        RWO            Delete           Bound    default/pvc-webserver   standard                47m

NAME                                  STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/pvc-redis       Bound    pvc-25b35b29-616c-488d-9f94-0e9e95321da3   3Gi        RWO            standard       47s
persistentvolumeclaim/pvc-webserver   Bound    pvc-f30ccc20-4f70-48e4-923c-b9d541348327   2Gi        RWO            standard       47m
```

#### 3. Modifica el fichero del despliegue de redis, modificando las xxxxxxxxxxxx por los valores correctos: el nombre del PersistentVolumenClaim y el directorio de montaje en el contenedor (como hemos visto anteriormente es /data).

Queda de la siguiente forma el yaml:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
  labels:
    app: redis
    tier: backend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
      tier: backend
  template:
    metadata:
      labels:
        app: redis
        tier: backend
    spec:
      volumes:
        - name: volumen-redis
          persistentVolumeClaim:
            claimName: pvc-redis
      containers:
        - name: contenedor-redis
          image: redis
          command: ["redis-server"]
          args: ["--appendonly", "yes"]
          ports:
            - name: redis-server
              containerPort: 6379
          volumeMounts:
            - mountPath: "/data"
              name: volumen-redis
```

#### 4. Crea el despliegue de redis. El despliegue de la aplicación guestbook y la creación de los Services de acceso se hace con los ficheros que ya utilizamos anteriormente: guestbook-deployment.yaml, guestbook-srv.yaml y redis-srv.yaml.

Desplegamos guestbook con los fichero del taller anterior:

```txt
alemd@debian:~/minikube$ kubectl apply -f guestbook-deployment.yaml
deployment.apps/guestbook created
```

Desplegamos el servicio de guestbook:

```txt
alemd@debian:~/minikube$ kubectl apply -f service-guestbook.yaml 
service/guestbook created
```

Desplegamos el servicio de redis:

```txt
alemd@debian:~/minikube$ kubectl apply -f service-guestbook2.yaml 
service/redis created
```

Desplegamos el nuevo redis:

```txt
alemd@debian:~/minikube$ cd taller6/
alemd@debian:~/minikube/taller6$ kubectl apply -f redis-despliegue.yaml 
deployment.apps/redis created
```

#### 5. Accede a la aplicación y escribe algunos mensajes.

![SRI-K8S-VI.3.png](/img/SRI-K8S-VI.3.png)

#### 6. Comprobemos la persistencia: elimina el despliegue de redis, vuelve a crearlo, vuelve a acceder desde el navegador y comprueba que los mensajes no se han perdido.

Eliminamos y volvemos a crear el despliegue:

```txt
alemd@debian:~/minikube/taller6$ kubectl delete deployments.apps redis 
deployment.apps "redis" deleted
alemd@debian:~/minikube/taller6$ kubectl apply -f redis-despliegue.yaml 
deployment.apps/redis created
```

Accedo nuevamente al navegador:

![SRI-K8S-VI.4.png](/img/SRI-K8S-VI.4.png)


### Haciendo persistente la aplicación Nextcloud.

**Esta actividad es la continuación de la actividad realizada en el taller 5.**

**Siguiendo la guía que hemos desarrollado en Ejemplo 3: Wordpress con almacenamiento persistente vamos a configurar el despliegue de Nextcloud para que use volúmenes (vamos a usar dos volúmenes, uno para la aplicación y otro para la base de datos) para que la información no se pierda.**

**Realiza los siguientes pasos:**

#### 1. Crea los ficheros yaml para definir los recursos PersistentVolumenClaim para solicitar dos volúmenes de 4Gb.

El contenido del pvc para nextcloud es el siguiente:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
    name: nextcloud-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 4Gi
```

El contenido del pvc para mariadb es el siguiente:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
    name: mariadb-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 4Gi
```

#### 2. Crea esos recursos y comprueba que se ha asociado un volumen de forma dinámica a cada solicitud.

Creamos los recursos:

```txt
alemd@debian:~/minikube/nextcloud$ kubectl apply -f nextcloud-pvc.yaml 
persistentvolumeclaim/nextcloud-pvc created
alemd@debian:~/minikube/nextcloud$ kubectl apply -f mariadb-pvc.yaml 
persistentvolumeclaim/mariadb-pvc created
```

Comprobamos que se han creado correctamente:

![SRI-K8S-VI.5.png](/img/SRI-K8S-VI.5.png)


#### 3. Modifica los ficheros de despliegue de la aplicación y la base de datos para asociar los volúmenes a cada uno. Según la documentación de la imagen Nextcloud en Docker Hub, la forma más sencilla de hacer persistente la aplicación es montar el volumen en el directorio/var/www/html/.

Modificamos el fichero `nextcloud-deployment.yaml`.

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
          volumeMounts:
            - name: nextcloud-vol
              mountPath: /var/www/html
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
      volumes:
        - name: nextcloud-vol
          persistentVolumeClaim:
            claimName: nextcloud-pvc
```

Modificamos el fichero `mariadb-deployment.yaml`.

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
          volumeMounts:
            - name: mariadb-vol
              mountPath: /var/lib/mysql
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
      volumes:
        - name: mariadb-vol
          persistentVolumeClaim:
            claimName: mariadb-pvc
```

Aplicamos los cambios:

```txt
alemd@debian:~/minikube/nextcloud$ kubectl apply -f mariadb-deployment.yaml
deployment.apps/mariadb-deployment configured
alemd@debian:~/minikube/nextcloud$ kubectl apply -f nextcloud-deployment.yaml
deployment.apps/nextcloud-deployment configured
```

#### 4. Accede a la aplicación, configúrala y sube un fichero.

Accedemos a la aplicación y la instalamos. Una vez instalada subimos un fichero. He creado el fichero persistente.md:

![SRI-K8S-VI.6.png](/img/SRI-K8S-VI.6.png)

#### 5. Comprobemos la persistencia: elimina los despliegues, vuelve a crearlos y vuelve a acceder desde el navegador y comprueba que la aplicación está configurada y mantiene el fichero que habías subido.

Eliminamos los despliegues:

```txt
alemd@debian:~/minikube/nextcloud$ kubectl delete deployments.apps nextcloud-deployment
deployment.apps "nextcloud-deployment" deleted
alemd@debian:~/minikube/nextcloud$ kubectl delete deployments.apps mariadb-deployment 
deployment.apps "mariadb-deployment" deleted
```

Volvemos a crear los despliegues:

```txt
alemd@debian:~/minikube/nextcloud$ kubectl apply -f nextcloud-deployment.yaml
deployment.apps/nextcloud-deployment created
alemd@debian:~/minikube/nextcloud$ kubectl apply -f mariadb-deployment.yaml
deployment.apps/mariadb-deployment created
```

Y volvemos a acceder a la aplicación:

![SRI-K8S-VI.7.png](/img/SRI-K8S-VI.7.png)

Como vemos nos pide volver a iniciar sesión. Metemos el usuario y contraseña que indicamos al configurar nextcloud:

![SRI-K8S-VI.8.png](/img/SRI-K8S-VI.8.png)

Al iniciar sesión vemos que sigue creado el fichero `persistente.md`

---
## **Documento realizado por:**

 ✒️ **Alejandro Montes Delgado** - *2º ASIR*