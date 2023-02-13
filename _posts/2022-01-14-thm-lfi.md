---
layout: single
title: Local File Inclusion - Try Hack Me
excerpt: Local File Inclusion maquina catalogada de dificultad “Easy” nos centraremos en la vulnerabilidad LFI además de escalación de escalación de privilegios.
date: 2022-01-14
classes: wide
header:
  teaser: /assets/images/thm-lfi/1.jpg
  teaser_home_page: true
  icon: /assets/images/thm.webp
categories:
  - TryHackMe
tags:  
  - web
  - file inclusion
  - lfi
  - sudo
  - Easy
  
---

![](/assets/images/thm-lfi/1.jpg)

Local File Inclusion maquina catalogada de dificultad “Easy” nos centraremos en la vulnerabilidad LFI además de escalación de escalación de privilegios.

## 1-Escaneo Nmap y enumeración de puertos.

realizamos un escaneo con Nmap para descubrir todos los puertos abiertos: nmap -p- --open -T5 -v -n 10.10.47.79 -oN allports.txt

Utilizaremos los parametros -p- (Indicar todos los puertos) --open (Filtrar por puertos abiertos) -T5 (Temporizado del escaneo, para más velocidad) -v (Arrojar un poco de inforación mientras se realiza el escaneo) -n (No aplicar resolución DNS) -oN (Guardar archivo)

Nmap nos devuelve todo esto:

```
nmap -p- --open -T5 -v -n 10.10.47.79 -oN allports.txt
Starting Nmap 7.92 ( https://nmap.org ) at 2022-01-13 23:33 CET
Initiating Ping Scan at 23:33
Scanning 10.10.47.79 [2 ports]
Completed Ping Scan at 23:33, 0.04s elapsed (1 total hosts)
Initiating Connect Scan at 23:33
Scanning 10.10.47.79 [65535 ports]
Discovered open port 80/tcp on 10.10.47.79
Discovered open port 22/tcp on 10.10.47.79
Completed Connect Scan at 23:33, 18.65s elapsed (65535 total ports)
Nmap scan report for 10.10.47.79
Host is up (0.046s latency).
Not shown: 53799 closed tcp ports (conn-refused), 11734 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 18.83 seconds
```

Vemos que nos devuelve bastantes puertos abiertos. Algunos insteresantes son el puerto 80 (http) , 22 (ssh).

A continuación vamos a realizar un escaneo más profundo a estos puertos con el siguiente comando:

nmap -sC -sV 10.10.47.79 -oN ports.txt

Parametros -sC (Utilizar scripts comunes) y -sV (Mostrar versiones y servicios) -oN (Guardar el archivo)


Nmap nos devulve esto:

```
nmap -sC -sV 10.10.47.79 -oN ports.txt
Starting Nmap 7.92 ( https://nmap.org ) at 2022-01-13 23:45 CET
Nmap scan report for 10.10.47.79 (10.10.47.79)
Host is up (0.043s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 e6:3a:2e:37:2b:35:fb:47:ca:90:30:d2:14:1c:6c:50 (RSA)
|   256 73:1d:17:93:80:31:4f:8a:d5:71:cb:ba:70:63:38:04 (ECDSA)
|_  256 d3:52:31:e8:78:1b:a6:84:db:9b:23:86:f0:1f:31:2a (ED25519)
80/tcp open  http    Werkzeug httpd 0.16.0 (Python 3.6.9)
|_http-title: My blog
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.49 seconds
```

Utilizaremos también whatweb para enumerar rapidamente el servicio http y ver ante que nos estamos enfrentando.

Nos duelve esto:

```
whatweb 10.10.47.79
http://10.10.47.79 [200 OK] Bootstrap, Country[RESERVED][ZZ], HTML5, HTTPServer[Werkzeug/0.16.0 Python/3.6.9], IP[10.10.47.79], Python[3.6.9], Title[My blog], Werkzeug[0.16.0]
```

