---
title: "Connecting to Azure using Azure PowerShell and a service principal"
date: "2019-05-30"
categories: 
  - "technology"
tags: 
  - "azure"
  - "powershell"
  - "sysadmin"

socialshare: true
---

Azure PowerShell provides a cross-platform way to manage your Azure resources without actually having to log in to the portal. Obvious benefits here include being able to script away management tasks and speed up administration tasks by using the command line. Microsoft provides some great documentation and installation instructions here:

[https://docs.microsoft.com/en-us/powershell/azure/overview?view=azps-2.1.0](https://docs.microsoft.com/en-us/powershell/azure/overview?view=azps-2.1.0)

Before getting started, make sure you are connected to Azure.

```powershell
Connect-AzAccount
```

This cmdlet outputs a URL and token to enter. Basically, just go to [https://microsoft.com/devicelogin](https://microsoft.com/devicelogin) and enter the code shown. You will then be connected to Azure. Make sure your account has permission to create a service principal. If you don't have permission, you will need to get your Azure Active Directory admin to create it for you.

The first thing we need to do is create the service principal. An Azure service principal is a special identity to use with your automated tools, applications, and hosted services. Service principals allow you to control which Azure resources can be accessed and doesn't require logging in with a full user identity. Creating the service principal is easy with the New-AzADServicePrincipal cmdlet.

```powershell
$sp = New-AzADServicePrincipal -DisplayName ServicePrincipalName
```

This creates the service principal and returns back the object so we can copy out the various bits of information we need to add to our script. The problem here is that Microsoft, it it's documentation wisdom, makes sure to note that the returned object contains the member named `Secret`, which is a `SecureString` with the password that was generated. We need this password. However, the value is not displayed anywhere in the console output.

We can get the plain text of the password, however, like this:

```powershell
$pass = (New-Object PSCredential $sp.ID,$sp.Secret).GetNetworkCredential().Password
```

One more thing we have to setup before we can log in is the role for the new service principal. Because I'm planning on using this service principal to update things, I need to give it the **`Contributor`** role. If you only need to be able to read information, you should use the **`Reader`** role.

```powershell
New-AzRoleAssignment -ApplicationId $sp.ApplicationId -RoleDefinitionName "Contributor"
```

After this, we are ready to connect using our new service principal account. Open a new PowerShell terminal and use Connect-AzAccount again, but pass it the needed options. I'm referencing previously created objects in the sample code below ($sp, $pass), but you'll probably want to get those IDs and copy them into your PowerShell script. The values will look like "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"

```powershell
$servicePrincipalID = $sp.ID
$servicePrincipalAppID = $sp.ApplicationID
$servicePrincipalPassID = $pass
$tenantID = "<YourAzureTenantID>"
$subscriptionID = "<YourAzureSubscriptionID>"

$SecurePassword = ConvertTo-SecureString $servicePrincipalPassID -AsPlainText -Force
$psCredential = New-Object System.Management.Automation.PSCredential ($servicePrincipalAppID, $SecurePassword)

Connect-AzAccount -Credential $psCredential -ServicePrincipal -Tenant $tenantID -Subscription $subscriptionID
```

Now you are connected to Azure and can do all sorts of fun things with your resources, like automate changing the size of a VM, which I'll discuss in a future post...
