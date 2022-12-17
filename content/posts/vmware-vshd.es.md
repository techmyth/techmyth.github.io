---
title: 'Implementación de VMware Skyline Health Diagnostics'
date: '2021-08-09T12:10:00-04:00'
tags:
    - VMware
---

Hola a tod@s

En esta oportunidad vengo a mostrarles la integración de «VMware Skyline Health Diagnostics» (VSHD) con VMware vCenter. También veremos cómo generar los diagnostico para saber como esta la salud de nuestra infraestructura Virtual. VSHD es una plataforma de auto-diagnóstico que permitedetectar y solucionar problemas en la línea de productos de vSphere y vSAN.

Esta herramienta proporciona recomendaciones en forma de artículos tipo «Knowledge Base» o enlaces con procedimiento para solucionar los problemas. Los administradores de vSphere pueden utilizar esta herramienta para solucionar los problemas antes de ponerse en contacto con el servicio de soporte global de VMware.

![Text](/img/2021-08-08_14-29.webp)

Beneficios:

- Basado en síntomas, según VMware VSHD proporciona de manera automática enlaces a artículos con pasos para resolver el problema.
- El auto-servicio mejora el tiempo para obtener recomendaciones que asistan en la resolución del problema.
- Una rápida reparación que contribuya a recuperar la infraestructura de un fallo y garantice que la empresa funcione con una menor interrupción.

Para comenzar debemos de acceder el siguiente enlace donde podemos descargar los el archivo tipo OVA que permite gestionar la creación de la maquina virtual donde corren los servicios de VSHD.

```http
https://my.vmware.com/group/vmware/get-download?downloadGroup=SKYLINE_HD_VSPHERE
```

Una vez autenticamos en el portal de VMware nos redirigirá al área donde podemos descargar en archivo con la extensión OVA.

![Text](/img/2021-07-25_11-37.webp)

A continuación les incluyo el proceso de crear la VM utilizando el template que bajamos en el formato OVA.

#### **Installing VSHD through VMware vCenter**

Comenzamos utilizando el wizard de **«Deploy OVF Template»** donde añadimos el archivo de instalación oprimiendo **«UPLOAD FILES»**,

![Text](/img/2021-07-25_11-54.webp)

Configuramos un nombre y seleccionamos en que carpeta se creara el objeto de la VM.

![Text](/img/2021-07-25_11-54_1.webp)

Seleccionamos el «Compute Cluster» y oprimimos **«NEXT»**.

![Text](/img/2021-07-25_11-55.webp)

Validamos la información y oprimimos **«NEXT»**.

![Text](/img/2021-07-25_11-57.webp)

Aceptamos el Acuerdo de Licenciamiento y oprimimos **«NEXT»**.

![Text](/img/2021-07-25_11-57_1.webp)

Seleccionamos la ubicación de almacenamiento donde se creara la VM y oprimimos **«NEXT»**.

![Text](/img/2021-07-25_11-57_2.webp)

Seleccionamos la red que utilizará la VM y oprimimos **«NEXT»**.

![Text](/img/2021-07-25_11-59.webp)

En esta etapa se definen las propiedades únicas de la VM como el Hostname, la contraseña de las cuentas de administración y la información de la direcciones de IP.

![Text](/img/2021-07-25_12-05.png)

Luego de validar la información procedemos a oprimir **«FINISH»** para completar el proceso

![Text](/img/2021-07-25_12-06.webp)

El proceso de instalación puede ser supervisado desde la pestaña de **«Recent Tasks»**.

![Text](/img/2021-07-25_12-06_1.webp)

Un requisito opcional es la asociación de un nombre de DNS al IP utilizado en el proceso de instalación. En la siguiente pantalla podemos ver como se registra un nombre de DNS **«FQDN»** utilizando Powershell desde una consola de Windows

```text
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

PS C:\Users\Administrator> Add-DnsServerResourceRecordA -Name vmware-shd -IPv4Address 192.168.5.70 -ZoneName zenprsolutions.local -CreatePtr -AllowUpdateAny

PS C:\Users\Administrator> Get-DnsServerResourceRecord -Name vmware-shd -ZoneName zenprsolutions.local

HostName                  RecordType Type       Timestamp            TimeToLive      RecordData
--------                  ---------- ----       ---------            ----------      ----------
vmware-shd                A          1          0                    01:00:00        192.168.5.70

PS C:\Users\Administrator>
```

Una vez culminado el proceso de instalación del archivo OVA podemos proceder a encender la VM que utilizaremos para el servicio de VSHD

![Text](/img/2021-07-30_10-52.webp)

Al encender la VM podemos ver la información del nombre de DNS y la dirección IP que configuramos previamente.

![Text](/img/2021-07-30_12-14.webp)

Con la dirección de IP podemos acceder el portal de administración de la aplicación utilizando las credenciales de:

- Usuario: **shd-admin**
- Contraseña: **previously established**

![Text](/img/2021-07-30_15-01.webp)

El próximo paso es añadir la información del vCenter/ESXi y las credenciales.

![Text](/img/2021-07-30_15-08.webp)

Podemos validar que la información ingresada es correcta oprimiendo el botón de **«CHECK CONNECTION»**.

![Text](/img/2021-07-30_15-08_1.webp)

Luego de validar la comunicación y las credenciales proseguimos a correr el diagnostico oprimiendo **«RUN DIAGNOSTIC»**.

![Text](/img/2021-07-30_15-10.webp)

En esta pantalla seleccionamos los servidores de ESXi y vCenter que deseamos escanear y que tipo de pluging se ejecutará durante la recopilación de la información de diagnóstico

![Text](/img/2021-07-30_15-11-1.webp)

De manera opcional podemos configurar un «Tag» que puede ser utilizado para una fácil búsqueda de los diagnósticos. Luego podemos presionar **«FINISH»**.

![Text](/img/2021-07-30_15-13.webp)

En esta próxima imagen podemos ver el progreso de la recopilación de la información de diagnóstico.

![Text](/img/2021-07-30_15-14_1-1.webp)

Adicionalmente desde la consola de manejo del vCenter podemos ver una tarea relacionada al proceso de recopilación.

![Text](/img/2021-07-30_15-15.webp)

Una vez terminado el proceso podemos ver el estado de la tarea presionando la opción de **«SHOW SUMMARY»**.

![Text](/img/2021-07-30_15-41-1.webp)

En esta pantalla podemos observar que las tareas fueron ejecutadas sin problemas.

![Text](/img/2021-07-30_15-41_1.webp)

Presionando el botón de **«SHOW REPORT»** podemos ver los reportes generados.

![Text](/img/2021-07-30_15-42.webp)

Para ver el reporte podemos presionar el icono de ojo según se muestra en la siguiente imagen.

![Text](/img/2021-07-30_16-16-1.webp)

Aquí les muestro varios ejemplo del reporte generado donde podemos identificar varios problemas un la infraestructura utilizada como ejemplo.

![Text](/img/2021-07-30_16-18.webp)

### Resumen

En este laboratorio instalamos y configuramos el «VMware Skyline Health Diagnostics» (VSHD) que permite a los administradores de vSphere utilizar esta herramienta para solucionar los problemas antes de ponerse en contacto con el soporte de VMware. Algo bueno de esta herramienta es que está disponible de forma gratuita. Espero que este laboratorio les haya gustado. Si tienes dudas o alguna pregunta sobre este laboratorio, déjalo en los comentarios. Saludos.
