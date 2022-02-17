# A Journey through Azure Networking

## Overview
Network is a critical component of a cloud. As an organization's cloud footprint scales and matures, the more complex the networking and routing becomes. While cloud providers such as Microsoft provide a means to enumerate [effective routes](https://docs.microsoft.com/en-us/azure/virtual-network/diagnose-network-routing-problem), it becomes a chore to visualize the effective routes across multiple virtual networks and subnets.

This repository contains a collection of diagrams that walk through the evolution of networking patterns organizations typically take when adopting Microsoft Azure. Each diagram includes a table displaying what the effective routes would look like for each of the virtual network subnets. It also includes important facts about each pattern and considerations organization's should be aware of if when they adopt a specific pattern.

For detail on the traffic flows of the more complex network architectures listed here, reference [this repository](https://github.com/mattfeltonma/azure-networking-patterns).

## Sections
* [Single Virtual Network and Single Subnet](#single-vnet-and-single-subnet)
* [Single Virtual Network and Multiple Subnets](#single-vnet-and-multiple-subnets)
* [Single Virtual Network and Multiple Workloads](#single-vnet-and-multiple-workloads)
* [Single Virtaul Network and On-Premises Connectivity](#single-vnet-and-on-premises-connectivity)
* [Peered Virtual Networks and On-Premises Connectivity and Multiple Workloads](#peered-vnets-and-on-premises-connectivity-with-multiple-workloads)
* [Hub and Spoke With a Flat Network](#hub-and-spoke-with-a-flat-network)
* [Hub and Spoke With a Flat Network and Forced Tunneling](#hub-and-spoke-with-a-flat-network-and-forced-tunneling)
* [Hub and Spoke With North and South Firewall](#hub-and-spoke-with-north-and-south-firewall)
* [Hub and Spoke With East and West Firewall and Forced Tunneling](#hub-and-spoke-with-east-and-west-firewall-and-forced-tunneling)
* [Hub and Spoke With Single Firewall For North South East West](#hub-and-spoke-with-single-firewall-for-north-south-east-west)
* [Hub and Spoke With Dedicated North and South Firewall and Dedicated East and West Firewall](#hub-and-spoke-with-dedicated-north-and-south-firewall-and-dedicated-east-and-west-firewall)

## Single VNet And Single Subnet
In this pattern there is a single virtual network with a single subnet all resources are placed in. VMs (virtual machines) use the default system routes to communicate with each other.

Considerations of this pattern include:

* Scaling this pattern can be a significant problem because subnets cannot be resized once network interfaces are associated with them.
* Mediation of traffic can be done with Network Security Groups, but it can be complex to manage the rules and could lead to mistakes.
* All resources have direct access to the Internet through the default system route.
* This pattern does not allow for connectivity back on-premises.

## Single VNet And Multiple Subnets
In this pattern there is a single virtual network with multiple subnets to allow for segmentation of each tier of a workload. VMs use the default system routes to communicate with each other and individual Network Security Groups can be configured and assigned to each subnet. This pattern is often used for proof-of-concepts with non-production data and no requirement for on-premises connectivity.

Considerations of this pattern include:
* All resources have direct access to the Internet through the default system route.
* This pattern does not allow for connectivity back on-premises.

## Single VNet And Multiple Workloads
In this pattern there is a single virtual network with multiple subnets hosting multiple workloads. VMs (virtual machines) use the default system routes to communicate with each other. Workloads are separated into different subnets and traffic is mediated with dedidated network security groups associated to each subnet. This allows for filtering communication between workloads.

Considerations of this pattern include:
* All resources will need to be in the same Azure subscription creating a single blast radius for both workloads.
* There is no simple way to restrict Azure RBAC permissions to management of a specific subnet(s) in the instance the workloads are managed by different teams.
* All resources have direct access to the Internet through the default system route.
* This pattern does not allow for connectivity back on-premises.

## Single VNet And On-Premises Connectivity
In this pattern there is a single virtual network with multiple subnets to allow for segmentation of each tier of a workload. Connectivity back on-premises is provided by a Virtual Network Gateway with either a VPN or ExpressRoute connection and routes are exchanged between on-premises and Azure using BGP. VMs use the default system routes to communicate with each other and individual Network Security Groups can be configured and assigned to each subnet. This pattern is often used for proof-of-concepts with non-production data but with a requirement for on-premises connectivity.

Considerations of this pattern include:
* The Virtual Network Gateway resource is in the same subscription as the workload. This creates a single blast radius which includes both on-premises connectivity and workload resources.
* All resources have direct access to the Internet through the default system route.

## Peered VNets And On-Premises Connectivity With Multiple Workloads
In this pattern there are two virtual networks in separate subscriptions which are peered. Workload resources for each workload are in separate subscriptions. Each virtual network is dedicated to a specific workload and is configured with multiple subnets to allow for segmentation of each tier of the workload. On-premises connectivity is provided to both workloads through a Virtual Network Gateway in one of the virtual networks with the peerings for the virtual networks configured to allow for [gateway transit](https://docs.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-peering-gateway-transit). Routes are exchanged between the Azure and on-premises using BGP. VMs use the default system routes to communicate with each other both within a virtual network and to its peered virtual network. Individual Network Security Groups can be configured and assigned to each subnet. This allows for filtering communication between workloads.

Considerations of this pattern include:
* The Virtual Network Gateway resource is in the same subscription as the workload. This creates a single blast radius for on-premises connectivity for both workloads.
* All resources have direct access to the Internet through the default system route.

## Hub And Spoke With A Flat Network
In this pattern there is a dedicated virtual network used for on-premises connectivity and workload-to-workload communication via a Virtual Network Gateway with either a VPN or ExpressRoute connection and routes are exchanged between on-premises and Azure using BGP. Each workload has a dedicated virtual network in a dedicated subscription. Peerings with the transit are configured to support [gateway transit](https://docs.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-peering-gateway-transit) allowing the routes advertised from on-premises to propagate into the workload virtual networks. Workloads can communicate with each other through the Virtual Network Gateway. Individual Network Security Groups can be configured and assigned to each subnet. This allows for filtering communication between workloads.

Considerations of this pattern include:
* This pattern creates a flat network where the only option for mediation between workloads is the Network Security Groups.
* All resources have direct access to the Internet through the default system route.

## Hub And Spoke With A Flat Network And Forced Tunneling

## Hub And Spoke With North And South Firewall

## Hub And Spoke With East and West Firewall And Forced Tunneling

## Hub And Spoke With Single Firewall For North South East West

## Hub And Spoke With Dedicated North And South Firewall And Dedicated East And West Firewall
