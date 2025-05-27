---
title: Implementing Secure Network Infrastructure with Azure Network Security Groups (NSGs)
description: Learn how to secure your Azure cloud environments using Network Security Groups (NSGs), including architecture fundamentals, deployment strategies, monitoring, and best practices.
date: 2024-06-12
image: "placeholder.jpg"
tags: [Azure, Network Security, NSG, Cloud Security, Infrastructure, Zero Trust, Monitoring]
---

# Implementing Secure Network Infrastructure with Azure Network Security Groups (NSGs)

## Introduction

The modern cloud environment presents a rapidly evolving threat landscape, where attackers continuously adapt their techniques to exploit misconfigurations, exposed services, and overly permissive access controls. As organizations migrate critical workloads to the cloud, the attack surface expands, making it imperative to implement layered and proactive security measures.

Despite the availability of sophisticated identity-based and behavioral security mechanisms, network-level protection remains a cornerstone of cloud security architecture. Network security controls provide deterministic enforcement points, isolating traffic and reducing the blast radius of potential compromises. In Azure, this foundational protection is implemented through **Network Security Groups (NSGs)**.

Azure NSGs act as stateful packet filters, controlling inbound and outbound traffic to Azure resources within Virtual Networks (VNets). Administrators can define granular rules based on source/destination IP addresses, ports, and protocols, thereby enforcing least-privilege access at the network layer. Whether applied at the subnet or individual network interface level, NSGs offer a lightweight yet powerful mechanism to restrict access and monitor traffic flows.

