---
title: "Changing the size of an Azure VM using Azure PowerShell"
date: "2019-06-18"
categories: 
  - "technology"
tags: 
  - "azure"
  - "powershell"
  - "sysadmin"

socialshare: true
---

A few weeks ago, I wrote about how to connect to Azure via PowerShell and a service principal. It's super useful, and allows us to do things like change the size of a VM.

https://www.stephenwuebker.com/2019/05/30/connecting-to-azure-using-azure-powershell-and-a-service-principal/

In our environment, we need to toggle a VM between sizes on a fairly regular basis. Why? Well, because for some weeks of the month, the workload on that VM is high and we need the performance of a more powerful VM. However, during the rest of the month, the workload is relatively low and we don't need as many resources, so we'll reduce the size to save money.

The main benefit of changing VM properties via a PowerShell script is that it can be scheduled to run overnight outside of business hours. Before I wrote this, I would have to log into the Azure portal late in the evening and make the change by hand. Nobody likes that.

This script is setup like a toggle switch. Run it and if the VM is in it's larger size, it will update it to be the smaller size and vice-versa. The first thing to do is connect to Azure using your service principal.

```powershell
Connect-AzAccount -Credential $psCredential -ServicePrincipal -Tenant $tenantID -Subscription $subscriptionID
```

Ok, now you are connected and we can go ahead with the update. Call Get-AzVM to get your VM. Then figure out what the size is currently. Then, we can compare to the up and down sizes we want to use, and set that as the new size. Finally, call Update-AzVM to save your changes and reboot the VM using your new size.

```powershell
$resourceGroup = "<Your Resource Group>"
$vmName = "<VM Name>"
$upSize = "Standard_DS4_v2"
$downSize = "Standard_DS3_v2"
$vm = Get-AzVM -ResourceGroupName $resourceGroup -VMName $vmName
$curSize = $vm.HardwareProfile.VmSize

if ($curSize=$upSize)
{ 
    $vm.HardwareProfile.VmSize = $downSize
}
elseif ($curSize=$downSize)
{
    $vm.HardwareProfile.VmSize = $upSize
}

Update-AzVM -VM $vm -ResourceGroupName $resourceGroup
```
