# फर्मवेयर विश्लेषण

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live) **पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** द्वारा PRs सबमिट करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos का उपयोग करें।

</details>

## **परिचय**

फर्मवेयर महत्वपूर्ण सॉफ़्टवेयर है जो उपकरणों को सही ढंग से काम करने की संभावना देता है जिससे हार्डवेयर संघटनों और उपयोगकर्ताओं के संवाद को प्रबंधित और सुविधाजनक बनाने में मदद मिलती है। यह स्थायी स्मृति में संग्रहीत होता है, इसे सुनिश्चित करता है कि उपकरण महत्वपूर्ण निर्देशों तक पहुंच सकता है जब यह चालू किया जाता है, जिससे ऑपरेटिंग सिस्टम का लॉन्च होता है। फर्मवेयर की जांच और संभावना से फर्मवेयर में परिवर्तन करना सुरक्षा दोषों की पहचान में एक महत्वपूर्ण कदम है।

## **जानकारी एकत्र करना**

**जानकारी एकत्र करना** एक उपकरण के निर्माण और उसमें उपयोग की तकनीकों को समझने का एक महत्वपूर्ण प्रारंभिक कदम है। इस प्रक्रिया में डेटा एकत्रित करना शामिल है:

- CPU वास्तुकला और ऑपरेटिंग सिस्टम जिस पर यह चलता है
- बूटलोडर विशेषताएँ
- हार्डवेयर लेआउट और डेटाशीट्स
- कोडबेस मैट्रिक्स और स्रोत स्थान
- बाह्य पुस्तकालय और लाइसेंस प्रकार
- अपडेट इतिहास और नियामक प्रमाणपत्र
- वास्तुकला और फ्लो डायग्राम
- सुरक्षा मूल्यांकन और पहचाने गए सुरक्षा दोष

