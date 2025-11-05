---
marp: true
class:
  - invert
transition: fade-out
---

# Making $ With COMPUTER$
## Queen City Con 0x3
### John Askew & Jake Hildreth
#### 2025-11-07

<!-- Both -->

---

![bg right:60% 60%](images/john.png)
# John
Hacker
Red Team Lead

<!-- John -->

---

# Jake
![bg right:60% 60% blur: 20px](1719237510669.jpeg)
Defender
Security Consultant

<!-- Jake -->

---

# Agenda
![bg right:60% 90%](<Pasted image 20250705093555.png>)
- The Problem
- Some Context
- A Demo or Two
- Some Solutions

<!-- John -->

---

# **The Problem**

---
<!-- _transition: none -->

## If a user can join a computer to your domain
## they can own your Active Directory forest

<!-- Jake -->

---
## If a user can join a computer to your domain
## they can own your Active Directory forest in **minutes**

<!-- John -->

---

# The Problem
- `ms-DS-MachineAccountQuota` attribute
  * Number of computers a single user can add to the domain
  * Active Directory (AD) default value: 10
- `SeMachineAccountPrivilege` User Right
  * Who is allowed to add computers to a domain
  * AD default: Authenticated Users
* Made sense 25 years ago
* But now computers are more dangerous!

<!-- Jake -->

---

# Why This Matters:
* MACHINE$ accounts are valuable to attackers!
* MANY attack chains need an attacker-controlled machine account
* Defaults = easier to create one than to compromise one
* Remove defaults -> break attack chains
<!-- John -->

---

# **Some Context**

---

# What Makes Computer Accounts Special?
- Password differences:
  * Complex & 120 characters long
  * Changed automatically by AD every 30 days
- Service Principal Names (SPNs)
  * Tells others what services are available
* Note: local admin password is NOT the computer account password
<!-- Jake -->

---

# The Attacker's Advantage
- Machine accounts...
  * tend to be less scrutinized (evasion, persistence)
  * often have different permissions (privilege escalation)
  * can be created without creds using relaying (initial access)
* Controlling an SPN is a powerful attack primitive!
<!-- John -->

---

# Real-World Experience
- How common is this?
  * Jake: 80% at default
  * John: never seen it set properly when first engaging a customer
- Why hasn't this been fixed?
  * Relatively unknown outside security circles
  * Conflicting hardening guidance
  * Operations > Security

<!-- Jake & John -->

---

# **A Demo or Two or Three**
<!-- JOHN DEMOS HERE -->

---

# ACL Abuse
* Look for extra permissions granted to Domain Computers
  * Added to privileged groups?
  * Access to other AD objects via ACLs?
* Create a machine account and you can abuse those permissions
<!-- John -->

---

# RBCD Abuse
* "Resource-Based Constrained Delegation"
* Turn a "GenericWrite" permission on a computer object into a full compromise
* https://eladshamir.com/2019/01/28/Wagging-the-Dog.html
<!-- John -->

---

# ADCS Abuse (DEMO)
* Active Directory Certificate Services is EASY to misconfigure
* Domain Computers are often allowed to enroll templates (ESC1)
* Turn `altSecurityIdentities` write access into a full compromise (ESC14A)
  - (The default Domain Computers cert template meets the other attack requirements)
* References:
  - https://posts.specterops.io/certified-pre-owned-d95910965cd2
  - https://posts.specterops.io/adcs-esc14-abuse-technique-333a004dc2b9
<!-- John -->

---

# Other Attacks
* Persistence using stale machine accounts
* Unconstrained Delegation (need SeEnableDelegationPrivilege)
* "SPN-in-the-Middle"
* "DumpGuard" scenario
<!-- John -->

---

# **Some Solutions**

<!-- Jake -->

---

# Prevention:
## Set `ms-DS-MachineAccountQuota` to 0
```powershell
#requires -Modules ActiveDirectory

# Set variables
$MAQ = 'ms-DS-MachineAccountQuota'
$Domain = Get-ADDomain -Identity example.com

# Set Correct Value
Set-ADDomain -Identity $Domain -Replace @{$MAQ=0}
```
Now only Administrators can add computers to domain without first precreating a computer account.

---
# Prevention:
## Restrict `SeMachineAccountPrivilege`
![h:450](image.png)

---

# Prevention:
## Follow New Domain Join Guidance: Prepare
Configure Trusted Computer Account Owners:
![h:450](image-1.png)

---

# Prevention:
## Follow New Domain Join Guidance: Perform
1. Admin01, *a Trusted Computer Account Owner,* pre-creates the Computer object in a target OU/container
2. Admin02, *a Computer Joiner,* performs domain join w/minimal privileges required
3. Admin02's permissions are removed from the computer account.
<br>

<sub>More details, including required permissions for each admin:
https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/active-directory-domain-join-permissions</sub>

---

# Prevention:
## Offline Domain Join (more secure!)
1. Using a _Trusted Computer Account Owner_ with appropriate permissions, provision a new computer object:
    ```
    djoin /provision /domain contoso.com /machine NewComputer /savefile offlinedomainjoin.txt
    ```
2. On physical computer, complete the join:
    ```
    djoin /requestODJ /loadfile offlinedomainjoin.txt /windowspath %SystemRoot% /localos
    ```

### No additional permissions are required!
<sub>More details:
https://learn.microsoft.com/en-us/windows-server/remote/remote-access/directaccess/directaccess-offline-domain-join</sub>

---

# Remediation:
## Find Inactive and Suspicious Computer Objects:
Huy Kha (aka DebugPrivilege) wrote an article with an easy-to-use script that finds computer accounts that look *funky*
![](image-2.png)
https://medium.com/@Debugger/machines-gone-rogue-a01d726f5f10

---

![bg right 75%](images/event_id_4741.png)

# Detection
## Monitor for New Machine Accounts
- Event ID 4741 is logged on Domain Controllers
* (you ARE collecting logs from ALL your domain controllers... right!?)

---

# Detection
## Investigate Machine Accounts
- The machine account creator is indicated in the `mS-DS-CreatorSID` field
- By default the user account cannot delete the machine account it created
- https://learn.microsoft.com/en-us/windows/win32/adschema/a-ms-ds-creatorsid
![](images/adexplorer_msds-creatorsid.png)

---

# **Outro** 🎸

---

# Key Takeaways
- Design decisions from 25 years ago aren't always so great
- Computer accounts are more dangerous than most people realize
- There are multiple attack paths from COMPUTER$ to $
- Defense requires technical controls AND organizational change

---

# Call to Action

- Check your environment TODAY
- Start the conversation with your security and identity teams
- Don't wait for an incident to fix this

---

<style scoped>
div.twocols {
  margin-top: 35px;
  column-count: 2;
}
div.twocols p:first-child,
div.twocols h1:first-child,
div.twocols h2:first-child,
div.twocols ul:first-child,
div.twocols ul li:first-child,
div.twocols ul li p:first-child {
  margin-top: 0 !important;
}
div.twocols p.break {
  break-before: column;
  margin-top: 0;
}
</style>

# Thanks
<div class="twocols">

## John Askew
- item
- item
- item
- item

## Jake Hildreth
- item
- item
- item
- item
</div>

---

# Thanks!

| | John | Jake |
|-|-|-|
| Email | john@terrapinlabs.io | jake@jakehildreth.com |
| Web | terrapinlabs.io | jakehildreth.com |
| GitHub | sk3w | jakehildreth |
| LinkedIn | /in/sk3w | /in/jakehildreth |
| BlueSky | @sk3w.bsky.social | @dotdot.horse |
| QR (if you trust us 😉) | | ![w:175](image-3.png) |
