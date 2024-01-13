# Docker फोरेंसिक्स

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) को **फॉलो करें**.
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स शेयर करें।

</details>

## कंटेनर संशोधन

शक है कि कुछ docker कंटेनर समझौता किया गया था:
```bash
docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
cc03e43a052a        lamp-wordpress      "./run.sh"          2 minutes ago       Up 2 minutes        80/tcp              wordpress
```
आप आसानी से **इस कंटेनर में किए गए संशोधनों का पता लगा सकते हैं जो इमेज के संदर्भ में हैं** इसके साथ:
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
पिछले कमांड में **C** का मतलब **Changed** और **A,** का मतलब **Added** है।\
यदि आप पाते हैं कि `/etc/shadow` जैसी कोई रोचक फाइल में परिवर्तन किया गया है, तो आप उसे कंटेनर से डाउनलोड कर सकते हैं और दुर्भावनापूर्ण गतिविधि की जांच के लिए इसे देख सकते हैं:
```bash
docker cp wordpress:/etc/shadow.
```
आप इसकी तुलना **मूल फ़ाइल के साथ कर सकते हैं** एक नया कंटेनर चलाकर और उससे फ़ाइल निकालकर:
```bash
docker run -d lamp-wordpress
docker cp b5d53e8b468e:/etc/shadow original_shadow #Get the file from the newly created container
diff original_shadow shadow
```
यदि आप पाते हैं कि **कुछ संदिग्ध फाइल जोड़ी गई थी** तो आप कंटेनर तक पहुँच सकते हैं और इसे जांच सकते हैं:
```bash
docker exec -it wordpress bash
```
## इमेज मॉडिफिकेशन्स

जब आपको एक्सपोर्टेड डॉकर इमेज (संभवतः `.tar` फॉर्मेट में) दी जाती है, तो आप [**container-diff**](https://github.com/GoogleContainerTools/container-diff/releases) का उपयोग करके **मॉडिफिकेशन्स का सारांश निकाल सकते हैं**:
```bash
docker save <image> > image.tar #Export the image to a .tar file
container-diff analyze -t sizelayer image.tar
container-diff analyze -t history image.tar
container-diff analyze -t metadata image.tar
```
तब, आप इमेज को **decompress** कर सकते हैं और **blobs तक पहुँच** सकते हैं ताकि आप परिवर्तनों के इतिहास में पाए गए संदिग्ध फाइलों की खोज कर सकें:
```bash
tar -xf image.tar
```
### मूल विश्लेषण

आप चित्र से **मूल जानकारी** प्राप्त कर सकते हैं इसे चलाकर:
```bash
docker inspect <image>
```
आप **परिवर्तनों का संक्षिप्त इतिहास** भी प्राप्त कर सकते हैं:
```bash
docker history --no-trunc <image>
```
आप **dockerfile को एक इमेज से** इसके साथ जनरेट भी कर सकते हैं:
```bash
alias dfimage="docker run -v /var/run/docker.sock:/var/run/docker.sock --rm alpine/dfimage"
dfimage -sV=1.36 madhuakula/k8s-goat-hidden-in-layers>
```
### Dive

डॉकर इमेजेस में जोड़े गए/संशोधित फाइलों को ढूंढने के लिए आप [**dive**](https://github.com/wagoodman/dive) (इसे [**releases**](https://github.com/wagoodman/dive/releases/tag/v0.10.0) से डाउनलोड करें) उपयोगिता का भी उपयोग कर सकते हैं:
```bash
#First you need to load the image in your docker repo
sudo docker load < image.tar                                                                                                                                                                                                         1 ⨯
Loaded image: flask:latest

#And then open it with dive:
sudo dive flask:latest
```
यह आपको **डॉकर इमेजेज के विभिन्न ब्लॉब्स के माध्यम से नेविगेट करने** और जांचने की अनुमति देता है कि कौन सी फाइलें संशोधित/जोड़ी गईं। **लाल** का मतलब जोड़ा गया है और **पीला** का मतलब संशोधित है। दूसरे दृश्य में जाने के लिए **टैब** का उपयोग करें और फोल्डर्स को संकुचित/खोलने के लिए **स्पेस** का उपयोग करें।

die के साथ आप इमेज के विभिन्न चरणों की सामग्री तक पहुँचने में सक्षम नहीं होंगे। ऐसा करने के लिए आपको प्रत्येक परत को **डिकंप्रेस करना होगा और उस तक पहुँचना होगा**।\
आप इमेज की सभी परतों को उस निर्देशिका से डिकंप्रेस कर सकते हैं जहाँ इमेज को डिकंप्रेस किया गया था, निम्नलिखित कमांड निष्पादित करके:
```bash
tar -xf image.tar
for d in `find * -maxdepth 0 -type d`; do cd $d; tar -xf ./layer.tar; cd ..; done
```
## मेमोरी से क्रेडेंशियल्स

ध्यान दें कि जब आप एक होस्ट के अंदर एक docker कंटेनर चलाते हैं **तो आप होस्ट से कंटेनर पर चल रही प्रक्रियाओं को देख सकते हैं** बस `ps -ef` चलाकर।

इसलिए (रूट के रूप में) आप **प्रक्रियाओं की मेमोरी को डंप कर सकते हैं** होस्ट से और **क्रेडेंशियल्स के लिए खोज** कर सकते हैं बस [**निम्नलिखित उदाहरण की तरह**](../../linux-hardening/privilege-escalation/#process-memory)।

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें।
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह।
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) को **फॉलो करें**।
* **अपनी हैकिंग ट्रिक्स साझा करें PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>
