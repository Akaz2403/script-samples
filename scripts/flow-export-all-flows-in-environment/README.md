---
plugin: add-to-gallery
---

# Export all flows in environment

## Summary

When was the last time you backed up all the flows in your environment?

By combining the CLI for Microsoft 365 and PowerShell along with a new pure PnP PowerShell example we can make this task easy and repeatable.

This script will get all flows in your default environment and export them as both a ZIP file for importing back into Power Automate and as a JSON file for importing into Azure as an Azure Logic App.

# [CLI for Microsoft 365 with PowerShell](#tab/cli-m365-ps)
```powershell

Write-Output "Getting environment info..."
$environment = m365 flow environment list --query '[?contains(displayName,`default`)] .name'

Write-Output "Getting Flows info..."
$flows = m365 flow list --environment $environment --asAdmin --output json | ConvertFrom-JSON

Write-Output "Found $($flows.Count) Flows to export..."

$flows | ForEach-Object {
    Write-Output "Exporting as ZIP & JSON... $($_.displayName)"
    $filename = $_.displayName.Replace(" ","")
    $timestamp = Get-Date -Format "yyyymmddhhmmss"
    $exportPath = "$($filename)_$($timestamp)"
    $flowId = $_.Name
    
    m365 flow export --id $flowId --environment $environment --packageDisplayName $_.displayName --path "$exportPath.zip"
    m365 flow export --id $flowId --environment $environment --format json --path "$exportPath.json"
}

Write-Output "Complete"

```
[!INCLUDE [More about CLI for Microsoft 365](../../docfx/includes/MORE-CLIM365.md)]

# [PnP PowerShell](#tab/pnpps)
```powershell
$environmentName = "Personal Productivity"

$FlowEnv = Get-PnPFlowEnvironment | Where-Object { $_.Properties.DisplayName -eq $environmentName }

Write-Host "Getting All Flows in $environmentName Environment"
$flows = Get-PnPFlow -Environment $FlowEnv -AsAdmin #Remove -AsAdmin Parameter to only target Flows you have permission to access

Write-Host "Found $($flows.Count) Flows to export..."

foreach ($flow in $flows) {

    Write-Host "Exporting as ZIP & JSON... $($flow.Properties.DisplayName)"
    $filename = $flow.Properties.DisplayName.Replace(" ", "")
    $timestamp = Get-Date -Format "yyyymmddhhmmss"
    $exportPath = "$($filename)_$($timestamp)"
    $exportPath = $exportPath.Split([IO.Path]::GetInvalidFileNameChars()) -join '_'
    Export-PnPFlow -Environment $FlowEnv -Identity $flow.Name -PackageDisplayName $flow.Properties.DisplayName -AsZipPackage -OutPath "$exportPath.zip" -Force
    Export-PnPFlow -Environment $FlowEnv -Identity $flow.Name | Out-File "$exportPath.json"

}

```
[!INCLUDE [More about PnP PowerShell](../../docfx/includes/MORE-PNPPS.md)]
***

## Source Credit

Sample first appeared on [Export all flows in environment | CLI for Microsoft 365](https://pnp.github.io/cli-microsoft365/sample-scripts/flow/export-all-flows-in-environment/)

Added PnP PowerShell version from [PnP.PowerShell + Bonus Script: Export all Flows using PnP.PowerShell](https://www.leonarmston.com/2021/01/testing-out-the-new-power-automate-flow-commands-in-pnp-powershell-bonus-script-export-all-flows-using-pnp-powershell/)
## Contributors

| Author(s) |
|-----------|
| Garry Trinder |
| Luise Freese |
| [Leon Armston](https://github.com/LeonArmston) |

[!INCLUDE [DISCLAIMER](../../docfx/includes/DISCLAIMER.md)]
<img src="https://telemetry.sharepointpnp.com/script-samples/scripts/flow-export-all-flows-in-environment" aria-hidden="true" />
