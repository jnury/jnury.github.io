---
layout: post
title: JEA - The term '...' is not recognized as the name of a cmdlet
---

*The term '...' is not recognized as the name of a cmdlet* error occurs when you launch a command on a JEA endpoint. But when you launch the same command directly on the host, it works ...

![_config.yml]({{ site.baseurl }}/images/disappointment_everywhere.jpg)

There may be 3 different causes. Fear the third one ;-)

Note: this article is based on the following PS Session Configuration parameters:

* SessionType = 'RestrictedRemoteServer'
* RunAsVirtualAccount = $true

# The cmdlet or function your try to launch is not visible

By default, JEA endpoints only expose 8 functions:

```
[localhost]: PS>Get-Command

CommandType     Name
-----------     ----
Function        Clear-Host
Function        Exit-PSSession
Function        Get-Command
Function        Get-FormatData
Function        Get-Help
Function        Measure-Object
Function        Out-Default
Function        Select-Object
```

To make other functions visible, you have to edit your PSSession Configuration file or your Role Capability file. See: 

# The module providing a required cmdlet is not loaded

If no *VisibleFunctions* or *VisibleCmdlets* argument is provided in either PSSession Configuration or a Role Capability file, **all** modules located in an eligible module directory ($env:PSModulePath) are loaded.

If a *VisibleFunctions* or *VisibleCmdlets* argument is provided in either PSSession Configuration or a Role Capability file, only a few modules are loaded at session initialization time :

* Microsoft.PowerShell.Core
* Microsoft.PowerShell.Diagnostics
* Microsoft.PowerShell.Host
* Microsoft.PowerShell.Management
* Microsoft.PowerShell.Security
* Microsoft.PowerShell.Utility
* Microsoft.WSMan.Management

If the command you run need a cmdlet provided by a module that is not loaded at session initialization, you have to explicitly import it thru a ModuleToImport statement in either your PSSession Configuration or a Role Capability file.

# You face a PowerShell JEA bug

When a *VisibleFunctions* or *VisibleCmdlets* argument is provided in either PSSession Configuration or a Role Capability file, functions of default loaded modules disappear.

I filled an issue for that on GitHub: <https://github.com/PowerShell/JEA/issues/42>
