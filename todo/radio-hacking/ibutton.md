# आईबटन

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ हैकट्रिक्स क्लाउड ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 ट्विटर 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ ट्विच 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 यूट्यूब 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **हैकट्रिक्स में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करना चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**द पीएस फैमिली**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एकल [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक पीएस और हैकट्रिक्स स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **ट्विटर** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का पालन करें।**
* **हैकिंग ट्रिक्स साझा करें और** [**हैकट्रिक्स रेपो**](https://github.com/carlospolop/hacktricks) **और** [**हैकट्रिक्स-क्लाउड रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में पीआर जमा करके**।

</details>

## परिचय

आईबटन एक साधारण नाम है जो एक **सिक्के के आकार के धातु डिब्बे में पैक किए गए** इलेक्ट्रॉनिक पहचान कुंजी के लिए उपयोग होता है। इसे डैलस टच मेमोरी या संपर्क मेमोरी भी कहा जाता है। हालांकि, यह अक्सर गलती से "चुंबकीय" कुंजी के रूप में संदर्भित किया जाता है, लेकिन इसमें **कुछ भी चुंबकीय नहीं** होता है। वास्तव में, इसके भीतर एक पूर्ण-विकसित **माइक्रोचिप** जो एक डिजिटल प्रोटोकॉल पर संचालित होता है, छिपा होता है।

<figure><img src="../../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

### आईबटन क्या है? <a href="#what-is-ibutton" id="what-is-ibutton"></a>

सामान्यतः, आईबटन में कुंजी और रीडर के भौतिक रूप को दर्शाता है - एक गोल सिक्का जिसमें दो संपर्क होते हैं। इसके आसपास के फ्रेम के लिए, सबसे आम प्लास्टिक होलदार से लेकर छल्ले, पेंडेंट आदि तक कई विविधताएं होती हैं।

<figure><img src="../../.gitbook/assets/image (23) (2).png" alt=""><figcaption></figcaption></figure>

जब कुंजी रीडर तक पहुंचती है, तो **संपर्क स्पर्श होते हैं** और कुंजी को उसकी पहचान भेजने के लिए सक्षम किया जाता है। कभी-कभी कुंजी **तुरंत पढ़ी नहीं जाती** क्योंकि इंटरकॉम का **संपर्क पीएसडी** उससे अधिक बड़ा होता है। इसलिए कुंजी के बाहरी सीमाओं और रीडर के संपर्क नहीं हो सकते। यदि ऐसा है, तो आपको कुंजी को रीडर की दीवारों में से एक पर दबाना होगा।

<figure><img src="../../.gitbook/assets/image (21) (2).png" alt=""><figcaption></figcaption></figure>

### **1-वायर प्रोटोकॉल** <a href="#1-wire-protocol" id="1-wire-protocol"></a>

डैलस कुंजी 1-वायर प्रोटोकॉल का उपयोग करके डेटा आपस में विनिमय करती है। इसमें डेटा संचार के लिए केवल एक संपर्क (!!) होता है, दोनों दिशाओं में, मास्टर से स्लेव और उल्टे। 1-वायर प्रोटोकॉल मास्टर-स्लेव मॉडल के अनुसार काम करता है। इस टोपोलॉजी में, मास्टर हमेशा संचार आरंभ करता ह
