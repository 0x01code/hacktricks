# डॉकर फोरेंसिक्स

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ हैकट्रिक्स क्लाउड ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 ट्विटर 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ ट्विच 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 यूट्यूब 🎥</strong></a></summary>

- क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को हैकट्रिक्स में विज्ञापित करना चाहते हैं**? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)

- प्राप्त करें [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **फॉलो** करें मुझे **ट्विटर** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स को [हैकट्रिक्स रेपो](https://github.com/carlospolop/hacktricks) और [हैकट्रिक्स-क्लाउड रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके साझा करें।**

</details>

## कंटेनर संशोधन

कुछ संदेह है कि किसी डॉकर कंटेनर में हमला हुआ है:
```bash
docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
cc03e43a052a        lamp-wordpress      "./run.sh"          2 minutes ago       Up 2 minutes        80/tcp              wordpress
```
आप इस कंटेनर में की गई संशोधनों को आसानी से खोज सकते हैं जो छवि के संबंध में किए गए हैं, इसके साथ:
```bash
docker diff wordpress
C /var
C /var/lib
C /var/lib/mysql
A /var/lib/mysql/ib_logfile0
A /var/lib/mysql/ib_logfile1
A /var/lib/mysql/ibdata1
A /var/lib/mysql/mysql
A /var/lib/mysql/mysql/time_zone_leap_second.MYI
A /var/lib/mysql/mysql/general_log.CSV
...
```
पिछले कमांड में **C** का मतलब **बदला हुआ** होता है और **A,** का मतलब **जोड़ा गया** होता है।\
यदि आपको पता चलता है कि कोई दिलचस्प फ़ाइल जैसे `/etc/shadow` में संशोधन किया गया है, तो आप इसे कंटेनर से डाउनलोड करके जांच सकते हैं कि क्या इसमें कोई दुष्ट गतिविधि है:
```bash
docker cp wordpress:/etc/shadow.
```
आप इसे मूल फ़ाइल के साथ **तुलना कर सकते हैं** एक नया कंटेनर चला कर और उससे फ़ाइल निकालकर:
```bash
docker run -d lamp-wordpress
docker cp b5d53e8b468e:/etc/shadow original_shadow #Get the file from the newly created container
diff original_shadow shadow
```
यदि आपको पता चलता है कि **कुछ संदिग्ध फ़ाइल जोड़ी गई है**, तो आप कंटेनर तक पहुंच सकते हैं और इसे जांच सकते हैं:
```bash
docker exec -it wordpress bash
```
## छवि संशोधन

जब आपको एक निर्यातित डॉकर छवि (संभवतः `.tar` प्रारूप में) दी जाती है, तो आप [**container-diff**](https://github.com/GoogleContainerTools/container-diff/releases) का उपयोग करके **संशोधनों का सारांश निकाल सकते हैं**:
```bash
docker save <image> > image.tar #Export the image to a .tar file
container-diff analyze -t sizelayer image.tar
container-diff analyze -t history image.tar
container-diff analyze -t metadata image.tar
```
तब, आप इमेज को **डीकंप्रेस** कर सकते हैं और **ब्लॉब्स तक पहुंच सकते हैं** ताकि आप चेंज हिस्ट्री में पाए गए संदेहजनक फ़ाइलों की खोज कर सकें:
```bash
tar -xf image.tar
```
### मूल्यांकन का मूलभूत विश्लेषण

आप चल रही इमेज से **मूलभूत जानकारी** प्राप्त कर सकते हैं:
```bash
docker inspect <image>
```
आप इसके साथ एक संक्षेपिक **परिवर्तन का इतिहास** भी प्राप्त कर सकते हैं:
```bash
docker history --no-trunc <image>
```
आप एक छवि से भी एक **डॉकरफ़ाइल उत्पन्न कर सकते हैं** इसके साथ:
```bash
alias dfimage="docker run -v /var/run/docker.sock:/var/run/docker.sock --rm alpine/dfimage"
dfimage -sV=1.36 madhuakula/k8s-goat-hidden-in-layers>
```
### डाइव

डॉकर इमेज में जोड़ी गई / संशोधित फ़ाइलें ढूंढ़ने के लिए आप यहां उपयोग कर सकते हैं [**डाइव**](https://github.com/wagoodman/dive) (इसे [**रिलीज़**](https://github.com/wagoodman/dive/releases/tag/v0.10.0) से डाउनलोड करें) उपयोगी:
```bash
#First you need to load the image in your docker repo
sudo docker load < image.tar                                                                                                                                                                                                         1 ⨯
Loaded image: flask:latest

#And then open it with dive:
sudo dive flask:latest
```
यह आपको **डॉकर इमेज के विभिन्न ब्लॉब के माध्यम से नेविगेट करने** और जांचने की अनुमति देता है कि कौन से फ़ाइलें संशोधित/जोड़ी गई हैं। **लाल** रंग में जोड़ी गई और **पीला** रंग में संशोधित हुई हैं। दूसरे दृश्य में जाने के लिए **टैब** का उपयोग करें और फ़ोल्डर को संकुचित/खोलने के लिए **स्पेस** का उपयोग करें।

डाई के साथ आप इमेज के विभिन्न स्टेज के सामग्री तक पहुंच नहीं पा एंगे। इसके लिए आपको **प्रत्येक लेयर को डीकंप्रेस करके उस तक पहुंचने की आवश्यकता होगी**।
आप इमेज के सभी लेयर को डीकंप्रेस कर सकते हैं जो इमेज को डीकंप्रेस किया गया था उसी निर्देशिका से निर्वाचित करके निम्नलिखित को निष्पादित करके:
```bash
tar -xf image.tar
for d in `find * -maxdepth 0 -type d`; do cd $d; tar -xf ./layer.tar; cd ..; done
```
## मेमोरी से क्रेडेंशियल्स

ध्यान दें कि जब आप एक होस्ट के अंदर एक डॉकर कंटेनर चलाते हैं **तो आप होस्ट से कंटेनर पर चल रहे प्रोसेस को देख सकते हैं** बस `ps -ef` चलाकर

इसलिए (रूट के रूप में) आप **होस्ट से प्रोसेसों की मेमोरी डंप कर सकते हैं** और **क्रेडेंशियल्स** की खोज कर सकते हैं बस [**निम्नलिखित उदाहरण की तरह**](../../linux-hardening/privilege-escalation/#process-memory)।

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह।

- प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स साझा करें, [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके।**

</details>
