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
## Follow New Domain Join Guidance
1. **Admin01** Pre-creates the Computer object in a target OU/container
2. **Admin02** Performs domain join using existing account
   - System uses `Ldap_modify()` operation (not create)
3. Computer successfully joins with minimal permissions required
<br>

<sub>More details, including required permissions for each admin:
https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/active-directory-domain-join-permissions</sub>

---

# Prevention:
## Follow Alternate Domain Join Guidance (more secure!)
1. On Tier 0 device, provision a new computer object:
```
djoin /provision /domain contoso.com /machine NewComputer /savefile offlinedomainjoin.txt
```
2. On physical computer, complete the join:
```
djoin /requestODJ /loadfile offlinedomainjoin.txt /windowspath %SystemRoot% /localos
```
<br>

<sub>More details:
https://learn.microsoft.com/en-us/windows-server/remote/remote-access/directaccess/directaccess-offline-domain-join</sub>

---

# Remediation:



---

# Detection
