---
title: "Writeup - Tortuga | The Hacker Labs"
author: Sergi Castro
date: 2025-10-08
categories: [Pentesting, Escalado de Privilegios, Writeup]
tags: [nmap, hydra, linux, python, capabilities, gtfobins, ssh, suid]
---

[Link Máquina](https://labs.thehackerslabs.com/machine/131)

# Contenido
- [Introducción](#introduccion)
- [Escaneo de Red y Reconocimiento](#escaneo-de-red-y-reconocimiento)
- [Acceso Inicial con Hydra](#acceso-inicial-con-hydra)
- [Escalado Horizontal](#escalado-horizontal)
- [Escalado de Privilegios a Root](#escalado-de-privilegios-a-root)
- [Conclusiones](#conclusiones)
- [Resumen de Vulnerabilidades](#resumen-de-vulnerabilidades)

---

## Introducción {#introduccion}

El objetivo de esta máquina es obtener acceso al sistema principal y escalar los privilegios hasta nivel **root**, logrando así las dos flags ocultas en el entorno.

---

## Escaneo de Red y Reconocimiento {#escaneo-de-red-y-reconocimiento}

Se realiza un reconocimiento inicial mediante **nmap** para descubrir los servicios activos en la máquina objetivo:

```
nmap -p- sCV -Pn -n --min-rate 5000 <IPObjetivo>
```
-	Flag -p- :  Escanea todos los puertos disponibles (1-65535).
-	Flag sCV: Une el uso de -sC (scripts defecto) con -sV  (detección de versiones).
-	Flag n: No resuelve nombres DNS lo que lo hace más rápido.
-	Flag min-rate: Velocidad mínima de escaneo, en este caso 5000 paquetes por segundo.

<img width="886" height="395" alt="imagen" src="https://github.com/user-attachments/assets/612079af-0d13-4537-bb90-697152074350" />

-	22: OpenSSH versión 9.2p1 
-	80: Apache HTTPD 2.4.6.2, Título Isla Tortuga.

Navegaremos primero a la web, donde vemos la siguiente página:

<img width="886" height="363" alt="imagen" src="https://github.com/user-attachments/assets/f0f8217a-1e45-434c-8bca-4e270ffd7b79" />

A simple vista no vemos nada, pero si nos vamos a View Source Page e investigamos las páginas encontraremos alguna que otra pista.

<img width="886" height="506" alt="imagen" src="https://github.com/user-attachments/assets/40363af2-80fe-41e8-bcbe-bced1cb855a0" />

Nos marca un mapa de un tesoro enterrado y que nos han dejado una nota oculta en el camarote, pero especial atención porque nos revela que somos un grumete por lo que podemos entender que existe dicho usuario, el camarote lo podemos entender como su directorio personal del sistema.
Ahora bien, no tenemos mucha información sobre sus credenciales, pero usando Hydra como la vez anterior podemos intentar hacernos con el acceso SSH para así acceder a los directorios conociendo el usuario y una lista de palabras claves donde alguna sea el password.

## Acceso Inicial con Hydra {#acceso-inicial-con-hydra}

Conociendo el usuario grumete, se intenta un ataque de fuerza bruta a SSH usando Hydra:

```
hydra -l grumete -P /usr/share/wordlists/rockyou.txt ssh://<IPObjetivo>
```

Es similar a la de Red1 pero usando el rockyou esta vez, ya que es el diccionario más utilizado.

<img width="886" height="205" alt="imagen" src="https://github.com/user-attachments/assets/181c7cfc-7839-41e6-b250-65e8f897ab47" />

Finalmente nos da acceso a la contraseña que es 1234 (la más utilizada de rockyou). Esto nos permitirá acceder con los permisos del usuario grumete dentro 
de la máquina objetivo.

<img width="886" height="328" alt="imagen" src="https://github.com/user-attachments/assets/2db13493-c545-4154-8daa-5b6f6ab124a0" />

Si exploramos el directorio del usuario obtenemos la primera flag.

<img width="886" height="175" alt="imagen" src="https://github.com/user-attachments/assets/2084767a-c070-4c20-94e4-6699275f88e6" />

Ahora bien ya dentro de la máquina, podemos hacer un reconocimiento mucho más amplio de forma interna, normalmente no podríamos porque necesitaríamos permisos, sin embargo el usuario grumete tiene permisos de lectura del fichero /etc/passwd.

<img width="886" height="485" alt="imagen" src="https://github.com/user-attachments/assets/77a52406-5365-4ac6-afda-a14aecd5b9a3" />

Vemos que entre los usuarios tenemos otro temático que es el de capitán lo cual es otro dato a apuntar.

## Escalado Horizontal {#escalado-horizontal}

Listando los archivos ocultos con:

```
ls -la
```
Se encuentra una nota oculta en el “camarote”, escrita por el capitán. En ella, se revela la contraseña de su propio usuario.

<img width="886" height="316" alt="imagen" src="https://github.com/user-attachments/assets/5911282e-c339-43bf-82e5-aa7b081e9e76" />

Esto permite realizar un escalado horizontal, cambiando al usuario capitan sin aumentar privilegios globales, pero accediendo a nuevos recursos internos.

## Escalado de Privilegios a Root {#escalado-de-privilegios-a-root}

Ahora debemos escalar a root, podríamos listar los binarios SUID:

```
find / -perm -4000 2>/dev/null
```
<img width="886" height="297" alt="imagen" src="https://github.com/user-attachments/assets/01110b6c-af0a-40a7-803c-233028e305f3" />

Investigando estas SUID no permiten escalado per existe otra alternativa, que es buscar las capabilities de Linux asignadas a los binarios.

```
Una capability es un atributo de seguridad que permite una gestión de privilegios dividiendo las poderosas funciones del usuario root (superusuario) en unidades discretas e independientes. En lugar de darle a un proceso o archivo todos los permisos de root, se le asignan solo las capacidades específicas que necesita para realizar una tarea privilegiada.
```

Existe un comando que permite buscarlas (de forma similar a los SUID):

```
getcap -r / 2>/dev/null
```

