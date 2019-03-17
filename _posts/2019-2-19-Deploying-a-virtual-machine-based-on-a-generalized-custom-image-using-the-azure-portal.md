---
layout: post
title: Deploying a virtual machine based on a generalized custom image using the Azure Portal  
excerpt_separator: <!--more-->
comments: true
---
{% assign postPath = page.path | remove: '%0A' | remove: '_posts' | remove: '.md' %}

In this blog post we are showing quickly how to deploy a VM in Azure from a custom image using the [Azure Portal](https://portal.azure.com).

<!--more-->

## Quick steps to deploy a custom VM image from Azure Portal

This blog post is an updated version of my previous post [Deploying a virtual machine based on a generalized custom image using the Azure Portal](https://blogs.technet.microsoft.com/paulomarques/2017/02/17/deploying-a-virtual-machine-based-on-a-generalized-custom-image-using-the-azure-portal)

### Locating the Images service
![finding-images](/assets/{{postPath}}/finding-images.png)

1. Open [Azure Portal](https://portal.azure.com) and click "All services" in the left-hand side.
1. Within "All services" search text box, type "images"
1. Click on "Images" icon from the search result (Optionally, you can click the star icon next to Images item to make it available in the left navigation pane)

### Deploying a VM from the custom managed image

1. With the "Images" blade opened, click the custom image you want to create a new VM from
   
    ![finding-images](/assets/{{postPath}}/selecting-custom-image.png)

2. Click "+ Create VM" button in the `<your custom image name>` blade to deploy your new VM. If you included data disks before capturing the image you will notice the empty slots for data disks, this means that when your VM is deployed from this custom image it will have that number of new empty data disks attached to it. If you want these disks to be automatically configured as a raid 0 volume within your Linux Vm please refer to [Automating raid0 MDADM volume creation on a Linux VM when using a custom image in Azure](https://paulomarquesc.github.io/Azure-Linux-VM-Create-Raid0-Volume)
   
    ![finding-images](/assets/{{postPath}}/create-new-vm.png)

3. Follow next steps as you would normally do when creating a VM from the marketplace in the "Create a virtual machine" blade.
   
   ![finding-images](/assets/{{postPath}}/create-vm-wizard.png)

### References
* [Azure and Linux](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/overview)
* [Quickstart: Create a Linux virtual machine in the Azure portal](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/quick-create-portal)



