---
title: 'Nueva versión de AsBuiltReport.Veeam.VBR v0.8.13'
date: '2024-12-11T10:00:00-04:00'
draft: false
tags:
    - VEEAM
    - AsBuiltReport
    - Powershell
---

Hola,

Recientemente, he realizado varios cambios en el informe de [AsBuiltReport.Veeam.VBR](https://github.com/AsBuiltReport/AsBuiltReport.Veeam.VBR), por esta razón resumiré aquí todas las nuevas capacidades añadidas.

Aquí les dejo el enlace al informe más reciente en formato HTML: [Reporte](https://htmlpreview.github.io/?https://raw.githubusercontent.com/AsBuiltReport/AsBuiltReport.Veeam.VBR/dev/Samples/Sample%20Veeam%20Backup%20%26%20Replication%20As%20Built%20Report.html)

El primer cambio que discutiré es el soporte a `Microsoft Entra ID`. En este caso el módulo de Powershell de `Veeam Backup & Replication (VBR)` permite extraer la información de los `Tenant` que están configurados en la infraestructura de VBR.

Esta pantalla muestra la información de los `Tenant` como el Nombre, los ID, la Región y el repositorio cache configurado entre otros.

![abr-veeam-vbr-0813-01](/img/2024/abr-veeam-vbr-0_8_13/abr-veeam-vbr-0813-01.webp)

En esta parte del reporte se muestra el resumen de las tareas de backup de los objetos de `Entra ID`.

![abr-veeam-vbr-0813-02](/img/2024/abr-veeam-vbr-0_8_13/abr-veeam-vbr-0813-02.webp)

Aquí se muestra la configuración de las tareas de backup de `Entra ID`.

![abr-veeam-vbr-0813-03](/img/2024/abr-veeam-vbr-0_8_13/abr-veeam-vbr-0813-03.webp)

![abr-veeam-vbr-0813-04](/img/2024/abr-veeam-vbr-0_8_13/abr-veeam-vbr-0813-04.webp)

En la próxima pantalla se muestra la tabla de `Malware Detection` donde se añadió la información relacionada con `Signature Detection`. Aquí se muestra la información de `Veeam Threat Hunter` introducida en la versión 12.3

![abr-veeam-vbr-0813-05](/img/2024/abr-veeam-vbr-0_8_13/abr-veeam-vbr-0813-05.webp)

La versión 12.3 de VBR añade un `Best Practice` adicional en la sección de `Security & Compliance`. En la siguiente pantalla se muestra el nombre y su estado.

![abr-veeam-vbr-0813-06](/img/2024/abr-veeam-vbr-0_8_13/abr-veeam-vbr-0813-06.webp)

Adicionalmente, les muestro la tabla de `Syslog Event Filters` que se encuentra en la sección de `Event Forwarding`.

![abr-veeam-vbr-0813-07](/img/2024/abr-veeam-vbr-0_8_13/abr-veeam-vbr-0813-07.webp)

En esta nueva versión del reporte se añadió información de los repositorios configurados en `Google Cloud Storage`. Hace tiempo estaba en mi lista añadir soporte a este tipo de repositorio...

![abr-veeam-vbr-0813-08](/img/2024/abr-veeam-vbr-0_8_13/abr-veeam-vbr-0813-08.webp)

En la nueva versión del informe `v0.8.13` se añadió la posibilidad de diagramar objetos relacionados con:

```markdown
- EntraID Tenant Information
- VMware Infrastructure Information
  - vCenter Servers
  - vSphere Clusters
    - Esxi Servers
```

Aquí les muestro el diagrama con los nuevos cambios.

![AsBuiltReport.Veeam.VBR](/img/2024/abr-veeam-vbr-0_8_13/AsBuiltReport.Veeam.VBR.webp)

Para finalizar, aquí les dejo el resto de los cambios que se añadieron o se corrigieron en esta nueva versión del reporte:

```markdown
## [0.8.13] - 2024-12-11

### Added 

- Add EntraID Tenant configuration
  - Add Objects Backup Job information
  - Add EntraID Tenant information to the Infrastructure diagram
- Update Malware detection setting
  - Add Signature Detection
- Update Security & Compliance Best Practice content
- Add Syslog Event Filter information
- Add Google Cloud Storage repository information
- Add VMware Infrastructure information to the Infrastructure diagram

### Changed

- Increase Veeam.Diagrammer minimum requirement to v0.6.18
- Change the infrastructure diagram default save location to $OutputFolderPath
- Increase AsBuiltReport.Core to v1.4.1

### Fixed

- Fix error "A positional parameter cannot be found that accepts argument '-'" at Get-AbrVbrConfigurationBackupSetting cmdlet
- Fix ConvertTo-HashToYN cmdlet not generating an ordereddictionary output
```

¡Espero que todos estos cambios les sean de gran utilizad!

¡Hasta la Próxima!

![Veeam Vanguard](/img/2024/abr-veeam-vbr-0_8_13/veeam_vanguard.webp#center)

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/F1F8DEV80)
