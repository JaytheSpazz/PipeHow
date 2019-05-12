---
title: "Building PowerShell Modules in Azure Pipelines"
date: 2019-05-10T19:45:10+02:00
summary: "You've heard of them, you've seen them, maybe you've created some before! Modules in PowerShell are created and used in different ways, let's take it to the next level using Azure, Pipelines, Plaster and gyPSum!"
description: "Creating PowerShell Modules using Azure DevOps, Pipelines, Plaster and gyPSum!"
keywords:
- Blog
- Azure
- PowerShell
- DevOps
- Pipelines
- Plaster
- Pester
- gyPSum
- Modules
draft: true
---

PowerShell has something beautiful called modules. You can think of them as building blocks that you can use in many projects or share with your friends, a true foundation of working in PowerShell. Generally you have one module per system, integration or area of your project, so if I created a PowerShell module that managed all my bills and bank transfers (technology needs to move faster) I could call it something along the lines of ```PSBanking```.

Deciding on a naming convention before you create too many modules is a good idea, so why don't you run the following line of code in your console and check out the modules already on your computer.

```ps1
PS PipeHow:\Blog> (Get-Command).Module.Name | Sort-Object -Unique
```

Note how it's often fairly obvious what type of commands each module contains, this is something to keep in mind when creating a module. Strive to name your modules something that represents the type of commands that will reside therein.

PowerShell has loads of built-in modules that you use every day, in fact all Cmdlets reside in a module of some sort. [Modules can vary in size and form](https://docs.microsoft.com/en-us/powershell/developer/module/understanding-a-windows-powershell-module) but most frequently you will find them written as ```.psm1``` files, containing "public" (I'll get back to this) PowerShell functions that are exported from the module file. Modules can also be written as a binary module using C# (that's a future blog post!) or created dynamically during runtime using the ```New-Module``` command.

Making modules is fairly easy, but making them well is a little more tricky because of how you're supposed to create both the code in a ```.psm1``` file or a binary, but also the [module manifest](https://docs.microsoft.com/en-us/powershell/developer/module/how-to-write-a-powershell-module-manifest) ```.psd1``` file containing all the meta data about your module.

Enter [Plaster](https://github.com/PowerShell/Plaster), a "template-based file and project generator written in PowerShell". In short Plaster is a scaffolding tool, letting you change the way you work when building modules, to (among many other ways outside of the scope of this article) what I would argue is a more tried-and-tested workflow used in traditional programming with a working copy and a build output which you then publish. Together with Plaster we will use [gyPSum](https://github.com/SimonWahlin/gyPSum/tree/master/Module), a Plaster template designed to fit what is considered best practices in the PowerShell comunity when building modules. This will also create a nice structure for our code with folders for our private and public functions as well as for our tests, so go ahead and download that together with Plaster.

```ps1
PS PipeHow:\Blog> Install-Module Plaster
```

If you aren't able to use ```Install-Module``` you will find links to each of the modules as I introduce them, where you can follow the installation instructions.

We will need a few more module dependencies before we're ready to get started, so let's install those right away. First up is [Invoke-Build](https://github.com/nightroman/Invoke-Build) which we will use to invoke the module build job from the codebase we will put together. Secondly we have [PowerShellGet](https://docs.microsoft.com/en-us/powershell/module/powershellget), a module with commands to manage modules and scripts among other things. Next there is [ModuleBuilder](https://github.com/PoshCode/ModuleBuilder), not too unsurprisingly also used as part of building our module. Lastly we have [Pester](https://github.com/pester/Pester), our go-to code testing framework in PowerShell.

```ps1
PS PipeHow:\Blog> Install-Module InvokeBuild
PS PipeHow:\Blog> Install-Module PowerShellGet
PS PipeHow:\Blog> Install-Module ModuleBuilder
PS PipeHow:\Blog> Install-Module Pester
```

That's everything for the setup of building our first module using Plaster!