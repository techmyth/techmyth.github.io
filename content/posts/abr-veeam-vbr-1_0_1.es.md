---
title: 'Nueva versión de AsBuiltReport.Veeam.VBR v1.0.1'
date: '2026-04-22T10:00:00-04:00'
draft: false
tags:
    - VEEAM
    - AsBuiltReport
    - Powershell
---

Hola desde el Caribe 🥥🌴🌺🌅🌊,

Continuando con el desarrollo del reporte [AsBuiltReport.Veeam.VBR](https://github.com/AsBuiltReport/AsBuiltReport.Veeam.VBR), esta nueva versión `v1.0.1` trae mejoras significativas en dos áreas principales: la documentación de la infraestructura de **Alta Disponibilidad (High Availability)** de `Veeam Backup & Replication` y el **GUI tool** (`Start-AsBuiltReportVBR`).

#### Alta Disponibilidad (High Availability)

Una de las novedades más destacadas de esta versión es la incorporación de la información de configuración de **High Availability** al reporte, junto con un nuevo diagrama del clúster de Alta Disponibilidad. Esto permite documentar y visualizar de manera clara los componentes configurados para garantizar la continuidad del servicio en `Veeam Backup & Replication`.

![HA Report Section](/img/2026/abr-veeam-vbr-1_0_1/HA-Report-Section.webp#center)

#### Mejoras en el GUI tool (Start-AsBuiltReportVBR)

El GUI tool `Start-AsBuiltReportVBR`, introducido en la versión `v1.0.0`, recibe en esta versión varias mejoras que enriquecen la experiencia del usuario:

- **Programación con Windows Task Scheduler**: ahora es posible configurar la ejecución del reporte de manera automática mediante el Programador de tareas de Windows directamente desde el GUI.

![AsBuiltReport.Veeam.VBR GUI](/img/2026/abr-veeam-vbr-1_0_1/GUITaskScheduler.webp#center)

- **Botón de Verbose logging**: se añadió un botón para habilitar el registro detallado durante la generación del reporte, facilitando la resolución de problemas.
- **Acceso a la carpeta de salida**: se incorporó la opción de abrir la carpeta de destino (`OutputFolder`) directamente desde la interfaz.

![AsBuiltReport.Veeam.VBR GUI](/img/2026/abr-veeam-vbr-1_0_1/Sample-Gui.webp#center)

- **Exportación exclusiva de diagramas**: ahora es posible exportar únicamente los diagramas en diferentes formatos (PDF, PNG, SVG) sin necesidad de generar el reporte completo.

![AsBuiltReport.Veeam.VBR GUI](/img/2026/abr-veeam-vbr-1_0_1/GUIExportDiagram.webp#center)

#### Nuevo cmdlet: Export-AsBuiltReportVBRDiagram

Esta versión también introduce el cmdlet `Export-AsBuiltReportVBRDiagram`, que permite exportar los diagramas de infraestructura de forma independiente al proceso de generación del reporte. A continuación se muestra un ejemplo de uso:

```powershell
# Exportar el diagrama de infraestructura en formato PNG
$Creds = Get-Credential
Export-AsBuiltReportVBRDiagram -Target veeam-vbr.pharmax.local -Credential $Creds -Format PNG -OutputFolderPath 'C:\Users\Jon\Documents' -DiagramType 'Backup-Infrastructure'
```

Para finalizar, les incluyo el resto de los cambios que se añadieron o se corrigieron en esta nueva versión del reporte:

```markdown
## [1.0.1] - 2026-04-23

### :toolbox: Added

- Add required values to the AsBuiltReport.json configuration file for the GUI tool
- Add High Availability configuration information to the report
- Add High Availability Cluster diagram to the report
- Add the capability to schedule the report using Windows Task Schedule with the GUI tool
- Add Verbose logging button to the GUI
- Add the capability to open the OutPutFolder from the GUI
- Add the capability to only export the diagrams in different formats (PDF, PNG, SVG) from the GUI
- Add Export-AsBuiltReportVBRDiagram cmdlet to allow exporting the diagrams separately from the report generation process

### :arrows_clockwise: Changed

- Update module version to `v1.0.1`
- Update AsBuiltReport.Diagram module to `v1.0.6`
- Update the GUI tool by adding new features and improving the user experience

### :bug: Fixed

- Fix issue with the AsBuiltReport Global Configurator form not properly saving the new configuration file when using the GUI tool (Start-AsBuiltReportVBR)
```

Todos estos nuevos cambios han sido añadidos al reporte de [AsBuiltReport.Veeam.VBR](https://github.com/AsBuiltReport/AsBuiltReport.Veeam.VBR) para mejorar la documentación de la infraestructura de `Veeam Backup & Replication`.

#### Ejemplo del reporte

[AsBuiltReport.Veeam.VBR Ejemplo del reporte](https://htmlpreview.github.io/?https://raw.githubusercontent.com/AsBuiltReport/AsBuiltReport.Veeam.VBR/dev/Samples/Sample%20Veeam%20Backup%20%26%20Replication%20As%20Built%20Report.html)

¡Hasta la Próxima!
> Recientemente me han contactado para preguntar sobre el estado de este proyecto. Mantener este reporte y todas las herramientas que lo hacen posible requiere tiempo y recursos. Si deseas que este proyecto continúe, apoya su desarrollo realizando una donación a través de ko-fi.

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/F1F8DEV80)
