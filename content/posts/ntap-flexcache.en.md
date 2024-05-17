---
title: "Using Flexcache volumes to accelerate Windows shares data access"
date: 2021-05-27T08:39:31-04:00
author: 'Jonathan Colon Feliciano'
draft: false
tags:
  - "NetApp"
---

Starting In Ontap 9.8 release NetApp decided to add support for the Windows SMB protocol to the FlexCache technology. At last…..

In this blog, I will create a source volume as origin and a flexcache volume on a remote cluster. In the lab example I will also validate the benefit offered by the ability to extend a central CIFS share natively.

I used the NetApp documentation as a reference to define what a Flexcache volume is and what it is used for.

![Text](/img/f751fba8c585a669d5f9f36497c9c310-2.webp#center)

> A FlexCache volume is a sparsely populated volume that is backed by an origin volume. The FlexCache volume can be on the same cluster as or on a different cluster than that of the origin volume. The FlexCache volume provides access to data in the origin volume without requiring that all of the data be in the FlexCache volume. Starting in ONTAP 9.8, a FlexCache volume also supports SMB protocol.

##### [NetApp Documentation Portal](https://docs.netapp.com/ontap-9/topic/com.netapp.doc.pow-fc-mgmt/GUID-F4CE375E-DB00-403E-A20D-FB6CE6116D07.html)

To begin with, I used as a reference the following diagram showing an Active Directory domain with two sites named Gurabo and Ponce. Both sites have an Ontap cluster with version 9.8P4. Flexcache requires the creation of `Intercluster` type interfaces..

##### Note: The Ontap simulator was used for the lab

![Text](/img/Flexcache.webp#center)

The configuration I performed on the NAS-EDGE remote `vserver` was documented in case you are interested in seeing how to create a SVM from scratch.

In order to start with the lab it is needed to create an peer relationship between the local and remote vserver. To achieve this i use the command `vserver peer create` specifying the `applications` as `flexcache`.

##### Reference: [vserver peer create](http://docs.netapp.com/ontap-9/topic/com.netapp.doc.dot-cm-cmpr-940/vserver__peer__create.html)

##### Note: Previously, a cluster level peer relationship was performed with the `cluster peer create` command

```text
OnPrem-HQ::> vserver peer create -vserver NAS -peer-cluster OnPrem-EDGE -peer-vserver NAS-EDGE -applications flexcache 

Info: [Job 883] 'vserver peer create' job queued 
```

Once the peer relationship has been created between both vservers, you can continue to validate that the source volume was created as required. To validate the volume, the `volume show` command is used from the local cluster shell. In this lab I am going to use the volume named `share`.

```text
OnPrem-HQ::*> volume show -vserver NAS                
Vserver   Volume       Aggregate    State      Type       Size  Available Used%
--------- ------------ ------------ ---------- ---- ---------- ---------- -----
NAS       NAS_root     OnPrem_HQ_01_SSD_1 online RW      20MB    17.66MB    7%
NAS       share        OnPrem_HQ_01_SSD_1 online RW      10.3GB   8.04GB   20%
19 entries were displayed.

OnPrem-HQ::*> 
```

Once the volume is identified, you can create the flexcache volume using the command `volume flexcache create`. It is important to mention that flexcache technology uses `FlexGroup` as a dependency when creating a volume. It is for this reason that the aggr-list option is used to specify which aggregates will be used to create the `FlexGroup` type volumes.

```text
OnPrem-EDGE::> volume flexcache create -vserver NAS-EDGE -volume share_edge -aggr-list OnPrem_EDGE_0* -origin-vserver NAS -origin-volume share -size 10GB -junction-path /share_edge
[Job 595] Job succeeded: Successful.                                           

OnPrem-EDGE::>
```

From the remote cluster shell you can verify the created volume by using the `vol flexcache show` command.

```text
OnPrem-EDGE::> vol flexcache show
Vserver Volume      Size       Origin-Vserver Origin-Volume Origin-Cluster
------- ----------- ---------- -------------- ------------- --------------
NAS-EDGE share_edge 10GB       NAS            shares            OnPrem-HQ

OnPrem-EDGE::> 
```

From the local cluster shell you can see the source volume with the command `volume flexcache origin show-caches`. The flexcache volume previously created can be validated in the command result.

```text
OnPrem-HQ::*> volume flexcache origin show-caches
Origin-Vserver Origin-Volume  Cache-Vserver  Cache-Volume  Cache-Cluster
-------------- -------------- -------------- ------------- --------------
NAS            share         NAS-EDGE       share_edge    OnPrem-EDGE
1 entries were displayed.

OnPrem-HQ::*> 
```

Now i proceed to share the share_edge cache volume using the SMB protocol. The command `vserver cifs share create` is used with the option of `-path /share_edge` to specify the `junction-path` of the flexclone volume.

```text
OnPrem-EDGE::> vserver cifs share create -vserver NAS-EDGE -share-name share_edge -path /share_edge

OnPrem-EDGE::>
```

Now you can see that the `Share` was created in the `share_edge` volume.

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

I have used the smbmap tool to validate that the shared folder can be accessed over the network.

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

In the performed test I copied the `Very_Big_File.iso` file to each site cluster `SHARE` volume.

##### Note: I modified the original diagram to show how the clients are connected

![Text](/img/Flexcache2.webp#center)

In this section you can see the commands used to connect the clients to the `SHARE` volume.

##### Note: Ubuntu Linux 20.04 was used for this lab scenario

`Client-HQ-01V:`

```sh
root@CLIENT-HQ-01V:/home/godadmin# mount -t cifs -o username=administrator@zenprsolutions.local,password=XXXXXXXX //nas/shares /mnt/share/
root@CLIENT-HQ-01V:/home/godadmin# cd /mnt/share/
root@CLIENT-HQ-01V:/mnt/share# ls
RecApp-2021-02-20.webm   RecApp-2021-02-27.webm   Very_Big_File.iso   WSUS-Cleanup.ps1
root@CLIENT-HQ-01V:/mnt/share#
```

`Client-EDGE-01V:`

```sh
root@CLIENT-EDGE-01V:/home/godadmin# mount -t cifs -o username=administrator@zenprsolutions.local,password=XXXXXXXX //nas-edge/share_edge /mnt/share_edge/
root@CLIENT-EDGE-01V:/home/godadmin# cd /mnt/share_edge/
root@CLIENT-EDGE-01V:/mnt/share_edge# ls
RecApp-2021-02-20.webm   RecApp-2021-02-27.webm   Very_Big_File.iso   WSUS-Cleanup.ps1
root@CLIENT-EDGE-01V:/mnt/share_edge#
```

`Client-EDGE-02V:`

```sh
root@CLIENT-EDGE-02V:/home/godadmin# mount -t cifs -o username=administrator@zenprsolutions.local,password=XXXXXXXX //nas-edge/share_edge /mnt/share_edge/
root@CLIENT-EDGE-02V:/home/godadmin# cd /mnt/share_edge/
root@CLIENT-EDGE-02V:/mnt/share_edge# ls
RecApp-2021-02-20.webm   RecApp-2021-02-27.webm   Very_Big_File.iso   WSUS-Cleanup.ps1
root@CLIENT-EDGE-02V:/mnt/share_edge#
```

In this last step the `cp` command was used to copy the `Very_Big_File.iso` file from the cluster to a local folder on the client. To measure the elapsed time of transfer the Linux `time` command was used.

`Client-HQ-01V:`

```sh
root@CLIENT-HQ-01V:/mnt/share# time cp Very_Big_File.iso /home/godadmin/

real	2m7.513s
user	0m0.016s
sys	0m6.236s
root@CLIENT-HQ-01V:/mnt/share#
```

`Client-EDGE-01V:`

```sh
root@CLIENT-EDGE-01V:/mnt/share_edge# time cp Very_Big_File.iso /home/godadmin/

real	4m2.391s
user	0m0.021s
sys	0m6.902s
root@CLIENT-EDGE-01V:/mnt/share_edge#
```

`Client-EDGE-02V:`

```sh
root@CLIENT-EDGE-02V:/mnt/share_edge# time cp Very_Big_File.iso /home/godadmin/

real	2m16.169s
user	0m0.054s
sys	0m6.128s
root@CLIENT-EDGE-02V:/mnt/share_edge# 
```

Further on, the following table shows the elapsed time transfer of each test performed. As you can see the CLIENT-HQ-01V located at the Gurabo site has direct access to the shared folder at the origin volume helping to achieve a lower transfer time of `2m7.513s`. The CLIENT-EDGE-01V is connected to the Ponce site using the shared folder from the flexcache volume where you can see that since the content was not initially in the cache the transfer time was higher `4m2.391s`. This behavior is due to the need to load the entire contents of `Very_Big_File.iso` from the source volume over the InterCluster LIF connection. Finally the CLIENT-EDGE-02V had a transfer time similar to CLIENT-HQ-01V (2m16.169s) since the content of the `Very_Big_File.iso` file is already in the cache of the flexcache volume.

| Client Name  |     Elapsed Time    |  Share  |  Description |
|:----------------------:|:--------------------:|:--------------------:|:--------------------:|
|   CLIENT-HQ-01V    | 2m7.513s| \\NAS\SHARE | Copy from OnPrem-HQ Share |
|   CLIENT-EDGE-01V    | 4m2.391s| \\NAS-EDGE\SHARE-EDGE | Copy from OnPrem-EDGE Share without content cached (Flexcache) |
|   CLIENT-EDGE-02V    | 2m7.513s|  \\NAS-EDGE\SHARE-EDGE | Copy from OnPrem-EDGE Share with content cached (Flexcache) |

`Hasta Luego Amigos!`

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/F1F8DEV80)

![Text](/img/looney-tunes-mejores-cortos-1-300x219.webp#center)
