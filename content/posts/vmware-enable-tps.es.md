---
title: "HomeLab: Cómo habilitar VMware TPS para lograr una mayor consolidación de las máquinas virtuales"
date: 2021-04-26T08:39:31-04:00
author: 'Jonathan Colon Feliciano'
draft: false
tags:
  - "VMware"
---

En este blog, estaré hablando sobre cómo podemos optimizar la utilización de memoria RAM en nuestro ambiente de prueba `HomeLab`. El objetivo principal de este procedimiento es que podamos tener mayores niveles de consolidación a la hora de realizar nuestros laboratorios.

`Transparent Page Sharing (TPS)` es un método por el cual se eliminan las copias de páginas de memoria duplicadas. En otras palabras, el concepto de TPS es algo similar a la deduplicación. Esto ayuda al servidor de ESXi a liberar bloques de memoria repetidas de una máquina virtual permitiendo aumentar los niveles de consolidación.

![Text](/img/image003.webp#center)

Si desean saber un poco más sobre TPS su beneficios y riesgos pueden acceder el siguiente enlace [Transparent Page Sharing (TPS) in hardware MMU systems](https://kb.vmware.com/s/article/1021095). Aunque se sabe que el uso de TPS puede ser una alarma de seguridad, entiendo que no representa un riesgo importante en un entorno de pruebas como el nuestro. Les facilito una referencia para esta información:

> In a nutshell, independent research indicates that TPS can be abused to gain unauthorized access to data under certain highly controlled conditions. In line with its `secure by default` security posture, VMware has opted to change the default behavior of TPS and provide customers with a configurable option for selectively and more securely enabling TPS in their environment.

#### [Disabling TPS in vSphere – Impact on Critical Applications](https://blogs.vmware.com/apps/2014/10/disabling-tps-vsphere-impact-critical-applications.html)

#### Nota: Te muestro cómo cambiar este valor usando Powershell porque te permite hacer el cambio en varios servidores al mismo tiempo

Para empezar debemos verificar qué valor está configurado actualmente en los servidores ESXi. Para realizar esta tarea utilizamos el comando `Get-VMHost` para extraer la información de los servidores conectados al vCenter. El resultado se envía al comando `Get-AdvancedSetting -Name Mem.ShareForceSalting` que nos permite extraer el valor configurado en la variable `Mem.ShareForceSalting`.

```text
PS /home/blabla> Get-VMHost | Get-AdvancedSetting -Name Mem.ShareForceSalting | Select-Object Entity,Name,Value,Type | Format-Table -Wrap -AutoSize

Entity                           Name                  Value   Type
------                           ----                  -----   ----
(esxsvr-00f.zenprsolutions.local Mem.ShareForceSalting     2 VMHost)
comp-02a.zenprsolutions.local    Mem.ShareForceSalting     2 VMHost
comp-01a.zenprsolutions.local    Mem.ShareForceSalting     2 VMHost

PS /home/blabla> 
```

En este ejemplo en particular los servidores ESXi están configurados con un valor por defecto de `#2`. Utilizando la documentación de VMware como referencia, este valor indica que la función TPS `Inter-VM` está desactivada.

![Text](/img/2021-06-03_17-39-1.webp#center)

Para activar la función TPS `Inter-VM` utilizamos el comando `Set-AdvancedSetting` con el valor de `#0`. Cabe mencionar que este comando se puede activar con las VMs encendidas o sin que el servidor esté en mantenimiento.

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

De nuevo validamos con el comando `Get-AdvancedSetting` si el valor configurado es el que especificamos anteriormente.

```text
PS /home/blabla> Get-VMHost | Get-AdvancedSetting -Name Mem.ShareForceSalting | Select-Object Entity,Name,Value,Type | Format-Table -Wrap -AutoSize

Entity                          Name                  Value   Type
------                          ----                  -----   ----
esxsvr-00f.zenprsolutions.local Mem.ShareForceSalting     0 VMHost
comp-02a.zenprsolutions.local   Mem.ShareForceSalting     2 VMHost
comp-01a.zenprsolutions.local   Mem.ShareForceSalting     2 VMHost

PS /home/blabla> 
```

Para esta prueba encendimos 23 máquinas virtuales (Windows) para poner el servidor en modo de contención y así poder ver qué beneficios tiene el TPS. Este resultado que les muestro a continuación representa las estadísticas de memoria del servidor ESXi obtenidas con el comando `esxtop`. Aquí podemos ver las estadísticas antes de que configuráramos la función `TPS Inter-VM` con el valor `#2`. La variable que vemos en negrita `PSHARE/MB` representa el valor de memoria compartida que tiene actualmente el servidor, es decir, sólo se está utilizando TPS en modo `Intra-VM`. Esta variable tiene un valor de `1400/MB`.

```text
2:58:11pm up 50 min, 722 worlds, 23 VMs, 45 vCPUs; MEM overcommit avg: 1.10, 1.10, 0.99
PMEM  /MB: 65398   total: 2139     vmk,60782 other, 2358 free
VMKMEM/MB: 65073 managed:  1265 minfree,  7501 rsvd,  57572 ursvd,  high state
PSHARE/MB:    1648  shared,     248  (common:    1400 saving)
```

Ahora pasamos a validar el beneficio de tener habilitada la función `TPS Inter-VM` con el valor #0 en nuestro clúster. Como se puede ver en el siguiente resultado del comando `esxtop`, hubo un ahorro sustancial `(32097/MB)` de memoria. Esto nos permitió aumentar los niveles de consolidación de nuestro `HomeLab`.

```text
3:36:46pm up  1:29, 1024 worlds, 23 VMs, 45 vCPUs; MEM overcommit avg: 1.05, 1.05, 0.95
PMEM  /MB: 65398   total: 2262     vmk,60078 other, 3057 free
VMKMEM/MB: 65073 managed:  1265 minfree,  9092 rsvd,  55980 ursvd, clear state
PSHARE/MB:   33038  shared,     941  (common:   32097 saving)
```

`Hasta Luego Amigos!`

![Text](/img/Google-Chrome-the-RAM-eater.webp#center)
