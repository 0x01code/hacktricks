# स्टेगो ट्रिक्स

<details>

<summary><strong>शून्य से नायक तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS रेड टीम एक्सपर्ट)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) को **फॉलो करें**.
* **HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपोज़ में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें.

</details>

<figure><img src="../.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

सबसे महत्वपूर्ण कमजोरियों को खोजें ताकि आप उन्हें तेजी से ठीक कर सकें। Intruder आपकी अटैक सरफेस को ट्रैक करता है, प्रोएक्टिव थ्रेट स्कैन चलाता है, और आपके पूरे टेक स्टैक में, APIs से लेकर वेब ऐप्स और क्लाउड सिस्टम्स तक, मुद्दों को खोजता है। [**आज ही मुफ्त में आजमाएं**](https://www.intruder.io/?utm\_source=referral\&utm\_campaign=hacktricks).

{% embed url="https://www.intruder.io/?utm_campaign=hacktricks&utm_source=referral" %}

***

## सभी फाइलों से डेटा निकालना

### Binwalk <a href="#binwalk" id="binwalk"></a>

Binwalk एक टूल है जो बाइनरी फाइलों, जैसे कि इमेजेस और ऑडियो फाइलों, में छिपी हुई फाइलों और डेटा की खोज करता है।\
इसे `apt` के साथ इंस्टॉल किया जा सकता है, और [सोर्स](https://github.com/ReFirmLabs/binwalk) Github पर मिल सकता है।\
**उपयोगी कमांड्स**:\
`binwalk file` : दिए गए फाइल में एम्बेडेड डेटा दिखाता है\
`binwalk -e file` : दिए गए फाइल से डेटा दिखाता है और निकालता है\
`binwalk --dd ".*" file` : दिए गए फाइल से डेटा दिखाता है और निकालता है

### Foremost <a href="#foremost" id="foremost"></a>

Foremost एक प्रोग्राम है जो उनके हेडर्स, फुटर्स, और आंतरिक डेटा संरचनाओं के आधार पर फाइलों को पुनः प्राप्त करता है। मुझे यह png इमेजेस के साथ डील करते समय विशेष रूप से उपयोगी लगता है। आप **/etc/foremost.conf** में कॉन्फिग फाइल बदलकर Foremost द्वारा निकाली जाने वाली फाइलों का चयन कर सकते हैं।\
इसे `apt` के साथ इंस्टॉल किया जा सकता है, और [सोर्स](https://github.com/korczis/foremost) Github पर मिल सकता है।\
**उपयोगी कमांड्स:**\
`foremost -i file` : दिए गए फाइल से डेटा निकालता है।

### Exiftool <a href="#exiftool" id="exiftool"></a>

कभी-कभी, महत्वपूर्ण चीजें एक इमेज या फाइल के मेटाडेटा में छिपी होती हैं; exiftool फाइल मेटाडेटा देखने के लिए बहुत मददगार हो सकता है।\
आप इसे [यहाँ](https://www.sno.phy.queensu.ca/\~phil/exiftool/) से प्राप्त कर सकते हैं।\
**उपयोगी कमांड्स:**\
`exiftool file` : दिए गए फाइल का मेटाडेटा दिखाता है

### Exiv2 <a href="#exiv2" id="exiv2"></a>

exiftool के समान एक टूल।\
इसे `apt` के साथ इंस्टॉल किया जा सकता है, और [सोर्स](https://github.com/Exiv2/exiv2) Github पर मिल सकता है।\
[आधिकारिक वेबसाइट](http://www.exiv2.org/)\
**उपयोगी कमांड्स:**\
`exiv2 file` : दिए गए फाइल का मेटाडेटा दिखाता है

### File

जांचें कि आपके पास किस प्रकार की फाइल है

### Strings

फाइल से स्ट्रिंग्स निकालें।\
उपयोगी कमांड्स:\
`strings -n 6 file`: कम से कम 6 की लंबाई वाली स्ट्रिंग्स निकालें\
`strings -n 6 file | head -n 20`: पहली 20 स्ट्रिंग्स निकालें जिनकी लंबाई कम से कम 6 हो\
`strings -n 6 file | tail -n 20`: आखिरी 20 स्ट्रिंग्स निकालें जिनकी लंबाई कम से कम 6 हो\
`strings -e s -n 6 file`: 7bit स्ट्रिंग्स निकालें\
`strings -e S -n 6 file`: 8bit स्ट्रिंग्स निकालें\
`strings -e l -n 6 file`: 16bit स्ट्रिंग्स निकालें (लिटिल-एंडियन)\
`strings -e b -n 6 file`: 16bit स्ट्रिंग्स निकालें (बिग-एंडियन)\
`strings -e L -n 6 file`: 32bit स्ट्रिंग्स निकालें (लिटिल-एंडियन)\
`strings -e B -n 6 file`: 32bit स्ट्रिंग्स निकालें (बिग-एंडियन)

### cmp - तुलना

यदि आपके पास कोई **संशोधित** इमेज/ऑडियो/वीडियो है, तो जांचें कि क्या आप इंटरनेट पर **बिल्कुल मूल वाला** खोज सकते हैं, फिर दोनों फाइलों की **तुलना** करें:
```
cmp original.jpg stego.jpg -b -l
```
## पाठ में छिपे हुए डेटा को निकालना

### स्पेस में छिपा डेटा

यदि आप पाते हैं कि एक **पाठ पंक्ति** सामान्य से **बड़ी** है, तो कुछ **छिपी हुई जानकारी** **स्पेस** में अदृश्य अक्षरों का उपयोग करके शामिल की जा सकती है।󐁈󐁥󐁬󐁬󐁯󐀠󐁴󐁨\
**डेटा** को **निकालने** के लिए, आप इसका उपयोग कर सकते हैं: [https://www.irongeek.com/i.php?page=security/unicode-steganography-homoglyph-encoder](https://www.irongeek.com/i.php?page=security/unicode-steganography-homoglyph-encoder)

<figure><img src="../.gitbook/assets/image (3) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks) का उपयोग करके आसानी से और **ऑटोमेट वर्कफ्लोज़** को बनाएं जो दुनिया के **सबसे उन्नत** समुदाय टूल्स द्वारा संचालित होते हैं।\
आज ही एक्सेस प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## छवियों से डेटा निकालना

### पहचान

[GraphicMagick](https://imagemagick.org/script/download.php) टूल यह जांचने के लिए कि एक फाइल किस प्रकार की छवि है। यह यह भी जांचता है कि छवि क्षतिग्रस्त तो नहीं है।
```
./magick identify -verbose stego.jpg
```
यदि छवि क्षतिग्रस्त है, तो आप इसमें मेटाडेटा टिप्पणी जोड़कर इसे पुनर्स्थापित कर सकते हैं (यदि यह बहुत अधिक क्षतिग्रस्त है तो यह काम नहीं करेगा):
```bash
./magick mogrify -set comment 'Extraneous bytes removed' stego.jpg
```
### Steghide \[JPEG, BMP, WAV, AU] <a href="#steghide" id="steghide"></a>

Steghide एक स्टेगनोग्राफी प्रोग्राम है जो विभिन्न प्रकार की इमेज और ऑडियो फाइलों में डेटा छिपाता है। यह निम्नलिखित फाइल प्रारूपों का समर्थन करता है: `JPEG, BMP, WAV और AU`। यह अन्य फाइलों से एम्बेडेड और एन्क्रिप्टेड डेटा निकालने के लिए भी उपयोगी है।\
इसे `apt` के साथ इंस्टॉल किया जा सकता है, और [स्रोत](https://github.com/StefanoDeVuono/steghide) Github पर मिल सकता है।\
**उपयोगी कमांड्स:**\
`steghide info file` : यह जानकारी दिखाता है कि क्या फाइल में एम्बेडेड डेटा है या नहीं।\
`steghide extract -sf file [--passphrase password]` : एक फाइल से एम्बेडेड डेटा निकालता है \[पासवर्ड का उपयोग करके]

आप Steghide से वेब का उपयोग करके भी सामग्री निकाल सकते हैं: [https://futureboy.us/stegano/decinput.html](https://futureboy.us/stegano/decinput.html)

**Bruteforcing** Steghide: [stegcracker](https://github.com/Paradoxis/StegCracker.git) `stegcracker <file> [<wordlist>]`

### Zsteg \[PNG, BMP] <a href="#zsteg" id="zsteg"></a>

zsteg एक टूल है जो png और bmp फाइलों में छिपे हुए डेटा का पता लगा सकता है।\
इसे इंस्टॉल करने के लिए: `gem install zsteg`। स्रोत भी [Github](https://github.com/zed-0xff/zsteg) पर मिल सकता है।\
**उपयोगी कमांड्स:**\
`zsteg -a file` : दिए गए फाइल पर हर डिटेक्शन मेथड चलाता है।\
`zsteg -E file` : दिए गए पेलोड के साथ डेटा निकालता है (उदाहरण: zsteg -E b4,bgr,msb,xy name.png)

### stegoVeritas JPG, PNG, GIF, TIFF, BMP

सरल और उन्नत ट्रिक्स की एक विस्तृत विविधता के लिए सक्षम, यह टूल फाइल मेटाडेटा की जांच कर सकता है, ट्रांसफॉर्म्ड इमेजेज बना सकता है, LSB को ब्रूट फोर्स कर सकता है, और अधिक। `stegoveritas.py -h` पर जाकर इसकी पूरी क्षमताओं के बारे में पढ़ें। सभी चेक्स चलाने के लिए `stegoveritas.py stego.jpg` को निष्पादित करें।

### Stegsolve

कभी-कभी इमेज में ही एक संदेश या टेक्स्ट छिपा होता है जिसे देखने के लिए, रंग फिल्टर्स लगाने की आवश्यकता होती है, या कुछ रंग स्तरों को बदलना पड़ता है। हालांकि आप इसे GIMP या Photoshop जैसी चीज़ों के साथ कर सकते हैं, Stegsolve इसे आसान बनाता है। यह एक छोटा जावा टूल है जो इमेजेज पर कई उपयोगी रंग फिल्टर्स लगाता है; CTF चैलेंजेज में, Stegsolve अक्सर एक वास्तविक समय बचाने वाला होता है।\
आप इसे [Github](https://github.com/eugenekolo/sec-tools/tree/master/stego/stegsolve/stegsolve) से प्राप्त कर सकते हैं।\
इसे उपयोग करने के लिए, बस इमेज खोलें और `<` `>` बटनों पर क्लिक करें।

### FFT

Fast Fourier T का उपयोग करके छिपी हुई सामग्री खोजने के लिए:

* [http://bigwww.epfl.ch/demo/ip/demos/FFT/](http://bigwww.epfl.ch/demo/ip/demos/FFT/)
* [https://www.ejectamenta.com/Fourifier-fullscreen/](https://www.ejectamenta.com/Fourifier-fullscreen/)
* [https://github.com/0xcomposure/FFTStegPic](https://github.com/0xcomposure/FFTStegPic)
* `pip3 install opencv-python`

### Stegpy \[PNG, BMP, GIF, WebP, WAV]

इमेज और ऑडियो फाइलों में स्टेगनोग्राफी के माध्यम से जानकारी एन्कोड करने के लिए एक प्रोग्राम। यह डेटा को प्लेनटेक्स्ट या एन्क्रिप्टेड के रूप में स्टोर कर सकता है।\
इसे [Github](https://github.com/dhsdshdhk/stegpy) पर खोजें।

### Pngcheck

PNG फाइल के बारे में विवरण प्राप्त करें (या यहां तक कि पता लगाएं कि यह वास्तव में कुछ और है!)।\
`apt-get install pngcheck`: टूल इंस्टॉल करें\
`pngcheck stego.png` : PNG के बारे में जानकारी प्राप्त करें

### कुछ अन्य इमेज टूल्स जिनका उल्लेख करना उचित है

* [http://magiceye.ecksdee.co.uk/](http://magiceye.ecksdee.co.uk/)
* [https://29a.ch/sandbox/2012/imageerrorlevelanalysis/](https://29a.ch/sandbox/2012/imageerrorlevelanalysis/)
* [https://github.com/resurrecting-open-source-projects/outguess](https://github.com/resurrecting-open-source-projects/outguess)
* [https://www.openstego.com/](https://www.openstego.com/)
* [https://diit.sourceforge.net/](https://diit.sourceforge.net/)

## ऑडियो से डेटा निकालना

### [Steghide \[JPEG, BMP, WAV, AU\]](stego-tricks.md#steghide) <a href="#steghide" id="steghide"></a>

### [Stegpy \[PNG, BMP, GIF, WebP, WAV\]](stego-tricks.md#stegpy-png-bmp-gif-webp-wav)

### ffmpeg

ffmpeg का उपयोग ऑडियो फाइलों की अखंडता की जांच करने, फाइल के बारे में विभिन्न जानकारी रिपोर्ट करने, साथ ही किसी भी त्रुटियों का पता लगाने के लिए किया जा सकता है।\
`ffmpeg -v info -i stego.mp3 -f null -`

### Wavsteg \[WAV] <a href="#wavsteg" id="wavsteg"></a>

WavSteg एक Python3 टूल है जो wav फाइलों में डेटा छिपा सकता है, least significant bit का उपयोग करके। यह wav फाइलों से डेटा खोजने और निकालने के लिए भी काम कर सकता है।\
आप इसे [Github](https://github.com/ragibson/Steganography#WavSteg) से प्राप्त कर सकते हैं।\
उपयोगी कमांड्स:\
`python3 WavSteg.py -r -b 1 -s soundfile -o outputfile` : एक आउटपुट फाइल में निकालता है (केवल 1 lsb लेते हुए)\
`python3 WavSteg.py -r -b 2 -s soundfile -o outputfile` : एक आउटपुट फाइल में निकालता है (केवल 2 lsb लेते हुए)

### Deepsound

AES-265 के साथ एन्क्रिप्टेड जानकारी को साउंड फाइलों में छिपाएं, और जांचें। [आधिकारिक पृष्ठ](http://jpinsoft.net/deepsound/download.aspx) से डाउनलोड करें।\
छिपी हुई जानकारी की खोज के लिए, बस प्रोग्राम चलाएं और साउंड फाइल खोलें। यदि DeepSound कोई भी छिपा हुआ डेटा पाता है, तो आपको इसे अनलॉक करने के लिए पासवर्ड प्रदान करना होगा।

### Sonic visualizer <a href="#sonic-visualizer" id="sonic-visualizer"></a>

Sonic visualizer एक टूल है जो ऑडियो फाइलों की सामग्री को देखने और विश्लेषण करने के लिए है। जब ऑडियो स्टेगनोग्राफी चैलेंजेज का सामना करते हैं, तो यह बहुत मददगार हो सकता है; आप ऑडियो फाइलों में छिपे हुए आकारों को प्रकट कर सकते हैं जो कई अन्य टूल्स पता नहीं लगा पाएंगे।\
यदि आप फंस गए हैं, तो हमेशा ऑडियो का स्पेक्ट्रोग्राम चेक करें। [आधिकारिक वेबसाइट](https://www.sonicvisualiser.org/)

### DTMF Tones - Dial tones

* [https://unframework.github.io/dtmf-detect/](https://unframework.github.io/dtmf-detect/)
* [http://dialabc.com/sound/detect/index.html](http://dialabc.com/sound/detect/index.html)

## अन्य ट्रिक्स

### Binary length SQRT - QR Code

यदि आपको बाइनरी डेटा प्राप्त होता है जिसकी SQRT लंबाई पूरी संख्या की होती है, तो यह किसी प्रकार का QR कोड हो सकता है:
```
import math
math.sqrt(2500) #50
```
बाइनरी "1" और "0" को सही इमेज में बदलने के लिए: [https://www.dcode.fr/binary-image](https://github.com/carlospolop/hacktricks/tree/32fa51552498a17d266ff03e62dfd1e2a61dcd10/binary-image/README.md)\
QR कोड पढ़ने के लिए: [https://online-barcode-reader.inliteresearch.com/](https://online-barcode-reader.inliteresearch.com/)

### ब्रेल

[https://www.branah.com/braille-translator](https://www.branah.com/braille-translator\))

## **संदर्भ**

* [**https://0xrick.github.io/lists/stego/**](https://0xrick.github.io/lists/stego/)
* [**https://github.com/DominicBreuker/stego-toolkit**](https://github.com/DominicBreuker/stego-toolkit)

<figure><img src="../.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

सबसे महत्वपूर्ण vulnerabilities को खोजें ताकि आप उन्हें तेजी से ठीक कर सकें। Intruder आपके attack surface को ट्रैक करता है, proactive threat scans चलाता है, और आपके पूरे tech stack में issues ढूंढता है, APIs से लेकर web apps और cloud systems तक। [**इसे मुफ्त में आजमाएं**](https://www.intruder.io/?utm\_source=referral\&utm\_campaign=hacktricks) आज ही।

{% embed url="https://www.intruder.io/?utm_campaign=hacktricks&utm_source=referral" %}

<details>

<summary><strong>Learn AWS hacking from zero to hero with</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) देखें!
* [**official PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा exclusive [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) को **फॉलो** करें।
* **अपने hacking tricks को साझा करें PRs जमा करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>
