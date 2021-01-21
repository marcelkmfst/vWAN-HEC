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
While it is possible to treat the SAP HEC environment as branches and connect them by setting up Azure VPN Gateways and establish IPSec Tunnels, the recommended method is to leverage an Virtual WAN connection. An Virtual WAN VNet connection establishes a peering between an VNet and an Virtual WAN Hub Network. The Virtual WAN HUB Router instances route traffic to and from the connected VNet to all other VNets and branches. There is no requirement to provision Virtual Network Gateways.
An Virtual WAN VNet Connection can be established even if the VNet is not in the same Azure Active Directory (AAD) Tenant hence it can be used to interconnect a SAP HEC environment.
For more information on how to establish a Virtual WAN VNet connection across tenants see [Connect cross-tenant VNets to a Virtual WAN Hub](https://docs.microsoft.com/en-us/azure/virtual-wan/cross-tenant-vnet).

> [!WARNING]
> Advertising a default route to the Virtual WAN Connection of the SAP HEC Virtual Networks is not supported and can result in SAP HEC service unavailability.

### Interconnect SAP HEC environment with Single Virtual WAN Hub
The below diagram shows a scenario where a DR enabled SAP HEC environment is connected to a single Virtual WAN Hub via Virtual WAN VNet connections. 

![SAPVWNASINGLE](/SAPVWANSINGLEHUB.jpg)

While still being owned & managed by SAP, the provided services are available within the customer environment like customer owned ones. Depending on the customer requirements and the routing configuration, the SAP HEC services can made reachable from  all connected branches, mobile users and customer VNets or only from specific ones.  For more  information about Virtual WAN Routing see [About virtual hub routing](https://docs.microsoft.com/en-us/azure/virtual-wan/about-virtual-hub-routing#:~:text=%20Please%20consider%20the%20following%20when%20configuring%20Virtual,via%20Azure%20Firewall%20is%20currently%20not...%20More).

### Interconnect SAP HEC environment with multiple Virtual WAN Hubs
In order to protect against rare the scenario where the primary SAP HEC VNet and the Virtual WAN Hub to which it is connected becomes unavailable, multiple Virtual WAN Hubs and connections are required. 

![SAPVWNANMULTI](/SAPVWANMULTIHUB.jpg)

This architecture follows the recommended approach where a combination of multiple Hubs and connections are used. In this example, the SAP HEC primary VNet is connected to a Virtual WAN Hub in West Europe. The secondary VNet is provisioned in the paired Azure Region North Europe and is connected to a Virtual WAN Hub in North Europe. With this configuration, connected branches and VNets would be able to reach the SAP provided Services in both regions. If SAP HEC operations initiate a failover, all connected branches, mobile users and VNets can connect to the SAP HEC Services at the DR Site.
However, if the Virtual WAN Hub in West Europe would also become unavailable, all branches connected would not be able to reach the SAP HEC services in North Europe. 
In order to protect against this scenario, branches should always be connected to both Hubs. Redundant connections ensure that connectivity to the SAP HEC services (and other virtual WAN connections like Branch to Branch) succeed even if a virtual WAN Hub is unavailable. 

> [!IMPORTANT]
> You cannot connect a VNet to two Virtual WAN Hubs.

C
Missing
Firewall
Pricing




