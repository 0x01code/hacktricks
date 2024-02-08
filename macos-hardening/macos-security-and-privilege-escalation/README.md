# macOS सुरक्षा और विशेषाधिकार उन्नयन

<details>

<summary><strong>शून्य से हीरो तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप अपनी कंपनी का विज्ञापन HackTricks में देखना चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं तो [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **हमारे साथ जुड़ें** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में शामिल हों या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** द्वारा PRs सबमिट करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

<figure><img src="../../.gitbook/assets/image (1) (3) (1).png" alt=""><figcaption></figcaption></figure>

[**HackenProof Discord**](https://discord.com/invite/N3FrSbmwdy) सर्वर में शामिल होकर अनुभवी हैकर्स और बग बाउंटी हंटर्स के साथ संवाद करें!

**हैकिंग इंसाइट्स**\
हैकिंग के रोमांच और चुनौतियों में डूबने वाली सामग्री के साथ जुड़ें

**रियल-टाइम हैक न्यूज़**\
तेजी से बदलते हैकिंग विश्व के साथ कदम से कदम रखें न्यूज़ और अंदाज

**नवीनतम घोषणाएं**\
नवीनतम बग बाउंटी लॉन्च और महत्वपूर्ण प्लेटफॉर्म अपडेट्स के साथ सूचित रहें

**हमारे साथ जुड़ें** [**Discord**](https://discord.com/invite/N3FrSbmwdy) और आज ही शीर्ष हैकर्स के साथ सहयोग करना शुरू करें!

## मूल MacOS

यदि आप macOS के परिचित नहीं हैं, तो आपको macOS की मूल जानकारी शुरू करनी चाहिए:

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
* `tar.gz` डाउनलोड करने के लिए एक URL बदलें जैसे [https://opensource.apple.com/**source**/dyld/](https://opensource.apple.com/source/dyld/) से [https://opensource.apple.com/**tarballs**/dyld/**dyld-852.2.tar.gz**](https://opensource.apple.com/tarballs/dyld/dyld-852.2.tar.gz)।

### MacOS MDM

कंपनियों में **macOS** सिस्टमों को बहुत संभावित रूप से **MDM के साथ प्रबंधित किया जाएगा**। इसलिए, हमलेवर के दृष्टिकोण से यह जानना दिलचस्प है कि **यह कैसे काम करता है**:

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

यदि **रूट के रूप में चल रही प्रक्रिया** एक फ़ाइल लिखती है जिसे एक उपयोगकर्ता द्वारा नियंत्रित किया जा सकता है, तो उपयोगकर्ता इसका दुरुपयोग करके **विशेषाधिकारों को उन्नत कर सकता है**।\
यह निम्नलिखित स्थितियों में हो सकता है:

* उपयोग की गई फ़ाइल पहले से उपयोगकर्ता द्वारा बनाई गई थी (उपयोगकर्ता द्वारा स्वामित)
* उपयोग की गई फ़ाइल एक समूह के कारण उपयोगकर्ता द्वारा लिखी जा सकती है
* उपयोग की गई फ़ाइल उपयोगकर्ता द्वारा स्वामित एक निर्देशिका के अंदर है (उपयोगकर्ता फ़ाइल बना सकता है)
* उपयोग की गई फ़ाइल रूट द्वारा स्वामित एक निर्देशिका के अंदर है लेकिन उपयोगकर्ता के पास इसके लिखने का अधिकार है क्योंकि एक समूह के कारण (उपयोगकर्ता फ़ाइल बना सकता है)

**रूट द्वारा उपयोग की जाने वाली एक फ़ाइल बनाने** की क्षमता, एक उपयोगकर्ता को इसकी सामग्री का लाभ उठाने या यहाँ वहाँ इसे दिखाने के लिए **सिमलिंक/हार्डलिंक** बनाने की अनुमति देती है।

इस प्रकार की सुरक्षा दोषों के लिए **विकल्पशील `.pkg` इंस्टॉलर** की जांच न भूलें:

{% content-ref url="macos-files-folders-and-binaries/macos-installers-abuse.md" %}
[macos-installers-abuse.md](macos-files-folders-and-binaries/macos-installers-abuse.md)
{% endcontent-ref %}



### फ़ाइल एक्सटेंशन और URL स्कीम ऐप हैंडलर

अजीब ऐप्स जो फ़ाइल एक्सटेंशन द्वारा पंजीकृत हैं, उनका दुरुपयोग किया जा सकता है और विभिन्न एप्लिकेशन विशेष प्रोटोकॉल खोलने के लिए पंजीकृत किए जा सकते हैं

{% content-ref url="macos-file-extension-apps.md" %}
[macos-file-extension-apps.md](macos-file-extension-apps.md)
{% endcontent-ref %}

## macOS TCC / SIP विशेषाधिकार उन्नयन

macOS में **एप्लिकेशन और बाइनरी की अनुमतियाँ** हो सकती हैं जो उन्हें अन्यों से अधिक विशेषाधिकृत बना सकती हैं।

इसलिए, एक हमलावर जो एक macOS मशीन को सफलतापूर्वक कंप्रमाइज़ करना चाहता है, उसे अपनी TCC विशेषाधिकारों को **उन्नत करने** की आवश्यकता होगी (या यहाँ तक कि **SIP को छलना**, उसकी आवश्यकता के आधार पर)।

इन विशेषाधिकारों को आम तौर पर एप्लिकेशन के साथ हस्ताक्षरित किए गए **अधिकारों** के रूप में दिया जाता है, या एप्लिकेशन सक्षमताएँ मांग सकती हैं और उसके बाद **उपयोगकर्ता द्वारा मंजूरी देने** के बाद वे **TCC डेटाबेस** में पाए जा सकते हैं। एक प्रक्रिया इन विशेषाधिकारों को प्राप्त करने का एक और तरीका यह है कि यह उन विशेषाधिकारों वाले एक प्रक्रिया का **बच्चा हो** क्योंकि वे आम तौर पर **विरासत में** होते हैं।

इन लिंकों का पालन करें और विभिन्न तरीकों को खोजें [**TCC में विशेषाधिकारों को उन्नत करने**](macos-security-protections/macos-tcc/#tcc-privesc-and-bypasses), [**TCC को छलना**](macos-security-protections/macos-tcc/macos-tcc-bypasses/) और पिछले में कैसे [**SIP को छला गया**](macos-security-protections/macos-sip.md#sip-bypasses)।

## macOS पारंपरिक विशेष
