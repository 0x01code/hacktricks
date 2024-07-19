# Physical Attacks

{% hint style="success" %}
Learn & practice AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Learn & practice GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Check the [**subscription plans**](https://github.com/sponsors/carlospolop)!
* **Join the** 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) or the [**telegram group**](https://t.me/peass) or **follow** us on **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Share hacking tricks by submitting PRs to the** [**HackTricks**](https://github.com/carlospolop/hacktricks) and [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}

### [WhiteIntel](https://whiteintel.io)

<figure><img src="/.gitbook/assets/image (1224).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) एक **डार्क-वेब** द्वारा संचालित सर्च इंजन है जो **मुफ्त** कार्यक्षमताएँ प्रदान करता है ताकि यह जांचा जा सके कि कोई कंपनी या उसके ग्राहक **संपर्कित** हैं या नहीं **चोरी करने वाले मालवेयर** द्वारा।

WhiteIntel का प्राथमिक लक्ष्य जानकारी चुराने वाले मालवेयर के परिणामस्वरूप खाता अधिग्रहण और रैनसमवेयर हमलों से लड़ना है।

आप उनकी वेबसाइट पर जा सकते हैं और **मुफ्त** में उनके इंजन का प्रयास कर सकते हैं:

{% embed url="https://whiteintel.io" %}

---

## BIOS पासवर्ड पुनर्प्राप्ति और सिस्टम सुरक्षा

**BIOS को रीसेट करना** कई तरीकों से किया जा सकता है। अधिकांश मदरबोर्ड में एक **बैटरी** होती है, जिसे लगभग **30 मिनट** के लिए हटाने पर BIOS सेटिंग्स, जिसमें पासवर्ड भी शामिल है, रीसेट हो जाती हैं। वैकल्पिक रूप से, **मदरबोर्ड पर एक जंपर** को समायोजित किया जा सकता है ताकि इन सेटिंग्स को विशिष्ट पिनों को जोड़कर रीसेट किया जा सके।

उन स्थितियों के लिए जहां हार्डवेयर समायोजन संभव या व्यावहारिक नहीं हैं, **सॉफ़्टवेयर उपकरण** एक समाधान प्रदान करते हैं। **Kali Linux** जैसी वितरणों के साथ **Live CD/USB** से सिस्टम चलाने पर **_killCmos_** और **_CmosPWD_** जैसे उपकरणों तक पहुंच मिलती है, जो BIOS पासवर्ड पुनर्प्राप्ति में मदद कर सकते हैं।

यदि BIOS पासवर्ड अज्ञात है, तो इसे गलत तरीके से **तीन बार** दर्ज करने पर आमतौर पर एक त्रुटि कोड प्राप्त होता है। इस कोड का उपयोग [https://bios-pw.org](https://bios-pw.org) जैसी वेबसाइटों पर किया जा सकता है ताकि संभावित रूप से एक उपयोगी पासवर्ड प्राप्त किया जा सके।

### UEFI सुरक्षा

आधुनिक सिस्टम के लिए जो पारंपरिक BIOS के बजाय **UEFI** का उपयोग करते हैं, **chipsec** उपकरण का उपयोग UEFI सेटिंग्स का विश्लेषण और संशोधन करने के लिए किया जा सकता है, जिसमें **Secure Boot** को अक्षम करना शामिल है। इसे निम्नलिखित कमांड के साथ पूरा किया जा सकता है:

`python chipsec_main.py -module exploits.secure.boot.pk`

### RAM विश्लेषण और कोल्ड बूट हमले

RAM डेटा को पावर कटने के बाद थोड़े समय के लिए बनाए रखता है, आमतौर पर **1 से 2 मिनट**। इस स्थिरता को ठंडे पदार्थों, जैसे तरल नाइट्रोजन, को लागू करके **10 मिनट** तक बढ़ाया जा सकता है। इस विस्तारित अवधि के दौरान, **मेमोरी डंप** बनाया जा सकता है जिसका विश्लेषण करने के लिए **dd.exe** और **volatility** जैसे उपकरणों का उपयोग किया जा सकता है।

### डायरेक्ट मेमोरी एक्सेस (DMA) हमले

**INCEPTION** एक उपकरण है जो **फिजिकल मेमोरी मैनिपुलेशन** के लिए DMA के माध्यम से डिज़ाइन किया गया है, जो **FireWire** और **Thunderbolt** जैसी इंटरफेस के साथ संगत है। यह किसी भी पासवर्ड को स्वीकार करने के लिए मेमोरी को पैच करके लॉगिन प्रक्रियाओं को बायपास करने की अनुमति देता है। हालाँकि, यह **Windows 10** सिस्टम के खिलाफ प्रभावी नहीं है।

### सिस्टम एक्सेस के लिए लाइव CD/USB

**_sethc.exe_** या **_Utilman.exe_** जैसे सिस्टम बाइनरी को **_cmd.exe_** की एक प्रति के साथ बदलने से सिस्टम विशेषाधिकारों के साथ एक कमांड प्रॉम्प्ट प्राप्त किया जा सकता है। **chntpw** जैसे उपकरणों का उपयोग Windows इंस्टॉलेशन की **SAM** फ़ाइल को संपादित करने के लिए किया जा सकता है, जिससे पासवर्ड परिवर्तन की अनुमति मिलती है।

**Kon-Boot** एक उपकरण है जो Windows सिस्टम में लॉगिन करने की सुविधा प्रदान करता है बिना पासवर्ड जाने, Windows कर्नेल या UEFI को अस्थायी रूप से संशोधित करके। अधिक जानकारी [https://www.raymond.cc](https://www.raymond.cc/blog/login-to-windows-administrator-and-linux-root-account-without-knowing-or-changing-current-password/) पर मिल सकती है।

### Windows सुरक्षा सुविधाओं को संभालना

#### बूट और पुनर्प्राप्ति शॉर्टकट

- **Supr**: BIOS सेटिंग्स तक पहुँचें।
- **F8**: पुनर्प्राप्ति मोड में प्रवेश करें।
- Windows बैनर के बाद **Shift** दबाने से ऑटो-लॉगिन बायपास हो सकता है।

#### BAD USB उपकरण

**Rubber Ducky** और **Teensyduino** जैसे उपकरण **बद USB** उपकरण बनाने के लिए प्लेटफार्मों के रूप में कार्य करते हैं, जो लक्षित कंप्यूटर से जुड़े होने पर पूर्वनिर्धारित पेलोड को निष्पादित करने में सक्षम होते हैं।

#### वॉल्यूम शैडो कॉपी

व्यवस्थापक विशेषाधिकार संवेदनशील फ़ाइलों की प्रतियों को बनाने की अनुमति देते हैं, जिसमें PowerShell के माध्यम से **SAM** फ़ाइल शामिल है।

### BitLocker एन्क्रिप्शन को बायपास करना

यदि **पुनर्प्राप्ति पासवर्ड** एक मेमोरी डंप फ़ाइल (**MEMORY.DMP**) में पाया जाता है, तो BitLocker एन्क्रिप्शन को संभावित रूप से बायपास किया जा सकता है। इस उद्देश्य के लिए **Elcomsoft Forensic Disk Decryptor** या **Passware Kit Forensic** जैसे उपकरणों का उपयोग किया जा सकता है।

### पुनर्प्राप्ति कुंजी जोड़ने के लिए सामाजिक इंजीनियरिंग

एक नया BitLocker पुनर्प्राप्ति कुंजी सामाजिक इंजीनियरिंग तकनीकों के माध्यम से जोड़ी जा सकती है, एक उपयोगकर्ता को एक कमांड निष्पादित करने के लिए मनाकर जो शून्य से बनी एक नई पुनर्प्राप्ति कुंजी जोड़ता है, जिससे डिक्रिप्शन प्रक्रिया को सरल बनाया जा सके।

### [WhiteIntel](https://whiteintel.io)

<figure><img src="/.gitbook/assets/image (1224).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) एक **डार्क-वेब** द्वारा संचालित सर्च इंजन है जो **मुफ्त** कार्यक्षमताएँ प्रदान करता है ताकि यह जांचा जा सके कि कोई कंपनी या उसके ग्राहक **संपर्कित** हैं या नहीं **चोरी करने वाले मालवेयर** द्वारा।

WhiteIntel का प्राथमिक लक्ष्य जानकारी चुराने वाले मालवेयर के परिणामस्वरूप खाता अधिग्रहण और रैनसमवेयर हमलों से लड़ना है।

आप उनकी वेबसाइट पर जा सकते हैं और **मुफ्त** में उनके इंजन का प्रयास कर सकते हैं:

{% embed url="https://whiteintel.io" %}

{% hint style="success" %}
Learn & practice AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Learn & practice GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Check the [**subscription plans**](https://github.com/sponsors/carlospolop)!
* **Join the** 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) or the [**telegram group**](https://t.me/peass) or **follow** us on **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Share hacking tricks by submitting PRs to the** [**HackTricks**](https://github.com/carlospolop/hacktricks) and [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}
