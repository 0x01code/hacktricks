# Docker सुरक्षा

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में** देखना चाहते हैं या **HackTricks को PDF में डाउनलोड** करना चाहते हैं तो [**सब्सक्रिप्शन प्लान**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** द्वारा PRs सबमिट करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करें और आसानी से **वर्कफ़्लो** बनाएं और **स्वत: कार्यक्रमित** करें जो दुनिया के **सबसे उन्नत** समुदाय उपकरणों द्वारा संचालित है।\
आज ही पहुंचें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## **मूल Docker Engine सुरक्षा**

**Docker engine** ने लिनक्स कर्नेल के **नेमस्पेस** और **सीग्रुप्स** का उपयोग करके कंटेनरों को अलग करने के लिए उपयोग किया है, जो एक मूल सुरक्षा स्तर प्रदान करता है। **क्षमताएँ छोड़ना**, **Seccomp**, और **SELinux/AppArmor** के माध्यम से अतिरिक्त सुरक्षा प्रदान की जाती है, जो कंटेनर को अलग करती है। एक **ऑथ प्लगइन** उपयोगकर्ता क्रियाओं को और अधिक प्रतिबंधित कर सकता है।

![Docker सुरक्षा](https://sreeninet.files.wordpress.com/2016/03/dockersec1.png)

### Docker Engine के लिए सुरक्षित पहुंच

Docker engine को या तो स्थानीय रूप से एक यूनिक्स सॉकेट के माध्यम से या HTTP का उपयोग करके दूरस्थ रूप से पहुंचा जा सकता है। दूरस्थ पहुंच के लिए, गोपनीयता, अखंडता, और प्रमाणीकरण सुनिश्चित करने के लिए HTTPS और **TLS** का उपयोग करना महत्वपूर्ण है।

डॉकर इंजन, डिफ़ॉल्ट रूप से, `unix:///var/run/docker.sock` पर यूनिक्स सॉकेट पर सुनता है। उबुंटू सिस्टम पर, डॉकर की स्टार्टअप विकल्प `/etc/default/docker` में परिभाषित हैं। डॉकर API और क्लाइंट के लिए दूरस्थ पहुंच सक्षम करने के लिए, निम्नलिखित सेटिंग्स जोड़कर डॉकर डेमन को एक HTTP सॉकेट पर व्यक्त करें:
```bash
DOCKER_OPTS="-D -H unix:///var/run/docker.sock -H tcp://192.168.56.101:2376"
sudo service docker restart
```
हालांकि, Docker डेमन को HTTP के माध्यम से अनुसारित करना सुरक्षा संबंधी चिंताओं के कारण सिफारिश नहीं है। HTTPS का उपयोग करके कनेक्शन को सुरक्षित बनाना उत्तम है। कनेक्शन को सुरक्षित बनाने के दो मुख्य दृष्टिकोण हैं:

1. क्लाइंट सर्वर की पहचान सत्यापित करता है।
2. क्लाइंट और सर्वर एक-दूसरे की पहचान स्वीकृति करते हैं।

प्रमाणपत्रों का उपयोग सर्वर की पहचान सत्यापित करने के लिए किया जाता है। दोनों विधियों के विस्तृत उदाहरणों के लिए, [**इस गाइड**](https://sreeninet.wordpress.com/2016/03/06/docker-security-part-3engine-access/) का संदर्भ दें।

### कंटेनर इमेज की सुरक्षा

कंटेनर इमेज को निजी या सार्वजनिक रिपॉजिटरी में संग्रहित किया जा सकता है। Docker कंटेनर इमेजों के लिए कई संग्रहण विकल्प प्रदान करता है:

* [**Docker Hub**](https://hub.docker.com): Docker से एक सार्वजनिक रजिस्ट्री सेवा।
* [**Docker Registry**](https://github.com/docker/distribution): एक ओपन-सोर्स परियोजना जो उपयोगकर्ताओं को अपनी रजिस्ट्री होस्ट करने की अनुमति देती है।
* [**Docker Trusted Registry**](https://www.docker.com/docker-trusted-registry): Docker का व्यावसायिक रजिस्ट्री प्रस्ताव, जिसमें भूमिका-आधारित उपयोगकर्ता प्रमाणीकरण और LDAP निर्देशिका सेवाओं के साथ सम्मिलित है।

### इमेज स्कैनिंग

कंटेनर में **सुरक्षा दोष** हो सकते हैं या तो आधार इमेज के कारण हो सकते हैं या आधार इमेज पर स्थापित सॉफ़्टवेयर के कारण हो सकते हैं। Docker एक परियोजना पर काम कर रहा है जिसे **नौटिलस** कहा जाता है जो कंटेनरों की सुरक्षा स्कैन करता है और सुरक्षा दोषों की सूची बनाता है। नौटिलस यह काम करता है जिसमें हर कंटेनर इमेज लेयर को सुरक्षा दोष रिपॉजिटरी के साथ तुलना करके सुरक्षा खोजता है।

अधिक [**जानकारी के लिए यह पढ़ें**](https://docs.docker.com/engine/scan/).

* **`docker scan`**

**`docker scan`** कमांड आपको विदित Docker इमेजों को स्कैन करने की अनुमति देता है उपयोगकर्ता नाम या आईडी का उपयोग करके। उदाहरण के लिए, hello-world इमेज को स्कैन करने के लिए निम्नलिखित कमांड चलाएं:
```bash
docker scan hello-world

Testing hello-world...

Organization:      docker-desktop-test
Package manager:   linux
Project name:      docker-image|hello-world
Docker image:      hello-world
Licenses:          enabled

✓ Tested 0 dependencies for known issues, no vulnerable paths found.

Note that we do not currently have vulnerability data for your image.
```
* [**`trivy`**](https://github.com/aquasecurity/trivy)
```bash
trivy -q -f json <container_name>:<tag>
```
* [**`snyk`**](https://docs.snyk.io/snyk-cli/getting-started-with-the-cli)
```bash
snyk container test <image> --json-file-output=<output file> --severity-threshold=high
```
* [**`clair-scanner`**](https://github.com/arminc/clair-scanner)
```bash
clair-scanner -w example-alpine.yaml --ip YOUR_LOCAL_IP alpine:3.5
```
### Docker इमेज साइनिंग

Docker इमेज साइनिंग इस्तेमाल किए जाने वाले कंटेनर्स में इमेज की सुरक्षा और अखंडता सुनिश्चित करती है। यहाँ एक संक्षेपित स्पष्टीकरण है:

* **Docker Content Trust** नोटरी परियोजना का उपयोग करता है, जो The Update Framework (TUF) पर आधारित है, इमेज साइनिंग को प्रबंधित करने के लिए। अधिक जानकारी के लिए, [Notary](https://github.com/docker/notary) और [TUF](https://theupdateframework.github.io) देखें।
* Docker content trust को सक्रिय करने के लिए, `export DOCKER_CONTENT_TRUST=1` सेट करें। यह सुविधा Docker संस्करण 1.10 और उसके बाद के लिए डिफ़ॉल्ट रूप से बंद है।
* इस सुविधा को सक्षम करने के साथ, केवल साइन की गई इमेजेस को डाउनलोड किया जा सकता है। प्रारंभिक इमेज पुश करने के लिए, रूट और टैगिंग कुंजी के लिए पासफ्रेज सेट करना आवश्यक है, जिसके साथ Docker भी उन्नत सुरक्षा के लिए Yubikey का समर्थन करता है। अधिक विवरण [यहाँ](https://blog.docker.com/2015/11/docker-content-trust-yubikey/) मिल सकते हैं।
* साइन किए गए इमेज को डाउनलोड करने का प्रयास करने पर, content trust सक्षम होने पर "No trust data for latest" त्रुटि आती है।
* पहले के बाद इमेज पुश करने के लिए, Docker इमेज को साइन करने के लिए रिपॉजिटरी कुंजी के पासफ्रेज की मांग करता है।

अपनी निजी कुंजी की बैकअप करने के लिए, निम्नलिखित कमांड का उपयोग करें:
```bash
tar -zcvf private_keys_backup.tar.gz ~/.docker/trust/private
```
Docker होस्ट बदलते समय, संचालन बनाए रखने के लिए मूल और रिपॉजिटरी कुंजी को स्थानांतरित करना आवश्यक है।

***

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks) का उपयोग करें और आसानी से विश्व के सबसे उन्नत समुदाय उपकरणों द्वारा संचालित **कार्यप्रवाह** निर्मित करें।\
आज ही पहुंचें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## कंटेनर सुरक्षा सुविधाएं

<details>

<summary>कंटेनर सुरक्षा सुविधाओं का सारांश</summary>

#### मुख्य प्रक्रिया विलयन सुविधाएं

कंटेनरीकृत वातावरणों में परियोजनाओं और उनकी प्रक्रियाओं को अलग करना सुरक्षा और संसाधन प्रबंधन के लिए महत्वपूर्ण है। यहां मुख्य अवधारणाओं का एक सरलीकृत व्याख्यान है:

**नेमस्पेस**

* **उद्देश्य**: प्रक्रियाओं, नेटवर्क, और फ़ाइल सिस्टम जैसे संसाधनों का अलगाव सुनिश्चित करना। विशेष रूप से Docker में, नेमस्पेस कंटेनर की प्रक्रियाओं को मेज़बान और अन्य कंटेनरों से अलग रखते हैं।
* **`unshare` का उपयोग**: नए नेमस्पेस बनाने के लिए `unshare` कमांड (या अंतर्निहित सिसकॉल) का उपयोग किया जाता है, जो एक अतिरिक्त अलगाव की परत प्रदान करता है। हालांकि, Kubernetes इसे स्वाभाविक रूप से नहीं ब्लॉक करता है, Docker ब्लॉक करता है।
* **सीमा**: नए नेमस्पेस बनाने से किसी प्रक्रिया को मेज़बान की डिफ़ॉल्ट नेमस्पेस पर वापस नहीं लौटने देता है। मेज़बान नेमस्पेस में प्रवेश के लिए, आमतौर पर, `nsenter` का उपयोग करके मेज़बान के `/proc` निर्देशिका तक पहुंच की आवश्यकता होती है।

**नियंत्रण समूह (CGroups)**

* **कार्य**: प्रक्रियाओं के बीच संसाधन आवंटित करने के लिए प्राथमिक रूप से उपयोग किया जाता है।
* **सुरक्षा पहलू**: CGroups स्वयं अलगाव सुरक्षा नहीं प्रदान करते हैं, केवल `release_agent` सुविधा, जो अगर गलती से कॉन्फ़िगर की जाए, अनधिकृत पहुंच के लिए उत्पादित की जा सकती है।

**क्षमता छोड़ें**

* **महत्व**: प्रक्रिया विलयन के लिए एक महत्वपूर्ण सुरक्षा सुविधा है।
* **कार्यक्षमता**: यह एक महत्वपूर्ण सुरक्षा सुविधा है जो किसी रूट प्रक्रिया को कुछ विशेष क्षमताओं को छोड़कर करने की सीमा लगाती है। यदि किसी प्रक्रिया के पास रूट विशेषाधिकार होते हुए भी आवश्यक क्षमताएँ नहीं होती हैं, तो यह उसे विशेषाधिकारित क्रियाएँ करने से रोकता है, क्योंकि अपर्याप्त अनुमति के कारण सिसकॉल विफल हो जाएंगे।

ये हैं प्रक्रिया जो अन्य क्षमताएँ छोड़ने के बाद बची हैं:
```
Current: cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap=ep
```
{% endcode %}

**Seccomp**

यह डॉकर में डिफ़ॉल्ट रूप से सक्यूर है। यह प्रक्रिया को कॉल करने की सिसकॉल्स को और भी सीमित करने में मदद करता है।\
**डिफ़ॉल्ट डॉकर Seccomp प्रोफ़ाइल** यहाँ मिल सकता है: [https://github.com/moby/moby/blob/master/profiles/seccomp/default.json](https://github.com/moby/moby/blob/master/profiles/seccomp/default.json)

**AppArmor**

डॉकर में एक टेम्पलेट है जिसे आप सक्षम कर सकते हैं: [https://github.com/moby/moby/tree/master/profiles/apparmor](https://github.com/moby/moby/tree/master/profiles/apparmor)

यह सुरक्षा बढ़ाने की अनुमति देगा, सिस्टम कॉल्स, फ़ाइल और फ़ोल्डर्स तक पहुँच को कम करने के लिए...

</details>

### नेमस्पेस

**नेमस्पेस** लिनक्स कर्नेल की एक विशेषता है जो कर्नेल संसाधनों को विभाजित करती है ताकि एक सेट की **प्रक्रियाएँ** एक सेट के **संसाधनों** को **देखें** जबकि **एक** सेट की **प्रक्रियाएँ** एक **विभिन्न** सेट के संसाधनों को देखें। यह सुविधा एक सेट के संसाधनों और प्रक्रियाओं के लिए एक ही नेमस्पेस होने के बावजूद, वे नेमस्पेस विभिन्न संसाधनों को संदर्भित करते हैं। संसाधन एकाधिक स्थानों में मौजूद हो सकते हैं।

डॉकर निम्नलिखित लिनक्स कर्नेल नेमस्पेस का उपयोग करता है ताकि कंटेनर विलयन को प्राप्त कर सके:

* pid नेमस्पेस
* माउंट नेमस्पेस
* नेटवर्क नेमस्पेस
* ipc नेमस्पेस
* UTS नेमस्पेस

**नेमस्पेस के बारे में अधिक जानकारी** के लिए निम्नलिखित पृष्ठ की जाँच करें:

{% content-ref url="namespaces/" %}
[namespaces](namespaces/)
{% endcontent-ref %}

### cgroups

लिनक्स कर्नेल सुविधा **cgroups** प्रक्रियाओं के सेट के बीच संसाधनों जैसे cpu, मेमोरी, आईओ, नेटवर्क बैंडविड्थ को **प्रतिबंधित करने की क्षमता प्रदान करती है**। डॉकर cgroup सुविधा का उपयोग करने देता है जिससे विशिष्ट कंटेनर के लिए संसाधन नियंत्रण संभव होता है।\
निम्नलिखित एक कंटेनर उपयोगकर्ता स्थान मेमोरी को 500m तक सीमित करने के साथ बनाया गया है, कर्नेल मेमोरी को 50m तक सीमित करने के साथ, cpu शेयर को 512, blkioweight को 400 करने के साथ। सीपीयू शेयर एक अनुपात है जो कंटेनर के सीपीयू उपयोग को नियंत्रित करता है। इसका डिफ़ॉल्ट मूल्य 1024 है और 0 और 1024 के बीच रेंज है। यदि तीन कंटेनर्स का सीपीयू शेयर 1024 है, तो प्रतिस्पर्धी सीपीयू संसाधन की स्थिति में प्रत्येक कंटेनर 33% तक का सीपीयू ले सकता है। ब्ल्किओ-वेट एक अनुपात है जो कंटेनर के आईओ को नियंत्रित करता है। इसका डिफ़ॉल्ट मूल्य 500 है और 10 और 1000 के बीच रेंज है।
```
docker run -it -m 500M --kernel-memory 50M --cpu-shares 512 --blkio-weight 400 --name ubuntu1 ubuntu bash
```
किसी कंटेनर का cgroup प्राप्त करने के लिए आप निम्नलिखित कर सकते हैं:
```bash
docker run -dt --rm denial sleep 1234 #Run a large sleep inside a Debian container
ps -ef | grep 1234 #Get info about the sleep process
ls -l /proc/<PID>/ns #Get the Group and the namespaces (some may be uniq to the hosts and some may be shred with it)
```
For more information check:

{% content-ref url="cgroups.md" %}
[cgroups.md](cgroups.md)
{% endcontent-ref %}

### Capabilities

क्षमताएँ **रूट उपयोगकर्ता के लिए जो क्षमताएँ अनुमत हो सकती हैं, उनका और अधिक नियंत्रण प्रदान करती हैं**। Docker लिनक्स कर्नेल क्षमता सुविधा का उपयोग करता है **एक कंटेनर के भीतर किए जा सकने वाले ऑपरेशनों को सीमित करने** के लिए यहाँ तक कि उपयोगकर्ता के प्रकार के बावजूद।

जब एक डॉकर कंटेनर चलाया जाता है, **प्रक्रिया उसे बहिष्कार से बाहर निकलने के लिए प्रयोग कर सकती थी संवेदनशील क्षमताएँ छोड़ देती हैं**। यह सुनिश्चित करने का प्रयास करता है कि प्रक्रिया संवेदनशील कार्रवाई करने और बाहर निकलने के लिए सक्षम न हो:

{% content-ref url="../linux-capabilities.md" %}
[linux-capabilities.md](../linux-capabilities.md)
{% endcontent-ref %}

### Docker में Seccomp

यह एक सुरक्षा सुविधा है जो Docker को अंदर कंटेनर में उपयोग किए जा सकने वाले **सिसकॉल्स को सीमित करने** की अनुमति देती है:

{% content-ref url="seccomp.md" %}
[seccomp.md](seccomp.md)
{% endcontent-ref %}

### Docker में AppArmor

**AppArmor** एक कर्नेल एन्हांसमेंट है जो **कंटेनरों** को **सीमित** संसाधनों के **परियोजना प्रोफाइल** के साथ सीमित करने के लिए उपयोग किया जाता है।:

{% content-ref url="apparmor.md" %}
[apparmor.md](apparmor.md)
{% endcontent-ref %}

### Docker में SELinux

* **लेबलिंग सिस्टम**: SELinux प्रत्येक प्रक्रिया और फ़ाइल साधन को एक अद्वितीय लेबल सौंपता है।
* **नीति प्रवर्तन**: यह सुनिश्चित करता है कि क्या क्रियाएँ एक प्रक्रिया लेबल अन्य लेबलों पर प्रदर्शित कर सकती हैं जो सिस्टम के भीतर हैं।
* **कंटेनर प्रक्रिया लेबल**: जब कंटेनर इंजन कंटेनर प्रक्रियाएँ प्रारंभ करते हैं, तो उन्हें सामान्यत: एक सीमित SELinux लेबल जैसे `container_t` प्रदान किया जाता है।
* **कंटेनर के भीतर फ़ाइल लेबलिंग**: कंटेनर के भीतर फ़ाइलें आमतौर पर `container_file_t` के रूप में लेबलिंग की जाती हैं।
* **नीति नियम**: SELinux नीति मुख्य रूप से सुनिश्चित करती है कि `container_t` लेबल वाली प्रक्रियाएँ केवल उन फ़ाइलों के साथ व्यवहार (पढ़ना, लिखना, क्रियान्वित करना) कर सकती हैं जिन्हें `container_file_t` के रूप में लेबल किया गया है।

यह तंत्र सुनिश्चित करता है कि यदि कंटेनर के भीतर कोई प्रक्रिया क्षतिग्रस्त होती है, तो वह केवल उन वस्तुओं के साथ ही व्यवहार कर सकती है जिनके पास संबंधित लेबल हैं, इस प्रकार ऐसी क्षति से सीमित करते हुए।

{% content-ref url="../selinux.md" %}
[selinux.md](../selinux.md)
{% endcontent-ref %}

### AuthZ & AuthN

Docker में, एक अधिकृति प्लगइन सुरक्षा में महत्वपूर्ण भूमिका निभाता है जिसे यह निर्धारित करता है कि Docker डेमन को अनुमति देने या रोकने के लिए अनुरोध को क्या करना चाहिए। यह निर्णय दो मुख्य संदर्भों की जांच करके लिया जाता है:

* **प्रमाणीकरण संदर्भ**: इसमें उपयोगकर्ता के बारे में विस्तृत जानकारी शामिल है, जैसे कि वे कौन हैं और वे अपने आप को कैसे प्रमाणित कर रहे हैं।
* **कमांड संदर्भ**: यह अनुरोध से संबंधित सभी प्रासंगिक डेटा को समाविष्ट करता है।

ये संदर्भ सुनिश्चित करते हैं कि केवल प्रमाणित उपयोगकर्ताओं के से वैध अनुरोध प्रसंस्कृत किए जाते हैं, Docker कार्रवाइयों की सुरक्षा को बढ़ाते हुए।

{% content-ref url="authz-and-authn-docker-access-authorization-plugin.md" %}
[authz-and-authn-docker-access-authorization-plugin.md](authz-and-authn-docker-access-authorization-plugin.md)
{% endcontent-ref %}

## कंटेनर से DoS

यदि आप सही ढंग से कंटेनर के उपयोग करने वाले संसाधनों की सीमा नहीं लगाते हैं, तो एक क्षतिग्रस्त कंटेनर मेज़बान को DoS कर सकता है जहाँ वह चल रहा है।

* CPU DoS
```bash
# stress-ng
sudo apt-get install -y stress-ng && stress-ng --vm 1 --vm-bytes 1G --verify -t 5m

# While loop
docker run -d --name malicious-container -c 512 busybox sh -c 'while true; do :; done'
```
* बैंडविड्थ डीओएस
```bash
nc -lvp 4444 >/dev/null & while true; do cat /dev/urandom | nc <target IP> 4444; done
```
## दिलचस्प Docker फ्लैग

### --privileged फ्लैग

निम्नलिखित पृष्ठ पर आप सीख सकते हैं **कि `--privileged` फ्लैग का क्या मतलब है**:

{% content-ref url="docker-privileged.md" %}
[docker-privileged.md](docker-privileged.md)
{% endcontent-ref %}

### --security-opt

#### no-new-privileges

यदि आप एक कंटेनर चला रहे हैं जहां एक हमलावर को कम विशेषाधिकार वाले उपयोगकर्ता के रूप में पहुंचने में सफलता मिल जाती है। यदि आपके पास एक **गलत रूप से कॉन्फ़िगर किया गया suid बाइनरी** है, तो हमलावर इसका दुरुपयोग कर सकता है और **कंटेनर के अंदर विशेषाधिकारों को उन्नत कर सकता है**। जिससे उसे इससे बाहर निकलने की अनुमति हो सकती है।

**`no-new-privileges`** विकल्प सक्षम करके कंटेनर को चलाना इस प्रकार की विशेषाधिकारों की उन्नति को **रोकेगा**।
```
docker run -it --security-opt=no-new-privileges:true nonewpriv
```
#### अन्य
```bash
#You can manually add/drop capabilities with
--cap-add
--cap-drop

# You can manually disable seccomp in docker with
--security-opt seccomp=unconfined

# You can manually disable seccomp in docker with
--security-opt apparmor=unconfined

# You can manually disable selinux in docker with
--security-opt label:disable
```
अधिक **`--security-opt`** विकल्पों के लिए देखें: [https://docs.docker.com/engine/reference/run/#security-configuration](https://docs.docker.com/engine/reference/run/#security-configuration)

## अन्य सुरक्षा विचार

### रहस्य प्रबंधन: सर्वोत्तम प्रथाएं

डॉकर छवियों में सीधे रहस्य एम्बेड करने या पर्यावरण चरणों का उपयोग करने से बचना महत्वपूर्ण है, क्योंकि ये विधियां आपकी संवेदनशील जानकारी को `docker inspect` या `exec` जैसे कमांड के माध्यम से कंटेनर तक पहुंचने वाले किसी भी व्यक्ति को उजागर करती हैं।

**डॉकर वॉल्यूम्स** एक सुरक्षित विकल्प हैं, जो संवेदनशील जानकारी तक पहुंचने के लिए सिफारिश किया जाता है। ये एक अस्थायी फ़ाइल सिस्टम के रूप में उपयोग किए जा सकते हैं, `docker inspect` और लॉगिंग के साथ जुड़ी खतरों को कम करते हैं। हालांकि, रूट उपयोगकर्ताओं और उन लोगों तक जो कंटेनर में `exec` एक्सेस रखते हैं, रहस्यों तक पहुंच सकते हैं।

**डॉकर सीक्रेट्स** संवेदनशील जानकारी को संभालने के लिए एक और अधिक सुरक्षित विधि प्रदान करते हैं। छवियों के निर्माण चरण के दौरान रहस्यों की आवश्यकता होने पर, **BuildKit** एक कुशल समाधान प्रस्तुत करता है जो बिल्ड-टाइम रहस्यों का समर्थन करता है, बिल्ड गति को बढ़ाता है और अतिरिक्त सुविधाएँ प्रदान करता है।

BuildKit का उपयोग करने के लिए, इसे तीन तरीकों से सक्रिय किया जा सकता है:

1. एक पर्यावरण चरण के माध्यम से: `export DOCKER_BUILDKIT=1`
2. कमांडों को उपसर्ग देकर: `DOCKER_BUILDKIT=1 docker build .`
3. डॉकर कॉन्फ़िगरेशन में इसे डिफ़ॉल्ट रूप से सक्षम करके: `{ "features": { "buildkit": true } }`, इसके बाद एक डॉकर पुनरारंभ।

BuildKit बिल्ड-टाइम रहस्यों का उपयोग करने की अनुमति देता है `--secret` विकल्प के साथ, यह सुनिश्चित करता है कि ये रहस्य छवि निर्माण कैश या अंतिम छवि में शामिल नहीं हैं, जैसे कि निम्नलिखित कमांड का उपयोग करके:
```bash
docker build --secret my_key=my_value ,src=path/to/my_secret_file .
```
एक चल रहे कंटेनर में आवश्यक रहने वाले रहस्यों के लिए, **Docker Compose और Kubernetes** मजबूत समाधान प्रदान करते हैं। Docker Compose एक `secrets` कुंजी का उपयोग सेवा परिभाषा में गोपनीय फ़ाइलों को निर्दिष्ट करने के लिए करता है, जैसा कि एक `docker-compose.yml` उदाहरण में दिखाया गया है:
```yaml
version: "3.7"
services:
my_service:
image: centos:7
entrypoint: "cat /run/secrets/my_secret"
secrets:
- my_secret
secrets:
my_secret:
file: ./my_secret_file.txt
```
यह कॉन्फ़िगरेशन Docker Compose के साथ सेवाएं शुरू करते समय रहस्यों का उपयोग करने की अनुमति देता है।

कुबरनेटीज़ परिवेशों में, रहस्यों का प्राकृतिक रूप से समर्थन किया जाता है और [Helm-Secrets](https://github.com/futuresimple/helm-secrets) जैसे उपकरणों के साथ और अधिक प्रबंधित किया जा सकता है। कुबरनेटीज़ का भूमिका आधारित पहुंच नियंत्रण (RBAC) रहस्य प्रबंधन सुरक्षा को बढ़ाता है, जैसे Docker Enterprise।

### gVisor

**gVisor** एक एप्लिकेशन कर्नल है, जो गो में लिखा गया है, जो लिनक्स सिस्टम सर्फेस का एक बड़ा हिस्सा कार्यान्वित करता है। इसमें एक [ओपन कंटेनर इनीशिएटिव (OCI)](https://www.opencontainers.org) रनटाइम `runsc` शामिल है जो एप्लिकेशन और होस्ट कर्नल के बीच एक **अलगाव सीमा** प्रदान करता है। `runsc` रनटाइम Docker और Kubernetes के साथ एकीकृत होता है, जिससे सैंडबॉक्स कंटेनर्स चलाना सरल हो जाता है।

{% embed url="https://github.com/google/gvisor" %}

### Kata Containers

**Kata Containers** एक ओपन सोर्स समुदाय है जो एक सुरक्षित कंटेनर रनटाइम बनाने के लिए काम कर रहा है जिसमें हल्के वर्चुअल मशीन्स का उपयोग करके कंटेनर्स जैसा अनुभव और प्रदर्शन करते हैं, लेकिन **हार्डवेयर वर्चुअलाइज़ेशन** प्रौद्योगिकी का उपयोग करके **मजबूत कार्यभार अलगाव** प्रदान करते हैं।

{% embed url="https://katacontainers.io/" %}

### सारांश युक्तियाँ

* **`--privileged` फ्लैग का उपयोग न करें और कंटेनर के अंदर** [**डॉकर सॉकेट माउंट**](https://raesene.github.io/blog/2016/03/06/The-Dangers-Of-Docker.sock/)** न करें।** डॉकर सॉकेट कंटेनर्स को उत्पन्न करने की अनुमति देता है, इसलिए यह होस्ट पर पूरी नियंत्रण में आने का एक आसान तरीका है, उदाहरण के लिए, `--privileged` फ्लैग के साथ एक और कंटेनर चलाने के द्वारा।
* **कंटेनर के अंदर रूट के रूप में न चलाएं। एक** [**विभिन्न उपयोगकर्ता**](https://docs.docker.com/develop/develop-images/dockerfile\_best-practices/#user) **का उपयोग करें और** [**उपयोगकर्ता नेमस्पेस**](https://docs.docker.com/engine/security/userns-remap/)** का उपयोग करें।** कंटेनर में रूट होस्ट के समान है जब तक उपयोगकर्ता नेमस्पेस के साथ पुनर्मार्पित नहीं किया जाता है। यह केवल, मुख्य रूप से, लिनक्स नेमस्पेस, क्षमताएँ, और सीग्रुप्स द्वारा हल्के से प्रतिबंधित है।
* [**सभी क्षमताएँ छोड़ें**](https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities) **(`--cap-drop=all`) और केवल वही जो आवश्यक है उन्हें सक्षम करें** (`--cap-add=...`)। अधिकांश कार्यभारों को कोई क्षमताएँ नहीं चाहिए और उन्हें जोड़ने से एक संभावित हमले की दायरा बढ़ जाता है।
* [**“no-new-privileges” सुरक्षा विकल्प का उपयोग करें**](https://raesene.github.io/blog/2019/06/01/docker-capabilities-and-no-new-privs/) ताकि प्रक्रियाएं अधिक विशेषाधिकार प्राप्त न करें, उदाहरण के लिए suid बाइनरी के माध्यम से।
* [**कंटेनर के लिए उपलब्ध संसाधनों की सीमा निर्धारित करें**](https://docs.docker.com/engine/reference/run/#runtime-constraints-on-resources)**।** संसाधन सीमाएँ मशीन को डीओएस हमलों से सुरक्षित रख सकती हैं।
* **सेक्कॉम्प** [**AppArmor**](https://docs.docker.com/engine/security/apparmor/) **(या SELinux)** प्रोफाइल को संक्षेपित करने के लिए सेट करें ताकि कंटेनर के लिए उपलब्ध क्रियाएँ और सिसकॉल्स को न्यूनतम आवश्यक हों।
* **[आधिकारिक डॉकर छवियाँ](https://docs.docker.com/docker-hub/official\_images/) का उपयोग करें और हस्ताक्षरों की आवश्यकता** या उन पर आधारित अपनी छवियाँ बनाएं। [बैकडोर](https://arstechnica.com/information-technology/2018/06/backdoored-images-downloaded-5-million-times-finally-removed-from-docker-hub/) छवियों को विरासत में न लें या उपयोग न करें। इसके अलावा, रूट कुंजी, पासफ़्रेज को एक सुरक्षित स्थान में स्टोर करें। डॉकर के पास UCP के साथ कुंजियों का प्रबंधन करने की योजनाएँ हैं।
* अपनी छवियों को **नियमित रूप से पुनः निर्माण करें** ताकि होस्ट और छवियों पर सुरक्षा सुधार लागू हो सके।
* अपने **रहस्यों का बुद्धिमानी से प्रबंधन** करें ताकि हमलावर को उन तक पहुंचना कठिन हो।
* यदि आप **डॉकर डेमन को उजागर करते हैं तो HTTPS** का उपयोग करें साथ ही क्लाइंट और सर्वर प्रमाणीकरण के साथ।
* अपने Dockerfile में **ADD की बजाय COPY का उपयोग करें**। ADD स्वचालित रूप से ज़िप फ़ाइलों को निकालता है और यूआरएल से फ़ाइलें कॉपी कर सकता है। COPY के पास ये क्षमताएँ नहीं हैं। जितना संभव हो, ADD का उपयोग न करें ताकि आप दूरस्थ यूआरएल और ज़िप फ़ाइलों के माध्यम से हमलों के लिए संक्रमित न हों।
* प्रत्येक माइक्रो सेवा के लिए **अलग कंटेनर्स** होने चाहिए
* कंटेनर के अंदर **ssh न डालें**, “docker exec” का उपयोग कंटेनर में ssh करने के लिए किया जा सकता है।
* **छोटे** कंटेनर **छवियाँ रखें**
<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

दूसरे तरीके HackTricks का समर्थन करने के लिए:

* अगर आप चाहते हैं कि आपकी **कंपनी HackTricks में विज्ञापित हो** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>
