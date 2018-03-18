---
layout: post
title: DSC - How to reset LCM configuration
---

If you want your LCM to stop querying a PullServer or simply restore default values of the Local Configuration Manager, you have to apply a configuration based on default values.

![_config.yml]({{ site.baseurl }}/images/reset.jpg)

These are two version of the script as lCM is slightly different between PowerShell 4 and Powershell 5.

Default values come from the PowerShell command *Get-DscLocalConfigurationManager* on a clean machine.

Just save one of these scripts as *Reset-DscLocalConfigurationManager.ps1* and run as follow to reset a remote host's LCM:

```
.\Reset-DscLocalConfigurationManager.ps1 -ComputerName 'remote_computer_to_reset'
```

Or simply *.\Reset-DscLocalConfigurationManager.ps1* to reset localhost's LCM.

# Reset LCM configuration on PowerShell 5

The following script will reset all parameters of the LCM to their default values on PowerShell 5.0 and PowerShell 5.1.

```powershell
#Requires -Version 5.0

[CmdletBinding()]
Param(
    [Parameter()]
    [string[]]
    $ComputerName = @('localhost')
)

[DscLocalConfigurationManager()]
Configuration ResetLCM {
    Param (
        [String[]]
        $NodeName
    )
    Node $NodeName {
        Settings {
            ActionAfterReboot              = 'ContinueConfiguration'
            AllowModuleOverwrite           = $false
            CertificateID                  = $null
            ConfigurationDownloadManagers  = @{} 
            ConfigurationID                = $null
            ConfigurationMode              = 'ApplyAndMonitor'
            ConfigurationModeFrequencyMins = 15
            DebugMode                      = @('NONE')
            MaximumDownloadSizeMB          = 500
            RebootNodeIfNeeded             = $True
            RefreshFrequencyMins           = 30
            RefreshMode                    = 'PUSH'
            ReportManagers                 = @{}
            ResourceModuleManagers         = @{}
            SignatureValidations           = @{}
            StatusRetentionTimeInDays      = 10
        }
    }
}

ResetLCM -NodeName $ComputerName -OutputPath '.\ResetLCM'

Set-DscLocalConfigurationManager -Path '.\ResetLCM' -ComputerName $ComputerName
```

There are 2 more parameters in PowerShell V5:

* PartialConfigurations (doesn't accept $null value)
* SignatureValidationPolicy (doesn't accept 'NONE' value)

As these 2 parameters don't accept values returned by Get-DscLocalConfigurationManager, I did not included them in the script.

For now, I was not able to test what happen if these values aren't reset.

# Reset LCM configuration on PowerShell 4

The following script will reset all parameters of the LCM to their default values on PowerShell 5.0 and PowerShell 5.1.

```powershell
[CmdletBinding()]
Param(
    [Parameter()]
    [string[]]
    $ComputerName = @('localhost')
)

Configuration ResetLCM {
    Param (
        [String[]]
        $NodeName
    )
    Node $NodeName {
        LocalConfigurationManager {
            ActionAfterReboot              = 'ContinueConfiguration'
            AllowModuleOverwrite           = $false
            CertificateID                  = $null
            ConfigurationID                = $null
            ConfigurationMode              = 'ApplyAndMonitor'
            ConfigurationModeFrequencyMins = 15
            Credential                     = $null
            DebugMode                      = @('NONE')
            DownloadManagerCustomData      = $null
            DownloadManagerName            = $null
            RebootNodeIfNeeded             = $True
            RefreshFrequencyMins           = 30
            RefreshMode                    = 'PUSH'
        }
    }
}

ResetLCM -NodeName $ComputerName -OutputPath '.\ResetLCM'

Set-DscLocalConfigurationManager -Path '.\ResetLCM' -ComputerName $ComputerName
```
