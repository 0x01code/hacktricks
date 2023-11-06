# अन्य वेब ट्रिक्स

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आप **PEASS के नवीनतम संस्करण का उपयोग करना चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT संग्रह**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल हों** या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)** का पालन करें**.
* **अपने हैकिंग ट्रिक्स को** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में पीआर जमा करके अपने ट्रिक्स साझा करें**.

</details>

### होस्ट हैडर

कई बार बैक-एंड **होस्ट हैडर** पर भरोसा करता है कि कुछ कार्रवाई करें। उदाहरण के लिए, यह इसका मूल्य उपयोग करके **पासवर्ड रीसेट भेजने के लिए डोमेन** के रूप में इस्तेमाल कर सकता है। तो जब आपको अपना पासवर्ड रीसेट करने के लिए एक लिंक वाला ईमेल मिलता है, तो उपयोग किया जा रहा डोमेन वही होता है जिसे आपने होस्ट हैडर में डाला है। फिर, आप अन्य उपयोगकर्ताओं के पासवर्ड रीसेट को अनुरोध कर सकते हैं और उनके पासवर्ड रीसेट कोड चुराने के लिए अपने नियंत्रण में एक डोमेन पर बदल सकते हैं। [WriteUp](https://medium.com/nassec-cybersecurity-writeups/how-i-was-able-to-take-over-any-users-account-with-host-header-injection-546fff6d0f2).

{% hint style="warning" %}
ध्यान दें कि यह संभव है कि आपको उपयोगकर्ता को रीसेट पासवर्ड लिंक पर क्लिक करने की प्रतीक्षा नहीं करनी पड़े, क्योंकि शायद **स्पैम फ़िल्टर या अन्य बीचक उपकरण / बॉट इसे विश्लेषण करने के लिए क्लिक करेंगे**।
{% endhint %}

### सत्र बूलियन

कभी-कभी जब आप कुछ सत्यापन सही तरीके से पूरा करते हैं, तो बैक-एंड **आपके सत्र में सुरक्षा गुणधर्म के लिए "True" के साथ एक बूलियन जोड़ देगा**। फिर, एक अलग एंडपॉइंट जानेगा कि क्या आपने उस जांच को सफलतापूर्वक पार किया है।\
हालांकि, यदि आप **जांच पास करते हैं** और आपके सत्र को सुरक्षा गुणधर्म में "True" मान दिया जाता है, तो आप उन संसाधनों का प्रयास कर सकते हैं जो **उसी गुणधर्म पर निर्भर करते हैं** लेकिन आपको **पहुँचने की अनुमति नहीं होनी चाहिए**। [WriteUp](https://medium.com/@ozguralp/a-less-known-attack-vector-second-order-idor-attacks-14468009781a).

### पंजीकरण कार्यक्षमता

एक पहले से मौजूदा उपयोगकर्ता के रूप में पंजीकरण करने का प्रयास करें। समानार्थक वर्णों (डॉट, बहुत सारे अंतरिक्ष और यूनिकोड) का उपयोग करने का प्रयास करें।

### ईमेल का तबादला

एक ईमेल पंजीकरण करें, पुष्टि करने से पहले ईमेल बदलें, फिर, यदि नया पुष्टिकरण ईमेल पहले पंजीकृत ईमेल पर भेज
