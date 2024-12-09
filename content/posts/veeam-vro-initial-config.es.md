---
title: 'Veeam: Configuración inicial de Recovery Orchestrator'
date: '2024-12-05T00:00:00-00:00'
author: 'Jonathan Colon'
draft: false
tags:
    - VEEAM
    - Powershell
    - Recovery Orchestration
---

Hola a todos,

En la [parte 1]({{< ref "/posts/veeam-vro-install" >}} "Instalación de Recovery Orchestrator") de esta serie de laboratorios sobre `Veeam Recovery Orchestrator (VRO)` les mostré como realizar una instalación básica del producto. En esta ocasión estaré mostrando como realizar la configuración inicial de VRO.

Una vez accedemos el portal de VRO y hacemos `Login` se nos presenta un una serie de pasos para configurar la aplicación.

![vro_install_00](/img/2024/vro_initial_config/vbo_initial_config_00.webp)

En la pantalla inicial del `Initial Configuration Wizard` procedemos a presionar `Next`.

![vro_install_01](/img/2024/vro_initial_config/vbo_initial_config_01.webp)

Luego añadimos una cuenta para que pueda administrar la plataforma.

![vro_install_02](/img/2024/vro_initial_config/vbo_initial_config_02.webp)

En esta pantalla especificamos la cuenta de dominio en el formato de `Domain\Username` y sus credenciales.

![vro_install_03](/img/2024/vro_initial_config/vbo_initial_config_03.webp)

Aquí presionamos `Add` para añadir la cuenta.

![vro_install_04](/img/2024/vro_initial_config/vbo_initial_config_04.webp)

Una vez se añaden todas las cuentas para administrar VRO presionamos `Apply`.

![vro_install_05](/img/2024/vro_initial_config/vbo_initial_config_05.webp)

Luego especificamos los detalles del servidor de VRO. Esta información se utiliza en los reportes.

![vro_install_06](/img/2024/vro_initial_config/vbo_initial_config_06.webp)

Finalizado este proceso presionamos `Next`.

![vro_install_07](/img/2024/vro_initial_config/vbo_initial_config_07.webp)

El próximo paso es instalar el agente de VRO en el servidor de `Veeam Backup & Replication (VBR)`. Para esto presionamos `Deploy Orchestrator agent`.

![vro_install_08](/img/2024/vro_initial_config/vbo_initial_config_08.webp)

En esta pantalla presionamos `Add`.

![vro_install_09](/img/2024/vro_initial_config/vbo_initial_config_09.webp)

Luego especificamos los parámetros de conexión del servidor de VBR. Si es necesario creamos una nueva credencial para autenticar al servidor de VBR. Finalmente presionamos `Add` para añadir el servidor.

![vro_install_10](/img/2024/vro_initial_config/vbo_initial_config_10.webp)

Una ves hayamos añadido el servidor de VBR presionamos `Apply`.

![vro_install_11](/img/2024/vro_initial_config/vbo_initial_config_11.webp)

El próximo paso es añadir la infraestructura de VMware presionando `Connect vCenter Server`

![vro_install_12](/img/2024/vro_initial_config/vbo_initial_config_12.webp)

En esta pantalla presionamos `Add`.

![vro_install_13](/img/2024/vro_initial_config/vbo_initial_config_13.webp)

Luego especificamos los parámetros de conexión del servidor de vCenter.

![vro_install_14](/img/2024/vro_initial_config/vbo_initial_config_14.webp)

Si es necesario creamos una nueva credencial para autenticar al servidor.

![vro_install_15](/img/2024/vro_initial_config/vbo_initial_config_15.webp)

![vro_install_16](/img/2024/vro_initial_config/vbo_initial_config_16.webp)

Finalmente presionamos `Add` para añadir el servidor.

![vro_install_17](/img/2024/vro_initial_config/vbo_initial_config_17.webp)

Una ves hayamos añadido el servidor de vCenter presionamos `Apply` para guardar los cambios.

![vro_install_18](/img/2024/vro_initial_config/vbo_initial_config_18.webp)

En un laboratorio futuro añadiremos un `Storage System` pero para propósitos de este laboratorio presionamos `Next`.

![vro_install_19](/img/2024/vro_initial_config/vbo_initial_config_19.webp)

En esta pantalla podemos validar los parámetros que utilizamos para el proceso inicial de configuración y presionamos `Finish` para guardar los cambios y finalmente configura el servidor de VRO!

![vro_install_20](/img/2024/vro_initial_config/vbo_initial_config_20.webp)

Finalmente, podemos ver la pantalla de `Overview` que refleja la infraestructura que añadimos utilizando el `Initial Configuration Wizard`.

![vro_install_21](/img/2024/vro_initial_config/vbo_initial_config_21.webp)

Espero que este laboratorio te haya gustado y que sea de utilidad en su jornada profesional.

### Hasta Luego Amigos

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/F1F8DEV80)
