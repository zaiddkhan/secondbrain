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

![[Pasted image 20250305201409.png]]


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
 

![[Pasted image 20250306094316.png]]


![[Pasted image 20250306094724.png]]



### Defining VPC CIDR Blocks

Rules and Guidelines
- CIDR block size can be between /16 and /28
- The CIDR block must not overlap with any existing CIDR block that's associated with the VPC
- You cannot increase or decrease the size of an existing CIDR block.
-  The first four and last IP address are not available for use.
- AWS recommend to use CIDR blocks from the RFC1918 ranges:

![[Pasted image 20250306203630.png]]


![[Pasted image 20250306203827.png]]

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

![[Pasted image 20250306204321.png]]


## Creating a custom vpc.

![[Pasted image 20250309222000.png]]

 We're going to create a vpc with a 10.0.0.0/16 CIDR block.
 
 The 1a 1b in the subnet name are associated to the availability zone they're present in.
 
 Also the private and public subn. ets have different route table, the MAIN table has a route to internet.

To create a new vpc, open the vpc dashboard  -> Your VPC's  -> Create VPC.

Step 1. Give a name to the vpc 

Step 2. Assign a cidr block to the vpc.

Step 3. Open the actions tab and also enable the domain hostname. That means we'll get the dns hostname for our ec2 instances.

Step 4. Create subnets
     i. Subnets tab -> Create subnet, select the vpc created before
     ii. Give a name to your subnet
     iii. Associate it to an availability zone
     iv. Assign a IPv4 CIDR block to it

Likewise create all the 4 subnets.

Step 5. For our public subnets -> Go to setting -> Modify auto-assign IP settings and enable the auto-assign public IPv4 address.

Step 6. Create route table for the private subnets -> Go to the route table dashboard -> Create route table
    i. Enter a name for our route table
    ii. Select a vpc

Step 7. Associate route table to the private subnet -> From the route table dashboard -> Go to edit subnet associations -> Select the private subnets.


Step 8. Creating the internet gateway  -> Go to the internet gateway dashboard -> Create internet gateway , give it a name and create the gateway.

Step 9. Attaching the internet gateway -> From the internet gateways dashboard -> Go to actions -> Attach to VPC.

Step 10. Adding a route to the internet gateway from our MAIN route table -> Go to route tables dashboard, select the public route table -> Select the routes tab -> Edit route -> Add route -> select 0.0.0.0/0 -> select our internet gateway as the target.


## Launching EC2 instances in the vpc

Create a NAT gateway.

What is a NAT gateway? 

-> NAT gateway is a service that allows private network resources to access the internet while keeping them secure. 
It translates private IP addresses to public IP addresses.

Why do we need a NAT gateway? 

-> After launching an ec2 instance, we need to get some binaries downloaded for which we need to have a internet connection & to be able to connect to the internet from our private subnet we need to have a NAT gateway.

After creating the NAT gateway, we have to alter the route table of our private subnet, add an ip address of 0.0.0.0/0 and associate it to the NAT gateway.


Create a Security Group

-> Security group is a firewall that allows/denies any inbound or outgoing traffic from our network, for now we'll allow all the traffic from both inbound and outbound.

Launching an instance using the cli

```
aws ec2 run-instances --image-id <imageid> --instance-type <value> --security-group-ids <value> --subnet-id <value> --key-name <value> --user-data file:// <value>
```


## Security Groups and Network ACLs


![[Pasted image 20250310142422.png]]


Both network acl and security groups are two different kinds of firewall, 
network acl is a firewall acting at the subnet level whereas security groups are applied at the instance level, the network interface of the instances.

Stateful vs Stateless Firewalls


![[Pasted image 20250310143109.png]]


The security group is a stateful firewall whereas the network acl is a stateless firewall.
The whole concept of stateful and stateless just revolves around how different firewall investigates the return traffic.

As in the picture above. 
A client connecting to our webserver, which is considered as an inbound traffic has the entry in the route table accordingly.
 Security groups automatically allows the return traffic thus stateful.
Whereas network acls do check for the outbound traffic even if the outbound message is going to the same connection from which it has received some incoming message.

Security groups don't have to have a rule for the return traffic but we might need an outbound rule for traffic going out from our instances or when our instances are initiating connections outbound.

![[Pasted image 20250310144533.png]]

