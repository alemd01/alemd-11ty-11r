---
layout: post
title: Talleres unidad 1 Implantación Aplicaciones Web.
excerpt: Talleres realizados en Implantación de Aplicaciones WEB
date: 2022-10-05
updatedDate: 2022-10-05
tags:
  - post
  - IAW
---
##TALLER 1: Introducción a git y GitHub
1. Una captura de pantalla donde se vea que has creado el repositorio.

![TALLER1-IAW.png](/img/TALLER1-IAW.png)

2. El contenido del fichero .git/config para que se vea que has clonado el repositorio con la URL ssh.
```diff-js
[core]
	repositoryformatversion = 0
	filemode = true
	bare = false
	logallrefupdates = true
[remote "origin"]
	url = git@github.com:alemd01/2ASIR.git
	fetch = +refs/heads/*:refs/remotes/origin/*
[branch "main"]
	remote = origin
	merge = refs/heads/main
```
3. La salida de la instrucción git log para ver los commits que has realizado (debe aparecer como autor tu nombre completo).
```diff-js
commit 421dec9a6988d91a1ac7c871ca9e4b042e803df7
Author: alemd01 <aaleemd11@gmail.com>
Date:   Mon Sep 19 10:28:31 2022 +0200
  He borrado el fichero prueba.txt

commit d584d99ece167dedc0b8b649471c11a453652d2b
Author: alemd01 <aaleemd11@gmail.com>
Date:   Mon Sep 19 10:27:57 2022 +0200
  He cambiado el nombre al fichero

commit 2793b586ee7a7d89184f7313ec4d92ced9f2c64a
Author: alemd01 <aaleemd11@gmail.com>
Date:   Mon Sep 19 10:26:59 2022 +0200
  He modificado el fichero prueba.txt

commit e6195890e08484aa86eb1fc9c7c93d8448328c33
Author: alemd01 <aaleemd11@gmail.com>
Date:   Mon Sep 19 10:17:13 2022 +0200
  He creado el fichero prueba.txt

commit 2093f3e2515054c1d7223d79b373c89526beebc4
Author: alemd01 <78356101+alemd01@users.noreply.github.com>
Date:   Mon Sep 19 10:15:13 2022 +0200
  Initial commit
```
4. buscar información para crear un nuevo repositorio llamado prueba2_tu_nombre. en esta ocasión, crea primero el repositorio local (usando git init) y luego busca información para sincronizarlo y crear el repositorio remoto en GitHub. Comenta los pasos que has realizado y manda alguna prueba de funcionamiento.
Primero hay que crear un repositorio en Github, después hay que crear un directorio en nuestra máquina local. Creamos por ejemplo el archivo README. Entramos en la carpeta y hacemos un git init.

```diff-js
     mkdir prueba2_AlejandroMontes
 
     cd prueba2_AlejandroMontes/

     echo "# prueba2_AlejandroMontes" >> README.md
```
![taller1-2.png](/img/taller1-2.png)

Una vez hecho esto, hacemos un git add y un commit.
```diff-js
     git add README.md

     git commit -m "primer commit"
```
![taller1-3.png](/img/taller1-3.png)

Ahora usamos este comando 
```diff-js
     git branch -M main
```

A continuación, tenemos que hacer un git remote add origin
```diff-js
     git remote add origin git@github.com:alemd01/prueba2_AlejandroMontes.git

```
por último hacemos un git push
```diff-js
      git push -u origin main
```

![taller1-4.png](/img/taller1-4.png)


