---
title: "PowerShell Collections"
date: 2019-12-14T15:24:00+02:00
summary: "What are our options when working with collections in PowerShell. How do we declare an array, list, arraylist or another type of collection, and what are even the differences? In this series we're taking a look at different types of collections in PowerShell!"
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

The first type of collection you come across when looking for a traditional "list" in PowerShell is often an array. If you've ever seen code where a variable has square brackets afterwards, it shows that it's an array or collection. For example an array of objects of the type `[string]` would look like `[string[]]`. Although creating arrays is easy, you're rarely in a situation where you're required to say that an array is of a specific data type in PowerShell, since PowerShell isn't a very hard-typed language. Most of the time you either create an array implicitly through getting several results from a command, or by creating an empty array.

The type `System.Object` has a method called `GetType()` that we can use to figure out what type our created object is, which is useful since everything in .NET derives from `System.Object`. We can also use `Get-Member`, as long as we don't simply pipe our array to it, as that would call the command on the individual items in the collection and not show us the type of the actual array. If we really want to pipe our array to `Get-Member` we can do it using `Write-Output` with the parameter `NoEnumerate`.

```ps1
PS PipeHow:\Blog> (Get-Service).GetType().FullName
System.Object[]
PS PipeHow:\Blog> Get-Member -InputObject (Get-Service)

   TypeName: System.Object[]

Name           MemberType            Definition
----           ----------            ----------
Count          AliasProperty         Count = Length
Add            Method                int IList.Add(System.Object value)
Address        Method                System.Object&, mscorlib, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089 Address(int )
Clear          Method                void IList.Clear()
...
```

As we can see, when we declare an array without specifying a type, PowerShell simply creates a collection with the type `System.Object[]`, an array of items with the type `System.Object`. This allows for one array to contain several different types, since everything is an object.

We can explore this further by looking at the hidden `PSObject` property of any object in PowerShell. It contains a collection called `TypeNames` with the different types that the object inherits from, all the way to the top where `System.Object` resides as the base type.

```ps1
PS PipeHow:\Blog> (Get-Service)[0].PSObject.TypeNames
System.ServiceProcess.ServiceController
System.ComponentModel.Component
System.MarshalByRefObject
System.Object
```

What I did just now is refer to the first element of the array that `Get-Service` returns, using square brackets and an index. Arrays start at 0, so the first element of a collection is referred to by appending `[0]`. We could achieve the same result by piping to `Select-Object` with the parameter `First`.

```ps1
PS PipeHow:\Blog> $MyService = Get-Service | Select-Object -First 1
PS PipeHow:\Blog> $MyService.PSObject.TypeNames
System.ServiceProcess.ServiceController
System.ComponentModel.Component
System.MarshalByRefObject
System.Object
```

Almost every collection of items in PowerShell is an object array, if you haven't declared it as something else. You can declare your own arrays as well, either by declaring an empty array that can contain any type of object, or by specifying a type.

There are many ways to declare arrays in PowerShell. It often involves using `@(...)` to cast something into an array, but can be done in other ways as well.

```ps1
# Empty array
$MyArray = @()

# Array with one element 
$MyArray = @(1)
# Array with one element, the comma makes PowerShell cast it to an array
$MyArray = ,1

# Create an array with several elements
$MyArray = @(1,2)
$MyArray = 1,2
```

All of the above will result in a `System.Object[]` array, just as the commands we've previously run. The only difference between the results of the commands is the actual content of the array.

Sometimes we want to make sure that our array can only contains certain things, such as text or numbers. We can do this by explicitly declaring an array as a specific type. 

```ps1
PS PipeHow:\Blog> [int[]]$MyArray = @(1)
PS PipeHow:\Blog> $MyArray.GetType().FullName
System.Int32[]
```

This can be done with any type! That means we can force results of commands to be saved to an appropriate type as well, instead of PowerShell storing it in an array of objects we could use `Get-Service` to get us an array of `ServiceController[]`.

```ps1
PS PipeHow:\Blog> [System.ServiceProcess.ServiceController[]]$ServiceArray = Get-Service
PS PipeHow:\Blog> $ServiceArray.GetType().FullName
System.ServiceProcess.ServiceController[]
```

