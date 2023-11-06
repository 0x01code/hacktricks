# PsExec/Winexec/ScExec

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)

- प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **Twitter** पर **फॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके साझा करें**.

</details>

## वे कैसे काम करते हैं

1. SMB के माध्यम से सेवा बाइनरी की प्रतिलिपि ADMIN$ शेयर में करें
2. दूरस्थ मशीन पर एक सेवा बनाएं जो बाइनरी को निशानित करती है
3. सेवा को दूरस्थ रूप से शुरू करें
4. बाइनरी को बंद करने पर, सेवा को रोकें और हटाएं

## **मैन्युअल PsExec का उपयोग करना**

सबसे पहले यह मान लेते हैं कि हमारे पास एक पेलोड एक्जीक्यूटेबल है जिसे हमने msfvenom के साथ उत्पन्न किया है और Veil के साथ अस्पष्ट किया है (ताकि AV इसे न चिह्नित करें)। इस मामले में, मैंने एक मीटरप्रेटर रिवर्स\_एचटीटीपी पेलोड बनाई और इसे 'met8888.exe' नामक रखा है

**बाइनरी की प्रतिलिपि करें**। "जर्रिएटा" कमांड प्रॉम्प्ट से, सीधे बाइनरी की प्रतिलिपि ADMIN$ में करें। हालांकि, यह किसी भी फाइल सिस्टम पर कहीं भी कॉपी और छिपा सकता है।

![](../../.gitbook/assets/copy\_binary\_admin.png)

**एक सेवा बनाएं**। Windows `sc` कमांड का उपयोग करके Windows सेवाओं का प्रश्न करने, बनाने, हटाने, आदि करने के लिए उपयोग किया जाता है और इसे दूरस्थ रूप से उपयोग किया जा सकता है। इसके बारे में और अधिक पढ़ें [यहां](https://technet.microsoft.com/en-us/library/bb490995.aspx)। हमारे कमांड प्रॉम्प्ट से, हम दूरस्थ रूप से एक सेवा बनाएंगे जिसे "मीटरप्रेटर" कहा जाता है और हमारे अपलोड किए गए बाइनरी को निशानित करता है:

![](../../.gitbook/assets/sc\_create.png)

**सेवा शुरू करें**। अंतिम चरण है सेवा शुरू करना और बाइनरी को निष्पादित करना। _नोट:_ जब सेवा शुरू होती है, तो यह "समय समाप्त हो जाएगी" और एक त्रुटि उत्पन्न करेगी। यह इसलिए है क्योंकि हमारा मीटरप्रेटर बाइनरी वास्तविक सेवा बाइनरी नहीं है और उम्मीद की जाने वाली प्रतिक्रिया को वापस नहीं करेगा। यह ठीक है क्योंकि हमें इसे केवल एक बार निष्पादित करने की जरूरत है:

![](../../.gitbook/assets/sc\_start\_error.png)

यदि हम अपने Metasploit सुनने वाले को देखें, तो हमें देखेंगे कि सत्र खोल दिया गया है।

**सेवा को साफ करें।**

![](../../.gitbook/assets/sc\_delete.png)

यहां से निकाला गया है: [https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/](https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/)

**आप Windows Sysinternals बाइनरी PsExec.exe का भी उपयोग कर सकते हैं:**

![](<../../.gitbook/assets/image (165).png>)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/s
