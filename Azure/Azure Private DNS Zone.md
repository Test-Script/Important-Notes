========================================== **Virtual Network Peering** ============================================ 

Remote Virtual network summary

Peering link Name : Spoke_vnet

Subscription : Test

Vitual Network : Spoke

Remote Virtual network summary

Peering link name : Hub_vnet

============================================== **Private DNS Zone** ================================================
Basics:

Instance details

Name : testdns.com

Resource group location : East US.

Private DNS Zone Editor : Using a zone file is a quick, reliable and convenient way to transfer a DNS zone into or out of Azure Private DNS. It's important to note that files should not exceed 10,000 lines, and the DNS zone import feature currently supports a maximum of 799 record sets.

Virtual Network Link : Once the zone is linked to the following Virtual Networks, resources hosted in them can query this private DNS zone.

Add Virtual Network Link :

link name : Hub_dns_link

    Subscription : Test
    Virtual Network : Hub (RG-VirtualNetwork)

    Configuration :
    
    Enable auto registeration : Yes (Checked)

    Enable fallback to internet : Greyed-Out

Azure DNS Zone Template:

================================================================================
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "privateDnsZone": {
            "type": "String"
        }
    },
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Network/privateDnsZones",
            "apiVersion": "2024-06-01",
            "name": "[parameters('privateDnsZone')]",
            "location": "global",
            "dependsOn": [],
            "properties": {},
            "resources": []
        },
        {
            "type": "Microsoft.Network/privateDnsZones/virtualNetworkLinks",
            "apiVersion": "2024-06-01",
            "name": "[concat(parameters('privateDnsZone'), '/', 'Hub_dns_link')]",
            "location": "global",
            "dependsOn": [
                "[resourceId('Microsoft.Network/privateDnsZones', parameters('privateDnsZone'))]"
            ],
            "properties": {
                "virtualNetwork": {
                    "id": "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/RG-VirtualNetwork/providers/Microsoft.Network/virtualNetworks/Hub"
                },
                "registrationEnabled": true
            }
        },
        {
            "type": "Microsoft.Network/privateDnsZones/virtualNetworkLinks",
            "apiVersion": "2024-06-01",
            "name": "[concat(parameters('privateDnsZone'), '/', 'Spoke')]",
            "location": "global",
            "dependsOn": [
                "[resourceId('Microsoft.Network/privateDnsZones', parameters('privateDnsZone'))]"
            ],
            "properties": {
                "virtualNetwork": {
                    "id": "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/RG-VirtualNetwork/providers/Microsoft.Network/virtualNetworks/Spoke"
                },
                "registrationEnabled": true
            }
        }
    ],
    "outputs": {}
}


======================================= **Azure Virtual Machine Deployment** =======================================

