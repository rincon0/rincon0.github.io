---
layout: single
title: Avengers - Try Hack Me
excerpt: Avengers, maquina catalogada de dificultad “Easy” en esta ocasion nos enfrentaremos a enumeracion de diferentes servicios, además de realizar un ataque sql injection y Remote code execution.
date: 2022-01-27
classes: wide
header:
  teaser: /assets/images/thm-avengers/1.JPG
  teaser_home_page: true
  icon: /assets/images/thm.webp
categories:
  - TryHackMe
tags:  
  - web
  - sqli
  - linux
  - rce
  - Easy
  
---

![](/assets/images/thm-avengers/1.JPG)

Avengers, maquina catalogada de dificultad “Easy” en esta ocasion nos enfrentaremos a enumeracion de diferentes servicios, además de realizar un ataque sql injection y Remote code execution.

## 1-Escaneo Nmap y enumeración de puertos.

realizamos un escaneo con Nmap para descubrir todos los puertos abiertos: nmap -p- --open -T5 -v -n -Pn 10.10.47.79 -oN allports.txt

Utilizaremos los parametros -p- (Indicar todos los puertos) --open (Filtrar por puertos abiertos) -T5 (Temporizado del escaneo, para más velocidad) -v (Arrojar un poco de inforación mientras se realiza el escaneo) -n (No aplicar resolución DNS) -oN (Guardar archivo)

Nmap nos devuelve todo esto:

```
nmap -p- --open -T5 -v -n -Pn 10.10.78.136 -oN nmap.txt
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.92 ( https://nmap.org ) at 2022-01-27 00:12 CET
Initiating Connect Scan at 00:12
Scanning 10.10.78.136 [65535 ports]
Discovered open port 80/tcp on 10.10.78.136
Discovered open port 22/tcp on 10.10.78.136
Discovered open port 21/tcp on 10.10.78.136
Completed Connect Scan at 00:12, 14.47s elapsed (65535 total ports)
Nmap scan report for 10.10.78.136
Host is up (0.044s latency).
Not shown: 63655 closed tcp ports (conn-refused), 1877 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 14.65 seconds
```

Vemos que nos devuelve bastantes puertos abiertos. Algunos insteresantes son el puerto 80 (http) , 22 (ssh) y 21(ftp)

A continuación vamos a realizar un escaneo más profundo a estos puertos con el siguiente comando:

nmap -sC -sV  10.10.78.136 -oN ports.txt

Parametros -sC (Utilizar scripts comunes) y -sV (Mostrar versiones y servicios) -oN (Guardar el archivo)


Nmap nos devulve esto:

```
nmap -sC -sV 10.10.78.136 -oN PortsNmap.txt
Starting Nmap 7.92 ( https://nmap.org ) at 2022-01-27 00:14 CET
Nmap scan report for 10.10.78.136 (10.10.78.136)
Host is up (0.042s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 ee:66:2a:9d:e6:13:69:9a:25:03:11:4f:93:72:84:a2 (RSA)
|   256 04:cb:94:96:06:05:94:88:97:5f:ae:4c:d7:8e:bc:43 (ECDSA)
|_  256 3b:fa:fa:37:13:4f:8d:15:75:32:9b:c6:0b:9f:8b:d0 (ED25519)
80/tcp open  http    Node.js Express framework
|_http-title: Avengers! Assemble!
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.05 seconds
```

Utilizaremos también whatweb para enumerar rapidamente el servicio http y ver ante que nos estamos enfrentando.

Nos duelve esto:

```
whatweb 10.10.78.136
http://10.10.78.136 [200 OK] Bootstrap, Cookies[connect.sid], Country[RESERVED][ZZ], HTML5, HttpOnly[connect.sid], IP[10.10.78.136], JQuery, Script, Title[Avengers! Assemble!], UncommonHeaders[flag2], X-Powered-By[Express]
```

Vemos que nos arroja poca información, página escrita en python con el titulo My blog, entre otras cosas...

## 2-Enumeración puerto 80.

Vamos a ver el aspecto de la pagina web.

![](/assets/images/thm-avengers/2.JPG)

Tenemos una web simple con una imagen y unos comentarios de los superheroes de avengers.

Vamos a mirar el código fuente para ver que encontramos.

![](/assets/images/thm-avengers/3.JPG)

Lo único interesante que nos encontramos es un comentario HTML "<!-- Hint -->" Si nos metemos en el enlace que nos lleva al script js, vemos una pagina con esto.

![](/assets/images/thm-avengers/4.JPG)

Es el valor de una cookie que hace refencia a la primera flag, es la misma cookie que se nos asocia en la página web.

