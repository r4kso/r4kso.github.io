---
layout: single
title:  Crear un backdoor en un PDF (troyano)
excerpt: "En este post te explico cómo crear un backdoor en un archivo PDF para Windows (troyanizar un PDF)"
date: 2021-04-19
classes: wide
header:
    teaser: /assets/images/que-es-un-socket-y-como-funciona/portada.png
    teaser_home_page: true
categories:
    - Explicaciones
tags:
    - Tutoriales
    - Intrusión
    - Troyano
---
<p align="center">
<img src="/assets/images/que-es-un-socket-y-como-funciona/portada.png" width="50%">
</p>

Existen diferentes métodos para crear un *backdoor* en un PDF. El único que he usado (y que posiblemente sea el más sencillo) es mediante Metasploit donde haremos una inyección en un documento ya existente, cuyo backdoor será de tipo `.exe` (por lo que el SO de la víctima deberá ser Windows). Aunque también tiene un hándicap, y es que **sólo funcionará si el PDF es leído con Adobe PDF Reader**.

> *Backdoor* (puerta trasera) se refiere a una secuencia o código escondido detrás de otro para evitar sistemas de seguridad y acceder al sistema.

Bien, pues el exploit que usaremos será `adobe_pdf_embedded_exe`, así que accedemos a metasploit mediante el comando `msfconsole` y mostramos las opciones que nos ofrece.

[INSERTAR IMAGEN]

Primero cambiaremos el nombre del archivo en `FILENAME`. Yo lo llamaré `pdf_with_backdoor.pdf`

[INSERTAR IMAGEN]

Ahora, indicamos en `INFILENAME` la dirección del PDF al que queremos inyectarle el .exe, que en mi caso es la siguiente:

[INSERTAR IMAGEN]

También, cambiamos la configuración de `LAUNCH_MESSAGE`. Este será un mensaje que aparecerá al abrir el PDF, donde la víctima deberá clickar en 'open' para que se pueda ejecutar el .exe y generar así el backdoor. Yo pondré el siguiente mensaje para tratar de engañar al usuario (ya que lógicamente no le vamos a decir: "Eh! si le das a OPEN voy a entrar en tu sistema):

[INSERTAR IMAGEN]

Por último configuramos la dirección IP donde queremos que se envíe la información que consigamos mediante el backdoor (en este caso mi sistema) y el puerto donde vamos a recibirlo: