---
layout: post
title:  "Azure Policy"
categories: Azure Policy Resource Security
author: Marcin Krawczyk
---

# How to write custom Azure Policy
Last update: 2018.11.04

On the Internet you can find a lots of Azure Policy script, but there always may be situation when you need to create your own custom policy. So I decided to write this article about it. 

<b>PLEASE DON’T USE DENY POLICY WHEN YOU ARE CREATING FIRST POLICY. USE AUDIT!!!</b>

It is written in the JSON format. You have to know how to build section, what kind if field you can use (unfortunately not all you would like to). At the time of writing this article there is about 700 custom field you can use to build custom policy. You can use Azure Policy directly for those services:<br>
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
There is also possibility to write custom Policy for different type like Event Hub when you are not able to use dedicated properties for some properties.

# How to begin?
First of all you have to choose JSON editor or write custom policy in Azure portal. If you’ll be writing custom policy in Azure portal, then you will get autocomplete fields for some value like aliases (later about it). If you would like to built custom policy in JSON editor, then you can choose Visual Studio or Visual Studio Code. Here I’ll work on Visual Studio 2017 with Azure SDK installed. For the beginning I’ll create new JSON file:

<b>Select File -> New -> File -> Web -> JSON File</b>

Then you should insert schema definition for Azure Policy into Visual Studio. The newest version for policy definition is 2018-05-01. 

https://schema.management.azure.com/schemas/2018-05-01/policyDefinition.json

!({{ site.baseurl }}/images/Azure-Policy/AzurePolicy01.png "")


So when you do that, you can easily write custom policy with autocomplete options 😊

!({{ site.baseurl }}/images/Azure-Policy/AzurePolicy02.png "") 

# Azure Policy definition structure
When you create a custom policy you have to decide how you implemented them. You have couple of possibility:
<ul>
<li>Azure Portal</li>
<li>PowerShell</li>
<li>Azure CLI</li>
</ul>

There is a difference between them. When you are using Azure Portal, then you don’t need to add couple policy properties like displayName, description and so on. Below you can find sample policy which ensure that only https traffic is enable for storage account. This policy is created for PowerShell and CLI:

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

                        "equals": "Microsoft.Storage/storageAccounts"

                    },

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


The same script for Azure Portal deployment will look like this:

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

I strongly recommended to set mode to All in most case. There is only one situation, when you create a policy for tags and location, when you would like to use indexed mode.  Be careful when you copy Azure Policy from Internet. There are quite a lots of situation when Mode is set to indexed!!! This may also happened when you use default definition in Azure Portal (see Mode value below)!!!! So please DON’T USE built-in policy with Mode Indexed. Duplicate and create your own!!!
!({{ site.baseurl }}/images/Azure-Policy/AzurePolicy03.png "")
!({{ site.baseurl }}/images/Azure-Policy/AzurePolicy04.png "")
<b>IMPORTANT:
When you not specify mode parameter manually, then this value would be directly assign by Azure. Automatically assign value is different when you are using Powershell and CLI. Below you can find default value for them:
<ul>
<li>Powershell – By default Mode is set to ALL</li>
<li>Azure CLI – By default Mode is set to NULL, which is equivalent to indexed!!! Please change it.</li>
<li>Azure Portal - By default Mode is set to ALL</li>
<li>Created By Microsoft Built-in – A lots of MODE is set to INDEXED!!!!!!</b></li>
</ul>

## Parameters
You can use Parameters when you would like to choose a value from Azure. For instant you would like to choose Allowed location or Allowed Region in Azure, then you create Parameters, where after assigned to policy you choose exact value from Azure. There are ONLY some types of Parameters which you can use:
<ul>
<li>"location"</li>
<li>"resourceTypes”</li>
<li>"storageSkus”</li>
<li>"vmSKUs”</li>
<li>"existingResourceGroups”</li>
<li>"omsWorkspace”</li>

</ul>
Below you can find an example for Parameters:

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


