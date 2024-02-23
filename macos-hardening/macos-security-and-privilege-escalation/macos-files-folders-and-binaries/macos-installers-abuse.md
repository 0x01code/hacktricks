# macOS Installers दुरुपयोग

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** द्वारा **पीआर जमा करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

## Pkg मूलभूत जानकारी

एक macOS **इंस्टॉलर पैकेज** (जिसे `.pkg` फ़ाइल के रूप में भी जाना जाता है) एक फ़ाइल प्रारूप है जिसका macOS द्वारा **सॉफ़्टवेयर वितरित** करने के लिए उपयोग किया जाता है। ये फ़ाइलें एक **डिस्ट्रीब्यूशन बॉक्स की तरह होती हैं जो किसी सॉफ़्टवेयर को** सही ढंग से स्थापित और चलाने के लिए आवश्यक सभी चीज़ों को शामिल करती है।

पैकेज फ़ाइल स्वयं एक आर्काइव है जो एक **विजेता** कंप्यूटर पर स्थापित किए जाने वाले **फ़ाइलों और निर्देशिकाओं का वर्गीकरण** रखता है। यह उपकरणों की पुरानी संस्करणों को साफ़ करने या विन्यास फ़ाइलें सेट करने जैसे कार्यों को करने के लिए **स्क्रिप्ट** भी शामिल कर सकता है।

### वर्गीकरण

<figure><img src="../../../.gitbook/assets/Pasted Graphic.png" alt="https://www.youtube.com/watch?v=iASSG0_zobQ"><figcaption></figcaption></figure>

