---
layout: post
title: JEA - A parameter cannot be found that matches parameter name '...'
---

*A parameter cannot be found that matches parameter name '...'* error occurs when you launch a command on a JEA endpoint. But when you launch the same command directly on the host, it works ...

![_config.yml]({{ site.baseurl }}/images/but_why.jpg)

When a JEA endpoint is setted with the *SessionType = 'RestrictedRemoteServer'* parameter, the following functions are visibles:

* Clear-Host
* Exit-PSSession
* Get-Command
* Get-FormatData
* Get-Help
* Measure-Object
* Out-Default
* Select-Object

These 8 functions are not the 'real' functions. They are proxy functions intended to let you have a minimal set of command to interact with the endpoint while not being able to do something harmfull.

For example, the 'real' *Get-Command* function accepts the following syntaxes:

```powershell
Get-Command [[-ArgumentList] <Object[]>] [-Verb <string[]>] [-Noun <string[]>] [-Module <string[]>] [-FullyQualifiedModule <ModuleSpecification[]>] [-TotalCount <int>] [-Syntax] [-ShowCommandInfo] [-All] [-ListImported] [-ParameterName <string[]>] [-ParameterType <PSTypeName[]>] [<CommonParameters>]

Get-Command [[-Name] <string[]>] [[-ArgumentList] <Object[]>] [-Module <string[]>] [-FullyQualifiedModule <ModuleSpecification[]>] [-CommandType <CommandTypes>] [-TotalCount <int>] [-Syntax] [-ShowCommandInfo] [-All] [-ListImported] [-ParameterName <string[]>] [-ParameterType <PSTypeName[]>] [<CommonParameters>]
```

But the *Get-Command exposed in a JEA endpoint only has a few parameters:

```powershell
Get-Command [[-Name] <string[]>] [[-Module] <string[]>] [[-ArgumentList] <Object[]>] [[-CommandType] <CommandTypes>] [-ListImported] [-ShowCommandInfo] [<CommonParameters>]
```

# Command ran directly inside the session

If you try 'Get-Command -All' in a RestrictedRemoteServer JEA enpoint, you'll get an error: *A parameter cannot be found that matches parameter name 'all'*

In a RestrictedRemoteServer endpoint you are forced to use the proxy function and you cannot call the 'real' function directly.

This is different if the command is ran inside a module or a custom function.

# Command ran inside a module or function

If you launch a module function that use a restricted command with a non exposed parameter, you'll get the same error.

For example, if you try to import the Pester module inside a RestrictedRemoteServer JEA endpoint with the following Role Capability file:

```powershell
@{
GUID = 'c847874d-614a-4eca-afeb-a913b82d77a2'
ModulesToImport = 'Pester'
VisibleFunctions = 'Invoke-Pester'
}
```

You'll get the following error while trying to connect to the endpoint:

```
Enter-PSSession : One or more errors occurred processing the module 'Pester' specified in the InitialSessionState object used to create this runspace. See the ErrorRecords property for a complete list of errors. The first error was: A parameter cannot be found that matches parameter name 'All'.
```

That's because inside the Pester.psm1 file you have a call to 'Get-Command -Name Add-Member -Module Microsoft.PowerShell.Utility **-All**'.

Inside a module, or inside a custom function (defined in a Role Capablity for example), you can call the 'real' function by using its module prefix: *Microsoft.PowerShell.Core\Get-Command*

So, inside a module, the following command will work:

```powershell
Microsoft.PowerShell.Core\Get-Command -All
```

# Restricted functions syntax

In addition, this is the syntax of the 8 default functions in a RestrictedRemoteServer JEA (Windows PowerShell 5.1.14409 on W2012R2 server):

```powershell
Clear-Host

Exit-PSSession [<CommonParameters>]

Get-Command [[-Name] <string[]>] [[-Module] <string[]>] [[-ArgumentList] <Object[]>] [[-CommandType] <CommandTypes>] [-ListImported] [-ShowCommandInfo] [<CommonParameters>]

Get-FormatData [[-TypeName] <string[]>] [<CommonParameters>]

Get-Help [[-Name] <string>] [[-Category] <string[]>] [<CommonParameters>]

Measure-Object [-InputObject] <Object> [<CommonParameters>]

Out-Default [-InputObject] <Object> [<CommonParameters>]

Select-Object [[-Property] <string[]>] [-InputObject] <Object> [<CommonParameters>]
```

Note: the Get-Help function doesn't work in a RestrictedRemoteServer JEA endpoint :-/