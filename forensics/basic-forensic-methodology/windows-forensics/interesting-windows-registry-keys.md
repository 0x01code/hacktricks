# दिलचस्प Windows रजिस्ट्री कुंजी

## दिलचस्प Windows रजिस्ट्री कुंजी

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप एक **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स को** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में पीआर जमा करके अपना योगदान दें।**

</details>

## **Windows सिस्टम जानकारी**

### संस्करण

* **`Software\Microsoft\Windows NT\CurrentVersion`**: Windows संस्करण, सेवा पैक, स्थापना समय और पंजीकृत मालिक

### होस्टनाम

* **`System\ControlSet001\Control\ComputerName\ComputerName`**: होस्टनाम

### समय क्षेत्र

* **`System\ControlSet001\Control\TimeZoneInformation`**: समय क्षेत्र

### अंतिम पहुंच समय

* **`System\ControlSet001\Control\Filesystem`**: अंतिम पहुंच समय (डिफ़ॉल्ट रूप से `NtfsDisableLastAccessUpdate=1` के साथ अक्षम होता है, यदि `0` है, तो यह सक्षम होता है)।
* इसे सक्षम करने के लिए: `fsutil behavior set disablelastaccess 0`

### शटडाउन समय

* `System\ControlSet001\Control\Windows`: शटडाउन समय
* `System\ControlSet001\Control\Watchdog\Display`: शटडाउन गिनती (केवल XP)

### नेटवर्क सूचना

* **`System\ControlSet001\Services\Tcpip\Parameters\Interfaces{GUID_INTERFACE}`**: नेटवर्क इंटरफेसेज
* **`Software\Microsoft\Windows NT\CurrentVersion\NetworkList\Signatures\Unmanaged` & `Software\Microsoft\Windows NT\CurrentVersion\NetworkList\Signatures\Managed` & `Software\Microsoft\Windows NT\CurrentVersion\NetworkList\Nla\Cache`**: पहली और अंतिम बार नेटवर्क कनेक्शन किया गया था और VPN के माध्यम से कनेक्शन
* **`Software\Microsoft\WZCSVC\Parameters\Interfaces{GUID}` (XP के लिए) & `Software\Microsoft\Windows NT\CurrentVersion\NetworkList\Profiles`**: नेटवर्क प्रकार (0x47-वायरलेस, 0x06-केबल, 0x17-3G) और श्रेणी (0-सार्वजनिक, 1-निजी/घर, 2-डोमेन/कार्य) और अंतिम कनेक्शन

### साझा फ़ोल्डर

* **`System\ControlSet001\Services\lanmanserver\Shares\`**: साझा फ़ोल्डर और उनके कॉन्फ़िगरेशन। यदि **क्लाइंट साइड कैशिंग** (CSCFLAGS) सक्षम है, तो साझा फ़ाइलों की एक प्रतिलिपि क्लाइंट और सर्वर में `C:\Windows\CSC` में सहेजी जाएगी।
* CSCFlag=0 -> डिफ़ॉल्ट रूप से उपयोगकर्ता को इंगित करना होगा कि वह कौन सी फ़ाइलें कैश करना चाहता है
* CSCFlag=16 -> स्वचालित रूप से कैशिंग दस्तावेज़। "साझा फ़ोल्डर से उपयोगकर्ता द्वारा खोले गए सभी फ़ाइलें ऑफ़लाइन उपलब्ध होती हैं" जबकि "प्रदर्शन के लिए अनुकूलित" अनचेक किया जाता है।
* CSCFlag=32 -> पिछले विकल्पों की तरह "प्रदर्शन के लिए अनुकूलित" चेक किया जाता है
* CSCFlag=48 -> कैश अक्षम है।
* CSCFlag=2048: यह सेटिंग केवल विन 7 और 8 पर है और यह डिफ़ॉल्ट सेटिंग है जब तक आप "सरल फ़ाइल साझा करना" अक्षम नहीं करते हैं या "उन्नत" साझा करने का विकल्प उपयोग नहीं करते हैं। यह "होमग्रुप" के लिए डिफ़ॉल्ट सेटिंग भी लगता है।
* CSCFlag=768 -> यह सेटिंग केवल साझा मुद्रण उपकरणों पर देखा गया था।

### ऑटोस्टार्ट कार्यक्रम

* `NTUSER
### MRUs

* `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\ComDlg32\LastVisitedMRU`
* `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\ComDlg32\LasVisitedPidlMRU`

यह पाठ बताता है कि किस पथ से executable चलाया गया था।

* `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\ComDlg32\Op enSaveMRU` (XP)
* `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\ComDlg32\Op enSavePidlMRU`

