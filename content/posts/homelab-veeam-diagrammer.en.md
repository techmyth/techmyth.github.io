---
title: 'HomeLab: Veeam Diagrammer installation and setup'
date: '2022-12-22T10:50:00-04:00'
tags:
    - VEEAM
    - Diagramming
---

Hello everyone,

This time I'll be talking a bit about a project I've been working on for the last few months that has the use of creating basic diagrams of the components of the Veeam Backup & Replication infrastructure. The tool is called Veeam.Diagrammer and you can see the project page on Github. Here is the link so you can see the features of this tool.

- <https://github.com/rebelinux/Veeam.Diagrammer>

![Text](/img/github_veeam_diagrammer.webp#center)

Moreover, this project is still under continuous development, so you may encounter errors or bugs 😁.

`Important: This module is published in PowerShell Gallery.

Now, to get started, the following requirements must be met:

- Windows OS (Veeam modules work only on Windows)
- PowerShell v5.1+
- Veeam.Backup.PowerShell module >= 1.0

To validate the Powershell version, the `$PSVersionTable` variable can be used from a PowerShell console:

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

To confirm if the required Veeam modules are present, you can use the `Get-Module` cmdlet as shown in the following example:

```text
PS C:\Users\jocolon> Get-Module -ListAvailable -Name @('Veeam.Backup.PowerShell')

    Directory: C:\Program Files\Veeam\Backup and Replication\Console


ModuleType Version    Name                                ExportedCommands
---------- -------    ----                                ----------------
Manifest   1.0        Veeam.Backup.PowerShell             {Get-VBRComputerFileProxyServe....}


PS C:\Users\jocolon> 
```

If the command does not return any results, it means that the module is not installed. It is important to note that Veeam.Backup.PowerShell modules are available on the Veeam Backup server or on any device where the management console is already installed. Reference:

> The remote computer from which Veeam PowerShell commands are executed must have the Veeam Backup & Replication console installed. After installing the console, the Veeam powershell module will be installed by default.
>
> [Veeam PowerShell Reference](https://helpcenter.veeam.com/docs/backup/powershell/)

To install the `Veeam.Diagrammer` from the `PowerShell Gallery` use the `Install-Module` command:

```text
PS C:\Users\jocolon> Install-Module -Name Veeam.Diagrammer                                                                                                                    
PS C:\Users\jocolon> 
```

To confirm if all dependencies are installed, you can use the "Get-Module" cmdlet.

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

Finally, the `New-VeeamDiagram` cmdlet is used to build the diagram. It is important to note that you need to use the IP address or the FQDN of the Veeam Backup Server as `Target`.

#### Store Credentials in a Variable

```text
PS C:\Users\jocolon> $Credential = Get-Credential
PowerShell credential request
Enter your credentials.
User: Veeam_Admin
Password for user Veeam_Admin: `````*
```

#### New-VeeamDiagram command

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

Here are several examples of the generated diagram.

#### Tape Infrastructure Diagram

![Text](/img/Output.webp#center)

#### Backup Repository Diagram

![Text](/img/Output1.webp#center)

#### Scale-Out Backup Repository Diagram

![Text](/img/Output2.webp#center)

I also include several options on how to construct the diagram.

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
