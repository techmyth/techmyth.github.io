---
title: 'Veeam.Diagrammer v0.6.1+ new version features'
date: '2024-09-02T00:00:00-04:00'
tags:
    - VEEAM
    - AsBuiltReport
    - Powershell
    - Diagram
---

Hola,

I will be sharing today the improvements I have made to the `Veeam Backup & Replication` infrastructure diagramming tool. This tool uses [Graphviz](https://graphviz.org/) as the engine to draw the diagram and [PSGraph](https://psgraph.readthedocs.io/en/latest/about/) module to generate the code from PowerShell. Here is the link to the project on GitHub.

- <https://github.com/rebelinux/Veeam.Diagrammer>

This tool is used within the [AsbuiltReport.Veeam.VBR](https://htmlpreview.github.io/?https://raw.githubusercontent.com/AsBuiltReport/AsBuiltReport.Veeam.VBR/dev/Samples/Sample%20Veeam%20Backup%20%26%20Replication%20As%20Built%20Report.html) report to generate the diagram of the following components:

```markdown
- VMware and Hyper-V Proxy
- Tape Infrastructure
- Backup Repositories
- SOBR repositories
- Wan Accelerators
- etc..
```

In version 0.6.1 the NAS type repositories (SMB and NFS) were added to the diagram, which before this version had not been defined in the code.

Here is the code added in this version: [Link](https://github.com/rebelinux/Veeam.Diagrammer/blob/2c7092cac1fbf90860d4dafa56a24a6b961d5660/Src/Private/Get-DiagBackupToRepo.ps1#L47):

![Image1](/img/2024/veeam.diagrammer-0.6.1plus/vscode1.webp)

The following image shows the NAS components identified in the diagram.

![Image2](/img/2024/veeam.diagrammer-0.6.1plus/diagramer0.webp)

Another improvement to the code is the detection of the Veeam PowerShell module version. Previously, this version was not correctly identified:

![Image3](/img/2024/veeam.diagrammer-0.6.1plus/vscode2.webp)

Within `Graphviz` there is a concept called `Cluster` that is used to group objects within the diagram. In previous versions it was difficult to code a link between these `Clusters`, but in this new version the option to make links between them were added.

#### Before v0.6.1

![Image4](/img/2024/veeam.diagrammer-0.6.1plus/diagramer2.webp)

#### After v0.6.1

![Image5](/img/2024/veeam.diagrammer-0.6.1plus/diagramer3.webp)

The last change I will discuss is not directly related to `Veeam.Diagrammer`, but to a dependency called `Diagrammer.Core`. In this new version support for `Diagrammer.Core` version 0.2.3 was added. In this new version support for `Graphviz` v12.1 is added, here is the link to the new capabilities of this tool:

- <https://gitlab.com/graphviz/graphviz/-/blob/main/CHANGELOG.md>

Lastly, here are the rest of the changes that were introduced or fixed in the new version of `Veeam.Diagrammer`:

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
