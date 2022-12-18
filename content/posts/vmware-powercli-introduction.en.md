---
title: 'PowerShell: VMware PowerCLI Introduction'
date: '2021-11-04T21:45:52-04:00'
tags:
    - VMware
---

Hi all,

In this post we will see how to install and use the PowerCLI tool. Recently on the VMware Blog portal they conducted a survey on the top 10 most used VMware administrators’ tools. A curious fact is that the PowerCLI tool achieved the first place on the list for the year 2021. It is for this reason that I took the initiative to create a series of introduction articles on how to perform basic tasks using PowerCLI.

VMware describes PowerCLI as:

> VMware PowerCLI is a command-line and scripting tool built on Windows PowerShell, and provides more than 800 cmdlets for managing and automating VMware vSphere, VMware Cloud Director, vRealize Operations Manager, vSAN, VMware NSX-T Data Center, VMware Cloud Services, VMware Cloud on AWS, VMware HCX, VMware Site Recovery Manager, and VMware Horizon environments.
>
> [VMware {Code}](https://developer.vmware.com/web/tool/12.4/vmware-powercli)

As you can see PowerCLI is directly integrated into the VMware ecosystem and is the main tool for automation and development. With this tool you can create simple scripts as well as schedule or automate tasks in large physical and virtual datacenters.

To start using PowerCLI it is required to install the tool, to achieve this goal you can install it from a PowerShell console. In my case, I will be using ‘PowerShell Core’ on linux but it can be installed in the same way on Windows or Macos. To access PowerShell from Linux we use the **“pwsh”** command from a console or **“shell”**.

```text
[rebelinux@xxxxx ~]$ pwsh
PowerShell 7.1.5
Copyright (c) Microsoft Corporation.

https://aka.ms/powershell
Type 'help' to get help.

PS /home/rebelinux>
```

Once you have access to the **“pwsh”** console you can use the **“Install-Module”** command to install the tool. The following example shows the result:

```text
PS /home/rebelinux> Install-Module -Name VMware.PowerCLI                                                                                                                                                                                                                                             Installing package 'VMware.PowerCLI'
Installing dependent package 'VMware.Vim' 
Installing package 'VMware.Vim'
Downloaded 13.78 MB out of 15.25 MB.  
[ooooooooooooooooooooooooooooooooooooooooooooooooooooooooooo]
PS /home/rebelinux>                                                                                                                        
```

To validate which version of PowerCLI you have installed you can use the **“Get-Module”** command:

```text
PS /home/rebelinux> Get-Module -ListAvailable VMware.PowerCLI

    Directory: /home/rebelinux/.local/share/powershell/Modules

ModuleType Version    PreRelease Name                                PSEdition ExportedCommands
---------- -------    ---------- ----                                --------- ----------------
Manifest   12.4.1.18…            VMware.PowerCLI                     Desk      

PS /home/rebelinux> 
```

Once PowerCLI is installed the administration modules can be used. To identify which commands or the so-called PowerShell **cmdlets** are available we can use the **“Get-command”** command.

```text
PS /home/rebelinux> Get-Command -Module VMware* <em>get-vm* </em>    

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

In the specific case of the previous example a filter was applied to the command to show only the cmdlet starting with **“get-vm\*”**. In the next article I will be showing you how to make the initial connection to vCenter or to a standalone ESXi server.

See you soon!

![Text](/img/thats-my-secret-p0jvg2.webp#center)
