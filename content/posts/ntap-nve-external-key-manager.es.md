---
title: "Configuración de cifrado en volumen de NetApp con Gestor de llaves externo"
date: 2021-05-16T08:39:31-04:00
draft: false
tags:
  - "NetApp"
  - "KMS"
  - "Encryption"
  - "External Manager"
---

Utilizando como referencia la documentación de NetApp:

> NetApp Volume Encryption (NVE) de NetApp es una tecnología basada en software para cifrar los datos en reposo de un volumen a la vez. Una clave de cifrado a la que sólo puede acceder el sistema de almacenamiento garantiza que los datos del volumen no puedan leerse si el dispositivo subyacente se reutiliza, se devuelve, se extravía o se roba.

##### [NetApp Documentation](https://docs.netapp.com/ontap-9/topic/com.netapp.doc.pow-nve/GUID-EAD13D8E-0219-45B6-A2C6-B25B76C9CA1A.html)

En este tutorial les explico lo fácil que es configurar y administrar esta impresionante característica de seguridad.

Antes de comenzar a configurar esta característica es necesario tener un **Servicio de Gestión de LLaves** existente. Para el propósito de este tutorial usaré el KMS de «HyTrust KeyControl» del que ya explique previamente en el blog. Si quieres saber más sobre el tema, lee el siguiente [post](http://192.168.7.40/2021/05/16/hytrust-keycontrol-key-management-server-setup/).

**Paso 1:** Crear un certificado de cliente para propósitos de autenticación:

Vaya a **[KMIP > Client Certificate]** y seleccione Create Certificate en el menú de Actions.

![Text](/img/2021-05-16_15-25-1024x563.webp#center)

Seleccione un nombre para el certificado y presione **Create**.

![Text](/img/2021-05-16_15-27-1024x565.webp#center)

Una vez creado el certificado se debe proceder a guardarlo en un lugar seguro.

![Text](/img/2021-05-16_15-28-1024x737.webp#center)

El archivo descargado contiene el certificado del **Cliente** y del **Root CA** necesarios para configurar la opción de KMS en Ontap.

![Text](/img/2021-05-16_15-31-1024x493.webp#center)

**Paso 2:** Validación del clúster Ontap

Los aspectos importantes a tener en cuenta antes de poder configurar esta función de seguridad son el estado del clúster y la licencia necesaria que le brinde soporte a la función de encriptación.

El comando **cluster show** muestra el estado general del cluster de Ontap.

```bash
OnPrem-HQ::> cluster show 
Node                  Health  Eligibility
--------------------- ------- ------------
OnPrem-HQ-01         true    true
OnPrem-HQ-02         true    true
2 entries were displayed.
```

El comando **system node show** muestra la salud del nodo y el modelo del sistema.

```bash
OnPrem-HQ::> system node show
Node      Health Eligibility Uptime        Model       Owner    Location  
--------- ------ ----------- ------------- ----------- -------- ---------------
OnPrem-HQ-01 true true           00:18:10 SIMBOX
OnPrem-HQ-02 true true           00:18:08 SIMBOX
2 entries were displayed.
```

Con el comando **system license show** se puede validar la licencia instalada en el cluster. Aquí se puede ver que la licencia de encriptación de volumen está instalada en ambos nodos.

```bash
OnPrem-HQ::> system license show -package VE

Serial Number: X-XX-XXXXXXXXXXXXXXXXXXXXXXXXXX
Owner: OnPrem-HQ-01
Installed License: Legacy Key
Capacity: -
Package           Type     Description           Expiration
----------------- -------- --------------------- -------------------
VE                license  Volume Encryption License 
                                                 -

Serial Number: X-XX-XXXXXXXXXXXXXXXXXXXXXXXXXX
Owner: OnPrem-HQ-02
Installed License: Legacy Key
Capacity: -
Package           Type     Description           Expiration
----------------- -------- --------------------- -------------------
VE                license  Volume Encryption License 
                                                 -
2 entries were displayed.

OnPrem-HQ::> 
```

**Paso 3:** Configuración del certificado de KMS en Ontap:

#### Según establecido en la documentación de NetApp:

> El clúster y el servidor KMIP utilizan los certificados SSL de KMIP para verificar la identidad del otro y establecer una conexión SSL. Antes de configurar la conexión SSL con el servidor KMIP, debe instalar los certificados SSL de cliente KMIP para el clúster y el certificado público SSL para la autoridad de certificación (CA) raíz del servidor KMIP.

##### [NetApp KMIP Documentation](https://docs.netapp.com/ontap-9/topic/com.netapp.doc.pow-nve/GUID-D1593ED6-AAF8-4DEE-A2A7-6AEF239C6874.html)

Para instalar el certificado de cliente KMIP en el cluster de NetApp, ejecute los siguientes comandos:

##### Nota: El contendido de los certificados son extraídos de los archivos previamente descargados. Los archivos ONTAPEncryption.pem y cacert.pem contienen la información necesaria.

```text
OnPrem-HQ::> security certificate install –vserver OnPrem-HQ -type client –subtype kmip-cert
Please enter Certificate: Press <Enter> when done
-----BEGIN CERTIFICATE----- 
Certificate Content
-----END CERTIFICATE-----
Please enter Private Key: Press <Enter> when done
-----BEGIN PRIVATE KEY-----  
Certificate Private Key Content
-----END PRIVATE KEY-----
You should keep a copy of the private key and the CA-signed digital certificate for future
reference.

The installed certificate's CA and serial number for reference:
CA: HyTrust KeyControl Certificate Authority
serial: C9A148B9

The certificate's generated name for reference: ONTAPEncryption
```

```text
OnPrem-HQ::> security certificate install -vserver OnPrem-HQ -type server-ca -subtype kmip-cert 

Please enter Certificate: Press <Enter> when done
-----BEGIN CERTIFICATE-----  
Certificate Content
-----END CERTIFICATE-----
You should keep a copy of the CA-signed digital certificate for future reference.

The installed certificate's CA and serial number for reference:
CA: HyTrust KeyControl Certificate Authority
serial: 60A148B6

The certificate's generated name for reference: HyTrustKeyControlCertificateAuthority
```

**Paso 4:** Configuración de la solución Volume Encryption de NetApp:

Para este tutorial es necesario configurar un servidor de gestión de llaves «KMS» externo para que el sistema de almacenamiento pueda almacenar y recuperar de forma segura las llaves de autenticación para la solución de NetApp Volume Encryption (NVE).

##### Nota: NetApp recomienda un mínimo de dos servidores para la redundancia y la recuperación de desastres.

El siguiente comando permite añadir el servidor de KMS al sistema de Ontap utilizando la dirección de IP 192.168.7.201, el puerto TCP/5696 y utilizando los certificados configurados previamente.

```text
OnPrem-HQ::> security key-manager external enable -vserver OnPrem-HQ -key-servers 192.168.7.201:5696 -client-cert ONTAPEncryption -server-ca-certs HyTrustKeyControlCertificateAuthority 

OnPrem-HQ::> security key-manager external show                                                                                                                                           

                  Vserver: OnPrem-HQ
       Client Certificate: ONTAPEncryption
   Server CA Certificates: HyTrustKeyControlCertificateAuthority
          Security Policy: -

Key Server
------------------------------------------------
192.168.7.201:5696
```

Es importante validar que el servicio de KMS esté **available** antes de proseguir a crear los volúmenes encriptados. El comando security key-manager external show-status no permite validar el estado del servicio.

```text
OnPrem-HQ::> security key-manager external show-status

Node  Vserver  Key Server                                   Status
----  -------  -------------------------------------------  ---------------
OnPrem-HQ-01
      OnPrem-HQ
               192.168.7.201:5696                           available
OnPrem-HQ-02
      OnPrem-HQ
               192.168.7.201:5696                           available
2 entries were displayed.

OnPrem-HQ::> 
```

**Paso 5:** Crear un volumen encriptado (NVE)

En ésta etapa final del tutorial probaremos que la configuración realizada es la correcta al crear un volumen encriptado dentro de Ontap. Para este paso usaremos el comando **vol create** utilizando la opcion de **encrypt true**.

```text
OnPrem-HQ::> vol create TEST_Encryption -vserver SAN -size 10G -aggregate OnPrem_HQ_01_SSD_1 -encrypt true 
  
[Job 763] Job succeeded: Successful 
```

Con el comando **vol show** podemos verificar que el volumen haya sido creado con la opción de **encriptación**.

```text
OnPrem-HQ::> vol show -encryption -vserver SAN -encryption-state full 
Vserver   Volume       Aggregate    State      Encryption State
--------- ------------ ------------ ---------- ----------------
SAN       TEST_Encryption OnPrem_HQ_01_SSD_1 online full

OnPrem-HQ::> 
```

**Paso 6:** Validar en el servidor KMS la información de **Encriptación**.

En este ultimo paso entramos al portal de administración de la aplicación **HyTrust KeyControl** para validar que las llaves de encriptacion estén grabadas en la plataforma. Para validar ésta información vamos al menú de **[KMIP > Objects]** donde se puede validar que las llaves fueron creadas luego de creado el volumen dentro de Ontap.

![Text](/img/HytrustDashBoard.webp#center)

### Resumen

En este tutorial mostramos como configurar el servicio de KMS dentro de Ontap que nos permite crear volúmenes encriptados permitiendo aumentar o mejorar la postura de seguridad en nuestra infraestructura u organización.

### Hasta Luego Amigos!
