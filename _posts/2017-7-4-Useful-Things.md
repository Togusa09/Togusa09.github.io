---
layout: post
title: Useful Practices
---

# Tools
## OneNote
Allows saving notes and sharing them between different platforms. It has mobile, windows and web apps and is accessible through onedrive. It also supports drawing on mobile devices, tables and touch screens.

## Zendesk
Provides a single place to handle customer communication through. It allows threads of communication with clients to be managed through a single interface, allowing people to be added or removed from the discussion at any time. Private notes can be used for distributing messages to other members of the team without the client seeing them. This is for keeping technical or background discussion noise away from the client. If you're sending emails straight to a customer, you're probably doing it wrong.

## LinqPad
Scratchpad for c#. It is useful for developing one off scripts, or scripting scenarios to load into environments. Dlls and nuget packages can be imported to interact with existing codebases.

## Google Chrome
While choice of browsers is always a topic of debate, Google Chrome has support for user profiles, which simplifies managing different login sessions that would normally conflict (ie. Logging in multiple Microsoft accounts). These sessions can be saved and reactivated when the profile is resumed, so you don't have to re-enter the credentials if the login hasn't expired. 

Chrome also supports debugging of Cordova apps running under Android. This can be accessed at chrome://inspect/. You can also map ports on your dev machine to localhost ports on the android device, which removes the need for messy bindings when hosting in iisexpress.

## LastPass
Password manager with integration with multiple browsers. Includes a strong password generator and removes most excuses for using weak or repeated passwords.

## mRemoteNG
mRemoteNG is a free client for managing remote connections to computers. It supports a variety of protocols, including RDP and VNC. The client allows import/export of connection lists, which allows them to be easily passed to another user. It is also possible to set inherited properties, so that the same username and password can be shared between multiple connections. 4K screens on the client 

# Practices

The following are practices that can be useful 

* Keep notes through the day: Having a record of what you did, issues encountered or discussions you had with people is invaluable. When switching clients it can be hard to remember the details of what you did that morning, let alone the previous day.
* Record Timesheets at end of day, or when switching customer: Timesheets can be a pain and easy to forget, especially when you just want to get home on a Friday. By making it a habit to fill them in every day, it's one less thing to remember, and you don't have to spend time going back through notes to check what you were doing Monday morning.
* At the start of the day, send the client list of items you plan to complete, and raise any issues that might hinder you
* Send updates through the day as you being or are blocked on tasks. Whenever you do something, think about who is interested in the task, and how it affects them. This can range between completing a task or encountering an obstruction through to making a production deployment.
* Send to client a summary of tasks completed at the end of the day: There will be some overlap between this and the rediupdates notes, but this summary is focused on answering any questions the client might have. This also serves as a reminder of any work you require the client to complete before your next work day.
* Update ticket notes with new information you encounter as you work on a task. This is also useful when someone else has to take over a partly completed work item.
* Link commits and branches to tickets: VSTS supports linking of branches and commits to a work item. This can add far more context to the changes that the commit messages do.
* Break down work items into individual tasks. This is supported as a feature in VSTS, and allows you to check the off when completed. This is also useful for tasks that need to be repeated in multiple environments, so you don't lose track of progress.