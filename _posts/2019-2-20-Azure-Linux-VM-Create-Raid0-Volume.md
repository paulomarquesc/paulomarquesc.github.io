---
layout: post
title: Automating raid0 MDADM volume creation on a Linux VM when using a custom image in Azure  
excerpt_separator: <!--more-->
comments: true
---
{% assign postPath = page.path | remove: '%0A' | remove: '_posts' | remove: '.md' %}

In this blog post we are looking into one of the multiple ways to automate the process of adding Azure data disks as a single raid 0 volume when using custom images in Azure.

<!--more-->

### Intro
There are a number of ways to automate this process, in this post we are showing one way when you're using a custom Linux image in Azure. Customizing an Azure marketplace Linux image is a very common scenario in Azure and since this customization process is happening anyways, this example shows how to create this new volume as a raid 0 device and mount it while deploying the VM without the need to rely on Azure VM Custom Script Extension for example.

This post assumes you are starting the customization process from a master VM deployed in Azure with an image from the marketplace (this example was tested with CentOS 7.6 from Rogue Wave Software), this simplifies the image build process. If this is not the case, the [Prepare a CentOS-based virtual machine for Azure](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/create-upload-centos) and [Information for Non-endorsed Distributions](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/create-upload-generic) documents can help you with more detailed information.

Most of the steps outlined here will be showed only as highlevel tasks, if using the same Azure image as mentioned before, you can refer to [How to create an image of a virtual machine or VHD](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/capture-image) document for more detailed information.

### Steps

1. Deploy a CentOS 7.6 VM from the marketplace as your master image VM, choosing at least a Standard_D4S_v3 to be able to use Accelarated Networking (https://docs.microsoft.com/en-us/azure/virtual-machines/linux/quick-create-portal)
2. Attach the data disks manually or if you prefer, use [Azure Cloud Shell](https://shell.azure.com) and download the add_datadisks.sh script that helps automating this task by downloading it and executing from within Cloud Shell. {% gist paulomarquesc/294a9239713ebe6c0ff01341e4c2352a %}

3. SSH into the newly deployed VM (https://docs.microsoft.com/en-us/azure/virtual-machines/linux/quick-create-portal#connect-to-virtual-machine)
   
4. Download these two (and change attribute to eXecute) scripts from within the VM. {% gist paulomarquesc/28594fd24fe0837b1aea4efbb66dcbdf %}

5. Install the create_linux_raid_volume.sh script to be executed from cron after VM gets started (or rebooted). {% gist paulomarquesc/59635ae365c46f4d24e196b6364520f9 %}
    > Note: the three arguments states the mount point of the new raid 0 volume, the file system type (ext4 or xfs) and the number of data disks this custom image have, in this example 8 data disks. It is very important to set the number of disks with the same number of data disks you are attaching before generating the image.

    > IMPORTANT: After downloading **create_linux_raid_volume.sh** script, please review the [mount options](https://github.com/paulomarquesc/linux-raid-script/blob/master/create_linux_raid_volume.sh#L203-L214) it configures before capturing the image since those on my sample script may not meet your needs, therefore you have the opportunity to change it as needed.

6. Check if script create_linux_raid_volume.sh was copied to /root folder and that there is an entry on cron to execute this script at reboot. {% gist paulomarquesc/ff8416dd58f00846ad7d5c59b15f59a7 %}

    Result:
    ![checking-script-and-cron](/assets/{{postPath}}/checking-script-and-cron.png)

7. Deprovision this VM so it can be captured {% gist paulomarquesc/dfeda5bdbbb2f00c4a5076273e8fdf12 %}

8. Back to the [Azure Cloud Shell](https://shell.azure.com), execute the following commands in order to create your image {% gist paulomarquesc/f9c6019825bf11e507087752b3f59f73 %}
    Example {% gist paulomarquesc/14ac2c7760c11dd0db1f54c3b72d948f %}


> Note: When deploying this image, bear in mind that in this case, the data disk creation will happen in the background and any other automation you put in place after the VM is deployed should test/wait for the availability of this raid 0 volume's mount point.

### Deploying a VM from custom image using Azure CLI

Now that you have your image created, you can use Azure CLI to deploy a new VM from this custom image. This example deploys a VM attaching to an existing virtual network (no public access is definied here nor network security group for the sake of the example only). This can also be achieved via Templates, Powershell or [Azure Portal](https://portal.azure.com).

From [Azure Cloud Shell](https://shell.azure.com):

1. Create a NIC (we are enabling Accelerated Networking). {% gist paulomarquesc/c4ad1485130414f9d1aa6f98e662167c %}

2. Deploy VM from new custom image. {% gist paulomarquesc/02400dcbf39b275f07d2b3a3994b0147 %}

> Note: above commands requires that the vnet and image be in the same resource group.

The following [blog post](https://paulomarquesc.github.io/Deploying-a-virtual-machine-based-on-a-generalized-custom-image-using-the-azure-portal/
) shows an example on how to deploy this custom image using the Azure portal.

### References
* [linux-raid-script repository](https://github.com/paulomarquesc/linux-raid-script)
* [Azure and Linux](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/overview)
* [Deploying a virtual machine based on a generalized custom image using the Azure Portal](https://paulomarquesc.github.io/Deploying-a-virtual-machine-based-on-a-generalized-custom-image-using-the-azure-portal)
* [Prepare a CentOS-based virtual machine for Azure](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/create-upload-centos)
* [Information for Non-endorsed Distributions](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/create-upload-generic)
* [Quickstart: Create a Linux virtual machine in the Azure portal](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/quick-create-portal)


