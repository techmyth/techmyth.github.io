---
title: 'HomeLab: Documentación de Veeam VBR utilizando AsBuiltReport'
date: '2021-12-22T10:50:00-04:00'
tags:
    - VEEAM
---

Hola a tod@s!

Como ya bien saben debido al impacto de la pandemia de Covid-19 no he tenido mucho espacio para hacer otra cosa que no sea estar encerrado en mi casa 😒. Más aún cuando tantos de mis familiares han dado positivo al virus. De manera que esta situación ha sido la mejor excusa para ponerme a programar. Recientemente he recibido muchas peticiones de varios usuario y amigos para que desarrolle un reporte que documentación de la aplicación de Backup **“Veeam Backup &amp; Replication”**.

Por esta razón, me di a la tarea de desarrollar otro reporte más, sí otro más 🤣 y está relacionado a documentar las implementaciones de **“Veeam Backup &amp; Replication”** específicamente para la versión **11.** En un futuro evaluaré la posibilidad de añadir soporte a la versiones **pre-11**. Este reporte utiliza como base el **“framework”** del proyecto de AsBuildReport creado por “Tim Carman” [@tpcarman](https://twitter.com/tpcarman) (Aquí el [enlace](http://192.168.7.40/es/2021/09/27/homelab-ontap-documentacion-asbuiltreport/)).

El reporte se encuentra en estado inicial y en constante desarrollo, pero decidí liberarlo públicamente con el objetivo de recibir recomendaciones o más bien para fomentar que otros desarrolladores aporten a mejorar su contenido. El website de desarrollo del reporte se encuentra en Github les dejo el enlace para que puedan ver el alcance y el objetivo del proyecto.

- <https://github.com/AsBuiltReport/AsBuiltReport.Veeam.VBR>

![Text](/img/Veeam_VBR_Portal.webp#center)

**Importante: El reporte se encuentra actualmente disponible en *PowerShell Gallery*.**

Ahora bien, para comenzar necesitamos cumplir con los siguientes requisitos:

- Plataforma de Windows (Los módulos de Veeam solo corren en Windows)
- PowerShell v5.1+
- El módulo de AsBuiltReport.Core >= 1.1.0
- El módulo de Veeam.Backup.PowerShell >= 1.0

Este reporte utiliza la versión de PowerShell 5.+, para validar la versión podemos utilizar la variable **“$PSVersionTable**” desde la consola de PowerShell:

```text
PS C:\Users\jocolon> $PSVersionTable


Name                           Value
----                           -----
PSVersion                      5.1.19041.1320
PSEdition                      Desktop
PSCompatibleVersions           {1.0, 2.0, 3.0, 4.0...}
BuildVersion                   10.0.19041.1320
CLRVersion                     4.0.30319.42000
WSManStackVersion              3.0
PSRemotingProtocolVersion      2.3
SerializationVersion           1.1.0.1


PS C:\Users\jocolon>
```

Para validar si tenemos los módulos requeridos de Veeam podemos utilizar el comando “**Get-Module**” según se muestra en el siguiente ejemplo:

```text
PS C:\Users\jocolon> Get-Module -ListAvailable -Name @('Veeam.Backup.PowerShell')

    Directory: C:\Program Files\Veeam\Backup and Replication\Console


ModuleType Version    Name                                ExportedCommands
---------- -------    ----                                ----------------
Manifest   1.0        Veeam.Backup.PowerShell             {Get-VBRComputerFileProxyServer, New-VBRSanInt...


PS C:\Users\jocolon> 
```

Si el comando no produce algún resultado quiere decir que el módulos no están instalados. Ahora bien para instalar los módulos de **Veeam.Backup.PowerShell** es importante mencionar que estos están disponible en el servidor de Backup de Veeam o en cualquier dispositivo donde la consola de manejo esté instalada. Referencia:

> The remote machine from which you run Veeam PowerShell commands must have the Veeam Backup & Replication Console installed. After you install the Veeam Backup & Replication Console, Veeam PowerShell module will be installed by default
>
> [Veeam PowerShell Reference ](https://helpcenter.veeam.com/docs/backup/powershell/)

Para instalar el reporte ****AsBuiltReport.Veeam.VBR****  desde **PowerShell Gallery** utilizamos el comando tradicional **“Install-Module”**:

```text
PS C:\Users\jocolon> Install-Module -Name AsBuiltReport.Veeam.VBR  
                                                                                                                        
Installing package 'AsBuiltReport.Veeam.VBR' 
[Downloaded 16.73 MB out of 23.76 MB.]

PS C:\Users\jocolon> 
```

Para confirmar si se han instalado todas las dependencias puede utilizar el cmdlet **“Get-Module -ListAvailable -Name @('Veeam.Backup.PowerShell','AsBuiltReport.Veeam.VBR','AsBuiltReport.Core')”**.

```text
PS C:\Users\jocolon> Get-Module -ListAvailable -Name @('Veeam.Backup.PowerShell','AsBuiltReport.Veeam.VBR','AsBuiltReport.Core')


    Directory: C:\Users\jocolon\Documents\WindowsPowerShell\Modules


ModuleType Version    Name                                ExportedCommands
---------- -------    ----                                ----------------
Script     0.7.2      AsBuiltReport.Veeam.VBR             Invoke-AsBuiltReport.Veeam.VBR


    Directory: C:\Program Files\WindowsPowerShell\Modules


ModuleType Version    Name                                ExportedCommands
---------- -------    ----                                ----------------
Script     1.3.0      AsBuiltReport.Core                  {New-AsBuiltReport, New-AsBuiltConfig, New-AsBuiltReportConfig}


    Directory: C:\Program Files\Veeam\Backup and Replication\Console


ModuleType Version    Name                                ExportedCommands
---------- -------    ----                                ----------------
Manifest   1.0        Veeam.Backup.PowerShell             {Add-VBRBackupCopyJob, Get-VBRBackupCopyJob, Set-VBRBackupCopyJob, Copy-VBRBackupCopyJob...}


PS C:\Users\jocolon> 
```

**Si por alguna razón no puedes instalar el informe a través de la PowerShellGallery te dejo aquí el método de instalación manual.**

Un requisito opcional es generar los archivos de configuración que te permite establecer los parámetros de la organización que son utilizados para generar el reporte. Este proceso genera unos archivos tipo JSON que son utilizados como plantillas **“templates”** de forma que no tengas que llenar la información repetitiva cuando generes los reportes.

#### ****Archivos de configuración**** (****AsBuiltReport** JSON**)

El “cmdlet” de powershell **New-AsBuiltConfig** te permite generar la plantilla que utilizaremos como base del reporte. Esta plantilla establece los parámetros no técnicos del reporte.

```text
PS C:\Users\jocolon>  New-AsBuiltConfig

---------------------------------------------
 <        As Built Report Information      >
---------------------------------------------
Enter the name of the Author for this As Built Report [jocolon]: Jonathan Colon
```

```text
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

```text
---------------------------------------------
 <            Email Configuration          >
---------------------------------------------
Would you like to enter SMTP configuration? (y/n): n
```

```text
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
                       "Path":  "C:\Users\jocolon\AsBuiltReport"
                   },
    "Report":  {
                   "Author":  "Jonathan Colon"
               }
}
```

El comando **New-AsBuiltReportConfig** permite establecer los parámetros técnico del reporte como el nivel y tipo de información **“verbose level”**.

```text
PS C:\Users\jocolon> New-AsBuiltReportConfig Veeam.VBR -FolderPath C:\Users\jocolon\AsBuiltReport\
```

Una vez culminado el proceso se creará un archivo tipo JSON con el siguiente contenido:

```json
{
    "Report": {
        "Name": "Veeam VBR As Built Report",
        "Version": "1.0",
        "Status": "Released",
        "ShowCoverPageImage": true,
        "ShowTableOfContents": true,
        "ShowHeaderFooter": true,
        "ShowTableCaptions": true
    },
    "Options": {

    },
    "InfoLevel": {
        "_comment_": "Please refer to the AsBuiltReport project contributing guide for information about how to define InfoLevels.",
        "_comment_": "0 = Disabled, 1 = Enabled, 2 = Adv Summary, 3 = Detailed",
        "Infrastructure": {
            "Section": 1,
            "BackupServer": 1,
            "Proxy": 1,
            "Settings": 1,
            "BR": 1,
            "Licenses": 1,
            "SOBR": 1,
            "WANAccel": 1,
            "SureBackup": 1

        },
        "Tape": {
            "Section": 1,
            "Server": 1,
            "Library": 1
        }
    },
    "HealthCheck": {
        "Infrastructure": {
            "Proxy": true,
            "Settings": true,
            "BR": true,
            "SOBR": true,
            "Server": true,
            "Status": true

        },
        "Tape": {
            "Status": true
        }

    }
}
```

Este archivo de configuración se puede utilizar para especificar el nivel de detalle del reporte como también que sesiones del reporte van a ser habilitadas.

Luego podemos generar el reporte utilizando el comando **“*New-AsBuiltReport -Report Veeam.VBR -Target Backup_Server_FQDN*_or_IP”**. Es importante recalcar que es requerido utilizar la dirección del servidor de backup de **“Veeam”** como **“Target”**.

```text
PS C:\Users\jocolon> New-AsBuiltReport -Report Veeam.VBR -Target veeam-vbr.pharmax.local -AsBuiltConfigFilePath AsBuiltReport.json -OutputFolderPath C:\Users\jocolon\AsBuiltReport\ -Credential $cred -Format HTML -ReportConfigFilePath AsBuiltReport.Veeam.VBR.json -EnableHealthCheck -Verbose

VERBOSE: Loading As Built Report configuration from 'C:\Users\jocolon\AsBuiltReport\AsBuiltReport.json'.
VERBOSE: Loading module from path 'C:\Users\jocolon\Documents\WindowsPowerShell\Modules\AsBuiltReport.Veeam.VBR\AsBuiltReport.Veeam.VBR.psm1'.
VERBOSE: Loading AsBuiltReport.Veeam.VBR report configuration file from path 'C:\Users\jocolon\AsBuiltReport\AsBuiltReport.Veeam.VBR.json'.
VERBOSE: Setting report filename to 'Veeam VBR As Built Report'.
VERBOSE: [ 15:04:09:872 ] [ Document ] - Document 'Veeam VBR As Built Report' processing started.
VERBOSE: [ 15:04:10:005 ] [ Document ] - Executing report style script from path 'C:\Users\jocolon\Documents\WindowsPowerShell\Modules\AsBuiltReport.Veeam.VBR\AsBuiltReport.Veeam.VBR.Style.ps1'.
VERBOSE: [ 15:04:10:036 ] [ Document ] - Setting global document options.
VERBOSE: [ 15:04:10:056 ] [ Document ] - Enabling section/heading numbering.
VERBOSE: [ 15:04:10:057 ] [ Document ] - Setting default font(s) to 'Arial'.
VERBOSE: [ 15:04:10:061 ] [ Document ] - Setting page top margin to '25.05'mm.
VERBOSE: [ 15:04:10:067 ] [ Document ] - Setting page right margin to '25.05'mm.
VERBOSE: [ 15:04:10:069 ] [ Document ] - Setting page bottom margin to '25.05'mm.
VERBOSE: [ 15:04:10:070 ] [ Document ] - Setting page left margin to '25.05'mm.
VERBOSE: [ 15:04:10:072 ] [ Document ] - Setting page size to 'A4'.
VERBOSE: [ 15:04:10:074 ] [ Document ] - Setting page orientation to 'Portrait'.
VERBOSE: [ 15:04:10:079 ] [ Document ] - Setting page height to '297'mm.
VERBOSE: [ 15:04:10:081 ] [ Document ] - Setting page width to '210'mm.
VERBOSE: [ 15:04:10:083 ] [ Document ] - Setting document style 'Title'.
VERBOSE: [ 15:04:10:085 ] [ Document ] - Setting document style 'Title2'.
VERBOSE: [ 15:04:10:105 ] [ Document ] - Setting document style 'Title3'.
VERBOSE: [ 15:04:10:110 ] [ Document ] - Setting document style 'Heading1'.
VERBOSE: [ 15:04:10:117 ] [ Document ] - Setting document style 'Heading2'.
VERBOSE: [ 15:04:10:121 ] [ Document ] - Setting document style 'Heading3'.
VERBOSE: [ 15:04:10:126 ] [ Document ] - Setting document style 'Heading4'.
VERBOSE: [ 15:04:10:129 ] [ Document ] - Setting document style 'Heading5'.
VERBOSE: [ 15:04:10:134 ] [ Document ] - Setting document style 'Heading6'.
VERBOSE: [ 15:04:10:136 ] [ Document ] - Setting document style 'Normal'.
VERBOSE: [ 15:04:10:159 ] [ Document ] - Setting document style 'Caption'.
VERBOSE: [ 15:04:10:162 ] [ Document ] - Setting document style 'Header'.
VERBOSE: [ 15:04:10:174 ] [ Document ] - Setting document style 'Footer'.
VERBOSE: [ 15:04:10:176 ] [ Document ] - Setting document style 'TOC'.
VERBOSE: [ 15:04:10:179 ] [ Document ] - Setting document style 'TableDefaultHeading'.
VERBOSE: [ 15:04:10:182 ] [ Document ] - Setting document style 'TableDefaultRow'.
VERBOSE: [ 15:04:10:185 ] [ Document ] - Setting document style 'Critical'.
VERBOSE: [ 15:04:10:192 ] [ Document ] - Setting document style 'Warning'.
VERBOSE: [ 15:04:10:216 ] [ Document ] - Setting document style 'Info'.
VERBOSE: [ 15:04:10:238 ] [ Document ] - Setting document style 'OK'.
VERBOSE: [ 15:04:10:264 ] [ Document ] - Setting table style 'TableDefault'.
VERBOSE: [ 15:04:10:281 ] [ Document ] - Setting table style 'Borderless'.
VERBOSE: [ 15:04:10:285 ] [ Document ] - Processing document header started.
VERBOSE: [ 15:04:10:288 ] [ Document ] - Processing paragraph 'Veeam VBR As Built Report - v1.0'.
VERBOSE: [ 15:04:10:295 ] [ Document ] - Processing document header completed.
VERBOSE: [ 15:04:10:298 ] [ Document ] - Processing document footer started.
VERBOSE: [ 15:04:10:302 ] [ Document ] - Processing paragraph 'Page <!# PageNumber #!>'.
VERBOSE: [ 15:04:10:318 ] [ Document ] - Processing document footer completed.
VERBOSE: [ 15:04:10:320 ] [ Document ] - Processing blank line.
VERBOSE: [ 15:04:10:343 ] [ Document ] - Processing image 'Veeam Logo'.
VERBOSE: [ 15:04:10:394 ] [ Document ] - Processing blank line.
VERBOSE: [ 15:04:10:396 ] [ Document ] - Processing paragraph 'Veeam VBR As Built Report'.
VERBOSE: [ 15:04:10:400 ] [ Document ] - Processing blank line.
VERBOSE: [ 15:04:10:419 ] [ Document ] - Processing paragraph 'Zen Pr Solutions'.
VERBOSE: [ 15:04:10:421 ] [ Document ] - Processing blank line.
VERBOSE: [ 15:04:10:423 ] [ Document ] - Processing table 'Cover Page'.
VERBOSE: [ 15:04:10:449 ] [ Document ] - Processing page break.
VERBOSE: [ 15:04:10:453 ] [ Document ] - Processing table of contents 'Table of Contents'.
VERBOSE: [ 15:04:10:465 ] [ Document ] - Processing page break.
VERBOSE: [ 15:04:10:742 ] [ Document ] - Trying to import Veeam B&R modules.
VERBOSE: [ 15:04:10:993 ] [ Document ] - Identifying Veeam Powershell module version.
VERBOSE: [ 15:04:10:995 ] [ Document ] - Using Veeam Powershell module version 11.
VERBOSE: [ 15:04:11:231 ] [ Document ] - Establishing the initial connection to the Backup Server: veeam-vbr.pharmax.local.
VERBOSE: [ 15:04:11:233 ] [ Document ] - Looking for veeam existing server connection.
VERBOSE: [ 15:04:11:335 ] [ Document ] - Existing veeam server connection found
VERBOSE: [ 15:04:11:337 ] [ Document ] - Validating connection to veeam-vbr.pharmax.local
VERBOSE: [ 15:04:11:460 ] [ Document ] - Successfully connected to veeam-vbr.pharmax.local VBR Server.
VERBOSE: [ 15:04:11:462 ] [ Document ] - Processing section 'Implementation Report' started.
VERBOSE: [ 15:04:11:464 ] [ Document ] - Processing paragraph 'The following section provides a sum[..]'.
VERBOSE: [ 15:04:11:471 ] [ Document ] - Processing blank line.
VERBOSE: [ 15:04:11:486 ] [ Document ] - Backup Infrastructure InfoLevel set at 2.
VERBOSE: [ 15:04:11:488 ] [ Document ] - Processing section 'Backup Infrastructure Summary' started.
VERBOSE: [ 15:04:11:508 ] [ Document ] - Discovering Veeam VBR Infrastructure Summary from veeam-vbr.pharmax.local.
Output Truncated!
VERBOSE: [ 15:06:02:777 ] [ Document ] - Discovered Pharmax - Veeam Tape Vault Type Vault.
VERBOSE: [ 15:06:02:979 ] [ Document ] - Processing table 'Tape Vault - VEEAM-VBR'.
VERBOSE: [ 15:06:02:986 ] [ Document ] - Processing section 'Tape Vault Summary' completed.
VERBOSE: [ 15:06:02:989 ] [ Document ] - Processing section 'Tape Infrastructure Summary' completed.
VERBOSE: [ 15:06:02:991 ] [ Document ] - Processing section 'Implementation Report' completed.
VERBOSE: [ 15:06:03:130 ] [ Document ] - Document 'Veeam VBR As Built Report' processing completed.
VERBOSE: [ 15:06:03:141 ] [ Document ] - Total processing time '1.89' minutes.
Veeam VBR As Built Report 'Veeam VBR As Built Report' has been saved to 'C:\Users\jocolon\AsBuiltReport\'.
PS C:\Users\jocolon> 
```

Aquí les dejo el ejemplo del reporte generado.

{{< embed-pdf url="./img/Veeam-VBR-As-Built-Report.pdf" >}}

Adicionalmente les incluyo varias opciones de cómo invocar el reporte.

```text
# Generate a Veeam VBR As Built Report for Backup Server '192.168.7.60' using specified credentials. Export report to HTML & DOCX formats. Use default report style. Append timestamp to report filename. Save reports to 'C:\Users\Jon\Documents'

PS C:\> New-AsBuiltReport -Report Veeam.VBR -Target 192.168.7.60 -Username 'Domain\veeam_admin' -Password 'P@ssw0rd' -Format Html,Word -OutputFolderPath 'C:\Users\Jon\Documents' -Timestamp

# Generate a Veeam VBR As Built Report for Backup Server 192.168.7.60 using specified credentials and report configuration file. Export report to Text, HTML & DOCX formats. Use default report style. Save reports to 'C:\Users\Jon\Documents'. Display verbose messages to the console.

PS C:\> New-AsBuiltReport -Report Veeam.VBR -Target 192.168.7.60 -Username 'Domain\veeam_admin' -Password 'P@ssw0rd' -Format Text,Html,Word -OutputFolderPath 'C:\Users\Jon\Documents' -ReportConfigFilePath 'C:\Users\Jon\AsBuiltReport\AsBuiltReport.Veeam.VBR.json' -Verbose

# Generate a Veeam VBR As Built Report for Backup Server 192.168.7.60 using stored credentials. Export report to HTML & Text formats. Use default report style. Highlight environment issues within the report. Save reports to 'C:\Users\Jon\Documents'.

PS C:\> $Creds = Get-Credential
PS C:\> New-AsBuiltReport -Report Veeam.VBR -Target 192.168.7.60 -Credential $Creds -Format Html,Text -OutputFolderPath 'C:\Users\Jon\Documents' -EnableHealthCheck

# Generate a Veeam VBR As Built Report for Backup Server 192.168.7.60 using stored credentials. Export report to HTML & DOCX formats. Use default report style. Reports are saved to the user profile folder by default. Attach and send reports via e-mail.

PS C:\> New-AsBuiltReport -Report Veeam.VBR -Target 192.168.7.60 -Username 'Domain\veeam_admin' -Password 'P@ssw0rd' -Format Html,Word -OutputFolderPath 'C:\Users\Jon\Documents' -SendEmail
```

## Hasta Luego Amigos!

![Text](/img/backup_meme.webp#center)
