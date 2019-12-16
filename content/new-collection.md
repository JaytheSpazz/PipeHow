---
title: "Working with Collections in PowerShell"
date: 2019-12-14T15:24:00+02:00
summary: "PowerShell often creates collections of items for us, but what are our options when explicitly wanting to declare an array, list, arraylist or another type of collection, and what are even the differences?"
description: "Creating different types of lists and arrays in PowerShell!"
keywords:
- Blog
- PowerShell
- Array
- Arrays
- List
- Lists
- Hashtable
- Hashtables
- Dictionary
- Dictionaries
- .NET
- dotnet
draft: true
---

When I started writing PowerShell I quickly realized that there are several ways to create collections of items and they all have different uses and behaviors. PowerShell will automatically create an array for you if something returns several results, but sometimes you need to create an array, arraylist or list manually. What are the differences, use cases and how do they work?

## Arrays

The first type of collection you come across when looking for a traditional "list" in PowerShell is often an array. If you've ever seen code where a variable has square brackets afterwards, it shows that it's an array or collection. For example an array of `[string]` would look like `[string[]]`. Although creating arrays is easy, you're rarely in a situation where you're required to say that an array is of a specific data type in PowerShell, since PowerShell isn't a hard-typed language. Most of the time you either create an array implicitly through getting several results from a command, or by creating an empty array.

Since everything in .NET derives from `System.Object` we can use one of the inherited methods called `GetType()` to figure out what type our object is. We can also use `Get-Member`, as long as we don't pipe to it, as that would call the command on the individual items in the collection and not show us the type of the actual array.

```ps1
PS PipeHow:\Blog> (Get-Process).GetType().FullName
System.Object[]
PS PipeHow:\Blog> Get-Member -InputObject (Get-Process)

   TypeName: System.Object[]

Name           MemberType            Definition
----           ----------            ----------
Add            Method                int IList.Add(System.Object value)
Address        Method                System.Object&, System.Private.CoreLib, Version=4.0.0.0, Culture=neutral, PublicKeyTo… 
Clear          Method                void IList.Clear()
...
```

As we can see, when we declare an array without specifying a type, PowerShell simply creates a collection with the type `System.Object[]`, an array of items with the type `System.Object`. This allows for one array to contain several different types, since everything is an object.

We can explore this further by looking at the hidden `PSObject` property of any object in PowerShell. It contains a collection called `TypeNames` with the different types that the object inherits from, all the way to the top where `System.Object` resides as the base type.

```ps1
PS PipeHow:\Blog> (Get-Process)[0].PSObject.TypeNames
System.Diagnostics.Process
System.ComponentModel.Component
System.MarshalByRefObject
System.Object
```

What I did just now is refer to the first element of the array that `Get-Process` returns, using square brackets and an index. Arrays start at 0, so the first element of a collection is referred to by appending `[0]`. We could achieve the same result by piping to `Select-Object` with the parameter `First`.

```ps1
PS PipeHow:\Blog> $MyProcess = Get-Process | Select-Object -First 1
PS PipeHow:\Blog> $MyProcess.PSObject.TypeNames
System.Diagnostics.Process
System.ComponentModel.Component
System.MarshalByRefObject
System.Object
```

Compare performance
Compare methods
Mention LINQ and IEnumerable
.PSObject

Array vs Arraylist
List vs HashSet
Hashtable vs Dictionary