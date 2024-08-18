---
title: 'HomeLab: Object First OOTBI VSA initial Setup'
date: '2024-08-14T12:10:00-04:00'
tags:
    - VMware
    - OOTBI
    - ObjectFirst
    - Veeam
---

Hello everyone!

In this opportunity I am going to show you the initial implementation of the virtual storage appliance from the company `Object First`. I will also show you how to access the Web management portal.

This `OOTBI` appliance provides secure storage for the `Veeam Backup & Replication` backup application. Here is the `Object First` web link:

- <https://objectfirst.com/object-storage/>

According to Object First the `OOTBI` offers:

> Ransomware-proof and immutable out-of-the-box storage, `OOTBI` offers secure, simple and powerful backup storage.

Benefits:

- A storage 100% developed for Veeam.
- `Zero Trust Data Resilience (ZTDR)` architecture.
- S3 native object locking. Secure, resilient and purpose-built: data is isolated and untouchable.
- Object-based systems anticipate horizontal scalability, which never sacrifices performance for capacity.
- Bugs, quirks, or limitations of a traditional file system are absent in object-based architectures.
- Secure and seamless on-premises-to-cloud data copy in S3 format with no overhead.
- The S3 protocol has guaranteed data delivery (unlike SMB or NFS), so businesses get end-to-end data reliability.
- Natively solves many of the availability needs organizations consider when considering backup storage and the 3-2-1 backup rule.

To start download the OVA file from the following link:

<https://objectfirst.com/virtual-storage-appliance/setup-file-download/>

![Text](/img/2024/homelab-ootbi-initial-setup/OOTBI-00.webp)

Below is the process of creating the VM using the template downloaded from the `Object First` web site.

#### Deploying `OOTBI` through VMware vCenter

In this section a DNS record is created that will be used to do the name resolution of the storage appliance.

##### Creating a type A DNS resources record with Powershell

```powershell
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

PS C:\Users\Administrator> Add-DnsServerResourceRecordA -Name veeam-ootbi-00v -IPv4Address 192.168.5.51 -ZoneName pharmax.local -CreatePtr -AllowUpdateAny

PS C:\Users\Administrator> Get-DnsServerResourceRecord -Name veeam-ootbi-00v -ZoneName pharmax.local

HostName                  RecordType Type       Timestamp            TimeToLive      RecordData
--------                  ---------- ----       ---------            ----------      ----------
veeam-ootbi-00v           A          1          0                    01:00:00        192.168.5.50

PS C:\Users\Administrator> Add-DnsServerResourceRecordA -Name veeam-ootbi-01v -IPv4Address 192.168.5.51 -ZoneName pharmax.local -CreatePtr -AllowUpdateAny

PS C:\Users\Administrator> Get-DnsServerResourceRecord -Name veeam-ootbi-01v -ZoneName pharmax.local

HostName                  RecordType Type       Timestamp            TimeToLive      RecordData
--------                  ---------- ----       ---------            ----------      ----------
veeam-ootbi-01v           A          1          0                    01:00:00        192.168.5.51

PS C:\Users\Administrator>
```

##### Validating DNS host names with Powershell

```powershell
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

Try the new cross-platform PowerShell https://aka.ms/pscore6

PS C:\Users\jocolon> Resolve-DnsName -Name 192.168.5.50

Name                           Type   TTL   Section    NameHost
----                           ----   ---   -------    --------
50.5.168.192.in-addr.arpa      PTR    3600  Answer     VEEAM-OOTBI-00V.pharmax.local


PS C:\Users\jocolon> Resolve-DnsName -Name 192.168.5.51

Name                           Type   TTL   Section    NameHost
----                           ----   ---   -------    --------
51.5.168.192.in-addr.arpa      PTR    3600  Answer     VEEAM-OOTBI-01V.pharmax.local


PS C:\Users\jocolon>
```

Now that everything is done with the DNS name we can move on to deploy the appliance. In this case log in to vCenter to deploy the OVA file.

