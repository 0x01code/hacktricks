# एडी सीएस सर्टिफिकेट चोरी

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) और **मुझे** **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos पर PRs सबमिट करके।

</details>

**यह एक छोटा सारांश है चोरी अध्यायों का जो शानदार रिसर्च से है [https://www.specterops.io/assets/resources/Certified\_Pre-Owned.pdf](https://www.specterops.io/assets/resources/Certified\_Pre-Owned.pdf)**

## मैं सर्टिफिकेट के साथ क्या कर सकता हूँ

सर्टिफिकेट चोरी कैसे करें इसे चेक करने से पहले आपको यहाँ कुछ जानकारी है कि सर्टिफिकेट का उपयोग क्या है:
```powershell
# Powershell
$CertPath = "C:\path\to\cert.pfx"
$CertPass = "P@ssw0rd"
$Cert = New-Object
System.Security.Cryptography.X509Certificates.X509Certificate2 @($CertPath, $CertPass)
$Cert.EnhancedKeyUsageList

# cmd
certutil.exe -dump -v cert.pfx
```
## प्रमाणपत्रों को क्रिप्टो एपीआई का उपयोग करके निकालना - चोरी1

**एक इंटरैक्टिव डेस्कटॉप सत्र** में, उपयोगकर्ता या मशीन प्रमाणपत्र को प्राप्त करना, साथ ही निजी कुंजी के साथ, यदि **निजी कुंजी निर्यातीय है**, आसानी से किया जा सकता है। इसे `certmgr.msc` में प्रमाणपत्र पर जाकर, उस पर राइट-क्लिक करके, और `All Tasks → Export` का चयन करके किया जा सकता है ताकि एक पासवर्ड से सुरक्षित .pfx फ़ाइल उत्पन्न हो।

**कार्यक्रमात्मक दृष्टिकोण** के लिए, PowerShell `ExportPfxCertificate` cmdlet जैसे उपकरण या [TheWover’s CertStealer C# project](https://github.com/TheWover/CertStealer) जैसे परियोजनाएँ उपलब्ध हैं। ये **Microsoft CryptoAPI** (CAPI) या Cryptography API: Next Generation (CNG) का उपयोग करते हैं प्रमाणपत्र स्टोर के साथ संवाद स्थापित करने के लिए। ये एपीआई क्रिप्टोग्राफिक सेवाएँ प्रदान करते हैं, जिनमें प्रमाणपत्र संग्रहण और प्रमाणीकरण के लिए आवश्यक सेवाएँ शामिल हैं।

हालांकि, यदि निजी कुंजी को निर्यातीय निर्धारित किया गया है, तो दोनों CAPI और CNG सामान्यत: ऐसे प्रमाणपत्रों की निकासी को ब्लॉक करेंगे। इस प्रतिबंध को अनदेखा करने के लिए, Mimikatz जैसे उपकरण का उपयोग किया जा सकता है। Mimikatz `crypto::capi` और `crypto::cng` कमांड प्रदान करता है जिनका उपयोग करके उपयोग कुंजी को निर्यात करने की अनुमति देता है। विशेष रूप से, `crypto::capi` वर्तमान प्रक्रिया के भीतर CAPI को पैच करता है, जबकि `crypto::cng` **lsass.exe** की स्मृति को पैच करने के लिए लक्ष्य रखता है।

## उपयोगकर्ता प्रमाणपत्र चोरी DPAPI के माध्यम से - चोरी2

DPAPI के बारे में अधिक जानकारी:

{% content-ref url="../../windows-local-privilege-escalation/dpapi-extracting-passwords.md" %}
[dpapi-extracting-passwords.md](../../windows-local-privilege-escalation/dpapi-extracting-passwords.md)
{% endcontent-ref %}

Windows में, **प्रमाणपत्र निजी कुंजी DPAPI द्वारा सुरक्षित हैं**। यह महत्वपूर्ण है कि **उपयोगकर्ता और मशीन निजी कुंजी के भंडारण स्थान** अलग होते हैं, और फ़ाइल संरचनाएँ ऑपरेटिंग सिस्टम द्वारा उपयोग किए गए विज्ञान की एपीआई पर निर्भर करती हैं। **SharpDPAPI** एक उपकरण है जो DPAPI ब्लॉब को डिक्रिप्ट करते समय इन अंतर को स्वचालित रूप से नेविगेट कर सकता है।

**उपयोगकर्ता प्रमाणपत्र** मुख्य रूप से रजिस्ट्री में `HKEY_CURRENT_USER\SOFTWARE\Microsoft\SystemCertificates` के तहत स्थित होते हैं, लेकिन कुछ प्रमाणपत्र `%APPDATA%\Microsoft\SystemCertificates\My\Certificates` निर्देशिका में भी पाई जा सकती हैं। इन प्रमाणपत्रों के संबंधित **निजी कुंजी** सामान्यत: `%APPDATA%\Microsoft\Crypto\RSA\User SID\` में संग्रहीत होती हैं **CAPI** कुंजियों के लिए और `%APPDATA%\Microsoft\Crypto\Keys\` **CNG** कुंजियों के लिए।

**प्रमाणपत्र और इसकी संबंधित निजी कुंजी को निकालने** की प्रक्रिया शामिल है:

1. **उपयोगकर्ता के स्टोर से लक्ष्य प्रमाणपत्र का चयन** करना और उसकी कुंजी स्टोर नाम प्राप्त करना।
2. **उस संबंधित DPAPI मास्टरकी को ढूंढना** जिसका उपयोग करके संबंधित निजी कुंजी को डिक्रिप्ट किया जा सकता है।
3. **प्लेनटेक्स्ट DPAPI मास्टरकी का उपयोग करके निजी कुंजी को डिक्रिप्ट** करना।

**प्लेनटेक्स्ट DPAPI मास्टरकी** प्राप्त करने के लिए, निम्नलिखित दृष्टिकोण का उपयोग किया जा सकता हैं:
```bash
# With mimikatz, when running in the user's context
dpapi::masterkey /in:"C:\PATH\TO\KEY" /rpc

# With mimikatz, if the user's password is known
dpapi::masterkey /in:"C:\PATH\TO\KEY" /sid:accountSid /password:PASS
```
अध्यायनकर्ता फ़ाइलों के मास्टरकी और निजी कुंजी फ़ाइलों की डिक्रिप्शन को सुगम बनाने के लिए, [**SharpDPAPI**](https://github.com/GhostPack/SharpDPAPI) से `certificates` कमांड फायदेमंद साबित होता है। यह निजी कुंजी और जुड़े हुए प्रमाणपत्रों को डिक्रिप्ट करने के लिए `/pvk`, `/mkfile`, `/password`, या `{GUID}:KEY` को तर्क के रूप में स्वीकार करता है, जिससे एक `.pem` फ़ाइल उत्पन्न होती है।
```bash
# Decrypting using SharpDPAPI
SharpDPAPI.exe certificates /mkfile:C:\temp\mkeys.txt

# Converting .pem to .pfx
openssl pkcs12 -in cert.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out cert.pfx
```
## मशीन प्रमाणपत्र चोरी DPAPI के माध्यम से - THEFT3

Windows द्वारा रजिस्ट्री में संग्रहीत मशीन प्रमाणपत्र `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\SystemCertificates` और संबंधित निजी कुंजीयाँ `%ALLUSERSPROFILE%\Application Data\Microsoft\Crypto\RSA\MachineKeys` (CAPI के लिए) और `%ALLUSERSPROFILE%\Application Data\Microsoft\Crypto\Keys` (CNG के लिए) मशीन के DPAPI मास्टर कुंजीयों का उपयोग करके एन्क्रिप्ट किए जाते हैं। ये कुंजीयाँ डोमेन की DPAPI बैकअप कुंजी के साथ डिक्रिप्ट नहीं की जा सकती हैं; इसके बजाय, **DPAPI_SYSTEM LSA गुप्त** कुंजी की आवश्यकता होती है, जिसको केवल SYSTEM उपयोगकर्ता तक पहुंच सकता है।

मैनुअल डिक्रिप्शन को **Mimikatz** में `lsadump::secrets` कमांड को निष्पादित करके प्राप्त किया जा सकता है ताकि DPAPI_SYSTEM LSA गुप्त को निकाला जा सके, और इसके बाद इस कुंजी का उपयोग करके मशीन मास्टरकुंजीयों को डिक्रिप्ट किया जा सकता है। वैकल्पिक रूप से, Mimikatz के `crypto::certificates /export /systemstore:LOCAL_MACHINE` कमांड का उपयोग करके CAPI/CNG को पैच करने के बाद किया जा सकता है जैसा पहले से वर्णित किया गया है।

**SharpDPAPI** अपने प्रमाणपत्र कमांड के साथ एक और स्वचालित दृष्टिकोण प्रदान करता है। जब `/machine` फ्लैग को उच्च अनुमतियों के साथ उपयोग किया जाता है, तो यह SYSTEM में उन्नति करता है, DPAPI_SYSTEM LSA गुप्त को डंप करता है, इसका उपयोग करके मशीन DPAPI मास्टर कुंजीयों को डिक्रिप्ट करता है, और फिर इन सादा कुंजीयों का उपयोग करके किसी भी मशीन प्रमाणपत्र निजी कुंजीयों को डिक्रिप्ट करने के लिए एक खोज सारणी के रूप में काम करता है।


## प्रमाणपत्र फ़ाइलें खोजना - THEFT4

प्रमाणपत्र कभी-कभी फ़ाइल सिस्टम में सीधे पाई जाती हैं, जैसे फ़ाइल शेयर या डाउनलोड्स फ़ोल्डर में। Windows परिवेशों के लिए लक्षित सबसे आम प्रकार की प्रमाणपत्र फ़ाइलें `.pfx` और `.p12` फ़ाइलें हैं। हालांकि कम आमतौर पर, `.pkcs12` और `.pem` जैसे एक्सटेंशन वाली फ़ाइलें भी दिखाई देती हैं। अतिरिक्त महत्वपूर्ण प्रमाणपत्र संबंधित फ़ाइल एक्सटेंशन शामिल हैं:
- निजी कुंजीयों के लिए `.key`,
- केवल प्रमाणपत्र के लिए `.crt`/`.cer`,
- प्रमाणपत्र साइनिंग अनुरोधों के लिए `.csr`, जिसमें प्रमाणपत्र या निजी कुंजीयाँ नहीं होती हैं,
- जावा केस्टोर्स के लिए `.jks`/`.keystore`/`.keys`, जिनमें जावा एप्लिकेशन्स द्वारा उपयोग की जाने वाली प्रमाणपत्रों के साथ निजी कुंजीयाँ हो सकती हैं।

इन फ़ाइलों को PowerShell या कमांड प्रॉम्प्ट का उपयोग करके उल्लेखित एक्सटेंशन की खोज करके खोजा जा सकता है।

जब एक PKCS#12 प्रमाणपत्र फ़ाइल मिलती है और इसे पासवर्ड से सुरक्षित किया गया है, तो `pfx2john.py` का उपयोग करके हैश का निकालना संभव है, जो [fossies.org](https://fossies.org/dox/john-1.9.0-jumbo-1/pfx2john_8py_source.html) पर उपलब्ध है। इसके बाद, JohnTheRipper का उपयोग करके पासवर्ड को क्रैक करने का प्रयास किया जा सकता है।
```powershell
# Example command to search for certificate files in PowerShell
Get-ChildItem -Recurse -Path C:\Users\ -Include *.pfx, *.p12, *.pkcs12, *.pem, *.key, *.crt, *.cer, *.csr, *.jks, *.keystore, *.keys

# Example command to use pfx2john.py for extracting a hash from a PKCS#12 file
pfx2john.py certificate.pfx > hash.txt

# Command to crack the hash with JohnTheRipper
john --wordlist=passwords.txt hash.txt
```
## NTLM प्रमाणीकरण चोरी PKINIT के माध्यम से - THEFT5

दिया गया सामग्री NTLM प्रमाणीकरण चोरी के लिए एक विधि को समझाती है PKINIT के माध्यम से, विशेष रूप से चोरी विधि के रूप में चिह्नित THEFT5. यहाँ एक पासिव वॉयस में पुनर्व्याख्यान है, सामग्री को अनामित किया गया है और जहाँ योग्य हो, संक्षेपित किया गया है:

एनटीएलएम प्रमाणीकरण [MS-NLMP] का समर्थन करने के लिए ऐप्लिकेशनों के लिए जो केरबेरोस प्रमाणीकरण को सुविधा नहीं प्रदान करते, KDC को उपयोग करने पर उपयोगकर्ता का एनटीएलएम वन-वे फ़ंक्शन (OWF) प्रिविलेज एट्रिब्यूट प्रमाणपत्र (PAC) में वापस देने के लिए डिज़ाइन किया गया है, विशेष रूप से `PAC_CREDENTIAL_INFO` बफ़र में, जब PKCA का उपयोग किया जाता है। इस परिणामस्वरूप, यदि एक खाता प्रमाणीकृत होता है और PKINIT के माध्यम से एक टिकट-ग्रांटिंग टिकट (TGT) सुरक्षित करता है, तो वर्तमान होस्ट को TGT से NTLM हैश निकालने की सुविधा प्रदान की जाती है जो पुराने प्रमाणीकरण प्रोटोकॉल को समर्थन करने के लिए। यह प्रक्रिया `PAC_CREDENTIAL_DATA` संरचना का डिक्रिप्शन है, जो मौलिक रूप से NTLM प्लेनटेक्स्ट का एक NDR सीरियलाइज्ड चित्रण है।

**Kekeo** उपयोगकर्तव्य, [https://github.com/gentilkiwi/kekeo](https://github.com/gentilkiwi/kekeo) पर उपलब्ध है, जिसे इस विशेष डेटा को समाहित करने वाले एक TGT का अनुरोध करने की क्षमता के रूप में उल्लेख किया गया है। इस उद्देश्य के लिए उपयोग किया गया आदेश निम्नलिखित है:
```bash
tgt::pac /caname:generic-DC-CA /subject:genericUser /castore:current_user /domain:domain.local
```
Additionally, it is noted that Kekeo can process smartcard-protected certificates, given the pin can be retrieved, with reference made to [https://github.com/CCob/PinSwipe](https://github.com/CCob/PinSwipe). The same capability is indicated to be supported by **Rubeus**, available at [https://github.com/GhostPack/Rubeus](https://github.com/GhostPack/Rubeus).

This explanation encapsulates the process and tools involved in NTLM credential theft via PKINIT, focusing on the retrieval of NTLM hashes through TGT obtained using PKINIT, and the utilities that facilitate this process.

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Other ways to support HackTricks:

* If you want to see your **company advertised in HackTricks** or **download HackTricks in PDF** Check the [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Get the [**official PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Discover [**The PEASS Family**](https://opensea.io/collection/the-peass-family), our collection of exclusive [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Join the** 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) or the [**telegram group**](https://t.me/peass) or **follow** me on **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **Share your hacking tricks by submitting PRs to the** [**HackTricks**](https://github.com/carlospolop/hacktricks) and [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
