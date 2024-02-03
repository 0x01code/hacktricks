<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स शेयर करें।

</details>


**मूल पोस्ट** [**https://itm4n.github.io/windows-registry-rpceptmapper-eop/**](https://itm4n.github.io/windows-registry-rpceptmapper-eop/) पर है

## सारांश
स्क्रिप्ट के आउटपुट से पता चलता है कि वर्तमान उपयोगकर्ता के पास दो रजिस्ट्री कुंजियों पर लिखने की अनुमति है:

- `HKLM\SYSTEM\CurrentControlSet\Services\Dnscache`
- `HKLM\SYSTEM\CurrentControlSet\Services\RpcEptMapper`

RpcEptMapper सेवा की अनुमतियों की आगे जांच करने के लिए, उपयोगकर्ता regedit GUI के उपयोग का उल्लेख करता है और Advanced Security Settings विंडो के Effective Permissions टैब की उपयोगिता को उजागर करता है। यह टैब उपयोगकर्ताओं को व्यक्तिगत ACEs की जांच किए बिना किसी विशेष उपयोगकर्ता या समूह को दी गई प्रभावी अनुमतियों की जांच करने की अनुमति देता है।

प्रदान किए गए स्क्रीनशॉट में कम-विशेषाधिकार वाले lab-user खाते की अनुमतियां दिखाई देती हैं। अधिकांश अनुमतियां मानक हैं, जैसे कि Query Value, लेकिन एक अनुमति बाहर खड़ी है: Create Subkey। इस अनुमति का सामान्य नाम AppendData/AddSubdirectory है, जो स्क्रिप्ट द्वारा रिपोर्ट की गई बात से मेल खाता है।

उपयोगकर्ता आगे समझाता है कि इसका मतलब है कि वे कुछ मानों को सीधे संशोधित नहीं कर सकते हैं लेकिन केवल नए सबकीज़ बना सकते हैं। वे एक उदाहरण दिखाते हैं जहां ImagePath मान को संशोधित करने की कोशिश करने पर एक्सेस डिनाइड त्रुटि होती है।

हालांकि, वे स्पष्ट करते हैं कि यह एक गलत सकारात्मक नहीं है और यहां एक दिलचस्प अवसर है। वे Windows रजिस्ट्री संरचना की जांच करते हैं और RpcEptMapper सेवा के लिए डिफ़ॉल्ट रूप से मौजूद नहीं होने वाली Performance सबकी का लाभ उठाने का एक संभावित तरीका खोजते हैं। यह सबकी DLL पंजीकरण और प्रदर्शन मॉनिटरिंग की अनुमति दे सकती है, जो विशेषाधिकार वृद्धि के लिए एक अवसर प्रदान करती है।

वे उल्लेख करते हैं कि उन्होंने Performance सबकी से संबंधित दस्तावेज़ीकरण पाया और इसका उपयोग प्रदर्शन मॉनिटरिंग के लिए कैसे करें। इससे उन्हें एक प्रूफ-ऑफ-कॉन्सेप्ट DLL बनाने और आवश्यक फ़ंक्शन्स: OpenPerfData, CollectPerfData, और ClosePerfData को लागू करने के लिए कोड दिखाने की प्रेरणा मिलती है। वे इन फ़ंक्शन्स को बाहरी उपयोग के लिए भी निर्यात करते हैं।

उपयोगकर्ता rundll32 का उपयोग करके DLL का परीक्षण करते हैं ताकि सुनिश्चित किया जा सके कि यह अपेक्षित रूप से कार्य करता है, सफलतापूर्वक जानकारी लॉग करता है।

आगे, वे समझाते हैं कि चुनौती RPC Endpoint Mapper सेवा को उनके Performance DLL को लोड करने के लिए चालाकी से बहकाने की है। वे उल्लेख करते हैं कि उन्होंने PowerShell में Performance Data से संबंधित WMI क्लासेस को क्वेरी करते समय अपनी लॉग फ़ाइल बनते हुए देखी। इससे उन्हें LOCAL SYSTEM के रूप में चलने वाली WMI सेवा के संदर्भ में मनमाने कोड को निष्पादित करने की अनुमति मिलती है, जो उन्हें अप्रत्याशित और उन्नत पहुंच प्रदान करती है।

निष्कर्ष में, उपयोगकर्ता इस कमजोरी की अनसमझी दृढ़ता और इसके संभावित प्रभाव को उजागर करते हैं, जो पोस्ट-एक्सप्लॉइटेशन, लेटरल मूवमेंट, और एंटीवायरस/EDR इवेज़न तक विस्तारित हो सकता है।

वे यह भी उल्लेख करते हैं कि जबकि उन्होंने अपनी स्क्रिप्ट के माध्यम से अनजाने में इस कमजोरी को सार्वजनिक किया, इसका प्रभाव सीमित है असमर्थित Windows संस्करणों (जैसे, Windows 7 / Server 2008 R2) तक जिनकी स्थानीय पहुंच है।


<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स शेयर करें।

</details>
