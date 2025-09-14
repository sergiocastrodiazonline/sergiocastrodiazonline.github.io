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
- [Exploración de las Páginas Web](#exploración-de-las-páginas-web)
- [Listado de Ficheros Internos](#listado-de-ficheros-internos)
- [Desencriptando los Mensajes](#desencriptando-los-mensajes)
- [Conclusiones](#conclusiones)

## Introducción {#introduccion}

Hola buenas, traigo la solución de la máquina de Vulnhub llamada Earth que es parte de una serie de máquinas de temática espacial, 
esta de base nos proporciona dos pistas interesantes:

Tenemos dos flags, la del usuario normal y la del usuario root por lo que el objetivo es parecido a Jangow, comprometer el sistema y escalar privilegios.

Espero disfrutéis de la guía/lectura.

## Resumen General de Conceptos {#resumen-general-de-conceptos}

## Fase de Enumeración {#fase-de-enumeración}

Normalmente empezaremos enumerando los puertos, sin embargo esta vez contamos con una pequeña desventaja con respecto a la vez anterior, la máquina no nos proporciona la IP por lo que tendremos que descubrirla, podemos utilizar ARP para ver la tabla de conexiones pero existe una tool que simplifica el trabajo, esta permite descubrir los hosts conectados a una red por una interfaz x. Se llama netdiscover y para averiguar la IP podemos usarla así:

```
sudo netdiscover -r RED/MASCARA
```
-r: Es el parametro para indicar el escaneo a un direccion de red general y su máscara, en mi caso es máscara 24. 

Una vez realizado nos muestra los dispositivos conectados, gracias al saber que son máquinas sabemos que nuestro equipo vulnerable Earth es la dirección 104.

<img width="778" height="241" alt="image" src="https://github.com/user-attachments/assets/16beeed2-4fb7-46d8-b26a-c8f5224777fc" />

Aplicaremos ahora un nmap sobre esta IP para ver los puertos abiertos con el siguiente comando, donde he descubierto cosas interesantes para Nmap además de lo aprendido la otra vez.

```
nmap -sV -sC -p- -o earth.txt IP
```

Explico los nuevos parámetros:

-sC: Se le indica a nmap que use solo los scripts por defecto (ya que puedes añadir scripts extras).

-p-: Se especifica que se buscarán todos los puertos, si se quiere un rango usar -p 600-1000 por ejemplo del 6000 al 10000 o -p-1000 que permite escanear solo los primeros 1000 puertos.

-o: Manda la salida del comando a un fichero de salida, útil para repasar información ya obtenida o para otro software que permita usar estos datos para profundizar en análisis.

Una vez realizado vemos lo siguiente:

<img width="957" height="560" alt="image" src="https://github.com/user-attachments/assets/31c7cc62-bce8-46ab-9a78-ce4920530ddd" />

Observamos una gran cantidad de información de 3 puertos abiertos en específico:

- 22: SSH abierto donde nos muestra su versión y las claves hostkey

- 80: Puerto HTTP abierto, con su versión 2.4.5.1

- 443: Puerto HTTPs con una serie de datos de cabeceras y con dos datos interesantes de SSL, **dos hostname alternativos** llamados earth.local y terratest.earth.local

Sin estos hostname es totalmente imposible seguir ya que no podemos entrar a la web que contiene ya que necesita que el sistema que quiera entrar esté con dicho DNS, podemos ver como rechaza la conexión aquí:

<img width="1915" height="810" alt="image" src="https://github.com/user-attachments/assets/957adb4b-0a10-4ab7-b80a-5c885abc0415" />

Para arreglar esto añadimos los hostname a nuestro fichero hosts en etc usando en mi caso nano (podeís usar cualquier otro).

<img width="510" height="48" alt="image" src="https://github.com/user-attachments/assets/06478d71-511c-4020-a540-c581f5293761" />

Esto lo que hará es hacer que al conectarnos a earth.local, nos podamos conectar a la 104 desde SSL permitiendo visualizar la web que haya por atrás.

Una vez añadido nos vamos adentro de https://earth.local y veamos su contenido.

## Exploración de las Páginas Web {#exploración-de-las-páginas-web}

Abriendo earth.local, vemos esto:

<img width="1918" height="900" alt="image" src="https://github.com/user-attachments/assets/6c661708-96a1-4875-bb0b-4288ba7ad8aa" />

Vemos que nos aparece una web para mandar un mensaje a la Tierra y nos pide el mensaje y una clave para el mensaje, además podemos ver los anteriores mensajes encriptados.

Si abrimos la terratest, nos aparece lo siguiente:

<img width="1918" height="853" alt="image" src="https://github.com/user-attachments/assets/9d93e3c2-4276-4609-a595-ea778a1c1144" />

Nos dirá que está en pruebas y que ignoremos la web.

A simple vista no vemos nada en ninguna ni siquiera con el View Source

<img width="1455" height="540" alt="image" src="https://github.com/user-attachments/assets/0918b615-0033-4075-b4ef-53408d2ddf13" />

<img width="1252" height="237" alt="image" src="https://github.com/user-attachments/assets/3edaf341-21ea-4c7b-b030-3d150fe28f6e" />

## Listado de Ficheros Internos {#listado-de-ficheros-internos}

Existen varias formas de automatizar el proceso, investigando he encontrado la aplicación gobuster donde pasándole un simple diccionario de directorios es capaz de identificar de una web todo tipo de directorios que coincidan con el diccionario, al ser algo sencillo creo que puede ser de utilidad así que lo vamos a usar.

En mi caso usaré el small de Dirbuster (similar a gobuster) pero también puede servir el big o el medium

<img width="1180" height="622" alt="image" src="https://github.com/user-attachments/assets/94fc9af9-32fa-4c46-891d-f6a36560781f" />

Descargamos el fichero y una vez descargados aplicamos el gobuster sobre earth.local a ver que descubrimos:

```
gobuster dir -u https://earth.local -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-small.txt
```
<img width="1077" height="440" alt="image" src="https://github.com/user-attachments/assets/a72e5295-3bb8-4b1c-b9e0-9cc7159d8c3e" />

Una vez acabados podemos ver que ha encontrado un directorio oculto llamado /admin, si nos vamos a admin en la página nos aparecerá lo siguiente:

<img width="1736" height="352" alt="image" src="https://github.com/user-attachments/assets/df09158f-cc9b-4645-a2b5-6758c8883c6d" />

Nos aparecerá una herramienta de administración para usuarios admin, y si nos logeamos nos pedirá usuario y contraseña que no conocemos.

<img width="1503" height="396" alt="image" src="https://github.com/user-attachments/assets/865f9da1-18a7-4476-baaa-ec6dacd0f577" />

Vamos a intentar irnos al dominio terratest, que en principio no nos daba información, a ver si igualmente existen directorios ocultos con gobuster (esta vez con un diccionario de términos de web comunes como el .htaccess etc).

```
gobuster dir -u https://terratest.earth.local/ -k -w /usr/share/wordlists/dirb/common.txt
```

PD: El parámetro k es para evitar la verificación del certificado SSL (por lo que podemos usar https).

Una vez terminado la búsqueda tenemos:

<img width="871" height="448" alt="image" src="https://github.com/user-attachments/assets/befb7861-f4d7-4bcf-b0ba-27b9ba052c4f" />

Nos lista los directorios encontrados en el diccionario y que coincide con el contenido existente en la web, aquí vemos que ha localizado el fichero robots.txt usado para indicar lo que no debe ser indexado. Veamos que contiene:

<img width="1368" height="855" alt="image" src="https://github.com/user-attachments/assets/4bedb9b8-874e-4e70-90af-fd026fdef0c9" />

Vemos una serie de extensiones y un nombre interesante: testingnotes, sabiendo que puede ser cualquiera de esa extensiones y usando la lógica podemos pensar que hay un fichero más oculto dentro de la web con ese nombre y con la extensión txt ya que podría es algo que se entiende con el nombre de notas, probemos con esa extensión y veremos que sucede:

<img width="1187" height="377" alt="image" src="https://github.com/user-attachments/assets/1cfa6536-8770-4b66-b0fb-bc6e0a654e32" />

Descubrimos la nota y vemos una serie de notas con pistas:

- El usuario del panel de administración es terra
- testdata.txt es usado para probar el sistema de encriptación de los mensajes y la clave se encuentra hay dentro.
- Los mensajes son encriptados usando XOR.

Sabiendo estas pistas veamos la clave en el fichero testdata:

<img width="1918" height="315" alt="image" src="https://github.com/user-attachments/assets/c63dbc5c-93f9-42ce-8171-b8f87f8f40ba" />

Vemos la clave que es un mensaje relacionado con el espacio. Ahora lo que haremos será pasar a la desencriptación del mensaje de earth.local una vez conocido el sistema que utiliza.

## Desencriptando los Mensajes {#desencriptando-los-mensajes}

Para hacerlo podemos usar multitud de herramientas, sin embargo yo voy a usar una muy conocida llamada cyberchef, que contiene multitud de algoritmos para encriptar y desencriptar mensajes.



## Conclusiones {#conclusiones}
