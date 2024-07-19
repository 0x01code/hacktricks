# macOS SIP

{% hint style="success" %}
Learn & practice AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Learn & practice GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Check the [**subscription plans**](https://github.com/sponsors/carlospolop)!
* **Join the** 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) or the [**telegram group**](https://t.me/peass) or **follow** us on **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Share hacking tricks by submitting PRs to the** [**HackTricks**](https://github.com/carlospolop/hacktricks) and [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}

### [WhiteIntel](https://whiteintel.io)

<figure><img src="../../../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) एक **डार्क-वेब** द्वारा संचालित सर्च इंजन है जो यह जांचने के लिए **फ्री** कार्यक्षमताएँ प्रदान करता है कि क्या किसी कंपनी या उसके ग्राहकों को **समझौता** किया गया है **चोरी करने वाले मालवेयर** द्वारा।

WhiteIntel का प्राथमिक लक्ष्य जानकारी-चोरी करने वाले मालवेयर के परिणामस्वरूप अकाउंट टेकओवर और रैनसमवेयर हमलों से लड़ना है।

आप उनकी वेबसाइट पर जा सकते हैं और **फ्री** में उनके इंजन को आजमा सकते हैं:

{% embed url="https://whiteintel.io" %}

***

## **Basic Information**

**सिस्टम इंटीग्रिटी प्रोटेक्शन (SIP)** macOS में एक तंत्र है जिसे प्रमुख सिस्टम फ़ोल्डरों में अनधिकृत परिवर्तनों को रोकने के लिए डिज़ाइन किया गया है, यहां तक कि सबसे विशेषाधिकार प्राप्त उपयोगकर्ताओं से भी। यह सुविधा सिस्टम की अखंडता बनाए रखने में महत्वपूर्ण भूमिका निभाती है, जैसे कि संरक्षित क्षेत्रों में फ़ाइलों को जोड़ना, संशोधित करना या हटाना। SIP द्वारा संरक्षित प्रमुख फ़ोल्डर में शामिल हैं:

* **/System**
* **/bin**
* **/sbin**
* **/usr**

SIP के व्यवहार को नियंत्रित करने वाले नियम **`/System/Library/Sandbox/rootless.conf`** में स्थित कॉन्फ़िगरेशन फ़ाइल में परिभाषित हैं। इस फ़ाइल में, उन पथों को जो एक तारांकित (\*) के साथ प्रारंभ होते हैं, अन्यथा कठोर SIP प्रतिबंधों के अपवाद के रूप में दर्शाया गया है।

नीचे दिए गए उदाहरण पर विचार करें:
```javascript
/usr
* /usr/libexec/cups
* /usr/local
* /usr/share/man
```
यह स्निपेट यह संकेत करता है कि जबकि SIP सामान्यतः **`/usr`** निर्देशिका को सुरक्षित करता है, कुछ विशिष्ट उपनिर्देशिकाएँ (`/usr/libexec/cups`, `/usr/local`, और `/usr/share/man`) हैं जहाँ संशोधन की अनुमति है, जैसा कि उनके पथों के पहले अsterisk (\*) द्वारा संकेतित है।

यह सत्यापित करने के लिए कि क्या कोई निर्देशिका या फ़ाइल SIP द्वारा सुरक्षित है, आप **`ls -lOd`** कमांड का उपयोग कर सकते हैं ताकि **`restricted`** या **`sunlnk`** ध्वज की उपस्थिति की जांच की जा सके। उदाहरण के लिए:
```bash
ls -lOd /usr/libexec/cups
drwxr-xr-x  11 root  wheel  sunlnk 352 May 13 00:29 /usr/libexec/cups
```
इस मामले में, **`sunlnk`** ध्वज यह दर्शाता है कि `/usr/libexec/cups` निर्देशिका स्वयं **हटाई नहीं जा सकती**, हालांकि इसके भीतर फ़ाइलें बनाई, संशोधित या हटाई जा सकती हैं।

दूसरी ओर:
```bash
ls -lOd /usr/libexec
drwxr-xr-x  338 root  wheel  restricted 10816 May 13 00:29 /usr/libexec
```
यहाँ, **`restricted`** ध्वज इंगित करता है कि `/usr/libexec` निर्देशिका SIP द्वारा संरक्षित है। SIP-संरक्षित निर्देशिका में फ़ाइलें बनाई, संशोधित या हटाई नहीं जा सकतीं।

इसके अलावा, यदि एक फ़ाइल में **`com.apple.rootless`** विस्तारित **attribute** है, तो वह फ़ाइल भी **SIP द्वारा संरक्षित** होगी।

**SIP अन्य रूट क्रियाओं को भी सीमित करता है** जैसे:

* अविश्वसनीय कर्नेल एक्सटेंशन लोड करना
* Apple-हस्ताक्षरित प्रक्रियाओं के लिए कार्य-ports प्राप्त करना
* NVRAM चर को संशोधित करना
* कर्नेल डिबगिंग की अनुमति देना

विकल्प nvram चर में एक बिटफ्लैग के रूप में बनाए रखे जाते हैं (`csr-active-config` Intel पर और `lp-sip0` ARM के लिए बूट किए गए डिवाइस ट्री से पढ़ा जाता है)। आप XNU स्रोत कोड में `csr.sh` में ध्वज पा सकते हैं:

<figure><img src="../../../.gitbook/assets/image (1192).png" alt=""><figcaption></figcaption></figure>

### SIP स्थिति

आप निम्नलिखित कमांड के साथ जांच सकते हैं कि क्या SIP आपके सिस्टम पर सक्षम है:
```bash
csrutil status
```
यदि आपको SIP को अक्षम करने की आवश्यकता है, तो आपको अपने कंप्यूटर को रिकवरी मोड में पुनः प्रारंभ करना होगा (स्टार्टअप के दौरान Command+R दबाकर), फिर निम्नलिखित कमांड निष्पादित करें:
```bash
csrutil disable
```
यदि आप SIP को सक्षम रखना चाहते हैं लेकिन डिबगिंग सुरक्षा को हटाना चाहते हैं, तो आप ऐसा कर सकते हैं:
```bash
csrutil enable --without debug
```
### अन्य प्रतिबंध

* **असत्यापित कर्नेल एक्सटेंशन** (kexts) को लोड करने की अनुमति नहीं देता, यह सुनिश्चित करता है कि केवल सत्यापित एक्सटेंशन सिस्टम कर्नेल के साथ इंटरैक्ट करें।
* **macOS सिस्टम प्रक्रियाओं** का डिबगिंग रोकता है, जिससे कोर सिस्टम घटकों को अनधिकृत पहुंच और संशोधन से सुरक्षित रखा जाता है।
* **dtrace जैसे उपकरणों** को सिस्टम प्रक्रियाओं का निरीक्षण करने से रोकता है, जिससे सिस्टम के संचालन की अखंडता को और अधिक सुरक्षित किया जाता है।

[**इस वार्ता में SIP जानकारी के बारे में अधिक जानें**](https://www.slideshare.net/i0n1c/syscan360-stefan-esser-os-x-el-capitan-sinking-the-ship)**।**

## SIP बायपास

SIP को बायपास करने से एक हमलावर को:

* **उपयोगकर्ता डेटा तक पहुंच**: सभी उपयोगकर्ता खातों से संवेदनशील उपयोगकर्ता डेटा जैसे मेल, संदेश और सफारी इतिहास पढ़ें।
* **TCC बायपास**: TCC (Transparency, Consent, and Control) डेटाबेस को सीधे संशोधित करें ताकि वेबकैम, माइक्रोफोन और अन्य संसाधनों तक अनधिकृत पहुंच प्रदान की जा सके।
* **स्थायीता स्थापित करें**: SIP-सुरक्षित स्थानों में मैलवेयर रखें, जिससे इसे हटाने के लिए प्रतिरोधी बनाया जा सके, यहां तक कि रूट विशेषाधिकारों द्वारा भी। इसमें मैलवेयर हटाने के उपकरण (MRT) के साथ छेड़छाड़ करने की संभावना भी शामिल है।
* **कर्नेल एक्सटेंशन लोड करें**: हालांकि अतिरिक्त सुरक्षा उपाय हैं, SIP को बायपास करना असत्यापित कर्नेल एक्सटेंशन लोड करने की प्रक्रिया को सरल बनाता है।

### इंस्टॉलर पैकेज

**एप्पल के प्रमाणपत्र से हस्ताक्षरित इंस्टॉलर पैकेज** इसकी सुरक्षा को बायपास कर सकते हैं। इसका मतलब है कि यदि वे SIP-सुरक्षित निर्देशिकाओं को संशोधित करने का प्रयास करते हैं तो मानक डेवलपर्स द्वारा हस्ताक्षरित पैकेज भी अवरुद्ध हो जाएंगे।

### अस्तित्वहीन SIP फ़ाइल

एक संभावित छिद्र यह है कि यदि **`rootless.conf` में एक फ़ाइल निर्दिष्ट की गई है लेकिन वर्तमान में मौजूद नहीं है**, तो इसे बनाया जा सकता है। मैलवेयर इसका लाभ उठाकर **सिस्टम पर स्थायीता स्थापित** कर सकता है। उदाहरण के लिए, एक दुर्भावनापूर्ण प्रोग्राम `/System/Library/LaunchDaemons` में एक .plist फ़ाइल बना सकता है यदि यह `rootless.conf` में सूचीबद्ध है लेकिन मौजूद नहीं है।

### com.apple.rootless.install.heritable

{% hint style="danger" %}
अधिकार **`com.apple.rootless.install.heritable`** SIP को बायपास करने की अनुमति देता है
{% endhint %}

#### [CVE-2019-8561](https://objective-see.org/blog/blog\_0x42.html) <a href="#cve" id="cve"></a>

यह पाया गया कि **सिस्टम द्वारा इसके कोड** हस्ताक्षर की पुष्टि करने के बाद इंस्टॉलर पैकेज को **स्वैप करना संभव था** और फिर, सिस्टम मूल के बजाय दुर्भावनापूर्ण पैकेज स्थापित करेगा। चूंकि ये क्रियाएँ **`system_installd`** द्वारा की गई थीं, यह SIP को बायपास करने की अनुमति देगा।

#### [CVE-2020–9854](https://objective-see.org/blog/blog\_0x4D.html) <a href="#cve-unauthd-chain" id="cve-unauthd-chain"></a>

यदि एक पैकेज एक माउंटेड इमेज या बाहरी ड्राइव से स्थापित किया गया था, तो **इंस्टॉलर** **उस फ़ाइल सिस्टम** से बाइनरी को **निष्पादित** करेगा (SIP-सुरक्षित स्थान से नहीं), जिससे **`system_installd`** एक मनमाना बाइनरी निष्पादित करेगा।

#### CVE-2021-30892 - Shrootless

[**इस ब्लॉग पोस्ट के शोधकर्ताओं**](https://www.microsoft.com/en-us/security/blog/2021/10/28/microsoft-finds-new-macos-vulnerability-shrootless-that-could-bypass-system-integrity-protection/) ने macOS के सिस्टम इंटीग्रिटी प्रोटेक्शन (SIP) तंत्र में एक भेद्यता का पता लगाया, जिसे 'Shrootless' भेद्यता कहा गया। यह भेद्यता **`system_installd`** डेमन के चारों ओर केंद्रित है, जिसमें एक अधिकार है, **`com.apple.rootless.install.heritable`**, जो इसके किसी भी बाल प्रक्रिया को SIP के फ़ाइल सिस्टम प्रतिबंधों को बायपास करने की अनुमति देता है।

**`system_installd`** डेमन उन पैकेजों को स्थापित करेगा जो **Apple** द्वारा हस्ताक्षरित हैं।

शोधकर्ताओं ने पाया कि Apple द्वारा हस्ताक्षरित पैकेज (.pkg फ़ाइल) की स्थापना के दौरान, **`system_installd`** पैकेज में शामिल किसी भी **पोस्ट-इंस्टॉल** स्क्रिप्ट को **चलाता है**। ये स्क्रिप्ट डिफ़ॉल्ट शेल, **`zsh`** द्वारा निष्पादित की जाती हैं, जो स्वचालित रूप से **`/etc/zshenv`** फ़ाइल से कमांड चलाती है, यदि यह मौजूद है, यहां तक कि गैर-इंटरैक्टिव मोड में भी। इस व्यवहार का उपयोग हमलावरों द्वारा किया जा सकता है: एक दुर्भावनापूर्ण `/etc/zshenv` फ़ाइल बनाकर और **`system_installd` को `zsh`** को सक्रिय करने की प्रतीक्षा करके, वे डिवाइस पर मनमाने ऑपरेशन कर सकते हैं।

इसके अलावा, यह पाया गया कि **`/etc/zshenv` को एक सामान्य हमले की तकनीक के रूप में उपयोग किया जा सकता है**, न कि केवल SIP बायपास के लिए। प्रत्येक उपयोगकर्ता प्रोफ़ाइल में एक `~/.zshenv` फ़ाइल होती है, जो `/etc/zshenv` की तरह व्यवहार करती है लेकिन रूट अनुमतियों की आवश्यकता नहीं होती। इस फ़ाइल का उपयोग एक स्थायीता तंत्र के रूप में किया जा सकता है, जो हर बार `zsh` शुरू होने पर ट्रिगर होता है, या विशेषाधिकार बढ़ाने के तंत्र के रूप में। यदि एक व्यवस्थापक उपयोगकर्ता `sudo -s` या `sudo <command>` का उपयोग करके रूट में बढ़ता है, तो `~/.zshenv` फ़ाइल ट्रिगर होगी, प्रभावी रूप से रूट में बढ़ते हुए।

#### [**CVE-2022-22583**](https://perception-point.io/blog/technical-analysis-cve-2022-22583/)

[**CVE-2022-22583**](https://perception-point.io/blog/technical-analysis-cve-2022-22583/) में यह पाया गया कि वही **`system_installd`** प्रक्रिया अभी भी दुरुपयोग की जा सकती है क्योंकि यह **`/tmp` के अंदर SIP द्वारा संरक्षित एक यादृच्छिक नामित फ़ोल्डर के अंदर पोस्ट-इंस्टॉल स्क्रिप्ट डाल रही थी**। बात यह है कि **`/tmp` स्वयं SIP द्वारा संरक्षित नहीं है**, इसलिए एक **वर्चुअल इमेज को उस पर माउंट** करना संभव था, फिर **इंस्टॉलर** वहां **पोस्ट-इंस्टॉल स्क्रिप्ट** डालता, **वर्चुअल इमेज को अनमाउंट** करता, **सभी फ़ोल्डरों को फिर से बनाता** और **पेलोड** के साथ **पोस्ट इंस्टॉलेशन** स्क्रिप्ट को जोड़ता।

#### [fsck\_cs उपयोगिता](https://www.theregister.com/2016/03/30/apple\_os\_x\_rootless/)

एक भेद्यता की पहचान की गई जहां **`fsck_cs`** को एक महत्वपूर्ण फ़ाइल को भ्रष्ट करने के लिए गुमराह किया गया, इसकी **सांकेतिक लिंक** का पालन करने की क्षमता के कारण। विशेष रूप से, हमलावरों ने _`/dev/diskX`_ से फ़ाइल `/System/Library/Extensions/AppleKextExcludeList.kext/Contents/Info.plist` के लिए एक लिंक तैयार किया। _`/dev/diskX`_ पर **`fsck_cs`** को निष्पादित करने से `Info.plist` का भ्रष्टाचार हुआ। इस फ़ाइल की अखंडता ऑपरेटिंग सिस्टम के SIP (सिस्टम इंटीग्रिटी प्रोटेक्शन) के लिए महत्वपूर्ण है, जो कर्नेल एक्सटेंशन के लोडिंग को नियंत्रित करता है। एक बार भ्रष्ट होने पर, SIP की कर्नेल अपवाद प्रबंधन की क्षमता प्रभावित होती है।

इस भेद्यता का लाभ उठाने के लिए आदेश हैं:
```bash
ln -s /System/Library/Extensions/AppleKextExcludeList.kext/Contents/Info.plist /dev/diskX
fsck_cs /dev/diskX 1>&-
touch /Library/Extensions/
reboot
```
इस कमजोरियों के शोषण के गंभीर परिणाम हैं। `Info.plist` फ़ाइल, जो सामान्यतः कर्नेल एक्सटेंशन के लिए अनुमतियों का प्रबंधन करने के लिए जिम्मेदार होती है, अप्रभावी हो जाती है। इसमें कुछ एक्सटेंशनों, जैसे `AppleHWAccess.kext` को ब्लैकलिस्ट करने की असमर्थता शामिल है। परिणामस्वरूप, SIP के नियंत्रण तंत्र के खराब होने के कारण, इस एक्सटेंशन को लोड किया जा सकता है, जो सिस्टम की RAM तक अनधिकृत पढ़ने और लिखने की पहुंच प्रदान करता है।

#### [SIP संरक्षित फ़ोल्डरों पर माउंट करें](https://www.slideshare.net/i0n1c/syscan360-stefan-esser-os-x-el-capitan-sinking-the-ship)

**सुरक्षा को बायपास करने के लिए SIP संरक्षित फ़ोल्डरों पर एक नई फ़ाइल प्रणाली को माउंट करना संभव था।**
```bash
mkdir evil
# Add contento to the folder
hdiutil create -srcfolder evil evil.dmg
hdiutil attach -mountpoint /System/Library/Snadbox/ evil.dmg
```
#### [Upgrader bypass (2016)](https://objective-see.org/blog/blog\_0x14.html)

सिस्टम को `Install macOS Sierra.app` के भीतर एक एम्बेडेड इंस्टॉलर डिस्क इमेज से बूट करने के लिए सेट किया गया है ताकि OS को अपग्रेड किया जा सके, `bless` उपयोगिता का उपयोग करते हुए। उपयोग की जाने वाली कमांड इस प्रकार है:
```bash
/usr/sbin/bless -setBoot -folder /Volumes/Macintosh HD/macOS Install Data -bootefi /Volumes/Macintosh HD/macOS Install Data/boot.efi -options config="\macOS Install Data\com.apple.Boot" -label macOS Installer
```
The security of this process can be compromised if an attacker alters the upgrade image (`InstallESD.dmg`) before booting. The strategy involves substituting a dynamic loader (dyld) with a malicious version (`libBaseIA.dylib`). This replacement results in the execution of the attacker's code when the installer is initiated.

The attacker's code gains control during the upgrade process, exploiting the system's trust in the installer. The attack proceeds by altering the `InstallESD.dmg` image via method swizzling, particularly targeting the `extractBootBits` method. This allows the injection of malicious code before the disk image is employed.

Moreover, within the `InstallESD.dmg`, there's a `BaseSystem.dmg`, which serves as the upgrade code's root file system. Injecting a dynamic library into this allows the malicious code to operate within a process capable of altering OS-level files, significantly increasing the potential for system compromise.

#### [systemmigrationd (2023)](https://www.youtube.com/watch?v=zxZesAN-TEk)

In this talk from [**DEF CON 31**](https://www.youtube.com/watch?v=zxZesAN-TEk), it's shown how **`systemmigrationd`** (which can bypass SIP) executes a **bash** and a **perl** script, which can be abused via env variables **`BASH_ENV`** and **`PERL5OPT`**.

#### CVE-2023-42860 <a href="#cve-a-detailed-look" id="cve-a-detailed-look"></a>

As [**detailed in this blog post**](https://blog.kandji.io/apple-mitigates-vulnerabilities-installer-scripts), a `postinstall` script from `InstallAssistant.pkg` packages allowed was executing:
```bash
/usr/bin/chflags -h norestricted "${SHARED_SUPPORT_PATH}/SharedSupport.dmg"
```
and it was possible to crate a symlink in `${SHARED_SUPPORT_PATH}/SharedSupport.dmg` that would allow a user to **unrestrict any file, bypassing SIP protection**.

### **com.apple.rootless.install**

{% hint style="danger" %}
The entitlement **`com.apple.rootless.install`** allows to bypass SIP
{% endhint %}

The entitlement `com.apple.rootless.install` is known to bypass System Integrity Protection (SIP) on macOS. This was notably mentioned in relation to [**CVE-2022-26712**](https://jhftss.github.io/CVE-2022-26712-The-POC-For-SIP-Bypass-Is-Even-Tweetable/).

In this specific case, the system XPC service located at `/System/Library/PrivateFrameworks/ShoveService.framework/Versions/A/XPCServices/SystemShoveService.xpc` possesses this entitlement. This allows the related process to circumvent SIP constraints. Furthermore, this service notably presents a method that permits the movement of files without enforcing any security measures.

## Sealed System Snapshots

Sealed System Snapshots are a feature introduced by Apple in **macOS Big Sur (macOS 11)** as a part of its **System Integrity Protection (SIP)** mechanism to provide an additional layer of security and system stability. They are essentially read-only versions of the system volume.

Here's a more detailed look:

1. **Immutable System**: Sealed System Snapshots make the macOS system volume "immutable", meaning that it cannot be modified. This prevents any unauthorised or accidental changes to the system that could compromise security or system stability.
2. **System Software Updates**: When you install macOS updates or upgrades, macOS creates a new system snapshot. The macOS startup volume then uses **APFS (Apple File System)** to switch to this new snapshot. The entire process of applying updates becomes safer and more reliable as the system can always revert to the previous snapshot if something goes wrong during the update.
3. **Data Separation**: In conjunction with the concept of Data and System volume separation introduced in macOS Catalina, the Sealed System Snapshot feature makes sure that all your data and settings are stored on a separate "**Data**" volume. This separation makes your data independent from the system, which simplifies the process of system updates and enhances system security.

Remember that these snapshots are automatically managed by macOS and don't take up additional space on your disk, thanks to the space sharing capabilities of APFS. It’s also important to note that these snapshots are different from **Time Machine snapshots**, which are user-accessible backups of the entire system.

### Check Snapshots

The command **`diskutil apfs list`** lists the **details of the APFS volumes** and their layout:

<pre><code>+-- Container disk3 966B902E-EDBA-4775-B743-CF97A0556A13
|   ====================================================
|   APFS Container Reference:     disk3
|   Size (Capacity Ceiling):      494384795648 B (494.4 GB)
|   Capacity In Use By Volumes:   219214536704 B (219.2 GB) (44.3% used)
|   Capacity Not Allocated:       275170258944 B (275.2 GB) (55.7% free)
|   |
|   +-&#x3C; Physical Store disk0s2 86D4B7EC-6FA5-4042-93A7-D3766A222EBE
|   |   -----------------------------------------------------------
|   |   APFS Physical Store Disk:   disk0s2
|   |   Size:                       494384795648 B (494.4 GB)
|   |
|   +-> Volume disk3s1 7A27E734-880F-4D91-A703-FB55861D49B7
|   |   ---------------------------------------------------
<strong>|   |   APFS Volume Disk (Role):   disk3s1 (System)
</strong>|   |   Name:                      Macintosh HD (Case-insensitive)
<strong>|   |   Mount Point:               /System/Volumes/Update/mnt1
</strong>|   |   Capacity Consumed:         12819210240 B (12.8 GB)
|   |   Sealed:                    Broken
|   |   FileVault:                 Yes (Unlocked)
|   |   Encrypted:                 No
|   |   |
|   |   Snapshot:                  FAA23E0C-791C-43FF-B0E7-0E1C0810AC61
|   |   Snapshot Disk:             disk3s1s1
<strong>|   |   Snapshot Mount Point:      /
</strong><strong>|   |   Snapshot Sealed:           Yes
</strong>[...]
+-> Volume disk3s5 281959B7-07A1-4940-BDDF-6419360F3327
|   ---------------------------------------------------
|   APFS Volume Disk (Role):   disk3s5 (Data)
|   Name:                      Macintosh HD - Data (Case-insensitive)
<strong>    |   Mount Point:               /System/Volumes/Data
</strong><strong>    |   Capacity Consumed:         412071784448 B (412.1 GB)
</strong>    |   Sealed:                    No
|   FileVault:                 Yes (Unlocked)
</code></pre>

In the previous output it's possible to see that **user-accessible locations** are mounted under `/System/Volumes/Data`.

Moreover, **macOS System volume snapshot** is mounted in `/` and it's **sealed** (cryptographically signed by the OS). So, if SIP is bypassed and modifies it, the **OS won't boot anymore**.

It's also possible to **verify that seal is enabled** by running:
```bash
csrutil authenticated-root status
Authenticated Root status: enabled
```
इसके अलावा, स्नैपशॉट डिस्क को **पढ़ने के लिए केवल** के रूप में भी माउंट किया गया है:
```bash
mount
/dev/disk3s1s1 on / (apfs, sealed, local, read-only, journaled)
```
### [WhiteIntel](https://whiteintel.io)

<figure><img src="../../../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) एक **डार्क-वेब** द्वारा संचालित सर्च इंजन है जो **मुफ्त** कार्यक्षमताएँ प्रदान करता है ताकि यह जांचा जा सके कि क्या कोई कंपनी या उसके ग्राहक **स्टीलर मालवेयर** द्वारा **समझौता** किए गए हैं।

WhiteIntel का प्राथमिक लक्ष्य जानकारी-चोरी करने वाले मालवेयर के परिणामस्वरूप अकाउंट टेकओवर और रैनसमवेयर हमलों से लड़ना है।

आप उनकी वेबसाइट पर जा सकते हैं और **मुफ्त** में उनके इंजन का प्रयास कर सकते हैं:

{% embed url="https://whiteintel.io" %}
{% hint style="success" %}
Learn & practice AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Learn & practice GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Check the [**subscription plans**](https://github.com/sponsors/carlospolop)!
* **Join the** 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) or the [**telegram group**](https://t.me/peass) or **follow** us on **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Share hacking tricks by submitting PRs to the** [**HackTricks**](https://github.com/carlospolop/hacktricks) and [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}
</details>
