# दिलचस्प Windows रजिस्ट्री कुंजियाँ

## दिलचस्प Windows रजिस्ट्री कुंजियाँ

<details>

<summary><strong>Learn AWS hacking from zero to hero with</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **अपनी हैकिंग ट्रिक्स साझा करें, HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके।

</details>

## **Windows सिस्टम जानकारी**

### संस्करण

* **`Software\Microsoft\Windows NT\CurrentVersion`**: Windows संस्करण, सर्विस पैक, स्थापना समय और पंजीकृत मालिक

### होस्टनाम

* **`System\ControlSet001\Control\ComputerName\ComputerName`**: होस्टनाम

### समय क्षेत्र

* **`System\ControlSet001\Control\TimeZoneInformation`**: समय क्षेत्र

### अंतिम पहुँच समय

* **`System\ControlSet001\Control\Filesystem`**: अंतिम पहुँच समय (डिफ़ॉल्ट रूप से यह `NtfsDisableLastAccessUpdate=1` के साथ अक्षम है, यदि `0`, तो, यह सक्षम है)।
* इसे सक्षम करने के लिए: `fsutil behavior set disablelastaccess 0`

### शटडाउन समय

* `System\ControlSet001\Control\Windows`: शटडाउन समय
* `System\ControlSet001\Control\Watchdog\Display`: शटडाउन गणना (केवल XP)

### नेटवर्क जानकारी

* **`System\ControlSet001\Services\Tcpip\Parameters\Interfaces{GUID_INTERFACE}`**: नेटवर्क इंटरफेस
* **`Software\Microsoft\Windows NT\CurrentVersion\NetworkList\Signatures\Unmanaged` & `Software\Microsoft\Windows NT\CurrentVersion\NetworkList\Signatures\Managed` & `Software\Microsoft\Windows NT\CurrentVersion\NetworkList\Nla\Cache`**: नेटवर्क कनेक्शन का पहला और अंतिम समय और VPN के माध्यम से कनेक्शन
* **`Software\Microsoft\WZCSVC\Parameters\Interfaces{GUID}` (XP के लिए) & `Software\Microsoft\Windows NT\CurrentVersion\NetworkList\Profiles`**: नेटवर्क प्रकार (0x47-वायरलेस, 0x06-केबल, 0x17-3G) और श्रेणी (0-सार्वजनिक, 1-निजी/घर, 2-डोमेन/काम) और अंतिम कनेक्शन

### साझा फ़ोल्डर

* **`System\ControlSet001\Services\lanmanserver\Shares\`**: साझा फ़ोल्डर और उनकी कॉन्फ़िगरेशन। यदि **Client Side Caching** (CSCFLAGS) सक्षम है, तो, साझा फ़ाइलों की एक प्रति क्लाइंट्स और सर्वर में `C:\Windows\CSC` में सहेजी जाएगी
* CSCFlag=0 -> डिफ़ॉल्ट रूप से उपयोगकर्ता को उन फ़ाइलों को इंगित करना होता है जिन्हें वह कैश करना चाहता है
* CSCFlag=16 -> स्वचालित कैशिंग दस्तावेज़। "सभी फ़ाइलें और प्रोग्राम जो उपयोगकर्ता साझा फ़ोल्डर से खोलते हैं वे स्वचालित रूप से ऑफ़लाइन उपलब्ध होते हैं" "प्रदर्शन के लिए अनुकूलित" अनचेक के साथ।
* CSCFlag=32 -> पिछले विकल्पों की तरह "प्रदर्शन के लिए अनुकूलित" टिक किया गया है
* CSCFlag=48 -> कैश अक्षम है।
* CSCFlag=2048: यह सेटिंग केवल Win 7 & 8 पर है और यह "सरल फ़ाइल साझाकरण" को अक्षम करने या "उन्नत" साझाकरण विकल्प का उपयोग करने तक डिफ़ॉल्ट सेटिंग है। यह "Homegroup" के लिए भी डिफ़ॉल्ट सेटिंग प्रतीत होती है
* CSCFlag=768 -> यह सेटिंग केवल साझा प्रिंट उपकरणों पर देखी गई थी।

### ऑटोस्टार्ट प्रोग्राम

* `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Run`
* `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\RunOnce`
* `Software\Microsoft\Windows\CurrentVersion\Runonce`
* `Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\Run`
* `Software\Microsoft\Windows\CurrentVersion\Run`

### एक्सप्लोरर खोजें

* `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\WordwheelQuery`: उपयोगकर्ता ने एक्सप्लोरर/सहायक का उपयोग करके क्या खोजा। `MRU=0` वाली वस्तु अंतिम है।

### टाइप किए गए पथ

* `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\TypedPaths`: एक्सप्लोरर में टाइप किए गए पथ (केवल W10)

### हाल के दस्तावेज़

* `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs`: उपयोगकर्ता द्वारा खोले गए हाल के दस्तावेज़
* `NTUSER.DAT\Software\Microsoft\Office{Version}{Excel|Word}\FileMRU`:हाल के ऑफिस दस्तावेज़। संस्करण:
* 14.0 Office 2010
* 12.0 Office 2007
* 11.0 Office 2003
* 10.0 Office X
* `NTUSER.DAT\Software\Microsoft\Office{Version}{Excel|Word} UserMRU\LiveID_###\FileMRU`: हाल के ऑफिस दस्तावेज़। संस्करण:
* 15.0 office 2013
* 16.0 Office 2016

