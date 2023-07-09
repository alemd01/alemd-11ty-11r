---
layout: post
title: "Migración de Aplicaciones Web al VPS"
excerpt: En este artículo, documentaré el proceso de migración de GLPI y Nextcloud.
date: 2022-11-24
updatedDate: 2022-11-24
tags:
  - post
  - IAW
---

### GLPI

La primera aplicación web que migraremos será glpi, a continuación detallo los pasos que he seguido para migrarla.

#### Creación del virtual Host.

El primer paso que he dado para realizar la migración es configurar un virtualhost en mi vps. Muestro la configuración del virtual host:

```txt
upstream php-handler {
    #server 127.0.0.1:9000;
    server unix:/var/run/php/php7.4-fpm.sock;
}

map $arg_v $asset_immutable {
    "" "";
    default "immutable";
}

server {
    listen 80;
    listen [::]:80;
    server_name www.alejandro-montes.es;
    rewrite ^/$ /portal;
    root /var/www/alejandromontes;

    location = /robots.txt {
        allow all;
        log_not_found off;
        access_log off;
    }

    location ^~ /.well-known {

        location = /.well-known/carddav { return 301 /cloud/remote.php/dav/; }
        location = /.well-known/caldav  { return 301 /cloud/remote.php/dav/; }

        location /.well-known/acme-challenge    { try_files $uri $uri/ =404; }
        location /.well-known/pki-validation    { try_files $uri $uri/ =404; }

        return 301 /cloud/index.php$request_uri;
    }
    location ^~ /portal {
        client_max_body_size 512M;
        client_body_timeout 300s;
        fastcgi_buffers 64 4K;

        gzip on;
        gzip_vary on;
        gzip_comp_level 4;
        gzip_min_length 256;
        gzip_proxied expired no-cache no-store private no_last_modified no_etag auth;
        gzip_types application/atom+xml application/javascript application/json application/ld+json application/manifest+json application/rss+xml appl
ication/vnd.geo+json application/vnd.ms-fontobject application/wasm application/x-font-ttf application/x-web-app-manifest+json application/xhtml+xml a
pplication/xml font/opentype image/bmp image/svg+xml image/x-icon text/cache-manifest text/css text/plain text/vcard text/vnd.rim.location.xloc text/v
tt text/x-component text/x-cross-domain-policy;

        client_body_buffer_size 512k;
        add_header Referrer-Policy                      "no-referrer"   always;
        add_header X-Content-Type-Options               "nosniff"       always;
        add_header X-Download-Options                   "noopen"        always;
        add_header X-Frame-Options                      "SAMEORIGIN"    always;
        add_header X-Permitted-Cross-Domain-Policies    "none"          always;
        add_header X-Robots-Tag                         "none"          always;
        add_header X-XSS-Protection                     "1; mode=block" always;

        fastcgi_hide_header X-Powered-By;

        index index.php index.html /portal/index.php$request_uri;

        location ~ \.php(?:$|/) {

            fastcgi_split_path_info ^(.+?\.php)(/.*)$;
            set $path_info $fastcgi_path_info;

            try_files $fastcgi_script_name =404;

            include fastcgi_params;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            fastcgi_param PATH_INFO $path_info;

            fastcgi_param modHeadersAvailable true;         # Avoid sending the security headers twice
            fastcgi_param front_controller_ac8tive true;     # Enable pretty urls
            fastcgi_pass php-handler;

            fastcgi_intercept_errors on;
            fastcgi_request_buffering off;

            fastcgi_max_temp_file_size 0;
        }

        location /portal {
            try_files $uri $uri/ /portal/index.php$request_uri;
        }
    }
    location ~ /\.ht {
          deny all;
         }

    location ^~ /cloud {
        client_max_body_size 513M;
        client_body_timeout 300s;
        fastcgi_buffers 64 4K;
        gzip on;
        gzip_vary on;
        gzip_vary on;
        gzip_comp_level 4;
        gzip_min_length 256;
        gzip_proxied expired no-cache no-store private no_last_modified no_etag auth;
        gzip_types application/atom+xml application/javascript application/json application/ld+json application/manifest+json application/rss+xml appl
ication/vnd.geo+json application/vnd.ms-fontobject application/wasm application/x-font-ttf application/x-web-app-manifest+json application/xhtml+xml a
pplication/xml font/opentype image/bmp image/svg+xml image/x-icon text/cache-manifest text/css text/plain text/vcard text/vnd.rim.location.xloc text/v
tt text/x-component text/x-cross-domain-policy;
        add_header Referrer-Policy                      "no-referrer"   always;
        add_header X-Content-Type-Options               "nosniff"       always;
        add_header X-Download-Options                   "noopen"        always;
        add_header X-Frame-Options                      "SAMEORIGIN"    always;
        add_header X-Permitted-Cross-Domain-Policies    "none"          always;
        add_header X-Robots-Tag                         "none"          always;
        add_header X-XSS-Protection                     "1; mode=block" always;
        fastcgi_hide_header X-Powered-By;
        index index.php index.html /cloud/index.php$request_uri;

        location = /cloud {
            if ( $http_user_agent ~ ^DavClnt ) {
                return 302 /cloud/remote.php/webdav/$is_args$args;
            }
        }

        location ~ ^/cloud/(?:build|tests|config|lib|3rdparty|templates|data)(?:$|/)    { return 404; }
        location ~ ^/cloud/(?:\.|autotest|occ|issue|indie|db_|console)                  { return 404; }
        location ~ \.php(?:$|/) {
            rewrite ^/cloud/(?!index|remote|public|cron|core\/ajax\/update|status|ocs\/v[12]|updater\/.+|oc[ms]-provider\/.+|.+\/richdocumentscode\/pr
oxy) /cloud/index.php$request_uri;

            fastcgi_split_path_info ^(.+?\.php)(/.*)$;
            set $path_info $fastcgi_path_info;
            try_files $fastcgi_script_name =404;
            include fastcgi_params;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            fastcgi_param PATH_INFO $path_info;
           # fastcgi_param HTTPS on;

            fastcgi_param modHeadersAvailable true;         # Avoid sending the security headers twice
            fastcgi_param front_controller_active true;     # Enable pretty urls
            fastcgi_pass php-handler;

            fastcgi_intercept_errors on;
            fastcgi_request_buffering off;

            fastcgi_max_temp_file_size 0;
        }
        location ~ \.(?:css|js|svg|gif|png|jpg|ico|wasm|tflite|map)$ {
            try_files $uri /cloud/index.php$request_uri;
            add_header Cache-Control "public, max-age=15778463, $asset_immutable";
            access_log off;     # Optional: Don't log access to assets

            location ~ \.wasm$ {
                default_type application/wasm;
            }
        }
        location ~ \.woff2?$ {
            try_files $uri /cloud/index.php$request_uri;
            expires 7d;         # Cache-Control policy borrowed from `.htaccess`
            access_log off;     # Optional: Don't log access to assets
        }
        location /cloud/remote {
            return 301 /cloud/remote.php$request_uri;
        }
        location /cloud {
            try_files $uri $uri/ /cloud/index.php$request_uri;
        }
    }
}
```

