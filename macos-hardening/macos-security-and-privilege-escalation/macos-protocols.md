# macOS नेटवर्क सेवाएं और प्रोटोकॉल

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप एक **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करना चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह
* प्राप्त करें [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **फॉलो** करें मुझे **ट्विटर** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को।**

</details>

## रिमोट एक्सेस सेवाएं

ये मानक macOS सेवाएं हैं जिनका रिमोट एक्सेस किया जा सकता है।\
आप इन सेवाओं को `सिस्टम सेटिंग्स` --> `शेयरिंग` में सक्षम/अक्षम कर सकते हैं

* **VNC**, जिसे "स्क्रीन शेयरिंग" कहा जाता है (tcp:5900)
* **SSH**, जिसे "रिमोट लॉगिन" कहा जाता है (tcp:22)
* **Apple Remote Desktop** (ARD), या "रिमोट मैनेजमेंट" (tcp:3283, tcp:5900)
* **AppleEvent**, जिसे "रिमोट Apple Event" कहा जाता है (tcp:3031)

चेक करें कि क्या कोई सेवा सक्षम है या नहीं इसके लिए निम्नलिखित को चलाएं:
```bash
rmMgmt=$(netstat -na | grep LISTEN | grep tcp46 | grep "*.3283" | wc -l);
scrShrng=$(netstat -na | grep LISTEN | egrep 'tcp4|tcp6' | grep "*.5900" | wc -l);
flShrng=$(netstat -na | grep LISTEN | egrep 'tcp4|tcp6' | egrep "\*.88|\*.445|\*.548" | wc -l);
rLgn=$(netstat -na | grep LISTEN | egrep 'tcp4|tcp6' | grep "*.22" | wc -l);
rAE=$(netstat -na | grep LISTEN | egrep 'tcp4|tcp6' | grep "*.3031" | wc -l);
bmM=$(netstat -na | grep LISTEN | egrep 'tcp4|tcp6' | grep "*.4488" | wc -l);
printf "\nThe following services are OFF if '0', or ON otherwise:\nScreen Sharing: %s\nFile Sharing: %s\nRemote Login: %s\nRemote Mgmt: %s\nRemote Apple Events: %s\nBack to My Mac: %s\n\n" "$scrShrng" "$flShrng" "$rLgn" "$rmMgmt" "$rAE" "$bmM";
```
### पेंटेस्टिंग ARD

(यह भाग [**इस ब्लॉग पोस्ट से लिया गया था**](https://lockboxx.blogspot.com/2019/07/macos-red-teaming-206-ard-apple-remote.html))

यह मूल रूप से कुछ **अतिरिक्त macOS विशेषताओं** के साथ एक बदला हुआ [VNC](https://en.wikipedia.org/wiki/Virtual\_Network\_Computing) है।\
हालांकि, **स्क्रीन शेयरिंग विकल्प** बस एक **बेसिक VNC** सर्वर है। इसके अलावा, एक उन्नत ARD या रिमोट मैनेजमेंट विकल्प है जिसमें आप एक नियंत्रण स्क्रीन पासवर्ड सेट कर सकते हैं जो ARD को VNC क्लाइंट के लिए **संगत बना देगा**। हालांकि, इस प्रमाणीकरण विधि के एक कमजोरी है जो इस **पासवर्ड** को एक **8 अक्षर ऑथ बफर** तक सीमित करती है, जिससे इसे [Hydra](https://thudinh.blogspot.com/2017/09/brute-forcing-passwords-with-thc-hydra.html) या [GoRedShell](https://github.com/ahhh/GoRedShell/) जैसे उपकरणों के साथ बहुत आसानी से **ब्रूट फोर्स** किया जा सकता है (डिफ़ॉल्ट रेट सीमाएं भी नहीं हैं)।\
आप **स्क्रीन शेयरिंग के संक्रमित उदाहरण** या रिमोट मैनेजमेंट को **nmap** का उपयोग करके खोज सकते हैं, स्क्रिप्ट `vnc-info` का उपयोग करके, और यदि सेवा `VNC प्रमाणीकरण (2)` का समर्थन करती है तो वे ब्रूट फोर्स के लिए **संक्रमित होने की संभावना** हैं। सेवा सभी पासवर्ड को 8 अक्षरों तक काट देगी, इसलिए यदि आप VNC प्रमाणीकरण को "पासवर्ड" सेट करते हैं, तो "पासवर्ड" और "पासवर्ड123" दोनों प्रमाणित होंगे।

<figure><img src="../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

यदि आप इसे उच्चतम अधिकार प्राप्त करने के लिए सक्षम करना चाहते हैं (TCC प्रॉम्प्ट को स्वीकार करें), GUI के साथ उपयोग करें या उपयोगकर्ता का जासूसी करें, तो इसे सक्षम करना संभव है:

{% code overflow="wrap" %}
```bash
sudo /System/Library/CoreServices/RemoteManagement/ARDAgent.app/Contents/Resources/kickstart -activate -configure -allowAccessFor -allUsers -privs -all -clientopts -setmenuextra -menuextra yes
```
{% endcode %}

आप बटन के द्वारा उपयोगकर्ता की जांच करने से उपयोगकर्ता के डेस्कटॉप पर कब्जा करने जैसे विभिन्न मोड में स्विच कर सकते हैं। इसके अलावा, यदि आपको ARD सत्र का पहुंच मिल जाता है, तो वह सत्र सत्र समाप्त होने तक खुला रहेगा, यहां तक कि सत्र के दौरान उपयोगकर्ता का पासवर्ड बदल दिया जाता है।

आप ARD के माध्यम से **यूनिक्स कमांड सीधे भेज सकते हैं** और यदि आप प्रशासनिक उपयोगकर्ता हैं तो रूट उपयोगकर्ता के रूप में चीजों को निष्पादित करने के लिए रूट उपयोगकर्ता को निर्दिष्ट कर सकते हैं। आप इस यूनिक्स कमांड विधि का उपयोग करके दूरस्थ कार्यों को निर्धारित समय पर चलाने के लिए भी इस्तेमाल कर सकते हैं, हालांकि यह निर्दिष्ट समय पर एक नेटवर्क कनेक्शन के रूप में होता है (टारगेट सर्वर पर संग्रहीत और निष्पादित नहीं होता है)। अंत में, दूरस्थ स्पॉटलाइट मेरी पसंदीदा सुविधाओं में से एक है। यह वास्तव में शानदार है क्योंकि आप तेजी से और दूरस्थ से एक कम प्रभाव वाली इंडेक्स सर्च चला सकते हैं। यह संवेदनशील फ़ाइलों की खोज के लिए सोने की अवस्था है क्योंकि यह तेज है, यह आपको एकाधिक मशीनों पर समयानुसार खोज चलाने देता है और CPU को बढ़ाने की आवश्यकता नहीं होती है।

## Bonjour Protocol

**Bonjour** एक Apple द्वारा डिज़ाइन की गई तकनीक है जो कंप्यूटर और **उपकरणों को सीधे नेटवर्क पर सेवाएं सीखने की सुविधा प्रदान करती है**। इसका उद्देश्य यह है कि कोई भी Bonjour-जागरूक उपकरण TCP/IP नेटवर्क में प्लग इन किया जा सकता है और यह एक IP पता **चुनेगा** और उस नेटवर्क पर अन्य कंप्यूटरों को अपनी सेवाओं के बारे में **जागरूक करेगा**। Bonjour कभी-कभी Rendezvous, **Zero Configuration** या Zeroconf के रूप में भी उल्लेख किया जाता है।\
Zero Configuration Networking, जैसे Bonjour प्रदान करता है:

* एक IP पता **प्राप्त कर सकना चाहिए** (यहां तक कि DHCP सर्वर के बिना भी)
* **नाम-से-पता अनुवाद** कर सकना चाहिए (यहां तक कि DNS सर्वर के बिना भी)
* नेटवर्क पर सेवाओं की **खोज कर सकना चाहिए**

उपकरण को **169.254/16** रेंज में एक **IP पता मिलेगा** और यह जांचेगा कि क्या कोई अन्य उपकरण उस IP पते का उपयोग कर रहा है। यदि नहीं, तो यह IP पता रखेगा। Macs इस सबनेट के लिए अपने रूटिंग टेबल में एक प्रविष्टि रखते हैं: `netstat -rn | grep 169`

DNS के लिए **मल्टीकास्ट DNS (mDNS) प्रोटोकॉल का उपयोग किया जाता है**। [**mDNS सेवाएं** **5353/UDP** पोर्ट पर सुनती हैं](../../network-services-pentesting/5353-udp-multicast-dns-mdns.md), **नियमित DNS क्वेरी** का उपयोग करती हैं और इसके बजाय अनुरोध को केवल एक IP पते को भेजने के लिए **मल्टीकास्ट पते 224.0.0.251** का उपयोग करती है। इन अनुरोधों को सुनने वाली कोई भी मशीन उत्तर देगी, आमतौर पर एक मल्टीकास्ट पते को, ताकि सभी उपकरण अपनी टेबल को अपडेट कर सकें।\
प्रत्येक उपकरण नेटवर्क तक पहुंचते समय अपना खुद का नाम **चुनेगा**, उपकरण एक नाम **.local से समाप्त होने वाला** चुनेगा (यह होस्टनाम पर आधारित हो सकता है या पूरी तरह से यादृच्छिक हो सकता है)।

**सेवाओं की खोज के लिए DNS सेवा खोज (DNS-SD)** का उपयोग किया जाता है।

Zero Configuration Networking की अंतिम आवश्यकता **DNS सेवा खोज (DNS-SD)** द्वारा पूरी की जाती है। DNS सेवा खोज DNS SRV रिकॉर्डों से वाक्यांश का उपयोग करती है, लेकिन एकाधिक परिणाम लौटाए जा सकते हैं यदि एक से अधिक होस्ट एक विशेष सेवा प्रदान करता है। एक ग्राहक नाम `<सेवा>.<डोमेन>` के लिए PTR लुकअप का अनुरोध करता है और **एक सूची** शून्य या अधिक PTR रिकॉर्ड प्राप्त करता है जिनका प्रारूप `<इंस्टेंस>.<सेवा>.<डोमेन>` होता है।

`dns-sd` बाइनरी का उपयोग सेवाओं का **विज्ञापन करने और खोज करने** के लिए किया जा सकता है।
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
जब एक नई सेवा शुरू होती है, तो **नई सेवा सभी** सबनेट पर मौजूद लोगों को अपनी मौजूदगी के बारे में मल्टीकास्ट करती है। सुनने वाले को पूछने की आवश्यकता नहीं थी; उसे सिर्फ सुनना ही था।

आप अपने मौजूदा स्थानीय नेटवर्क में **प्रदान की जाने वाली सेवाओं** को देखने के लिए [**इस उपकरण**](https://apps.apple.com/us/app/discovery-dns-sd-browser/id1381004916?mt=12) का उपयोग कर सकते हैं।
या आप [**python-zeroconf**](https://github.com/jstasiak/python-zeroconf) के साथ पायथन में अपने खुद के स्क्रिप्ट लिख सकते हैं:
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
यदि आपको ऐसा लगता है कि बोनजोर अधिक सुरक्षित हो सकता है तो आप इसे निष्क्रिय कर सकते हैं:
```bash
sudo launchctl unload -w /System/Library/LaunchDaemons/com.apple.mDNSResponder.plist
```
## संदर्भ

* [**The Mac Hacker's Handbook**](https://www.amazon.com/-/es/Charlie-Miller-ebook-dp-B004U7MUMU/dp/B004U7MUMU/ref=mt\_other?\_encoding=UTF8\&me=\&qid=)
* [**https://taomm.org/vol1/analysis.html**](https://taomm.org/vol1/analysis.html)
* [**https://lockboxx.blogspot.com/2019/07/macos-red-teaming-206-ard-apple-remote.html**](https://lockboxx.blogspot.com/2019/07/macos-red-teaming-206-ard-apple-remote.html)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करना चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें और PRs सबमिट करें** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को**।

</details>
