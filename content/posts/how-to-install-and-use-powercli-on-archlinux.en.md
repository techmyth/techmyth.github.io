---
title: 'How To Install and Use VMware PowerCLI on ArchLinux'
date: '2021-06-05T19:27:18-04:00'
author: 'Jonathan Colon Feliciano'
tags:
    - VMware
---

In this blog I will be showing you how to install the PowerCLI tool specifically on the ArchLinux operating system. Archlinux is an advanced Linux distribution that is characterized by being simple and lightweight. Additionally, it offers the user full control in managing and modifying everything related to the system.

Well, to install and use PowerCLI we have to install Powershell first. PowerShell is a cross-platform automation and configuration tool. PowerShell has a large number of commands oriented to system administration. But at the same time, PowerShell is a full-featured programming language that allows writing functional programs. There are many Powershell-based administration tools from different manufacturers such as:

- VMware PowerCLI
- Cisco UCS PowerTool
- NetApp PowerShell Toolkit
- DELL\EMC Unity-Powershell
- Amazon AWS Tools for PowerShell

As you can see Powershell is a tool highly used by infrastructure manufacturers and is offered as a method for automation or rapid deployment of software-based infrastructure. Powershell is my second preferred programming language with Python at the top of my list.

The first thing we have to do is to install Powershell and for this I will use the program `yay` which is a tool in Archlinux to install programs from the unofficial repository `Arch User Repository`. With the command `yay -S powershell-bin` we can install the Powershell program to the system.

#### yay -S powershell-bin

```bash
[rebelinux@blabla ~]$ yay -S powershell-bin
:: Checking for conflicts...
:: Checking for inner conflicts...
[Aur:1]  powershell-bin-7.1.3-1

:: Downloaded PKGBUILD (1/1): powershell-bin
  1 powershell-bin                           (Installed) (Build Files Exist)
==> Diffs to show?
==> [N]one [A]ll [Ab]ort [I]nstalled [No]tInstalled or (1 2 3, 1-3, ^4)
==> 
:: (1/1) Parsing SRCINFO: powershell-bin
==> Making package: powershell-bin 7.1.3-1 (Sat 05 Jun 2021 08:08:03 PM AST)
==> Retrieving sources...
  -> Downloading powershell_7.1.3-1.ubuntu.18.04_amd64.deb...
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   647  100   647    0     0   1473      0 --:--:-- --:--:-- --:--:--  1473
100 65.1M  100 65.1M    0     0  1182k      0  0:00:56  0:00:56 --:--:-- 1201k
==> Validating source files with sha256sums...
    powershell_7.1.3-1.ubuntu.18.04_amd64.deb ... Passed
==> Making package: powershell-bin 7.1.3-1 (Sat 05 Jun 2021 08:09:00 PM AST)
==> Checking runtime dependencies...
==> Checking buildtime dependencies...
==> Retrieving sources...
  -> Found powershell_7.1.3-1.ubuntu.18.04_amd64.deb
==> Validating source files with sha256sums...
    powershell_7.1.3-1.ubuntu.18.04_amd64.deb ... Passed
==> Removing existing $srcdir/ directory...
==> Extracting sources...
  -> Extracting powershell_7.1.3-1.ubuntu.18.04_amd64.deb with bsdtar
==> Sources are ready.
==> Making package: powershell-bin 7.1.3-1 (Sat 05 Jun 2021 08:09:01 PM AST)
==> Checking runtime dependencies...
==> Checking buildtime dependencies...
==> WARNING: Using existing $srcdir/ tree
==> Entering fakeroot environment...
==> Starting package()...
==> Tidying install...
  -> Removing libtool files...
  -> Purging unwanted files...
  -> Compressing man and info pages...
==> Checking for packaging issues...
==> Creating package "powershell-bin"...
  -> Generating .PKGINFO file...
  -> Generating .BUILDINFO file...
  -> Adding install file...
  -> Generating .MTREE file...
  -> Compressing package...
==> Leaving fakeroot environment.
==> Finished making: powershell-bin 7.1.3-1 (Sat 05 Jun 2021 08:09:05 PM AST)
==> Cleaning up...
[sudo] password for rebelinux:
loading packages...
warning: powershell-bin-7.1.3-1 is up to date -- reinstalling
resolving dependencies...
looking for conflicting packages...

Packages (1) powershell-bin-7.1.3-1

Total Installed Size:  170.21 MiB
Net Upgrade Size:        0.00 MiB

:: Proceed with installation? [Y/n]
(1/1) checking keys in keyring                                                                                                                  [########################################################################################] 100%
(1/1) checking package integrity                                                                                                                [########################################################################################] 100%
(1/1) loading package files                                                                                                                     [########################################################################################] 100%
(1/1) checking for file conflicts                                                                                                               [########################################################################################] 100%
(1/1) checking available disk space                                                                                                             [########################################################################################] 100%
:: Processing package changes...
(1/1) reinstalling powershell-bin                                                                                                               [########################################################################################] 100%
:: Running post-transaction hooks...
(1/1) Arming ConditionNeedsUpdate...
[rebelinux@blabla ~]$
```

