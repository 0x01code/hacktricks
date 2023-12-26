# macOS xpc\_connection\_get\_audit\_token हमला

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबरसिक्योरिटी कंपनी** में काम करते हैं? क्या आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे**? या क्या आप **PEASS के नवीनतम संस्करण तक पहुँच चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) में या **Twitter** पर मुझे **फॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपनी हैकिंग ट्रिक्स साझा करें, PRs जमा करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **में.**

</details>

**यह तकनीक** [**https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/**](https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/) **से कॉपी की गई थी**

## Mach Messages मूल जानकारी

यदि आपको Mach Messages के बारे में पता नहीं है तो इस पृष्ठ को देखें:

{% content-ref url="../../../../mac-os-architecture/macos-ipc-inter-process-communication/" %}
[macos-ipc-inter-process-communication](../../../../mac-os-architecture/macos-ipc-inter-process-communication/)
{% endcontent-ref %}

फिलहाल याद रखें कि:
Mach messages _mach port_ पर भेजे जाते हैं, जो कि mach kernel में निर्मित **एकल प्राप्तकर्ता, बहुसंख्यक प्रेषक संचार** चैनल है। **बहुत सारी प्रक्रियाएँ संदेश भेज सकती हैं** mach port पर, लेकिन किसी भी समय **केवल एक प्रक्रिया इससे पढ़ सकती है**। फाइल डिस्क्रिप्टर्स और सॉकेट्स की तरह, mach ports को कर्नेल द्वारा आवंटित और प्रबंधित किया जाता है और प्रक्रियाएँ केवल एक पूर्णांक देखती हैं, जिसका उपयोग वे कर्नेल को यह बताने के लिए कर सकते हैं कि उनके किस mach port का उपयोग करना चाहते हैं।

## XPC कनेक्शन

यदि आपको पता नहीं है कि XPC कनेक्शन कैसे स्थापित किया जाता है तो देखें:

{% content-ref url="../" %}
[..](../)
{% endcontent-ref %}

## Vuln सारांश

आपके लिए जानना दिलचस्प है कि **XPC का अमूर्तीकरण एक-से-एक कनेक्शन है**, लेकिन यह एक ऐसी तकनीक पर आधारित है जिसमें **बहुसंख्यक प्रेषक हो सकते हैं, इसलिए:**

* Mach ports एकल प्राप्तकर्ता, _**बहुसंख्यक प्रेषक**_ हैं।
* XPC कनेक्शन का audit token _**सबसे हाल ही में प्राप्त संदेश से कॉपी किया गया**_ audit token होता है।
* XPC कनेक्शन के **audit token** को प्राप्त करना कई **सुरक्षा जांचों** के लिए महत्वपूर्ण है।

हालांकि पिछली स्थिति आशाजनक लगती है, कुछ परिदृश्य हैं जहाँ यह समस्याएँ पैदा नहीं करेगा:

* Audit tokens अक्सर एक प्राधिकरण जांच के लिए उपयोग किए जाते हैं ताकि यह तय किया जा सके कि क्या एक कनेक्शन स्वीकार करना है। जैसा कि यह सेवा पोर्ट को एक संदेश का उपयोग करके होता है, अभी तक **कोई कनेक्शन स्थापित नहीं हुआ है**। इस पोर्ट पर और अधिक संदेश केवल अतिरिक्त कनेक्शन अनुरोधों के रूप में संभाले जाएंगे। इसलिए किसी भी **जांच से पहले कनेक्शन स्वीकार करना सुरक्षित नहीं है** (इसका यह भी मतलब है कि `-listener:shouldAcceptNewConnection:` के भीतर audit token सुरक्षित है)। इसलिए हम **उन XPC कनेक्शनों की तलाश कर रहे हैं जो विशिष्ट क्रियाओं की जांच करते हैं**।
* XPC इवेंट हैंडलर्स को सिंक्रोनसली संभाला जाता है। इसका मतलब है कि एक संदेश के लिए इवेंट हैंडलर को पूरा करना होगा इससे पहले कि इसे अगले के लिए बुलाया जा सके, यहां तक कि समानांतर डिस्पैच कतारों पर भी। इसलिए एक **XPC इवेंट हैंडलर के भीतर audit token को अन्य सामान्य (गैर-उत्तर!) संदेशों द्वारा अधिलेखित नहीं किया जा सकता**।

इससे हमें दो अलग-अलग तरीकों का विचार आया जिससे यह संभव हो सकता है:

1. विकल्प1:
* **Exploit** सेवा **A** और सेवा **B** से **जुड़ता है**
* सेवा **B** सेवा A में एक **विशेषाधिकार प्राप्त कार्यक्षमता** को बुला सकती है जो उपयोगकर्ता नहीं कर सकता
* सेवा **A** **`xpc_connection_get_audit_token`** को बुलाती है जबकि _**नहीं**_ एक कनेक्शन के लिए **इवेंट हैंडलर** के भीतर **`dispatch_async`** में।
* इसलिए एक **अलग** संदेश **Audit Token को अधिलेखित कर सकता है** क्योंकि यह इवेंट हैंडलर के बाहर असिंक्रोनसली डिस्पैच किया जा रहा है।
* Exploit **सेवा B को सेवा A के लिए SEND अधिकार पास करता है**।
* इसलिए svc **B** वास्तव में **संदेश भेज रही है** सेवा **A** को।
* **Exploit** कोशिश करता है **विशेषाधिकार प्राप्त कार्यक्षमता को बुलाने के लिए**। RC में svc **A** इस **क्रिया** के प्राधिकरण की **जांच करती है** जबकि **svc B ने Audit token को अधिलेखित किया** (जिससे exploit को विशेषाधिकार प्राप्त कार्यक्षमता को बुलाने की पहुँच मिलती है)।
2. विकल्प 2:
* सेवा **B** सेवा A में एक **विशेषाधिकार प्राप्त कार्यक्षमता** को बुला सकती है जो उपयोगकर्ता नहीं कर सकता
* Exploit **सेवा A** से जुड़ता है जो exploit को एक **संदेश भेजती है जिसकी एक उत्तर की उम्मीद है** एक विशिष्ट **प्रतिक्रिया** **पोर्ट** में।
* Exploit **सेवा** B को एक संदेश भेजता है **उस प्रतिक्रिया पोर्ट को पास करते हुए**।
* जब सेवा **B उत्तर देती है**, यह **सेवा A को संदेश भेजती है**, **जबकि** **exploit** एक अलग **संदेश सेवा A को भेजता है** जिससे वह एक **विशेषाधिकार प्राप्त कार्यक्षमता तक पहुँचने की कोशिश करता है** और उम्मीद करता है कि सेवा B से प्रतिक्रिया Audit token को सही समय पर अधिलेखित कर देगी (Race Condition)।

## विकल्प 1: एक इवेंट हैंडलर के बाहर xpc\_connection\_get\_audit\_token को बुलाना <a href="#variant-1-calling-xpc_connection_get_audit_token-outside-of-an-event-handler" id="variant-1-calling-xpc_connection_get_audit_token-outside-of-an-event-handler"></a>
