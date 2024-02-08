# Volatility - CheatSheet

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

दूसरे तरीके HackTricks का समर्थन करने के लिए:

* अगर आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके।

</details>

​

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

​​[**RootedCON**](https://www.rootedcon.com/) स्पेन में सबसे महत्वपूर्ण साइबर सुरक्षा घटना है और यूरोप में सबसे महत्वपूर्ण में से एक है। **तकनीकी ज्ञान को बढ़ावा देने** के मिशन के साथ, यह कांग्रेस प्रौद्योगिकी और साइबर सुरक्षा विशेषज्ञों के लिए एक उबाऊ मिलन स्थल है।

{% embed url="https://www.rootedcon.com/" %}

अगर आप कुछ **तेज़ और पागल** चाहते हैं जो कई Volatility प्लगइन्स को समयानुसार चालू करेगा तो आप इस्तेमाल कर सकते हैं: [https://github.com/carlospolop/autoVolatility](https://github.com/carlospolop/autoVolatility)
```bash
python autoVolatility.py -f MEMFILE -d OUT_DIRECTORY -e /home/user/tools/volatility/vol.py # It will use the most important plugins (could use a lot of space depending on the size of the memory)
```
## स्थापना

### volatility3
```bash
git clone https://github.com/volatilityfoundation/volatility3.git
cd volatility3
python3 setup.py install
python3 vol.py —h
```
#### volatility2

{% tabs %}
{% tab title="Method1" %}
```
Download the executable from https://www.volatilityfoundation.org/26
```
{% endtab %}

{% tab title="Method 2" %}हिंदी{% endtab %}
```bash
git clone https://github.com/volatilityfoundation/volatility.git
cd volatility
python setup.py install
```
{% endtab %}
{% endtabs %}

## Volatility Commands

आधिकारिक दस्तावेज़ में पहुंचें [Volatility command reference](https://github.com/volatilityfoundation/volatility/wiki/Command-Reference#kdbgscan)

### "सूची" बनाम "स्कैन" प्लगइन्स पर एक नोट

Volatility के दो मुख्य प्लगइन्स के लिए दो मुख्य दृष्टिकोण हैं, जो कभी-कभी उनके नामों में प्रकट होते हैं। "सूची" प्लगइन्स विंडोज कर्नेल संरचनाओं के माध्यम से जानकारी प्राप्त करने के लिए प्रयास करेंगे जैसे प्रक्रियाएँ (यादृच्छिक रूप से मेमोरी में `_EPROCESS` संरचनाओं की लिंक्ड सूची को ढूंढें और चलें), ओएस हैंडल्स (हैंडल टेबल का पता लगाना, पाए गए किसी भी प्वाइंटर को डिरेफरेंस करना, आदि)। वे अधिकांश रूप से विंडोज API की तरह व्यवहार करते हैं जैसे यदि मांग की जाए कि प्रक्रियाएँ सूचीबद्ध की जाएं।

यह "सूची" प्लगइन्स काफी तेज होते हैं, लेकिन विंडोज API के तरह ही मैलवेयर द्वारा परिवर्तन के लिए विकल्पनीय हैं। उदाहरण के लिए, यदि मैलवेयर DKOM का उपयोग करके किसी प्रक्रिया को `_EPROCESS` लिंक्ड सूची से अलग करता है, तो वह टास्क प्रबंधक में प्रकट नहीं होगा और न ही यह pslist में दिखाई देगा।

दूसरी ओर, "स्कैन" प्लगइन्स एक ऐसा दृष्टिकोण अपनाएंगे जो यादृच्छिक संरचनाओं के रूप में डिरेफरेंस किए जाने पर समझ में आ सकती चीजों के लिए मेमोरी को कार्व करने के लिए। `psscan` उदाहरण के लिए मेमोरी पढ़ेगा और इससे `_EPROCESS` ऑब्ज
```bash
./volatility_2.6_lin64_standalone --info | grep "Profile"
```
यदि आप **एक नया प्रोफ़ाइल जिसे आपने डाउनलोड किया है** (उदाहरण के लिए एक लिनक्स वाला) उसे उपयोग करना चाहते हैं, तो आपको निम्नलिखित फ़ोल्डर संरचना बनानी होगी: _plugins/overlays/linux_ और इस फ़ोल्डर में प्रोफ़ाइल को समाहित करने वाली ज़िप फ़ाइल डालें। फिर, निम्नलिखित कमांड का उपयोग करके प्रोफ़ाइल की संख्या प्राप्त करें:
```bash
./vol --plugins=/home/kali/Desktop/ctfs/final/plugins --info
Volatility Foundation Volatility Framework 2.6


Profiles
--------
LinuxCentOS7_3_10_0-123_el7_x86_64_profilex64 - A Profile for Linux CentOS7_3.10.0-123.el7.x86_64_profile x64
VistaSP0x64                                   - A Profile for Windows Vista SP0 x64
VistaSP0x86                                   - A Profile for Windows Vista SP0 x86
```
आप **Linux और Mac profiles** को [https://github.com/volatilityfoundation/profiles](https://github.com/volatilityfoundation/profiles) से डाउनलोड कर सकते हैं।

पिछले चंक में आप देख सकते हैं कि प्रोफ़ाइल का नाम `LinuxCentOS7_3_10_0-123_el7_x86_64_profilex64` है, और आप इसका उपयोग कुछ इस प्रकार से कर सकते हैं:
```bash
./vol -f file.dmp --plugins=. --profile=LinuxCentOS7_3_10_0-123_el7_x86_64_profilex64 linux_netscan
```
#### खोज प्रोफ़ाइल
```
volatility imageinfo -f file.dmp
volatility kdbgscan -f file.dmp
```
#### **imageinfo और kdbgscan के बीच अंतर**

[**यहाँ से**](https://www.andreafortuna.org/2017/06/25/volatility-my-own-cheatsheet-part-1-image-identification/): imageinfo के विपरीत, जो केवल प्रोफ़ाइल सुझाव प्रदान करता है, **kdbgscan** का उद्देश्य सही प्रोफ़ाइल और सही KDBG पता (यदि कई हों) की पहचान करना है। यह प्लगइन Volatility प्रोफ़ाइल्स से जुड़े KDBGHeader हस्ताक्षरों के लिए स्कैन करता है और गलत सक्रिय परिणाम को कम करने के लिए सानिति जांच लागू करता है। आउटपुट की व्यापकता और सानिति जांचों की संख्या इस पर निर्भर करती है कि Volatility क्या एक DTB ढूंढ सकता है, इसलिए यदि आपको पहले से सही प्रोफ़ाइल पता है (या यदि आपके पास imageinfo से प्रोफ़ाइल सुझाव है), तो सुनिश्चित करें कि आप इसे से उपयोग करते हैं।

हमेशा **देखें कि kdbgscan ने कितने प्रक्रियाएँ खोजी हैं**। कभी-कभी imageinfo और kdbgscan **एक से अधिक** उपयुक्त **प्रोफ़ाइल** ढूंढ सकते हैं, लेकिन केवल **वैध एक में कुछ प्रक्रिया संबंधित** होगा (यह इसलिए है कि सही KDBG पता निकालने के लिए प्रक्रियाएँ आवश्यक हैं)
```bash
# GOOD
PsActiveProcessHead           : 0xfffff800011977f0 (37 processes)
PsLoadedModuleList            : 0xfffff8000119aae0 (116 modules)
```

```bash
# BAD
PsActiveProcessHead           : 0xfffff800011947f0 (0 processes)
PsLoadedModuleList            : 0xfffff80001197ac0 (0 modules)
```
#### KDBG

**कर्नेल डीबगर ब्लॉक**, जिसे Volatility द्वारा **KDBG** के रूप में संदर्भित किया जाता है, जांचकर्ताओं द्वारा किए जाने वाले फोरेंसिक कार्यों के लिए महत्वपूर्ण है। `_KDDEBUGGER_DATA64` नामक और `_KDDEBUGGER_DATA64` प्रकार का `KdDebuggerDataBlock` के रूप में पहचाना जाता है, जिसमें `PsActiveProcessHead` जैसे महत्वपूर्ण संदर्भ होते हैं। यह विशिष्ट संदर्भ सभी प्रक्रियाओं की सूची बनाने की क्षमता को सक्षम करता है, जो समग्र मेमोरी विश्लेषण के लिए मौलिक है।

## ओएस सूचना
```bash
#vol3 has a plugin to give OS information (note that imageinfo from vol2 will give you OS info)
./vol.py -f file.dmp windows.info.Info
```
यह प्लगइन `banners.Banners` डंप में **लिनक्स बैनर्स** खोजने के लिए **vol3 में** उपयोग किया जा सकता है।

## Hashes/Passwords

SAM हैश, [डोमेन कैश्ड क्रेडेंशियल्स](../../../windows-hardening/stealing-credentials/credentials-protections.md#cached-credentials) और [lsa सीक्रेट्स](../../../windows-hardening/authentication-credentials-uac-and-efs.md#lsa-secrets) को निकालें।

{% tabs %}
{% tab title="vol3" %}
```bash
./vol.py -f file.dmp windows.hashdump.Hashdump #Grab common windows hashes (SAM+SYSTEM)
./vol.py -f file.dmp windows.cachedump.Cachedump #Grab domain cache hashes inside the registry
./vol.py -f file.dmp windows.lsadump.Lsadump #Grab lsa secrets
```
{% endtab %}

{% tab title="vol2" %}यहाँ वोलेटिलिटी चीटशीट का अनुभाग है। यह चीटशीट वोलेटिलिटी फ्रेमवर्क के उपयोग के लिए महत्वपूर्ण तकनीकी संसाधनों का सारांश प्रदान करता है। यह विभिन्न मेमोरी डंप विश्लेषण तकनीकों के लिए उपयुक्त आदेशों को संकलित करता है। यह चीटशीट विभिन्न विशेषज्ञताओं के लिए उपयोगी है जो डिजिटल जांच और डेटा रिकवरी के क्षेत्र में काम करते ह।{% endtab %}
```bash
volatility --profile=Win7SP1x86_23418 hashdump -f file.dmp #Grab common windows hashes (SAM+SYSTEM)
volatility --profile=Win7SP1x86_23418 cachedump -f file.dmp #Grab domain cache hashes inside the registry
volatility --profile=Win7SP1x86_23418 lsadump -f file.dmp #Grab lsa secrets
```
## मेमोरी डंप

प्रक्रिया का मेमोरी डंप प्रक्रिया की वर्तमान स्थिति को **निकालेगा**। **Procdump** मॉड्यूल केवल **कोड** को **निकालेगा**।
```
volatility -f file.dmp --profile=Win7SP1x86 memdump -p 2168 -D conhost/
```
<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

​​​[**RootedCON**](https://www.rootedcon.com/) स्पेन में सबसे महत्वपूर्ण साइबर सुरक्षा घटना है और यूरोप में सबसे महत्वपूर्ण में से एक है। **तकनीकी ज्ञान को बढ़ावा देने** के मिशन के साथ, यह कांग्रेस प्रौद्योगिकी और साइबर सुरक्षा विशेषज्ञों के लिए एक उफनती बैठक स्थान है हर विषय में।

{% embed url="https://www.rootedcon.com/" %}

## Processes

### List processes

**संदेहजनक** प्रक्रियाओं (नाम द्वारा) या **अप्रत्याशित** बच्चा **प्रक्रियाएं** (उदाहरण के लिए iexplorer.exe के बच्चे के रूप में cmd.exe) खोजने का प्रयास करें।\
यह देखने के लिए दिलचस्प हो सकता है कि pslist के परिणाम को psscan के साथ तुलना करें छुपी हुई प्रक्रियाओं की पहचान करने के लिए।

{% tabs %}
{% tab title="vol3" %}
```bash
python3 vol.py -f file.dmp windows.pstree.PsTree # Get processes tree (not hidden)
python3 vol.py -f file.dmp windows.pslist.PsList # Get process list (EPROCESS)
python3 vol.py -f file.dmp windows.psscan.PsScan # Get hidden process list(malware)
```
{% endtab %}

{% tab title="vol2" %}वोलेटिलिटी चीटशीट

### वोलेटिलिटी चीटशीट

#### वोलेटिलिटी चीटशीट

- **वोलेटिलिटी इंस्टॉलेशन चेक:**
  - `volatility -h`

- **डेटा प्राप्ति:**
  - `volatility -f <डंप फ़ाइल> imageinfo`
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़ाइल> pslist`

- **कर्नल मेमोरी डंप:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़ाइल> memdump -p <प्रोसेस ID> -D <आउटपुट डिरेक्टरी>`

- **कर्नल मेमोरी डंप (ऑल प्रोसेसेस):**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़ाइल> memdump -D <आउटपुट डिरेक्टरी>`

- **डेटा एनालिसिस:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़ाइल> consoles`
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़ाइल> cmdscan`

- **नेटवर्क एनालिसिस:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़ाइल> netscan`

- **रजिस्ट्री एनालिसिस:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़ाइल> printkey -o <रजिस्ट्री की पथ>`

- **डेटा एक्सफ़िल्ट्रेशन:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़ाइल> filescan`
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़ाइल> dumpfiles -Q <फ़ाइल का पथ> -D <आउटपुट डिरेक्टरी>`

- **डेटा फिंगरप्रिंटिंग:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़ाइल> shimcache`

- **डेटा डिफ़्फ़िंग:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़ाइल> malfind`

- **डेटा विश्लेषण:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़ाइल> pstree`

- **डेटा इंजेक्शन:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़ाइल> malfind`

- **डेटा इंजेक्शन:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़इल> malfind`

- **डेटा इंजेक्शन:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़इल> malfind`

- **डेटा इंजेक्शन:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़इल> malfind`

- **डेटा इंजेक्शन:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़इल> malfind`

- **डेटा इंजेक्शन:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़इल> malfind`

- **डेटा इंजेक्शन:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़इल> malfind`

- **डेटा इंजेक्शन:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़इल> malfind`

- **डेटा इंजेक्शन:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़इल> malfind`

- **डेटा इंजेक्शन:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़इल> malfind`

- **डेटा इंजेक्शन:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़इल> malfind`

- **डेटा इंजेक्शन:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़इल> malfind`

- **डेटा इंजेक्शन:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़इल> malfind`

- **डेटा इंजेक्शन:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़इल> malfind`

- **डेटा इंजेक्शन:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़इल> malfind`

- **डेटा इंजेक्शन:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़इल> malfind`

- **डेटा इंजेक्शन:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़इल> malfind`

- **डेटा इंजेक्शन:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़इल> malfind`

- **डेटा इंजेक्शन:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़इल> malfind`

- **डेटा इंजेक्शन:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़इल> malfind`

- **डेटा इंजेक्शन:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़इल> malfind`

- **डेटा इंजेक्शन:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़इल> malfind`

- **डेटा इंजेक्शन:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़इल> malfind`

- **डेटा इंजेक्शन:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़इल> malfind`

- **डेटा इंजेक्शन:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़इल> malfind`

- **डेटा इंजेक्शन:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़इल> malfind`

- **डेटा इंजेक्शन:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़इल> malfind`

- **डेटा इंजेक्शन:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़इल> malfind`

- **डेटा इंजेक्शन:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़इल> malfind`

- **डेटा इंजेक्शन:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़इल> malfind`

- **डेटा इंजेक्शन:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़इल> malfind`

- **डेटा इंजेक्शन:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़इल> malfind`

- **डेटा इंजेक्शन:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़इल> malfind`

- **डेटा इंजेक्शन:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़इल> malfind`

- **डेटा इंजेक्शन:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़इल> malfind`

- **डेटा इंजेक्शन:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़इल> malfind`

- **डेटा इंजेक्शन:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़इल> malfind`

- **डेटा इंजेक्शन:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़इल> malfind`

- **डेटा इंजेक्शन:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़इल> malfind`

- **डेटा इंजेक्शन:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़इल> malfind`

- **डेटा इंजेक्शन:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़इल> malfind`

- **डेटा इंजेक्शन:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़इल> malfind`

- **डेटा इंजेक्शन:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़इल> malfind`

- **डेटा इंजेक्शन:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़इल> malfind`

- **डेटा इंजेक्शन:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़इल> malfind`

- **डेटा इंजेक्शन:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़इल> malfind`

- **डेटा इंजेक्शन:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़इल> malfind`

- **डेटा इंजेक्शन:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़इल> malfind`

- **डेटा इंजेक्शन:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़इल> malfind`

- **डेटा इंजेक्शन:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़इल> malfind`

- **डेटा इंजेक्शन:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़इल> malfind`

- **डेटा इंजेक्शन:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़इल> malfind`

- **डेटा इंजेक्शन:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़इल> malfind`

- **डेटा इंजेक्शन:**
  - `volatility -f <डंप फ़ाइल> --profile=<प्रोफ़इल> malfind`

- **डेटा इं
```bash
volatility --profile=PROFILE pstree -f file.dmp # Get process tree (not hidden)
volatility --profile=PROFILE pslist -f file.dmp # Get process list (EPROCESS)
volatility --profile=PROFILE psscan -f file.dmp # Get hidden process list(malware)
volatility --profile=PROFILE psxview -f file.dmp # Get hidden process list
```
### डंप प्रोस

{% tabs %}
{% tab title="vol3" %}
```bash
./vol.py -f file.dmp windows.dumpfiles.DumpFiles --pid <pid> #Dump the .exe and dlls of the process in the current directory
```
{% endtab %}

{% tab title="vol2" %}वोलेटिलिटी चीटशीट

### वोलेटिलिटी चीटशीट

#### वोलेटिलिटी इंस्टॉलेशन

वोलेटिलिटी इंस्टॉलेशन की जांच करने के लिए निम्नलिखित आदेश का पालन करें:

```bash
sudo apt-get install python-dev python-pip gcc
sudo pip install openpyxl
sudo pip install ujson
sudo pip install pycrypto
sudo pip install distorm3
sudo pip install pytz
sudo pip install yara
sudo pip install openpyxl
sudo pip install ujson
sudo pip install pycrypto
sudo pip install distorm3
sudo pip install pytz
sudo pip install yara
```

#### वोलेटिलिटी उपयोग

वोलेटिलिटी का उपयोग करने के लिए निम्नलिखित आदेश का पालन करें:

```bash
volatility -f <memory_dump> <plugin_name>
```

उदाहरण:

```bash
volatility -f memdump.mem imageinfo
```

#### वोलेटिलिटी उपयोगकर्ता गाइड

वोलेटिलिटी के उपयोग के लिए उपयोगकर्ता गाइड के लिए [यहाँ](https://github.com/volatilityfoundation/volatility/wiki) देखें।

#### वोलेटिलिटी चीटशीट

वोलेटिलिटी के अधिक आदेशों और उपकरणों के लिए वोलेटिलिटी चीटशीट को देखें।

{% endtab %}
```bash
volatility --profile=Win7SP1x86_23418 procdump --pid=3152 -n --dump-dir=. -f file.dmp
```
{% endtab %}
{% endtabs %}

### कमांड लाइन

क्या कोई संदेहास्पद क्रियाएँ की गई थीं?
```bash
python3 vol.py -f file.dmp windows.cmdline.CmdLine #Display process command-line arguments
```
{% endtab %}

{% टैब शीर्षक = "vol2" %}
```bash
volatility --profile=PROFILE cmdline -f file.dmp #Display process command-line arguments
volatility --profile=PROFILE consoles -f file.dmp #command history by scanning for _CONSOLE_INFORMATION
```
{% endtab %}
{% endtabs %}

`cmd.exe` में चलाए गए कमांड **`conhost.exe`** (या Windows 7 से पहले सिस्टम पर `csrss.exe`) द्वारा प्रबंधित किए जाते हैं। इसका मतलब है कि यदि किसी हमलावर द्वारा **`cmd.exe`** को बंद कर दिया जाता है पहले मेमोरी डंप प्राप्त किया जाता है, तो फिर भी **`conhost.exe`** की मेमोरी से सत्र का कमांड हिस्ट्री पुनः प्राप्त की जा सकती है। इसे करने के लिए, यदि कंसोल के मॉड्यूल में असामान्य गतिविधि का पता चलता है, तो संबंधित **`conhost.exe`** प्रक्रिया की मेमोरी डंप की जानी चाहिए। फिर, इस डंप में **स्ट्रिंग्स** की खोज करके, सत्र में उपयोग किए गए कमांड लाइन्स को संभावित रूप से निकाला जा सकता है।

### पर्यावरण

प्रत्येक चल रही प्रक्रिया की एनवायरनमेंट वेरिएबल्स प्राप्त करें। कुछ दिलचस्प मान्यताएँ हो सकती हैं।
```bash
python3 vol.py -f file.dmp windows.envars.Envars [--pid <pid>] #Display process environment variables
```
{% endtab %}

{% tab title="vol2" %}वोलेटिलिटी चीटशीट

### वोलेटिलिटी चीटशीट

#### वोलेटिलिटी इंस्टॉलेशन

- वोलेटिलिटी इंस्टॉलेशन की जांच करें
  ```bash
  volatility -h
  ```

#### वोलेटिलिटी फाइल सिस्टम

- फाइल सिस्टम की जांच करें
  ```bash
  volatility -f <file> imageinfo
  ```

#### वोलेटिलिटी प्रोसेसेस

- प्रोसेसेस की सूची प्राप्त करें
  ```bash
  volatility -f <file> --profile=<profile> pslist
  ```

#### वोलेटिलिटी नेटवर्क

- नेटवर्क कनेक्शन्स की जांच करें
  ```bash
  volatility -f <file> --profile=<profile> connections
  ```

#### वोलेटिलिटी रेजिस्ट्री

- रेजिस्ट्री की जांच करें
  ```bash
  volatility -f <file> --profile=<profile> hivelist
  ```

#### वोलेटिलिटी फाइल्स

- फाइल्स की सूची प्राप्त करें
  ```bash
  volatility -f <file> --profile=<profile> filescan
  ```

#### वोलेटिलिटी टास्क्स

- टास्क्स की सूची प्राप्त करें
  ```bash
  volatility -f <file> --profile=<profile> pstree
  ```

#### वोलेटिलिटी डंप

- मेमोरी डंप निकालें
  ```bash
  volatility -f <file> --profile=<profile> memdump -p <pid> -D <output_directory>
  ```

#### वोलेटिलिटी यूजर्स

- यूजर्स की सूची प्राप्त करें
  ```bash
  volatility -f <file> --profile=<profile> getsids
  ```

#### वोलेटिलिटी इंजेक्शन

- इंजेक्शन की जांच करें
  ```bash
  volatility -f <file> --profile=<profile> malfind
  ```

#### वोलेटिलिटी रिपोर्ट्स

- रिपोर्ट्स बनाएं
  ```bash
  volatility -f <file> --profile=<profile> report -O <output_directory>
  ```
```bash
volatility --profile=PROFILE envars -f file.dmp [--pid <pid>] #Display process environment variables

volatility --profile=PROFILE -f file.dmp linux_psenv [-p <pid>] #Get env of process. runlevel var means the runlevel where the proc is initated
```
### टोकन विशेषाधिकार

अप्रत्याशित सेवाओं में विशेषाधिकार टोकन की जांच करें।
कुछ विशेषाधिकृत टोकन का उपयोग करने वाले प्रक्रियाओं की सूची बनाना दिलचस्प हो सकता है।
```bash
#Get enabled privileges of some processes
python3 vol.py -f file.dmp windows.privileges.Privs [--pid <pid>]
#Get all processes with interesting privileges
python3 vol.py -f file.dmp windows.privileges.Privs | grep "SeImpersonatePrivilege\|SeAssignPrimaryPrivilege\|SeTcbPrivilege\|SeBackupPrivilege\|SeRestorePrivilege\|SeCreateTokenPrivilege\|SeLoadDriverPrivilege\|SeTakeOwnershipPrivilege\|SeDebugPrivilege"
```
{% endtab %}

{% tab title="vol2" %}यहाँ वोलेटिलिटी चीटशीट का अनुभाग है। यह चीटशीट वोलेटिलिटी फ्रेमवर्क के उपयोग के लिए महत्वपूर्ण तकनीकों को संक्षेपित रूप से प्रदान करती है। यह विभिन्न मेमोरी डंप विश्लेषण के लिए उपयोगी आदेशों का संग्रह है। यह चीटशीट विभिन्न ऑपरेटिंग सिस्टम्स पर वोलेटिलिटी का उपयोग करते समय आपकी मदद करेगी।{% endtab %}
```bash
#Get enabled privileges of some processes
volatility --profile=Win7SP1x86_23418 privs --pid=3152 -f file.dmp | grep Enabled
#Get all processes with interesting privileges
volatility --profile=Win7SP1x86_23418 privs -f file.dmp | grep "SeImpersonatePrivilege\|SeAssignPrimaryPrivilege\|SeTcbPrivilege\|SeBackupPrivilege\|SeRestorePrivilege\|SeCreateTokenPrivilege\|SeLoadDriverPrivilege\|SeTakeOwnershipPrivilege\|SeDebugPrivilege"
```
{% endtab %}
{% endtabs %}

### एसआईडी

प्रक्रिया द्वारा स्वामित्व में रखे गए प्रत्येक एसएसआईडी की जांच करें।\
यह दिलचस्प हो सकता है कि एक विशेषाधिकार एसआईडी का उपयोग करने वाली प्रक्रियाओं की सूची बनाना (और कुछ सेवा एसआईडी का उपयोग करने वाली प्रक्रियाओं की सूची बनाना)।
```bash
./vol.py -f file.dmp windows.getsids.GetSIDs [--pid <pid>] #Get SIDs of processes
./vol.py -f file.dmp windows.getservicesids.GetServiceSIDs #Get the SID of services
```
{% endtab %}

{% टैब शीर्षक = "vol2" %}
```bash
volatility --profile=Win7SP1x86_23418 getsids -f file.dmp #Get the SID owned by each process
volatility --profile=Win7SP1x86_23418 getservicesids -f file.dmp #Get the SID of each service
```
### हैंडल्स

प्रक्रिया के पास हैंडल है किसी अन्य फ़ाइल, कुंजियों, धागों, प्रक्रियाओं... के लिए (खोल रखी है) किसे जानने के लिए उपयोगी हैं।
```bash
vol.py -f file.dmp windows.handles.Handles [--pid <pid>]
```
{% endtab %}

{% tab title="vol2" %}वोलेटिलिटी चीटशीट

### वोलेटिलिटी चीटशीट

#### वोलेटिलिटी कमांड्स

- **डिबगिंग कमांड्स**
  - `pslist`: सिस्टम प्रोसेस की सूची प्रदर्शित करें
  - `pstree`: प्रोसेस का पेड़ डिस्प्ले करें
- **नेटवर्क कमांड्स**
  - `netscan`: नेटवर्क कनेक्शन की सूची प्रदर्शित करें
  - `sockets`: सॉकेट की सूची प्रदर्शित करें
- **फाइल सिस्टम कमांड्स**
  - `filescan`: खुले फाइल की सूची प्रदर्शित करें
  - `cmdscan`: कमांड रेजिस्ट्री की सूची प्रदर्शित करें

#### वोलेटिलिटी प्लगइन्स

- **डिबगिंग प्लगइन्स**
  - `malfind`: संदिग्ध प्रक्रियाओं की खोज करें
  - `apihooks`: API हुक्स की सूची प्रदर्शित करें
- **नेटवर्क प्लगइन्स**
  - `connscan`: कनेक्शन की सूची प्रदर्शित करें
  - `sockscan`: सॉकेट की सूची प्रदर्शित करें
- **फाइल सिस्टम प्लगइन्स**
  - `filescan`: खुले फाइल की सूची प्रदर्शित करें
  - `cmdscan`: कमांड रेजिस्ट्री की सूची प्रदर्शित करें

#### वोलेटिलिटी उपयोगिता

- **डिबगिंग उपयोगिता**
  - `vol.py -f <डंप फ़ाइल> --profile=<प्रोफ़ाइल> <कमांड>`
- **नेटवर्क उपयोगिता**
  - `vol.py -f <डंप फ़ाइल> --profile=<प्रोफ़ाइल> <कमांड>`
- **फाइल सिस्टम उपयोगिता**
  - `vol.py -f <डंप फ़ाइल> --profile=<प्रोफ़ाइल> <कमांड>`{% endtab %}
```bash
volatility --profile=Win7SP1x86_23418 -f file.dmp handles [--pid=<pid>]
```
{% endtab %}
{% endtabs %}

### DLLs

{% tabs %}
{% tab title="vol3" %}
```bash
./vol.py -f file.dmp windows.dlllist.DllList [--pid <pid>] #List dlls used by each
./vol.py -f file.dmp windows.dumpfiles.DumpFiles --pid <pid> #Dump the .exe and dlls of the process in the current directory process
```
{% endtab %}

{% tab title="vol2" %}वोलेटिलिटी चीटशीट

### वोलेटिलिटी चीटशीट

#### वोलेटिलिटी इंस्टॉलेशन

- वोलेटिलिटी इंस्टॉल करें: `pip install volatility`

#### बुनियादी उपकरण

- वोलेटिलिटी का उपयोग करने के लिए बुनियादी उपकरण:
  - `imageinfo`: इमेज की मेटाडेटा प्रदान करता हृ
  - `pslist`: प्रक्रियाओं की सूची प्रदान करता हृ
  - `pstree`: प्रक्रियाओं का पेड़ प्रदान करता हृ
  - `psscan`: संदर्भ स्थिति स्कैन करता हृ
  - `dlllist`: DLL सूची प्रदान करता हृ
  - `handles`: हैंडल सूची प्रदान करता हृ
  - `cmdline`: प्रक्रिया की कमांड लाइन प्रदान करता ह
```bash
volatility --profile=Win7SP1x86_23418 dlllist --pid=3152 -f file.dmp #Get dlls of a proc
volatility --profile=Win7SP1x86_23418 dlldump --pid=3152 --dump-dir=. -f file.dmp #Dump dlls of a proc
```
### प्रक्रियाओं के लिए स्ट्रिंग्स

Volatility हमें यह जांचने की अनुमति देता है कि एक स्ट्रिंग किस प्रक्रिया से संबंधित है।
```bash
strings file.dmp > /tmp/strings.txt
./vol.py -f /tmp/file.dmp windows.strings.Strings --strings-file /tmp/strings.txt
```
{% endtab %}

{% टैब शीर्षक = "vol2" %}
```bash
strings file.dmp > /tmp/strings.txt
volatility -f /tmp/file.dmp windows.strings.Strings --string-file /tmp/strings.txt

volatility -f /tmp/file.dmp --profile=Win81U1x64 memdump -p 3532 --dump-dir .
strings 3532.dmp > strings_file
```
यह भी एक प्रक्रिया के अंदर स्ट्रिंग की खोज करने की अनुमति देता है जिसका उपयोग yarascan मॉड्यूल का उपयोग करके किया जा सकता है:
```bash
./vol.py -f file.dmp windows.vadyarascan.VadYaraScan --yara-rules "https://" --pid 3692 3840 3976 3312 3084 2784
./vol.py -f file.dmp yarascan.YaraScan --yara-rules "https://"
```
{% endtab %}

{% टैब शीर्षक = "vol2" %}
```bash
volatility --profile=Win7SP1x86_23418 yarascan -Y "https://" -p 3692,3840,3976,3312,3084,2784
```
### UserAssist

**Windows** उनके उपयोगकर्ता द्वारा चलाए गए कार्यक्रमों का ट्रैक रखता है जिसके लिए रजिस्ट्री में एक विशेषता उपयोग की जाती है जिसे **UserAssist keys** कहा जाता है। ये कुंजी यह दर्ज करती हैं कि प्रत्येक कार्यक्रम कितनी बार चलाया गया है और यह कब आखिरी बार चलाया गया था।
```bash
./vol.py -f file.dmp windows.registry.userassist.UserAssist
```
{% endtab %}

{% टैब शीर्षक = "vol2" %}
```
volatility --profile=Win7SP1x86_23418 -f file.dmp userassist
```
{% endtab %}
{% endtabs %}

​

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

​​​​[**RootedCON**](https://www.rootedcon.com/) स्पेन में सबसे महत्वपूर्ण साइबर सुरक्षा घटना है और यूरोप में सबसे महत्वपूर्ण में से एक है। **तकनीकी ज्ञान को बढ़ावा देने की मिशन** के साथ, यह कांग्रेस प्रौद्योगिकी और साइबर सुरक्षा विशेषज्ञों के लिए एक उफनती मिलन स्थल है हर विषय में।

{% embed url="https://www.rootedcon.com/" %}

## सेवाएं

{% tabs %}
{% tab title="vol3" %}
```bash
./vol.py -f file.dmp windows.svcscan.SvcScan #List services
./vol.py -f file.dmp windows.getservicesids.GetServiceSIDs #Get the SID of services
```
{% endtab %}

{% tab title="vol2" %}हैकिंग तकनीकियों की एक पुस्तक से निम्नलिखित सामग्री है। निम्नलिखित सामग्री फ़ाइल generic-methodologies-and-resources/basic-forensic-methodology/memory-dump-analysis/volatility-cheatsheet.md से है। अंग्रेजी मूल सामग्री का हिंदी में अनुवाद करें और अनुवाद को वही मार्कडाउन और एचटीएमएल विन्यास बनाए रखें। कोड, हैकिंग तकनीक नाम, हैकिंग शब्द, क्लाउड/एसएएस प्लेटफ़ॉर्म नाम (जैसे Workspace, aws, gcp...), 'लीक', पेंटेस्टिंग और मार्कडाउन टैग्स जैसी चीजें अनुवाद न करें। अनुवाद के अलावा कोई अतिरिक्त सामग्री न जोड़ें।%}
```bash
#Get services and binary path
volatility --profile=Win7SP1x86_23418 svcscan -f file.dmp
#Get name of the services and SID (slow)
volatility --profile=Win7SP1x86_23418 getservicesids -f file.dmp
```
## नेटवर्क

{% tabs %}
{% tab title="vol3" %}
```bash
./vol.py -f file.dmp windows.netscan.NetScan
#For network info of linux use volatility2
```
{% endtab %}

{% टैब शीर्षक="vol2" %}
```bash
volatility --profile=Win7SP1x86_23418 netscan -f file.dmp
volatility --profile=Win7SP1x86_23418 connections -f file.dmp#XP and 2003 only
volatility --profile=Win7SP1x86_23418 connscan -f file.dmp#TCP connections
volatility --profile=Win7SP1x86_23418 sockscan -f file.dmp#Open sockets
volatility --profile=Win7SP1x86_23418 sockets -f file.dmp#Scanner for tcp socket objects

volatility --profile=SomeLinux -f file.dmp linux_ifconfig
volatility --profile=SomeLinux -f file.dmp linux_netstat
volatility --profile=SomeLinux -f file.dmp linux_netfilter
volatility --profile=SomeLinux -f file.dmp linux_arp #ARP table
volatility --profile=SomeLinux -f file.dmp linux_list_raw #Processes using promiscuous raw sockets (comm between processes)
volatility --profile=SomeLinux -f file.dmp linux_route_cache
```
## रजिस्ट्री हाइव

### उपलब्ध हाइव्स प्रिंट करें

{% tabs %}
{% tab title="vol3" %}
```bash
./vol.py -f file.dmp windows.registry.hivelist.HiveList #List roots
./vol.py -f file.dmp windows.registry.printkey.PrintKey #List roots and get initial subkeys
```
{% endtab %}

{% टैब शीर्षक = "vol2" %}
```bash
volatility --profile=Win7SP1x86_23418 -f file.dmp hivelist #List roots
volatility --profile=Win7SP1x86_23418 -f file.dmp printkey #List roots and get initial subkeys
```
### एक मान प्राप्त करें

{% tabs %}
{% tab title="vol3" %}
```bash
./vol.py -f file.dmp windows.registry.printkey.PrintKey --key "Software\Microsoft\Windows NT\CurrentVersion"
```
{% endtab %}

{% टैब शीर्षक = "vol2" %}
```bash
volatility --profile=Win7SP1x86_23418 printkey -K "Software\Microsoft\Windows NT\CurrentVersion" -f file.dmp
# Get Run binaries registry value
volatility -f file.dmp --profile=Win7SP1x86 printkey -o 0x9670e9d0 -K 'Software\Microsoft\Windows\CurrentVersion\Run'
```
### डंप
```bash
#Dump a hive
volatility --profile=Win7SP1x86_23418 hivedump -o 0x9aad6148 -f file.dmp #Offset extracted by hivelist
#Dump all hives
volatility --profile=Win7SP1x86_23418 hivedump -f file.dmp
```
## फ़ाइल सिस्टम

### माउंट

{% tabs %}
{% tab title="vol3" %}
```bash
#See vol2
```
{% endtab %}

{% टैब शीर्षक = "vol2" %}
```bash
volatility --profile=SomeLinux -f file.dmp linux_mount
volatility --profile=SomeLinux -f file.dmp linux_recover_filesystem #Dump the entire filesystem (if possible)
```
{% endtab %}
{% endtabs %}

### स्कैन/डंप

{% tabs %}
{% tab title="vol3" %}
```bash
./vol.py -f file.dmp windows.filescan.FileScan #Scan for files inside the dump
./vol.py -f file.dmp windows.dumpfiles.DumpFiles --physaddr <0xAAAAA> #Offset from previous command
```
{% endtab %}

{% टैब शीर्षक="vol2" %}
```bash
volatility --profile=Win7SP1x86_23418 filescan -f file.dmp #Scan for files inside the dump
volatility --profile=Win7SP1x86_23418 dumpfiles -n --dump-dir=/tmp -f file.dmp #Dump all files
volatility --profile=Win7SP1x86_23418 dumpfiles -n --dump-dir=/tmp -Q 0x000000007dcaa620 -f file.dmp

volatility --profile=SomeLinux -f file.dmp linux_enumerate_files
volatility --profile=SomeLinux -f file.dmp linux_find_file -F /path/to/file
volatility --profile=SomeLinux -f file.dmp linux_find_file -i 0xINODENUMBER -O /path/to/dump/file
```
### मास्टर फ़ाइल टेबल

{% tabs %}
{% tab title="vol3" %}
```bash
# I couldn't find any plugin to extract this information in volatility3
```
{% endtab %}

{% टैब शीर्षक="vol2" %}
```bash
volatility --profile=Win7SP1x86_23418 mftparser -f file.dmp
```
{% endtab %}
{% endtabs %}

**NTFS फ़ाइल सिस्टम** एक महत्वपूर्ण घटक का उपयोग करता है जिसे _मास्टर फ़ाइल टेबल_ (MFT) के रूप में जाना जाता है। इस तालिका में कम से कम एक प्रविष्टि होती है हर वॉल्यूम पर हर फ़ाइल के लिए, MFT खुद को भी शामिल करते हुए। प्रत्येक फ़ाइल के बारे में महत्वपूर्ण विवरण, जैसे **आकार, समय चिह्न, अनुमतियाँ, और वास्तविक डेटा**, MFT प्रविष्टियों में या MFT के बाहरी क्षेत्रों में संकेतित किए गए इन प्रविष्टियों द्वारा बंधी होते हैं। अधिक विवरण [आधिकारिक दस्तावेज़ीकरण](https://docs.microsoft.com/en-us/windows/win32/fileio/master-file-table) में उपलब्ध है।

### SSL कुंजी/प्रमाणपत्र
```bash
#vol3 allows to search for certificates inside the registry
./vol.py -f file.dmp windows.registry.certificates.Certificates
```
{% endtab %}

{% tab title="vol2" %}हैकिंग तकनीकियों की एक पुस्तक से निम्नलिखित सामग्री है। निम्नलिखित सामग्री फ़ाइल generic-methodologies-and-resources/basic-forensic-methodology/memory-dump-analysis/volatility-cheatsheet.md से है। अंग्रेजी मूल सामग्री का हिंदी में अनुवाद करें और अनुवाद को वही मार्कडाउन और एचटीएमएल विन्यास बनाए रखें। कोड, हैकिंग तकनीक नाम, हैकिंग शब्द, क्लाउड/एसएएस प्लेटफ़ॉर्म नाम (जैसे Workspace, aws, gcp...), 'लीक', पेंटेस्टिंग और मार्कडाउन टैग्स जैसी चीजें अनुवाद न करें। अनुवाद में और कोई अतिरिक्त सामग्री न जोड़ें, केवल अनुवाद और मार्कडाउन विन्यास रखें।{% endtab %}
```bash
#vol2 allos you to search and dump certificates from memory
#Interesting options for this modules are: --pid, --name, --ssl
volatility --profile=Win7SP1x86_23418 dumpcerts --dump-dir=. -f file.dmp
```
## मैलवेयर

{% tabs %}
{% tab title="vol3" %}
```bash
./vol.py -f file.dmp windows.malfind.Malfind [--dump] #Find hidden and injected code, [dump each suspicious section]
#Malfind will search for suspicious structures related to malware
./vol.py -f file.dmp windows.driverirp.DriverIrp #Driver IRP hook detection
./vol.py -f file.dmp windows.ssdt.SSDT #Check system call address from unexpected addresses

./vol.py -f file.dmp linux.check_afinfo.Check_afinfo #Verifies the operation function pointers of network protocols
./vol.py -f file.dmp linux.check_creds.Check_creds #Checks if any processes are sharing credential structures
./vol.py -f file.dmp linux.check_idt.Check_idt #Checks if the IDT has been altered
./vol.py -f file.dmp linux.check_syscall.Check_syscall #Check system call table for hooks
./vol.py -f file.dmp linux.check_modules.Check_modules #Compares module list to sysfs info, if available
./vol.py -f file.dmp linux.tty_check.tty_check #Checks tty devices for hooks
```
{% endtab %}

{% tab title="vol2" %}हैकिंग तकनीकियों की एक पुस्तक से निम्नलिखित सामग्री है। निम्नलिखित सामग्री फ़ाइल generic-methodologies-and-resources/basic-forensic-methodology/memory-dump-analysis/volatility-cheatsheet.md से है। अंग्रेजी में दिए गए पाठ का हिंदी में अनुवाद करें और अनुवाद को वही मार्कडाउन और एचटीएमएल विन्यास बनाए रखें। कोड, हैकिंग तकनीक नाम, हैकिंग शब्द, क्लाउड/एसएएस प्लेटफ़ॉर्म नाम (जैसे Workspace, aws, gcp...), 'लीक', पेंटेस्टिंग और मार्कडाउन टैग्स जैसी चीजें अनुवाद न करें। अनुवाद में और कोई अतिरिक्त सामग्री न जोड़ें, केवल अनुवाद और मार्कडाउन सिंटैक्स रखें।{% endtab %}
```bash
volatility --profile=Win7SP1x86_23418 -f file.dmp malfind [-D /tmp] #Find hidden and injected code [dump each suspicious section]
volatility --profile=Win7SP1x86_23418 -f file.dmp apihooks #Detect API hooks in process and kernel memory
volatility --profile=Win7SP1x86_23418 -f file.dmp driverirp #Driver IRP hook detection
volatility --profile=Win7SP1x86_23418 -f file.dmp ssdt #Check system call address from unexpected addresses

volatility --profile=SomeLinux -f file.dmp linux_check_afinfo
volatility --profile=SomeLinux -f file.dmp linux_check_creds
volatility --profile=SomeLinux -f file.dmp linux_check_fop
volatility --profile=SomeLinux -f file.dmp linux_check_idt
volatility --profile=SomeLinux -f file.dmp linux_check_syscall
volatility --profile=SomeLinux -f file.dmp linux_check_modules
volatility --profile=SomeLinux -f file.dmp linux_check_tty
volatility --profile=SomeLinux -f file.dmp linux_keyboard_notifiers #Keyloggers
```
{% endtab %}
{% endtabs %}

### यारा के साथ स्कैनिंग

इस स्क्रिप्ट का उपयोग करके गिथब से सभी यारा मैलवेयर नियम डाउनलोड और मर्ज करें: [https://gist.github.com/andreafortuna/29c6ea48adf3d45a979a78763cdc7ce9](https://gist.github.com/andreafortuna/29c6ea48adf3d45a979a78763cdc7ce9)\
_**rules**_ निर्देशिका बनाएं और इसे निष्पादित करें। इससे _**malware\_rules.yar**_ नाम की एक फ़ाइल बनेगी जिसमें सभी मैलवेयर के लिए यारा नियम होंगे।
```bash
wget https://gist.githubusercontent.com/andreafortuna/29c6ea48adf3d45a979a78763cdc7ce9/raw/4ec711d37f1b428b63bed1f786b26a0654aa2f31/malware_yara_rules.py
mkdir rules
python malware_yara_rules.py
#Only Windows
./vol.py -f file.dmp windows.vadyarascan.VadYaraScan --yara-file /tmp/malware_rules.yar
#All
./vol.py -f file.dmp yarascan.YaraScan --yara-file /tmp/malware_rules.yar
```
{% endtab %}

{% tab title="vol2" %}हिंदी{% endtab %}
```bash
wget https://gist.githubusercontent.com/andreafortuna/29c6ea48adf3d45a979a78763cdc7ce9/raw/4ec711d37f1b428b63bed1f786b26a0654aa2f31/malware_yara_rules.py
mkdir rules
python malware_yara_rules.py
volatility --profile=Win7SP1x86_23418 yarascan -y malware_rules.yar -f ch2.dmp | grep "Rule:" | grep -v "Str_Win32" | sort | uniq
```
## अन्य

### बाह्य प्लगइन

यदि आप बाह्य प्लगइन का उपयोग करना चाहते हैं तो सुनिश्चित करें कि प्लगइन से संबंधित फोल्डर पहला पैरामीटर है जो उपयोग किया जाता है।
```bash
./vol.py --plugin-dirs "/tmp/plugins/" [...]
```
{% endtab %}

{% tab title="vol2" %}हैकिंग तकनीकियों की एक पुस्तक से निम्नलिखित सामग्री है। निम्नलिखित सामग्री फ़ाइल generic-methodologies-and-resources/basic-forensic-methodology/memory-dump-analysis/volatility-cheatsheet.md से है। अंग्रेजी मूल सामग्री का हिंदी में अनुवाद करें और अनुवाद को वही मार्कडाउन और एचटीएमएल विन्यास बनाए रखें। कोड, हैकिंग तकनीक नाम, हैकिंग शब्द, क्लाउड/एसएएस प्लेटफ़ॉर्म नाम (जैसे Workspace, aws, gcp...), 'लीक', पेंटेस्टिंग और मार्कडाउन टैग्स जैसी चीजें अनुवाद न करें। अनुवाद के अलावा कोई अतिरिक्त सामग्री न जोड़ें।%}
```bash
volatilitye --plugins="/tmp/plugins/" [...]
```
#### Autoruns

इसे [https://github.com/tomchop/volatility-autoruns](https://github.com/tomchop/volatility-autoruns) से डाउनलोड करें।
```
volatility --plugins=volatility-autoruns/ --profile=WinXPSP2x86 -f file.dmp autoruns
```
### म्यूटेक्स

{% tabs %}
{% tab title="vol3" %}
```
./vol.py -f file.dmp windows.mutantscan.MutantScan
```
{% endtab %}

{% tab title="vol2" %}हैकिंग तकनीकों की एक किताब से निम्नलिखित सामग्री है। निम्नलिखित सामग्री फ़ाइल generic-methodologies-and-resources/basic-forensic-methodology/memory-dump-analysis/volatility-cheatsheet.md से है। अंग्रेजी में लिखे गए पाठ का हिंदी में अनुवाद करें और अनुवाद को वही मार्कडाउन और एचटीएमएल विन्यास बनाए रखें। कोड, हैकिंग तकनीक नाम, हैकिंग शब्द, क्लाउड/एसएएस प्लेटफ़ॉर्म नाम (जैसे Workspace, aws, gcp...), 'लीक', पेंटेस्टिंग और मार्कडाउन टैग्स जैसी चीजें अनुवाद न करें। अनुवाद के अलावा कोई अतिरिक्त सामग्री न जोड़ें।%}
```bash
volatility --profile=Win7SP1x86_23418 mutantscan -f file.dmp
volatility --profile=Win7SP1x86_23418 -f file.dmp handles -p <PID> -t mutant
```
### सिमलिंक्स

{% tabs %}
{% tab title="vol3" %}
```bash
./vol.py -f file.dmp windows.symlinkscan.SymlinkScan
```
{% endtab %}

{% tab title="vol2" %}वोलेटिलिटी चीटशीट

### वोलेटिलिटी चीटशीट

#### वोलेटिलिटी इंस्टॉलेशन

- वोलेटिलिटी इंस्टॉलेशन की जांच करें
  ```bash
  volatility -h
  ```

#### बेसिक कमांड्स

- प्रक्रियाओं की सूची देखें
  ```bash
  volatility -f <डंप_फ़ाइल> --profile=<प्रोफ़ाइल> pslist
  ```

- रजिस्ट्री कुंजी की सूची देखें
  ```bash
  volatility -f <डंप_फ़ाइल> --profile=<प्रोफ़ाइल> hivelist
  ```

- फ़ाइल्सिस्टम की जांच करें
  ```bash
  volatility -f <डंप_फ़ाइल> --profile=<प्रोफ़ाइल> filescan
  ```

#### डेटा एनालिसिस

- डेटा एनालिसिस के लिए विभिन्न प्लगइन्स का उपयोग करें
  ```bash
  volatility -f <डंप_फ़ाइल> --profile=<प्रोफ़ाइल> <प्लगइन>
  ```

#### नेटवर्क एनालिसिस

- नेटवर्क कनेक्शन्स की सूची देखें
  ```bash
  volatility -f <डंप_फ़ाइल> --profile=<प्रोफ़ाइल> connections
  ```

- नेटवर्क ट्रैफ़िक का विश्लेषण करें
  ```bash
  volatility -f <डंप_फ़ाइल> --profile=<प्रोफ़ाइल> connscan
  ```

#### डेटा रिकवरी

- डिलीटेड फ़ाइल्स की जाँच करें
  ```bash
  volatility -f <डंप_फ़ाइल> --profile=<प्रोफ़ाइल> filescan
  ```

- डेटा को रिकवर करने के लिए फाइल कार्विंग का उपयोग करें
  ```bash
  volatility -f <डंप_फ़ाइल> --profile=<प्रोफ़ाइल> photorec
  ```

#### अन्य उपयोगी कमांड्स

- डंप फ़ाइल की जानकारी प्राप्त करें
  ```bash
  volatility -f <डंप_फ़ाइल> imageinfo
  ```

- डंप फ़ाइल से प्रक्रियाओं की जानकारी प्राप्त करें
  ```bash
  volatility -f <डंप_फ़ाइल> --profile=<प्रोफ़ाइल> pstree
  ```

- डंप फ़ाइल से रजिस्ट्री की जानकारी प्राप्त करें
  ```bash
  volatility -f <डंप_फ़ाइल> --profile=<प्रोफ़ाइल> printkey -o <रजिस्ट्री_कुंजी>
  ```
{% endtab %}
```bash
volatility --profile=Win7SP1x86_23418 -f file.dmp symlinkscan
```
{% endtab %}
{% endtabs %}

### बैश

मेमोरी से बैश इतिहास पढ़ना संभव है। आप _.bash\_history_ फ़ाइल को भी डंप कर सकते हैं, लेकिन यदि यह अक्षम है तो आप इस volatility मॉड्यूल का उपयोग कर सकते हैं।
```
./vol.py -f file.dmp linux.bash.Bash
```
{% endtab %}

{% tab title="vol2" %}हैकर्स के लिए एक अच्छा उपकरण है वॉलेटिलिटी, जो रैम डंप विश्लेषण के लिए उपयुक्त है। यह एक उच्च स्तरीय रूप से मोड्यूलर और प्लग-इन आधारित फ्रेमवर्क है जो विभिन्न ऑपरेटिंग सिस्टम के लिए रैम डंप विश्लेषण करने की क्षमता प्रदान करता है। यह चीजें जैसे कि प्रक्रिया, नेटवर्क कनेक्शन्स, फाइल सिस्टम, रजिस्ट्री और अन्य विभिन्न विशेषताएं जांचने के लिए उपकरण प्रदान करता है। यह एक शक्तिशाली और लोकप्रिय उपकरण है जो डिजिटल जांच के लिए उपयुक्त है।{% endtab %}
```
volatility --profile=Win7SP1x86_23418 -f file.dmp linux_bash
```
### समयरेखा

{% tabs %}
{% tab title="vol3" %}
```bash
./vol.py -f file.dmp timeLiner.TimeLiner
```
{% endtab %}

{% tab title="vol2" %}हैकर, अनुवादक और लेखक हैं। वे सभी जानकारी को सुपर स्पष्ट और संक्षेप में लिखते हैं बिना किसी जानकारी को खोने के।{% endtab %}
```
volatility --profile=Win7SP1x86_23418 -f timeliner
```
{% endtab %}
{% endtabs %}

### ड्राइवर्स

{% tabs %}
{% tab title="vol3" %}
```
./vol.py -f file.dmp windows.driverscan.DriverScan
```
{% endtab %}

{% tab title="vol2" %}वोलेटिलिटी चीटशीट

### वोलेटिलिटी चीटशीट

#### वोलेटिलिटी टूल्स का उपयोग

- **डेटा कलेक्शन:** डेटा कलेक्शन के लिए वोलेटिलिटी टूल्स का उपयोग करें।
- **डेटा एनालिसिस:** डेटा एनालिसिस के लिए वोलेटिलिटी टूल्स का उपयोग करें।
- **डेटा रिकवरी:** डेटा रिकवरी के लिए वोलेटिलिटी टूल्स का उपयोग करें।

#### वोलेटिलिटी टूल्स की चीटशीट

- **वर्शन कमांड:** `volatility -v`
- **प्रोसेसेस कमांड:** `volatility --profile=ProfileName pslist`
- **नेटवर्क कनेक्शन्स कमांड:** `volatility --profile=ProfileName connections`
- **रजिस्ट्री कमांड:** `volatility --profile=ProfileName printkey -K "HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Run"`
- **डंप फाइल के लिए फाइल इन्फॉर्मेशन कमांड:** `volatility imageinfo -f /path/to/memorydump.dd`
- **करंट रनिंग प्रोसेस के लिए रिपोर्ट कमांड:** `volatility --profile=ProfileName pstree`
- **करंट रनिंग प्रोसेस के लिए रिपोर्ट कमांड:** `volatility --profile=ProfileName pstree`
- **करंट रनिंग प्रोसेस के लिए रिपोर्ट कमांड:** `volatility --profile=ProfileName pstree`
- **करंट रनिंग प्रोसेस के लिए रिपोर्ट कमांड:** `volatility --profile=ProfileName pstree`
- **करंट रनिंग प्रोसेस के लिए रिपोर्ट कमांड:** `volatility --profile=ProfileName pstree`
- **करंट रनिंग प्रोसेस के लिए रिपोर्ट कम
```bash
volatility --profile=Win7SP1x86_23418 -f file.dmp driverscan
```
### क्लिपबोर्ड प्राप्त करें
```bash
#Just vol2
volatility --profile=Win7SP1x86_23418 clipboard -f file.dmp
```
### IE इतिहास प्राप्त करें
```bash
#Just vol2
volatility --profile=Win7SP1x86_23418 iehistory -f file.dmp
```
### नोटपैड टेक्स्ट प्राप्त करें
```bash
#Just vol2
volatility --profile=Win7SP1x86_23418 notepad -f file.dmp
```
### स्क्रीनशॉट
```bash
#Just vol2
volatility --profile=Win7SP1x86_23418 screenshot -f file.dmp
```
### मास्टर बूट रिकॉर्ड (MBR)
```bash
volatility --profile=Win7SP1x86_23418 mbrparser -f file.dmp
```
**मास्टर बूट रिकॉर्ड (MBR)** एक स्टोरेज मीडियम के तार्किक विभाजनों का प्रबंधन करने में महत्वपूर्ण भूमिका निभाता है, जो विभिन्न [फ़ाइल सिस्टम](https://en.wikipedia.org/wiki/File_system) के साथ संरचित होते हैं। यह केवल विभाजन खाका जानकारी नहीं रखता है बल्कि एक बूट लोडर के रूप में कार्य करने वाले क्रियात्मक कोड भी शामिल होता है। यह बूट लोडर या तो सीधे ऑपरेटिंग सिस्टम की दूसरे स्टेज लोडिंग प्रक्रिया को प्रारंभ करता है (देखें [द्वितीय स्टेज बूट लोडर](https://en.wikipedia.org/wiki/Second-stage_boot_loader)) या प्रत्येक विभाजन के [वॉल्यूम बूट रिकॉर्ड](https://en.wikipedia.org/wiki/Volume_boot_record) (VBR) के साथ समन्वय में काम करता है। विस्तृत जानकारी के लिए, [MBR Wikipedia पृष्ठ](https://en.wikipedia.org/wiki/Master_boot_record) पर जाएं।

## संदर्भ
* [https://andreafortuna.org/2017/06/25/volatility-my-own-cheatsheet-part-1-image-identification/](https://andreafortuna.org/2017/06/25/volatility-my-own-cheatsheet-part-1-image-identification/)
* [https://scudette.blogspot.com/2012/11/finding-kernel-debugger-block.html](https://scudette.blogspot.com/2012/11/finding-kernel-debugger-block.html)
* [https://or10nlabs.tech/cgi-sys/suspendedpage.cgi](https://or10nlabs.tech/cgi-sys/suspendedpage.cgi)
* [https://www.aldeid.com/wiki/Windows-userassist-keys](https://www.aldeid.com/wiki/Windows-userassist-keys)
​* [https://learn.microsoft.com/en-us/windows/win32/fileio/master-file-table](https://learn.microsoft.com/en-us/windows/win32/fileio/master-file-table)
* [https://answers.microsoft.com/en-us/windows/forum/all/uefi-based-pc-protective-mbr-what-is-it/0fc7b558-d8d4-4a7d-bae2-395455bb19aa](https://answers.microsoft.com/en-us/windows/forum/all/uefi-based-pc-protective-mbr-what-is-it/0fc7b558-d8d4-4a7d-bae2-395455bb19aa)

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

[**RootedCON**](https://www.rootedcon.com/) **स्पेन** में सबसे महत्वपूर्ण साइबर सुरक्षा घटना है और **यूरोप** में सबसे महत्वपूर्ण में से एक है। **तकनीकी ज्ञान को बढ़ावा देने** के मिशन के साथ, यह कांग्रेस प्रौद्योगिकी और साइबर सुरक्षा विशेषज्ञों के लिए एक उफानता सम्मेलन है।

{% embed url="https://www.rootedcon.com/" %}

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी कंपनी का विज्ञापन HackTricks में देखना चाहते हैं या **HackTricks को PDF में डाउनलोड** करना चाहते हैं तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos को PRs सबमिट करके।

</details>
