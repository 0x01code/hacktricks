# बाहरी जासूसी की मेथडोलॉजी

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **हैकट्रिक्स में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह देखें
* [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल** हों या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का** अनुसरण करें।**
* **हैकिंग ट्रिक्स साझा करें, PRs के माध्यम से** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को सबमिट करके।**

</details>

<img src="../../.gitbook/assets/i3.png" alt="" data-size="original">\
**बग बाउंटी टिप**: **Intigriti** में **साइन अप** करें, एक प्रीमियम **बग बाउंटी प्लेटफॉर्म जो हैकर्स द्वारा बनाई गई है**! आज ही [**https://go.intigriti.com/hacktricks**](https://go.intigriti.com/hacktricks) पर शामिल हों और **$100,000** तक की बाउंटी कमाएं!

{% embed url="https://go.intigriti.com/hacktricks" %}

## संपत्ति की खोज

> तो आपको कहा गया है कि कंपनी की सभी चीजें स्कोप में हैं, और आप जानना चाहते हैं कि यह कंपनी वास्तव में क्या स्वामित्व में रखती है।

इस चरण का उद्देश्य है **मुख्य कंपनी द्वारा स्वामित्व में रखी गई सभी कंपनियों** को प्राप्त करना और फिर इन कंपनियों की **संपत्ति** को प्राप्त करना। इसके लिए, हम निम्नलिखित करेंगे:

1. मुख्य कंपनी की अधिग्रहणों का पता लगाएं, इससे हमें स्कोप में शामिल कंपनियां मिलेंगी।
2. प्रत्येक कंपनी का ASN (यदि कोई हो) खोजें, इससे हमें प्रत्येक कंपनी द्वारा स्वामित्व में रखे जाने वाले IP रेंज मिलेंगे।
3. रिवर्स व्होइस लुकअप का उपयोग करें, पहले वाले से संबंधित अन्य प्रविष्टियों (संगठन के नाम, डोमेन...) की खोज करने के लिए (यह पुनरावृत्ति के रूप में किया जा सकता है)।
4. शोडन `org` और `ssl` फ़िल्टर जैसी अन्य तकनीकों का उपयोग करें, अन्य संपत्तियों की खोज करने के लिए (यह `ssl` ट्रिक पुनरावृत्ति के रूप में किया जा सकता है)।

### **अधिग्रहण**

सबसे पहले, हमें जानना होगा कि मुख्य कंपनी द्वारा **अन्य कंपनियां किसके स्वामित्व में हैं**।\
एक विकल्प है [https://www.crunchbase.com/](https://www.crunchbase.com) पर जाना, **मुख्य कंपनी** की **खोज** करें, और "**अधिग्रहण**" पर **क्लिक** करें। वहां आपको मुख्य कंपनी द्वारा अधिग्रहण की गई अन्य कंपनियां दिखेंगी।\
दूसरा विकल्प है मुख्य कंपनी के **Wikipedia** पृष्ठ पर जाना और **अधिग्रहण** की खोज करना।

> ठीक है, इस बिंदु पर आपको स्कोप में शामिल कंपनियों की संपत्ति कैसे खोजनी है, इसे समझते हैं।

### **ASNs**

एक स्वतंत्र सिस्टम नंबर (**ASN**) एक **अवार्डित स्वतंत्र सिस्टम** (AS) को **अद्यतित संख्या** द्वारा **असाइन** किया जाता है, जिसे **इंटरनेट असाइन्ड नंबर्स अथॉरिटी (IANA)** द्वारा प्रशासित किया जाता है।\
एक **AS** में **ब्लॉक** ऑफ **IP पतों** का समावेश होता है जिनके लिए बाहरी नेटवर्कों तक पहुंच के लिए एक विशेष नीति होती है और इसे एकल संगठन द्वारा प्रशासित किया जाता है लेकिन इसमें कई ऑपरेटर्स हो सकते हैं।

यह देखना दिलचस्प होता ह
```bash
#You can try "automate" this with amass, but it's not very recommended
amass intel -org tesla
amass intel -asn 8911,50313,394161
```
इसके अलावा, [**BBOT**](https://github.com/blacklanternsecurity/bbot)**'s** सबडोमेन गणना स्वचालित रूप से स्कैन के अंत में ASNs को संकलित और संक्षेपित करती है।
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
आप एक संगठन के IP रेंज भी खोज सकते हैं, इसके लिए [http://asnlookup.com/](http://asnlookup.com) का उपयोग कर सकते हैं (इसमें मुफ्त API है)।\
आप एक डोमेन के IP और ASN को भी खोज सकते हैं, इसके लिए [http://ipv4info.com/](http://ipv4info.com) का उपयोग कर सकते हैं।

### **कमजोरियों की तलाश**

इस बिंदु पर हमें **स्कोप के अंदर के सभी संपत्तियों का पता चल गया है**, इसलिए अगर आपको अनुमति है तो आप कुछ **कमजोरियों की जांच करने वाला स्कैनर** (Nessus, OpenVAS) चला सकते हैं।\
इसके अलावा, आप कुछ [**पोर्ट स्कैन**](../pentesting-network/#discovering-hosts-from-the-outside) भी चला सकते हैं या shodan जैसी सेवाओं का उपयोग कर सकते हैं **खुले पोर्ट्स खोजने के लिए और आपको जो मिलता है उसके आधार पर आपको** इस पुस्तक में दिए गए कई संभावित सेवाओं को पेंटेस्ट करने के लिए इस पुस्तक में देखना चाहिए।\
**इसके अलावा, यह भी महत्वपूर्ण हो सकता है कि आप कुछ** डिफ़ॉल्ट उपयोगकर्ता नाम **और** पासवर्ड **सूची तैयार कर सकते हैं और** [https://github.com/x90skysn3k/brutespray](https://github.com/x90skysn3k/brutespray) का उपयोग करके सेवाओं को** ब्रूटफ़ोर्स कर सकते हैं।

## डोमेन

> हमें स्कोप के अंदर की सभी कंपनियों और उनकी संपत्तियों का पता चल गया है, अब समय है स्कोप के अंदर के डोमेन खोजने का।

_कृपया ध्यान दें कि निम्नलिखित प्रस्तावित तकनीकों में आप सबडोमेन भी खोज सकते हैं और उस सूचना को कम महत्व न दें।_

सबसे पहले आपको प्रत्येक कंपनी के **मुख्य डोमेन**(s) की खोज करनी चाहिए। उदाहरण के लिए, _Tesla Inc._ के लिए _tesla.com_ होगा।

### **रिवर्स DNS**

जब आपने सभी डोमेनों के IP रेंज खोज लिए हैं, तो आप उन IP पर **रिवर्स DNS लुकअप** करने का प्रयास कर सकते हैं और स्कोप के अंदर और डोमेन खोज सकते हैं। पीडीएन विक्टिम के कुछ डीएनएस सर्वर या किसी प्रसिद्ध डीएनएस सर्वर (1.1.1.1, 8.8.8.8) का उपयोग करने का प्रयास करें।
```bash
dnsrecon -r <DNS Range> -n <IP_DNS>   #DNS reverse of all of the addresses
dnsrecon -d facebook.com -r 157.240.221.35/24 #Using facebooks dns
dnsrecon -r 157.240.221.35/24 -n 1.1.1.1 #Using cloudflares dns
dnsrecon -r 157.240.221.35/24 -n 8.8.8.8 #Using google dns
```
इसके लिए कार्य करने के लिए, प्रशासक को PTR को मैन्युअली सक्षम करना होगा।
आप इस जानकारी के लिए एक ऑनलाइन टूल भी उपयोग कर सकते हैं: [http://ptrarchive.com/](http://ptrarchive.com)

### **रिवर्स व्होइस (लूप)**

एक **व्होइस** के अंदर आपको कई रोचक **जानकारी** मिल सकती है जैसे **संगठन का नाम**, **पता**, **ईमेल**, फोन नंबर... लेकिन जो और भी रोचक है, वह यह है कि आप यदि आप किसी भी फ़ील्ड के द्वारा **रिवर्स व्होइस लुकअप** करते हैं (उदाहरण के लिए जहां एक ही ईमेल प्रदर्शित होता है, वहां अन्य व्होइस रजिस्ट्री) तो आप कंपनी से संबंधित **अधिक संपत्ति खोज सकते हैं**।
आप ऑनलाइन टूल का उपयोग कर सकते हैं जैसे:

* [https://viewdns.info/reversewhois/](https://viewdns.info/reversewhois/) - **मुफ्त**
* [https://domaineye.com/reverse-whois](https://domaineye.com/reverse-whois) - **मुफ्त**
* [https://www.reversewhois.io/](https://www.reversewhois.io) - **मुफ्त**
* [https://www.whoxy.com/](https://www.whoxy.com) - **मुफ्त** वेब, मुफ्त एपीआई नहीं।
* [http://reversewhois.domaintools.com/](http://reversewhois.domaintools.com) - मुफ्त नहीं
* [https://drs.whoisxmlapi.com/reverse-whois-search](https://drs.whoisxmlapi.com/reverse-whois-search) - मुफ्त नहीं (केवल **100 मुफ्त** खोज)
* [https://www.domainiq.com/](https://www.domainiq.com) - मुफ्त नहीं

आप [**DomLink** ](https://github.com/vysecurity/DomLink) का उपयोग करके इस कार्य को स्वचालित कर सकते हैं (जिसके लिए एक whoxy API कुंजी की आवश्यकता होती है)।
आप [amass](https://github.com/OWASP/Amass) के साथ कुछ स्वचालित रिवर्स व्होइस डिस्कवरी भी कर सकते हैं: `amass intel -d tesla.com -whois`

**ध्यान दें कि आप इस तकनीक का उपयोग करके हर बार जब आप एक नया डोमेन खोजते हैं, तो अधिक डोमेन नाम खोजने के लिए कर सकते हैं।**

### **ट्रैकर्स**

यदि आप 2 अलग-अलग पृष्ठों में **एक ही ट्रैकर की एक ही आईडी** पाते हैं, तो आप सोच सकते हैं कि **दोनों पृष्ठ** को **एक ही टीम द्वारा प्रबंधित किया जाता है**।
उदाहरण के लिए, यदि आप कई पृष्ठों पर **एक ही Google Analytics आईडी** या **एक ही Adsense आईडी** देखते हैं।

कुछ पृष्ठ और टूल हैं जिनका उपयोग आप इन ट्रैकर्स और अधिक द्वारा खोज सकते हैं:

* [**Udon**](https://github.com/dhn/udon)
* [**BuiltWith**](https://builtwith.com)
* [**Sitesleuth**](https://www.sitesleuth.io)
* [**Publicwww**](https://publicwww.com)
* [**SpyOnWeb**](http://spyonweb.com)

### **फविकॉन**

क्या आप जानते हैं कि हम एक ही फविकॉन आइकन हैश की खोज करके अपने लक्ष्य से संबंधित डोमेन और सब डोमेन खोज सकते हैं? यह वही है जिसे [@m4ll0k2](https://twitter.com/m4ll0k2) द्वारा बनाए गए [favihash.py](https://github.com/m4ll0k/Bug-Bounty-Toolz/blob/master/favihash.py) टूल करता है। यहां इसका उपयोग करने का तरीका है:
```bash
cat my_targets.txt | xargs -I %% bash -c 'echo "http://%%/favicon.ico"' > targets.txt
python3 favihash.py -f https://target/favicon.ico -t targets.txt -s
```
![favihash - उसी favicon आइकन हैश वाले डोमेन्स की खोज](https://www.infosecmatter.com/wp-content/uploads/2020/07/favihash.jpg)

सीधे कहें तो, favihash हमें हमारे लक्ष्य के साथ एक ही favicon आइकन हैश वाले डोमेन्स की खोज करने की अनुमति देगा।

इसके अलावा, आप फविकॉन हैश का उपयोग करके तकनीकों की खोज भी कर सकते हैं जैसा कि [**इस ब्लॉग पोस्ट**](https://medium.com/@Asm0d3us/weaponizing-favicon-ico-for-bugbounties-osint-and-what-not-ace3c214e139) में समझाया गया है। इसका मतलब है कि यदि आपको किसी वेब तकनीक के एक संकटपूर्ण संस्करण के favicon का हैश पता है, तो आप शोडन में इसे खोजकर **और संकटपूर्ण स्थानों को खोज सकते हैं**:
```bash
shodan search org:"Target" http.favicon.hash:116323821 --fields ip_str,port --separator " " | awk '{print $1":"$2}'
```
यहाँ वेब के **फविकॉन हैश की गणना** कैसे कर सकते हैं:
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

वेब पेजों के भीतर खोजें **ऐसी स्ट्रिंग्स जो संगठन के विभिन्न वेब्साइटों के बीच साझा की जा सकती हैं**। कॉपीराइट स्ट्रिंग इसका एक अच्छा उदाहरण हो सकती है। फिर उस स्ट्रिंग को **गूगल**, अन्य **ब्राउज़र्स** या यहां तक कि **शोडन** में भी खोजें: `shodan search http.html:"Copyright string"`

### **CRT समय**

एक क्रॉन जॉब के रूप में एक सामान्य होता हैं जैसे कि
```bash
# /etc/crontab
37 13 */10 * * certbot renew --post-hook "systemctl reload nginx"
```
इस सर्वर पर सभी डोमेन प्रमाणपत्रों को नवीनीकृत करें। इसका अर्थ है कि यदि इसके लिए इस्तेमाल किए गए सीए ने इसे वैधता समय में उत्पन्न करने के लिए समय नहीं निर्धारित किया है, तो **प्रमाणपत्र पारदर्शिता लॉगों में उसी कंपनी के संबंध में डोमेन खोजना संभव है**।
इस [**व्रिटअप**](https://swarm.ptsecurity.com/discovering-domains-via-a-time-correlation-attack/) को जांचें और अधिक जानकारी के लिए।

### **पैशिव टेकओवर**

ज्ञात है कि लोगों को आमतौर पर ऐसे आईपी पतों को सबडोमेनों से जोड़ने के लिए नियुक्त किया जाता है जो क्लाउड प्रदाताओं के होते हैं और किसी समय उस आईपी पते को खो देते हैं लेकिन डीएनएस रिकॉर्ड को हटाने के बारे में भूल जाते हैं। इसलिए, बस किसी क्लाउड (जैसे डिजिटल ओशन) में एक वीएम बनाने से आप वास्तव में कुछ सबडोमेनों को **अधिकार में ले सकते हैं**।

[**इस पोस्ट**](https://kmsec.uk/blog/passive-takeover/) में इसके बारे में एक कहानी का वर्णन है और एक स्क्रिप्ट का प्रस्ताव किया गया है जो **डिजिटलओशन में एक वीएम बनाता है**, नई मशीन का **IPv4** प्राप्त करता है, और उसे इसके लिए संबंधित सबडोमेन रिकॉर्ड्स की खोज करता है।

### **अन्य तरीके**

**ध्यान दें कि आप इस तकनीक का उपयोग करके हर बार जब आप एक नया डोमेन खोजते हैं, अधिक डोमेन नाम खोजने के लिए कर सकते हैं।**

**शोडन**

आप पहले से ही जानते हैं कि आईपी स्थान के मालिकाना का नाम क्या है। आप उस डेटा के आधार पर शोडन में खोज कर सकते हैं: `org:"Tesla, Inc."` खोजे गए होस्ट्स को टीएलएस प्रमाणपत्र में नए अप्रत्याशित डोमेन खोजें।

आप मुख्य वेब पृष्ठ के **टीएलएस प्रमाणपत्र** तक पहुंच सकते हैं, **संगठन का नाम** प्राप्त कर सकते हैं और फिर **शोडन** द्वारा ज्ञात सभी वेब पृष्ठों के **टीएलएस प्रमाणपत्रों** में उस नाम की खोज कर सकते हैं जिन्हें फ़िल्टर के साथ जाना जाता है: `ssl:"Tesla Motors"` या [**sslsearch**](https://github.com/HarshVaragiya/sslsearch) जैसा एक उपकरण उपयोग करें।

**Assetfinder**

[**Assetfinder**](https://github.com/tomnomnom/assetfinder) एक ऐसा उपकरण है जो एक मुख्य डोमेन के संबंध में **संबंधित डोमेन** और उनके **सबडोमेन** की खोज करता है, बहुत अद्भुत।

### **कमजोरियों की तलाश**

[डोमेन टेकओवर](../../pentesting-web/domain-subdomain-takeover.md#domain-takeover) की जांच करें। शायद कोई कंपनी **किसी डोमेन का उपयोग कर रही हो** लेकिन उसे **स्वामित्व खो दिया हो**। इसे पंजीकृत करें (यदि पर्याप्त सस्ता हो) और कंपनी को सूचित करें।

यदि आपको किसी डोमेन में से किसी भी **अलग आईपी** के साथ कोई डोमेन मिलता है जो आपने पहले से ही एसेट्स डिस्कवरी में पाए गए हैं, तो आपको एक **मूलभूत सुरक्षा स्कैन** (Nessus या OpenVAS का उपयोग करके) और कुछ [**पोर्ट स्कैन**](../pentesting-network/#discovering-hosts-from-the-outside) जैसे **nmap/masscan/shodan** के साथ करना चाहिए। चलिए देखते हैं कि कौन सी सेवाएं चल रही हैं, आप उन्हें "हमला" करने के लिए इस पुस्तक में कुछ ट्रिक्स पा सकते हैं।
_ध्यान दें कि कभी-कभी डोमेन को उनके नियंत्रण में नहीं होने वाले आईपी के अंदर होस्ट किया जाता है, इसलिए यह स्कोप में नहीं होता है, सतर्क रहें।_

<img src="../../.gitbook/assets/i3.png" alt="" data-size="original">\
**बग बाउंटी टिप**: **Intigriti** में **साइन अप** करें, एक प्रीमियम **बग बाउंटी प्लेटफॉर्म जो हैकर्स द्वारा बनाई गई है**! आज ही [**https://go.intigriti.com/hacktricks**](https://go.intigriti.com/hacktricks) पर हमारे साथ शामिल हों और **$100,000** तक के बाउंटी कमाना शुरू करें!

{% embed url="https://go.intigriti.com/hacktricks" %}

## सबडोमेन

> हमें स्कोप में शामिल सभी कंपनियों के बारे में जानकारी है, प्रत्येक कंपनी के सभी एसेट्स और कंपनियों से संबंधित सभी डोमेन।

अब हमें प्राप्त हुए प्रत्येक डोमेन के संभावित सभी सबडोमेन ढूंढ़ने का समय है।

### **डीएनएस**

हमें **डीएनएस** रिकॉर्ड्स से सबडोमेन प्राप्त करने का प्रयास करें। हमें **ज़ोन ट्रांसफर** के लिए भी प्रयास करना चाहिए (यदि संकटग्रस्त हो, तो इसे रिपोर्ट करना चाहिए)।
```bash
dnsrecon -a -d tesla.com
```
### **OSINT**

सबडोमेन की बहुत सारी जानकारी प्राप्त करने का सबसे तेज़ तरीका बाहरी स्रोतों में खोज करना है। सबसे अधिक प्रयोग किए जाने वाले **उपकरण** निम्नलिखित हैं (बेहतर परिणामों के लिए API कुंजी कॉन्फ़िगर करें):

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
* [**OneForAll**](https://github.com/shmilylty/OneForAll/tree/master/docs/hi-in)
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
यहां **अन्य रोचक उपकरण/APIs** हैं जो सबडोमेन खोजने में सीधे विशेषज्ञ नहीं हैं, लेकिन उपयोगी हो सकते हैं, जैसे:

* [**Crobat**](https://github.com/cgboal/sonarsearch)**:** सबडोमेन प्राप्त करने के लिए [https://sonar.omnisint.io](https://sonar.omnisint.io) API का उपयोग करता है।
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
* [**gau**](https://github.com/lc/gau)**:** एलियनवॉल्ट के ओपन थ्रेट एक्सचेंज, द वेबवैक मशीन और कॉमन क्रॉल से किसी भी दिए गए डोमेन के लिए ज्ञात URL लाता है।
```bash
# Get subdomains from GAUs found URLs
gau --subs tesla.com | cut -d "/" -f 3 | sort -u
```
* [**SubDomainizer**](https://github.com/nsonaniya2010/SubDomainizer) **और** [**subscraper**](https://github.com/Cillian-Collins/subscraper): वे वेब को खोजकर JS फ़ाइलें खोजते हैं और वहां से subdomains निकालते हैं।
```bash
# Get only subdomains from SubDomainizer
python3 SubDomainizer.py -u https://tesla.com | grep tesla.com

# Get only subdomains from subscraper, this already perform recursion over the found results
python subscraper.py -u tesla.com | grep tesla.com | cut -d " " -f
```
* [**शोदन**](https://www.shodan.io/)
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
* [**securitytrails.com**](https://securitytrails.com/) में एक मुफ्त API है जिसका उपयोग सबडोमेन और IP इतिहास के लिए कर सकते हैं
* [**chaos.projectdiscovery.io**](https://chaos.projectdiscovery.io/#/)

यह प्रोजेक्ट **मुफ्त में बग-बाउंटी प्रोग्राम के संबंधित सबडोमेन सभी प्रदान करता है**। आप इस डेटा तक पहुंचने के लिए [chaospy](https://github.com/dr-0x0x/chaospy) का उपयोग कर सकते हैं या इस प्रोजेक्ट द्वारा उपयोग किए जाने वाले स्कोप तक पहुंच सकते हैं [https://github.com/projectdiscovery/chaos-public-program-list](https://github.com/projectdiscovery/chaos-public-program-list)

आप इन टूल्स की **तुलना** यहां कर सकते हैं: [https://blog.blacklanternsecurity.com/p/subdomain-enumeration-tool-face-off](https://blog.blacklanternsecurity.com/p/subdomain-enumeration-tool-face-off)

### **DNS ब्रूट फोर्स**

हम कोशिश करेंगे कि DNS सर्वरों को ब्रूट-फोर्सिंग करके नए **सबडोमेन** ढूंढ़ें जिन्हें संभावित सबडोमेन नामों का उपयोग करके प्राप्त किया जा सकता है।

इस कार्रवाई के लिए आपको कुछ **सामान्य सबडोमेन वर्डलिस्ट** की आवश्यकता होगी जैसे:

* [https://gist.github.com/jhaddix/86a06c5dc309d08580a018c66354a056](https://gist.github.com/jhaddix/86a06c5dc309d08580a018c66354a056)
* [https://wordlists-cdn.assetnote.io/data/manual/best-dns-wordlist.txt](https://wordlists-cdn.assetnote.io/data/manual/best-dns-wordlist.txt)
* [https://localdomain.pw/subdomain-bruteforce-list/all.txt.zip](https://localdomain.pw/subdomain-bruteforce-list/all.txt.zip)
* [https://github.com/pentester-io/commonspeak](https://github.com/pentester-io/commonspeak)
* [https://github.com/danielmiessler/SecLists/tree/master/Discovery/DNS](https://github.com/danielmiessler/SecLists/tree/master/Discovery/DNS)

और भी अच्छे DNS रेज़ॉल्वरों के IP। विश्वसनीय DNS रेज़ॉल्वरों की सूची उत्पन्न करने के लिए आप [https://public-dns.info/nameservers-all.txt](https://public-dns.info/nameservers-all.txt) से रेज़ॉल्वर डाउनलोड कर सकते हैं और [**dnsvalidator**](https://github.com/vortexau/dnsvalidator) का उपयोग करके उन्हें फ़िल्टर कर सकते हैं। या आप इस्तेमाल कर सकते हैं: [https://raw.githubusercontent.com/trickest/resolvers/main/resolvers-trusted.txt](https://raw.githubusercontent.com/trickest/resolvers/main/resolvers-trusted.txt)

DNS ब्रूट फोर्स के लिए सबसे अधिक सिफारिश की जाने वाली टूल्स हैं:

* [**massdns**](https://github.com/blechschmidt/massdns): यह पहला टूल था जो एक प्रभावी DNS ब्रूट फोर्स करता था। यह बहुत तेज़ है हालांकि यह गलत सक्षम होता है।
```bash
sed 's/$/.domain.com/' subdomains.txt > bf-subdomains.txt
./massdns -r resolvers.txt -w /tmp/results.txt bf-subdomains.txt
grep -E "tesla.com. [0-9]+ IN A .+" /tmp/results.txt
```
* [**gobuster**](https://github.com/OJ/gobuster): इसमें मुझे लगता है कि केवल 1 रिज़ॉल्वर का उपयोग होता है।
```
gobuster dns -d mysite.com -t 50 -w subdomains.txt
```
* [**shuffledns**](https://github.com/projectdiscovery/shuffledns) एक `massdns` के आसपास लिखी गई एक व्रैपर है, जो आपको सक्रिय ब्रूटफोर्स का उपयोग करके मान्य सबडोमेन्स की गणना करने, साथ ही वाइल्डकार्ड हैंडलिंग और आसान इनपुट-आउटपुट समर्थन के साथ सबडोमेन्स को हल करने की अनुमति देता है।
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
### दूसरे DNS Brute-Force दौर

ओपन स्रोतों और brute-forcing का उपयोग करके सबडोमेन्स को खोजने के बाद, आप पाए गए सबडोमेन्स के अतिरिक्त परिवर्तन उत्पन्न कर सकते हैं ताकि और भी खोज सकें। इस उद्देश्य के लिए कई टूल उपयोगी होते हैं:

* [**dnsgen**](https://github.com/ProjectAnte/dnsgen)**:** डोमेन और सबडोमेन्स को दिए जाने पर परिवर्तन उत्पन्न करें।
```bash
cat subdomains.txt | dnsgen -
```
* [**goaltdns**](https://github.com/subfinder/goaltdns): डोमेन और सबडोमेन दिए गए होने पर परिणाम उत्पन्न करें।
* आप यहां [**वर्डलिस्ट**](https://github.com/subfinder/goaltdns/blob/master/words.txt) में goaltdns परिणाम प्राप्त कर सकते हैं।
```bash
goaltdns -l subdomains.txt -w /tmp/words-permutations.txt -o /tmp/final-words-s3.txt
```
* [**gotator**](https://github.com/Josue87/gotator)**:** डोमेन और सबडोमेन दिए गए होने पर परिणामों को उत्पन्न करें। यदि कोई परिणाम फ़ाइल निर्दिष्ट नहीं की गई है, तो gotator अपनी फ़ाइल का उपयोग करेगा।
```
gotator -sub subdomains.txt -silent [-perm /tmp/words-permutations.txt]
```
* [**altdns**](https://github.com/infosec-au/altdns): उपनामों के परिवर्तन उत्पन्न करने के अलावा, यह उन्हें हल करने की कोशिश भी कर सकता है (लेकिन पिछले टूल का उपयोग करना बेहतर होता है।)
* आप यहां [**अल्टडीएनएस**](https://github.com/infosec-au/altdns/blob/master/words.txt) में उपनामों के परिवर्तन की **वर्डलिस्ट** प्राप्त कर सकते हैं।
```
altdns -i subdomains.txt -w /tmp/words-permutations.txt -o /tmp/asd3
```
* [**dmut**](https://github.com/bp0lr/dmut): उपनामों के परिवर्तन, म्यूटेशन और बदलाव करने के लिए एक औजार। यह औजार परिणाम को ब्रूट फोर्स करेगा (यह dns वाइल्डकार्ड का समर्थन नहीं करता है)।
* आप यहां [**यहां**](https://raw.githubusercontent.com/bp0lr/dmut/main/words.txt) dmut परिवर्तन शब्दसूची प्राप्त कर सकते हैं।
```bash
cat subdomains.txt | dmut -d /tmp/words-permutations.txt -w 100 \
--dns-errorLimit 10 --use-pb --verbose -s /tmp/resolvers-trusted.txt
```
* [**alterx**](https://github.com/projectdiscovery/alterx)**:** एक डोमेन पर आधारित होकर यह **नए संभावित सबडोमेन नाम** उत्पन्न करता है जो नए सबडोमेन खोजने के लिए निर्दिष्ट पैटर्न पर आधारित होते हैं।

#### स्मार्ट पर्म्युटेशन उत्पन्न

* [**regulator**](https://github.com/cramppet/regulator): अधिक जानकारी के लिए इस [**पोस्ट**](https://cramppet.github.io/regulator/index.html) को पढ़ें, लेकिन यह मुख्य भागों को **खोजे गए सबडोमेन** से प्राप्त करेगा और उन्हें मिश्रित करके अधिक सबडोमेन खोजेगा।
```bash
python3 main.py adobe.com adobe adobe.rules
make_brute_list.sh adobe.rules adobe.brute
puredns resolve adobe.brute --write adobe.valid
```
* [**subzuf**](https://github.com/elceef/subzuf)**:** _subzuf_ एक सबडोमेन ब्रूट-फोर्स फज़र है जिसे एक अत्यंत सरल लेकिन प्रभावी DNS प्रतिक्रिया-निर्देशित एल्गोरिदम के साथ जोड़ा गया है। यह एक दिए गए इनपुट डेटा सेट का उपयोग करता है, जैसे एक विशेष वर्डलिस्ट या ऐतिहासिक DNS/TLS रिकॉर्ड, ताकि DNS स्कैन के दौरान एकत्रित जानकारी के आधार पर और अधिक संबंधित डोमेन नामों को सही ढंग से संश्लेषित करें और उन्हें और भी आगे बढ़ाएं।
```
echo www | subzuf facebook.com
```
### **सबडोमेन खोज वर्कफ़्लो**

इस ब्लॉग पोस्ट को देखें जिसमें मैंने लिखा है कि कैसे मैं अपने कंप्यूटर में बहुत सारे टूल्स को मैन्युअल लॉन्च करने की जगह **ट्रिकेस्ट वर्कफ़्लो का उपयोग करके सबडोमेन खोज को स्वचालित कर सकता हूँ**:

{% embed url="https://trickest.com/blog/full-subdomain-discovery-using-workflow/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

{% embed url="https://trickest.com/blog/full-subdomain-brute-force-discovery-using-workflow/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

### **VHosts / वर्चुअल होस्ट्स**

यदि आपको किसी आईपी पते में **एक या एक से अधिक वेब पेज** मिलते हैं जो सबडोमेन के हैं, तो आप उस आईपी में **अन्य सबडोमेन ढूंढने का प्रयास कर सकते हैं** जिसमें वेब होस्ट कर रहे डोमेनों को **ओएसआईएनटी स्रोतों में ढूंढकर** या **उस आईपी में VHost डोमेन नामों को ब्रूट-फ़ोर्स करके**।

#### ओएसआईएनटी

आप कुछ **आईपी में VHosts को ढूंढकर** [**HostHunter**](https://github.com/SpiderLabs/HostHunter) **या अन्य एपीआई का उपयोग करके** कर सकते हैं।

**ब्रूट फ़ोर्स**

यदि आपको लगता है कि किसी सबडोमेन को वेब सर्वर में छिपा हुआ हो सकता है, तो आप इसे ब्रूट फ़ोर्स करने का प्रयास कर सकते हैं:
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

कभी-कभी आप पृष्ठों को भी मिलेगा जो केवल तब _**Access-Control-Allow-Origin**_ हेडर को वापस लौटाते हैं जब एक मान्य डोमेन/उप-डोमेन _**Origin**_ हेडर में सेट होता है। इन स्थितियों में, आप इस व्यवहार का दुरुपयोग करके नए **उप-डोमेन** की **खोज** कर सकते हैं।
```bash
ffuf -w subdomains-top1million-5000.txt -u http://10.10.10.208 -H 'Origin: http://FUZZ.crossfit.htb' -mr "Access-Control-Allow-Origin" -ignore-body
```
### **बकेट्स ब्रूट फोर्स**

**सबडोमेन्स** की तलाश करते समय ध्यान दें कि क्या यह किसी प्रकार के **बकेट** को **पॉइंट कर रहा है**, और ऐसे मामले में [**अनुमतियों की जांच करें**](../../network-services-pentesting/pentesting-web/buckets/)**।**\
इसके अलावा, इस बिंदु पर आपको पता होगा कि स्कोप में सभी डोमेन हैं, इसलिए [**संभावित बकेट नामों को ब्रूट फोर्स करें और अनुमतियों की जांच करें**](../../network-services-pentesting/pentesting-web/buckets/)।

### **मॉनिटरिंग**

आप यह देख सकते हैं कि क्या कोई डोमेन के **नए सबडोमेन** बनाए जाते हैं, **सर्टिफिकेट ट्रांसपेरेंसी** लॉग्स [**sublert** ](https://github.com/yassineaboukir/sublert/blob/master/sublert.py) के माध्यम से जांच करके।

### **दुर्बलताओं की तलाश**

[**सबडोमेन टेकओवर**](../../pentesting-web/domain-subdomain-takeover.md#subdomain-takeover) के लिए जांचें।\
यदि **सबडोमेन** किसी **S3 बकेट** को पॉइंट कर रहा है, तो [**अनुमतियों की जांच करें**](../../network-services-pentesting/pentesting-web/buckets/)।

यदि आपको assets discovery में पहले से पाए गए IP के अलावा कोई **सबडोमेन अलग IP के साथ** मिलता है, तो आपको एक **बेसिक वल्नरेबिलिटी स्कैन** (Nessus या OpenVAS का उपयोग करके) और कुछ [**पोर्ट स्कैन**](../pentesting-network/#discovering-hosts-from-the-outside) नंप/masscan/shodan के साथ करना चाहिए। चलिए देखते हैं कि इन सेवाओं को "हमला" करने के लिए इस किताब में कुछ ट्रिक्स हैं।\
_ध्यान दें कि कभी-कभी सबडोमेन को क्लाइंट द्वारा नियंत्रित नहीं किए जा रहे IP में होस्ट किया जाता है, इसलिए यह स्कोप में नहीं है, सतर्क रहें।_

## IPs

प्रारंभिक चरणों में आपने **कुछ IP रेंज, डोमेन और सबडोमेन पाए होंगे**।\
अब उन रेंजों से **सभी IPs को एकत्र करें** और **डोमेन/सबडोमेन (DNS क्वेरीज)** के लिए।

निम्नलिखित **मुफ्त एपीआई** सेवाओं का उपयोग करके आप डोमेन और सबडोमेन द्वारा पहले से उपयोग किए गए **पिछले IPs** भी खोज सकते हैं। ये IPs अभी भी क्लाइंट के पास हो सकते हैं (और आपको [**CloudFlare bypasses**](../../network-services-pentesting/pentesting-web/uncovering-cloudflare.md) खोजने की अनुमति दे सकते हैं)

* [**https://securitytrails.com/**](https://securitytrails.com/)

आप टूल [**hakip2host**](https://github.com/hakluke/hakip2host) का उपयोग करके एक विशिष्ट IP पते पर पॉइंट करने वाले डोमेन की जांच भी कर सकते हैं।

### **दुर्बलताओं की तलाश**

**CDN के बाहर के सभी IPs को पोर्ट स्कैन** करें (क्योंकि आपको उनमें कुछ भी रुचिकर नहीं मिलेगा)। खोजी गई चल रही सेवाओं में आप **दुर्बलताएं खोज सकते हैं**।

**होस्ट स्कैन करने के लिए** [**एक गाइड**](../pentesting-network/) **खोजें।**

## वेब सर्वर्स की खोज

> हमने सभी कंपनियों और उनके एसेट्स को खोज लिया है और हमें स्कोप में आईपी रेंज, डोमेन और सबडोमेन पता है। अब वेब सर्वर्स की खोज करने का समय है।

पिछले चरणों में आपने शायद पहले से ही खोज की होगी **खोज की होगी** तो आपको **सभी संभावित वेब सर्वर्स** पहले से ही मिल गए होंगे। हालांकि, यदि आपने नहीं किया है तो अब हम कुछ **तेज़ तरीके देखेंगे जिनसे स्कोप के अंदर वेब सर्वर्स खोजें**।

कृपया ध्यान दें कि यह **वेब ऐप्स खोज के लिए उन्मुख** होगा, इसलिए आपको **दुर्बलता** और **पोर्ट स्कैनिंग** भी करनी चाहिए (**यदि स्कोप द्वारा अनुमति हो**)।

[**masscan का उपयोग करके वेब सर्वर्स** के संबंधित **खुले पोर्ट्स की खोज करने के लिए एक तेज़ तरीका** यहां मिल सकता है](../pentesting-network/#http-port-discovery)।\
वेब सर्वर्स की खोज करने के लिए एक और उपयोगी टूल है [**httprobe**](https://github.com/tomnomnom/httprobe)**,** [**fprobe**](https://github.com/theblackturtle/fprobe) और [**httpx**](https://github.com/projectdiscovery/httpx)। आप एक डोमेनों की सूची पास करते हैं और यह 80 (http) और 443 (https) पोर्ट से कनेक्ट करने की कोशिश करेगा। इसके अलावा, आप अन्य पोर्ट की कोशिश करने के लिए इंगित कर सकते हैं:
```bash
cat /tmp/domains.txt | httprobe #Test all domains inside the file for port 80 and 443
cat /tmp/domains.txt | httprobe -p http:8080 -p https:8443 #Check port 80, 443 and 8080 and 8443
```
### **स्क्रीनशॉट्स**

अब जब आपने स्कोप में मौजूद सभी वेब सर्वरों की पता लगा ली है (कंपनी के IPs, सभी डोमेन और सबडोमेन के बीच) तो शायद आपको पता नहीं होगा कि शुरुआत कहां से करें। इसलिए, आइए इसे सरल बनाते हैं और सभी की स्क्रीनशॉट लेने से शुरू करें। मुख्य पृष्ठ पर देखकर आप ऐसे अंत-बिंदुओं को खोज सकते हैं जो अधिक संभावित हैं कि वे कमजोर होंगे।

प्रस्तावित विचार को कार्यान्वित करने के लिए आप [**EyeWitness**](https://github.com/FortyNorthSecurity/EyeWitness), [**HttpScreenshot**](https://github.com/breenmachine/httpscreenshot), [**Aquatone**](https://github.com/michenriksen/aquatone), [**Shutter**](https://shutter-project.org/downloads/third-party-packages/) या [**webscreenshot**](https://github.com/maaaaz/webscreenshot) का उपयोग कर सकते हैं।

इसके अलावा, आप फिर [**eyeballer**](https://github.com/BishopFox/eyeballer) का उपयोग करके सभी स्क्रीनशॉट पर चल सकते हैं ताकि आपको पता चल सके कि कौन सी संभावना है कि कमजोरियां हो सकती हैं और कौन नहीं।

## सार्वजनिक क्लाउड संपत्ति

किसी कंपनी की संभावित क्लाउड संपत्ति खोजने के लिए आपको उस कंपनी की पहचान करने वाले शब्दों की सूची के साथ शुरुआत करनी चाहिए। उदाहरण के लिए, एक क्रिप्टो कंपनी के लिए आप "क्रिप्टो", "वॉलेट", "डाओ", "<डोमेन_नाम>", "<सबडोमेन_नाम>" जैसे शब्दों का उपयोग कर सकते हैं।

आपको बाल्टी में उपयोग होने वाले सामान्य शब्दों की सूचियों की भी आवश्यकता होगी:

* [https://raw.githubusercontent.com/cujanovic/goaltdns/master/words.txt](https://raw.githubusercontent.com/cujanovic/goaltdns/master/words.txt)
* [https://raw.githubusercontent.com/infosec-au/altdns/master/words.txt](https://raw.githubusercontent.com/infosec-au/altdns/master/words.txt)
* [https://raw.githubusercontent.com/jordanpotti/AWSBucketDump/master/BucketNames.txt](https://raw.githubusercontent.com/jordanpotti/AWSBucketDump/master/BucketNames.txt)

फिर, उन शब्दों के साथ आपको **परिणामस्वरूप परिवर्तन** उत्पन्न करने चाहिए (अधिक जानकारी के लिए [**दूसरे दूसरे DNS ब्रूट-फोर्स**](./#second-dns-bruteforce-round) देखें)।

परिणामस्वरूप शब्द सूचियों के साथ आप [**cloud\_enum**](https://github.com/initstring/cloud\_enum), [**CloudScraper**](https://github.com/jordanpotti/CloudScraper), [**cloudlist**](https://github.com/projectdiscovery/cloudlist) या [**S3Scanner**](https://github.com/sa7mon/S3Scanner) जैसे उपकरणों का उपयोग कर सकते हैं।

ध्यान दें कि क्लाउड संपत्ति खोजने के लिए आपको **AWS में बाल्टी के अलावा भी देखना चाहिए**।

### **कमजोरियों की खोज**

यदि आपको खुले बाल्टी या खुले क्लाउड फंक्शन जैसी चीजें मिलती हैं, तो आपको उन्हें **एक्सेस** करना चाहिए और देखने की कोशिश करनी चाहिए कि वे आपको क्या प्रदान करते हैं और क्या आप उनका दुरुपयोग कर सकते हैं।

## ईमेल

स्कोप में मौजूद डोमेन और सबडोमेन के साथ आपके पास मौजूद हैं वह सब कुछ जिसकी आपको ईमेल खोजने की आवश्यकता होती है। एक कंपनी के ईमेल खोजने के लिए निम्नलिखित **APIs** और **उपकरण** मेरे लिए सबसे अच्छे काम कर चुके हैं:

* [**theHarvester**](https://github.com/laramies/theHarvester) - APIs के साथ
* [**https://hunter.io/**](https://hunter.io/) (मुफ्त संस्करण) का API
* [**https://app.snov.io/**](https://app.snov.io/) (मुफ्त संस्करण) का API
* [**https://minelead.io/**](https://minelead.io/) (मुफ्त संस्करण) का API

### **कमजोरियों की खोज**

ईमेल बाद में **वेब लॉगिन और प्रमाणीकरण सेवाओं** (जैसे SSH) को ब्रूट-फोर्स करने के लिए उपयोगी होंगे। इसके अलावा, ये **फिशिंग** के लिए आवश्यक होते हैं। इसके अलावा, ये APIs आपको ईमेल के पीछे के व्यक्ति के बारे में और अधिक **जानकारी** देंगे, जो फिशिंग अभियान के लिए उपयोगी होती है।

## क्रेडेंशियल लीक

डोमेन, सबडोमेन और ईमेल के साथ आप उन ईमेलों के लिए लीक हुए क्रेडेंशियल खोजना शुरू कर सकते हैं जो पहले से ही उन ईमेलों के हिस्से थे:

* [https://leak-lookup.com](https://leak-lookup.com/account/login)
* [https://www.dehashed.com/](https
### **दुर्बलताओं की खोज**

यदि आपको **मान्य लीक** क्रेडेंशियल या API टोकन मिल जाते हैं, तो यह बहुत आसान जीत होती है।

## सार्वजनिक कोड की दुर्बलताएं

यदि आपको पता चलता है कि कंपनी के पास **ओपन-सोर्स कोड** है, तो आप इसे **विश्लेषण** कर सकते हैं और इसमें **दुर्बलताएं** खोज सकते हैं।

**भाषा के आधार पर** आपके पास विभिन्न **उपकरण** हो सकते हैं:

{% content-ref url="../../network-services-pentesting/pentesting-web/code-review-tools.md" %}
[code-review-tools.md](../../network-services-pentesting/pentesting-web/code-review-tools.md)
{% endcontent-ref %}

ऐसे भी मुफ्त सेवाएं हैं जो आपको **सार्वजनिक रिपॉजिटरी की स्कैनिंग** करने की अनुमति देती हैं, जैसे:

* [**Snyk**](https://app.snyk.io/)

## [**वेब पेंटेस्टिंग मेथडोलॉजी**](../../network-services-pentesting/pentesting-web/)

बग हंटर्स द्वारा पाए गए **बहुमत** वेब एप्लिकेशन में होती हैं, इसलिए इस बिंदु पर मैं एक **वेब एप्लिकेशन टेस्टिंग मेथडोलॉजी** के बारे में बात करना चाहूंगा, और आप इस [**जानकारी को यहां पा सकते हैं**](../../network-services-pentesting/pentesting-web/)।

मैं यहां [**वेब स्वचालित स्कैनर ओपन सोर्स उपकरण**](../../network-services-pentesting/pentesting-web/#automatic-scanners) के बारे में एक विशेष उल्लेख भी करना चाहूंगा, क्योंकि, यदि आप उन्हें आपसी रूप से बहुत संवेदनशील दुर्बलताओं को खोजने की उम्मीद नहीं करते हैं, तो वे **कुछ प्राथमिक वेब जानकारी को लागू करने के लिए उपयोगी होते हैं।**

## संक्षेपण

> बधाई हो! इस बिंदु पर आपने पहले से ही **सभी मूल संख्यान** को पूरा कर लिया है। हाँ, यह मूल है क्योंकि और भी बहुत सारे संख्यान किए जा सकते हैं (बाद में और ट्रिक्स देखेंगे)।

तो आपने पहले से ही किया है:

1. स्कोप में सभी **कंपनियों** को खोज लिया है
2. कंपनियों के सभी **संपत्तियों** को खोज लिया है (और यदि स्कोप में है तो कुछ वल्न स्कैन भी किया है)
3. कंपनियों के सभी **डोमेन** को खोज लिया है
4. डोमेन के सभी **सबडोमेन** को खोज लिया है (कोई सबडोमेन ओवरटेकओवर?)
5. स्कोप में से सभी **आईपी** (सीडीएन के नहीं) को खोज लिया है।
6. स्कोप में सभी **वेब सर्वर** को खोज लिया है और उनकी **स्क्रीनशॉट** ली है (कुछ अजीब चीजें हैं जिन्हें गहरी जांच के लिए देखने के लिए?)
7. कंपनी के सभी **संभावित सार्वजनिक क्लाउड संपत्तियों** को खोज लिया है।
8. **ईमेल**, **क्रेडेंशियल लीक**, और **सीक्रेट लीक** जो आपको **बहुत आसानी से बड़ी जीत दे सकते हैं**।
9. आपने पाए सभी वेब को **पेंटेस्ट** किया है

## **पूर्ण संख्यान स्वचालित उपकरण**

वहां कई उपकरण हैं जो दिए गए स्कोप के खिलाफ प्रस्तावित कार्रवाई का हिस्सा करेंगे।

* [**https://github.com/yogeshojha/rengine**](https://github.com/yogeshojha/rengine)
* [**https://github.com/j3ssie/Osmedeus**](https://github.com/j3ssie/Osmedeus)
* [**https://github.com/six2dez/reconftw**](https://github.com/six2dez/reconftw)
* [**https://github.com/hackerspider1/EchoPwn**](https://github.com/hackerspider1/EchoPwn) - थोड़ा पुराना है और अद्यतित नहीं है

## **संदर्भ**

* [**@Jhaddix**](https://twitter.com/Jhaddix) के **सभी मुफ्त कोर्सेज** (जैसे [**The Bug Hunter's Methodology v4.0 - Recon Edition**](https://www.youtube.com/watch?v=p4JgIu1mceI)**)**
* **बग बाउंटी टिप**: **Intigriti** में **साइन अप** करें, एक प्रीमियम **बग बाउंटी प्लेटफॉर्म जो हैकर्स द्वारा बनाई गई है**! आज ही [**https://go.intigriti.com/hacktricks**](https://go.intigriti.com/hacktricks) पर हमारे साथ जुड़ें, और **$100,000** तक की इनामी प्राप्त करना शुरू करें!

{% embed url="https://go.intigriti.com/hacktricks" %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करत
