---
title: "HomeLab – How to enable VMware TPS for greater virtual machine consolidation"
date: 2021-04-26T08:39:31-04:00
author: 'Jonathan Colon Feliciano'
draft: false
tags:
  - "VMware"
---

In this blog, I will be talking about how you can optimize RAM utilization in your **HomeLab**. The main objective of this tutorial is that you can achieve higher levels of consolidation while running your labs.

**Transparent Page Sharing (TPS)** is a method by which duplicate copies of memory pages are consolidated. In other words, the concept of TPS is somewhat similar to deduplication. This helps the ESXi server to free up repeated memory blocks of a virtual machine allowing for increased levels of consolidation.

![Text](/img/image003.webp#center)

If you want to know a little more about TPS, its benefits and risks, you can access the following link [Transparent Page Sharing (TPS) in hardware MMU systems](https://kb.vmware.com/s/article/1021095). Although it is known that the use of TPS can be a security risk, it is my understanding that it may not pose a significant risk in a test environment such as ours. I provide a reference for this information:

> In a nutshell, independent research indicates that TPS can be abused to gain unauthorized access to data under certain highly controlled conditions. In line with its “secure by default” security posture, VMware has opted to change the default behavior of TPS and provide customers with a configurable option for selectively and more securely enabling TPS in their environment.

#### [Disabling TPS in vSphere – Impact on Critical Applications](https://blogs.vmware.com/apps/2014/10/disabling-tps-vsphere-impact-critical-applications.html)

#### Note: I show you how to change this value using Powershell because it allows you to make the change to multiple servers at the same time

To begin we must verify what value is currently configured on the ESXi servers. To accomplish this task i use the **Get-VMHost** command to extract the information of the servers connected to the vCenter. The result is then sent to the **Get-AdvancedSetting -Name Mem.ShareForceSalting** command which allows us to extract the value configured in the **Mem.ShareForceSalting** variable.

```text
PS /home/blabla> Get-VMHost | Get-AdvancedSetting -Name Mem.ShareForceSalting | Select-Object Entity,Name,Value,Type | Format-Table -Wrap -AutoSize

Entity                           Name                  Value   Type
------                           ----                  -----   ----
(esxsvr-00f.zenprsolutions.local Mem.ShareForceSalting     2 VMHost)
comp-02a.zenprsolutions.local    Mem.ShareForceSalting     2 VMHost
comp-01a.zenprsolutions.local    Mem.ShareForceSalting     2 VMHost

PS /home/blabla> 
```

In this particular example the ESXi servers are configured with a default value of **#2**. Using the VMware documentation as a reference this value indicates that the **Inter-VM** TPS feature is disabled.

![Text](/img/2021-06-03_17-39-1.webp#center)

To activate the TPS “Inter-VM” function you can use the command **Set-AdvancedSetting** with the value of #0. It is worth to mention that this command can be activated with the VMs powered on or without the server being in maintenance.

```text
PS /home/blabla> Get-VMHost -Name esxsvr-00f.zenprsolutions.local | Get-AdvancedSetting -Name Mem.ShareForceSalting | Set-AdvancedSetting -Value 0        

Perform operation?
Modifying advanced setting 'Mem.ShareForceSalting'.
[Y] Yes  [A] Yes to All  [N] No  [L] No to All  [S] Suspend  [?] Help (default is "Y"): Y

Name                 Value                Type                 Description
----                 -----                ----                 -----------
Mem.ShareForceSalting 0                    VMHost               

PS /home/blabla> 
```

Once again you can validate with the **Get-AdvancedSetting** command if the configured value is the one i specified previously.

```text
PS /home/blabla> Get-VMHost | Get-AdvancedSetting -Name Mem.ShareForceSalting | Select-Object Entity,Name,Value,Type | Format-Table -Wrap -AutoSize

Entity                          Name                  Value   Type
------                          ----                  -----   ----
esxsvr-00f.zenprsolutions.local Mem.ShareForceSalting     0 VMHost
comp-02a.zenprsolutions.local   Mem.ShareForceSalting     2 VMHost
comp-01a.zenprsolutions.local   Mem.ShareForceSalting     2 VMHost

PS /home/blabla> 
```

For this test i turned on 23 virtual machines (Windows) to bring the server in contention mode so that i can see what benefits the TPS has. This result I show you below represents the memory statistics of the ESXi server obtained with the command **esxtop**. Here you can see the statistics before i configured the **“TPS Inter-VM”** function with the value **#2**. The variable that is in bold **“PSHARE/MB”** represents the value of shared memory that the server currently has, i.e. only TPS is being used in **“Intra-VM”** mode. This variable has a value of **1400/MB**.

```text
2:58:11pm up 50 min, 722 worlds, 23 VMs, 45 vCPUs; MEM overcommit avg: 1.10, 1.10, 0.99
PMEM  /MB: 65398   total: 2139     vmk,60782 other, 2358 free
VMKMEM/MB: 65073 managed:  1265 minfree,  7501 rsvd,  57572 ursvd,  high state
PSHARE/MB:    1648  shared,     248  (common:    1400 saving)
```

Now i move on to validate the benefit of having the **“TPS Inter-VM”** feature enabled. As you can see in the following result of the **esxtop** command there was a substantial saving **(32097/MB)** of memory. This allowed us to increase the consolidation ratios of our **“HomeLab”**.

```text
3:36:46pm up  1:29, 1024 worlds, 23 VMs, 45 vCPUs; MEM overcommit avg: 1.05, 1.05, 0.95
PMEM  /MB: 65398   total: 2262     vmk,60078 other, 3057 free
VMKMEM/MB: 65073 managed:  1265 minfree,  9092 rsvd,  55980 ursvd, clear state
PSHARE/MB:   33038  shared,     941  (common:   32097 saving)
```

**Hasta Luego Amigos!**

![Text](/img/Google-Chrome-the-RAM-eater.webp#center)
