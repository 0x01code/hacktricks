# macOS Installers दुरुपयोग

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम विशेषज्ञ (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम विशेषज्ञ (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, हैकट्रिक्स**](https://github.com/carlospolop/hacktricks) और [**हैकट्रिक्स क्लाउड**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में PR जमा करके।

</details>
{% endhint %}

## Pkg मूलभूत जानकारी

एक macOS **इंस्टॉलर पैकेज** (जिसे `.pkg` फ़ाइल के रूप में भी जाना जाता है) एक फ़ाइल प्रारूप है जिसका macOS द्वारा **सॉफ़्टवेयर वितरित** करने के लिए उपयोग किया जाता है। ये फ़ाइलें एक **डिस्ट्रीब्यूशन (वर्गीकरण) की तरह होती हैं जो सॉफ़्टवेयर** को सही ढंग से स्थापित और चलाने के लिए सब कुछ जिसकी आवश्यकता होती है, को समेटती है।

पैकेज फ़ाइल स्वयं एक आर्काइव है जो एक **फ़ाइल और निर्देशिकाओं का वर्गीकरण है जो लक्ष्य** कंप्यूटर पर स्थापित किया जाएगा। इसमें **स्क्रिप्ट** भी शामिल हो सकते हैं जो स्थापना से पहले और बाद में कार्यों को करने के लिए होते हैं, जैसे कॉन्फ़िगरेशन फ़ाइलें सेट करना या सॉफ़्टवेयर के पुराने संस्करणों को साफ़ करना।

### वर्गीकरण

<figure><img src="../../../.gitbook/assets/Pasted Graphic.png" alt="https://www.youtube.com/watch?v=iASSG0_zobQ"><figcaption></figcaption></figure>

* **डिस्ट्रीब्यूशन (xml)**: कस्टमाइज़ेशन (शीर्षक, स्वागत पाठ...) और स्क्रिप्ट/स्थापना जांचें
* **पैकेजइन्फो (xml)**: जानकारी, स्थापना आवश्यकताएँ, स्थापना स्थान, स्क्रिप्ट के पथ चलाने के लिए
* **सामग्री का बिल (bom)**: स्थापित करने, अपडेट करने या हटाने के लिए फ़ाइलों की सूची, फ़ाइल अनुमतियाँ के साथ
* **पेलोड (CPIO आर्काइव gzip संपीड़ित)**: `पैकेजइन्फो` से `स्थापन-स्थान` में स्थापित करने के लिए फ़ाइलें
* **स्क्रिप्ट (CPIO आर्काइव gzip संपीड़ित)**: प्री और पोस्ट स्थापना स्क्रिप्ट और अधिक संसाधन निष्कर्षण के लिए एक अस्थायी निर्देशिका में क्रियान्वयन के लिए।

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

DMG फ़ाइलें, या एप्पल डिस्क इमेजेस, एप्पल के macOS द्वारा उपयोग किए जाने वाले फ़ाइल प्रारूप हैं। एक DMG फ़ाइल मूल रूप से एक **माउंट करने योग्य डिस्क इमेज** है (इसमें अपना फ़ाइल सिस्टम होता है) जो सामान्य रूप से संपीड़ित और कभी-कभी एन्क्रिप्टेड रॉ ब्लॉक डेटा को शामिल करता है। जब आप एक DMG फ़ाइल खोलते हैं, macOS उसे एक भौतिक डिस्क की तरह **माउंट करता है**, जिससे आप इसकी सामग्री तक पहुंच सकते हैं।

{% hint style="danger" %}
ध्यान दें कि **`.dmg`** इंस्टॉलर समर्थित **इतने सारे प्रारूप** हैं कि इसका उपयोग करके कुछ वे सुरक्षा दोष संभावना वाले थे जिन्हें अभयुक्ति प्राप्त करने के लिए **कर्नेल कोड निष्पादन** किया गया था।
{% endhint %}

### वर्गीकरण

<figure><img src="../../../.gitbook/assets/image (225).png" alt=""><figcaption></figcaption></figure>

DMG फ़ाइल का वर्गीकरण सामग्री के आधार पर भिन्न हो सकता है। हालांकि, एप्लिकेशन DMG के लिए, यह आम तौर पर इस संरचना का पालन करता है:

* शीर्ष स्तर: यह डिस्क इमेज की जड़ है। यह आम तौर पर एप्लिकेशन और संभावित रूप से एप्लिकेशन्स फ़ोल्डर के लिए एक लिंक शामिल करता है।
* एप्लिकेशन (.app): यह वास्तविक एप्लिकेशन है। macOS में, एक एप्लिकेशन आम तौर पर एक पैकेज होता है जिसमें एप्लिकेशन का गठन करने वाली कई व्यक्तिगत फ़ाइलें और फ़ोल्डर होते हैं।
* एप्लिकेशन्स लिंक: यह macOS में एप्लिकेशन्स फ़ोल्डर के लिए एक शॉर्टकट है। इसका उद्देश्य यह है कि आपको एप्लिकेशन इंस्टॉल करना आसान हो। आप इस शॉर्टकट पर .app फ़ाइल को इंस्टॉल करने के लिए खींच सकते हैं।

## pkg दुरुपयोग के माध्यम से विशेषाधिकार

### सार्वजनिक निर्देशिकाओं से निष्पादन

यदि कोई पूर्व या पोस्ट स्थापना स्क्रिप्ट उदाहरण के लिए **`/var/tmp/Installerutil`** से निष्पादित हो रहा है, और हमलावर उस स्क्रिप्ट को नियंत्रित कर सकता है तो वह जब भी निष्पादित होता है उस समय वह विशेषाधिकार बढ़ा सकता है। या एक और समान उदाहरण:

<figure><img src="../../../.gitbook/assets/Pasted Graphic 5.png" alt="https://www.youtube.com/watch?v=iASSG0_zobQ"><figcaption><p><a href="https://www.youtube.com/watch?v=kCXhIYtODBg">https://www.youtube.com/watch?v=kCXhIYtODBg</a></p></figcaption></figure>

### AuthorizationExecuteWithPrivileges

यह एक [सार्वजनिक फ़ंक्शन](https://developer.apple.com/documentation/security/1540038-authorizationexecutewithprivileg) है जिसे कई इंस्टॉलर और अपडेटर्स रूट के रूप में कुछ को निष्पादित करने के लिए कॉल करेंगे। यह फ़ंक्शन **फ़ाइल** का **पथ** स्वीकार करता है जिसे **निष्पादित** करने के लिए पैरामीटर के रूप में, हालांकि, यदि कोई हमलावर इस फ़ाइल को **संशोधित** कर सकता है, तो वह अपने विशेषाधिकारों को बढ़ाने के लिए इसका दुरुपयोग कर सकता है।
```bash
# Breakpoint in the function to check wich file is loaded
(lldb) b AuthorizationExecuteWithPrivileges
# You could also check FS events to find this missconfig
```
### माउंटिंग द्वारा निष्पादन

यदि एक स्थापक `/tmp/fixedname/bla/bla` में लिखता है, तो `/tmp/fixedname` पर **कोई मालिक नहीं** के साथ एक माउंट बनाना संभव है, जिससे आप स्थापना प्रक्रिया का दुरुपयोग करने के लिए **किसी भी फ़ाइल को संशोधित** कर सकते हैं।

इसका एक उदाहरण **CVE-2021-26089** है जिसने **एक आवधिक स्क्रिप्ट को अधिकारी के रूप में निष्पादित** करने के लिए सफलतापूर्वक **अधिक लिख दिया**। अधिक जानकारी के लिए इस टॉक को देखें: [**OBTS v4.0: "Mount(ain) of Bugs" - Csaba Fitzl**](https://www.youtube.com/watch?v=jSYPazD4VcE)

## मैलवेयर के रूप में pkg

### खाली पेलोड

केवल **प्री और पोस्ट-स्थापना स्क्रिप्ट** के साथ एक **`.pkg`** फ़ाइल उत्पन्न करना संभव है बिना किसी पेलोड के।

### वितरण xml में JS

पैकेज के वितरण xml फ़ाइल में **`<script>`** टैग जोड़ना संभव है और उस कोड को निष्पादित किया जाएगा और यह **`system.run`** का उपयोग करके **कमांड्स निष्पादित** कर सकता है:

<figure><img src="../../../.gitbook/assets/image (1043).png" alt=""><figcaption></figcaption></figure>

## संदर्भ

* [**DEF CON 27 - Unpacking Pkgs A Look Inside Macos Installer Packages And Common Security Flaws**](https://www.youtube.com/watch?v=iASSG0\_zobQ)
* [**OBTS v4.0: "The Wild World of macOS Installers" - Tony Lambert**](https://www.youtube.com/watch?v=Eow5uNHtmIg)
* [**DEF CON 27 - Unpacking Pkgs A Look Inside MacOS Installer Packages**](https://www.youtube.com/watch?v=kCXhIYtODBg)
