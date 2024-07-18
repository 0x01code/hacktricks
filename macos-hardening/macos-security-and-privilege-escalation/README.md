# macOS सुरक्षा और प्रिविलेज उन्नति

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम विशेषज्ञ (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम विशेषज्ञ (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **जुड़ें** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) से या **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर हमें **फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, हैकट्रिक्स**](https://github.com/carlospolop/hacktricks) और [**हैकट्रिक्स क्लाउड**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में PR जमा करके।

</details>
{% endhint %}

<figure><img src="../../.gitbook/assets/image (380).png" alt=""><figcaption></figcaption></figure>

[**HackenProof Discord**](https://discord.com/invite/N3FrSbmwdy) सर्वर में शामिल होकर अनुभवी हैकर्स और बग बाउंटी हंटर्स के साथ संवाद करें!

**हैकिंग इंसाइट्स**\
हैकिंग के रोमांच और चुनौतियों में डूबने वाली सामग्री के साथ जुड़ें

**रियल-टाइम हैक समाचार**\
तेजी से बदलती हैकिंग दुनिया के साथ अद्यतन रहें और अंतर्दृष्टि प्राप्त करें

**नवीनतम घोषणाएं**\
नवीनतम बग बाउंटी और महत्वपूर्ण प्लेटफॉर्म अपडेट्स के साथ सूचित रहें

**हमारे साथ जुड़ें** [**Discord**](https://discord.com/invite/N3FrSbmwdy) और आज ही शीर्ष हैकर्स के साथ सहयोग करना शुरू करें!

## मूल MacOS

यदि आप macOS के बारे में नहीं जानते हैं, तो आपको macOS की मूल बातों का अध्ययन करना चाहिए:

* विशेष macOS **फ़ाइलें और अनुमतियाँ:**

{% content-ref url="macos-files-folders-and-binaries/" %}
[macos-files-folders-and-binaries](macos-files-folders-and-binaries/)
{% endcontent-ref %}

* सामान्य macOS **उपयोगकर्ता**

{% content-ref url="macos-users.md" %}
[macos-users.md](macos-users.md)
{% endcontent-ref %}

* **AppleFS**

{% content-ref url="macos-applefs.md" %}
[macos-applefs.md](macos-applefs.md)
{% endcontent-ref %}

* कर्नेल की **वास्तुकला**

{% content-ref url="mac-os-architecture/" %}
[mac-os-architecture](mac-os-architecture/)
{% endcontent-ref %}

* सामान्य macOS नेटवर्क सेवाएं और प्रोटोकॉल्स**

{% content-ref url="macos-protocols.md" %}
[macos-protocols.md](macos-protocols.md)
{% endcontent-ref %}

* **ओपनसोर्स** macOS: [https://opensource.apple.com/](https://opensource.apple.com/)
* `tar.gz` डाउनलोड करने के लिए एक URL को बदलें जैसे [https://opensource.apple.com/**source**/dyld/](https://opensource.apple.com/source/dyld/) से [https://opensource.apple.com/**tarballs**/dyld/**dyld-852.2.tar.gz**](https://opensource.apple.com/tarballs/dyld/dyld-852.2.tar.gz)

### MacOS MDM

कंपनियों में **macOS** सिस्टमों को बहुत संभावित रूप से **एक MDM के साथ प्रबंधित किया जाएगा**। इसलिए, हमलावार के दृष्टिकोण से यह जानना दिलचस्प है कि **यह कैसे काम करता है**:

{% content-ref url="../macos-red-teaming/macos-mdm/" %}
[macos-mdm](../macos-red-teaming/macos-mdm/)
{% endcontent-ref %}

### MacOS - जांच, डीबगिंग और फजिंग

{% content-ref url="macos-apps-inspecting-debugging-and-fuzzing/" %}
[macos-apps-inspecting-debugging-and-fuzzing](macos-apps-inspecting-debugging-and-fuzzing/)
{% endcontent-ref %}

## MacOS सुरक्षा संरक्षण

{% content-ref url="macos-security-protections/" %}
[macos-security-protections](macos-security-protections/)
{% endcontent-ref %}

## हमला सतह

### फ़ाइल अनुमतियाँ

यदि **रूट के रूप में चल रही प्रक्रिया** एक फ़ाइल लिखती है जिसे उपयोगकर्ता द्वारा नियंत्रित किया जा सकता है, तो उपयोगकर्ता इसका दुरुपयोग करके **प्रिविलेज उन्नति** कर सकता है।\
यह निम्नलिखित स्थितियों में हो सकता है:

* उपयोगकर्ता द्वारा पहले से बनाई गई फ़ाइल का उपयोग किया गया था (उपयोगकर्ता द्वारा स्वामित्व)
* उपयोगकर्ता द्वारा लिखने योग्य फ़ाइल क्योंकि एक समूह के कारण यह लिखने योग्य है
* उपयोगकर्ता द्वारा स्वामित्व वाले एक निर्देशिका में उपयोग की गई फ़ाइल (उपयोगकर्ता फ़ाइल बना सकता है)
* उपयोगकर्ता द्वारा लिखने योग्य एक निर्देशिका में उपयोग की गई फ़ाइल जिसका स्वामित्व रूट द्वारा है लेकिन उपयोगकर्ता के पास इसके उपर लिखने का अधिकार है क्योंकि एक समूह के कारण (उपयोगकर्ता फ़ाइल बना सकता है)

**रूट द्वारा उपयोग किया जाने वाली एक फ़ाइल बनाने** की क्षमता, उपयोगकर्ता को इसकी सामग्री का लाभ उठाने या यहाँ वहाँ पॉइंट करने के लिए **सिमलिंक/हार्डलिंक** बनाने की अनुमति देती है।

इस प्रकार की कमजोरियों के लिए विचारशील `.pkg` इंस्टॉलर्स** की जांच न भूलें:

{% content-ref url="macos-files-folders-and-binaries/macos-installers-abuse.md" %}
[macos-installers-abuse.md](macos-files-folders-and-binaries/macos-installers-abuse.md)
{% endcontent-ref %}

### फ़ाइल एक्सटेंशन और URL स्कीम ऐप हैंडलर

अजीब ऐप्स जो फ़ाइल एक्सटेंशन से पंजीकृत हैं, उनका दुरुपयोग किया जा सकता है और विभिन्न एप्लिकेशन विशेष प्रोटोकॉल खोलने के लिए पंजीकृत किए जा सकते हैं

{% content-ref url="macos-file-extension-apps.md" %}
[macos-file-extension-apps.md](macos-file-extension-apps.md)
{% endcontent-ref %}

## macOS TCC / SIP प्रिविलेज उन्नति

macOS में **एप्लिकेशन और बाइनरी** को फ़ोल्डर या सेटिंग्स तक पहुंचने की अनुमति हो सकती है जो उन्हें दूसरों से अधिक प्रिविलेज्ड बनाती है।

इसलिए, एक हमलावर जो एक macOS मशीन को सफलतापूर्वक कंप्रमाइज करना चाहता है, उसे अपनी TCC प्रिविलेजेज को **उन्नत करने** की आवश्यकता होगी (या यहाँ तक कि उसकी आवश्यकताओं के आधार पर **SIP को छलना** भी)।

इन प्रिविलेजेज आम तौर पर एप्लिकेशन के साथ **एंटाइटलमेंट्स** के रूप में दिए जाते हैं, या एप्लिकेशन ने कुछ पहुंचने की मांग की हो और उसके बाद **उपयोगकर्ता ने उन्हें स्वीकृति दी** हो तो वे **TCC डेटाबेस** में पाए जा सकते हैं। एक प्रक्रिया इन प्रिविलेजेज को प्राप्त करने का एक और तरीका यह है कि यह उन प्रिविलेजेज के साथ एक प्रक्रिया के **बच्चा** हो जो उन प्रिविलेजेज के साथ होती है क्योंकि वे आम तौर पर **विरासत में मिलते** हैं।

इन लिंक्स का पालन करें और विभिन्न तरीकों को खोजने के लिए [**TCC में प्रिविलेज उन्नति**](macos-security-protections/macos-tcc/#tcc-privesc-and-bypasses), [**TCC को छलना**](macos-security-protections/macos-tcc/macos-tcc-bypasses/) और पिछले में कैसे [**SIP को छला गया था**](macos-security-prote
## संदर्भ

* [**OS X घटना प्रतिक्रिया: स्क्रिप्टिंग और विश्लेषण**](https://www.amazon.com/OS-Incident-Response-Scripting-Analysis-ebook/dp/B01FHOHHVS)
* [**https://taomm.org/vol1/analysis.html**](https://taomm.org/vol1/analysis.html)
* [**https://github.com/NicolasGrimonpont/Cheatsheet**](https://github.com/NicolasGrimonpont/Cheatsheet)
* [**https://assets.sentinelone.com/c/sentinal-one-mac-os-?x=FvGtLJ**](https://assets.sentinelone.com/c/sentinal-one-mac-os-?x=FvGtLJ)
* [**https://www.youtube.com/watch?v=vMGiplQtjTY**](https://www.youtube.com/watch?v=vMGiplQtjTY)

<figure><img src="../../.gitbook/assets/image (380).png" alt=""><figcaption></figcaption></figure>

[**HackenProof Discord**](https://discord.com/invite/N3FrSbmwdy) सर्वर में शामिल होकर अनुभवी हैकर्स और बग बाउंटी हंटर्स के साथ संवाद करें!

**हैकिंग इंसाइट्स**\
हैकिंग के रोमांच और चुनौतियों में डूबने वाली सामग्री के साथ जुड़ें

**रियल-टाइम हैक न्यूज़**\
रियल-टाइम समाचार और अंदरूनी दुनिया के साथ अपडेट रहें

**नवीनतम घोषणाएं**\
नवीनतम बग बाउंटी लॉन्च और महत्वपूर्ण प्लेटफ़ॉर्म अपडेट के साथ सूचित रहें

[**Discord**](https://discord.com/invite/N3FrSbmwdy) पर हमारे साथ जुड़ें और आज ही शीर्ष हैकर्स के साथ सहयोग करना शुरू करें!

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम एक्सपर्ट (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम एक्सपर्ट (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* **💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में शामिल हों या हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में PR जमा करके।

</details>
{% endhint %}
