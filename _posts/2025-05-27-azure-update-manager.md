---
title: Updating Azure Virtual Machines with Azure Update Manager
description: Learn how to configure, deploy, and monitor automated patch management for Azure and hybrid VMs using Azure Update Manager, with best practices on compliance, hotpatching, and update automation.
date: 2025-05-27
image: "/images/placeholder value"
tags: [Azure Update Manager, Patch Management, Virtual Machines, Azure Arc, Hotpatching, Compliance, Automation]
---

# Updating Azure Virtual Machines with Azure Update Manager

Timely patching of virtual machines (VMs) is a cornerstone of cloud infrastructure security. With cyberattacks increasingly exploiting known software vulnerabilities, automated update management becomes not just a convenience but a necessity. Azure Update Manager provides a centralized, cross-platform solution to orchestrate updates across Azure and hybrid environments. This guide provides a comprehensive overview of configuring, deploying, and monitoring updates with Azure Update Manager—while emphasizing best practices in vulnerability patching and system availability.

## Understanding the Risk Landscape and Why Patch Management Matters

Patch management is a foundational element of any organization's cybersecurity strategy. Unpatched software creates openings for attackers, and adversaries are increasingly exploiting known vulnerabilities faster and more aggressively.

> "60% of cybersecurity breaches are caused by unpatched vulnerabilities" — [Gitnux](https://gitnux.org/patch-management-statistics/?utm%5Fsource=openai)

### The Impact of Delayed Patching

Timeliness is critical. A significant portion of vulnerabilities are not zero-days—they are known issues for which patches already exist. The delay between patch availability and deployment provides attackers with a window of opportunity.

* **42% of vulnerabilities are exploited after a patch is released**, indicating that attackers actively monitor patch releases and reverse-engineer fixes to craft exploits ([Micron21](https://www.micron21.com/blog/42-of-vulnerabilities-exploited-after-patch-already-released---live-patching-helps-keep-you-secure?utm%5Fsource=openai)).
* **32% of cyberattacks target outdated software**, suggesting that legacy systems and slow update cycles are high-risk vectors ([arxiv.org](https://arxiv.org/abs/2505.13922?utm%5Fsource=openai)).

### The Case for Automation

Manual patching is prone to delays, inconsistencies, and errors. Automation mitigates these risks by ensuring patches are applied uniformly and quickly across environments.

* **Automated patching leads to up to 70% faster patch deployment**, significantly narrowing the attacker’s window ([Gitnux](https://gitnux.org/patch-management-statistics/?utm%5Fsource=openai)).
* Automation also **reduces human error** and **standardizes update workflows**, leading to greater operational stability and compliance.

### Azure’s Role in the Modern Enterprise

Azure Update Manager provides a centralized framework for managing server updates at scale:

* **Centralized control of VM updates** across Windows and Linux platforms simplifies operational overhead.
* **Integration with Azure Arc** extends patch management capabilities to on-premises and multi-cloud environments, enabling consistent policy enforcement regardless of where workloads reside.

Together, these capabilities make Azure a strategic platform for enterprises seeking to reduce their attack surface and improve patch compliance through automation and visibility.

![](https://oaidalleapiprodscus.blob.core.windows.net/private/org-2h1hGiSpEV2xL1QmmetBTfNv/user-ugXgjn5iKT1karGVlJhSKvX2/img-DfZRaf3LMo4Q8whrrBMur6vd.png?st=2025-05-27T22%3A47%3A25Z&se=2025-05-28T00%3A47%3A25Z&sp=r&sv=2024-08-04&sr=b&rscd=inline&rsct=image/png&skoid=cc612491-d948-4d2e-9821-2683df3719f5&sktid=a48cca56-e6da-484e-a814-9c849652bcb3&skt=2025-05-27T15%3A52%3A39Z&ske=2025-05-28T15%3A52%3A39Z&sks=b&skv=2024-08-04&sig=JjrWimXdjEAT7J6gHdUCwYq14vCfyoPKZPHx8UGsqBE%3D)

## Enabling and Configuring Azure Update Manager

### Prerequisites

Before configuring Azure Update Manager, ensure the following prerequisites are met:

* **Azure VM or Arc-enabled server**: The target machine must be an Azure virtual machine or an on-premises/other cloud server connected via Azure Arc.
* **Azure Monitor Agent**: The Azure Monitor Agent (AMA) must be installed on each machine to collect update data and enable patching actions.
* **Role-Based Access Control (RBAC)**: The user performing configuration must have at least the **Contributor** role on the target scope (subscription, resource group, or VM level). For finer control, custom roles with specific Update Management actions can be defined.

### Initial Setup

To begin using Azure Update Manager:

1. Open the **Azure Portal**.
2. Search for and navigate to **Azure Update Manager**.
3. Select **Machines** and identify the VM(s) or Arc-enabled server(s) you want to onboard.
4. Click **Enable Azure Update Manager**. This action configures the selected machines for update management.
5. Review and confirm the installation of the Azure Monitor Agent if it is not already present. Machines must have AMA deployed for Update Manager to function.

You can enable update management:

* **Individually**: Select and onboard a single VM or server.
* **At scale**: Use Azure Policy or bulk selection to onboard multiple machines across subscriptions or resource groups.

### Grouping Resources with Update Configurations

Azure Update Manager allows administrators to define **update configurations**, which group machines for consistent update behavior.

* **Scopes**: Define the scope of an update configuration using:
* **Subscription**
* **Resource Group**
* **Tags** (e.g., `Environment=Prod`, `OS=Linux`)
* **Dynamic Scoping**: Update configurations can use **dynamic scoping**, which automatically includes new machines that match the configuration criteria (e.g., machines with a specific tag or in a particular resource group). This ensures that newly provisioned or re-tagged machines are automatically managed without manual intervention.

Example:

```json
{
  "scope": {
    "subscriptions": ["/subscriptions/abc123"],
    "tags": {"Environment": ["Production"]}
  },
  "updateSettings": {
    "rebootSetting": "IfRequired",
    "updateClassification": ["Security", "Critical"]
  }
}

```

This configuration targets all production machines in a specific subscription and applies only security and critical updates, rebooting only if needed.

![](https://oaidalleapiprodscus.blob.core.windows.net/private/org-2h1hGiSpEV2xL1QmmetBTfNv/user-ugXgjn5iKT1karGVlJhSKvX2/img-h9LrBAL3nes2d2Ku5GGXuENt.png?st=2025-05-27T22%3A47%3A45Z&se=2025-05-28T00%3A47%3A45Z&sp=r&sv=2024-08-04&sr=b&rscd=inline&rsct=image/png&skoid=cc612491-d948-4d2e-9821-2683df3719f5&sktid=a48cca56-e6da-484e-a814-9c849652bcb3&skt=2025-05-27T21%3A12%3A37Z&ske=2025-05-28T21%3A12%3A37Z&sks=b&skv=2024-08-04&sig=kLZISwICaLqBCbU0ShX4q3mugs%2BMhSffRdW3oK4t3g0%3D)

## Assessing VM Update Compliance and Scheduling Deployments

### Reviewing Update Assessments

To evaluate the update status of your virtual machines (VMs), use the **Compliance** tab in Azure Update Manager. This view provides a breakdown of missing updates by:

* **Severity**: Highlights updates categorized as critical, important, or optional.
* **Type**: Differentiates between security, quality, and feature updates.
* **Age**: Shows how long each update has been pending.

You can filter the compliance data based on:

* **Operating System (OS)**: Focus on specific OS families such as Windows or Linux.
* **Patch Classification**: Narrow your view by update categories like security, critical, or other.

This allows for granular targeting of remediation efforts, especially useful for prioritizing high-risk vulnerabilities.

### Creating Update Deployments

Azure Update Manager supports both **recurring** and **one-time** update deployments. When creating a deployment, you can specify:

* **Patch Categories**: Select which types of updates to apply, such as security-only or include all quality updates.
* **Reboot Behavior**:
* _Always reboot_ after updates.
* _Never reboot_ (not recommended for critical patches).
* _Reboot if required_ (default behavior).
* **Pre/Post Scripts**: Define scripts to run before or after update application. This is useful for preparing workloads or validating system health post-patch.

Target machines can be selected using:

* **Saved Scopes**: Predefined groups of machines based on tags or resource groups.
* **Custom Queries**: Use Azure Resource Graph queries for dynamic targeting.

### Hotpatching and Reboot Considerations

A recent enhancement to Azure Update Manager is support for **Hotpatching** on **Arc-enabled servers** ([Microsoft Docs](https://learn.microsoft.com/en-us/azure/update-manager/whats-new?utm%5Fsource=openai)).

Hotpatching enables the application of critical security updates **without requiring a reboot**. This is particularly beneficial in environments with strict **Service-Level Agreements (SLAs)** or high availability requirements.

Key considerations:

* Hotpatching is currently available on specific Windows Server versions.
* It significantly reduces downtime, but not all updates qualify for hotpatching.
* When planning deployments, verify hotpatch eligibility to minimize disruption.

Use hotpatching in conjunction with compliance assessments and targeted deployments to implement a resilient and minimally disruptive patching strategy.

![](https://oaidalleapiprodscus.blob.core.windows.net/private/org-2h1hGiSpEV2xL1QmmetBTfNv/user-ugXgjn5iKT1karGVlJhSKvX2/img-BZ0edqj8HKlIWhNnGSlzLZDI.png?st=2025-05-27T22%3A48%3A01Z&se=2025-05-28T00%3A48%3A01Z&sp=r&sv=2024-08-04&sr=b&rscd=inline&rsct=image/png&skoid=cc612491-d948-4d2e-9821-2683df3719f5&sktid=a48cca56-e6da-484e-a814-9c849652bcb3&skt=2025-05-27T13%3A19%3A05Z&ske=2025-05-28T13%3A19%3A05Z&sks=b&skv=2024-08-04&sig=5gcUHuTDQqvQoYpHG88nJwWTEfZYpXomDHmuHQyzc30%3D)

## Monitoring and Troubleshooting Update Deployments

### Viewing Deployment Status

Azure Update Manager provides real-time visibility into update deployments across your virtual machines (VMs). From the Azure portal:

1. Navigate to **Update Manager** \> **Machines**.
2. Select a target machine or deployment schedule.
3. View the **Deployment status** tab to track update progress, including:
* Total number of updates installed
* Updates failed or skipped
* Duration and current step of each deployment

For more detailed insight, you can examine the **Update logs** for each VM. Look specifically for:

* **Skipped updates**: Often due to prerequisites not being met or updates being superseded.
* **Timeouts**: Caused by network latency or the machine being offline.
* **Agent issues**: Such as the Azure VM agent being outdated or unresponsive. These appear as error codes in the deployment report and can be correlated with logs in the VM’s local Event Viewer.

### Handling Failures

Deployment failures are logged automatically and can be exported for analysis. To configure this:

1. Go to the Update Manager settings in the Azure portal.
2. Under **Diagnostic settings**, enable log forwarding to one or more of the following:
* **Log Analytics** workspace
* **Event Hub**
* **Azure Storage Account**

This enables long-term storage and query-based troubleshooting using Kusto Query Language (KQL). Example KQL query to find failed patches:

```kusto
UpdateSummary
| where Status == "Failed"
| summarize count() by Computer, UpdateClassification

```

Once identified, failed patches can be:

* **Manually remediated** by logging into the affected VM and installing the update directly.
* **Re-deployed** using the original update schedule or a new targeted deployment.

### Integration with Alerts and Automation

To ensure proactive monitoring, integrate Azure Update Manager with Azure Monitor alerts:

* Define **compliance thresholds**, such as a rule that triggers when less than 90% of updates are successfully installed across a VM group.
* Use **Log Analytics** to define a custom query and alert rule.

Example alert logic:

```kusto
UpdateCompliance
| summarize complianceRate = avg(IsCompliant * 100) by TimeGenerated
| where complianceRate < 90

```

When an alert is triggered, you can:

* Send notifications via email or SMS.
* Trigger **Logic Apps** for complex workflows (e.g., opening a ticket in ServiceNow).
* Launch **Azure Automation Runbooks** to re-initiate the failed deployment or perform diagnostics.

This integration ensures that update compliance is continuously monitored and remediated automatically where possible.

![](https://oaidalleapiprodscus.blob.core.windows.net/private/org-2h1hGiSpEV2xL1QmmetBTfNv/user-ugXgjn5iKT1karGVlJhSKvX2/img-ViZcJbupQchKdn1yXCuJH477.png?st=2025-05-27T22%3A48%3A14Z&se=2025-05-28T00%3A48%3A14Z&sp=r&sv=2024-08-04&sr=b&rscd=inline&rsct=image/png&skoid=cc612491-d948-4d2e-9821-2683df3719f5&sktid=a48cca56-e6da-484e-a814-9c849652bcb3&skt=2025-05-27T21%3A45%3A55Z&ske=2025-05-28T21%3A45%3A55Z&sks=b&skv=2024-08-04&sig=DhkZoMDozVo1m8MlJC0iSeQimTzEwDu6R9rcTMTBzWo%3D)

## Best Practices for Update Management at Scale

Effective update management across a large fleet of Azure Virtual Machines (VMs) requires consistent enforcement, monitoring, and adaptation. This section outlines best practices in three key areas: governance and compliance, continuous improvement, and hybrid or multi-cloud environments.

### Governance and Compliance

To maintain a secure and compliant environment, define and enforce organizational patching standards using Azure-native tools:

* **Enforce patching policies through Azure Policy**: Azure Policy offers built-in definitions like `DeployIfNotExists` to ensure that VMs are configured for automatic updates or enrolled in Azure Update Manager. Custom policies can be authored to enforce OS-specific patching schedules, minimum compliance thresholds, or required reboot behaviors.
* **Audit update compliance via Azure Resource Graph queries**: Azure Resource Graph enables scalable, cross-subscription queries to evaluate patch status. Example query:

```kusto
  UpdateResources
  | where UpdateState == 'Needed'
  | summarize count() by OSName, SubscriptionId

```

Use these queries to identify non-compliant VMs, stale update configurations, or missed patch windows. Integrate them into Power BI dashboards or Azure Monitor workbooks for regular reporting.

### Continuous Improvement

Update management should not be a set-it-and-forget-it process. Implement a feedback loop to improve reliability and coverage:

* **Conduct monthly review of update compliance metrics**: Export data from Update Manager into Log Analytics or Azure Monitor. Track key indicators such as patch success rate, average deployment duration, and failure causes.
* **Tune deployment scopes and schedules based on failure patterns**: If recurring failures are localized to a region, OS version, or workload type, adjust deployment groups accordingly. For example, reschedule updates for database VMs to less disruptive hours, or stagger updates using maintenance windows to reduce concurrency-induced failures.

### Hybrid and Multi-Cloud Considerations

In mixed environments, achieving consistent patching behavior requires extending Azure-native capabilities:

* **Extend Update Manager to on-prem & other cloud VMs with Azure Arc**: Azure Arc enables Update Manager support for non-Azure machines by projecting them as resource objects in Azure. This allows unified policy enforcement, update orchestration, and compliance monitoring, regardless of location.
* **Ensure consistent patching posture across environments**: Standardize tagging conventions, update classifications, and maintenance window definitions across cloud and on-premises. Use Azure Resource Graph to compare compliance rates by environment type. Configuration drift between environments often leads to security gaps or audit failures.

By adhering to these practices, organizations can scale their update management processes with greater reliability, visibility, and security alignment.

![](https://oaidalleapiprodscus.blob.core.windows.net/private/org-2h1hGiSpEV2xL1QmmetBTfNv/user-ugXgjn5iKT1karGVlJhSKvX2/img-7uKgHmRJkHWme3wWWImQQR8j.png?st=2025-05-27T22%3A48%3A25Z&se=2025-05-28T00%3A48%3A25Z&sp=r&sv=2024-08-04&sr=b&rscd=inline&rsct=image/png&skoid=cc612491-d948-4d2e-9821-2683df3719f5&sktid=a48cca56-e6da-484e-a814-9c849652bcb3&skt=2025-05-27T17%3A43%3A53Z&ske=2025-05-28T17%3A43%3A53Z&sks=b&skv=2024-08-04&sig=MUN2e4IynrtrpU3imfEYgZ24nGhQWtKRWO9hCZqNY7g%3D)

## Conclusion

Automated update management with Azure Update Manager is a critical component of enterprise IT operations. It directly addresses key challenges in vulnerability management, compliance enforcement, and uptime assurance. Features such as **hotpatching**—which applies patches without requiring a reboot—and **dynamic scoping**—which allows targeting update deployments based on resource attributes—make it feasible to patch at scale without disrupting services.

Unpatched systems continue to be among the most exploited vectors in data breaches. By adopting Azure Update Manager, organizations can systematically reduce exposure windows, enforce update baselines, and align with security best practices. In practical terms, this shifts patching from a reactive, manual process to a proactive, policy-driven workflow.

The technical overhead of maintaining update compliance across hundreds or thousands of virtual machines (VMs) is non-trivial. Azure Update Manager abstracts much of this complexity, enabling teams to define, monitor, and enforce patching strategies centrally. The result is a more secure infrastructure, reduced operational risk, and measurable gains in system reliability.

In short, automated patch management is no longer a best practice—it is a baseline requirement. Azure Update Manager provides the tooling to meet that requirement at enterprise scale.

![](https://oaidalleapiprodscus.blob.core.windows.net/private/org-2h1hGiSpEV2xL1QmmetBTfNv/user-ugXgjn5iKT1karGVlJhSKvX2/img-f64HIkfuoUUFenbQ9QRBiowS.png?st=2025-05-27T22%3A48%3A43Z&se=2025-05-28T00%3A48%3A43Z&sp=r&sv=2024-08-04&sr=b&rscd=inline&rsct=image/png&skoid=cc612491-d948-4d2e-9821-2683df3719f5&sktid=a48cca56-e6da-484e-a814-9c849652bcb3&skt=2025-05-27T16%3A23%3A18Z&ske=2025-05-28T16%3A23%3A18Z&sks=b&skv=2024-08-04&sig=96QNjGsirpSAM0slJPnjFjmBpb%2B8432wEzKaEwySRhw%3D)