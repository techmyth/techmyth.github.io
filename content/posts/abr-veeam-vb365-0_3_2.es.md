---
title: 'HomeLab: Nueva version del reporte de AsBuiltReport.Veeam.VB365 v0.3.2'
date: '2024-05-27T12:00:00-04:00'
tags:
    - VEEAM
    - AsBuiltReport
    - Powershell
---

Hola,

Hace unos meses, trabajé varios cambios al reporte de `AsBuiltReport.Veeam.VB365` para añadir soporte a los `Best Practice` que fueron creados por el grupo 4 del evento de `Veeam Community Hackathon 2023`.

&nbsp;

![Image1](/img/2024/abr-veem-vb365-0_3_2/vb365_bp_00.webp#center)

&nbsp;

El equipo 4 creo varias verificaciones como mejores prácticas sobre la implementación de `Veeam Backup for Microsoft 365`, puedes ver el video del evento en este enlace:

&nbsp;

{{< youtube rhIiILD2z1I >}}

&nbsp;

En la versión `v0.3.2` se añadió la mayoría de los `Best Practice`, pero en este artículo estaré hablando de los más significativos.

En esta imagen se muestra la verificación de dos mejores prácticas:

1. Se valida que el portal de restauración este habilitado.
2. Se identifica que el certificado del Portal de Restauración es de tipo `Self-Signed`. También se genera un `HealthCheck` similar al de la imagen.

![Image1](/img/2024/abr-veem-vb365-0_3_2/vb365_bp_01.webp)

Otra mejor práctica es la configuración de las notificaciones por correo electrónico. Como se muestra en la siguiente imagen, se genera un `HealthCheck` donde se muestra la razón para este mensaje.

![Image2](/img/2024/abr-veem-vb365-0_3_2/vb365_bp_02.webp)

El próximo `Best Practice` que discutiremos es si el servidor `Proxy` está ligado a un dominio de `Active Directory (AD)`. Como se presenta en esta imagen el servidor pertenece a un dominio de `AD` y por consiguiente se genera un `HealthCheck`. Está mejor práctica sigue las recomendaciones del portal de Veeam BP.

- <https://bp.veeam.com/vb365/guide/design/hardening/Workgroup_or_Domain.html>

![Image3](/img/2024/abr-veem-vb365-0_3_2/vb365_bp_03.webp)

Las próximas dos imágenes están relacionadas a los repositorios donde se valida la utilización de cifrado e inmutabilidad.

- <https://bp.veeam.com/vb365/guide/design/hardening/Repo_specifics.html>

![Image4](/img/2024/abr-veem-vb365-0_3_2/vb365_bp_04.webp)

![Image5](/img/2024/abr-veem-vb365-0_3_2/vb365_bp_05.webp)

Relacionado a las `Organizaciones` se valida si están utilizando `Secure Sockets Layer (SSL)` como también si la `Organización` ha sido previamente resguardada.

![Image6](/img/2024/abr-veem-vb365-0_3_2/vb365_bp_06.webp)

Este último `Best Practice` fue un poco difícil de implementar, pero ahora podemos validar que los certificados no estén próximos a expirar. Según se muestra en la siguiente imagen el `HealthCheck` se configuró para validar certificados con fecha de expiración en los próximos 90 días.

![Image7](/img/2024/abr-veem-vb365-0_3_2/vb365_bp_07.webp)

Para finalizar, aquí les dejo el resto de los cambios que se introdujeron o se arreglaron en la nueva versión del reporte:

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

Quiero añadir que este reporte cuenta con la opción de crear un diagrama básico de la infraestructura de `Veeam Backup for Microsoft 365`. ¿No les parece asombroso? Yay!!!!

![Image8](/img/2024/abr-veem-vb365-0_3_2/vb365_bp_08.webp)

Aquí les dejo el enlace del reporte en formato HTML: [Reporte](https://htmlpreview.github.io/?https://raw.githubusercontent.com/AsBuiltReport/AsBuiltReport.Veeam.VB365/dev/Samples/Sample%20Veeam%20VB365%20As%20Built%20Report.html)

¡Hasta la Próxima!

![Veeam Vanguard](/img/2024/abr-veeam-vbr-0_8_8/veeam_vanguard.webp#center)

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/F1F8DEV80)
