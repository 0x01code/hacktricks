# macOS सुरक्षा और विशेषाधिकार वृद्धि

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert) के साथ शून्य से नायक तक AWS हैकिंग सीखें</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**टेलीग्राम समूह**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>

<figure><img src="../../.gitbook/assets/image (1) (3) (1).png" alt=""><figcaption></figcaption></figure>

[**HackenProof Discord**](https://discord.com/invite/N3FrSbmwdy) सर्वर में शामिल हों और अनुभवी हैकर्स और बग बाउंटी हंटर्स के साथ संवाद करें!

**Hacking Insights**\
हैकिंग की उत्तेजना और चुनौतियों में गहराई से जाने वाली सामग्री के साथ जुड़ें

**Real-Time Hack News**\
रियल-टाइम समाचार और अंतर्दृष्टि के माध्यम से तेजी से बढ़ते हैकिंग विश्व के साथ अद्यतन रहें

**Latest Announcements**\
नवीनतम बग बाउंटीज़ लॉन्चिंग और महत्वपूर्ण प्लेटफॉर्म अपडेट्स के साथ सूचित रहें

[**Discord**](https://discord.com/invite/N3FrSbmwdy) पर हमसे जुड़ें और आज ही शीर्ष हैकर्स के साथ सहयोग करना शुरू करें!

## मूल MacOS

यदि आप macOS के साथ परिचित नहीं हैं, तो आपको macOS की मूल बातें सीखना शुरू करना चाहिए:

* विशेष macOS **फाइलें और अनुमतियां:**

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

* k**ernel** की **आर्किटेक्चर**

{% content-ref url="mac-os-architecture/" %}
[mac-os-architecture](mac-os-architecture/)
{% endcontent-ref %}

* सामान्य macOS n**etwork सेवाएं और प्रोटोकॉल**

{% content-ref url="macos-protocols.md" %}
[macos-protocols.md](macos-protocols.md)
{% endcontent-ref %}

* **Opensource** macOS: [https://opensource.apple.com/](https://opensource.apple.com/)
* `tar.gz` डाउनलोड करने के लिए, [https://opensource.apple.com/**source**/dyld/](https://opensource.apple.com/source/dyld/) जैसे URL को [https://opensource.apple.com/**tarballs**/dyld/**dyld-852.2.tar.gz**](https://opensource.apple.com/tarballs/dyld/dyld-852.2.tar.gz) में बदलें

### MacOS MDM

कंपनियों में **macOS** सिस्टम्स को अधिक संभावना से **MDM के साथ प्रबंधित किया जाएगा**। इसलिए, एक हमलावर के दृष्टिकोण से यह जानना दिलचस्प है कि **यह कैसे काम करता है**:

{% content-ref url="../macos-red-teaming/macos-mdm/" %}
[macos-mdm](../macos-red-teaming/macos-mdm/)
{% endcontent-ref %}

### MacOS - निरीक्षण, डिबगिंग और फज़िंग

{% content-ref url="macos-apps-inspecting-debugging-and-fuzzing/" %}
[macos-apps-inspecting-debugging-and-fuzzing](macos-apps-inspecting-debugging-and-fuzzing/)
{% endcontent-ref %}

## MacOS सुरक्षा संरक्षण

{% content-ref url="macos-security-protections/" %}
[macos-security-protections](macos-security-protections/)
{% endcontent-ref %}

## अटैक सरफेस

### फाइल अनुमतियां

यदि **root के रूप में चल रही प्रक्रिया** एक फाइल लिखती है जिसे उपयोगकर्ता द्वारा नियंत्रित किया जा सकता है, तो उपयोगकर्ता इसका दुरुपयोग करके **विशेषाधिकार वृद्धि** कर सकता है।\
यह निम्नलिखित स्थितियों में हो सकता है:

* फाइल पहले से ही एक उपयोगकर्ता द्वारा बनाई गई थी (उपयोगकर्ता द्वारा स्वामित्व)
* फाइल उपयोगकर्ता द्वारा एक समूह के कारण लिखने योग्य है
* फाइल एक ऐसी निर्देशिका के अंदर है जिसका स्वामित्व उपयोगकर्ता के पास है (उपयोगकर्ता फाइल बना सकता है)
* फाइल एक ऐसी निर्देशिका के अंदर है जिसका स्वामित्व root के पास है लेकिन उपयोगकर्ता के पास एक समूह के कारण इस पर लिखने की पहुंच है (उपयोगकर्ता फाइल बना सकता है)

**root द्वारा उपयोग की जाने वाली फाइल बनाने में सक्षम होना**, उपयोगकर्ता को इसकी सामग्री का लाभ उठाने या यहां तक कि **symlinks/hardlinks** बनाने की अनुमति देता है जो इसे दूसरी जगह पर इंगित करता है।

इस प्रकार की कमजोरियों के लिए **vulnerable `.pkg` installers की जांच करना न भूलें**:

{% content-ref url="macos-files-folders-and-binaries/macos-installers-abuse.md" %}
[macos-installers-abuse.md](macos-files-folders-and-binaries/macos-installers-abuse.md)
{% endcontent-ref %}



### फाइल एक्सटेंशन और URL स्कीम ऐप हैंडलर्स

फाइल एक्सटेंशन द्वारा पंजीकृत अजीब ऐप्स का दुरुपयोग किया जा सकता है और विभिन्न अनुप्रयोगों को विशिष्ट प्रोटोकॉल खोलने के लिए पंजीकृत किया जा सकता है

{% content-ref url="macos-file-extension-apps.md" %}
[macos-file-extension-apps.md](macos-file-extension-apps.md)
{% endcontent-ref %}

## macOS TCC / SIP विशेषाधिकार वृद्धि

macOS में **अनुप्रयोगों और बाइनरीज़ को अनुमतियां हो सकती हैं** जो उन्हें दूसरों की तुलना में अधिक विशेषाधिकार प्रदान करती हैं।

इसलिए, एक हमलावर जो macOS मशीन को सफलतापूर्वक समझौता करना चाहता है, उसे अपनी TCC विशेषाधिकारों को **वृद्धि करने की आवश्यकता होगी** (या यहां तक कि **SIP को बायपास करना होगा**, उसकी जरूरतों के आधार पर)।

इन विशेषाधिकारों को आमतौर पर अनुप्रयोग के साथ हस्ताक्षरित **entitlements** के रूप में दिया जाता है, या अनुप्रयोग ने कुछ पहुंचों का अनुरोध किया होगा और **उपयोगकर्ता की मंजूरी के बाद** वे **TCC डेटाबेस** में पाए जा सकते हैं। एक प्रक्रिया इन विशेषाधिकारों को प्राप्त करने का एक और तरीका उन प्रक्रियाओं का **बच्चा होना** है जिनके पास वे **विशेषाधिकार** हैं क्योंकि वे आमतौर पर **विरासत में मिले** होते हैं।

TCC में विशेषाधिकार वृद्धि करने के विभिन्न तरीकों को खोजने के लिए इन लिंक्स का अनुसरण करें, TCC को [**बायपास करने**](macos-security-protections/macos-tcc/macos-tcc-bypasses/) के लिए और अतीत में कैसे [**SIP को बायपास किया गया है**](macos-security-protections/macos-sip.md#sip-bypasses)।

## macOS पारंपरिक विशेषाधिकार वृद्धि

बेशक, एक लाल टीम के दृष्टिकोण से आपको root तक वृद्धि करने में भी रुचि होनी चाहिए। कुछ संकेतों के लिए निम्नलिखित पोस्ट द
