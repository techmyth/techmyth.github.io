---
title: "NetApp PowerShell Toolkit – Obtener información de Lun"
date: 2021-05-15T08:39:31-04:00
author: 'Jonathan Colon Feliciano'
draft: false
tags:
  - "NetApp"
  - "Lun"
  - "Powershell"
---

Recientemente en un [post](https://community.netapp.com/t5/Microsoft-Virtualization-Discussions/PSTK-Combine-the-output-of-Get-NcLunMap-and-Get-NcIgroup-custom-object/m-p/164648/highlight/true#M6354) del foro de NetApp un usuario solicito ayuda para crear una función en Powershell utilizando las librerías de DataOntap. Aquí les muestro como utilizamos estas librerías para conseguir unir múltiples objetos con información relacionada a los LUN asignados en NetApp.

Un dato curioso sobre esta petición es que de manera nativa las librerías de Ontap no te permiten filtrar la información requerida y que pueda ser desplegada en una sola Tabla. Para esto creamos un objeto dentro de PowerShell donde podemos construir el formato de la información y que este tenga un sentido mas lógico.

Las librerías de NetApp pueden ser instaladas desde el `PowerShell Gallery`:

[https://www.powershellgallery.com/packages/DataONTAP/9.8.0](https://www.powershellgallery.com/packages/DataONTAP/9.8.0)

#### [Código de la función get-luninfo]

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

#### Ejemplo de la información obtenida

![Text](/img/2021-03-06_19-30-1024x300.webp#center)

### Hasta Luego Amigos!
