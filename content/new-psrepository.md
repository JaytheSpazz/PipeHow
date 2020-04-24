---
title: "Creating a Private PSRepository in Azure"
date: 2020-04-22T15:28:00+02:00
summary: "In the Azure DevOps suite we have a service called Artifacts, which lets you create your own package feeds. It lets you publish your scripts and modules and share them with only a select few people if desired."
description: "Creating an internal PowerShell repository in Azure DevOps Artifacts."
keywords:
- Blog
- PowerShell
- Repository
- PSRepository
- PowerShellGet
- Private
- PSGallery
- Gallery
- Azure
- DevOps
- Artifacts
- Package
- Feed
- NuGet
draft: true
---

I set out to create an internal PSRepository for my scripts and modules, like a [PowerShell Gallery](https://www.powershellgallery.com/) but with authentication and only for a select few people. In the [Azure DevOps](https://azure.microsoft.com/en-us/services/devops/) suite there is something called [Azure Artifacts](https://docs.microsoft.com/en-gb/azure/devops/artifacts/overview?view=azure-devops), where you can create your own package feeds for hosting published code in different forms. This lets you manage part of your build dependencies in a structured way, and in PowerShell you can set it up and use it with [PowerShellGet](https://github.com/PowerShell/PowerShellGet), more known as the module in which `Install-Module` resides.

Creating the package feed was simple, but when I wanted to publish my modules I encountered more obstacles than I expected. A lot of the guides out there on hosting your own PowerShell repository are either out of date or not set up in Azure, so I thought I would demonstrate another way to do it.

## Creating the Feed

The first we will need is an organization and a project in Azure DevOps. In the project's Artifacts section we have the option to create a package feed. We have a few options mainly regarding visibility, or who should be able to use the feed. We can uncheck "Upstream sources" as we're only using and publishing our own PowerShell modules. If needed we can always add upstream sources later.

![New Package Feed](/img/new-psrepository/azure_devops_newfeed.png)

Technically we're already done with creating a PowerShell Repository, we just need to connect to it. The easy part is done!

## Working with the PowerShell Repository

We have our new feed, so our next step is to consume it somehow.

There are two main ways to work with the package feed as a PowerShell repository.

### NuGet CLI

The [NuGet CLI](https://docs.microsoft.com/en-gb/nuget/reference/nuget-exe-cli-reference) is what at least [one of the articles](https://docs.microsoft.com/en-us/azure/devops/artifacts/tutorials/private-powershell-library?view=azure-devops#creating-packaging-and-sending-a-module) in the official Microsoft documentation guides you through. It works, and while it might seem intuitive if you've worked with NuGet and .NET before I would like to argue that it doesn't feel natural when working in PowerShell. It requires you to create package files from the module catalogue which you then publish. When I want to publish a new module to my internal repository I would prefer it if I could use a more PowerShell-native way of doing it, avoiding to bring in further dependencies and commands to remember outside of PowerShell.

### The PowerShellGet module

There is a module called PowerShellGet dedicated to working with PowerShell repositories, modules, scripts and package management. The module is included with both Windows 10 and Windows Server 2016 or newer, as well as any modern PowerShell version, so most likely you already have at least an early version of it installed.

As of writing I'm using the highest live version on PSGallery, version `2.2.4.1`, and I'd recommend you to install it. [Version 3.0 is in preview](https://devblogs.microsoft.com/powershell/powershellget-3-0-preview-1/) however and reworks the entire way that the module works, removing dependencies and more. We will go into more detail on this further down.

## Registering the Repository

We're using the command `Register-PSRepository` from the current live version of PowerShellGet, and as you may have guessed it lets us register a specified PowerShell repository, such as our newly created package feed in Azure Artifacts.

In my case I want to register it as a PSRepository for my own computer, but it could also be an on-premises server, for an [Azure Function](https://azure.microsoft.com/en-us/services/functions/) or something else entirely.

### NuGet Feed URLs

The PowerShell repository that we're creating uses the NuGet feed in Azure Artifacts, which is also the reason that the NuGet CLI is one of the ways to publish modules for it. [PowerShell does not support version 3 feeds for NuGet](https://docs.microsoft.com/en-us/azure/devops/artifacts/tutorials/private-powershell-library?view=azure-devops#connecting-to-the-feed-as-a-powershell-repo), so we need to use version 2. We can find our feed address by clicking the "Connect to feed" button in Azure DevOps and clicking "NuGet.exe". The "Project setup" section has an URL that we can use.

![Connect to Package Feed](/img/new-psrepository/azure_devops_connectfeed.png)

The feed URL follows a standardized format which can be applied to all PowerShell repositories hosted in Azure Artifacts.

```PowerShell
# V3 feeds

# My feed URL
https://pkgs.dev.azure.com/pipehow/PSRepository/_packaging/PipeHowFeed/nuget/v3/index.json

# Feed URL format
https://pkgs.dev.azure.com/<organisation>/<project>/_packaging/<feedname>/nuget/v3/index.json
```

There are a couple of interesting things I found when trying to overcome the obstacle that is registering the repository in PowerShell, one being that the format obviously is not for v2.

The v2 feed is however very similar, you simply swap the `/v3/index.json` to `v2`, without the JSON file reference at the end. The documentation on finding this NuGet endpoint isn't very clear, although there is some information spread throughout some [seemingly outdated articles](https://docs.microsoft.com/en-us/azure/devops/artifacts/nuget/nuget-exe?view=azure-devops).

```PowerShell
# V2 feeds

# My feed URL
https://pkgs.dev.azure.com/pipehow/PSRepository/_packaging/PipeHowFeed/nuget/v2

# Feed URL format
https://pkgs.dev.azure.com/<organisation>/<project>/_packaging/<feedname>/nuget/v2
```

Several sources on the internet refer to the URL without "\<project\>" but I found that when I did not provide the project as part of the URL, PowerShell had troubles "resolving the package source". When I tried to find and install the module that I had published to my repository while testing the NuGet CLI, it couldn't. In the search for a v2 endpoint that worked, I took a look at what the v3 JSON file actually contained. It turned out that it had a couple of endpoints that let me find the correct format, including the project in the URL.

```JSON
{
    "@context": {
        "@vocab": "http://schema.nuget.org/services#",
        "comment": "http://www.w3.org/2000/01/rdf-schema#comment",
        "label": "http://www.w3.org/2000/01/rdf-schema#label"
    },
    "resources": [
        {
            "@id": "https://pkgs.dev.azure.com/pipehow/4aa30cdb-2056-4868-8e89-cf6200d8ee30/_packaging/3a0a60cf-6a69-4ecb-ae90-21eafb4160ff/nuget/v2/",
            "@type": "PackagePublish/2.0.0"
        },
        {
            "@id": "https://pkgs.dev.azure.com/pipehow/4aa30cdb-2056-4868-8e89-cf6200d8ee30/_packaging/3a0a60cf-6a69-4ecb-ae90-21eafb4160ff/nuget/v2/",
            "@type": "LegacyGallery/2.0.0"
        },
        {
            "@id": "https://pkgs.dev.azure.com/pipehow/4aa30cdb-2056-4868-8e89-cf6200d8ee30/_packaging/3a0a60cf-6a69-4ecb-ae90-21eafb4160ff/nuget/v3/registrations2/",
            "@type": "RegistrationsBaseUrl/3.0.0-beta"
        },
        {
            "@id": "https://pkgs.dev.azure.com/pipehow/4aa30cdb-2056-4868-8e89-cf6200d8ee30/_packaging/3a0a60cf-6a69-4ecb-ae90-21eafb4160ff/nuget/v3/registrations2-semver2/",
            "@type": "RegistrationsBaseUrl/3.6.0",
            "comment": "This base URL includes SemVer 2.0.0 packages."
        },
        {
            "@id": "https://pkgs.dev.azure.com/pipehow/4aa30cdb-2056-4868-8e89-cf6200d8ee30/_packaging/3a0a60cf-6a69-4ecb-ae90-21eafb4160ff/nuget/v3/registrations2-semver2/",
            "@type": "RegistrationsBaseUrl/Versioned",
            "comment": "This base URL includes SemVer 2.0.0 packages."
        },
        {
            "@id": "https://pkgs.dev.azure.com/pipehow/4aa30cdb-2056-4868-8e89-cf6200d8ee30/_packaging/3a0a60cf-6a69-4ecb-ae90-21eafb4160ff/nuget/v3/query2/",
            "@type": "SearchQueryService/3.0.0-beta"
        },
        {
            "@id": "https://pkgs.dev.azure.com/pipehow/4aa30cdb-2056-4868-8e89-cf6200d8ee30/_packaging/3a0a60cf-6a69-4ecb-ae90-21eafb4160ff/nuget/v3/flat2/",
            "@type": "PackageBaseAddress/3.0.0"
        },
        {
            "@id": "https://pkgs.dev.azure.com/pipehow/",
            "@type": "VssBaseUrl"
        },
        {
            "@id": "urn:uuid:3a0a60cf-6a69-4ecb-ae90-21eafb4160ff",
            "@type": "VssFeedId",
            "label": "PipeHowFeed"
        },
        {
            "@id": "com.visualstudio.feeds.feedview:3a0a60cf-6a69-4ecb-ae90-21eafb4160ff",
            "@type": "VssQualifiedFeedViewId",
            "label": "PipeHowFeed"
        },
        {
            "@id": "urn:uuid:4aa30cdb-2056-4868-8e89-cf6200d8ee30",
            "@type": "AzureDevOpsProjectId",
            "label": "PSRepository"
        }
    ],
    "version": "3.0.0-beta"
}
```

Take a look at the first two resources in the file.

```JSON
{
    "@id": "https://pkgs.dev.azure.com/pipehow/4aa30cdb-2056-4868-8e89-cf6200d8ee30/_packaging/3a0a60cf-6a69-4ecb-ae90-21eafb4160ff/nuget/v2/",
    "@type": "PackagePublish/2.0.0"
},
{
    "@id": "https://pkgs.dev.azure.com/pipehow/4aa30cdb-2056-4868-8e89-cf6200d8ee30/_packaging/3a0a60cf-6a69-4ecb-ae90-21eafb4160ff/nuget/v2/",
    "@type": "LegacyGallery/2.0.0"
}
```

The JSON file that the v3 feed consists of actually has two links to the v2 feed! Technically it's the same link, but the interesting thing that I found with it is that it still matches the format of the v2 feed.

Take another look at the feed format below, compared with our link from the JSON file.

```PowerShell
# Feed URL from JSON
https://pkgs.dev.azure.com/pipehow/4aa30cdb-2056-4868-8e89-cf6200d8ee30/_packaging/3a0a60cf-6a69-4ecb-ae90-21eafb4160ff/nuget/v2/
# Feed URL format
https://pkgs.dev.azure.com/<organisation>/<project>/_packaging/<feedname>/nuget/v2
```

It still matches! Except for the trailing slash the obvious difference is that the project and feed names have been replaced by their corresponding identifiers. After some testing I found that the GUIDs you see here and the names of the project and feed are interchangeable, meaning you can also have a mix of them if you'd like. The organization is still named although I suspect that if you found the GUID for it, it might be a valid substitute as well.

```PowerShell
# Mixing ids and names
https://pkgs.dev.azure.com/pipehow/PSRepository/_packaging/3a0a60cf-6a69-4ecb-ae90-21eafb4160ff/nuget/v2/
```

Enough URL format talk, let's register the repository in PowerShell now that we have our v2 feed URL.

### Register-PSRepository

Registering the repository is done using `Register-PSRepository` and providing the v2 feed URL for both the `-SourceLocation` and `-PublishLocation` parameters.

```PowerShell
$V2 = 'https://pkgs.dev.azure.com/pipehow/PSRepository/_packaging/PipeHowFeed/nuget/v2'

# Hashtable with parameters for splatting
$Params = @{
    'Name' = 'PipeHow'
    'InstallationPolicy' = 'Trusted'
    'SourceLocation' = $V2
    'PublishLocation' = $V2
}

Register-PSRepository @Params
```

The first time we register the repository we will get prompted for a manual browser login using a device code.

```plaintext
[Minimal] [CredentialProvider]DeviceFlow: https://pkgs.dev.azure.com/pipehow/_packaging/PipeHowFeed/nuget/v2
[Minimal] [CredentialProvider]To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code CBWSEFM23 to authenticate.
```

But after logging in we will be hit by an error.

*Register-PSRepository: The specified repository 'PipeHowFeed' is unauthorized and cannot be registered. Try running with -Credential.*

This brings us to what I've found to be the main problem with the current version of PowerShellGet, the authentication. When registering a repository hosted in Azure Artifacts there are supposedly three ways to manage the authentication, according to [an official blog post on the subject](https://devblogs.microsoft.com/powershell/using-powershellget-with-azure-artifacts/):

1. Register the repository without the -Credential parameter and use the device flow URL.
2. Explicitly provide a credential. To do this use the -Credential parameter when you register the PSRepository and provide a personal access token (PAT). For more information on how to get a PAT check out the documentation. Note that if you chose this method the credentials will not be cached.
3. Configure an environment variable with your credentials. To do set the VSS_NUGET_EXTERNAL_FEED_ENDPOINTS variable to [...]

Something to note from my experience when trying to authenticate with the three different methods is that I was still prompted to login using a device code, even when providing credentials. There are a few other inconsistencies between the information in the blog and what actually works, that I'll mention as we go through the three mentioned methods of authentication.

#### 1. The Device Flow

As we saw from the error message, simply going through the device flow without specifying the `-Credential` parameter didn't seem to work as well as it may have been intended according to the blog post. This is because I'm working from my personal computer with a local user.

#### 2. The Credential Parameter

The credential parameter turns out to be the only way to make it work, but what the blog post doesn't mention is that the credential to register the repository using `Register-PSRepository` doesn't need to have a [Personal Access Token (PAT)](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops) as password, it can also simply contain your password. Storing your password somewhere is of course often considered poor practice, but I thought it was worth noting. I'll come back to the credential parameter further down, as it turned out to be the key to solving the puzzle of making it work using the current version of PowerShellGet.

#### 3. The Environment Variable

The environment variable `VSS_NUGET_EXTERNAL_FEED_ENDPOINTS` can be set to a JSON configuration confined to one line.

```JSON
{"endpointCredentials": [{"endpoint":"https://pkgs.dev.azure.com/yourorganizationname/_packaging/yourfeedname/nuget/v2", "username":"yourusername", "password":"personalaccesstoken"}]}
```

Let's expand it and see what information is required.

```JSON
{
    "endpointCredentials": [
        {
            "endpoint": "https://pkgs.dev.azure.com/yourorganizationname/_packaging/yourfeedname/nuget/v2",
            "username": "yourusername",
            "password": "personalaccesstoken"
        }
    ]
}
```

The endpoint needs to be set to a valid v2 feed URL, and as I mentioned earlier I only managed to get it working by including the project name or id after the organization.When trying this I simply set it to the URL we found earlier.

This method of authentication looks like a decent option for some cases, assuming we would be comfortable having our PAT in plain text as an environment variable, but unfortunately it turned out that this way of authenticating didn't work either. It does work for a few of the other commands in the PowerShellGet module, but not for `Register-PSRepository`. If you would like to use this way of authentication, I did note that it solves the problem of needing to provide credential for at least the commands `Find-Module` and `Install-Module`, but the repository needs to already be registered.

In the end I chose to register my repository by passing a `PSCredential` object containing my account credentials to the command.

```PowerShell
$Cred = Get-Credential
$V2 = 'https://pkgs.dev.azure.com/pipehow/PSRepository/_packaging/PipeHowFeed/nuget/v2'
$Params = @{
    'Name' = 'PipeHow'
    'InstallationPolicy' = 'Trusted'
    'SourceLocation' = $V2
    'PublishLocation' = $V2
    'Credential' = $Cred
}
Register-PSRepository @Params
```

## Publishing a Module

The repository is now added, let's see if we can publish a module to it. How about trying the example module "PSBanking" that we made in a [previous blog post]({{< relref "new-plastermodule.md" >}})?

```PowerShell
Publish-Module -Path '.\PSBanking' -NuGetApiKey (New-Guid) -Repository PipeHow
```

`Publish-Module` works fine without providing specified credentials, which was a nice surprise. When publishing modules on PowerShell Gallery, the `-NuGetApiKey` would be the API key from your account, but when working with Azure Artifacts we authenticate differently. In our case this parameter can be any string, so I'm simply generating a new GUID.

The module is now uploaded and anyone can use it! Finding the module is easy, simply use `Find-Module` and optionally specify the repository to avoid any possible conflicts.

```plaintext
PS PipeHow:\Blog> Find-Module -Repository PipeHow

Version Name      Repository Description
------- ----      ---------- -----------
1.0.0   PSBanking PipeHow    A module that pretends to do bank transa…
```

Let's try to install our `Install-Module` to confirm it all works as intended.

```PowerShell
Install-Module PSBanking -Repository PipeHow
```

**PackageManagement\Install-Package : Unable to resolve package source [...].**

This is when I encountered the main problem, where I spent the most time trying to figure out what I did wrong. After testing several ways of adding the repository with different credentials and URLs it turns out that to install the module we need to provide credentials again. Not just any credentials though, this time we need a `PSCredential` object with the account as username and a [PAT with the rights to read, write and manage our feeds and packages](https://docs.microsoft.com/en-us/azure/devops/artifacts/tutorials/private-powershell-library?view=azure-devops#create-a-pat-to-get-command-line-access-to-azure-devops-services) as the password.

![New Personal Access Token](/img/new-psrepository/azure_devops_newpat.png)

The important thing when creating the token is to make sure to save it somewhere securely. In my case I directly entered it into `Get-Credential` in PowerShell so that I could provide it as part of the credential for `Install-Module`, but I also knew that I wanted to see if I could find another solution, so a temporary variable was fine for me.

```PowerShell
$PATCred = Get-Credential
Install-Module PSBanking -Repository PipeHow -Credential $PATCred
```

In a few of the ways in which I tried to make this work, `Install-Module` did not throw any errors, but when I looked closer with the `-Verbose` and `-Debug` parameters on the command I could see that it found the module, tried to download it three times and then seemed to simply give up. Using the credential with a PAT was the only way I found that worked.

```plaintext
PS PipeHow:\Blog> Get-InstalledModule PSBanking

Version Name      Repository Description
------- ----      ---------- -----------
1.0.0   PSBanking PipeHow    A module that pretends to do bank transa…
```

As we can see, the module was successfully installed and can now be used. Even if I would like to avoid the requirement for a PAT somehow, it seems to be the way to do it. Since it's generated once and not saved anywhere where you can read it, you will need to keep track of it somewhere secure. You will also need to keep track of the validity of the token, since it's time-based.

## Avoiding the Device Code

The thing that still bothered me about the solution even if we managed to use the repository is that once in a while, probably when an internal token expires, the commands pointing to our added repository will prompt you for a new device code login in the browser. This would ruin any attempt at automation since it requires manual intervention.

### PowerShellGet v3

Since the problem seemed to stem from how PowerShellGet was written to manage authentication, its' dependency on NuGet and more, let's take a look at installing version 3 of the module, and see if it can help us out.

```plaintext
PS PipeHow:\Blog> Find-Module PowerShellGet -AllowPrerelease

Version     Name          Repository Description
-------     ----          ---------- -----------
3.0.0-beta1 PowerShellGet PSGallery  PowerShell module with commands for discovering, installing, updating and publishing the PowerShell artifacts like Modules, DSC Resources, Role Capabilities and Scripts.
```

Since version 3 is not released yet, we need to specify the `-AllowPrerelease` switch to search for modules that are still in preview. Since it seemed to find what we needed, let's install it. Running `Find-Module` before `Install-Module` is a good way to know that you're installing the right module, since the PowerShell Gallery could potentially contain malicious code.

```PowerShell
Install-Module PowerShellGet -AllowPrerelease -Force
```

Specifying `-Force` in this case lets it install side-by-side with my current version. Let's see if we can find out what changed.

```plaintext
PS PipeHow:\Blog> Get-Command -Module PowerShellGet

CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Function        Find-Command                                       2.2.4.1    PowerShellGet
Function        Find-DscResource                                   2.2.4.1    PowerShellGet
Function        Find-Module                                        2.2.4.1    PowerShellGet
Function        Find-RoleCapability                                2.2.4.1    PowerShellGet
Function        Find-Script                                        2.2.4.1    PowerShellGet
Function        Get-CredsFromCredentialProvider                    2.2.4.1    PowerShellGet
Function        Get-InstalledModule                                2.2.4.1    PowerShellGet
Function        Get-InstalledScript                                2.2.4.1    PowerShellGet
Function        Get-PSRepository                                   2.2.4.1    PowerShellGet
Function        Install-Module                                     2.2.4.1    PowerShellGet
Function        Install-Script                                     2.2.4.1    PowerShellGet
Function        New-ScriptFileInfo                                 2.2.4.1    PowerShellGet
Function        Publish-Module                                     2.2.4.1    PowerShellGet
Function        Publish-Script                                     2.2.4.1    PowerShellGet
Function        Register-PSRepository                              2.2.4.1    PowerShellGet
Function        Save-Module                                        2.2.4.1    PowerShellGet
Function        Save-Script                                        2.2.4.1    PowerShellGet
Function        Set-PSRepository                                   2.2.4.1    PowerShellGet
Function        Test-ScriptFileInfo                                2.2.4.1    PowerShellGet
Function        Uninstall-Module                                   2.2.4.1    PowerShellGet
Function        Uninstall-Script                                   2.2.4.1    PowerShellGet
Function        Unregister-PSRepository                            2.2.4.1    PowerShellGet
Function        Update-Module                                      2.2.4.1    PowerShellGet
Function        Update-ModuleManifest                              2.2.4.1    PowerShellGet
Function        Update-Script                                      2.2.4.1    PowerShellGet
Function        Update-ScriptFileInfo                              2.2.4.1    PowerShellGet
Cmdlet          Find-PSResource                                    3.0.0      PowerShellGet
Cmdlet          Get-PSResource                                     3.0.0      PowerShellGet
Cmdlet          Get-PSResourceRepository                           3.0.0      PowerShellGet
Cmdlet          Install-PSResource                                 3.0.0      PowerShellGet
Cmdlet          Register-PSResourceRepository                      3.0.0      PowerShellGet
Cmdlet          Save-PSResource                                    3.0.0      PowerShellGet
Cmdlet          Set-PSResourceRepository                           3.0.0      PowerShellGet
Cmdlet          Uninstall-PSResource                               3.0.0      PowerShellGet
Cmdlet          Unregister-PSResourceRepository                    3.0.0      PowerShellGet
Cmdlet          Update-PSResource                                  3.0.0      PowerShellGet
```

They weren't joking when they said they reworked the module. None of the old commands remain, and they added new ones. Another thing to note is that the old ones are marked as functions, while the new ones are cmdlets. Functions are written in PowerShell while cmdlets are written in C#, and reading through [the official blog post](https://devblogs.microsoft.com/powershell/powershellget-3-0-preview-1/) it seems they have indeed rewritten the whole thing to abstract the different types of PowerShell resources into a single type. That sounds promising, let's see if we can register a PSResourceRepository!

```PowerShell
# Adding PSGallery since I didn't have it by default
Register-PSResourceRepository -PSGallery

# Registering our Azure Artifacts repository with the v2 URL as a trusted repository
Register-PSResourceRepository -Name PipeHow -URL 'https://pkgs.dev.azure.com/pipehow/PSRepository/_packaging/PipeHowFeed/nuget/v2' -Trusted
```

Trying to add the repository went smoothly, note that it still using the v2 endpoint and not v3. In fact it was so quick that I was actually doubting that anything happened, but looking at our registered repositories shows that it did. The registration did not require credentials so I suspect that the authentication occurs when you connect to the repository to find and install modules.

```plaintext
PS PipeHow:\Blog> Get-PSResourceRepository

Name      Url                                                                             Trusted Priority
----      ---                                                                             ------- --------
PipeHow   https://pkgs.dev.azure.com/pipehow/PSRepository/_packaging/PipeHowFeed/nuget/v2 true    50
PSGallery https://www.powershellgallery.com/api/v2                                        false   50
```

The new repositories come with a Priority property.

> "For the cmdlets Register-PSRepository and Set-PSRepository a -Priority parameter allows setting the search order of repositories with a lower value indicating a higher priority. If not specified, the default value is 50. The PSGallery, which is registered by default, has an editable value of 50. If two PSRepositories have the same priority the “Trusted” one will be chosen, if they also have the same level of trust the first one alphabetically will be selected."

```PowerShell
Find-PSResource PSBanking
```

**Find-PSResource: Failed to fetch results from V2 feed at 'https\://pkgs.dev.azure.com/pipehow/PSRepository/_packaging/PipeHowFeed/nuget/v2/FindPackagesById()?id='PSBanking'&semVerLevel=2.0.0' with following message : Response status code does not indicate success: 401 (Unauthorized).**

Using `Find-PSResource` and `Install-PSResource` still requires a credential with PAT. Unfortunately PowerShellGet v3 [hasn't implemented credential persistence yet](https://github.com/PowerShell/PowerShellGet/issues/102), so even if we would use the `-Credential` parameter when registering the repository it would not make a difference, but it might be something to keep track of!

We can then install the module Install-PSResource.

```PowerShell
# Uninstall the module from the previous installation
Uninstall-Module PSBanking

# Install the module again using PowerShellGet v3
Install-PSResource 'PSBanking' -Repository PipeHow -Credential $PATCred
```

The module is now installed, and you can remove it with `Uninstall-PSResource` if you'd like. The version 3 is still being developed, so among many other things the functionality to publish modules among other things is still not implemented, for now we still need to use `Publish-Module`.

It does however remove the need for the device code for some of the commands!

Stay on the lookout for some more functionality to the v3 module, which may make it more convenient to publish and work with modules in Azure Artifacts. Until then, this is how you do it!
