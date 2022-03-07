---
title: Explotación de vulnerabilidades en Web
author:
  name: Javier Fernández Santos
  link: https://github.com/JaviFS97
date: 2021-07-10 017:35:00 +0800
categories: [Ciberseguridad, Seguridad Ofensiva]
tags: [Pentesting, Metasploi, Mutillidae, BurpSuite, Skipfish, SQLi, SQLmap, PathTraversal, WebShell, XSS, XSStrike]
math: true
mermaid: true
assets_path: /assets/img/posts/20210710

---
Durante los posts anteriores hemos visto métodos para recopilar y analizar información sobre nuestros objetivos. Durante este post veremos como explotar (en Web) aquellas vulnerabilidades que presentan.


# Contenido
- [Preparación de una aplicación web vulnerable: Mutillidae](#preparación-de-una-aplicación-web-vulnerable-mutillidae)
  * [¿Qué es Mutillidae?](#qué-es-mutillidae)
  * [Instalación](#instalación)
- [Burp Suite](#burp-suite)
  * [Proxy de interceptación de tráfico de navegación](#proxy-de-interceptación-de-tráfico-de-navegación)
    + [Caso de uso](#caso-de-uso)
  * [Spider para el descubrimiento e indexación de contenido](#spider-para-el-descubrimiento-e-indexación-de-contenido)
- [Inyección de código](#inyección-de-código)
  * [SQL Injection (SQLi)](#sql-injection-sqli)
  * [Tipos de SQLi](#tipos-de-sqli)
  * [SQLmap](#sqlmap)
    + [Caso de uso](#caso-de-uso-1)
  * [Path Traversal](#path-traversal)
    + [Caso de uso](#caso-de-uso-2)
  * [WebShells](#webshells)
    + [Caso de uso](#caso-de-uso-3)
  * [Unrestricted File Upload](#unrestricted-file-upload)
    + [Caso de uso](#caso-de-uso-4)
  * [HTML Injection y Cross-Site-Scripting (XSS)](#html-injection-y-cross-site-scripting-xss)
    + [Caso de uso](#caso-de-uso-5)
  * [Cross-site request forgery (CSRF)](#cross-site-request-forgery-csrf)
    + [Caso de uso](#caso-de-uso-6)
  * [XSStrike (Automatización de ataques XSS)](#xsstrike-automatización-de-ataques-xss)
    + [Caso de uso](#caso-de-uso-7)




# Preparación de una aplicación web vulnerable: Mutillidae 
## ¿Qué es Mutillidae?
OWASP Mutillidae II es una aplicación web libre, open source, deliberadamente vulnerable, la cual proporciona un objetivo para entusiastas en seguridad web. Mutillidae puede ser instalado sobre Linux y Window utilizando LAMP, WAMP, y XAMMP. Incluye docenas de vulnerabilidades y sugerencias para ayudar al usuario. Es un entorno para hacking web fácil de utilizar, diseñado para laboratorios, entusiastas de seguridad, salones de clase, CTF, y herramientas para la evaluación de vulnerabilidades. Mutillidae ha sido utilizado en cursos de seguridad, cursos de entrenamiento para empresas, y como objetivo para evaluar diverso software de seguridad.


![Mutillidae]({{ page.assets_path}}/mutillidae-home.png){: .shadow style="min-width: 70%;" }
_Captura de la página principal de Mutillidae_


## Instalación
Seguir los pasos detallados en [github mutillidae](https://github.com/webpwnized/mutillidae).



# Burp Suite
Plataforma de auditoría web la cual incorpora un potente grupo de herramientas que nos permiten realizar desde la búsqueda de vulnerabilidades hasta las pruebas de explotación de las mismas.

Incorpora los siguientes módulos:
* Proxy de interceptación de tráfico de navegación, tanto HTTP como HTTPS.
* Spider para el descubrimiento e indexación de contenido.
* Escáner de aplicaciones web.
* Herramientas avanzadas de Fuzzing.
* Análisis de sesiones.
* Soporte para plugins.

Es una de las herramientas más potentes de auditorias web que se pueden encontrar, pero aunque disponga de una versión gratuita (Community Edition) en ella no se incluyen muchas de las funcionalidades que incorporan las versiones de pago (Enterprise o Proffesional), por lo que si queremos obtener buenos resultados no nos queda más remedio que comprar su licencia.


## Proxy de interceptación de tráfico de navegación
Como función principal de BurpSuite, el módulo Proxy intercepta el servidor proxy HTTP/S y actúa como intermediario entre el navegador y la aplicación de destino, lo que le permite interceptar, ver y modificar el flujo de datos original en ambas direcciones. 

Diseñado para pruebas de penetración, permite encontrar y explorar las vulnerabilidades de la aplicación al monitorear y manipular los parámetros clave y otros datos transmitidos por la aplicación.

### Caso de uso
Supongamos que queremos interceptar una petición de recuperación de contraseñas. Para recuperar la contraseña, además del email del usuario, se nos pregunta por el color preferido del usuario. Antes de proceder con el ataque debemos configurar un par de cosas.

Para que Burp Suite intercepte las peticiones es necesario configurar el navegador para que dirija todo el tráfico al puerto donde esté el proxy a la escucha. A la hora de ejecutar el módulo proxy de BurpSuite nos indicará en que puerto se encuentra alojado. En este caso, el sevicio está corriendo en el puerto 8080. En el navegador configuramos para que haga uso del proxy alojado en localhost:8080.
![Configurando uso de proxy en el navegador]({{ page.assets_path}}/burpsuite1proxy.png){: .shadow style="min-width: 70%;" }
_Configurando uso de proxy en el navegador_

Una vez configurado, si navegamos por la web podemos ver que esta se queda en espera (cargando), esto se debe a que hemos capturado la petición y hasta que no digamos lo contrario seguirá en espera. De esta forma podemos capturar las peticiones que hacemos contra la web.
![Configurando uso de proxy en el navegador]({{ page.assets_path}}/burpsuite2proxy.png){: .shadow style="min-width: 70%;" }
_Configurando uso de proxy en el navegador_

Como los colores son finitos, podemos realizar un ataque por diccionario para recuperar la contraseña del usuario objetivo. Burpsuite dispone de una herramienta que permite realizar estas acciones, el módulo `intruder`. Dada una petición interceptada, te permite seleccionar qué parámetros quieres que usen un diccionario. En este caso, queremos que el parametro 'color' use un diccionario, de esta forma iteraremos sobre todos los posibles colores hasta encontrar el correcto.
![Ataque diccionario con módulo intruder BurpSuite]({{ page.assets_path}}/burpsuite3proxy.png){: .shadow style="min-width: 70%;" }
_Ataque diccionario con módulo intruder BurpSuite_

Si lanzamos el ataque, se envía una petición por cada elemento que se encuentre en el diccionario. Cuando se introduce mal el color, el servidor devuelve la misma página de error siempre. Podemos observar que para el color 'green' la respuesta del servidor tiene una longitud distinta el resto. Esto quiere decir que hemos encontrado el color con el que recuperar la cuenta de dicho usuario.

![Resultado ataque diccionario con módulo intruder BurpSuite]({{ page.assets_path}}/burpsuite4proxy.png){: .shadow style="min-width: 70%;" }
_Resultado ataque diccionario con módulo intruder BurpSuite_


## Spider para el descubrimiento e indexación de contenido
Burp Suite realiza un escaneo pasivo o activo para el descubrimiento del contenido de la aplicación web objetivo.

Para el `escaneo pasivo`, tendremos que acceder a la pestaña `target` -> `site map`. Podemos observar que a la izquierda presenta una lista con una única ip. Esa ip es la de la web objetivo. Si desplegamos la ip, vemos el nombre que presenta la aplicación web. Dentro de esta vemos una sección `index.php` que presenta todas las rutas que Burp Suite ha sido capaz de descubrir de manera pasiva. Las que aparecen en negro son aquellas rutas que ya hemos visitado.

![Escaneo pasivo en burp suite]({{ page.assets_path}}/Burpsuite1.png){: .shadow style="min-width: 70%;" }
_Escaneo pasivo en burp suite_


Si queremos realizar un `escaneo activo`, tendremos que acceder a la pestaña `dashboard`. Pinchamos sobre la acción de `New live task` y podemos observar que el escaneo activo no está presenta en la versión gratuita de Burp Suite (En la version 1.X si que lo estaba). Por esta razón, si queremos realizar un escaneo activo, debemos comprar la licencia o usar otras herramientas que nos permitan realizarlo de manera gratuita. 


![Escaneo activo en burp suite]({{ page.assets_path}}/Burpsuite2.png){: .shadow style="min-width: 70%;" }
_Escaneo activo en burp suite_
Vamos a usar la herramienta gratuita `skipfish` para realizar el escaneo activo.

Para ello, ejecutamos el comando `skipfish -YO -o <ruta_salida_informe> <ruta_web>` en nuestro caso la <ruta> es http://192.168.16.133/mutillidae/index.php.

![Escaneo activo con skipfish]({{ page.assets_path}}/skipfish1.png){: .shadow style="min-width: 70%;" }
_Escaneo activo con skipfish_

Para finalizar el escaneo (puede llevar horas) podemos hacer ctrl+c. Esto generará un informe .html en la <ruta_salida_informe>. En este informe podemos encontrar todo tipo de información, desde tipos de archivos que haya encontrado hasta vulnerabilidades.
![Informe escaneo skipfish]({{ page.assets_path}}/skipfish2.png){: .shadow style="min-width: 70%;" }
_Informe escaneo skipfish_


# Inyección de código
Este tipo de ataques tienen como objetivo lograr la ejecución de comandos dentro del sistema objetivo, para lo cual se aprovechan de vulnerabilidades presentes en aplicaciones web.

Mediante la ejecución del código dañino que lleven asociadas pueden realizar un amplio abanico de actividades maliciosas. Pueden resultar sumamente peligrosas, más si tenemos en cuenta que son ataques muy difíciles de descubrir mediante pruebas funcionales, y se debe recurrir a un análisis del código fuente de la aplicación para descubrir estas inyecciones.

La causa más habitual que propia estos ataques es la falta de una correcta validación de los datos de entrada y salida de la aplicación.

## SQL Injection (SQLi)
La expansión en Internet de aplicaciones web que interactúan con bases de datos ha conllevado la aparición de estas vulnerabilidades en una gran cantidad de sitios web, convirtiéndose en uno de los mayores riesgos actuales para la seguridad de este tipo de entornos.

Los efectos de este tipo de ataques pueden ir desde la consulta, la modificación o la eliminación de los datos almacenados, hasta lograr comprometer credenciales de usuarios, o llegar a tomar el control total del servidor donde se aloja la aplicación.

Para poder probar si una página web presenta este tipo de vulnerabilidades es muy habitual la inclusión de determinados caracteres dentro de los parámetros de entrada, como `'` o el uso de comparaciones matemáticas como `1=1` que nos permitan extraer conclusiones sobre la estructura y la información presente en la base de datos.


## Tipos de SQLi
Existen tres tipos de ataques SQLi:
1. `Por error`: son los más habituales y fáciles de explotar. Consiste en ir probando diferentes comandos SQL pasados como parámetros, y analizar la respuesta (y los errores) que nos devuelve la aplicación.
![SQLi por error]({{ page.assets_path}}/SQLi1.png){: .shadow style="min-width: 70%;" }
_SQLi por error_


2. `Por unión`: No se sustituye la consulta original que la aplicación realiza sobre la base de datos, sino que se concatena nuestra sentencia SQL detrás de la suya. El resultado obtenido será devuelto por la consulta inicial.
![SQLi por unión]({{ page.assets_path}}/SQLi2.png){: .shadow style="min-width: 70%;" }
_SQLi por unión_

3. `Ciego (Blind)`: es el más difícil de explotar, recurriendo a él cuando ninguno de los ataques anteriores haya tenido éxito. Consiste en ir haciendo preguntas del tipo si/no a la base de datos, y analizar los resultados que vamos obteniendo. Podemos encontrar algunos `basados en contenido` (por ejemplo, muéstrame la información si la respuesta es si, y no me la muestres si es no), `basados en tiempo` (por ejemplo, muéstrame los resultados de manera inmediata si la respuesta es sí, y muéstramela con un determinado retardo si es no. `'union select null, sleep(20)`, devolverá el resultado a los 20 segundos. Si tarda 20 segundos en devolver el resultado => es vulnerable a ataques SQLi).
![SQLi ciego]({{ page.assets_path}}/SQLi3.png){: .shadow style="min-width: 70%;" }
_SQLi ciego_

## SQLmap
De todas las herramientas creadas para la automatización de ataques SQL destaca SQLmap. Se trata de una herramienta de código abierto que nos permite realizar inyecciones SQL de forma automática, pudiendo auditar de este modo los parámetros de entrada que tiene implementados una aplicación web determinada en busca de vulnerabilidades SQLi.

Es capaz de detectar todos los tipos de vulnerabilidades SLQi (por error, por unión y ciegas) en un amplio abanico de bases de datos.

A parte de todas estas características, incorpora funcionalidades adicionales que nos permitirán realizar otro tipo de ataques, como la subida de ficheros, inyección de comandos, fuerza bruta sobre los hashes de las contraseñas que haya podido recolectar e incluso escalada de privilegios dentro del sistema.

### Caso de uso
Para ver si la web es vulnerable a estos ataques, podemos pasar a SQLmap la url que queremos probar o guardar en un fichero la petición que se realiza en un determinado formulario. Aquí se puede observar que el parámetro username es vulnerable al ataque.
![Ejecución de SQLmap]({{ page.assets_path}}/SQLmap1.png){: .shadow style="min-width: 70%;" }
_Ejecución de SQLmap_

Otra acción que podemos hacer, por ejemplo, es sacar los nombres de todas las tablas presentes en la base de datos, para ello ejecutar el siguiente comando
![Obtención de tablas con SLQmap]({{ page.assets_path}}/SQLmap2.png){: .shadow style="min-width: 70%;" }
_Obtención de tablas con SLQmap_

## Path Traversal
Un directory traversal (o salto de directorio o cruce de directorio o path traversal) consiste en explotar una vulnerabilidad informática que ocurre cuando no existe suficiente seguridad en cuanto a la validación de un usuario, permitiéndole acceder a cualquier tipo de directorio superior (padre) sin ningún control.

La finalidad de este ataque es ordenar a la aplicación a acceder a un archivo al que no debería poder acceder o no debería ser accesible. Este ataque se basa en la falta de seguridad en el código. El software está actuando exactamente como debe actuar y en este caso el atacante no está aprovechando un bug en el código.

Directory traversal también es conocido como el ../ ataque punto punto barra, escalado de directorios y backtracking.

### Caso de uso
Vemos que la forma que tiene la web objetivo para obtener las páginas (archivos) es mediante el parámetro `page`:
![Analizando parámetros de la URL]({{ page.assets_path}}/PathTraversal1.png){: .shadow style="min-width: 70%;" }
_Analizando parámetros de la URL_

Para ver si es vulnerable, sustituimos el valor del parámetro `page` por otro en el que intentemos subir hasta la raíz de los ficheros del sistema y buscando alguno de los ficheros ubicados en esa ruta, en concreto passwd
![Path traversal para obtener el fichero de contraseñas del sistema]({{ page.assets_path}}/PathTraversal2.png){: .shadow style="min-width: 70%;" }
_Path traversal para obtener el fichero de contraseñas del sistema_

Podemos observar que es vulnerable porque nos muestra todas las contraseñas almacenadas en ese fichero
![Contraseñas extraídas gracias al Path traversal]({{ page.assets_path}}/PathTraversal3.png){: .shadow style="min-width: 70%;" }
_Contraseñas extraídas gracias al Path traversal_



## WebShells
Una WebShells no es una vulnerabilidad como tal, sino que es un script que vamos a inyectar en la máquina que está alojando la aplicación web, de manera que nosotros podamos ganar cosas como: la ejecución remota de comandos, persistencia en esa máquina, etcétera.

### Caso de uso
Para inyectar la webshell en la máquina objetivo vamos a usar los conceptos vistos anteriormente -> sql injection + Path Traversal.
Como sabemos que la web es vulnerable a los ataques SQLi, vamos a inyectar por medio de una consulta SQL un fichero que va a contener la webshell, aquí se puede observar la consulta SQLi:
![Insertando webShell]({{ page.assets_path}}/webshell1.png){: .shadow style="min-width: 70%;" }
_Insertando webShell_

Estamos inyectando un fichero con la webshell en la ruta `../../htdocs/mutillidae/backdoor.php` la webshell que luego, por medio de la vulnerabilidad path traversal (pasar por la url el parámetro `page=backdoor.php`) vamos a usar. Por ejemplo, realizando un `ls` obtenemos esta información:
![Ejecutando comandos en la webShell]({{ page.assets_path}}/webshell2.png){: .shadow style="min-width: 70%;" }
_Ejecutando comandos en la webShell_


## Unrestricted File Upload
Probablemente esta sea la vulnerabilidad que más fácilmente nos permite comprometer una máquina objetivo.

Es tan sencillo como aprovechar que la página web nos permite subir ficheros para subir una webshell. Si tengo la suerte de que me lo incluye en un directorio que sea accesible desde el navegador podríamos usarlo sin problema.

### Caso de uso
Accedemos al formulario que nos permite subir ficheros y seleccionamos aquel fichero que contenta la webshell (la misma utilizada en el apartado anterior). Al subirlo, podemos ver que nos indica la ruta en la que ha subido el fichero, si accedemos a él mediante Path Traversal ya tendríamos control sobre la webshell

![Aprovechando formulario para subir la webShell]({{ page.assets_path}}/UnrestrictedFileUpload1.png){: .shadow style="min-width: 70%;" }
_Aprovechando formulario para subir la webShell_

## HTML Injection y Cross-Site-Scripting (XSS)
La inyección HTML es un tipo de vulnerabilidad de inyección que se produce cuando un usuario es capaz de controlar un punto de entrada y es capaz de inyectar código HTML arbitrario en una página web vulnerable.

Cross-site scripting es un tipo de vulnerabilidad informática o agujero de seguridad típico de las aplicaciones Web, que puede permitir a una tercera persona inyectar en páginas web visitadas por el usuario código JavaScript o en otro lenguaje similar.


### Caso de uso
Existen 3 tipos de ataques XSS:
1. **XSS Reflected**:
El cross-site scripting reflejado (o XSS) surge cuando una aplicación recibe datos en una petición HTTP e incluye esos datos en la respuesta inmediata de forma insegura.

      Supongamos que un sitio web tiene una función de búsqueda que recibe el término de búsqueda proporcionado por el usuario en un parámetro de la URL:
`https://insecure-website.com/search?term=gift`

      La aplicación se hace eco del término de búsqueda suministrado en la respuesta a esta URL:
`<p>You searched for: gift</p>`

      Asumiendo que la aplicación no realiza ningún otro procesamiento de los datos, un atacante puede construir un ataque como este
`https://insecure-website.com/search?term=<script>/*+Malo+aquí...+*/</script>`

      Esta URL da lugar a la siguiente respuesta:
`<p>You searched for: <script>/* Bad stuff here... */</script></p>`

      Si otro usuario de la aplicación solicita la URL del atacante, entonces el script suministrado por el atacante se ejecutará en el navegador del usuario víctima, en el contexto de su sesión con la aplicación. **¿Esto qué quiere decir?, ¿Cómo podemos aprovecharlo?** Que podríamos crear un ataque de pishing donde pasaríamos la url que queremos que ejecuten nuestros objetivos, teniendo como script un código, que por ejemplo, solicite inicio de sesión y envíe dichas credenciales a un servidor que tengamos a la escucha.

![Diagrama ataque XSS Reflected]({{ page.assets_path}}/XSS_Reflected.png){: .shadow style="min-width: 70%;" }
_Diagrama ataque XSS Reflected_


2. **XSS Persistent:** 
Los ataques de Cross-site Scripting persistente (XSS almacenado) representan uno de los tres tipos principales de Cross-site Scripting. Los otros dos tipos de ataques de este tipo son el XSS no persistente (XSS reflejado) y el XSS basado en el DOM. En general, los ataques XSS se basan en la confianza de la víctima en una aplicación o sitio web legítimo pero vulnerable.

      Un ataque XSS persistente es posible cuando un sitio web o una aplicación web almacena la entrada del usuario y posteriormente la sirve a otros usuarios. Una aplicación es vulnerable si no valida la entrada del usuario antes de almacenar el contenido e incrustarlo en las páginas de respuesta HTML. Los atacantes utilizan las páginas web vulnerables para inyectar código malicioso y almacenarlo en el servidor web para su uso posterior. El payload se sirve automáticamente a los usuarios que navegan por las páginas web y se ejecuta en su contexto. Así, las víctimas no necesitan hacer click en un enlace malicioso para ejecutar el payload (como en el caso del XSS no persistente). Todo lo que tienen que hacer es visitar una página web vulnerable.

![Diagrama ataque XSS Persistent]({{ page.assets_path}}/XSS_Persistent.png){: .shadow style="min-width: 70%;" }
_Diagrama ataque XSS Persistent_


## Cross-site request forgery (CSRF)
Se relaciona muchísimo con estos dos fallos anteriores (HTML Injection y Cross-Site-Scripting) y que incluso yo puedo llegar a considerar casi más un tipo de payload de Cross-Site-Scripting.

Es como si nosotros estuviésemos explotando un Cross-Site-Scripting que podría ser Reflect o Persistent, pero insertando un JavaScript o cualquier tipo de código que va a provocar que el usuario realice algún tipo de acción sobre esa aplicación web en la que está logueado y que nos pueda resultar en un beneficio.

### Caso de uso
Imaginemos que nuestra web objetivo es un foro privado, eso quiere decir que el registro de usuarios solo se puede hacer si algún miembro del foro te invita. Podríamos utilizar un Cross-Site-Scripting donde el payload se encargue de realizar el registro aprovechando que algún usuario logueado caiga en nuestro Cross-Site-Scripting. De esta forma, dicho usuario nos estaría invitando al foro sin que él lo supiera.

![Diagrama ataque XSS CSRF]({{ page.assets_path}}/CSRF1.png){: .shadow style="min-width: 70%;" }
_Diagrama ataque CSRF_

Otro ejemplo puede ser:

Un ejemplo muy clásico se da cuando un sitio web, llamémoslo `"example1.com"`, posee un sistema de administración de usuarios. En dicho sistema, cuando un administrador se conecta y ejecuta el siguiente REQUEST GET, elimina al usuario de ID: "63":` http://example1.com/usuarios/eliminar/63`

Una forma de ejecutar la vulnerabilidad CSRF, se daría si otro sitio web, llamemos "example2.com", en su sitio web añade el siguiente código HTML: `<img src="http://example1.com/usuarios/eliminar/63">`

Cuando el usuario administrador (conectado en `example1.com`) navegue por este sitio atacante, su navegador web intentará buscar una imagen en la URL y al realizarse el REQUEST GET hacia esa URL eliminará al usuario 63.

## XSStrike (Automatización de ataques XSS)
XSStrike es una suite de detección de Cross Site Scripting equipada con cuatro analizadores escritos a mano, un generador inteligente de payloads, un potente motor de fuzzing y un crawler increíblemente rápido.

En lugar de inyectar payloads y comprobar que funcionan, como hacen todas las demás herramientas, XSStrike analiza la respuesta con múltiples analizadores sintácticos y, a continuación, crea payloads cuyo funcionamiento está garantizado por el análisis de contexto integrado con un motor de fuzzing.

### Caso de uso
Ejecutamos el programa indicando la url sobre la que queremos realizar el XSS. Dice que ha encontrado 5 XSS reflected y nos genera payloads personalizados para explotar la vulnerabilidad.
![Detección de XSS mediante XSStrike]({{ page.assets_path}}/XSStrike.png){: .shadow style="min-width: 70%;" }
_Detección de XSS mediante XSStrike_