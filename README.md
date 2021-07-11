<!-- About The Module -->

# Module - VPC with Dynamic Subnets (and more!)

Created by: Kevon Mayers
www.kevonmayers.com

Version 1.0.0

## About

This module can dynamically create PUBLIC subnets, PRIVATE subnets, a custom VPC, Internet Gateway, NAT Gateway, Elastic IP (EIP), Route Tables, Subnet Associations, a Load Balancer (ALB, NLB, or both) and an Auto Scaling Group. The module uses Terraform's **_data source_** feature to fetch all AZs in whichever region you set in your provider block in your **main.tf** file. The subnets are then created from those AZs, mapped one-to-one starting from the first AZ in a region.

<!-- This module creates a VPC with a dynamic amount of public/private subnets determined by the values of the variables 'PUBLIC_SUBNET_COUNT' and 'PRIVATE_SUBNET_COUNT -->

## Default Configuration

The default configuration for the module is to launch the following:

- **Custom VPC** - with your specified CIDR range. Default name is **_customvpc_**

Every other resource and the amounts that you desire must be EXPLICITLY defined using the following variables:

## Variables

### VPC

- **MAKE_CUSTOM_VPC** - Enables VPC to be created. Default is **TRUE**. Set this to **FALSE** to bypass resource (but the point of this module is to make a custom VPC (and more!) so why would you do that :wink:)
- **CUSTOM_VPC_CIDR_BLOCK** - **_(REQUIRED)_** - CIDR block used for Custom VPC. Must be set by the user
- **CUSTOM_VPC_NAME** - **_(OPTIONAL)_** - Name tag for Custom VPC - default is **customvpc**

### Subnets

#### Public Subnets

- **PUBLIC_SUBNET_COUNT** - **_(OPTIONAL)_** - Number of **PUBLIC** subnets created. Must be set by the user
- **PUBLIC_SUBNET_CIDR** - **_(REQUIRED if making public subnets)_** - CIDR ranges for each **PUBLIC** subnet created. Must be set by the user
- **PUBLIC_SUBNET_NAME** - **_(OPTIONAL)_** - Name tag for **PUBLIC** subnet(s) created. Tag is **_YourSubnetName-pub-(count.index)+1_** with the default being **_subnet-pub-(count.index)+1_**

#### Private Subnets

- **PRIVATE_SUBNET_COUNT** - **_(OPTIONAL)_** - Number of **PRIVATE** subnets created.Must be set by the user
- **PRIVATE_SUBNET_CIDR** - **_(REQUIRED if making private subnets)_** - CIDR ranges for each **PRIVATE** subnet created. Must be set by the user
- **PRIVATE_SUBNET_NAME** - **_(OPTIONAL)_** - Name tag for **PRIVATE** subnets created. Tag is **_YourSubnetName-priv-(count.index)+1_** with the default being **_subnet-priv-(count.index)+1_**

### Internet Gateway

- **MAKE_IGW** - **_(REQUIRED if making public subnets)_** - Makes an Internet Gateway. Default is set to **FALSE**. Must set this to **TRUE** to create the resource. The name of the IGW will be **_YourCustomVPCName-igw_**
  **_Note: an IGW is REQUIRED for a subnet to be considered public._**

### Route Tables

#### Public RT

- **MAKE_PUBLIC_RT** - **_(REQUIRED if making public subnets)_** - Makes a custom Public Route Table. Default is set **FALSE**. Must set this to **TRUE** to create the resource. Default name is **_YourCustomVPCNAME-public-RT_**
  **_IMPORTANT: to enable hosts in a public subnet to access the Internet, a route table with a route to the IGW is REQUIRED. If this resource is not created, the IGW will not be attached to any route table, and the hosts will not have Internet access._**

- - **CUSTOM_VPC_PUBLIC_RT_CIDR_IPV4** - **_(OPTIONAL)_** - CIDR ranges for the Public RT created. Default is **_0.0.0.0/0_**

#### Private RT

