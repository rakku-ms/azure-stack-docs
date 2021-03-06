---
title: Troubleshoot validation as a service
titleSuffix: Azure Stack Hub
description: Troubleshoot validation as a service for Azure Stack Hub.
author: mattbriggs
ms.topic: article
ms.date: 04/20/2020
ms.author: mabrigg
ms.reviewer: johnhas
ms.lastreviewed: 11/11/2019


ROBOTS: NOINDEX

# Intent: As an Azure Stack Hub user, I want to troubleshoot validation as a service for Azure Stack Hub.
# Keyword: azure stack hub validation troubleshoot

---


# Troubleshoot Validation as a Service

[!INCLUDE [Azure_Stack_Partner](./includes/azure-stack-partner-appliesto.md)]

The following are common problems unrelated to software releases and their solutions.

## Local agent

### The portal shows local agent in debug mode

This problem is likely because the agent is unable to send heartbeats to the service because of an unstable network connection. A heartbeat is sent every five minutes. If the service doesn't receive a heartbeat for 15 minutes, then the service considers the agent inactive and tests will no longer be scheduled on it. Check the error message in the *Agenthost.log* file located in the directory where the agent was started.

> [!Note]
> Any tests already running on the agent will continue to run, but if the heartbeat isn't restored before the test ends, the agent will fail to update the test status or upload logs. The test will always show up as **running** and will need to be canceled.

### Agent process on machine was shut down while executing test. What to expect?

If the agent process is shut down ungracefully, then the test that was running on it will continue to show as **running**. An example of an ungraceful shutdown is machine restarted and process killed (CTRL+C on the agent window is considered graceful shutdown). If the agent is restarted, then the agent will update the status of the test to **canceled**. If the agent isn't restarted, then the test appears as **running** and you must manually cancel the test.

> [!Note]
> Tests within a workflow are scheduled to run sequentially. **Pending** tests won't get executed until tests in the **running** state in the same workflow complete.

## VM images

### Handle slow network connectivity

You can download the PIR image to a share in your local datacenter. And then you can check the image.

<!-- This is from the appendix to the Deploy local agent topic. -->

#### Download PIR image to local share in case of slow network traffic

