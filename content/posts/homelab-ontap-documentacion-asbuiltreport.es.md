---
title: 'HomeLab: Documentación automatizada utilizando AsBuiltReport'
date: '2021-09-27T15:05:57-04:00'
tags:
    - NetApp
---

Hola a tod@s!

En esta oportunidad vengo a hablarles sobre un proyecto el cual he estado trabajando durante unos meses. Si bien recordarán hace un tiempo atrás escribí sobre la herramienta AsBuildReport específicamente el módulo para documentar las infraestructuras basadas en VMware vSphere ver el post [here](http://192.168.7.40/2021/06/06/homelab-automated-vmware-infrastructure-documentation/). Pues hoy vengo a mostrarle un módulo que ayude a desarrollar con el propósito de documentar los dispositivos de almacenamiento de la compañía NetApp específicamente sobre el sistema operativo ONTAP.

El reporte se encuentra en estado inicial y en constante desarrollo, pero decidí liberarlo públicamente para poder recibir recomendaciones o más bien para fomentar que otros desarrolladores aporte a mejorar su contenido. El website de desarrollo del reporte se encuentra en Github les dejo el enlace para que puedan ver el alcance y el objetivo del proyecto.

<https://github.com/AsBuiltReport/AsBuiltReport.NetApp.ONTAP>

![Text](/img/AsBuildReport_NetApp_ONTAP.webp#center)

Debo aclarar que este reporte no está diseñado para reemplazar o competir de ninguna forma con la herramienta **NetAppDocs**. Me percate de las muchas peticiones en el foro de NetApp de varios usuarios que tienen la necesidad de generar un reporte actualizado de su infraestructura de almacenamiento. Es por esta razón que tome la iniciativa de crear un reporte disponible de forma gratuita para los clientes y/o usuarios de NetApp. Una ventaja de utilizar el proyecto **AsBuiltReport**, es que te permite crear el reporte en múltiples formato (Word,Html o Texto) y hasta puedes automatizar su envío a través de correo electrónico.

Ahora bien, para comenzar necesitamos cumplir con los siguientes requisitos:

- Multi-plataforma Windows, Linux o MAC
- PowerShell v5.1+ ó v7
- El módulo de «NetApp PowerShell Toolkit» >= 9.9.1.2106
- El módulo de AsBuiltReport.Core >= 1.1.0

Este reporte utiliza la versión de PowerShell 5.+ ó PSCore 7, para validar la versión podemos utilizar la variable $PSVersionTable desde la consola de PowerShell:

```text
PS /home/rebelinux> >$PSVersionTable
>
Name                           Value
----                           -----
>PSVersion>                      >7.1.4>
PSEdition                      Core
GitCommitId                    7.1.4
OS                             Linux 5.14.2-arch1-2 #1 SMP PREEMPT Thu, 09 Sep 2021 09:42:35 +0000
Platform                       Unix
PSCompatibleVersions           {1.0, 2.0, 3.0, 4.0…}
PSRemotingProtocolVersion      2.3
SerializationVersion           1.1.0.1
WSManStackVersion              3.0

PS /home/rebelinux>
```

El reporte fue creado específicamente para la versión de **«NetApp PowerShell Toolkit»** >= 9.9.1.2106. Para validar que versión tenemos o si ha sido instalada podemos utilizar el comando **Get-Module** según se muestra en el siguiente ejemplo:

#### Nota: Adicionalmente validamos la versión de «AsBuiltReport.Core» que es una dependencia adicional para poder generar el reporte

```text
PS /home/rebelinux> >Get-Module> -ListAvailable -Name >@('AsBuiltReport.Core','Netapp.Ontap')>                                    

    Directory: /home/rebelinux/.local/share/powershell/Modules

ModuleType Version    PreRelease Name                                PSEdition ExportedCommands
---------- -------    ---------- ----                                --------- ----------------
Script     >1.1.0>                 >AsBuiltReport.Core>                  Desk      {New-AsBuiltReport, New-AsBuiltConfig}
Manifest   >9.9.1.2106>            >NetApp.ONTAP >                       Desk      Add-NaHelpInfoUri

PS /home/rebelinux>
```

Si el comando no produce algún resultado quiere decir que ninguno de los módulos está instalado. Para instalar estas dependencias utilizamos el comando **Install-Module**:

```text
PS /home/rebelinux> >Install-Module> -Name >@('AsBuiltReport.Core','Netapp.Ontap')>
Installing 2 modules.
Installing Module 'NetApp.ONTAP' (2 of 2).
[ooooooooooooooooooooooooooooooooooooooooooooooooooo]
Installing package 'NetApp.ONTAP'
Downloaded 1.47 MB out of 14.60 MB.
[oooooooooo                                         ]
PS /home/rebelinux>
```

Luego procedemos a instalar el reporte de ONTAP utilizando el siguiente comando:

```text
PS /home/rebelinux> >Install-Module> -Name >AsBuiltReport.NetApp.ONTAP>
Installing 1 modules.
Installing Module 'AsBuiltReport.NetApp.ONTAP' (1 of 1).
[ooooooooooooooooooooooooooooooooooooooooooooooooooo]
Installing package 'AsBuiltReport.NetApp.ONTAP'
Downloaded 0.2 MB out of 0.88 MB.
[oooooooooo                                         ]
PS /home/rebelinux>
```

Para validar si la instalación fue exitosa utilizamos el comando **Get-Module**.

```text
PS /home/rebelinux> >Get-Module> -ListAvailable >"AsBuiltReport.NetApp.ONTAP"> 


    Directory: C:\Program Files\WindowsPowerShell\Modules

ModuleType Version    PreRelease Name                                PSEdition ExportedCommands
---------- -------    ---------- ----                                --------- ----------------
Script     >0.4.0>                 >AsBuiltReport.NetApp.ONTAP>          Desk      Invoke-AsBuiltReport.NetApp.ONTAP

PS /home/rebelinux>
```

#### Nota: Como se puede ver se realizó la instalación del módulo versión 0.4.0

Un requisito opcional es generar los archivos de configuración que te permite establecer los parámetros de la organización que son utilizados para generar el reporte. Este proceso genera unos archivos tipo JSON que son utilizados como plantillas **“templates”** de forma que no tengas que llenar la información repetitiva cuando generes los reportes.

El «cmdlet» de powershell **New-AsBuiltConfig** te permite generar la plantilla que utilizaremos como base del reporte. Esta plantilla establece los parámetros no técnicos del reporte.

```text
PS C:\WINDOWS\system32> >New-AsBuiltConfig
>
---------------------------------------------
 <        As Built Report Information      >
---------------------------------------------
Enter the name of the Author for this As Built Report [jocolon]: >Jonathan Colon>
```

```text
---------------------------------------------
 <           Company Information           >
---------------------------------------------
Would you like to enter Company information for the As Built Report? (y/n): >y>
Enter the Full Company Name: >Zen PR Solutions>
Enter the Company Short Name: >ZENPR>
Enter the Company Contact: >Jonathan Colon>
Enter the Company Email Address: >jcolonf@zenprsolutions.com>
Enter the Company Phone: >XXX-XXX-XXXX>
Enter the Company Address:> Puerto Rico>
```

```text
---------------------------------------------
 <            Email Configuration          >
---------------------------------------------
Would you like to enter SMTP configuration? (y/n): >n>
```

```text
----------------------------------------------
 <       As Built Report Configuration      >
----------------------------------------------
Would you like to save the As Built Report configuration file? (y/n): >y>
Enter a name for the As Built Report configuration file [AsBuiltReport]: >HomeLab ONTAP Report>
Enter the path to save the As Built Report configuration file [>C:\Users\jocolon\AsBuiltReport>]:

Name                           Value
----                           -----
Email                          {Port, Credentials, Server, To...}
Company                        {FullName, Contact, Phone, Email...}
UserFolder                     {Path}
Report                         {Author}


PS C:\WINDOWS\system32>
```

Una vez culminado el proceso se creará un archivo tipo JSON con el siguiente contenido:

```json
{
    "Email":  {
                  "Port":  null,
                  "Credentials":  null,
                  "Server":  null,
                  "To":  null,
                  "From":  null,
                  "UseSSL":  null,
                  "Body":  null
              },
    "Company":  {
                    "FullName":  "Zen PR Solutions",
                    "Contact":  "Jonathan Colon",
                    "Phone":  "787-203-2790",
                    "Email":  "jcolonf@zenprsolutions.com",
                    "ShortName":  "ZENPR",
                    "Address":  "Puerto Rico"
                },
    "UserFolder":  {
                       "Path":  "C:\\Users\\jocolon\\AsBuiltReport"
                   },
    "Report":  {
                   "Author":  "Jonathan Colon"
               }
}
```

El comando **New-AsBuiltReportConfig** permite establecer los parámetros técnico del reporte como el nivel y tipo de información **«verbose level»**.

```text
PS C:\WINDOWS\system32> >New-AsBuiltReportConfig NetApp.ONTAP> -FolderPath C:\Users\jocolon\AsBuiltReport\
```

Una vez culminado el proceso se creará un archivo tipo JSON con el siguiente contenido:

```json
{
    "Report": {
        "Name": "NetApp ONTAP As Built Report",
        "Version": "0.3.0",
        "Status": "Released",
        "ShowCoverPageImage": false,
        "ShowTableOfContents": true,
        "ShowHeaderFooter": true,
        "ShowTableCaptions": true
    },
    "Options": {

    },
    "InfoLevel": {
        "_comment_": "0 = Disabled, 1 = Enabled",
        "Cluster": 1,
        "Node": 1,
        "Storage": 1,
        "License": 1,
        "Network": 1,
        "Vserver": 1,
        "Replication": 1,
        "Efficiency": 1,
        "Security": 1,
        "System": 1
    },
    "HealthCheck": {
        "Cluster": {
            "Summary": true,
            "HA": true,
            "AutoSupport": true
        },
        "Node": {
            "HW": true,
            "ServiceProcessor": true
        },
        "Storage": {
            "Aggr": true,
            "DiskStatus": true,
            "ShelfStatus": true,
            "Efficiency": true,
            "FabricPool": true
        },
        "Network": {
            "Port": true,
            "Interface": true
        },
        "License": {
            "RiskSummary": true,
            "ServiceProcessor": true
        },
        "Vserver": {
            "Status": true,
            "Iscsi": true,
            "FCP": true,
            "NFS": true,
            "Quota": true,
            "CIFS": true,
            "Snapshot": true
        },
        "Replication": {
            "Status": true,
            "ClusterPeer": true,
            "VserverPeer": true,
            "Relationship": true,
            "History": true,
            "Mediator": true
        },
        "System": {
            "NTP": true,
            "DNS": true,
            "EMS": true,
            "Backup": true
        },
        "Security": {
            "Users": true,
            "KMS": true
    }
    }
}
```

Luego podemos generar el reporte utilizando el comando **«New-AsBuiltReport -Report NetApp.ONTAP -Target IP/FQDN«**

```text
PS /home/rebelinux> >New-AsBuiltReport> -Report >NetApp.ONTAP> -AsBuiltConfigFilePath /home/rebelinux/script/AsBuiltReport.json -OutputFolderPath /home/rebelinux/script >-Target> >192.168.7.60> -Format HTML  >-EnableHealthCheck> -Credential $cred -ReportConfigFilePath /home/rebelinux/script/AsBuiltReport.NetApp.ONTAP.json               

WARNING: Encountered Error No 81 on volume NTAPSOL_LUN_1. Reason: Volume is offline
WARNING: Encountered Error No 81 on volume NTAPSOL_LUN_1. Reason: Volume is offline
WARNING: Encountered Error No 81 on volume NTAPSOL_LUN_1. Reason: Volume is offline
WARNING: Encountered Error No 81 on volume NTAPSOL_LUN_1. Reason: Volume is offline
WARNING: Encountered Error No 81 on volume NTAPSOL_LUN_1. Reason: Volume is offline
WARNING: Encountered Error No 81 on volume NTAPSOL_LUN_1. Reason: Volume is offline
WARNING: Encountered Error No 81 on volume NTAPSOL_LUN_1. Reason: Volume is offline
WARNING: [ 13:17:34:209 ] [ Document ] - List table captions are only supported on tables with a single row. Removing caption from table 'Vserver Lun Information - PHARMAX-HQ'.
WARNING: Encountered Error No 81 on volume NTAPSOL_LUN_1. Reason: Volume is offline
WARNING: Encountered Error No 81 on volume NTAPSOL_LUN_1. Reason: Volume is offline
WARNING: Encountered Error No 81 on volume NTAPSOL_LUN_1. Reason: Volume is offline
WARNING: Encountered Error No 81 on volume NTAPSOL_LUN_1. Reason: Volume is offline
WARNING: Encountered Error No 81 on volume NTAPSOL_LUN_1. Reason: Volume is offline
WARNING: Encountered Error No 81 on volume NTAPSOL_LUN_1. Reason: Volume is offline
WARNING: Encountered Error No 81 on volume NTAPSOL_LUN_1. Reason: Volume is offline
WARNING: Encountered Error No 81 on volume NTAPSOL_LUN_1. Reason: Volume is offline
WARNING: [ 13:17:36:487 ] [ Document ] - List table captions are only supported on tables with a single row. Removing caption from table 'Vserver Igroup Information - PHARMAX-HQ'.
WARNING: Encountered Error No 81 on volume NTAPSOL_LUN_1. Reason: Volume is offline
WARNING: [ 13:17:37:516 ] [ Document ] - List table captions are only supported on tables with a single row. Removing caption from table 'Vserver NFS Service Options Summary - PHARMAX-HQ'.
WARNING: [ 13:17:49:537 ] [ Document ] - List table captions are only supported on tables with a single row. Removing caption from table 'Vserver CIFS Service Information - PHARMAX-HQ'.
WARNING: [ 13:17:50:264 ] [ Document ] - List table captions are only supported on tables with a single row. Removing caption from table 'Vserver CIFS Service Security Information - PHARMAX-HQ'.
WARNING: [ 13:17:51:834 ] [ Document ] - List table captions are only supported on tables with a single row. Removing caption from table 'Vserver CIFS Service Options Summary - PHARMAX-HQ'.
WARNING: [ 13:17:55:913 ] [ Document ] - List table captions are only supported on tables with a single row. Removing caption from table 'Replication - SnapMirror relationship Information - PHARMAX-HQ'.
WARNING: [ 13:17:56:662 ] [ Document ] - List table captions are only supported on tables with a single row. Removing caption from table 'Replication - SnapMirror Destination (List-Destinations) Information - PHARMAX-HQ'.
>NetApp ONTAP As Built Report 'NetApp ONTAP As Built Report' has been saved to '/home/rebelinux/script'.
>PS /home/rebelinux> 
```

Aquí les dejo el ejemplo del reporte generado.

{{< embed-pdf url="./img/Sample-NetApp-ONTAP-As-Built-Report.pdf" >}}

Adicionalmente les incluyo varios ejemplos de cómo generar el reporte. ¡Espero sea de su agrado!

```text
# Generate a NetApp ONTAP As Built Report for Cluster array '192.168.7.60' using specified credentials. Export report to HTML & DOCX formats. Use default report style. Append timestamp to report filename. Save reports to 'C:\Users\Jon\Documents'>

PS C:\> New-AsBuiltReport -Report NetApp.ONTAP -Target 192.168.7.60 -Username 'admin' -Password 'P@ssw0rd' -Format Html,Word -OutputFolderPath 'C:\Users\Jon\Documents' -Timestamp>

# Generate a NetApp ONTAP As Built Report for Cluster array 192.168.7.60 using specified credentials and report configuration file. Export report to Text, HTML & DOCX formats. Use default report style. Save reports to 'C:\Users\Jon\Documents'. Display verbose messages to the console.>

PS C:\> New-AsBuiltReport -Report NetApp.ONTAP -Target 192.168.7.60 -Username 'admin' -Password 'P@ssw0rd' -Format Text,Html,Word -OutputFolderPath 'C:\Users\Jon\Documents' -ReportConfigFilePath 'C:\Users\Jon\AsBuiltReport\AsBuiltReport.NetApp.ONTAP.json' -Verbose>

# Generate a NetApp ONTAP As Built Report for Cluster array 192.168.7.60 using stored credentials. Export report to HTML & Text formats. Use default report style. Highlight environment issues within the report. Save reports to 'C:\Users\Jon\Documents'.>

PS C:\> $Creds = Get-Credential
PS C:\> New-AsBuiltReport -Report NetApp.ONTAP -Target 192.168.7.60 -Credential $Creds -Format Html,Text -OutputFolderPath 'C:\Users\Jon\Documents' -EnableHealthCheck

# Generate a NetApp ONTAP As Built Report for Cluster array 192.168.7.60 using stored credentials. Export report to HTML & DOCX formats. Use default report style. Reports are saved to the user profile folder by default. Attach and send reports via e-mail.>

PS C:\> New-AsBuiltReport -Report NetApp.ONTAP -Target 192.168.7.60 -Username 'admin' -Password 'P@ssw0rd' -Format Html,Word -OutputFolderPath 'C:\Users\Jon\Documents' -SendEmail>
```

Hasta Luego Amigos!

![Text](/img/infrastructure-so-complex-no-body-that-knows-how-it-was-35156082.webp#center)