## Display Name
Used only when you create Azure Policy in Powershell or Azure CLI. It’s a name of policy.

## Description
Used only when you create Azure Policy in Powershell or Azure CLI. It’s a description of policy.

## Policy rule
Policy rule is consists If and Then section. Policy rule look like that:


{
    "if": {

        <condition> | <logical operator>

    },

    "then": {

        "effect": "deny | audit | append | auditIfNotExists | deployIfNotExists"

    }

}


You may use couple of logical operators like:
<ul>
<li>"not”:{}</li>
<li>"allOf”:[{},{}] – working like logical AND</li>
<li>"anyOf”:[{},{}] – working like logical OR</li>
</ul>

A Conditions are used for field criteria. Supported conditions are:
<ul>
<li>contains
<li>containsKeys
<li>equals
<li>exists
<li>in
<li>like
<li>match
<li>notContains
<li>notContainsKeys
<li>notEquals
<li>notIn
<li>notLike
<li>notMatch
</ul>
When you are using Like and notLike, you can use * in the value. There may be ONLY one wildcard * in criteria.
When you are using march and notMatch, you can use
<ul>
<li># - represent a digit
<li>? – represent a letter
<li>. – match all characters
</ul>

## Fields

Fields represents properties for resource you choose. Those fields value are supported:
<ul>
<li>aliases
<li>name
<li>fullName
<li>kind
<li>type
<li>location
<li>tags
<li>source – with ACTION – UNDOCUMENTED 😊 – You can AUDIT OR DENY SOME ACTION ON AZURE PORTAL 😊 Yes You can write custom policy with audit or deny Azure action. The whole list of action you can find here: https://docs.microsoft.com/en-us/azure/role-based-access-control/resource-provider-operations
</ul>
<b>name</b> – The name of resource which you use when created policy

<b>fullName</b> – The name of resource with parent object

<b>kind</b> – depends on type resources

<b>type</b> – Some types example: "Microsoft.Storage/storageAccounts", "Microsoft.Network/virtualNetworks", "Microsoft.Network/publicIPAddresses", "Microsoft.Network/networkSecurityGroups", "Microsoft.Sql/servers”, "Microsoft.EventGrid/topics", "Microsoft.KeyVault/vaults" and many more

<b>location</b> – "westeurope”, "northeurope” and of course many more

<b>Source</b> - Action – Undocumented one. You can use block or audit some Azure action if you wish. It may be useful when you don’t won’t to create custom RBAC role or deny some action for all users (even Owners).

This is Azure Policy example for Action, when I would like to block creation of VM Extension:

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

<b>Aliases</b> – The value for specific service properties. At time I write this article there is 696 aliases created. You can list them by running this script:

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



Ok, I have 696 possible value, but how can I know which fields values are possible. The best approach is the set properly all those value for such resource and then:

<ul>
<li>We can open Resource Explorer - https://resources.azure.com and get value for resource
<li>We can use Azure Resource Graph – You can read about it on Michal Smereczynski MVP blog - https://lnx.azurewebsites.net/saving-time-with-azure-resource-graph
<li>We can use Resource Explorer in Azure Portal – All services -> Resource Explorer
</ul>

At the end of this article I’ll put the whole list of Aliases which you can use. You can always ask for specific aliases if you can’t find something very useful for you. If so, please go to such URL:

https://github.com/Azure/azure-policy/blob/master/1-contribution-guide/request-alias.md

## Effect
Azure Policy supports the following effect:
<ul>
<li>Deny – Unable to create resource when policy is enable. If you create policy and you had some resources already created, then they will be on non-compliant list.
<li>Audit – Inform you about non-compliant resources
<li>Append – adds extra fields 
<li>AuditIfNotExists – Inform you about something is not exist
<li>DeployIfNotExists – deploy something if it is not exist. For example VM extension when it doesn’t exist
</ul>
<b>PLEASE DON’T USE DENY POLICY WHEN YOU ARE CREATING FIRST POLICY. USE AUDIT!!!</b>