#### Step 1: OVF template deployment

![Text](/img/2024/homelab-ootbi-initial-setup/OOTBI-01.webp)

#### Step 2: Select the OVA file location

![Text](/img/2024/homelab-ootbi-initial-setup/OOTBI-02.webp)

#### Step 3: Configure the Name and the folder location of the VM

![Text](/img/2024/homelab-ootbi-initial-setup/OOTBI-11.webp)

![Text](/img/2024/homelab-ootbi-initial-setup/OOTBI-03.webp)

#### Step 4: vSphere DataCenter selection

![Text](/img/2024/homelab-ootbi-initial-setup/OOTBI-04.webp)

#### Step 5: Review details

![Text](/img/2024/homelab-ootbi-initial-setup/OOTBI-05.webp)

#### Step 6: Select the virtual disk type and the Datastore destination

![Text](/img/2024/homelab-ootbi-initial-setup/OOTBI-06.webp)

#### Step 7: Select the Network PortGroup where the VM will be connected

![Text](/img/2024/homelab-ootbi-initial-setup/OOTBI-07.webp)
![Text](/img/2024/homelab-ootbi-initial-setup/OOTBI-08.webp)

#### Step 8: Review the configuration

![Text](/img/2024/homelab-ootbi-initial-setup/OOTBI-09.webp)

After finishing the VM creation process, proceed to power on the VM to perform the initial configuration.

![Text](/img/2024/homelab-ootbi-initial-setup/OOTBI-12.webp)

#### OOTBI initial configuration

Here in this page you must accept the license to be able to continue...

![Text](/img/2024/homelab-ootbi-initial-setup/OOTBI-14.webp)

Next, proceed to create a new cluster.

![Text](/img/2024/homelab-ootbi-initial-setup/OOTBI-15.webp)

In this section the parameters for connecting the OOTBI to our network are configured.

![Text](/img/2024/homelab-ootbi-initial-setup/OOTBI-16.webp)

Proceed with the configuration of the `HostName` of the VM.

![Text](/img/2024/homelab-ootbi-initial-setup/OOTBI-17.webp)

In this dialog the cluster name and IP address can be specified.

![Text](/img/2024/homelab-ootbi-initial-setup/OOTBI-20.webp)

** Note: This IP address must be different from the one configured for the node.

Then we go on to specify the password for the “default” account `objectfirst`.

![Text](/img/2024/homelab-ootbi-initial-setup/OOTBI-21.webp)

After completing all these steps, the new cluster will be configured with the parameters you set.

![Text](/img/2024/homelab-ootbi-initial-setup/OOTBI-22.webp)

Finally the cluster was successfully created! Yay.

![Text](/img/2024/homelab-ootbi-initial-setup/OOTBI-23.webp)

On this next screen it allows you to choose whether Telemetry is enabled or not.

![Text](/img/2024/homelab-ootbi-initial-setup/OOTBI-24.webp)

**  Note: These parameters can be set later through the Web portal.

In this screen the version and the IP address of the VM can be seen. With this information the OOTBI management Web Portal can be accessed.

![Text](/img/2024/homelab-ootbi-initial-setup/OOTBI-28.webp)

Finally, I will show you how to access the `Object First OOTBI` management portal. To access the portal it is required to use the `objectfirst` user and the password that we established in the cluster configuration.

![Text](/img/2024/homelab-ootbi-initial-setup/OOTBI-26.webp)

Below is the Management portal!

![Text](/img/2024/homelab-ootbi-initial-setup/OOTBI-27.webp)

### That's all folks

In this lab I installed and configured the `Object First OOTBI` virtual appliance that allows Veeam administrators to use this appliance as an S3-like repository. I hope you liked this lab. If you have any doubts or questions about this lab, leave them in the comments.

Hasta la Próxima!

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/F1F8DEV80)

![Text](/img/hasta-luego-5937ba.webp#center)
