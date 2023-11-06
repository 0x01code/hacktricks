<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित करना** चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने** की पहुंच चाहिए? [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह

- प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में PR जमा करके साझा करें।**

</details>


# टाइमस्टैम्प

एक हमलावर को फ़ाइलों के टाइमस्टैम्प को **बदलने** के लिए रुचि हो सकती है ताकि उसे पकड़ा न जाए।\
टाइमस्टैम्प को **MFT** में एट्रिब्यूट्स `$STANDARD_INFORMATION` __ और __ `$FILE_NAME` में ढूंढ़ा जा सकता है।

दोनों एट्रिब्यूट्स में 4 टाइमस्टैम्प होते हैं: **संशोधन**, **पहुंच**, **सृजन** और **MFT रजिस्ट्री संशोधन** (MACE या MACB)।

**Windows explorer** और अन्य उपकरण **`$STANDARD_INFORMATION`** से जानकारी दिखाते हैं।

## टाइमस्टॉंप - एंटी-फोरेंसिक टूल

यह टूल **`$STANDARD_INFORMATION`** के भीतर टाइमस्टैम्प जानकारी को **संशोधित** करता है **लेकिन** **`$FILE_NAME`** के भीतर जानकारी को **नहीं**। इसलिए, यह संदेहास्पद **गतिविधि की पहचान** करना संभव है।

## Usnjrnl

**USN Journal** (Update Sequence Number Journal) या Change Journal, Windows NT फ़ाइल सिस्टम (NTFS) की एक सुविधा है जो वॉल्यूम पर किए गए बदलावों का एक रिकॉर्ड बनाए रखती है।\
इस रिकॉर्ड के संशोधनों की खोज के लिए उपकरण [**UsnJrnl2Csv**](https://github.com/jschicht/UsnJrnl2Csv) का उपयोग किया जा सकता है।

![](<../../.gitbook/assets/image (449).png>)

पिछली छवि टूल द्वारा दिखाई गई **आउटपुट** है जहां देखा जा सकता है कि कुछ **परिवर्तन किए गए** हैं।

## $LogFile

फ़ाइल सिस्टम पर सभी मेटाडेटा परिवर्तनों को लॉग किया जाता है ताकि सिस्टम क्रैश के बाद महत्वपूर्ण फ़ाइल सिस्टम संरचनाओं का संगत पुनर्स्थापन सुनिश्चित किया जा सके। इसे [write-ahead logging](https://en.wikipedia.org/wiki/Write-ahead\_logging) कहा जाता है।\
लॉग किए गए मेटाडेटा को "डॉलर लॉगफ़ाइल" नामक एक फ़ाइल में संग्रहीत किया जाता है, जो NTFS फ़ाइल सिस्टम के एक रूट निर्देशिका में पाई जाती है।\
इस फ़ाइल को पार्स करने और परिवर्तनों को खोजने के लिए [LogFileParser](https://github.com/jschicht/LogFileParser) जैसे उपकरण का उपयोग किया जा सकता है।

![](<../../.gitbook/assets/image (450).png>)

फिर से, टूल के आउटपुट में देखा जा सकता है कि **कुछ परिवर्तन किए गए** हैं।

एक ही उपकरण का उपयोग करके यह पहचाना जा सकता है कि **टाइमस्टैम्प को किस समय संशोधित किया गया था**:

![](<../../.gitbook/assets/image (451).png>)

* CTIME: फ़ाइल का निर्माण समय
* ATIME: फ़ाइल का संशोधन समय
* MTIME: फ़ाइल का MFT रजिस्ट्री संशोधन
* RTIME: फ़ाइल का पहुंच समय

## `$STANDARD_INFORMATION` और `$FILE_NAME` तुलना

संदेहास्पद संशोधित फ़ाइलों की पहचान करने का एक और तरीका होगा कि दोनों एट्रिब्यूट्स पर समय की तुलना की जाए, जहां **मिलान नहीं होता**
# सुरक्षित हटाना

[https://github.com/Claudio-C/awesome-data-sanitization](https://github.com/Claudio-C/awesome-data-sanitization)

# विंडोज कॉन्फ़िगरेशन

यह संभव है कि विंडोज लॉगिंग विधियों को अक्षम करके फॉरेंसिक्स जांच को कठिन बनाया जा सके।

## टाइमस्टैम्प्स को अक्षम करें - UserAssist

यह एक रजिस्ट्री की है जो प्रत्येक एक्ज़ीक्यूटेबल को उपयोगकर्ता द्वारा चलाया गया था जब तारीख और घंटे को बनाए रखती है।

UserAssist को अक्षम करने के लिए दो कदम होते हैं:

1. रजिस्ट्री की दो कुंजी सेट करें, `HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Advanced\Start_TrackProgs` और `HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Advanced\Start_TrackEnabled`, दोनों को शून्य सेट करें ताकि हमें UserAssist अक्षम करना है इसकी सूचना मिल सके।
2. `HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist\<hash>` जैसे दिखने वाले रजिस्ट्री सबट्री को साफ़ करें।

## टाइमस्टैम्प्स को अक्षम करें - Prefetch

यह विंडोज सिस्टम के प्रदर्शन को सुधारने का उद्देश्य रखने वाले एप्लिकेशन्स के बारे में जानकारी सहेजेगा। हालांकि, यह फॉरेंसिक्स प्रथाओं के लिए भी उपयोगी हो सकता है।

* `regedit` चलाएं
* फ़ाइल पथ `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SessionManager\Memory Management\PrefetchParameters` का चयन करें
* `EnablePrefetcher` और `EnableSuperfetch` पर दायां क्लिक करें
* इनमें से प्रत्येक को संशोधित करने के लिए 1 (या 3) से 0 मान बदलें
* पुनरारंभ करें

## टाइमस्टैम्प्स को अक्षम करें - अंतिम पहुँच समय

जब एक फ़ोल्डर को विंडोज़ एनटीएफ़एस वॉल्यूम पर खोला जाता है, तो सिस्टम प्रत्येक सूचीबद्ध फ़ोल्डर पर एक टाइमस्टैम्प फ़ील्ड को अद्यतित करने के लिए समय लेता है, जिसे अंतिम पहुँच समय कहा जाता है। एक अधिक प्रयोग होने वाले एनटीएफ़एस वॉल्यूम पर, यह प्रदर्शन प्रभावित कर सकता है।

1. रजिस्ट्री संपादक (Regedit.exe) खोलें।
2. `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\FileSystem` तक ब्राउज़ करें।
3. `NtfsDisableLastAccessUpdate` ढूंढें। यदि यह मौजूद नहीं है, तो इस DWORD को जोड़ें और इसके मान को 1 सेट करें, जिससे प्रक्रिया अक्षम हो जाएगी।
4. रजिस्ट्री संपादक बंद करें और सर्वर को पुनरारंभ करें।

## USB इतिहास को हटाएं

सभी **USB डिवाइस एंट्री** विंडोज़ रजिस्ट्री में **USBSTOR** रजिस्ट्री कुंजी के तहत संग्रहित होती है, जिसमें उपकरण को अपने पीसी या लैपटॉप में प्लग करने पर उत्पन्न सबकुंजी होती है। आप इस कुंजी को यहां पाएंगे H`KEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Enum\USBSTOR`। **इसे हटाने** से आप USB इतिहास को हटा देंगे।\
आप इसे हटाने के लिए उपकरण [**USBDeview**](https://www.nirsoft.net/utils/usb\_devices\_view.html) भी उपयोग कर सकते हैं (और इसे हटाने के लिए)।

एक और फ़ाइल जो USB के बारे में जानकारी सहेजती है, वह है `C:\Windows\INF` के अंदर `setupapi.dev.log` फ़ाइल। इसे भी हटा देना चाहिए।

## शैडो कॉपियों को अक्षम करें

`vssadmin list shadowstorage` के साथ **शैडो कॉपियों की सूची** बनाएं\
`vssadmin delete shadow` चलाकर उन्हें **हटा दें**

आप इन्हें GUI के माध्यम से भी हटा सकते हैं, जिसके लिए [https://www.ubackup.com/windows-10/how-to-delete-shadow-copies-windows-10-5740.html](https://www.ubackup.com/windows-10/how-to-delete-shadow-copies-windows-10-5740.html) में प्रस्तावित चरणों का पालन करें

शैडो कॉपियों को अक्षम करने के लिए:

1. विंडोज़ स्टार्ट बटन पर जाएं और "सेवाएं" टेक्स्ट खोज बॉक्स में टाइप करें; सेवाएं प्रोग्राम खोलें।
2. स
