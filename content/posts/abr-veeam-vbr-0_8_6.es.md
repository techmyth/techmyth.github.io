---
title: 'Nueva versión de AsBuiltReport.Veeam.VBR v0.8.6'
date: '2024-04-29T10:50:00-04:00'
tags:
    - VEEAM
    - AsBuiltReport
    - Powershell
---

En esta ocasión estaré hablando sobre varios cambios que he estado realizando para mejorar el reporte de AsBuiltReport.Veeam.VBR.

Aquí les dejo el enlace del reporte en formato HTML: [Reporte](https://htmlpreview.github.io/?https://raw.githubusercontent.com/AsBuiltReport/AsBuiltReport.Veeam.VBR/dev/Samples/Sample%20Veeam%20Backup%20%26%20Replication%20As%20Built%20Report.html)

Una petición que me realizó un usuario fue añadir un `health check` para verificar la expiración de la cuenta de acceso al `Tenant` en `Cloud Connect`.

![cloud_connect_tenant_healthcheck](/img/2024/abr-veeam-vbr-0_8_6/clouc_connect_tenant_healthcheck.webp)

Otro aspecto que se añadió al reporte fue capturar los recursos de vCloud Director dentro de la sesión de Service Providers.

![service_provider_vCD](/img/2024/abr-veeam-vbr-0_8_6/service_provider_vCD.webp)

Añadir una gráfica para mostrar el espacio utilizado en los repositorios de `Veeam VBR` fue una de las cosas que siempre quise hacer y esta vez pude añadir esta capacidad utilizando el módulo de Powershell de [PScriboCharts](https://github.com/iainbrighton/PScriboCharts)

![backup_repo_space_utiliaction_chart](/img/2024/abr-veeam-vbr-0_8_6/backup_repo_space_utiliaction_chart.webp)

En la versión de 1.4.0 del módulo de AsBuiltReport.Core se definió un tema gráfico predeterminado y en esta versión v0.8.6 se corrigió un bug que no permitía que el reporte se generara. Adicionalmente, se añadió una opción `Reportstyle` para que el usuario defina qué tema utilizar: `Veeam` o `AsBuiltReport`.

![abr_reportstyle](/img/2024/abr-veeam-vbr-0_8_6/abr_reportstyle.webp)

Adicionalmente, se modificó el tema de las gráficas para dar soporte a este nuevo cambio.

##### New default style:

![new_abr_default_theme](/img/2024/abr-veeam-vbr-0_8_6/new_abr_default_theme.webp)

##### Veeam style:

![abr_veeam_theme](/img/2024/abr-veeam-vbr-0_8_6/abr_veeam_theme.webp)

Uno de los cambios que se realizó en la versión 0.8.1 fue remover la sesión de `Infrastructure Hardening`, ya que la mayoría del contenido fue integrado a través de `Health Check` distribuidos entre varias secciones del reporte. Una de las secciones que no se integró fue la identificación de parchos de Windows no instalados en el servidor de Backup. En esta versión se añadió la sesión para validar que el servidor de Backup tiene instalado los parches más recientes del sistema operativo.

![missing_patches](/img/2024/abr-veeam-vbr-0_8_6/missing_patches.webp)

Uno de los propósitos principales de esta nueva versión fue añadir la capacidad de generar un diagrama simple de la infraestructura de `Veeam Backup & Replication`. Este diagrama que permita observar los componentes implementados en la infraestructura.

Es esta versión inicial el diagrama solo muestra los siguientes componentes:

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

Poco a poco estaré añadiendo componentes adicionales al diagrama como también información de la relación entre los componentes.

![AsBuiltReport.Veeam.VBR](/img/2024/abr-veeam-vbr-0_8_6/AsBuiltReport.Veeam.VBR.webp)

Para finalizar, aquí les dejo el resto de los cambios que se introdujeron o se arreglaron en esta nueva versión del reporte:

```markdown
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

¡Hasta la Próxima!
