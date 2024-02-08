<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप चाहते हैं कि आपकी **कंपनी HackTricks में विज्ञापित हो** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>


**मूल पोस्ट है** [**https://itm4n.github.io/windows-registry-rpceptmapper-eop/**](https://itm4n.github.io/windows-registry-rpceptmapper-eop/)

## सारांश

वर्तमान उपयोगकर्ता द्वारा दो रजिस्ट्री कुंजी मिली जिन्हें लिखने में सक्षम पाया गया था:

- **`HKLM\SYSTEM\CurrentControlSet\Services\Dnscache`**
- **`HKLM\SYSTEM\CurrentControlSet\Services\RpcEptMapper`**

**RpcEptMapper** सेवा की अनुमतियों की जांच करने की सुझाव दी गई थी, **regedit GUI** का उपयोग करके, विशेष रूप से **Advanced Security Settings** विंडो के **Effective Permissions** टैब। यह दृष्टिकोण विशिष्ट उपयोगकर्ताओं या समूहों को दी गई अनुमतियों का मूल्यांकन करने की संभावना बिना प्रत्येक पहुंच नियंत्रण प्रविष्टि (ACE) की जांच की आवश्यकता के बिना।

एक स्क्रीनशॉट में कम-अधिकारी उपयोगकर्ता को सौंपी गई अनुमतियाँ दिखाई गईं, जिनमें से **Create Subkey** अनुमति विशेष रूप से उल्लेखनीय थी। यह अनुमति, जिसे **AppendData/AddSubdirectory** भी कहा जाता है, स्क्रिप्ट के खोजों के साथ मेल खाती है।

कुछ मान्यताएँ सीधे कुछ मान्यताएँ सीधे संशोधित करने की असमर्थता, लेकिन नए सबकुंजियाँ बनाने की क्षमता को ध्यान में रखा गया था। एक उदाहरण हाइलाइट किया गया था जिसमें **ImagePath** मान को संशोधित करने का प्रयास किया गया था, जिसका परिणाम एक पहुंच निषेध संदेश था।

इन सीमाओं के बावजूद, एक विशेषाधिकार उच्चाधिकार के लिए संभावना की पहचान की गई थी जिसे **RpcEptMapper** सेवा के रजिस्ट्री संरचना के भीतर **Performance** सबकुंजी का उपयोग करके किया जा सकता था, जो डिफ़ॉल्ट रूप से मौजूद नहीं था। यह DLL पंजीकरण और प्रदर्शन मॉनिटरिंग को संभावित बना सकता था।

**Performance** सबकुंजी पर दस्तावेज़ीकरण और इसके प्रदर्शन मॉनिटरिंग के लिए उपयोग की जानकारी पर ध्यान दिया गया, जिससे **OpenPerfData**, **CollectPerfData**, और **ClosePerfData** फ़ंक्शनों के कार्यान्वयन का प्रमाण बनाने के लिए एक प्रमाण-अवसर DLL विकसित किया गया। यह DLL, **rundll32** के माध्यम से टेस्ट किया गया, इसकी सफलता की पुष्टि करता है।

लक्ष्य था **RPC Endpoint Mapper सेवा** को बनाए रखना कि निर्मित प्रदर्शन DLL को लोड करें। अवलोकनों ने दिखाया कि PowerShell के माध्यम से प्रदर्शन डेटा से संबंधित WMI क्लास क्वेरी को क्रियान्वित करने से एक लॉग फ़ाइल बनाई जाती है, जिससे **LOCAL SYSTEM** संदर्भ के तहत विशेषाधिकार प्राप्त होते हैं, जिससे उच्चाधिकार प्राप्त होते हैं।

इस दुरुपयोग की स्थायित्व और संभावित परिणामों पर जोर दिया गया था, इसका महत्व पोस्ट-उत्पीडन रणनीतियों, पार की गति, और एंटीवायरस/EDR सिस्टमों से बचाव के लिए उपयोगी होने की उचितता को हाइलाइट किया गया।

हालांकि, यह दुरुपयोग पहले से ही स्क्रिप्ट के माध्यम से अनजाने में खुलासा किया गया था, इसका दुरुपयोग पुराने Windows संस्करणों (जैसे **Windows 7 / Server 2008 R2**) पर सीमित है और स्थानीय पहुंच की आवश्यकता है।

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप चाहते हैं कि आपकी **कंपनी HackTricks में विज्ञापित हो** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>
