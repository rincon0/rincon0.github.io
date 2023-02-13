---
layout: single
title: Mustacchio - Try Hack Me
excerpt: Mustacchio maquina catalogada de dificultad “Easy” en esta ocasion nos enfrentaremos a un trabajo de enumeración, además ganaremos acceso a la maquina mediande la vulnerabilidad xxe y escalaremos privilegios.
date: 2022-02-25
classes: wide
header:
  teaser: /assets/images/thm-mustacchio/1.jpg
  teaser_home_page: true
  icon: /assets/images/thm.webp
categories:
  - TryHackMe
tags:  
  - xxe
  - enumeration
  - privesc
  - web
  - Easy
  
---

![](/assets/images/thm-mustacchio/1.jpg)

Mustacchio maquina catalogada de dificultad “Easy” en esta ocasion nos enfrentaremos a un trabajo de enumeración, además ganaremos acceso a la maquina mediande la vulnerabilidad xxe y escalaremos privilegios.

## 1-Escaneo Nmap y enumeración de puertos.

realizamos un escaneo con Nmap para descubrir todos los puertos abiertos: sudo nmap -p- -sS --min-rate 5000 --open -vvv -n 10.10.125.119 -oG allports

Utilizaremos los parametros -p- (Indicar todos los puertos) -sS (Utilizan un TCP Syn port scan) --min-rate 5000 (Indicar paquetes no mas lentos que 5000 paquetes por segundo) --open (Mostrar aquellos puertos que tengan un estatus "open") -vvv (Arrojar un poco de inforación mientras se realiza el escaneo) -n (No aplicar resolución DNS) -oN (Guardar archivo en formato grepeable)

Nmap nos devuelve todo esto:

```
sudo nmap -p- -sS --min-rate 5000 --open -vvv -n 10.10.125.119 -oG allports
[sudo] password for rincon: 
Starting Nmap 7.92 ( https://nmap.org ) at 2022-02-23 19:24 CET
Initiating Ping Scan at 19:24
Scanning 10.10.125.119 [4 ports]
Completed Ping Scan at 19:24, 0.07s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 19:24
Scanning 10.10.125.119 [65535 ports]
Discovered open port 22/tcp on 10.10.125.119
Discovered open port 80/tcp on 10.10.125.119
Discovered open port 8765/tcp on 10.10.125.119
Completed SYN Stealth Scan at 19:25, 26.48s elapsed (65535 total ports)
Nmap scan report for 10.10.125.119
Host is up, received echo-reply ttl 63 (0.055s latency).
Scanned at 2022-02-23 19:24:43 CET for 26s
Not shown: 65532 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE        REASON
22/tcp   open  ssh            syn-ack ttl 63
80/tcp   open  http           syn-ack ttl 63
8765/tcp open  ultraseek-http syn-ack ttl 63

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 26.74 seconds
           Raw packets sent: 131090 (5.768MB) | Rcvd: 23 (996B)
```

Vemos que nos devuelve bastantes puertos abiertos. Algunos insteresantes son el puerto 80 y 8765 (http) , 22 (ssh).

A continuación vamos a realizar un escaneo más profundo a estos puertos con el siguiente comando:

nmap -sC -sV -p22,80,8765 10.10.125.119 -oN ports

Parametros -sC (Utilizar scripts comunes) y -sV (Mostrar versiones y servicios) -p (Indicar puertos en especifico) -oN (Guardar el archivo)


Nmap nos devulve esto:

```
nmap -sC -sV -p22,80,8765 10.10.125.119 -oN ports
Starting Nmap 7.92 ( https://nmap.org ) at 2022-02-23 19:33 CET
Nmap scan report for 10.10.125.119 (10.10.125.119)
Host is up (0.039s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 58:1b:0c:0f:fa:cf:05:be:4c:c0:7a:f1:f1:88:61:1c (RSA)
|   256 3c:fc:e8:a3:7e:03:9a:30:2c:77:e0:0a:1c:e4:52:e6 (ECDSA)
|_  256 9d:59:c6:c7:79:c5:54:c4:1d:aa:e4:d1:84:71:01:92 (ED25519)
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-robots.txt: 1 disallowed entry 
|_/
|_http-title: Mustacchio | Home
|_http-server-header: Apache/2.4.18 (Ubuntu)
8765/tcp open  http    nginx 1.10.3 (Ubuntu)
|_http-title: Mustacchio | Login
|_http-server-header: nginx/1.10.3 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.82 seconds

```

