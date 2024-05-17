---
title: 'HomeLab: Automated VMware Infrastructure Documentation'
date: '2021-06-06T16:17:25-04:00'
author: 'Jonathan Colon Feliciano'
tags:
    - VMware
---

In this blog I will be talking about how to automate the creation of documentation reports of our virtual infrastructure. There are several commercial solutions to generate this type of report but I will be talking about [`As Built Report`](https://www.asbuiltreport.com/) a free tool that uses powershell as a base to build the reports.

The `As Built Report` tool uses the VMware.PowerCLI modules that we explained previously in our blog. If you want to know more about PowerCLI follow this link [`here`](http://192.168.7.40/2021/06/05/how-to-install-and-use-powercli-on-archlinux/). An important fact about `As Built Report` is that it is not only used to generate reports on VMware but also supports the following products:

- VMware vSphere, NSX & SRM
- Cisco UCS Manager
- Nutanix Prism Element
- Pure Storage FlashArray
- Rubrik
- Zerto
- Dell/EMC VxRail
- Cohesity DataPlatform
- etc…

First of all to use this tool we need to validate the requirements that in general consist of the following:

- Windows PowerShell 5.1 o later
- VMware.PowerCLI

To install the `As Built Report` powershell module use the command `Install-Module` followed by the module name `AsBuiltReport`.

```powershell
PS /home/blabla> Install-Module -Name AsBuiltReport

Untrusted repository
You are installing the modules from an untrusted repository. If you trust this repository, change its InstallationPolicy value by running the Set-PSRepository cmdlet. Are you sure you want to install the modules from 'PSGallery'?
[Y] Yes  [A] Yes to All  [N] No  [L] No to All  [S] Suspend  [?] Help (default is "N"): A
PS /home/blabla>   
```

An optional requirement is to build a configuration file that allows you to set the organization parameters that are used to generate the report. This process generates a JSON file which is used as a template so that you do not have to fill in repetitive information when generating reports.

#### AsBuiltReport JSON Configuration File

The powershell cmdlet `New-AsBuiltConfig` allows you to generate a template that will be used as the basis of the report. This template sets the non-technical parameters of the report.

```powershell
PS C:\WINDOWS\system32> New-AsBuiltConfig

---------------------------------------------
 <        As Built Report Information      >
---------------------------------------------
Enter the name of the Author for this As Built Report [jocolon]: Jonathan Colon
```

```powershell
---------------------------------------------
 <           Company Information           >
---------------------------------------------
Would you like to enter Company information for the As Built Report? (y/n): y
Enter the Full Company Name: Zen PR Solutions
Enter the Company Short Name: ZENPR
Enter the Company Contact: Jonathan Colon
Enter the Company Email Address: jcolonf@zenprsolutions.com
Enter the Company Phone: XXX-XXX-XXXX
Enter the Company Address: Puerto Rico
```

```powershell
---------------------------------------------
 <            Email Configuration          >
---------------------------------------------
Would you like to enter SMTP configuration? (y/n): n
```

```powershell
----------------------------------------------
 <       As Built Report Configuration      >
----------------------------------------------
Would you like to save the As Built Report configuration file? (y/n): y
Enter a name for the As Built Report configuration file [AsBuiltReport]: HomeLab VMware Report
Enter the path to save the As Built Report configuration file [C:\Users\jocolon\AsBuiltReport]:

Name                           Value
----                           -----
Email                          {Port, Credentials, Server, To...}
Company                        {FullName, Contact, Phone, Email...}
UserFolder                     {Path}
Report                         {Author}


PS C:\WINDOWS\system32>

```

Once the process is completed, a JSON file will be created with the following content:

```json
{
    "Email":  {
                  "Port":  null,
                  "Credentials":  null,
                  "Server":  null,
                  "To":  null,
                  "From":  null,
                  "UseSSL":  null,
                  "Body":  null
              },
    "Company":  {
                    "FullName":  "Zen PR Solutions",
                    "Contact":  "Jonathan Colon",
                    "Phone":  "787-203-2790",
                    "Email":  "jcolonf@zenprsolutions.com",
                    "ShortName":  "ZENPR",
                    "Address":  "Puerto Rico"
                },
    "UserFolder":  {
                       "Path":  "C:\\Users\\jocolon\\AsBuiltReport"
                   },
    "Report":  {
                   "Author":  "Jonathan Colon"
               }
}
```

The `New-AsBuiltReportConfig` command allows you to set the technical parameters of the report such as the verbose level and type of information collected.

```batch
PS C:\WINDOWS\system32> New-AsBuiltReportConfig VMware.vSphere -FolderPath C:\Users\jocolon\AsBuiltReport\ -Filename ReportConfig

```

Once the process is completed, a JSON file will be created with the following content:

```json
{
    "Report": {
        "Name": "VMware vSphere As Built Report",
        "Version": "1.0",
        "Status": "Released"
    },
    "Options": {
        "ShowLicenseKeys": false,
        "ShowVMSnapshots": true
    },
    "InfoLevel": {
        "_comment_": "0 = Disabled, 1 = Summary, 2 = Informative, 3 = Detailed, 4 = Adv Detailed, 5 = Comprehensive",
        "vCenter": 3,
        "Cluster": 3,
        "ResourcePool": 3,
        "VMHost": 3,
        "Network": 3,
        "vSAN": 3,
        "Datastore": 3,
        "DSCluster": 3,
        "VM": 2,
        "VUM": 3
    },
    "HealthCheck": {
        "vCenter": {
            "Mail": true,
            "Licensing": true
        },
        "Cluster": {
            "HAEnabled": true,
            "HAAdmissionControl": true,
            "HostFailureResponse": true,
            "HostMonitoring": true,
            "DatastoreOnPDL": true,
            "DatastoreOnAPD": true,
            "APDTimeOut": true,
            "vmMonitoring": true,
            "DRSEnabled": true,
            "DRSAutomationLevelFullyAuto": true,
            "PredictiveDRS": false,
            "DRSVMHostRules": true,
            "DRSRules": true,
            "vSANEnabled": false,
            "EVCEnabled": true,
            "VUMCompliance": true
        },
        "VMHost": {
            "ConnectionState": true,
            "HyperThreading": true,
            "ScratchLocation": true,
            "IPv6": true,
            "UpTimeDays": true,
            "Licensing": true,
            "SSH": true,
            "ESXiShell": true,
            "NTP": true,
            "StorageAdapter": true,
            "NetworkAdapter": true,
            "LockdownMode": true,
            "VUMCompliance": true
        },
        "vSAN": {},
        "Datastore": {
            "CapacityUtilization": true
        },
        "DSCluster": {
            "CapacityUtilization": true,
            "SDRSAutomationLevelFullyAuto": true
        },
        "VM": {
            "PowerState": true,
            "ConnectionState": true,
            "CpuHotAdd": true,
            "CpuHotRemove": true,
            "MemoryHotAdd": true,
            "ChangeBlockTracking": true,
            "SpbmPolicyCompliance": true,
            "VMToolsStatus": true,
            "VMSnapshots": true
        }
    }
}
```

Finally, we generate the report using the `New-AsBuiltReport` command with the vCenter information parameters and referencing the JSON file we have created as templates.

```powershell
PS C:\WINDOWS\system32> New-AsBuiltReport -Report VMware.vSphere -Target vcenter-01v.zenprsolutions.local -Username administrator@vsphere.local -Password XXXXX -Format Word,Text,HTML -OutputFolderPath 'C:\Users\jocolon\OneDrive\Desktop\' -EnableHealthCheck -AsBuiltConfigFilePath 'HomeLab VMware Report.json' -ReportConfigFilePath 'ReportConfig.json'

VMware vSphere As Built Report 'VMware vSphere As Built Report' has been saved to 'C:\Users\jocolon\OneDrive\Desktop\'.
PS C:\WINDOWS\system32>
```

Once the process of collecting the information from the vCenter is finished, the command saves the report as specified with the `OutputFolderPath` parameter. The following image shows the generated report in the `Word,Text,HTML` format.

![Text](/img/2021-06-06_13-57.webp#center)

Below I show you some images showing the result of the report collected from the vCenter `vcenter-01v`:

![Text](/img/asbuiltreport-vsphere.webp#center)

### Summary

In this lab we learned how easy it is to create documentation about our virtual infrastructure by using freely available tools. `As Built Report` is a robust tool that facilitates the manual process of creating or updating our documentation.
