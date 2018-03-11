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

To make other functions visible, you have to edit or create a Role Capability file to add *VisibleFunctions* or *VisibleCmdlets* statement(s). See: <https://docs.microsoft.com/en-us/powershell/jea/role-capabilities>

# The module providing a required cmdlet is not loaded

If no *VisibleFunctions* or *VisibleCmdlets* argument is provided in a Role Capability file, **all** modules located in an eligible module directory ($env:PSModulePath) are loaded.

If a *VisibleFunctions* or *VisibleCmdlets* argument is provided in a Role Capability file, only a few modules are loaded at session initialization time :

* Microsoft.PowerShell.Core
* Microsoft.PowerShell.Diagnostics
* Microsoft.PowerShell.Host
* Microsoft.PowerShell.Management
* Microsoft.PowerShell.Security
* Microsoft.PowerShell.Utility
* Microsoft.WSMan.Management

If the command you run needs a cmdlet provided by a module that is not loaded at session initialization, you have to explicitly import it thru a ModuleToImport statement in either your Role Capability file.

# You face a PowerShell JEA bug

When a *VisibleFunctions* or *VisibleCmdlets* argument is provided in a Role Capability file, functions of default loaded modules disappear.

So, you may encounter an error on New-Guid function even if Microsoft.PowerShell.Utility is loaded by default.

I filled an issue for that on GitHub: <https://github.com/PowerShell/JEA/issues/42>

This bug only affects functions from the modules that are loaded without being in the ModulesToImport list. So far, I only found 6 functions from the Microsoft.PowerShell.Utility; on Windows 2012R2 these functions are:

* ConvertFrom-SddlString
* Format-Hex
* Get-FileHash
* Import-PowerShellDataFile
* New-Guid
* New-TemporaryFile

There are 3 workarounds you can use.

## Don't use these functions

If you authored the PowerShell code calling these missing functions, you can rewrite your code to avoid using them.

New-Guid can easily be replaced by the expression *[guid]::newGuid()*

Other functions are more complex but you can find the code behind them in file C:\Windows\System32\WindowsPowerShell\v1.0\Modules\Microsoft.PowerShell.Utility\Microsoft.PowerShell.Utility.psm1

## Don't use VisibleFunctions or VisibleCmdlets

If you have not authored the code and don't want to change it. You may replace Visiblexxx parameter by a proxy function.

Let's say you only want to expose one simple command in your JEA endpoint: myCustomModule\Invoke-MyCustomFunction.

You should have a Role Capability file containing:

```powershell
ModulesToImport = 'myCustomModule'
VisibleFunctions = 'myCustomModule\Invoke-MyCustomFunction'
```

The following configuration will provide the same functionnality:

```powershell
ModulesToImport = 'myCustomModule'
FunctionDefinitions = @(
	@{Name = 'Invoke-MyCustomFunction'; ScriptBlock = { Param ($MyParam1, $MyParam2) myCustomModule\Invoke-MyCustomFunction -Param1 $MyParam1 -Param2 $MyParam2 }}
)
```

Of course, it only works with 'simple ParameterSet' functions and you'll have to update the Role Capability each time Param block of the exposed function eveolves.

## Duplicate missing functions in an other module

It may be the easier solution, but it's not very 'elegant' ...

As the missing functions are from the same module (Microsoft.PowerShell.Utility), the following method is easy to apply:

* Copy the file C:\Windows\System32\WindowsPowerShell\v1.0\Modules\Microsoft.PowerShell.Utility\Microsoft.PowerShell.Utility.psm1 to C:\Program Files\WindowsPowerShell\Modules\MPS.Utility\MPS.Utility.psm1
* Copy the file C:\Windows\System32\WindowsPowerShell\v1.0\Modules\Microsoft.PowerShell.Utility\Microsoft.PowerShell.Utility.psd1 to C:\Program Files\WindowsPowerShell\Modules\MPS.Utility\MPS.Utility.psd1
* Edit C:\Program Files\WindowsPowerShell\Modules\MPS.Utility\MPS.Utility.psd1 to remove cmdlets and replace the GUID

The file should looks like:

```powershell
@{
GUID="389eba42-94a4-44b8-afd9-176d0961063c"
FunctionsToExport= "Get-FileHash", "New-TemporaryFile", "New-Guid", "Format-Hex", "Import-PowerShellDataFile", "ConvertFrom-SddlString"
NestedModules="MPS.Utility.psm1"
}
```

Now import the module in your Role Capability configuration:

```powershell
ModulesToImport = 'myCustomModule', 'MPS.Utility'
VisibleFunctions = 'myCustomModule\Invoke-MyCustomFunction'
```

It works because when ModulesToImport is processed, functions from Microsoft.PowerShell.Utility have already disappeared and there is no duplicate declaration.

By the way there may be some side effect and you may miss updated functions at the next PowerShell update ...

Hope this bug will be adressed by MS team :-)