---
title: "Encriptación a nivel de agregado en ONTAP"
date: 2021-05-20T08:39:31-04:00
draft: false
tags:
  - "NetApp"
  - "KMS"
  - "Encryption"
---

Previament en un  [post](http://192.168.7.40/2021/05/16/netapp-volume-encryption-setup-external-key-manager/) expliqué como configurar un volumen encriptado utilizando un administrador de llaves de encriptación (KMS) especificamente de la compañía [HyTrust](https://www.hytrust.com/products/keycontrol/). En este caso específico cada volumen es encriptado individualmente utilizado llaves independientes. Una desventaja de este método es que afecta la posibilidad de aumentar los niveles de eficiencia de reducción de datos como lo son; de-duplicación, compresión y compactación (cross-volume-dedupe).

![Text](/img/NVE-vs-NAE.webp#center)

Para eliminar esta desventaja los gurus de NetApp se ingeniaron la idea de aplicar la función de encriptación a nivel de agregado permitiendo que los volúmenes que residan dentro del mismo agregado compartan la llave de encriptación. Esta tecnología es conocida como **"NetApp Aggregate Encryption (NAE)"**. Esto permite que los clientes tengan la opción de aprovecharse de las tecnologías de eficiencia de almacenamiento en conjunto al proceso de encriptación.

Ahora nos toca hablar de como podemos crear un agregado encriptado en Ontap pero primero que nada… ¿Que es un agregado dentro de Ontap?

Utilizando como referencia el portal de «Knowledge Base» de NetApp:

> Un agregado es una colección de discos (o particiones) organizados en uno o más grupos de RAID. Es el objeto de almacenamiento más básico dentro de ONTAP y es necesario para permitir el aprovisionamiento de espacio para los dispositivos conectados.

##### [NetApp Knowledge Base](https://kb.netapp.com/Advice_and_Troubleshooting/Data_Storage_Software/ONTAP_OS/What_is_an_aggregate%3F)

![Text](/img/2021-05-20_21-21-1024x746.webp#center)

**Paso 1:** Validar pre-requisitos en Ontap.

Para poder utilizar la opción de encriptación a nivel de agregado en necesario tener una versión de Ontap 9.6 o mayor y que estén instaladas las licencias necesarias en el cluster. En este caso utilizamos el comando **version** para validar la versión actual del cluster y el comando **license show -package VE** para desplegar la información de las licencias.

```bash
OnPrem-HQ::> version
NetApp Release 9.9.1RC1: Fri Apr 30 06:35:11 UTC 2021
 
OnPrem-HQ::> license show -package VE -fields package,owner,description,type  
  (system license show)
serial-number                  package owner         description               type    
------------------------------ ------- ------------- ------------------------- ------- 
X-XX-XXXXXXXXXXXXXXXXXXXXXXXXX VE      OnPrem-HQ-01 Volume Encryption License license 
X-XX-XXXXXXXXXXXXXXXXXXXXXXXXX VE      OnPrem-HQ-02 Volume Encryption License license 
2 entries were displayed.

OnPrem-HQ::> 
```

##### Nota: previamente he realizado la configuración del KMS externo en Ontap. [Link](http://192.168.7.40/2021/05/16/netapp-volume-encryption-setup-external-key-manager/)

**Paso 2:** Validar los disco «Spare» disponibles.

Para comenzar, existen dos formas para encriptar un agregado; inicialmente cuando es creado o la conversión en vivo de un agregado existente. Inicialmente estaré creando un agregado nuevo y luego en otro tutorial les mostrare como podemos convertir un agregado existente. Para crear un agregado es necesario tener disco duros disponibles o en el estado de **spare** como comúnmente le llama NetApp.

El comando **storage aggregate show-spare-disks** nos permite ver cuantos discos particionado están disponibles en el nodo donde crearemos el nuevo agregado encriptado. En este caso en particular podemos ver que existen 24 disco particionados utilizando la opción de **Root-Data1-Data2**. Para conocer mas sobre esta estrategia de disco favor de seguir el siguiente enlace:

> [ADP(v1) and ADPv2 in a nutshell, it’s delicious!](https://blog.iops.ca/2016/11/10/adpv1-and-adpv2-in-a-nutshell-its-delicious/)
>
> © 2021 Chris Maki

```bash
OnPrem-HQ::> storage aggregate show-spare-disks -original-owner OnPrem-HQ-01 
                                                                      
Original Owner: OnPrem-HQ-01
 Pool0
  Root-Data1-Data2 Partitioned Spares
                                                              Local    Local
                                                               Data     Root Physical
 Disk             Type   Class          RPM Checksum         Usable   Usable     Size Status
 ---------------- ------ ----------- ------ -------------- -------- -------- -------- --------
 VMw-1.1          SSD    solid-state      - block           11.63GB       0B  26.67GB zeroed
 VMw-1.2          SSD    solid-state      - block           11.63GB       0B  26.67GB zeroed
 VMw-1.3          SSD    solid-state      - block           11.63GB       0B  26.67GB zeroed
 VMw-1.4          SSD    solid-state      - block           11.63GB       0B  26.67GB zeroed
 VMw-1.5          SSD    solid-state      - block           11.63GB       0B  26.67GB zeroed
 VMw-1.6          SSD    solid-state      - block           11.63GB       0B  26.67GB zeroed
 VMw-1.7          SSD    solid-state      - block           11.63GB       0B  26.67GB zeroed
 VMw-1.8          SSD    solid-state      - block           11.63GB       0B  26.67GB zeroed
 VMw-1.9          SSD    solid-state      - block           11.63GB       0B  26.67GB zeroed
 VMw-1.10         SSD    solid-state      - block           11.63GB       0B  26.67GB zeroed
 VMw-1.11         SSD    solid-state      - block           11.63GB   3.35GB  26.67GB zeroed
 VMw-1.12         SSD    solid-state      - block           11.63GB   3.35GB  26.67GB zeroed
 VMw-1.13         SSD    solid-state      - block           11.63GB       0B  26.67GB zeroed
 VMw-1.14         SSD    solid-state      - block           11.63GB       0B  26.67GB zeroed
 VMw-1.15         SSD    solid-state      - block           11.63GB       0B  26.67GB zeroed
 VMw-1.16         SSD    solid-state      - block           11.63GB       0B  26.67GB zeroed
 VMw-1.17         SSD    solid-state      - block           11.63GB       0B  26.67GB zeroed
 VMw-1.18         SSD    solid-state      - block           11.63GB       0B  26.67GB zeroed
 VMw-1.19         SSD    solid-state      - block           11.63GB       0B  26.67GB zeroed
 VMw-1.20         SSD    solid-state      - block           11.63GB       0B  26.67GB zeroed
 VMw-1.21         SSD    solid-state      - block           11.63GB       0B  26.67GB zeroed
 VMw-1.22         SSD    solid-state      - block           11.63GB       0B  26.67GB zeroed
 VMw-1.23         SSD    solid-state      - block           11.63GB       0B  26.67GB zeroed
 VMw-1.24         SSD    solid-state      - block           11.63GB       0B  26.67GB zeroed
24 entries were displayed.

OnPrem-HQ::> 
```

**PAso 3:** Crear el agregado encriptado.

Para crear el agregado encriptado utilizamos el comando **storage aggregate create** con la opción de **encrypt-with-aggr-key true**. En este caso creamos un agregado seguro compuesto de 23 disco «particiones».

##### Nota: Para este ejemplo se utilizo el tipo de RAID **Dual Parity**

```bash
OnPrem-HQ::> storage aggregate create -aggregate OnPrem_HQ_01_SSD_1 -diskcount 23 -node OnPrem-HQ-01 -raidtype raid_dp -encrypt-with-aggr-key true 

Info: The layout for aggregate "OnPrem_HQ_01_SSD_1" on node "OnPrem-HQ-01"
      would be:
      
      First Plex
      
        RAID Group rg0, 23 disks (block checksum, raid_dp)
                                                            Usable Physical
          Position   Disk                      Type           Size     Size
          ---------- ------------------------- ---------- -------- --------
          shared     VMw-1.1                   SSD               -        -
          shared     VMw-1.2                   SSD               -        -
          shared     VMw-1.3                   SSD         11.61GB  11.64GB
          shared     VMw-1.4                   SSD         11.61GB  11.64GB
          shared     VMw-1.5                   SSD         11.61GB  11.64GB
          shared     VMw-1.6                   SSD         11.61GB  11.64GB
          shared     VMw-1.7                   SSD         11.61GB  11.64GB
          shared     VMw-1.8                   SSD         11.61GB  11.64GB
          shared     VMw-1.9                   SSD         11.61GB  11.64GB
          shared     VMw-1.10                  SSD         11.61GB  11.64GB
          shared     VMw-1.18                  SSD         11.61GB  11.64GB
          shared     VMw-1.16                  SSD         11.61GB  11.64GB
          shared     VMw-1.13                  SSD         11.61GB  11.64GB
          shared     VMw-1.14                  SSD         11.61GB  11.64GB
          shared     VMw-1.15                  SSD         11.61GB  11.64GB
          shared     VMw-1.19                  SSD         11.61GB  11.64GB
          shared     VMw-1.20                  SSD         11.61GB  11.64GB
          shared     VMw-1.21                  SSD         11.61GB  11.64GB
          shared     VMw-1.17                  SSD         11.61GB  11.64GB
          shared     VMw-1.22                  SSD         11.61GB  11.64GB
          shared     VMw-1.11                  SSD         11.61GB  11.64GB
          shared     VMw-1.12                  SSD         11.61GB  11.64GB
          shared     VMw-1.23                  SSD         11.61GB  11.64GB
      
      Aggregate capacity available for volume use would be 219.5GB.
      
Do you want to continue? {y|n}: y
[Job 817] Job succeeded: DONE                                                  

OnPrem-HQ::>
```

Una vez creado pasaremos a verificar el agregado, para esto utilizaremos el comando **storage aggregate show** y filtraremos el resultado con la opción de **encrypt-with-aggr-key**.

```bash
OnPrem-HQ::> storage aggregate show -fields aggregate,size,availsize,usedsize,state,node,raidstatus,encrypt-with-aggr-key 
aggregate           node          availsize raidstatus      size    state  usedsize encrypt-with-aggr-key 
------------------- ------------- --------- --------------- ------- ------ -------- --------------------- 
OnPrem_HQ_01_SSD_1 OnPrem-HQ-01 219.5GB   raid_dp, normal 219.5GB online 480KB    true                  
OnPrem_HQ_02_SSD_1 OnPrem-HQ-02 209.3GB   raid_dp, normal 219.5GB online 10.12GB  false                 
aggr0_OnPrem_HQ_01 OnPrem-HQ-01 1.11GB    raid_dp, normal 22.80GB online 21.69GB  false                 
aggr0_OnPrem_HQ_02 OnPrem-HQ-02 1.11GB    raid_dp, normal 22.80GB online 21.69GB  false                 
4 entries were displayed.

OnPrem-HQ::> 
```

En el resultado del comando podemos ver que fue creado el agregado con la encriptación habilitada.

**Paso 4:** Crear un volumen dentro del agregado encriptado.

A diferencia de la encriptación a nivel de volúmenes NVE, al utilizar encriptación a nivel de agregado no es necesario especificar la opción de encriptar a la hora de crear el volumen. El comando **vol create** crea un volumen encriptado de forma predeterminada cuando el volumen reside en un agregado que utiliza NAE.

```bash
OnPrem-HQ::> vol create -vserver SAN -volume Secure_Vol -aggregate OnPrem_HQ_01_SSD_1 -size 10GB -space-guarantee none 
[Job 818] Job succeeded: Successful                                            

OnPrem-HQ::>
```

Al utilizar el comando **vol show** con la opción de **encryption-state full** podemos ver que el volumen fue creado encriptado de forma predeterminada.

```bash
OnPrem-HQ::> vol show -encryption-state full -aggregate OnPrem_HQ_01_SSD_1 -fields Vserver,Volume,encrypt,encryption-type,encryption-state 
vserver volume     encryption-type encrypt encryption-state 
------- ---------- --------------- ------- ---------------- 
SAN     Secure_Vol aggregate       true    full             

OnPrem-HQ::>
```

### Resumen

En este tutorial le mostré cómo configurar la tecnología de encriptación de nivel agregado dentro de Ontap que nos permite utilizar una clave única para crear volúmenes encriptados. Esto nos permite utilizar tecnologías de reducción y eficiencia de datos en conjunto con mecanismos de seguridad que mejoran o fortalecen la postura de seguridad de la organización.

**Hasta luego!!!**

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/F1F8DEV80)

![Text](/img/hasta-luego-5937ba.webp#center)
