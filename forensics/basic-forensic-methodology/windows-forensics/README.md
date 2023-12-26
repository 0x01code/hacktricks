# Windows Artifacts

## Windows Artifacts

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे**? या क्या आप **PEASS के नवीनतम संस्करण तक पहुँचना चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* **[**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) में शामिल हों या [**telegram समूह**](https://t.me/peass) या **Twitter पर** मुझे **फॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **hacktricks repo** और [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>

## Generic Windows Artifacts

### Windows 10 Notifications

पथ `\Users\<username>\AppData\Local\Microsoft\Windows\Notifications` में आप डेटाबेस `appdb.dat` (Windows anniversary से पहले) या `wpndatabase.db` (Windows Anniversary के बाद) पा सकते हैं।

इस SQLite डेटाबेस के अंदर, आप `Notification` टेबल में सभी नोटिफिकेशन (XML प्रारूप में) पा सकते हैं जिसमें दिलचस्प डेटा हो सकता है।

### Timeline

Timeline एक Windows विशेषता है जो वेब पेजों के दौरे, संपादित दस्तावेज़ों, और निष्पादित अनुप्रयोगों का **कालानुक्रमिक इतिहास** प्रदान करती है।

डेटाबेस पथ `\Users\<username>\AppData\Local\ConnectedDevicesPlatform\<id>\ActivitiesCache.db` में स्थित है। इस डेटाबेस को SQLite टूल के साथ या [**WxTCmd**](https://github.com/EricZimmerman/WxTCmd) टूल के साथ खोला जा सकता है **जो 2 फाइलें उत्पन्न करता है जिन्हें [**TimeLine Explorer**](https://ericzimmerman.github.io/#!index.md) टूल के साथ खोला जा सकता है।**

### ADS (Alternate Data Streams)

डाउनलोड की गई फाइलें **ADS Zone.Identifier** युक्त हो सकती हैं जो इंगित करती हैं कि फाइल को इंट्रानेट, इंटरनेट, आदि से **कैसे** **डाउनलोड** किया गया था। कुछ सॉफ्टवेयर (जैसे ब्राउज़र) आमतौर पर **और भी** **जानकारी** जोड़ते हैं जैसे कि फाइल कहाँ से डाउनलोड की गई थी उसका **URL**।

## **File Backups**

### Recycle Bin

Vista/Win7/Win8/Win10 में **Recycle Bin** ड्राइव की जड़ में फोल्डर **`$Recycle.bin`** में पाया जा सकता है (`C:\$Recycle.bin`).\
जब इस फोल्डर में एक फाइल को हटाया जाता है तो 2 विशिष्ट फाइलें बनाई जाती हैं:

* `$I{id}`: फाइल जानकारी (जब यह हटाया गया था उसकी तारीख)
* `$R{id}`: फाइल की सामग्री

![](<../../../.gitbook/assets/image (486).png>)

इन फाइलों को होने पर आप [**Rifiuti**](https://github.com/abelcheung/rifiuti2) टूल का उपयोग करके हटाए गए फाइलों के मूल पते और हटाने की तारीख प्राप्त कर सकते हैं (Vista – Win10 के लिए `rifiuti-vista.exe` का उपयोग करें)।
```
.\rifiuti-vista.exe C:\Users\student\Desktop\Recycle
```
### वॉल्यूम शैडो कॉपीज

शैडो कॉपी एक तकनीक है जो माइक्रोसॉफ्ट विंडोज में शामिल है जो कंप्यूटर फाइलों या वॉल्यूम्स की **बैकअप कॉपीज** या स्नैपशॉट्स बना सकती है, यहां तक कि जब वे इस्तेमाल में हों।

ये बैकअप आमतौर पर फाइल सिस्टम के रूट से `\System Volume Information` में स्थित होते हैं और नाम **UIDs** से बना होता है जैसा कि निम्नलिखित चित्र में दिखाया गया है:

![](<../../../.gitbook/assets/image (520).png>)

**ArsenalImageMounter** के साथ फोरेंसिक इमेज को माउंट करने के बाद, [**ShadowCopyView**](https://www.nirsoft.net/utils/shadow\_copy\_view.html) टूल का उपयोग करके शैडो कॉपी का निरीक्षण किया जा सकता है और यहां तक कि शैडो कॉपी बैकअप्स से **फाइलों को निकाला** जा सकता है।

![](<../../../.gitbook/assets/image (521).png>)

रजिस्ट्री प्रविष्टि `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\BackupRestore` में **बैकअप नहीं करने के लिए** फाइलें और कुंजियाँ होती हैं:

![](<../../../.gitbook/assets/image (522).png>)

रजिस्ट्री `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\VSS` में भी `Volume Shadow Copies` के बारे में कॉन्फ़िगरेशन जानकारी होती है।

### ऑफिस ऑटोसेव्ड फाइल्स

आप ऑफिस ऑटोसेव्ड फाइल्स को यहाँ पा सकते हैं: `C:\Usuarios\\AppData\Roaming\Microsoft{Excel|Word|Powerpoint}\`

## शेल आइटम्स

एक शेल आइटम एक ऐसी वस्तु होती है जिसमें दूसरी फाइल तक पहुँचने के लिए जानकारी होती है।

### हाल के दस्तावेज़ (LNK)

विंडोज **स्वचालित रूप से** ये **शॉर्टकट्स** बनाता है जब उपयोगकर्ता **खोलता है, इस्तेमाल करता है या फाइल बनाता है**:

* Win7-Win10: `C:\Users\\AppData\Roaming\Microsoft\Windows\Recent\`
* Office: `C:\Users\\AppData\Roaming\Microsoft\Office\Recent\`

जब एक फोल्डर बनाया जाता है, तो उस फोल्डर, माता-पिता फोल्डर, और दादा-दादी फोल्डर का भी लिंक बनाया जाता है।

ये स्वचालित रूप से बनाई गई लिंक फाइलें **मूल के बारे में जानकारी रखती हैं** जैसे कि यह एक **फाइल** है **या** एक **फोल्डर**, **MAC** **समय** उस फाइल का, **वॉल्यूम जानकारी** जहाँ फाइल संग्रहीत है और **लक्ष्य फाइल का फोल्डर**। यह जानकारी उन फाइलों को पुनः प्राप्त करने में उपयोगी हो सकती है यदि वे हटा दी गई हों।

साथ ही, लिंक फाइल की **बनाई गई तारीख** मूल फाइल की **पहली बार** **इस्तेमाल की गई** तारीख है और लिंक फाइल की **संशोधित तारीख** मूल फाइल की आखिरी बार इस्तेमाल की गई तारीख है।

इन फाइलों का निरीक्षण करने के लिए आप [**LinkParser**](http://4discovery.com/our-tools/) का उपयोग कर सकते हैं।

इस टूल में आपको **2 सेट** समयसीमा मिलेगी:

* **पहला सेट:**
1. FileModifiedDate
2. FileAccessDate
3. FileCreationDate
* **दूसरा सेट:**
1. LinkModifiedDate
2. LinkAccessDate
3. LinkCreationDate.

पहला सेट समयसीमा **फाइल स्वयं के समयसीमा का संदर्भ देता है**। दूसरा सेट **लिंक की गई फाइल के समयसीमा का संदर्भ देता है**।

आप विंडोज CLI टूल का उपयोग करके भी समान जानकारी प्राप्त कर सकते हैं: [**LECmd.exe**](https://github.com/EricZimmerman/LECmd)
```
LECmd.exe -d C:\Users\student\Desktop\LNKs --csv C:\Users\student\Desktop\LNKs
```
### Jumplists

ये वे हाल की फाइलें हैं जो प्रत्येक एप्लिकेशन के लिए इंगित की जाती हैं। यह **एप्लिकेशन द्वारा इस्तेमाल की गई हाल की फाइलों की सूची** है जिसे आप प्रत्येक एप्लिकेशन पर एक्सेस कर सकते हैं। ये **स्वचालित रूप से बनाई जा सकती हैं या कस्टम** हो सकती हैं।

स्वचालित रूप से बनाई गई **jumplists** `C:\Users\{username}\AppData\Roaming\Microsoft\Windows\Recent\AutomaticDestinations\` में संग्रहीत की जाती हैं। Jumplists का नाम `{id}.autmaticDestinations-ms` प्रारूप का अनुसरण करते हुए रखा जाता है जहां प्रारंभिक ID एप्लिकेशन की ID होती है।

कस्टम jumplists `C:\Users\{username}\AppData\Roaming\Microsoft\Windows\Recent\CustomDestination\` में संग्रहीत की जाती हैं और ये आमतौर पर एप्लिकेशन द्वारा बनाई जाती हैं क्योंकि फाइल के साथ कुछ **महत्वपूर्ण** हुआ होता है (शायद पसंदीदा के रूप में चिह्नित)

किसी भी jumplist का **निर्मित समय** इंगित करता है **पहली बार फाइल को एक्सेस किया गया था** और **संशोधित समय अंतिम बार**।

आप [**JumplistExplorer**](https://ericzimmerman.github.io/#!index.md) का उपयोग करके jumplists का निरीक्षण कर सकते हैं।

(_ध्यान दें कि JumplistExplorer द्वारा प्रदान किए गए समयांक jumplist फाइल स्वयं से संबंधित हैं_)

### Shellbags

[**इस लिंक का अनुसरण करें जानने के लिए कि shellbags क्या हैं।**](interesting-windows-registry-keys.md#shellbags)

## Windows USBs का उपयोग

यह संभव है कि एक USB डिवाइस के उपयोग की पहचान की जा सकती है निम्नलिखित के निर्माण के लिए धन्यवाद:

* Windows Recent Folder
* Microsoft Office Recent Folder
* Jumplists

ध्यान दें कि कुछ LNK फाइल के बजाय मूल पथ की ओर इंगित करने के लिए, WPDNSE फोल्डर की ओर इंगित करती है:

### Registry Information

[इस पृष्ठ की जांच करें](interesting-windows-registry-keys.md#usb-information) जो रजिस्ट्री कुंजियाँ USB से जुड़े उपकरणों के बारे में दिलचस्प जानकारी रखती हैं।

### setupapi

फाइल `C:\Windows\inf\setupapi.dev.log` की जांच करें USB कनेक्शन के बारे में समयांक प्राप्त करने के लिए (खोज के लिए `Section start`).

### USB Detective

[**USBDetective**](https://usbdetective.com) का उपयोग करके उन USB उपकरणों के बारे में जानकारी प्राप्त की जा सकती है जो एक इमेज से जुड़े हुए हैं।

### Plug and Play Cleanup

'Plug and Play Cleanup' निर्धारित कार्य ड्राइवरों के पुराने संस्करणों को **साफ** करने के लिए जिम्मेदार है। यह प्रतीत होता है (ऑनलाइन रिपोर्टों के आधार पर) कि यह **30 दिनों में इस्तेमाल नहीं किए गए ड्राइवरों को भी उठाता है**, इसके विवरण के बावजूद जो कहता है कि "प्रत्येक ड्राइवर पैकेज का सबसे वर्तमान संस्करण रखा जाएगा"। इस प्रकार, **30 दिनों के लिए जुड़े नहीं गए हटाने योग्य उपकरणों के ड्राइवरों को हटाया जा सकता है**।

निर्धारित कार्य स्वयं 'C:\Windows\System32\Tasks\Microsoft\Windows\Plug and Play\Plug and Play Cleanup' पर स्थित है, और इसकी सामग्री नीचे प्रदर्शित की गई है:

### Emails

ईमेल में **2 दिलचस्प भाग होते हैं: हेडर्स और ईमेल की सामग्री**। **हेडर्स** में आप जानकारी पा सकते हैं जैसे:

* **किसने** ईमेल भेजे (ईमेल पता, IP, मेल सर्वर जिन्होंने ईमेल को रीडायरेक्ट किया)
* **कब** ईमेल भेजा गया था

साथ ही, `References` और `In-Reply-To` हेडर्स में आप संदेशों की ID पा सकते हैं:

### Windows Mail App

यह एप्लिकेशन ईमेल को HTML या टेक्स्ट में सेव करता है। आप `\Users\<username>\AppData\Local\Comms\Unistore\data\3\` के अंदर सबफोल्डर्स में ईमेल पा सकते हैं। ईमेल `.dat` एक्सटेंशन के साथ सेव किए जाते हैं।

ईमेल की **मेटाडेटा** और **संपर्क** **EDB डेटाबेस** के अंदर पाए जा सकते हैं: `\Users\<username>\AppData\Local\Comms\UnistoreDB\store.vol`

**फाइल के एक्सटेंशन को बदलें** `.vol` से `.edb` में और आप [ESEDatabaseView](https://www.nirsoft.net/utils/ese\_database\_view.html) टूल का उपयोग करके इसे खोल सकते हैं। `Message` टेबल के अंदर आप ईमेल देख सकते हैं।

### Microsoft Outlook

जब Exchange सर्वर या Outlook क्लाइंट्स का उपयोग किया जाता है तो कुछ MAPI हेडर्स होंगे:

* `Mapi-Client-Submit-Time`: सिस्टम का समय जब ईमेल भेजा गया था
* `Mapi-Conversation-Index`: थ्रेड के बच्चों के संदेशों की संख्या और थ्रेड के प्रत्येक संदेश का समयांक
* `Mapi-Entry-ID`: संदेश पहचानकर्ता।
* `Mappi-Message-Flags` और `Pr_last_Verb-Executed`: MAPI क्लाइंट के बारे में जानकारी (संदेश पढ़ा गया? नहीं पढ़ा गया? जवाब दिया गया? रीडायरेक्ट किया गया? ऑफिस से बाहर?)

Microsoft Outlook क्लाइंट में, सभी भेजे/प्राप्त संदेश, संपर्क डेटा, और कैलेंडर डेटा एक PST फाइल में संग्रहीत किए जाते हैं:

* `%USERPROFILE%\Local Settings\Application Data\Microsoft\Outlook` (WinXP)
* `%USERPROFILE%\AppData\Local\Microsoft\Outlook`

रजिस्ट्री पथ `HKEY_CURRENT_USER\Software\Microsoft\WindowsNT\CurrentVersion\Windows Messaging Subsystem\Profiles\Outlook` इंगित करता है कि कौन सी फाइल इस्तेमाल की जा रही है।

आप [**Kernel PST Viewer**](https://www.nucleustechnologies.com/es/visor-de-pst.html) टूल का उपयोग करके PST फाइल को खोल सकते हैं।

### Outlook OST

जब Microsoft Outlook को **IMAP** का उपयोग करके या एक **Exchange** सर्वर का उपयोग करके कॉन्फ़िगर किया जाता है, तो यह एक **OST** फाइल उत्पन्न करता है जो लगभग वही जानकारी PST फाइल के रूप में संग्रहीत करता है। यह फाइल सर्वर के साथ **पिछले 12 महीनों** के लिए सिंक्रनाइज़ रखती है, एक **अधिकतम फाइल-आकार 50GB** के साथ और **वही फोल्डर में जहां PST** फाइल सेव की जाती है। आप इस फाइल का निरीक्षण [**Kernel OST viewer**](https://www.nucleustechnologies.com/ost-viewer.html) का उपयोग करके कर सकते हैं।

### Recovering Attachments

आप उन्हें फोल्डर में पा सकते हैं:

* `%APPDATA%\Local\Microsoft\Windows\Temporary Internet Files\Content.Outlook` -> IE10
* `%APPDATA%\Local\Microsoft\InetCache\Content.Outlook` -> IE11+

### Thunderbird MBOX

**Thunderbird** जानकारी को **MBOX** **फाइलों** में संग्रहीत करता है फोल्डर `\Users\%USERNAME%\AppData\Roaming\Thunderbird\Profiles` में।

## Thumbnails

जब एक उपयोगकर्ता किसी फोल्डर तक पहुंचता है और इसे थंबनेल्स का उपयोग करके व्यवस्थित करता है, तो एक `thumbs.db` फाइल बनाई जाती है। यह db **फोल्डर की छवियों के थंबनेल्स को संग्रहीत करता है** यहां तक कि अगर वे हटा दिए जाते हैं। WinXP और Win 8-8.1 में यह फाइल स्वचालित रूप से बनाई जाती है। Win7/Win10 में, यह स्वचालित रूप से बनाई जाती है अगर यह UNC पथ (\IP\folder...) के माध्यम स
```bash
.\PECmd.exe -d C:\Users\student\Desktop\Prefetch --html "C:\Users\student\Desktop\out_folder"
```
### Superprefetch

**Superprefetch** का उद्देश्य भी prefetch की तरह है, **प्रोग्राम्स को तेजी से लोड करना** यह अनुमान लगाकर कि अगला क्या लोड होने वाला है। हालांकि, यह prefetch सेवा का विकल्प नहीं है।\
यह सेवा `C:\Windows\Prefetch\Ag*.db` में डेटाबेस फाइलें उत्पन्न करती है।

इन डेटाबेस में आप **प्रोग्राम** का **नाम**, **संख्या** **चलाने की**, **खोली गई** **फाइलें**, **पहुंची गई** **वॉल्यूम**, **पूरा** **पथ**, **समय सीमाएं** और **टाइमस्टैम्प्स** पा सकते हैं।

आप इस जानकारी को [**CrowdResponse**](https://www.crowdstrike.com/resources/community-tools/crowdresponse/) टूल का उपयोग करके प्राप्त कर सकते हैं।

### SRUM

**System Resource Usage Monitor** (SRUM) **प्रक्रिया द्वारा उपभोग किए गए** **संसाधनों** की **निगरानी** करता है। यह W8 में आया था और यह डेटा को `C:\Windows\System32\sru\SRUDB.dat` में स्थित ESE डेटाबेस में संग्रहीत करता है।

यह निम्नलिखित जानकारी देता है:

* AppID और पथ
* उपयोगकर्ता जिसने प्रक्रिया चलाई
* भेजे गए बाइट्स
* प्राप्त बाइट्स
* नेटवर्क इंटरफेस
* कनेक्शन अवधि
* प्रक्रिया अवधि

यह जानकारी हर 60 मिनट में अपडेट होती है।

आप इस फाइल से डेटा [**srum\_dump**](https://github.com/MarkBaggett/srum-dump) टूल का उपयोग करके प्राप्त कर सकते हैं।
```bash
.\srum_dump.exe -i C:\Users\student\Desktop\SRUDB.dat -t SRUM_TEMPLATE.xlsx -o C:\Users\student\Desktop\srum
```
### AppCompatCache (ShimCache)

**Shimcache**, जिसे **AppCompatCache** के नाम से भी जाना जाता है, **Application Compatibility Database** का एक घटक है, जिसे **Microsoft** द्वारा बनाया गया था और जिसका उपयोग ऑपरेटिंग सिस्टम द्वारा एप्लिकेशन संगतता समस्याओं की पहचान के लिए किया जाता है।

कैश में ऑपरेटिंग सिस्टम के आधार पर विभिन्न फाइल मेटाडेटा संग्रहीत होते हैं, जैसे कि:

* फाइल पूरा पथ
* फाइल आकार
* **$Standard\_Information** (SI) अंतिम संशोधित समय
* ShimCache अंतिम अपडेट समय
* प्रोसेस निष्पादन ध्वज

यह जानकारी रजिस्ट्री में निम्नलिखित स्थानों पर मिल सकती है:

* `SYSTEM\CurrentControlSet\Control\SessionManager\Appcompatibility\AppcompatCache`
* XP (96 प्रविष्टियाँ)
* `SYSTEM\CurrentControlSet\Control\SessionManager\AppcompatCache\AppCompatCache`
* Server 2003 (512 प्रविष्टियाँ)
* 2008/2012/2016 Win7/Win8/Win10 (1024 प्रविष्टियाँ)

आप इस जानकारी को पार्स करने के लिए [**AppCompatCacheParser**](https://github.com/EricZimmerman/AppCompatCacheParser) टूल का उपयोग कर सकते हैं।

![](<../../../.gitbook/assets/image (488).png>)

### Amcache

**Amcache.hve** फाइल एक रजिस्ट्री फाइल है जो निष्पादित एप्लिकेशनों की जानकारी संग्रहीत करती है। यह `C:\Windows\AppCompat\Programas\Amcache.hve` में स्थित है।

**Amcache.hve** हाल ही में चलाए गए प्रोसेसों का रिकॉर्ड रखता है और उन फाइलों का पथ सूचीबद्ध करता है जिन्हें निष्पादित किया गया है, जिसका उपयोग निष्पादित किए गए प्रोग्राम का पता लगाने के लिए किया जा सकता है। यह प्रोग्राम का SHA1 भी रिकॉर्ड करता है।

आप इस जानकारी को पार्स करने के लिए [**Amcacheparser**](https://github.com/EricZimmerman/AmcacheParser) टूल का उपयोग कर सकते हैं।
```bash
AmcacheParser.exe -f C:\Users\student\Desktop\Amcache.hve --csv C:\Users\student\Desktop\srum
```
### RecentFileCache

यह आर्टिफैक्ट केवल W7 में `C:\Windows\AppCompat\Programs\RecentFileCache.bcf` में पाया जा सकता है और इसमें कुछ बाइनरीज़ के हालिया निष्पादन की जानकारी होती है।

आप फ़ाइल को पार्स करने के लिए [**RecentFileCacheParse**](https://github.com/EricZimmerman/RecentFileCacheParser) टूल का उपयोग कर सकते हैं।

### Scheduled tasks

आप इन्हें `C:\Windows\Tasks` या `C:\Windows\System32\Tasks` से निकाल सकते हैं और उन्हें XML के रूप में पढ़ सकते हैं।

### Services

आप इन्हें रजिस्ट्री के अंतर्गत `SYSTEM\ControlSet001\Services` में पा सकते हैं। आप देख सकते हैं कि क्या निष्पादित होने वाला है और कब।

### **Windows Store**

स्थापित एप्लिकेशन `\ProgramData\Microsoft\Windows\AppRepository\` में पाए जा सकते हैं।\
इस रिपॉजिटरी में डेटाबेस **`StateRepository-Machine.srd`** के अंदर सिस्टम में **प्रत्येक एप्लिकेशन स्थापित** के साथ एक **लॉग** होता है।

इस डेटाबेस के एप्लिकेशन टेबल में, "Application ID", "PackageNumber", और "Display Name" कॉलम पाए जा सकते हैं। इन कॉलमों में प्री-इंस्टॉल्ड और स्थापित एप्लिकेशनों की जानकारी होती है और यह भी पता चल सकता है कि कुछ एप्लिकेशन अनइंस्टॉल किए गए थे क्योंकि स्थापित एप्लिकेशनों के आईडी क्रमिक होने चाहिए।

रजिस्ट्री पथ में **स्थापित एप्लिकेशन** भी पाए जा सकते हैं: `Software\Microsoft\Windows\CurrentVersion\Appx\AppxAllUserStore\Applications\`\
और **अनइंस्टॉल्ड** **एप्लिकेशन** यहाँ पर हैं: `Software\Microsoft\Windows\CurrentVersion\Appx\AppxAllUserStore\Deleted\`

## Windows Events

Windows इवेंट्स के अंदर जो जानकारी आती है वह हैं:

* क्या हुआ
* Timestamp (UTC + 0)
* शामिल उपयोगकर्ता
* शामिल होस्ट्स (hostname, IP)
* पहुँचे गए एसेट्स (फ़ाइलें, फ़ोल्डर, प्रिंटर, सेवाएँ)

लॉग्स Windows Vista से पहले `C:\Windows\System32\config` में स्थित हैं और Windows Vista के बाद `C:\Windows\System32\winevt\Logs` में हैं। Windows Vista से पहले, इवेंट लॉग्स बाइनरी फॉर्मेट में थे और उसके बाद, वे **XML फॉर्मेट** में हैं और **.evtx** एक्सटेंशन का उपयोग करते हैं।

इवेंट फ़ाइलों का स्थान SYSTEM रजिस्ट्री में पाया जा सकता है **`HKLM\SYSTEM\CurrentControlSet\services\EventLog\{Application|System|Security}`**

उन्हें Windows Event Viewer (**`eventvwr.msc`**) से देखा जा सकता है या अन्य टूल्स जैसे कि [**Event Log Explorer**](https://eventlogxp.com) **या** [**Evtx Explorer/EvtxECmd**](https://ericzimmerman.github.io/#!index.md)** का उपयोग करके।**

### Security

यह एक्सेस इवेंट्स को रजिस्टर करता है और सुरक्षा कॉन्फ़िगरेशन की जानकारी देता है जो `C:\Windows\System32\winevt\Security.evtx` में पाई जा सकती है।

इवेंट फ़ाइल का **अधिकतम आकार** कॉन्फ़िगर किया जा सकता है, और जब अधिकतम आकार पहुँच जाता है तो यह पुरानी घटनाओं को ओवरराइट करना शुरू कर देगा।

रजिस्टर की गई घटनाएँ हैं:

* लॉगिन/लॉगऑफ
* उपयोगकर्ता की क्रियाएँ
* फ़ाइलों, फ़ोल्डरों और साझा एसेट्स तक पहुँच
* सुरक्षा कॉन्फ़िगरेशन का संशोधन

उपयोगकर्ता प्रमाणीकरण से संबंधित घटनाएँ:

| EventID   | विवरण                        |
| --------- | ---------------------------- |
| 4624      | सफल प्रमाणीकरण              |
| 4625      | प्रमाणीकरण त्रुटि           |
| 4634/4647 | लॉग ऑफ                      |
| 4672      | व्यवस्थापक अनुमतियों के साथ लॉगिन |

EventID 4634/4647 के अंदर दिलचस्प उप-प्रकार हैं:

* **2 (इंटरैक्टिव)**: लॉगिन इंटरैक्टिव था कीबोर्ड या VNC या `PSexec -U-` जैसे सॉफ़्टवेयर का उपयोग करके
* **3 (नेटवर्क)**: साझा फ़ोल्डर से कनेक्शन
* **4 (बैच)**: प्रक्रिया निष्पादित की गई
* **5 (सेवा)**: सेवा नियंत्रण प्रबंधक द्वारा सेवा शुरू की गई
* **6 (प्रॉक्सी):** प्रॉक्सी लॉगिन
* **7 (अनलॉक)**: पासवर्ड का उपयोग करके स्क्रीन अनब्लॉक की गई
* **8 (नेटवर्क क्लियरटेक्स्ट)**: उपयोगकर्ता ने स्पष्ट पाठ पासवर्ड भेजकर प्रमाणीकरण किया। यह घटना IIS से आती थी
* **9 (नई क्रेडेंशियल्स)**: यह तब उत्पन्न होता है जब `RunAs` कमांड का उपयोग किया जाता है या उपयोगकर्ता अलग क्रेडेंशियल्स के साथ नेटवर्क सेवा तक पहुँचता है।
* **10 (रिमोट इंटरैक्टिव)**: टर्मिनल सेवाओं या RDP के माध्यम से प्रमाणीकरण
* **11 (कैश इंटरैक्टिव)**: डोमेन कंट्रोलर से संपर्क नहीं हो पाने के कारण अंतिम कैश किए गए क्रेडेंशियल्स का उपयोग करके पहुँच
* **12 (कैश रिमोट इंटरैक्टिव)**: कैश किए गए क्रेडेंशियल्स के साथ दूरस्थ रूप से लॉगिन (10 और 11 का संयोजन)।
* **13 (कैश्ड अनलॉक)**: कैश किए गए क्रेडेंशियल्स के साथ लॉक किए गए मशीन को अनलॉक करना।

इस पोस्ट में, आप पा सकते हैं कि इन सभी प्रकार के लॉगिन को कैसे नकल करें और उनमें से किनमें आप मेमोरी से क्रेडेंशियल्स डंप कर पाएंगे: [https://www.alteredsecurity.com/post/fantastic-windows-logon-types-and-where-to-find-credentials-in-them](https://www.alteredsecurity.com/post/fantastic-windows-logon-types-and-where-to-find-credentials-in-them)

घटनाओं की स्थिति और उप-स्थिति की जानकारी घटना के कारणों के बारे में अधिक विवरण बता सकती है। उदाहरण के लिए, Event ID 4625 के निम्नलिखित स्थिति और उप-स्थिति कोड्स पर एक नज़र डालें:

![](<../../../.gitbook/assets/image (455).png>)

### Recovering Windows Events

संदिग्ध PC को **अनप्लग करके** बंद करना अत्यधिक सिफारिश की जाती है ताकि Windows इवेंट्स को रिकवर करने की संभावना को अधिकतम किया जा सके। अगर वे हटा दिए गए हों, तो उन्हें रिकवर करने के लिए उपयोगी टूल हो सकता है [**Bulk_extractor**](../partitions-file-systems-carving/file-data-carving-recovery-tools.md#bulk-extractor) जिसमें **evtx** एक्सटेंशन का संकेत दिया गया हो।

## Identifying Common Attacks with Windows Events

* [https://redteamrecipe.com/event-codes/](https://redteamrecipe.com/event-codes/)

### Brute Force Attack

एक ब्रूट फोर्स अटैक को आसानी से पहचाना जा सकता है क्योंकि **कई EventIDs 4625 दिखाई देंगे**। अगर अटैक **सफल** था, तो EventIDs 4625 के बाद, **एक EventID 4624 दिखाई देगा**।

### Time Change

यह फोरेंसिक टीम के लिए भयानक है क्योंकि सभी टाइमस्टैम्प्स मॉडिफाइड हो जाएंगे। यह घटना Security Event लॉग में EventID 4616 द्वारा रिकॉर्ड की जाती है।

### USB devices

निम्नलिखित System EventIDs उपयोगी हैं:

* 20001 / 20003 / 10000: पहली बार इस्तेमाल किया गया
* 10100: ड्राइ
