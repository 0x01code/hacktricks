# AD CS प्रमाणपत्र चोरी

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)

- प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **फॉलो** करें मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स साझा करें, [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud)** को PR जमा करके।

</details>

## मैं प्रमाणपत्र के साथ क्या कर सकता हूँ

प्रमाणपत्रों को चुराने के तरीकों की जांच करने से पहले, यहां आपको प्रमाणपत्र का उपयोग करने के लिए कुछ जानकारी है:
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
## क्रिप्टो API का उपयोग करके प्रमाणपत्रों को निकालना - चोरी1

एक उपयोगकर्ता या मशीन प्रमाणपत्र और निजी कुंजी को निकालने का सबसे आसान तरीका **इंटरैक्टिव डेस्कटॉप सत्र** के माध्यम से है। यदि **निजी कुंजी** **निर्यात योग्य** है, तो कोई भी सीधे प्रमाणपत्र पर दायां क्लिक करके `certmgr.msc` में जा सकता है, और `All Tasks → Export` ... पर जाकर पासवर्ड संरक्षित .pfx फ़ाइल को निर्यात कर सकता है। \
इसे **कार्यक्रमात्मक रूप से** भी प्राप्त किया जा सकता है। उदाहरणों में PowerShell का `ExportPfxCertificate` cmdlet या [TheWover का CertStealer C# प्रोजेक्ट](https://github.com/TheWover/CertStealer) शामिल है।

इन तरीकों के नीचे, ये विधियाँ प्रमाणपत्र संग्रह के साथ बातचीत करने के लिए **माइक्रोसॉफ्ट क्रिप्टोएपीआई** (CAPI) या अधिक आधुनिक क्रिप्टोग्राफी एपीआई: अगली पीढ़ी (CNG) का उपयोग करते हैं। ये एपीआई प्रमाणपत्र संग्रह और प्रमाणीकरण के लिए आवश्यक विभिन्न गणितीय सेवाएं करते हैं (अन्य उपयोगों के बीच)।

यदि निजी कुंजी निर्यात योग्य नहीं है, तो CAPI और CNG निर्यात योग्य प्रमाणपत्रों को निकालने की अनुमति नहीं देंगे। **Mimikatz के** `crypto::capi` और `crypto::cng` कमांड CAPI और CNG को **निजी कुंजी** की निर्यात की अनुमति देने के लिए पैच कर सकते हैं। `crypto::capi` **वर्तमान प्रक्रिया** में CAPI को **पैच करता है** जबकि `crypto::cng` **lsass.exe** की मेमोरी को **पैच करने** की आवश्यकता होती है।

## उपयोगकर्ता प्रमाणपत्र चोरी DPAPI के माध्यम से - चोरी2

DPAPI के बारे में अधिक जानकारी:

{% content-ref url="../../windows-local-privilege-escalation/dpapi-extracting-passwords.md" %}
[dpapi-extracting-passwords.md](../../windows-local-privilege-escalation/dpapi-extracting-passwords.md)
{% endcontent-ref %}

Windows **DPAPI का उपयोग करके प्रमाणपत्र निजी कुंजी संग्रहित करता है**। माइक्रोसॉफ्ट उपयोगकर्ता और मशीन निजी कुंजी के लिए संग्रह स्थानों को अलग करता है। DPAPI ब्लॉब्स को मैन्युअल रूप से डिक्रिप्ट करते समय, एक डेवलपर को समझना चाहिए कि ऑपरेटिंग सिस्टम ने कौन सी क्रिप्टोग्राफी एपीआई का उपयोग किया है क्योंकि दो एपीआई के बीच निजी कुंजी फ़ाइल संरचना में अंतर होता है। SharpDPAPI का उपयोग करते समय, यह स्वचालित रूप से इन फ़ाइल प्रारूप अंतरों का ध्यान रखता है।&#x20;

Windows सबसे **आमतौर पर उपयोगकर्ता प्रमाणपत्रों को** रजिस्ट्री में `HKEY_CURRENT_USER\SOFTWARE\Microsoft\SystemCertificates` कुंजी में संग्रहित करता है, हालांकि कुछ उपयोगकर्ताओं के व्यक्तिगत प्रमाणपत्र भी `%APPDATA%\Microsoft\SystemCertificates\My\Certificates` में संग्रहित होते हैं। संबंधित उपयोगकर्ता **निजी कुंजी स्थान** मुख्य रूप से `%APPDATA%\Microsoft\Crypto\RSA\User SID\` के लिए **CAPI** कुंजी और `%APPDATA%\Microsoft\Crypto\Keys\` के लिए **CNG** कुंजी में होते हैं।

प्रमाणपत्र और इसके संबंधित निजी कुंजी को प्राप्त करने के लिए, निम्नलिखित कार्रवाई की आवश्यकता होती है:

1. उपयोगकर्ता के प्रमाणपत्र संग्रह से **किस प्रमाणपत्र को चुराना चाहते हैं** उसे पहचानें और कुंजी संग्रह का नाम निकालें।
2. संबंधित निजी कुंजी को डिक्रिप्ट करने के लिए आवश्यक **DPAPI मास्टरकी** खोजें।
3. प्लेनटेक्स्ट DPAPI मास्टरकी प्राप्त करें और इसका उपयोग करके निजी कुंजी को **डिक्रिप्ट करें**।

**प्लेनटेक्स्ट DPAPI मास्टरकी प्राप्त करने के लिए**:
```bash
# With mimikatz
## Running in a process in the users context
dpapi::masterkey /in:"C:\PATH\TO\KEY" /rpc

# with mimikatz
## knowing the users password
dpapi::masterkey /in:"C:\PATH\TO\KEY" /sid:accountSid /password:PASS
```
गोपनीयता कुंजी फ़ाइल और निजी कुंजी फ़ाइल के डिक्रिप्शन को सरल बनाने के लिए, [**SharpDPAPI**](https://github.com/GhostPack/SharpDPAPI) के `certificates` कमांड का उपयोग `/pvk`, `/mkfile`, `/password`, या `{GUID}:KEY` तर्कों के साथ किया जा सकता है ताकि निजी कुंजी और संबंधित प्रमाणपत्रों को डिक्रिप्ट किया जा सके और एक `.pem` पाठ फ़ाइल को आउटपुट किया जा सके।
```bash
SharpDPAPI.exe certificates /mkfile:C:\temp\mkeys.txt

# Transfor .pem to .pfx
openssl pkcs12 -in cert.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out cert.pfx
```
## मशीन प्रमाणपत्र चोरी DPAPI के माध्यम से - THEFT3

Windows मशीन प्रमाणपत्रों को रजिस्ट्री की कुंजी `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\SystemCertificates` में संग्रहीत करता है और खाते के आधार पर निजी कुंजी कई अलग-अलग स्थानों में संग्रहीत करता है।\
हालांकि, SharpDPAPI इन सभी स्थानों की खोज करेगा, लेकिन सबसे दिलचस्प परिणाम `%ALLUSERSPROFILE%\Application Data\Microsoft\Crypto\RSA\MachineKeys` (CAPI) और `%ALLUSERSPROFILE%\Application Data\Microsoft\Crypto\Keys` (CNG) से आते हैं। ये **निजी कुंजी** मशीन प्रमाणपत्र संग्रह के साथ जुड़ी होती हैं और Windows इन्हें **मशीन के DPAPI मास्टर कुंजी** से एन्क्रिप्ट करता है।\
इन कुंजीयों को डोमेन की DPAPI बैकअप कुंजी का उपयोग करके नहीं खोला जा सकता है, बल्कि इसे सिस्टम पर **केवल SYSTEM उपयोगकर्ता** द्वारा पहुंचने योग्य **DPAPI\_SYSTEM LSA सीक्रेट** का उपयोग करना होगा।

आप इसे मैन्युअल रूप से **Mimikatz** के **`lsadump::secrets`** कमांड के साथ कर सकते हैं और फिर निकाली गई कुंजी का उपयोग करके **मशीन मास्टरकुंजी** को **डिक्रिप्ट** कर सकते हैं।\
आप CAPI/CNG को पैच कर सकते हैं और फिर **Mimikatz** के `crypto::certificates /export /systemstore:LOCAL_MACHINE` कमांड का उपयोग कर सकते हैं।\
**SharpDPAPI** के प्रमाणपत्र कमांड के साथ **`/machine`** फ्लैग (उच्चस्तर पर) स्वचालित रूप से **SYSTEM** को **उच्चस्तर** करेगा, **DPAPI\_SYSTEM** LSA सीक्रेट को डंप करेगा, इसका उपयोग करके मशीन DPAPI मास्टर कुंजी को डिक्रिप्ट करेगा और कुंजी के प्लेनटेक्स्ट को एक खोज सारणी के रूप में उपयोग करके किसी भी मशीन प्रमाणपत्र निजी कुंजी को डिक्रिप्ट करेगा।

## प्रमाणपत्र फ़ाइलें खोजना - THEFT4

कभी-कभी प्रमाणपत्र सिर्फ़ फ़ाइल सिस्टम में होते हैं, जैसे फ़ाइल शेयर्स या डाउनलोड्स फ़ोल्डर में।\
हमने देखा है कि Windows के ध्यान केंद्रित प्रमाणपत्र फ़ाइलों के सबसे आम प्रकार **`.pfx`** और **`.p12`** फ़ाइलें होती हैं, जबकि कभी-कभी **`.pkcs12`** और **`.pem`** भी दिखाई देती हैं, लेकिन कम बार।\
अन्य दिलचस्प प्रमाणपत्र संबंधित फ़ाइल एक्सटेंशन हैं: **`.key`** (_निजी कुंजी_), **`.crt/.cer`** (_केवल प्रमाणपत्र_), **`.csr`** (_प्रमाणपत्र साइन करने का अनुरोध, इसमें प्रमाणपत्र या निजी कुंजी शामिल नहीं होती_), **`.jks/.keystore/.keys`** (_जावा कीस्टोर। जावा एप्लिकेशन द्वारा उपयोग की जाने वाली प्रमाणपत्र + निजी कुंजी शामिल हो सकती हैं_)

इन फ़ाइलों को खोजने के लिए, पावरशेल या cmd का उपयोग करके उन एक्सटेंशन की खोज करें।

यदि आपको एक **PKCS#12** प्रमाणपत्र फ़ाइल मिलती है और वह **पासवर्ड से सुरक्षित** है, तो आप [pfx2john.py](https://fossies.org/dox/john-1.9.0-jumbo-1/pfx2john\_8py\_source.html) का उपयोग करके एक हैश निकाल सकते हैं और JohnTheRipper का उपयोग करके इसे **क्रैक** कर सकते हैं।

## PKINIT के माध्यम से NTLM क्रेडेंशियल चोरी - THEFT5

> NTLM प्रमाणीकरण का समर्थन करने के लिए \[MS-NLMP] नेटवर्क सेवाओं से कनेक्ट होने वाले अनुप्रयोगों के लिए, PKCA का उपयोग करते समय, KDC प्रिविलेज विशेषता प्रमाणपत्र (PAC) **`PAC_CREDENTIAL_INFO`** बफर में उपयोगकर्ता का NTLM वन-वेतानिक फ़ंक्शन (OWF) लौटाता है

तो, यदि खाता प्रमाणीकृत होता है और PKINIT के माध्यम से एक **TGT प्राप्त** करता है, तो वर्तमान होस्ट को **हमारे NTLM हैश को TGT से प्राप्त** करने के लिए एक "फेलसेफ" होता है जो पुराने प्रमाणीकरण का समर्थन करने के लिए NTLM सादा-पाठ की एक **`PAC_CREDENTIAL_DATA`** **संरचना** को **डिक्रिप्ट** करने का समर्थन करता है जो एक नेटवर्क डेटा प्रतिनिधित्व (NDR) सीरीयलाइज़ की गई प्रतिनिधित्व है।

[**Kekeo**](https://github.com/gentilkiwi/kekeo) का उपयोग करके इस जानकारी के साथ एक TGT के लिए पूछ सकते हैं और उपयोगकर्ता के NTML हैश को प्राप्त कर सकते हैं
```bash
tgt::pac /caname:thename-DC-CA /subject:harmj0y /castore:current_user /domain:domain.local
```
Kekeo का अनुपालन करने वाला अंतर्गत यह भी काम करेगा जो स्मार्टकार्ड से सुरक्षित प्रमाणित प्रमाणपत्र हैं जो वर्तमान में प्लग इन हैं, यदि आप [**पिन को पुनर्प्राप्त कर सकते हैं**](https://github.com/CCob/PinSwipe)**.** यह [**Rubeus**](https://github.com/GhostPack/Rubeus) में भी समर्थित होगा।

## संदर्भ

* सभी जानकारी [https://www.specterops.io/assets/resources/Certified\_Pre-Owned.pdf](https://www.specterops.io/assets/resources/Certified\_Pre-Owned.pdf) से ली गई है।

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह।

- प्राप्त करें [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स साझा करें, PRs को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में सबमिट करके।**

</details>
