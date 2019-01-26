---
layout: post
title:  "Azure Policy"
categories: Azure Policy Resource Security
author: Marcin Krawczyk
---

# How to write custom Azure Policy
Last update: 2019.01.26

In the Internet you can find a lot of Azure Policy scripts, but there always may be a situation when you need to create your own custom policy. So I decided to write this article about it. 

<b>PLEASE DON’T USE DENY POLICY WHEN YOU ARE CREATING FIRST POLICY. USE AUDIT!!!</b>

It is written in the JSON format. You have to know how to build section, what kind of field you can use (unfortunately not all that you would like to). At the time of writing this article there is about 948 custom fields you can use to build custom policy. You can use Azure Policy directly for those services:<br>

<ul>
<li>Analysis Services</li>
<li>App Service</li>
<li>Application Gateway</li>
<li>Azure Automation</li>
<li>Azure Locks</li>
<li>Batch</li>
<li>CDN</li>
<li>Cosmos DB</li>
<li>Data Factory</li>
<li>Data Lake Store</li>
<li>Disks</li>
<li>Express Route</li>
<li>Guest Configuration Assignments</li>
<li>HDInsight</li>
<li>Insights</li>
<li>IoT Hubs</li>
<li>Key Vault</li>
<li>Load Balancer</li>
<li>Logic Apps</li>
<li>Network Interface</li>
<li>Network Security Group</li>
<li>Notification Hubs</li>
<li>Operational Insights</li>
<li>Public IP</li>
<li>Redis Cache</li>
<li>Role Assignments</li>
<li>Role Definitions</li>
<li>Scheduler</li>
<li>Search</li>
<li>Security</li>
<li>Service Fabric</li>
<li>SQL</li>
<li>Storage Account</li>
<li>Terraform OSS</li>
<li>Traffic Manager</li>
<li>Virtual Machine</li>
<li>Virtual Machine Extension</li>
<li>Virtual Machine Scale Sets</li>
<li>Virtual Machine Scale Sets Extensions</li>
<li>Virtual Network</li>
<li>Virtual Network Gateway</li>

</ul>
There is also a possibility to write custom Policy for different resource types, like Event Hub, where you are not able to use dedicated properties for some properties.

# How to begin?
First of all you have to choose JSON editor or write custom policy in Azure portal. If you will be writing custom policy in Azure portal, then you will get autocomplete fields for some value like aliases (later about that). If you would like to built custom policy in JSON editor, then you can choose Visual Studio or Visual Studio Code. Here I’ll work on Visual Studio 2017 with Azure SDK installed. For the beginning I’ll create new JSON file:

<b>Select File -> New -> File -> Web -> JSON File</b>

Then you should insert schema definition for Azure Policy into Visual Studio. The newest version for policy definition is 2018-05-01. 


