---
title: What Is Microsoft Entra Domain Services?
description: Learn how Microsoft Entra Domain Services provides managed domain services like LDAP, Kerberos, and Group Policy without deploying domain controllers—ideal for hybrid and legacy application scenarios.
date: 2025-08-05
image: /images/entra-id-server.png
tags: [Entra DS, Identity, Hybrid]
---

# What Is Microsoft Entra Domain Services?

Microsoft Entra Domain Services (Entra DS) provides managed domain services such as domain join, group policy, LDAP, and Kerberos/NTLM authentication—without the need to deploy, manage, or patch domain controllers in the cloud. It bridges the gap between modern cloud identity (Microsoft Entra ID, formerly Azure AD) and legacy on-premises applications that rely on traditional Active Directory APIs and protocols.

This article explains what Microsoft Entra Domain Services is, how it works, and how to set it up in your Azure environment. We'll focus on practical implementation, architectural considerations, and common use cases.

## Understanding Microsoft Entra Domain Services

### What Is Microsoft Entra Domain Services?

Microsoft Entra Domain Services is a fully managed domain service hosted in Microsoft Azure. It provides core capabilities traditionally offered by Windows Server Active Directory, but without the need to deploy, manage, or patch domain controllers yourself.

Entra Domain Services supports legacy protocols and features, including **Lightweight Directory Access Protocol (LDAP)**, **Kerberos**, and **NT LAN Manager (NTLM)** authentication. These are commonly used by older applications that rely on traditional Active Directory environments. It also enables **Group Policy Objects (GPOs)** for centralized configuration management and allows virtual machines in Azure to be **domain-joined**, just as they would be in an on-premises Active Directory domain.

Importantly, Entra Domain Services integrates directly with your existing Microsoft Entra ID (formerly Azure Active Directory) users and groups. This means that accounts already managed in Microsoft Entra ID can seamlessly authenticate to resources that depend on legacy authentication protocols, without requiring separate identity management.

### Key Differences vs. Azure Active Directory

While both Microsoft Entra Domain Services and Azure Active Directory (Azure AD) are identity services offered in Azure, they serve different purposes.

Azure AD is designed for **modern authentication** scenarios, such as applications that use **OAuth 2.0** or **OpenID Connect**. It supports cloud-native models, including single sign-on (SSO) to SaaS applications, device management, and conditional access policies.

In contrast, Microsoft Entra Domain Services is built for **legacy protocol support**. It enables applications that depend on **Kerberos**, **LDAP**, or **NTLM** to continue functioning in a cloud environment. Unlike traditional Active Directory, Entra Domain Services does not require you to deploy or manage **domain controllers**—it is a fully managed service, reducing administrative overhead.

### Use Cases

Microsoft Entra Domain Services is particularly useful in scenarios where legacy application compatibility is required in a cloud-first or hybrid environment.

One common use case is the **lift-and-shift** migration of legacy, on-premises applications to Azure. These applications often rely on domain join, LDAP, or Kerberos for user authentication. Entra Domain Services enables them to operate in Azure without significant modification.

It is also suitable for applications that require **LDAP or Kerberos integration** but are being hosted in Azure. This includes internal line-of-business applications that were not designed with modern identity protocols in mind.

In **hybrid environments**, where organizations use Microsoft Entra ID for cloud-based identity management but still maintain applications that rely on legacy protocols, Entra Domain Services provides a bridge between the two worlds.

Finally, it supports **centralized identity management** in multi-tiered application architectures, allowing both modern and legacy components to authenticate against a common directory infrastructure.

## Architecture & Service Features

### Core Features

Entra Domain Services (Entra DS) provides a set of managed domain capabilities designed to support legacy authentication protocols and directory-aware applications within modern Azure environments. One of the primary features is the ability to **domain-join Azure virtual machines (VMs)** directly to the managed domain. This enables centralized identity management and policy enforcement without deploying or managing domain controllers.

Administrators can apply **Group Policy** to enforce security baselines, configuration settings, and operational controls across domain-joined VMs. These policies are compatible with traditional Active Directory environments, making migration or hybrid configurations more straightforward.

Entra DS supports **Lightweight Directory Access Protocol (LDAP) read access**, which allows applications and services to query directory information such as user attributes or group memberships. This is particularly useful for legacy apps that depend on directory lookups.

For authentication, Entra DS includes support for **Kerberos** and **NT LAN Manager (NTLM)**. These protocols enable secure authentication workflows for applications and services that do not support modern OAuth or SAML-based mechanisms.

To support encrypted directory access, Entra DS also offers **Secure LDAP (LDAPS)**. This ensures sensitive information is transmitted over encrypted connections, aligning with enterprise security requirements.

Finally, Entra DS is designed for **high availability**. When deployed in regions that support Azure Availability Zones, the service spans across multiple zones to provide resilience against zone-level failures.

