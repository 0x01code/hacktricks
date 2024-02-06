# दिलचस्प Windows रजिस्ट्री कुंजी

## दिलचस्प Windows रजिस्ट्री कुंजी

<details>

<summary><strong>शून्य से हीरो तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी कंपनी का विज्ञापन HackTricks में देखना चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं तो [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 पर **फॉलो** करें [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>

## **Windows सिस्टम जानकारी**

### संस्करण

* **`Software\Microsoft\Windows NT\CurrentVersion`**: Windows संस्करण, सेवा पैक, स्थापना समय और पंजीकृत मालिक

### होस्टनाम

* **`System\ControlSet001\Control\ComputerName\ComputerName`**: होस्टनाम

### समय क्षेत्र

* **`System\ControlSet001\Control\TimeZoneInformation`**: समय क्षेत्र

### अंतिम पहुंच समय

* **`System\ControlSet001\Control\Filesystem`**: अंतिम पहुंच समय (डिफ़ॉल्ट रूप से `NtfsDisableLastAccessUpdate=1` के साथ अक्षम है, यदि `0`, तो यह सक्षम है)।
* इसे सक्षम करने के लिए: `fsutil behavior set disablelastaccess 0`

### शटडाउन समय

* `System\ControlSet001\Control\Windows`: शटडाउन समय
* `System\ControlSet001\Control\Watchdog\Display`: शटडाउन गिनती (केवल XP)

### नेटवर्क सूचना

* **`System\ControlSet001\Services\Tcpip\Parameters\Interfaces{GUID_INTERFACE}`**: नेटवर्क इंटरफेसेस
* **`Software\Microsoft\Windows NT\CurrentVersion\NetworkList\Signatures\Unmanaged` & `Software\Microsoft\Windows NT\CurrentVersion\NetworkList\Signatures\Managed` & `Software\Microsoft\Windows NT\CurrentVersion\NetworkList\Nla\Cache`**: पहली और आखिरी बार नेटवर्क कनेक्शन किया गया था और VPN के माध्यम से कनेक्शन
* **`Software\Microsoft\WZCSVC\Parameters\Interfaces{GUID}` (XP के लिए) & `Software\Microsoft\Windows NT\CurrentVersion\NetworkList\Profiles`**: नेटवर्क प्रकार (0x47-वायरलेस, 0x06-केबल, 0x17-3G) और श्रेणी (0-सार्वजनिक, 1-निजी/घर, 2-डोमेन/काम) और अंतिम कनेक्शन

### साझा फोल्डर

* **`System\ControlSet001\Services\lanmanserver\Shares\`**: शेयर फोल्डर और उनकी विन्यास। यदि **क्लाइंट साइड कैशिंग** (CSCFLAGS) सक्षम है, तो, साझा फ़ाइलों की एक प्रतिलिपि क्लाइंट्स और सर्वर में `C:\Windows\CSC` में सहेजी जाएगी
* CSCFlag=0 -> डिफ़ॉल्ट रूप से उपयोगकर्ता को इसका संकेत देना होगा जिन फ़ाइलों को वह कैश करना चाहता है
* CSCFlag=16 -> स्वचालित कैशिंग दस्तावेज़। "उपयोगकर्ता द्वारा साझा फ़ोल्डर से खोली गई सभी फ़ाइलें ऑफ़लाइन उपलब्ध होती हैं" जिसमें "प्रदर्शन के लिए अनुकूलित करें" अचिन्हित है।
* CSCFlag=32 -> पिछले विकल्पों की तरह "प्रदर्शन के लिए अनुकूलित करें" टिका है
* CSCFlag=48 -> कैश अक्षम है।
* CSCFlag=2048: यह सेटिंग केवल विंडोज 7 और 8 पर है और जब तक आप "सिम्पल फ़ाइल सेयरिंग" को अक्षम नहीं करते या "उन्नत" साझा करने का विकल्प नहीं उपयोग करते हैं, तब तक डिफ़ॉल्ट सेटिंग है। यह "होमग्रुप" के लिए डिफ़ॉल्ट सेटिंग भी लगता है।
* CSCFlag=768 -> यह सेटिंग केवल साझा प्रिंट उपकरणों पर देखा गया था।

### ऑटोस्टार्ट प्रोग्राम

* `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Run`
* `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\RunOnce`
* `Software\Microsoft\Windows\CurrentVersion\Runonce`
* `Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\Run`
* `Software\Microsoft\Windows\CurrentVersion\Run`

### एक्सप्लोरर खोजें

* `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\WordwheelQuery`: उपयोगकर्ता द्वारा एक्सप्लोरर/हेल्पर का उपयोग करके खोजा गया क्या है। `MRU=0` वाला आइटम आखिरी है।

### टाइप किए गए पथ

* `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\TypedPaths`: एक्सप्लोरर में टाइप किए गए पथ (केवल W10)

### हाल के दस्तावेज़

* `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs`: उपयोगकर्ता द्वारा खोले गए हाल के दस्तावेज़
* `NTUSER.DAT\Software\Microsoft\Office{Version}{Excel|Word}\FileMRU`: हाल के ऑफिस दस्तावेज़। संस्करण:
* 14.0 ऑफिस 2010
* 12.0 ऑफिस 2007
* 11.0 ऑफिस 2003
* 10.0 ऑफिस X
* `NTUSER.DAT\Software\Microsoft\Office{Version}{Excel|Word} UserMRU\LiveID_###\FileMRU`: हाल के ऑफिस दस्तावेज़। संस्करण:
* 15.0 ऑफिस 2013
* 16.0 ऑफिस 2016

### MRUs

* `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\ComDlg32\LastVisitedMRU`
* `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\ComDlg32\LasVisitedPidlMRU`

कोड के पथ को दर्शाता है जहां से कार्यक्रम चलाया गया था

* `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\ComDlg32\Op enSaveMRU` (XP)
* `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\ComDlg32\Op enSavePidlMRU`

खिड़की के अंदर खोली गई फ़ाइलें दर्शाता है

### अंतिम रन कमांड्स

* `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\RunMRU`
* `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\Policies\RunMR`

### उपयोगकर्ता सहायकी

* `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist\{GUID}\Count`

GUID एप्लिकेशन की पहचान है। डेटा सहेजा गया है:

* अंतिम रन समय
* रन गिनती
* GUI एप्लिकेशन नाम (इसमें अब्स पथ और अधिक जानकारी शामिल है)
* फोकस समय और फोकस नाम

## शेलबैग्स

जब आप एक निर्देशिका खोलते हैं, तो Windows रजिस्ट्री में निर्देशिका को कैसे दिखाना है के बारे में डेटा सहेजता है। इन प्रविष्टियों को शेलबैग्स के रूप में जाना जाता है।

एक्सप्लोरर पहुंच:

* `USRCLASS.DAT\Local Settings\Software\Microsoft\Windows\Shell\Bags`
* `USRCLASS.DAT\Local Settings\Software\Microsoft\Windows\Shell\BagMRU`

डेस्कटॉप पहुंच:

* `NTUSER.DAT\Software\Microsoft\Windows\Shell\BagMRU`
* `NTUSER.DAT\Software\Microsoft\Windows\Shell\Bags`

शेलबैग्स का विश्लेषण करने के लिए आप [**Shellbag Explorer**](https://ericzimmerman.github.io/#!index.md) का उपयोग कर सकते हैं और आपको फोल्डर के **MAC समय** और शेलबैग के **निर्माण तिथि और संशोधन तिथि** भी मिलेगी जो पहली बार और आखिरी बार फ़ोल्डर तक पहुंचा गया था से संबंधित हैं।

निम्नलिखित छवि से 2 बातें ध्यान दें:

1. हमें पता है कि **E: में डाली गई USB** के **फोल्डरों का नाम** क्या था
2. हमें पता है कि **शेलबैग कब बनाया और संशोधित** किया गया था और फोल्डर कब बनाया और पहुंचा गया था

![](<../../../.gitbook/assets/image (475).png>)

## USB जानकारी

### डिवाइस जानकारी

रजिस्ट्री `HKLM\SYSTEM\ControlSet001\Enum\USBSTOR` पीसी स
