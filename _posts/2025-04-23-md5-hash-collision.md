---
title: El Problema de las Colisiones en MD5
description:  MD5 es un algoritmo criptográfico ampliamente utilizado, pero sus colisiones lo hacen vulnerable.
date: 2025-04-23
categories: [reversing, breaking-md5, criptografia]
tags: [md5, hash-collisions]
img_path: /assets/img/MD5-Hash-Collision.png
image: /assets/img/MD5-Hash-Collision.png
---

## **Introducción**

**MD5** es un algoritmo que convierte cualquier texto en un *hash* de 32 caracteres. Sin embargo, como el número de posibles hashes es *limitado* y existen **infinitas combinaciones** de texto, es posible que dos textos distintos generen el **mismo hash**. A esto se le llama [**colisión**](https://es.wikipedia.org/wiki/Colisi%C3%B3n_(hash)), y hoy en día, ya es posible generar colisiones de forma intencionada, lo que hace que MD5 ya no sea considerado seguro para fines criptográficos o de integridad.

## **Crypto Challenge**
### **Teniendo un primer contacto**

Tenemos el siguiente [Script MD5 Hash Collision Server](/assets/resources/md5_hash_collision.py.md), el cual analizaremos para entender donde reside la vulnerabilidad. Este script podemos encontrarlo en el apartado de [Challenges de Hack The Box](https://app.hackthebox.com/challenges/alphascii%2520clashing).

Al conectarnos a la instancia, podemos ver el siguiente menú que se encuentra en la función *get_option* del script.

![menu md5 hash collision script.png](/assets/img/menu-md5-hash-collision-script.png)

Nos dice que el formato que debemos proporcionar es *Json*, por lo que analizando el script vemos que hay un parámetro "*option*", donde le tendremos que especificar que opción queremos seleccionar. Además, podemos identificar las opciones *login* y *register*, donde en la primera tendremos la posibilidad de iniciar sesión con un usuario existente, y la segunda registrar un nuevo usuario.

```python
if 'option' not in option:
    print('[-] please, enter a valid option!')
    continue

option = option['option']

if option == 'login':
...

elif option == 'register':
...
```

Teniendo esto en cuenta, podemos intentar iniciar sesión con los usuarios que se muestran en la parte superior del script.

```python
users = {
    'HTBUser132' : [md5(b'HTBUser132').hexdigest(), 'secure123!'],
    'JohnMarcus' : [md5(b'JohnMarcus').hexdigest(), '0123456789']
}
```

Para ello, con el formato *Json* adecuado seleccionaremos la opción de **iniciar sesión**.

```json
{"option":"login"}
```

![option login script md5 hash collision.png](/assets/img/option-login-script-md5-hash-collision.png)

Nos solicita introducir las credenciales del usuario, pero aún no conocemos con certeza qué parámetros debemos proporcionarle. Por ello, continuaremos analizando el script, centrándonos específicamente en la sección correspondiente a la opción de login, para intentar identificar cómo se procesan estos datos y qué estructura espera recibir.

```python
if option == 'login':
    creds = json.loads(input('enter credentials (json format) :: '))

    usr, pwd = creds['username'], creds['password']
```

Observamos que cuando entramos en la opción de inicio de sesión, nos muestra el mensaje que vimos anteriormente, "*enter credentials (json format)*", donde le tendremos que especificar las credenciales correspondientes. Como se muestra en el código, existen las opciones *username* y *password* con las que le indicaremos dichas credenciales.

```json
{"username":"HTBUser132","password":"secure123!"}
```

![iniciando sesion como htbuser132 script md5 hash collision.png](/assets/img/iniciando-sesion-como-htbuser132-script-md5-hash-collision.png)

### **Análisis del código**

Para entender porque pasa esto, vamos a analizar esta parte del código:

```python
usr, pwd = creds['username'], creds['password']
usr_hash = md5(usr.encode()).hexdigest()
```

La funcionalidad de este código es la siguiente:

- Como hemos visto, los datos se almacenan en un diccionario de *clave-valor* como el siguiente: *`'HTBUser132' : [md5(b'HTBUser132').hexdigest(), 'secure123!']`*.
- *`usr, pwd = creds['username'], creds['password']`* ➜ Primero almacena en `usr` el nombre de usuario, y en `pwd` su contraseña.
- *`usr_hash = md5(usr.encode()).hexdigest()`* ➜ En `usr_hash` almacena el propio usuario en formato MD5.

Después el script continua con la siguiente porción de código:

```python
for db_user, v in users.items():
	if [usr_hash, pwd] == v:
	    if usr == db_user:
	        print(f'[+] welcome, {usr} 🤖!')
	    else:
	        print(f"[+] what?! this was unexpected. shutting down the system :: {open('flag.txt').read()} 👽")
            exit()
```

Esta vez la funcionalidad de este código es la siguiente:

- Por cada uno de los usuarios que están almacenados, guarda el nombre de usuario en la variable `db_user` (en este caso *HTBUser132*), y en `v` el usuario en MD5 (*`md5(b'HTBUser132').hexdigest()`*) y su contraseña (*`secure123!`*).
- Posteriormente compara el contenido de `v` (que contiene el usuario en MD5 y su contraseña), con los valores de *usr_hash* (que contiene el usuario en MD5) y *pwd* (que contiene la contraseña del usuario). En caso de que sean iguales entrará dentro del siguiente `if`.
- Por último, compara si el contenido de la variable `usr` (que contiene el usuario que nosotros proporcionamos) con la variable `db_user` (que contiene el nombre de usuario almacenado, en este caso *HTBUser132*). Si ambos valores coinciden, nos muestra el mensaje de bienvenida.
- En el caso en el que no sean iguales, nos muestra el contenido de la *flag* (`flag.txt`).

De todo esto podemos concluir que, si el nombre de usuario convertido a _hash MD5_ coincide con el de otro usuario (aunque tengan nombres diferentes), se cumplirá la condición del `elif`, permitiéndonos acceder y visualizar la _flag_.

### **Colisión de Hashes MD5**

Para lograr esto, necesitamos encontrar dos cadenas diferentes que, al ser convertidas a _hash MD5_, generen el mismo resultado. Hoy en día es fácil generar dos cadenas de este tipo, sin embargo, podemos obtenerlas del [siguiente artículo](https://www.johndcook.com/blog/2024/03/20/md5-hash-collision/), en el que nos muestra dos cadenas que [*Marc Stevens* publicó en su *Twitter*](https://x.com/realhashbreaker/status/1770161965006008570).

```bash
TEXTCOLLBYfGiJUETHQ4hAcKSMd5zYpgqf1YRDhkmxHkhPWptrkoyz28wnI9V0aHeAuaKnak
TEXTCOLLBYfGiJUETHQ4hEcKSMd5zYpgqf1YRDhkmxHkhPWptrkoyz28wnI9V0aHeAuaKnak
```

La diferencia entre ambas cadenas radica en un único byte, donde la posición 21, la letra _A_ es reemplazada por una _E_. Sin embargo, al convertirlas en *MD5* da el mismo resultado.

```bash
echo -n "TEXTCOLLBYfGiJUETHQ4hAcKSMd5zYpgqf1YRDhkmxHkhPWptrkoyz28wnI9V0aHeAuaKnak" | md5sum
echo -n "TEXTCOLLBYfGiJUETHQ4hEcKSMd5zYpgqf1YRDhkmxHkhPWptrkoyz28wnI9V0aHeAuaKnak" | md5sum
```

![md5 hash collision.png](/assets/img/md5-hash-collision.png)

Podemos intentar registrar dos usuarios con estos nombres y la misma contraseña, para que a la hora de registrarse, lograremos esa colisión de hashes, y el script nos muestre la flag.

```json
{"option":"register"}
{"username":"TEXTCOLLBYfGiJUETHQ4hAcKSMd5zYpgqf1YRDhkmxHkhPWptrkoyz28wnI9V0aHeAuaKnak","password":"password123"}

{"option":"register"}
{"username":"TEXTCOLLBYfGiJUETHQ4hEcKSMd5zYpgqf1YRDhkmxHkhPWptrkoyz28wnI9V0aHeAuaKnak","password":"password123"}

{"option":"login"}
{"username":"TEXTCOLLBYfGiJUETHQ4hEcKSMd5zYpgqf1YRDhkmxHkhPWptrkoyz28wnI9V0aHeAuaKnak","password":"password123"}
```

De esta manera al iniciar sesión con cualquiera de los usuarios mencionados anteriormente, lograremos colisionar los hashes de ambos usuarios permitiéndonos ver la *flag*.

![md5 hash collision login flag.png](/assets/img/md5-hash-collision-login-flag.png)

## **Enlaces de Referencia**

A continuación, se muestran algunos de los enlaces que he usado como recurso para entender el funcionamiento de las colisiones de hashes en MD5:

- [Alphascii Clashing - HackTheBox Challenge](https://app.hackthebox.com/challenges/alphascii%2520clashing)
- [Alphascii Clashing - Crypto Challenge \| HackTheBox](https://www.youtube.com/watch?v=Z1sCfCKZ_C0&ab_channel=S4viSinFiltro)
- [Colisión (hash) - Wikipedia](https://es.wikipedia.org/wiki/Colisi%C3%B3n_(hash))
- [MD5 Collision Attack — SEED Security Labs](https://ms-geeky.medium.com/md5-collision-attack-seed-security-labs-26c0014a4062)

## **Despedida**

En resumen, aunque MD5 fue durante mucho tiempo un pilar en la seguridad digital, las colisiones demostradas y explotables hoy en día lo convierten en una opción obsoleta para aplicaciones criptográficas serias.

Gracias por leer, ¡nos vemos en el próximo post!