Even arrays can be arrays, so called 2D (of even more multi-dimensional) arrays. Arrays of arrays.

```ps1
PS PipeHow:\Blog> ([System.Array[]]@(1)).GetType().FullName
System.Array[]
```

While that may look a little confusing, I'm simply wrapping the first array in parentheses instead of saving it to a variable.

There is however a hefty downside to using arrays when it comes to adding or removing values from them. Every time you change the size of the array, a completely new array is created which replaces the previous array. This means that if you have 100 snippets of text in an array, adding another one will in the background require PowerShell to create a new array and add the 101 entries to it, throwing away the old array and then giving you a new one.

Let's create a function where we add X amount of elements to an array.

```ps1
function New-LargeArray {
    param([int]$Size)
    $Array = @()
    for ($i = 0; $i -lt $Size; $i++) {
        $Array += $i
    }
    Write-Output $Array
}
```

We can demonstrate the performance impact by using `Measure-Command` on our new function.

```ps1
PS PipeHow:\Blog> Measure-Command { $MyArray = New-LargeArray 5000 }

Days              : 0
Hours             : 0
Minutes           : 0
Seconds           : 0
Milliseconds      : 449
Ticks             : 4496647
TotalDays         : 5,2044525462963E-06
TotalHours        : 0,000124906861111111
TotalMinutes      : 0,00749441166666667
TotalSeconds      : 0,4496647
TotalMilliseconds : 449,6647
```

Not so bad, one might think, it's only half a second for 5000 iterations! If we increase it tenfold it would then be something like 4-5 seconds.

Well, not quite.

```ps1
PS PipeHow:\Blog> Measure-Command { $MyArray = New-LargeArray 50000 }

Days              : 0
Hours             : 0
Minutes           : 1
Seconds           : 12
Milliseconds      : 177
Ticks             : 721771485
TotalDays         : 0,000835383663194444
TotalHours        : 0,0200492079166667
TotalMinutes      : 1,202952475
TotalSeconds      : 72,1771485
TotalMilliseconds : 72177,1485
```

The problem as we can see with this way of increasing size is that adding an element to the array becomes exponentially slower as the size of the array increases. 

So what are our options?

There are a few ways for us to get around the problem, the easiest being not to create large arrays. This doesn't seem like a realistic goal, though, as we're eventually going to run into a situation where we need to gather a whole ton of users, computers or something else from a system and we're going to need to save them to a list of some sort.

We can avoid adding to the array during every iteration. If we rewrite our function to save the value being output from our loop directly, we would save a ton of time as the array would only need to be created once.

```ps1
function New-LargeArray {
    param([int]$Size)
    $Array = for ($i = 0; $i -lt $Size; $i++) {
        $i
    }
    Write-Output $Array
}
```

Let's run it again!

```ps1
PS PipeHow:\Blog> Measure-Command { $MyArray = New-LargeArray 50000 }

Days              : 0
Hours             : 0
Minutes           : 0
Seconds           : 0
Milliseconds      : 12
Ticks             : 127618
TotalDays         : 1,47706018518519E-07
TotalHours        : 3,54494444444444E-06
TotalMinutes      : 0,000212696666666667
TotalSeconds      : 0,0127618
TotalMilliseconds : 12,7618
```

From 1 minute and 12 seconds to **12 milliseconds**, that's quite the improvement. We're still running the same logic and if we take a look at our array we can see that it contains all the numbers that we added.

```ps1
PS PipeHow:\Blog> $MyArray.Count
50000
```

The third and final ways to not be affected by slowdowns when working with arrays, is not to work with arrays at all! We can use other collection types in .NET that grant us significant performance advantages over arrays, which we'll take a look at next.

For more reading on arrays, check out `Get-Help about_Arrays` or the online version on [docs](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_arrays).

- Compare performance
- Compare methods
- Mention LINQ and IEnumerable
- .PSObject

- Array vs Arraylist
- List vs HashSet
- Hashtable vs Dictionary