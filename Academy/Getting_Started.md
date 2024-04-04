# Academia: Getting Started

En este apartado abordaremos el primer apartado de la academia de HTB. Por tanto, comencemos con el apartado de la enumeración web:

## Enumeracion Web

Durante el escaneo de servicios, a menudo nos encontraremos con servidores web ejecutándose en los puertos 80 y 443. Los servidores web alojan aplicaciones web (a veces más de una)
que a menudo proporcionan una superficie de ataque considerable y un objetivo de alto valor durante una prueba de penetración. La enumeración web adecuada es fundamental,
especialmente cuando una organización no expone muchos servicios o esos servicios están parcheados adecuadamente.


### Gobuster

Después de descubrir una aplicación web, siempre vale la pena verificar si podemos descubrir archivos o directorios ocultos en el servidor web que no están destinados para acceso público.
Podemos usar una herramienta como ffuf o GoBuster para realizar esta enumeración de directorios.A veces encontraremos funcionalidades ocultas o páginas/directorios que exponen datos sensibles
que pueden ser aprovechados para acceder a la aplicación web o incluso ejecución de código remoto en el servidor web mismo.


### Enumeración de Directorios/Archivos

GoBuster es una herramienta versátil que permite realizar fuerza bruta en DNS, vhost y directorios. La herramienta tiene funcionalidades adicionales, como la enumeración de buckets AWS S3 públicos.
Para los propósitos de este módulo, estamos interesados en los modos de fuerza bruta de directorios (y archivos) especificados con el switch dir. Vamos a realizar un escaneo simple utilizando la lista
de palabras comunes dirb common.txt.

A continuación estableceré los comandos necesarios para ello y que información nos debe proporcionar, todas las secciones de código irán precedidas de una línea de símbolos de "$"

$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
dv16d$~ gobuster dir -u http://10.10.10.121/ -w /usr/share/dirb/wordlists/common.txt

**Explicación del comando:**
En este caso este comando que estamos ejecutando de Gobuster lo utilizamos para realizar una búsqueda recursiva sobre los directorios de un servidor web en la dirección que le específiquemos, en este caso
nos proporcionan una dirección de prueba sobre un wordpress y para esta búsqueda utilizamos un conjunto de palabras que proporciona el archivo common.txt.

Es importante también saber que dirb es una herramienta que nos proporciona un listado de directorios, esto nos sirve para hacer esa búsqueda recursiva (Es algo antigua y está poco actualizada).

git clone https://github.com/v0re/dirb.git  --> Nos clonará dirb


Con esto lo que conseguimos es evaluar posibles puntos de entrada o vulnerabilidades al servidor web.

*Salida del comando:*

/.hta (Status: 403)           // Código 403 --> Prohibido
/.htpasswd (Status: 403)      // Código 403 --> Prohibido
/.htaccess (Status: 403)      // Código 403 --> Prohibido
/index.php (Status: 200)      // Código 200 --> Éxito
/server-status (Status: 403)  // Código 403 --> Prohibido
/wordpress (Status: 301)      // Código 301 --> Redirección

$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$

Un código de estado HTTP 200 revela que la solicitud del recurso fue exitosa, mientras que un código de estado HTTP 403 indica que estamos prohibidos de acceder al recurso. Un código de estado 301 indica que
estamos siendo redirigidos, lo cual no es un caso de falla. Vale la pena familiarizarse con los diversos códigos de estado HTTP, que se pueden encontrar aquí. El módulo de la Academia de Solicitudes Web también
cubre los códigos de estado HTTP de manera más profunda.

El escaneo se completó con éxito e identifica una instalación de WordPress en /wordpress. WordPress es el CMS (Sistema de Gestión de Contenidos) más utilizado y tiene una enorme superficie de ataque potencial.
En este caso, al visitar http://10.10.10.121/wordpress en un navegador, se revela que WordPress aún está en modo de configuración, lo que nos permitirá obtener ejecución remota de código (RCE) en el servidor.