Utilizaremos también whatweb para enumerar rapidamente el servicio http y ver ante que nos estamos enfrentando.

Nos duelve esto:

![](/assets/images/thm-mustacchio/2.jpg)


Vemos que nos arroja algo de información. En el puerto 80 tenemos una página llamada MUstacchio, utiliza un servidor web Apache...
Lo interesante parece estar en el puerto 8765: Tenemos una web llamada Mustacchio | Login y utiliza el servidor web Nginx. 

## 2-Enumeración

Vamos a enumerar la pagina web principal del puerto 80 y luego iremos a la otra.

![](/assets/images/thm-mustacchio/3.jpg)

Vemos que es una pagina normal, con sus categorias, html... Vamos a realizar fuzzing.

Vamos a utilizar la herramienta gobuster para buscar contenido en la web -> gobuster dir -u http://10.10.125.119/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .html.php.txt

El parametro dir -u (Lo utilizaremos para indicar la URL) , el parametro -w (Para indicar una wordlist o diccionario) y el parametro -x (Es para que nos filtre por extensiones)

![](/assets/images/thm-mustacchio/4.jpg)

Vamos a mirar dentro del directorio custom, vemos que tenemos una carpeta de JS y CSS , vamos a mirar dentro de la carpeta de JS y vemos que tenemos un archivo llamado users.bak , vamos a descargarlo y ver lo que contiene.

![](/assets/images/thm-mustacchio/5.png)

Vamos a ver que tipo de archivo es con el comando "file" y vemos que es un archivo SQL, una forma de ver el contenido de este archivo es utilizar el comando "strings".

![](/assets/images/thm-mustacchio/6.jpg)

Tenemos al parecer unas posibles credenciales .> admin1868e36a6d2b17d4c2745f1659433a54d4bc5f4b

User -> admin
Pass -> 1868e36a6d2b17d4c2745f1659433a54d4bc5f4b

Parece ser que la contraseña esta encriptada o esta en algun formato de hash.
La manera mas rapida de encontrar descifrar este tipo de hashes es irnos a alguna pagina web para que nos compare la cadena, haber si previamente existe alguna coincidencia, en mi caso vamos a utilizar la web: https://crackstation.net/


![](/assets/images/thm-mustacchio/7.jpg)

Bingo, parece ser que tenemos unas credenciales potenciales del usuario admin, recordaís el puerto 8765 que era un Login, vamos a probar las credenciales.

![](/assets/images/thm-mustacchio/8.jpg)

Ya estamos dentro.

![](/assets/images/thm-mustacchio/9.jpg)

## 3-Explotación 

Vemos una pagina simple, que contiene un recuadro para "añadir un comentario a la web" , vamos a probar a escribir "hola".

![](/assets/images/thm-mustacchio/10.jpg)

Parece ser que estamos frente a un panel que solo admite comentarios en formato XML , por la estructura que ofrece.
Vamos a seguir enumerando el panel de administrador, pulsaremos Ctrl + U para ver el código fuente de la página.

![](/assets/images/thm-mustacchio/11.png)


Aqui tenemos que destacar tres cosas.
1-La primera es el código en javascript, que trata de mirar el contenido que hemos introducido en la caja de comentario, si el value de la caja es 0 , es decir no hemos escrito nada o esta vacia, este nos devuelve un mensajito diciendonos que insertemos código XML. Pero si nos fijamos bien tenemos un archivo al parecer de ejemplo, luego veremos de que trata.
2-Tenemos un comentario HTML , el cual nos comenta que Barry ya puede acceder a SSH utilizando su clave.
3-Por ultimo al final tenemos lo que podría ser nuestra estructura HTML

Vamos a mirar el archivo de ejemplo.

![](/assets/images/thm-mustacchio/12.png)

Tenemos un comentario totalmente absurdo, pero lo que a nosotros nos importa es la estructura XML que sige, vamos a intentar utilizarla para conseguir introducir un comentario en la página.

Vamos a escribir la siguiente estructura XML.

