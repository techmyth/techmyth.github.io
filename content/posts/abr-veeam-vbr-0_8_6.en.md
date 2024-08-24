---
title: 'HomeLab: New Release of AsBuiltReport.Veeam.VBR v0.8.6'
date: '2024-04-29T10:50:00-04:00'
tags:
    - VEEAM
    - AsBuiltReport
    - Powershell
---

On this occasion I will be talking about several enhancements I have been making to improve the AsBuiltReport.Veeam.VBR script.

Here is the link to the report in HTML format: [Sample Report](https://htmlpreview.github.io/?https://raw.githubusercontent.com/AsBuiltReport/AsBuiltReport.Veeam.VBR/dev/Samples/Sample%20Veeam%20Backup%20%26%20Replication%20As%20Built%20Report.html)

A request made by a user was to add a `health check` to verify the expiration of the `Tenant` access account in `Cloud Connect`.

![cloud_connect_tenant_healthcheck](/img/2024/abr-veeam-vbr-0_8_6/clouc_connect_tenant_healthcheck.webp)

Another aspect that was added to the report was to capture vCloud Director resources within the Service Providers section.

![service_provider_vCD](/img/2024/abr-veeam-vbr-0_8_6/service_provider_vCD.webp)

Adding a graph to plot the space used in the Backup Repositories was one of the things I always wanted to do and this time I was able to add this feature by using the [PScriboCharts](https://github.com/iainbrighton/PScriboCharts) Powershell module.

![backup_repo_space_utiliaction_chart](/img/2024/abr-veeam-vbr-0_8_6/backup_repo_space_utiliaction_chart.webp)

In v1.4.0 of the `AsBuiltReport.Core` module a default style theme was defined. This caused a Bug that did not allow the report to be generated. In this new release the Bug was fixed and in addition a `Reportstyle` option was added to let the user define which theme to use `Veeam` or `AsBuiltReport`.

![abr_reportstyle](/img/2024/abr-veeam-vbr-0_8_6/abr_reportstyle.webp)

Additionally, the style theme for the charts was modified to support this new change.

New default style:
![new_abr_default_theme](/img/2024/abr-veeam-vbr-0_8_6/new_abr_default_theme.webp)

Veeam style:
![abr_veeam_theme](/img/2024/abr-veeam-vbr-0_8_6/abr_veeam_theme.webp)

One of the changes made in version 0.8.1 was to remove the `Infrastructure Hardening` section since most of the content was integrated through `Health Check` distributed among several sections of the report. One of the sections that was not integrated was the identification of missing Windows patches on the Backup server. In this version a section was added to validate that the Backup server has the latest operating system patches installed.

![missing_patches](/img/2024/abr-veeam-vbr-0_8_6/missing_patches.webp)

Among the main purposes of this new version was to add the ability to generate a simple diagram of the `Veeam Backup & Replication` infrastructure. This diagram allows visualizing the components implemented in the infrastructure.

In this initial release the diagram only features the following components:

- Backup Server
  - Database Server
  - Enterprise Manager
- Proxy Servers
- Backup Repositories
  - Object Storage Repositories
  - Archive Object Storage Repositories
- Scale-Out Backup Repositories
- Wan Accelerators

Gradually I will be adding additional components to the diagram as well as information on the relationship between the components.

![AsBuiltReport.Veeam.VBR](/img/2024/abr-veeam-vbr-0_8_6/AsBuiltReport.Veeam.VBR.webp)

Finally, here are the rest of the changes that were introduced or fixed in this new release of the report:

```text
## [0.8.6] - 2024-04-29

### Added

- Add Backup Infrastructure Diagram (WIP)
- Close [#155](https://github.com/AsBuiltReport/AsBuiltReport.Veeam.VBR/issues/155)
- Add vCD Resources to Service Provider section
- Add Backup Repository Space Utilization chart

### Changed

- Increase AsBuiltReport.Core modules to v1.4.0
- Migrate NOTOCHeading3 to NOTOCHeading4 to fix section heading
- Change charts palette to follow new AsBuiltReport.Core theme


### Fixed

- Fix [#149](https://github.com/AsBuiltReport/AsBuiltReport.Veeam.VBR/issues/149)
- Fix [#151](https://github.com/AsBuiltReport/AsBuiltReport.Veeam.VBR/issues/151)
- Fix [#150](https://github.com/AsBuiltReport/AsBuiltReport.Veeam.VBR/issues/150)

### Removed

- Remove EnableCharts option.
- Remove Infrastructure Charts
```

Hasta la Proxima!