Activamos el virtualhost.
```txt
root@nabil:/etc/nginx/sites-enabled# sudo ln -s /etc/nginx/sites-available/alejandromontes.conf /etc/nginx/sites-enabled/
```

Reiniciamos nginx.

```txt
root@nabil:/etc/nginx/sites-enabled# systemctl reload nginx
```


#### Creación y configuración de la base de datos.

Una vez creado el virtualhost, configuraremos la base de datos del vps, para ello, lo primero que debemos de hacer es añadir un nombre de dominio en el /etc/hosts.

```txt
nano /etc/hosts
```
añadimos la siguiente línea:

```txt
127.0.0.1       bd.alejandro-montes.es
```

Ahora, accederemos al servidor mariadb. Crearemos un usuario, una base de datos y daremos privilegios al usuario sobre la base de datos.

```txt
MariaDB [(none)]> CREATE DATABASE glpi;
Query OK, 1 row affected (0.005 sec)
```

```txt
MariaDB [(none)]> CREATE USER 'alemd'@'localhost' IDENTIFIED BY 'passwd';
```

```txt
MariaDB [(none)]> GRANT ALL PRIVILEGES ON glpi.* TO 'alemd'@'LOCALHOST';
Query OK, 0 rows affected (0.003 sec)

MariaDB [(none)]> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.004 sec)
```

#### Realizando la migración.

