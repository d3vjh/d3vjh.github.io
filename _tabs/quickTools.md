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

## Reconocimiento

### Nmap

Es una herramienta de escaneo de puertos que permite descubrir hosts en una red y determinar qué servicios están disponibles en esos hosts. Ayuda a recopilar información sobre la infraestructura de red y los sistemas objetivo.

Es importante tener permisos de Amdn, para así poder utilizar el parámetro `-Pn`
```bash
sudo nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn {ip_target} -oG allPorts
```

Una vez tenemos los puertos, es importante obtener la versión y el servicio que se estan ejecutando en los puertos anteriormente encontrados
```bash
nmap -sCV -p{ports} {ip_target} -oN target
```

### Gobuster


```bash
gobuster dir -u {url} -w {wordList} -t {Threads}
```

A nivel de Subdominios: 

```bash
gobuster vhost -u {url} -w {wordList} -t {Threads:20}
```

### Puertos y servicios

#### lsof
Puede ser útil en la fase de enumeración para identificar procesos en ejecución y los archivos asociados a ellos, lo que puede revelar información valiosa sobre el sistema objetivo.

```bash
lsof -i :{port_number}
lsof -i :80
```

#### netstat

Se utiliza para mostrar información y estadísticas relacionadas con las conexiones de red activas, los puertos abiertos y otros datos de red.

```bash
netstat -tulnp
netstat -nat
```

|Parámetro|Función                                    |
|:-------:|:-----------------------------------------:|
| `-t`    |conexiones TCP establecidas                |
| `-u`    |conexiones UDP establecidas                |
| `-l`    |Puertos en escucha                         |
| `-n`    |Direcciones IP y puertos en números        |
| `-p`    |Proceso asociado a cada conexión o servicio|
| `-a`   |Todas las conexiones y puertos, incluyendo los que están en escucha y los que están establecidos|

### tcpdump
Para ponernos en escucha, en espera a recibir una traza icmp
```bash
tcpdump -i {interface} icmp -n
```

### Tratamiento de la TTY
```bash
script /dev/null -c bash
Ctrl + Z
stty raw -echo; fg
reset xterm
export TERM=xterm-256color
source /etc/skel/.bashrc
→ stty size 
stty rows (44) columns (184)
```


### Tareas Cron

Para ver que se está ejecutando a intervalos regulares de tiempo, podemos ejecutar este comando
```bash
systemctl list-timers
```

Este sirve para ver que está corriendo, y que usuario lo ha ejecutado

```bash
ps -eo user,command
```
### John The Ripper
```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hash
```

### watch
Para revisar el mismo comando cada $x$ tiempo:
```bash
watch -n {s} {command}
watch -n 1 ls -l /bin/bash
```

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

```bash
❯ searchsploit -x php/webapps/37637.pl
  Exploit: Elastix 2.2.0 - 'graph.php' Local File Inclusion
      URL: https://www.exploit-db.com/exploits/37637
     Path: /usr/share/exploitdb/exploits/php/webapps/37637.pl
    Codes: N/A
 Verified: True
File Type: ASCII text
```
