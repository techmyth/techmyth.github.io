---
title: 'HomeLab: Instalación y utilización de Veeam Diagrammer'
date: '2022-12-22T10:50:00-04:00'
tags:
    - VEEAM
    - Diagramming
---

Hola a todos,

En esta ocasión estaré hablando un poco de un proyecto que he estado trabajando en los últimos meses y que tiene la utilidad de crear diagramas básicos de los componentes de la infraestructura de `Veeam Backup & Replication`. La utilidad se llama `Veeam.Diagrammer` y pueden ver la página del proyecto en GitHub. A continuación les dejo el enlace para que puedan ver las características de esta utilidad.

- <https://github.com/rebelinux/Veeam.Diagrammer>

![Text](/img/github_veeam_diagrammer.webp#center)

Demás está en decir que este proyecto está actualmente en continuo desarrollo por ende puede que se encuentren con errores o bugs 😁.

#### Importante: Este módulo se encuentra publicado en PowerShell Gallery

Ahora, para empezar, hay que cumplir los siguientes requisitos:

- Windows OS (Los módulos de Veeam solo funcionan en Windows)
- PowerShell v5.1+
- Veeam.Backup.PowerShell module >= 1.0

Para validar la versión de Powershell, la variable `$PSVersionTable` puede utilizarse desde una consola PowerShell:

```text
PS C:\Users\jocolon> $PSVersionTable

Name                           Value
----                           -----
PSVersion                      5.1.19041.1682
PSEdition                      Desktop
PSCompatibleVersions           {1.0, 2.0, 3.0, 4.0...}
BuildVersion                   10.0.19041.1682
CLRVersion                     4.0.30319.42000
WSManStackVersion              3.0
PSRemotingProtocolVersion      2.3
SerializationVersion           1.1.0.1

PS C:\Users\jocolon>
```

Para confirmar si los módulos de Veeam necesarios están presentes, se puede utilizar el cmdlet `Get-Module` como se muestra en el siguiente ejemplo:

```text
PS C:\Users\jocolon> Get-Module -ListAvailable -Name @('Veeam.Backup.PowerShell')

    Directory: C:\Program Files\Veeam\Backup and Replication\Console


ModuleType Version    Name                                ExportedCommands
---------- -------    ----                                ----------------
Manifest   1.0        Veeam.Backup.PowerShell             {Get-VBRComputerFileProxyServe....}


PS C:\Users\jocolon> 
```

Si el comando no devuelve ningún resultado, significa que el módulo no está instalado. Es importante señalar que los módulos `Veeam.Backup.PowerShell` están disponibles en el servidor de Veeam Backup o en cualquier dispositivo donde ya esté instalada la consola de gestión. Referencia:

> El equipo remoto desde el que se ejecutan los comandos de Veeam PowerShell debe tener instalada la consola de `Veeam Backup & Replication`. Después de instalar la consola, el módulo de powershell de Veeam se instalará de forma predeterminada.
>
> [Veeam PowerShell Reference](https://helpcenter.veeam.com/docs/backup/powershell/)

Para instalar el `Veeam.Diagrammer` desde la `PowerShell Gallery`, utilice el comando `Install-Module`:

```text
PS C:\Users\jocolon> Install-Module -Name Veeam.Diagrammer                                                                                                                    
PS C:\Users\jocolon> 
```

Para confirmar si se han instalado todas las dependencias, puede utilizar el cmdlet `Get-Module`.

```text
PS C:\Users\jocolon> Get-Module -ListAvailable -Name @('Veeam.Backup.PowerShell','Veeam.Diagrammer')     

    Directory: C:\Users\jocolon\Documents\WindowsPowerShell\Modules


ModuleType Version    Name                                ExportedCommands
---------- -------    ----                                ----------------
Script     0.5.0      Veeam.Diagrammer                    New-VeeamDiagram


    Directory: C:\Program Files\Veeam\Backup and Replication\Console


ModuleType Version    Name                                ExportedCommands
---------- -------    ----                                ----------------
Manifest   1.0        Veeam.Backup.PowerShell             {Get-VBRComputerFileProxyServer, New-VBRSanIntegrationOptions, New-VBRSelectedPersonalFolders, New-VBRUnixScheduleOptions...}

PS C:\Users\jocolon> 
```

Por último, se utiliza el cmdlet `New-VeeamDiagram` para producir el diagrama. Es importante tener en cuenta que es necesario utilizar la dirección IP o el FQDN del servidor de Veeam Backup como `Target`.

#### Guardar Credenciales en una Variable

```text
PS C:\Users\jocolon> $Credential = Get-Credential
PowerShell credential request
Enter your credentials.
User: Veeam_Admin
Password for user Veeam_Admin: `````*
```

#### Command final

```text
PS C:\Users\jocolon> New-VeeamDiagram -Target veeam-vbr.pharmax.local -Username "pharmax\administrator" -Password p@ssw0rd -Format pdf,dot -Direction top-to-bottom -OutputFolderPath C:\Users\jocolon\ -EdgeType polyline -DiagramType Backup-to-Proxy -Verbose -EnableEdgeDebug -EnableSubGraphDebug
VERBOSE: Loading module from path 'C:\Program Files\Veeam\Backup and Replication\Console\Veeam.Backup.PowerShell\..\Veeam.Backup.PowerShell.dll'.
VERBOSE: Trying to import Veeam B&R modules.
VERBOSE: Loading module from path 'C:\Program Files\Veeam\Backup and Replication\Console\Veeam.Backup.PowerShell\..\Veeam.Backup.PowerShell.dll'.
VERBOSE: Identifying Veeam Powershell module version.
VERBOSE: Using Veeam Powershell module version 11.
VERBOSE: Loading module from path 'C:\Program Files\Veeam\Backup and Replication\Console\Veeam.Backup.PowerShell\..\Veeam.Backup.PowerShell.dll'.
VERBOSE: Establishing initial connection to Backup Server: veeam-vbr.pharmax.local.
VERBOSE: Looking for veeam existing server connection.
VERBOSE: Existing veeam server connection found
VERBOSE: Validating connection to veeam-vbr.pharmax.local
VERBOSE: Successfully connected to veeam-vbr.pharmax.local:9392 Backup Server.
VERBOSE: Operation '' complete.
VERBOSE: Collecting Backup Server information from VEEAM-VBR.pharmax.local.
VERBOSE: VEEAM-VBR.pharmax.local
VERBOSE: VEEAM-VBR.pharmax.local
VERBOSE: VEEAM-SQL.pharmax.local
VERBOSE: VEEAM-SQL.pharmax.local
VERBOSE: VEEAM-EM.pharmax.local
VERBOSE: VEEAM-EM.pharmax.local
VERBOSE: Collecting Backup Proxy information from VEEAM-VBR.pharmax.local.
VERBOSE: VEEAM-VBR.pharmax.local
VERBOSE: VEEAM-VBR.pharmax.local
VERBOSE: VEEAM-VBR-02V.pharmax.local
VERBOSE: VEEAM-VBR-02V.pharmax.local
VERBOSE: Collecting Backup Proxy information from VEEAM-VBR.pharmax.local.
VERBOSE: pharmax-cluster.pharmax.local
VERBOSE: pharmax-cluster.pharmax.local
VERBOSE: VEEAM-HV-02
VERBOSE: VEEAM-HV-02
VERBOSE: VEEAM-HV-01
VERBOSE: VEEAM-HV-01
VERBOSE: Collecting Backup Proxy information from VEEAM-VBR.pharmax.local.
VERBOSE: VEEAM-VBR.pharmax.local
VERBOSE: VEEAM-VBR.pharmax.local
VERBOSE: VEEAM-VBR-02V.pharmax.local
VERBOSE: VEEAM-VBR-02V.pharmax.local
VERBOSE: Collecting Backup Proxy information from VEEAM-VBR.pharmax.local.
VERBOSE: pharmax-cluster.pharmax.local
VERBOSE: pharmax-cluster.pharmax.local
VERBOSE: VEEAM-HV-02
VERBOSE: VEEAM-HV-02
VERBOSE: VEEAM-HV-01
VERBOSE: VEEAM-HV-01
Diagram 'Output.pdf' has been saved to 'C:\Users\jocolon\'.
Diagram 'Output.dot' has been saved to 'C:\Users\jocolon\'.
PS C:\Users\jocolon>  
```

He aquí ejemplos del diagrama generado.

#### Diagrama de la infraestructura de Cintas

![Text](/img/Output.webp#center)

#### Diagrama de los Repositorios de Backup

![Text](/img/Output1.webp#center)

#### Diagrama de los repositorios Scale-Out  

![Text](/img/Output2.webp#center)

También incluyo varias opciones sobre cómo construir el diagrama.

```powershell
.SYNOPSIS
    Diagram the configuration of Veeam Backup & Replication infrastructure in PDF/SVG/DOT/PNG formats using PSGraph and Graphviz.
.DESCRIPTION
    Diagram the configuration of Veeam Backup & Replication infrastructure in PDF/SVG/DOT/PNG formats using PSGraph and Graphviz.
.PARAMETER DiagramType
    Specifies the type of veeam vbr diagram that will be generated.
.PARAMETER Target
    Specifies the IP/FQDN of the system to connect.
    Multiple targets may be specified, separated by a comma.
.PARAMETER Port
    Specifies a optional port to connect to Veeam VBR Service.
    By default, port will be set to 9392
.PARAMETER Credential
    Specifies the stored credential of the target system.
.PARAMETER Username
    Specifies the username for the target system.
.PARAMETER Password
    Specifies the password for the target system.
.PARAMETER Format
    Specifies the output format of the diagram.
    The supported output formats are PDF, PNG, DOT & SVG.
    Multiple output formats may be specified, separated by a comma.
.PARAMETER Direction
    Set the direction in which resource are plotted on the visualization
    By default, direction will be set to top-to-bottom.
.PARAMETER NodeSeparation
    Controls Node separation ratio in visualization
    By default, NodeSeparation will be set to .60.
.PARAMETER SectionSeparation
    Controls Section (Subgraph) separation ratio in visualization
    By default, NodeSeparation will be set to .75.
.PARAMETER EdgeType
    Controls how edges lines appear in visualization
    By default, EdgeType will be set to spline.
.PARAMETER OutputFolderPath
    Specifies the folder path to save the diagram.
.PARAMETER Filename
    Specifies a filename for the diagram.
.PARAMETER EnableEdgeDebug
    Control to enable edge debugging ( Dummy Edge and Node lines ).
.PARAMETER EnableSubGraphDebug
    Control to enable subgraph debugging ( Subgraph Lines ).
```

#### Hasta Luego Amigos
