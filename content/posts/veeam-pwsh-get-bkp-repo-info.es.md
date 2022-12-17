---
title: 'Veeam – Obtener información de los repositorios utilizando Powershell'
date: '2021-08-15T19:41:00-04:00'
tags:
    - VEEAM
---

Hola a todos,

Estos últimos días he estado realizando varios servicios de soportes relacionados a Veeam Backup & Replication donde he tenido la oportunidad de utilizar más los módulos de Powershell de Veeam. De manera que en esta oportunidad les estare mostrando de forma básica como obtener información relacionada a los repositorios conectados al servidor de Backup. Como siempre es necesario establecer conexión con el servidor de Backup utilizando el comando **“Connect-VBRServer”**.

```text
PS C:\Users\jocolon> Connect-VBRServer -Server veeam-vbr.pharmax.local -Credential (Get-Credential)

cmdlet Get-Credential at command pipeline position 1
Supply values for the following parameters:
User: pharmax\administrator
Password for user pharmax\administrator: ********

PS C:\Users\jocolon> 
```

Luego de conectarnos al servidor podemos utilizar el comando **“Get-VBRBackupRepository”** para identificar los repositorio conectados al servidor de Backup.

```text
PS C:\Users\jocolon> Get-VBRBackupRepository                                           

Name                      Type         Host            FriendlyPath    Description
----                      ----         ----            ------------    -----------
VEEAM-DD                  DDBoost      VEEAM-VBR.ph... ddboost://VE... Created by PHARMAX\administrator
ONTAP Snapshot            SanSnapsh... VEEAM-VBR.ph... ONTAP Storage   Primary storage snapshot only
VEEAM-HPE-StoreOnce-VSA   HPStoreOn... VEEAM-VBR.ph... storeonce://... Created by PHARMAX\administrator
Linux - Hardened Repos... LinuxLocal   veeam-lnx-px... /backup_data... Created by PHARMAX\jocolon

PS C:\Users\jocolon>
```

Como pueden ver el resultado del cmdlet suena un tanto incompleto ya que no muestra cierta información importante como el espacio utilizado. Par lograr obtener esta información les comparto un ejemplo básico que permite obtener información más relevante.

```powershell
$OutObj = @()
if ((Get-VBRServerSession).Server) {
    try {
        [Array]$BackupRepos = Get-VBRBackupRepository | Where-Object {$_.Type -ne "SanSnapshotOnly"}
        [Array]$ScaleOuts = Get-VBRBackupRepository -ScaleOut
        if ($ScaleOuts) {
            foreach ($ScaleOut in $ScaleOuts) {
                $Extents = Get-VBRRepositoryExtent -Repository $ScaleOut
                foreach ($Extent in $Extents) {
                    $BackupRepos = $BackupRepos + $Extent.repository
                }
            }
        }
        foreach ($BackupRepo in $BackupRepos) {
            $PercentFree = 0
            if (@($($BackupRepo.GetContainer().CachedTotalSpace.InGigabytes),$($BackupRepo.GetContainer().CachedFreeSpace.InGigabytes)) -ne 0) {
                $UsedSpace = ($($BackupRepo.GetContainer().CachedTotalSpace.InGigabytes-$($BackupRepo.GetContainer().CachedFreeSpace.InGigabytes)))
                if ($UsedSpace -ne 0) {
                    $PercentFree = ($UsedSpace/$($BackupRepo.GetContainer().CachedTotalSpace.InGigabytes)).tostring("P")
                }
            }
            $inObj = [ordered] @{
                'Name' = $BackupRepo.Name
                'Total Space' = "$($BackupRepo.GetContainer().CachedTotalSpace.InGigabytes)Gb"
                'Free Space' = "$($BackupRepo.GetContainer().CachedFreeSpace.InGigabytes)Gb"
                'Percent Used' = $PercentFree
                'Status' = Switch ($BackupRepo.IsUnavailable) {
                    'False' {'Available'}
                    'True' {'Unavailable'}
                    default {$BackupRepo.IsUnavailable}
                }
            }
            $OutObj += [pscustomobject]$inobj
        }
        $OutObj | Format-Table -Wrap -AutoSize
    }
    catch {
        Write-Output $_.Exception.Message
    }
}
```

En el siguiente ejemplo les muestro el resultado del código para obtener la información sobre los repositorios conectados a nuestro servidor de Backup de Veeam.

```text
Name                                  Total Space Free Space Percent Used Status     
----                                  ----------- ---------- ------------ ------
VEEAM-DD                              351Gb       292Gb      16.81%       Available
VEEAM-HPE-StoreOnce-VSA               0Gb         0Gb        0            Unavailable
Linux - Hardened Repository           199Gb       198Gb      0.50%        Available
E - Backup Repository                 499Gb       461Gb      7.62%        Available
F - Backup Repository                 99Gb        99Gb       0            Available
E - Backup Repository - VEEAM-VBR-02V 99Gb        5Gb        94.95%       Available
F - Backup Repository - VEEAM-VBR-02V 99Gb        98Gb       1.01%        Available
```

Espero que este post les haya gustado y les sea de utilizad en su jornada profesional.

## Hasta Luego Amigos!
