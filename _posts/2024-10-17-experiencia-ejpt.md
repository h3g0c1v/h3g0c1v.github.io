---
title: Mi Experiencia con la certificación eJPTv2
description:  El objetivo de este blog es brindar apoyo a las personas que se estén adentrando al mundo de la ciberseguridad.
date: 2024-10-17
categories: [Certifications, eJPTv2]
tags: [eJPTv2, INE Security, Junior Penetration Tester]
img_path: https://api.accredible.com/v1/frontend/credential_website_embed_image/certificate/118676081
image: https://api.accredible.com/v1/frontend/credential_website_embed_image/certificate/118676081
---

## **Introducción**
¡Muy buenas a todos! Soy Héctor Civantos, *aka Hegociv* y hace poco me enfrenté al examen del *eJPTv2* (**eLearnSecurity Junior Penetration Tester**) de *INE Security*. El objetivo de este blog es brindar apoyo a las personas que se estén adentrando al mundo de la ciberseguridad y quieran subir un escalón más en el intrepidante universo de las certificaciones.

El **eJPT** es conocido como el **primer paso** que debes dar para adentrarte al mundo del pentesting y la ciberseguridad ofensiva, y es por ello que es esencial tener los conocimientos necesarios para poder **enfrentarte al examen**. Esta certificación tiene un **nivel básico** y valida los conocimientos en pruebas de penetración (*pentesting*).

## **Qué es el eJPT**
El **eJPT** es la primera certificación que debes de obtener si quieres trabajar en un equipo *Red Team* en el departamento de ciberseguridad. Según *INE Security*, el examen evalúa los siguientes conocimientos:

- **Auditoría de Host y Red**.
- **Metodologías de Evaluación**.
- **Pentesting de Host y Red**.
- **Pentesting en Aplicaciones Web**.