### MRUs

* `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\ComDlg32\LastVisitedMRU`
* `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\ComDlg32\LasVisitedPidlMRU`

यह इंगित करता है कि किस पथ से निष्पादन योग्य चलाया गया था

* `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\ComDlg32\Op enSaveMRU` (XP)
* `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\ComDlg32\Op enSavePidlMRU`

यह इंगित करता है कि खुली विंडो के अंदर कौन सी फ़ाइलें खोली गईं

### अंतिम चलाए गए कमांड

* `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\RunMRU`
* `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\Policies\RunMR`

### यूजर असिस्टकी

* `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist\{GUID}\Count`

GUID एप्लिकेशन की आईडी है। सहेजी गई डेटा:

* अंतिम चलाने का समय
* चलाने की गिनती
* GUI एप्लिकेशन का नाम (इसमें अब्स पथ और अधिक जानकारी होती है)
* फोकस समय और फोकस नाम

## Shellbags

जब आप एक निर्देशिका खोलते हैं, Windows रजिस्ट्री में निर्देशिका को देखने के तरीके के बारे में डेटा सहेजता है। ये प्रविष्टियाँ Shellbags के रूप में जानी जाती हैं।

एक्सप्लोरर पहुँच:

* `USRCLASS.DAT\Local Settings\Software\Microsoft\Windows\Shell\Bags`
* `USRCLASS.DAT\Local Settings\Software\Microsoft\Windows\Shell\BagMRU`

डेस्कटॉप पहुँच:

* `NTUSER.DAT\Software\Microsoft\Windows\Shell\BagMRU`
* `NTUSER.DAT\Software\Microsoft\Windows\Shell\Bags`

Shellbags का विश्लेषण करने के लिए आप [**Shellbag Explorer**](https://ericzimmerman.github.io/#!index.md) का उपयोग कर सकते हैं और आप फ़ोल्डर के **MAC समय** और शेलबैग की निर्माण तिथि और संशोधित तिथि पा सकते हैं जो फ़ोल्डर की **पहली बार और अंतिम बार** पहुँच से संबंधित हैं।

निम्नलिखित छवि से दो बातें नोट करें:

1. हमें **USB के फ़ोल्डरों के नाम** का पता चलता है जो **E:** में डाला गया था
2. हमें पता चलता है कि **शेलबैग कब बनाया गया और संशोधित किया गया** और फ़ोल्डर कब बनाया गया और पहुँचा गया

![](<../../../.gitbook/assets/image (475).png>)

## USB जानकारी

### डिवाइस जानकारी

रजिस्ट्री `HKLM\SYSTEM\ControlSet001\Enum\USBSTOR` पीसी से जुड़े हर USB डिवाइस की निगरानी करती है।\
इस रजिस्ट्री में निम्नलिखित पाया जा सकता है:

* निर्माता का नाम
* उत