En el escenario vagrant, teníamos una máquina que actuaba de servidor web y otra del servidor de base de datos. Empezaremos la migración con la máquina que actuaba como servidor de base de datos. Crearemos una copia de seguridad del servidor de base de datos.

```txt
root@servidorbd-alejandro:/home/vagrant# mysqldump glpi > ./backup.sql
root@servidorbd-alejandro:/home/vagrant# ls -lah
total 772K
drwxr-xr-x 3 vagrant vagrant 4.0K Nov 24 09:42 .
drwxr-xr-x 3 root    root    4.0K Sep 12 05:17 ..
-rw------- 1 vagrant vagrant  406 Nov 22 19:18 .bash_history
-rw-r--r-- 1 vagrant vagrant  220 Mar 27  2022 .bash_logout
-rw-r--r-- 1 vagrant vagrant 3.5K Mar 27  2022 .bashrc
-rw------- 1 vagrant vagrant   33 Nov 22 18:05 .mysql_history
-rw-r--r-- 1 vagrant vagrant  807 Mar 27  2022 .profile
drwx------ 2 vagrant vagrant 4.0K Nov 17 08:59 .ssh
-rw-r--r-- 1 root    root    739K Nov 24 09:42 backup.sql
```

Ahora mediante scp mandamos la copia de seguridad. Tenemos que tener en cuenta que en nuestro vps no está la clave pública del servidor de base de datos, lo que haré será enviarlo a mi máquina y después al vps.

Primero lo enviamos a mi máquina:

```txt
root@servidorbd-alejandro:/home/vagrant# scp ./backup.sql alemd@192.168.122.1:
The authenticity of host '192.168.122.1 (192.168.122.1)' can't be established.
ECDSA key fingerprint is SHA256:P4VynMzXNSUoRSzuUrFgWShiT4hYwT/y6DqQ7Sf/ujs.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.122.1' (ECDSA) to the list of known hosts.
alemd@192.168.122.1's password: 
backup.sql                                    100%  739KB  39.9MB/s   00:00  
```

desde mi máquina lo enviamos al servidor vps.

```txt
alemd@debian:~$ scp backup.sql alemd@nabil.alejandro-montes.es:~/
backup.sql                                    100%  739KB 622.3KB/s   00:01 
```

Ahora el directorio donde está alojada la aplicación web lo comprimiremos y lo enviaremos por scp al vps.

```txt
vagrant@servidorweb-alejandro:/var/www$ sudo tar -zcf glpi.tar.gz alejandromontes/*
vagrant@servidorweb-alejandro:/var/www$ sudo scp ./glpi.tar.gz alemd@192.168.122.1:~/
alemd@debian:~$ scp glpi.tar.gz alemd@nabil.alejandro-montes.es:~/
glpi.tar.gz                                   100%   55MB   2.7MB/s   00:20 
```

Ahora restauraremos la copia de seguridad de la base de datos indicando cual es la base de datos que queremos que recupere.

```txt
root@nabil:/home/alemd# mysql glpi < backup.sql
```

Una vez restaurada la copia de seguridad, extraeremos la aplicación web.

```txt
alemd@nabil:/var/www/glpi$ sudo tar -zxf ~/glpi.tar.gz 
```

Le cambiamos el dueño al host virtual.

```txt
alemd@nabil:/var/www$ sudo chown www-data:www-data glpi/
```

La aplicación web está configurada para que acceda a una base de datos remota. Cambiaremos la configuración haciendo los siguientes pasos:

Nos movemos a la carpeta config y editamos el archivo config_db.php.

```txt
root@nabil:/var/www/glpi# cd /var/www/glpi/config
root@nabil:/var/www/glpi/config# nano config_db.php 
```

Lo editamos y queda de la siguiente manera:

```txt
<?php
class DB extends DBmysql {
   public $dbhost = 'bd.alejandro-montes.es';
   public $dbuser = 'alemd';
   public $dbpassword = 'passwd_user';
   public $dbdefault = 'glpi';
   public $use_utf8mb4 = true;
   public $allow_myisam = false;
   public $allow_datetime = false;
   public $allow_signed_keys = false;
}
```

Antes de entrar al sitio, instalaremos las librerías que necesitamos en el escenario local.

