<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप चाहते हैं कि आपकी कंपनी **HackTricks में विज्ञापित हो** या **HackTricks को PDF में डाउनलोड** करना चाहते हैं तो [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* **शामिल हों** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)** पर **फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>


# GUI एप्लिकेशन के अंदर संभावित क्रियाएँ जांचें

**सामान्य डायलॉग** वे विकल्प हैं जैसे कि **एक फ़ाइल को सहेजना**, **एक फ़ाइल को खोलना**, फ़ॉन्ट का चयन करना, रंग का चयन करना... ज्यादातर इनमें **पूर्ण एक्सप्लोरर कार्यक्षमता प्रदान की जाएगी**। इसका मतलब है कि आप एक्सप्लोरर कार्यक्षमताओं तक पहुंच सकेंगे अगर आप इन विकल्पों तक पहुंच सकते हैं:

* बंद करें/बंद करें जैसा
* खोलें/खोलें के साथ
* प्रिंट
* निर्यात/आयात
* खोज
* स्कैन

आपको यह जांचना चाहिए कि क्या आप:

* फ़ाइलों को संशोधित या नई फ़ाइलें बना सकते हैं
* प्रतीकात्मक लिंक बना सकते हैं
* प्रतिबंधित क्षेत्रों तक पहुंच सकते हैं
* अन्य ऐप्स को चला सकते हैं

## कमांड निष्पादन

शायद **`खोलें के साथ`** विकल्प का उपयोग करके आप किसी प्रकार की शैल को खोल/निष्पादित कर सकते हैं।

### Windows

उदाहरण के लिए _cmd.exe, command.com, Powershell/Powershell ISE, mmc.exe, at.exe, taskschd.msc..._ यहाँ अधिक बाइनरी जो कमांड निष्पादित करने के लिए उपयोग किए जा सकते हैं उन्हें खोजें: [https://lolbas-project.github.io/](https://lolbas-project.github.io)

### \*NIX __

_bash, sh, zsh..._ अधिक यहाँ: [https://gtfobins.github.io/](https://gtfobins.github.io)

# Windows

## पथ प्रतिबंधों को छोड़ना

* **पर्यावरण चर**: कई पर्यावरण चर हैं जो किसी पथ को संकेतित कर रहे हैं
* **अन्य प्रोटोकॉल**: _about:, data:, ftp:, file:, mailto:, news:, res:, telnet:, view-source:_
* **प्रतीकात्मक लिंक**
* **शॉर्टकट्स**: CTRL+N (नई सत्र खोलें), CTRL+R (कमांड निष्पादित करें), CTRL+SHIFT+ESC (कार्य प्रबंधक),  Windows+E (एक्सप्लोरर खोलें), CTRL-B, CTRL-I (पसंदीदा), CTRL-H (इतिहास), CTRL-L, CTRL-O (फ़ाइल/खोलें संवाद), CTRL-P (प्रिंट संवाद), CTRL-S (इस रूप में सहेजें)
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

## अपने बाइनरी डाउनलोड करें

कंसोल: [https://sourceforge.net/projects/console/](https://sourceforge.net/projects/console/)\
एक्सप्लोरर: [https://sourceforge.net/projects/explorerplus/files/Explorer%2B%2B/](https://sourceforge.net/projects/explorerplus/files/Explorer%2B%2B/)\
रजिस्ट्री संपादक: [https://sourceforge.net/projects/uberregedit/](https://sourceforge.net/projects/uberregedit/)

## ब्राउज़र से फ़ाइल सिस्टम तक पहुंच

| पथ                | पथ              | पथ               | पथ                |
| ------------------- | ----------------- | ------------------ | ------------------- |
| File:/C:/windows    | File:/C:/windows/ | File:/C:/windows\\ | File:/C:\windows    |
| File:/C:\windows\\  | File:/C:\windows/ | File://C:/windows  | File://C:/windows/  |
| File://C:/windows\\ | File://C:\windows | File://C:\windows/ | File://C:\windows\\ |
| C:/windows          | C:/windows/       | C:/windows\\       | C:\windows          |
| C:\windows\\        | C:\windows/       | %WINDIR%           | %TMP%               |
| %TEMP%              | %SYSTEMDRIVE%     | %SYSTEMROOT%       | %APPDATA%           |
| %HOMEDRIVE%         | %HOMESHARE        |                    | <p><br></p>         |

## शॉर्टकट्स

* Sticky Keys – SHIFT 5 बार दबाएं
* Mouse Keys – SHIFT+ALT+NUMLOCK
* High Contrast – SHIFT+ALT+PRINTSCN
* Toggle Keys – NUMLOCK को 5 सेकंड के लिए धारित करें
* Filter Keys – दाएं SHIFT को 12 सेकंड के लिए धारित करें
* WINDOWS+F1 – Windows खोज
* WINDOWS+D – डेस्कटॉप दिखाएं
* WINDOWS+E – Windows एक्सप्लोरर लॉन्च करें
* WINDOWS+R – रन
* WINDOWS+U – सुविधा केंद्र
* WINDOWS+F – खोज
* SHIFT+F10 – संदर्भ मेनू
* CTRL+SHIFT+ESC – कार्य प्रबंधक
* CTRL+ALT+DEL – नए Windows संस्करणों पर स्प्लैश स्क्रीन
* F1 – Help F3 – खोज
* F6 – पता बार
* F11 – इंटरनेट एक्सप्लोरर में पूर्ण स्क्रीन टॉगल करें
* CTRL+H – इंटरनेट एक्सप्लोरर इतिहास
* CTRL+T – इंटरनेट एक्सप्लोरर – नया टैब
* CTRL+N – इंटरनेट एक्सप्लोरर – नई पेज
* CTRL+O – फ़ाइल खोलें
* CTRL+S – सहेजें CTRL+N – नया RDP / Citrix

## स्वाइप

* बाएं ओर से दाएं ओर तक स्वाइप करें ताकि सभी खुले विंडो देखें, KIOSK ऐप को कम करें और पूरे ओएस तक सीधे पहुंचें;
* दाएं ओर से बाएं स्वाइप करें ताकि क्रिया केंद्र खोलें, KIOSK ऐप को कम करें और पूरे ओएस तक सीधे पहुंचें;
* ऊपर से एक किनारे से स्वाइप करें ताकि एक ऐप जो पूरी स्क्रीन मोड में खुली हो उसके शीर्ष पट्टी दिखाई दे;
* नीचे से ऊपर की ओर स्वाइप करें ताकि पूरी स्क्रीन ऐप में टास्कबार दिखाएं।

## इंटरनेट एक्सप्लोरर ट्रिक्स

### 'इमेज टूलबार'

यह एक टूलबार है जो छवि के ऊपर बाएं-दाएं में दिखाई देती है। आपको सहेजने, प्रिंट करने, मेल भेजने, एक्सप्लोरर में "मेरी तस्वीरें" खोलने की सुविधा मिलेगी। Kiosk को इंटरनेट एक्सप्लोरर का उपयोग करना चाहिए।

### शैल प्रोटोकॉल

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
* `shell:::{{208D