{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "location": {
            "type": "String"
        },
        "networkInterfaceName": {
            "type": "String"
        },
        "networkSecurityGroupName": {
            "type": "String"
        },
        "networkSecurityGroupRules": {
            "type": "Array"
        },
        "subnetName": {
            "type": "String"
        },
        "virtualNetworkId": {
            "type": "String"
        },
        "publicIpAddressName": {
            "type": "String"
        },
        "publicIpAddressType": {
            "type": "String"
        },
        "publicIpAddressSku": {
            "type": "String"
        },
        "pipDeleteOption": {
            "type": "String"
        },
        "virtualMachineName": {
            "type": "String"
        },
        "virtualMachineComputerName": {
            "type": "String"
        },
        "virtualMachineRG": {
            "type": "String"
        },
        "osDiskType": {
            "type": "String"
        },
        "osDiskDeleteOption": {
            "type": "String"
        },
        "virtualMachineSize": {
            "type": "String"
        },
        "nicDeleteOption": {
            "type": "String"
        },
        "hibernationEnabled": {
            "type": "Bool"
        },
        "adminUsername": {
            "type": "String"
        },
        "adminPassword": {
            "type": "SecureString"
        },
        "enablePeriodicAssessment": {
            "type": "String"
        }
    },
    "variables": {
        "nsgId": "[resourceId(resourceGroup().name, 'Microsoft.Network/networkSecurityGroups', parameters('networkSecurityGroupName'))]",
        "vnetId": "[parameters('virtualNetworkId')]",
        "vnetName": "[last(split(variables('vnetId'), '/'))]",
        "subnetRef": "[concat(variables('vnetId'), '/subnets/', parameters('subnetName'))]"
    },
    "resources": [
        {
            "type": "Microsoft.Network/networkInterfaces",
            "apiVersion": "2022-11-01",
            "name": "[parameters('networkInterfaceName')]",
            "location": "[parameters('location')]",
            "dependsOn": [
                "[concat('Microsoft.Network/networkSecurityGroups/', parameters('networkSecurityGroupName'))]",
                "[concat('Microsoft.Network/publicIpAddresses/', parameters('publicIpAddressName'))]"
            ],
            "properties": {
                "ipConfigurations": [
                    {
                        "name": "ipconfig1",
                        "properties": {
                            "subnet": {
                                "id": "[variables('subnetRef')]"
                            },
                            "privateIPAllocationMethod": "Dynamic",
                            "publicIpAddress": {
                                "id": "[resourceId(resourceGroup().name, 'Microsoft.Network/publicIpAddresses', parameters('publicIpAddressName'))]",
                                "properties": {
                                    "deleteOption": "[parameters('pipDeleteOption')]"
                                }
                            }
                        }
                    }
                ],
                "networkSecurityGroup": {
                    "id": "[variables('nsgId')]"
                }
            }
        },
        {
            "type": "Microsoft.Network/networkSecurityGroups",
            "apiVersion": "2020-05-01",
            "name": "[parameters('networkSecurityGroupName')]",
            "location": "[parameters('location')]",
            "properties": {
                "securityRules": "[parameters('networkSecurityGroupRules')]"
            }
        },
        {
            "type": "Microsoft.Network/publicIpAddresses",
            "apiVersion": "2023-06-01",
            "name": "[parameters('publicIpAddressName')]",
            "location": "[parameters('location')]",
            "sku": {
                "name": "[parameters('publicIpAddressSku')]"
            },
            "properties": {
                "publicIpAllocationMethod": "[parameters('publicIpAddressType')]"
            }
        },
        {
            "type": "Microsoft.Compute/virtualMachines",
            "apiVersion": "2024-03-01",
            "name": "[parameters('virtualMachineName')]",
            "location": "[parameters('location')]",
            "dependsOn": [
                "[concat('Microsoft.Network/networkInterfaces/', parameters('networkInterfaceName'))]"
            ],
            "properties": {
                "hardwareProfile": {
                    "vmSize": "[parameters('virtualMachineSize')]"
                },
                "storageProfile": {
                    "osDisk": {
                        "createOption": "fromImage",
                        "managedDisk": {
                            "storageAccountType": "[parameters('osDiskType')]"
                        },
                        "deleteOption": "[parameters('osDiskDeleteOption')]"
                    },
                    "imageReference": {
                        "publisher": "canonical",
                        "offer": "ubuntu-24_04-lts",
                        "sku": "server",
                        "version": "latest"
                    }
                },
                "networkProfile": {
                    "networkInterfaces": [
                        {
                            "id": "[resourceId('Microsoft.Network/networkInterfaces', parameters('networkInterfaceName'))]",
                            "properties": {
                                "deleteOption": "[parameters('nicDeleteOption')]"
                            }
                        }
                    ]
                },
                "securityProfile": {},
                "additionalCapabilities": {
                    "hibernationEnabled": false
                },
                "osProfile": {
                    "computerName": "[parameters('virtualMachineComputerName')]",
                    "adminUsername": "[parameters('adminUsername')]",
                    "adminPassword": "[parameters('adminPassword')]",
                    "linuxConfiguration": {
                        "patchSettings": {
                            "assessmentMode": "[parameters('enablePeriodicAssessment')]",
                            "patchMode": "ImageDefault"
                        }
                    }
                }
            }
        }
    ],
    "outputs": {
        "adminUsername": {
            "type": "String",
            "value": "[parameters('adminUsername')]"
        }
    }
}

================================================== **Azure Storage Account** =======================================

Always use customer managed key for the Storage Account data encryption.

https://learn.microsoft.com/en-us/azure/storage/common/infrastructure-encryption-enable?tabs=portal