## WHEN DENY IS NOT DENYING ☹
There is a situation when Deny policy doesn’t work properly. Let me show you that on example. I would like to prevent creating of Storage Account without network ACLS (Firewall) enabled. I create such policy:

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


As you can see, there is deny effect enabled. So we can assume that we can’t create Storage Account without Network ACLS. What will really happened we assign this policy??? We will create it and after that it’ll be NON-Compliant resource. WHY THIS HAPPENED???? Because when you are creating such resource there is no information about networkAcls.defaultAction is template!!!  

So, we would like to write a custom script for Deny create a Storage Account when there is no secure transfer enabled. I use Resource Explorer in portal and get such information below. 
!({{ site.baseurl }}/images/Azure-Policy/AzurePolicy05.png "")
So it’s very easy for me to write policy based on aliases which I know.

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


## Services Aliases:
### Analysis Services
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

### App Service
Microsoft.Web/hostingEnvironments/clusterSettings[*].name                                                               
Microsoft.Web/hostingEnvironments/clusterSettings[*].value                                                              
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

### Application Gateway
Microsoft.Network/applicationGateways/sku.capacity                                                                      
Microsoft.Network/applicationGateways/sku.name                                                                          
Microsoft.Network/applicationGateways/sku.tier                                                                          
Microsoft.Network/applicationGateways/webApplicationFirewallConfiguration.disabledRuleGroups[*].ruleGroupName           
Microsoft.Network/applicationGateways/webApplicationFirewallConfiguration.enabled                                       
Microsoft.Network/applicationGateways/webApplicationFirewallConfiguration.firewallMode                                  
Microsoft.Network/applicationGateways/webApplicationFirewallConfiguration.requestBodyCheck

### Azure Automation
Microsoft.Automation/automationAccounts/variables/isEncrypted                                                           
Microsoft.Automation/automationAccounts/webhooks/creationTime                                                           
Microsoft.Automation/automationAccounts/webhooks/expiryTime

### Azure Locks
Microsoft.Authorization/locks/level

### Batch
Microsoft.Batch/batchAccounts/autoStorage.storageAccountId                                                              
Microsoft.Batch/batchAccounts/keyVaultReference.id                                                                      
Microsoft.Batch/batchAccounts/poolAllocationMode

### CDN
Microsoft.CDN/profiles/sku.name                                                                                         
microsoft.cdn/profiles/endpoints/isHttpAllowed                                                                          
microsoft.cdn/profiles/endpoints/isHttpsAllowed

### Cosmos DB
Microsoft.DocumentDB/databaseAccounts/consistencyPolicy.defaultConsistencyLevel                                         
Microsoft.DocumentDB/databaseAccounts/enableAutomaticFailover                                                           
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
Microsoft.DocumentDB/databaseAccounts/virtualNetworkRules[*]                                                            
Microsoft.DocumentDB/databaseAccounts/virtualNetworkRules[*].id                                                         
Microsoft.DocumentDB/databaseAccounts/writeLocations                                                                    
Microsoft.DocumentDB/databaseAccounts/writeLocations[*]                                                                 
Microsoft.DocumentDB/databaseAccounts/writeLocations[*].locationName

### Data Factory
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

### Data Lake Store
Microsoft.DataLakeStore/accounts/encryptionState                                                                        
Microsoft.DataLakeStore/accounts/firewallState                                                                          
Microsoft.DataLakeStore/accounts/firewallRules/endIpAddress                                                             
Microsoft.DataLakeStore/accounts/firewallRules/startIpAddress


### Disks
Microsoft.Compute/disks/sku.name                                                                                        
Microsoft.Compute/imageId                                                                                               
Microsoft.Compute/imageOffer                                                                                            
Microsoft.Compute/imagePublisher                                                                                        
Microsoft.Compute/imageSku                                                                                              
Microsoft.Compute/imageVersion

