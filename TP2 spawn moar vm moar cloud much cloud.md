# Part I : Programmatic approach

## I. Premiers pas

### VM Creation
```console
┌──(alt㉿Altiera)-[~/Documents/School/Cloud]  
└─$ az group create --name tp2 --location westeurope                                                          
{  
 "id": "/subscriptions/eae5cd03-388e-4052-8aa9-90a5958d32b8/resourceGroups/tp2",  
 "location": "westeurope",  
 "managedBy": null,  
 "name": "tp2",  
 "properties": {  
   "provisioningState": "Succeeded"  
 },  
 "tags": null,  
 "type": "Microsoft.Resources/resourceGroups"  
}

┌──(alt㉿Altiera)-[~/Documents/School/Cloud]  
└─$ az vm create -g tp2 -n poder --image Ubuntu2204 --admin-username alt --ssh-key-values ~/.ssh/id_ed25519.pub    
{  
 "fqdns": "",  
 "id": "/subscriptions/eae5cd03-388e-4052-8aa9-90a5958d32b8/resourceGroups/tp2/providers/Microsoft.Compute/virtualMachines/poder",  
 "location": "westeurope",  
 "macAddress": "00-0D-3A-45-AE-44",  
 "powerState": "VM running",  
 "privateIpAddress": "10.0.0.4",  
 "publicIpAddress": "13.81.29.167",  
 "resourceGroup": "tp2",  
 "zones": ""  
}
```

### Connection test
```console
┌──(alt㉿Altiera)-[~/Documents/School/Cloud]  
└─$ az vm list-ip-addresses --output table  
  
VirtualMachine    PublicIPAddresses    PrivateIPAddresses  
----------------  -------------------  --------------------  
TP1-YMCA          51.137.90.104        10.0.0.4  
poder             13.81.29.167         10.0.0.4  
                                                                                                                                                                                             
┌──(alt㉿Altiera)-[~/Documents/School/Cloud]  
└─$ ssh alt@13.81.29.167    

...
  
alt@poder:~$
```

### Checking walinuxagent & cloud init
```console 
alt@poder:~$ systemctl status walinuxagent  
● walinuxagent.service - Azure Linux Agent  
    Loaded: loaded (/lib/systemd/system/walinuxagent.service; enabled; vendor preset: enabled)  
    Active: active (running) since Tue 2025-04-01 08:31:54 UTC; 4min 15s ago
    ...
    
alt@poder:~$ systemctl status cloud-init  
● cloud-init.service - Cloud-init: Network Stage  
    Loaded: loaded (/lib/systemd/system/cloud-init.service; enabled; vendor preset: enabled)  
    Active: active (exited) since Tue 2025-04-01 08:31:54 UTC; 4min 59s ago  
  Main PID: 506 (code=exited, status=0/SUCCESS)  
       CPU: 1.600s
       ...
```

## II. Un ptit LAN

### VNET creation
```console 
──(alt㉿Altiera)-[~/Documents/School/Cloud]  
└─$ az network vnet create -g tp2 -n TP2_VNET --address-prefix 10.0.0.0/16 --subnet-name TP2Subnet --subnet-prefixes 10.0.0.0/24    
{  
 "newVNet": {  
   "addressSpace": {  
     "addressPrefixes": [  
       "10.0.0.0/16"  
     ]  
   },  
   "enableDdosProtection": false,  
   "etag": "W/\"deeba048-d073-41aa-9e67-4c161ac30672\"",  
   "id": "/subscriptions/eae5cd03-388e-4052-8aa9-90a5958d32b8/resourceGroups/tp2/providers/Microsoft.Network/virtualNetworks/TP2_VNET",  
   "location": "westeurope",  
   "name": "TP2_VNET",  
   "privateEndpointVNetPolicies": "Disabled",  
   "provisioningState": "Succeeded",  
   "resourceGroup": "tp2",  
   "resourceGuid": "efa29c5a-3956-4ee3-85d1-9bc54084a30a",  
   "subnets": [  
     {  
       "addressPrefix": "10.0.0.0/24",  
       "delegations": [],  
       "etag": "W/\"deeba048-d073-41aa-9e67-4c161ac30672\"",  
       "id": "/subscriptions/eae5cd03-388e-4052-8aa9-90a5958d32b8/resourceGroups/tp2/providers/Microsoft.Network/virtualNetworks/TP2_VNET/subnets/TP2Subnet",  
       "name": "TP2Subnet",  
       "privateEndpointNetworkPolicies": "Disabled",  
       "privateLinkServiceNetworkPolicies": "Enabled",  
       "provisioningState": "Succeeded",  
       "resourceGroup": "tp2",  
       "type": "Microsoft.Network/virtualNetworks/subnets"  
     }  
   ],  
   "type": "Microsoft.Network/virtualNetworks",  
   "virtualNetworkPeerings": []  
 }  
}
```