- - **MAKE_PRIVATE_RT** - **_(REQUIRED if making private subnets)_** - Makes a Private Route Table. Default is set **FALSE**. Must set this to **TRUE** to create the resource. Default name is **_YourCustomVPCNAME-private-RT_**
    **_IMPORTANT: to enable hosts in a private subnet to access the Internet, you will either need to create a NAT Gateway (recommended), NAT Instance, or use one of the hosts in a public subnet as a Bastion(Jump) Host. If using a NAT Gateway, a route table with a route to the NAT Gateway must be created._**

- - **CUSTOM_VPC_PRIVATE_RT_CIDR_IPV4** - **_(REQUIRED if making private subnets)_** - CIDR ranges for the Private RT created. Default is **_0.0.0.0/0_**

### NAT Gateway

- - **MAKE_NAT_GW** - **_(OPTIONAL)_** - Enables NAT Gateway to be created. Set this to be false to ignore. Default is set to **_true_**. Default name is **_YourCustomVPCName-nat-gw_**

### Elastic IP (NAT)

- - **MAKE_EIP** - **_(REQUIRED if making a NAT Gateway)_** - Enables EIP to be created. Default is **FALSE**. Must set this to **TRUE** to create resource. Default name is **_YourCustomVPCName-nat-eip_**

## Variable Usage

### Ex. 1 - Create 2 Public Subnets and 2 Private Subnets

```go
// VPC
CUSTOM_VPC_CIDR_BLOCK = "192.168.0.0/24"
CUSTOM_VPC_NAME = "My_Custom_VPC"
// PUBLIC SUBNET(S)
PUBLIC_SUBNET_COUNT = 2
PUBLIC_SUBNET_CIDR = ["192.168.0.0/26", "192.168.0.64/26"]
PUBLIC_SUBNET_NAME = "Yusuke"
// PRIVATE SUBNET(S)
PRIVATE_SUBNET_COUNT = 2
PRIVATE_SUBNET_CIDR = ["192.168.0.128/26", "192.168.0.192/26"]
PRIVATE_SUBNET_NAME = "Togoro"
```

### Ex. 2 - Create 2 Public Subnets ONLY (no private subnets or related resources)

```go
 // VPC
CUSTOM_VPC_CIDR_BLOCK = "192.168.0.0/24"
CUSTOM_VPC_NAME = "My_Custom_VPC"
// PUBLIC SUBNET(S)
PUBLIC_SUBNET_COUNT = 2
PUBLIC_SUBNET_CIDR = ["192.168.0.0/25", "192.168.0.128/25"]
PUBLIC_SUBNET_NAME = "Kuwabara"
// PRIVATE SUBNET(S)
PRIVATE_SUBNET_COUNT = 0
MAKE_PRIVATE_RT = false
MAKE_EIP = false
MAKE_NAT_GW = false
```

## Availability Zone Functionality

The AZ for each subnet is dynamically set using Terraform's **_data source_** feature to fetch all AZs in whichever region you set in your provider block. The data source returns an array, or list, depending on which programming language you're familiar with. The subnets are then created from those AZs, mapped one-to-one starting from the first AZ in a region.

**IMPORTANT!** - If you set the count of a subnet to be greater than the amount of availbility zones for your selected region, the module will return to the beginning of the AZ list after iterating through it, and then continue down the list.

### Ex. 1 AWS_REGION = "us-east-2" (This region has 3 AZs)

```go
PUBLIC_SUBNET_COUNT = 4
```

Assumming you selected non-overlapping CIDR blocks, the following will be created:

- 1st Public Subnet: AZ - us-east-2a
- 2nd Public Subnet: AZ - us-east-2b
- 3rd Public Subnet: AZ - us-east-2c
- 4th Public Subnet: AZ - us-east-2a

### Ex. 2 AWS_REGION = "us-east-1" (This region has 6 AZs)

```go
PUBLIC_SUBNET_COUNT = 6
```

Assumming you selected non-overlapping CIDR blocks, the following will be created:

- 1st Public Subnet: AZ - us-east-1a
- 2nd Public Subnet: AZ - us-east-1b
- 3rd Public Subnet: AZ - us-east-1c
- 4th Public Subnet: AZ - us-east-1d
- 5th Public Subnet: AZ - us-east-1e
- 6th Public Subnet: AZ - us-east-1f

