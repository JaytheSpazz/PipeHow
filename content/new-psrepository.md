---
title: "Hosting a Private PSRepository in Azure"
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

The [NuGet CLI](https://docs.microsoft.com/en-us/azure/devops/artifacts/tutorials/private-powershell-library?view=azure-devops#creating-packaging-and-sending-a-module) is what at least one of the articles in the official Microsoft documentation guides you through. It works, and while it might seem intuitive if you've worked with NuGet and .NET before I would like to argue that it doesn't feel natural when working in PowerShell. It requires you to create package files from the module catalogue which you then publish. When I want to publish a new module to my internal repository I would prefer it if I could use a more PowerShell-native way of doing it, avoiding to bring in further dependencies and commands to remember outside of PowerShell.

### The PowerShellGet module

There is a module called PowerShellGet dedicated to working with PowerShell repositories, modules, scripts and package management. The module is included with both Windows 10 and Windows Server 2016 or newer, as well as any modern PowerShell version, so most likely you already have at least an early version of it installed.

As of writing I'm using the highest live version on PSGallery, version `2.2.4.1`, and I'd recommend you to install it. [Version 3.0 is in preview](https://devblogs.microsoft.com/powershell/powershellget-3-0-preview-1/) however and reworks the entire way that the module works, removing internal dependencies and more. We will go into more detail on this further down.

## Registering the Repository

We're using the command `Register-PSRepository` from the current live version of PowerShellGet, and as you may have guessed it lets us register a specified PowerShell repository, such as our newly created package feed in Azure Artifacts.

In my case I want to register it as a PSRepository for my own computer, but it could also be an on-premises server, for an [Azure Function](https://azure.microsoft.com/en-us/services/functions/) or something else entirely.

### NuGet Feed URLs

The PowerShell repository that we're creating uses the NuGet feed in Azure Artifacts, which is also the reason that the NuGet CLI is one of the ways to publish modules for it. PowerShell does not support version 3 feeds for NuGet, so we need to use version 2. We can find our feed address by clicking the "Connect to feed" button in Azure DevOps and clicking "NuGet.exe". The "Project setup" section has an URL that we can use.

![Connect to Package Feed](/img/new-psrepository/azure_devops_connectfeed.png)

The feed URL follows a standardized format which can be applied to all PowerShell repositories hosted in Azure Artifacts.

```PowerShell
# Version 3 feeds

# My feed URL
https://pkgs.dev.azure.com/pipehow/PSRepository/_packaging/PipeHowFeed/nuget/v3/index.json

# Feed URL format
https://pkgs.dev.azure.com/<organisation>/<project>/_packaging/<feedname>/nuget/v3/index.json
```

There are a couple of interesting things I found when trying to overcome the obstacle that is registering the repository in PowerShell, one being that the format obviously is not for version 2.

The version 2 feed is however very similar, you simply swap the `/v3/index.json` to `v2`, without the JSON file reference at the end. The documentation on finding this NuGet endpoint isn't very clear, although there is some information spread throughout some [seemingly outdated articles](https://docs.microsoft.com/en-us/azure/devops/artifacts/nuget/nuget-exe?view=azure-devops).

```PowerShell
# Version 2 feeds

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

Just for the sake of it, let's compare the results between specifying the project name (or id) and only specifying the organization.

```PowerShell
$V2NoProj = 'https://pkgs.dev.azure.com/pipehow/_packaging/PipeHowFeed/nuget/v2'
$Params = @{
    'Name' = 'PipeHowFeed'
    'InstallationPolicy' = 'Trusted'
    'SourceLocation' = $V2NoProj
    'PublishLocation' = $V2NoProj
}
Register-PSRepository @Params
```

The first time we register the repository we will get prompted for a manual browser login using a device code. 

```plaintext
[Minimal] [CredentialProvider]DeviceFlow: https://pkgs.dev.azure.com/pipehow/_packaging/PipeHowFeed/nuget/v2
[Minimal] [CredentialProvider]To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code CBWSEFM23 to authenticate.
```

Even after logging in we will still be hit by an error.

*Register-PSRepository: The specified repository 'PipeHowFeed' is unauthorized and cannot be registered. Try running with -Credential.*

This brings us to what I've found to be the main problem with the current version of PowerShellGet, the authentication. When registering a repository hosted in Azure Artifacts there are supposedly three ways to manage the authentication, according to [an official blog post on the subject](https://devblogs.microsoft.com/powershell/using-powershellget-with-azure-artifacts/):

1. Register the repository without the -Credential parameter and use the device flow URL.
2. Explicitly provide a credential. To do this use the -Credential parameter when you register the PSRepository and provide a personal access token (PAT). For more information on how to get a PAT check out the documentation. Note that if you chose this method the credentials will not be cached.
3. Configure an environment variable with your credentials. To do set the VSS_NUGET_EXTERNAL_FEED_ENDPOINTS variable to [...]

Something to note from my experience when trying to authenticate with the three different methods is that I was still prompted to login using a device code, even when providing credentials. There are a few other inconsistencies between the information in the blog and what actually works, that I'll mention as we go through the three mentioned methods of authentication.

#### 1. The Device Flow

As we saw from the error message, simply going through the device flow without specifying the `-Credential` parameter didn't seem to work as well as it may have been intended according to the blog post.

#### 2. The Credential Parameter

The credential parameter turns out to be the only way to make it work, but what the blog post doesn't mention is that the credential to register the repository using `Register-PSRepository` doesn't need to have a Personal Access Token (PAT) as password, it can also simply contain your password. Storing your password somewhere is of course often considered poor practice, but I thought it was worth noting. I'll come back to the credential parameter further down, as it turned out to be the key to solving the puzzle of making it work using the current version of PowerShellGet.

#### 3. The Environment Variable

The environment variable `VSS_NUGET_EXTERNAL_FEED_ENDPOINTS` can be set to a JSON configuration confined to one line.

```JSON
{"endpointCredentials": [{"endpoint":"https://pkgs.dev.azure.com/yourorganizationname/_packaging/yourfeedname/nuget/v2", "username":"yourusername", "password":"accesstoken"}]}
```

Let's expand it and see what information is required.

```JSON
{
    "endpointCredentials": [
        {
            "endpoint": "https://pkgs.dev.azure.com/yourorganizationname/_packaging/yourfeedname/nuget/v2",
            "username": "yourusername",
            "password": "accesstoken"
        }
    ]
}
```

The endpoint needs to be set to a valid v2 feed URL, and as I mentioned earlier I only managed to get it working by including the project name or id after the organization.

This method of authentication looks like a decent option for some cases, assuming we would be comfortable having our PAT in plain text as an environment variable, but unfortunately it turned out that this way of authenticating didn't work either. It does work for a few of the other commands in the PowerShellGet module, but not for `Register-PSRepository`. If you would like to use this way of authentication, I did note that it solves the problem of needing to provide credential for at least `Find-Module` and `Install-Module`, but the repository needs to already be registered.

---

Must do it with -Credential unless you're logged into the account on your computer, for example at a work computer.

## Avoiding a Personal Access Token

We keep coming back to needing a PAT, so how do we get one.

And can we avoid the requirement somehow? As it's Time-based, security etc

---

Credentials or PAT when registering repository | PAT credentials when installing the module
Inputting credentials every time is a bother

or

$env:VSS_NUGET_EXTERNAL_FEED_ENDPOINTS
{"endpointCredentials": [{"endpoint": "https://pkgs.dev.azure.com/advaniase/TeamAutomation/_packaging/PwrOps/nuget/v2","username": "account","password": "PAT"}]}
this is bad for security

What if we want to not use a PAT - Let's check out PowerShellGet v3

Works with both v2 endpoints - Both GUID and by Name

Need to install higher versions of PowerShellGet than 1.0, recommend going for highest live version (before going into v3 details)


v2 legacy endpoint from the v3 JSON, v3 is not supported for powershell (https://docs.microsoft.com/en-us/azure/devops/artifacts/tutorials/private-powershell-library?view=azure-devops#creating-packaging-and-sending-a-module)
a lot of information mention nuget, PAT and the commandlet asks you for a device code, but none seemed to work
https://devblogs.microsoft.com/powershell/using-powershellget-with-azure-artifacts/
PowerShellGet has v3 in preview

Mention the /Packages JSON blob metadata info thing

Does Version 3 feeds work with v3 PowerShellGet?