# FZ - इन्फ्रारेड

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ हैकट्रिक्स क्लाउड ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 ट्विटर 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ ट्विच 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 यूट्यूब 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **हैकट्रिक्स में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने** की उपलब्धता चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**द पीएस फैमिली**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल हों** या मुझे **ट्विटर** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)** का** **अनुसरण** करें।**
* **हैकिंग ट्रिक्स को शेयर करें द्वारा PRs सबमिट करके** [**हैकट्रिक्स रेपो**](https://github.com/carlospolop/hacktricks) **और** [**हैकट्रिक्स-क्लाउड रेपो**](https://github.com/carlospolop/hacktricks-cloud) **को।**

</details>

## परिचय <a href="#ir-signal-receiver-in-flipper-zero" id="ir-signal-receiver-in-flipper-zero"></a>

इन्फ्रारेड काम करने के बारे में अधिक जानकारी के लिए देखें:

{% content-ref url="../infrared.md" %}
[infrared.md](../infrared.md)
{% endcontent-ref %}

## फ्लिपर जीरो में आईआर सिग्नल रिसीवर <a href="#ir-signal-receiver-in-flipper-zero" id="ir-signal-receiver-in-flipper-zero"></a>

फ्लिपर जीरो एक डिजिटल आईआर सिग्नल रिसीवर TSOP का उपयोग करता है, जो **आईआर रिमोट से सिग्नल को अवरोधित करने की अनुमति देता है**। कुछ **स्मार्टफोन** जैसे Xiaomi में भी एक आईआर पोर्ट होता है, लेकिन ध्यान दें कि **इनमें से अधिकांश केवल सिग्नल ट्रांसमिट कर सकते हैं** और उन्हें **सिग्नल प्राप्त नहीं कर सकते** हैं।

फ्लिपर इन्फ्रारेड **रिसीवर काफी संवेदनशील है**। आप इसे तब भी **सिग्नल पकड़ सकते हैं** जब आप रिमोट और टीवी के बीच कहीं **बीच में रहते हैं**। फ्लिपर के आईआर पोर्ट की ओर सीधे रिमोट को देखाने की आवश्यकता नहीं है। यह उपयोगी होता है जब कोई टीवी के पास खड़ा होकर चैनल बदल रहा होता है, और आप और फ्लिपर दोनों दूरी में होते हैं।

आईआर सिग्नल के **डिकोडिंग** का काम **सॉफ़्टवेयर** द्वारा होता है, इसलिए फ्लिपर जीरो संभावित रूप से किसी भी आईआर रिमोट कोड के **प्राप्ति और प्रसारण का समर्थन** करता है। अज्ञात प्रोटोकॉल के मामले में जो पहचान नहीं की जा सकती है - यह उसे **यथार्थ रूप से रिकॉर्ड और प्लेबैक** करता है।

## क्रियाएँ

### यूनिवर्सल रिमोट

फ्लिपर जीरो को **यूनिवर्सल रिमोट के रूप में उपयोग** किया जा सकता है ताकि किसी भी टीवी, एयर कंडीशनर या मीडिया सेंटर को नियंत्रित किया जा सके। इस मोड में, फ्लिपर **एसडी कार्ड के शब्दकोश के अनुसार सभी समर्थित निर्माताओं के सभी ज्ञात कोड** का **ब्रूटफ़ोर्स** करता है। आपको एक विशेष रिमोट का चयन करने की आवश्यकता नहीं है एक रेस्टोरेंट टीवी को बं
