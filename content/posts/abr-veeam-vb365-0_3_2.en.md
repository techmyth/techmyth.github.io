---
title: 'New version release of the AsBuiltReport.Veeam.VB365 report v0.3.2'
date: '2024-05-27T12:00:00-04:00'
tags:
    - VEEAM
    - AsBuiltReport
    - Powershell
---

Hola,

A few months ago, I worked on several changes I made to the `AsBuiltReport.Veeam.VB365` report to add support for the `Best Practice` that was created by the group 4 of the `Veeam Community Hackathon 2023` event.

&nbsp;

![Image1](/img/2024/abr-veem-vb365-0_3_2/vb365_bp_00.webp#center)

&nbsp;

Team 4 created several recommendations as best practices on the implementation of `Veeam Backup for Microsoft 365`, you can watch the video of the event at this link:

&nbsp;

{{< youtube rhIiILD2z1I >}}

&nbsp;

In version `v0.3.2` most of the `Best Practice` were added, but in this article I will be talking about the most significant ones.

The first image shows the verification of two best practices:

1. The `Restore Portal` is validated to ensure that it is properly enabled.
2. It identifies that the certificate of the `Restore Portal` is of type `Self-Signed`. If so a `HealthCheck` similar to the one in the image is generated.

![Image1](/img/2024/abr-veem-vb365-0_3_2/vb365_bp_01.webp)

Another best practice is to validate if the e-mail notifications were set up. As shown in the following image, a `HealthCheck` is generated showing the reason for this warning.

![Image2](/img/2024/abr-veem-vb365-0_3_2/vb365_bp_02.webp)

The next `Best Practice` I will discuss is, if the `Proxy` server is bound to an `Active Directory (AD)` domain. As presented in the next image the server belongs to an `AD` domain and therefore a `HealthCheck` is generated. This best practice follows the recommendations of the Veeam Best Practice portal.

- <https://bp.veeam.com/vb365/guide/design/hardening/Workgroup_or_Domain.html>

![Image3](/img/2024/abr-veem-vb365-0_3_2/vb365_bp_03.webp)

The next two images are related to the repositories where the use of encryption and immutability is validated.

- <https://bp.veeam.com/vb365/guide/design/hardening/Repo_specifics.html>

![Image4](/img/2024/abr-veem-vb365-0_3_2/vb365_bp_04.webp)

![Image5](/img/2024/abr-veem-vb365-0_3_2/vb365_bp_05.webp)

In relation to `Organizations` it is validated if they are using `Secure Sockets Layer (SSL)` as well as if the `Organization` objects has been previously backedup.

![Image6](/img/2024/abr-veem-vb365-0_3_2/vb365_bp_06.webp)

This last `Best Practice` was a bit difficult to implement, but now the report can validate that the SSL certificates are not close to expiring. As shown in the following image the `HealthCheck` was configured to validate certificates with expiration date in the next 90 days.

![Image7](/img/2024/abr-veem-vb365-0_3_2/vb365_bp_07.webp)

Finally, here are the rest of the changes that were introduced or fixed in the new version of the report:

```markdown
## [0.3.2] - 2024-05-25

### Changed

- Move 'Licensed Users' section to InfoLevel 2

### Fixed

- Fix [#23](https://github.com/AsBuiltReport/AsBuiltReport.Veeam.VB365/issues/23)
- Fix [#24](https://github.com/AsBuiltReport/AsBuiltReport.Veeam.VB365/issues/24)
- Fix [#27](https://github.com/AsBuiltReport/AsBuiltReport.Veeam.VB365/issues/27)
- Fix [#28](https://github.com/AsBuiltReport/AsBuiltReport.Veeam.VB365/issues/28)
- Fix [#29](https://github.com/AsBuiltReport/AsBuiltReport.Veeam.VB365/issues/29)
- Fix [#30](https://github.com/AsBuiltReport/AsBuiltReport.Veeam.VB365/issues/30)
- Fix [#31](https://github.com/AsBuiltReport/AsBuiltReport.Veeam.VB365/issues/31)
```

I want to add that this report has the option to create a basic diagram of the `Veeam Backup for Microsoft 365` infrastructure. Don't you think it's amazing? Yay!!!!

![Image8](/img/2024/abr-veem-vb365-0_3_2/vb365_bp_08.webp)

Here is the link to the report in HTML format: [Report](https://htmlpreview.github.io/?https://raw.githubusercontent.com/AsBuiltReport/AsBuiltReport.Veeam.VB365/dev/Samples/Sample%20Veeam%20VB365%20As%20Built%20Report.html)

¡Hasta la Próxima!

![Veeam Vanguard](/img/2024/abr-veem-vb365-0_3_2/veeam_vanguard.webp#center)

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/F1F8DEV80)
