---
title: "Web Scraping with PowerShell"
date: 2019-08-19T22:05:00+02:00
summary: "If you've ever been in a situation where you would like to index a lot of pages on a webpage or get the data straight from the HTML because the site doesn't have an API, PowerShell could have been of help. If you haven't been in the situation yet, check out these simple tricks you can use to gather raw data straight from the web!"
description: "Scraping web pages using PowerShell!"
keywords:
- Blog
- PowerShell
- Web
- Scraping
- Webscraping
- Regex
- RegExp
draft: true
---

Sometimes you end up in situations where you want to get information from an online source such as a webpage, but the service has no API available for you to get information through and it's too much data to manually copy and paste. Or maybe you need to register a lot of entries on a website, but don't have a bored friend to help out. Fear not, PowerShell can be your bored friend if you gently ask it in the right way!



Sites as examples
http://toscrape.com/
http://books.toscrape.com/
http://testing-ground.scraping.pro/login

More advanced site with buttons and continous session?

Invoke-WebRequest (talk about Invoke-RestMethod too for APIs), use Regex to find what you need from the raw HTML (.Content)
Talk about .NET "web requests" in comparison and different encodings

UseBasicParsing is deprecated from PowerShell 6 and onwards, but used to automatically parse the HTML into objects.