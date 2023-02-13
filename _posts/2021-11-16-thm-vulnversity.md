---
layout: single
title: vulnversity - Try Hack Me
excerpt: vulnversity, maquina catalogada de dificultad “easy”, basada en reconocimiento, busqueda de directorios, web y escalación de privilegios!
date: 2021-11-07
classes: wide
header:
  teaser: /assets/images/thm-vulnversity/1.JPG
  teaser_home_page: true
  icon: /assets/images/thm.webp
categories:
  - TryHackMe
tags:  
  - Linux
  - recon
  - webappsec
  - privesc
  - Easy
  
---

![](/assets/images/thm-vulnversity/1.JPG)

Vulnversity , maquina catalogada de dificultad “easy”, basada en reconocimiento, busqueda de directorios, web y escalación de privilegios!

## 1-Escaneo Nmap

Iniciamos la maquina y hacemos un escaneo con Nmap básico: nmap -sC -sV 10.10.89.245

Para hacer un escaneo completo y así asegurarnos que no nos dejamos ningun puerto atrás utilizaremos: nmap -p -T5 10.10.89.245

Nmap nos devuelve todo esto:

```
┌──(rincon㉿rincon)-[~]
└─$ nmap -sCV 10.10.87.19 -o PortsNmap
Starting Nmap 7.91 ( https://nmap.org ) at 2021-11-16 16:21 CET
Nmap scan report for 10.10.87.19 (10.10.87.19)
Host is up (0.042s latency).
Not shown: 994 closed ports
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 3.0.3
22/tcp   open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 5a:4f:fc:b8:c8:76:1c:b5:85:1c:ac:b2:86:41:1c:5a (RSA)
|   256 ac:9d:ec:44:61:0c:28:85:00:88:e9:68:e9:d0:cb:3d (ECDSA)
|_  256 30:50:cb:70:5a:86:57:22:cb:52:d9:36:34:dc:a5:58 (ED25519)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
3128/tcp open  http-proxy  Squid http proxy 3.5.12
|_http-server-header: squid/3.5.12
|_http-title: ERROR: The requested URL could not be retrieved
3333/tcp open  http        Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Vuln University
Service Info: Host: VULNUNIVERSITY; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 1h40m00s, deviation: 2h53m12s, median: 0s
|_nbstat: NetBIOS name: VULNUNIVERSITY, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: vulnuniversity
|   NetBIOS computer name: VULNUNIVERSITY\x00
|   Domain name: \x00
|   FQDN: vulnuniversity
|_  System time: 2021-11-16T10:21:31-05:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-11-16T15:21:31
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 28.38 seconds
```

```
┌──(rincon㉿rincon)-[~]
└─$ nmap -p- -T5 10.10.87.19                  
Starting Nmap 7.91 ( https://nmap.org ) at 2021-11-16 16:22 CET
Warning: 10.10.87.19 giving up on port because retransmission cap hit (2).
Nmap scan report for 10.10.87.19 (10.10.87.19)
Host is up (0.041s latency).
Not shown: 63216 closed ports, 2313 filtered ports
PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
3128/tcp open  squid-http
3333/tcp open  dec-notes

Nmap done: 1 IP address (1 host up) scanned in 43.47 seconds
```

Vemos que nos devuelve bastantes puertos abiertos. Nos quedaremos con el puerto 3333 que parece que tiene un servicio web.

![](/assets/images/thm-vulnversity/2.JPG)

## 2-Fuzzing con gobuster 

Ahora vamos a hacer un escaneo con gobuster, vemos que tenemos varios directorios, el que más nos interesa es el de “internal”

![](/assets/images/thm-vulnversity/3.JPG)

![](/assets/images/thm-vulnversity/4.JPG)

## 3-Comprometiendo al servidor

Vemos que encontramos una pagina de carga de archivos, aquí podemos intentar subir algún archivo para obtener una reverse shell y así obtener acceso a la maquina, para comprobar si tiene algún filtro voy a intentar subir un archivo aleatorio con la extension .txt y vemos que nos lo bloquea, esto nos indica de que posiblemente estaremos frente a una lista blanca en el backend.

![](/assets/images/thm-vulnversity/5.JPG)

Para no ir probando extension por extension, utilizaremos BurpSuite para capturar el paquete y poder modificar la extensión con un diccionario definido en kali con la herramienta intruder desde BurpSuite.

![](/assets/images/thm-vulnversity/6.JPG)

Ahora pulsamos Ctrl+I para enviarlo al intruder, modificaremos el apartado que queremos cambiar (Pondriamos las llaves en el .txt), desactivaremos el "Payload Enconding" de la parte baja para que no sustituya los puntos y para hacer fuerza bruta a las extensiones podríamos utilizar algún diccionario o probar con las extensiones php mas comunes por ejemplo.

![](/assets/images/thm-vulnversity/7.JPG)

Vemos que la extensión .phtml la acepta, ahora vamos a descargar la shell que nos proporciona la propia tarea y vamos a añadirle la extensión .phtml , dentro tendremos que configurarle nuestra dirección IP (tun0) y un puerto aleatorio por el cual nos pondremos en escucha.

Para que nos de la reverse shell necesitaremos acceder al archivo "shell.phtml" para ello vamos a probar el directorio http://10.10.87.19:3333/internal/uploads/ y bingo parece que hemos encontrado a la primera donde se guardan los archivos en el servidor, normalmente tendriamos que probar más, aunque lo mejor es hacer una busqueda de subdirectorios aunque en este caso no es necesario.

![](/assets/images/thm-vulnversity/8.JPG)

Vemos que ya tenemos la reverse shell. Asi que vamos a buscar alguna flag en el usuario, en este caso estaba en el usuario Bill.

![](/assets/images/thm-vulnversity/9.JPG)

![](/assets/images/thm-vulnversity/10.JPG)

![](/assets/images/thm-vulnversity/11.JPG)

![](/assets/images/thm-vulnversity/12.JPG)

![](/assets/images/thm-vulnversity/13.JPG)

## 4-Escalación de privilegios

Ahora vamos a escalar privilegios, en la misma actividad nos hace referencia a los permisos SUID así que vamos a utilizar el comando: find / -user root -perm -4000 -exec ls -ldb {} \; para buscar todos los binarios que podemos utilizar.

![](/assets/images/thm-vulnversity/14.JPG)

Vemos que hay un SUID que se llama /bin /systemctl , es bastante curioso que podamos acceder a el ya que es una utilidad para administrar el sistema y el administrador de servicios.

Para ver si podemos escalar privilegios con el yo utilizare esta web: https://gtfobins.github.io

Voy a buscar por “Systemctl” y encontramos esto:

![](/assets/images/thm-vulnversity/15.JPG)

Vamos a ejecutar los comandos y a escalar privilegios!

![](/assets/images/thm-vulnversity/16.JPG)

Vamos a sacar una bash con permisos de superusuario, por lo cual le vamos a añadir unos permisos de SUID al mismo y con el comando "bash -p" la vamos a llamar y con el -p hacemos que se mantenga la sesión con privilegios.

![](/assets/images/thm-vulnversity/17.JPG)

