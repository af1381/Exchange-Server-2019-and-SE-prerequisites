# پیش‌نیازهای Exchange Server 2019 و SE

این README شامل پیش‌نیازهای سیستم برای نصب Exchange Server 2019 و Subscription Edition (SE) است، شامل نقش‌های Mailbox و Edge Transport و همچنین ابزارهای مدیریت روی کامپیوترهای کلاینت.

## نکات قبل از نصب

- **Active Directory**: اطمینان حاصل کنید که AD با Exchange 2019 و SE سازگار باشد.
- **سیستم‌عامل پشتیبانی‌شده**: از نسخه‌های پشتیبانی‌شده Windows Server استفاده کنید.
- **به‌روزرسانی‌های ویندوز**: مطمئن شوید آخرین به‌روزرسانی‌ها نصب شده‌اند.
- **سرویس Remote Registry**: باید روی `Automatic` تنظیم شده و غیرفعال نباشد.

## پیش‌نیازهای Windows Server برای Exchange Server

### آماده‌سازی Active Directory

**نرم‌افزارهای موردنیاز:**
- نسخه پشتیبانی‌شده .NET Framework
- Visual C++ Redistributable برای Visual Studio 2012

**نصب ابزارهای RSAT:**
```powershell
Install-WindowsFeature RSAT-ADDS
```

### ابزارهای مدیریت Exchange

**نرم‌افزارهای موردنیاز:**
- نسخه پشتیبانی‌شده .NET Framework
- Visual C++ Redistributable برای Visual Studio 2012

**نصب ویژگی‌های ویندوز:**
- برای سیستم‌عامل سرور:
```powershell
Install-WindowsFeature -Name Web-Mgmt-Console, Web-Metabase
```
- برای سیستم‌عامل کلاینت:
```powershell
Enable-WindowsOptionalFeature -Online -FeatureName IIS-ManagementConsole, IIS-Metabase -All
```

### نقش Mailbox

**نرم‌افزارهای موردنیاز:**
- نسخه پشتیبانی‌شده .NET Framework
- Visual C++ Redistributable برای Visual Studio 2012 و 2013

**نصب ویژگی‌های ویندوز:**
```powershell
Install-WindowsFeature Server-Media-Foundation, NET-Framework-45-Core, NET-Framework-45-ASPNET, NET-WCF-HTTP-Activation45, NET-WCF-Pipe-Activation45, NET-WCF-TCP-Activation45, NET-WCF-TCP-PortSharing45, RPC-over-HTTP-proxy, RSAT-Clustering, RSAT-Clustering-CmdInterface, RSAT-Clustering-Mgmt, RSAT-Clustering-PowerShell, WAS-Process-Model, Web-Asp-Net45, Web-Basic-Auth, Web-Client-Auth, Web-Digest-Auth, Web-Dir-Browsing, Web-Dyn-Compression, Web-Http-Errors, Web-Http-Logging, Web-Http-Redirect, Web-Http-Tracing, Web-ISAPI-Ext, Web-ISAPI-Filter, Web-Metabase, Web-Mgmt-Console, Web-Mgmt-Service, Web-Net-Ext45, Web-Request-Monitor, Web-Server, Web-Stat-Compression, Web-Static-Content, Web-Windows-Auth, Web-WMI, Windows-Identity-Foundation, RSAT-ADDS
```
- برای Server Core:
```powershell
Install-WindowsFeature Server-Media-Foundation, NET-Framework-45-Core, NET-Framework-45-ASPNET, NET-WCF-HTTP-Activation45, NET-WCF-Pipe-Activation45, NET-WCF-TCP-Activation45, NET-WCF-TCP-PortSharing45, RPC-over-HTTP-proxy, RSAT-Clustering, RSAT-Clustering-CmdInterface, RSAT-Clustering-PowerShell, WAS-Process-Model, Web-Asp-Net45, Web-Basic-Auth, Web-Client-Auth, Web-Digest-Auth, Web-Dir-Browsing, Web-Dyn-Compression, Web-Http-Errors, Web-Http-Logging, Web-Http-Redirect, Web-Http-Tracing, Web-ISAPI-Ext, Web-ISAPI-Filter, Web-Metabase, Web-Mgmt-Service, Web-Net-Ext45, Web-Request-Monitor, Web-Server, Web-Stat-Compression, Web-Static-Content, Web-Windows-Auth, Web-WMI, RSAT-ADDS
```

### نقش Edge Transport

**نرم‌افزارهای موردنیاز:**
- نسخه پشتیبانی‌شده .NET Framework
- Visual C++ Redistributable برای Visual Studio 2012

**نصب ویژگی‌های ویندوز:**
```powershell
Install-WindowsFeature ADLDS
```
