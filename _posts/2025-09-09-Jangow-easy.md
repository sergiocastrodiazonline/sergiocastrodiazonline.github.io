---
title : "Writeup - Jangow Vulnhub | Sergio Castro"
author: Sergi Castro
date: 2025-09-09
categories: [Vulnhub, Vulnhub-Linux, Vulnhub-Easy]
tags: [nmap,exploits,suid]
---


[Link Máquina](https://www.vulnhub.com/entry/jangow-101,754/)

# Contenido

- [Indice](#indice)

- [Introducción](#Introducción)

- [Resumen General de Conceptos](#resumen-general-de-conceptos)

- [Primer Vistazo a la Máquina](#primer-vistazo-a-la-máquina)
  
- [Enumeración de los Puertos](#enumeración-de-los-puertos)

- [Análisis de Aplicación Web](#analisis-de-aplicación-web)

- [Fuzzing de Directorios](#fuzzing-de-directorios)

- [Escalado de Privilegios](#escalado-de-privilegios)

- [Conclusiones](#conclusiones)


## Introducción

Esta es mi primera resolución de CTF, Máquina o Aplicación vulnerable
para Github y Linkedin, y tal vez no se entienda muy bien o haya faltas
ortográficas, cualquier cosa escribirme en alguna de las dos
plataformas.

En cualquier caso, saludos y espero disfrutéis de la guía/lectura.

## Resumen General de Conceptos

-   Nmap y varios de sus tipos de escaneo (-sS y -sV).

-   View Source en la web para encontrar rutas desconocidas

-   Vulnerabilidad Local File Inclusion y como usarla para extraer el
    contenido ficheros del sistema operativo.

-   Métodos de Escalamiento sencillos como Shell reversas de PHP, abrir
    conexiones traseras con Netcat.

-   Aprendizaje de los binarios SUID utilizados para escalar privilegios
    en distribuciones Linux.

## Primer Vistazo a la Máquina

La máquina es de nivel fácil y está disponible en Vulnhub para su
descarga, al ser de nivel fácil nos podremos dar cuenta que no tendrá
mucha complejidad, esta máquina nos da varias pistas entre ellas la
técnica de la enumeración donde podremos incluir puertos u otra
información útil para la obtención de la FLAG, el cual es el objetivo
final de esta máquina.

Para empezar, veremos un poco que nos aparece si la iniciamos, si es un
sistema operativo Linux, Windows etc.

<img width="886" height="664" alt="image" src="https://github.com/user-attachments/assets/713c94c2-3499-465e-8ad1-4a598672ff2b" />


Como podemos nos da la IP de la máquina y vemos además que nos pide un
acceso donde no conocemos ni el usuario y la contraseña, podríamos
suponer que el usuario podría ser jangow01 pero normalmente no sería
efectivo.

Usaremos una técnica que siempre debemos utilizar, nada más tener una
máquina en la red disponible, la enumeración de puertos para encontrar
algún posible puerto, en este caso la haremos con Nmap, una herramienta
muy utilizada para la enumeración de los puertos u otros detalles de la
máquina, ya está incluida en Kali Linux, la distribución que utilizaré
sin embargo podemos usar cualquier otra que queramos.

## Enumeración de los Puertos

Vamos a realizar la enumeración con el siguiente comando:

<img width="485" height="46" alt="image" src="https://github.com/user-attachments/assets/71fdf2c7-5d22-42b8-9654-7446a9aa4d31" />


Explico los dos flags para que no solo se demuestre que esto es un copia
y pega de otras guías si no que se ha hecho un aprendizaje de lo que se
está utilizando (esto se hará cada vez que haya algo así).

- 

**sS:** Se usa para utilizar el \"escaneo SYN\" o \"escaneo sigiloso\"
    (Stealth Scan). En lugar de completar la conexión TCP de tres vías,
    Nmap envía un paquete SYN y espera la respuesta. Si recibe un
    SYN/ACK, significa que el puerto está abierto. Luego, Nmap envía un
    RST (reset) para cerrar la conexión antes de que se establezca por
    completo, esto es menos detectable IDS y evita dejar registros
    completos de conexión en el servidor objetivo.

-   

**sV:** Se usa para la \"detección de versiones\" (Version Detection).
    Una vez que Nmap ha identificado qué puertos están abiertos, utiliza
    este flag para determinar qué servicio y qué versión se está
    ejecutando en cada uno de ellos. Esto es muy útil para identificar
    vulnerabilidades conocidas asociadas a versiones específicas de
    software.

Una vez explicado seguimos, una vez lo aplicamos vemos lo siguiente:

<img width="789" height="271" alt="image" src="https://github.com/user-attachments/assets/ef9a43bc-4fb1-45f3-8636-79926d4ea6a2" />


Como vemos tenemos dos puertos activos tras el escaneo, explico ambos:

-   21 que por defecto es el puerto de cliente del servicio FTP
    (servicio muy utilizado para compartir información, podemos intentar
    acceder a el usando nuestro servicio ssh de nuestra maquina usando
    un usuario Anonymous, sin embargo, también cabe la posibilidad de
    que tenga contraseña.

<img width="598" height="195" alt="image" src="https://github.com/user-attachments/assets/1a295912-9b05-40d3-b4fd-f98e925bafed" />


Como vemos no tenemos la password por lo que es poco útil.

-   80: Puerto del servicio HTTP por lo que indica una página web
    establecida en la máquina. Vamos a intentar acceder por un navegador
    a ver que encontramos.

## Análisis de Aplicación Web

<img width="886" height="555" alt="image" src="https://github.com/user-attachments/assets/08ae75c4-28d8-4664-b7c2-78115e0a7306" />


Vemos que al entrar a la web (no hace falta indicar el puerto porque es
el por defecto). Vemos que tenemos un directorio abierto por lo que no
existe un archivo index en general. Lo que tenemos es un directorio
site, veamos que pasa si lo pulsamos.

<img width="886" height="543" alt="image" src="https://github.com/user-attachments/assets/b73b5771-6115-4aa7-9362-e8a554fb0294" />


Vemos que tenemos esta aplicación web donde tenemos una plantilla en
Bootstrap, digamos que es una especie de blog. La clave aquí está en
buscar alguna vulnerabilidad, podemos intentar usar el panel de Buscar
ya que suele ser clave para vulnerabilidades XSS (injección de scripts
etc).

Sin embargo, otra estrategia puede ser mirar el **Inspeccionar Elementos
o View Source** de la web ya que esta herramienta interna da acceso a
veces a recursos que a simple vista no se ven. Probemos a ver que vemos
con esto:

<img width="886" height="554" alt="image" src="https://github.com/user-attachments/assets/1836cd2b-c98a-4da5-880f-5da155b70820" />


Nos percatamos de algo curioso, gracias a un error del usuario
identificamos que el botón de búsqueda usa el lenguaje PHP (fichero
busque.php) para hacer las consultas y además nos da información del
parámetro utilizado en la consulta: ?buscar='

Esto es el comienzo de un parámetro en PHP. La visualización de esos
parámetros es normal en consultas GET que son aquellas que se ven en la
URL, mientras que POST se envían en solicitudes del header HTTP y no se
deben ver.

Si probamos a usar este php en la barra de búsqueda enviaremos una
petición GET para traer las búsquedas del blog.

<img width="886" height="173" alt="image" src="https://github.com/user-attachments/assets/e24dc3bc-e583-4863-8288-f32945ce319f" />


Como vemos al buscar trae una especie de lista, hay una vulnerabilidad
muy utilizada contra este tipo de sistemas llamada **LFI (Local File
Inclusion).**

Esta vulnerabilidad permite a los atacantes manipular ficheros
arbitrarios manipulando los puntos de entrada, en este caso podríamos
manipular el parámetro buscar= para añadir un comando de Linux, sabiendo
que usamos este tipo de sistema operativo, con esto tenemos este
resultado:

## Fuzzing de Directorios

<img width="886" height="217" alt="image" src="https://github.com/user-attachments/assets/20a9c079-7c0e-4d6c-91c3-c71c3fb8411b" />


Podemos listar todos los ficheros del directorio web y con ello vemos el
directorio wordpress, el cual estaba oculto, veamos que fichero pueden
contener.

<img width="886" height="255" alt="image" src="https://github.com/user-attachments/assets/4df911ca-4996-4a76-9908-e17065b88e15" />


Vemos que contiene el index.html y un script en PHP llamado config.php,
este script se suele utilizar para la configuración de wordpress y es un
fichero que normalmente debería estar capado tanto de visualización
tanto fuera como del contenido, pero vemos que los permisos están de
lectura para todo el mundo por lo que podremos verlo y con ello
encontramos lo siguiente:

<img width="886" height="102" alt="image" src="https://github.com/user-attachments/assets/609c0a62-4250-4f8b-ba8c-d545c0c495d4" />


Nos devuelve un error de conexión porque requiere la contraseña del
sistema como tal dándonos un usuario pero una contraseña necesaria, pero
como la web es vulnerable a LFI podremos hacer un cat y mostrar el
contenido saltándonos este paso.

<img width="886" height="291" alt="image" src="https://github.com/user-attachments/assets/51efa7d3-ffa0-4e88-9261-8061d05b9d59" />


Este fichero nos ha proporcionado la clave para el servidor SQL que
corre detrás de la máquina, sin embargo, esto no nos sirve para la
contraseña general de acceso ni para FTP. Tendremos que buscar otras
credenciales, investigando recordé que el directorio de Linux general
para los ficheros que cambian constantemente y el lugar donde suele
estar las páginas web Apache (en `/var/www/html`) y dentro contiene la
web, podemos mirar este directorio ya que a veces contiene otro tipo de
ficheros aparte del wordpress. Con esto observamos:

<img width="886" height="372" alt="image" src="https://github.com/user-attachments/assets/58260dd8-e175-4509-9198-230fd785e8b8" />


Accedemos a www/html

<img width="886" height="193" alt="image" src="https://github.com/user-attachments/assets/5e8be96f-efe2-44a7-ac8e-3f436b722743" />


Averiguamos un fichero .backup (de copias de seguridad) en la ruta de
nuestro site oculto a simple vista pero gracias a LFI podemos ver que
está dentro del sistema. Con ello si lo abrimos:

<img width="886" height="258" alt="image" src="https://github.com/user-attachments/assets/7b652b01-4fe6-4159-ac50-43a87bcd5c45" />


Nos proporciona la clave y el usuario jangow01 de SQL, si seguimos una
cierta lógica podríamos entender que podría incluso servirnos para
conectarnos a la propia máquina usando FTP (ya abierto en el 21) como
ese usuario (jangow01).

Para conectarnos podemos usar SSH o bien la propia máquina. Lo haré con
la propia maquina ya que SSH está desactivado.

También hemos aplicado un análisis con Hydra donde pasando el usuario
con -l y password con -p comprobamos que en efecto es la contraseña del
usuario.

<img width="886" height="370" alt="image" src="https://github.com/user-attachments/assets/89d7ab6e-8eef-4fa7-93b5-cf8527817dac" />


Y con esto accedemos con FTP y comprobamos en efecto que estamos dentro.

<img width="600" height="254" alt="image" src="https://github.com/user-attachments/assets/62e26e42-f3a9-4a38-b507-ab7631189284" />


Antes también comprobamos que jangow01 en efecto es el usuario de la
máquina.

<img width="886" height="224" alt="image" src="https://github.com/user-attachments/assets/74b7bf1f-9337-488e-843f-31e205c7298c" />


<img width="798" height="429" alt="image" src="https://github.com/user-attachments/assets/815bac4c-3e8b-4000-af44-bee1035ce3b3" />


Con esto obtenemos la primera flag que está dentro del usuario.

Sin embargo, aún queda hacernos root y escalar privilegios en la
máquina, con ello pasamos a la siguiente parte.

## Escalado de Privilegios

Podemos realizarlo de varias formas:

-   PHP Shell Reversa: Podemos subir una Shell que hagamos nosotros o
    encontrada en internet y que el servidor ejecute la misma al usar
    LFI.

<img width="886" height="453" alt="image" src="https://github.com/user-attachments/assets/f15bbfb4-5599-4fd2-9303-a319e6aa5682" />


Descargando por ejemplo esta podríamos cambiar la IP y el puerto para
realizar la conexión, sin embargo, no tenemos suficientes permisos como
jangow para subirla, como podemos ver aquí.

<img width="873" height="471" alt="image" src="https://github.com/user-attachments/assets/653420f3-be14-4793-8862-6493982c68a0" />


No puede crear el fichero.

Otra posible solución es usar Netcat para crear la conexión reversa, sin
embargo tampoco funcionará ya que los permisos no son suficientes.

Finalmente investigando encontré lo que son los SUID, algo muy importante en la seguridad de sistemas Linux:

-   S**UID** significa **Set User ID**.

-   Es un **bit de permiso especial en Linux/Unix** que se aplica a
    **archivos ejecutables**.

-   Cuando un binario tiene el bit SUID activado, significa que **se
    ejecuta con los permisos del propietario del archivo**, no con los
    del usuario que lo ejecuta.

-   Normalmente se usa en binarios que necesitan permisos elevados para
    realizar tareas específicas.

Algunos son el etc/passwd sin embargo el que nos interesa para escalar
de privilegios es el pkexec. Para listar los binarios usamos el
siguiente comando:

find / -perm -4000 -type f 2\>/dev/null

-   -perm -4000 → busca archivos con el bit SUID.

-   -type f → solo archivos normales (no directorios).

-   2\>/dev/null → ignora mensajes de error por falta de permisos en
    directorios.

<img width="875" height="525" alt="image" src="https://github.com/user-attachments/assets/6ff11f32-57b4-4099-a375-8e5983e36fe8" />

Veamos los permisos de dicho binario.

<img width="671" height="88" alt="image" src="https://github.com/user-attachments/assets/4542e3a9-255b-42cc-a684-a05617f277d3" />


Tiene los permisos suficientes.

Existe una vulnerabilidad descubierta y publicada en el 2021
(CVE-2021-4034) también llama Pwnkit.

<https://www.qualys.com/2022/01/25/cve-2021-4034/pwnkit.txt>

Basicamente la vulnerabilidad aprovecha los permisos de dicho binario
para que cualquier usuario no root pueda escalar privilegios en la
máquina. Para ello podemos y no serviremos del exploit ya desarrollado
por el usuario arthepsy en Github:

<https://github.com/arthepsy/CVE-2021-4034/blob/main/cve-2021-4034-poc.c>

Para poder compilarlo (ya que está hecho en C) me lo llevaré con FTP a
la máquina al directorio `/tmp` donde el usuario simple tiene permisos
para copiar y pegar ficheros.

<img width="886" height="353" alt="image" src="https://github.com/user-attachments/assets/68536466-423b-47ca-af5b-42e1a509e829" />


Metemos finalmente el exploit en la máquina objetivo, ahora lo
compilaremos desde la máquina objetivo ya comprometida.

<img width="886" height="338" alt="image" src="https://github.com/user-attachments/assets/32911a2f-558a-4479-ac00-b128879481ec" />


Finalmente tenemos nuestro exploit y ya solo queda ejecutarlo.

<img width="708" height="141" alt="image" src="https://github.com/user-attachments/assets/41a699da-6a26-4ec4-ae4c-542aed80b52f" />


Finalmente somos el usuario root escalando privilegios de la máquina y
ahora podemos acceder a su directorio para obtener la flag final.

<img width="886" height="644" alt="image" src="https://github.com/user-attachments/assets/8fb2d996-cad1-4a89-8e23-99eeaabaa1bc" />


## Conclusiones

Finalmente he terminado mi primer writeup de máquinas vulnerables,
realmente no fue tan complicado (es de nivel fácil), sin embargo, se
complicó un poco por el tema del locale en portugués de la máquina
remota, existen más formas de poder resolver esta máquina sin embargo
esta la he visto mucho más interesante ya que me ha permitido aprender
sobre algunos conceptos sencillos (resumen arriba en la parte superior).

Me despido y nos vemos en el siguiente.