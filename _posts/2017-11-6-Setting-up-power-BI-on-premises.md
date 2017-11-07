---
layout: post
title: Setting up Power BI On-premises Report Server
categories: powerbi
---

Power BI is a cloud hosted reporting and business analytics platform created by Microsoft to allow users to easily manipulate and report on data from a large range of supported sources. While it does provide an interface to retrieve data from on premises database installation, not all companies are comfortable using cloud based systems as part of their day to day business. Microsoft now provide an on premises version of the Power BI report server for companies to host internally as a bridging solution that allows customers to develop reporting that is cloud compatible, without having to commit to a full migration immediately.

# Requirements
- .Net 4.6
- Microsoft SQL Server 2008+
- Windows 8+ or Windows Server 2012+

[Full Power BI Server Requirements](https://powerbi.microsoft.com/en-us/documentation/reportserver-system-requirements/)

# Installation
- Download the [Power BI Report Server installer](https://aka.ms/pbireportserver)
- Run the installer on the target server
- After installation, you will be presented with the server configuration screen. If you've closed this before completing configuration, it can be found under `Microsoft Power BI Report Server\Report Server Configuration Manager` in the start bar.
- Go through and configure the "Web Service URL", "Web Portal URL" and "Database" sections in the configuration manager. If you miss one of these, the report server will not work, and will give unhelpful error messages when you try to access it. By default the portal and service are setup on port 80, which will conflict with an existing SSRS installation with the same URLs, and may conflict with IIS.
![Configuration Options]({{"/Images/PowerBIReportServer/ReportServerConfigurationManager.png"}})
- After setup, the Power BI Web Portal can be viewed at `http://localhost/Reports/browse/`

[Official Installation Instructions](https://powerbi.microsoft.com/en-us/documentation/reportserver-quickstart-install-report-server/)

# Deploying reports to On-Premises Power BI
On-Premises Power BI does not provide the same quality of report publishing experience that is provided for the cloud service. 
The Power BI Desktop client can open existing reports, then edit and save them back to the server, but there is no way to publish new reports to a custom server from, requiring them to be uploaded through the web portal.

# Pricing of On-Premises Power BI 

The On Premises Power BI licence is only available as part of "Power BI Premium", which provides dedicated cloud resources for Power BI. This arrangement is intended to transition customers towards using the cloud based service. The base price for Power BI Premium is $6,350 per month, with additional monthly costs per user. The high cost of On Premises Power BI means that this may not be the an appealing option for companies wanting to take advantage of Power BIs features, but do not plan to move to the cloud.

[Power BI Pricing calculator](https://powerbi.microsoft.com/en-us/calculator/)

