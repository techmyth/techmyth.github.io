---
title: 'HomeLab: vSphere documentation with vDocumentation'
date: '2021-12-16T23:00:00-04:00'
tags:
    - VMware
---

This time I will be showing a tool that I use regularly to quickly identify inventory information in a vSphere infrastructure. This tool is called `vDocumentation` and precisely uses PowerCLI to obtain the inventory information. The creator of this tool `Ariel Sanchez Mora` @arielsanchezmor defines vDocumentation as:

> vDocumentation provides a community-created set of PowerCLI scripts that produce infrastructure documentation of vSphere environments in CSV or Excel file format.
>
> [Ariel Sánchez Mora Github page](https://github.com/arielsanchezmora/vDocumentation)

![Text](/img/inventory_meme.webp#center)

However, to get started, the following requirements must be met:

- Windows OS platform
- PowerShell v5.1+ or v7
- ImportExcel module >= 7+
- VMware PowerCLI module >= 12.3+

To install this tool we only need to use the `Install-Module` command from a Powershell console.

```powershell
Install-Module -Name VMware.PowerCLI
Install-Module -Name ImportExcel
Install-Module -Name vDocumentation
```

Once the modules are installed the installation can be verified using the `Get-Module` command.

```text
PS C:\Users\Administrator> Get-Module -ListAvailable -Name @('VMware.PowerCLI','ImportExcel','vDocumentation')


ModuleType Version    PreRelease Name                                PSEdition ExportedCommands
---------- -------    ---------- ----                                --------- ----------------
Script     7.4.1                 ImportExcel                         Desk      {Add-Conditional...}
Script     2.4.7                 vDocumentation                      Desk      {Get-ESXStorage...}
Manifest   12.4.1.18…            VMware.PowerCLI                     Desk      

PS C:\Users\Administrator>
```

After validating the installation process, use the `Get-Command` command to identify the cmdlets that this tool brings to PowerShell.

```text
PS C:\Users\Administrator> Get-Command -Module vDocumentation


CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Function        Get-ESXInventory                                   2.4.7      vDocumentation
Function        Get-ESXIODevice                                    2.4.7      vDocumentation
Function        Get-ESXNetworking                                  2.4.7      vDocumentation
Function        Get-ESXPatching                                    2.4.7      vDocumentation
Function        Get-ESXSpeculativeExecution                        2.4.7      vDocumentation
Function        Get-ESXStorage                                     2.4.7      vDocumentation
Function        Get-VMSpeculativeExecution                         2.4.7      vDocumentation
Function        Get-vSANInfo                                       2.4.7      vDocumentation

PS C:\Users\Administrator>
```

Before using the modules it is required to first establish the connection to the vCenter server or ESXi. To achieve this task we use the command `Connect-VIServer`.

```text
PS C:\Users\Administrator> Connect-VIServer -Server 192.168.5.2 -Username administrator@vsphere.local -Password XXXXXX


Name                           Port  User
----                           ----  ----
192.168.5.2                    443   VSPHERE.LOCAL\Administrator

PS C:\Users\Administrator>
```

After the connection is established the modules can be used. In this example I will use the command ``Get-ESXInventory` to get the inventory information of the ESXi servers connected to the vCenter.

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

The next example I will show you is about the `Get-ESXStorage` command. This command allows you to get storage device information connected to the ESXi server.

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

The last example I will show you is the `Get-ESXNetworking` command used to retrieve the information of the existing physical and virtual network ports on the server.

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

I hope you have found this reporting tool useful. See you later!
