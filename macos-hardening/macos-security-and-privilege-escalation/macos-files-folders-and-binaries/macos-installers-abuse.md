# macOS Installers दुरुपयोग

<details>

<summary><strong>जीरो से हीरो तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में** देखना चाहते हैं या **HackTricks को PDF में डाउनलोड** करना चाहते हैं तो [**सब्सक्रिप्शन प्लान**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें और PRs सबमिट करें** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

## Pkg मूलभूत जानकारी

एक macOS **इंस्टॉलर पैकेज** (जिसे `.pkg` फ़ाइल के रूप में भी जाना जाता है) एक फ़ाइल प्रारूप है जिसका macOS द्वारा **सॉफ़्टवेयर वितरित** करने के लिए उपयोग किया जाता है। ये फ़ाइलें एक **डिस्ट्रीब्यूशन (xml)**: कस्टमाइज़ेशन (शीर्षक, स्वागत पाठ...) और स्क्रिप्ट/स्थापना जांचें, **पैकेज फ़ाइल स्वयं एक आर्काइव है जो एक विशिष्ट सॉफ़्टवेयर को स्थापित और सही रूप से चलाने के लिए सब कुछ जिसकी आवश्यकता है वह समेटती है।

पैकेज फ़ाइल स्वयं एक **फ़ाइलों और निर्देशिकाओं का एक वर्गीकरण है जो लक्ष्य वाले** कंप्यूटर पर स्थापित किए जाएंगे। यह उन्हें सेटअप कॉन्फ़िगरेशन फ़ाइलें सेट करने या पुराने संस्करणों को साफ़ करने जैसे कार्यों को करने के लिए **स्क्रिप्ट** भी शामिल कर सकता है।

### वर्गीकरण

<figure><img src="../../../.gitbook/assets/Pasted Graphic.png" alt="https://www.youtube.com/watch?v=iASSG0_zobQ"><figcaption></figcaption></figure>

* **डिस्ट्रीब्यूशन (xml)**: कस्टमाइज़ेशन (शीर्षक, स्वागत पाठ...) और स्क्रिप्ट/स्थापना जांचें
* **पैकेज जानकारी (xml)**: जानकारी, स्थापना आवश्यकताएँ, स्थापना स्थान, स्क्रिप्ट के पथ चलाने के लिए
* **सामग्री का बिल (bom)**: फ़ाइलों की सूची जो स्थापित, अपडेट या हटाई जाएगी और फ़ाइल अनुमतियाँ
* **पेलोड (CPIO आर्काइव gzip संपीड़ित)**: `install-location` में स्थापित करने के लिए फ़ाइलें
* **स्क्रिप्ट (CPIO आर्काइव gzip संपीड़ित)**: प्री और पोस्ट स्थापना स्क्रिप्ट और अधिक संसाधन निष्कर्षित करने के लिए एक अस्थायी निर्देशिका में क्रियान्वयन।

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

DMG फ़ाइलें, या एप्पल डिस्क इमेजेस, एप्पल के macOS द्वारा उपयोग किया जाने वाला एक फ़ाइल प्रारूप है। एक DMG फ़ाइल मूल रूप से एक **माउंट करने योग्य डिस्क इमेज** है (यह अपनी फ़ाइल सिस्टम शामिल करता है) जो सामान्यत: संपीड़ित और कभी-कभी एन्क्रिप्टेड रॉ ब्लॉक डेटा को शामिल करता है। जब आप एक DMG फ़ाइल खोलते हैं, macOS उसे एक भौतिक डिस्क की तरह **माउंट करता है**, जिससे आप इसकी सामग्री तक पहुँच सकते हैं।

{% hint style="danger" %}
ध्यान दें कि **`.dmg`** इंस्टॉलर समर्थित **इतने सारे प्रारूप** हैं कि इससे पहले कुछ में सुरक्षा गड़बड़ी होने के कारण **कर्नेल कोड निष्पादन** प्राप्त करने के लिए **दुरुपयोग** किया गया था।
{% endhint %}

### वर्गीकरण

<figure><img src="../../../.gitbook/assets/image (222).png" alt=""><figcaption></figcaption></figure>

DMG फ़ाइल का वर्गीकरण सामग्री के आधार पर भिन्न हो सकता है। हालांकि, एप्लिकेशन DMGs के लिए, यह आमतौर पर इस संरचना का पालन करता है:

* शीर्ष स्तर: यह डिस्क इमेज की जड़ है। यह आमतौर पर एप्लिकेशन और संभावित रूप से एप्लिकेशन्स फ़ोल्डर के लिंक को शामिल करता है।
* एप्लिकेशन (.app): यह वास्तविक एप्लिकेशन है। macOS में, एक एप्लिकेशन आमतौर पर एक पैकेज होता है जिसमें एप्लिकेशन का निर्माण करने वाली कई व्यक्तिगत फ़ाइलें और फ़ोल्डर होते हैं।
* एप्लिकेशन्स लिंक: यह macOS में एप्लिकेशन इंस्टॉल करने को आसान बनाने के लिए एप्लिकेशन्स फ़ोल्डर का एक शॉर्टकट है। इसका उद्देश्य यह है कि आप एप्लिकेशन इंस्टॉल करने के लिए .app फ़ाइल को इस शॉर्टकट पर खींच सकते हैं।

## pkg दुरुपयोग के माध्यम से विशेषाधिकार

### सार्वजनिक निर्देशिकाओं से निष्पादन

यदि कोई पूर्व या पोस्ट स्थापना स्क्रिप्ट उदाहरण के लिए **`/var/tmp/Installerutil`** से निष्पादित हो रहा है, और हमलावर उस स्क्रिप्ट को नियंत्रित कर सकता है तो वह जब भी निष्पादित होता है वह विशेषाधिकार बढ़ा सकता है। या एक और समान उदाहरण:

<figure><img src="../../../.gitbook/assets/Pasted Graphic 5.png" alt="https://www.youtube.com/watch?v=iASSG0_zobQ"><figcaption><p><a href="https://www.youtube.com/watch?v=kCXhIYtODBg">https://www.youtube.com/watch?v=kCXhIYtODBg</a></p></figcaption></figure>

### AuthorizationExecuteWithPrivileges

यह एक [सार्वजनिक फ़ंक्शन](https://developer.apple.com/documentation/security/1540038-authorizationexecutewithprivileg) है जिसे कई इंस्टॉलर और अपडेटर्स **रूट के रूप में कुछ निष्पादित** करने के लिए कहेंगे। यह फ़ंक्शन **फ़ाइल** का **पथ** स्वीकार करता है जिसे **निष्पादित** करने के लिए पैरामीटर के रूप में, हालांकि, यदि कोई हमलावर इस फ़ाइल को **संशोधित** कर सकता है, तो वह अपने विशेषाधिकारों को बढ़ाने के लिए इसका दुरुपयोग कर सकता है।
```bash
# Breakpoint in the function to check wich file is loaded
(lldb) b AuthorizationExecuteWithPrivileges
# You could also check FS events to find this missconfig
```
### माउंटिंग द्वारा निष्पादन

यदि कोई स्थापक `/tmp/fixedname/bla/bla` में लिखता है, तो `/tmp/fixedname` पर **कोई मालिक नहीं** के साथ एक माउंट बनाना संभव है, जिससे आप **स्थापना प्रक्रिया का दुरुपयोग करने के लिए किसी भी फ़ाइल को संशोधित** कर सकते हैं।

इसका एक उदाहरण है **CVE-2021-26089** जिसने **एक आवधिक स्क्रिप्ट को अधिकारी के रूप में निष्पादित** करने के लिए सफलतापूर्वक **अधिकलेखन** किया। अधिक जानकारी के लिए इस टॉक को देखें: [**OBTS v4.0: "Mount(ain) of Bugs" - Csaba Fitzl**](https://www.youtube.com/watch?v=jSYPazD4VcE)

## मैलवेयर के रूप में pkg

### खाली पेलोड

केवल **पूर्व और पोस्ट स्थापना स्क्रिप्ट** के साथ एक **`.pkg`** फ़ाइल उत्पन्न करना संभव है बिना किसी पेलोड के।

### वितरण xml में JS

पैकेज के वितरण xml फ़ाइल में **`<script>`** टैग जोड़ना संभव है और उस कोड को निष्पादित किया जाएगा और यह **`system.run`** का उपयोग करके **कमांड्स निष्पादित** कर सकता है:

<figure><img src="../../../.gitbook/assets/image (1040).png" alt=""><figcaption></figcaption></figure>

## संदर्भ

* [**DEF CON 27 - Unpacking Pkgs A Look Inside Macos Installer Packages And Common Security Flaws**](https://www.youtube.com/watch?v=iASSG0\_zobQ)
* [**OBTS v4.0: "The Wild World of macOS Installers" - Tony Lambert**](https://www.youtube.com/watch?v=Eow5uNHtmIg)
* [**DEF CON 27 - Unpacking Pkgs A Look Inside MacOS Installer Packages**](https://www.youtube.com/watch?v=kCXhIYtODBg)
