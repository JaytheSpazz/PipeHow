---
title: "Printing Documents using PowerShell"
date: 2019-09-12T21:21:00+02:00
summary: "How does one go about printing a ton of documents based on data from a source such as a database, REST API or user input? Let's take a look at how to print documents and graphics using PowerShell!"
description: "Printing documents and images using PowerShell and .NET!"
keywords:
- Blog
- PowerShell
- .NET
- Printing
- Print
- Printer
- Document
- Image
- PDF
- dotnet
draft: false
---

Printing is an interesting problem that you don't often encounter in your daily work when scripting or programming, at least that has been the case for me. No one ever asked me what I would do if I was suddenly required to print a document for every user in my database, or monitor every time someone logged into a specific server and print their username on the closest printer. Both sound like dumb ideas, but knowledge is power!

With great power comes great responsibility, however, so I'd advice you to avoid ruining the poor trees for no reason!

Not too long ago I got tasked with the challenge of assembling a picture in PowerShell of a user's badge access card and sending it to a card printer as a part of a bigger automation implementation, which I gladly jumped on.

## Printing from PowerShell

There are a few built-in commands that have something to do with printing, as we can see by running `Get-Command` and filtering on things that contain "Print".

```PowerShell
PS PipeHow:\Blog> Get-Command *Print*
```

There are quite a few results, most of them are functions in PowerShell but there are also some actual executable programs that come with windows. After looking through the list I realized that there weren't a lot of functions that could help me out, aside from potentially `Out-Printer` which I decided to check out more closely.

```PowerShell
PS PipeHow:\Blog> Get-Help Out-Printer -Examples
```

`Out-Printer` only has a few parameters and none of them seem to let me customize very much. You can specify which printer you wish to use, but other than that it only takes an input object and has some [common parameters](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_commonparameters). Browsing the help documentation for the parameters and examples made me realize that PowerShell might not have quite enough out of the box for what I needed, which sent me on a journey on the internet to dig deeper.

## Printing in .NET

