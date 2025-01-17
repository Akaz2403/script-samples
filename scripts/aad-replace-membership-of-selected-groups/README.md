---
plugin: add-to-gallery
---

# Replace a user's membership in selected Microsoft 365 Groups or Teams

## Summary

This script can be used to replace the membership of a user for a selected list of Groups. It might be useful when a person changes role in an organization or is about to leave it.
 
[!INCLUDE [Delete Warning](../../docfx/includes/DELETE-WARN.md)]

# [CLI for Microsoft 365 with PowerShell](#tab/cli-m365-ps)
```powershell
$fileInput = "<PUTYOURPATHHERE.csv>"
$oldUser = "upnOfOldUser"
$newUser = "upnOfNewUser"
# Parameters end

$m365Status = m365 status

if ($m365Status -eq "Logged Out") {
  # Connection to Microsoft 365
  m365 login
}

# configure the CLI to output JSON on each execution
m365 cli config set --key output --value json
m365 cli config set --key errorOutput --value stdout
m365 cli config set --key showHelpOnFailure --value false
m365 cli config set --key printErrorsAsPlainText --value false

function Get-CLIValue {
  [cmdletbinding()]
  param(
    [parameter(Mandatory = $true, ValueFromPipeline = $true)]
    $input
  )
    $output = $input | ConvertFrom-Json
    if ($output.error -ne $null) {
      throw $output.error
    }
    return $output
 }

function Replace-Membership {
  [cmdletbinding()]
  param(
    [parameter(Mandatory = $true)]
    $fileInput ,
    [parameter(Mandatory = $true)]
    $oldUser,
    [parameter(Mandatory = $true)]
    $newUser
  )
  $groupsToProcess = Import-Csv $fileInput 
  $groupsToProcess.id | ForEach-Object {
    $groupId = $_
    Write-Host "Processing Group ($groupId)" -ForegroundColor DarkGray -NoNewline

    $group = $null
    try {
      $group = m365 aad o365group get --id $groupId | Get-CLIValue 
    }
    catch {
      Write-Host
      Write-Host $_.Exception.Message -ForegroundColor Red
      return
    }
    Write-Host " - $($group.displayName)" -ForegroundColor DarkGray

    $isTeam = $group.resourceProvisioningOptions.Contains("Team");

    $users = $null
    $users = m365 aad o365group user list --groupId $groupId | Get-CLIValue
    $users | Where-Object { $_.userPrincipalName -eq $oldUser } | ForEach-Object {
      $user = $_
      $isMember = $user.userType -eq "Member"
      $isOwner = $user.userType -eq "Owner"

      Write-Host "Found $oldUser with $($user.userType.tolower()) rights" -ForegroundColor Green

      # owners must be explicitly added as members if it is a team
      if ($isMember -or $isTeam) {
        try {
          Write-Host "Granting $newUser member rights"
          m365 aad o365group user add --groupId $groupId --userName $newUser | Get-CLIValue
        }
        catch {
          Write-Host $_.Exception.Message -ForegroundColor White
        }
      }

      if ($isOwner) {
        try {
          Write-Host "Granting $newUser owner rights"
          m365 aad o365group user add --groupId $groupId --userName $newUser --role Owner | Get-CLIValue
        }
        catch {
          Write-Host $_.Exception.Message -ForegroundColor White
        }
      }

      try {
        Write-Host "Removing $oldUser..."
        m365 aad o365group user remove --groupId $groupId --userName $oldUser --confirm $false | Get-CLIValue
      }
      catch {
        Write-Host $_.Exception.Message -ForegroundColor Red
        continue
      }
    }
  }
}

Replace-Membership $fileInput $oldUser $newUser
```
[!INCLUDE [More about CLI for Microsoft 365](../../docfx/includes/MORE-CLIM365.md)]
*** 

```csv
id
<groupId>
<groupId>
<groupId>
```
## Source Credit

Sample first appeared on [Replace a user's membership in selected Microsoft 365 Groups or Teams | CLI for Microsoft 365](https://pnp.github.io/cli-microsoft365/sample-scripts/aad/replace-membership-of-selected-groups/)

## Contributors

| Author(s) |
|-----------|
| Alan Eardley |
| Patrick Lamber |


[!INCLUDE [DISCLAIMER](../../docfx/includes/DISCLAIMER.md)]
<img src="https://telemetry.sharepointpnp.com/script-samples/scripts/aad-replace-membership-of-selected-groups" aria-hidden="true" />