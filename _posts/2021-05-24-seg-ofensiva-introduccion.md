---
title: Introducción al Hacking Ético y Penetration Testing
author:
  name: Javier Fernández Santos
  link: https://github.com/JaviFS97
date: 2021-05-24 12:37:00 +0800
categories: [Ciberseguridad, Seguridad Ofensiva]
tags: [Pentesting, OSSTMM]
math: true
mermaid: true
assets_path: /assets/img/posts/20210525

image:
  src: /assets/img/posts/20210524/ethical-hacking.jpg
  width: 1000   # in pixels
  height: 400   # in pixels
  alt: image alternative text

---
La protección de los sistemas requiere una comprensión amplia de las estrategias de ataque y un conocimiento profundo de las tácticas, herramientas y motivaciones del ciberdelincuente.

En este curso recopilaremos las principales herramientas y técnicas aplicadas en cada uno de las fases del pentesting.

# Índice y Estructura Principal
- [Introducción](#introducción)
- [Metodologías](#metodologías)
  + [Metodologías principales](#metodologías-principales)
  + [Metodología a seguir en el curso](#metodología-a-seguir-en-el-curso)
- [Definición del alcance del hacking ético](#definición-del-alcance-del-hacking-ético)


# Introducción
A la hora de llevar a cabo una actividad de hacking ético es importante el uso de metodologías previamente definidas para aumentar las probabiliades de éxito. Por ello, es importante conocer qué es una metodología, por qué es relevante que las apliquemos a nuestra forma de trabajo y también cuáles son algunas de las más relevantes.

# Metodologías
Las metodologías nos facilitan la realización de un conjunto de actividades en un orden determinado y estableciendo una prioridad adecuada para intentar garantizar el éxito y alcanzar un objetivo final.
La salida de una fase se va a corresponder con la entrada de la siguiente fase.

## Metodologías principales
* OSSTMM (Open-Source Security Testing Methodology Manual): la más utilizada.
* The Penetration Testing Execution Standard: quizás algo anticuada.
* ISSAF (Information System Security Assement Framework)
* OTP (OWASP Testing Project)

## Metodología a seguir en el curso
Como no se está trabajando sobre un entorno real no es necesario aplicar todas las fases. En nuestro caso, vamos a saltarnos el primer y último punto.
1. ~~Definición del alcance del test de penetración.~~
2. Recopilación de información. (Pasiva, semi-pasiva y activa)
3. Identificación y análisis de vulnerabilidades.
4. Explotación de las vulnerabilidades. (En host, en app. web y en redes)
5. Post-explotación.
6. ~~Elaboración de un documento de reporte.~~


# Definición del alcance del hacking ético
Si se realizase la definición del alcance (nosotros no la vamos a realizar) habría que tener en consideración los siguientes aspectos:
* Antes de realizar ninguna acción, discutir con el cliente las tareas que llevará a cabo el analista, así como los roles y la responsabilidad de ambos.
* Asegurar mediante contrato firmado que las acciones que se llevan a cabo son en representación del cliente.
* Análisis de las políticas de la organización que definen el uso que los usuarios hacen de los sistemas y de la infraestructura.
* Procedimiento en caso de localizar una intrusión de un tercero.


Lista de todos los apartados que componen el curso:
<div class="posts">
  <ol>
  {% for post in site.categories['Seguridad Ofensiva'] reversed %}
    <article class="post">
      <li>
        <a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a>
      </li>
    </article>
  {% endfor %}
  </ol>
</div>