### How It Works

Entra DS operates by synchronizing identity information from **Microsoft Entra ID** (formerly Azure Active Directory). This includes users, groups, and credential data. The synchronization process includes **password hashes**, which are transferred using a secure, one-way hashing mechanism to preserve credential confidentiality.

Once provisioned, Entra DS exposes a managed domain within a specified **Azure virtual network (VNet)**. This enables domain-join operations and directory service access to resources within the network. The domain is **region-bound**, meaning it is tied to the Azure region in which it is created and cannot be moved to another region post-deployment. This constraint should be factored into network and resource planning.

### Security Considerations

Entra DS enforces a strict **no administrative access** policy to the underlying domain controllers. This means customers cannot log in to or manage the domain controllers directly, reducing the attack surface and simplifying operational responsibility.

The service also restricts **schema modification** capabilities. While standard attributes and object types are available, extending the schema is not supported. This limitation ensures consistency and stability across managed domains.

To protect directory services from unauthorized network access, it's critical to configure **Network Security Groups (NSGs)** or **Azure Firewall** rules. These controls allow administrators to define which IP ranges, subnets, or services can access the domain services, ensuring only trusted systems interact with the managed domain.

## Setting Up Entra Domain Services in Azure

Setting up Microsoft Entra Domain Services (Entra DS) in Azure allows you to provide domain join, group policy, and Lightweight Directory Access Protocol (LDAP) capabilities without deploying and managing domain controllers. The process requires several prerequisites and a series of configuration steps, which are detailed below.

### Prerequisites

Before beginning, ensure the following requirements are met:

* You need an active **Azure subscription** to provision resources.
* A **Microsoft Entra tenant** must be available, with directory synchronization enabled to replicate user credentials.
* At least one **user account** must have both Global Administrator and Contributor roles to perform setup and configuration tasks.
* An **Azure virtual network (VNet)** must exist in the same region where Entra DS will be deployed. The managed domain will be bound to this VNet.

### Step-by-Step Setup

#### Enable Password Hash Synchronization

Entra DS relies on password hash synchronization to allow users to authenticate. This must be enabled prior to domain creation. Without it, users will not be able to sign in to domain-joined machines or services.

