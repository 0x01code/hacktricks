<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप चाहते हैं कि **आपकी कंपनी HackTricks में विज्ञापित हो** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)** पर **फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>

## Firmware Integrity

**कस्टम फर्मवेयर और/या कंपाइल किए गए बाइनरी कोड को अखंडता या हस्ताक्षर सत्यापन दोषों का शिकार करने के लिए अपलोड किया जा सकता है**। निम्नलिखित चरण बैकडोर बाइंड शैल कॉम्पाइलेशन के लिए अनुसरण किया जा सकता है:

1. फर्मवेयर को फर्मवेयर-मॉड-किट (FMK) का उपयोग करके निकाला जा सकता है।
2. लक्षित फर्मवेयर आर्किटेक्चर और एंडियनेस को पहचाना जाना चाहिए।
3. एन्वायरनमेंट के लिए बिल्डरूट या अन्य उपयुक्त विधियों का उपयोग करके एक क्रॉस कंपाइलर तैयार किया जा सकता है।
4. क्रॉस कंपाइलर का उपयोग करके बैकडोर तैयार किया जा सकता है।
5. बैकडोर को निकाले गए फर्मवेयर /usr/bin निर्देशिका में कॉपी किया जा सकता है।
6. उचित QEMU बाइनरी को निकाले गए फर्मवेयर rootfs में कॉपी किया जा सकता है।
7. chroot और QEMU का उपयोग करके बैकडोर को अनुकरण किया जा सकता है।
8. बैकडोर को netcat के माध्यम से एक्सेस किया जा सकता है।
9. QEMU बाइनरी को निकाले गए फर्मवेयर rootfs से हटा दिया जाना चाहिए।
10. फर्मवेयर को फिर से FMK का उपयोग करके पैकेज किया जा सकता है।
11. बैकडोर फर्मवेयर को टेस्ट करने के लिए इसे फर्मवेयर विश्लेषण उपकरण (FAT) के साथ अनुकरण करके और netcat का उपयोग करके लक्षित बैकडोर IP और पोर्ट से कनेक्ट करके टेस्ट किया जा सकता है।

यदि पहले से ही डायनामिक विश्लेषण, बूटलोडर परिवर्तन या हार्डवेयर सुरक्षा परीक्षण के माध्यम से रूट शैल प्राप्त किया गया है, तो पूर्व-कॉम्पाइल किए गए दुरुपयोगी बाइनरी जैसे इम्प्लांट्स या रिवर्स शैल्स को निष्पादित किया जा सकता है। 'Msfvenom' के जैसे स्वचालित पेलोड/इम्प्लांट उपकरणों का उपयोग करके Metasploit फ्रेमवर्क और एमएसएफवेनम का उपयोग करके निम्नलिखित चरणों का उपयोग किया जा सकता है:

1. लक्षित फर्मवेयर आर्किटेक्चर और एंडियनेस को पहचाना जाना चाहिए।
2. Msfvenom का उपयोग करके लक्षित पेलोड, हमलावर होस्ट आईपी, सुनने वाले पोर्ट नंबर, फाइल प्रकार, आर्किटेक्चर, प्लेटफॉर्म, और आउटपुट फाइल को निर्दिष्ट किया जा सकता है।
3. पेलोड को प्रभावित उपकरण पर स्थानांतरित किया जा सकता है और सुनिश्चित किया जा सकता है कि इसमें क्रियान्वयन अनुमतियाँ हैं।
4. पेलोड को संदिग्ध उपकरण पर भेजने के लिए Metasploit को तैयार किया जा सकता है, msfconsole शुरू करके और सेटिंग्स को पेलोड के अनुसार कॉन्फ़िगर करके।
5. संदिग्ध उपकरण पर मीटरप्रीटर रिवर्स शैल को निष्पादित किया जा सकता है।
6. मीटरप्रीटर सेशन को खोलते समय मॉनिटर किया जा सकता है।
7. पोस्ट-एक्सप्लोइटेशन गतिविधियाँ की जा सकती हैं।

यदि संभव हो, स्टार्टअप स्क्रिप्ट्स में मौजूद दोषों का उपयोग करके एक उपकरण को बूट के बाद एक उपकरण में स्थायी पहुंच प्राप्त करने के लिए उत्पन्न किया जा सकता है। ये दोष स्थानांकित स्थानों पर कोड का संदर्भ करते हैं, [प्रतीकात्मक लिंक](https://www.chromium.org/chromium-os/chromiumos-design-docs/hardening-against-malicious-stateful-data) करते हैं, या अविश्वसनीय माउंट किए गए स्थानों में स्थित डेटा संग्रहित करने के लिए उपयोग किए जाने वाले SD कार्ड और फ्लैश वॉल्यूम्स में उपयुक्तता के दौरान उत्पन्न होते हैं।

## संदर्भ
* अधिक जानकारी के लिए देखें [https://scriptingxss.gitbook.io/firmware-security-testing-methodology/](https://scriptingxss.gitbook.io/firmware-security-testing-methodology/)

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप चाहते हैं कि **आपकी कंपनी HackTricks में विज्ञापित हो** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)** पर **फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
