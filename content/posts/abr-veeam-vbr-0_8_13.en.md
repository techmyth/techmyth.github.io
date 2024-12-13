---
title: 'New version of AsBuiltReport.Veeam.VBR v0.8.13'
date: '2024-12-11T10:00:00-04:00'
draft: false
tags:
    - VEEAM
    - AsBuiltReport
    - Powershell
---

Hello,

Recently, I have made several changes to the `AsBuiltReport.Veeam.VBR` report, so I will summarize here all the new capabilities added.

Here is the link to the most recent report in HTML format: [Report](https://htmlpreview.github.io/?https://raw.githubusercontent.com/AsBuiltReport/AsBuiltReport.Veeam.VBR/dev/Samples/Sample%20Veeam%20Backup%20%26%20Replication%20As%20Built%20Report.html)

The first change I will discuss is the support for `Microsoft Entra ID`. In this case the `Veeam Backup & Replication (VBR)` Powershell module allows extracting the information of the `Tenants` that are configured in the VBR infrastructure.

This screen shows the `Tenant` information such as Name, IDs, Region and the configured cache repository among others.

![abr-veeam-vbr-0813-01](/img/2024/abr-veeam-vbr-0_8_13/abr-veeam-vbr-0813-01.webp)

In this part of the report the summary of the backup tasks of the `Entra ID` objects is shown.

![abr-veeam-vbr-0813-02](/img/2024/abr-veeam-vbr-0_8_13/abr-veeam-vbr-0813-02.webp)

The configuration of the `Entra ID` backup tasks is shown here.

![abr-veeam-vbr-0813-03](/img/2024/abr-veeam-vbr-0_8_13/abr-veeam-vbr-0813-03.webp)

![abr-veeam-vbr-0813-04](/img/2024/abr-veeam-vbr-0_8_13/abr-veeam-vbr-0813-04.webp)

On the next screen the `Malware Detection` table is shown where the information related to `Signature Detection` was added. The `Veeam Threat Hunter` information introduced in version 12.3 is shown here.

![abr-veeam-vbr-0813-05](/img/2024/abr-veeam-vbr-0_8_13/abr-veeam-vbr-0813-05.webp)

VBR version 12.3 adds another `Best Practice` in the `Security & Compliance` section. The following screen shows the name and its status.

![abr-veeam-vbr-0813-06](/img/2024/abr-veeam-vbr-0_8_13/abr-veeam-vbr-0813-06.webp)

Additionally, I show you the `Syslog Event Filters` table found in the `Event Forwarding` section.

![abr-veeam-vbr-0813-07](/img/2024/abr-veeam-vbr-0_8_13/abr-veeam-vbr-0813-07.webp)

In this new version of the report we added information about the repositories configured in `Google Cloud Storage`. It was on my list some time ago to add support for this type of repository...

![abr-veeam-vbr-0813-08](/img/2024/abr-veeam-vbr-0_8_13/abr-veeam-vbr-0813-08.webp)

In the new version of the report `v0.8.13` was added the possibility to diagram objects related to:

```markdown
- EntraID Tenant Information
- VMware Infrastructure Information
  - vCenter Servers
  - vSphere Clusters
    - Esxi Servers
```

Here is the diagram with the new changes.

![AsBuiltReport.Veeam.VBR](/img/2024/abr-veeam-vbr-0_8_13/AsBuiltReport.Veeam.VBR.webp)

Finally, here are the rest of the changes that were added or corrected in this new version of the report:

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

I hope all these improvements will please you!

¡Hasta la Próxima!

![Veeam Vanguard](/img/2024/abr-veeam-vbr-0_8_13/veeam_vanguard.webp#center)

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/F1F8DEV80)
