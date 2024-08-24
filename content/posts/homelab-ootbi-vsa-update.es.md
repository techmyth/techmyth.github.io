---
title: 'HomeLab: Actualización de Object First OOTBI VSA'
date: '2024-08-23T12:10:00-04:00'
tags:
    - OOTBI
    - ObjectFirst
    - Veeam
---

Hola a tod@s

En esta oportunidad vengo a mostrarles cómo actualizar la versión de la unidad de almacenamiento de la compañía `Object First`. Anteriormente, les mostramos cómo realizar la implementación inicial de este producto. Pueden acceder el artículo [aquí]({{< ref "homelab-ootbi-vsa-initial-setup.es.md" >}} "aquí")

Para comenzar, desde el portal de vCenter ubicamos la instancia de la VM para tener acceso al "Text User Interface" (TUI) de `OOTBI`.

![Text](/img/2024/homelab-ootbi-update-version/OOTBI-vcenter-Console.png)

Luego de eso, realizamos los siguientes pasos:

#### Paso 1: Acceder la consola de OOTBI (TUI)

En este paso accedemos a la consola de manejo de `OOTBI` donde seleccionamos el `Software Updates` desde menú de texto.

![Text](/img/2024/homelab-ootbi-update-version/OOTBI-Update-00.webp)

#### Paso 2: Aceptar los cambios de la actualización

Si el wizard encuentra una versión nueva, te notificará que aceptes los cambios presionando la opción de `Enter`.

![Text](/img/2024/homelab-ootbi-update-version/OOTBI-Update-01.webp)

#### Paso 3: Instalación de las actualizaciones

Aquí en este paso te permite ver el progreso de la actualización.

![Text](/img/2024/homelab-ootbi-update-version/OOTBI-Update-02.webp)

#### Paso 4: Culminación del proceso de actualización

Luego de que el proceso de la actualización culmine, puedes observar en la parte inferior que la versión del producto fue incrementada.

![Text](/img/2024/homelab-ootbi-update-version/OOTBI-Update-03.webp)

### Eso es todo amigos

En este laboratorio te mostramos que sencillo es instalar las actualizaciones del appliance virtual de `Object First OOTBI`. Espero que este laboratorio les haya gustado. Si tienes dudas o alguna pregunta sobre este laboratorio, déjalo en los comentarios.

Hasta la Próxima!

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/F1F8DEV80)

![Text](/img/2024/homelab-ootbi-update-version/HastaLuegoJirafales.png#center)
