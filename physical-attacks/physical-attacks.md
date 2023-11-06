# शारीरिक हमले

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)

- प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके साझा करें।**

</details>

## BIOS पासवर्ड

### बैटरी

अधिकांश **मदरबोर्ड** में एक **बैटरी** होती है। यदि आप इसे **30 मिनट** के लिए **निकाल दें**, तो BIOS की सेटिंग्स **रीस्टार्ट हो जाएगी** (पासवर्ड सहित)।

### जंपर CMOS

अधिकांश **मदरबोर्ड** में एक **जंपर** होता है जो सेटिंग्स को रीसेट कर सकता है। यदि आप इन पिन्स को जोड़ते हैं तो मदरबोर्ड रीसेट हो जाएगा।

### लाइव टूल्स

यदि आप किसी लाइव CD/USB से उदाहरण के लिए **Kali** Linux चला सकते हैं, तो आप _**killCmos**_ या _**CmosPWD**_ जैसे टूल्स का उपयोग कर सकते हैं (यह अंतिम टूल Kali में शामिल है) और आप **BIOS का पासवर्ड पुनर्प्राप्त करने का प्रयास** कर सकते हैं।

### ऑनलाइन BIOS पासवर्ड रिकवरी

BIOS के पासवर्ड को **3 बार गलत** डालें, फिर BIOS **एक त्रुटि संदेश दिखाएगा** और यह ब्लॉक हो जाएगा।\
[https://bios-pw.org](https://bios-pw.org) पेज पर जाएं और BIOS द्वारा दिखाए गए **त्रुटि कोड** दर्ज करें और आपको भाग्यशाली होकर एक **वैध पासवर्ड** मिल सकता है (एक ही खोज आपको विभिन्न पासवर्ड दिखा सकती है और 1 से अधिक वैध हो सकते हैं)।

## UEFI

UEFI की सेटिंग्स की जांच करने और कुछ प्रकार के हमले करने के लिए आपको [chipsec](https://github.com/chipsec/chipsec/blob/master/chipsec-manual.pdf) का प्रयास करना चाहिए।\
इस टूल का उपयोग करके आप आसानी से Secure Boot को अक्षम कर सकते हैं:
```
python chipsec_main.py -module exploits.secure.boot.pk
```
## RAM

### कोल्ड बूट

**रैम मेमोरी 1 से 2 मिनट तक स्थायी होती है** जब कंप्यूटर बंद होता है। यदि आप मेमोरी कार्ड पर **कोल्ड** (उदाहरण के लिए ताजगी) लागू करें, तो आप इस समय को **10 मिनट तक बढ़ा सकते हैं**।

फिर, आप **dd.exe, mdd.exe, Memoryze, win32dd.exe या DumpIt** जैसे उपकरणों का उपयोग करके **मेमोरी डंप** कर सकते हैं और मेमोरी का विश्लेषण कर सकते हैं।

आपको **volatility** का उपयोग करके मेमोरी का विश्लेषण करना चाहिए।

### [INCEPTION](https://github.com/carmaa/inception)

Inception एक **फिजिकल मेमोरी मैनिपुलेशन** और हैकिंग टूल है जो PCI-आधारित DMA का उपयोग करता है। यह टूल **FireWire**, **Thunderbolt**, **ExpressCard**, PC Card और किसी भी अन्य PCI/PCIe HW इंटरफेस पर हमला कर सकता है।

अपने कंप्यूटर को पीड़ित कंप्यूटर से किसी भी इंटरफेस के माध्यम से **कनेक्ट** करें और **INCEPTION** आपको **पहुंच** देने के लिए **फिजिकल मेमोरी** को **पैच** करने का प्रयास करेगा।

**यदि INCEPTION सफल होता है, तो दर्ज किया गया कोई भी पासवर्ड मान्य होगा।**

**यह Windows10 के साथ काम नहीं करता है।**

## Live CD/USB

### स्टिकी कीज़ और अधिक

* **SETHC:** _sethc.exe_ जब SHIFT 5 बार दबाया जाता है तो इसे आवंटित किया जाता है
* **UTILMAN:** _Utilman.exe_ WINDOWS+U दबाने पर आवंटित किया जाता है
* **OSK:** _osk.exe_ WINDOWS+U दबाने के बाद ऑन-स्क्रीन कीबोर्ड लॉन्च करने के लिए आवंटित किया जाता है
* **DISP:** _DisplaySwitch.exe_ WINDOWS+P दबाने पर आवंटित किया जाता है

ये बाइनरीज़ _**C:\Windows\System32**_ के अंदर स्थित हैं। आप इनमें से किसी भी एक को **cmd.exe** की एक **कॉपी** के लिए बदल सकते हैं (जो भी उसी फ़ोल्डर में है) और जब भी आप इनमें से किसी भी बाइनरी को आवंटित करते हैं, तो एक कमांड प्रॉम्प्ट **सिस्टम** के रूप में प्रदर्शित होगी।

### SAM को संशोधित करना

आप _**chntpw**_ टूल का उपयोग करके माउंट किए गए Windows फ़ाइल सिस्टम के _**SAM**_ **फ़ाइल** को **संशोधित** कर सकते हैं। फिर, आप उदाहरण के लिए Administrator उपयोगकर्ता का पासवर्ड बदल सकते हैं।

यह टूल KALI में उपलब्ध है।
```
chntpw -h
chntpw -l <path_to_SAM>
```
**एक Linux सिस्टम के अंदर आप** _**/etc/shadow**_ **या** _**/etc/passwd**_ **फ़ाइल को संशोधित कर सकते हैं।**

### **Kon-Boot**

**Kon-Boot** एक बेहतरीन उपकरणों में से एक है जो आपको पासवर्ड जाने के बिना Windows में लॉग इन कर सकता है। यह काम करता है **सिस्टम BIOS में हुक करके और Windows कर्नल की सामग्री को अस्थायी रूप से बदलकर** जब बूट करते हैं (नए संस्करणों में **UEFI** के साथ भी काम करता है)। इसके बाद आपको लॉगिन के दौरान **कुछ भी पासवर्ड के रूप में दर्ज करने की अनुमति देता है**। जब आप Kon-Boot के बिना कंप्यूटर को फिर से चालू करते हैं, मूल पासवर्ड वापस आ जाएगा, अस्थायी बदलाव छोड़ दिए जाएंगे और सिस्टम को ऐसा लगेगा जैसे कुछ नहीं हुआ हो।\
अधिक जानें: [https://www.raymond.cc/blog/login-to-windows-administrator-and-linux-root-account-without-knowing-or-changing-current-password/](https://www.raymond.cc/blog/login-to-windows-administrator-and-linux-root-account-without-knowing-or-changing-current-password/)

यह एक लाइव सीडी/USB है जो **मेमोरी को पैच कर सकती है** ताकि आपको लॉगिन करने के लिए पासवर्ड की आवश्यकता नहीं होती है।\
Kon-Boot अवश्यकता के अनुसार **StickyKeys** ट्रिक भी करता है, ताकि आप _**Shift**_ **को 5 बार दबाकर एक व्यवस्थापक cmd प्राप्त कर सकें**।

## **Windows चलाना**

### प्रारंभिक शॉर्टकट

### बूट करने के शॉर्टकट

* supr - BIOS
* f8 - पुनर्प्राप्ति मोड
* _supr_ - BIOS इनी
* _f8_ - पुनर्प्राप्ति मोड
* _Shitf_ (विंडोज बैनर के बाद) - ऑटोलॉगिन की बजाय लॉगिन पेज पर जाएं (ऑटोलॉगिन से बचें)

### **बुरी USBs**

#### **रबर डकी ट्यूटोरियल्स**

* [ट्यूटोरियल 1](https://github.com/hak5darren/USB-Rubber-Ducky/wiki/Tutorials)
* [ट्यूटोरियल 2](https://blog.hartleybrody.com/rubber-ducky-guide/)

#### **टींसीड्यूइनो**

* [पेलोड और ट्यूटोरियल](https://github.com/Screetsec/Pateensy)

अपनी खुद की बुरी USB बनाने के बारे में भी ढेर सारे ट्यूटोरियल हैं।

### वॉल्यूम शैडो कॉपी

व्यवस्थापक विशेषाधिकार और पावरशेल के साथ आप SAM फ़ाइल की प्रतिलिपि बना सकते हैं। [इस कोड को देखें](../windows-hardening/basic-powershell-for-pentesters/#volume-shadow-copy)।

## बिटलॉकर को छलना

बिटलॉकर **2 पासवर्ड** का उपयोग करता है। उपयोगकर्ता द्वारा उपयोग किया जाने वाला एक पासवर्ड और **रिकवरी** पासवर्ड (48 अंकों वाला)।

यदि आप भाग्यशाली हैं और विंडोज के वर्तमान सत्र में _**C:\Windows\MEMORY.DMP**_ फ़ाइल मौजूद है (यह एक मेमोरी डंप है), तो आप कोशिश कर सकते हैं कि आप इसमें से रिकवरी पासवर्ड की खोज करें। आप इस फ़ाइल को **प्राप्त कर सकते हैं** और फ़ाइलसिस्टम की **प्रतिलिपि** का उपयोग करके _Elcomsoft Forensic Disk Decryptor_ का उपयोग करके इसकी सामग्री प्राप्त कर सकते हैं (यह केवल तब काम करेगा जब पासवर्ड मेमोरी डंप में हो)। आप _Sysinternals_ के _**NotMyFault**_ का उपयोग करके मेमोरी डंप को बदलने की कोशिश भी कर सकते हैं, लेकिन इससे सिस्टम रिबूट होगा और इसे व्यवस्थापक के रूप में चलाया जाना चाहिए।

आप _**Passware Kit Forensic**_ का उपयोग करके एक **ब्रूटफ़ोर्स हमला** भी कर सकते हैं।

### सामाजिक इंजीनियरिंग

अंत में, आप उपयोगकर्ता को एक नया रिकवरी पासवर्ड जोड़ने के लिए उन्हें व्यवस्थापक के रूप में चलाने के लिए कह सकते हैं:
```bash
schtasks /create /SC ONLOGON /tr "c:/windows/system32/manage-bde.exe -protectors -add c: -rp 000000-000000-000000-000000-000000-000000-000000-000000" /tn tarea /RU SYSTEM /f
```
यह अगले लॉगिन में एक नया रिकवरी की (48 शून्यों से मिलकर बनाया गया) जोड़ेगा।

मान्य रिकवरी की की जांच करने के लिए आप निम्नलिखित को निष्पादित कर सकते हैं:
```
manage-bde -protectors -get c:
```
<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **हैकट्रिक्स में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करना चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)

- प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके साझा करें।**

</details>
