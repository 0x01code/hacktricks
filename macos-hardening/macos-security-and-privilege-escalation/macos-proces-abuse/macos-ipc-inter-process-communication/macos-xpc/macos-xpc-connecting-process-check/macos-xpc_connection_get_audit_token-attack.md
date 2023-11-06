# macOS xpc\_connection\_get\_audit\_token हमला

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल** हों या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का** **अनुसरण** करें।**
* **अपने हैकिंग ट्रिक्स को** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में पीआर जमा करके अपना योगदान दें।**

</details>

**यह तकनीक** [**https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/**](https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/) **से कॉपी की गई है**

## Mach संदेश मूल जानकारी

यदि आपको पता नहीं है कि Mach संदेश क्या होते हैं, तो इस पृष्ठ की जांच करना शुरू करें:

{% content-ref url="../../../../mac-os-architecture/macos-ipc-inter-process-communication/" %}
[macos-ipc-inter-process-communication](../../../../mac-os-architecture/macos-ipc-inter-process-communication/)
{% endcontent-ref %}

अब तक याद रखें कि:\
Mach संदेश _mach पोर्ट_ के माध्यम से भेजे जाते हैं, जो मैक कर्नल में बने **एकल प्राप्तकर्ता, एकाधिक भेजने वाला संचार** चैनल है। **एकाधिक प्रक्रियाएं संदेश भेज सकती हैं** मगर किसी भी समय **केवल एक प्रक्रिया संदेश पढ़ सकती है**। फ़ाइल डिस्क्रिप्टर और सॉकेट की तरह, मैक पोर्ट्स को कर्नल द्वारा आवंटित और प्रबंधित किया जाता है और प्रक्रियाएं केवल एक पूर्णांक देखती हैं, जिसे वे कर्नल को इंगित करने के लिए उपयोग कर सकती हैं कि वे अपने मैक पोर्ट्स में से कौन सा उपयोग करना चाहती हैं।

## XPC कनेक्शन

यदि आपको पता नहीं है कि XPC कनेक्शन कैसे स्थापित किया जाता है, तो जांचें:

{% content-ref url="../" %}
[..](../)
{% endcontent-ref %}

## Vuln सारांश

आपके लिए दिलचस्प होने वाली बात यह है कि **XPC का अभिकल्पन एक-से-एक कनेक्शन है**, लेकिन यह एक तकनीक पर आधारित है जिसमें **एकाधिक भेजने वाले** हो सकते हैं, इसलिए:

* Mach पोर्ट एकल प्राप्तकर्ता, _**एकाधिक भेजने वाला**_ होता है।
* XPC कनेक्शन का ऑडिट टोकन _**सबसे हाल ही में प्राप्त किए गए संदेश से कॉपी किया जाता है**_।
* XPC कनेक्शन के **ऑडिट टोकन** को बहुत सारे **सुरक्षा जांचों** के लिए प्राथमिकता होती है।

हालांकि, पिछली स्थिति वादास्तविक लगती है कि कुछ स्थितियों में यह समस्या उत्पन्न नहीं करेगी:

* ऑडिट टोकन अक्सर एक अधिकृतता जांच के लिए उपयोग होते हैं ताकि कनेक्शन को स्वीकार करने का निर्णय लिया जा सके। क्योंकि यह संदेश का उपयोग सेवा पोर्ट को करने के लिए होता है, इसलिए **अभी तक कनेक्शन स्थापित नहीं हुआ है**। इस पोर्ट पर अधिक संदेश केवल अतिरिक्त कनेक्शन अनुरोधों के रूप में हैंडल किए जाएंगे। इसलिए किसी भी **कनेक्शन को स्वीकार करने से पहले की जांचें विकल्पशील नहीं होती हैं** (इसका यह भी मतलब है
## वेरिएंट 1: इवेंट हैंडलर के बाहर xpc_connection_get_audit_token को कॉल करना <a href="#variant-1-calling-xpc_connection_get_audit_token-outside-of-an-event-handler" id="variant-1-calling-xpc_connection_get_audit_token-outside-of-an-event-handler"></a>

परिदृश्य:

* दो मैक **सेवाएं** _**A**_ और _**B**_ जिनसे हम दोनों कनेक्ट कर सकते हैं (सैंडबॉक्स प्रोफ़ाइल और कनेक्शन स्वीकार करने से पहले अधिकारीकरण जांचों पर आधारित)।
* _**A**_ को किसी विशेष **कार्रवाई के लिए अधिकारीकरण जांच** होनी चाहिए जिसे _**B**_ (लेकिन हमारे ऐप नहीं) पास कर सकता है।
* उदाहरण के लिए, यदि B के पास कुछ **अधिकार** हैं या वह **रूट** के रूप में चल रहा है, तो यह उसे अधिकारीकृत कार्रवाई करने की अनुमति देगा।
* इस अधिकारीकरण जांच के लिए, _**A**_ **असिंक्रोनस्ली ऑडिट टोकन प्राप्त करता है**, उदाहरण के लिए `dispatch_async` से `xpc_connection_get_audit_token` को कॉल करके।

{% hint style="danger" %}
इस मामले में एक हमलावर एक **रेस कंडीशन** ट्रिगर कर सकता है जो एक **एक्सप्लॉइट** बनाता है जो **A से कार्रवाई करने के लिए कहता है** कई बार जबकि **B A को संदेश भेजता है**। जब RC **सफल होता है**, तो **B** का **ऑडिट टोकन** मेमोरी में कॉपी हो जाएगा **जबकि** हमारे **एक्सप्लॉइट** का अनुरोध A द्वारा **हैंडल** किया जा रहा होता है, जिससे इसे **अधिकारीकृत कार्रवाई तक पहुंच मिलती है जिसे केवल B कर सकता है**।
{% endhint %}

यह _**A**_ के साथ हुआ **`smd` के रूप में** और _**B**_ के साथ **`diagnosticd` के रूप में**। smb के [`SMJobBless`](https://developer.apple.com/documentation/servicemanagement/1431078-smjobbless?language=objc) फ़ंक्शन का उपयोग करके एक नया विशेषाधिकारित सहायक टूल स्थापित करने के लिए किया जा सकता है (जैसे कि **रूट** के रूप में)। यदि **रूट के रूप में चल रहे प्रक्रिया संपर्क** करती है **smd**, तो कोई अन्य जांच नहीं की जाएगी।

इसलिए, सेवा **B** **`diagnosticd` है** क्योंकि यह **रूट** के रूप में चलता है और एक प्रक्रिया का **मॉनिटरिंग** करने के लिए उपयोग किया जा सकता है, इसलिए एक बार मॉनिटरिंग शुरू हो जाएगी, यह **हर सेकंड एकाधिक संदेश भेजेगा।**

हम हमला करने के लिए निम्नलिखित करते हैं:

1. हम आम XPC प्रोटोकॉल का पालन करके **`smd`** के साथ अपना **कनेक्शन स्थापित** करते हैं।
2. फिर, हम **`diagnosticd`** के साथ एक **कनेक्शन स्थापित** करते हैं, लेकिन दो नए मैक पोर्ट उत्पन्न करने और उन्हें भेजने की बजाय, हम क्लाइंट पोर्ट भेजने के लिए **`smd` के संदर्भ में हमारे पास हैंडल भेजने की एक कॉपी बनाते हैं**।
3. इसका मतलब है कि हम **`diagnosticd`** को XPC संदेश भेज सकते हैं, लेकिन कोई भी **संदेश `diagnosticd` `smd` पर जाते हैं**।&#x20;
* `smd` के लिए, हमारे और `diagnosticd` के संदेश दोनों ही एक ही कनेक्शन पर प्राप्त होते हैं।

<figure><img src="../../../../../../.gitbook/assets/image (1) (1).png" alt="" width="563"><figcaption></figcaption></figure>

4. हम **`diagnosticd`** से कहते हैं कि हमारी (या किसी भी सक्रिय) प्रक्रिया की **मॉनिटरिंग शुरू करें** और हम **`smd` को 1004 संदेशों को स्पैम करें** (एक विशेषाधिकारित उपकरण स्थापित करने के लिए)।
5. इससे एक रेस कंडीशन बनाता है जो `handle_bless` में एक बहुत विशेष खिड़की में हिट होनी चाहिए। हमें `xpc_connection_get_pid` को हमारी खुद की प्रक्रिया का पीआईडी लौटाने की आवश्यकता है, क्योंकि विशेषाधिकारित सहायक उपकरण हमारे ऐप बंडल में है। हालांकि, `connection_is_authorized` फ़ंक्शन के भीतर `xpc_connection_get_audit_token` को कॉल करने की कॉल को `diganosticd` के ऑडिट टोकन का उपयोग करना चाहिए।

## वेरिएंट 2: उत्तर फॉरवर्डिंग

जैसा कि पहले कहा गया है
## सुधार <a href="#the-fix" id="the-fix"></a>

अंत में, हमने `smd` में सामान्य समस्या और विशेष समस्या की रिपोर्ट की। Apple ने इसे केवल `smd` में ही ठीक किया है, जहां `xpc_connection_get_audit_token` को `xpc_dictionary_get_audit_token` के साथ बदल दिया गया है।

फ़ंक्शन `xpc_dictionary_get_audit_token` मशीन संदेश से ऑडिट टोकन की कॉपी करता है, जिसका मतलब यह है कि यह सुरक्षित है। हालांकि, `xpc_dictionary_get_audit_token` की तरह, यह सार्वजनिक API का हिस्सा नहीं है। उच्च स्तरीय `NSXPCConnection` API के लिए, मौजूदा संदेश के ऑडिट टोकन को प्राप्त करने के लिए कोई स्पष्ट विधि मौजूद नहीं है, क्योंकि इसमें सभी संदेशों को विधि कॉल में छिपाने का काम किया जाता है।

हमें यह स्पष्ट नहीं है कि Apple ने क्यों एक और सामान्य सुधार नहीं लागू किया, उदाहरण के लिए कनेक्शन के सहेजे गए ऑडिट टोकन के मेल नहीं खाने वाले संदेशों को छोड़ देना। ऐसे स्थितियाँ हो सकती हैं जहां प्रक्रिया के ऑडिट टोकन में वास्तविक बदलाव होता है लेकिन कनेक्शन खुली रहनी चाहिए (उदाहरण के लिए, `setuid` कोल करने पर UID फ़ील्ड बदल जाता है), लेकिन एक अलग PID या PID संस्करण के तरह के बदलाव अपेक्षित नहीं हैं।

किसी भी स्थिति में, यह समस्या अभी भी iOS 17 और macOS 14 के साथ मौजूद है, इसलिए अगर आप इसे खोजना चाहते हैं, तो शुभकामनाएं!

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप चाहते हैं कि आपकी **कंपनी HackTricks में विज्ञापित** की जाए? या क्या आप **PEASS की नवीनतम संस्करण देखना चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह देखें
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का** **अनुसरण करें।**
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को।**

</details>