[https://schema.management.azure.com/schemas/2018-05-01/policyDefinition.json](https://schema.management.azure.com/schemas/2018-05-01/policyDefinition.json)





![AzurePolicy](https://mkpkmkpk.github.io/Images/Azure-Policy/AzurePolicy01.png "AzurePolicy")



So when you do that, you can easily write custom policy with autocomplete options 😊

![AzurePolicy](https://mkpkmkpk.github.io/Images/Azure-Policy/AzurePolicy02.png "AzurePolicy")

# Azure Policy definition structure
When you create a custom policy you have to decide how you implemented them. You have couple of possibility:
<ul>
<li>Azure Portal</li>
<li>PowerShell</li>
<li>Azure CLI</li>
</ul>

There is a difference between them. When you are using Azure Portal, then you don’t need to add couple policy properties like displayName, description and so on. Below you can find sample policy which ensure that only https traffic is enable for storage account. This policy is created for PowerShell and CLI:


```

{
    "properties": {
        "mode": "All",
        "parameters": {},
        "displayName": "Ensure https traffic only for storage account",
        "description": "Ensure https traffic only for storage account",
        "policyRule": {
            "if": {
                "allOf": [
                    {
                        "field": "type",
                        "equals": "Microsoft.Storage/storageAccounts"                    },
                    {
                        "not": {
                            "field": "Microsoft.Storage/storageAccounts/supportsHttpsTrafficOnly",
                            "equals": "true"
                        }
                    }
                ]
            },
            "then": {
                "effect": "deny"
            }
        }
    }
}

```

The same script for Azure Portal deployment will look like this:

```

{
  "mode": "All",
  "policyRule": {
    "if": {
      "allOf": [
        {
          "field": "type",
          "equals": "Microsoft.Storage/storageAccounts"
        },
        {
          "not": {
            "field": "Microsoft.Storage/storageAccounts/supportsHttpsTrafficOnly",
            "equals": "True"
          }
        }
      ]
    },
    "then": {
      "effect": "deny"
    }
  }
}
```

For the purpose of this article I’ll use Azure Portal method of creation.

So as you can see, there are some policy definitions contains such elements like:
<ul>
<li>Mode</li>
<li>Parameters</li>
<li>Display Name</li>
<li>Description</li>
<li>Policy rule</li>
<li>Logical evaluation</li>
<li>Effect</li>
</ul>


## Mode

There are two possibly value for Mode field:
<ul>
<li>All</li>
<li>Indexed</li>
</ul>

I strongly recommend to set mode to All in most cases. There is only one situation, when you create a policy for tags and location in which you would like to use indexed mode.  Be careful when you copy Azure Policy from Internet. There are quite many situations when Mode is set to indexed!!! This may also happened when you use default definition in Azure Portal (see Mode value below)!!!! The difference between these two modes is that the indexed mode influences only object which we can assign Tags. The ALL mode influences all the objects. So when you creat your own policies always use ALL mode. For built-in policies from portal it is best to duplicate them and set the ALL mode.  



![AzurePolicy](https://mkpkmkpk.github.io/Images/Azure-Policy/AzurePolicy03.png "AzurePolicy")

![AzurePolicy](https://mkpkmkpk.github.io/Images/Azure-Policy/AzurePolicy04.png "AzurePolicy")


<b>IMPORTANT:

When you not specify mode parameter manually, then this value would be directly assign by Azure. Automatically assign value is different when you are using Powershell and CLI. Below you can find default value for them:
<ul>
<li>Powershell – By default Mode is set to ALL</li>
<li>Azure CLI – By default Mode is set to NULL, which is equivalent to indexed!!! Please change it.</li>
<li>Azure Portal - By default Mode is set to ALL</li>
<li>Created By Microsoft Built-in – A huge number of ready to use scripts has MODE set to INDEXED!!!!!!</b></li>
</ul>


### Parameters
You can use Parameters when you would like to choose a value from a pull-down list or to manually write a different value for a specific Resource Group. For instance you would like to choose Allowed location or Allowed Region in Azure, then you create Parameters, where after assigned to policy you choose exact value from Azure. There are ONLY some types of Parameters which you can use:
<ul>
<li>"location"</li>
<li>"resourceTypes”</li>
<li>"storageSkus”</li>
<li>"vmSKUs”</li>
<li>"existingResourceGroups”</li>
<li>"omsWorkspace”</li>

</ul>
Below you can find an example for Parameters:


```

"parameters": {
    "allowedLocations": {
        "type": "array",
        "metadata": {
            "description": "The list of allowed locations for resources.",
            "displayName": "Allowed locations",
            "strongType": "location"
        }
    }
}

```

### Display Name
Used only when you create Azure Policy in Powershell or Azure CLI. It’s a name of a policy.

### Description
Used only when you create Azure Policy in Powershell or Azure CLI. It’s a description of apolicy.

### Policy rule
Policy rule consists If and Then section. Policy rules look like that:

```
{
    "if": {
        <condition> | <logical operator>
    },
    "then": {
        "effect": "deny | audit | append | auditIfNotExists | deployIfNotExists"
    }
}
```

You may use couple of logical operators like:
<ul>
<li>"not”:{}</li>
<li>"allOf”:[{},{}] – working like logical AND</li>
<li>"anyOf”:[{},{}] – working like logical OR</li>
</ul>

A Conditions are used for field criteria. Supported conditions are:
<ul>
<li>contains</li>
<li>containsKeys</li>
<li>equals</li>
<li>exists</li>
<li>in</li>
<li>like</li>
<li>match</li>
<li>notContains</li>
<li>notContainsKeys</li>
<li>notEquals</li>
<li>notIn</li>
<li>notLike</li>
<li>notMatch</li>
</ul>

When you are using Like and notLike operators, you can use * in the value. There may be ONLY one wildcard * in criteria.

When you are using march and notMatch, you can use:
<ul>
<li># - represent a digit</li>
<li>? – represent a letter</li>
<li>. – match all characters</li>
</ul>


### Fields

Fields represent properties for resource you choose. Only those types of fields are supported:
<ul>
<li>aliases</li>
<li>name</li>
<li>fullName</li>
<li>kind</li>
<li>type</li>
<li>location</li>
<li>tags</li>
<li>source – with ACTION – UNDOCUMENTED 😊 – You can use AUDIT or DENY  to a certain action on Azure Portal😊 Whole list of actions you can find here:  <a href="https://docs.microsoft.com/en-us/azure/role-based-access-control/resource-provider-operations">https://docs.microsoft.com/en-us/azure/role-based-access-control/resource-provider-operations</a></li>



</ul>
<b>name</b> – The name of resource which you use when creating policy

<b>fullName</b> – The name of resource with parent object

<b>kind</b> – depends on type of resources


<b>type</b> – Some types example: "Microsoft.Storage/storageAccounts", "Microsoft.Network/virtualNetworks", "Microsoft.Network/publicIPAddresses", "Microsoft.Network/networkSecurityGroups", "Microsoft.Sql/servers”, "Microsoft.EventGrid/topics", "Microsoft.KeyVault/vaults" and many more

<b>location</b> – "westeurope”, "northeurope” and of course many more

<b>Source</b> - Action – Undocumented value. You can block or audit some of Azure action if you wish. It may be useful when you don’t won’t to create custom RBAC role or deny some action for all users (even Owners).

This is Azure Policy example for Action, when I would like to block creation of VM Extension:


```

{
  "mode": "all",
  "policyRule": {
    "if": {
      "source": "action",
      "equals": "Microsoft.Compute/virtualMachines/extensions/write"
    },
    "then": {
      "effect": "deny"
    }
  },
  "parameters": {}
}

```

<b>Aliases</b> – The value for specific service properties. At the time I wrote this article there is 1030 aliases created. You can list them by running this script:

```

#Login first with Connect-AzureRmAccount if not using Cloud Shell
$azContext = Get-AzureRmContext
$azProfile = [Microsoft.Azure.Commands.Common.Authentication.Abstractions.AzureRmProfileProvider]::Instance.Profile
$profileClient = New-Object -TypeName Microsoft.Azure.Commands.ResourceManager.Common.RMProfileClient -ArgumentList ($azProfile)
$token = $profileClient.AcquireAccessToken($azContext.Subscription.TenantId)
$authHeader = @{
  'Authorization'='Bearer ' + $token.AccessToken
}

#Create a splatting variable for Invoke-RestMethod
$invokeRest = @{
  Uri = 'https://management.azure.com/providers/?api-version=2018-05-01&$expand=resourceTypes/aliases'
  Method = 'Get'
  ContentType = 'application/json'
  Headers = $authHeader
}
#Invoke the REST API
$response = Invoke-RestMethod @invokeRest
#Create an List to hold discovered aliases
$aliases = [System.Collections.Generic.List[pscustomobject]]::new()
foreach ($ns in $response.value)
{
    foreach ($rT in $ns.resourceTypes)
    {
        if ($rT.aliases)
        {
            foreach ($obj in $rT.aliases)
            {
                $alias = [PSCustomObject]@{
                    Namespace    = $ns.namespace
                    resourceType = $rT.resourceType
                    alias        = $obj.name
                }
                $aliases.Add($alias)
            }
        }
    }
}
#Output the list and sort it by Namespace, resourceType and alias. You can customize with Where-Object to limit as desired.
$aliases | Sort-Object -Property Namespace, resourceType, alias
$aliases.count 

```

Ok, I have 1030 possible values, but how can I know which field values are possible? The best approach is to set properly all those values for such resource and then:

<ul>
<li>We can open Resource Explorer - <a href="https://resources.azure.com">https://resources.azure.com</a> and get value for resource</li>
<li>We can use Azure Resource Graph – You can read about it on Michal Smereczynski MVP blog - <a href="https://lnx.azurewebsites.net/saving-time-with-azure-resource-graph">https://lnx.azurewebsites.net/saving-time-with-azure-resource-graph</a></li>
<li>We can use Resource Explorer in Azure Portal – All services -> Resource Explorer</li>
</ul>

At the end of this article I’ll put the whole list of Aliases which you can use. You can always ask for specific aliases if you can’t find something useful for yourself. If so, please go to such URL:


[https://github.com/Azure/azure-policy/blob/master/1-contribution-guide/request-alias.md](https://github.com/Azure/azure-policy/blob/master/1-contribution-guide/request-alias.md)

### Effect
Azure Policy supports the following effect:
<ul>
<li>Deny – Unable to create resource when policy is enable. If you create policy and you had some resources already created, then they will be on non-compliant list.</li>
<li>Audit – Inform you about non-compliant resources</li>
<li>Append – adds extra fields </li>
<li>AuditIfNotExists – Inform you about something is not exist</li>
<li>DeployIfNotExists – deploy something if it not exist. For example VM extension used after creating a virtual machine</li>
</ul>
<b>PLEASE DO NOT USE DENY POLICY WHEN YOU ARE CREATING POLICY FOR A FIRST TIME. USE AUDIT!!!</b>




### WHEN DENY IS NOT DENYING ☹
There is a situation when Deny policy doesn’t work properly. Let me show you that with an example. I would like to prevent creating Storage Account without network ACLS (Firewall) enabled. I create such policy:

```

{
  "mode": "all",
  "policyRule": {
    "if": {
      "allOf": [
        {
          "field": "type",
          "equals": "Microsoft.Storage/storageAccounts"
        },
        {
          "field": "Microsoft.Storage/storageAccounts/networkAcls.defaultAction",
          "equals": "Allow"
        }
      ]
    },
    "then": {
      "effect": "deny"
    }
  },
  "parameters": {}
}

```

As you can see, there is deny effect enabled. So we can assume that we can’t create Storage Account without Network ACLS. What will really happened as we assign this policy? We will create it and after that it’ll be NON-Compliant resource. WHY THIS HAPPENED?  Because when you are creating such resource there is no information about networkAcls.defaultAction in template!!!  

Below you can see how I deny creating Storage Accout resource when the Secure transfer option is not on. I use Resource Explorer in portal and came up with this value: 

![AzurePolicy](https://mkpkmkpk.github.io/Images/Azure-Policy/AzurePolicy05.png "AzurePolicy")

With that I can write policy rule using appropriate aliases in a simpler way.

```

{
  "mode": "All",
  "policyRule": {
    "if": {
      "allOf": [
        {
          "field": "type",
          "equals": "Microsoft.Storage/storageAccounts" 
        },
        {
          "not": {
            "field": "Microsoft.Storage/storageAccounts/supportsHttpsTrafficOnly",
            "equals": "True"
          }
        }
      ]
    },
    "then": {
      "effect": "deny”
    }
  },
  "parameters": {}
  }
}

```


# Services Aliases:
### Analysis Services - 16
```
Microsoft.AnalysisServices/servers/asAdministrators
Microsoft.AnalysisServices/servers/asAdministrators.members                                                             
Microsoft.AnalysisServices/servers/asAdministrators.members[*]
Microsoft.AnalysisServices/servers/backupBlobContainerUri                                                               
Microsoft.AnalysisServices/servers/ipV4FirewallSettings                                                                 
Microsoft.AnalysisServices/servers/ipV4FirewallSettings.enablePowerBIService                                            
Microsoft.AnalysisServices/servers/ipV4FirewallSettings.firewallRules                                                   
Microsoft.AnalysisServices/servers/ipV4FirewallSettings.firewallRules[*]                                                
Microsoft.AnalysisServices/servers/ipV4FirewallSettings.firewallRules[*].firewallRuleName                               
Microsoft.AnalysisServices/servers/ipV4FirewallSettings.firewallRules[*].rangeEnd                                       
Microsoft.AnalysisServices/servers/ipV4FirewallSettings.firewallRules[*].rangeStart                                     
Microsoft.AnalysisServices/servers/managedMode                                                                          
Microsoft.AnalysisServices/servers/querypoolConnectionMode                                                              
Microsoft.AnalysisServices/servers/serverFullName                                                                       
Microsoft.AnalysisServices/servers/serverMonitorMode                                                                    
Microsoft.AnalysisServices/servers/state
```
### App Service - 122
```
Microsoft.Web/hostingEnvironments/clusterSettings[*].name
Microsoft.Web/hostingEnvironments/clusterSettings[*].value   
*NEW Microsoft.Web/hostingEnvironments/virtualNetwork
*NEW Microsoft.Web/hostingEnvironments/virtualNetwork.id
*NEW Microsoft.Web/hostingEnvironments/virtualNetwork.name
*NEW Microsoft.Web/hostingEnvironments/virtualNetwork.subnet
*NEW Microsoft.Web/hostingEnvironments/vnetName
*NEW Microsoft.Web/hostingEnvironments/vnetResourceGroupName
*NEW Microsoft.Web/hostingEnvironments/vnetSubnetName
Microsoft.Web/serverfarms/hostingEnvironmentProfile.id                                                                  
Microsoft.Web/serverfarms/sku.capacity                                                                                  
Microsoft.Web/serverfarms/sku.name                                                                                      
Microsoft.Web/sites/availabilityState                                                                                   
Microsoft.Web/sites/clientCertEnabled                                                                                   
Microsoft.Web/sites/hostNames[*]                                                                                        
Microsoft.Web/sites/hostNameSslStates[*].sslState                                                                       
Microsoft.Web/sites/httpsOnly                                                                                           
Microsoft.Web/sites/serverFarmId                                                                                        
Microsoft.Web/sites/usageState                                                                                          
Microsoft.Web/sites/config/alwaysOn                                                                                     
Microsoft.Web/sites/config/apiDefinition                                                                                
Microsoft.Web/sites/config/appCommandLine                                                                               
Microsoft.Web/sites/config/appSettings                                                                                  
Microsoft.Web/sites/config/autoHealEnabled                                                                              
Microsoft.Web/sites/config/autoHealRules                                                                                
Microsoft.Web/sites/config/autoSwapSlotName                                                                             
Microsoft.Web/sites/config/azureStorageAccounts                                                                         
Microsoft.Web/sites/config/connectionStrings                                                                            
Microsoft.Web/sites/config/cors                                                                                         
Microsoft.Web/sites/config/customAppPoolIdentityAdminState                                                              
Microsoft.Web/sites/config/customAppPoolIdentityTenantState                                                             
Microsoft.Web/sites/config/defaultDocuments                                                                             
Microsoft.Web/sites/config/defaultDocuments[*]                                                                          
Microsoft.Web/sites/config/detailedErrorLoggingEnabled                                                                  
Microsoft.Web/sites/config/documentRoot                                                                                 
Microsoft.Web/sites/config/experiments                                                                                  
Microsoft.Web/sites/config/experiments.rampUpRules                                                                      
Microsoft.Web/sites/config/experiments.rampUpRules[*]                                                                   
Microsoft.Web/sites/config/ftpsState                                                                                    
Microsoft.Web/sites/config/handlerMappings                                                                              
Microsoft.Web/sites/config/http20Enabled                                                                                
Microsoft.Web/sites/config/httpLoggingEnabled                                                                           
Microsoft.Web/sites/config/ipSecurityRestrictions                                                                       
Microsoft.Web/sites/config/ipSecurityRestrictions[*].ipAddress                                                          
Microsoft.Web/sites/config/ipSecurityRestrictions[*].subnetMask                                                         
Microsoft.Web/sites/config/javaContainer                                                                                
Microsoft.Web/sites/config/javaContainerVersion                                                                         
Microsoft.Web/sites/config/javaVersion                                                                                  
Microsoft.Web/sites/config/limits                                                                                       
Microsoft.Web/sites/config/linuxFxVersion                                                                               
Microsoft.Web/sites/config/loadBalancing                                                                                
Microsoft.Web/sites/config/localMySqlEnabled                                                                            
Microsoft.Web/sites/config/logsDirectorySizeLimit                                                                       
Microsoft.Web/sites/config/machineKey                                                                                   
Microsoft.Web/sites/config/managedPipelineMode                                                                          
Microsoft.Web/sites/config/managedServiceIdentityId                                                                     
Microsoft.Web/sites/config/metadata                                                                                     
Microsoft.Web/sites/config/minTlsVersion                                                                                
Microsoft.Web/sites/config/netFrameworkVersion                                                                          
Microsoft.Web/sites/config/nodeVersion                                                                                  
Microsoft.Web/sites/config/numberOfWorkers                                                                              
Microsoft.Web/sites/config/phpVersion                                                                                   
Microsoft.Web/sites/config/publishingPassword                                                                           
Microsoft.Web/sites/config/publishingUsername                                                                           
Microsoft.Web/sites/config/push                                                                                         
Microsoft.Web/sites/config/pythonVersion                                                                                
Microsoft.Web/sites/config/remoteDebuggingEnabled                                                                       
Microsoft.Web/sites/config/remoteDebuggingVersion                                                                       
Microsoft.Web/sites/config/requestTracingEnabled                                                                        
Microsoft.Web/sites/config/reservedInstanceCount                                                                        
Microsoft.Web/sites/config/routingRules                                                                                 
Microsoft.Web/sites/config/routingRules[*]                                                                              
Microsoft.Web/sites/config/runtimeADUser                                                                                
Microsoft.Web/sites/config/runtimeADUserPassword                                                                        
Microsoft.Web/sites/config/scmType                                                                                      
Microsoft.Web/sites/config/siteAuthEnabled                                                                              
Microsoft.Web/sites/config/siteAuthSettings                                                                             
Microsoft.Web/sites/config/siteAuthSettings.additionalLoginParams                                                       
Microsoft.Web/sites/config/siteAuthSettings.allowedAudiences                                                            
Microsoft.Web/sites/config/siteAuthSettings.allowedExternalRedirectUrls                                                 
Microsoft.Web/sites/config/siteAuthSettings.clientId                                                                    
Microsoft.Web/sites/config/siteAuthSettings.clientSecret                                                                
Microsoft.Web/sites/config/siteAuthSettings.defaultProvider                                                             
Microsoft.Web/sites/config/siteAuthSettings.enabled                                                                     
Microsoft.Web/sites/config/siteAuthSettings.facebookAppId                                                               
Microsoft.Web/sites/config/siteAuthSettings.facebookAppSecret                                                           
Microsoft.Web/sites/config/siteAuthSettings.facebookOAuthScopes                                                         
Microsoft.Web/sites/config/siteAuthSettings.googleClientId                                                              
Microsoft.Web/sites/config/siteAuthSettings.googleClientSecret                                                          
Microsoft.Web/sites/config/siteAuthSettings.googleOAuthScopes                                                           
Microsoft.Web/sites/config/siteAuthSettings.isAadAutoProvisioned                                                        
Microsoft.Web/sites/config/siteAuthSettings.issuer                                                                      
Microsoft.Web/sites/config/siteAuthSettings.microsoftAccountClientId                                                    
Microsoft.Web/sites/config/siteAuthSettings.microsoftAccountClientSecret                                                
Microsoft.Web/sites/config/siteAuthSettings.microsoftAccountOAuthScopes                                                 
Microsoft.Web/sites/config/siteAuthSettings.tokenStoreEnabled                                                           
Microsoft.Web/sites/config/siteAuthSettings.twitterConsumerKey                                                          
Microsoft.Web/sites/config/siteAuthSettings.twitterConsumerSecret                                                       
Microsoft.Web/sites/config/siteAuthSettings.unauthenticatedClientAction                                                 
Microsoft.Web/sites/config/tracingOptions                                                                               
Microsoft.Web/sites/config/use32BitWorkerProcess                                                                        
Microsoft.Web/sites/config/virtualApplications                                                                          
Microsoft.Web/sites/config/virtualApplications[*]                                                                       
Microsoft.Web/sites/config/virtualApplications[*].physicalPath                                                          
Microsoft.Web/sites/config/virtualApplications[*].preloadEnabled                                                        
Microsoft.Web/sites/config/virtualApplications[*].virtualDirectories                                                    
Microsoft.Web/sites/config/virtualApplications[*].virtualPath                                                           
Microsoft.Web/sites/config/vnetName                                                                                     
Microsoft.Web/sites/config/webSocketsEnabled                                                                            
Microsoft.Web/sites/config/winAuthAdminState                                                                            
Microsoft.Web/sites/config/winAuthTenantState                                                                           
Microsoft.Web/sites/config/windowsFxVersion                                                                             
Microsoft.Web/sites/config/xManagedServiceIdentityId
*NEW Microsoft.Web/sites/slots/availabilityState                                                                             
*NEW Microsoft.Web/sites/slots/clientCertEnabled                                                                             
*NEW Microsoft.Web/sites/slots/hostNames[*]                                                                                  
*NEW Microsoft.Web/sites/slots/hostNameSslStates[*].name                                                                     
*NEW Microsoft.Web/sites/slots/httpsOnly                                                                                     
*NEW Microsoft.Web/sites/slots/serverFarmId                                                                                  
*NEW Microsoft.Web/sites/slots/usageState                                                                                    
*NEW Microsoft.Web/sites/slots/config/ftpsState                                                                              
*NEW Microsoft.Web/sites/slots/config/minTlsVersion
```
### Application Gateway - 19
```
*NEW Microsoft.Network/applicationGateways/backendAddressPools[*]                                                            
*NEW Microsoft.Network/applicationGateways/backendAddressPools[*].backendAddresses[*]                                        
*NEW Microsoft.Network/applicationGateways/backendAddressPools[*].backendAddresses[*].fqdn                                   
*NEW Microsoft.Network/applicationGateways/backendAddressPools[*].name                                                       
*NEW Microsoft.Network/applicationGateways/frontendIPConfigurations[*].publicIPAddress.id                                    
*NEW Microsoft.Network/applicationGateways/gatewayIPConfigurations[*].subnet.id 
Microsoft.Network/applicationGateways/sku.capacity                                                                      
Microsoft.Network/applicationGateways/sku.name                                                                          
Microsoft.Network/applicationGateways/sku.tier 
*NEW Microsoft.Network/applicationGateways/sslPolicy                                                                         
*NEW Microsoft.Network/applicationGateways/sslPolicy.policyName                                                              
*NEW Microsoft.Network/applicationGateways/sslPolicy.policyType
*NEW Microsoft.Network/applicationGateways/webApplicationFirewallConfiguration                                                                        
Microsoft.Network/applicationGateways/webApplicationFirewallConfiguration.disabledRuleGroups[*].ruleGroupName           
Microsoft.Network/applicationGateways/webApplicationFirewallConfiguration.enabled                                       
Microsoft.Network/applicationGateways/webApplicationFirewallConfiguration.firewallMode                                  
Microsoft.Network/applicationGateways/webApplicationFirewallConfiguration.requestBodyCheck
*NEW Microsoft.Network/applicationGateways/webApplicationFirewallConfiguration.ruleSetType                                   
*NEW Microsoft.Network/applicationGateways/webApplicationFirewallConfiguration.ruleSetVersion
```
### Azure Automation - 3
```
Microsoft.Automation/automationAccounts/variables/isEncrypted                                                           
Microsoft.Automation/automationAccounts/webhooks/creationTime                                                           
Microsoft.Automation/automationAccounts/webhooks/expiryTime
```
### Azure Locks - 1
```
Microsoft.Authorization/locks/level
```
### Batch - 3
```
Microsoft.Batch/batchAccounts/autoStorage.storageAccountId                                                              
Microsoft.Batch/batchAccounts/keyVaultReference.id                                                                      
Microsoft.Batch/batchAccounts/poolAllocationMode
```
### CDN - 3
```
Microsoft.CDN/profiles/sku.name                                                                                         
microsoft.cdn/profiles/endpoints/isHttpAllowed                                                                          
microsoft.cdn/profiles/endpoints/isHttpsAllowed
```
### Container Registry - 7
```
*NEW Microsoft.ContainerRegistry/registries/adminUserEnabled
*NEW Microsoft.ContainerRegistry/registries/creationDate
*NEW Microsoft.ContainerRegistry/registries/firewallRules
*NEW Microsoft.ContainerRegistry/registries/firewallRules[*]
*NEW Microsoft.ContainerRegistry/registries/firewallRulesEnabled 
*NEW Microsoft.ContainerRegistry/registries/loginServer
*NEW Microsoft.ContainerRegistry/registries/provisioningState
```
### Container Service - 29
```
*NEW Microsoft.ContainerService/managedClusters/agentPoolProfiles                                                            
*NEW Microsoft.ContainerService/managedClusters/agentPoolProfiles[*]                                                         
*NEW Microsoft.ContainerService/managedClusters/agentPoolProfiles[*].count                                                   
*NEW Microsoft.ContainerService/managedClusters/agentPoolProfiles[*].maxPods                                                 
*NEW Microsoft.ContainerService/managedClusters/agentPoolProfiles[*].name                                                    
*NEW Microsoft.ContainerService/managedClusters/agentPoolProfiles[*].osDiskSizeGB                                            
*NEW Microsoft.ContainerService/managedClusters/agentPoolProfiles[*].osType                                                  
*NEW Microsoft.ContainerService/managedClusters/agentPoolProfiles[*].storageProfile                                          
*NEW Microsoft.ContainerService/managedClusters/agentPoolProfiles[*].vmSize                                                  
*NEW Microsoft.ContainerService/managedClusters/dnsPrefix                                                                    
*NEW Microsoft.ContainerService/managedClusters/enableRBAC                                                                   
*NEW Microsoft.ContainerService/managedClusters/fqdn                                                                         
*NEW Microsoft.ContainerService/managedClusters/kubernetesVersion                                                            
*NEW Microsoft.ContainerService/managedClusters/linuxProfile                                                                 
*NEW Microsoft.ContainerService/managedClusters/linuxProfile.adminUsername                                                   
*NEW Microsoft.ContainerService/managedClusters/linuxProfile.ssh                                                             
*NEW Microsoft.ContainerService/managedClusters/linuxProfile.ssh.publicKeys                                                  
*NEW Microsoft.ContainerService/managedClusters/linuxProfile.ssh.publicKeys[*]                                               
*NEW Microsoft.ContainerService/managedClusters/linuxProfile.ssh.publicKeys[*].keyData                                       
*NEW Microsoft.ContainerService/managedClusters/networkProfile                                                               
*NEW Microsoft.ContainerService/managedClusters/networkProfile.dnsServiceIP                                                  
*NEW Microsoft.ContainerService/managedClusters/networkProfile.dockerBridgeCidr                                              
*NEW Microsoft.ContainerService/managedClusters/networkProfile.networkPlugin                                                 
*NEW Microsoft.ContainerService/managedClusters/networkProfile.podCidr                                                       
*NEW Microsoft.ContainerService/managedClusters/networkProfile.serviceCidr                                                   
*NEW Microsoft.ContainerService/managedClusters/nodeResourceGroup                                                            
*NEW Microsoft.ContainerService/managedClusters/provisioningState                                                            
*NEW Microsoft.ContainerService/managedClusters/servicePrincipalProfile                                                      
*NEW Microsoft.ContainerService/managedClusters/servicePrincipalProfile.clientId
```
### Cosmos DB - 22
```
Microsoft.DocumentDB/databaseAccounts/consistencyPolicy.defaultConsistencyLevel                                         
Microsoft.DocumentDB/databaseAccounts/enableAutomaticFailover
*NEW Microsoft.DocumentDB/databaseAccounts/enableMultipleWriteLocations                                                           
Microsoft.DocumentDB/databaseAccounts/failoverPolicies                                                                  
Microsoft.DocumentDB/databaseAccounts/failoverPolicies[*]                                                               
Microsoft.DocumentDB/databaseAccounts/failoverPolicies[*].locationName                                                  
Microsoft.DocumentDB/databaseAccounts/ipRangeFilter                                                                     
Microsoft.DocumentDB/databaseAccounts/isVirtualNetworkFilterEnabled                                                     
Microsoft.DocumentDB/databaseAccounts/Locations                                                                         
Microsoft.DocumentDB/databaseAccounts/Locations[*]                                                                      
Microsoft.DocumentDB/databaseAccounts/Locations[*].locationName                                                         
Microsoft.DocumentDB/databaseAccounts/readLocations                                                                     
Microsoft.DocumentDB/databaseAccounts/readLocations[*]                                                                  
Microsoft.DocumentDB/databaseAccounts/readLocations[*].locationName                                                     
Microsoft.DocumentDB/databaseAccounts/sku.name
*NEW Microsoft.DocumentDB/databaseAccounts/virtualNetworkRules                                                                          
Microsoft.DocumentDB/databaseAccounts/virtualNetworkRules[*]                                                            
Microsoft.DocumentDB/databaseAccounts/virtualNetworkRules[*].id 
*NEW Microsoft.DocumentDB/databaseAccounts/virtualNetworkRules[*].ignoreMissingVNetServiceEndpoint                                                    
Microsoft.DocumentDB/databaseAccounts/writeLocations                                                                    
Microsoft.DocumentDB/databaseAccounts/writeLocations[*]                                                                 
Microsoft.DocumentDB/databaseAccounts/writeLocations[*].locationName
```
### Data Factory - 49
```
Microsoft.DataFactory/factories/version                                                                                 
Microsoft.DataFactory/factories/vstsConfiguration                                                                       
Microsoft.DataFactory/factories/vstsConfiguration.accountName                                                           
Microsoft.DataFactory/factories/vstsConfiguration.collaborationBranch                                                   
Microsoft.DataFactory/factories/vstsConfiguration.lastCommitId                                                          
Microsoft.DataFactory/factories/vstsConfiguration.projectName                                                           
Microsoft.DataFactory/factories/vstsConfiguration.repositoryName                                                        
Microsoft.DataFactory/factories/vstsConfiguration.rootFolder                                                            
Microsoft.DataFactory/factories/vstsConfiguration.tenantId                                                              
Microsoft.DataFactory/factories/datasets/linkedServiceName                                                              
Microsoft.DataFactory/factories/datasets/linkedServiceName.referenceName                                                
Microsoft.DataFactory/factories/datasets/linkedServiceName.type                                                         
Microsoft.DataFactory/factories/datasets/parameters                                                                     
Microsoft.DataFactory/factories/datasets/type                                                                           
Microsoft.DataFactory/factories/datasets/typeProperties                                                                 
Microsoft.DataFactory/factories/integrationruntimes/state                                                               
Microsoft.DataFactory/factories/integrationruntimes/type                                                                
Microsoft.DataFactory/factories/integrationruntimes/typeProperties                                                      
Microsoft.DataFactory/factories/integrationruntimes/typeProperties.computeProperties                                    
Microsoft.DataFactory/factories/integrationruntimes/typeProperties.computeProperties.location                           
Microsoft.DataFactory/factories/integrationruntimes/typeProperties.computeProperties.maxParallelExecutionsPerNode       
Microsoft.DataFactory/factories/integrationruntimes/typeProperties.computeProperties.nodeSize                           
Microsoft.DataFactory/factories/integrationruntimes/typeProperties.computeProperties.numberOfNodes                      
Microsoft.DataFactory/factories/integrationruntimes/typeProperties.ssisProperties                                       
Microsoft.DataFactory/factories/integrationruntimes/typeProperties.ssisProperties.catalogInfo                           
Microsoft.DataFactory/factories/integrationruntimes/typeProperties.ssisProperties.catalogInfo.catalogAdminPassword      
Microsoft.DataFactory/factories/integrationruntimes/typeProperties.ssisProperties.catalogInfo.catalogAdminPassword.type 
Microsoft.DataFactory/factories/integrationruntimes/typeProperties.ssisProperties.catalogInfo.catalogAdminPassword.value
Microsoft.DataFactory/factories/integrationruntimes/typeProperties.ssisProperties.catalogInfo.catalogAdminUserName      
Microsoft.DataFactory/factories/integrationruntimes/typeProperties.ssisProperties.catalogInfo.catalogPricingTier        
Microsoft.DataFactory/factories/integrationruntimes/typeProperties.ssisProperties.catalogInfo.catalogServerEndpoint     
Microsoft.DataFactory/factories/integrationruntimes/typeProperties.ssisProperties.licenseType                           
Microsoft.DataFactory/factories/linkedservices/type                                                                     
Microsoft.DataFactory/factories/linkedservices/typeProperties                                                           
Microsoft.DataFactory/factories/linkedservices/typeProperties.connectionString                                          
Microsoft.DataFactory/factories/linkedservices/typeProperties.connectionString.type                                     
Microsoft.DataFactory/factories/linkedservices/typeProperties.encryptedCredential                                       
Microsoft.DataFactory/factories/linkedservices/typeProperties.useEncryptedEndpoints                                     
Microsoft.DataFactory/factories/pipelines/activities                                                                    
Microsoft.DataFactory/factories/pipelines/activities[*]                                                                 
Microsoft.DataFactory/factories/pipelines/activities[*].name                                                            
Microsoft.DataFactory/factories/pipelines/activities[*].policy                                                          
Microsoft.DataFactory/factories/pipelines/activities[*].policy.retry                                                    
Microsoft.DataFactory/factories/pipelines/activities[*].policy.retryIntervalInSeconds                                   
Microsoft.DataFactory/factories/pipelines/activities[*].policy.secureOutput                                             
Microsoft.DataFactory/factories/pipelines/activities[*].policy.timeout                                                  
Microsoft.DataFactory/factories/pipelines/activities[*].type                                                            
Microsoft.DataFactory/factories/pipelines/activities[*].typeProperties                                                  
Microsoft.DataFactory/factories/pipelines/parameters
```
### Data Lake Analytics - 43
```
*NEW Microsoft.DataLakeAnalytics/accounts/accountId                                                                          
*NEW Microsoft.DataLakeAnalytics/accounts/computePolicies                                                                    
*NEW Microsoft.DataLakeAnalytics/accounts/computePolicies[*]                                                                 
*NEW Microsoft.DataLakeAnalytics/accounts/computePolicies[*].maxDegreeOfParallelismPerJob                                    
*NEW Microsoft.DataLakeAnalytics/accounts/computePolicies[*].minPriorityPerJob                                               
*NEW Microsoft.DataLakeAnalytics/accounts/computePolicies[*].objectId                                                        
*NEW Microsoft.DataLakeAnalytics/accounts/computePolicies[*].objectType                                                      
*NEW Microsoft.DataLakeAnalytics/accounts/creationTime                                                                       
*NEW Microsoft.DataLakeAnalytics/accounts/currentTier                                                                        
*NEW Microsoft.DataLakeAnalytics/accounts/dataLakeStoreAccounts                                                              
*NEW Microsoft.DataLakeAnalytics/accounts/dataLakeStoreAccounts[*]                                                           
*NEW Microsoft.DataLakeAnalytics/accounts/dataLakeStoreAccounts[*].id                                                        
*NEW Microsoft.DataLakeAnalytics/accounts/dataLakeStoreAccounts[*].name                                                      
*NEW Microsoft.DataLakeAnalytics/accounts/dataLakeStoreAccounts[*].suffix                                                    
*NEW Microsoft.DataLakeAnalytics/accounts/defaultDataLakeStoreAccount                                                        
*NEW Microsoft.DataLakeAnalytics/accounts/endIpAddress                                                                       
*NEW Microsoft.DataLakeAnalytics/accounts/endpoint                                                                           
*NEW Microsoft.DataLakeAnalytics/accounts/firewallAllowAzureIps                                                              
*NEW Microsoft.DataLakeAnalytics/accounts/firewallRules                                                                      
*NEW Microsoft.DataLakeAnalytics/accounts/firewallRules[*]                                                                   
*NEW Microsoft.DataLakeAnalytics/accounts/firewallRules[*].endIpAddress                                                      
*NEW Microsoft.DataLakeAnalytics/accounts/firewallRules[*].startIpAddress                                                    
*NEW Microsoft.DataLakeAnalytics/accounts/firewallState                                                                      
*NEW Microsoft.DataLakeAnalytics/accounts/lastModifiedTime                                                                   
*NEW Microsoft.DataLakeAnalytics/accounts/maxDegreeOfParallelism                                                             
*NEW Microsoft.DataLakeAnalytics/accounts/maxDegreeOfParallelismPerJob                                                       
*NEW Microsoft.DataLakeAnalytics/accounts/maxJobCount                                                                        
*NEW Microsoft.DataLakeAnalytics/accounts/minPriorityPerJob                                                                  
*NEW Microsoft.DataLakeAnalytics/accounts/newTier                                                                            
*NEW Microsoft.DataLakeAnalytics/accounts/objectId                                                                           
*NEW Microsoft.DataLakeAnalytics/accounts/objectType                                                                         
*NEW Microsoft.DataLakeAnalytics/accounts/provisioningState                                                                  
*NEW Microsoft.DataLakeAnalytics/accounts/queryStoreRetention                                                                
*NEW Microsoft.DataLakeAnalytics/accounts/startIpAddress                                                                     
*NEW Microsoft.DataLakeAnalytics/accounts/state                                                                              
*NEW Microsoft.DataLakeAnalytics/accounts/storageAccounts                                                                    
*NEW Microsoft.DataLakeAnalytics/accounts/storageAccounts[*]                                                                 
*NEW Microsoft.DataLakeAnalytics/accounts/storageAccounts[*].accessKey                                                       
*NEW Microsoft.DataLakeAnalytics/accounts/storageAccounts[*].id                                                              
*NEW Microsoft.DataLakeAnalytics/accounts/storageAccounts[*].name                                                            
*NEW Microsoft.DataLakeAnalytics/accounts/storageAccounts[*].suffix                                                          
*NEW Microsoft.DataLakeAnalytics/accounts/systemMaxDegreeOfParallelism                                                       
*NEW Microsoft.DataLakeAnalytics/accounts/systemMaxJobCount
```
### Data Lake Store - 5
```
Microsoft.DataLakeStore/accounts/encryptionState                                                                        
Microsoft.DataLakeStore/accounts/firewallState
*NEW Microsoft.DataLakeStore/accounts/newTier                                                                          
Microsoft.DataLakeStore/accounts/firewallRules/endIpAddress                                                             
Microsoft.DataLakeStore/accounts/firewallRules/startIpAddress
```
### Disks - 27
```
*NEW Microsoft.Compute/disks/accountType                                                           
*NEW Microsoft.Compute/disks/creationData                                                          
*NEW Microsoft.Compute/disks/creationData.createOption                                             
*NEW Microsoft.Compute/disks/creationData.imageReference                                           
*NEW Microsoft.Compute/disks/creationData.imageReference.id                                        
*NEW Microsoft.Compute/disks/diskSizeGB                                                            
*NEW Microsoft.Compute/disks/diskState                                                             
*NEW Microsoft.Compute/disks/encryptionSettings                                                    
*NEW Microsoft.Compute/disks/encryptionSettings.diskEncryptionKey                                  
*NEW Microsoft.Compute/disks/encryptionSettings.diskEncryptionKey.secretUrl                        
*NEW Microsoft.Compute/disks/encryptionSettings.diskEncryptionKey.sourceVault                      
*NEW Microsoft.Compute/disks/encryptionSettings.diskEncryptionKey.sourceVault.id                   
*NEW Microsoft.Compute/disks/encryptionSettings.enabled                                            
*NEW Microsoft.Compute/disks/encryptionSettings.keyEncryptionKey                                   
*NEW Microsoft.Compute/disks/encryptionSettings.keyEncryptionKey.keyUrl                            
*NEW Microsoft.Compute/disks/encryptionSettings.keyEncryptionKey.sourceVault                       
*NEW Microsoft.Compute/disks/encryptionSettings.keyEncryptionKey.sourceVault.id  
Microsoft.Compute/disks/osType
Microsoft.Compute/disks/provisioningState
Microsoft.Compute/disks/sku.name   
*NEW Microsoft.Compute/disks/sku.tier
*NEW Microsoft.Compute/disks/timeCreated                                                                                      
Microsoft.Compute/imageId                                                                                               
Microsoft.Compute/imageOffer                                                                                            
Microsoft.Compute/imagePublisher                                                                                        
Microsoft.Compute/imageSku                                                                                              
Microsoft.Compute/imageVersion
```
### Event Hub - 48
```
*NEW Microsoft.EventHub/namespaces/archiveDescription                                                                        
*NEW Microsoft.EventHub/namespaces/archiveDescription.destination                                                            
*NEW Microsoft.EventHub/namespaces/archiveDescription.destination.archiveNameFormat                                          
*NEW Microsoft.EventHub/namespaces/archiveDescription.destination.blobContainer                                              
*NEW Microsoft.EventHub/namespaces/archiveDescription.destination.name                                                       
*NEW Microsoft.EventHub/namespaces/archiveDescription.destination.storageAccountResourceId                                   
*NEW Microsoft.EventHub/namespaces/archiveDescription.enabled                                                                
*NEW Microsoft.EventHub/namespaces/archiveDescription.encoding                                                               
*NEW Microsoft.EventHub/namespaces/archiveDescription.intervalInSeconds                                                      
*NEW Microsoft.EventHub/namespaces/archiveDescription.sizeLimitInBytes                                                       
*NEW Microsoft.EventHub/namespaces/captureDescription                                                                        
*NEW Microsoft.EventHub/namespaces/captureDescription.destination                                                            
*NEW Microsoft.EventHub/namespaces/captureDescription.destination.archiveNameFormat                                          
*NEW Microsoft.EventHub/namespaces/captureDescription.destination.blobContainer                                              
*NEW Microsoft.EventHub/namespaces/captureDescription.destination.name                                                       
*NEW Microsoft.EventHub/namespaces/captureDescription.destination.storageAccountResourceId                                   
*NEW Microsoft.EventHub/namespaces/captureDescription.enabled                                                                
*NEW Microsoft.EventHub/namespaces/captureDescription.encoding                                                               
*NEW Microsoft.EventHub/namespaces/captureDescription.intervalInSeconds                                                      
*NEW Microsoft.EventHub/namespaces/captureDescription.sizeLimitInBytes                                                       
*NEW Microsoft.EventHub/namespaces/createdAt                                                                                 
*NEW Microsoft.EventHub/namespaces/critical                                                                                  
*NEW Microsoft.EventHub/namespaces/enabled                                                                                   
*NEW Microsoft.EventHub/namespaces/eventHubEnabled                                                                           
*NEW Microsoft.EventHub/namespaces/isAutoInflateEnabled                                                                      
*NEW Microsoft.EventHub/namespaces/kafkaEnabled                                                                              
*NEW Microsoft.EventHub/namespaces/maximumThroughputUnits                                                                    
*NEW Microsoft.EventHub/namespaces/messageRetentionInDays                                                                    
*NEW Microsoft.EventHub/namespaces/messagingSku                                                                              
*NEW Microsoft.EventHub/namespaces/messagingSkuPlan                                                                          
*NEW Microsoft.EventHub/namespaces/messagingSkuPlan.isAutoInflateEnabled                                                     
*NEW Microsoft.EventHub/namespaces/messagingSkuPlan.maximumThroughputUnits                                                   
*NEW Microsoft.EventHub/namespaces/messagingSkuPlan.selectedEventHubUnit                                                     
*NEW Microsoft.EventHub/namespaces/messagingSkuPlan.sku                                                                      
*NEW Microsoft.EventHub/namespaces/metricId                                                                                  
*NEW Microsoft.EventHub/namespaces/namespaceType                                                                             
*NEW Microsoft.EventHub/namespaces/partitionCount                                                                            
*NEW Microsoft.EventHub/namespaces/partitionIds                                                                              
*NEW Microsoft.EventHub/namespaces/partitionIds[*]                                                                           
*NEW Microsoft.EventHub/namespaces/path                                                                                      
*NEW Microsoft.EventHub/namespaces/provisioningState                                                                         
*NEW Microsoft.EventHub/namespaces/serviceBusEndpoint                                                                        
*NEW Microsoft.EventHub/namespaces/sku.capacity                                                                              
*NEW Microsoft.EventHub/namespaces/sku.name                                                                                  
*NEW Microsoft.EventHub/namespaces/sku.tier                                                                                  
*NEW Microsoft.EventHub/namespaces/status                                                                                    
*NEW Microsoft.EventHub/namespaces/updatedAt                                                                                 
*NEW Microsoft.EventHub/namespaces/zoneRedundant                               
```
### Express Route - 5
```
Microsoft.Network/expressRouteCircuits/serviceProvider.bandwidthInMbps                                                  
Microsoft.Network/expressRouteCircuits/serviceProvider.peeringLocation                                                  
Microsoft.Network/expressRouteCircuits/sku.family                                                                       
Microsoft.Network/expressRouteCircuits/sku.name                                                                         
Microsoft.Network/expressRouteCircuits/sku.tier
```
### Guest Configuration Assignments - 1
```
Microsoft.GuestConfiguration/guestConfigurationAssignments/complianceStatus
```
### HDInsight - 9
```
Microsoft.HDInsight/clusters/clusterDefinition.kind                                                                     
Microsoft.HDInsight/clusters/clusterVersion                                                                             
Microsoft.HDInsight/clusters/computeProfile.roles[*].virtualNetworkProfile.id                                           
Microsoft.HDInsight/clusters/computeProfile.roles[*].virtualNetworkProfile.subnet                                       
Microsoft.HDInsight/clusters/osType                                                                                     
Microsoft.HDInsight/clusters/securityProfile.directoryType                                                              
Microsoft.HDInsight/clusters/securityProfile.domain                                                                     
Microsoft.HDInsight/clusters/securityProfile.ldapsUrls[*]                                                               
Microsoft.HDInsight/clusters/tier
```
### Insights - 43
```
Microsoft.Insights/ActivityLogAlerts/condition                                                                          
Microsoft.Insights/ActivityLogAlerts/condition.allOf                                                                    
Microsoft.Insights/ActivityLogAlerts/condition.allOf[*]                                                                 
Microsoft.Insights/ActivityLogAlerts/condition.allOf[*].containsAny                                                     
Microsoft.Insights/ActivityLogAlerts/condition.allOf[*].equals                                                          
Microsoft.Insights/ActivityLogAlerts/condition.allOf[*].field                                                           
Microsoft.Insights/ActivityLogAlerts/enabled                                                                            
Microsoft.Insights/ActivityLogAlerts/scopes                                                                             
Microsoft.Insights/ActivityLogAlerts/scopes[*]                                                                          
Microsoft.Insights/alertRules/actions[*].customEmails                                                                   
Microsoft.Insights/alertRules/actions[*].customEmails[*]                                                                
Microsoft.Insights/alertRules/actions[*].odata.type                                                                     
Microsoft.Insights/alertRules/actions[*].sendToServiceOwners                                                            
Microsoft.Insights/alertRules/actions[*].serviceUri                                                                     
Microsoft.Insights/alertRules/condition.dataSource.metricName                                                           
Microsoft.Insights/alertRules/condition.dataSource.odata.type                                                           
Microsoft.Insights/alertRules/condition.dataSource.resourceUri                                                          
Microsoft.Insights/alertRules/condition.operator                                                                        
Microsoft.Insights/alertRules/condition.threshold                                                                       
Microsoft.Insights/alertRules/condition.timeAggregation                                                                 
Microsoft.Insights/alertRules/condition.windowSize                                                                      
Microsoft.Insights/alertRules/isEnabled                                                                                 
Microsoft.Insights/diagnosticSettings/eventHubAuthorizationRuleId                                                       
Microsoft.Insights/diagnosticSettings/eventHubName                                                                      
Microsoft.Insights/diagnosticSettings/logs.enabled                                                                      
Microsoft.Insights/diagnosticSettings/logs[*].category                                                                  
Microsoft.Insights/diagnosticSettings/logs[*].retentionPolicy.days                                                      
Microsoft.Insights/diagnosticSettings/logs[*].retentionPolicy.enabled                                                   
Microsoft.Insights/diagnosticSettings/metrics.enabled                                                                   
Microsoft.Insights/diagnosticSettings/metrics[*].category                                                               
Microsoft.Insights/diagnosticSettings/metrics[*].retentionPolicy.days                                                   
Microsoft.Insights/diagnosticSettings/metrics[*].retentionPolicy.enabled                                                
Microsoft.Insights/diagnosticSettings/storageAccountId                                                                  
Microsoft.Insights/diagnosticSettings/workspaceId                                                                       
Microsoft.Insights/logProfiles/categories                                                                               
Microsoft.Insights/logProfiles/categories[*]                                                                            
Microsoft.Insights/logProfiles/locations                                                                                
Microsoft.Insights/logProfiles/locations[*]                                                                             
Microsoft.Insights/logProfiles/retentionPolicy                                                                          
Microsoft.Insights/logProfiles/retentionPolicy.days                                                                     
Microsoft.Insights/logProfiles/retentionPolicy.enabled                                                                  
Microsoft.Insights/logProfiles/serviceBusRuleId                                                                         
Microsoft.Insights/logProfiles/storageAccountId
```
### IoT Hubs - 6
```
Microsoft.Devices/IotHubs/ipFilterRules[*].action                                                                       
Microsoft.Devices/IotHubs/ipFilterRules[*].filterName                                                                   
Microsoft.Devices/IotHubs/ipFilterRules[*].ipMask                                                                       
Microsoft.Devices/IotHubs/sku.capacity                                                                                  
Microsoft.Devices/IotHubs/sku.name                                                                                      
Microsoft.Devices/IotHubs/sku.tier
```
### Key Vault - 20
```
Microsoft.KeyVault/vaults/enabledForDeployment                                                                          
Microsoft.KeyVault/vaults/enabledForDiskEncryption                                                                      
Microsoft.KeyVault/vaults/enabledForTemplateDeployment                                                                  
Microsoft.KeyVault/vaults/enablePurgeProtection                                                                         
Microsoft.KeyVault/vaults/enableSoftDelete                                                                              
Microsoft.KeyVault/vaults/networkAcls.defaultAction                                                                     
Microsoft.KeyVault/vaults/networkAcls.ipRules                                                                           
Microsoft.KeyVault/vaults/networkAcls.ipRules[*]                                                                        
Microsoft.KeyVault/vaults/networkAcls.ipRules[*].value                                                                  
Microsoft.KeyVault/vaults/networkAcls.virtualNetworkRules                                                               
Microsoft.KeyVault/vaults/networkAcls.virtualNetworkRules[*]                                                            
Microsoft.KeyVault/vaults/networkAcls.virtualNetworkRules[*].id                                                         
Microsoft.KeyVault/vaults/sku.family                                                                                    
Microsoft.KeyVault/vaults/sku.name                                                                                      
Microsoft.KeyVault/vaults/secrets/attributes                                                                            
Microsoft.KeyVault/vaults/secrets/attributes.created                                                                    
Microsoft.KeyVault/vaults/secrets/attributes.enabled                                                                    
Microsoft.KeyVault/vaults/secrets/attributes.exp                                                                        
Microsoft.KeyVault/vaults/secrets/attributes.nbf                                                                        
Microsoft.KeyVault/vaults/secrets/attributes.updated
```
### Load Balancer - 4
```
Microsoft.Network/loadBalancers/frontendIPConfigurations[*].privateIPAllocationMethod                                   
Microsoft.Network/loadBalancers/frontendIPConfigurations[*].publicIPAddress.id                                          
Microsoft.Network/loadBalancers/frontendIPConfigurations[*].subnet.id                                                   
Microsoft.Network/loadBalancers/sku.name
```
### Logic Apps - 26
```
Microsoft.Logic/workflows/accessControl                                                                                 
Microsoft.Logic/workflows/accessControl.actions                                                                         
Microsoft.Logic/workflows/accessControl.actions.allowedCallerIpAddresses                                                
Microsoft.Logic/workflows/accessControl.actions.allowedCallerIpAddresses[*]                                             
Microsoft.Logic/workflows/accessControl.actions.allowedCallerIpAddresses[*].addressRange                                
Microsoft.Logic/workflows/accessControl.contents                                                                        
Microsoft.Logic/workflows/accessControl.contents.allowedCallerIpAddresses                                               
Microsoft.Logic/workflows/accessControl.contents.allowedCallerIpAddresses[*]                                            
Microsoft.Logic/workflows/accessControl.contents.allowedCallerIpAddresses[*].addressRange                               
Microsoft.Logic/workflows/accessControl.triggers                                                                        
Microsoft.Logic/workflows/accessControl.triggers.allowedCallerIpAddresses                                               
Microsoft.Logic/workflows/accessControl.triggers.allowedCallerIpAddresses[*]                                            
Microsoft.Logic/workflows/accessControl.triggers.allowedCallerIpAddresses[*].addressRange                               
Microsoft.Logic/workflows/endpointsConfiguration                                                                        
Microsoft.Logic/workflows/endpointsConfiguration.connector                                                              
Microsoft.Logic/workflows/endpointsConfiguration.connector.outgoingIpAddresses                                          
Microsoft.Logic/workflows/endpointsConfiguration.connector.outgoingIpAddresses[*]                                       
Microsoft.Logic/workflows/endpointsConfiguration.connector.outgoingIpAddresses[*].address                               
Microsoft.Logic/workflows/endpointsConfiguration.workflow                                                               
Microsoft.Logic/workflows/endpointsConfiguration.workflow.accessEndpointIpAddresses                                     
Microsoft.Logic/workflows/endpointsConfiguration.workflow.accessEndpointIpAddresses[*]                                  
Microsoft.Logic/workflows/endpointsConfiguration.workflow.accessEndpointIpAddresses[*].address                          
Microsoft.Logic/workflows/endpointsConfiguration.workflow.outgoingIpAddresses                                           
Microsoft.Logic/workflows/endpointsConfiguration.workflow.outgoingIpAddresses[*]                                        
Microsoft.Logic/workflows/endpointsConfiguration.workflow.outgoingIpAddresses[*].address                                
Microsoft.Logic/workflows/state
```
### MySQL - 31
```
*NEW Microsoft.DBforMySQL/servers/administratorLogin                                                                         
*NEW Microsoft.DBforMySQL/servers/earliestRestoreDate                                                                        
*NEW Microsoft.DBforMySQL/servers/fullyQualifiedDomainName                                                                   
*NEW Microsoft.DBforMySQL/servers/masterServerId                                                                             
*NEW Microsoft.DBforMySQL/servers/replicaCapacity                                                                            
*NEW Microsoft.DBforMySQL/servers/replicationRole                                                                            
*NEW Microsoft.DBforMySQL/servers/sku.capacity                                                                               
*NEW Microsoft.DBforMySQL/servers/sku.family                                                                                 
*NEW Microsoft.DBforMySQL/servers/sku.name                                                                                   
*NEW Microsoft.DBforMySQL/servers/sku.tier                                                                                   
*NEW Microsoft.DBforMySQL/servers/sslEnforcement                                                                             
*NEW Microsoft.DBforMySQL/servers/storageProfile                                                                             
*NEW Microsoft.DBforMySQL/servers/storageProfile.backupRetentionDays                                                         
*NEW Microsoft.DBforMySQL/servers/storageProfile.geoRedundantBackup                                                          
*NEW Microsoft.DBforMySQL/servers/storageProfile.storageMB                                                                   
*NEW Microsoft.DBforMySQL/servers/userVisibleState                                                                           
*NEW Microsoft.DBforMySQL/servers/version                                                                                    
*NEW Microsoft.DBforMySQL/servers/firewallRules/endIpAddress                                                                 
*NEW Microsoft.DBforMySQL/servers/firewallRules/startIpAddress                                                               
*NEW Microsoft.DBforMySQL/servers/securityAlertPolicies/disabledAlerts                                                       
*NEW Microsoft.DBforMySQL/servers/securityAlertPolicies/disabledAlerts[*]                                                    
*NEW Microsoft.DBforMySQL/servers/securityAlertPolicies/emailAccountAdmins                                                   
*NEW Microsoft.DBforMySQL/servers/securityAlertPolicies/emailAddresses                                                       
*NEW Microsoft.DBforMySQL/servers/securityAlertPolicies/emailAddresses[*]                                                    
*NEW Microsoft.DBforMySQL/servers/securityAlertPolicies/retentionDays                                                        
*NEW Microsoft.DBforMySQL/servers/securityAlertPolicies/state                                                                
*NEW Microsoft.DBforMySQL/servers/securityAlertPolicies/storageAccountAccessKey                                              
*NEW Microsoft.DBforMySQL/servers/securityAlertPolicies/storageEndpoint                                                      
*NEW Microsoft.DBforMySQL/servers/virtualNetworkRules/ignoreMissingVnetServiceEndpoint                                       
*NEW Microsoft.DBforMySQL/servers/virtualNetworkRules/state                                                                  
*NEW Microsoft.DBforMySQL/servers/virtualNetworkRules/virtualNetworkSubnetId
```
### Network Interface - 7
```
Microsoft.Network/networkInterfaces/enableIPForwarding                                                                  
Microsoft.Network/networkInterfaces/ipconfigurations[*].applicationSecurityGroups                                       
Microsoft.Network/networkInterfaces/ipconfigurations[*].applicationSecurityGroups[*].id                                 
Microsoft.Network/networkInterfaces/ipconfigurations[*].privateIPAllocationMethod                                       
Microsoft.Network/networkInterfaces/ipconfigurations[*].publicIpAddress.id                                              
Microsoft.Network/networkInterfaces/ipconfigurations[*].subnet.id                                                       
Microsoft.Network/networkInterfaces/networkSecurityGroup.id
```
### Network Security Group - 32
```
Microsoft.Network/networkSecurityGroups/networkInterfaces[*].id                                                         
Microsoft.Network/networkSecurityGroups/securityRules[*].access
*NEW Microsoft.Network/networkSecurityGroups/securityRules[*].description                                                         
Microsoft.Network/networkSecurityGroups/securityRules[*].destinationAddressPrefix                                       
Microsoft.Network/networkSecurityGroups/securityRules[*].destinationAddressPrefixes[*]                                  
Microsoft.Network/networkSecurityGroups/securityRules[*].destinationApplicationSecurityGroups[*]                        
Microsoft.Network/networkSecurityGroups/securityRules[*].destinationPortRange                                           
Microsoft.Network/networkSecurityGroups/securityRules[*].destinationPortRanges[*]                                       
Microsoft.Network/networkSecurityGroups/securityRules[*].direction                                                      
Microsoft.Network/networkSecurityGroups/securityRules[*].priority                                                       
Microsoft.Network/networkSecurityGroups/securityRules[*].protocol                                                       
Microsoft.Network/networkSecurityGroups/securityRules[*].sourceAddressPrefix                                            
Microsoft.Network/networkSecurityGroups/securityRules[*].sourceAddressPrefixes[*]                                       
Microsoft.Network/networkSecurityGroups/securityRules[*].sourceApplicationSecurityGroups[*]                             
Microsoft.Network/networkSecurityGroups/securityRules[*].sourcePortRange                                                
Microsoft.Network/networkSecurityGroups/securityRules[*].sourcePortRanges[*]                                            
Microsoft.Network/networkSecurityGroups/subnets[*].id                                                                   
Microsoft.Network/networkSecurityGroups/securityRules/access
*NEW Microsoft.Network/networkSecurityGroups/securityRules/description                                                             
Microsoft.Network/networkSecurityGroups/securityRules/destinationAddressPrefix                                          
Microsoft.Network/networkSecurityGroups/securityRules/destinationAddressPrefixes[*]                                     
Microsoft.Network/networkSecurityGroups/securityRules/destinationApplicationSecurityGroups[*]                           
Microsoft.Network/networkSecurityGroups/securityRules/destinationPortRange                                              
Microsoft.Network/networkSecurityGroups/securityRules/destinationPortRanges[*]                                          
Microsoft.Network/networkSecurityGroups/securityRules/direction                                                         
Microsoft.Network/networkSecurityGroups/securityRules/priority                                                          
Microsoft.Network/networkSecurityGroups/securityRules/protocol                                                          
Microsoft.Network/networkSecurityGroups/securityRules/sourceAddressPrefix                                               
Microsoft.Network/networkSecurityGroups/securityRules/sourceAddressPrefixes[*]                                          
Microsoft.Network/networkSecurityGroups/securityRules/sourceApplicationSecurityGroups[*]                                
Microsoft.Network/networkSecurityGroups/securityRules/sourcePortRange                                                   
Microsoft.Network/networkSecurityGroups/securityRules/sourcePortRanges[*]
```
### Notification Hubs - 4
```
Microsoft.NotificationHubs/Namespaces/AuthorizationRules/rights                                                         
Microsoft.NotificationHubs/Namespaces/AuthorizationRules/rights[*]                                                      
Microsoft.NotificationHubs/namespaces/notificationHubs/authorizationRules/rights                                        
Microsoft.NotificationHubs/namespaces/notificationHubs/authorizationRules/rights[*]
```
### Operational Insights - 8
```
Microsoft.OperationalInsights/workspaces/retentionInDays                                                                
Microsoft.OperationalInsights/workspaces/sku.name                                                                       
Microsoft.OperationalInsights/workspaces/dataSources/counterName                                                        
Microsoft.OperationalInsights/workspaces/dataSources/eventLogName                                                       
Microsoft.OperationalInsights/workspaces/dataSources/eventTypes[*].eventType                                            
Microsoft.OperationalInsights/workspaces/dataSources/instanceName                                                       
Microsoft.OperationalInsights/workspaces/dataSources/intervalSeconds                                                    
Microsoft.OperationalInsights/workspaces/dataSources/objectName
```
### Postgre - 31
```
*NEW Microsoft.DBforPostgreSQL/servers/administratorLogin                                                                    
*NEW Microsoft.DBforPostgreSQL/servers/earliestRestoreDate                                                                   
*NEW Microsoft.DBforPostgreSQL/servers/fullyQualifiedDomainName                                                              
*NEW Microsoft.DBforPostgreSQL/servers/masterServerId                                                                        
*NEW Microsoft.DBforPostgreSQL/servers/replicaCapacity                                                                       
*NEW Microsoft.DBforPostgreSQL/servers/replicationRole                                                                       
*NEW Microsoft.DBforPostgreSQL/servers/sku.capacity                                                                          
*NEW Microsoft.DBforPostgreSQL/servers/sku.family                                                                            
*NEW Microsoft.DBforPostgreSQL/servers/sku.name                                                                              
*NEW Microsoft.DBforPostgreSQL/servers/sku.tier                                                                              
*NEW Microsoft.DBforPostgreSQL/servers/sslEnforcement                                                                        
*NEW Microsoft.DBforPostgreSQL/servers/storageProfile                                                                        
*NEW Microsoft.DBforPostgreSQL/servers/storageProfile.backupRetentionDays                                                    
*NEW Microsoft.DBforPostgreSQL/servers/storageProfile.geoRedundantBackup                                                     
*NEW Microsoft.DBforPostgreSQL/servers/storageProfile.storageMB                                                              
*NEW Microsoft.DBforPostgreSQL/servers/userVisibleState                                                                      
*NEW Microsoft.DBforPostgreSQL/servers/version                                                                               
*NEW Microsoft.DBforPostgreSQL/servers/firewallRules/endIpAddress                                                            
*NEW Microsoft.DBforPostgreSQL/servers/firewallRules/startIpAddress                                                          
*NEW Microsoft.DBforPostgreSQL/servers/securityAlertPolicies/disabledAlerts                                                  
*NEW Microsoft.DBforPostgreSQL/servers/securityAlertPolicies/disabledAlerts[*]                                               
*NEW Microsoft.DBforPostgreSQL/servers/securityAlertPolicies/emailAccountAdmins                                              
*NEW Microsoft.DBforPostgreSQL/servers/securityAlertPolicies/emailAddresses                                                  
*NEW Microsoft.DBforPostgreSQL/servers/securityAlertPolicies/emailAddresses[*]                                               
*NEW Microsoft.DBforPostgreSQL/servers/securityAlertPolicies/retentionDays                                                   
*NEW Microsoft.DBforPostgreSQL/servers/securityAlertPolicies/state                                                           
*NEW Microsoft.DBforPostgreSQL/servers/securityAlertPolicies/storageAccountAccessKey                                         
*NEW Microsoft.DBforPostgreSQL/servers/securityAlertPolicies/storageEndpoint                                                 
*NEW Microsoft.DBforPostgreSQL/servers/virtualNetworkRules/ignoreMissingVnetServiceEndpoint                                  
*NEW Microsoft.DBforPostgreSQL/servers/virtualNetworkRules/state                                                             
*NEW Microsoft.DBforPostgreSQL/servers/virtualNetworkRules/virtualNetworkSubnetId
```
### Public IP - 2
```
*NEW Microsoft.Network/publicIPAddresses/publicIPAllocationMethod
Microsoft.Network/publicIPAddresses/sku.name
```
### Redis Cache - 41
```
*NEW Microsoft.Cache/Redis/accessKeys                                                              
*NEW Microsoft.Cache/Redis/accessKeys.primaryKey                                                   
*NEW Microsoft.Cache/Redis/accessKeys.secondaryKey
Microsoft.Cache/Redis/enableNonSslPort
*NEW Microsoft.Cache/Redis/endIP                                                                   
*NEW Microsoft.Cache/Redis/hostName                                                                
*NEW Microsoft.Cache/Redis/linkedRedisCacheId                                                      
*NEW Microsoft.Cache/Redis/linkedRedisCacheLocation                                                
*NEW Microsoft.Cache/Redis/linkedServers                                                           
*NEW Microsoft.Cache/Redis/linkedServers.properties[*].id                                          
*NEW Microsoft.Cache/Redis/linkedServers[*]                                                        
*NEW Microsoft.Cache/Redis/linkedServers[*].id                                                     
*NEW Microsoft.Cache/Redis/minimumTlsVersion                                                       
*NEW Microsoft.Cache/Redis/port                                                                    
*NEW Microsoft.Cache/Redis/provisioningState                                                       
*NEW Microsoft.Cache/Redis/redisConfiguration                                                      
*NEW Microsoft.Cache/Redis/redisConfiguration.additionalProperties                                 
*NEW Microsoft.Cache/Redis/redisVersion                                                            
*NEW Microsoft.Cache/Redis/scheduleEntries[*]                                                      
*NEW Microsoft.Cache/Redis/scheduleEntries[*].dayOfWeek                                            
*NEW Microsoft.Cache/Redis/scheduleEntries[*].maintenanceWindow                                    
*NEW Microsoft.Cache/Redis/scheduleEntries[*].startHourUtc                                         
*NEW Microsoft.Cache/Redis/serverRole                                                              
*NEW Microsoft.Cache/Redis/shardCount                                                              
*NEW Microsoft.Cache/Redis/sku
Microsoft.Cache/Redis/shardCount
*NEW Microsoft.Cache/Redis/sku
Microsoft.Cache/Redis/sku.capacity                                                                                      
Microsoft.Cache/Redis/sku.family                                                                                        
Microsoft.Cache/Redis/sku.name
Microsoft.Cache/Redis/sslPort                                                                 
*NEW Microsoft.Cache/Redis/startIP                                                                 
*NEW Microsoft.Cache/Redis/staticIP                                                                
*NEW Microsoft.Cache/Redis/subnet                                                                  
*NEW *Microsoft.Cache/Redis/subnetId                                                                
*NEW Microsoft.Cache/Redis/tenantSettings                                                          
*NEW Microsoft.Cache/Redis/tenantSettings.additionalProperties                                     
*NEW Microsoft.Cache/Redis/virtualNetwork                                                          
*NEW Microsoft.Cache/Redis/zones[*]
Microsoft.Cache/Redis/firewallrules/endIP                                                                               
Microsoft.Cache/Redis/firewallrules/startIP    
```
### Resources - 3
```
Microsoft.Resources/links/notes
Microsoft.Resources/links/sourceId
Microsoft.Resources/links/targetId
```
### Role Assignments - 3
```
Microsoft.Authorization/roleAssignments/principalId                                                                     
Microsoft.Authorization/roleAssignments/principalType                                                                   
Microsoft.Authorization/roleAssignments/roleDefinitionId
```
### Role Definitions - 4
```
Microsoft.Authorization/roleDefinitions/permissions.actions[*]                                                          
Microsoft.Authorization/roleDefinitions/permissions.notActions[*]                                                       
Microsoft.Authorization/roleDefinitions/roleName                                                                        
Microsoft.Authorization/roleDefinitions/type       
```
### Scheduler - 1
```
Microsoft.Scheduler/jobcollections/sku.name
```
### Search - 4
```
Microsoft.Search/searchServices/hostingMode                                                                             
Microsoft.Search/searchServices/partitionCount                                                                          
Microsoft.Search/searchServices/replicaCount                                                                            
Microsoft.Search/searchServices/sku.name
```
### Security - 41
```
*NEW Microsoft.Security/advancedThreatProtectionSettings/isEnabled
Microsoft.Security/alerts/state                                                                                         
Microsoft.Security/complianceResults/resourceStatus                                                                     
Microsoft.Security/locations/alerts/state                                                                               
Microsoft.Security/policies/logCollection                                                                               
Microsoft.Security/policies/logsConfiguration                                                                           
Microsoft.Security/policies/logsConfiguration.storages                                                                  
Microsoft.Security/policies/name                                                                                        
Microsoft.Security/policies/omsWorkspaceConfiguration                                                                   
Microsoft.Security/policies/omsWorkspaceConfiguration.workspaces                                                        
Microsoft.Security/policies/policyLevel                                                                                 
Microsoft.Security/policies/pricingConfiguration                                                                        
Microsoft.Security/policies/pricingConfiguration.active                                                                 
Microsoft.Security/policies/pricingConfiguration.level                                                                  
Microsoft.Security/policies/pricingConfiguration.resourceGroup                                                          
Microsoft.Security/policies/pricingConfiguration.selectedPricingTier                                                    
Microsoft.Security/policies/pricingConfiguration.standardBundles                                                        
Microsoft.Security/policies/pricingConfiguration.standardBundles[*]                                                     
Microsoft.Security/policies/recommendations                                                                             
Microsoft.Security/policies/recommendations.acls                                                                        
Microsoft.Security/policies/recommendations.antimalware                                                                 
Microsoft.Security/policies/recommendations.appWhitelisting                                                             
Microsoft.Security/policies/recommendations.baseline                                                                    
Microsoft.Security/policies/recommendations.diskEncryption                                                              
Microsoft.Security/policies/recommendations.jitNetworkAccess                                                            
Microsoft.Security/policies/recommendations.ngfw                                                                        
Microsoft.Security/policies/recommendations.nsgs                                                                        
Microsoft.Security/policies/recommendations.patch                                                                       
Microsoft.Security/policies/recommendations.sqlAuditing                                                                 
Microsoft.Security/policies/recommendations.sqlTde                                                                      
Microsoft.Security/policies/recommendations.storageEncryption                                                           
Microsoft.Security/policies/recommendations.vulnerabilityAssessment                                                     
Microsoft.Security/policies/recommendations.waf                                                                         
Microsoft.Security/policies/securityContactConfiguration                                                                
Microsoft.Security/policies/securityContactConfiguration.areNotificationsOn                                             
Microsoft.Security/policies/securityContactConfiguration.securityContactEmails                                          
Microsoft.Security/policies/securityContactConfiguration.securityContactEmails[*]                                       
Microsoft.Security/policies/securityContactConfiguration.securityContactPhone                                           
Microsoft.Security/policies/securityContactConfiguration.sendToAdminOn                                                  
Microsoft.Security/policies/unique                                                                                      
Microsoft.Security/pricings/pricingTier
```
### Service Fabric - 6
```
Microsoft.ServiceFabric/clusters/azureActiveDirectory.tenantId                                                          
Microsoft.ServiceFabric/clusters/certificate.thumbprint                                                                 
Microsoft.ServiceFabric/clusters/certificate.x509StoreName                                                              
Microsoft.ServiceFabric/clusters/fabricSettings[*].name                                                                 
Microsoft.ServiceFabric/clusters/fabricSettings[*].parameters[*].name                                                   
Microsoft.ServiceFabric/clusters/fabricSettings[*].parameters[*].value
```
### SQL - 127
```
*NEW Microsoft.Sql/managedInstances/licenseType                                                                              
*NEW Microsoft.Sql/managedInstances/databases/securityAlertPolicies/disabledAlerts                                           
*NEW Microsoft.Sql/managedInstances/databases/securityAlertPolicies/disabledAlerts[*]                                        
*NEW Microsoft.Sql/managedInstances/databases/securityAlertPolicies/emailAccountAdmins                                       
*NEW Microsoft.Sql/managedInstances/databases/securityAlertPolicies/emailAddresses                                           
*NEW Microsoft.Sql/managedInstances/databases/securityAlertPolicies/emailAddresses[*]                                        
*NEW Microsoft.Sql/managedInstances/databases/securityAlertPolicies/retentionDays                                            
*NEW Microsoft.Sql/managedInstances/databases/securityAlertPolicies/state                                                    
*NEW Microsoft.Sql/managedInstances/databases/securityAlertPolicies/storageEndpoint                                          
*NEW Microsoft.Sql/managedInstances/databases/vulnerabilityAssessments/endTime                                               
*NEW Microsoft.Sql/managedInstances/databases/vulnerabilityAssessments/numberOfFailedSecurityChecks                          
*NEW Microsoft.Sql/managedInstances/databases/vulnerabilityAssessments/recurringScans                                        
*NEW Microsoft.Sql/managedInstances/databases/vulnerabilityAssessments/recurringScans.emails                                 
*NEW Microsoft.Sql/managedInstances/databases/vulnerabilityAssessments/recurringScans.emails[*]                              
*NEW Microsoft.Sql/managedInstances/databases/vulnerabilityAssessments/recurringScans.emailSubscriptionAdmins                
*NEW Microsoft.Sql/managedInstances/databases/vulnerabilityAssessments/recurringScans.isEnabled                              
*NEW Microsoft.Sql/managedInstances/databases/vulnerabilityAssessments/scanId                                                
*NEW Microsoft.Sql/managedInstances/databases/vulnerabilityAssessments/startTime                                             
*NEW Microsoft.Sql/managedInstances/databases/vulnerabilityAssessments/state                                                 
*NEW Microsoft.Sql/managedInstances/databases/vulnerabilityAssessments/storageAccountAccessKey                               
*NEW Microsoft.Sql/managedInstances/databases/vulnerabilityAssessments/storageContainerPath                                  
*NEW Microsoft.Sql/managedInstances/databases/vulnerabilityAssessments/triggerType                                           
*NEW Microsoft.Sql/managedInstances/securityAlertPolicies/disabledAlerts                                                     
*NEW Microsoft.Sql/managedInstances/securityAlertPolicies/disabledAlerts[*]                                                  
*NEW Microsoft.Sql/managedInstances/securityAlertPolicies/emailAccountAdmins                                                 
*NEW Microsoft.Sql/managedInstances/securityAlertPolicies/emailAddresses                                                     
*NEW Microsoft.Sql/managedInstances/securityAlertPolicies/emailAddresses[*]                                                  
*NEW Microsoft.Sql/managedInstances/securityAlertPolicies/retentionDays                                                      
*NEW Microsoft.Sql/managedInstances/securityAlertPolicies/state                                                              
*NEW Microsoft.Sql/managedInstances/securityAlertPolicies/storageEndpoint                                                    
*NEW Microsoft.Sql/managedInstances/vulnerabilityAssessments/recurringScans                                                  
*NEW Microsoft.Sql/managedInstances/vulnerabilityAssessments/recurringScans.emails                                           
*NEW Microsoft.Sql/managedInstances/vulnerabilityAssessments/recurringScans.emails[*]                                        
*NEW Microsoft.Sql/managedInstances/vulnerabilityAssessments/recurringScans.emailSubscriptionAdmins                          
*NEW Microsoft.Sql/managedInstances/vulnerabilityAssessments/recurringScans.isEnabled                                        
*NEW Microsoft.Sql/managedInstances/vulnerabilityAssessments/storageAccountAccessKey                                         
*NEW Microsoft.Sql/managedInstances/vulnerabilityAssessments/storageContainerPath
Microsoft.Sql/servers/version                                                                                           
Microsoft.Sql/servers/administrators/administratorType                                                                  
Microsoft.Sql/servers/administrators/login                                                                              
Microsoft.Sql/servers/administrators/sid                                                                                
Microsoft.Sql/servers/administrators/tenantId                                                                           
Microsoft.Sql/auditingSettings.state   
*NEW Microsoft.Sql/servers/auditingSettings/auditActionsAndGroups[*]                                                                                  
Microsoft.Sql/servers/auditingSettings/retentionDays                                                                    
Microsoft.Sql/servers/auditingSettings/state                                                                            
Microsoft.Sql/servers/auditingSettings/storageEndpoint                                                                  
*NEW Microsoft.Sql/automaticTuning.state 
Microsoft.Sql/servers/automaticTuning/desiredState                                                                      
Microsoft.Sql/servers/automaticTuning/options.createIndex                                                               
Microsoft.Sql/servers/automaticTuning/options.dropIndex                                                                 
Microsoft.Sql/servers/automaticTuning/options.forceLastGoodPlan                                                         
Microsoft.Sql/servers/connectionPolicies/connectionType   
*NEW Microsoft.Sql/servers/databases/currentSku                                                                              
*NEW Microsoft.Sql/servers/databases/currentSku.capacity                                                                     
*NEW Microsoft.Sql/servers/databases/currentSku.family                                                                       
*NEW Microsoft.Sql/servers/databases/currentSku.name                                                                         
*NEW Microsoft.Sql/servers/databases/currentSku.size                                                                         
*NEW Microsoft.Sql/servers/databases/currentSku.tier
Microsoft.Sql/servers/databases/edition                                                                                 
Microsoft.Sql/servers/databases/elasticPoolName     
*NEW Microsoft.Sql/servers/databases/licenseType                                                                    
Microsoft.Sql/servers/databases/requestedServiceObjectiveId                                                             
Microsoft.Sql/servers/databases/requestedServiceObjectiveName                                                           
Microsoft.Sql/auditingSettings.state                                                                                    
Microsoft.Sql/servers/databases/auditingSettings/retentionDays                                                          
Microsoft.Sql/servers/databases/auditingSettings/state                                                                  
Microsoft.Sql/servers/databases/auditingSettings/storageEndpoint
*NEW Microsoft.Sql/automaticTuning.state                                                        
Microsoft.Sql/servers/databases/automaticTuning/desiredState                                                            
Microsoft.Sql/servers/databases/automaticTuning/options.createIndex                                                     
Microsoft.Sql/servers/databases/automaticTuning/options.dropIndex                                                       
Microsoft.Sql/servers/databases/automaticTuning/options.forceLastGoodPlan                                               
Microsoft.Sql/servers/databases/connectionPolicies/proxyDnsName                                                         
Microsoft.Sql/servers/databases/connectionPolicies/proxyPort                                                            
Microsoft.Sql/servers/databases/connectionPolicies/redirectionState                                                     
Microsoft.Sql/servers/databases/connectionPolicies/securityEnabledAccess                                                
Microsoft.Sql/servers/databases/connectionPolicies/state                                                                
Microsoft.Sql/servers/databases/connectionPolicies/useServerDefault                                                     
Microsoft.Sql/servers/databases/connectionPolicies/visibility                                                           
Microsoft.Sql/securityAlertPolicies.disabledAlerts                                                                      
Microsoft.Sql/securityAlertPolicies.emailAccountAdmins                                                                  
Microsoft.Sql/securityAlertPolicies.state                                                                               
Microsoft.Sql/servers/databases/securityAlertPolicies/disabledAlerts                                                    
Microsoft.Sql/servers/databases/securityAlertPolicies/emailAccountAdmins                                                
Microsoft.Sql/servers/databases/securityAlertPolicies/emailAddresses                                                    
Microsoft.Sql/servers/databases/securityAlertPolicies/retentionDays                                                     
Microsoft.Sql/servers/databases/securityAlertPolicies/state                                                             
Microsoft.Sql/servers/databases/securityAlertPolicies/storageEndpoint                                                   
Microsoft.Sql/transparentDataEncryption.status 
*NEW Microsoft.Sql/servers/databases/vulnerabilityAssessments/endTime                                                        
*NEW Microsoft.Sql/servers/databases/vulnerabilityAssessments/numberOfFailedSecurityChecks                                   
*NEW Microsoft.Sql/servers/databases/vulnerabilityAssessments/recurringScans                                                 
*NEW Microsoft.Sql/servers/databases/vulnerabilityAssessments/recurringScans.emails                                          
*NEW Microsoft.Sql/servers/databases/vulnerabilityAssessments/recurringScans.emails[*]                                       
*NEW Microsoft.Sql/servers/databases/vulnerabilityAssessments/recurringScans.emailSubscriptionAdmins                         
*NEW Microsoft.Sql/servers/databases/vulnerabilityAssessments/recurringScans.isEnabled                                       
*NEW Microsoft.Sql/servers/databases/vulnerabilityAssessments/scanId                                                         
*NEW Microsoft.Sql/servers/databases/vulnerabilityAssessments/startTime                                                      
*NEW Microsoft.Sql/servers/databases/vulnerabilityAssessments/state                                                          
*NEW Microsoft.Sql/servers/databases/vulnerabilityAssessments/storageAccountAccessKey                                        
*NEW Microsoft.Sql/servers/databases/vulnerabilityAssessments/storageContainerPath                                           
*NEW Microsoft.Sql/servers/databases/vulnerabilityAssessments/triggerType
Microsoft.Sql/servers/elasticPools/dtu                                                                                  
Microsoft.Sql/servers/elasticPools/edition                                                                              
Microsoft.Sql/servers/firewallRules/endIpAddress                                                                        
Microsoft.Sql/servers/firewallRules/startIpAddress                                                                      
Microsoft.Sql/securityAlertPolicies.disabledAlerts                                                                      
Microsoft.Sql/securityAlertPolicies.emailAccountAdmins                                                                  
Microsoft.Sql/securityAlertPolicies.state                                                                               
Microsoft.Sql/servers/securityAlertPolicies/disabledAlerts                                                              
Microsoft.Sql/servers/securityAlertPolicies/disabledAlerts[*]                                                           
Microsoft.Sql/servers/securityAlertPolicies/emailAccountAdmins                                                          
Microsoft.Sql/servers/securityAlertPolicies/emailAddresses                                                              
Microsoft.Sql/servers/securityAlertPolicies/emailAddresses[*]                                                           
Microsoft.Sql/servers/securityAlertPolicies/retentionDays                                                               
Microsoft.Sql/servers/securityAlertPolicies/state                                                                       
Microsoft.Sql/servers/securityAlertPolicies/storageEndpoint                                                             
Microsoft.Sql/servers/virtualNetworkRules/ignoreMissingVnetServiceEndpoint                                              
Microsoft.Sql/servers/virtualNetworkRules/virtualNetworkSubnetId
*NEW Microsoft.Sql/servers/vulnerabilityAssessments/recurringScans                                                           
*NEW Microsoft.Sql/servers/vulnerabilityAssessments/recurringScans.emails                                                    
*NEW Microsoft.Sql/servers/vulnerabilityAssessments/recurringScans.emails[*]                                                 
*NEW Microsoft.Sql/servers/vulnerabilityAssessments/recurringScans.emailSubscriptionAdmins                                   
*NEW Microsoft.Sql/servers/vulnerabilityAssessments/recurringScans.isEnabled                                                 
*NEW Microsoft.Sql/servers/vulnerabilityAssessments/storageAccountAccessKey                                                  
*NEW Microsoft.Sql/servers/vulnerabilityAssessments/storageContainerPath
```
### Storage Account - 39
```
Microsoft.Storage/storageAccounts/accessTier                                                                            
Microsoft.Storage/storageAccounts/accountType                                                                           
Microsoft.Storage/storageAccounts/enableBlobEncryption                                                                  
Microsoft.Storage/storageAccounts/enableFileEncryption
*NEW Microsoft.Storage/storageAccounts/encryption                                                                            
*NEW Microsoft.Storage/storageAccounts/encryption.keySource                                                                  
*NEW Microsoft.Storage/storageAccounts/encryption.keyvaultproperties.keyname                                                 
*NEW Microsoft.Storage/storageAccounts/encryption.keyvaultproperties.keyvaulturi                                             
*NEW Microsoft.Storage/storageAccounts/encryption.keyvaultproperties.keyversion                                              
*NEW Microsoft.Storage/storageAccounts/encryption.services                                                                   
*NEW Microsoft.Storage/storageAccounts/encryption.services.blob                                                              
*NEW Microsoft.Storage/storageAccounts/encryption.services.blob.enabled                                                      
*NEW Microsoft.Storage/storageAccounts/encryption.services.file                                                              
*NEW Microsoft.Storage/storageAccounts/encryption.services.file.enabled                                                      
*NEW Microsoft.Storage/storageAccounts/isHnsEnabled                                                                          
*NEW Microsoft.Storage/storageAccounts/networkAcls                                                                           
*NEW Microsoft.Storage/storageAccounts/networkAcls.bypass                      
Microsoft.Storage/storageAccounts/networkAcls.defaultAction                                                             
Microsoft.Storage/storageAccounts/networkAcls.ipRules                                                                   
Microsoft.Storage/storageAccounts/networkAcls.ipRules[*]
*NEW Microsoft.Storage/storageAccounts/networkAcls.ipRules[*].action                                                                
Microsoft.Storage/storageAccounts/networkAcls.ipRules[*].value                                                          
Microsoft.Storage/storageAccounts/networkAcls.virtualNetworkRules                                                       
Microsoft.Storage/storageAccounts/networkAcls.virtualNetworkRules[*]
*NEW Microsoft.Storage/storageAccounts/networkAcls.virtualNetworkRules[*].action                                                    
Microsoft.Storage/storageAccounts/networkAcls.virtualNetworkRules[*].id
*NEW Microsoft.Storage/storageAccounts/networkAcls.virtualNetworkRules[*].state
*NEW Microsoft.Storage/storageAccounts/primaryEndpoints                                                                      
*NEW Microsoft.Storage/storageAccounts/primaryEndpoints.blob                                                                 
*NEW Microsoft.Storage/storageAccounts/primaryEndpoints.file                                                                 
*NEW Microsoft.Storage/storageAccounts/primaryEndpoints.queue                                                                
*NEW Microsoft.Storage/storageAccounts/primaryEndpoints.table                                                                
*NEW Microsoft.Storage/storageAccounts/primaryEndpoints.web                                                                  
*NEW Microsoft.Storage/storageAccounts/primaryLocation                                                                       
*NEW Microsoft.Storage/storageAccounts/provisioningState
Microsoft.Storage/storageAccounts/sku.name
*NEW Microsoft.Storage/storageAccounts/sku.tier                                                                              
*NEW Microsoft.Storage/storageAccounts/statusOfPrimary                                                                              
Microsoft.Storage/storageAccounts/supportsHttpsTrafficOnly
```
### Terraform OSS - 1
```
Microsoft.TerraformOSS/resources/image
```
### Traffic Manager - 1
```
Microsoft.Network/trafficmanagerprofiles/monitorConfig.protocol
```
### Virtual Machine - 45
```
Microsoft.Compute/imageId                                                                                               
Microsoft.Compute/imageOffer                                                                                            
Microsoft.Compute/imagePublisher                                                                                        
Microsoft.Compute/imageSku                                                                                              
Microsoft.Compute/imageVersion                                                                                          
Microsoft.Compute/licenseType                                                                                           
Microsoft.Compute/virtualMachines/availabilitySet.id                                                                    
Microsoft.Compute/virtualMachines/diagnosticsProfile.bootDiagnostics                                                    
Microsoft.Compute/virtualMachines/diagnosticsProfile.bootDiagnostics.enabled                                            
Microsoft.Compute/virtualMachines/diagnosticsProfile.bootDiagnostics.storageUri                                         
Microsoft.Compute/virtualMachines/imageOffer                                                                            
Microsoft.Compute/virtualMachines/imagePublisher                                                                        
Microsoft.Compute/virtualMachines/imageSku                                                                              
Microsoft.Compute/virtualMachines/imageVersion                                                                          
Microsoft.Compute/virtualMachines/networkInterfaces[*].id                                                               
Microsoft.Compute/virtualMachines/osDisk.Uri                                                                            
Microsoft.Compute/virtualMachines/osProfile.adminPassword                                                               
Microsoft.Compute/virtualMachines/osProfile.adminUsername
*NEW Microsoft.Compute/virtualMachines/osProfile.customData                                                               
Microsoft.Compute/virtualMachines/osProfile.linuxConfiguration                                                          
Microsoft.Compute/virtualMachines/osProfile.linuxConfiguration.disablePasswordAuthentication                            
Microsoft.Compute/virtualMachines/osProfile.windowsConfiguration                                                        
Microsoft.Compute/virtualMachines/osProfile.windowsConfiguration.enableAutomaticUpdates                                 
Microsoft.Compute/virtualMachines/osProfile.windowsConfiguration.provisionVMAgent                                       
*NEW Microsoft.Compute/virtualMachines/provisioningState                                      
Microsoft.Compute/virtualMachines/sku.name
Microsoft.Compute/virtualMachines/storageProfile.dataDisks[*].caching                                                   
Microsoft.Compute/virtualMachines/storageProfile.dataDisks[*].createOption                                              
Microsoft.Compute/virtualMachines/storageProfile.dataDisks[*].diskSizeGB                                                
Microsoft.Compute/virtualMachines/storageProfile.dataDisks[*].image.uri                                                 
Microsoft.Compute/virtualMachines/storageProfile.dataDisks[*].lun                                                       
Microsoft.Compute/virtualMachines/storageProfile.dataDisks[*].managedDisk.id                                            
Microsoft.Compute/virtualMachines/storageProfile.dataDisks[*].managedDisk.storageAccountType                            
Microsoft.Compute/virtualMachines/storageProfile.dataDisks[*].name                                                      
Microsoft.Compute/virtualMachines/storageProfile.dataDisks[*].vhd.uri                                                   
Microsoft.Compute/virtualMachines/storageProfile.osDisk.caching                                                         
Microsoft.Compute/virtualMachines/storageProfile.osDisk.createOption                                                    
Microsoft.Compute/virtualMachines/storageProfile.osDisk.encryptionSettings                                              
Microsoft.Compute/virtualMachines/storageProfile.osDisk.encryptionSettings.enabled                                      
Microsoft.Compute/virtualMachines/storageProfile.osDisk.managedDisk.id                                                  
Microsoft.Compute/virtualMachines/storageProfile.osDisk.managedDisk.storageAccountType                                  
Microsoft.Compute/virtualMachines/storageProfile.osDisk.name                                                            
Microsoft.Compute/virtualMachines/storageProfile.osDisk.osType                                                          
Microsoft.Compute/virtualMachines/storageProfile.osDisk.vhd                                                             
Microsoft.Compute/virtualMachines/storageProfile.osDisk.vhd.uri
```
### Virtual Machine Extension - 23
```
Microsoft.Compute/virtualMachines/extensions/autoUpgradeMinorVersion
*NEW Microsoft.Compute/virtualMachines/extensions/enableAutomaticUpgrade                                                    
Microsoft.Compute/virtualMachines/extensions/provisioningState                                                          
Microsoft.Compute/virtualMachines/extensions/publisher                  
*NEW Microsoft.Compute/virtualMachines/extensions/resources
*NEW Microsoft.Compute/virtualMachines/extensions/resources[*]
*NEW Microsoft.Compute/virtualMachines/extensions/resources[*].autoUpgradeMinorVersion
*NEW Microsoft.Compute/virtualMachines/extensions/resources[*].forceUpdateTag
*NEW Microsoft.Compute/virtualMachines/extensions/resources[*].id
*NEW Microsoft.Compute/virtualMachines/extensions/resources[*].location
*NEW Microsoft.Compute/virtualMachines/extensions/resources[*].name
*NEW Microsoft.Compute/virtualMachines/extensions/resources[*].protectedSettings
*NEW Microsoft.Compute/virtualMachines/extensions/resources[*].provisioningState
*NEW Microsoft.Compute/virtualMachines/extensions/resources[*].publisher
*NEW Microsoft.Compute/virtualMachines/extensions/resources[*].settings
*NEW Microsoft.Compute/virtualMachines/extensions/resources[*].tags
*NEW Microsoft.Compute/virtualMachines/extensions/resources[*].tags.additionalProperties
*NEW Microsoft.Compute/virtualMachines/extensions/resources[*].type
*NEW Microsoft.Compute/virtualMachines/extensions/resources[*].typeHandlerVersion
Microsoft.Compute/virtualMachines/extensions/settings                                                                   
Microsoft.Compute/virtualMachines/extensions/settings.workspaceId                                                       
Microsoft.Compute/virtualMachines/extensions/type                                                                       
Microsoft.Compute/virtualMachines/extensions/typeHandlerVersion
```
### Virtual Machine Scale Sets - 38
```
Microsoft.Compute/imageId                                                                                               
Microsoft.Compute/imageOffer                                                                                            
Microsoft.Compute/imagePublisher                                                                                        
Microsoft.Compute/imageSku                                                                                              
Microsoft.Compute/imageVersion                                                                                          
Microsoft.Compute/licenseType                                                                                           
Microsoft.Compute/VirtualMachineScaleSets/computerNamePrefix   
*NEW Microsoft.Compute/VirtualMachineScaleSets/extensionProfile.extensions[*]                                                
*NEW Microsoft.Compute/VirtualMachineScaleSets/extensionProfile.extensions[*].autoUpgradeMinorVersion                        
*NEW Microsoft.Compute/VirtualMachineScaleSets/extensionProfile.extensions[*].enableAutomaticUpgrade                         
*NEW Microsoft.Compute/VirtualMachineScaleSets/extensionProfile.extensions[*].name                                           
*NEW Microsoft.Compute/VirtualMachineScaleSets/extensionProfile.extensions[*].provisioningState                              
*NEW Microsoft.Compute/VirtualMachineScaleSets/extensionProfile.extensions[*].publisher                                      
*NEW Microsoft.Compute/VirtualMachineScaleSets/extensionProfile.extensions[*].settings                                       
*NEW Microsoft.Compute/VirtualMachineScaleSets/extensionProfile.extensions[*].settings.workspaceId                           
*NEW Microsoft.Compute/VirtualMachineScaleSets/extensionProfile.extensions[*].type                                           
*NEW Microsoft.Compute/VirtualMachineScaleSets/extensionProfile.extensions[*].typeHandlerVersion
Microsoft.Compute/VirtualMachineScaleSets/networkInterfaceConfigurations[*].enableIPForwarding                          
Microsoft.Compute/VirtualMachineScaleSets/networkInterfaceConfigurations[*].ipConfigurations[*].subnet.id               
Microsoft.Compute/VirtualMachineScaleSets/networkInterfaceConfigurations[*].networkSecurityGroup.id                     
Microsoft.Compute/VirtualMachineScaleSets/networkProfile.healthProbe.id                                                 
Microsoft.Compute/VirtualMachineScaleSets/osDisk.caching                                                                
Microsoft.Compute/VirtualMachineScaleSets/osDisk.createOption                                                           
Microsoft.Compute/VirtualMachineScaleSets/osdisk.imageUrl                                                               
Microsoft.Compute/VirtualMachineScaleSets/osDisk.managedDisk.storageAccountType                                         
Microsoft.Compute/VirtualMachineScaleSets/osDisk.name                                                                   
Microsoft.Compute/VirtualMachineScaleSets/osdisk.vhdContainers                                                          
Microsoft.Compute/VirtualMachineScaleSets/osProfile.adminPassword                                                       
Microsoft.Compute/VirtualMachineScaleSets/osProfile.adminUsername                                                       
Microsoft.Compute/VirtualMachineScaleSets/osProfile.linuxConfiguration                                                  
Microsoft.Compute/VirtualMachineScaleSets/osProfile.linuxConfiguration.disablePasswordAuthentication                    
Microsoft.Compute/VirtualMachineScaleSets/osProfile.windowsConfiguration                                                
Microsoft.Compute/VirtualMachineScaleSets/osProfile.windowsConfiguration.enableAutomaticUpdates                         
Microsoft.Compute/VirtualMachineScaleSets/osProfile.windowsConfiguration.provisionVMAgent                               
Microsoft.Compute/VirtualMachineScaleSets/sku.name                                                                      
Microsoft.Compute/VirtualMachineScaleSets/sku.tier                                                                      
Microsoft.Compute/VirtualMachineScaleSets/upgradePolicy.automaticOSUpgrade
*NEW Microsoft.Compute/VirtualMachineScaleSets/virtualMachineProfile 
```
### Virtual Machine Scale Sets Extensions - 9
```
Microsoft.Compute/virtualMachineScaleSets/extensions/autoUpgradeMinorVersion  
*NEW Microsoft.Compute/virtualMachineScaleSets/extensions/enableAutomaticUpgrade                                          
Microsoft.Compute/virtualMachineScaleSets/extensions/provisioningState                                                  
Microsoft.Compute/virtualMachineScaleSets/extensions/publisher                                                          
Microsoft.Compute/virtualMachineScaleSets/extensions/settings                                                           
Microsoft.Compute/virtualMachineScaleSets/extensions/settings.workspaceId                                               
Microsoft.Compute/virtualMachineScaleSets/extensions/type                                                               
Microsoft.Compute/virtualMachineScaleSets/extensions/typeHandlerVersion
*NEW Microsoft.Compute/virtualMachineScaleSets/virtualMachines/osProfile.customData
```
### Virtual Network - 17
```
Microsoft.Network/virtualNetworks/addressSpace.addressPrefixes[*]                                                       
Microsoft.Network/virtualNetworks/ddosProtectionPlan.id                                                                 
Microsoft.Network/virtualNetworks/subnets[*].addressPrefix                                                              
Microsoft.Network/virtualNetworks/subnets[*].ipConfigurations[*].id                                                     
Microsoft.Network/virtualNetworks/subnets[*].name                                                                       
Microsoft.Network/virtualNetworks/subnets[*].networkSecurityGroup.id                                                    
Microsoft.Network/virtualNetworks/subnets[*].routeTable.id                                                              
Microsoft.Network/virtualNetworks/subnets[*].serviceEndpoints[*].locations[*]                                           
Microsoft.Network/virtualNetworks/subnets[*].serviceEndpoints[*].service                                                
Microsoft.Network/virtualNetworks/subnets/addressPrefix                                                                 
Microsoft.Network/virtualNetworks/subnets/ipConfigurations[*].id                                                        
Microsoft.Network/virtualNetworks/subnets/networkSecurityGroup.id                                                       
Microsoft.Network/virtualNetworks/subnets/routeTable.id                                                                 
Microsoft.Network/virtualNetworks/subnets/serviceEndpoints[*].locations[*]                                              
Microsoft.Network/virtualNetworks/subnets/serviceEndpoints[*].service                                                   
Microsoft.Network/virtualNetworks/virtualNetworkPeerings/remoteVirtualNetwork.id
*NEW Microsoft.Network/virtualNetworks/virtualNetworkPeerings/useRemoteGateways
```
### Virtual Network Gateway - 2
```
Microsoft.Network/virtualNetworkGateways/gatewayType                                                                    
Microsoft.Network/virtualNetworkGateways/sku.name
```



Tags: #Azure #AzurePolicy #Policy #ResourceGroup #Resource #AzureSecurity #AzureCompliance 