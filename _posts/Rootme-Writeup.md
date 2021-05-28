---
layout: single
title: TryHackMe - RootMe (Detallado)
excerpt: "En este write-up te explico de forma detallada cómo resolver Root Me de TryHackMe"
date: 2021-04-19
classes: wide
header:
    teaser: /assets/images/que-es-un-socket-y-como-funciona/portada.png
    teaser_home_page: true
categories:
    - Write-up
tags:
    - Write-up
    - TryHackMe
---
<p align="center">
<img src="/assets/images/que-es-un-socket-y-como-funciona/portada.png" width="50%">
</p>

## Despliegue de máquina
Desplegamos la máquina y preparamos los directorios para mantener una buena organización durante el ataque. Una buena metodología es clave para
En mi caso, creo una carpeta con el nombre de la máquina, y dentro de ella creo 4 directorios mediante la utilidad [mkt](https://pastebin.com/tQnf6v78) (by s4vitar)

Los directorios serán:
    - nmap: Donde guardaremos todo aquello necesario durante la fase de reconocimiento
    - content: Aquello que nos descarguemos de la máquina
    - exploits: Aquellos exploits que necesitemos usar
    - scripts: Pequeños scripts que tengamos que realizar


## Reconocimiento
Para la enumeración de puertos de esta máquina utilizaremos nmap con los siguientes parámetros (siendo IPMAQUINA la IP de la máquina):
`nmap -p- --open -T5 -v -oG allPorts IPMAQUINA -n`

Mediante estos parámetros buscaremos aquellos puertos que se encuentren abiertos y exportaremos la información obtenida al fichero "allPorts" en formato grepeable.
El resultado es el siguiente:

[INSERTAR IMAGEN]

Vemos que hay 2 puertos abiertos, 22 (SSH) y 80 (HTTP)

Ahora escaneamos servicios y versiones mediante nmap con los siguientes parámetros:
`nmap -p$(cat allPorts | grep -oP '\d{2,5}/open' | awk '{print $1}' FS="/" | xargs | tr ' ' ',') -sC -sV IPMAQUINA -oN targeted`

Los servicios se encontrarán listados en el fichero "targeted"