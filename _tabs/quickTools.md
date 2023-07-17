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



