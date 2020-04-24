---
title: "PowerShell Collections: ArrayList"
date: 2020-01-17T22:24:00+02:00
summary: "\"I wanted to create a list of objects in PowerShell and it turned out a list is called an array, then what is this arraylist thing?\""
description: "Creating arraylists in PowerShell!"
keywords:
- Blog
- PowerShell
- Arraylist
- Arraylists
- Array
- Arrays
- Collection
- Collections
- .NET
- dotnet
draft: false
---

[Last time]({{< relref "new-array.md" >}}) we took a look at the most basic type of collection in PowerShell, and explored how we could specify exactly what type of array we wanted. It turned out that arrays are extremely slow when modifying size, as they are actually static structures that need to be re-created every time we add an element. This post will look at the first alternative to arrays that one usually encounters: `ArrayList`.

**A disclaimer before diving into the ArrayList, however, is that it is deprecated and the recommendation from Microsoft is that it should not be used for any new development. I'm writing about it to help explore its basic functionality and the differences between collections as you are likely to encounter it in existing scripts, but will follow this post by writing about recommended alternatives.**

Let's start with how to create an ArrayList in PowerShell, and then discuss how it works.

As with all things in PowerShell there are a couple of different ways to create an ArrayList.

```ps1
PS PipeHow:\Blog> $ArrayList = New-Object System.Collections.ArrayList
PS PipeHow:\Blog> $ArrayList = [System.Collections.ArrayList]::new()
```

One uses `New-Object` and the other calls the constructor of the class. You're free to use whichever you prefer, although when sharing scripts it might be worth considering that people that are just getting into PowerShell and learning commands may be confused at the second one.

## The [ArrayList] Type

So far it's nothing too exciting, but let's have a look at the type and members of it to see if we can spot some differences to a normal `Array`.

```ps1
PS PipeHow:\Blog> $ArrayList.GetType()

IsPublic IsSerial Name                                     BaseType
-------- -------- ----                                     --------
True     True     ArrayList                                System.Object
```

Something interesting to note here is that compared to an array, the ArrayList is not strongly typed, it can contain any object!

*"But wait.. I've created arrays that contain several types, you even talked about how PowerShell lets you do that in your last post!"*

You're not wrong! But let's have a look at the type of a normal array first.

```ps1
PS PipeHow:\Blog> (@()).GetType()

IsPublic IsSerial Name                                     BaseType
-------- -------- ----                                     --------
True     True     Object[]                                 System.Array
```

The result is an array that can only contain elements of the type `Object`. The reason we get away with putting anything in it is that, as we went through last time, any object we declare inherits from `System.Object` in .NET, so although the array can have different objects in it, it is still strongly typed to only contain that specific type.

The collection on today's topic is however not strongly typed, and therefore it can inherently contain anything.

Diving a little deeper, we saw earlier that `ArrayList` inherits directly from `System.Object` while `Object[]` had `System.Array` as it's base type. Last time we looked at the inheritance chain of the results of `Get-Service`, which looked like this:

```ps1
PS PipeHow:\Blog> (Get-Service)[0].PSObject.TypeNames
System.ServiceProcess.ServiceController
System.ComponentModel.Component
System.MarshalByRefObject
System.Object
```

At the top, so to speak, is unsurprisingly `System.Object`, but that doesn't show the entire picture of what type of functionality that the object implements.

## Interfaces

In .NET there is something called interfaces, not often touched upon in PowerShell circles, but if you have experience with C# you'll recognize it. You can think of an interface as a contract defining how a class, such as `ArrayList`, will behave. Implementing an interface in your class will require you to implement the functionality that the interface has defined, such as certain methods.

In PowerShell we can find out what interfaces something has implemented by looking at the underlying type.

```ps1
PS PipeHow:\Blog> $ArrayList.GetType().GetInterfaces()

IsPublic IsSerial Name                                     BaseType
-------- -------- ----                                     --------
True     False    IList
True     False    ICollection
True     False    IEnumerable
True     False    ICloneable
```

Naming conventions of interfaces dictate that they should start with a capital I and we can see that it implements a few of them. We could dive into the rabbit hole of exploring the interfaces one by one to see what members they implement but it would make for a very long post, so if you're curious I suggest that you have a look at the documentation of the interfaces as they contain some useful information in understanding why collections behave like they do.

## ArrayList vs Array

Next up is of course to compare the interfaces with our good old `Array`. To keep you from scrolling up and down, let's use `Compare-Object`!


