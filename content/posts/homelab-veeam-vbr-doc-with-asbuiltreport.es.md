---
title: 'HomeLab: Documentación de Veeam VBR utilizando AsBuiltReport'
date: '2024-02-20T10:50:00-04:00'
tags:
    - VEEAM
---

Hola a tod@s!

Como ya bien saben debido al impacto de la pandemia de Covid-19 no he tenido mucho espacio para hacer otra cosa que no sea estar encerrado en mi casa 😒. Más aún cuando tantos de mis familiares han dado positivo al virus. De manera que esta situación ha sido la mejor excusa para ponerme a programar. Recientemente he recibido muchas peticiones de varios usuario y amigos para que desarrolle un reporte que documentación de la aplicación de Backup `Veeam Backup & Replication`.

Por esta razón, me di a la tarea de desarrollar otro reporte más, sí otro más 🤣 y está relacionado a documentar las implementaciones de `Veeam Backup & Replication` específicamente para la versión `12+` en adelante. Este reporte utiliza como base el `framework` del proyecto de AsBuildReport creado por [Tim Carman](https://www.asbuiltreport.com/).

El website de desarrollo del reporte se encuentra en Github les dejo el enlace para que puedan ver el alcance y el objetivo del proyecto.

- <https://github.com/AsBuiltReport/AsBuiltReport.Veeam.VBR>

![Text](/img/Veeam_VBR_Portal.webp#center)

** Importante: El reporte se encuentra actualmente disponible en `PowerShell Gallery`.

Ahora bien, para comenzar necesitamos cumplir con los siguientes requisitos:

- Plataforma de Windows (Los módulos de Veeam solo corren en Windows)
- PowerShell v5.1+
- El módulo de AsBuiltReport.Core >= 1.3.0
- El módulo de Veeam.Backup.PowerShell >= 1.0
- El módulo de Veeam.Diagrammer >= 0.5.9
- El modulo de Diagrammer.Core >= 0.2.0
- El modulo de PSGraph >= 2.1.38.27
- El modulo de Pscribo >= 0.10.0
- El modulo de PscriboCharts >= 0.9.0


Este reporte utiliza la versión de PowerShell 5.+, para validar la versión podemos utilizar la variable `$PSVersionTable` desde la consola de PowerShell:

```powershell
PS C:\Users\jocolon> $PSVersionTable

Name                           Value
----                           -----
PSVersion                      5.1.19041.3803
PSEdition                      Desktop
PSCompatibleVersions           {1.0, 2.0, 3.0, 4.0...}
BuildVersion                   10.0.19041.3803
CLRVersion                     4.0.30319.42000
WSManStackVersion              3.0
PSRemotingProtocolVersion      2.3
SerializationVersion           1.1.0.1

PS C:\Users\jocolon>
```

Para validar si tenemos los módulos requeridos de Veeam podemos utilizar el comando `Get-Module -ListAvailable -Name @('Veeam.Backup.PowerShell') | Format-Table -AutoSize` según se muestra en el siguiente ejemplo:

```powershell
PS C:\Users\jocolon> Get-Module -ListAvailable -Name @('Veeam.Backup.PowerShell') | Format-Table -AutoSize

    Directory: C:\Program Files\Veeam\Backup and Replication\Console

ModuleType Version     Name                    ExportedCommands
---------- -------     ----                    ----------------
Manifest   12.1.0.2131 Veeam.Backup.PowerShell {Get-VBRAmazonEC2VM, Get-VBRAzureVM, New-VBRAzureTag, New-VBRHealthCheckOptions...}

PS C:\Users\jocolon>
```

Si el comando no produce algún resultado quiere decir que el módulos no están instalados. Ahora bien para instalar los módulos de `Veeam.Backup.PowerShell` es importante mencionar que estos están disponible en el servidor de Backup de Veeam o en cualquier dispositivo donde la consola de manejo esté instalada. 

#### Referencia:

> The remote machine from which you run Veeam PowerShell commands must have the Veeam Backup & Replication Console installed. After you install the Veeam Backup & Replication Console, Veeam PowerShell module will be installed by default
>
> [Veeam PowerShell Reference](https://helpcenter.veeam.com/docs/backup/powershell/)

Para instalar el reporte ``AsBuiltReport.Veeam.VBR``  desde `PowerShell Gallery` utilizamos el comando tradicional `Install-Module -Name AsBuiltReport.Veeam.VBR`:

```powershell
PS C:\Users\jocolon> Install-Module -Name AsBuiltReport.Veeam.VBR  
                                                                                                                        
Installing package 'AsBuiltReport.Veeam.VBR' 
[Downloaded 16.73 MB out of 23.76 MB.]

PS C:\Users\jocolon> 
```

Para confirmar si se han instalado todas las dependencias puede utilizar el cmdlet `Get-Module -ListAvailable -Name @('Veeam.Backup.PowerShell','PScribo', 'PScriboCharts', 'AsBuiltReport.Core', 'AsBuiltReport.Veeam.VBR', 'Veeam.Diagrammer', 'Diagrammer.Core', 'PSGraph') | select-object -Property Name, Version | Format-Table -AutoSize`.

```powershell
PS C:\Users\jocolon> Get-Module -ListAvailable -Name @('Veeam.Backup.PowerShell','PScribo', 'PScriboCharts', 'AsBuiltReport.Core', 'AsBuiltReport.Veeam.VBR', 'Veeam.Diagrammer', 'Diagrammer.Core', 'PSGraph') | select-object -Property Name, Version | Format-Table -AutoSize

Name                    Version    
----                    -------
AsBuiltReport.Veeam.VBR 0.8.5
Veeam.Diagrammer        0.5.9
Diagrammer.Core         0.2.0
AsBuiltReport.Core      1.3.0
PScribo                 0.10.0
PScriboCharts           0.9.0
PSGraph                 2.1.38.27
Veeam.Backup.PowerShell 12.1.0.2131

PS C:\Users\jocolon>
```

`Si por alguna razón no puedes instalar el informe a través de la PowerShellGallery te dejo aquí el método de instalación manual.`

Un requisito opcional es generar los archivos de configuración que te permite establecer los parámetros de la organización que son utilizados para generar el reporte. Este proceso genera unos archivos tipo JSON que son utilizados como plantillas `templates` de forma que no tengas que llenar la información repetitiva cuando generes los reportes.

#### Archivos de configuración (AsBuiltReport JSON`)

El cmdlet de powershell `New-AsBuiltConfig` te permite generar la plantilla que utilizaremos como base del reporte. Esta plantilla establece los parámetros no técnicos del reporte.

```powershell
PS C:\Users\jocolon>  New-AsBuiltConfig

---------------------------------------------
 <        As Built Report Information      >
---------------------------------------------
Enter the name of the Author for this As Built Report [jocolon]: Jonathan Colon
```

```powershell
---------------------------------------------
 <           Company Information           >
---------------------------------------------
Would you like to enter Company information for the As Built Report? (y/n): y
Enter the Full Company Name: Zen PR Solutions
Enter the Company Short Name: ZENPR
Enter the Company Contact: Jonathan Colon
Enter the Company Email Address: jcolonf@zenprsolutions.com
Enter the Company Phone: XXX-XXX-XXXX
Enter the Company Address: Puerto Rico
```

```powershell
---------------------------------------------
 <            Email Configuration          >
---------------------------------------------
Would you like to enter SMTP configuration? (y/n): n
```

```powershell
----------------------------------------------
 <       As Built Report Configuration      >
----------------------------------------------
Would you like to save the As Built Report configuration file? (y/n): y
Enter a name for the As Built Report configuration file [AsBuiltReport]: HomeLab Veeam Report
Enter the path to save the As Built Report configuration file [C:\Users\jocolon\AsBuiltReport]:

Name                           Value
----                           -----
Email                          {Port, Credentials, Server, To...}
Company                        {FullName, Contact, Phone, Email...}
UserFolder                     {Path}
Report                         {Author}


PS C:\Users\jocolon>
```

Una vez culminado el proceso se creará un archivo tipo JSON con el siguiente contenido:

```json
{
  "Company": {
    "FullName": "As Built Report",
    "Phone": "888-888-8888",
    "Address": "Saun Juan, Pr",
    "ShortName": "ZENPR",
    "Contact": "Jonathan Colon",
    "Email": "jcolonf@zenprsolutions.com"
  },
  "Email": {
    "Credentials": true,
    "Body": "Reporte",
    "From": "jcolonf@zenprsolutions.com",
    "UseSSL": true,
    "Server": "smtp.gmail.com",
    "To": [
      "rebelinux@gmail.com"
    ],
    "Port": "587"
  },
  "Report": {
    "Author": "As Built Report"
  },
  "UserFolder": {
    "Path": ".\\AsBuiltReport"
  }
}

```

El comando `New-AsBuiltReportConfig` permite establecer los parámetros técnico del reporte como el nivel y tipo de información `verbose level`.

```powershell
PS C:\Users\jocolon> New-AsBuiltReportConfig Veeam.VBR -FolderPath C:\Users\jocolon\AsBuiltReport\
```

Una vez culminado el proceso se creará un archivo tipo JSON con el siguiente contenido:

```json
{
    "Report": {
        "Name": "Veeam Backup & Replication As Built Report",
        "Version": "1.0",
        "Status": "Released",
        "ShowCoverPageImage": true,
        "ShowTableOfContents": true,
        "ShowHeaderFooter": true,
        "ShowTableCaptions": true
    },
    "Options": {
        "BackupServerPort": 9392,
        "PSDefaultAuthentication": "Default",
        "EnableCharts": false,
        "EnableHardwareInventory": false,
        "EnableDiagrams": false
    },
    "InfoLevel": {
        "_comment_": "Please refer to the AsBuiltReport project contributing guide for information about how to define InfoLevels.",
        "_comment_": "0 = Disabled, 1 = Enabled, 2 = Adv Summary, 3 = Detailed",
        "Infrastructure": {
            "BackupServer": 1,
            "Proxy": 1,
            "Settings": 1,
            "BR": 1,
            "Licenses": 1,
            "SOBR": 1,
            "WANAccel": 1,
            "ServiceProvider": 1,
            "SureBackup": 1

        },
        "Tape": {
            "Server": 1,
            "Library": 1,
            "MediaPool": 1,
            "Vault": 1,
            "NDMP": 1
        },
        "Inventory": {
            "VI": 1,
            "PHY": 1,
            "FileShare": 1
        },
        "Storage": {
            "ONTAP": 1,
            "ISILON": 1
        },
        "Replication": {
            "FailoverPlan": 1,
            "Replica": 1
        },
        "CloudConnect": {
            "Certificate": 1,
            "PublicIP": 1,
            "CloudGateway": 1,
            "GatewayPools": 1,
            "Tenants": 1,
            "BackupStorage": 1,
            "ReplicaResources": 1
        },
        "Jobs": {
            "Backup": 1,
            "BackupCopy": 1,
            "Tape": 1,
            "Surebackup": 1,
            "Agent": 1,
            "FileShare": 1,
            "Replication": 1
        }
    },
    "HealthCheck": {
        "Infrastructure": {
            "BackupServer": true,
            "Proxy": true,
            "Settings": true,
            "BR": true,
            "SOBR": true,
            "Server": true,
            "Status": true,
            "BestPractice": true
        },
        "Tape": {
            "Status": true,
            "BestPractice": true
        },
        "Replication": {
            "Status": true,
            "BestPractice": true
        },
        "Security": {
            "BestPractice": true
        },
        "CloudConnect": {
            "Tenants": true,
            "BackupStorage": true,
            "ReplicaResources": true,
            "BestPractice": true
        },
        "Jobs": {
            "Status": true,
            "BestPractice": true
        }
    }
}
```

Este archivo de configuración se puede utilizar para especificar el nivel de detalle del reporte como también que sesiones del reporte van a ser habilitadas.

Luego podemos generar el reporte utilizando el comando `New-AsBuiltReport -Report Veeam.VBR -Target Backup_Server_FQDN_or_IP -AsBuiltConfigFilePath AsBuiltReport.json -OutputFolderPath C:\Users\jocolon\AsBuiltReport\ -Credential $Cred -Format HTML -ReportConfigFilePath AsBuiltReport.Veeam.VBR.json -EnableHealthCheck -Verbose`. Es importante recalcar que es requerido utilizar la dirección del servidor de backup de Veeam como `Target`.

```powershell
# Build the Credential Object
PS C:\Users\jocolon> $Creds = Get-Credential

# Run the report
PS C:\Users\jocolon> New-AsBuiltReport -Report Veeam.VBR -Target 'backup-server.fqdn.local' -AsBuiltConfigFilePath AsBuiltReport.json -OutputFolderPath C:\Users\jocolon\AsBuiltReport\ -Credential $Creds -Format HTML -ReportConfigFilePath AsBuiltReport.Veeam.VBR.json -EnableHealthCheck

Please wait while the Veeam VBR As Built Report is being generated.
WARNING: [ 09:52:51:541 ] [ Document ] - Please refer to the AsBuiltReport.Veeam.VBR github website for more detailed information about this project.
WARNING: [ 09:52:51:557 ] [ Document ] - Do not forget to update your report configuration file after each new version release.
WARNING: [ 09:52:51:557 ] [ Document ] - Documentation: https://github.com/AsBuiltReport/AsBuiltReport.Veeam.VBR
WARNING: [ 09:52:51:557 ] [ Document ] - Issues or bug reporting: https://github.com/AsBuiltReport/AsBuiltReport.Veeam.VBR/issues
WARNING: [ 09:52:51:682 ] [ Document ] - AsBuiltReport.Veeam.VBR 0.8.5 is currently installed.
Veeam VBR As Built Report 'Veeam Backup & Replication As Built Report' has been saved to 'C:\Users\jocolon\AsBuiltReport\'.
```

Aquí les dejo el ejemplo del reporte generado.

{{< embed-pdf url="./img/Veeam-VBR-As-Built-Report.pdf" >}}

&nbsp;

Adicionalmente les incluyo varias opciones de cómo invocar el reporte.

```powershell
# Generate a Veeam VBR As Built Report for Backup Server 'backup-server.fqdn.local' using specified credentials. Export report to HTML & DOCX formats. Use default report style. Append timestamp to report filename. Save reports to 'C:\Users\Jon\Documents'

PS C:\> New-AsBuiltReport -Report Veeam.VBR -Target "backup-server.fqdn.local" -Username 'Domain\veeam_admin' -Password 'P@ssw0rd' -Format Html,Word -OutputFolderPath 'C:\Users\Jon\Documents' -Timestamp

# Generate a Veeam VBR As Built Report for Backup Server 'backup-server.fqdn.local' using specified credentials and report configuration file. Export report to Text, HTML & DOCX formats. Use default report style. Save reports to 'C:\Users\Jon\Documents'. Display verbose messages to the console.

PS C:\> New-AsBuiltReport -Report Veeam.VBR -Target "backup-server.fqdn.local" -Username 'Domain\veeam_admin' -Password 'P@ssw0rd' -Format Text,Html,Word -OutputFolderPath 'C:\Users\Jon\Documents' -ReportConfigFilePath 'C:\Users\Jon\AsBuiltReport\AsBuiltReport.Veeam.VBR.json' -Verbose

# Generate a Veeam VBR As Built Report for Backup Server 'backup-server.fqdn.local' using stored credentials. Export report to HTML & Text formats. Use default report style. Highlight environment issues within the report. Save reports to 'C:\Users\Jon\Documents'.

PS C:\> $Creds = Get-Credential
PS C:\> New-AsBuiltReport -Report Veeam.VBR -Target "backup-server.fqdn.local" -Credential $Creds -Format Html,Text -OutputFolderPath 'C:\Users\Jon\Documents' -EnableHealthCheck

# Generate a Veeam VBR As Built Report for Backup Server 'backup-server.fqdn.local' using stored credentials. Export report to HTML & DOCX formats. Use default report style. Reports are saved to the user profile folder by default. Attach and send reports via e-mail.

PS C:\> New-AsBuiltReport -Report Veeam.VBR -Target "backup-server.fqdn.local" -Username 'Domain\veeam_admin' -Password 'P@ssw0rd' -Format Html,Word -OutputFolderPath 'C:\Users\Jon\Documents' -SendEmail
```

## Hasta Luego Amigos

![Text](/img/backup_meme.webp#center)
