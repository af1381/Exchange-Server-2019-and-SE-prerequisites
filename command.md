# 🛠️📬 Exchange 2019 & Active Directory — Admin README

A concise, copy‑paste friendly guide for common **Exchange Server 2019** tasks and **AD password policy** management. Examples use the domain **hse.local** and user **`a.fathi`**.

> ℹ️ Password aging/complexity is **controlled by Active Directory**, not Exchange. Exchange just honors AD policies.

---

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Open the Right Consoles](#open-the-right-consoles)
3. [Exchange Databases](#exchange-databases)
4. [Mailbox & Message Size Limits](#mailbox--message-size-limits)
5. [Enable Mailboxes for Users in an OU](#enable-mailboxes-for-users-in-an-ou)
6. [AD Password Policy (Domain-wide)](#ad-password-policy-domain-wide)
7. [Unlimited (Never Expire) Passwords](#unlimited-never-expire-passwords)
8. [Per-OU Strategy with FGPP (Optional)](#per-ou-strategy-with-fgpp-optional)
9. [Force First Login Change, Then Never Expire (Flow)](#force-first-login-change-then-never-expire-flow)
10. [Quick User Checks (a.fathi)](#quick-user-checks-afathi)
11. [Good Practices](#good-practices)

---

## Prerequisites
- Run **Exchange cmdlets** in **Exchange Management Shell (EMS)**.
- Run **AD cmdlets** on a **Domain Controller** or a Windows admin host with **RSAT** and the **ActiveDirectory** module.
- Perform disruptive actions (e.g., moving DB paths) in a maintenance window.

---

## Open the Right Consoles
- **Exchange Management Shell:** Start Menu → Exchange Management Shell
- **Exchange Admin Center (EAC):** https://\<Exchange-FQDN\>/ecp
- **Group Policy Management:** `Win + R` → `gpmc.msc`
- **Local Security Policy (standalone/testing):** `secpol.msc` (does **not** affect domain users)

---

## Exchange Databases

### List mailbox databases
```powershell
Get-MailboxDatabase
```

### Rename a mailbox database
```powershell
Set-MailboxDatabase -Identity "Mailbox Database 1843079470" -Name "EXC-DB-01"
```

### Move database and log paths
```powershell
Move-DatabasePath -Identity "Database-Name" `
  -EdbFilePath "D:\Exchange\DBs\Database-Name.edb" `
  -LogFolderPath "E:\Exchange\Logs\Database-Name"
```
> 💡 The cmdlet will prompt to dismount/mount the DB. Plan downtime accordingly.

---

## Mailbox & Message Size Limits

### Per-user mailbox quota
```powershell
Set-Mailbox -Identity "ahmad" `
  -IssueWarningQuota 900MB `
  -ProhibitSendQuota 950MB `
  -ProhibitSendReceiveQuota 1GB
```

### Global transport limits
```powershell
Set-TransportConfig -MaxSendSize 100MB -MaxReceiveSize 100MB
```

### All connectors
```powershell
Get-SendConnector    | Set-SendConnector    -MaxMessageSize 100MB
Get-ReceiveConnector | Set-ReceiveConnector -MaxMessageSize 100MB
```

### All mailboxes (bulk)
```powershell
Get-Mailbox -ResultSize Unlimited | Set-Mailbox -MaxSendSize 100MB -MaxReceiveSize 100MB
```

### Report per-mailbox send/receive limits
```powershell
Get-Mailbox -ResultSize Unlimited | Select-Object Name, MaxSendSize, MaxReceiveSize
```

### Limit a specific user
```powershell
Set-Mailbox -Identity "user@domain.local" -MaxSendSize 15MB -MaxReceiveSize 15MB
```

---

## Enable Mailboxes for Users in an OU
```powershell
Get-User -OrganizationalUnit "oman" | Enable-Mailbox -Database "db-name"
```

---

## AD Password Policy (Domain-wide)

> 📚 Password age/complexity is AD-level. Exchange does **not** set password retention itself.

### Set **Maximum password age = 180 days** (domain: hse.local)
```powershell
Import-Module ActiveDirectory

Set-ADDefaultDomainPasswordPolicy -Identity "hse.local" `
  -MaxPasswordAge (New-TimeSpan -Days 180)

Get-ADDefaultDomainPasswordPolicy -Identity "hse.local" |
  Format-List MaxPasswordAge, MinPasswordAge, PasswordHistoryCount, MinPasswordLength, ComplexityEnabled
```
- Effective immediately (after AD replication). Expiry = **PasswordLastSet + 180 days**.

---

## Unlimited (Never Expire) Passwords

### Domain-wide (NOT recommended for most orgs)
```powershell
Import-Module ActiveDirectory
Set-ADDefaultDomainPasswordPolicy -Identity "hse.local" -MaxPasswordAge ([TimeSpan]::Zero)

# Verify
Get-ADDefaultDomainPasswordPolicy -Identity "hse.local" | Format-List MaxPasswordAge
# Expected: 0.00:00:00
```
> ⚠️ If any **Fine-Grained Password Policy (FGPP/PSO)** applies to a user, that PSO overrides the domain policy.

### Per-user (toggle “Password never expires”)
```powershell
Set-ADUser jdoe -PasswordNeverExpires $true -ChangePasswordAtLogon $false
```

---

## Per-OU Strategy with FGPP (Optional)

PSOs don’t link to OUs; apply them to **groups**. Example for OU **HesabaUsers** (set to 180 days or unlimited by PSO).

```powershell
Import-Module ActiveDirectory

$OuName    = "HesabaUsers"
$GroupName = "GG-HesabaUsers-Policy"
$PsoName   = "PSO-HesabaUsers"

# Find OU DN
$ou = Get-ADOrganizationalUnit -LDAPFilter "(name=$OuName)"
if (!$ou) { throw "OU '$OuName' not found." }
$SearchBase = $ou.DistinguishedName

# Ensure group exists
if (-not (Get-ADGroup -Filter "Name -eq '$GroupName'" -ErrorAction SilentlyContinue)) {
  New-ADGroup -Name $GroupName -GroupScope Global -GroupCategory Security
}

# Add all users from the OU to the group
$ouUsers = Get-ADUser -Filter * -SearchBase $SearchBase -SearchScope Subtree
Add-ADGroupMember -Identity $GroupName -Members $ouUsers -ErrorAction SilentlyContinue

# Create/Update PSO (choose ONE MaxPasswordAge line)

# A) 180 days
if (-not (Get-ADFineGrainedPasswordPolicy -Filter "Name -eq '$PsoName'" -ErrorAction SilentlyContinue)) {
  New-ADFineGrainedPasswordPolicy `
    -Name $PsoName -Precedence 20 `
    -MaxPasswordAge (New-TimeSpan -Days 180) `
    -MinPasswordAge (New-TimeSpan -Days 1) `
    -PasswordHistoryCount 24 `
    -MinPasswordLength 12 `
    -ComplexityEnabled $true `
    -ReversibleEncryptionEnabled $false `
    -LockoutThreshold 10 `
    -LockoutDuration (New-TimeSpan -Minutes 15) `
    -LockoutObservationWindow (New-TimeSpan -Minutes 15)
} else {
  Set-ADFineGrainedPasswordPolicy $PsoName -MaxPasswordAge (New-TimeSpan -Days 180)
}

# B) Unlimited (Never expire)
# Set-ADFineGrainedPasswordPolicy $PsoName -MaxPasswordAge ([TimeSpan]::Zero)

# Link PSO to the group
Add-ADFineGrainedPasswordPolicySubject -Identity $PsoName -Subjects $GroupName -ErrorAction SilentlyContinue
```

> 🧠 If multiple PSOs apply, **lower Precedence number wins**.

---

## Force First Login Change, Then Never Expire (Flow)

1) **Force change at next logon** for all enabled users in OU (keep default passwords from lingering):
```powershell
Import-Module ActiveDirectory
$SearchBase = "OU=HesabaUsers,DC=hesaba,DC=net"

Get-ADUser -Filter * -SearchBase $SearchBase -SearchScope Subtree -Properties Enabled |
  Where-Object { $_.Enabled -eq $true } |
  ForEach-Object {
    Set-ADAccountControl $_ -CannotChangePassword $false
    Set-ADUser $_ -ChangePasswordAtLogon $true -PasswordNeverExpires $false
  }
```

2) **After** users change their password (pwdLastSet ≠ 0), make them **never expire**:
```powershell
Import-Module ActiveDirectory
$SearchBase = "OU=HesabaUsers,DC=hesaba,DC=net"

Get-ADUser -Filter * -SearchBase $SearchBase -SearchScope Subtree -Properties pwdLastSet, PasswordNeverExpires |
  Where-Object { $_.pwdLastSet -ne 0 -and $_.PasswordNeverExpires -ne $true } |
  Set-ADUser -PasswordNeverExpires $true -ChangePasswordAtLogon $false
```

---

## Quick User Checks (a.fathi)

### Classic check
```powershell
net user a.fathi /domain
```

### Deep AD check
```powershell
Import-Module ActiveDirectory

Get-ADUser a.fathi -Properties `
  Enabled,PasswordLastSet,pwdLastSet,PasswordNeverExpires, `
  "msDS-UserPasswordExpiryTimeComputed","msDS-ResultantPSO" |
Select-Object SamAccountName, Enabled,
  @{N='MustChangeAtNextLogon';E={ $_.pwdLastSet -eq 0 }},
  PasswordNeverExpires, PasswordLastSet,
  @{N='ExpiresOn';E={
      if ($_.PasswordNeverExpires) { 'Never (PasswordNeverExpires=True)' }
      elseif ($_."msDS-UserPasswordExpiryTimeComputed") { [datetime]::FromFileTime($_."msDS-UserPasswordExpiryTimeComputed") }
      else { 'Never (MaxPasswordAge=0 or PSO=0)' }
  }},
  @{N='ExpiresInDays';E={
      if (-not $_.PasswordNeverExpires -and $_."msDS-UserPasswordExpiryTimeComputed") {
        [int](([datetime]::FromFileTime($_."msDS-UserPasswordExpiryTimeComputed") - (Get-Date)).TotalDays)
      }
  }},
  "msDS-ResultantPSO"
```

---

## Good Practices
- 🔐 Keep **Complexity** enabled and **MinPasswordLength ≥ 12**.
- 🛡️ Configure **Account Lockout** (threshold/duration/observation window) to mitigate brute force.
- 🧪 After any change, verify with:
```powershell
Get-ADDefaultDomainPasswordPolicy -Identity "hse.local" |
  Format-List MaxPasswordAge, MinPasswordAge, PasswordHistoryCount, MinPasswordLength, ComplexityEnabled
```
- 🔁 Allow AD replication time across DCs before validating everywhere.