Follow the official [Microsoft instructions](https://learn.microsoft.com/en-us/entra/identity/domain-services/overview) to enable this setting in your Entra ID Connect configuration.

#### Create a Managed Domain

In the Azure portal, navigate to **Microsoft Entra Domain Services** and select **Create**. You will be prompted to:

* Choose a domain name, such as `xyz.onmicrosoft.com` or a custom verified domain.
* Select or create a virtual network. The domain will be deployed into this VNet, and it must exist in the same Azure region.

Optionally, you may use ARM (Azure Resource Manager) templates for automated deployment.

#### Configure DNS Settings

Once the managed domain is provisioned, Azure assigns it internal IP addresses. Update your VNet’s DNS settings to point to these addresses. This step is critical: all virtual machines (VMs) that need to join the domain must use these DNS servers to resolve domain names correctly.

To update DNS settings:

* Go to the VNet resource.
* Under **DNS servers**, select **Custom** and enter the IP addresses provided by Entra DS.
* Restart the VMs or reconfigure their network interfaces to use the updated settings.

#### Enable Secure LDAP (if needed)

Secure LDAP (LDAPS) allows applications to perform LDAP queries over SSL. This is optional but often required for legacy applications.

To enable LDAPS:

* Upload a valid SSL certificate, either from a public certificate authority or an internal one.
* Configure network security group (NSG) rules or firewall policies to allow TCP port 636 from trusted sources.

#### Join Azure VMs to the Managed Domain

Once DNS is configured, VMs can be joined to the domain using the standard domain join process:

* Open **System Properties** on the VM.
* Under **Computer Name**, select **Change** and enter the domain name.
* Provide the credentials of a synchronized Entra ID user who is part of the managed domain.

#### Configure Group Policy

To manage computers and users in the domain, use Group Policy Objects (GPOs):

* Deploy a management VM that is domain-joined.
* Install **Group Policy Management Console** (`gpmc.msc`).
* From here, apply built-in security baselines or define custom policies.

Changes to GPOs will apply to all computers and users within the scope of the policies.

### Important Limitations

Entra Domain Services comes with several architectural constraints:

* Once created, a managed domain **cannot be renamed or moved** to a different region or VNet.
* **Forest trust relationships** with other domains are not supported.
* Advanced Active Directory features such as **schema extensions** and **Read-Only Domain Controllers (RODCs)** are not available.

These limitations should be considered during planning to ensure compatibility with your application and infrastructure requirements.

## Integration with Applications & Services

### Application Compatibility

Microsoft Entra Domain Services (Entra DS) supports many directory-aware applications without requiring modification. Entra DS provides **Lightweight Directory Access Protocol (LDAP)** support, **Kerberos-based authentication**, and **Windows Integrated Authentication**, making it compatible with software expecting a traditional Active Directory (AD) environment.

Applications that rely on LDAP for user lookups or authentication can query Entra DS in the same way they would query an on-premises AD. Similarly, services requiring Kerberos tickets to authenticate users—such as file shares or internal web applications—can function normally when joined to an Entra DS domain. Windows Integrated Authentication, often used in intranet web applications, also works seamlessly when the application server is domain-joined to Entra DS.

### Migrating Legacy Apps

For legacy applications that were originally built to depend on on-premises AD, Entra DS provides a migration path that avoids significant re-architecture. These applications can be **lifted and shifted** to **Azure virtual machines (VMs)** that are domain-joined to Entra DS. This approach allows the application to continue using familiar authentication protocols such as LDAP or Kerberos, without requiring **Active Directory Federation Services (AD FS)** or a hybrid AD setup.

After migrating the app to Azure, reconfiguration typically involves pointing the application's authentication settings to the Entra DS domain. Since Entra DS exposes the same schema and protocols as traditional AD, most applications can authenticate users and apply access controls without changes to the application logic.

### Identity Centralization

By integrating with Entra DS, applications benefit from centralized identity management. Users authenticate using their **Microsoft Entra credentials**, which are the same credentials used for other Microsoft cloud services. This reduces the number of identities that must be managed and synchronized across environments.

Access control can be implemented using **Entra ID groups**, which are synchronized automatically with Entra DS. This enables consistent role-based access management across cloud and on-premises environments. Applications integrated with Entra DS can also participate in Microsoft Entra’s application provisioning workflows, streamlining the management of user access to SaaS and line-of-business applications.

For advanced provisioning scenarios, including custom attribute mappings and logic, see Microsoft’s documentation on [Functions for Customizing Application Data](https://learn.microsoft.com/en-us/entra/identity/app-provisioning/functions-for-customizing-application-data).

## Cost & Performance Considerations

Understanding the cost and performance characteristics of Entra Domain Services is essential for planning and maintaining an efficient deployment. The service is designed to offer predictable pricing and scalable performance, but proper management is necessary to avoid unexpected charges and performance bottlenecks.

### Pricing Tiers

Microsoft Entra Domain Services uses a tiered pricing model based on the number of **directory objects**, which include users, groups, and devices. Each pricing tier supports a specific range of directory objects, and moving to a higher tier occurs automatically when object counts exceed the current tier’s threshold. This can lead to increased costs if not closely monitored.

All pricing tiers include a **99.9% service-level agreement (SLA)** for availability. This ensures high reliability for authentication and domain-related services. Updated pricing information is available on the [official pricing page](https://azure.microsoft.com/en-us/pricing/details/microsoft-entra-ds/).

### Performance Scaling

Entra Domain Services is engineered to scale automatically based on demand. It uses a pool of **backend domain controllers** that are load-balanced to distribute authentication and directory service traffic efficiently. This automatic scaling means that performance typically remains stable even during usage spikes, without requiring manual intervention.

For optimal performance, it is recommended to deploy virtual machines (VMs) and applications that rely on Entra Domain Services within the same **virtual network (VNet)**. Doing so minimizes network latency for authentication and directory queries, resulting in faster response times for dependent services.

### Budgeting Tips

To avoid unnecessary costs, organizations should proactively plan and manage their directory object usage. Since pricing tiers are based on object count, keeping the number of users, groups, and devices within a tier's limit can prevent automatic and potentially costly tier upgrades.

Azure Cost Management tools provide detailed billing insights and trend analyses, allowing teams to monitor usage and forecast expenses. Regularly reviewing these reports can help detect cost anomalies early.

Additionally, applying **network security groups (NSGs)** and configuring **firewall rules** to restrict traffic to only necessary resources can reduce load on the service. This not only helps control costs but also enhances security by limiting exposure to unnecessary network activity.

## Conclusion

Microsoft Entra Domain Services (Entra DS) provides a vital compatibility layer for organizations shifting from traditional on-premises setups to cloud-first architectures. It supports key Active Directory features—such as Kerberos authentication, LDAP, and Group Policy—without requiring the deployment or maintenance of domain controllers in the cloud.

This service is particularly useful in scenarios where legacy applications still depend on domain-bound authentication or where centralized policy enforcement is required within Azure-hosted environments. Entra DS integrates directly with Microsoft Entra ID, reducing administrative overhead and ensuring consistency across hybrid or cloud-only infrastructures.

By abstracting away the complexity of domain controller operations and offering managed support for legacy protocols, Entra DS allows IT teams to focus on modernization efforts without sacrificing compatibility or security.

For implementation details, refer to the [Microsoft Entra Domain Services tutorial](https://docs.azure.cn/en-us/entra/identity/domain-services/tutorial-create-instance).