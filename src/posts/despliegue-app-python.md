---
layout: post
title: Despliegue de aplicaciones python.
excerpt: En esta práctica vamos a desplegar la aplicación del tutorial de django.
date: 2023-01-04
updatedDate: 2023-01-04
tags:
  - post
  - IAW
---

### Tarea 1: Entorno de desarrollo.

Lo primero que haremos será forkear el repositorio y clonarlo a bravo.

```txt
[usuario@bravo ~]$ git clone https://github.com/alemd01/django_tutorial.git
Cloning into 'django_tutorial'...
remote: Enumerating objects: 158, done.
remote: Counting objects: 100% (5/5), done.
remote: Compressing objects: 100% (5/5), done.
remote: Total 158 (delta 0), reused 2 (delta 0), pack-reused 153
Receiving objects: 100% (158/158), 4.25 MiB | 6.63 MiB/s, done.
Resolving deltas: 100% (50/50), done.
```

Una vez clonado el repositorio, lo que haremos será crear un entorno virtual. para ello, Crearemos una carpeta donde guardaremos todos los entornos virtuales y dentro crearemos el entorno virtual.

```txt
[usuario@bravo ~]$ mkdir venv
[usuario@bravo ~]$ cd venv
[usuario@bravo venv]$ python3 -m venv venv
```

Una vez creado el entorno virtual, lo activamos.

```txt
[usuario@bravo venv]$ source bin/activate
(venv) [usuario@bravo venv]$
```

Instalamos los requisitos usando el requirements.txt

```txt
(venv) [usuario@bravo django_tutorial]$ pip install -r requirements.txt 
Collecting Django
  Downloading Django-4.1.5-py3-none-any.whl (8.1 MB)
     |████████████████████████████████| 8.1 MB 1.7 MB/s 
Collecting asgiref<4,>=3.5.2
  Downloading asgiref-3.6.0-py3-none-any.whl (23 kB)
Collecting sqlparse>=0.2.2
  Downloading sqlparse-0.4.3-py3-none-any.whl (42 kB)
     |████████████████████████████████| 42 kB 946 kB/s 
Installing collected packages: sqlparse, asgiref, Django
Successfully installed Django-4.1.5 asgiref-3.6.0 sqlparse-0.4.3
```

Como podemos observar en el fichero settings.py, usaremos sqlite y el nombre de la base de datos será db.sqlite3

```txt
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}
```
Para generar la base de datos, tenemos que realizar una migración del fichero manage.py y así se creará un archivo sqlite3.

```txt
(venv) [usuario@bravo django_tutorial]$ python3 manage.py migrate
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, polls, sessions
Running migrations:
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK
  Applying admin.0003_logentry_add_action_flag_choices... OK
  Applying contenttypes.0002_remove_content_type_name... OK
  Applying auth.0002_alter_permission_name_max_length... OK
  Applying auth.0003_alter_user_email_max_length... OK
  Applying auth.0004_alter_user_username_opts... OK
  Applying auth.0005_alter_user_last_login_null... OK
  Applying auth.0006_require_contenttypes_0002... OK
  Applying auth.0007_alter_validators_add_error_messages... OK
  Applying auth.0008_alter_user_username_max_length... OK
  Applying auth.0009_alter_user_last_name_max_length... OK
  Applying auth.0010_alter_group_name_max_length... OK
  Applying auth.0011_update_proxy_permissions... OK
  Applying auth.0012_alter_user_first_name_max_length... OK
  Applying polls.0001_initial... OK
  Applying sessions.0001_initial... OK
(venv) [usuario@bravo django_tutorial]$ ls
README.md   django_tutorial  polls
db.sqlite3  manage.py        requirements.txt

```

Con el siguiente comando crearemos un usuario para la base de datos:

```txt
(venv) [usuario@bravo django_tutorial]$ python3 manage.py createsuperuser
Username (leave blank to use 'usuario'): alejandro
Email address: aaleemd11@gmail.com
Password: 
Password (again): 
This password is too short. It must contain at least 8 characters.
Bypass password validation and create user anyway? [y/N]: y
Superuser created successfully.
```

