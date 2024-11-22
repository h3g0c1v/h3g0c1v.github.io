---
title: Active Directory - ASREPRoast y Kerberoast Attacks
description: En este artículo se explicarán dos ataques muy comunes en entornos AD - ASRERoast Attack y Kerberoast Attack.
date: 2024-11-22
categories: [active-directory, asreroast, kerberoast]
tags: [casos-de-ataques-ad, impacket, getnpusers, getuserspn]
img_path: /assets/img/asreproast-y-kerberoast.png
image: /assets/img/asreproast-y-kerberoast.png
---

## **Introducción**
En entornos Active Directory, uno de los ataques más comunes que se intenta efectuar son el **ASRERoast y Kerberoast Attack**. Por esto, es por lo que es necesario tener los conocimientos necesarios para poder realizarlos.

En este artículo, aprenderás a como realizar estos dos ataques y lograrás entender un poco más en detalle el funcionamiento de **Kerberos** frente a ello. El funcionamiento de Kerberos puede ser un poco abrumador al principio, por lo que se intentará explicar de la manera más sencilla.

## **Funcionamiento general de Kerberos**
**Kerberos** es un protocolo de autenticación basado en *tickets*, en lugar de transmitir contraseñas de usuario a través de la red. Los Controladores de Dominio (DC) constan de un *Centro de Distribución de Claves de Kerberos (KDC)* que, básicamente, es el encargado de emitir los *tickets*. Cuando un usuario intenta iniciar sesión en un ordenador, el usuario que usa para autenticarse solicita un ticket al *KDC*, cifrando la solicitud con la contraseña del usuario. En caso de que el *KDC* pueda **descifrar la solicitud (AS-REQ)** utilizando la contraseña al usuario, se creará un *Ticket de Concesión de Tickets (TGT)* y lo enviará al usuario.

Posteriormente, el usuario presenta su *TGT* al *DC* para solicitar un ticket del *Servicio de Concesión de Tickets (TGS)*, cifrando el hash de la contraseña *NTLM* del servicio asociado. Finalmente, el cliente solicita acceso al servicio requerido presentando el *TGS* a la aplicación o servicio. Si todo el proceso se completa correctamente, se permitirá al usuario acceder al servicio o aplicación solicitada.

La autenticación *Kerberos* separa eficazmente las credenciales del usuario de sus solicitudes a los recursos consumibles, asegurando que la contraseña NO se transmita a través de la red. El *Centro de Distribución de Claves Kerberos* (*KDC*) no registra transacciones anteriores. En cambio, el ticket del *Servicio de Concesión de Tickets* (*TGS*) depende de un *Ticket de Concesión de Tickets* (*TGT*) válido. Se asume que si el usuario tiene un *TGT* válido, ha demostrado su identidad.

El siguiente diagrama ayudará a entender lo anteriormente explicado.

![Funcionamiento de Kerberos](/assets/img/funcionamiento-de-kerberos.png)

>1. El usuario inicia sesión y su contraseña se convierte en un hash *NTLM*, que se utiliza para cifrar el ticket *TGT*. Esto separa las credenciales del usuario de las solicitudes a los recursos.
>2. El servicio *KDC* en el *Controlador de Dominio* (*DC*) verifica la solicitud de servicio de autenticación (*AS-REQ*), confirma la información del usuario y crea un *Ticket de Concesión de Tickets* (*TGT*), que se entrega al usuario.
>3. El usuario presenta el *TGT* al *DC* para solicitar un ticket del *Servicio de Concesión de Tickets* (*TGS*) para un servicio específico. Esta solicitud se llama *TGS-REQ*. Si el *TGT* se valida correctamente, sus datos se copian para crear un ticket *TGS*.
>4. El *TGS* se cifra con el hash *NTLM* de la contraseña de la *cuenta del servicio o del equipo* bajo cuyo contexto se ejecuta la instancia del servicio, y se entrega al usuario en el *TGS_REP*.
>5. El usuario presenta el *TGS* al servicio y, si es válido, se permite al usuario conectarse al recurso (*AP_REQ*).

El protocolo *Kerberos* utiliza el puerto *88* (tanto *TCP* como *UDP*). Al enumerar un entorno de Directorio Activo, a menudo podemos localizar los *Controladores de Dominio* realizando escaneos de puertos para buscar el puerto 88 abierto, utilizando una herramienta como `nmap`.

## **ASRERoast Attack**
Empezaremos explicando el ataque **ASREPRoast**. Se le denomina *ASREPRoast Attack* debido a que, justamente de lo que se aprovecha es de, la solicitud *AS-REQ* que se realiza al *KDC* para pedir el ticket *TGT*. El ataque *ASREPRoast Attack* consiste en, enumerar los usuarios que tienen el atributo **UF_DONT_REQUIRE_PREAUTH**. Este atributo, lo que permite es que el usuario que lo contenga no necesite una autenticación previa de Kerberos. Los atacantes se pueden aprovechar de ello para que el propio DC le responda con el ticket *TGT* del usuario correspondiente. Este ticket *TGT* podemos intentarlo craquear con herramientas como `hashcat` o `john`.

### **Efectuando un ASREPRoast Attack**
Dejando de lado la teoría, vamos a ponernos la práctica que es con lo que realmente se aprende. Para efectuar el ataque, usaremos la *suite de impacket*, concretamente la herramienta *GetNPUsers*. En este ataque, **NO es necesario tener unas credenciales válidas**, simplemente, a partir de un fichero de usuarios que pueden ser válidos a nivel de dominio, comprobaremos quien tiene el atributo **UF_DONT_REQUIRE_PREAUTH** configurado y obtendremos su *TGT*.

```bash
GetNPUsers.py -no-pass -usersfile FICHERO-DE-USUARIOS DOMINIO/
```

![asreproast attack explication](/assets/img/asreproast-attack-explication.png)

Como todos los ataques hacia Kerberos, necesitaremos tener sincronizado nuestro reloj con el del DC, por lo contrario no funcionará. Para sincronizar el reloj con el *DC* podemos usar la herramienta `ntpdate` seguido del parámetro `-d` y la dirección *IP* del contralor del dominio.

```bash
ntpdate -d DIRECCION-IP-DEL-DC
```

## **Kerberoast Attack**
El *Kerberoast Attack* consiste en la obtención del ticket *TGS* de las cuentas de usuario que cuenten con un *Service Principal Name (SPN)*. Los **SPN** se usan para asociar un nombre de servicio con una cuenta de usuario.

### **Efectuando un Kerberoast Attack**
Para efectuar el ataque, usaremos de nuevo la *suite de impacket*, concretamente la herramienta *GetUserSPNs*. En este ataque **SI es necesario unas credenciales válidas**, aunque pueden ser de cualquier usuario del dominio, independientemente de los privilegios que tenga.

Antes de obtener el *TGS*, observaremos si hay algún usuario desde el cual lo podamos obtener.

```bash
impacket-GetUserSPNs DOMINIO/USUARIO:CONTRASEÑA -dc-ip IP-DC
```

![Comprobamos si hay algún usuario Kerberoasteable](/assets/img/kerberoasting-attack-ejemplo.png)

Una vez que vemos que hay un usuario que es *kerberoateable*, añadiremos el parámetro `-request` para obtener su ticket *TGS*.

```bash
impacket-GetUserSPNs DOMINIO/USUARIO:CONTRASEÑA -dc-ip IP-DC -request
```

![Obteniendo TGS Kerberoasting Attack Ejemplo](/assets/img/obteniendo-tgs-kerberoasting-attack-ejemplo.png)

Al igual que comentamos en el ataque anterior, necesitaremos tener sincronizado nuestro reloj con el del *DC*.

## Enlaces de Referencia
Los siguientes son recursos de utilidad que me ayudaron a realizar este artículo y, a aprender sobre ello.

- [Resolución de la Máquina Forest - S4vitar](https://www.youtube.com/watch?v=7G5wkoBpFWU&t=3984s)
- [Explicación en Detalle del Funcionamiento de Kerberos - Tarlogic](https://www.tarlogic.com/es/blog/como-funciona-kerberos/)
- [Humilde Intento de Explicar Kerberos - DeepHacking](https://deephacking.tech/humilde-intento-de-explicar-kerberos/)
- [Curso de Active Directory - HackTheBox](https://academy.hackthebox.com/module/details/74)

## Despedida
Espero que este artículo te haya sido de utilidad para entender los conceptos de *Kerberos*, *ASREPRoast Attack* y *Kerberoast Attack*. ¡Nos vemos en el próximo!