### Public IPs creation
```console 
┌──(alt㉿Altiera)-[~/Documents/School/Cloud]  
└─$ az network public-ip create --resource-group TP2 --name TP2_VM_IP1 --sku Standard  
az network public-ip create --resource-group TP2 --name TP2_VM_IP2 --sku Standard  
  

{  
 "publicIp": {  
   "ddosSettings": {  
     "protectionMode": "VirtualNetworkInherited"  
   },  
   "etag": "W/\"037a96ef-4ad6-4673-8307-d376ed15f23c\"",  
   "id": "/subscriptions/eae5cd03-388e-4052-8aa9-90a5958d32b8/resourceGroups/TP2/providers/Microsoft.Network/publicIPAddresses/TP2_VM_IP1",  
   "idleTimeoutInMinutes": 4,  
   "ipAddress": "20.23.229.191",  
   "ipTags": [],  
   "location": "westeurope",  
   "name": "TP2_VM_IP1",  
   "provisioningState": "Succeeded",  
   "publicIPAddressVersion": "IPv4",  
   "publicIPAllocationMethod": "Static",  
   "resourceGroup": "TP2",  
   "resourceGuid": "6c8d64d4-848c-4a01-9e77-7ba966a83c05",  
   "sku": {  
     "name": "Standard",  
     "tier": "Regional"  
   },  
   "type": "Microsoft.Network/publicIPAddresses"  
 }  
}  
 
{  
 "publicIp": {  
   "ddosSettings": {  
     "protectionMode": "VirtualNetworkInherited"  
   },  
   "etag": "W/\"e6948411-c1d8-4e4b-b2e4-260da9f4aab2\"",  
   "id": "/subscriptions/eae5cd03-388e-4052-8aa9-90a5958d32b8/resourceGroups/TP2/providers/Microsoft.Network/publicIPAddresses/TP2_VM_IP2",  
   "idleTimeoutInMinutes": 4,  
   "ipAddress": "20.23.229.227",  
   "ipTags": [],  
   "location": "westeurope",  
   "name": "TP2_VM_IP2",  
   "provisioningState": "Succeeded",  
   "publicIPAddressVersion": "IPv4",  
   "publicIPAllocationMethod": "Static",  
   "resourceGroup": "TP2",  
   "resourceGuid": "469ecf25-0c76-4ada-b0fd-7fddbf3579fb",  
   "sku": {  
     "name": "Standard",  
     "tier": "Regional"  
   },  
   "type": "Microsoft.Network/publicIPAddresses"  
 }  
}
```
### Network Security Group Creation
```console
┌──(alt㉿Altiera)-[~/Documents/School/Cloud]  
└─$ az network nsg create --resource-group TP2 --name TP2NSG  
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
       "etag": "W/\"5cfc141e-747b-4d4b-9496-be7a4bda6737\"",  
       "id": "/subscriptions/eae5cd03-388e-4052-8aa9-90a5958d32b8/resourceGroups/TP2/providers/Microsoft.Network/networkSecurityGroups/TP2NSG/defaultSecurityRules/AllowVnetInBound",  
       "name": "AllowVnetInBound",  
       "priority": 65000,  
       "protocol": "*",  
       "provisioningState": "Succeeded",  
       "resourceGroup": "TP2",  
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
       "etag": "W/\"5cfc141e-747b-4d4b-9496-be7a4bda6737\"",  
       "id": "/subscriptions/eae5cd03-388e-4052-8aa9-90a5958d32b8/resourceGroups/TP2/providers/Microsoft.Network/networkSecurityGroups/TP2NSG/defaultSecurityRules/AllowAzureLoadBalancerI  
nBound",  
       "name": "AllowAzureLoadBalancerInBound",  
       "priority": 65001,  
       "protocol": "*",  
       "provisioningState": "Succeeded",  
       "resourceGroup": "TP2",  
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
       "etag": "W/\"5cfc141e-747b-4d4b-9496-be7a4bda6737\"",  
       "id": "/subscriptions/eae5cd03-388e-4052-8aa9-90a5958d32b8/resourceGroups/TP2/providers/Microsoft.Network/networkSecurityGroups/TP2NSG/defaultSecurityRules/DenyAllInBound",  
       "name": "DenyAllInBound",  
       "priority": 65500,  
       "protocol": "*",  
       "provisioningState": "Succeeded",  
       "resourceGroup": "TP2",  
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
       "etag": "W/\"5cfc141e-747b-4d4b-9496-be7a4bda6737\"",  
       "id": "/subscriptions/eae5cd03-388e-4052-8aa9-90a5958d32b8/resourceGroups/TP2/providers/Microsoft.Network/networkSecurityGroups/TP2NSG/defaultSecurityRules/AllowVnetOutBound",  
       "name": "AllowVnetOutBound",  
       "priority": 65000,  
       "protocol": "*",  
       "provisioningState": "Succeeded",  
       "resourceGroup": "TP2",  
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
       "etag": "W/\"5cfc141e-747b-4d4b-9496-be7a4bda6737\"",  
       "id": "/subscriptions/eae5cd03-388e-4052-8aa9-90a5958d32b8/resourceGroups/TP2/providers/Microsoft.Network/networkSecurityGroups/TP2NSG/defaultSecurityRules/AllowInternetOutBound",  
       "name": "AllowInternetOutBound",  
       "priority": 65001,  
       "protocol": "*",  
       "provisioningState": "Succeeded",  
       "resourceGroup": "TP2",  
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
       "etag": "W/\"5cfc141e-747b-4d4b-9496-be7a4bda6737\"",  
       "id": "/subscriptions/eae5cd03-388e-4052-8aa9-90a5958d32b8/resourceGroups/TP2/providers/Microsoft.Network/networkSecurityGroups/TP2NSG/defaultSecurityRules/DenyAllOutBound",  
       "name": "DenyAllOutBound",  
       "priority": 65500,  
       "protocol": "*",  
       "provisioningState": "Succeeded",  
       "resourceGroup": "TP2",  
       "sourceAddressPrefix": "*",  
       "sourceAddressPrefixes": [],  
       "sourcePortRange": "*",  
       "sourcePortRanges": [],  
       "type": "Microsoft.Network/networkSecurityGroups/defaultSecurityRules"  
     }  
   ],  
   "etag": "W/\"5cfc141e-747b-4d4b-9496-be7a4bda6737\"",  
   "id": "/subscriptions/eae5cd03-388e-4052-8aa9-90a5958d32b8/resourceGroups/TP2/providers/Microsoft.Network/networkSecurityGroups/TP2NSG",  
   "location": "westeurope",  
   "name": "TP2NSG",  
   "provisioningState": "Succeeded",  
   "resourceGroup": "TP2",  
   "resourceGuid": "d58932e2-de56-4e70-b41b-b5165f09d503",  
   "securityRules": [],  
   "type": "Microsoft.Network/networkSecurityGroups"  
 }  
}
```