इस उद्देश्य के लिए, **ओपन-सोर्स इंटेलिजेंस (OSINT)** उपकरण अमूल्य हैं, जैसे कि किसी भी उपलब्ध ओपन-सोर्स सॉफ़्टवेयर घटकों का मैन्युअल और स्वचालित समीक्षा प्रक्रियाओं के माध्यम से विश्लेषण। [Coverity Scan](https://scan.coverity.com) और [Semmle’s LGTM](https://lgtm.com/#explore) जैसे उपकरण मुफ्त स्थिर विश्लेषण प्रदान करते हैं जिनका उपयोग संभावित मुद्दों को खोजने के लिए किया जा सकता है।

## **फर्मवेयर प्राप्त करना**

फर्मवेयर प्राप्त करने के लिए विभिन्न तरीकों से उपाय किया जा सकता है, प्रत्येक के अपने स्तर के साथ:

- **स्रोत से सीधे** (डेवलपर, निर्माता)
- प्रदत्त निर्देशों से इसे **बिल्ड** करना
- आधिकारिक समर्थन साइट से **डाउनलोड** करना
- **होस्ट किए गए फर्मवेयर फ़ाइलें खोजने के लिए** गूगल डॉर्क क्वेरी का उपयोग करना
- [S3Scanner](https://github.com/sa7mon/S3Scanner) जैसे उपकरण के साथ **क्लाउड स्टोरेज** का सीधा उपयोग करना
- मध्य में मैन-इन-द-मिडिल तकनीकों के माध्यम से **अपडेट** को एक्सेस करना
- **उपकरण के माध्यम से निकालना** जैसे कनेक्शन के माध्यम से फर्मवेयर को **यूएआरटी**, **जेटैग** या **पिसिट** के माध्यम से
- उपकरण संचार में **अपड
```bash
file <bin>
strings -n8 <bin>
strings -tx <bin> #print offsets in hex
hexdump -C -n 512 <bin> > hexdump.out
hexdump -C <bin> | head # might find signatures in header
fdisk -lu <bin> #lists a drives partition and filesystems if multiple
```
यदि आप उन उपकरणों से बहुत कुछ नहीं पा रहे हैं, तो छवि की **एंट्रोपी** की जांच करें `binwalk -E <bin>` के साथ, यदि एंट्रोपी कम है, तो यह संभावना नहीं है कि यह एन्क्रिप्टेड है। यदि एंट्रोपी अधिक है, तो यह संभावना है कि यह एन्क्रिप्टेड है (या किसी प्रकार से संकुचित है)।

इसके अतिरिक्त, आप इन उपकरणों का उपयोग कर सकते हैं **फर्मवेयर के भीतर संबोधित फ़ाइलें निकालने के लिए**:

{% content-ref url="../../forensics/basic-forensic-methodology/partitions-file-systems-carving/file-data-carving-recovery-tools.md" %}
[file-data-carving-recovery-tools.md](../../forensics/basic-forensic-methodology/partitions-file-systems-carving/file-data-carving-recovery-tools.md)
{% endcontent-ref %}

या [**binvis.io**](https://binvis.io/#/) ([कोड](https://code.google.com/archive/p/binvis/)) फ़ाइल की जांच करने के लिए।

### फ़ाइल सिस्टम प्राप्त करना

पिछले टिप्पणीत उपकरणों जैसे `binwalk -ev <bin>` के साथ आपको **फ़ाइल सिस्टम को निकालने** की क्षमता होनी चाहिए।\
बिनवॉक आम तौर पर इसे एक **फ़ोल्डर के भीतर निकालता है जिसे फ़ाइल सिस्टम प्रकार के रूप में नामित किया गया है**, जो आम तौर पर निम्नलिखित में से एक होता है: squashfs, ubifs, romfs, rootfs, jffs2, yaffs2, cramfs, initramfs।

#### मैनुअल फ़ाइल सिस्टम निकालना

कभी-कभी, बिनवॉक के **सिग्नेचर में फ़ाइल सिस्टम का जादू बाइट नहीं होगा**। इन मामलों में, बिनवॉक का उपयोग करें ताकि फ़ाइल सिस्टम का ऑफसेट पता लगाएं और बाइनरी से संकुचित फ़ाइल सिस्टम को निकालें और फिर निम्नलिखित चरणों का पालन करके फ़ाइल सिस्टम को **मैन्युअल रूप से निकालें**।
```
$ binwalk DIR850L_REVB.bin

DECIMAL HEXADECIMAL DESCRIPTION
----------------------------------------------------------------------------- ---

0 0x0 DLOB firmware header, boot partition: """"dev=/dev/mtdblock/1""""
10380 0x288C LZMA compressed data, properties: 0x5D, dictionary size: 8388608 bytes, uncompressed size: 5213748 bytes
1704052 0x1A0074 PackImg section delimiter tag, little endian size: 32256 bytes; big endian size: 8257536 bytes
1704084 0x1A0094 Squashfs filesystem, little endian, version 4.0, compression:lzma, size: 8256900 bytes, 2688 inodes, blocksize: 131072 bytes, created: 2016-07-12 02:28:41
```
निम्नलिखित **dd कमांड** चलाएं और Squashfs फ़ाइल सिस्टम कार्विंग करें।
```
$ dd if=DIR850L_REVB.bin bs=1 skip=1704084 of=dir.squashfs

8257536+0 records in

8257536+0 records out

8257536 bytes (8.3 MB, 7.9 MiB) copied, 12.5777 s, 657 kB/s
```
वैकल्पिक रूप से, निम्नलिखित कमांड भी चलाया जा सकता है।

`$ dd if=DIR850L_REVB.bin bs=1 skip=$((0x1A0094)) of=dir.squashfs`

* स्क्वॉशएफएस के लिए (ऊपर के उदाहरण में उपयोग किया गया)

`$ unsquashfs dir.squashfs`

फ़ाइलें इसके बाद "`squashfs-root`" निर्देशिका में होंगी।

* सीपीआईओ आर्काइव फ़ाइलें

`$ cpio -ivd --no-absolute-filenames -F <bin>`

* jffs2 फ़ाइल सिस्टम के लिए

`$ jefferson rootfsfile.jffs2`

* NAND फ़्लैश के साथ ubifs फ़ाइल सिस्टम के लिए

`$ ubireader_extract_images -u UBI -s <start_offset> <bin>`

`$ ubidump.py <bin>`


## फर्मवेयर विश्लेषण

एक बार फर्मवेयर प्राप्त हो जाने पर, इसे उसकी संरचना और संभावित सुरक्षा दोषों को समझने के लिए विश्लेषित करना महत्वपूर्ण है। इस प्रक्रिया में फर्मवेयर छवि से मूल्यवान डेटा को विश्लेषित करने के लिए विभिन्न उपकरणों का उपयोग किया जाता है।

### प्रारंभिक विश्लेषण उपकरण

एक सेट के रूप में कमांड प्रदान किए गए हैं जो बाइनरी फ़ाइल (जिसे `<bin>` के रूप में संदर्भित किया गया है) की प्रारंभिक जांच के लिए मददगार हैं। ये कमांड फ़ाइल प्रकारों की पहचान करने, स्ट्रिंग्स निकालने, बाइनरी डेटा का विश्लेषण करने, और विभाजन और फ़ाइल सिस्टम विवरणों को समझने में मदद करते हैं:
```bash
file <bin>
strings -n8 <bin>
strings -tx <bin> #prints offsets in hexadecimal
hexdump -C -n 512 <bin> > hexdump.out
hexdump -C <bin> | head #useful for finding signatures in the header
fdisk -lu <bin> #lists partitions and filesystems, if there are multiple
```
छवि के एन्क्रिप्शन स्थिति का मूल्यांकन करने के लिए **entropy** को `binwalk -E <bin>` के साथ जांचा जाता है। कम entropy एन्क्रिप्शन की कमी का सुझाव देता है, जबकि उच्च entropy संभावित एन्क्रिप्शन या संपीड़न को दर्शाता है।

**एम्बेडेड फ़ाइल** को निकालने के लिए, **file-data-carving-recovery-tools** दस्तावेज़ और फ़ाइल जांच के लिए **binvis.io** जैसे उपकरण और संसाधनों की सिफारिश की जाती है।

### फ़ाइल सिस्टम निकालना

`binwalk -ev <bin>` का उपयोग करके, आम तौर पर फ़ाइल सिस्टम निकाला जा सकता है, अक्सर फ़ाइल सिस्टम प्रकार के नाम से नामित एक निर्देशिका में (जैसे, squashfs, ubifs)। हालांकि, जब **binwalk** मैजिक बाइट्स की कमी के कारण फ़ाइल सिस्टम प्रकार को पहचानने में असमर्थ हो जाता है, तो मैन्युअल निकालना आवश्यक होता है। इसमें शामिल है `binwalk` का उपयोग करके फ़ाइल सिस्टम के ऑफसेट को ढूंढना, जिसके बाद `dd` कमांड का उपयोग करके फ़ाइल सिस्टम को निकालना:
```bash
$ binwalk DIR850L_REVB.bin

$ dd if=DIR850L_REVB.bin bs=1 skip=1704084 of=dir.squashfs
```
### फाइलसिस्टम विश्लेषण

फाइलसिस्टम को निकालने के बाद, सुरक्षा दोषों की खोज शुरू होती है। असुरक्षित नेटवर्क डेमन, हार्डकोड क्रेडेंशियल्स, एपीआई एंडपॉइंट्स, अपडेट सर्वर कार्यक्षमताएं, अनकंपाइल्ड कोड, स्टार्टअप स्क्रिप्ट्स, और ऑफलाइन विश्लेषण के लिए कंपाइल्ड बाइनरी के लिए ध्यान दिया जाता है।

**महत्वपूर्ण स्थान** और **आइटम** जिन्हें जांचने के लिए शामिल किया जाता है:

- **etc/shadow** और **etc/passwd** उपयोगकर्ता क्रेडेंशियल्स के लिए
- **etc/ssl** में SSL सर्टिफिकेट और कुंजी
- संभावित जोखिम के लिए कॉन्फ़िगरेशन और स्क्रिप्ट फ़ाइलें
- और विश्लेषण के लिए एम्बेडेड बाइनरी
- सामान्य आईओटी डिवाइस वेब सर्वर और बाइनरी

फाइलसिस्टम के भीतर संवेदनशील जानकारी और दोषों का पता लगाने में कई उपकरण सहायक हैं:

- [**LinPEAS**](https://github.com/carlospolop/PEASS-ng) और [**Firmwalker**](https://github.com/craigz28/firmwalker) संवेदनशील जानकारी खोज के लिए
- [**The Firmware Analysis and Comparison Tool (FACT)**](https://github.com/fkie-cad/FACT\_core) व्यापक फर्मवेयर विश्लेषण के लिए
- [**FwAnalyzer**](https://github.com/cruise-automation/fwanalyzer), [**ByteSweep**](https://gitlab.com/bytesweep/bytesweep), [**ByteSweep-go**](https://gitlab.com/bytesweep/bytesweep-go), और [**EMBA**](https://github.com/e-m-b-a/emba) स्थिर और गतिशील विश्लेषण के लिए

### कंपाइल्ड बाइनरी पर सुरक्षा जांच

फाइलसिस्टम में पाए जाने वाले स्रोत कोड और कंपाइल्ड बाइनरी को दोषों के लिए जांचना चाहिए। **checksec.sh** जैसे उनिक्स बाइनरी और **PESecurity** जैसे विंडोज बाइनरी के लिए उपकरण असुरक्षित बाइनरी की पहचान करने में मदद करते हैं।

## डायनामिक विश्लेषण के लिए फर्मवेयर का अनुकरण

फर्मवेयर का अनुकरण करने की प्रक्रिया एक डिवाइस के संचालन या एक व्यक्तिगत प्रोग्राम का **डायनामिक विश्लेषण** संभव बनाती है। इस दृष्टिकोण को हार्डवेयर या वास्तुकला निर्भरताओं के साथ चुनौतियों का सामना कर सकता है, लेकिन जैसे ही रूट फ़ाइलसिस्टम या विशेष बाइनरी को मैचिंग वास्तुकला और एंडियनेस के साथ डिवाइस जैसे रास्पबेरी पाई या पूर्व-निर्मित वर्चुअल मशीन पर स्थानांतरित किया जा सकता है, जैसे कि एक रास्पबेरी पाई, या एक पूर्व-निर्मित वर्चुअल मशीन, विस्तारित परीक्षण को सुविधाजनक बना सकता है।

### व्यक्तिगत बाइनरी का अनुकरण

एकल कार्यक्रमों की जांच के लिए, कार्यक्रम की एंडियनेस और सीपीयू वास्तव में महत्वपूर्ण है।

#### MIPS वास्तुकला के साथ उदाहरण
```bash
file ./squashfs-root/bin/busybox
```
और आवश्यक अनुकरण उपकरण स्थापित करने के लिए:
```bash
sudo apt-get install qemu qemu-user qemu-user-static qemu-system-arm qemu-system-mips qemu-system-x86 qemu-utils
```
### MIPS (बड़े-एंडियन) के लिए, `qemu-mips` का उपयोग किया जाता है, और छोटे-एंडियन बाइनरी के लिए, `qemu-mipsel` एक विकल्प होगा।

#### ARM आर्किटेक्चर इम्युलेशन

ARM बाइनरी के लिए, प्रक्रिया समान है, `qemu-arm` इम्युलेटर का उपयोग किया जाता है।

### पूर्ण सिस्टम इम्युलेशन

[Firmadyne](https://github.com/firmadyne/firmadyne), [Firmware Analysis Toolkit](https://github.com/attify/firmware-analysis-toolkit) और अन्य उपकरण, पूर्ण फर्मवेयर इम्युलेशन को सुविधाजनक बनाते हैं, प्रक्रिया को स्वचालित करते हैं और डायनामिक विश्लेषण में सहायक होते हैं।

## प्रैक्टिस में डायनामिक विश्लेषण

इस स्टेज पर, विश्लेषण के लिए एक वास्तविक या इम्युलेटेड डिवाइस वातावरण का उपयोग किया जाता है। ओएस और फाइल सिस्टम तक पहुंच बनाए रखना महत्वपूर्ण है। इम्युलेशन हार्डवेयर इंटरेक्शन को पूरी तरह से नकल नहीं कर सकती, जिससे कभी-कभी इम्युलेशन पुनरारंभ की आवश्यकता होती है। विश्लेषण को फाइल सिस्टम पर फिर से जाना चाहिए, उजागरित वेबपेज और नेटवर्क सेवाओं का शोध करना चाहिए, और बूटलोडर की कमजोरियों की खोज करनी चाहिए। फर्मवेयर अखंडता परीक्षण महत्वपूर्ण है ताकि संभावित पिछवाड़े की खोज की जा सके।

## रनटाइम विश्लेषण तकनीकें

रनटाइम विश्लेषण में, gdb-multiarch, Frida, और Ghidra जैसे उपकरणों का उपयोग करके प्रक्रिया या बाइनरी के संचालन वातावरण के साथ बातचीत करना, ब्रेकपॉइंट सेट करना और फजिंग और अन्य तकनीकों के माध्यम से खोजने के लिए विकल्पितताओं की पहचान करना शामिल है।

## बाइनरी शोषण और प्रमाण-ऑफ-कॉन्सेप्ट

पहचानी गई विकल्पितताओं के लिए एक प्रमाण-ऑफ-कॉन्सेप्ट (PoC) विकसित करने के लिए लक्ष्य आर्किटेक्चर और निचले स्तर की भाषाओं में प्रोग्रामिंग की गहरी समझ की आवश्यकता है। एम्बेडेड सिस्टम में बाइनरी रनटाइम सुरक्षा सुरक्षा उपाय दुर्लभ हैं, लेकिन जब मौजूद होते हैं, तो Return Oriented Programming (ROP) जैसी तकनीकें आवश्यक हो सकती हैं।

## फर्मवेयर विश्लेषण के लिए तैयार ऑपरेटिंग सिस्टम

[AttifyOS](https://github.com/adi0x90/attifyos) और [EmbedOS](https://github.com/scriptingxss/EmbedOS) जैसे ऑपरेटिंग सिस्टम फर्मवेयर सुरक्षा परीक्षण के लिए पूर्व-कॉन्फ़िगर किए गए वातावरण प्रदान करते हैं, जिन्हें आवश्यक उपकरणों से लोड किया गया है।

## फर्मवेयर का विश्लेषण करने के लिए तैयार ओएस

* [**AttifyOS**](https://github.com/adi0x90/attifyos): AttifyOS एक डिस्ट्रो है जो आपको इंटरनेट ऑफ थिंग्स (IoT) डिवाइस की सुरक्षा मूल्यांकन और पेनेट्रेशन टेस्टिंग करने में मदद करने के लिए निर्देशित है। यह आपको सभी आवश्यक उपकरणों से लोड किए गए पूर्व-कॉन्फ़िगर वातावरण प्रदान करके आपके बहुत समय बचाता है।
* [**EmbedOS**](https://github.com/scriptingxss/EmbedOS): यूबंटू 18.04 पर आधारित एम्बेडेड सुरक्षा परीक्षण ऑपरेटिंग सिस्टम जो फर्मवेयर सुरक्षा परीक्षण उपकरणों से पूर्व-लोड है।

## अभ्यास के लिए विकल्पित फर्मवेयर

फर्मवेयर में विकल्पितताओं की खोज करने के लिए, निम्नलिखित विकल्पित फर्मवेयर परियोजनाओं का उपयोग करें।

* OWASP IoTGoat
* [https://github.com/OWASP/IoTGoat](https://github.com/OWASP/IoTGoat)
* The Damn Vulnerable Router Firmware Project
* [https://github.com/praetorian-code/DVRF](https://github.com/praetorian-code/DVRF)
* Damn Vulnerable ARM Router (DVAR)
* [https://blog.exploitlab.net/2018/01/dvar-damn-vulnerable-arm-router.html](https://blog.exploitlab.net/2018/01/dvar-damn-vulnerable-arm-router.html)
* ARM-X
* [https://github.com/therealsaumil/armx#downloads](https://github.com/therealsaumil/armx#downloads)
* Azeria Labs VM 2.0
* [https://azeria-labs.com/lab-vm-2-0/](https://azeria-labs.com/lab-vm-2-0/)
* Damn Vulnerable IoT Device (DVID)
* [https://github.com/Vulcainreo/DVID](https://github.com/Vulcainreo/DVID)

## संदर्भ

* [https://scriptingxss.gitbook.io/firmware-security-testing-methodology/](https://scriptingxss.gitbook.io/firmware-security-testing-methodology/)
* [Practical IoT Hacking: The Definitive Guide to Attacking the Internet of Things](https://www.amazon.co.uk/Practical-IoT-Hacking-F-Chantzis/dp/1718500904)

## प्रशिक्षण और प्रमाण-पत्र

* [https://www.attify-store.com/products/offensive-iot-exploitation](https://www.attify-store.com/products/offensive-iot-exploitation)
