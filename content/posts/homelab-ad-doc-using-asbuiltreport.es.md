---
title: 'HomeLab: Documentación Automatizada de Active Directory utilizando AsBuiltReport'
date: '2021-10-14T07:22:42-04:00'
tags:
    - HomeLab
    - Active Directory
---

Hola a tod@s!

Anteriormente les hable sobre el reporte de NetApp.Ontap que ayude a desarrollar a través del proyecto de AsBuildReport creado por «Tim Carman» [@tpcarman](https://twitter.com/tpcarman) (Aquí el [enlace](http://192.168.7.40/2021/09/27/homelab-ontap-docs-asbuiltreport/)).Pues esta vez les estaré hablando de otro reporte que he estado trabajando durante estos últimos meses relacionado a generar documentación sobre la infraestructura de «Active Directory» (AD). Durante muchos años he realizado varias implementaciones y/o consultorías relacionadas a este servicio y siempre he tenido la intención de crear un reporte condensado de cómo está diseñada esta infraestructura de servicios que para efectos estadísticos corre en más del 95% de las empresas mundiales.

Aunque tengo que reconocer que existen un sin número de expertos que han creado varios reportes similares, un dato único de este reporte es que utiliza [AsBuildReport](https://www.asbuiltreport.com/) como base para generar la programación necesaria para establecer la estructura del reporte. De forma que yo como programador me puedo concentrar en la tecnología que deseo documentar y no en añadir código para generar el formato en Word o HTML.

El reporte se encuentra en estado inicial y en constante desarrollo, pero decidí liberarlo públicamente para poder recibir recomendaciones o más bien para fomentar que otros desarrolladores aporte a mejorar su contenido. El website de desarrollo del reporte se encuentra en Github les dejo el enlace para que puedan ver el alcance y el objetivo del proyecto.

- <https://github.com/AsBuiltReport/AsBuiltReport.Microsoft.AD>

![Text](/img/AsbuildReport_AD.webp)

Este reporte solo puede ser ejecutado en Windows 10+ o Windows Server 2012+. Adicionalmente es requerida la versión de PowerShell 5.1 o PowerShell 7+ y los siguientes módulos son requeridos para poder generar el reporte:

- [AsBuiltReport.Microsoft.AD](https://www.powershellgallery.com/packages/AsBuiltReport.Microsoft.AD/0.3.0)
- [ActiveDirectory](https://docs.microsoft.com/en-us/powershell/module/activedirectory/?view=windowsserver2022-ps)
- [PSPKI](https://www.powershellgallery.com/packages/PSPKI/3.7.2)
- [GroupPolicy](https://docs.microsoft.com/en-us/powershell/module/grouppolicy/?view=windowsserver2019-ps)

Este reporte utiliza la versión de PowerShell 5.+ ó PSCore 7, para validar la versión podemos utilizar la variable **«$PSVersionTable»** desde la consola de PowerShell:

```text
PS C:\Users\jocolon> $PSVersionTable

Name                           Value
----                           -----
PSVersion                      5.1.19041.1237
PSEdition                      Desktop
PSCompatibleVersions           {1.0, 2.0, 3.0, 4.0...}
BuildVersion                   10.0.19041.1237
CLRVersion                     4.0.30319.42000
WSManStackVersion              3.0
PSRemotingProtocolVersion      2.3
SerializationVersion           1.1.0.1


PS C:\Users\jocolon>
```

Para validar si tenemos los módulos requeridos podemos utilizar el comando **«Get-Module»** según se muestra en el siguiente ejemplo:

```text
PS C:\Users\jocolon> Get-Module -ListAvailable -Name @('ActiveDirectory','GroupPolicy','PSPKI')

    Directory: C:\Program Files\WindowsPowerShell\Modules

ModuleType Version    Name                                ExportedCommands
---------- -------    ----                                ----------------
Script     3.7.2      PSPKI                               {Set-CertificateExtension, Get-CertificationAuthorityDbSch...

    Directory: C:\WINDOWS\system32\WindowsPowerShell\v1.0\Modules

ModuleType Version    Name                                ExportedCommands
---------- -------    ----                                ----------------
Manifest   1.0.1.0    ActiveDirectory                     {Add-ADCentralAccessPolicyMember...
Manifest   1.0.0.0    GroupPolicy                         {Backup-GPO, Block-GPInheritance...

PS C:\Users\jocolon>
```

Si el comando no produce algún resultado quiere decir que ninguno de los módulos está instalado. Para instalar estas dependencias utilizamos el comando **«Install-Module»**:

```text
Installation of the PSPKI module:
PS C:\WINDOWS\system32> Install-Module -Name PSPKI                                                                                                                                                                                                                                                                                Installing package 'PSPKI'                                                                                                                                                                                                                                                        Copying unzipped package to 'C:\Users\jocolon\AppData\Local\Temp\2052046370\PSPKI'                                                                                                                                                                                             [oooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooo]    
PS C:\WINDOWS\system32>                                                                                                                                                                                                                                                                                 
```

Para instalar los módulos de **«ActiveDirectory»** y **«GroupPolicy»** utilizamos el comando **«Add-WindowsCapability»** para Windows 10 o el comando **«Install-WindowsFeature»** para Windows Server 2012+

```text
Installation of ActiveDirectory and GroupPolicy module:
PS C:\WINDOWS\system32> Add-WindowsCapability -online -Name 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1 0'                                                                                                                                                                                                                                                                                                                                                                                                                                                 
Path          :
Online        : True
RestartNeeded : False

PS C:\WINDOWS\system32> Add-WindowsCapability -online -Name 'Rsat.GroupPolicy.Management.Tools~~~~0.0.1.0'

Path          :
Online        : True
RestartNeeded : False

PS C:\WINDOWS\system32>

```

Una vez instalamos los prerrequisito podemos continuar con la instalación del módulo principal **«AsBuiltReport.Microsoft.AD«**.

```text
PS C:\WINDOWS\system32> Install-Module -Name AsBuiltReport.Microsoft.AD

Untrusted repository
You are installing the modules from an untrusted repository. If you trust this repository, change its InstallationPolicy value by running the Set-PSRepository cmdlet. Are you sure you want to install the modules from 'PSGallery'?
[Y] Yes  [A] Yes to All  [N] No  [L] No to All  [S] Suspend  [?] Help (default is "N"): Y

    Directory: C:\Program Files\WindowsPowerShell\Modules

PS C:\WINDOWS\system32>
```

Para validar que el módulo fue instalado correctamente podemos utilizar el comando ***«Get-Module«***.

```text
PS C:\WINDOWS\system32> Get-Module -ListAvailable -Name AsBuiltReport.Microsoft.AD


    Directory: C:\Users\jocolon\Documents\WindowsPowerShell\Modules


ModuleType Version    Name                                ExportedCommands
---------- -------    ----                                ----------------
Script     0.4.0      AsBuiltReport.Microsoft.AD          Invoke-AsBuiltReport.Microsoft.AD


PS C:\WINDOWS\system32>
```

#### Nota: Como se puede ver se realizó la instalación del módulo versión «0.4.0«

Un requisito opcional es generar los archivos de configuración que te permite establecer los parámetros de la organización que son utilizados para generar el reporte. Este proceso genera unos archivos tipo JSON que son utilizados como plantillas “templates” de forma que no tengas que llenar la información repetitiva cuando generes los reportes.

#### **Archivos de configuración (AsBuiltReport JSON)**

El «cmdlet» de powershell **New-AsBuiltConfig** te permite generar la plantilla que utilizaremos como base del reporte. Esta plantilla establece los parámetros no técnicos del reporte.

```text
PS C:\WINDOWS\system32> New-AsBuiltConfig

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
Enter a name for the As Built Report configuration file [AsBuiltReport]: HomeLab ONTAP Report
Enter the path to save the As Built Report configuration file [C:\Users\jocolon\AsBuiltReport]:

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
PS C:\WINDOWS\system32> New-AsBuiltReportConfig Microsoft.AD -FolderPath C:\Users\jocolon\AsBuiltReport\
```

Una vez culminado el proceso se creará un archivo tipo JSON con el siguiente contenido:

```json
{
    "Report": {
        "Name": "Microsoft AD As Built Report",
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
        "_comment_": "0 = Disabled, 1 = Enabled, 2 = Adv Summary, 3 = Detailed",
        "Forest": 1,
        "Domain": 1,
        "DHCP": 1,
        "DNS": 1,
        "CA": 0,
        "Security": 0
    },
    "HealthCheck": {
        "Domain": {
            "GMSA": true,
            "GPO": true
        },
        "DomainController": {
            "Diagnostic": true,
            "Services": true
        },
        "Site": {
            "Replication": true
        },
        "DNS": {
            "Aging": true
        },
        "DHCP": {
            "Summary": true,
            "Credential": true,
            "Statistics": true,
            "BP": true
        }
    }
}
```

Este archivo de configuración se puede utilizar para especificar el nivel de detalle del reporte como también que sesiones del reporte van a ser habilitadas.

Luego podemos generar el reporte utilizando el comando **«New-AsBuiltReport -Report Microsoft.AD -Target DC_FQDN«**. Es importante recalcar que es requerido que la computadora donde se genere el reporte esté añadida al dominio de AD que se quiere documentar. También es requerido utilizar el **«fully qualified domain name (FQDN)«** del servidor con el rol de **«Domain Controller»** que esté dentro del «Forest» de AD.

```text
PS C:\Users\jocolon>  New-AsBuiltReport -Report Microsoft.AD -Target server-dc-01v.zenpr.local -Format HTML -AsBuiltConfigFilePath C:\Users\jocolon\AsBuiltReport\AsBuiltReport.json -OutputFolderPath C:\Users\jocolon\AsBuiltReport\ -ReportConfigFilePath '.\AsBuiltReport\AsBuiltReport.Microsoft.AD.json' -Credential $cred -EnableHealthCheck               

Please wait while the Microsoft AD As Built Report is being generated.
WARNING: [ 20:03:36:175 ] [ Document ] - List table captions are only supported on tables with a single row. Removing caption from table 'Group Managed Service Accounts Information - ZENPR.LOCAL'.
WARNING: [ 20:03:52:062 ] [ Document ] - List table captions are only supported on tables with a single row. Removing caption from table 'AD Domain Controller Hardware Information - ZENPR.LOCAL'.
WARNING: [ 20:04:12:144 ] [ Document ] - List table captions are only supported on tables with a single row. Removing caption from table 'Site Replication Information - ZENPR.LOCAL'.
WARNING: [ 20:04:12:412 ] [ Document ] - List table captions are only supported on tables with a single row. Removing caption from table 'Site Replication Failure Information - ZENPR.LOCAL'.
WARNING: [ 20:05:53:997 ] [ Document ] - Error: Collecting Active Directory Group Policy Objects with Startup/Shutdown Script for domain ACAD.ZENPR.LOCAL.
WARNING: [ 20:06:21:709 ] [ Document ] - Cannot bind argument to parameter 'Rows' because it is an empty collection.
WARNING: [ 20:06:21:853 ] [ Document ] - List table captions are only supported on tables with a single row. Removing caption from table 'IPv4 Scope Failover Cofiguration Information - SERVER-DC-01V'.
WARNING: [ 20:06:22:399 ] [ Document ] - Cannot bind argument to parameter 'Rows' because it is an empty collection.
WARNING: [ 20:06:22:440 ] [ Document ] - Cannot bind argument to parameter 'Rows' because it is an empty collection.
WARNING: [ 20:06:22:525 ] [ Document ] - Cannot bind argument to parameter 'Rows' because it is an empty collection.
WARNING: [ 20:06:22:555 ] [ Document ] - Cannot bind argument to parameter 'Rows' because it is an empty collection.
WARNING: [ 20:06:22:589 ] [ Document ] - Cannot bind argument to parameter 'Rows' because it is an empty collection.
WARNING: [ 20:06:22:897 ] [ Document ] - Error: Retreiving DHCP Server IPv4 Scope Failover Setting from ACADE-DC-01V.
Microsoft AD As Built Report 'Microsoft AD As Built Report' has been saved to 'C:\Users\jocolon\AsBuiltReport\'.
PS C:\Users\jocolon>
```

Aquí les dejo el ejemplo de la documentación generada.

{{< embed-pdf url="./img/Sample-Microsoft-AD-As-Built-Report.pdf" >}}

Adicionalmente les incluyo varios ejemplos de cómo invocar el reporte.

```text
# Generate a Microsoft Active Directory As Built Report for Domain Controller Server 'admin-dc-01v.contoso.local' using specified credentials. Export report to HTML & DOCX formats. Use default report style. Append timestamp to report filename. Save reports to 'C:\Users\Jon\Documents'

PS C:\> New-AsBuiltReport -Report Microsoft.AD -Target 'admin-dc-01v.contoso.local' -Username 'administrator@contoso.local' -Password 'P@ssw0rd' -Format Html,Word -OutputFolderPath 'C:\Users\Jon\Documents' -Timestamp

# Generate a Microsoft Active Directory As Built Report for Domain Controller Server 'admin-dc-01v.contoso.local' using specified credentials and report configuration file. Export report to Text, HTML & DOCX formats. Use default report style. Save reports to 'C:\Users\Jon\Documents'. Display verbose messages to the console.

PS C:\> New-AsBuiltReport -Report Microsoft.AD -Target 'admin-dc-01v.contoso.local' -Username 'administrator@contoso.local' -Password 'P@ssw0rd' -Format Text,Html,Word -OutputFolderPath 'C:\Users\Jon\Documents' -ReportConfigFilePath 'C:\Users\Jon\AsBuiltReport\AsBuiltReport.Microsoft.AD.json' -Verbose

# Generate a Microsoft Active Directory As Built Report for Domain Controller Server 'admin-dc-01v.contoso.local' using stored credentials. Export report to HTML & Text formats. Use default report style. Highlight environment issues within the report. Save reports to 'C:\Users\Jon\Documents'.

PS C:\> $Creds = Get-Credential
PS C:\> New-AsBuiltReport -Report Microsoft.AD -Target 'admin-dc-01v.contoso.local' -Credential $Creds -Format Html,Text -OutputFolderPath 'C:\Users\Jon\Documents' -EnableHealthCheck

# Generate a Microsoft Active Directory As Built Report for Domain Controller Server 'admin-dc-01v.contoso.local' using specified credentials. Export report to HTML & DOCX formats. Use default report style. Reports are saved to the user profile folder by default. Attach and send reports via e-mail.

PS C:\> New-AsBuiltReport -Report Microsoft.AD -Target 'admin-dc-01v.contoso.local' -Username 'administrator@contoso.local' -Password 'P@ssw0rd' -Format Html,Word -OutputFolderPath 'C:\Users\Jon\Documents' -SendEmail
```

### Hasta Luego Amigos!

![Text](/img/ad_meme.webp#center)
