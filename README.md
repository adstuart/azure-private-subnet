# azure-private-subnet
Terse logs from testing Private Subnet feature (preview)

# Context

https://azure.microsoft.com/en-us/updates/default-outbound-access-for-vms-in-azure-will-be-retired-transition-to-a-new-method-of-internet-access/

# [1] Baseline (NAT-GW for outbound)

- deploy new VNet
- deploy new subnet with [Private Subnet](https://learn.microsoft.com/en-us/azure/virtual-network/ip-services/default-outbound-access#utilize-the-private-subnet-parameter) preview feature flag = enabled
- deploy VM (with no PIP) in to Subnet
- deploy NAT-GW in to Subnet
- access internet site = yes, src.ip = nat-gw = OK
- access storage account blob = OK
- enable service endpoints to _Microsoft.Storage.Global_ on VM subnet, access still = OK
- restrict storage account FW inbound to only specific VNet/subnet, access still = OK
- remove Service endpoints
- configure UDR, dest = service tag _"Storage"_, Nexthop = Internet. Access still = OK

Baseline verification tests = all OK, all outbound scenarios enabled via NAT GW as expected.

# [2] Private Subnet feature test scenarios (No explicit outbound configured)

- reset lab, remove NAT-GW, delete service endpoints, delete UDR
- access internet site = **broken**
- access storage account blob = **broken**
- enable service endpoints to _Microsoft.Storage.Global_ on VM subnet, access = **OK**
- restrict storage account FW inbound to only specific VNet/subnet, access still = **OK**
- remove Service endpoints
- access storage account blob = **broken again**
- configure UDR, dest = service tag _"Storage"_, Nexthop = Internet. Access = **broken**

Conclusion - Outbound access disabled as per Private Subnet feature purpose. Access to Storage still possible via Service Endpoint.

# [3] Repeat Service Endpoint test with different PaaS services

- leave lab as per [2], private subnet = enabled, no NAT-GW
  
- deploy Web App, access = **broken**
- enable service endpoint for Microsoft.Web, access = **OK**

- deploy SQL database, access = **broken**
- enable service endpoint for Microsoft.SQL, access = **OK**

- deploy Eventhub, try access to Service Bus endpoint = **broken**
- enable service endpoint for Microsoft.EventHub, access = **OK**

Conclusion - Outbound access to Service Endpoints is unaffected by Private Subnet feature. In other words, access to PaaS services over Service Endpoints will continue to function, even after deprecation of default Internet Outbound access.


