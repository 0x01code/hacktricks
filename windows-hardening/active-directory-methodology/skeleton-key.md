# स्केलेटन की

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert) के साथ शून्य से हीरो तक AWS हैकिंग सीखें</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स शेयर करें।

</details>

## **स्केलेटन की**

**से:** [**https://blog.stealthbits.com/unlocking-all-the-doors-to-active-directory-with-the-skeleton-key-attack/**](https://blog.stealthbits.com/unlocking-all-the-doors-to-active-directory-with-the-skeleton-key-attack/)

Active Directory खातों को समझौता करने के लिए कई तरीके हैं जिनका उपयोग हमलावर विशेषाधिकार बढ़ाने और अपने डोमेन में स्थापित होने के बाद पर्सिस्टेंस बनाने के लिए कर सकते हैं। स्केलेटन की एक विशेष रूप से डरावना मैलवेयर है जो Active Directory डोमेन्स को लक्षित करता है ताकि किसी भी खाते को हाईजैक करना चिंताजनक रूप से आसान हो जाए। यह मैलवेयर **LSASS में खुद को इंजेक्ट करता है और एक मास्टर पासवर्ड बनाता है जो डोमेन में किसी भी खाते के लिए काम करेगा**। मौजूदा पासवर्ड भी काम करते रहेंगे, इसलिए यह जानना बहुत मुश्किल है कि यह हमला हो चुका है जब तक आपको पता नहीं होता कि क्या देखना है।

आश्चर्यजनक नहीं है, यह कई हमलों में से एक है जो [Mimikatz](https://github.com/gentilkiwi/mimikatz) का उपयोग करके पैकेज किया गया है और करना बहुत आसान है। आइए देखते हैं कि यह कैसे काम करता है।

### स्केलेटन की हमले के लिए आवश्यकताएँ

इस हमले को अंजाम देने के लिए, **हमलावर के पास डोमेन एडमिन अधिकार होने चाहिए**। इस हमले को **प्रत्येक और हर डोमेन कंट्रोलर पर पूर्ण समझौता के लिए किया जाना चाहिए, लेकिन एक अकेले डोमेन कंट्रोलर को लक्षित करना भी प्रभावी हो सकता है**। **रिबूटिंग** एक डोमेन कंट्रोलर **इस मैलवेयर को हटा देगा** और इसे हमलावर द्वारा फिर से तैनात किया जाना होगा।

### स्केलेटन की हमला करना

हमला करना बहुत सरल है। इसके लिए केवल निम्नलिखित **कमांड को प्रत्येक डोमेन कंट्रोलर पर चलाना आवश्यक है**: `misc::skeleton`. उसके बाद, आप Mimikatz के डिफ़ॉल्ट पासवर्ड के साथ किसी भी उपयोगकर्ता के रूप में प्रमाणित कर सकते हैं।

![Injecting a skeleton key using the misc::skeleton into a domain controller with Mimikatz](https://blog.stealthbits.com/wp-content/uploads/2017/07/1-3.png)

यहाँ एक डोमेन एडमिन सदस्य के लिए प्रमाणीकरण है जो स्केलेटन की को पासवर्ड के रूप में उपयोग करके एक डोमेन कंट्रोलर के लिए प्रशासनिक पहुँच प्राप्त करता है:

![Using the skeleton key as a password with the misc::skeleton command to get administrative access to a domain controller with the default password of Mimikatz](https://blog.stealthbits.com/wp-content/uploads/2017/07/2-5.png)

नोट: यदि आपको संदेश मिलता है कि, “सिस्टम त्रुटि 86 हो गई है। निर्दिष्ट नेटवर्क पासवर्ड सही नहीं है”, तो बस उपयोगकर्ता नाम के लिए डोमेन\खाता प्रारूप का उपयोग करें और यह काम करना चाहिए।

![Using the domain\account format for the username if you get a message saying System error 86 has occurred The specified network password is not correct](https://blog.stealthbits.com/wp-content/uploads/2017/07/3-3.png)

यदि lsass पहले से ही स्केलेटन के साथ **पैच किया गया था**, तो यह **त्रुटि** दिखाई देगी:

![](<../../.gitbook/assets/image (160).png>)

### उपाय

* घटनाएँ:
* सिस्टम इवेंट ID 7045 - सिस्टम में एक सेवा स्थापित की गई थी। (प्रकार कर्नेल मोड ड्राइवर)
* सुरक्षा इवेंट ID 4673 – संवेदनशील विशेषाधिकार का उपयोग ("ऑडिट प्रिविलेज उपयोग" सक्षम होना चाहिए)
* इवेंट ID 4611 – एक विश्वसनीय लॉगऑन प्रक्रिया को स्थानीय सुरक्षा प्राधिकरण के साथ पंजीकृत किया गया है ("ऑडिट प्रिविलेज उपयोग" सक्षम होना चाहिए)
* `Get-WinEvent -FilterHashtable @{Logname='System';ID=7045} | ?{$_.message -like "`_`Kernel Mode Driver"}`_
* यह केवल mimidrv का पता लगाता है `Get-WinEvent -FilterHashtable @{Logname='System';ID=7045} | ?{$`_`.message -like "Kernel Mode Driver" -and $`_`.message -like "`_`mimidrv`_`"}`
* उपाय:
* lsass.exe को एक सुरक्षित प्रक्रिया के रूप में चलाएं, यह हमलावर को कर्नेल मोड ड्राइवर लोड करने के लिए मजबूर करता है
* `New-ItemProperty HKLM:\SYSTEM\CurrentControlSet\Control\Lsa -Name RunAsPPL -Value 1 -Verbose`
* रिबूट के बाद सत्यापित करें: `Get-WinEvent -FilterHashtable @{Logname='System';ID=12} | ?{$_.message -like "`_`protected process"}`_

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert) के साथ शून्य से हीरो तक AWS हैकिंग सीखें</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स शेयर करें।

</details>
