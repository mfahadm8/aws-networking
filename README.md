# Enigma: AWS Network Provisioning

## Overview

The project is designed to provision and manage AWS network infrastructure for the Enigma . The infrastructure spans multiple regions, including Oregon, Ireland, Singapore (for Core infrastructure), and Ohio (for VCL infrastructure). Enigma automates the creation of various components to ensure a seamless and robust network setup.

## Architecture
![Enigma Architecture](./images/enigma.png "Enigma Network Architecture")

## Features

### 1. Transit Gateways

Enigma creates Transit Gateways to facilitate efficient communication between different AWS regions. This enhances the connectivity and performance of the network.

**Details:**
- Creates 4 Transit Gateways:
  1. `tgw-main` in Oregon (us-west-2) region.
  2. `sg-tgw` in Singapore (ap-southeast-1) region.
  3. `ir-tgw` in Ireland (eu-west-1) region.
  4. `tgw-vcl` in Ohio (us-east-2) region.

#### `tgw-main`:
- Serves as the central transit gateway.
- All core infrastructure of both development and production environments (including workspace VPC, lab VPC, microservices VPC, and omnispin VPC) is attached to this transit gateway.
- The Site-to-Site VPN connection linking Azure Cloud to our AWS Cloud is also attached to this transit gateway.
- The VpnVPC, housing the OpenVPN server, is connected to this transit gateway.
- `sg-tgw`, `ir-tgw`, and `tgw-vcl` are peered with this transit gateway.

#### `sg-tgw`:
- It connects the Lab VPC's of both development and production environments from singapore region to `tgw-main` in oregon.

#### `ir-tgw`:
- It connects the Lab VPC's of both development and production environments from ireland region to `tgw-main` in oregon.

#### `tgw-vcl`:
- It connects the VCL (DB Cloud Lake) Infrastructure from ohio region to `tgw-main` in oregon.


### 2. Peering Automation

The project automates the peering process between Transit Gateways, streamlining the establishment of connections across regions. It automates the peering connection initiation from one Transit Gateway as well as accepting the connection request on receiver Transit Gateway.

### 3. Routing Management

Enigma adds routes to Peered Transit Gateway route tables, ensuring proper routing of network traffic and optimizing communication between regions.

### 4. Core Network Infrastructure

Enigma creates VPCs, subnets, internet gateways, and NAT gateways for both development and production environments. It also adds routes to VPC route tables and shares them with development and production accounts using AWS Resource Access Manager.

### 5. DB Cloud DB Integration

Enigma provisions the DB Cloud DB VPC, updates the VCL (DB Cloud Lake) VPC route table, and creates VPC endpoints in the VCL VPC. This facilitates the connection between DB Tenant and the Enigma environment. DNS entries for the VCL VPC endpoint are also created in development and production accounts for the DB endpoint in the environment.

### 6. Cloud-to-Cloud VPN

Enigma establishes a VPN connection that links the AWS Cloud to the Azure Cloud. This includes creating a Customer Gateway and a site-to-site VPN, connecting AWS and Azure Clouds through VPN over the Internet.

### 7. Common VPN VPC

A VPN VPC is created, housing the OpenVPN server, serving as a common VPC for the entire infrastructure. This VPC facilitates secure communication within the environment.

---

## Details

### Deployment Steps

1. Deploy all Transit Gateways.
   1. Deploy Stack `TGWMain`.
   2. Deploy Stack `TGWLabSG`. 
   3. Deploy Stack `TGWLabIR`.
   4. Export the parameters (refer to `.gitlab-ci.yml`).
2. Perform the Transit Gateway Peering and add Routes.
   1. Deploy the Stack `SGTGWPeering`.
      1. Accept Peering attachment request.
      2. Deploy Stack `TGWMainSGPeeringRoute`.
      3. Deploy Stack `TGWSGMainPeeringRoute`.
   2. Deploy the Stack `IRTGWPeering`.
      1. Accept Peering attachment request.
      2. Deploy Stack `TGWMainIRPeeringRoute`.
      3. Deploy Stack `TGWIRMainPeeringRoute`.
3. Deploy Core Infrastructure.
   1. Deploy Stack `InfraDev`.
   2. Deploy Stack `InfraProd`.
   3. Deploy Stack `InfraDevSGLab`.
   4. Deploy Stack `InfraProdSGLab`.
   5. Deploy Stack `InfraDevIRLab`.
   6. Deploy Stack `InfraProdIRLab`.
   7. Export the parameters (refer to `.gitlab-ci.yml`).
4.  Share the Infrastructure Using RAM.
    1.  Deploy Stack `RamDev`.
    2.  Deploy Stack `RamDevSG`.
    3.  Deploy Stack `RamDevIR`.
    4.  Deploy Stack `RamProd`.
    5.  Deploy Stack `RamProdSG`.
    6.  Deploy Stack `RamProdIR`.
    7.  Export the parameters (refer to `.gitlab-ci.yml`).
    8.  Create SSM parameters in accounts (refer to `.gitlab-ci.yml`).
5.  Deploy VCL Transit Gateway.
    1.  Deploy Stack `TGWVCL`.
    2.  Export the parameters (refer to `pipeline/vcl/vcl_deploy.yml`).
    3.  Perform the Transit Gateway Peering and add Routes.
        1. Deploy Stack `TGWPeering`.
            1. Accept Peering attachment request.
            2. Deploy Stack `TGWMainPeeringRoute`.
            3. Deploy Stack `TGWVCLPeeringRoute`.
