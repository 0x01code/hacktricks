# Stego Tricks

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स को** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में पीआर जमा करके** अपना योगदान दें।

</details>

<figure><img src="../.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

सबसे महत्वपूर्ण संकटों को खोजें ताकि आप उन्हें तेजी से ठीक कर सकें। Intruder आपकी हमला सतह का ट्रैक करता है, प्रोएक्टिव धमकी स्कैन चलाता है, आपकी पूरी टेक स्टैक से, एपीआई से वेब ऐप्स और क्लाउड सिस्टम तक, मुद्दों को खोजता है। [**इसे मुफ़्त में आज़माएं**](https://www.intruder.io/?utm\_source=referral\&utm\_campaign=hacktricks)।

{% embed url="https://www.intruder.io/?utm_campaign=hacktricks&utm_source=referral" %}

***

## सभी फ़ाइलों से डेटा निकालना

### Binwalk <a href="#binwalk" id="binwalk"></a>

Binwalk एक टूल है जो छिपे हुए फ़ाइलों और डेटा के लिए बाइनरी फ़ाइलों, जैसे छवियाँ और ऑडियो फ़ाइलों, की खोज करने के लिए है।\
इसे `apt` के साथ स्थापित किया जा सकता है, और [स्रोत](https://github.com/ReFirmLabs/binwalk) Github पर मिल सकता है।\
**उपयोगी कमांडें**:\
`binwalk file` : दिए गए फ़ाइल में छिपा हुआ डेटा दिखाता है\
`binwalk -e file` : दिए गए फ़ाइल से डेटा दिखाता है और निकालता है\
`binwalk --dd ".*" file` : दिए गए फ़ाइल से डेटा दिखाता है और निकालता है

### Foremost <a href="#foremost" id="foremost"></a>

Foremost एक प्रोग्राम है जो हेडर, फ़ुटर और आंतरिक डेटा संरचनाओं के आधार पर फ़ाइलें बहाल करता है। मुझे यह विशेष रूप से png छवियों के साथ काम करते समय उपयोगी लगता है। आप Foremost द्वारा निकालने वाली फ़ाइलें चुन सकते हैं जिन्हें आप बदल सकते हैं **/etc/foremost.conf** में कॉन्फ़िग फ़ाइल को बदलकर।\
इसे `apt` के साथ स्थापित किया जा सकता है, और [स्रोत](https://github.com/korczis/foremost) Github पर मिल सकता है।\
**उपयोगी कमांडें:**\
`foremost -i file` : दिए गए फ़ाइल से डेटा निकालता है।

### Exiftool <a href="#exiftool" id="exiftool"></a>

कभी-कभी, महत्वपूर्ण चीजें छवि या फ़ाइल के मेटाडेटा में छिपी होती हैं; exiftool फ़ाइल मेटाडेटा देखने के लिए बहुत सहायक हो सकता है।\
आप इसे [यहां](https://www.sno.phy.queensu.ca/\~phil/exiftool/) से प्राप्त कर सकते हैं।\
**उपयोगी कमांडें:**\
`exiftool file` : दिए गए फ़ाइल के मेटाडेटा दिखाता है

### Exiv2 <a href="#exiv2" id="exiv2"></a>

exiftool के एक समान उपकरण।\
इसे `apt` के साथ स्थापित किया जा सकता है, और [स्रोत](https://github.com/Exiv2/exiv2) Github पर मिल सकता है।\
[आधिकारिक वेबसाइट](http://www.exiv2.org/)\
**उपयोगी कमांडें:**\
`exiv2 file` : दिए गए फ़ाइल के मेटाडेटा दिखाता है

### File

देखें आपके पास किस प्रकार की फ़ाइल है

### Strings

फ़ाइल से स्ट्रिंग्स निकालें।\
उपयोगी कमांडें:\
`strings -n 6 file`: 6 की न्यूनतम लंबाई वाली स्ट्रिंग्स निक
```
cmp original.jpg stego.jpg -b -l
```
## पाठ में छिपी हुई डेटा निकालना

### अंतर्निहित डेटा स्थानों में

यदि आपको लगता है कि कोई **पाठ लाइन** उससे **अधिक** बड़ा है, तो कुछ **छिपी हुई जानकारी** स्थानों में शामिल हो सकती है जहां अदृश्य वर्णों का उपयोग किया जाता है। 󐁈󐁥󐁬󐁬󐁯󐀠󐁴󐁨\
**डेटा** को **निकालने** के लिए, आप इस्तेमाल कर सकते हैं: [https://www.irongeek.com/i.php?page=security/unicode-steganography-homoglyph-encoder](https://www.irongeek.com/i.php?page=security/unicode-steganography-homoglyph-encoder)

<figure><img src="../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करें ताकि आप आसानी से बना सकें और **स्वचालित कार्यप्रवाह** बना सकें जो दुनिया के **सबसे उन्नत** समुदाय उपकरणों द्वारा संचालित होता है।\
आज ही पहुंच प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## छवियों से डेटा निकालना

### पहचान

[GraphicMagick](https://imagemagick.org/script/download.php) टूल फ़ाइल के लिए जांचने के लिए किस प्रकार की छवि है। यह भी जांचता है कि छवि क्या क्षतिग्रस्त है।
```
./magick identify -verbose stego.jpg
```
यदि छवि क्षतिग्रस्त है, तो आप उसे सीधे मेटाडेटा टिप्पणी जोड़कर उसे पुनर्स्थापित कर सकते हैं (यदि यह बहुत बुरी तरह से क्षतिग्रस्त है तो यह काम नहीं करेगा):
```bash
./magick mogrify -set comment 'Extraneous bytes removed' stego.jpg
```
### Steghide \[JPEG, BMP, WAV, AU] <a href="#steghide" id="steghide"></a>

Steghide एक स्टेगनोग्राफी प्रोग्राम है जो विभिन्न प्रकार की छवि और ऑडियो फ़ाइलों में डेटा छिपाता है। यह निम्नलिखित फ़ाइल प्रारूपों का समर्थन करता है: `JPEG, BMP, WAV और AU`। यह अन्य फ़ाइलों से एम्बेडेड और एन्क्रिप्टेड डेटा निकालने के लिए भी उपयोगी है।\
इसे `apt` के साथ स्थापित किया जा सकता है, और [स्रोत](https://github.com/StefanoDeVuono/steghide) Github पर मिल सकता है।\
**उपयोगी कमांडेज:**\
`steghide info फ़ाइल` : फ़ाइल के बारे में जानकारी प्रदर्शित करता है कि क्या फ़ाइल में एम्बेडेड डेटा है या नहीं।\
`steghide extract -sf फ़ाइल [--passphrase पासवर्ड]` : फ़ाइल से एम्बेडेड डेटा निकालता है \[पासवर्ड का उपयोग करके]

आप स्टेगहाइड से सामग्री निकालने के लिए वेब का उपयोग भी कर सकते हैं: [https://futureboy.us/stegano/decinput.html](https://futureboy.us/stegano/decinput.html)

**Steghide को Bruteforcing करना:** [stegcracker](https://github.com/Paradoxis/StegCracker.git) `stegcracker <फ़ाइल> [<वर्डलिस्ट>]`

### Zsteg \[PNG, BMP] <a href="#zsteg" id="zsteg"></a>

zsteg एक टूल है जो png और bmp फ़ाइलों में छिपे हुए डेटा का पता लगा सकता है।\
इसे स्थापित करने के लिए: `gem install zsteg`। स्रोत भी [Github](https://github.com/zed-0xff/zsteg) पर मिल सकता है।\
**उपयोगी कमांडेज:**\
`zsteg -a फ़ाइल` : दिए गए फ़ाइल पर हर डिटेक्शन मेथड चलाता है\
`zsteg -E फ़ाइल` : दिए गए पेलोड के साथ डेटा निकालता है (उदाहरण: zsteg -E b4,bgr,msb,xy name.png)

### stegoVeritas JPG, PNG, GIF, TIFF, BMP

इस टूल के पास एक विस्तृत और उन्नत ट्रिक का विकल्प है, यह फ़ाइल मेटाडेटा की जांच कर सकता है, परिवर्तित छवि बना सकता है, ब्रूट फ़ोर्स LSB कर सकता है, और बहुत कुछ। इसकी पूरी क्षमताओं के बारे में पढ़ने के लिए `stegoveritas.py -h` देखें। सभी जांचें चलाने के लिए `stegoveritas.py stego.jpg` को निष्पादित करें।

### Stegsolve

कभी-कभी छवि में संदेश या पाठ छिपा होता है जिसे देखने के लिए, उसे रंग फ़िल्टर लागू करना होता है, या कुछ रंग स्तर बदलने होते हैं। यदि आप इसे GIMP या Photoshop जैसी कुछ से कर सकते हैं, तो Stegsolve इसे आसान बनाता है। यह एक छोटा जावा टूल है जो छवियों पर कई उपयोगी रंग फ़िल्टर लागू करता है; CTF चुनौतियों में, Stegsolve अक्सर वास्तव में समय बचाने वाला होता है।\
आप इसे [Github](https://github.com/eugenekolo/sec-tools/tree/master/stego/stegsolve/stegsolve) से प्राप्त कर सकते हैं।\
इसका उपयोग करने के लिए, बस छवि खोलें और `<` `>` बटन पर क्लिक करें।

### FFT

छिपी हुई सामग्री ढूंढ़ने के लिए Fast Fourier T का उपयोग करें:

* [http://bigwww.epfl.ch/demo/ip/demos/FFT/](http://bigwww.epfl.ch/demo/ip/demos/FFT/)
* [https://www.ejectamenta.com/Fourifier-fullscreen/](https://www.ejectamenta.com/Fourifier-fullscreen/)
* [https://github.com/0xcomposure/FFTStegPic](https://github.com/0xcomposure/FFTStegPic)
* `pip3 install opencv-python`

### Stegpy \[PNG, BMP, GIF, WebP, WAV]

छवि और ऑडियो फ़ाइलों में जानकारी को स्टेगनोग्राफी के माध्यम से एनकोड करने के लिए एक प्रोग्राम है। यह डेटा को सादा पाठ या एन्क्रिप्टेड दोनों रूपों में संग्रहीत कर सकता है।\
इसे [Github](https://github.com/dhsdshdhk/stegpy) पर खोजें।

### Pngcheck

PNG फ़ाइल पर विवरण प्राप्त करें (या यह जानें कि यह वास्तव में कुछ और है या नहीं)।\
`apt-get install pngcheck`: टूल स्थापित करें\
`pngcheck stego.png` : PNG के बारे में जानकारी प्राप्त करें

### कुछ अन्य छवि उपकरण जिनका उल्लेख करना योग्य है

* [http://magiceye.ecksdee.co.uk/](http://magiceye.ecksdee.co.uk/)
* [https://29a.ch/sandbox/2012/imageerrorlevelanalysis/](https://29a.ch/sandbox/2012/imageerrorlevelanalysis/)
* [https://github.com/resurrecting-open-source-projects/outguess](https://github.com/resurrecting-open-source-projects/outguess)
* [https://www.openstego.com/](https://www.openstego.com/)
* [https://diit.sourceforge.net/](https://diit.sourceforge.net/)

## ऑडियो से डेटा निकालना

### [Steghide \[JPEG, BMP, WAV, AU\]](st
### DTMF टोन - डायल टोन्स

* [https://unframework.github.io/dtmf-detect/](https://unframework.github.io/dtmf-detect/)
* [http://dialabc.com/sound/detect/index.html](http://dialabc.com/sound/detect/index.html)

## अन्य ट्रिक्स

### बाइनरी लंबाई SQRT - QR कोड

यदि आपको पूरे संख्यात्मक एक एसक्यूएलटी लंबाई के साथ बाइनरी डेटा प्राप्त होता है, तो यह किसी प्रकार का QR कोड हो सकता है:
```
import math
math.sqrt(2500) #50
```
बाइनरी "1" और "0" को एक उचित छवि में रूपांतरित करने के लिए: [https://www.dcode.fr/binary-image](https://github.com/carlospolop/hacktricks/tree/32fa51552498a17d266ff03e62dfd1e2a61dcd10/binary-image/README.md)\
एक QR कोड को पढ़ने के लिए: [https://online-barcode-reader.inliteresearch.com/](https://online-barcode-reader.inliteresearch.com/)

### ब्रेल

[https://www.branah.com/braille-translator](https://www.branah.com/braille-translator\))

## **संदर्भ**

* [**https://0xrick.github.io/lists/stego/**](https://0xrick.github.io/lists/stego/)
* [**https://github.com/DominicBreuker/stego-toolkit**](https://github.com/DominicBreuker/stego-toolkit)

<figure><img src="../.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

विशेषता को खोजें जो सबसे अधिक मायने रखती हैं ताकि आप उन्हें तेजी से ठीक कर सकें। Intruder आपकी हमला सतह का ट्रैक करता है, प्रोएक्टिव धमकी स्कैन चलाता है, आपकी पूरी टेक स्टैक, एपीआई से वेब ऐप्स और क्लाउड सिस्टम तक, में समस्याएं खोजता है। [**इसे मुफ्त में आज़माएं**](https://www.intruder.io/?utm\_source=referral\&utm\_campaign=hacktricks)।

{% embed url="https://www.intruder.io/?utm_campaign=hacktricks&utm_source=referral" %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की अनुमति चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का पालन करें।**
* **अपने हैकिंग ट्रिक्स साझा करें और PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को डाउनलोड करें।**

</details>
