---
layout: post
title: Upgrading Umbraco from v4 to v7
---

Recently for work I've had the opportunity to upgrade a website from using version 4 of Umbraco to version 7. Due to the significant investment already made in the website, the upgrade needed to be able to  reuse as much of the existing customisations and templates as possible. Fortunately version 7 still supports the older template format that used ASP.net master pages.

Upgrading an Umbraco website is composed of two main parts. Upgrading the umbraco database, and upgrading the website content itself.

# Upgrading Database
Normally the database is upgraded the first time that the website is started after being upgraded. An administration screen is displayed on startup, prompting the user to login and authorise the upgrade.

While this is practical for upgrades between minor versions, there can be larger database changes that prevent this being done in a single step, such as the changes to the authentication system in 7.3.X. 
To handle the database upgrade process, I suggest incrementally stepping through versions to perform the upgrade. An upgrade project can be done by creating an empty mvc website, then installing the desired umbraco version through nuget, and configuring it to use your database. Recent versions of Umbraco have also changed to use a connection string instead of an appsetting for the database connection.
 
 - current version config settings

For my upgrade, I stepped through the following versions:
- 6.2.4
- 7.3.1
- 7.5.15 (Initially went to 7.6.4, but had to go to an earlier version after discovering a bug).

- upgrade to 6, then 7.3, then destination version
- Version 7 is currently broken for winforms templates

# Upgrading website
Moving from Umbraco 4 to 7 not only brings a number of API changes, but also raises the minimum supported version of .net to 4.5.2. When the .net version on a project is changed, the nuget packages need to be reinstalled so that the correct version of the assembly is targeted. By changing the .net version before upgrading the Umbraco libraries, the amount of manual reinstallation of packages can be reduced.

Reinstall nuget package:
Update-Package <package_name> -ProjectName MyProject -reinstall

https://docs.microsoft.com/en-us/nuget/consume-packages/reinstalling-and-updating-packages