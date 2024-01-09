# AD CS प्रमाणपत्र चोरी

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** 🐦 पर **मुझे फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>

## मैं प्रमाणपत्र के साथ क्या कर सकता हूँ

प्रमाणपत्रों को चुराने का तरीका जांचने से पहले यहाँ कुछ जानकारी है कि प्रमाणपत्र किस काम आ सकता है:
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
## क्रिप्टो APIs का उपयोग करके प्रमाणपत्र निर्यात करना – THEFT1

उपयोगकर्ता या मशीन प्रमाणपत्र और निजी कुंजी निकालने का सबसे आसान तरीका एक **इंटरैक्टिव डेस्कटॉप सत्र** के माध्यम से है। यदि **निजी कुंजी** **निर्यात योग्य** है, तो कोई भी `certmgr.msc` में प्रमाणपत्र पर राइट क्लिक कर सकता है, और `All Tasks → Export`... पर जाकर एक पासवर्ड सुरक्षित .pfx फाइल निर्यात कर सकता है। \
इसे **प्रोग्रामेटिकली** भी पूरा किया जा सकता है। उदाहरणों में PowerShell का `ExportPfxCertificate` cmdlet या [TheWover की CertStealer C# प्रोजेक्ट](https://github.com/TheWover/CertStealer) शामिल हैं।

इन तरीकों के नीचे, ये **Microsoft CryptoAPI** (CAPI) या अधिक आधुनिक Cryptography API: Next Generation (CNG) का उपयोग करके प्रमाणपत्र स्टोर के साथ इंटरैक्ट करते हैं। ये APIs प्रमाणपत्र संग्रहण और प्रमाणीकरण के लिए विभिन्न क्रिप्टोग्राफिक सेवाएं प्रदान करते हैं (अन्य उपयोगों के बीच में)।

यदि निजी कुंजी निर्यात योग्य नहीं है, तो CAPI और CNG निर्यात योग्य नहीं होने वाले प्रमाणपत्रों का निष्कर्षण नहीं करेंगे। **Mimikatz के** `crypto::capi` और `crypto::cng` कमांड्स CAPI और CNG को **निर्यात की अनुमति देने के लिए पैच** कर सकते हैं। `crypto::capi` **पैच** **CAPI** को वर्तमान प्रक्रिया में जबकि `crypto::cng` **lsass.exe की** मेमोरी को **पैच** करने की आवश्यकता होती है।

## DPAPI के माध्यम से उपयोगकर्ता प्रमाणपत्र चोरी – THEFT2

DPAPI के बारे में अधिक जानकारी के लिए:

{% content-ref url="../../windows-local-privilege-escalation/dpapi-extracting-passwords.md" %}
[dpapi-extracting-passwords.md](../../windows-local-privilege-escalation/dpapi-extracting-passwords.md)
{% endcontent-ref %}

Windows **DPAPI का उपयोग करके प्रमाणपत्र निजी कुंजियों को संग्रहीत करता है**। Microsoft उपयोगकर्ता और मशीन निजी कुंजियों के लिए संग्रहण स्थानों को अलग करता है। जब मैन्युअल रूप से एन्क्रिप्टेड DPAPI ब्लॉब्स को डिक्रिप्ट करना होता है, तो डेवलपर को समझना चाहिए कि OS ने कौन सा क्रिप्टोग्राफी API का उपयोग किया है क्योंकि निजी कुंजी फाइल संरचना दो APIs के बीच भिन्न होती है। SharpDPAPI का उपयोग करते समय, यह स्वचालित रूप से इन फाइल प्रारूप भिन्नताओं को समायोजित करता है।

Windows सबसे **आमतौर पर उपयोगकर्ता प्रमाणपत्रों को रजिस्ट्री में `HKEY_CURRENT_USER\SOFTWARE\Microsoft\SystemCertificates` कुंजी में संग्रहीत करता है**, हालांकि कुछ व्यक्तिगत प्रमाणपत्र उपयोगकर्ताओं के लिए **भी** `%APPDATA%\Microsoft\SystemCertificates\My\Certificates` में संग्रहीत होते हैं। संबंधित उपयोगकर्ता **निजी कुंजी स्थान** मुख्य रूप से `%APPDATA%\Microsoft\Crypto\RSA\User SID\` पर **CAPI** कुंजियों के लिए और `%APPDATA%\Microsoft\Crypto\Keys\` पर **CNG** कुंजियों के लिए होते हैं।

एक प्रमाणपत्र और उसकी संबंधित निजी कुंजी प्राप्त करने के लिए, आपको:

1. पहचानें कि **कौन सा प्रमाणपत्र आप उपयोगकर्ता के प्रमाणपत्र स्टोर से चुराना चाहते हैं** और कुंजी स्टोर का नाम निकालें।
2. **DPAPI मास्टरकी** खोजें जो संबंधित निजी कुंजी को डिक्रिप्ट करने के लिए आवश्यक है।
3. प्लेनटेक्स्ट DPAPI मास्टरकी प्राप्त करें और इसका उपयोग करके **निजी कुंजी को डिक्रिप्ट करें**।

**प्लेनटेक्स्ट DPAPI मास्टरकी प्राप्त करने के लिए**:
```bash
# With mimikatz
## Running in a process in the users context
dpapi::masterkey /in:"C:\PATH\TO\KEY" /rpc

# with mimikatz
## knowing the users password
dpapi::masterkey /in:"C:\PATH\TO\KEY" /sid:accountSid /password:PASS
```
मास्टरकी फाइल और प्राइवेट की फाइल के डिक्रिप्शन को सरल बनाने के लिए, [**SharpDPAPI’s**](https://github.com/GhostPack/SharpDPAPI) `certificates` कमांड का उपयोग `/pvk`, `/mkfile`, `/password`, या `{GUID}:KEY` आर्ग्यूमेंट्स के साथ किया जा सकता है ताकि प्राइवेट कीज़ और संबंधित सर्टिफिकेट्स को डिक्रिप्ट किया जा सके, जिससे एक `.pem` टेक्स्ट फाइल का आउटपुट मिलता है।
```bash
SharpDPAPI.exe certificates /mkfile:C:\temp\mkeys.txt

# Transfor .pem to .pfx
openssl pkcs12 -in cert.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out cert.pfx
```
## मशीन सर्टिफिकेट चोरी DPAPI के माध्यम से – THEFT3

Windows मशीन सर्टिफिकेट्स को रजिस्ट्री कुंजी `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\SystemCertificates` में संग्रहीत करता है और निजी कुंजियों को खाते के आधार पर कई अलग-अलग स्थानों पर संग्रहीत करता है।\
हालांकि SharpDPAPI इन सभी स्थानों की खोज करेगा, सबसे दिलचस्प परिणाम अक्सर `%ALLUSERSPROFILE%\Application Data\Microsoft\Crypto\RSA\MachineKeys` (CAPI) और `%ALLUSERSPROFILE%\Application Data\Microsoft\Crypto\Keys` (CNG) से आते हैं। ये **निजी कुंजियाँ** **मशीन सर्टिफिकेट** स्टोर से जुड़ी होती हैं और Windows उन्हें **मशीन के DPAPI मास्टर कुंजियों** से एन्क्रिप्ट करता है।\
इन कुंजियों को डोमेन की DPAPI बैकअप कुंजी का उपयोग करके डिक्रिप्ट नहीं किया जा सकता है, बल्कि **अवश्य** **DPAPI\_SYSTEM LSA सीक्रेट** का उपयोग करना चाहिए जो सिस्टम पर **केवल SYSTEM उपयोगकर्ता द्वारा ही सुलभ है**।&#x20;

आप इसे मैन्युअली **Mimikatz’** के **`lsadump::secrets`** कमांड के साथ कर सकते हैं और फिर निकाली गई कुंजी का उपयोग करके **मशीन मास्टरकीज़ को डिक्रिप्ट** कर सकते हैं। \
आप CAPI/CNG को पहले की तरह पैच भी कर सकते हैं और **Mimikatz’** के `crypto::certificates /export /systemstore:LOCAL_MACHINE` कमांड का उपयोग कर सकते हैं। \
**SharpDPAPI’s** सर्टिफिकेट्स कमांड **`/machine`** फ्लैग के साथ (जब एलिवेटेड हो) स्वचालित रूप से **SYSTEM** को **एलिवेट** करेगा, **DPAPI\_SYSTEM** LSA सीक्रेट को **डंप** करेगा, इसका उपयोग करके मशीन DPAPI मास्टरकीज़ को **डिक्रिप्ट** करेगा, और कुंजी प्लेनटेक्स्ट्स का उपयोग किसी भी मशीन सर्टिफिकेट निजी कुंजियों को डिक्रिप्ट करने के लिए लुकअप टेबल के रूप में करेगा।

## सर्टिफिकेट फाइलें खोजना – THEFT4

कभी-कभी **सर्टिफिकेट्स फाइल सिस्टम में ही होते हैं**, जैसे कि फाइल शेयर्स या डाउनलोड्स फोल्डर में।\
हमने जो सबसे आम प्रकार के Windows-केंद्रित सर्टिफिकेट फाइलें देखी हैं, वे हैं **`.pfx`** और **`.p12`** फाइलें, जिनमें **`.pkcs12`** और ** `.pem` ** कभी-कभी दिखाई देते हैं लेकिन कम बार।\
अन्य दिलचस्प सर्टिफिकेट-संबंधित फाइल एक्सटेंशन हैं: **`.key`** (_निजी कुंजी_), **`.crt/.cer`** (_केवल सर्टिफिकेट_), **`.csr`** (_सर्टिफिकेट साइनिंग रिक्वेस्ट, इसमें सर्ट्स या निजी कुंजियाँ नहीं होती_), **`.jks/.keystore/.keys`** (_जावा कीस्टोर. जावा एप्लिकेशन्स द्वारा उपयोग किए जाने वाले सर्ट्स + निजी कुंजियाँ हो सकती हैं_)।

इन फाइलों को खोजने के लिए, बस उन एक्सटेंशन्स के लिए powershell या cmd का उपयोग करके खोजें।

यदि आप एक **PKCS#12** सर्टिफिकेट फाइल पाते हैं और यह **पासवर्ड से सुरक्षित है**, तो आप [pfx2john.py](https://fossies.org/dox/john-1.9.0-jumbo-1/pfx2john\_8py\_source.html) का उपयोग करके एक हैश निकाल सकते हैं और इसे JohnTheRipper का उपयोग करके **क्रैक** कर सकते हैं।

## NTLM क्रेडेंशियल चोरी PKINIT के माध्यम से – THEFT5

> नेटवर्क सेवाओं से जुड़ने वाले एप्लिकेशन्स के लिए **NTLM प्रमाणीकरण का समर्थन** करने के लिए, जो **Kerberos प्रमाणीकरण का समर्थन नहीं करते**, जब PKCA का उपयोग किया जाता है, KDC **उपयोगकर्ता के NTLM** वन-वे फंक्शन (OWF) को प्रिविलेज एट्रिब्यूट सर्टिफिकेट (PAC) **`PAC_CREDENTIAL_INFO`** बफर में लौटाता है

इसलिए, यदि खाता प्रमाणीकरण करता है और **PKINIT के माध्यम से TGT प्राप्त करता है**, तो एक बिल्ट-इन "फेलसेफ" होता है जो वर्तमान होस्ट को **TGT से हमारे NTLM हैश प्राप्त करने की अनुमति देता है** ताकि पुराने प्रमाणीकरण का समर्थन किया जा सके। इसमें **`PAC_CREDENTIAL_DATA`** **संरचना को डिक्रिप्ट करना** शामिल है जो NTLM प्लेनटेक्स्ट का एक Network Data Representation (NDR) सीरियलाइज्ड प्रतिनिधित्व है।

[**Kekeo**](https://github.com/gentilkiwi/kekeo) का उपयोग करके इस जानकारी के साथ TGT के लिए पूछा जा सकता है और उपयोगकर्ताओं के NTML प्राप्त किए जा सकते हैं।
```bash
tgt::pac /caname:thename-DC-CA /subject:harmj0y /castore:current_user /domain:domain.local
```
Kekeo का कार्यान्वयन स्मार्टकार्ड-संरक्षित प्रमाणपत्रों के साथ भी काम करेगा जो वर्तमान में प्लग इन हैं यदि आप [**पिन पुनः प्राप्त**](https://github.com/CCob/PinSwipe) कर सकते हैं। यह [**Rubeus**](https://github.com/GhostPack/Rubeus) में भी समर्थित होगा।

## संदर्भ

* सभी जानकारी [https://www.specterops.io/assets/resources/Certified\_Pre-Owned.pdf](https://www.specterops.io/assets/resources/Certified\_Pre-Owned.pdf) से ली गई थी।

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सदस्यता योजनाओं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **HackTricks** के [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>
