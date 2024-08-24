---
title: 'PowerShell: Conexión inicial a vCenter utilizando VMware PowerCLI'
date: '2021-11-10T06:22:00-04:00'
tags:
    - VMware
---

Hola tod@s

En el artículo [PowerShell: Introducción básica de VMware PowerCLI]({{< ref "/posts/vmware-powercli-introduction.es.md" >}} "PowerShell: Introducción básica de VMware PowerCLI") les mostré cómo realizar la instalación de VMware PowerCLI y una introducción básica del propósito de esta herramienta de administración. En este post veremos cómo realizar la conexión inicial a nuestro vCenter utilizando la herramienta de PowerCLI. Es importante mencionar que PowerCLI se puede utilizar para conectarse tanto a vCenter como al servidor de vSphere `ESXi` de forma independiente, pero en este post estaré mostrando ejemplos solo haciendo referencia al servidor de vCenter.

Para comenzar, necesitamos establecer la conexión inicial hacia nuestro servidor de vCenter. PowerCLI ofrece el comando `Connect-VIServer` para este propósito, pero también existe otros comando para manejar y hasta validar las conexiones existentes. El siguiente ejemplo muestra cómo desplegar los comando de conexión/desconexión de vCenter

```text
PS /home/rebelinux> Get-Command *VIserver*


CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Alias           Get-VIServer                                       12.4.0.18… VMware.VimAutomation.Core
Cmdlet          Connect-VIServer                                   12.4.0.18… VMware.VimAutomation.Core
Cmdlet          Disconnect-VIServer                                12.4.0.18… VMware.VimAutomation.Core

PS /home/rebelinux>
```

Utilizando el comando `Connect-VIServer` establecemos la conexión inicial hacia el vCenter en mi HomeLab con la dirección de IP `192.168.5.2`.

```text
PS /home/rebelinux> Connect-VIServer -Server 192.168.5.2 -Credential (Get-Credential)

PowerShell credential request
Enter your credentials.
User: administrator@vsphere.local
Password for user administrator@vsphere.local: ````

Name                           Port  User
----                           ----  ----
192.168.5.2                    443   VSPHERE.LOCAL\Administrator

PS /home/rebelinux> 
```

En este ejemplo se puede ver que realizamos la conexión al vCenter utilizando la opción de `(Get-Credential)` para que sea solicitado las credenciales en consola. De igual forma pudimos crear una variable con las credenciales ya preestablecidas. En este ejemplo les muestro como hacerlo.

```text
PS /home/rebelinux> $Credenciales = Get-Credential

PowerShell credential request
Enter your credentials.
User: administrator@vsphere.local
Password for user administrator@vsphere.local: ````
PS /home/rebelinux> Connect-VIServer -Server 192.168.5.2 -Credential $Credenciales   


Name                           Port  User
----                           ----  ----
192.168.5.2                    443   VSPHERE.LOCAL\Administrator

PS /home/rebelinux> 
```

En el ejemplo anterior guardamos las credenciales en una variable con el nombre de `$Credenciales` para luego utilizar esta variable como `Input` en el comando `Connect-VIServer` utilizando la opción de `(-Credential $Credenciales)`. Una vez realicemos la conexión inicial podemos utilizar la variable `$defaultVIServer` para validar la conexión existente. Es importante mencionar que la sesión que se encuentra en la variable `$defaultVIServer` sseráutilizada de manera predeterminada cuando se utilizan los cmdlet si no se le especifica la opción de `-Server`.

```text
PS /home/rebelinux> $defaultVIServer


Name                           Port  User
----                           ----  ----
192.168.5.2                    443   VSPHERE.LOCAL\Administrator

PS /home/rebelinux> 
```

Al utilizando la variable `$defaultVIServers` podemos ver todas la conexiones previamente realizadas. Como se puede ver en el siguiente ejemplo se muestran las múltiples conexiones donde la sesión con la dirección de IP `192.168.5.253`` pertenece a un servidor de ESXi.

```text
PS /home/rebelinux> $defaultVIServers                                                  


