---
title: 'HomeLab: Using vCheck for vSphere Infrastructure Health Accessment'
date: '2021-06-08T21:40:47-04:00'
author: 'Jonathan Colon Feliciano'
tags:
    - VMware
---

In this opportunity I come to show you how to download and use the vCheck tool that is used to validate the health status of the VMware vSphere infrastructure. This tool is developed by [Alan Renouf](http://www.virtu-al.net) as a mechanism to identify possible failures or misconfiguration in the vSphere implementation. To learn more about this tool I will use the vCheck documentation as a reference:

> This script picks on the key known issues and potential issues scripted as plugins for various technologies written as Powershell scripts and reports it all in one place so all you do in the morning is check your email.
>
> #### [vCheck Github Page](https://github.com/alanrenouf/vCheck-vSphere)

![Text](/img/health-check-300x300.webp#center)

In this area I present to you what is checked when using vCheck.

- General Details
- Number of Hosts
- Number of VMs
- Number of Templates
- Number of Clusters
- Number of Datastores
- Number of Active VMs
- Number of Inactive VMs
- Number of DRS Migrations for the last days
- Snapshots over x Days old
- Datastores with less than x% free space
- VMs created over the last x days
- VMs removed over the last x days
- VMs with No Tools
- VMs with CD-Roms connected
- VMs with Floppy Drives Connected
- VMs with CPU ready over x%
- VMs with over x amount of vCPUs
- List of DRS Migrations
- Hosts in Maintenance Mode
- Hosts in disconnected state
- NTP Server check for a given NTP Name
- NTP Service check
- vmkernel warning messages ov the last x days
- VC Error Events over the last x days
- VC Windows Event Log Errors for the last x days with VMware in the details
- VC VMware Service details
- VMs stored on datastores attached to only one host
- VM active alerts
- Cluster Active Alerts
- If HA Cluster is set to use host datastore for swapfile, check the host has a swapfile location set
- Host active Alerts
- Dead SCSI Luns
- VMs with over x amount of vCPUs
- vSphere check: Slot Sizes
- vSphere check: Outdated VM Hardware (Less than V7)
- VMs in Inconsistent folders (the name of the folder is not the same as the name)
- VMs with high CPU usage
- Guest disk size check
- Host over committing memory check
- VM Swap and Ballooning
- ESXi hosts without Lockdown enabled
- ESXi hosts with unsupported mode enabled
- General Capacity information based on CPU/MEM usage of the VMs
- vSwitch free ports
- Disk over commit check
- Host configuration issues
- VCB Garbage (left snapshots)
- HA VM restarts and resets
- Inaccessible VMs

It is important to mention that vCheck has support for other products as shown in the following image:

![Text](/img/2021-06-10_00-24-768x405.webp#center)

To get started you need to download the tool from the [Github](https://github.com/alanrenouf/vCheck-vSphere) portal where it is continuously developed. To download vCheck directly click on the following link [Download](https://github.com/alanrenouf/vCheck-vSphere)

Once the script is downloaded, it is necessary to unzip it.

![Text](/img/2021-06-10_08-51-1024x551.webp#center)

The first time vCheck is run it will start the configuration process, this configuration allows you to create a template with the information that will be used for all future runs of the program. To modify these parameters again you can use the **-config** option as follows:

```powershell
[blabla@blabla ~]$ pwsh #Powershell core on Linux :)
PowerShell 7.1.3
Copyright (c) Microsoft Corporation.

https://aka.ms/powershell
Type 'help' to get help.

PS /home/blabla/vCheck> ./vCheck.ps1 -config
```

In this area I demonstrate the vCheck configuration process.

{{< collapse summary="Configuration process example" >}}

```text
PS /home/blabla/vCheck> ./vCheck.ps1 -config
WARNING: GlobalVariables
# Report header [vCheck]: 
# Would you like the report displayed in the local browser once completed ? [$true]: 
# Display the report even if it is empty? [$true]: 
# Use the following item to define if an email report should be sent once completed [$false]: 
# Please Specify the SMTP server address (and optional port) [servername(:port)] [mysmtpserver.mydomain.local]: 
# Would you like to use SSL to send email? [$false]: 
# Please specify the email address who will send the vCheck report [me@mydomain.local]: 
# Please specify the email address(es) who will receive the vCheck report (separate multiple addresses with comma) [me@mydomain.local]: 
# Please specify the email address(es) who will be CCd to receive the vCheck report (separate multiple addresses with comma) []: 
# Please specify an email subject [$Server vCheck Report]: 
# Send the report by e-mail even if it is empty? [$true]: 
# If you would prefer the HTML file as an attachment then enable the following: [$false]: 
# Set the style template to use. [Clarity]: 
# Do you want to include plugin details in the report? [$true]: 
# List Enabled plugins first in Plugin Report? [$true]: 
# Set the following setting to $true to see how long each Plugin takes to run as part of the report [$true]: 
# Report on plugins that take longer than the following amount of seconds [30]: 
WARNING: Connection settings for vCenter
# Please Specify the address (and optional port) of the vCenter server to connect to [servername(:port)] [vcsa.local.lab]: vcenter-01v.pharmax.local
WARNING: General Information
# Set the number of days of DRS Migrations to report and count on [1]: 
# Set the number of days of Storage DRS Migrations to report and count on [1]: 
WARNING: Checking VI Events
# Set the number of days of VC Events to check for errors [1]: 
WARNING: Windows vCenter Error Event Logs
# Set the number of days of VC Events to check for errors [1]: 
# Set the number of days of VC Event Logs to check for warnings and errors [1]: 
WARNING: Windows vCenter Error Event Logs
# Set the number of days of VC Events to check for errors [1]: 
# Set the number of days of VC Event Logs to check for warnings and errors [1]: 
WARNING: vCenter Sessions Age
# Enter maximum vCenter session length in hours [48]: 
# Enter minimum vCenter session length in minutes (IdleMinutes) [10]: 
# Do not report on usernames that are defined here (regex) [DOMAIN\\user1|DOMAIN\\user2]: 
WARNING: vCenter License Report
# Display Eval licenses? [$true]: 
WARNING: HA configuration issues
# HA Configuration Issues, do not report on any Clusters that are defined here [Example_Cluster_*|Test_Cluster_*]: 
# HA should be set to ... [$true]: 
# HA host monitoring should be set to ... [$true]: 
# HA Admission Control should be set to ... [$true]: 
WARNING: 
HA VMs restarted
# HA VM restart day(s) number [5]: 
WARNING: 
DRS & SDRS Migrations
# Set the number of days of DRS Migrations to report and count on [1]: 
# Set the number of days of Storage DRS Migrations to report and count on [1]: 
WARNING: Cluster Slot Sizes
# Minimum number of slots available in a cluster [10]: 
WARNING: Datastore Consistency
# Do not report on any Datastores that are defined here (Datastore Consistency Plugin) [local*|datastore*]: 
WARNING: Clusters with DRS disabled
# Clusters with DRS Disabled, do not report on any Clusters that are defined here [VM1_*|VM2_*]: 
WARNING: QuickStats Capacity Planning
# Max CPU usage for non HA cluster [0.6]: 
# Max MEM usage for non HA cluster [0.6]: 
WARNING: s/vMotion Information
# Set the number of days to go back and check for s/vMotions [5]: 
# Include vMotions in report [$true;]: 
# Include Storage vMotions in report [$true;]: 
WARNING: DRS Rules
# Display VM affinity rules? [$true]: 
# Display VM anti-affinity rules? [$true]: 
# Display HOSTaffinity rules? [$true]: 
# Set DRS Rule name exception (regex) [ExcludeMe]: 
WARNING: Hosts Overcommit state
# Return results in GB or MB? [GB]: 
WARNING: Active Directory Authentication
# Show "OK" results? [$false]: 
# Expected Domain name [mydomain.local]: 
# Expected Admin Group [ESX Admins]: 
WARNING: NTP Name and Service
# The NTP server which should be set on your hosts (comma-separated) [pool.ntp.org,pool2.ntp.org]: 
WARNING: VMKernel Warnings
# Disabling displaying Google/KB links in order to have wider message column [$true]: 
WARNING: Syslog Name
# The Syslog server(s) which should be set on your hosts (comma-separated) [udp://syslogserver]: 
WARNING: Disk Max Total Latency
# Disk Max Total Latency Settings in Milliseconds [50]: 
# Disk Max Total Latency range to inspect (1-24) Hours [24]: 
WARNING: Lost Access to Volume
# Set the number of days of Lost Action Volume to report and count on [1]: 
WARNING: Check LUNS have the recommended number of paths
# Set the Recommended number of paths per LUN [2]: 
WARNING: ESXi Inode Exhaustion
# Set the ESXi filesystem free Inode threshold in percent [40]: 
WARNING: Host Profile Compliance
# Show detailed information in report [$true]: 
# Show compliant servers [$false]: 
WARNING: Hosts with Upcoming Certificate Expiration
# How many days to warn before cert expiration (Default 60) [60]: 
WARNING: Host Multipath Policy
# The Multipath Policy (PSP Plugin) your hosts should be configured to use [VMW_PSP_RR]: 
WARNING: Host Power Management Policy
# Which power management policy should your hosts use? For Balanced enter "dynamic" (this is the ESXi default policy), for High Performance enter "static", for Low power enter "low". [dynamic]: 
WARNING: Datastore Information
# Set the warning threshold for Datastore % Free Space [15]: 
# Do not report on any Datastores that are defined here (Datastore Free Space Plugin) [local]: 
WARNING: Number of VMs per Datastore
# Max number of VMs per Datastore [5]: 
# Exclude these datastores from report [ExcludeMe]: 
WARNING: Datastore OverAllocation
# Datastore OverAllocation % [50]: 
# Exclude these datastores from report []: 
WARNING: Datastores with Storage IO Control Disabled
# Do not report on any Datastores that are defined here (Storage IO Control disabled Plugin) [local]: 
WARNING: sDRS VM Behavior not Default
# Exclude these VMs from report []: 
WARNING: VSAN Datastore Capacity
# Set the warning threshold for VSAN Datastore % Free Space [15]: 
WARNING: VSAN Configuration Maximum Disk Group Per Host Report
# Percentage threshold to warn? [80]: 
WARNING: VSAN Configuration Maximum Magnetic Disks Per Disk Group Report
# Percentage threshold to warn? [50]: 
WARNING: VSAN Configuration Maximum Total Magnetic Disks In All Disk Groups Per Host Report
# Percentage threshold to warn? [50]: 
WARNING: VSAN Configuration Maximum Components Per Host Report
# Percentage threshold to warn? [50]: 
WARNING: VSAN Configuration Maximum Hosts Per VSAN Cluster Report
# Percentage threshold to warn? [45]: 
WARNING: VSAN Configuration Maximum VMs Per Host Report
# Percentage threshold to warn? [50]: 
WARNING: VSAN Configuration Maximum VMs Per VSAN Cluster Report
# Percentage threshold to warn? [50]: 
WARNING: Checking Standard vSwitch Ports Free
# vSwitch Port Left [5]: 
WARNING: Checking Distributed vSwitch Port Groups for Ports Free
# Distributed vSwitch PortGroup Ports Left [10]: 
WARNING: vSwitch Security
# Warn for AllowPromiscuous enabled? [$true]: 
# Warn for ForgedTransmits enabled? [$true]: 
# Warn for MacChanges enabled? [$true]: 
WARNING: Snapshot Information
# Set the warning threshold for snapshots in days old [14]: 
# Set snapshot name exception (regex) [ExcludeMe]: 
# Set snapshot description exception (regex) [ExcludeMe]: 
# Set snapshot creator exception (regex) [ExcludeMe]: 
WARNING: Map disk region event
# Set the number of days to show Map disk region event for [5]: 
WARNING: Created or cloned VMs
# Set the number of days to show VMs created for [5]: 
WARNING: Removed VMs
# Set the number of days to show VMs removed for [5]: 
WARNING: VMs with over CPU Count
# Define the maximum amount of vCPUs your VMs are allowed [2]: 
WARNING: VMs restarted due to Guest OS Error
# HA VM reset day(s) number due to Guest OS error [5]: 
WARNING: Guests with less than X MB free
# VM Disk space left, set the amount you would like to report on MBFree [1024]: 
# VM Disk space left, set the amount you would like to report on MBDiskMinSize [1024]: 
WARNING: Checking VM Hardware Version
# Hardware Version to check for at least [8]: 
# Adding filter for dsvas, vShield appliances or any other vms that will remain on a lower HW version [vShield*|dsva*]: 
WARNING: VMs in inconsistent folders
# Specify which Datastore(s) to filter from report [local]: 
WARNING: No VM Tools
# Do not report on any VMs who are defined here (regex) []: 
WARNING: VM Tools Issues
# VM Tools Issues, do not report on any VMs who are defined here []: 
WARNING: Removable Media Connected
# VMs with removable media not to report on []: 
WARNING: Single Storage VMs
# Local Stored VMs, do not report on any VMs who are defined here [Template_*|VDI*]: 
# Local Datastores, do not report on any VMs within these datastores [Local|datastore1]: 
WARNING: VM CPU %RDY
# CPU ready on VMs should not exceed [10.0]: 
WARNING: VM CPU Usage
# VM Not to go over the following amount of CPU [75]: 
# VM CPU not allowed to go over the previous amount for how many days? [1]: 
WARNING: Backup Garbage
# Names used in backup product snapshots. Defaults include VCB, Veeam, NetBackup, and Commvault [VCB|Consolidate|veeam|NBU_SNAPSHOT|GX_BACKUP]: 
WARNING: Find VMs with thick or thin provisioned vmdk
# Report on disk formats that are not "thin" or "thick", which format is not allowed? [thick]: 
# Specify Datastores to filter from report [local]: 
WARNING: Virtual machines with incorrect OS configuration
# VMs with incorrect OS Configuration, do not report on any VMs who are defined here [VM1_*|VM2_*]: 
WARNING: Virtual machines with less hard disks than partitions
# Do not report on any VMs who are defined here (regex) [VM1_*|VM2_*]: 
WARNING: Powered Off VMs
# VMs not to report on [Windows7*]: 
#VmPathName not to report on [-backup-]: 
# Report VMs powered off over this many days [7]: 
WARNING: Unwanted virtual hardware found
# Find unwanted virtual hardware [VirtualUSBController|VirtualParallelPort|VirtualSerialPort]: 
WARNING: Mis-named virtual machines
# Misnamed VMs, do not report on any VMs who are defined here [VM1_*|VM2_*]: 
WARNING: VM Network State
# Only show NICs that are set to Connect at Startup [$true]: 
WARNING: Reset VMs
# Set the number of days to show reset VMs [1]: 
WARNING: Snapshot activity
# Set the number of days to show Snapshots for [5]: 
# User exception for Snapshot removed [s-veeam]: 
WARNING: VMs with CPU or Memory Reservations Configured
# Do not report on any VMs who are defined here []: 
WARNING: VM Logging
# The number of logs to keep for each VM [10]: 
# The size logs can reach before rotating to a new log (bytes) [1000000]: 
WARNING: VM Tools Not Up to Date
# Do not report on any VMs who are defined here (regex) []: 
# Maximum number of VMs shown [30]: 
WARNING: NonPersistent Disks
# Exclude all virtual machines from report [^DV-|^MLB-]: 
WARNING: VMs Memory/CPU Hot Add configuration
# Should CPU hot plug be enabled [$true]: 
# Should Memory hot add be enabled [$true]: 
WARNING: VM - Display all VMs with CBT unexpected status
# Should CBT be enabled (true/false) [$false]: 
WARNING: Site Recovery Manager - RPO Violation Report
# SRM RPO Violations: Set the number of minutes an RPO has exceeded to report on [240]: 
# SRM RPO Violations: Only look for RPO events on VMs with these names: (regex) []: 
# SRM RPO Violations: Report on unresolved RPO violations only? [$true]: 
Specify Credential
Please specify server credential
User: administrator@vsphere.local
Password for user administrator@vsphere.local: ********
```

{{< /collapse >}}

After setting up the initial configuration we can start running the main script of the tool using the **vCheck.ps1 -Outputpath** command. The **“Outputpath”** option allows us to set where the report will be saved. When you run the command it will ask you for the vCenter login credentials. In my case I used the default administrator account but it is recommended to use an account with read-only privileges.

```powershell
PS /home/blabla/vCheck> ./vCheck.ps1 -Outputpath vcheck-reports/                                   

Specify Credential
Please specify server credential
User: administrator@vsphere.local #vCenter credentials
Password for user administrator@vsphere.local: ********
```

In this area I show you the example of the vCheck collection process.

```text
Begin Plugin Processing
[21:54:30] ..start calculating Connection settings for vCenter by Alan Renouf v1.20 [1 of 116]
[21:54:52] ..finished calculating Connection settings for vCenter by Alan Renouf v1.20 [1 of 116]
[21:54:52] ..start calculating General Information by Alan Renouf, Frederic Martin v1.3 [2 of 116]                                                           [21:55:04] ..finished calculating General Information by Alan Renouf, Frederic Martin v1.3 [2 of 116]                                                         [21:55:04] ..start calculating Checking VI Events by Alan Renouf v1.2 [3 of 116]
[21:55:09] ..start calculating ESXi with Technical Support mode or ESXi Shell enabled by Alan Renouf v1.3 [26 of 116]
[21:55:09] ..finished calculating ESXi with Technical Support mode or ESXi Shell enabled by Alan Renouf v1.3 [26 of 116]
[21:55:09] ..start calculating ESXi hosts which do not have Lockdown mode enabled by Alan Renouf v1.1 [27 of 116]
[21:55:09] ..finished calculating ESXi hosts which do not have Lockdown mode enabled by Alan Renouf v1.1 [27 of 116]
[21:55:09] ..start calculating Active Directory Authentication by Bill Wall, Dan Barr v1.2 [28 of 116]
[21:55:10] ..finished calculating Active Directory Authentication by Bill Wall, Dan Barr v1.2 [28 of 116]
[21:55:10] ..start calculating NTP Name and Service by Alan Renouf, Dan Barr v1.3 [29 of 116]
[21:55:10] ..finished calculating NTP Name and Service by Alan Renouf, Dan Barr v1.3 [29 of 116]
[21:55:10] ..start calculating Host Configuration Issues by Alan Renouf, Dan Barr v1.2 [30 of 116]
[21:55:10] ..finished calculating Host Configuration Issues by Alan Renouf, Dan Barr v1.2 [30 of 116]
[21:55:10] ..start calculating Host Alarms by Alan Renouf, John Sneddon v1.3 [31 of 116] [22:33:17] 
..Displaying HTML results
```

Once the command finishes, an **.html** file will be created with the result of the report. vCheck has the feature of being able to schedule the report to be sent by e-mail on a recurring basis. So, you can have a weekly report of how is the health of your virtual infrastructure.

Here are several examples of report generated with vCheck.

![Text](/img/vcheck_report.webp#center)

I hope you liked this tool. If you have any questions or comments about this post, leave them in comments. Regards.

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/F1F8DEV80)
