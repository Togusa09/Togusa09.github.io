---
layout: post
title: Useful practices
---

# Tools
## OneNote
Allows saving notes and sharing them between different platforms. It has mobile, windows and web apps and is accessible through onedrive. It also supports drawing on mobile devices, tables and touch screens.

## Zendesk
Provides a single place to handle customer communication through. It allows threads of communication with clients to be managed through a single interface, allowing people to be added or removed from the discussion at any time. Private notes can be used for distributing messages to other members of the team without the client seeing them. This is for keeping technical or background discussion noise away from the client. If you're sending emails straight to a customer, you're probably doing it wrong.

## LinqPad
Scratchpad for c#. It is useful for developing one off scripts, or scripting scenarios to load into environments. Dlls and nuget packages can be imported to interact with existing codebases.

## Google Chrome
While choice of browsers is always a topic of debate, Google Chrome has multiprofile support simplifies managing different login sessions that would normally conflict (ie. Logging in multiple Microsoft accounts). These sessions can be saved and reactivated when the profile is resumed, so you don't have to re-enter the credentials if the login hasn't expired. 

## Tenrox
More of a necessary item than a useful tool, tenrox is a timesheeting system for tracking hours worked. Time entered here feeds into the Support Contract Management System.

## LastPass
Password manager with integration with multiple browsers. Includes a strong password generator and removes most excuses for using weak or repeated passwords.

# Readify Tools

## Support Contract Management
Also known as "Blobfish" for an unknown reason. This is an internal Readify product that allows tracking of the contract time used for each client as the month goes on. Watching this is important if you have a client that has less than 20 hours, or often goes over hours so you can avoid overage. UI Can be configured to only show relevant clients. Automatically imports data from Tenrox to calculate the remaining hours.

Is there a user guide for this anywhere?

## RediUpdates
Readify portal for compiling and distributing weekly summaries. It provides tracking of what was done that week, what will be done next week, any upcoming opportunities and ongoing issues. There is currently no "Save" button on the site, but hitting "next" or "back" will save what you have entered.

# Practices

The following are practices that I've found useful 

* Keep notes through the day: Having a record of what you did, issues encountered or discussions you had with people is invaluable. When switching clients it can be hard to remember the details of what you did that morning, let alone the previous day.
* Record Timesheets at end of day, or when switching customer: Timesheets can be a pain and easy to forget, especially when you just want to get home on a Friday. By making it a habit to fill them in every day, it's one less thing to remember, and you don't have to spend time going back through notes to check what you were doing Monday morning.
* Update Rediupdates at end of day. There is always a chance you won't be on the customer tomorrow. 
* At the start of the day, send the client list of items you plan to complete, and raise any issues that might hinder you
* Send updates through the day as you being or are blocked on tasks. Whenever you do something, think about who is interested in the task, and how it affects them. This can range between completing a task or encountering an obstruction through to making a production deployment.
* Send to client a summary of tasks completed at the end of the day: There will be some overlap between this and the rediupdates notes, but this summary is focused on answering any questions the client might have. This also serves as a reminder of any work you require the client to complete before your next work day.
* Update ticket notes with new information you encounter as you work on a task. This is also useful when someone else has to take over a partly completed work item.
* Link commits and branches to tickets: VSTS supports linking of branches and commits to a work item. This can add far more context to the changes that the commit messages do.