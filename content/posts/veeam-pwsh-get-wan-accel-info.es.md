---
title: 'Veeam: Obtener información de los Aceleradores WAN utilizando Powershell'
date: '2024-09-22T00:00:00-00:00'
author: 'Jonathan Colon'
tags:
    - VEEAM
    - Powershell
---

Hola a todos,

Estos últimos días he estado desarrollando varios scripts relacionados con `Veeam Backup & Replication` donde he tenido la oportunidad de utilizar más los módulos de `Powershell` de Veeam. De manera que en esta oportunidad les estaré mostrando de forma básica como obtener información relacionada con los aceleradores WAN configurados en el servidor de Backup.

Como siempre es necesario establecer conexión con el servidor de Backup utilizando el comando `Connect-VBRServer`.

```powershell
PS C:\Users\jocolon> Connect-VBRServer -Server veeam-vbr.pharmax.local -Credential (Get-Credential)

cmdlet Get-Credential at command pipeline position 1
Supply values for the following parameters:
User: pharmax\administrator
Password for user pharmax\administrator: ````

PS C:\Users\jocolon> 
```

Luego de conectarnos al servidor podemos utilizar el comando `Get-VBRWANAccelerator` para identificar los aceleradores WAN conectados al servidor de Backup.

```powershell
PS C:\Users\jocolon> Get-VBRWANAccelerator                                           

Id          : 460d48bc-9e6f-4330-8e05-05d7cdc035cd
Name        : VEEAM-REPO-01V.pharmax.local
Description : Created by PHARMAX\administrator at 10/12/2022 1:20 PM.
HostId      : f551fdee-3479-4631-86d2-863c27440b22
Options     : Veeam.Backup.Core.CDomWaOptions

Id          : 52dba096-8c13-4c8c-a548-3dffa1d1ae3a
Name        : VEEAM-VBR
Description : Created by PHARMAX\administrator at 12/15/2019 7:55 PM.
HostId      : 6745a759-2205-4cd2-b172-8ec8f7e60ef8
Options     : Veeam.Backup.Core.CDomWaOptions

Id          : a77760ff-f5bf-4ab0-9d95-4668c88724b7
Name        : VEEAM-WAN-01V.pharmax.local
Description : Created by PHARMAX\administrator at 10/12/2022 1:37 PM.
HostId      : f25bb5e2-c08b-47e4-b338-2f15a68592e8
Options     : Veeam.Backup.Core.CDomWaOptions

Id          : 9e8fbcf6-d65a-4079-b77c-50efe60d2194
Name        : VEEAM-VBR-02V.pharmax.local
Description : Created by PHARMAX\jocolon at 12/20/2021 12:53 PM.
HostId      : 8452413e-92ee-4461-951d-516c7cf27cec
Options     : Veeam.Backup.Core.CDomWaOptions


PS C:\Users\jocolon> 
```

Como pueden ver el resultado del cmdlet suena un tanto incompleto, ya que no muestra cierta información importante como el puerto TCP utilizado o el espacio en caché configurado. Par lograr obtener esta información les comparto un ejemplo básico que permite obtener información más relevante.

```powershell
$OutObj = @()
$WANAccels = Get-VBRWANAccelerator | Sort-Object -Property Name
foreach ($WANAccel in $WANAccels) {
    $IsWaHasAnyCaches = 'Unknown'
    try {
        Write-Verbose "Discovered $($WANAccel.Name) Wan Accelerator."
        try {
            $IsWaHasAnyCaches = $WANAccel.IsWaHasAnyCaches()
        } catch {
            Write-Verbose "Wan Accelerator $($WANAccel.Name) IsWaHasAnyCaches() Item: $($_.Exception.Message)"
        }
        try {
            $ServiceIPAddress = $WANAccel.GetWaConnSpec().Endpoints.IP -join ", "
        } catch {
            Write-Verbose "Wan Accelerator $($WANAccel.Name) GetWaConnSpec() Item: $($_.Exception.Message)"
        }
        $inObj = [ordered] @{
            'Name' = $WANAccel.Name
            'Host Name' = $WANAccel.GetHost().Name
            'Is Public' = $WANAccel.GetType().IsPublic
            'Management Port' = "$($WANAccel.GetWaMgmtPort())\TCP"
            'Service IP Address' = $ServiceIPAddress
            'Traffic Port' = "$($WANAccel.GetWaTrafficPort())\TCP"
            'Max Tasks Count' = $WANAccel.FindWaHostComp().Options.MaxTasksCount
            'Download Stream Count' = $WANAccel.FindWaHostComp().Options.DownloadStreamCount
            'Enable Performance Mode' = $WANAccel.FindWaHostComp().Options.EnablePerformanceMode
            'Configured Cache' = $IsWaHasAnyCaches
            'Cache Path' = $WANAccel.FindWaHostComp().Options.CachePath
            'Max Cache Size' = "$($WANAccel.FindWaHostComp().Options.MaxCacheSize) $($WANAccel.FindWaHostComp().Options.SizeUnit)"
        }
        $OutObj = [pscustomobject]$inobj
    } catch {
        Write-Verbose "Wan Accelerator $($WANAccel.Name) Table: $($_.Exception.Message)"
    }
}
```

En el siguiente ejemplo les muestro el resultado del código para obtener la información sobre los aceleradores WAN conectados a nuestro servidor de Backup de Veeam.

```powershell

Name                    : VEEAM-REPO-01V.pharmax.local
Host Name               : VEEAM-REPO-01V.pharmax.local
Is Public               : True
Management Port         : 6164\TCP
Service IP Address      :
Traffic Port            : 6165\TCP
Max Tasks Count         : 1
Download Stream Count   : 5
Enable Performance Mode : False
Configured Cache        : Unknown
Cache Path              : E:\VeeamWAN
Max Cache Size          : 10 GB

Name                    : VEEAM-VBR
Host Name               : VEEAM-VBR.pharmax.local
Is Public               : True
Management Port         : 6164\TCP
Service IP Address      :
Traffic Port            : 6165\TCP
Max Tasks Count         : 1
Download Stream Count   : 5
Enable Performance Mode : True
Configured Cache        : False
Cache Path              : E:\VeeamWAN
Max Cache Size          : 10 GB

Name                    : VEEAM-VBR-02V.pharmax.local
Host Name               : VEEAM-VBR-02V.pharmax.local
Is Public               : True
Management Port         : 6164\TCP
Service IP Address      :
Traffic Port            : 6165\TCP
Max Tasks Count         : 1
Download Stream Count   : 5
Enable Performance Mode : False
Configured Cache        : Unknown
Cache Path              : E:\VeeamWAN
Max Cache Size          : 100 GB

Name                    : VEEAM-WAN-01V.pharmax.local
Host Name               : VEEAM-WAN-01V.pharmax.local
Is Public               : True
Management Port         : 6164\TCP
Service IP Address      :
Traffic Port            : 6165\TCP
Max Tasks Count         : 1
Download Stream Count   : 5
Enable Performance Mode : False
Configured Cache        : Unknown
Cache Path              : C:\VeeamWAN
Max Cache Size          : 10 GB

PS C:\Users\jocolon> 
```

Como pueden ver podemos de manera avanzada extender la información de cualquier comando de PowerShell utilizando objetos tipo `PSCustomObject`. Con esto podemos crear una tabla con la información necesaria sin tener que crear un cmdlet adicional para este propósito.

Espero que este post les haya gustado y les sea de utilidad en su jornada profesional.

### Hasta Luego Amigos

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/F1F8DEV80)
