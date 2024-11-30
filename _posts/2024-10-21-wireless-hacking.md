---
title: Hacking en Redes Wi-Fi - Vulnerabilidades y Cómo Protegerte
description: Hacking a redes Wi-Fi usando la suite de aircrack-ng para descifrar la contraseña de la red en cuestión de segundos.
date: 2024-10-21
categories: [hacking, Wi-Fi, redes]
tags: [hacking-wifi, aircrack-ng, airmon-ng, aireplay-ng, airodump-ng, proteger, ap, router, WPA2, WPS, WPA3, cracking, fuerza-bruta, handshake, diccionario]
img_path: /assets/img/wirelesshack.png
image: /assets/img/wirelesshack.png
---

## **Introducción**
Hoy en día, todos tenemos una red Wi-Fi en nuestro hogar y, generalmente, el router lo dejamos desprotegido ya sea porque no sabemos configurarlo o porque tenemos miedo de liarla y quedarnos sin conexión. Hoy veremos como nuestra red, si no está bien protegida, **puede ser totalmente vulnerable** frente a cualquier ciberdelincuente que quiera realizar acciones maliciosas en ella.

Veremos algunos tipos de ataques y como protegernos de ello para estar lo más seguros posibles en nuestra red y despreocuparnos de cualquier persona malintencionada.

## **WPA2 y Handshake**
El término de *4-way handshake*, se refiere a la **secuencia de mensajes** que se intercambian el router y el dispositivo, con el fin de establecer una conexión **segura y autenticada**. Durante la comunicación, **no se transmite la contraseña directamente**, sin embargo, viaja el denominado *handshake* que, a grandes rasgos, es la contraseña cifrada. En caso de que un atacante lo intercepte, podrá realizar técnicas de **cracking** para intentar descifrarlo.

**WPA2**, es el protocolo que se usa hoy en día en las conexiones Wi-Fi (aunque se está implementando el WPA3). Su función principal es **proteger la red y cifrar** los datos que se envían y reciben, evitando que terceros intercepten o accedan a la información.

