---
title: Recopilación Activa de información
author:
  name: Javier Fernández Santos
  link: https://github.com/JaviFS97
date: 2021-06-21 13:35:00 +0800
categories: [Ciberseguridad, Seguridad Ofensiva]
tags: [Pentesting, Metasploitable3, NMAP, DNSrecon]
math: true
mermaid: true
assets_path: /assets/img/posts/20210621

---
En esta sección se han recopilado algunas de las principales herramientas usadas durante la fase de recopilación activa de información.

- [Antes de continuar...](#antes-de-continuar)
  * [¿Dónde entra el concepto de anonimato?](#dónde-entra-el-concepto-de-anonimato)
  * [¿Qué es lo que tratamos de conseguir con las técnicas que intentar garantizar el anonimato en Internet?](#qué-es-lo-que-tratamos-de-conseguir-con-las-técnicas-que-intentar-garantizar-el-anonimato-en-internet)
  * [¡¡A tener en cuenta!! LEER ANTES DE SEGUIR CON EL RESTO](#a-tener-en-cuenta-leer-antes-de-seguir-con-el-resto)
  * [Importancia del anonimato](#importancia-del-anonimato)
- [Objetivo](#objetivo)
- [Presentación del entorno vulnerable: Metasploitable3.](#presentación-del-entorno-vulnerable-metasploitable3)
- [DNSrecon y transferencia de zona](#dnsrecon-y-transferencia-de-zona)
- [Nmap](#nmap)
  * [Descubrimiento de Hosts](#descubrimiento-de-hosts)
  * [Escaneo de puertos](#escaneo-de-puertos)
  * [Descubrimiento de servicios](#descubrimiento-de-servicios)
  * [Identificación del sistema operativo](#identificación-del-sistema-operativo)

# Antes de continuar...

## ¿Dónde entra el concepto de anonimato?
Utilizando algún método para garantizar el anonimato como puede ser una VPN, un proxy, red tor,... de manera genérica lo que estamos haciendo es tunelizar nuestro tráfico de red aplicando algún tipo de cifrado desde una máquina en nuestra red doméstica/local hasta un servidor en Internet que se encargará de descifrar el tráfico y realizar la petición a donde nosotros queramos dirigirla. Es importante entender, que aunque el contenido del tráfico de red vaya cifrado, **ese tráfico de red sigue pasando por nuestro router para poder llegar al servidor que lo descifra y, por lo tanto, nuestro router sigue estando expuesto a Internet.**

**El riesgo de seguridad al que estamos expuestos todos**, independientemente de utilizar mecanismos para garantizar el anonimato o no, **es a la explotación de un posible fallo de seguridad en nuestro router de manera que permita ganar acceso a los sistemas que se encuentren conectados a nuestra red local**. Este riesgo no puede mitigarse por completo debido a que el router tiene que estar expuesto a Internet para poder habilitar la conexión de los dispositivos que se encuentran en nuestra red privada.

## ¿Qué es lo que tratamos de conseguir con las técnicas que intentar garantizar el anonimato en Internet?

Con este tipo de técnicas y herramientas **tratamos de garantizar la privacidad y confidencialidad de nuestras conexiones**. De esta manera, cuando nos conectamos a una máquina en Internet, por ejemplo un servidor de aplicación que proporciona una página web, lo hacemos a través de una o varias máquinas intermedias con las que establecemos una comunicación cifrada.

Aquí hay varias cosas que se deben tener en cuenta. Las máquinas que se utilizan para salir a internet cuando utilizas mecanismos de anonimato como una VPN, tienen que descifrar tu tráfico de red y, por lo tanto, pueden monitorizarlo y registrarlo. **Si realizas cualquier tipo de actividad ilegal utilizando estos mecanismos, nada te garantiza que no vayas a ser identificado, sobre todo si estás utilizando un servicio gratuito.**

Como reitero en varias ocasiones, **no debería utilizarse ninguna técnica de hacking que sea ilegal sobre un sistema o una organización con la que no tienes un contrato establecido para la realización de un hacking ético**. En caso contrario, utilices o no técnicas que garanticen el anonimato, te estás exponiendo a que ser identificado.

## ¡¡A tener en cuenta!! LEER ANTES DE SEGUIR CON EL RESTO
En secciones anteriores no se presentan herramientas para garantizar el anonimato porque las técnicas de recopilación pasiva de información no son ilegales y no necesitas ocultar tu dirección ip pública. Son técnicas que se basan en búsqueda de información en fuentes abiertas y no van dirigidas a ningún objetivo en particular. A partir de ahora, donde empezamos a ver técnicas de recopilación activa de información, en las que se dirigen a objetivos específicos, **establecemos un laboratorio virtual donde realizamos todas las pruebas sobre servidores y máquinas en nuestro entorno controlado, nunca hacia Internet**. No os recomiendo bajo ningún concepto que utilicéis técnicas activas de hacking sobre objetivos que se encuentren en Internet y para los que no disponéis de un contrato previo para realización de un Hacking Ético, ni siquiera utilizando herramientas para garantizar el anonimato.

## Importancia del anonimato
Una vez presentada esta reflexión sobre las técnicas para garantizar el anonimato en Internet, me gustaría hablar sobre la importancia que tienen para ciertas tareas. Uno de los ejemplos más claros en los que pueden servir técnicas como una VPN es cuando estamos conectados a una red local no confiable, como puede ser la red wifi de una cafetería o de un aeropuerto. En estos casos no sabemos que dispositivos hay conectados a nuestra misma red local o si pueden estar monitorizando nuestra comunicación (como se presenta en la sección "Explotación y hacking de vulnerabilidades en Red"), por lo tanto, el uso de una VPN, que cifra el tráfico de red desde nuestro equipo a un servidor en Internet, nos garantiza que alguien interceptando nuestra comunicación no pueda ver nada.

Para terminar con el artículo, me gustaría indicaros algunas de las soluciones y herramientas más populares para garantizar el anonimato en Internet:
1. La opción más sencilla es utilizar un proxy. Si utilizas un proxy gratuito y público ten en cuenta que en esa máquina de destino por la que estas saliendo a Internet pueden estar monitorizando tu navegación.
2. Otro de los mecanismos más comunes es utilizar una VPN.
3. Aunque es menos popular, podéis conectaros a Internet a través de la Red Tor, esto garantiza que los nodos de salida a Internet pertenezcan a esta red y, por lo tanto, su dirección ip pública sea diferente a la tuya. En este caso también debes tener en cuenta que mucha gente utiliza este mecanismos y sus direcciones ip públicas están bloqueadas frecuentemente por los buscadores más populares como Google.
4. Podéis establecer vuestra propia máquina de salto en la nube. Puedes crear una cuenta gratuita de AWS (Amazon Web Services) y crear una máquina (instancia) en la nube que puedes utilizar de máquina de salto para conectarte a Internet y hacer las peticiones a través de ella, sería muy similar a tener tu proxy personal.



# Objetivo
Recolección de información sobre un objetivo determinado utilizando **métodos que interactúan de manera directa con la organización** normalmente mediante el envío de tráfico de red.

Alcance: 
  * Escáneres de hosts.
  * Escáneres de puertos.
  * Escáneres de servicios.

En muchas ocasiones la actividad de este tipo de técnicas suele ser detectada como actividad sospechosa o maliciosa.


# Presentación del entorno vulnerable: Metasploitable3.
Metasploitable3 es una máquina virtual gratuita de Windows Server 2008 y ubuntu 16.04 que cuenta con varias vulnerabilidades listas para ser explotadas y practicar nuestras técnicas de hacking o simular ataques, Metasploitable3 ha sido desarrollado por Rapid7, los mismos desarrolladores del conocido Framework MetaSploit.

Es usado por gran parte de la industria de la ciberseguridad por varios motivos, cómo por ejemplo, la formación de personal para la explotación de redes, pruebas de software o demostración y viabilidad de ataques.

Usar una máquina virtual para probar nuestros ataques es la manera más cómoda y rápida de conocer hasta dónde podemos llegar a explotar un sistema sin ponerlo en riesgo, además, siempre contamos con la ventaja de que si algo sale mal, volvemos a reinstalar y tenemos de nuevo un sistema vulnerable en cuestión de minutos.


# DNSrecon y transferencia de zona
[TODO]


# Nmap
Nmap ("Network Mapper") es una herramienta de código abierto para la exploración de redes y la auditoría de seguridad. Fue diseñada para escanear rápidamente grandes redes, aunque funciona bien contra hosts individuales. Nmap utiliza paquetes para determinar qué hosts están disponibles en la red, qué servicios (nombre y versión de la aplicación) ofrecen esos hosts, qué sistemas operativos (y versiones del SO) están ejecutando, qué tipo de filtros de paquetes/firewalls están en uso, y docenas de otras características. 

Aunque Nmap se utiliza habitualmente para auditorías de seguridad, muchos administradores de sistemas y redes lo encuentran útil para tareas rutinarias como el inventario de la red, la gestión de los programas de actualización de servicios y la supervisión del tiempo de actividad de los hosts o servicios.

En la ruta `usr/share/nmap/scripts` tenemos una colección de script que nos pueden ayudar a obtener información valiosa. Para llevarlo a cabo, filtrar en esa carpeta los scripts que tenemos para un determinado servicio mediante `ls http*`, escogemos el script y ejecutamos  `nmap -v <acción> -p <puerto> --script=<script_puerto_escogido> <ip>`

## Descubrimiento de Hosts

* **Descubrimiento de un host en particular**:
`nmap -sn <ip_objetivo>` -> determina si el servidor está levantado. Realizar un TCP SYNC contra los puertos 80 y 443. Si no hay nada corriendo en esos puertos dará el servidor como no levantado.

  Si se ejecuta con permisos de administrador `sudo`, será menos intrusivo pues se hará una petición ARP para ver si el servidor está activo.

* **Descubrimiento de todos los hosts de una red**:
`nmap -sn <ip_objetivo>/<mascara_red>` -> determina cuantos de los hosts están levantados. De nuevo, realizar un TCP SYN contra los puertos 80 y 443. Si no hay nada corriendo en esos puertos dará el host como apagado.

   Si se ejecuta con permisos de administrador `sudo`, será menos intrusivo pues se hará una petición ARP para ver si el servidor está activo.

Documentación oficial sobre [host discovery](https://nmap.org/book/man-host-discovery.html). Se observa que en este apartado existen más comandos para el descubrimiento de host, pero muchos de ellos son descubrimiento de puertos, lo cual genera mucho tráfico de red para determinar si un host está o no levantado.


## Escaneo de puertos
Esta técnica sirve para **determinar** el estado de los puertos. 

Esta técnica **NO determina** qué aplicación hay corriendo por detrás del puerto. Por ejemplo, si encuentra el puerto 80 abierto dirá que es HTTP, pero puede que en el servidor hayan cambiado el servicio HTTP a otro puerto.

Más info [aquí](https://nmap.org/book/man-port-scanning-techniques.html).

Los 6 estados reconocidos por nmap: 
1. **Abierto:**
Una aplicación acepta conexiones TCP o paquetes UDP en este puerto. El encontrar esta clase de puertos es generalmente el objetivo primario de realizar un sondeo de puertos. Las personas orientadas a la seguridad saben que cada puerto abierto es un vector de ataque. Los atacantes y las personas que realizan pruebas de intrusión intentan aprovechar puertos abiertos, por lo que los administradores intentan cerrarlos, o protegerlos con cortafuegos, pero sin que los usuarios legítimos pierdan acceso al servicio. Los puertos abiertos también son interesantes en sondeos que no están relacionados con la seguridad porque indican qué servicios están disponibles para ser utilizados en una red.

2. **Cerrado:**
Un puerto cerrado es accesible: recibe y responde a las sondas de Nmap, pero no tiene una aplicación escuchando en él. Pueden ser útiles para determinar si un equipo está activo en cierta dirección IP (mediante descubrimiento de sistemas, o sondeo ping), y es parte del proceso de detección de sistema operativo. Como los puertos cerrados son alcanzables, o sea, no se encuentran filtrados, puede merecer la pena analizarlos pasado un tiempo, en caso de que alguno se abra. Los administradores pueden querer considerar bloquear estos puertos con un cortafuegos. Si se bloquean aparecerían filtrados, como se discute a continuación.

3. **Filtrado:**
Nmap no puede determinar si el puerto se encuentra abierto porque un filtrado de paquetes previene que sus sondas alcancen el puerto. El filtrado puede provenir de un dispositivo de cortafuegos dedicado, de las reglas de un enrutador, o por una aplicación de cortafuegos instalada en el propio equipo. Estos puertos suelen frustrar a los atacantes, porque proporcionan muy poca información. A veces responden con mensajes de error ICMP del tipo 3, código 13 (destino inalcanzable: comunicación prohibida por administradores), pero los filtros que sencillamente descartan las sondas sin responder son mucho más comunes. Esto fuerza a Nmap a reintentar varias veces, considerando que la sonda pueda haberse descartado por congestión en la red en vez de haberse filtrado. Esto ralentiza drásticamente los sondeos.

4. **No filtrado:**
Este estado indica que el puerto es accesible, pero que Nmap no puede determinar si se encuentra abierto o cerrado. Solamente el sondeo ACK, utilizado para determinar las reglas de un cortafuegos, clasifica a los puertos según este estado. El analizar puertos no filtrados con otros tipos de análisis, como el sondeo Window, SYN o FIN, pueden ayudar a determinar si el puerto se encuentra abierto.

5. **abierto|filtrado:**
Nmap marca a los puertos en este estado cuando no puede determinar si el puerto se encuentra abierto o filtrado. Esto ocurre para tipos de análisis donde no responden los puertos abiertos. La ausencia de respuesta puede también significar que un filtro de paquetes ha descartado la sonda, o que se elimina cualquier respuesta asociada. De esta forma, Nmap no puede saber con certeza si el puerto se encuentra abierto o filtrado. Los sondeos UDP, protocolo IP, FIN, Null y Xmas clasifican a los puertos de esta manera.

6. **cerrado|filtrado:**
Este estado se utiliza cuando Nmap no puede determinar si un puerto se encuentra cerrado o filtrado, y puede aparecer aparecer sólo durante un sondeo IP ID pasivo.


* Opciones
   * -sS (TCP SYN scan) -> `nmap -sS <ip>`
 Esta técnica se denomina **a menudo escaneo semiabierto**, porque no se abre una conexión TCP completa (no realiza el Three-way Handshake).
 Envías un paquete SYN, como si fueras a abrir una conexión real y luego esperas una respuesta. 
   a) Un SYN/ACK indica que el puerto está escuchando (abierto),
   b) mientras que un RST (reinicio) es indicativo de que no está escuchando. Si no se recibe respuesta tras varias retransmisiones, el puerto se marca como filtrado.


  * -sT (TCP connect scan) -> `nmap -sT <ip>` La técnica anterior, al no realizar la conexión TCP completa (Three-way Handshake) puede levantar sospechas en el IDS. Para evitar que salten las alarmas lo realizamos con -sT, que lanza una conexión TCP completa (en un buen IDS también saltarán al realizarlo con -sT).

  * -oX -> Exportar los resultados a una hoja xml para ver con más claridad los resultados. Por ejemplo: `nmap -sS <ip> -oX <archivo_destino>.xml --stylesheet="https://svn.nmap.org/nmap/docs/namp.xsl"` Más info [aquí](https://nmap.org/book/man-output.html).

![Nmap xml]({{ page.assets_path}}/Nmap.png){: .shadow style="min-width: 70%;" }
_Nmap xml_

## Descubrimiento de servicios
Vamos a ir un paso más allá y vamos a ver qué servicio se encuentra corriendo detrás de un puerto abierto.

* -sV (Version detection) -> `nmap -sV <ip>`. Nos devuelve los puertos que estén abiertos y la aplicación que está corriendo. Además, tambien nos informa del SO de la maquina. Con toda esta información, podremos buscar si existe algún exploit que comprometa la seguridad de las aplicaciones o del SO. 

![Nmap descubrimiento de servicios]({{ page.assets_path}}/Nmap-DescubrimientoServicios.png){: .shadow style="min-width: 70%;" }
_Nmap descubrimiento de servicios_

## Identificación del sistema operativo
* -O (Enable OS detection) -> `nmap -O <ip>`. 
Más info [aquí](https://nmap.org/book/man-os-detection.html)