```txt
root@nabil:/var/www/glpi/config# apt install  php7.4-mbstring php7.4-curl php7.4-gd php7.4-xml php7.4-intl php7.4-bz2 php7.4-ldap php7.4-xmlrpc php7.4-zip 
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following packages were automatically installed and are no longer required:
  apache2-bin apache2-data apache2-utils libapr1 libaprutil1
  libaprutil1-dbd-sqlite3 libaprutil1-ldap libjansson4 liblua5.3-0
Use 'sudo apt autoremove' to remove them.
The following additional packages will be installed:
  libonig5 libxmlrpc-epi0 libzip4
The following NEW packages will be installed:
  libonig5 libxmlrpc-epi0 libzip4 php7.4-bz2 php7.4-curl php7.4-gd php7.4-intl
  php7.4-ldap php7.4-mbstring php7.4-xml php7.4-xmlrpc php7.4-zip
0 upgraded, 12 newly installed, 0 to remove and 0 not upgraded.
Need to get 1025 kB of archives.
```
reiniciamos el servicio para que funcione correctamente:

```txt
root@nabil:/var/www/glpi/config# systemctl reload php7.4-fpm
```

### Nextcloud

#### Creación y configuración de la base de datos.

En el vps creamos la base de datos, el usuario y damos al usuario privilegios sobre la base de datos.

```txt
MariaDB [(none)]> CREATE USER 'nextclouduser'@'localhost' IDENTIFIED BY 'PASSWORD';
Query OK, 0 rows affected (0.004 sec)
MariaDB [(none)]> CREATE DATABASE nextcloud_db;
Query OK, 1 row affected (0.000 sec)
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nextcloud_db.* TO 'nextclouduser'@'localhost';
Query OK, 0 rows affected (0.002 sec)
```

Nos vamos al servidor del anterior escenario y exportamos la base de datos:

```txt
root@servidorbd-alejandro:/home/vagrant# mysqldump nextcloud_db > backup_nextcloud.sql
```

Exportamos el archivo generado al vps.

```txt
alemd@debian:~$ scp backup_nextcloud.sql alemd@nabil.alejandro-montes.es:~/
backup_nextcloud.sql                                                                                                100%  197KB 189.8KB/s   00:01    
```

#### Realizando la migración.

Una vez exportada la base de datos, exportaremos el nextcloud, para ello lo comprimiremos y lo enviaremos por scp al vps.

```txt
vagrant@servidorweb-alejandro:/var/www$ sudo tar -zcf nextcloud.tar.gz nextcloud
vagrant@servidorweb-alejandro:/var/www$ sudo scp ./nextcloud.tar.gz alemd@192.168.122.1:~/
alemd@192.168.122.1's password: 
nextcloud.tar.gz                                                                                                    100%  246MB  69.7MB/s   00:03   
alemd@debian:~$ scp nextcloud.tar.gz alemd@nabil.alejandro-montes.es:~/
nextcloud.tar.gz                                                                                                    100%  246MB   4.6MB/s   00:53 
```

Regeneramos la base de datos de nextcloud.

```txt
root@nabil:/home/alemd# mysql nextcloud_bd < backup.sql
```

extraemos el nexcloud.

```txt
root@nabil:/var/www/alejandromontes/cloud# tar -zxf nextcloud.tar.gz
```

editamos el fichero de configuración en el que indica donde está alojada la base de datos.

```txt
root@nabil:/var/www/alejandromontes/cloud# nano config/config.php 
```

```txt
<?php
7$CONFIG = array (
  'instanceid' => 'ocku5u9frj18',
  'passwordsalt' => 'cVo/fptYzJDN8NjvuFh1ey5IoxZ18R',
  'secret' => 'ZKcMNlQ1H4uE+J2u95cg7LSnM+/KkLHP0NbvtE8VEYfskTBR',
  'trusted_domains' => 
  array (
    0 => 'www.alejandro-montes.es',
  ),
  'datadirectory' => '/var/www/alejandromontes/cloud/data',
  'dbtype' => 'mysql',
  'version' => '22.1.1.2',
  'overwrite.cli.url' => 'http://www.alejandro-montes.es/cloud',
  'dbname' => 'nextcloud_db',
  'dbhost' => 'localhost',
  'dbport' => '',
  'dbtableprefix' => 'oc_',
  'mysql.utf8mb4' => true,
  'dbuser' => 'nextclouduser',
  'dbpassword' => 'password',
  'installed' => true,
);
```




## **Documento realizado por:**

 ✒️ **Alejandro Montes Delgado** - *2º ASIR*