```ps1
PS PipeHow:\Blog> Compare-Object (@()).GetType().GetInterfaces() $ArrayList.GetType().GetInterfaces() -IncludeEqual

InputObject                                                     SideIndicator
-----------                                                     -------------
System.Collections.IList                                        ==
System.Collections.ICollection                                  ==
System.ICloneable                                               ==
System.Collections.IEnumerable                                  ==
System.Collections.IStructuralComparable                        <=
System.Collections.IStructuralEquatable                         <=
System.Collections.Generic.IList`1[System.Object]               <=
System.Collections.Generic.ICollection`1[System.Object]         <=
System.Collections.Generic.IEnumerable`1[System.Object]         <=
System.Collections.Generic.IReadOnlyList`1[System.Object]       <=
System.Collections.Generic.IReadOnlyCollection`1[System.Object] <=
```

From the side indicators we can see that `Array` implemements the same interfaces as `ArrayList`, but has a few more as well. This is because `Array` is a generic class that needs to be typed, as we did during the last post when creating specifically typed arrays, or what PowerShell does for us when creating an `Object[]`. The \`1 is the .NET representation of how many type parameters there are on the given generic type, its so-called [arity](https://en.wikipedia.org/wiki/Arity). For some type of collections in future posts we'll see \`2 instead, so stay tuned for that excitement!

Actually looking through the list gives us some hints to why `Arrays` behave like they do and it explains why it needs to create new arrays whenever the size is modified, they're read-only!

Arraylists have plenty of methods you can explore by using `Get-Member`, and while some are shared with arrays as shown above, some are different as well. The main one to use is of course `Add` and something to keep in mind when using it is that it will output the index of the element it just stored.

You may want to suppress this output unless you're going to use it, which is possible using a few different ways depending on if you're after readability or performance, from top to bottom as follows.

```ps1
PS PipeHow:\Blog> $ArrayList.Add(1) | Out-Null
PS PipeHow:\Blog> [void]$ArrayList.Add(1)
PS PipeHow:\Blog> $null = $ArrayList.Add(1)
```

Let's finish up with a comparison of performance between an `Array` and an `ArrayList` now that we've gone through some structural differences between the classes.

```ps1
function New-LargeArray {
    param([int]$Size)
    $Array = @()
    for ($i = 0; $i -lt $Size; $i++) {
        $Array += $i
    }
    Write-Output $Array
}

function New-LargeArrayList {
    param([int]$Size)
    $Array = New-Object System.Collections.ArrayList
    for ($i = 0; $i -lt $Size; $i++) {
        $ArrayList.Add($i)
    }
    Write-Output $ArrayList
}
```

Above are a couple of functions just like in the last post, to help us measure the performance when adding a lot of elements to the two types of collections.

```ps1
PS PipeHow:\Blog> Measure-Command { $MyArray = New-LargeArray 50000 }

Days              : 0
Hours             : 0
Minutes           : 0
Seconds           : 45
Milliseconds      : 610
Ticks             : 456108072
TotalDays         : 0,000527902861111111
TotalHours        : 0,0126696686666667
TotalMinutes      : 0,76018012
TotalSeconds      : 45,6108072
TotalMilliseconds : 45610,8072
PS PipeHow:\Blog> Measure-Command { $MyArrayList = New-LargeArrayList 50000 }

Days              : 0
Hours             : 0
Minutes           : 0
Seconds           : 0
Milliseconds      : 32
Ticks             : 323949
TotalDays         : 3,74940972222222E-07
TotalHours        : 8,99858333333333E-06
TotalMinutes      : 0,000539915
TotalSeconds      : 0,0323949
TotalMilliseconds : 32,3949
```

As we can see, there is a substantial difference between them, which is the main reason one would use arraylists above arrays.

## Don't Use ArrayLists!

What we don't have with an `ArrayList` though is type safety, and compared to using a typed instance of a collection we also lose out on performance as PowerShell will need to cast between `System.Object` and the actual type of the element in the list when we reference it.

This is also true for the default array that PowerShell creates for us, which is another reason to declare your arrays as the type you will store in them unless you're planning to store different types in the same array.

I will leave you with a quote from [.NET on GitHub](https://github.com/dotnet/platform-compat/blob/master/docs/DE0006.md) on the topic.

*"When .NET was created, generic data types didn't exist, which is why the collection types in the System.Collections namespace are untyped. However, since then, generic data types were introduced and thus a new set of collections were made available in the System.Collections.Generic and System.Collections.ObjectModel namespaces."* 

And with that, look forward to next part where we're taking a look at more collection types!