---
title: 'Veeam Powershell Module Introduction'
date: '2021-06-19T14:11:00-04:00'
author: 'Jonathan Colon Feliciano'
tags:
    - VEEAM
---

Hello to everyone,

This time I will be showing an introduction of the Veeam Powershell module and how to make the initial connection to the backup server. In Veeam Backup & Replication version 11 a new powershell module called `Veeam.Backup.PowerShell` was introduced. According to the Veeam documentation portal there are two ways to use this powershell module:

1. Accessing Powershell from the Veeam Backup &amp; Replication console.

![Text](/img/Powershell_Console_Veeam.webp#center)

2. Importing the module from a traditional Powershell console.

```powershell
PS C:\Users\jocolon> Import-Module -Name Veeam.Backup.PowerShell

WARNING: The names of some imported commands from the module 'Veeam.Backup.PowerShell' include unapproved verbs that might make them less discoverable. To find the commands with unapproved verbs, run the Import-Module command again with the Verbose parameter. For a list of approved verbs, type Get-Verb.
PS C:\Users\jocolon> 
```

It is important to mention that the Powershell module is available on the Veeam Backup server or on any device where the management console is installed. Reference:

> The remote machine from which you run Veeam PowerShell commands must have the Veeam Backup & Replication Console installed. After you install the Veeam Backup & Replication Console, Veeam PowerShell module will be installed by default
>
> [Veeam PowerShell Reference](https://helpcenter.veeam.com/docs/backup/powershell/)

To validate if the module was loaded successfully you can use the `Get-Module` command. As you can see in the example version 1.0 of `Veeam.Backup.PowerShell` was loaded on the system.

```powershell
PS C:\Users\jocolon> Get-Module -ListAvailable -Name Veeam.Backup.PowerShell


    Directory: C:\Program Files\Veeam\Backup and Replication\Console

ModuleType Version    Name                                ExportedCommands
---------- -------    ----                                ----------------
Manifest   1.0        Veeam.Backup.PowerShell             {Get-VBRComputerFileProxyServer, New-VBRSanI

PS C:\Users\jocolon>
```

Speaking a bit of history, the traditional way to load the Powershell module in pre-V11 versions was to use the following command:

```powershell
 Add-PSSnapin -Name VeeamPSSnapIn #Veeam Powershell Pre11
```

Now in Veeam VBR version 11 the traditional `Import-Module` command is used like any other Powershell module. In the case of Veeam 11 we use the command referring to the module called `Veeam.Backup.PowerShell`. Once the module is loaded you can verify which cmdlets are available with the `Get-Command` cmdlet.

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

Now that everything is ready you can establish the initial connection to your Backup server. To accomplish this process use the command `Connect-VBRServer`. In the following example we use the option to request credentials but Veeam modules accept traditional methods to provide credentials in Powershell ([Referencias](https://duffney.io/addcredentialstopowershellfunctions/)).

```powershell
PS C:\Users\jocolon> Connect-VBRServer -Server veeam-vbr.pharmax.local -Credential (Get-Credential)

cmdlet Get-Credential at command pipeline position 1
Supply values for the following parameters:
User: pharmax\veeam_admin
Password for user pharmax\veeam_admin: ````
PS C:\Users\jocolon> 
```

In this example I connect to the server with the name `veeam-vbr.pharmax.local` with the username `pharmax\\veeam\_admin`. It is important to mention that only users with the `Veeam Backup Administrator` role can establish connection via powershell. If you try to connect with an account that does not have this privilege level, you will get the following error:

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

To validate existing sessions use the command `Get-VBRServerSession`. The following example shows the connection to the `veeam-vbr` server using the `veeam\_admin` user.

```powershell
PS C:\Users\jocolon> Get-VBRServerSession                 


User        Server                  Port
----        ------                  ----
veeam_admin veeam-vbr.pharmax.local 9392


PS C:\Users\jocolon>
```

Once connected to the Veeam server you can run any cmdlet to extract the required information. As an example I will show you the backup tasks that are created on my Veeam test server.

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

I hope this information is helpful. If you have any doubts or questions about this post, leave them in the comments. Hasta Luego Amigos!

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/F1F8DEV80)

![Text](/img/backups-backups-everywhere.webp#center)
