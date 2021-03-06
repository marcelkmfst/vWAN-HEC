---
# This basic template provides core metadata fields for Markdown articles on docs.microsoft.com.

# Mandatory fields.
title: Intent and product brand in a unique string of 43-59 chars including spaces. Do not include site identifier (it is auto-generated).
description: 115-145 characters including spaces. This abstract displays in the search result.
author: NO VALUE SET
ms.author: NO VALUE SET # Microsoft employees only
ms.date: 1/21/2021
ms.topic: article-type-from-white-list
# Use ms.service for services or ms.prod for on-prem products. Remove the # before the relevant field.
# ms.service: service-name-from-white-list
# ms.prod: product-name-from-white-list

# Optional fields. Don't forget to remove # if you need a field.
# ms.custom: can-be-multiple-comma-separated
# ms.reviewer: MSFT-alias-of-reviewer
# manager: MSFT-alias-of-manager-or-PM-counterpart
---
# Interconnect SAP HANA Enterprise Cloud environment running on Azure with Virtual WAN

> [!IMPORTANT]
> All options discussed in this article must be validated with customers SAP representative as SAP validated interconnect options may change over time. 

This article describes the high-level architecture to interconnect a SAP HANA Enterprise Cloud (HEC) environment running on Azure to an Azure virtual WAN. 
The customer systems of an Azure based SAP HEC environments are provisioned in dedicated Azure Virtual Networks (VNets) operated by SAP. Depending on the chosen Service level, usually a single VNet is used for non Disaster Recovery (DR) enabled deployments and two VNets are used for DR enabled deployments where the second VNet is provisioned in the designated DR Region. SAP establishes a global VNet peering between those networks to facilitate replication traffic.

The below diagram depicts a standard Azure based SAP HEC customer environment with DR enabled.

![SAPHECDR](/SAPHECDR.jpg)

The primary VNet is provisioned within the Azure Region West Europe. Another VNet is provisioned within the North Europe Azure Region. A global VNet peering is established between those two networks in order to allow replication traffic to flow between the two regions. 

