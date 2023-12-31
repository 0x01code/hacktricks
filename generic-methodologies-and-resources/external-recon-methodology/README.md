# बाहरी रेकॉन मेथडोलॉजी

<details>

<summary><strong>AWS हैकिंग सीखें शुरुआत से लेकर एक्सपर्ट बनने तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) को **फॉलो करें**.
* **अपनी हैकिंग ट्रिक्स साझा करें** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके.

</details>

<img src="../../.gitbook/assets/i3.png" alt="" data-size="original">\
**Bug bounty tip**: **Intigriti** के लिए **साइन अप** करें, एक प्रीमियम **bug bounty platform जो हैकर्स द्वारा, हैकर्स के लिए बनाया गया है**! आज ही हमसे [**https://go.intigriti.com/hacktricks**](https://go.intigriti.com/hacktricks) पर जुड़ें, और **$100,000** तक के बाउंटीज कमाना शुरू करें!

{% embed url="https://go.intigriti.com/hacktricks" %}

## संपत्तियों की खोज

> तो आपको बताया गया था कि किसी कंपनी से संबंधित सभी चीजें स्कोप के अंदर हैं, और आप यह पता लगाना चाहते हैं कि इस कंपनी के पास वास्तव में क्या है।

इस चरण का उद्देश्य मुख्य कंपनी द्वारा स्वामित्व वाली सभी **कंपनियों** को प्राप्त करना है और फिर इन कंपनियों की सभी **संपत्तियों** को। इसके लिए, हम:

1. मुख्य कंपनी के अधिग्रहणों को ढूंढेंगे, जिससे हमें स्कोप के अंदर की कंपनियां मिलेंगी।
2. प्रत्येक कंपनी के ASN (यदि कोई हो) को ढूंढेंगे, जिससे हमें प्रत्येक कंपनी द्वारा स्वामित्व वाली IP रेंजेस मिलेंगी।
3. पहले वाले से संबंधित अन्य प्रविष्टियों (संगठन नाम, डोमेन...) की खोज के लिए रिवर्स व्होइस लुकअप्स का उपयोग करेंगे (यह पुनरावृत्ति के साथ किया जा सकता है)
4. अन्य संपत्तियों की खोज के लिए shodan `org` और `ssl` फिल्टर्स जैसी अन्य तकनीकों का उपयोग करेंगे (यह `ssl` ट्रिक पुनरावृत्ति के साथ की जा सकती है)।

### **अधिग्रहण**

सबसे पहले, हमें यह जानना होगा कि **मुख्य कंपनी द्वारा कौन सी अन्य कंपनियों का स्वामित्व है**।\
एक विकल्प है [https://www.crunchbase.com/](https://www.crunchbase.com) पर जाना, **मुख्य कंपनी** के लिए **खोज** करना, और "**अधिग्रहण**" पर **क्लिक** करना। वहां आपको मुख्य कंपनी द्वारा अधिग्रहित अन्य कंपनियां दिखाई देंगी।\
दूसरा विकल्प है मुख्य कंपनी के **Wikipedia** पेज पर जाना और **अधिग्रहण** के लिए खोज करना।

> ठीक है, इस बिंदु पर आपको स्कोप के अंदर सभी कंपनियों के बारे में पता होना चाहिए। चलिए उनकी संपत्तियों को ढूंढने का तरीका जानते हैं।

### **ASNs**

एक स्वायत्त प्रणाली संख्या (**ASN**) एक **अद्वितीय संख्या** होती है जो **इंटरनेट असाइन्ड नंबर्स अथॉरिटी (IANA)** द्वारा एक **स्वायत्त प्रणाली** (AS) को असाइन की जाती है।\
एक **AS** में **IP पतों के ब्लॉक** होते हैं जिनकी बाहरी नेटवर्क्स तक पहुंचने के लिए एक स्पष्ट नीति होती है और जिन्हें एक ही संगठन द्वारा प्रशासित किया जाता है लेकिन इसमें कई ऑपरेटर्स हो सकते हैं।

यह जानना दिलचस्प है कि क्या **कंपनी को कोई ASN असाइन किया गया है** ताकि उसकी **IP रेंजेस** का पता चल सके। यह **स्कोप** के अंदर सभी **होस्ट्स** के खिलाफ एक **वल्नरेबिलिटी टेस्ट** करने और इन IPs के अंदर **डोमेन्स की खोज** करने के लिए दिलचस्प होगा।\
आप [**https://bgp.he.net/**](https://bgp.he.net) पर कंपनी **नाम**, **IP** या **डोमेन** द्वारा **खोज** कर सकते हैं।\
**कंपनी के क्षेत्र के आधार पर ये लिंक्स अधिक डेटा एकत्र करने के लिए उपयोगी हो सकते हैं:** [**AFRINIC**](https://www.afrinic.net) **(अफ्रीका),** [**Arin**](https://www.arin.net/about/welcome/region/)**(उत्तरी अमेरिका),** [**APNIC**](https://www.apnic.net) **(एशिया),** [**LACNIC**](https://www.lacnic.net) **(लैटिन अमेरिका),** [**RIPE NCC**](https://www.ripe.net) **(यूरोप)। वैसे भी, शायद सभी** उपयोगी जानकारी **(IP रेंजेस और Whois)** पहले लिंक में ही दिखाई दे जाएगी।
```bash
#You can try "automate" this with amass, but it's not very recommended
amass intel -org tesla
amass intel -asn 8911,50313,394161
```
इसके अलावा, [**BBOT**](https://github.com/blacklanternsecurity/bbot) का सबडोमेन एन्यूमरेशन स्वचालित रूप से ASNs को स्कैन के अंत में संग्रहीत और सारांशित करता है।
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
आप [http://asnlookup.com/](http://asnlookup.com) का उपयोग करके भी किसी संगठन की IP रेंज पा सकते हैं (इसमें मुफ्त API है)।\
आप [http://ipv4info.com/](http://ipv4info.com) का उपयोग करके किसी डोमेन की IP और ASN पा सकते हैं।

### **कमजोरियों की तलाश**

इस बिंदु पर हमें **स्कोप के अंदर सभी एसेट्स का पता** चल गया है, इसलिए अगर आपको अनुमति है तो आप सभी होस्ट्स पर कुछ **वल्नरेबिलिटी स्कैनर** (Nessus, OpenVAS) लॉन्च कर सकते हैं।\
साथ ही, आप कुछ [**पोर्ट स्कैन्स**](../pentesting-network/#discovering-hosts-from-the-outside) लॉन्च कर सकते हैं **या shodan जैसी सेवाओं का उपयोग करके** खुले पोर्ट्स **पा सकते हैं और जो कुछ भी आप पाते हैं उसके आधार पर आपको इस पुस्तक में देखना चाहिए कि कैसे कई संभावित सेवाओं का पेंटेस्ट किया जाए।**\
**यह भी उल्लेखनीय हो सकता है कि आप कुछ** default username **और** passwords **की सूची तैयार कर सकते हैं और [https://github.com/x90skysn3k/brutespray](https://github.com/x90skysn3k/brutespray) का उपयोग करके सेवाओं को** bruteforce करने की कोशिश कर सकते हैं।

## डोमेन्स

> हमें स्कोप के अंदर सभी कंपनियों और उनके एसेट्स का पता है, अब स्कोप के अंदर डोमेन्स की तलाश करने का समय है।

_कृपया ध्यान दें कि निम्नलिखित प्रस्तावित तकनीकों में आप सबडोमेन्स भी पा सकते हैं और उस जानकारी को कम नहीं आंका जाना चाहिए।_

सबसे पहले आपको प्रत्येक कंपनी के **मुख्य डोमेन**(s) की तलाश करनी चाहिए। उदाहरण के लिए, _Tesla Inc._ के लिए यह _tesla.com_ होगा।

### **रिवर्स DNS**

जैसा कि आपने डोमेन्स की सभी IP रेंज पा ली हैं, आप उन **IPs पर रिवर्स dns लुकअप्स** करने की कोशिश कर सकते हैं ताकि **स्कोप के अंदर और अधिक डोमेन्स पा सकें**। पीड़ित के कुछ dns सर्वर या कुछ प्रसिद्ध dns सर्वर (1.1.1.1, 8.8.8.8) का उपयोग करने की कोशिश करें।
```bash
dnsrecon -r <DNS Range> -n <IP_DNS>   #DNS reverse of all of the addresses
dnsrecon -d facebook.com -r 157.240.221.35/24 #Using facebooks dns
dnsrecon -r 157.240.221.35/24 -n 1.1.1.1 #Using cloudflares dns
dnsrecon -r 157.240.221.35/24 -n 8.8.8.8 #Using google dns
```
इसके लिए काम करने के लिए, प्रशासक को मैन्युअल रूप से PTR सक्षम करना होगा।\
आप इस जानकारी के लिए एक ऑनलाइन टूल भी उपयोग कर सकते हैं: [http://ptrarchive.com/](http://ptrarchive.com)

### **Reverse Whois (पाश)**

**whois** के अंदर आपको बहुत सारी दिलचस्प **जानकारी** मिल सकती है जैसे कि **संगठन का नाम**, **पता**, **ईमेल**, फोन नंबर... लेकिन जो और भी अधिक रोचक है वह यह है कि आप **कंपनी से संबंधित अधिक संपत्तियों** को खोज सकते हैं यदि आप **उन क्षेत्रों में से किसी एक द्वारा reverse whois lookups करते हैं** (उदाहरण के लिए अन्य whois रजिस्ट्री जहां वही ईमेल दिखाई देता है)।\
आप ऑनलाइन टूल्स का उपयोग कर सकते हैं जैसे:

* [https://viewdns.info/reversewhois/](https://viewdns.info/reversewhois/) - **मुफ्त**
* [https://domaineye.com/reverse-whois](https://domaineye.com/reverse-whois) - **मुफ्त**
* [https://www.reversewhois.io/](https://www.reversewhois.io) - **मुफ्त**
* [https://www.whoxy.com/](https://www.whoxy.com) - **मुफ्त** वेब, API के लिए मुफ्त नहीं।
* [http://reversewhois.domaintools.com/](http://reversewhois.domaintools.com) - मुफ्त नहीं
* [https://drs.whoisxmlapi.com/reverse-whois-search](https://drs.whoisxmlapi.com/reverse-whois-search) - मुफ्त नहीं (केवल **100 मुफ्त** खोजें)
* [https://www.domainiq.com/](https://www.domainiq.com) - मुफ्त नहीं

आप इस कार्य को [**DomLink**](https://github.com/vysecurity/DomLink) (whoxy API कुंजी की आवश्यकता होती है) का उपयोग करके स्वचालित कर सकते हैं।\
आप [amass](https://github.com/OWASP/Amass) के साथ कुछ स्वचालित reverse whois खोज भी कर सकते हैं: `amass intel -d tesla.com -whois`

**ध्यान दें कि जब भी आप एक नया डोमेन पाते हैं, आप इस तकनीक का उपयोग अधिक डोमेन नामों की खोज के लिए कर सकते हैं।**

### **Trackers**

यदि आप दो अलग-अलग पृष्ठों पर **एक ही ट्रैकर की समान ID** पाते हैं, तो आप मान सकते हैं कि **दोनों पृष्ठों** का प्रबंधन **एक ही टीम** द्वारा किया जाता है।\
उदाहरण के लिए, यदि आप कई पृष्ठों पर एक ही **Google Analytics ID** या एक ही **Adsense ID** देखते हैं।

कुछ पृष्ठ और टूल्स हैं जो आपको इन ट्रैकर्स और अधिक के द्वारा खोजने की अनुमति देते हैं:

* [**Udon**](https://github.com/dhn/udon)
* [**BuiltWith**](https://builtwith.com)
* [**Sitesleuth**](https://www.sitesleuth.io)
* [**Publicwww**](https://publicwww.com)
* [**SpyOnWeb**](http://spyonweb.com)

### **Favicon**

क्या आप जानते हैं कि हम एक ही favicon आइकन हैश की तलाश करके हमारे लक्ष्य से संबंधित डोमेन और सब डोमेन पा सकते हैं? यही काम [@m4ll0k2](https://twitter.com/m4ll0k2) द्वारा बनाए गए [favihash.py](https://github.com/m4ll0k/Bug-Bounty-Toolz/blob/master/favihash.py) टूल द्वारा किया जाता है। इसका उपयोग कैसे करें यहाँ है:
```bash
cat my_targets.txt | xargs -I %% bash -c 'echo "http://%%/favicon.ico"' > targets.txt
python3 favihash.py -f https://target/favicon.ico -t targets.txt -s
```
```markdown
![favihash - उसी favicon icon hash वाले डोमेन्स की खोज करें](https://www.infosecmatter.com/wp-content/uploads/2020/07/favihash.jpg)

सरल शब्दों में, favihash हमें उन डोमेन्स की खोज करने की अनुमति देगा जिनका favicon icon hash हमारे लक्ष्य के समान है।

इसके अलावा, आप favicon hash का उपयोग करके तकनीकों की खोज भी कर सकते हैं जैसा कि [**इस ब्लॉग पोस्ट**](https://medium.com/@Asm0d3us/weaponizing-favicon-ico-for-bugbounties-osint-and-what-not-ace3c214e139) में समझाया गया है। इसका मतलब है कि यदि आपको किसी वेब तकनीक के कमजोर संस्करण के favicon का **hash पता है** तो आप shodan में खोज कर सकते हैं और **अधिक कमजोर स्थानों का पता लगा सकते हैं**:
```
```bash
shodan search org:"Target" http.favicon.hash:116323821 --fields ip_str,port --separator " " | awk '{print $1":"$2}'
```
यह है कि आप कैसे वेब के **favicon hash की गणना कर सकते हैं**:
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
### **कॉपीराइट / यूनिक स्ट्रिंग**

वेब पेजों के अंदर **स्ट्रिंग्स की खोज करें जो एक ही संगठन की विभिन्न वेबसाइटों में साझा की जा सकती हैं**। **कॉपीराइट स्ट्रिंग** एक अच्छा उदाहरण हो सकता है। फिर उस स्ट्रिंग को **गूगल** में, अन्य **ब्राउज़रों** में या यहां तक कि **शोडन** में खोजें: `shodan search http.html:"कॉपीराइट स्ट्रिंग"`

### **CRT समय**

आमतौर पर एक क्रॉन जॉब होता है जैसे
```bash
# /etc/crontab
37 13 */10 * * certbot renew --post-hook "systemctl reload nginx"
```
सर्वर पर सभी डोमेन प्रमाणपत्रों को नवीनीकृत करने के लिए। इसका मतलब है कि यहां तक कि अगर CA इसके लिए समय नहीं निर्धारित करता है, तो भी प्रमाणपत्र पारदर्शिता लॉग्स में **एक ही कंपनी से संबंधित डोमेन्स का पता लगाना संभव है**।
इस [**लेखन को अधिक जानकारी के लिए देखें**](https://swarm.ptsecurity.com/discovering-domains-via-a-time-correlation-attack/).

### **पैसिव टेकओवर**

ऐसा प्रतीत होता है कि लोग अक्सर सबडोमेन्स को क्लाउड प्रोवाइडर्स के IP पतों से जोड़ते हैं और किसी बिंदु पर **उस IP पते को खो देते हैं लेकिन DNS रिकॉर्ड हटाना भूल जाते हैं**। इसलिए, बस **एक VM को स्पॉन करके** क्लाउड में (जैसे कि Digital Ocean) आप वास्तव में **कुछ सबडोमेन्स का टेकओवर कर रहे होंगे**।

[**यह पोस्ट**](https://kmsec.uk/blog/passive-takeover/) इसके बारे में एक कहानी बताती है और एक स्क्रिप्ट का प्रस्ताव करती है जो **DigitalOcean में एक VM को स्पॉन करती है**, **नई मशीन का IPv4 प्राप्त करती है**, और **Virustotal में उसके लिए इंगित करने वाले सबडोमेन रिकॉर्ड्स की खोज करती है**।

### **अन्य तरीके**

**ध्यान दें कि जब भी आप एक नया डोमेन पाते हैं, आप इस तकनीक का उपयोग अधिक डोमेन नामों की खोज के लिए कर सकते हैं।**

**Shodan**

जैसा कि आप पहले से ही जानते हैं कि IP स्पेस के मालिक का नाम। आप उस डेटा को shodan में इस प्रकार खोज सकते हैं: `org:"Tesla, Inc."` TLS प्रमाणपत्र में नए अप्रत्याशित डोमेन्स के लिए मिले होस्ट्स की जांच करें।

आप मुख्य वेब पेज के **TLS प्रमाणपत्र** तक पहुंच सकते हैं, **संगठन का नाम** प्राप्त कर सकते हैं और फिर उस नाम की खोज **shodan** द्वारा ज्ञात सभी वेब पेजों के **TLS प्रमाणपत्रों** में कर सकते हैं फिल्टर के साथ: `ssl:"Tesla Motors"` या [**sslsearch**](https://github.com/HarshVaragiya/sslsearch) जैसे उपकरण का उपयोग कर सकते हैं।

**Assetfinder**

[**Assetfinder**](https://github.com/tomnomnom/assetfinder) एक उपकरण है जो मुख्य डोमेन से संबंधित **डोमेन्स** और उनके **सबडोमेन्स** की खोज करता है, बहुत अद्भुत।

### **कमजोरियों की खोज**

कुछ [डोमेन टेकओवर](../../pentesting-web/domain-subdomain-takeover.md#domain-takeover) की जांच करें। हो सकता है कि कोई कंपनी **किसी डोमेन का उपयोग कर रही हो** लेकिन उन्होंने **मालिकाना हक खो दिया हो**। बस इसे रजिस्टर करें (अगर पर्याप्त सस्ता हो) और कंपनी को सूचित करें।

यदि आप कोई **डोमेन अलग IP के साथ पाते हैं** जो आपको पहले से ही एसेट्स डिस्कवरी में मिले हैं, तो आपको एक **बेसिक वल्नरेबिलिटी स्कैन** (Nessus या OpenVAS का उपयोग करके) और कुछ [**पोर्ट स्कैन**](../pentesting-network/#discovering-hosts-from-the-outside) **nmap/masscan/shodan** के साथ करना चाहिए। जो सेवाएं चल रही हैं उनके आधार पर आप **इस पुस्तक में कुछ चालें "हमला" करने के लिए पा सकते हैं**।\
_ध्यान दें कि कभी-कभी डोमेन किसी ऐसे IP के अंदर होस्ट किया जाता है जो क्लाइंट द्वारा नियंत्रित नहीं होता है, इसलिए यह स्कोप में नहीं है, सावधान रहें।_

<img src="../../.gitbook/assets/i3.png" alt="" data-size="original">\
**बग बाउंटी टिप**: **Intigriti के लिए साइन अप करें**, एक प्रीमियम **बग बाउंटी प्लेटफॉर्म जो हैकर्स द्वारा, हैकर्स के लिए बनाया गया है**! हमसे आज ही [**https://go.intigriti.com/hacktricks**](https://go.intigriti.com/hacktricks) पर जुड़ें, और **$100,000** तक की बाउंटी कमाना शुरू करें!

{% embed url="https://go.intigriti.com/hacktricks" %}

## सबडोमेन्स

> हमें स्कोप के अंदर सभी कंपनियों के बारे में पता है, प्रत्येक कंपनी की सभी संपत्तियां और कंपनियों से संबंधित सभी डोमेन्स।

अब समय है प्रत्येक पाए गए डोमेन के सभी संभावित सबडोमेन्स को खोजने का।

### **DNS**

आइए **DNS** रिकॉर्ड्स से **सबडोमेन्स** प्राप्त करने की कोशिश करें। हमें **Zone Transfer** के लिए भी प्रयास करना चाहिए (यदि भेद्य है, तो आपको इसे रिपोर्ट करना चाहिए)।
```bash
dnsrecon -a -d tesla.com
```
### **OSINT**

बहुत सारे सबडोमेन्स प्राप्त करने का सबसे तेज़ तरीका बाहरी स्रोतों में खोज करना है। सबसे अधिक प्रयुक्त **उपकरण** निम्नलिखित हैं (बेहतर परिणामों के लिए API कुंजियों को कॉन्फ़िगर करें):

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
* [**vita**](https://github.com/junnlikestea/vita)
```
vita -d tesla.com
```
* [**theHarvester**](https://github.com/laramies/theHarvester)
```bash
theHarvester -d tesla.com -b "anubis, baidu, bing, binaryedge, bingapi, bufferoverun, censys, certspotter, crtsh, dnsdumpster, duckduckgo, fullhunt, github-code, google, hackertarget, hunter, intelx, linkedin, linkedin_links, n45ht, omnisint, otx, pentesttools, projectdiscovery, qwant, rapiddns, rocketreach, securityTrails, spyse, sublist3r, threatcrowd, threatminer, trello, twitter, urlscan, virustotal, yahoo, zoomeye"
```
कुछ **अन्य रोचक उपकरण/APIs** हैं जो भले ही सीधे तौर पर उपडोमेन खोजने में विशेषज्ञ न हों, फिर भी उपडोमेन खोजने के लिए उपयोगी हो सकते हैं, जैसे कि:

* [**Crobat**](https://github.com/cgboal/sonarsearch)**:** API [https://sonar.omnisint.io](https://sonar.omnisint.io) का उपयोग करके उपडोमेन प्राप्त करता है
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
* [**gau**](https://github.com/lc/gau)**:** AlienVault's Open Threat Exchange, Wayback Machine, और Common Crawl से किसी भी दिए गए डोमेन के लिए ज्ञात URLs प्राप्त करता है।
```bash
# Get subdomains from GAUs found URLs
gau --subs tesla.com | cut -d "/" -f 3 | sort -u
```
* [**SubDomainizer**](https://github.com/nsonaniya2010/SubDomainizer) **और** [**subscraper**](https://github.com/Cillian-Collins/subscraper): वे वेब को स्क्रैप करते हैं JS फाइलों की खोज में और वहां से सबडोमेन्स निकालते हैं।
```bash
# Get only subdomains from SubDomainizer
python3 SubDomainizer.py -u https://tesla.com | grep tesla.com

# Get only subdomains from subscraper, this already perform recursion over the found results
python subscraper.py -u tesla.com | grep tesla.com | cut -d " " -f
```
* [**Shodan**](https://www.shodan.io/)
```bash
# Get info about the domain
shodan domain <domain>
# Get other pages with links to subdomains
shodan search "http.html:help.domain.com"
```
* [**Censys उपडोमेन खोजक**](https://github.com/christophetd/censys-subdomain-finder)
```bash
export CENSYS_API_ID=...
export CENSYS_API_SECRET=...
python3 censys-subdomain-finder.py tesla.com
```
* [**DomainTrail.py**](https://github.com/gatete/DomainTrail)
```bash
python3 DomainTrail.py -d example.com
```
* [**securitytrails.com**](https://securitytrails.com/) में उपडोमेन्स और IP इतिहास की खोज के लिए मुफ्त API है
* [**chaos.projectdiscovery.io**](https://chaos.projectdiscovery.io/#/)

यह परियोजना **मुफ्त में सभी उपडोमेन्स जो बग-बाउंटी प्रोग्राम्स से संबंधित हैं** प्रदान करती है। आप इस डेटा तक [chaospy](https://github.com/dr-0x0x/chaospy) का उपयोग करके या इस परियोजना द्वारा प्रयुक्त स्कोप तक पहुँच सकते हैं [https://github.com/projectdiscovery/chaos-public-program-list](https://github.com/projectdiscovery/chaos-public-program-list)

आप इन उपकरणों की **तुलना** यहाँ पा सकते हैं: [https://blog.blacklanternsecurity.com/p/subdomain-enumeration-tool-face-off](https://blog.blacklanternsecurity.com/p/subdomain-enumeration-tool-face-off)

### **DNS Brute force**

आइए DNS सर्वर्स को ब्रूट-फोर्सिंग करके नए **उपडोमेन्स** खोजने की कोशिश करें, संभावित उपडोमेन नामों का उपयोग करते हुए।

इस क्रिया के लिए आपको कुछ **सामान्य उपडोमेन्स वर्डलिस्ट्स की आवश्यकता होगी जैसे**:

* [https://gist.github.com/jhaddix/86a06c5dc309d08580a018c66354a056](https://gist.github.com/jhaddix/86a06c5dc309d08580a018c66354a056)
* [https://wordlists-cdn.assetnote.io/data/manual/best-dns-wordlist.txt](https://wordlists-cdn.assetnote.io/data/manual/best-dns-wordlist.txt)
* [https://localdomain.pw/subdomain-bruteforce-list/all.txt.zip](https://localdomain.pw/subdomain-bruteforce-list/all.txt.zip)
* [https://github.com/pentester-io/commonspeak](https://github.com/pentester-io/commonspeak)
* [https://github.com/danielmiessler/SecLists/tree/master/Discovery/DNS](https://github.com/danielmiessler/SecLists/tree/master/Discovery/DNS)

और अच्छे DNS रिज़ॉल्वर्स के IPs भी। विश्वसनीय DNS रिज़ॉल्वर्स की एक सूची उत्पन्न करने के लिए आप [https://public-dns.info/nameservers-all.txt](https://public-dns.info/nameservers-all.txt) से रिज़ॉल्वर्स डाउनलोड कर सकते हैं और उन्हें फ़िल्टर करने के लिए [**dnsvalidator**](https://github.com/vortexau/dnsvalidator) का उपयोग कर सकते हैं। या आप उपयोग कर सकते हैं: [https://raw.githubusercontent.com/trickest/resolvers/main/resolvers-trusted.txt](https://raw.githubusercontent.com/trickest/resolvers/main/resolvers-trusted.txt)

DNS ब्रूट-फोर्स के लिए सबसे अनुशंसित उपकरण हैं:

* [**massdns**](https://github.com/blechschmidt/massdns): यह पहला उपकरण था जिसने प्रभावी DNS ब्रूट-फोर्स किया। यह बहुत तेज़ है हालांकि यह गलत सकारात्मक परिणामों के प्रति संवेदनशील है।
```bash
sed 's/$/.domain.com/' subdomains.txt > bf-subdomains.txt
./massdns -r resolvers.txt -w /tmp/results.txt bf-subdomains.txt
grep -E "tesla.com. [0-9]+ IN A .+" /tmp/results.txt
```
* [**gobuster**](https://github.com/OJ/gobuster): मुझे लगता है यह केवल 1 resolver का उपयोग करता है
```
gobuster dns -d mysite.com -t 50 -w subdomains.txt
```
* [**shuffledns**](https://github.com/projectdiscovery/shuffledns) `massdns` के आसपास लिखा गया एक wrapper है, जो आपको सक्रिय bruteforce का उपयोग करके मान्य subdomains का अनुक्रमण करने की अनुमति देता है, साथ ही wildcard हैंडलिंग के साथ subdomains को हल करने और आसान इनपुट-आउटपुट समर्थन के साथ।
```
shuffledns -d example.com -list example-subdomains.txt -r resolvers.txt
```
* [**puredns**](https://github.com/d3mondev/puredns): यह भी `massdns` का उपयोग करता है।
```
puredns bruteforce all.txt domain.com
```
* [**aiodnsbrute**](https://github.com/blark/aiodnsbrute) asyncio का उपयोग करके डोमेन नामों को असिंक्रोनस रूप से brute force करता है।
```
aiodnsbrute -r resolvers -w wordlist.txt -vv -t 1024 domain.com
```
### दूसरा DNS ब्रूट-फोर्स राउंड

ओपन सोर्सेज और ब्रूट-फोर्सिंग का उपयोग करके सबडोमेन्स का पता लगाने के बाद, आप पाए गए सबडोमेन्स के विकल्प बनाकर और भी अधिक सबडोमेन्स की खोज करने का प्रयास कर सकते हैं। इस उद्देश्य के लिए कई उपकरण उपयोगी हैं:

* [**dnsgen**](https://github.com/ProjectAnte/dnsgen)**:** दिए गए डोमेन्स और सबडोमेन्स से परिवर्तनों का निर्माण करें।
```bash
cat subdomains.txt | dnsgen -
```
* [**goaltdns**](https://github.com/subfinder/goaltdns): दिए गए डोमेन्स और सबडोमेन्स से परिवर्तन उत्पन्न करें।
* आप goaltdns परिवर्तनों की **wordlist** [**यहाँ**](https://github.com/subfinder/goaltdns/blob/master/words.txt) प्राप्त कर सकते हैं।
```bash
goaltdns -l subdomains.txt -w /tmp/words-permutations.txt -o /tmp/final-words-s3.txt
```
* [**gotator**](https://github.com/Josue87/gotator)**:** डोमेन और सबडोमेन दिए जाने पर परिवर्तन (permutations) उत्पन्न करता है। यदि परिवर्तन फाइल निर्दिष्ट नहीं की गई है, तो gotator अपनी स्वयं की फाइल का उपयोग करेगा।
```
gotator -sub subdomains.txt -silent [-perm /tmp/words-permutations.txt]
```
* [**altdns**](https://github.com/infosec-au/altdns): यह सबडोमेन्स के परम्यूटेशन्स जनरेट करने के अलावा, उन्हें रिजॉल्व भी करने की कोशिश कर सकता है (लेकिन पिछले कमेंटेड टूल्स का उपयोग करना बेहतर है)।
* आप altdns परम्यूटेशन्स **वर्डलिस्ट** [**यहाँ**](https://github.com/infosec-au/altdns/blob/master/words.txt) प्राप्त कर सकते हैं।
```
altdns -i subdomains.txt -w /tmp/words-permutations.txt -o /tmp/asd3
```
* [**dmut**](https://github.com/bp0lr/dmut): उपडोमेन्स के परिवर्तन, उत्परिवर्तन और बदलाव करने के लिए एक और उपकरण। यह उपकरण परिणाम को ब्रूट फोर्स करेगा (इसमें dns वाइल्ड कार्ड का समर्थन नहीं है)।
* आप dmut परिवर्तन शब्दसूची [**यहाँ**](https://raw.githubusercontent.com/bp0lr/dmut/main/words.txt) प्राप्त कर सकते हैं।
```bash
cat subdomains.txt | dmut -d /tmp/words-permutations.txt -w 100 \
--dns-errorLimit 10 --use-pb --verbose -s /tmp/resolvers-trusted.txt
```
* [**alterx**](https://github.com/projectdiscovery/alterx)**:** यह एक डोमेन के आधार पर **नए संभावित सबडोमेन नाम उत्पन्न करता है** निर्दिष्ट पैटर्न के आधार पर और अधिक सबडोमेन की खोज करने के लिए।

#### स्मार्ट परम्यूटेशन जनरेशन

* [**regulator**](https://github.com/cramppet/regulator): अधिक जानकारी के लिए इस [**पोस्ट**](https://cramppet.github.io/regulator/index.html) को पढ़ें लेकिन यह मूल रूप से **प्राप्त सबडोमेन्स के मुख्य भागों** को प्राप्त करेगा और उन्हें मिलाकर और अधिक सबडोमेन खोजने का प्रयास करेगा।
```bash
python3 main.py adobe.com adobe adobe.rules
make_brute_list.sh adobe.rules adobe.brute
puredns resolve adobe.brute --write adobe.valid
```
* [**subzuf**](https://github.com/elceef/subzuf)**:** _subzuf_ एक सबडोमेन ब्रूट-फोर्स फज़र है जो एक अत्यंत सरल परंतु प्रभावी DNS प्रतिक्रिया-निर्देशित एल्गोरिदम के साथ जोड़ा गया है। यह इनपुट डेटा के प्रदान किए गए सेट का उपयोग करता है, जैसे कि एक विशेष रूप से तैयार की गई वर्डलिस्ट या ऐतिहासिक DNS/TLS रिकॉर्ड्स, ताकि सटीक रूप से अधिक संबंधित डोमेन नामों का संश्लेषण किया जा सके और DNS स्कैन के दौरान एकत्रित जानकारी के आधार पर उन्हें और भी आगे विस्तारित किया जा सके।
```
echo www | subzuf facebook.com
```
### **सबडोमेन डिस्कवरी वर्कफ्लो**

इस ब्लॉग पोस्ट को देखें जो मैंने लिखा है कि कैसे **सबडोमेन डिस्कवरी को ऑटोमेट** किया जा सकता है एक डोमेन से **Trickest वर्कफ्लो** का उपयोग करके ताकि मुझे मैन्युअली अपने कंप्यूटर में ढेर सारे टूल्स लॉन्च करने की जरूरत न पड़े:

{% embed url="https://trickest.com/blog/full-subdomain-discovery-using-workflow/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

{% embed url="https://trickest.com/blog/full-subdomain-brute-force-discovery-using-workflow/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

### **VHosts / वर्चुअल होस्ट्स**

यदि आपको एक IP पते में **एक या कई वेब पेज** मिले हैं जो सबडोमेन्स से संबंधित हैं, तो आप **उस IP में अन्य सबडोमेन्स के वेब्स को खोजने का प्रयास** कर सकते हैं **OSINT स्रोतों** में एक IP में डोमेन्स की खोज करके या **उस IP में VHost डोमेन नामों को ब्रूट-फोर्सिंग** करके।

#### OSINT

आप कुछ **VHosts को IPs में उपयोग करके खोज सकते हैं** [**HostHunter**](https://github.com/SpiderLabs/HostHunter) **या अन्य APIs का उपयोग करके**।

**ब्रूट फोर्स**

यदि आपको संदेह है कि कुछ सबडोमेन एक वेब सर्वर में छिपा हो सकता है तो आप इसे ब्रूट फोर्स करने का प्रयास कर सकते हैं:
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
इस तकनीक का उपयोग करके आप आंतरिक/छिपे हुए endpoints तक भी पहुँच सकते हैं।
{% endhint %}

### **CORS Brute Force**

कभी-कभी आपको ऐसे पृष्ठ मिलेंगे जो केवल तब _**Access-Control-Allow-Origin**_ हेडर लौटाते हैं जब एक मान्य डोमेन/सबडोमेन _**Origin**_ हेडर में सेट किया जाता है। इन परिदृश्यों में, आप इस व्यवहार का दुरुपयोग करके नए **सबडोमेन** की **खोज** कर सकते हैं।
```bash
ffuf -w subdomains-top1million-5000.txt -u http://10.10.10.208 -H 'Origin: http://FUZZ.crossfit.htb' -mr "Access-Control-Allow-Origin" -ignore-body
```
### **Buckets Brute Force**

**सबडोमेन्स** की खोज करते समय ध्यान दें कि क्या वह किसी प्रकार के **bucket** की ओर **इशारा** कर रहा है, और उस स्थिति में [**अनुमतियों की जांच करें**](../../network-services-pentesting/pentesting-web/buckets/)**।**\
इसके अलावा, इस बिंदु पर आपको सभी डोमेन्स के बारे में पता चल जाएगा जो स्कोप के अंदर हैं, संभावित bucket नामों को [**brute force करें और अनुमतियों की जांच करें**](../../network-services-pentesting/pentesting-web/buckets/).

### **Monitorization**

आप **मॉनिटर** कर सकते हैं कि क्या किसी डोमेन के **नए सबडोमेन्स** बनाए गए हैं, **Certificate Transparency** Logs की मॉनिटरिंग करके [**sublert**](https://github.com/yassineaboukir/sublert/blob/master/sublert.py) करता है।

### **भेद्यताओं की खोज**

संभावित [**सबडोमेन टेकओवर्स**](../../pentesting-web/domain-subdomain-takeover.md#subdomain-takeover) की जांच करें।\
यदि **सबडोमेन** किसी **S3 bucket** की ओर इशारा कर रहा है, [**अनुमतियों की जांच करें**](../../network-services-pentesting/pentesting-web/buckets/).

यदि आपको कोई **सबडोमेन अलग IP के साथ मिलता है** जो आपको पहले से पता नहीं है, तो आपको **बेसिक भेद्यता स्कैन** (Nessus या OpenVAS का उपयोग करके) और कुछ [**पोर्ट स्कैन**](../pentesting-network/#discovering-hosts-from-the-outside) **nmap/masscan/shodan** के साथ करना चाहिए। जो सेवाएं चल रही हैं उनके आधार पर आप **इस पुस्तक में कुछ तरकीबें पा सकते हैं उन्हें "अटैक" करने के लिए**।\
_ध्यान दें कि कभी-कभी सबडोमेन किसी ऐसे IP के अंदर होस्ट किया जाता है जो क्लाइंट द्वारा नियंत्रित नहीं होता है, इसलिए वह स्कोप में नहीं होता है, सावधान रहें।_

## IPs

प्रारंभिक चरणों में आपने **कुछ IP रेंज, डोमेन्स और सबडोमेन्स पाए होंगे**।\
अब समय है उन रेंजों से सभी IPs को **एकत्रित करने का** और **डोमेन्स/सबडोमेन्स के लिए (DNS queries).**

निम्नलिखित **मुफ्त apis** की सेवाओं का उपयोग करके आप **पिछले IPs जो डोमेन्स और सबडोमेन्स द्वारा उपयोग किए गए थे** भी पा सकते हैं। ये IPs अभी भी क्लाइंट के स्वामित्व में हो सकते हैं (और आपको [**CloudFlare बायपासेस**](../../network-services-pentesting/pentesting-web/uncovering-cloudflare.md) खोजने में मदद कर सकते हैं)

* [**https://securitytrails.com/**](https://securitytrails.com/)

आप उस टूल [**hakip2host**](https://github.com/hakluke/hakip2host) का भी उपयोग कर सकते हैं जो विशेष IP पते की ओर इशारा करने वाले डोमेन्स की जांच करता है।

### **भेद्यताओं की खोज**

**CDNs के अलावा सभी IPs का पोर्ट स्कैन करें** (क्योंकि वहां आपको कुछ भी दिलचस्प नहीं मिलेगा)। चल रही सेवाओं में आप **भेद्यताएं पा सकते हैं**।

**होस्ट्स को स्कैन करने के बारे में एक** [**गाइड**](../pentesting-network/) **खोजें।**

## Web servers hunting

> हमने सभी कंपनियों और उनकी संपत्तियों को खोज लिया है और हमें IP रेंज, डोमेन्स और सबडोमेन्स के बारे में पता है जो स्कोप के अंदर हैं। अब समय है वेब सर्वर्स की खोज करने का।

पिछले चरणों में आपने शायद पहले ही कुछ **IPs और डोमेन्स की रेकॉन की होगी**, इसलिए आपने शायद पहले ही सभी संभावित वेब सर्वर्स को खोज लिया होगा। हालांकि, अगर आपने नहीं किया है तो हम अब कुछ **तेज तरकीबें देखेंगे वेब सर्वर्स की खोज के लिए** स्कोप के अंदर।

कृपया ध्यान दें कि यह **वेब ऐप्स की खोज के लिए उन्मुख होगा**, इसलिए आपको **भेद्यता** और **पोर्ट स्कैनिंग** भी करनी चाहिए (**अगर स्कोप द्वारा अनुमति दी गई हो**).

**वेब** सर्वर्स से संबंधित **खुले पोर्ट्स की खोज करने का एक तेज तरीका** [**masscan का उपयोग करके यहां पाया जा सकता है**](../pentesting-network/#http-port-discovery).\
वेब सर्वर्स की खोज के लिए एक अन्य उपयोगी टूल है [**httprobe**](https://github.com/tomnomnom/httprobe)**,** [**fprobe**](https://github.com/theblackturtle/fprobe) और [**httpx**](https://github.com/projectdiscovery/httpx). आप बस डोमेन्स की एक सूची पास करते हैं और यह पोर्ट 80 (http) और 443 (https) से कनेक्ट करने की कोशिश करेगा। इसके अलावा, आप अन्य पोर्ट्स की कोशिश करने के लिए इंगित कर सकते हैं:
```bash
cat /tmp/domains.txt | httprobe #Test all domains inside the file for port 80 and 443
cat /tmp/domains.txt | httprobe -p http:8080 -p https:8443 #Check port 80, 443 and 8080 and 8443
```
### **स्क्रीनशॉट्स**

जब आपने कंपनी के **IPs**, सभी **डोमेन्स** और **सबडोमेन्स** में मौजूद सभी **वेब सर्वर्स** की खोज कर ली है, तो शायद आप **नहीं जानते कि शुरुआत कहां से करें**। तो, चलिए इसे सरल बनाते हैं और सभी के स्क्रीनशॉट्स लेना शुरू करते हैं। केवल **मुख्य पृष्ठ** पर **नजर डालने** से आप **अजीब** एंडपॉइंट्स पा सकते हैं जो **असुरक्षित** होने के लिए अधिक **प्रवण** होते हैं।

प्रस्तावित विचार को करने के लिए आप [**EyeWitness**](https://github.com/FortyNorthSecurity/EyeWitness), [**HttpScreenshot**](https://github.com/breenmachine/httpscreenshot), [**Aquatone**](https://github.com/michenriksen/aquatone), [**Shutter**](https://shutter-project.org/downloads/third-party-packages/) या [**webscreenshot**](https://github.com/maaaaz/webscreenshot) का उपयोग कर सकते हैं।

इसके अलावा, आप [**eyeballer**](https://github.com/BishopFox/eyeballer) का उपयोग कर सकते हैं जो सभी **स्क्रीनशॉट्स** पर चलकर आपको बताएगा **किनमें असुरक्षितताएं होने की संभावना है**, और किनमें नहीं।

## सार्वजनिक क्लाउड एसेट्स

किसी कंपनी के संभावित क्लाउड एसेट्स को खोजने के लिए आपको **कंपनी की पहचान करने वाले कीवर्ड्स की एक सूची से शुरुआत करनी चाहिए**। उदाहरण के लिए, एक क्रिप्टो कंपनी के लिए आप शब्दों का उपयोग कर सकते हैं जैसे: `"क्रिप्टो", "वॉलेट", "डाओ", "<डोमेन_नाम>", <"सबडोमेन_नाम">`।

आपको **बकेट्स में प्रयुक्त सामान्य शब्दों** की वर्डलिस्ट्स की भी आवश्यकता होगी:

* [https://raw.githubusercontent.com/cujanovic/goaltdns/master/words.txt](https://raw.githubusercontent.com/cujanovic/goaltdns/master/words.txt)
* [https://raw.githubusercontent.com/infosec-au/altdns/master/words.txt](https://raw.githubusercontent.com/infosec-au/altdns/master/words.txt)
* [https://raw.githubusercontent.com/jordanpotti/AWSBucketDump/master/BucketNames.txt](https://raw.githubusercontent.com/jordanpotti/AWSBucketDump/master/BucketNames.txt)

फिर, उन शब्दों के साथ आपको **परिवर्तन** उत्पन्न करने चाहिए (अधिक जानकारी के लिए [**दूसरा दौर DNS ब्रूट-फोर्स**](./#second-dns-bruteforce-round) देखें)।

परिणामी वर्डलिस्ट्स के साथ आप [**cloud\_enum**](https://github.com/initstring/cloud\_enum), [**CloudScraper**](https://github.com/jordanpotti/CloudScraper), [**cloudlist**](https://github.com/projectdiscovery/cloudlist) **या** [**S3Scanner**](https://github.com/sa7mon/S3Scanner) जैसे उपकरणों का उपयोग कर सकते हैं।

याद रखें कि क्लाउड एसेट्स की खोज करते समय आपको AWS में केवल बकेट्स से अधिक की तलाश करनी चाहिए।

### **असुरक्षितताओं की खोज**

यदि आपको **खुले बकेट्स या क्लाउड फंक्शन्स एक्सपोज्ड** मिलते हैं तो आपको उन्हें **एक्सेस करना चाहिए** और देखना चाहिए कि वे आपको क्या प्रदान करते हैं और आप उनका दुरुपयोग कर सकते हैं या नहीं।

## ईमेल

**डोमेन्स** और **सबडोमेन्स** के साथ आपके पास वह सब कुछ है जो आपको ईमेल खोजने के लिए **शुरुआत करने की आवश्यकता है**। ये वे **APIs** और **उपकरण** हैं जो मेरे लिए किसी कंपनी के ईमेल खोजने में सबसे अच्छे साबित हुए हैं:

* [**theHarvester**](https://github.com/laramies/theHarvester) - APIs के साथ
* [**https://hunter.io/**](https://hunter.io/) का API (निःशुल्क संस्करण)
* [**https://app.snov.io/**](https://app.snov.io/) का API (निःशुल्क संस्करण)
* [**https://minelead.io/**](https://minelead.io/) का API (निःशुल्क संस्करण)

### **असुरक्षितताओं की खोज**

ईमेल बाद में **वेब लॉगिन्स और ऑथ सेवाओं के ब्रूट-फोर्स** के लिए काम आएंगे (जैसे SSH)। साथ ही, वे **फिशिंग्स** के लिए आवश्यक हैं। इसके अलावा, ये APIs आपको ईमेल के पीछे के व्यक्ति के बारे में और भी **जानकारी देंगे**, जो फिशिंग अभियान के लिए उपयोगी है।

## क्रेडेंशियल लीक्स

**डोमेन्स,** **सबडोमेन्स**, और **ईमेल्स** के साथ आप उन ईमेल्स से संबंधित अतीत में लीक हुए क्रेडेंशियल्स की खोज शुरू कर सकते हैं:

* [https://leak-lookup.com](https://leak-lookup.com/account/login)
* [https://www.dehashed.com/](https://www.dehashed.com/)

### **असुरक्षितताओं की खोज**

यदि आपको **मान्य लीक** क्रेडेंशियल्स मिलते हैं, तो यह एक बहुत ही आसान जीत है।

## सीक्रेट्स लीक्स

क्रेडेंशियल लीक्स उन हैक्स से संबंधित हैं जहां **संवेदनशील जानकारी लीक और बेची गई थी**। हालांकि, कंपनियां **अन्य लीक्स** से भी प्रभावित हो सकती हैं जिनकी जानकारी उन डेटाबेस में नहीं होती:

### Github लीक्स

क्रेडेंशियल्स और APIs **कंपनी** के **सार्वजनिक रिपॉजिटरीज** में या उस कंपनी के द्वारा काम करने वाले **यूजर्स** के रिपॉजिटरीज में लीक हो सकते हैं।\
आप **उपकरण** [**Leakos**](https://github.com/carlospolop/Leakos) का उपयोग करके एक **संगठन** के सभी **सार्वजनिक रिपॉजिटरीज** और उसके **डेवलपर्स** को **डाउनलोड** कर सकते हैं और उन पर [**gitleaks**](https://github.com/zricethezav/gitleaks) को स्वचालित रूप से चला सकते हैं।

**Leakos** का उपयोग कभी-कभी **वेब पेजों में भी सीक्रेट्स होते हैं** इसलिए उसे पास किए गए सभी **टेक्स्ट** प्रदान किए गए **URLs** के खिलाफ **gitleaks** चलाने के लिए भी किया जा सकता है।

#### Github डॉर्क्स

आप जिस संगठन पर हमला कर रहे हैं उसमें संभावित **github डॉर्क्स** के लिए भी इस **पेज** की जांच करें:

{% content-ref url="github-leaked-secrets.md" %}
[github-leaked-secrets.md](github-leaked-secrets.md)
{% endcontent-ref %}

### Pastes लीक्स

कभी-कभी हमलावर या सिर्फ कर्मचारी **कंपनी की सामग्री को पेस्ट साइट पर प्रकाशित कर देंगे**। इसमें **संवेदनशील जानकारी** हो सकती है या नहीं, लेकिन इसकी खोज करना बहुत दिलचस्प है।\
आप एक ही समय में 80 से अधिक पेस्ट साइट्स में खोज करने के लिए उपकरण [**Pastos**](https://github.com/carlospolop/Pastos) का उपयोग कर सकते हैं।

### Google डॉर्क्स

पुराने लेकिन सोने के google डॉर्क्स हमेशा उपयोगी होते हैं जो **उजागर जानकारी जो वहां नहीं होनी चाहिए** को खोजने के लिए। एकमात्र समस्या यह है कि [**google-hacking-database**](https://www.exploit-db.com/google-hacking-database) में कई **हजारों** संभावित क्वेरीज होती हैं जिन्हें आप मैन्युअल रूप से नहीं चला सकते। तो, आप अपने पसंदीदा 10 लोगों को प्राप्त कर सकते हैं या आप एक **उपकरण जैसे** [**Gorks**](https://github.com/carlospolop/Gorks) **का उपयोग करके उन सभी को चला सकते हैं**।

_ध्यान दें कि नियमित Google ब्राउज़र का उपयोग करके पूरे डेटाबेस क
