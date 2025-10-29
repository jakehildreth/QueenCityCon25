## Opening Hook
Start with the shocking reality: "In most Active Directory environments, any authenticated user can create 10 computer accounts. Let me show you how that turns into Domain Admin in under 5 minutes."

---
## Intro Slides

## Act 1: The Setup (5 minutes) 

### The Problem - Jake
- MachineAccountQuota exists in 99% of AD environments
- Default value: 10 computers per user
- Originally designed 25 years ago when computers and (computer objects) were considered more secure than users
- Today: Computer accounts are MORE dangerous than user accounts

### Why This Matters - John
- Computer accounts have unique privileges
- They're trusted differently by AD
- They're rarely scrutinized like user accounts
- Most organizations don't even know this setting exists

---

## Act 2: Understanding the Landscape (8 minutes)

### What Makes Computer Accounts Special - Jake
- Member of Domain Computers group
- Can have Service Principal Names (SPNs)
- Can be delegated
- Different password rules

### The Attacker's Advantage -John
- Easy to create: `net computer /add` or PowerShell
- Less scruntinized overall
- Rarely trigger alerts
- Perfect for persistence

### Real-World Context - Jake
- How common is this? (Statistics)
- Who is vulnerable?
- Why hasn't this been fixed?

---

## Act 3: The Exploitation Playbook (20 minutes) - John

**Note:** This is the heart of the talk - show multiple attack paths, each building on the last
### Attack 1: Resource-Based Constrained Delegation
- The concept: Delegating authentication on behalf of others
- The attack: Create computer, set msDS-AllowedToActOnBehalfOfOtherIdentity
- The impact: Compromise any service, escalate to admin
- **Demo:** Live walkthrough

### Attack 2: ACL Abuse
- Hidden permissions granted to Domain Computers group
- GenericAll, WriteDACL on sensitive objects
- Finding these misconfigurations
- **Demo:** Exploitation for privilege escalation

### Attack 3: Certificate Services Exploitation
- Many ADCS templates allow Domain Computers to enroll
- ESC14: Manipulating altSecurityIdentities 
- Certificate-based authentication = long-term access
- **Demo:** Certificate request and authentication

### Attack Path 4: Advanced Techniques (Rapid Fire)
- SPN-in-the-Middle attacks: Novel MITM using computer SPNs
- Stale computer backdoors: Persistence through abandoned accounts
- Unconstrained delegation scenarios
- Silver ticket attacks with computer credentials
- Using DumpGuard from SepcterOps to 

### The Domino Effect
- How these attacks chain together
- One computer account = multiple paths to Domain Admin
- Why defenders are playing catch-up

---

## Act 4: Fighting Back (10 minutes) - Jake

### Detection
- What to monitor: Event IDs 4741, 4742, 4743
- Behavioral indicators of abuse
- Certificate enrollment anomalies
- Unusual SPN registrations

### Prevention
- The nuclear option: Set MachineAccountQuota to 0
- GPO-based option: 
- The right way: Domain join hardening guidance from MS
- Compensating controls when you can't change it yet
- Template hardening, ACL cleanup, delegation policies

### Remediation
- Audit current state
- Clean up stale computer accounts
- Implement proper controls
- Ongoing monitoring and maintenance

---

## Closing (2 minutes)

### Key Takeaways
- MachineAccountQuota is a 25-year-old design decision that's now a critical vulnerability
- Computer accounts are more dangerous than most people realize
- There are multiple attack paths from "create computer" to "domain admin"
- Defense requires both technical controls AND organizational change

### Call to Action
- Check your environment TODAY
- Start the conversation with your security and identity teams
- Don't wait for an incident to fix this

### Q&A
- Open floor for questions
