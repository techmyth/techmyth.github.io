---
title: 'Nueva versión de AsBuiltReport.Veeam.VBR v0.8.22'
date: '2025-07-22T10:00:00-04:00'
draft: false
tags:
    - VEEAM
    - AsBuiltReport
    - Powershell
---

Hola desde el Caribe 🥥🌴🌺🌅🌊,

Desde que dejé de formar parte del programa `Veeam Vanguard`, he estado algo distanciado del desarrollo del reporte [AsBuiltReport.Veeam.VBR](https://github.com/AsBuiltReport/AsBuiltReport.Veeam.VBR). Sin embargo, el mes pasado dediqué tiempo a mejorar el módulo [Veeam.Diagrammer](https://github.com/rebelinux/Veeam.Diagrammer), añadiendo funcionalidades para generar diagramas de la infraestructura de Cloud Connect y los recursos utilizados por los `Tenants` en `Veeam Backup & Replication`.

Gracias a estas mejoras en Veeam.Diagrammer, ahora es posible obtener diagramas detallados de la infraestructura de Cloud Connect y sus componentes. En la siguiente imagen se pueden apreciar los elementos configurados dentro de `Veeam Backup & Replication`.

![abr-veeam-vbr-0813-01](/img/2025/abr-veeam-vbr-0_8_22/CloudConnectInfra.webp)

Otra capacidad destacada en esta versión es la generación de diagramas específicos por Tenant, lo que permite visualizar de manera clara los recursos utilizados dentro de la infraestructura. Ahora, con `Veeam.Diagrammer`, es posible extraer información detallada sobre recursos de réplica (vSphere, VCloud Director y Hyper-V), recursos de respaldo, SubTenants, WAN Accelerators y dispositivos de red, facilitando así el análisis y la gestión de los entornos de cada Tenant.

#### Ejemplo de los diagramas de Tenants

![abr-veeam-vbr-0813-01](/img/2025/abr-veeam-vbr-0_8_22/CloudConnect-Tenant2.webp)

![abr-veeam-vbr-0813-01](/img/2025/abr-veeam-vbr-0_8_22/CloudConnect-Tenant3.webp)

> **Nota:** Para que los diagramas de Tenant sean generados la opción de Tenant dentro de [InfoLevel](https://github.com/AsBuiltReport/AsBuiltReport.Veeam.VBR?tab=readme-ov-file#infolevel) debe de ser cambiada a 2

A continuación, les presento el diagrama principal de la infraestructura de `Veeam`, en el que se han incorporado los nuevos componentes de Cloud Connect para ofrecer una visión más completa y detallada del entorno.

![AsBuiltReport.Veeam.VBR](/img/2025/abr-veeam-vbr-0_8_22/AsBuiltReport.Veeam.VBR.webp)

Para finalizar, les incluyo el resto de los cambios que se añadieron o se corrigieron en esta nueva versión del reporte:

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

Todos estos nuevos cambios dentro de [Veeam.Diagrammer](https://github.com/rebelinux/Veeam.Diagrammer) han sido añadidos al reporte de [AsBuiltReport.Veeam.VBR](https://github.com/AsBuiltReport/AsBuiltReport.Veeam.VBR) para generar los diagramas de la infraestructura de `Veeam Backup & Replication`.

#### Ejemplo del reporte

[AsBuiltReport.Veeam.VBR Ejemplo del reporte](https://htmlpreview.github.io/?https://raw.githubusercontent.com/AsBuiltReport/AsBuiltReport.Veeam.VBR/dev/Samples/Sample%20Veeam%20Backup%20%26%20Replication%20As%20Built%20Report.html)

¡Hasta la Próxima!
> Recientemente me han contactado para preguntar sobre el estado de este proyecto. Mantener este reporte y todas las herramientas que lo hacen posible requiere tiempo y recursos. Si deseas que este proyecto continúe, apoya su desarrollo realizando una donación a través de ko-fi.

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/F1F8DEV80)
