---
layout: post
title: Protecting Windows Virtual Desktops Session Hosts Using Azure Recovery Services Vault
excerpt_separator: <!--more-->
comments: true
---
{% assign postPath = page.path | remove: '%0A' | remove: '_posts' | remove: '.md' %}

In this post I will walk you through the steps to allow you to protect WVD Session Hosts while using Persistent Desktop scenarios, where backups are important since data may reside in the virtual machine itself. This also very important on some types for regulatory compliance scenarios, where for example you're required to keep the user's data for X amount of time.

<!--more-->

This document will guide you through all steps for configuring backup. This capability is backed by an Azure Service called **Backup and Site Recovery(OMS)**, amongst other features, it provides the backup/snapshot infrastructure for VMs in Azure.

### In this blog post

- [Creating a Recovery Services Vault](#creating-a-recovery-services-vault)
- [Configuring a Backup Policy](#configuring-a-backup-policy)
- [(Optional) Changing Recovery Services Vault Storage Replication Configuration](#optional-changing-recovery-services-vault-storage-replication-configuration)
- [Enabling Scheduled Session Host Backup](#enabling-scheduled-session-host-backup)
  - [Using the PowerShell Script](#using-the-powershell-script)
    - [Pre-Requisites](#pre-requisites)
    - [Steps](#steps)
  - [Checking the results](#checking-the-results)
- [Starting an On-Demand Session Host Backup](#starting-an-on-demand-session-host-backup)
  - [Backup Based on a User Principal Name from PowerShell](#backup-based-on-a-user-principal-name-from-powershell)
  - [Manual Backup using the Azure Portal](#manual-backup-using-the-azure-portal)
- [Recovering a Session Host](#recovering-a-session-host)
  - [Recovering an existing Session Host into the same VM](#recovering-an-existing-session-host-into-the-same-vm)
  - [Recovering a Session Host that does not exist anymore (e.g. deleted)](#recovering-a-session-host-that-does-not-exist-anymore-eg-deleted)
- [Considerations](#considerations)
- [References](#references)

## Creating a Recovery Services Vault

Recovery Services Vault is the key component of Backup and Site Recovery services, it is responsible for holding in a secure manner VM backups.

One key concept here is that we will have one Recovery Services Vault per region where Session Hosts are deployed, for example, if your Session Hosts for a given Host Pool are located in East US, then a vault will be required to be created in the same region (East US). We also need to pay attention to the limit of 250 server per vault, create more vaults per region as needed.

1. From Azure Portal, click on **“+ Create a resource”** item in the left-hand navigation pane

2. At “Search the Marketplace” field, type **Backup and Site Recovery (OMS)** and select it from the combo box

3. Click **“Create”** button

4. Taking into consideration the regional backups and limits, consider a good naming convention, in my example I’ll use “PMC-EUS-RecoveryVault-01” as the name of my vault.

5. Choose the Subscription, Resource Group (create a new one if needed) and Location where it will be located

6. Click **“Create”** button
   
    ![Recovery Services Vault Creation](/assets/{{postPath}}/0d7e62355ba8076f8f3031eacb761996.png)

## Configuring a Backup Policy

Since different Host Pools in the same region may have different backup needs it would be a good idea that you specify one policy per **Host Pool**, even if they are the same to begin with, this will give you flexibility to change in the future if you decide for a different policy and avoid extra work to revisit your VMs to change the policy later.

1. Select your newly created Recovery Services Vault, in my case **PMC-EUS-RecoveryVault-01** and click **“Backup policies”** item in the left blade pane.

2. Click **“+Add”** button

3. Select **Azure Virtual Machine**

4. At **“Policy name”**, type the Host Pool name so it makes it easier for you to manage and associate with which Host Pool this is about (Notice that no special characters and spaces are allowed).

5. Regarding policies, of course each company and even Host Pools will have their own needs, this is what I’m configuring in this example:

    *Backup schedule*
    - Frequency: **Daily** (Azure VMs allows one backup a day)

    - Time: **11:30 PM**

    - Timezone: **East US** (backups will be scheduled and executed using the same timezone of the Session Hosts in this example)

    *Instant Restore*

    - Retain instant recovery snapshot(s) for: **5** Day(s) (in this case we’re allowing faster recovery within a 5 days period, more than this will recover data from the vault instead of a storage account which will take more time to complete)

    *Retention range*

    - Daily backup point
        - For: **180** Day(s) (maximum limit of 9999 days)

    - Retention of weekly backup point
        - On: **Sunday**
        - At: **11:30 PM**
        - For: **12** Week(s) (maximum limit of 5163 weeks)

    - Retention of monthly backup point
        - **Week Based**
        - On: **First**
        - Day: **Sunday**
        - At: **11:30 PM**
        - For: **60** Month(s) (maximum limit of 1188 weeks)

    - Retention of yearly backup point
        - **Week Based**
        - In: **January**
        - On: **First**
        - Day: **Sunday**
        - At: **11:30 PM**
        - For: **10** Year(s) (maximum of 99 year)
   
        ![Create Policy Example](/assets/{{postPath}}/36e6465e50ac1501a75387168df11a3b.png)

6. Click **“Create”** button

## (Optional) Changing Recovery Services Vault Storage Replication Configuration

**By default, backup storage replication type is set as Geo-Redundant** which gives you regional failure resilience but if needed, you can change it to Locally-redundant type. Notice that this setting can be changed only while you don’t have any backup items protected to the vault. To change it for Locally-redundant setting:

1. Open your recovery services (PMC-EUS-RecoveryVault-01 in this example)

1. Click **“Properties”** item of Recovery Services vault blade

1. Click **“Update”** link under BACKUP-\>Backup Configuration

1. Click “Locally-redundant” on “Storage replication type”

1. Click “Save” button

    ![Recovery Vault Replication Change](/assets/{{postPath}}/e1e493b2907c097981768f8a6ef01f54.png)

## Enabling Scheduled Session Host Backup

Our last step is enabling automatic backups based on a specific backup policy on each Session Host. Although this can be done through the Portal, we’re presenting a method with a PowerShell script that will allow you to enable backup to all Session Hosts automatically within a Host Pool, also it will allow you to re-execute the script and enable it to new Session Hosts recently added to the pool that does not have backup enabled yet.

### Using the PowerShell Script

#### Pre-Requisites

-   Windows PowerShell 5.0 and 5.1

-   Az PowerShell module 1.6 or greater

-   Microsoft.RDInfra.RDPowerShell PowerShell module

-   Azure Recovery Services Vault already deployed within the same region as the
    Session Hosts

#### Steps

1. From a PowerShell command prompt window, download the Enable-RecoveryServicesSessionHostBackup.ps1 script (this is a single line command)

    {% gist paulomarquesc/d437e38c95f87593345d18ef1ba08e46 %}

1. Authenticate against Azure and select subscription if needed

    {% gist paulomarquesc/225401da514b9b86d4e44c4c27fb5391 %}

1. Authenticate against WVD Tenant and set the context to the desired Tenant
    Group

    {% gist paulomarquesc/086f7bd54a7efb6b50adb7d54e1ec947 %}

1. Execute the script

    {% gist paulomarquesc/7838ca7a31f4d0688f0a971f3841a993 %}

This is a sample result, where one backup job to enable backup will be created per Session Host:

![Sample execution result](/assets/{{postPath}}/8f0c0b879fa41e55a6d292832e442d1b.png)

> Note: This script allows you to execute it again and it will inform you if VMs are already protected and will not error out, plus will enable to new
unprotected VMs.

![Re-execution result](/assets/{{postPath}}/ef73c731def2ee9929b3e20478ef5d2d.png)

### Checking the results

To make sure you have enabled backup on all Session Hosts, you can check in two places, one will be at the recovery vault and the other way is to check the VM itself

- Recovery Services Backup Job Status for enabling the VMs

    ![Recovery Services Job Statuses](/assets/{{postPath}}/445b92763a16dd7e79e3ff37115be9e6.png)

- Checking backup status directly on Session Host VM

    ![Checking status in VM itself](/assets/{{postPath}}/68ed1ea9a652b25259380db10f9a1d93.png)

## Starting an On-Demand Session Host Backup

As you noticed we applied a backup policy to the Session Hosts, and it will back up our VMs every day but there will be times where you will need to manually or through a script that is triggered by integration with another system using PowerShell. This can be achieved in multiple ways and in this document we are show casing one option using PowerShell and one very basic which is using the Azure Portal backing up the VM by its Backup option.

### Backup Based on a User Principal Name from PowerShell

If this backup process is mandatory due to certain compliance regulations, the easiest way to execute this backup is through the PowerShell script below, one of the parameters will be the User Principal Name, then it will identify which Session Host needs to be backed up if the Host Pool is using Persistent Desktop scenario.

1. From a PowerShell command prompt window, download the Enable-RecoveryServicesSessionHostBackup.ps1 script (this is a single line command)

    {% gist paulomarquesc/f69632950dfadcf3e542b0fdc6f6af9e %}

1. Authenticate against Azure and select subscription if needed

    {% gist paulomarquesc/225401da514b9b86d4e44c4c27fb5391 %}

1. Authenticate against WVD Tenant and set the context to the desired Tenant Group

    {% gist paulomarquesc/086f7bd54a7efb6b50adb7d54e1ec947 %}

1. Execute the script

    {% gist paulomarquesc/c7c3d8fec4768692c4a4d8089a407942 %}

Result screen:

![Starting Backup Job Result](/assets/{{postPath}}/196243caa302feb7d72f04c13a8f09c2.png)

This script also returns each backup job object if you want to perform further operations like checking the backup status, for that, just add a variable before the script, for example:

![Starting Backup Job Result and returning value to a variable](/assets/{{postPath}}/6d06fc0d3031a50e0623e1be5574410c.png)

> Note: this script allows you to backup a specific Session Host or all Session Hosts within a Host Pool, for more info on these options, execute:
`Get-Help .\Start-SessionHostBackup.ps1` in the same folder where your script is located and the help text will show these other options.

![Job object content](/assets/{{postPath}}/8e09790d13b2cf376ea52f5d36809e34.png)

To get an updated status from PowerShell by using this object (notice that this could be an array of multiple Session Hosts being backed up by the script, in this case you need to reference the specific array item):

1. Get Recovery Vault reference (parameter values are examples only)

    {% gist paulomarquesc/4f6a93e0f2df92382dd3f8892b4fb5ff %}

1. Getting an updated summary

    {% gist paulomarquesc/7a8530433f324729a3e2ea4c4814a423 %}

    ![Job status](/assets/{{postPath}}/3907edb36f2b6edf3fab913bf61941c8.png)

1. Getting more detailed information

    {% gist paulomarquesc/f29ec25e0454ddbd6036da3789aac733 %}

    ![Detailed job status](/assets/{{postPath}}/fa009c6f2dd253d50fe9389092bafaa2.png)

If you don’t have the job id or the job object you can always list all jobs (and possibly apply PS filtering capabilities) to check all our backup jobs:

![All jobs status](/assets/{{postPath}}/753713a0c1cac9a1518c9aea7c591996.png)

### Manual Backup using the Azure Portal

1. Within the Azure Portal, open your VM resource and click Backup item on your Session Host VM blade

1. Click **“Backup now”** button

1. Pick a retention period in **“Retain Backup Till”** date field

1. Click **“OK”** button

![Manual backup](/assets/{{postPath}}/5544e83c0ba758bbed54041687a6ca9b.png)

## Recovering a Session Host

Similarly, to backup options, there are many ways to restore a VM, to a point where you can restore individual files, the following steps will guide on one of the options which is fully restoring a VM over the existing one. Notice that for this scenario, the VM needs to be in **deallocated** state.

### Recovering an existing Session Host into the same VM

1. Navigate to the Session Host you need to restore within the Portal and then to Backup option

1. Click **“Restore VM”** button

1. Select the desired recovery point, if restore point is up to 5 days old this restore will happen from a snapshot which is a quicker method then bringing the data from Recovery Vault.

1. Click **“OK”** button

1. Select **“Replace existing”**

1. Select a **“Staging Location”** (make sure you have a storage account available in the same region as your Session Host)

1. Click **“OK”** button

1. Click **“Restore”** button

![Recovering Steps](/assets/{{postPath}}/4e6d7feb8c8be700927f8295e23d9a19.png)

You can monitor the job status from PowerShell as exemplified earlier or monitor it directly in the Portal from the Recovery Services Backup Report item:

![Monitoring Restore](/assets/{{postPath}}/1c79e77156d413b87d9f0ae07dbb1511.png)

### Recovering a Session Host that does not exist anymore (e.g. deleted)

In an extreme case where the Session Host was deleted, you can always restore it from the Recovery Vault itself. Follow these steps to recover a completely deleted VM (or even a VM that was decommissioned as part of its lifecycle, e.g. employee left the company):

1. Navigate to your Recovery Vault resource

1. Click **“Backup items”** item

1. Click **“Azure Virtual Machine”** from the Backup Management Type list

1. Click **\<VM Name\>** that is the Session Host to want to restore

1. Click **“Restore VM”** button

1. Select the desired recovery point.

1. Click **“OK”** button

1. Select **“Create new”** (remember that this is the case where the VM does not exist anymore)

1. Under **“Restore Type”**, select **“Create virtual machine”**

1. Type the VM Name

1. Select the same resource group, virtual network, subnet (ideal if replacing an accidentally deleted VM, if this is for compliance purposes, make sure you plan accordingly on where to restore this VM to)

1. Select a **“Staging Location”** (make sure you have a storage account available in the same region as your Session Host)

1. Click **“OK”** button

1. Click **“Restore”** button

![Restoring as New VM](/assets/{{postPath}}/47f31504e5faf2c426b3e0722e6065bb.png)

> Important: if you are restoring a Session Host for compliance purposes, don’t restore in the same production virtual network otherwise you may have conflicts with possibly a VM with same name existing in Active Directory, it also may introduce issues due to computer account staleness in Active Directory as well, make sure you reset the local user administrator password by using **Support + troubleshooting-\> Reset Password** VM option.

For other restore options, please refer to:

[Restore Azure VMs](https://docs.microsoft.com/en-us/azure/backup/backup-azure-arm-restore-vms)

## Considerations

Azure IaaS VM Backup offers one backup per day, if two are required then Azure Backup Server or System Center DPM (both IaaS based solutions) should be considered since they offer more flexibility in this regard, but the downside is that you will need to architect and manage the infrastructure.

Basic tiering in Azure IaaS VM Backup, Snapshots are kept in a storage account for up to 5 days, after this period, it will be available only from Recovery Vault which makes it less costly but the restore time will take longer, as explained earlier as well, it also supports Local or Geo redundancy, which can be changed only if there is no protected items and backups, if a different tiering type is needed, System Center DPM offers backup to tape option at this time but this is an on-premises solution only.

For more information please refer to:

[What are the features of each Backup component?](https://docs.microsoft.com/en-us/azure/backup/backup-introduction-to-azure-backup#what-are-the-features-of-each-backup-component)

[Backup and retention](https://docs.microsoft.com/en-us/azure/backup/backup-introduction-to-azure-backup#backup-and-retention)

## References

* [What is Windows Virtual Desktop?](https://docs.microsoft.com/en-us/azure/virtual-desktop/overview)
* [Overview of the features in Azure Backup](https://docs.microsoft.com/en-us/azure/backup/backup-introduction-to-azure-backup)
* [About Azure VM backup](https://docs.microsoft.com/en-us/azure/backup/backup-azure-vms-introduction)
* [Back up Azure VMs in a Recovery Services vault](https://docs.microsoft.com/en-us/azure/backup/backup-azure-arm-vms-prepare)
* [Back up an Azure VM from the VM settings](https://docs.microsoft.com/en-us/azure/backup/backup-azure-vms-first-look-arm)
* [Back up a virtual machine in Azure with PowerShell](https://docs.microsoft.com/en-us/azure/backup/quick-backup-vm-powershell)
* [Back up a virtual machine in Azure](https://docs.microsoft.com/en-us/azure/backup/quick-backup-vm-portal)
* [Enable and Start Backup Scripts repo](https://github.com/Azure/RDS-Templates/tree/master/EnableBackupScript)
