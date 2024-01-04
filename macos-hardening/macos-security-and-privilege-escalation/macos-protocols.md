# macOS नेटवर्क सेवाएं और प्रोटोकॉल

<details>

<summary><strong> AWS हैकिंग सीखें शून्य से लेकर हीरो तक </strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) में या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) **का अनुसरण करें**.
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें.

</details>

## रिमोट एक्सेस सेवाएं

ये macOS की सामान्य सेवाएं हैं जिन्हें दूरस्थ रूप से एक्सेस किया जा सकता है।\
आप इन सेवाओं को `System Settings` --> `Sharing` में सक्षम/अक्षम कर सकते हैं

* **VNC**, जिसे “Screen Sharing” के नाम से जाना जाता है (tcp:5900)
* **SSH**, जिसे “Remote Login” कहा जाता है (tcp:22)
* **Apple Remote Desktop** (ARD), या “Remote Management” (tcp:3283, tcp:5900)
* **AppleEvent**, जिसे “Remote Apple Event” के नाम से जाना जाता है (tcp:3031)

यह जांचने के लिए कि कोई भी सक्षम है या नहीं, निम्नलिखित चलाएं:
```bash
rmMgmt=$(netstat -na | grep LISTEN | grep tcp46 | grep "*.3283" | wc -l);
scrShrng=$(netstat -na | grep LISTEN | egrep 'tcp4|tcp6' | grep "*.5900" | wc -l);
flShrng=$(netstat -na | grep LISTEN | egrep 'tcp4|tcp6' | egrep "\*.88|\*.445|\*.548" | wc -l);
rLgn=$(netstat -na | grep LISTEN | egrep 'tcp4|tcp6' | grep "*.22" | wc -l);
rAE=$(netstat -na | grep LISTEN | egrep 'tcp4|tcp6' | grep "*.3031" | wc -l);
bmM=$(netstat -na | grep LISTEN | egrep 'tcp4|tcp6' | grep "*.4488" | wc -l);
printf "\nThe following services are OFF if '0', or ON otherwise:\nScreen Sharing: %s\nFile Sharing: %s\nRemote Login: %s\nRemote Mgmt: %s\nRemote Apple Events: %s\nBack to My Mac: %s\n\n" "$scrShrng" "$flShrng" "$rLgn" "$rmMgmt" "$rAE" "$bmM";
```
### ARD पेंटेस्टिंग

