<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित करना** चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)

- प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके साझा करें।**

</details>


# GUI एप्लिकेशन में संभावित क्रियाएँ जांचें

**सामान्य संवाद** वे विकल्प हैं जो **फ़ाइल सहेजें**, **फ़ाइल खोलें**, फ़ॉन्ट का चयन करें, रंग का चयन करें... इनमें से अधिकांश विकल्पों में आपको **पूर्ण एक्सप्लोरर कार्यक्षमता प्रदान की जाएगी**। इसका मतलब है कि आप इन विकल्पों तक पहुंच सकते हैं अगर आप इन विकल्पों तक पहुंच सकते हैं:

* बंद करें/बंद करें जैसा
* खोलें/के साथ खोलें
* प्रिंट
* निर्यात/आयात
* खोजें
* स्कैन

आपको यह जांचना चाहिए कि क्या आप कर सकते हैं:

* फ़ाइलों को संशोधित या नई फ़ाइलें बना सकते हैं
* प्रतीकात्मक लिंक बना सकते हैं
* सीमित क्षेत्रों तक पहुंच प्राप्त कर सकते हैं
* अन्य ऐप्स को चला सकते हैं

## कमांड निष्पादन

शायद **_**Open with**_** विकल्प का उपयोग करके आप किसी प्रकार के शैल को खोल/निष्पादित कर सकते हैं।

### Windows

