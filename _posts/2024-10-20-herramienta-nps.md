---
title: NPS - La herramienta perfecta para encontrar scripts de Nmap fácilmente
description: NPS es una herramienta que nos permite buscar por los scripts que contiene nmap para el protocolo especificado.
date: 2024-10-20
categories: [herramientas, nmap]
tags: [nps, bash, reconocimiento, protocolos]
img_path: /assets/img/nps.png
image: /assets/img/nps.png
---

## **Introducción**

En este artículo, vamos a explicar el funcionamiento de la herramienta [nps](https://github.com/h3g0c1v/nps) o también llamada *Nmap Scripts for a Protocol*. Esta herramienta permite **buscar todos los scripts** disponibles en *Nmap* **para los protocolos que le especifiquemos**, facilitando así el proceso de enumeración.

## **Funcionamiento de NPS**

*NPS* está completamente escrita en *Bash* y resulta **muy útil durante la fase de reconocimiento** en una auditoría, ya que automatiza la búsqueda de scripts relevantes, lo que ahorra tiempo y mejora la eficiencia a la hora de identificar posibles vulnerabilidades en servicios y protocolos específicos.

El funcionamiento principal de `nps` consiste en filtrar el contenido del directorio `/usr/share/nmap/scripts/`, donde se encuentran los scripts de *Nmap*. Utilizando una expresión regular, la herramienta **busca y filtra** los scripts relacionados con el/los protocolo/s que se le hayan especificado, y finalmente muestra una **lista de los scripts que Nmap proporciona para dichos protocolos**.

## **Clonando la herramienta**
Empezaremos **clonándonos la herramienta**, por lo que nos dirigiremos al directorio que deseemos y ejecutaremos los siguientes comandos.

```bash
git clone https://github.com/h3g0c1v/nps.git
cd nps
```

Para que el script funcione, tendremos que darle **permisos de ejecución**.

```bash
chmod u+x nps.sh
```

Algo que cómodo que podemos hacer, es moverlo a alguna ruta del *$PATH* para ejecutarlo de una manera más rápida.

```bash
cp nps.sh /usr/bin/nps
```

De esta manera, podemos ejecutarlo en cualquier momento.

```bash
nps
```

## **Poniéndolo en práctica**
Bien, hasta ahora solo hemos hablado del funcionamiento de nps y de cómo clonarla, pero aún no la hemos probado. Para ver el **panel de ayuda**, simplemente necesitamos ejecutar la herramienta. Esto nos proporcionará información sobre los **parámetros** que podemos utilizar.

Cabe recalcar, que la herramienta **deberá de ejecutarse como root**, en caso contrario nos saldrá el siguiente mensaje de error.

![Ejecución de NPS sin root](/assets/img/ejecucion-sin-root-nps.png)

Supongamos que nos encontramos en la etapa de reconocimiento y tenemos un host en el que el servicio *RPC* está activo. Si necesitamos ejecutar un script de *Nmap* pero **no recordamos su nombre**, esta es una de las **funcionalidades clave** de *NPS*.

```bash
❯ nps rpc

[+] RPC
	bitcoinrpc-info.nse
	deluge-rpc-brute.nse
	metasploit-msgrpc-brute.nse
	metasploit-xmlrpc-brute.nse
	msrpc-enum.nse
	nessus-xmlrpc-brute.nse
	rpc-grind.nse
	xmlrpc-methods.nse
```

Pongámonos en situación que queremos ejecutar **varios scripts de diferentes protocolos**. Con *NPS*, podemos buscarlos todos a la vez. Para ello, deberemos pasarle como argumentos tantos protocolos como deseemos.

```bash
❯ nps ftp ssh rdp

[+] FTP
	ftp-anon.nse
	ftp-bounce.nse
	ftp-brute.nse
	ftp-libopie.nse
	ftp-proftpd-backdoor.nse
	ftp-syst.nse
	ftp-vsftpd-backdoor.nse
	ftp-vuln-cve2010-4221.nse
	tftp-enum.nse
	tftp-version.nse

[+] SSH
	ssh-auth-methods.nse
	ssh-brute.nse
	ssh-hostkey.nse
	ssh-publickey-acceptance.nse
	ssh-run.nse

[+] RDP
	rdp-enum-encryption.nse
	rdp-ntlm-info.nse
	rdp-vuln-ms12-020.nse
```

## **Despedida**
Muchas gracias por leer este blog y espero que la herramienta te sea de utilidad. Si tienes alguna aportación, no dudes en escribirme por [LinkedIn](https://www.linkedin.com/in/h%C3%A9ctor-civantos-cid-aka-hegociv-5ab997212). !Un abrazo!