### Express Route
Microsoft.Network/expressRouteCircuits/serviceProvider.bandwidthInMbps                                                  
Microsoft.Network/expressRouteCircuits/serviceProvider.peeringLocation                                                  
Microsoft.Network/expressRouteCircuits/sku.family                                                                       
Microsoft.Network/expressRouteCircuits/sku.name                                                                         
Microsoft.Network/expressRouteCircuits/sku.tier

### Guest Configuration Assignments
Microsoft.GuestConfiguration/guestConfigurationAssignments/complianceStatus

### HDInsight
Microsoft.HDInsight/clusters/clusterDefinition.kind                                                                     
Microsoft.HDInsight/clusters/clusterVersion                                                                             
Microsoft.HDInsight/clusters/computeProfile.roles[*].virtualNetworkProfile.id                                           
Microsoft.HDInsight/clusters/computeProfile.roles[*].virtualNetworkProfile.subnet                                       
Microsoft.HDInsight/clusters/osType                                                                                     
Microsoft.HDInsight/clusters/securityProfile.directoryType                                                              
Microsoft.HDInsight/clusters/securityProfile.domain                                                                     
Microsoft.HDInsight/clusters/securityProfile.ldapsUrls[*]                                                               
Microsoft.HDInsight/clusters/tier

### Insights
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

### IoT Hubs
Microsoft.Devices/IotHubs/ipFilterRules[*].action                                                                       
Microsoft.Devices/IotHubs/ipFilterRules[*].filterName                                                                   
Microsoft.Devices/IotHubs/ipFilterRules[*].ipMask                                                                       
Microsoft.Devices/IotHubs/sku.capacity                                                                                  
Microsoft.Devices/IotHubs/sku.name                                                                                      
Microsoft.Devices/IotHubs/sku.tier

### Key Vault
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

### Load Balancer
Microsoft.Network/loadBalancers/frontendIPConfigurations[*].privateIPAllocationMethod                                   
Microsoft.Network/loadBalancers/frontendIPConfigurations[*].publicIPAddress.id                                          
Microsoft.Network/loadBalancers/frontendIPConfigurations[*].subnet.id                                                   
Microsoft.Network/loadBalancers/sku.name

### Logic Apps
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

### Network Interface
Microsoft.Network/networkInterfaces/enableIPForwarding                                                                  
Microsoft.Network/networkInterfaces/ipconfigurations[*].applicationSecurityGroups                                       
Microsoft.Network/networkInterfaces/ipconfigurations[*].applicationSecurityGroups[*].id                                 
Microsoft.Network/networkInterfaces/ipconfigurations[*].privateIPAllocationMethod                                       
Microsoft.Network/networkInterfaces/ipconfigurations[*].publicIpAddress.id                                              
Microsoft.Network/networkInterfaces/ipconfigurations[*].subnet.id                                                       
Microsoft.Network/networkInterfaces/networkSecurityGroup.id

### Network Security Group
Microsoft.Network/networkSecurityGroups/networkInterfaces[*].id                                                         
Microsoft.Network/networkSecurityGroups/securityRules[*].access                                                         
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

### Notification Hubs
Microsoft.NotificationHubs/Namespaces/AuthorizationRules/rights                                                         
Microsoft.NotificationHubs/Namespaces/AuthorizationRules/rights[*]                                                      
Microsoft.NotificationHubs/namespaces/notificationHubs/authorizationRules/rights                                        
Microsoft.NotificationHubs/namespaces/notificationHubs/authorizationRules/rights[*]

### Operational Insights
Microsoft.OperationalInsights/workspaces/retentionInDays                                                                
Microsoft.OperationalInsights/workspaces/sku.name                                                                       
Microsoft.OperationalInsights/workspaces/dataSources/counterName                                                        
Microsoft.OperationalInsights/workspaces/dataSources/eventLogName                                                       
Microsoft.OperationalInsights/workspaces/dataSources/eventTypes[*].eventType                                            
Microsoft.OperationalInsights/workspaces/dataSources/instanceName                                                       
Microsoft.OperationalInsights/workspaces/dataSources/intervalSeconds                                                    
Microsoft.OperationalInsights/workspaces/dataSources/objectName

