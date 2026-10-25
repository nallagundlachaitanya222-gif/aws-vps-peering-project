# AWS VPC Peering Project - EWB Cloud Labs

- **Student Name:** Chaitanya Nallagundla
- **Region:** ap-south-1 (Mumbai)
- **Project Goal:** Design and document a secure VPC Peering architecture connecting two isolated Virtual Private Clouds (VPC A and VPC B), configuring routing tables, security groups, a public Bastion host, and private application instances.

---

## 1. Architecture & IP Addressing Plan

### VPC A (`ewb-vpc-a`)
- **CIDR Block:** `10.10.0.0/16`
- **Public Subnet (`ewb-a-public`):** `10.10.1.0/24`
  - *Bastion Host IP:* `10.10.1.10`
- **Private Subnet (`ewb-a-private`):** `10.10.2.0/24`
  - *Application Instance A IP:* `10.10.2.10`

### VPC B (`ewb-vpc-b`)
- **CIDR Block:** `10.20.0.0/16`
- **Private Subnet (`ewb-b-private`):** `10.20.1.0/24`
  - *Application Instance B IP:* `10.20.1.10`

---

## 2. Step-by-Step Implementation Guide

### Step 1: VPC and Subnet Creation
1. Created **VPC A** with CIDR `10.10.0.0/16` and **VPC B** with CIDR `10.20.0.0/16`.
2. Created an Internet Gateway (`igw-01`) and attached it to VPC A to enable internet connectivity for the public subnet.
3. Configured subnets:
   - Public Subnet in VPC A (`10.10.1.0/24`) linked to a public route table with a route `0.0.0.0/0 -> igw-01`.
   - Private Subnet in VPC A (`10.10.2.0/24`) and Private Subnet in VPC B (`10.20.1.0/24`) linked to private route tables.

### Step 2: VPC Peering Connection Setup
1. Initiated a VPC Peering Connection (`pcx-xxxxxx`) from **VPC A (Requester)** to **VPC B (Accepter)**.
2. Accepted the peering request from VPC B's console side.
3. Updated Route Tables to enable cross-VPC communication:
   - **VPC A Private Route Table:** Added route `10.20.0.0/16` -> Target: Peering Connection (`pcx-xxxxxx`).
   - **VPC B Private Route Table:** Added route `10.10.0.0/16` -> Target: Peering Connection (`pcx-xxxxxx`).

### Step 3: Security Groups & Instances
1. **Bastion Security Group (VPC A):** Allowed inbound SSH (`Port 22`) from `0.0.0.0/0`.
2. **App-A Security Group (VPC A Private):** Allowed inbound SSH (`Port 22`) and ICMP (Ping) strictly from the Bastion host and VPC B CIDR (`10.20.0.0/16`).
3. **App-B Security Group (VPC B Private):** Allowed inbound SSH (`Port 22`) and ICMP (Ping) strictly from VPC A CIDR (`10.10.0.0/16`).
4. Launched an EC2 Bastion instance in the public subnet of VPC A, and private EC2 instances (`App-A` and `App-B`) in their respective private subnets.

---

## 3. Network Validation & Testing Plan

- **Baseline Test (Pre-Peering):** Pinging from App-A (`10.10.2.10`) to App-B (`10.20.1.10`) results in `100% packet loss` due to network isolation.
- **Post-Peering Test:** Once the peering connection and route tables are active, pinging from App-A to App-B yields `0% packet loss`.
- **Reverse Path Test:** Pinging from App-B back to App-A confirms bidirectional connectivity.
- **Secure Access:** Connecting to private instances securely via SSH using the Bastion host as a proxy (`ProxyJump`).
