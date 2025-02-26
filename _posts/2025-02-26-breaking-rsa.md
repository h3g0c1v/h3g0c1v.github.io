---
title: Factoring n to Derive the Private Key - Breaking RSA 
description: La seguridad de RSA radica en el problema de la factorización de dos números enteros.
date: 2025-02-26
categories: [reversing, ingenieria-inversa, breaking-rsa]
tags: [factorizar, par-de-claves, clave-privada, clave-publica, rsa, funcion-modular-multiplicativa-inversa]
img_path: /assets/img/breaking-rsa.png
image: /assets/img/breaking-rsa.png
---

## **Introducción**

El algoritmo de encriptación **RSA** opera utilizando números primos grandes elegidos al azar (mantenidos en secreto) y, la seguridad de este algoritmo radica en el problema de la **[factorización](https://es.wikipedia.org/wiki/Factorizaci%C3%B3n) de dos números enteros (*`p`* y *`q`*)**. De forma que, dados dos valores (*`p`* y *`q`*) multiplicados entre sí, devuelven el valor de *`n`*.

Se cree que **RSA** será seguro mientras no se conozcan formas rápidas de **factorizar un número grande (*`n`*)**, es decir, ser capaces de obtener esos valores *`p`* y *`q`*, que han sido seleccionados al azar para generar la clave. Una clave **RSA**, está compuesta por valores llamados:

- **`n`** ➜ Es el resultado de multiplicar ***`p`*** y ***`q`*** (*`p` * `q`*).
- **`e`** ➜ Es el *exponente de cifrado*, y generalmente su valor es *65537*. Para la clave pública se usa este valor *`e`* como exponente de *`n`*.
- **`d`** ➜ Es el *exponente de descifrado*. Para la clave privada se usa este valor *`d`* como exponente de *`n`*.
- **`p`** y **`q`** ➜ Son *números primos* que multiplicados entre si, forman *`n`* (son *factores de `n`*).

Como todo sistema de par de claves, cada usuario posee dos claves de cifrado: una *publica* (se comparte con todos) y otra *privada* (se mantiene en secreto). Cuando se quiere enviar un mensaje confidencial, el **emisor** (el que envía el mensaje) busca la **clave pública** del **receptor** (el que recibe el mensaje), y una vez que el mensaje cifrado llega al receptor, este se ocupa de **descifrarlo** usando **su clave privada**.

En el caso de saber, cual es el valor de **`p`** y **`q`** podríamos ser capaz de descifrar estos mensajes que anteriormente fueron cifrados ya que, tendríamos la capacidad de reconstruir la clave privada (con la que descifrar estos mensajes) a partir de la clave pública.

### **Funcionamiento básico del cifrado con clave pública**

El proceso de **cifrado y descifrado** funciona de la siguiente manera. Lo primero que debemos de hacer es *cifrar el mensaje* que queremos enviar:

1. El **remitente** (el que envía el mensaje) cifra el mensaje con la **clave pública del destinatario** (el que recibe el mensaje).
2. Una vez cifrado, **solo la clave privada del destinatario** (el que recibe el mensaje) **puede descifrarlo**.

Para *descifrar el mensaje que ha sido cifrado* anteriormente:

1. El **destinatario** (el que recibe el mensaje) usa **su clave privada** para **descifrar** el mensaje y leerlo.
2. **Nadie más puede hacerlo porque la clave privada es secreta**.

Entonces, supongamos que **Ana** quiere *enviar un mensaje secreto* a **Carlos**. Entonces, Carlos tiene una clave pública, que comparte con todos y una clave privada, que solo él conoce.

1. **Ana** usa la **clave pública de Carlos** para *cifrar* su mensaje.
2. **Ana envía el mensaje**  cifrado a Carlos.
3. **Carlos** usa su **clave privada** para *descifrar* el mensaje y leerlo.

Por lo que, **solo Carlos puede descifrarlo** ya que solo él tiene su clave privada. En la siguiente imagen, podemos verlo de una manera más visual el *[funcionamiento basico del cifrado con clave publica](https://excalidraw.com/#json=ccoJLi4-oRUpvjjcVgZpW,LTJkQ83t189qsb4N31RWUA)*.

<img src="/assets/img/Funcionamiento-basico-del-cifrado-con-clave-publica.png" style="border: 1px solid black; max-width: 400px"/>

## **Configuración del Laboratorio**

Para entender mejor como podemos llegar a romper *RSA* vamos a ver un ejemplo práctico, y para ello cifraremos un mensaje para después descifrarlo sin tener de primeras la clave privada. Las claves que vamos a estar generado, surgirán de **valores primos muy pequeños**, y es ahí donde surge esta vulnerabilidad que permite romper el cifrado RSA.

Para obtener dos valores primos (*`p`* y *`q`*) pequeños, vamos a coger cualquier valor de la [siguiente página web](https://t5k.org/lists/small/small.html). Esta [página](https://t5k.org/lists/small/) nos proporciona diferentes números primos y, dependiendo de la longitud que escojamos, nos mostrará números más, o menos grandes. A más grande sea estos números primos, mayor dificultad tendremos para después factorizar y obtener el valor de *`n`*.

Para este ejemplo, estaremos escogiendo los siguientes valores de *`p`* y *`q`*.

```python
p = 671998030559713968361666935769
q = 2425967623052370772757633156976982469681
```

Por lo que siguiendo el concepto que hemos estado comentado, la multiplicación de ambos valores dará como resultado *`n`*.

```python
n = p*q
```

Con estos valores, generaremos una clave privada usando la librería `Crypto` de *Python*. Para generar la clave privada, necesitaremos los valores *`n`*, *`e`*, *`d`*, *`p`* y *`q`*, tal y como mencionamos en la introducción.

### **Definiendo los valores para generar la clave privada**

El valor de *`e`*, siempre suele ser el mismo (*65537*), así que lo definiremos en nuestro código.

```python
e = 65537
```

El valor de *`d`*, surge de aplicar la *función modular multiplicativa inversa* de los valores de *`e`* y *`m`*. El valor de *`m`* se define como *`n-(p+q-1)`*, y los valores *`n`*, *`p`* y *`q`* ya los tenemos definimos anteriormente.

```python
m = n-(p+q-1)
d = modinv(e, m)
```

La [*función modular multiplicativa inversa de `e` y `m`*](https://stackoverflow.com/questions/4798654/modular-multiplicative-inverse-function-in-python) (*`modinv(e, m)`*), realizará las operatorias necesarias para devolvernos el valor correcto de *`d`*.

```python
def egcd(a, b):
    if a == 0:
        return (b, 0, 1)
    else:
        g, y, x = egcd(b % a, a)
        return (g, x - (b // a) * y, y)

def modinv(a, m):
    g, x, y = egcd(a, m)
    if g != 1:
        raise Exception('modular inverse does not exist')
    else:
        return x % m
```

De esta manera, podemos generar la clave privada con los valores definidos con anterioridad.
```python
key = RSA.construct((n, e, d, p, q))
print(key.exportKey().decode())
```

El código completo lo podemos obtener en [Generación de una clave privada con números primos muy pequeños](/assets/resources/privateKeyGenerated.py.md)

### **Obteniendo la clave privada**

Al ejecutar el script, deberemos de ver una salida como la siguiente.

![clave privada RSA generada.png](/assets/img/clave-privada-RSA-generada.png)

Nos guardaremos esta clave privada en un fichero con nombre `private.pem`.

```bash
python3 privateKeyGenerated.py | tail -n +4 > private.pem
```

![guardando la clave privada en un archivo.png](/assets/img/guardando-la-clave-privada-en-un-archivo.png)

### **Obteniendo la clave pública**

De una clave privada, podemos obtener su clave pública. Con esta **clave pública**, tendremos la capacidad de **cifrar** los mensajes para después, con la *clave privada, poder descifrar* dicho mensaje. Usaremos `openssl` para obtener la clave pública de la clave privada.

```bash
openssl rsa -in private.pem -pubout -out public.pem 2>/dev/null
```

![clave publica generada openssl.png](/assets/img/clave-publica-generada-openssl.png)

### **Analizando las claves generadas**

A partir de la **clave pública**, los valores *`n`* y *`e`* podemos verlos.

```python
from Crypto.PublicKey import RSA
f = open("public.pem")
key = RSA.importKey(f.read())
key.n
key.e
```

![visualizando valores n y e de la clave publica.png](/assets/img/visualizando-valores-n-y-e-de-la-clave-publica.png)

Sin embargo, los valores *`p`* y *`q`* no somos capaces de verlo ya que, en caso contrario, supondría un problema de seguridad en la criptografía de **RSA**. Entonces, a priori no deberíamos ser capaces de obtener estos *factores de `n`*.

```python
key.p
key.q
```

![intentando ver los valores p y q de la clave publica.png](/assets/img/intentando-ver-los-valores-p-y-q-de-la-clave-publica.png)

No obstante, si nos abrimos la **clave privada** si seremos capaz de ver todos los valores que hemos usado para generarla ya que, en un principio, esta clave privada no debería de poder verlo nadie más que su propietario.

```python
from Crypto.PublicKey import RSA
f = open("private.pem")
key = RSA.importKey(f.read())
key
```

![visualizando los valores de la clave privada.png](/assets/img/visualizando-los-valores-de-la-clave-privada.png)

Por lo que teniendo los valores *`p`* y *`q`*, tenemos la capacidad de calcular *`n`*.

```python
p = key.p
q = key.q
n = p*q
n
```

![calculando el valor de n sabiendo p y q.png](/assets/img/calculando-el-valor-de-n-sabiendo-p-y-q.png)

### **Cifrando el mensaje**

A continuación, vamos a cifrar el mensaje que, posteriormente, intentaremos descifrar. Para ello, metemos en un archivo `.txt` el siguiente contenido.

```text
echo "Breaking RSA" > secret.txt
```

![mensaje para encriptar.png](/assets/img/mensaje-para-encriptar.png)

Para poder cifrar este contenido, es necesario emplear la **clave pública** que ha sido derivada de la **clave privada**.

```bash
openssl pkeyutl -encrypt -inkey public.pem -pubin -in secret.txt -out secret.encrypted
```

![mensaje encriptado con la clave publica RSA.png](/assets/img/mensaje-encriptado-con-la-clave-publica-RSA.png)

Por último, para terminar de configurar el laboratorio, borraremos tanto la clave privada, como el fichero `secret.txt` que contiene el mensaje en texto claro, del mensaje que hemos cifrado. Por lo que los únicos ficheros con los que nos tendremos que quedar son `public.pem` (clave pública) y `secret.encrypted` (mensaje cifrado).

![ficheros necesarios para el laboratorio breaking rsa.png](/assets/img/ficheros-necesarios-para-el-laboratorio-breaking-rsa.png)

## **Rompiendo RSA**

Supongamos que hemos conseguido de alguna manera, llegar a poder visualizar la clave pública, y en mitad de la comunicación interceptamos un mensaje que está cifrado con dicha clave pública. Entonces, nos encontramos con los ficheros `public.pem` (**clave pública**) y `secret.encrypted` (**mensaje cifrado**).

![ficheros necesarios para el laboratorio breaking rsa.png](/assets/img/ficheros-necesarios-para-el-laboratorio-breaking-rsa.png)

Al visualizar la clave pública `public.pem`, nos percatamos con que esta clave es demasiado pequeña. Esto nos da a entender que, los valores primos *`p`* y *`q`* que se han usado para generar la **clave privada** son demasiado pequeños, por lo que deberíamos de ser capaces de obtenerlos y **generar la clave privada a partir de la clave pública**, aplicando un poco de *Ingeniería Inversa* y/o *Reversing*.

![Visualizando Clave Publica](/assets/img/visualizando-clave-publica-public-pem.png)

Para ello, nos crearemos un script en *Python* que será capaz de generarnos esta clave privada. Tal y como se comentó con anterioridad, para generar una clave privada necesitamos los valores *`n`*, *`e`*, *`d`*, *`p`* y *`q`*, así que vamos a recabarlos.

Lo primero que haremos es sacar el valor de *`n`*. Para ello nos abriremos la clave pública, y con la librería `Crypto` podremos mostrar dicho valor.

```python
from Crypto.PublicKey import RSA
f = open("public.pem")
key = RSA.importKey(f.read())
key.n
```

![obteniendo el valor de n a través de la clave publica.png](/assets/img/obteniendo-el-valor-de-n-a-traves-de-la-clave-publica.png)

Como ya sabemos, *`n`* es el resultado de **dos números primos** (*`p`* y *`q`*) multiplicados entre sí. Cuando el valor de *`n`* es muy pequeño, como es el caso, podemos ser capaces de obtener estos dos números primeros *`p`* y *`q`*. Para realizar la **factorización de ambos números** usaremos la página de **[FactorDB](https://factordb.com/)**, que realizará las operaciones necesarias para devolvernos dichos valores.

![valores de p y q a partir de n.png](/assets/img/valores-de-p-y-q-a-partir-de-n.png)

Si queremos ver el valor de cada uno, pincharemos sobre el que queramos ver y se nos pondrá en la barra superior (en el campo en el que pusimos el valor de *`n`*) desde donde lo podremos copiar. Ahora que tenemos estos valores *`p`* y *`q`*, los definiremos en nuestro script.

```python
p = 671998030559713968361666935769
q = 2425967623052370772757633156976982469681
n = p*q
```

El valor de *`e`* sabemos que es *65537*, por lo que también lo definiremos.

```python
e = 65537
```

El valor de `d` es el resultado de la *función modular multiplicativa inversa* de *`e`* y *`m`*. Y *`m`* es el resultado de *`n-(p+q-1)`*.

```python
m = n-(p+q-1)

def egcd(a, b):
    if a == 0:
        return (b, 0, 1)
    else:
        g, y, x = egcd(b % a, a)
        return (g, x - (b // a) * y, y)

def modinv(a, m):
    g, x, y = egcd(a, m)
    if g != 1:
        raise Exception('modular inverse does not exist')
    else:
        return x % m

d = modinv(e, m)
```

En este punto, ya sabemos todos los valores necesarios para mostrar la clave privada. Lo único que nos faltaría es construirla y visualizarla por pantalla.

```python
key = RSA.construct((n, e, d, p, q))

print("\n[+] Mostrando la clave privada\n")
print(key.exportKey().decode())
```

De esta manera, al ejecutar el script habremos sido capaces de, a partir de una **clave pública con valores primos muy pequeños**, conseguir sacar **su clave privada** para poder **descifrar el mensaje cifrado**.

```bash
python3 breakingRSA.py
```

![breaking RSA obteniendo la clave privada a partir de la publica.png](/assets/img/breaking-RSA-obteniendo-la-clave-privada-a-partir-de-la-publica.png)

Este contenido nos lo guardaremos en un fichero `private.pem`, y lo usaremos para descifrar el mensaje que anteriormente ciframos.

![descifrando mensaje cifrado breaking RSA.png](/assets/img/descifrando-mensaje-cifrado-breaking-RSA.png)

## **Enlaces de referencia**

Los siguientes enlaces, se han usado para el entendimiento y creación de esta explicación en la que se explica como romper el algoritmo de encriptación RSA.

- **[Máquina Yummy de Hack The Box - S4vitaar](https://www.youtube.com/watch?v=8A4HFycriHk)**.
- **[Explicación del funcionamiento de RSA - Wikipedia](https://es.wikipedia.org/wiki/RSA)**.
- **[Prime Numbers & RSA Encryption Algorithm - Computerphile](https://www.youtube.com/watch?v=JD72Ry60eP4&t=141s)**.
- **[Listas de números primos pequeños](https://t5k.org/lists/small/)**.
- **[Factorizador Online](https://factordb.com/)**.
- **[Esquema del funcionamiento básico de un algoritmo de clave pública - Excalidraw](https://excalidraw.com/#json=ccoJLi4-oRUpvjjcVgZpW,LTJkQ83t189qsb4N31RWUA)**.

## **Conclusión**

Debido a que se han usado **números primos demasiado pequeños** para generar la clave privada, se han podido **factorizar** y obtener el valor de *`p`* y *`q`* los cuales, han permitido **volver a generar la clave privada** que se ha usado para generar la clave pública. De esta manera, teniendo en posesión la clave privada podemos **descifrar** el mensaje que, en un principio, únicamente debería de ser capaz de descifrar el propietario de la clave privada.
