# AWS Custom VPC with Public and Private Subnets

## Overview

This project demonstrates the creation and configuration of a custom Amazon Virtual Private Cloud (VPC) with public and private subnets to host Linux and Windows EC2 instances.

The implementation covers subnet design, route table configuration, Internet Gateway attachment, NAT Gateway configuration, EC2 deployment, and network connectivity testing.

The objective was to understand how AWS networking components work together to control internet access and connectivity between cloud resources.

## Objectives

- Create a custom VPC with an IPv4 CIDR block.
- Create public and private subnets.
- Configure separate route tables for public and private subnets.
- Associate the route tables with the appropriate subnets.
- Create and attach an Internet Gateway (IGW).
- Launch Linux EC2 instances in public and private subnets.
- Launch Windows EC2 instances and practise remote connectivity.
- Configure a NAT Gateway to enable outbound internet access from the private subnet.
- Test connectivity and understand the differences between public and private instances.

## AWS Services Used

- Amazon VPC
- Amazon EC2
- Internet Gateway (IGW)
- NAT Gateway
- Route Tables
- Public and Private Subnets
- Security Groups

## Technologies and Tools

- AWS Management Console
- Amazon Linux 2023
- Windows Server EC2
- SSH
- PuTTY
- Remote Desktop Protocol (RDP)
- Windows Command Prompt

## Architecture

The VPC contains public and private subnets. The public subnet uses an Internet Gateway for internet connectivity, while the private subnet uses a NAT Gateway for outbound internet access.

```text
                         Internet
                            |
                 +----------+----------+
                 |                     |
                 |    Internet Gateway |
                 |         (IGW)       |
                 +----------+----------+
                            |
                 +---------------------+
                 |    Public Subnet    |
                 |                     |
                 |  Linux EC2          |
                 |  Windows EC2        |
                 |                     |
                 |  Public Route Table |
                 +---------------------+
                            |
                       AWS VPC
                     10.0.0.0/16
                            |
                 +---------------------+
                 |   Private Subnet    |
                 |                     |
                 |  Linux EC2          |
                 |  Windows EC2        |
                 |                     |
                 | Private Route Table |
                 +----------+----------+
                            |
                      NAT Gateway
                    (Public Subnet)
                            |
                     Internet Gateway
                            |
                         Internet
```

**Note:** The diagram represents the intended routing design. The NAT Gateway must be placed in a public subnet and have a route to the Internet Gateway. Private subnet resources use the NAT Gateway for outbound internet access.

## Implementation Steps

### 1. Create a Custom VPC

Created a custom VPC named `Linux VPC` with the IPv4 CIDR block `10.0.0.0/16`.

This CIDR block defines the private IPv4 address range available for the VPC and its subnets.

Example AWS CLI command from the task:

```bash
aws ec2 create-vpc \
  --no-amazon-provided-ipv6-cidr-block \
  --instance-tenancy default \
  --cidr-block 10.0.0.0/16 \
  --tag-specifications \
  'ResourceType=vpc,Tags=[{Key=Name,Value=Linux VPC}]'
```

### 2. Create Public and Private Subnets

Created separate subnets inside the custom VPC.

The public subnet is intended for resources that require direct internet connectivity, such as a NAT Gateway or an EC2 instance configured with a public IP address.

The private subnet is intended for resources that should not be directly reachable from the public internet.

The subnet CIDR blocks must be within the VPC CIDR range and must not overlap.

### 3. Create Separate Route Tables

Created separate route tables for public and private subnet routing.

**Public Route Table**

Configured the public route table to use the Internet Gateway as its default route.

```text
Destination: 0.0.0.0/0
Target:      Internet Gateway
```

**Private Route Table**

Configured a separate route table for the private subnet. After creating the NAT Gateway, the private route table was updated to route internet-bound traffic through the NAT Gateway.

```text
Destination: 0.0.0.0/0
Target:      NAT Gateway
```

### 4. Associate Route Tables with Subnets

Associated the public route table with the public subnet and the private route table with the private subnet.

These associations determine which routes the resources in each subnet use.

### 5. Create and Attach an Internet Gateway

Created an Internet Gateway and attached it to the custom VPC.

Added a default route through the Internet Gateway to the public route table.

An Internet Gateway provides the VPC with a path to the internet, but an EC2 instance also needs suitable addressing, routing, and security configuration to communicate with external hosts.

### 6. Launch Linux EC2 Instances

Launched Linux EC2 instances in the public and private subnets.