![](/assets/images/thm-avengers/5.JPG)

![](/assets/images/thm-avengers/6.JPG)

La segunda flag la encontraremos en el encabezado de la petición HTTP. Si nos vamos a la consola de desarrollador , pulsando Ctrl + Shift + C , en el apartado de red , seleccionamos la última petición GET y vemos el contenido de la flag.

![](/assets/images/thm-avengers/7.JPG)

![](/assets/images/thm-avengers/10.JPG)

Para seguir enumerando la página web, vamos a utilizar la herramienta gobuster para realizar fuzzing a los directorios.

gobuster -u http://10.10.78.136/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt

![](/assets/images/thm-avengers/8.JPG)

Tenemos un directorio intersante llamado /portal y dentro tenemos un login.

![](/assets/images/thm-avengers/9.JPG)

![](/assets/images/thm-avengers/11.JPG)

## 3-SQL Injection 

Si nos fijamos en el código fuente de la página del login, hay un comentario que nos chiva "Recuerde desinfectar el nombre de usuario y la contraseña para detener SQLi" , esto nos da una pista , el login podría ser vulnerable a ataques SQL Injection, vamos a probar.

![](/assets/images/thm-avengers/cod.JPG)

Hay muchas maneras de realizar este tipo de ataques, nosotros vamos a comenzar probando si la entrada esta desinfectada o no.

Para ello vamos a abrir BurpSuite, vamos a capturar un petición de login y vamos a mandarlo al Intruder para hacer fuzzing.

![](/assets/images/thm-avengers/12.JPG)

Seleccionamos el campo que queramos fuzzear , en este caso comenzaremos probando con el usuario, nos vamos a payloads y seleccionamos una lista de SQL inyection.

IMPORTANTE , retirar el URL encoding para que no nos modifique la petición.

![](/assets/images/thm-avengers/13.JPG)

Iniciamos el ataque y vemos que ninguno funciona, todos nos devuelve la misma longitud, entonces vamos a probar fuzzeando el campo de la contraseña.

![](/assets/images/thm-avengers/14.JPG)

Tendríamos que realizar el mismo procedimiento solo que seleccionando esta vez el campo de "password"

![](/assets/images/thm-avengers/15.JPG)

Vemos que tampoco nos deja acceder de esta manera, por lo cual vamos a fuzzear los dos campos a la vez, "username" y "password", para ello seleccionamos el modo "BATTERING RAM"

![](/assets/images/thm-avengers/16.JPG)

![](/assets/images/thm-avengers/17.JPG)

Vemos que ahora sí, nos devuelve una longitud diferente, vamos a probar en el login y ¡BINGO!

![](/assets/images/thm-avengers/18.JPG)

![](/assets/images/thm-avengers/19.JPG)

![](/assets/images/thm-avengers/sq.JPG)

## 4-Remote Code Execution

Tenemos un RCE para poder ejecutar comandos, vamos a intentar poner alguno.

![](/assets/images/thm-avengers/20.JPG)

Vemos que tenemos algunos filtros, vamos a intentar evasionarlos utilizando diferentes técnicas.

"ls" nos deja utilizarlo, por lo cual vamos a intentar listar el contenido de /etc/passwd

![](/assets/images/thm-avengers/a.JPG)

Vamos a leer el contenido del archivo.

![](/assets/images/thm-avengers/b.JPG)

El comando "cat" esta prohibido, utilizaremos uno similar "less"

![](/assets/images/thm-avengers/c.JPG)

Fijaos en el primer usuario llamado "ubuntu" 1000:1000 , vamos a listar el /home

![](/assets/images/thm-avengers/d.JPG)

Tenemos dos usuarios, ubuntu y groot, listaremos el contenido de ubuntu en busca de la quinta flag.

![](/assets/images/thm-avengers/21.JPG)

![](/assets/images/thm-avengers/22.JPG)

Aqui tenemos la quinta flag.

![](/assets/images/thm-avengers/ult.JPG)

## 5-FTP

Recordemos que teniamos un puerto 21 abierto, el servicio FTP, la propia actividad nos da un usuario y contraseña para utilizar y enumerar.

User:groot

Pass:iamgroot

Utilizaremos -> ftp 10.10.78.136 , para conectarnos y proporcionamos las credenciales.

![](/assets/images/thm-avengers/23.JPG)

Vemos que dentro del directorio "files" tenemos un archivo llamado flag3.txt , nos lo descargamos y vemos su contenido.

![](/assets/images/thm-avengers/24.JPG)

![](/assets/images/thm-avengers/25.JPG)
