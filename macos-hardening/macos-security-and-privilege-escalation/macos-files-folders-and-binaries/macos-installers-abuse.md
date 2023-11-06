# macOS इंस्टॉलर दुरुपयोग

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT संग्रह**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल हों** या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का पालन करें**.
* **अपने हैकिंग ट्रिक्स को** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में पीआर जमा करके** अपना योगदान दें।

</details>

## Pkg मूलभूत जानकारी

macOS **इंस्टॉलर पैकेज** (जिसे `.pkg` फ़ाइल के रूप में भी जाना जाता है) एक फ़ाइल प्रारूप है जिसका उपयोग macOS द्वारा **सॉफ़्टवेयर वितरित करने** के लिए किया जाता है। ये फ़ाइलें एक **बॉक्स की तरह होती हैं जो सॉफ़्टवेयर को स्थापित और सही ढंग से चलाने के लिए सब कुछ शामिल करती हैं**।

पैकेज फ़ाइल स्वयं एक आर्काइव है जो टारग्जिट कंप्यूटर पर स्थापित होने वाले फ़ाइलों और निर्देशिकाओं की **एक प्रारंभिक** होती है। इसमें स्क्रिप्ट भी शामिल हो सकते हैं जो स्थापना से पहले और बाद में कार्यों को करने के लिए होते हैं, जैसे कि कॉन्फ़िगरेशन फ़ाइलें सेट करना या सॉफ़्टवेयर के पुराने संस्करणों को साफ़ करना।

### व्यवस्था

<figure><img src="../../../.gitbook/assets/Pasted Graphic.png" alt=""><figcaption></figcaption></figure>

* **वितरण (xml)**: अनुकूलन (शीर्षक, स्वागत पाठ...) और स्क्रिप्ट/स्थापना जांचें
* **PackageInfo (xml)**: जानकारी, स्थापना आवश्यकताएं, स्थापना स्थान, स्क्रिप्ट चलाने के लिए पथ
* **वस्त्रागार (bom)**: फ़ाइलों की सूची जो स्थापित, अद्यतन या हटाए जाएंगी और फ़ाइल अनुमतियों के साथ
* **पेलोड (CPIO आर्काइव gzip संपीड़ित)**: PackageInfo से `install-location` में स्थापित करने के लिए फ़ाइलें
* **स्क्रिप्ट (CPIO आर्काइव gzip संपीड़ित)**: प्री और पोस्ट स्थापना स्क्रिप्ट और अधिक संसाधन जो कार्यान्वयन के लिए एक अस्थायी निर्देशिका में निकाले जाते हैं।

### डीकंप्रेस
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
## DMG मूलभूत जानकारी

DMG फ़ाइलें, यानी Apple Disk Images, Apple के macOS द्वारा डिस्क इमेज के लिए उपयोग होने वाली फ़ाइल प्रारूप हैं। एक DMG फ़ाइल मूल रूप से एक **माउंट करने योग्य डिस्क इमेज** है (इसमें अपनी फ़ाइल सिस्टम होती है) जो आमतौर पर संपीड़ित और कभी-कभी एन्क्रिप्टेड रॉ ब्लॉक डेटा को संग्रहित करती है। जब आप एक DMG फ़ाइल खोलते हैं, macOS इसे एक भौतिक डिस्क की तरह माउंट करता है, जिससे आप इसकी सामग्री तक पहुंच सकते हैं।

### व्यवस्था

<figure><img src="../../../.gitbook/assets/image (12) (2).png" alt=""><figcaption></figcaption></figure>

DMG फ़ाइल की व्यवस्था सामग्री के आधार पर अलग हो सकती है। हालांकि, आवेदन DMG के लिए, यह आमतौर पर इस संरचना का पालन करता है:

