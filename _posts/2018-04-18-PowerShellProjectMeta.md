---
layout: post
title:  "PowerShell Project Meta"
subtitle: ""
date:   2018-04-18
tags: PowerShell
background: '/img/posts/default.jpg'
comments: true
---

When starting out with PowerShell you copy random stuff from the web to your shell.
Then you start writing some scripts you can actually re-run.
In time you might start to build modules, but then... how does this scale?
Where do I put everything? How do other people work?

Many people have written about this and here I would like to summarize the parts I used.

I will try to reference as many websites/blogs as possible that have influenced me, but I did not really keep a log so I will probably forget a few.

**Table of Content**

[Top]: #powershell-project-meta

- [Editor](#editor)
- [Module](#module)
- [Versioning](#versioning)
- [Documentation](#documentation)
- [Tests](#tests)
- [Building](#building)
- [Publishing](#publishing)

## Editor

I did no extensive testing on this part but I started out using PowerGUI several years ago, and this was more then sufficient at that time. I tried PowerShell ISE a few times but was never really convinced.

Once [**VSCode**](https://code.visualstudio.com/){:target="_blank"} came out I switched never looked back.  
The possible modifications and extensions are really good and I don't see any other editor out there that is this well developed and supported.

[Top]

## Module

Once you start writing more scripts and tackling more complex problems you will need to start writing Modules.

I am not going into detail as to why and how you can build a module because this has been too extensively documented before. (My favorite article about this is Michael Sorens' [Further down the rabit hole](https://www.red-gate.com/simple-talk/dotnet/net-tools/further-down-the-rabbit-hole-powershell-modules-and-encapsulation/){:target="_blank"})

I will, however, go into the details of [the structure I use for my PowerShell modules (and projects)]({{ site.url }}/2018/04/17/PowerShellModuleStructure){:target="_blank"}.

[Top]

## Versioning

Like many Ops-guys I started scripting without actually knowing anything about version control.
Of course after a while, when the pile of code started becoming a lot bigger I understood the need.

At the moment I think the choice of versioning system is a no-brainer: [**GIT**](https://git-scm.com/){:target="_blank"}  
[Learn it](https://try.github.io){:target="_blank"}, live it, love it!

[Top]

## Documentation

You think you understand what you write, you think the code speaks for itself.  
It doesn't! Try reading the oldest complex-ish script you ever wrote. Without documentation and/or comments it will take you way too long to decipher what is there.

**Use comment based help! Use inline comments to support your thought process.**
```powershell
# Check if someData meets the Complex output requirements
if(Get-ComplexOutput -Stuff $someData){
    # someData is really complex so we should do stuff
    Do-Stuff
} else {
    # someData is not really complex so we should gracefully inform the user.
    throw 'you mad?'
}
```
> **Tip**: You can use `Write-Verbose` instead of a lot of your comment based help. This will be usefull when debugging later on.

[Top]

## Tests

I wasn't a big fan of test driven development. And in some cases I might still even agree that it is a bit too much work. But the satisfaction, ease of mind, and security well designed tests give you is worth a lot.

When going fully into the test driven development, where you build your tests before you write your code, your goals become very clear an you mitigate the risk of scope creep. Writing these test forces you to really think about what it is you want to deliver.

As to how you should develop your tests, there is only one choice for PowerShell: [**Pester**](https://github.com/pester/Pester)

## Building

Building always sounds a bit weird when it comes to PowerShell. *This already works, no need to compile or anything, right?*

But when you start creating more complex modules you start to realize there are always several tasks you want to execute.  
For me it usually consists of running the tests, generating/updating the documentation, and sometimes publishing.

When building I have used [InvokeBuild](https://github.com/nightroman/Invoke-Build){:target="_blank"} to write my build script. I've been very happy with the flexibility and readability of the scripts. I also like the .Build.ps1 file format since it complements the .Tests.ps1 file format of Pester.

[Top]

## Publishing

You won't want to publish everything publically but you will want to **thing about how you want to deliver what you make to whoever or whatever will consume it**.

This is an area I am still growing in and I won't make any bold statements here, just describe how I solved this in a few cases.

### Company internal or private Modules

For internal use the easiest way is to setup an internal PowerShell repository using a share. 
[Kevin Marquette](https://twitter.com/kevinmarquette){:target="_blank"} has a [nice article](https://kevinmarquette.github.io/2017-05-30-Powershell-your-first-PSScript-repository/){:target="_blank"} that describes how to do this.

### Public Modules

When you want to make your modules available to the public the best way is to publish them to [PSGallery](https://www.powershellgallery.com/){:target="_blank"}.
Make sure you read Microsoft's documentation about [publishing to the PowerShell Gallery](https://docs.microsoft.com/en-us/powershell/gallery/psgallery/psgallery-publishingguidelines){:target="_blank"}.

> **Tip**: You can use describe your publishing steps in a build task to fully automate the publishing of a module.

[Top]
