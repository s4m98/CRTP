# General
📝 **Note:** import PowerView.ps1 script;)

{% tabs %} 
{% tab title="PowerView" %}

### Get Current User
```powershell
Get-DomainUser
# Specific user by identity
Get-DomainUser -Identity student1
```
### Expand a Single Property
```powershell
Get-DomainUser | Select-Object -ExpandProperty samaccountname
```
### Select Multiple Properties (without expand)
```powershell
Get-DomainUser | Select-Object samaccountname, logoncount
```
### Get current domain
```powershell
Get-Domain
```
### Get object of another domain
```powershell
Get-Domain -Domain moneycorp.local
```
### Get domain SID for the current domain
```powershell
Get-DomainSID
```
### Get domain policy for the current domain
```powershell
Get-DomainPolicyData (Get-DomainPolicyData).systemaccess
```
### Get domain policy for another domain
```powershell
(Get-DomainPolicyData -domain moneycorp.local).systemaccess
```
### Get domain controllers for the current domain
```powershell
Get-DomainController
```
### Get domain controllers for another domain
```powershell
Get-DomainController -Domain moneycorp.local
```
### Get a list of users in the current domain
```powershell
Get-DomainUser Get-DomainUser -Identity student1
```
### Get list of all properties for users in the current domain
```powershell
Get-DomainUser -Identity student1 -Properties * Get-DomainUser -Properties samaccountname,logonCount
```
📝 **Note:**
If the logon count and the bad password count of a user is tending to 0 it might be a decoy account. If the password last set of a user was also long back it might be a decoy account.

### Search for a particular string in a user’s attributes:
```powershell
Get-DomainUser -LDAPFilter "Description=*built*" | Select name,Description
```
### Get a list of computers in the current domain
```powershell
Get-DomainComputer | select Name Get-DomainComputer -OperatingSystem "*Server 2022*" Get-DomainComputer -Ping
```
### Get all the groups in the current domain
```powershell
Get-DomainGroup | select Name Get-DomainGroup -Domain <targetdomain>
Get-ADGroup -Identity "Domain Admins"
Get-ADGroupMember -Identity "Domain Admins"
Get-ADGroupMember -Identity "Enterprise Admins"
Get-ADGroupMember -Identity "Administrators"
```
### Get all groups containing the word “admin” in group name
```powershell
Get-DomainGroup *admin*
```
### Get all the members of the Domain Admins group
```powershell
Get-DomainGroupMember -Identity "Domain Admins" -Recurse
```
### Get the group membership for a user:
```powershell
Get-DomainGroup -UserName "student27"
```
### List all the local groups on a machine (needs administrator privs on non-dc machines) :
```powershell
Get-NetLocalGroup -ComputerName dcorp-dc
```
### Get members of the local group “Administrators” on a machine (needs administrator privs on non-dc machines) :
```powershell
Get-NetLocalGroupMember -ComputerName dcorp-dc -GroupName Administrators
```
### Get actively logged users on a computer (needs local admin rights on the target)
```powershell
Get-NetLoggedon -ComputerName dcorp-adminsrv
```
### Get locally logged users on a computer (needs remote registry on the target - started by-default on server OS)
```powershell
Get-LoggedonLocal -ComputerName dcorp-adminsrv
```
### Get the last logged user on a computer (needs administrative rights and remote registry on the target)
```powershell
Get-LastLoggedOn -ComputerName dcorp-adminsrv
```
{% endtab %} 

{% tab title="AD Module" %}

Enumerate all the users in the current domain using the ADModule:
**Note:** First Import AD Module DLL file using this command
```powershell
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
Import-Module C:\AD\Tools\ADModule-master\Microsoft.ActiveDirectory.Management.dll
Import-Module C:\AD\Tools\ADModule-master\ActiveDirectory\ActiveDirectory.psd1

Get-ADUser -Filter *
```
{% endtab %} 
{% endtabs %}


### Tasklist

```powershell
tasklist /svc
:: Search for specific string
tasklist /V | findstr svcadmin
```

### ENV variables

```powershell
set
ls env:
$env:computername
$env:username
```

### Detailed OS Information

```batch
systeminfo
systeminfo | findstr /B /C:"OS Name" /C:"OS Version" 
```

### Patches and Updates

{% tabs %}
{% tab title="Cmd" %}
```batch
wmic qfe get Caption,Description,HotFixID,InstalledOn 
```
{% endtab %}

{% tab title="PowerShell" %}
```powershell
Get-HotFix | ft -AutoSize
```
{% endtab %}
{% endtabs %}

### Kllist

```powershell
# Get all the key list
klist
# Delete all the keys
klist purge
# Opsec safe klist. Use with loader
Rubeus.exe klist
```

### Understand Powershell or CMD

Run `asdfgh` or any arbitary string that is not a command.

* In CMD, you'll get something like:\
  `'asdfgh' is not recognized as an internal or external command...`
* In PowerShell, you'll get:\
  `The term 'asdfgh' is not recognized as...`
