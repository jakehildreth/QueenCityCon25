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

![bg right:60% 60%](1650739011354.jpeg)
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
<!-- John -->

---

# Real-World Context
- How common is this?
  * Jake: 80% at default
  * John: never seen it set properly when first engaging a customer
- Why hasn't this been fixed?
  * Relatively outside security circles
  * Conflicting hardening guidance
  * Operations > Security

<!-- Jake -->

---

# **A Demo or Two**
<!-- JOHN DEMOS HERE -->

---

# **Some Solutions**

<!-- Jake -->

---

# Prevention:
## Set `ms-DS-MachineAccountQuota` to 0
```powershell
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
1. On Tier 0 device, using a Trusted Computer Account Owner, provision a new computer object:
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
Huy Kha (aka DebugPrivilege) wrote an article with an easy-to-use script that finds computer accounts tha look *funky* 
![](image-2.png)
https://medium.com/@Debugger/machines-gone-rogue-a01d726f5f10

---

# Detection
## Monitor
