---
title: 'New Release: AsBuiltReport.Veeam.VBR v0.8.22'
date: '2025-07-22T10:00:00-04:00'
draft: false
tags:
    - VEEAM
    - AsBuiltReport
    - Powershell
---

Hello from the Caribbean 🥥🌴🌺🌅🌊,

Since stepping away from the `Veeam Vanguard` program, I’ve been somewhat distant from the development of the [AsBuiltReport.Veeam.VBR](https://github.com/AsBuiltReport/AsBuiltReport.Veeam.VBR) report. However, last month I dedicated time to enhancing the [Veeam.Diagrammer](https://github.com/rebelinux/Veeam.Diagrammer) module, adding new features to generate diagrams of Cloud Connect infrastructure and the resources used by `Tenants` in `Veeam Backup & Replication`.

Thanks to these improvements in Veeam.Diagrammer, it’s now possible to generate detailed diagrams of the Cloud Connect infrastructure and its components. The following image showcases the elements configured within `Veeam Backup & Replication`.

![abr-veeam-vbr-0813-01](/img/2025/abr-veeam-vbr-0_8_22/CloudConnectInfra.webp)

Another key feature in this release is the ability to generate tenant-specific diagrams, providing a clear visualization of the resources used within the infrastructure. With `Veeam.Diagrammer`, you can now extract detailed information about replica resources (vSphere, VCloud Director, and Hyper-V), backup resources, SubTenants, WAN Accelerators, and network devices. This makes it much easier to analyze and manage each tenant’s environment.

#### Examples of Tenant Diagrams

![abr-veeam-vbr-0813-01](/img/2025/abr-veeam-vbr-0_8_22/CloudConnect-Tenant2.webp)

![abr-veeam-vbr-0813-01](/img/2025/abr-veeam-vbr-0_8_22/CloudConnect-Tenant3.webp)

> **Note:** To generate tenant-specific diagrams, set the Tenant [InfoLevel](https://github.com/AsBuiltReport/AsBuiltReport.Veeam.VBR?tab=readme-ov-file#infolevel) option to `2`.

Below is the main diagram of the `Veeam` infrastructure, now updated to include the new Cloud Connect components. This provides a more comprehensive and detailed view of the environment.

![AsBuiltReport.Veeam.VBR](/img/2025/abr-veeam-vbr-0_8_22/AsBuiltReport.Veeam.VBR.webp)

To conclude, here are the additional changes and fixes included in this new version of the report:

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

All these new changes in [Veeam.Diagrammer](https://github.com/rebelinux/Veeam.Diagrammer) have been added to the [AsBuiltReport.Veeam.VBR](https://github.com/AsBuiltReport/AsBuiltReport.Veeam.VBR) report to generate diagrams of the `Veeam Backup & Replication` infrastructure.

¡Hasta la Próxima!

> I have recently been contacted to ask about the status of this project. Maintaining this report and all the tools that make this project work is time and resource consuming. If you want to keep this project alive, support its development by donating through ko-fi.

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/F1F8DEV80)
