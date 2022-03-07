---
title: Explotación de vulnerabilidades en Red
author:
  name: Javier Fernández Santos
  link: https://github.com/JaviFS97
date: 2021-07-10 017:35:00 +0800
categories: [Ciberseguridad, Seguridad Ofensiva]
tags: [Pentesting, Bettercap, ARPspoofing, DNSspoofing, SET]
math: true
mermaid: true
assets_path: /assets/img/posts/20210713

---
Durante los posts anteriores hemos visto métodos para recopilar y analizar información sobre nuestros objetivos. Durante este post veremos como explotar (en Red) aquellas vulnerabilidades que presentan.


# Contenido
- [Man In The Middle (MITM)](#man-in-the-middle-mitm)
  * [ARP Spoofing](#arp-spoofing)
    + [Caso de uso](#caso-de-uso)
  * [DNS Spoofing](#dns-spoofing)
    + [Caso de uso](#caso-de-uso-1)
  * [Social Engineering Toolkit (SET)](#social-engineering-toolkit-set)
    + [Caso de uso](#caso-de-uso-2)



# Man In The Middle (MITM)
Es un ataque en el que se adquiere la capacidad de leer, insertar y modificar a voluntad. El atacante debe ser capaz de observar e interceptar mensajes entre las dos víctimas y procurar que ninguna de las víctimas conozca que el enlace entre ellos ha sido violado.

![Esquema MITM]({{ page.assets_path}}/MITM.png){: .shadow style="min-width: 70%;" }
_Esquema MITM_

## ARP Spoofing
La suplantación de ARP (en inglés ARP spoofing) es enviar mensajes ARP falsos a la Ethernet. Normalmente la finalidad es asociar la dirección MAC del atacante con la dirección IP de otro nodo (el nodo atacado), como por ejemplo la puerta de enlace predeterminada. Cualquier tráfico dirigido a la dirección IP de ese nodo, será erróneamente enviado al atacante, en lugar de a su destino real. El atacante, puede entonces elegir, entre reenviar el tráfico a la puerta de enlace predeterminada real (ataque pasivo o escucha), o modificar los datos antes de reenviarlos (ataque activo).

### Caso de uso
Vamos a envenenar la tabla ARP de la máquina objetivo (ip 192.168.216.129) para que el tráfico no pase directamente por la puerta de enlace predeterminada (el router, con ip 192.168.216.2), sino que pase por nuestra máquina (192.168.216.128). En la siguiente imagen podemos ver cual es el estado del que se parte y el estado al que se quiere llegar.
![Estado anterior y posterior al MITM]({{ page.assets_path}}/ARPspoofing1.png){: .shadow style="min-width: 70%;" }
_Estado anterior y posterior al MITM_


Antes de realizar el ataque, vamos a comprobar una serie de cosas en la máquina Windows:
1. Comprobamos cual es la IP de la máquina Windows y vemos cual es el contenido de la tabla ARP que tiene almacenada en caché. Podemos ver que la máquina Kali y el gateway tienen direcciones MAC distintas.
![Tabla ARP de la máquina Windows (la objetivo)]({{ page.assets_path}}/ARPspoofing2.png){: .shadow style="min-width: 70%;" }
_Tabla ARP de la máquina Windows (la objetivo)_

2. Ejecutamos un traceroute para observar las rutas que toma la máquina. En este caso podemos ver que el primer salto que realiza la máquina Windows es contra el gateway (192.168.216.2).
![Traceroute (antes del MITM) desde la máquina Windows]({{ page.assets_path}}/ARPspoofing3.png){: .shadow style="min-width: 70%;" }
_Traceroute (antes del MITM) desde la máquina Windows_


Visto el estado anterior al ataque MITM. Ahora nos desplazamos a la máquina Kali. En ella vamos a usar la herramienta Bettercap para realizar el ataque de envenenamiento. Los pasos a seguir son:
1. Ejecutamos la herramienta con permisos de administrador.
2. Indicamos los objetivos del ataque de envenenamiento, en nuestro caso sobre la máquina Windows (192.168.216.129).
3. Lanzamos el ataque mediante el comando "arp.spoof on".
4. Colocamos un sniffer, en este caso wireshark, para comprobar el tráfico que se genera tras la ejecución del ataque.

![Ejecución del ataque MITM mediante Bettercap]({{ page.assets_path}}/ARPspoofing4.png){: .shadow style="min-width: 70%;" }
_Ejecución del ataque MITM mediante Bettercap_

En la captura anterior podemos extraer mucha información, vamos a analizarla.

En la esquina inferior derecha tenemos una consola donde hemos ejecutado la herramienta Bettercap con lo comentado anteriormente. Hemos colocado como objetivo la máquina windows (192.168.216.129) y posteriormente hemos lanzado el ataque. Esto a generado, como se puede ver en wireshark, una inundación de paquetes ARP desde la máquina kali (MAC 00:0c:29:a7:95:6b) hacía la máquina Windows (MAC 00:0c:29:d9:4b:01). El contenido de ese mensaje es el siguiente: “192.168.216.2 is at 00:0c:29:a7:95:6b”. En esto consiste el envenenamiento, en hacerle pensar a la máquina objetivo que eres el router o gateway (que se encuentra en 192.168.216.2).

Por último, para comprobar si el ataque ha sido exitoso, volvemos a la máquina Windows para hacer algunas comprobaciones.

Comprobamos el contenido de la tabla ARP que tiene en caché y vemos que la máquina Kali ahora comparte la dirección MAC con el gateway de la red.

Si ejecutamos un traceroute, podemos observar que el primer salto que realiza la máquina Windows es contra la máquina del atacante, y después es redireccionado al gateway.
![Tabla ARP + Traceroute (después del MITM) desde la máquina Windows]({{ page.assets_path}}/ARPspoofing5.png){: .shadow style="min-width: 70%;" }
_Tabla ARP + Traceroute (después del MITM) desde la máquina Windows_



## DNS Spoofing 
La suplantación de DNS, también conocida como envenenamiento de la caché de DNS, es una forma de piratería informática en la que se introducen datos corruptos del Sistema de Nombres de Dominio en la caché del resolvedor de DNS, haciendo que el servidor de nombres devuelva un registro de resultados incorrecto, por ejemplo, una dirección IP. Esto hace que el tráfico se desvíe al ordenador del atacante (o a cualquier otro ordenador)

### Caso de uso
Con el ARP Spoofing que hicimos antes conseguimos estar entre medias de la comunicación, pero no sacamos ningún uso a esta ventaja. Ahora lo que haremos será crear una web falsa que sea devuelta cuando el usuario objetivo quiera entrar en la web de facebook.

Para ello, debemos volver a realizar ARP Spoofing. En la siguiente imagen podemos comprobar la ruta que se sigue desde el ordenador objetivo cuando realiza una petición DNS de google. Vemos que antes de pasar por el router pasa por el ordenador del atacante.

![Tabla ARP + Traceroute desde la máquina Windows]({{ page.assets_path}}/DNS-Spoofing1.png){: .shadow style="min-width: 70%;" }
_Tabla ARP + Traceroute desde la máquina Windows_


En la máquina del atacante configuramos el ataque de DNS Spoofing. Para ello, indicamos que se haga uso del módulo dns.spoof, asignándole como ruta `facebook.es` y la dirección de nuestra máquina atacante `192.168.239.129`, donde tendremos un servidor apache levantado con una web muy básica que contendrá el siguiente contenido `Visitando la página web del atacante`.
![Uso de Bettercap para DNS spoofing]({{ page.assets_path}}/DNS-Spoofing2.png){: .shadow style="min-width: 70%;" }
_Uso de Bettercap para DNS spoofing_

Lanzamos el módulo. Vemos que la dirección de facebook queda asociada a la ip del atacante.

![Uso de Bettercap para DNS spoofing]({{ page.assets_path}}/DNS-Spoofing3.png){: .shadow style="min-width: 70%;" }
_Uso de Bettercap para DNS spoofing_

Accedemos a la máquina objetivo, usando un navegador visitamos la página de facebook. Como se puede comprobar, devuelve la página alojada en la máquina atacante.
![Navegando a Facebook desde la máquina objetivo]({{ page.assets_path}}/DNS-Spoofing4.png){: .shadow style="min-width: 70%;" }
_Navegando a Facebook desde la máquina objetivo_

## Social Engineering Toolkit (SET)
SET es una completísima suite dedicada a la ingeniería social , que nos permite automatizar tareas que van desde el de envío de SMS (mensajes de texto) falsos, con los que podemos suplantar el numero telefónico que envía el mensaje, a clonar cualquier pagina web y poner en marcha un servidor para hacer phishing en cuestión de segundos.
 
El kit de herramientas SET está especialmente diseñado para realizar ataques avanzados contra el elemento humano. Originalmente, este instrumento fue diseñado para ser publicado con el lanzamiento de http://www.social-engineer.org y rápidamente se ha convertido en una herramienta estándar en el arsenal de los pentesters.

### Caso de uso
Sería realizar el mismo procedimiento que en el caso de uso del apartado de DNS Spoofing, con la diferencia de clonar el login de facebook. Esto se haría por medio de la Social Engineering Toolkit.

