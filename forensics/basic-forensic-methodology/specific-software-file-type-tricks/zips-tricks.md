# ZIPs ट्रिक्स

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **अपनी हैकिंग ट्रिक्स साझा करें PRs जमा करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में.

</details>

ZIP फाइलों के लिए कुछ कमांड-लाइन टूल्स हैं जो जानना उपयोगी होगा।

* `unzip` अक्सर यह जानकारी देता है कि क्यों एक ZIP डिकंप्रेस नहीं हो रहा है।
* `zipdetails -v` फॉर्मेट के विभिन्न फील्ड्स में मौजूद मानों पर गहराई से जानकारी प्रदान करता है।
* `zipinfo` ZIP फाइल की सामग्री के बारे में जानकारी दिखाता है, बिना इसे निकाले।
* `zip -F input.zip --out output.zip` और `zip -FF input.zip --out output.zip` एक भ्रष्ट ZIP फाइल की मरम्मत करने का प्रयास करते हैं।
* [fcrackzip](https://github.com/hyc/fcrackzip) ZIP पासवर्ड का ब्रूट-फोर्स अनुमान लगाता है (7 अक्षरों से कम के पासवर्ड्स के लिए)।

[Zip फाइल फॉर्मेट स्पेसिफिकेशन](https://pkware.cachefly.net/webdocs/casestudies/APPNOTE.TXT)

पासवर्ड-प्रोटेक्टेड ZIP फाइलों के बारे में एक महत्वपूर्ण सुरक्षा-संबंधित नोट यह है कि वे फाइलनेम और मूल फाइल साइज़ को एन्क्रिप्ट नहीं करते हैं, जो कि वे संपीड़ित फाइलें रखते हैं, जैसे कि पासवर्ड-प्रोटेक्टेड RAR या 7z फाइलें।

ZIP क्रैकिंग के बारे में एक और नोट यह है कि यदि आपके पास एन्क्रिप्टेड ZIP में संपीड़ित किसी भी एक फाइल की एक अनएन्क्रिप्टेड/अनकंप्रेस्ड प्रति है, तो आप "plaintext attack" कर सकते हैं और ZIP को क्रैक कर सकते हैं, जैसा कि [यहाँ विस्तार से बताया गया है](https://www.hackthis.co.uk/articles/known-plaintext-attack-cracking-zip-files), और [इस पेपर](https://www.cs.auckland.ac.nz/\~mike/zipattacks.pdf) में समझाया गया है। ZIP फाइलों को पासवर्ड-प्रोटेक्ट करने की नई योजना (AES-256 के साथ, "ZipCrypto" के बजाय) में यह कमजोरी नहीं है।

से: [https://app.gitbook.com/@cpol/s/hacktricks/\~/edit/drafts/-LlM5mCby8ex5pOeV4pJ/forensics/basic-forensics-esp/zips-tricks](https://app.gitbook.com/o/Iwnw24TnSs9D9I2OtTKX/s/-L\_2uGJGU7AVNRcqRvEi/)

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **अपनी हैकिंग ट्रिक्स साझा करें PRs जमा करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में.

</details>
