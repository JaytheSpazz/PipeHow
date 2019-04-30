---
title: "Choosing Lunch through PowerShell"
date: 2018-10-23T12:18:11+02:00
summary: "In the start of the past summer, a coworker and I were discussing our lunch plans for the day, not quite able to decide what to go for. I jokingly said that I should just write a PowerShell script to decide for us, an idea that I took to the next level."
---

If you're not interested in the how or the why, skip ahead to the [project on GitHub](https://github.com/MrEpiX/Get-Lunch).

## Begin
In the start of the past summer, a coworker and I were discussing our lunch plans for the day, not quite able to decide what to go for. I jokingly said that I should just write a PowerShell script to decide for us, which I then actually decided to implement a quick version of.

I started listing restaurants in an array...

```ps1
$Restaurants = @(
"Decent Burger Place",
"Close-by Buffé",
"Pasta Take-Away")
```
While typing out what I could think of I asked my coworker, over my shoulder, what more places to add. He responded with a thought that it would be cool to instead of listing them manually, finding them through what’s closest on Google.

That’s possible, I thought; I *can* make that.

## Process
On [Google’s Cloud Platform](https://console.cloud.google.com/apis/) you can sign up, create a free project and get an API key (it looks like “abCdeFghIjkLmnOpqRstUvxYzaB-cdEfgHijKlm”) with a bunch of different APIs enabled. After looking around I found the [Places API](https://developers.google.com/places/web-service/intro) which seemed to be what I was looking for. The catch is that the request requires the location to search from to be in the format of “latitude,longitude”. Luckily, there’s [another API](https://developers.google.com/maps/documentation/geocoding/start) for that!

Using the API is generally done in JSON, which PowerShell manages very well through ConvertFrom-Json and ConvertTo-Json. A quick example of how to get latitude and longitude through the API using Invoke-WebRequest:

```ps1
$GeoResult = Invoke-WebRequest -Method Get -ContentType "application/json" -Uri "$URL/geocode/json?address=$Address&key=$APIKey" | ConvertFrom-Json
$GeoResult.results.geometry.location.lat
$GeoResult.results.geometry.location.lng
```

There are a few ways of finding nearby restaurants, either get restaurants within a certain radius, or the closest ones from a location. I found that using the parameter `` ```rankby=distance` and `type=restaurant` would give me the 20 closest, which was a good start for my idea.

```ps1
$NearbyResult = Invoke-WebRequest -Method Get -ContentType "application/json" -Uri "$URL/place/nearbysearch/json?oe=utf-8&language=sv&location=$($Latitude),$($Longitude)&type=restaurant&rankby=distance&key=$APIKey" | ConvertFrom-Json
```

I found that the result of the request also includes something called a page token to use for repeated requests for more restaurants. You can therefore do another request for 20 more restaurants twice, for a total of 60.

I implemented a while loop that sends requests until the token is null, with a small delay because of the following information from Google’s API documentation:

*There is a short delay between when a next_page_token is issued, and when it will become valid. Requesting the next page before it is available will return an INVALID_REQUEST response.*

```ps1
$Token = $NearbyResult.next_page_token

# loop through the next_page_tokens to get 40 more restaurants, you can only get 3 "pages"
$Results = $NearbyResult.results
while ($Token)
{
    # A delay is needed for the next page token to exist on the response
    Start-Sleep -Seconds 5
    $NearbyResult = Invoke-WebRequest -Method Get -ContentType "application/json" -Uri "$URL/place/nearbysearch/json?pagetoken=$Token&key=$APIKey" | ConvertFrom-Json
    $Results += $NearbyResult.results
    $Token = $NearbyResult.next_page_token
}
```

“That’s it!”, I thought after populating a list of restaurants, adding filtering on restaurants currently open, sorting it randomly and selecting a restaurant.

```ps1
$Restaurant = $Results | Where-Object { $_.opening_hours.open_now -eq $true } | Select-Object -ExpandProperty name -Unique | Sort-Object { Get-Random } | Select-Object -First 1
```

I got my result, but I wasn’t quite happy. It needed polish, and I needed it to be available to my coworkers so that I wasn’t the only one to have to eat something I hadn’t chosen.

I implemented support to exclude restaurants that we really didn’t like and decided to make it send an email, adding a [Straw Poll](https://www.strawpoll.me/) to the email through [their API](https://github.com/strawpoll/strawpoll/wiki/API) for ease of access. The code was growing, so I split the code into three files for better structure:

* Get-RandomRestaurant.ps1 takes some parameters such as amount of restaurants to get, where to search from and if any restaurants should be blacklisted, or excluded. The script returns a list of restaurants with information about them from Google’s Places API: name, website, rating but also distance and time to walk there through the [Distance Matrix API](https://developers.google.com/maps/documentation/distance-matrix/start).
* Send-LunchEmail.ps1 takes parameters with information about who to mail it to, all of the restaurants returned from Get-RandomRestaurant.ps1 and whether or not to include a poll.
* Save-LunchToFile.ps1 to simply to output a HTML file instead of the email. A good idea for refactoring would be to put these into the same script with some more switches.

## End
After shamelessly stealing pictures and HTML layout online, I managed to scratch together something that was actually quite nice to look at. Manipulating strings with HTML in PowerShell is surprisingly not terrible when using multiline so-called [here-strings](https://blogs.technet.microsoft.com/heyscriptingguy/2015/12/31/powertip-use-here-strings-with-powershell/).

Here is some code I add to the body through a loop that goes through each restaurant provided to Send-LunchEmail.ps1:

```ps1
$InfoHTML = @"
<td align="center" valign="middle" width="50%"><a href="$($Item.Website)"><span style="color: #$($TextColor)">$($Item.Name) ★$($Item.Rating)★</span></a></td>
<td align="center" valign="middle" width="50%"><span style="color: #$($TextColor); text-decoration-line: underline;">$($Item.Time) ($($Item.Distance))</span></td>
"@
```

PowerShell’s Send-MailMessage didn’t seem to like embedding images in the HTML body, so I found [this adaption of Send-MailMessage](https://gallery.technet.microsoft.com/scriptcenter/Send-MailMessage-3a920a6d) which solved that problem.

I also didn’t want to get tired of the look, so I added random colors with an inverted color for the button with the poll link. That’s for an article of its own, though!

I’d like to say that the final product can be seen below, but there’s still a bunch of improvements I’d like to attempt. Check it out [on GitHub](https://github.com/MrEpiX/Get-Lunch) where you can clone the repository and play around with it yourself!

![Lunch Email](https://raw.githubusercontent.com/MrEpiX/Get-Lunch/master/Resources/ExampleImage.png "Lunch Email")