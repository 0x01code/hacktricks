# Docker सुरक्षा

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) को **फॉलो करें**.
* **अपनी हैकिंग ट्रिक्स साझा करें, HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके.

</details>

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks) का उपयोग करके आसानी से **वर्कफ्लोज़ को बिल्ड और ऑटोमेट** करें जो दुनिया के **सबसे उन्नत** समुदाय टूल्स द्वारा संचालित होते हैं.\
आज ही एक्सेस प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## **बेसिक Docker Engine सुरक्षा**

Docker engine कंटेनर्स को चलाने और प्रबंधित करने का मुख्य कार्य करता है. Docker engine Linux kernel की विशेषताओं जैसे कि **Namespaces** और **Cgroups** का उपयोग करता है ताकि कंटेनर्स के बीच बुनियादी **अलगाव** प्रदान किया जा सके. यह **Capabilities dropping**, **Seccomp**, **SELinux/AppArmor का उपयोग करके बेहतर अलगाव** प्राप्त करने के लिए भी करता है.

अंत में, एक **auth plugin** का उपयोग करके उपयोगकर्ताओं द्वारा किए जा सकने वाले क्रियाओं को **सीमित किया जा सकता है**.

![](<../../../.gitbook/assets/image (625) (1) (1).png>)

### **Docker engine सुरक्षित पहुँच**

Docker client Docker engine तक **स्थानीय रूप से Unix socket का उपयोग करके या दूरस्थ रूप से http** माध्यम से पहुँच सकता है. इसे दूरस्थ रूप से उपयोग करने के लिए, https और **TLS** का उपयोग करना आवश्यक है ताकि गोपनीयता, अखंडता और प्रमाणीकरण सुनिश्चित किया जा सके.

डिफ़ॉल्ट रूप से Unix socket `unix:///var/`\
`run/docker.sock` पर सुनता है और Ubuntu वितरणों में, Docker स्टार्ट विकल्प `/etc/default/docker` में निर्दिष्ट किए जाते हैं. Docker API और क्लाइंट को Docker engine तक दूरस्थ रूप से पहुँचने के लिए, हमें **Docker daemon को http socket का उपयोग करके उजागर करना होगा**. यह किया जा सकता है:
```bash
DOCKER_OPTS="-D -H unix:///var/run/docker.sock -H
tcp://192.168.56.101:2376" -> add this to /etc/default/docker
Sudo service docker restart -> Restart Docker daemon
```
```
Docker daemon को http का उपयोग करके उजागर करना अच्छी प्रथा नहीं है और इसे https का उपयोग करके कनेक्शन को सुरक्षित करने की आवश्यकता है। दो विकल्प हैं: पहला विकल्प **client द्वारा server की पहचान की पुष्टि** के लिए है और दूसरे विकल्प में **client और server दोनों एक दूसरे की पहचान की पुष्टि करते हैं**। प्रमाणपत्र server की पहचान स्थापित करते हैं। दोनों विकल्पों का उदाहरण देखने के लिए [**इस पृष्ठ को देखें**](https://sreeninet.wordpress.com/2016/03/06/docker-security-part-3engine-access/).

### **Container image सुरक्षा**

Container images या तो private repository या public repository में संग्रहीत की जाती हैं। Docker द्वारा Container images को संग्रहित करने के लिए निम्नलिखित विकल्प प्रदान किए जाते हैं:

* [Docker hub](https://hub.docker.com) – यह Docker द्वारा प्रदान की गई एक public registry सेवा है
* [Docker registry](https://github.com/%20docker/distribution) – यह एक open source परियोजना है जिसका उपयोग users अपनी registry होस्ट करने के लिए कर सकते हैं।
* [Docker trusted registry](https://www.docker.com/docker-trusted-registry) – यह Docker का commercial implementation है Docker registry का और यह LDAP directory service integration के साथ role based user authentication प्रदान करता है।

### Image Scanning

Containers में **security vulnerabilities** हो सकती हैं या तो base image के कारण या फिर base image के ऊपर स्थापित सॉफ्टवेयर के कारण। Docker एक परियोजना पर काम कर रहा है जिसे **Nautilus** कहा जाता है जो Containers की security scan करता है और vulnerabilities की सूची बनाता है। Nautilus vulnerability repository के साथ प्रत्येक Container image layer की तुलना करके security holes की पहचान करता है।

अधिक [**जानकारी के लिए यह पढ़ें**](https://docs.docker.com/engine/scan/).

* **`docker scan`**

**`docker scan`** command आपको मौजूदा Docker images को image name या ID का उपयोग करके scan करने की अनुमति देता है। उदाहरण के लिए, hello-world image को scan करने के लिए निम्नलिखित command चलाएं:
```
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
### Docker इमेज साइनिंग

