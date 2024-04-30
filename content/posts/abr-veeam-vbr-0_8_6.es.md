---
title: 'HomeLab:  Nueva versión de AsBuiltReport.Veeam.VBR v0.8.6'
date: '2024-04-29T10:50:00-04:00'
tags:
    - VEEAM
    - AsBuiltReport
    - Powershell
---

En esta ocasión estaré hablando sobre varias mejoras que he estado realizando para mejorar el reporte de AsBuiltReport.Veeam.VBR.

Una petición que me realizo un usuario fue añadir un `health check` para verificar la expiración de la cuenta de acceso al `Tenant` en `Cloud Connect`.

![cloud_connect_tenant_healthcheck](/img/2024/abr-veeam-vbr-0_8_6/clouc_connect_tenant_healthcheck.webp)

Otro aspecto que se añadió al reporte fue capturar los recursos de vCloud Director dentro de la session de Service Providers.

![service_provider_vCD](/img/2024/abr-veeam-vbr-0_8_6/service_provider_vCD.webp)

Añadir una gráfica para mostrar el espacio utilizado en los repositorio de Veeam VBR fue una de las cosas que siempre quise hacer y esta ves pude añadir esta capacidad utilizando el modulo de Powershell [PScriboCharts](https://github.com/iainbrighton/PScriboCharts)

![backup_repo_space_utiliaction_chart](/img/2024/abr-veeam-vbr-0_8_6/backup_repo_space_utiliaction_chart.webp)

En la version de 1.4.0 del modulo de AsBuiltReport.Core se definió un tema gráfico predeterminado y en esta version v0.8.6 se corrigió un Bug que no permitía que el reporte se generara. Adicionalmente se añadió una opción `Reportstyle` para que el usuario defina que tema utilizar `Veeam` o `AsBuiltReport`.

![abr_reportstyle](/img/2024/abr-veeam-vbr-0_8_6/abr_reportstyle.webp)

Adicionalmente se modifico el tema de las gráficas para dar soporte a este nuevo cambio.

![new_abr_default_theme](/img/2024/abr-veeam-vbr-0_8_6/new_abr_default_theme.webp)

Uno de los cambios que se realizo en la version 0.8.1 fue remover la session de `Infrastructure Hardening` ya que la mayoría del contenido fue integrado a través de `Health Check` distribuidos entre varias secciones del reporte. Una de las secciones que no se integro fue la identificación de parcho de Windows no instalados en el servidor de Backup. En esta version se añadió la session para validad que el servidor de Backup tiene instalado los parches mas recientes del sistema operativo.

![missing_patches](/img/2024/abr-veeam-vbr-0_8_6/missing_patches.webp)

Uno de los propósitos principales de esta nueva version fue añadir la capacidad de generar un diagrama simple de la infraestructura de `Veeam Backup & Replication`. Este diagrama que permita observar los componentes implementados en la infraestructura.

Es esta version inicial el diagrama solo muestra los siguientes componentes:

- Backup Server
  - Database Server
  - Enterprise Manager
- Proxy Servers
- Backup Repositories
  - Object Storage Repositories
  - Archive Object Storage Repositories
- Scale-Out Backup Repositories
- Wan Accelerators

Poco a poco estaré añadiendo componentes adicionales al diagrama como también información de la relación entre los componentes.

![AsBuiltReport.Veeam.VBR](/img/2024/abr-veeam-vbr-0_8_6/AsBuiltReport.Veeam.VBR.webp)

Para finalizar, aquí les dejo el resto de los cambio que se introdujeron o se arreglaron en esta nueva version de reporte:

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
