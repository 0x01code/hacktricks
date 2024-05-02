<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप अपनी कंपनी का विज्ञापन **HackTricks** में देखना चाहते हैं या **HackTricks को PDF में डाउनलोड** करना चाहते हैं तो [**सब्सक्रिप्शन प्लान**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारा संग्रह [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) और हमें **ट्विटर** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

<figure><img src="https://pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}


# समय-चिह्न

किसी हमलावर को **फ़ाइलों के समय-चिह्नों को बदलने** में दिलचस्पी हो सकती है।\
एमएफटी में समय-चिह्न ढूंढना संभव है जो गुणों `$STANDARD_INFORMATION` __ और __ `$FILE_NAME` में हैं।

दोनों गुणों में 4 समय-चिह्न होते हैं: **संशोधन**, **पहुंच**, **निर्माण**, और **एमएफटी रजिस्ट्री संशोधन** (MACE या MACB)।

**Windows एक्सप्लोरर** और अन्य उपकरण **`$STANDARD_INFORMATION`** से जानकारी दिखाते हैं।

## टाइमस्टॉम्प - एंटी-फोरेंसिक टूल

यह उपकरण **`$STANDARD_INFORMATION`** के भीतर समय-चिह्न जानकारी को **संशोधित करता है** **लेकिन** **`$FILE_NAME`** के भीतर की जानकारी को **नहीं**। इसलिए, यह संदेहास्पद गतिविधि की पहचान करना संभव है।

## Usnjrnl

**USN Journal** (अपडेट सीक्वेंस नंबर जर्नल) NTFS (Windows NT फ़ाइल सिस्टम) का एक विशेषता है जो वॉल्यूम परिवर्तनों का पता लगाता है। [**UsnJrnl2Csv**](https://github.com/jschicht/UsnJrnl2Csv) उपकरण इन परिवर्तनों का परीक्षण करने की अनुमति देता है।

![](<../../.gitbook/assets/image (449).png>)

पिछली छवि उपकरण द्वारा दिखाई गई **आउटपुट** है जहां देखा जा सकता है कि कुछ **परिवर्तन किए गए थे** फ़ाइल में।

## $LogFile

एक फ़ाइल सिस्टम में सभी मेटाडेटा परिवर्तनों को एक प्रक्रिया में लॉग किया जाता है जिसे [व्राइट-अहेड लॉगिंग](https://en.wikipedia.org/wiki/Write-ahead_logging) के रूप में जाना जाता है। लॉग किए गए मेटाडेटा को `**$LogFile**` नामक एक फ़ाइल में रखा जाता है, जो एक NTFS फ़ाइल सिस्टम के मूल निर्देशिका में स्थित है। [LogFileParser](https://github.com/jschicht/LogFileParser) जैसे उपकरण का उपयोग इस फ़ाइल को पार्स करने और परिवर्तनों की पहचान करने के लिए किया जा सकता है।

![](<../../.gitbook/assets/image (450).png>)

फिर से, उपकरण के आउटपुट में देखा जा सकता है कि **कुछ परिवर्तन किए गए थे**।

इसी उपकरण का उपयोग करके समय-चिह्नों को **किस समय संशोधित किया गया था** की पहचान करना संभव है:

![](<../../.gitbook/assets/image (451).png>)

* CTIME: फ़ाइल का निर्माण समय
* ATIME: फ़ाइल का संशोधन समय
* MTIME: फ़ाइल का MFT रजिस्ट्री संशोधन
* RTIME: फ़ाइल का पहुंच समय

## `$STANDARD_INFORMATION` और `$FILE_NAME` तुलना

संदेहास्पद संशोधित फ़ाइलों की पहचान करने के लिए एक और तरीका होगा कि दोनों गुणों पर समय की तुलना करें और **मिलान** के लिए देखें **असंगतियाँ**।

## नैनोसेकंड

**NTFS** समय-चिह्नों का एक **प्रेसिजन** है **100 नैनोसेकंड**। फिर, 2010-10-10 10:10:**00.000:0000 जैसे समय-चिह्न वाली फ़ाइलें खोजना बहुत संदेहास्पद है।

## SetMace - एंटी-फोरेंसिक टूल

यह उपकरण दोनों गुणों `$STARNDAR_INFORMATION` और `$FILE_NAME` को संशोधित कर सकता है। हालांकि, Windows Vista से, इस जानकारी को संशोधित करने के लिए लाइव ओएस की आवश्यकता है।

# डेटा छुपाना

NFTS एक क्लस्टर और न्यूनतम जानकारी आकार का उपयोग करता है। इसका मतलब है कि यदि एक फ़ाइल एक क्लस्टर और आधा उपयोग करती है, तो **शेष आधा कभी भी उपयोग नहीं होगा** जब तक फ़ाइल हटाई नहीं जाती। फिर, इस "छिपी" जगह में डेटा छुपाना संभव है।

इस तरह के "छिपी" जगह में डेटा छुपाने की अनुमति देने वाले उपकरण जैसे slacker हैं। हालांकि, `$logfile` और `$usnjrnl` का विश्लेषण दिखा सकता है कि कुछ डेटा जोड़ा गया था:

![](<../../.gitbook/assets/image (452).png>)

फिर, FTK Imager जैसे उपकरण का उपयोग करके छिपी जगह पुनः प्राप्त किया जा सकता है। ध्यान दें कि इस प्रकार के उपकरण सामग्री को अंधाकृत या यहाँ तक कि एन्क्रिप्टेड भी सहेज सकते हैं।

# UsbKill

यह एक उपकरण है जो **USB** पोर्ट में किसी भी परिवर्तन को डिटेक्ट करता है तो कंप्यूटर को **बंद कर देगा**।\
इसे खोजने का एक तरीका है कि चल रहे प्रक्रियाओं की जांच करें और **प्रत्येक पायथन स्क्रिप्ट की जांच** करें।

# लाइव लिनक्स वितरण

ये डिस्ट्रो **रैम मेमोरी के अंदर चलाए जाते हैं**। इन्हें पहचानने का एकमात्र तरीका है **यदि NTFS फाइल-सिस्टम को लेखन अनुमतियों के साथ माउंट किया गया है**। यदि यह केवल पढ़ने की अनुमतियों के साथ माउंट किया गया है तो उपद्रव का पता लगाना संभव नहीं होगा।

# सुरक्षित मिटाना

[https://github.com/Claudio-C/awesome-data-sanitization](https://github.com/Claudio-C/awesome-data-sanitization)

# Windows कॉन्फ़िगरेशन

फोरेंसिक्स जांच को कठिन बनाने के लिए कई विंडोज लॉगिंग विधियों को अक्षम करना संभव है।

## समय-चिह्नों को अक्षम करें - UserAssist

यह एक रजिस्ट्री कुंजी है जो प्रत्येक क्रियाशील द्वारा चलाई गई प्रत्येक कार्यक्रम के दिनांक और घंटे को बनाए रखती है।

UserAssist को अक्षम करने के लिए दो कदम हैं:

1. दो रजिस्ट्री कुंजियों को सेट करें, `HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Advanced\Start_TrackProgs` और `HKEY_CURRENT_USER\SOFTWARE
## हटाएं USB इतिहास

सभी **USB डिवाइस एंट्रीज** Windows रजिस्ट्री में **USBSTOR** रजिस्ट्री कुंजी के तहत संग्रहीत होती हैं जो उपकंजी शामिल करती हैं जो जब भी आप अपने PC या लैपटॉप में एक USB डिवाइस प्लग करते हैं। आप इस कुंजी को यहाँ पा सकते हैं `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Enum\USBSTOR`। **इसे हटाने** से आप USB इतिहास को हटा देंगे।\
आप इसे हटाने के लिए उपकरण [**USBDeview**](https://www.nirsoft.net/utils/usb_devices_view.html) का भी उपयोग कर सकते हैं (और इन्हें हटाने के लिए)।

एक और फ़ाइल जो USBs के बारे में जानकारी सहेजती है, वह है फ़ाइल `setupapi.dev.log` `C:\Windows\INF` के अंदर। इसे भी हटाया जाना चाहिए।

## शैडो कॉपियों को अक्षम करें

`vssadmin list shadowstorage` के साथ शैडो कॉपियों की **सूची** बनाएं\
उन्हें रन करके हटाएं `vssadmin delete shadow`

आप GUI के माध्यम से भी उन्हें हटा सकते हैं जिसके लिए [यहाँ प्रस्तावित चरणों](https://www.ubackup.com/windows-10/how-to-delete-shadow-copies-windows-10-5740.html) का पालन करें।

शैडो कॉपियों को अक्षम करने के लिए [यहाँ से चरण](https://support.waters.com/KB_Inf/Other/WKB15560_How_to_disable_Volume_Shadow_Copy_Service_VSS_in_Windows):

1. "सेवाएं" कार्यक्रम खोलें जिसे विंडोज स्टार्ट बटन पर क्लिक करने के बाद "सेवाएं" लिखकर टेक्स्ट खोज बॉक्स में टाइप करके।
2. सूची से "वॉल्यूम शैडो कॉपी" ढूंढें, इसे चुनें, और फिर राइट-क्लिक करके प्रॉपर्टीज़ तक पहुँचें।
3. "स्टार्टअप प्रकार" ड्रॉप-डाउन मेनू से "डिसेबल्ड" चुनें, और फिर बदलाव की पुष्टि करने के लिए "एप्लाई" और "ओके" पर क्लिक करें।

यह भी संभव है कि रजिस्ट्री में `HKLM\SYSTEM\CurrentControlSet\Control\BackupRestore\FilesNotToSnapshot` में कौन सी फ़ाइलें शैडो कॉपी में कॉपी की जाएंगी, उसकी विन्यास को संशोधित किया जा सकता है।

## हटाएं हटाए गए फ़ाइलें

* आप एक **Windows उपकरण** का उपयोग कर सकते हैं: `cipher /w:C` यह cipher को सी ड्राइव के उपलब्ध अप्रयुक्त डिस्क स्थान से किसी भी डेटा को हटाने के लिए संकेत देगा।
* आप [**Eraser**](https://eraser.heidi.ie) जैसे उपकरण भी उपयोग कर सकते हैं

## हटाएं Windows घटना लॉग

* Windows + R --> eventvwr.msc --> "Windows Logs" को विस्तारित करें --> प्रत्येक श्रेणी पर राइट क्लिक करें और "क्लियर लॉग" चुनें
* `for /F "tokens=*" %1 in ('wevtutil.exe el') DO wevtutil.exe cl "%1"`
* `Get-EventLog -LogName * | ForEach { Clear-EventLog $_.Log }`

## Windows घटना लॉग को अक्षम करें

* `reg add 'HKLM\SYSTEM\CurrentControlSet\Services\eventlog' /v Start /t REG_DWORD /d 4 /f`
* सेवाओं खंड के अंदर सेवा "Windows Event Log" को अक्षम करें
* `WEvtUtil.exec clear-log` या `WEvtUtil.exe cl`

## $UsnJrnl को अक्षम करें

* `fsutil usn deletejournal /d c:`

<figure><img src="https://pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}
