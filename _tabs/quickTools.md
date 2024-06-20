---
layout: page
icon: fas fa-code
order: 4
---

El propósito de esta sección, es ir alimentandolo, a medida que encuentre comandos recurrentes e importantes, para así tener un acceso rápido a estos, junto a una pequeña descripción

- [Reconocimiento](#reconocimiento)
  * [Nmap](#nmap)
  * [Puertos y Servicios](#puertos-y-servicios)
    - [lsof](#lsof)
    - [netstat](#netstat)
- [Análisis de Vulnerabilidades](#explotación-y-post-explotación)
  * [Hydra](#hydra)
- [Explotación y Post-Explotación](#explotación-y-post-explotación)
  * [Tratamiento TTY Linux](#tratamiento-de-la-tty---linux)
  * [Tratamiento TTY Windows](#tratamiento-de-la-tty---windows)
  * [Nmap](#nmap)

[comment]: <> ([+]---------------------Reconocimiento---------------------[+])
## Reconocimiento

### Nmap

Es una herramienta de escaneo de puertos que permite descubrir hosts en una red y determinar qué servicios están disponibles en esos hosts. Ayuda a recopilar información sobre la infraestructura de red y los sistemas objetivo.

Es importante tener permisos de Amdn, para así poder utilizar el parámetro `-Pn`
```bash
sudo nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn {ip_target} -oG allPorts
```
{: .nolineno }

Una vez tenemos los puertos, es importante obtener la versión y el servicio que se estan ejecutando en los puertos anteriormente encontrados
```bash
nmap -sCV -p{ports} {ip_target} -oN target
```
{: .nolineno }

### Gobuster

```bash
gobuster dir -u {url} -w {wordList} -t {Threads}
```
{: .nolineno }

A nivel de Subdominios: 

```bash
gobuster vhost -u {url} -w {wordList} -t {Threads:20}
```
{: .nolineno }

### WFUZZ

### Hydra
Hydra es la mejor herramienta para aplicar fuerza bruta en distintos componentes

#### ssh
Fuerza bruta a un servicio ssh
```bash
hydra -L <dicUsers.txt> -P <dicPass.txt> <IP Target> ssh -t <threads> # Usamos diccionarios
hydra -l pepito -p pepitoPass <IP Target> ssh -t <threads> # Conocemos Usuario o Contraseña
hydra -s <port> # Cambia el puerto
hydra -M <IP Targets List> # Tenemos una lista de IPs
-V # Agrega verbosidad por pantalla
-e nsr # Uso en CTFs n:Null - s:SamePass - r:reverse
```
{: .nolineno }


### Puertos y servicios

#### lsof
Puede ser útil en la fase de enumeración para identificar procesos en ejecución y los archivos asociados a ellos, lo que puede revelar información valiosa sobre el sistema objetivo.

```bash
lsof -i :{port_number}
lsof -i :80
```
{: .nolineno }

#### netstat

Se utiliza para mostrar información y estadísticas relacionadas con las conexiones de red activas, los puertos abiertos y otros datos de red.

```bash
netstat -tulnp
netstat -nat
```
{: .nolineno }

|Parámetro|Función                                    |
|:-------:|:-----------------------------------------:|
| `-t`    |conexiones TCP establecidas                |
| `-u`    |conexiones UDP establecidas                |
| `-l`    |Puertos en escucha                         |
| `-n`    |Direcciones IP y puertos en números        |
| `-p`    |Proceso asociado a cada conexión o servicio|
| `-a`    |Todas las conexiones y puertos, incluyendo los que están en escucha y los que están establecidos|

### tcpdump
Para ponernos en escucha, en espera a recibir una traza icmp

```bash
tcpdump -i {interface} icmp -n
```
{: .nolineno }



[comment]: <> ([+]---------------------Explotación y Post-Explotación---------------------[+])
## Explotación y Post-Explotación

### Tratamiento de la TTY - Linux

Primero es importante colocar algún puerto en escucha utilizando ncat

```bash
nc -nlvp <port>
```
{: .nolineno }


|Parámetro|Función                                    |
|:-------:|:-----------------------------------------:|
| `-n`    |Solo conexiónes mediante IP no DNS         |
| `-l`    |Escucha (Listen)                           |
| `-v`    |Verbose, más detalle por pantalla          |
| `-p`    |Puerto por el que va a trabajar (Escuchar) |

Posterior vamos a ejecutar esta serie de comandos, lo que esté en {} va a ser opcional

```bash
script /dev/null -c bash
Ctrl + Z
stty raw -echo; fg
reset xterm
export TERM=xterm
# Si quieres colores puedes realizar esto
export TERM=xterm-256color
source /etc/skel/.bashrc

→ stty size 
stty rows (45) columns (184) 
```

### Tratamiento de la TTY - Windows

Primero es importante colocar algún puerto en escucha utilizando ncat, acá podemos jugar también con `rlwrap`

```bash
rlwrap -cAr nc -nlvp <port>
```

|Parámetro|Función                                                |
|:-------:|:-----------------------------------------------------:|
| `-c`    | Habilita el autocompletado de nombres de archivo      |
| `-A`    | Funciona mejor con comandos que utilizan colores ANSI |
| `-r`    | recordar todas las palabras para autocompletado       |








[comment]: <> ([+]---------------------Análisis de Vulnerabilidades---------------------[+])



### Tareas Cron

Para ver que se está ejecutando a intervalos regulares de tiempo, podemos ejecutar este comando
```bash
systemctl list-timers
```
{: .nolineno }


Este sirve para ver que está corriendo, y que usuario lo ha ejecutado

```bash
ps -eo user,command
```
{: .nolineno }

### John The Ripper
```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hash
```
{: .nolineno }

### watch
Para revisar el mismo comando cada $x$ tiempo:
```bash
watch -n {s} {command}
watch -n 1 ls -l /bin/bash
```
{: .nolineno }

### searchsploit
```bash
❯ searchsploit elastix
------------------------------------------------------------------------------ ------------------------
 Exploit Title                                                                |  Path
------------------------------------------------------------------------------ ------------------------
Elastix - 'page' Cross-Site Scripting                                         | php/webapps/38078.py
Elastix - Multiple Cross-Site Scripting Vulnerabilities                       | php/webapps/38544.txt
Elastix 2.0.2 - Multiple Cross-Site Scripting Vulnerabilities                 | php/webapps/34942.txt
Elastix 2.2.0 - 'graph.php' Local File Inclusion                              | php/webapps/37637.pl
Elastix 2.x - Blind SQL Injection                                             | php/webapps/36305.txt
Elastix < 2.5 - PHP Code Injection                                            | php/webapps/38091.php
FreePBX 2.10.0 / Elastix 2.2.0 - Remote Code Execution                        | php/webapps/18650.py
------------------------------------------------------------------------------ -------------------------
Shellcodes: No Results
```
{: .nolineno }

```bash
❯ searchsploit -x php/webapps/37637.pl
  Exploit: Elastix 2.2.0 - 'graph.php' Local File Inclusion
      URL: https://www.exploit-db.com/exploits/37637
     Path: /usr/share/exploitdb/exploits/php/webapps/37637.pl
    Codes: N/A
 Verified: True
File Type: ASCII text
```
{: .nolineno }



### crackmapexec
```bash
crackmapexec smb <ip_target>
```

### responder




### smbclient




### rpcclient


### kerberos

### evil-winrm
```bash
evil-winrm -i {ip_target} -u {user_target} -p {user_passwd}
```
{: .nolineno }



## Ataques

### AS-REP Roasting Attack

Que utiliza la suite `impacket`, se debe tener en cuenta que debemos tener:
[ x ] Listado de usuarios (users.txt)
[x] Dominio (contoso.com)

La máquina Forest toca esta vulnerabilidad
```bash
GetNPUsers.py contoso.com/ -no-pass -usersfile users.txt
```
{: .nolineno }
