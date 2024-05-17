---
title: 'VMware Skyline Health Diagnostics Deployment'
date: '2021-08-09T12:10:00-04:00'
tags:
    - VMware
---

Hello everyone

This time I come to show you the integration of `VMware Skyline Health Diagnostics` (VSHD) with VMware vCenter. I will also show you how to run the diagnostics to know how is the health of your Virtual infrastructure. VSHD is a self-diagnostic platform that allows to detect and solve problems in both vSphere and vSAN product line.

This tool provides recommendations in the form of Knowledge Base articles or links to troubleshooting procedures. vSphere administrators can use this tool to troubleshoot issues before contacting VMware Global Support.

![Text](/img/2021-08-08_14-29.webp)

Benefits:

- Based on symptoms, according to VMware VSHD automatically provides links to articles with steps to resolve the problem.
- Self-service improves the time to get recommendations to assist in problem resolution.
- Rapid repair to help recover the infrastructure from a failure and ensure that the business operates with less disruption.

To start we must access the following link where we can download the OVA file that allows us to manage the creation of the virtual machine where the VSHD services run.

- https://my.vmware.com/group/vmware/get-download?downloadGroup=SKYLINE_HD_VSPHERE


Once authenticated on the VMware portal you will be redirected to the area where you can download the file OVA.

![Text](/img/2021-07-25_11-37.webp)

Below is the process of creating the VM using the OVA template that you downloaded.

#### Installing VSHD through VMware vCenter

Start by using the `Deploy OVF Template` wizard where you upload the installation file by pressing `UPLOAD FILES`.

![Text](/img/2021-07-25_11-54.webp)

Set a name and select the folder in which the VM object will be created.

![Text](/img/2021-07-25_11-54_1.webp)

Select the `Compute Cluster` and press `NEXT`.

![Text](/img/2021-07-25_11-55.webp)

Confirm the information and press `NEXT`.

![Text](/img/2021-07-25_11-57.webp)

Accept the Licensing Agreement and press `NEXT`.

![Text](/img/2021-07-25_11-57_1.webp)

Select the storage location where the VM will be created and press `NEXT`.

![Text](/img/2021-07-25_11-57_2.webp)

Select the network to be used by the VM and press `NEXT`.

![Text](/img/2021-07-25_11-59.webp)

At this stage the unique properties of the VM such as the hostname, the password of the administration accounts and the IP address information are defined.

![Text](/img/2021-07-25_12-05.png)

After the information has been validated, click `FINISH` to complete the process.

![Text](/img/2021-07-25_12-06.webp)

The installation process can be monitored from the `Recent Tasks` tab.

![Text](/img/2021-07-25_12-06_1.webp)

An optional requirement is the association of a DNS name to the IP used in the installation process. In the following screen we can see how to register a DNS name `FQDN` using Powershell from a Windows console

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

Once the OVA file installation process is complete, you can proceed to power on the VM that will be used for the VSHD service.

![Text](/img/2021-07-30_10-52.webp)

When we turn on the VM we can see the DNS name information and the IP address that we previously configured.

![Text](/img/2021-07-30_12-14.webp)

With the IP address we can access the administration portal of the application using the following credentials:

- Username: `shd-admin`
- Password: `previously established`

![Text](/img/2021-07-30_15-01.webp)

The next step is to add the vCenter/ESXi information and corresponding credentials.

![Text](/img/2021-07-30_15-08.webp)

We can validate that the information entered is correct by pressing the `CHECK CONNECTION` button.

![Text](/img/2021-07-30_15-08_1.webp)

After validating the information and credentials, you proceed to run the diagnostic by pressing `RUN DIAGNOSTIC`.

![Text](/img/2021-07-30_15-10.webp)

In this screen we select the ESXi and vCenter servers you wish to scan and what type of plug-in to run during the collection of diagnostic information.

![Text](/img/2021-07-30_15-11-1.webp)

Optionally you can set up a `Tag` that can be used for an easy search of the diagnostics. Then you can click on `FINISH`.

![Text](/img/2021-07-30_15-13.webp)

In this next image we can see the progress of the diagnostic information gathering.

![Text](/img/2021-07-30_15-14_1-1.webp)

Additionally from the vCenter management console you can see a task related to the gathering process.

![Text](/img/2021-07-30_15-15.webp)

Once the process has finished you can view the status of the task by clicking on the `SHOW SUMMARY` button.

![Text](/img/2021-07-30_15-41-1.webp)

In this screen we can see that the tasks were executed without problems.

![Text](/img/2021-07-30_15-41_1.webp)

By pressing the `SHOW REPORT` button you can view the resulting reports.

![Text](/img/2021-07-30_15-42.webp)

To view the report press the eye icon as shown in the following image.

![Text](/img/2021-07-30_16-16-1.webp)

Here I show you several examples of the resulting report where you can pinpoint some of the problems with the infrastructure used as an example.

![Text](/img/2021-07-30_16-18.webp)

### Summary

In this lab, I installed and configured the VMware Skyline Health Diagnostics` (VSHD) which allows vSphere administrators to use this tool to troubleshoot issues before contacting VMware support. One nice thing about this tool is that it is freely available. I hope you liked this lab. If you have any doubts or questions about this lab, leave them in the comments. Regards.
