# macOS इंस्टालर्स का दुरुपयोग

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) को **फॉलो करें**.
* **HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स शेयर करें.

</details>

## Pkg बेसिक जानकारी

macOS **इंस्टालर पैकेज** (जिसे `.pkg` फाइल भी कहा जाता है) एक फाइल प्रारूप है जिसका उपयोग macOS द्वारा **सॉफ्टवेयर वितरित करने** के लिए किया जाता है। ये फाइलें एक **बॉक्स की तरह होती हैं जिसमें सॉफ्टवेयर को सही ढंग से इंस्टॉल और चलाने के लिए जरूरी सब कुछ होता है**.

पैकेज फाइल खुद एक आर्काइव होती है जिसमें **फाइलों और डायरेक्टरीज की एक हायरार्की होती है जो लक्ष्य कंप्यूटर पर इंस्टॉल की जाएगी**. इसमें **स्क्रिप्ट्स** भी शामिल हो सकती हैं जो इंस्टालेशन से पहले और बाद में कार्य करने के लिए होती हैं, जैसे कॉन्फ़िगरेशन फाइलों को सेटअप करना या सॉफ्टवेयर के पुराने संस्करणों को साफ करना.

### हायरार्की

<figure><img src="../../../.gitbook/assets/Pasted Graphic.png" alt=""><figcaption></figcaption></figure>

* **डिस्ट्रीब्यूशन (xml)**: कस्टमाइजेशन (शीर्षक, स्वागत पाठ…) और स्क्रिप्ट/इंस्टालेशन चेक्स
* **PackageInfo (xml)**: जानकारी, इंस्टाल आवश्यकताएं, इंस्टाल स्थान, स्क्रिप्ट्स के पथ जो चलाए जाएंगे
* **बिल ऑफ मटेरियल्स (bom)**: इंस्टॉल, अपडेट या हटाने के लिए फाइलों की सूची फाइल अनुमतियों के साथ
* **पेलोड (CPIO आर्काइव gzip संपीड़ित)**: PackageInfo से `install-location` में इंस्टॉल करने के लिए फाइलें
* **स्क्रिप्ट्स (CPIO आर्काइव gzip संपीड़ित)**: प्री और पोस्ट इंस्टॉल स्क्रिप्ट्स और अधिक संसाधन जो निष्पादन के लिए एक अस्थायी निर्देशिका में निकाले जाते हैं.

### डिकम्प्रेस
```bash
# Tool to directly get the files inside a package
pkgutil —expand "/path/to/package.pkg" "/path/to/out/dir"

# Get the files ina. more manual way
mkdir -p "/path/to/out/dir"
cd "/path/to/out/dir"
xar -xf "/path/to/package.pkg"

# Decompress also the CPIO gzip compressed ones
cat Scripts | gzip -dc | cpio -i
cpio -i < Scripts
```
## DMG मूल जानकारी

DMG फाइलें, या Apple डिस्क इमेजेज, Apple के macOS के लिए डिस्क इमेजेज का एक फाइल प्रारूप हैं। एक DMG फाइल मूल रूप से एक **माउंटेबल डिस्क इमेज** होती है (इसमें अपनी फाइलसिस्टम होती है) जिसमें आमतौर पर संकुचित और कभी-कभी एन्क्रिप्टेड ब्लॉक डेटा होता है। जब आप एक DMG फाइल खोलते हैं, macOS इसे ऐसे माउंट करता है जैसे यह एक भौतिक डिस्क हो, जिससे आप इसकी सामग्री तक पहुँच सकते हैं।

### पदानुक्रम

<figure><img src="../../../.gitbook/assets/image (12) (2).png" alt=""><figcaption></figcaption></figure>

एक DMG फाइल का पदानुक्रम सामग्री के आधार पर भिन्न हो सकता है। हालांकि, एप्लिकेशन DMG के लिए, यह आमतौर पर इस संरचना का अनुसरण करता है:

