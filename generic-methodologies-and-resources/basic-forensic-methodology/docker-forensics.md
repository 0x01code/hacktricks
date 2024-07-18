# डॉकर फोरेंसिक्स

{% hint style="success" %}
AWS हैकिंग सीखें और प्रैक्टिस करें: <img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम एक्सपर्ट (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और प्रैक्टिस करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम एक्सपर्ट (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सब्सक्रिप्शन प्लान**](https://github.com/sponsors/carlospolop) की जाँच करें!
* 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में शामिल हों या हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में PR जमा करके हैकिंग ट्रिक्स साझा करें।

</details>
{% endhint %}

## कंटेनर मोडिफिकेशन

कुछ संदेह है कि किसी डॉकर कंटेनर में हमला हुआ था:
```bash
docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
cc03e43a052a        lamp-wordpress      "./run.sh"          2 minutes ago       Up 2 minutes        80/tcp              wordpress
```
आप इस कंटेनर में की गई संशोधनों को छवि के संदर्भ में आसानी से **खोज सकते हैं**:
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
पिछले कमांड में **C** का मतलब **बदल गया** है और **A,** **जोड़ा गया**।\
अगर आपको लगता है कि कोई दिलचस्प फ़ाइल जैसे `/etc/shadow` में संशोधन किया गया है तो आप इसे कंटेनर से डाउनलोड करके जांच सकते हैं कि क्या कोई हानिकारक गतिविधि है।
```bash
docker cp wordpress:/etc/shadow.
```
आप एक नए कंटेनर को चला कर इसकी तुलना **मूल फ़ाइल से कर सकते हैं** और उससे फ़ाइल निकाल सकते हैं:
```bash
docker run -d lamp-wordpress
docker cp b5d53e8b468e:/etc/shadow original_shadow #Get the file from the newly created container
diff original_shadow shadow
```
यदि आपको पता चलता है कि **कुछ संदेहपूर्ण फ़ाइल जोड़ी गई थी** तो आप कंटेनर तक पहुँच सकते हैं और इसे जांच सकते हैं:
```bash
docker exec -it wordpress bash
```
## छवियों की संशोधन

जब आपको एक निर्यातित डॉकर छवि दी जाती है (संभावित रूप में `.tar` प्रारूप में), तो आप [**container-diff**](https://github.com/GoogleContainerTools/container-diff/releases) का उपयोग कर सकते हैं **संशोधनों की सारांश** निकालने के लिए:
```bash
docker save <image> > image.tar #Export the image to a .tar file
container-diff analyze -t sizelayer image.tar
container-diff analyze -t history image.tar
container-diff analyze -t metadata image.tar
```
फिर, आप **छवि को डीकंप्रेस** कर सकते हैं और **ब्लॉब्स तक पहुंच सकते हैं** जिससे आप संशयात्मक फ़ाइलों की खोज कर सकते हैं जो आपने परिवर्तन इतिहास में पाई हो सकती हैं:
```bash
tar -xf image.tar
```
### मूल विश्लेषण

आप चल रही छवि से **मूल जानकारी** प्राप्त कर सकते हैं:
```bash
docker inspect <image>
```
आप निम्नलिखित के साथ एक सारांश **परिवर्तन का इतिहास** प्राप्त कर सकते हैं:
```bash
docker history --no-trunc <image>
```
आप एक छवि से एक **डॉकरफ़ाइल भी बना सकते हैं** इसके साथ:
```bash
alias dfimage="docker run -v /var/run/docker.sock:/var/run/docker.sock --rm alpine/dfimage"
dfimage -sV=1.36 madhuakula/k8s-goat-hidden-in-layers>
```
### डाइव

डॉकर इमेजेस में जोड़ी गई/संशोधित फ़ाइलें खोजने के लिए आप [**डाइव**](https://github.com/wagoodman/dive) (इसे [**रिलीज़**](https://github.com/wagoodman/dive/releases/tag/v0.10.0) से डाउनलोड करें) उपयोगी है:
```bash
#First you need to load the image in your docker repo
sudo docker load < image.tar                                                                                                                                                                                                         1 ⨯
Loaded image: flask:latest

#And then open it with dive:
sudo dive flask:latest
```
यह आपको **डॉकर इमेज के विभिन्न ब्लॉब के माध्यम से नेविगेट करने** और जांचने की अनुमति देता है कि कौन से फ़ाइलें संशोधित/जोड़ी गई थीं। **लाल** जोड़ी गई है और **पीला** संशोधित कर दिया गया है। **टैब** का उपयोग दूसरे दृश्य में जाने के लिए और **स्पेस** का उपयोग फ़ोल्डर को संक्षिप्त/खोलने के लिए करें।

इसके साथ आप इमेज के विभिन्न स्टेज के सामग्री तक पहुंच नहीं पाएंगे। इसे करने के लिए आपको **प्रत्येक लेयर को डीकंप्रेस करना और उस तक पहुंचना** होगा।\
आप इमेज से सभी लेयर को डीकंप्रेस कर सकते हैं जो इमेज को डीकंप्रेस किया गया था उस निर्देशिका से जहां इमेज को डीकंप्रेस किया गया था क्रियान्वित करके:
```bash
tar -xf image.tar
for d in `find * -maxdepth 0 -type d`; do cd $d; tar -xf ./layer.tar; cd ..; done
```
## मेमोरी से क्रेडेंशियल्स

ध्यान दें कि जब आप एक होस्ट के अंदर एक डॉकर कंटेनर चलाते हैं **तो आप होस्ट से कंटेनर पर चल रहे प्रोसेस देख सकते हैं** बस `ps -ef` चलाकर

इसलिए (रूट के रूप में) आप **होस्ट से प्रोसेसेस की मेमोरी डंप** कर सकते हैं और **क्रेडेंशियल्स** के लिए खोज कर सकते हैं बस [**निम्नलिखित उदाहरण की तरह**](../../linux-hardening/privilege-escalation/#process-memory)।
