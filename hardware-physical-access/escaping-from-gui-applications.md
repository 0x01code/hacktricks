# KIOSKs से बाहर निकलना

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप चाहते हैं कि आपकी **कंपनी HackTricks में विज्ञापित हो** या **HackTricks को PDF में डाउनलोड करें** तो [**सदस्यता योजनाएं देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, **The PEASS Family** की खोज करें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** पर **फॉलो** करें 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**।**
* **अपने हैकिंग ट्रिक्स साझा करें, HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके।

</details>

#### [WhiteIntel](https://whiteintel.io)

<figure><img src="../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) एक **डार्क-वेब** से प्रेरित खोज इंजन है जो **मुफ्त** सुविधाएं प्रदान करता है ताकि जांच सकें कि क्या कोई कंपनी या उसके ग्राहकों को **स्टीलर मैलवेयर्स** द्वारा **क्षति पहुंचाई गई** है।

WhiteIntel का मुख्य उद्देश्य खाता हाथियाने और रैंसमवेयर हमलों का मुकाबला करना है जो जानकारी चुराने वाले मैलवेयर से होते हैं।

आप उनकी वेबसाइट चेक कर सकते हैं और **मुफ्त** में उनका इंजन आजमा सकते हैं:

{% embed url="https://whiteintel.io" %}

---

## भौतिक उपकरण की जांच

|   घटक   | क्रिया                                                               |
| ------------- | -------------------------------------------------------------------- |
| पावर बटन  | उपकरण को बंद और फिर से चालू करने से स्टार्ट स्क्रीन उजागर हो सकता है      |
| पावर केबल   | जांचें कि उपकरण को थोड़ी देर के लिए पावर काटने पर रीबूट होता है   |
| USB पोर्ट्स     | फिजिकल कीबोर्ड कनेक्ट करें जिसमें अधिक शॉर्टकट्स हों                        |
| ईथरनेट      | नेटवर्क स्कैन या स्निफिंग से और अधिक शोषण संभावना हो सकती है             |


## GUI एप्लिकेशन के अंदर संभावित क्रियाएँ जांचें

**सामान्य संवाद** वह विकल्प हैं जैसे **एक फ़ाइल को सहेजना**, **एक फ़ाइल को खोलना**, एक फ़ॉन्ट, एक रंग का चयन... ज्यादातर में ये **पूर्ण एक्सप्लोरर कार्यक्षमता प्रदान करेंगे**। इसका मतलब है कि आप एक्सप्लोरर कार्यक्षमताओं तक पहुंच सकेंगे अगर आप इन विकल्पों तक पहुंच सकते हैं:

* बंद करें/के रूप में बंद करें
* खोलें/के साथ खोलें
* प्रिंट
* निर्यात/आयात
* खोज
* स्कैन

आपको यह जांचनी चाहिए कि क्या आप:

* फ़ाइलों को संशोधित या नई फ़ाइलें बना सकते हैं
* प्रतीकात्मक लिंक बना सकते हैं
* प्रतिबंधित क्षेत्रों तक पहुंच सकते हैं
* अन्य ऐप्स को चला सकते हैं

### कमांड निष्पादन

शायद **`Open with`** विकल्प का उपयोग करके आप किसी प्रकार का शैल खोल/निष्पादित कर सकते हैं।

#### Windows

उदाहरण के लिए _cmd.exe, command.com, Powershell/Powershell ISE, mmc.exe, at.exe, taskschd.msc..._ यहाँ और भी बाइनरी हैं जो कमांड निष्पादित करने के लिए उपयोग किए जा सकते हैं (और अप्रत्याशित क्रियाएँ कर सकते हैं) यहाँ: [https://lolbas-project.github.io/](https://lolbas-project.github.io)

#### \*NIX \_\_

_bash, sh, zsh..._ अधिक यहाँ: [https://gtfobins.github.io/](https://gtfobins.github.io)

## Windows

### पथ प्रतिबंधों को छोड़ना

* **पर्यावरण चर**: कई पर्यावरण चर हैं जो किसी पथ को इंगित कर रहे हैं
* **अन्य प्रोटोकॉल**: _about:, data:, ftp:, file:, mailto:, news:, res:, telnet:, view-source:_
* **प्रतीकात्मक लिंक्स**
* **शॉर्टकट्स**: CTRL+N (नई सत्र खोलें), CTRL+R (कमांड निष्पादित करें), CTRL+SHIFT+ESC (कार्य प्रबंधक), Windows+E (एक्सप्लोरर खोलें), CTRL-B, CTRL-I (पसंद), CTRL-H (इतिहास), CTRL-L, CTRL-O (फ़ाइल/खोलने का संवाद), CTRL-P (प्रिंट संवाद), CTRL-S (सहेजें जैसा)
* छिपी हुई प्रशासनिक मेनू: CTRL-ALT-F8, CTRL-ESC-F9
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

* स्टिकी कीज - SHIFT 5 बार दबाएं
* माउस कीज - SHIFT+ALT+NUMLOCK
* हाई कंट्रास्ट - SHIFT+ALT+PRINTSCN
* टॉगल कीज - NUMLOCK को 5 सेकंड के लिए दबाएं
* फ़िल्टर कीज - दाएं SHIFT को 12 सेकंड के लिए दबाएं
* WINDOWS+F1 - Windows खोज
* WINDOWS+D - डेस्कटॉप दिखाएं
* WINDOWS+E - Windows एक्सप्लोरर लॉन्च करें
* WINDOWS+R - रन
* WINDOWS+U - आसान पहुंच केंद्र
* WINDOWS+F - खोज
* SHIFT+F10 - संदर्भ मेनू
* CTRL+SHIFT+ESC - टास्क प्रबंधक
* CTRL+ALT+DEL - नए Windows संस्करणों पर स्प्लैश स्क्रीन
* F1 - मदद F3 - खोज
* F6 - पता बार
* F11 - इंटरनेट एक्सप्लोरर के भीतर पूर्ण स्क्रीन टॉगल
* CTRL+H - इंटरनेट एक्सप्लोरर इतिहास
* CTRL+T - इंटरनेट एक्सप्लोरर - नया टैब
* CTRL+N - इंटरनेट एक्सप्लोरर - नया पेज
* CTRL+O - फ़ाइल खोलें
* CTRL+S - सहेजें CTRL+N - नया RDP / Citrix

### स्वाइप

* छोड़कर दाएं ओर से दाएं ओर देखने के लिए छोटे से बड़े स्क्रीन पर सभी खुले विंडोज, KIOSK ऐप को कम करना और पूरे ओएस तक सीधे पहुंचना;
* दाएं ओर से बाएं ओर स्वाइप करने के लिए कार्रवाई केंद्र खोलने के लिए, KIOSK ऐप को कम करना और पूरे ओएस तक सीधे पहुंचना;
* ऊपर से स्वाइप करने से एप्लिकेशन खोलने के लिए शीर्ष पट्टी दिखाई देगी;
* नीचे से ऊपर की ओर स्वाइप करने से पूर्ण स्क्रीन ऐप में टास्कबार दिखाई देगी।

### इंटरनेट एक्सप्लोरर ट्रिक्स

#### 'इमेज टूलबार'

यह एक टूलबार है जो छवि के ऊपर बाएं-दाएं में दिखाई देती है जब इसे क्लिक किया जाता है। आप "सेव करें, प्रिंट, मेलटो, अपनी तस्वीरें" खोल सकेंगे। Kiosk को इंटरनेट एक्सप्लोरर का उपयोग करना चाहिए।

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
* `shell:::{{208D2C60-3AEA-1069-A2D7-08002B30309D}}` --> मेरे नेटवर्क स्थान
* `shell:::{871C5380-42A0-1069-A2EA-08002B30309D}` --> इंटरनेट एक्सप्लोरर

### फ़ाइल एक्सटेंशन दिखाएं

अधिक जानकारी के लिए इस पेज की जाँच करें: [https://www.howtohaven.com/system/show-file-extensions-in-windows-explorer.shtml](https://www.howtohaven.com/system/show-file-extensions-in-windows-explorer.shtml)

## ब्राउज़र ट्रिक्स

आईकैट संस्करण की बैकअप:

[http://swin.es/k/](http://swin.es/k/)\
[http://www.ikat.kronicd.net/](http://www.ikat.kronicd.net)\\

जावास्क्रिप्ट का उपयोग करके सामान्य संवाद बनाएं और फ़ाइल एक्सप्लोरर तक पहुंचें: `document.write('<input/type=file>')`\
स्रोत: https://medium.com/@Rend\_/give-me-a-browser-ill-give-you-a-shell-de19811defa0

## आईपैड

### इशारे और बटन

* चार (या पांच) उंगलियों के साथ ऊपर स्वाइप करें / होम बटन को दोहराएं: मल्टीटास्क दृश्य देखने और ऐप बदलने के लिए
* चार या पांच उंगलियों के साथ एक तरफ या दूसरी तरफ स्वाइप करें: अगले/पिछले ऐप में बदलने के लिए
* पांच उंगलियों के साथ स्क्रीन को पिंच करें / होम बटन को छूनें / तेजी से ऊपर से एक उंगली से ऊपर की ओर स्वाइप करें: होम तक पहुंचने के लिए
* एक उंगली से स्क्रीन के नीचे से ऊपर स्वाइप करें: अपने नोटिफिकेशन देखने के लिए
* एक उंगली से स्क्रीन के ऊपर-दाएं कोने से नीचे स्वाइप करें: आईपैड प्रो का नियंत्रण केंद्र देखने के लिए
* स्क्रीन के बाएं किनारे से एक उंगली से दाएं की ओर 1-2 इंच (धीमे से): डॉक दिखाई देगा
* एक उंगली से डिस्प्ले के ऊपर से नीचे स्वाइप करें: अपने नोटिफिकेशन देखने के लिए
* एक उंगली से डिस्प्ले के ऊपर-दाएं कोने से नीचे स्वाइप करें: आईपैड प्रो का नियंत्रण केंद्र देखने के लिए
* बाएं या दाएं से स्क्रीन के केंद्र से तेजी से एक उंगली से दाएं या बाएं: अगले/पिछले ऐप में बदलने के लिए
* ऊपर-दाएं कोने पर ऊपर-दाएं कोने पर दबाएं और रखें: आईपैड को बंद करने के लिए
* ऊपर-दाएं कोने पर ऊपर-दाएं कोने पर दबाएं और होम बटन कुछ सेकंड के लिए: हार्ड पावर ऑफ करने के लिए
* ऊपर-दाएं कोने पर ऊपर-दाएं कोने पर दबाएं और होम बटन तेजी से: एक स्क्रीनशॉट लेने के लिए जो डिस्प्ले के निचले बाएं में पॉप अप होगा। दोनों बटनों को एक साथ बहुत कम समय तक दबाएं जैसे कि आप कुछ सेकंड तक उन्हें पकड़ते हैं तो हार्ड पावर ऑफ हो जाएगा।

### शॉर्टकट्स

आपके पास एक आईपैड कुंजीपटल या एक USB कुंजीपटल अडैप्टर होना चाहिए। यहाँ केवल उन शॉर्टकट्स को दिखाया जाएगा जो ऐप्लिकेशन से बाहर निकलने में मदद कर सकते हैं।

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

| शॉर्टकट | कार्रवाई                                                                         |
| -------- | ------------------------------------------------------------------------------ |
| F1       | स्क्रीन को धीमा करें                                                                    |
| F2       | स्क्रीन को उज्ज्वल करें                                                                |
| F7       | पिछला गाना                                                                  |
| F8       | प्ले/पॉज़ करें                                                                     |
| F9       | गाना छोड़ें                                                                      |
| F10      | म्यूट करें                                                                           |
| F11      | ध्वनि का स्तर कम करें                                                                |
| F12      | ध्वनि का स्तर बढ़ाएं                                                                |
| ⌘ Space  | उपलब्ध भाषाओं की सूची प्रदर्शित करें; एक चुनने के लिए, फिर स्पेस बार दबाएं। |

#### आईपैड नेविगेशन

| शॉर्टकट                                           | कार्रवाई                                                  |
| -------------------------------------------------- | ------------------------------------------------------- |
| ⌘H                                                 | होम पर जाएं                                              |
| ⌘⇧H (कमांड-शिफ्ट-एच)                              | होम पर जाएं                                              |
| ⌘ (स्पेस)                                          | स्पॉटलाइट खोलें                                          |
| ⌘⇥ (कमांड-टैब)                                   | पिछले दस
#### सफारी शॉर्टकट्स

| शॉर्टकट                | कार्रवाई                                           |
| ----------------------- | ------------------------------------------------ |
| ⌘L (कमांड-एल)          | स्थान खोलें                                    |
| ⌘T                      | नया टैब खोलें                                   |
| ⌘W                      | वर्तमान टैब बंद करें                            |
| ⌘R                      | वर्तमान टैब को रिफ्रेश करें                          |
| ⌘.                      | वर्तमान टैब को लोडिंग रोकें                     |
| ^⇥                      | अगले टैब पर स्विच करें                           |
| ^⇧⇥ (कंट्रोल-शिफ्ट-टैब) | पिछले टैब पर जाएं                         |
| ⌘L                      | पाठ इनपुट/URL फ़ील्ड का चयन करें और संशोधित करें     |
| ⌘⇧T (कमांड-शिफ्ट-टी)   | आखिरी बंद किए गए टैब खोलें (कई बार प्रयोग किया जा सकता है) |
| ⌘\[                     | आपके ब्राउज़िंग इतिहास में एक पृष्ठ पीछे जाएं      |
| ⌘]                      | आपके ब्राउज़िंग इतिहास में एक पृष्ठ आगे जाएं   |
| ⌘⇧R                     | रीडर मोड सक्रिय करें                             |

#### मेल शॉर्टकट्स

| शॉर्टकट                   | कार्रवाई                       |
| -------------------------- | ---------------------------- |
| ⌘L                         | स्थान खोलें                |
| ⌘T                         | नया टैब खोलें               |
| ⌘W                         | वर्तमान टैब बंद करें        |
| ⌘R                         | वर्तमान टैब को रिफ्रेश करें  |
| ⌘.                         | वर्तमान टैब को लोडिंग रोकें |
| ⌘⌥F (कमांड-ऑप्शन/ऑल्ट-एफ) | अपने मेलबॉक्स में खोजें       |

## संदर्भ

* [https://www.macworld.com/article/2975857/6-only-for-ipad-gestures-you-need-to-know.html](https://www.macworld.com/article/2975857/6-only-for-ipad-gestures-you-need-to-know.html)
* [https://www.tomsguide.com/us/ipad-shortcuts,news-18205.html](https://www.tomsguide.com/us/ipad-shortcuts,news-18205.html)
* [https://thesweetsetup.com/best-ipad-keyboard-shortcuts/](https://thesweetsetup.com/best-ipad-keyboard-shortcuts/)
* [http://www.iphonehacks.com/2018/03/ipad-keyboard-shortcuts.html](http://www.iphonehacks.com/2018/03/ipad-keyboard-shortcuts.html)

#### [WhiteIntel](https://whiteintel.io)

<figure><img src="../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) एक **डार्क-वेब** पर आधारित खोज इंजन है जो **मुफ्त** सुविधाएं प्रदान करता है ताकि यह जांच सकें कि कोई कंपनी या उसके ग्राहकों को **स्टीलर मैलवेयर्स** द्वारा **क्षति पहुंचाई गई** है या नहीं।

व्हाइटइंटेल का मुख्य उद्देश्य खाता हथियाने और जानकारी चुराने वाले मैलवेयर से होने वाले खतरे से निपटना है।

आप उनकी वेबसाइट चेक कर सकते हैं और **मुफ्त** में उनके इंजन का प्रयोग कर सकते हैं:

{% embed url="https://whiteintel.io" %}

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो बनाने के लिए</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी कंपनी का विज्ञापन HackTricks में देखना चाहते हैं या **HackTricks को PDF में डाउनलोड** करना चाहते हैं तो [**सब्सक्रिप्शन प्लान**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में PR जमा करके।

</details>
