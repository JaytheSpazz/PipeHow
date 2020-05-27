---
title: "Azure Table Storage with PowerShell"
date: 2019-06-30T16:35:00+02:00
summary: "Azure Table is a fantastic way to store output information from scripts in PowerShell. In this post we'll go through what steps you take when you need to quickly store structured information in the cloud and how to work with it using a SAS token!"
description: "Storing structured data in Azure Table using PowerShell without logging in!"
keywords:
- Blog
- PowerShell
- Azure
- Storage
- Table
- SAS
- Structured
- Data
draft: false
---

Sometimes you're in a spot where you really need a place to store structured information but you don't have or want a database that you can use for the task. It could be that you run a script and want to save the result of it for reporting, maybe you need detailed logging for security reasons but can't write to any local source because of disk failure reasons. Using an [Azure Table Storage](https://azure.microsoft.com/en-in/services/storage/tables/) is an incredibly convenient, lightweight way to save the result of a script, especially if it runs on many devices. You could use a GPO or deployment of some kind to send out a script but it's often hard to get the script to report back to a common source without an external solution. Something like a configuration baseline can work to see custom compliance if you work with [System Center Configuration Manager](https://www.microsoft.com/en-us/cloud-platform/system-center-configuration-manager), but sometimes you need more detail and better reporting.

Today we'll take a look at how we create an Azure Table Storage, and more importantly how to work with one from a PowerShell session that is not logged into the Azure account using a SAS token.

## Creating the Table

We can create the storage account using either the portal or PowerShell. Note that we will not be able to create the storage account and table using the same PowerShell session running the script that we're later creating. This is because the creation of the table requires us to be able to access, create and manage a storage account in the subscription while the situation of our hypothetical script use case only has a SAS token and no azure access.

I will assume you have navigated the portal before and created a storage account, from there you can simply scroll down, add a table and generate a SAS token. If you only want to see how to actually work with the table using PowerShell you can head directly to the **Using Azure Table** section. Let's take a look at creating it using PowerShell instead and to do that we'll need a few Azure modules. You can download them individually but for simplicity I install the ```Az``` module which includes most of the whole suite, then we also need the module ```AzTable```.

```PowerShell
PS PipeHow:\Blog> Install-Module Az
PS PipeHow:\Blog> Install-Module AzTable
```

We now have a lot of commands that we can run but before we start creating the storage we need to connect to our Azure account using ```Connect-AzAccount```. Running it without parameters will open a window where you can login. I'm only doing this to set up the storage account, later when working with the storage account we will pretend we don't have an Azure account with access.

Once logged in we're mostly interested in commands in the module called ```Az.Storage```.

```PowerShell
PS PipeHow:\Blog> Get-Command *Table* -Module Az.Storage

CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Cmdlet          Get-AzStorageTable                                 1.3.0      Az.Storage
Cmdlet          Get-AzStorageTableStoredAccessPolicy               1.3.0      Az.Storage
Cmdlet          New-AzStorageTable                                 1.3.0      Az.Storage
Cmdlet          New-AzStorageTableSASToken                         1.3.0      Az.Storage
Cmdlet          New-AzStorageTableStoredAccessPolicy               1.3.0      Az.Storage
Cmdlet          Remove-AzStorageTable                              1.3.0      Az.Storage
Cmdlet          Remove-AzStorageTableStoredAccessPolicy            1.3.0      Az.Storage
Cmdlet          Set-AzStorageTableStoredAccessPolicy               1.3.0      Az.Storage
```

We'll need a few others, but ```New-AzStorageTable``` seems like a good place to start. We now need to figure out what kind of information we need to use it. 

```PowerShell
PS PipeHow:\Blog> Get-Help New-AzStorageTable -Examples
```

We have a couple of simple examples, but if you try running one there will be an error complaning about storage context. This is because we first need a storage account to use as context for the new table. You can create a context simply by referencing a storage account's context property, but I'll demonstrate how to create one using one of the access keys of the storage account.

```PowerShell
PS PipeHow:\Blog> New-AzResourceGroup -Name 'TableRG' -Location 'westeurope'
PS PipeHow:\Blog> New-AzStorageAccount -ResourceGroupName 'TableRG' -Name 'pipehowtablestorage' -SkuName Standard_LRS -Location 'westeurope' -Kind StorageV2
PS PipeHow:\Blog> $Key = (Get-AzStorageAccountKey -ResourceGroupName 'TableRG' -Name 'pipehowtablestorage')[0].Value
PS PipeHow:\Blog> $Context = New-AzStorageContext -StorageAccountName 'pipehowtablestorage' -StorageAccountKey $Key
```

We did a few things here:

* Created a resource group to hold our storage account
* Created a storage account to hold our table
* Retrieved a storage account key to make sure our context has full access
* Created a storage context based on the access key from the storage account

We created the resources in west europe, currently the closest region to where I live, in Sweden. We created the storage account with local redundancy which is the cheapest and quickest option, and as a general-purpose V2 kind, the newest option among the two that support table storage. If you want to dive into the different options of a storage account I'd recommend reading through [Microsoft's overview](https://docs.microsoft.com/en-us/azure/storage/common/storage-account-overview). We could have created our context without using our account key, but then our context would not have had access to create a table in our storage account. The keys give us full access, which also makes it important to keep them a secret!

Now that we have our storage context we can create our table!

```PowerShell
PS PipeHow:\Blog> New-AzStorageTable -Name 'PipeHowTable' -Context $Context
```

## Creating a SAS token

Before we move onto the exercise of writing to our table we want to make sure that we have prepared for our scenario, which means we will need a Shared Access Signature token. We could of course create this through the portal as well, but we'll do it through PowerShell to learn something.

This is a token based on one of our access keys, such as the one we used to create our context. A SAS token can be seen as an access key, but when creating it we specify how long it will be valid and what type of access it will grant. Since it is based on one of the access keys for the account we can also invalidate the SAS token whenever we want by refreshing the key it was based on, but be aware that this will invalidate all SAS tokens based on that key.

```PowerShell
PS PipeHow:\Blog> $SAS = New-AzStorageAccountSASToken -Service Table -ResourceType Container,Service,Object -StartTime (Get-Date) -ExpiryTime (Get-Date).AddHours(6) -Context $Context -Permission 'rwdlau'
```

We create a SAS token for the storage account, the validity period is 6 hours and it is based on the context we created earlier, meaning it uses Key1 as the access key that it's based on. We specify permissions defined as a string where each letter is a specified permission, in this case it has full access to permissions linked to the table service. All permissions are not valid for all resource types, Microsoft says [the following](https://docs.microsoft.com/en-us/rest/api/storageservices/constructing-an-account-sas) about the permissions:

- **Read \(r)**: Valid for all signed resources types (Service, Container, and Object). Permits read permissions to the specified resource type.
- **Write \(w)**: Valid for all signed resources types (Service, Container, and Object). Permits write permissions to the specified resource type.
- **Delete \(d)**: Valid for Container and Object resource types, except for queue messages.
- **List \(l)**: Valid for Service and Container resource types only.
- **Add \(a)**: Valid for the following Object resource types only: queue messages, table entities, and append blobs.
- **Create \(c)**: Valid for the following Object resource types only: blobs and files. Users can create new blobs or files, but may not overwrite existing blobs or files.
- **Update \(u)**: Valid for the following Object resource types only: queue messages and table entities.
- **Process \(p)**: Valid for the following Object resource type only: queue messages.

## Using Azure Table Storage

Using only our SAS key we can now work with our table without being connected to Azure. If we sent this key to someone else, they could now do whatever they wanted in the table for 6 hours. With this in mind it's a good idea to store it securely.

Let's pretend we're someone else who doesn't have access, to do this we can disconnect our Azure account and remove the context we retrieved using one of the access keys.

```PowerShell
PS PipeHow:\Blog> Disconnect-AzAccount
PS PipeHow:\Blog> Remove-Variable Context
```

We can then create a new context using our SAS token, and then get the table that we will be working with.

```PowerShell
PS PipeHow:\Blog> $Context = New-AzStorageContext -StorageAccountName 'pipehowtablestorage' -SasToken $SAS
PS PipeHow:\Blog> $Table = Get-AzStorageTable -Context $Context
```

Working with the table is now as easy as calling a simple command. Let's try adding some information, note that we need to use the CloudTable property when performing operations on the table.

```PowerShell
PS PipeHow:\Blog> Add-AzTableRow -Table $Table.CloudTable -PartitionKey 'Result' -RowKey ((New-Guid).Guid) -property @{ Result = 'Success'; Source = 'PipeHow' }
PS PipeHow:\Blog> Add-AzTableRow -Table $Table.CloudTable -PartitionKey 'Result' -RowKey ((New-Guid).Guid) -property @{ Result = 'Failure'; Source = 'PipeHow' }
```

When creating an entry, or a row, in a table in Azure we need to specify a unique row key, the most convenient way is to create a new guid to make sure it's unique. You can see it as an identifier for the entry. All information you want to write to the table needs to be in the hashtable you send with the ```property``` parameter.

We can now go into the Storage Explorer of our storage account and look at our table rows.

![Storage Explorer of Table](/img/write-azuretable/azure_storageexplorer_table.png)

We can of course also retrieve the data using PowerShell.

```PowerShell
PS PipeHow:\Blog> Get-AzTableRow -Table $Table.CloudTable

Result         : Success
Source         : PipeHow
PartitionKey   : Result
RowKey         : 2aba23cd-9341-4f69-a8f4-714df5c2dac5
TableTimestamp : 2019-06-30 14:46:02 +02:00
Etag           : W/"datetime'2019-06-30T12%3A46%3A02.9309598Z'"

Result         : Failure
Source         : PipeHow
PartitionKey   : Result
RowKey         : efcc92ba-23c5-42c4-ba52-4272d447d4fd
TableTimestamp : 2019-06-30 14:46:08 +02:00
Etag           : W/"datetime'2019-06-30T12%3A46%3A08.3207654Z'"
```

We can also create filters, either using methods from classes the Table namespace in Azure or by simply writing the string yourself. The two lines below create the same filter.

```PowerShell
PS PipeHow:\Blog> $Filter = [Microsoft.Azure.Cosmos.Table.TableQuery]::GenerateFilterCondition("Result",[Microsoft.Azure.Cosmos.Table.QueryComparisons]::Equal,"Success")
PS PipeHow:\Blog> $Filter = "Result eq 'Success'"
```

If we then want to remove all entries that were successful because we're only interested in the ones that failed we can do so by using ```Remove-AzTableRow```. The ```Etag``` property is required when removing the table row, so piping the results from ```Get-AzTableRow``` is the simplest.

```PowerShell
PS PipeHow:\Blog> Get-AzTableRow -Table $Table.CloudTable -CustomFilter $Filter | Remove-AzTableRow -Table $Table.CloudTable
```

Quick and simple, Azure Table Storage is a great way to throw up information quickly and securely to a central logging point. There are a few different commands you can use when working with Azure tables in PowerShell so if you want more information on the subject you can have a look at [Microsoft's documention](https://docs.microsoft.com/bs-latn-ba/azure/storage/tables/table-storage-how-to-use-powershell).