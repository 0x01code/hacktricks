# macOS इंस्टॉलर्स का दुरुपयोग

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह, **The PEASS Family** की खोज करें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PR जमा करके।

</details>

## Pkg मूलभूत जानकारी

एक macOS **इंस्टॉलर पैकेज** (जिसे `.pkg` फ़ाइल के रूप में भी जाना जाता है) एक फ़ाइल प्रारूप है जिसका macOS द्वारा **सॉफ़्टवेयर वितरित** करने के लिए उपयोग किया जाता है। ये फ़ाइलें एक **डिस्ट्रीब्यूशन (वितरण)** हैं जो किसी सॉफ़्टवेयर को स्थापित और सही ढंग से चलाने के लिए सब कुछ जिसकी आवश्यकता होती है, को समेटने वाले एक **डिब्बे** की तरह होती हैं।

पैकेज फ़ाइल स्वयं एक आर्काइव है जो एक **फ़ाइलों और निर्देशिकाओं का वर्गीकरण होल्ड करता है जो लक्ष्य** कंप्यूटर पर स्थापित किए जाएंगे। यह उसमें **स्क्रिप्ट्स** भी शामिल हो सकते हैं जो स्थापना से पहले और बाद में कार्यों को करने के लिए होते हैं, जैसे कि कॉन्फ़िगरेशन फ़ाइलें सेट करना या पुराने संस्करणों को साफ़ करना।

### वर्गीकरण

<figure><img src="../../../.gitbook/assets/Pasted Graphic.png" alt="https://www.youtube.com/watch?v=iASSG0_zobQ"><figcaption></figcaption></figure>

* **डिस्ट्रीब्यूशन (xml)**: कस्टमाइजेशन (शीर्षक, स्वागत पाठ...) और स्क्रिप्ट/स्थापना जांचें
* **पैकेजइन्फो (xml)**: जानकारी, स्थापना आवश्यकताएँ, स्थापना स्थान, स्क्रिप्ट्स के पथ चलाने के लिए
* **सामग्री का बिल (bom)**: स्थापित करने, अपडेट करने या हटाने वाली फ़ाइलों की सूची फ़ाइल अनुमतियों के साथ
* **पेलोड (CPIO आर्काइव gzip संपीड़ित)**: `पैकेजइन्फो` से `स्थापन-स्थान` में स्थापित करने के लिए फ़ाइलें
* **स्क्रिप्ट्स (CPIO आर्काइव gzip संपीड़ित)**: प्री और पोस्ट स्थापना स्क्रिप्ट्स और अधिक संसाधन निष्कर्षित करने के लिए एक अस्थायी निर्देशिका में क्रियान्वयन के लिए।

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
## डीएमजी मूल जानकारी

डीएमजी फ़ाइलें, या एप्पल डिस्क इमेजेस, एप्पल के macOS द्वारा डिस्क इमेजेस के लिए उपयोग की जाने वाली एक फ़ाइल प्रारूप हैं। एक डीएमजी फ़ाइल मूल रूप से एक **माउंट करने योग्य डिस्क इमेज** है (इसमें अपना फ़ाइल सिस्टम होता है) जो आम तौर पर संपीड़ित और कभी-कभी एन्क्रिप्टेड रॉ ब्लॉक डेटा को शामिल करता है। जब आप एक डीएमजी फ़ाइल खोलते हैं, macOS उसे एक भौतिक डिस्क की तरह **माउंट करता है**, जिससे आप इसकी सामग्री तक पहुंच सकते हैं।

### वर्गीकरण

<figure><img src="../../../.gitbook/assets/image (12) (2).png" alt=""><figcaption></figcaption></figure>

एक डीएमजी फ़ाइल की वर्गीकरण विषय के आधार पर भिन्न हो सकती है। हालांकि, एप्लिकेशन डीएमजी के लिए, यह आम तौर पर इस संरचना का पालन करती है:

