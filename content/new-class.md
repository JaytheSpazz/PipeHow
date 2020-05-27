---
title: "PowerShell Classes and Enums"
date: 2020-05-27T20:28:00+02:00
summary: "Classes are an often overlooked tool in PowerShell, familiar for programmers but sometimes foreign and confusing for those new to the world of PowerShell. There are often alternatives to writing classes but there are also situations where it shines. In this post we'll go through how to create one so that you can learn some of the strengths!"
description: "PowerShell classes and enums for directly binding behavior and validation to your data structure!"
keywords:
- Blog
- PowerShell
- Class
- Classes
- Method
- Get-Member
- Enum
- Enums
- Bit
- Flags
- Bitflags
- Structured
- Data
draft: false
---

Run a command with me.

```PowerShell
Get-Help 'about_Classes'
```

This could have been the whole blog post. There is so much useful information in [Microsoft's documentation](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_classes), directly integrated with PowerShell's help system.

Classes were added in PowerShell 5 and the feature lets people familiar with traditional object-oriented programming get to know PowerShell from a different angle. [Desired State Configuration](https://docs.microsoft.com/en-us/powershell/scripting/dsc/overview/overview) makes use of classes, and there are many areas where writing a PowerShell class could be a solution. But when is it the right one?

I've seen classes used in different ways, both where they solved a problem in a very elegant and structured way, but also where they could have been replaced by a simple, more clearly defined PowerShell function. It can sometimes be hard to know when to look at classes as an option.

## PowerShell Classes

I believe that the best way to learn code is to write code, so in this post we will together look at the different parts of a class, how to build in PowerShell and how to utilize the features that come with it. If you use Visual Studio Code with the PowerShell extension, there's a snipped called `ex-class` to create a quick example class to base your own class from.

```PowerShell
class MyClass {
    # Property: Holds name
    [String] $Name

    # Constructor: Creates a new MyClass object, with the specified name
    MyClass([String] $NewName) {
        # Set name for MyClass
        $this.Name = $NewName
    }

    # Method: Method that changes $Name to the default name
    [void] ChangeNameToDefault() {
        $this.Name = "DefaultName"
    }
}
```

A class is defined with a the keyword `class` followed by the name of the class and curly brackets. Everything belonging to the class is defined within the scope of those brackets, such as properties and methods. Above is a basic example which has a name property and lets you change it using a method.

### Methods

*Methods, you say? I've seen those when running `Get-Member` but compared to properties I've never actually created one. How do they work?*

You can think of a method as a function belonging to an object. A member function, so to say. Let's look at the objects returned from `Get-ChildItem` when run in a file system as an example.

```PowerShell
PS PipeHow:\Blog> $Files = Get-ChildItem 'C:\Temp' -File
PS PipeHow:\Blog> $Files

    Directory: C:\Temp

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a---          2020-05-27    11:51              9 test.txt
-a---          2020-05-27    11:51              6 test1.txt

PS PipeHow:\Blog> $Files | Get-Member -MemberType Method

   TypeName: System.IO.FileInfo

Name                      MemberType Definition
----                      ---------- ----------
AppendText                Method     System.IO.StreamWriter AppendText()
CopyTo                    Method     System.IO.FileInfo CopyTo(string destFileName), System.IO.FileInfo CopyTo(string destFileName, bool overwrite)
Create                    Method     System.IO.FileStream Create()
CreateText                Method     System.IO.StreamWriter CreateText()
Decrypt                   Method     void Decrypt()
Delete                    Method     void Delete()
Encrypt                   Method     void Encrypt()
Equals                    Method     bool Equals(System.Object obj)
GetHashCode               Method     int GetHashCode()
GetLifetimeService        Method     System.Object GetLifetimeService()
GetObjectData             Method     void GetObjectData(System.Runtime.Serialization.SerializationInfo info, System.Runtime.Serialization.StreamingContext context), void ISerializable.GetObj… 
GetType                   Method     type GetType()
InitializeLifetimeService Method     System.Object InitializeLifetimeService()
MoveTo                    Method     void MoveTo(string destFileName), void MoveTo(string destFileName, bool overwrite)
Open                      Method     System.IO.FileStream Open(System.IO.FileMode mode), System.IO.FileStream Open(System.IO.FileMode mode, System.IO.FileAccess access), System.IO.FileStream… 
OpenRead                  Method     System.IO.FileStream OpenRead()
OpenText                  Method     System.IO.StreamReader OpenText()
OpenWrite                 Method     System.IO.FileStream OpenWrite()
Refresh                   Method     void Refresh()
Replace                   Method     System.IO.FileInfo Replace(string destinationFileName, string destinationBackupFileName), System.IO.FileInfo Replace(string destinationFileName, string d… 
ToString                  Method     string ToString()
```

Using `Get-Member` we can find many methods on the `FileInfo` objects, this makes sense since they represent files and there are many operations we can do with them. The methods are often ways to modify the specific object, in this case we could among other things copy the file, move it or delete it using its member methods.

A method has an orderly structure that makes it quick to see what data it returns, if any, and what parameters it takes. Looking at the method `CopyTo` from above we can see that it has two variants, so-called overloads. Overloads are used when the same method can be run with different parameters, similar to parameter sets for PowerShell functions.

Calling members of an object is done with a dot, but something that sets properties and methods apart is that methods are called by using parentheses after the method, where the parameters are provided. Even if no input is needed for the method, such as `Delete` in the list above, the parentheses are still part of the syntax to call the method.

Trying to call a method without parentheses will therefore give you a different result than you might expect. Not to say it's not useful though, because that syntax is to list the overloads of the specified method. Let's look at an example for the first file we found in our folder.

```PowerShell
PS PipeHow:\Blog> $Files[0].CopyTo

OverloadDefinitions
-------------------
System.IO.FileInfo CopyTo(string destFileName)
System.IO.FileInfo CopyTo(string destFileName, bool overwrite)
```

There are two overloads of `CopyTo` that take different sets of parameters. Presumably one lets us copy the file to a destination and the other one lets us also overwrite any existing file there with the same name.

```PowerShell
PS PipeHow:\Blog> $Files[0].CopyTo('C:\Temp\test2.txt')

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a---          2020-05-27    11:51              9 test2.txt
```

We now have three files in our folder, but what if we try to copy the same file again now that it also exists with the new name?

```PowerShell
PS PipeHow:\Blog> $Files[0].CopyTo('C:\Temp\test2.txt')
```

**MethodInvocationException: Exception calling "CopyTo" with "1" argument(s): "The file 'C:\Temp\test2.txt' already exists."**

That was expected. So let's try the other overload and enable overwrite.

```PowerShell
PS PipeHow:\Blog> $Files[0].CopyTo('C:\Temp\test2.txt', $true)

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a---          2020-05-27    11:51              9 test2.txt
```

That was a simple example of how overloads can differ in behavior based on the parameters provided. You will find many other methods and overloads if you dive into the objects you work with daily.

Going back to the overload definitions, and even the syntax of a method, we can also see the return type of the method. Methods compared to functions can only output one object or collection of objects. It can also only output once, as the final thing of the method's code path, and the type of the output needs to be specified when writing the code. This provides you with clear information from just a glance if you're interested in knowing what a method provides you.

If we take a look back at the method we used above we will see that we already have the information of what it gives us back, the return type is specified in the very start of the method.

```plaintext
System.IO.FileInfo CopyTo(string destFileName, bool overwrite)
```

`System.IO.FileInfo` is the return type, meaning we get the information about the file copy we just created in case we want to further do something with it.

The method `Delete` on the other hand has the return type `[void]` which meaning it does not return anything.

```PowerShell
PS PipeHow:\Blog> $Files | Get-Member Delete

   TypeName: System.IO.FileInfo

Name   MemberType Definition
----   ---------- ----------
Delete Method     void Delete()
```

Confirming this is as simple as using the function on our file.

```PowerShell
PS PipeHow:\Blog> $Files[0].Delete()
```

No output, no error, and when looking at our directory the file is gone.

### Static Members

Members of a class can also be static. Compared to above where we needed a file object to copy from, static members lets you access them even without an instance of an object. A commonly used static method is the `IsNullOrEmpty` from the `[string]` class, used to validate whether or not a string is null or empty, though I'll admit I'm more of a `IsNullOrWhiteSpace` kind of guy.

These methods don't need you to already have a string to work with, you call them directly through the string class using the double colon operator `::`, the static member accessor.

```PowerShell
PS PipeHow:\Blog> [string]::IsNullOrWhiteSpace("    ")
True
```

We can see all static members of a class by using `Get-Member` on an object and specifying the parameter `-Static`.

```plaintext
PS PipeHow:\Blog> "ExampleString" | Get-Member -Static

   TypeName: System.String

Name               MemberType Definition
----               ---------- ----------
Compare            Method     static int Compare(string strA, string strB, bool ignoreCase), static int Compare(string strA, string strB, System.StringComparison comparisonType), static int … 
CompareOrdinal     Method     static int CompareOrdinal(string strA, string strB), static int CompareOrdinal(string strA, int indexA, string strB, int indexB, int length)
Concat             Method     static string Concat(System.Object arg0), static string Concat(System.Object arg0, System.Object arg1), static string Concat(System.Object arg0, System.Object a… 
Copy               Method     static string Copy(string str)
Create             Method     static string Create[TState](int length, TState state, System.Buffers.SpanAction[char,TState] action)
Equals             Method     static bool Equals(string a, string b), static bool Equals(string a, string b, System.StringComparison comparisonType), static bool Equals(System.Object objA, S… 
Format             Method     static string Format(string format, System.Object arg0), static string Format(string format, System.Object arg0, System.Object arg1), static string Format(strin… 
GetHashCode        Method     static int GetHashCode(System.ReadOnlySpan[char] value), static int GetHashCode(System.ReadOnlySpan[char] value, System.StringComparison comparisonType)
Intern             Method     static string Intern(string str)
IsInterned         Method     static string IsInterned(string str)
IsNullOrEmpty      Method     static bool IsNullOrEmpty(string value)
IsNullOrWhiteSpace Method     static bool IsNullOrWhiteSpace(string value)
Join               Method     static string Join(char separator, Params string[] value), static string Join(char separator, Params System.Object[] values), static string Join[T](char separat…
new                Method     string new(char[] value), string new(char[] value, int startIndex, int length), string new(System.Char*, System.Private.CoreLib, Version=4.0.0.0, Culture=neutra… 
ReferenceEquals    Method     static bool ReferenceEquals(System.Object objA, System.Object objB)
Empty              Property   static string Empty {get;}
```

In general you will see more static methods than properties among the members of a class, but properties can be static too as shown by the `Empty` property.

```PowerShell
PS PipeHow:\Blog> [string]::Empty -eq ''
True
```

As we can see, this is simply another way to write `''` or `""`. Note that properties are not referred to with parentheses, the way methods are.

## An Example Class - Money

As an exercise, let's build a class to represent money. As a simple start we can add a property to store the amount of money that our object has.

```PowerShell
class Money {
    [int] $Amount
}
```

Between each example in this section, assume that I run the code defining the class again to re-define the class in my PowerShell session, assume that I create a new `[Money]` object using the line below.

```PowerShell
PS PipeHow:\Blog> $Money = New-Object Money
```

Creating a class isn't harder than that, but what is hard is to see how a simple representation of an of money amount is going to be very useful.

```PowerShell
PS PipeHow:\Blog> $Money

Amount
------
     0
PS PipeHow:\Blog> $Money.Amount = 20
PS PipeHow:\Blog> $Money

Amount
------
    20
```

Let's add to our class. We want to be able to specify currency, and add a so-called constructor. You can think of the constructor as a method that is run when the object is created. The constructor is always, and needs to be, named the same as the class. Just like methods it can have different overloads and is in fact what is called in the background when using the cmdlet `New-Object`, as well as the `new` method you may see once in a while using the .NET syntax of object instantiation. We could for example use it like `[datetime]::new()`.

This syntax also lets us quickly look at what different constructors a class has by not adding the parentheses.

```PowerShell
PS PipeHow:\Blog> [datetime]::new

OverloadDefinitions
-------------------
datetime new(long ticks)
datetime new(long ticks, System.DateTimeKind kind)
datetime new(int year, int month, int day)
datetime new(int year, int month, int day, System.Globalization.Calendar calendar)
datetime new(int year, int month, int day, int hour, int minute, int second)
datetime new(int year, int month, int day, int hour, int minute, int second, System.DateTimeKind kind)
datetime new(int year, int month, int day, int hour, int minute, int second, System.Globalization.Calendar calendar)
datetime new(int year, int month, int day, int hour, int minute, int second, int millisecond)
datetime new(int year, int month, int day, int hour, int minute, int second, int millisecond, System.DateTimeKind kind)
datetime new(int year, int month, int day, int hour, int minute, int second, int millisecond, System.Globalization.Calendar calendar)
datetime new(int year, int month, int day, int hour, int minute, int second, int millisecond, System.Globalization.Calendar calendar, System.DateTimeKind kind)
```

Without getting too side-tracked, let's add the currency property and constructor to our class.

```PowerShell
class Money {
    [int] $Amount
    [string] $Currency

    # We can have several constructors, as well as empty ones for default values
    Money() {
        $this.Currency = 'SEK'
    }

    # We specify parameters comma separated as "[type] name"
    # Not specifying type will make it "System.Object"
    Money([int] $Amount, [string] $Currency) {
        # Set the $Amount property to the value of the $Amount parameter
        $this.Amount = $Amount
        $this.Currency = $Currency
    }
}
```

Looking at the code we can see something that is used only in PowerShell classes. The variable `$this` refers to the current instance of the class.

If we take a look at our class with `Get-Member` we can see that our class looks just like any other .NET object, and will also have a few methods inherited from `System.Object` by default, such as `ToString` and `GetType`. We can also see our constructors by looking at the `new` method.

```PowerShell
PS PipeHow:\Blog> $Money | Get-Member

   TypeName: Money

Name        MemberType Definition
----        ---------- ----------
Equals      Method     bool Equals(System.Object obj)
GetHashCode Method     int GetHashCode()
GetType     Method     type GetType()
ToString    Method     string ToString()
Amount      Property   int Amount {get;set;}
Currency    Property   string Currency {get;set;}

PS PipeHow:\Blog> [Money]::new

OverloadDefinitions
-------------------
Money new()
Money new(int Amount, string Currency)
```

For our third and final iteration of the class, let's go deeper (arguably too deep for an example) and add live data for exchange rates from an API, and functionality to validate the currency string using an enum.

## Enums

Enum? Isn't that a huge bird? No, that's the emu. An enum is an enumeration, a collection of labels representing pre-defined values, often used for validation for data where you know that it can only contain a value amongst a known set. Think of it as similar to `ValidateSet` used for function parameters.

In PowerShell you may have encountered the `System.DayOfWeek` enum when looking at `datetime` objects.

```PowerShell
PS PipeHow:\Blog> Get-Date | Get-Member DayOfWeek

   TypeName: System.DateTime

Name      MemberType Definition
----      ---------- ----------
DayOfWeek Property   System.DayOfWeek DayOfWeek {get;}

PS PipeHow:\Blog> [System.DayOfWeek]::Wednesday.GetType()

IsPublic IsSerial Name      BaseType
-------- -------- ----      --------
True     True     DayOfWeek System.Enum

PS PipeHow:\Blog> [int][System.DayOfWeek]::Wednesday
3
```

An enum is behind the scenes simply a set of defined numbers, and an instance of an enum is one of the numbers or sometimes a combination of them. This means we can also cast the enum to numbers if needed. The numbers start at 0 by default but the labels can be set manually. If only one label is set to a number, the following numbers will simply increment the number by 1.

In PowerShell you can define an enum using the `enum` keyword and separating each value by line.

### Bit Flags

There is also a way to create a structure known as bit flags in PowerShell, by using the `[Flags()]` attribute on the enum. This lets an instance of the enum have several values, and be represented by a sum of the numbers that the values represent. For it to work properly, each label must be set to a power of two value.

```PowerShell
[Flags()]
enum Weather {
    Sun = 1
    Rain = 2
    Wind = 4
    Hail = 8
    Snow = 16
}
```

We could then say that the weather is a combination, by using a sum of its numbers.

```PowerShell
PS PipeHow:\Blog> [Weather]7  

Sun, Rain, Wind
```

### A Currency Enum

To finish our `Money` class we want to represent the currencies using an enum, and change the type of the property `$Currency` to the new enum `[Currency]` below.

```PowerShell
enum Currency {
    SEK
    EUR
    USD
}
```

Simple! But what if we want the enum to be dynamic and read an external source of currencies? It's not possible to create a PowerShell enum without explicitly naming each label, but we can generate a string of the code to create the enum, and run it using `Invoke-Expression`.

Let's first get all the different currencies using the public [Exchange rates API](https://exchangeratesapi.io/) published by the European Central Bank.

```PowerShell
PS PipeHow:\Blog> $RatesData = (Invoke-RestMethod 'https://api.exchangeratesapi.io/latest').rates
PS PipeHow:\Blog> $RatesData

CAD : 0,1438382767
HKD : 0,8059452649
ISK : 14,5979178311
PHP : 5,2440722602
DKK : 0,7062891354
HUF : 33,1176643331
CZK : 2,5646296524
GBP : 0,0841942726
...
```

Next we will gather the names of each property, and store them in a list that we can use for our enum string.

```PowerShell
PS PipeHow:\Blog> $Rates = ($RateData | Get-Member -MemberType NoteProperty).Name
```

Finally we will create our string and generate each label of the enum as part of the string, as a one-liner but with adding new-lines in the loop.

```PowerShell
# Inside the currency brackets we loop through each rate and add new lines
$CurrencyEnum = "enum Currency{$(foreach ($Rate in $Rates){"`n$Rate`"})`n}"
Invoke-Expression $CurrencyEnum
```

Creating an enum in PowerShell is similar to C#, and you may see other solutions online using `Add-Type` with C# code instead of `Invoke-Expression`. Remember to not use these cmdlets on any strings you don't have full control over, or you may end up running malicious code that someone has injected.

```PowerShell
PS PipeHow:\Blog> [enum]::GetValues([Currency])

AUD
BGN
BRL
CAD
CHF
CNY
CZK
DKK
...
```

All of the different currencies are now in our enum, so let's put together our final version of the class.

## The Final Money Class

To finalize our class we add live data for exchange rates from the same API, and validation for our currency using our new enum.

```PowerShell
class Money {
    [int] $Amount
    # Change to Currency type instead of string to force input to be a valid option
    [Currency] $Currency
    # Static hashtable to store rates of chosen currency
    static [hashtable] $ExchangeRates

    Money() {
        # Call hidden initialize method with default values
        $this.Init(0,'SEK')
    }
    Money([int] $Amount, [Currency] $Currency) {
        $this.Init($Amount, $Currency)
    }

    # Create a hidden method to use from constructors
    hidden Init([int] $Amount, [Currency] $Currency) {
        $this.Amount = $Amount
        $this.Currency = $Currency

        # Create or reset the static ExchangeRates hashtable
        [Money]::ExchangeRates = @{}
        # Get live rates from API
        $Rates = (Invoke-RestMethod "https://api.exchangeratesapi.io/latest?base=$Currency").rates
        # Go through rates and add each value to the hashtable with name as key
        foreach ($Rate in ($Rates | Get-Member -MemberType NoteProperty).Name) {
            [Money]::ExchangeRates[$Rate] = $Rates.$Rate
        }
    }

    # Add method to convert the current amount to a chosen currency
    [decimal] GetAmountAsCurrency([Currency] $Currency) {
        return $this.Amount * [Money]::ExchangeRates[$Currency.ToString()]
    }
}
```

A few things have been added and changed as you can see. Firstly we changed the `$Currency` property to be of the type `[Currency]`, our dynamic enum that we generated. This makes sure that it will always be of a valid currency whenever we pass values to the different methods.

PowerShell does not have support for so-called constructor chaining, a way to call a constructor from another constructor. This would have let us provide default values in the no-parameter constructor to our overloaded one. Instead I opted to create a hidden method called `Init` that takes the same parameters and sets them, to have a consistent initializing of our class instance.

There is a new static hashtable called `ExchangeRates` which holds all the current exchange rates, but since it is static it is not bound to a specific object. Instead there will only ever be one hashtable at a time, which comes with its own challenges, but we could expand on this implementation to clear it only when needed and minimize the amount of API calls between new `[Money]` objects.

Finally we use the static hashtable in our new method `GetAmountAsCurrency` where we return the current amount converted to any specified currency with live rates from the API, validated using our enum.

There are endless ways to use classes and enums, and many more neat things you can do with them than we went through today. I hope you found your interest sparked to learn more!
