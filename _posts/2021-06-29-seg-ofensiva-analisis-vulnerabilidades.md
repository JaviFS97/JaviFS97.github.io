---
title: Análisis de vulnerabilidades
author:
  name: Javier Fernández Santos
  link: https://github.com/JaviFS97
date: 2021-06-29 11:35:00 +0800
categories: [Ciberseguridad, Seguridad Ofensiva]
tags: [Pentesting, CVE, CVEDetails, CVSS, CPE, NMAP, Nessus]
math: true
mermaid: true
assets_path: /assets/img/posts/20210629

---
Durante esta sección veremos las diferentes alternativas, tanto manuales como automáticas, de las que disponemos para realizar análisis de vulnerabilidades sobre un objetivo.

# Contenido
- [Objetivo](#objetivo)
- [CVE, CVSS, CPE](#cve-cvss-cpe)
  * [¿Cómo buscamos vulnerabilidades asociadas a una aplicación/protocolo/servicio? (CVE)](#cómo-buscamos-vulnerabilidades-asociadas-a-una-aplicaciónprotocoloservicio-cve)
  * [¿Cómo de peligrosa es una vulnerabilidad? (CVSS)](#cómo-de-peligrosa-es-una-vulnerabilidad-cvss)
  * [¿Cómo saber a cuántas versiones afecta esta vulnerabilidad? (CPE)](#cómo-saber-a-cuántas-versiones-afecta-esta-vulnerabilidad-cpe)
  * [Web alternativa](#web-alternativa)
- [Herramientas automáticas para el análisis de vulnerabilidades](#herramientas-automáticas-para-el-análisis-de-vulnerabilidades)
  * [NMAP](#nmap)
  * [Nessus](#nessus)
    + [Puesta en marcha](#puesta-en-marcha)
    + [Caso de uso](#caso-de-uso)
  * [Otras herramientas](#otras-herramientas)

# Objetivo
Llegados a este punto ya hemos recopilado información de nuestro objetivo mediante técnicas más o menos intrusivas. Ahora tenemos que encontrar problemas de seguridad en esos sistemas. En la siguiente sección (6. Explotación y hacking de vulnerabilidades) veremos cómo explotar los fallos de seguridad encontramos en esta sección para por ejemplo: ganar acceso al sistema, elevar privilegios, poderse mover lateralmente por la red, etc

El tipo de fallos abarca desde errores en la configuración de un servicio hasta vulnerabilidades en determinados servicios que sean públicos y puedan comprometer la integridad del mismo

# CVE, CVSS, CPE

Durante el escaneo nmap que hicimos en fases anteriores vimos el conjunto de servicios que tenía el servidor. Con dicha información podemos comenzar a buscar vulnerabilidades. En este caso vamos a analizar si la aplicación `ProFTDP` con versión `1.3.5` presenta alguna vulnerabilidad.
![Análisis Nmap]({{ page.assets_path}}/CVE1.png){: .shadow style="min-width: 70%;" }
_Análisis Nmap_

## ¿Cómo buscamos vulnerabilidades asociadas a una aplicación/protocolo/servicio? (CVE)
Se puede usar el repositorio CVE ([Common Vulnerabilities and Exposures ](https://cve.mitre.org/)) para buscar vulnerabilidades reportadas. Ya que es una lista de información registrada sobre vulnerabilidades de seguridad conocidas, en la que cada referencia tiene un número de identificación CVE-ID, descripción de la vulnerabilidad, que versiones del software están afectadas, posible solución al fallo (si existe) o como configurar para mitigar la vulnerabilidad y referencias a publicaciones o entradas de foros o blog donde se ha hecho pública la vulnerabilidad o se demuestra su explotación. Además suele también mostrarse un enlace directo a la información de la base de datos de vulnerabilidades del NIST (NVD), en la que pueden conseguirse más detalles de la vulnerabilidad y su valoración.

Realizamos la búsqueda y encontramos que hay una vulnerabilidad reportada. En las referencias podemos ver mucha información sobre la vulnerabilidad, y hasta incluso un exploit (se verá en la fase de explotación).
![CVE]({{ page.assets_path}}/CVE2.png){: .shadow style="min-width: 70%;" }
_CVE_

## ¿Cómo de peligrosa es una vulnerabilidad? (CVSS)
En CVSS (Common Vulnerability Scoring System) podemos ver el nivel de peligrosidad de una vulnerabilidad.

Si buscamos la vulnerabilidad de antes, podemos observar que es una vulnerabilidad con una puntuación de 10. Por lo que se trata de una vulnerabilidad crítica.
![CV3]({{ page.assets_path}}/CVE3.png){: .shadow style="min-width: 70%;" }
_CV3_

## ¿Cómo saber a cuántas versiones afecta esta vulnerabilidad? (CPE)
Mediante el CPE (Common Platform Enumeration). Es un esquema de nomenclatura estructurado para sistemas, software y paquetes de tecnología de la información. Basado en la sintaxis genérica de los Identificadores Uniformes de Recursos (URI), CPE incluye un formato de nombre formal, un método para comprobar los nombres con un sistema y un formato de descripción para vincular texto y pruebas a un nombre.
![CV4]({{ page.assets_path}}/CVE4.png){: .shadow style="min-width: 70%;" }
_CV4_

## Web alternativa
Existe otra web alternativa ([cvedetails](https://www.cvedetails.com/)) donde se aglutina toda la información recopilada en las webs que hemos visto hace un momento (consulta las APIs de CVE Y CVSS).


# Herramientas automáticas para el análisis de vulnerabilidades
En vez de realizarlo de manera manual, podemos usar una serie de aplicaciones que automaticen el análisis de las posibles vulnerabilidades presentes en un objetivo.

* Como pros: se hace un escaneo global y automático, evitamos hacer el escaneo puerto por puerto de forma manual.
* Como contras: genera mucho ruido. Puede no encontrar todas las vulnerabilidades, ejemplo: seguir leyendo. 

## NMAP
  NMAP presenta una serie de scripts categorizados que nos facilitan el trabajo. Una de esas categorías es la de vulnerabilidades, en concreto, `vuln` --> These scripts check for specific known vulnerabilities and generally only report results if they are found.

  Mediante ese script NMAP va a hacer un análisis de vulnerabilidades sobre el objetivo que indiquemos. Ejecutando el comando `sudo nmap -v -sS --script=vuln <ip_objetivo>` se empezará a realizar ese análisis. Al cabo de unos minutos se mostrará por consola toda la información recopilada. Es recomendable exportar este resultado para que sea más legible, para ello, ejecutamos el siguiente comando `sudo nmap -v -sS --script=vuln --stylesheet="https://svn.nmap.org/nmap/docs/nmap.xsl" <ip_objetivo>`. Este comando producirá la siguiente salida:

![Nmap analisis de vulnerabilidades]({{ page.assets_path}}/Nmap-vulnerabilidades.png){: .shadow style="min-width: 70%;" }
_Nmap analisis de vulnerabilidades_

_**[IMPORTANTE]**_ --> Como dato curioso: hemos escaneado el mismo objetivo que escaneamos de forma manual, y en el puerto 21 nmap no ha sido capaz de encontrar la vulnerabilidad asociada con la aplicación `ProFTDP`. El uso de nmap con ese script para el análisis de vulnerabilidades no es infalible.

## Nessus
Es sin lugar a dudas la herramienta más potente que podemos encontrar para el análisis de vulnerabilidades tanto de sistemas como de entornos web, ya que dispone del mayor repositorio de pruebas de vulnerabilidades del mercado.
Desarrollada por la empresa Tenable, en cuya web podemos descargar la versión gratuita `Essentials`, la cual tiene funcionalidades limitadas.

Está compuesta por 2 componentes:
 * Un demonio, llamado `nessusd`
 * Un cliente web que nos permite interactuar con la aplicación.

### Puesta en marcha
Una vez descargado de su página oficial, debemos arrancar el demonio de la siguiente forma `/etc/init.d/nessusd start`. Una vez arrancado, podemos ir al puerto 8834 de nuestro localhost para acceder al cliente web.


Encontramos una serie de plantillas que nos provee Nessus para iniciar ciertas acciones, como se puede ver en la siguiente imagen.
![Nessus]({{ page.assets_path}}/Nessus1.png){: .shadow style="min-width: 70%;" }
_Nessus_

### Caso de uso
Si ejecutamos un escaneo de puertos sobre el rango 192.168.236.0/24 obtenemos los siguiente resultados:
![Nessus2]({{ page.assets_path}}/Nessus2.png){: .shadow style="min-width: 70%;" }
_Nessus2_


Si ejecutamos un análisis de vulnerabilidades sobre el mismo rango anterior, obtenemos los siguientes resultados:
![Nessus3]({{ page.assets_path}}/Nessus3.png){: .shadow style="min-width: 70%;" }
_Nessus3_
Al igual que nos pasó con nmap, no ha identificado la vulnerabilidad de la aplicación ProFTDP sobre el puerto 21. Esto se debe a que Nessus funciona mediante **plugins**, es decir, cada plugin se encarga de realizar búsquedas de un conjunto de vulnerabilidades. En este caso no está instalado. Esta es la razón por la que Nessus es tan utiliza, porque los plugins son creados por la comunidad y actualizados con mucha frecuencia.


Si queremos realizar un **escaneo exhaustivo**, podemos hacerlo mediante la opción "Advanced Scan". Esta opción por defecto utiliza todos los plugins presentes en Nessus, lo cual nos permite analizar los objetivos de manera completa. La única desventaja de este escaneo es que es muy intrusivo. La salida de este "Advanced Scan" es la que podemos ver a continuación:
![Nessus4]({{ page.assets_path}}/Nessus4.png){: .shadow style="min-width: 70%;" }
_Nessus4_
Como se puede comprobar, ahora arroja una gran cantidad de vulnerabilidades. Esta es una forma rápida y sencilla de analizar nuestros objetivos. Si queremos exportar el resultado del análisis, usar el botón "Export" en la esquina superior derecha.

## Otras herramientas
La más conocida y utilizada es Nessus, pero hay otras alternativas:

| Nombre | Propietario| Licencia | Plataforma |
| ------------- |:-------------:| :-----:| :-----:|
|Acunetix|Acunetix|Comercial/gratis| Windows/Linux|
|AppScan|IBM|Comercial| Windows|
|Qualys|Qualys|Comercial| Cloud|
