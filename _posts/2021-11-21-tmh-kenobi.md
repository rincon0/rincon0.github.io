---
layout: single
title: kenobi - Try Hack Me
excerpt: Kenobi maquina catalogada de dificultad “Easy”basada en samba, manipulación de la variable PATH, Suid y Smb vamos a resolverla! 
date: 2021-11-21
classes: wide
header:
  teaser: /assets/images/thm-kenobi/1.JPG
  teaser_home_page: true
  icon: /assets/images/thm.webp
categories:
  - TryHackMe
tags:  
  - Linux
  - path
  - suid
  - smb
  - samba
  - Easy
  
---

![](/assets/images/thm-kenobi/1.JPG)

Kenobi maquina catalogada de dificultad “Easy”basada en samba, manipulación de la variable PATH, Suid y Smb vamos a resolverla, esta maquina es muy interesante ya que tenemos que combinar varias técnicas e ir combinandolas hasta llegar a la flag final.

## 1-Escaneo Nmap

Iniciamos la maquina y hacemos un escaneo con Nmap básico: nmap -sC -sV 10.10.36.172 - o EscaneoNmap

Utilizaremos los parametros -sC (Utilizar los scripts más comunes) -sV (Mostrar versiones) -o (Guardar archivo)

Nmap nos devuelve todo esto:

```
 nmap -sC -sV 10.10.36.172 -o EscaneoNmap
Starting Nmap 7.92 ( https://nmap.org ) at 2021-11-20 23:55 CET
Nmap scan report for 10.10.36.172 (10.10.36.172)
Host is up (0.041s latency).
Not shown: 993 closed tcp ports (conn-refused)
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         ProFTPD 1.3.5
22/tcp   open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 b3:ad:83:41:49:e9:5d:16:8d:3b:0f:05:7b:e2:c0:ae (RSA)
|   256 f8:27:7d:64:29:97:e6:f8:65:54:65:22:f7:c8:1d:8a (ECDSA)
|_  256 5a:06:ed:eb:b6:56:7e:4c:01:dd:ea:bc:ba:fa:33:79 (ED25519)
80/tcp   open  http        Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
| http-robots.txt: 1 disallowed entry 
|_/admin.html
111/tcp  open  rpcbind     2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/tcp6  nfs
|   100003  2,3,4       2049/udp   nfs
|   100003  2,3,4       2049/udp6  nfs
|   100005  1,2,3      40217/tcp   mountd
|   100005  1,2,3      42324/udp6  mountd
|   100005  1,2,3      42429/udp   mountd
|   100005  1,2,3      55883/tcp6  mountd
|   100021  1,3,4      33199/tcp   nlockmgr
|   100021  1,3,4      35438/udp   nlockmgr
|   100021  1,3,4      38773/tcp6  nlockmgr
|   100021  1,3,4      39277/udp6  nlockmgr
|   100227  2,3         2049/tcp   nfs_acl
|   100227  2,3         2049/tcp6  nfs_acl
|   100227  2,3         2049/udp   nfs_acl
|_  100227  2,3         2049/udp6  nfs_acl
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
2049/tcp open  nfs_acl     2-3 (RPC #100227)
Service Info: Host: KENOBI; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 2h00m01s, deviation: 3h27m51s, median: 0s
|_nbstat: NetBIOS name: KENOBI, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-time: 
|   date: 2021-11-20T22:55:18
|_  start_date: N/A
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: kenobi
|   NetBIOS computer name: KENOBI\x00
|   Domain name: \x00
|   FQDN: kenobi
|_  System time: 2021-11-20T16:55:18-06:00

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 17.03 seconds
```

Vemos que nos devuelve bastantes puertos abiertos. Algunos insteresantes son 21 (FTP) 22 (SSH) 445 (Samba)

En el servidor web nos indica que es una trampa, por lo cual no hay nada interesante por ningun lado.

![](/assets/images/thm-kenobi/web1.JPG)

![](/assets/images/thm-kenobi/2.JPG)

## 2-Enumeración samba

