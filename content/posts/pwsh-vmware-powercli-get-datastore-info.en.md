---
title: 'PowerShell – VMware PowerCLI get Datastore Information'
date: '2021-12-18T15:54:00-04:00'
tags:
    - VMware
---

Hello,

In this post I will show you how to get the datastore list in a vSphere infrastructure using PowerCLI. Now, first of all we have to establish connection to our vCenter/ESXi to get this information. We can do this using the **“Connect-VIServer”** command.

```text
PS C:\Users\jocolon> Connect-VIServer -Server 192.168.5.2

Specify Credential
Please specify server credential
User: administrator@vsphere.local
Password for user administrator@vsphere.local: ********

Name                           Port  User
----                           ----  ----
192.168.5.2                    443   VSPHERE.LOCAL\Administrator


PS C:\Users\jocolon> 
```

After the connection is established you can use the **“Get-Datastore”** command.

```text
PS C:\Users\jocolon> Get-Datastore 


Name                               FreeSpaceGB      CapacityGB
----                               -----------      ----------
SSD-VM-HIGH-CAPACITY-PERF-KN           399.387         894.000
NVME-VFLASH-01                          50.054         238.250
SSD-VM-HIGH-CAPACITY-PERF-MK           505.415         931.250
NVME-VM-HIGH-PERF-01                   275.067         476.750
HDD-VM-MED-PERF-01                   2,165.800       3,726.000
HDD-VM-MED-PERF-02                   2,230.938       3,726.000
esx-00f                                110.801         111.750
SRM_LAB_STORAGE_01                      46.285          49.750
SRM_POL_DEDUP_01                        18.694          50.000
SRM_LAB_STORAGE_02                      47.909          50.000
SRM_PLACEHOLDER_01                      13.345          14.750
SRM_HQ_EDGE_01                          47.659          49.750
HDD-VM-ISO-LOW-PERF                    461.363         931.250


PS C:\Users\jocolon>
```

This command shows the existing Datastore and the basic information about the used space. With this command we can also filter the search allowing to obtain additional Datastore information. if we add the Datastore name to the **“Get-Datastore”** command with the **“-Name”** option. For example if we use the command **“Get-Datastore -Name &lt;DatastoreName&gt; | Format-List”** the following result is shown.

```text
PS C:\Users\jocolon> Get-Datastore -Name NVME-VFLASH-01 | Format-List  



FileSystemVersion              : 6.82
DatacenterId                   : Datacenter-datacenter-21
Datacenter                     : PHARMAX-VSI-DC
ParentFolderId                 : StoragePod-group-p1396
ParentFolder                   :
DatastoreBrowserPath           : vmstores:\192.168.5.2@443\PHARMAX-VSI-DC\NVME-VFLASH-01
FreeSpaceMB                    : 51255
CapacityMB                     : 243968
Accessible                     : True
Type                           : VMFS
StorageIOControlEnabled        : True
CongestionThresholdMillisecond : 10
State                          : Available
ExtensionData                  : VMware.Vim.Datastore
CapacityGB                     : 238.25
FreeSpaceGB                    : 50.0537109375
Name                           : NVME-VFLASH-01
Id                             : Datastore-datastore-268983
Uid                            : /VIServer=vsphere.local\administrator@192.168.5.2:443/Datastore=Datastore-datastore-268983/



PS C:\Users\jocolon>
```

Like most Powershell commands, the PowerCLI commands allow the use of **[“Pipeline”](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_pipelines?view=powershell-7.2)**. If we use the **“Get-VMHost”** command to get the ESXi servers together with the “Get-Datastore” command you can further filter the content of the Datastore connected to the server.

```text
PS C:\Users\jocolon> Get-VMHost -Name esxsvr-00f.pharmax.local | Get-Datastore


Name                               FreeSpaceGB      CapacityGB
----                               -----------      ----------
esx-00f                                110.801         111.750
HDD-VM-ISO-LOW-PERF                    461.363         931.250
SSD-VM-HIGH-CAPACITY-PERF-MK           505.414         931.250
SSD-VM-HIGH-CAPACITY-PERF-KN           399.321         894.000
HDD-VM-MED-PERF-01                   2,165.800       3,726.000
HDD-VM-MED-PERF-02                   2,230.938       3,726.000
NVME-VM-HIGH-PERF-01                   275.065         476.750
NVME-VFLASH-01                          50.053         238.250


PS C:\Users\jocolon>
```

Finally, I would like to mention that there are endless possibilities in Powershell when it comes to interlinking cmdlets, allowing you to refine the search for data and even transform the information you want to display. I hope this information has been useful. Hasta luego!