probaremos el servidor en desarrollo, para ello utilizaremos el siguiente comando:

```txt
(venv) [usuario@bravo django_tutorial]$ python3 manage.py runserver 0.0.0.0:8000
Watching for file changes with StatReloader
Performing system checks...

System check identified no issues (0 silenced).
January 19, 2023 - 10:05:34
Django version 4.1.5, using settings 'django_tutorial.settings'
Starting development server at http://0.0.0.0:8000/
Quit the server with CONTROL-C.
```

Ahora en alfa, añadiremos una regla DNAT para que el tráfico del puerto 8000 vaya a bravo.

```txt
usuario@alfa:~$ sudo iptables -t nat -A PREROUTING -p tcp --dport 8000 -i ens3 -j DNAT --to 172.16.0.200
usuario@alfa:~$ sudo iptables -L -t nat
Chain PREROUTING (policy ACCEPT)
target     prot opt source               destination         
DNAT       tcp  --  anywhere             anywhere             tcp dpt:http to:172.16.0.200:80
DNAT       udp  --  anywhere             anywhere             udp dpt:domain to:192.168.0.2:53
DNAT       tcp  --  anywhere             anywhere             tcp dpt:smtp to:192.168.0.3
DNAT       tcp  --  anywhere             anywhere             tcp dpt:8000 to:172.16.0.200
```

Una vez hecho esto, podremos acceder desde nuestro navegador web a django en fase de desarrollo:

![IAW-T4.P1.1.png](/img/IAW-T4.P1.1.png)

Nos logeamos con el usuario y contraseña que creamos anteriormente y nos aparecerá el siguiente menú:

![IAW-T4.P1.2.png](/img/IAW-T4.P1.2.png)

En el apartado `Questions` pulsamos `Add` para añadir preguntas. 

![IAW-T4.P1.3.png](/img/IAW-T4.P1.3.png)

Rellenamos 2 preguntas con sus respuestas y pulsamos sobre `Questions` el resultado será el siguiente:

![IAW-T4.P1.4.png](/img/IAW-T4.P1.4.png)

Ahora si nos metemos en /polls veremos las preguntas creadas:

![IAW-T4.P1.5.png](/img/IAW-T4.P1.5.png)

Una vez hecho esto, pondremos en producción django en bravo. Lo primero que haremos será mover django que está en el home a `/var/www`

```txt
(venv) [usuario@bravo ~]$ sudo mv django_tutorial/ /var/www
(venv) [usuario@bravo ~]$ sudo chown -R apache:apache /var/www/django_tutorial/
```

Ahora editamos la configuración de django para indicar la nueva ruta estática:

```txt
(venv) [usuario@bravo www]$ sudo nano django_tutorial/settings.py 
```

Al final del archivo añadimos:

```txt
STATIC_ROOT = '/var/www/django_tutorial/static'
```

Ahora ejecutamos:

```txt
(django) [usuario@bravo django_tutorial]$ python3 manage.py collectstatic

133 static files copied to '/var/www/django_tutorial/static'.

```

Nos salimos del entorno virtual e instalaremos mod_wsgi:

```txt
(django) [usuario@bravo django_tutorial]$ deactivate
[usuario@bravo django_tutorial]$ sudo dnf install python3-mod_wsgi -y
Failed to set locale, defaulting to C.UTF-8
Last metadata expiration check: 2:14:47 ago on Sun Jan 22 12:59:59 2023.
Dependencies resolved.
==========================================================================
 Package               Arch        Version           Repository      Size
==========================================================================
Installing:
 python3-mod_wsgi      x86_64      4.7.1-11.el9      appstream      931 k

Transaction Summary
==========================================================================
Install  1 Package

Total download size: 931 k
Installed size: 5.6 M
Downloading Packages:
python3-mod_wsgi-4.7.1-11.el9.x86_64.rpm  1.7 MB/s | 931 kB     00:00    
--------------------------------------------------------------------------
Total                                     857 kB/s | 931 kB     00:01     

```

