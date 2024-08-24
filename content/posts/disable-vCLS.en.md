---
title: 'HomeLab: How to disable vSphere Cluster Services (vCLS)'
author: 'Jonathan Colon Feliciano'
date: 2020-12-18T08:39:31-04:00
draft: false
tags:
  - 'VMware'
---

In vSphere 7 update 1 VMware added a new capability for `Distributed Resource Scheduler (DRS)` technology consisting of three VMs called agents. The agent VMs form the quorum state of the cluster and have the ability to self-healing. So if you turn off or delete the VMs called vCLS the vCenter server will turn the VMs back on or re-create the VMs again. For HomeLab purposes this new feature consumes CPU resource, Memory and disk space which although minimal is not worth having a configuration that adds nothing to a test and development environment.

![Text](/img/25079036c801ac924d3ff7d4cb3b9438.webp#center)

In this blog I will be showing how to remove this feature, but it is important to emphasize not to implement this change in production environments. The following image shows the minimum amount of vCLS VMs used in vCenter 7U1.

##### Note: vSphere DRS depends on the status of vCLS services as of vSphere 7.0 Update 1

![Text](/img/2021-05-30_12-21.webp#center)

As you can see in my case the RegionA01-COMP cluster is composed of two ESXi servers where there are three vCLS VMs.

![Text](/img/2021-05-29_22-52-1024x649.webp#center)

The first thing you need to do is to identify the cluster domain ID that is required to be able to add an advanced value in the vCenter configuration. This ID can be identified in two ways: From vCenter or using Powershell with PowerCLI.

vCenter: In this part you can get the cluster ID by navigating to the [Hosts and Clusters] tab then select the cluster you are going to edit where you can see that the ID is in the URL address of the browser.

![Text](/img/2021-05-30_00-11-1024x309.webp#center)

PowerCLI: Using Powershell with the `VMware.PowerCLI` module you can obtain the cluster ID by invoking the `Get-Cluster` command. As shown in the result of the command you can see the value of the ID `domain-c81` for the cluster named RegionA01-COMP.

```text
PS /home/rebelinux> Get-Cluster RegionA01-COMP | FL

ParentId                        : Folder-group-h23
ParentFolder                    : host
HAEnabled                       : True
HAAdmissionControlEnabled       : True
HAFailoverLevel                 : 1
HARestartPriority               : Medium
HAIsolationResponse             : DoNothing
VMSwapfilePolicy                : WithVM
DrsEnabled                      : True
DrsMode                         : FullyAutomated
DrsAutomationLevel              : FullyAutomated
CryptoMode                      : OnDemand
CollectiveHostManagementEnabled : False
Name                            : RegionA01-COMP
ExtensionData                   : VMware.Vim.ClusterComputeResource
Id                              : ClusterComputeResource-(domain-c81)

PS /home/rebelinux>
```

Once we have the cluster ID which in my case is `domain-c81` we proceed to add the configuration in vCenter by navigating to `[Advanced Settings => Configure => Edit Settings]`.

![Text](/img/2021-05-29_21-47-1024x599.webp#center)

In this screen add the `config.vcls.clusters.domain-cID.enabled` value which in my case would be `config.vcls.clusters.domain-c81.enabled` with the value of `false` in the `Value` field.

![Text](/img/2021-05-29_22-53-1024x871.webp#center)

Once the value is added the VMs of type vCLS will be removed from the cluster as you can see in the image as the VMs no longer exist.

![Text](/img/2021-05-29_23-12-1024x648.webp#center)

In the following image you can see the tasks performed by vCenter to remove the vCLS VMs.

![Text](/img/2021-05-29_23-14-1024x426.webp#center)

### Hasta luego!!!

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/F1F8DEV80)

![Text](/img/hasta-luego-5937ba.webp#center)
