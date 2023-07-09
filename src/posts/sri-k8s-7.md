---
layout: post
title: Ejercicios de K8s VII.
excerpt: Séptimo taller de la unidad de Kubernetes.
date: 2023-02-23
updatedDate: 2023-02-23
tags:
  - post
  - SRI
  - Kubernetes
---

### Instalación de un CMS con Helm.

Vamos a instalar el CMS Wordpress usando Helm. Para ello, realiza los siguientes pasos:


#### 1. Instala la última versión de Helm.

Para instalar helm, tenemos que seguir los siguientes pasos:

```txt
alemd@debian:~/minikube$ curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--    0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--  100  1700  100  1700    0     0   2014      0 --:--:-- --:--:-- --:--:--  2016
```

Instalamos el siguiente paquete:

```txt
alemd@debian:~/minikube$ sudo apt-get install apt-transport-https --yes
Leyendo lista de paquetes... Hecho
Creando árbol de dependencias... Hecho
Leyendo la información de estado... Hecho
apt-transport-https ya está en su versión más reciente (2.2.4).
0 actualizados, 0 nuevos se instalarán, 0 para eliminar y 98 no actualizados.
```

Ejecutamos el siguiente comando:

```txt
alemd@debian:~/minikube$ echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
deb [arch=amd64 signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main
```

Realizamos un update:

```txt
alemd@debian:~/minikube$ sudo apt update
Obj:1 http://repository.spotify.com stable InRelease
Obj:2 http://download.virtualbox.org/virtualbox/debian bullseye InRelease
Obj:3 https://deb.debian.org/debian bullseye InRelease                   
Obj:4 https://security.debian.org/debian-security bullseye-security InRelease
Obj:5 http://packages.microsoft.com/repos/code stable InRelease          
Obj:6 https://deb.debian.org/debian bullseye-backports InRelease         
Obj:7 https://deb.debian.org/debian bullseye-updates InRelease           
Des:8 https://baltocdn.com/helm/stable/debian all InRelease [7.651 B]    
Des:9 https://baltocdn.com/helm/stable/debian all/main amd64 Packages [3.268 B]
Descargados 10,9 kB en 2s (5.884 B/s)
Leyendo lista de paquetes... Hecho
Creando árbol de dependencias... Hecho
Leyendo la información de estado... Hecho
Se pueden actualizar 98 paquetes. Ejecute «apt list --upgradable» para verlos.
```

Instalamos helm:

```txt
alemd@debian:~/minikube$ sudo apt install helm
Leyendo lista de paquetes... Hecho
Creando árbol de dependencias... Hecho
Leyendo la información de estado... Hecho
Se instalarán los siguientes paquetes NUEVOS:
  helm
0 actualizados, 1 nuevos se instalarán, 0 para eliminar y 98 no actualizados.
Se necesita descargar 15,0 MB de archivos.
Se utilizarán 46,9 MB de espacio de disco adicional después de esta operación.
Des:1 https://baltocdn.com/helm/stable/debian all/main amd64 helm amd64 3.11.1-1 [15,0 MB]
Descargados 15,0 MB en 40s (373 kB/s)                                    
Seleccionando el paquete helm previamente no seleccionado.
(Leyendo la base de datos ... 373507 ficheros o directorios instalados act
ualmente.)
Preparando para desempaquetar .../helm_3.11.1-1_amd64.deb ...
Desempaquetando helm (3.11.1-1) ...
Configurando helm (3.11.1-1) ...
Procesando disparadores para man-db (2.9.4-2) ...
```

Como podemos comprobar se ha instalado correctamente helm:

```txt
alemd@debian:~/minikube$ helm version
version.BuildInfo{Version:"v3.11.1", GitCommit:"293b50c65d4d56187cd4e2f390f0ada46b4c4737", GitTreeState:"clean", GoVersion:"go1.18.10"}
```

#### 2. Añade el repositorio de bitnami

Para añadir el repositorio de bitnami, ejecutamos el siguiente comando:

