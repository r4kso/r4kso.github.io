---
layout: single
title:  Qué es un socket y cómo funciona
excerpt: "En este post te explico qué es un socket y cómo funciona, con un ejemplo práctico hecho en python (un echo server)"
date: 2021-04-16
classes: wide
header:
    teaser: /assets/images/que-es-un-socket-y-como-funciona/portada.png
    teaser_home_page: true
categories:
    - Explicaciones
tags:
    - Tutoriales
    - Redes
    - Python
    - Sockets
---
<p align="center">
<img src="/assets/images/que-es-un-socket-y-como-funciona/portada.png" width="50%">
</p>

Si tienes prisa, un ***socket*** se puede definir rápidamente como un **"dispositivo" virtual** generado por el sistema operativo a través del cual puedes **enviar y recibir información de otros procesos** que también se comuniquen mediante *sockets*.

Aunque claro, esta definición se queda un poco corta...
## ¿Qué es un socket? - Explicación larga
Podría hablarte de la historia y de por qué se crearon, pero si quieres saber eso te dejo este enlace por [aquí](https://dosideas.com/noticias/actualidad/574-pasado-presente-y-futuro-de-los-sockets). Lo que realmente queremos saber es *qué es un socket* y *cómo funciona*.

Pues bien, como dije anteriormente un socket es un """"""dispositivo""""" (nótese las comillas) virtual generado por el sistema operativo. ¿Qué quiero decir con esto? Pues que un *socket* realmente es un concepto abstracto que irá definido por la dirección IP y un puerto.

Un *socket* pertenece a un proceso y sirve como medio de comunicación entre otro proceso (sea que esté ejecutandose en la misma máquina u otra en tu misma red local o conectada a Internet). Esto nos permite intercambiar información. Para hacer esto posible existen algunos protocolos como TCP o UDP (en el ejemplo práctico usaremos TCP ya que es el más común).

> UDP (User Datagram Protocol) es un protocolo  **SIN CONEXION**, es decir, envía paquetes sin llegar a conectarse al destino para así maximizar la velocidad de transmisión de información (en este caso la información se envía en **datagramas**)

> TCP (Transmission Control Protocol) este protocolo es **CON CONEXION**, lo que garantiza que los paquetes serán entregados en su destino sin errores y en el mismo orden que en el que se emitieron.

<p align="center">
<img src="/assets/images/que-es-un-socket-y-como-funciona/img1.jpeg" width="100%">
</p>

En este esquema hay 2 ordenadores con su dirección IP propia (`174.123.241.13` y `142.250.134.127`). En cada ordenador se ejecuta un proceso (A y B) que hacen uso de los puertos `8000` y `5834` respectivamente. Como hemos dicho antes, cada socket será identificado por su IP y su puerto.

Así pues el socket A será: `174.123.241.13:8000` y el socket B será: `142.250.134.127:5834`.
## Ejemplo práctico - Echo server
Ahora que conocemos qué es un socket, vamos a ver cómo funcionan a en la práctica (lo haremos en python).

Un echo server consiste en mandarle una información al servidor y recibir a través de ese servidor lo mismo que tú has enviado (el equivalente al comando `echo` en una terminal linux).

Para ello, necesitaremos crear el servidor (socket A) y el cliente (socket B).
### SERVIDOR
Antes de nada, para mayor comodidad definiremos 2 constantes: `HOST` que almacenará la dirección IP que usaremos y `PORT` que almacenará el puerto que usaremos. Importamos previamente la libreria `socket` ya que posteriormente la necesitaremos.

```python
import socket

HOST = '127.0.0.1'      # Localhost
PORT = 5346
```
La dirección IP que aparece en HOST es la de *localhost*, ya que buscamos trabajar en local. El puerto escogido ha sido uno aleatorio. Existen un total de 65536 puertos, aunque los primeros 1024 se encuentran reservados para el uso del sistema. Estos pueden usarse aunque no es recomendable ya que podríamos interferir con algún otro proceso importante que haga uso del puerto usado.

Ahora, ya con la librería importada, crearemos un socket mediante la clase `socket`. Utilizamos `with` ya que posteriormente lo necesitaremos para seguir definiendo el código.
```python
#...
with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:    # A partir de ahora el socket será nombrado como "s"
    # Continuaremos el código a partir del "with"
```
Con el primer parámetro indicamos la **familia de dirección** (AddressFamily.INternET) que en este caso es IPv4 (si usaramos IPv6 deberíamos cambiarlo por `socket.AF_INET6`). Con el segundo parámetro indicamos el tipo de protocolo que usaremos para la transferencia de información, que en este caso será TCP.

Ahora llamamos a la función `bind` de la clase socket. Con esta función podremos indicar al socket a qué dirección IP pertenece y en qué puerto deberá estar escuchando.
```python
#...
s.bind((HOST, PORT))
```
Una vez que el socket ha sido configurado, hacemos que espere a la conexión de otro socket.
```python
#...
s.listen()
```
Esto hará que el programa se quede pausado en ese punto, hasta que otro socket se intente conectar, cumpliendo así las características de un servidor.

Cuando ocurra un intento de conexión, el servidor la acepta automáticamente mediante el método de la clase socket `accept()`. Este método devuelve **otro socket** (importante) que contiene la información socket que se ha conectado y también devuelve su dirección, que almacenaremos en `csocket` y `caddress` respectivamente.
```python
#...
(csocket, caddress) = s.accept()
```
Ahora que disponemos de la información del socket cliente, recibimos la información que nos envíe y se la reenviamos:
```python
#...
with csocket:
    print("Connected by : ", caddress)            # Imprime la dirección del cliente
    while True:
        data = csocket.recv(1024)                 # Lee los datos enviados por el cliente
        if not data:                                        
            break;                                # En caso de que no queden datos por leer, terminamos.    
        
        csocket.sendall(data);                    # Le devuelve los datos al cliente
```
En los comentarios se encuentra explicado qué hace línea, aunque viene bien resaltar que cuando llamamos a `recv()`, pasamos el parámetro `1024` para indicar el tamaño del buffer, es decir, la cantidad de bytes que puede leer. Si el mensaje fuera más grande, podríamos aumentar su tamaño.

El bucle está hecho para poder leer todos los datos que envíe el cliente. Una vez los haya leído todos, sale del bucle y finaliza. En este caso no hace falta cerrar el socket con `s.close()` ya que al finalizar el propio "with" este se cierra automáticamente gracias a que anteriormente creamos el socket mediante `socket.socket()`.

Pues con esto **hemos finalizado el código del servidor**. Crea el socket, lo configura, espera por una conexión, se conecta, recibe los datos y los envía. Un servidor en toda regla ;)

CÓDIGO COMPLETO:
```python
import socket

HOST = "127.0.0.1" # Localhost
PORT = 5346

with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:    # Parameters specify the address family and the socket type. Using "with" there is no need to call s.close()
    s.bind((HOST, PORT))                                        # Associate the socket to a specific network interface and port number
    s.listen()                                                  # Enable the socket to accept connectios. It makes it a listening socket.
    (csocket, caddress) = s.accept()                            # Blocks and waits for an incoming connection. Get a new socket object.
    with csocket:
        print("Connected by : ", caddress)                      # Prints the name of the client
        while True:
            data = csocket.recv(1024)                           # Read the data from the client
            if not data:                
                break;                                          # When there is no more data, we finish
            
            csocket.sendall(data);                              # Return the data to de client
```
PD: Los comentarios en inglés para cumplir con las buenas prácticas
### CLIENTE
El cliente es algo más sencillo que el servidor, ya que solo debemos conectarnos y enviar la información. Ahora que ya hemos hecho el servidor, este será pan comido.

En primer lugar importamos la librería socket y creamos las constantes. No necesitamos indicar el puerto y la IP del socket que vamos a crear ahora ya que no nos afecta en nada, será generado de forma aleatoria a conveniencia del sistema operativo. Lo que si necesitamos saber es la IP del servidor y su puerto, es por ello que las almacenamos en las constantes `SHOST` y `SPORT` respectivamente.
```python
import socket

SHOST = '127.0.0.1'      # Server IP
SPORT = 5346            # Server port
```
Creamos el socket al igual que hicimos en el servidor.
```python
#...
with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
    # Aquí continuará el código
```
Y ahora hacemos una solicitud de conexión con el servidor.
```python
#...
s.connect((SHOST, SPORT))
```
Una vez conectados, le enviamos los datos que queramos. Yo enviaré un simple "hello world".
```python
#...
s.sendall(b"Hello world! This is an echo server made with python")
```
Fijate en la b que se encuentra detrás de la cadena de texto que queremos enviar. Esto es para indicar que es un "byte string", que es el tipo de formato en el que debemos de enviar los datos, dado que la clase String internamente es un array de bytes. Otra forma de hacerlo sería guardando el string en una variable y codificarla mediante el método `encode()`.

Luego, guardamos en la variable `data` lo que nos envía el servidor (recuerda que el 1024 indica el tamaño del buffer).
```python
data = s.recv(1024)
```
Y ya por último, fuera del "with" imprimimos por pantalla los datos recibidos.
```python
#...
print('Data received by the server: ' + repr(data))
```
Fíjate en el método `repr()`. Lo que hace es convertir lo que le pasemos por parámetro en algo representable por la terminal. Necesitamos usarlo ya que los datos se encuentran en "byte string".

CODIGO COMPLETO:
```python
import socket

SHOST = '127.0.0.1'      # Localhost
SPORT = 5346

with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
    s.connect((SHOST, SPORT))
    s.sendall(b"Hello world! This is an echo server made with python.")
    data = s.recv(1024)

print('Received: ' + repr(data))
```
## Ejecutando el echo server
Para hacer funcionar el echo server, necesitamos ejecutar tanto el servidor como el cliente.

Primero ejecutamos el servidor. Yo utilizo VSCode por lo que lo abro desde la terminal del propio IDE y lo ejecuto como `python .\server.py` (o el nombre que le hayas dado al archivo de tu servidor):

<p align="center">
<img src="/assets/images/que-es-un-socket-y-como-funciona/img2.png" width="100%">
</p>

Como vemos, ya está ejecutándose

Ahora, abrimos otra pestaña de la terminal (el botón al lado del "+" en la parte superior derecha de la terminal) y ejecutamos `python .\client.py` (o el nombre que le hayas dado al archivo de tu cliente):

<p align="center">
<img src="/assets/images/que-es-un-socket-y-como-funciona/img3.png" width="100%">
</p>

Y ahora el servidor muestra:

<p align="center">
<img src="/assets/images/que-es-un-socket-y-como-funciona/img4.png" width="100%">
</p>

Como se puede ver, el resultado del cliente ha sido el esperado. El cliente se ha conectado desde la IP `127.0.0.1`, luego ha envíado `b"Hello world! This is an echo server made with python"`, el servidor lo ha recibido y lo ha reenviado de nuevo al cliente. Después, el cliente lo ha mostrado por pantalla y ambos han finalizado su ejecución.

Podemos ver el socket abierto **si dejamos el servidor ejecutándose**. Para ello, abrimos la terminal (en este caso Windows) y ejecutamos el comando `netstat -an`. Haciendo scroll hasta los sockets TCP, podemos ver el que hemos creado, que se encuentra en la IP `127.0.0.1` y en el puerto `5346`:

<p align="center">
<img src="/assets/images/que-es-un-socket-y-como-funciona/img5.png" width="80%">
</p>

Una vez finalicemos la ejecución del servidor, el socket se cerrará y ya no lo podremos ver en la lista.