### Network Interfaces creation

```console
┌──(alt㉿Altiera)-[~/Documents/School/Cloud]  
└─$ az network nic create --resource-group TP2 --name TP2NIC1 --vnet-name TP2_VNET --subnet TP2Subnet --network-security-group TP2NSG --public-ip-address TP2_VM_IP1    
  
{  
 "NewNIC": {  
   "auxiliaryMode": "None",  
   "auxiliarySku": "None",  
   "disableTcpStateTracking": false,  
   "dnsSettings": {  
     "appliedDnsServers": [],  
     "dnsServers": [],  
     "internalDomainNameSuffix": "lkokf10whhru3bortpcubbfdbc.ax.internal.cloudapp.net"  
   },  
   "enableAcceleratedNetworking": false,  
   "enableIPForwarding": false,  
   "etag": "W/\"d014d3b8-bac5-4d50-a9d2-8954a734f98c\"",  
   "hostedWorkloads": [],  
   "id": "/subscriptions/eae5cd03-388e-4052-8aa9-90a5958d32b8/resourceGroups/TP2/providers/Microsoft.Network/networkInterfaces/TP2NIC1",  
   "ipConfigurations": [  
     {  
       "etag": "W/\"d014d3b8-bac5-4d50-a9d2-8954a734f98c\"",  
       "id": "/subscriptions/eae5cd03-388e-4052-8aa9-90a5958d32b8/resourceGroups/TP2/providers/Microsoft.Network/networkInterfaces/TP2NIC1/ipConfigurations/ipconfig1",  
       "name": "ipconfig1",  
       "primary": true,  
       "privateIPAddress": "10.0.0.4",  
       "privateIPAddressVersion": "IPv4",  
       "privateIPAllocationMethod": "Dynamic",  
       "provisioningState": "Succeeded",  
       "publicIPAddress": {  
         "id": "/subscriptions/eae5cd03-388e-4052-8aa9-90a5958d32b8/resourceGroups/TP2/providers/Microsoft.Network/publicIPAddresses/TP2_VM_IP1",  
         "resourceGroup": "TP2"  
       },  
       "resourceGroup": "TP2",  
       "subnet": {  
         "id": "/subscriptions/eae5cd03-388e-4052-8aa9-90a5958d32b8/resourceGroups/TP2/providers/Microsoft.Network/virtualNetworks/TP2_VNET/subnets/TP2Subnet",  
         "resourceGroup": "TP2"  
       },  
       "type": "Microsoft.Network/networkInterfaces/ipConfigurations"  
     }  
   ],  
   "location": "westeurope",  
   "name": "TP2NIC1",  
   "networkSecurityGroup": {  
     "id": "/subscriptions/eae5cd03-388e-4052-8aa9-90a5958d32b8/resourceGroups/TP2/providers/Microsoft.Network/networkSecurityGroups/TP2NSG",  
     "resourceGroup": "TP2"  
   },  
   "nicType": "Standard",  
   "provisioningState": "Succeeded",  
   "resourceGroup": "TP2",  
   "resourceGuid": "9d797956-2d3a-4df8-818c-1114271d3edd",  
   "tapConfigurations": [],  
   "type": "Microsoft.Network/networkInterfaces",  
   "vnetEncryptionSupported": false  
 }  
}  
 
{  
 "NewNIC": {  
   "auxiliaryMode": "None",  
   "auxiliarySku": "None",  
   "disableTcpStateTracking": false,  
   "dnsSettings": {  
     "appliedDnsServers": [],  
     "dnsServers": [],  
     "internalDomainNameSuffix": "lkokf10whhru3bortpcubbfdbc.ax.internal.cloudapp.net"  
   },  
   "enableAcceleratedNetworking": false,  
   "enableIPForwarding": false,  
   "etag": "W/\"9671b9c8-03a3-41de-b4f4-4795186e4083\"",  
   "hostedWorkloads": [],  
   "id": "/subscriptions/eae5cd03-388e-4052-8aa9-90a5958d32b8/resourceGroups/TP2/providers/Microsoft.Network/networkInterfaces/TP2NIC2",  
   "ipConfigurations": [  
     {  
       "etag": "W/\"9671b9c8-03a3-41de-b4f4-4795186e4083\"",  
       "id": "/subscriptions/eae5cd03-388e-4052-8aa9-90a5958d32b8/resourceGroups/TP2/providers/Microsoft.Network/networkInterfaces/TP2NIC2/ipConfigurations/ipconfig1",  
       "name": "ipconfig1",  
       "primary": true,  
       "privateIPAddress": "10.0.0.5",  
       "privateIPAddressVersion": "IPv4",  
       "privateIPAllocationMethod": "Dynamic",  
       "provisioningState": "Succeeded",  
       "publicIPAddress": {  
         "id": "/subscriptions/eae5cd03-388e-4052-8aa9-90a5958d32b8/resourceGroups/TP2/providers/Microsoft.Network/publicIPAddresses/TP2_VM_IP2",  
         "resourceGroup": "TP2"  
       },  
       "resourceGroup": "TP2",  
       "subnet": {  
         "id": "/subscriptions/eae5cd03-388e-4052-8aa9-90a5958d32b8/resourceGroups/TP2/providers/Microsoft.Network/virtualNetworks/TP2_VNET/subnets/TP2Subnet",  
         "resourceGroup": "TP2"  
       },  
       "type": "Microsoft.Network/networkInterfaces/ipConfigurations"  
     }  
   ],  
   "location": "westeurope",  
   "name": "TP2NIC2",  
   "networkSecurityGroup": {  
     "id": "/subscriptions/eae5cd03-388e-4052-8aa9-90a5958d32b8/resourceGroups/TP2/providers/Microsoft.Network/networkSecurityGroups/TP2NSG",  
     "resourceGroup": "TP2"  
   },  
   "nicType": "Standard",  
   "provisioningState": "Succeeded",  
   "resourceGroup": "TP2",  
   "resourceGuid": "7eda4c0f-0bb0-4fa2-bcbb-7b095a60d8be",  
   "tapConfigurations": [],  
   "type": "Microsoft.Network/networkInterfaces",  
   "vnetEncryptionSupported": false  
 }  
}
```