1. Download AzCopy from: [vaasexternaldependencies(AzCopy)](https://vaasexternaldependencies.blob.core.windows.net/prereqcomponents/AzCopy.zip).

2. Extract AzCopy.zip and change to the directory containing `AzCopy.exe`.

3. Open Windows PowerShell from an elevated prompt. Run the following commands:

```powershell  
    .\azcopy.exe /Source:'https://azurestacktemplate.blob.core.windows.net/azurestacktemplate-public-container' /Dest:'<LocalFileShare>' /Pattern:'Server2016DatacenterFullBYOL.vhd' /NC:12 /V:azcopylog.log /Y
    .\azcopy.exe /Source:'https://azurestacktemplate.blob.core.windows.net/azurestacktemplate-public-container' /Dest:'<LocalFileShare>' /Pattern:'Server2016DatacenterCoreBYOL.vhd' /NC:12 /V:azcopylog.log /Y
    .\azcopy.exe /Source:'https://azurestacktemplate.blob.core.windows.net/azurestacktemplate-public-container' /Dest:'<LocalFileShare>' /Pattern:'WindowsServer2012R2DatacenterBYOL.vhd' /NC:12 /V:azcopylog.log /Y
    .\azcopy.exe /Source:'https://azurestacktemplate.blob.core.windows.net/azurestacktemplate-public-container' /Dest:'<LocalFileShare>' /Pattern:'Ubuntu1404LTS.vhd' /NC:12 /V:azcopylog.log /Y
    .\azcopy.exe /Source:'https://azurestacktemplate.blob.core.windows.net/azurestacktemplate-public-container' /Dest:'<LocalFileShare>' /Pattern:'Ubuntu1604-20170619.1.vhd' /NC:12 /V:azcopylog.log /Y
    .\azcopy.exe /Source:'https://azurestacktemplate.blob.core.windows.net/azurestacktemplate-public-container' /Dest:'<LocalFileShare>' /Pattern:'OpenLogic-CentOS-69-20180105.vhd' /NC:12 /V:azcopylog.log /Y
    .\azcopy.exe /Source:'https://azurestacktemplate.blob.core.windows.net/azurestacktemplate-public-container' /Dest:'<LocalFileShare>' /Pattern:'Debian8_latest.vhd' /NC:12 /V:azcopylog.log /Y
```

> [!Note]  
> LocalFileShare is the share path or local path.

#### Verifying PIR Image file hash value

You can use **Get-HashFile** cmdlet to get the hash value for the downloaded public image repository image files to check the integrity of the images.

| File Name | SHA256 |
|---------------------------------------|------------------------------------------------------------------|
| Server2016DatacenterFullBYOL.vhd | 6ED58DCA666D530811A1EA563BA509BF9C29182B902D18FCA03C7E0868F733E9 |
| WindowsServer2012R2DatacenterBYOL.vhd | 9792CBF742870B1730B9B16EA814C683A8415EFD7601DDB6D5A76D0964767028 |
| Server2016DatacenterCoreBYOL.vhd | 5E80E1A6721A48A10655E6154C1B90E320DF5558487D6A0D7BFC7DCD32C4D9A5 |
| Ubuntu1404LTS.vhd | B24CDD12352AAEBC612A4558AB9E80F031A2190E46DCB459AF736072742E20E0 |
| Ubuntu1604-20170619.1.vhd | C481B88B60A01CBD5119A3F56632A2203EE5795678D3F3B9B764FFCA885E26CB |
| OpenLogic-CentOS-69-20180105.vhd | C8B874FE042E33B488110D9311AF1A5C7DC3B08E6796610BF18FDD6728C7913C |
| Debian8_latest.vhd | 06F8C11531E195D0C90FC01DFF5DC396BB1DD73A54F8252291ED366CACD996C1 |

### Failure happens when uploading VM image in the `VaaSPreReq` script

First check that the environment is healthy:

1. From the DVM / jump box, check that you can successfully sign in to the administrator portal using the admin credentials.
1. Confirm that there are no alerts or warnings.

If the environment is healthy, manually upload the five VM Images required for VaaS test runs:

1. Sign in as the service admin to the administrator portal. You can find the administrator portal URL from ECE store or your stamp information file. For instructions, see [Environment parameters](azure-stack-vaas-parameters.md#environment-parameters).
1. Select **More services** > **Resource Providers** > **Compute** > **VM Images**.
1. Select the **+ Add** button at the top of the **VM Images** blade.
1. Modify or check values of the following fields for the first VM image:
    > [!IMPORTANT]
    > Not all defaults are correct for the existing marketplace item.

    | Field  | Value  |
    |---------|---------|
    | Publisher | MicrosoftWindowsServer |
    | Offer | WindowsServer |
    | OS Type | Windows |
    | SKU | 2012-R2-Datacenter |
    | Version | 1.0.0 |
    | OS Disk Blob URI | https://azurestacktemplate.blob.core.windows.net/azurestacktemplate-public-container/WindowsServer2012R2DatacenterBYOL.vhd |

1. Select the **Create** button.
1. Repeat for the remaining VM images.

The properties of all five VM images are as follows:

| Publisher  | Offer  | OS Type | SKU | Version | OS Disk Blob URI |
|---------|---------|---------|---------|---------|---------|
| MicrosoftWindowsServer| WindowsServer | Windows | 2012-R2-Datacenter | 1.0.0 | https://azurestacktemplate.blob.core.windows.net/azurestacktemplate-public-container/WindowsServer2012R2DatacenterBYOL.vhd |
| MicrosoftWindowsServer | WindowsServer | Windows | 2016-Datacenter | 1.0.0 | https://azurestacktemplate.blob.core.windows.net/azurestacktemplate-public-container/Server2016DatacenterFullBYOL.vhd |
| MicrosoftWindowsServer | WindowsServer | Windows | 2016-Datacenter-Server-Core | 1.0.0 | https://azurestacktemplate.blob.core.windows.net/azurestacktemplate-public-container/Server2016DatacenterCoreBYOL.vhd |
| Canonical | UbuntuServer | Linux | 14.04.3-LTS | 1.0.0 | https://azurestacktemplate.blob.core.windows.net/azurestacktemplate-public-container/Ubuntu1404LTS.vhd |
| Canonical | UbuntuServer | Linux | 16.04-LTS | 16.04.20170811 | https://azurestacktemplate.blob.core.windows.net/azurestacktemplate-public-container/Ubuntu1604-20170619.1.vhd |
| OpenLogic | CentOS | Linux | 6.9 | 1.0.0 | https://azurestacktemplate.blob.core.windows.net/azurestacktemplate-public-container/OpenLogic-CentOS-69-20180105.vhd |
| credativ | Debian | Linux | 8 | 1.0.0 | https://azurestacktemplate.blob.core.windows.net/azurestacktemplate-public-container/Debian8_latest.vhd |

## Next steps

- Review [Release notes for validation as a service](azure-stack-vaas-release-notes.md) for changes in the latest releases.