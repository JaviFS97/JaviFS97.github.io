---
title: Recopilación Semi-Pasiva de información
author:
  name: Javier Fernández Santos
  link: https://github.com/JaviFS97
date: 2021-06-10 18:03:00 +0800
categories: [Ciberseguridad, Seguridad Ofensiva]
tags: [Pentesting, FOCA, DNSdumpster, Sniffers, Wireshark, TCPdump]
math: true
mermaid: true
assets_path: /assets/img/posts/20210610

---
En esta sección se han recopilado algunas de las principales herramientas usadas durante la fase de recopilación semi-pasiva de información.

# Contenido
- [Objetivo](#objetivo)
- [FOCA](#foca)
  * [¿Qué son los metadatos?](#qué-son-los-metadatos)
  * [¿Cómo funciona?](#cómo-funciona)
  * [Caso de uso](#caso-de-uso)
- [DNSdumpster](#dnsdumpster)
  * [Tipos de registro DNS más importantes](#tipos-de-registro-dns-más-importantes)
  * [Caso de uso](#caso-de-uso-1)
- [Sniffers](#sniffers)
  * [Wireshark](#wireshark)
    + [Caso de uso](#caso-de-uso-2)
  * [TCPdump](#tcpdump)

# Objetivo
En la fase anterior hemos visto en qué consistía la fase de recopilación de información pasiva. 

**En muchas ocasiones no se hace distinción entre la fase de recopilación pasiva y semi-pasiva**. Sin embargo, nosotros hacemos distinción entre ambas, porque como se dijo en la fase anterior, mediante la **recopilación pasiva** no se genera tráfico contra el objetivo, es decir, **no existía ningún tipo de interacción el objetivo**, por eso extraíamos la información de fuente públicas.

**En esta fase**, empezamos a **generar tráfico contra el objetivo de manera que nuestro no se dé cuenta de que ese tráfico de red** viene como consecuencia de que estamos haciendo una recopilación de información.

---

Como resumen:
* Recolección de información sobre un objetivo utilizando métodos que se asimilen al tráfico de red y comportamiento real que suele recibir.
* Se encuentran actividades como:
   * Consultas a servidores DNS.
   * Accesos a recursos internos de las aplicaciones web.
   * Análisis a metadatos de documentos.
* Quedan fuera del alcance actividades que generen comportamiento anómalo. 



# FOCA
Su objetivo es la obtención de información relevante (metadatos) de nuestros objetivos a través de documentos de su propiedad que se puedan encontrar de forma libre en internet. Solo para windows.

## ¿Qué son los metadatos?
Los metadatos, literalmente «sobre datos», son datos que describen otros datos.

Por ejemplo, cuando se crea un documento PDF se le asocian una serie de metadatos como la fecha de creación, nombre del equipo donde se generó el archivo, el SO, etc.

## ¿Cómo funciona?
Realiza una búsqueda de estos ficheros en determinados buscadores, para posteriormente analizar toda la información que se pueda extraer de los metadatos que haya en los mismos y de la urls donde se hayan descargado.


## Caso de uso
Podemos obtener nombres de equipos, SO usado y con ello usar vulnerabilidades que tenga, emails, etc.

En esta imagen se puede observar los nombres de equipos o usuarios que han creado ciertos archivos
![FOCA]({{ page.assets_path}}/FOCA.png){: .shadow style="min-width: 70%;" }
_FOCA_




# DNSdumpster
Nos permite realizar un análisis de la red y de todo el dominio que estamos analizando. Esto nos ayudará a conocer mejor por dónde nos estamos moviendo, así como descubrir posibles rutas y subdominios, así como los distintos hosts que forman la red.

Es una herramienta opensource (alojada en https://dnsdumpster.com/) diseñada para facilitarnos el análisis de cualquier dominio y poder obtener en segundos todo tipo de información relacionada con este. No utiliza la fuerza bruta como pueden usar otras similares, sino que recurre a la Open Source Intelligence (OSINT) para encontrar de manera silenciosa toda la información relacionada con el dominio que nos interese analizar.

## Tipos de registro DNS más importantes

| Siglas | Nombre           | Descripción|
| ------------- |:-------------:|-----|
| A |  Registro de dirección | Devuelve una dirección IP. Este registro sirve para resolver nombres de alojamientos a un número IPv4, teniendo en cuenta si la IP es dinámica o fija. Por ejemplo, para apuntar nuestro nombre de dominio a un servidor. |
| AAAA  | Registro de dirección IPv6 |  Los registros AAAA son muy parecidos a los A, es decir, ambos devuelven una dirección IP. En el caso de los AAAA, las IPs que se almacenan son IPv6. Este tipo de registro de DNS, al igual que A, sirve para apuntar nuestro dominio a un determinado servidor.  |
| CAA  | Autorización de la Autoridad de Certificación |  Este parámetro de DNS es un mecanismo de seguridad que permite limitar las autoridades certificadoras válidas para un dominio. Es decir, indicar a qué autoridades certificadoras permitimos emitir certificados de seguridad (SSL) para el dominio. |
| CNAME  | Registro de nombre canónico  | Este registro se suele utilizar para crear alias de un nombre. CNAME es una forma de hacer que el dominio apunte a otro dominio diferente o a un subdominio. También puede usarse cuando distintos servicios están utilizando una misma IP, de forma que cada servicio tenga su propia entrada DNS. |
| MX  | Registro de intercambio del correo | Los registros MX apuntar al servidor de correo del dominio y es posible establecer tantos como sean necesarios. En relación con estos registros ten en cuenta que, de forma automática, se establecen prioridades. Es decir, el primer registro MX que introduzcas tendrá prioridad sobre los siguientes.  |
| PTR  | Registro de puntero | O registro inverso, ya que funciona de manera opuesta a A, se encarga de traducir IPs a nombres de dominio. Generalmente PTR se utiliza en el archivo de configuración de la zona DNS inversa. |
| SRV  | Localizador de servicios | Es un registro para servicios especiales que proporciona información relacionada con los servicios disponibles para un determinado dominio. Es habitual su uso con XMPP, LDAP o SIP. |
| TXT  | Registro de texto | Registro para insertar el texto que desees. Suele utilizarse para verificar la autoridad del dominio o para evitar usos incorrectos de las direcciones de correo. Además, TXT permite la creación de registros especiales y Domain-Keys. |



## Caso de uso
Introducimos el dominio que vamos a analizar, por ejemplo, tesla.com. Automáticamente esta plataforma empezará a analizar todo lo relacionado con este dominio. Cuando finalice el análisis del mismo podremos ver ya toda la información relacionada con él (además de los subdominios, nos devolverá información del servidor DNS, registros MX, registros TXT y un interesante esquema de las relaciones del dominio analizado.)

En esta imagen se puede observar los nombres de equipos o usuarios que han creado ciertos archivos
![Información extraída]({{ page.assets_path}}/DNSdumpster2.png){: .shadow style="min-width: 70%;" }
_Información extraída_

![DNSdumpster grafo de relaciones]({{ page.assets_path}}/DNSdumpster.png){: .shadow style="min-width: 70%;" }
_DNSdumpster grafo de relaciones_


# Sniffers
Puede entenderse como un programa con la capacidad de observar el flujo de datos en tránsito por una red, y obtener información de éste; está diseñado para analizar los paquetes de datos que pasan por la red y no están destinados para él, lo que bajo ciertas circunstancias es muy útil, y bajo otras, a la vez, muy peligroso.

## Wireshark
Es un potente sniffer de red basado en pcap. Desarrollado como software libre y se encuentra disponible para multitud de SO.

Es el captador pasivo de tráfico de red más utilizado en la actualidad, permitiendo interceptar paquetes de información tanto en redes cableadas como Wireless.

### Caso de uso
El primer paso que tenemos que dar para realizar una captura pasiva de tráfico es habilitar el modo promiscuo en la interfaz de red que vamos a utilizar. (en entorno linux `ifconfig <nombre_interfaz> promisc`). 

Vamos a realizar una prueba para comprobar el peligro que corren aquello usuarios que inician sesión en una página web que no utiliza protocolos seguros. Para ello, buscamos una web que esté desarrollada bajo el protocolo HTTP. Realizamos el inicio de sesión y podremos observar el paquete en wireshark con las credenciales que ha usado el usuario.

![Wireshark]({{ page.assets_path}}/Wireshark.png){: .shadow style="min-width: 70%;" }
_Wireshark_

 
## TCPdump
Si no podemos acceder una interfaz gráfica y únicamente tenemos acceso a una interfaz de comandos (shell) la única forma que tenemos de realizar el análisis de la red es por medio de este programa.

Como se hace más difícil la lectura de los paquetes en la shell, podemos guardar mediante el comando `tcpdump -i <nombre_interfaz>  -w <nombre_fichero_salida>.pcap` el análisis realizado. Ese archivo generado podemos importarlo a wireshark para su interpretación.
