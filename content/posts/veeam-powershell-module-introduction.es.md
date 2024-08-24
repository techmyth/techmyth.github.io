---
title: 'Veeam: Módulo de Powershell Introducción'
date: '2021-06-19T14:11:00-04:00'
author: 'Jonathan Colon Feliciano'
tags:
    - VEEAM
---

Hola a todos,

En esta ocasión estaré mostrando una introducción del módulo de Powershell de Veeam y cómo realizar la conexión inicial al servidor de backup. En la versión de 11 de `Veeam Backup & Replication` fue introducido un nuevo módulo de powershell llamado `Veeam.Backup.PowerShell`. Según el portal de documentación de Veeam existen dos formas para poder utilizar este módulo de Powershell:

1. Accediendo Powershell desde la consola de `Veeam Backup & Replication`.

![Text](/img/Powershell_Console_Veeam.webp#center)

2. Cargando el módulo desde una consola de Powershell.

```powershell
PS C:\Users\jocolon> Import-Module -Name Veeam.Backup.PowerShell

WARNING: The names of some imported commands from the module 'Veeam.Backup.PowerShell' include unapproved verbs that might make them less discoverable. To find the commands with unapproved verbs, run the `Import-Module` command again with the Verbose parameter. For a list of approved verbs, type `Get-Verb`.
PS C:\Users\jocolon> 
```

Es importante mencionar que este módulo de Powershell está disponible en el servidor de Backup de Veeam o en cualquier dispositivo donde la consola de manejo esté instalada. Referencia:

> The remote machine from which you run Veeam PowerShell commands must have the Veeam Backup & Replication Console installed. After you install the Veeam Backup & Replication Console, Veeam PowerShell module will be installed by default
>
> [Veeam PowerShell Reference](https://helpcenter.veeam.com/docs/backup/powershell/)

Para validar si el módulo fue cargado exitosamente pueden utilizar el comando ``Get-Module``. Como pueden ver en el ejemplo la versión 1.0 de ``Veeam.Backup.PowerShell`` está cargada en el sistema.

```powershell
PS C:\Users\jocolon> Get-Module -ListAvailable -Name Veeam.Backup.PowerShell


    Directory: C:\Program Files\Veeam\Backup and Replication\Console

ModuleType Version    Name                                ExportedCommands
---------- -------    ----                                ----------------
Manifest   1.0        Veeam.Backup.PowerShell             {Get-VBRComputerFileProxyServer, New-VBRSanI

PS C:\Users\jocolon>
```

Hablando un poco de historia la forma tradicional de cargar el módulo de Powershell en las versiones pre-v11 era utilizando el siguiente comando:

```powershell
 Add-PSSnapin -Name VeeamPSSnapIn #Veeam Powershell Pre11
```

Ahora en la versión 11 de Veeam VBR se utiliza el comando tradicional ``Import-Module`` como cualquier otro módulo de Powershell. En el caso de Veeam 11 se utiliza el comando haciendo referencia al módulo llamado ``Veeam.Backup.PowerShell``. Una vez es cargado el módulo podemos verificar que `cmdlets` están disponible utilizando el comando `Get-Command`.

```powershell
PS C:\Users\jocolon> Get-Command -Module Veeam.Backup.PowerShell

CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Alias           Add-VBRCloudTenantToTapeJob                        1.0        Veeam.Backup.PowerShell
Alias           Add-VBRHPSnapshot                                  1.0        Veeam.Backup.PowerShell
Alias           Add-VBRHPStorage                                   1.0        Veeam.Backup.PowerShell
Alias           Clone-VBRJob                                       1.0        Veeam.Backup.PowerShell
Alias           Get-VBRHPCluster                                   1.0        Veeam.Backup.PowerShell
Alias           Get-VBRHPSnapshot                                  1.0        Veeam.Backup.PowerShell
Alias           Get-VBRHPStorage                                   1.0        Veeam.Backup.PowerShell
Alias           Get-VBRHPVolume                                    1.0        Veeam.Backup.PowerShell
Alias           Remove-VBRHPSnapshot                               1.0        Veeam.Backup.PowerShell
Alias           Remove-VBRHPStorage                                1.0        Veeam.Backup.PowerShell
Alias           Set-VBRCloudTenantToTapeJob                        1.0        Veeam.Backup.PowerShell
Truncated.......
PS C:\Users\jocolon>
```

Ahora que tenemos todo listo podemos establecer la conexión inicial hacia nuestro servidor de Backup. Para lograr este proceso utilizaremos el comando ``Connect-VBRServer``. En este ejemplo se utiliza la opción para solicitar las credenciales, pero los módulos de Veeam aceptan los métodos tradicionales para proveer las credenciales en Powershell ([Referencias](https://duffney.io/addcredentialstopowershellfunctions/)).

```powershell
PS C:\Users\jocolon> Connect-VBRServer -Server veeam-vbr.pharmax.local -Credential (Get-Credential)

cmdlet Get-Credential at command pipeline position 1
Supply values for the following parameters:
User: pharmax\veeam_admin
Password for user pharmax\veeam_admin: ````
PS C:\Users\jocolon> 
```

En este ejemplo nos conectamos al servidor con el nombre de ``veeam-vbr.pharmax.local`` utilizando el nombre de usuario ``pharmax\veeam_admin``. Es importante mencionar que solo los usuarios con el rol de ``Veeam Backup Administrator`` pueden establecer conexión a través de powershell. Si tratamos con una cuenta que no tenga ese nivel de privilegio nos presentara el siguiente error ([Referencias](https://duffney.io/addcredentialstopowershellfunctions/)).:

```powershell
PS C:\Users\jocolon> Connect-VBRServer -Server veeam-vbr.pharmax.local -Credential (Get-Credential)

cmdlet Get-Credential at command pipeline position 1
Supply values for the following parameters:
User: pharmax\vrauser
Password for user pharmax\vrauser: ````
Connect-VBRServer : Only users with Veeam Backup Administrator role assigned can use Veeam Backup PowerShell Snap-in
At line:1 char:1
+ Connect-VBRServer -Server veeam-vbr.pharmax.local -Credential (Get-Cr ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : InvalidOperation: (:) [Connect-VBRServer], Exception
    + FullyQualifiedErrorId : AccessCheckerErrorId,Veeam.Backup.PowerShell.Cmdlets.ConnectVBRServer

PS C:\Users\jocolon> 
```

Podemos validar con el comando ``Get-VBRServerSession`` la secciones existentes. En el siguiente ejemplo se muestra la conexión al servidor ``veeam-vbr`` utilizando el usuario ``veeam_admin``.

```powershell
PS C:\Users\jocolon> Get-VBRServerSession                 


User        Server                  Port
----        ------                  ----
veeam_admin veeam-vbr.pharmax.local 9392


PS C:\Users\jocolon>
```

Una vez conectados al servidor de Veeam podemos ejecutar cualquier cmdlet para extraer la información requerida. Como ejemplo les mostraré las tareas de resguardo que se encuentran creadas en mi servidor de prueba de Veeam.

```powershell
PS C:\Users\jocolon> Get-VBRJob

WARNING: This cmdlet is no longer supported for computer backup jobs. Use "Get-VBRComputerBackupJob" instead.

Job Name                  Type            State      Last Result  Description
--------                  ----            -----      -----------  -----------
Server With Netapp LUN... Windows Agen... Stopped    None         Created by PHARMAX\administrator
HyperV-Backup-Job         Hyper-V Backup  Stopped    None         Created by PHARMAX\administrator 
WIN HyperV VM Backup      Hyper-V Backup  Stopped    Success      Created by PHARMAX\jocolon
PHARMAX-HQ-SVR            VMware Backup   Stopped    None         Created by PHARMAX\administrator
SOBR - TEST               VMware Backup   Stopped    None         Created by PHARMAX\administrator
COMP-CLUSTER-NFS          VMware Backup   Stopped    None         Created by PHARMAX\administrator 
HPE-StoreOnce-Copy-Job    VMware Backu... Stopped    Success      Created by PHARMAX\administrator
VM - Test - AWS           VMware Backup   Stopped    Failed       Created by PHARMAX\jocolon

PS C:\Users\jocolon> 
```

Espero esta información les sirva de ayuda. Si tienes dudas o alguna pregunta sobre este laboratorio, déjalo en los comentarios. Hasta Luego Amigos!

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/F1F8DEV80)

![Text](/img/backups-backups-everywhere.webp#center)
