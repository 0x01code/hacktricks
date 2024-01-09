<details>

<summary><strong>AWS हैकिंग सीखें शून्य से नायक तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **अपनी हैकिंग ट्रिक्स साझा करें, HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके.

</details>


# पैक किए गए बाइनरीज की पहचान

* **स्ट्रिंग्स की कमी**: अक्सर देखा जाता है कि पैक किए गए बाइनरीज में लगभग कोई स्ट्रिंग नहीं होती है
* बहुत सारी **अनुपयोगी स्ट्रिंग्स**: जब कोई मैलवेयर किसी प्रकार के कमर्शियल पैकर का उपयोग कर रहा होता है, तो बिना क्रॉस-रेफरेंसेस के बहुत सारी स्ट्रिंग्स पाई जाती हैं। यहां तक कि अगर ये स्ट्रिंग्स मौजूद हैं, इसका यह मतलब नहीं है कि बाइनरी पैक नहीं है।
* आप कुछ टूल्स का उपयोग करके यह भी पता लगा सकते हैं कि किस पैकर का उपयोग करके बाइनरी को पैक किया गया था:
* [PEiD](http://www.softpedia.com/get/Programming/Packers-Crypters-Protectors/PEiD-updated.shtml)
* [Exeinfo PE](http://www.softpedia.com/get/Programming/Packers-Crypters-Protectors/ExEinfo-PE.shtml)
* [Language 2000](http://farrokhi.net/language/)

# मूल सिफारिशें

* **शुरू करें** पैक किए गए बाइनरी का विश्लेषण **IDA में नीचे से और ऊपर की ओर बढ़ते हुए**। अनपैकर्स एक बार अनपैक किए गए कोड के बाहर निकल जाते हैं, इसलिए यह संभावना कम है कि अनपैकर शुरुआत में अनपैक किए गए कोड को निष्पादन पास करता है।
* **JMP's** या **CALLs** की खोज करें **रजिस्टर्स** या **मेमोरी के क्षेत्रों** में। साथ ही, **फंक्शन्स की खोज करें जो तर्क और एक पते की दिशा को पुश करते हैं और फिर `retn` को कॉल करते हैं**, क्योंकि उस मामले में फंक्शन की वापसी उस पते को कॉल कर सकती है जिसे स्टैक पर पुश किया गया था।
* `VirtualAlloc` पर एक **ब्रेकपॉइंट** लगाएं क्योंकि यह मेमोरी में जगह आवंटित करता है जहां प्रोग्राम अनपैक किए गए कोड को लिख सकता है। "रन टू यूजर कोड" का उपयोग करें या F8 का उपयोग करके **EAX के अंदर के मूल्य तक पहुंचें** फंक्शन को निष्पादित करने के बाद और "**उस पते को डंप में फॉलो करें**"। आप कभी नहीं जानते कि यह वह क्षेत्र हो सकता है जहां अनपैक किया गया कोड सहेजा जा रहा है।
* **`VirtualAlloc`** जिसमें मान "**40**" एक तर्क के रूप में होता है, का मतलब है Read+Write+Execute (कुछ कोड जिसे निष्पादन की आवश्यकता होती है, यहां कॉपी किया जाने वाला है)।
* **कोड अनपैक करते समय** यह सामान्य है कि **कई कॉल्स** **अंकगणितीय ऑपरेशन्स** और फंक्शन्स जैसे **`memcopy`** या **`Virtual`**`Alloc` को पाया जाता है। यदि आप खुद को एक फंक्शन में पाते हैं जो केवल अंकगणितीय ऑपरेशन्स करता है और शायद कुछ `memcopy`, तो सिफारिश है कि फंक्शन के अंत को खोजने की कोशिश करें (शायद किसी रजिस्टर को JMP या कॉल) **या** कम से कम **अंतिम फंक्शन को कॉल करें** और तब तक चलाएं क्योंकि कोड दिलचस्प नहीं है।
* कोड अनपैक करते समय **ध्यान दें** जब भी आप **मेमोरी क्षेत्र बदलते हैं** क्योंकि मेमोरी क्षेत्र में बदलाव अनपैक किए गए कोड की शुरुआत का संकेत दे सकता है। आप आसानी से Process Hacker का उपयोग करके मेमोरी क्षेत्र को डंप कर सकते हैं (process --> properties --> memory)।
* कोड अनपैक करने की कोशिश करते समय यह जानने का एक अच्छा तरीका कि आप पहले से ही अनपैक किए गए कोड के साथ काम कर रहे हैं (ताकि आप बस इसे डंप कर सकें) यह है कि **बाइनरी की स्ट्रिंग्स की जांच करें**। यदि किसी बिंदु पर आप एक जंप करते हैं (शायद मेमोरी क्षेत्र बदलते हुए) और आप देखते हैं कि **बहुत अधिक स्ट्रिंग्स जोड़े गए थे**, तो आप जान सकते हैं **आप अनपैक किए गए कोड के साथ काम कर रहे हैं**।\
हालांकि, अगर पैकर पहले से ही बहुत सारी स्ट्रिंग्स रखता है तो आप देख सकते हैं कि कितनी स्ट्रिंग्स में शब्द "http" होता है और देखें कि क्या यह संख्या बढ़ती है।
* जब आप मेमोरी के क्षेत्र से एक एक्जीक्यूटेबल डंप करते हैं तो आप [PE-bear](https://github.com/hasherezade/pe-bear-releases/releases) का उपयोग करके कुछ हेडर्स को ठीक कर सकते हैं।


<details>

<summary><strong>AWS हैकिंग सीखें शून्य से नायक तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **अपनी हैकिंग ट्रिक्स साझा करें, HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके.

</details>
