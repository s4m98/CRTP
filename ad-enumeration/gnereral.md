# General

### Required Tools

[PowerView](https://github.com/ZeroDayLab/PowerSploit/blob/master/Recon/PowerView.ps1) or [SharpView](https://github.com/tevora-threat/SharpView)

```powershell
. C:\AD\Tools\PowerView.ps1
```

[AD module](https://github.com/samratashok/ADModule) - MS singed

```powershell
Import-Module C:\AD\Tools\ADModule-master\Microsoft.ActiveDirectory.Management.dll
Import-Module C:\AD\Tools\ADModule-master\ActiveDirectory\ActiveDirectory.psd1
```

### Domain

{% tabs %}
{% tab title="PowerView" %}
Get Current domain

<pre class="language-powershell"><code class="lang-powershell">Get-Domain
<strong># Displays the domain that the current computer is joined to
</strong>(Get-Domain).Name
</code></pre>

Get Object of another domain

```powershell
Get-Domain -Domain moneycorp.local
```

Get domain SID for the current domain

```powershell
Get-DomainSID
```

Get domain policy for the current domain

```powershell
Get-DomainPolicyData
(Get-DomainPolicyData).systemaccess
```

Get domain policy for another domain

```powershell
(Get-DomainPolicyData -domain moneycorp.local).systemaccess
```
{% endtab %}

{% tab title="AD module" %}
Get Current domain

```powershell
Get-ADDomain
```

Get Object of another domain

```powershell
Get-ADDomain -Identity moneycorp.local
```

Get domain SID for the current domain

```powershell
(Get-ADDomain).DomainSID
```
{% endtab %}
{% endtabs %}

### Domain Controller

{% tabs %}
{% tab title="PowerView" %}
Get domain controllers for the current domain

```powershell
Get-DomainController
```

Get domain controllers for another domain

```powershell
Get-DomainController -Domain moneycorp.local
```
{% endtab %}

{% tab title="AD module" %}
Get domain controllers for the current domain

```powershell
Get-ADDomainController
```

Get domain controllers for another domain

```powershell
Get-ADDomainController -DomainName moneycorp.local -Discover
```
{% endtab %}
{% endtabs %}

### Domain Users

{% tabs %}
{% tab title="PowerView" %}
Get a list of users in the current domain

```powershell
Get-DomainUser
# Limit the result to first 2
Get-DomainUser| Select-Object -First 2
Get-DomainUser -Identity student1
```

Get a list of all properties for users in the current domain

```powershell
Get-DomainUser -Identity student1 -Properties * 
Get-DomainUser -Properties samaccountname,logonCount
# Another way
Get-DomainUser | select samaccountname
```

Different filter options in a user's attributes

```powershell
Get-DomainUser -LDAPFilter "Description=*built*" | Select name,Description
# Search with patterns like Control1User, Control2User etc
Get-DomainUser -LDAPFilter "(name=Control*User)" | select name
# Check group in which user is added
Get-DomainUser  -Identity "studentx" | Select-Object -ExpandProperty memberof
# Pipe results to a loop to filter results
Get-DomainUser -LDAPFilter "(name=Control*User)" | %{Get-DomainGroup -MemberIdentity $_.name} | select samaccountname
```

Get actively logged users on a computer (requires local admin privileges)

```powershell
Get-NetLoggedon -ComputerName dcorp-adminsrv
```

Get locally logged users on a computer (requires remote registry)

```powershell
Get-LoggedonLocal -ComputerName dcorp-adminsrv
```

Get the last logged user on a computer (requires admin privileges and remote registry)

```powershell
Get-LastLoggedOn -ComputerName dcorp-adminsrv
```
{% endtab %}

{% tab title="AD module" %}
Get a list of users in the current domain

```powershell
Get-ADUser -Filter * -Properties *
Get-ADUser -Identity student1 -Properties *
```

Get list of all properties for users in the current domain

```powershell
Get-ADUser -Filter * -Properties * | select -First 1 | Get-Member -MemberType *Property | select Name
Get-ADUser -Filter * -Properties * | select name,logoncount,@{expression={[datetime]::fromFileTime($_.pwdlastset)}}
```

Search for a particular string in a user's attributes

```powershell
Get-ADUser -Filter 'Description -like "*built*"' -Properties Description | select name,Desc
```
{% endtab %}
{% endtabs %}

### Domain Computers

{% tabs %}
{% tab title="PowerView" %}
Get a list of computers in the current domain

```powershell
Get-DomainComputer | select Name
Get-DomainComputer -OperatingSystem "*Server 2022*"
Get-DomainComputer -Ping
```
{% endtab %}

{% tab title="AD Module" %}
Get a list of computers in the current domain

```powershell
Get-ADComputer -Filter * | select Name
Get-ADComputer -Filter * -Properties *
Get-ADComputer -Filter 'OperatingSystem -like "*Server 2022*"' -Properties OperatingSystem | select Name,OperatingSystem
Get-ADComputer -Filter * -Properties DNSHostName | %{Test-Connection -Count 1 -ComputerName $_.DNSHostName}
```
{% endtab %}
{% endtabs %}

### Domain Groups

{% tabs %}
{% tab title="PowerView" %}
Get all the groups

```powershell
Get-DomainGroup | select Name
Get-DomainGroup -Domain <targetdomain>
```

Get all groups containing the word "admin" in group name. _Enterprise Admins_ will not be shown in the results if it's not a forest root domain.

```powershell
Get-DomainGroup *admin*
# To get enterprise admins use -Domain <forest_root_domain>
```

Get all the members of the Domain Admins group. SID ends with 500-1000 is reserved for the domain. Any created objects will be having SID ends after 1000.

```powershell
Get-DomainGroupMember -Identity "Domain Admins" -Recurse
Get-DomainGroupMember -Identity "Enterprise Admins" -Domain moneycorp.local
```

Get the group membership for a user

```powershell
Get-DomainGroup -UserName "student1"
```
{% endtab %}

{% tab title="AD Module" %}
Get all the groups

```powershell
Get-ADGroup -Filter * | select Name 
Get-ADGroup -Filter * -Properties *
```

Get all groups containing the word "admin" in group name

```powershell
Get-ADGroup -Filter 'Name -like "*admin*"' | select Name
```

Get all the members of the Domain Admins group

```powershell
Get-ADGroupMember -Identity "Domain Admins" -Recursive
```

Get the group membership for a user

```powershell
Get-ADPrincipalGroupMembership -Identity student1
```
{% endtab %}
{% endtabs %}

### Local Groups

{% tabs %}
{% tab title="PowerView" %}
List all the local groups on a machine (requires admin privileges)

```powershell
Get-NetLocalGroup -ComputerName dcorp-dc
```

Get members of the local group "Administrators" on a machine (requires admin privileges for non-dc machines)

```powershell
Get-NetLocalGroupMember -ComputerName dcorp-dc -GroupName Administrators
```
{% endtab %}
{% endtabs %}

### Shares

{% tabs %}
{% tab title="PowerView" %}
Find shares on hosts in current domain.

```powershell
Invoke-ShareFinder -Verbose
```

Find sensitive files on computers in the domain

```powershell
Invoke-FileFinder -Verbose
```

Get all file servers of the domain

```powershell
Get-NetFileServer
```
{% endtab %}
{% endtabs %}

Another useful tool may be **PowerHuntShares** - [https://github.com/NetSPI/PowerHuntShares](https://github.com/NetSPI/PowerHuntShares)

<pre class="language-powershell"><code class="lang-powershell">Import-Module PowerHuntShares.psm1
# If it aborts by showing 0 permission. Try again in new powershell session.
<strong>Invoke-HuntSMBShares -NoPing -OutputDirectory &#x3C;path> -HostList &#x3C;list of computer names>
</strong></code></pre>
