# बाहरी जासूसी मेथडोलॉजी

<details>

<summary><strong>जीरो से हीरो तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** द्वारा PRs सबमिट करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

अगर आप **हैकिंग करियर** में रुचि रखते हैं और अनहैकेबल को हैक करना चाहते हैं - **हम नियुक्ति कर रहे हैं!** (_फ्लूएंट पोलिश लिखित और बोली जानी चाहिए_).

{% embed url="https://www.stmcyber.com/careers" %}

## संपत्तियों का खोज

> तो आपको कहा गया था कि किसी कंपनी के सभी वस्तुएं स्कोप में हैं, और आपको यह जानना है कि यह कंपनी वास्तव में क्या स्वामित्व रखती है।

इस चरण का उद्देश्य है कि प्राप्त करें **मुख्य कंपनी द्वारा स्वामित्वित कंपनियों** और फिर इन कंपनियों की **सभी संपत्तियां**। इसे करने के लिए हमें करना होगा:

1. मुख्य कंपनी की अधिग्रहणों का पता लगाना, यह हमें स्कोप में आने वाली कंपनियों देगा।
2. प्रत्येक कंपनी का ASN (यदि कोई हो) खोजना, यह हमें प्रत्येक कंपनी द्वारा स्वामित्वित IP रेंज देगा।
3. पहले वाले से संबंधित अन्य एंट्री (संगठन के नाम, डोमेन आदि) के लिए रिवर्स whois लुकअप का उपयोग करें (यह पुनरावृत्ति किया जा सकता है)।
4. अन्य तकनीकों का उपयोग करें जैसे shodan `org` और `ssl` फ़िल्टर्स अन्य संपत्तियों के लिए खोजने के लिए (`ssl` ट्रिक को पुनरावृत्ति किया जा सकता है)।

### **अधिग्रहण**

