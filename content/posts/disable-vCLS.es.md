---
title: "HomeLab: Cómo desabilitar vSphere Cluster Services (vCLS)"
author: 'Jonathan Colon Feliciano'
date: 2020-12-18T08:39:31-04:00
draft: false
tags:
  - "VMware"
---

En la versión de vSphere 7 update 1 VMware añadió una nueva capacidad para la tecnología de `Distributed Resource Scheduler` (DRS) que consiste en tres VM  llamadas agentes. Las VMs agentes forman el estado de quórum del cluster y tienen la capacidad de auto corregirse. De forma que si apagas o borrar las VMs llamadas vCLS el servidor de vCenter volverá a encender o crear las VM nuevamente. Para efectos de un HomeLab esta nueva función consume recurso de CPU, Memoria y espacio de disco que aunque es mínimo no vale la pena tener una configuración que no añade nada a un ambiente de prueba y desarrollo.

![Text](/img/25079036c801ac924d3ff7d4cb3b9438.webp#center)

En este blog estaré mostrando como eliminar esta función pero es importante recalcar de no implementar este cambio en ambientes de producción. En la siguiente imagen les muestro la cantidad mínimas de VM utilizadas en vCenter 7U1.

##### Nota: vSphere DRS depende del estado de los servicios vCLS a partir de vSphere 7.0 Update 1.

![Text](/img/2021-05-30_12-21.webp#center)

Como pueden ver en mi caso el clúster RegionA01-COMP esta compuesto de dos servidores ESXi donde existen tres VM  tipo vCLS.

![Text](/img/2021-05-29_22-52-1024x649.webp#center)

Lo primero que debemos hacer es identificar el ID de dominio del clúster que es necesario para poder añadir un valor avanzado en la configuración del vCenter. Este ID puede ser identificado de dos formas: Desde vCenter o utilizando Powershell con PowerCLI

vCenter: En esta parte puedes obtener el ID del clúster navegando a la pestaña de [Hosts and Clusters] luego seleccionamos el clúster que vamos a editar donde se puede ver que el ID esta en la dirección URL del navegador.

![Text](/img/2021-05-30_00-11-1024x309.webp#center)

PowerCLI: Utilizando Powershell con el modulo de VMware.PowerCLI podemos obtener el ID del clúster al invocar el commando `Get-Cluster`. Según se muestra en el resultado del commando se puede ver el valor del Id `domain-c81` para el clúster llamado RegionA01-COMP.

```text
PS /home/rebelinux> Get-Cluster RegionA01-COMP | FL

ParentId                        : Folder-group-h23
ParentFolder                    : host
HAEnabled                       : True
HAAdmissionControlEnabled       : True
HAFailoverLevel                 : 1
HARestartPriority               : Medium
HAIsolationResponse             : DoNothing
VMSwapfilePolicy                : WithVM
DrsEnabled                      : True
DrsMode                         : FullyAutomated
DrsAutomationLevel              : FullyAutomated
CryptoMode                      : OnDemand
CollectiveHostManagementEnabled : False
Name                            : RegionA01-COMP
ExtensionData                   : VMware.Vim.ClusterComputeResource
Id                              : ClusterComputeResource-(domain-c81)

PS /home/rebelinux>
```

Una vez tengamos el ID del clúster que en mi caso es `domain-c81` procedemos a añadir la configuracion en vCenter navegando a `[Advanced Settings => Configure => Edit Settings]`.

![Text](/img/2021-05-29_21-47-1024x599.webp#center)

En esta pantalla añadimos el valor `config.vcls.clusters.domain-domain-cID.enabled` que en mi caso sería `config.vcls.clusters.domain-c81.enabled` con el valor de `false` en el campo de `Value`.

![Text](/img/2021-05-29_22-53-1024x871.webp#center)

Una vez se añada el valor las VM de tipo vCLS serán eliminadas del clúster como pueden ver en la imagen ya las VM no existen.

![Text](/img/2021-05-29_23-12-1024x648.webp#center)

En esta imagen se puede ver las tareas que realizo el vCenter para remover las VM vCLS

![Text](/img/2021-05-29_23-14-1024x426.webp#center)

### Hasta luego!!!

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/F1F8DEV80)

![Text](/img/hasta-luego-5937ba.webp#center)