(यह भाग [**इस ब्लॉग पोस्ट से लिया गया है**](https://lockboxx.blogspot.com/2019/07/macos-red-teaming-206-ard-apple-remote.html))

यह मूल रूप से एक विकृत [VNC](https://en.wikipedia.org/wiki/Virtual\_Network\_Computing) है जिसमें कुछ **अतिरिक्त macOS विशिष्ट सुविधाएँ** हैं।\
हालांकि, **स्क्रीन शेयरिंग विकल्प** एक **मूल VNC** सर्वर है। एक उन्नत ARD या रिमोट मैनेजमेंट विकल्प भी है जो **कंट्रोल स्क्रीन पासवर्ड सेट करने** की अनुमति देता है जो ARD को VNC क्लाइंट्स के लिए पीछे की ओर **संगत** बनाता है। हालांकि इस प्रमाणीकरण विधि में एक कमजोरी है जो इस **पासवर्ड** को एक **8 अक्षर ऑथ बफर** तक **सीमित** कर देती है, जिससे इसे [Hydra](https://thudinh.blogspot.com/2017/09/brute-forcing-passwords-with-thc-hydra.html) या [GoRedShell](https://github.com/ahhh/GoRedShell/) जैसे टूल के साथ ब्रूट फोर्स करना बहुत आसान हो जाता है (और मूल रूप से **कोई रेट लिमिट नहीं होती**).\
आप **nmap** का उपयोग करके **स्क्रीन शेयरिंग** या रिमोट मैनेजमेंट के **कमजोर उदाहरणों** की पहचान कर सकते हैं, स्क्रिप्ट `vnc-info` का उपयोग करके, और अगर सेवा `VNC Authentication (2)` का समर्थन करती है तो वे संभवतः **ब्रूट फोर्स के लिए कमजोर** होते हैं। सेवा सभी पासवर्डों को तार पर भेजे जाने वाले 8 अक्षरों तक सीमित कर देगी, इस प्रकार यदि आप VNC प्रमाणीकरण को "password" सेट करते हैं, तो "passwords" और "password123" दोनों प्रमाणित होंगे।

<figure><img src="../../.gitbook/assets/image (9) (3).png" alt=""><figcaption></figcaption></figure>

यदि आप इसे प्रिविलेज एस्केलेशन (TCC प्रॉम्प्ट्स स्वीकार करने), GUI के साथ एक्सेस करने या उपयोगकर्ता की जासूसी करने के लिए सक्षम करना चाहते हैं, तो इसे सक्षम करना संभव है:

{% code overflow="wrap" %}
```bash
sudo /System/Library/CoreServices/RemoteManagement/ARDAgent.app/Contents/Resources/kickstart -activate -configure -allowAccessFor -allUsers -privs -all -clientopts -setmenuextra -menuextra yes
```
{% endcode %}

आप **निरीक्षण** मोड, **साझा नियंत्रण**, और **पूर्ण नियंत्रण** के बीच स्विच कर सकते हैं, जिससे आप एक उपयोगकर्ता पर जासूसी करने से लेकर उनके डेस्कटॉप पर नियंत्रण करने तक एक बटन के क्लिक पर जा सकते हैं। इसके अलावा, यदि आपको ARD सत्र तक पहुँच मिल जाती है, तो वह सत्र समाप्त होने तक खुला रहेगा, भले ही सत्र के दौरान उपयोगकर्ता का पासवर्ड बदल जाए।

आप ARD के जरिए **सीधे unix कमांड्स भेज सकते हैं** और यदि आप एक प्रशासनिक उपयोगकर्ता हैं तो चीजों को रूट के रूप में निष्पादित करने के लिए रूट उपयोगकर्ता को निर्दिष्ट कर सकते हैं। आप इस unix कमांड विधि का उपयोग विशिष्ट समय पर दूरस्थ कार्यों को निर्धारित करने के लिए भी कर सकते हैं, हालांकि यह निर्दिष्ट समय पर एक नेटवर्क कनेक्शन के रूप में होता है (लक्ष्य सर्वर पर संग्रहीत होने और निष्पादित होने के बजाय)। अंत में, दूरस्थ Spotlight मेरी पसंदीदा सुविधाओं में से एक है। यह वास्तव में अच्छा है क्योंकि आप तेजी से और दूरस्थ रूप से कम प्रभाव, सूचीबद्ध खोज कर सकते हैं। यह संवेदनशील फाइलों की खोज के लिए सोना है क्योंकि यह तेज है, आपको एक साथ कई मशीनों पर खोजें चलाने देता है, और CPU को स्पाइक नहीं करेगा।

## Bonjour Protocol

**Bonjour** एक Apple-डिज़ाइन की गई तकनीक है जो कंप्यूटरों और **उपकरणों को एक ही नेटवर्क पर स्थित होने पर अन्य कंप्यूटरों और उपकरणों द्वारा प्रदान की गई सेवाओं के बारे में जानने में सक्षम बनाती है**। इसे इस तरह से डिज़ाइन किया गया है कि कोई भी Bonjour-जागरूक उपकरण को TCP/IP नेटवर्क में प्लग किया जा सकता है और यह **एक IP पता चुनेगा** और उस नेटवर्क पर अन्य कंप्यूटरों को **उसकी प्रदान की गई सेवाओं के बारे में जागरूक करेगा**। Bonjour को कभी-कभी Rendezvous, **Zero Configuration**, या Zeroconf के रूप में संदर्भित किया जाता है।\
Zero Configuration Networking, जैसे कि Bonjour प्रदान करता है:

* एक **IP पता प्राप्त करने में सक्षम** होना चाहिए (भले ही DHCP सर्वर न हो)
* नाम-से-पता अनुवाद करने में सक्षम होना चाहिए (भले ही DNS सर्वर न हो)
* नेटवर्क पर **सेवाओं की खोज करने में सक्षम** होना चाहिए

उपकरण को **169.254/16 रेंज में एक IP पता मिलेगा** और यह जांचेगा कि कोई अन्य उपकरण उस IP पते का उपयोग कर रहा है या नहीं। यदि नहीं, तो वह IP पता रखेगा। Macs इस सबनेट के लिए अपनी रूटिंग टेबल में एक प्रविष्टि रखते हैं: `netstat -rn | grep 169`

DNS के लिए **Multicast DNS (mDNS) प्रोटोकॉल का उपयोग किया जाता है**। [**mDNS** **सेवाएं** पोर्ट **5353/UDP** पर सुनती हैं](../../network-services-pentesting/5353-udp-multicast-dns-mdns.md), **नियमित DNS प्रश्नों** का उपयोग करती हैं और एक IP पते पर अनुरोध भेजने के बजाय **मल्टीकास्ट पते 224.0.0.251** का उपयोग करती हैं। इन अनुरोधों को सुनने वाली कोई भी मशीन प्रतिक्रिया देगी, आमतौर पर एक मल्टीकास्ट पते पर, ताकि सभी उपकरण अपनी तालिकाओं को अपडेट कर सकें।\
प्रत्येक उपकरण नेटवर्क तक पहुँचने पर अपना नाम **चुनेगा**, उपकरण एक नाम चुनेगा जो **.local में समाप्त होता है** (यह होस्टनाम पर आधारित हो सकता है या एक पूरी तरह से यादृच्छिक एक हो सकता है)।

**सेवाओं की खोज के लिए DNS Service Discovery (DNS-SD)** का उपयोग किया जाता है।

Zero Configuration Networking की अंतिम आवश्यकता **DNS Service Discovery (DNS-SD)** द्वारा पूरी की जाती है। DNS Service Discovery DNS SRV रिकॉर्ड्स के सिंटैक्स का उपयोग करती है, लेकिन यदि एक से अधिक होस्ट किसी विशेष सेवा की पेशकश करते हैं तो कई परिणाम वापस करने के लिए **DNS PTR रिकॉर्ड्स का उपयोग करती है**। एक क्लाइंट `<Service>.<Domain>` के नाम के लिए PTR लुकअप का अनुरोध करता है और **शून्य या अधिक PTR रिकॉर्ड्स की एक सूची प्राप्त करता है** जो `<Instance>.<Service>.<Domain>` के रूप में होते हैं।

`dns-sd` बाइनरी का उपयोग **सेवाओं का विज्ञापन करने और सेवाओं के लिए लुकअप करने** के लिए किया जा सकता है:
```bash
#Search ssh services
dns-sd -B _ssh._tcp

Browsing for _ssh._tcp
DATE: ---Tue 27 Jul 2021---
12:23:20.361  ...STARTING...
Timestamp     A/R    Flags  if Domain               Service Type         Instance Name
12:23:20.362  Add        3   1 local.               _ssh._tcp.           M-C02C934RMD6R
12:23:20.362  Add        3  10 local.               _ssh._tcp.           M-C02C934RMD6R
12:23:20.362  Add        2  16 local.               _ssh._tcp.           M-C02C934RMD6R
```

```bash
#Announce HTTP service
dns-sd -R "Index" _http._tcp . 80 path=/index.html

#Search HTTP services
dns-sd -B _http._tcp
```
जब एक नई सेवा शुरू होती है, तो **नई सेवा अपनी उपस्थिति सबनेट पर सभी को मल्टीकास्ट करती है**। श्रोता को पूछना नहीं पड़ा; उसे केवल सुनना था।

आप अपने वर्तमान स्थानीय नेटवर्क में **प्रस्तावित सेवाओं** को देखने के लिए [**इस उपकरण**](https://apps.apple.com/us/app/discovery-dns-sd-browser/id1381004916?mt=12) का उपयोग कर सकते हैं।\
या आप [**python-zeroconf**](https://github.com/jstasiak/python-zeroconf) के साथ अपनी खुद की स्क्रिप्ट्स पायथन में लिख सकते हैं:
```python
from zeroconf import ServiceBrowser, Zeroconf


class MyListener:

def remove_service(self, zeroconf, type, name):
print("Service %s removed" % (name,))

def add_service(self, zeroconf, type, name):
info = zeroconf.get_service_info(type, name)
print("Service %s added, service info: %s" % (name, info))


zeroconf = Zeroconf()
listener = MyListener()
browser = ServiceBrowser(zeroconf, "_http._tcp.local.", listener)
try:
input("Press enter to exit...\n\n")
finally:
zeroconf.close()
```
यदि आपको लगता है कि Bonjour **disabled** होने पर अधिक सुरक्षित हो सकता है, तो आप इसे निम्नलिखित के साथ कर सकते हैं:
```bash
sudo launchctl unload -w /System/Library/LaunchDaemons/com.apple.mDNSResponder.plist
```
## संदर्भ

* [**The Mac Hacker's Handbook**](https://www.amazon.com/-/es/Charlie-Miller-ebook-dp-B004U7MUMU/dp/B004U7MUMU/ref=mt\_other?\_encoding=UTF8\&me=\&qid=)
* [**https://taomm.org/vol1/analysis.html**](https://taomm.org/vol1/analysis.html)
* [**https://lockboxx.blogspot.com/2019/07/macos-red-teaming-206-ard-apple-remote.html**](https://lockboxx.blogspot.com/2019/07/macos-red-teaming-206-ard-apple-remote.html)

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert) के साथ शून्य से नायक तक AWS हैकिंग सीखें</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) में शामिल हों या मुझे **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) पर **फॉलो** करें।
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>
