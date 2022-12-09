---
title: "NetApp Aggregate Encryption (NAE) in ONTAP"
date: 2021-05-20T08:39:31-04:00
draft: false
tags:
  - "NetApp"
  - "KMS"
  - "Encryption"
---

Previously in a [post](http://192.168.7.40/2021/05/16/netapp-volume-encryption-setup-external-key-manager/) I explained how to set up an encrypted volume using an encryption key manager (KMS) specifically from the company [HyTrust](https://www.hytrust.com/products/keycontrol/). In this specific case each volume is encrypted individually using independent keys. A disadvantage of this method is that it affects the possibility of increasing the efficiency levels of data reduction such as compression, compaction and de-duplication (cross-volume-dedupe).

![Text](/img/NVE-vs-NAE.webp#center)

To eliminate this disadvantage the NetApp gurus came up with the idea of applying the encryption feature at the aggregate level by allowing volumes residing within the same aggregate to share the encryption key. This technology is known as **“NetApp Aggregate Encryption” (NAE)**. This allows customers the option to take advantage of storage efficiency technologies in conjunction with the encryption process.

Now it’s time to talk about how we can create an encrypted aggregate in Ontap but first of all… What is an aggregate within Ontap?

Using the NetApp Knowledge Base portal as a reference:

> An aggregate is a collection of disks (or partitions) arranged into one or more RAID groups.  It is the most basic storage object within ONTAP and is required to allow for the provisioning of space for connected hosts.

##### [NetApp Knowledge Base](https://kb.netapp.com/Advice_and_Troubleshooting/Data_Storage_Software/ONTAP_OS/What_is_an_aggregate%3F)

![Text](/img/2021-05-20_21-21-1024x746.webp#center)

**Step 1:** Validate Ontap requirements.

In order to use the encryption option at the aggregate level, it is required to have a version of Ontap 9.6 or higher also make sure the required licenses are installed in the cluster. In this case we use the command **version** to validate the current version of the cluster and the command **license show -package VE** to display the license information.

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

##### Note: I previously done the external KMS setup in Ontap. [Link](http://192.168.7.40/2021/05/16/netapp-volume-encryption-setup-external-key-manager/)

**Step 2:** Validate the available “Spare” discs.

To begin with, there are two ways to encrypt an aggregate; initially when it is created or the live conversion of an existing one. Initially I will be creating a new aggregate and then in another tutorial I will show you how easy is to convert an existing one. To create an aggregate you need to have disk drives available or in the “spare” state as NetApp commonly calls it.

The **storage aggregate show-spare-disks** command allows us to see how many partitioned disks are available on the node where i will create the new encrypted aggregate. In this particular case you can see that there are 24 partitioned disks using the “Root-Data1-Data2” option. To learn more about this disk strategy please follow the link below:

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

**Step 3:** Create an encrypted aggregate.

To create the encrypted aggregate we use the **storage aggregate create** command with the option **encrypt-with-aggr-key true** turned on. In this case we create a secure aggregate composed of 23 disks “partitions”.

##### Note: For this example the RAID type **Dual Parity** was used

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

Once created it is required to validate the aggregate, to do so you must use the command **storage aggregate show** by filtering the result with the **encrypt-with-aggr-key** option.

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

In the command result you can see that the aggregate was created with encryption capability enabled.

**Step 4:** Create a volume within the encrypted aggregate.

Unlike volume-level encryption NVE, when using aggregate-level encryption it is not required to specify the encrypt option to create the volume. The command **vol create** creates an encrypted volume by default when the volume resides in an aggregate configured with NAE.

```bash
OnPrem-HQ::> vol create -vserver SAN -volume Secure_Vol -aggregate OnPrem_HQ_01_SSD_1 -size 10GB -space-guarantee none 
[Job 818] Job succeeded: Successful                                            

OnPrem-HQ::>
```

By using the **vol show** command with the **encryption-state full** filter option you can see the volume was created encrypted by default.

```bash
OnPrem-HQ::> vol show -encryption-state full -aggregate OnPrem_HQ_01_SSD_1 -fields Vserver,Volume,encrypt,encryption-type,encryption-state 
vserver volume     encryption-type encrypt encryption-state 
------- ---------- --------------- ------- ---------------- 
SAN     Secure_Vol aggregate       true    full             

OnPrem-HQ::>
```

### Summary

In this tutorial I showed you how to configure the aggregate level encryption technology within Ontap that allows us to use a unique security key to create encrypted volumes. This allows us to use data reduction technologies in conjunction with security mechanisms that enhance or strengthen the security posture of the organization.

**Hasta luego!!!**

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/F1F8DEV80)

![Text](/img/hasta-luego-5937ba.webp#center)
