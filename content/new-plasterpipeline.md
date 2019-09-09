---
title: "Building PowerShell Modules in Azure"
date: 2019-05-21T22:17:00+02:00
summary: "Last time we took a look at the Plaster scaffolding tool to let us have a different mindset when thinking about how to create a module in PowerShell. Today we'll be taking it to the cloud using Azure pipelines to automate the building process!"
description: "Automated creation of PowerShell Modules in Azure DevOps pipelines using Plaster and gyPSum!"
keywords:
- Blog
- PowerShell
- Azure
- DevOps
- Pipelines
- Plaster
- Pester
- gyPSum
- Modules
draft: false
---

*This post will be fairly standalone, but it does build upon the module we created last time in [Building PowerShell Modules using Plaster]({{< relref "new-plastermodule.md" >}}), so if you haven't checked that out yet you can read through the demo project from scratch.*

Today we'll take a look at creating the same module as last time, but in the cloud using pipelines in Azure DevOps. The first thing to do is to sign up to [Azure DevOps](https://dev.azure.com/) if you haven't and then create an organisation and a project within it. I'm making mine public so you can look at the result, and I'm calling it [PlasterPipelines](https://dev.azure.com/pipehow/PlasterPipelines).

The second thing I did for the project was to create a [public repository](https://github.com/MrEpiX/PlasterPipeline/tree/master/PSBanking) on my GitHub where I store all my public code. If you joined me last time when we created the module you may remember that because we used gyPSum as the project template we got a few files for free to use when working with Git, namely the ```.gitignore```, ```license.txt``` and ```Readme.md``` files. I haven't changed a single thing since generating them, and because of gyPSum they're fully fleshed out with information on how to get started with the module.

Do make sure to include the whole file structure in the repository, including the main folder PSBanking, since some of the paths in the build scripts and tests are relative to it. This was something I got stuck on for a bit since my first attempts had been without the PSBanking folder and only the contents, which results in failed loading of the module in the tests. The alternative could of course be to change the paths around in the build and test scripts, but I chose to stick to the template.

Since we didn't include any [PowerShell Classes](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_classes) our Classes folder is empty. We will need the Classes folder to build our module but if you've worked with Git before you might know that Git does not keep track of empty repositories. To make sure that it was added to my repository I created a file called ```.gitkeep``` in the Classes folder, which is one of the few tricks you can use to get around this problem without weighing down the repository with files without a purpose. The file itself is as simple as it looks, it's an empty file only there to make Git accept the Classes folder into the repository.

```ps1
PS PipeHow:\Blog\PSBanking\Source\Classes> New-Item .gitkeep
```

## Module Building in the Cloud

Just like we explored in [Serverless Blogging using Hugo in Azure]({{< relref "new-blog.md" >}}) I created a build pipeline in our new DevOps project. This build pipeline is simple with two tasks, the first one being a PowerShell task with an inline script that installs the modules needed for building the module and simply executes Invoke-Build in the correct directory.

```
Install-Module InvokeBuild -Force
Install-Module PowerShellGet -Force
Install-Module ModuleBuilder -Force
Install-Module Pester -Force

cd .\PSBanking

Invoke-Build
```

The second task is of the type "Publish Pipeline Artifact", a very simple task which outputs my finished module as an artifact to be used or downloaded. I named it PSBanking, with the path to publish set to ```PSBanking\bin\PSBanking```.

I expected this blog post to drag on a bit longer but was surprised at how easy the project it was to set up in Azure DevOps. It took a few tries to get the directory levels correct in the paths, but as long as you make sure to import all required modules and don't change any paths around too much it's extremely easy to get a proper output. You could improve this even more and make scripts to create dynamic variables with the paths before-hand, or add them as variables in the pipeline to be able to re-use the whole pipeline after exporting it.

The only question left is what to use your module for. Will you [download it manually](https://dev.azure.com/pipehow/PlasterPipelines/_build/results?buildId=15&view=results), create a release pipeline and copy the artifact to an external storage or will you create a trigger that sends an email to your team telling them that the module has a new version? The next step is what you make it!