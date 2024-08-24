---
title: 'Veeam: Powershell get Backup Proxies Information'
date: '2021-07-10T13:35:00-04:00'
tags:
  - VEEAM
  - Powershell
---

Hola a todos,

En esta oportunidad les estaré mostrando como obtener desde powershell la información relacionada con los `Backup Proxy` en una infraestructura de `Veeam Backup & Replication`. Para comenzar es necesario establecer la conexión inicial al `Backup Server` utilizando el comando `Connect-VBRServer`. En mi caso el FQDN de mi Servidor de Backup es el `veeam-vbr.pharmax.local`.

```text
PS C:\Users\jocolon> Connect-VBRServer -Server veeam-vbr.pharmax.local -Credential (Get-Credential)

cmdlet Get-Credential at command pipeline position 1
Supply values for the following parameters:
User: pharmax\administrator
Password for user pharmax\administrator: ````
PS C:\Users\jocolon> 
```

Luego de conectarnos al servidor podemos utilizar el cmdlet `Get-VBRViProxy` para identificar cuáles son los servidores `Backup Proxy` en nuestra infraestructura.

```text
PS C:\Users\jocolon> Get-VBRViProxy | Format-Table -Wrap -AutoSize


Name                        Type Host                        IsDisabled Description
----                        ---- ----                        ---------- -----------
veeam-lnx-px.pharmax.local  Vi   veeam-lnx-px.pharmax.local  False      Created by PHARMAX\jocolon...
VMware Backup Proxy         Vi   VEEAM-VBR.pharmax.local     False      Created by Veeam Backu...
VEEAM-VBR-02V.pharmax.local Vi   VEEAM-VBR-02V.pharmax.local False      Created by PHARMAX\jocolon ...


PS C:\Users\jocolon>
```

Como pueden ver este comando muestra información básica sobre los `Backup Proxy`, pero si desean ver el contenido completo se puede intercambiar el comando de `Format-Table` por `Format-list`.

```text
PS C:\Users\jocolon> Get-VBRViProxy | Format-List                 

Id                : ae2cd836-9e0a-440c-9b51-089e71c16ac6
Name              : veeam-lnx-px.pharmax.local
Description       : Created by PHARMAX\jocolon at 12/28/2021 1:24 PM.
Info              : Veeam.Backup.Model.CBackupProxyInfo
HostId            : cd06c723-875d-4da3-a633-ccac571ee296 
Host              : Veeam.Backup.Core.Common.CHost       
Type              : Vi
IsDisabled        : False
Options           : Veeam.Backup.Model.CDomViProxyOptions
MaxTasksCount     : 2
UseSsl            : False
FailoverToNetwork : True
TransportMode     : Auto
ChosenVm          : 
ChassisType       : ViVirtual

Id                : 18b661c1-d9dc-4233-90a0-7e7b10dc2d09
Name              : VMware Backup Proxy
Description       : Created by Veeam Backup & Replication
Info              : Veeam.Backup.Model.CBackupProxyInfo
HostId            : 6745a759-2205-4cd2-b172-8ec8f7e60ef8
Host              : Veeam.Backup.Core.Common.CHost
Type              : Vi
IsDisabled        : False
Options           : Veeam.Backup.Model.CDomViProxyOptions
MaxTasksCount     : 2
UseSsl            : False
FailoverToNetwork : True
TransportMode     : Auto
ChosenVm          :
ChassisType       : ViVirtual

Id                : 07c6322d-8e42-4929-8015-87c002ec0838
Name              : VEEAM-VBR-02V.pharmax.local
Description       : Created by PHARMAX\jocolon at 12/22/2021 8:46 PM.
Info              : Veeam.Backup.Model.CBackupProxyInfo
HostId            : 8452413e-92ee-4461-951d-516c7cf27cec
Host              : Veeam.Backup.Core.Common.CHost
Type              : Vi
IsDisabled        : False
Options           : Veeam.Backup.Model.CDomViProxyOptions
MaxTasksCount     : 2
UseSsl            : False
FailoverToNetwork : True
TransportMode     : Auto
ChosenVm          :
ChassisType       : ViVirtual


PS C:\Users\jocolon> 
```

Para culminar les dejo un pequeño código para ver mejor el contenido sobre los `Backup Proxy` de Veeam:

```powershell
$BackupProxies = Get-VBRViProxy
$Outobj = @()
foreach ($BackupProxy in $BackupProxies) {
    $inObj = [ordered] @{
        'Name' = $BackupProxy.Name
        'Host Name' = $BackupProxy.Host.Name
        'Disabled' = $BackupProxy.IsDisabled
        'Max Tasks Count' = $BackupProxy.MaxTasksCount
        'Use Ssl' = $BackupProxy.UseSsl
        'Failover To Network' = $BackupProxy.FailoverToNetwork
        'Transport Mode' = $BackupProxy.TransportMode
        'Chassis Type' = $BackupProxy.ChassisType
        'OS Type' = $BackupProxy.Host.Type
        'Services Credential' = $BackupProxy.Host.ProxyServicesCreds.Name
        'Status' = Switch (($BackupProxy.Host).IsUnavailable) {
            'False' {'Available'}
            'True' {'Unavailable'}
            default {($BackupProxy.Host).IsUnavailable}
        }
    }
    $OutObj += [pscustomobject]$inobj
}
$OutObj | Format-Table -Wrap -AutoSize
```

Aquí les dejo el resultado del código.

![Text](/img/BackupProxy_Code-scaled.webp#center)

## Hasta Luego Amigos!

![Text](/img/i-got-a-5b205b.webp#center)
