<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करना चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)

- [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या **Twitter** पर **फॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **हैकिंग ट्रिक्स साझा करें** [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में PR जमा करके।

</details>


From: [https://trailofbits.github.io/ctf/forensics/](https://trailofbits.github.io/ctf/forensics/)

छवि फ़ाइल प्रारूपों की तरह, ऑडियो और वीडियो फ़ाइल चालाकी CTF फोरेंसिक्स चुनौतियों में एक सामान्य विषय है, न कि इस तरह वास्तविक दुनिया में कभी हैकिंग या डेटा छिपाने का कोई उपयोग होता है, बल्कि केवल इसलिए कि ऑडियो और वीडियो मजेदार होते हैं। छवि फ़ाइल प्रारूपों की तरह, स्टेगनोग्राफी का उपयोग सामग्री डेटा में एक गुप्त संदेश समाहित करने के लिए किया जा सकता है, और फिर से आपको संकेतों के लिए फ़ाइल मेटाडेटा क्षेत्रों की जांच करनी चाहिए। आपका पहला कदम [mediainfo](https://mediaarea.net/en/MediaInfo) टूल (या `exiftool`) के साथ एक नज़र डालना चाहिए और सामग्री प्रकार की पहचान करनी चाहिए और इसके मेटाडेटा पर देखना चाहिए।

[Audacity](http://www.audacityteam.org/) प्रमुख ओपन-सोर्स ऑडियो फ़ाइल और तार देखने वाला उपकरण है। CTF चुनौती लेखकों को पाठ में पाठ संकेतों को एक्सपोर्ट करने के लिए ऑडियो तारों में पाठ को कोड करने का बहुत पसंद है, जिसे आप स्पेक्ट्रोग्राम दृश्य का उपयोग करके देख सकते हैं (हालांकि इस कार्य के लिए एक विशेष उपकरण जिसे [Sonic Visualiser](http://www.sonicvisualiser.org/) कहा जाता है, इससे बेहतर है)। Audacity आपको धीमा करने, उलटा करने और अन्य परिवर्तन करने की अनुमति देता है जो यदि आपको लगता है कि एक छिपा हुआ संदेश है तो एक छिपा हुआ संदेश प्रकट कर सकता है (यदि आप गड़बड़ ऑडियो, अवरोधन या स्थिरता सुन सकते हैं)। [Sox](http://sox.sourceforge.net/) ऑडियो फ़ाइलों को परिवर्तित और प्रबंधित करने के लिए एक उपयोगी कमांड-लाइन उपकरण है।

एक गुप्त संदेश के लिए अद्यतन लेस्ट साइनिफिकेंट बिट्स (LSB) की जांच भी सामान्य है। अधिकांश ऑडियो और वीडियो मीडिया प्रारूप विभाजनशील (निश्चित आकार) "चंक" का उपयोग करते हैं ताकि वे स्ट्रीम किए जा सकें; उन चंकों के LSB सामान्य जगह हैं जहां फ़ाइल को दिखाई नहीं देते हुए कुछ डेटा छिपाया जा सकता है।

कभी-कभी, एक संदेश को ऑडियो में [DTMF टोन](http://dialabc.com/sound/detect/index.html) या मोर्स कोड के रूप में कोड किया जा सकता है। इनके लिए, उन्हें डिकोड कर
