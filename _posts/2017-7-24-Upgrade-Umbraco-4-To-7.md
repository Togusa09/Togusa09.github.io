---
layout: post
intro: Upgrading Umbraco from v4 to v7
---

Recently for work I've had the opportunity to upgrade a website from using version 4 of Umbraco to version 7. Due to the significant investment already made in the website, the upgrade needed to be able to  reuse as much of the existing customisations and templates as possible. Fortunately version 7 still supports the older template format that used ASP.net master pages.

Upgrading an Umbraco website is composed of two main parts. Upgrading the umbraco database, and upgrading the website content itself.

# Upgrading Database
Normally the database is upgraded the first time that the website is started after being upgraded. An administration screen is displayed on startup, prompting the user to login and authorise the upgrade.

While this is practical for upgrades between minor versions, there can be larger database changes that prevent this being done in a single step, such as the changes to the authentication system in 7.3.X. 
- upgrade to 6, then 7.3, then destination version
- Version 7 is currently broken for winforms templates