The importance of this control is underscored by industry data. In 2023, 45% of companies experiencing network security incidents reported cloud servers as the point of origin ([Atmosera](https://www.atmosera.com/blog/what-are-azure-network-security-groups/?utm%5Fsource=openai)). This statistic highlights the necessity of implementing robust traffic filtering controls early in the cloud deployment lifecycle.

By integrating NSGs into your Azure network architecture, you establish a foundational security posture that supports more advanced protective measures while reducing your exposure to common cloud-native threats.

![](https://oaidalleapiprodscus.blob.core.windows.net/private/org-2h1hGiSpEV2xL1QmmetBTfNv/user-ugXgjn5iKT1karGVlJhSKvX2/img-fuSGAmm4GfCdDqbSeb5ISSih.png?st=2025-05-27T19%3A56%3A36Z&se=2025-05-27T21%3A56%3A36Z&sp=r&sv=2024-08-04&sr=b&rscd=inline&rsct=image/png&skoid=cc612491-d948-4d2e-9821-2683df3719f5&sktid=a48cca56-e6da-484e-a814-9c849652bcb3&skt=2025-05-27T15%3A49%3A33Z&ske=2025-05-28T15%3A49%3A33Z&sks=b&skv=2024-08-04&sig=2jprIK2ywy1pvOc1aj0/BSb1TXXwa5ax10/8gfp4PYQ%3D)

## Azure NSGs: Core Concepts and Architecture

### What is an NSG?

A **Network Security Group (NSG)** is a virtual firewall in Microsoft Azure that governs inbound and outbound traffic to networked Azure resources. It operates at the network layer and is designed to enforce **least privilege access** by limiting traffic based on defined rules.

NSGs can be associated with either a subnet or individual network interfaces (NICs). When associated with a subnet, the rules apply to all resources within that subnet. When associated with a NIC, the rules apply only to the specific virtual machine (VM) connected to that NIC.

### NSG Components

Each NSG consists of one or more **security rules**. These rules are defined using a **5-tuple** model:

* **Source IP address**
* **Source port range**
* **Destination IP address**
* **Destination port range**
* **Protocol** (TCP, UDP, or \* for any)

Each rule also includes a **priority** value. Lower numbers indicate higher priority. Azure processes rules in ascending order of priority until it finds a match. Once a match is found, no further rules are evaluated.

There are two types of rules:

* **Default rules**: Predefined by Azure to allow basic infrastructure communication (e.g., allowing VNet traffic and denying internet-sourced traffic).
* **Custom rules**: User-defined to meet specific security and access requirements. These override default rules if they have higher priority.

### NSG Rule Evaluation Logic

NSGs follow a deterministic evaluation path based on the direction of traffic.

#### Inbound Traffic

1. NSG rules are first evaluated at the **subnet level**, if an NSG is associated with the subnet.
2. Then, NSG rules are evaluated at the **NIC level**, if an NSG is associated with the NIC.

#### Outbound Traffic

1. NSG rules are evaluated first at the **NIC level**.
2. Then, NSG rules are evaluated at the **subnet level**.

If no rule matches the traffic after evaluation, Azure applies an **implicit deny** rule, which drops the traffic.

Reference: [Microsoft Learn – How Network Security Groups Filter Network Traffic](https://learn.microsoft.com/en-us/azure/virtual-network/network-security-group-how-it-works?utm%5Fsource=openai)

![](https://oaidalleapiprodscus.blob.core.windows.net/private/org-2h1hGiSpEV2xL1QmmetBTfNv/user-ugXgjn5iKT1karGVlJhSKvX2/img-274HMpCAUUi77MigRIFdf0gk.png?st=2025-05-27T19%3A56%3A56Z&se=2025-05-27T21%3A56%3A56Z&sp=r&sv=2024-08-04&sr=b&rscd=inline&rsct=image/png&skoid=cc612491-d948-4d2e-9821-2683df3719f5&sktid=a48cca56-e6da-484e-a814-9c849652bcb3&skt=2025-05-27T17%3A10%3A42Z&ske=2025-05-28T17%3A10%3A42Z&sks=b&skv=2024-08-04&sig=0mclIY03NO4F0U1lxOj52gkr50OeJjyqaKwk0Uu2/DE%3D)

## NSG Association and Deployment Strategies

### Where You Can Apply NSGs

Azure Network Security Groups (NSGs) can be associated with two primary scopes: **subnets** and **network interfaces (NICs)**. Understanding the difference in scope and impact is critical for designing secure and maintainable network architectures.

* **Subnets**: Applying an NSG at the subnet level controls traffic to all resources within the subnet. This is ideal for coarse-grained access control, such as restricting external access or enforcing internal segmentation policies across a tier (e.g., all application servers).
* **Network Interfaces (NICs)**: NSGs associated with individual NICs enable fine-grained control per virtual machine (VM). This is especially useful when specific VMs require exceptions to general subnet rules, or for implementing zero-trust-like policies within a shared subnet.

When both a subnet-level and NIC-level NSG are associated, Azure evaluates the effective rules as the intersection of both sets—only traffic allowed by both NSGs is permitted.

### Combining NSGs with Route Tables and Application Security Groups

NSGs are most effective when used in conjunction with other Azure network constructs:

* **Route Tables**: While NSGs control _whether_ traffic is allowed, route tables define _where_ traffic goes. For example, you might route all outbound traffic through a firewall VM, and use NSGs to ensure only traffic from trusted IP ranges reaches that VM.
* **Application Security Groups (ASGs)**: ASGs allow you to group VMs logically, independent of IP addressing or subnet boundaries. NSGs can then use ASGs as source or destination in rules, simplifying policy definition. For instance, rather than allowing traffic from a specific IP, you allow traffic from the "WebServers" ASG.

This combination supports scalable, readable network policies, especially in environments with dynamic or autoscaling workloads.

### Traffic Filtering Scenarios

#### Internet-facing Services

NSGs should be configured with a default-deny posture:

* Block all inbound traffic by default.
* Explicitly allow only necessary ports, such as TCP port 443 for HTTPS.
* Restrict access to administrative ports like SSH (22) or RDP (3389) using IP-based allow rules. For example, permit only known management IPs to access these services ([CoreStack](https://www.corestack.io/azure-security-tools/nsg-azure/?utm%5Fsource=openai)).

#### Internal Traffic Control

NSGs can also be used to segment internal traffic:

* Define rules that control traffic flow between application tiers, such as allowing traffic from web tier to app tier only on specific ports.
* Implement **microsegmentation** by using NSGs in combination with ASGs, enabling precise control over east-west traffic within the virtual network.

### Network Segmentation with NSGs

NSGs are a key component in enforcing **network segmentation**:

* Isolate environments such as development, staging, and production by assigning different NSGs to their respective subnets.
* Apply **defense-in-depth** by layering NSGs at both subnet and NIC levels.
* Follow the **principle of least privilege** by allowing only explicitly required traffic and denying all else ([Medium](https://medium.com/%40jdahunsi5/demystifying-azure-network-security-groups-nsgs-6e417c5a458a?utm%5Fsource=openai)).

This layered approach reduces the attack surface and improves security posture without introducing undue operational complexity.

![](https://oaidalleapiprodscus.blob.core.windows.net/private/org-2h1hGiSpEV2xL1QmmetBTfNv/user-ugXgjn5iKT1karGVlJhSKvX2/img-hJjQUF2OueBA6NIkI6RxpdtR.png?st=2025-05-27T19%3A57%3A20Z&se=2025-05-27T21%3A57%3A20Z&sp=r&sv=2024-08-04&sr=b&rscd=inline&rsct=image/png&skoid=cc612491-d948-4d2e-9821-2683df3719f5&sktid=a48cca56-e6da-484e-a814-9c849652bcb3&skt=2025-05-27T09%3A39%3A02Z&ske=2025-05-28T09%3A39%3A02Z&sks=b&skv=2024-08-04&sig=PwxsxH1Ce8wGKV9SqnwxcR3YjTk697STDjwvjCmtOH8%3D)

## Operationalizing NSGs: Monitoring, Analytics, and Governance

### Using Azure NSG Analytics

Azure Network Security Groups (NSGs) generate diagnostic data that can be ingested and analyzed using native Azure tools. Azure NSG flow logs, available through Network Watcher, provide detailed insights into allowed and denied traffic flows. This data is essential for identifying traffic patterns, diagnosing misconfigurations, and improving rule effectiveness.

By integrating NSG diagnostics with **Azure Monitor** and **Microsoft Defender for Cloud** (formerly Azure Security Center), administrators can:

* Track trends in allowed vs. denied traffic to detect anomalies or policy drift.
* Identify misconfigured or redundant rules that never match traffic.
* Leverage prebuilt dashboards and visualizations for actionable insights ([MobilesTalk](https://mobilestalk.net/azure-network-security-group-analytics/?utm%5Fsource=openai)).

These insights support both reactive troubleshooting and proactive security posture improvement.

### Best Practices for NSG Management

Effective NSG governance requires rule hygiene and architectural discipline. Recommended practices include:

* **Deny by default; allow by exception**: This principle ensures that only explicitly allowed traffic is permitted.
* **Minimize rule count**: Use Application Security Groups (ASGs) and service tags to reduce rule complexity and group similar endpoints logically.
* **Use augmented security rules**: These allow for multiple IPs and ports in a single rule, simplifying configurations and improving readability ([CoreStack](https://www.corestack.io/azure-security-tools/nsg-azure/?utm%5Fsource=openai)).
* **Document and audit**: Maintain up-to-date documentation of NSG rule intent and regularly audit rule sets for relevance, redundancy, and compliance.

### Common Pitfalls and Misconfigurations

Several common issues can compromise NSG effectiveness:

* **Overly permissive rules**: For example, allowing all inbound traffic (`*` source, `*` destination port) exposes workloads unnecessarily.
* **Incorrect rule prioritization**: NSG rules are processed in order of priority (lower numbers first). A misordered allow rule can override a more restrictive deny rule.
* **Rule conflicts between subnet and NIC levels**: NSGs can be applied at both levels, and the effective policy is the combination. Conflicting rules can result in unintended access or blocked traffic.

Regular validation and the use of policy-as-code strategies can mitigate these risks and support more robust governance workflows.

![](https://oaidalleapiprodscus.blob.core.windows.net/private/org-2h1hGiSpEV2xL1QmmetBTfNv/user-ugXgjn5iKT1karGVlJhSKvX2/img-3fP4uvF1hvYBPdCb7eJizyH9.png?st=2025-05-27T19%3A57%3A38Z&se=2025-05-27T21%3A57%3A38Z&sp=r&sv=2024-08-04&sr=b&rscd=inline&rsct=image/png&skoid=cc612491-d948-4d2e-9821-2683df3719f5&sktid=a48cca56-e6da-484e-a814-9c849652bcb3&skt=2025-05-27T17%3A55%3A39Z&ske=2025-05-28T17%3A55%3A39Z&sks=b&skv=2024-08-04&sig=tCu9hSidhkFiTjG0xZQZFwQLsZU1r8LjGRHtfGjOMmY%3D)

## Implementation Walkthroughs & Next Steps

### Step-by-Step: Creating & Applying an NSG

Implementing a Network Security Group (NSG) in Azure involves a clear sequence of tasks. The following steps assume a working familiarity with the Azure portal, Azure CLI, or infrastructure-as-code tools such as ARM templates.

1. **Define Security Rules (Least Privilege First)**  
Begin by identifying your application's traffic flows and apply the **principle of least privilege**. That means only allowing traffic that's explicitly required, and denying everything else by default. Each rule consists of:
* Priority (lower values = higher priority)
* Source & destination (IP ranges, Application Security Groups (ASGs), etc)
* Protocol (TCP, UDP, or \* for all)
* Port range
* Direction (inbound or outbound)
* Action (allow or deny)
1. **Create NSG**  
You can create an NSG using several methods:  
**Azure Portal:**
* Navigate to "Network Security Groups"
* Click "+ Create"
* Configure the NSG name, subscription, resource group, and region  
**Azure CLI:**

```bash
   az network nsg create \
     --resource-group MyResourceGroup \
     --name MyNSG \
     --location eastus

```

**ARM Template:** Include a `Microsoft.Network/networkSecurityGroups` resource block with rule definitions.

1. **Associate NSG to Subnet and/or NIC**  
NSGs can be associated with **subnets** (for all resources in the subnet) or **Network Interface Cards (NICs)** (for individual VMs). The effective rules are a union of subnet-level and NIC-level NSGs.  
Azure CLI example:

```bash
   az network vnet subnet update \
     --resource-group MyResourceGroup \
     --vnet-name MyVNet \
     --name MySubnet \
     --network-security-group MyNSG

```

1. **Validate Rule Behavior with NSG Flow Logs**  
Enable **NSG flow logs** to capture traffic metadata. This helps confirm whether rules are having their intended effect.  
Azure CLI example:

```bash
   az network watcher flow-log configure \
     --resource-group MyResourceGroup \
     --nsg MyNSG \
     --enabled true \
     --retention 7 \
     --storage-account MyStorageAccount

```

Use **Azure Network Watcher** or third-party analytics tools to visualize and analyze flow logs.

### Advanced Use Cases

#### NSGs with Azure Firewall

NSGs and Azure Firewall serve complementary purposes:

* **NSGs**: Operate at the OSI network layer (Layer 3 & 4). Best for access control per subnet or NIC.
* **Azure Firewall**: Works at Layer 7 (application layer), enabling filtering by domain names, TLS inspection, and logging.

In practice, NSGs restrict traffic within the virtual network, while Azure Firewall governs traffic entering and exiting the network.

#### NSGs in Hub-and-Spoke Architectures

In a **hub-and-spoke topology**, NSGs are typically applied to:

* The spoke subnets, to limit lateral movement between workloads
* The hub's VPN or ExpressRoute gateway subnet, to control ingress/egress

Use NSG rules to enforce traffic flow policies between spokes via the hub, and restrict direct spoke-to-spoke communication unless explicitly required.

#### NSG Integration with Azure Policy

Azure Policy can enforce the existence of NSGs and specific rule configurations. Common scenarios include:

* Ensuring all subnets have an NSG attached
* Denying inbound RDP (TCP/3389) from the internet

Example Azure Policy definition:

```json
{
  "if": {
    "allOf": [
      {
        "field": "Microsoft.Network/virtualNetworks/subnets/networkSecurityGroup.id",
        "exists": "false"
      }
    ]
  },
  "then": {
    "effect": "deny"
  }
}

```

### Learning Resources & Documentation

* [Microsoft Learn](https://learn.microsoft.com/en-us/azure/virtual-network/network-security-group-how-it-works?utm%5Fsource=openai)
* [Atmosera NSG Overview](https://www.atmosera.com/blog/what-are-azure-network-security-groups/?utm%5Fsource=openai)
* [MobilesTalk: NSG Analytics](https://mobilestalk.net/azure-network-security-group-analytics/?utm%5Fsource=openai)
* [CoreStack Security Best Practices](https://www.corestack.io/azure-security-tools/nsg-azure/?utm%5Fsource=openai)

> "Security is not a product, but a process." — Bruce Schneier

![](https://oaidalleapiprodscus.blob.core.windows.net/private/org-2h1hGiSpEV2xL1QmmetBTfNv/user-ugXgjn5iKT1karGVlJhSKvX2/img-FghSr0QOsIsFLlZRvi9Hl473.png?st=2025-05-27T19%3A57%3A57Z&se=2025-05-27T21%3A57%3A57Z&sp=r&sv=2024-08-04&sr=b&rscd=inline&rsct=image/png&skoid=cc612491-d948-4d2e-9821-2683df3719f5&sktid=a48cca56-e6da-484e-a814-9c849652bcb3&skt=2025-05-27T17%3A18%3A52Z&ske=2025-05-28T17%3A18%3A52Z&sks=b&skv=2024-08-04&sig=OH%2BNjw7fiXXDViOhfoW99nPCBy3PSS%2Bb1AyZgVPNWbE%3D)