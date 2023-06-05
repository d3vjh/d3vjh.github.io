---
# the default layout is 'page'
icon: fas fa-code
order: 4
---

El propósito de esta sección, es ir alimentandolo, a medida que encuentre comandos recurrentes e importantes, para así tener un acceso rápido a estos, junto a una pequeña descripción

## Nmap

Escanéo de puertos a una pc, es importante tener permisos de Amdn, para así poder utilizar el parámetro `-Pn`
```bash
sudo nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn {ip_target} -oG allPorts
```

Una vez tenemos los puertos, es importante obtener la versión y el servicio que se estan ejecutando en los puertos anteriormente encontrados
```bash
nmap -sCV -p{ports} {ip_target} -oN target
```

## Puertos y servicios

Para ver que servicio se está ejecutando en un puerto específico, puedes utilizar el comando 
```bash
lsof -i :{port_number}
lsof -i :80
```

Para ver la lista de las conexiones de red y servicios de ejecución en tu equipo
- `-t:` conexiones TCP establecidas
- `-u:` conexiones UDP establecidas
- `-l:` Puertos en escucha
- `-n:` Direcciones IP y puertos en números
- `-p:` Proceso asociado a cada conexión o servicio

- `-a:` Todas las conexiones y puertos, incluyendo los que están en escucha y los que están establecidos

```bash
netstat -tulnp
netstat -nat
```



## Github Pages
### Lista

- [x] Tarea 1
- [ ] Tarea 1

[comment]: También sirve para la identación de las letras, dependiendo los puntos: :-- :--: --:

### Tabla

|Columna1|Columna 2|
|:---|:---:|
|Prueba 1|Prueba 2|

### Bloques
> Hola, esto es un bloque informativo
{: .prompt-info }

> Hola, esto es un tip 
{: .prompt-tip }

> Hola, esto es un Danger
{: .prompt-danger }

> Hola, esto es un warning
{: .prompt-warning }

