# इन्फ्रारेड

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ हैकट्रिक्स क्लाउड ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 ट्विटर 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ ट्विच 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 यूट्यूब 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **हैकट्रिक्स में विज्ञापित करना** चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल हों** या मुझे **ट्विटर** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का** **अनुसरण** करें।**
* **अपने हैकिंग ट्रिक्स को** [**हैकट्रिक्स रेपो**](https://github.com/carlospolop/hacktricks) **और** [**हैकट्रिक्स-क्लाउड रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में पीआर जमा करके साझा करें।**

</details>

## इन्फ्रारेड कैसे काम करता है <a href="#how-the-infrared-port-works" id="how-the-infrared-port-works"></a>

**मानवों के लिए इन्फ्रारेड प्रकाश अदृश्य होता है**। आईआर तरंगदैर्घ्य **0.7 से 1000 माइक्रोन** तक होता है। घरेलू रिमोट्स डेटा प्रसारण के लिए आईआर सिग्नल का उपयोग करते हैं और 0.75..1.4 माइक्रोन के तरंगदैर्घ्य सीमा में कार्य करते हैं। रिमोट में एक माइक्रोकंट्रोलर आईआर एलईडी को एक विशेष आवृत्ति के साथ चमकाता है, जिससे डिजिटल सिग्नल को आईआर सिग्नल में बदल दिया जाता है।

आईआर सिग्नल प्राप्त करने के लिए एक **फोटोरिसीवर** का उपयोग किया जाता है। यह आईआर प्रकाश को वोल्टेज पल्स में परिवर्तित करता है, जो पहले से ही **डिजिटल सिग्नल** होते हैं। आमतौर पर, प्राप्तकर्ता में एक **डार्क लाइट फ़िल्टर** होता है, जो केवल चाहिए वेवलेंथ को आने देता है और शोर को काट देता है।

### विभिन्न आईआर प्रोटोकॉल <a href="#variety-of-ir-protocols" id="variety-of-ir-protocols"></a>

आईआर प्रोटोकॉल में 3 कारकों में अंतर होता है:

* बिट एनकोडिंग
* डेटा संरचना
* कैरियर फ़्रिक्वेंसी - अक्सर 36..38 किलोहर्ट्ज़ के रेंज में

#### बिट एनकोडिंग तरीके <a href="#bit-encoding-ways" id="bit-encoding-ways"></a>

**1. पल्स दूरी एनकोडिंग**

बिट्स को पल्स के बीच के समय की अवधि को मोड्यूलेट करके एनकोड किया जाता है। पल्स की चौड़ाई स्वयं निरंतर होती है।

<figure><img src="../../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

**2. पल्स चौड़ाई एनकोडिंग**

बिट्स को पल्स चौड़ाई के माध्यम से एनकोड किया जाता है। पल्स बर्स्ट के बाद के स्थान की चौड़ाई स्थिर होती है।

<figure><img src="../../.gitbook/assets/image (29) (1).png" alt=""><figcaption></figcaption></figure>

**3. फेज एनकोडिंग**

इसे मैंचेस्टर एनकोडिंग के रूप में भी जाना जाता है। तार्किक मान को पल्स बर्स्ट और स्थान के बीच के प्रतिपोलिता द्वारा परिभाषित किया जाता है। "स्थान से पल्स बर्स्ट" तार्किक "0" को दर्शाता है, "पल्स बर्स्ट से स्थान" तार्किक "1" को दर्शाता है।

<figure><img src="../../.gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

**4. पिछले तरीकों का संयोजन और अन्य अनोखे**

{% hint style="info" %}
कुछ ऐसे आईआर प्रोटोकॉल हैं जो कई प्रकार के उपकरणों के लिए **विश्वसनीय बनने की कोशिश** कर रहे हैं। सबसे
### एयर कंडीशनर्स

अन्य रिमोटों की तुलना में, **एयर कंडीशनर्स केवल दबाए गए बटन का कोड नहीं भेजते हैं**। वे यह भी **सुनिश्चित करने के लिए सभी जानकारी भेजते हैं** कि जब एक बटन दबाया जाता है, तो एयर कंडीशनर मशीन और रिमोट सिंक्रनाइज़ हो जाएं।\
इससे यह बचाया जाएगा कि एक मशीन जिसे 20ºC के रूप में सेट किया गया है, उसे एक रिमोट के साथ 21ºC तक बढ़ाया जाए, और फिर जब एक और रिमोट का उपयोग करके तापमान को और बढ़ाया जाता है, तो यह "बढ़ा" कर 21ºC (और नहीं 22ºC में बढ़ाया जाता है जो कि 21ºC में होने की धारणा करता है)।

### हमले

आप फ्लिपर जीरो के साथ इन्फ्रारेड को हमला कर सकते हैं:

{% content-ref url="flipper-zero/fz-infrared.md" %}
[fz-infrared.md](flipper-zero/fz-infrared.md)
{% endcontent-ref %}

## संदर्भ

* [https://blog.flipperzero.one/infrared/](https://blog.flipperzero.one/infrared/)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **हैकट्रिक्स में विज्ञापित** देखना चाहते हैं? या क्या आप **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड** करने का उद्देश्य रखते हैं? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह।
* प्राप्त करें [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का** अनुसरण करें।**
* **अपने हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud)।

</details>
