---
title: Getting Started with Microsoft Graph PowerShell SDK A Practical Guide
description: Learn how to install, configure, and use the Microsoft Graph PowerShell SDK for automating Microsoft 365 administration through scripts and secure authentication.
date: 2025-05-27
image: "/images/ITProffessional-1.png"
tags: [Microsoft Graph, PowerShell, Azure AD]
slug: getting-started-with-microsoft-graph-powershell-sdk
---

# Getting Started with Microsoft Graph PowerShell SDK: A Practical Guide

## Overview: What is the Microsoft Graph PowerShell SDK?

The **Microsoft Graph PowerShell SDK** is a module that enables administrators and developers to interact with Microsoft Graph via PowerShell. Microsoft Graph itself is a RESTful web API that acts as a single endpoint to access data and intelligence from Microsoft 365 services such as Exchange Online, Microsoft Teams, SharePoint, OneDrive, and Azure Active Directory (Azure AD).

This SDK abstracts the complexities of direct API calls by providing a structured, PowerShell-native interface. It allows scripting and automation of various administrative tasks without writing raw HTTP requests.

### Use Cases

Common scenarios where the SDK is especially useful include:

- **Automating Microsoft 365 administration**: Tasks like managing Teams policies, provisioning mailboxes, or handling Azure AD users can be scripted and scheduled.
- **Querying and modifying user data**: Retrieve user profiles, update properties, or manage licenses programmatically.
- **Batch operations and scheduled jobs**: Integrate with task schedulers or CI/CD pipelines to perform repetitive jobs, like compliance checks or usage reporting.

### Key Benefits