### Public IP
Microsoft.Network/publicIPAddresses/sku.name

### Redis Cache
Microsoft.Cache/Redis/enableNonSslPort                                                                                  
Microsoft.Cache/Redis/shardCount                                                                                        
Microsoft.Cache/Redis/sku.capacity                                                                                      
Microsoft.Cache/Redis/sku.family                                                                                        
Microsoft.Cache/Redis/sku.name
Microsoft.Cache/Redis/firewallrules/endIP                                                                               
Microsoft.Cache/Redis/firewallrules/startIP    

### Role Assignments
Microsoft.Authorization/roleAssignments/principalId                                                                     
Microsoft.Authorization/roleAssignments/principalType                                                                   
Microsoft.Authorization/roleAssignments/roleDefinitionId

### Role Definitions
Microsoft.Authorization/roleDefinitions/permissions.actions[*]                                                          
Microsoft.Authorization/roleDefinitions/permissions.notActions[*]                                                       
Microsoft.Authorization/roleDefinitions/roleName                                                                        
Microsoft.Authorization/roleDefinitions/type       

### Scheduler
Microsoft.Scheduler/jobcollections/sku.name

### Search
Microsoft.Search/searchServices/hostingMode                                                                             
Microsoft.Search/searchServices/partitionCount                                                                          
Microsoft.Search/searchServices/replicaCount                                                                            
Microsoft.Search/searchServices/sku.name

### Security
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

### Service Fabric
Microsoft.ServiceFabric/clusters/azureActiveDirectory.tenantId                                                          
Microsoft.ServiceFabric/clusters/certificate.thumbprint                                                                 
Microsoft.ServiceFabric/clusters/certificate.x509StoreName                                                              
Microsoft.ServiceFabric/clusters/fabricSettings[*].name                                                                 
Microsoft.ServiceFabric/clusters/fabricSettings[*].parameters[*].name                                                   
Microsoft.ServiceFabric/clusters/fabricSettings[*].parameters[*].value

### SQL
Microsoft.Sql/servers/version                                                                                           
Microsoft.Sql/servers/administrators/administratorType                                                                  
Microsoft.Sql/servers/administrators/login                                                                              
Microsoft.Sql/servers/administrators/sid                                                                                
Microsoft.Sql/servers/administrators/tenantId                                                                           
Microsoft.Sql/auditingSettings.state                                                                                    
Microsoft.Sql/servers/auditingSettings/retentionDays                                                                    
Microsoft.Sql/servers/auditingSettings/state                                                                            
Microsoft.Sql/servers/auditingSettings/storageEndpoint                                                                  
Microsoft.Sql/servers/automaticTuning/desiredState                                                                      
Microsoft.Sql/servers/automaticTuning/options.createIndex                                                               
Microsoft.Sql/servers/automaticTuning/options.dropIndex                                                                 
Microsoft.Sql/servers/automaticTuning/options.forceLastGoodPlan                                                         
Microsoft.Sql/servers/connectionPolicies/connectionType                                                                 
Microsoft.Sql/servers/databases/edition                                                                                 
Microsoft.Sql/servers/databases/elasticPoolName                                                                         
Microsoft.Sql/servers/databases/requestedServiceObjectiveId                                                             
Microsoft.Sql/servers/databases/requestedServiceObjectiveName                                                           
Microsoft.Sql/auditingSettings.state                                                                                    
Microsoft.Sql/servers/databases/auditingSettings/retentionDays                                                          
Microsoft.Sql/servers/databases/auditingSettings/state                                                                  
Microsoft.Sql/servers/databases/auditingSettings/storageEndpoint                                                        
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

