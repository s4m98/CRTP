# BloodHound & SOAPHound

### BloodHound

BloodHound Versions:

* [BloodHound Legacy](https://github.com/BloodHoundAD/BloodHound)
* [BloodHound](https://github.com/SpecterOps/BloodHound)

```bash
# start db server
sudo neo4j console

# run bloodhound
bloodhound 
```

SharpHound collector:

* [SharpHound PowerShell](https://github.com/BloodHoundAD/BloodHound/blob/master/Collectors/SharpHound.ps1)
* [SharpHound executable](https://github.com/BloodHoundAD/BloodHound/blob/master/Collectors/SharpHound.exe)

{% code overflow="wrap" %}
```powershell
# PowerShell
. SharpHound.ps1
Invoke-BloodHound -CollectionMethod All 
# Remove noisy collections like RDP,DCOM,PSRemote and Local Admin
Invoke-BloodHound –Steatlh 
Invoke-BloodHound -ExcludeDCs # Avoid MDI

# executable
SharpHound.exe -c All
# Remove noisy collections like RDP,DCOM,PSRemote and Local Admin
SharpHound.exe –-steatlh
# Custom OpSec Friendly with exclude DCs
SharpHound.exe --collectionmethods Group, GPOLocalGroup, Session, Trusts, ACL, Container, ObjectProps, SPNTargets, CertServices --excludedcs
```
{% endcode %}

{% hint style="info" %}
In **legacy** Bloodhound remove **Certservices**. For Community Edition it’s fine.
{% endhint %}

### To enumerate the ACLs for the Applocked and DevOps GPO, let's use the BloodHound CE UI.

### Search for Applocker in the UI -> Click on the node -> Click on Inboud Object Control
<figure><img src="../assets/bloodhound-1.png" alt=""><figcaption></figcaption></figure>

### It turns out that the RDPUsers group has GenericAll over the policy.

### Similary, search for DevOps and look at its 'Inbound Object Control':

<figure><img src="../assets/bloodhound-2.png" alt=""><figcaption></figcaption></figure>

### A user named 'devopsadmin' has 'WriteDACL' on DevOps Policy.


### SoapHound

More stealthy option which uses port 9389

```powershell
soaphound.exe —buildcache -c <path_to_save>
# Output will be BloodHound compatitable
soaphound.exe -c <cache_path> —bhdump -o <output_path> -nolaps
```