{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "location": {
            "type": "String"
        },
        "storageAccountName": {
            "type": "String"
        },
        "accountType": {
            "type": "String"
        },
        "kind": {
            "type": "String"
        },
        "minimumTlsVersion": {
            "type": "String"
        },
        "supportsHttpsTrafficOnly": {
            "type": "Bool"
        },
        "allowBlobPublicAccess": {
            "type": "Bool"
        },
        "allowSharedKeyAccess": {
            "type": "Bool"
        },
        "defaultOAuth": {
            "type": "Bool"
        },
        "accessTier": {
            "type": "String"
        },
        "publicNetworkAccess": {
            "type": "String"
        },
        "allowCrossTenantReplication": {
            "type": "Bool"
        },
        "networkAclsBypass": {
            "type": "String"
        },
        "networkAclsDefaultAction": {
            "type": "String"
        },
        "networkAclsIpRules": {
            "type": "Array"
        },
        "networkAclsIpv6Rules": {
            "type": "Array"
        },
        "publishIpv6Endpoint": {
            "type": "Bool"
        },
        "dnsEndpointType": {
            "type": "String"
        },
        "largeFileSharesState": {
            "type": "String"
        },
        "keySource": {
            "type": "String"
        },
        "encryptionEnabled": {
            "type": "Bool"
        },
        "keyTypeForTableAndQueueEncryption": {
            "type": "String"
        },
        "infrastructureEncryptionEnabled": {
            "type": "Bool"
        },
        "isBlobSoftDeleteEnabled": {
            "type": "Bool"
        },
        "blobSoftDeleteRetentionDays": {
            "type": "Int"
        },
        "isContainerSoftDeleteEnabled": {
            "type": "Bool"
        },
        "containerSoftDeleteRetentionDays": {
            "type": "Int"
        },
        "isShareSoftDeleteEnabled": {
            "type": "Bool"
        },
        "shareSoftDeleteRetentionDays": {
            "type": "Int"
        }
    },
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Storage/storageAccounts",
            "apiVersion": "2025-06-01",
            "name": "[parameters('storageAccountName')]",
            "location": "[parameters('location')]",
            "dependsOn": [],
            "tags": {},
            "sku": {
                "name": "[parameters('accountType')]"
            },
            "kind": "[parameters('kind')]",
            "properties": {
                "minimumTlsVersion": "[parameters('minimumTlsVersion')]",
                "supportsHttpsTrafficOnly": "[parameters('supportsHttpsTrafficOnly')]",
                "allowBlobPublicAccess": "[parameters('allowBlobPublicAccess')]",
                "allowSharedKeyAccess": "[parameters('allowSharedKeyAccess')]",
                "defaultToOAuthAuthentication": "[parameters('defaultOAuth')]",
                "accessTier": "[parameters('accessTier')]",
                "publicNetworkAccess": "[parameters('publicNetworkAccess')]",
                "allowCrossTenantReplication": "[parameters('allowCrossTenantReplication')]",
                "networkAcls": {
                    "bypass": "[parameters('networkAclsBypass')]",
                    "defaultAction": "[parameters('networkAclsDefaultAction')]",
                    "ipRules": "[parameters('networkAclsIpRules')]",
                    "ipv6Rules": "[parameters('networkAclsIpv6Rules')]"
                },
                "dualStackEndpointPreference": {
                    "publishIpv6Endpoint": "[parameters('publishIpv6Endpoint')]"
                },
                "dnsEndpointType": "[parameters('dnsEndpointType')]",
                "largeFileSharesState": "[parameters('largeFileSharesState')]",
                "encryption": {
                    "keySource": "[parameters('keySource')]",
                    "services": {
                        "blob": {
                            "enabled": "[parameters('encryptionEnabled')]"
                        },
                        "file": {
                            "enabled": "[parameters('encryptionEnabled')]"
                        },
                        "table": {
                            "enabled": "[parameters('encryptionEnabled')]"
                        },
                        "queue": {
                            "enabled": "[parameters('encryptionEnabled')]"
                        }
                    },
                    "requireInfrastructureEncryption": "[parameters('infrastructureEncryptionEnabled')]"
                }
            }
        },
        {
            "type": "Microsoft.Storage/storageAccounts/blobServices",
            "apiVersion": "2025-06-01",
            "name": "[concat(parameters('storageAccountName'), '/default')]",
            "dependsOn": [
                "[concat('Microsoft.Storage/storageAccounts/', parameters('storageAccountName'))]"
            ],
            "properties": {
                "deleteRetentionPolicy": {
                    "enabled": "[parameters('isBlobSoftDeleteEnabled')]",
                    "days": "[parameters('blobSoftDeleteRetentionDays')]"
                },
                "containerDeleteRetentionPolicy": {
                    "enabled": "[parameters('isContainerSoftDeleteEnabled')]",
                    "days": "[parameters('containerSoftDeleteRetentionDays')]"
                }
            }
        },
        {
            "type": "Microsoft.Storage/storageAccounts/fileservices",
            "apiVersion": "2025-06-01",
            "name": "[concat(parameters('storageAccountName'), '/default')]",
            "dependsOn": [
                "[concat('Microsoft.Storage/storageAccounts/', parameters('storageAccountName'))]",
                "[concat(concat('Microsoft.Storage/storageAccounts/', parameters('storageAccountName')), '/blobServices/default')]"
            ],
            "properties": {
                "protocolSettings": null,
                "shareDeleteRetentionPolicy": {
                    "enabled": "[parameters('isShareSoftDeleteEnabled')]",
                    "days": "[parameters('shareSoftDeleteRetentionDays')]"
                }
            }
        }
    ],
    "outputs": {}
}

