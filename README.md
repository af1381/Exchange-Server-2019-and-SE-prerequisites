# Exchange Server 2019 & SE Prerequisites

This README includes the system prerequisites for installing Exchange Server 2019 and Subscription Edition (SE), including both Mailbox and Edge Transport roles, as well as management tools for client machines.

## Pre-Installation Notes

- **Active Directory**: Ensure your AD is compatible with Exchange 2019 and SE.
- **Supported Operating Systems**: Use a supported Windows Server version.
- **Windows Updates**: Make sure all updates are installed.
- **Remote Registry Service**: Must be set to `Automatic` and not disabled.

## Windows Server Prerequisites for Exchange Server

### Prepare Active Directory

**Required software:**
- Supported version of .NET Framework
- Visual C++ Redistributable for Visual Studio 2012

**Install RSAT tools:**
```powershell
Install-WindowsFeature RSAT-ADDS
```

### Exchange Management Tools

**Required software:**
- Supported .NET Framework
- Visual C++ Redistributable for Visual Studio 2012

**Install Windows features:**
- Server OS:
```powershell
Install-WindowsFeature -Name Web-Mgmt-Console, Web-Metabase
```
- Client OS:
```powershell
Enable-WindowsOptionalFeature -Online -FeatureName IIS-ManagementConsole, IIS-Metabase -All
```

### Mailbox Role

**Required software:**
- Supported .NET Framework
- Visual C++ Redistributable for Visual Studio 2012 & 2013

**Install Windows features:**
```powershell
Install-WindowsFeature Server-Media-Foundation, NET-Framework-45-Core, NET-Framework-45-ASPNET, NET-WCF-HTTP-Activation45, NET-WCF-Pipe-Activation45, NET-WCF-TCP-Activation45, NET-WCF-TCP-PortSharing45, RPC-over-HTTP-proxy, RSAT-Clustering, RSAT-Clustering-CmdInterface, RSAT-Clustering-Mgmt, RSAT-Clustering-PowerShell, WAS-Process-Model, Web-Asp-Net45, Web-Basic-Auth, Web-Client-Auth, Web-Digest-Auth, Web-Dir-Browsing, Web-Dyn-Compression, Web-Http-Errors, Web-Http-Logging, Web-Http-Redirect, Web-Http-Tracing, Web-ISAPI-Ext, Web-ISAPI-Filter, Web-Metabase, Web-Mgmt-Console, Web-Mgmt-Service, Web-Net-Ext45, Web-Request-Monitor, Web-Server, Web-Stat-Compression, Web-Static-Content, Web-Windows-Auth, Web-WMI, Windows-Identity-Foundation, RSAT-ADDS
```
- For Server Core:
```powershell
Install-WindowsFeature Server-Media-Foundation, NET-Framework-45-Core, NET-Framework-45-ASPNET, NET-WCF-HTTP-Activation45, NET-WCF-Pipe-Activation45, NET-WCF-TCP-Activation45, NET-WCF-TCP-PortSharing45, RPC-over-HTTP-proxy, RSAT-Clustering, RSAT-Clustering-CmdInterface, RSAT-Clustering-PowerShell, WAS-Process-Model, Web-Asp-Net45, Web-Basic-Auth, Web-Client-Auth, Web-Digest-Auth, Web-Dir-Browsing, Web-Dyn-Compression, Web-Http-Errors, Web-Http-Logging, Web-Http-Redirect, Web-Http-Tracing, Web-ISAPI-Ext, Web-ISAPI-Filter, Web-Metabase, Web-Mgmt-Service, Web-Net-Ext45, Web-Request-Monitor, Web-Server, Web-Stat-Compression, Web-Static-Content, Web-Windows-Auth, Web-WMI, RSAT-ADDS
```

### Edge Transport Role

**Required software:**
- Supported .NET Framework
- Visual C++ Redistributable for Visual Studio 2012

**Install Windows features:**
```powershell
Install-WindowsFeature ADLDS
