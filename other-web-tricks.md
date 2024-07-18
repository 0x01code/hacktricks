# अन्य वेब ट्रिक्स

{% hint style="success" %}
**AWS हैकिंग सीखें और प्रैक्टिस करें:**<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम एक्सपर्ट (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
**GCP हैकिंग सीखें और प्रैक्टिस करें:** <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम एक्सपर्ट (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks का समर्थन करें</summary>

* [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर **फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>
{% endhint %}

### होस्ट हेडर

कई बार बैक-एंड **होस्ट हेडर** पर भरोसा करता है **कुछ क्रियाएँ करने के लिए**। उदाहरण के लिए, यह इसका मूल्य **डोमेन के रूप में उपयोग कर सकता है एक पासवर्ड रीसेट भेजने के लिए**। तो जब आप एक ईमेल प्राप्त करते हैं जिसमें अपना पासवर्ड रीसेट करने के लिए एक लिंक होता है, तो उपयोग किया जा रहा डोमेन वह है जिसे आपने होस्ट हेडर में डाला है। फिर, आप अन्य उपयोगकर्ताओं का पासवर्ड रीसेट करने का अनुरोध कर सकते हैं और उनके पासवर्ड रीसेट कोड चुरा सकते हैं जिसे आपके द्वारा नियंत्रित एक डोमेन में बदल सकते हैं। [WriteUp](https://medium.com/nassec-cybersecurity-writeups/how-i-was-able-to-take-over-any-users-account-with-host-header-injection-546fff6d0f2).

{% hint style="warning" %}
ध्यान दें कि संभावना है कि आपको उपयोगकर्ता को रीसेट पासवर्ड लिंक पर क्लिक करने का इंतजार नहीं करना पड़े, क्योंकि शायद **स्पैम फ़िल्टर या अन्य बीचक उपकरण/बॉट इसे विश्लेषित करने के लिए क्लिक करेंगे**।
{% endhint %}

### सत्र बूलियन

कभी-कभी जब आप कुछ सत्यापन सही तरीके से पूरा करते हैं तो बैक-एंड **केवल एक बूलियन जो मान "सच" है को एक सुरक्षा विशेषता में आपके सत्र में जोड़ देगा**। फिर, एक विभिन्न अंतर्वार्ता जानेगा कि क्या आपने उस जांच को सफलतापूर्वक पार किया है।\
हालांकि, यदि आप **जांच पार करते हैं** और आपके सत्र को सुरक्षा विशेषता में "सच" मान दिया जाता है, तो आप कोशिश कर सकते हैं **अन्य संसाधनों तक पहुंचने** की जो **उसी विशेषता पर निर्भर** करते हैं लेकिन जिनकी आपको **पहुंचने की अनुमति नहीं है**। [WriteUp](https://medium.com/@ozguralp/a-less-known-attack-vector-second-order-idor-attacks-14468009781a).

### रजिस्टर कार्यक्षमता

पहले से मौजूदा उपयोगकर्ता के रूप में पंजीकरण करने की कोशिश करें। समानांतर वर्ण (डॉट, बहुत सारे अंतरिक्ष और यूनिकोड) का उपयोग करने की कोशिश करें।

### ईमेल का अधिकार

एक ईमेल पंजीकरण करें, पुष्टि करने से पहले ईमेल बदलें, फिर, यदि नया पुष्टि ईमेल पहले पंजीकृत ईमेल पर भेजा जाता है, तो आप किसी भी ईमेल को अधिकार में ले सकते हैं। या यदि आप पहले वाले को दूसरे ईमेल की पुष्टि करने की अनुमति दे सकते हैं, तो आप किसी भी खाता को अधिकार में ले सकते हैं।

### कंपनियों के आटलासियन का आंतरिक सर्विसडेस्क एक्सेस

{% embed url="https://yourcompanyname.atlassian.net/servicedesk/customer/user/login" %}

### TRACE विधि

डेवलपर्स कई बार उत्पादन वातावरण में विभिन्न डीबगिंग विकल्पों को अक्षम करना भूल जाते हैं। उदाहरण के लिए, HTTP `TRACE` विधि निदानात्मक उद्देश्यों के लिए डिज़ाइन की गई है। यदि सक्षम है, तो वेब सर्वर उन अनुरोधों का जवाब देगा जो `TRACE` विधि का उपयोग करते हैं और प्रतिक्रिया में उस अनुरोध को गुणाकार करेगा जो प्राप्त किया गया था। यह व्यवहार अक्सर अहानिकारक होता है, लेकिन कभी-कभी सूचना खुलासा करता है, जैसे आंतरिक प्रमाणीकरण हेडरों का नाम जो पुनर्प्रोक्सी द्वारा अनुरोधों में जोड़े जा सकते हैं।![Image for post](https://miro.medium.com/max/60/1\*wDFRADTOd9Tj63xucenvAA.png?q=20)

![Image for post](https://miro.medium.com/max/1330/1\*wDFRADTOd9Tj63xucenvAA.png)


{% hint style="success" %}
**AWS हैकिंग सीखें और प्रैक्टिस करें:**<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम एक्सपर्ट (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
**GCP हैकिंग सीखें और प्रैक्टिस करें:** <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम एक्सपर्ट (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks का समर्थन करें</summary>

* [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर **फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>
{% endhint %}
