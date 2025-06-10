---
title: Streamlining Credential Management in Azure AD App Registrations
description: As organizations grow and adopt cloud services, managing the lifecycle of Azure AD App Registration credentials—namely, secrets and certificates—becomes a critical, yet often cumbersome administrative task. With these credentials being essential for securing API integrations and application access, missing an expiration date can lead to unexpected downtime or security vulnerabilities. This article explores the administrative challenges associated with managing these expirations and demonstrates how a Microsoft Graph PowerShell SDK script can automate the process, ensuring that administrators stay ahead of potential issues.
date: 2025-02-19 14:50:00 +0300
image: "/images/Cloud-Tech-1.jpg"
tags: [Administration, Microsoft Graph, Azure AD]
---

Organizations leveraging Azure AD App Registrations rely heavily on client secrets and certificates for secure authentication between applications and services. However, these credentials have expiration dates that, if left unchecked, can lead to service outages, security risks, and administrative burden.

Manually tracking the expiration of these secrets and certificates introduces significant operational overhead, especially in large enterprises where there may be hundreds or thousands of app registrations. Administrators must:

- Regularly review each app registration in Azure AD.
- Identify expiring secrets and certificates in a timely manner.
- Notify relevant teams to update credentials before expiration.

Failure to proactively manage these expirations can result in:

- Application downtime due to expired authentication credentials.
- Security vulnerabilities caused by outdated secrets lingering in production environments.
- Increased workload for IT teams responding to last-minute failures.

To mitigate these risks, organizations need an automated solution that continuously monitors Azure AD secrets and certificates, providing timely notifications.

## Automating Expiration Monitoring with Microsoft Graph PowerShell SDK

A more scalable approach to tracking app secrets and certificates involves leveraging the Microsoft Graph PowerShell SDK. With a PowerShell script, we can:

- Connect to Microsoft Graph.
- Retrieve all Azure AD App Registrations.
- Identify secrets and certificates that are nearing expiration.
- Send an automated email notification with a summary of expiring credentials.

## Understanding the PowerShell Script

This script automates the expiration tracking process and includes key functionalities:

1. Connects to Microsoft Graph API using a provided Tenant ID, Client ID, and Client Secret.
2. Retrieves all Azure AD App Registrations.
3. Extracts all secrets and certificates from the applications.
4. Filters out expired or soon-to-expire credentials, based on a user-defined threshold (e.g., 30 days).
5. Formats the data into a readable table.
6. Sends an email notification to administrators with the expiration details.

## How the Script Works

### 1. Defining Parameters

The script accepts key parameters to customize execution:

- $DaysToCheck – Specifies how many days ahead to look for expiring credentials.
- $SendEmail – A switch to enable email notifications.
- $EmailTo – The recipient email address for notifications.
- $EmailFrom – The sender's email (UPN or Object ID).
- $TenantId, $ClientId, $ClientSecret – Used to authenticate against Microsoft Graph.

### 2. Connecting to Microsoft Graph

```powershell
Connect-MgGraph -TenantId $TenantId -ClientId $ClientId -ClientSecret $ClientSecret
```

This command authenticates to Microsoft Graph with the necessary permissions to read App Registration details.

### 3. Retrieving Azure AD App Registrations

```powershell
$appRegistrations = Get-MgApplication
```

This retrieves all applications registered in Azure AD.

### 4. Extracting Expiring Secrets and Certificates

```powershell
foreach ($app in $AppRegistrations) {
        $certs = $app.KeyCredentials
        $secrets = $app.PasswordCredentials...
}
```

This filters client secrets that are expiring within the specified timeframe.

### 5. Sending an Email Notification

If the $SendEmail flag is enabled, the script formats the expiring credentials into an email-friendly format and sends a notification using Microsoft Graph's email API:

```powershell
try {
        # Send email using Microsoft Graph
        Send-MgUserMail -UserId $From -BodyParameter $params
        Write-Output "Email notification sent successfully using Microsoft Graph."
    }
```

## Sample Email Output

When the script detects an expiring credential, it generates an email similar to this:

> ### Subject: Azure AD App Registration - Expiring Certificates and Secrets
>
> #### The following certificates and secrets will expire within the next 30 days:
>
> | App Name | App ID        | Type        | Expiry Date | Days Remaining |
> | -------- | ------------- | ----------- | ----------- | -------------- |
> | AppOne   | 12345678-ABCD | Certificate | 2024-08-15  | 15             |
> | AppTwo   | 87654321-DCBA | Secret      | 2024-08-20  | 20             |
> | AppThree | 11223344-EFGH | Certificate | 2024-08-29  | 29             |
>
> ##### Please take action to renew these items before they expire.

## Here is the PowerShell Script

```powershell
param(
    [Parameter(Mandatory = $false)]
    [int]$DaysToCheck = 30,

    [Parameter(Mandatory = $false)]
    [switch]$SendEmail = $false,

    [Parameter(Mandatory = $false)]
    [string]$EmailTo = "<RecipientEmail>",

    [Parameter(Mandatory = $false)]
    [string]$EmailFrom = "<SenderEmail",

    [Parameter(Mandatory = $false)]
    [string]$TenantId = "<TenantID>",

    [Parameter(Mandatory = $false)]
    [string]$AppId = "<ApplicationClientID",

    [Parameter(Mandatory = $false)]
    [string]$ClientSecret = "<ApplicationSecretValue>"
)

function Connect-ToMicrosoftGraph {
    param (
        [Parameter(Mandatory = $false)]
        [string]$TenantId,

        [Parameter(Mandatory = $false)]
        [string]$ClientId,

        [Parameter(Mandatory = $false)]
        [string]$ClientSecret
    )

    try {
        if ([string]::IsNullOrEmpty($TenantId) -or [string]::IsNullOrEmpty($ClientId) -or [string]::IsNullOrEmpty($ClientSecret)) {
            # For Azure Automation, use Managed Identity
            Connect-MgGraph -Identity -Scopes "Application.Read.All", "Mail.Send"
            Write-Output "Successfully connected to Microsoft Graph using Managed Identity."
        } else {
            # Use client credentials flow
            $SecureSecret = ConvertTo-SecureString -String $ClientSecret -AsPlainText -Force
            $ClientSecretCredential = New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $ClientId, $SecureSecret
            Connect-MgGraph -TenantId $TenantId -ClientSecretCredential $ClientSecretCredential
            Write-Output "Successfully connected to Microsoft Graph using Client Credentials."
        }
    }
    catch {
        Write-Error "Failed to connect to Microsoft Graph: $_"
        throw
    }
}

function Get-AppRegistrations {
    try {
        $appRegistrations = Get-MgApplication
        Write-Output "Retrieved $($appRegistrations.Count) app registrations."
        return $appRegistrations
    }
    catch {
        Write-Error "Failed to retrieve app registrations: $_"
        throw
    }
}

function Get-ExpiringItems {
    param (
        [Parameter(Mandatory = $true)]
        [object[]]$AppRegistrations,

        [Parameter(Mandatory = $true)]
        [datetime]$ExpirationDate
    )

    $expiringItems = @()

    foreach ($app in $AppRegistrations) {
        $certs = $app.KeyCredentials
        $secrets = $app.PasswordCredentials

        # Check certificates
        foreach ($cert in $certs) {
            if ($cert.EndDateTime -lt $ExpirationDate) {
                $expiringItems += [PSCustomObject]@{
                    AppName = $app.DisplayName
                    AppId = $app.AppId
                    Type = "Certificate"
                    ExpiryDate = $cert.EndDateTime
                    DaysRemaining = [math]::Floor(($cert.EndDateTime - (Get-Date)).TotalDays)
                }
            }
        }

        # Check secrets
        foreach ($secret in $secrets) {
            if ($secret.EndDateTime -lt $ExpirationDate) {
                $expiringItems += [PSCustomObject]@{
                    AppName = $app.DisplayName
                    AppId = $app.AppId
                    Type = "Secret"
                    ExpiryDate = $secret.EndDateTime
                    DaysRemaining = [math]::Floor(($secret.EndDateTime - (Get-Date)).TotalDays)
                }
            }
        }
    }

    return $expiringItems
}

function Send-GraphEmail {
    param (
        [Parameter(Mandatory = $true)]
        [object[]]$ExpiringItems,

        [Parameter(Mandatory = $true)]
        [string]$To,

        [Parameter(Mandatory = $true)]
        [string]$From
    )

    $subject = "Azure AD App Registration - Expiring Certificates and Secrets Alert"

    # Create HTML table content
    $tableRows = ""
    foreach ($item in $ExpiringItems) {
        $tableRows += @"
<tr>
    <td style='padding:8px;border:1px solid #ddd;'>$($item.AppName)</td>
    <td style='padding:8px;border:1px solid #ddd;'>$($item.AppId)</td>
    <td style='padding:8px;border:1px solid #ddd;'>$($item.Type)</td>
    <td style='padding:8px;border:1px solid #ddd;'>$($item.ExpiryDate)</td>
    <td style='padding:8px;border:1px solid #ddd;'>$($item.DaysRemaining)</td>
</tr>
"@
    }

    # Complete HTML body
    $htmlContent = @"
<h2>Azure AD App Registration - Expiring Certificates and Secrets</h2>
<p>The following certificates and secrets will expire within the next $DaysToCheck days:</p>
<table style='border-collapse:collapse;width:100%;border:1px solid #ddd;'>
    <tr style='background-color:#f2f2f2;'>
        <th style='padding:8px;border:1px solid #ddd;'>App Name</th>
        <th style='padding:8px;border:1px solid #ddd;'>App ID</th>
        <th style='padding:8px;border:1px solid #ddd;'>Type</th>
        <th style='padding:8px;border:1px solid #ddd;'>Expiry Date</th>
        <th style='padding:8px;border:1px solid #ddd;'>Days Remaining</th>
    </tr>
    $tableRows
</table>
<p>Please take action to renew these items before they expire.</p>
"@

    # Create email parameters for Microsoft Graph
    $params = @{
        message = @{
            subject = $subject
            body = @{
                contentType = "HTML"
                content = $htmlContent
            }
            toRecipients = @(
                @{
                    emailAddress = @{
                        address = $To
                    }
                }
            )
        }
        saveToSentItems = "true"
    }

    try {
        # Send email using Microsoft Graph
        Send-MgUserMail -UserId $From -BodyParameter $params
        Write-Output "Email notification sent successfully using Microsoft Graph."
    }
    catch {
        Write-Error "Failed to send email notification through Microsoft Graph: $_"
    }
}

# Main Script Execution
try {
    # Connect to Microsoft Graph with appropriate credentials
    Connect-ToMicrosoftGraph -TenantId $TenantId -ClientId $AppId -ClientSecret $ClientSecret

    # Calculate expiration date
    $expirationDate = (Get-Date).AddDays($DaysToCheck)
    Write-Output "Checking for items expiring before: $expirationDate"

    # Get all App Registrations
    $appRegistrations = Get-AppRegistrations

    # Get expiring items
    $expiringItems = Get-ExpiringItems -AppRegistrations $appRegistrations -ExpirationDate $expirationDate

    # Output results
    if ($expiringItems.Count -eq 0) {
        Write-Output "No certificates or secrets are expiring within the next $DaysToCheck days."
    }
    else {
        Write-Output "Found $($expiringItems.Count) expiring items:"
        $expiringItems | Sort-Object -Property DaysRemaining | Format-Table -Property AppName, Type, ExpiryDate, DaysRemaining -AutoSize


        # Send email notification if enabled
        $sendEmail = $true
        if ($SendEmail -and -not [string]::IsNullOrEmpty($EmailTo) -and -not [string]::IsNullOrEmpty($EmailFrom)) {
            Send-GraphEmail -ExpiringItems $expiringItems -To $EmailTo -From $EmailFrom
        }
    }
}
catch {
    Write-Error "An error occurred during script execution: $_"
}
finally {
    # Disconnect from Microsoft Graph
    Disconnect-MgGraph
    Write-Output "Disconnected from Microsoft Graph."
}
```

# Conclusion

Manually tracking Azure AD App Registration secrets and certificates is not scalable for enterprise environments. By leveraging Microsoft Graph PowerShell SDK, administrators can:

- Automate expiration monitoring.
- Reduce operational overhead.
- Ensure proactive notifications to prevent outages.

This script provides a robust solution to managing Azure AD credentials efficiently, ensuring security and continuity in enterprise environments. Consider scheduling this script as a daily or weekly automation using Azure Automation or a scheduled task to keep your environment secure.