## **Manos a la obra**
Como víctima tendremos la red *Empleados* que está configurada en el **AP** (*Access Point*). En todo momento, como tarjeta de red estaré usando el adaptador de red *[ALFA Network AWUS036NHA](https://www.tienda-alfanetwork.com/alfa-awus036nha-antena-wifi-usb-atheros-ar9271.html)*.

Como primer paso, debemos instalar la suite de `aircrack-ng` que nos proporciona diversas herramientas para realizar auditorías en la red. Para instalarla, podemos usar el siguiente comando.

```bash
sudo apt install aircrack-ng
```
A continuación, pondremos la interfaz a la cual está conectado el adaptador de red, en modo monitor. Esto, nos permitirá interceptar y analizar los paquetes que viajan en la red. En mi caso, la interfaz correspondiente es la **wlan0**.

```bash
sudo airmon-ng start wlan0
```
De esta manera, si ejecutamos el comando `iwconfig` veremos como nuestra interfaz de red, se le ha puesto el prefijo *mon*. Esto significa que dicha interfaz , en este momento se encuentra en modo monitor.

```bash
wlan0mon: flags=867<UP,BROADCAST,NOTRAILERS,RUNNING,PROMISC,ALLMULTI>  mtu 1500
        unspec XX-XX-XX-XX-XX-XX  txqueuelen 1000  (UNSPEC)
        RX packets 46661  bytes 6043966 (5.7 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

Ahora, podemos ponernos a escuchar las redes disponibles en los alrededores de nuestro lugar. Para ello, usaremos el comando `airodump-ng` seguido de nuestra interfaz.

```bash
sudo airodump-ng wlan0mon
```

![Resultado de la ejecución del comando sudo airodump-ng wlan0mon](/assets/img/airodump-ng-output.png)

Hemos encontrado nuestro objetivo, por lo que ahora tenemos que intentar obtener el *handshake* que contiene la clave cifrada y `airodump-ng` nos permite esto mismo. Esta vez, lo ejecutaremos de una manera diferente. Esta vez, tenemos que copiar el *bssid* de la red objetivo y el canal por el que transmite.

Si nos fijamos en la captura anterior, observamos que hay diferentes columnas los cuales contienen datos como:

- **BSSID** ➜ Dirección MAC de la red.
- **Beacons** ➜ Mensajes que transmite el AP para anunciar su presencia en la red.
- **CH** ➜ Canal por el que transmiten.
- **ENC** ➜ Tipo de encriptado que usan.
- **AUTH** ➜ Tipo de autenticación que usan.
- **ESSID** ➜ Nombre de la red.

Una vez que tengamos el *bssid* correspondiente, monitorizaremos la red del punto de acceso. Esto, nos dará información como las estaciones que están conectadas. Lo deberemos de dejar en segundo plano hasta conseguir el *handshake*.

```bash
sudo airodump-ng --bssid "E6:XX:XX:XX:65:C4" wlan0mon
```

![Output de airodump-ng sin el Handshake](/assets/img/monitorizando-la-red.png)

Para obtener el *handshake*, tenemos dos opciones:

1. Esperar a que **alguien se conecte** a la red.
2. Efectuar un **ataque deauth** o desautenticación.

Podemos optar por la primera opción en casos en los que la conexión y desconexión de dispositivos a la red es bastante continua, como lo son en las Wi-Fi públicas. En este blog, estaremos efectuando un **ataque de desautenticación**.

Para realizar este ataque tenemos de nuevo a `aireplay-ng`. Con el parámetro `-0`, le estaremos indicando cuantos **paquetes de desautenticación** queremos enviar y, con el parámetro `-c`, deberemos de introducirle la dirección **MAC del cliente** que recibirá estos paquetes de desautenticación.

Para obtener la MAC del cliente, tendremos que fijarnos en el campo *`STATION`* de la captura anterior.

![Dirección MAC de la estación víctima](/assets/img/MAC-estacion.png)

Con esta información obtenida, procederemos a efectuar el ataque.

```bash
sudo aireplay-ng -0 5 -a "E6:XX:XX:XX:65:C4" -c "D2:XX:XX:XX:A6:AE" wlan0mon
```

![Ataque deauth a la estación conectada al AP](/assets/img/deauth-attack.png)

Ahora, si volvemos a la monitorización de la red, veremos cómo hemos obligado a la estación a **desconectarse y volver a conectarse** a la red, lo que ha resultado en el *4-way handshake* que hemos conseguido interceptar. Si no hemos logrado capturarlo, no te preocupes, podemos realizar el ataque nuevamente, ya que en ocasiones no se obtiene el *handshake* a la primera.

![Obtención del handshake después de efectuar un deauth a la estación](/assets/img/wpa-handshake.png)

En este punto, podemos parar de monitorizar la red y proceder efectuar un ataque de fuerza bruta con el fin de obtener la contraseña en texto plano. Para ello, usaremos `aircrack-ng` y con el parámetro `-w` le indicaremos la ruta del diccionario de contraseñas. Al final del comando, le tenemos que introducir el fichero `.cap` que nos ha generado durante el tiempo que hemos estado monitorizando la red.

```bash
sudo aircrack-ng -w /usr/share/wordlists/rockyou.txt -b "E6:XX:XX:XX:65:C4" output-01.cap
```

Después de un tiempo, si la contraseña de la red está presente en el diccionario que hemos especificado, lograremos obtenerla (en este caso, es *elemental*)

![Cracking al handshake obtenido](/assets/img/wifi-key-found.png)

De esta manera, hemos conseguido **obtener la contraseña** del punto de acceso mediante un **ataque de desautenticación** contra una estación que ya estaba conectada a la red. Este ataque la **obligó a reconectarse**, lo que nos permitió interceptar el *4-way handshake* necesario para descifrar la contraseña.

Para verificar si la contraseña que hemos encontrado es válida, utilizaremos la herramienta [netpw](https://github.com/h3g0c1v/netpw). Esta herramienta nos permite **comprobar la validez de la contraseña** en la red Wi-Fi y también realizar ataques de **fuerza bruta** a partir de un diccionario.

```bash
./netpw -n "Empleados" -p "elemental"
```

![Comprobando la credencial encontrada con netpw](/assets/img/comprobar-credencial-netpw.png)

Ahora sí, podemos decir que hemos conseguido vulnerar la red Wi-Fi.

## **Como protegernos**
Hasta ahora, hemos estado viendo como vulnerar una red Wi-Fi, pero igual de importante es entender cómo proteger nuestras redes de estos ataques. A continuación, revisaremos varias prácticas esenciales que puedes implementar para aumentar la seguridad de tu red Wi-Fi y minimizar las probabilidades de sufrir un ataque:

1. **Utiliza WPA3** ➜ Si tu router lo permite, el cifrado **WPA3** es la opción más segura disponible hoy en día para proteger la red.
2. **Cambia el SSID y la contraseña por defecto** ➜ Los routers suelen venir con un **SSID** y una contraseña por defecto que, generalmente, son fáciles de adivinar o incluso encontrar en Internet.
3. **Desactiva el WPS (*Wifi Protected Setup*)** ➜ Aunque el **WPS** facilita la conexión de dispositivos en la red, es una vía muy común para ataques de fuerza bruta.
4. **Usa una contraseña segura** ➜ Tanto la clave de acceso al router como la de la red, suelen ser bastante vulnerables. Por ello, es fundamental configurar una **contraseña robusta** para aumentar la seguridad.
5. **Monitorea tu red regularmente** ➜ Monitorear los dispositivos conectados a tu red te permitirá detectar cualquier **acceso sospechoso** y/o **no autorizado**.

## **Despedida**
Muchas gracias por haber dedicado su tiempo en leer este blog. Espero que le haya servido de ayuda. ¡Recuerda, la seguridad en las redes es esencial para mantener tus conexiones protegidas!"