* शीर्ष स्तर: यह डिस्क इमेज की जड़ है। इसमें अक्सर एप्लिकेशन और संभवतः एप्लिकेशन्स फोल्डर का लिंक होता है।
* एप्लिकेशन (.app): यह वास्तविक एप्लिकेशन है। macOS में, एक एप्लिकेशन आमतौर पर एक पैकेज होता है जिसमें कई व्यक्तिगत फाइलें और फोल्डर्स होते हैं जो एप्लिकेशन को बनाते हैं।
* एप्लिकेशन्स लिंक: यह macOS में एप्लिकेशन्स फोल्डर का शॉर्टकट है। इसका उद्देश्य आपके लिए एप्लिकेशन को इंस्टॉल करना आसान बनाना है। आप इस शॉर्टकट पर .app फाइल को खींचकर एप्लिकेशन को इंस्टॉल कर सकते हैं।

## Privesc via pkg abuse

### सार्वजनिक निर्देशिकाओं से निष्पादन

यदि एक प्री या पोस्ट इंस्टालेशन स्क्रिप्ट उदाहरण के लिए **`/var/tmp/Installerutil`** से निष्पादित हो रही है, और हमलावर उस स्क्रिप्ट को नियंत्रित कर सकता है तो वह जब भी यह निष्पादित होती है तो वह विशेषाधिकार बढ़ा सकता है। या एक अन्य समान उदाहरण:

<figure><img src="../../../.gitbook/assets/Pasted Graphic 5.png" alt=""><figcaption></figcaption></figure>

### AuthorizationExecuteWithPrivileges

यह एक [सार्वजनिक फंक्शन](https://developer.apple.com/documentation/security/1540038-authorizationexecutewithprivileg) है जिसे कई इंस्टालर्स और अपडेटर्स रूट के रूप में कुछ **निष्पादित करने के लिए कॉल करेंगे**। यह फंक्शन **पथ** को **फाइल** के **निष्पादन** के लिए पैरामीटर के रूप में स्वीकार करता है, हालांकि, यदि एक हमलावर इस फाइल को **संशोधित** कर सकता है, तो वह रूट के साथ इसके निष्पादन का **दुरुपयोग** करके विशेषाधिकार बढ़ाने में सक्षम होगा।
```bash
# Breakpoint in the function to check wich file is loaded
(lldb) b AuthorizationExecuteWithPrivileges
# You could also check FS events to find this missconfig
```
अधिक जानकारी के लिए इस वार्ता को देखें: [https://www.youtube.com/watch?v=lTOItyjTTkw](https://www.youtube.com/watch?v=lTOItyjTTkw)

### माउंटिंग द्वारा निष्पादन

यदि एक इंस्टॉलर `/tmp/fixedname/bla/bla` पर लिखता है, तो `/tmp/fixedname` पर noowners के साथ एक **माउंट बनाना** संभव है ताकि आप **स्थापना के दौरान किसी भी फ़ाइल को संशोधित कर सकें** और स्थापना प्रक्रिया का दुरुपयोग कर सकें।

इसका एक उदाहरण **CVE-2021-26089** है जिसने रूट के रूप में निष्पादन प्राप्त करने के लिए एक **पीरियोडिक स्क्रिप्ट को ओवरराइट किया**। अधिक जानकारी के लिए वार्ता देखें: [**OBTS v4.0: "Mount(ain) of Bugs" - Csaba Fitzl**](https://www.youtube.com/watch?v=jSYPazD4VcE)

## pkg के रूप में मैलवेयर

### खाली पेलोड

बिना किसी पेलोड के **`.pkg`** फ़ाइल को केवल **प्री और पोस्ट-इंस्टॉल स्क्रिप्ट्स** के साथ जनरेट करना संभव है।

### डिस्ट्रीब्यूशन xml में JS

पैकेज की **डिस्ट्रीब्यूशन xml** फ़ाइल में **`<script>`** टैग्स जोड़ना संभव है और वह कोड निष्पादित होगा और यह **`system.run`** का उपयोग करके **कमांड्स निष्पादित कर सकता है**:

<figure><img src="../../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

## संदर्भ

* [**DEF CON 27 - Unpacking Pkgs A Look Inside Macos Installer Packages And Common Security Flaws**](https://www.youtube.com/watch?v=iASSG0_zobQ)
* [**OBTS v4.0: "The Wild World of macOS Installers" - Tony Lambert**](https://www.youtube.com/watch?v=Eow5uNHtmIg)

<details>

<summary><strong>Learn AWS hacking from zero to hero with</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग प्राप्त करें**](https://peass.creator-spring.com)
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) **का पालन करें**।
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>
