<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने की आवश्यकता** है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT संग्रह**](https://opensea.io/collection/the-peass-family)

- [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके साझा करें**.

</details>


### इस पृष्ठ की प्रतिलिपि [https://scriptingxss.gitbook.io/firmware-security-testing-methodology/](https://scriptingxss.gitbook.io/firmware-security-testing-methodology/) से ली गई है

**फर्मवेयर और/या कंपाइल किए गए बाइनरी** को अखंडता या हस्ताक्षर सत्यापन दोषों के लिए अपलोड करने का प्रयास करें। उदाहरण के लिए, निम्नलिखित चरणों का पालन करके बूट होने पर शुरू होने वाला एक बैकडोर बाइंड शेल कंपाइल करें।

1. फर्मवेयर को फर्मवेयर-मॉड-किट (FMK) के साथ निकालें
2. लक्षित फर्मवेयर आर्किटेक्चर और एंडियनेस की पहचान करें
3. बिल्डरूट के साथ एक क्रॉस कंपाइलर बनाएं या अपने पर्यावरण के अनुसार अन्य विधियों का उपयोग करें
4. क्रॉस कंपाइलर का उपयोग करके बैकडोर को बनाएं
5. बैकडोर को निकाले गए फर्मवेयर /usr/bin में कॉपी करें
6. उत्पन्न फर्मवेयर रूटफ़ाइल पर उचित QEMU बाइनरी कॉपी करें
7. chroot और QEMU का उपयोग करके बैकडोर को नकली करें
8. netcat के माध्यम से बैकडोर से कनेक्ट करें
9. निकाले गए फर्मवेयर रूटफ़ाइल से QEMU बाइनरी को हटाएं
10. FMK के साथ संशोधित फर्मवेयर को पुनः पैकेज करें
11. फर्मवेयर विश्लेषण टूलकिट (FAT) के साथ नकली फर्मवेयर का परीक्षण करें और netcat का उपयोग करके लक्षित बैकडोर IP और पोर्ट से कनेक्ट करें

यदि डायनेमिक विश्लेषण, बूटलोडर मानिपुलेशन या हार्डवेयर सुरक्षा परीक्षण के साधनों से पहले ही रूट शेल प्राप्त की गई है, तो ऐसे पूर्व-कंपाइल किए गए दुष्प्रभावी बाइनरी (जैसे implants या reverse shells) को निष्पादित करने का प्रयास करें। कमांड और नियंत्रण (C\&C) ढांचा के लिए उपयोग किए जाने वाले स्वचालित payload/implant उपकरणों का उपयोग करने का विचार करें। उदाहरण के लिए, Metasploit framework और 'msfvenom' का उपयोग निम्नलिखित चरणों का पालन करके किया जा सकता है।

1. लक्षित फर्मवेयर आर्किटेक्चर और एंडियनेस की पहचान करें
2. `msfvenom` का उपयोग करके उचित लक्षित payload (-p), हमलावर होस्ट IP (LHOST=), सुनने व
