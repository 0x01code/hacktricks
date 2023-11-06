# ACLs - DACLs/SACLs/ACEs

<figure><img src="../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करके आसानी से बनाएं और **स्वचालित कार्यप्रवाह** बनाएं, जो दुनिया के **सबसे उन्नत सामुदायिक उपकरणों** द्वारा संचालित होते हैं।\
आज ही पहुंच प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप चाहते हैं कि आपकी **कंपनी HackTricks में विज्ञापित** की जाए? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की अनुमति चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह
* प्राप्त करें [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का** अनुसरण करें।**
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को**

</details>

## **पहुंच नियंत्रण सूची (ACL)**

एक **ACL एक क्रमबद्ध सूची है** जो एक ऑब्जेक्ट और उसकी गुणवत्ताओं पर लागू सुरक्षा सुरक्षा को परिभाषित करती है। प्रत्येक **ACE** एक सुरक्षा **प्रिंसिपल** की पहचान करता है और उस सुरक्षा प्रिंसिपल के लिए अनुमति देने वाले, निषेधित या ऑडिट करने वाले एक सेट के अधिकारों को निर्धारित करता है।

एक ऑब्जेक्ट की सुरक्षा विवरणिका में **दो ACLs** हो सकते हैं:

1. एक **DACL** जो उन **उपयोगकर्ताओं** और **समूहों** की **पहचान करता है** जिन्हें **पहुंच** की **अनुमति** या **निषेध** है
2. एक **SACL** जो **पहुंच** को **ऑडिट कैसे** करेगा नियंत्रित करता है

जब एक उपयोगकर्ता फ़ाइल तक पहुंचने की कोशिश करता है, विंडोज सिस्टम एक AccessCheck चलाता है और सुरक्षा विवरणिका को उपयोगकर्ता के पहुंच टोकन के साथ तुलना करता है और मूल्यांकन करता है कि क्या उपयोगकर्ता को पहुंच मिली है और ACEs सेट के आधार पर किस प्रकार की पहुंच मिली है।

### **विवेकाधीन पहुंच नियंत्रण सूची (DACL)**

DACL (अक्सर ACL के रूप में उल्लेख किया जाता है) उन उपयोगकर्ताओं और समूहों की पहचान करता है जिन्हें ऑब्जेक्ट पर पहुंच अनुमति या निषेध दी गई है। इसमें सुरक्षित ऑब्जेक्ट के लिए जोड़े गए ACEs (खाता + पहुंच अधिकार) की एक सूची होती है।

### **सिस्टम पहुंच नियंत्रण सूची (SACL)**

SACLs सुरक्षित ऑब्जेक्ट्स तक पहुंच की निगरानी करने की संभावना बनाते हैं। SACL में ACEs निर्धारित करते हैं कि **सुरक्षा घटना लॉग में कौन सी पहुंच लॉग की गई है**। मॉनिटरिंग उपकरणों के साथ यह उचित लोगों को अल
### ACEs का क्रम

क्योंकि **सिस्टम ACEs की जांच करना बंद कर देता है जब अनुरोधित पहुंच स्पष्ट रूप से दिया जाता है या इनकार किया जाता है**, इसलिए DACL में ACEs का क्रम महत्वपूर्ण है।

DACL में ACEs की प्राथमिक क्रम को "कैननिकल" क्रम कहा जाता है। Windows 2000 और Windows Server 2003 के लिए, कैननिकल क्रम निम्नलिखित है:

1. सभी **निर्दिष्ट** ACEs को किसी समूह के **पहले** रखा जाता है **इनहेरिटेड** ACEs से पहले।
2. **निर्दिष्ट** ACEs के समूह में, **पहुंच-इनकार** ACEs को **पहुंच-अनुमति** ACEs से पहले रखा जाता है।
3. **इनहेरिटेड** समूह के भीतर, ACEs जो **बाल वस्त्र के माता-पिता से इनहेरिटेड** होते हैं, उनके बाद ACEs जो **दादा-दादी से इनहेरिटेड** होते हैं, और इसी तरह आगे जाते हैं। इसके बाद, **पहुंच-इनकार** ACEs को **पहुंच-अनुमति** ACEs से पहले रखा जाता है।

निम्नलिखित चित्र में ACEs का कैननिकल क्रम दिखाया गया है:

### ACEs का कैननिकल क्रम

![ACE](https://www.ntfs.com/images/screenshots/ACEs.gif)

कैननिकल क्रम सुनिश्चित करता है कि निम्नलिखित बातें होती हैं:

* एक स्पष्ट **पहुंच-इनकार ACE को संपूर्ण स्पष्ट पहुंच-अनुमति ACE के बावजूद प्रयोग किया जाता है**। इसका मतलब है कि ऑब्जेक्ट के मालिक अनुमति निर्धारित कर सकता है जो उपयोगकर्ताओं के समूह को पहुंच देती है और उस समूह के एक उपसमूह को पहुंच इनकार करती है।
* सभी **निर्दिष्ट ACEs को किसी भी इनहेरिटेड ACE से पहले प्रोसेस किया जाता है**। यह विवेकाधीन पहुंच नियंत्रण की अवधारणा के साथ मेल खाता है: एक बाल ऑब्जेक्ट (उदाहरण के लिए एक फ़ाइल) की पहुंच बाल के मालिक के विवेक पर निर्भर करती है, न कि माता-पिता ऑब्जेक्ट (उदाहरण के लिए एक फ़ोल्डर) के मालिक के विवेक पर। एक बाल ऑब्जेक्ट के मालिक बच्चे के ऊपर सीधे अनुमतियाँ परिभाषित कर सकता है। परिणामस्वरूप, इनहेरिटेड अनुमतियों के प्रभाव संशोधित हो जाते हैं।

<figure><img src="../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करें और आसानी से वर्कफ़्लो बनाएं और संचालित करें, जो दुनिया के **सबसे उन्नत** समुदाय उपकरणों द्वारा संचालित होते हैं।\
आज ही पहुंच प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

### GUI उदाहरण

यह फ़ोल्डर का क्लासिक सुरक्षा टैब है जिसमें ACL, DACL और ACEs दिखाए जाते हैं:

![](../../.gitbook/assets/classicsectab.jpg)

यदि हम **एडवांस्ड बटन** पर क्लिक करेंगे तो हमें इनहेरिटेंस जैसे अधिक विकल्प मिलेंगे:

![](../../.gitbook/assets/aceinheritance.jpg)

और यदि आप एक सुरक्षा प्रिंसिपल जोड़ें या संपादित करें:

![](../../.gitbook/assets/editseprincipalpointers1.jpg)

और अंत में हमारे पास ऑडिटिंग टैब में SACL है:

![](../../.gitbook/assets/audit-tab.jpg)

### उदाहरण: समूह के लिए स्पष्ट पहुंच-इनकार

इस उदाहरण में, पहुंच-अनुमति समूह हर कोई है और पहुंच-इनकार समूह मार्केटिंग है, हर कोई का एक उपसमूह।

आप एक कॉस्ट फ़ोल्डर को मार्केटिंग समूह को पहुंच देने से रोकना चाहते हैं। यदि कॉस्ट फ़ोल्डर के ACEs कैननिकल क्रम में होंगे, तो मार्केटिंग को इनकार करने वाला ACE हर किसी के इजाज
### पहुंच नियंत्रण प्रविष्टि लेआउट

| ACE फ़ील्ड   | विवरण                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| ----------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| प्रकार        | ACE का प्रकार दर्शाता है। Windows 2000 और Windows Server 2003 में छह प्रकार के ACE का समर्थन किया जाता है: सभी सुरक्षित वस्तुओं के साथ जुड़े तीन सामान्य ACE प्रकार। तीन वस्तु-विशिष्ट ACE प्रकार जो केवल Active Directory वस्तुओं के लिए हो सकते हैं।                                                                                                                                                                                                                                                            |
| ध्वज       | इनहेरिटेंस और ऑडिटिंग को नियंत्रित करने वाले बिट ध्वज का सेट।                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| आकार        | ACE के लिए आवंटित मेमोरी के बाइटों की संख्या।                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| पहुंच मास्क | 32-बिट मान जिसके बिट वस्तु के लिए पहुंच अधिकारों के साथ मेल खाते हैं। बिट को चालू या बंद किया जा सकता है, लेकिन सेटिंग का अर्थ ACE प्रकार पर निर्भर करता है। उदाहरण के लिए, यदि पहुंच अधिकार के लिए अधिकारियों को पढ़ने के लिए बिट चालू है, और ACE प्रकार निषेध है, तो ACE पहुंच अधिकार को निषेध करता है। यदि वही बिट सेट है लेकिन ACE प्रकार अनुमति है, तो ACE पहुंच अधिकार को अनुमति देता है। पहुंच मास्क के अधिक विवरण अगले तालिका में दिए गए हैं। |
| SID         | इस ACE द्वारा नियंत्रित या मॉनिटर किए जाने वाले उपयोगकर्ता या समूह को पहचानता है।                                                                                                                                                                                                                                                                                                                                                                                                                                 |

### पहुंच मास्क लेआउट

| बिट (सीमा) | अर्थ                            | विवरण/उदाहरण                       |
| ----------- | ---------------------------------- | ----------------------------------------- |
| 0 - 15      | वस्तु विशिष्ट पहुंच अधिकार      | डेटा पढ़ें, निष्पादित करें, डेटा जोड़ें           |
| 16 - 22     | मानक पहुंच अधिकार             | हटाएँ, ACL लिखें, मालिक लिखें            |
| 23          | सुरक्षा ACL तक पहुंच संभव       |                                           |
| 24 - 27     | आरक्षित                           |                                           |
| 28          | सार्वभौमिक सब (पढ़ें, लिखें, निष्पादित करें) | नीचे सब कुछ                          |
| 29          | सार्वभौमिक निष्पादित करें                    | किसी प्रोग्राम को निष्पादित करने के लिए सभी चीजें |
| 30          | सार्वभौमिक लिखें                      | एक फ़ाइल में लिखने के लिए सभी चीजें   |
| 31          | सार्वभौमिक पढ़ें                       | एक फ़ाइल को पढ़ने के लिए सभी चीजें       |

## संदर्भ

* [https://www.ntfs.com/ntfs-permissions-acl-use.htm](https://www.ntfs.com/ntfs-permissions-acl-use.htm)
* [https://secureidentity.se/acl-dacl-sacl-and-the-ace/](https://secureidentity.se/acl-dacl-sacl-and-the-ace/)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आप **PEASS के नवीनतम संस्करण का उपयोग करना चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का** अनुसरण करें।**
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को।**

</details>

<figure><img src="../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करें और आसानी से बनाएं और **स्वचालित कार्यप्रवाह** बनाएं जो दुनिया के **सबसे उन्नत** समुदाय उपकरणों द्वारा संचालित होता है।\
आज ही पहुंच प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}
