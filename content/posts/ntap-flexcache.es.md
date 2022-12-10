---
title: "Uso de Ontap FlexCache para acelerar el acceso a los datos compartidos de Window"
date: 2021-05-27T08:39:31-04:00
draft: false
tags:
  - "NetApp"
---

En la versión de Ontap 9.8 NetApp decidió añadirle compatibilidad al protocolo SMB de Windows al utilizar la tecnología de FlexCache, Por fin….

En este laboratorio de práctica crearé un volumen flexcache de origen y uno de caché en un clúster remoto. En el ejemplo de laboratorio también validare el beneficio ofrecido por la capacidad de ampliar un recurso compartido CIFS central de forma nativa.

Para comenzar utilizaré como referencia la documentación de NetApp para definir que rayos es un volumen Flexcache y para que es utilizado:

![Text](/img/f751fba8c585a669d5f9f36497c9c310-2.webp#center)

> Un volumen FlexCache es un volumen poco poblado que está respaldado por un volumen de origen. El volumen FlexCache puede estar en el mismo clúster o en un clúster diferente al del volumen de origen. El volumen FlexCache proporciona acceso a los datos del volumen de origen sin necesidad de que todos los datos estén en el volumen FlexCache. A partir de ONTAP 9.8, un volumen FlexCache también soporta el protocolo SMB.

##### [NetApp Documentation Portal](https://docs.netapp.com/ontap-9/topic/com.netapp.doc.pow-fc-mgmt/GUID-F4CE375E-DB00-403E-A20D-FB6CE6116D07.html)

Para este laboratorio utilizaré como referencia el siguiente diagrama donde se muestra un dominio de «Active Directory» con dos «Sites» llamados **Gurabo** y **Ponce**. Ambos **Sites** poseen un clúster de Ontap con la versión 9.8P4. Flexcache requiere la configuración de interfaces tipo **Intercluster**.

##### Nota: Para el laboratorio se utilizó el simulador de Ontap

![Text](/img/Flexcache.webp#center)

Para poder comenzar con el laboratorio es necesario crear una asociación entre ambos **vservers** el local **NAS** y el remoto **NAS-EDGE**. Para lograr esto utilizamos el comando **vserver peer create** especificando que el «applications» sea como **flexcache**.

In order to start with the lab it is needed to create an peer relationship between the local and remote vserver. To achieve this i use the command **vserver peer create** specifying the **“applications”** as **“flexcache”**.

##### Referencia: [vserver peer create](http://docs.netapp.com/ontap-9/topic/com.netapp.doc.dot-cm-cmpr-940/vserver__peer__create.html)

##### Nota: Previamente se realizo la asociación a nivel de clúster con el comando **cluster peer create**.

```text
OnPrem-HQ::> vserver peer create -vserver NAS -peer-cluster OnPrem-EDGE -peer-vserver NAS-EDGE -applications flexcache 

Info: [Job 883] 'vserver peer create' job queued 
```

Una vez creada la asociación entre ambos vserver podemos comenzar a validar que el volumen que usaremos como origen este creado. Para esto utilizamos el comando **volume show** desde el clúster local. Para éste laboratorio utilizaremos el volumen llamado share. Les dejaré como referencia el cómo crear un volumen en Ontap desde el comienzo. [Enlace](https://docs.netapp.com/ontap-9/topic/com.netapp.doc.dot-cm-cmpr-960/volume__create.html)

```text
OnPrem-HQ::*> volume show -vserver NAS                
Vserver   Volume       Aggregate    State      Type       Size  Available Used%
--------- ------------ ------------ ---------- ---- ---------- ---------- -----
NAS       NAS_root     OnPrem_HQ_01_SSD_1 online RW      20MB    17.66MB    7%
NAS       share        OnPrem_HQ_01_SSD_1 online RW      10.3GB   8.04GB   20%
19 entries were displayed.

OnPrem-HQ::*> 
```

Ya identificado el volumen que utilizaremos podemos crear el volumen tipo flexcache utilizando el comando **volume flexcache create**. Es importante mencionar que flexcache utiliza **FlexGroup** para crear el volumen. Es por esta razón, que se utiliza la opción de **aggr-list** para especificar cuales agregados se utilizarán para crear los volúmenes tipo

```text
OnPrem-EDGE::> volume flexcache create -vserver NAS-EDGE -volume share_edge -aggr-list OnPrem_EDGE_0* -origin-vserver NAS -origin-volume share -size 10GB -junction-path /share_edge
[Job 595] Job succeeded: Successful.                                           

OnPrem-EDGE::>
```

Desde el clúster remoto podemos verificar el volumen creado utilizando el comando **vol flexcache show**.

```text
OnPrem-EDGE::> vol flexcache show
Vserver Volume      Size       Origin-Vserver Origin-Volume Origin-Cluster
------- ----------- ---------- -------------- ------------- --------------
NAS-EDGE share_edge 10GB       NAS            shares            OnPrem-HQ

OnPrem-EDGE::> 
```

Desde el clúster local podemos ver el volumen de origen con el comando **volume flexcache origin show-caches**. En el resultado del comando podemos ver el volumen flexcache previamente creado.

```text
OnPrem-HQ::*> volume flexcache origin show-caches
Origin-Vserver Origin-Volume  Cache-Vserver  Cache-Volume  Cache-Cluster
-------------- -------------- -------------- ------------- --------------
NAS            share         NAS-EDGE       share_edge    OnPrem-EDGE
1 entries were displayed.

OnPrem-HQ::*> 
```

Ahora procedemos a compartir el volumen caché share_edge utilizando el protocolo SMB. Para esto el comando **vserver cifs share create** es utilizado con la opción de **-path /share_edge** para especificar el «junction-path» del volumen flexclone.

```text
OnPrem-EDGE::> vserver cifs share create -vserver NAS-EDGE -share-name share_edge -path /share_edge

OnPrem-EDGE::>
```

Ahora podemos ver que el «Share» fue creado en el volumen **share_edge**.

```text
OnPrem-EDGE::> vserver cifs share show -share-name share_edge
Vserver        Share         Path              Properties Comment  ACL
-------------- ------------- ----------------- ---------- -------- -----------
NAS-EDGE       share_edge    /share_edge       oplocks    -        Everyone / Full Control
                                               browsable
                                               changenotify
                                               show-previous-versions

OnPrem-EDGE::> 
```

Utilicé la herramienta de **smbmap** para validar que la carpeta compartida puede ser accedida desde la red.

```sh
[rebelinux@blabla ~]$ smbmap.py -H 10.10.33.20 -p "XXXXX" -d ZENPRSOLUTIONS -u administrator 
[+] IP: 10.10.33.20:445	Name: NAS-EDGE.zenprsolutions.local                            
Disk                                              Permissions Comment
----                                              ----------- -------
share_edge                                        READ, WRITE
ipc$                                              NO ACCESS
c$                                                READ, WRITE
[rebelinux@blabla ~]$
```

Modifiqué el diagrama original para mostrar como están conectados los clientes que utilizaré para realizar las pruebas de transferencia. Para esta prueba copiaré desde cada **SHARE** el archivo **Very_Big_File.iso**.

![Text](/img/Flexcache2.webp#center)

En esta parte se puede ver los comandos utilizados para conectar los clientes al **SHARE**. Se utilizó Ubuntu Linux 20.04 para este laboratorio.

**Client-HQ-01V:**

```sh
root@CLIENT-HQ-01V:/home/godadmin# mount -t cifs -o username=administrator@zenprsolutions.local,password=XXXXXXXX //nas/shares /mnt/share/
root@CLIENT-HQ-01V:/home/godadmin# cd /mnt/share/
root@CLIENT-HQ-01V:/mnt/share# ls
RecApp-2021-02-20.webm   RecApp-2021-02-27.webm   Very_Big_File.iso   WSUS-Cleanup.ps1
root@CLIENT-HQ-01V:/mnt/share#
```

**Client-EDGE-01V:**

```sh
root@CLIENT-EDGE-01V:/home/godadmin# mount -t cifs -o username=administrator@zenprsolutions.local,password=XXXXXXXX //nas-edge/share_edge /mnt/share_edge/
root@CLIENT-EDGE-01V:/home/godadmin# cd /mnt/share_edge/
root@CLIENT-EDGE-01V:/mnt/share_edge# ls
RecApp-2021-02-20.webm   RecApp-2021-02-27.webm   Very_Big_File.iso   WSUS-Cleanup.ps1
root@CLIENT-EDGE-01V:/mnt/share_edge#
```

**Client-EDGE-02V:**

```sh
root@CLIENT-EDGE-02V:/home/godadmin# mount -t cifs -o username=administrator@zenprsolutions.local,password=XXXXXXXX //nas-edge/share_edge /mnt/share_edge/
root@CLIENT-EDGE-02V:/home/godadmin# cd /mnt/share_edge/
root@CLIENT-EDGE-02V:/mnt/share_edge# ls
RecApp-2021-02-20.webm   RecApp-2021-02-27.webm   Very_Big_File.iso   WSUS-Cleanup.ps1
root@CLIENT-EDGE-02V:/mnt/share_edge#
```

En esta parte se utilizó el comando **cp** para copiar el archivo **Very_Big_File.iso** desde la carpeta en el clúster hacia la carpeta local en el cliente. Para medir el tiempo de transferencia se utilizó el comando **time**.

**Client-HQ-01V:**

```sh
root@CLIENT-HQ-01V:/mnt/share# time cp Very_Big_File.iso /home/godadmin/

real	2m7.513s
user	0m0.016s
sys	0m6.236s
root@CLIENT-HQ-01V:/mnt/share#
```

**Client-EDGE-01V:**

```sh
root@CLIENT-EDGE-01V:/mnt/share_edge# time cp Very_Big_File.iso /home/godadmin/

real	4m2.391s
user	0m0.021s
sys	0m6.902s
root@CLIENT-EDGE-01V:/mnt/share_edge#
```

**Client-EDGE-02V:**

```sh
root@CLIENT-EDGE-02V:/mnt/share_edge# time cp Very_Big_File.iso /home/godadmin/

real	2m16.169s
user	0m0.054s
sys	0m6.128s
root@CLIENT-EDGE-02V:/mnt/share_edge# 
```

La tabla muestra el tiempo de transferencia de cada prueba realizada. Como pueden ver el cliente CLIENT-HQ-01V ubicado en el **site** Gurabo tiene acceso directo a la carpeta compartida desde el volumen origen teniendo un tiempo menor de transferencia **2m7.513s**. El cliente CLIENT-EDGE-01V esta conectado al **site** de Ponce utilizando la carpeta compartida del volumen de flexcache donde podemos ver que, al no estar el contenido inicialmente en el caché el tiempo de transferencia fue mayor **4m2.391s**. Esto es causado porque es necesario cargar toda la data desde el volumen de origen. Por último, el cliente CLIENT-EDGE-02V tuvo un tiempo de transferencia similar al CLIENT-HQ-01V, ya que el contenido del archivo **Very_Big_File.iso** se encuentra ya en el caché del volumen de flexcache.

| Client Name  |     Elapsed Time    |  Share  |  Description |
|:----------------------:|:--------------------:|:--------------------:|:--------------------:|
|   CLIENT-HQ-01V    | 2m7.513s| \\NAS\SHARE | Copy from OnPrem-HQ Share |
|   CLIENT-EDGE-01V    | 4m2.391s| \\NAS-EDGE\SHARE-EDGE | Copy from OnPrem-EDGE Share without content cached (Flexcache) |
|   CLIENT-EDGE-02V    | 2m7.513s|  \\NAS-EDGE\SHARE-EDGE | Copy from OnPrem-EDGE Share with content cached (Flexcache) |

**Hasta Luego Amigos!**

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/F1F8DEV80)

![Text](/img/looney-tunes-mejores-cortos-1-300x219.webp#center)
