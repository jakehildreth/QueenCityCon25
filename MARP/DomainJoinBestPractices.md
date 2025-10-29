---
marp: true
theme: default
paginate: true
class: 
  - invert
---

## The Old **Insecure** Way

### Creating Computer Accounts During Join
- User joins computer without pre-provisioned account
- System creates account on-the-fly using `Ldap_Add()` operation
- Relies on "Add workstations to domain" right (`SeMachineAccountPrivilege`)

### Why This Is Problematic:
- Known security vulnerabilities (CVE-2021-42291)
- Lacks proper oversight and control
- No audit trail for account creation

⚠️ **Microsoft does NOT recommend this approach**

---

## A Better Alternative: Computer Object Reuse

### The Recommended Workflow:
1. **Admin01** Pre-creates the Computer object in a target OU/container
2. **Admin02** Performs domain join using existing account
   - System uses `Ldap_modify()` operation (not create)
3. Computer successfully joins with minimal permissions required

---

## What Gets Updated During Join:
- `samAccountName` (if name changes)
- `dnsHostName` (if not set)
- `servicePrincipalName` (if not set)
- `userAccountControl` (enable if disabled)
- `unicodePwd` (computer password)

---

## Required Permissions - Admin01
- Computer account creator is a member of the Administrators group
  **OR**
- Computer account creator is listed in `ComputerAccountReuseAllowlist` GPO (KB5020276)

---

## Required Permissions for Admin02
- Read (All Properties, List Contents)
- Allowed to authenticate
- Change password
- Reset password
- Write on the `userAccountControl` attribute
- Validated Write to the `dnsHostName` attribute
- Validated Write to `servicePrincipalName` attribute

#### Additional Permissions for Rename:
Write on `name`, `displayName`, and `description` attributes

---

## The Best Alternative: Offline Domain Join
- Provision a new computer object:
`djoin /provision /domain contoso.com /machine computer1 /savefile offlinedomainjoin.txt`
- On physical computer, complete the join:
`djoin /requestODJ /loadfile offlinedomainjoin.txt /windowspath %SystemRoot% /localos`

#### Additional benefits:
- Can be performed without line of sight to DC.
- Requires **ZERO** permissions on computer account.

