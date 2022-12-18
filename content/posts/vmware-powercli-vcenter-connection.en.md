---
title: 'PowerShell – VMware PowerCLI initial vCenter Connection'
date: '2021-11-10T06:20:00-04:00'
tags:
    - VMware
---

Hi all,

In a previous post [PowerShell: VMware PowerCLI Introduction]({{< ref "/posts/vmware-powercli-introduction.en.md" >}} "PowerShell: VMware PowerCLI Introduction") I showed you how to install VMware PowerCLI and a basic introduction of the purpose of this management tool. In this post I will show you how to do the initial connection to the vCenter using the PowerCLI tool. It is important to mention that PowerCLI can be used to connect to both vCenter and vSphere “ESXi” server independently but in this post I will be showing examples only referring to the vCenter server.

To start, we need to establish the initial connection to our vCenter server. PowerCLI provides the **“Connect-VIServer”** command for this purpose but there are also other commands to manage and even validate existing connections. The following example shows how to deploy the vCenter connect/disconnect commands.

```text
PS /home/rebelinux> Get-Command *VIserver*


CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Alias           Get-VIServer                                       12.4.0.18… VMware.VimAutomation.Core
Cmdlet          Connect-VIServer                                   12.4.0.18… VMware.VimAutomation.Core
Cmdlet          Disconnect-VIServer                                12.4.0.18… VMware.VimAutomation.Core

PS /home/rebelinux>
```

By using the **“Connect-VIServer”** command we establish the initial connection to the vCenter with the IP address **“192.168.5.2”** on my HomeLab.

```text
PS /home/rebelinux> Connect-VIServer -Server 192.168.5.2 -Credential (Get-Credential)

PowerShell credential request
Enter your credentials.
User: administrator@vsphere.local
Password for user administrator@vsphere.local: ********

Name                           Port  User
----                           ----  ----
192.168.5.2                    443   VSPHERE.LOCAL\Administrator

PS /home/rebelinux> 
```

On this example you can see that a connection to vCenter was made by using the **(Get-Credential)** option to request the credentials on the console. In the same way we could create a variable with preset credentials. In this example I show you how to do it.

```text
PS /home/rebelinux> $Credenciales = Get-Credential

PowerShell credential request
Enter your credentials.
User: administrator@vsphere.local
Password for user administrator@vsphere.local: ********
PS /home/rebelinux> Connect-VIServer -Server 192.168.5.2 -Credential $Credenciales   


Name                           Port  User
----                           ----  ----
192.168.5.2                    443   VSPHERE.LOCAL\Administrator

PS /home/rebelinux> 
```

In the previous example the credentials are saved in a variable with the name $Credentials and then this variable is used as **“Input”** in the **“Connect-VIServer”** command using the **(-Credential $Credentials)** option. Once the initial connection is made you can use the variable **$defaultVIServer** to validate the existing connection. It is important to mention that the session found in the **$defaultVIServer** variable will be used by default when using the cmdlet if the **“-Server”** option is not specified.

```text
PS /home/rebelinux> $defaultVIServer


Name                           Port  User
----                           ----  ----
192.168.5.2                    443   VSPHERE.LOCAL\Administrator

PS /home/rebelinux> 
```

By using the **$defaultVIServers** variable we can see all the connections previously made. As you can see in the following example multiple connections are shown where the session with the IP address **“192.168.5.253”** belongs to an ESXi server.

```text
PS /home/rebelinux> $defaultVIServers                                                  


Name                           Port  User
----                           ----  ----
192.168.5.253                  443   root
192.168.5.2                    443   VSPHERE.LOCAL\Administrator

PS /home/rebelinux> 
```

After making the connection to our vCenter we can use any cmdlet included in PowerCLI. On the following example all VM managed by the vCenter are listed using the **“Get-VM”** cmdlet.

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

If there are connections to multiple vCenter sessions all VMs of all connected vCenters will be shown. To filter the result to only a specific vCenter you can use the **“-Server”** option when executing the cmdlet.

```text
PS /home/rebelinux> Get-VMHost -Server 192.168.5.2


Name                 ConnectionState PowerState NumCpu CpuUsageMhz CpuTotalMhz   MemoryUsageGB   MemoryTotalGB Version
----                 --------------- ---------- ------ ----------- -----------   -------------   ------------- -------
esxsvr-00f.pharmax.… Connected       PoweredOn       4         348       13628          17.792          63.865   7.0.3
comp-01a.pharmax.lo… NotResponding   Unknown         2           0        6816           0.000           7.995        
comp-02a.pharmax.lo… NotResponding   Unknown         2           0        6816           0.000           7.995        

PS /home/rebelinux> 
```

Once finished interacting with the vCenter session you can use the **“Disconnect-VIServer”** cmdlet to disconnect the session.

```text
PS /home/rebelinux> Disconnect-VIServer -Server 192.168.5.2

Confirm
Are you sure you want to perform this action?
Performing the operation "Disconnect VIServer" on target "User: VSPHERE.LOCAL\Administrator, Server: 192.168.5.2, Port: 443".
[Y] Yes [A] Yes to All [N] No [L] No to All [S] Suspend [?] Help (default is "Yes"): Y
PS /home/rebelinux> 
```

Finally, by using the **$defaultVIServers** variable you can validate if the session was disconnected successfully.

```text
PS /home/rebelinux> $defaultVIServers                      


Name                           Port  User
----                           ----  ----
192.168.5.253                  443   root

PS /home/rebelinux> 
```

### See you later Amigos!
