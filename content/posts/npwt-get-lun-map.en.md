---
title: "NetApp PowerShell Toolkit – Get Lun Mapping Information"
date: 2021-05-15T08:39:31-04:00
draft: false
tags:
  - "NetApp"
  - "Lun"
  - "Powershell"
---

Recently in a [post](https://community.netapp.com/t5/Microsoft-Virtualization-Discussions/PSTK-Combine-the-output-of-Get-NcLunMap-and-Get-NcIgroup-custom-object/m-p/164648/highlight/true#M6354) on the NetApp forum a user asked for help to create a function in Powershell using the DataOntap libraries. Here I show you how we use these libraries to join multiple objects with information related to the LUNs assigned in NetApp.

A curious fact about this request is that natively Ontap libraries do not allow you to filter the required information and that it can be displayed in a single table. For this we create an object within PowerShell where we can build the format of the information and that this has a more logical sense.

NetApp libraries can be installed from the PowerShell Gallery:

[https://www.powershellgallery.com/packages/DataONTAP/9.8.0](https://www.powershellgallery.com/packages/DataONTAP/9.8.0)

#### [Code of the get-luninfo function.]

```powershell
Import-Module dataontap
 
 
#Connect to Ontap Storage
 
Connect-NcController -Name <cluster> -Vserver <vserver>
 
 
#Get the list of LUNs
 
$luntable = Get-NcLun | Select-Object Path -ExpandProperty Path
 
 
#Declare Function
 
function get-luninfo {
 
 param(
 
     #Declare the required variable
 
     [string]$lunpath
 
 )
 
 #Check if the lun is mapped to any Host (IGROUP)
 
 if (get-nclunmap $lunpath) {
 
     #get the lun information
 
     $lunid = Get-NcLunmap $lunpath | Select-Object LunId -ExpandProperty LunId
 
     $lunigroup = get-nclunmap $lunpath | Select-Object InitiatorGroup -ExpandProperty InitiatorGroup
 
     $vserver =  Get-NcLun $lunpath | Select-Object Vserver -ExpandProperty Vserver
 
     $lunigrouptype = Get-NcIgroup -Name $lunigroup | Select-Object InitiatorGroupType -ExpandProperty InitiatorGroupType
 
     $lunigrouptypeOS = Get-NcIgroup -Name $lunigroup | Select-Object InitiatorGroupOsType -ExpandProperty InitiatorGroupOsType
 
     $lunigroupAluaEna = Get-NcIgroup -Name $lunigroup | Select-Object InitiatorGroupAluaEnabled -ExpandProperty InitiatorGroupAluaEnabled
 
     $initiators = Get-NcIgroup -Name $lunigroup | Select-Object Initiators -Unique -ExpandProperty Initiators
 
     $initiatorstatus = @()
 
     #Loop to find the initiators online status
 
     foreach ($in in $initiators.Initiators.InitiatorName) {
 
         $status = Confirm-NcLunInitiatorLoggedIn -VserverContext $vserver -Initiator $in | Select-Object Value -ExpandProperty Value
 
         $initiatorstatus += @(@{Initiator="$in";Online="$status"})
 
         }
 
     foreach ($object in $initiatorstatus) {
 
         $initiatoronline += $object.ForEach({[PSCustomObject]$_})
 
         }
 
     #Create a Object to better display and Glue the Information
 
     $obj = New-Object -TypeName PSObject
 
     $obj | add-member -MemberType NoteProperty -Name "vServer" -Value $vserver
 
     $obj | add-member -MemberType NoteProperty -Name "Lun ID" -Value $lunid
 
     $obj | add-member -MemberType NoteProperty -Name "IGROUP Name" -Value $lunigroup
 
     $obj | add-member -MemberType NoteProperty -Name "IGROUP TYPE" -Value $lunigrouptype
 
     $obj | add-member -MemberType NoteProperty -Name "IGROUP TYPE OS" -Value $lunigrouptypeOS
 
     $obj | add-member -MemberType NoteProperty -Name "IGROUP ALUA ENABLE" -Value $lunigroupAluaEna
 
     $obj | add-member -MemberType NoteProperty -Name "Lun Path" -Value $lunpath
 
     #$obj | add-member -MemberType NoteProperty -Name "Initiator Info" -Value $initiatoronline
 
     
 
     #Return the Formated Information
 
     Write-Output $obj | FT
 
     Write-Output $initiatoronline
 
 }
 
 # If the LUN isnt mapped to any HOST, display the available information.
 
 else {
 
     $vserver =  Get-NcLun $lunpath | Select-Object Vserver -ExpandProperty Vserver
 
     $obj = New-Object -TypeName PSObject
 
     $obj | add-member -MemberType NoteProperty -Name "vServer" -Value $vserver
 
     $obj | add-member -MemberType NoteProperty -Name "Lun Path" -Value $lunpath
 
     $obj | add-member -MemberType NoteProperty -Name "Lun Mapping" -Value "Lun Not Mapped"
 
     Write-Output $obj | FT -Wrap -AutoSize
 
 }
 
}
 
#Calling the Function
 
foreach ($lun in $luntable) {
 
 get-luninfo($lun)
 
 }
 ```

#### Example of the retrieved information

![Text](/img/2021-03-06_19-30-1024x300.webp#center)

### Hasta Luego Amigos!