सबसे पहले, हमें जानना होगा कि **मुख्य कंपनी द्वारा किसी अन्य कंपनियों का स्वामित्व** है।\
एक विकल्प है [https://www.crunchbase.com/](https://www.crunchbase.com) पर जाना, **मुख्य कंपनी** के लिए **खोज** करना, और "**अधिग्रहण**" पर **क्लिक** करना। वहां आपको मुख्य कंपनी द्वारा अधिग्रहित अन्य कंपनियों का पता चलेगा।\
दूसरा विकल्प है मुख्य कंपनी के **विकिपीडिया** पृष्ठ पर जाना और **अधिग्रहण** के लिए खोजना।

> ठीक है, इस बिंदु पर आपको स्कोप में आने वाली सभी कंपनियों का पता होना चाहिए। चलिए उनकी संपत्तियों को कैसे खोजें, इसे समझते हैं।

### **ASNs**

एक स्वतंत्र प्रणाली संख्या (**ASN**) एक **अटोनोमस सिस्टम** (AS) को **इंटरनेट निर्धारित संख्या प्राधिकरण (IANA)** द्वारा एक **अद्वितीय संख्या** के रूप में सौंपी जाती है।\
एक **AS** में **ब्लॉक** होते हैं **IP पतों** के जिनके लिए बाहरी नेटवर्कों तक पहुंचने के लिए एक स्पष्ट निर्धारित नीति होती है और एक ही संगठन द्वारा प्रबंधित होते हैं लेकिन इनमें कई ऑपरेटर्स हो सकते हैं।

यह दिलचस्प होगा कि क्या **कंपनी ने किसी ASN को सौंपा है** ताकि इसके **IP रेंज** पता लगाया जा सके। स्कोप के सभी **होस्ट** के खिलाफ एक **सुरक्षा जांच** करना दिलचस्प होगा और इन IP में **डोमेन** की खोज करनी होगी।\
आप [**https://bgp.he.net/**](https://bgp.he.net) में कंपनी के **नाम**, **IP** या **डोमेन** से खोज कर सकते हैं।\
**कंपनी के क्षेत्र के आधार पर यह लिंक अधिक डेटा जुटाने के लिए उपयोगी हो सकते हैं:** [**AFRINIC**](https://www.afrinic.net) **(अफ्रीका),** [**Arin**](https://www.arin.net/about/welcome/region/)**(उत्तर अमेरिका),** [**APNIC**](https://www.apnic.net) **(एशिया),** [**LACNIC**](https://www.lacnic.net) **(लैटिन अमेरिका),** [**RIPE NCC**](https://www.ripe.net) **(यूरोप)। जैसे-जैसे, संभावित है कि सभी** उपयोगी जानकारी **(IP रेंज और Whois)** पहले लिंक में ही दिखाई दे रही हो।
```bash
#You can try "automate" this with amass, but it's not very recommended
amass intel -org tesla
amass intel -asn 8911,50313,394161
```
इसके अलावा, [**BBOT**](https://github.com/blacklanternsecurity/bbot)**'s** सबडोमेन जांच स्वचालित रूप से स्कैन के अंत में ASNs को संकलित और सारांशित करता है।
```bash
bbot -t tesla.com -f subdomain-enum
...
[INFO] bbot.modules.asn: +----------+---------------------+--------------+----------------+----------------------------+-----------+
[INFO] bbot.modules.asn: | AS394161 | 8.244.131.0/24      | 5            | TESLA          | Tesla Motors, Inc.         | US        |
[INFO] bbot.modules.asn: +----------+---------------------+--------------+----------------+----------------------------+-----------+
[INFO] bbot.modules.asn: | AS16509  | 54.148.0.0/15       | 4            | AMAZON-02      | Amazon.com, Inc.           | US        |
[INFO] bbot.modules.asn: +----------+---------------------+--------------+----------------+----------------------------+-----------+
[INFO] bbot.modules.asn: | AS394161 | 8.45.124.0/24       | 3            | TESLA          | Tesla Motors, Inc.         | US        |
[INFO] bbot.modules.asn: +----------+---------------------+--------------+----------------+----------------------------+-----------+
[INFO] bbot.modules.asn: | AS3356   | 8.32.0.0/12         | 1            | LEVEL3         | Level 3 Parent, LLC        | US        |
[INFO] bbot.modules.asn: +----------+---------------------+--------------+----------------+----------------------------+-----------+
[INFO] bbot.modules.asn: | AS3356   | 8.0.0.0/9           | 1            | LEVEL3         | Level 3 Parent, LLC        | US        |
[INFO] bbot.modules.asn: +----------+---------------------+--------------+----------------+----------------------------+-----------+

```
आप एक संगठन के IP रेंज भी [http://asnlookup.com/](http://asnlookup.com) का उपयोग करके पा सकते हैं (इसमें मुफ्त API है)।\
आप एक डोमेन का IP और ASN [http://ipv4info.com/](http://ipv4info.com) का उपयोग करके पा सकते हैं।

### **वंरबिलिटीज़ खोजना**

इस बिंदु पर हमें **दायरा में सभी संपत्तियों का पता चल गया है**, इसलिए अगर आपको अनुमति है तो आप कुछ **वंरबिलिटी स्कैनर** (Nessus, OpenVAS) को सभी होस्ट पर लॉन्च कर सकते हैं।\
इसके अलावा, आप कुछ [**पोर्ट स्कैन**](../pentesting-network/#discovering-hosts-from-the-outside) **भी लॉन्च कर सकते हैं या** shodan **जैसी सेवाओं का उपयोग करके** खुले पोर्ट्स **खोज सकते हैं और आपको मिलने वाली जानकारी के आधार पर** इस पुस्तक में कैसे कुछ संभावित सेवाओं का पेंटेस्ट करना है उसे **देखना चाहिए।\
**इसके अलावा, यह उल्लेखनीय हो सकता है कि आप कुछ** डिफ़ॉल्ट उपयोगकर्ता नाम **और** पासवर्ड **सूचियाँ भी तैयार कर सकते हैं और [https://github.com/x90skysn3k/brutespray](https://github.com/x90skysn3k/brutespray) का उपयोग करके सेवाओं को** ब्रूटफ़ोर्स **करने की कोशिश कर सकते हैं।

## डोमेन

> हमें दायरा में सभी कंपनियों और उनकी संपत्तियों का पता चल गया है, अब हमें दायरा में डोमेन्स ढूंढने का समय है।

_कृपया ध्यान दें कि निम्नलिखित प्रस्तावित तकनीकों में आप सबडोमेन्स भी पा सकते हैं और उस जानकारी को कम महत्व न दें।_

सबसे पहले आपको प्रत्येक कंपनी के **मुख्य डोमेन**(s) की खोज करनी चाहिए। उदाहरण के लिए, _Tesla Inc._ के लिए _tesla.com_ होगा।

### **रिवर्स DNS**

जैसे ही आपने सभी डोमेनों के IP रेंज पाए हैं, आप उन IP पर **रिवर्स DNS लुकअप्स** करने की कोशिश कर सकते हैं ताकि दायरा में और डोमेन पाए जा सकें। पीडीएन्स के कुछ डीएनएस सर्वर या किसी पीडीएनएस सर्वर (1.1.1.1, 8.8.8.8) का उपयोग करने की कोशिश करें।
```bash
dnsrecon -r <DNS Range> -n <IP_DNS>   #DNS reverse of all of the addresses
dnsrecon -d facebook.com -r 157.240.221.35/24 #Using facebooks dns
dnsrecon -r 157.240.221.35/24 -n 1.1.1.1 #Using cloudflares dns
dnsrecon -r 157.240.221.35/24 -n 8.8.8.8 #Using google dns
```
### **पलटें व्हॉइस (लूप)**

एक **व्हॉइस** में आपको कई दिलचस्प **जानकारी** मिल सकती है जैसे **संगठन का नाम**, **पता**, **ईमेल**, फोन नंबर... लेकिन जो और भी दिलचस्प है वह यह है कि आप कंपनी से संबंधित **अधिक संपत्तियाँ** खोज सकते हैं अगर आप **उन क्षेत्रों में से किसी भी क्षेत्र के लिए पलटें व्हॉइस लुकअप** करते हैं (उदाहरण के लिए जहां एक ही ईमेल प्रकट होता है वहां अन्य व्हॉइस रजिस्ट्रीज जहां यह ईमेल प्रकट होता है)।\
आप ऑनलाइन उपकरणों का उपयोग कर सकते हैं जैसे:

* [https://viewdns.info/reversewhois/](https://viewdns.info/reversewhois/) - **मुफ्त**
* [https://domaineye.com/reverse-whois](https://domaineye.com/reverse-whois) - **मुफ्त**
* [https://www.reversewhois.io/](https://www.reversewhois.io) - **मुफ्त**
* [https://www.whoxy.com/](https://www.whoxy.com) - **मुफ्त** वेब, मुफ्त एपीआई नहीं।
* [http://reversewhois.domaintools.com/](http://reversewhois.domaintools.com) - मुफ्त नहीं
* [https://drs.whoisxmlapi.com/reverse-whois-search](https://drs.whoisxmlapi.com/reverse-whois-search) - मुफ्त नहीं (केवल **100 मुफ्त** खोजें)
* [https://www.domainiq.com/](https://www.domainiq.com) - मुफ्त नहीं

आप इस कार्य को [**DomLink** ](https://github.com/vysecurity/DomLink)(जिसके लिए एक whoxy एपीआई कुंजी की आवश्यकता है) का उपयोग करके स्वचालित कर सकते हैं।\
आप [amass](https://github.com/OWASP/Amass) के साथ कुछ स्वचालित पलटें व्हॉइस डिस्कवरी भी कर सकते हैं: `amass intel -d tesla.com -whois`

**ध्यान दें कि आप इस तकनीक का उपयोग करके हर बार जब आप एक नया डोमेन पाते हैं तो अधिक डोमेन नामों की खोज करने के लिए इस तकनीक का उपयोग कर सकते हैं।**

### **ट्रैकर्स**

अगर आप 2 विभिन्न पृष्ठों में **एक ही ट्रैकर की एक ही आईडी** पाते हैं तो आप समझ सकते हैं कि **दोनों पृष्ठ** **एक ही टीम द्वारा प्रबंधित** हैं।\
उदाहरण के लिए, अगर आप कई पृष्ठों पर **एक ही Google Analytics आईडी** या **एक ही Adsense आईडी** देखते हैं।

कुछ पृष्ठ और उपकरण हैं जो आपको इन ट्रैकर्स और अधिक द्वारा खोजने देते हैं:

* [**Udon**](https://github.com/dhn/udon)
* [**BuiltWith**](https://builtwith.com)
* [**Sitesleuth**](https://www.sitesleuth.io)
* [**Publicwww**](https://publicwww.com)
* [**SpyOnWeb**](http://spyonweb.com)

### **फेविकॉन**

क्या आप जानते हैं कि हम लक्ष्य के संबंधित डोमेन और सब डोमेन खोज सकते हैं जो एक ही फेविकॉन आइकन हैश की तलाश में हैं? यह वही है जिसे [favihash.py](https://github.com/m4ll0k/Bug-Bounty-Toolz/blob/master/favihash.py) उपकरण द्वारा किया जाता है जिसे [@m4ll0k2](https://twitter.com/m4ll0k2) ने बनाया है। यहाँ है कैसे इसका उपयोग करें:
```bash
cat my_targets.txt | xargs -I %% bash -c 'echo "http://%%/favicon.ico"' > targets.txt
python3 favihash.py -f https://target/favicon.ico -t targets.txt -s
```
![favihash - उसी favicon आइकन हैश के साथ डोमेन खोजें](https://www.infosecmatter.com/wp-content/uploads/2020/07/favihash.jpg)

सीधे शब्दों में कहा जाए, favihash हमें हमारे लक्ष्य के समान favicon आइकन हैश वाले डोमेन खोजने की अनुमति देगा।

इसके अतिरिक्त, आप फेविकॉन हैश का उपयोग करके तकनीकों की भी खोज कर सकते हैं जैसा कि [**इस ब्लॉग पोस्ट**](https://medium.com/@Asm0d3us/weaponizing-favicon-ico-for-bugbounties-osint-and-what-not-ace3c214e139) में स्पष्ट किया गया है। इसका मतलब है कि अगर आपको किसी वेब तकनीक के एक संक्षिप्त संस्करण के favicon का हैश पता है तो आप शोडन में खोज कर सकते हैं और **और अधिक संकटग्रस्त स्थानों** को खोज सकते हैं:
```bash
shodan search org:"Target" http.favicon.hash:116323821 --fields ip_str,port --separator " " | awk '{print $1":"$2}'
```
यहाँ वेबसाइट के **फेविकॉन हैश की गणना** कैसे की जाती है:
```python
import mmh3
import requests
import codecs

def fav_hash(url):
response = requests.get(url)
favicon = codecs.encode(response.content,"base64")
fhash = mmh3.hash(favicon)
print(f"{url} : {fhash}")
return fhash
```
### **कॉपीराइट / अद्वितीय स्ट्रिंग**

वेब पेज्स के अंदर खोजें **स्ट्रिंग्स जो संगठन के विभिन्न वेब्स में साझा की जा सकती हैं**। **कॉपीराइट स्ट्रिंग** एक अच्छा उदाहरण हो सकता है। फिर उस स्ट्रिंग को **google**, अन्य **ब्राउज़र्स** या यहाँ तक कि **shodan** में खोजें: `shodan search http.html:"कॉपीराइट स्ट्र
```bash
# /etc/crontab
37 13 */10 * * certbot renew --post-hook "systemctl reload nginx"
```
### **पासिव ताकता**

ज्ञात होता है कि लोगों के लिए सबडोमेन्स को ऐसे आईपी पतों से जोड़ना सामान्य है जो क्लाउड प्रदाताओं के हैं और किसी समय उस आईपी पता को खो देते हैं लेकिन डीएनएस रिकॉर्ड हटाने के बारे में भूल जाते हैं। इसलिए, बस क्लाउड में एक वीएम उत्पन्न करने से आप वास्तव में कुछ सबडोमेन्स को **अधिकार में ले रहे होंगे**।

[**इस पोस्ट**](https://kmsec.uk/blog/passive-takeover/) में इसके बारे में एक कहानी है और एक स्क्रिप्ट का प्रस्ताव है जो **DigitalOcean में एक वीएम उत्पन्न करता है**, नए मशीन का **IPv4** प्राप्त करता है, और उसके लिए उक्तियों के लिए Virustotal में सबडोमेन रिकॉर्ड खोजता है।

### **अन्य तरीके**

**ध्यान दें कि आप इस तकनीक का उपयोग करके हर बार जब आप एक नया डोमेन खोजते हैं तो अधिक डोमेन नामों की खोज कर सकते हैं।**

**Shodan**

आपको पहले से ही पता है कि आईपी स्पेस के मालिकाने का नाम क्या है। आप उस डेटा के द्वारा shodan में इसकी खोज कर सकते हैं: `org:"Tesla, Inc."` टीएलएस प्रमाणपत्र में नए अप्रत्याशित डोमेनों के लिए पाए गए होस्ट्स की जांच करें।

आप मुख्य वेब पृष्ठ के **टीएलएस प्रमाणपत्र** तक पहुंच सकते हैं, **संगठन का नाम** प्राप्त कर सकते हैं और फिर उस नाम की खोज कर सकते हैं **shodan** के सभी वेब पृष्ठों के **टीएलएस प्रमाणपत्रों** में जिन्हें फ़िल्टर के साथ जाना जाता है: `ssl:"Tesla Motors"` या [**sslsearch**](https://github.com/HarshVaragiya/sslsearch) जैसा एक उपकरण का उपयोग करें।

**Assetfinder**

[**Assetfinder** ](https://github.com/tomnomnom/assetfinder) एक उपकरण है जो मुख्य डोमेन से संबंधित **डोमेन** और उनके **सबडोमेन्स** की खोज करता है, बहुत अद्भुत।

### **दुर्बलताओं की खोज**

कुछ [डोमेन अधिकार](../../pentesting-web/domain-subdomain-takeover.md#domain-takeover) की जांच करें। शायद कोई कंपनी **किसी डोमेन का उपयोग कर रही हो** लेकिन उसका **स्वामित्व खो दिया हो**। बस इसे रजिस्टर करें (अगर सस्ता हो) और कंपनी को सूचित करें।

यदि आप किसी **डोमेन को पाते हैं जिसका आईपी पता अलग है** जो आपने पहले असेट्स डिस्कवरी में पाए हैं, तो आपको एक **मूलभूत दुर्बलता स्कैन** (Nessus या OpenVAS का उपयोग करके) और कुछ [**पोर्ट स्कैन**](../pentesting-network/#discovering-hosts-from-the-outside) करना चाहिए **nmap/masscan/shodan** के साथ। जिस सेवाएं चल रही हैं, उन्हें "हमला" करने के लिए **इस पुस्तक में कुछ ट्रिक्स** मिल सकती हैं।\
_ध्यान दें कि कभी-कभी डोमेन उस आईपी के अंदर होस्ट किया जाता है जिस पर ग्राहक का नियंत्रण नहीं है, इसलिए यह स्कोप में नहीं है, सावधान रहें।_
```bash
dnsrecon -a -d tesla.com
```
### **OSINT**

बहुत सारे सबडोमेन प्राप्त करने का सबसे तेज तरीका बाहरी स्रोतों में खोजना है। सबसे अधिक उपयोग किए जाने वाले **उपकरण** निम्नलिखित हैं (बेहतर परिणाम प्राप्त करने के लिए API कुंजियों को कॉन्फ़िगर करें):

* [**BBOT**](https://github.com/blacklanternsecurity/bbot)
```bash
# subdomains
bbot -t tesla.com -f subdomain-enum

# subdomains (passive only)
bbot -t tesla.com -f subdomain-enum -rf passive

# subdomains + port scan + web screenshots
bbot -t tesla.com -f subdomain-enum -m naabu gowitness -n my_scan -o .
```
* [**Amass**](https://github.com/OWASP/Amass)
```bash
amass enum [-active] [-ip] -d tesla.com
amass enum -d tesla.com | grep tesla.com # To just list subdomains
```
* [**subfinder**](https://github.com/projectdiscovery/subfinder)
```bash
# Subfinder, use -silent to only have subdomains in the output
./subfinder-linux-amd64 -d tesla.com [-silent]
```
* [**findomain**](https://github.com/Edu4rdSHL/findomain/)
```bash
# findomain, use -silent to only have subdomains in the output
./findomain-linux -t tesla.com [--quiet]
```
* [**OneForAll**](https://github.com/shmilylty/OneForAll/tree/master/docs/en-us)
```bash
python3 oneforall.py --target tesla.com [--dns False] [--req False] [--brute False] run
```
* [**assetfinder**](https://github.com/tomnomnom/assetfinder)
```bash
assetfinder --subs-only <domain>
```
* [**Sudomy**](https://github.com/Screetsec/Sudomy)
```bash
# It requires that you create a sudomy.api file with API keys
sudomy -d tesla.com
```
* [**विटा**](https://github.com/junnlikestea/vita)
```
vita -d tesla.com
```
* [**theHarvester**](https://github.com/laramies/theHarvester)
```bash
theHarvester -d tesla.com -b "anubis, baidu, bing, binaryedge, bingapi, bufferoverun, censys, certspotter, crtsh, dnsdumpster, duckduckgo, fullhunt, github-code, google, hackertarget, hunter, intelx, linkedin, linkedin_links, n45ht, omnisint, otx, pentesttools, projectdiscovery, qwant, rapiddns, rocketreach, securityTrails, spyse, sublist3r, threatcrowd, threatminer, trello, twitter, urlscan, virustotal, yahoo, zoomeye"
```
ऐसे **और दिलचस्प उपकरण/APIs** भी हैं जो सबडोमेन्स खोजने में सीधे रूप से विशेषज्ञ नहीं हैं, लेकिन सबडोमेन्स खोजने में उपयोगी हो सकते हैं, जैसे:

* [**Crobat**](https://github.com/cgboal/sonarsearch)**:** API [https://sonar.omnisint.io](https://sonar.omnisint.io) का उपयोग सबडोमेन्स प्राप्त करने के लिए करता है
```bash
# Get list of subdomains in output from the API
## This is the API the crobat tool will use
curl https://sonar.omnisint.io/subdomains/tesla.com | jq -r ".[]"
```
* [**JLDC मुफ्त API**](https://jldc.me/anubis/subdomains/google.com)
```bash
curl https://jldc.me/anubis/subdomains/tesla.com | jq -r ".[]"
```
* [**RapidDNS**](https://rapiddns.io) मुफ्त API
```bash
# Get Domains from rapiddns free API
rapiddns(){
curl -s "https://rapiddns.io/subdomain/$1?full=1" \
| grep -oE "[\.a-zA-Z0-9-]+\.$1" \
| sort -u
}
rapiddns tesla.com
```
* [**https://crt.sh/**](https://crt.sh)
```bash
# Get Domains from crt free API
crt(){
curl -s "https://crt.sh/?q=%25.$1" \
| grep -oE "[\.a-zA-Z0-9-]+\.$1" \
| sort -u
}
crt tesla.com
```
* [**gau**](https://github.com/lc/gau)**:** किसी दिए गए डोमेन से AlienVault के ओपन थ्रेट एक्सचेंज, द वेबैक मशीन, और कॉमन क्रॉल से ज्ञात यूआरएल लाता है।
```bash
# Get subdomains from GAUs found URLs
gau --subs tesla.com | cut -d "/" -f 3 | sort -u
```
* [**SubDomainizer**](https://github.com/nsonaniya2010/SubDomainizer) **&** [**subscraper**](https://github.com/Cillian-Collins/subscraper): वे वेब को स्क्रैप करते हैं जाकर JS फ़ाइलें खोजते हैं और वहां से subdomains निकालते हैं।
```bash
# Get only subdomains from SubDomainizer
python3 SubDomainizer.py -u https://tesla.com | grep tesla.com

# Get only subdomains from subscraper, this already perform recursion over the found results
python subscraper.py -u tesla.com | grep tesla.com | cut -d " " -f
```
* [**शोडन**](https://www.shodan.io/)
```bash
# Get info about the domain
shodan domain <domain>
# Get other pages with links to subdomains
shodan search "http.html:help.domain.com"
```
* [**Censys सबडोमेन खोजक**](https://github.com/christophetd/censys-subdomain-finder)
```bash
export CENSYS_API_ID=...
export CENSYS_API_SECRET=...
python3 censys-subdomain-finder.py tesla.com
```
* [**DomainTrail.py**](https://github.com/gatete/DomainTrail)
```bash
python3 DomainTrail.py -d example.com
```
* [**securitytrails.com**](https://securitytrails.com/) के पास उपनाम और आईपी इतिहास खोजने के लिए एक मुफ्त API है
* [**chaos.projectdiscovery.io**](https://chaos.projectdiscovery.io/#/)

यह परियोजना **मुफ्त में सभी उपनाम प्रदान करती है जो बग-बाउंटी कार्यक्रमों से संबंधित हैं**। आप इस डेटा तक पहुंच सकते हैं [chaospy](https://github.com/dr-0x0x/chaospy) का उपयोग करके या इस परियोजना द्वारा उपयोग किए जाने वाले स्कोप तक पहुंच सकते हैं [https://github.com/projectdiscovery/chaos-public-program-list](https://github.com/projectdiscovery/chaos-public-program-list)

आप इन उपकरणों की **तुलना** यहाँ कर सकते हैं: [https://blog.blacklanternsecurity.com/p/subdomain-enumeration-tool-face-off](https://blog.blacklanternsecurity.com/p/subdomain-enumeration-tool-face-off)

### **DNS ब्रूट फोर्स**

चलिए नए **उपनाम** ढूंढने के लिए DNS सर्वरों को ब्रूट-फोर्स करने का प्रयास करें, संभावित उपनाम नामों का उपयोग करके।

इस क्रिया के लिए आपको कुछ **सामान्य उपनाम शब्द सूचियों की आवश्यकता होगी जैसे**:

* [https://gist.github.com/jhaddix/86a06c5dc309d08580a018c66354a056](https://gist.github.com/jhaddix/86a06c5dc309d08580a018c66354a056)
* [https://wordlists-cdn.assetnote.io/data/manual/best-dns-wordlist.txt](https://wordlists-cdn.assetnote.io/data/manual/best-dns-wordlist.txt)
* [https://localdomain.pw/subdomain-bruteforce-list/all.txt.zip](https://localdomain.pw/subdomain-bruteforce-list/all.txt.zip)
* [https://github.com/pentester-io/commonspeak](https://github.com/pentester-io/commonspeak)
* [https://github.com/danielmiessler/SecLists/tree/master/Discovery/DNS](https://github.com/danielmiessler/SecLists/tree/master/Discovery/DNS)

और भी अच्छे DNS रिज़ॉल्वर्स के आईपी। विश्वसनीय DNS रिज़ॉल्वर्स की सूची उत्पन्न करने के लिए आप रिज़ॉल्वर्स को डाउनलोड कर सकते हैं [https://public-dns.info/nameservers-all.txt](https://public-dns.info/nameservers-all.txt) और उन्हें फ़िल्टर करने के लिए [**dnsvalidator**](https://github.com/vortexau/dnsvalidator) का उपयोग कर सकते हैं। या आप इस्तेमाल कर सकते हैं: [https://raw.githubusercontent.com/trickest/resolvers/main/resolvers-trusted.txt](https://raw.githubusercontent.com/trickest/resolvers/main/resolvers-trusted.txt)

DNS ब्रूट फोर्स के लिए सबसे अधिक सिफारिश की जाने वाली उपकरण हैं:

* [**massdns**](https://github.com/blechschmidt/massdns): यह पहला उपकरण था जो एक प्रभावी DNS ब्रूट फोर्स करता था। यह बहुत तेज है हालांकि यह गलत सकारात्मक के लिए प्रवृत है।
```bash
sed 's/$/.domain.com/' subdomains.txt > bf-subdomains.txt
./massdns -r resolvers.txt -w /tmp/results.txt bf-subdomains.txt
grep -E "tesla.com. [0-9]+ IN A .+" /tmp/results.txt
```
* [**gobuster**](https://github.com/OJ/gobuster): इसमें मुझे लगता है कि केवल 1 रिज़ॉल्वर का उपयोग होता है।
```
gobuster dns -d mysite.com -t 50 -w subdomains.txt
```
* [**shuffledns**](https://github.com/projectdiscovery/shuffledns) एक `massdns` के चारों ओर लिपटा हुआ है, गो में लिखा गया है, जो आपको सक्रिय ब्रूटफोर्स का उपयोग करके वैध सबडोमेन की जांच करने की अनुमति देता है, साथ ही वाइल्डकार्ड हैंडलिंग और आसान इनपुट-आउटपुट समर्थन के साथ सबडोमेन को हल करने की अनुमति देता है।
```
shuffledns -d example.com -list example-subdomains.txt -r resolvers.txt
```
* [**puredns**](https://github.com/d3mondev/puredns): यह भी `massdns` का उपयोग करता है।
```
puredns bruteforce all.txt domain.com
```
* [**aiodnsbrute**](https://github.com/blark/aiodnsbrute) एसिंक्रोनसली डोमेन नामों को ब्रूट फोर्स करने के लिए asyncio का उपयोग करता है।
```
aiodnsbrute -r resolvers -w wordlist.txt -vv -t 1024 domain.com
```
### दूसरे DNS ब्रूट-फोर्स राउंड

ओपन सोर्सेज और ब्रूट-फोर्सिंग का उपयोग करके सबडोमेन्स पाने के बाद, आप पाए गए सबडोमेन्स के बदलाव उत्पन्न कर सकते हैं ताकि और भी अधिक पाए जा सकें। इस उद्देश्य के लिए कई उपकरण उपयुक्त हैं:

* [**dnsgen**](https://github.com/ProjectAnte/dnsgen)**:** डोमेन और सबडोमेन्स देकर परिमुटेशन उत्पन्न करें।
```bash
cat subdomains.txt | dnsgen -
```
* [**goaltdns**](https://github.com/subfinder/goaltdns): डोमेन और सबडोमेन दिए गए परिपत्रन उत्पन्न करें।
* आप यहाँ [**वर्डलिस्ट**](https://github.com/subfinder/goaltdns/blob/master/words.txt) में goaltdns परिपत्रन प्राप्त कर सकते हैं।
```bash
goaltdns -l subdomains.txt -w /tmp/words-permutations.txt -o /tmp/final-words-s3.txt
```
* [**gotator**](https://github.com/Josue87/gotator)**:** डोमेन और सबडोमेन दिए गए परिमाण उत्पन्न करें। यदि कोई परिमाण फ़ाइल निर्दिष्ट नहीं की गई है तो gotator अपना उपयोग करेगा।
```
gotator -sub subdomains.txt -silent [-perm /tmp/words-permutations.txt]
```
* [**altdns**](https://github.com/infosec-au/altdns): सबडोमेन परिमुटेशन उत्पन्न करने के अलावा, यह उन्हें सुलझाने की कोशिश भी कर सकता है (लेकिन पिछले टिप्पणीत किए गए उपकरणों का उपयोग करना बेहतर है)।
* आप यहाँ [**यहाँ**](https://github.com/infosec-au/altdns/blob/master/words.txt) altdns परिमुटेशन **वर्डलिस्ट** प्राप्त कर सकते हैं।
```
altdns -i subdomains.txt -w /tmp/words-permutations.txt -o /tmp/asd3
```
* [**dmut**](https://github.com/bp0lr/dmut): एक औजार जो सबडोमेन की संवर्धन, म्युटेशन और परिवर्तन करने के लिए है। यह उपकरण परिणाम को ब्रूट फोर्स करेगा (यह डीएनएस वाइल्ड कार्ड का समर्थन नहीं करता)।
* आप dmut permutations वर्डलिस्ट [**यहाँ**](https://raw.githubusercontent.com/bp0lr/dmut/main/words.txt) में प्राप्त कर सकते हैं।
```bash
cat subdomains.txt | dmut -d /tmp/words-permutations.txt -w 100 \
--dns-errorLimit 10 --use-pb --verbose -s /tmp/resolvers-trusted.txt
```
* [**alterx**](https://github.com/projectdiscovery/alterx)**:** एक डोमेन पर आधारित है जो **नए संभावित सबडोमेन नाम** उत्पन्न करता है जो निर्दिष्ट पैटर्न के आधार पर अधिक सबडोमेन खोजने की कोशिश करता है।

#### स्मार्ट परिमुटेशन जनरेशन

* [**regulator**](https://github.com/cramppet/regulator): अधिक जानकारी के लिए इस [**पोस्ट**](https://cramppet.github.io/regulator/index.html) को पढ़ें लेकिन यह मुख्य भागों को **खोजे गए सबडोमेन** से प्राप्त करेगा और उन्हें मिश्रित करेगा ताकि अधिक सबडोमेन मिल सकें।
```bash
python3 main.py adobe.com adobe adobe.rules
make_brute_list.sh adobe.rules adobe.brute
puredns resolve adobe.brute --write adobe.valid
```
* [**subzuf**](https://github.com/elceef/subzuf)**:** _subzuf_ एक सबडोमेन ब्रूट-फोर्स फज़र है जिसे एक अत्यधिक सरल लेकिन प्रभावी DNS प्रतिक्रिया-निर्देशित एल्गोरिथ्म के साथ जोड़ा गया है। यह एक विशेषित वर्डलिस्ट या ऐतिहासिक DNS/TLS रिकॉर्ड्स जैसे प्रदत्त इनपुट डेटा का उपयोग करता है, ताकि अधिक संबंधित डोमेन नामों को सार्थक रूप से सिन्थेसाइज़ किया जा सके और DNS स्कैन के दौरान जुटाई गई जानकारी के आधार पर उन्हें और अधिक विस्तारित किया जा सके।
```
echo www | subzuf facebook.com
```
### **सबडोमेन खोज प्रक्रिया**

इस ब्लॉग पोस्ट को जांचें जिसमें मैंने लिखा है कि कैसे **ट्रिकेस्ट वर्कफ़्लो** का उपयोग करके डोमेन से सबडोमेन खोज को **स्वचालित** बनाया जा सकता है ताकि मुझे अपने कंप्यूटर पर मैन्युअल रूप से कई उपकरणों को लॉन्च करने की आवश्यकता न हो:

{% embed url="https://trickest.com/blog/full-subdomain-discovery-using-workflow/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

{% embed url="https://trickest.com/blog/full-subdomain-brute-force-discovery-using-workflow/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

### **VHosts / वर्चुअल होस्ट्स**

अगर आपने किसी आईपी पते को पाया है जिसमें **एक या कई वेब पेज** सबडोमेन के साथ हैं, तो आप उस आईपी में **अन्य सबडोमेन ढूंढने का प्रयास** कर सकते हैं जिनमें वेब हो, या तो **ओएसआईएनटी स्रोतों** में जाकर उस आईपी में डोमेन खोज सकते हैं या फिर **उस आईपी में VHost डोमेन नामों को ब्रूट-फ़ोर्स कर सकते हैं**।

#### OSINT

आप कुछ **आईपी में VHosts को खोजने के लिए** [**HostHunter**](https://github.com/SpiderLabs/HostHunter) **या अन्य एपीआई** का उपयोग कर सकते हैं।

**ब्रूट फ़ोर्स**

अगर आपको लगता है कि किसी सबडोमेन को वेब सर्वर में छिपा हो सकता है तो आप इसे ब्रूट फ़ोर्स करने की कोशिश कर सकते हैं:
```bash
ffuf -c -w /path/to/wordlist -u http://victim.com -H "Host: FUZZ.victim.com"

gobuster vhost -u https://mysite.com -t 50 -w subdomains.txt

wfuzz -c -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-20000.txt --hc 400,404,403 -H "Host: FUZZ.example.com" -u http://example.com -t 100

#From https://github.com/allyshka/vhostbrute
vhostbrute.py --url="example.com" --remoteip="10.1.1.15" --base="www.example.com" --vhosts="vhosts_full.list"

#https://github.com/codingo/VHostScan
VHostScan -t example.com
```
{% hint style="info" %}
इस तकनीक के साथ आप आंतरिक/छिपे हुए एंडपॉइंट्स तक पहुंच सकते हैं।
{% endhint %}

### **CORS ब्रूट फोर्स**

कभी-कभी आप पृष्ठ मिलेंगे जो केवल वैध डोमेन/सबडोमेन सेट होने पर _**Origin**_ हेडर में वापस _**Access-Control-Allow-Origin**_ हेडर वापस करते हैं। इन परिदृश्यों में, आप इस व्यवहार का दुरुपयोग करके **नए सबडोमेन** की **खोज** कर सकते हैं।
```bash
ffuf -w subdomains-top1million-5000.txt -u http://10.10.10.208 -H 'Origin: http://FUZZ.crossfit.htb' -mr "Access-Control-Allow-Origin" -ignore-body
```
### **बकेट्स ब्रूट फोर्स**

**सबडोमेन्स** की खोज करते समय ध्यान रखें कि क्या यह किसी प्रकार के **बकेट** को **पॉइंट** कर रहा है, और उस मामले में [**अनुमतियों की जांच करें**](../../network-services-pentesting/pentesting-web/buckets/)**.**\
इसके अलावा, इस समय आपको यह पता चल जाएगा कि दायरा में सभी डोमेन्स के बारे में, [**संभावित बकेट नामों को ब्रूट फोर्स करें और अनुमतियों की जांच करें**](../../network-services-pentesting/pentesting-web/buckets/).

### **मॉनिटरिजेशन**

आप **नए सबडोमेन्स** की **मॉनिटरिंग** कर सकते हैं जो किसी डोमेन के द्वारा बनाए गए हैं, [**सर्टिफिकेट ट्रांसपेरेंसी** लॉग](https://github.com/yassineaboukir/sublert/blob/master/sublert.py) की मॉनिटरिंग करके जैसे कि **sublert** करता है।

### **वंर्णन की खोज**

[**सबडोमेन टेकओवर्स**](../../pentesting-web/domain-subdomain-takeover.md#subdomain-takeover) के लिए जांच करें।\
अगर **सबडोमेन** किसी **S3 बकेट** को पॉइंट कर रहा है, [**अनुमतियों की जांच करें**](../../network-services-pentesting/pentesting-web/buckets/).

अगर आपको assets discovery में पहले से पता चले गए डोमेन्स से अलग **आईपी वाला सबडोमेन** मिलता है, तो आपको एक **बेसिक वलनरेबिलिटी स्कैन** (Nessus या OpenVAS का उपयोग करके) और कुछ [**पोर्ट स्कैन**](../pentesting-network/#discovering-hosts-from-the-outside) जैसे **nmap/masscan/shodan** के साथ करना चाहिए। चल रही सेवाओं पर आपको **इस पुस्तक में कुछ ट्रिक्स मिल सकती हैं जिनका उपयोग करके उन्हें "हमला" कर सकते हैं**।\
_ध्यान दें कि कभी-कभी सबडोमेन एक ऐसे आईपी के अंदर होस्ट किया जा रहा है जो ग्राहक द्वारा नियंत्रित नहीं है, इसलिए यह दायरे में नहीं है, सावधान रहें।_

## आईपी

प्रारंभिक चरणों में आपने **कुछ आईपी रेंज, डोमेन और सबडोमेन्स** पाए होंगे।\
अब उस रेंज से **सभी आईपी को एकत्र करने** और **डोमेन/सबडोमेन्स (DNS क्वेरीज)** के लिए समय है।

निम्नलिखित **मुफ्त एपीआई** सेवाओं का उपयोग करके आप डोमेन और सबडोमेन्स द्वारा पहले उपयोग किए गए **पिछले आईपी** भी खोज सकते हैं। ये आईपी अब भी ग्राहक द्वारा स्वामित्व में हो सकते हैं (और आपको [**क्लाउडफ्लेयर बाइपास**](../../network-services-pentesting/pentesting-web/uncovering-cloudflare.md) खोजने की अनुमति देने की संभावना है)

* [**https://securitytrails.com/**](https://securitytrails.com/)

आप टूल [**hakip2host**](https://github.com/hakluke/hakip2host) का उपयोग करके एक विशिष्ट आईपी पते पर पॉइंट करने वाले डोमेन की जांच भी कर सकते हैं।

### **वंर्णन की खोज**

**सभी सीडीएन्स के नहीं होने वाली सभी आईपी को पोर्ट स्कैन करें** (क्योंकि आपको वहां कुछ भी दिलचस्प नहीं मिलेगा)। चल रही सेवाओं में खोजे गए वलनरेबिलिटीज़ मिल सकती हैं।

**होस्ट स्कैन करने के लिए** [**गाइड**](../pentesting-network/) **खोजें।**

## वेब सर्वर्स की खोज

> हमने सभी कंपनियों और उनके संपत्तियों को खोज लिया है और हमें दायरे में आईपी रेंज, डोमेन और सबडोमेन्स के बारे में पता चल गया है। अब वेब सर्वर्स की खोज करने का समय है।

पिछले चरणों में आपने संकलित किए गए आईपी और डोमेन की **रिकॉन** प्रदर्शन किया हो सकता है, इसलिए आपने **संभावित वेब सर्वर्स** पहले ही पता कर लिए होंगे। हालांकि, अगर आपने नहीं किया है तो हम अब देखेंगे कि दायरे में **वेब सर्वर्स** खोजने के लिए कुछ **तेज ट्रिक्स**।

कृपया ध्यान दें कि यह **वेब ऐप्स खोज** के लिए होगा, इसलिए आपको **वलनरेबिलिटी** और **पोर्ट स्कैनिंग** भी करना चाहिए (**यदि दायरे में अनुमति है**।)

[**masscan का उपयोग करके वेब सर्वर्स से संबंधित खुले पोर्ट्स** की खोज करने के लिए एक **तेज विधि** यहाँ मिल सकती है](../pentesting-network/#http-port-discovery)।\
एक और दोस्ताना टूल वेब सर्वर्स की खोज करने के लिए [**httprobe**](https://github.com/tomnomnom/httprobe)**,** [**fprobe**](https://github.com/theblackturtle/fprobe) और [**httpx**](https://github.com/projectdiscovery/httpx) है। आप एक डोमेन्स की सूची पास करेंगे और यह पोर्ट 80 (http) और 443 (https) से कनेक्ट करने की कोशिश करेगा। इसके अतिरिक्त, आप अन्य पोर्ट्स की कोशिश करने के लिए इंडिकेट कर सकते हैं:
```bash
cat /tmp/domains.txt | httprobe #Test all domains inside the file for port 80 and 443
cat /tmp/domains.txt | httprobe -p http:8080 -p https:8443 #Check port 80, 443 and 8080 and 8443
```
### **स्क्रीनशॉट्स**

अब जब आपने स्कोप में मौजूद **सभी वेब सर्वर** (कंपनी के **IPs** और सभी **डोमेन** और **सबडोमेन्स**) का पता लगा लिया है, तो आपको शायद **शुरू कहाँ से करें** यह पता नहीं होगा। इसलिए, आइए इसे सरल बनाते हैं और सभी का स्क्रीनशॉट लेने से शुरू करें। **मुख्य पृष्ठ** पर एक **अजीब** एंडपॉइंट्स मिल सकते हैं जो अधिक **वंशीय** हो सकते हैं।

प्रस्तावित विचार को कार्यान्वित करने के लिए आप [**EyeWitness**](https://github.com/FortyNorthSecurity/EyeWitness), [**HttpScreenshot**](https://github.com/breenmachine/httpscreenshot), [**Aquatone**](https://github.com/michenriksen/aquatone), [**Shutter**](https://shutter-project.org/downloads/third-party-packages/) या [**webscreenshot**](https://github.com/maaaaz/webscreenshot) का उपयोग कर सकते हैं।

इसके अतिरिक्त, आप फिर [**eyeballer**](https://github.com/BishopFox/eyeballer) का उपयोग कर सकते हैं ताकि वह सभी **स्क्रीनशॉट्स** पर चले और आपको बताए कि **किसमें संभावित रूप से कमियां हो सकती हैं**, और क्या नहीं।

## सार्वजनिक क्लाउड संपत्तियाँ

किसी कंपनी की संभावित क्लाउड संपत्तियों का पता लगाने के लिए आपको उस कंपनी को पहचानने वाले शब्दों की सूची से शुरू करना चाहिए। उदाहरण के लिए, एक क्रिप्टो कंपनी के लिए आप शब्द जैसे कि: `"क्रिप्टो", "वॉलेट", "डाओ", "<डोमेन_नाम>", <"सबडोमेन_नाम">` का उपयोग कर सकते हैं।

आपको उन सामान्य शब्दों की सूचियों की आवश्यकता होगी जो बकेट्स में उपयोग किए जाते हैं:

* [https://raw.githubusercontent.com/cujanovic/goaltdns/master/words.txt](https://raw.githubusercontent.com/cujanovic/goaltdns/master/words.txt)
* [https://raw.githubusercontent.com/infosec-au/altdns/master/words.txt](https://raw.githubusercontent.com/infosec-au/altdns/master/words.txt)
* [https://raw.githubusercontent.com/jordanpotti/AWSBucketDump/master/BucketNames.txt](https://raw.githubusercontent.com/jordanpotti/AWSBucketDump/master/BucketNames.txt)

फिर, उन शब्दों से आपको **अनुक्रमणिकाएँ** उत्पन्न करनी चाहिए (अधिक जानकारी के लिए [**Second Round DNS Brute-Force**](./#second-dns-bruteforce-round) देखें)।

नतीजन्त: शब्द सूचियों के साथ आप [**cloud\_enum**](https://github.com/initstring/cloud\_enum), [**CloudScraper**](https://github.com/jordanpotti/CloudScraper), [**cloudlist**](https://github.com/projectdiscovery/cloudlist) या [**S3Scanner**](https://github.com/sa7mon/S3Scanner) जैसे उपकरणों का उपयोग कर सकते हैं।

ध्यान दें कि क्लाउड संपत्तियों की खोज करते समय आपको **AWS में बकेट्स के अलावा भी देखना चाहिए**।

### **कमियों की खोज**

यदि आपको **खुले बकेट्स या क्लाउड फंक्शन उजागर** जैसी चीजें मिलती हैं, तो आपको उन्हें **एक्सेस** करना चाहिए और देखने की कोशिश करनी चाहिए कि वे आपको क्या प्रदान करते हैं और क्या आप उनका दुरुपयोग कर सकते हैं।

## ईमेल

स्कोप में मौजूद **डोमेन** और **सबडोमेन्स** के साथ आपके पास आम तौर पर ईमेल खोजने के लिए सभी कुछ है। ये हैं उन **APIs** और **उपकरण** जो मेरे लिए सबसे अच्छे साबित हुए हैं ईमेल खोजने के लिए:

* [**theHarvester**](https://github.com/laramies/theHarvester) - APIs के साथ
* [**https://hunter.io/**](https://hunter.io/) (नि:शुल्क संस्करण) का API
* [**https://app.snov.io/**](https://app.snov.io/) (नि:शुल्क संस्करण) का API
* [**https://minelead.io/**](https://minelead.io/) (नि:शुल्क संस्करण) का API

### **कमियों की खोज**

ईमेल बाद में **वेब लॉगिन और प्रमाणीकरण सेवाओं के लिए ब्रूट-फोर्स** (जैसे SSH) करने के लिए उपयोगी होंगे। इसके अतिरिक्त, ये **फिशिंग** के लिए भी आवश्यक हैं। इसके अतिरिक्त, ये APIs आपको ईमेल के पीछे व्यक्ति के बारे में और अधिक **जानकारी** देंगे, जो फिशिंग अभियान के लिए उपयोगी है।

## प्रमाणक लीक

**डोमेन,** **सबडोमेन्स,** और **ईमेल्स** के साथ आप उन ईमेल्स के साथ संबंधित पहले लीक हुए प्रमाणकों की खोज करना शुरू कर सकते हैं:

* [https://leak-lookup.com](https://leak-lookup.com/account/login)
* [https://www.dehashed.com/](https://www.dehashed.com/)

### **कमियों की खोज**

यदि आप **मान्य लीक** प्रमाणक पाते हैं, तो यह बहुत ही आसान जीत है।

## रहस्य लीक

प्रमाणक लीक कंपनियों के हैक्स से संबंधित होती हैं जहां **संवेदनशील जानकारी लीक और बेची गई** थी। हालांकि, कंपनियों को **अन्य लीक** से प्रभावित किया जा सकता है जिनकी जानकारी उन डेटाबेसों में नहीं है:

### गिटहब लीक

प्रमाणक और API गिटहब कंपनी के **सार्वजनिक रिपॉजिटरी** या उस गिटहब कंपनी के काम करने वाले **उपयोगकर्ताओं** के सार्वजनिक रिपॉजिटरी में लीक हो सकते हैं।\
आप उपकरण [**Leakos**](https://github.com/carlospolop/Leakos) का उपयोग कर सकते हैं एक संगठन के और उसके डेवलपर्स के सभी **सार्वजनिक रेपो** को **डाउनलोड** करने और उन पर [**gitleaks**](https://github.com/zricethezav/gitleaks) को स्वचालित रूप से चलाने के लिए।

**Leakos** का उपयोग भी सभी **टेक्स्ट** प्रदान किए गए **URLs पास** करने के लिए किया जा सकता है क्योंकि कभी-कभी **वेब पेज में भी रहस्य होते हैं**।

#### गिटहब डोर्क्स

आप उस संगठन के लिए भी खोज सकते हैं **गिटहब डोर्क्स** के लिए जिन्हें आप आक्रमण कर रहे हैं:

{% content-ref url="github-leaked-secrets.md" %}
[github-leaked-secrets.md](github-leaked-secrets.md)
{% endcontent-ref %}

### पेस्ट्स लीक

कभी-कभी हमलावर या केवल कर्मचारी कंपनी सामग्री को एक पेस्ट साइट में प्रकाशित करेंगे। इसमें **संवेदनशील जानकारी** हो सकती है या नहीं हो सकती है, लेकिन इसे खोजना बहुत दिलचस्प है।\
आप उपकरण [**Pastos**](https://github.com/carlospolop/Pastos) का उपयोग कर सकते हैं एक साथ 80 से अधिक पेस्ट साइटों में खोजने के लिए।

### गूगल डोर्क्स

पुराने लेकिन सोने के गूगल डोर्क्स हमेशा उपयोगी होते हैं **उस जानकारी को खोजने के लिए जो वहाँ नहीं होना चाहिए**। एकमात्र समस्या यह है कि [**google-hacking-database**](https://www.exploit-db.com/google-hacking-database) कई हजार संभावित क्वेरी शामिल करता है जिन्हें आप मैन्युअल रूप से नहीं चला सकते। इसलिए, आप अपने पसंदीदा 10 क्वेरी ले सकते हैं या आप एक **उपकरण जैसे** [**Gorks**](https://github.com/carlospolop/Gorks) **का उपयोग कर सकते हैं सभी को चलाने के लिए**।

_ध्यान दें कि उपकरण जो सामान्य Google ब्राउज़र का उपयोग करने की उम्मीद करते हैं वे कभी समाप्त नहीं होंगे क्योंकि ग
## [**पेंटेस्टिंग वेब मेथडोलॉजी**](../../network-services-pentesting/pentesting-web/)

**बग हंटर्स** द्वारा पाए गए **अधिकांश सुरक्षा गड़बड़** **वेब एप्लिकेशन** के अंदर होती है, इसलिए इस बिंदु पर मैं एक **वेब एप्लिकेशन टेस्टिंग मेथडोलॉजी** के बारे में बात करना चाहूंगा, और आप इस [**जानकारी को यहाँ पा सकते हैं**](../../network-services-pentesting/pentesting-web/).

मैं भी विशेष रूप से ध्यान देना चाहूंगा [**वेब स्वचालित स्कैनर ओपन सोर्स टूल्स**](../../network-services-pentesting/pentesting-web/#automatic-scanners) के खंड को, क्योंकि, यदि आपको उन्हें आपको बहुत संवेदनशील गड़बड़ताओं को खोजने की उम्मीद नहीं है, तो वे **कुछ प्रारंभिक वेब जानकारी पर लागू करने में मददगार होते हैं।**

## संक्षेपण

> बधाई हो! इस बिंदु पर आपने पहले से ही **सभी मौलिक जांच** कर ली है। हाँ, यह मौलिक है क्योंकि और भी अधिक जांच की जा सकती है (बाद में और ट्रिक्स देखेंगे)।

तो आपने पहले से ही किया है:

1. स्कोप में सभी **कंपनियों** को खोज लिया है
2. कंपनियों के सभी **संपत्तियों** को खोज लिया है (और स्कोप में है तो कुछ वल्न स्कैन किया है)
3. कंपनियों के सभी **डोमेन** को खोज लिया है
4. डोमेन के सभी **सबडोमेन** को खोज लिया है (कोई सबडोमेन ताक़तवर हो गया है?)
5. स्कोप में सभी **आईपी** (सीडीएन से **नहीं**) को खोज लिया है।
6. सभी **वेब सर्वर** को खोज लिया है और उनका **स्क्रीनशॉट** लिया है (कुछ अजीब चीजें जो एक गहरी झलक के लिए योग्य हैं?)
7. कंपनी के सभी **संभावित सार्वजनिक क्लाउड संपत्तियाँ** को खोज लिया है।
8. **ईमेल**, **क्रेडेंशियल लीक**, और **रहस्य लीक** जो आपको एक **बड़ी जीत बहुत आसानी से** दे सकते हैं।
9. आपने पाए सभी वेब्स को **पेंटेस्ट** किया है

## **पूर्ण रिकॉन स्वचालित टूल्स**

कई ऐसे टूल्स हैं जो दिए गए स्कोप के खिलाफ प्रस्तावित कार्रवाई का हिस्सा करेंगे।

* [**https://github.com/yogeshojha/rengine**](https://github.com/yogeshojha/rengine)
* [**https://github.com/j3ssie/Osmedeus**](https://github.com/j3ssie/Osmedeus)
* [**https://github.com/six2dez/reconftw**](https://github.com/six2dez/reconftw)
* [**https://github.com/hackerspider1/EchoPwn**](https://github.com/hackerspider1/EchoPwn) - थोड़ा पुराना है और अपडेट नहीं है

## **संदर्भ**

* सभी मुफ्त पाठ्यक्रम [**@Jhaddix**](https://twitter.com/Jhaddix) के जैसे [**The Bug Hunter's Methodology v4.0 - Recon Edition**](https://www.youtube.com/watch?v=p4JgIu1mceI)

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

यदि आप **हैकिंग करियर** में रुचि रखते हैं और अनहैकेबल को हैक करना चाहते हैं - **हम नियुक्ति कर रहे हैं!** (_फ्लूएंट पोलिश लिखित और बोली जरुरी है_)

{% embed url="https://www.stmcyber.com/careers" %}

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो बनाने के साथ</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी कंपनी को **हैकट्रिक्स में विज्ञापित करना चाहते हैं** या **हैकट्रिक्स को पीडीएफ में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में पीआर जमा करके।

</details>
