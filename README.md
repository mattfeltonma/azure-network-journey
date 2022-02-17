# (WIP) A Journey through Azure Networking

## Overview
Network is a critical component of a cloud. As an organization's cloud footprint scales and matures, the more complex the networking and routing becomes. While cloud providers such as Microsoft provide a means to enumerate [effective routes](https://docs.microsoft.com/en-us/azure/virtual-network/diagnose-network-routing-problem), it becomes a chore to visualize the effective routes across multiple virtual networks and subnets.

This repository contains a collection of diagrams that walk through the evolution of networking patterns organizations typically take when adopting Microsoft Azure. Each diagram includes a table displaying what the effective routes would look like for each of the virtual network subnets. It also includes important facts about each pattern and considerations organization's should be aware of if when they adopt a specific pattern.

**Note that in the patterns with firewalls, the firewalls are assumed to have a separate dedicated network interface in a management subnet. This is not shown in the diagrams.**

**Credit for blackhole routes in the advanced patterns to [Palo Alto](https://www.paloaltonetworks.com/resources/guides/azure-transit-vnet-deployment-guide).**

For detail on the traffic flows of the more complex network architectures listed here, reference [this repository](https://github.com/mattfeltonma/azure-networking-patterns).

## Sections
* [Single Virtual Network and Single Subnet](#single-vnet-and-single-subnet)
* [Single Virtual Network and Multiple Subnets](#single-vnet-and-multiple-subnets)
* [Single Virtual Network and Multiple Workloads](#single-vnet-and-multiple-workloads)
* [Single Virtaul Network and On-Premises Connectivity](#single-vnet-and-on-premises-connectivity)
* [Peered Virtual Networks and On-Premises Connectivity and Multiple Workloads](#peered-vnets-and-on-premises-connectivity-with-multiple-workloads)
* [Hub and Spoke With a Flat Network](#hub-and-spoke-with-a-flat-network)
* [Hub and Spoke With a Flat Network and Forced Tunneling](#hub-and-spoke-with-a-flat-network-and-forced-tunneling)
* [Hub and Spoke With East and West Firewall and Forced Tunneling](#hub-and-spoke-with-east-and-west-firewall-and-forced-tunneling)
* [Hub and Spoke With Single Firewall For North South East West](#hub-and-spoke-with-single-firewall-for-north-south-east-west)
* [Hub and Spoke With Dedicated North and South Firewall and Dedicated East and West Firewall](#hub-and-spoke-with-dedicated-north-and-south-firewall-and-dedicated-east-and-west-firewall)

## Single VNet And Single Subnet
In this pattern there is a single virtual network with a single subnet all resources are placed in. 

Facts:
* VMs (virtual machines) use the default system routes to communicate with each other.

Considerations of this pattern include:

* Scaling this pattern can be a significant problem because subnets cannot be resized once network interfaces are associated with them.
* Mediation of traffic can be done with Network Security Groups, but it can be complex to manage the rules and could lead to mistakes.
* All resources have direct access to the Internet through the default system route.
* This pattern does not allow for connectivity back on-premises.

## Single VNet And Multiple Subnets
In this pattern there is a single virtual network with multiple subnets.

This is a common pattern for proof-of-concepts where there is no requirement for on-premises connectivity.

Facts:
* Individual subnets can be assigned separate Network Security Groups controlling inbound and outbound traffic for each tier of the workload.
* VMs (virtual machines) use the default system routes to communicate with each other.

Considerations of this pattern include:
* All resources have direct access to the Internet through the default system route.
* This pattern does not allow for connectivity back on-premises.

## Single VNet And Multiple Workloads
In this pattern there is a single virtual network with multiple subnets hosting multiple workloads. 

Facts:
* Individual subnets can be assigned separate Network Security Groups controlling inbound and outbound traffic for each tier of the workload and between workloads.
* VMs (virtual machines) use the default system routes to communicate with each other which includes cross workload.

Considerations of this pattern include:
* All resources will need to be in the same Azure subscription creating a single blast radius for both workloads.
* There is no simple way to restrict Azure RBAC permissions to management of a specific subnet(s) in the situation where the workloads are managed by different teams.
* All resources have direct access to the Internet through the default system route.
* This pattern does not allow for connectivity back on-premises.

## Single VNet And On-Premises Connectivity
In this pattern there is a single virtual network with multiple subnets and the workloads require on-premises connectivity. 

This is a common pattern for proof-of-concepts where on-premises connectivity is required.

## Facts
* VMs (virtual machines) use the default system routes to communicate with each other.
* Connectivity back on-premises is provided by a Virtual Network Gateway with either a VPN or ExpressRoute connection.
* Routes are exchanged between on-premises network equipment and the Virtual Network Gateway using BGP. 

Considerations of this pattern include:
* The Virtual Network Gateway resource is in the same subscription as the workload. This creates a single blast radius which includes both on-premises connectivity and workload resources.
* All resources have direct access to the Internet through the default system route.

## Peered VNets And On-Premises Connectivity With Multiple Workloads
In this pattern each workload is its own virtual network and requires on-premises connectivity. 

Facts:
* VMs (virtual machines) use the default system routes to communicate with each other which includes cross workload.
* Workloads have dedicated subscriptions containing their virtual network and workload resources creating separate blast radiuses.
* Connectivity back on-premises is provided to both workloads by a Virtual Network Gateway in one of the virtual networks.
* Peerings between the virtual networks are configured [gateway transit](https://docs.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-peering-gateway-transit) allowing the workloads to share the on-premises connectivity provided by the Virtual Network Gateway because the learned routes are propagated into the peered virtual network.
* Individual subnets can be assigned separate Network Security Groups controlling inbound and outbound traffic for each tier of the workload and between workloads.

Considerations of this pattern include:
* The Virtual Network Gateway resource is in the same subscription as the workload. This creates a single blast radius for on-premises connectivity for both workloads.
* All resources have direct access to the Internet through the default system route.

## Hub And Spoke With A Flat Network
In this pattern there is a dedicated virtual network used for on-premises connectivity which is shared with each workload that each have their own dedicated virtual network.

### Facts:
* VMs (virtual machines) use the default system routes to communicate with each other within a virtual network.
* Workloads have dedicated subscriptions containing their virtual network and workload resources creating separate blast radiuses.
* Connectivity back on-premises is provided to both workloads by a Virtual Network Gateway in the transit virtual network.
* Peerings between the transit virtual networks and workload virtual networks is configured with [gateway transit](https://docs.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-peering-gateway-transit) allowing the workloads to share the on-premises connectivity provided by the Virtual Network Gateway because the learned routes are propagated into each workload virtual network.
* VMs use routes learned from the Virtual Network Gateway to communicate cross workload.
* Individual subnets can be assigned separate Network Security Groups controlling inbound and outbound traffic for each tier of the workload and between workloads.

### Considerations of this pattern include:
* This pattern creates a flat network where the only option for mediation between workloads is the Network Security Groups.
* All resources have direct access to the Internet through the default system route.

## Hub And Spoke With A Flat Network And Forced Tunneling
In this pattern there is a dedicated virtual network used for on-premises connectivity which is shared with each workload that each have their own dedicated virtual network and there is a requirement to send Internet-bound traffic back on-premises for inspection, mediation, filtering, and/or logging.

This is a common pattern for organizations new to Azure that may have a significant capital investment in security appliances on-premises that are not yet fully depreciated and are comfortable mediating network traffic between workloads using Network Security Groups.

Facts:
* VMs (virtual machines) use the default system routes to communicate with each other within a virtual network.
* Workloads have dedicated subscriptions containing their virtual network and workload resources creating separate blast radiuses.
* Connectivity back on-premises is provided to both workloads by a Virtual Network Gateway in the transit virtual network.
* Peerings between the transit virtual networks and workload virtual networks is configured with [gateway transit](https://docs.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-peering-gateway-transit) allowing the workloads to share the on-premises connectivity provided by the Virtual Network Gateway because the learned routes are propagated into each workload virtual network.
* VMs use routes learned from the Virtual Network Gateway to communicate cross workload.
* Individual subnets can be assigned separate Network Security Groups controlling inbound and outbound traffic for each tier of the workload and between workloads.
* A default route learned from on-premises is propagated to each workload virtual network invalidate the default system route to the Internet forcing Internet-bound traffic back on-premises.

Considerations of this pattern include:
* This pattern creates a flat network where the only option for mediation between workloads is the Network Security Groups.
* Additional costs and latency will be incurred for egressing Internet-bound traffic back on-premises.

## Hub And Spoke With East and West Firewall And Forced Tunneling
In this pattern there is a dedicated virtual network used for on-premises connectivity which is shared with each workload that each have their own dedicated virtual network and there is a requirement to send Internet-bound traffic back on-premises for inspection, mediation, filtering, and/or logging. There is also a requirement for inspection, mediation, and/or logging for traffic between on-premises and workloads and for traffic between workloads but these activities must be performed by a firewall in Azure.

This is a common pattern for organizations new to Azure that may have a significant capital investment in security appliances on-premises that are not yet fully depreciated but want mediation, inspection, and/or centralized logging between workloads which is provided by a firewall in Azure.

Facts:
* VMs (virtual machines) use the default system routes to communicate with each other within a virtual network.
* Workloads have dedicated subscriptions containing their virtual network and workload resources creating separate blast radiuses.
* Connectivity back on-premises is provided to both workloads by a Virtual Network Gateway in the transit virtual network.
* Peerings between the transit virtual networks and workload virtual networks are not configured for [gateway transit](https://docs.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-peering-gateway-transit). Virtual networks do not learn on-premises routes or the routes of other workload virtual networks.
* Firewalls are configured with a network interface in a private subnet fronted by an ILB (internal load balancer). The default route learned from on-premises is propagated to the route table of this subnet and forces Internet-bound traffic processed by the firewall to route back on-premises.
* A static route is configured using a UDR (User Defined Routes) on each subnet's route table in each workload virtual network with a default route pointing to the firewall's ILB (internal load balancer) IP address. This forces all traffic to the Internet, on-premises, and other workloads to be routed to the firewall. Static routes are also configured using UDRs on the route table assigned Virtual Network Gateway subnet (GatewaySubnet) to force traffic destined for the workload virtual networks through the firewall's ILB IP address. This ensures symmetric routing.

Considerations of this pattern include:
* This pattern creates a flat network where the only option for mediation between workloads is the Network Security Groups.
* Additional costs of the firewall running in Azure.
* Additional costs and latency will be incurred for egressing Internet-bound traffic back on-premises.


## Hub And Spoke With Single Firewall For North South East West
In this pattern there is a dedicated virtual network used for on-premises connectivity which is shared with each workload that each have their own dedicated virtual network. There is requirement for Internet-bound traffic, traffic between on-premises and Azure, and traffic between workloads in Azure to be mediated, inspected, and/or centrally logged by firewalls in Azure.

This is one of the more common patterns for organizations using Azure.

Organized are encouraged to explore [Azure Virtual WAN](https://docs.microsoft.com/en-us/azure/virtual-wan/virtual-wan-about) in place of this pattern due to the additional capabilities and managed nature of Virtual WAN. There are significant considerations to using [Azure Virtual WAN](https://docs.microsoft.com/en-us/azure/virtual-wan/virtual-wan-faq) and organizations are encouraged to consult with their Microsoft account teams or trusted Microsoft partners before moving ahead with Azure Virtual WAN.

Facts:
* VMs (virtual machines) use the default system routes to communicate with each other within a virtual network.
* Workloads have dedicated subscriptions containing their virtual network and workload resources creating separate blast radiuses.
* Connectivity back on-premises is provided to both workloads by a Virtual Network Gateway in the transit virtual network.
* Peerings between the transit virtual networks and workload virtual networks are not configured for [gateway transit](https://docs.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-peering-gateway-transit). Virtual networks do not learn on-premises routes or the routes of other workload virtual networks.
* Firewalls are configured with a network interface in a private subnet fronted by an ILB (internal load balancer) in addition to an interface in a public subnet fronted by a ELB (external load balancer).
* Static routes are configured using UDRs (User Defined Routes) on the firewall's public subnet to blackhole traffic destined for the firewall's private subnet or any workload virtual network. The route table is also configured with BGP propagation disabled to ensure it does not learn about the routes from on-premises. This allows the organization to control traffic coming in from the Internet using the firewall.
* Static routes are configured using UDRs on the firewall's private subnet to blackhole traffic destined for the firewall's public subnet and to direct any Internet-bound traffic to the firewall's ILB IP address.
* A static route is configured using a UDR on each subnet's route table in each workload virtual network with a default route pointing to the firewall's ILB (internal load balancer) IP address. This forces all traffic to the Internet, on-premises, and other workloads to be routed to the firewall. Static routes are also configured using UDRs on the route table assigned Virtual Network Gateway subnet (GatewaySubnet) to force traffic destined for the workload virtual networks through the firewall's ILB IP address. This ensures symmetric routing.
* A static route is configured using a UDR on the route table assigned to the Virtual Network Gateway subnet (GatewaySubnet) to blackhole traffic destined for the firewall's public subnet.

Considerations of this pattern include:
* All north/south/east/west traffic flows through a single set of firewalls which could create a bottleneck.
* Additional costs of the firewall running in Azure.

## Hub And Spoke With Dedicated North And South Firewall And Dedicated East And West Firewall
In this pattern there is a dedicated virtual network used for on-premises connectivity which is shared with each workload that each have their own dedicated virtual network. There is requirement for Internet-bound traffic, traffic between on-premises and Azure, and traffic between workloads in Azure to be mediated, inspected, and/or centrally logged by firewalls in Azure. There is a separate firewall stack for north/south traffic and another for east/west traffic.

This is one of the more common patterns for organizations using Azure that have a significant amount of internal and externallly facing workloads and would like to mitigate the risk of a bottleneck.

Organized are encouraged to explore [Azure Virtual WAN](https://docs.microsoft.com/en-us/azure/virtual-wan/virtual-wan-about) in place of this pattern due to the additional capabilities and managed nature of Virtual WAN. There are significant considerations to using [Azure Virtual WAN](https://docs.microsoft.com/en-us/azure/virtual-wan/virtual-wan-faq) and organizations are encouraged to consult with their Microsoft account teams or trusted Microsoft partners before moving ahead with Azure Virtual WAN.

Facts:
* VMs (virtual machines) use the default system routes to communicate with each other within a virtual network.
* Workloads have dedicated subscriptions containing their virtual network and workload resources creating separate blast radiuses.
* Connectivity back on-premises is provided to both workloads by a Virtual Network Gateway in the transit virtual network.
* Peerings between the transit virtual networks and workload virtual networks are not configured for [gateway transit](https://docs.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-peering-gateway-transit). Virtual networks do not learn on-premises routes or the routes of other workload virtual networks.
* The north/south firewalls are configured with a network interface in a private subnet fronted by an ILB (internal load balancer) in addition to an interface in a public subnet fronted by a ELB (external load balancer).
* The east/west firewalls are configured with a network interface in a private subnet fronted by an ILB.
* Static routes are configured using UDRs (User Defined Routes) on the north/south firewall's public subnet to blackhole traffic destined for both the north/south and east/west firewall's private subnets or any workload virtual network. The route table is also configured with BGP propagation disabled to ensure it does not learn about the routes from on-premises. This allows the organization to control traffic coming in from the Internet using the firewall.
* Static routes are configured using UDRs on the both the north/south and east/west firewall's private subnet to blackhole traffic destined for the firewall's public subnet and to direct any Internet-bound traffic to the north/south firewall's ILB IP address.
* A static route is configured using a UDR on each subnet's route table in each workload virtual network with a default route pointing to the north/south firewall's ILB IP address. Another set of static route are configured to force all traffic destined for on-premises or the address range dedicated to Azure to the east/west firewall's ILB IP address. This forces all traffic to the Internet, on-premises, and other workloads to be routed through the relevant firewall. Static routes are also configured using UDRs on the route table assigned Virtual Network Gateway subnet (GatewaySubnet) to force traffic destined for the workload virtual networks through the east/west firewall's ILB IP address. This ensures symmetric routing.
* A static route is configured using a UDR on the route table assigned to the Virtual Network Gateway subnet (GatewaySubnet) to blackhole traffic destined for the north/south firewall's public subnet.

Considerations of this pattern include:
* Requires significant planning with IP space to ensure the UDRs in the workload virtual networks can remain relatively static.
* Routing can be complex and prone to misconfiguration.
* Additional costs of running separate firewall stacks dedicated to north/south and east/west.

