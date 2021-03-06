---
title: "在 Azure 中建立 Windows 的虛擬機器擴展集 | Microsoft Docs"
description: "在 Windows VM 上使用虛擬機器擴展集，建立及部署高可用性應用程式"
services: virtual-machine-scale-sets
documentationcenter: 
author: iainfoulds
manager: timlt
editor: 
tags: 
ms.assetid: 
ms.service: virtual-machine-scale-sets
ms.workload: infrastructure-services
ms.tgt_pltfrm: na
ms.devlang: 
ms.topic: article
ms.date: 05/01/2017
ms.author: iainfou
ms.translationtype: Human Translation
ms.sourcegitcommit: be3ac7755934bca00190db6e21b6527c91a77ec2
ms.openlocfilehash: bbd4f044d85f2e22f27edc44b91fd42aef304ed2
ms.contentlocale: zh-tw
ms.lasthandoff: 05/03/2017

---

# <a name="create-a-virtual-machine-scale-set-and-deploy-a-highly-available-app-on-windows"></a>在 Windows 上建立虛擬機器擴展集及部署高可用性應用程式
在本教學課程中，您會了解 Azure 中的虛擬機器擴展集如何讓您快速地擴展執行您應用程式的虛擬機器 (VM) 數目。 虛擬機器擴展集可讓您部署和管理一組相同、自動調整的虛擬機器。 您可以手動調整擴展集中的 VM 數目，或定義規則以根據 CPU 使用量、記憶體需求或網路流量自動調整。 若要查看作用中的虛擬機器擴展集，您可建置在多部 Windows VM 上執行的基本 IIS 網站。

您可以使用最新的 [Azure PowerShell](/powershell/azureps-cmdlets-docs/) 模組來完成本教學課程中的步驟。


## <a name="scale-set-overview"></a>擴展集概觀
擴展集使用的元件，與您在[建立高可用性 VM](tutorial-availability-sets.md) 的先前教學課程中所學習到的元件類似。 擴展集中的 VM 會分散於容錯網域和更新網域，如同可用性設定組中的 VM。

VM 會視需要建立於擴展集中。 您將定義自動調整規則，以控制如何及何時新增或移除擴展集中的 VM。 您可以根據計量 (例如 CPU 負載、記憶體使用量或網路流量) 觸發這些規則。

當您使用 Azure 平台映像時，擴展集可支援多達 1,000 部 VM。 對於包含重要安裝作業或 VM 自訂要求的工作負載，您可能想要[建立自訂的 VM 映像](tutorial-custom-images.md)。 使用自訂映像時，您可以在擴展集中建立多達 100 部 VM。


## <a name="create-an-app-to-scale"></a>建立要調整的應用程式
建立擴展集之前，請先使用 [New-AzureRmResourceGroup](/powershell/module/azurerm.resources/new-azurermresourcegroup) 來建立資源群組。 下列範例會在 westus 位置建立名為 myResourceGroupAutomate 的資源群組：

```powershell
New-AzureRmResourceGroup -ResourceGroupName myResourceGroupScaleSet -Location westus
```

在先前的教學課程中，您已了解如何使用自訂指令碼擴充功能來[自動化 VM 組態](tutorial-automate-vm-deployment.md)。 建立擴展集組態，然後套用自訂指令碼擴充功能來安裝及設定 IIS：

```powershell
# Create a config object
$vmssConfig = New-AzureRmVmssConfig `
    -Location WestUS `
    -SkuCapacity 2 `
    -SkuName Standard_DS2 `
    -UpgradePolicyMode Automatic

# Define the script for your Custom Script Extension to run
$publicSettings = @{
    "fileUris" = (,"https://raw.githubusercontent.com/iainfoulds/azure-samples/master/automate-iis.ps1");
    "commandToExecute" = "powershell -ExecutionPolicy Unrestricted -File automate-iis.ps1"
}

# Use Custom Script Extension to install IIS and configure basic website
Add-AzureRmVmssExtension -VirtualMachineScaleSet $vmssConfig `
    -Name "customScript" `
    -Publisher "Microsoft.Compute" `
    -Type "CustomScriptExtension" `
    -TypeHandlerVersion 1.8 `
    -Setting $publicSettings
```

## <a name="create-scale-load-balancer"></a>建立擴展負載平衡器
Azure Load Balancer 是 Layer-4 (TCP、UDP) 負載平衡器，可將連入流量分散於狀況良好的 VM 來提供高可用性。 負載平衡器健康狀態探查會監視每部 VM 上指定的連接埠，且只會將流量分散至作業 VM。 如需詳細資訊，請參閱下一個教學課程的[如何平衡 Windows 虛擬機器的負載](tutorial-load-balancer.md)。

建立負載平衡器，包含公用 IP 位址及並分散連接埠 80 上的網路流量︰

```powershell
# Create a public IP address
$publicIP = New-AzureRmPublicIpAddress `
  -ResourceGroupName myResourceGroupScaleSet `
  -Location westus `
  -AllocationMethod Static `
  -Name myPublicIP