Name                           Port  User
----                           ----  ----
192.168.5.253                  443   root
192.168.5.2                    443   VSPHERE.LOCAL\Administrator

PS /home/rebelinux> 
```

Luego de realizar la conexión hacia nuestro Vcenter podemos utilizar los múltiples cmdlet que vienen incluidos en PowerCLI. En el siguiente ejemplo enlistamos todas las máquinas virtuales que se encuentran manejadas en el vCenter utilizando el cmdlet `Get-VM`.

```text
PS /home/rebelinux> Get-VM 


Name                 PowerState Num CPUs MemoryGB
----                 ---------- -------- --------
Horizon-OPM-01V      PoweredOff 4        16.000
FreeNAS              PoweredOff 2        8.000
Horizon-CS-01V       PoweredOff 2        5.000
vcenter-01v          PoweredOn  2        12.000
Horizon-APV-01V      PoweredOff 2        4.000
cluster1-03          PoweredOff 2        8.000
Horizon-CAP-01V      PoweredOff 2        4.000
horizon-tp-01v       PoweredOff 2        4.000
cluster1-04          PoweredOff 2        8.000
Horizon-LNX-01T      PoweredOff 1        2.000
Horizon-IDM-01V      PoweredOff 2        6.000
Horizon-SQL-01V      PoweredOff 2        4.000
Horizon-IDM-01V      PoweredOff 2        6.000
Horizon-RDS-01T      PoweredOff 2        4.000
Horizon-RDS-01T      PoweredOff 2        4.000
Horizon-OPM-01V      PoweredOff 4        16.000
Horizon-UAG-02V      PoweredOff 2        4.000
Horizon-UAG-01V      PoweredOff 2        4.000
horizon-tp-01v       PoweredOff 2        4.000
Management-PC        PoweredOff 2        8.000
csr-pharmax-hq       PoweredOff 1        4.000
GNS3 Server          PoweredOff 4        20.000
Server-DC-01V        PoweredOn  2        4.000
csr-pharmax-dr       PoweredOff 1        4.000

PS /home/rebelinux
```

Si existen conexiones a múltiples sesiones de vCenter se mostrarán todas las VM de todos los vCenter conectados. Para filtrar el resultado a solo un vCenter en específico podemos utilizar la opción de `-Server` cuando se ejecuta el cmdlet.

```text
PS /home/rebelinux> Get-VMHost -Server 192.168.5.2


Name                 ConnectionState PowerState NumCpu CpuUsageMhz CpuTotalMhz   MemoryUsageGB   MemoryTotalGB Version
----                 --------------- ---------- ------ ----------- -----------   -------------   ------------- -------
esxsvr-00f.pharmax.… Connected       PoweredOn       4         348       13628          17.792          63.865   7.0.3
comp-01a.pharmax.lo… NotResponding   Unknown         2           0        6816           0.000           7.995        
comp-02a.pharmax.lo… NotResponding   Unknown         2           0        6816           0.000           7.995        

PS /home/rebelinux> 
```

Una vez terminamos de interactuar con la sesión del vCenter podemos utilizar el cmdlet `Disconnect-VIServer` para desconectar la sesión.

```text
PS /home/rebelinux> Disconnect-VIServer -Server 192.168.5.2

Confirm
Are you sure you want to perform this action?
Performing the operation "Disconnect VIServer" on target "User: VSPHERE.LOCAL\Administrator, Server: 192.168.5.2, Port: 443".
[Y] Yes [A] Yes to All [N] No [L] No to All [S] Suspend [?] Help (default is "Yes"): Y
PS /home/rebelinux> 
```

Utilizando la variable `$defaultVIServers` podemos validar si la sesión fue desconectada correctamente.

```text
PS /home/rebelinux> $defaultVIServers                      


Name                           Port  User
----                           ----  ----
192.168.5.253                  443   root

PS /home/rebelinux> 
```

### Hasta Luego Amigos!
