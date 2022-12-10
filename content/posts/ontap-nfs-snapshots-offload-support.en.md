---
title: 'vSphere 7 Update 2 NFS Array Snapshots Offload Support'
date: '2021-05-30T21:07:02-04:00'
author: 'Jonathan Colon Feliciano'
tags:
    - NetApp
    - VMware
---

The vSphere 7.0 U2 release provides the ability to use native snapshot when using the NFS protocol as the access mechanism. As described on the VMware blog:

> **NFS Improvements**
>
> NFS required a clone to be created first for a newly created VM and the subsequent ones could be offloaded to the array. With the release of vSphere 7.0 U2, we have enabled NFS array snapshots of full, non-cloned VMs to not use redo logs but instead use the snapshot technology of the NFS array in order to provide better snapshot performance. The improvement here will remove the requirement/limitation of creating a clone and enables the first snapshot also to be offloaded to the array.
>
> [What’s New in vSphere 7 Update 2 Core Storage](https://core.vmware.com/blog/whats-new-vsphere-7-update-2-core-storage)

![Text](/img/5066053.webp#center)

In this blog I explain the configuration needed to test this new feature. To start we should validate the prerequisites to be able to implement this solution. According to the VMware documentation portal the prerequisites are as follows:

- Verify that the NAS array supports the fast file clone operation with the VAAI NAS program.
- On your ESXi host, install vendor-specific NAS plug-in that supports the fast file cloning with VAAI.
- Follow the recommendations of your NAS storage vendor to configure any required settings on both the NAS array and ESXi.

The NFS configuration will be done in NetApp Ontap using the **“NetApp NFS Plug-in for VMware VAAI”** plugin that recently added native NFS snapshot offload support.

##### Note: The plug-in can be downloaded from the NetApp support portal at the following link [NetApp Support](https://mysupport.netapp.com/site/products/all/details/nfsplugin-vmware-vaai/downloads-tab/download/61278/2.0)

![Text](/img/NetApp-NFS-Plugin-1-1024x577.webp#center)

Once we are in the NetApp support portal we must download version 2.0 of the plugin as shown in the following image. To install the plugin we need to unzip the downloaded file and rename the file inside the folder named vib20 with the extension .vib to the new name **NetAppNasPlugin.vib**.

![Text](/img/2021-05-30_20-33-1024x707.webp#center)

In the next step I used the NetApp Ontap Tools to install the plugin but it can also be installed using VMware Lifecycle Manager.

![Text](/img/2021-05-30_20-49-1024x510.webp#center)

To install the plugin go to **[ONTAP tools => Settings => NFS VAAI tools]** and in the “Existing version:” section press **“BROWSE”** to select where the **“NetAppNasPlugin.vib”** file is stored. Once the file is located press **“UPLOAD”** to load and install the plugin.

![Text](/img/2021-05-30_20-36-1024x541.webp#center)

In this step we can see how to install the plugin to the ESXi servers by pressing the **“INSTALL”** button.

![Text](/img/2021-05-30_20-47-1024x674.webp#center)

The following image shows that the installation of the plugin was successful. An advantage of the new version of the plugin is that no reboot of the ESXi server is required.

![Text](/img/2021-05-30_20-48-1024x288.webp#center)

After installing the plugin we will proceed to validate that the Ontap Storage has support for VMware vStorage APIs for Array Integration (VAAI) features in the NFS environment. This can be verified with the command **vserver nfs show -fields vstorage**. As you can see the vStorage function is currently disabled in the SVM called NFS. To enable the vStorage function use the **vserver nfs modify -vstorage enabled** command.

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

The next requirement to be able to use native snapshot offload is the creation of an advanced setting in the VM configuration called snapshot.alwaysAllowNative. To add this value you have to go to the VM properties then to **[VM Options => Advanced => EDIT CONFIGURATION]**.

![Text](/img/2021-05-31_14-46-768x721.webp#center)

The following image shows the value of the **snapshot.alwaysAllowNative** variable that according to VMware documentation must have a value equal to **“TRUE”**. You can use the following link as reference [VMware Documentation](https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.vsphere.storage.doc/GUID-4F43BDCD-3748-4BE5-B57E-663211249EE3.html)

![Text](/img/2021-05-31_14-45-768x343.webp#center)

Now i start testing to validate that the native snapshot is working in Ontap. First i will create a snapshot with the **snapshot.alwaysAllowNative** function set to **FALSE**. Then i will make changes to the VM so that i can measure the speed of deleting and applying the snapshot changes to the base disk. In the example shown below the command **New-Snapshot** in PowerCLI was used to create a snapshot of the VM named **RocaWeb**

```sh
PS /home/rebelinux> get-vm -Name RocaWeb | New-Snapshot -Name PRE_Native_Array_Snapshot | Format-Table -Wrap -AutoSize  

Name                        Description                 PowerState 
----                        -----------                 ----------
PRE_Native_Array_Snapshot                               PoweredOff
PS /home/rebelinux> 
```

In this step a **10GB** file was copied to grow the snapshot so that i can measure how fast the changes are applied to the base disk when the snapshot is deleted. In this example the file **“RocaWeb_2-000001-delta.vmdk”** represents the delta where the snapshot changes are saved. This represents a traditional VMware snapshot.

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

The following image shows the time it took to apply the snapshot changes to the base disk when the snapshot was removed. In summary the operation took **9 minutes** in total using traditional VMware snapshot.

##### Note: Ontap simulator was used for this lab

![Text](/img/2021-05-31_20-05-1024x134.webp#center)

In this last example the **New-Snapshot** command was also used to create the snapshot but with the **snapshot.alwaysAllowNative** option set to **“TRUE”**. In that way i can test the use of Native Snapshot Offload in NFS. Here again, a **10GB** file was copied to the VM to grow the snapshot, so i can measure how quickly changes are applied to the base disk when the snapshot is deleted.

```text
PS /home/rebelinux> get-vm -Name RocaWeb | New-Snapshot -Name POST_Native_Array_Snapshot | Format-Table -Wrap -AutoSize

Name                       Description                        PowerState
----                       -----------                        ----------
POST_Native_Array_Snapshot                                    PoweredOff
PS /home/rebelinux> 
```

Here we can see that there is no **“-delta.vmdk”** file but there is a file named **“RocaWeb\_2-000001-flat.vmdk”** with the same size of **500GB** as the **“RocaWeb\_2-flat.vmdk”** file. This allows us to confirm that the NFS Native Snapshot Offload feature is enabled in Ontap.

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

The following image shows the time it took to apply the snapshot changes to the base disk when the snapshot was removed using the NFS Native Snapshot Offload. In summary, you can see that applying the snapshot changes to the base disk took no time at all to finish.

![Image](/img/2021-05-31_20-20-1024x125.webp#centered)

### Summary

NFS native snapshot offload operations are so fast because ONTAP references metadata when it creates a Snapshot copy, rather than copying data blocks, that why Snapshot copies are so efficient. Doing so eliminates the seek time that other systems incur in locating the blocks to copy, as well as the cost of making the copy itself.
