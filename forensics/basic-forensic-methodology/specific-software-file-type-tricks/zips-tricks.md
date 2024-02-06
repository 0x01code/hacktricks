# ZIPs ट्रिक्स

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** पर **फॉलो** करें 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

zip फ़ाइलों के लिए कुछ कमांड-लाइन टूल हैं जिन्हें जानना उपयोगी होगा।

* `unzip` अक्सर zip को डीकंप्रेस क्यों नहीं करेगा, इस पर मददगार जानकारी देगा।
* `zipdetails -v` प्रारूप के विभिन्न क्षेत्रों में मौजूद मानों पर विस्तृत जानकारी प्रदान करेगा।
* `zipinfo` zip फ़ाइल की सामग्री के बारे में जानकारी सूचीबद्ध करेगा, बिना इसे निकाले।
* `zip -F input.zip --out output.zip` और `zip -FF input.zip --out output.zip` एक क्षतिग्रस्त zip फ़ाइल को मरम्मत करने का प्रयास करेंगे।
* [fcrackzip](https://github.com/hyc/fcrackzip) brute-force एक zip पासवर्ड की अनुमानित अनुमान लगाता है (पासवर्ड <7 वर्ण या इससे कम के लिए)।

[Zip फ़ाइल प्रारूप विनिर्देशिका](https://pkware.cachefly.net/webdocs/casestudies/APPNOTE.TXT)

पासवर्ड से सुरक्षा संबंधित एक महत्वपूर्ण नोट zip फ़ाइलों के बारे में यह है कि वे नामों और संक्षिप्त फ़ाइलों के मूल आकार को नहीं एन्क्रिप्ट करते हैं, उनके बीच संक्षिप्त फ़ाइलों के लिए पासवर्ड सुरक्षित RAR या 7z फ़ाइलों की तरह।

एक और नोट zip क्रैकिंग के बारे में यह है कि यदि आपके पास एन्क्रिप्ट नहीं/अनकंप्रेस किया हुआ किसी भी एक फ़ाइल की एक प्रति है जो एन्क्रिप्टेड zip में संक्षिप्त है, तो आप "प्लेनटेक्स्ट हमला" कर सकते हैं और zip को क्रैक कर सकते हैं, जैसा कि [यहाँ विस्तार से बताया गया है](https://www.hackthis.co.uk/articles/known-plaintext-attack-cracking-zip-files), और [इस पेपर में](https://www.cs.auckland.ac.nz/\~mike/zipattacks.pdf) स्पष्ट किया गया है। पासवर्ड से सुरक्षित zip फ़ाइलों के लिए नया योजना (AES-256 के साथ, "ZipCrypto" की बजाय) इस कमजोरी को नहीं रखती है।

से: [https://app.gitbook.com/@cpol/s/hacktricks/\~/edit/drafts/-LlM5mCby8ex5pOeV4pJ/forensics/basic-forensics-esp/zips-tricks](https://app.gitbook.com/o/Iwnw24TnSs9D9I2OtTKX/s/-L\_2uGJGU7AVNRcqRvEi/)
