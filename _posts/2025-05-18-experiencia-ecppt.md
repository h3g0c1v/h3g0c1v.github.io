---
title: Mi Experiencia con la certificación eCPPTv3
description:  El objetivo de este blog es brindar apoyo a las personas que estén en progreso de obtener la certificación eCPPTv3.
date: 2025-05-18
categories: [Certifications, eCPPTv3]
tags: [eCPPTv3, INE Security, Certified Professional Penetration Tester]
img_path: https://api.accredible.com/v1/frontend/credential_website_embed_image/certificate/143580237
image: https://api.accredible.com/v1/frontend/credential_website_embed_image/certificate/143580237
---

## **Introducción**
¡Bievenidos a todos! En este artículo vamos a cubrir algunos conceptos claves que te serán de ayuda frente a la certificación ***eCPPTv3* (*Certified Professional Penetration Tester*) de *INE Security***. Hace un tiempo publiqué un artículo en el que contaba [Mi Experiencia con la certificación eJPTv2](https://h3g0c1v.github.io/posts/experiencia-ejpt/), y al igual que hice con ella, mi objetivo principal es contarte todo lo que necesitas saber antes de enfrentarte al examen del *eCPPTv3*.

Recientemente esta certificación sufrió una actualización, introduciendo conceptos de *Active Directory* los cuales anteriormente no estaban presentes, y reduciendo drásticamente el porcentaje de evaluación en temas como el Buffer Overflow, e incluso eliminando el pivoting y post reporting. Es por ello por lo que vengo a aclarar todos los cambios que han habido.

## **Qué es el eCPPT**
El **eCPPT** es una de las certificaciones con las que cuenta **INE Security**, y está enfocada a personas con un nivel **básico-intermedio** en el mundo de la **ciberseguridad y el pentesting**. Tal y como comenta [INE en su página oficial](https://security.ine.com/certifications/ecppt-certification/), esta certificación valida los conocimientos necesarios *para cumplir con el rol de un Penetration Tester Moderno*:

- **Information Gathering & Reconnaissance** (10%)
- **Initial Access** (15%)
- **Web Application Penetration Testing** (15%)
- **Exploitation & Post-Exploitation** (25%)
- **Exploit Development** (5%)
- **Active Directory Penetration Testing** (30%)

En mi caso, el coste de la certificación fue considerablemente más bajo que el habitual, ya que pude aprovechar una oferta del 50%, no obstante, el precio de la certificación es de **$600**. El voucher incluye el plan "*Premium*", que proporciona acceso al curso de [*Penetration Testing Professional (PTP)*](https://my.ine.com/CyberSecurity/learning-paths/5e26d0ba-d258-49e0-a421-56cc06626f46/penetration-testing-professional-new-2024) durante **tres meses**, sin embargo, tendrás **seis meses para presentarte al examen**.

Al igual que comenté en mi experiencia con el *eJPT*, a diferencia del formato del curso, soy una persona que necesita ponerse manos a la obra para aprender, por lo que decidí practicar en plataformas como [Hack The Box](https://app.hackthebox.com/) o [PortSwigger](https://portswigger.net/).

Previamente a la obtención del voucher que me permitiría poder presentarme al examen, me estuve informando acerca del curso y las opiniones eran bastante contundentes, "*el contenido no es suficiente para preparar adecuadamente al estudiante para el examen*", y es una de las razones por las cuales no realice el curso, pero te invito a que lo pruebes y decidas por ti mismo.

## **Formato del examen**
El examen se realiza en un **laboratorio dentro de un navegador** con un *Kali Linux* ya configurado con las herramientas y diccionarios necesarios para la correcta resolución del examen. La plataforma que se utiliza es *Guacamole*, y su ambiente es **pésimo**, así que te recomiendo que tengas paciencia porque te vas a encontrar con múltiples inconvenientes a causa de dicho entorno. Este entorno **no tiene acceso a Internet**, aunque podrás utilizar todas las aplicaciones que necesites fuera del laboratorio.

Te encontrarás con máquinas **Windows** (la mayoría) y **Linux**, las cuales estarán unidas a un dominio. La realidad es que el groso del examen es ***AD***, la **enumeración y fuerza bruta**, así que trata de conseguir algún usuario que te pueda servir como punto de apoyo para lograr comprometer las máquinas. En lo personal me encontré con un dominio y un subdominio, donde tuve que encontrar primero cuales eran los *DCs* para poder realizar la mayoría de los ataques.

Además de las máquinas tendrás que responder **45 preguntas**, algunas tipo test y otras no, en un plazo de **24 horas** con las que se te evaluará y dependerá tu nota del examen. Para poder superarlo es necesario obtener, al menos, un 70% de respuestas correctas y, cabe destacar, que cuentas con **dos oportunidades para aprobar**, por lo que si no lo consigues la priemra vez, no te preocupes, vas a tener una segunda oportunidad.

## **Experiencia y preparación**
### Experiencia previa
Con respecto a mi experiencia previa para el examen, contaba con una base y conocimientos básicos del pentesting en *Directorio Activo* y *Pentesting Web*, así que me tuve que poner manos a la obra con ambos temas. En un principio pensaba que conceptos como el *pivoting* y *Buffer Overflow* aún seguían presentes en el examen, y les dedique el tiempo suficiente, sin embargo cuando llegué al examen la situación era totalmente diferente, nada de *pivoting* ni de *BoF*, por lo que no te hará falta estudiarlos porque no tienen cabida en el examen.

La gran mayoría de la práctica la dedique en la [Academia de Hack The Box](https://academy.hackthebox.com/), realizando algunos módulos que corresponden al **path del [*CPTS*](https://academy.hackthebox.com/exams/3/)**, ya que esta certificación toca algunos conceptos que están presentes en el *eCPPTv3*:

- **[Active Directory Enumeration & Attacks](https://academy.hackthebox.com/module/143/)**.
- **[SQL Injection Fundamentals](https://academy.hackthebox.com/module/33/)**.
- **[Cross-Site Scripting (XSS)](https://academy.hackthebox.com/module/103/)**.
- **[Windows Privilege Escalation](https://academy.hackthebox.com/module/67/)**.
- **[Attacking Common Services](https://academy.hackthebox.com/module/116/)**.

Aprovechando el recurso que realice con el *eJPT*, he agregado un apartado para esta certificación con algunas de las máquinas y/o laboratorios que recomiendo realizar en la preparación del examen.

> [Máquinas para Prepararte el eCPPTv3](https://docs.google.com/spreadsheets/d/1W8MXfDbTfQHFZ5XRMAKMysSBwvOpXuUS/edit?gid=1995623675#gid=1995623675)

Quiero destacar que este listado **no es un camino a seguir que te prepara completamente para la certificación**, y te recomiendo que practiques mucho más por tu cuenta, realizando diferentes máquinas que se adapten a los criterios de este examen

### Preparación
Mi preparación se basó en realizar las máquinas CTF del listado, algunas máquinas más que no he considerado recomendar, algunos laboratorios de *PortSwigger*, y los módulos de HTB mencionados con anterioridad. En este caso no me hizo falta montar ningún laboratorio, ya que con los conocimientos adquiridos de las máquinas eran más que suficientes.

Especialmente puedo recomendar [este video de Savitar](https://www.youtube.com/watch?v=osmFGqnFe8c&t=4743s), ya que explica con especial detalle cual es el funcionamiento básico de la **autenticación en Kerberos**, lo cual es crucial para entender el funcionamiento de un entorno ***AD***.

## **Consejos**
1. **Se organizado**: Créate una carpeta por cada host que encuentres y almacena todo el contenido de forma estructurada.
2. **Toma apuntes**: Utiliza herramientas como *Obsidian* para tomar apuntes, ya que te ayudará en el futuro cuando necesites recuperar cierta información.
3. **Lee las preguntas antes de empezar el laboratorio**: Leer las preguntas antes de comenzar el laboratorio te ayudará a acordarte de ellas en función de lo que vayas encontrando y te permitirá responderlas con facilidad.
4. **Responde por cada Host**: Te recomiendo que intentes responder todas las preguntas asociadas al host que acabas de vulnerar antes de pasar al siguiente.
5. **Aprovecha las respuestas**: Presta mucha atención a las respuestas, ya que muchas de ellas te preguntan algo como: "*¿Que usuario de los siguientes es vulnerable a Password Spraying?*". Create un diccionario con los usuarios que se te muestran en la respuesta y prueba a hacerle un ataque de fuerza bruta, seguro que encuentras la solución.
6. **Relájate y tomate tu tiempo**: Aunque tengas 24 horas para superar el examen, cuentas con tiempo de sobra. Haz los descansos que necesites, levántate de la silla, hidrátate, lee un libro, date un paseo. Cuando estás atascado, la mejor opción es despejar la mente y luego volver, lo verás todo mucho más claro.
7. **Última opción `rockyou.txt`**: En el examen cuentas con varios diccionarios. Te recomiendo que te ciñas a `seasons.txt`, `months.txt`, `xato-1000`, `common-corporate-passwords`, y deja como última opción `rockyou.txt`.

## **Conclusión**
Si estás realizando esta certificación como introducción al mundo de la ciberseguridad, o simplemente quieres una certificación que sea un poco más avanzada **no te la recomiendo**. Sinceramente creo que es mejor elegir primero el *eJPT* y después dar un salto al *CPTS*, *OSCP* o alguna por el estilo. Ahora bien, si estás buscando una certificación que te sirva como introducción al mundo de **AD**, es una muy buena opción.

Según INE, estas son las herramientas que se recomiendan utilizar durante el examen:

- *`CrackMapExec`*.
- *`Evil-WinRM`*.
- *`PowersShell-Empire`*.
- *`Metasploit Framework`*.
- *`PowerSploit`*.
- *`Kerbrute`*.

En lo personal, y al menos en lo que he estado leyendo frente a la anterior versión, creo que la certificación *eCPPTv3* **ha dado un salto hacia atrás en cuanto a dificultad**, y una de las partes del examen que más importante consideraba, el post reporting, la han eliminado. Por supuesto, no digo que no sea una certificación con la que no vayas a aprender, si no que conceptos como *AD* se podrían haber integrado perfectamente con la versión antigua de esta certificación, haciendo que realmente mereciera la pena.

## **Algunos recursos que recomiendo**
- [Path del HTB CPTS](https://academy.hackthebox.com/exams/3/) ➜ Puedes aprovechar algunos de los módulos que se ofrecen en la academia de HTB, para estudiar y practicar conceptos que posteriormente tocarás en el examen.
- [PortSwigger All Labs](https://portswigger.net/web-security/all-labs) ➜ Realiza los laboratorios de [SQL Injection](https://portswigger.net/web-security/all-labs#sql-injection), [Cross-Site Scripting \(XSS\)](https://portswigger.net/web-security/all-labs#cross-site-scripting) y/o [Command Injection](https://portswigger.net/web-security/all-labs#os-command-injection) en sus niveles de *Apprentice* y/o *Practitioner*, ya que son algunas de las vulnerabilidades web que pueden aparecer.
- [Máquinas con temática de AD - Hack The Box](https://app.hackthebox.com/machines/list/retired?sort_type=desc&difficulty=easy&difficulty=medium&tag=1058) ➜ Realiza máquinas que toquen conceptos de *Active Directory*.

## **Despedida**
Espero que con este artíclo tengas más claro el enfoque del *eCPPTv3*. ¡Mucha suerte a todos!

![Mi Certificación eCPPT](https://api.accredible.com/v1/frontend/credential_website_embed_image/certificate/143580237)
