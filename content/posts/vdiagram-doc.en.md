---
title: 'HomeLab: vDiagram Draw your Virtual Infrastructure'
date: '2021-08-27T07:11:00-04:00'
tags:
    - VMware
---

Hello everyone

Taking as a reference the [Top 10 VMware Admin Tools](https://core.vmware.com/blog/top-10-vmware-admin-tools) list, this time I am going to show you how to use the [vDiagram](https://github.com/Tony-SouthFLVMUG/vDiagram2.0) tool that has the #6 position in the list of the most used tools by VMware infrastructure administrators. In essence this PowerShell script captures and draws a `VMware vSphere` infrastructure using Microsoft Visio. Originally this tool was created by `Alan Renouf` [@alanrenouf](https://twitter.com/alanrenouf) and currently the project is maintained by `Tony Gonzalez` [@vDiagram_Tony](https://twitter.com/vDiagram_Tony).

![Text](/img/invisible-infrastructure-300x200.webp#center)

To use this tool, the following requirements must be met:

1. PowerShell >= 5.1
2. PowerCLI module (`Install-Module -Name VMware.PowerCLI`)
3. Microsoft Visio 2013+

Once all the requirements are fulfilled, proceed to download the PowerShell code. To download the application, click on the following link:

<https://github.com/Tony-SouthFLVMUG/vDiagram2.0>

![Text](/img/2021-08-24_09-06.webp#center)

Once the package is downloaded proceed to unpack the contents.

```text
PS C:\> Expand-Archive -LiteralPath .\vDiagram2.0-master.zip -DestinationPath .
PS C:\> ls vD*


    Directory: C:\Users\jocolon\Downloads


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         8/26/2021  11:15 AM                vDiagram2.0-master
-a----         7/21/2021   9:37 AM       12470234 vDiagram2.0-master.zip


PS C:\>
```

Move to the unzipped folder and validate the content with the `ls` or `dir` command.

```text
PS C:\> cd .\vDiagram2.0-master\
PS C:\vDiagram2.0-master> ls


    Directory: C:\vDiagram2.0-master


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         2/15/2021   5:29 PM                archived
-a----         2/15/2021   5:29 PM             66 .gitattributes
-a----         2/15/2021   5:29 PM           5771 README.md
-a----         2/15/2021   5:29 PM         109288 vDiagram.ico
-a----         2/15/2021   5:29 PM         673926 vDiagram_2.0.11.ps1
-a----         2/15/2021   5:29 PM         985128 vDiagram_2.0.11.vssx
-a----         2/15/2021   5:29 PM         116398 vDiagram_Scheduled_Task_2.0.11.ps1
-a----         2/15/2021   5:29 PM         254037 vDiagram_Standard.png


PS C:\vDiagram2.0-master>
```

Use the `Unblock-File` command that allows us to execute files that have been downloaded from the Internet.

```text
PS C:\vDiagram2.0-master> Unblock-File .\vDiagram_2.0.11.ps1
PS C:\vDiagram2.0-master>
```

In this step with the `$PSVersionTable` command confirm the PowerShell version locally installed. Reviewing the requirements section above you can see that in order to use the `vDiagram` tool you need to have a PowerShell version 5.1.x or higher. In the example below you can see that my computer has version `5.1.19041.1151`.

```text
PS C:\vDiagram2.0-master> $PSVersionTable

Name                           Value
----                           -----
PSVersion                      5.1.19041.1151
PSEdition                      Desktop
PSCompatibleVersions           {1.0, 2.0, 3.0, 4.0...}
BuildVersion                   10.0.19041.1151
CLRVersion                     4.0.30319.42000
WSManStackVersion              3.0
PSRemotingProtocolVersion      2.3
SerializationVersion           1.1.0.1


PS C:\vDiagram2.0-master>
```

Additionally, you must validate the `PowerCLI` module. With the `Get-Module` command you can validate the PowerCLI installed version.

```text
PS C:\vDiagram2.0-master> Get-Module -ListAvailable -Name 'VMware.PowerCLI' | Sort-Object -Property Version -Descending | Select-Object -First 1


    Directory: C:\Program Files\WindowsPowerShell\Modules


ModuleType Version    Name                                ExportedCommands
---------- -------    ----                                ----------------
Manifest   12.3.0.... VMware.PowerCLI


PS C:\vDiagram2.0-master> 
```

After all requirements are met you can run the script by calling file `.vDiagram_2.0.11.ps1`.

```text
PS C:\vDiagram2.0-master> 

PS C:\vDiagram2.0-master> .\vDiagram_2.0.11.ps1
[08/24/2021 10:45:16] VMware PowerCLI Module(s) 12.3.0.17860403 11.5.0.14912921  found on this machine.


```

Once the program finishes running you can see in the `Prerequisites` tab a summary of all the dependencies and their status. In the below example all dependencies are shown in green color, which indicates that they are installed.

![Text](/img/2021-08-24_10-50.webp#center)

Following the steps as shown in the `Directions` tab, it is necessary to enter the `IP/FQDN` address of the `vCenter` and the credentials. It is important to mention that a read only account is the only needed privileges needed to connect and extract the required information.

![Text](/img/2021-08-24_10-51.webp#center)

Once the vCenter information is filled in, proceed to validate the connection by pressing the `Connect to vCenter` button. As you can see in the below image the button changes to green indicating that there was a successful connection to vCenter with the provided credentials.

![Text](/img/2021-08-24_10-51_1.webp#center)

The next step would be to select the `Capture CSVs for Visio` tab and specify the folder where the reports will be saved. In the below example I used the `<Desktop/Output>` folder.

![Text](/img/2021-08-24_10-54-1.webp#center)

It is important to mention that for each selected value a file will be created with the information of the respective element.

![Text](/img/2021-08-24_10-55_1.webp#center)

Then proceed to Click on `Collect CSV Data` to start the data collection process.

![Text](/img/2021-08-24_10-56-1.webp#center)

Once the data collection process ends, select the `Draw Visio` tab and configure the `Select CSV Input Folder` option.

![Text](/img/2021-08-24_16-08-2.webp#center)

Next, the folder where the previously collected data was stored is selected. In the following example it would be the `<Desktop/Output>` folder that I used in the `Capture CSVs for Visio` section.

![Text](/img/2021-08-24_16-09-1.webp#center)

It is now important to verify that all the information needed to build the diagram has been provided by clicking on the `CSV Validation Complete` button.

![Text](/img/2021-08-24_16-10-2.webp#center)

In the following step it is required to specify a folder where the diagram will be saved once it has been generated. To do so, click on `Select Visio output folder` and then select the folder to be used for this purpose. In the following example I have selected `<Desktop/Output>`.

![Text](/img/2021-08-24_16-12.webp#center)

In the `Visio Output Folder` area select the multiple options available to generate the diagram. Once you have selected the `Output` folder you can generate the diagram by clicking on the `Draw Visio` button.

![Text](/img/2021-08-24_16-14-1.webp#center)

At this step click `OK` in the notification about the diagram creation.

![Text](/img/2021-08-24_16-26-1.webp#center)

To view the generated diagram of our virtual infrastructure, click on the `Open Visio Diagram` button.

![Text](/img/2021-08-24_16-26_1-1.webp#center)

Finally, here are some sample images of the diagrams generated using my `HomeLab` virtual infrastructure as an example.

![Text](/img/VM-To-Host-1-1024x560.webp#center)

### Summary

In this lab a tool called `vDiagram` is demonstrated, which allows us to make a logical representation of how the components of our virtual infrastructure are related. The good thing about this tool is that it is available for free. I hope you liked this lab. If you have any doubts or questions about this lab, leave them in the comments. Hasta Luego!!!!
