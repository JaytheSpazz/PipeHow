---
title: "Building PowerShell Modules with Plaster"
date: 2019-05-15T22:12:10+02:00
summary: "You've heard of them, you've seen them, maybe you've created some before! Modules in PowerShell are created and used in different ways. In the first part we'll take a look at creating a module using Plaster and gyPSum and in the second part we'll do it again but using Azure DevOps pipelines to automate it!"
description: "Creating PowerShell Modules using Plaster and gyPSum!"
keywords:
- Blog
- PowerShell
- Plaster
- Pester
- gyPSum
- Modules
draft: false
---

PowerShell has something beautiful called modules. You can think of them as building blocks that you can use in many projects or share with your friends, a true foundation of working in PowerShell. Generally you have one module per system, integration or area of your project, so if I created a PowerShell module that managed all my bills and bank transfers (technology needs to move faster) I could call it something along the lines of `PSBanking`.

Deciding on a naming convention before you create too many modules is a good idea, so why don't you run the following line of code in your console and check out the modules already on your computer.

```PowerShell
(Get-Command).Module.Name | Sort-Object -Unique
```

Note how it's often fairly obvious what type of commands each module contains, this is something to keep in mind when creating a module. Strive to name your modules something that represents the type of commands that will reside therein.

PowerShell has loads of built-in modules that you use every day, in fact all Cmdlets reside in a module of some sort. [Modules can vary in size and form](https://docs.microsoft.com/en-us/powershell/developer/module/understanding-a-windows-powershell-module) but most frequently you will find them written as `.psm1` files, containing "public" (I'll get back to this) PowerShell functions that are exported from the module file. Modules can also be written as a binary module using C# (that's a future blog post!) or created dynamically during runtime using the `New-Module` command.

Making modules is fairly easy, but making them well is a little more tricky because of how you're supposed to create both the code in a `.psm1` file or a binary, but also the [module manifest](https://docs.microsoft.com/en-us/powershell/developer/module/how-to-write-a-powershell-module-manifest) `.psd1` file containing all the meta data about your module.

Enter [Plaster](https://github.com/PowerShell/Plaster), a "template-based file and project generator written in PowerShell". In short Plaster is a scaffolding tool, letting you change the way you work when building modules, to (among many other ways outside of the scope of this article) what I would argue is a more tried-and-tested workflow used in traditional programming with a working copy and a build output which you then publish. Together with Plaster we will use [gyPSum](https://github.com/SimonWahlin/gyPSum/tree/master/Module), a Plaster template designed to fit what is considered best practices in the PowerShell community when building modules. This will also create a nice structure for our code with folders for our private and public functions as well as for our tests, so go ahead and download that together with Plaster.

```PowerShell
Install-Module Plaster
```

If you aren't able to use `Install-Module` you will find links to each of the modules as I introduce them, where you can follow the installation instructions.

We will need a few more module dependencies before we're ready to get started, so let's install those right away. First up is [Invoke-Build](https://github.com/nightroman/Invoke-Build) which we will use to invoke the module build job from the codebase we will put together. Secondly we have [PowerShellGet](https://docs.microsoft.com/en-us/powershell/module/powershellget), a module with commands to manage modules and scripts among other things. This one is included in Windows 10 among other setups, so chances are you might already have it (and if you don't, running Install-Module might be tricky to begin with), but you can update it if you like. Next there is [ModuleBuilder](https://github.com/PoshCode/ModuleBuilder), not too unsurprisingly also used as part of building our module. Lastly we have [Pester](https://github.com/pester/Pester), our go-to code testing framework in PowerShell.

```PowerShell
Install-Module InvokeBuild
Install-Module PowerShellGet
Install-Module ModuleBuilder
Install-Module Pester
```

That's everything for the setup of building our first module using Plaster!

## Setting up the Project

Let's start by creating a module project using `Invoke-Plaster` and specifying our gyPSum template file.

```PowerShell
Invoke-Plaster -TemplatePath .\gyPSum\Module -DestinationPath .
```

Plaster will greet you with some beautiful ASCII art and ask you to provide information based on the template used, in our case gyPSum. To continue with my earlier example, I'll create the module PSBanking.

```plaintext
  ____  _           _
 |  _ \| | __ _ ___| |_ ___ _ __
 | |_) | |/ _` / __| __/ _ \ `__|
 |  __/| | (_| \__ \ ||  __/ |
 |_|   |_|\__,_|___/\__\___|_|
                                            v1.1.3
==================================================
Enter the name of the module: PSBanking
Enter the description for the module (PSBanking module.): A module that pretends to do bank transactions as an example of how to build modules in Plaster.
Enter your full name: Emanuel Palm
Enter company name: PipeHow
Enter the version number of the module (0.1.0): 1.0.0

Select a editor for editor integration (or None):
[N] None [C] Visual Studio Code [?] Help (default is "None"): C

Select a license for your module
[A] Apache [M] MIT [C] Commercial [N] None [?] Help (default is "MIT"): M

Select desired options
[P] Pester test support [S] PSake build script [B] Invoke-Build build script [G] Gitingore [R] Readme [A] Appveyor [N] None [?] Help (default is "Pester test support, Invoke-Build build script, Gitingore, Readme, Appveyor"):

Scaffolding your PowerShell Module...

   Create PSBanking/Source\
   Create PSBanking/Source/Private\
   Create PSBanking/Source/Public\
   Create PSBanking/Source/Classes\
   Create PSBanking\Source\_PrefixCode.ps1
   Create PSBanking\Source\PSBanking.psd1
   Create PSBanking\Source\build.psd1
   Create PSBanking\Source\PSBanking.psm1
   Create PSBanking\Test\Unit\PSBanking.Tests.ps1
   Create PSBanking\.gitignore
   Create PSBanking\Readme.md
   Create PSBanking\PSBanking.build.ps1
   Create PSBanking\.vscode\settings.json
   Create PSBanking\.vscode\launch.json
   Create PSBanking\license.txt
   Create PSBanking\appveyor.yml
   Verify The required module Pester (minimum version: 3.4.0) is already installed.
   Verify The required module InvokeBuild (minimum version: 3.6.5) is already installed.
   Verify The required module ModuleBuilder (minimum version: 1.0.0) is already installed.

Your new PowerShell module project 'PSBanking' has been created.
A Pester test has been created to validate the module's manifest file. Add additional tests to the test directory.
```

Long block, I know, but you can safely ignore most of it. It's fairly self-explanatory, so if we move on and check out the destination path we specified we'll see that Plaster has created an empty project for us to fill with great PowerShell code that will be built into our new module. This is when the template really comes into play, and how the workflow of your project changes. Let's have a look at our new folder structure!

```plaintext
PSBanking/
├── .gitignore
├── appveyor.yml
├── license.txt
├── PSBanking.build.ps1
├── .vscode/
│   ├── launch.json
│   └── settings.json
├── Source/
│   ├── Classes/
│   ├── Private/
│   ├── Public/
│   ├── _PrefixCode.ps1
│   ├── build.psd1
│   ├── PSBanking.psd1
│   └── PSBanking.psm1
└── Test/
    └── Unit/
        └── PSBanking.Tests.ps1
```

The files that are already there are files that will be used when building the module. Have a look through them if you like, but you don't need to know exactly how they work to make use of it.

## The Project Structure

Using Plaster with the gyPSum template we will get a working directory with a few folders central to the module we will build. The key difference to how you normally would go about building a module is that with this setup we will split each function into its own `.ps1` file. If you've programmed in other languages you'll probably see some of the similarities I mentioned earlier.

### Private

All functions that you would traditionally mark as private should be put here. This means all module-internal functions that the user does not need or should not have access to. One of the main perks compared to a normal `.psm1` file is how we make sure that these functions do not end up exported from the module for the user to see.

In our case let's create an example function that the module could call internally. If we're handling imaginary transations it needs to validate an imaginary account number, so our first function should be `Test-AccountNumber`. It should take an account number as a parameter and return true or false depending on if it matches our imaginary banking system.

```PowerShell
function Test-AccountNumber
{
    [CmdletBinding()]
    param(
        [Parameter(
            Mandatory,
            Position = 0,
            ValueFromPipeline,
            ValueFromPipelineByPropertyName)]
        [ValidateNotNullOrEmpty()]
        [string]$AccountNumber
    )

    [bool]($AccountNumber -match '^\d{8}$')
}
```

Our function takes a mandatory string as input, checks if it contains exactly 8 digits and outputs true or false. We could have made it take an integer as input, but that would mean numbers starting with zero might make it not behave the way we expect it to. Functions in PowerShell output anything you write to the pipeline, so the three lines below would result in the same functionality.

```PowerShell
[bool]($AccountNumber -match '^\d{8}$')
return [bool]($AccountNumber -match '^\d{8}$')
Write-Object [bool]($AccountNumber -match '^\d{8}$')
```

Let's add two more private functions before we move onto the public ones, that actually moves our imaginary money. We'll also put a limit per transaction to 500 of whatever arbitrary currency this would use, let's say this was a specified need from the bank manager that ordered the module since their policy says they have to call the bank to transfer larger amounts.

```PowerShell
function Add-MoneyToAccount
{
    [CmdletBinding()]
    param(
        [Parameter(
            Mandatory,
            Position = 0,
            ValueFromPipelineByPropertyName)]
        [ValidateNotNullOrEmpty()]
        [string]$AccountNumber,

        [Parameter(
            Mandatory,
            Position = 1,
            ValueFromPipelineByPropertyName)]
        [ValidateRange(1,500)]
        [int]$Amount
    )

    Write-Verbose "Adding $($Amount) to account $($AccountNumber)."
}

function Remove-MoneyFromAccount
{
    [CmdletBinding()]
    param(
        [Parameter(
            Mandatory,
            Position = 0,
            ValueFromPipelineByPropertyName)]
        [ValidateNotNullOrEmpty()]
        [string]$AccountNumber,

        [Parameter(
            Mandatory,
            Position = 1,
            ValueFromPipelineByPropertyName)]
        [ValidateRange(1,500)]
        [int]$Amount
    )

    Write-Verbose "Removing $($Amount) from account $($AccountNumber)."
}
```

### Public

All the functions that should be exported from the module, or visible to the user, should be put in the public folder. Other than that our setup is the same, so let's create two more functions.

```PowerShell
function Get-AccountBalance
{
    [CmdletBinding()]
    param(
        [Parameter(
            Mandatory,
            Position = 0,
            ValueFromPipeline,
            ValueFromPipelineByPropertyName)]
        [ValidateScript({
            if(!(Test-AccountNumber $_))
            {
                throw 'Please provide a valid account number containing exactly 8 digits!'
            }
            $true
        })]
        [string]$AccountNumber
    )

    Get-Random 5000
}
```

As you can see, we use the private function Test-AccountNumber in our parameter validation, this is one of the ways we could utilize private functions in our module. The behavior is the same as I mentioned previously, I simply output `$true` if we haven't thrown an exception during the account number check.

Finally, let's throw together a transaction function for the user.

```PowerShell
function New-MoneyTransaction
{
    [CmdletBinding()]
    param(
        [Parameter(
            Mandatory,
            Position = 0,
            ValueFromPipelineByPropertyName)]
        [int]$Amount,

        [Parameter(
            Mandatory,
            Position = 1,
            ValueFromPipelineByPropertyName)]
        [ValidateScript({
            if(!(Test-AccountNumber $_))
            {
                throw 'Please provide a valid account number containing exactly 8 digits!'
            }
            $true
        })]
        [string]$FromAccountNumber,

        [Parameter(
            Mandatory,
            Position = 2,
            ValueFromPipelineByPropertyName)]
        [ValidateScript({
            if(!(Test-AccountNumber $_))
            {
                throw 'Please provide a valid account number containing exactly 8 digits!'
            }
            $true
        })]
        [string]$ToAccountNumber
    )

    if ($Amount -lt 1 -or $Amount -gt 500)
    {
        throw "The amount is larger than what is supported, please contact the bank!"
    }

    Remove-MoneyFromAccount -AccountNumber $FromAccountNumber -Amount $Amount
    Add-MoneyToAccount -AccountNumber $ToAccountNumber -Amount $Amount
}
```

It's a bit of code for such a small function, but all it really does is take two account numbers and an amount of money, validate the parameters and call our previously written private functions.

### Tests

If you haven't checked out Pester before, this will only be a brief overview of the framework. I plan to blog about it in the future, so look for those posts if they exist when you read this, otherwise simply google it!

The tests folder will all tests for our functions. The tests will be run when building the module, so we can make sure that everything is working as intended. There are generally two categories of tests, `Unit` and `Integration`. In this project we'll only be unit testing the code since there's no environment to actually do integration tests against, so we'll place our tests in the "Unit" folder.

For the sake of simplicity I'll place all the tests into the file `PSBanking.Tests.ps1` that Plaster generated for us. You could also decide to split the tests differently, in larger projects it's a good idea to for example split up test files per module even if you don't build them using Plaster. Something to keep in mind is that if we want to test the private functions of the module we need to either `Mock` their use, or use the function InModuleScope to run the code inside the scope of the module since the private functions are not exported and accessible otherwise.

Enough chatting, let's look at the test code.

```PowerShell
Describe 'PSBanking Function Tests' -Tag 'Unit' {
    BeforeAll {
        Import-Module "$ModulePath\$ModuleName.psd1"
    }

    AfterAll {
        Get-Module -Name $ModuleName | Remove-Module -Force
    }

    # Private functions tests in module scope
    InModuleScope $ModuleName {
        Context 'Test-AccountNumber' {
            It 'Is true when correct format' {
                Test-AccountNumber '12345678' | Should -BeTrue
            }
            It 'Is false when incorrect format' {
                Test-AccountNumber 'abc123' | Should -BeFalse
            }
        }

        Context 'Add-MoneyToAccount' {
            It 'Does not throw with correct values' {
                { Add-MoneyToAccount -AccountNumber '12345678' -Amount 250 } | Should -Not -Throw
            }
            It 'Throws with too high amount' {
                { Add-MoneyToAccount -AccountNumber '12345678' -Amount 750 } | Should -Throw
            }
            It 'Throws with negative amount' {
                { Add-MoneyToAccount -AccountNumber '12345678' -Amount -50 } | Should -Throw
            }
        }

        Context 'Remove-MoneyFromAccount' {
            It 'Does not throw with correct values' {
                { Remove-MoneyFromAccount -AccountNumber '12345678' -Amount 250 } | Should -Not -Throw
            }
            It 'Throws with too high amount' {
                { Remove-MoneyFromAccount -AccountNumber '12345678' -Amount 750 } | Should -Throw
            }
            It 'Throws with negative amount' {
                { Remove-MoneyFromAccount -AccountNumber '12345678' -Amount -50 } | Should -Throw
            }
        }
    }

    # Public function tests
    Context 'Get-AccountBalance' {
        It 'Does not throw with correct values' {
            $Balance = Get-AccountBalance '12345678'

            $Balance | Should -BeGreaterOrEqual 0
            $Balance | Should -BeLessOrEqual 5000
        }
        It 'Throws with incorrect account number format' {
            { Get-AccountBalance 'abc123' } | Should -Throw
        }
    }

    Context 'New-MoneyTransaction' {
        # Another way to manage private functions in tests
        Mock 'Add-MoneyToAccount' -ModuleName $ModuleName -MockWith {} -Verifiable
        Mock 'Remove-MoneyFromAccount' -ModuleName $ModuleName -MockWith {} -Verifiable

        It 'Adds and removes money and does not throw with correct calues' {
            { New-MoneyTransaction 250 '12345678' '87654321' } | Should -Not -Throw
        }

        Assert-VerifiableMock

        It 'Throws with incorrect values' {
            { New-MoneyTransaction 750 'abc123' 'def456' }
        }
    }
}
```

I realize that some tests are not true unit tests since they call the functions without mocking external sources, for example Get-Random, but that's not the focus of this post so I will leave that for another day. The gist of it is that I make sure to test the different scenarios of each function such as true or false, or if it throws or not depending on the values provided through the parameters. I also do some simple mocking in the test for `New-MoneyTransaction` to show how you can test the module's private functions.

### Classes
Classes is another folder that you can utilize, but nothing we will touch on in the scope of this demo. The purpose of the folder is to contain declarations of [PowerShell Classes](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_classes), but since we're not using any classes we'll move on.

# Building the Module

That's it really!

Running `Invoke-Build` in the root directory of the project will create a `bin` folder containing the module for use, distribution or publishing. Plaster handles all the formatting as well as most of the documentation of the module through the generated manifest, although some things can or should be tweaked for an optimal result.

```PowerShell
PS PipeHow:\Blog\PSBanking> Invoke-Build
PS PipeHow:\Blog\PSBanking> Import-Module .\bin\PSBanking\1.0.0\PSBanking.psd1
PS PipeHow:\Blog\PSBanking> Get-Command -Module PSBanking

CommandType Name                 Version Source
----------- ----                 ------- ------
Function    Get-AccountBalance   1.0.0   PSBanking
Function    New-MoneyTransaction 1.0.0   PSBanking
```

As you can see, if we import the `.psd1` file the only functions that were exported from the module were the ones we put in the `Public` folder. And since we're both curious if it actually works with the internal usage of our private functions, let's test it out!

```PowerShell
PS PipeHow:\Blog\PSBanking> Get-AccountBalance '12345678'
1238

PS PipeHow:\Blog\PSBanking> New-MoneyTransaction 150 '12345678' '87654321' -Verbose
VERBOSE: Removing 150 moneys from account 12345678.
VERBOSE: Adding 150 moneys to account 87654321.
```

We specified mandatory and positional parameters all throughout the module functions as well as validation, which you can really appreciate once you're on the using side of it, even if it's some extra lines of code. 

This might have been a fairly long read, but hopefully you didn't find it to be a too very complex one because in the next part we'll take it to the next level! We'll look at recreating the steps in Azure Pipelines to build the module when new code is pushed to the GitHub repository that we'll create for the project!