# केरबेरोस प्रमाणीकरण

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **हैकट्रिक्स में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने की अनुमति** चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह
* प्राप्त करें [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **हैकिंग ट्रिक्स साझा करें और PRs सबमिट करें [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud)** के माध्यम से।

</details>

**यह जानकारी पोस्ट से निकाली गई थी:** [**https://www.tarlogic.com/en/blog/how-kerberos-works/**](https://www.tarlogic.com/en/blog/how-kerberos-works/)

## केरबेरोस (I): केरबेरोस कैसे काम करता है? - सिद्धांत

20 - मार्च - 2019 - एलॉय पेरेज़

इस पोस्ट की श्रृंखला का उद्देश्य केरबेरोस काम कैसे करता है, केवल हमलों को पेश करने से अधिक है। इसका कारण यह है कि कई बार कुछ तकनीकों का काम करने या नहीं करने का कारण स्पष्ट नहीं होता है। इस ज्ञान के पास होने से पेंटेस्ट में इन हमलों में से किसे उपयोग करना है, यह जानने में मदद मिलती है।

इसलिए, दस्तावेज़ीकरण में डुबकी लगाने और विषय के बारे में कई पोस्टों में डुबकी लगाने के बाद, हमने इस पोस्ट में लिखने का प्रयास किया है जिसमें सभी महत्वपूर्ण विवरण शामिल हैं जिन्हें एक अडिटर को जानना चाहिए कि केरबेरोस प्रोटोकॉल का उपयोग कैसे करें।

इस पहले पोस्ट में केवल मूलभूत कार्यक्षमता पर चर्चा की जाएगी। आगामी पोस्ट में हम देखेंगे कि हम हमले कैसे करें और ज्यादा जटिल पहलुओं काम करते हैं, जैसे कि अधिकारप्राप्ति।

यदि आपको विषय के बारे में कोई संदेह है जिसे अच्छी तरह से समझाया नहीं गया है, तो इसके बारे में एक टिप्पणी या प्रश्न पूछने से नहीं डरें। अब, विषय पर आगे बढ़ते हैं।

### केरबेरोस क्या है?

पहले, केरबेरोस एक प्रमाणीकरण प्रोटोकॉल है, अधिकृत नहीं। दूसरे शब्दों में, यह प्रत्येक उपयोगकर्ता की पहचान करने की अनुमति देता है, जो एक गुप्त पासवर्ड प्रदान करता है, हालांकि, यह यह नहीं सत्यापित करता है कि इस उपयोगकर्ता को कौन से संसाधनों या सेवाओं तक पहुंच हो सकती है।

केरबेरोस को एक्टिव डायरेक्टरी में उपयोग किया जाता है। इस प्लेटफ़ॉर्म में, केरबेरोस प्रत्येक उपयोगकर्ता की विशेषाधिकारों के बारे में जानकारी प्रदान करता है, लेकिन यह उपयोगकर्ता को इसके संसाधनों तक पहुंच होने की जिम्मेदारी है।

### केर
### प्रमाणीकरण प्रक्रिया

इस खंड में, प्रमाणीकरण को करने के लिए संदेशों की क्रमबद्धता का अध्ययन किया जाएगा, जहां एक उपयोगकर्ता बिना टिकट के शुरू होकर चाहिए सेवा के खिलाफ प्रमाणित किया जाएगा।

**KRB\_AS\_REQ**

पहले, उपयोगकर्ता को KDC से एक TGT प्राप्त करना होगा। इसे प्राप्त करने के लिए, एक KRB\_AS\_REQ भेजा जाना चाहिए:

![KRB\_AS\_REQ स्कीमा संदेश](<../../.gitbook/assets/image (175) (1).png>)

_KRB\_AS\_REQ_ में, इनमें से कुछ फ़ील्ड होते हैं:

* उपयोगकर्ता को प्रमाणित करने और पुनरावृत्ति हमलों से बचाने के लिए एन्क्रिप्टेड **टाइमस्टैम्प** के साथ, क्लाइंट कुंजी के साथ
* प्रमाणित किए गए उपयोगकर्ता का **उपयोगकर्ता नाम**
* **krbtgt** खाते से जुड़ी सेवा **SPN**
* उपयोगकर्ता द्वारा उत्पन्न एक **नॉन्स**

नोट: एन्क्रिप्टेड टाइमस्टैम्प केवल तभी आवश्यक होता है जब उपयोगकर्ता पूर्व प्रमाणीकरण की आवश्यकता होती है, जो सामान्य रूप से होता है, यदि [_DONT\_REQ\_PREAUTH_](https://support.microsoft.com/en-us/help/305144/how-to-use-the-useraccountcontrol-flags-to-manipulate-user-account-pro) फ़्लैग उपयोगकर्ता खाते में सेट होता है।

**KRB\_AS\_REP**

अनुरोध प्राप्त करने के बाद, KDC टाइमस्टैम्प को डिक्रिप्ट करके उपयोगकर्ता पहचान की पुष्टि करता है। यदि संदेश सही है, तो यह _KRB\_AS\_REP_ के साथ प्रतिक्रिया करनी चाहिए:

![KRB\_AS\_REP स्कीमा संदेश](<../../.gitbook/assets/image (176) (1).png>)

_KRB\_AS\_REP_ में निम्नलिखित जानकारी शामिल होती है:

* **उपयोगकर्ता नाम**
* **TGT**, जिसमें है:
* **उपयोगकर्ता नाम**
* **सत्र कुंजी**
* TGT की **समाप्ति तिथि**
* KDC द्वारा हस्ताक्षरित उपयोगकर्ता विशेषाधिकारों वाला **PAC**
* उपयोगकर्ता कुंजी के साथ कुछ **एन्क्रिप्टेड डेटा**, जिसमें है:
* **सत्र कुंजी**
* TGT की **समाप्ति तिथि**
* उपयोगकर्ता **नॉन्स**, पुनरावृत्ति हमलों से बचने के लिए

समाप्त होने के बाद, उपयोगकर्ता के पास पहले से ही TGT होता है, जिसे TGS का अनुरोध करने और फिर सेवाओं तक पहुंचने के लिए उपयोग किया जा सकता है।

**KRB\_TGS\_REQ**

TGS का अनुरोध करने के लिए, KDC को _KRB\_TGS\_REQ_ संदेश भेजना होगा:

![KRB\_TGS\_REQ स्कीमा संदेश](<../../.gitbook/assets/image (177).png>)

_KRB\_TGS\_REQ_ में शामिल होता है:

* सत्र कुंजी के साथ **एन्क्रिप्टेड डेटा**:
* **उपयोगकर्ता नाम**
* **टाइमस्टैम्प**
* **TGT**
* अनुरोधित सेवा का **SPN**
* उपयोगकर्ता द्वारा उत्पन्न एक **नॉन्स**

**KRB\_TGS\_REP**

_KRB\_TGS\_REQ_ संदेश प्राप्त करने के बाद, KDC एक TGS को _KRB\_TGS\_REP_ में वापस करता है:

![KRB\_TGS\_REP स्कीमा संदेश](<../../.gitbook/assets/image (178) (1).png>)

_KRB\_TGS\_REP_ में शामिल होता है:

* **उपयोगकर्ता नाम**
* **TGS**, जिसमें है:
* **सेवा सत्र कुंजी**
* **उपयोगकर्ता नाम**
* TGS की **समाप्ति तिथि**
* KDC द्वारा हस्ताक्षरित उपयोगकर्ता विशेषाधिकारों वाला **PAC**
* सत्र कुंजी के साथ **एन्क्रिप्टेड डेटा**:
* **सेवा सत्र कुंजी**
* TGS की **समाप्ति तिथि**
* उपयोगकर्ता **नॉन्स**, पुनरावृत्ति हमलों से बचने के लिए

**KRB\_AP\_REQ**

अंत में, यदि सब कुछ ठीक रहा है, तो उपयोगकर्ता के पास पहले से ही एक म
* Kerberoasting – Part 2: [https://room362.com/post/2016/kerberoast-pt2/](https://room362.com/post/2016/kerberoast-pt2/)
* Roasting AS-REPs: [https://www.harmj0y.net/blog/activedirectory/roasting-as-reps/](https://www.harmj0y.net/blog/activedirectory/roasting-as-reps/)
* PAC Validation: [https://passing-the-hash.blogspot.com.es/2014/09/pac-validation-20-minute-rule-and.html](https://passing-the-hash.blogspot.com.es/2014/09/pac-validation-20-minute-rule-and.html)
* Understanding PAC Validation: [https://blogs.msdn.microsoft.com/openspecification/2009/04/24/understanding-microsoft-kerberos-pac-validation/](https://blogs.msdn.microsoft.com/openspecification/2009/04/24/understanding-microsoft-kerberos-pac-validation/)
* Reset the krbtgt acoount password/keys: [https://gallery.technet.microsoft.com/Reset-the-krbtgt-account-581a9e51](https://gallery.technet.microsoft.com/Reset-the-krbtgt-account-581a9e51)
* Mitigating Pass-the-Hash (PtH) Attacks and Other Credential Theft: [https://www.microsoft.com/en-us/download/details.aspx?id=36036](https://www.microsoft.com/en-us/download/details.aspx?id=36036)
* Fun with LDAP, Kerberos (and MSRPC) in AD Environments: [https://speakerdeck.com/ropnop/fun-with-ldap-kerberos-and-msrpc-in-ad-environments?slide=58](https://speakerdeck.com/ropnop/fun-with-ldap-kerberos-and-msrpc-in-ad-environments?slide=58)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड** करने की अनुमति चाहिए? [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)** का** अनुसरण करें।**
* **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके अपना योगदान दें।**

</details>