### DNS Subdomain Enumeration 
También puede haber recursos importantes alojados en subdominios, como paneles de administración o aplicaciones con funcionalidades adicionales que podrían ser explotadas.
Podemos usar GoBuster para enumerar los subdominios disponibles de un dominio dado utilizando la bandera dns para especificar el modo DNS. Primero, vamos a clonar el
repositorio de GitHub de SecLists, que contiene muchas listas útiles para fuzzing y explotación.


$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
dav16d$~ git clone https://github.com/danielmiessler/SecLists

**Explicación del comando:**
Realiza una clonación del directorio SecLists sobre el respositorio, esta clonación la haremos desde el directorio que nosostros deseemos en mi caso la estoy haciendo sobre el directorio raíz 
*Salida del comando:*


---------------------------------------------------------
dv16d$~ sudo apt install seclists -y

**Explicación del comando:**
Realizará la instalación de seclists, la opción -y significa que ante las preguntas que le realice la máquina al hacer la instalación pertinente esta responderá con sí a todo (Respuesta automatizada)

$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$


A continuación, agrega un Servidor DNS como 1.1.1.1 al archivo /etc/resolv.conf. Nos enfocaremos en el dominio inlanefreight.com, el sitio web de una empresa ficticia de transporte de carga y logística.
Si estas perdido a la hora de hacer esto te explico como se hace; en primer lugar abriremos el archivo resol.conf mediante algún editor de texto (nano, vi, etc...), en mi caso con vim, añadiremos las
siguientes lines a este archivo: (Primera Linea) nameserver 1.1.1.1  (Segunda linea) domain inlanefreight.com. A continuación guardamos el contenido del archivo y ya estaría especificado el DNS de 
nuestro sitio ficticio.

$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
dv16d$~ gobuster dns -d inlanefreight.com -w /usr/share/SecLists/Discovery/DNS/namelist.txt

**Explicación del comando:**
Con este comando estaríamos realizando un escaneo DNS sobre el dominio objetivo que sería el que específicamos después de la d, a continuación le proporcionaríamos una lista de palabras que están ubicadas
dentro de nuestro namelist.txt

*Salida del comando:*

$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$

Este tipo de escaneo nos permite averiguar los subdominios que pueden estudiarse repspectoa una dirección específica.

## Web Enumeration Tips

Ahora pasaremos a ver unos ciertos tips sobre web enumeration.

### Banner Grabbing / Web Server Headers

En la última sección, discutimos sobre la obtención de banners con propósitos generales. Los encabezados del servidor web proporcionan una buena imagen de lo que se aloja en un servidor web.
Pueden revelar el marco de aplicación específico que se utiliza, las opciones de autenticación y si el servidor carece de opciones de seguridad esenciales o ha sido mal configurado.
Podemos utilizar cURL para recuperar información de encabezado del servidor desde la línea de comandos. cURL es otra adición esencial a nuestra caja de herramientas de pruebas de penetración,
y se recomienda familiarizarse con sus muchas opciones.

$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
Dv16d$~ curl -IL https://www.inlanefreight.com

**Explicación del comando:**
Estamos "conectandonos" a una página web, dicha página web nos ha contestado de manera éxitosa, a su vez podemos saber sobre que servidor se sostiene dicha web entre otras cosas.

*Salida del comando:*
┌──(root㉿kali)-[/home/kali/Downloads]
└─# curl -IL https://www.inlanefreight.com
HTTP/1.1 200 OK
Date: Thu, 04 Apr 2024 14:28:01 GMT
Server: Apache/2.4.41 (Ubuntu)
Link: <https://www.inlanefreight.com/index.php/wp-json/>; rel="https://api.w.org/"
Link: <https://www.inlanefreight.com/index.php/wp-json/wp/v2/pages/7>; rel="alternate"; type="application/json"
Link: <https://www.inlanefreight.com/>; rel=shortlink
Content-Type: text/html; charset=UTF-8

$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$











