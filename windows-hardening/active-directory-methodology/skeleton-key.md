# स्केलेटन की

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें और PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

## स्केलेटन की हमला

**स्केलेटन की हमला** एक विवेकी तकनीक है जो हमलावादियों को **एक मास्टर पासवर्ड डोमेन कंट्रोलर में इंजेक्ट करके** **एक्टिव डायरेक्टरी प्रमाणीकरण** को **छलकरने** की अनुमति देती है। इससे हमलावाद किसी भी उपयोगकर्ता के रूप में प्रमाणीकृत हो सकता है बिना उनके पासवर्ड के, जिससे उन्हें डोमेन के लिए असीमित पहुंच मिलती है।

इसे [Mimikatz](https://github.com/gentilkiwi/mimikatz) का उपयोग करके किया जा सकता है। इस हमले को करने के लिए **डोमेन व्यवस्थापक अधिकार आवश्यक है**, और हमलावाद को समग्र उल्लंघन सुनिश्चित करने के लिए प्रत्येक डोमेन कंट्रोलर को लक्षित करना होगा। हालांकि, हमले का प्रभाव अस्थायी है, क्योंकि **डोमेन कंट्रोलर को पुनः आरंभ करने से मैलवेयर मिटा देता है**, जिससे निरंतर पहुंच के लिए पुनर्योजना की आवश्यकता होती है।

**हमला को क्रियान्वित करने** के लिए एक एकल कमांड की आवश्यकता है: `misc::skeleton`.

## सुरक्षा उपाय

इस प्रकार के हमलों के खिलाफ सुरक्षा उपाय में विशेष घटकों की निगरानी करना शामिल है जो सेवाओं की स्थापना या संवेदनशील विशेषाधिकारों का उपयोग करने की सूचित करने वाले घटना आईडी के लिए है। विशेष रूप से, सिस्टम घटना आईडी 7045 या सुरक्षा घटना आईडी 4673 की खोज संदेहपूर्ण गतिविधियों को उजागर कर सकती है। इसके अतिरिक्त, `lsass.exe` को संरक्षित प्रक्रिया के रूप में चलाना हमलावादों के प्रयासों को काफी बाधित कर सकता है, क्योंकि इसके लिए उन्हें एक कर्नेल मोड ड्राइवर का उपयोग करना होगा, हमले की जटिलता बढ़ाता है।

यहाँ सुरक्षा उपायों को मजबूत करने के लिए PowerShell कमांड हैं:

- संदिग्ध सेवाओं की स्थापना का पता लगाने के लिए, इस्तेमाल करें: `Get-WinEvent -FilterHashtable @{Logname='System';ID=7045} | ?{$_.message -like "*Kernel Mode Driver*"}`

- विशेष रूप से, Mimikatz के ड्राइवर का पता लगाने के लिए, निम्नलिखित कमांड का उपयोग किया जा सकता है: `Get-WinEvent -FilterHashtable @{Logname='System';ID=7045} | ?{$_.message -like "*Kernel Mode Driver*" -and $_.message -like "*mimidrv*"}`

- `lsass.exe` को मजबूत करने के लिए, इसे संरक्षित प्रक्रिया के रूप में सक्षम करना सुझाया जाता है: `New-ItemProperty HKLM:\SYSTEM\CurrentControlSet\Control\Lsa -Name RunAsPPL -Value 1 -Verbose`

प्रोटेक्टिव उपायों को सफलतापूर्वक लागू किया गया है यह सुनिश्चित करने के लिए एक सिस्टम पुनरारंभ के बाद सत्यापन महत्वपूर्ण है। इसे इसके माध्यम से प्राप्त किया जा सकता है: `Get-WinEvent -FilterHashtable @{Logname='System';ID=12} | ?{$_.message -like "*protected process*`

## संदर्भ
* [https://blog.netwrix.com/2022/11/29/skeleton-key-attack-active-directory/](https://blog.netwrix.com/2022/11/29/skeleton-key-attack-active-directory/)

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें और PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>
