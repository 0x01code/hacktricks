# Windows आर्टिफैक्ट्स

## Windows आर्टिफैक्ट्स

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप चाहते हैं कि आपकी **कंपनी HackTricks में विज्ञापित** हो? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का एक्सेस** होना चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** पर **फॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें और PRs सबमिट करें** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को**.

</details>

## सामान्य Windows आर्टिफैक्ट्स

### Windows 10 सूचनाएं

पथ `\Users\<username>\AppData\Local\Microsoft\Windows\Notifications` में आपको डेटाबेस `appdb.dat` (Windows anniversary से पहले) या `wpndatabase.db` (Windows Anniversary के बाद) मिलेगा।

इस SQLite डेटाबेस के अंदर, आप `Notification` टेबल में सभी सूचनाएं (XML प्रारूप में) पाएंगे जो दिलचस्प डेटा समेत हो सकती हैं।

### टाइमलाइन

टाइमलाइन एक Windows विशेषता है जो वेब पेज, संपादित दस्तावेज़ और चलाए गए एप्लिकेशनों का **कालांतरित इतिहास** प्रदान करता है।

डेटाबेस पथ `\Users\<username>\AppData\Local\ConnectedDevicesPlatform\<id>\ActivitiesCache.db` में स्थित होता है। इस डेटाबेस को एक SQLite टूल या टूल [**WxTCmd**](https://github.com/EricZimmerman/WxTCmd) के साथ खोला जा सकता है जो 2 फ़ाइलें उत्पन्न करता है जो टूल [**TimeLine Explorer**](https://ericzimmerman.github.io/#!index.md) के साथ खोली जा सकती हैं।

### ADS (वैकल्पिक डेटा स्ट्रीम्स)

डाउनलोड की गई फ़ाइलों में **ADS Zone.Identifier** हो सकता है जो इसका इंट्रानेट, इंटरनेट आदि से **कैसे** डाउनलोड किया गया था इंडिकेट करता है। कुछ सॉफ़्टवेयर (जैसे ब्राउज़र) आमतौर पर **अधिक** **जानकारी** भी डालते हैं जैसे फ़ाइल को कहां से डाउनलोड किया गया था का **URL**।

## **फ़ाइल बैकअप**

### रीसायकल बिन

Vista/Win7/Win8/Win10 में **रीसायकल बिन** ड्राइव की जड़ में फ़ोल्डर **`$Recycle.bin`** में पाया जा सकता है (`C:\$Recycle.bin`)।\
जब इस फ़ोल्डर में एक फ़ाइल हटाई जाती है तो 2 विशिष्ट फ़ाइलें बनाई जाती हैं:

* `$I{id}`: फ़ाइल की जानकारी (जब यह हटाई गई थी उसकी तारीख}
* `$R{id}`: फ़ाइल की सामग्री

![](<../../../.gitbook/assets/image (486).png>)

इन फ़ाइलों के साथ आप टूल [**Rifiuti**](https://github.com/abelcheung/rifiuti2) का उपयोग करके हटाई गई फ़ाइलों का मूल पता और हटाई गई तारीख प्राप्त कर सकते हैं (Vista - Win10 के लिए `rifiuti-vista.exe` का उपयोग करें)।
```
.\rifiuti-vista.exe C:\Users\student\Desktop\Recycle
```
![](<../../../.gitbook/assets/image (495) (1) (1) (1).png>)

### वॉल्यूम शैडो कॉपी

शैडो कॉपी एक तकनीक है जो माइक्रोसॉफ्ट विंडोज में शामिल है और यह कंप्यूटर फ़ाइल या वॉल्यूम की **बैकअप कॉपियां** या स्नैपशॉट बना सकता है, यहां तक कि जब वे उपयोग में हों।

ये बैकअप आमतौर पर फ़ाइल सिस्टम की जड़ में `\System Volume Information` में स्थित होते हैं और उनका नाम निम्नलिखित छवि में दिखाए गए **UIDs** से मिलकर बनता है:

![](<../../../.gitbook/assets/image (520).png>)

फॉरेंसिक्स इमेज को **ArsenalImageMounter** के साथ माउंट करके, टूल [**ShadowCopyView**](https://www.nirsoft.net/utils/shadow\_copy\_view.html) का उपयोग एक शैडो कॉपी की जांच और शैडो कॉपी बैकअप से **फ़ाइलें निकालने** के लिए किया जा सकता है।

![](<../../../.gitbook/assets/image (521).png>)

रजिस्ट्री एंट्री `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\BackupRestore` में फ़ाइलें और कुंजी **बैकअप न करने** के बारे में जानकारी होती है:

![](<../../../.gitbook/assets/image (522).png>)

रजिस्ट्री `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\VSS` में भी `वॉल्यूम शैडो कॉपियों` के बारे में कॉन्फ़िगरेशन जानकारी होती है।

### ऑफिस ऑटोसेव्ड फ़ाइलें

आप ऑफिस ऑटोसेव्ड फ़ाइलें यहां पाएंगे: `C:\Usuarios\\AppData\Roaming\Microsoft{Excel|Word|Powerpoint}\`

## शैल आइटम्स

शैल आइटम एक ऐसा आइटम है जिसमें दूसरी फ़ाइल तक पहुंच करने के बारे में जानकारी होती है।

### हाल के दस्तावेज़ (LNK)

विंडोज **स्वचालित रूप से** ये **शॉर्टकट** बनाता है जब उपयोगकर्ता एक फ़ाइल **खोलता है, उपयोग करता है या बनाता है**:

* Win7-Win10: `C:\Users\\AppData\Roaming\Microsoft\Windows\Recent\`
* Office: `C:\Users\\AppData\Roaming\Microsoft\Office\Recent\`

जब एक फ़ोल्डर बनाया जाता है, तो एक लिंक फ़ाइल, माता-पिता फ़ोल्डर के लिए एक लिंक और पितामह फ़ोल्डर के लिए भी बनाया जाता है।

ये स्वचालित रूप से बनाए गए लिंक फ़ाइलें **मूल के बारे में जानकारी** शामिल करती हैं जैसे कि यह एक **फ़ाइल** है या एक **फ़ोल्डर**, उस फ़ाइल की **MAC** **समय** के बारे में, फ़ाइल को कहां संग्रहीत किया गया है की **जानकारी** और **लक्ष्य फ़ाइल का फ़ोल्डर**। यह जानकारी उन फ़ाइलों को पुनः प्राप्त करने में सहायक हो सकती है जिन्हें हटा दिया गया हो।

इसके अलावा, लिंक फ़ाइल की **निर्माण तिथि** मूल फ़ाइल का **पहला समय** होती है और लिंक फ़ाइल की **संशोधित तिथि** मूल फ़ाइल का **अंतिम समय** होती है।

इन फ़ाइलों की जांच करने के लिए आप [**LinkParser**](http://4discovery.com/our-tools/) का उपयोग कर सकते हैं।

इस टूल में आपको **2 सेट** के टाइमस्टैंप मिलेंगे:

* **पहला सेट:**
1. FileModifiedDate
2. FileAccessDate
3. FileCreationDate
* **दूसरा सेट:**
1. LinkModifiedDate
2. LinkAccessDate
3. LinkCreationDate.

पहला सेट टाइमस्टैंप **फ़ाइल इसल्फ़ के टाइमस्टैंप** को संदर्भित करता है। दूसरा सेट लिंक फ़ाइल के टाइमस्टैंप को संदर्भित करता है।

आप विंडोज CLI टूल चलाकर भी यही जानकारी प्राप्त कर सकते हैं: [**LECmd.exe**](https://github.com/EricZimmerman/LECmd)
```
LECmd.exe -d C:\Users\student\Desktop\LNKs --csv C:\Users\student\Desktop\LNKs
```
इस मामले में, जानकारी CSV फ़ाइल में सहेजी जाएगी।

### जम्पलिस्ट्स

ये हाल ही में फ़ाइलें हैं जो प्रति एप्लिकेशन द्वारा दर्शाई जाती हैं। यह एक ऐसी सूची है जिसमें आप प्रत्येक एप्लिकेशन पर पहुंच सकते हैं। ये आपके द्वारा बनाए जा सकते हैं या कस्टम भी हो सकते हैं।

ऑटोमेटिक रूप से बनाए गए जम्पलिस्ट्स `C:\Users\{username}\AppData\Roaming\Microsoft\Windows\Recent\AutomaticDestinations\` में संग्रहीत होते हैं। जम्पलिस्ट्स का नाम `{id}.autmaticDestinations-ms` नामकरण प्रारूप का होता है जहां प्रारंभिक आईडी एप्लिकेशन की आईडी होती है।

कस्टम जम्पलिस्ट्स `C:\Users\{username}\AppData\Roaming\Microsoft\Windows\Recent\CustomDestination\` में संग्रहीत होते हैं और ये आमतौर पर एप्लिकेशन द्वारा बनाए जाते हैं क्योंकि फ़ाइल के साथ कुछ महत्वपूर्ण घटना हुई है (शायद पसंदीदा के रूप में चिह्नित किया गया हो)

किसी भी जम्पलिस्ट का **निर्माण समय** फ़ाइल का **पहली बार पहुंच करने का समय दर्शाता है** और **संशोधित समय अंतिम बार** दर्शाता है।

आप [**JumplistExplorer**](https://ericzimmerman.github.io/#!index.md) का उपयोग करके जम्पलिस्ट्स की जांच कर सकते हैं।

![](<../../../.gitbook/assets/image (474).png>)

(_ध्यान दें कि JumplistExplorer द्वारा प्रदान की गई टाइमस्टैम्प जम्पलिस्ट फ़ाइल से संबंधित हैं_)

### शेलबैग्स

[**शेलबैग्स क्या होते हैं इसे जानने के लिए इस लिंक पर क्लिक करें।**](interesting-windows-registry-keys.md#shellbags)

## Windows USB का उपयोग

USB डिवाइस का उपयोग हुआ है इसकी पहचान करना संभव है धन्यवाद:

* Windows हाल की फ़ोल्डर
* Microsoft Office हाल की फ़ोल्डर
* जम्पलिस्ट्स

ध्यान दें कि कुछ LNK फ़ाइल मूल पथ की बजाय WPDNSE फ़ोल्डर को पॉइंट करती हैं:

![](<../../../.gitbook/assets/image (476).png>)

WPDNSE फ़ोल्डर में फ़ाइलें मूल वाली की कॉपी होती हैं, फिर यह PC को रीस्टार्ट करने पर बच नहीं सकती हैं और GUID एक शेलबैग से लिया जाता है।

### रजिस्ट्री सूचना

यूएसबी कनेक्टेड डिवाइस के बारे में दिलचस्प जानकारी कौन सी रजिस्ट्री कुंजी में होती है, इसे जानने के लिए इस पेज की जांच करें। (interesting-windows-registry-keys.md#usb-information)

### setupapi

USB कनेक्शन किस समय हुआ है इसके टाइमस्टैम्प प्राप्त करने के लिए फ़ाइल `C:\Windows\inf\setupapi.dev.log` की जांच करें (खोज करें `Section start` के लिए)।

![](<../../../.gitbook/assets/image (477) (2) (2) (2) (2) (2) (2) (2) (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (14).png>)

### USB Detective

[**USBDetective**](https://usbdetective.com) का उपयोग करके एक इमेज में कनेक्ट किए गए USB डिवाइस के बारे में जानकारी प्राप्त की जा सकती है।

![](<../../../.gitbook/assets/image (483).png>)

### प्लग और प्ले सफाई

'प्लग और प्ले सफाई' निर्धारित कार्य के लिए जिम्मेदार है जो लेगेस संस्करणों को साफ़ करता है। यह ऐसा दिखाई देता है (ऑनलाइन रिपोर्टों के आधार पर) कि यह 30 दिनों से अधिक समय तक उपयोग नहीं हुए ड्राइवर्स को भी हटा लेता है, हालांकि इसकी विवरण में यह कहा गया है कि "प्रत्येक ड्राइवर पैकेज का सबसे नवीनतम संस्करण संग्रहित रखा जाएगा"। इस प्रकार, 30 दिनों से अधिक समय तक कनेक्ट नहीं हुए हटाने वाले उपकरणों के ड्राइवर हटा दिए
### Outlook OST

जब Microsoft Outlook को **IMAP** या **Exchange** सर्वर का उपयोग करके कॉन्फ़िगर किया जाता है, तो यह एक **OST** फ़ाइल उत्पन्न करता है जो PST फ़ाइल की तरहीं जानकारी संग्रहित करती है। यह फ़ाइल सर्वर के साथ **पिछले 12 महीनों** के लिए सिंक्रनाइज़ की जाती है, **50GB** की **अधिकतम फ़ाइल आकार** रखती है और **PST** फ़ाइल के साथ **एक ही फ़ोल्डर में** सहेजी जाती है। आप [**Kernel OST viewer**](https://www.nucleustechnologies.com/ost-viewer.html) का उपयोग करके इस फ़ाइल की जांच कर सकते हैं।

### अटैचमेंट की पुनर्प्राप्ति

आप उन्हें निम्नलिखित फ़ोल्डर में ढूंढ़ सकते हैं:

* `%APPDATA%\Local\Microsoft\Windows\Temporary Internet Files\Content.Outlook` -> IE10
* `%APPDATA%\Local\Microsoft\InetCache\Content.Outlook` -> IE11+

### Thunderbird MBOX

**Thunderbird** जानकारी को **MBOX** **फ़ाइलों** में `\Users\%USERNAME%\AppData\Roaming\Thunderbird\Profiles` फ़ोल्डर में संग्रहीत करता है।

## थंबनेल

जब एक उपयोगकर्ता एक फ़ोल्डर तक पहुंचता है और उसे थंबनेल का उपयोग करके संगठित करता है, तो एक `thumbs.db` फ़ाइल बनाई जाती है। यह डीबी **फ़ोल्डर की छवियों के थंबनेल** को संग्रहीत करती है, यदि वे हटा दिए जाते हैं तो भी। WinXP और Win 8-8.1 में यह फ़ाइल स्वचालित रूप से बनाई जाती है। Win7/Win10 में, यह स्वचालित रूप से बनाई जाती है अगर इसे UNC पथ (\IP\folder...) के माध्यम से एक्सेस किया जाता है।

आप [**Thumbsviewer**](https://thumbsviewer.github.io) उपकरण का उपयोग करके इस फ़ाइल को पढ़ सकते हैं।

### Thumbcache

Windows Vista के साथ शुरू होकर, **थंबनेल पूर्वावलोकन सांकेतिक स्थान पर संग्रहीत होते हैं**। इससे सिस्टम को उनके स्थान के अनुभव के बिना छवियों तक पहुंच मिलती है और Thumbs.db फ़ाइलों की स्थानीयता समस्याओं को सुलझाता है। कैश **`%userprofile%\AppData\Local\Microsoft\Windows\Explorer`** में कई फ़ाइलें होती हैं जिनका नामकरण **thumbcache\_xxx.db** (आकार के अनुसार संख्याबद्ध); साथ ही हर आकार के डेटाबेस में थंबनेल ढूंढ़ने के लिए एक सूचकांक भी होता है।

* Thumbcache\_32.db -> छोटा
* Thumbcache\_96.db -> मध्यम
* Thumbcache\_256.db -> बड़ा
* Thumbcache\_1024.db -> अतिरिक्त बड़ा

आप [**ThumbCache Viewer**](https://thumbcacheviewer.github.io) उपकरण का उपयोग करके इस फ़ाइल को पढ़ सकते हैं।

## Windows रजिस्ट्री

Windows रजिस्ट्री में **सिस्टम और उपयोगकर्ताओं के कार्यों** के बारे में बहुत सारी **जानकारी** होती है।

रजिस्ट्री संबंधी फ़ाइलें निम्नलिखित स्थानों पर स्थित होती हैं:

* %windir%\System32\Config\*_SAM\*_: `HKEY_LOCAL_MACHINE`
* %windir%\System32\Config\*_SECURITY\*_: `HKEY_LOCAL_MACHINE`
* %windir%\System32\Config\*_SYSTEM\*_: `HKEY_LOCAL_MACHINE`
* %windir%\System32\Config\*_SOFTWARE\*_: `HKEY_LOCAL_MACHINE`
* %windir%\System32\Config\*_DEFAULT\*_: `HKEY_LOCAL_MACHINE`
* %UserProfile%{User}\*_NTUSER.DAT\*_: `HKEY_CURRENT_USER`

Windows Vista और Windows 2008 Server से ऊपर, `HKEY_LOCAL_MACHINE` रजिस्ट्री फ़ाइलों की कुछ बैकअप **`%Windir%\System32\Config\RegBack\`** में होती है।

इन संस्करणों से, रजिस्ट्री फ़ाइल **`%UserProfile%\{User}\AppData\Local\Microsoft\Windows\USERCLASS.DAT`** बनाई जाती है जो कार्यक्रम के निष्पादन के बारे में जानकारी संग्रहित करती है।

### उपयोगी उपकरण

रजिस्ट्री फ़ाइलों का विश्लेषण करने के लिए कुछ उपयोगी उपकरण हैं:

* **रजिस्ट्री संपादक**: यह Windows में स्थापित होता है। यह वर्तमान सत्र के Windows रजिस्ट्री में नेविगेट करने के लिए एक GUI है।
* [**रजिस्ट्री एक्सप्लोरर**](https://ericzimmerman.github.io/#!index.md): यह आपको रजिस्ट्री फ़ाइल लोड करने और उन्हें एक GUI के साथ नेविगेट करने की अनुमति देता है। यह रजिस्ट्री में दिलचस्प जानकारी वाले कुंजीयों को हाइलाइट करन
```bash
.\PECmd.exe -d C:\Users\student\Desktop\Prefetch --html "C:\Users\student\Desktop\out_folder"
```
![](<../../../.gitbook/assets/image (487).png>)

### सुपरप्रीफेच

**सुपरप्रीफेच** का उद्देश्य प्रीफेच की तरह है, **प्रोग्राम को तेजी से लोड करना** जिसके द्वारा यह पूर्वानुमान लगाता है कि अगले क्या लोड होने वाला है। हालांकि, यह प्रीफेच सेवा को प्रतिस्थापित नहीं करता है।\
यह सेवा `C:\Windows\Prefetch\Ag*.db` में डेटाबेस फ़ाइलें उत्पन्न करेगी।

इन डेटाबेस में आप प्रोग्राम का **नाम**, **चलाने की संख्या**, **खोले गए फ़ाइलें**, **एक्सेस की गई वॉल्यूम**, **पूरा पथ**, **समयअंतराल** और **टाइमस्टैम्प** खोज सकते हैं।

आप इस जानकारी तक पहुंच सकते हैं उपकरण [**CrowdResponse**](https://www.crowdstrike.com/resources/community-tools/crowdresponse/) का उपयोग करके।

### SRUM

**सिस्टम रिसोर्स उपयोग मॉनिटर** (SRUM) **प्रक्रिया द्वारा उपयोग किए जाने वाले संसाधनों** का **मॉनिटरिंग** करता है। यह W8 में प्रकट हुआ था और इसे `C:\Windows\System32\sru\SRUDB.dat` में स्थित एक ESE डेटाबेस में डेटा संग्रहीत करता है।

यह निम्नलिखित जानकारी प्रदान करता है:

* ऐपआईडी और पथ
* प्रक्रिया को चलाने वाला उपयोगकर्ता
* भेजे गए बाइट
* प्राप्त बाइट
* नेटवर्क इंटरफ़ेस
* कनेक्शन अवधि
* प्रक्रिया अवधि

यह जानकारी हर 60 मिनट में अपडेट होती है।

आप इस फ़ाइल से तिथि प्राप्त कर सकते हैं उपकरण [**srum\_dump**](https://github.com/MarkBaggett/srum-dump) का उपयोग करके।
```bash
.\srum_dump.exe -i C:\Users\student\Desktop\SRUDB.dat -t SRUM_TEMPLATE.xlsx -o C:\Users\student\Desktop\srum
```
### AppCompatCache (ShimCache)

**Shimcache**, जिसे **AppCompatCache** भी कहा जाता है, **Application Compatibility Database** का एक घटक है, जिसे **Microsoft** ने बनाया था और ऑपरेटिंग सिस्टम द्वारा अनुप्रयोग संगतता समस्याओं की पहचान के लिए उपयोग किया जाता है।

कैश ऑपरेटिंग सिस्टम के आधार पर विभिन्न फ़ाइल मेटाडेटा संग्रहीत करता है, जैसे:

* फ़ाइल पूरा पथ
* फ़ाइल का आकार
* **$Standard\_Information** (SI) की अंतिम संशोधित समय
* ShimCache की अंतिम अद्यतन समय
* प्रक्रिया निष्पादन ध्वज

यह जानकारी रजिस्ट्री में मिल सकती है:

* `SYSTEM\CurrentControlSet\Control\SessionManager\Appcompatibility\AppcompatCache`
* XP (96 प्रविष्टियाँ)
* `SYSTEM\CurrentControlSet\Control\SessionManager\AppcompatCache\AppCompatCache`
* Server 2003 (512 प्रविष्टियाँ)
* 2008/2012/2016 Win7/Win8/Win10 (1024 प्रविष्टियाँ)

आप इस जानकारी को पार्स करने के लिए उपकरण [**AppCompatCacheParser**](https://github.com/EricZimmerman/AppCompatCacheParser) का उपयोग कर सकते हैं।

![](<../../../.gitbook/assets/image (488).png>)

### Amcache

**Amcache.hve** फ़ाइल एक रजिस्ट्री फ़ाइल है जो निष्पादित अनुप्रयोगों की जानकारी संग्रहीत करती है। यह `C:\Windows\AppCompat\Programas\Amcache.hve` में स्थित होती है।

**Amcache.hve** नवीनतम प्रक्रियाओं को रिकॉर्ड करती है और उन फ़ाइलों के पथ की सूची को दर्ज करती है जो निष्पादित कार्यक्रम की खोज के लिए उपयोग की जा सकती है। यह भी कार्यक्रम के SHA1 को रिकॉर्ड करती है।

आप इस जानकारी को उपकरण [**Amcacheparser**](https://github.com/EricZimmerman/AmcacheParser) के साथ पार्स कर सकते हैं।
```bash
AmcacheParser.exe -f C:\Users\student\Desktop\Amcache.hve --csv C:\Users\student\Desktop\srum
```
सबसे दिलचस्प CVS फ़ाइल जो उत्पन्न होती है, वह है `Amcache_Unassociated file entries`.

### RecentFileCache

इस आर्टिफैक्ट को केवल W7 में `C:\Windows\AppCompat\Programs\RecentFileCache.bcf` में पाया जा सकता है और इसमें कुछ बाइनरीज के हालिया निष्पादन के बारे में जानकारी होती है।

आप इस फ़ाइल को पार्स करने के लिए टूल [**RecentFileCacheParse**](https://github.com/EricZimmerman/RecentFileCacheParser) का उपयोग कर सकते हैं।

### Scheduled tasks

आप उन्हें `C:\Windows\Tasks` या `C:\Windows\System32\Tasks` से निकाल सकते हैं और उन्हें XML के रूप में पढ़ सकते हैं।

### Services

आप उन्हें रजिस्ट्री में `SYSTEM\ControlSet001\Services` के तहत ढूंढ सकते हैं। आप देख सकते हैं कि क्या निष्पादित होने वाला है और कब।

### **Windows Store**

स्थापित एप्लिकेशन `\ProgramData\Microsoft\Windows\AppRepository\` में पाए जा सकते हैं।
इस रिपॉजिटरी में एक **लॉग** होता है जिसमें विभिन्न एप्लिकेशनों की **प्रत्येक स्थापित एप्लिकेशन** की जानकारी होती है जो डेटाबेस **`StateRepository-Machine.srd`** के भीतर होती है।

इस डेटाबेस के Application टेबल में, "Application ID", "PackageNumber", और "Display Name" नामक कॉलम्स पाए जा सकते हैं। इन कॉलम्स में प्री-स्थापित और स्थापित एप्लिकेशनों के बारे में जानकारी होती है और यह पता लगाया जा सकता है कि क्या कुछ एप्लिकेशन अनइंस्टॉल किए गए हैं क्योंकि स्थापित एप्लिकेशनों के आईडी सीक्वेंशियल होने चाहिए।

रजिस्ट्री पथ `Software\Microsoft\Windows\CurrentVersion\Appx\AppxAllUserStore\Applications\` में स्थापित एप्लिकेशन भी **ढूंढ़ा जा सकता है**।
और **अनइंस्टॉल** **एप्लिकेशन** में: `Software\Microsoft\Windows\CurrentVersion\Appx\AppxAllUserStore\Deleted\`

## Windows Events

Windows इवेंट्स में दिखाई देने वाली जानकारी है:

* क्या हुआ
* टाइमस्टैम्प (UTC + 0)
* संलग्न उपयोगकर्ता
* संलग्न होस्ट (होस्टनाम, आईपी)
* पहुंचे गए संपत्ति (फ़ाइलें, फ़ोल्डर, प्रिंटर, सेवाएं)

लॉग्स `C:\Windows\System32\config` में Windows Vista से पहले और `C:\Windows\System32\winevt\Logs` में Windows Vista के बाद स्थित होते हैं। Windows Vista से पहले, इवेंट लॉग्स बाइनरी प्रारूप में थे और इसके बाद, वे **XML प्रारूप** में होते हैं और **.evtx** एक्सटेंशन का उपयोग करते हैं।

इवेंट फ़ाइलों की स्थान प्रणाली रजिस्ट्री में **`HKLM\SYSTEM\CurrentControlSet\services\EventLog\{Application|System|Security}`** में पाई जा सकती है।

इन्हें Windows Event Viewer (**`eventvwr.msc`**) से देखा जा सकता है या [**Event Log Explorer**](https://eventlogxp.com) जैसे अन्य उपकरणों के साथ देखा जा सकता है।

### सुरक्षा

इसमें पहुंच घटनाएं रजिस्टर होती हैं और सुरक्षा कॉन्फ़िगरेशन के बारे में जानकारी देती है जो `C:\Windows\System32\winevt\Security.evtx` में पाई जा सकती है।

इवेंट फ़ाइल का **अधिकतम आकार** विन्यासयोग्य होता है, और जब अधिकतम आकार प्राप्त हो जाता है, तो पुरानी घटनाएं अधिलेखित कर दी जाती हैं।

इन घटनाओं को इस प्रकार रजिस्टर किया जाता है:

* लॉगिन/लॉगआउट
* उपयोगकर्ता की क्रियाएं
* फ़ाइलों, फ़ोल्डरों और साझा संपत्तियों का उपयोग
* सुरक्षा कॉन्फ़िगरेशन का संशोधन

उपयोगकर्ता प्रमाणीकरण से संबंधित घटनाएं:

| EventID   | विवरण                        |
| --------- | ---------------------------- |
| 4624      | सफल प्रमाणीकरण            |
| 4625      | प्रमाणीकरण त्रुटि         |
| 4634/4647 | लॉगआउट                     |
| 4672      | प्रशासनिक अनुमतियों के साथ लॉगिन |

EventID 4634/4647 के अंदर
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) **में** या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**।**
* **अपने हैकिंग ट्रिक्स को साझा करें द्वारा PRs सबमिट करके** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud)।

</details>
