# Volatility - चीटशीट

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करना चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **फॉलो** करें मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) को PR जमा करके।

</details>

​

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

​​[**RootedCON**](https://www.rootedcon.com/) स्पेन में सबसे महत्वपूर्ण साइबर सुरक्षा इवेंट है और यूरोप में सबसे महत्वपूर्ण माना जाता है। **तकनीकी ज्ञान को बढ़ावा देने** की मिशन के साथ, यह कांग्रेस प्रौद्योगिकी और साइबर सुरक्षा विशेषज्ञों के लिए एक उबलता हुआ मिलन स्थल है।

{% embed url="https://www.rootedcon.com/" %}

यदि आपको कुछ **तेज़ और पागल** चाहिए जो कई Volatility प्लगइन्स को पैरलेल पर लॉन्च करेगा, तो आप इस्तेमाल कर सकते हैं: [https://github.com/carlospolop/autoVolatility](https://github.com/carlospolop/autoVolatility)
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
### volatility2

{% tabs %}
{% tab title="Method1" %}
```
Download the executable from https://www.volatilityfoundation.org/26
```
{% endtab %}

{% tab title="Method 2" %}विधि 2
```bash
git clone https://github.com/volatilityfoundation/volatility.git
cd volatility
python setup.py install
```
{% endtab %}
{% endtabs %}

## Volatility Commands

आधिकारिक दस्तावेज़ में पहुंचें [Volatility command reference](https://github.com/volatilityfoundation/volatility/wiki/Command-Reference#kdbgscan)

### "सूची" और "स्कैन" प्लगइन पर एक नोट

Volatility के पास प्लगइनों के लिए दो मुख्य दृष्टिकोण हैं, जो कभी-कभी उनके नामों में प्रतिबिंबित होते हैं। "सूची" प्लगइन विंडोज़ कर्नल संरचनाओं के माध्यम से जानकारी प्राप्त करने का प्रयास करेंगे, जैसे कि प्रक्रियाओं को (मेमोरी में `_EPROCESS` संरचनाओं की लिंक सूची का पता लगाएं और चलते हुए), ओएस हैंडल (हैंडल टेबल का पता लगाना और सूचीबद्ध करना, पायंटर्स को डिरेफ़ेरेंस करना, आदि)। वे अधिकांश रूप से विंडोज़ API की तरह व्यवहार करते हैं जैसा कि अगर उदाहरण के लिए प्रक्रियाओं की सूची देखने के लिए अनुरोध किया जाए।

इसलिए, "सूची" प्लगइन बहुत तेज होते हैं, लेकिन विंडोज़ API के रूप में मैलवेयर द्वारा प्रबंधित किए जाने के लिए उत्पन्न होते हैं। उदाहरण के लिए, यदि मैलवेयर द्वारा DKOM का उपयोग करके प्रक्रिया को `_EPROCESS` लिंक सूची से अनलिंक करता है, तो यह टास्क प्रबंधक में दिखाई नहीं देगा और न ही यह pslist में दिखाई देगा।

दूसरी ओर, "स्कैन" प्लगइन एक ऐसे संरचनाओं को डिरेफ़ेरेंस करने पर स्पष्ट रूप से समझने के लिए मेमोरी को कार्व करने के लिए एक ऐसा दृष्टिकोण अपनाएंगे। उदाहरण के लिए, `psscan` मेमोरी को पढ़ेगा और इसे `_EPROCESS` ऑब्जेक्ट्स बनाने की कोशिश करेगा (यह पूल-टैग स्कैनिंग का उपयोग करता है, जो रुचि की संरचना की मौजूदगी की संकेत करने वाले 4-बाइट स्ट्रिंग्स की खोज करता है)। इसका फायदा यह है कि यह प्रक्रियाओं को खोज सकता है जो बाहर निकल गई हैं, और यदि मैलवेयर `_EPROCESS` लिंक सूची को बदलता है, तो प्लगइन फिर भी मेमोरी में मौजूद संरचना को खोजेगा (क्योंकि प्रक्रिया चलाने के लिए यह अभी भी मौजूद होना चाहिए)। इसका नुकसान यह है कि "स्कैन" प्लगइन "सूची" प्लगइनों से थोड़ा धीमे होते हैं, और कभी-कभी गलत पॉजिटिव्स दे सकते हैं (जो प्रक्रिया है जो बहुत पहले बाहर निकल गई है और उसकी संरचना के कुछ हिस्से अन्य कार्रवाईयों द्वारा अधिलिखित किए गए हैं)।

स्रोत: [http://tomchop.me/2016/11/21/tutorial-volatility-plugins-malware-analysis/](http://tomchop.me/2016/11/21/tutorial-volatility-plugins-malware-analysis/)

## ओएस प्रोफ़ाइल

### Volatility3

जैसा कि readme में समझाया गया है, आपको _volatility3/volatility/symbols_ में समर्थित ओएस के **सिम्बल टेबल** को रखने की आवश्यकता होती है।\
विभिन्न ऑपरेटिंग सिस्टम के लिए सिम्बल टेबल पैक डाउनलोड के लिए उपलब्ध हैं:

* [https://downloads.volatilityfoundation.org/volatility3/symbols/windows.zip](https://downloads.volatilityfoundation.org/volatility3/symbols/windows.zip)
* [https://downloads.volatilityfoundation.org/volatility3/symbols/mac.zip](https://downloads.volatilityfoundation.org/volatility3/symbols/mac.zip)
* [https://downloads.volatilityfoundation.org/volatility3/symbols/linux.zip](https://downloads.volatilityfoundation.org/volatility3/symbols/linux.zip)

### Volatility2

#### बाहरी प्रोफ़ाइल

आप समर्थित प्रोफ़ाइल की सूची प्राप्त कर सकते हैं:
```bash
./volatility_2.6_lin64_standalone --info | grep "Profile"
```
यदि आप एक **नया प्रोफ़ाइल जिसे आपने डाउनलोड किया है** (उदाहरण के लिए लिनक्स वाला) उपयोग करना चाहते हैं, तो आपको निम्नलिखित फ़ोल्डर संरचना बनानी होगी: _plugins/overlays/linux_ और इस फ़ोल्डर में प्रोफ़ाइल संबंधी ज़िप फ़ाइल डालें। फिर, निम्नलिखित कमांड का उपयोग करके प्रोफ़ाइलों की संख्या प्राप्त करें:
```bash
./vol --plugins=/home/kali/Desktop/ctfs/final/plugins --info
Volatility Foundation Volatility Framework 2.6


Profiles
--------
LinuxCentOS7_3_10_0-123_el7_x86_64_profilex64 - A Profile for Linux CentOS7_3.10.0-123.el7.x86_64_profile x64
VistaSP0x64                                   - A Profile for Windows Vista SP0 x64
VistaSP0x86                                   - A Profile for Windows Vista SP0 x86
```
आप [https://github.com/volatilityfoundation/profiles](https://github.com/volatilityfoundation/profiles) से Linux और Mac प्रोफ़ाइल डाउनलोड कर सकते हैं।

पिछले चंक में आप देख सकते हैं कि प्रोफ़ाइल का नाम `LinuxCentOS7_3_10_0-123_el7_x86_64_profilex64` है, और आप इसका उपयोग करके कुछ ऐसा कर सकते हैं:
```bash
./vol -f file.dmp --plugins=. --profile=LinuxCentOS7_3_10_0-123_el7_x86_64_profilex64 linux_netscan
```
#### प्रोफ़ाइल खोजें
```
volatility imageinfo -f file.dmp
volatility kdbgscan -f file.dmp
```
#### **imageinfo और kdbgscan के बीच का अंतर**

imageinfo केवल प्रोफ़ाइल सुझाव प्रदान करता है, जबकि **kdbgscan** सही प्रोफ़ाइल और सही KDBG पता (यदि कई हों) की पहचान करने के लिए डिज़ाइन किया गया है। यह प्लगइन Volatility प्रोफ़ाइल्स से जुड़े KDBGHeader हस्ताक्षरों के लिए स्कैन करता है और गलत पॉजिटिव को कम करने के लिए सेनिटी चेक लागू करता है। आउटपुट की व्याख्यानता और सेनिटी चेक की संख्या उस पर निर्भर करती है कि Volatility क्या एक DTB ढूंढ सकता है, इसलिए यदि आप पहले से ही सही प्रोफ़ाइल जानते हैं (या यदि आपको imageinfo से प्रोफ़ाइल सुझाव मिलता है), तो सुनिश्चित करें कि आप इसे उपयोग करते हैं (यहां से [यहां](https://www.andreafortuna.org/2017/06/25/volatility-my-own-cheatsheet-part-1-image-identification/) से)।

हमेशा **kdbgscan द्वारा पाए गए प्रक्रियाओं की संख्या पर एक नज़र डालें**। कभी-कभी imageinfo और kdbgscan एक से अधिक उपयुक्त प्रोफ़ाइल ढूंढ सकते हैं, लेकिन केवल **वैध एक में कुछ प्रक्रिया संबंधित** होगी (यह इसलिए है क्योंकि प्रक्रियाओं को निकालने के लिए सही KDBG पता की आवश्यकता होती है)।
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

**कर्नल डीबगर ब्लॉक** (जिसे Volatility द्वारा \_KDDEBUGGER\_DATA64 के प्रकार का KdDebuggerDataBlock या **KDBG** कहा जाता है) Volatility और डीबगर्स द्वारा कई चीजों के लिए महत्वपूर्ण है। उदाहरण के लिए, इसमें PsActiveProcessHead के संदर्भ होता है जो प्रक्रिया सूची के लिए सभी प्रक्रियाओं के सूची का मुख्य है।

## ओएस सूचना
```bash
#vol3 has a plugin to give OS information (note that imageinfo from vol2 will give you OS info)
./vol.py -f file.dmp windows.info.Info
```
प्लगइन `banners.Banners` का उपयोग **डंप में लिनक्स बैनर्स खोजने के लिए vol3 में किया जा सकता है।**

## हैश/पासवर्ड

[SAM हैश](../../../windows-hardening/stealing-credentials/credentials-protections.md#cached-credentials), [डोमेन कैश्ड क्रेडेंशियल](../../../windows-hardening/stealing-credentials/credentials-protections.md#cached-credentials) और [लेएसए सीक्रेट्स](../../../windows-hardening/authentication-credentials-uac-and-efs.md#lsa-secrets) को निकालें।

{% tabs %}
{% tab title="vol3" %}
```bash
./vol.py -f file.dmp windows.hashdump.Hashdump #Grab common windows hashes (SAM+SYSTEM)
./vol.py -f file.dmp windows.cachedump.Cachedump #Grab domain cache hashes inside the registry
./vol.py -f file.dmp windows.lsadump.Lsadump #Grab lsa secrets
```
## वोलेटिलिटी चीटशीट

### वोलेटिलिटी क्या है?

वोलेटिलिटी एक खुला स्रोत (open-source) रूपांतरण उपकरण है जिसे डिजिटल फोरेंसिक्स और मेमोरी फंडा विशेषज्ञों द्वारा उपयोग किया जाता है। यह उपकरण विंडोज, लिनक्स और मैक ऑपरेटिंग सिस्टम पर काम करता है और इन सिस्टमों के मेमोरी डंप को विश्लेषण करने की क्षमता प्रदान करता है।

### वोलेटिलिटी के उपयोग

वोलेटिलिटी का उपयोग डिजिटल फोरेंसिक्स और मेमोरी फंडा विशेषज्ञों द्वारा विभिन्न कार्यों के लिए किया जाता है, जैसे:

- अद्यतन और विश्लेषण के लिए विंडोज, लिनक्स और मैक मेमोरी डंप का उपयोग करना।
- प्रक्रिया और थ्रेड की जांच करने के लिए विंडोज, लिनक्स और मैक मेमोरी डंप का उपयोग करना।
- नेटवर्क विश्लेषण के लिए पैकेट कैप्चर फ़ाइल का उपयोग करना।
- रजिस्ट्री और फ़ाइल सिस्टम की जांच करने के लिए विंडोज मेमोरी डंप का उपयोग करना।

### वोलेटिलिटी के फायदे

वोलेटिलिटी के उपयोग से निम्नलिखित फायदे हो सकते हैं:

- विंडोज, लिनक्स और मैक मेमोरी डंप का विश्लेषण करने की क्षमता।
- प्रक्रिया और थ्रेड की जांच करने की क्षमता।
- नेटवर्क विश्लेषण के लिए पैकेट कैप्चर फ़ाइल का उपयोग करने की क्षमता।
- रजिस्ट्री और फ़ाइल सिस्टम की जांच करने की क्षमता।

### वोलेटिलिटी के उपकरण

वोलेटिलिटी के उपकरण निम्नलिखित हो सकते हैं:

- `volatility`: यह वोलेटिलिटी का मुख्य उपकरण है और विंडोज, लिनक्स और मैक मेमोरी डंप का विश्लेषण करने के लिए उपयोग किया जाता है।
- `volshell`: यह वोलेटिलिटी का एक अतिरिक्त उपकरण है जिसे इंटरैक्टिव मोड में उपयोग किया जा सकता है।
- `volshell`: यह वोलेटिलिटी का एक अतिरिक्त उपकरण है जिसे इंटरैक्टिव मोड में उपयोग किया जा सकता है।

### वोलेटिलिटी के बेसिक कमांड्स

वोलेटिलिटी के बेसिक कमांड्स निम्नलिखित हो सकते हैं:

- `imageinfo`: मेमोरी डंप की जानकारी प्रदान करता है।
- `pslist`: चल रही प्रक्रियाओं की सूची प्रदान करता है।
- `pstree`: प्रक्रिया वृक्ष प्रदान करता है।
- `dlllist`: प्रक्रियाओं के लिए लोड किए गए DLL की सूची प्रदान करता है।
- `handles`: प्रक्रियाओं के लिए हैंडल की सूची प्रदान करता है।
- `cmdline`: प्रक्रियाओं के लिए कमांड लाइन प्रदान करता है।
- `filescan`: फ़ाइलों की सूची प्रदान करता है।
- `netscan`: नेटवर्क कनेक्शन की सूची प्रदान करता है।
- `connections`: नेटवर्क कनेक्शन की सूची प्रदान करता है।
- `malfind`: संदेहास्पद या अवैध कार्यों की खोज करता है।
- `cmdscan`: कमांड इंजेक्शन की खोज करता है।
- `ssdt`: SSDT (System Service Descriptor Table) की सूची प्रदान करता है।
- `driverscan`: लोड किए गए ड्राइवरों की सूची प्रदान करता है।
- `modscan`: लोड किए गए मॉड्यूलों की सूची प्रदान करता है।
- `ssdt`: SSDT (System Service Descriptor Table) की सूची प्रदान करता है।
- `driverscan`: लोड किए गए ड्राइवरों की सूची प्रदान करता है।
- `modscan`: लोड किए गए मॉड्यूलों की सूची प्रदान करता है।

### वोलेटिलिटी के उपयोगी रेफरेंस

वोलेटिलिटी के उपयोगी रेफरेंस निम्नलिखित हो सकते हैं:

- [वोलेटिलिटी डॉक्यूमेंटेशन](https://github.com/volatilityfoundation/volatility/wiki)
- [वोलेटिलिटी चीटशीट](https://github.com/sans-dfir/sift-cheatsheet/blob/master/cheatsheets/Volatility%20Cheatsheet.pdf)
- [वोलेटिलिटी विश्लेषण के लिए ट्यूटोरियल](https://www.youtube.com/watch?v=VXJfXwWZBZQ)
```bash
volatility --profile=Win7SP1x86_23418 hashdump -f file.dmp #Grab common windows hashes (SAM+SYSTEM)
volatility --profile=Win7SP1x86_23418 cachedump -f file.dmp #Grab domain cache hashes inside the registry
volatility --profile=Win7SP1x86_23418 lsadump -f file.dmp #Grab lsa secrets
```
{% endtab %}
{% endtabs %}

## मेमोरी डंप

प्रक्रिया का मेमोरी डंप, प्रक्रिया की वर्तमान स्थिति का सब कुछ निकालेगा। **प्रोसडंप** मॉड्यूल केवल **कोड** को ही **निकालेगा**।
```
volatility -f file.dmp --profile=Win7SP1x86 memdump -p 2168 -D conhost/
```
<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

​​​[**RootedCON**](https://www.rootedcon.com/) स्पेन में सबसे महत्वपूर्ण साइबर सुरक्षा कार्यक्रम है और यूरोप में सबसे महत्वपूर्ण में से एक है। **तकनीकी ज्ञान को बढ़ावा देने** की मिशन के साथ, यह सम्मेलन प्रौद्योगिकी और साइबर सुरक्षा विशेषज्ञों के लिए एक उबलता हुआ मिलन स्थान है।

{% embed url="https://www.rootedcon.com/" %}

## प्रक्रियाएँ

### प्रक्रियाओं की सूची

**संदिग्ध** प्रक्रियाओं (नाम द्वारा) या **अप्रत्याशित** बच्ची **प्रक्रियाएं** (उदाहरण के लिए iexplorer.exe के बच्चे के रूप में cmd.exe) ढूंढ़ने का प्रयास करें।
यह देखने के लिए रोचक हो सकता है कि pslist के परिणाम को psscan के परिणाम के साथ तुलना करें छिपी हुई प्रक्रियाओं की पहचान करने के लिए।

{% tabs %}
{% tab title="vol3" %}
```bash
python3 vol.py -f file.dmp windows.pstree.PsTree # Get processes tree (not hidden)
python3 vol.py -f file.dmp windows.pslist.PsList # Get process list (EPROCESS)
python3 vol.py -f file.dmp windows.psscan.PsScan # Get hidden process list(malware)
```
## Volatility चीटशीट

### विशेषताएँ

- यह एक खुला स्रोत फ्रेमवर्क है जिसे डिजिटल फोरेंसिक्स और मेमोरी फंडा विश्लेषण के लिए उपयोग किया जा सकता है।
- यह विभिन्न ऑपरेटिंग सिस्टमों के लिए उपलब्ध है, जैसे Windows, Linux, macOS, Android, iOS आदि।
- यह विभिन्न फाइल सिस्टमों के साथ काम कर सकता है, जैसे NTFS, FAT, EXT, HFS +, APFS, आदि।
- यह विभिन्न प्रकार के मेमोरी डंप फ़ाइलों का समर्थन करता है, जैसे raw, crash, hibernation, VMware snapshot, आदि।

### उपयोगी आदेश

- `imageinfo`: इमेज की मेमोरी फ़ाइल के बारे में महत्वपूर्ण जानकारी प्रदान करता है।
- `pslist`: सिस्टम प्रोसेस की सूची प्रदान करता है।
- `pstree`: प्रोसेस के वृक्षाकार रूप में संगठित करता है।
- `dlllist`: प्रोसेस के साथ लोड किए गए DLL की सूची प्रदान करता है।
- `handles`: प्रोसेस के द्वारा उपयोग किए जाने वाले हैंडल की सूची प्रदान करता है।
- `filescan`: फ़ाइल सिस्टम में फ़ाइलों की खोज करता है।
- `cmdline`: प्रोसेस के लिए चल रहे आदेश प्रदान करता है।
- `malfind`: संदेहास्पद या अवैध कार्यों के लिए मेमोरी की खोज करता है।
- `dumpfiles`: फ़ाइलों को मेमोरी से निकालता है।
- `hashdump`: पासवर्ड हैश को निकालता है।
- `hivelist`: रजिस्ट्री हाइव की सूची प्रदान करता है।
- `hivedump`: रजिस्ट्री हाइव को निकालता है।

### उदाहरण

```plaintext
volatility -f memory.dmp imageinfo
volatility -f memory.dmp pslist
volatility -f memory.dmp pstree
volatility -f memory.dmp dlllist -p <PID>
volatility -f memory.dmp handles -p <PID>
volatility -f memory.dmp filescan
volatility -f memory.dmp cmdline -p <PID>
volatility -f memory.dmp malfind
volatility -f memory.dmp dumpfiles -Q <PID> -D <output_directory>
volatility -f memory.dmp hashdump -y <output_file>
volatility -f memory.dmp hivelist
volatility -f memory.dmp hivedump -o <offset> -y <output_file>
```

### उपयोगी संदर्भ

- [Volatility डॉक्यूमेंटेशन](https://github.com/volatilityfoundation/volatility/wiki)
- [Volatility गाइड](https://github.com/volatilityfoundation/volatility/wiki/Volatility-Usage)
```bash
volatility --profile=PROFILE pstree -f file.dmp # Get process tree (not hidden)
volatility --profile=PROFILE pslist -f file.dmp # Get process list (EPROCESS)
volatility --profile=PROFILE psscan -f file.dmp # Get hidden process list(malware)
volatility --profile=PROFILE psxview -f file.dmp # Get hidden process list
```
{% endtab %}
{% endtabs %}

### डंप प्रोसेस

{% tabs %}
{% tab title="vol3" %}
```bash
./vol.py -f file.dmp windows.dumpfiles.DumpFiles --pid <pid> #Dump the .exe and dlls of the process in the current directory
```
## Volatility चीटशीट

यहां आपको Volatility टूल के लिए एक चीटशीट मिलेगी। यह चीटशीट आपको विभिन्न मेमोरी डंप फ़ाइलों का विश्लेषण करने के लिए उपयोगी आदेशों की जानकारी प्रदान करेगी।

### विश्लेषण आदेश

#### जीनेरिक विश्लेषण

- `imageinfo` - इमेज की जानकारी प्रदान करता है
- `pslist` - प्रक्रियाओं की सूची प्रदान करता है
- `pstree` - प्रक्रिया पेड़ प्रदान करता है
- `psscan` - छिपी हुई प्रक्रियाएं खोजता है
- `dlllist` - प्रक्रियाओं के लिए DLL सूची प्रदान करता है
- `handles` - हैंडल सूची प्रदान करता है
- `filescan` - फ़ाइलों की सूची प्रदान करता है
- `cmdline` - प्रक्रियाओं की कमांड लाइन प्रदान करता है
- `consoles` - कंसोल सूची प्रदान करता है
- `vadinfo` - वर्चुअल एड्रेस स्पेस की जानकारी प्रदान करता है
- `vadtree` - वर्चुअल एड्रेस स्पेस का पेड़ प्रदान करता है
- `vaddump` - वर्चुअल एड्रेस स्पेस को डंप करता है
- `vadwalk` - वर्चुअल एड्रेस स्पेस की जांच करता है
- `vadtree` - वर्चुअल एड्रेस स्पेस का पेड़ प्रदान करता है
- `vaddump` - वर्चुअल एड्रेस स्पेस को डंप करता है
- `vadwalk` - वर्चुअल एड्रेस स्पेस की जांच करता है

#### नेटवर्क विश्लेषण

- `connections` - संबंधों की सूची प्रदान करता है
- `connscan` - खोज के लिए संबंधों की सूची प्रदान करता है
- `sockets` - सॉकेट सूची प्रदान करता है
- `sockscan` - खोज के लिए सॉकेट सूची प्रदान करता है
- `netscan` - नेटवर्क डेटा की खोज करता है

#### रजिस्ट्री विश्लेषण

- `hivelist` - रजिस्ट्री हाइव की सूची प्रदान करता है
- `printkey` - रजिस्ट्री कुंजी की जानकारी प्रदान करता है
- `printval` - रजिस्ट्री मान की जानकारी प्रदान करता है
- `hashdump` - रजिस्ट्री संकेतक की जानकारी प्रदान करता है

#### फ़ाइल विश्लेषण

- `malfind` - संदेहास्पद फ़ाइलों की खोज करता है
- `filescan` - फ़ाइलों की सूची प्रदान करता है
- `dumpfiles` - फ़ाइलों को डंप करता है
- `handles` - हैंडल सूची प्रदान करता है
- `cmdline` - प्रक्रियाओं की कमांड लाइन प्रदान करता है
- `consoles` - कंसोल सूची प्रदान करता है
- `vadinfo` - वर्चुअल एड्रेस स्पेस की जानकारी प्रदान करता है
- `vadtree` - वर्चुअल एड्रेस स्पेस का पेड़ प्रदान करता है
- `vaddump` - वर्चुअल एड्रेस स्पेस को डंप करता है
- `vadwalk` - वर्चुअल एड्रेस स्पेस की जांच करता है

#### अन्य उपयोगी आदेश

- `modscan` - लोड किए गए मॉड्यूलों की सूची प्रदान करता है
- `ldrmodules` - लोड किए गए मॉड्यूलों की सूची प्रदान करता है
- `ssdt` - SSDT की जानकारी प्रदान करता है
- `gdt` - GDT की जानकारी प्रदान करता है
- `idt` - IDT की जानकारी प्रदान करता है
- `callbacks` - कॉलबैक सूची प्रदान करता है
- `driverscan` - ड्राइवरों की सूची प्रदान करता है
- `svcscan` - सेवा सूची प्रदान करता है
- `devicetree` - डिवाइस पेड़ प्रदान करता है
- `devicetree` - डिवाइस पेड़ प्रदान करता है
- `devicetree` - डिवाइस पेड़ प्रदान करता है
- `devicetree` - डिवाइस पेड़ प्रदान करता है
- `devicetree` - डिवाइस पेड़ प्रदान करता है
- `devicetree` - डिवाइस पेड़ प्रदान करता है
- `devicetree` - डिवाइस पेड़ प्रदान करता है
- `devicetree` - डिवाइस पेड़ प्रदान करता है
- `devicetree` - डिवाइस पेड़ प्रदान करता है
- `devicetree` - डिवाइस पेड़ प्रदान करता है
- `devicetree` - डिवाइस पेड़ प्रदान करता है
- `devicetree` - डिवाइस पेड़ प्रदान करता है
- `devicetree` - डिवाइस पेड़ प्रदान करता है
- `devicetree` - डिवाइस पेड़ प्रदान करता है
- `devicetree` - डिवाइस पेड़ प्रदान करता है
- `devicetree` - डिवाइस पेड़ प्रदान करता है
- `devicetree` - डिवाइस पेड़ प्रदान करता है
- `devicetree` - डिवाइस पेड़ प्रदान करता है
- `devicetree` - डिवाइस पेड़ प्रदान करता है
- `devicetree` - डिवाइस पेड़ प्रदान करता है
- `devicetree` - डिवाइस पेड़ प्रदान करता है
- `devicetree` - डिवाइस पेड़ प्रदान करता है
- `devicetree` - डिवाइस पेड़ प्रदान करता है
- `devicetree` - डिवाइस पेड़ प्रदान करता है
- `devicetree` - डिवाइस पेड़ प्रदान करता है
- `devicetree` - डिवाइस पेड़ प्रदान करता है
- `devicetree` - डिवाइस पेड़ प्रदान करता है
- `devicetree` - डिवाइस पेड़ प्रदान करता है
- `devicetree` - डिवाइस पेड़ प्रदान करता है
- `devicetree` - डिवाइस पेड़ प्रदान करता है
- `devicetree` - डिवाइस पेड़ प्रदान करता है
- `devicetree` - डिवाइस पेड़ प्रदान करता है
- `devicetree` - डिवाइस पेड़ प्रदान करता है
- `devicetree` - डिवाइस पेड़ प्रदान करता है
- `devicetree` - डिवाइस पेड़ प्रदान करता है
- `devicetree` - डिवाइस पेड़ प्रदान करता है
- `devicetree` - डिवाइस पेड़ प्रदान करता है
- `devicetree` - डिवाइस पेड़ प्रदान करता है
- `devicetree` - डिवाइस पेड़ प्रदान करता है
- `devicetree` - डिवाइस पेड़ प्रदान करता है
- `devicetree` - डिवाइस पेड़ प्रदान करता है
- `devicetree` - डिवाइस पेड़ प्रदान करता है
- `devicetree` - डिवाइस पेड़ प्रदान करता है
- `devicetree` - डिवाइस पेड़ प्रदान करता है
- `devicetree` - डिवाइस पेड़ प्रदान करता है
- `devicetree` - डिवाइस पेड़ प्रदान करता है
- `devicetree` - डिवाइस पेड़ प्रदान करता है
- `devicetree` - डिवाइस पेड़ प्रदान करता है
- `devicetree` - डिवाइस पेड़ प्रदान करता है
- `devicetree` - डिवाइस पेड़ प्रदान करता है
- `devicetree` - डिवाइस पेड़ प्रदान करता है
- `devicetree` - डिवाइस पेड़ प्रदान करता है
- `devicetree` - डिवाइस पेड़ प्रदान करता है
- `devicetree` - डिवाइस पेड़ प्रदान करता है
- `devicetree` - डिवाइस पेड़ प्रदान करता है
- `devicetree` - डिवाइस पेड़ प्रदान करता
```bash
volatility --profile=Win7SP1x86_23418 procdump --pid=3152 -n --dump-dir=. -f file.dmp
```
{% endtab %}
{% endtabs %}

### कमांड लाइन

क्या कोई संदेहास्पद कार्रवाई की गई थी?
```bash
python3 vol.py -f file.dmp windows.cmdline.CmdLine #Display process command-line arguments
```
## वोलेटिलिटी चीटशीट

### वोलेटिलिटी क्या है?

वोलेटिलिटी एक खुला स्रोत रूपांतरण टूल है जिसे डिजिटल फोरेंसिक्स और मेमोरी फोरेंसिक्स के लिए उपयोग किया जाता है। यह विंडोज, लिनक्स और मैक ओएस पर काम करता है और विभिन्न प्रकार के मेमोरी डंप फ़ाइलों का विश्लेषण करने की क्षमता प्रदान करता है।

### वोलेटिलिटी के लाभ

- विभिन्न प्रकार के मेमोरी डंप फ़ाइलों का विश्लेषण करने की क्षमता
- प्रक्रिया और सेवा की जांच करने के लिए विभिन्न टेक्निकल डेटा का प्राप्त करना
- रजिस्ट्री और फ़ाइल सिस्टम की जांच करने के लिए विभिन्न टेक्निकल डेटा का प्राप्त करना
- नेटवर्क और वेब ट्रैफ़िक की जांच करने के लिए विभिन्न टेक्निकल डेटा का प्राप्त करना
- विभिन्न टेक्निकल डेटा का प्राप्त करके अद्यतन और नई जानकारी प्राप्त करना

### वोलेटिलिटी का उपयोग करने के लिए आवश्यकताएं

- Python 2.7 या 3.4+ (वोलेटिलिटी 3 के लिए)
- वोलेटिलिटी इंस्टॉलेशन

### वोलेटिलिटी कमांड्स

यहां कुछ वोलेटिलिटी कमांड्स हैं जो आपके लिए उपयोगी हो सकते हैं:

- **imageinfo**: डंप फ़ाइल की मेमोरी इमेज की जानकारी प्रदान करता है।
- **pslist**: सिस्टम प्रक्रियाओं की सूची प्रदान करता है।
- **pstree**: प्रक्रिया पेड़ की जानकारी प्रदान करता है।
- **dlllist**: प्रक्रियाओं के लिए लोड किए गए DLL की सूची प्रदान करता है।
- **handles**: प्रक्रियाओं के लिए हैंडल की सूची प्रदान करता है।
- **filescan**: फ़ाइल सिस्टम में फ़ाइलों की खोज करता है।
- **netscan**: नेटवर्क विन्यास की जांच करता है।
- **connscan**: नेटवर्क कनेक्शन की जांच करता है।
- **cmdline**: प्रक्रियाओं के लिए कमांड लाइन आउटपुट प्रदान करता है।
- **malfind**: संदिग्ध मेमोरी रीज़र्वेशन और अनुप्रयोगों की खोज करता है।
- **dumpfiles**: फ़ाइल सिस्टम से फ़ाइलों की नकल बनाता है।
- **hivelist**: रजिस्ट्री हाइव की सूची प्रदान करता है।
- **printkey**: रजिस्ट्री कुंजी की जानकारी प्रदान करता है।
- **hashdump**: रजिस्ट्री समेत संबंधित खातों के हैश डंप प्रदान करता है।
- **mftparser**: एमएफटी (Master File Table) की जांच करता है।
- **iehistory**: इंटरनेट एक्सप्लोरर इतिहास की जांच करता है।
- **svcscan**: सेवा की जांच करता है।
- **userassist**: उपयोगकर्ता सहायता की जांच करता है।
- **cmdscan**: कमांड इतिहास की जांच करता है।
- **mbrparser**: मास्टर बूट रिकॉर्ड (MBR) की जांच करता है।
- **ssdt**: SSDT (System Service Descriptor Table) की जांच करता है।
- **gdt**: GDT (Global Descriptor Table) की जांच करता है।
- **ldrmodules**: लोडर मॉड्यूल की सूची प्रदान करता है।
- **ldrmodules**: लोडर मॉड्यूल की सूची प्रदान करता है।
- **modscan**: लोडर मॉड्यूल की जांच करता है।
- **ssdt**: SSDT (System Service Descriptor Table) की जांच करता है।
- **gdt**: GDT (Global Descriptor Table) की जांच करता है।
- **ldrmodules**: लोडर मॉड्यूल की सूची प्रदान करता है।
- **ldrmodules**: लोडर मॉड्यूल की सूची प्रदान करता है।
- **modscan**: लोडर मॉड्यूल की जांच करता है।

### वोलेटिलिटी उपयोग करने की विधि

1. वोलेटिलिटी को डाउनलोड करें और इंस्टॉल करें।
2. वोलेटिलिटी कमांड प्राप्त करें और उपयोग करें।
3. विश्लेषण के लिए वोलेटिलिटी कमांड का उपयोग करें।

### वोलेटिलिटी संसाधन

- [वोलेटिलिटी वेबसाइट](https://www.volatilityfoundation.org/)
- [वोलेटिलिटी गिटहब रेपो](https://github.com/volatilityfoundation/volatility)
```bash
volatility --profile=PROFILE cmdline -f file.dmp #Display process command-line arguments
volatility --profile=PROFILE consoles -f file.dmp #command history by scanning for _CONSOLE_INFORMATION
```
{% endtab %}
{% endtabs %}

cmd.exe में दर्ज किए गए कमांड conhost.exe द्वारा प्रसंस्कृत किए जाते हैं (Windows 7 से पहले csrss.exe द्वारा)। इसलिए, यदि किसी हमलावर ने cmd.exe को हमें एक मेमोरी डंप प्राप्त करने से पहले मार दिया है, तो अभी भी conhost.exe की मेमोरी से कमांड लाइन सत्र का इतिहास पुनर्प्राप्त करने की अच्छी संभावना है। यदि आप कुछ अजीब चीज़ (कंसोल के मॉड्यूल का उपयोग करके) पाते हैं, तो conhost.exe से संबंधित प्रक्रिया की मेमोरी को डंप करने और उसमें स्ट्रिंग्स की खोज करने के लिए प्रयास करें।

### पर्यावरण

प्रत्येक चल रही प्रक्रिया के एनवायरनमेंट वेरिएबल प्राप्त करें। कुछ दिलचस्प मान्यांकन हो सकते हैं।

{% tabs %}
{% tab title="vol3" %}
```bash
python3 vol.py -f file.dmp windows.envars.Envars [--pid <pid>] #Display process environment variables
```
## वोलेटिलिटी चीटशीट

### वोलेटिलिटी क्या है?

वोलेटिलिटी एक खुला स्रोत (open-source) रूपांतरण उपकरण है जिसे डिजिटल फोरेंसिक्स और मेमोरी फंडा विशेषज्ञों द्वारा उपयोग किया जाता है। यह उपकरण विंडोज, लिनक्स और मैक ऑपरेटिंग सिस्टम पर काम करता है और इन सिस्टमों के मेमोरी डंप को विश्लेषण करने की क्षमता प्रदान करता है।

### वोलेटिलिटी के उपयोग

वोलेटिलिटी का उपयोग डिजिटल फोरेंसिक्स और मेमोरी फंडा विशेषज्ञों द्वारा विभिन्न कार्यों के लिए किया जाता है, जैसे:

- मालवेयर विश्लेषण
- रूटकिट विश्लेषण
- नेटवर्क विश्लेषण
- प्रक्रिया विश्लेषण
- रजिस्ट्री विश्लेषण
- फ़ाइल सिस्टम विश्लेषण
- डिजिटल फोरेंसिक्स अभ्यास

### वोलेटिलिटी कमांड्स

यहां कुछ महत्वपूर्ण वोलेटिलिटी कमांड्स हैं:

- `imageinfo`: मेमोरी डंप की जानकारी प्रदान करता है।
- `pslist`: चल रही प्रक्रियाओं की सूची प्रदान करता है।
- `pstree`: प्रक्रिया वृक्ष प्रदान करता है।
- `dlllist`: प्रक्रियाओं के लिए लोड किए गए DLL की सूची प्रदान करता है।
- `handles`: हैंडल की सूची प्रदान करता है।
- `filescan`: फ़ाइलों के लिए स्कैन करता है।
- `cmdline`: प्रक्रियाओं की कमांड लाइन प्रदान करता है।
- `vadinfo`: वर्चुअल एड्रेस स्पेस की जानकारी प्रदान करता है।

### वोलेटिलिटी प्लगइन्स

वोलेटिलिटी के कई प्लगइन्स उपलब्ध हैं जो विशेष विश्लेषण कार्यों के लिए उपयोगी हो सकते हैं। कुछ महत्वपूर्ण प्लगइन्स निम्नलिखित हैं:

- `malfind`: मालवेयर के लिए अद्यतनित रेखाएं प्रदान करता है।
- `svcscan`: सेवा की सूची प्रदान करता है।
- `connscan`: नेटवर्क कनेक्शन की सूची प्रदान करता है।
- `modscan`: लोड किए गए मॉड्यूल की सूची प्रदान करता है।
- `ssdt`: SSDT (System Service Descriptor Table) की जानकारी प्रदान करता है।
- `apihooks`: API हुक की सूची प्रदान करता है।
- `ldrmodules`: लोड किए गए मॉड्यूल की सूची प्रदान करता है।

### वोलेटिलिटी इंस्टॉलेशन

वोलेटिलिटी को इंस्टॉल करने के लिए निम्नलिखित चरणों का पालन करें:

1. पायथन (Python) इंस्टॉल करें।
2. पायथन पैकेज प्रबंधक (Python Package Manager) को इंस्टॉल करें।
3. वोलेटिलिटी को डाउनलोड करें और इंस्टॉल करें।
4. वोलेटिलिटी के लिए प्लगइन्स डाउनलोड करें और इंस्टॉल करें।

### वोलेटिलिटी उपयोग करना

वोलेटिलिटी का उपयोग करने के लिए निम्नलिखित चरणों का पालन करें:

1. टर्मिनल खोलें और वोलेटिलिटी को चलाने के लिए निम्नलिखित कमांड दर्ज करें:

```bash
volatility
```

2. वोलेटिलिटी कमांड प्राप्त करने के लिए `--help` विकल्प का उपयोग करें:

```bash
volatility --help
```

3. वोलेटिलिटी के उपयोग के लिए उपयुक्त कमांड दर्ज करें, जैसे:

```bash
volatility imageinfo -f memory.dmp
```

### वोलेटिलिटी संसाधन

यहां कुछ महत्वपूर्ण वोलेटिलिटी संसाधन हैं:

- [वोलेटिलिटी वेबसाइट](https://www.volatilityfoundation.org/)
- [वोलेटिलिटी डॉक्यूमेंटेशन](https://github.com/volatilityfoundation/volatility/wiki)
- [वोलेटिलिटी प्लगइन्स](https://github.com/volatilityfoundation/community/tree/master/plugins)
- [वोलेटिलिटी फोरम](https://forum.volatilityfoundation.org/)

### अभिप्रेत विधि

वोलेटिलिटी एक शक्तिशाली डिजिटल फोरेंसिक्स उपकरण है जो मेमोरी डंप विश्लेषण के लिए उपयोगी हो सकता है। इस चीटशीट में हमने वोलेटिलिटी के बारे में महत्वपूर्ण जानकारी, कमांड्स, प्लगइन्स और संसाधनों को समझाया है। यदि आप डिजिटल फोरेंसिक्स या मेमोरी फंडा में रुचि रखते हैं, तो वोलेटिलिटी आपके लिए एक महत्वपूर्ण उपकरण हो सकता है।
```bash
volatility --profile=PROFILE envars -f file.dmp [--pid <pid>] #Display process environment variables

volatility --profile=PROFILE -f file.dmp linux_psenv [-p <pid>] #Get env of process. runlevel var means the runlevel where the proc is initated
```
{% endtab %}
{% endtabs %}

### टोकन प्रिविलेज

अप्रत्याशित सेवाओं में टोकन प्रिविलेज की जांच करें।
कुछ प्रिविलेज्ड टोकन का उपयोग करने वाले प्रक्रियाओं की सूची बनाना दिलचस्प हो सकता है।

{% tabs %}
{% tab title="vol3" %}
```bash
#Get enabled privileges of some processes
python3 vol.py -f file.dmp windows.privileges.Privs [--pid <pid>]
#Get all processes with interesting privileges
python3 vol.py -f file.dmp windows.privileges.Privs | grep "SeImpersonatePrivilege\|SeAssignPrimaryPrivilege\|SeTcbPrivilege\|SeBackupPrivilege\|SeRestorePrivilege\|SeCreateTokenPrivilege\|SeLoadDriverPrivilege\|SeTakeOwnershipPrivilege\|SeDebugPrivilege"
```
## Volatility चीटशीट

यहां आपको Volatility टूल के लिए एक चीटशीट मिलेगी। यह चीटशीट आपको विभिन्न मेमोरी डंप फ़ाइलों का विश्लेषण करने के लिए उपयोगी आदेशों की जानकारी प्रदान करेगी।

### विश्लेषण आदेश

#### जीवंत प्रक्रियाओं की सूची

```plaintext
volatility -f <फ़ाइल> --profile=<प्रोफ़ाइल> pslist
```

#### डंप फ़ाइल के लिए प्रोफ़ाइल खोजें

```plaintext
volatility -f <फ़ाइल> imageinfo
```

#### रजिस्ट्री कुंजी की सूची

```plaintext
volatility -f <फ़ाइल> --profile=<प्रोफ़ाइल> hivelist
```

#### रजिस्ट्री कुंजी का विश्लेषण करें

```plaintext
volatility -f <फ़ाइल> --profile=<प्रोफ़ाइल> printkey -K <कुंजी>
```

#### फ़ाइल का विश्लेषण करें

```plaintext
volatility -f <फ़ाइल> --profile=<प्रोफ़ाइल> filescan
```

#### नेटवर्क कनेक्शन की सूची

```plaintext
volatility -f <फ़ाइल> --profile=<प्रोफ़ाइल> connscan
```

#### नेटवर्क ट्रैफ़िक का विश्लेषण करें

```plaintext
volatility -f <फ़ाइल> --profile=<प्रोफ़ाइल> netscan
```

#### फ़ाइल के लिए विश्लेषण चालू करें

```plaintext
volatility -f <फ़ाइल> --profile=<प्रोफ़ाइल> malfind -D <डायरेक्टरी>
```

#### विश्लेषण के लिए विंडोज प्रक्रियाओं की सूची

```plaintext
volatility -f <फ़ाइल> --profile=<प्रोफ़ाइल> psxview
```

#### विंडोज प्रक्रिया का विश्लेषण करें

```plaintext
volatility -f <फ़ाइल> --profile=<प्रोफ़ाइल> pstree -p <प्रक्रिया ID>
```

#### विंडोज सेवा की सूची

```plaintext
volatility -f <फ़ाइल> --profile=<प्रोफ़ाइल> svcscan
```

#### विंडोज सेवा का विश्लेषण करें

```plaintext
volatility -f <फ़ाइल> --profile=<प्रोफ़ाइल> svcscan -s <सेवा नाम>
```

#### विंडोज टास्क की सूची

```plaintext
volatility -f <फ़ाइल> --profile=<प्रोफ़ाइल> psscan
```

#### विंडोज टास्क का विश्लेषण करें

```plaintext
volatility -f <फ़ाइल> --profile=<प्रोफ़ाइल> psscan -p <प्रक्रिया ID>
```

#### विंडोज ड्राइव की सूची

```plaintext
volatility -f <फ़ाइल> --profile=<प्रोफ़ाइल> drvlist
```

#### विंडोज ड्राइव का विश्लेषण करें

```plaintext
volatility -f <फ़ाइल> --profile=<प्रोफ़ाइल> drvobj -D <ड्राइव>
```

#### विंडोज ड्राइव के लिए फ़ाइल की सूची

```plaintext
volatility -f <फ़ाइल> --profile=<प्रोफ़ाइल> filescan -D <ड्राइव>
```

#### विंडोज ड्राइव के लिए फ़ाइल का विश्लेषण करें

```plaintext
volatility -f <फ़ाइल> --profile=<प्रोफ़ाइल> filescan -D <ड्राइव> -F <फ़ाइल>
```

#### विंडोज ड्राइव के लिए फ़ाइल का विश्लेषण करें (फ़ाइल नाम के साथ)

```plaintext
volatility -f <फ़ाइल> --profile=<प्रोफ़ाइल> filescan -D <ड्राइव> -N <फ़ाइल नाम>
```

#### विंडोज ड्राइव के लिए फ़ाइल का विश्लेषण करें (फ़ाइल पथ के साथ)

```plaintext
volatility -f <फ़ाइल> --profile=<प्रोफ़ाइल> filescan -D <ड्राइव> -P <फ़ाइल पथ>
```

#### विंडोज ड्राइव के लिए फ़ाइल का विश्लेषण करें (फ़ाइल विवरण के साथ)

```plaintext
volatility -f <फ़ाइल> --profile=<प्रोफ़ाइल> filescan -D <ड्राइव> -S <फ़ाइल विवरण>
```

#### विंडोज ड्राइव के लिए फ़ाइल का विश्लेषण करें (फ़ाइल विवरण और फ़ाइल पथ के साथ)

```plaintext
volatility -f <फ़ाइल> --profile=<प्रोफ़ाइल> filescan -D <ड्राइव> -S <फ़ाइल विवरण> -P <फ़ाइल पथ>
```

#### विंडोज ड्राइव के लिए फ़ाइल का विश्लेषण करें (फ़ाइल विवरण और फ़ाइल नाम के साथ)

```plaintext
volatility -f <फ़ाइल> --profile=<प्रोफ़ाइल> filescan -D <ड्राइव> -S <फ़ाइल विवरण> -N <फ़ाइल नाम>
```

#### विंडोज ड्राइव के लिए फ़ाइल का विश्लेषण करें (फ़ाइल नाम और फ़ाइल पथ के साथ)

```plaintext
volatility -f <फ़ाइल> --profile=<प्रोफ़ाइल> filescan -D <ड्राइव> -N <फ़ाइल नाम> -P <फ़ाइल पथ>
```

#### विंडोज ड्राइव के लिए फ़ाइल का विश्लेषण करें (फ़ाइल नाम, फ़ाइल पथ और फ़ाइल विवरण के साथ)

```plaintext
volatility -f <फ़ाइल> --profile=<प्रोफ़ाइल> filescan -D <ड्राइव> -N <फ़ाइल नाम> -P <फ़ाइल पथ> -S <फ़ाइल विवरण>
```

#### विंडोज ड्राइव के लिए फ़ाइल का विश्लेषण करें (फ़ाइल नाम, फ़ाइल पथ, फ़ाइल विवरण और फ़ाइल आयाम के साथ)

```plaintext
volatility -f <फ़ाइल> --profile=<प्रोफ़ाइल> filescan -D <ड्राइव> -N <फ़ाइल नाम> -P <फ़ाइल पथ> -S <फ़ाइल विवरण> -A <फ़ाइल आयाम>
```

#### विंडोज ड्राइव के लिए फ़ाइल का विश्लेषण करें (फ़ाइल नाम, फ़ाइल पथ, फ़ाइल विवरण, फ़ाइल आयाम और फ़ाइल तिथि के साथ)

```plaintext
volatility -f <फ़ाइल> --profile=<प्रोफ़ाइल> filescan -D <ड्राइव> -N <फ़ाइल नाम> -P <फ़ाइल पथ> -S <फ़ाइल विवरण> -A <फ़ाइल आयाम> -T <फ़ाइल तिथि>
```

#### विंडोज ड्राइव के लिए फ़ाइल का विश्लेषण करें (फ़ाइल नाम, फ़ाइल पथ, फ़ाइल विवरण, फ़ाइल आयाम, फ़ाइल तिथि और फ़ाइल समय के साथ)

```plaintext
volatility -f <फ़ाइल> --profile=<प्रोफ़ाइल> filescan -D <ड्राइव> -N <फ़ाइल नाम> -P <फ़ाइल पथ> -S <फ़ाइल विवरण> -A <फ़ाइल आयाम> -T <फ़ाइल तिथि> -M <फ़ाइल समय>
```

#### विंडोज ड्राइव के लिए फ़ाइल का विश्लेषण करें (फ़ाइल नाम, फ़ाइल पथ, फ़ाइल विवरण, फ़ाइल आयाम, फ़ाइल तिथि, फ़ाइल समय और फ़ाइल अनुमति के साथ)

```plaintext
volatility -f <फ़ाइल> --profile=<प्रोफ़ाइल> filescan -D <ड्राइव> -N <फ़ाइल नाम> -P <फ़ाइल पथ> -S <फ़ाइल विवरण> -A <फ़ाइल आयाम> -T <फ़ाइल तिथि> -M <फ़ाइल समय> -R <फ़ाइल अनुमति>
```

#### विंडोज ड्राइव के लिए फ़ाइल का विश्लेषण करें (फ़ाइल नाम, फ़ाइल पथ, फ़ाइल विवरण, फ़ाइल आयाम, फ़ाइल तिथि, फ़ाइल समय, फ़ाइल अनुमति और फ़ाइल आयाम के साथ)

```plaintext
volatility -f <फ़ाइल> --profile=<प्रोफ़ाइल> filescan -D <ड्राइव> -N <फ़ाइल
```bash
#Get enabled privileges of some processes
volatility --profile=Win7SP1x86_23418 privs --pid=3152 -f file.dmp | grep Enabled
#Get all processes with interesting privileges
volatility --profile=Win7SP1x86_23418 privs -f file.dmp | grep "SeImpersonatePrivilege\|SeAssignPrimaryPrivilege\|SeTcbPrivilege\|SeBackupPrivilege\|SeRestorePrivilege\|SeCreateTokenPrivilege\|SeLoadDriverPrivilege\|SeTakeOwnershipPrivilege\|SeDebugPrivilege"
```
{% endtab %}
{% endtabs %}

### SIDs

प्रक्रिया द्वारा स्वामित्व में आने वाले प्रत्येक SSID की जांच करें।
यह दिलचस्प हो सकता है कि कौन सी प्रक्रियाएं एक विशेषाधिकार SID का उपयोग कर रही हैं (और कौन सी सेवा SID का उपयोग कर रही हैं)।

{% tabs %}
{% tab title="vol3" %}
```bash
./vol.py -f file.dmp windows.getsids.GetSIDs [--pid <pid>] #Get SIDs of processes
./vol.py -f file.dmp windows.getservicesids.GetServiceSIDs #Get the SID of services
```
## वोलेटिलिटी चीटशीट

यह चीटशीट वोलेटिलिटी टूल के लिए एक संक्षेप मार्गदर्शिका है। यह आपको वोलेटिलिटी टूल के साथ विभिन्न मेमोरी डंप विश्लेषण तकनीकों की एक सारगर्भित सूची प्रदान करेगी।

### वोलेटिलिटी टूल का उपयोग करना

वोलेटिलिटी टूल को चलाने के लिए निम्नलिखित चरणों का पालन करें:

1. वोलेटिलिटी टूल को डाउनलोड करें और स्थापित करें।
2. टर्मिनल खोलें और वोलेटिलिटी टूल के इंजन को चलाने के लिए निम्नलिखित कमांड का उपयोग करें:

```bash
volatility
```

### वोलेटिलिटी चीटशीट

यहां वोलेटिलिटी टूल के कुछ महत्वपूर्ण कमांडों की सूची है:

#### जीनेरिक कमांड्स

- **imageinfo**: मेमोरी डंप की जानकारी प्रदान करता है।
- **pslist**: सिस्टम प्रोसेस की सूची प्रदान करता है।
- **pstree**: प्रोसेस के वृक्षाकार आकार की सूची प्रदान करता है।
- **dlllist**: प्रोसेस के संलग्न लाइब्रेरी की सूची प्रदान करता है।
- **handles**: प्रोसेस के हैंडल की सूची प्रदान करता है।
- **cmdline**: प्रोसेस के कमांड लाइन आदेश की सूची प्रदान करता है।
- **filescan**: फ़ाइल सिस्टम की स्कैनिंग करता है और खुले फ़ाइलों की सूची प्रदान करता है।
- **netscan**: नेटवर्क की स्कैनिंग करता है और खुले कनेक्शनों की सूची प्रदान करता है।
- **connections**: प्रोसेस के नेटवर्क कनेक्शन की सूची प्रदान करता है।
- **malfind**: संदिग्ध मेमोरी क्षेत्रों की खोज करता है।
- **dumpfiles**: फ़ाइलों को डंप करता है जो खुले हैंडल या नेटवर्क कनेक्शन के साथ संबंधित हैं।
- **cmdscan**: प्रोसेस के कमांड लाइन आदेश की सूची प्रदान करता है।
- **consoles**: प्रोसेस के कंसोल की सूची प्रदान करता है।
- **vadinfo**: प्रोसेस के वर्चुअल एड्रेस स्पेस की सूची प्रदान करता है।
- **vaddump**: वर्चुअल एड्रेस स्पेस को डंप करता है।
- **vadtree**: वर्चुअल एड्रेस स्पेस के वृक्षाकार आकार की सूची प्रदान करता है।
- **vadwalk**: वर्चुअल एड्रेस स्पेस की वॉकिंग करता है।
- **vadtree**: वर्चुअल एड्रेस स्पेस के वृक्षाकार आकार की सूची प्रदान करता है।
- **vadwalk**: वर्चुअल एड्रेस स्पेस की वॉकिंग करता है।
- **vadtree**: वर्चुअल एड्रेस स्पेस के वृक्षाकार आकार की सूची प्रदान करता है।
- **vadwalk**: वर्चुअल एड्रेस स्पेस की वॉकिंग करता है।
- **vadtree**: वर्चुअल एड्रेस स्पेस के वृक्षाकार आकार की सूची प्रदान करता है।
- **vadwalk**: वर्चुअल एड्रेस स्पेस की वॉकिंग करता है।
- **vadtree**: वर्चुअल एड्रेस स्पेस के वृक्षाकार आकार की सूची प्रदान करता है।
- **vadwalk**: वर्चुअल एड्रेस स्पेस की वॉकिंग करता है।
- **vadtree**: वर्चुअल एड्रेस स्पेस के वृक्षाकार आकार की सूची प्रदान करता है।
- **vadwalk**: वर्चुअल एड्रेस स्पेस की वॉकिंग करता है।
- **vadtree**: वर्चुअल एड्रेस स्पेस के वृक्षाकार आकार की सूची प्रदान करता है।
- **vadwalk**: वर्चुअल एड्रेस स्पेस की वॉकिंग करता है।
- **vadtree**: वर्चुअल एड्रेस स्पेस के वृक्षाकार आकार की सूची प्रदान करता है।
- **vadwalk**: वर्चुअल एड्रेस स्पेस की वॉकिंग करता है।
- **vadtree**: वर्चुअल एड्रेस स्पेस के वृक्षाकार आकार की सूची प्रदान करता है।
- **vadwalk**: वर्चुअल एड्रेस स्पेस की वॉकिंग करता है।
- **vadtree**: वर्चुअल एड्रेस स्पेस के वृक्षाकार आकार की सूची प्रदान करता है।
- **vadwalk**: वर्चुअल एड्रेस स्पेस की वॉकिंग करता है।
- **vadtree**: वर्चुअल एड्रेस स्पेस के वृक्षाकार आकार की सूची प्रदान करता है।
- **vadwalk**: वर्चुअल एड्रेस स्पेस की वॉकिंग करता है।
- **vadtree**: वर्चुअल एड्रेस स्पेस के वृक्षाकार आकार की सूची प्रदान करता है।
- **vadwalk**: वर्चुअल एड्रेस स्पेस की वॉकिंग करता है।
- **vadtree**: वर्चुअल एड्रेस स्पेस के वृक्षाकार आकार की सूची प्रदान करता है।
- **vadwalk**: वर्चुअल एड्रेस स्पेस की वॉकिंग करता है।
- **vadtree**: वर्चुअल एड्रेस स्पेस के वृक्षाकार आकार की सूची प्रदान करता है।
- **vadwalk**: वर्चुअल एड्रेस स्पेस की वॉकिंग करता है।
- **vadtree**: वर्चुअल एड्रेस स्पेस के वृक्षाकार आकार की सूची प्रदान करता है।
- **vadwalk**: वर्चुअल एड्रेस स्पेस की वॉकिंग करता है।
- **vadtree**: वर्चुअल एड्रेस स्पेस के वृक्षाकार आकार की सूची प्रदान करता है।
- **vadwalk**: वर्चुअल एड्रेस स्पेस की वॉकिंग करता है।
- **vadtree**: वर्चुअल एड्रेस स्पेस के वृक्षाकार आकार की सूची प्रदान करता है।
- **vadwalk**: वर्चुअल एड्रेस स्पेस की वॉकिंग करता है।
- **vadtree**: वर्चुअल एड्रेस स्पेस के वृक्षाकार आकार की सूची प्रदान करता है।
- **vadwalk**: वर्चुअल एड्रेस स्पेस की वॉकिंग करता है।
- **vadtree**: वर्चुअल एड्रेस स्पेस के वृक्षाकार आकार की सूची प्रदान करता है।
- **vadwalk**: वर्चुअल एड्रेस स्पेस की वॉकिंग करता है।
- **vadtree**: वर्चुअल एड्रेस स्पेस के वृक्षाकार आकार की सूची प्रदान करता है।
- **vadwalk**: वर्चुअल एड्रेस स्पेस की वॉकिंग करता है।
- **vadtree**: वर्चुअल एड्रेस स्पेस के वृक्षाकार आकार की सूची प्रदान करता है।
- **vadwalk**: वर्चुअल एड्रेस स्पेस की वॉकिंग करता है।
- **vadtree**: वर्चुअल एड्रेस स्पेस के वृक्षाकार आकार की सूची प्रदान करता है।
- **vadwalk**: वर्चुअल एड्रेस स्पेस की वॉकिंग करता है।
```bash
volatility --profile=Win7SP1x86_23418 getsids -f file.dmp #Get the SID owned by each process
volatility --profile=Win7SP1x86_23418 getservicesids -f file.dmp #Get the SID of each service
```
{% endtab %}
{% endtabs %}

### हैंडल्स

इस्तेमाली जानने के लिए कि किस अन्य फ़ाइल, कुंजी, थ्रेड, प्रक्रिया... के लिए एक प्रक्रिया के पास एक हैंडल है (खोला हुआ है)
```bash
vol.py -f file.dmp windows.handles.Handles [--pid <pid>]
```
## वोलेटिलिटी चीटशीट

### वोलेटिलिटी क्या है?

वोलेटिलिटी एक ओपन सोर्स रूपांतरण और विश्लेषण उपकरण है जिसे डिजिटल अभियांत्रिकी और फोरेंसिक्स के लिए उपयोग किया जाता है। यह रनटाइम डेटा को विश्लेषण करने के लिए डिजाइन किया गया है और विभिन्न ऑपरेटिंग सिस्टम पर काम करता है। वोलेटिलिटी के माध्यम से, आप मेमोरी डंप को विश्लेषण कर सकते हैं और इससे अद्यतित जानकारी प्राप्त कर सकते हैं जो आपको अभियांत्रिकी और फोरेंसिक्स कार्यों में मदद कर सकती है।

### वोलेटिलिटी के लिए चीटशीट

यहां वोलेटिलिटी के लिए कुछ महत्वपूर्ण और उपयोगी कमांड्स हैं:

#### बेसिक कमांड्स

- `imageinfo`: इमेज की जानकारी प्रदान करता है।
- `pslist`: सिस्टम प्रोसेस की सूची प्रदान करता है।
- `pstree`: प्रोसेस के वृक्षाकार रूपांतरण को प्रदान करता है।
- `dlllist`: प्रोसेस के लिए लोड किए गए DLL की सूची प्रदान करता है।
- `handles`: प्रोसेस के लिए हैंडल की सूची प्रदान करता है।
- `cmdline`: प्रोसेस के लिए कमांड लाइन विवरण प्रदान करता है।
- `filescan`: फ़ाइल सिस्टम की स्कैनिंग करता है और खुली हुई फ़ाइलों की सूची प्रदान करता है।
- `netscan`: नेटवर्क की स्कैनिंग करता है और खुली हुई कनेक्शनों की सूची प्रदान करता है।
- `connections`: नेटवर्क कनेक्शन की सूची प्रदान करता है।
- `svcscan`: सेवा की सूची प्रदान करता है।
- `malfind`: संदेहास्पद या अवैध गतिविधि के संकेतों की खोज करता है।
- `ssdt`: SSDT (System Service Descriptor Table) की सूची प्रदान करता है।
- `gdt`: GDT (Global Descriptor Table) की सूची प्रदान करता है।
- `idt`: IDT (Interrupt Descriptor Table) की सूची प्रदान करता है।
- `ldrmodules`: लोड किए गए मॉड्यूल की सूची प्रदान करता है।
- `modscan`: मॉड्यूल की सूची प्रदान करता है।
- `ss`: स्क्रीनशॉट लेता है।
- `memdump`: प्रोसेस की मेमोरी डंप बनाता है।

#### विशेष कमांड्स

- `malfind`: संदेहास्पद या अवैध गतिविधि के संकेतों की खोज करता है।
- `malfind`: संदेहास्पद या अवैध गतिविधि के संकेतों की खोज करता है।
- `malfind`: संदेहास्पद या अवैध गतिविधि के संकेतों की खोज करता है।

#### डेटा विश्लेषण

- `strings`: मेमोरी डंप से स्ट्रिंग्स को खोजता है।
- `hashdump`: पासवर्ड हैश की सूची प्रदान करता है।
- `hivelist`: रजिस्ट्री हाइव की सूची प्रदान करता है।
- `hivedump`: रजिस्ट्री हाइव को डंप करता है।
- `printkey`: रजिस्ट्री की कुंजी की सूची प्रदान करता है।
- `dumpkey`: रजिस्ट्री की कुंजी को डंप करता है।
- `cmdscan`: कमांड इतिहास की सूची प्रदान करता है।
- `consoles`: कंसोल की सूची प्रदान करता है।
- `clipboard`: क्लिपबोर्ड की सूची प्रदान करता है।
- `shellbags`: शैलबैग्स की सूची प्रदान करता है।
- `mftparser`: MFT (Master File Table) को पार्स करता है।
- `usnparser`: USN (Update Sequence Number) को पार्स करता है।
- `eventhooks`: इवेंट हुक की सूची प्रदान करता है।
- `timeliner`: टाइमलाइन रिपोर्ट बनाता है।

#### नेटवर्क विश्लेषण

- `connscan`: कनेक्शन की सूची प्रदान करता है।
- `sockets`: सॉकेट की सूची प्रदान करता है।
- `sockscan`: सॉकेट की सूची प्रदान करता है।
- `netscan`: नेटवर्क की स्कैनिंग करता है और खुली हुई कनेक्शनों की सूची प्रदान करता है।
- `connections`: नेटवर्क कनेक्शन की सूची प्रदान करता है।
- `connscan`: कनेक्शन की सूची प्रदान करता है।
- `sockets`: सॉकेट की सूची प्रदान करता है।
- `sockscan`: सॉकेट की सूची प्रदान करता है।

#### अन्य उपयोगी कमांड्स

- `malfind`: संदेहास्पद या अवैध गतिविधि के संकेतों की खोज करता है।
- `malfind`: संदेहास्पद या अवैध गतिविधि के संकेतों की खोज करता है।
- `malfind`: संदेहास्पद या अवैध गतिविधि के संकेतों की खोज करता है।

यहां आपको वोलेटिलिटी के कुछ महत्वपूर्ण कमांड्स की एक सारणी दी गई है:

| कमांड | विवरण |
| --- | --- |
| `imageinfo` | इमेज की जानकारी प्रदान करता है। |
| `pslist` | सिस्टम प्रोसेस की सूची प्रदान करता है। |
| `pstree` | प्रोसेस के वृक्षाकार रूपांतरण को प्रदान करता है। |
| `dlllist` | प्रोसेस के लिए लोड किए गए DLL की सूची प्रदान करता है। |
| `handles` | प्रोसेस के लिए हैंडल की सूची प्रदान करता है। |
| `cmdline` | प्रोसेस के लिए कमांड लाइन विवरण प्रदान करता है। |
| `filescan` | फ़ाइल सिस्टम की स्कैनिंग करता है और खुली हुई फ़ाइलों की सूची प्रदान करता है। |
| `netscan` | नेटवर्क की स्कैनिंग करता है और खुली हुई कनेक्शनों की सूची प्रदान करता है। |
| `connections` | नेटवर्क कनेक्शन की सूची प्रदान करता है। |
| `svcscan` | सेवा की सूची प्रदान करता है। |
| `malfind` | संदेहास्पद या अवैध गतिविधि के संकेतों की खोज करता है। |
| `ssdt` | SSDT (System Service Descriptor Table) की सूची प्रदान करता है। |
| `gdt` | GDT (Global Descriptor Table) की सूची प्रदान करता है। |
| `idt` | IDT (Interrupt Descriptor Table) की सूची प्रदान करता है। |
| `ldrmodules` | लोड किए गए मॉड्यूल की सूची प्रदान करता है। |
| `modscan` | मॉड्यूल की सूची प्रदान करता है। |
| `ss` | स्क्रीनशॉट लेता है। |
| `memdump` | प्रोसेस की मेमोरी डंप बनाता है। |

यहां वोलेटिलिटी के कुछ विशेष कमांड्स की एक सारणी दी गई है:

| कमांड | विवरण |
| --- | --- |
| `malfind` | संदेहास्पद या अवैध गतिविधि के संकेतों की खोज करता है। |
| `malfind` | संदेहास्पद या अवैध गतिविधि के संकेतों की खोज करता है। |
| `malfind` | संदेहास्पद या अवैध गतिविधि के संकेतों क
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
## वोलेटिलिटी चीटशीट

### वोलेटिलिटी क्या है?

वोलेटिलिटी एक खुला स्रोत (open-source) रूपांतरण उपकरण है जिसे डिजिटल फोरेंसिक्स और मेमोरी फंडा विशेषज्ञों द्वारा उपयोग किया जाता है। यह उपकरण विंडोज, लिनक्स और मैक ऑपरेटिंग सिस्टम पर काम करता है और विभिन्न प्रकार के मेमोरी डंप (memory dump) फ़ाइलों का विश्लेषण करने की क्षमता प्रदान करता है।

### वोलेटिलिटी के उपयोग

वोलेटिलिटी का उपयोग डिजिटल फोरेंसिक्स, मेमोरी फंडा विशेषज्ञता, मालवेयर विश्लेषण और संगठनों के साइबर सुरक्षा टीमों द्वारा किया जाता है। यह उपकरण विभिन्न तकनीकी जानकारी को प्राप्त करने के लिए उपयोगी होता है, जैसे प्रक्रिया के दौरान चल रहे प्रोसेस, लोड किए गए मॉड्यूल, रजिस्ट्री एंट्री, नेटवर्क कनेक्शन और खुले फ़ाइलों की सूची।

### वोलेटिलिटी के फीचर्स

वोलेटिलिटी के कुछ मुख्य फीचर्स निम्नलिखित हैं:

- विभिन्न ऑपरेटिंग सिस्टमों का समर्थन करता है, जैसे विंडोज, लिनक्स और मैक।
- विभिन्न प्रकार के मेमोरी डंप फ़ाइलों का समर्थन करता है, जैसे कर्नल और फ़ुल मेमोरी डंप।
- विभिन्न प्रकार के डेटा और संरचनाओं का विश्लेषण करता है, जैसे प्रोसेस, थ्रेड्स, मॉड्यूल्स, रजिस्ट्री, नेटवर्क कनेक्शन, फ़ाइलों की सूची और बहुत कुछ।
- विभिन्न प्लगइन्स का समर्थन करता है, जो अतिरिक्त फंडा विश्लेषण क्षमताओं को प्रदान करते हैं।

### वोलेटिलिटी कमांड्स

यहां कुछ उपयोगी वोलेटिलिटी कमांड्स हैं:

- `imageinfo`: मेमोरी डंप फ़ाइल की मेटाडेटा जानकारी प्रदान करता है।
- `pslist`: चल रहे प्रोसेस की सूची प्रदान करता है।
- `pstree`: प्रोसेस के वृक्षीय रूपांतरण को प्रदर्शित करता है।
- `dlllist`: लोड किए गए मॉड्यूल की सूची प्रदान करता है।
- `handles`: हैंडल्स की सूची प्रदान करता है।
- `connections`: नेटवर्क कनेक्शन की सूची प्रदान करता है।
- `filescan`: खुले फ़ाइलों की सूची प्रदान करता है।

### वोलेटिलिटी प्लगइन्स

यहां कुछ उपयोगी वोलेटिलिटी प्लगइन्स हैं:

- `malfind`: मालवेयर के लिए संकेतों की खोज करता है।
- `timeliner`: विभिन्न घटनाओं को समय के आधार पर प्रदर्शित करता है।
- `vadtree`: वॉल्यूम एट्रिब्यूट डेटा (VAD) के वृक्षीय रूपांतरण को प्रदर्शित करता है।
- `svcscan`: सेवा की सूची प्रदान करता है।
- `ssdt`: SSDT (System Service Descriptor Table) की सूची प्रदान करता है।
- `gdt`: GDT (Global Descriptor Table) की सूची प्रदान करता है।

### वोलेटिलिटी इंस्टॉलेशन

वोलेटिलिटी को इंस्टॉल करने के लिए निम्नलिखित चरणों का पालन करें:

1. पायथन (Python) इंस्टॉल करें।
2. पायथन पैकेज प्रबंधक (Python Package Manager) को इंस्टॉल करें।
3. वोलेटिलिटी को डाउनलोड करें और इंस्टॉल करें।
4. वोलेटिलिटी को वॉकिंग डायरेक्टरी में जोड़ें।

### वोलेटिलिटी उपयोग करना

वोलेटिलिटी का उपयोग करने के लिए निम्नलिखित चरणों का पालन करें:

1. टर्मिनल खोलें और वोलेटिलिटी को चलाने के लिए निम्नलिखित कमांड दर्ज करें:

```bash
volatility
```

2. वोलेटिलिटी कमांड प्रारंभ हो जाएगा। अब आप विभिन्न कमांड्स का उपयोग कर सकते हैं।

### वोलेटिलिटी संसाधन

यहां कुछ उपयोगी वोलेटिलिटी संसाधन हैं:

- [वोलेटिलिटी ऑफिशियल वेबसाइट](https://www.volatilityfoundation.org/)
- [वोलेटिलिटी गिटहब रेपो](https://github.com/volatilityfoundation/volatility)
- [वोलेटिलिटी डॉक्यूमेंटेशन](https://github.com/volatilityfoundation/volatility/wiki)
```bash
volatility --profile=Win7SP1x86_23418 dlllist --pid=3152 -f file.dmp #Get dlls of a proc
volatility --profile=Win7SP1x86_23418 dlldump --pid=3152 --dump-dir=. -f file.dmp #Dump dlls of a proc
```
{% endtab %}
{% endtabs %}

### प्रक्रियाओं के लिए स्ट्रिंग्स

Volatility हमें यह जांचने की अनुमति देता है कि एक स्ट्रिंग किस प्रक्रिया का हिस्सा है।

{% tabs %}
{% tab title="vol3" %}
```bash
strings file.dmp > /tmp/strings.txt
./vol.py -f /tmp/file.dmp windows.strings.Strings --strings-file /tmp/strings.txt
```
## वोलेटिलिटी चीटशीट

यह चीटशीट वोलेटिलिटी टूल के लिए एक संक्षेप मार्गदर्शिका है। यह आपको वोलेटिलिटी टूल के साथ विभिन्न मेमोरी डंप विश्लेषण तकनीकों की एक सारगर्भित सूची प्रदान करेगी।

### वोलेटिलिटी टूल का उपयोग करना

1. वोलेटिलिटी टूल को डाउनलोड करें और स्थापित करें।
2. टर्मिनल खोलें और वोलेटिलिटी टूल के निर्देशानुसार उपयोग करें।

### वोलेटिलिटी टूल की मुख्य आदेश

- `imageinfo`: इमेज की जानकारी प्रदान करता है।
- `pslist`: प्रक्रियाओं की सूची प्रदान करता है।
- `pstree`: प्रक्रियाओं का पेड़ दर्शाता है।
- `dlllist`: प्रक्रियाओं के लिए लोड किए गए DLL प्रदान करता है।
- `handles`: हैंडल्स की सूची प्रदान करता है।
- `cmdline`: प्रक्रिया के लिए आपूर्ति लाइन प्रदान करता है।
- `consoles`: कंसोल की सूची प्रदान करता है।
- `filescan`: फ़ाइलों की सूची प्रदान करता है।
- `malfind`: संदेहास्पद विचारों की खोज करता है।
- `svcscan`: सेवा की सूची प्रदान करता है।
- `connections`: संबंधों की सूची प्रदान करता है।
- `sockets`: सॉकेट की सूची प्रदान करता है।
- `netscan`: नेटवर्क की सूची प्रदान करता है।
- `privs`: उपयोगकर्ता की विशेषाधिकार सूची प्रदान करता है।
- `cmdscan`: कमांड लाइन की सूची प्रदान करता है।
- `driverirp`: ड्राइवर के IRP संदेश प्रदान करता है।
- `ssdt`: SSDT की सूची प्रदान करता है।
- `gdt`: GDT की सूची प्रदान करता है।
- `idt`: IDT की सूची प्रदान करता है।
- `ldrmodules`: लोड किए गए मॉड्यूलों की सूची प्रदान करता है।
- `modscan`: मॉड्यूलों की सूची प्रदान करता है।
- `ss`: स्थिति की सूची प्रदान करता है।
- `callbacks`: कॉलबैक की सूची प्रदान करता है।
- `devicetree`: डिवाइस ट्री की सूची प्रदान करता है।
- `hivelist`: हाइव की सूची प्रदान करता है।
- `hivedump`: हाइव को डंप करता है।
- `hashdump`: हैश डंप करता है।
- `userassist`: उपयोगकर्ता सहायता की सूची प्रदान करता है।
- `shellbags`: शैलबैग की सूची प्रदान करता है।
- `mftparser`: MFT को पार्स करता है।
- `mftparser -D`: MFT को पार्स करता है और विवरण प्रदान करता है।
- `mftparser -F`: MFT को पार्स करता है और फ़ाइलों की सूची प्रदान करता है।
- `mftparser -i`: MFT को पार्स करता है और इंडेक्स की सूची प्रदान करता है।
- `mftparser -r`: MFT को पार्स करता है और रेकॉर्ड की सूची प्रदान करता है।
- `mftparser -a`: MFT को पार्स करता है और अद्यतन की सूची प्रदान करता है।
- `mftparser -s`: MFT को पार्स करता है और संकेतक की सूची प्रदान करता है।
- `mftparser -c`: MFT को पार्स करता है और बचाव की सूची प्रदान करता है।
- `mftparser -p`: MFT को पार्स करता है और प्रोपर्टी की सूची प्रदान करता है।
- `mftparser -x`: MFT को पार्स करता है और एक्स्ट्रा की सूची प्रदान करता है।
- `mftparser -l`: MFT को पार्स करता है और लिंक की सूची प्रदान करता है।
- `mftparser -u`: MFT को पार्स करता है और उपयोगकर्ता की सूची प्रदान करता है।
- `mftparser -n`: MFT को पार्स करता है और नेम की सूची प्रदान करता है।
- `mftparser -t`: MFT को पार्स करता है और समय की सूची प्रदान करता है।
- `mftparser -w`: MFT को पार्स करता है और विंडोज की सूची प्रदान करता है।
- `mftparser -e`: MFT को पार्स करता है और ईवेंट की सूची प्रदान करता है।
- `mftparser -m`: MFT को पार्स करता है और माउंट की सूची प्रदान करता है।
- `mftparser -g`: MFT को पार्स करता है और ग्रुप की सूची प्रदान करता है।
- `mftparser -b`: MFT को पार्स करता है और बूट की सूची प्रदान करता है।
- `mftparser -d`: MFT को पार्स करता है और डिरेक्टरी की सूची प्रदान करता है।
- `mftparser -y`: MFT को पार्स करता है और सिंबोलिक की सूची प्रदान करता है।
- `mftparser -k`: MFT को पार्स करता है और ट्रंक की सूची प्रदान करता है।
- `mftparser -v`: MFT को पार्स करता है और वॉल्यूम की सूची प्रदान करता है।
- `mftparser -q`: MFT को पार्स करता है और विशेषता की सूची प्रदान करता है।
- `mftparser -z`: MFT को पार्स करता है और ज़ोन की सूची प्रदान करता है।
- `mftparser -o`: MFT को पार्स करता है और ऑब्जेक्ट की सूची प्रदान करता है।
- `mftparser -j`: MFT को पार्स करता है और जंक की सूची प्रदान करता है।
- `mftparser -h`: MFT को पार्स करता है और हैश की सूची प्रदान करता है।
- `mftparser -y`: MFT को पार्स करता है और सिंबोलिक की सूची प्रदान करता है।
- `mftparser -k`: MFT को पार्स करता है और ट्रंक की सूची प्रदान करता है।
- `mftparser -v`: MFT को पार्स करता है और वॉल्यूम की सूची प्रदान करता है।
- `mftparser -q`: MFT को पार्स करता है और विशेषता की सूची प्रदान करता है।
- `mftparser -z`: MFT को पार्स करता है और ज़ोन की सूची प्रदान करता है।
- `mftparser -o`: MFT को पार्स करता है और ऑब्जेक्ट की सूची प्रदान करता है।
- `mftparser -j`: MFT को पार्स करता है और जंक की सूची प्रदान करता है।
- `mftparser -h`: MFT को पार्स करता है और हैश की सूची प्रदान करता है।
- `mftparser -y`: MFT को पार्स करता है और सिंबोलिक की सूची प्रदान करता है।
- `mftparser -k`: MFT को पार्स करता है और ट्रंक की सूची प्रदान करता है।
- `mftparser -v`: MFT को पार्स करता है और वॉल्यूम की सूची प्रदान करता है।
- `mftparser -q`: MFT को पार्स करता है और विशेषता की सूची प्रदान करता है।
- `mftparser -z`: MFT को पार्स करता है और ज़ोन की सूची प्रदान करता है।
- `mftparser -o`: MFT को पार्स करता है और ऑब्जेक्ट की सूची प्रदान करता है।
- `mftparser -j`: MFT को पार्स करता है और
```bash
strings file.dmp > /tmp/strings.txt
volatility -f /tmp/file.dmp windows.strings.Strings --string-file /tmp/strings.txt

volatility -f /tmp/file.dmp --profile=Win81U1x64 memdump -p 3532 --dump-dir .
strings 3532.dmp > strings_file
```
{% endtab %}
{% endtabs %}

यह यूजरलैंड विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका के रूप में विशेषता तालिका क
```bash
./vol.py -f file.dmp windows.vadyarascan.VadYaraScan --yara-rules "https://" --pid 3692 3840 3976 3312 3084 2784
./vol.py -f file.dmp yarascan.YaraScan --yara-rules "https://"
```
## वोलेटिलिटी चीटशीट

### वोलेटिलिटी क्या है?

वोलेटिलिटी एक खुला स्रोत (open-source) रूपांतरण उपकरण है जिसे डिजिटल फोरेंसिक्स और मेमोरी फंडा विशेषज्ञों द्वारा उपयोग किया जाता है। यह उपकरण विंडोज, लिनक्स और मैक ऑपरेटिंग सिस्टम पर काम करता है और इन सिस्टमों के मेमोरी डंप को विश्लेषण करने की क्षमता प्रदान करता है।

### वोलेटिलिटी के उपयोग

वोलेटिलिटी का उपयोग डिजिटल फोरेंसिक्स और मेमोरी फंडा विशेषज्ञों द्वारा विभिन्न कार्यों के लिए किया जाता है, जैसे:

- अद्यतन और विश्लेषण के लिए विंडोज, लिनक्स और मैक मेमोरी डंप का उपयोग करना।
- प्रक्रिया और थ्रेड की जांच करने के लिए विंडोज, लिनक्स और मैक मेमोरी डंप का उपयोग करना।
- नेटवर्क विश्लेषण के लिए प्रोटोकॉल, कनेक्शन और सत्र का उपयोग करना।
- रजिस्ट्री और फ़ाइल सिस्टम के विश्लेषण के लिए विंडोज मेमोरी डंप का उपयोग करना।

### वोलेटिलिटी कमांड्स

यहां कुछ वोलेटिलिटी कमांड्स हैं जो आपको विभिन्न कार्यों के लिए उपयोगी साबित हो सकते हैं:

- `imageinfo`: मेमोरी डंप की जानकारी प्रदान करता है।
- `pslist`: चल रही प्रक्रियाओं की सूची प्रदान करता है।
- `pstree`: प्रक्रिया वृक्ष प्रदान करता है।
- `dlllist`: प्रक्रियाओं के लिए लोड किए गए DLL की सूची प्रदान करता है।
- `handles`: प्रक्रियाओं के लिए हैंडल की सूची प्रदान करता है।
- `filescan`: फ़ाइल सिस्टम में फ़ाइलों की खोज करता है।
- `netscan`: नेटवर्क कनेक्शन की सूची प्रदान करता है।

### वोलेटिलिटी इंस्टॉल करना

वोलेटिलिटी को इंस्टॉल करने के लिए निम्नलिखित चरणों का पालन करें:

1. वोलेटिलिटी को डाउनलोड करें और इंस्टॉल करें।
2. वोलेटिलिटी को पथ में जोड़ें।
3. वोलेटिलिटी की संस्करण की जांच करें।

### वोलेटिलिटी उपयोग करना

वोलेटिलिटी का उपयोग करने के लिए निम्नलिखित चरणों का पालन करें:

1. वोलेटिलिटी कमांड लाइन टूल को चलाएं।
2. उपयुक्त कमांड का उपयोग करें।
3. परिणामों को विश्लेषण करें और उचित कार्रवाई करें।

### वोलेटिलिटी संसाधन

यहां कुछ वोलेटिलिटी संसाधन हैं जो आपके लिए उपयोगी साबित हो सकते हैं:

- [वोलेटिलिटी डॉक्यूमेंटेशन](https://www.volatilityfoundation.org/): वोलेटिलिटी के लिए आधिकारिक डॉक्यूमेंटेशन।
- [वोलेटिलिटी गिटहब रेपो](https://github.com/volatilityfoundation/volatility): वोलेटिलिटी का गिटहब रेपो।
- [वोलेटिलिटी फोरम](https://forum.volatilityfoundation.org/): वोलेटिलिटी संबंधित प्रश्नों और उत्तरों के लिए एक समुदाय।

### वोलेटिलिटी टिप्स

यहां कुछ वोलेटिलिटी टिप्स हैं जो आपके लिए उपयोगी साबित हो सकते हैं:

- वोलेटिलिटी कमांड्स का उपयोग करने से पहले वोलेटिलिटी की संस्करण की जांच करें।
- वोलेटिलिटी कमांड्स के लिए उपयुक्त प्राथमिक और वैकल्पिक चयन का उपयोग करें।
- वोलेटिलिटी कमांड्स के लिए उपयुक्त फ़िल्टर और ऑप्शन का उपयोग करें।
- वोलेटिलिटी कमांड्स के लिए उपयुक्त आउटपुट फॉर्मेट का उपयोग करें।

### वोलेटिलिटी उपयोगी स्रोत

यहां कुछ वोलेटिलिटी उपयोगी स्रोत हैं जो आपके लिए उपयोगी साबित हो सकते हैं:

- [वोलेटिलिटी चीटशीट](https://github.com/sans-dfir/sift-cheatsheet/blob/master/cheatsheets/Volatility%20Cheatsheet.pdf): वोलेटिलिटी के लिए एक चीटशीट।
- [वोलेटिलिटी ट्विटर हैंडल](https://twitter.com/volatility): वोलेटिलिटी के नवीनतम अपडेट के लिए ट्विटर हैंडल।
- [वोलेटिलिटी यूट्यूब चैनल](https://www.youtube.com/channel/UCj9yvBAXAg8gd2qxKgXgqxA): वोलेटिलिटी संबंधित वीडियो ट्यूटोरियल्स के लिए एक यूट्यूब चैनल।

### वोलेटिलिटी उपयोगी टूल्स

यहां कुछ वोलेटिलिटी उपयोगी टूल्स हैं जो आपके लिए उपयोगी साबित हो सकते हैं:

- [VolUtility](https://github.com/kevthehermit/VolUtility): वोलेटिलिटी के लिए एक ग्राफिकल उपयोगकर्ता इंटरफ़ेस (GUI) टूल।
- [VolDiff](https://github.com/kevthehermit/VolDiff): वोलेटिलिटी मेमोरी डंप के बीच अंतरों का खोजने के लिए एक टूल।
- [Volatility-Plugins](https://github.com/volatilityfoundation/community): वोलेटिलिटी के लिए विभिन्न प्लगइन्स का संग्रह।

### वोलेटिलिटी उपयोगी वेबसाइट

यहां कुछ वोलेटिलिटी उपयोगी वेबसाइट हैं जो आपके लिए उपयोगी साबित हो सकते हैं:

- [वोलेटिलिटी फोरेंसिक्स विकी](https://github.com/volatilityfoundation/volatility/wiki): वोलेटिलिटी के लिए एक विकीपीडिया संसाधन।
- [वोलेटिलिटी ब्लॉग](https://volatility-labs.blogspot.com/): वोलेटिलिटी संबंधित लेखों के लिए एक ब्लॉग।
- [वोलेटिलिटी फोरेंसिक्स ट्विटर हैंडल](https://twitter.com/volatility): वोलेटिलिटी फोरेंसिक्स के नवीनतम अपडेट के लिए ट्विटर हैंडल।

### वोलेटिलिटी उपयोगी बुक्स

यहां कुछ वोलेटिलिटी उपयोगी बुक्स हैं जो आपके लिए उपयोगी साबित हो सकते हैं:

- [वोलेटिलिटी फोरेंसिक्स कुकबुक](https://www.amazon.com/Volatility-Forensics-Cookbook-Michael-Hale/dp/1785281102): वोलेटिलिटी फोरेंसिक्स के लिए एक कुकबुक।
- [वोलेटिलिटी फोरेंसिक्स कुकबुक 2](https://www.amazon.com/Volatility-Forensics-Cookbook-Michael-Hale/dp/1789134503): वोलेटिलिटी फोरेंसिक्स के लिए एक और कुकबुक।
- [वोलेटिलिटी फोरेंसिक्स कुकबुक 3](https://www.amazon.com/Volatility-Forensics-Cookbook-Michael-Hale/dp/1838643576): वोलेटिलिटी फोरेंसिक्स के लिए एक और कुकबुक।

### वोलेटिलिटी उपयोगी वीडियो

यहां कुछ वोलेटिलिटी उपयोगी वीडियो हैं जो आपके लिए उपय
```bash
volatility --profile=Win7SP1x86_23418 yarascan -Y "https://" -p 3692,3840,3976,3312,3084,2784
```
{% endtab %}
{% endtabs %}

### UserAssist

**Windows** सिस्टम रजिस्ट्री डेटाबेस में एक सेट के रूप में **कुंजी** रखते हैं (**UserAssist कुंजी**) जिनका उपयोग प्रोग्रामों के उपयोग को ट्रैक करने के लिए किया जाता है। इन **कुंजियों** में उपलब्ध होती हैं प्रयोगों की संख्या और अंतिम प्रयोग की तारीख और समय।
```bash
./vol.py -f file.dmp windows.registry.userassist.UserAssist
```
## वोलेटिलिटी चीटशीट

### वोलेटिलिटी क्या है?

वोलेटिलिटी एक खुला स्रोत (open-source) रूपांतरण उपकरण है जिसे डिजिटल फोरेंसिक्स और मेमोरी एनालिसिस के लिए उपयोग किया जाता है। यह उपकरण विभिन्न ऑपरेटिंग सिस्टम पर चलाया जा सकता है और इसका उपयोग करके आप मेमोरी डंप का विश्लेषण कर सकते हैं।

### वोलेटिलिटी के लाभ

- वोलेटिलिटी आपको ऑपरेटिंग सिस्टम के विभिन्न पहलुओं का विश्लेषण करने की अनुमति देता है, जैसे प्रक्रियाओं, सेवाओं, नेटवर्क कनेक्शन्स, रजिस्ट्री एंट्रीज़, और बहुत कुछ।
- यह आपको विभिन्न ऑपरेटिंग सिस्टम पर चल रहे मालवेयर और रूटकिट्स का पता लगाने में मदद करता है।
- यह आपको डिजिटल फोरेंसिक्स के लिए उपयोगी जानकारी प्रदान करता है, जैसे कि प्रक्रिया के दौरान चल रहे थ्रेड्स, फ़ाइल के खोलने और बंद करने के लिए उपयोग हो रहे हैंडल्स, और बहुत कुछ।

### वोलेटिलिटी के उपयोग

वोलेटिलिटी का उपयोग करके आप निम्नलिखित कार्रवाईयों को कर सकते हैं:

- प्रक्रिया और सेवाओं का विश्लेषण करें
- नेटवर्क कनेक्शन्स का विश्लेषण करें
- रजिस्ट्री एंट्रीज़ का विश्लेषण करें
- फ़ाइल और डायरेक्टरी संरचना का विश्लेषण करें
- डिस्क और फ़ाइल सिस्टम का विश्लेषण करें
- रूटकिट्स और मालवेयर का पता लगाएं
- विभिन्न ऑपरेटिंग सिस्टम पर चल रहे थ्रेड्स का विश्लेषण करें
- डिजिटल फोरेंसिक्स के लिए उपयोगी जानकारी प्राप्त करें

### वोलेटिलिटी कमांड्स

यहां कुछ उपयोगी वोलेटिलिटी कमांड्स हैं:

- `imageinfo`: मेमोरी डंप की जानकारी प्रदान करता है।
- `pslist`: चल रही प्रक्रियाओं की सूची प्रदान करता है।
- `pstree`: प्रक्रिया के वृक्षाकार रूप में चल रही प्रक्रियाओं की सूची प्रदान करता है।
- `dlllist`: प्रक्रियाओं के लिए लोड किए गए DLL फ़ाइलों की सूची प्रदान करता है।
- `handles`: प्रक्रियाओं के लिए खोले गए हैंडल्स की सूची प्रदान करता है।
- `connections`: नेटवर्क कनेक्शन्स की सूची प्रदान करता है।
- `svcscan`: चल रही सेवाओं की सूची प्रदान करता है।
- `malfind`: मालवेयर के संकेतों की खोज करता है।
- `filescan`: फ़ाइलों और डायरेक्टरीज़ की सूची प्रदान करता है।
- `cmdline`: प्रक्रियाओं के लिए चल रहे कमांडलाइन विधानों की सूची प्रदान करता है।
- `vadinfo`: वर्चुअल एड्रेस स्पेस की जानकारी प्रदान करता है।

### वोलेटिलिटी प्लगइन्स

यहां कुछ उपयोगी वोलेटिलिटी प्लगइन्स हैं:

- `malfind`: मालवेयर के संकेतों की खोज करता है।
- `timeliner`: विभिन्न घटनाओं के लिए टाइमलाइन बनाता है।
- `vadtree`: वर्चुअल एड्रेस स्पेस के वृक्षाकार रूप में वर्चुअल एड्रेस डेस्क्रिप्टर विवरण प्रदान करता है।
- `vaddump`: वर्चुअल एड्रेस स्पेस के डेटा को डंप करता है।
- `vadinfo`: वर्चुअल एड्रेस स्पेस की जानकारी प्रदान करता है।
- `vadwalk`: वर्चुअल एड्रेस स्पेस के वॉकर विवरण प्रदान करता है।

### वोलेटिलिटी इंस्टॉलेशन

वोलेटिलिटी को इंस्टॉल करने के लिए निम्नलिखित चरणों का पालन करें:

1. वोलेटिलिटी को डाउनलोड करें और इंस्टॉल करें।
2. वोलेटिलिटी को अपने प्रणाली पर चलाएं।
3. वोलेटिलिटी कमांड्स का उपयोग करें और मेमोरी डंप का विश्लेषण करें।

### वोलेटिलिटी संसाधन

यहां कुछ उपयोगी वोलेटिलिटी संसाधन हैं:

- [वोलेटिलिटी वेबसाइट](https://www.volatilityfoundation.org/)
- [वोलेटिलिटी गिटहब पेज](https://github.com/volatilityfoundation/volatility)
- [वोलेटिलिटी डॉक्यूमेंटेशन](https://github.com/volatilityfoundation/volatility/wiki)
- [वोलेटिलिटी फोरम](https://volatility.groups.io/g/main)
- [वोलेटिलिटी चीटशीट](https://github.com/sans-dfir/sift/blob/master/Volatility%20Cheatsheet.pdf)

### वोलेटिलिटी समर्थन

यदि आपको वोलेटिलिटी का उपयोग करने में किसी भी समस्या का सामना करना पड़ता है, तो आप निम्नलिखित संसाधनों का उपयोग कर सकते हैं:

- [वोलेटिलिटी वेबसाइट](https://www.volatilityfoundation.org/)
- [वोलेटिलिटी गिटहब पेज](https://github.com/volatilityfoundation/volatility)
- [वोलेटिलिटी डॉक्यूमेंटेशन](https://github.com/volatilityfoundation/volatility/wiki)
- [वोलेटिलिटी फोरम](https://volatility.groups.io/g/main)
```
volatility --profile=Win7SP1x86_23418 -f file.dmp userassist
```
{% endtab %}
{% endtabs %}

​

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

​​​​[**RootedCON**](https://www.rootedcon.com/) स्पेन में सबसे महत्वपूर्ण साइबर सुरक्षा कार्यक्रम है और यूरोप में सबसे महत्वपूर्ण में से एक है। **तकनीकी ज्ञान को बढ़ावा देने** की मिशन के साथ, यह सम्मेलन प्रौद्योगिकी और साइबर सुरक्षा विशेषज्ञों के लिए एक उबलता हुआ मिलन स्थान है।

{% embed url="https://www.rootedcon.com/" %}

## सेवाएं

{% tabs %}
{% tab title="vol3" %}
```bash
./vol.py -f file.dmp windows.svcscan.SvcScan #List services
./vol.py -f file.dmp windows.getservicesids.GetServiceSIDs #Get the SID of services
```
## वोलेटिलिटी चीटशीट

### वोलेटिलिटी क्या है?

वोलेटिलिटी एक खुला स्रोत (open-source) रूपांतरण उपकरण है जिसे डिजिटल फोरेंसिक्स और मेमोरी फंडा विशेषज्ञों द्वारा उपयोग किया जाता है। यह उपकरण विंडोज, लिनक्स और मैक ऑपरेटिंग सिस्टम पर काम करता है और इन सिस्टमों के मेमोरी डंप को विश्लेषण करने की क्षमता प्रदान करता है।

### वोलेटिलिटी के उपयोग

वोलेटिलिटी का उपयोग डिजिटल फोरेंसिक्स और मेमोरी फंडा विशेषज्ञों द्वारा विभिन्न कार्यों के लिए किया जाता है, जैसे:

- अद्यतन और विश्लेषण के लिए विंडोज, लिनक्स और मैक मेमोरी डंप का उपयोग करना।
- प्रक्रिया और थ्रेड की जांच करने के लिए विंडोज, लिनक्स और मैक मेमोरी डंप का उपयोग करना।
- नेटवर्क विश्लेषण के लिए पैकेट कैप्चर फ़ाइल का उपयोग करना।
- रजिस्ट्री और फ़ाइल सिस्टम की जांच करने के लिए विंडोज मेमोरी डंप का उपयोग करना।

### वोलेटिलिटी के फायदे

वोलेटिलिटी के उपयोग से निम्नलिखित फायदे हो सकते हैं:

- विंडोज, लिनक्स और मैक मेमोरी डंप का विश्लेषण करने की क्षमता।
- प्रक्रिया और थ्रेड की जांच करने की क्षमता।
- नेटवर्क विश्लेषण के लिए पैकेट कैप्चर फ़ाइल का उपयोग करने की क्षमता।
- रजिस्ट्री और फ़ाइल सिस्टम की जांच करने की क्षमता।

### वोलेटिलिटी के उपकरण

वोलेटिलिटी के उपकरण निम्नलिखित हो सकते हैं:

- `volatility`: यह वोलेटिलिटी का मुख्य उपकरण है और विंडोज, लिनक्स और मैक मेमोरी डंप का विश्लेषण करने के लिए उपयोग किया जाता है।
- `volshell`: यह वोलेटिलिटी का एक अतिरिक्त उपकरण है जिसे इंटरैक्टिव मोड में उपयोग किया जा सकता है।
- `volshell`: यह वोलेटिलिटी का एक अतिरिक्त उपकरण है जिसे इंटरैक्टिव मोड में उपयोग किया जा सकता है।

### वोलेटिलिटी के बेसिक कमांड्स

वोलेटिलिटी के बेसिक कमांड्स निम्नलिखित हो सकते हैं:

- `imageinfo`: मेमोरी डंप की जानकारी प्रदान करता है।
- `pslist`: चल रही प्रक्रियाओं की सूची प्रदान करता है।
- `pstree`: प्रक्रिया वृक्ष प्रदान करता है।
- `dlllist`: प्रक्रियाओं के लिए लोड किए गए DLL की सूची प्रदान करता है।
- `handles`: प्रक्रियाओं के लिए हैंडल की सूची प्रदान करता है।
- `cmdline`: प्रक्रियाओं के लिए कमांड लाइन प्रदान करता है।
- `filescan`: फ़ाइलों की सूची प्रदान करता है।
- `netscan`: नेटवर्क कनेक्शन की सूची प्रदान करता है।
- `connections`: नेटवर्क कनेक्शन की सूची प्रदान करता है।
- `malfind`: संदेहास्पद या अवैध कार्यों की खोज करता है।
- `cmdscan`: कमांड इंजेक्शन की खोज करता है।
- `ssdt`: SSDT (System Service Descriptor Table) की सूची प्रदान करता है।
- `driverscan`: लोड किए गए ड्राइवरों की सूची प्रदान करता है।
- `modscan`: लोड किए गए मॉड्यूलों की सूची प्रदान करता है।
- `ssdt`: SSDT (System Service Descriptor Table) की सूची प्रदान करता है।
- `driverscan`: लोड किए गए ड्राइवरों की सूची प्रदान करता है।
- `modscan`: लोड किए गए मॉड्यूलों की सूची प्रदान करता है।

### वोलेटिलिटी के उपयोगी रेफरेंस

वोलेटिलिटी के उपयोगी रेफरेंस निम्नलिखित हो सकते हैं:

- [वोलेटिलिटी डॉक्यूमेंटेशन](https://github.com/volatilityfoundation/volatility/wiki)
- [वोलेटिलिटी ट्यूटोरियल](https://www.volatilityfoundation.org/#!volatility-tutorial/c1qkz)
- [वोलेटिलिटी फोरेंसिक्स टेस्ट केस](https://github.com/volatilityfoundation/volatility/wiki/Memory-Samples)
- [वोलेटिलिटी फोरेंसिक्स टेस्ट केस](https://github.com/volatilityfoundation/volatility/wiki/Memory-Samples)
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
## वोलेटिलिटी चीटशीट

यह चीटशीट वोलेटिलिटी टूल के लिए एक संक्षेप मार्गदर्शिका है। यह आपको वोलेटिलिटी टूल के साथ विभिन्न मेमोरी डंप विश्लेषण तकनीकों की एक सारगर्भित सूची प्रदान करेगी।

### वोलेटिलिटी टूल का उपयोग करना

1. वोलेटिलिटी टूल को डाउनलोड करें और स्थापित करें।
2. टर्मिनल खोलें और वोलेटिलिटी टूल के निर्देशानुसार उपयोग करें।

### वोलेटिलिटी टूल की मुख्य आदेश

- `imageinfo`: मेमोरी डंप की जानकारी प्रदान करता है।
- `pslist`: सिस्टम प्रक्रियाओं की सूची प्रदान करता है।
- `pstree`: प्रक्रिया पेड़ का विवरण प्रदान करता है।
- `dlllist`: प्रक्रियाओं के लिए लोड किए गए DLL की सूची प्रदान करता है।
- `handles`: प्रक्रियाओं के लिए हैंडल की सूची प्रदान करता है।
- `cmdline`: प्रक्रियाओं के लिए चल रहे आदेशों की सूची प्रदान करता है।
- `filescan`: फ़ाइल सिस्टम में फ़ाइलों की खोज करता है।
- `netscan`: नेटवर्क कनेक्शन की सूची प्रदान करता है।
- `connections`: नेटवर्क कनेक्शन की सूची प्रदान करता है।
- `malfind`: संदिग्ध मेमोरी क्षेत्रों की खोज करता है।
- `dumpfiles`: फ़ाइलों को मेमोरी से निकालता है।
- `cmdscan`: प्रक्रियाओं के लिए चल रहे आदेशों की सूची प्रदान करता है।
- `hivelist`: रजिस्ट्री हाइव की सूची प्रदान करता है।
- `hivedump`: रजिस्ट्री हाइव से डेटा निकालता है।
- `hashdump`: पासवर्ड हैश की सूची प्रदान करता है।
- `privs`: प्रक्रियाओं के लिए अनुमतियों की सूची प्रदान करता है।
- `getsids`: प्रक्रियाओं के लिए सुरक्षा अभियांत्रिकी अभियांत्रिकी आईडी की सूची प्रदान करता है।
- `envars`: प्रक्रियाओं के लिए वातावरण चरों की सूची प्रदान करता है।
- `svcscan`: सेवा की सूची प्रदान करता है।
- `driverirp`: ड्राइवर के IRP की सूची प्रदान करता है।
- `ssdt`: SSDT की सूची प्रदान करता है।
- `gdt`: GDT की सूची प्रदान करता है।
- `idt`: IDT की सूची प्रदान करता है।
- `ldrmodules`: लोड किए गए मॉड्यूल की सूची प्रदान करता है।
- `modscan`: लोड किए गए मॉड्यूल की सूची प्रदान करता है।
- `ss`: स्क्रीनशॉट लेता है।
- `memdump`: मेमोरी डंप बनाता है।
- `mbrparser`: MBR की जानकारी प्रदान करता है।
- `yarascan`: यारा नियमों के आधार पर संदिग्ध फ़ाइलों की खोज करता है।
- `vadinfo`: VAD की जानकारी प्रदान करता है।
- `vaddump`: VAD से डेटा निकालता है।
- `vadtree`: VAD पेड़ का विवरण प्रदान करता है।
- `vadwalk`: VAD पेड़ की जांच करता है।
- `vadscan`: VAD की सूची प्रदान करता है।
- `vadpfn`: VAD के लिए PFN की सूची प्रदान करता है।
- `vadtree`: VAD पेड़ का विवरण प्रदान करता है।
- `vadwalk`: VAD पेड़ की जांच करता है।
- `vadscan`: VAD की सूची प्रदान करता है।
- `vadpfn`: VAD के लिए PFN की सूची प्रदान करता है।

### वोलेटिलिटी टूल के अतिरिक्त आदेश

- `kdbgscan`: डीबगर की जानकारी प्रदान करता है।
- `ssdt`: SSDT की सूची प्रदान करता है।
- `gdt`: GDT की सूची प्रदान करता है।
- `idt`: IDT की सूची प्रदान करता है।
- `ldrmodules`: लोड किए गए मॉड्यूल की सूची प्रदान करता है।
- `modscan`: लोड किए गए मॉड्यूल की सूची प्रदान करता है।
- `ss`: स्क्रीनशॉट लेता है।
- `memdump`: मेमोरी डंप बनाता है।
- `mbrparser`: MBR की जानकारी प्रदान करता है।
- `yarascan`: यारा नियमों के आधार पर संदिग्ध फ़ाइलों की खोज करता है।
- `vadinfo`: VAD की जानकारी प्रदान करता है।
- `vaddump`: VAD से डेटा निकालता है।
- `vadtree`: VAD पेड़ का विवरण प्रदान करता है।
- `vadwalk`: VAD पेड़ की जांच करता है।
- `vadscan`: VAD की सूची प्रदान करता है।
- `vadpfn`: VAD के लिए PFN की सूची प्रदान करता है।
- `vadtree`: VAD पेड़ का विवरण प्रदान करता है।
- `vadwalk`: VAD पेड़ की जांच करता है।
- `vadscan`: VAD की सूची प्रदान करता है।
- `vadpfn`: VAD के लिए PFN की सूची प्रदान करता है।
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
{% endtab %}
{% endtabs %}

## रजिस्ट्री हाइव

### उपलब्ध हाइव्स प्रिंट करें

{% tabs %}
{% tab title="vol3" %}
```bash
./vol.py -f file.dmp windows.registry.hivelist.HiveList #List roots
./vol.py -f file.dmp windows.registry.printkey.PrintKey #List roots and get initial subkeys
```
## वोलेटिलिटी चीटशीट

यह चीटशीट वोलेटिलिटी टूल के लिए एक संक्षेप मार्गदर्शिका है। यह आपको वोलेटिलिटी टूल के साथ विभिन्न मेमोरी डंप विश्लेषण तकनीकों की एक सारगर्भित सूची प्रदान करेगी।

### वोलेटिलिटी टूल का उपयोग करना

1. वोलेटिलिटी टूल को डाउनलोड करें और स्थापित करें।
2. टर्मिनल खोलें और वोलेटिलिटी टूल के निर्देशानुसार उपयोग करें।

### वोलेटिलिटी टूल की मुख्य आदेश

- `imageinfo`: इमेज की जानकारी प्रदान करता है।
- `pslist`: प्रक्रियाओं की सूची प्रदान करता है।
- `pstree`: प्रक्रियाओं का पेड़ दर्शाता है।
- `dlllist`: प्रक्रियाओं के लिए लोड किए गए DLL प्रदान करता है।
- `handles`: हैंडल्स की सूची प्रदान करता है।
- `filescan`: फ़ाइलों की सूची प्रदान करता है।
- `cmdline`: प्रक्रियाओं की कमांड लाइन प्रदान करता है।
- `consoles`: कंसोल्स की सूची प्रदान करता है।
- `vadinfo`: वर्चुअल एड्रेस स्पेस की जानकारी प्रदान करता है।
- `vadtree`: वर्चुअल एड्रेस स्पेस का पेड़ दर्शाता है।
- `vaddump`: वर्चुअल एड्रेस स्पेस को डंप करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `ldrmodules`: लोड किए गए मॉड्यूलों की सूची प्रदान करता है।
- `apihooks`: API हुक की सूची प्रदान करता है।
- `svcscan`: सेवा की सूची प्रदान करता है।
- `ssdt`: SSDT की सूची प्रदान करता है।
- `gdt`: GDT की सूची प्रदान करता है।
- `idt`: IDT की सूची प्रदान करता है।
- `modules`: मॉड्यूलों की सूची प्रदान करता है।
- `modscan`: मॉड्यूलों की सूची प्रदान करता है।
- `ss`: स्क्रीनशॉट लेता है।
- `memdump`: मेमोरी डंप करता है।
- `hashdump`: हैश डंप करता है।
- `hivelist`: हाइव की सूची प्रदान करता है।
- `hivedump`: हाइव डंप करता है।
- `printkey`: रजिस्ट्री की कुंजी की जानकारी प्रदान करता है।
- `dumpregistry`: रजिस्ट्री को डंप करता है।
- `cmdscan`: कमांड लाइन की सूची प्रदान करता है।
- `getsids`: सिक्योरिटी आइडेंटिफ़ायर की सूची प्रदान करता है।
- `getsid`: सिक्योरिटी आइडेंटिफ़ायर की जानकारी प्रदान करता है।
- `envars`: एनवायरनमेंट वेरिएबल्स की सूची प्रदान करता है।
- `svcscan`: सेवा की सूची प्रदान करता है।
- `handles`: हैंडल्स की सूची प्रदान करता है।
- `privs`: विशेषाधिकारों की सूची प्रदान करता है।
- `priv2system`: विशेषाधिकारों को सिस्टम विशेषाधिकारों में परिवर्तित करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खो
```bash
volatility --profile=Win7SP1x86_23418 -f file.dmp hivelist #List roots
volatility --profile=Win7SP1x86_23418 -f file.dmp printkey #List roots and get initial subkeys
```
{% endtab %}
{% endtabs %}

### एक मान प्राप्त करें

{% tabs %}
{% tab title="vol3" %}
```bash
./vol.py -f file.dmp windows.registry.printkey.PrintKey --key "Software\Microsoft\Windows NT\CurrentVersion"
```
## वोलेटिलिटी चीटशीट

### वोलेटिलिटी क्या है?

वोलेटिलिटी एक खुला स्रोत (open-source) रूपांतरण उपकरण है जिसे डिजिटल फोरेंसिक्स और मेमोरी एनालिसिस के लिए उपयोग किया जाता है। यह उपकरण विभिन्न ऑपरेटिंग सिस्टम पर चलाया जा सकता है और इसका उपयोग करके आप मेमोरी डंप का विश्लेषण कर सकते हैं।

### वोलेटिलिटी के लाभ

- वोलेटिलिटी आपको ऑपरेटिंग सिस्टम के विभिन्न पहलुओं का विश्लेषण करने की अनुमति देता है, जैसे प्रक्रियाओं, सेवाओं, नेटवर्क कनेक्शन्स, रजिस्ट्री एंट्रीज़, और अधिक।
- यह आपको विभिन्न ऑपरेटिंग सिस्टम पर चल रहे मालवेयर और रूटकिट्स का पता लगाने में मदद करता है।
- इसका उपयोग करके आप डिजिटल फोरेंसिक्स के क्षेत्र में अधिक विशेषज्ञता प्राप्त कर सकते हैं।

### वोलेटिलिटी कमांड्स

यहां कुछ महत्वपूर्ण वोलेटिलिटी कमांड्स हैं:

- `imageinfo`: मेमोरी डंप की मेटाडेटा जानने के लिए उपयोग किया जाता है।
- `pslist`: सिस्टम प्रक्रियाओं की सूची प्रदर्शित करने के लिए उपयोग किया जाता है।
- `pstree`: प्रक्रिया हियरार्की को प्रदर्शित करने के लिए उपयोग किया जाता है।
- `dlllist`: प्रक्रियाओं के लिए लोड किए गए DLL फ़ाइलों की सूची प्रदर्शित करने के लिए उपयोग किया जाता है।
- `handles`: प्रक्रियाओं द्वारा उपयोग किए जाने वाले हैंडल्स की सूची प्रदर्शित करने के लिए उपयोग किया जाता है।
- `cmdline`: प्रक्रियाओं के लिए चल रहे कमांड लाइन विधि को प्रदर्शित करने के लिए उपयोग किया जाता है।
- `filescan`: फ़ाइल सिस्टम में फ़ाइलों की खोज करने के लिए उपयोग किया जाता है।
- `netscan`: नेटवर्क कनेक्शन्स की सूची प्रदर्शित करने के लिए उपयोग किया जाता है।
- `connections`: नेटवर्क कनेक्शन्स की सूची प्रदर्शित करने के लिए उपयोग किया जाता है।
- `malfind`: संदेहास्पद मालवेयर के लिए खोज करने के लिए उपयोग किया जाता है।
- `dumpfiles`: फ़ाइलों को मेमोरी डंप से बाहर निकालने के लिए उपयोग किया जाता है।

### वोलेटिलिटी प्लगइन्स

यहां कुछ महत्वपूर्ण वोलेटिलिटी प्लगइन्स हैं:

- `malfind`: संदेहास्पद मालवेयर के लिए खोज करने के लिए उपयोग किया जाता है।
- `timeliner`: विभिन्न घटनाओं के बीच समय की गति का विश्लेषण करने के लिए उपयोग किया जाता है।
- `vadinfo`: विभिन्न प्रक्रियाओं के लिए वर्चुअल एड्रेस स्पेस की जानकारी प्रदान करने के लिए उपयोग किया जाता है।
- `vadtree`: वर्चुअल एड्रेस स्पेस की विभिन्न प्रक्रियाओं के लिए विवरण प्रदान करने के लिए उपयोग किया जाता है।
- `vaddump`: वर्चुअल एड्रेस स्पेस से डेटा निकालने के लिए उपयोग किया जाता है।
- `vadwalk`: वर्चुअल एड्रेस स्पेस की विभिन्न प्रक्रियाओं के लिए विवरण प्रदान करने के लिए उपयोग किया जाता है।

### वोलेटिलिटी इंस्टॉलेशन

वोलेटिलिटी को इंस्टॉल करने के लिए निम्नलिखित चरणों का पालन करें:

1. वोलेटिलिटी को डाउनलोड करें और इंस्टॉल करें।
2. वोलेटिलिटी को पथ में जोड़ें।
3. वोलेटिलिटी की संस्करण जांचें।
4. वोलेटिलिटी को सफलतापूर्वक चलाएं।

### वोलेटिलिटी उपयोग

वोलेटिलिटी का उपयोग करने के लिए निम्नलिखित चरणों का पालन करें:

1. वोलेटिलिटी को चलाएं और उपयोगकर्ता अनुमतियों के साथ इसे चलाएं।
2. वोलेटिलिटी कमांड्स का उपयोग करें और विश्लेषण करने के लिए उपयुक्त डेटा प्राप्त करें।
3. प्राप्त डेटा का विश्लेषण करें और आवश्यक जानकारी प्राप्त करें।

### वोलेटिलिटी संदर्भ

यहां कुछ महत्वपूर्ण वोलेटिलिटी संदर्भ हैं:

- [वोलेटिलिटी डॉक्यूमेंटेशन](https://www.volatilityfoundation.org/)
- [वोलेटिलिटी गिटहब रेपो](https://github.com/volatilityfoundation/volatility)
```bash
volatility --profile=Win7SP1x86_23418 printkey -K "Software\Microsoft\Windows NT\CurrentVersion" -f file.dmp
# Get Run binaries registry value
volatility -f file.dmp --profile=Win7SP1x86 printkey -o 0x9670e9d0 -K 'Software\Microsoft\Windows\CurrentVersion\Run'
```
{% endtab %}
{% endtabs %}

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
## वोलेटिलिटी चीटशीट

यह चीटशीट वोलेटिलिटी टूल के लिए एक संक्षेप मार्गदर्शिका है। यह आपको वोलेटिलिटी टूल के साथ विभिन्न मेमोरी डंप विश्लेषण तकनीकों की एक सारगर्भित सूची प्रदान करेगी।

### वोलेटिलिटी टूल का उपयोग करना

1. वोलेटिलिटी टूल को डाउनलोड करें और स्थापित करें।
2. टर्मिनल खोलें और वोलेटिलिटी टूल के निर्देशानुसार उपयोग करें।

### वोलेटिलिटी टूल की मुख्य आदेश

- `imageinfo`: मेमोरी डंप की जानकारी प्रदान करता है।
- `pslist`: सिस्टम प्रक्रियाओं की सूची प्रदान करता है।
- `pstree`: प्रक्रिया पेड़ का विवरण प्रदान करता है।
- `dlllist`: प्रक्रियाओं के लिए लोड किए गए DLL की सूची प्रदान करता है।
- `handles`: प्रक्रियाओं के लिए हैंडल की सूची प्रदान करता है।
- `cmdline`: प्रक्रियाओं के लिए चल रहे आदेशों की सूची प्रदान करता है।
- `filescan`: फ़ाइल सिस्टम में फ़ाइलों की खोज करता है।
- `netscan`: नेटवर्क कनेक्शन की सूची प्रदान करता है।
- `connections`: नेटवर्क कनेक्शन की सूची प्रदान करता है।
- `malfind`: संदिग्ध मेमोरी क्षेत्रों की खोज करता है।
- `dumpfiles`: फ़ाइलों को मेमोरी से निकालता है।
- `cmdscan`: प्रक्रियाओं के लिए चल रहे आदेशों की सूची प्रदान करता है।
- `hivelist`: रजिस्ट्री हाइव की सूची प्रदान करता है।
- `hivedump`: रजिस्ट्री हाइव से डेटा निकालता है।
- `hashdump`: पासवर्ड हैश की सूची प्रदान करता है।
- `privs`: प्रक्रियाओं के लिए अनुमतियों की सूची प्रदान करता है।
- `getsids`: प्रक्रियाओं के लिए सुरक्षा अभियांत्रिकी अभियांत्रिकी आईडी की सूची प्रदान करता है।
- `envars`: प्रक्रियाओं के लिए वातावरण चरों की सूची प्रदान करता है।
- `svcscan`: सेवा की सूची प्रदान करता है।
- `driverirp`: ड्राइवर के IRP की सूची प्रदान करता है।
- `ssdt`: SSDT की सूची प्रदान करता है।
- `gdt`: GDT की सूची प्रदान करता है।
- `idt`: IDT की सूची प्रदान करता है।
- `ldrmodules`: लोड किए गए मॉड्यूल की सूची प्रदान करता है।
- `modscan`: लोड किए गए मॉड्यूल की सूची प्रदान करता है।
- `ss`: स्क्रीनशॉट लेता है।
- `memdump`: मेमोरी डंप बनाता है।
- `mbrparser`: MBR की जानकारी प्रदान करता है।
- `yarascan`: यारा नियमों के आधार पर संदिग्ध फ़ाइलों की खोज करता है।
- `vadinfo`: VAD की जानकारी प्रदान करता है।
- `vaddump`: VAD से डेटा निकालता है।
- `vadtree`: VAD पेड़ का विवरण प्रदान करता है।
- `vadwalk`: VAD पेड़ की जांच करता है।
- `vadscan`: VAD की सूची प्रदान करता है।
- `vadpfn`: VAD के लिए PFN की सूची प्रदान करता है।
- `vadtree`: VAD पेड़ का विवरण प्रदान करता है।
- `vadwalk`: VAD पेड़ की जांच करता है।
- `vadscan`: VAD की सूची प्रदान करता है।
- `vadpfn`: VAD के लिए PFN की सूची प्रदान करता है।

### वोलेटिलिटी टूल के अतिरिक्त आदेश

- `kdbgscan`: डीबगर की जानकारी प्रदान करता है।
- `ssdt`: SSDT की सूची प्रदान करता है।
- `gdt`: GDT की सूची प्रदान करता है।
- `idt`: IDT की सूची प्रदान करता है।
- `ldrmodules`: लोड किए गए मॉड्यूल की सूची प्रदान करता है।
- `modscan`: लोड किए गए मॉड्यूल की सूची प्रदान करता है।
- `ss`: स्क्रीनशॉट लेता है।
- `memdump`: मेमोरी डंप बनाता है।
- `mbrparser`: MBR की जानकारी प्रदान करता है।
- `yarascan`: यारा नियमों के आधार पर संदिग्ध फ़ाइलों की खोज करता है।
- `vadinfo`: VAD की जानकारी प्रदान करता है।
- `vaddump`: VAD से डेटा निकालता है।
- `vadtree`: VAD पेड़ का विवरण प्रदान करता है।
- `vadwalk`: VAD पेड़ की जांच करता है।
- `vadscan`: VAD की सूची प्रदान करता है।
- `vadpfn`: VAD के लिए PFN की सूची प्रदान करता है।
- `vadtree`: VAD पेड़ का विवरण प्रदान करता है।
- `vadwalk`: VAD पेड़ की जांच करता है।
- `vadscan`: VAD की सूची प्रदान करता है।
- `vadpfn`: VAD के लिए PFN की सूची प्रदान करता है।
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
## वोलेटिलिटी चीटशीट

यह चीटशीट वोलेटिलिटी टूल के लिए एक संक्षेप मार्गदर्शिका है। यह आपको वोलेटिलिटी टूल के साथ विभिन्न मेमोरी डंप विश्लेषण तकनीकों की एक सारगर्भित सूची प्रदान करेगी।

### वोलेटिलिटी टूल का उपयोग करना

1. वोलेटिलिटी टूल को डाउनलोड करें और स्थापित करें।
2. टर्मिनल खोलें और वोलेटिलिटी टूल के निर्देशानुसार उपयोग करें।

### वोलेटिलिटी टूल की मुख्य आदेश

- `imageinfo`: इमेज की जानकारी प्रदान करता है।
- `pslist`: प्रक्रियाओं की सूची प्रदान करता है।
- `pstree`: प्रक्रियाओं का पेड़ दर्शाता है।
- `dlllist`: प्रक्रियाओं के लिए लोड किए गए DLL प्रदान करता है।
- `handles`: हैंडल्स की सूची प्रदान करता है।
- `cmdline`: प्रक्रिया के लिए आपूर्ति लाइन प्रदान करता है।
- `consoles`: कंसोल की सूची प्रदान करता है।
- `filescan`: फ़ाइलों की सूची प्रदान करता है।
- `malfind`: संदेहास्पद विचारों की खोज करता है।
- `svcscan`: सेवाओं की सूची प्रदान करता है।
- `connections`: संबंधों की सूची प्रदान करता है।
- `sockets`: सॉकेट की सूची प्रदान करता है।
- `netscan`: नेटवर्क की सूची प्रदान करता है।
- `privs`: उपयोगकर्ता की विशेषाधिकार सूची प्रदान करता है।
- `cmdscan`: कमांड लाइन की सूची प्रदान करता है।
- `driverirp`: ड्राइवर के IRP संदेश प्रदान करता है।
- `ssdt`: SSDT की सूची प्रदान करता है।
- `gdt`: GDT की सूची प्रदान करता है।
- `idt`: IDT की सूची प्रदान करता है।
- `ldrmodules`: लोड किए गए मॉड्यूलों की सूची प्रदान करता है।
- `modscan`: मॉड्यूलों की सूची प्रदान करता है।
- `ss`: स्थिति की सूची प्रदान करता है।
- `callbacks`: कॉलबैक की सूची प्रदान करता है।
- `devicetree`: डिवाइस ट्री की सूची प्रदान करता है।
- `hivelist`: हाइव की सूची प्रदान करता है।
- `hivedump`: हाइव को डंप करता है।
- `hashdump`: हैश डंप करता है।
- `userassist`: उपयोगकर्ता सहायता की सूची प्रदान करता है।
- `shellbags`: शैलबैग की सूची प्रदान करता है।
- `mftparser`: MFT को पार्स करता है।
- `mftparser -D`: MFT को विस्तार से पार्स करता है।
- `mftparser -F <file>`: निर्दिष्ट फ़ाइल की MFT को पार्स करता है।
- `mftparser -r <registry>`: निर्दिष्ट रजिस्ट्री की MFT को पार्स करता है।
- `mftparser -R <registry>`: निर्दिष्ट रजिस्ट्री की MFT को विस्तार से पार्स करता है।
- `mftparser -s <file>`: निर्दिष्ट फ़ाइल की MFT को विस्तार से पार्स करता है।
- `mftparser -t <file>`: निर्दिष्ट फ़ाइल की MFT को पार्स करता है और उसकी विवरणात्मक जानकारी प्रदान करता है।
- `mftparser -v <file>`: निर्दिष्ट फ़ाइल की MFT को पार्स करता है और उसकी विवरणात्मक जानकारी प्रदान करता है।
- `mftparser -x <file>`: निर्दिष्ट फ़ाइल की MFT को पार्स करता है और उसकी विवरणात्मक जानकारी प्रदान करता है।
- `mftparser -y <file>`: निर्दिष्ट फ़ाइल की MFT को पार्स करता है और उसकी विवरणात्मक जानकारी प्रदान करता है।
- `mftparser -z <file>`: निर्दिष्ट फ़ाइल की MFT को पार्स करता है और उसकी विवरणात्मक जानकारी प्रदान करता है।

### वोलेटिलिटी टूल के लिए अतिरिक्त संसाधन

- [वोलेटिलिटी टूल की आधिकारिक वेबसाइट](https://www.volatilityfoundation.org/)
- [वोलेटिलिटी टूल का स्रोत कोड](https://github.com/volatilityfoundation/volatility)
- [वोलेटिलिटी टूल की डॉक्यूमेंटेशन](https://github.com/volatilityfoundation/volatility/wiki)
```bash
volatility --profile=Win7SP1x86_23418 filescan -f file.dmp #Scan for files inside the dump
volatility --profile=Win7SP1x86_23418 dumpfiles -n --dump-dir=/tmp -f file.dmp #Dump all files
volatility --profile=Win7SP1x86_23418 dumpfiles -n --dump-dir=/tmp -Q 0x000000007dcaa620 -f file.dmp

volatility --profile=SomeLinux -f file.dmp linux_enumerate_files
volatility --profile=SomeLinux -f file.dmp linux_find_file -F /path/to/file
volatility --profile=SomeLinux -f file.dmp linux_find_file -i 0xINODENUMBER -O /path/to/dump/file
```
{% endtab %}
{% endtabs %}

### मास्टर फ़ाइल टेबल

{% tabs %}
{% tab title="vol3" %}
```bash
# I couldn't find any plugin to extract this information in volatility3
```
## वोलेटिलिटी चीटशीट

यह चीटशीट वोलेटिलिटी टूल के लिए एक संक्षेप मार्गदर्शिका है। यह आपको वोलेटिलिटी टूल के साथ विभिन्न मेमोरी डंप विश्लेषण तकनीकों की एक सारगर्भित सूची प्रदान करेगी।

### वोलेटिलिटी टूल का उपयोग करना

1. वोलेटिलिटी टूल को डाउनलोड करें और स्थापित करें।
2. टर्मिनल खोलें और वोलेटिलिटी टूल के निर्देशानुसार उपयोग करें।

### वोलेटिलिटी टूल की मुख्य आदेश

- `imageinfo`: इमेज की जानकारी प्रदान करता है।
- `pslist`: प्रक्रियाओं की सूची प्रदान करता है।
- `pstree`: प्रक्रियाओं का पेड़ दर्शाता है।
- `dlllist`: प्रक्रियाओं के लिए लोड किए गए DLL प्रदान करता है।
- `handles`: हैंडल्स की सूची प्रदान करता है।
- `filescan`: फ़ाइलों की सूची प्रदान करता है।
- `cmdline`: प्रक्रियाओं की कमांड लाइन प्रदान करता है।
- `consoles`: कंसोल्स की सूची प्रदान करता है।
- `vadinfo`: वर्चुअल एड्रेस स्पेस की जानकारी प्रदान करता है।
- `vadtree`: वर्चुअल एड्रेस स्पेस का पेड़ दर्शाता है।
- `vaddump`: वर्चुअल एड्रेस स्पेस को डंप करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `ldrmodules`: लोड किए गए मॉड्यूलों की सूची प्रदान करता है।
- `apihooks`: API हुक की सूची प्रदान करता है।
- `svcscan`: सेवा की सूची प्रदान करता है।
- `ssdt`: SSDT की सूची प्रदान करता है।
- `gdt`: GDT की सूची प्रदान करता है।
- `idt`: IDT की सूची प्रदान करता है।
- `modules`: मॉड्यूलों की सूची प्रदान करता है।
- `modscan`: मॉड्यूलों की सूची प्रदान करता है।
- `ss`: स्क्रीनशॉट लेता है।
- `memdump`: मेमोरी डंप करता है।
- `hashdump`: हैश डंप करता है।
- `hivelist`: हाइव की सूची प्रदान करता है।
- `hivedump`: हाइव डंप करता है।
- `printkey`: रजिस्ट्री की कुंजी की जानकारी प्रदान करता है।
- `dumpregistry`: रजिस्ट्री को डंप करता है।
- `cmdscan`: कमांड शेल की सूची प्रदान करता है।
- `mbrparser`: MBR का विश्लेषण करता है।
- `yarascan`: यारा नियमों के आधार पर स्कैन करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी रीजर्वेशन की खोज करता है।
- `malfind`: संदिग्ध मेमोरी र
```bash
volatility --profile=Win7SP1x86_23418 mftparser -f file.dmp
```
{% endtab %}
{% endtabs %}

NTFS फ़ाइल सिस्टम में _मास्टर फ़ाइल टेबल_ या MFT नामक एक फ़ाइल होती है। MFT में कम से कम एक प्रविष्टि होती है हर NTFS फ़ाइल सिस्टम वॉल्यूम पर, जिसमें MFT खुद को भी शामिल है। **एक फ़ाइल के बारे में सभी जानकारी, जिसमें उसका आकार, समय और तिथि स्टैम्प, अनुमतियाँ और डेटा सामग्री शामिल हैं**, या तो MFT प्रविष्टियों में संग्रहीत होती हैं, या MFT द्वारा वर्णित बाहरी स्थान में। [यहाँ से](https://docs.microsoft.com/en-us/windows/win32/fileio/master-file-table)।

### SSL कुंजी / प्रमाणपत्र
```bash
#vol3 allows to search for certificates inside the registry
./vol.py -f file.dmp windows.registry.certificates.Certificates
```
## वोलेटिलिटी चीटशीट

### वोलेटिलिटी क्या है?

वोलेटिलिटी एक खुला स्रोत (open-source) रूपांतरण उपकरण है जिसे डिजिटल फोरेंसिक्स और मेमोरी फंडा विशेषज्ञों द्वारा उपयोग किया जाता है। यह उपकरण विंडोज, लिनक्स और मैक ऑपरेटिंग सिस्टम पर काम करता है और इन सिस्टमों के मेमोरी डंप को विश्लेषण करने की क्षमता प्रदान करता है।

### वोलेटिलिटी के लिए चीटशीट

यहां वोलेटिलिटी उपकरण के उपयोग के लिए कुछ महत्वपूर्ण आदेश दिए गए हैं:

#### विंडोज पर वोलेटिलिटी चलाएं

```bash
volatility -f <फ़ाइल> imageinfo
```

विंडोज मेमोरी डंप के बारे में महत्वपूर्ण जानकारी प्राप्त करने के लिए इस आदेश का उपयोग करें।

```bash
volatility -f <फ़ाइल> --profile=<प्रोफ़ाइल> pslist
```

विंडोज प्रोसेस सूची प्राप्त करने के लिए इस आदेश का उपयोग करें।

```bash
volatility -f <फ़ाइल> --profile=<प्रोफ़ाइल> memdump -p <प्रक्रिया ID> -D <डंप फ़ोल्डर>
```

विंडोज प्रक्रिया के लिए मेमोरी डंप बनाने के लिए इस आदेश का उपयोग करें।

#### लिनक्स पर वोलेटिलिटी चलाएं

```bash
volatility -f <फ़ाइल> linux_banner
```

लिनक्स कर्नल बैनर प्राप्त करने के लिए इस आदेश का उपयोग करें।

```bash
volatility -f <फ़ाइल> --profile=<प्रोफ़ाइल> linux_pslist
```

लिनक्स प्रोसेस सूची प्राप्त करने के लिए इस आदेश का उपयोग करें।

```bash
volatility -f <फ़ाइल> --profile=<प्रोफ़ाइल> linux_dump_map -p <प्रक्रिया ID> -D <डंप फ़ोल्डर>
```

लिनक्स प्रक्रिया के लिए मेमोरी डंप बनाने के लिए इस आदेश का उपयोग करें।

#### मैक पर वोलेटिलिटी चलाएं

```bash
volatility -f <फ़ाइल> mac_banner
```

मैक कर्नल बैनर प्राप्त करने के लिए इस आदेश का उपयोग करें।

```bash
volatility -f <फ़ाइल> --profile=<प्रोफ़ाइल> mac_pslist
```

मैक प्रोसेस सूची प्राप्त करने के लिए इस आदेश का उपयोग करें।

```bash
volatility -f <फ़ाइल> --profile=<प्रोफ़ाइल> mac_dump_map -p <प्रक्रिया ID> -D <डंप फ़ोल्डर>
```

मैक प्रक्रिया के लिए मेमोरी डंप बनाने के लिए इस आदेश का उपयोग करें।

### वोलेटिलिटी उपकरण के अतिरिक्त संदर्भ

यहां कुछ अतिरिक्त संदर्भ दिए गए हैं जिनका उपयोग आप वोलेटिलिटी के साथ कर सकते हैं:

- [वोलेटिलिटी डॉकर इमेज](https://github.com/volatilityfoundation/volatility/wiki/Docker)
- [वोलेटिलिटी वेब इंटरफ़ेस](https://github.com/volatilityfoundation/volatility/wiki/Web-Interface)
- [वोलेटिलिटी एपीआई](https://github.com/volatilityfoundation/volatility/wiki/API)
- [वोलेटिलिटी प्लगइन्स](https://github.com/volatilityfoundation/volatility/wiki/Plugins)
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
## वोलेटिलिटी चीटशीट

यह चीटशीट वोलेटिलिटी टूल के लिए एक संक्षेप मार्गदर्शिका है। यह आपको वोलेटिलिटी टूल के साथ विभिन्न मेमोरी डंप विश्लेषण तकनीकों की एक सारगर्भित सूची प्रदान करेगी।

### वोलेटिलिटी टूल का उपयोग करना

1. वोलेटिलिटी टूल को डाउनलोड करें और स्थापित करें।
2. टर्मिनल खोलें और वोलेटिलिटी टूल के निर्देशानुसार उपयोग करें।

### वोलेटिलिटी टूल की मुख्य आदेश

- `imageinfo`: मेमोरी डंप की जानकारी प्रदान करता है।
- `pslist`: सिस्टम प्रक्रियाओं की सूची प्रदान करता है।
- `pstree`: प्रक्रिया पेड़ का विवरण प्रदान करता है।
- `dlllist`: प्रक्रियाओं के लिए लोड किए गए DLL की सूची प्रदान करता है।
- `handles`: प्रक्रियाओं के लिए हैंडल की सूची प्रदान करता है।
- `cmdline`: प्रक्रियाओं के लिए चल रहे आदेशों की सूची प्रदान करता है।
- `filescan`: फ़ाइल सिस्टम में फ़ाइलों की खोज करता है।
- `netscan`: नेटवर्क कनेक्शन की सूची प्रदान करता है।
- `connections`: नेटवर्क कनेक्शन की सूची प्रदान करता है।
- `malfind`: संदिग्ध मेमोरी क्षेत्रों की खोज करता है।
- `dumpfiles`: फ़ाइलों को डंप करता है।
- `cmdscan`: प्रक्रियाओं के लिए चल रहे आदेशों की सूची प्रदान करता है।
- `hivelist`: रजिस्ट्री हाइव की सूची प्रदान करता है।
- `hivedump`: रजिस्ट्री हाइव को डंप करता है।
- `hashdump`: पासवर्ड हैश को डंप करता है।
- `userassist`: उपयोगकर्ता सहायता रिकॉर्ड की सूची प्रदान करता है।
- `printkey`: रजिस्ट्री कुंजी की जानकारी प्रदान करता है।
- `dumpregistry`: रजिस्ट्री को डंप करता है।
- `mbrparser`: मास्टर बूट रिकॉर्ड का विश्लेषण करता है।
- `ssdt`: सिस्टम सर्विस टेबल की सूची प्रदान करता है।
- `gdt`: ग्लोबल डेस्क्रिप्टर टेबल की सूची प्रदान करता है।
- `ldrmodules`: लोडर मॉड्यूल की सूची प्रदान करता है।
- `ldrmodules -p <PID>`: एक निर्दिष्ट प्रक्रिया के लिए लोडर मॉड्यूल की सूची प्रदान करता है।
- `modscan`: लोडर मॉड्यूल की सूची प्रदान करता है।
- `modscan -p <PID>`: एक निर्दिष्ट प्रक्रिया के लिए लोडर मॉड्यूल की सूची प्रदान करता है।
- `ssdt -s`: सिस्टम सर्विस टेबल की सूची प्रदान करता है।
- `ssdt -s <SSDT>`: एक निर्दिष्ट SSDT की जानकारी प्रदान करता है।
- `gdt -g`: ग्लोबल डेस्क्रिप्टर टेबल की सूची प्रदान करता है।
- `gdt -g <GDT>`: एक निर्दिष्ट GDT की जानकारी प्रदान करता है।
- `apihooks`: API हुक की सूची प्रदान करता है।
- `apihooks -p <PID>`: एक निर्दिष्ट प्रक्रिया के लिए API हुक की सूची प्रदान करता है।
- `idt`: इंटररप्ट डेस्क्रिप्टर टेबल की सूची प्रदान करता है।
- `idt -g`: इंटररप्ट डेस्क्रिप्टर टेबल की सूची प्रदान करता है।
- `idt -g <IDT>`: एक निर्दिष्ट IDT की जानकारी प्रदान करता है।
- `callbacks`: कॉलबैक की सूची प्रदान करता है।
- `callbacks -p <PID>`: एक निर्दिष्ट प्रक्रिया के लिए कॉलबैक की सूची प्रदान करता है।
- `driverirp`: ड्राइवर IRP की सूची प्रदान करता है।
- `driverirp -p <PID>`: एक निर्दिष्ट प्रक्रिया के लिए ड्राइवर IRP की सूची प्रदान करता है।
- `ssdt -s <SSDT>`: एक निर्दिष्ट SSDT की जानकारी प्रदान करता है।
- `gdt -g <GDT>`: एक निर्दिष्ट GDT की जानकारी प्रदान करता है।
- `apihooks -p <PID>`: एक निर्दिष्ट प्रक्रिया के लिए API हुक की सूची प्रदान करता है।
- `idt -g <IDT>`: एक निर्दिष्ट IDT की जानकारी प्रदान करता है।
- `callbacks -p <PID>`: एक निर्दिष्ट प्रक्रिया के लिए कॉलबैक की सूची प्रदान करता है।
- `driverirp -p <PID>`: एक निर्दिष्ट प्रक्रिया के लिए ड्राइवर IRP की सूची प्रदान करता है।

### वोलेटिलिटी टूल के अतिरिक्त आदेश

- `malfind`: संदिग्ध मेमोरी क्षेत्रों की खोज करता है।
- `malfind -p <PID>`: एक निर्दिष्ट प्रक्रिया के लिए संदिग्ध मेमोरी क्षेत्रों की खोज करता है।
- `malfind -D <DIRECTORY>`: निर्दिष्ट निर्देशिका में संदिग्ध मेमोरी क्षेत्रों की खोज करता है।
- `malfind -p <PID> -D <DIRECTORY>`: एक निर्दिष्ट प्रक्रिया के लिए निर्दिष्ट निर्देशिका में संदिग्ध मेमोरी क्षेत्रों की खोज करता है।
- `malfind -D <DIRECTORY> -r`: निर्दिष्ट निर्देशिका में संदिग्ध मेमोरी क्षेत्रों की खोज करता है और उन्हें डंप करता है।
- `malfind -p <PID> -D <DIRECTORY> -r`: एक निर्दिष्ट प्रक्रिया के लिए निर्दिष्ट निर्देशिका में संदिग्ध मेमोरी क्षेत्रों की खोज करता है और उन्हें डंप करता है।
- `vadinfo`: विशेष एक्सेस डेटा (VAD) की जानकारी प्रदान करता है।
- `vadinfo -p <PID>`: एक निर्दिष्ट प्रक्रिया के लिए विशेष एक्सेस डेटा (VAD) की जानकारी प्रदान करता है।
- `vadtree`: विशेष एक्सेस डेटा (VAD) के पेड़ की जानकारी प्रदान करता है।
- `vadtree -p <PID>`: एक निर्दिष्ट प्रक्रिया के लिए विशेष एक्सेस डेटा (VAD) के पेड़ की जानकारी प्रदान करता है।
- `vaddump -p <PID> -D <DIRECTORY>`: एक निर्दिष्ट प्रक्रिया के लिए विशेष एक्सेस डेटा (VAD) को निर्दिष्ट निर्देशिका में डंप करता है।
- `vaddump -D <DIRECTORY>`: निर्दिष्ट निर्देशिका में सभी विशेष एक्सेस डेटा (VAD) को डंप करता है।
- `vaddump -p <PID>`: एक निर्दिष्ट प्रक्रिया के लिए सभी विशेष एक्सेस डेटा (VAD) को डंप करता है।
- `vaddump -p <PID> -D <DIRECTORY> -r`: एक निर्दिष्ट प्रक्रिया के लिए विशेष एक्सेस डेटा (VAD) को निर्दिष्ट निर्देशिका में डंप करता है और उन्हें डंप करता है।
- `vaddump -D <DIRECTORY> -r`: निर्दिष्ट निर्देशिका में सभी विशेष एक्सेस ड
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

### यारा के साथ स्कैन करना

इस स्क्रिप्ट का उपयोग करके गिटहब से सभी यारा मैलवेयर नियमों को डाउनलोड और मर्ज करें: [https://gist.github.com/andreafortuna/29c6ea48adf3d45a979a78763cdc7ce9](https://gist.github.com/andreafortuna/29c6ea48adf3d45a979a78763cdc7ce9)\
_**rules**_ नामक निर्देशिका बनाएं और इसे चलाएं। इससे _**malware\_rules.yar**_ नामक एक फ़ाइल बनेगी जिसमें मैलवेयर के लिए सभी यारा नियम होंगे।
```bash
wget https://gist.githubusercontent.com/andreafortuna/29c6ea48adf3d45a979a78763cdc7ce9/raw/4ec711d37f1b428b63bed1f786b26a0654aa2f31/malware_yara_rules.py
mkdir rules
python malware_yara_rules.py
#Only Windows
./vol.py -f file.dmp windows.vadyarascan.VadYaraScan --yara-file /tmp/malware_rules.yar
#All
./vol.py -f file.dmp yarascan.YaraScan --yara-file /tmp/malware_rules.yar
```
## वोलेटिलिटी चीटशीट

### वोलेटिलिटी क्या है?

वोलेटिलिटी एक खुला स्रोत (open-source) रूपांतरण उपकरण है जिसे डिजिटल फोरेंसिक्स और मेमोरी फंडा विशेषज्ञों द्वारा उपयोग किया जाता है। यह उपकरण विंडोज, लिनक्स और मैक ऑपरेटिंग सिस्टम पर काम करता है और इन सिस्टमों के मेमोरी डंप को विश्लेषण करने की क्षमता प्रदान करता है।

### वोलेटिलिटी के उपयोग

वोलेटिलिटी का उपयोग डिजिटल फोरेंसिक्स और मेमोरी फंडा विशेषज्ञों द्वारा विभिन्न कार्यों के लिए किया जाता है, जैसे:

- अद्यतन और विश्लेषण के लिए विंडोज, लिनक्स और मैक मेमोरी डंप का उपयोग करना।
- प्रक्रिया और थ्रेड की जांच करने के लिए विंडोज, लिनक्स और मैक मेमोरी डंप का उपयोग करना।
- नेटवर्क विश्लेषण के लिए विंडोज, लिनक्स और मैक मेमोरी डंप का उपयोग करना।
- रजिस्ट्री और फ़ाइल सिस्टम के विश्लेषण के लिए विंडोज मेमोरी डंप का उपयोग करना।

### वोलेटिलिटी कमांड्स

यहां कुछ वोलेटिलिटी कमांड्स हैं जो आपको विभिन्न कार्यों के लिए उपयोगी साबित हो सकते हैं:

- `imageinfo`: मेमोरी डंप की जानकारी प्रदान करता है।
- `pslist`: चल रही प्रक्रियाओं की सूची प्रदान करता है।
- `pstree`: प्रक्रिया वृक्ष प्रदान करता है।
- `dlllist`: प्रक्रियाओं के लिए लोड किए गए DLL की सूची प्रदान करता है।
- `handles`: प्रक्रियाओं के लिए हैंडल की सूची प्रदान करता है।
- `filescan`: फ़ाइल सिस्टम में फ़ाइलों की खोज करता है।
- `netscan`: नेटवर्क कनेक्शन की सूची प्रदान करता है।

### वोलेटिलिटी प्लगइन्स

वोलेटिलिटी के साथ कई प्लगइन्स उपलब्ध हैं जो विशेष कार्यों के लिए उपयोगी हो सकते हैं। यहां कुछ प्लगइन्स की सूची है:

- `malfind`: संदेहास्पद या अवैध प्रक्रियाओं की खोज करता है।
- `svcscan`: सेवा की सूची प्रदान करता है।
- `connscan`: नेटवर्क कनेक्शन की सूची प्रदान करता है।
- `cmdscan`: कमांड लाइन इतिहास की जांच करता है।
- `mftparser`: विंडोज फ़ाइल सिस्टम की जांच करता है।
- `hivelist`: रजिस्ट्री हाइव की सूची प्रदान करता है।

### वोलेटिलिटी इंस्टॉलेशन

वोलेटिलिटी को इंस्टॉल करने के लिए निम्नलिखित चरणों का पालन करें:

1. पायथन (Python) इंस्टॉल करें।
2. पायथन पैकेज प्रबंधक (Python Package Manager) को इंस्टॉल करें।
3. वोलेटिलिटी को डाउनलोड करें और इंस्टॉल करें।

### वोलेटिलिटी उपयोग करना

वोलेटिलिटी का उपयोग करने के लिए निम्नलिखित चरणों का पालन करें:

1. टर्मिनल खोलें और वोलेटिलिटी को चलाने के लिए निम्नलिखित कमांड दर्ज करें:

```bash
volatility
```

2. वोलेटिलिटी कमांड प्राप्त करने के लिए `--help` विकल्प का उपयोग करें:

```bash
volatility --help
```

3. वोलेटिलिटी के उपयोगी कमांड्स का उपयोग करें और उनका उपयोग करने के लिए आवश्यक विकल्पों का उपयोग करें।

### वोलेटिलिटी संसाधन

यहां कुछ वोलेटिलिटी संसाधन हैं जो आपके लिए उपयोगी साबित हो सकते हैं:

- [वोलेटिलिटी ऑफिशियल वेबसाइट](https://www.volatilityfoundation.org/)
- [वोलेटिलिटी गिटहब रेपो](https://github.com/volatilityfoundation/volatility)
- [वोलेटिलिटी डॉक्यूमेंटेशन](https://github.com/volatilityfoundation/volatility/wiki)
- [वोलेटिलिटी फोरम](https://forum.volatilityfoundation.org/)

### अभिप्रेत विधि

वोलेटिलिटी एक शक्तिशाली और उपयोगी उपकरण है जो डिजिटल फोरेंसिक्स और मेमोरी फंडा विशेषज्ञों को मेमोरी डंप विश्लेषण करने में मदद करता है। इस चीटशीट में हमने वोलेटिलिटी के बारे में जानकारी, उपयोगी कमांड्स, प्लगइन्स और संसाधनों के बारे में बात की है। यदि आप डिजिटल फोरेंसिक्स या मेमोरी फंडा में रुचि रखते हैं, तो वोलेटिलिटी आपके लिए एक महत्वपूर्ण उपकरण हो सकता है।
```bash
wget https://gist.githubusercontent.com/andreafortuna/29c6ea48adf3d45a979a78763cdc7ce9/raw/4ec711d37f1b428b63bed1f786b26a0654aa2f31/malware_yara_rules.py
mkdir rules
python malware_yara_rules.py
volatility --profile=Win7SP1x86_23418 yarascan -y malware_rules.yar -f ch2.dmp | grep "Rule:" | grep -v "Str_Win32" | sort | uniq
```
{% endtab %}
{% endtabs %}

## विविध

### बाहरी प्लगइन

यदि आप बाहरी प्लगइन का उपयोग करना चाहते हैं तो सुनिश्चित करें कि प्लगइन के संबंधित फ़ोल्डर पहले पैरामीटर के रूप में उपयोग किए जाते हैं।

{% tabs %}
{% tab title="vol3" %}
```bash
./vol.py --plugin-dirs "/tmp/plugins/" [...]
```
## वोलेटिलिटी चीटशीट

### वोलेटिलिटी क्या है?

वोलेटिलिटी एक खुला स्रोत (open-source) रूपांतरण उपकरण है जिसे डिजिटल फोरेंसिक्स और मेमोरी फंडा विशेषज्ञों द्वारा उपयोग किया जाता है। यह उपकरण विंडोज, लिनक्स और मैक ऑपरेटिंग सिस्टम पर काम करता है और इन सिस्टमों के मेमोरी डंप को विश्लेषण करने की क्षमता प्रदान करता है।

### वोलेटिलिटी के उपयोग

वोलेटिलिटी का उपयोग डिजिटल फोरेंसिक्स और मेमोरी फंडा विशेषज्ञों द्वारा विभिन्न कार्यों के लिए किया जाता है, जैसे:

- अद्यतन और विश्लेषण के लिए विंडोज, लिनक्स और मैक मेमोरी डंप का उपयोग करना।
- प्रक्रिया और थ्रेड की जांच करने के लिए विंडोज, लिनक्स और मैक मेमोरी डंप का उपयोग करना।
- नेटवर्क विश्लेषण के लिए विंडोज, लिनक्स और मैक मेमोरी डंप का उपयोग करना।
- रजिस्ट्री और फ़ाइल सिस्टम के विश्लेषण के लिए विंडोज मेमोरी डंप का उपयोग करना।

### वोलेटिलिटी कमांड्स

यहां कुछ वोलेटिलिटी कमांड्स हैं जो आपको विभिन्न कार्यों के लिए उपयोगी साबित हो सकते हैं:

- `imageinfo`: मेमोरी डंप की जानकारी प्रदान करता है।
- `pslist`: चल रही प्रक्रियाओं की सूची प्रदान करता है।
- `pstree`: प्रक्रिया वृक्ष प्रदान करता है।
- `dlllist`: प्रक्रियाओं के लिए लोड किए गए DLL की सूची प्रदान करता है।
- `handles`: प्रक्रियाओं के लिए हैंडल की सूची प्रदान करता है।
- `filescan`: फ़ाइल सिस्टम में फ़ाइलों की खोज करता है।
- `netscan`: नेटवर्क कनेक्शन की सूची प्रदान करता है।

### वोलेटिलिटी प्लगइन्स

वोलेटिलिटी के साथ कई प्लगइन्स उपलब्ध हैं जो विशेष कार्यों के लिए उपयोगी हो सकते हैं। यहां कुछ प्लगइन्स की सूची है:

- `malfind`: संदेहास्पद या अवैध प्रक्रियाओं की खोज करता है।
- `svcscan`: सेवा की सूची प्रदान करता है।
- `connscan`: नेटवर्क कनेक्शन की सूची प्रदान करता है।
- `cmdline`: प्रक्रियाओं की कमांड लाइन विवरण प्रदान करता है।
- `vadinfo`: वर्चुअल एड्रेस स्पेस की जानकारी प्रदान करता है।
- `modscan`: लोड किए गए मॉड्यूलों की सूची प्रदान करता है।

### वोलेटिलिटी इंस्टॉलेशन

वोलेटिलिटी को इंस्टॉल करने के लिए निम्नलिखित चरणों का पालन करें:

1. पायथन (Python) इंस्टॉल करें।
2. पायथन पैकेज प्रबंधक (Python Package Manager) को इंस्टॉल करें।
3. वोलेटिलिटी को डाउनलोड करें और इंस्टॉल करें।

### वोलेटिलिटी उपयोग करना

वोलेटिलिटी का उपयोग करने के लिए निम्नलिखित चरणों का पालन करें:

1. टर्मिनल खोलें और वोलेटिलिटी को चलाने के लिए निम्नलिखित कमांड दर्ज करें:

```bash
volatility
```

2. वोलेटिलिटी कमांड प्रारंभ हो जाएगा और आपको प्राथमिक प्रदर्शन दिखाई देगा।

### वोलेटिलिटी रिपोर्ट

वोलेटिलिटी का उपयोग करके रिपोर्ट बनाने के लिए निम्नलिखित चरणों का पालन करें:

1. टर्मिनल खोलें और वोलेटिलिटी को चलाने के लिए निम्नलिखित कमांड दर्ज करें:

```bash
volatility
```

2. वोलेटिलिटी कमांड प्रारंभ हो जाएगा।

3. वोलेटिलिटी कमांड प्रारंभ होने के बाद, आप विभिन्न कमांड्स का उपयोग करके रिपोर्ट बना सकते हैं। उदाहरण के लिए, निम्नलिखित कमांड द्वारा प्रक्रियाओं की सूची प्रदर्शित करें:

```bash
volatility pslist
```

4. रिपोर्ट दिखाई देगा और आप उसे अपनी पसंद के अनुसार सहेज सकते हैं।

### वोलेटिलिटी स्क्रिप्ट

वोलेटिलिटी का उपयोग करके स्क्रिप्ट बनाने के लिए निम्नलिखित चरणों का पालन करें:

1. टर्मिनल खोलें और वोलेटिलिटी को चलाने के लिए निम्नलिखित कमांड दर्ज करें:

```bash
volatility
```

2. वोलेटिलिटी कमांड प्रारंभ हो जाएगा।

3. वोलेटिलिटी कमांड प्रारंभ होने के बाद, आप विभिन्न कमांड्स का उपयोग करके स्क्रिप्ट बना सकते हैं। उदाहरण के लिए, निम्नलिखित कमांड द्वारा प्रक्रियाओं की सूची प्रदर्शित करें:

```bash
volatility pslist
```

4. स्क्रिप्ट दिखाई देगा और आप उसे अपनी पसंद के अनुसार सहेज सकते हैं।

### वोलेटिलिटी संसाधन

वोलेटिलिटी के लिए निम्नलिखित संसाधन उपलब्ध हैं:

- [वोलेटिलिटी वेबसाइट](https://www.volatilityfoundation.org/)
- [वोलेटिलिटी गिटहब रिपोजिटरी](https://github.com/volatilityfoundation/volatility)
```bash
volatilitye --plugins="/tmp/plugins/" [...]
```
{% endtab %}
{% endtabs %}

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
## वोलेटिलिटी चीटशीट

यह चीटशीट वोलेटिलिटी टूल के लिए एक संक्षेप मार्गदर्शिका है। यह आपको वोलेटिलिटी टूल के विभिन्न आदेशों और फ़ंक्शन के बारे में जानकारी प्रदान करेगा जो मेमोरी डंप विश्लेषण के लिए उपयोगी हो सकती हैं।

### वोलेटिलिटी टूल की शुरुआत करना

वोलेटिलिटी टूल को शुरू करने के लिए निम्नलिखित आदेश का उपयोग करें:

```bash
volatility
```

### उपयोगी आदेश

#### जीनेरिक आदेश

- **imageinfo**: मेमोरी डंप की जानकारी प्रदान करता है।
- **kdbgscan**: डंप के लिए KDBG तालिका की खोज करता है।
- **kpcrscan**: डंप के लिए KPCR तालिका की खोज करता है।
- **pslist**: सिस्टम प्रक्रियाओं की सूची प्रदान करता है।
- **pstree**: प्रक्रिया पेड़ का विवरण प्रदान करता है।
- **dlllist**: प्रक्रियाओं के लिए DLL सूची प्रदान करता है।
- **handles**: प्रक्रियाओं के लिए हैंडल सूची प्रदान करता है।
- **cmdline**: प्रक्रियाओं के लिए कमांड लाइन प्रदान करता है।
- **filescan**: फ़ाइलों की सूची प्रदान करता है।
- **malfind**: संदिग्ध मेमोरी क्षेत्रों की खोज करता है।
- **svcscan**: सेवा तालिका की खोज करता है।
- **connections**: नेटवर्क कनेक्शन की सूची प्रदान करता है।
- **connscan**: नेटवर्क कनेक्शन की खोज करता है।
- **netscan**: नेटवर्क इंटरफ़ेस की खोज करता है।
- **modscan**: लोड किए गए मॉड्यूलों की सूची प्रदान करता है।
- **ssdt**: SSDT तालिका की खोज करता है।
- **gdt**: GDT तालिका की खोज करता है।
- **idt**: IDT तालिका की खोज करता है।
- **ldrmodules**: लोड किए गए मॉड्यूलों की सूची प्रदान करता है।
- **apihooks**: API हुक की सूची प्रदान करता है।
- **gdit**: GDI तालिका की खोज करता है।
- **atomscan**: एटम तालिका की खोज करता है।
- **ssdt**: SSDT तालिका की खोज करता है।
- **gdt**: GDT तालिका की खोज करता है।
- **idt**: IDT तालिका की खोज करता है।
- **ldrmodules**: लोड किए गए मॉड्यूलों की सूची प्रदान करता है।
- **apihooks**: API हुक की सूची प्रदान करता है।
- **gdit**: GDI तालिका की खोज करता है।
- **atomscan**: एटम तालिका की खोज करता है।

#### विंडोज आदेश

- **hivelist**: रजिस्ट्री हाइव की सूची प्रदान करता है।
- **printkey**: रजिस्ट्री कुंजी की जानकारी प्रदान करता है।
- **hashdump**: पासवर्ड हैश की खोज करता है।
- **userassist**: उपयोगकर्ता सहायता तालिका की खोज करता है।
- **shellbags**: शैलबैग्स की खोज करता है।
- **mftparser**: MFT तालिका की खोज करता है।
- **usnparser**: USN तालिका की खोज करता है।
- **eventlogs**: ईवेंट लॉग की सूची प्रदान करता है।
- **evtlogs**: ईवेंट लॉग की खोज करता है।
- **iehistory**: इंटरनेट इक्का इतिहास की खोज करता है।
- **prefetchparser**: प्रीफ़ेच फ़ाइल की खोज करता है।
- **shimcacheparser**: शिम कैश तालिका की खोज करता है।
- **mbrparser**: MBR की खोज करता है।
- **yarascan**: यारा नियमों के आधार पर संदिग्ध फ़ाइलों की खोज करता है।
- **vadinfo**: VAD तालिका की जानकारी प्रदान करता है।
- **vaddump**: VAD तालिका की खोज करता है और उसे डंप करता है।
- **vadtree**: VAD तालिका की वृक्षणीय जानकारी प्रदान करता है।
- **vadwalk**: VAD तालिका की खोज करता है और उसे वॉक करता है।
- **vadinfo**: VAD तालिका की जानकारी प्रदान करता है।
- **vaddump**: VAD तालिका की खोज करता है और उसे डंप करता है।
- **vadtree**: VAD तालिका की वृक्षणीय जानकारी प्रदान करता है।
- **vadwalk**: VAD तालिका की खोज करता है और उसे वॉक करता है।

### वोलेटिलिटी टूल के लिए उपयोगी संसाधन

- [वोलेटिलिटी टूल का आधिकारिक डॉक्यूमेंटेशन](https://www.volatilityfoundation.org/)
- [वोलेटिलिटी टूल का GitHub रेपो](https://github.com/volatilityfoundation/volatility)
- [वोलेटिलिटी टूल के लिए विस्तृत ट्यूटोरियल](https://www.andreafortuna.org/2019/03/28/volatility-2-6-tutorial-dump-and-analyze-a-malware-infected-memory-part-1/)
- [वोलेटिलिटी टूल के लिए विस्तृत ट्यूटोरियल](https://www.andreafortuna.org/2019/03/28/volatility-2-6-tutorial-dump-and-analyze-a-malware-infected-memory-part-2/)

### वोलेटिलिटी टूल के लिए अन्य चीटशीट

- [वोलेटिलिटी टूल के लिए अन्य चीटशीट](https://github.com/sans-dfir/sift-cheatsheet/blob/master/cheatsheets/Volatility-Commands.pdf)
```bash
volatility --profile=Win7SP1x86_23418 mutantscan -f file.dmp
volatility --profile=Win7SP1x86_23418 -f file.dmp handles -p <PID> -t mutant
```
{% endtab %}
{% endtabs %}

### सिमलिंक्स

{% tabs %}
{% tab title="vol3" %}
```bash
./vol.py -f file.dmp windows.symlinkscan.SymlinkScan
```
## वोलेटिलिटी चीटशीट

### वोलेटिलिटी क्या है?

वोलेटिलिटी एक खुला स्रोत (open-source) रूपांतरण उपकरण है जिसे डिजिटल फोरेंसिक्स और मेमोरी फंडा विशेषज्ञों द्वारा उपयोग किया जाता है। यह उपकरण विंडोज, लिनक्स और मैक ऑपरेटिंग सिस्टम पर काम करता है और इन सिस्टमों के मेमोरी डंप को विश्लेषण करने की क्षमता प्रदान करता है।

### वोलेटिलिटी के उपयोग

वोलेटिलिटी का उपयोग डिजिटल फोरेंसिक्स और मेमोरी फंडा विशेषज्ञों द्वारा विभिन्न कार्यों के लिए किया जाता है, जैसे:

- अद्यतन और विश्लेषण के लिए विंडोज, लिनक्स और मैक मेमोरी डंप का उपयोग करना।
- प्रक्रिया और थ्रेड की जांच करने के लिए विंडोज, लिनक्स और मैक मेमोरी डंप का उपयोग करना।
- नेटवर्क विश्लेषण के लिए पैकेट कैप्चर फ़ाइल का उपयोग करना।
- रजिस्ट्री और फ़ाइल सिस्टम की जांच करने के लिए विंडोज मेमोरी डंप का उपयोग करना।

### वोलेटिलिटी के फायदे

वोलेटिलिटी के उपयोग से निम्नलिखित फायदे हो सकते हैं:

- विंडोज, लिनक्स और मैक मेमोरी डंप का विश्लेषण करने की क्षमता।
- प्रक्रिया और थ्रेड की जांच करने की क्षमता।
- नेटवर्क विश्लेषण के लिए पैकेट कैप्चर फ़ाइल का उपयोग करने की क्षमता।
- रजिस्ट्री और फ़ाइल सिस्टम की जांच करने की क्षमता।

### वोलेटिलिटी के उपकरण

वोलेटिलिटी के उपकरण निम्नलिखित हो सकते हैं:

- `volatility`: यह वोलेटिलिटी का मुख्य उपकरण है और विंडोज, लिनक्स और मैक मेमोरी डंप का विश्लेषण करने के लिए उपयोग किया जाता है।
- `volshell`: यह वोलेटिलिटी का एक अतिरिक्त उपकरण है जिसे इंटरैक्टिव मोड में उपयोग किया जा सकता है।
- `volshell`: यह वोलेटिलिटी का एक अतिरिक्त उपकरण है जिसे इंटरैक्टिव मोड में उपयोग किया जा सकता है।

### वोलेटिलिटी के बेसिक कमांड्स

वोलेटिलिटी के कुछ बेसिक कमांड्स निम्नलिखित हो सकते हैं:

- `imageinfo`: मेमोरी डंप की जानकारी प्रदान करता है।
- `pslist`: चल रही प्रक्रियाओं की सूची प्रदान करता है।
- `pstree`: प्रक्रिया वृक्ष प्रदान करता है।
- `dlllist`: प्रक्रियाओं के लिए लोड किए गए DLL की सूची प्रदान करता है।
- `handles`: प्रक्रियाओं के लिए हैंडल की सूची प्रदान करता है।
- `cmdline`: प्रक्रियाओं के लिए कमांड लाइन प्रदान करता है।
- `filescan`: फ़ाइल सिस्टम में फ़ाइलों की खोज करता है।
- `netscan`: नेटवर्क विश्लेषण के लिए नेटवर्क कनेक्शन की सूची प्रदान करता है।

### वोलेटिलिटी के उपयोगी रेफरेंस

वोलेटिलिटी के उपयोगी रेफरेंस निम्नलिखित हो सकते हैं:

- [वोलेटिलिटी डॉक्यूमेंटेशन](https://github.com/volatilityfoundation/volatility/wiki)
- [वोलेटिलिटी चीटशीट](https://github.com/sans-dfir/sift-cheatsheet/blob/master/cheatsheets/Volatility%20Cheatsheet.pdf)

### वोलेटिलिटी के उपयोगी लिंक्स

वोलेटिलिटी के उपयोगी लिंक्स निम्नलिखित हो सकते हैं:

- [वोलेटिलिटी डाउनलोड](https://www.volatilityfoundation.org/releases)
- [वोलेटिलिटी संग्रहालय](https://github.com/volatilityfoundation/community)

### वोलेटिलिटी के उपयोगी वीडियो

वोलेटिलिटी के उपयोगी वीडियो निम्नलिखित हो सकते हैं:

- [वोलेटिलिटी का उपयोग करके डिजिटल फोरेंसिक्स](https://www.youtube.com/watch?v=3kEfedtQVOY)
- [वोलेटिलिटी का उपयोग करके मेमोरी फंडा](https://www.youtube.com/watch?v=3kEfedtQVOY)
```bash
volatility --profile=Win7SP1x86_23418 -f file.dmp symlinkscan
```
{% endtab %}
{% endtabs %}

### बैश

यह संभव है कि आप **मेमोरी से बैश इतिहास पढ़ सकते हैं।** आप यह भी कर सकते हैं कि आप _.bash\_history_ फ़ाइल को डंप करें, लेकिन यदि यह अक्षम है तो आप इस volatility मॉड्यूल का उपयोग कर सकते हैं।
```
./vol.py -f file.dmp linux.bash.Bash
```
## वोलेटिलिटी चीटशीट

### वोलेटिलिटी क्या है?

वोलेटिलिटी एक खुला स्रोत (open-source) रूपांतरण उपकरण है जिसे डिजिटल फोरेंसिक्स और मेमोरी फंडा विशेषज्ञों द्वारा उपयोग किया जाता है। यह उपकरण विंडोज, लिनक्स और मैक ऑपरेटिंग सिस्टम पर काम करता है और इन सिस्टमों के मेमोरी डंप को विश्लेषण करने की क्षमता प्रदान करता है।

### वोलेटिलिटी के उपयोग

वोलेटिलिटी का उपयोग डिजिटल फोरेंसिक्स और मेमोरी फंडा विशेषज्ञों द्वारा विभिन्न कार्यों के लिए किया जाता है, जैसे:

- अद्यतन और विश्लेषण के लिए विंडोज, लिनक्स और मैक मेमोरी डंप का उपयोग करना।
- प्रक्रिया और थ्रेड की जांच करने के लिए विंडोज, लिनक्स और मैक मेमोरी डंप का उपयोग करना।
- नेटवर्क विश्लेषण के लिए पैकेट कैप्चर फ़ाइल का उपयोग करना।
- रजिस्ट्री और फ़ाइल सिस्टम की जांच करने के लिए विंडोज मेमोरी डंप का उपयोग करना।

### वोलेटिलिटी के फायदे

वोलेटिलिटी के उपयोग से निम्नलिखित फायदे हो सकते हैं:

- विंडोज, लिनक्स और मैक मेमोरी डंप का विश्लेषण करने की क्षमता।
- प्रक्रिया और थ्रेड की जांच करने की क्षमता।
- नेटवर्क विश्लेषण के लिए पैकेट कैप्चर फ़ाइल का उपयोग करने की क्षमता।
- रजिस्ट्री और फ़ाइल सिस्टम की जांच करने की क्षमता।

### वोलेटिलिटी के उपकरण

वोलेटिलिटी के उपकरण निम्नलिखित हो सकते हैं:

- `volatility`: यह वोलेटिलिटी का मुख्य उपकरण है और विंडोज, लिनक्स और मैक मेमोरी डंप का विश्लेषण करने के लिए उपयोग किया जाता है।
- `volshell`: यह वोलेटिलिटी का एक अतिरिक्त उपकरण है जो विंडोज, लिनक्स और मैक मेमोरी डंप के लिए एक इंटरैक्टिव शेल प्रदान करता है।
- `volshell`: यह वोलेटिलिटी का एक अतिरिक्त उपकरण है जो विंडोज, लिनक्स और मैक मेमोरी डंप के लिए एक इंटरैक्टिव शेल प्रदान करता है।

### वोलेटिलिटी कमांड्स

वोलेटिलिटी के कुछ महत्वपूर्ण कमांड्स निम्नलिखित हो सकते हैं:

- `imageinfo`: मेमोरी डंप की जानकारी प्रदान करता है।
- `pslist`: चल रही प्रक्रियाओं की सूची प्रदान करता है।
- `pstree`: प्रक्रिया वृक्ष प्रदान करता है।
- `dlllist`: प्रक्रियाओं के लिए लोड किए गए DLL की सूची प्रदान करता है।
- `handles`: हैंडल की सूची प्रदान करता है।
- `cmdline`: प्रक्रियाओं की कमांड लाइन प्रदान करता है।
- `filescan`: फ़ाइलों की सूची प्रदान करता है।
- `netscan`: नेटवर्क कनेक्शन की सूची प्रदान करता है।
- `connections`: नेटवर्क कनेक्शन की सूची प्रदान करता है।
- `malfind`: संदेहास्पद या अवैध कार्यों की खोज करता है।
- `dumpfiles`: फ़ाइलों को डंप करता है।
- `hivelist`: रजिस्ट्री हाइव की सूची प्रदान करता है।
- `printkey`: रजिस्ट्री कुंजी की जानकारी प्रदान करता है।
- `hashdump`: पासवर्ड हैश की जानकारी प्रदान करता है।
- `mbrparser`: मास्टर बूट रिकॉर्ड (MBR) का विश्लेषण करता है।
- `ssdt`: SSDT (System Service Descriptor Table) की सूची प्रदान करता है।
- `gdt`: GDT (Global Descriptor Table) की सूची प्रदान करता है।
- `idt`: IDT (Interrupt Descriptor Table) की सूची प्रदान करता है।
- `ldrmodules`: लोड किए गए मॉड्यूल की सूची प्रदान करता है।
- `modscan`: लोड किए गए मॉड्यूल की सूची प्रदान करता है।
- `ssdt`: SSDT (System Service Descriptor Table) की सूची प्रदान करता है।
- `gdt`: GDT (Global Descriptor Table) की सूची प्रदान करता है।
- `idt`: IDT (Interrupt Descriptor Table) की सूची प्रदान करता है।
- `ldrmodules`: लोड किए गए मॉड्यूल की सूची प्रदान करता है।
- `modscan`: लोड किए गए मॉड्यूल की सूची प्रदान करता है।

### वोलेटिलिटी के उपयोग की प्रक्रिया

वोलेटिलिटी के उपयोग की प्रक्रिया निम्नलिखित हो सकती है:

1. वोलेटिलिटी उपकरण को डाउनलोड और स्थापित करें।
2. वोलेटिलिटी के उपयोग के लिए आवश्यक डेटा को प्राप्त करें, जैसे मेमोरी डंप फ़ाइल या पैकेट कैप्चर फ़ाइल।
3. वोलेटिलिटी कमांड्स का उपयोग करके डेटा का विश्लेषण करें।
4. प्राप्त की गई जानकारी का उपयोग करके आपत्तियों की पहचान करें और सुरक्षा की कमीज़ को ठीक करें।

### वोलेटिलिटी के लिए संदर्भ

वोलेटिलिटी के लिए निम्नलिखित संदर्भ उपयोगी हो सकते हैं:

- [वोलेटिलिटी डॉक्यूमेंटेशन](https://www.volatilityfoundation.org/)
- [वोलेटिलिटी गिटहब रेपो](https://github.com/volatilityfoundation/volatility)
```
volatility --profile=Win7SP1x86_23418 -f file.dmp linux_bash
```
{% endtab %}
{% endtabs %}

### टाइमलाइन

{% tabs %}
{% tab title="vol3" %}
```bash
./vol.py -f file.dmp timeLiner.TimeLiner
```
## वोलेटिलिटी चीटशीट

### वोलेटिलिटी क्या है?

वोलेटिलिटी एक खुला स्रोत (open-source) रूपांतरण उपकरण है जिसे डिजिटल फोरेंसिक्स और मेमोरी फंडा विशेषज्ञों द्वारा उपयोग किया जाता है। यह उपकरण विंडोज, लिनक्स और मैक ऑपरेटिंग सिस्टम पर काम करता है और इन सिस्टमों के मेमोरी डंप को विश्लेषण करने की क्षमता प्रदान करता है।

### वोलेटिलिटी के उपयोग

वोलेटिलिटी का उपयोग डिजिटल फोरेंसिक्स और मेमोरी फंडा विशेषज्ञों द्वारा विभिन्न कार्यों के लिए किया जाता है, जैसे:

- अद्यतन और विश्लेषण के लिए विंडोज, लिनक्स और मैक मेमोरी डंप का उपयोग करना।
- प्रक्रिया और थ्रेड की जांच करने के लिए विंडोज, लिनक्स और मैक मेमोरी डंप का उपयोग करना।
- नेटवर्क विश्लेषण के लिए पैकेट कैप्चर फ़ाइल का उपयोग करना।
- रजिस्ट्री और फ़ाइल सिस्टम की जांच करने के लिए विंडोज मेमोरी डंप का उपयोग करना।

### वोलेटिलिटी के फायदे

वोलेटिलिटी के उपयोग से निम्नलिखित फायदे हो सकते हैं:

- विंडोज, लिनक्स और मैक मेमोरी डंप का विश्लेषण करने की क्षमता।
- प्रक्रिया और थ्रेड की जांच करने की क्षमता।
- नेटवर्क विश्लेषण के लिए पैकेट कैप्चर फ़ाइल का उपयोग करने की क्षमता।
- रजिस्ट्री और फ़ाइल सिस्टम की जांच करने की क्षमता।

### वोलेटिलिटी के उपकरण

वोलेटिलिटी के उपकरण निम्नलिखित हो सकते हैं:

- `volatility`: यह वोलेटिलिटी का मुख्य उपकरण है और विंडोज, लिनक्स और मैक मेमोरी डंप का विश्लेषण करने के लिए उपयोग किया जाता है।
- `volshell`: यह वोलेटिलिटी का एक अतिरिक्त उपकरण है जिसे इंटरैक्टिव मोड में उपयोग किया जा सकता है।
- `volshell`: यह वोलेटिलिटी का एक अतिरिक्त उपकरण है जिसे इंटरैक्टिव मोड में उपयोग किया जा सकता है।

### वोलेटिलिटी के बेसिक कमांड्स

वोलेटिलिटी के बेसिक कमांड्स निम्नलिखित हो सकते हैं:

- `imageinfo`: मेमोरी डंप की जानकारी प्रदान करता है।
- `pslist`: चल रही प्रक्रियाओं की सूची प्रदान करता है।
- `pstree`: प्रक्रिया वृक्ष प्रदान करता है।
- `dlllist`: प्रक्रियाओं के लिए लोड किए गए DLL की सूची प्रदान करता है।
- `handles`: प्रक्रियाओं के लिए हैंडल की सूची प्रदान करता है।
- `cmdline`: प्रक्रियाओं के लिए कमांड लाइन प्रदान करता है।
- `filescan`: फ़ाइल सिस्टम में फ़ाइलों की खोज करता है।
- `netscan`: नेटवर्क विश्लेषण के लिए नेटवर्क कनेक्शन की सूची प्रदान करता है।
- `connections`: नेटवर्क कनेक्शन की सूची प्रदान करता है।
- `malfind`: संदिग्ध मेमोरी क्षेत्रों की खोज करता है।
- `dumpfiles`: फ़ाइलों को मेमोरी से निकालता है।
- `hivelist`: रजिस्ट्री हाइव की सूची प्रदान करता है।
- `printkey`: रजिस्ट्री कुंजी की जानकारी प्रदान करता है।
- `hashdump`: पासवर्ड हैश की जानकारी प्रदान करता है।
- `mbrparser`: मास्टर बूट रिकॉर्ड (MBR) का विश्लेषण करता है।
- `ssdt`: सिस्टम सर्विस टेबल (SSDT) की सूची प्रदान करता है।
- `driverscan`: लोड किए गए ड्राइवरों की सूची प्रदान करता है।
- `modscan`: लोड किए गए मॉड्यूलों की सूची प्रदान करता है।
- `ssdt`: सिस्टम सर्विस टेबल (SSDT) की सूची प्रदान करता है।
- `driverscan`: लोड किए गए ड्राइवरों की सूची प्रदान करता है।
- `modscan`: लोड किए गए मॉड्यूलों की सूची प्रदान करता है।

### वोलेटिलिटी के उपयोग की विधि

वोलेटिलिटी के उपयोग की विधि निम्नलिखित हो सकती है:

1. वोलेटिलिटी को डाउनलोड करें और स्थापित करें।
2. वोलेटिलिटी के उपकरण को चलाएं और उपयोग करने के लिए उपयुक्त कमांड का उपयोग करें।
3. विश्लेषण के लिए उपयुक्त फ़ाइल या डायरेक्टरी का उपयोग करें।
4. प्राप्त जानकारी का विश्लेषण करें और आवश्यकता अनुसार कार्रवाई करें।

### वोलेटिलिटी के उपयोग की विधि का उदाहरण

वोलेटिलिटी के उपयोग की विधि का उदाहरण निम्नलिखित हो सकता है:

```bash
volatility -f memory.dmp imageinfo
volatility -f memory.dmp pslist
volatility -f memory.dmp pstree
volatility -f memory.dmp dlllist
volatility -f memory.dmp handles
volatility -f memory.dmp cmdline
volatility -f memory.dmp filescan
volatility -f memory.dmp netscan
volatility -f memory.dmp connections
volatility -f memory.dmp malfind
volatility -f memory.dmp dumpfiles
volatility -f memory.dmp hivelist
volatility -f memory.dmp printkey
volatility -f memory.dmp hashdump
volatility -f memory.dmp mbrparser
volatility -f memory.dmp ssdt
volatility -f memory.dmp driverscan
volatility -f memory.dmp modscan
```

### वोलेटिलिटी के उपयोग की विधि के उदाहरण का आउटपुट

वोलेटिलिटी के उपयोग की विधि के उदाहरण का आउटपुट निम्नलिखित हो सकता है:

```bash
Volatility Foundation Volatility Framework 2.6.1
INFO    : volatility.debug    : Determining profile based on KDBG search...
          Suggested Profile(s) : Win7SP1x86_23418, Win7SP0x86, Win7SP1x86
                     AS Layer1 : IA32PagedMemoryPae (Kernel AS)
                     AS Layer2 : FileAddressSpace (/home/user/memory.dmp)
                      PAE type : PAE
                           DTB : 0x185000L
                          KDBG : 0x82925be8L
          Number of Processors : 1
     Image Type (Service Pack) : 1
                KPCR for CPU 0 : 0x82926c00L
             KUSER_SHARED_DATA : 0xffdf0000L
           Image date and time : 2021-01-01 00:00:00 UTC+0000
     Image local date and time : 2021-01-01 00:00:00 +0000
INFO    : volatility.debug    : Using system-wide physical memory range(s) from config file
Volatility Foundation Volatility Framework 2.6.1
Offset(V)  Name                    PID   PPID   Thds     Hnds   Sess  Wow64 Start                          Exit
---------- -------------------- ------ ------ ------ -------- ------ ------ ------------------------------ ------------------------------
0x823f6da0 System                    4      0     59      271 ------      0 2021-01-01 00:00:00 UTC+0000
0x823f6b28 smss.exe                276      4      2       29 ------      0 2021-01-01 00:00:00 UTC+0000
0x823f1da0 csrss.exe               352    344      9      358      0      0 2021-01-01 00:00:00 UTC+0000
0x823f0da0 wininit.exe             400    344      3       78      0      0 2021-01-01 00:00:00 UTC+0000
0x823f6da0 System                    4      0     59      271 ------      0 2021-01-01 00:00:00 UTC+0000
0x823f6b28 smss.exe                276      4      2       29 ------      0 2021-01-01 00:00:00 UTC+0000
0x823f1da0 csrss.exe               352    344      9      358      0      0 2021-01-01 00:00:00 UTC+0000
0x823f0da0 wininit.exe             400    344      3       78      0      0 2021-01-01 00:00:00 UTC+0000
0x823
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
## Volatility चीटशीट

यहां आपको Volatility टूल के लिए एक चीटशीट मिलेगी। यह चीटशीट आपको विभिन्न मेमोरी डंप फ़ाइलों का विश्लेषण करने के लिए उपयोगी आदेशों की जानकारी प्रदान करेगी।

### विश्लेषण के लिए आदेश

#### जीनेरिक आदेश

- `imageinfo` - इमेज की जानकारी प्रदान करता है
- `pslist` - प्रक्रियाओं की सूची प्रदान करता है
- `pstree` - प्रक्रिया पेड़ प्रदान करता है
- `psscan` - छिपी हुई प्रक्रियाओं की सूची प्रदान करता है
- `dlllist` - प्रक्रियाओं के लिए DLL सूची प्रदान करता है
- `handles` - हैंडल्स की सूची प्रदान करता है
- `filescan` - फ़ाइलों की सूची प्रदान करता है
- `cmdline` - प्रक्रियाओं की कमांड लाइन प्रदान करता है
- `consoles` - कंसोल्स की सूची प्रदान करता है
- `vadinfo` - वर्चुअल एड्रेस स्पेस की जानकारी प्रदान करता है
- `vadtree` - वर्चुअल एड्रेस स्पेस का पेड़ प्रदान करता है
- `vaddump` - वर्चुअल एड्रेस स्पेस को डंप करता है
- `vadwalk` - वर्चुअल एड्रेस स्पेस की सूची प्रदान करता है
- `vadtree` - वर्चुअल एड्रेस स्पेस का पेड़ प्रदान करता है
- `vaddump` - वर्चुअल एड्रेस स्पेस को डंप करता है
- `vadwalk` - वर्चुअल एड्रेस स्पेस की सूची प्रदान करता है

#### नेटवर्क आदेश

- `connections` - संबंधों की सूची प्रदान करता है
- `connscan` - खुली हुई संबंधों की सूची प्रदान करता है
- `sockets` - सॉकेट की सूची प्रदान करता है
- `sockscan` - खुली हुई सॉकेट की सूची प्रदान करता है
- `netscan` - नेटवर्क विवरण प्रदान करता है

#### रजिस्ट्री आदेश

- `hivelist` - रजिस्ट्री हाइव की सूची प्रदान करता है
- `printkey` - रजिस्ट्री कुंजी की जानकारी प्रदान करता है
- `printval` - रजिस्ट्री मान की जानकारी प्रदान करता है
- `hashdump` - रजिस्ट्री हाश डंप करता है

#### अन्य आदेश

- `malfind` - संदेहास्पद वस्तुओं की सूची प्रदान करता है
- `modscan` - लोड किए गए मॉड्यूलों की सूची प्रदान करता है
- `ldrmodules` - लोड किए गए मॉड्यूलों की सूची प्रदान करता है
- `ssdt` - SSDT की सूची प्रदान करता है
- `gdt` - GDT की सूची प्रदान करता है
- `idt` - IDT की सूची प्रदान करता है
- `callbacks` - कॉलबैक की सूची प्रदान करता है
- `driverirp` - ड्राइवर IRP की सूची प्रदान करता है
- `devicetree` - डिवाइस ट्री की सूची प्रदान करता है
- `devicetree` - डिवाइस ट्री की सूची प्रदान करता है
- `driverirp` - ड्राइवर IRP की सूची प्रदान करता है
- `callbacks` - कॉलबैक की सूची प्रदान करता है
- `idt` - IDT की सूची प्रदान करता है
- `gdt` - GDT की सूची प्रदान करता है
- `ssdt` - SSDT की सूची प्रदान करता है
- `ldrmodules` - लोड किए गए मॉड्यूलों की सूची प्रदान करता है
- `modscan` - लोड किए गए मॉड्यूलों की सूची प्रदान करता है
- `malfind` - संदेहास्पद वस्तुओं की सूची प्रदान करता है

### अन्य उपयोगी आदेश

- `memdump` - मेमोरी डंप करता है
- `memmap` - मेमोरी मैप की सूची प्रदान करता है
- `memstrings` - मेमोरी स्ट्रिंग्स की सूची प्रदान करता है
- `memdump` - मेमोरी डंप करता है
- `memmap` - मेमोरी मैप की सूची प्रदान करता है
- `memstrings` - मेमोरी स्ट्रिंग्स की सूची प्रदान करता है

### अन्य उपयोगी आदेश

- `memdump` - मेमोरी डंप करता है
- `memmap` - मेमोरी मैप की सूची प्रदान करता है
- `memstrings` - मेमोरी स्ट्रिंग्स की सूची प्रदान करता है
```bash
volatility --profile=Win7SP1x86_23418 -f file.dmp driverscan
```
{% endtab %}
{% endtabs %}

### क्लिपबोर्ड प्राप्त करें
```bash
#Just vol2
volatility --profile=Win7SP1x86_23418 clipboard -f file.dmp
```
### IE इतिहास प्राप्त करें

```plaintext
$ volatility -f <memory_dump> --profile=<profile> iehistory
```

इस आदेश का उपयोग करके, आप एक मेमोरी डंप फ़ाइल (`<memory_dump>`) और प्रोफ़ाइल (`<profile>`) के साथ Volatility टूल का उपयोग करके IE ब्राउज़र का इतिहास प्राप्त कर सकते हैं।
```bash
#Just vol2
volatility --profile=Win7SP1x86_23418 iehistory -f file.dmp
```
### नोटपैड टेक्स्ट प्राप्त करें

```bash
$ volatility -f memory_dump.mem notepad
```

यदि आप एक मेमोरी डंप फ़ाइल (`memory_dump.mem`) के साथ नोटपैड के टेक्स्ट को प्राप्त करना चाहते हैं, तो निम्नलिखित आदेश का उपयोग करें:

```bash
$ volatility -f memory_dump.mem notepad
```
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

The Master Boot Record (MBR) is a small section of a computer's hard drive that contains important information about the disk's partitions and the boot process. It is located in the first sector of the disk and is responsible for loading the operating system.

मास्टर बूट रिकॉर्ड (MBR) कंप्यूटर के हार्ड ड्राइव का एक छोटा सेक्शन है जिसमें डिस्क के पार्टीशन और बूट प्रक्रिया के बारे में महत्वपूर्ण जानकारी होती है। यह डिस्क के पहले सेक्टर में स्थित होता है और ऑपरेटिंग सिस्टम को लोड करने के लिए जिम्मेदार होता है।

### Volatile Memory Analysis

Volatile memory analysis is a technique used in digital forensics to extract and analyze information from a computer's volatile memory (RAM). Volatile memory contains data that is lost when the computer is powered off or restarted, making it a valuable source of evidence in forensic investigations.

वॉलेटाइल मेमोरी विश्लेषण एक तकनीक है जिसका उपयोग डिजिटल फोरेंसिक्स में किया जाता है ताकि कंप्यूटर की वॉलेटाइल मेमोरी (रैम) से जानकारी को निकाला और विश्लेषण किया जा सके। वॉलेटाइल मेमोरी में डेटा होता है जो कंप्यूटर को पावर ऑफ करने या रीस्टार्ट करने पर खो जाता है, जिससे यह फोरेंसिक जांचों में सबूत का मूल्यवान स्रोत बनती है।

### Memory Dump Analysis

Memory dump analysis is the process of examining the contents of a computer's memory dump file. A memory dump file is created when a computer crashes or experiences a system failure. By analyzing the memory dump, forensic analysts can uncover valuable information such as running processes, open files, network connections, and potential malware.

मेमोरी डंप विश्लेषण एक प्रक्रिया है जिसमें कंप्यूटर की मेमोरी डंप फ़ाइल की सामग्री की जांच की जाती है। एक मेमोरी डंप फ़ाइल तब बनाई जाती है जब कंप्यूटर क्रैश होता है या सिस्टम में फेल होता है। मेमोरी डंप का विश्लेषण करके, फोरेंसिक विश्लेषक में चल रहे प्रक्रियाओं, खुले फ़ाइलों, नेटवर्क कनेक्शनों और संभावित मैलवेयर जैसी मूल्यवान जानकारी का पता लगा सकते हैं।
```
volatility --profile=Win7SP1x86_23418 mbrparser -f file.dmp
```
MBR में जानकारी होती है कि वहां कैसे तार्किक विभाजन, जिसमें [फ़ाइल सिस्टम](https://en.wikipedia.org/wiki/File\_system) होती है, उस माध्यम पर संगठित होती हैं। MBR में स्थापित ऑपरेटिंग सिस्टम के लिए लोडर के रूप में कार्य करने के लिए निष्पादन योग्य कोड भी होता है - आमतौर पर लोडर के [द्वितीय स्तर](https://en.wikipedia.org/wiki/Second-stage\_boot\_loader) को या प्रत्येक विभाजन के [वॉल्यूम बूट रिकॉर्ड](https://en.wikipedia.org/wiki/Volume\_boot\_record) (VBR) के साथ। इस MBR कोड को आमतौर पर [बूट लोडर](https://en.wikipedia.org/wiki/Boot\_loader) के रूप में संदर्भित किया जाता है। [यहां](https://en.wikipedia.org/wiki/Master\_boot\_record) से।

​

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

[**RootedCON**](https://www.rootedcon.com/) स्पेन में सबसे महत्वपूर्ण साइबर सुरक्षा घटना है और यूरोप में सबसे महत्वपूर्ण में से एक है। तकनीकी ज्ञान को बढ़ावा देने की मिशन के साथ, यह सम्मेलन प्रौद्योगिकी और साइबर सुरक्षा विशेषज्ञों के लिए एक उबलता हुआ मिलन स्थान है।

{% embed url="https://www.rootedcon.com/" %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण तक पहुंच या HackTricks को PDF में डाउनलोड करने की आवश्यकता है**? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को।**

</details>