For more information refer to Terraform's documentation on the element() function.
https://www.terraform.io/docs/language/functions/element.html

Ex. AWS_REGION = "us-east-1" (This region has 6 AZs)
PUBLIC_SUBNET_COUNT = 6

<!-- Subnetting -->

## VPC/Subnet CIDR Ranges

The default configuration deploys 6 subnets, 3 public and 3 private. The default values are as follows:

```go
PUBLIC_SUBNET_COUNT = 3
PUBLIC_SUBNET_CIDR = ["192.168.0.0/27","192.168.0.32/27","192.168.0.64/27"]
PRIVATE_SUBNET_COUNT = 3
PRIVATE_SUBNET_CIDR = ["192.168.0.96/27","192.168.0.128/27","192.168.0.160/27"]
```

These values can be modified by simply defining your own values for the PUBLIC_SUBNET_CIDR and PRIVATE_SUBNET_CIDR arrays, as well as modifying the count for the PUBLIC_SUBNET_COUNT and PRIVATE_SUBNET_COUNT variables.

When deciding on your CIDR ranges, keep in mind a few things:

- A VPC must be between a /16 and /28 range.
- CIDR ranges must not overlap
- Beyond the **2** IPs reserved for **gateway** and **broadcast addresses**, AWS reserves 3 additional addresses, so **5 in total.**

- **IMPORTANT!** If you have a NAT Gateway, it must be attached to a public subnet. The NAT gateway will use **_1_** of the available IP addresses. So, for a public subnet with a **_NAT Gatway_** attached to it, it will have 1 available IP address less than public subnets without a NAT Gateway attached.

For more information, refer to the VPC FAQs on the AWS website.
https://aws.amazon.com/vpc/faqs/#:~:text=Amazon%20reserves%20the%20first%20four,subnet%20for%20IP%20networking%20purposes.

## Internet Gateway

An Internet Gateway will automatically be created and the PUBLIC subnets will be associated with it. The default name for the Internet Gateway is **_YourCustomVPCName-igw._**

## Route Table for Custom VPC (IGW Route for PUBLIC Subnets)

By default, a custom Route table will automatically be created with a route for the IGW. The custom public subnets will automatically be associated with this route table. This is separate from the default route table that is provided by AWS with every new VPC creation.

The default CIDR block for the route to IGW is **_0.0.0.0/0_** (all ips). You can change this by modifying the vaule of the variable named **_CUSTOM_VPC_PUBLIC_RT_CIDR_IPV4_**.

For a **PUBLIC RT**, the default name for the route table is **_YourCustomVPCName-public-RT._**For a **PRIVATE RT**, the default name for the route table is **_YourCustomVPCName-private-RT._**

## NAT Gateway

By Default, a NAT Gateway will automatically be created and the PRIVATE subnets will be associated with it. Also, only ONE NAT Gateway will be created. To disable creation of the NAT Gateway, you must change the value of **_MAKE_NAT_GATEWAY_** from the default value of **_true_** to **_false_**. The default name for the NAT Gateway is **_YourCustomVPCName-nat-gw_**

## Elastic IP (EIP)

By Default, an EIP will automatically be created and associated with the NAT Gateway. To disable creation of the eip you must change the value of **_MAKE_EIP_** from the default of **_true_** to **_false_**. The default name for the NAT Gateway is **_YourCustomVPCName-nat-eip_**

## Route Table for Custom VPC (NAT Gateway for PRIVATE Subnets)

By default, a custom Route table will automatically be created with a route for the NAT Gateway. The custom private subnets will automatically be associated with this route table. This is separate from the default route table that is provided by AWS with every new VPC creation. To disable creation of the route table for the PRIVATE subnets, you must change the value of **_MAKE_PRIVATE_RT_** from the default value of **_true_** to **_false_**.

The default CIDR block for the route to NAT Gateway is 0.0.0.0/0 (all ips). You can change this by modifying the vaule of the variable named **_CUSTOM_VPC_PRIVATE_RT_CIDR_IPV4_**.
