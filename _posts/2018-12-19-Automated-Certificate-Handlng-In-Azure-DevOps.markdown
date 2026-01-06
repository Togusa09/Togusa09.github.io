---
layout: post
title: Automated certificate handling in Azure DevOps
categories: AzureDevops
---

While Azure services have quite good support for certificate handling these days, on premises deployments using Azure Devops dont provide much in the way of certificate handling. To work around this, I've scripted up a solution that retrives a certicate, installs it on the host and then puts the certificate thumbprint into a variable for use later in the build or release pipeline.

The certificates are stored securely in the [secure files library](https://projectname.visualstudio.com/SAGE/_library?itemType=SecureFiles) which provides a neater alternative storing in the git repository. ([Additional Information on secure files](https://docs.microsoft.com/en-us/azure/devops/pipelines/library/secure-files?view=vsts))

The task performs the following actions:

- Download secure file to server by name using the [Download Secure Files Task](https://docs.microsoft.com/en-us/azure/devops/pipelines/tasks/utility/download-secure-file?view=vsts). This downloaded the file to a location stored in `$(Agent.TempDirectory)`
- Load certificate in powershell and retrieve certificate thumbnail, and assign to specified return variable
- Use password variable to install certificate into the specified local store

Script variables
`cert.name` - name of certificate file  
`cert.Passphrase` - password for certificate  
`cert.Store` - name of the certificate store to place the certificate. ie. `Cert:\LocalMachine\My` or `Cert:\LocalMachine\TrustedPeople`  
`cert.thumbprintVariableName` - The output variable to store the certificate thumbprint in

To make the script reusable, it needs to be placed in a task group, which will then expose the variables as input properties. If the script is included as a powershell step straight into the pipeline, it will use pipeline varaibles directly limiting to a single certificate.

Powershell script:
{% highlight powershell %}

Write-Host "Starting certificate deployment"

$certDir = "$(Agent.TempDirectory)"
$certName = $(cert.Name)
$certPassphrase = $(cert.Passphrase)
$certStore = $(cert.store)
$thumbprintVarName = $(cert.thumbprintVariableName)

$certPath = Join-Path $certDir $certName

Write-Host "Importing cert from $certPath into store $certStore"

Import-PfxCertificate -FilePath "$certPath" -CertStoreLocation "$certStore"  -Password $certPassphrase

$certificateObject = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2
$certificateObject.Import($certPath, $certPassphrase, [System.Security.Cryptography.X509Certificates.X509KeyStorageFlags]::DefaultKeySet)
$thumbprint = $certificateObject.Thumbprint

Write-Host "##vso[task.setvariable variable=$thumbprintVarName;]$thumbprint"

{% endhighlight %}