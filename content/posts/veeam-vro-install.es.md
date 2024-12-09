---
title: 'Veeam: Instalación de Recovery Orchestrator'
date: '2024-12-03T00:00:00-00:00'
author: 'Jonathan Colon'
tags:
    - VEEAM
    - Powershell
    - Recovery Orchestration
---

Hola a todos,

En esta ocasión estaré mostrando como realizar una instalación básica del producto `Veeam Recovery Orchestrator (VRO)`. Esta aplicación es una alternativa viable al `Site Recovery Manager` de `VMware`, pero en mi opinión el VRO tiene mayores capacidades!

Los archivos de instalación pueden ser descargados desde la página Web de Veeam. Para este laboratorio estaré utilizando la versión de prueba de 30 días.

Pueden utilizar el siguiente enlace para registrarse: [Recovery Orchestrator Trial](https://www.veeam.com/products/veeam-data-platform/recovery-orchestration.html)

![vro_install_00](/img/2024/vro_install/vro_install_00.webp#center)

Aquí le dejo el enlace del portal de documentación para que puedan validar los requerimientos de sistema.

[Requerimientos del sistema](https://helpcenter.veeam.com/docs/vro/userguide/system_requirements.html?ver=70)

En mi caso estaré utilizando los requisitos mínimos para poder correr esta solución en mi servidor local.

```powershell
PS /home/rebelinux> Get-VM -Name VEEAM-DRO-01V | Select-Object -Property Name,NumCpu,MemoryGB,ProvisionedSpaceGB

Name NumCpu MemoryGB ProvisionedSpaceGB
---- ------ -------- ------------------
VEEAM-DRO-01V      2    8.000            204.263

PS /home/rebelinux> 
```

Una vez tenemos acceso a los archivos de instalación procedemos a ejecutar el archivo `Setup.exe`.

![vro_install_01](/img/2024/vro_install/vro_install_01.webp#center)

Para comenzar el proceso debemos presionar el botón de `Install`.

![vro_install_02](/img/2024/vro_install/vro_install_02.webp#center)

En esta pantalla escogemos la opción de instalar `Veeam Recovery Orchestrator 7.1`

![vro_install_03](/img/2024/vro_install/vro_install_03.webp#center)

Leemos y Aceptamos la licencia...

![vro_install_05](/img/2024/vro_install/vro_install_05.webp#center)

En esta pantalla podemos seleccionar los productos a instalar.

![vro_install_06](/img/2024/vro_install/vro_install_06.webp#center)

Identificamos que método utilizaremos para proveer la licencia.

![vro_install_07](/img/2024/vro_install/vro_install_07.webp#center)

Seleccionamos la licencia a utilizar.

![vro_install_08](/img/2024/vro_install/vro_install_08.webp#center)

Una vez se identifica la licencia podemos continuar con los próximos pasos de instalación.

![vro_install_09](/img/2024/vro_install/vro_install_09.webp#center)

En la siguiente pantalla seleccionamos la cuenta con privilegios de administrador local en el servidor donde instalaremos la aplicación

![vro_install_10](/img/2024/vro_install/vro_install_10.webp#center)

El próximo paso es la configuración del servidor de SQL. En mi caso será instalado en el mismo servidor de VRO, pero si tiene un servidor dedicado a SQL puedes especificar la configuración remota de la instancia de SQL que se utilizara para este propósito.

![vro_install_10_2](/img/2024/vro_install/vro_install_10_2.webp#center)

En esta etapa del proceso se identifican los potenciales problemas o prerrequisitos necesarios para poder continuar!

![vro_install_11](/img/2024/vro_install/vro_install_11.webp#center)

En esta pantalla podemos validar la configuración establecida. ¡Si todo cumple con nuestros requerimientos podemos continuar con la instalación!

![vro_install_12](/img/2024/vro_install/vro_install_12.webp#center)

En esta etapa nos tomamos un café y esperamos a que el proceso de instalación finalice.

![vro_install_13](/img/2024/vro_install/vro_install_13.webp#center)

Una vez finalizado el proceso de instalación presionamos `Finish`.

![vro_install_14](/img/2024/vro_install/vro_install_14.webp#center)

En mi caso me recomienda reiniciar el servidor para que se pueda aplicar la configuración.

![vro_install_15](/img/2024/vro_install/vro_install_15.webp#center)

Luego de que el servidor termine de inicializar podemos acceder al portal de manejo de la aplicación utilizando el icono que fue creado en el `Desktop`.

![vro_install_16](/img/2024/vro_install/vro_install_16.webp#center)

Para finalizar les presento el porta de manejo. ¡En esta pantalla nos pide realizar una serie de pasos para configurar el servicio, pero dejaremos este proceso para un próximo laboratorio en el futuro!

![vro_install_17](/img/2024/vro_install/vro_install_17.webp#center)

Espero que este laboratorio te haya gustado y que sea de utilidad en su jornada profesional.

### Hasta Luego Amigos

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/F1F8DEV80)
