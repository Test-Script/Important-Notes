tejas [ ~ ]$ az group create \
  --name $RESOURCE_GROUP \
  --location $LOCATION
{
  "id": "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-identity-test",
  "location": "centralindia",
  "managedBy": null,
  "name": "rg-identity-test",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null,
  "type": "Microsoft.Resources/resourceGroups"
}


==================

tejas [ ~ ]$ az keyvault create \
  --name $KEYVAULT_NAME \
  --resource-group $RESOURCE_GROUP \
  --location $LOCATION \
  --enable-rbac-authorization true
{
  "id": "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-security-test/providers/Microsoft.KeyVault/vaults/kv-tt-app-test-001",
  "location": "centralindia",
  "name": "kv-tt-app-test-001",
  "properties": {
    "accessPolicies": [],
    "createMode": null,
    "enablePurgeProtection": null,
    "enableRbacAuthorization": true,
    "enableSoftDelete": true,
    "enabledForDeployment": false,
    "enabledForDiskEncryption": false,
    "enabledForTemplateDeployment": false,
    "hsmPoolResourceId": null,
    "networkAcls": null,
    "privateEndpointConnections": null,
    "provisioningState": "Succeeded",
    "publicNetworkAccess": "Enabled",
    "sku": {
      "family": "A",
      "name": "standard"
    },
    "softDeleteRetentionInDays": 90,
    "tenantId": "7faa8f70-7da4-4fa9-a24d-9bbd6682a962",
    "vaultUri": "https://kv-tt-app-test-001.vault.azure.net/"
  },
  "resourceGroup": "rg-security-test",
  "systemData": {
    "createdAt": "2026-01-24T16:38:08.827000+00:00",
    "createdBy": "tejastarghale47@gmail.com",
    "createdByType": "User",
    "lastModifiedAt": "2026-01-24T17:42:10.972000+00:00",
    "lastModifiedBy": "tejastarghale47@gmail.com",
    "lastModifiedByType": "User"
  },
  "tags": {},
  "type": "Microsoft.KeyVault/vaults"
}

==============

tejas [ ~ ]$ az group create \
  --name $RESOURCE_GROUP \
  --location $LOCATION
{
  "id": "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-network-security-test",
  "location": "centralindia",
  "managedBy": null,
  "name": "rg-network-security-test",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null,
  "type": "Microsoft.Resources/resourceGroups"
}

=============

tejas [ ~ ]$ az network vnet create \
  --resource-group $RESOURCE_GROUP \
  --name $VNET_NAME \
  --address-prefix $VNET_CIDR \
  --subnet-name $SUBNET1_NAME \
  --subnet-prefix $SUBNET1_CIDR \
  --location $LOCATION
{
  "newVNet": {
    "addressSpace": {
      "addressPrefixes": [
        "10.10.0.0/16"
      ]
    },
    "enableDdosProtection": false,
    "etag": "W/\"ace9e14c-8bce-43c0-9bd4-183439f4d8c7\"",
    "id": "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-network-security-test/providers/Microsoft.Network/virtualNetworks/vnet-app-test",
    "location": "centralindia",
    "name": "vnet-app-test",
    "privateEndpointVNetPolicies": "Disabled",
    "provisioningState": "Succeeded",
    "resourceGroup": "rg-network-security-test",
    "resourceGuid": "31db91fa-5a84-4967-8b86-2a72f334cdc0",
    "subnets": [
      {
        "addressPrefix": "10.10.1.0/24",
        "delegations": [],
        "etag": "W/\"ace9e14c-8bce-43c0-9bd4-183439f4d8c7\"",
        "id": "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-network-security-test/providers/Microsoft.Network/virtualNetworks/vnet-app-test/subnets/subnet-vm",
        "name": "subnet-vm",
        "privateEndpointNetworkPolicies": "Disabled",
        "privateLinkServiceNetworkPolicies": "Enabled",
        "provisioningState": "Succeeded",
        "resourceGroup": "rg-network-security-test",
        "type": "Microsoft.Network/virtualNetworks/subnets"
      }
    ],
    "type": "Microsoft.Network/virtualNetworks",
    "virtualNetworkPeerings": []
  }
}

===========

tejas [ ~ ]$ az network vnet subnet create \
  --resource-group $RESOURCE_GROUP \
  --vnet-name $VNET_NAME \
  --name $SUBNET2_NAME \
  --address-prefix $SUBNET2_CIDR
{
  "addressPrefix": "10.10.2.0/24",
  "delegations": [],
  "etag": "W/\"042bad31-1024-43ec-acf4-d0724b12e4f2\"",
  "id": "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-network-security-test/providers/Microsoft.Network/virtualNetworks/vnet-app-test/subnets/subnet-service",
  "name": "subnet-service",
  "privateEndpointNetworkPolicies": "Disabled",
  "privateLinkServiceNetworkPolicies": "Enabled",
  "provisioningState": "Succeeded",
  "resourceGroup": "rg-network-security-test",
  "type": "Microsoft.Network/virtualNetworks/subnets"
}

===========

