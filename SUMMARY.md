# Table of contents

* 🛠️ [CRTP Notes](README.md)
* ⚙️ [CRTP Methodology](crtp-methodology.md)

## 💡 Misc

* [PowerShell Basics](misc/powershell-basics.md)
* [Bypass Defenses](misc/bypass-defenses.md)
* [Transfer Files](misc/transfer-files.md)

## 🔨 Basic Enumeration

* [General](basic-enumeration/general.md)
* [Network](basic-enumeration/network.md)
* [Kerberos Authentication](basic-enumeration/kerberos-authentication.md)
* [AD-Methodology-Master](basic-enumeration/AD-Methodology-Master.md)
  
## ⛏️ AD Enumeration

* [General](ad-enumeration/general.md)
* [BloodHound & SOAPHound](ad-enumeration/bloodhound-and-soaphound.md)
* [ACL](ad-enumeration/acl.md)
* [GPO & OU](ad-enumeration/gpo-and-ou.md)
* [Forests and Trusts](ad-enumeration/forests-and-trusts.md)
* [User Hunting](ad-enumeration/user-hunting.md)

## 🔪 Privilege Escalation

* [Local Privilege Escalation](privilege-escalation/local-privilege-escalation.md)
* [Domain Privilege Escalation](privilege-escalation/domain-privilege-escalation/README.md)
  * [Kerberoast](privilege-escalation/domain-privilege-escalation/kerberoast.md)
  * [AS-REP Roasting](privilege-escalation/domain-privilege-escalation/as-rep-roasting.md)
  * [Delegations](privilege-escalation/domain-privilege-escalation/delegations.md)

* [Cross Domain Privilege Escalation](privilege-escalation/cross-domain-privilege-escalation/README.md)
  * [Trusts](privilege-escalation/cross-domain-privilege-escalation/trusts.md)
  * [AD CS](privilege-escalation/cross-domain-privilege-escalation/ad-cs.md)
  * [MSSQL Servers](privilege-escalation/cross-domain-privilege-escalation/mssql-servers.md)
  * [LSASS Dump](privilege-escalation/cross-domain-privilege-escalation/lsass-dump.md)

## 🏎️ Lateral Movement

* [WinRM](lateral-movement/winrm.md)
* [Credentials Dumping](lateral-movement/lsass-dump.md)
* [DC Sync](lateral-movement/dc-sync.md)
* [Over Pass The Hash](lateral-movement/over-pass-the-hash.md)
* [Runas](lateral-movement/runas.md)
* [GPO Abuse NTLM relay](lateral-movement/GPO-Abuse-NTLM-relay.md)

## 🔧 Persistence

* [Kerberos](persistence/kerberos/README.md)
  * [Golden Ticket](persistence/kerberos/golden-ticket.md)
  * [Silver Ticket](persistence/kerberos/silver-ticket.md)
  * [Diamond Ticket](persistence/kerberos/diamond-ticket.md)

* [Skeleton Key](persistence/skeleton-key.md)
* [DSRM](persistence/dsrm.md)
* [Custom SSP](persistence/custom-ssp.md)
* [AdminSDHolder](persistence/adminsdholder.md)
* [ACL – Domain Object](persistence/acl.md)
* [Security Descriptors](persistence/security-descriptors.md)

* [🛡️ Mitigations](defense-best-practices.md)
* [📚 References](references.md)
