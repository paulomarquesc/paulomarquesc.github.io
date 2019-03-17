---
layout: post
title: How to recover Service Principal Information
excerpt_separator: <!--more-->
comments: true
---

This blog post helps you recover Azure AD Service Principal information and also explains how to append a new password in order to reuse it.

<!--more-->

Some automation scripts/templates sometimes requires the use of Service Principals, as an example, [this](https://github.com/Azure/Avere/tree/master/src/vfxt) Avere vFXT deployment guide requires it. [Avere vFXT](https://www.averesystems.com) is a Microsoft Storage solution for HPC workloads.

In this deployment example, the following Service Principal information is required in order to deploy Avere vFXT cluster using the aforementioned template solution, Service Principal Tenant Id, App Id and password, please execute the following steps in [CloudShell](https://shell.azure.com) to recover it.

1. If at  Service Principal creation time the "--name" was not specified, **az ad sp create-for-rbac** command creates the service principal with its displayName starting with **azure-cli**, followed by date and time, in this case you can first obtain the list of service principals and then select one from the output list, if you already know the full displayName, skip to step 2. {% gist paulomarquesc/1da97d0ab77cb74d92fb5ab4161fe71b %}

2. If you know the exact displayName you can use this commandLine (make sure you change the variable content in the first line) {% gist paulomarquesc/a590959bf2b0586264c1468247e0e7f1 %}

3. Either commands will output all needed information with exception of the password as follows. {% gist paulomarquesc/0793a450de0f1d4adac6c3e66c7c76c8 %}
   
4. Add a new password to the service principal, make sure that the value of **--name** is the same as the **AppDisplayName** from the output you obtained from the above queries (password will be appended to the Application associated to the Service Principal and it may or may not match the DisplayName). {% gist paulomarquesc/24bad6f7d89c6b40c1a3c6d3928300ac %}

5. The following output is all you need to continue your deployment (notice that with exception of the password, the values matches your previous query results): {% gist paulomarquesc/eb331f9b2b9bc98c7242e3d7e0eec121 %}