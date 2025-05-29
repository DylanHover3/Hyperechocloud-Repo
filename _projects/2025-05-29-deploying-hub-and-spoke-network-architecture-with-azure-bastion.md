---
title: Hub-and-Spoke Network in Azure Using Bicep
description: Learn how to deploy a secure, scalable hub-and-spoke network topology in Azure using Bicep with integrated Bastion access, Private DNS, and spoke VM provisioning.
date: 2025-05-29
image: "/images/Hub-n-Spoke-Topology.jpg"
label: Network Connectivity
featured: true
---

# Deploying a Bastion-Backed Hub-and-Spoke Network in Azure Using Bicep

In this article, I cover the architecture and function of a **hub-and-spoke Azure Virtual Network (VNet) topology** defined in a Bicep configuration built for proof-of-concept (POC) scenarios. The configuration deploys a centralized networking pattern with connectivity, security, and remote access control for internal virtual machines across multiple virtual networks.

This deployment is particularly suited for controlled sandbox testing of inter-network connectivity, identity-aware access via Azure Bastion, and scalable DNS integration using a private zone model.

---

## Purpose of the Deployment

The core objective of this Bicep configuration is to provision a baseline **Bastion-enabled hub-and-spoke network topology** tailored for experimentation or demonstration. It includes key components necessary for secure, segmented, and manageable VNet designs, and isolates cloud resources for clear network architectural boundaries—critical for scalable enterprise network strategies.

This deployment achieves:

- Secure, browser-based RDP/SSH access via Azure Bastion without public IPs
- Configurable number of _spoke VNets_, each hosting a Windows VM
- Private DNS integration with automatic/resource-aware registration
- Fine-grained network security rules pinned to each spoke
- Simplified scaling for additional spoke networks via parameter tuning

---

## Major Components

### 1\. **Hub VNet with Bastion Host**

The **hub virtual network ("Hub1")** represents the central point of network control. It includes:

- A **Bastion host** in a subnet named `AzureBastionSubnet`, allowing secure inbound access to VMs in the hub or spokes via the Azure Portal without needing public IPs.
- A second subnet (`Subnet-2`) hosting a VM for validation purposes; protected with an NSG that allows RDP from the Bastion subnet.

Azure Bastion's tight subnet and IP constraints are well-handled here, avoiding a common deployment failure due to address misalignment between the VNet and the Bastion subnet.

### 2\. **Spoke VNets and VMs**

A variable number (default: 3) of **spoke VNets** are deployed, each with:

- A unique address space (`10.1.0.0/16`, `10.2.0.0/16`, etc)
- A subnet (`Subnet-1`) bound to its own Network Security Group (NSG)
- A Windows VM configured with Trusted Launch (if enabled) and diagnostic settings

Each spoke’s network is named predictably (`SpokeNetwork1`, `SpokeNetwork2`, etc), allowing scalable testing scenarios and consistent scripting behavior when integrating with CI/CD pipelines or Infrastructure as Code (IaC) workflows.

### 3\. **Network Peering: Hub-and-Spoke Connectivity**

Each spoke VNet is **peered bidirectionally with the hub**, enabling full network access and traffic forwarding between them:

- Hub → Spoke Peering: `hubNetwork-to-SpokeNetworkX`
- Spoke → Hub Peering: `SpokeX-to-Hub1`

These connections enable a simulated enterprise backbone, where the hub can host shared services (e.g., logging, security tools) accessed securely by spoke resources.

### 4\. **Private DNS Zone: hyperechocloud.com**

Instead of relying on Azure-provided DNS or static hostnames, a centralized **Private DNS Zone** (`hyperechocloud.com`) is created and linked to all VNets.

Each VNet has an associated `virtualNetworkLink`, enabling **per-network DNS registration** (`registrationEnabled: true`). This allows Azure VMs to automatically register their hostnames into the private DNS zone, improving service discovery for applications operating within this network fabric.

Though modest, this is a powerful enterprise pattern, commonly used in hybrid DNS extension architectures (i.e., when combined with Azure DNS Resolver traffic forwarding or on-prem DNS conditional forwarders).

### 5\. **Application Security Groups (ASGs)**

The deployment automatically provisions one **Application Security Group** per spoke VM (`Allow-RDM-SpokeX-VM`). Though not strictly necessary for basic RDP access via Bastion (already controlled via NSGs), this design enables _application-centric security policies_ to be layered on top in subsequent extensions.

Compared to traditional IP-centric network rules, ASGs allow security configurations to reference named resource groups instead of specific IPs—a useful abstraction in dynamic environments.

---

## Key Features & Observations

- **Scalability by Design**: The `spokeNetworkCount` parameter controls the proliferation of spokes, eliminating the need for manual replication in testing diverse VNets.
- **Security-centric Defaults**: NSGs block anything except RDP from the Bastion range; Trusted Launch is supported for UEFI/TPM-secured boot landscapes.
- **Region-agnostic**: The configuration uses `resourceGroup().location` by default, ensuring location consistency without manual override.
- **Cloud Native Service Composition**: Does not depend on custom scripts or third-party modules—adheres strictly to Azure-native Bicep/ARM constructs.

---

## Potential Extensions

This hub-and-spoke configuration offers a strong base, but buildup opportunities abound:

- Add **Azure Firewall or Azure Route Server** in the hub for advanced control.
- Configure **custom DNS forwarding/resolution** scenarios with Azure DNS Resolver.
- Integrate **Azure Policy** for compliance or tagging.
- Implement **Log Analytics agents** on each VM for diagnostics.

---

## Deployment Context

This is intended primarily for **sandbox, proof-of-concept, or educational environments**. It balances production-grade patterns (Bastion + DNS + NSGs + peering) with scriptable flexibility (parameterized spoke count, dynamic names). Operational hardening—including automatic updates, identity integration, and backup strategies—would need to be layered on for long-lived or regulated production use.

---

## 📁 Infrastructure Source

Ready to deploy this hub-and-spoke architecture? The complete Bicep template is available for download below:

### 🚀 Download the Template

**[📥 Download main.bicep](/cloudbuilds/HubnSpoke_main.bicep)**

#### **💡 Quick Start:** Once downloaded, you can deploy this template using Azure CLI with:

```bash
 `az deployment group create --resource-group <your-rg> --template-file main.bicep`
```

---

## Summary

This Bicep deployment defines and deploys a coherent and scalable **hub-and-spoke network architecture** in Azure with:

- Secure and credential-less access via Azure Bastion
- Private DNS zone integration for automated name resolution
- Application and network-level segmentation
- Multi-spoke VNet support with minimal config duplication

Operators exploring mesh alternatives, zero trust expansion, or hybrid DNS strategies will find this a solid base on which to iterate.
