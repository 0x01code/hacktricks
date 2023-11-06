<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित करना** चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)

- प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके साझा करें।**

</details>


# अखंडता स्तर

Windows Vista से, सभी **संरक्षित ऑब्जेक्टों को अखंडता स्तर के साथ लेबल लगाया जाता है**। सिस्टम पर अधिकांश उपयोगकर्ता और सिस्टम फ़ाइल और रजिस्ट्री कुंजीयों को "मध्यम" अखंडता का डिफ़ॉल्ट लेबल होता है। प्राथमिक अपवाद एक सेट के रूप में हैं, जिन्हें इंटरनेट एक्सप्लोरर 7 द्वारा लिखने योग्य निर्धारित किया जाता है। **अधिकांश प्रक्रियाएं** मानक **उपयोगकर्ताओं** द्वारा **मध्यम अखंडता** के साथ चलाई जाती हैं (यहां तक कि वे जो प्रशासक समूह के भीतर एक उपयोगकर्ता द्वारा शुरू की गई हों), और अधिकांश **सेवाएं** **सिस्टम अखंडता** के साथ चलाई जाती हैं। रूट निर्देशिका को उच्च-अखंडता लेबल से संरक्षित किया जाता है।\
ध्यान दें कि **एक निम्न अखंडता स्तर वाली प्रक्रिया एक उच्च अखंडता स्तर वाले ऑब्जेक्ट में लिखने के लिए नहीं कर सकती है**।\
यहां कई अखंडता स्तर हैं:

* **अविश्वसनीय** - अविश्वसनीय रूप से लॉग ऑन होने वाली प्रक्रियाएं स्वचालित रूप से अविश्वसनीय निर्धारित की जाती हैं। _उदाहरण: Chrome_
* **निम्न** - निम्न अखंडता स्तर इंटरनेट के साथ इंटरैक्शन के लिए डिफ़ॉल्ट रूप से उपयोग किया जाता है। जब तक इंटरनेट एक्सप्लोरर अपनी डिफ़ॉल्ट स्थिति, सुरक्षित मोड, में चलाया जाता है, तब तक इसके साथ संबंधित सभी फ़ाइलें और प्रक्रियाएं निम्न अखंडता स्तर के साथ संबंधित की जाती हैं। कुछ फ़ोल्डर, जैसे कि **टेम्पररी इंटरनेट फ़ोल्डर**, डिफ़ॉल्ट रूप से निम्न अखंडता स्तर के साथ निर्धारित किए जाते हैं। हालांकि, ध्यान दें कि **निम्न अखंडता प्रक्रिया** बहुत **प्रतिबंधित** होती है, यह **रजिस्ट्री** में **लिखने के लिए नहीं** हो सकती है और यह **वर्तमान उपयोगकर्ता के प्रोफ़ाइल के अधिकांश स्थानों** में से लिखने से प्रतिबंधित होती है। _उदाहरण: Internet Explorer या Microsoft Edge_
* **मध्यम** - मध्यम वह संदर्भ है जिसमें **अधिकांश ऑब्जेक्ट चलेंगे**। मानक उपयोगकर्ताएं मध्यम अखंडता स्तर प्राप्त करती हैं, और कोई भी ऑब्जेक्ट जो निर्दिष्ट रूप से निम्न या उच्च अखंडता स्तर के साथ निर्धारित नहीं है, डिफ़ॉल्ट रूप से मध्यम होता है। ध्यान दें कि एक प्रशासक समूह के भीतर एक उपयोगकर्ता डिफ़ॉल्ट रूप से मध्यम अखंडता स्तर का उपयोग करेगा।
* **उच्च** - **प्रशासकों** को उच्च अखंडता स्तर प्रदान किया जाता है। इससे सुनिश्चित किया जाता है कि प्रशासक मध्यम या निम्न अखंडता स्त
```
echo asd >asd.txt
icacls asd.txt
asd.txt BUILTIN\Administrators:(I)(F)
DESKTOP-IDJHTKP\user:(I)(F)
NT AUTHORITY\SYSTEM:(I)(F)
NT AUTHORITY\INTERACTIVE:(I)(M,DC)
NT AUTHORITY\SERVICE:(I)(M,DC)
NT AUTHORITY\BATCH:(I)(M,DC)
```
अब, चलो फ़ाइल को **उच्च** अखंडता स्तर के लिए न्यूनतम अखंडता स्तर निर्धारित करें। यह **एक कंसोल से किया जाना चाहिए** जो **व्यवस्थापक** के रूप में चल रहा हो, क्योंकि **सामान्य कंसोल** माध्यम अखंडता स्तर में चल रहा होगा और एक ऑब्जेक्ट को उच्च अखंडता स्तर निर्धारित करने की अनुमति नहीं होगी:
```
icacls asd.txt /setintegritylevel(oi)(ci) High
processed file: asd.txt
Successfully processed 1 files; Failed processing 0 files

C:\Users\Public>icacls asd.txt
asd.txt BUILTIN\Administrators:(I)(F)
DESKTOP-IDJHTKP\user:(I)(F)
NT AUTHORITY\SYSTEM:(I)(F)
NT AUTHORITY\INTERACTIVE:(I)(M,DC)
NT AUTHORITY\SERVICE:(I)(M,DC)
NT AUTHORITY\BATCH:(I)(M,DC)
Mandatory Label\High Mandatory Level:(NW)
```
यहाँ चीजें दिलचस्प होती हैं। आप देख सकते हैं कि उपयोगकर्ता `DESKTOP-IDJHTKP\user` को फ़ाइल पर **पूर्ण अधिकार** हैं (वास्तव में यही उपयोगकर्ता ने फ़ाइल बनाई थी), हालांकि, न्यूनतम अखंडता स्तर के कारण उसे फ़ाइल को संशोधित करने की अनुमति नहीं होगी यदि वह उच्च अखंडता स्तर में नहीं चल रहा है (ध्यान दें कि वह इसे पढ़ सकेगा):
```
echo 1234 > asd.txt
Access is denied.

del asd.txt
C:\Users\Public\asd.txt
Access is denied.
```
{% hint style="info" %}
**इसलिए, जब एक फ़ाइल का न्यूनतम अखंडता स्तर होता है, तो उसे संशोधित करने के लिए आपको कम से कम उस अखंडता स्तर में चल रहे होना चाहिए।**
{% endhint %}