- **Unified authentication and permissions model**: Uses the Microsoft identity platform for consistent access control across services.
- **Flexible authentication options**: Supports interactive login (useful for testing) and non-interactive authentication such as certificate-based or client secret flows for service accounts.
- **Cross-platform compatibility**: Operates across Windows PowerShell 5.1 and PowerShell Core (v7+), enabling use on Windows, Linux, and macOS.
- **Standardized, discoverable cmdlets**: Cmdlets follow a predictable naming pattern (e.g., `Get-MgUser`, `New-MgTeam`) and are grouped by service area, simplifying learning and usage. ([lee-ford.co.uk](https://www.lee-ford.co.uk/graph-api-powershell-sdk/?utm_source=openai))

![](https://oaidalleapiprodscus.blob.core.windows.net/private/org-2h1hGiSpEV2xL1QmmetBTfNv/user-ugXgjn5iKT1karGVlJhSKvX2/img-V0XOg8NIFUgSFlFeZOhHiceE.png?st=2025-05-26T22%3A49%3A03Z&se=2025-05-27T00%3A49%3A03Z&sp=r&sv=2024-08-04&sr=b&rscd=inline&rsct=image/png&skoid=cc612491-d948-4d2e-9821-2683df3719f5&sktid=a48cca56-e6da-484e-a814-9c849652bcb3&skt=2025-05-26T17%3A15%3A43Z&ske=2025-05-27T17%3A15%3A43Z&sks=b&skv=2024-08-04&sig=PIxfJQ9gikE22tJdAy%2BjemCK3zZst/A0rDmna3MiQT8%3D)

## Prerequisites & Environment Setup

### Installation Requirements

To begin working with the Microsoft Graph PowerShell SDK, confirm that your environment meets the following requirements:

- **PowerShell 5.1 or later**: This is the minimum version required. PowerShell Core (7+) is supported across Windows, macOS, and Linux.
- **.NET Core Runtime**: Required for PowerShell Core usage. Ensure the appropriate version is installed based on your PowerShell version.
- **Microsoft 365 Tenant Access**: You need access to a Microsoft 365 tenant. This typically requires:
  - A Microsoft work or school account.
  - An Exchange Online mailbox.

For more details on account prerequisites, refer to the [Microsoft Graph PowerShell tutorial](https://learn.microsoft.com/en-us/graph/tutorials/powershell?utm_source=openai).

### SDK Installation

To install the Microsoft Graph PowerShell SDK, use the following command:

```powershell
Install-Module Microsoft.Graph -Scope CurrentUser
```

This installs the latest version of the SDK for the current user. If you encounter module name conflicts, append the `-AllowClobber` flag:

```powershell
Install-Module Microsoft.Graph -Scope CurrentUser -AllowClobber
```

For installing a specific version (e.g., version 2.x.x), use:

```powershell
Install-Module Microsoft.Graph -RequiredVersion "2.x.x"
```

Version 2 of the SDK introduces a modular structure, reducing installation size and improving load performance. For a detailed overview of these improvements, see the [TechTarget summary](https://www.techtarget.com/searchWindowsServer/tip/Whats-new-in-Microsoft-Graph-PowerShell-v2?utm_source=openai).

Once installed, verify the module is available:

```powershell
Get-Module -ListAvailable Microsoft.Graph
```

You're now ready to authenticate and begin making Microsoft Graph requests.

![](https://oaidalleapiprodscus.blob.core.windows.net/private/org-2h1hGiSpEV2xL1QmmetBTfNv/user-ugXgjn5iKT1karGVlJhSKvX2/img-IYvF2dNZTj9eTLQzn3I5hJJt.png?st=2025-05-26T22%3A49%3A20Z&se=2025-05-27T00%3A49%3A20Z&sp=r&sv=2024-08-04&sr=b&rscd=inline&rsct=image/png&skoid=cc612491-d948-4d2e-9821-2683df3719f5&sktid=a48cca56-e6da-484e-a814-9c849652bcb3&skt=2025-05-26T11%3A47%3A39Z&ske=2025-05-27T11%3A47%3A39Z&sks=b&skv=2024-08-04&sig=xiyUuFBJN7KEJWiTEjQeifnZXq0diw4c%2BtqnlGhJWMQ%3D)

## Authentication & Authorization

### Authentication Modes

Microsoft Graph PowerShell SDK supports multiple authentication modes, depending on the use case. Each mode uses OAuth2 under the hood but differs in interactivity and credential flow.

#### Interactive (Device Code)

This mode is suitable for testing or running scripts manually. It prompts the user to sign in through a browser using a device code.

```powershell
Connect-MgGraph -Scopes "User.Read, Mail.Send"
```

This command initiates a device code flow and requests delegated permissions for the specified scopes.

#### Client Credential Flow

Designed for automation scenarios, this mode requires an app registration in Azure AD and uses certificates or secrets for authentication. It provides application-level access without a signed-in user.

```powershell
Connect-MgGraph -ClientId <app-id> -TenantId <tenant-id> -CertificateThumbprint <thumbprint>
```

This command authenticates using a certificate tied to the registered application.

### Registering an App in Azure AD

To use the client credential flow, you must register an application in Azure Active Directory:

1. Navigate to **Azure Active Directory** in the Azure portal.
2. Select **App registrations**, then click **New registration**.
3. Provide a name and, if needed, specify redirect URIs.
4. Under **API permissions**, assign required permissions:
   - **Delegated** permissions for user-context operations.
   - **Application** permissions for app-only access.
5. Click **Grant admin consent** to finalize permission changes.

For a detailed walkthrough, see [thesleepyadmins.com](https://thesleepyadmins.com/2020/11/22/using-microsoft-graph-powershell-sdk/?utm_source=openai).

### Token Handling

The SDK handles OAuth2 tokens automatically. It caches tokens and refreshes them as needed, reducing manual intervention.

Use the following command to inspect the current authentication context and token details:

```powershell
Get-MgContext
```

This reveals the current scopes, tenant, and token expiration.

More information on token behavior is available at [lee-ford.co.uk](https://www.lee-ford.co.uk/graph-api-powershell-sdk/?utm_source=openai).

![](https://oaidalleapiprodscus.blob.core.windows.net/private/org-2h1hGiSpEV2xL1QmmetBTfNv/user-ugXgjn5iKT1karGVlJhSKvX2/img-yyIYixKb2NAP6bG66SmgbAoM.png?st=2025-05-26T22%3A49%3A37Z&se=2025-05-27T00%3A49%3A37Z&sp=r&sv=2024-08-04&sr=b&rscd=inline&rsct=image/png&skoid=cc612491-d948-4d2e-9821-2683df3719f5&sktid=a48cca56-e6da-484e-a814-9c849652bcb3&skt=2025-05-26T15%3A12%3A44Z&ske=2025-05-27T15%3A12%3A44Z&sks=b&skv=2024-08-04&sig=s6Pey5UHyE7Z2GIG3S6FS4Jjb4fz4TysviiKpM5rOGU%3D)

## Core Concepts: Cmdlet Structure, Modules, and Permissions

### Cmdlet Naming Convention

Microsoft Graph PowerShell SDK cmdlets follow a consistent naming convention designed around the format:

```powershell
Verb-Mg<Resource>
```

For example:

- `Get-MgUser` — Retrieves a user object from Microsoft Graph.
- `Send-MgUserMail` — Sends email on behalf of a user.

To discover all available cmdlets across Microsoft Graph modules, use:

```powershell
Get-Command -Module Microsoft.Graph*
```

This command lists all cmdlets from installed Microsoft Graph modules, making it easier to explore functionality by resource.

### Module Structure (v2+)

Starting with version 2, the Microsoft Graph PowerShell SDK is modular. Instead of installing a monolithic package, you can now install only the modules you need. This leads to smaller install footprints and faster import times.

For example, to work with users:

```powershell
Install-Module Microsoft.Graph.Users
Import-Module Microsoft.Graph.Users
```

This modular design improves performance and aligns with PowerShell best practices. [TechTarget explains the advantages in more detail](https://www.techtarget.com/searchWindowsServer/tip/Whats-new-in-Microsoft-Graph-PowerShell-v2?utm_source=openai).

### Permissions Model

Microsoft Graph enforces a robust permissions model. Cmdlets require either **delegated** or **application** permissions, depending on the execution context.

- **Delegated Permissions**: These run under a signed-in user’s context. They are suitable for interactive scenarios—e.g., a script run by an admin in their own session.

- **Application Permissions**: These are used in app-only contexts, such as background services or daemon processes. They require certificate-based authentication or client secrets.

For a deeper dive into permission types and how to configure them, see [Practical365’s introduction](https://practical365.com/introduction-to-the-microsoft-graph-powershell-sdk/?utm_source=openai).

Understanding the interplay between cmdlets, modules, and permissions is critical for building secure, maintainable scripts with the Microsoft Graph PowerShell SDK.

![](https://oaidalleapiprodscus.blob.core.windows.net/private/org-2h1hGiSpEV2xL1QmmetBTfNv/user-ugXgjn5iKT1karGVlJhSKvX2/img-FFjn2Y5a8FqHHBqP5SEx8y9I.png?st=2025-05-26T22%3A49%3A54Z&se=2025-05-27T00%3A49%3A54Z&sp=r&sv=2024-08-04&sr=b&rscd=inline&rsct=image/png&skoid=cc612491-d948-4d2e-9821-2683df3719f5&sktid=a48cca56-e6da-484e-a814-9c849652bcb3&skt=2025-05-26T15%3A38%3A30Z&ske=2025-05-27T15%3A38%3A30Z&sks=b&skv=2024-08-04&sig=OkIuzKUm2OgxwgWyb5SdM/CuEpnm1bpkFi4LNsX2jgk%3D)

## Hands-On Examples

The Microsoft Graph PowerShell SDK provides a PowerShell interface to the Microsoft Graph API, enabling access to Microsoft 365 data. Below are practical examples demonstrating common use cases.

### 1. Get Current User Info

To retrieve information about the currently signed-in user:

```powershell
Connect-MgGraph -Scopes "User.Read"
Get-MgUser -UserId me
```

The `-Scopes` parameter specifies the permission scope required. `me` is a special alias referring to the authenticated user.

### 2. List Inbox Messages

To fetch the five most recent messages in the signed-in user's inbox:

```powershell
Get-MgUserMessage -UserId me -Top 5
```

The `-Top` parameter controls pagination by limiting the number of results returned.

### 3. Send an Email

To send an email from the authenticated user:

```powershell
Send-MgUserMail -UserId me -Message @{
    Subject = "Demo"
    Body = @{ ContentType = "Text"; Content = "This is a test." }
    ToRecipients = @(@{ EmailAddress = @{ Address = "example@domain.com" } })
} -SaveToSentItems
```

This sends a plain-text email and saves a copy in the Sent Items folder. Adjust the `ContentType` to `HTML` for rich formatting.

### 4. Use Filters and Pagination

To filter users whose display names start with "Dylan" and limit results to ten entries:

```powershell
Get-MgUser -Filter "startsWith(displayName,'Dylan')" -Top 10
```

This uses OData-style filtering supported by Microsoft Graph. Note that not all properties are filterable.

### 5. Schedule a Script with Non-Interactive Auth

For automation scenarios (e.g., scripts running as scheduled tasks or background jobs), use certificate-based authentication. This avoids interactive sign-ins.

1. Register an Azure AD app and upload a certificate.
2. Use the `-CertificateThumbprint` parameter with `Connect-MgGraph`:

```powershell
Connect-MgGraph -ClientId "<app-id>" -TenantId "<tenant-id>" -CertificateThumbprint "<thumbprint>"
```

This enables secure, unattended access—suitable for CI/CD pipelines or service accounts.

![](https://oaidalleapiprodscus.blob.core.windows.net/private/org-2h1hGiSpEV2xL1QmmetBTfNv/user-ugXgjn5iKT1karGVlJhSKvX2/img-CKzItIxgrhkBPDDK2PL12BDQ.png?st=2025-05-26T22%3A50%3A11Z&se=2025-05-27T00%3A50%3A11Z&sp=r&sv=2024-08-04&sr=b&rscd=inline&rsct=image/png&skoid=cc612491-d948-4d2e-9821-2683df3719f5&sktid=a48cca56-e6da-484e-a814-9c849652bcb3&skt=2025-05-26T13%3A37%3A03Z&ske=2025-05-27T13%3A37%3A03Z&sks=b&skv=2024-08-04&sig=tNZwnejR8F8RB%2Bhk0WSjJ8jSks//45L38ZuWAp%2BME7M%3D)

## Troubleshooting & Best Practices

### Common Issues

**Permission Denied**  
This typically occurs when the signed-in user or app lacks the necessary Microsoft Graph permissions. Verify that admin consent has been granted for the required scopes. You can inspect current permissions using `Get-MgContext` and consult the Azure Portal to confirm role assignments.

**Token Expiry**  
Access tokens issued by Microsoft Identity Platform are short-lived (usually 1 hour). If you receive authentication-related errors after a time delay, re-authentication may be necessary. For long-running scripts, implement token refresh logic or use `Connect-MgGraph` with certificate-based authentication.

**Cmdlet Not Found**  
This usually indicates that the appropriate module or command set is not installed or imported. Use `Get-Command -Module Microsoft.Graph*` to list installed cmdlets. You can install the latest SDK modules with:

```powershell
Install-Module Microsoft.Graph -Scope CurrentUser
```

Check for specific workloads (e.g., Users, Groups) with:

```powershell
Find-MgGraphCommand -Command "Get-MgUser"
```

### Tips

- Apply the **principle of least privilege** when assigning API permissions. Grant only the scopes needed by the script or user context.
- Use `Select-MgProfile -Name beta` to target Microsoft Graph beta endpoints. Be aware that these APIs are subject to change and not supported for production workloads.
- Run `Disconnect-MgGraph` at the end of a session to explicitly clear authentication context, especially important when switching users or automating multiple runs.

### Community & Documentation

- [Microsoft Official Tutorial](https://learn.microsoft.com/en-us/graph/tutorials/powershell?utm_source=openai)
- [TheSleepyAdmins](https://thesleepyadmins.com/2020/11/22/using-microsoft-graph-powershell-sdk/?utm_source=openai)
- [Practical365](https://practical365.com/introduction-to-the-microsoft-graph-powershell-sdk/?utm_source=openai)

By mastering the Microsoft Graph PowerShell SDK, IT professionals can build reliable, scalable automation for cloud-based environments—using familiar tools with enterprise-grade extensibility.

![](https://oaidalleapiprodscus.blob.core.windows.net/private/org-2h1hGiSpEV2xL1QmmetBTfNv/user-ugXgjn5iKT1karGVlJhSKvX2/img-5WyxWo4Ty6wiFC8cZw2Qy4V2.png?st=2025-05-26T22%3A50%3A27Z&se=2025-05-27T00%3A50%3A27Z&sp=r&sv=2024-08-04&sr=b&rscd=inline&rsct=image/png&skoid=cc612491-d948-4d2e-9821-2683df3719f5&sktid=a48cca56-e6da-484e-a814-9c849652bcb3&skt=2025-05-26T13%3A02%3A30Z&ske=2025-05-27T13%3A02%3A30Z&sks=b&skv=2024-08-04&sig=nRy/SKFxCKht7vgXoc0f5Kukd9HvWt63%2BHPWi2XqmpE%3D)
