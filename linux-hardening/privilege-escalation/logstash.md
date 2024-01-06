<details>

<summary><strong>शून्य से नायक तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS रेड टीम विशेषज्ञ)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग प्राप्त करें**](https://peass.creator-spring.com)
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह में शामिल हों**](https://discord.gg/hRep4RUj7f) या [**telegram समूह**](https://t.me/peass) या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>


# मूल जानकारी

Logstash का उपयोग लॉग्स को एकत्रित करने, परिवर्तित करने और आउटपुट करने के लिए किया जाता है। यह **पाइपलाइन्स** का उपयोग करके साकार किया जाता है, जिसमें इनपुट, फिल्टर और आउटपुट मॉड्यूल होते हैं। यह सेवा तब दिलचस्प हो जाती है जब एक मशीन को समझौता किया गया होता है जो Logstash को एक सेवा के रूप में चला रही होती है।

## पाइपलाइन्स

पाइपलाइन कॉन्फ़िगरेशन फ़ाइल **/etc/logstash/pipelines.yml** सक्रिय पाइपलाइन्स के स्थानों को निर्दिष्ट करती है:
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
यहाँ आप **.conf** फाइलों के पथ पा सकते हैं, जिनमें कॉन्फ़िगर किए गए पाइपलाइन्स होते हैं। यदि **Elasticsearch आउटपुट मॉड्यूल** का उपयोग किया जाता है, तो **पाइपलाइन्स** में एक Elasticsearch इंस्टेंस के लिए मान्य **क्रेडेंशियल्स** होने की संभावना होती है। ये क्रेडेंशियल्स अक्सर अधिक अधिकार वाले होते हैं, क्योंकि Logstash को Elasticsearch में डेटा लिखना होता है। यदि वाइल्डकार्ड्स का उपयोग किया जाता है, तो Logstash उस फोल्डर में स्थित सभी पाइपलाइन्स को चलाने की कोशिश करता है जो वाइल्डकार्ड से मेल खाते हैं।

## लिखने योग्य पाइपलाइन्स के साथ प्रिवेस्क

अपने अधिकारों को बढ़ाने की कोशिश करने से पहले आपको यह जांचना चाहिए कि कौन सा उपयोगकर्ता logstash सेवा चला रहा है, क्योंकि आप बाद में उसी उपयोगकर्ता को अपना बना लेंगे। डिफ़ॉल्ट रूप से logstash सेवा **logstash** उपयोगकर्ता के अधिकारों के साथ चलती है।

जांचें कि क्या आपके पास आवश्यक अधिकारों में से **एक** है:

* आपके पास किसी पाइपलाइन **.conf** फाइल पर **लिखने की अनुमति** है **या**
* **/etc/logstash/pipelines.yml** में एक वाइल्डकार्ड होता है और आपको निर्दिष्ट फोल्डर में लिखने की अनुमति है

आगे **एक** आवश्यकता पूरी होनी चाहिए:

* आप logstash सेवा को पुनः आरंभ करने में सक्षम हैं **या**
* **/etc/logstash/logstash.yml** में प्रविष्टि **config.reload.automatic: true** होती है

यदि एक वाइल्डकार्ड निर्दिष्ट है, तो उस वाइल्डकार्ड से मेल खाने वाली फाइल बनाने की कोशिश करें। निम्नलिखित सामग्री को फाइल में लिखा जा सकता है ताकि आदेशों को निष्पादित किया जा सके:
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
**इंटरवल** सेकंड्स में समय निर्दिष्ट करता है। इस उदाहरण में **whoami** कमांड हर 120 सेकंड में निष्पादित होती है। कमांड का आउटपुट **/tmp/output.log** में सहेजा जाता है।

यदि **/etc/logstash/logstash.yml** में प्रविष्टि **config.reload.automatic: true** होती है तो आपको केवल कमांड निष्पादित होने तक प्रतीक्षा करनी होगी, क्योंकि Logstash स्वचालित रूप से नए पाइपलाइन कॉन्फ़िगरेशन फाइलों या मौजूदा पाइपलाइन कॉन्फ़िगरेशनों में किसी भी परिवर्तन को पहचान लेगा। अन्यथा logstash सेवा को पुनः आरंभ करने के लिए ट्रिगर करें।

यदि कोई वाइल्डकार्ड का उपयोग नहीं किया जाता है, तो आप इन परिवर्तनों को मौजूदा पाइपलाइन कॉन्फ़िगरेशन में लागू कर सकते हैं। **सुनिश्चित करें कि आप चीजों को नहीं तोड़ते हैं!**

# संदर्भ

* [https://insinuator.net/2021/01/pentesting-the-elk-stack/](https://insinuator.net/2021/01/pentesting-the-elk-stack/)


<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>