tejas [ ~ ]$ az network private-endpoint create \
  --name $PRIVATE_ENDPOINT_NAME \
  --resource-group $RESOURCE_GROUP \
  --location $LOCATION \
  --vnet-name $VNET_NAME \
  --subnet $SUBNET_NAME \
  --private-connection-resource-id $KV_RESOURCE_ID \
  --group-id vault \
  --connection-name "kv-private-connection"
{
  "customDnsConfigs": [
    {
      "fqdn": "kv-tt-app-test-001.vault.azure.net",
      "ipAddresses": [
        "10.10.2.4"
      ]
    }
  ],
  "customNetworkInterfaceName": "",
  "etag": "W/\"1ff99b4d-07da-49ed-830e-5eca85ac28a4\"",
  "id": "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-network-security-test/providers/Microsoft.Network/privateEndpoints/pe-kv-app-prod",
  "ipConfigurations": [],
  "location": "centralindia",
  "manualPrivateLinkServiceConnections": [],
  "name": "pe-kv-app-prod",
  "networkInterfaces": [
    {
      "id": "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-network-security-test/providers/Microsoft.Network/networkInterfaces/pe-kv-app-prod.nic.3647f128-96a9-4080-80d5-73aa9399595d",
      "resourceGroup": "rg-network-security-test"
    }
  ],
  "privateLinkServiceConnections": [
    {
      "etag": "W/\"1ff99b4d-07da-49ed-830e-5eca85ac28a4\"",
      "groupIds": [
        "vault"
      ],
      "id": "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-network-security-test/providers/Microsoft.Network/privateEndpoints/pe-kv-app-prod/privateLinkServiceConnections/kv-private-connection",
      "name": "kv-private-connection",
      "privateLinkServiceConnectionState": {
        "actionsRequired": "None",
        "description": "",
        "status": "Approved"
      },
      "privateLinkServiceId": "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-security-test/providers/Microsoft.KeyVault/vaults/kv-tt-app-test-001",
      "provisioningState": "Succeeded",
      "resourceGroup": "rg-network-security-test",
      "type": "Microsoft.Network/privateEndpoints/privateLinkServiceConnections"
    }
  ],
  "provisioningState": "Succeeded",
  "resourceGroup": "rg-network-security-test",
  "subnet": {
    "id": "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-network-security-test/providers/Microsoft.Network/virtualNetworks/vnet-app-test/subnets/subnet-service",
    "resourceGroup": "rg-network-security-test"
  },
  "type": "Microsoft.Network/privateEndpoints"
}

===========

tejas [ ~ ]$ az network private-dns zone create \
  --resource-group $RESOURCE_GROUP \
  --name $PRIVATE_DNS_ZONE
{
  "etag": "caac0901-7fbd-426c-b5ae-c2ebffeb2463",
  "id": "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-network-security-test/providers/Microsoft.Network/privateDnsZones/privatelink.vaultcore.azure.net",
  "location": "global",
  "maxNumberOfRecordSets": 25000,
  "maxNumberOfVirtualNetworkLinks": 1000,
  "maxNumberOfVirtualNetworkLinksWithRegistration": 100,
  "name": "privatelink.vaultcore.azure.net",
  "numberOfRecordSets": 1,
  "numberOfVirtualNetworkLinks": 0,
  "numberOfVirtualNetworkLinksWithRegistration": 0,
  "provisioningState": "Succeeded",
  "resourceGroup": "rg-network-security-test",
  "type": "Microsoft.Network/privateDnsZones"
}

===========

tejas [ ~ ]$ az network private-dns link vnet create \
  --resource-group $RESOURCE_GROUP \
  --zone-name $PRIVATE_DNS_ZONE \
  --name $DNS_LINK_NAME \
  --virtual-network $VNET_NAME \
  --registration-enabled false
{
  "etag": "\"63010dbd-0000-0100-0000-6975077b0000\"",
  "id": "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-network-security-test/providers/Microsoft.Network/privateDnsZones/privatelink.vaultcore.azure.net/virtualNetworkLinks/dnslink-kv-prod",
  "location": "global",
  "name": "dnslink-kv-prod",
  "provisioningState": "Succeeded",
  "registrationEnabled": false,
  "resolutionPolicy": "Default",
  "resourceGroup": "rg-network-security-test",
  "type": "Microsoft.Network/privateDnsZones/virtualNetworkLinks",
  "virtualNetwork": {
    "id": "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-network-security-test/providers/Microsoft.Network/virtualNetworks/vnet-app-test",
    "resourceGroup": "rg-network-security-test"
  },
  "virtualNetworkLinkState": "Completed"
}

==========

tejas [ ~ ]$ az network private-endpoint dns-zone-group create \
  --resource-group $RESOURCE_GROUP \
  --endpoint-name $PRIVATE_ENDPOINT_NAME \
  --name "kv-dns-zone-group" \
  --private-dns-zone $PRIVATE_DNS_ZONE \
  --zone-name "kvZone"
{
  "etag": "W/\"d119a9a7-3952-44de-9bf9-86970287a654\"",
  "id": "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-network-security-test/providers/Microsoft.Network/privateEndpoints/pe-kv-app-prod/privateDnsZoneGroups/kv-dns-zone-group",
  "name": "kv-dns-zone-group",
  "privateDnsZoneConfigs": [
    {
      "name": "kvZone",
      "privateDnsZoneId": "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-network-security-test/providers/Microsoft.Network/privateDnsZones/privatelink.vaultcore.azure.net",
      "recordSets": [
        {
          "fqdn": "kv-tt-app-test-001.privatelink.vaultcore.azure.net",
          "ipAddresses": [
            "10.10.2.4"
          ],
          "provisioningState": "Succeeded",
          "recordSetName": "kv-tt-app-test-001",
          "recordType": "A",
          "ttl": 10
        }
      ]
    }
  ],
  "provisioningState": "Succeeded",
  "resourceGroup": "rg-network-security-test"
}

===========

tejas [ ~ ]$ az network private-endpoint show \
  --name $PRIVATE_ENDPOINT_NAME \
  --resource-group $RESOURCE_GROUP \
  --query "privateLinkServiceConnections[].privateLinkServiceConnectionState.status"
[
  "Approved"
]

=========== az keyvault update

