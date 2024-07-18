# KIOSKs से बाहर निकलना

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम एक्सपर्ट (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम एक्सपर्ट (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में PR जमा करके।

</details>
{% endhint %}

#### [WhiteIntel](https://whiteintel.io)

<figure><img src="../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) एक **डार्क-वेब** पर आधारित खोज इंजन है जो **मुफ्त** सुविधाएं प्रदान करता है ताकि जांच सकें कि क्या कोई कंपनी या उसके ग्राहकों को **स्टीलर मैलवेयर** द्वारा **क्षति** पहुंचाई गई है।

WhiteIntel का मुख्य उद्देश्य खाता हाथियाने और रैंसमवेयर हमलों का मुकाबला करना है जो जानकारी चोरी करने वाले मैलवेयर से होते हैं।

आप उनकी वेबसाइट चेक कर सकते हैं और **मुफ्त** में उनका इंजन आजमा सकते हैं:

{% embed url="https://whiteintel.io" %}

---

## भौतिक उपकरण की जाँच

|   घटक   | क्रिया                                                               |
| ------------- | -------------------------------------------------------------------- |
| पावर बटन  | उपकरण को बंद और फिर से चालू करने से प्रारंभ स्क्रीन प्रकट हो सकता है      |
| पावर केबल   | जांचें कि उपकरण को थोड़ी देर के लिए बिजली काटने पर पुनरारंभ होता है   |
| USB पोर्ट्स     | फिजिकल कीबोर्ड कनेक्ट करें जिसमें अधिक शॉर्टकट्स हों                        |
| ईथरनेट      | नेटवर्क स्कैन या स्निफिंग से और अधिक शोषण संभव हो सकता है             |


## GUI एप्लिकेशन के अंदर संभावित क्रियाएँ जांचें

**सामान्य संवाद** वह विकल्प हैं जो **एक फ़ाइल सहेजना**, **एक फ़ाइल खोलना**, एक फ़ॉन्ट, एक रंग का चयन करने के... ज्यादातर में ये **पूर्ण एक्सप्लोरर कार्यक्षमता प्रदान करेंगे**। इसका मतलब है कि आप एक्सप्लोरर कार्यक्षमताओं तक पहुंच सकेंगे अगर आप इन विकल्पों तक पहुंच सकते हैं:

* बंद करें/बंद करें जैसा
* खोलें/के साथ खोलें
* प्रिंट
* निर्यात/आयात
* खोज
* स्कैन

आपको यह जांचनी चाहिए कि क्या आप:

* फ़ाइलों को संशोधित या नई फ़ाइलें बना सकते हैं
* प्रतीकात्मक लिंक बना सकते हैं
* प्रतिबंधित क्षेत्रों तक पहुंच प्राप्त कर सकते हैं
* अन्य ऐप्स को क्रियान्वित कर सकते हैं

### कमांड निष्पादन

शायद **`खोलें के साथ`** विकल्प का उपयोग करके आप किसी प्रकार की शैल खोल/निष्पादित कर सकते हैं।

#### Windows

उदाहरण के लिए _cmd.exe, command.com, Powershell/Powershell ISE, mmc.exe, at.exe, taskschd.msc..._ यहाँ और भी बाइनरी हैं जो कमांड निष्पादित करने के लिए उपयोग किए जा सकते हैं (और अप्रत्याशित क्रियाएँ कर सकते हैं): [https://lolbas-project.github.io/](https://lolbas-project.github.io)

#### \*NIX \_\_

_bash, sh, zsh..._ अधिक यहाँ: [https://gtfobins.github.io/](https://gtfobins.github.io)

## Windows

### पथ प्रतिबंधों को छोड़ना

* **पर्यावरण चर**: कई पर्यावरण चर हैं जो किसी पथ को दिखा रहे हैं
* **अन्य प्रोटोकॉल**: _about:, data:, ftp:, file:, mailto:, news:, res:, telnet:, view-source:_
* **प्रतीकात्मक लिंक्स**
* **शॉर्टकट्स**: CTRL+N (नई सत्र खोलें), CTRL+R (कमांड निष्पादित करें), CTRL+SHIFT+ESC (कार्य प्रबंधक), Windows+E (एक्सप्लोरर खोलें), CTRL-B, CTRL-I (पसंद), CTRL-H (इतिहास), CTRL-L, CTRL-O (फ़ाइल/खोलें संवाद), CTRL-P (प्रिंट संवाद), CTRL-S (इस रूप में सहेजें)
* छिपा हुआ प्रशासनिक मेनू: CTRL-ALT-F8, CTRL-ESC-F9
* **शैल URI**: _shell:Administrative Tools, shell:DocumentsLibrary, shell:Librariesshell:UserProfiles, shell:Personal, shell:SearchHomeFolder, shell:Systemshell:NetworkPlacesFolder, shell:SendTo, shell:UsersProfiles, shell:Common Administrative Tools, shell:MyComputerFolder, shell:InternetFolder_
* **UNC पथ**: साझा फ़ोल्डरों से कनेक्ट करने के लिए पथ। आपको स्थानीय मशीन के C$ से कनेक्ट करने की कोशिश करनी चाहिए ("\\\127.0.0.1\c$\Windows\System32")
* **अधिक UNC पथ:**

| UNC                       | UNC            | UNC                  |
| ------------------------- | -------------- | -------------------- |
| %ALLUSERSPROFILE%         | %APPDATA%      | %CommonProgramFiles% |
| %COMMONPROGRAMFILES(x86)% | %COMPUTERNAME% | %COMSPEC%            |
| %HOMEDRIVE%               | %HOMEPATH%     | %LOCALAPPDATA%       |
| %LOGONSERVER%             | %PATH%         | %PATHEXT%            |
| %ProgramData%             | %ProgramFiles% | %ProgramFiles(x86)%  |
| %PROMPT%                  | %PSModulePath% | %Public%             |
| %SYSTEMDRIVE%             | %SYSTEMROOT%   | %TEMP%               |
| %TMP%                     | %USERDOMAIN%   | %USERNAME%           |
| %USERPROFILE%             | %WINDIR%       |                      |

### अपने बाइनरी डाउनलोड करें

कंसोल: [https://sourceforge.net/projects/console/](https://sourceforge.net/projects/console/)\
एक्सप्लोरर: [https://sourceforge.net/projects/explorerplus/files/Explorer%2B%2B/](https://sourceforge.net/projects/explorerplus/files/Explorer%2B%2B/)\
रजिस्ट्री संपादक: [https://sourceforge.net/projects/uberregedit/](https://sourceforge.net/projects/uberregedit/)

### ब्राउज़र से फ़ाइल सिस्टम तक पहुंच

| PATH                | PATH              | PATH               | PATH                |
| ------------------- | ----------------- | ------------------ | ------------------- |
| File:/C:/windows    | File:/C:/windows/ | File:/C:/windows\\ | File:/C:\windows    |
| File:/C:\windows\\  | File:/C:\windows/ | File://C:/windows  | File://C:/windows/  |
| File://C:/windows\\ | File://C:\windows | File://C:\windows/ | File://C:\windows\\ |
| C:/windows          | C:/windows/       | C:/windows\\       | C:\windows          |
| C:\windows\\        | C:\windows/       | %WINDIR%           | %TMP%               |
| %TEMP%              | %SYSTEMDRIVE%     | %SYSTEMROOT%       | %APPDATA%           |
| %HOMEDRIVE%         | %HOMESHARE        |                    | <p><br></p>         |
### शॉर्टकट्स

* स्टिकी की – SHIFT 5 बार दबाएं
* माउस की – SHIFT+ALT+NUMLOCK
* हाई कंट्रास्ट – SHIFT+ALT+PRINTSCN
* टॉगल की – NUMLOCK को 5 सेकंड के लिए दबाएं
* फ़िल्टर की – दाएं SHIFT को 12 सेकंड के लिए दबाएं
* WINDOWS+F1 – Windows खोज
* WINDOWS+D – डेस्कटॉप दिखाएं
* WINDOWS+E – Windows एक्सप्लोरर लॉन्च करें
* WINDOWS+R – रन
* WINDOWS+U – इज ऑफ़ एक्सेस सेंटर
* WINDOWS+F – खोज
* SHIFT+F10 – संदर्भ मेनू
* CTRL+SHIFT+ESC – टास्क मैनेजर
* CTRL+ALT+DEL – नए Windows संस्करणों पर स्प्लैश स्क्रीन
* F1 – मदद F3 – खोज
* F6 – पता बार
* F11 – इंटरनेट एक्सप्लोरर के भीतर पूर्ण स्क्रीन टॉगल
* CTRL+H – इंटरनेट एक्सप्लोरर इतिहास
* CTRL+T – इंटरनेट एक्सप्लोरर – नया टैब
* CTRL+N – इंटरनेट एक्सप्लोरर – नया पेज
* CTRL+O – फ़ाइल खोलें
* CTRL+S – सहेजें CTRL+N – नया RDP / Citrix

### स्वाइप

* छोड़ें की तरफ से दाएं ओर स्वाइप करें ताकि सभी खुली विंडो दिखाई दें, KIOSK ऐप को कम करें और पूरे ओएस तक सीधे पहुंचें;
* दाएं ओर से बाएं की तरफ स्वाइप करें ताकि क्रिया केंद्र खोलें, KIOSK ऐप को कम करें और पूरे ओएस तक सीधे पहुंचें;
* ऊपर की किनारे से स्वाइप करें ताकि पूर्ण स्क्रीन मोड में खोले गए ऐप के लिए शीर्ष बार दिखाई दे;
* नीचे से ऊपर की ओर स्वाइप करें ताकि पूर्ण स्क्रीन ऐप में टास्कबार दिखाई दे।

### इंटरनेट एक्सप्लोरर ट्रिक्स

#### 'इमेज टूलबार'

यह एक टूलबार है जो छवि के ऊपर बाएं-दाएं में दिखाई देती है जब इसे क्लिक किया जाता है। आप "सेव", "प्रिंट", "मेलटो", "माय पिक्चर्स" इंएक्सप्लोरर में खोलने की क्षमता होगी। किओस्क को इंटरनेट एक्सप्लोरर का उपयोग करना चाहिए।

#### शैल प्रोटोकॉल

एक्सप्लोरर दृश्य प्राप्त करने के लिए इस URL को टाइप करें:

* `shell:Administrative Tools`
* `shell:DocumentsLibrary`
* `shell:Libraries`
* `shell:UserProfiles`
* `shell:Personal`
* `shell:SearchHomeFolder`
* `shell:NetworkPlacesFolder`
* `shell:SendTo`
* `shell:UserProfiles`
* `shell:Common Administrative Tools`
* `shell:MyComputerFolder`
* `shell:InternetFolder`
* `Shell:Profile`
* `Shell:ProgramFiles`
* `Shell:System`
* `Shell:ControlPanelFolder`
* `Shell:Windows`
* `shell:::{21EC2020-3AEA-1069-A2DD-08002B30309D}` --> कंट्रोल पैनल
* `shell:::{20D04FE0-3AEA-1069-A2D8-08002B30309D}` --> मेरा कंप्यूटर
* `shell:::{{208D2C60-3AEA-1069-A2D7-08002B30309D}}` --> मेरे नेटवर्क प्लेसेस
* `shell:::{871C5380-42A0-1069-A2EA-08002B30309D}` --> इंटरनेट एक्सप्लोरर

### फ़ाइल एक्सटेंशन दिखाएं

अधिक जानकारी के लिए इस पेज की जाँच करें: [https://www.howtohaven.com/system/show-file-extensions-in-windows-explorer.shtml](https://www.howtohaven.com/system/show-file-extensions-in-windows-explorer.shtml)

## ब्राउज़र ट्रिक्स

आईकैट संस्करण की बैकअप:

[http://swin.es/k/](http://swin.es/k/)\
[http://www.ikat.kronicd.net/](http://www.ikat.kronicd.net)\\

जावास्क्रिप्ट का उपयोग करके एक सामान्य संवाद बनाएं और फ़ाइल एक्सप्लोरर तक पहुंचें: `document.write('<input/type=file>')`\
स्रोत: https://medium.com/@Rend\_/give-me-a-browser-ill-give-you-a-shell-de19811defa0

## आईपैड

### इशारे और बटन

* चार (या पांच) उंगलियों के साथ ऊपर स्वाइप करें / होम बटन को दोहराएं: मल्टीटास्क दृश्य देखने और ऐप बदलने के लिए
* चार या पांच उंगलियों के साथ किसी भी तरह स्वाइप करें: अगले/पिछले ऐप में बदलने के लिए
* पांच उंगलियों के साथ स्क्रीन को पिंच करें / होम बटन को छूनें / एक उंगली से तेजी से स्क्रीन के नीचे से ऊपर स्वाइप करें: होम तक पहुंचने के लिए
* एक उंगली से स्क्रीन के नीचे से सिर्फ 1-2 इंच (धीमे से) स्वाइप करें: डॉक दिखाई देगा
* एक उंगली से डिस्प्ले के ऊपर से नीचे स्वाइप करें: अपने सूचनाएं देखने के लिए
* एक उंगली से स्क्रीन के ऊपर-दाएं कोने से नीचे स्वाइप करें: आईपैड प्रो का नियंत्रण केंद्र देखने के लिए
* एक उंगली से स्क्रीन के बाएं किनारे से 1-2 इंच: आज का दृश्य देखने के लिए
* एक उंगली से स्क्रीन के केंद्र से दाएं या बाएं तेजी से स्वाइप करें: अगले/पिछले ऐप में बदलने के लिए
* ऊपर-दाएं कोने में ओन/**ऑफ़**/स्लीप बटन को दबाएं + स्लाइड को **पावर ऑफ़** स्लाइडर को पूरी तरह से दाएं ले जाएं: पावर ऑफ़ करने के लिए
* ऊपर-दाएं कोने में ओन/**ऑफ़**/स्लीप बटन को दबाएं + होम बटन को कुछ सेकंड के लिए: हार्ड पावर ऑफ़ करने के लिए
* ऊपर-दाएं कोने में ओन/**ऑफ़**/स्लीप बटन को दबाएं + होम बटन को तेजी से: एक स्क्रीनशॉट लेने के लिए जो डिस्प्ले के नीचे बाएं में पॉप अप होगा। दोनों बटनों को एक साथ बहुत हल्के से दबाएं जैसे कि आप कुछ सेकंड होल्ड करते हैं तो हार्ड पावर ऑफ़ हो जाएगा।

### शॉर्टकट्स

आपके पास एक आईपैड कुंजीपटल या एक USB कुंजीपटल अडैप्टर होना चाहिए। यहाँ केवल उन शॉर्टकट्स को दिखाया जाएगा जो ऐप से बाहर निकलने में मदद कर सकते हैं।

| कुंजी | नाम         |
| --- | ------------ |
| ⌘   | कमांड      |
| ⌥   | विकल्प (ऑल्ट) |
| ⇧   | शिफ्ट        |
| ↩   | वापस       |
| ⇥   | टैब          |
| ^   | नियंत्रण      |
| ←   | बाएं तीर   |
| →   | दाएं तीर  |
| ↑   | ऊपर तीर     |
| ↓   | नीचे तीर   |

#### सिस्टम शॉर्टकट्स

ये शॉर्टकट्स विजुअल सेटिंग्स और ध्वनि सेटिंग्स के लिए हैं, आईपैड के उपयोग पर निर्भर करते हैं।

| शॉर्टकट | क्रिया                                                                         |
| -------- | ------------------------------------------------------------------------------ |
| F1       | स्क्रीन कम करें                                                                    |
| F2       | स्क्रीन चमकाएं                                                                |
| F7       | पिछला गाना                                                                  |
| F8       | प्ले/पॉज़ करें                                                                     |
| F9       | गाना छोड़ें                                                                      |
| F10      | म्यूट करें                                                                           |
| F11      | ध्वनि की गति कम करें                                                                |
| F12      | ध्वनि की गति बढ़ाएं                                                                |
| ⌘ Space  | उपलब्ध भाषाओं की सूची दिखाएं; चुनने के लिए, फिर स्पेस बार दबाएं। |

#### आईपैड नेविगेशन

| शॉर्टकट                                           | क्रिया                                                  |
| -------------------------------------------------- | ------------------------------------------------------- |
| ⌘H                                                 | होम पर जाएं                                              |
| ⌘⇧H (कमांड-शिफ्ट-एच)                              | होम पर जाएं                                              |
| ⌘ (स्पेस)                                          | स्पॉटलाइट खोलें                                          |
| ⌘⇥ (कमांड-टैब)                                   | पिछले दस उपयोग किए गए ऐप्स की सूची                  |
| ⌘\~                                                | अंतिम ऐप पर जाएं                                       |
|
#### सफारी शॉर्टकट्स

| शॉर्टकट                | क्रिया                                           |
| ----------------------- | ------------------------------------------------ |
| ⌘L (कमांड-एल)          | स्थान खोलें                                    |
| ⌘T                      | नया टैब खोलें                                   |
| ⌘W                      | वर्तमान टैब बंद करें                            |
| ⌘R                      | वर्तमान टैब को रिफ्रेश करें                          |
| ⌘.                      | वर्तमान टैब को लोडिंग रोकें                     |
| ^⇥                      | अगले टैब पर स्विच करें                           |
| ^⇧⇥ (कंट्रोल-शिफ्ट-टैब) | पिछले टैब पर जाएं                         |
| ⌘L                      | पाठ इनपुट/URL फील्ड का चयन करें और संशोधित करें     |
| ⌘⇧T (कमांड-शिफ्ट-टी)   | आखिरी बंद किए गए टैब खोलें (कई बार प्रयोग किया जा सकता है) |
| ⌘\[                     | आपके ब्राउज़िंग हिस्ट्री में एक पृष्ठ पीछे जाएं      |
| ⌘]                      | आपके ब्राउज़िंग हिस्ट्री में एक पृष्ठ आगे जाएं   |
| ⌘⇧R                     | रीडर मोड सक्रिय करें                             |

#### मेल शॉर्टकट्स

| शॉर्टकट                   | क्रिया                       |
| -------------------------- | ---------------------------- |
| ⌘L                         | स्थान खोलें                |
| ⌘T                         | नया टैब खोलें               |
| ⌘W                         | वर्तमान टैब बंद करें        |
| ⌘R                         | वर्तमान टैब को रिफ्रेश करें  |
| ⌘.                         | वर्तमान टैब को लोडिंग रोकें |

## संदर्भ

* [https://www.macworld.com/article/2975857/6-only-for-ipad-gestures-you-need-to-know.html](https://www.macworld.com/article/2975857/6-only-for-ipad-gestures-you-need-to-know.html)
* [https://www.tomsguide.com/us/ipad-shortcuts,news-18205.html](https://www.tomsguide.com/us/ipad-shortcuts,news-18205.html)
* [https://thesweetsetup.com/best-ipad-keyboard-shortcuts/](https://thesweetsetup.com/best-ipad-keyboard-shortcuts/)
* [http://www.iphonehacks.com/2018/03/ipad-keyboard-shortcuts.html](http://www.iphonehacks.com/2018/03/ipad-keyboard-shortcuts.html)

#### [WhiteIntel](https://whiteintel.io)

<figure><img src="../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) एक **डार्क-वेब** प्रेरित खोज इंजन है जो उपयोगकर्ता कंपनी या उसके ग्राहकों को **स्टीलर मैलवेयर** द्वारा **कंप्रोमाइज़** किया गया है या नहीं जांचने की **मुफ्त** सुविधाएं प्रदान करता है।

व्हाइटइंटेल का मुख्य उद्देश्य खाता हथियाने और रैंसमवेयर हमलों का मुकाबला करना है जो जानकारी चोरी करने वाले मैलवेयर से होते हैं।

आप उनकी वेबसाइट चेक कर सकते हैं और उनके इंजन का **मुफ्त** प्रयोग कर सकते हैं:

{% embed url="https://whiteintel.io" %}

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) और हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें द्वारा पीआर जमा करके** [**हैकट्रिक्स**](https://github.com/carlospolop/hacktricks) और [**हैकट्रिक्स क्लाउड**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में।

</details>
{% endhint %}
