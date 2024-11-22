---
title: Active Directory - Abusando del Permiso AllowedToDelegate
description: Abusando del permiso AllowedToDelegateTo para poder impersonar al usuario Administrator mediante un ticket ST.
date: 2024-11-06
categories: [active-directory, allowed-to-delegate]
tags: [allowed-to-delegate-to, impersonar, gmsa, tgs, cuenta-de-servicio, ccache, getst, impacket, gmsadumper, pywerview, spn, hash-nt, service-ticket, st, krb5ccname, wmiexec, bloodhound]
img_path: /assets/img/Abusing-AllowedToDelegate-Rights.png
image: /assets/img/Abusing-AllowedToDelegate-Rights.png
---

## **Introducción**

El atributo **AllowedToDelegate** determina qué servicios o cuentas pueden delegar sus credenciales a otros servicios en nombre de un usuario o grupo. Si un atacante consigue acceso a una cuenta con permisos de delegación, puede utilizar esos derechos para **impersonar a otros usuarios y acceder a sus recursos**, incluso si esas cuentas tienen privilegios más altos.

### **Gráfico del escenario**

El siguiente, es un gráfico representativo de lo que acabamos de explicar.

![Abusing AllowedToDelegate Rights](/assets/img/Abusing-AllowedToDelegate-Rights.png)

### **Contexto**

Nos encontramos con el usuario *Ted.Graves* con el cual hemos obtenido sus credenciales. Este usuario pertenece al grupo *ITSupport* desde donde tiene permiso de *ReadGMSAPassword* sobre la cuenta de servicio **svc_int**. Los **Group Managed Service Accounts** o *GMSA* son un tipo de objeto especial de **Active Directory** en el que la contraseña de dicho objeto es gestionada automáticamente por los DC del dominio.

>**Sobre los Group Managed Service Accounts (GMSA)**<br>
>El uso normal de una *GMSA* es permitir que determinadas cuentas de equipo recuperen la contraseña de la *GMSA* y, a continuación ejecuten servicios locales como *GMSA*, sin embargo, un atacante con control sobre una cuenta autorizada puede abusar de este privilegio para hacerse pasar por la *GMSA*.

Una vez que obtengamos la contraseña de la *GMSA* de **svc_int**, vemos que tenemos el privilegio *AllowedToDelegate* sobre *dc.intelligence.htb*. Este privilegio nos otorga la capacidad de solicitar el *Service Ticket* (*TGS*) del usuario *Administrator*. Usando herramientas como `getST.py`, almacenaremos ese ticket de servicio en un archivo `.ccache`. Este archivo contiene las credenciales necesarias para poder autenticarnos en el DC (*dc.intelligence.htb*).

## Manos a la obra

Al lanzar **BloodHound**, nos encontramos con el siguiente caso potencial de escalada de privilegios.

![BloodHound máquina Intelligence](/assets/img/BloodHound-maquina-Intelligence.png)

Nosotros como *Ted.Graves*, tenemos el permiso para leer la contraseña de la cuenta de servicio **GMSA** asociada a **svc_int**. Para ello, usaremos la herramienta `gMSADumper.py` la cual, podremos descargar del [repositorio oficial de GitHub](https://github.com/micahvandeusen/gMSADumper). Con el parámetro `-u` le indicaremos nuestro usuario y con `-p` su contraseña. Por último, le indicaremos el dominio con el parámetro `-d`.

```bash
gMSADumper.py -u 'USUARIO' -p 'CONTRASEÑA' -d 'DOMINIO'
```

Una vez ejecutado, el resultado que obtendremos será el *hash NT* del usuario *svc_int*.

![gmsadumper con ted graves para svc_int](/assets/img/gmsadumper-con-ted-graves-para-svc_int.png)

Para poder usar la herramienta `getST` e impersonar al usuario **Administrator** utilizando el _hash NT_ obtenido, primero necesitamos identificar el **SPN** (_Service Principal Name_) asociado a la cuenta de servicio **svc_int** que tiene el privilegio de **AllowedToDelegate**.

Para obtener este *SPN*, utilizaremos la herramienta `pywerview`. Con el módulo `get-netcomputer` y el parámetro `--full-data`, podremos recuperar la información completa sobre los grupos. Una vez ejecutado el comando, deberemos revisar los resultados y enfocarnos en el valor `msds-allowedtodelegateto`, ya que ese valor contendrá el **SPN** necesario para realizar la **delegación de servicios**.

```bash
pywerview get-netcomputer -u "Ted.Graves" -p "Mr.Teddy" -t 10.10.10.248 --full-data
```

![spn de svc_int pywerview](/assets/img/spn-de-svc_int-pywerview.png)

Ahora sí, con `getST` de la suite de *impacket*, indicaremos:

- `-spn` ➜ El **SPN** que hemos obtenido.
- `-impersonate` ➜ El usuario que queremos **impersonar** (*Administrator*).
- `-hashes` ➜ El **hash NT** del usuario *svc_int*.

```bash
getST.py -spn "WWW/dc.intelligence.htb" -impersonate "Administrator" -hashes ":1d7a055a77db01cde7db3f4d006081fb" intelligence.htb/svc_int
```

![obteniendo ccache del administrador](/assets/img/obteniendo-ccache-del-administrador.png)

Si visualizamos el contenido de nuestro directorio, veremos el fichero `Administrator@WWW_dc.intelligence.htb@INTELLIGENCE.HTB.ccache` el cual, contiene el ticket de servicio del **administrador del dominio** con el cual, le podremos impersonar.

![ticket de servicio archivo ccache](/assets/img/ticket-de-servicio-archivo-ccache.png)

Renombraré el fichero como `Administrator.ccache` y, para poder usar este ticket, es necesario exportar la variable *KRB5CCNAME* con el nombre del archivo `ccache`. Esto paso es fundamental para podernos autenticar con `wmiexec`.

```bash
export KRB5CCNAME=Administrator.ccache
```

Por último con `wmiexec` conseguiremos autenticarnos como el usuario *Administrator*, indicándole con el parámetro `-k` que queremos usar un *Service Ticket* el cual, hemos configurado en la variable de entorno *KRB5CCNAME*.

```bash
wmiexec.py dc.intelligence.htb -k
```

![conectandonos con el usuario administrator por archivo ccache](/assets/img/conectandonos-con-el-usuario-administrator-por-archivo-ccache.png)

Con el usuario *Administrator* en nuestro poder, podemos realizar cualquier acción sobre el dominio completo.

## **Enlaces de referencia**

Los siguientes son recursos de utilidad que me ayudaron a realizar este artículo y, a aprender sobre ello.

- [Unconstrained Delegation Kerberos](https://deephacking.tech/unconstrained-delegation-kerberos/)
- [Máquina Intelligence - HackTheBox](https://app.hackthebox.com/machines/Intelligence)
- [Resolución de la máquina Intelligence - S4vitar](https://www.youtube.com/watch?v=LI8wnTUc5-I)
- [Buscador de máquinas de S4vitar por categorías](https://infosecmachines.io/)

## **Despedida**
Espero que te haya servido de ayuda este artículo. ¡Nos vemos en el próximo!
