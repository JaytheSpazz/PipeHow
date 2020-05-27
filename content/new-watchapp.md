---
title: "Building Watch Apps in C#"
date: 2020-02-24T18:17:00+02:00
summary: "I recently bought a smartwatch upon hearing that you can develop apps for it in .NET using Xamarin. This is my start-up guide and experience with the platform so far."
description: "Developing smart watch apps using .NET and Xamarin with Tizen."
keywords:
- Blog
- App
- C#
- Xamarin
- Forms
- Tizen
- .NET
draft: true
---

For many years I've been intrigued with the idea of developing apps for my phone, and I've dabbled in app development a little bit using [Xamarin](https://dotnet.microsoft.com/apps/xamarin) and C# which has been a lot of fun. Between all the app ideas I write down and plan to some day start developing, there's a certain one that my mind always wander back to. Without spoiling too much of the fun, let's just say it has passed the stage of ideas and become a proof of concept for myself.

Roughly a month back I was talking to my coworker [Simon](https://twitter.com/simonwahlin) who had bought a Samsung Galaxy Watch Active 2 and he mentioned that you can develop apps for it in .NET. I'd been considering getting a smartwatch, but this together with the fact that it supports eSIM so you can use it independently of your phone when needed sold me. Developing fun apps for a watch that isn't dependent on keeping your Bluetooth-enabled phone close-by? Sign me up!

## Tizen .NET

I ended up buying the same watch as my coworker and started looking into the development platform of Tizen, the operating system that runs on the watch I had bought amongst other Samsung products such as smart TVs.

I've had to Google my way through a lot of the problems when getting started, but the best starting point that I found was the [Tizen article](https://docs.microsoft.com/en-us/xamarin/xamarin-forms/platform/other/tizen) in the Xamarin Forms documentation, with links to various resources.

### Getting Started

You're able to develop apps for Tizen using Visual Studio, Visual Studio Code as well as Tizen Studio, an IDE based on Eclipse which is mostly used for Java development. I decided to use Visual Studio but also checked out the [Visual Studio Code extension](https://marketplace.visualstudio.com/items?itemName=tizen.vscode-tizen-csharp) as a fully functional alternative.

Getting started with Tizen .NET in Visual Studio has two main prerequisites, in the form of the SDK and the Visual Studio extension [Visual Studio Tools for Tizen](https://developer.tizen.org/development/visual-studio-tools-tizen/installing-visual-studio-tools-tizen), which in turn has more prerequisites to be able to compile and deploy your apps.

A few things are required for Visual Studio Tools for Tizen to work.

#### Java

As of writing, you need to have either Oracle JDK 8 or OpenJDK 12 and OpenJFX to use the Tizen SDK. I tried both, but had the best experience with the Oracle JDK. If you decide to go for the open source one instead, make sure not to miss the installation of OpenJFX and also to select the correct version of OpenJDK according to the [installation guide](https://developer.tizen.org/development/articles/openjdk-and-openjfx-installation-guide#install-openjdk-for-windows).

Depending on your choice you may have to set an environment variable called `JAVA_HOME` as well as edit your `Path` variable. You can verify your Java version by running `java -version` on your computer, using for example cmd or PowerShell, mine is "java version 1.8.0_241".

If your version doesn't seem to match, or if you're running OpenJDK and not getting the correct result when running `java -version` make sure your environment variables are set correctly. `JAVA_HOME` should point to wherever the JDK is located, like "...\Java\jdk-12.x.x". The `Path` should also have the same directory added, but with "\bin" at the end. See [this answer](https://stackoverflow.com/a/32241360) of a question at StackOverflow regarding the setting if you're curious.

#### Emulation

If you're running Windows and want to run an emulator to develop against, make sure to disable Hyper-V on your computer. You can easily do this with administrative rights using PowerShell.

```PowerShell
PS PipeHow:\Blog> Disable-WindowsOptionalFeature -FeatureName Microsoft-Hyper-V-Hypervisor -Online
```

#### Visual Studio Tools

[Visual Studio Tools for Tizen](https://marketplace.visualstudio.com/items?itemName=tizen.VisualStudioToolsforTizen) is an extension for Visual Studio, required to be able to work with the Tizen .NET platform from Visual Studio. In Visual Studio, go to Tools > Extensions > Manage Extensions and find and install "Visual Studio Tools for Tizen" under the Online-section. This will install the Tizen development suite and integrate it with Visual Studio.



---

Samsung Certificate Extension

Tizen visual studio things
- Emulator packages + samsung cert extension (samsung wearable extension cause it can't hurt)
    - Versions matching phone at least (4 for me)
    - Tizen SDK tools
JAVA_HOME Path
OpenJDK and OpenJFX

Visual Studio
SDB

Remote Device Manager
IP Address (DHCP or Router)

Samsung Account Certificate Profile
Options>Tizen>Certification>Sign bla with new profile
Include your device's DUID (should show up if connected?) Can be retrieved from Device Manager
Direct Registration works, but not profile manager? Both seem to work, but was tricky at start. Possibly something with password complexity?
https://developer.samsung.com/forum/board/thread/view.do?boardName=SDK&messageId=361966&listLines=40&startId=zzzzz~&searchType=ALL&searchText=2.3

https://github.com/Samsung/Tizen-CSharp-Samples/tree/master/Wearable

https://developer.tizen.org/ko/development/tizen-studio/web-tools/running-and-testing-your-app/sdb?langredirect=1

VS 2019 
https://samsung.github.io/Tizen.NET/tizen%20.net/Using-Tizen-NET-Sdk-as-SDK-style/#manual-migration
https://samsung.github.io/Tizen.NET/tizen%20.net/Update-TizenNETSdk-package/#dont-panic-application-built-on-visual-studio-163-or-above-will-crash

https://developer.tizen.org/forums/tizen-.net/2d-graphic-on-samsung-watch
    <RuntimeIdentifier>tizen-armel</RuntimeIdentifier>
    <PackageReference Include="Tizen.NET.Sdk" Version="1.0.5" />
Not valid ^

Reference Tizen .NET
Xamarin Forms
Xamarin Essentials - Does not seem to work? Maybe with permissions?
Otherwise we get "Frame not in module"

try catch works to get error messages, exceptions will cause above error ^

Location
https://developer.tizen.org/development/guides/.net-application/location-and-sensors/location-information#start

Privilege requesting
https://developer.tizen.org/development/guides/.net-application/security/privacy-related-permissions

Device manager
etc