{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "location": {
            "value": "eastus"
        },
        "storageAccountName": {
            "value": "testdingns"
        },
        "accountType": {
            "value": "Standard_LRS"
        },
        "kind": {
            "value": "StorageV2"
        },
        "minimumTlsVersion": {
            "value": "TLS1_2"
        },
        "supportsHttpsTrafficOnly": {
            "value": true
        },
        "allowBlobPublicAccess": {
            "value": false
        },
        "allowSharedKeyAccess": {
            "value": true
        },
        "defaultOAuth": {
            "value": false
        },
        "accessTier": {
            "value": "Hot"
        },
        "publicNetworkAccess": {
            "value": "Disabled"
        },
        "allowCrossTenantReplication": {
            "value": false
        },
        "networkAclsBypass": {
            "value": "AzureServices"
        },
        "networkAclsDefaultAction": {
            "value": "Deny"
        },
        "networkAclsIpRules": {
            "value": []
        },
        "networkAclsIpv6Rules": {
            "value": []
        },
        "publishIpv6Endpoint": {
            "value": false
        },
        "dnsEndpointType": {
            "value": "Standard"
        },
        "largeFileSharesState": {
            "value": "Enabled"
        },
        "keySource": {
            "value": "Microsoft.Storage"
        },
        "encryptionEnabled": {
            "value": true
        },
        "keyTypeForTableAndQueueEncryption": {
            "value": "Account"
        },
        "infrastructureEncryptionEnabled": {
            "value": false
        },
        "isBlobSoftDeleteEnabled": {
            "value": true
        },
        "blobSoftDeleteRetentionDays": {
            "value": 7
        },
        "isContainerSoftDeleteEnabled": {
            "value": true
        },
        "containerSoftDeleteRetentionDays": {
            "value": 7
        },
        "isShareSoftDeleteEnabled": {
            "value": true
        },
        "shareSoftDeleteRetentionDays": {
            "value": 7
        }
    }
}
================================================ **Azure Private Endpoint** ========================================

Basic:

    Instance details
        Name: Storage_PE
        Network Interface Name: Storage_PE-nic
        Region: East US
    
Resource:

    Connection method:
        Connect to an Azure resource in my directory. ✅

            Subscription: Test
            Resource type: Microsoft.Storage/storageAccount
            Resource: testingdns
            Target Sub-Target: file

        Connect to an Azure resource by resource ID or alias.

Virtual Network:

    Virtual network:
    Subnet:
    Network policy for private endpoints:
    Private IP configuration:
    	Dynamically allocate IP address
        Statically allocate IP address

**Manully entry of private endpoint has to be created in the private dns zone**