Docker Container इमेजेज को सार्वजनिक या निजी रजिस्ट्री में संग्रहीत किया जा सकता है। Container इमेजेज को साइन करना आवश्यक है ताकि यह पुष्टि की जा सके कि इमेजेज में छेड़छाड़ नहीं की गई है। सामग्री प्रकाशक Container इमेज को साइन करने और उसे रजिस्ट्री में धकेलने का काम करता है।\
निम्नलिखित Docker सामग्री विश्वास पर कुछ विवरण हैं:

* Docker सामग्री विश्वास [Notary open source project](https://github.com/docker/notary) का एक कार्यान्वयन है। Notary open source project [The Update Framework (TUF) project](https://theupdateframework.github.io) पर आधारित है।
* Docker सामग्री विश्वास को `export DOCKER_CONTENT_TRUST=1` के साथ सक्षम किया जाता है। Docker संस्करण 1.10 के अनुसार, सामग्री विश्वास डिफ़ॉल्ट रूप से सक्षम **नहीं** है।
* **जब** सामग्री विश्वास **सक्षम होता है**, हम केवल साइन की गई इमेजेज ही **खींच सकते हैं**। जब इमेज को धकेला जाता है, हमें टैगिंग कुंजी दर्ज करनी होती है।
* जब प्रकाशक पहली **बार** docker push का उपयोग करके इमेज को **धकेलता है**, तब **रूट कुंजी और टैगिंग कुंजी** के लिए एक **पासफ्रेज** दर्ज करने की आवश्यकता होती है। अन्य कुंजियाँ स्वचालित रूप से उत्पन्न की जाती हैं।
* Docker ने Yubikey का उपयोग करके हार्डवेयर कुंजियों के लिए भी समर्थन जोड़ा है और विवरण [यहाँ](https://blog.docker.com/2015/11/docker-content-trust-yubikey/) उपलब्ध हैं।

निम्नलिखित वह **त्रुटि** है जो हमें मिलती है जब **सामग्री विश्वास सक्षम होता है और इमेज साइन नहीं की गई होती है**।
```shell-session
$ docker pull smakam/mybusybox
Using default tag: latest
No trust data for latest
```
निम्नलिखित आउटपुट में **Docker hub पर image को signing सक्षम के साथ push किया जा रहा है** दिखाया गया है। चूंकि यह पहली बार नहीं है, इसलिए उपयोगकर्ता से केवल repository key के लिए passphrase दर्ज करने के लिए कहा जा रहा है।
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
```markdown
रूट की, रिपॉजिटरी की और पासफ्रेज को सुरक्षित स्थान पर संग्रहित करना आवश्यक है। निजी कुंजियों का बैकअप लेने के लिए निम्नलिखित कमांड का उपयोग किया जा सकता है:
```
```bash
tar -zcvf private_keys_backup.tar.gz ~/.docker/trust/private
```
जब मैंने Docker होस्ट बदला, तो मुझे नए होस्ट से संचालित करने के लिए रूट कीज़ और रिपॉजिटरी कीज़ को स्थानांतरित करना पड़ा।

***

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करके आसानी से **वर्कफ्लोज़ को बनाएं और स्वचालित करें** जो दुनिया के **सबसे उन्नत** समुदाय टूल्स द्वारा संचालित होते हैं।\
आज ही पहुंच प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## कंटेनर सुरक्षा विशेषताएं

<details>

<summary>कंटेनर सुरक्षा विशेषताओं का सारांश</summary>

**Namespaces**

Namespaces एक प्रोजेक्ट को दूसरे से अलग करने के लिए उपयोगी होते हैं, प्रोसेस संचार, नेटवर्क, माउंट्स को अलग करते हैं... यह Docker प्रोसेस को अन्य प्रोसेसेस से अलग करने में सहायक होता है (और यहां तक कि /proc फोल्डर से भी) ताकि यह अन्य प्रोसेसेस का दुरुपयोग करके बाहर न निकल सके।

यह संभव हो सकता है "बच निकलना" या अधिक सटीक रूप से **नए namespaces बनाना** बाइनरी **`unshare`** का उपयोग करके (जो **`unshare`** syscall का उपयोग करता है)। Docker डिफ़ॉल्ट रूप से इसे रोकता है, लेकिन kubernetes नहीं करता (इस लेखन के समय)।\
वैसे भी, यह नए namespaces बनाने के लिए सहायक है, लेकिन **होस्ट के डिफ़ॉल्ट namespaces में वापस जाने के लिए नहीं** (जब तक आपके पास होस्ट namespaces के अंदर कुछ `/proc` तक पहुंच न हो, जहां आप **`nsenter`** का उपयोग करके होस्ट namespaces में प्रवेश कर सकते हैं।)

**CGroups**

यह संसाधनों को सीमित करने की अनुमति देता है और प्रोसेस के अलगाव की सुरक्षा को प्रभावित नहीं करता है (छोड़कर `release_agent` जिसका उपयोग बच निकलने के लिए किया जा सकता है)।

**Capabilities Drop**

मुझे यह प्रोसेस अलगाव सुरक्षा के संबंध में **सबसे महत्वपूर्ण** विशेषताओं में से एक लगता है। यह इसलिए है क्योंकि बिना capabilities के, यहां तक कि अगर प्रोसेस रूट के रूप में चल रहा है **आप कुछ विशेषाधिकार प्राप्त क्रियाएं नहीं कर पाएंगे** (क्योंकि कॉल किया गया **`syscall`** अनुमति त्रुटि वापस कर देगा क्योंकि प्रोसेस के पास आवश्यक capabilities नहीं हैं)।

ये वे **शेष capabilities** हैं जो प्रोसेस दूसरों को छोड़ने के बाद रहती हैं:

{% code overflow="wrap" %}
```
Current: cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap=ep
```
{% endcode %}

**Seccomp**

यह Docker में डिफ़ॉल्ट रूप से सक्षम होता है। यह **प्रोसेस द्वारा कॉल किए जा सकने वाले syscalls को और भी सीमित करने में मदद करता है**।\
**डिफ़ॉल्ट Docker Seccomp प्रोफ़ाइल** यहाँ पाई जा सकती है: [https://github.com/moby/moby/blob/master/profiles/seccomp/default.json](https://github.com/moby/moby/blob/master/profiles/seccomp/default.json)

**AppArmor**

Docker में एक टेम्पलेट है जिसे आप सक्रिय कर सकते हैं: [https://github.com/moby/moby/tree/master/profiles/apparmor](https://github.com/moby/moby/tree/master/profiles/apparmor)

इससे क्षमताओं, syscalls, फ़ाइलों और फ़ोल्डरों तक पहुँच को कम करने में मदद मिलेगी...

</details>

### Namespaces

**Namespaces** लिनक्स कर्नेल की एक विशेषता है जो **कर्नेल संसाधनों को विभाजित करती है** ताकि एक समूह के **प्रोसेस** एक समूह के **संसाधनों** को **देख** सकें जबकि **दूसरे** समूह के **प्रोसेस** एक **अलग** समूह के संसाधनों को देखें। यह विशेषता एक ही नामस्थान का उपयोग करके संसाधनों और प्रोसेसों के एक समूह के लिए काम करती है, लेकिन वे नामस्थान विशिष्ट संसाधनों को संदर्भित करते हैं। संसाधन कई स्थानों में मौजूद हो सकते हैं।

Docker निम्नलिखित लिनक्स कर्नेल Namespaces का उपयोग करके कंटेनर अलगाव प्राप्त करता है:

* pid namespace
* mount namespace
* network namespace
* ipc namespace
* UTS namespace

**Namespaces के बारे में अधिक जानकारी के लिए** निम्नलिखित पृष्ठ देखें:

{% content-ref url="namespaces/" %}
[namespaces](namespaces/)
{% endcontent-ref %}

### cgroups

लिनक्स कर्नेल विशेषता **cgroups** प्रोसेसों के एक समूह के बीच cpu, memory, io, network bandwidth जैसे संसाधनों को **प्रतिबंधित करने की क्षमता प्रदान करती है**। Docker cgroup विशेषता का उपयोग करके कंटेनर बनाने की अनुमति देता है जो विशिष्ट कंटेनर के लिए संसाधन नियंत्रण की अनुमति देता है।\
निम्नलिखित एक कंटेनर है जिसमें उपयोगकर्ता स्थान की मेमोरी 500m तक सीमित है, कर्नेल मेमोरी 50m तक सीमित है, cpu शेयर 512 है, blkioweight 400 है। CPU शेयर एक अनुपात है जो कंटेनर के CPU उपयोग को नियंत्रित करता है। इसका डिफ़ॉल्ट मान 1024 है और यह 0 से 1024 के बीच होता है। यदि तीन कंटेनरों का CPU शेयर समान 1024 है, तो CPU संसाधन संघर्ष की स्थिति में प्रत्येक कंटेनर CPU का 33% तक ले सकता है। blkio-weight एक अनुपात है जो कंटेनर के IO को नियंत्रित करता है। इसका डिफ़ॉल्ट मान 500 है और यह 10 से 1000 के बीच होता है।
```
docker run -it -m 500M --kernel-memory 50M --cpu-shares 512 --blkio-weight 400 --name ubuntu1 ubuntu bash
```
किसी कंटेनर का cgroup प्राप्त करने के लिए आप कर सकते हैं:
```bash
docker run -dt --rm denial sleep 1234 #Run a large sleep inside a Debian container
ps -ef | grep 1234 #Get info about the sleep process
ls -l /proc/<PID>/ns #Get the Group and the namespaces (some may be uniq to the hosts and some may be shred with it)
```
अधिक जानकारी के लिए देखें:

{% content-ref url="cgroups.md" %}
[cgroups.md](cgroups.md)
{% endcontent-ref %}

### क्षमताएँ (Capabilities)

क्षमताएँ रूट उपयोगकर्ता के लिए **अधिक सूक्ष्म नियंत्रण प्रदान करती हैं** जो क्षमताएँ अनुमति दी जा सकती हैं। Docker लिनक्स कर्नेल क्षमता सुविधा का उपयोग करता है ताकि **कंटेनर के अंदर किए जा सकने वाले कार्यों को सीमित किया जा सके** चाहे उपयोगकर्ता का प्रकार कुछ भी हो।

जब एक डॉकर कंटेनर चलाया जाता है, तो **प्रक्रिया संवेदनशील क्षमताएँ छोड़ देती है जिनका उपयोग प्रक्रिया अलगाव से बचने के लिए कर सकती है**। इससे यह सुनिश्चित किया जाता है कि प्रक्रिया संवेदनशील कार्यों को अंजाम नहीं दे पाएगी और बच नहीं पाएगी:

{% content-ref url="../linux-capabilities.md" %}
[linux-capabilities.md](../linux-capabilities.md)
{% endcontent-ref %}

### Docker में Seccomp

यह एक सुरक्षा सुविधा है जो Docker को **सिस्टम कॉल्स को सीमित करने की अनुमति देती है** जो कंटेनर के अंदर उपयोग की जा सकती हैं:

{% content-ref url="seccomp.md" %}
[seccomp.md](seccomp.md)
{% endcontent-ref %}

### Docker में AppArmor

**AppArmor** एक कर्नेल सुधार है जो **कंटेनरों** को **सीमित** संसाधनों के साथ **प्रति-प्रोग्राम प्रोफाइल** के अनुसार संयमित करता है।:

{% content-ref url="apparmor.md" %}
[apparmor.md](apparmor.md)
{% endcontent-ref %}

### Docker में SELinux

[SELinux](https://www.redhat.com/en/blog/latest-container-exploit-runc-can-be-blocked-selinux) एक **लेबलिंग** **सिस्टम** है। हर **प्रक्रिया** और हर **फाइल सिस्टम ऑब्जेक्ट** का एक **लेबल** होता है। SELinux नीतियाँ नियम निर्धारित करती हैं कि एक **प्रक्रिया लेबल क्या कर सकती है सिस्टम पर अन्य सभी लेबलों के साथ**।

कंटेनर इंजन **कंटेनर प्रक्रियाओं को एक एकल संयमित SELinux लेबल के साथ लॉन्च करते हैं**, आमतौर पर `container_t`, और फिर कंटेनर के अंदर के कंटेनर को लेबल `container_file_t` के रूप में सेट करते हैं। SELinux नीति नियम मूल रूप से कहते हैं कि **`container_t` प्रक्रियाएँ केवल `container_file_t` लेबल वाली फाइलों को पढ़/लिख/निष्पादित कर सकती हैं**।

{% content-ref url="../selinux.md" %}
[selinux.md](../selinux.md)
{% endcontent-ref %}

### AuthZ & AuthN

एक प्राधिकरण प्लगइन **अनुमोदित** या **अस्वीकार** करता है **अनुरोधों** को Docker **डेमन** के लिए आधारित होता है वर्तमान **प्रमाणीकरण** संदर्भ और **कमांड** **संदर्भ** पर। **प्रमाणीकरण** **संदर्भ** में सभी **उपयोगकर्ता विवरण** और **प्रमाणीकरण** **विधि** शामिल होती है। **कमांड संदर्भ** में सभी **प्रासंगिक** **अनुरोध** डेटा शामिल होता है।

{% content-ref url="authz-and-authn-docker-access-authorization-plugin.md" %}
[authz-and-authn-docker-access-authorization-plugin.md](authz-and-authn-docker-access-authorization-plugin.md)
{% endcontent-ref %}

## कंटेनर से DoS

यदि आप उचित रूप से सीमित नहीं कर रहे हैं कि एक कंटेनर कितने संसाधनों का उपयोग कर सकता है, तो एक समझौता किया गया कंटेनर जिस पर वह चल रहा है उस होस्ट को DoS कर सकता है।

* CPU DoS
```bash
# stress-ng
sudo apt-get install -y stress-ng && stress-ng --vm 1 --vm-bytes 1G --verify -t 5m

# While loop
docker run -d --name malicious-container -c 512 busybox sh -c 'while true; do :; done'
```
* बैंडविड्थ DoS
```bash
nc -lvp 4444 >/dev/null & while true; do cat /dev/urandom | nc <target IP> 4444; done
```
## दिलचस्प Docker फ्लैग्स

### --privileged फ्लैग

निम्नलिखित पृष्ठ पर आप सीख सकते हैं **`--privileged` फ्लैग का क्या अर्थ है**:

{% content-ref url="docker-privileged.md" %}
[docker-privileged.md](docker-privileged.md)
{% endcontent-ref %}

### --security-opt

#### no-new-privileges

यदि आप एक कंटेनर चला रहे हैं जहां एक हमलावर कम विशेषाधिकार वाले उपयोगकर्ता के रूप में पहुँच प्राप्त कर लेता है। यदि आपके पास एक **गलत-कॉन्फ़िगर सुइड बाइनरी** है, तो हमलावर इसका दुरुपयोग कर सकता है और **कंटेनर के अंदर विशेषाधिकार बढ़ा सकता है**। जो, उसे इससे बाहर निकलने की अनुमति दे सकता है।

कंटेनर को **`no-new-privileges`** विकल्प सक्षम के साथ चलाने से **इस प्रकार के विशेषाधिकार वृद्धि को रोका जा सकता है**।
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

### सीक्रेट्स का प्रबंधन

सबसे पहले, **उन्हें अपनी इमेज के अंदर न रखें!**

साथ ही, **पर्यावरण चरों का उपयोग अपनी संवेदनशील जानकारी के लिए न करें**। कोई भी **जो `docker inspect` या `exec` कंटेनर में कर सकता है, वह आपके सीक्रेट को ढूंढ सकता है**।

Docker volumes बेहतर हैं। वे Docker दस्तावेज़ में आपकी संवेदनशील जानकारी तक पहुँचने के लिए सुझाए गए तरीके हैं। आप **एक volume का उपयोग स्मृति में अस्थायी फ़ाइल सिस्टम के रूप में कर सकते हैं**। Volumes `docker inspect` और लॉगिंग जोखिम को हटा देते हैं। हालांकि, **रूट उपयोगकर्ता अभी भी सीक्रेट देख सकते हैं, जैसे कोई भी जो कंटेनर में `exec` कर सकता है**।

Volumes से भी **बेहतर, Docker secrets का उपयोग करें**।

यदि आपको केवल **अपनी इमेज में सीक्रेट की आवश्यकता है**, तो आप **BuildKit** का उपयोग कर सकते हैं। BuildKit निर्माण समय को काफी कम करता है और अन्य अच्छी विशेषताएं भी हैं, जिसमें **निर्माण-समय सीक्रेट्स समर्थन** शामिल है।

BuildKit बैकएंड को निर्दिष्ट करने के तीन तरीके हैं ताकि आप इसकी विशेषताओं का उपयोग अब कर सकें:

1. इसे एक पर्यावरण चर के रूप में सेट करें `export DOCKER_BUILDKIT=1` के साथ।
2. अपने `build` या `run` कमांड को `DOCKER_BUILDKIT=1` के साथ शुरू करें।
3. BuildKit को डिफ़ॉल्ट रूप से सक्षम करें। /_etc/docker/daemon.json_ में कॉन्फ़िगरेशन सेट करें _true_ के साथ: `{ "features": { "buildkit": true } }`। फिर Docker को पुनः आरंभ करें।
4. फिर आप निर्माण समय पर `--secret` ध्वज के साथ सीक्रेट्स का उपयोग कर सकते हैं इस तरह:
```bash
docker build --secret my_key=my_value ,src=path/to/my_secret_file .
```
जहां आपकी फ़ाइल आपके रहस्यों को key-value जोड़ी के रूप में निर्दिष्ट करती है।

ये रहस्य इमेज बिल्ड कैश से और अंतिम इमेज से बाहर रखे जाते हैं।

यदि आपको अपने इमेज को बिल्ड करते समय नहीं बल्कि अपने **चल रहे कंटेनर में रहस्य की आवश्यकता है**, तो **Docker Compose या Kubernetes** का उपयोग करें।

Docker Compose के साथ, एक सेवा में secrets key-value जोड़ी को जोड़ें और रहस्य फ़ाइल को निर्दिष्ट करें। [Stack Exchange उत्तर](https://serverfault.com/a/936262/535325) के लिए धन्यवाद जिस पर नीचे दिया गया उदाहरण आधारित है।

रहस्यों के साथ `docker-compose.yml` का उदाहरण:
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
```
फिर सामान्य रूप से Compose को `docker-compose up --build my_service` के साथ शुरू करें।

यदि आप [Kubernetes](https://kubernetes.io/docs/concepts/configuration/secret/) का उपयोग कर रहे हैं, तो इसमें secrets के लिए समर्थन है। [Helm-Secrets](https://github.com/futuresimple/helm-secrets) K8s में secrets प्रबंधन को आसान बना सकता है। इसके अलावा, K8s में Role Based Access Controls (RBAC) है - जैसा कि Docker Enterprise में भी है। RBAC teams के लिए Secrets प्रबंधन को अधिक प्रबंधनीय और सुरक्षित बनाता है।

### gVisor

**gVisor** एक एप्लिकेशन कर्नेल है, जो Go में लिखा गया है, जो Linux सिस्टम सरफेस का एक महत्वपूर्ण हिस्सा लागू करता है। इसमें एक [Open Container Initiative (OCI)](https://www.opencontainers.org) रनटाइम `runsc` शामिल है जो एप्लिकेशन और होस्ट कर्नेल के बीच एक **अलगाव सीमा प्रदान करता है**। `runsc` रनटाइम Docker और Kubernetes के साथ एकीकृत होता है, जिससे सैंडबॉक्स्ड कंटेनर्स को चलाना सरल हो जाता है।

{% embed url="https://github.com/google/gvisor" %}

### Kata Containers

**Kata Containers** एक ओपन सोर्स समुदाय है जो एक सुरक्षित कंटेनर रनटाइम बनाने के लिए काम कर रहा है जिसमें हल्के वर्चुअल मशीनें होती हैं जो कंटेनर्स की तरह महसूस करती हैं और प्रदर्शन करती हैं, लेकिन हार्डवेयर वर्चुअलाइजेशन तकनीक का उपयोग करके दूसरी परत के रूप में **मजबूत कार्यभार अलगाव प्रदान करती हैं**।

{% embed url="https://katacontainers.io/" %}

### Summary Tips

* **`--privileged` फ्लैग का उपयोग न करें या कंटेनर के अंदर** [**Docker socket को माउंट न करें**](https://raesene.github.io/blog/2016/03/06/The-Dangers-Of-Docker.sock/)**।** Docker socket कंटेनर्स को स्पॉन करने की अनुमति देता है, इसलिए यह होस्ट का पूर्ण नियंत्रण लेने का एक आसान तरीका है, उदाहरण के लिए, `--privileged` फ्लैग के साथ दूसरा कंटेनर चलाकर।
* कंटेनर के अंदर **root के रूप में न चलाएं। एक** [**अलग यूजर का उपयोग करें**](https://docs.docker.com/develop/develop-images/dockerfile\_best-practices/#user) **और** [**user namespaces**](https://docs.docker.com/engine/security/userns-remap/)**।** कंटेनर में root होस्ट पर वही होता है जब तक कि user namespaces के साथ रीमैप नहीं किया जाता। यह मुख्य रूप से Linux namespaces, capabilities, और cgroups द्वारा हल्के रूप से प्रतिबंधित होता है।
* [**सभी capabilities को ड्रॉप करें**](https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities) **(`--cap-drop=all`) और केवल उन्हीं को सक्षम करें जो आवश्यक हैं** (`--cap-add=...`). बहुत से कार्यभारों को किसी भी capabilities की आवश्यकता नहीं होती और उन्हें जोड़ने से संभावित हमले की गुंजाइश बढ़ जाती है।
* [**“no-new-privileges” सुरक्षा विकल्प का उपयोग करें**](https://raesene.github.io/blog/2019/06/01/docker-capabilities-and-no-new-privs/) ताकि प्रक्रियाएं अधिक विशेषाधिकार प्राप्त न कर सकें, उदाहरण के लिए suid binaries के माध्यम से।
* [**कंटेनर के लिए उपलब्ध संसाधनों को सीमित करें**](https://docs.docker.com/engine/reference/run/#runtime-constraints-on-resources)**।** संसाधन सीमाएं मशीन को सेवा अस्वीकृति हमलों से बचा सकती हैं।
* **समायोजित करें** [**seccomp**](https://docs.docker.com/engine/security/seccomp/)**,** [**AppArmor**](https://docs.docker.com/engine/security/apparmor/) **(या SELinux)** प्रोफाइल को कंटेनर के लिए आवश्यक न्यूनतम कार्यों और syscalls तक सीमित करने के लिए।
* **उपयोग करें** [**आधिकारिक docker इमेजेस**](https://docs.docker.com/docker-hub/official\_images/) **और हस्ताक्षर की मांग करें** या उनके आधार पर अपनी खुद की बनाएं। [backdoored](https://arstechnica.com/information-technology/2018/06/backdoored-images-downloaded-5-million-times-finally-removed-from-docker-hub/) इमेजेस को विरासत में न लें या उपयोग न करें। साथ ही root keys, passphrase को सुरक्षित स्थान पर रखें। Docker की UCP के साथ keys को प्रबंधित करने की योजना है।
* **नियमित रूप से** अपनी इमेजेस को **पुनः बनाएं ताकि होस्ट और इमेजेस में सुरक्षा पैच लागू किए जा सकें।**
* अपने **secrets को बुद्धिमानी से प्रबंधित करें** ताकि हमलावर के लिए उन तक पहुंचना कठिन हो।
* यदि आप **docker daemon को उजागर करते हैं तो HTTPS का उपयोग करें** क्लाइंट और सर्वर प्रमाणीकरण के साथ।
* अपनी Dockerfile में, **ADD के बजाय COPY को प्राथमिकता दें**। ADD स्वचालित रूप से ज़िप फाइलों को निकालता है और URL से फाइलों की प्रतिलिपि बना सकता है। COPY के ये क्षमताएं नहीं हैं। जब भी संभव हो, ADD का उपयोग न करें ताकि आप रिमोट URLs और Zip फाइलों के माध्यम से हमलों के प्रति संवेदनशील न हों।
* प्रत्येक माइक्रो-सर्विस के लिए **अलग कंटेनर्स रखें**
* कंटेनर के अंदर **ssh न डालें**, “docker exec” का उपयोग करके कंटेनर में ssh किया जा सकता है।
* **छोटी** कंटेनर **इमेजेस रखें**

## Docker Breakout / Privilege Escalation

यदि आप **एक docker कंटेनर के अंदर हैं** या आपके पास **docker समूह में एक उपयोगकर्ता की पहुंच है**, तो आप **बच निकलने और विशेषाधिकार बढ़ाने की कोशिश कर सकते हैं**:

{% content-ref url="docker-breakout-privilege-escalation/" %}
[docker-breakout-privilege-escalation](docker-breakout-privilege-escalation/)
{% endcontent-ref %}

## Docker Authentication Plugin Bypass

यदि आपके पास docker socket तक पहुंच है या आपके पास **docker समूह में एक उपयोगकर्ता की पहुंच है लेकिन आपकी क्रियाओं को docker auth plugin द्वारा सीमित किया जा रहा है**, तो जांचें कि क्या आप **इसे बायपास कर सकते हैं:**

{% content-ref url="authz-and-authn-docker-access-authorization-plugin.md" %}
[authz-and-authn-docker-access-authorization-plugin.md](authz-and-authn-docker-access-authorization-plugin.md)
{% endcontent-ref %}

## Hardening Docker

* उपकरण [**docker-bench-security**](https://github.com/docker/docker-bench-security) एक स्क्रिप्ट है जो प्रोडक्शन में Docker कंटेनर्स को तैनात करने के आसपास की दर्जनों सामान्य बेस्ट-प्रैक्टिसेज की जांच करती है। टेस्ट सभी स्वचालित हैं, और [CIS Docker Benchmark v1.3.1](https://www.cisecurity.org/benchmark/docker/) पर आधारित हैं।\
आपको इस उपकरण को docker चला रहे होस्ट से या पर्याप्त विशेषाधिकारों वाले कंटेनर से चलाना होगा। **कैसे चलाना है इसके लिए README में जानकारी पाएं:** [**https://github.com/docker/docker-bench-security**](https://github.com/docker/docker-bench-security).

## References

* [https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/)
* [https://twitter.com/\_fel1x/status/1151487051986087936](https://twitter.com/\_fel1x/status/1151487051986087936)
* [https://ajxchapman.github.io/containers/2020/11/19/privileged-container-escape.html](https://ajxchapman.github.io/containers/2020/11/19/privileged-container-escape.html)
* [https://sreeninet.wordpress.com/2016/03/06/docker-security-part-1overview/](https://sreeninet.wordpress.com/2016/03/06/docker-security-part-1overview/)
* [https://sreeninet.wordpress.com/2016/03/06/docker-security-part-2docker-engine/](https://sreeninet.wordpress.com/2016/03/06/docker-security-part-2docker-engine/)
* [https://sreeninet.wordpress.com/2016/03/06/docker-security-part-3engine-access/](https://sreeninet.wordpress.com/2016/03/06/docker-security-part-3engine-access/)
* [https://sreeninet.wordpress.com/2016/03/06/docker-security-part-4container-image/](https://sreeninet.wordpress.com/2016/03/06/docker-security-part-4container-image/)
* [https://en.wikipedia.org/wiki/Linux\_namespaces](https://en.wikipedia.org/wiki/Linux\_namespaces)
* [https://t