## Interconnect SAP HEC environment with Azure Virtual WAN
While it is possible to treat the SAP HEC environment as branches and connect them by setting up Azure VPN Gateways and establish IPSec Tunnels, the recommended method is to leverage an Virtual WAN connection. An Virtual WAN VNet connection establishes a peering between an VNet and an Virtual WAN Hub Network. The Virtual WAN HUB Router instances route traffic to and from the connected VNet to all other VNets. Traffic coming from on-premises/branches will terminate in the respective VPN/ExpressRoute gateways and only go through a router if accessing a remote hub’s VNet.
An Virtual WAN VNet Connection can be established even if the VNet is not in the same Azure Active Directory (AAD) Tenant hence it can be used to interconnect a SAP HEC environment.
For more information on how to establish a Virtual WAN VNet connection across tenants see [Connect cross-tenant VNets to a Virtual WAN Hub](https://docs.microsoft.com/en-us/azure/virtual-wan/cross-tenant-vnet).

> [!WARNING]
> When using secured Virtual Hubs to filter Internet traffic of VNet connections, a default route of 0.0.0.0/0 with a next hop of the Azure Firewall instance in the virtual Hub is advertised to the VNet route table. Advertising such a default route to the Virtual WAN Connection of the SAP HEC Virtual Networks is not supported and can result in SAP HEC service unavailability.
You can disable the propagation of a default route advertised by a secured virtual Hub for single cross-tenant VNet connections via the vWAN Powershell cmdlets and the parameter "enableinternetsecurityflag". Fore more information see [vWAN Powershell cmdlets - New-AzVirtualHubVnetConnection](https://docs.microsoft.com/en-us/powershell/module/az.network/new-azvirtualhubvnetconnection?view=azps-5.4.0) and to change existing cross-Tenant peerings see [vWAN Powershell cmdlets - Update-AzVirtualHubVnetConnection](https://docs.microsoft.com/en-us/powershell/module/az.network/update-azvirtualhubvnetconnection?view=azps-5.4.0)

### Interconnect SAP HEC environment with Single Virtual WAN Hub
The below diagram shows a scenario where a DR enabled SAP HEC environment is connected to a single Virtual WAN Hub via Virtual WAN VNet connections. 

![SAPVWNASINGLE](/SAPVWANSINGLEHUB.jpg)

While still being owned & managed by SAP, the provided services are available within the customer environment like customer owned ones. Depending on the customer requirements and the routing configuration, the SAP HEC services can made reachable from  all connected branches, mobile users and customer VNets or only from specific ones.  For more  information about Virtual WAN Routing see [About virtual hub routing](https://docs.microsoft.com/en-us/azure/virtual-wan/about-virtual-hub-routing#:~:text=%20Please%20consider%20the%20following%20when%20configuring%20Virtual,via%20Azure%20Firewall%20is%20currently%20not...%20More).

### Interconnect SAP HEC environment with multiple Virtual WAN Hubs
In Theory, a single Virtual WAN Hub can become a single point of failure. In the rare event that an entire Virtual WAN Hub becomes unavailable, connections to the connected VNEts are also no longer possible.
In order to protect against this scenario redundant multiple Virtual WAN Hubs and connections are required. 

![SAPVWNANMULTI](/SAPVWANMULTIHUB.jpg)

This architecture follows the recommended approach where a combination of multiple Hubs and connections are used. In this example, the SAP HEC primary VNet is connected to a Virtual WAN Hub in West Europe. The secondary VNet is provisioned in the paired Azure Region North Europe and is connected to a Virtual WAN Hub in North Europe. With this configuration, connected branches and VNets would be able to reach the SAP provided Services in both regions. If SAP HEC operations initiate a failover, all connected branches, mobile users and VNets can connect to the SAP HEC Services at the DR Site.
However, if the Virtual WAN Hub in West Europe would also become unavailable, all branches connected would not be able to reach the SAP HEC services in North Europe. 
In order to protect against this scenario, branches should always be connected to both Hubs. Redundant connections ensure that connectivity to the SAP HEC services (and other virtual WAN connections like Branch to Branch) succeed even if a virtual WAN Hub is unavailable. 

> [!IMPORTANT]
> You cannot connect a VNet to two Virtual WAN Hubs.

### Interconnect SAP HEC environment with Single secured Virtual WAN Hub

In order to filter traffic to and from the SAP HEC VNets, virtual WAN Routing can be configured so that the traffic is routed over Azure Firewall instances deployed in the virtual WAN Hub. For more information about secured virtual WAN Hubs see [What is a secured virtual hub?](https://docs.microsoft.com/en-us/azure/firewall-manager/secured-virtual-hub)

> [!NOTE]
> Traffic from the customers environment to and from the HANA Enterprise Cloud environment is not restricted by SAP. The customer is responsible to filter this traffic. 

> [!NOTE]
> Azure is working with select networking partners to enable customers to deploy a third-party Network Virtual Appliance (NVA) directly into the virtual hub. For more information see [Network Virtual Appliance in an Azure Virtual WAN hub](https://docs.microsoft.com/en-us/azure/virtual-wan/about-nva-hub)

> [!NOTE]
> Customers using a Network Virtual Appliance which is not yet supported in an Virtual WAN hub and don´t want to use Azure Firewall, can also deploy a Network Virtual Appliance within a virtual WAN Spoke VNet. For more information see [Route traffic through an NVA](https://docs.microsoft.com/en-us/azure/virtual-wan/scenario-route-through-nva)

![SAPVWNASINGLEHUBFW](/SAPVWANSINGLEHUBFW.jpg)


In the example above, traffic to and from the SAP HEC VNets is forwarded over Azure Firewall instances deployed in the virtual WAN Hub. This can be achieved by using virtual WAN Custom Route Tables. 

> [!NOTE]
> Custom Route Tables can also be used to control the advertisement of the Routes pointing to the SAP HEC VNets. For example, it can be configured that the routes to the SAP HEC VNets are only visible from certain connected VNets. 

The configuration of traffic rules is done within an Azure Firewall Policy and the Firewall logs all traffic. 


> [!NOTE]
> Internet Traffic within the SAP HEC environment is controlled by SAP and can´t be forwarded over a secured virtual Hub. 

### Interconnect SAP HEC environment with multiple secured Virtual WAN Hub

Customers which leverage multiple global distributed secured virtual WAN Hubs can still filter traffic to and from the SAP HEC environment. However, currently only traffic from branches, Mobile Users and VNets connected to the virtual WAN Hub where the SAP HEC VNets are connected can be inspected. Branches, VNets and Mobile users connected to a remote Hub must bypass the Azure Firewall instance in the secured Virtual Hub in order to communicate with the SAP HEC systems.

