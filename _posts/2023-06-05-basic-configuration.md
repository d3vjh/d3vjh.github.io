---
title: Basic Configuration
author: d3vjh
date: 2023-06-05
categories: [Networks, Packet Tracer]
tags: [Router, Switch, DHCP, Server]
img_path: /assets/img/Networks/basicConfiguration/
image: packetTracer.png
---

# ¿Que es red?

Primero tenemos que entender como funciona las porciones de red/host, dentro de una red Lan o Wan

Vamos a iniciar con las clases C, debido a que son las más utilizadas a la hora de hablar de redes privadas, o de hogar, mejor conocidas como SOHO

Como máximo solo vamos a tener 254 hosts disponibles en una conexión.

![Light mode only](WPorciones.png){: .light }
![Dark mode only](DPorciones.png){: .dark }

El rango de las direcciones ip son

|  Clase  |    Inicio      |   Fin            | 
|:-------:|:--------------:|:----------------:|
|Class A  | 0.0.0.0        | 127.255.255.255  |     
|Class B  | 128.0.0.0      | 191.255.255.255  |     
|Class C  | 192.0.0.0      | 223.255.255.255  |     
|Class D  | 224.0.0.0      | 239.255.255.255  |     
|Class E  | 240.0.0.0      | 255.255.255.255  |     

Vamos a dividir los ejemplos así: `Porcion de Red`Porcion de Host

## Ejemplos

### Ayuda

Este es el rango que (IANA) ha reservado a direcciónes IP para redes privadas

|  Clase  |    Inicio      |   Fin            | Direciones Disponibles | 
|:-------:|:--------------:|:----------------:|:--:|
| Class A | `10`.0.0.0     | `10`.255.255.255 |16,777,214|
| Class B | `172.16`.0.0   | `172.31`.255.255 |65,534|
| Class C | `192.168.1`.0  | `192.168.1`.255  |254|


Teniendo esto claro, vamos a configurar los dispositivos

## Configuración

## Router1

Iniciamos con la parte `LAN`

![Router 1 Light](WRouter1.png){: .light }
![Router 1 Dark](DRouter1.png){: .dark }

Cambiamos el nombre

```python
Router>enable
Router#configure terminal
Router(config)#hostname Router1
Router1(config)#
```
{: .nolineno }

### Contraseñas

Asignamos contraseñas:

Al Modo privilegiado
```python
Router1(config)#enable secret p4$$w0rd
```
{: .nolineno }


Al modo no privilegiado

```python
Router1(config)#line console 0
Router1(config-line)#password pass
Router1(config-line)#login
Router1(config-line)#exit
Router1(config)#
```
{: .nolineno }

Y por último a las conexiones virtuales

```python
Router1(config)#line vty 0 15
Router1(config-line)#password pass
Router1(config-line)#login
Router1(config-line)#exit
```
{: .nolineno }

Es importante realizar la encripción de las contraseñas, para que no se guarden en texto claro

```python
Router1(config)#service password-encryption
```
{: .nolineno }

Ahora escribimos la configuración que tenemos en `running-config` en el `startup-config`

```python
Router1(config)#exit
Router1#write 
```
{: .nolineno }



### Redes

Ahora apagamos el Router, y le agregamos el módulo `HWIC-2T` el cual soporta conexiones seriales

![HWIC-2t](/assets/img/Networks/basicConfiguration/HWIC-2T.png)_HWIC-2T_

Y adicional, recomiendo agregar el módulo `HWIC-4ESW` el cual nos soporta 4 interfaces adicionales

![HWIC-4ESW](/assets/img/Networks/basicConfiguration/HWIC-4ESW.png)_HWIC-4ESW_

### Red Privada

Agregamos un Switch1 el cual conectaremos del `GigabitEthernet0/0` del router al `Switch1`
Utilizaremos una IP clase `C  192.168.0.0/24` 

```python
Router1(config)#interface gigabitEthernet 0/0
Router1(config-if)#ip address 192.168.0.1 255.255.255.0
ROuter1(config-if)#no shutdown
Router1(config-if)#end
```
{: .nolineno }


![Router Switch Light](WRouterSwitch.png){: .light }
![Router Switch Dark](DRouterSwitch.png){: .dark }


### Red Pública

Importante tener los puertos serial en ambos Switch

Vamos a utilizar una IP Clase `A  100.0.0.0` y vamos a conectarlos por el puerto `Se0/0/0` de cada uno


![Router Switch Light](WRouterSerial.png){: .light }
![Router Switch Dark](DRouterSerial.png){: .dark }

```python
Router1(config)#interface serial 0/0/0
Router1(config-if)#ip address 100.0.0.1 255.255.255.0
Router1(config-if)#no shutdown
Router1(config-if)#end
```

Es decir, hasta el momento llevamos nuestra topología asii

![Router Switch Light](WTopologia1.png){: .light }
![Router Switch Dark](DTopologia1.png){: .dark }


## Servidor

### DHCP

El servicio DHCP permite repartir IPs dentro de la red, de manera automática

1. Encendemos el servicio DHCP
2. Agregamos el Default Gateway (El Router) 192.168.0.1
3. Indicamos que va a iniciar a repartir desde la 20
4. Le indicamos la máscara de Sub Red 255.255.255.0
5. Establecemos un límite máximo de usuarios
6. Guardamos la configuración


![Router Switch Dark](DHCP.png)_DHCP Configuration_

## Topología Final

Agregamos dos computadores, y les establecemos que reciban la IP por DHCP

![Router Switch Dark](DHCPRequest.png)_DHCP Request_

Y finalizamos así

![Router Switch Light](WFinal.png){: .light }
![Router Switch Dark](DFinal.png){: .dark }



























> Post Under construction <br> I am currently working on this page
{: .prompt-warning}
