# भौतिक हमले

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे**? या क्या आप **PEASS के नवीनतम संस्करण तक पहुँचना चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!

- [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह।

- [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें।

- [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) में या **Twitter** पर मुझे **फॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपनी हैकिंग ट्रिक्स साझा करें, [hacktricks repo](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud repo](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके**।

</details>

## BIOS पासवर्ड

### बैटरी

अधिकांश **मदरबोर्ड्स** में एक **बैटरी** होती है। यदि आप इसे **30 मिनट के लिए हटा दें** तो BIOS की सेटिंग्स **रीस्टार्ट** हो जाएंगी (पासवर्ड सहित)।

### जम्पर CMOS

अधिकांश **मदरबोर्ड्स** में एक **जम्पर** होता है जो सेटिंग्स को रीस्टार्ट कर सकता है। यह जम्पर एक केंद्रीय पिन को दूसरे से जोड़ता है, यदि आप **इन पिनों को जोड़ दें तो मदरबोर्ड रीसेट हो जाएगा**।

### लाइव टूल्स

यदि आप उदाहरण के लिए एक **Kali** Linux को Live CD/USB से **चला सकते हैं** तो आप _**killCmos**_ या _**CmosPWD**_ (यह आखिरी वाला Kali में शामिल है) जैसे टूल्स का उपयोग करके **BIOS के पासवर्ड को पुनः प्राप्त करने का प्रयास कर सकते हैं**।

### ऑनलाइन BIOS पासवर्ड रिकवरी

BIOS का पासवर्ड **3 बार गलत डालें**, फिर BIOS एक **त्रुटि संदेश दिखाएगा** और यह ब्लॉक हो जाएगा।\
[https://bios-pw.org](https://bios-pw.org) पेज पर जाएं और BIOS द्वारा दिखाए गए **त्रुटि कोड को डालें** और आप भाग्यशाली हो सकते हैं और एक **मान्य पासवर्ड प्राप्त कर सकते हैं** (एक ही खोज से आपको विभिन्न पासवर्ड दिखाई दे सकते हैं और एक से अधिक मान्य हो सकते हैं)।

## UEFI

UEFI की सेटिंग्स की जांच करने और किसी प्रकार का हमला करने के लिए आपको [chipsec](https://github.com/chipsec/chipsec/blob/master/chipsec-manual.pdf) का प्रयास करना चाहिए।\
इस टूल का उपयोग करके आप आसानी से Secure Boot को अक्षम कर सकते हैं:
```
python chipsec_main.py -module exploits.secure.boot.pk
```
## RAM

### कोल्ड बूट

**RAM मेमोरी 1 से 2 मिनट तक पर्सिस्टेंट रहती है** जब कंप्यूटर बंद होता है। यदि आप मेमोरी कार्ड पर **कोल्ड** (उदाहरण के लिए लिक्विड नाइट्रोजन) लगाते हैं, तो आप इस समय को **10 मिनट तक** बढ़ा सकते हैं।

फिर, आप **मेमोरी डंप** कर सकते हैं (dd.exe, mdd.exe, Memoryze, win32dd.exe या DumpIt जैसे टूल्स का उपयोग करके) मेमोरी का विश्लेषण करने के लिए।

आपको मेमोरी का **विश्लेषण volatility का उपयोग करके** करना चाहिए।

### [INCEPTION](https://github.com/carmaa/inception)

Inception एक **फिजिकल मेमोरी मैनिपुलेशन** और हैकिंग टूल है जो PCI-आधारित DMA का शोषण करता है। यह टूल **FireWire**, **Thunderbolt**, **ExpressCard**, PC Card और किसी भी अन्य PCI/PCIe HW इंटरफेसेज पर हमला कर सकता है।\
अपने कंप्यूटर को उन **इंटरफेसेज** में से किसी एक के माध्यम से पीड़ित कंप्यूटर से **कनेक्ट** करें और **INCEPTION** आपको **एक्सेस** देने के लिए **फिजिकल मेमोरी** में **पैच** लगाने की कोशिश करेगा।

**यदि INCEPTION सफल होता है, तो कोई भी पासवर्ड मान्य होगा।**

**यह Windows10 के साथ काम नहीं करता।**

## लाइव CD/USB

### स्टिकी कीज और अधिक

* **SETHC:** _sethc.exe_ को तब इन्वोक किया जाता है जब SHIFT को 5 बार दबाया जाता है
* **UTILMAN:** _Utilman.exe_ को WINDOWS+U दबाने पर इन्वोक किया जाता है
* **OSK:** _osk.exe_ को WINDOWS+U दबाने और फिर ऑन-स्क्रीन कीबोर्ड लॉन्च करने पर इन्वोक किया जाता है
* **DISP:** _DisplaySwitch.exe_ को WINDOWS+P दबाने पर इन्वोक किया जाता है

ये बाइनरीज _**C:\Windows\System32**_ के अंदर स्थित हैं। आप उनमें से किसी को भी **cmd.exe** की एक **कॉपी** से **बदल** सकते हैं (जो उसी फोल्डर में भी है) और जब भी आप उन बाइनरीज में से किसी को इन्वोक करेंगे एक कमांड प्रॉम्प्ट **SYSTEM** के रूप में प्रकट होगा।

### SAM को मॉडिफाई करना

आप एक माउंटेड Windows फाइलसिस्टम की _**SAM**_ **फाइल को मॉडिफाई करने के लिए** _**chntpw**_ टूल का उपयोग कर सकते हैं। फिर, आप उदाहरण के लिए, Administrator यूजर का पासवर्ड बदल सकते हैं।\
यह टूल KALI में उपलब्ध है।
```
chntpw -h
chntpw -l <path_to_SAM>
```
**लिनक्स सिस्टम के अंदर आप** _**/etc/shadow**_ **या** _**/etc/passwd**_ **फाइल को मोडिफाई कर सकते हैं।**

### **कोन-बूट**

**कोन-बूट** बेहतरीन टूल्स में से एक है जो आपको विंडोज में पासवर्ड जाने बिना लॉग इन करने देता है। यह सिस्टम BIOS में **हुकिंग** करके और बूटिंग के दौरान विंडोज कर्नेल की सामग्री को अस्थायी रूप से बदलकर काम करता है (नए संस्करण **UEFI** के साथ भी काम करते हैं)। इसके बाद यह आपको लॉगिन के दौरान **कुछ भी पासवर्ड के रूप में दर्ज करने की अनुमति देता है**। कोन-बूट के बिना अगली बार जब आप कंप्यूटर स्टार्ट करेंगे, मूल पासवर्ड वापस आ जाएगा, अस्थायी परिवर्तन त्याग दिए जाएंगे और सिस्टम ऐसा व्यवहार करेगा जैसे कुछ हुआ ही नहीं।\
और पढ़ें: [https://www.raymond.cc/blog/login-to-windows-administrator-and-linux-root-account-without-knowing-or-changing-current-password/](https://www.raymond.cc/blog/login-to-windows-administrator-and-linux-root-account-without-knowing-or-changing-current-password/)

यह एक लाइव CD/USB है जो **मेमोरी को पैच** कर सकता है ताकि आपको लॉगिन करने के लिए पासवर्ड जानने की **जरूरत नहीं पड़े**।\
कोन-बूट **StickyKeys** ट्रिक भी करता है ताकि आप _**Shift**_ **को 5 बार दबाकर एक एडमिनिस्ट्रेटर cmd प्राप्त कर सकें**।

## **विंडोज चलाना**

### प्रारंभिक शॉर्टकट्स

### बूटिंग शॉर्टकट्स

* supr - BIOS
* f8 - Recovery mode
* _supr_ - BIOS ini
* _f8_ - Recovery mode
* _Shitf_ (विंडोज बैनर के बाद) - ऑटोलॉगन के बजाय लॉगिन पेज पर जाएं (ऑटोलॉगन से बचें)

### **बैड USBs**

#### **रबर डकी ट्यूटोरियल्स**

* [ट्यूटोरियल 1](https://github.com/hak5darren/USB-Rubber-Ducky/wiki/Tutorials)
* [ट्यूटोरियल 2](https://blog.hartleybrody.com/rubber-ducky-guide/)

#### **Teensyduino**

* [पेलोड्स और ट्यूटोरियल्स](https://github.com/Screetsec/Pateensy)

अपना खुद का बैड USB बनाने के बारे में भी ढेर सारे ट्यूटोरियल्स हैं।

### Volume Shadow Copy

एडमिनिस्ट्रेटर प्रिविलेजेस और पावरशेल के साथ आप SAM फाइल की एक कॉपी बना सकते हैं।[ इस कोड को देखें](../windows-hardening/basic-powershell-for-pentesters/#volume-shadow-copy)।

## बिटलॉकर को बायपास करना

बिटलॉकर **2 पासवर्ड** का उपयोग करता है। एक **उपयोगकर्ता** द्वारा इस्तेमाल किया जाने वाला, और **रिकवरी** पासवर्ड (48 अंक)।

यदि आप भाग्यशाली हैं और वर्तमान विंडोज सत्र में _**C:\Windows\MEMORY.DMP**_ फाइल मौजूद है (यह एक मेमोरी डंप है) तो आप **रिकवरी पासवर्ड को इसके अंदर खोजने की कोशिश कर सकते हैं**। आप इस फाइल को **प्राप्त कर सकते हैं** और फाइलसिस्टम की एक **कॉपी** ले सकते हैं और फिर _Elcomsoft Forensic Disk Decryptor_ का उपयोग करके सामग्री प्राप्त कर सकते हैं (यह केवल तभी काम करेगा जब पासवर्ड मेमोरी डंप के अंदर हो)। आप _**NotMyFault**_ का उपयोग करके मेमोरी डंप को **जबरदस्ती कर सकते हैं**, लेकिन इससे सिस्टम रिबूट होगा और इसे एडमिनिस्ट्रेटर के रूप में निष्पादित किया जाना चाहिए।

आप _**Passware Kit Forensic**_ का उपयोग करके एक **ब्रूटफोर्स अटैक** भी आजमा सकते हैं।

### सोशल इंजीनियरिंग

अंत में, आप उपयोगकर्ता को एक नया रिकवरी पासवर्ड जोड़ने के लिए बना सकते हैं, उसे एडमिनिस्ट्रेटर के रूप में निष्पादित करवाकर:
```bash
schtasks /create /SC ONLOGON /tr "c:/windows/system32/manage-bde.exe -protectors -add c: -rp 000000-000000-000000-000000-000000-000000-000000-000000" /tn tarea /RU SYSTEM /f
```
यह अगले लॉगिन में एक नया रिकवरी की (48 शून्यों से बना) जोड़ देगा।

मान्य रिकवरी कीज़ की जांच करने के लिए आप निम्न कमांड चला सकते हैं:
```
manage-bde -protectors -get c:
```
<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप **साइबरसिक्योरिटी कंपनी** में काम करते हैं? क्या आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे**? या क्या आप **PEASS के नवीनतम संस्करण तक पहुँच चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!

- [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह।

- [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें।

- [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) में या **Twitter** पर मुझे **फॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपनी हैकिंग ट्रिक्स साझा करें, [hacktricks repo](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud repo](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके**।

</details>