* शीर्ष स्तर: यह डिस्क इमेज की जड़ है। यह आम तौर पर एप्लिकेशन और संभावित रूप से एप्लिकेशन्स फ़ोल्डर के लिंक को शामिल करता है।
* एप्लिकेशन (.app): यह वास्तविक एप्लिकेशन है। macOS में, एक एप्लिकेशन आम तौर पर एक पैकेज होता है जिसमें एप्लिकेशन का निर्माण करने वाली कई व्यक्तिगत फ़ाइलें और फ़ोल्डर होते हैं।
* एप्लिकेशन्स लिंक: यह macOS में एप्लिकेशन्स फ़ोल्डर का एक शॉर्टकट है। इसका उद्देश्य यह है कि आपको एप्लिकेशन को स्थापित करना आसान हो। आप इस शॉर्टकट पर .app फ़ाइल को खींचकर एप्लिकेशन स्थापित कर सकते हैं।

## प्राइवेस्क माध्यम से पीकेज दुरुपयोग

### सार्वजनिक निर्देशिकाओं से निष्पादन

यदि किसी पूर्व या पोस्ट स्थापना स्क्रिप्ट उदाहरण के लिए **`/var/tmp/Installerutil`** से निष्पादित हो रहा है, और हमलावर उस स्क्रिप्ट को नियंत्रित कर सकता है तो वह जब भी निष्पादित होता है वह विशेषाधिकारों को उन्नत कर सकता है। या एक और समान उदाहरण:

<figure><img src="../../../.gitbook/assets/Pasted Graphic 5.png" alt="https://www.youtube.com/watch?v=iASSG0_zobQ"><figcaption></figcaption></figure>

### AuthorizationExecuteWithPrivileges

यह एक [सार्वजनिक फ़ंक्शन](https://developer.apple.com/documentation/security/1540038-authorizationexecutewithprivileg) है जिसे कई स्थापना और अपडेटर्स रूट के रूप में कुछ को निष्पादित करने के लिए कहेंगे। यह फ़ंक्शन **पथ** को **निष्पादित करने के लिए फ़ाइल** को **पैरामीटर** के रूप में स्वीकार करता है, हालांकि, यदि किसी हमलावर ने इस फ़ाइल को **संशोधित** किया है, तो वह अपनी विशेषाधिकारों को उन्नत करने के लिए इसकी निष्पादन का दुरुपयोग कर सकता है।
```bash
# Breakpoint in the function to check wich file is loaded
(lldb) b AuthorizationExecuteWithPrivileges
# You could also check FS events to find this missconfig
```
### माउंटिंग द्वारा निष्पादन

यदि कोई स्थापक `/tmp/fixedname/bla/bla` में लिखता है, तो `/tmp/fixedname` पर **कोई मालिक नहीं** के साथ एक माउंट बनाना संभव है, ताकि आप स्थापना प्रक्रिया का दुरुपयोग करने के लिए **किसी भी फ़ाइल को संशोधित** कर सकें।

इसका एक उदाहरण है **CVE-2021-26089** जिसने **एक आवधिक स्क्रिप्ट को अधिकारी के रूप में निर्धारित** करने के लिए सफलतापूर्वक **अधिकलेखन** किया। अधिक जानकारी के लिए इस टॉक को देखें: [**OBTS v4.0: "Mount(ain) of Bugs" - Csaba Fitzl**](https://www.youtube.com/watch?v=jSYPazD4VcE)

## मैलवेयर के रूप में pkg

### खाली पेलोड

केवल **प्री और पोस्ट-स्थापना स्क्रिप्ट** के साथ एक **`.pkg`** फ़ाइल उत्पन्न करना संभव है बिना किसी पेलोड के।

### वितरण xml में JS

पैकेज के वितरण xml फ़ाइल में **`<script>`** टैग जोड़ना संभव है और उस कोड को निष्पादित किया जाएगा और यह **`system.run`** का उपयोग करके **कमांड्स निष्पादित** कर सकता है:

<figure><img src="../../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

## संदर्भ

* [**DEF CON 27 - Unpacking Pkgs A Look Inside Macos Installer Packages And Common Security Flaws**](https://www.youtube.com/watch?v=iASSG0\_zobQ)
* [**OBTS v4.0: "The Wild World of macOS Installers" - Tony Lambert**](https://www.youtube.com/watch?v=Eow5uNHtmIg)
