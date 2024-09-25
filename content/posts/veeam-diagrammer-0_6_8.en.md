---
title: 'New version of Veeam.Diagrammer v0.6.8'
date: '2024-09-24T00:00:00-04:00'
tags:
    - VEEAM
    - AsBuiltReport
    - Powershell
    - Diagram
---

Hola,

Today I am going to share the improvements I have made to the `Veeam Backup & Replication` infrastructure diagramming tool. This tool uses [Graphviz](https://graphviz.org/) as the engine to draw the diagram and the [PSGraph](https://psgraph.readthedocs.io/en/latest/about/) module to generate the code from PowerShell. Here is the link to the project on GitHub:

- <https://github.com/rebelinux/Veeam.Diagrammer>

In version 0.6.8 information about `SureBackup` was added to the infrastructure diagram. In particular, the ability to diagram `Application Groups` and `Virual Labs` has been added.

![SureBackup](/img/2024/veeam.diagrammer-0.6.8/SureBackup.webp)

Here is the code added in this release: [Enlace](https://github.com/rebelinux/Veeam.Diagrammer/blob/455bb8b4ff42a7b2ddb6672a5d9d5eee9122fd76/Src/Private/Get-VbrInfraDiagram.ps1#L288):

In the following image you can see the `SureBackup` components identified within the diagram.

![AsBuiltReport.Veeam.VBR](/img/2024/veeam.diagrammer-0.6.8/AsBuiltReport.Veeam.VBR.webp)

The last change I will discuss is related to the ability to set the diagram themes. In this case 2 additional themes were added:

#### - DiagramTheme Neon

![AsBuiltReport.Veeam.VBR.Neon](/img/2024/veeam.diagrammer-0.6.8/AsBuiltReport.Veeam.VBR_neon.webp)

#### - DiagramTheme Black

![AsBuiltReport.Veeam.VBR.Black](/img/2024/veeam.diagrammer-0.6.8/AsBuiltReport.Veeam.VBR_black.webp)

Finally, here are the rest of the changes that were introduced or fixed in the new release:

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
