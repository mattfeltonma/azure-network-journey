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

## Single VNet And Multiple Subnets

## Single VNet And Multiple Workloads

## Single VNet And On-Premises Connectivity

## Peered VNets And On-Premises Connectivity With Multiple Workloads

## Hub And Spoke With A Flat Network

## Hub And Spoke With A Flat Network And Forced Tunneling

## Hub And Spoke With North And South Firewall

## Hub And Spoke With East and West Firewall And Forced Tunneling

## Hub And Spoke With Single Firewall For North South East West

## Hub And Spoke With Dedicated North And South Firewall And Dedicated East And West Firewall