* शीर्ष स्तर: यह डिस्क इमेज की जड़ है। यह आमतौर पर आवेदन और संभवतः एप्लिकेशन फ़ोल्डर के लिए एक लिंक समेत होता है।
* आवेदन (.app): यह वास्तविक आवेदन है। macOS में, एक आवेदन आमतौर पर एक पैकेज होता है जिसमें आवेदन का गठन करने वाले कई व्यक्तिगत फ़ाइलें और फ़ोल्डर होते हैं।
* आवेदन लिंक: यह macOS में एप्लिकेशन फ़ोल्डर के लिए एक शॉर्टकट है। इसका उद्देश्य यह है कि आप आवेदन को स्थापित करने के लिए इस शॉर्टकट पर .app फ़ाइल को खींच सकें।

## pkg द्वारा Privesc द्वारा दुरुपयोग

### सार्वजनिक निर्देशिकाओं से निष्पादन

यदि पूर्व या पद स्थापना स्क्रिप्ट उदाहरण के लिए **`/var/tmp/Installerutil`** से निष्पादित हो रहा है, और हमलावर उस स्क्रिप्ट को नियंत्रित कर सकता है, तो वह अपनी विशेषाधिकार को बढ़ा सकता है जब भी यह निष्पादित होता है। या एक और समान उदाहरण:

<figure><img src="../../../.gitbook/assets/Pasted Graphic 5.png" alt=""><figcaption></figcaption></figure>

### AuthorizationExecuteWithPrivileges

यह एक [सार्वजनिक फ़ंक्शन](https://developer.apple.com/documentation/security/1540038-authorizationexecutewithprivileg) है जिसे कई स्थापना और अद्यतनकर्ता बुलाएंगे **रूट के रूप में कुछ निष्पादित करने** के लिए। इस फ़ंक्शन को **फ़ाइल** के **निष्पादन** के लिए **पथ** के रूप में स्वीकार करता है, हालांकि, यदि हमलावर इस फ़ाइल को **संशोधित** कर सकता है, तो वह अपनी विशेषाधिकार को बढ़ा सकेगा और विशेषाधिकार को बढ़ा सकेगा।
```bash
# Breakpoint in the function to check wich file is loaded
(lldb) b AuthorizationExecuteWithPrivileges
# You could also check FS events to find this missconfig
```
अधिक जानकारी के लिए इस टॉक की जांच करें: [https://www.youtube.com/watch?v=lTOItyjTTkw](https://www.youtube.com/watch?v=lTOItyjTTkw)

### माउंट करके निष्पादन

यदि एक स्थापक `/tmp/fixedname/bla/bla` में लिखता है, तो यह संभव है कि आप `/tmp/fixedname` पर **एक माउंट बना सकते हैं** जिसमें कोई मालिक नहीं होता है, इसलिए आप स्थापना प्रक्रिया का दुरुपयोग करने के लिए किसी भी फ़ाइल को संशोधित कर सकते हैं।

इसका एक उदाहरण है **CVE-2021-26089** जिसने **एक नियमित स्क्रिप्ट को अधिलेखित** करके रूट के रूप में निष्पादन प्राप्त किया। अधिक जानकारी के लिए टॉक की जांच करें: [**OBTS v4.0: "Mount(ain) of Bugs" - Csaba Fitzl**](https://www.youtube.com/watch?v=jSYPazD4VcE)

## मैलवेयर के रूप में pkg

### खाली पेलोड

केवल **पूर्व और पश्चात स्थापना स्क्रिप्ट** के साथ एक **`.pkg`** फ़ाइल उत्पन्न करना संभव है बिना किसी पेलोड के।

### वितरण xml में JS

पैकेज के वितरण xml फ़ाइल में **`<script>`** टैग जोड़ना संभव है और उस कोड को निष्पादित किया जाएगा और इसका उपयोग करके यह **`system.run`** का उपयोग करके आदेश निष्पादित कर सकता है:

<figure><img src="../../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

## संदर्भ

* [**DEF CON 27 - Unpacking Pkgs A Look Inside Macos Installer Packages And Common Security Flaws**](https://www.youtube.com/watch?v=iASSG0\_zobQ)
* [**OBTS v4.0: "The Wild World of macOS Installers" - Tony Lambert**](https://www.youtube.com/watch?v=Eow5uNHtmIg)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आप **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड** करने का उपयोग करना चाहते हैं? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह
* प्राप्त करें [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को।**

</details>
