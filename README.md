# Disable Creation of Teams Sites

This repository provides a solution to restrict the creation of Teams sites for users except administrators. By disabling the ability to create Microsoft 365 groups, we indirectly disable the creation of Teams sites. This solution also provides a PowerShell script `disableTeamsCreation.ps1` to automate the process.

Note: Disabling the creation of Microsoft 365 groups will also affect the ability to create groups in other Office 365 services such as Outlook, SharePoint, and Yammer.

## Prerequisites

Before using the `disableTeamsCreation.ps1` script, ensure that you have completed the following prerequisites:

### AzureAD PowerShell module:

Install the AzureAD PowerShell module by running the following command in PowerShell:
```
Install-Module AzureAD
```


If you have the older AzureAD module installed, uninstall it first using the following command:
```
Uninstall-Module AzureAD
```


Then, install the AzureADPreview module:
```
Install-Module AzureADPreview
```

### Azure AD permissions:

Ensure that you have the necessary Azure AD permissions to run the script. You should have Global Administrator or User Administrator privileges. To run the script, you will be prompted to sign in with your Azure AD credentials using the `Connect-AzureAD` cmdlet.

## The disableTeamsCreation.ps1 Script

The provided script, disableTeamsCreation.ps1, connects to Azure AD and modifies the directory settings to disable group creation for all users except those specified in the designated security group. The script performs the following steps:

1. Connects to Azure AD.
2. Checks if the "Group.Unified" setting exists. If not, it creates one using the "group.unified" template.
3. Modifies the "EnableGroupCreation" setting to disable group creation for all users.
4. Sets the "GroupCreationAllowedGroupId" setting to the Object ID of the specified security group, allowing only its members to create groups.

To use the script, replace `$GroupName` with the name of the security group that you want to allow group creation. For example, 

```
$GroupName = "Group Creators"
```


## How to Restrict Group and Teams Site Creation

1. Install the AzureADPreview PowerShell module if you haven't already. To do this, run the following commands in your PowerShell session:
```
Uninstall-Module AzureAD
Install-Module AzureADPreview
```

2. Create a security group in the Microsoft 365 Admin Center. Add members who should be allowed to create Microsoft 365 groups and Teams sites. Note the name of this group, as you will need it later.
3. Download the disableTeamsCreation.ps1 script from this repository.
4. Edit the script and replace <GroupName> with the name of the security group you created in step 2.
5. Save the script and run it in a PowerShell session. Sign in with your administrator account when prompted.
6. Verify the settings have been applied by running the following commands in a PowerShell session:
```
$settingsObjectID = (Get-AzureADDirectorySetting | Where-object -Property Displayname -Value "Group.Unified" -EQ).id
(Get-AzureADDirectorySetting -Id $settingsObjectID).Values
```

This will display the updated settings, including the "GroupCreationAllowedGroupId" value, which should match the Object ID of the security group you specified.

## Conclusion

By using the provided `disableTeamsCreation.ps1` script and following the steps above, you can restrict the creation of Teams sites and Microsoft 365 groups to members of a designated security group. This allows for better control and governance over group creation in your organization.

## Script Breakdown

### Define variables:

```
$GroupName = "&lt;GroupName&gt;"
$AllowGroupCreation = $False
```

`$GroupName` stores the name of the security group that should be allowed to create Microsoft 365 groups and Teams sites.
`$AllowGroupCreation` is set to `$False` to disable group creation for all users.

### Connect to Azure AD:

```
Connect-AzureAD
```

The script connects to Azure Active Directory (Azure AD) using the Connect-AzureAD cmdlet.

### Check and create "Group.Unified" directory setting:

```
$settingsObjectID = (Get-AzureADDirectorySetting | Where-object -Property Displayname -Value "Group.Unified" -EQ).id
if(!$settingsObjectID)
{
    $template = Get-AzureADDirectorySettingTemplate | Where-object {$_.displayname -eq "group.unified"}
    $settingsCopy = $template.CreateDirectorySetting()
    New-AzureADDirectorySetting -DirectorySetting $settingsCopy
    $settingsObjectID = (Get-AzureADDirectorySetting | Where-object -Property Displayname -Value "Group.Unified" -EQ).id
}
```

The script checks if the "Group.Unified" directory setting exists. If it doesn't, the script creates a new "Group.Unified" directory setting using the "group.unified" template.

### Modify the "EnableGroupCreation" setting:

```
$settingsCopy = Get-AzureADDirectorySetting -Id $settingsObjectID
$settingsCopy["EnableGroupCreation"] = $AllowGroupCreation
```

The script modifies the "EnableGroupCreation" setting to disable group creation for all users, as specified by the `$AllowGroupCreation` variable.

### Set the "GroupCreationAllowedGroupId" setting:

```
if($GroupName)
{
    $settingsCopy["GroupCreationAllowedGroupId"] = (Get-AzureADGroup -SearchString $GroupName).objectid
} else {
    $settingsCopy["GroupCreationAllowedGroupId"] = $GroupName
}
Set-AzureADDirectorySetting -Id $settingsObjectID -DirectorySetting $settingsCopy
```

The script sets the "GroupCreationAllowedGroupId" setting to the Object ID of the specified security group (`$GroupName`). This allows only members of that security group to create Microsoft 365 groups and Teams sites.

### Display the updated settings:

```
(Get-AzureADDirectorySetting -Id $settingsObjectID).Values
```

Finally, the script displays the updated directory settings, including the "GroupCreationAllowedGroupId" value.