Ahora tendríamos que crear el fichero wsgi.py en django pero ya viene creado. Lo que haremos será crear el virtual host:

```txt
[usuario@bravo django_tutorial]$ sudo nano /etc/httpd/sites-available/django.conf
```

Añadimos lo siguiente:

```txt
<VirtualHost *:80>
        ServerName python.alejandro-montes.gonzalonazareno.org
        DocumentRoot /var/www/django_tutorial
        Alias /static/ /var/www/django_tutorial/static/
        WSGIDaemonProcess django_tutorial python-path=/var/www/django_tutorial:/var/www/django/lib/python3.9/site-packages/
        WSGIProcessGroup django_tutorial
        WSGIScriptAlias / /var/www/django_tutorial/django_tutorial/wsgi.py
        ErrorLog /var/log/httpd/django_tutorial_error.log
        CustomLog /var/log/httpd/django_tutorial_access.log combined
</VirtualHost>
```

Activamos el virtual host:

```txt
[usuario@bravo django_tutorial]$ sudo ln -s /etc/httpd/sites-available/django.conf /etc/httpd/sites-enabled/django.conf
```

Reiniciamos apache para que se apliquen los cambios:

```txt
[usuario@bravo django_tutorial]$ sudo systemctl restart httpd
```

Ahora en charlie añadimos un CNAME para todas las zonas llamado python.

```txt
ubuntu@charlie:~$ sudo nano /var/cache/bind/db.dmz.alejandro-montes.gonzalonazareno.org
ubuntu@charlie:~$ sudo nano /var/cache/bind/db.interna.alejandro-montes.gonzalonazareno.org
```

añadimos al final lo siguiente:

```txt
python                  IN      CNAME   bravo
```

ahora reiniciamos el servidor dns y borramos la cache:

```txt
ubuntu@charlie:~$ sudo rndc flush
ubuntu@charlie:~$ sudo systemctl restart bind9
```


Una vez hecho esto, estaría funcionando el servidor dns y el virtualhost. Ahora probaremos a acceder a python.alejandro-montes.gonzalonazareno.org a través de internet:

![IAW-T4.P1.6.png](/img/IAW-T4.P1.6.png)

Por últimos subimos los cambios a git para realizar la migración al vps.

```txt
[usuario@bravo django_tutorial]$ git add .
[usuario@bravo django_tutorial]$ git commit -m "corregido"
[usuario@bravo django_tutorial]$ git push
```


### Tarea 2: Entorno de producción.

Lo primero que haremos será conectarnos al vps a traves de ssh e instalaremos git:

```txt
alemd@nabil:~$ sudo apt install git
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  git-man libcurl3-gnutls liberror-perl patch
Suggested packages:
  git-daemon-run | git-daemon-sysvinit git-doc git-el git-email git-gui
  gitk gitweb git-cvs git-mediawiki git-svn ed diffutils-doc
The following NEW packages will be installed:
  git git-man libcurl3-gnutls liberror-perl patch
0 upgraded, 5 newly installed, 0 to remove and 3 not upgraded.
Need to get 7855 kB of archives.
After this operation, 38.9 MB of additional disk space will be used.
Do you want to continue? [Y/n] y
```

Clonaremos el repositorio con el siguiente comando:

```txt
alemd@nabil:~$ git clone git@github.com:alemd01/django_tutorial.git
```

Ahora instalamos si es necesario el módulo de python para crear entornos virtuales:

```txt
alemd@nabil:~$ sudo apt install python3-venv
```

Y creamos uno:

```txt
alemd@nabil:~$ mkdir django
alemd@nabil:~$ python3 -m venv django
```

Activamos el entorno virtual:

```txt
alemd@nabil:~$ source django/bin/activate
(django) alemd@nabil:~$ 
```

Instalaremos las dependencias de nuestra aplicación:

```txt
(django) alemd@nabil:~/django_tutorial$ pip install -r requirements.txt 
Collecting Django
  Downloading Django-4.1.5-py3-none-any.whl (8.1 MB)
     |████████████████████████████████| 8.1 MB 4.9 MB/s 
Collecting asgiref<4,>=3.5.2
  Downloading asgiref-3.6.0-py3-none-any.whl (23 kB)
Collecting sqlparse>=0.2.2
  Downloading sqlparse-0.4.3-py3-none-any.whl (42 kB)
     |████████████████████████████████| 42 kB 1.5 MB/s 
Installing collected packages: sqlparse, asgiref, Django
Successfully installed Django-4.1.5 asgiref-3.6.0 sqlparse-0.4.3
```

También instalaremos mysqlclient ya que la aplicación trabajará con una base de datos de mariadb:

```txt
(django) alemd@nabil:~/django_tutorial$ sudo apt install python3-dev default-libmysqlclient-dev build-essential -y
(django) alemd@nabil:~/django_tutorial$ pip install mysqlclient
```

Ahora crearemos una base de datos en mysql, con un usuario y sus respectivos privilegios para la base de datos:

```txt
root@nabil:/home/alemd/django_tutorial# mysql -u root

MariaDB [(none)]> CREATE DATABASE DJANGO;
Query OK, 1 row affected (0.009 sec)

MariaDB [(none)]> CREATE USER 'alejandro'@'%' IDENTIFIED BY 'usuario';
Query OK, 0 rows affected (0.021 sec)

MariaDB [(none)]> GRANT ALL PRIVILEGES ON DJANGO.* TO 'alejandro'@'%';
Query OK, 0 rows affected (0.007 sec)

```

Ahora editaremos la aplicación para que utilice MySQL en vez de sqlite3:

```txt
(django) alemd@nabil:~/django_tutorial$ nano django_tutorial/settings.py
```

editamos la zona de databases y tiene que quedar así:

```txt
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'DJANGO',
        'USER': 'alejandro',
        'PASSWORD': 'usuario',
        'HOST': 'localhost',
        'PORT': '',
    }
```

Ahora creamos una copia de seguridad de la base de datos del entorno de desarrollo(bravo) y la subimos a github y actualizamos el repositorio en el vps:

```txt
(django) [usuario@bravo django_tutorial]$ python3 manage.py dumpdata > db.json
(django) [usuario@bravo django_tutorial]$ git add .
(django) [usuario@bravo django_tutorial]$ git commit -m "copia de seguridad"
(django) [usuario@bravo django_tutorial]$ git push
```


Ahora hacemos un pull al vps para que sincronice con el repositorio:

```txt
(django) alemd@nabil:~/django_tutorial$ git pull
(django) alemd@nabil:~/django_tutorial$ ls
README.md  django_tutorial  polls             static
db.json    manage.py        requirements.txt
```

Una vez tengamos la copia de seguridad en el vps, realizaremos la migración:

```txt
(django) alemd@nabil:~/django_tutorial$ python3 manage.py migrate
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, polls, sessions
Running migrations:
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK
  Applying admin.0003_logentry_add_action_flag_choices... OK
  Applying contenttypes.0002_remove_content_type_name... OK
  Applying auth.0002_alter_permission_name_max_length... OK
  Applying auth.0003_alter_user_email_max_length... OK
  Applying auth.0004_alter_user_username_opts... OK
  Applying auth.0005_alter_user_last_login_null... OK
  Applying auth.0006_require_contenttypes_0002... OK
  Applying auth.0007_alter_validators_add_error_messages... OK
  Applying auth.0008_alter_user_username_max_length... OK
  Applying auth.0009_alter_user_last_name_max_length... OK
  Applying auth.0010_alter_group_name_max_length... OK
  Applying auth.0011_update_proxy_permissions... OK
  Applying auth.0012_alter_user_first_name_max_length... OK
  Applying polls.0001_initial... OK
  Applying sessions.0001_initial... OK
```

Ahora cargamos los datos:

```txt
(django) alemd@nabil:~/django_tutorial$ python3 manage.py loaddata db.json 
Installed 53 object(s) from 1 fixture(s)
```

Ahora instalaremos con pip un servidor de aplicaciones como uwsgi:

```txt
alemd@nabil:~$ source django/bin/activate
(django) alemd@nabil:~$ pip install uwsgi
Collecting uwsgi
  Downloading uwsgi-2.0.21.tar.gz (808 kB)
     |████████████████████████████████| 808 kB 5.1 MB/s 
Using legacy 'setup.py install' for uwsgi, since package 'wheel' is not installed.
Installing collected packages: uwsgi
    Running setup.py install for uwsgi ... done
Successfully installed uwsgi-2.0.21
```

Y crearemos una unidad de systemd:

```txt
(django) alemd@nabil:/var/www$ nano django/uwsgi.ini 

[uwsgi]
http = :8080
chdir = /var/www/django_tutorial
wsgi-file = /var/www/django_tutorial/django_tutorial/wsgi.py
processes = 4
threads = 2

```

```txt
(django) alemd@nabil:/var/www$ sudo nano /etc/systemd/system/uwsgi-django.service 

[Unit]
Description=uWSGI instance to serve django_tutorial
After=network.target

[Install]
WantedBy=multi-user.target

[Service]
User=www-data
Group=www-data
Restart=always

ExecStart=/var/www/django/bin/uwsgi --ini /var/www/django/uwsgi.ini
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s TERM $MAINPID

WorkingDirectory=/var/www/django_tutorial
Environment=PYTHONPATH='/var/www/django_tutorial/django_tutorial:/var/www>

PrivateTmp=true

```

Una vez hecho esto recargaremos el demonio de systemctl para que se apliquen los cambios y habilitaremos la unidad:

```txt
(django) alemd@nabil:/var/www/django$ sudo systemctl daemon-reload
(django) alemd@nabil:/var/www/django$ sudo systemctl enable --now uwsgi-django.service
Created symlink /etc/systemd/system/multi-user.target.wants/uwsgi-django.service → /etc/systemd/system/uwsgi-django.service.
```

Ahora configuraremos el virtual host en el servidor nginx del vps:

```txt
(django) alemd@nabil:/var/www/django$ sudo nano /etc/nginx/sites-available/django.conf

server {
        listen 80;
        listen [::]:80;

        root /var/www/django_tutorial;

        index index.html index.php index.htm index.nginx-debian.html;

        server_name python.alejandro-montes.es;

        location / {
                proxy_pass http://localhost:8080;
                include proxy_params;
        }

        location /static {
                alias /var/www/django_tutorial/static;
        }
}

```

Crearemos el enlace simbólico para activar el virtual host:

```txt
(django) alemd@nabil:/var/www/django$ sudo ln -s  /etc/nginx/sites-available/django.conf /etc/nginx/sites-enabled/
```

Ahora crearemos una entrada en la configuración de nuestro dns para poder acceder a través de internet.

![IAW-T4.P1.7.png](/img/IAW-T4.P1.7.png)

Como podemos ver en la casilla marcada, hemos indicado a nuestro proveedor de dns un cname a nabil que es la máquina del vps. Ahora muestro una captura de pantalla de django funcionando en el vps:

![IAW-T4.P1.8.png](/img/IAW-T4.P1.8.png)

### Tarea 3: Modificación de nuestra aplicación.

Lo primero que realizaremos será entrar en el entorno de desarrollo (bravo) y modificaremos el apartado polls para que aparezca nuestro nombre:


    [usuario@bravo django_tutorial]$ nano polls/templates/polls/index.html


Añadimos lo siguiente:

![IAW-T4.P1.13.png](/img/IAW-T4.P1.13.png)

Ahora cambiaremos el fondo de la aplicación para ello bajaremos una imagen y la pasaremos al servidor por scp. Una vez la imagen esté en el servidor, la moveremos a la siguiente ruta:

    [usuario@bravo ~]$ sudo mv background2.jpg /var/www/django_tutorial/static/polls/images/ 