यह खोले गए विंडो में खोले गए फ़ाइलें दिखाता है।

### Last Run Commands

* `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\RunMRU`
* `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\Policies\RunMR`

### User AssistKey

* `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist\{GUID}\Count`

GUID एप्लिकेशन की आईडी है। डेटा सहेजा जाता है:

* अंतिम चलाने का समय
* चलाने की गिनती
* GUI एप्लिकेशन का नाम (इसमें अब्स पथ और अधिक जानकारी होती है)
* फ़ोकस का समय और फ़ोकस का नाम

## Shellbags

जब आप एक निर्देशिका खोलते हैं, तो Windows रजिस्ट्री में निर्देशिका को दिखाने के लिए डेटा सहेजता है। इन प्रविष्टियों को Shellbags के रूप में जाना जाता है।

Explorer Access:

* `USRCLASS.DAT\Local Settings\Software\Microsoft\Windows\Shell\Bags`
* `USRCLASS.DAT\Local Settings\Software\Microsoft\Windows\Shell\BagMRU`

Desktop Access:

* `NTUSER.DAT\Software\Microsoft\Windows\Shell\BagMRU`
* `NTUSER.DAT\Software\Microsoft\Windows\Shell\Bags`

Shellbags का विश्लेषण करने के लिए आप [**Shellbag Explorer**](https://ericzimmerman.github.io/#!index.md) का उपयोग कर सकते हैं और आप निर्देशिका के **MAC समय** को खोज सकेंगे और यहां तक कि आप निर्देशिका के **निर्माण तिथि और संशोधन तिथि** भी खोज सकेंगे जो पहली बार और अंतिम बार निर्देशिका तक पहुँच करते हैं।

इस चित्र से 2 बातें ध्यान दें:

1. हमें पता है कि **E:** में स्थापित USB के **नाम के निर्देशिकाएं** हैं।
2. हमें पता है कि **शेलबैग कब बनाए और संशोधित** किया गया था और निर्देशिका कब बनाई और पहुँची गई थी।

![](<../../../.gitbook/assets/image (475).png>)

## USB जानकारी

### उपकरण जानकारी

रजिस्ट्री `HKLM\SYSTEM\ControlSet001\Enum\USBSTOR` पीसी से कनेक्ट किए गए प्रत्येक USB उपकरण का मॉनिटर करता है।\
इस रजिस्ट्री में निम्नलिखित चीजें मिल सकती हैं:

* निर्माता का नाम
* उत्पाद का नाम और संस्करण
* उपकरण वर्ग आईडी
* वॉल्यूम नाम (निम्नलिखित चित्रों में वॉल्यूम नाम हाइलाइट किया गया है)

![](<../../../.gitbook/assets/image (477).png>)

![](<../../../.gitbook/assets/image (479) (1).png>)

इसके अलावा, रजिस्ट्री `HKLM\SYSTEM\ControlSet001\Enum\USB` की जांच करके और उप-की मानों के मानों की तुलना करके VID मान पाया जा सकता है।

![](<../../../.gitbook/assets/image (478).png>)

पिछली जानकारी के साथ, रजिस्ट्री `SOFTWARE\Microsoft\Windows Portable Devices\Devices` का उपयोग करके **`{GUID}`** प्राप्त किया जा सकता है:

![](<../../../.gitbook/assets/image (480).png>)

### उपयोगकर्ता जिसने उपकरण का उपयोग किया

उपकरण के **{GUID}** के साथ, अब आप **सभी उपयोगकर्ताओं के NTUDER.DAT हाइव की जांच कर सकते हैं**, जहां आप उसे एक में ढूंढ़ते हैं (`NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\Mountpoints2`)।

![](<../../../.gitbook/assets/image (481).png>)

### अंतिम माउंटेड

रजिस्ट्री `System\MoutedDevices` की जांच करके, **कौन सा उपकरण अंतिम रूप में माउंट हुआ था** यह पता लगाया जा सकता है। निम्नलिखित चित्र में देखें कि कैसे उपकरण Toshiba था जो `E:` में माउंट हुआ था (उपकरण एक्सप्लोरर उपकरण का उपयोग करके)।

![](<../../../.gitbook/assets/image (483) (1) (1).png>)

### वॉल्यूम सीरियल नंबर

`Software\Microsoft\Windows NT\CurrentVersion\EMDMgmt` में आप वॉल्यूम सीरियल नंबर पा सकते हैं। **वॉल्यूम नाम और वॉल्यूम सीरियल नंबर जानकारी के साथ आप उस जानकारी को संबंधित LNK फ़ाइलों से सम्बद्ध कर सकते हैं**।

ध्यान दें कि जब एक USB उपकरण को फॉर्मेट क
