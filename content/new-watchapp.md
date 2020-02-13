---
title: "Building Galaxy Watch Apps in .NET"
date: 2020-01-27T22:17:00+02:00
summary: ""
description: ""
keywords:
- Blog
- App
- C#
- .NET
draft: true
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