* **डिस्ट्रीब्यूशन (xml)**: अनुकूलन (शीर्षक, स्वागत पाठ...) और स्क्रिप्ट/स्थापना जांचें
* **पैकेजइन्फो (xml)**: जानकारी, स्थापना आवश्यकताएँ, स्थापना स्थान, स्क्रिप्ट के पथ चलाने के लिए
* **सामग्री का बिल (bom)**: फ़ाइलों की सूची जो स्थापित, अपडेट या हटाई जाएगी, फ़ाइल अनुमतियाँ के साथ
* **पेलोड (CPIO आर्काइव gzip संपीड़ित)**: `पैकेजइन्फो` से `स्थापन-स्थान` में स्थापित करने के लिए फ़ाइलें
* **स्क्रिप्ट (CPIO आर्काइव gzip संपीड़ित)**: पूर्व और पछतावे स्थापित स्क्रिप्ट और अधिक संसाधन निष्कर्षित करने के लिए एक अस्थायी निर्देशिका में क्रियान्वयन के लिए।

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
इंस्टॉलर की सामग्री को मैन्युअल रूप से डीकंप्रेस किए बिना देखने के लिए आप मुफ्त टूल [**Suspicious Package**](https://mothersruin.com/software/SuspiciousPackage/) का भी उपयोग कर सकते हैं।

## DMG मूलभूत जानकारी

DMG फ़ाइलें, या एप्पल डिस्क इमेजेस, एप्पल के macOS द्वारा डिस्क इमेजेस के लिए उपयोग की जाने वाली एक फ़ाइल प्रारूप हैं। एक DMG फ़ाइल मूल रूप से एक **माउंट करने योग्य डिस्क इमेज** है (यह अपनी अपनी फ़ाइल सिस्टम शामिल करती है) जिसमें सामान्य रूप से संपीड़ित और कभी-कभी एन्क्रिप्टेड रॉ ब्लॉक डेटा होता है। जब आप एक DMG फ़ाइल खोलते हैं, macOS उसे **ऐसे ही एक भौतिक डिस्क के रूप में माउंट** करता है, जिससे आप इसकी सामग्री तक पहुँच सकते हैं।

### वर्गीकरण

<figure><img src="../../../.gitbook/assets/image (12) (2).png" alt=""><figcaption></figcaption></figure>

DMG फ़ाइल की वर्गीकरण सामग्री के आधार पर भिन्न हो सकती है। हालांकि, एप्लिकेशन DMGs के लिए, यह आम तौर पर इस संरचना का पालन करता है:

* शीर्ष स्तर: यह डिस्क इमेज की जड़ है। इसमें आम तौर पर एप्लिकेशन और संभावित रूप से एप्लिकेशन्स फ़ोल्डर के लिए एक लिंक होता है।
* एप्लिकेशन (.app): यह वास्तविक एप्लिकेशन होता है। macOS में, एक एप्लिकेशन आम तौर पर एक पैकेज होता है जिसमें एप्लिकेशन का निर्माण करने वाली कई व्यक्तिगत फ़ाइलें और फ़ोल्डर होते हैं।
* एप्लिकेशन्स लिंक: यह macOS में एप्लिकेशन्स फ़ोल्डर का एक शॉर्टकट होता है। इसका उद्देश्य यह है कि आपको एप्लिकेशन इंस्टॉल करना आसान हो। आप इस शॉर्टकट पर .app फ़ाइल को खींचकर एप्लिकेशन को इंस्टॉल कर सकते हैं।

## pkg द्वारा Privesc दुरुपयोग

### सार्वजनिक निर्देशिकाओं से निष्पादन

यदि कोई पूर्व या पोस्ट स्थापना स्क्रिप्ट उदाहरण के लिए **`/var/tmp/Installerutil`** से निष्पादित हो रहा है, और हमलावर उस स्क्रिप्ट को नियंत्रित कर सकता है तो वह जब भी निष्पादित होता है वह विशेषाधिकारों को उन्नत कर सकता है। या एक और समान उदाहरण:

<figure><img src="../../../.gitbook/assets/Pasted Graphic 5.png" alt="https://www.youtube.com/watch?v=iASSG0_zobQ"><figcaption></figcaption></figure>

### AuthorizationExecuteWithPrivileges

यह एक [सार्वजनिक फ़ंक्शन](https://developer.apple.com/documentation/security/1540038-authorizationexecutewithprivileg) है जिसे कई इंस्टॉलर और अपडेटर्स **रूट के रूप में कुछ निष्पादित करने के लिए** कहेंगे। यह फ़ंक्शन **फ़ाइल** का **पथ** स्वीकार करता है जिसे **निष्पादित** करने के लिए, हालांकि, अगर हमलावर इस फ़ाइल को **संशोधित** कर सकता है, तो वह अपनी विशेषाधिकारों को उन्नत करने के लिए इसका दुरुपयोग कर सकता है।
```bash
# Breakpoint in the function to check wich file is loaded
(lldb) b AuthorizationExecuteWithPrivileges
# You could also check FS events to find this missconfig
```
### क्रियान्वयन द्वारा माउंटिंग

यदि एक स्थापक `/tmp/fixedname/bla/bla` में लिखता है, तो `/tmp/fixedname` पर **कोई मालिक नहीं** के साथ एक माउंट बनाना संभव है, ताकि आप **स्थापना के दौरान किसी भी फ़ाइल को संशोधित** कर सकें और स्थापना प्रक्रिया का दुरुपयोग कर सकें।

इसका एक उदाहरण है **CVE-2021-26089** जिसने **एक आवधिक स्क्रिप्ट को अधिक करने** के लिए व्यवस्थित किया था ताकि वह रूट के रूप में क्रियान्वयन प्राप्त कर सके। अधिक जानकारी के लिए टॉक देखें: [**OBTS v4.0: "Mount(ain) of Bugs" - Csaba Fitzl**](https://www.youtube.com/watch?v=jSYPazD4VcE)

## मैलवेयर के रूप में pkg

### खाली पेलोड

केवल **प्री और पोस्ट-स्थापना स्क्रिप्ट** के साथ एक **`.pkg`** फ़ाइल उत्पन्न करना संभव है बिना किसी पेलोड के।

### वितरण xml में JS

पैकेज के वितरण xml फ़ाइल में **`<script>`** टैग जोड़ना संभव है और उस कोड को क्रियान्वित किया जाएगा और यह **`system.run`** का उपयोग करके **कमांड्स को क्रियान्वित** कर सकता है:

<figure><img src="../../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

## संदर्भ

* [**DEF CON 27 - Unpacking Pkgs A Look Inside Macos Installer Packages And Common Security Flaws**](https://www.youtube.com/watch?v=iASSG0\_zobQ)
* [**OBTS v4.0: "The Wild World of macOS Installers" - Tony Lambert**](https://www.youtube.com/watch?v=Eow5uNHtmIg)