# Create a frontend and backend IP pool
$frontendIP = New-AzureRmLoadBalancerFrontendIpConfig `
  -Name myFrontEndPool `
  -PublicIpAddress $publicIP
$backendPool = New-AzureRmLoadBalancerBackendAddressPoolConfig -Name myBackEndPool

# Create the load balancer
$lb = New-AzureRmLoadBalancer `
  -ResourceGroupName myResourceGroupScaleSet `
  -Name myLoadBalancer `
  -Location westus `
  -FrontendIpConfiguration $frontendIP `
  -BackendAddressPool $backendPool

# Create a load balancer health probe on port 80
Add-AzureRmLoadBalancerProbeConfig -Name myHealthProbe `
  -LoadBalancer $lb `
  -Protocol tcp `
  -Port 80 `
  -IntervalInSeconds 15 `
  -ProbeCount 2

# Create a load balancer rule to distribute traffic on port 80
Add-AzureRmLoadBalancerRuleConfig `
  -Name myLoadBalancerRule `
  -LoadBalancer $lb `
  -FrontendIpConfiguration $lb.FrontendIpConfigurations[0] `
  -BackendAddressPool $lb.BackendAddressPools[0] `
  -Protocol Tcp `
  -FrontendPort 80 `
  -BackendPort 80

# Update the load balancer configuration
Set-AzureRmLoadBalancer -LoadBalancer $lb
```

## <a name="create-a-scale-set"></a>建立擴展集
現在使用 [New-AzureRmVmss](/powershell/module/azurerm.compute/new-azurermvm) 建立虛擬機器擴展集。 下列範例會建立名為 myScaleSet 的擴展集：

```powershell
# Reference a virtual machine image from the gallery
Set-AzureRmVmssStorageProfile $vmssConfig `
  -ImageReferencePublisher MicrosoftWindowsServer `
  -ImageReferenceOffer WindowsServer `
  -ImageReferenceSku 2016-Datacenter `
  -ImageReferenceVersion latest

# Set up information for authenticating with the virtual machine
Set-AzureRmVmssOsProfile $vmssConfig `
  -AdminUsername azureuser `
  -AdminPassword P@ssword! `
  -ComputerNamePrefix myVM

# Create the virtual network resources
$subnet = New-AzureRmVirtualNetworkSubnetConfig `
  -Name "mySubnet" `
  -AddressPrefix 10.0.0.0/24
$vnet = New-AzureRmVirtualNetwork `
  -ResourceGroupName "myResourceGroupScaleSet" `
  -Name "myVnet" `
  -Location "westus" `
  -AddressPrefix 10.0.0.0/16 `
  -Subnet $subnet
$ipConfig = New-AzureRmVmssIpConfig `
  -Name "myIPConfig" `
  -LoadBalancerBackendAddressPoolsId $lb.BackendAddressPools[0].Id `
  -SubnetId $vnet.Subnets[0].Id

# Attach the virtual network to the config object
Add-AzureRmVmssNetworkInterfaceConfiguration `
  -VirtualMachineScaleSet $vmssConfig `
  -Name "network-config" `
  -Primary $true `
  -IPConfiguration $ipConfig

# Create the scale set with the config object (this step might take a few minutes)
New-AzureRmVmss `
  -ResourceGroupName myResourceGroupScaleSet `
  -Name myScaleSet `
  -VirtualMachineScaleSet $vmssConfig     
```

建立及設定所有擴展集資源和 VM 需要幾分鐘的時間。


## <a name="test-your-app"></a>測試應用程式
若要查看作用中的 IIS 網站，可使用 [Get-AzureRmPublicIPAddress](/powershell/module/azurerm.network/get-azurermpublicipaddress)取得負載平衡器的公用 IP 位址。 下列範例會取得建立作為擴展集一部分的 myPublicIP IP 位址︰

```powershell
Get-AzureRmPublicIPAddress -ResourceGroupName myResourceGroupScaleSet -Name myPublicIP | select IpAddress
```

在 Web 瀏覽器中輸入公用 IP 位址。 應用程式隨即顯示，包括負載平衡器分散流量之 VM 的主機名稱︰

![執行中的 IIS 網站](./media/tutorial-create-vmss/running-iis-site.png)

若要查看作用中的擴展集，請強制重新整理您的 Web 瀏覽器，以查看負載平衡器如何將流量分散於執行應用程式的所有 VM。


## <a name="management-tasks"></a>管理工作
在擴展集生命週期中，您可能需要執行一或多個管理工作。 此外，您可以建立指令碼來自動化各種生命週期工作。 Azure PowerShell 提供快速的方式來執行這些工作。 以下是一些常見工作。

