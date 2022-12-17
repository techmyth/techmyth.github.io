---
title: 'HomeLab – Diagrama de la Infraestructura Virtual con vDiagram'
date: '2021-08-27T07:11:00-04:00'
tags:
    - VMware
---

Hola a tod@s

Tomando como referencia la lista de [Top 10 VMware Admin Tools](https://core.vmware.com/blog/top-10-vmware-admin-tools) en esta oportunidad vengo a mostrarles como utilizar la herramienta de [vDiagram](https://github.com/Tony-SouthFLVMUG/vDiagram2.0) que dispone de la posición #6 de la lista de herramienta mas utilizadas por los administradores de infraestructuras de VMware. En esencia este «script» de Powershell captura y dibuja una infraestructura de VMware vSphere utilizando Microsoft Visio. Originalmente esta herramienta fue creada por **Alan Renouf** [@alanrenouf](https://twitter.com/alanrenouf) y actualmente el proyecto es mantenido por **Tony Gonzalez** [@vDiagram_Tony](https://twitter.com/vDiagram_Tony).

![Text](/img/invisible-infrastructure-300x200.webp#center)

Para utilizar esta herramienta es necesario cumplir con los siguientes requisitos:

1. Powershell >= 5.1
2. Modulos de PowerCLI («Install-Module -Name VMware.PowerCLI»)
3. Microsoft Visio 2013+

Una vez tengamos todos los requisitos procedemos a descargar el código de Powershell para poder obtener nuestro diagrama. Para descargar el archivo accedemos al siguiente enlace:

<https://github.com/Tony-SouthFLVMUG/vDiagram2.0>

![Text](/img/2021-08-24_09-06.webp#center)

Una vez descargamos el paquete procedemos a descomprimir el contenido.

```text
PS C:\> Expand-Archive -LiteralPath .\vDiagram2.0-master.zip -DestinationPath .
PS C:\> ls vD*


    Directory: C:\Users\jocolon\Downloads


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         8/26/2021  11:15 AM                vDiagram2.0-master
-a----         7/21/2021   9:37 AM       12470234 vDiagram2.0-master.zip


PS C:\>
```

Nos movemos a la carpeta descomprimida y validamos el contenido con el comando **«ls»** o **«dir»**.

```text
PS C:\> cd .\vDiagram2.0-master\
PS C:\vDiagram2.0-master> ls


    Directory: C:\vDiagram2.0-master


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         2/15/2021   5:29 PM                archived
-a----         2/15/2021   5:29 PM             66 .gitattributes
-a----         2/15/2021   5:29 PM           5771 README.md
-a----         2/15/2021   5:29 PM         109288 vDiagram.ico
-a----         2/15/2021   5:29 PM         673926 vDiagram_2.0.11.ps1
-a----         2/15/2021   5:29 PM         985128 vDiagram_2.0.11.vssx
-a----         2/15/2021   5:29 PM         116398 vDiagram_Scheduled_Task_2.0.11.ps1
-a----         2/15/2021   5:29 PM         254037 vDiagram_Standard.png


PS C:\vDiagram2.0-master>
```

Utilizamos el comando **«Unblock-File»** que nos permite poder ejecutar los archivos que han sido descargados desde el Internet.

```text
PS C:\vDiagram2.0-master> Unblock-File .\vDiagram_2.0.11.ps1
PS C:\vDiagram2.0-master>
```

El próximo paso seria utilizar el comando **«$PSVersionTable»** para validar la versión de Powershell instalada localmente. Repasando la sección de requisitos podemos ver que para utilizar la herramienta de vDiagram necesitamos tener una versión de Powershell 5.1.x o mayor. En este ejemplo podemos ver que mi computadora tiene la versión **«5.1.19041.1151«**.

```text
PS C:\vDiagram2.0-master> $PSVersionTable

Name                           Value
----                           -----
PSVersion                      5.1.19041.1151
PSEdition                      Desktop
PSCompatibleVersions           {1.0, 2.0, 3.0, 4.0...}
BuildVersion                   10.0.19041.1151
CLRVersion                     4.0.30319.42000
WSManStackVersion              3.0
PSRemotingProtocolVersion      2.3
SerializationVersion           1.1.0.1


PS C:\vDiagram2.0-master>
```

Adicionalmente validamos que el módulo de PowerCLI esté instalado utilizando el comando **«Get-Module»**.

```text
PS C:\vDiagram2.0-master> Get-Module -ListAvailable -Name 'VMware.PowerCLI' | Sort-Object -Property Version -Descending | Select-Object -First 1


    Directory: C:\Program Files\WindowsPowerShell\Modules


ModuleType Version    Name                                ExportedCommands
---------- -------    ----                                ----------------
Manifest   12.3.0.... VMware.PowerCLI


PS C:\vDiagram2.0-master> 
```

Luego de validar todos los requisitos podemos ejecutar el «script» utilizando el archivo principal **".\vDiagram_2.0.11.ps1″**.

```text
PS C:\vDiagram2.0-master> 

PS C:\vDiagram2.0-master> .\vDiagram_2.0.11.ps1
[08/24/2021 10:45:16] VMware PowerCLI Module(s) 12.3.0.17860403 11.5.0.14912921  found on this machine.


```

Una vez el programa termine de ejecutar podemos ver en la pestaña de **«Prerequisites»** un resumen de todas las dependencias y su estado. En este ejemplo podemos ver que todas las dependencias se muestran en color verde que nos indica que están instaladas.

![Text](/img/2021-08-24_10-50.webp#center)

Siguiendo los pasos según se muestra en la pestaña de **«Directions»** es necesario ingresar la dirección de **IP/FQDN** del vCenter y las credenciales con privilegios que permita conectarse y extraer la información.

![Text](/img/2021-08-24_10-51.webp#center)

Une vez completada la información del vCenter procedemos a validar la conexión presionando el botón de **«Connect to vCenter»**. Como se puede apreciar en la siguiente imagen el botón cambia a color verde indicando que hubo una conexión exitosa al vCenter con las credenciales provistas.

![Text](/img/2021-08-24_10-51_1.webp#center)

El próximo paso seria seleccionar la pestaña de **«Capture CSVs for Visio»** y especificar la carpeta donde se grabaran los reportes con la información temporera para generar el diagrama. En este ejemplo utilice la carpeta **<Desktop/Output>**.

![Text](/img/2021-08-24_10-54-1.webp#center)

Es importante mencionar que por cada valor seleccionado se generará un archivo con la información del respectivo elemento.

![Text](/img/2021-08-24_10-55_1.webp#center)

Luego procedemos a presionar **«Collect CSV Data»** para iniciar el proceso de recopilación de los datos.

![Text](/img/2021-08-24_10-56-1.webp#center)

Una vez finalizado el proceso de recopilación de los datos podemos seleccionar la pestaña de **«Draw Visio»** y configurar la opción de **«Select CSV Input Folder»**.

![Text](/img/2021-08-24_16-08-2.webp#center)

Luego seleccionas la carpeta donde están los datos previamente recopilados. En mi ejemplo sería la carpeta **<Desktop/Output>** que utilice en la sección de **«Capture CSVs for Visio»**.

![Text](/img/2021-08-24_16-09-1.webp#center)

Ahora es necesario validar que la información necesaria para generar el diagrama esta disponible al oprimir el botón **«CSV Validation Complete»**.

![Text](/img/2021-08-24_16-10-2.webp#center)

En este próximo paso es necesario especificar la carpeta donde se guardará el diagrama una vez generado. Para esto presionamos **«Select Visio Output Folder»** y seleccionamos la carpeta que se utilizara para este propósito. En este ejemplo se seleccionó **<Desktop/Output>**.

![Text](/img/2021-08-24_16-12.webp#center)

En la área de **«Visio Output Folder»** podemos selecionar la multiples opciones disponibles para generar el diagrama. Luego de seleccionar la carpeta de «Output» podemos comenzar a generar el diagrama presionando el botón **«Draw Visio»**.

![Text](/img/2021-08-24_16-14-1.webp#center)

En este paso del ejemplo presionamos **«OK»** en la notificación donde nos indica que el diagrama fue creado.

![Text](/img/2021-08-24_16-26-1.webp#center)

Para ver el diagrama generado sobre nuestra infraestructura virtual presionamos el botón **«Open Visio Diagram»**.

![Text](/img/2021-08-24_16-26_1-1.webp#center)

Finalmente les dejo varias imágenes de ejemplo sobre los diagramas de mi infraestructura virtual **«HomeLab»**.

![Text](/img/VM-To-Host-1-1024x560.webp#center)

### Resumen

En este laboratorio utilizamos la herramienta llamada vDiagram que nos permite hacer un representación lógica de como esta relacionado los componentes de nuestra infraestructura virtual. Algo bueno de esta herramienta es que está disponible de forma gratuita. Espero que este laboratorio les haya gustado. Si tienes dudas o alguna pregunta sobre este laboratorio, déjalo en los comentarios. Hasta Luego!.