tejas [ ~ ]$ az keyvault update \
  --name $KEYVAULT_NAME \
  --resource-group $RESOURCE_GROUP \
  --public-network-access Disabled
{
  "id": "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-security-test/providers/Microsoft.KeyVault/vaults/kv-tt-app-test-001",
  "location": "centralindia",
  "name": "kv-tt-app-test-001",
  "properties": {
    "accessPolicies": [],
    "createMode": null,
    "enablePurgeProtection": null,
    "enableRbacAuthorization": true,
    "enableSoftDelete": true,
    "enabledForDeployment": false,
    "enabledForDiskEncryption": false,
    "enabledForTemplateDeployment": false,
    "hsmPoolResourceId": null,
    "networkAcls": null,
    "privateEndpointConnections": [
      {
        "etag": null,
        "id": "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-security-test/providers/Microsoft.KeyVault/vaults/kv-tt-app-test-001/privateEndpointConnections/kv-private-connection",
        "privateEndpoint": {
          "id": "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-network-security-test/providers/Microsoft.Network/privateEndpoints/pe-kv-app-prod",
          "resourceGroup": "rg-network-security-test"
        },
        "privateLinkServiceConnectionState": {
          "actionsRequired": "None",
          "description": null,
          "status": "Approved"
        },
        "provisioningState": "Succeeded",
        "resourceGroup": "rg-security-test"
      }
    ],
    "provisioningState": "Succeeded",
    "publicNetworkAccess": "Disabled",
    "sku": {
      "family": "A",
      "name": "standard"
    },
    "softDeleteRetentionInDays": 90,
    "tenantId": "7faa8f70-7da4-4fa9-a24d-9bbd6682a962",
    "vaultUri": "https://kv-tt-app-test-001.vault.azure.net/"
  },
  "resourceGroup": "rg-security-test",
  "systemData": {
    "createdAt": "2026-01-24T16:38:08.827000+00:00",
    "createdBy": "tejastarghale47@gmail.com",
    "createdByType": "User",
    "lastModifiedAt": "2026-01-24T18:01:02.334000+00:00",
    "lastModifiedBy": "tejastarghale47@gmail.com",
    "lastModifiedByType": "User"
  },
  "tags": {},
  "type": "Microsoft.KeyVault/vaults"
}

=========== 

tejas [ ~ ]$ az group create \
  --name $RESOURCE_GROUP \
  --location $LOCATION
{
  "id": "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-vm-test",
  "location": "centralindia",
  "managedBy": null,
  "name": "rg-vm-test",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null,
  "type": "Microsoft.Resources/resourceGroups"
}

===========

tejas [ ~ ]$ az network nsg create \
  --resource-group $RESOURCE_GROUP \
  --name $NSG_NAME \
  --location $LOCATION
{
  "NewNSG": {
    "defaultSecurityRules": [
      {
        "access": "Allow",
        "description": "Allow inbound traffic from all VMs in VNET",
        "destinationAddressPrefix": "VirtualNetwork",
        "destinationAddressPrefixes": [],
        "destinationPortRange": "*",
        "destinationPortRanges": [],
        "direction": "Inbound",
        "etag": "W/\"fafedb79-4b10-4b7c-bf95-64f2b5a979aa\"",
        "id": "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-network-security-test/providers/Microsoft.Network/networkSecurityGroups/nsg-vm-app-test/defaultSecurityRules/AllowVnetInBound",
        "name": "AllowVnetInBound",
        "priority": 65000,
        "protocol": "*",
        "provisioningState": "Succeeded",
        "resourceGroup": "rg-network-security-test",
        "sourceAddressPrefix": "VirtualNetwork",
        "sourceAddressPrefixes": [],
        "sourcePortRange": "*",
        "sourcePortRanges": [],
        "type": "Microsoft.Network/networkSecurityGroups/defaultSecurityRules"
      },
      {
        "access": "Allow",
        "description": "Allow inbound traffic from azure load balancer",
        "destinationAddressPrefix": "*",
        "destinationAddressPrefixes": [],
        "destinationPortRange": "*",
        "destinationPortRanges": [],
        "direction": "Inbound",
        "etag": "W/\"fafedb79-4b10-4b7c-bf95-64f2b5a979aa\"",
        "id": "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-network-security-test/providers/Microsoft.Network/networkSecurityGroups/nsg-vm-app-test/defaultSecurityRules/AllowAzureLoadBalancerInBound",
        "name": "AllowAzureLoadBalancerInBound",
        "priority": 65001,
        "protocol": "*",
        "provisioningState": "Succeeded",
        "resourceGroup": "rg-network-security-test",
        "sourceAddressPrefix": "AzureLoadBalancer",
        "sourceAddressPrefixes": [],
        "sourcePortRange": "*",
        "sourcePortRanges": [],
        "type": "Microsoft.Network/networkSecurityGroups/defaultSecurityRules"
      },
      {
        "access": "Deny",
        "description": "Deny all inbound traffic",
        "destinationAddressPrefix": "*",
        "destinationAddressPrefixes": [],
        "destinationPortRange": "*",
        "destinationPortRanges": [],
        "direction": "Inbound",
        "etag": "W/\"fafedb79-4b10-4b7c-bf95-64f2b5a979aa\"",
        "id": "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-network-security-test/providers/Microsoft.Network/networkSecurityGroups/nsg-vm-app-test/defaultSecurityRules/DenyAllInBound",
        "name": "DenyAllInBound",
        "priority": 65500,
        "protocol": "*",
        "provisioningState": "Succeeded",
        "resourceGroup": "rg-network-security-test",
        "sourceAddressPrefix": "*",
        "sourceAddressPrefixes": [],
        "sourcePortRange": "*",
        "sourcePortRanges": [],
        "type": "Microsoft.Network/networkSecurityGroups/defaultSecurityRules"
      },
      {
        "access": "Allow",
        "description": "Allow outbound traffic from all VMs to all VMs in VNET",
        "destinationAddressPrefix": "VirtualNetwork",
        "destinationAddressPrefixes": [],
        "destinationPortRange": "*",
        "destinationPortRanges": [],
        "direction": "Outbound",
        "etag": "W/\"fafedb79-4b10-4b7c-bf95-64f2b5a979aa\"",
        "id": "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-network-security-test/providers/Microsoft.Network/networkSecurityGroups/nsg-vm-app-test/defaultSecurityRules/AllowVnetOutBound",
        "name": "AllowVnetOutBound",
        "priority": 65000,
        "protocol": "*",
        "provisioningState": "Succeeded",
        "resourceGroup": "rg-network-security-test",
        "sourceAddressPrefix": "VirtualNetwork",
        "sourceAddressPrefixes": [],
        "sourcePortRange": "*",
        "sourcePortRanges": [],
        "type": "Microsoft.Network/networkSecurityGroups/defaultSecurityRules"
      },
      {
        "access": "Allow",
        "description": "Allow outbound traffic from all VMs to Internet",
        "destinationAddressPrefix": "Internet",
        "destinationAddressPrefixes": [],
        "destinationPortRange": "*",
        "destinationPortRanges": [],
        "direction": "Outbound",
        "etag": "W/\"fafedb79-4b10-4b7c-bf95-64f2b5a979aa\"",
        "id": "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-network-security-test/providers/Microsoft.Network/networkSecurityGroups/nsg-vm-app-test/defaultSecurityRules/AllowInternetOutBound",
        "name": "AllowInternetOutBound",
        "priority": 65001,
        "protocol": "*",
        "provisioningState": "Succeeded",
        "resourceGroup": "rg-network-security-test",
        "sourceAddressPrefix": "*",
        "sourceAddressPrefixes": [],
        "sourcePortRange": "*",
        "sourcePortRanges": [],
        "type": "Microsoft.Network/networkSecurityGroups/defaultSecurityRules"
      },
      {
        "access": "Deny",
        "description": "Deny all outbound traffic",
        "destinationAddressPrefix": "*",
        "destinationAddressPrefixes": [],
        "destinationPortRange": "*",
        "destinationPortRanges": [],
        "direction": "Outbound",
        "etag": "W/\"fafedb79-4b10-4b7c-bf95-64f2b5a979aa\"",
        "id": "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-network-security-test/providers/Microsoft.Network/networkSecurityGroups/nsg-vm-app-test/defaultSecurityRules/DenyAllOutBound",
        "name": "DenyAllOutBound",
        "priority": 65500,
        "protocol": "*",
        "provisioningState": "Succeeded",
        "resourceGroup": "rg-network-security-test",
        "sourceAddressPrefix": "*",
        "sourceAddressPrefixes": [],
        "sourcePortRange": "*",
        "sourcePortRanges": [],
        "type": "Microsoft.Network/networkSecurityGroups/defaultSecurityRules"
      }
    ],
    "etag": "W/\"fafedb79-4b10-4b7c-bf95-64f2b5a979aa\"",
    "id": "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-network-security-test/providers/Microsoft.Network/networkSecurityGroups/nsg-vm-app-test",
    "location": "centralindia",
    "name": "nsg-vm-app-test",
    "provisioningState": "Succeeded",
    "resourceGroup": "rg-network-security-test",
    "resourceGuid": "13b54886-c1e0-4c6b-ae01-ad54557b0964",
    "securityRules": [],
    "type": "Microsoft.Network/networkSecurityGroups"
  }
}

