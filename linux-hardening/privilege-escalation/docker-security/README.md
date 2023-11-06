# डॉकर सुरक्षा

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ हैकट्रिक्स क्लाउड ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 ट्विटर 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ ट्विच 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 यूट्यूब 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को हैकट्रिक्स में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT संग्रह**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल हों** या मुझे **ट्विटर** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का** अनुसरण करें।**
* **अपने हैकिंग ट्रिक्स साझा करें, PRs के माध्यम से** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को सबमिट करके**।

</details>

<figure><img src="../../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करें और आसानी से वर्कफ़्लो बनाएं और संचालित करें, जो दुनिया के **सबसे उन्नत समुदाय उपकरणों** द्वारा संचालित होते हैं।\
आज ही पहुंच प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## **बेसिक डॉकर इंजन सुरक्षा**

डॉकर इंजन कंटेनर्स को चलाने और प्रबंधित करने का भारी काम करता है। डॉकर इंजन नेमस्पेस और सीग्रुप्स जैसे लिनक्स कर्नल के सुविधाओं का उपयोग करके कंटेनर्स के बीच मूलभूत अलगाव प्रदान करता है। यह बेहतर अलगाव प्राप्त करने के लिए कैपेबिलिटीज़ ड्रॉपिंग, सेकॉम्प, SELinux/AppArmor जैसी सुविधाओं का उपयोग करता है।

अंत में, एक **ऑथ प्लगइन** का उपयोग करके उपयोगकर्ताओं की कार्रवाई को सीमित करने के लिए किया जा सकता है।

![](<../../../.gitbook/assets/image (625) (1) (1).png>)

### **डॉकर इंजन सुरक्षित पहुंच**

डॉकर क्लाइंट डॉकर इंजन का उपयोग करके स्थानीय रूप से Unix सॉकेट या http मेकेनिज़्म का उपयोग करके कर सकता है। इसे दूरस्थता में उपयोग करने के लिए, यह आवश्यक है कि https और **TLS** का उपयोग किया जाए ताकि गोपनीयता, अखंडता और प्रमाणीकरण सुनिश्चित किया जा सके।

