---
title: 'HomeLab: New Release of AsBuiltReport.Veeam.VBR v0.8.8'
date: '2024-08-24T10:00:00-04:00'
tags:
    - VEEAM
    - AsBuiltReport
    - Powershell
---

Hola,

Last month I added several enhancements to the `AsBuiltReport.Veeam.VBR` report to improve its content as well as adding more infrastructure objects to the diagram.

Here is the link to the report in HTML format: [Report](https://htmlpreview.github.io/?https://raw.githubusercontent.com/AsBuiltReport/AsBuiltReport.Veeam.VBR/dev/Samples/Sample%20Veeam%20Backup%20%26%20Replication%20As%20Built%20Report.html)

In version `v0.8.6` the ability to generate a simple diagram of the `Veeam Backup & Replication` infrastructure was added.

In the last version of the diagram only the following components are shown:

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

In the new version of the report `v0.8.8` was added the ability to display objects related to:

```markdown
- Tape Infrastructure
  - Tape Server
  - Tape Library
  - Tape Vault
- Service Provider 
```

Here I show you the diagram with the new changes, isn't it amazing! Yay....

![AsBuiltReport.Veeam.VBR](/img/2024/abr-veeam-vbr-0_8_8/AsBuiltReport.Veeam.VBR.webp)

To conclude, here are the rest of the changes that were introduced or fixed in this new version of the report:

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