उदाहरण के लिए _cmd.exe, command.com, Powershell/Powershell ISE, mmc.exe, at.exe, taskschd.msc..._ यहां और बाइनरी ढूंढें जिन्हें कमांड निष्पादित करने के लिए उपयोग किया जा सकता है (और अप्रत्याशित क्रियाएँ कर सकता है): [https://lolbas-project.github.io/](https://lolbas-project.github.io)

### \*NIX __

_bash, sh, zsh..._ यहां और अधिक ढूंढें: [https://gtfobins.github.io/](https://gtfobins.github.io)

# Windows

## पथ प्रतिबंधों को छोड़ना

* **पर्यावरणीय चर**: बहुत सारे पर्यावरणीय चर हैं जो किसी पथ की ओर पॉइंट कर रहे हैं
* **अन्य प्रोटोकॉल**: _about:, data:, ftp:, file:, mailto:, news:, res:, telnet:, view-source:_
* **प्रतीकात्मक लिंक**
* **शॉर्टकट**: CTRL+N (नई सत्र खोलें), CTRL+R (कमांड निष्पादित करें), CTRL+SHIFT+ESC (कार्य प्रबंधक),  Windows+E (एक्सप्लोरर खोलें), CTRL-B, CTRL-I (पसंदीदा), CTRL-H (इतिहास), CTRL-L, CTRL-O (फ़ाइल/खोलें संवाद), CTRL-P (मुद्रण संवाद), CTRL-S (इस रूप में सहेजें)
* छिपा हुआ प्रशासनिक मेनू: CTRL-ALT-F8, CTRL-ESC-F9
* **शैल URIs**: _shell:Administrative Tools, shell:DocumentsLibrary, shell:Librariesshell:UserProfiles, shell:Personal, shell:SearchHomeFolder, shell:Systemshell:NetworkPlacesFolder, shell:SendTo, shell:UsersProfiles, shell:Common Administrative Tools, shell:MyComputerFolder, shell:InternetFolder_
* **UNC पथ**: साझा फ़ोल्डरों से कनेक्ट करने के लिए पथ। आपको स्थानीय मशीन के C$ से कनेक्ट करने की कोशिश करनी चाहिए ("\\\127.0.0.1\c$\Windows\System32")
* **और भी बहुत सारे UNC पथ:**

| UNC                       | UNC            | UNC                  |
| ------------------------- | -------------- | -------------------- |
| %ALLUSERSPROFILE%         | %APPDATA%      | %CommonProgramFiles% |
| %COMMONPROGRAMFILES(x86)% | %COMPUTERNAME% | %COMSPEC%            |
| %HOMEDRIVE%               | %HOM
## शॉर्टकट्स

* स्टिकी कीज़ - SHIFT को 5 बार दबाएं
* माउस कीज़ - SHIFT+ALT+NUMLOCK
* हाई कंट्रास्ट - SHIFT+ALT+PRINTSCN
* टॉगल कीज़ - NUMLOCK को 5 सेकंड तक दबाएं
* फ़िल्टर कीज़ - दायां SHIFT को 12 सेकंड तक दबाएं
* WINDOWS+F1 - Windows खोजें
* WINDOWS+D - डेस्कटॉप दिखाएं
* WINDOWS+E - Windows Explorer लॉन्च करें
* WINDOWS+R - चलाएँ
* WINDOWS+U - सुविधा पहुँच केंद्र
* WINDOWS+F - खोजें
* SHIFT+F10 - संदर्भ मेनू
* CTRL+SHIFT+ESC - कार्य प्रबंधक
* CTRL+ALT+DEL - नए Windows संस्करणों पर स्प्लैश स्क्रीन
* F1 - मदद F3 - खोजें
* F6 - पता बार
* F11 - इंटरनेट एक्सप्लोरर में पूर्ण स्क्रीन टॉगल करें
* CTRL+H - इंटरनेट एक्सप्लोरर इतिहास
* CTRL+T - इंटरनेट एक्सप्लोरर - नई टैब
* CTRL+N - इंटरनेट एक्सप्लोरर - नया पेज
* CTRL+O - फ़ाइल खोलें
* CTRL+S - सहेजें CTRL+N - नया RDP / Citrix

## स्वाइप

* विंडो को देखने के लिए बाएं ओर से दाएं ओर स्वाइप करें, KIOSK ऐप को कम करें और पूरे ओएस तक पहुँचें;
* एक्शन सेंटर खोलने के लिए दाएं ओर से बाएं ओर स्वाइप करें, KIOSK ऐप को कम करें और पूरे ओएस तक पहुँचें;
* शीर्ष किनारे से अपने ऐप में पूर्ण स्क्रीन मोड में शीर्षक बार दिखाने के लिए ऊपर से स्वाइप करें;
* बॉटम से ऊपर स्वाइप करें ताकि पूर्ण स्क्रीन ऐप में टास्कबार दिखाएं।

## इंटरनेट एक्सप्लोरर ट्रिक्स

### 'इमेज टूलबार'

यह टूलबार छवि के ऊपरी बाएं कोने पर प्रदर्शित होती है जब इसे क्लिक किया जाता है। आप इसे सहेज सकेंगे, प्रिंट कर सकेंगे, मेलटू कर सकेंगे, एक्सप्लोरर में "माय पिक्चर्स" खोल सकेंगे। Kiosk को इंटरनेट एक्सप्लोरर का उपयोग करना चाहिए।

### शैल प्रोटोकॉल

एक एक्सप्लोरर दृश्य प्राप्त करने के लिए इस URL को टाइप करें:

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

# ब्राउज़र ट्रिक्स

iKat संस्करणों का बैकअप करें:

[http://swin.es/k/](http://swin.es/k/)\
[http://www.ikat.kronicd.net/](http://www.ikat.kronicd.net)\

JavaScript का उपयोग करके एक सामान्य संवाद बनाएं और फ़ाइल एक्सप्लोरर तक पहुँचें: `document.write('<input/type=file>')`
स्रोत: https://medium.com/@Rend_/give-me-a-browser-ill-give-you-a-shell-de19811defa0

# iPad

## जेस्चर्स और बटन

### चार (या पांच) उंगलियों के साथ ऊपर स्वाइप करें / होम बटन पर दोहराएं टैप करें

मल्टीटास्क दृश्य देखने और ऐप बदलने के लिए

### चार या पांच उंगलियों के साथ एक तरफ स्वाइप करें

अगले / पिछले ऐप में बदलने के लिए

### पांच उंगलियों के साथ स्क्रीन को पिंच करें / होम बटन को छूने / स्क्रीन के नीचे से ऊपर एक उंगली के साथ तेजी से ऊपर स्वाइप करें

होम तक पहुँचने के लिए

### स्क्रीन के नीचे से एक उंगली से स्वाइप करें, स्क्रीन के नीचे 1-2 इंच (धीमी
| ⌘⇧⇥ (Command-Shift-Tab)                            | पिछले ऐप में स्विच करें                              |
| ⌘⇥ (Command-Tab)                                   | मूल ऐप में वापस स्विच करें                         |
| ←+→, फिर Option + ← या Option+→                   | डॉक के माध्यम से नेविगेट करें                                   |

### Safari शॉर्टकट्स

| शॉर्टकट                | कार्रवाई                                           |
| ----------------------- | ------------------------------------------------ |
| ⌘L (Command-L)          | स्थान खोलें                                    |
| ⌘T                      | एक नया टैब खोलें                                   |
| ⌘W                      | वर्तमान टैब बंद करें                            |
| ⌘R                      | वर्तमान टैब को रिफ्रेश करें                          |
| ⌘.                      | वर्तमान टैब को लोड करना बंद करें                     |
| ^⇥                      | अगले टैब में स्विच करें                           |
| ^⇧⇥ (Control-Shift-Tab) | पिछले टैब में जाएं                         |
| ⌘L                      | पाठ इनपुट/URL फ़ील्ड का चयन करें और इसे संशोधित करें     |
| ⌘⇧T (Command-Shift-T)   | अंतिम बंद किए गए टैब खोलें (कई बार उपयोग किया जा सकता है) |
| ⌘\[                     | अपने ब्राउज़िंग इतिहास में एक पेज पीछे जाएं      |
| ⌘]                      | अपने ब्राउज़िंग इतिहास में एक पेज आगे जाएं   |
| ⌘⇧R                     | रीडर मोड सक्रिय करें                             |

### मेल शॉर्टकट्स

| शॉर्टकट                   | कार्रवाई                       |
| -------------------------- | ---------------------------- |
| ⌘L                         | स्थान खोलें                |
| ⌘T                         | एक नया टैब खोलें           |
| ⌘W                         | वर्तमान टैब बंद करें        |
| ⌘R                         | वर्तमान टैब को रिफ्रेश करें  |
| ⌘.                         | वर्तमान टैब को लोड करना बंद करें |
| ⌘⌥F (Command-Option/Alt-F) | अपने मेलबॉक्स में खोजें       |

## संदर्भ

* [https://www.macworld.com/article/2975857/6-only-for-ipad-gestures-you-need-to-know.html](https://www.macworld.com/article/2975857/6-only-for-ipad-gestures-you-need-to-know.html)
* [https://www.tomsguide.com/us/ipad-shortcuts,news-18205.html](https://www.tomsguide.com/us/ipad-shortcuts,news-18205.html)
* [https://thesweetsetup.com/best-ipad-keyboard-shortcuts/](https://thesweetsetup.com/best-ipad-keyboard-shortcuts/)
* [http://www.iphonehacks.com/2018/03/ipad-keyboard-shortcuts.html](http://www.iphonehacks.com/2018/03/ipad-keyboard-shortcuts.html)


<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks** में विज्ञापित करना चाहते हैं? या क्या आप **PEASS** के नवीनतम संस्करण देखना चाहते हैं या **HackTricks** को PDF में डाउनलोड करना चाहते हैं? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)

- प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)** का** अनुसरण करें।**

- **अपने हैकिंग ट्रिक्स साझा करें, [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके।**

</details>