Name : testdns.com
Number of records sets: 1
Max number of record set: 25000
Number of Virtual network links : 2/1000
Number of Virtual network links with auto registration enabled: 2/100
Resource group: RG-DNS

{
  "code": "Conflict",
  "message": "A virtual network can only be linked to 1 Private DNS zone(s) with auto-registration enabled; conflicting Private DNS zone is '/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourcegroups/rg-dns/providers/microsoft.network/privatednszones/testdns.com'."
}

=========================================== **Azure Files Json view** ==============================================

{
    "apiVersion": "2025-01-01",
    "id": "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/RG-Storage/providers/Microsoft.Storage/storageAccounts/testdingns",
    "name": "testdingns",
    "type": "microsoft.storage/storageaccounts",
    "sku": {
        "name": "Standard_LRS",
        "tier": "Standard"
    },
    "kind": "StorageV2",
    "location": "eastus",
    "tags": {},
    "properties": {
        "dualStackEndpointPreference": {
            "defaultDualStackEndpoints": false,
            "publishIpv4Endpoint": false,
            "publishIpv6Endpoint": false
        },
        "dnsEndpointType": "Standard",
        "defaultToOAuthAuthentication": false,
        "publicNetworkAccess": "Disabled",
        "keyCreationTime": {
            "key1": "2025-12-15T18:57:05.958Z",
            "key2": "2025-12-15T18:57:05.958Z"
        },
        "allowCrossTenantReplication": false,
        "privateEndpointConnections": [
            {
                "id": "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/RG-Storage/providers/Microsoft.Storage/storageAccounts/testdingns/privateEndpointConnections/testdingns.ee56e61a-bb69-441b-a6e4-519992a3477f",
                "name": "testdingns.ee56e61a-bb69-441b-a6e4-519992a3477f",
                "type": "Microsoft.Storage/storageAccounts/privateEndpointConnections",
                "properties": {
                    "provisioningState": "Succeeded",
                    "privateEndpoint": {
                        "id": "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/RG-PrivateEndpoint/providers/Microsoft.Network/privateEndpoints/Storage_PE"
                    },
                    "privateLinkServiceConnectionState": {
                        "status": "Approved",
                        "description": "Auto-Approved",
                        "actionRequired": "None"
                    }
                }
            }
        ],
        "minimumTlsVersion": "TLS1_2",
        "allowBlobPublicAccess": false,
        "allowSharedKeyAccess": true,
        "largeFileSharesState": "Enabled",
        "networkAcls": {
            "ipv6Rules": [],
            "bypass": "AzureServices",
            "virtualNetworkRules": [],
            "ipRules": [],
            "defaultAction": "Deny"
        },
        "supportsHttpsTrafficOnly": true,
        "encryption": {
            "requireInfrastructureEncryption": false,
            "services": {
                "file": {
                    "keyType": "Account",
                    "enabled": true,
                    "lastEnabledTime": "2025-12-15T18:57:05.974Z"
                },
                "blob": {
                    "keyType": "Account",
                    "enabled": true,
                    "lastEnabledTime": "2025-12-15T18:57:05.974Z"
                }
            },
            "keySource": "Microsoft.Storage"
        },
        "accessTier": "Hot",
        "provisioningState": "Succeeded",
        "creationTime": "2025-12-15T18:57:05.661Z",
        "primaryEndpoints": {
            "dfs": "https://testdingns.dfs.core.windows.net/",
            "web": "https://testdingns.z13.web.core.windows.net/",
            "blob": "https://testdingns.blob.core.windows.net/",
            "queue": "https://testdingns.queue.core.windows.net/",
            "table": "https://testdingns.table.core.windows.net/",
            "file": "https://testdingns.file.core.windows.net/"
        },
        "primaryLocation": "eastus",
        "statusOfPrimary": "available"
    }
}

========================================== **Azure Private DNS libraries** =========================================

https://learn.microsoft.com/en-us/python/api/overview/azure/private-dns?view=azure-python

========================================== **Azure Private DNS Resolver** ==========================================

Azure DNS private resolver bridges on-premises DNS namespaces with private DNS zones hosted on Azure DNS without the burden of deploying VM-based custom DNS servers. You can resolve DNS queries from on-premises networks and do conditional forwarding to on-premises DNS zones.
