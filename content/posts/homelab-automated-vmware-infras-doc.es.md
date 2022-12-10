---
title: 'HomeLab – Documentación Automatizada de Infraestructuras basadas en VMware'
date: '2021-06-06T16:17:25-04:00'
author: 'Jonathan Colon Feliciano'
tags:
    - VMware
---

En este blog estaré hablando sobre como automatizar la creación de reportes de documentación de nuestra infraestructura virtual. Existen varias soluciones comerciales para generar este tipo de reporte pero estaré hablando de [**“As Built Report”**](https://www.asbuiltreport.com/) una herramienta gratuita que utiliza powershell como base para generar los reportes.

La herramienta «As Built Report» utiliza los módulos de VMware.PowerCLI que explicamos anteriormente en nuestro blog. Si de sea saber un poco más sobre PowerCLI sigue este enlace [**here**](http://192.168.7.40/2021/06/05/how-to-install-and-use-powercli-on-archlinux/). Un dato importante sobre **«As Built Report»** es que no solo se utiliza para generar reporte sobre VMware también cuenta con soporte para los siguientes productos:

- VMware vSphere, NSX & SRM
- Cisco UCS Manager
- Nutanix Prism Element
- Pure Storage FlashArray
- Rubrik
- Zerto
- Dell/EMC VxRail
- Cohesity DataPlatform
- etc…

Primero que todo para utilizar esta herramienta necesitamos validar los requisitorios que en general son:

- Windows PowerShell 5.1 o later
- VMware.PowerCLI

Para instalar el modulo de powershell de **«As Built Report»** utilizamos el comando **Install-Module** seguido del nombre del modulo AsBuiltReport.

```powershell
PS /home/blabla> Install-Module -Name AsBuiltReport

Untrusted repository
You are installing the modules from an untrusted repository. If you trust this repository, change its InstallationPolicy value by running the Set-PSRepository cmdlet. Are you sure you want to install the modules from 'PSGallery'?
[Y] Yes  [A] Yes to All  [N] No  [L] No to All  [S] Suspend  [?] Help (default is "N"): A
PS /home/blabla>   
```

Un requisito opcional es generar los archivo de configuración que te permite establecer los parámetros de la organización que son utilizados para generar el reporte. Este proceso genera unos archivos tipo JSON que son utilizados como plantillas **«templates»** de forma que no tengas que llenar la información repetitiva cuando generes los reportes.

#### AsBuiltReport archivo de configuratiom tipo JSON

El **«cmdlet»** de powershell New-AsBuiltConfig te permite generar la plantilla que utilizaremos como base del reporte. Esta plantilla establece los parámetros no técnicos del reporte.

```powershell
PS C:\WINDOWS\system32> New-AsBuiltConfig

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
Enter a name for the As Built Report configuration file [AsBuiltReport]: HomeLab VMware Report
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

```batch
PS C:\WINDOWS\system32> New-AsBuiltReportConfig VMware.vSphere -FolderPath C:\Users\jocolon\AsBuiltReport\ -Filename ReportConfig

```

Una vez culminado el proceso se creará un archivo tipo JSON con el siguiente contenido:

```json
{
    "Report": {
        "Name": "VMware vSphere As Built Report",
        "Version": "1.0",
        "Status": "Released"
    },
    "Options": {
        "ShowLicenseKeys": false,
        "ShowVMSnapshots": true
    },
    "InfoLevel": {
        "_comment_": "0 = Disabled, 1 = Summary, 2 = Informative, 3 = Detailed, 4 = Adv Detailed, 5 = Comprehensive",
        "vCenter": 3,
        "Cluster": 3,
        "ResourcePool": 3,
        "VMHost": 3,
        "Network": 3,
        "vSAN": 3,
        "Datastore": 3,
        "DSCluster": 3,
        "VM": 2,
        "VUM": 3
    },
    "HealthCheck": {
        "vCenter": {
            "Mail": true,
            "Licensing": true
        },
        "Cluster": {
            "HAEnabled": true,
            "HAAdmissionControl": true,
            "HostFailureResponse": true,
            "HostMonitoring": true,
            "DatastoreOnPDL": true,
            "DatastoreOnAPD": true,
            "APDTimeOut": true,
            "vmMonitoring": true,
            "DRSEnabled": true,
            "DRSAutomationLevelFullyAuto": true,
            "PredictiveDRS": false,
            "DRSVMHostRules": true,
            "DRSRules": true,
            "vSANEnabled": false,
            "EVCEnabled": true,
            "VUMCompliance": true
        },
        "VMHost": {
            "ConnectionState": true,
            "HyperThreading": true,
            "ScratchLocation": true,
            "IPv6": true,
            "UpTimeDays": true,
            "Licensing": true,
            "SSH": true,
            "ESXiShell": true,
            "NTP": true,
            "StorageAdapter": true,
            "NetworkAdapter": true,
            "LockdownMode": true,
            "VUMCompliance": true
        },
        "vSAN": {},
        "Datastore": {
            "CapacityUtilization": true
        },
        "DSCluster": {
            "CapacityUtilization": true,
            "SDRSAutomationLevelFullyAuto": true
        },
        "VM": {
            "PowerState": true,
            "ConnectionState": true,
            "CpuHotAdd": true,
            "CpuHotRemove": true,
            "MemoryHotAdd": true,
            "ChangeBlockTracking": true,
            "SpbmPolicyCompliance": true,
            "VMToolsStatus": true,
            "VMSnapshots": true
        }
    }
}
```

Por ultimo, generamos el reporte utilizando el comando **New-AsBuiltReport** con los parámetros de información del vCenter y haciendo referencia a los archivo JSON que generamos como plantillas.

```powershell
PS C:\WINDOWS\system32> New-AsBuiltReport -Report VMware.vSphere -Target vcenter-01v.zenprsolutions.local -Username administrator@vsphere.local -Password XXXXX -Format Word,Text,HTML -OutputFolderPath 'C:\Users\jocolon\OneDrive\Desktop\' -EnableHealthCheck -AsBuiltConfigFilePath 'HomeLab VMware Report.json' -ReportConfigFilePath 'ReportConfig.json'

VMware vSphere As Built Report 'VMware vSphere As Built Report' has been saved to 'C:\Users\jocolon\OneDrive\Desktop\'.
PS C:\WINDOWS\system32>
```

Una vez termine el proceso de recopilar la información desde el vCenter el comando graba el reporte según se le especificó con el parámetro de **OutputFolderPath**. En la siguiente imagen se muestra el reporte generado en el formato de **Word,Text,HTML** según se le especificó.

![Text](/img/2021-06-06_13-57.webp#center)

A continuación les muestro varias imágenes que muestran el resultado del reporte generado para el vCenter **vcenter-01v**:

![Text](/img/asbuiltreport-vsphere.webp#center)

### Resumen

En este laboratorio vimos que fácil es generar documentación sobre nuestra infraestructura virtual utilizando herramientas disponibles gratuitamente. «As Built Report» es una herramienta robusta que nos facilita el proceso manual de crear o mantener actualizada nuestra documentación.
