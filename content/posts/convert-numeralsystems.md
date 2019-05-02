---
title: "Working with Numeral Systems in PowerShell"
date: 2019-05-02T16:45:10+02:00
summary: ""
draft: false
---

Sometimes you need to work with number systems other than our good old decimal base 10. There are various ways to do this depending on what your goal is, and today I will list a few easy ways and some more ambitious ones.

Since PowerShell is built with a strong connection to the .NET framework (and Core!) we can access the static methods from ```[System.Convert]``` which helps us out in most of the cases, using ```[Convert]``` for short. When using the Convert class we will always need to decide what data type we want our result in. The Convert class has many different integer sizes to convert to, so how do you know which one to use? It depends on how large numbers you want to be able to handle, but generally when talking about integers we actually mean an ```Int32``` which the normal ```[int]``` data type is in fact an alias for. This can be verified by simply writing the data type in the console.

```ps1
PS PipeHow:\Blog> [int]

IsPublic IsSerial Name  BaseType
-------- -------- ----  --------
True     True     Int32 System.ValueType
```

The only downside of using the Convert class is that it only has support for binary, octal, decimal and hexadecimal, but we'll look at a solution for the edge (or all) cases later.

## Binary
Binary is pretty straight forward. If we need to convert to binary we need to handle the result as a string, since PowerShell does not have support for binary itself as a data type yet. Some upcoming changes however are happening in PowerShell 7.0 according to [this pull request](https://github.com/PowerShell/PowerShell/pull/7993) and the fact that [PowerShell is skipping 6.3 straight to 7.0](https://devblogs.microsoft.com/powershell/the-next-release-of-powershell-powershell-7/) which means we might have a way to change that. Meanwhile, we can use the previously mentioned class by specifying our decimal number (you can also pass it as a string) and what base we want to convert to, in this case binary which is base 2.

```ps1
# From Decimal to Binary
PS PipeHow:\Blog> [Convert]::ToString(13, 2)
1101

# From Binary to Decimal
PS PipeHow:\Blog> [Convert]::ToInt32(1101, 2)
13
```

## Hexadecimal
Hexadecimal (not to be confused with ```[int16]```!) is a common numeral system to encounter when working with color values, among other things. When working with colors each byte usually represents the amount of red, green or blue, this is the foundation of the RGB color system. In PowerShell you can write hexadecimal by prepending the number with ```0x```. When converting to hexadecimal we can use the Convert class again.

```ps1
# From Decimal to Hexadecimal
PS PipeHow:\Blog> [Convert]::ToString(321321, X)
4e729

# From Hexadecimal to Decimal
PS PipeHow:\Blog> 0x4e729
321321
PS PipeHow:\Blog> [Convert]::ToInt32(0x4e729, 16)
3281697
```

Wait, what happened there? We tried to convert it in the same way as we did with binary, but not only did the number not match, it's also way bigger!

As we see in the line above, ```0x4e729``` is interpreted as ```321321```. This is correct, but it also means that when we pass it to the method we're telling PowerShell that we want to convert ```321321``` from base 16 which isn't quite what we intend to do.

```ps1
PS PipeHow:\Blog> [Convert]::ToInt32(321321, 16)
3281697
```

As seen above this will give us an incorrect result, the equivalent of ```0x321321```. To fix it we simply need to pass it as a string so that PowerShell will understand that it's not actually ```321321``` in base 16.

```ps1
PS PipeHow:\Blog> [Convert]::ToInt32('0x4e729', 16)
321321
```

There are also a few other ways that we can convert to and from hexadecimal using [string formatting](https://docs.microsoft.com/en-us/dotnet/standard/base-types/standard-numeric-format-strings).

```ps1
# From Decimal to Hexadecimal
PS PipeHow:\Blog> '{0:x}' -f 321321
4e729
PS PipeHow:\Blog> (321321).ToString('x')
4e729
PS PipeHow:\Blog> [string]::Format('{0:x}', 321321)
4e729
```

You can also use Format-Hex, but do so at your own risk.

```ps1
# Exploring Format-Hex
PS PipeHow:\Blog> $Hex = Format-Hex -InputObject 321321
PS PipeHow:\Blog> $Hex

           Path:

           00 01 02 03 04 05 06 07 08 09 0A 0B 0C 0D 0E 0F

00000000   29 E7 04 00                                      )ç..

# You can get the number as a byte array
PS PipeHow:\Blog> $Hex.Bytes
41
231
4
0

# The string seems to be our only real way to extract the hexadecimal number
PS PipeHow:\Blog> $Hex.ToString()
00000000   29 E7 04 00                                      )ç..

# Parsing Hexadecimal number from string
# Get index of first space to filter out zeroes
PS PipeHow:\Blog> $Index = $Hex.ToString().IndexOf(' ')
# Select only the hexadecimal characters and spaces until the next value
PS PipeHow:\Blog> $HexByteString = $Hex.ToString().Substring($Index).TrimStart(' ').Substring(0, 47).TrimEnd(' ')
PS PipeHow:\Blog> $HexByteString
29 E7 04 00
# Split the string, reverse the array and join it back into a string
PS PipeHow:\Blog> $HexByteArray = $HexByteString.Split(' ')
PS PipeHow:\Blog> $HexValue = $HexByteArray[($HexByteArray.Count)..0] -join ''
PS PipeHow:\Blog> $HexValue
0004E729
# Confirm the value
PS PipeHow:\Blog> [Convert]::ToInt32($HexValue, 16)
321321
```

Format-Hex is as you can see not the most convenient command to use when simply trying to convert a number since it's in reverse byte order and you have to parse it from the string, its purpose is better suited to converting file names or strings.

## Octal
If you need to work with base 8 you can do it in the same way with the Convert class, converting to an int to octal and a string back to decimal.

```ps1
# From Decimal to Octal
PS PipeHow:\Blog> [Convert]::ToInt32(707, 8)
455
# From Octal to Decimal
PS PipeHow:\Blog> [Convert]::ToString(455, 8)
707
```

## Bytes (Base 256)
While maybe not considered a numeral system, you can convert any number to bytes using the ```[System.Bitconverter]```.

```ps1
PS PipeHow:\Blog> [BitConverter]::GetBytes(321321)
41
231
4
0
```

Recognize that line of numbers? It's the same one we got from the ```Bytes``` property of the result from ```Format-Hex```. When you keep converting values back and forth you eventually notice how it all fits together!

## Base 2-36
If you haven't had enough of the numeral bases available through .NET you can convert between any arbitrary numeral base you want using a couple of functions I adapted from [ss64](https://ss64.com/ps/syntax-base36.html) with support for conversion between any numeral base between 2 and 32. You can add more bases by either mixing upper- and lowercase letters to get more symbols or by definting a system or a standard for your use context that sets what symbols to use when you run out of our alphabetical letters.

```ps1
function ConvertTo-NumeralBase
{
    <#
    .SYNOPSIS
        Converts a number to any numeral base between 2 and 36.
    
    .DESCRIPTION
        Converts a number to any numeral base between 2 and 36.

    .EXAMPLE
        PS PipeHow:\Blog> ConvertTo-NumeralBase -Number 300 -Base 16
        12C

        Converts the number 300 to hexadecimal (base 16).
    
    .EXAMPLE
        PS PipeHow:\Blog> ConvertTo-NumeralBase -Number 210 -Base 12
        156

        Converts the number 210 to base 12.
    #>
    [CmdletBinding()]
    [OutputType([string])]
    param(
        [Parameter(
            Position = 0,
            Mandatory,
            ValueFromPipelineByPropertyName)]
        [ValidateNotNullOrEmpty()]
        [int]$Number,
        [Parameter(
            Position = 1,
            Mandatory,
            ValueFromPipelineByPropertyName)]
        [ValidateNotNullOrEmpty()]
        [ValidateRange(2,36)]
        [int]$Base
    )
    
    $Characters = "0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ"

    while ($Number -gt 0)
    {
        $Remainder = $Number % $Base
        $CurrentCharacter = $Characters[$Remainder]
        $ResultNumber = "$CurrentCharacter$ResultNumber"
        $Number = ($Number - $Remainder) / $Base
    }

    $ResultNumber
}

function ConvertFrom-NumeralBase
{
    <#
    .SYNOPSIS
        Converts a number from any numeral base between 2 and 36.
    
    .DESCRIPTION
        Converts a number from any numeral base between 2 and 36.

    .EXAMPLE
        PS PipeHow:\Blog> ConvertFrom-NumeralBase -Number 12C -Base 16
        300

        Converts the number 12C from hexadecimal (base 16).
    
    .EXAMPLE
        PS PipeHow:\Blog> ConvertFrom-NumeralBase -Number 156 -Base 12
        210

        Converts the number 156 from base 12.
    #>
    [CmdletBinding()]
    [OutputType([string])]
    param(
        [Parameter(
            Position = 0,
            Mandatory,
            ValueFromPipelineByPropertyName)]
        [ValidateNotNullOrEmpty()]
        [string]$Number,
        [Parameter(
            Position = 1,
            Mandatory,
            ValueFromPipelineByPropertyName)]
        [ValidateNotNullOrEmpty()]
        [ValidateRange(2,36)]
        [int]$Base
    )
    
    $Characters = "0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ"

    $NumberCharArray = $Number.ToUpper().ToCharArray()
    $NumberCharArray = $NumberCharArray[($NumberCharArray.Count)..0]

    [long]$DecimalNumber = 0
    $Position = 0

    foreach ($Character in $NumberCharArray)
    {
        $DecimalNumber += $Characters.IndexOf($Character) * [long][Math]::Pow($Base, $Position)
        $Position++
    }

    $DecimalNumber
}
```

I hope you got some value out of it all, it's always good to know how to manage data in different forms!