### VM Creation
```console
┌──(alt㉿Altiera)-[~/Documents/School/Cloud]  
└─$ az vm create --resource-group TP2 --name TP2_VM1 --image Ubuntu2204 --admin-username alt --nics TP2NIC1 --ssh-key-value /home/alt/.ssh/id_ed25519.pub       
   
{  
 "fqdns": "",  
 "id": "/subscriptions/eae5cd03-388e-4052-8aa9-90a5958d32b8/resourceGroups/TP2/providers/Microsoft.Compute/virtualMachines/TP2_VM1",  
 "location": "westeurope",  
 "macAddress": "00-0D-3A-AF-18-A4",  
 "powerState": "VM running",  
 "privateIpAddress": "10.0.0.4",  
 "publicIpAddress": "20.23.229.191",  
 "resourceGroup": "TP2",  
 "zones": ""  
}

┌──(alt㉿Altiera)-[~/Documents/School/Cloud]  
└─$ az vm create --resource-group TP2 --name TP2_VM2 --image Ubuntu2204 --admin-username alt --nics TP2NIC2 --ssh-key-value /home/alt/.ssh/id_ed25519.pub     
  
{  
 "fqdns": "",  
 "id": "/subscriptions/eae5cd03-388e-4052-8aa9-90a5958d32b8/resourceGroups/TP2/providers/Microsoft.Compute/virtualMachines/TP2_VM2",  
 "location": "westeurope",  
 "macAddress": "00-0D-3A-BD-E6-56",  
 "powerState": "VM running",  
 "privateIpAddress": "10.0.0.5",  
 "publicIpAddress": "20.23.229.227",  
 "resourceGroup": "TP2",  
 "zones": ""  
}
```

### Start VMS

```console
┌──(alt㉿Altiera)-[~/Documents/School/Cloud]  
└─$ az vm start --name TP2_VM2 --resource-group TP2  
                                                                                                                                                                                             
┌──(alt㉿Altiera)-[~/Documents/School/Cloud]  
└─$ az vm start --name TP2_VM1 --resource-group TP2
```

### Allow port 22
```console
┌──(alt㉿Altiera)-[~/Documents/School/Cloud]  
└─$ az network nsg rule create --resource-group TP2 --nsg-name TP2NSG --name SSH_Rule --priority 100 --direction Inbound --access Allow --protocol Tcp --source-address-prefix '*' --destination-port-range 22    
{  
 "access": "Allow",  
 "destinationAddressPrefix": "*",  
 "destinationAddressPrefixes": [],  
 "destinationPortRange": "22",  
 "destinationPortRanges": [],  
 "direction": "Inbound",  
 "etag": "W/\"fb2db950-7480-409f-899b-07ebea761421\"",  
 "id": "/subscriptions/eae5cd03-388e-4052-8aa9-90a5958d32b8/resourceGroups/TP2/providers/Microsoft.Network/networkSecurityGroups/TP2NSG/securityRules/SSH_Rule",  
 "name": "SSH_Rule",  
 "priority": 100,  
 "protocol": "Tcp",  
 "provisioningState": "Succeeded",  
 "resourceGroup": "TP2",  
 "sourceAddressPrefix": "*",  
 "sourceAddressPrefixes": [],  
 "sourcePortRange": "*",  
 "sourcePortRanges": [],  
 "type": "Microsoft.Network/networkSecurityGroups/securityRules"  
}
```