### Storage Account
Microsoft.Storage/storageAccounts/accessTier                                                                            
Microsoft.Storage/storageAccounts/accountType                                                                           
Microsoft.Storage/storageAccounts/enableBlobEncryption                                                                  
Microsoft.Storage/storageAccounts/enableFileEncryption                                                                  
Microsoft.Storage/storageAccounts/networkAcls.defaultAction                                                             
Microsoft.Storage/storageAccounts/networkAcls.ipRules                                                                   
Microsoft.Storage/storageAccounts/networkAcls.ipRules[*]                                                                
Microsoft.Storage/storageAccounts/networkAcls.ipRules[*].value                                                          
Microsoft.Storage/storageAccounts/networkAcls.virtualNetworkRules                                                       
Microsoft.Storage/storageAccounts/networkAcls.virtualNetworkRules[*]                                                    
Microsoft.Storage/storageAccounts/networkAcls.virtualNetworkRules[*].id                                                 
Microsoft.Storage/storageAccounts/sku.name                                                                              
Microsoft.Storage/storageAccounts/supportsHttpsTrafficOnly

### Terraform OSS
Microsoft.TerraformOSS/resources/image

### Traffic Manager
Microsoft.Network/trafficmanagerprofiles/monitorConfig.protocol

### Virtual Machine
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
Microsoft.Compute/virtualMachines/osProfile.linuxConfiguration                                                          
Microsoft.Compute/virtualMachines/osProfile.linuxConfiguration.disablePasswordAuthentication                            
Microsoft.Compute/virtualMachines/osProfile.windowsConfiguration                                                        
Microsoft.Compute/virtualMachines/osProfile.windowsConfiguration.enableAutomaticUpdates                                 
Microsoft.Compute/virtualMachines/osProfile.windowsConfiguration.provisionVMAgent                                       
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

### Virtual Machine Extension
Microsoft.Compute/virtualMachines/extensions/autoUpgradeMinorVersion                                                    
Microsoft.Compute/virtualMachines/extensions/provisioningState                                                          
Microsoft.Compute/virtualMachines/extensions/publisher                                                                  
Microsoft.Compute/virtualMachines/extensions/settings                                                                   
Microsoft.Compute/virtualMachines/extensions/settings.workspaceId                                                       
Microsoft.Compute/virtualMachines/extensions/type                                                                       
Microsoft.Compute/virtualMachines/extensions/typeHandlerVersion

### Virtual Machine Scale Sets
Microsoft.Compute/imageId                                                                                               
Microsoft.Compute/imageOffer                                                                                            
Microsoft.Compute/imagePublisher                                                                                        
Microsoft.Compute/imageSku                                                                                              
Microsoft.Compute/imageVersion                                                                                          
Microsoft.Compute/licenseType                                                                                           
Microsoft.Compute/VirtualMachineScaleSets/computerNamePrefix                                                            
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

### Virtual Machine Scale Sets Extensions
Microsoft.Compute/virtualMachineScaleSets/extensions/autoUpgradeMinorVersion                                            
Microsoft.Compute/virtualMachineScaleSets/extensions/provisioningState                                                  
Microsoft.Compute/virtualMachineScaleSets/extensions/publisher                                                          
Microsoft.Compute/virtualMachineScaleSets/extensions/settings                                                           
Microsoft.Compute/virtualMachineScaleSets/extensions/settings.workspaceId                                               
Microsoft.Compute/virtualMachineScaleSets/extensions/type                                                               
Microsoft.Compute/virtualMachineScaleSets/extensions/typeHandlerVersion

### Virtual Network
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

### Virtual Network Gateway
Microsoft.Network/virtualNetworkGateways/gatewayType                                                                    
Microsoft.Network/virtualNetworkGateways/sku.name

Tags: #Azure #AzurePolicy #Policy #ResourceGroup #Resource #AzureSecurity #AzureCompliance 