---
title: 'PowerShell – VMware PowerCLI obtener información sobre los Datastore'
date: '2021-12-18T15:56:00-04:00'
tags:
    - VMware
---

Hola a todos,

En este post le mostraré como obtener la lista de datastore en una infraestructura de vSphere utilizando PowerCLI. Ahora bien, primero que todo tenemos que establecer conexión a nuestro vCenter/ESXi para obtener esta información. Esto podemos hacerlo utilizando el comando **“Connect-VIServer”**.

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

Luego de establecida la conexión podemos utilizar el comando **“Get-Datastore”**.

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

Este comando muestra los Datastore existentes y la información básica del espacio utilizado. Con este comando podemos también filtrar la búsqueda permitiendo obtener información adicional del Datastore si añadimos el nombre del Datastore al comando **“Get-Datastore”** con la opción de **“-Name”**. Por ejemplo si utilizamos el comando **“Get-Datastore -Name &lt;Datastore Name&gt; | Format-List ”** se muestra el siguiente resultado.

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

Como la mayoría de los comandos de Powershell, los comando de PowerCLi permiten la utilización de [“Pipeline”](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_pipelines?view=powershell-7.2). Si utilizamos el comando **“Get-VMHost**” para desplegar los servidores de ESXi junto con el comando **“Get-Datastore”** podemos filtrar aún más el contenido de los Datastore conectados al servidor.

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

Para finalizar les comento que existe un sin fin de posibilidades en Powershell a la hora de entrelazar los cmdlets permitiendo afinar la búsqueda de datos y hasta transformar la información que deseamos desplegar. Espero les haya sido de utilidad esta información. Hasta luego!
