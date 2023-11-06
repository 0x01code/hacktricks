# लेखक के बारे में

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करना चाहते हैं? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल** हों या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में PR जमा करके साझा करें**.

</details>

### नमस्ते!!

मैं **कार्लोस पोलोप** हूँ।

सबसे पहले, मैं यह दर्शाना चाहता हूँ कि **मुझे इस पूरी किताब का मालिक नहीं हूँ**, बहुत सी **जानकारी को अन्य वेबसाइटों से कॉपी/पेस्ट किया गया है और वह सामग्री उनकी है** (यह पृष्ठों पर इंद्रजालिक रूप से दिखाया गया है)।

मैं यह भी कहना चाहता हूँ कि **इंटरनेट पर साइबर सुरक्षा से संबंधित जानकारी मुफ्त में साझा करने वाले सभी लोगों का धन्यवाद** करता हूँ। उनकी मदद से मैं हैकिंग तकनीकों को सीखता हूँ जिन्हें मैं फिर Hacktricks में जोड़ता हूँ।\
इसके अलावा, मैं यहां HackTricks में अपने खुद के शोध के परिणाम भी **लिखता हूँ**।

इसलिए, मैं उम्मीद करता हूँ कि **HackTricks** में आपको इंटरनेट पर एक **विषय** के बारे में उपलब्ध सभी **सुरक्षा ट्रिक्स** + संभावित रूप से कुछ **अतिरिक्त** चीजें मिलेंगी। अगर आपको कुछ **अनुपस्थित** लगता है, कृपया, Hacktricks Github पर **पुल अनुरोध** भेजें!

### जीवनी

मेरा **लिंक्डइन** देखें: [https://www.linkedin.com/in/carlos-polop-martin/](https://www.linkedin.com/in/carlos-polop-martin/)

{% hint style="warning" %}
यदि आपको HackTricks बहुत उपयोगी लगता है, तो कृपया **इसे समर्थन करने का विचार करें**!
{% endhint %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करना चाहते हैं? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल** हों या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में PR जमा करके साझा करें**.

</details>