### Connection Test
```console 
alt@TP2VM2:~$ ip a  
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000  
   link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00  
   inet 127.0.0.1/8 scope host lo  
      valid_lft forever preferred_lft forever  
   inet6 ::1/128 scope host    
      valid_lft forever preferred_lft forever  
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000  
   link/ether 00:0d:3a:bd:e6:56 brd ff:ff:ff:ff:ff:ff  
   inet 10.0.0.5/24 metric 100 brd 10.0.0.255 scope global eth0  
      valid_lft forever preferred_lft forever  
   inet6 fe80::20d:3aff:febd:e656/64 scope link    
      valid_lft forever preferred_lft forever  
alt@TP2VM2:~$ ping 10.0.0.4  
PING 10.0.0.4 (10.0.0.4) 56(84) bytes of data.  
64 bytes from 10.0.0.4: icmp_seq=1 ttl=64 time=0.786 ms  
64 bytes from 10.0.0.4: icmp_seq=2 ttl=64 time=0.939 ms  
64 bytes from 10.0.0.4: icmp_seq=3 ttl=64 time=0.828 ms  
64 bytes from 10.0.0.4: icmp_seq=4 ttl=64 time=4.50 ms  
^C  
--- 10.0.0.4 ping statistics ---  
4 packets transmitted, 4 received, 0% packet loss, time 3081ms  
rtt min/avg/max/mdev = 0.786/1.763/4.500/1.581 ms
```

# Part II : cloud-init
## 2. Gooooo
### VM Creation
```console 
┌──(alt㉿Altiera)-[~/Documents/School/Cloud]  
└─$ az vm create --resource-group TP2 --name TP2_VM3 --image Ubuntu2204 --admin-username alt --nics TP2NIC2 --ssh-key-value /home/alt/.ssh/id_ed25519.pub --custom-data ./cloud-init.txt  
  
{  
 "fqdns": "",  
 "id": "/subscriptions/eae5cd03-388e-4052-8aa9-90a5958d32b8/resourceGroups/TP2/providers/Microsoft.Compute/virtualMachines/TP2_VM3",  
 "location": "westeurope",  
 "macAddress": "00-0D-3A-BD-E6-56",  
 "powerState": "VM running",  
 "privateIpAddress": "10.0.0.5",  
 "publicIpAddress": "20.23.229.227",  
 "resourceGroup": "TP2",  
 "zones": ""  
}
```

### Connection Test
```console
┌──(alt㉿Altiera)-[~/Documents/School/Cloud]  
└─$ ssh altiera@20.23.229.227 -i altiera    
...
altiera@TP2VM3:~$ id  
uid=1000(altiera) gid=1000(altiera) groups=1000(altiera)
```

## 3. Write your own

### Cloud init
```cloud-init
#cloud-config  
users:  
 - default  
 - name: altiera  
   sudo: ALL=(ALL) NOPASSWD:ALL  
   shell: /bin/bash  
   groups:  
     - docker  
   password: "$6$r8idKZCZVIy7uMrM$HtRueXAL0in0exYZM4IpEIGorfyl7FOrDijXo9H5vbWUeEHYF.eujX.48l1t2D2XiKV5gGXy2zqqiyENuSD9./"  
   ssh_authorized_keys:  
     - ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIO5czm59jQujy/+xwBwnS3FrlptjbPHWbZ5oz+cuk1jo alt@Altiera  
  
chpasswd:  
 expire: false  
  
runcmd:  
 - curl -fsSL https://get.docker.com | sh  
 - docker pull alpine:latest
```

### Checks
```console
┌──(alt㉿Altiera)-[~/Documents/School/Cloud]  
└─$ ssh altiera@20.23.229.227 -i altiera 
  
altiera@TP2VM3:~$ sudo -l  
Matching Defaults entries for altiera on TP2VM3:  
   env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty  
  
User altiera may run the following commands on TP2VM3:  
   (ALL) NOPASSWD: ALL  
altiera@TP2VM3:~$ docker images  
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE  
alpine       latest    aded1e1a5b37   6 weeks ago   7.83MB  
altiera@TP2VM3:~$ id  
uid=1000(altiera) gid=1001(altiera) groups=1001(altiera),1000(docker)
```

# Part III : Terraform

## 2. Copy paste

### Checks
```console
┌──(alt㉿Altiera)-[~/Documents/School/Cloud/Terraform]  
└─$ az vm list -o table                                                       
  
Name            ResourceGroup          Location    Zones  
--------------  ---------------------  ----------  -------  
tp2magueule-vm  TP2MAGUEULE-RESOURCES  westeurope  
                                                                                                                                                                                             
┌──(alt㉿Altiera)-[~/Documents/School/Cloud/Terraform]  
└─$ az vm show --name tp2magueule-vm --resource-group tp2magueule-resources -o table  
Name            ResourceGroup          Location    Zones  
--------------  ---------------------  ----------  -------  
tp2magueule-vm  tp2magueule-resources  westeurope  
                                                                                                                                                                                             
┌──(alt㉿Altiera)-[~/Documents/School/Cloud/Terraform]  
└─$ az group list -o table                                                             
Name                   Location    Status  
---------------------  ----------  ---------  
NetworkWatcherRG       westeurope  Succeeded  
tp2                    westeurope  Succeeded  
tp2magueule-resources  westeurope  Succeeded

```

### 3. Do it yourself