Vemos que nos arroja poca información, página escrita en python con el titulo My blog, entre otras cosas...

## 2-Local File Inclusion

Primero vamos a explicar en que consta esta vulnerabilidad para enterderla.

Es cuando un atacante solicita un recurso al cual no puede acceder a una pagina web mal configurda Este archivo puede contener bases de datos, contenido sensibles… Este ataque se debe principalmente debido a que la entrada de datos no esta validada.

La idea es esta pero, ¿en un caso practico como sería? , vamos a la pagina web y vamos a explicarlo detalladamente.

![](/assets/images/thm-lfi/2.jpg)

Este es el aspecto de la página web, pero fijemonos en la URL.

http://10.10.47.79/ -> Es una URL normal solicitando un archivo, perfecto vamos a movernos por la pagina.

![](/assets/images/thm-lfi/3.jpg)

http://10.10.47.79/article?name=hacking

Al pulsar en View Details del primer articulo, vemos como la URL cambia y solicita un articulo llamado "hacking" , ¿como explotaremos esto? , sabiendo que el sistema utiliza la variable "name" para encontrar un archivo que pasaría si provamos a hacerlo hacia atrás (path traversal).
Es decir nosotros nos encontramos normalmente por defecto en (No tiene que ser este caso es un ejemplo para enterdelo): /var/www/www-data 

hay es donde esta montado el sitio web , por lo cual nosotros podremos acceder a archivos a partir de ese directorio por ejemplo: /home , que nos muestra la pagina inicial , para nosotros solo solicitamos: /home , pero el sistema internamente realiza esta acción: 
/var/www/www-data/home

Si intentamos realizar un path traversal, podríamos ver lo que esta en todo el sistema, por ejemplo: ../../../../../../etc/passwd

Entonces el sistema retrocederia hasta que estuviera en su raíz: / y nos devolveria el archivo solicitado.

![](/assets/images/thm-lfi/4.jpg)

Vemos que en este caso funciona sin ningun problema, podriamos ejecutar cualquier comando en el sistema o ver cualquier archivo, pero a nosotros lo que principalmente nos interesa es ganar acceso al sistema.

En la salida de /etc/passwd podemos encontramos un usuario llamado "falconfeast" que tiene permisos de usuario y grupo 1000 (root) , con sus directorio inicial /home/falconfeast , vamos a probar a encontrar la flag directamente pidiendo el archivo en /home/falconfeast/user.txt

![](/assets/images/thm-lfi/5.jpg)

Vamos a probar lo mismo con el usuario root, /root/root.txt

![](/assets/images/thm-lfi/6.jpg)

Y vemos que tenemos la flag del usuario root.

## 3-Ganando acceso a la maquina victima. 

Pero... ¿Muy fácil todo no?, para no quedarnos con mal sabor de boca vamos a ganar acceso a la maquina, recordais cuando vimos el contenido de /etc/passwd un comentario , #rootpassword , vamos a utilizar el servicio SHH para conectarnos con el usuario de falconfeast y la contraseña rootpasword.


![](/assets/images/thm-lfi/7.jpeg)

Bien, vemos que ganamos acceso, asi que vamos a buscar de nuevo las flags.

![](/assets/images/thm-lfi/8.jpg)

A continuacion vamos a escalar privilegios para encontrar la flag de root.

Ejecutaremos el comando sudo -l, para ver que comandos puede ejecutar nuestro usuario como root.


![](/assets/images/thm-lfi/9.jpg)

Vemos que podemos ejecutar socat como usuario root, asi que ahora vamos a GTFObins para ver que podemos hacer con esto, nos encontramos con este comando que nos da acceso a una terminal como root -> sudo socat stdin exec:/bin/sh

Vamos a ejecutarlo y a buscar la flag.

![](/assets/images/thm-lfi/10.jpg)

Ya tendriamos la flag del usuario root.


![](/assets/images/thm-lfi/ultima.jpg)