The public instance was configured with a public IP address to support SSH access from the local machine, subject to the security group and network configuration.

The private instance did not have a public IP address and was used to explore private network connectivity.

### 7. Configure Security Groups

Configured security groups during EC2 creation and connectivity testing.

Security groups act as virtual firewalls controlling inbound and outbound traffic.

For secure access:

- Allow SSH (TCP port 22) from a trusted source for Linux administration.
- Allow RDP (TCP port 3389) from a trusted source for Windows administration.
- Permit only the traffic required for the task.
- Avoid exposing administrative ports to the entire internet.

The security group configuration must also permit the required traffic for the intended connectivity tests.

### 8. Test Public EC2 Connectivity

Connected to the public Linux EC2 instance using SSH from the local machine.

Tested internet connectivity from the instance using:

```bash
ping google.com
```

This test helped validate name resolution and outbound network connectivity, subject to the remote host responding to ICMP requests.

### 9. Test Private EC2 Connectivity

Attempted to connect directly to the private EC2 instance from the local machine.

The connection failed because the instance did not have a public IP address and was not directly reachable from the local machine over the internet.

This demonstrated an important networking concept: a private IP address is not directly reachable from a typical external internet connection.

Private instances generally require an appropriate access path, such as a bastion host, VPN, or AWS Systems Manager Session Manager, depending on the environment.

### 10. Launch Windows EC2 Instances

Launched Windows EC2 instances in the VPC and practised accessing them using Remote Desktop Protocol (RDP).

Used the associated key pair to retrieve the Windows administrator password through the AWS console.

The task also demonstrated that each Windows EC2 instance has its own generated administrator password, even when the same key pair is used to decrypt the password.

### 11. Create a NAT Gateway

Created a NAT Gateway in the public subnet and allocated an Elastic IP address.

A NAT Gateway allows resources in private subnets to initiate outbound connections to the internet without requiring public IP addresses on those private resources.

The NAT Gateway must have a valid route to the Internet Gateway through the public route table.

### 12. Update the Private Route Table

Updated the private route table to direct internet-bound traffic to the NAT Gateway.

```text
Destination: 0.0.0.0/0
Target:      NAT Gateway
```

This configuration allows eligible private subnet resources to initiate outbound internet connections through the NAT Gateway.

A NAT Gateway does not provide unsolicited inbound internet access to private instances.

### 13. Validate Private Subnet Internet Access

After configuring the NAT Gateway and private route table, tested internet connectivity from the private Windows instance.

The test demonstrated how a private instance can initiate outbound internet communication without having a public IP address.

Successful connectivity depends on the route table, NAT Gateway state, security rules, network ACLs, DNS configuration, and the destination service being reachable.

## Testing and Validation

The following scenarios were explored during the task:

| Test | Purpose |
|---|---|
| SSH to public Linux EC2 | Practise remote access to a public instance |
| Ping an internet hostname | Test outbound connectivity and name resolution |
| Direct connection to private EC2 | Understand the limitations of direct internet access to a private instance |
| RDP to Windows EC2 | Practise Windows remote administration |
| Configure NAT Gateway | Provide a path for private subnet outbound traffic |
| Update private route table | Route internet-bound traffic through the NAT Gateway |
| Test private instance internet access | Validate outbound connectivity after NAT configuration |

## Key Learnings

- Creating a custom VPC and planning IPv4 CIDR ranges.
- Understanding public and private subnet design.
- Configuring and associating route tables.
- Using an Internet Gateway for public subnet internet connectivity.
- Understanding NAT Gateway functionality.
- Differentiating direct inbound reachability from outbound internet access.
- Connecting to Linux EC2 instances using SSH.
- Connecting to Windows EC2 instances using RDP.
- Understanding the role of security groups in network access.
- Troubleshooting connectivity using practical tests.

## Conclusion

This hands-on project strengthened my understanding of AWS networking fundamentals and the components required to build a VPC with public and private subnets.

By configuring route tables, an Internet Gateway, a NAT Gateway, and EC2 instances, I explored how AWS controls network traffic and how private instances can access the internet without being directly exposed to it.

This project provides a foundation for learning more advanced AWS networking concepts, including bastion hosts, VPC endpoints, network ACLs, VPC peering, and hybrid connectivity.

## Project Documentation

This repository is part of my AWS and DevOps hands-on portfolio.

GitHub repository:  
https://github.com/pranitha-cloud-portfolio/aws-devops-hands-on
