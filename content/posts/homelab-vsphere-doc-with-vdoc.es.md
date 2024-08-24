---
title: 'HomeLab: Documentación de vSphere utilizando vDocumentation'
date: '2021-12-16T23:00:00-04:00'
tags:
    - VMware
---

En esta ocasión estaré mostrando una herramienta que utilizo regularmente para identificar de manera rápida la información de inventario en una infraestructura de vsphere. Esta herramienta se llama `vDocumentation` y precisamente utiliza PowerCLI para obtener la información de inventario. El creador de esta herramienta `Ariel Sánchez Mora` @arielsanchezmor define vDocumentation como:

> vDocumentation provides a community-created set of PowerCLI scripts that produce infrastructure documentation of vSphere environments in CSV or Excel file format.
>
>[Ariel Sánchez Mora Github page](https://github.com/arielsanchezmora/vDocumentation)

![Text](/img/inventory_meme.webp#center)

Ahora bien, para comenzar es necesario cumplir con los siguientes requisitos:

- Plataforma Windows OS
- PowerShell v5.1+ ó v7
- El módulo de ImportExcel >= 7+
- El módulo de VMware PowerCLI >= 12.3+

Para instalar esta herramienta solo necesitamos utilizar el comando `Install-Module` desde una consola de Powershell.

```text
Install-Module -Name VMware.PowerCLI
Install-Module -Name ImportExcel
Install-Module -Name vDocumentation
```

Una vez instalado los módulos podemos verificar la instalación utilizando el comando `Get-Module`.

```text
PS C:\Users\Administrator> <strong>Get-Module</strong> -ListAvailable -Name @('VMware.PowerCLI','ImportExcel','vDocumentation')


ModuleType Version    PreRelease Name                                PSEdition ExportedCommands
---------- -------    ---------- ----                                --------- ----------------
Script     7.4.1                 <strong>ImportExcel</strong>                         Desk      {Add-Conditional...}
Script     2.4.7                 <strong>vDocumentation</strong>                      Desk      {Get-ESXStorage...}
Manifest   12.4.1.18…            <strong>VMware.PowerCLI</strong>                     Desk      

PS C:\Users\Administrator>
```

Luego de validar la instalación puedes utilizar el comando `Get-Command` para identificar los cmdlets que incluye esta herramienta a PowerShell.

```text
PS C:\Users\Administrator> <strong>Get-Command</strong> -Module <strong>vDocumentation</strong>


CommandType     <strong>Name</strong>                                               Version    Source
-----------     ----                                               -------    ------
Function        <strong>Get-ESXInventory</strong>                                   2.4.7      vDocumentation
Function        <strong>Get-ESXIODevice</strong>                                    2.4.7      vDocumentation
Function        <strong>Get-ESXNetworking</strong>                                  2.4.7      vDocumentation
Function        <strong>Get-ESXPatching</strong>                                    2.4.7      vDocumentation
Function        <strong>Get-ESXSpeculativeExecution </strong>                       2.4.7      vDocumentation
Function        <strong>Get-ESXStorage</strong>                                     2.4.7      vDocumentation
Function        <strong>Get-VMSpeculativeExecution</strong>                         2.4.7      vDocumentation
Function        <strong>Get-vSANInfo</strong>                                       2.4.7      vDocumentation

PS C:\Users\Administrator>
```

Antes de utilizar los módulos es necesario establecer la conexión inicial con el servidor de vCenter o el ESXi. Para lograr este objetivo utilizamos el comando `Connect-VIServer`.

```text
PS C:\Users\Administrator> <strong>Connect-VIServer</strong> -Server 192.168.5.2 -Username administrator@vsphere.local -Password XXXXXX


Name                           Port  User
----                           ----  ----
192.168.5.2                    443   VSPHERE.LOCAL\Administrator

PS C:\Users\Administrator>
```

Luego de establecida la conexión podemos utilizar los módulos. En este ejemplo utilizaré el comando `Get-ESXInventory` para obtener la información de inventario de los servidores ESXi conectados al vCenter con la dirección de IP `192.168.5.2`

```text
PS C:\Users\Administrator> Get-ESXInventory
        Connected to 192.168.5.2
        Gathering host list...
        Gathering Hardware inventory from esxsvr-00f.pharmax.local ...
        Gathering configuration details from esxsvr-00f.pharmax.local ...
WARNING:  Check Connection State or Host name
WARNING:  Skipped hosts:


 ESXi Hardware Inventory:


Hostname           : esxsvr-00f.pharmax.local
Management IP      : {192.168.5.253, 10.0.0.74}
RAC IP             :
RAC MAC            :
RAC Firmware       :
Product            : VMware ESXi
Version            : 7.0.3 U3
Build              : 18825058
Make               : System manufacturer
Model              : System Product Name
S/N                : System Serial Number
BIOS               : 3802
BIOS Release Date  : 03/15/2018
CPU Model          : Intel(R) Core(TM) i7-6700 CPU @ 3.40GHz
CPU Count          : 1
CPU Core Total     : 4
Speed (MHz)        : 3408
Memory (GB)        : 64
Memory Slots Count : -1
Memory Slots Used  : 0
Power Supplies     : 0
NIC Count          : 3




 ESXi Host Configuration:


Hostname                  : esxsvr-00f.pharmax.local
Make                      : System manufacturer
Model                     : System Product Name
CPU Model                 : Intel(R) Core(TM) i7-6700 CPU @ 3.40GHz
Hyper-Threading           : True
Max EVC Mode              : intel-broadwell
Product                   : VMware ESXi
Version                   : 7.0.3 U3
Build                     : 18825058
Install Type              : Installable
Boot From                 : /vmfs/volumes/755b8362-294638aa-a59f-74ccb58569b3
Device Model              : ATA SAMSUNG MZHPV128
Boot Device               : Local ATA Disk (t10.ATA_____SAMSUNG_MZHPV128HDGM2D00000______________S1X3NYAH200184______)
Runtime Name              : vmhba3:C0:T0:L0
Device Path               : /vmfs/devices/disks/t10.ATA_____SAMSUNG_MZHPV128HDGM2D00000______________S1X3NYAH200184______
Image Profile             : (Updated) ESXi-7.0U3a-18825058-standard
Acceptance Level          : CommunitySupported
Boot Time                 : 12/16/2021 6:40:22 AM
Uptime                    : 0 Day(s), 16 Hour(s), 3 Minute(s)
Install Date              : 5/22/2020 9:36:56 AM
Last Patched              : 2021-10-29
License Version           : vSphere 7 Enterprise Plus with Add-on for Kubernetes
License Key               : 4Y28A-FV213-488Q8-202E6-1P024
Connection State          : Connected
Standalone                : False
Cluster                   : RegionHQ-MGMT
Virtual Datacenter        : PHARMAX-VSI-DC
vCenter                   : 192.168.5.2
NTP                       : NTP Daemon
NTP Running               : True
NTP Startup Policy        : on
NTP Client Enabled        : True
NTP Server                : 192.168.5.1,0.north-america.pool.ntp.org
SSH                       : SSH
SSH Running               : True
SSH Startup Policy        : on
SSH TimeOut               : 0
SSH Server Enabled        : True
ESXi Shell                : ESXi Shell
ESXi Shell Running        : False
ESXi Shell Startup Policy : off
ESXi Shell TimeOut        : 0
Syslog Server             : :
Syslog Client Enabled     : False



PS C:\Users\Administrator>
```

El próximo ejemplo que les mostraré es sobre el comando `Get-ESXStorage`. Este comando permite obtener información sobre los dispositivos de almacenamiento conectados al servidor de ESXi.

```text
PS C:\Users\Administrator> Get-ESXStorage
        Connected to 192.168.5.2
        Gathering host list...
        Gathering storage adapter details from esxsvr-00f.pharmax.local ...
        Gathering Datastore details from esxsvr-00f.pharmax.local ...
WARNING:  Check Connection State or Host name
WARNING:  Skipped hosts:

 ESXi Datastores:


Hostname               : esxsvr-00f.pharmax.local
Datastore Name         : esx-00f
Device Name            : Local ATA Disk
Canonical Name         : t10.ATA_____SAMSUNG_MZHPV128HDGM2D00000______________S1X3NYAH200184______
LUN                    : 0
Type                   : VMFS
Datastore Cluster      :
Capacity (GB)          : 111.75
Provisioned Space (GB) : 0.95
Free Space (GB)        : 110.80
Transport              : Block
Mount Point            : /vmfs/volumes/598a435f-a2e283e6-6159-9c5c8ed12a16/
Multipath Policy       :
File System Version    : 5.81

Hostname               : esxsvr-00f.pharmax.local
Datastore Name         : HDD-VM-ISO-LOW-PERF
Device Name            : Local ATA Disk
Canonical Name         : t10.ATA_____WDC_WD10EZEX2D00WN4A0_________________________WD2DWCC6Y2KP8070
LUN                    : 0
Type                   : VMFS
Datastore Cluster      :
Capacity (GB)          : 931.25
Provisioned Space (GB) : 469.89
Free Space (GB)        : 461.36
Transport              : Block
Mount Point            : /vmfs/volumes/598f33a5-24870100-5609-9c5c8ed12a16/
Multipath Policy       :
File System Version    : 6.81

PS C:\Users\Administrator>
```

El último comando que les mostraré es el `Get-ESXNetworking` utilizado para obtener la información de los puertos de red físicos y virtuales existentes en el servidor.

```text
PS C:\Users\Administrator> Get-ESXNetworking
        Connected to 192.168.5.2
        Gathering host list...
        Gathering physical adapter details from esxsvr-00f.pharmax.local ...
        Gathering VMkernel adapter details from esxsvr-00f.pharmax.local ...
        Gathering virtual vSwitches details from esxsvr-00f.pharmax.local ...
WARNING:  Check Connection State or Host name
WARNING:  Skipped hosts:


 ESXi Physical Adapters:


Hostname           : esxsvr-00f.pharmax.local
Name               : vmnic1
Slot Description   : PCIEX16_3
Device             : Emulex Corporation HP NC552SFP Dual Port 10GbE Server Adapter
Duplex             : Full
Link               : Up
MAC                : 10:60:4b:93:20:68
MTU                : 9000
Speed              : 10000
vSwitch            : PHARMAX-DVS
vSwitch MTU        : 9000
Discovery Protocol : CDP
Device ID          : MikroTik
Device IP          : 0.0.0.0
Port               : bonding1/sfp-sfpplus1





 ESXi VMkernel Adapters:


Hostname         : esxsvr-00f.pharmax.local
Name             : vmk1
MAC              : 00:50:56:6e:c1:24
MTU              : 9000
IP               : 192.168.5.253
Subnet Mask      : 255.255.255.0
TCP/IP Stack     : defaultTcpipStack
Default Gateway  : 192.168.5.254
DNS              : 192.168.5.1,192.168.1.1
PortGroup Name   : DVS-ESXi-MANAGEMENT
VLAN ID          :
Enabled Services : Management
vSwitch          : PHARMAX-DVS
vSwitch MTU      : 9000
Active adapters  :
Standby adapters :
Unused adapters  :

Hostname         : esxsvr-00f.pharmax.local
Name             : vmk0
MAC              : 00:50:56:6f:e2:03
MTU              : 1500
IP               : 10.0.0.74
Subnet Mask      : 255.255.255.0
TCP/IP Stack     : defaultTcpipStack
Default Gateway  : 192.168.5.254
DNS              : 192.168.5.1,192.168.1.1
PortGroup Name   : Management Network
VLAN ID          : 0
Enabled Services : Management
vSwitch          : vSwitch0
vSwitch MTU      : 1500
Active adapters  : vmnic0
Standby adapters :
Unused adapters  :


 ESXi Virtual Switches:


Hostname                                        : esxsvr-00f.pharmax.local
Type                                            : Standard
Version                                         :
Name                                            : Pharmax-VL
Uplink/ConnectedAdapters                        :
PortGroup                                       : Pharmax-VL DVS-Esxi-VM-Network
VLAN ID                                         : 7
Active adapters                                 :
Standby adapters                                :
Unused adapters                                 :
Security Promiscuous/MacChanges/ForgedTransmits : Reject/Reject/Reject

Hostname                                        : esxsvr-00f.pharmax.local
Type                                            : Standard
Version                                         :
Name                                            : Virtual Lab - Cert
Uplink/ConnectedAdapters                        :
PortGroup                                       : Virtual Lab - Cert DVS-ESXi-MANAGEMENT-EPH
VLAN ID                                         : 5
Active adapters                                 :
Standby adapters                                :
Unused adapters                                 :
Security Promiscuous/MacChanges/ForgedTransmits : Accept/Accept/Accept

Hostname                                        : esxsvr-00f.pharmax.local
Type                                            : Standard
Version                                         :
Name                                            : Virtual Lab - Cert
Uplink/ConnectedAdapters                        :
PortGroup                                       : Virtual Lab - Cert DVS-Esxi-VM-Network
VLAN ID                                         : 7
Active adapters                                 :
Standby adapters                                :
Unused adapters                                 :
Security Promiscuous/MacChanges/ForgedTransmits : Accept/Accept/Accept


PS C:\Users\Administrator>
```

Espero les haya sido de utilidad esta herramienta de reporte. ¡Hasta Luego!
