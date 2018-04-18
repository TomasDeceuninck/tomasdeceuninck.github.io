---
layout: post
title:  "Structure of a PowerShell Module"
subtitle: "and it's project"
date:   2018-04-17
tags: PowerShell
background: '/img/posts/default.jpg'
comments: true
---

Since I started developing with PowerShell I've been reading and testing several module structures.

In this article I would like to summarize the parts that have really become fixed parts of my modules.
I also created a [Plaster](https://github.com/PowerShell/Plaster){:target="_blank"} template that scaffolds the basics outlined in this article.

[My Plaster Template](https://github.com/Invoke-Automation/IAPlasterProjectTemplate)

## Project structure

I want to discuss the entire project folder structure because serval PowerShell-Project essentials are not placed in the Module that is distributed.

The project consists of several building blocks, some of which could be considered optional:

[Top]: #project-structure

- [The Actual PowerShell Module](#module-structure)
- [Pester Tests](#tests)
- [Documentation](#docs)
- [Local Builds](#builds)
- [VSCode Settings](#vscode)
- [Extra Meta Infomation](#meta)

This is an example project structure for a module 'TestModule':

![See ProjectStructure.png]({{ site.url }}/img/posts/ProjectStructure.png)

## Module Structure

The Module folder structure looks like this:

```asci
ProjectFolder/
├── ...
├── ModuleName/
|   ├── en-US/
|   │   └── ModuleName-help.xml
|   ├── Private/
|   │   ├── _Classes.ps1
|   │   ├── Get-PrivateFunction.ps1
|   │   └── ...
|   ├── Public/
|   │   ├── New-PublicFunction.ps1
|   │   └── ...
|   ├── Settings.xml
|   ├── ModuleName.psd1
|   └── ModuleName.psm1
└── ...
```

### Public / Private

Every cmdlet has its own file which has the same name as the function/cmdlet that is defined in it.
Each cmdlet-file is then placed in either the Private or Public folder.

The Module file will then read through these folders, dot-source each cmdlet-file, and export those functions/cmdlets for which the file was placed in the Public folder.

```PowerShell
# Get public and private function definition files.
# Sort to make sure files that start with '_' get loaded first
$Private = @(Get-ChildItem -Path $PSScriptRoot\Private -Recurse -Filter "*.ps1") | Sort-Object Name
$Public = @(Get-ChildItem -Path $PSScriptRoot\Public -Recurse -Filter "*.ps1") | Sort-Object Name

# Dots source the private files
foreach ($import in $Private) {
    try {
        . $import.fullName
        Write-Verbose -Message ("Imported private function {0}" -f $import.fullName)
    } catch {
        Write-Error -Message ("Failed to import private function {0}: {1}" -f $import.fullName, $_)
    }
}
# Dots source the public files
foreach ($import in $Public) {
    try {
        . $import.fullName
        Write-Verbose -Message ("Imported public function {0}" -f $import.fullName)
    } catch {
        Write-Error -Message ("Failed to import public function {0}: {1}" -f $import.fullName, $_)
    }
}

Export-ModuleMember -Function $Public.BaseName
```

This approach makes it very easy to see all available cmdlet and making a cmdlet Public or Private is just a matter of moving the file.

### Settings

Because sometimes you have a few parameters you want to use throughout your module I setup a Settings.xml which is also processed by the Module file.

```PowerShell
# Load Module settings file
try {
    $script:SETTINGS = ([xml](Get-Content "$PSScriptRoot\Settings.xml")).Settings
} catch {
    throw 'Could not load settings.xml file.'
}
```

The upside on this approach is that there is no need to change anything in the module file and all parameters are centralized.

> **Note**: It might be better to start using a json-file for this since that is a bit more readable, but for now xml still works for me. Feel free to play around with this and give me your feedback!

### en-US

The `ModuleName-help.xml` under the `en-US` folder is generated with PlatyPS to supply all your [documentation](#docs) to your users.

[Top]

## Tests

The Test folder and files looks like this:

```asci
ProjectFolder/
├── ...
├── Tests/
│   ├── _InitializeTests.ps1
│   ├── Get-PrivateFunction.Tests.ps1
│   ├── Get-PublicFunction.Tests.ps1
│   ├── Get-ModuleName.Tests.ps1
│   └── ...
├── PSScriptAnalyzerSettings.psd1
└── ...
```

Every cmdlet has its own `NameOfCmdlet.Tests.ps1` file that holds all the unit tests for that cmdlet.

### _initializeTests

At the start of each `.Tests.ps1` file the `_InitializeTests.ps1` file is dot-sourced.

```PowerShell
. $PSScriptRoot\_InitializeTests.ps1
```

This way you can define any required Modules and/or parameters in this script.

### PSSCriptAnalyzer

One part of the general module tests that can be implemented uses [PSScriptAnalyzer](https://github.com/PowerShell/PSScriptAnalyzer){:target="_blank"} to evaluate the quality of your code.
Since you sometimes want to specify some settings a `PSScriptAnalyzerSettings.psd1` can be included.

[Top]

## Docs

The Documentation folder and files looks like this:

```asci
ProjectFolder/
├── ...
├── docs/
│   ├── Get-PrivateFunction.md
│   ├── Get-PublicFunction.md
│   └── ...
├── ModuleName/
│   ├── en-US/
│   |   └── ModuleName-help.xml
│   └── ...
└── ...
```

These files are initially generated and afterwards updated by [PlatyPS](https://github.com/PowerShell/platyPS){:target="_blank"}.
The content of these files is based on any [comment based help](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_comment_based_help?view=powershell-6){:target="_blank"} and the analysis PlatyPS makes of your exported cmdlets.

The external help (`ModuleName-help.xml`) is generated/updated using the `New-ExternalHelp` PlatyPS-cmdlet which uses the already generated/updated md-files under the `docs` folder to create the xml-file that ships with your module.

[Top]

## Build

The Build folder and files looks like this:

```asci
ProjectFolder/
├── ...
├── build/
│   ├── ModuleName/
│   |   ├── 0.0.2/
│   |   |   ├── en-US/
│   |   |   |   └── ModuleName-help.xml
│   |   |   ├── Private/
│   |   |   |   ├── Get-PrivateFunction.ps1
│   |   |   |   └── ...
│   |   |   ├── Public/
│   |   |   |   ├── Get-PublicFunction.ps1
│   |   |   |   └── ...
│   |   |   ├── ModuleName.psd1
│   |   |   ├── ModuleName.psm1
│   |   |   └── Settings.xml
│   |   ├── 0.0.3/
│   |   |   └── ...
│   |   └── ...
│   └── ...
├── ModuleName.Build.ps1
├── appveyor.yml
└── ...
```

### InvokeBuild

For all building tasks I use [InvokeBuild](https://github.com/nightroman/Invoke-Build){:target="_blank"}. I never really looked at [psake](https://github.com/psake/psake){:target="_blank"} because I've always been happy with the InvokeBuild functionality.
I also like the `.Build.ps1` file name structure which has a nice similarity to the way Pester works.

> **Note**: If you know any reasons to choose psake over InvokeBuild feel free to give me your feedback!

### Appveyor

For some projects I use Appveyor to build and publish my modules. The `appveyor.yml` file lets you orchestrate this process.

[Top]

## VSCode

The VSCode folder and files looks like this:

```asci
ProjectFolder/
├── ...
├── .vscode/
│   ├── settings.json
│   └── tasks.json
└── ...
```

Since I only use VSCode to develop my PowerShell projects the .vscode folder has become a standard addition to any project.

The great thing is that you can enforce a few rules for everyone in the project using the settings.json:

```json
{
    // TABS not spaces!
    "editor.tabSize": 4,
    "editor.insertSpaces": false,
    // PowerShell formatting
    "powershell.codeFormatting.preset": "OTBS",
    // When enabled, will trim trailing whitespace when you save a file.
    "files.trimTrailingWhitespace": true
}
```

[Top]

## Meta

Every project has a few meta-files that define the what, who, how, and why.
These are the files I usually include:

```asci
ProjectFolder/
├── ...
├── .gitignore
├── LICENSE.txt
├── README.md
└── ...
```

### README

Just like it is essential you document your module it is also important you describe what the goal of your project is.

Also try to include a quick 'Getting Started' guide that has a step-by-step guide to assist people in getting a quick POC.

### LICENSE

[Choose a license!](https://choosealicense.com/)

Think about how people will interact with your project and what they could do with the code.

[Top]
