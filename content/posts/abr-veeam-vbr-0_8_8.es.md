---
title: 'Nueva versión de AsBuiltReport.Veeam.VBR v0.8.8'
date: '2024-08-24T10:00:00-04:00'
tags:
    - VEEAM
    - AsBuiltReport
    - Powershell
---

Hola,

El mes pasado añadí varios cambios al reporte de `AsBuiltReport.Veeam.VBR` para mejorar su contenido como también añadir más objetos de infraestructura al diagrama.

Aquí les dejo el enlace del reporte en formato HTML: [Reporte](https://htmlpreview.github.io/?https://raw.githubusercontent.com/AsBuiltReport/AsBuiltReport.Veeam.VBR/dev/Samples/Sample%20Veeam%20Backup%20%26%20Replication%20As%20Built%20Report.html)

En la versión `v0.8.6` se añadió la capacidad de generar un diagrama simple de la infraestructura de `Veeam Backup & Replication`.

En la pasada versión del diagrama solo se muestra los siguientes componentes:

```markdown
- Backup Server
  - Database Server
  - Enterprise Manager
- Proxy Servers
- Backup Repositories
  - Object Storage Repositories
  - Archive Object Storage Repositories
- Scale-Out Backup Repositories
- Wan Accelerators
```

En la nueva versión del reporte `v0.8.8` se añadió la capacidad de mostrar los objetos relacionados con:

```markdown
- Tape Infrastructure
  - Tape Server
  - Tape Library
  - Tape Vault
- Service Provider 
```

Aquí les muestro el diagrama con los nuevos cambios. ¡No es algo asombroso! Yay....

![AsBuiltReport.Veeam.VBR](/img/2024/abr-veeam-vbr-0_8_8/AsBuiltReport.Veeam.VBR.webp)

Para finalizar, aquí les dejo el resto de los cambios que se introdujeron o se arreglaron en esta nueva versión del reporte:

```markdown
## [0.8.8] - 2024-07-26

### Added

- Add Tape Infrastructure to the diagram
  - Tape Server
  - Tape Library
  - Tape Vault
- Add Service Provider to the diagram
- Improve Infrastructure diagram error handling
```

¡Hasta la Próxima!

![Veeam Vanguard](/img/2024/abr-veeam-vbr-0_8_8/veeam_vanguard.webp#center)

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/F1F8DEV80)
