# भौतिक हमले

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** 🐦 पर **मुझे फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स शेयर करें।

</details>

## BIOS पासवर्ड

### बैटरी

अधिकांश **मदरबोर्ड्स** में एक **बैटरी** होती है। यदि आप इसे **30 मिनट के लिए हटा दें** तो BIOS की सेटिंग्स **रीस्टार्ट** हो जाएंगी (पासवर्ड सहित)।

### जम्पर CMOS

अधिकांश **मदरबोर्ड्स** में एक **जम्पर** होता है जो सेटिंग्स को रीस्टार्ट कर सकता है। यह जम्पर एक केंद्रीय पिन को दूसरे के साथ जोड़ता है, यदि आप **इन पिनों को जोड़ दें तो मदरबोर्ड रीसेट हो जाएगा**।

### लाइव टूल्स

यदि आप उदाहरण के लिए एक **Kali** Linux को Live CD/USB से **चला सकते हैं** तो आप _**killCmos**_ या _**CmosPWD**_ (यह आखिरी वाला Kali में शामिल है) जैसे टूल्स का उपयोग करके **BIOS के पासवर्ड को रिकवर करने की कोशिश कर सकते हैं**।

### ऑनलाइन BIOS पासवर्ड रिकवरी

BIOS का पासवर्ड **3 बार गलत डालें**, फिर BIOS एक **एरर मैसेज दिखाएगा** और यह ब्लॉक हो जाएगा।\
[https://bios-pw.org](https://bios-pw.org) पेज पर जाएं और BIOS द्वारा दिखाए गए **एरर कोड को डालें** और आप भाग्यशाली हो सकते हैं और एक **वैध पासवर्ड प्राप्त कर सकते हैं** (एक ही खोज से आपको विभिन्न पासवर्ड दिखाई दे सकते हैं और एक से अधिक वैध हो सकते हैं)।

## UEFI

UEFI की सेटिंग्स की जांच करने और किसी प्रकार का हमला करने के लिए आपको [chipsec](https://github.com/chipsec/chipsec/blob/master/chipsec-manual.pdf) का प्रयास करना चाहिए।\
इस टूल का उपयोग करके आप आसानी से Secure Boot को डिसेबल कर सकते हैं:
```
python chipsec_main.py -module exploits.secure.boot.pk
```
## RAM

### कोल्ड बूट

**RAM मेमोरी 1 से 2 मिनट तक पर्सिस्टेंट रहती है** जब कंप्यूटर बंद होता है। यदि आप **कोल्ड** (उदाहरण के लिए लिक्विड नाइट्रोजन) मेमोरी कार्ड पर लगाते हैं तो आप इस समय को **10 मिनट तक** बढ़ा सकते हैं।

फिर, आप **मेमोरी डंप** कर सकते हैं (dd.exe, mdd.exe, Memoryze, win32dd.exe या DumpIt जैसे टूल्स का उपयोग करके) मेमोरी का विश्लेषण करने के लिए।

आपको मेमोरी का **विश्लेषण volatility का उपयोग करके** करना चाहिए।

### [INCEPTION](https://github.com/carmaa/inception)

Inception एक **फिजिकल मेमोरी मैनिपुलेशन** और हैकिंग टूल है जो PCI-आधारित DMA का शोषण करता है। यह टूल **FireWire**, **Thunderbolt**, **ExpressCard**, PC Card और किसी भी अन्य PCI/PCIe HW इंटरफेसेज पर हमला कर सकता है।\
अपने कंप्यूटर को उन **इंटरफेसेज** में से किसी एक के माध्यम से पीड़ित कंप्यूटर से **कनेक्ट** करें और **INCEPTION** **फिजिकल मेमोरी** में **पैच** लगाने की कोशिश करेगा ताकि आपको **एक्सेस** मिल सके।

**यदि INCEPTION सफल होता है, तो कोई भी पासवर्ड मान्य होगा।**

**यह Windows10 के साथ काम नहीं करता।**

## लाइव CD/USB

### स्टिकी कीज और अधिक

* **SETHC:** _sethc.exe_ को तब इन्वोक किया जाता है जब SHIFT को 5 बार दबाया जाता है
* **UTILMAN:** _Utilman.exe_ को WINDOWS+U दबाने से इन्वोक किया जाता है
* **OSK:** _osk.exe_ को WINDOWS+U दबाने और फिर ऑन-स्क्रीन कीबोर्ड लॉन्च करने से इन्वोक किया जाता है
* **DISP:** _DisplaySwitch.exe_ को WINDOWS+P दबाने से इन्वोक किया जाता है

ये बाइनरीज _**C:\Windows\System32**_ के अंदर स्थित हैं। आप उनमें से किसी को भी बाइनरी **cmd.exe** की एक **कॉपी** से **बदल** सकते हैं (जो उसी फोल्डर में भी है) और जब भी आप उन बाइनरीज में से किसी को इन्वोक करेंगे एक कमांड प्रॉम्प्ट **SYSTEM** के रूप में प्रकट होगा।

### SAM को मॉडिफाई करना

आप टूल _**chntpw**_ का उपयोग करके माउंटेड Windows फाइलसिस्टम की _**SAM**_ **फाइल को मॉडिफाई** कर सकते हैं। फिर, आप उदाहरण के लिए, Administrator यूजर का पासवर्ड बदल सकते हैं।\
यह टूल KALI में उपलब्ध है।
```
chntpw -h
chntpw -l <path_to_SAM>
```
**लिनक्स सिस्टम के अंदर आप** _**/etc/shadow**_ **या** _**/etc/passwd**_ **फाइल को मोडिफाई कर सकते हैं।**

### **कोन-बूट**

**कोन-बूट** बेहतरीन टूल्स में से एक है जो आपको बिना पासवर्ड जाने विंडोज में लॉग इन करने देता है। यह **सिस्टम BIOS में हुकिंग करके और विंडोज कर्नेल की सामग्री को अस्थायी रूप से बदलकर** काम करता है जबकि बूटिंग (नए संस्करण **UEFI** के साथ भी काम करते हैं)। इसके बाद यह आपको लॉगिन के दौरान **कुछ भी पासवर्ड के रूप में दर्ज करने की अनुमति देता है**। जब आप अगली बार कोन-बूट के बिना कंप्यूटर स्टार्ट करते हैं, मूल पासवर्ड वापस आ जाएगा, अस्थायी परिवर्तन त्याग दिए जाएंगे और सिस्टम ऐसा व्यवहार करेगा जैसे कुछ हुआ ही नहीं।\
और पढ़ें: [https://www.raymond.cc/blog/login-to-windows-administrator-and-linux-root-account-without-knowing-or-changing-current-password/](https://www.raymond.cc/blog/login-to-windows-administrator-and-linux-root-account-without-knowing-or-changing-current-password/)

यह एक लाइव CD/USB है जो **मेमोरी को पैच** कर सकता है ताकि आपको लॉगिन करने के लिए पासवर्ड जानने की **जरूरत नहीं पड़े**।\
कोन-बूट **स्टिकीकीज** ट्रिक भी करता है ताकि आप _**Shift**_ **को 5 बार दबाकर एक एडमिनिस्ट्रेटर cmd प्राप्त कर सकें**।

## **विंडोज चलाना**

### प्रारंभिक शॉर्टकट्स

### बूटिंग शॉर्टकट्स

* supr - BIOS
* f8 - रिकवरी मोड
* _supr_ - BIOS ini
* _f8_ - रिकवरी मोड
* _Shift_ (विंडोज बैनर के बाद) - ऑटोलॉगन के बजाय लॉगिन पेज पर जाएं (ऑटोलॉगन से बचें)

### **बैड USBs**

#### **रबर डकी ट्यूटोरियल्स**

* [ट्यूटोरियल 1](https://github.com/hak5darren/USB-Rubber-Ducky/wiki/Tutorials)
* [ट्यूटोरियल 2](https://blog.hartleybrody.com/rubber-ducky-guide/)

#### **Teensyduino**

* [पेलोड्स और ट्यूटोरियल्स](https://github.com/Screetsec/Pateensy)

अपना खुद का बैड USB बनाने के बारे में भी ढेर सारे ट्यूटोरियल्स हैं।

### वॉल्यूम शैडो कॉपी

एडमिनिस्ट्रेटर प्रिविलेजेस और पावरशेल के साथ आप SAM फाइल की एक कॉपी बना सकते हैं।[ इस कोड को देखें](../windows-hardening/basic-powershell-for-pentesters/#volume-shadow-copy)।

## बिटलॉकर को बायपास करना

बिटलॉकर **2 पासवर्ड** का उपयोग करता है। एक **उपयोगकर्ता** द्वारा प्रयुक्त, और **रिकवरी** पासवर्ड (48 अंक)।

यदि आप भाग्यशाली हैं और वर्तमान विंडोज सत्र में _**C:\Windows\MEMORY.DMP**_ फाइल मौजूद है (यह एक मेमोरी डंप है) तो आप **उसमें रिकवरी पासवर्ड की खोज करने का प्रयास कर सकते हैं**। आप इस फाइल को **प्राप्त कर सकते हैं** और फाइलसिस्टम की एक **कॉपी** और फिर _Elcomsoft Forensic Disk Decryptor_ का उपयोग करके सामग्री प्राप्त कर सकते हैं (यह केवल काम करेगा यदि पासवर्ड मेमोरी डंप के अंदर है)। आप _**NotMyFault**_ का उपयोग करके **मेमोरी डंप को फोर्स** भी कर सकते हैं _Sysinternals,_ लेकिन इससे सिस्टम रिबूट होगा और इसे एडमिनिस्ट्रेटर के रूप में निष्पादित किया जाना चाहिए।

आप _**Passware Kit Forensic**_ का उपयोग करके **ब्रूटफोर्स अटैक** भी कर सकते हैं।

### सोशल इंजीनियरिंग

अंत में, आप उपयोगकर्ता को एक नया रिकवरी पासवर्ड जोड़ने के लिए बना सकते हैं उसे एडमिनिस्ट्रेटर के रूप में निष्पादित करवाकर:
```bash
schtasks /create /SC ONLOGON /tr "c:/windows/system32/manage-bde.exe -protectors -add c: -rp 000000-000000-000000-000000-000000-000000-000000-000000" /tn tarea /RU SYSTEM /f
```
यह अगले लॉगिन में एक नयी रिकवरी की (48 शून्यों से बनी) जोड़ देगा।

मान्य रिकवरी कीज़ की जाँच करने के लिए आप निम्न कमांड चला सकते हैं:
```
manage-bde -protectors -get c:
```
<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **अपनी हैकिंग ट्रिक्स साझा करें, HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके.

</details>