```console
alt@node1:~$ ping 10.0.2.5  
PING 10.0.2.5 (10.0.2.5) 56(84) bytes of data.  
64 bytes from 10.0.2.5: icmp_seq=1 ttl=64 time=0.030 ms  
64 bytes from 10.0.2.5: icmp_seq=2 ttl=64 time=0.056 ms  
64 bytes from 10.0.2.5: icmp_seq=3 ttl=64 time=0.048 ms  
^C  
--- 10.0.2.5 ping statistics ---  
3 packets transmitted, 3 received, 0% packet loss, time 2031ms  
rtt min/avg/max/mdev = 0.030/0.044/0.056/0.010 ms  

alt@node1:~$ ssh alt@node2 -i .ssh/id_ed    
...
alt@node2:~$ ip a  
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000  
   link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00  
   inet 127.0.0.1/8 scope host lo  
      valid_lft forever preferred_lft forever  
   inet6 ::1/128 scope host    
      valid_lft forever preferred_lft forever  
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000  
   link/ether 60:45:bd:f4:19:ab brd ff:ff:ff:ff:ff:ff  
   inet 10.0.2.4/24 metric 100 brd 10.0.2.255 scope global eth0  
      valid_lft forever preferred_lft forever  
   inet6 fe80::6245:bdff:fef4:19ab/64 scope link    
      valid_lft forever preferred_lft forever
```

```yml
provider "azurerm" {
  features {}
  subscription_id = "eae5cd03-388e-4052-8aa9-90a5958d32b8"
}

resource "azurerm_resource_group" "main" {
  name     = "${var.prefix}-resources"
  location = var.location
}

resource "azurerm_virtual_network" "main" {
  name                = "${var.prefix}-network1"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
}

resource "azurerm_subnet" "internal" {
  name                 = "internal"
  resource_group_name  = azurerm_resource_group.main.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = ["10.0.2.0/24"]
}

resource "azurerm_public_ip" "pip" {
  name                = "${var.prefix}-pip"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  allocation_method   = "Static"
}

resource "azurerm_network_interface" "main" {
  name                = "${var.prefix}-nic1"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location

  ip_configuration {
    name                          = "primary"
    subnet_id                     = azurerm_subnet.internal.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.pip.id
  }
}

resource "azurerm_network_interface" "nic2" {
  name                = "${var.prefix}-nic2"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location

  ip_configuration {
    name                          = "primary"
    subnet_id                     = azurerm_subnet.internal.id
    private_ip_address_allocation = "Dynamic"
  }
}

resource "azurerm_network_interface" "nic-internal" {
  name                = "${var.prefix}-nic-internal"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location

  ip_configuration {
    name                          = "internal"
    subnet_id                     = azurerm_subnet.internal.id
    private_ip_address_allocation = "Dynamic"
  }
}

resource "azurerm_network_security_group" "ssh" {
  name                = "ssh"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name

  security_rule {
    name                       = "ssh"
    priority                   = 100
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    source_address_prefix      = "*"
    destination_port_range     = "22"
    destination_address_prefix = "*"  # Changed to allow SSH access
  }
}

resource "azurerm_network_interface_security_group_association" "main" {
  network_interface_id      = azurerm_network_interface.main.id
  network_security_group_id = azurerm_network_security_group.ssh.id
}

resource "azurerm_linux_virtual_machine" "main" {
  name                = "node1"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  size                = "Standard_F2"
  admin_username      = "alt"

  network_interface_ids = [
    azurerm_network_interface.main.id,
    azurerm_network_interface.nic-internal.id,
  ]

  admin_ssh_key {
    username   = "alt"
    public_key = file("/home/alt/.ssh/id_ed25519.pub")
  }

  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-jammy"
    sku       = "22_04-lts"
    version   = "latest"
  }

  os_disk {
    storage_account_type = "Standard_LRS"
    caching              = "ReadWrite"
  }
}

resource "azurerm_linux_virtual_machine" "node2" {
  name                = "node2"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  size                = "Standard_F2"
  admin_username      = "alt"

  network_interface_ids = [
    azurerm_network_interface.nic2.id,
  ]

  admin_ssh_key {
    username   = "alt"
    public_key = file("/home/alt/.ssh/id_ed25519.pub")
  }

  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-jammy"
    sku       = "22_04-lts"
    version   = "latest"
  }

  os_disk {
    storage_account_type = "Standard_LRS"
    caching              = "ReadWrite"
  }
}

```

### Checks 
```console
┌──(alt㉿Altiera)-[~/Documents/School/Cloud/TerraformV3]  
└─$ ssh altiera@20.224.74.119 -i ../altiera    
altiera@solong-vm:~$ id  
uid=1000(altiera) gid=1001(altiera) groups=1001(altiera),1000(docker)  
altiera@solong-vm:~$ docker images  
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE  
alpine       latest    aded1e1a5b37   6 weeks ago   7.83MB
```

