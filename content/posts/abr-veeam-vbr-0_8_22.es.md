---
title: 'Nueva versión de AsBuiltReport.Veeam.VBR v0.8.22'
date: '2025-07-22T10:00:00-04:00'
draft: true
tags:
    - VEEAM
    - AsBuiltReport
    - Powershell
---

Hola desde el Caribe 🥥🌴🌺🌅🌊,

Les soy sincero desde que deje de ser parte del programa de `Veeam Vanguard` he estado un poco alejado del desarrollo de reporte de [AsBuiltReport.Veeam.VBR](https://github.com/AsBuiltReport/AsBuiltReport.Veeam.VBR) pero aun asi este pasado mes estuve añadiendo código dentro del modulo de [Veeam.Diagrammer](https://github.com/rebelinux/Veeam.Diagrammer) para generar diagramas sobre la infraestructura de Cloud Connect y los recursos que utilizan los `Tenant` dentro de la infraestructura de `Veeam Backup & Replication`.

Utilizando Veeam.Diagrammer es posible ahora obtener un diagram de la infraestructura de Cloud Connect y sus componentes. Como se puede ver en la siguiente imagen donde se muestra los componentes configurados dentro de `Veeam Backup & Replication`.

![abr-veeam-vbr-0813-01](/img/2025/abr-veeam-vbr-0_8_22/CloudConnectInfra.webp)


![abr-veeam-vbr-0813-01](/img/2025/abr-veeam-vbr-0_8_22/CloudConnect-Tenant2.webp)

![abr-veeam-vbr-0813-01](/img/2025/abr-veeam-vbr-0_8_22/CloudConnect-Tenant3.webp)

Aquí les muestro el diagrama con los nuevos cambios.

![AsBuiltReport.Veeam.VBR](/img/2025/abr-veeam-vbr-0_8_22/AsBuiltReport.Veeam.VBR.webp)

Para finalizar, aquí les dejo el resto de los cambios que se añadieron o se corrigieron en esta nueva versión del reporte:

```markdown
## [0.8.22] - 2025-07-23

### Added

- Add Cloud Connect infrastructure diagram
- Add per Tenant resources diagram

### Changed

- Updated Get-AbrVbrCloudConnectTenant to include diagram generation
- Modified Invoke-AsBuiltReport to handle tenant-specific diagrams
- Improved handling of diagram parameters and outputs
- Updated changelog and version numbers across scripts
- Refactor diagram section in Get-AbrVbrCloudConnectTenant function for improved error handling and clarity

### Changed

- Bump module to v0.8.22
- Bump Veeam.Diagrammer module to v0.6.30
- Bump Diagrammer.Core module to v0.2.27
- Update workflow to use Windows 2022 for publishing PowerShell module
```

¡Espero que todos estos cambios les sean de gran utilizad!

¡Hasta la Próxima!

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/F1F8DEV80)
