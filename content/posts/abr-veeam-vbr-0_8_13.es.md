---
title: 'Nueva versión de AsBuiltReport.Veeam.VBR v0.8.13'
date: '2024-12-11T10:00:00-04:00'
draft: true
tags:
    - VEEAM
    - AsBuiltReport
    - Powershell
---

Hola,

Recientemente realice varios cambios al reporte de `AsBuiltReport.Veeam.VBR` para añadir soporte a la version de `Veeam Backup & Replication v12.3`, para corregir varios errores como también añadir más objetos al diagrama de infraestructura.

Aquí les dejo el enlace del mas reciente reporte en formato HTML: [Reporte](https://htmlpreview.github.io/?https://raw.githubusercontent.com/AsBuiltReport/AsBuiltReport.Veeam.VBR/dev/Samples/Sample%20Veeam%20Backup%20%26%20Replication%20As%20Built%20Report.html)

En la nueva versión del reporte `v0.8.13` se añadió la capacidad de mostrar los objetos relacionados con:

```markdown
- EntraID Tenant Information
- VMware Infrastructure Information
  - vCenter Servers
  - vSphere Clusters
    - Esxi Servers
```

Aquí les muestro el diagrama con los nuevos cambios. ¡No es algo asombroso! Yay....

![AsBuiltReport.Veeam.VBR](/img/2024/abr-veeam-vbr-0_8_13/AsBuiltReport.Veeam.VBR.webp)

Para finalizar, aquí les dejo el resto de los cambios que se introdujeron o se arreglaron en esta nueva versión del reporte:

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

¡Hasta la Próxima!

![Veeam Vanguard](/img/2024/abr-veeam-vbr_0_8_13/veeam_vanguard.webp#center)

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/F1F8DEV80)
