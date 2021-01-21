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
# Interconnect SAP HANA Enterprise Cloud environment Running on Azure with Virtual WAN

> [!IMPORTANT]
> All options discussed in this article must be validated with customers SAP representative as SAP validated interconnect options may change over time. 

This article describes the high-level architecture to interconnect a SAP HANA Enterprise Cloud (HEC) environment running on Azure to an Azure virtual WAN. 
The customer systems of an Azure based SAP HEC environments are provisioned in dedicated Azure Virtual Networks (VNets) operated by SAP. Depending on the chosen Service level, usually a single VNet is used for non Disaster Recovery (DR) enabled deployments and two VNets are used for DR enabled deployments where the second VNet is provisioned in the designated DR Region. SAP establishes a global VNet peering between those networks to facilitate replication traffic.

The below diagram depicts a standard Azure based SAP HEC customer environment with DR enabled.

:::image type="content" source="SAPDR.jpg" alt-text="SAPDR":::

The primary VNet is provisioned within the Azure Region West Europe. Another VNet is provisioned within the North Europe Azure Region. A global VNet peering is established between those two networks in order to allow replication traffic to flow between the two regions. 


Interconnect SAP HEC environment with Azure Virtual WAN 
While it is possible to treat the SAP HEC environment as branches and connect them by setting up Azure VPN Gateways and establish IPSEC Tunnels, the recommended method is to leverage an Azure vWAN VNet connection. An Azure vWAN VNet connection establishes a peering between an VNet and an virtual WAN Hub Network. The virtual WAN HUB Router instances route traffic to and from the connected VNet to all other VNets and branches. There is no requirement to provision Virtual Network Gateways.
An Virtual WAN VNet Connection can be established even if the VNet is not in the same Azure Active Directory (AAD) Tenant hence it can be used to interconnect a SAP HEC environment. 
Fore more information on how to establish a Virtual WAN VNet connection across tenants see  

> [!IMPORTANT]
> You cannot connect an Azure Virtual Network to two Azure vWAN Hubs.

Site-2-Site VPN
This option interconnects the SAP environment as branches to Azure virtual WAN. Within the SAP managed Virtual Networks, Azure Virtual Networks Gateway are provisioned and IPSec Tunnels are established between the redundant Azure virtual WAN Gateways provisioned in the Hub and the redundant Azure VPN Gateways in the SAP Virtual Network.
Fore information on howto configure 

[!NOTE]
Inter-hub with firewall is currently not supported. Traffic between hubs will move directly bypassing the Azure Firewall in each hub.

Note
All options discussed in this article must be validated with customers SAP representative as SAP validated interconnect options may change over time.

Notes:
Global VNET Peering
Default Route 
Firewall 



Links – azure vwan VPN und Global VNET peering 