Scripts: 
### Terraform
```yml
provider "azurerm" {  
 features {}  
 subscription_id = "eae5cd03-388e-4052-8aa9-90a5958d32b8"  
}  
resource "azurerm_resource_group" "main" {  
 name     = "${var.prefix}-resources"  
 location = var.location  
}  
resource "azurerm_virtual_network" "main" {  
 name                = "${var.prefix}-network"  
 address_space       = ["10.0.0.0/16"]  
 location            = azurerm_resource_group.main.location  
 resource_group_name = azurerm_resource_group.main.name  
}  
resource "azurerm_subnet" "internal" {  
 name                 = "internal"  
 resource_group_name  = azurerm_resource_group.main.name  
 virtual_network_name = azurerm_virtual_network.main.name  
 address_prefixes     = ["10.0.2.0/24"]  
}  
resource "azurerm_public_ip" "pip" {  
 name                = "${var.prefix}-pip"  
 resource_group_name = azurerm_resource_group.main.name  
 location            = azurerm_resource_group.main.location  
 allocation_method   = "Static"  
}  
resource "azurerm_network_interface" "main" {  
 name                = "${var.prefix}-nic1"  
 resource_group_name = azurerm_resource_group.main.name  
 location            = azurerm_resource_group.main.location  
 ip_configuration {  
   name                          = "primary"  
   subnet_id                     = azurerm_subnet.internal.id  
   private_ip_address_allocation = "Dynamic"  
   public_ip_address_id          = azurerm_public_ip.pip.id  
 }  
}  
resource "azurerm_network_interface" "internal" {  
 name                = "${var.prefix}-nic2"  
 resource_group_name = azurerm_resource_group.main.name  
 location            = azurerm_resource_group.main.location  
 ip_configuration {  
   name                          = "internal"  
   subnet_id                     = azurerm_subnet.internal.id  
   private_ip_address_allocation = "Dynamic"  
 }  
}  
resource "azurerm_network_security_group" "ssh" {  
 name                = "ssh"  
 location            = azurerm_resource_group.main.location  
 resource_group_name = azurerm_resource_group.main.name  
 security_rule {  
   access                     = "Allow"  
   direction                  = "Inbound"  
   name                       = "ssh"  
   priority                   = 100  
   protocol                   = "Tcp"  
   source_port_range          = "*"  
   source_address_prefix      = "*"  
   destination_port_range     = "22"  
   destination_address_prefix = azurerm_network_interface.main.private_ip_address  
 }  
}  
resource "azurerm_network_interface_security_group_association" "main" {  
 network_interface_id      = azurerm_network_interface.main.id  
 network_security_group_id = azurerm_network_security_group.ssh.id  
}  
resource "azurerm_linux_virtual_machine" "main" {  
 name                            = "${var.prefix}-vm"  
 resource_group_name             = azurerm_resource_group.main.name  
 location                        = azurerm_resource_group.main.location  
 size                            = "Standard_F2"  
 admin_username                  = "alt"  
 network_interface_ids = [  
   azurerm_network_interface.main.id,  
   azurerm_network_interface.internal.id,  
 ]  
 admin_ssh_key {  
   username   = "alt"  
   public_key = file("/home/alt/.ssh/id_ed25519.pub")  
 }  
 source_image_reference {  
   publisher = "Canonical"  
   offer     = "0001-com-ubuntu-server-jammy"  
   sku       = "22_04-lts"  
   version   = "latest"  
 }  
 os_disk {  
   storage_account_type = "Standard_LRS"  
   caching              = "ReadWrite"  
 }  
 custom_data = filebase64("${path.module}/cloud-init.txt")  
}
```
### Cloud init
```yml
#cloud-config  
users:  
 - default  
 - name: altiera  
   sudo: ALL=(ALL) NOPASSWD:ALL  
   shell: /bin/bash  
   groups:  
     - docker  
   password: "$6$r8idKZCZVIy7uMrM$HtRueXAL0in0exYZM4IpEIGorfyl7FOrDijXo9H5vbWUeEHYF.eujX.48l1t2D2XiKV5gGXy2zqqiyENuSD9./"  
   ssh_authorized_keys:  
     - ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIO5czm59jQujy/+xwBwnS3FrlptjbPHWbZ5oz+cuk1jo alt@Altiera  
  
chpasswd:  
 expire: false  
  
runcmd:  
 - curl -fsSL https://get.docker.com | sh  
 - docker pull alpine:latest
```


### B. Go further

#### cloud-init.txt
```yml
#cloud-config  
users:  
 - default  
 - name: altiera  
   sudo: ALL=(ALL) NOPASSWD:ALL  
   shell: /bin/bash  
   groups:  
     - docker  
   password: "$6$r8idKZCZVIy7uMrM$HtRueXAL0in0exYZM4IpEIGorfyl7FOrDijXo9H5vbWUeEHYF.eujX.48l1t2D2XiKV5gGXy2zqqiyENuSD9./"  
   ssh_authorized_keys:  
     - ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIO5czm59jQujy/+xwBwnS3FrlptjbPHWbZ5oz+cuk1jo alt@Altiera 
  
chpasswd:  
 expire: false  
  
write_files:  
 - path: /opt/wikijs/docker-compose.yml  
   content: |  
     version: '3'  
     services:  
       db:  
         image: postgres:15-alpine  
         environment:  
           POSTGRES_DB: wiki  
           POSTGRES_USER: wikijs  
           POSTGRES_PASSWORD: wikijsrocks  
         logging:  
           driver: none  
         restart: unless-stopped  
         volumes:  
           - db-data:/var/lib/postgresql/data  
  
       wiki:  
         image: ghcr.io/requarks/wiki:2  
         depends_on:  
           - db  
         environment:  
           DB_TYPE: postgres  
           DB_HOST: db  
           DB_PORT: 5432  
           DB_USER: wikijs  
           DB_PASS: wikijsrocks  
           DB_NAME: wiki  
         restart: unless-stopped  
         ports:  
           - "10101:3000"  
  
     volumes:  
       db-data:  
  
runcmd:  
 - curl -fsSL https://get.docker.com | sh  
 - sudo docker compose -f /opt/wikijs/docker-compose.yml up -d
```