## बाइनरी में अखंडता स्तर

मैंने `cmd.exe` की एक प्रतिलिपि `C:\Windows\System32\cmd-low.exe` बनाई और इसे **एक प्रशासक कंसोल से निम्नतम अखंडता स्तर के रूप में सेट किया है:**
```
icacls C:\Windows\System32\cmd-low.exe
C:\Windows\System32\cmd-low.exe NT AUTHORITY\SYSTEM:(I)(F)
BUILTIN\Administrators:(I)(F)
BUILTIN\Users:(I)(RX)
APPLICATION PACKAGE AUTHORITY\ALL APPLICATION PACKAGES:(I)(RX)
APPLICATION PACKAGE AUTHORITY\ALL RESTRICTED APP PACKAGES:(I)(RX)
Mandatory Label\Low Mandatory Level:(NW)
```
अब, जब मैं `cmd-low.exe` चलाता हूँ, तो यह **कम-अखंडता स्तर** के तहत चलेगा बजाय मध्यम अखंडता स्तर के:

![](<../../.gitbook/assets/image (320).png>)

जिज्ञासु लोगों के लिए, यदि आप किसी बाइनरी को उच्च अखंडता स्तर देते हैं (`icacls C:\Windows\System32\cmd-high.exe /setintegritylevel high`), तो यह उच्च अखंडता स्तर के साथ स्वचालित रूप से नहीं चलेगा (यदि आप इसे मध्यम अखंडता स्तर से आह्वान करते हैं -डिफ़ॉल्ट रूप से-, तो यह मध्यम अखंडता स्तर के तहत चलेगा)।

## प्रक्रियाओं में अखंडता स्तर

सभी फ़ाइलें और फ़ोल्डरों के पास एक न्यूनतम अखंडता स्तर नहीं होता है, **लेकिन सभी प्रक्रियाएं एक अखंडता स्तर के तहत चल रही होती हैं**। और फ़ाइल-सिस्टम के साथ जो हुआ था, **यदि कोई प्रक्रिया किसी अन्य प्रक्रिया में लिखना चाहती है, तो उसके पास कम से कम वही अखंडता स्तर होना चाहिए**। इसका मतलब है कि कम अखंडता स्तर वाली प्रक्रिया मध्यम अखंडता स्तर वाली प्रक्रिया के साथ पूर्ण पहुंच वाला हैंडल नहीं खोल सकती है।

इस और पिछले खंड में बताए गए प्रतिबंधों के कारण, सुरक्षा के दृष्टिकोण से हमेशा **संभावित सबसे कम अखंडता स्तर में प्रक्रिया चलाना सिफारिश किया जाता है**।


<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने की सुविधा** चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह।

- प्राप्त करें [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर सबमिट करके साझा करें**।

</details>
