# बाहरी जासूसी मेथडोलॉजी

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert) के साथ</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

<img src="../../.gitbook/assets/i3.png" alt="" data-size="original">\
**बग बाउंटी टिप**: **Intigriti** के लिए **साइन अप करें**, एक प्रीमियम **बग बाउंटी प्लेटफॉर्म जो हैकर्स द्वारा बनाई गई है**! आज हमारे साथ शामिल हों [**https://go.intigriti.com/hacktricks**](https://go.intigriti.com/hacktricks) और शुरू करें बाउंटी अप तक **$100,000** तक कमाना!

{% embed url="https://go.intigriti.com/hacktricks" %}

## संपत्तियों की खोज

> तो आपको कहा गया था कि किसी कंपनी के सभी वस्तुएं स्कोप में हैं, और आपको यह जानना है कि यह कंपनी वास्तव में क्या स्वामित्व रखती है।

इस चरण का उद्देश्य है कि **मुख्य कंपनी द्वारा स्वामित्व में रखी सभी कंपनियों** को प्राप्त करना और फिर इन कंपनियों की **संपत्तियों** को प्राप्त करना। इसे करने के लिए हमें निम्नलिखित करना होगा:

1. मुख्य कंपनी की अधिग्रहणों का पता लगाना, यह हमें स्कोप में आने वाली कंपनियों देगा।
2. प्रत्येक कंपनी का एएसएन (यदि कोई हो) खोजना, यह हमें प्रत्येक कंपनी द्वारा स्वामित्व में रखे गए आईपी सीमाएं देगा।
3. रिवर्स व्हॉइस लुकअप का उपयोग करके पहले वाले से संबंधित अन्य एंट्री (संगठन के नाम, डोमेन...) की खोज करना (यह पुनरावृत्ति किया जा सकता है)
4. अन्य तकनीकों का उपयोग करना जैसे shodan `org` और `ssl` फ़िल्टर्स का उपयोग करके अन्य संपत्तियों की खोज करना (यह `ssl` ट्रिक पुनरावृत्ति किया जा सकता है)।

### **अधिग्रहण**

सबसे पहले, हमें जानना होगा कि **मुख्य कंपनी द्वारा किन अन्य कंपनियों का स्वामित्व है**।\
एक विकल्प है [https://www.crunchbase.com/](https://www.crunchbase.com) पर जाने, **मुख्य कंपनी** के लिए **खोज** करें, और "**अधिग्रहण**" पर **क्लिक** करें। वहां आपको मुख्य कंपनी द्वारा अधिग्रहित की गई अन्य कंपनियों को दिखाया जाएगा।\
दूसरा विकल्प है मुख्य कंपनी के **विकिपीडिया** पृष्ठ पर जाना और **अधिग्रहण** के लिए खोज करना।

> ठीक है, इस बिंदु पर आपको स्कोप में आने वाली सभी कंपनियों को जानना चाहिए। चलिए उनकी संपत्तियों को कैसे खोजें, इसे समझते हैं।

### **एएसएन्स**

एक स्वतंत्र प्रणाली संख्या (**ASN**) एक **अटोनोमस सिस्टम** (AS) को **अंतर्निहित संख्या** द्वारा **असाइन** की जाती है जो **इंटरनेट असाइन्ड नंबर्स अथॉरिटी (आईएएनए)** द्वारा किया जाता है।\
एक **एएस** में **आईपी पतों** के **ब्लॉक** होते हैं जिनके लिए बाहरी नेटवर्कों तक पहुंचने के लिए एक स्पष्ट निर्धारित नीति होती है और एक ही संगठन द्वारा प्रबंधित होते हैं लेकिन कई ऑपरेटर्स से मिलकर बने हो सकते हैं।

यह दिलचस्प होगा कि क्या **कंपनी ने किसी एएसएन को सौंपा है** ताकि इसके **आईपी सीमाएं पता लगाई जा सकें**। स्कोप के सभी **होस्ट** के खिलाफ एक **सुरक्षा परीक्षण** करना दिलचस्प होगा और इन आईपी में डोमेन्स की खोज करनी होगी।\
आप [**https://bgp.he.net/**](https://bgp.he.net) में कंपनी के **नाम**, **आईपी** या **डोमेन** से खोज कर सकते हैं।\
**कंपनी के क्षेत्र के आधार पर यह लिंक्स अधिक डेटा इकट्ठा करने के लिए उपयोगी हो सकते हैं:** [**AFRINIC**](https://www.afrinic.net) **(अफ्रीका),** [**Arin**](https://www.arin.net/about/welcome/region/)**(उत्तर अमेरिका),** [**APNIC**](https://www.apnic.net) **(एशिया),** [**LACNIC**](https://www.lacnic.net) **(लैटिन अमेरिका),** [**RIPE NCC**](https://www.ripe.net) **(यूरोप)। फिर भी, संभावित है कि सभी** उपयोगी जानकारी **(आईपी सीमाएं और व्हॉइस)** पहले लिंक में ही दिखाई देती है।
```bash
#You can try "automate" this with amass, but it's not very recommended
amass intel -org tesla
amass intel -asn 8911,50313,394161
```
इसके अलावा, [**BBOT**](https://github.com/blacklanternsecurity/bbot)**'s** सबडोमेन जांच स्वचालित रूप से ASNs को समेकित और सारांशित करता है स्कैन के अंत में।
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

### **वंर्णित सुरक्षा दोषों की खोज**

इस समय हमें **दायरा में सभी संपत्तियों** के बारे में पता है, इसलिए यदि आपको अनुमति है तो आप कुछ **सुरक्षा स्कैनर** (Nessus, OpenVAS) को सभी होस्ट पर लॉन्च कर सकते हैं।\
इसके अलावा, आप कुछ [**पोर्ट स्कैन**](../pentesting-network/#discovering-hosts-from-the-outside) भी **लॉन्च कर सकते हैं या खोले गए द्वारों को खोजने के लिए** shodan **जैसी सेवाओं का उपयोग कर सकते हैं और आपको जो मिलता है उसके आधार पर आपको** इस पुस्तक में कुछ संभावित सेवाओं को पेंटेस्ट करने के लिए कैसे देखना चाहिए।\
**इसके अलावा, यह उल्लेखनीय हो सकता है कि आप कुछ** डिफ़ॉल्ट उपयोगकर्ता **और** पासवर्ड **सूचियाँ भी तैयार कर सकते हैं और [https://github.com/x90skysn3k/brutespray](https://github.com/x90skysn3k/brutespray) का उपयोग करके सेवाओं को** ब्रूटफ़ोर्स **करने की कोशिश कर सकते हैं।

## डोमेन

> हमें दायरा में सभी कंपनियों और उनकी संपत्तियों के बारे में पता है, अब हमें दायरा में डोमेन्स खोजने का समय है।

_कृपया ध्यान दें कि निम्नलिखित प्रस्तावित तकनीकों में आप सबडोमेन्स भी पा सकते हैं और उस जानकारी को कम महत्व न दें।_

सबसे पहले आपको प्रत्येक कंपनी के **मुख्य डोमेन**(s) की खोज करनी चाहिए। उदाहरण के लिए, _Tesla Inc._ के लिए _tesla.com_ होगा।

### **रिवर्स DNS**

जैसे ही आपने सभी डोमेनों के IP रेंज पाए हैं, आप उन IP पर **रिवर्स DNS लुकअप** करने का प्रयास कर सकते हैं ताकि दायरा में और डोमेन पाए जा सकें। पीडीएन या किसी पीडीएन के डीएनएस सर्वर (1.1.1.1, 8.8.8.8) का उपयोग करने की कोशिश करें।
```bash
dnsrecon -r <DNS Range> -n <IP_DNS>   #DNS reverse of all of the addresses
dnsrecon -d facebook.com -r 157.240.221.35/24 #Using facebooks dns
dnsrecon -r 157.240.221.35/24 -n 1.1.1.1 #Using cloudflares dns
dnsrecon -r 157.240.221.35/24 -n 8.8.8.8 #Using google dns
```
### **पलटें व्हॉइस (लूप)**

एक **व्हॉइस** के अंदर आपको कई दिलचस्प **जानकारी** मिल सकती है जैसे **संगठन का नाम**, **पता**, **ईमेल**, फोन नंबर... लेकिन जो और भी दिलचस्प है वह यह है कि आप कंपनी से संबंधित **अधिक संपत्तियाँ** खोज सकते हैं अगर आप **उन क्षेत्रों में से किसी भी क्षेत्र के लिए पलटें व्हॉइस लुकअप करते हैं** (उदाहरण के लिए जहां एक ही ईमेल प्रकट होता है वहां अन्य व्हॉइस रजिस्ट्रीज जहां यह ईमेल प्रकट होता है)।\
आप ऑनलाइन उपकरणों का उपयोग कर सकते हैं:

* [https://viewdns.info/reversewhois/](https://viewdns.info/reversewhois/) - **मुफ्त**
* [https://domaineye.com/reverse-whois](https://domaineye.com/reverse-whois) - **मुफ्त**
* [https://www.reversewhois.io/](https://www.reversewhois.io) - **मुफ्त**
* [https://www.whoxy.com/](https://www.whoxy.com) - **मुफ्त** वेब, मुफ्त एपीआई नहीं।
* [http://reversewhois.domaintools.com/](http://reversewhois.domaintools.com) - मुफ्त नहीं
* [https://drs.whoisxmlapi.com/reverse-whois-search](https://drs.whoisxmlapi.com/reverse-whois-search) - मुफ्त नहीं (केवल **100 मुफ्त** खोजें)
* [https://www.domainiq.com/](https://www.domainiq.com) - मुफ्त नहीं

आप इस कार्य को [**DomLink** ](https://github.com/vysecurity/DomLink)(whoxy API कुंजी की आवश्यकता है) का उपयोग करके स्वचालित कर सकते हैं।\
आप [amass](https://github.com/OWASP/Amass) के साथ कुछ स्वचालित पलटें व्हॉइस डिस्कवरी भी कर सकते हैं: `amass intel -d tesla.com -whois`

**ध्यान दें कि आप इस तकनीक का उपयोग करके हर बार जब आप एक नया डोमेन पाते हैं तो अधिक डोमेन नामों की खोज करने के लिए इस तकनीक का उपयोग कर सकते हैं।**

### **ट्रैकर्स**

यदि आप 2 विभिन्न पृष्ठों में **एक ही ट्रैकर की एक ही आईडी** पाते हैं तो आप समझ सकते हैं कि **दोनों पृष्ठ** **एक ही टीम द्वारा प्रबंधित** हैं।\
उदाहरण के लिए, यदि आप कई पृष्ठों पर **एक ही Google Analytics आईडी** या **एक ही Adsense आईडी** देखते हैं।

कुछ पृष्ठ और उपकरण हैं जो आपको इन ट्रैकर्स और अधिक द्वारा खोजने देते हैं:

* [**Udon**](https://github.com/dhn/udon)
* [**BuiltWith**](https://builtwith.com)
* [**Sitesleuth**](https://www.sitesleuth.io)
* [**Publicwww**](https://publicwww.com)
* [**SpyOnWeb**](http://spyonweb.com)

### **फेविकॉन**

क्या आप जानते हैं कि हम अपने लक्ष्य के संबंधित डोमेन और सब डोमेन्स को एक ही फेविकॉन आइकन हैश खोजकर खोज सकते हैं? यह वही है जिसे [favihash.py](https://github.com/m4ll0k/Bug-Bounty-Toolz/blob/master/favihash.py) उपकरण द्वारा किया जा सकता है जिसे [@m4ll0k2](https://twitter.com/m4ll0k2) ने बनाया है। यहाँ इसका उपयोग कैसे करें:
```bash
cat my_targets.txt | xargs -I %% bash -c 'echo "http://%%/favicon.ico"' > targets.txt
python3 favihash.py -f https://target/favicon.ico -t targets.txt -s
```
![favihash - उसी favicon आइकन हैश के साथ डोमेन खोजें](https://www.infosecmatter.com/wp-content/uploads/2020/07/favihash.jpg)

सीधे शब्दों में कहा जाए, favihash हमें हमारे लक्ष्य के समान favicon आइकन हैश वाले डोमेन खोजने की अनुमति देगा।

इसके अतिरिक्त, आप फेविकॉन हैश का उपयोग करके तकनीकों की भी खोज कर सकते हैं जैसा कि [**इस ब्लॉग पोस्ट**](https://medium.com/@Asm0d3us/weaponizing-favicon-ico-for-bugbounties-osint-and-what-not-ace3c214e139) में स्पष्ट किया गया है। इसका मतलब है कि यदि आपको किसी वेब तकनीक के एक संक्षिप्त संस्करण के favicon का हैश पता है तो आप शोडन में खोज कर सकते हैं और **और अधिक संकटपूर्ण स्थानों** को खोज सकते हैं।
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

वेब पेजों में खोजें **स्ट्रिंग्स जो एक ही संगठन के विभिन्न वेब्स में साझा की जा सकती हैं**। **कॉपीराइट स्ट्रिंग** एक अच्छा उदाहरण हो सकता है। फिर उस स्ट्रिंग को **google**, अन्य **ब्राउज़र्स** या यहाँ तक कि **shodan** में खोजें: `shodan search http.html:"कॉपीराइट स्ट्र
```bash
# /etc/crontab
37 13 */10 * * certbot renew --post-hook "systemctl reload nginx"
```
### **पासिव ताकता**

ज्ञात होता है कि लोगों के लिए सामान्य है कि वे सबडोमेन को ऐसे आईपी पतों पर असाइन करें जो क्लाउड प्रदाताओं के होते हैं और किसी समय उस आईपी पता को खो दें लेकिन DNS रिकॉर्ड को हटाने के बारे में भूल जाएं। इसलिए, बस क्लाउड में एक वीएम उत्पन्न करने से आप वास्तव में कुछ सबडोमेन (स) को **अधिकार में ले रहे होंगे**।

[**इस पोस्ट**](https://kmsec.uk/blog/passive-takeover/) में इसके बारे में एक कहानी है और एक स्क्रिप्ट का प्रस्ताव है जो **DigitalOcean में एक वीएम उत्पन्न करता है**, नए मशीन का **IPv4** प्राप्त करता है, और उसे इस पर प्वाइंट करने वाले सबडोमेन रिकॉर्ड के लिए Virustotal में खोजता है।

### **अन्य तरीके**

**ध्यान दें कि आप इस तकनीक का उपयोग करके हर बार जब आप एक नया डोमेन खोजते हैं, अधिक डोमेन नामों की खोज करने के लिए कर सकते हैं।**

**Shodan**

आपको पहले से ही पता है कि आईपी स्पेस के स्वामित्व में कौन संगठन है। आप उस डेटा के द्वारा shodan में खोज कर सकते हैं: `org:"Tesla, Inc."` टीएलएस प्रमाणपत्र में नए अप्रत्याशित डोमेनों के लिए पाए गए होस्ट की जाँच करें।

आप मुख्य वेब पृष्ठ के **TLS प्रमाणपत्र** तक पहुंच सकते हैं, **संगठन का नाम** प्राप्त कर सकते हैं और फिर उस नाम की खोज कर सकते हैं **shodan** के सभी वेब पृष्ठों के **TLS प्रमाणपत्रों** में जिन्हें आप जानते हैं फ़िल्टर के साथ: `ssl:"Tesla Motors"` या [**sslsearch**](https://github.com/HarshVaragiya/sslsearch) जैसा एक टूल का उपयोग करें।

**Assetfinder**

[**Assetfinder**](https://github.com/tomnomnom/assetfinder) एक टूल है जो मुख्य डोमेन से संबंधित **डोमेन** और उनके **सबडोमेन** की खोज करता है, बहुत शानदार।

### **दुर्बलताओं की खोज**

कुछ [डोमेन अधिग्रहण](../../pentesting-web/domain-subdomain-takeover.md#domain-takeover) की जाँच करें। शायद कोई कंपनी **किसी डोमेन का उपयोग कर रही हो** लेकिन उन्होंने **स्वामित्व खो दिया हो**। बस इसे रजिस्टर करें (अगर सस्ता हो) और कंपनी को सूचित करें।

यदि आप किसी **डोमेन को पाते हैं जिसका आईपी अलग है** जो आपने पहले ही उत्पन्न किए गए संपत्तियों के आईपी पतों से नहीं मिलता है, तो आपको एक **मूलभूत दुर्बलता स्कैन** (Nessus या OpenVAS का उपयोग करके) और कुछ [**पोर्ट स्कैन**](../pentesting-network/#discovering-hosts-from-the-outside) करना चाहिए **nmap/masscan/shodan** के साथ। जिस सेवाएं चल रही हैं, उन्हें "हमला" करने के लिए **इस पुस्तक में कुछ ट्रिक्स** मिल सकती हैं।\
_ध्यान दें कि कभी-कभी डोमेन उस आईपी के अंदर होस्ट किया जाता है जिस पर ग्राहक का नियंत्रण नहीं है, इसलिए यह स्कोप में नहीं है, सावधान रहें।_
```bash
dnsrecon -a -d tesla.com
```
### **OSINT**

बहुत सारे सबडोमेन प्राप्त करने का सबसे तेज तरीका बाहरी स्रोतों में खोजना है। सबसे अधिक **उपयोग किए जाने वाले उपकरण** निम्नलिखित हैं (बेहतर परिणाम प्राप्त करने के लिए API कुंजियों को कॉन्फ़िगर करें):

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
* [**वीटा**](https://github.com/junnlikestea/vita)
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
* [**gau**](https://github.com/lc/gau)**:** किसी दिए गए डोमेन से AlienVault के ओपन थ्रेट एक्सचेंज, वे बैक मशीन, और कॉमन क्रॉल से ज्ञात यूआरएल लाता है।
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
* [**securitytrails.com**](https://securitytrails.com/) के पास उपडोमेन्स और आईपी इतिहास खोजने के लिए एक मुफ्त API है
* [**chaos.projectdiscovery.io**](https://chaos.projectdiscovery.io/#/)

यह परियोजना **मुफ्त में सभी बग-बाउंटी कार्यक्रमों से संबंधित सभी उपडोमेन्स** प्रदान करती है। आप इस डेटा तक पहुंच सकते हैं [chaospy](https://github.com/dr-0x0x/chaospy) का उपयोग करके या इस परियोजना द्वारा उपयोग किए जाने वाले स्कोप तक पहुंच सकते हैं [https://github.com/projectdiscovery/chaos-public-program-list](https://github.com/projectdiscovery/chaos-public-program-list)

आप इन उपकरणों की **तुलना** यहाँ पा सकते हैं: [https://blog.blacklanternsecurity.com/p/subdomain-enumeration-tool-face-off](https://blog.blacklanternsecurity.com/p/subdomain-enumeration-tool-face-off)

### **DNS ब्रूट फोर्स**

चलिए नए **उपडोमेन्स** ढूंढने के लिए DNS सर्वरों को ब्रूट-फोर्स करने का प्रयास करें जिसमें संभावित उपडोमेन नामों का उपयोग किया जाता है।

इस क्रिया के लिए आपको कुछ **सामान्य उपडोमेन वर्डलिस्ट की आवश्यकता होगी जैसे**:

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
* [**gobuster**](https://github.com/OJ/gobuster): इसमें मुझे लगता है कि केवल 1 रिज़ॉल्वर का उपयोग होता है
```
gobuster dns -d mysite.com -t 50 -w subdomains.txt
```
* [**shuffledns**](https://github.com/projectdiscovery/shuffledns) एक `massdns` के चारों ओर लिखा गया एक व्रापर है, जो आपको सक्रिय ब्रूटफोर्स का उपयोग करके वैध सबडोमेन की जांच करने की अनुमति देता है, साथ ही वाइल्डकार्ड हैंडलिंग और आसान इनपुट-आउटपुट समर्थन के साथ सबडोमेन को सुलझाने की अनुमति देता है।
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
### दूसरा DNS ब्रूट-फोर्स राउंड

ओपन सोर्सेज और ब्रूट-फोर्सिंग का उपयोग करके सबडोमेन्स पाने के बाद, आप पाए गए सबडोमेन्स के अल्टरेशन उत्पन्न कर सकते हैं ताकि और भी अधिक पाए जा सकें। इस उद्देश्य के लिए कई टूल उपयोगी हैं:

* [**dnsgen**](https://github.com/ProjectAnte/dnsgen)**:** डोमेन और सबडोमेन्स देकर परिवर्तन उत्पन्न करें।
```bash
cat subdomains.txt | dnsgen -
```
* [**goaltdns**](https://github.com/subfinder/goaltdns): डोमेन और सबडोमेन दिए गए परिमुटेशन उत्पन्न करें।
* आप यहाँ [**वर्डलिस्ट**](https://github.com/subfinder/goaltdns/blob/master/words.txt) में goaltdns परिमुटेशन प्राप्त कर सकते हैं।
```bash
goaltdns -l subdomains.txt -w /tmp/words-permutations.txt -o /tmp/final-words-s3.txt
```
* [**gotator**](https://github.com/Josue87/gotator)**:** डोमेन और सबडोमेन दिए गए परिमाणों को उत्पन्न करें। यदि कोई परिमाण फ़ाइल निर्दिष्ट नहीं की गई है तो gotator अपना उपयोग करेगा।
```
gotator -sub subdomains.txt -silent [-perm /tmp/words-permutations.txt]
```
* [**altdns**](https://github.com/infosec-au/altdns): सबडोमेन परिमुटेशन उत्पन्न करने के अलावा, यह उन्हें सुलझाने की कोशिश भी कर सकता है (लेकिन पिछले टिप्पणीत किए गए उपकरणों का उपयोग करना बेहतर है)।
* आप यहाँ [**यहाँ**](https://github.com/infosec-au/altdns/blob/master/words.txt) altdns परिमुटेशन **वर्डलिस्ट** प्राप्त कर सकते हैं।
```
altdns -i subdomains.txt -w /tmp/words-permutations.txt -o /tmp/asd3
```
* [**dmut**](https://github.com/bp0lr/dmut): एक औजार जो सबडोमेन की संवर्धन, परिवर्तन और बदलाव करने के लिए एक औजार है। यह औजार परिणाम को ब्रूट फोर्स करेगा (यह dns वाइल्ड कार्ड का समर्थन नहीं करता)।
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
* [**subzuf**](https://github.com/elceef/subzuf)**:** _subzuf_ एक सबडोमेन ब्रूट-फोर्स फज़र है जिसे एक अत्यधिक सरल लेकिन प्रभावी DNS प्रतिक्रिया-निर्देशित एल्गोरिथ्म के साथ जोड़ा गया है। यह एक विशेष वर्डलिस्ट या ऐतिहासिक DNS/TLS रिकॉर्ड्स जैसे प्रदत्त इनपुट डेटा सेट का उपयोग करता है, ताकि अधिक संबंधित डोमेन नामों को सार्थक रूप से सिन्थेसाइज़ किया जा सके और DNS स्कैन के दौरान जुटाई गई जानकारी के आधार पर उन्हें और भी अधिक विस्तारित किया जा सके।
```
echo www | subzuf facebook.com
```
### **सबडोमेन खोज वर्कफ़्लो**

जांचें इस ब्लॉग पोस्ट को जिसमें मैंने लिखा है कि कैसे **ट्रिकेस्ट वर्कफ़्लो** का उपयोग करके डोमेन से सबडोमेन खोज को **स्वचालित** बनाया जा सकता है ताकि मुझे अपने कंप्यूटर में मैन्युअल रूप से कई उपकरणों को लॉन्च करने की आवश्यकता न हो:

{% embed url="https://trickest.com/blog/full-subdomain-discovery-using-workflow/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

{% embed url="https://trickest.com/blog/full-subdomain-brute-force-discovery-using-workflow/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

### **VHosts / वर्चुअल होस्ट्स**

अगर आपने किसी आईपी पते को पाया है जिसमें **एक या कई वेब पेज** सबडोमेन के साथ हैं, तो आप **उस आईपी में अन्य सबडोमेन खोज सकते हैं जिनमें उस आईपी में वेब्स हैं** डोमेन्स के लिए **ओएसआईएनटी स्रोतों** में देखकर या **उस आईपी में VHost डोमेन नामों को ब्रूट-फ़ोर्सिंग** करके।

#### ओएसआईएनटी

आप कुछ **आईपी में VHosts को खोज सकते हैं** [**HostHunter**](https://github.com/SpiderLabs/HostHunter) **या अन्य एपीआई का उपयोग करके**।

**ब्रूट फ़ोर्स**

अगर आपको लगता है कि कुछ सबडोमेन एक वेब सर्वर में छिपा हो सकता है तो आप इसे ब्रूट फ़ोर्स करने की कोशिश कर सकते हैं:
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

कभी-कभी आप पृष्ठों को पाएंगे जो केवल तब _**Access-Control-Allow-Origin**_ हेडर वापस करते हैं जब एक मान्य डोमेन/सबडोमेन _**Origin**_ हेडर में सेट किया जाता है। इन परिदृश्यों में, आप इस व्यवहार का दुरुपयोग करके **नए सबडोमेन** की **खोज** कर सकते हैं।
```bash
ffuf -w subdomains-top1million-5000.txt -u http://10.10.10.208 -H 'Origin: http://FUZZ.crossfit.htb' -mr "Access-Control-Allow-Origin" -ignore-body
```
### **बकेट्स ब्रूट फोर्स**

**सबडोमेन्स** की खोज करते समय ध्यान रखें कि क्या यह किसी प्रकार के **बकेट** को **पॉइंट** कर रहा है, और उस मामले में [**अनुमतियों की जांच करें**](../../network-services-pentesting/pentesting-web/buckets/)**.**\
इसके अलावा, इस समय आपको यह पता चल जाएगा कि दायरा में सभी डोमेन्स के अंदर कौन-कौन से हैं, इसलिए [**संभावित बकेट नामों की ब्रूट फोर्स करें और अनुमतियों की जांच करें**](../../network-services-pentesting/pentesting-web/buckets/).

### **मॉनिटरिजेशन**

आप यह देख सकते हैं कि क्या किसी डोमेन के **नए सबडोमेन्स** बनाए गए हैं, **सर्टिफिकेट ट्रांसपेरेंसी** लॉग्स [**sublert** ](https://github.com/yassineaboukir/sublert/blob/master/sublert.py) की मॉनिटरिंग करके।

### **विकल्पों की खोज**

संभावित [**सबडोमेन टेकओवर्स**](../../pentesting-web/domain-subdomain-takeover.md#subdomain-takeover) के लिए जांच करें।\
अगर **सबडोमेन** किसी **S3 बकेट** को पॉइंट कर रहा है, तो [**अनुमतियों की जांच करें**](../../network-services-pentesting/pentesting-web/buckets/).

अगर आपको किसी **सबडोमेन** में किसी भी पहले से नहीं मिलने वाले आईपी का पता चलता है, तो आपको एक **बुनियादी सुरक्षा स्कैन** (Nessus या OpenVAS का उपयोग करके) और कुछ [**पोर्ट स्कैन**](../pentesting-network/#discovering-hosts-from-the-outside) भी करना चाहिए **nmap/masscan/shodan** के साथ। चल रही सेवाओं पर आपको **इस पुस्तक में कुछ ट्रिक्स मिल सकती हैं जिनका उपयोग करके उन्हें "हमला" कर सकते हैं**।\
_ध्यान दें कि कभी-कभी सबडोमेन एक ऐसे आईपी के अंदर होस्ट किया जा रहा है जो ग्राहक द्वारा नियंत्रित नहीं है, इसलिए यह दायरे में नहीं है, सावधान रहें._

## आईपी

प्रारंभिक चरणों में आपने **कुछ आईपी रेंज, डोमेन और सबडोमेन्स** पाए होंगे।\
अब समय है कि आप **उन रेंजों से सभी आईपी एवं डोमेन/सबडोमेन्स को पुनः जुटाएं** (DNS क्वेरीज का उपयोग करके)।

निम्नलिखित **मुफ्त एपीआई** सेवाओं का उपयोग करके आप डोमेन और सबडोमेन्स द्वारा पहले उपयोग किए गए आईपी भी खोज सकते हैं। ये आईपी अब भी ग्राहक द्वारा स्वामित्व में हो सकते हैं (और आपको [**क्लाउडफ्लेयर बाइपास**](../../network-services-pentesting/pentesting-web/uncovering-cloudflare.md) खोजने की अनुमति देने की संभावना है)

* [**https://securitytrails.com/**](https://securitytrails.com/)

आप टूल [**hakip2host**](https://github.com/hakluke/hakip2host) का उपयोग करके किसी विशिष्ट आईपी पते को पॉइंट करने वाले डोमेन की जांच भी कर सकते हैं।

### **विकल्पों की खोज**

**सभी सीडीएन्स से नहीं जुड़ी सभी आईपी का पोर्ट स्कैन करें** (क्योंकि आपको वहां कुछ भी दिलचस्प नहीं मिलेगा)। चल रही सेवाओं में पाए जाने पर आप **संभावित रूप से विकल्पों को खोज सकते हैं**।

**होस्ट स्कैन करने के लिए** [**गाइड**](../pentesting-network/) **खोजें।**

## वेब सर्वर्स हंटिंग

> हमने सभी कंपनियों और उनके संपत्तियों को खोज लिया है और हमें दायरा में आईपी रेंज, डोमेन और सबडोमेन्स के बारे में पता चल गया है। अब वेब सर्वर्स की खोज करने का समय है।

पिछले चरणों में आपने **खोज की होगी आईपी और डोमेन** की जांच, इसलिए आपने **संभावित वेब सर्वर्स** पहले ही पाए होंगे। हालांकि, अगर आपने नहीं किया है तो अब हम **देखेंगे कुछ तेज तरीके जिनसे वेब सर्वर्स की खोज की जा सकती है** दायरा के अंदर।

कृपया ध्यान दें कि यह **वेब ऐप्स खोज के लिए अभिवृत्त होगा**, इसलिए आपको **यदि दायरे द्वारा अनुमति दी गई है** तो **विकल्पों की खोज** और **पोर्ट स्कैनिंग** भी करनी चाहिए।

**वेब** सर्वर्स से संबंधित **खुले पोर्ट्स** खोजने के लिए [**masscan** यहाँ पाया जा सकता है](../pentesting-network/#http-port-discovery)।\
एक और मित्रशील टूल वेब सर्वर्स की खोज करने के लिए [**httprobe**](https://github.com/tomnomnom/httprobe)**,** [**fprobe**](https://github.com/theblackturtle/fprobe) और [**httpx**](https://github.com/projectdiscovery/httpx) है। आप एक डोमेन की सूची पास करते हैं और यह 80 (http) और 443 (https) पोर्ट से कनेक्ट करने की कोशिश करेगा। इसके अतिरिक्त, आप अन्य पोर्ट्स की कोशिश करने के लिए भी संकेत कर सकते हैं:
```bash
cat /tmp/domains.txt | httprobe #Test all domains inside the file for port 80 and 443
cat /tmp/domains.txt | httprobe -p http:8080 -p https:8443 #Check port 80, 443 and 8080 and 8443
```
### **स्क्रीनशॉट्स**

अब जब आपने स्कोप में मौजूद **सभी वेब सर्वर** (कंपनी के **IPs** और सभी **डोमेन** और **सबडोमेन्स** के बीच) का पता लगा लिया है, तो आपको शायद **पता नहीं होगा कि कहाँ से शुरू करें**। इसलिए, आइए इसे सरल बनाते हैं और उन सभी का स्क्रीनशॉट लेने की शुरुआत करें। **मुख्य पृष्ठ** पर एक नजर डालकर आप **अजीब** एंडपॉइंट्स खोज सकते हैं जो अधिक **वंशी** हो सकते हैं।

प्रस्तावित विचार को कार्यान्वित करने के लिए आप [**EyeWitness**](https://github.com/FortyNorthSecurity/EyeWitness), [**HttpScreenshot**](https://github.com/breenmachine/httpscreenshot), [**Aquatone**](https://github.com/michenriksen/aquatone), [**Shutter**](https://shutter-project.org/downloads/third-party-packages/) या [**webscreenshot**](https://github.com/maaaaz/webscreenshot) का उपयोग कर सकते हैं।

इसके अतिरिक्त, आप फिर [**eyeballer**](https://github.com/BishopFox/eyeballer) का उपयोग कर सकते हैं ताकि वह सभी **स्क्रीनशॉट्स** पर चले और आपको बताए कि **किसमें संभावित रूप से दोष** हो सकता है, और क्या नहीं।

## सार्वजनिक क्लाउड संपत्तियाँ

किसी कंपनी की संभावित क्लाउड संपत्तियों का पता लगाने के लिए आपको उस कंपनी की पहचान करने वाले शब्दों की सूची के साथ शुरुआत करनी चाहिए। उदाहरण के लिए, एक क्रिप्टो कंपनी के लिए आप शब्दों का उपयोग कर सकते हैं जैसे: `"क्रिप्टो", "वॉलेट", "डाओ", "<डोमेन_नाम>", <"सबडोमेन_नाम">`।

आपको उन सामान्य शब्दों की सूचियों की आवश्यकता होगी जो बकेट्स में उपयोग किए जाते हैं:

* [https://raw.githubusercontent.com/cujanovic/goaltdns/master/words.txt](https://raw.githubusercontent.com/cujanovic/goaltdns/master/words.txt)
* [https://raw.githubusercontent.com/infosec-au/altdns/master/words.txt](https://raw.githubusercontent.com/infosec-au/altdns/master/words.txt)
* [https://raw.githubusercontent.com/jordanpotti/AWSBucketDump/master/BucketNames.txt](https://raw.githubusercontent.com/jordanpotti/AWSBucketDump/master/BucketNames.txt)

फिर, उन शब्दों का विभिन्नाकरण उत्पन्न करना चाहिए (अधिक जानकारी के लिए [**दूसरे दूसरे DNS ब्रूट-फोर्स**](./#second-dns-bruteforce-round) की जांच करें)।

परिणामी शब्द सूचियों के साथ आप [**cloud\_enum**](https://github.com/initstring/cloud\_enum), [**CloudScraper**](https://github.com/jordanpotti/CloudScraper), [**cloudlist**](https://github.com/projectdiscovery/cloudlist) या [**S3Scanner**](https://github.com/sa7mon/S3Scanner) जैसे उपकरणों का उपयोग कर सकते हैं।

ध्यान दें कि क्लाउड संपत्तियों की खोज करते समय आपको **AWS में बकेट्स** के अलावा भी देखना चाहिए।

### **दोषों की खोज**

यदि आपको **खुले बकेट्स या क्लाउड फंक्शन उजागर** जैसी चीजें मिलती हैं, तो आपको उन्हें **एक्सेस** करना चाहिए और देखने की कोशिश करनी चाहिए कि वे आपको क्या प्रदान करते हैं और क्या आप उनका दुरुपयोग कर सकते हैं।

## ईमेल

स्कोप में मौजूद **डोमेन** और **सबडोमेन** के साथ आपके पास आम तौर पर ईमेल खोजने के लिए सभी वह है जो आपको **आरंभ करने के लिए आवश्यक है**। ये हैं उन **APIs** और **उपकरण** जो मेरे लिए सबसे अच्छे साबित हुए हैं ईमेल खोजने के लिए:

* [**theHarvester**](https://github.com/laramies/theHarvester) - एपीआई के साथ
* [**https://hunter.io/**](https://hunter.io/) का एपीआई (नि: शुल्क संस्करण)
* [**https://app.snov.io/**](https://app.snov.io/) का एपीआई (नि: शुल्क संस्करण)
* [**https://minelead.io/**](https://minelead.io/) का एपीआई (नि: शुल्क संस्करण)

### **दोषों की खोज**

ईमेल बाद में **वेब लॉगिन और प्रमाणीकरण सेवाओं के लिए ब्रूट-फोर्स** (जैसे SSH) करने के लिए उपयोगी होंगे। इसके अतिरिक्त, ये **फिशिंग** के लिए भी आवश्यक हैं। इसके अतिरिक्त, ये एपीआई आपको ईमेल के पीछे व्यक्ति के बारे में और अधिक **जानकारी देंगे**, जो फिशिंग अभियान के लिए उपयोगी है।

## प्रमाणक लीक

**डोमेन,** **सबडोमेन** और **ईमेल** के साथ आप उन ईमेलों के लिए लीक हुए प्रमाणकों की खोज शुरू कर सकते हैं जो उन ईमेलों के लिए लीक हुए हैं:

* [https://leak-lookup.com](https://leak-lookup.com/account/login)
* [https://www.dehashed.com/](https://www.dehashed.com/)

### **दोषों की खोज**

यदि आप **मान्य लीक** प्रमाणक पाते हैं, तो यह बहुत ही आसान जीत है।

## रहस्य लीक

**डोमेन,** **सबडोमेन** और **ईमेल** के साथ आप उन डेटा लीक हुए प्रमाणकों की खोज शुरू कर सकते हैं जो उन ईमेलों के लिए लीक हुए हैं:

* [https://leak-lookup.com](https://leak-lookup.com/account/login)
* [https://www.dehashed.com/](https://www.dehashed.com/)

### **दोषों की खोज**

यदि आप **मान्य लीक** प्रमाणक पाते हैं, तो यह बहुत ही आसान जीत है।
