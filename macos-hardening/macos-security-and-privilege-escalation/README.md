# macOS सुरक्षा और प्रिविलेज एस्केलेशन

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एकल [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* **हमारे साथ जुड़ें** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में शामिल हों या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का** अनुसरण करें।**
* **अपने हैकिंग ट्रिक्स को** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में पीआर जमा करके** अपने हैकिंग ट्रिक्स साझा करें।

</details>

<figure><img src="../../.gitbook/assets/image (1) (3) (1).png" alt=""><figcaption></figcaption></figure>

अनुभवी हैकर्स और बग बाउंटी हंटर्स के साथ संवाद करने के लिए [**HackenProof Discord**](https://discord.com/invite/N3FrSbmwdy) सर्वर में शामिल हों!

**हैकिंग इंसाइट्स**\
हैकिंग के रोमांच और चुनौतियों में डूबने वाली सामग्री के साथ संघर्ष करें

**रीयल-टाइम हैक न्यूज़**\
रीयल-टाइम समाचार और अद्यतनों के माध्यम से तेजी से बदलती हैकिंग दुनिया में अद्यतित रहें

**नवीनतम घोषणाएं**\
नवीनतम बग बाउंटी लॉन्च और महत्वपूर्ण प्लेटफॉर्म अपडेट के साथ अवगत रहें

**हमारे साथ** [**Discord**](https://discord.com/invite/N3FrSbmwdy) **में शामिल हों** और आज ही शीर्ष हैकर्स के साथ सहयोग करना शुरू करें!

## मूल MacOS

यदि आप macOS के परिचय से अवगत नहीं हैं, तो आपको macOS के मूल तत्वों का अध्ययन करना चाहिए:

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

* कर्नल की **वास्तुरचि**

{% content-ref url="mac-os-architecture/" %}
[mac-os-architecture](mac-os-architecture/)
{% endcontent-ref %}

* सामान्य macOS नेटवर्क सेवाएं और प्रोटोकॉल्स**

{% content-ref url="macos-protocols.md" %}
[macos-protocols.md](macos-protocols.md)
{% endcontent-ref %}

* **ओपनसोर्स** macOS: [https://opensource.apple.com/](https://opensource.apple.com/)
* `tar.gz` डाउनलोड करने के लिए URL को बदलें, जैसे [https://opensource.apple.com/**source**/dyld/](https://opensource.apple.com/source/dyld/) से [https://opensource.apple.com/**tarballs**/dyld/**dyld-852.2.tar.gz**](https://opensource.apple.com/tarballs/dyld/dyld-852.2.tar.gz)

### MacOS MDM

कंपनियों में **macOS** सिस्टमों को बहुत संभावित रूप से **एमडीएम के साथ प्रबंधित किया जाएगा**। इसलिए, एक हमलावर के दृष्टिकोण से यह जानना महत्वपूर्ण है कि **यह कैसे काम करता है**:

{% content-ref url="../macos-red-teaming/macos-mdm/" %}
[macos-mdm](../macos-red-teaming/macos-mdm/)
{% endcontent-ref %}

### MacOS - निरीक्षण, डीबगिंग और फ़ज़िंग

{% content-ref url="macos-apps-inspecting-debugging-and-fuzzing/" %}
[macos-apps-inspecting-debugging-and-fuzzing](macos-apps-inspecting-debugging-and-fuzzing/)
{% endcontent-ref %}

## MacOS सुरक्षा संरक्षण

{% content-ref url="macos-security-protections/" %}
[macos-security-protections](macos-security-protections/)
{% endcontent-ref %}

## हमला सतह

### फ़ाइल अनुमतियाँ

यदि एक **रूट के रूप में चल रही प्रक्रिया** एक ऐसी फ़ाइल लिखती है जिसे एक उपयोगकर्ता नियंत्रित कर सकता है, तो उपयोगकर्ता इसे **प्रिविलेज एस्केलेशन** के लिए दुरुपयोग कर सकता है।\
यह निम्नलिखित स्थितियों में हो सकता है:

* उपयोगकर्त
### फ़ाइल एक्सटेंशन और URL स्कीम ऐप हैंडलर्स

अजीब ऐप्स जो फ़ाइल एक्सटेंशन के द्वारा पंजीकृत हैं, इस्तेमाल किए जा सकते हैं और विभिन्न एप्लिकेशन विशेष प्रोटोकॉल खोलने के लिए पंजीकृत किए जा सकते हैं।

{% content-ref url="macos-file-extension-apps.md" %}
[macos-file-extension-apps.md](macos-file-extension-apps.md)
{% endcontent-ref %}

## macOS TCC / SIP विशेषाधिकार उन्नयन

macOS में **एप्लिकेशन और बाइनरी को अनुमतियाँ हो सकती हैं** जिनसे वे अन्यों की तुलना में अधिक विशेषाधिकारी हो सकते हैं।

इसलिए, एक हमलावर जो एक macOS मशीन को सफलतापूर्वक कंप्रोमाइज़ करना चाहता है, उसे अपनी TCC विशेषाधिकारों को **उन्नयन करने** (या यदि उसकी आवश्यकता हो तो **SIP को छलना** भी) होगा।

ये विशेषाधिकारों काम वाले अनुमतियों के रूप में दिए जाते हैं, जिनके साथ एप्लिकेशन साइन किया जाता है, या एप्लिकेशन ने कुछ एक्सेस मांगे हों और उन्हें **उपयोगकर्ता द्वारा मंजूरी देने** के बाद वे **TCC डेटाबेस** में पाए जा सकते हैं। एक प्रक्रिया इन विशेषाधिकारों को प्राप्त करने का एक और तरीका है जब वह किसी प्रक्रिया के **बच्चे के रूप में** होती है जिसके पास वे **विशेषाधिकार** होते हैं क्योंकि वे आमतौर पर **वारिस** होते हैं।

इन लिंक्स का पालन करें और जानें कि [**TCC में विशेषाधिकारों को उन्नयन कैसे करें**](macos-security-protections/macos-tcc/#tcc-privesc-and-bypasses), [**TCC को छलना कैसे करें**](macos-security-protections/macos-tcc/macos-tcc-bypasses/) और पिछले में [**SIP को छलना कैसे किया गया है**](macos-security-protections/macos-sip.md#sip-bypasses)।

## macOS पारंपरिक विशेषाधिकार उन्नयन

बेशक एक लाल टीम के दृष्टिकोण से आपको रूट तक उन्नयन करने में भी रुचि होनी चाहिए। कुछ संकेतों के लिए निम्नलिखित पोस्ट की जांच करें:

{% content-ref url="macos-privilege-escalation.md" %}
[macos-privilege-escalation.md](macos-privilege-escalation.md)
{% endcontent-ref %}

## संदर्भ

* [**OS X Incident Response: Scripting and Analysis**](https://www.amazon.com/OS-Incident-Response-Scripting-Analysis-ebook/dp/B01FHOHHVS)
* [**https://taomm.org/vol1/analysis.html**](https://taomm.org/vol1/analysis.html)
* [**https://github.com/NicolasGrimonpont/Cheatsheet**](https://github.com/NicolasGrimonpont/Cheatsheet)
* [**https://assets.sentinelone.com/c/sentinal-one-mac-os-?x=FvGtLJ**](https://assets.sentinelone.com/c/sentinal-one-mac-os-?x=FvGtLJ)
* [**https://www.youtube.com/watch?v=vMGiplQtjTY**](https://www.youtube.com/watch?v=vMGiplQtjTY)

<figure><img src="../../.gitbook/assets/image (1) (3) (1).png" alt=""><figcaption></figcaption></figure>

[**HackenProof Discord**](https://discord.com/invite/N3FrSbmwdy) सर्वर में शामिल हों और अनुभवी हैकर्स और बग बाउंटी हंटर्स के साथ संवाद करें!

**हैकिंग इंसाइट्स**\
हैकिंग के रोमांच और चुनौतियों पर आधारित सामग्री के साथ संघर्ष करें

**रीयल-टाइम हैक न्यूज़**\
रीयल-टाइम समाचार और अंतर्दृष्टि के माध्यम से तेजी से बदलती हैकिंग दुनिया के साथ अद्यतित रहें

**नवीनतम घोषणाएं**\
नवीनतम बग बाउंटी लॉन्च और महत्वपूर्ण प्लेटफ़ॉर्म अपडेट के साथ अवगत रहें

**हमारे साथ जुड़ें** [**Discord**](https://discord.com/invite/N3FrSbmwdy) और आज ही शीर्ष हैकर्स के साथ सहयोग करना शुरू करें!

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने की पहुंच** चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल** हों या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolop
