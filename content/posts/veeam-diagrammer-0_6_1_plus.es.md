---
title: 'HomeLab: Nueva version de Veeam.Diagrammer v0.6.1+'
date: '2024-09-02T00:00:00-04:00'
tags:
    - VEEAM
    - AsBuiltReport
    - Powershell
    - Diagram
---

Hola,

Hoy estaré compartiendo las mejoras que he realizado a la herramienta para diagramar la infraestructura de `Veeam Backup & Replication`. Esta herramienta utiliza [Graphviz](https://graphviz.org/) como motor para dibujar el diagrama y el módulo de [PSGraph](https://psgraph.readthedocs.io/en/latest/about/) para generar el código desde PowerShell. Aquí les dejo el enlace del proyecto en Github:

- <https://github.com/rebelinux/Veeam.Diagrammer>

Esta herramienta es utilizada dentro del reporte de `AsbuiltReport.Veeam.VBR` para generar el diagrama de los siguientes componentes:

```markdown
- VMware and Hyper-V Proxy
- Tape Infrastructure
- Backup Repositories
- SOBR repositories
- Wan Accelerators
- etc..
```

En la versión 0.6.1 se añadió al diagrama los repositorios tipo NAS (SMB y NFS) que antes de esta versión no habían sido identificados en el código.

Aquí les dejo el código añadido en esta versión: [Enlace](https://github.com/rebelinux/Veeam.Diagrammer/blob/2c7092cac1fbf90860d4dafa56a24a6b961d5660/Src/Private/Get-DiagBackupToRepo.ps1#L47):

![Image1](/img/2024/veeam.diagrammer-0.6.1plus/vscode1.webp)

En la siguiente imagen se puede ver los componentes de NAS identificados dentro del diagrama.

![Image2](/img/2024/veeam.diagrammer-0.6.1plus/diagramer0.webp)

Otra mejora al código es la detección de la versión del módulo de PowerShell de Veeam. Anteriormente, no se detectaba correctamente esta versión:

![Image3](/img/2024/veeam.diagrammer-0.6.1plus/vscode2.webp)

Dentro de `Graphviz` existe un concepto llamado `Cluster` que es utilizado para agrupar los objetos dentro del diagrama. En las versiones previas era difícil codificar un enlace entre estos `Clusters`, pero en esta nueva versión se añadió la opción de poder hacer enlaces entre estos.

#### Antes de la versión 0.6.1

![Image4](/img/2024/veeam.diagrammer-0.6.1plus/diagramer2.webp)

#### Ahora en la versión 0.6.1

![Image5](/img/2024/veeam.diagrammer-0.6.1plus/diagramer3.webp)

El último cambio que discutiré no está relacionado directamente a `Veeam.Diagrammer`, sino a una dependencia llamada `Diagrammer.Core`. En esta nueva versión se añadió soporte a la versión 0.2.3 de `Diagrammer.Core`. En esta nueva versión se añade soporte a `Graphviz` v12.1, aquí les dejo el enlace de las nuevas capacidades de esta herramienta:

- <https://gitlab.com/graphviz/graphviz/-/blob/main/CHANGELOG.md>

Para finalizar, aquí les dejo el resto de los cambios que se introdujeron o se arreglaron en la nueva versión de `Veeam.Diagrammer`:

```markdown
## [0.6.2] - 2024-08-31

### Changed

- Migrate diagrams to use Get-DiaHTMLNodeTable

## [0.6.1] - 2024-08-31

### Added

- Add support for NAS Repository (Backup-to-Repository)

### Changed

- Allow EDGE to connect between Subgraph Clusters
- Update Diagrammer.Core minimum to v0.2.3

### Fixed

- Fix veeam module version detection
```

¡Hasta la Próxima!

![Veeam Vanguard](/img/2024/abr-veeam-vbr-0_8_8/veeam_vanguard.webp#center)

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/F1F8DEV80)