```
<?xml version="1.0" encoding="UTF-8"?>
<comment>
  <name>rincon</name>
  <author>rincon2</author>
  <com>rincon3</com>
</comment>
```
![](/assets/images/thm-mustacchio/13.jpg)


Vemos que nos deja escribir el comentario, ahora vamos a intentar leer algún archivo del sistema mediante la vulnerabilidad xxe.

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE replace [<!ENTITY rincon3 SYSTEM "file:///etc/passwd"> ]>
<comment>
  <name>rincon</name>
  <author>rincon2</author>
  <com>&rincon3;</com>
</comment>
```

Este payload xxe, lo que ara es mediante la entidad "rincon3" , intentara leer el archivo /etc/passwd mediante el wrapper file://
Utilizaremos Ctrl + U para verlo mejor.

![](/assets/images/thm-mustacchio/14.jpg)

Perfecto ya tenemos el archivo /etc/passwd , podemos ver un usuario llamado "barry" , si recordamos anteriormente nos chivaron que barry ya tenia acceso a SSH mediante su clave asi que vamos a intentar listar su clave ssh.

Ahora utilizaremos la misma estructura solo que buscaremos el archivo /home/barry/.ssh/id_rsa
Utilizaremos Ctrl + U para verlo mejor.

![](/assets/images/thm-mustacchio/15.jpg)

Aqui tenemos su clave privada, asi que vamos a acceder vía SSH.

![](/assets/images/thm-mustacchio/16.jpg)

Vemos que la clave id_rsa esta protegida con una contraseña, en este caso utilizaremos ssh2john para descifrar este tipo de claves.
Utilizaremos primero el comando: python /usr/share/john/ssh2john id_rsa > id_rsa2.txt

Por ultimo ejecutaremos john para descifrarla.

![](/assets/images/thm-mustacchio/17.jpg)

Una vez descifrada vamos a intentar conectarnos

![](/assets/images/thm-mustacchio/18.jpg)

Ahora sí, ya vemos la flag del usuario.

## 4-Escalación de privilegios.

Si recordamos antes en el /etc/passwd , vimos otro usuario llamado joe, vamos a ver su directorio personal.

![](/assets/images/thm-mustacchio/19.jpg)

Si listamos los permisos del archivo "live_log" , por un lado sabemos que es un binario compilado y que es un SUID con el propietario root, por lo cual si conseguimos ejecutar este binario como usuario privilegiado ya tendriamos acceso a root. Con strings vamos a ver por encima que es el archivo.

![](/assets/images/thm-mustacchio/20.jpg)

Realmente vemos poca cosa, aunque si nos fijamos hay una línea llamada "system" , podría ser una línea que ejecuta un código que no conseguimos ver. Vamos a descargar el archivo y a verlo mas detenidamente.

![](/assets/images/thm-mustacchio/21.jpg)

Para ver mejor el contenido del archivo vamos a aplicar ingenieria inversa mediante la herramineta "Ghidra".

![](/assets/images/thm-mustacchio/22.jpg)

Bien, vemos que el código hace lo siguiente -> system("tail -f /var/log/nginx/access.log") ejecuta ese comando. Vemos que hay un gran fallo de seguridad en el código y es que ejecuta el binario del sistema "tail" sin su ruta completa, esto significa que si somos capaces de hacer que el sistema ejecute un binario llamado "tail" malicioso en vez del tail del sistema, conseguiremos acceso.

Vamos a crear un binario en nuestro directorio personal de barry y vamos a hacer que nos devuelva una bash (bash -p) , le asignamos privilegios de ejecución y a modificar el PATH.

![](/assets/images/thm-mustacchio/23.jpg)

Ahora vamos a movernos al directorio de joe y vamos a ejecutar el binario -> ./live_log

![](/assets/images/thm-mustacchio/24.jpg)

y del tiron tenemos una consola como root! , al crear un archivo llamado igual que el binario tail del sistema y modificar el PATH del usuario para que primero revise el directorio que nosotros queramos, eso nos da la autoridad de crear un archivo con el mismo nombre y el contenido que nosotros queramos ejecutar, simplemente nos movemos a la carpeta de root y listamos la flag.

![](/assets/images/thm-mustacchio/final.jpg)


















