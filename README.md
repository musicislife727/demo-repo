# Demo

Some description!

## Subheader

Subheader text


<!-- About The Module -->

# VPC with Dynamic Subnets

Created by: Kevon Mayers
www.kevonmayers.com

Version 1.0.0

## About

This module can dynamically create public subnets, private subnets, a vpc, internet gateway, nat gateway, elastic ip, route tables, and subnet associations. The module uses Terraform's **_data source_** feature to fetch all AZs in whichever region you set in your provider block. The subnets are then created from those AZs, mapped one-to-one starting from the first AZ in a region.

This module creates a VPC with a dynamic amount of public/private subnets determined by the values of the variables 'PUBLIC_SUBNET_COUNT' and 'PRIVATE_SUBNET_COUNT

## Default Configuration

The default configure for the module is to launch the following:

- **3 Public Subnets** - (each with 32 hosts)
- **3 Private Subnets** - (each with 32 hosts)
- **Public/Private Route** Tables - for the subnets
- **Subnet Associations** - respective to public/private route tables
- **Internet Gateway** - for the public subnet route table
- **Nat Gateway** - for internet connectivity for the private subnets
- **Elastic IP** - to be used with the NAT Gateway

## Variables

### VPC

- **MAKE_CUSTOM_VPC** - **_bool_** - Enables VPC to be created. Set this to FALSE to bypass resource
- **CUSTOM_VPC_CIDR_BLOCK** - **_string_** - CIDR block used for Custom VPC - Default is **_192.168.0.0/24_**
- **CUSTOM_VPC_NAME** - **_string_** - Name tag for Custom VPC - default is **_customvpc_**

### Subnets

#### Public Subnets

- - **PUBLIC_SUBNET_COUNT** - **_number_** - Number of **PUBLIC** subnets created - Default is **_3_**
- - **PUBLIC_SUBNET_CIDR** - **_list_** - CIDR ranges for each public subnet **PUBLIC** subnets created - Default is **_3_**
- - **PUBLIC_SUBNET_NAME** - **_string_** - Name tag for **PUBLIC** subnet(s) created. Tag is **_SubnetName-pub-count.index_**

#### Private Subnets

- - **PRIVATE_SUBNET_COUNT** - **_number_** - Number of **PRIVATE** subnets created - Default is **_3_**
- - **PRIVATE_SUBNET_CIDR** - **_list_** - CIDR ranges for each public subnet **PRIVATE** subnets created - Default is **_3_**
- - **PRIVATE_SUBNET_NAME** - **_string_** - Name tag for **PRIVATE** subnets created. Tag is **_SubnetName-priv-count.index_**

- - **PRIVATE_SUBNET_COUNT** - **_number_** - Number of **PRIVATE** subnets created - Default is **_3_**
- - **CUSTOM_VPC_CIDR_BLOCK** - **_string_** - (each with 32 hosts)
- - **CUSTOM_VPC_CIDR_BLOCK** - **_string_** - (each with 32 hosts)
- - **CUSTOM_VPC_CIDR_BLOCK** - **_string_** - (each with 32 hosts)
- - **CUSTOM_VPC_CIDR_BLOCK** - **_string_** - (each with 32 hosts)

### Internet Gateway

- - **PRIVATE_SUBNET_COUNT** - string - Name tag for IGW created - Default is **_customvpc-IGW_**

### Route Tables

#### Public RT

- - **CUSTOM_VPC_PUBLIC_RT_NAME** - string - Name tag for Public RT created. Default is **_customvpc-public-RT_**
- - **CUSTOM_VPC_PUBLIC_RT_CIDR_IPV4** - string - CIDR ranges for the Public RT created. Default is **_0.0.0.0/0_**

#### Private RT

- - **MAKE_PRIVATE_RT** - bool - Enables Private RT to be created. Set this to be false to ignore this. Default is **_true_**
- - **CUSTOM_VPC_PRIVATE_RT_NAME** - string - Name tag for Private RT created. Default is **_customvpc-private-RT_**
- - **CUSTOM_VPC_PRIVATE_RT_CIDR_IPV4** - string - CIDR ranges for the Private RT created. Default is **_0.0.0.0/0_**

### Elastic IP (NAT)

