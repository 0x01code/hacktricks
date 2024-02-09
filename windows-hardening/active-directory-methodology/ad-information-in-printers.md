<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live) **पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks) github repos में PRs सबमिट करके।

</details>


इंटरनेट पर कई ब्लॉग हैं जो **प्रिंटर को LDAP के साथ डिफ़ॉल्ट/कमजोर लॉगऑन क्रेडेंशियल्स के साथ कॉन्फ़िगर छोड़ने के खतरों को हाइलाइट करते हैं**।\
यह इसलिए है क्योंकि एक हमलावर **प्रिंटर को एक रूग LDAP सर्वर के खिलाफ प्रमाणित करने में धोखा दे सकता है** (सामान्यत: `nc -vv -l -p 444` पर्याप्त होता है) और प्रिंटर **क्रेडेंशियल्स को स्पष्ट-पाठ में कैप्चर** करने के लिए।

इसके अलावा, कई प्रिंटर में **उपयोगकर्ता नामों के लॉग** हो सकते हैं या यह वास्तव में **डोमेन कंट्रोलर से सभी उपयोगकर्ता नामों को डाउनलोड** कर सकते हैं।

यह सभी **संवेदनशील जानकारी** और सामान्य **सुरक्षा की कमी** प्रिंटर को हमलावरों के लिए बहुत दिलचस्प बनाती है।

इस विषय पर कुछ ब्लॉग:

* [https://www.ceos3c.com/hacking/obtaining-domain-credentials-printer-netcat/](https://www.ceos3c.com/hacking/obtaining-domain-credentials-printer-netcat/)
* [https://medium.com/@nickvangilder/exploiting-multifunction-printers-during-a-penetration-test-engagement-28d3840d8856](https://medium.com/@nickvangilder/exploiting-multifunction-printers-during-a-penetration-test-engagement-28d3840d8856)

## प्रिंटर कॉन्फ़िगरेशन
- **स्थान**: LDAP सर्वर सूची यहाँ पाई जाती है: `नेटवर्क > LDAP सेटिंग > LDAP सेटिंग सेटअप`।
- **व्यवहार**: इंटरफेस LDAP सर्वर संशोधन को पुनः प्रमाणिकरण के बिना संशोधित करने की अनुमति देता है, उपयोगकर्ता सुविधा का लक्ष्य रखता है लेकिन सुरक्षा जोखिम उठाता है।
- **शोषण**: शोषण में LDAP सर्वर पता एक नियंत्रित मशीन पर पुनर्निर्देशित करने और "कनेक्शन का परीक्षण" सुविधा का उपयोग करके क्रेडेंशियल्स को कैप्चर करने का उपयोग करता है।

## क्रेडेंशियल्स को कैप्चर करना

**अधिक विस्तृत कदमों के लिए, मूल [स्रोत](https://grimhacker.com/2018/03/09/just-a-printer/) पर संदर्भित करें।**

### विधि 1: Netcat लिस्टनर
एक सरल नेटकैट लिस्टनर पर्याप्त हो सकता है:
```bash
sudo nc -k -v -l -p 386
```
### वाला यह विधि की सफलता भिन्न होती है।

### विधि 2: स्लैपड के साथ पूर्ण LDAP सर्वर
एक और विश्वसनीय दृष्टिकोण में, एक पूर्ण LDAP सर्वर सेटअप करना शामिल है क्योंकि प्रिंटर एक शून्य बाइंड का प्रदर्शन करता है जिसके बाद यह पहले क्यूरी करता है और फिर पहुंचने की कोशिश करता है।

1. **LDAP सर्वर सेटअप**: गाइड [इस स्रोत](https://www.server-world.info/en/note?os=Fedora_26&p=openldap) से कदमों का पालन करता है।
2. **मुख्य कदम**:
- OpenLDAP इंस्टॉल करें।
- व्यवस्थापक पासवर्ड कॉन्फ़िगर करें।
- मौलिक स्कीमा आयात करें।
- LDAP डीबी पर डोमेन नाम सेट करें।
- LDAP TLS कॉन्फ़िगर करें।
3. **LDAP सेवा क्रियान्वयन**: एक बार सेटअप कर लिया जाए, LDAP सेवा को निम्नलिखित का उपयोग करके चलाया जा सकता है:
```bash
slapd -d 2
```
## संदर्भ
* [https://grimhacker.com/2018/03/09/just-a-printer/](https://grimhacker.com/2018/03/09/just-a-printer/)


<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>
