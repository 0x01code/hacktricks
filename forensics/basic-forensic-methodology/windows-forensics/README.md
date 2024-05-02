# Windows आर्टिफैक्ट्स

## Windows आर्टिफैक्ट्स

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

<figure><img src="https://pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}

## सामान्य Windows आर्टिफैक्ट्स

### Windows 10 सूचनाएँ

पथ `\Users\<username>\AppData\Local\Microsoft\Windows\Notifications` में आपको डेटाबेस `appdb.dat` (Windows anniversary से पहले) या `wpndatabase.db` (Windows Anniversary के बाद) मिल सकता है।

इस SQLite डेटाबेस में, आप `Notification` तालिका पा सकते हैं जिसमें सभी सूचनाएँ (XML प्रारूप में) हो सकती हैं जो दिलचस्प डेटा शामिल कर सकती हैं।

### टाइमलाइन

टाइमलाइन एक Windows विशेषता है जो वेब पेजों के यातायात, संपादित दस्तावेज़ और चलाए गए एप्लिकेशनों का **कालांतरिक इतिहास** प्रदान करता है।

डेटाबेस पथ `\Users\<username>\AppData\Local\ConnectedDevicesPlatform\<id>\ActivitiesCache.db` में स्थित है। इस डेटाबेस को एक SQLite उपकरण के साथ खोला जा सकता है या उपकरण [**WxTCmd**](https://github.com/EricZimmerman/WxTCmd) के साथ जो 2 फ़ाइलें उत्पन्न करता है जो उपकरण [**TimeLine Explorer**](https://ericzimmerman.github.io/#!index.md) के साथ खोली जा सकती हैं।

### ADS (वैकल्पिक डेटा स्ट्रीम्स)

डाउनलोड की गई फ़ाइलें **ADS Zone.Identifier** शामिल कर सकती हैं जो इसे इंट्रानेट, इंटरनेट से कैसे डाउनलोड किया गया था का संकेत देता है। कुछ सॉफ़्टवेयर (जैसे ब्राउज़र) आम तौर पर फ़ाइल को डाउनलोड किये गए URL जैसी **अधिक जानकारी** भी डालते हैं।

## **फ़ाइल बैकअप्स**

### रीसाइकल बिन

Vista/Win7/Win8/Win10 में **रीसाइकल बिन** ड्राइव की रूट में फ़ोल्डर **`$Recycle.bin`** में पाया जा सकता है (`C:\$Recycle.bin`)।\
जब इस फ़ोल्डर में एक फ़ाइल हटाई जाती है तो 2 विशिष्ट फ़ाइलें बनाई जाती हैं:

* `$I{id}`: फ़ाइल सूचना (जब यह हटाई गई थी की तारीख}
* `$R{id}`: फ़ाइल की सामग्री

![](<../../../.gitbook/assets/image (486).png>)

इन फ़ाइलों के साथ आप उपकरण [**Rifiuti**](https://github.com/abelcheung/rifiuti2) का उपयोग करके हटाई गई फ़ाइलों का मूल पता और हटाई गई तारीख प्राप्त कर सकते हैं (Vista – Win10 के लिए `rifiuti-vista.exe` का उपयोग करें)।
```
.\rifiuti-vista.exe C:\Users\student\Desktop\Recycle
```
![](<../../../.gitbook/assets/image (495) (1) (1) (1).png>)

### वॉल्यूम शैडो कॉपी

शैडो कॉपी एक तकनीक है जो माइक्रोसॉफ्ट विंडोज में शामिल है जो कंप्यूटर फ़ाइल या वॉल्यूम की **बैकअप कॉपियां** या स्नैपशॉट बना सकती है, भले ही वे उपयोग में हों।

ये बैकअप्स आम तौर पर `\System Volume Information` में रूट फ़ाइल सिस्टम से स्थित होते हैं और उनका नाम निम्नलिखित छवि में दिखाए गए **UIDs** से मिलकर बनता है:

![](<../../../.gitbook/assets/image (520).png>)

**ArsenalImageMounter** के साथ फोरेंसिक्स इमेज को माउंट करके, उपकरण [**ShadowCopyView**](https://www.nirsoft.net/utils/shadow\_copy\_view.html) का उपयोग एक शैडो कॉपी की जांच करने और शैडो कॉपी बैकअप से **फ़ाइलें निकालने** के लिए किया जा सकता है।

![](<../../../.gitbook/assets/image (521).png>)

रजिस्ट्री एंट्री `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\BackupRestore` फ़ाइलें और कुंजीयों को शामिल करती है **जिन्हें बैकअप न करें**:

![](<../../../.gitbook/assets/image (522).png>)

रजिस्ट्री `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\VSS` में भी `वॉल्यूम शैडो कॉपियों` के बारे में विन्यास सूचना होती है।

### ऑफिस ऑटोसेव्ड फ़ाइलें

आप ऑफिस ऑटोसेव्ड फ़ाइलें यहाँ पा सकते हैं: `C:\Usuarios\\AppData\Roaming\Microsoft{Excel|Word|Powerpoint}\`

## शैल आइटम्स

एक शैल आइटम एक आइटम है जो दूसरी फ़ाइल तक पहुँचने के बारे में जानकारी रखता है।

### हाल की दस्तावेज़ (LNK)

विंडोज **स्वचालित रूप से** ये **शॉर्टकट** बनाता है जब उपयोगकर्ता एक फ़ाइल **खोलता है, उपयोग करता है या बनाता है**:

* Win7-Win10: `C:\Users\\AppData\Roaming\Microsoft\Windows\Recent\`
* Office: `C:\Users\\AppData\Roaming\Microsoft\Office\Recent\`

जब एक फ़ोल्डर बनाया जाता है, तो एक लिंक फ़ाइल फ़ोल्डर, माता-पिता फ़ोल्डर और परदादी फ़ोल्डर का भी बनाया जाता है।

ये स्वचालित रूप से बनाए गए लिंक फ़ाइलें **मूल के बारे में जानकारी रखती हैं** जैसे कि यह कोई **फ़ाइल है** या एक **फ़ोल्डर**, उस फ़ाइल के **MAC समय**, जहाँ फ़ाइल संग्रहित है और **लक्षित फ़ाइल का फ़ोल्डर**। यह जानकारी उन फ़ाइलों को पुनः प्राप्त करने के लिए उपयोगी हो सकती है जिन्हें हटा दिया गया था।

इसके अलावा, लिंक फ़ाइल की **निर्मित तिथि** वह **समय** है जब मूल फ़ाइल का **पहली बार उपयोग** किया गया था और लिंक फ़ाइल का **संशोधित तिथि** वह **समय** है जब मूल फ़ाइल का **अंतिम बार उपयोग** किया गया था।

इन फ़ाइलों की जांच करने के लिए आप [**LinkParser**](http://4discovery.com/our-tools/) का उपयोग कर सकते हैं।

इस उपकरण में आपको **2 सेट** के टाइमस्टैम्प मिलेंगे:

* **पहला सेट:**
1. फ़ाइल संशोधित तिथि
2. फ़ाइल एक्सेस तिथि
3. फ़ाइल निर्माण तिथि
* **दूसरा सेट:**
1. लिंक संशोधित तिथि
2. लिंक एक्सेस तिथि
3. लिंक निर्माण तिथि।

पहले सेट का टाइमस्टैम्प **फ़ाइल इसल्फ के टाइमस्टैम्प्स** को संदर्भित करता है। दूसरा सेट **लिंक की फ़ाइल के टाइमस्टैम्प्स** को संदर्भित करता है।

आप विंडोज CLI उपकरण चलाकर इसी जानकारी को प्राप्त कर सकते हैं: [**LECmd.exe**](https://github.com/EricZimmerman/LECmd)
```
LECmd.exe -d C:\Users\student\Desktop\LNKs --csv C:\Users\student\Desktop\LNKs
```
### जम्पलिस्ट्स

ये हाल के फ़ाइलें हैं जो प्रति एप्लिकेशन द्वारा दर्शाई जाती हैं। यह एक ऐसी **सूची है जिसमें एक एप्लिकेशन द्वारा उपयोग की गई हाल की फ़ाइलें** हैं जिसे आप प्रत्येक एप्लिकेशन पर एक्सेस कर सकते हैं। ये **स्वचालित रूप से बनाई जा सकती हैं या कस्टम** हो सकती हैं।

**स्वचालित रूप से बनाए गए जम्पलिस्ट्स** `C:\Users\{username}\AppData\Roaming\Microsoft\Windows\Recent\AutomaticDestinations\` में संग्रहित होते हैं। जम्पलिस्ट्स का नाम इस प्रारूप का होता है `{id}.autmaticDestinations-ms` जहां प्रारंभिक आईडी एप्लिकेशन की आईडी होती है।

कस्टम जम्पलिस्ट्स `C:\Users\{username}\AppData\Roaming\Microsoft\Windows\Recent\CustomDestination\` में संग्रहित होती हैं और इन्हें एप्लिकेशन द्वारा सामान्यत: कुछ **महत्वपूर्ण** घटना होने के कारण बनाया जाता है (शायद पसंदीदा के रूप में चिह्नित किया गया हो)

किसी भी जम्पलिस्ट का **निर्माण समय** दर्शाता है **फ़ाइल तक पहुँचने का पहला समय** और **संशोधित समय** अंतिम बार।

आप [**JumplistExplorer**](https://ericzimmerman.github.io/#!index.md) का उपयोग करके जम्पलिस्ट्स की जांच कर सकते हैं।

![](<../../../.gitbook/assets/image (474).png>)

(_कृपया ध्यान दें कि JumplistExplorer द्वारा प्रदान की गई समय चिह्नित जम्पलिस्ट फ़ाइल से संबंधित हैं_)

### शेलबैग्स

[**इस लिंक पर जाएं और जानें कि शेलबैग्स क्या हैं।**](interesting-windows-registry-keys.md#shellbags)

## Windows USB का उपयोग

एक USB डिवाइस का उपयोग किया गया था यह पहचानना संभव है धन्यवाद है:

* Windows हाल की फ़ोल्डर
* Microsoft Office हाल की फ़ोल्डर
* जम्पलिस्ट्स

ध्यान दें कि कुछ LNK फ़ाइल मूल पथ की बजाय WPDNSE फ़ोल्डर को इंगित करती हैं:

![](<../../../.gitbook/assets/image (476).png>)

फ़ोल्डर WPDNSE में फ़ाइलें मूल वालों की प्रतिलिपि होती हैं, फिर वे PC की पुनरारंभी नहीं होंगी और GUID एक शेलबैग से लिया गया है।

### रजिस्ट्री सूचना

[जानने के लिए इस पृष्ठ की जाँच करें](interesting-windows-registry-keys.md#usb-information) कि कौन सी रजिस्ट्री कुंजीयाँ USB कनेक्टेड डिवाइस के बारे में दिलचस्प जानकारी रखती हैं।

### setupapi

USB कनेक्शन किया गया था कब हुआ यह जानने के लिए फ़ाइल `C:\Windows\inf\setupapi.dev.log` की जाँच करें (खोज करें `Section start`).

![](<../../../.gitbook/assets/image (477) (2) (2) (2) (2) (2) (2) (2) (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (14).png>)

### USB डिटेक्टिव

[**USBDetective**](https://usbdetective.com) का उपयोग एक छवि में कनेक्ट किए गए USB डिवाइस के बारे में जानकारी प्राप्त करने के लिए किया जा सकता है।

![](<../../../.gitbook/assets/image (483).png>)

### प्लग और प्ले सफाई

'प्लग और प्ले सफाई' नामक निर्धारित कार्य को पुराने ड्राइवर संस्करणों को हटाने के लिए प्राथमिक रूप से डिज़ाइन किया गया है। नवीनतम ड्राइवर पैकेज संस्करण को बनाए रखने के उसके निर्दिष्ट उद्देश्य के विपरीत, ऑनलाइन स्रोत सुझाव देते हैं कि यह 30 दिनों के लिए निष्क्रिय रहे ड्राइवर्स पर भी निशाना साधता है। इस परिणामस्वरूप, पिछले 30 दिनों में कनेक्ट नहीं किए गए हटाए जा सकते हैं।

कार्य से संबंधित निम्नलिखित पथ पर स्थित है:
`C:\Windows\System32\Tasks\Microsoft\Windows\Plug and Play\Plug and Play Cleanup`.

कार्य की सामग्री का चित्र प्रदर्शित किया गया है:
![](https://2.bp.blogspot.com/-wqYubtuR_W8/W19bV5S9XyI/AAAAAAAANhU/OHsBDEvjqmg9ayzdNwJ4y2DKZnhCdwSMgCLcBGAs/s1600/xml.png)

**कार्य के मुख्य घटक और सेटिंग्स:**
- **pnpclean.dll**: यह DLL वास्तविक सफाई प्रक्रिया के लिए जिम्मेदार है।
- **UseUnifiedSchedulingEngine**: `TRUE` पर सेट किया गया है, जो सामान्य कार्य निर्धारण इंजन का उपयोग करने की संकेत करता है।
- **MaintenanceSettings**:
- **अवधि ('P1M')**: कार्य सूचीकरणकर्ता को नियमित स्वचालित रूप से मासिक सफाई कार्य आरंभ करने के लिए निर्देशित करता है।
- **समयसीमा ('P2M')**: यदि कार्य दो लगातार महीनों के लिए विफल हो जाता है, तो टास्क स्केड्यूलर को आपातकालीन स्वचालित रूप से मासिक सफाई कार्य को करने के लिए निर्देशित करता है।

यह विन्यास ड्राइवरों की नियमित रखरखाव और सफाई सुनिश्चित करता है, जो लगातार विफलताओं के मामले में कार्य को पुनः प्रयास करने के लिए प्रावधान करता है।

**अधिक जानकारी के लिए देखें:** [**https://blog.1234n6.com/2018/07/windows-plug-and-play-cleanup.html**](https://blog.1234n6.com/2018/07/windows-plug-and-play-cleanup.html)

## ईमेल

ईमेल में **2 दिलचस्प भाग होते हैं: हेडर्स और सामग्री**। **हेडर्स** में आप जानकारी पा सकते हैं जैसे:

* **कौन** ईमेल भेजा (ईमेल पता, आईपी, ईमेल को पुनर्निर्देशित करने वाले मेल सर्वर)
* **कब** ईमेल भेजा गया था

इसके अलावा, `References` और `In-Reply-To` हेडर्स में आप मेसेज की आईडी पा सकते हैं:

![](<../../../.gitbook/assets/image (484).png>)

### Windows मेल ऐप

यह एप्लिकेशन ईमेल को HTML या पाठ में सहेजता है। आप ईमेल को नीचे उपफ़ोल्डर में पा सकते हैं `\Users\<username>\AppData\Local\Comms\Unistore\data\3\`। ईमेल `.dat` एक्सटेंशन के साथ सहेजे जाते हैं।

ईमेलों की **मेटाडेटा** और **संपर्क** को **EDB डेटाबेस** में पाया जा सकता है: `\Users\<username>\AppData\Local\Comms\UnistoreDB\store.vol`

फ़ाइल का एक्सटेंशन **बदलें** `.vol` से `.edb` और आप इसे खोलने के लिए उपकरण [ESEDatabaseView](https://www.nirsoft.net/utils/ese\_database\_view.html) का उपयोग कर सकते हैं। `Message` टेबल के अंदर आप ईमेल देख सकते हैं।

### Microsoft Outlook

जब Exchange सर्वर या Outlook क्लाइंट का उपयोग किया जाता है तो कुछ MAPI हेडर्स होते हैं:

* `Mapi-Client-Submit-Time`: ईमेल भेजे जाने का समय
* `Mapi-Conversation-Index`: थ्रेड के बच्चों की संख्या और हर मेसेज के समय का चिन्हांक
* `Mapi-Entry-ID`: मेसेज पहचानकर्ता।
* `Mappi-Message-Flags` और `Pr_last_Verb-Executed`: MAPI क्लाइंट के बारे में जानकारी (मेसेज पढ़ा गया? पढ़ा नहीं गया? प्रतिक्रिया दी गई? पुनर्निर्देशित किया गया? बाहर ऑफिस?)

Microsoft Outlook क्लाइंट में
### Microsoft Outlook OST Files

एक **OST फ़ाइल** Microsoft Outlook द्वारा उत्पन्न की जाती है जब यह **IMAP** या **Exchange** सर्वर के साथ कॉन्फ़िगर किया जाता है, जो एक PST फ़ाइल के समान जानकारी संग्रहित करती है। यह फ़ाइल सर्वर के साथ समकलीन होती है, **पिछले 12 महीने** तक के डेटा को **अधिकतम 50GB** आकार तक रखती है, और PST फ़ाइल के समान निर्देशिका में स्थित है। एक OST फ़ाइल देखने के लिए, [**Kernel OST viewer**](https://www.nucleustechnologies.com/ost-viewer.html) का उपयोग किया जा सकता है।

### अटैचमेंट पुनः प्राप्त करना

खो गए अटैचमेंट को निम्नलिखित से पुनः प्राप्त किया जा सकता है:

- **IE10 के लिए**: `%APPDATA%\Local\Microsoft\Windows\Temporary Internet Files\Content.Outlook`
- **IE11 और ऊपर के लिए**: `%APPDATA%\Local\Microsoft\InetCache\Content.Outlook`

### Thunderbird MBOX Files

**Thunderbird** डेटा संग्रहित करने के लिए **MBOX फ़ाइलें** का उपयोग करता है, जो `\Users\%USERNAME%\AppData\Roaming\Thunderbird\Profiles` में स्थित है।

### छवि थंबनेल्स

- **Windows XP और 8-8.1**: थंबनेल्स के साथ एक फ़ोल्डर तक पहुँचने पर `thumbs.db` फ़ाइल उत्पन्न होती है जो छवि पूर्वावलोकन संग्रहित करती है, भले ही उसके हटाने के बाद भी।
- **Windows 7/10**: `thumbs.db` उत्पन्न होती है जब UNC पथ के माध्यम से नेटवर्क के माध्यम से पहुँचा जाता है।
- **Windows Vista और नएरे**: थंबनेल पूर्वावलोकन `%userprofile%\AppData\Local\Microsoft\Windows\Explorer` में केंद्रीकृत है जिसमें फ़ाइलें **thumbcache\_xxx.db** नाम से होती हैं। [**Thumbsviewer**](https://thumbsviewer.github.io) और [**ThumbCache Viewer**](https://thumbcacheviewer.github.io) इन फ़ाइलों को देखने के लिए उपकरण हैं।

### Windows रजिस्ट्री सूचना

Windows रजिस्ट्री, विभिन्न `HKEY_LOCAL_MACHINE` उपकीजों के लिए फ़ाइलों में संग्रहित व्यापक सिस्टम और उपयोगकर्ता गतिविधि डेटा संग्रहित करता है:

- विभिन्न `HKEY_LOCAL_MACHINE` उपकीजों के लिए `%windir%\System32\Config` में।
- `HKEY_CURRENT_USER` के लिए `%UserProfile%{User}\NTUSER.DAT` में।
- Windows Vista और उसके बाद के संस्करण `%Windir%\System32\Config\RegBack\` में `HKEY_LOCAL_MACHINE` रजिस्ट्री फ़ाइलें बैकअप करते हैं।
- इसके अतिरिक्त, कार्यक्रम निष्पादन सूचना `%UserProfile%\{User}\AppData\Local\Microsoft\Windows\USERCLASS.DAT` में संग्रहित है Windows Vista और Windows 2008 सर्वर के बाद से।

### उपकरण

कुछ उपकरण रजिस्ट्री फ़ाइलों का विश्लेषण करने के लिए उपयोगी हैं:

* **रजिस्ट्री संपादक**: यह Windows में स्थापित है। यह वर्तमान सत्र की Windows रजिस्ट्री में नेविगेट करने के लिए एक GUI है।
* [**रजिस्ट्री एक्सप्लोरर**](https://ericzimmerman.github.io/#!index.md): यह आपको रजिस्ट्री फ़ाइल लोड करने और उनमें नेविगेट करने की अनुमति देता है एक GUI के साथ। यह बुकमार्क्स शामिल करता है जो दिलचस्प जानकारी वाले कुंजियों को हाइलाइट करता है।
* [**RegRipper**](https://github.com/keydet89/RegRipper3.0): फिर से, इसमें एक GUI है जो लोड किए गए रजिस्ट्री में नेविगेट करने की अनुमति देता है और लोड किए गए रजिस्ट्री में दिलचस्प जानकारी को हाइलाइट करने वाले प्लगइन्स भी शामिल हैं।
* [**Windows रजिस्ट्री रिकवरी**](https://www.mitec.cz/wrr.html): एक और GUI एप्लिकेशन जो रजिस्ट्री से महत्वपूर्ण जानकारी निकालने की क्षमता रखता है।

### हटाए गए तत्व को पुनः प्राप्त करना

जब एक कुंजी हटाई जाती है तो उसे ऐसा चिह्नित किया जाता है, लेकिन जब तक वह जगह जिसे वह धारण कर रही है आवश्यक नहीं होती, तब तक वह हटाया नहीं जाएगा। इसलिए, **रजिस्ट्री एक्सप्लोरर** जैसे उपकरण का उपयोग करके इन हटाए गए कुंजियों को पुनः प्राप्त किया जा सकता है।

### अंतिम लेखन समय

प्रत्येक कुंजी-मान में एक **समय चिह्नांक** होता है जो दर्शाता है कि आखिरी बार यह संशोधित किया गया था।

### SAM

फ़ाइल/हाइव **SAM** में सिस्टम के **उपयोगकर्ता, समूह और उपयोगकर्ता पासवर्ड** हैश संग्रहित होते हैं।

`SAM\Domains\Account\Users` में आप उपयोगकर्ता नाम, RID, अंतिम लॉगिन, अंतिम विफल लॉगिन, लॉगिन काउंटर, पासवर्ड नीति और खाता बनाया गया था की जानकारी प्राप्त कर सकते हैं। **हैश** प्राप्त करने के लिए आपको फ़ाइल/हाइव **SYSTEM** की भी **आवश्यकता** है।

### Windows रजिस्ट्री में दिलचस्प प्रविष्टियाँ

{% content-ref url="interesting-windows-registry-keys.md" %}
[interesting-windows-registry-keys.md](interesting-windows-registry-keys.md)
{% endcontent-ref %}

## निष्पादित कार्यक्रम

### मौलिक Windows प्रक्रियाएँ

[इस पोस्ट](https://jonahacks.medium.com/investigating-common-windows-processes-18dee5f97c1d) में आप संदेहात्मक व्यवहार का पता लगाने के लिए सामान्य Windows प्रक्रियाओं के बारे में सीख सकते हैं।

### Windows हाल के APPs

रजिस्ट्री `NTUSER.DAT` में पथ `Software\Microsoft\Current Version\Search\RecentApps` में आप **निष्पादित एप्लिकेशन**, **अंतिम समय** जब यह निष्पादित किया गया था, और **कितनी बार** यह चलाया गया था के बारे में जानकारी संग्रहित करने वाली सबकीज हो सकती है।

### BAM (पृष्ठभूमि गतिविधि नियंत्रक)

आप रजिस्ट्री संपादक के साथ `SYSTEM` फ़ाइल खोल सकते हैं और पथ `SYSTEM\CurrentControlSet\Services\bam\UserSettings\{SID}` में आप प्रत्येक उपयोगकर्ता द्वारा निष्पादित **एप्लिकेशन की जानकारी** (पथ में `{SID}` ध्यान दें) और **कब** वे निष्पादित किए गए थे (समय रजिस्ट्री के डेटा मान में होता है) प्राप्त कर सकते हैं।

### Windows Prefetch

प्रीफेचिंग एक तकनीक है जो एक कंप्यूटर को चुपचाप **उसे आवश्यक संसाधन लाने की अनुमति देती है जो एक उपयोगकर्ता निकट भविष्य में पहुँच सकता है** ताकि संसाधन तेजी से पहुँचे जा सकें।

Windows प्रीफेच में **निष्पादित कार्यक्रमों के कैश** बनाने से उन्हें तेजी से लोड करने की क्षमता होती है। ये कैशेज `.pf` फ़ाइलें बनाते हैं पथ में: `C:\Windows\Prefetch`। XP/VISTA/WIN7 में 128 फ़ाइलों की सीमा होती है और Win8/Win10 में 1024 फ़ाइलें होती हैं।

फ़ाइल नाम `{प्रोग्राम_नाम}-{हैश}.pf` के रूप में बनाई जाती है (हैश पथ और कार्यक्रम के तर्कों पर आधारित है)। W10 में ये फ़ाइलें संकुचित होती हैं। ध्यान दें कि फ़ाइल की केवल मौज
```bash
.\PECmd.exe -d C:\Users\student\Desktop\Prefetch --html "C:\Users\student\Desktop\out_folder"
```
![](<../../../.gitbook/assets/image (487).png>)

### सुपरप्रीफेच

**सुपरप्रीफेच** का उसी लक्ष्य है जैसे प्रीफेच, **प्रोग्राम्स को तेजी से लोड करना** अगले क्या लोड होने वाला है का पूर्वानुमान करके। हालांकि, यह प्रीफेच सेवा की जगह नहीं लेता है।\
यह सेवा डेटाबेस फ़ाइल्स उत्पन्न करेगी `C:\Windows\Prefetch\Ag*.db` में।

इन डेटाबेस में आप **प्रोग्राम** का **नाम**, **एक्जीक्यूशन्स** की **संख्या**, **फ़ाइलें खोली गई**, **वॉल्यूम एक्सेस**, **पूरा पथ**, **समयअंतर** और **टाइमस्टैम्प्स** देख सकते हैं।

आप इस जानकारी तक पहुंच सकते हैं उपकरण [**CrowdResponse**](https://www.crowdstrike.com/resources/community-tools/crowdresponse/) का उपयोग करके।

### SRUM

**सिस्टम रिसोर्स यूज़ेज मॉनिटर** (SRUM) **प्रक्रिया द्वारा उपयोग किए गए संसाधनों** का **निगरानी** करता है। यह W8 में प्रकट हुआ और डेटा को `C:\Windows\System32\sru\SRUDB.dat` में स्थित एक ESE डेटाबेस में संग्रहित करता है।

यह निम्नलिखित जानकारी प्रदान करता है:

* एप्लिकेशन आईडी और पथ
* प्रक्रिया को चलाने वाला उपयोगकर्ता
* भेजे गए बाइट्स
* प्राप्त बाइट्स
* नेटवर्क इंटरफेस
* कनेक्शन अवधि
* प्रक्रिया अवधि

यह जानकारी हर 60 मिनट में अपडेट होती है।

आप इस फ़ाइल से डेटा प्राप्त कर सकते हैं उपकरण [**srum\_dump**](https://github.com/MarkBaggett/srum-dump) का उपयोग करके।
```bash
.\srum_dump.exe -i C:\Users\student\Desktop\SRUDB.dat -t SRUM_TEMPLATE.xlsx -o C:\Users\student\Desktop\srum
```
### AppCompatCache (ShimCache)

**AppCompatCache**, जिसे **ShimCache** भी कहा जाता है, **Microsoft** द्वारा विकसित **Application Compatibility Database** का हिस्सा बनता है जो एप्लिकेशन संगतता समस्याओं का सामना करने के लिए है। यह सिस्टम कॉम्पोनेंट विभिन्न फ़ाइल मेटाडेटा को रिकॉर्ड करता है, जिसमें शामिल हैं:

- फ़ाइल का पूरा पथ
- फ़ाइल का आकार
- Last Modified time under **$Standard\_Information** (SI)
- Last Updated time of the ShimCache
- Process Execution Flag

Such data is stored within the registry at specific locations based on the version of the operating system:

- For XP, the data is stored under `SYSTEM\CurrentControlSet\Control\SessionManager\Appcompatibility\AppcompatCache` with a capacity for 96 entries.
- उपसर्वर 2003 के लिए, साथ ही Windows संस्करण 2008, 2012, 2016, 7, 8, और 10 के लिए, स्टोरेज पथ `SYSTEM\CurrentControlSet\Control\SessionManager\AppcompatCache\AppCompatCache` है, जिसमें 512 और 1024 एंट्री को समाहित किया गया है।

संग्रहित जानकारी को पार्स करने के लिए, [**AppCompatCacheParser** टूल](https://github.com/EricZimmerman/AppCompatCacheParser) का उपयोग करना सुझावित है।

![](<../../../.gitbook/assets/image (488).png>)

### Amcache

**Amcache.hve** फ़ाइल मुख्य रूप से एक रजिस्ट्री हाइव है जो एक सिस्टम पर निष्पादित की गई एप्लिकेशनों के विवरण को लॉग करता है। आमतौर पर यह `C:\Windows\AppCompat\Programas\Amcache.hve` पर पाया जाता है।

यह फ़ाइल हाल ही में निष्पादित प्रक्रियाओं के रिकॉर्ड संग्रहित करने के लिए महत्वपूर्ण है, जिसमें निष्पादनीय फ़ाइलों के पथ और उनके SHA1 हैश शामिल हैं। यह जानकारी सिस्टम पर एप्लिकेशनों की गतिविधि का ट्रैकिंग करने के लिए अमूल्य है।

**Amcache.hve** से डेटा निकालने और विश्लेषण करने के लिए, [**AmcacheParser**](https://github.com/EricZimmerman/AmcacheParser) टूल का उपयोग किया जा सकता है। निम्नलिखित कमांड एक उदाहरण है कि AmcacheParser का उपयोग कैसे किया जाए **Amcache.hve** फ़ाइल की सामग्री को पार्स करने और परिणामों को CSV प्रारूप में निकालने के लिए:
```bash
AmcacheParser.exe -f C:\Users\genericUser\Desktop\Amcache.hve --csv C:\Users\genericUser\Desktop\outputFolder
```
में उत्पन्न CSV फ़ाइलों में, `Amcache_Unassociated file entries` विशेष रूप से महत्वपूर्ण है क्योंकि यह असंबद्ध फ़ाइल प्रविष्टियों के बारे में विविध जानकारी प्रदान करता है।

सबसे दिलचस्प CVS फ़ाइल जो उत्पन्न होती है, वह है `Amcache_Unassociated file entries`।

### RecentFileCache

इस आर्टिफैक्ट को केवल W7 में `C:\Windows\AppCompat\Programs\RecentFileCache.bcf` में पाया जा सकता है और यह कुछ बाइनरी के हाल ही में निष्पादन के बारे में जानकारी रखता है।

आप उपकरण [**RecentFileCacheParse**](https://github.com/EricZimmerman/RecentFileCacheParser) का उपयोग फ़ाइल को पार्स करने के लिए कर सकते हैं।

### निर्धारित कार्य

आप इन्हें `C:\Windows\Tasks` या `C:\Windows\System32\Tasks` से निकाल सकते हैं और उन्हें XML के रूप में पढ़ सकते हैं।

### सेवाएं

आप उन्हें रजिस्ट्री में `SYSTEM\ControlSet001\Services` के तहत पा सकते हैं। आप देख सकते हैं कि क्या निष्पादित किया जा रहा है और कब।

### **Windows Store**

स्थापित एप्लिकेशन `\ProgramData\Microsoft\Windows\AppRepository\` में पाए जा सकते हैं।
इस भंडार में एक **लॉग** है जिसमें प्रत्येक एप्लिकेशन की स्थापना की गई है जो तंत्र में है डेटाबेस **`StateRepository-Machine.srd`**।

इस डेटाबेस के एप्लिकेशन तालिका में, "एप्लिकेशन आईडी", "पैकेज नंबर", और "प्रदर्शन नाम" मिल सकते हैं। ये स्तंभ पूर्व स्थापित और स्थापित एप्लिकेशनों के बारे में जानकारी रखते हैं और यह पता लगाया जा सकता है कि क्या कुछ एप्लिकेशन अनइंस्टॉल किए गए थे क्योंकि स्थापित एप्लिकेशनों के आईडी सतत होने चाहिए।

रजिस्ट्री पथ में भी स्थापित एप्लिकेशन पाए जा सकते हैं: `Software\Microsoft\Windows\CurrentVersion\Appx\AppxAllUserStore\Applications\`\
और **अनइंस्टॉल** **एप्लिकेशन** में: `Software\Microsoft\Windows\CurrentVersion\Appx\AppxAllUserStore\Deleted\`

## Windows घटनाएँ

Windows घटनाओं में जो जानकारी दिखाई देती है:

* क्या हुआ
* टाइमस्टैम्प (UTC + 0)
* शामिल हुए उपयोगकर्ता
* शामिल हुए होस्ट (होस्टनाम, आईपी)
* पहुंचे गए संपत्तियाँ (फ़ाइलें, फ़ोल्डर, प्रिंटर, सेवाएँ)

लॉग `C:\Windows\System32\config` में पहले Windows Vista से पहले और `C:\Windows\System32\winevt\Logs` में Windows Vista के बाद स्थित हैं। Windows Vista से पहले, इवेंट लॉग बाइनरी प्रारूप में थे और इसके बाद, वे **XML प्रारूप** में हैं और **.evtx** एक्सटेंशन का उपयोग करते हैं।

इवेंट फ़ाइलों की स्थिति सिस्टम रजिस्ट्री में मिल सकती है **`HKLM\SYSTEM\CurrentControlSet\services\EventLog\{Application|System|Security}`**

इन्हें Windows इवेंट व्यूअर (**`eventvwr.msc`**) से देखा जा सकता है या [**इवेंट लॉग एक्सप्लोरर**](https://eventlogxp.com) **या** [**Evtx एक्सप्लोरर/EvtxECmd**](https://ericzimmerman.github.io/#!index.md)** जैसे अन्य उपकरणों के साथ।**

## Windows सुरक्षा इवेंट लॉगिंग को समझना

सुरक्षा विन्यास फ़ाइल में एक्सेस इवेंट रिकॉर्ड किए जाते हैं जो `C:\Windows\System32\winevt\Security.evtx` में स्थित है। इस फ़ाइल का आकार समायोज्य है, और जब इसकी क्षमता पूरी हो जाती है, पुराने इवेंट ओवरराइट हो जाते हैं। रिकॉर्ड किए गए इवेंट में उपयोगकर्ता लॉगिन और लॉगऑफ, उपयोगकर्ता क्रियाएँ, और सुरक्षा सेटिंग्स में परिवर्तनों के साथ, साथ ही फ़ाइल, फ़ोल्डर, और साझा संपत्ति एक्सेस भी शामिल हैं।

### उपयोगकर्ता प्रमाणीकरण के लिए मुख्य इवेंट आईडी:

- **EventID 4624**: उपयोगकर्ता सफलतापूर्वक प्रमाणीकृत है।
- **EventID 4625**: प्रमाणीकरण विफलता की संकेत देता है।
- **EventIDs 4634/4647**: उपयोगकर्ता लॉगऑफ इवेंट को प्रस्तुत करता है।
- **EventID 4672**: प्रबंधनीय विशेषाधिकारों के साथ लॉगिन को दर्शाता है।
#### सिस्टम पावर इवेंट्स

इवेंटआईडी 6005 सिस्टम स्टार्टअप को दर्शाता है, जबकि इवेंटआईडी 6006 शटडाउन को चिह्नित करता है।

#### लॉग हटाना

सिक्योरिटी इवेंटआईडी 1102 लॉग को हटाने की सूचना देता है, जो फोरेंसिक विश्लेषण के लिए एक महत्वपूर्ण इवेंट है।

<figure><img src="https://pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}