```txt
alemd@debian:~/minikube$ helm repo add bitnami https://charts.bitnami.com/bitnami
"bitnami" has been added to your repositories
```

Listamos los repositorios y vemos que lo tenemos añadido:

```txt
alemd@debian:~/minikube$ helm repo list
NAME   	URL                               
bitnami	https://charts.bitnami.com/bitnami
```

#### 3. Busca el chart de bitnami para la instalación de Wordpress.

Actualizamos el repositorio:

```txt
alemd@debian:~/minikube$ helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "bitnami" chart repository
Update Complete. ⎈Happy Helming!⎈
```

Y buscamos el chart de bitnami para instalar Wordpress:

```txt
alemd@debian:~/minikube$ helm search repo wordpress
NAME                   	CHART VERSION	APP VERSION	DESCRIPTION                                       
bitnami/wordpress      	15.2.45      	6.1.1      	WordPress is the world's most popular blogging ...
bitnami/wordpress-intel	2.1.31       	6.1.1      	DEPRECATED WordPress for Intel is the most popu...
```

#### 4. Busca la documentación del chart y comprueba los parámetros para cambiar el tipo de Service y el nombre del blog.

Los parámetros que tenemos que usar para cambiar el tipo de service es `--set service.type=` y para cambiar el nombre del blog usamos  `--set wordpressBlogName=`. A continuación muestro el comando completo.



#### 5. Instala el chart definiendo el tipo del Service como NodePort y poniendo tu nombre como nombre del blog.

```txt
alemd@debian:~/minikube$ helm install servidor-web bitnami/wordpress --set service.type=NodePort --set wordpressBlogName=amontes
NAME: serverweb
LAST DEPLOYED: Thu Feb 23 08:54:04 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: wordpress
CHART VERSION: 15.2.45
APP VERSION: 6.1.1

** Please be patient while the chart is being deployed **

Your WordPress site can be accessed through the following DNS name from within your cluster:

    serverweb-wordpress.default.svc.cluster.local (port 80)

To access your WordPress site from outside the cluster follow the steps below:

1. Get the WordPress URL by running these commands:

   export NODE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].nodePort}" services serverweb-wordpress)
   export NODE_IP=$(kubectl get nodes --namespace default -o jsonpath="{.items[0].status.addresses[0].address}")
   echo "WordPress URL: http://$NODE_IP:$NODE_PORT/"
   echo "WordPress Admin URL: http://$NODE_IP:$NODE_PORT/admin"

2. Open a browser and access WordPress using the obtained URL.

3. Login with the following credentials below to see your blog:

  echo Username: user
  echo Password: $(kubectl get secret --namespace default serverweb-wordpress -o jsonpath="{.data.wordpress-password}" | base64 -d)

```

#### 6. Comprueba los Pods, ReplicaSet, Deployment y Services que se han creado.

```txt
alemd@debian:~/minikube$ kubectl get all
NAME                                          READY   STATUS    RESTARTS   AGE
pod/servidor-web-mariadb-0                    1/1     Running   0          2m25s
pod/servidor-web-wordpress-86cfdd7869-hpzqt   1/1     Running   1          2m25s

NAME                             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
service/kubernetes               ClusterIP   10.96.0.1       <none>        443/TCP                      12m
service/servidor-web-mariadb     ClusterIP   10.104.242.8    <none>        3306/TCP                     2m26s
service/servidor-web-wordpress   NodePort    10.107.162.99   <none>        80:30535/TCP,443:32693/TCP   2m26s

NAME                                     READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/servidor-web-wordpress   1/1     1            1           2m25s

NAME                                                DESIRED   CURRENT   READY   AGE
replicaset.apps/servidor-web-wordpress-86cfdd7869   1         1         1       2m25s

NAME                                    READY   AGE
statefulset.apps/servidor-web-mariadb   1/1     2m25s

```

#### 7. Accede a la aplicación.

![SRI-K8S-VII.1.png](/img/SRI-K8S-VII.1.png)


---
## **Documento realizado por:**

 ✒️ **Alejandro Montes Delgado** - *2º ASIR*