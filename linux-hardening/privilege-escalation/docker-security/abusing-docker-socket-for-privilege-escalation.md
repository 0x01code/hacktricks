# Docker सॉकेट का दुरुपयोग करके प्रिविलेज एस्कलेशन

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks क्लाउड ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 ट्विटर 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ ट्विच 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 यूट्यूब 🎥</strong></a></summary>

- क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- [**The PEASS परिवार**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT संग्रह**](https://opensea.io/collection/the-peass-family)

- [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **हैकिंग ट्रिक्स को साझा करें और PRs सबमिट करके [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में अपना योगदान दें।**

</details>

कुछ ऐसे मौके होते हैं जहां आपके पास **डॉकर सॉकेट तक पहुंच** होती है और आप इसे **प्रिविलेज एस्कलेशन** के लिए उपयोग करना चाहते हैं। कुछ कार्रवाई बहुत संदिग्ध हो सकती है और आप उन्हें टालना चाहेंगे, इसलिए यहां आपको प्रिविलेज एस्कलेशन के लिए उपयोगी होने वाले विभिन्न फ्लैग्स मिलेंगे:

### माउंट के माध्यम से

आप एक रूट के रूप में चल रहे कंटेनर में **फ़ाइल सिस्टम** के विभिन्न हिस्सों को **माउंट** कर सकते हैं और उन्हें **एक्सेस** कर सकते हैं।\
आप कंटेनर में प्रिविलेज एस्कलेशन के लिए **माउंट का दुरुपयोग कर सकते हैं**।

* **`-v /:/host`** -> कंटेनर में होस्ट फ़ाइल सिस्टम को माउंट करें ताकि आप **होस्ट फ़ाइल सिस्टम को पढ़ सकें।**
* यदि आप **होस्ट में होने की तरह महसूस करना चाहते हैं** लेकिन कंटेनर में होने के बावजूद आपके पास अन्य सुरक्षा युक्तियों को अक्षम करने के लिए झंडे का उपयोग करें:
* `--privileged`
* `--cap-add=ALL`
* `--security-opt apparmor=unconfined`
* `--security-opt seccomp=unconfined`
* `-security-opt label:disable`
* `--pid=host`
* `--userns=host`
* `--uts=host`
* `--cgroupns=host`
* \*\*`--device=/dev/sda1 --cap-add=SYS_ADMIN --security-opt apparmor=unconfined` \*\* -> यह पिछली विधि के समान है, लेकिन यहां हम **डिवाइस डिस्क को माउंट कर रहे हैं**। फिर, कंटेनर में `mount /dev/sda1 /mnt` चलाएं और आप `/mnt` में **होस्ट फ़ाइल सिस्टम तक पहुंच सकते हैं**।
* होस्ट में `fdisk -l` चलाएं और `</dev/sda1>` डिवाइस को माउंट करने के लिए खोजें
* **`-v /tmp:/host`** -> किसी कारण से आप केवल होस्ट से कुछ निर्देशिका माउंट कर सकते हैं और आपके पास होस्ट में एक्सेस होता है। इसे माउंट करें और माउंटेड निर्देशिका में **suid** के साथ **`/bin/bash`** बनाएं ताकि आप इसे होस्ट से चला सकें और रूट तक एस्केलेट कर सकें।

{% hint style="info" %}
ध्यान दें कि शायद आप `/tmp` फ़ोल्डर को माउंट नहीं कर सकते हैं लेकिन आप किसी **अलग लिखने योग्य फ़ोल्डर** को माउंट कर सक
- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**।**

- **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके साझा करें।**
