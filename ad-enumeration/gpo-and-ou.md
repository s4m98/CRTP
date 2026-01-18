# GPO & OU

### Group Policy

{% tabs %}
{% tab title="PowerView" %}
Get list of GPOs in current domain

```powershell
Get-DomainGPO
```

Get OUs on which the GPO is applied. From OUs get the objects.

```powershell
# Get OUs on which the GPO is linked
Get-DomainOU| Where-Object {
    $_.gplink -like "*{0BF8D01C-1F62-4BDC-958C-57140B67D147}*"
}

# Get the objects inside a OU
Get-DomainComputer -SearchBase "OU=DevOps,DC=dollarcorp,DC=moneycorp,DC=local"
# Change the Get-DomainComputer with Get-DomainUser and Get-DomainObject
```

Get GPO(s) which use Restricted Groups

```powershell
Get-DomainGPOLocalGroup
```

Get users which are in a local group of a machine using GPO

```powershell
Get-DomainGPOComputerLocalGroupMapping -ComputerIdentity dcorp-student1
```

Get machines where the given user is member of a specific group

```powershell
Get-DomainGPOUserLocalGroupMapping -Identity student1 -Verbose 
```
{% endtab %}
{% endtabs %}

### Organization Units

{% tabs %}
{% tab title="PowerView" %}
Get OUs in a domain

```powershell
# Get all domain OUs
Get-DomainOU
# Get Only OUs Name:
Get-DomainOU | select -ExpandProperty name
# Get all computers inside an OU
(Get-DomainOU -Identity StudentMachines).distinguishedname | %{Get-DomainComputer -SearchBase $_} | select name
```

Using `Get-NetOU`

```powershell
# Get all computers inside an OU
(Get-NetOU -Identity StudentMachines).distinguishedname | %{Get-DomainComputer -SearchBase $_} | select name

# Get GPO applied on an OU 
Get-NetOU -Identity "StudentMachines" | select gplink # Get GPO ID
Get-DomainGPO -Identity "{0D1CC23D-1F20-4EEE-AF64-D99597AE2A6E}" # Get GPO Info
```
{% endtab %}

{% tab title="AD Module" %}
Get OUs in a domain

```powershell
Get-ADOrganizationalUnit -Filter * -Properties *
```
{% endtab %}
{% endtabs %}