==========

tejas [ ~ ]$ az network nsg rule create \
  --resource-group $RESOURCE_GROUP \
  --nsg-name $NSG_NAME \
  --name Allow-SSH \
  --priority 1000 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --destination-port-range 22 \
  --source-address-prefixes <YOUR_PUBLIC_IP>
bash: syntax error near unexpected token `newline'

==========

tejas [ ~ ]$ az network nic create \
  --resource-group $RESOURCE_GROUP \
  --name $NIC_NAME \
  --vnet-name $VNET_NAME \
  --subnet $SUBNET_NAME \
  --network-security-group $NSG_NAME
{
  "NewNIC": {
    "auxiliaryMode": "None",
    "auxiliarySku": "None",
    "disableTcpStateTracking": false,
    "dnsSettings": {
      "appliedDnsServers": [],
      "dnsServers": [],
      "internalDomainNameSuffix": "5ki3wmmeljtutc2gfjzpgngnya.rx.internal.cloudapp.net"
    },
    "enableAcceleratedNetworking": false,
    "enableIPForwarding": false,
    "etag": "W/\"5711beda-9e0f-410a-9648-474ecc27ab89\"",
    "hostedWorkloads": [],
    "id": "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-network-security-test/providers/Microsoft.Network/networkInterfaces/nic-vm-app-test-01",
    "ipConfigurations": [
      {
        "etag": "W/\"5711beda-9e0f-410a-9648-474ecc27ab89\"",
        "id": "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-network-security-test/providers/Microsoft.Network/networkInterfaces/nic-vm-app-test-01/ipConfigurations/ipconfig1",
        "name": "ipconfig1",
        "primary": true,
        "privateIPAddress": "10.10.1.4",
        "privateIPAddressVersion": "IPv4",
        "privateIPAllocationMethod": "Dynamic",
        "provisioningState": "Succeeded",
        "resourceGroup": "rg-network-security-test",
        "subnet": {
          "id": "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-network-security-test/providers/Microsoft.Network/virtualNetworks/vnet-app-test/subnets/subnet-vm",
          "resourceGroup": "rg-network-security-test"
        },
        "type": "Microsoft.Network/networkInterfaces/ipConfigurations"
      }
    ],
    "location": "centralindia",
    "name": "nic-vm-app-test-01",
    "networkSecurityGroup": {
      "id": "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-network-security-test/providers/Microsoft.Network/networkSecurityGroups/nsg-vm-app-test",
      "resourceGroup": "rg-network-security-test"
    },
    "nicType": "Standard",
    "provisioningState": "Succeeded",
    "resourceGroup": "rg-network-security-test",
    "resourceGuid": "0f0aa015-5ab3-4f8f-b2c5-88ce1f40372e",
    "tapConfigurations": [],
    "type": "Microsoft.Network/networkInterfaces",
    "vnetEncryptionSupported": false
  }
}

============

tejas [ ~ ]$ az vm create \
  --resource-group $RESOURCE_GROUP \
  --name $VM_NAME \
  --location $LOCATION \
  --nics $NIC_NAME \
  --image Ubuntu2204 \
  --size $VM_SIZE \
  --admin-username $ADMIN_USERNAME \
  --admin-password $ADMIN_PASSWORD \
  --authentication-type password \
  --assign-identity \
  --public-ip-address ""
The default value of '--size' will be changed to 'Standard_D2s_v5' from 'Standard_DS1_v2' in a future release.
No access was given yet to the 'vm-app-test-01', because '--scope' was not provided. You should setup by creating a role assignment, e.g. 'az role assignment create --assignee <principal-id> --role contributor -g rg-network-security-test' would let it access the current resource group. To get the pricipal id, run 'az vm show -g rg-network-security-test -n vm-app-test-01 --query "identity.principalId" -otsv'
{
  "fqdns": "",
  "id": "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-network-security-test/providers/Microsoft.Compute/virtualMachines/vm-app-test-01",
  "identity": {
    "systemAssignedIdentity": "e40c9a57-b8d0-4e63-a467-60a208db0862",
    "userAssignedIdentities": {}
  },
  "location": "centralindia",
  "macAddress": "7C-ED-8D-26-AC-18",
  "powerState": "VM running",
  "privateIpAddress": "10.10.1.4",
  "publicIpAddress": "",
  "resourceGroup": "rg-network-security-test"
}

===========

tejas [ ~ ]$ az vm show \
  --resource-group $RESOURCE_GROUP \
  --name $VM_NAME \
  --output table
Name            ResourceGroup             Location      Zones
--------------  ------------------------  ------------  -------
vm-app-test-01  rg-network-security-test  centralindia

===========

tejas [ ~ ]$ az vm delete \
  --resource-group $RESOURCE_GROUP \
  --name $VM_NAME \
  --yes
tejas [ ~ ]$ 

===========

tejas [ ~ ]$ SUBSCRIPTION_ID="52f167e5-dfca-4d77-a744-e6c7bc1a3235"
RESOURCE_GROUP="rg-vm-test"
LOCATION="centralindia"

VM_NAME="vm-app-test-01"
VM_SIZE="Standard_B2s"

VNET_NAME="vnet-app-test"
SUBNET_NAME="subnet-vm"

NIC_NAME="nic-vm-app-test-01"
NSG_NAME="nsg-vm-app-test"

ADMIN_USERNAME="azureuser"
ADMIN_PASSWORD="P@ssw0rd@123!"
tejas [ ~ ]$ az vm create \
  --resource-group $RESOURCE_GROUP \
  --name $VM_NAME \
  --location $LOCATION \
  --nics $NIC_NAME \
  --image Ubuntu2204 \
  --size $VM_SIZE \
  --admin-username $ADMIN_USERNAME \
  --admin-password $ADMIN_PASSWORD \
  --authentication-type password \
  --assign-identity \
  --public-ip-address ""
The default value of '--size' will be changed to 'Standard_D2s_v5' from 'Standard_DS1_v2' in a future release.
{"status":"Failed","error":{"code":"DeploymentFailed","target":"/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-vm-test/providers/Microsoft.Resources/deployments/vm_deploy_NNVgl7odnSD73zVeXpk0zgJCmW9X6Ls9","message":"At least one resource deployment operation failed. Please list deployment operations for details. Please see https://aka.ms/arm-deployment-operations for usage details.","details":[{"code":"NotFound","message":"Resource /subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-vm-test/providers/Microsoft.Network/networkInterfaces/nic-vm-app-test-01 not found.","details":[]}]}}

=============

tejas [ ~ ]$ az network nsg create \
  --resource-group $RESOURCE_GROUP \
  --name $NSG_NAME \
  --location $LOCATION
{
  "NewNSG": {
    "defaultSecurityRules": [
      {
        "access": "Allow",
        "description": "Allow inbound traffic from all VMs in VNET",
        "destinationAddressPrefix": "VirtualNetwork",
        "destinationAddressPrefixes": [],
        "destinationPortRange": "*",
        "destinationPortRanges": [],
        "direction": "Inbound",
        "etag": "W/\"af1979e6-1263-48ea-a5f7-f6a158dc9de5\"",
        "id": "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-vm-test/providers/Microsoft.Network/networkSecurityGroups/nsg-vm-app-test/defaultSecurityRules/AllowVnetInBound",
        "name": "AllowVnetInBound",
        "priority": 65000,
        "protocol": "*",
        "provisioningState": "Succeeded",
        "resourceGroup": "rg-vm-test",
        "sourceAddressPrefix": "VirtualNetwork",
        "sourceAddressPrefixes": [],
        "sourcePortRange": "*",
        "sourcePortRanges": [],
        "type": "Microsoft.Network/networkSecurityGroups/defaultSecurityRules"
      },
      {
        "access": "Allow",
        "description": "Allow inbound traffic from azure load balancer",
        "destinationAddressPrefix": "*",
        "destinationAddressPrefixes": [],
        "destinationPortRange": "*",
        "destinationPortRanges": [],
        "direction": "Inbound",
        "etag": "W/\"af1979e6-1263-48ea-a5f7-f6a158dc9de5\"",
        "id": "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-vm-test/providers/Microsoft.Network/networkSecurityGroups/nsg-vm-app-test/defaultSecurityRules/AllowAzureLoadBalancerInBound",
        "name": "AllowAzureLoadBalancerInBound",
        "priority": 65001,
        "protocol": "*",
        "provisioningState": "Succeeded",
        "resourceGroup": "rg-vm-test",
        "sourceAddressPrefix": "AzureLoadBalancer",
        "sourceAddressPrefixes": [],
        "sourcePortRange": "*",
        "sourcePortRanges": [],
        "type": "Microsoft.Network/networkSecurityGroups/defaultSecurityRules"
      },
      {
        "access": "Deny",
        "description": "Deny all inbound traffic",
        "destinationAddressPrefix": "*",
        "destinationAddressPrefixes": [],
        "destinationPortRange": "*",
        "destinationPortRanges": [],
        "direction": "Inbound",
        "etag": "W/\"af1979e6-1263-48ea-a5f7-f6a158dc9de5\"",
        "id": "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-vm-test/providers/Microsoft.Network/networkSecurityGroups/nsg-vm-app-test/defaultSecurityRules/DenyAllInBound",
        "name": "DenyAllInBound",
        "priority": 65500,
        "protocol": "*",
        "provisioningState": "Succeeded",
        "resourceGroup": "rg-vm-test",
        "sourceAddressPrefix": "*",
        "sourceAddressPrefixes": [],
        "sourcePortRange": "*",
        "sourcePortRanges": [],
        "type": "Microsoft.Network/networkSecurityGroups/defaultSecurityRules"
      },
      {
        "access": "Allow",
        "description": "Allow outbound traffic from all VMs to all VMs in VNET",
        "destinationAddressPrefix": "VirtualNetwork",
        "destinationAddressPrefixes": [],
        "destinationPortRange": "*",
        "destinationPortRanges": [],
        "direction": "Outbound",
        "etag": "W/\"af1979e6-1263-48ea-a5f7-f6a158dc9de5\"",
        "id": "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-vm-test/providers/Microsoft.Network/networkSecurityGroups/nsg-vm-app-test/defaultSecurityRules/AllowVnetOutBound",
        "name": "AllowVnetOutBound",
        "priority": 65000,
        "protocol": "*",
        "provisioningState": "Succeeded",
        "resourceGroup": "rg-vm-test",
        "sourceAddressPrefix": "VirtualNetwork",
        "sourceAddressPrefixes": [],
        "sourcePortRange": "*",
        "sourcePortRanges": [],
        "type": "Microsoft.Network/networkSecurityGroups/defaultSecurityRules"
      },
      {
        "access": "Allow",
        "description": "Allow outbound traffic from all VMs to Internet",
        "destinationAddressPrefix": "Internet",
        "destinationAddressPrefixes": [],
        "destinationPortRange": "*",
        "destinationPortRanges": [],
        "direction": "Outbound",
        "etag": "W/\"af1979e6-1263-48ea-a5f7-f6a158dc9de5\"",
        "id": "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-vm-test/providers/Microsoft.Network/networkSecurityGroups/nsg-vm-app-test/defaultSecurityRules/AllowInternetOutBound",
        "name": "AllowInternetOutBound",
        "priority": 65001,
        "protocol": "*",
        "provisioningState": "Succeeded",
        "resourceGroup": "rg-vm-test",
        "sourceAddressPrefix": "*",
        "sourceAddressPrefixes": [],
        "sourcePortRange": "*",
        "sourcePortRanges": [],
        "type": "Microsoft.Network/networkSecurityGroups/defaultSecurityRules"
      },
      {
        "access": "Deny",
        "description": "Deny all outbound traffic",
        "destinationAddressPrefix": "*",
        "destinationAddressPrefixes": [],
        "destinationPortRange": "*",
        "destinationPortRanges": [],
        "direction": "Outbound",
        "etag": "W/\"af1979e6-1263-48ea-a5f7-f6a158dc9de5\"",
        "id": "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-vm-test/providers/Microsoft.Network/networkSecurityGroups/nsg-vm-app-test/defaultSecurityRules/DenyAllOutBound",
        "name": "DenyAllOutBound",
        "priority": 65500,
        "protocol": "*",
        "provisioningState": "Succeeded",
        "resourceGroup": "rg-vm-test",
        "sourceAddressPrefix": "*",
        "sourceAddressPrefixes": [],
        "sourcePortRange": "*",
        "sourcePortRanges": [],
        "type": "Microsoft.Network/networkSecurityGroups/defaultSecurityRules"
      }
    ],
    "etag": "W/\"af1979e6-1263-48ea-a5f7-f6a158dc9de5\"",
    "id": "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-vm-test/providers/Microsoft.Network/networkSecurityGroups/nsg-vm-app-test",
    "location": "centralindia",
    "name": "nsg-vm-app-test",
    "provisioningState": "Succeeded",
    "resourceGroup": "rg-vm-test",
    "resourceGuid": "24c3ce88-7624-4bbe-915a-972ff3333082",
    "securityRules": [],
    "type": "Microsoft.Network/networkSecurityGroups"
  }
}

==========

tejas [ ~ ]$ az network nic create \
  --resource-group $RESOURCE_GROUP \
  --name $NIC_NAME \
  --subnet $SUB_ID \
  --network-security-group $NSG_NAME
{
  "NewNIC": {
    "auxiliaryMode": "None",
    "auxiliarySku": "None",
    "disableTcpStateTracking": false,
    "dnsSettings": {
      "appliedDnsServers": [],
      "dnsServers": [],
      "internalDomainNameSuffix": "5ki3wmmeljtutc2gfjzpgngnya.rx.internal.cloudapp.net"
    },
    "enableAcceleratedNetworking": false,
    "enableIPForwarding": false,
    "etag": "W/\"2fb4e9a7-fda3-46b5-a009-293f0c9412ac\"",
    "hostedWorkloads": [],
    "id": "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-vm-test/providers/Microsoft.Network/networkInterfaces/nic-vm-app-test-01",
    "ipConfigurations": [
      {
        "etag": "W/\"2fb4e9a7-fda3-46b5-a009-293f0c9412ac\"",
        "id": "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-vm-test/providers/Microsoft.Network/networkInterfaces/nic-vm-app-test-01/ipConfigurations/ipconfig1",
        "name": "ipconfig1",
        "primary": true,
        "privateIPAddress": "10.10.1.4",
        "privateIPAddressVersion": "IPv4",
        "privateIPAllocationMethod": "Dynamic",
        "provisioningState": "Succeeded",
        "resourceGroup": "rg-vm-test",
        "subnet": {
          "id": "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-network-security-test/providers/Microsoft.Network/virtualNetworks/vnet-app-test/subnets/subnet-vm",
          "resourceGroup": "rg-network-security-test"
        },
        "type": "Microsoft.Network/networkInterfaces/ipConfigurations"
      }
    ],
    "location": "centralindia",
    "name": "nic-vm-app-test-01",
    "networkSecurityGroup": {
      "id": "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-vm-test/providers/Microsoft.Network/networkSecurityGroups/nsg-vm-app-test",
      "resourceGroup": "rg-vm-test"
    },
    "nicType": "Standard",
    "provisioningState": "Succeeded",
    "resourceGroup": "rg-vm-test",
    "resourceGuid": "7311657b-360c-4da2-ac8a-ed32e69bd3fa",
    "tapConfigurations": [],
    "type": "Microsoft.Network/networkInterfaces",
    "vnetEncryptionSupported": false
  }
}

==============

tejas [ ~ ]$ az vm create \
  --resource-group $RESOURCE_GROUP \
  --name $VM_NAME \
  --location $LOCATION \
  --nics $NIC_NAME \
  --image Ubuntu2204 \
  --size $VM_SIZE \
  --admin-username $ADMIN_USERNAME \
  --admin-password $ADMIN_PASSWORD \
  --authentication-type password \
  --assign-identity \
  --public-ip-address ""
The default value of '--size' will be changed to 'Standard_D2s_v5' from 'Standard_DS1_v2' in a future release.
No access was given yet to the 'vm-app-test-01', because '--scope' was not provided. You should setup by creating a role assignment, e.g. 'az role assignment create --assignee <principal-id> --role contributor -g rg-vm-test' would let it access the current resource group. To get the pricipal id, run 'az vm show -g rg-vm-test -n vm-app-test-01 --query "identity.principalId" -otsv'
{
  "fqdns": "",
  "id": "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-vm-test/providers/Microsoft.Compute/virtualMachines/vm-app-test-01",
  "identity": {
    "systemAssignedIdentity": "52c6a3c9-8c7f-4465-9f40-b7146ac4e71d",
    "userAssignedIdentities": {}
  },
  "location": "centralindia",
  "macAddress": "7C-ED-8D-27-29-A1",
  "powerState": "VM running",
  "privateIpAddress": "10.10.1.4",
  "publicIpAddress": "",
  "resourceGroup": "rg-vm-test"
}

=============

tejas [ ~ ]$ SUBSCRIPTION_ID="52f167e5-dfca-4d77-a744-e6c7bc1a3235"
RESOURCE_GROUP="rg-network-security-test"
LOCATION="centralindia"

PUBLIC_IP_NAME="pip-vm-app-test-01"
DNS_LABEL="vm-app-test-01"
tejas [ ~ ]$ az network public-ip create \
  --resource-group $RESOURCE_GROUP \
  --name $PUBLIC_IP_NAME \
  --location $LOCATION \
  --sku Standard \
  --allocation-method Static \
  --version IPv4 \
  --dns-name $DNS_LABEL
[Coming breaking change] In the coming release, the default behavior will be changed as follows when sku is Standard and zone is not provided: For zonal regions, you will get a zone-redundant IP indicated by zones:["1","2","3"]; For non-zonal regions, you will get a non zone-redundant IP indicated by zones:null.
{
  "publicIp": {
    "ddosSettings": {
      "protectionMode": "VirtualNetworkInherited"
    },
    "dnsSettings": {
      "domainNameLabel": "vm-app-test-01",
      "fqdn": "vm-app-test-01.centralindia.cloudapp.azure.com"
    },
    "etag": "W/\"8dcab29a-0cc5-4a87-9c37-1d1544ca025e\"",
    "id": "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-network-security-test/providers/Microsoft.Network/publicIPAddresses/pip-vm-app-test-01",
    "idleTimeoutInMinutes": 4,
    "ipAddress": "20.192.0.124",
    "ipTags": [],
    "location": "centralindia",
    "name": "pip-vm-app-test-01",
    "provisioningState": "Succeeded",
    "publicIPAddressVersion": "IPv4",
    "publicIPAllocationMethod": "Static",
    "resourceGroup": "rg-network-security-test",
    "resourceGuid": "b923ccf7-ee54-45b2-a0dc-0fa1b723f5ee",
    "sku": {
      "name": "Standard",
      "tier": "Regional"
    },
    "type": "Microsoft.Network/publicIPAddresses"
  }
}

===========

tejas [ ~ ]$ az network public-ip show \
  --resource-group $RESOURCE_GROUP \
  --name $PUBLIC_IP_NAME \
  --output table
Name                ResourceGroup             Location      Zones    Address       IdleTimeoutInMinutes    ProvisioningState
------------------  ------------------------  ------------  -------  ------------  ----------------------  -------------------
pip-vm-app-test-01  rg-network-security-test  centralindia           20.192.0.124  4                       Succeeded

===========

tejas [ ~ ]$ PUBLIC_IP_NAME=$(az network public-ip show \
  --resource-group rg-network-security-test \
  --name pip-vm-app-test-01 \
  --query id -o tsv)
tejas [ ~ ]$ echo "RG: $PUBLIC_IP_NAME"
RG: /subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-network-security-test/providers/Microsoft.Network/publicIPAddresses/pip-vm-app-test-01
tejas [ ~ ]$   az network nic ip-config update \
  --resource-group $RESOURCE_GROUP \
  --nic-name $NIC_NAME \
  --name ipconfig1 \
  --public-ip-address $PUBLIC_IP_NAME
{
  "etag": "W/\"de1cc005-895f-4e2c-a4aa-b58bbe4f9f76\"",
  "id": "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-vm-test/providers/Microsoft.Network/networkInterfaces/nic-vm-app-test-01/ipConfigurations/ipconfig1",
  "name": "ipconfig1",
  "primary": true,
  "privateIPAddress": "10.10.1.4",
  "privateIPAddressVersion": "IPv4",
  "privateIPAllocationMethod": "Dynamic",
  "provisioningState": "Succeeded",
  "publicIPAddress": {
    "id": "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-network-security-test/providers/Microsoft.Network/publicIPAddresses/pip-vm-app-test-01",
    "resourceGroup": "rg-network-security-test"
  },
  "resourceGroup": "rg-vm-test",
  "subnet": {
    "id": "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-network-security-test/providers/Microsoft.Network/virtualNetworks/vnet-app-test/subnets/subnet-vm",
    "resourceGroup": "rg-network-security-test"
  },
  "type": "Microsoft.Network/networkInterfaces/ipConfigurations"
}

============

tejas [ ~ ]$   az vm list-ip-addresses \
  --resource-group $RESOURCE_GROUP \
  --name $VM_NAME \
  --output table
VirtualMachine    PublicIPAddresses    PrivateIPAddresses
----------------  -------------------  --------------------
vm-app-test-01    20.192.0.124         10.10.1.4

============

tejas [ ~ ]$ SUBSCRIPTION_ID="52f167e5-dfca-4d77-a744-e6c7bc1a3235"
RESOURCE_GROUP="rg-vm-test"
LOCATION="centralindia"

VM_NAME="vm-app-test-01"
VM_SIZE="Standard_B2s"

VNET_NAME="vnet-app-test"
SUBNET_NAME="subnet-vm"

NIC_NAME="nic-vm-app-test-01"
NSG_NAME="nsg-vm-app-test"

ADMIN_USERNAME="azureuser"
ADMIN_PASSWORD="P@ssw0rd@123!"
tejas [ ~ ]$ az network nsg rule create \
  --resource-group $RESOURCE_GROUP \
  --nsg-name $NSG_NAME \
  --name Allow-SSH \
  --priority 1000 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --destination-port-range 22 \
  --source-address-prefixes 20.192.0.124
{
  "access": "Allow",
  "destinationAddressPrefix": "*",
  "destinationAddressPrefixes": [],
  "destinationPortRange": "22",
  "destinationPortRanges": [],
  "direction": "Inbound",
  "etag": "W/\"0be7490e-6741-4c92-b8c3-eb4ddf607453\"",
  "id": "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-vm-test/providers/Microsoft.Network/networkSecurityGroups/nsg-vm-app-test/securityRules/Allow-SSH",
  "name": "Allow-SSH",
  "priority": 1000,
  "protocol": "Tcp",
  "provisioningState": "Succeeded",
  "resourceGroup": "rg-vm-test",
  "sourceAddressPrefix": "20.192.0.124",
  "sourceAddressPrefixes": [],
  "sourcePortRange": "*",
  "sourcePortRanges": [],
  "type": "Microsoft.Network/networkSecurityGroups/securityRules"
}

==========

tejas [ ~ ]$ az network nsg rule show \
  --resource-group rg-vm-test \
  --nsg-name nsg-vm-app-test \
  --name Allow-SSH
{
  "access": "Allow",
  "destinationAddressPrefix": "*",
  "destinationAddressPrefixes": [],
  "destinationPortRange": "22",
  "destinationPortRanges": [],
  "direction": "Inbound",
  "etag": "W/\"0be7490e-6741-4c92-b8c3-eb4ddf607453\"",
  "id": "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-vm-test/providers/Microsoft.Network/networkSecurityGroups/nsg-vm-app-test/securityRules/Allow-SSH",
  "name": "Allow-SSH",
  "priority": 1000,
  "protocol": "Tcp",
  "provisioningState": "Succeeded",
  "resourceGroup": "rg-vm-test",
  "sourceAddressPrefix": "20.192.0.124",
  "sourceAddressPrefixes": [],
  "sourcePortRange": "*",
  "sourcePortRanges": [],
  "type": "Microsoft.Network/networkSecurityGroups/securityRules"
}
tejas [ ~ ]$ az network nsg rule update \
  --resource-group rg-vm-test \
  --nsg-name nsg-vm-app-test \
  --name Allow-SSH \
  --source-address-prefixes 103.248.75.84
{
  "access": "Allow",
  "destinationAddressPrefix": "*",
  "destinationAddressPrefixes": [],
  "destinationPortRange": "22",
  "destinationPortRanges": [],
  "direction": "Inbound",
  "etag": "W/\"3c5a4332-9ca7-487b-a464-cc37edf00353\"",
  "id": "/subscriptions/52f167e5-dfca-4d77-a744-e6c7bc1a3235/resourceGroups/rg-vm-test/providers/Microsoft.Network/networkSecurityGroups/nsg-vm-app-test/securityRules/Allow-SSH",
  "name": "Allow-SSH",
  "priority": 1000,
  "protocol": "Tcp",
  "provisioningState": "Succeeded",
  "resourceGroup": "rg-vm-test",
  "sourceAddressPrefix": "103.248.75.84",
  "sourceAddressPrefixes": [],
  "sourcePortRange": "*",
  "sourcePortRanges": [],
  "type": "Microsoft.Network/networkSecurityGroups/securityRules"
}

==========