# Gitea LDAP Integration & RBAC Configuration

## Objective

Integrate Gitea with Active Directory using LDAP authentication and implement Role-Based Access Control (RBAC) using AD Security Groups.

This ensures:

- Centralized authentication
- No local password management
- Enterprise-style role separation
- Identity-driven DevSecOps governance

---

# Identity Architecture Overview

Domain:
lab.internal

Base DN:
DC=lab,DC=internal

OU Structure:

```

OU=lab_internal
├── OU=Users
├── OU=Groups
└── OU=ServiceAccounts

````

Gitea integrates with Active Directory through a dedicated service account.

---

# Create Dedicated LDAP Service Account

## Why Not Use Administrator?

Using Administrator for LDAP binding is insecure.

Enterprise best practice:
- One service = one service account
- Least privilege
- No admin rights
- Easy credential rotation

---

## PowerShell – Create Service Account

Run on Domain Controller:

```powershell
$UserName = "svc_gitea_ldap"
$Password = ConvertTo-SecureString "Str0ngP@ssw0rd!" -AsPlainText -Force
$OU = "OU=ServiceAccounts,OU=lab_internal,DC=lab,DC=internal"
$Domain = "lab.internal"

New-ADUser `
 -Name "svc_gitea_ldap" `
 -SamAccountName $UserName `
 -UserPrincipalName "$UserName@$Domain" `
 -Path $OU `
 -AccountPassword $Password `
 -Enabled $true `
 -PasswordNeverExpires $true `
 -ChangePasswordAtLogon $false `
 -Description "Service Account for Gitea LDAP Bind"
````

Verify:

```powershell
Get-ADUser $UserName -Properties Enabled,PasswordNeverExpires,DistinguishedName
```

---

# Create Gitea RBAC Groups

Two security groups were created:

* Gitea_Admin
* Gitea_Analyst

These groups control application-level access.

---

## PowerShell – Create Groups

```powershell
$GroupsOU = "OU=Groups,OU=lab_internal,DC=lab,DC=internal"

New-ADGroup `
 -Name "Gitea_Admin" `
 -SamAccountName "Gitea_Admin" `
 -GroupScope Global `
 -GroupCategory Security `
 -Path $GroupsOU `
 -Description "Administrators for Gitea Platform"

New-ADGroup `
 -Name "Gitea_Analyst" `
 -SamAccountName "Gitea_Analyst" `
 -GroupScope Global `
 -GroupCategory Security `
 -Path $GroupsOU `
 -Description "Read/Analyst access for Gitea"
```

---

## Add Users to Groups

```powershell
Add-ADGroupMember -Identity "Gitea_Admin" -Members labadmin
Add-ADGroupMember -Identity "Gitea_Analyst" -Members labanalyst
```

Verify:

```powershell
Get-ADGroupMember "Gitea_Admin"
Get-ADGroupMember "Gitea_Analyst"
```

---

# Configure LDAP in Gitea

Login to Gitea as local admin.

Navigate:

Site Administration → Authentication Sources → Add LDAP (via BindDN)

Use the following configuration:

| Setting              | Value                                                             |
| -------------------- | ----------------------------------------------------------------- |
| Authentication Name  | ActiveDirectory-LDAP                                              |
| Security Protocol    | Unencrypted                                                       |
| Host                 | 172.16.1.10                                                       |
| Port                 | 389                                                               |
| Bind DN              | [svc_gitea_ldap@lab.internal]                                     |
| Bind Password        | (Service Account Password)                                        |
| User Search Base     | DC=lab,DC=internal                                                |
| User Filter          | (&(objectClass=user)(sAMAccountName=%s))                          |
| Username Attribute   | sAMAccountName                                                    |
| First Name Attribute | givenName                                                         |
| Surname Attribute    | sn                                                                |
| Email Attribute      | mail                                                              |

Save configuration.

---

# Validate LDAP Authentication

1. Log out of Gitea.
2. Log in using AD credentials.

Example:

Username:
labadmin

Password:
AD password

If successful:

* User auto-creates in Gitea
* Authentication now controlled by AD

---

# Optional – Admin Filter (RBAC Enforcement)

To automatically grant Gitea admin privileges only to members of Gitea_Admin:

Use Admin Filter:

```
(memberOf=CN=Gitea_Admin,OU=Groups,OU=lab_internal,DC=lab,DC=internal)
```

This enforces:

Only members of AD group Gitea_Admin become administrators in Gitea.

Enterprise RBAC in action.