6. Deploy VCL Infrastructure.
   1.  Deploy Stack `InfraVCL`.
   2.  Export the parameters (refer to `pipeline/vcl/vcl_deploy.yml`).
   3.  Create SSM parameters in accounts (refer to `pipeline/vcl/vcl_deploy.yml`).
   4.  Deploy Stack `VCLDNSRecordDev`.
   5.  Deploy Stack `VCLDNSRecordProd`.
 


### CIDR Ranges

- **Production IP Range:** `10.0.0.0/16` - `10.50.0.0/16`

- **Development IP Range:** `10.51.0.0/16` - `10.70.0.0/16`

- **Shared IP Range:** `10.71.0.0/16` - `10.80.0.0/16`

- **Azure IP Range:** `10.96.0.0/16` - `10.127.0.0/16` 
  - Currently using with a subnet mask of `10.96.0.0/11`, this way we achieve the communication by adding only one CIDR to our route tables for Azure and further network segmentation is done on Azure end.

###  VPC Usage

#### Lab

**Usage:**
- VILT labs

#### Workspace

**Usage:**
- SPT Workspaces
- VILT Workspaces

#### Microservices

**Usage:**
- SPT/VILT Micro-services

#### OmniSpin

**Usage:**
- JupyterHub Kubernetes

#### VCL

**Usage:**
- VCL Tenant Endpoint

### Transit Gateway Routes

#### tgw-main (ASN:64512) Route Table

| Route          | Destination        | Destination Type |
| -------------- | ------------------ | ----------------- |
| **Development**|                    |                   |
| 10.51.0.0/16   | LabVpcDev          | VPC               |
| 10.52.0.0/16   | WorkspaceVpcDev    | VPC               |
| 10.53.0.0/16   | MSVpcDev           | VPC               |
| 10.54.0.0/16   | OmniSpinVpcDev     | VPC               |
| 10.55.0.0/16   | SGLabVpcDev        | Peering           |
| 10.56.0.0/16   | IRLabVpcDev        | Peering           |
| **Production** |                    |                   |
| 10.0.0.0/16    | LabVpcProd         | VPC               |
| 10.1.0.0/16    | WorkspaceVpcProd   | VPC               |
| 10.2.0.0/16    | MSVpcProd          | VPC               |
| 10.3.0.0/16    | OmniSpinVpcProd    | VPC               |
| 10.4.0.0/16    | SGLabVpcProd       | Peering           |
| 10.5.0.0/16    | IRLabVpcProd       | Peering           |
| **Common**     |                    |                   |
| 10.71.0.0/16   | VCLVpc             | Peering           |
| 10.72.0.0/16   | VpnVpc             | VPC               |
| **VPN**        |                    |                   |
| 10.96.0.0/11   | Azure Cloud         | VPN               |


#### sg-tgw (ASN:64514) Route Table

| Route          | Destination        | Destination Type |
| -------------- | ------------------ | ----------------- |
| **Development**|                    |                   |
| 10.51.0.0/16   | LabVpcDev          | Peering           |
| 10.52.0.0/16   | WorkspaceVpcDev    | Peering           |
| 10.53.0.0/16   | MSVpcDev           | Peering           |
| 10.54.0.0/16   | OmniSpinVpcDev     | Peering           |
| 10.55.0.0/16   | SGLabVpcDev        | VPC               |
| **Production** |                    |                   |
| 10.0.0.0/16    | LabVpcProd         | Peering           |
| 10.1.0.0/16    | WorkspaceVpcProd   | Peering           |
| 10.2.0.0/16    | MSVpcProd          | Peering           |
| 10.3.0.0/16    | OmniSpinVpcProd    | Peering           |
| 10.4.0.0/16    | SGLabVpcProd       | VPC               |
| **Common**     |                    |                   |
| 10.72.0.0/16   | VpnVpc             | Peering           |


#### ir-tgw (ASN:64515) Route Table

| Route          | Destination        | Destination Type |
| -------------- | ------------------ | ----------------- |
| **Development**|                    |                   |
| 10.51.0.0/16   | LabVpcDev          | Peering           |
| 10.52.0.0/16   | WorkspaceVpcDev    | Peering           |
| 10.53.0.0/16   | MSVpcDev           | Peering           |
| 10.54.0.0/16   | OmniSpinVpcDev     | Peering           |
| 10.56.0.0/16   | IRLabVpcDev        | VPC               |
| **Production** |                    |                   |
| 10.0.0.0/16    | LabVpcProd         | Peering           |
| 10.1.0.0/16    | WorkspaceVpcProd   | Peering           |
| 10.2.0.0/16    | MSVpcProd          | Peering           |
| 10.3.0.0/16    | OmniSpinVpcProd    | Peering           |
| 10.5.0.0/16    | IRLabVpcProd       | VPC               |
| **Common**     |                    |                   |
| 10.72.0.0/16   | VpnVpc             | Peering           |


#### tgw-vcl (ASN:64513) Route Table

| Route          | Destination        | Destination Type |
| -------------- | ------------------ | ----------------- |
| **Development**|                    |                   |
| 10.51.0.0/16   | LabVpcDev          | Peering           |
| 10.52.0.0/16   | WorkspaceVpcDev    | Peering           |
| 10.53.0.0/16   | MSVpcDev           | Peering           |
| 10.54.0.0/16   | OmniSpinVpcDev     | Peering           |
| **Production** |                    |                   |
| 10.0.0.0/16    | LabVpcProd         | Peering           |
| 10.1.0.0/16    | WorkspaceVpcProd   | Peering           |
| 10.2.0.0/16    | MSVpcProd          | Peering           |
| 10.3.0.0/16    | OmniSpinVpcProd    | Peering           |
| **Common**     |                    |                   |
| 10.72.0.0/16   | VpnVpc             | Peering           |
| 10.71.0.0/16   | VCLVpc             | VPC               |