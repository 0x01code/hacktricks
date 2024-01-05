# macOS Office Sandbox Bypasses

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>

### Word Sandbox को Launch Agents के माध्यम से बायपास करना

एप्लिकेशन एक **कस्टम Sandbox** का उपयोग करता है जिसमें **`com.apple.security.temporary-exception.sbpl`** एंटाइटलमेंट होता है और यह कस्टम सैंडबॉक्स फाइलों को कहीं भी लिखने की अनुमति देता है बशर्ते फाइलनाम `~$` से शुरू हो: `(require-any (require-all (vnode-type REGULAR-FILE) (regex #"(^|/)~$[^/]+$")))`

इसलिए, बायपास करना उतना ही आसान था जितना कि **`plist` LaunchAgent** को `~/Library/LaunchAgents/~$escape.plist` में **लिखना**।

[**मूल रिपोर्ट यहाँ देखें**](https://www.mdsec.co.uk/2018/08/escaping-the-sandbox-microsoft-office-on-macos/).

### Word Sandbox को Login Items और zip के माध्यम से बायपास करना

याद रखें कि पहले एस्केप से, Word ऐसी मनमानी फाइलें लिख सकता है जिनका नाम `~$` से शुरू होता है हालांकि पिछले वल्न के पैच के बाद `/Library/Application Scripts` या `/Library/LaunchAgents` में लिखना संभव नहीं था।

यह पता चला कि सैंडबॉक्स के भीतर से **Login Item** बनाना संभव है (ऐप्स जो उपयोगकर्ता के लॉग इन करने पर निष्पादित होंगे)। हालांकि, ये ऐप्स **निष्पादित नहीं होंगे जब तक** वे **notarized नहीं होते** और **args जोड़ना संभव नहीं है** (इसलिए आप सिर्फ **`bash`** का उपयोग करके रिवर्स शेल नहीं चला सकते)।

पिछले Sandbox बायपास से, Microsoft ने `~/Library/LaunchAgents` में फाइलें लिखने का विकल्प अक्षम कर दिया था। हालांकि, यह पता चला कि यदि आप **Login Item के रूप में एक zip फाइल** डालते हैं तो `Archive Utility` बस उसे उसके वर्तमान स्थान पर **unzip** कर देगा। इसलिए, चूंकि डिफ़ॉल्ट रूप से `~/Library` से `LaunchAgents` फोल्डर नहीं बनाया गया है, इसलिए **`LaunchAgents/~$escape.plist` में एक plist को zip करना** और **`~/Library` में zip फाइल को रखना** संभव था ताकि जब डिकंप्रेस किया जाए तो यह पर्सिस्टेंस डेस्टिनेशन तक पहुँच जाए।

[**मूल रिपोर्ट यहाँ देखें**](https://objective-see.org/blog/blog\_0x4B.html).

### Word Sandbox को Login Items और .zshenv के माध्यम से बायपास करना

(याद रखें कि पहले एस्केप से, Word मनमानी फाइलें लिख सकता है जिनका नाम `~$` से शुरू होता है)।

हालांकि, पिछली तकनीक में एक सीमा थी, अगर फोल्डर **`~/Library/LaunchAgents`** मौजूद है क्योंकि किसी अन्य सॉफ्टवेयर ने इसे बनाया है, तो यह विफल हो जाएगा। इसके लिए एक अलग Login Items चेन की खोज की गई थी।

एक हमलावर **`.bash_profile`** और **`.zshenv`** फाइलें बना सकता है जिसमें पेलोड निष्पादित करने के लिए होता है और फिर उन्हें ज़िप करके और **पीड़ित के उपयोगकर्ता फोल्डर में ज़िप लिख सकता है**: **`~/~$escape.zip`**।

फिर, **Login Items** में ज़िप फाइल जोड़ें और फिर **`Terminal`** ऐप। जब उपयोगकर्ता फिर से लॉगिन करता है, ज़िप फाइल को उपयोगकर्ता की फाइल में अनज़िप किया जाएगा, **`.bash_profile`** और **`.zshenv`** को ओवरराइट कर देगा और इसलिए, टर्मिनल इनमें से एक फाइल को निष्पादित करेगा (यह निर्भर करता है कि bash या zsh का उपयोग किया जा रहा है)।

[**मूल रिपोर्ट यहाँ देखें**](https://desi-jarvis.medium.com/office365-macos-sandbox-escape-fcce4fa4123c).

### Word Sandbox Bypass with Open and env variables

Sandboxed प्रक्रियाओं से अभी भी अन्य प्रक्रियाओं को **`open`** उपयोगिता का उपयोग करके आमंत्रित करना संभव है। इसके अलावा, ये प्रक्रियाएं **अपने स्वयं के सैंडबॉक्स के भीतर चलेंगी**।

यह पता चला कि open उपयोगिता में **`--env`** विकल्प होता है जो एक ऐप को **विशिष्ट env** वेरिएबल्स के साथ चलाने के लिए होता है। इसलिए, यह संभव था कि **`.zshenv` फाइल** को सैंडबॉक्स के **अंदर** एक फोल्डर में बनाया जाए और फिर `open` का उपयोग करके `--env` के साथ **`HOME` वेरिएबल** को उस फोल्डर के लिए सेट करें जिससे `Terminal` ऐप खुलेगा, जो `.zshenv` फाइल को निष्पादित करेगा (किसी कारण से वेरिएबल `__OSINSTALL_ENVIROMENT` को भी सेट करना आवश्यक था)।

[**मूल रिपोर्ट यहाँ देखें**](https://perception-point.io/blog/technical-analysis-of-cve-2021-30864/).

### Word Sandbox Bypass with Open and stdin

**`open`** उपयोगिता ने **`--stdin`** पैरामीटर का भी समर्थन किया (और पिछले बायपास के बाद `--env` का उपयोग करना संभव नहीं था)।

बात यह है कि भले ही **`python`** को Apple द्वारा साइन किया गया था, यह **निष्पादित नहीं होगा** एक स्क्रिप्ट को जिसमें **`quarantine`** एट्रिब्यूट हो। हालांकि, यह संभव था कि उसे stdin से एक स्क्रिप्ट पास की जाए ताकि यह जांच न करे कि यह क्वारंटीन है या नहीं:&#x20;

1. एक **`~$exploit.py`** फाइल ड्रॉप करें जिसमें मनमाने Python कमांड हों।
2. _open_ चलाएं **`–stdin='~$exploit.py' -a Python`**, जो Python ऐप को हमारी ड्रॉप की गई फाइल के साथ चलाएगा जो इसके स्टैंडर्ड इनपुट के रूप में काम करेगी। Python हमारे कोड को खुशी-खुशी चलाता है, और चूंकि यह _launchd_ की चाइल्ड प्रोसेस है, यह Word के सैंडबॉक्स नियमों से बंधा नहीं है।

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) द
