# AWS Virtual Private Cloud (VPC) CIDR Block for VPC
## Internet Gateway
## Subnets
### Route Tables
#### Network Access Control list (NACLs)
##### Security Groups 

![](AWS_deployment_networking_security.png)

- What is a VPC?
- AWS isolated virtual network - It allows us to control virtual network environment including selectionof your own IPs address range, we can create multiple subnets within one VPC with specific network configuration. We can use both IPv4 and IPv6 for most resources, it provides security for your services or instances

- What is an Internet Gateway?
-  An Internet gateway - can transfer communications between an enterprise network and the internet - it allows internet access into the VPC
  
- What is Subnet?
- A Subnet is a segmented piece of a larget network - the goal of a subnet to split a large network intor a group of smaller, interconnected networks to help minimise traffice - or navigate traffice securly  

- Route Table RT
- RT contains set of rules, called routes, that are used to determine where the network traffic form your subnet or gateway is directed

- Network Access Control List (NACL)
- NACLs are stateless - we have to explicitly allow inbound and outbound rules - they are an added layer of security at subnet leve

- 4.3 Billion IP address in the world aproximatley 
- 
- Step 1: create a VPC with IPV valid CIDR block
- `10.0.0.0/16` - next tea member to use `10..10.0.0/16`

- Step 2: Create internet gateway
- 2.1: Attach the IG to your VPC

- Step 3: Create route table
- 3.1 Edit route and insert your IG in `target`

- Step 4: Create public subnet
- -`10.0.1.0/24`
- 4.1 associate public subnet with our RT

- Step 5: Create publick NACLs
- set inbound and outbound rules for this 

- Step 6: Create a Security group for our app















