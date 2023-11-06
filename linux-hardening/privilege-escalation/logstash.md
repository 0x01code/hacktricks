<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks क्लाउड ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 ट्विटर 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ ट्विच 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 यूट्यूब 🎥</strong></a></summary>

- क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित करना चाहते हैं**? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS परिवार**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)

- प्राप्त करें [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके साझा करें।**

</details>


# मूलभूत जानकारी

Logstash लॉग्स को संग्रहित, परिवर्तित और आउटपुट करने के लिए उपयोग किया जाता है। इसे **पाइपलाइन** का उपयोग करके प्राप्त किया जाता है, जिसमें इनपुट, फ़िल्टर और आउटपुट मॉड्यूल होते हैं। यह सेवा रोचक होती है जब एक मशीन को हैक कर लिया जाता है जिसमें Logstash को सेवा के रूप में चलाया जाता है।

## पाइपलाइन

पाइपलाइन कॉन्फ़िगरेशन फ़ाइल **/etc/logstash/pipelines.yml** निर्दिष्ट करती है कि सक्रिय पाइपलाइनों के स्थान क्या हैं:
```bash
# This file is where you define your pipelines. You can define multiple.
# For more information on multiple pipelines, see the documentation:
# https://www.elastic.co/guide/en/logstash/current/multiple-pipelines.html

- pipeline.id: main
path.config: "/etc/logstash/conf.d/*.conf"
- pipeline.id: example
path.config: "/usr/share/logstash/pipeline/1*.conf"
pipeline.workers: 6
```
यहां आपको **.conf** फ़ाइलों के पथ मिलेंगे, जिनमें कॉन्फ़िगर किए गए पाइपलाइन्स होती हैं। यदि **Elasticsearch आउटपुट मॉड्यूल** का उपयोग किया जाता है, तो **पाइपलाइन्स** में Elasticsearch इंस्टेंस के लिए मान्य **क्रेडेंशियल्स** होने की संभावना होती है। ये क्रेडेंशियल्स अक्सर अधिक विशेषाधिकारों के साथ होते हैं, क्योंकि Logstash को Elasticsearch में डेटा लिखना होता है। यदि वाइल्डकार्ड का उपयोग किया जाता है, तो Logstash उस फ़ोल्डर में स्थित सभी पाइपलाइन्स को चलाने का प्रयास करता है जो वाइल्डकार्ड के साथ मेल खाती हैं।

## लिखने योग्य पाइपलाइन्स के साथ Privesc

अपनी अधिकारों को उन्नत करने की कोशिश करने से पहले, आपको जांचना चाहिए कि लॉगस्टैश सेवा किस उपयोगकर्ता के द्वारा चलाई जा रही है, क्योंकि यह उपयोगकर्ता आपके पास होगा, जिसके बाद आप स्वामित्व में होंगे। डिफ़ॉल्ट रूप से लॉगस्टैश सेवा **लॉगस्टैश** उपयोगकर्ता की विशेषाधिकारों के साथ चलती है।

यहां आपको चाहिए कि आप में से **एक** आवश्यक अधिकारों में से एक हों:

* आपके पास एक पाइपलाइन **.conf** फ़ाइल पर **लिखने की अनुमति** हो या
* **/etc/logstash/pipelines.yml** में एक वाइल्डकार्ड है और आपको निर्दिष्ट फ़ोल्डर में लिखने की अनुमति है

इसके अलावा, निम्नलिखित में से **एक** आवश्यकता पूरी होनी चाहिए:

* आप लॉगस्टैश सेवा को पुनः आरंभ कर सकते हैं **या**
* **/etc/logstash/logstash.yml** में प्रविष्टि होनी चाहिए **config.reload.automatic: true**

यदि एक वाइल्डकार्ड निर्दिष्ट है, तो उस वाइल्डकार्ड के मेल खाते एक फ़ाइल बनाने का प्रयास करें। निम्नलिखित सामग्री फ़ाइल में लिखी जा सकती है ताकि कमांड निष्पादित हो सकें:
```bash
input {
exec {
command => "whoami"
interval => 120
}
}

output {
file {
path => "/tmp/output.log"
codec => rubydebug
}
}
```
**अंतराल** सेकंड में समय निर्धारित करता है। इस उदाहरण में **whoami** कमांड हर 120 सेकंड में निष्पादित की जाती है। कमांड का आउटपुट **/tmp/output.log** में सहेजा जाता है।

यदि **/etc/logstash/logstash.yml** में **config.reload.automatic: true** एंट्री होती है, तो आपको केवल कमांड के निष्पादन का इंतजार करना होगा, क्योंकि Logstash स्वचालित रूप से नई पाइपलाइन कॉन्फ़िगरेशन फ़ाइलों या मौजूदा पाइपलाइन कॉन्फ़िगरेशन में किसी भी परिवर्तन को स्वीकार करेगा। अन्यथा, logstash सेवा को पुनरारंभ करें।

यदि कोई वाइल्डकार्ड उपयोग नहीं किया जाता है, तो आप मौजूदा पाइपलाइन कॉन्फ़िगरेशन में इन परिवर्तनों को लागू कर सकते हैं। **सुनिश्चित करें कि आप चीजों को नहीं तोड़ते हैं!**

# संदर्भ

* [https://insinuator.net/2021/01/pentesting-the-elk-stack/](https://insinuator.net/2021/01/pentesting-the-elk-stack/)


<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह।

- प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर सबमिट करके साझा करें।**

</details>
