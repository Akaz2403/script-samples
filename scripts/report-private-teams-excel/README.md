---
plugin: add-to-gallery
---

# Get User Details from a PowerApp

## Summary

This Script gets all the User Details of a Particular Powerapp from an Environment . The Output is exported as an Excel.

> For this sample, you will require the below commands to be Run in Powershell(As Administrator)

   Install-Module -Name Microsoft.PowerApps.Administration.PowerShell<br>
   Install-Module -Name Microsoft.PowerApps.PowerShell -AllowClobber

# [PnP PowerShell]

```powershell

Write-Host "Running Script..."

#Get Input - App ID 
$MethodCall=Read-Host "Enter App ID" 
$MethodCall=$MethodCall.ToUpper() 

$user=Get-PowerAppRoleAssignment -Appname $MethodCall
        $user.PrincipalEmail
        $user.RoleType
        $user | select PrincipalDisplayName,PrincipalEmail,PrincipalType,RoleType,AppName,EnvironmentName| Export-Csv -path \TenantPowerApps.csv

Write-Host "Successful"

```
