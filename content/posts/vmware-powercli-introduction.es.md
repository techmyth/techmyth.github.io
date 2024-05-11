---
title: 'PowerShell: Introducción básica de VMware PowerCLI'
date: '2021-11-04T21:54:15-04:00'
tags:
    - VMware
---

Hola tod@s

En este post veremos cómo utilizar instalar y utilizar la herramienta de PowerCLI. Recientemente en el portal de Blog de VMware realizaron una encuesta sobre las 10 herramienta de los administradores de VMware más utilizada. Un dato curioso es que la herramienta de PowerCLI logró el primer lugar de la lista para este año 2021. Es por esta razón que tome la iniciativa de crear varios artículos introductorios sobre cómo realizar tareas básica utilizando PowerCLI.

VMware describe a PowerCLI como:

> VMware PowerCLI is a command-line and scripting tool built on Windows PowerShell, and provides more than 800 cmdlets for managing and automating VMware vSphere, VMware Cloud Director, vRealize Operations Manager, vSAN, VMware NSX-T Data Center, VMware Cloud Services, VMware Cloud on AWS, VMware HCX, VMware Site Recovery Manager, and VMware Horizon environments.
>
> [VMware {Code}](https://developer.vmware.com/web/tool/12.4/vmware-powercli)

Como pueden ver PowerCLI está directamente integrado en el ecosistema de VMware y es la herramienta principal de automatización y desarrollo. Con esta herramienta se pueden crear `script` sencillos como tambien es util a la hora de programar o automatizar tareas en datacenter físico y virtuales.

Para comenzar a utilizar PowerCLI es necesario instalar la herramienta, para alcanzar este objetivo podemos instalarlo desde una consola de PowerShell. En mi caso, estaré utilizando `PowerShell Core` en linux pero puede ser instalado de igual forma en Windows o Macos. Para acceder PowerShell desde Linux utilizamos el comando `pwsh` desde una consola o `shell`.

```text
[rebelinux@xxxxx ~]$ pwsh
PowerShell 7.1.5
Copyright (c) Microsoft Corporation.

https://aka.ms/powershell
Type 'help' to get help.

PS /home/rebelinux>
```

Una vez tengamos acceso a la consola de `pwsh` podemos utilizar el comando `Install-Module` para instalar la herramienta. En el siguiente ejemplo les muestro el resultado:

```text
PS /home/rebelinux> Install-Module -Name VMware.PowerCLI                                                                                                                                                                                                                                             Installing package 'VMware.PowerCLI'
Installing dependent package 'VMware.Vim' 
Installing package 'VMware.Vim'
Downloaded 13.78 MB out of 15.25 MB.  
[ooooooooooooooooooooooooooooooooooooooooooooooooooooooooooo]
PS /home/rebelinux>                                                                                                                        
```

Para validar que version de PowerCLI tenemos instalado podemos utilizar el comando `Get-Module`:

```text
\PS /home/rebelinux> Get-Module -ListAvailable VMware.PowerCLI

    Directory: /home/rebelinux/.local/share/powershell/Modules

ModuleType Version    PreRelease Name                                PSEdition ExportedCommands
---------- -------    ---------- ----                                --------- ----------------
Manifest   12.4.1.18…            VMware.PowerCLI                     Desk      

PS /home/rebelinux> 
```

Una vez PowerCLI es instalado podemos utilizar los módulos de administración. Para identificar qué comandos o los llamados [cmdlets](https://docs.microsoft.com/en-us/powershell/scripting/developer/cmdlet/cmdlet-overview?view=powershell-7.1) de PowerShell están disponibles podemos utilizar el comando `Get-command`.

```text
PS /home/rebelinux> Get-Command -Module VMware* *get-vm*

CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Cmdlet          Get-VM                                             12.4.0.18… VMware.VimAutomation.Core
Cmdlet          Get-VmcClusterEdrsPolicy                           12.4.0.18… VMware.VimAutomation.Vmc
Cmdlet          Get-VmcOrganization                                12.4.0.18… VMware.VimAutomation.Vmc
Cmdlet          Get-VmcSddc                                        12.4.0.18… VMware.VimAutomation.Vmc
Cmdlet          Get-VmcSddcCluster                                 12.4.0.18… VMware.VimAutomation.Vmc
Cmdlet          Get-VmcSddcNetworkService                          12.4.0.18… VMware.VimAutomation.Vmc
Cmdlet          Get-VmcSddcSiteRecovery                            12.4.0.18… VMware.VimAutomation.Vmc
Cmdlet          Get-VmcSddcSiteRecoveryInstance                    12.4.0.18… VMware.VimAutomation.Vmc
Cmdlet          Get-VmcService                                     12.4.0.18… VMware.VimAutomation.Vmc
Cmdlet          Get-VMGuest                                        12.4.0.18… VMware.VimAutomation.Core
Cmdlet          Get-VMGuestDisk                                    12.4.0.18… VMware.VimAutomation.Core
Cmdlet          Get-VMHost                                         12.4.0.18… VMware.VimAutomation.Core
Cmdlet          Get-VMHostAccount                                  12.4.0.18… VMware.VimAutomation.Core
Cmdlet          Get-VMHostAdvancedConfiguration                    12.4.0.18… VMware.VimAutomation.Core
Cmdlet          Get-VMHostAttributes                               7.0.2.178… VMware.DeployAutomation
Cmdlet          Get-VMHostAuthentication                           12.4.0.18… VMware.VimAutomation.Core
Cmdlet          Get-VMHostAvailableTimeZone                        12.4.0.18… VMware.VimAutomation.Core
Cmdlet          Get-VMHostDiagnosticPartition                      12.4.0.18… VMware.VimAutomation.Core
Cmdlet          Get-VMHostDisk                                     12.4.0.18… VMware.VimAutomation.Core
Cmdlet          Get-VMHostDiskPartition                            12.4.0.18… VMware.VimAutomation.Core
Cmdlet          Get-VMHostFirewallDefaultPolicy                    12.4.0.18… VMware.VimAutomation.Core
Cmdlet          Get-VMHostFirewallException                        12.4.0.18… VMware.VimAutomation.Core
Cmdlet          Get-VMHostFirmware                                 12.4.0.18… VMware.VimAutomation.Core
Cmdlet          Get-VMHostHardware                                 12.4.0.18… VMware.VimAutomation.Core
Cmdlet          Get-VMHostHba                                      12.4.0.18… VMware.VimAutomation.Core
Cmdlet          Get-VMHostImageProfile                             7.0.2.178… VMware.DeployAutomation
Cmdlet          Get-VMHostMatchingRules                            7.0.2.178… VMware.DeployAutomation
Cmdlet          Get-VMHostModule                                   12.4.0.18… VMware.VimAutomation.Core
Cmdlet          Get-VMHostNetwork                                  12.4.0.18… VMware.VimAutomation.Core
Cmdlet          Get-VMHostNetworkAdapter                           12.4.0.18… VMware.VimAutomation.Core
Cmdlet          Get-VMHostNetworkStack                             12.4.0.18… VMware.VimAutomation.Core
Cmdlet          Get-VMHostNtpServer                                12.4.0.18… VMware.VimAutomation.Core
Cmdlet          Get-VMHostPatch                                    12.4.0.18… VMware.VimAutomation.Core
Cmdlet          Get-VMHostPciDevice                                12.4.0.18… VMware.VimAutomation.Core
Cmdlet          Get-VMHostProfile                                  12.4.0.18… VMware.VimAutomation.Core
Cmdlet          Get-VMHostProfileImageCacheConfiguration           12.4.0.18… VMware.VimAutomation.Core
Cmdlet          Get-VMHostProfileRequiredInput                     12.4.0.18… VMware.VimAutomation.Core
Cmdlet          Get-VMHostProfileStorageDeviceConfiguration        12.4.0.18… VMware.VimAutomation.Core
Cmdlet          Get-VMHostProfileUserConfiguration                 12.4.0.18… VMware.VimAutomation.Core
Cmdlet          Get-VMHostProfileVmPortGroupConfiguration          12.4.0.18… VMware.VimAutomation.Core
Cmdlet          Get-VMHostRoute                                    12.4.0.18… VMware.VimAutomation.Core
Cmdlet          Get-VMHostService                                  12.4.0.18… VMware.VimAutomation.Core
Cmdlet          Get-VMHostSnmp                                     12.4.0.18… VMware.VimAutomation.Core
Cmdlet          Get-VMHostStartPolicy                              12.4.0.18… VMware.VimAutomation.Core
Cmdlet          Get-VMHostStorage                                  12.4.0.18… VMware.VimAutomation.Core
Cmdlet          Get-VMHostSysLogServer                             12.4.0.18… VMware.VimAutomation.Core
Cmdlet          Get-VMQuestion                                     12.4.0.18… VMware.VimAutomation.Core
Cmdlet          Get-VMResourceConfiguration                        12.4.0.18… VMware.VimAutomation.Core
Cmdlet          Get-VMStartPolicy                                  12.4.0.18… VMware.VimAutomation.Core

PS /home/rebelinux>
```

En el caso específico del ejemplo anterior se aplicó un filtro al comando para mostrar solo los cmdlet que comienzan con `get-vm *`. En el próximo artículo les estaré mostrando como realizar la conexión inicial hacia vCenter o a un servidor ESXi `standalone`.

Hasta luego!

![Text](/img/thats-my-secret-p0jvg2.webp#center)
