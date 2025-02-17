---
title: Recopilación Pasiva de información
author:
  name: Javier Fernández Santos
  link: https://github.com/JaviFS97
date: 2021-05-25 18:03:00 +0800
categories: [Ciberseguridad, Seguridad Ofensiva]
tags: [Pentesting, Google Hacking, Shodan, WHOIS, Archive.org, The Harverster, Maltego, Recon-ng]
math: true
mermaid: true
assets_path: /assets/img/posts/20210525

---
En esta sección se han recopilado algunas de las principales herramientas usadas durante la fase de recopilación pasiva de información.


# Índice
- [Objetivo](#objetivo)
- [Hacking con buscadores: Google Hacking](#hacking-con-buscadores-google-hacking)
  * [Comandos principales Google Hacking](#comandos-principales-google-hacking)
  * [Operadores Booleanos Google Hacking](#operadores-booleanos-google-hacking)
  * [Google Hacking Database](#google-hacking-database)
- [Shodan](#shodan)
  * [¿Cómo funciona?](#cómo-funciona)
  * [Comandos principales](#comandos-principales)
- [WHOIS](#whois)
- [Archive.org](#archiveorg)
  * [Caso de uso](#caso-de-uso)
- [The Harvester](#the-harvester)
  * [Caso de uso](#caso-de-uso-1)
  * [Comandos más importantes](#comandos-más-importantes)
- [Maltego](#maltego)
  * [¿Qué es un transformador?](#qué-es-un-transformador)
  * [¿Qué es un grafo?](#qué-es-un-grafo)
  * [Caso de uso](#caso-de-uso-2)
- [Recon-ng](#recon-ng)
  * [Caso de uso](#caso-de-uso-3)




# Objetivo
Obtener toda la información posible sobre nuestro objetivo **sin que las actividades realizadas por el analista sean mínimamente detectadas por dicho objetivo**.

Difícil de realizar y a menudo proporciona resultados poco concluyentes.

La manera habitual de recolección pasiva de información es mediante el acceso a la **información almacenada en lugares públicos**.

Raramente se utiliza de manera individual, normalmente se combina con recopilación semi-pasiva y activa. De esta manera podremos contrastar la información hallada.

# Hacking con buscadores: Google Hacking
Recopilaremos aquella información que haya sido indexada por el buscador.

## Comandos principales Google Hacking

A continuación se muestran los comandos principales que podemos utilizar con Google. Hay que tener en cuenta que todos ellos deben ir seguidos (sin espacios) de la consulta que quiere realizarse:


| Comando| Descripción |
|:----------------------:|--------------------------------------------------------------------------|
| `define:término`       |Se muestran definiciones procedentes de páginas web para el término buscado.|
| `filetype:término`     |Las búsquedas se restringen a páginas cuyos nombres acaben en el término especificado. Sobretodo se utiliza para determinar la extensión de los ficheros requeridos. Nota: el comando ext:término se usa de manera equivalente.|
| `site:sitio/dominio`   |Los resultados se restringen a los contenidos en el sitio o dominio especificado. Muy útil para realizar búsquedas en sitios que no tienen buscadores internos propios.|
| `link:url`             |Muestra páginas que apuntan a la definida por dicha url. La cantidad (y calidad) de los enlaces a una página determina su relevancia para los buscadores. Nota: sólo presenta aquellas páginas con pagerank 5 o más.|
| `cache:url`            |Se mostrará la versión de la página definida por url que Google tiene en su memoria, es decir, la copia que hizo el robot de Google la última vez que pasó por dicha página.|
| `info:url`             |Google presentará información sobre la página web que corresponde con la url.|
| `related:url`          | Google mostrará páginas similares a la que especifica la url.  Nota: Es difícil entender que tipo de relación tiene en cuenta Google para mostrar dichas páginas. Muchas veces carece de utilidad.|
| `allinanchor:términos` |Google restringe las búsquedas a aquellas páginas apuntadas por enlaces donde el texto contiene los términos buscados.|
| `inanchor:término`     |Las búsquedas se restringen a aquellas apuntadas por enlaces donde el texto contiene el término especificado. A diferencia de allinanchor se puede combinar con la búsqueda habitual.|
| `allintext:términos`   |Se restringen las búsquedas a los resultados que contienen los términos en el texto de la página.|
| `intext:término`       |Restringe los resultados a aquellos textos que contienen término en el texto. A diferencia de allintext se puede combinar con la búsqueda habitual de términos.|
| `allinurl:términos`    |Sólo se presentan los resultados que contienen los términos buscados en la url.|
| `inurl:término`        |Los resultados se restringen a aquellos que contienen término en la url. A diferencia de allinurl se puede combinar con la búsqueda habitual de términos.|
| `allintitle:términos`  |Restringe los resultados a aquellos que contienen los términos en el título.|
| `intitle:término`      |Restringe los resultados a aquellos documentos que contienen término en el título. A diferencia de allintitle se puede combinar con la búsqueda habitual de términos.|

## Operadores Booleanos Google Hacking

Google hace uso de los operadores booleanos para realizar búsquedas combinadas de varios términos. Esos operadores son una serie de símbolos que Google reconoce y modifican la búsqueda realizada:

| Operador | Descripción|
| :----------: |-------------|
| `" "`      |Busca las palabras exactas.| 
| `-`        |Excluye una palabra de la búsqueda. (Ej: gmail -hotmail, busca páginas en las que aparezca la palabra gmail y no aparezca la palabra hotmail).| 
| `OR ó \|` |Busca páginas que contengan un término u otro.| 
| `+`        |Permite incluir palabras que Google por defecto no tiene en cuenta al ser muy comunes (en español: "de", "el", "la".....). También se usa para que Google distinga acentos, diéresis y la letra ñ, que normalmente son elementos que no distingue.| 
| `*`        |Comodín. Utilizado para sustituir una palabra. Suele combinarse con el operador de literalidad (" ").| 

## Google Hacking Database
Es un [repositorio](https://www.exploit-db.com/google-hacking-database) donde se publican los denominados `Google Dork`. 
> Cada una de las consultas que tenemos en el apartado anterior se denominan `Google Dork`.

Hay que dejar claro que los Google Dorks van dejando de tener utilidad a medida que las empresas van descubriendo la información expuesta e indexada.



# Shodan
Es un motor de búsqueda que le permite al usuario encontrar información de equipos (routers, servidores, etc.) conectados a Internet a través de una variedad de filtros.

En lugar de indexar webs (como hace google), indexa los sistemas en base a los servicios que están corriendo en esos sistemas (concretamente al banner que nos proporcionan esos servicios).

## ¿Cómo funciona?
**Solicita conexiones a todas las direcciones IP imaginables en Internet** e indexando la información que obtiene de esas solicitudes de conexión. Rastrea la web en busca de dispositivos utilizando una red global de ordenadores y servidores que funcionan las 24 horas del día.

Realiza un escaneo de puertos. **Aquellos que estén abiertos responden con banners que contienen importantes metadatos** sobre los dispositivos a los que Shodan solicita una conexión.

Estos `banners` pueden proporcionar todo tipo de información de identificación, pero aquí están algunos de los campos más comunes que verá en un banner:
* Nombre del dispositivo: Cómo se llama el dispositivo en línea.
* Dirección IP.
* Número de puerto: El protocolo que utiliza su dispositivo para conectarse a la web.
* Organización: Qué empresa es la propietaria de tu "espacio IP".
* Ubicación.
* Algunos dispositivos incluso incluyen su nombre de usuario y contraseña por defecto, la marca y el modelo, y la versión del software, todo lo cual puede ser aprovechado por los hackers.  

También disponemos de la `herramienta SHODAN Diggity`, que permite realizar búsquedas automatizadas dentro de SHODAN, al cual se conecta a través de SHODAN API. Incorpora un repositorio de búsquedas habituales, denominado SHODAN Hacking Database.

## Comandos principales

| Operador | Descripción|
| :----------: |-------------|
|`after`|Only show results after the given date (dd/mm/yyyy) string|
|`asn`|Autonomous system number string|
|`before`|Only show results before the given date (dd/mm/yyyy) string|
|`category`|Available categories: ics, malwarestring|
|`city`|Name of the city string|
|`country`|2-letter country code string|
|`geo`|Accepts between 2 and 4 parameters. If 2 parameters: latitude, longitude. If 3 parameters: latitude, longitude, range. If 4 parameters: top left latitude, top left longitude, bottom right latitude, bottom right longitude.|
|`hash`|Hash of the data property integer|
|`has_ipv6`|True/False boolean|
|`has_screenshot`|True/False boolean|
|`server`|Devices or servers that contain a specific server header flag string|
|`hostname`| Full host name for the device string|
|`ip`|Alias for net filter string|
|`isp`|ISP managing the netblock string|
|`net`| Network range in CIDR notation (ex.199.4.1.0/24) string|
|`org`|Organization assigned the netblock string|
|`os`|Operating system string|
|`port`|Port number for the service integer|
|`postal`|Postal code (US-only) string|
|`product`|Name of the software/product providing the banner string|
|`region`|Name of the region/state string|
|`state`|Alias for region string|
|`version`|Version for the product string|
|`vuln`|CVE ID for a vulnerability string|

[Repositorio](https://github.com/jakejarvis/awesome-shodan-queries) con queries shodan bastante interesantes.

# WHOIS
WHOIS no es una aplicación como tal a la que vayamos a recurrir para obtener información sobre nuestro objetivo, sino que se trata de un protocolo TCP cuya misión es realizar búsquedas de información de dominios web, dentro de las bases de dataos de los organismos encargados de llevar a cabo el control sobre ellos.

Mediante este procedimiento podremos obtener datos tales como la persona que realizó la compra del dominio, la fecha en la que lo hizo, la fecha en la que expira la validez de la compra, los servidores DNS sobre los que está alojado, sus direcciones IP, incluso otros datos más personales como el número de teléfono y la dirección de correo electrónico de los administradores del sitio web.


# Archive.org
Es una biblioteca digital gestionada por una organización sin ánimo de lucro dedicada a la preservación de archivos, capturas de sitios públicos de la Web, recursos multimedia y también software.

## Caso de uso
El uso que se le puede dar es: recuperar información relevante que se haya modificado o eliminado del sitio web.
Por ejemplo, alguien sube a su repositorio de GitHub un proyecto y por error no elimina las credenciales. Cuando se dé cuenta, las elimina del proyecto, pero puede que estas hayan quedado capturadas en Archive.org.


# The Harvester
Herramienta muy simple de usar, pero poderosa y efectiva, diseñada para ser usada en las primeras fases del pentesting. Es usado para recopilar información de fuentes abiertas (OSINT). [Link a su repositorio](https://github.com/laramies/theHarvester).

La herramienta recopila correos electrónicos, nombres, subdominios, IPs y URLs utilizando múltiples fuentes de datos públicos como: google, baidu, linkedin, censys, shodan, github, etc.

## Caso de uso
Por ejemplo, si queremos lanzar un ataque de phising sobre los empleados de una cierta empresa, podemos hacer que recopile los correos electrónicos de sus trabajadores a través de una consulta por linkedin.

![The Harvester]({{ page.assets_path}}/TheHarvester.png){: .shadow style="min-width: 70%;" }
_Campaña OSINT con The Harvester_

Esta salida es la genérica, es decir, por terminal. Pero el parámetro `-f` permite exportar este resultado a un informe en HTML o XML.


## Comandos más importantes

| Parámetro| Descripción|
| :----------: |-------------|
|`-d`|Especifica el dominio sobre el que se va a realizar la búsqueda.|
|`-b`|Especifica la fuente de datos desde donde se van a obtener los resultados. Permite realizar consultas sobre motores de búsqueda, servidores PGP, redes sociales, etc. incluso sobre todas a la vez.|
|`-g`|Realiza la búsqueda dentro de los dorks de Google, en lugar de sobre el buscador normal.|
|`-f`|Permite exportar los resultados de la búsqueda a un informe en formato HTML o XML. **Opción muy interesante para ganar en claridad**.|
|`-l`|Limita el número de resultados a mostrar.|
|`-h`|Realiza la consulta sobre el motor de búsqueda SHODAN.|






# Maltego
Es una herramienta de inteligencia de código abierto (OSINT) y de análisis de enlaces gráficos para recopilar y conectar información para tareas de investigación.

## ¿Qué es un transformador?
Sirven para la recopilación OSINT de fuentes comunes en Internet, incluidas las consultas en servidores DNS, motores de búsqueda, redes sociales, diversas API y otras fuentes.

## ¿Qué es un grafo?
Es la estructura principal que tiene Maltego. Es un lienzo en blanco en el que nosotros podemos arrastrar `entidades`. Estas entidades representan distintos tipos de información sobre los que nosotros vamos a poder buscar en fuentes públicas aplicando transformadores (es decir, consultas).

## Caso de uso
Imaginemos que con las herramientas anteriores hemos conseguido el nombre y apellidos de un trabajador de una organización que estamos investigando.

Cogemos la `entidad "persona"` y la arrastramos al lienzo en blanco. Modificamos los atributos de la persona. Aplicamos los transformadores que queramos. Se irá generando un árbol con la información que va recopilando de cada transformador.

![Maltego grafo]({{ page.assets_path}}/Maltego.png){: .shadow style="min-width: 70%;" }
_Maltego grafo_


# Recon-ng

Permite realizar recolección de información y reconocimiento de redes de forma automatizada.

Basa su funcionamiento en el uso de diferentes módulos que ya vienen incluidos en la herramienta. Debiendo instalar aquellos que queramos usar por medio del marketplace. `> marketplace install nombre_module`.

Mediante el comando `> show modules` accedemos al listado de los módulos instalados. Están divididos en 5 tipos (Discovery, Explotation, Import, Recon y Reporting).

## Caso de uso
En primer lugar, debemos escoger qué modulo queremos usar. Para ello ejecutamos el comando `> use nombre_module`.
A continuación, para saber qué opciones tiene el módulo, usamos el comando `> show options`. Especificamos los valores que requiera el módulo.
Tras esto, lanzamos la ejecución del módulo mediante el comando `> run`.
Una vez finalizada la ejecución, podremos ver los resultados obtenidos. La herramienta permite la exportación de los resultados a un fichero HTML.