Y ahora editamos el css para aplicar los cambios:

```txt
[usuario@bravo ~]$ sudo nano /var/www/django_tutorial/static/polls/style.css 
```

El contenido del fichero ya editado es el siguiente:

```txt
li a {
    color: green;
    background-color: blanchedalmond;
}

body {
    background: white url("images/background2.jpg");
}
```

Muestro una captura de bravo con la pantalla editada:

![IAW-T4.P1.9.png](/img/IAW-T4.P1.9.png)

Una vez hecho esto, añadiremos una nueva tabla en la base de datos, para ello editaremos el archivo models.py situado en el directorio polls dentro de nuestra aplicación django:

```txt
[usuario@bravo django_tutorial]$ nano polls/models.py 
```

Añadimos lo siguiente al final del fichero:

```txt
class Categoria(models.Model):
    Abr = models.CharField(max_length=4)
    Nombre = models.CharField(max_length=50)

    def __str__(self):
        return self.Abr+" - "+self.Nombre
```

Crearemos una nueva migración:

```txt
(django) [usuario@bravo django_tutorial]$ python3 manage.py makemigrations
Migrations for 'polls':
  polls/migrations/0002_categoria.py
    - Create model Categoria
(django) [usuario@bravo django_tutorial]$ python3 manage.py migrate   
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, polls, sessions
Running migrations:
  Applying polls.0002_categoria... OK
```

Ahora añadimos el nuevo modelo al panel de administración:


```txt
(django) [usuario@bravo django_tutorial]$ nano polls/admin.py 
```

La segunda línea del fichero la cambiamos por la siguiente:

```txt
from .models import Choice, Question, Categoria
```

Y al final agregamos:

```txt
admin.site.register(Categoria)
```

Reiniciamos el servicio y muestro el panel de administración:

```txt
(django) [usuario@bravo django_tutorial]$ sudo systemctl restart httpd
```

![IAW-T4.P1.10.png](/img/IAW-T4.P1.10.png)


Como vemos, en los polls ahora aparece `Categorias`. Ahora pasaremos los cambios a bravo, para ello actualizaremos nuestro repositorio de github con todos los cambios:

```txt
(django) [usuario@bravo django_tutorial]$ git add .
(django) [usuario@bravo django_tutorial]$ git commit -m "modificacion"
[master 5122a1d] modificacion
 7 files changed, 35 insertions(+), 4 deletions(-)
 rewrite db.json (100%)
 create mode 100644 polls/migrations/0002_categoria.py
 create mode 100644 static/polls/images/background2.jpg
(django) [usuario@bravo django_tutorial]$ git push
Enumerating objects: 27, done.
Counting objects: 100% (27/27), done.
Compressing objects: 100% (15/15), done.
Writing objects: 100% (15/15), 43.47 KiB | 8.69 MiB/s, done.
Total 15 (delta 6), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (6/6), completed with 6 local objects.
To github.com:alemd01/django_tutorial.git
   48e377c..5122a1d  master -> master
```

Ahora en producción actualizamos los cambios:

```txt
alemd@nabil:/var/www/django_tutorial$ git pull
```

Ahora haremos un collectstatic para que cambie la parte estática de la aplicación:

```txt
(django) alemd@nabil:/var/www/django_tutorial$ python3 manage.py collectstatic

You have requested to collect static files at the destination
location as specified in your settings:

    /var/www/django_tutorial/static

This will overwrite existing files!
Are you sure you want to do this?

Type 'yes' to continue, or 'no' to cancel: yes
```

Por último muestro los cambios en producción:

Primero muestro que polls tiene el background cambiado y muestra mi nombre:

![IAW-T4.P1.11.png](/img/IAW-T4.P1.11.png)


Ahora muestro que en la zona de administración se ha creado el nuevo polls:


![IAW-T4.P1.12.png](/img/IAW-T4.P1.12.png)


---

## **Documento realizado por:**

 ✒️ **Alejandro Montes Delgado** - *2º ASIR*