There is another more advanced installation method that also allows you to install Powershell from the command line. I leave here the required commands.

#### Manual installation of Powershell

```bash
git clone https://aur.archlinux.org/powershell-bin.git
cd powershell-bin
makepkg -si
```

To access the Powershell application the `pwsh` command is used to call the interpreter. From the interpreter we can run Powershell commands that are commonly called `Cmdlets`.

```bash
[rebelinux@blabla ~]$ pwsh
<strong>PowerShell 7.1.3</strong>
Copyright (c) Microsoft Corporation.

https://aka.ms/powershell
Type 'help' to get help.

PS /home/rebelinux> 
```

The next step to run PowerCLI is to install its module by using the `Install-Module -name VMware.PowerCLI` command from the Powershell interpreter.

```powershell
PS /home/blabla> Install-Module -name VMware.PowerCLI

Untrusted repository
You are installing the modules from an untrusted repository. If you trust this repository, change its InstallationPolicy value by running the Set-PSRepository cmdlet. Are you sure you want to install the modules from 'PSGallery'?
[Y] Yes  [A] Yes to All  [N] No  [L] No to All  [S] Suspend  [?] Help (default is "N"): A
PS /home/blabla>  
```

To connect to the vCenter we use the `cmdlet` `Connect-VIServer` specifying the IP address or DNS name of the server.

```powershell
PS /home/blabla> Connect-VIServer vcenter-01v.zenprsolutions.local -Verbose -Username administrator@vsphere.local -Password XXXXXX

Name                            Port User
----                            ---- ----
vcenter-01v.zenprsolutions.local 443  VSPHERE.LOCAL\Administrator

PS /home/blabla> 
```

I will use a basic command to do connection testing against vCenter. In this test I will use the `Get-Cluster` command to check the currently created clusters.

```powershell
PS /home/blabla> Get-Cluster

Name                           HAEnabled  HAFailover DrsEnabled DrsAutomationLevel
                                          Level
----                           ---------  ---------- ---------- ------------------
RegionA01-EDGE                 False      1          True       FullyAutomated
RegionHQ-MGMT                  False      1          True       FullyAutomated
RegionA01-COMP                 True       1          True       FullyAutomated

PS /home/blabla> 
```

Another basic example is to use the `Get-Datastore` command to validate which datastores currently exist in the virtual DataCenter. In the output of the `Get-Datastore` command you can see the free and used space in the datastores configured in the DataCenter.

```powershell
PS /home/blabla> Get-Datastore

Name                               FreeSpaceGB      CapacityGB
----                               -----------      ----------
SSD-VM-HIGH-CAPACITY-PERF-KN           173.708         894.000
NVME-VM-HIGH-PERF-01                     0.017         476.750
SSD-VM-HIGH-CAPACITY-PERF-MK           251.536         931.250
HDD-VM-MED-PERF-02                   2,232.268       3,726.000
HDD-VM-MED-PERF-01                   2,509.246       3,726.000
esx-00f                                110.801         111.750
NVME-VFLASH-01                           0.840         238.250
HDD-VM-ISO-LOW-PERF                    606.936         931.250
NFS_SNAP_OFFLOAD                        29.258          50.000
SRM_PlaceHolder                         97.170          99.750
SERVER_DATASTORE                        92.444          99.750

PS /home/blabla> 
```

## Hasta Luego Amigos!
