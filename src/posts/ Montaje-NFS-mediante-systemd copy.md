---
layout: post
title: "Criptografía III. Certificados Digitales. HTTPS."
excerpt: "En este post realizaré la memoria de Criptografía 3."
date: 2023-01-05
tags:
  - post
  - SAD
---

## Certificado digital de persona física.

### Tarea 1: Instalación del certificado.

#### Una vez que hayas obtenido tu certificado, explica brevemente como se instala en tu navegador favorito.

Para importarlo en Firefox hacemos los siguientes pasos:

* Abrimos los ajustes de Firefox.

* Vamos a la pestaña Privacidad & Seguridad.

* En seguridad, hay un apartado de certificados y le damos a la opción de ver certificados. Se nos abrirá la siguiente pestaña:

![SAD-cripto3.t1.1.png](/img/SAD-cripto3.t1.1.png)

* Como vemos mi certificado ya está importado, pero tenemos que elegir la opción de importar y elegimos nuestro certificado. Le damos a aceptar para que se guarden los cambios y lo habremos instalado correctamente.

#### Muestra una captura de pantalla donde se vea las preferencias del navegador donde se ve instalado tu certificado.

Para no repetir la captura, en el anterior apartado muestro mi certificado instalado.

#### ¿Cómo puedes hacer una copia de tu certificado?, ¿Como vas a realizar la copia de seguridad de tu certificado?. Razona la respuesta.

Estando en el mismo apartado de ajuste que en el primer apartado, si seleccionamos nuestro certificado, nos da una opción que es  `Hacer copia...` .

![SAD-cripto3.t1.2.png](/img/SAD-cripto3.t1.2.png)

Elegimos un nombre para la copia y lo guardamos. Tendremos que elegir una contraseña de respaldo de la copia del certificado, para cuando vayamos a restaurarla la introduzcamos.

![SAD-cripto3.t1.3.png](/img/SAD-cripto3.t1.3.png)

#### Investiga como exportar la clave pública de tu certificado.

Lo primero que hacemos es generar el par de claves del certificado con el siguiente comando:

```txt
alemd@debian:~/2ASIR$ openssl pkcs12 -in MONTES_DELGADO_ALEJANDRO___49125015M.p12 -nocerts -nodes -out MONTES_DELGADO_ALEJANDRO___49125015M.key
Enter Import Password:
alemd@debian:~/2ASIR$
```

Ahora usamos el siguiente comando para extraer la clave pública:

```txt
alemd@debian:~/2ASIR$ openssl rsa -in MONTES_DELGADO_ALEJANDRO___49125015M.key -pubout -out MONTES_DELGADO_ALEJANDRO___49125015M.pub
writing RSA key
```

muestro un ls del directorio para comprobar que se han generado los ficheros correspondientes:

```txt
alemd@debian:~/2ASIR$ ls | grep ^MONTES*
MONTES_DELGADO_ALEJANDRO___49125015M.key
MONTES_DELGADO_ALEJANDRO___49125015M.p12
MONTES_DELGADO_ALEJANDRO___49125015M.pub
```

### Tarea 2: Validación del certificado.

#### Instala en tu ordenador el software autofirma y desde la página de VALIDe valida tu certificado. Muestra capturas de pantalla donde se comprueba la validación.

Lo primero que haremos será instalar las dependencias correspondiente para que funcione autofirma:

```txt
sudo apt install default-jdk libnss3-tools
```

