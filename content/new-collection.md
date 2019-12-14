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

```ps1
PS PipeHow:\Blog> 
```

Compare performance
Compare methods
Mention LINQ and IEnumerable
.PSObject

Array vs Arraylist
List vs HashSet
Hashtable vs Dictionary