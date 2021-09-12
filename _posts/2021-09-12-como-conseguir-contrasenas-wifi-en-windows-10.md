---
layout: single
title:  Como conseguir contraseñas wifi en Windows 10
excerpt: "En este post te voy a explicar cómo conseguir las contraseñas wifi almacenadas en un dispositivo que utiliza como sistema operativo Windows 10, y cómo utilizar Python para automatizarlo"
date: 2021-09-12
classes: wide
header:
    teaser: /assets/images/como-conseguir-contrasenas-wifi-en-windows-10/portada.jpg
    teaser_home_page: true
categories:
    - Tutoriales
tags:
    - Tutoriales
    - Passwords
    - Python
    - Windows 10
---
<p align="center">
<img src="/assets/images/como-conseguir-contrasenas-wifi-en-windows-10/portada.jpg" width="50%">
</p>

**Aviso:** Este post se ha realizado únicamente con fines educativos. No me hago responsable de las malas acciones que puedan realizar terceras personas, fruto del conocimiento transmitido.
{: .notice--danger}

Teniendo acceso a un dispositivo con Windows 10, es bastante sencillo obtener las contraseñas WiFi almacenadas en sus previas conexiones. Para conseguirlas, nos basaremos en el comando `netsh`.

## Obteniendo contraseñas WiFi desde la consola
Para empezar, vamos a listar las ID de las conexiones WiFi a las que el dispositivo ha accedido anteriormente y que siguen "almacenadas" dentro de él. Para ello, usaremos el siguiente comando dentro de la consola:

```bash
netsh wlan show profiles
```

Ahora veremos listadas las conexiones wifi a las que hemos accedido. En mi caso son las siguientes:

<p align="center">
<img src="/assets/images/como-conseguir-contrasenas-wifi-en-windows-10/comando-1.jpg" width="50%">
</p>

Ahora trataremos de obtener algo más de información sobre alguna de ellas. en este caso utilizaré la red WiFi a la que me encuentro conectado. Para ello utilizaremos el comando anterior añadiendo el ID de la red WiFi sobre la que queremos obtener información:

```bash
netsh wlan show profile MOVISTAR_PLUS_5905
```

<p align="center">
<img src="/assets/images/como-conseguir-contrasenas-wifi-en-windows-10/comando-2.jpg" width="50%">
</p>

Ahora podremos ver información general, ajustes de conectividad, de seguridad y de coste. En este punto nos interesa un valor en especial: "Security Key". Como vemos, nos indica que está presente, eso quiere decir que disponemos de la clave de seguridad.
Queremos obtenerla ¿no? Para ello, le añadiremos al comando anterior `key=clear`

```bash
netsh wlan show profile MOVISTAR_PLUS_5905 key=clear
```

<p align="center">
<img src="/assets/images/como-conseguir-contrasenas-wifi-en-windows-10/comando-3.jpg" width="50%">
</p>

Así de simple hemos podido obtener la contraseña, y esto mismo funcionará para el resto de IDs. Si buscásemos obtenerlas todas y hubieran 50 IDs diferentes, tardaríamos una eternidad y además sería bastante tedioso, así que... ¿Por qué no automatizarlo?

## Automatizando la obtención de contraseñas WiFi mediante Python
Sin duda alguna Python es de las mejores opciones para hacer scripts de forma rápida y automatizar la mayoría de cosas que se nos hacen tediosas. El objetivo de este post no es enseñar a programar, es por ello que explicaré muy por encima lo que hace el código. Notad que utilizaremos los comandos anteriormente mencionados haciendo uso del módulo "subprocess":

Empezaremos con los módulos necesarios.
```python
import subprocess
import re
```
El módulo "subprocess" nos permitirá usar comandos del sistema y el módulo "re" nos permitirá usar expresiones regulares para extraer información de los resultados obtenidos mediante los comandos de consola.

```python
#...
output = subprocess.run(["netsh", "wlan", "show", "profiles"], capture_output = True).stdout.decode()
names = (re.findall("All User Profile     : (.*)\r", output))

list = []
```
En estas líneas de código hemos creado una variable (output) que almacena el resultado del comando `netsh wlan show profiles` y otra variable en la que almacenamos todas las IDs de obtenidas en "output". Luego de esto creamos una lista para almacenar posteriormente la información.

```python
#...
if len(names) != 0:
    for name in names:
        wifi_profile = {}
        pinfo = subprocess.run(["netsh", "wlan", "show", "profile", name], capture_output = True).stdout.decode()
        if re.search("Security key           : Absent", pinfo):
            continue
        else:
            wifi_profile["ssid"] = name
            pinfo_pswd = subprocess.run(["netsh", "wlan", "show", "profile", name, "key=clear"], capture_output = True).stdout.decode()
            pswd = re.search("Key Content            : (.*)\r", pinfo_pswd)
            if pswd == None:
                wifi_profile["password"] = None
            else:
                wifi_profile["password"] = pswd[1]
            list.append(wifi_profile)
```
Mediante estas lineas de código comprobamos que existe al menos 1 ID almacenada. Si es así, mediante un bucle buscamos aquellas que tienen almacenada la contraseña ("Security Key : Present") y la conseguimos añadiéndola posteriormente a una lista. Las IDs que no tienen la contraseña guardada las descartamos. Ahora solo nos queda mostrarlas en pantalla

```python
for x in range(len(list)):
    print(list[x])
```
Recorremos la lista y la mostramos en pantalla. Así hemos automatizado la obtención de todas las contraseñas (que en mi caso son únicamente 3). Vamos a ejecutarlo y comprobar que funciona:

<p align="center">
<img src="/assets/images/como-conseguir-contrasenas-wifi-en-windows-10/script.jpg" width="50%">
</p>

Espero que te haya sido útil este post, recuerda utilizar esta información de forma adecuada. Puedes seguirme en mis redes sociales: [Instagram](https://www.instagram.com/notaboutfran/) y [Twitter](https://twitter.com/notaboutfran).