Enter the `[System.Drawing.Printing]` [namespace](https://docs.microsoft.com/en-us/dotnet/api/system.drawing.printing), which "provides print-related services for Windows Forms applications". The general consensus seems to be that it's the most painless way to implement native, customizable printing in C# and PowerShell.

I needed to print a picture onto a card, using a specified card printer that we installed as a network printer and shared to the server managing the automation. The picture on the card I was to print was in the form of a byte array and I needed to place it on top of another template picture together with the person's first and last name. I needed to figure out the size of the document and the pixel positioning of the image parts, then how to draw things on top of each other to assemble the full image and then send it on its way.

I had printed a card from the actual badge system to a PDF to have as comparison, so my goal was to print to PDF from PowerShell and then try to match size and picture positioning.

The first thing to do when working with the printing classes is to tell PowerShell that we want to import the `[System.Drawing]` namespace, where our printing functionality resides. We also need to create a [PrintDocument](https://docs.microsoft.com/en-us/dotnet/api/system.drawing.printing.printdocument) to configure for printing and eventually draw our images onto.

```PowerShell
PS PipeHow:\Blog> Add-Type -AssemblyName System.Drawing 
PS PipeHow:\Blog> $PrintDocument = New-Object System.Drawing.Printing.PrintDocument
```

If we look at the properties of our newly created print document we will see the different settings we can configure.

```PowerShell
PS PipeHow:\Blog> $PrintDocument | Get-Member

   TypeName: System.Drawing.Printing.PrintDocument
Name                      MemberType     Definition
----                      ----------     ----------
BeginPrint                Event          System.Drawing.Printing.PrintEventHandler BeginPrint(System.Object, System.Drawing.Printing.PrintEventArgs)
Disposed                  Event          System.EventHandler Disposed(System.Object, System.EventArgs)
EndPrint                  Event          System.Drawing.Printing.PrintEventHandler EndPrint(System.Object, System.Drawing.Printing.PrintEventArgs)
PrintPage                 Event          System.Drawing.Printing.PrintPageEventHandler PrintPage(System.Object, System.Drawing.Printing.PrintPageEventArgs)
QueryPageSettings         Event          System.Drawing.Printing.QueryPageSettingsEventHandler QueryPageSettings(System.Object, System.Drawing.Printing.QueryPageSettingsEventArgs)
Dispose                   Method         void Dispose(), void IDisposable.Dispose()
Equals                    Method         bool Equals(System.Object obj)
GetHashCode               Method         int GetHashCode()
GetLifetimeService        Method         System.Object GetLifetimeService()
GetType                   Method         type GetType()
InitializeLifetimeService Method         System.Object InitializeLifetimeService()
Print                     Method         void Print()
ToString                  Method         string ToString()
Container                 Property       System.ComponentModel.IContainer Container {get;}
DefaultPageSettings       Property       System.Drawing.Printing.PageSettings DefaultPageSettings {get;set;}
DocumentName              Property       string DocumentName {get;set;}
OriginAtMargins           Property       bool OriginAtMargins {get;set;}
PrintController           Property       System.Drawing.Printing.PrintController PrintController {get;set;}
PrinterSettings           Property       System.Drawing.Printing.PrinterSettings PrinterSettings {get;set;}
Site                      Property       System.ComponentModel.ISite Site {get;set;}
Color                     ScriptProperty System.Object Color {get=$this.PrinterSettings.SupportsColor;}
Duplex                    ScriptProperty System.Object Duplex {get=$this.PrinterSettings.Duplex;}
Name                      ScriptProperty System.Object Name {get=$this.PrinterSettings.PrinterName;}
```

If you look closely, and especially if you explore further with `Get-Member` on the properties `PrinterSettings` and `DefaultPageSettings`, you can tell that this is actually very much the same settings as in the menu that you often access when printing a document through normal applications such as text editors. Disregarding the methods and events for now, there are a few properties that we want to set for the card to printed correctly, namely the printer, document name, size and orientation. The document name is what will show up in the printer queue and anything that logs your print jobs.

As the badge card that I was setting out to recreate and print was in a landscape orientation, I decided to set the property `DefaultPageSettings.Landscape` to `true` and to use the paper size "Letter". I know from previous experience that it's sometimes used as a setting for label printers and thought that it could be the case here as well since a badge card is in a similar shape.

To find the different sizes you can look through the `PrinterSettings.PaperSizes`, from where you can choose one and set to your `DefaultPageSettings.PaperSize`.

```PowerShell
PS PipeHow:\Blog> $PrintDocument.PrinterSettings.PrinterName = 'Microsoft Print to PDF'
PS PipeHow:\Blog> $PrintDocument.DocumentName = "PipeHow Print Job"
PS PipeHow:\Blog> $PrintDocument.DefaultPageSettings.PaperSize = $PrintDocument.PrinterSettings.PaperSizes | Where-Object { $_.PaperName -eq 'Letter' }
PS PipeHow:\Blog> $PrintDocument.DefaultPageSettings.Landscape = $true
```

There weren't any more settings from the print document that I ended up using among the ones we listed, but there are a few more things that we will need from the list, specifically the events.

```PowerShell
PS PipeHow:\Blog> $PrintDocument | Get-Member -MemberType Event

   TypeName: System.Drawing.Printing.PrintDocument
Name              MemberType Definition
----              ---------- ----------
BeginPrint        Event      System.Drawing.Printing.PrintEventHandler BeginPrint(System.Object, System.Drawing.Printing.PrintEventArgs)
Disposed          Event      System.EventHandler Disposed(System.Object, System.EventArgs)
EndPrint          Event      System.Drawing.Printing.PrintEventHandler EndPrint(System.Object, System.Drawing.Printing.PrintEventArgs)
PrintPage         Event      System.Drawing.Printing.PrintPageEventHandler PrintPage(System.Object, System.Drawing.Printing.PrintPageEventArgs)
QueryPageSettings Event      System.Drawing.Printing.QueryPageSettingsEventHandler QueryPageSettings(System.Object, System.Drawing.Printing.QueryPageSettingsEventArgs)
```

In PowerShell you can bind traditional .NET events to objects using scriptblocks or functions by prepending `add_` before the event name, in our case we would call `$PrintDocument.add_PrintPage({...})` where the dots represent our code to run when the page is to be printed. We will need to define the event code before calling the method, so that it knows what to run. There are also a few other events that are good to know about: `BeginPrint` which is best used for initializing resources such as fonts or file streams, and `EndPrint` for the opposite - deallocating or cleaning up anything that was used during the printing.

We will need a picture to print, so I will import an example picture as a byte array using a method from the `[System.IO.File]` namespace.

```PowerShell
PS PipeHow:\Blog> $PictureByteArray = [System.IO.File]::ReadAllBytes($Path)
```

In my task I had a second picture which was a template to place the picture on which required some hard coded pixel values for position and size, but the process is the same for drawing a single image onto the print document. For our example I'll draw a rectangle as a background instead. We only print one page at a time but if you need to print documents with several pages you may need to implement code that handles that, for example checking the [HasMorePages](https://docs.microsoft.com/en-us/dotnet/api/system.drawing.printing.printpageeventargs.hasmorepages) property of the event data.

You'll see me mixing the ways I create objects in PowerShell between using `New-Object` and the `[Class]::new()` constructor. Both have the same functionality but sometimes when I pass new objects as parameters I like to use the .NET version to avoid needing extra parentheses around the `New-Object` calls.

```PowerShell
$PrintDocument.add_PrintPage({
    # Create an ImageConverter to convert our byte array into a Bitmap for drawing
    $ImageConverter = New-Object System.Drawing.ImageConverter
    [Drawing.Bitmap]$ProfileImg = $Imageconverter.ConvertFrom($PictureByteArray)
    
    # Create font and colors for text and background
    $Font = [System.Drawing.Font]::new('Arial', 12, [System.Drawing.FontStyle]::Bold)
    $BrushFG = [System.Drawing.SolidBrush]::new([System.Drawing.Color]::FromArgb(255,0,0,0))
    $BrushBG = [System.Drawing.SolidBrush]::new([System.Drawing.Color]::FromArgb(255,130,180,120))

    # Set an arbitrary width to scale the image to
    $Width = 100

    # Calculate the relationship between height and width to calculate with new width
    $ProfileScale = [double]$ProfileImg.Height / [double]$ProfileImg.Width 

    # Create source rectangle representing full size of image
    $ProfileRectSrc = New-Object Drawing.RectangleF(0, 0, $ProfileImg.Width, $ProfileImg.Height)
    # Change scale of destination rectangle based on the calculated width
    $ProfileRectDest = New-Object Drawing.RectangleF([System.Drawing.PointF]::new(10,10), [System.Drawing.Size]::new($Width, [int]($Width * $ProfileScale)))

    # Draw background rectangle and picture onto the graphics of the page passed as $_ in the PrintPage event
    # The order you draw to the page is important, with the background being first
    $_.Graphics.FillRectangle($BrushBG, (New-Object Drawing.RectangleF([System.Drawing.PointF]::new(0,0), [System.Drawing.Size]::new($Width * 2, [int]($Width * 1.2 * $ProfileScale)))))
    $_.Graphics.DrawImage($ProfileImg, $ProfileRectDest, $ProfileRectSrc, [Drawing.GraphicsUnit]::Pixel) 

    # Write first and last name
    $FirstName = 'Emanuel'
    $LastName = 'Palm'

    # Draw text to the right of the image
    $_.Graphics.DrawString($FirstName,$Font,$BrushFG,($Width + 20),20) 
    $_.Graphics.DrawString($LastName,$Font,$BrushFG,($Width + 20),40)
})
```

After defining the code to be run when we print our document, let's try it!

```PowerShell
PS PipeHow:\Blog> $PrintDocument.Print()
```

Since we set the printer to "Microsoft Print to PDF" we get a dialog box asking us to specify a location to save the file. Doing that gives us a fairly empty PDF where the top left corner contains the image we read from together with our background color and the specified name to the right of it. Printing to a PDF is a reasonable way to see how your output will look, especially if you're trying to match your output to an existing template of some sort.

![The Printed PDF](/img/invoke-print/print_output.png)

There are a lot of fun things you can do with automated printing that you may not normally think about, let me know if you create something cool with it!