Vamos a utilizar la herramienta "smb" que se encarga de enumerar samba. Es una herramienta diseñada por la comunidad Cybex (Enlace: https://cybexsec.es/smb.txt)

![](/assets/images/thm-kenobi/3.JPG)

Lo que podríamos destacar de aquí es el nombre del host llamado: anonymous y su variable PATH C:\home\kenobi\share

Vamos a conectarnos con el usuario anonymous sin indicar la contraseña: smbclient //10.10.36.172/anonymous -N , y vamos a listar contenido.

![](/assets/images/thm-kenobi/4.JPG)

Vemos que tenemos un archivo llamado "log.txt" , vamos a traernoslo a nuestro equipo con el comando "get" y vamos a echarle un ojo.

También podríamos descargar todo el contenido con: smbget -R smb://10.10.36.172/anonymous

![](/assets/images/thm-kenobi/5.JPG)

Ahora sabemos la ruta de la clave id_rsa pero no sabemos el punto de montaje del servidor para traernos el archivo, para ello vamos a enumerar el puerto 111 para saber el punto de montaje, ¿Por que escanear rpcbind? Este es un servidor que junta todos los procesos RPC (Remote Procedure Call) en una única dirección. Funciona de la siguiente manera, cuando un proceso RPC se ejecuta le dice al servidor rpcbind el programa que que esta llevando acabo y el número del mismo, para nuestro caso el puerto 111 es el acceso a un sistema de archivos de red.

Utilizaremos: nmap -p 111 --script=nfs-ls,nfs-statfs,nfs-showmount 10.10.36.172

También podemos utilizar: showmount -e

![](/assets/images/thm-kenobi/6.JPG)

![](/assets/images/thm-kenobi/7.JPG)

![](/assets/images/thm-kenobi/8.JPG)

![](/assets/images/thm-kenobi/9.JPG)

![](/assets/images/thm-kenobi/10.JPG)

## 3-Servidor FTP

Sabemos que la versión del servidor FTP es: ProFTPD 1.3.5 , utilizarmos "searchsploit" para buscar alguna vulnerabilidad

![](/assets/images/thm-kenobi/11.JPG)

Vamos a descargar con el comando "searchsploit -m" los archivos que queramos

Echaremos un ojo al archivo .py y al .txt

![](/assets/images/thm-kenobi/12.JPG)

Vemos que el archivo python es una ejecución de código .php para indicar el comando que queramos (Esto no sirviria de nada ya que como hemos visto antes en la página web no hay nada interesante), en el archivo .txt muestra algo interesante, Unos comandos para Copiar y Pegar (CPFR y CPTO) archivos dentro del propio servidor FTP. Con esta vulnerabilidad podriamos intentar copiar la clave id_rsa a un sitio donde tengamos acceso para poder descargarlos.

Vamos a ello, vamos a intentar conectarnos al servidor FTP mediante netcat con el comando: nc 10.10.36.172 21

![](/assets/images/thm-kenobi/13.JPG)

Copiariamos el archivo id_rsa a un sitio donde si podramos tener acceso, aunque aún no lo hayamos logrado, en el directorio /var/tmp (/var sería el punto de montaje y /tmp tendriamos permisos de escritura de forma generica) y le ponemos el nombre "id_rsa" quedaría: /var/tmp/id_rsa

¿Entonces ahora como accederiamos a ese archivo? , vamos a crear un punto de montaje local para intentar acceder al mismo, utilizariamos los siguientes comandos:

sudo mkdir idrsakenobi ----------------------------------------------------> Creamos una carpeta donde queramos montar el directorio /var
sudo mount 10.10.36.172:/var /home/rincon/thm/kenobi/idrsakenobi ----------> Montariamos el directio /var de la dirección IP de la maquina a nuestra carpeta personal

Lo cual esto nos permite tener acceso a todo el directorio /var en nuestra carpeta, vamos a listar el contenido haber si a funcionado correctamente.

![](/assets/images/thm-kenobi/14.JPG)

Bien, ahora vamos a traernos la clave id_rsa a nuestro equipo.

![](/assets/images/thm-kenobi/15.JPG)

Vamos a asignarle los permisos 600 para que esta clave se pueda utilizar y vamos a conectar con el usuario "kenobi" a la maquina ssh, vemos que tenemos acceso asi que vamos a listar el contenido y ver el archivo user.txt

![](/assets/images/thm-kenobi/16.JPG)

![](/assets/images/thm-kenobi/17.JPG)

![](/assets/images/thm-kenobi/18.JPG)

![](/assets/images/thm-kenobi/19.JPG)

## 4-Escalación de privilegios

Ahora vamos a escalar privilegios abusando de los permisos SUID, es una manera de ejecutar binarios que deberian estar restringidos solo por el usuario root.

Para abusar de estos permisos, para buscar que comandos podemos utilizar como SUID utilizariamos:

find / -perm -u=s -type f 2>/dev/null
find / -type f -a \( -perm -u+s -o -perm -g+s \) -exec ls -l {} \; 2> /dev/null

Una vez ejecutado algunos de estos comandos, vamos a mirar si encontramos algo inusual, me refiero a algo que no este por defecto en un sistema linux, o algo que no deberiamos de poder ejecutar.

![](/assets/images/thm-kenobi/20.JPG)

Encontramos el archivo /etc/passwd , que nos muestra los usuarios del equipo, además de otras muchas cosas. Aunque no logramos encontrar nada interesante.

Encontramos el archivo /usr/bin/menu , esto no es común en linux, asi que vamos a ejecutarlo y a saber que hace, vemos que es un menu simple que nos muestra varias opciones, vamos a ver como esta escrito. Podemos utilizar el comando "cat /usr/bin/menu" para ver el contenido o ejecutar el comando "strings /usr/bin/menu" que nos lo mostraría de una manera mas legible.

![](/assets/images/thm-kenobi/21.JPG)

![](/assets/images/thm-kenobi/22.JPG)

Vemos que el menu es bastante simple, solo ejecuta un comando "ifconfig o uname -r". Lo interesante de esto es que este menu se ejecuta con permisos de superusuario , es decir si somos capaces de engañar al sistema y que en vez de llamar a ifconfig, llamamos a /bin/sh estariamos ejecutando una shell como root. 

Lo bueno es que este programa no utiliza la ruta completa, por lo cual mediante el abuso de las variables de entorno PATH , podriamos ejecutar antes los comandos que indiquemos. Esto funciona por que cuando se llama a un binario solo por su nombre en vez de por su ruta completa, el sistema va a mirar en la variable $PATH para encontralo, si somos capaces de anteponernos antes de que encuentre la ruta original, seremos capaces de ejecutar lo que queramos.

Nos pondríamos en un directorio donde tengamos permisos de escritura (En nuestro caso /tmp) y definiarmos un binario malicioso , le dariamos permisos y definiriamos una nueva variable PATH

Una vez obtenida la shell como root solo nos quedaría encontrar las flags

![](/assets/images/thm-kenobi/23.JPG)

![](/assets/images/thm-kenobi/24.JPG)

![](/assets/images/thm-kenobi/25.JPG)

![](/assets/images/thm-kenobi/kenobi.JPG)