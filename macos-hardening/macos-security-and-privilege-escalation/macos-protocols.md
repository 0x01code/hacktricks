# macOS नेटवर्क सेवाएं और प्रोटोकॉल

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert) के साथ</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** द्वारा PRs सबमिट करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

## रिमोट एक्सेस सेवाएं

ये मैकओएस सेवाएं हैं जिन्हें दूरस्थ से एक्सेस किया जा सकता है।\
आप इन सेवाओं को `सिस्टम सेटिंग्स` --> `शेयरिंग` में सक्षम/अक्षम कर सकते हैं

* **VNC**, जिसे "स्क्रीन शेयरिंग" कहा जाता है (tcp:5900)
* **SSH**, "रिमोट लॉगिन" के रूप में जाना जाता है (tcp:22)
* **Apple Remote Desktop** (ARD), या "रिमोट मैनेजमेंट" (tcp:3283, tcp:5900)
* **AppleEvent**, जिसे "रिमोट Apple इवेंट" कहा जाता है (tcp:3031)

किसी भी सेवा को सक्षम है या नहीं, यह देखने के लिए निम्नलिखित को चलाएं:
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

एप्पल रिमोट डेस्कटॉप (ARD) वर्चुअल नेटवर्क कंप्यूटिंग (VNC) का एक एन्हांस्ड संस्करण है जो macOS के लिए तैयार किया गया है, जो अतिरिक्त विशेषताएं प्रदान करता है। ARD में एक प्रमुख दुर्बलता है इसका नियंत्रण स्क्रीन पासवर्ड के लिए प्रमाणीकरण विधि, जो केवल पासवर्ड के पहले 8 वर्णों का उपयोग करता है, जिससे यह [ब्रूट फोर्स हमलों](https://thudinh.blogspot.com/2017/09/brute-forcing-passwords-with-thc-hydra.html) के लिए संवेदनशील है, जैसे कि हाइड्रा या [गोरेडशेल](https://github.com/ahhh/GoRedShell/) जैसे उपकरणों के साथ, क्योंकि कोई डिफ़ॉल्ट दर सीमाएं नहीं हैं।

विकल्पनीय उदाहरणों को **nmap** के `vnc-info` स्क्रिप्ट का उपयोग करके पहचाना जा सकता है। `VNC Authentication (2)` का समर्थन करने वाली सेवाएं विशेष रूप से 8 वर्णों के पासवर्ड के काटने के कारण ब्रूट फोर्स हमलों के लिए अत्यधिक संवेदनशील हैं।

विभिन्न प्रशासनिक कार्यों जैसे विशेषाधिकार उन्नयन, GUI एक्सेस, या उपयोगकर्ता मॉनिटरिंग के लिए ARD को सक्षम करने के लिए निम्नलिखित कमांड का उपयोग करें:
```bash
sudo /System/Library/CoreServices/RemoteManagement/ARDAgent.app/Contents/Resources/kickstart -activate -configure -allowAccessFor -allUsers -privs -all -clientopts -setmenuextra -menuextra yes
```
## बोनजोर प्रोटोकॉल

बोनजोर, एक Apple डिज़ाइन की तकनीक, **एक ही नेटवर्क पर डिवाइसेस को एक-दूसरे की पेशकश की गई सेवाओं का पता लगाने** की अनुमति देता है। जिसे Rendezvous, **जीरो कॉन्फ़िगरेशन**, या Zeroconf भी कहा जाता है, यह एक डिवाइस को एक TCP/IP नेटवर्क में शामिल होने की अनुमति देता है, **स्वचालित रूप से एक IP पता चुनने** और अपनी सेवाओं को अन्य नेटवर्क डिवाइसेस को प्रसारित करने की।

जीरो कॉन्फ़िगरेशन नेटवर्किंग, जो बोनजोर द्वारा प्रदान की जाती है, सुनिश्चित करता है कि डिवाइसेस:
* **एक IP पता स्वचालित रूप से प्राप्त करें** भले ही एक DHCP सर्वर की अनुपस्थिति में।
* **नाम-से-पता अनुवाद** करें बिना DNS सर्वर की आवश्यकता के।
* नेटवर्क पर **उपलब्ध सेवाओं की खोज** करें।

Bonjour का उपयोग करने वाले डिवाइसेस खुद को **169.254/16 रेंज से एक IP पता सौंपेंगे** और नेटवर्क पर इसकी अद्वितीयता की जांच करेंगे। Macs इस सबनेट के लिए एक रूटिंग टेबल एंट्री बनाए रखते हैं, जिसे `netstat -rn | grep 169` के माध्यम से सत्यापित किया जा सकता है।

DNS के लिए, Bonjour मल्टीकास्ट DNS (mDNS) प्रोटोकॉल का उपयोग करता है। mDNS **पोर्ट 5353/UDP** पर कार्य करता है, **मानक DNS क्वेरी** का उपयोग करता है लेकिन **मल्टीकास्ट पता 224.0.0.251** को लक्ष्य बनाता है। यह दृष्टिकोण सुनिश्चित करता है कि नेटवर्क पर सभी सुनने वाले डिवाइसेस क्वेरी को प्राप्त करें और उसके जवाब दें, अपने रिकॉर्ड को अपडेट करने की सुविधा प्रदान करता है।

नेटवर्क में खोज की सुविधा को **DNS सेवा खोज (DNS-SD)** द्वारा सुविधाजनक बनाया गया है। DNS SRV रिकॉर्डों के प्रारूप का उपयोग करते हुए, DNS-SD **DNS PTR रिकॉर्ड** का उपयोग करता है एकाधिक सेवाओं की सूची बनाने के लिए। एक विशेष सेवा की खोज करने वाला ग्राहक `<सेवा>.<डोमेन>` के लिए एक PTR रिकॉर्ड का अनुरोध करेगा, और यदि सेवा कई होस्ट से उपलब्ध है तो `<इंस्टेंस>.<सेवा>.<डोमेन>` के प्रारूप में PTR रिकॉर्ड की सूची प्राप्त करेगा।

`dns-sd` यूटिलिटी का उपयोग **नेटवर्क सेवाओं की खोज और विज्ञापन** के लिए किया जा सकता है। यहाँ इसके उपयोग के कुछ उदाहरण हैं:

### SSH सेवाओं की खोज

नेटवर्क पर SSH सेवाओं की खोज करने के लिए, निम्नलिखित कमांड का उपयोग किया जाता है:
```bash
dns-sd -B _ssh._tcp
```
यह कमांड _ssh._tcp सेवाओं के लिए ब्राउज़िंग आरंभ करता है और समयचिह्न, फ्लैग, इंटरफेस, डोमेन, सेवा प्रकार, और इंस्टेंस नाम जैसी विवरण उत्पन्न करता है।

### एक HTTP सेवा का विज्ञापन करना

एक HTTP सेवा का विज्ञापन करने के लिए, आप इस्तेमाल कर सकते हैं:
```bash
dns-sd -R "Index" _http._tcp . 80 path=/index.html
```
यह कमांड पोर्ट 80 पर "/index.html" पथ के साथ "Index" नामक एक HTTP सेवा को रजिस्टर करता है।

फिर नेटवर्क पर HTTP सेवाओं की खोज करने के लिए:
```bash
dns-sd -B _http._tcp
```
जब एक सेवा शुरू होती है, तो यह अपनी उपस्थिति को सभी उपनेट के उपकरणों को मल्टीकास्ट करके घोषित करती है। इन सेवाओं में रुचि रखने वाले उपकरण अनुरोध भेजने की आवश्यकता नहीं है, बल्कि इन घोषणाओं के लिए सिर्फ सुनते रहते हैं।

एक और उपयोगकर्ता-मित्र सामग्री के लिए, **Discovery - DNS-SD Browser** ऐप जो Apple App Store पर उपलब्ध है, आपके स्थानीय नेटवर्क पर पेश की जाने वाली सेवाएं दृश्यात्मक रूप से प्रदर्शित कर सकता है।

वैकल्पिक रूप से, `python-zeroconf` पुस्तकालय का उपयोग करके सेवाओं को ब्राउज़ और खोजने के लिए कस्टम स्क्रिप्ट लिखा जा सकता है। [**python-zeroconf**](https://github.com/jstasiak/python-zeroconf) स्क्रिप्ट _http._tcp.local. सेवाओं के लिए एक सेवा ब्राउज़र बनाने का प्रदर्शन करता है, जो जोड़ी गई या हटाई गई सेवाएं प्रिंट करता है:
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
### बोनजोर को अक्षम करना
यदि सुरक्षा या अन्य कारणों से बोनजोर को अक्षम करने की चिंता है, तो निम्नलिखित कमांड का उपयोग करके इसे बंद किया जा सकता है:
```bash
sudo launchctl unload -w /System/Library/LaunchDaemons/com.apple.mDNSResponder.plist
```
## संदर्भ

* [**The Mac Hacker's Handbook**](https://www.amazon.com/-/es/Charlie-Miller-ebook-dp-B004U7MUMU/dp/B004U7MUMU/ref=mt\_other?\_encoding=UTF8\&me=\&qid=)
* [**https://taomm.org/vol1/analysis.html**](https://taomm.org/vol1/analysis.html)
* [**https://lockboxx.blogspot.com/2019/07/macos-red-teaming-206-ard-apple-remote.html**](https://lockboxx.blogspot.com/2019/07/macos-red-teaming-206-ard-apple-remote.html)

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **जुड़ें** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके।

</details>