#### main.tf
```yml
provider "azurerm" {  
 features {}  
 subscription_id = "eae5cd03-388e-4052-8aa9-90a5958d32b8"  
}  
resource "azurerm_resource_group" "main" {  
 name     = "${var.prefix}-resources"  
 location = var.location  
}  
resource "azurerm_virtual_network" "main" {  
 name                = "${var.prefix}-network"  
 address_space       = ["10.0.0.0/16"]  
 location            = azurerm_resource_group.main.location  
 resource_group_name = azurerm_resource_group.main.name  
}  
resource "azurerm_subnet" "internal" {  
 name                 = "internal"  
 resource_group_name  = azurerm_resource_group.main.name  
 virtual_network_name = azurerm_virtual_network.main.name  
 address_prefixes     = ["10.0.2.0/24"]  
}  
resource "azurerm_public_ip" "pip" {  
 name                = "${var.prefix}-pip"  
 resource_group_name = azurerm_resource_group.main.name  
 location            = azurerm_resource_group.main.location  
 allocation_method   = "Static"  
}  
resource "azurerm_network_interface" "main" {  
 name                = "${var.prefix}-nic1"  
 resource_group_name = azurerm_resource_group.main.name  
 location            = azurerm_resource_group.main.location  
 ip_configuration {  
   name                          = "primary"  
   subnet_id                     = azurerm_subnet.internal.id  
   private_ip_address_allocation = "Dynamic"  
   public_ip_address_id          = azurerm_public_ip.pip.id  
 }  
}  
resource "azurerm_network_interface" "internal" {  
 name                = "${var.prefix}-nic2"  
 resource_group_name = azurerm_resource_group.main.name  
 location            = azurerm_resource_group.main.location  
 ip_configuration {  
   name                          = "internal"  
   subnet_id                     = azurerm_subnet.internal.id  
   private_ip_address_allocation = "Dynamic"  
 }  
}  
resource "azurerm_network_security_group" "ssh" {  
 name                = "ssh"  
 location            = azurerm_resource_group.main.location  
 resource_group_name = azurerm_resource_group.main.name  
  
 security_rule {  
   access                     = "Allow"  
   direction                  = "Inbound"  
   name                       = "ssh"  
   priority                   = 100  
   protocol                   = "Tcp"  
   source_port_range          = "*"  
   source_address_prefix      = "*"  
   destination_port_range     = "22"  
   destination_address_prefix = "*"  
 }  
 security_rule {  
   access                     = "Allow"  
   direction                  = "Inbound"  
   name                       = "custom-port"  
   priority                   = 110  
   protocol                   = "Tcp"  
   source_port_range          = "*"  
   source_address_prefix      = "*"  
   destination_port_range     = "10101"  
   destination_address_prefix = "*"  
 }  
}  
  
  
resource "azurerm_network_interface_security_group_association" "main" {  
 network_interface_id      = azurerm_network_interface.main.id  
 network_security_group_id = azurerm_network_security_group.ssh.id  
}  
resource "azurerm_linux_virtual_machine" "main" {  
 name                            = "${var.prefix}-vm"  
 resource_group_name             = azurerm_resource_group.main.name  
 location                        = azurerm_resource_group.main.location  
 size                            = "Standard_F2"  
 admin_username                  = "alt"  
 network_interface_ids = [  
   azurerm_network_interface.main.id,  
   azurerm_network_interface.internal.id,  
 ]  
 admin_ssh_key {  
   username   = "alt"  
   public_key = file("/home/alt/.ssh/id_ed25519.pub")  
 }  
 source_image_reference {  
   publisher = "Canonical"  
   offer     = "0001-com-ubuntu-server-jammy"  
   sku       = "22_04-lts"  
   version   = "latest"  
 }  
 os_disk {  
   storage_account_type = "Standard_LRS"  
   caching              = "ReadWrite"  
 }  
 custom_data = filebase64("${path.module}/cloud-init.txt")  
}
```

```console
┌──(alt㉿Altiera)-[~/Documents/School/Cloud/TerraformV4]  
└─$ curl http://20.224.74.119:10101  
<!DOCTYPE html><html><head><meta http-equiv="X-UA-Compatible" content="IE=edge"><meta charset="UTF-8"><meta name="viewport" content="user-scalable=yes, width=device-width, initial-scale=1  
, maximum-scale=5"><meta name="theme-color" content="#1976d2"><meta name="msapplication-TileColor" content="#1976d2"><meta name="msapplication-TileImage" content="/_assets/favicons/mstile  
-150x150.png"><title>Wiki.js Setup</title><link rel="apple-touch-icon" sizes="180x180" href="/_assets/favicons/apple-touch-icon.png"><link rel="icon" type="image/png" sizes="192x192" href  
="/_assets/favicons/android-chrome-192x192.png"><link rel="icon" type="image/png" sizes="32x32" href="/_assets/favicons/favicon-32x32.png"><link rel="icon" type="image/png" sizes="16x16"  
href="/_assets/favicons/favicon-16x16.png"><link rel="mask-icon" href="/_assets/favicons/safari-pinned-tab.svg" color="#1976d2"><link rel="manifest" href="/_assets/manifest.json"><script>  
var siteConfig = {"title":"Wiki.js"}  
</script><link type="text/css" rel="stylesheet" href="/_assets/css/setup.22871ffac1b643eed4d9.css"><script type="text/javascript" src="/_assets/js/runtime.js?1742780132"></script><script  
type="text/javascript" src="/_assets/js/setup.js?1742780132"></script></head><body><div id="root"><setup wiki-version="2.5.307"></setup></div></body></html>
```