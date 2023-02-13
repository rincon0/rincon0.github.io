---
layout: single
title: RootMe - Try Hack Me
excerpt: RootMe maquina catalogada de dificultad “Easy” basada en la escalacion de privilegios en sistemas linux aprovechandonos de una vulnerabilidad web en la subida de archivos.
date: 2021-12-31
classes: wide
header:
  teaser: /assets/images/thm-rootme/1.JPG
  teaser_home_page: true
  icon: /assets/images/thm.webp
categories:
  - TryHackMe
tags:  
  - linux
  - security
  - web
  - privilege-escalation
  - Easy
  
---

![](/assets/images/thm-rootme/1.JPG)

RootMe maquina catalogada de dificultad “Easy” basada en la escalacion de privilegios en sistemas linux aprovechandonos de una vulnerabilidad web en la subida de archivos, nos encontraremos con un filtro en el lado del servidor y aprenderemos a realizar un bypass del mismo.

## 1-Escaneo Nmap y busqueda de directorios.

realizamos un escaneo con Nmap básico: nmap -sC -sV -p80,22 10.10.182.50 -oN PortsNmap.txt

Utilizaremos los parametros -sC (Utilizar los scripts más comunes) -sV (Mostrar versiones) -oN (Guardar archivo)

Nmap nos devuelve todo esto:

```
nmap -sC -sV 10.10.182.50 -oN PortsNmap.txt
Starting Nmap 7.92 ( https://nmap.org ) at 2021-12-31 02:26 CET
Nmap scan report for 10.10.182.50 (10.10.182.50)
Host is up (0.040s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 4a:b9:16:08:84:c2:54:48:ba:5c:fd:3f:22:5f:22:14 (RSA)
|   256 a9:a6:86:e8:ec:96:c3:f0:03:cd:16:d5:49:73:d0:82 (ECDSA)
|_  256 22:f6:b5:a6:54:d9:78:7c:26:03:5a:95:f3:f9:df:cd (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: HackIT - Home
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.37 seconds
```

Vemos que nos devuelve bastantes puertos abiertos. Algunos insteresantes son el puerto 80 (http) , 22 (ssh).

En el puerto 80 , vemos que tenemos una pagina aparentemente sin mucha información.

![](/assets/images/thm-rootme/2.JPG)

Vamos a realizar una busqueda de directorios para ver si la página tiene algo interesante.

Utilizaremos la herramienta gobuster.

![](/assets/images/thm-rootme/3.JPG)

Vemos que en el directorio /panel tenemos un formulario para subir archivos.

![](/assets/images/thm-rootme/4.JPG)

Vemos que con Wappalyzer la web utiliza el motor PHP, vamos a probar a subir un archivo .php malicioso para obtener una revershell, podriamos utilizar el de pentestmonkey

Lo descargamos y vamos a modificar la ip del script, recomiendo dejar el puerto por defecto.

![](/assets/images/thm-rootme/5.JPG)

Vamos a subirlo al servidor.

![](/assets/images/thm-rootme/6.JPG)

Vemos que nos da un error al subirlo, indicandonos que los archivos .php no estan permitidos, para realizar un bypass hay muchas tecnicas, pero hay que ir probando de más fácil a más complejo, la tecnica mas simple es utilizar una extensión php similar, por ejemplo php5.

![](/assets/images/thm-rootme/7.JPG)

Vemos que utilizando la extensión php5 si nos deja, nos iremos al directorio /uploads que descubrimos anteriormente y le pinchamos en el archivo para llamar a la reverse shell. Debemos tener una interfaz de netcat en el puerto indicado anteriormente.

![](/assets/images/thm-rootme/8.JPG)

Vamos a buscar la flag del usuario.

![](/assets/images/thm-rootme/9.JPG)

## 2-Escalación de privilegios.

Ejecutando el comando sudo -l , nos indica que binarios podemos ejecutar como superadministrador, vemos que en este caso ninguno.

Entonces vamos a buscar algún programa con el que nos podamos aprovechar de SUID.

Ejecutando el comando -> find / -perm -u=s -type f 2>/dev/null , encontraremos todos los binarios que podamos aprovecharnos de esta vulnerabilidad.

```
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/snapd/snap-confine
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
/usr/lib/eject/dmcrypt-get-device
/usr/lib/openssh/ssh-keysign
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/bin/traceroute6.iputils
/usr/bin/newuidmap
/usr/bin/newgidmap
/usr/bin/chsh
/usr/bin/python
/usr/bin/at
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/sudo
/usr/bin/newgrp
/usr/bin/passwd
/usr/bin/pkexec
/snap/core/8268/bin/mount
/snap/core/8268/bin/ping
/snap/core/8268/bin/ping6
/snap/core/8268/bin/su
/snap/core/8268/bin/umount
/snap/core/8268/usr/bin/chfn
/snap/core/8268/usr/bin/chsh
/snap/core/8268/usr/bin/gpasswd
/snap/core/8268/usr/bin/newgrp
/snap/core/8268/usr/bin/passwd
/snap/core/8268/usr/bin/sudo
/snap/core/8268/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/snap/core/8268/usr/lib/openssh/ssh-keysign
/snap/core/8268/usr/lib/snapd/snap-confine
/snap/core/8268/usr/sbin/pppd
/snap/core/9665/bin/mount
/snap/core/9665/bin/ping
/snap/core/9665/bin/ping6
/snap/core/9665/bin/su
/snap/core/9665/bin/umount
/snap/core/9665/usr/bin/chfn
/snap/core/9665/usr/bin/chsh
/snap/core/9665/usr/bin/gpasswd
/snap/core/9665/usr/bin/newgrp
/snap/core/9665/usr/bin/passwd
/snap/core/9665/usr/bin/sudo
/snap/core/9665/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/snap/core/9665/usr/lib/openssh/ssh-keysign
/snap/core/9665/usr/lib/snapd/snap-confine
/snap/core/9665/usr/sbin/pppd
/bin/mount
/bin/su
/bin/fusermount
/bin/ping
/bin/umount
```

Tenemos todos estos, en la página de GTFOBins encontraremos todos los programas que podamos explotar.

En este caso el binario /usr/bin/python , no esta defenido por defecto y si lo filtramos por GTFOBins vemos que podemos aprovecharnos de esto.

![](/assets/images/thm-rootme/10.JPG)

![](/assets/images/thm-rootme/11.JPG)