डिफ़ॉल्ट रूप से Unix सॉकेट `unix:///var/` पर सुनता है\
`run/docker.sock` और Ubuntu वितरणों में, डॉकर स्टार्ट विकल्प `/etc/default/docker` में निर्दिष्ट किए जाते हैं। डॉकर एपीआई और क्लाइंट को दूरस्थता में डॉकर इंजन तक पहुंचने की अनुमति देने के लिए, हमें **http सॉकेट का उपयोग करके डॉकर डेमन को उजागर करने की आवश्यकता होती है**। इसे निम्नलिखित तरीके से किया जा सकता है:
```bash
DOCKER_OPTS="-D -H unix:///var/run/docker.sock -H
tcp://192.168.56.101:2376" -> add this to /etc/default/docker
Sudo service docker restart -> Restart Docker daemon
```
एचटीटीपी का उपयोग करके डॉकर डेमन को एचटीटीपी का उपयोग करके उजागर करना अच्छी अभ्यास नहीं है और इसे एचटीटीपीएस का उपयोग करके सुरक्षित करना आवश्यक है। दो विकल्प हैं: पहला विकल्प है **क्लाइंट को सर्वर की पहचान सत्यापित करना** और दूसरा विकल्प है **क्लाइंट और सर्वर दोनों एक-दूसरे की पहचान सत्यापित करते हैं**। प्रमाणपत्र सर्वर की पहचान स्थापित करते हैं। इन दोनों विकल्पों की एक उदाहरण के लिए [**इस पृष्ठ की जांच करें**](https://sreeninet.wordpress.com/2016/03/06/docker-security-part-3engine-access/)।

### **कंटेनर इमेज सुरक्षा**

कंटेनर इमेज या तो निजी रिपॉजिटरी में संग्रहीत होती हैं या सार्वजनिक रिपॉजिटरी में। निम्नलिखित विकल्प हैं जो डॉकर प्रदान करता है कंटेनर इमेज संग्रहीत करने के लिए:

* [डॉकर हब](https://hub.docker.com) - यह डॉकर द्वारा प्रदान की जाने वाली एक सार्वजनिक रजिस्ट्री सेवा है।
* [डॉकर रजिस्ट्री](https://github.com/%20docker/distribution) - यह एक खुला स्रोत परियोजना है जिसका उपयोग उपयोगकर्ता अपनी खुद की रजिस्ट्री को होस्ट करने के लिए कर सकते हैं।
* [डॉकर विश्वसनीय रजिस्ट्री](https://www.docker.com/docker-trusted-registry) - यह डॉकर का व्यावसायिक अमलीयन है जो डॉकर रजिस्ट्री का विश्वसनीय अमलीयन है और इसमें भूमिका आधारित उपयोगकर्ता प्रमाणीकरण के साथ LDAP निर्देशिका सेवा एकीकरण भी प्रदान करता है।

### इमेज स्कैनिंग

कंटेनर में सुरक्षा संबंधी कमियां हो सकती हैं या तो बेस इमेज के कारण या बेस इमेज पर स्थापित सॉफ़्टवेयर के कारण। डॉकर एक परियोजना पर काम कर रहा है जिसका नाम है **नॉटिलस** जो कंटेनर की सुरक्षा स्कैन करता है और सुरक्षा खोजक के साथ कंटेनर इमेज की सुरक्षा कमियों की सूची बनाता है। नॉटिलस काम करता है जब वह हर कंटेनर इमेज की परत को सुरक्षा खोजक रिपॉजिटरी के साथ तुलना करके सुरक्षा खोजक को पहचानने के लिए।

अधिक जानकारी के लिए [**इसे पढ़ें**](https://docs.docker.com/engine/scan/)।

* **`docker scan`**

**`docker scan`** कमांड का उपयोग करके आप मौजूदा डॉकर इमेज को इमेज के नाम या आईडी का उपयोग करके स्कैन कर सकते हैं। उदाहरण के लिए, hello-world इमेज को स्कैन करने के लिए निम्नलिखित कमांड को चलाएं:
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
trivy -q -f json <ontainer_name>:<tag>
```
* [**`snyk`**](https://docs.snyk.io/snyk-cli/getting-started-with-the-cli)
```bash
snyk container test <image> --json-file-output=<output file> --severity-threshold=high
```
* [**`clair-scanner`**](https://github.com/arminc/clair-scanner)
```bash
clair-scanner -w example-alpine.yaml --ip YOUR_LOCAL_IP alpine:3.5
```
### डॉकर इमेज साइन करना

डॉकर कंटेनर इमेज को सार्वजनिक या निजी रजिस्ट्री में संग्रहीत किया जा सकता है। इमेज को नष्ट नहीं किया गया है यह सत्यापित करने के लिए इमेज को **साइन** करना आवश्यक होता है। सामग्री **प्रकाशक** इमेज को साइन करने और रजिस्ट्री में पुश करने का ध्यान रखता है।

निम्नलिखित हैं डॉकर सामग्री विश्वास के कुछ विवरण:

* डॉकर सामग्री विश्वास, [Notary ओपन सोर्स प्रोजेक्ट](https://github.com/docker/notary) का एक अमलन है। Notary ओपन सोर्स प्रोजेक्ट [The Update Framework (TUF) प्रोजेक्ट](https://theupdateframework.github.io) पर आधारित है।
* डॉकर सामग्री विश्वास को `export DOCKER_CONTENT_TRUST=1` के साथ सक्षम किया जाता है। डॉकर संस्करण 1.10 के रूप में, सामग्री विश्वास **डिफ़ॉल्ट रूप से सक्षम नहीं है**।
* सामग्री विश्वास सक्षम होने पर, हम केवल साइन की गई इमेजेज को ही पुल कर सकते हैं। जब इमेज पुश की जाती है, तो हमें टैगिंग कुंजी दर्ज करनी होती है।
* प्रकाशक जब डॉकर पुश का उपयोग करके इमेज को पहली बार पुश करता है, तो रूट कुंजी और टैगिंग कुंजी के लिए एक **पासवर्ड** दर्ज करने की आवश्यकता होती है। अन्य कुंजी स्वचालित रूप से उत्पन्न होती हैं।
* डॉकर ने Yubikey का उपयोग करके हार्डवेयर कुंजी के लिए भी समर्थन जोड़ा है और विवरण [यहां](https://blog.docker.com/2015/11/docker-content-trust-yubikey/) उपलब्ध हैं।

निम्नलिखित है **त्रुटि** जो हमें मिलती है जब **सामग्री विश्वास सक्षम होता है और इमेज साइन नहीं होती है**।
```shell-session
$ docker pull smakam/mybusybox
Using default tag: latest
No trust data for latest
```
निम्नलिखित आउटपुट में दिखाया गया है कि कंटेनर **इमेज को साइनिंग के साथ Docker हब पर पुश किया जा रहा है**। यह पहली बार नहीं है, इसलिए उपयोगकर्ता से केवल रिपॉजिटरी कुंजी के लिए पासफ्रेज़ दर्ज करने का अनुरोध किया जाता है।
```shell-session
$ docker push smakam/mybusybox:v2
The push refers to a repository [docker.io/smakam/mybusybox]
a7022f99b0cc: Layer already exists
5f70bf18a086: Layer already exists
9508eff2c687: Layer already exists
v2: digest: sha256:8509fa814029e1c1baf7696b36f0b273492b87f59554a33589e1bd6283557fc9 size: 2205
Signing and pushing trust metadata
Enter passphrase for repository key with ID 001986b (docker.io/smakam/mybusybox):
```
एक सुरक्षित स्थान में रूट की, रिपॉजिटरी की और पासफ्रेज को संग्रहीत करने की आवश्यकता होती है। निम्नलिखित कमांड का उपयोग निजी कुंजीयों का बैकअप लेने के लिए किया जा सकता है:
```bash
tar -zcvf private_keys_backup.tar.gz ~/.docker/trust/private
```
जब मैंने डॉकर होस्ट बदला, मुझे नए होस्ट से संचालित करने के लिए रूट कुंजी और रिपॉजिटरी कुंजी को हटाना पड़ा।

***

<figure><img src="../../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करें और आसानी से वर्कफ़्लो बनाएं और स्वचालित करें, जो दुनिया के सबसे उन्नत समुदाय उपकरणों द्वारा संचालित होते हैं।\
आज ही पहुंच प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## कंटेनर सुरक्षा सुविधाएं

<details>

<summary>कंटेनर सुरक्षा सुविधाओं का सारांश</summary>

**नेमस्पेस**

नेमस्पेस एक परियोजना को अन्य से अलग करने के लिए उपयोगी होते हैं, प्रक्रिया संचार, नेटवर्क, माउंट... को अलग करने के लिए। यह उपयोगी होता है डॉकर प्रक्रिया को अन्य प्रक्रियाओं (और यहां तक ​​कि /proc फ़ोल्डर) से अलग करने के लिए, ताकि यह अन्य प्रक्रियाओं का दुरुपयोग न कर सके।

यह संभव हो सकता है "भागना" या अधिक सटीकता से **नए नेमस्पेस बनाना** बाइनरी **`unshare`** (जो **`unshare`** सिसकॉल का उपयोग करता है) का उपयोग करके। डॉकर डिफ़ॉल्ट रूप से इसे रोकता है, लेकिन कुबरनेटीज़ नहीं करता है (इस लेखन के समय पर)।\
वैसे तो, यह नए नेमस्पेस बनाने के लिए मददगार है, लेकिन **होस्ट डिफ़ॉल्ट नेमस्पेस पर वापस जाने के लिए नहीं** (यदि आपके पास होस्ट नेमस्पेस में कुछ `/proc` तक पहुंच है, जहां आप **`nsenter`** का उपयोग करके होस्ट नेमस्पेस में प्रवेश कर सकते हैं।).

**CGroups**

इसके द्वारा संसाधनों की सीमा तय की जा सकती है और इसका प्रक्रिया के अलगाव की सुरक्षा पर प्रभाव नहीं पड़ता है (केवल `release_agent` को छोड़ने का उपयोग किया जा सकता है जिसका उपयोग भागने के लिए किया जा सकता है)।

**क्षमताएं छोड़ें**

मैं इसे प्रक्रिया अलगाव सुरक्षा के संबंध में **सबसे महत्वपूर्ण** सुविधाओं में से एक मानता हूं। इसका कारण है कि क्षमताओं के बिना, यदि प्रक्रिया रूट के रूप में चल रही है, तो आप कुछ विशेषाधिकारी कार्रवाई नहीं कर पाएंगे (क्योंकि बुलाए गए **`syscall`** को अनुमति त्रुटि वापस लौटाएगा क्योंकि प्रक्रिया के पास आवश्यक क्षमताएं नहीं हैं)।

ये हैं प्रक्रिया छोड़ने के बाद **शेष क्षमताएं**:

{% code overflow="wrap" %}
```
Current: cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap=ep
```
{% endcode %}

**Seccomp**

यह डॉकर में डिफ़ॉल्ट रूप से सक्षम है। यह मदद करता है **प्रक्रिया को और अधिक सीमित करने** के लिए सिसकॉल कोल कर सकता है।\
**डिफ़ॉल्ट डॉकर सिकॉम्प प्रोफ़ाइल** यहां मिल सकता है: [https://github.com/moby/moby/blob/master/profiles/seccomp/default.json](https://github.com/moby/moby/blob/master/profiles/seccomp/default.json)

**AppArmor**

डॉकर में एक टेम्पलेट है जिसे आप सक्रिय कर सकते हैं: [https://github.com/moby/moby/tree/master/profiles/apparmor](https://github.com/moby/moby/tree/master/profiles/apparmor)

इससे क्षमताओं, सिसकॉल, फ़ाइलों और फ़ोल्डरों तक पहुंच को कम किया जा सकता है...

</details>

### नेमस्पेस

**नेमस्पेस** लिनक्स कर्नल की एक सुविधा है जो कर्नल संसाधनों को **विभाजित करती है** ताकि एक सेट की **प्रक्रियाएं** एक सेट के **संसाधनों** को **देख सकें** जबकि **दूसरे** सेट की **प्रक्रियाएं** एक **अलग** सेट के संसाधनों को देखती हैं। यह सुविधा एक ही नेमस्पेस को एक सेट के संसाधनों और प्रक्रियाओं के लिए होने के बावजूद, वे नेमस्पेस अलग-अलग संसाधनों को संदर्भित करते हैं। संसाधनों कई स्थानों में मौजूद हो सकते हैं।

डॉकर नेमस्पेस विभाजन प्राप्त करने के लिए निम्नलिखित लिनक्स कर्नल नेमस्पेस का उपयोग करता है:

* पिड नेमस्पेस
* माउंट नेमस्पेस
* नेटवर्क नेमस्पेस
* आईपीसी नेमस्पेस
* UTS नेमस्पेस

**नेमस्पेस के बारे में अधिक जानकारी के लिए** निम्नलिखित पेज देखें:

{% content-ref url="namespaces/" %}
[namespaces](namespaces/)
{% endcontent-ref %}

### cgroups

लिनक्स कर्नल सुविधा **cgroups** प्रक्रियाओं के एक सेट के बीच सीपीयू, मेमोरी, आईओ, नेटवर्क बैंडविड्थ जैसे संसाधनों को **प्रतिबंधित करने की क्षमता प्रदान करती है**। डॉकर को cgroup सुविधा का उपयोग करके कंटेनर निर्मित करने की अनुमति देता है जो विशेष कंटेनर के लिए संसाधन नियंत्रण करने की अनुमति देता है।\
निम्नलिखित कंटेनर को उपयोगकर्ता स्थान मेमोरी को 500m, कर्नल मेमोरी को 50m, सीपीयू शेयर को 512, ब्ल्कआईओवेट को 400 के साथ निर्मित किया गया है। सीपीयू शेयर एक अनुपात है जो कंटेनर के सीपीयू उपयोग को नियंत्रित करता है। इसका डिफ़ॉल्ट मूल्य 1024 है और 0 और 1024 के बीच रेंज है। यदि तीन कंटेनरों का सीपीयू शेयर 1024 का है, तो प्रत्येक कंटेनर CPU संसाधन संघर्ष की स्थिति में 33% तक CPU ले सकता है। ब्ल्कआईओवेट एक अनुपात है जो कंटेनर के आईओ को नियंत्रित करता है। इसका डिफ़ॉल्ट मूल्य 500 है और 10 और 1000 के बीच रेंज है।
```
docker run -it -m 500M --kernel-memory 50M --cpu-shares 512 --blkio-weight 400 --name ubuntu1 ubuntu bash
```
एक कंटेनर का सीग्रुप प्राप्त करने के लिए आप निम्नलिखित कर सकते हैं:
```bash
docker run -dt --rm denial sleep 1234 #Run a large sleep inside a Debian container
ps -ef | grep 1234 #Get info about the sleep process
ls -l /proc/<PID>/ns #Get the Group and the namespaces (some may be uniq to the hosts and some may be shred with it)
```
अधिक जानकारी के लिए देखें:

{% content-ref url="cgroups.md" %}
[cgroups.md](cgroups.md)
{% endcontent-ref %}

### Capabilities

क्षमताएं रूट उपयोगकर्ता के लिए अनुमति देने के लिए अधिक संयंत्रण देती हैं। Docker लिनक्स कर्नल क्षमता सुविधा का उपयोग करता है ताकि कंटेनर के भीतर किए जा सकने वाले आपरेशनों को सीमित किया जा सके, चाहे उपयोगकर्ता का प्रकार कुछ भी हो।

जब एक डॉकर कंटेनर चलाया जाता है, तो प्रक्रिया उसे छानने के लिए संवेदनशील क्षमताएं छोड़ देती है जिनका उपयोग विभाजन से बाहर निकलने के लिए कर सकती थी। इसका प्रयास यह है कि प्रक्रिया संवेदनशील कार्रवाईयों को करने और बाहर निकलने के लिए संवेदनशील कार्रवाईयों को करने में सक्षम न हो सके:

{% content-ref url="../linux-capabilities.md" %}
[linux-capabilities.md](../linux-capabilities.md)
{% endcontent-ref %}

### Docker में Seccomp

यह एक सुरक्षा सुविधा है जो Docker को कंटेनर के भीतर उपयोग किए जा सकने वाले syscalls को सीमित करने की अनुमति देती है:

{% content-ref url="seccomp.md" %}
[seccomp.md](seccomp.md)
{% endcontent-ref %}

### Docker में AppArmor

AppArmor कंटेनर को सीमित संसाधन सेट के साथ एक परियोजना प्रोफ़ाइल में बंधन करने के लिए एक कर्नल उन्नति है।:

{% content-ref url="apparmor.md" %}
[apparmor.md](apparmor.md)
{% endcontent-ref %}

### Docker में SELinux

[SELinux](https://www.redhat.com/en/blog/latest-container-exploit-runc-can-be-blocked-selinux) एक लेबलिंग सिस्टम है। हर प्रक्रिया और हर फ़ाइल सिस्टम ऑब्जेक्ट का एक लेबल होता है। SELinux नीतियाँ प्रक्रिया लेबल के बारे में नियम परिभाषित करती हैं कि वह सभी अन्य लेबलों के साथ क्या कर सकता है।

कंटेनर इंजन एकल सीमित SELinux लेबल के साथ कंटेनर प्रक्रियाएं लॉन्च करता है, आमतौर पर `container_t` होता है, और फिर कंटेनर को `container_file_t` लेबल वाले कंटेनर के भीतर सेट करता है। SELinux नीति नियमों का मतलब यही है कि **`container_t` प्रक्रियाएं केवल `container_file_t` लेबल वाली फ़ाइलों को पढ़ सकती हैं/लिख सकती हैं/चला सकती हैं**।

{% content-ref url="../selinux.md" %}
[selinux.md](../selinux.md)
{% endcontent-ref %}

### AuthZ & AuthN

एक अधिकृति प्लगइन वर्तमान प्रमाणीकरण संदर्भ और आदेश संदर्भ दोनों के आधार पर डॉकर डेमन को अनुरोधों को स्वीकृत या अस्वीकृत करता है। प्रमाणीकरण संदर्भ में सभी उपयोगकर्ता विवरण और प्रमाणीकरण विधि शामिल होती है। आदेश संदर्भ में सभी प्रासंगिक अनुरोध डेटा शामिल होता है।

{% content-ref url="authz-and-authn-docker-access-authorization-plugin.md" %}
[authz-and-authn-docker-access-authorization-plugin.md](authz-and-authn-docker-access-authorization-plugin.md)
{% endcontent-ref %}

## कंटेनर से DoS

यदि आप सही ढंग से कंटेनर के उपयोग करने की सीमा नहीं लगा रहे हैं, तो एक संकटग्रस्त कंटेनर में जहां वह चल रहा है, वहां होस्ट को DoS कर सकता है।

* CPU DoS
```bash
# stress-ng
sudo apt-get install -y stress-ng && stress-ng --vm 1 --vm-bytes 1G --verify -t 5m

# While loop
docker run -d --name malicious-container -c 512 busybox sh -c 'while true; do :; done'
```
# बैंडविड्थ डीओएस

बैंडविड्थ डीओएस (Denial of Service) एक हैकिंग तकनीक है जिसमें हम एक निश्चित नेटवर्क या सिस्टम को अत्यधिक ट्रैफिक या डेटा के साथ ओवरवेल्म करके उसे अनुपलब्ध कर देते हैं। इसका परिणाम होता है कि विकल्प उपयोगकर्ताओं को सेवा का उपयोग नहीं कर पाते हैं और इससे उन्हें नुकसान होता है।

बैंडविड्थ डीओएस हमें निश्चित समय तक नेटवर्क या सिस्टम को अनुपलब्ध करने की अनुमति देता है। इसका उपयोग करके हैकर नेटवर्क या सिस्टम की क्षमता को ओवरलोड करते हैं और इससे उपयोगकर्ताओं को सेवा का उपयोग नहीं कर पाने की स्थिति उत्पन्न करते हैं।

बैंडविड्थ डीओएस अक्सर नेटवर्क या सिस्टम के अस्थायी या स्थायी नुकसान का कारण बनता है। इसलिए, इस तकनीक के खिलाफ सुरक्षा के लिए उच्च स्तर की सतर्कता आवश्यक होती है।
```bash
nc -lvp 4444 >/dev/null & while true; do cat /dev/urandom | nc <target IP> 4444; done
```
## दिलचस्प Docker फ्लैग्स

### --privileged फ्लैग

निम्नलिखित पृष्ठ पर आप यह जान सकते हैं कि **`--privileged` फ्लैग का अर्थ क्या होता है**:

{% content-ref url="docker-privileged.md" %}
[docker-privileged.md](docker-privileged.md)
{% endcontent-ref %}

### --security-opt

#### no-new-privileges

यदि आप एक कंटेनर चला रहे हैं जहां एक हमलावर कम अधिकार वाले उपयोगकर्ता के रूप में पहुंच प्राप्त करता है। यदि आपके पास एक **गलत रूप से कॉन्फ़िगर किया गया suid बाइनरी** है, तो हमलावर इसे दुरुपयोग कर सकता है और **कंटेनर के भीतर अधिकारों को बढ़ा सकता है**। जो, उसे इससे बाहर निकलने की अनुमति दे सकता है।

**`no-new-privileges`** विकल्प सक्षम करके कंटेनर को चलाने से इस तरह की अधिकारों की बढ़ाई को रोका जा सकता है।
```
docker run -it --security-opt=no-new-privileges:true nonewpriv
```
#### अन्य

---

##### Docker Security

##### Docker सुरक्षा

---

##### Docker Privilege Escalation

##### Docker विशेषाधिकार उन्नयन

---

##### Docker Security Checklist

##### Docker सुरक्षा जांच सूची

---

##### Docker Security Best Practices

##### Docker सुरक्षा के सर्वोत्तम अभ्यास

---

##### Docker Security Tools

##### Docker सुरक्षा उपकरण

---

##### Docker Security Vulnerabilities

##### Docker सुरक्षा की कमियों

---

##### Docker Security Tips

##### Docker सुरक्षा युक्तियाँ

---

##### Docker Security Resources

##### Docker सुरक्षा संसाधन

---

##### Docker Security Cheat Sheet

##### Docker सुरक्षा चीट शीट

---

##### Docker Security Training

##### Docker सुरक्षा प्रशिक्षण

---

##### Docker Security Blogs

##### Docker सुरक्षा ब्लॉग

---

##### Docker Security Videos

##### Docker सुरक्षा वीडियो

---

##### Docker Security Presentations

##### Docker सुरक्षा प्रस्तुतियाँ

---

##### Docker Security Papers

##### Docker सुरक्षा पत्र

---

##### Docker Security Conferences

##### Docker सुरक्षा सम्मेलन

---

##### Docker Security Communities

##### Docker सुरक्षा समुदाय

---

##### Docker Security News

##### Docker सुरक्षा समाचार

---

##### Docker Security Forums

##### Docker सुरक्षा फोरम

---

##### Docker Security Courses

##### Docker सुरक्षा पाठ्यक्रम

---

##### Docker Security Certifications

##### Docker सुरक्षा प्रमाणीकरण

---

##### Docker Security Challenges

##### Docker सुरक्षा चुनौतियाँ

---

##### Docker Security Solutions

##### Docker सुरक्षा समाधान

---

##### Docker Security Auditing

##### Docker सुरक्षा में मुआयना

---

##### Docker Security Hardening

##### Docker सुरक्षा को कठोर बनाना

---

##### Docker Security Policies

##### Docker सुरक्षा नीतियाँ

---

##### Docker Security Guidelines

##### Docker सुरक्षा दिशानिर्देश

---

##### Docker Security Measures

##### Docker सुरक्षा उपाय

---

##### Docker Security Risks

##### Docker सुरक्षा जोखिम

---

##### Docker Security Controls

##### Docker सुरक्षा नियंत्रण

---

##### Docker Security Frameworks

##### Docker सुरक्षा ढांचा

---

##### Docker Security Architecture

##### Docker सुरक्षा वास्तुकला

---

##### Docker Security Features

##### Docker सुरक्षा सुविधाएँ

---

##### Docker Security Updates

##### Docker सुरक्षा अपडेट

---

##### Docker Security Advisories

##### Docker सुरक्षा सलाह

---

##### Docker Security Incident Response

##### Docker सुरक्षा घटना प्रतिक्रिया

---

##### Docker Security Monitoring

##### Docker सुरक्षा मॉनिटरिंग

---

##### Docker Security Logging

##### Docker सुरक्षा लॉगिंग

---

##### Docker Security Compliance

##### Docker सुरक्षा अनुपालन

---

##### Docker Security Audits

##### Docker सुरक्षा मुआयना

---

##### Docker Security Assessments

##### Docker सुरक्षा मूल्यांकन

---

##### Docker Security Scanning

##### Docker सुरक्षा स्कैनिंग

---

##### Docker Security Patching

##### Docker सुरक्षा पैचिंग

---

##### Docker Security Configuration

##### Docker सुरक्षा कॉन्फ़िगरेशन

---

##### Docker Security Monitoring Tools

##### Docker सुरक्षा मॉनिटरिंग उपकरण

---

##### Docker Security Best Practices Checklist

##### Docker सुरक्षा के सर्वोत्तम अभ्यास जांच सूची

---

##### Docker Security Tips and Tricks

##### Docker सुरक्षा युक्तियाँ और ट्रिक्स

---

##### Docker Security Guidelines and Recommendations

##### Docker सुरक्षा दिशानिर्देश और सिफारिशें

---

##### Docker Security Resources and References

##### Docker सुरक्षा संसाधन और संदर्भ

---

##### Docker Security Cheat Sheet and Quick Reference

##### Docker सुरक्षा चीट शीट और त्वरित संदर्भ

---

##### Docker Security Training and Courses

##### Docker सुरक्षा प्रशिक्षण और पाठ्यक्रम

---

##### Docker Security Blogs and Articles

##### Docker सुरक्षा ब्लॉग और लेख

---

##### Docker Security Videos and Presentations

##### Docker सुरक्षा वीडियो और प्रस्तुतियाँ

---

##### Docker Security Papers and Research

##### Docker सुरक्षा पत्र और अनुसंधान

---

##### Docker Security Conferences and Events

##### Docker सुरक्षा सम्मेलन और आयोजन

---

##### Docker Security Communities and Forums

##### Docker सुरक्षा समुदाय और फोरम

---

##### Docker Security News and Updates

##### Docker सुरक्षा समाचार और अपडेट

---

##### Docker Security Challenges and Solutions

##### Docker सुरक्षा चुनौतियाँ और समाधान

---

##### Docker Security Controls and Measures

##### Docker सुरक्षा नियंत्रण और उपाय

---

##### Docker Security Frameworks and Architectures

##### Docker सुरक्षा ढांचा और वास्तुकला

---

##### Docker Security Features and Capabilities

##### Docker सुरक्षा सुविधाएँ और क्षमताएँ

---

##### Docker Security Updates and Patches

##### Docker सुरक्षा अपडेट और पैच

---

##### Docker Security Advisories and Alerts

##### Docker सुरक्षा सलाह और चेतावनियाँ

---

##### Docker Security Incident Response and Handling

##### Docker सुरक्षा घटना प्रतिक्रिया और हैंडलिंग

---

##### Docker Security Monitoring and Logging

##### Docker सुरक्षा मॉनिटरिंग और लॉगिंग

---

##### Docker Security Compliance and Auditing

##### Docker सुरक्षा अनुपालन और मुआयना

---

##### Docker Security Assessments and Testing

##### Docker सुरक्षा मूल्यांकन और परीक्षण

---

##### Docker Security Scanning and Vulnerability Assessment

##### Docker सुरक्षा स्कैनिंग और कमजोरी मूल्यांकन

---

##### Docker Security Patching and Updates

##### Docker सुरक्षा पैचिंग और अपडेट

---

##### Docker Security Configuration and Hardening

##### Docker सुरक्षा कॉन्फ़िगरेशन और कठोर बनाना

---

##### Docker Security Monitoring Tools and Utilities

##### Docker सुरक्षा मॉनिटरिंग उपकरण और उपयोगिताएँ

---

##### Docker Security Best Practices Checklist and Guidelines

##### Docker सुरक्षा के सर्वोत्तम अभ्यास जांच सूची और दिशानिर्देश

---

##### Docker Security Tips and Tricks for Secure Containers

##### Docker सुरक्षा युक्तियाँ और ट्रिक्स सुरक्षित कंटेनर के लिए

---

##### Docker Security Guidelines and Recommendations for Secure Deployment

##### Docker सुरक्षा दिशानिर्देश और सिफारिशें सुरक्षित डिप्लॉयमेंट के लिए

---

##### Docker Security Resources and References for Further Learning

##### Docker सुरक्षा संसाधन और संदर्भ आगे की सीखने के लिए

---

##### Docker Security Cheat Sheet and Quick Reference for Pentesters

##### Docker सुरक्षा चीट शीट और त्वरित संदर्भ पेंटेस्टर्स के लिए

---

##### Docker Security Training and Courses for Ethical Hackers

##### Docker सुरक्षा प्रशिक्षण और पाठ्यक्रम नैतिक हैकर्स के लिए

---

##### Docker Security Blogs and Articles for Security Enthusiasts

##### Docker सुरक्षा ब्लॉग और लेख सुरक्षा प्रेमियों के लिए

---

##### Docker Security Videos and Presentations for Knowledge Sharing

##### Docker सुरक्षा वीडियो और प्रस्तुतियाँ ज्ञान साझा करने के लिए

---

##### Docker Security Papers and Research for In-depth Understanding

##### Docker सुरक्षा पत्र और अनुसंधान गहरी समझ के लिए

---

##### Docker Security Conferences and Events for Networking

##### Docker सुरक्षा सम्मेलन और आयोजन नेटवर्किंग के लिए

---

##### Docker Security Communities and Forums for Collaboration

##### Docker सुरक्षा समुदाय और फोरम सहयोग के लिए

---

##### Docker Security News and Updates for Latest Information

##### Docker सुरक्षा समाचार और अपडेट नवीनतम जानकारी के लिए

---

##### Docker Security Challenges and Solutions for Problem-solving

##### Docker सुरक्षा चुनौतियाँ और समाधान समस्या का समाधान करने के लिए

---

##### Docker Security Controls and Measures for Risk Mitigation

##### Docker सुरक्षा नियंत्रण और उपाय जोखिम कम करने के लिए

---

##### Docker Security Frameworks and Architectures for Designing Secure Systems

##### Docker सुरक्षा ढांचा और वास्तुकला सुरक्षित सिस्टम डिज़ाइन के लिए

---

##### Docker Security Features and Capabilities for Enhanced Protection

##### Docker सुरक्षा सुविधाएँ और क्षमताएँ बढ़ी हुई सुरक्षा के लिए

---

##### Docker Security Updates and Patches for Bug Fixes

##### Docker सुरक्षा अपडेट और पैच बग फिक्स के लिए

---

##### Docker Security Advisories and Alerts for Timely Warnings

##### Docker सुरक्षा सलाह और चेतावनियाँ समय पर चेतावनी के लिए

---

##### Docker Security Incident Response and Handling for Effective Management

##### Docker सुरक्षा घटना प्रतिक्रिया और हैंडलिंग प्रभावी प्रबंधन के लिए

---

##### Docker Security Monitoring and Logging for Continuous Surveillance

##### Docker सुरक्षा मॉनिटरिंग और लॉगिंग निरंतर निगरानी के लिए

---

##### Docker Security Compliance and Auditing for Regulatory Requirements

##### Docker सुरक्षा अनुपालन और मुआयना नियामक आवश्यकताओं के लिए

---

##### Docker Security Assessments and Testing for Vulnerability Analysis

##### Docker सुरक्षा मूल्यांकन और परीक्षण कमजोरी विश्लेषण के लिए

---

##### Docker Security Scanning and Vulnerability Assessment for Risk Evaluation

##### Docker सुरक्षा स्कैनिंग और कमजोरी मूल्यांकन जोखिम मूल्यांकन के लिए

---

##### Docker Security Patching and Updates for System Maintenance

##### Docker सुरक्षा पैचिंग और अपडेट सिस्टम रखरखाव के लिए

---

##### Docker Security Configuration and Hardening for Secure Setup

##### Docker सुरक्षा कॉन्फ़िगरेशन और कठोर बनाना सुरक्षित सेटअप के लिए

---

##### Docker Security Monitoring Tools and Utilities for Real-time Analysis

##### Docker सुरक्षा मॉनिटरिंग उपकरण और उपयोगिताएँ वास्तविक समय विश्लेषण के लिए

---

##### Docker Security Best Practices Checklist and Guidelines for Secure Deployment

##### Docker सुरक्षा के सर्वोत्तम अभ्यास जांच सूची और दिशानिर्देश सुरक्षित
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

## अन्य सुरक्षा संबंधित विचार

### सीक्रेट्स का प्रबंधन

सबसे पहले, **उन्हें अपने इमेज के अंदर न डालें!**

अपनी संवेदनशील जानकारी के लिए, **पर्याप्त नहीं है कि आप environment variables का उपयोग करें**. कोई भी व्यक्ति जो `docker inspect` या `exec` कोंटेनर में चला सकता है, वह आपका सीक्रेट खोज सकता है।

Docker volumes बेहतर हैं। ये Docker डॉक्स में सिफारिश किए जाने वाले तरीके हैं जिनसे आप अपनी संवेदनशील जानकारी तक पहुंच सकते हैं। आप **एक वॉल्यूम को मेमोरी में स्थानिक फ़ाइल सिस्टम के रूप में उपयोग कर सकते हैं**। वॉल्यूम्स `docker inspect` और लॉगिंग के जोखिम को हटा देते हैं। हालांकि, **रूट उपयोगकर्ता अभी भी सीक्रेट देख सकते हैं, साथ ही कोई भी व्यक्ति जो कंटेनर में `exec` कर सकता है**।

वॉल्यूम्स से बेहतर, Docker सीक्रेट्स का उपयोग करें।

यदि आपको अपने इमेज में **सीक्रेट की आवश्यकता है**, तो आप **BuildKit** का उपयोग कर सकते हैं। BuildKit निर्माण समय को काफी कम करता है और अन्य अच्छी सुविधाएं हैं, जिसमें **निर्माण समय सीक्रेट समर्थन** शामिल है।

इसके तीन तरीके हैं जिनसे आप BuildKit बैकएंड को निर्दिष्ट कर सकते हैं ताकि आप इसकी सुविधाओं का उपयोग कर सकें:

1. इसे एक environment variable के रूप में सेट करें `export DOCKER_BUILDKIT=1`.
2. अपने `build` या `run` कमांड को `DOCKER_BUILDKIT=1` के साथ शुरू करें।
3. डिफ़ॉल्ट रूप में BuildKit को सक्षम करें। /_etc/docker/daemon.json_ में विन्यास सेट करें और इसे _true_ के साथ सेट करें: `{ "features": { "buildkit": true } }`। फिर Docker को पुनः चालू करें।
4. फिर आप इस तरह से `--secret` फ़्लैग के साथ निर्माण समय में सीक्रेट का उपयोग कर सकते हैं:
```bash
docker build --secret my_key=my_value ,src=path/to/my_secret_file .
```
जहां आपकी फ़ाइल कुंजी-मान-जोड़ के रूप में आपकी सीक्रेट को निर्दिष्ट करती है।

ये सीक्रेट इमेज निर्माण कैश से छूटते हैं और अंतिम इमेज से भी।

यदि आपको अपने **चल रहे कंटेनर में सीक्रेट की आवश्यकता है**, और सिर्फ अपनी इमेज निर्माण के समय नहीं, तो **Docker Compose या Kubernetes** का उपयोग करें।

Docker Compose के साथ, सीक्रेट कुंजी-मान-जोड़ को एक सेवा में जोड़ें और सीक्रेट फ़ाइल को निर्दिष्ट करें। नीचे दिए गए उदाहरण को [Stack Exchange उत्तर](https://serverfault.com/a/936262/535325) के लिए धन्यवाद देते हुए Docker Compose सीक्रेट्स युक्त यह उदाहरण संशोधित किया गया है।

सीक्रेट्स के साथ उदाहरण `docker-compose.yml`:
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
फिर सामान्य रूप से `docker-compose up --build my_service` के साथ Compose को शुरू करें।

यदि आप [Kubernetes](https://kubernetes.io/docs/concepts/configuration/secret/) का उपयोग कर रहे हैं, तो इसमें सीक्रेट का समर्थन होता है। [Helm-Secrets](https://github.com/futuresimple/helm-secrets) सीक्रेट्स प्रबंधन को K8s में आसान बनाने में मदद कर सकता है। इसके अलावा, K8s में Role Based Access Controls (RBAC) होता है - जैसा कि Docker Enterprise में भी होता है। RBAC सीक्रेट्स प्रबंधन को संघ के लिए प्रबंधन और सुरक्षित बनाने में सहायता करता है।

### gVisor

**gVisor** एक एप्लिकेशन कर्नल है, जो गो में लिखा गया है, जो लिनक्स सिस्टम सतह का एक बड़ा हिस्सा लागू करता है। इसमें एक [Open Container Initiative (OCI)](https://www.opencontainers.org) रनटाइम है जिसका नाम `runsc` है, जो एप्लिकेशन और होस्ट कर्नल के बीच एक **अलगाव सीमा** प्रदान करता है। `runsc` रनटाइम Docker और Kubernetes के साथ एकीकृत होता है, जिससे सैंडबॉक्स कंटेनर चलाना आसान हो जाता है।

{% embed url="https://github.com/google/gvisor" %}

### Kata Containers

**Kata Containers** एक ओपन सोर्स समुदाय है जो एक सुरक्षित कंटेनर रनटाइम बनाने के लिए काम कर रहा है, जिसमें हार्डवेयर वर्चुअलाइज़ेशन तकनीक का उपयोग करके कमजोर वर्कलोड अलगाव प्रदान किया जाता है। यह कंटेनर की तरह दिखता है और कार्य करता है, लेकिन एक दूसरे सुरक्षा स्तर के रूप में हार्डवेयर वर्चुअलाइज़ेशन तकनीक का उपयोग करके मजबूत वर्कलोड अलगाव प्रदान करता है।

{% embed url="https://katacontainers.io/" %}

### सारांश युक्तियाँ

* **`--privileged` फ्लैग का उपयोग न करें और कंटेनर के अंदर** [**Docker सॉकेट माउंट न करें**](https://raesene.github.io/blog/2016/03/06/The-Dangers-Of-Docker.sock/)**।** Docker सॉकेट कंटेनर को उत्पन्न करने की अनुमति देता है, इसलिए यह एक आसान तरीका है होस्ट पर पूर्ण नियंत्रण प्राप्त करने का, उदाहरण के लिए, `--privileged` फ्लैग के साथ एक और कंटेनर चलाकर।
* कंटेनर के अंदर **रूट के रूप में न चलाएं। एक** [**अलग उपयोगकर्ता**](https://docs.docker.com/develop/develop-images/dockerfile\_best-practices/#user) **और** [**उपयोगकर्ता नेमस्पेस**](https://docs.docker.com/engine/security/userns-remap/)** का उपयोग करें।** कंटेनर में रूट होस्ट के समान होता है जब तक उपयोगकर्ता नेमस्पेस के साथ पुनर्मापित नहीं किया जाता है। यह केवल लाइटली प्रतिबंधित होता है, मुख्य रूप से, लिनक्स नेमस्पेस, क्षमताएँ और सीग्रुप्स द्वारा।
* [**सभी क्षमताएँ छोड़ दें**](https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities) **(`--cap-drop=all`) और केवल वे क्षमताएँ सक्षम करें जो आवश्यक हों** (`--cap-add=...`)। बहुत सारे वर्कलोड को कोई क्षमताएँ नहीं चाहिए और उन्हें जोड़ने से एक संभावित हमले की दायरा बढ़ जाती है।
* [**“no-new-privileges” सुरक्षा विकल्प का उपयोग करें**](https://raesene.github.io/blog/2019/06/01/docker-capabilities-and-no-new-privs/) प्रक्रियाओं को अधिक विशेषाधिकार प्राप्त करने से रोकने के लिए, उदाहरण के लिए suid बाइनरी के माध्यम से।
* [**कंटेनर के लिए उपलब्ध संसाधनों की सीमा निर्धारित करें**](https://docs.docker.com/engine/reference/run/#runtime-constraints-on-resources)**।** संसाधन सीमाएं विलय सेवा हमलों से मशीन की सुरक्षा को सुरक्षित रख सकती हैं।
* **सीक्रेंप** [**seccomp**](https://docs.docker.com/engine/security/seccomp/)**,** [**AppArmor**](https://docs.docker.com/engine/security/apparmor/) **(या SELinux)** प्रोफाइल को
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करके आसानी से बनाएं और **स्वचालित कार्यप्रवाह** जो दुनिया के **सबसे उन्नत** समुदाय उपकरणों द्वारा संचालित होता है।\
आज ही पहुंच प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने की पहुंच** चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT संग्रह**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें,** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **में PR जमा करके**।

</details>