- - **MAKE_EIP** - bool - Enables EIP to be created. Set this to be false to ignore. Default is **_true_**
- - **CUSTOM_VPC_NAT_EIP** - string - Name tag for EIP (associated with NAT GW) created - Default is **_customvpc--nat-eip_**

### NAT Gateway

- - **MAKE_NAT_GW** - bool - Enables NAT Gateway to be created. Set this to be false to ignore. Default is **_true_**
- - **CUSTOM_VPC_NAT_GW** - string - Name tag for NAT GW created. Default is **_customvpc-nat-gw_**

## Variable Usage

### Ex. 1 - Create 2 Public Subnets and 2 Private Subnets

```go
// VPC
CUSTOM_VPC_CIDR_BLOCK = "192.168.0.0/24"
CUSTOM_VPC_NAME = "My_Custom_VPC"
// PUBLIC SUBNET(S)
PUBLIC_SUBNET_COUNT = 2
PUBLIC_SUBNET_CIDR = ["192.168.0.0/24", "192.168.64/26"]
PUBLIC_SUBNET_NAME = "Yusuke"
// PRIVATE SUBNET(S)
PRIVATE_SUBNET_COUNT = 2;
PRIVATE_SUBNET_CIDR = ["192.168.0.128/24", "192.168.192/26"]
PRIVATE_SUBNET_NAME = "Togoro"
```

### Ex. 2 - Create 2 Public Subnets ONLY (no private subnets or related resources)

```go
// VPC
CUSTOM_VPC_CIDR_BLOCK = "192.168.0.0/24"
CUSTOM_VPC_NAME = "My_Custom_VPC"
// PUBLIC SUBNET(S)
PUBLIC_SUBNET_COUNT = 2;
PUBLIC_SUBNET_CIDR = ["192.168.0.0/25", "192.168.128/2"]
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
PUBLIC_SUBNET_COUNT = 4;
```

Assumming you selected non-overlapping CIDR blocks, the following will be created:

- 1st Public Subnet: AZ - us-east-2a
- 2nd Public Subnet: AZ - us-east-2b
- 3rd Public Subnet: AZ - us-east-2c
- 4th Public Subnet: AZ - us-east-2a

### Ex. 2 AWS_REGION = "us-east-1" (This region has 6 AZs)

```go
PUBLIC_SUBNET_COUNT = 6;
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

## Internet Gateway

An Internet Gateway will automatically be created and the PUBLIC subnets will be associated with it. The default name for the Internet Gateway is **_YourCustomVPCName-igw._**

## Route Table for Custom VPC (IGW Route for PUBLIC Subnets)

A Route table will automatically be created with a route for the IGW. The custom public subnets will automatically be associated with this route table.

The default CIDR block for the route to IGW is **_0.0.0.0/0_** (all ips). You can change this by modifying the vaule of the variable named **_CUSTOM_VPC_PUBLIC_RT_CIDR_IPV4_**.

For a **PUBLIC RT**, the default name for the route table is **_YourCustomVPCName-public-RT._**For a **PRIVATE RT**, the default name for the route table is **_YourCustomVPCName-private-RT._**

## NAT Gateway

By Default, a NAT Gateway will automatically be created and the PRIVATE subnets will be associated with it. Also, only ONE NAT Gateway will be created. To disable creation of the NAT Gateway, you must change the value of **_MAKE_NAT_GATEWAY_** from the default value of **_true_** to **_false_**. The default name for the NAT Gateway is **_YourCustomVPCName-nat-gw_**

## Elastic IP (EIP)

By Default, an EIP will automatically be created and associated with the NAT Gateway. To disable creation of the eip you must change the value of **_MAKE_EIP_** from the default of 'true' to 'false'. The default name for the NAT Gateway is **_YourCustomVPCName-nat-eip_**

## Route Table for Custom VPC (NAT Gateway for PRIVATE Subnets)

By Default, a Route table will automatically be created with a route for the NAT Gateway. The custom private subnets will automatically be associated with this route table. To disable creation of the route table for the PRIVATE subnets, you must change the value of **_MAKE_PRIVATE_RT_** from the default value of 'true' to 'false'.

The default CIDR block for the route to NAT Gateway is 0.0.0.0/0 (all ips). You can change this by modifying the vaule of the variable named .**_CUSTOM_VPC_PRIVATE_RT_CIDR_IPV4_**.

