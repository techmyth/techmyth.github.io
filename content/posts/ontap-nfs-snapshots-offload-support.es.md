---
title: 'Snapshots Nativos en NFS con vSphere 7 Update 2'
date: '2021-05-30T21:07:02-04:00'
author: 'Jonathan Colon Feliciano'
tags:
    - NetApp
    - VMware
---

La versión vSphere 7.0 U2 ofrece la posibilidad de utilizar snapshots nativos cuando se utiliza el protocolo NFS como mecanismo de acceso. Como se describe en el blog de VMware:

> `NFS Improvements`
>
> NFS required a clone to be created first for a newly created VM and the subsequent ones could be offloaded to the array. With the release of vSphere 7.0 U2, we have enabled NFS array snapshots of full, non-cloned VMs to not use redo logs but instead use the snapshot technology of the NFS array in order to provide better snapshot performance. The improvement here will remove the requirement/limitation of creating a clone and enables the first snapshot also to be offloaded to the array.
>
> [What’s New in vSphere 7 Update 2 Core Storage](https://core.vmware.com/blog/whats-new-vsphere-7-update-2-core-storage)

![Text](/img/5066053.webp#center)

En este blog explico la configuración necesaria para probar esta nueva funcionalidad. Para empezar debemos validar los requisitos para poder implementar esta solución. Según el portal de documentación de VMware los requisitorios son los siguientes:

- Compruebe que la matriz NAS admite la operación de clonación rápida de archivos con el programa VAAI NAS.
- En su host ESXi, instale el plug-in NAS específico del proveedor que soporta la clonación rápida de archivos con VAAI.
- Siga las recomendaciones de su proveedor de almacenamiento NAS para configurar los ajustes necesarios tanto en la matriz NAS como en ESXi.

La configuración de NFS se realizará en NetApp Ontap mediante el complemento ``NetApp NFS Plug-in for VMware VAAI``, que recientemente ha añadido soporte nativo para la descarga de snapshot en NFS.

##### Note: El plug-in puede descargarse desde el portal de soporte de NetApp en el siguiente enlace [NetApp Support](https://mysupport.netapp.com/site/products/all/details/nfsplugin-vmware-vaai/downloads-tab/download/61278/2.0)

![Text](/img/NetApp-NFS-Plugin-1-1024x577.webp#center)

Una vez que estemos en el portal de soporte de NetApp debemos descargar la versión 2.0 del plugin como se muestra en la siguiente imagen. Para instalar el plugin debemos descomprimir el archivo descargado y renombrar el archivo dentro de la carpeta llamada vib20 con la extensión .vib al nuevo nombre `NetAppNasPlugin.vib`.

![Text](/img/2021-05-30_20-33-1024x707.webp#center)

En el siguiente paso utilicé el Ontap Tools de NetApp para instalar el plugin, pero también se puede instalar utilizando VMware Lifecycle Manager.

![Text](/img/2021-05-30_20-49-1024x510.webp#center)

Para instalar el plugin vaya a [ONTAP tools => Settings => NFS VAAI tools] y en la sección ``Existing version:`` pulse ``BROWSE`` para seleccionar donde está almacenado el archivo ``NetAppNasPlugin.vib``. Una vez localizado el archivo pulse ``UPLOAD`` para cargar e instalar el plugin.

![Text](/img/2021-05-30_20-36-1024x541.webp#center)

En este paso podemos ver cómo instalar el plugin en los servidores ESXi pulsando el botón ``INSTALL``.

![Text](/img/2021-05-30_20-47-1024x674.webp#center)

La siguiente imagen muestra que la instalación del plugin fue exitosa. Una ventaja de la nueva versión del plugin es que no es necesario reiniciar el servidor ESXi.

![Text](/img/2021-05-30_20-48-1024x288.webp#center)

Una vez instalado el plugin procederemos a validar que Ontap tiene soporte para las características de VMware vStorage APIs for Array Integration (VAAI) en el ambiente de NFS. Esto se puede verificar con el comando `vserver nfs show -fields vstorage`. Como puede ver la función vStorage está actualmente deshabilitada en el SVM llamado NFS. Para habilitar la función vStorage utilice el comando `vserver nfs modify -vstorage enabled`.

```text
OnPrem-HQ::> vserver nfs show -fields vstorage 
vserver vstorage 
------- -------- 
<strong>NFS     disabled </strong> 

OnPrem-HQ::> vserver nfs modify <strong>-vstorage enabled</strong> -vserver NFS 

OnPrem-HQ::> vserver nfs show -fields vstorage                 
vserver vstorage 
------- -------- 
<strong>NFS     enabled</strong>  

OnPrem-HQ::> 
```

El siguiente requisito para poder utilizar la descarga nativa de snapshot es la creación de un ajuste avanzado en la configuración de la VM llamado `snapshot.alwaysAllowNative`. Para añadir este valor hay que ir a las propiedades de la VM y luego a `[VM Options => Advanced => EDIT CONFIGURATION]`.

![Text](/img/2021-05-31_14-46-768x721.webp#center)

La siguiente imagen muestra el valor de la variable `snapshot.alwaysAllowNative` que según la documentación de VMware debe tener un valor igual a ``TRUE``. Puede utilizar el siguiente enlace como referencia ``VMware Documentation``

![Text](/img/2021-05-31_14-45-768x343.webp#center)

Ahora empiezo a hacer pruebas para validar que los snapshots nativo funcionan en Ontap. Primero crearé un snapshot con la función `snapshot.alwaysAllowNative` establecida en `FALSE`. Luego haré cambios en la VM para poder medir la velocidad de eliminación y aplicación de los cambios del snapshot al disco base. En el ejemplo que se muestra a continuación se utilizó el comando `New-Snapshot` en PowerCLI para crear una snapshot de la VM llamada RocaWeb

```sh
PS /home/rebelinux> get-vm -Name RocaWeb | New-Snapshot -Name PRE_Native_Array_Snapshot | Format-Table -Wrap -AutoSize  

Name                        Description                 PowerState 
----                        -----------                 ----------
PRE_Native_Array_Snapshot                               PoweredOff
PS /home/rebelinux> 
```

En este paso se ha copiado un archivo de `10GB` para hacer crecer el snapshot y así poder medir la rapidez con la que se aplican los cambios al disco base cuando se elimina el snapshot. En este ejemplo el archivo ``RocaWeb_2-000001-delta.vmdk`` representa el delta donde se guardan los cambios del snapshot. Esto representa un snapshot tradicional de VMware.

```sh
[root@comp-01a:/vmfs/volumes/55ab62ec-2abeb31b/RocaWeb] ls -alh
total 35180596
drwxr-xr-x    2 root     root        4.0K May 31 23:40 .
drwxr-xr-x    7 root     root        4.0K May 31 19:02 ..
-rw-------    1 root     root      276.0K May 31 23:40 RocaWeb-Snapshot15.vmsn
-rw-------    1 root     root        4.0G May 31 23:40 RocaWeb-a03f2017.vswp
-rw-------    1 root     root      264.5K May 31 23:40 RocaWeb.nvram
-rw-------    1 root     root         394 May 31 23:40 RocaWeb.vmsd
-rwxr-xr-x    1 root     root        3.4K May 31 23:40 RocaWeb.vmx
-rw-------    1 root     root       10.0G May 31 23:51 RocaWeb_2-000001-delta.vmdk #Delta (VMFS Based Snapshot)
-rw-------    1 root     root         301 May 31 23:40 RocaWeb_2-000001.vmdk
-rw-------    1 root     root      500.0G May 31 23:37 RocaWeb_2-flat.vmdk
-rw-------    1 root     root         631 May 31 23:37 RocaWeb_2.vmdk
[root@comp-01a:/vmfs/volumes/55ab62ec-2abeb31b/RocaWeb]
```

La siguiente imagen muestra el tiempo que se tardó en aplicar los cambios del snapshot al disco base cuando se eliminó el snapshot. En resumen, la operación tardó `9 minutos` en total utilizando snapshots tradicionales de VMware.

##### Note: El simulador de Ontap fue utilizado para el laboratorio

![Text](/img/2021-05-31_20-05-1024x134.webp#center)

En este último ejemplo también se utilizó el comando `New-Snapshot` para crear el snapshot pero con la opción `snapshot.alwaysAllowNative` establecida en ``TRUE``. De esta manera podemos probar el uso de ``Native Snapshot Offload`` en NFS. Aquí también se copió un archivo de `10GB` a la VM para hacer crecer el snapshot, de modo que podamos medir la rapidez con la que se aplican los cambios al disco base cuando se elimina el snapshot.

```text
PS /home/rebelinux> get-vm -Name RocaWeb | New-Snapshot -Name POST_Native_Array_Snapshot | Format-Table -Wrap -AutoSize

Name                       Description                        PowerState
----                       -----------                        ----------
POST_Native_Array_Snapshot                                    PoweredOff
PS /home/rebelinux> 
```

Aquí podemos ver que no hay ningún archivo ``-delta.vmdk`` pero hay un archivo llamado ``RocaWeb_2-000001-flat.vmdk`` con el mismo tamaño de `500GB` que el archivo ``RocaWeb_2-flat.vmdk``. Esto nos permite confirmar que la función `NFS Native Snapshot Offload` está habilitada en Ontap.

```sh
[root@comp-01a:/vmfs/volumes/55ab62ec-2abeb31b/RocaWeb] ls -alh
total 49419672
drwxr-xr-x    2 root     root        4.0K Jun  1 00:07 .
drwxr-xr-x    7 root     root        4.0K May 31 19:02 ..
-rw-------    1 root     root      276.0K Jun  1 00:07 RocaWeb-Snapshot16.vmsn
-rw-------    1 root     root        4.0G Jun  1 00:07 RocaWeb-a03f2017.vswp
-rw-------    1 root     root      264.5K Jun  1 00:07 RocaWeb.nvram
-rw-------    1 root     root         393 Jun  1 00:07 RocaWeb.vmsd
-rwxr-xr-x    1 root     root        3.4K Jun  1 00:07 RocaWeb.vmx
-rw-------    1 root     root      500.0G Jun  1 00:09 RocaWeb_2-000001-flat.vmdk #No Delta (Array Based Snapshot OffLoad)
-rw-------    1 root     root         650 Jun  1 00:07 RocaWeb_2-000001.vmdk
-rw-------    1 root     root      500.0G Jun  1 00:03 RocaWeb_2-flat.vmdk
-rw-------    1 root     root         631 Jun  1 00:07 RocaWeb_2.vmdk
[root@comp-01a:/vmfs/volumes/55ab62ec-2abeb31b/RocaWeb]
```

La siguiente imagen muestra el tiempo que se tardó en aplicar los cambios del snapshot al disco base cuando se eliminó el snapshot utilizando ``NFS Native Snapshot Offload``. En resumen, se puede ver que la aplicación de los cambios del snapshot al disco base no tardó nada en terminar.

![Image](/img/2021-05-31_20-20-1024x125.webp#centered)

### Resumen

Las operaciones con ``NFS native snapshot offload`` son tan rápidas porque ONTAP hace referencia a la metadata cuando crea una copia Snapshot, en lugar de copiar bloques de datos, que es la razón por la que las copias Snapshot son tan eficientes. Al hacerlo, se elimina el tiempo de búsqueda en el que incurren otros sistemas para localizar los bloques a copiar, así como el coste de realizar la propia copia.
