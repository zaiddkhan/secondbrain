---
title: AWS Networking
date: 2025-03-06
draft: false
tags:
---


Networking, one of the core concepts when it comes to backend engineering.
What is the very first that comes into your mind when you hear the word networking.

It's meeting/contacting/communication new people/objects/servers.

In this blog we'll specifically learn about networking in the context of our aws stack.


## Virtual Private Cloud 

A **Virtual Private Cloud (VPC)** is like your own private, customizable, and secure space in the cloud. It’s where you can build, organize, and protect your applications and data, just like building and securing a house in a gated community.


VPC's in the context of AWS can live inside a region, they cannot communicate across regions.
A VPC is a logically isolated portion of the AWS cloud within a region.

Inside a VPC we can create multiple subnets within an availability zone. A subnet is always assigned to a singe availability zone but there can be many subnets in a single availability zone.
It can't span across multiple az's.

VPC Router - It takes care of all the routing and data connection that goes out of the subnet , i can be routed to another subnet or somewhere outside in the internet.
The vpc router is not something that we can see all we see is the route table from where destination and target are configured.

!![Image Description](/images/Pasted%20image%2020250305201409.png)


To connect our VPC to the internet there is something present known as the Internet Gateway, that allows to send data out to the internet, egress traffic. And allows data to travel from the internet to the vpc, ingress traffic. There can be only one internet gateway per vpc.

We configure our route table to the internet gateway id that tells it to send all the traffic that doesn't fit one of the networks in the route table to the internet gateway.



There can be multiple vpc's in a single aws region
Each vpc has a CIDR block.
CIDR stands for Classless Interdomain Routing
CIDR block is the overall block of addresses from which you then create the addresses we assign to the subnet. So it's kind of like a master block of addresses. 
Each vpc has a different block of IP addresses .

From the CIDR block we can then create the network id's for our subnet.
And they fit within the overall address block. But they have a different subnet mask, 
so essentially thats a subset of the overall available addresses and this is why it's really important to make sure we specify our cidr blocks correctly so that we have enough networks and enough hosts per network.



A high level overview of each of the component:
 

!![Image Description](/images/Pasted%20image%2020250306094316.png)


!![Image Description](/images/Pasted%20image%2020250306094724.png)



### Defining VPC CIDR Blocks

Rules and Guidelines
- CIDR block size can be between /16 and /28
- The CIDR block must not overlap with any existing CIDR block that's associated with the VPC
- You cannot increase or decrease the size of an existing CIDR block.
-  The first four and last IP address are not available for use.
- AWS recommend to use CIDR blocks from the RFC1918 ranges:

!![Image Description](/images/Pasted%20image%2020250306203630.png)


!![Image Description](/images/Pasted%20image%2020250306203827.png)

10.0.0.0/16

Subnet 1 - 10.0.1.0/24
Subnet 2 - 10.0.2.0/24

Each of the subnet will have 254 addresses to allot to the ec2.

Additional Considerations
- Ensure you have enough networks and hosts
- Bigger CIDR blocks are typically better (more flexibility)
- Smaller subnets are OK for most use cases
- Consider deploying application tiers per subnet
- VPC peering requires non-overlapping CIDR blocks.
-
!![Image Description](/images/Pasted%20image%2020250306204321.png)