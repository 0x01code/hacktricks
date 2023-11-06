# ZIPs ट्रिक्स

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks क्लाउड ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 ट्विटर 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ ट्विच 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 यूट्यूब 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करना चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल हों** या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स को** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में पीआर जमा करके** अपना योगदान दें।

</details>

जिप फ़ाइलों के लिए कुछ कमांड-लाइन टूल हैं जिनका उपयोग करना उपयोगी होगा।

* `unzip` अक्सर जिप को डीकंप्रेस करने के बारे में मददगार जानकारी देता है।
* `zipdetails -v` प्रारूप के विभिन्न फ़ील्ड में मौजूद मानों के बारे में विस्तृत जानकारी प्रदान करेगा।
* `zipinfo` जिप फ़ाइल की सामग्री के बारे में जानकारी सूचीबद्ध करता है, इसे निकालता नहीं है।
* `zip -F input.zip --out output.zip` और `zip -FF input.zip --out output.zip` एक क्षतिग्रस्त जिप फ़ाइल को मरम्मत करने का प्रयास करेंगे।
* [fcrackzip](https://github.com/hyc/fcrackzip) जिप पासवर्ड को ब्रूट-फ़ोर्स गेस करता है (पासवर्ड <7 वर्ण या इससे कम होता है)।

[जिप फ़ाइल प्रारूप विनिर्देशिका](https://pkware.cachefly.net/webdocs/casestudies/APPNOTE.TXT)

पासवर्ड से सुरक्षित जिप फ़ाइलों के बारे में एक महत्वपूर्ण सुरक्षा संबंधित नोट है कि वे संपीड़ित फ़ाइलों के नाम और मूल फ़ाइल के आकार को एन्क्रिप्ट नहीं करते हैं, बारीक रूप में पासवर्ड से सुरक्षित RAR या 7z फ़ाइलों की तुलना में।

जिप क्रैकिंग के बारे में एक और नोट है कि यदि आपके पास एन्क्रिप्टेड जिप में संपीड़ित होने वाली किसी भी फ़ाइल की एक अनएन्क्रिप्टेड/अनसंपीड़ित प्रतिलिपि है, तो आप एक "प्लेनटेक्स्ट हमला" कर सकते हैं और जिप को क्रैक कर सकते हैं, जैसा कि [यहां](https://www.hackthis.co.uk/articles/known-plaintext-attack-cracking-zip-files) विस्तार से बताया गया है, और [इस पेपर](https://www.cs.auckland.ac.nz/\~mike/zipattacks.pdf) में समझाया गया है। जिप फ़ाइलों को पासवर्ड से सुरक्षित करने की नई योजना (AES-256 के साथ, "ZipCrypto" के बजाय) में यह कमजोरी नहीं है।

स्रोत: [https://app.gitbook.com/@cpol/s/hacktricks/\~/edit/drafts/-LlM5mCby8ex5pOeV4pJ/forensics/basic-forensics-esp/zips-tricks](http://127.0.0.1:5000/o/Iwnw24TnSs9D9I2OtTKX/s/-L\_2uGJGU7AVNRcqRvEi/)
