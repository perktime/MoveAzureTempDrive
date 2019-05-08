# MoveAzureTempDrive
ARM template using PowerShell DSC to move the Azure temporary drive to a different drive letter

On occasion you may have a need to move the Azure temporary drive to a different drive letter. Azure by default is set to use the D drive. This drive letter configuration may conflict with existing scripts or company OS installation standards. I’ve created an ARM template that uses PowerShell DSC to allow you to move the drive letter. It performs the following steps:

1) Disables the Windows Page File and reboots the VM
2) Changes the drive letter from the D drive to a drive letter you specify in the ARM template parameters file
3) Re-enables the Windows page file and reboots the VM

To use it, modify the azuredeploy.parameters.json file with your vmName and your desired tempDriveLetter:

{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
  "parameters": {
    "vmName": {
      "value": "<put_your_existing_vm_name_here>"
    },
    "assetLocation": {
      "value": "https://petedscutil.blob.core.windows.net/scripts"
    },
    "tempDriveLetter": {
      "value": "Z"
    }
  }
}

Optionally, you can copy the MoveAzureTempDrive.ps1.zip DSC file to your own Azure storage account and modify the assetLocation parameter as well. Also, if you have an existing DSC extension you will have to remove it before deploying this. 

If you are interested in how this works, here is the explanation (note: assuming you understand how PowerShell DSC works):

To disable the Windows page file, we use “gwmi win32_pagefilesetting” which uses WMI to first check if the page file is enabled or not. If it is, we use this script to delete it and restart the VM:

gwmi win32_pagefilesetting
$pf=gwmi win32_pagefilesetting
$pf.Delete()
Restart-Computer –Force

Once the VM restarts, the PowerShell DSC module will then change the drive letter to your desired drive and then re-enable the page file and reboot:

Get-Partition -DriveLetter "D"| Set-Partition -NewDriveLetter $TempDriveLetter
$TempDriveLetter = $TempDriveLetter + ":"
$drive = Get-WmiObject -Class win32_volume -Filter “DriveLetter = '$TempDriveLetter'”
Set-WMIInstance -Class Win32_PageFileSetting -Arguments @{ Name = "$TempDriveLetter\pagefile.sys"; MaximumSize = 0; }

Restart-Computer -Force      