Desde la página de [autofirma](https://autofirma.net/como-instalar-autofirma-en-linux/) descargaremos el zip y lo descomprimiremos.

```txt
alemd@debian:~/2ASIR$ unzip AutoFirma_Linux.zip
```
Como podemos comprobar autofirma funciona correctamente.

![SAD-cripto3.t2.1.png](/img/SAD-cripto3.t2.1.png)

Ahora en la página web de VALIDe, validaremos el certificado en la sección de validar el certificado, subiremos nuestro certificado y validaremos el captcha.

![SAD-cripto3.t2.png](/img/SAD-cripto3.t2.png)

### Tarea 3: Firma electrónica.

#### Utilizando la página VALIDe y el programa autofirma, firma un documento con tu certificado y envíalo por correo a un compañero.

Si usamos VALIDe, en la sección de realizar firma, pulsamos sobre el botón firmar y subimos un archivo nuestro. Guardaremos el documento y se lo enviaremos a nuestro compañero.

![SAD-cripto3.t3.1.png](/img/SAD-cripto3.t3.1.png)

Ahora muestro como he firmado otro documento con Autofirma. Pulsamos sobre seleccionar ficheros a firmar y buscamos el fichero que queremos firmar. Como podemos ver en los detalles está firmado por mí.

![SAD-cripto3.t3.2.png](/img/SAD-cripto3.t3.2.png)

Ahora envío los ficheros a mi compañero Juanje por correo.

#### Tu debes recibir otro documento firmado por un compañero y utilizando las herramientas anteriores debes visualizar la firma (Visualizar Firma) y (Verificar Firma). ¿Puedes verificar la firma aunque no tengas la clave pública de tu compañero?, ¿Es necesario estar conectado a internet para hacer la validación de la firma?. Razona tus respuestas.

Visualizo la firma en VALIDe:

![SAD-cripto3.t3.6.png](/img/SAD-cripto3.t3.6.png)

Visualizo la firma en Autofirma:

![SAD-cripto3.t3.5.png](/img/SAD-cripto3.t3.5.png)

Juanje me ha enviado 2 ficheros. El primero lo valido con VALIDe:

![SAD-cripto3.t3.4.png](/img/SAD-cripto3.t3.4.png)

Ahora valido el otro fichero con Autofirma:

![SAD-cripto3.t3.3.png](/img/SAD-cripto3.t3.3.png)



#### Entre dos compañeros, firmar los dos un documento, verificar la firma para comprobar que está firmado por los dos.

Ahora con Autofirma, firmo un documento que me ha enviado Juanje:

![SAD-cripto3.t3.7.png](/img/SAD-cripto3.t3.7.png)

Como podemos ver, el fichero es firmado por Juanje y por mí. También lo he firmado con VALIDe:

![SAD-cripto3.t3.8.png](/img/SAD-cripto3.t3.8.png)


### Tarea 4: Autentificación.

#### Utilizando tu certificado accede a alguna página de la administración pública )cita médica, becas, puntos del carnet,…). Entrega capturas de pantalla donde se demuestre el acceso a ellas.

Entraré en la [Sede de la DGT](https://sede.dgt.gob.es/es/) con mi certificado digital. Puelsaremos en el botón de arriba a la derecha `Acceso a mi dgt` :

![SAD-cripto3.t3.9.png](/img/SAD-cripto3.t3.9.png)

Nos llevará a la pasarela de clave para iniciar sesión con el certificado electrónico y elegiremos nuestro certificado.

![SAD-cripto3.t3.10.png](/img/SAD-cripto3.t3.10.png)

Como podemos ver, hemos iniciado sesión con mi certificado digital.

---

## HTTPS / SSL.

### Tarea 1: Certificado autofirmado. CA

#### Crear su autoridad certificadora (generar el certificado digital de la CA). Mostrar el fichero de configuración de la AC.

Lo primero que haré, será crear carpetas para mantener una organización con los archivos.

```txt
alemd@debian:~/2ASIR/SAD/cripto3/CA$ mkdir certs csr crl newcerts private
alemd@debian:~/2ASIR/SAD/cripto3/CA$ chmod 700 private
alemd@debian:~/2ASIR/SAD/cripto3/CA$ touch index.txt
alemd@debian:~/2ASIR/SAD/cripto3/CA$ touch index.txt.attr
alemd@debian:~/2ASIR/SAD/cripto3/CA$ echo 1000 > serial
```

Añado las siguientes variables de entorno:

```txt
alemd@debian:~/2ASIR/SAD/cripto3/CA$ countryName_default="ES"
alemd@debian:~/2ASIR/SAD/cripto3/CA$ stateOrProvinceName_default="Sevilla"
alemd@debian:~/2ASIR/SAD/cripto3/CA$ localityName_default="Utrera"
alemd@debian:~/2ASIR/SAD/cripto3/CA$ organizationName_default="Alejandro"
alemd@debian:~/2ASIR/SAD/cripto3/CA$ organizationalUnitName_default="ASIR2"
alemd@debian:~/2ASIR/SAD/cripto3/CA$ emailAddress_default="aaleemd11@gmail.com"
alemd@debian:~/2ASIR/SAD/cripto3/CA$ DIR_CA="./"
```

A continuación, creo el archivo openssl.cnf

```txt
 cat <<EOF>$DIR_CA/openssl.conf
[ ca ]
# man ca
default_ca = CA_default

[ CA_default ]
# Directory and file locations.
dir               = ${DIR_CA}
certs             = ${DIR_CA}certs
crl_dir           = ${DIR_CA}crl
new_certs_dir     = ${DIR_CA}newcerts
database          = ${DIR_CA}index.txt
serial            = ${DIR_CA}serial
RANDFILE          = ${DIR_CA}private/.rand

# The root key and root certificate.
private_key       = ${DIR_CA}private/private.key
certificate       = ${DIR_CA}certs/cacert.crt

# For certificate revocation lists.
crlnumber         = ${DIR_CA}crlnumber
crl               = ${DIR_CA}crl/ca.crl.pem
crl_extensions    = crl_ext
default_crl_days  = 30

# SHA-1 is deprecated, so use SHA-2 instead.
default_md        = sha256

name_opt          = ca_default
cert_opt          = ca_default
default_days      = 375
preserve          = no
policy            = policy_strict

[ policy_strict ]
# The root CA should only sign intermediate certificates that match.
EOFendedKeyUsage = critical, OCSPSignings (man ocsp).tg).yEnciphermentes.
```

Generamos la clave privada y le cambiamos los permisos a la misma:

```txt
alemd@debian:~/2ASIR/SAD/cripto3/CA$ openssl genrsa -aes256 -out private/private.key 4096
Generating RSA private key, 4096 bit long modulus (2 primes)
...........................................................++++
.......................................................................................................................................................................................................................................................++++
e is 65537 (0x010001)
Enter pass phrase for private/private.key:
Verifying - Enter pass phrase for private/private.key:
alemd@debian:~/2ASIR/SAD/cripto3/CA$ chmod 400 private/private.key
```

Con el siguiente comando, editamos el fichero openssl.cnf:

```txt
alemd@debian:~/2ASIR/SAD/cripto3/CA$ sed -i 's|xxxsubjectAltNamexxx =|subjectAltName = ${ENV::SAN}|g' openssl.conf
```

Y ahora, creamos el certificado autofirmado para la CA:

```txt
alemd@debian:~/2ASIR/SAD/cripto3/CA$ URL=juanjesus.iesgn.org
alemd@debian:~/2ASIR/SAD/cripto3/CA$ export SAN=DNS:$URL
alemd@debian:~/2ASIR/SAD/cripto3/CA$ openssl req -config openssl.conf -key private/private.key -new -x509 -days 3650 -sha256 -extensions v3_ca -out certs/cacert.crt
Enter pass phrase for private/private.key:
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [ES]:
State or Province Name [Sevilla]:
Locality Name [Utrera]:
Organization Name [Alejandro]:
Organizational Unit Name [ASIR2]:
Common Name []:alejandro.montes
Email Address [aaleemd11@gmail.com]:
alemd@debian:~/2ASIR/SAD/cripto3/CA$ chmod 444 certs/cacert.crt
```

#### Debe recibir el fichero CSR (Solicitud de Firmar un Certificado) de su compañero, debe firmarlo y enviar el certificado generado a su compañero.

Una vez que mi compañero Juanje me ha enviado el fichero, lo firmo:

```txt
alemd@debian:~/2ASIR/SAD/cripto3/CA$ openssl ca -config openssl.conf -extensions v3_req -days 3650 -notext -md sha256 -in csr/juanjesus-aleca.csr -out certs/juanjesus-aleca.crt
Using configuration from openssl.conf
Enter pass phrase for ./private/private.key:
Check that the request matches the signature
Signature ok
Certificate Details:
        Serial Number: 4096 (0x1000)
        Validity
            Not Before: Jan  8 12:21:19 2023 GMT
            Not After : Jan  5 12:21:19 2033 GMT
        Subject:
            countryName               = ES
            stateOrProvinceName       = Sevilla
            organizationName          = Alejandro
            organizationalUnitName    = ASIR2
            commonName                = juanjesus.debian
            emailAddress              = githubemail1asir@gmail.com
        X509v3 extensions:
            X509v3 Basic Constraints: 
                CA:FALSE
            X509v3 Key Usage: 
                Digital Signature, Non Repudiation, Key Encipherment
            X509v3 Subject Alternative Name: 
                DNS:juanjesus.iesgn.org
Certificate is to be certified until Jan  5 12:21:19 2033 GMT (3650 days)
Sign the certificate? [y/n]:y


1 out of 1 certificate requests certified, commit? [y/n]y
Write out database with 1 new entries
Data Base Updated
```

Ahora le envío a Juanje los ficheros `juanjesus-aleca.crt` y el fichero `cacert.crt`

#### ¿Qué otra información debes aportar a tu compañero para que éste configure de forma adecuada su servidor web con el certificado generado?

Mi compañero necesitara el fichero `cacert.crt` que es el certificado autofirmado de la CA para que se pueda verificar la firma del certificado que le he enviado.

### Tarea 1: Certificado autofirmado. Administrador Web

Antes que nada, dejo un resumen de los comandos usados para crear el servidor web con el virtualhost `alejandro.iesgn.org`.

```txt
usuario@alejandro:~$ sudo apt update
usuario@alejandro:~$ sudo apt install apache2
usuario@alejandro:~$ sudo a2dissite 000-default.conf
usuario@alejandro:~$ sudo nano -cl /etc/apache2/sites-available/alejandro.iesgn.org.conf
```

Añadimos lo siguiente a la configuración del virtualhost:

```txt
<VirtualHost *:80>
    ServerName alejandro.iesgn.org
    DocumentRoot /var/www/html/alejandro.iesgn.org
    ErrorLog ${APACHE_LOG_DIR}/error-alejandro.log
    CustomLog ${APACHE_LOG_DIR}/access-alejandro.log combined
</VirtualHost>
```

He puesto un fichero de prueba para visualizar la página web. Activamos el virtual host:

```txt
usuario@alejandro:~$ sudo a2ensite alejandro.iesgn.org.conf
Enabling site alejandro.iesgn.org.
To activate the new configuration, you need to run:
  systemctl reload apache2
usuario@alejandro:~$ sudo systemctl reload apache2
```

Y por último en mi máquina cliente añadimos al /etc/hosts lo siguiente:

```txt
172.22.9.231 alejandro.iesgn.org
```

#### Crea una clave privada RSA de 4096 bits para identificar el servidor.

Usamos el siguiente comando para crear la clave y le cambiamos los permisos al fichero:

```txt
usuario@alejandro:~$ sudo openssl genrsa -aes256 -out /etc/ssl/private/alejandro-priv.key 4096
Generating RSA private key, 4096 bit long modulus (2 primes)
........................++++
...............................................................................................................................................................++++
e is 65537 (0x010001)
Enter pass phrase for /etc/ssl/private/alejandro-priv.key:
Verifying - Enter pass phrase for /etc/ssl/private/alejandro-priv.key:
usuario@alejandro:~$ sudo chmod 400 /etc/ssl/private/alejandro-priv.key
```

#### Utiliza la clave anterior para generar un CSR, considerando que deseas acceder al servidor con el FQDN (tunombre.iesgn.org). 

El comando es el siguiente:

```txt
usuario@alejandro:~$ sudo openssl req -new -sha256 -key /etc/ssl/private/alejandro-priv.key -out alejandro.csr
Enter pass phrase for /etc/ssl/private/alejandro-priv.key:
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:ES
State or Province Name (full name) [Some-State]:Sevilla
Locality Name (eg, city) []:Dos Hermanas
Organization Name (eg, company) [Internet Widgits Pty Ltd]:Juanje
Organizational Unit Name (eg, section) []:ASIR2
Common Name (e.g. server FQDN or YOUR name) []:alejandro.montes
Email Address []:aaleemd11@gmail.com

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:       
An optional company name []:
```

#### Envía la solicitud de firma a la entidad certificadora (su compañero).

El fichero CSR ha sido enviado a mi compañero Juanje.

#### Recibe como respuesta un certificado X.509 para el servidor firmado y el certificado de la autoridad certificadora.

Juanje me envía los ficheros `alejandro.crt` y `cacert.crt`.

#### Configura tu servidor web con https en el puerto 443, haciendo que las peticiones http se redireccionen a https (forzar https).

Creo el virtualhost para https:

```txt
usuario@alejandro:~$ sudo nano -cl /etc/apache2/sites-available/ssl-alejandro.iesgn.org.conf
```

Añado lo siguiente:

```txt
 <IfModule mod_ssl.c>
     <VirtualHost *:443>
         ServerName alejandro.iesgn.org
         DocumentRoot /var/www/html/alejandro.iesgn.org
         ErrorLog ${APACHE_LOG_DIR}/error-alejandro.log
         CustomLog ${APACHE_LOG_DIR}/access-alejandro.log combined

         SSLEngine on
         SSLCertificateFile /etc/ssl/certs/alejandro.crt
         SSLCertificateKeyFile /etc/ssl/private/alejandro-priv.key
         SSLCertificateChainFile /etc/ssl/certs/cacert.crt

         <Directory /var/www/html/alejandro.iesgn.org>
             Options Indexes FollowSymLinks
             AllowOverride None
             Require all granted
         </Directory>
     </VirtualHost>
 </IfModule>
```

A continuación, muevo los ficheros a sus directorios correspondientes:

```txt
usuario@alejandro:~$ sudo mv alejandro.crt /etc/ssl/certs/
usuario@alejandro:~$ sudo mv cacert.crt /etc/ssl/certs/
usuario@alejandro:~$ sudo chown root:root /etc/ssl/certs/alejandro.crt
usuario@alejandro:~$ sudo chown root:root /etc/ssl/certs/cacert.crt
usuario@alejandro:~$ sudo chmod 644 /etc/ssl/certs/alejandro.crt
usuario@alejandro:~$ sudo chmod 644 /etc/ssl/certs/cacert.crt
```

por último, activamos el módulo SSL de apache, el virtualhost y recargamos el servicio:

```txt
usuario@alejandro:~$ sudo a2enmod ssl
Considering dependency setenvif for ssl:
Module setenvif already enabled
Considering dependency mime for ssl:
Module mime already enabled
Considering dependency socache_shmcb for ssl:
Enabling module socache_shmcb.
Enabling module ssl.
See /usr/share/doc/apache2/README.Debian.gz on how to configure SSL and create self-signed certificates.
To activate the new configuration, you need to run:
  systemctl restart apache2
usuario@alejandro:~$ sudo a2ensite ssl-alejandro.iesgn.org.conf
Enabling site ssl-alejandro.iesgn.org.
To activate the new configuration, you need to run:
  systemctl reload apache2
usuario@alejandro:~$ sudo systemctl reload apache2
🔐 Enter passphrase for SSL/TLS keys for alejandro.iesgn.org:443 (RSA): (p*********             
```

Para que nos redirija a HTTPS, tenemos que hacer lo siguiente:

```txt
alemd@debian:~$ sudo nano -cl /etc/apache2/sites-available/ssl-alejandro.iesgn.org.conf
```

Tenemos que dejar el fichero de la siguiente manera:

```txt
<VirtualHost *:80>
     ServerName alejandro.iesgn.org
     DocumentRoot /var/www/html/alejandro.iesgn.org
     ErrorLog ${APACHE_LOG_DIR}/error-alejandro.log
     CustomLog ${APACHE_LOG_DIR}/access-alejandro.log combined
     Redirect 301 / https://alejandro.iesgn.org/
</VirtualHost>
```

Volvemos a recargar apache para que se apliquen los cambios.

```txt
usuario@alejandro:~$ sudo systemctl reload apache2
```



Para que al entrar no nos aparezca el mensaje de que el certificado no es reconocido por mi navegador tendremos que importar en el navegador el certificado autofirmado de la CA  de Juanje que es el archivo `cacert.crt`.

![SAD-cripto3.t6.2.png](/img/SAD-cripto3.t6.2.png)


#### Instala ahora un servidor nginx, y realiza la misma configuración que anteriormente para que se sirva la página con HTTPS.

Lo primero que haremos será deshabilitar apache2:

```txt
usuario@alejandro:~$ sudo systemctl disable --now apache2
Synchronizing state of apache2.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install disable apache2
Removed /etc/systemd/system/multi-user.target.wants/apache2.service.
```

Instalamos nginx:

```txt
usuario@alejandro:~$ sudo apt install nginx
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  fontconfig-config fonts-dejavu-core geoip-database libdeflate0
  libfontconfig1 libgd3 libgeoip1 libjbig0 libjpeg62-turbo
  libnginx-mod-http-geoip libnginx-mod-http-image-filter
  libnginx-mod-http-xslt-filter libnginx-mod-mail libnginx-mod-stream
  libnginx-mod-stream-geoip libtiff5 libwebp6 libx11-6 libx11-data
  libxau6 libxcb1 libxdmcp6 libxpm4 libxslt1.1 nginx-common nginx-core
Suggested packages:
  libgd-tools geoip-bin fcgiwrap nginx-doc
The following NEW packages will be installed:
  fontconfig-config fonts-dejavu-core geoip-database libdeflate0
  libfontconfig1 libgd3 libgeoip1 libjbig0 libjpeg62-turbo
  libnginx-mod-http-geoip libnginx-mod-http-image-filter
  libnginx-mod-http-xslt-filter libnginx-mod-mail libnginx-mod-stream
  libnginx-mod-stream-geoip libtiff5 libwebp6 libx11-6 libx11-data
  libxau6 libxcb1 libxdmcp6 libxpm4 libxslt1.1 nginx nginx-common
  nginx-core
0 upgraded, 27 newly installed, 0 to remove and 0 not upgraded.
Need to get 8715 kB of archives.
After this operation, 24.2 MB of additional disk space will be used.
Do you want to continue? [Y/n] y
```

Creamos el fichero de configuración del virtualhost de la web:

```txt
usuario@alejandro:~$ sudo nano -cl /etc/nginx/sites-available/alejandro.iesgn.org.conf
```

agregamos lo siguiente:

```txt
server {
    listen 80;
    listen [::]:80;
    server_name alejandro.iesgn.org;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name alejandro.iesgn.org;

    ssl_certificate /etc/ssl/certs/alejandro.crt;
    ssl_certificate_key /etc/ssl/private/alejandro-priv.key;
    ssl_trusted_certificate /etc/ssl/certs/cacert.crt;

    root /var/www/html/alejandro.iesgn.org;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Ahora creamos el enlace simbólico del virtualhost:

```txt
 sudo ln -s /etc/nginx/sites-available/alejandro.iesgn.org.conf /etc/nginx/sites-enabled/
```

Modifico la página y reinicio el servicio:

```txt
usuario@alejandro:~$ sudo systemctl restart nginx
usuario@alejandro:~$ sudo nano -cl /var/www/html/alejandro.iesgn.org/index.html
```

Accedo a la web:

![SAD-cripto3.t6.1.png](/img/SAD-cripto3.t6.1.png)



---
## **Documento realizado por:**

 ✒️ **Alejandro Montes Delgado** - *2º ASIR*