### <a name="view-vms-in-a-scale-set"></a>檢視擴展集中的 VM
若要檢視在擴展集中執行的 VM 清單，請使用 [Get-AzureRmVmssVM](/powershell/module/azurerm.compute/get-azurermvmssvm)，如下所示︰

```powershell
# Get current scale set
$scaleset = Get-AzureRmVmss `
  -ResourceGroupName myResourceGroupScaleSet `
  -VMScaleSetName myScaleSet

# Loop through the instanaces in your scale set
for ($i=0; $i -le ($set.Sku.Capacity - 1); $i++) {
    Get-AzureRmVmssVM -ResourceGroupName myResourceGroupScaleSet `
      -VMScaleSetName myScaleSet `
      -InstanceId $i
}
```


### <a name="increase-or-decrease-vm-instances"></a>增加或減少 VM 執行個體
若要查看擴展集中目前擁有的執行個體數目，請使用 [Get-AzureRmVmss](/powershell/module/azurerm.compute/get-azurermvmss)並查詢 sku.capacity：

```powershell
Get-AzureRmVmss -ResourceGroupName myResourceGroupScaleSet `
    -VMScaleSetName myScaleSet | `
    Select -ExpandProperty Sku
```

然後，您可以使用 [Update-AzureRmVmss](/powershell/module/azurerm.compute/update-azurermvmss)，手動增加或減少擴展集中的虛擬機器數目。 下列範例會將擴展集中的 VM 數目設定為 5：

```powershell
# Get current scale set
$scaleset = Get-AzureRmVmss `
  -ResourceGroupName myResourceGroupScaleSet `
  -VMScaleSetName myScaleSet

# Set and update the capacity of your scale set
$scaleset.sku.capacity = 5
Update-AzureRmVmss -ResourceGroupName myResourceGroupScaleSet `
    -Name myScaleSet `
    -VirtualMachineScaleSet $scaleset
```

如果更新擴展集中指定的執行個體數目需費時幾分鐘。


### <a name="configure-autoscale-rules"></a>設定自動調整規則
除了手動調整擴展集中的執行個體數目，您可以定義自動調整規則。 這些規則會監視擴展集中的執行個體，並根據您定義的計量和臨界值進行回應。 下列範例會示範當 CPU 平均負載大於 60% 並持續 5 分鐘以上時，如何增加一個執行個體來相應放大執行個體數目。 如果之後 CPU 平均負載降到低於 30% 且持續 5 分鐘以上，則減少一個執行個體來相應縮小執行個體數目︰

```powershell
# Define your scale set information
$mySubscriptionId = (Get-AzureRmSubscription).SubscriptionId
$myResourceGroup = "myResourceGroupScaleSet"
$myScaleSet = "myScaleSet"
$myLocation = "West US"

# Create a scale up rule to increase the number instances after 60% average CPU usage exceeded for a 5 minute period
$myRuleScaleUp = New-AzureRmAutoscaleRule `
  -MetricName "Percentage CPU" `
  -MetricResourceId /subscriptions/$mySubscriptionId/resourceGroups/$myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/$myScaleSet `
  -Operator GreaterThan `
  -MetricStatistic Average `
  -Threshold 60 `
  -TimeGrain 00:01:00 `
  -TimeWindow 00:05:00 `
  -ScaleActionCooldown 00:05:00 `
  -ScaleActionDirection Increase `
  -ScaleActionValue 1

# Create a scale down rule to decrease the number of instances after 30% average CPU usage over a 5 minute period
$myRuleScaleDown = New-AzureRmAutoscaleRule `
  -MetricName "Percentage CPU" `
  -MetricResourceId /subscriptions/$mySubscriptionId/resourceGroups/$myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/$myScaleSet `
  -Operator LessThan `
  -MetricStatistic Average `
  -Threshold 30 `
  -TimeGrain 00:01:00 `
  -TimeWindow 00:05:00 `
  -ScaleActionCooldown 00:05:00 `
  -ScaleActionDirection Decrease `
  -ScaleActionValue 1

# Create a scale profile with your scale up and scale down rules
$myScaleProfile = New-AzureRmAutoscaleProfile `
  -DefaultCapacity 2  `
  -MaximumCapacity 10 `
  -MinimumCapacity 2 `
  -Rules $myRuleScaleUp,$myRuleScaleDown `
  -Name "autoprofile"

# Apply the autoscale rules
Add-AzureRmAutoscaleSetting `
  -Location $myLocation `
  -Name "autosetting" `
  -ResourceGroup $myResourceGroup `
  -TargetResourceId /subscriptions/$mySubscriptionId/resourceGroups/$myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/$myScaleSet `
  -AutoscaleProfiles $myScaleProfile
```


## <a name="next-steps"></a>後續步驟
在本教學課程中，您已了解如何建立虛擬機器擴展集。 請前進到下一個教學課程，以深入了解虛擬機器的負載平衡概念。

[平衡虛擬機器的負載](tutorial-load-balancer.md)
