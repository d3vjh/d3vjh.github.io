---
title: Captura de tráfico HTTP
authors: [2, 1]
date: 2024-05-28
categories: [Cybersecurity, HTTP]
tags: [Traffic, Wireshark, HTTP]
image: /assets/img/tareas/captura-trafico/http.png
---

# Introducción

En nuestra actualidad, la interacción entre usuarios y sistemas de información ha evolucionado gracias a avances en la comunicación y arquitectura web. La capacidad de enviar y recibir datos de manera rápida ha revolucionado sectores como el comercio electrónico y la educación en línea. Este informe aborda conceptos fundamentales relacionados con el tráfico HTTP.

![Figura 1](/assets/img/tareas/captura-trafico/1.png)

# Conceptos Importantes

### Introducción al Protocolo HTTP

HTTP permite la transmisión de datos entre un cliente y un servidor. Funciona bajo un modelo de petición-respuesta, donde el cliente realiza una solicitud HTTP (GET, POST, etc.) y el servidor responde con un código de estado y el contenido solicitado.

### Códigos de Estado HTTP

- **200 OK**: Solicitud procesada correctamente.
- **302 Found**: Redirección temporal.
- **404 Not Found**: Recurso no encontrado.
- **500 Internal Server Error**: Error interno del servidor.

### Proceso de Conexión TCP

El establecimiento de una conexión TCP para HTTP sigue tres pasos clave conocidos como el "three-way handshake":
1. **SYN**: El cliente solicita la conexión.
2. **SYN-ACK**: El servidor confirma la solicitud.
3. **ACK**: El cliente confirma la conexión.

![Figura 2](/assets/img/tareas/captura-trafico/2.png)

## Propuesta de Solución

Para capturar y analizar tráfico HTTP, se creó una instancia de Ubuntu en AWS con tráfico HTTP habilitado. Se implementó un servidor HTTP utilizando Flask para generar diferentes códigos de estado. A continuación, se describen los pasos realizados:

1. Instalación de dependencias:
   ```bash
   sudo apt install python3 python3-pip
   pip install flask
   ```
2. Implementación del servidor Flask:
   ```python
   from flask import Flask, redirect

   app = Flask(__name__)

   @app.route('/success')
   def success():
       return "Success!", 200

   @app.route('/redirect')
   def redirect_example():
       return redirect("http://example.com/success", 302)

   @app.route('/not_found')
   def not_found():
       return "Page not found", 404

   @app.route('/server_error')
   def server_error():
       return "Internal Server Error", 500

   if __name__ == '__main__':
       app.run(host='0.0.0.0', port=8080)
   ```

![Figura 3](/assets/img/tareas/captura-trafico/3.png)

## Resultados

### Código 200

![Figura 4](/assets/img/tareas/captura-trafico/4.png)

### Código 302

![Figura 5](/assets/img/tareas/captura-trafico/5.png)

### Código 404

![Figura 6](/assets/img/tareas/captura-trafico/6.png)

### Código 500

![Figura 7](/assets/img/tareas/captura-trafico/7.png)

## Análisis de Flags SYN y ACK

El análisis de Wireshark mostró el proceso de handshake con las flags SYN, ACK y el intercambio de datos subsecuente.

![Figura 8](/assets/img/tareas/captura-trafico/8.png)

## Lectura de la Captura

### Parámetros Analizados

- **Tamaño de la ventana**: Máximo de datos que se pueden recibir antes de enviar un ACK.
- **Longitud máxima del segmento (MSS)**: Tamaño máximo de un segmento TCP sin fragmentación.

### Análisis Final

El análisis detalló que la máquina cliente (192.168.1.68) realizó solicitudes HTTP a un servidor Ubuntu (18.216.173.176) y recibió respuestas con códigos de estado específicos.

![Figura 9](/assets/img/tareas/captura-trafico/9.png)

