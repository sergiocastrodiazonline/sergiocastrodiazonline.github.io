---
title : "Writeup - Earth Vulnhub | Sergio Castro"
author: Sergi Castro
date: 2025-09-10
categories: [Vulnhub, Vulnhub-Linux, Vulnhub-Easy]
tags: [nmap,exploits,netdiscover,]
---

[Link Máquina](https://www.vulnhub.com/entry/the-planets-earth,755/)

# Contenido

- [Introducción](#introduccion)
- [Resumen General de Conceptos](#resumen-general-de-conceptos)
- [Fase de Enumeración](#fase-de-enumeración)
- [Conclusiones](#conclusiones)

## Introducción {#introduccion}

Hola buenas, traigo la solución de la máquina de Vulnhub llamada Earth que es parte de una serie de máquinas de temática espacial, 
esta de base nos proporciona dos pistas interesantes:

Tenemos dos flags, la del usuario normal y la del usuario root por lo que el objetivo es parecido a Jangow, comprometer el sistema y escalar privilegios.

Espero disfrutéis de la guía/lectura.

## Resumen General de Conceptos {#resumen-general-de-conceptos}

## Fase de Enumeración {#fase-de-enumeración}

Normalmente empezaremos enumerando los puertos, sin embargo esta vez contamos con una pequeña desventaja con respecto a la vez anterior, la máquina no nos proporciona la IP por lo que tendremos que descubrirla,
podemos utilizar ARP para ver la tabla de conexiones pero existe una tool que simplifica el trabajo, esta permite descubrir los hosts conectados a una red por una interfaz x. Se llama netdiscover y para averiguar la IP podemos usarla así:

```
sudo netdiscover -r RED/MASCARA
```
-r: Es el parametro para indicar el escaneo a un direccion de red general y su máscara, en mi caso es máscara 24. 

Una vez realizado nos muestra los dispositivos conectados, gracias al saber que son máquinas sabemos que nuestro equipo vulnerable Earth es la dirección 104.

<img width="778" height="241" alt="image" src="https://github.com/user-attachments/assets/16beeed2-4fb7-46d8-b26a-c8f5224777fc" />

Aplicaremos ahora un nmap sobre esta IP con el siguiente comando, donde he descubierto cosas interesantes para Nmap además de lo aprendido la otra vez.



## Conclusiones {#conclusiones}