Personalmente, el costo de la certificación fue de *$150*, ya que tuve la suerte de aprovechar una oferta de casi el 50%. El precio normal **ronda los $250** aunque suelen haber muchas ofertas a lo largo del año. El voucher incluye el plan "*Fundamentals*", que proporciona acceso al curso de [*Penetration Testing Student (PTS)*](https://my.ine.com/CyberSecurity/learning-paths/61f88d91-79ff-4d8f-af68-873883dbbd8c/penetration-testing-student) durante **tres meses**, sin embargo, tendrás **seis meses para presentarte al examen**.

En cuanto al curso, sinceramente, soy una persona que necesita ponerse manos a la obra para aprender. No me adapto bien a las clases en las que se presenta información de forma teórica, ya que prefiero la práctica y la interacción directa con los contenidos. Aún así, **el curso es muy completo** y tiene vídeos y laboratorios prácticos que te preparan de sobra para aprobar el examen.

## **Formato del examen**
El examen se realiza en un **laboratorio dentro de un navegador** con un *Kali Linux* ya configurado con las herramientas y diccionarios necesarias para la correcta resolución del examen. Este entorno **no tiene aceso a Internet**, aunque podrás utilizar todas las aplicaciones que necesites fuera del laboratorio.

En primera instancia, tendrás acceso y perteneceras a una **red *DMZ***, en la que se encuentran diferentes máquinas tanto **Windows** como **Linux** las cuales tendrás que explotar. En una de las máquinas te encontrarás con **otra red** aislada desde la cual no tendrás acceso directamente desde tu Kali Linux. Aquí entran tus conocimientos de *pivoting*, ya que deberás de pivotar con *Metasploit* para poder obtener acceso a la siguiente red y vulnerar las máquinas que encuentres a continuación.

Además, de las máquinas, tendrás que responder a **35 preguntas**, algunas tipo test y otras no, en un plazo de 48 horas con las que se te evaluará y dependerá tu nota del examen. Es necesario, obtener al menos un 70% para poder aprobar el examen y cabe destacar que tienes dos oportunidades para aprobar, por lo que si suspendes la primera vez no te preocupes, ya que contarás con otra oportunidad.

## **Experiencia y preparación**
### Experiencia previa
En cuando a mi experiencia previa para el examen, **ya contaba con mucha más práctica** resolviendo múltiples máquinas en diferentes plataformas como *[Hack The Box](https://www.hackthebox.com/)*, *[Try Hack Me](https://tryhackme.com/)*, *[The Hacker Labs](https://thehackerslabs.com/)* o *[Vulnhub](https://www.vulnhub.com/)* entre otras, además de haber realizado **varios curso centrados en la seguridad ofensiva**. Es debido a esto, que no realicé el curso completo del *PTS*, ya que la mayoría de los temas que se tocan los tenía claros por mi experiencia previa.

En cuyo caso, decidí seguir practicando con diferentes máquinas que tenían un mayor enfoque en esta certificación y es por ello que dejo un recurso que he creado con todas mis máquinas realizadas en la preparación del examen.

> [Máquinas para Prepararte el eJPT](https://docs.google.com/spreadsheets/d/1W8MXfDbTfQHFZ5XRMAKMysSBwvOpXuUS/edit?gid=1173184797#gid=1173184797)

Si eres principiante y no tienes experiencia, te recomiento **aprovechar todo el contenido y manterial** que te proporciona el curso *PTS* de *INE Security*. Con el contenido de este curso, podrás ir totalmente seguro al examen.

### Preparación
Mi preparación se basó en realizar las máquinas CTF del listado que he compartido anteriormente. Puedo recomendar realizar el curso de *[Introducción al Hacking](https://hack4u.io/cursos/introduccion-al-hacking/)* de la academia de *[Hack4u](https://hack4u.io/)* ya que, aunque toca temas más avanzados que los que te encontrarás en la propia certificación, te da un enfoque de explotación más manual y práctico que el curso del *PTS*.

Por supuesto, monté varios laboratorios utilizando algunas de las máquinas del listado para practicar **pivoting con Metasploit**, tanto en **Windows** como en **Linux**. De esta manera, pude realizar **simulacros de examen** y estar más preparado para el examen real.

Con toda esta preparación, más la previa experiencia que contaba, estaba más que preparado y con mucha confianza para afrontar el examen.

## **Consejos**
1. **Se organizado**: Create una carpeta por cada host que encuentres y almacena el contenido de forma estructurada. Por ejemplo, guarda los escaneos en la carpeta `nmap`; los recursos que encuentres en la carpeta `content`; los exploits que uses en la carpeta `exploits`.
2. **Toma apuntes**: Utiliza herramientas como *Obsidian*, *Excalidraw*, *Draw.io* y/o *App Diagrams* para tomar apuntes y hacerte tu mapa visual de la red. Esto te ayudará muchisimo a organizarte.
3. **Lee las preguntas antes de empezar el laboratorio**: Leer las preguntas antes de comenzar el laboratorio te ayudará a acordarte de ellas en función de lo que vayas encontrando y te permitirá responderlas con facilidad.
4. **Responde por Host**: Te recomiendo que intentes responder todas las preguntas asociadas al host que acabas de vulnerar antes de pasar al siguiente.
5. **Descansa y relajate**: Haz descansos cada cierto tiempo, tienes tiempo de sobra. Levantate de la silla, bebe agua, tomate algo de comer, sal a la calle o lee un libro. Cuando estás atascado, la mejor opción es despejar la mente y luego volver, lo verás todo mucho más claro.
6. **Aprovecha las respuestas**: Muchas de las preguntas y respuestas del examen te dan una pista por donde pueden ir los tiros. Prestar atención a estos detalles puede ayudarte a identificar vulnerabilidades y a enfocar mejor el examen.
7. **Todo lo que necesitas está ahí**: No hace falta que te compliques con scripts adicionales. El entorno que se te ha proporcionado contiene todas las respuestas que necesitas.

## **Conclusión**
Personalmente, considero que el examen es **relativamente fácil**, habiendolo logrado concluir en 9 horas yendo despacio y apuntando todo con calma. Recomiendo encarecidamente esta certificación ya que te enfrenta a un **entorno real** y no a los típicos *CTFs* (aunque si que tendrás que introducir algunas flags).

Tanto si tienes conocimientos más avanzados como si eres principiante es bastante acertado **obtener el eJPT** y te ayuda a familiarizarte con las diferentes fases de una auditoría real y a dominar herramientas esenciales como:

- *`Nmap`*
- *`Dirb`*
- *`Nikto`*
- *`WPScan`*
- *`CrackMapExec`*
- *`The Metasploit Framework`*
- *`Searchsploit`*
- *`Hydra`*

Aún así, si tienes más experiencia y no quieres enfrentarte a una certificación fácil como la del **eJPT**, podrías considerar **certificaciones más avanzadas** como *eCPPT*, *eWPT*, *eWPTX*, *OSEP*, *OSCP*, y muchas otras más. Sin embargo, el *eJPT* te puede servir como primer examen para **perder el "miedo"** a este tipo de certificaciones, y es justamente de lo que **me ha servido esta certificación**.

## **Algunos recursos que recomiendo**

- [Curso de Introducción al Hacking en Hack4u (S4vitar)](https://hack4u.io/cursos/introduccion-al-hacking/) ➜ Curso avanzado que cubre más alla de los conocimientos del eJPT.
- [Curso de Preparación para la Certificación del eJPTv2 (El Pinguino de Mario)](https://elrincondelhacker.es/courses/preparacion-certificacion-ejptv2/) ➜ Curso especializado en prepararse para el eJPT.
- [Laboratorio de preparación eJPTv2 \| Simulación de examen (Xerosec)](https://www.youtube.com/watch?v=v20IsEd5nUU) ➜ Laboratorio para realizar un simulacro del examen del eJPT.
- [Guía de Preparación y Máquinas para el eJPTv2](https://r1vs3c.github.io/posts/review-ejpt/) ➜ Guía detallada con más recursos para la preparación del eJPT.
- [Preparación para el eJPTv2](https://rinku.tech/experiencia-ejptv2/) ➜ Listado de máquinas específicas para el eJPT.
- [Pivoting en Metasploit para entornos Windows](https://www.youtube.com/watch?v=WM8lHCHblDU) ➜ Tutorial para hacer pivoting con Metasploit en entornos Windows.
- [Pivoting en Metasploit para entornos Linux](https://www.youtube.com/watch?v=RotyKByc8Jc&t=708s) ➜ Tutorial para hacer pivoting con Metasploit en entornos Linux.
- [eJPT RoadMap](https://github.com/nyxragon/ejpt-roadmap) ➜ Una guía de ruta para seguir semana a semana con el fín de estar preparado para el examen del eJPT.

## **Despedida**
Espero que este artículo les haya servido de ayuda para el eJPTv2. De todo corazón, ¡mucha suerte a todos!.

![Mi certificación eJPT](https://api.accredible.com/v1/frontend/credential_website_embed_image/certificate/118676081)
