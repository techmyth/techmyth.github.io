---
title: 'Cómo instalar y utilizar VMware PowerCLI en ArchLinux'
date: '2021-06-05T19:27:18-04:00'
author: 'Jonathan Colon Feliciano'
tags:
    - VMware
---

En este blog les estaré mostrando como instalar la herramienta de PowerCLI específicamente en el sistema operativo de ArchLinux. Archlinux es una distribución avanzada de Linux que se caracteriza por ser simple y ligera. Adicionalmente le ofrece al usuario total control en el manejo y la modificación de todo lo relacionado al sistema.

Ahora bien, para instalar y utilizar PowerCLI tenemos que instalar primero Powershell. PowerShell es una herramienta de automatización y configuración multi-plataforma. PowerShell tiene un gran número de comandos orientados a la administración del sistema. Pero al mismo tiempo, PowerShell es un lenguaje de programación completo que permite escribir programas funcionales. Existen muchas herramientas de administración basadas en Powershell de diferentes manufactureros como por ejemplo:

- VMware PowerCLI
- Cisco UCS PowerTool
- NetApp PowerShell Toolkit
- DELL\EMC Unity-Powershell
- Amazon AWS Tools for PowerShell

Como pueden ver Powershell es una herramienta altamente desarrollada por los manufactureros de infraestructura y es ofrecida como un método para la automatización o la rápida implementación de infraestructura basada en `software`. En mi caso particular Powershell es mi segundo lenguaje de programación preferido siendo Python el primero en mí lista.

Lo primero que tenemos que hacer es instalar Powershell y para esto utilizaré el programa `yay` que es una herramienta en Archlinux para instalar programas desde el repositorio no oficial ``Arch User Repository``. Con el comando `yay -S powershell-bin` podemos instalar el programa de Powershell al sistema. Para ver el procedimiento de instalación, basta con hacer clic en el icono `+`.

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

Existe otro método de instalación más avanzado que permite también instalar Powershell desde linea de comando. Les dejo aquí los comando necesarios. Para ver el procedimiento de instalación, basta con hacer clic en el icono `+`.

#### Instalación manual de Powershell

```bash
git clone https://aur.archlinux.org/powershell-bin.git
cd powershell-bin
makepkg -si
```

Para acceder la aplicación de Powershell se utiliza el comando `pwsh` que permite acceder el interpretador. Desde el interpretador podemos correr los comando de Powershell que son comúnmente llamados `Cmdlets`.

```bash
[rebelinux@blabla ~]$ pwsh
<strong>PowerShell 7.1.3</strong>
Copyright (c) Microsoft Corporation.

https://aka.ms/powershell
Type 'help' to get help.

PS /home/rebelinux> 
```

El próximo paso para correr PowerCLI es instalar su modulo utilizando el comando `Install-Module -name VMware.PowerCLI` desde el interpretador de Powershell.

```powershell
PS /home/blabla> Install-Module -name VMware.PowerCLI

Untrusted repository
You are installing the modules from an untrusted repository. If you trust this repository, change its InstallationPolicy value by running the Set-PSRepository cmdlet. Are you sure you want to install the modules from 'PSGallery'?
[Y] Yes  [A] Yes to All  [N] No  [L] No to All  [S] Suspend  [?] Help (default is "N"): A
PS /home/blabla>  
```

Para conectarnos al vCenter utilzamos el `cmdlet` `Connect-VIServer` especificando la dirección IP o el nombre DNS del servidor.

```powershell
PS /home/blabla> Connect-VIServer vcenter-01v.zenprsolutions.local -Verbose -Username administrator@vsphere.local -Password XXXXXX

Name                            Port User
----                            ---- ----
vcenter-01v.zenprsolutions.local 443  VSPHERE.LOCAL\Administrator

PS /home/blabla> 
```

Utilizaré un comando básico para hacer pruebas de conexión contra el vCenter. En esta prueba utilizare el comando `Get-Cluster` para verificar los cluster actualmente creados.

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

Otro ejemplo básico es utilizar el comando `Get-Datastore` para validar que `Datastore` existen actualmente en el ``DataCenter`` virtual. En el resultado del comando se puede ver el espacio libre y utilizado en los ``datastore`` configurados en el ``DataCenter``.

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
