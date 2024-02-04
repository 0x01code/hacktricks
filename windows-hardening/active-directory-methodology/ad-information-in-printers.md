<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** का पालन करें।**
* **अपने हैकिंग ट्रिक्स साझा करें** PRs के माध्यम से [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में सबमिट करके।

</details>


इंटरनेट में कई ब्लॉग हैं जो **प्रिंटर को LDAP के साथ डिफ़ॉल्ट/कमजोर लॉगऑन क्रेडेंशियल्स के साथ छोड़ने के खतरों को हाइलाइट** करते हैं।\
इसका कारण यह है कि एक हमलाविता **प्रिंटर को धोखा देकर एक रूग LDAP सर्वर के खिलाफ प्रमाणीकरण करा सकता है** (सामान्यत: `nc -vv -l -p 444` पर्याप्त होता है) और प्रिंटर **क्रेडेंशियल्स को स्पष्ट-पाठ पर पकड़ सकता है**।

इसके अलावा, कई प्रिंटर में **उपयोगकर्ता नामों के लॉग** हो सकते हैं या वास्तव में **डोमेन कंट्रोलर से सभी उपयोगकर्ता नामों को डाउनलोड** कर सकते हैं।

यह सभी **संवेदनशील जानकारी** और सामान्य **सुरक्षा की कमी** प्रिंटर को हमलावरों के लिए बहुत रोचक बनाती है।

इस विषय पर कुछ ब्लॉग:

* [https://www.ceos3c.com/hacking/obtaining-domain-credentials-printer-netcat/](https://www.ceos3c.com/hacking/obtaining-domain-credentials-printer-netcat/)
* [https://medium.com/@nickvangilder/exploiting-multifunction-printers-during-a-penetration-test-engagement-28d3840d8856](https://medium.com/@nickvangilder/exploiting-multifunction-printers-during-a-penetration-test-engagement-28d3840d8856)

## प्रिंटर कॉन्फ़िगरेशन
- **स्थान**: LDAP सर्वर सूची यहाँ पाई जाती है: `नेटवर्क > LDAP सेटिंग > LDAP सेटिंग सेट करना।`
- **व्यवहार**: इंटरफेस बिना पुनः प्रमाणीकरण के LDAP सर्वर में परिवर्तन करने की अनुमति देता है, जिससे उपयोगकर्ता सुविधा का लक्ष्य होता है, लेकिन सुरक्षा के जोखिम उठाता है।
- **शोषण**: शोषण में LDAP सर्वर पता एक नियंत्रित मशीन पर पुनर्निर्देशित करने और "परीक्षण कनेक्शन" सुविधा का उपयोग करके क्रेडेंशियल्स को पकड़ने का उपयोग करता है।

## क्रेडेंशियल्स को पकड़ना

### विधि 1: Netcat लिस्टनर
एक सरल नेटकैट लिस्टनर पर्याप्त हो सकता है:
```bash
sudo nc -k -v -l -p 386
```
### तथ्य

हालांकि, इस विधि की सफलता भिन्न होती है।

### विधि 2: स्लैपडी के साथ पूर्ण LDAP सर्वर
एक और विश्वसनीय दृष्टिकोण में, एक पूर्ण LDAP सर्वर सेटअप करना शामिल है क्योंकि प्रिंटर क्रेडेंशियल बाइंडिंग की कोशिश से पहले एक नल बाइंड कार्य करता है।

1. **LDAP सर्वर सेटअप**: गाइड [इस स्रोत](https://www.server-world.info/en/note?os=Fedora_26&p=openldap) के चरणों का पालन करता है।
2. **मुख्य कदम**:
- OpenLDAP इंस्टॉल करें।
- व्यवस्थापक पासवर्ड कॉन्फ़िगर करें।
- मौलिक स्कीमा आयात करें।
- LDAP डीबी पर डोमेन नाम सेट करें।
- LDAP TLS कॉन्फ़िगर करें।
3. **LDAP सेवा क्रियान्वयन**: एक बार सेटअप कर लिया जाने पर, LDAP सेवा को निम्नलिखित का उपयोग करके चलाया जा सकता है:
```
slapd -d 2
```

**अधिक विस्तृत कदमों के लिए, मूल [स्रोत](https://grimhacker.com/2018/03/09/just-a-printer/) का संदर्भ देखें।**

# संदर्भ
* [https://grimhacker.com/2018/03/09/just-a-printer/](https://grimhacker.com/2018/03/09/just-a-printer/)
