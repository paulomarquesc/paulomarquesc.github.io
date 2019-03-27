---
layout: post
title: Working with Azure Tables from PowerShell - AzureRmStorageTable PS Module v2.0
excerpt_separator: <!--more-->
comments: true
---
{% assign postPath = page.path | remove: '%0A' | remove: '_posts' | remove: '.md' %}

In this blog post I’m going to show how to work with Azure Storage Tables from PowerShell, through a sample PowerShell module that I created for operations like Add, Retrieve, Update and Delete table rows/entities exposed as functions. Now supporting the new Az PowerShell module plus the SDK change from Microsoft.WindowsAzure.Storage to Microsoft.Azure.Cosmos assembly.

<!--more-->

# In this blog post

Topics covered in this blog post:

* [Updates](#updates)
* [Introduction](#intro)
* [Requirements](#requirements)
* [Installation/Source Code](#install)
* [Using Azure Storage Table PowerShell Module](#psstoragetable)
  * [Adding Rows/Entities](#adding)
  * [Retrieving Rows/Entities](#retrieving)
  * [Updating an entity](#updating)
  * [Deleting rows/entities](#deleting)
* [References](#references)

## Updates<a name="updates"></a>

<span style="color:red">**Update 03/26/2019**</span>: Updates to use the new Az.Storage PowerShell module, which is now the requirement for this module to work since the old Microsoft.WindowsAzure.Storage assembly got replaced by Microsoft.Azure.Cosmos. It also runs on PowerShell core as well, tested on PS 5.1, PS 6.2 and Linux PS. Kudos to [jakedenyer](https://github.com/jakedenyer) for his contributions on async methods.

**Update 03/08/2018**: Due to a number of conflicts related to assemblies needed on Storage Tables and Cosmos DB, as of this date, Cosmos DB support was removed from this module and I'll be working in a separate module for Cosmos DB and this will be release shortly.

**Update 12/04/2017**: Due to a large number of customers that have applications that adds a Timestamp entity attribute to the tables, the default Azure Table system attribute called Timestamp got renamed to TableTimeStamp so customers can consume their application’s own Timestamp value and avoid conflicts, this might be a breaking change but it is a necessary change. This change is implemented on version 1.0.0.21.

**Update 05/22/2017**: Since module version 1.0.0.10 it was added a cmdlet (Get-AzureStorageTableTable) and support for Azure Cosmos DB Tables, for more information please refer to Azure RM Storage Tables PowerShell module now includes support for Cosmos DB Tables blog post.

**Release Notes**: For more details about updates being done in the module, please refer to its release notes on [GitHub](https://github.com/paulomarquesc/AzureRmStorageTable/blob/master/ReleaseNotes.md).

## Introduction<a name="intro"></a>
Azure Storage Tables is one of the four Microsoft Azure Storage abstractions available (Blobs, Queues and Azure Files are the other ones) at the time that this blog was written. It is basically a way to store data in a structured way on a non relational database system (meaning, not an RDBMS system) based on key-value pairs.

Since up to today there are no official cmdlets to support entity/row management inside the tables from Azure PowerShell module, I decided to create this simple module to help IT Pros to leverage this service without having knowledge of .NET framework through some simple cmdlets as follows:

{: .table.gridtable}
| **Cmdlet**                         | **Description**                                                                                              |
| ---------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| Add-AzTableRow                     | Adds a row/entity to a specified table                                                                       |
| Get-AzTableRow                     | Used to return entities from a table with several options, this replaces all other Get-AzTable<XYZ> cmdlets. |
| Get-AzTableRowAll                  | (Deprecated) Returns all rows/entities from a storage table - no Filtering                                   |
| Get-AzTableRowByColumnName         | (Deprecated) Returns one or more rows/entities based on a specified column and its value                     |
| Get-AzTableRowByCustomFilter       | Returns one or more rows/entities based on custom Filter                                                     |
| Get-AzTableRowByPartitionKey       | Returns one or more rows/entities based on Partition Key                                                     |
| Get-AzTableRowByPartitionKeyRowKey | Returns one entity based on Partition Key and RowKey                                                         |
| Get-AzTableTable                   | Gets a Table object to be used in all other cmdlets                                                          |
| Remove-AzTableRow                  | Removes a specified table row                                                                                |
| Update-AzTableRow                  | Updates a table entity                                                                                       |

> Note: For compabilitiy purposes, aliases were created for the old noun style `AzureStorageTable`. New noun, `AzTable`, is the replacement that will allow you to automatically load the `AzureRmStorageTable`. If you wish to continue to use the old names, make sure you explicitly load the module before using them.

There are a number of use cases for an IT Pro work with Azure Tables from PowerShell where it becomes a great repository, the ones below are just few examples:

* Logging for SCCM executed scripts
* Azure Automation for VM expiration, shutdown, startup in classic mode (Azure Service Manager)
* VM Deployment scripts, where it becomes a central location for logs
* Easily extract Performance and Diagnostics data from VMs with Diagnostics enabled

## Requirements<a name="requirements"></a>
PowerShell 5.1 or greater (this is also supported on Linux PowerShell)
This module requires the new [Az Modules](https://docs.microsoft.com/en-us/powershell/azure/new-azureps-module-az?view=azps-1.5.0):
* Az.Storage (1.1.0 or greater)
* Az.Resources (1.2.0 or greater)

You can get this information by issuing the following command line from PowerShell:

{: body.gist.highlight % gist paulomarquesc/4f89769c3a243b79c5784053a24addfc %}
 
## Installation/Source Code<a name="install"></a>
Since this module is published on [PowerShell Gallery](https://www.powershellgallery.com/packages/AzureRmStorageTable), you can install this module directly from PowerShell 5.1 or greater and Windows 10 / Windows Server 2016 / Linux by executing the following cmdlet in an elevated (Windows) PowerShell command prompt window:

{% gist paulomarquesc/2540cd5260c2ac475be066c4aac18182 %}
 
As an alternative, you can manually download the module from my GitHub repository [here](https://github.com/paulomarquesc/AzureRmStorageTable) and extract to `C:\Program Files\WindowsPowerShell\Modules` for PowerShell 5.1 or  `C:\Program Files\PowerShell\Modules` for PowerShell 6.2 , or greater, and rename the folder to AzureRmStorageTable. Please remember to **unblock** the zip file before extracting it.

## Using Azure Storage Table PowerShell Module<a name="psstoragetable"></a>
The following steps will walk you through loading the module and perform one or more example tasks of the basic operations offered in this module.

Before you start working with it, you need to authenticate to Azure and select the correct subscription if you have multiple subscriptions:

{% gist paulomarquesc/225401da514b9b86d4e44c4c27fb5391 %}

Next, lets import AzureRmTableStorage PowerShell module and list the functions available on it:

{% gist paulomarquesc/5e6cb412f71e57818991ebbe0c48c5fc %}

You should see the following cmdlets:

![cmdlets](/assets/{{postPath}}/cmdlets.png)

For the sake of simplicity, we need to define some variables at this point to make our examples a little bit more clean, please, **make sure you have an storage account already created and that you change the values of the variables below to reflect your environment**. Notice that one of the variables is called **$partitionKey**, in this example we are using only one partition, please refer to the documentation at the end of this blog in order to get a better understanding on partitions.

{% gist paulomarquesc/22fb97aa3ad3c486938d8c2d85aa0e4e %}

Get a reference for your table with the following cmdlet:

{% gist paulomarquesc/afc9628b0df022330cd278e04958fd40 %}

You can check details of your table by executing typing `$table`:

![table details](/assets/{{postPath}}/table_details.png)

Optionally, you can obtain the table reference by using the Az.Storage cmdlets:

{% gist paulomarquesc/b2ded6595273f2164541b57bded19876 %}

> Note: With release of Az.Storage 1.1.0, you need to reference the CloudTable property of the table returned from Get-AzStorageTable, that was not needed in the previous versions, ideally you will use Get-AzTableTable cmdlet instead but there might be cases where you need the Get-AzStorageTable.

Up to this point we just prepared our PowerShell session by authenticating, importing the module, setting up some variables and getting our table, from this point moving forward we will focus on the basic operations exposed through the module. I’m creating a section per function/operation.

### Adding Rows/Entities<a name="adding"></a>
#### Adding lines one by one

{% gist paulomarquesc/8e5b613fe928b74d4ff238a159f79970 %}

Result, notice the 204 HttpsStatusCode, indicating that your operation succeeded.

![results](/assets/{{postPath}}/image388.png)

#### Getting data from a JSON string and using a foreach loop to load the data

{% gist paulomarquesc/47e7a24c018aec8092279be6f5737fbd %}

![results json](/assets/{{postPath}}/image389.png)

If we open [Azure Storage Explorer](https://azure.microsoft.com/en-us/features/storage-explorer/) and navigate to the table, we will see all inserted entities.

![storage explorer](/assets/{{postPath}}/image390.png)

### Retrieving Rows/Entities<a name="retrieving"></a>

When retrieving rows using the functions described below, they will return a **PSObject** instead of a **DynamicTableEntity** and since they will give you some extra work to manipulate/access the properties I decided to make the Get cmdlets return PSObject instead.

Example a DynamicTableEntity object:
![DynamicTableEntity Object](/assets/{{postPath}}/dynamicTableEntity.png)

Example of PSObject when it is returned from the functions exposed in this module:
![DynamicTableEntity Object](/assets/{{postPath}}/psobject.png)

> Note: when working with custom queries, the properties to be used are the ones that are shown in the DynamicTableEntity. If you're bringing more rows and then will be using PowerShell features for filtering (e.g. Where-Object), then you must refer to the PSObject properties instead. 

#### Retrieving all rows/entities

{% gist paulomarquesc/3262574b8a785a369e4fb5b945cf08ce %}

Result

![result all](/assets/{{postPath}}/image392.png)

#### Getting rows/entities by partition key

{% gist paulomarquesc/af8a8fedc230ae66637a20abba9946b6 %}

Result

![result by partition key](/assets/{{postPath}}/image393.png)

#### Getting rows/entities by specific column

{% gist paulomarquesc/b719219c54524c6e898be46f92bc7d77 %}

Result

![result by column](/assets/{{postPath}}/image394.png)

Example using a guid entity type:

{% gist paulomarquesc/9aed90a80fc6f0cf1f4238d5531489fb %}

![result by column and guid](/assets/{{postPath}}/guid.png)

#### Queries using custom filters with help of Microsoft.Azure.Cosmos.Table.TableQuery class and direct string text

##### Simple filter

{% gist paulomarquesc/c55735f6e52f5311fca51414d56843a0 %}

Result

![simple filter](/assets/{{postPath}}/image395.png)

##### Combined filter

{% gist paulomarquesc/8c370716372d5b38dfe1c9b46e8bf718 %}

Result

![combined filter](/assets/{{postPath}}/image396.png)

##### String filter

{% gist paulomarquesc/24f6a8090e8ddc7c3994b9e326c83d8d %}

Result

![string filter](/assets/{{postPath}}/image397.png)

### Updating an entity<a name="updating"></a>

This process requires three steps:

* Retrieve the row/entity to update
* Perform the change on this retrieved item
* Commit the change

> Note: **Update-AzTableRow** function will accept one entry at a time, don’t pass an array of entities or pipe an array of entities to the function.

Example:

{% gist paulomarquesc/92f2522254d788cc16e8d0a558538d38 %}

Result

![updating](/assets/{{postPath}}/image398.png)

### Deleting rows/entities<a name="deleting"></a>

Similarly to the update process here we have two steps as follows unless you know the partitionKey and rowKey properties, in this case you can delete directly:

1. Retrieve the entity
2. Delete the entity passing the retrieved one as argument

#### Deleting a single row/entity by piping the entity

{% gist paulomarquesc/dadbecfbd30a8154a96540af2fb42649 %}

Result

![deleting single row by entity piping](/assets/{{postPath}}/image404.png)

#### Deleting a single row/entity passing entity as argument

{% gist paulomarquesc/89a88877bcdd9d836f3a69b5b5213b58 %}

Result

![deleting single row by entity piping](/assets/{{postPath}}/image405.png)

#### Deleting an entry by using PartitionKey and RowKey directly

{% gist paulomarquesc/ea3907c5bf6b31d206eddb4bf98db618 %}

Result

![deleting single row by entity piping](/assets/{{postPath}}/image406.png)

#### Deleting everything

{% gist paulomarquesc/16311d57908ab06ebc866233bce92bc2 %}

## References<a name="references"></a>

For more information about Azure Storage Tables, please refer to the following documents:

* [Perform Azure Table storage operations with Azure PowerShell](https://docs.microsoft.com/en-us/azure/storage/tables/table-storage-how-to-use-powershell)
* [Get started with Azure Table storage using .NET](https://docs.microsoft.com/en-us/azure/storage/storage-dotnet-how-to-use-tables)
* [Azure Storage Client Library for .NET](https://msdn.microsoft.com/library/azure/mt347887.aspx)
* [Getting Started with Azure Table Storage in .NET](https://azure.microsoft.com/en-us/resources/samples/storage-table-dotnet-getting-started/)
* [Azure Storage Table Design Guide: Designing Scalable and Performant Tables](https://docs.microsoft.com/en-us/azure/storage/storage-table-design-guide)

