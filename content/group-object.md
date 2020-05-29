---
title: "Exploring Group-Object in PowerShell"
date: 2020-05-29T10:28:00+02:00
summary: "No more nested loops to create different arrays based on whether users have a certain property or not. Combine Group-Object with Where-Object and experience a whole new level of data structuring."
description: "Organizing and structuring data using Where-Object and Group-Object is an unbeatable combination in terms of both flexibility and code clarity."
keywords:
- Blog
- PowerShell
- Where-Object
- Group-Object
- Group
- Where
- Object
- In Depth
- Complex
- Criteria
- Structured
- Data
draft: true
---

At some point in our ever-expanding PowerShell learning curve we've probably written something along these lines:

```PowerShell
$UsersWithX = @()
$UsersWithoutX = @()
foreach ($User in $Users) {
    if ($User.PropertyX -eq 'VerySpecificValue') {
        $UsersWithX += $User
    }
    else {
        $UsersWithoutX += $User
    }
}
```

It could be that we just retrieved a lot of AD users and we're sorting them into lists based on whether they're member of a specific AD group, or maybe we're looking at users from an on-premises system to check who has a certain role.

You may be thinking *"That could use some `Where-Object`!"*, but even then we would end up in a situation where we make the same comparison twice. Let's use services as an example.

```PowerShell
$Services = Get-Service
$RunningServices = $Services | Where-Object Status -eq 'Running'
$StoppedServices = $Services | Where-Object Status -ne 'Running'
```

So while we definitely improved readability, you could argue that the code is less optimal as it has to evaluate the entire list of users twice. Do we have any alternatives?

## Group-Object

PowerShell has an underestimated cmdlet called `Group-Object` which lets us solve the problem in a single command instead.

$GroupedUsers = $Users | Group-Object PropertyX

---

PS7 has case sensitive hashtables as an option with -ashashtable
txt and TXT can be different keys