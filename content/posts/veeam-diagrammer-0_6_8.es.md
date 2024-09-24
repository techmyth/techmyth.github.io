---
title: 'Nueva version de Veeam.Diagrammer v0.6.8'
date: '2024-09-25T00:00:00-04:00'
tags:
    - VEEAM
    - AsBuiltReport
    - Powershell
    - Diagram
---

Hola,

Hoy estaré compartiendo las mejoras que he realizado a la herramienta para diagramar la infraestructura de `Veeam Backup & Replication`. Esta herramienta utiliza [Graphviz](https://graphviz.org/) como motor para dibujar el diagrama y el módulo de [PSGraph](https://psgraph.readthedocs.io/en/latest/about/) para generar el código desde PowerShell. Aquí les dejo el enlace del proyecto en GitHub:

- <https://github.com/rebelinux/Veeam.Diagrammer>

En la versión 0.6.8 se añadió al diagrama de la infraestructura la información sobre `SureBackup` específicamente sobre los `Application Groups` y los `Virual Labs`.

![SureBackup](/img/2024/veeam.diagrammer-0.6.8/SureBackup.webp)

Aquí les dejo el código añadido en esta versión: [Enlace](https://github.com/rebelinux/Veeam.Diagrammer/blob/455bb8b4ff42a7b2ddb6672a5d9d5eee9122fd76/Src/Private/Get-VbrInfraDiagram.ps1#L288):

En la siguiente imagen se puede ver los componentes de `SureBackup` identificados dentro del diagrama.

![AsBuiltReport.Veeam.VBR](/img/2024/veeam.diagrammer-0.6.8/AsBuiltReport.Veeam.VBR.webp)

El último cambio que discutiré está relacionado la capacidad de establecer temas gráficos. En este caso se añadieron 2 temas adicionales al Blanco como lo son:

#### - DiagramTheme Black

![AsBuiltReport.Veeam.VBR.Neon](/img/2024/veeam.diagrammer-0.6.8/AsBuiltReport.Veeam.VBR_neon.webp)

#### - DiagramTheme Neon

![AsBuiltReport.Veeam.VBR.Black](/img/2024/veeam.diagrammer-0.6.8/AsBuiltReport.Veeam.VBR_black.webp)

Para finalizar, aquí les dejo el resto de los cambios que se introdujeron o se arreglaron en la nueva versión de `Veeam.Diagrammer`:

- <https://gitlab.com/graphviz/graphviz/-/blob/main/CHANGELOG.md>

```markdown
## [0.6.8] - 2024-09-22

### Added

- Add diagram theme (Black/White/Neon)
- Add SureBackup support
  - Application Group
  - Virtual Lab

### Fixed

- Fix logic on Backup Server component detection (Backup, Database & EM Server)
```

¡Hasta la Próxima!

![Veeam Vanguard](/img/2024/abr-veeam-vbr-0_8_8/veeam_vanguard.webp#center)

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/F1F8DEV80)
