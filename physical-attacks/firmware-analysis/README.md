# फर्मवेयर विश्लेषण

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आप **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड** करने की इच्छा रखते हैं? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)

- प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)** का** अनुसरण करें।**

- **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके साझा करें।**

</details>

## परिचय

फर्मवेयर एक प्रकार का सॉफ़्टवेयर है जो एक उपकरण के हार्डवेयर के घटकों पर संचार और नियंत्रण प्रदान करता है। यह उपकरण को चलाने के लिए पहला कोड होता है। आमतौर पर, यह **ऑपरेटिंग सिस्टम को बूट करता है** और **विभिन्न हार्डवेयर कंपोनेंट्स के साथ संवाद करके** कार्यक्रमों के लिए बहुत विशिष्ट रनटाइम सेवाएं प्रदान करता है। अधिकांश, यदि नहीं सभी, इलेक्ट्रॉनिक उपकरणों में फर्मवेयर होता है।

उपकरण फर्मवेयर को ROM, EPROM या फ्लैश मेमोरी जैसे **नॉनवोलेटाइल मेमोरी** में संग्रहीत करते हैं।

यह महत्वपूर्ण है कि हम फर्मवेयर की **जांच** करें और फिर इसे **संशोधित** करने का प्रयास करें, क्योंकि हम इस प्रक्रिया के दौरान कई सुरक्षा समस्याओं को खोल सकते हैं।

## **जानकारी इकट्ठा करना और पूर्व जानकारी**

इस चरण में, लक्ष्य के बारे में जितनी संभव हो सके जानकारी इकट्ठा करें ताकि उसके सामान्य संरचना और आधारभूत प्रौद्योगिकी को समझा जा सके। निम्नलिखित को इकट्ठा करने का प्रयास करें:

* समर्थित सीपीयू आर्किटेक्चर(एस) 
* ऑपरेटिंग सिस्टम प्लेटफॉर्म
* बूटलोडर कॉन्फ़िगरेशन
* हार्डवेयर स्कीमैटिक्स
* डेटाशीट्स
* कोड की पंक्तियों (LoC) का अनुमान
* स्रोत कोड रेपोजिटरी स्थान
* थर्ड पार्टी कंपोनेंट्स
* ओपन सोर्स लाइसेंसेज (जैसे GPL)
* चेंजलॉग्स
* FCC आईडी
* डिज़ाइन और डेटा फ़्लो डायग्राम
* धमकी मॉडल
* पिछली पेनेट्रेशन टेस्टिंग रिपोर्ट्स
* बग ट्रैकिंग टिकट (जैसे Jira और बग बाउंटी प्लेटफॉर्म जैसे BugCrowd या HackerOne)

जहां संभव हो, ओपन सोर्स इंटेलिजेंस (OSINT) टूल और तकनीकों का उपयोग करके डेटा प्राप्त करें। यदि ओपन सोर्स सॉफ़्टवेयर का उपयोग किया जाता है, तो रेपोजिटरी डाउनलोड करें और कोड बेस के खिलाफ मैन्युअल और स्वचालित स्थिर विश्लेषण करें। कभी-कभी, ओपन सोर्स सॉफ़्टवेयर परियोजनाएं पहले से ही विक्रेताओं द्वारा प्रदान की जाने वाली मुफ्त स्थिर विश्लेषण उपकरणों का उपयोग करती हैं जो [Coverity Scan](https://scan.coverity.com) और [Semmle’s LGTM](https://lgtm.com/#explore) जैसे स्कैन परिणाम प्रदान करते हैं।

## फर्मवेयर प्राप्त करना

फर्मवेयर को डाउनलोड करने के लिए विभिन्न कठिनाई स्तरों के साथ विभिन्न तरीके हैं

* **सीधे** विकास टीम, निर्माता/विक्रेता या ग्राहक से
* निर्माता द्वारा प्रदान की गई वॉकथ्रू का उपयोग करके **शून्य से बनाएं**
* विक्रेता
```bash
file <bin>
strings -n8 <bin>
strings -tx <bin> #print offsets in hex
hexdump -C -n 512 <bin> > hexdump.out
hexdump -C <bin> | head # might find signatures in header
fdisk -lu <bin> #lists a drives partition and filesystems if multiple
```
यदि आप उन उपकरणों के साथ बहुत कुछ नहीं पाते हैं, तो `binwalk -E <bin>` के साथ छवि का **एंट्रोपी** जांचें, यदि एंट्रोपी कम है, तो यह संभवतः एन्क्रिप्ट नहीं है। यदि एंट्रोपी उच्च है, तो यह संभवतः एन्क्रिप्टेड है (या किसी तरह संपीड़ित है)।

इसके अलावा, आप इन उपकरणों का उपयोग करके **फर्मवेयर्ड में सम्मिलित फ़ाइलें निकाल सकते हैं**:

{% content-ref url="../../forensics/basic-forensic-methodology/partitions-file-systems-carving/file-data-carving-recovery-tools.md" %}
[file-data-carving-recovery-tools.md](../../forensics/basic-forensic-methodology/partitions-file-systems-carving/file-data-carving-recovery-tools.md)
{% endcontent-ref %}

या [**binvis.io**](https://binvis.io/#/) ([code](https://code.google.com/archive/p/binvis/)) का उपयोग करके फ़ाइल की जांच करें।

### फ़ाइलसिस्टम प्राप्त करना

पिछले टिप्पणीत उपकरणों जैसे `binwalk -ev <bin>` के साथ आपको **फ़ाइलसिस्टम को निकालने** में सक्षम होना चाहिए।\
बिनवॉक आमतौर पर इसे एक **फ़ोल्डर में निकालता है जिसका नाम फ़ाइलसिस्टम प्रकार के रूप में होता है**, जो आमतौर पर निम्नलिखित में से एक होता है: squashfs, ubifs, romfs, rootfs, jffs2, yaffs2, cramfs, initramfs.

#### मैन्युअल फ़ाइलसिस्टम निकालना

कभी-कभी, बिनवॉक **फ़ाइलसिस्टम के मैजिक बाइट को अपने संकेतों में नहीं रखेगा**। इन मामलों में, बाइनरी से आपको फ़ाइलसिस्टम के ऑफ़सेट का पता लगाने और संपीड़ित फ़ाइलसिस्टम को बाइनरी से **हटाकर** फ़ाइलसिस्टम को इसके प्रकार के अनुसार **मैन्युअल रूप से निकालें**।
```
$ binwalk DIR850L_REVB.bin

DECIMAL HEXADECIMAL DESCRIPTION
----------------------------------------------------------------------------- ---

0 0x0 DLOB firmware header, boot partition: """"dev=/dev/mtdblock/1""""
10380 0x288C LZMA compressed data, properties: 0x5D, dictionary size: 8388608 bytes, uncompressed size: 5213748 bytes
1704052 0x1A0074 PackImg section delimiter tag, little endian size: 32256 bytes; big endian size: 8257536 bytes
1704084 0x1A0094 Squashfs filesystem, little endian, version 4.0, compression:lzma, size: 8256900 bytes, 2688 inodes, blocksize: 131072 bytes, created: 2016-07-12 02:28:41
```
निम्नलिखित **dd कमांड** चलाएं और Squashfs फ़ाइलसिस्टम को खोदें।
```
$ dd if=DIR850L_REVB.bin bs=1 skip=1704084 of=dir.squashfs

8257536+0 records in

8257536+0 records out

8257536 bytes (8.3 MB, 7.9 MiB) copied, 12.5777 s, 657 kB/s
```
वैकल्पिक रूप से, निम्नलिखित कमांड भी चलाया जा सकता है।

`$ dd if=DIR850L_REVB.bin bs=1 skip=$((0x1A0094)) of=dir.squashfs`

* स्क्वैशफ्स के लिए (उपरोक्त उदाहरण में उपयोग किया जाता है)

`$ unsquashfs dir.squashfs`

फ़ाइलें "`squashfs-root`" निर्देशिका में होंगी।

* CPIO आर्काइव फ़ाइलें

`$ cpio -ivd --no-absolute-filenames -F <bin>`

* JFFS2 फ़ाइल सिस्टम के लिए

`$ jefferson rootfsfile.jffs2`

* NAND फ़्लैश के साथ UBIFS फ़ाइल सिस्टम के लिए

`$ ubireader_extract_images -u UBI -s <start_offset> <bin>`

`$ ubidump.py <bin>`

### फ़ाइल सिस्टम का विश्लेषण

अब जब आपके पास फ़ाइल सिस्टम है, तो आपको बुरी अभ्यासों की तलाश करनी शुरू करनी चाहिए, जैसे:

* पुराने **असुरक्षित नेटवर्क डेमन** जैसे telnetd (कभी-कभी निर्माता बाइनरी को छिपाने के लिए नाम बदल देते हैं)
* **हार्डकोड क्रेडेंशियल** (उपयोगकर्ता नाम, पासवर्ड, API कुंजी, SSH कुंजी, और बैकडोर वेरिएंट)
* **हार्डकोड API** एंडपॉइंट और बैकएंड सर्वर विवरण
* **अपडेट सर्वर कार्यक्षमता** जो एंट्री प्वाइंट के रूप में उपयोग किया जा सकता है
* **अनकंपाइल्ड कोड और स्टार्ट अप स्क्रिप्ट** की समीक्षा करें दूरस्थ कोड निष्पादन के लिए
* **कंपाइल किए गए बाइनरी** को निकालें और भविष्य के चरणों के लिए एक डिसएसेंबलर के साथ ऑफ़लाइन विश्लेषण के लिए उपयोग करें

फ़र्मवेयर में खोजने के लिए **रोचक चीजें**:

* etc/shadow और etc/passwd
* etc/ssl निर्देशिका को बाहर निकालें
* .pem, .crt आदि जैसी SSL संबंधित फ़ाइलें खोजें
* कॉन्फ़िगरेशन फ़ाइलें खोजें
* स्क्रिप्ट फ़ाइलें खोजें
* अन्य .bin फ़ाइलें खोजें
* व्यवस्थापक, पासवर्ड, दूरस्थ, AWS कुंजी आदि जैसे कीवर्ड खोजें
* IoT उपकरणों पर उपयोग किए जाने वाले सामान्य वेब सर्वर खोजें
* ssh, tftp, dropbear आदि जैसे सामान्य बाइनरी खोजें
* प्रतिबंधित c फ़ंक्शन खोजें
* सामान्य कमांड इंजेक्शन संक्रमित फ़ंक्शन खोजें
* URL, ईमेल पता और IP पता खोजें
* और अधिक...

इस तरह की जानकारी के लिए खोज करने वाले उपकरण (हालांकि, आपको हमेशा **फ़ाइल सिस्टम की संरचना को मैन्युअल रूप से देखना चाहिए**, उपकरण आपको **छिपे हुए चीजें** खोजने में मदद कर सकते हैं):

* [**LinPEAS**](https://github.com/carlospolop/PEASS-ng)**:** यह अद्वितीय बैश स्क्रिप्ट है जो इस मामले में फ़ाइल सिस्टम में **संवेदनशील जानकारी** खोजने के लिए उपयोगी है। बस **फ़ाइल सिस्टम में च्रूट बनाएं और इसे चलाएं**।
* [**Firmwalker**](https://github.com/craigz28/firmwalker)**:** संभावित संवेदनशील जानकारी की खोज करने के लिए बैश स्क्रिप्ट
* [**The Firmware Analysis and Comparison Tool (FACT)**](https://github.com/fkie-cad/FACT\_core):
* ऑपरेटिंग सिस्टम, सीपीयू आर्किटेक्चर, और तृतीय-पक्ष संघ के सॉफ़्टवेयर संघ की पहचान
* इमेज से फ़ाइल सिस्टम (ओं) की निकालाई
* प्रमाणपत्र और निजी कुंजी की पहचान
* CWE के साथ मैपिंग करने वाले कमजोर कार्यान्वयन की पहचान
* संक्रमितता की खोज के लिए फ़ीड और हस्ताक्षर आधारित डिटेक्शन
* मूल्यांकन का मूल्यांकन (डिफ़) फ़ाइरवेयर संस्करण और फ़ाइलें
* QEMU का उपयोग करके फ़ाइल सिस्टम बाइनरी के उपयोगकर्ता मोड अनुकरण
* NX, DEP, ASLR, स्टैक कैनरी, RELRO, और FORTIFY\_SOURCE जैसे बाइनरी संरक्षण की पहचान
* REST API
* और अधिक...
* [**FwAnalyzer**](https://github.com/cruise-automation/fwanalyzer): FwAnalyzer एक उपकरण है जो (ext2/3/4), FAT/VFat, SquashFS, UBIFS फ़ाइल सिस्टम इमेज, cpio आर्काइव, और निर्देशिका सामग्री का विश्लेषण करने के लिए एक सेट के साथ कॉन्फ़िगर किए गए नियमों का उपयोग करता है।
* [**ByteSweep**](https://gitlab.com/bytesweep/bytesweep): एक मुफ्त सॉफ़्टवेयर IoT फ़र्मवेयर सुरक्षा विश्लेषण उपकरण
* [
```bash
file ./squashfs-root/bin/busybox
./squashfs-root/bin/busybox: ELF 32-bit MSB executable, MIPS, MIPS32 rel2 version 1 (SYSV), dynamically linked, interpreter /lib/ld-uClibc.so.0, stripped
```
अब आप **QEMU** का उपयोग करके **busybox** executable को **emulate** कर सकते हैं।
```bash
sudo apt-get install qemu qemu-user qemu-user-static qemu-system-arm qemu-system-mips qemu-system-x86 qemu-utils
```
क्योंकि एक्सीक्यूटेबल **MIPS** के लिए कंपाइल किया जाता है और **बिग-एंडियन** बाइट आदेश का पालन करता है, हम **QEMU** का **`qemu-mips`** इम्युलेटर उपयोग करेंगे। **लिटिल-एंडियन** एक्सीक्यूटेबल को इम्युलेट करने के लिए, हमें `el` सफ़िक्स के साथ इम्युलेटर का चयन करना होगा (`qemu-mipsel`)।
```bash
qemu-mips -L ./squashfs-root/ ./squashfs-root/bin/ls
100              100.7z           15A6D2.squashfs  squashfs-root    squashfs-root-0
```
#### आरएम उदाहरण

```html
<details>
<summary>Click to expand!</summary>

##### Step 1: Dumping the firmware

To analyze the firmware of an ARM device, we first need to dump it. This can be done using various methods such as JTAG, UART, or SPI. Once the firmware is dumped, we can proceed with the analysis.

##### Step 2: Identifying the firmware type

Next, we need to identify the type of firmware we have dumped. This can be done by analyzing the file headers or using tools like `binwalk` or `firmware-mod-kit`.

##### Step 3: Extracting the firmware

Once we have identified the firmware type, we can extract its contents using tools like `binwalk`, `firmware-mod-kit`, or `dd`.

##### Step 4: Analyzing the firmware

Now that we have extracted the firmware, we can start analyzing it. This involves examining the file system, searching for vulnerabilities, and reverse engineering any binaries present.

##### Step 5: Patching or modifying the firmware

If we find any vulnerabilities or want to modify the firmware, we can patch or modify it accordingly. This can be done using tools like `firmware-mod-kit` or by manually editing the binaries.

##### Step 6: Flashing the modified firmware

Finally, we can flash the modified firmware back onto the ARM device. This can be done using methods like JTAG, UART, or SPI.

</details>
```

आरएम उदाहरण

```html
<details>
<summary>विस्तारित करने के लिए क्लिक करें!</summary>

##### चरण 1: फर्मवेयर को डंप करना

आरएम डिवाइस के फर्मवेयर का विश्लेषण करने के लिए, हमें पहले इसे डंप करना होगा। इसके लिए JTAG, UART या SPI जैसे विभिन्न तरीकों का उपयोग किया जा सकता है। एक बार फर्मवेयर डंप हो जाने के बाद, हम विश्लेषण के साथ आगे बढ़ सकते हैं।

##### चरण 2: फर्मवेयर प्रकार की पहचान करना

अगले, हमें डंप किए गए फर्मवेयर के प्रकार की पहचान करनी होगी। इसे फ़ाइल हैडर का विश्लेषण करके या `binwalk` या `firmware-mod-kit` जैसे उपकरणों का उपयोग करके किया जा सकता है।

##### चरण 3: फर्मवेयर को निकालना

एक बार जब हमने फर्मवेयर के प्रकार की पहचान कर ली है, हम उसकी सामग्री को `binwalk`, `firmware-mod-kit` या `dd` जैसे उपकरणों का उपयोग करके निकाल सकते हैं।

##### चरण 4: फर्मवेयर का विश्लेषण करना

अब जब हमने फर्मवेयर को निकाल लिया है, हम उसे विश्लेषण करना शुरू कर सकते हैं। इसमें फ़ाइल सिस्टम की जांच, संकटों की खोज और मौजूद बाइनरी का प्रतिपादन शामिल होता है।

##### चरण 5: फर्मवेयर को पैच या संशोधित करना

यदि हमें कोई संकट मिलता है या फर्मवेयर को संशोधित करना चाहते हैं, तो हम उसे अनुसार पैच या संशोधित कर सकते हैं। इसे `firmware-mod-kit` जैसे उपकरणों का उपयोग करके या बाइनरी को मैन्युअल रूप से संपादित करके किया जा सकता है।

##### चरण 6: संशोधित फर्मवेयर को फ्लैश करना

अंत में, हम संशोधित फर्मवेयर को आरएम डिवाइस पर फ्लैश कर सकते हैं। इसे JTAG, UART या SPI जैसे तरीकों का उपयोग करके किया जा सकता है।

</details>
```
```bash
file bin/busybox
bin/busybox: ELF 32-bit LSB executable, ARM, EABI5 version 1 (SYSV), dynamically linked, interpreter /lib/ld-musl-armhf.so.1, no section header
```
एम्युलेशन:
```bash
qemu-arm -L ./squashfs-root/ ./squashfs-root/bin/ls
1C00000.squashfs  B80B6C            C41DD6.xz         squashfs-root     squashfs-root-0
```
### पूर्ण सिस्टम नकल

कई टूल हैं, जो सामान्य रूप से **qemu** पर आधारित हैं, जो आपको पूर्ण फर्मवेयर की नकल बनाने की अनुमति देंगे:

* [**https://github.com/firmadyne/firmadyne**](https://github.com/firmadyne/firmadyne)**:**
* आपको कई चीजें स्थापित करनी होंगी, postgres को कॉन्फ़िगर करना होगा, फिर एक्सट्रैक्टर.py स्क्रिप्ट को चलाना होगा ताकि फर्मवेयर को निकाल सकें, गेटआर्च.sh स्क्रिप्ट का उपयोग करके आर्किटेक्चर प्राप्त करें। फिर, tar2db.py और makeImage.sh स्क्रिप्ट का उपयोग करके डेटाबेस में निकाले गए इमेज से जानकारी संग्रहित करें और एक QEMU इमेज उत्पन्न करें जिसे हम नकल बना सकते हैं। फिर, नेटवर्क इंटरफेस प्राप्त करने के लिए inferNetwork.sh स्क्रिप्ट का उपयोग करें, और अंत में, जो ऑटोमेटिक रूप से ./scratch/1/folder में बनाया जाता है, उसे चलाएं run.sh स्क्रिप्ट का उपयोग करें।
* [**https://github.com/attify/firmware-analysis-toolkit**](https://github.com/attify/firmware-analysis-toolkit)**:**
* यह टूल firmadyne पर निर्भर करता है और firmadyne का उपयोग करके फर्मवेयर की नकल बनाने की प्रक्रिया को स्वचालित करता है। इसे उपयोग करने से पहले आपको `fat.config` को कॉन्फ़िगर करना होगा: `sudo python3 ./fat.py IoTGoat-rpi-2.img --qemu 2.5.0`
* [**https://github.com/therealsaumil/emux**](https://github.com/therealsaumil/emux)
* [**https://github.com/getCUJO/MIPS-X**](https://github.com/getCUJO/MIPS-X)
* [**https://github.com/qilingframework/qiling#qltool**](https://github.com/qilingframework/qiling#qltool)

## **गतिशील विश्लेषण**

इस चरण में, आपके पास या तो एक फर्मवेयर को हमला करने के लिए चल रही डिवाइस होनी चाहिए या फिर एक फर्मवेयर को नकल बनाने के लिए उसे हमला करने के लिए उपयोग किया जा रहा होना चाहिए। हर हाल में, यह सुनिश्चित करना बहुत आवश्यक है कि आपके पास भी **एक शैल में शैल और फाइलसिस्टम में शैल** हो।

ध्यान दें कि कभी-कभी यदि आप फर्मवेयर की नकल बना रहे हैं तो **नकल में कुछ गतिविधियाँ विफल हो सकती हैं** और आपको इसे फिर से नकल बनाने की आवश्यकता हो सकती है। उदाहरण के लिए, एक वेब एप्लिकेशन को आमतौर पर उस डिवाइस की जानकारी की आवश्यकता होती है जिसके साथ मूल डिवाइस एकीकृत है लेकिन नकल नकल नहीं कर रही है।

आपको **फाइलसिस्टम की पुनर्जांच करनी चाहिए** जैसा कि हमने पहले किया है क्योंकि चल रहे माहौल में नई जानकारी उपलब्ध हो सकती है।

यदि **वेबपेज** उजागर हैं, तो कोड पढ़ें और उनका उपयोग करने के लिए उन्हें जांचें। हैकट्रिक्स में आपको विभिन्न वेब हैकिंग तकनीकों के बारे में बहुत सारी जानकारी मिलेगी।

यदि **नेटवर्क सेवाएं** उजागर हैं, तो आपको उन पर हमला करने का प्रयास करना चाहिए। हैकट्रिक्स में आपको विभिन्न नेटवर्क सेवा हैकिंग तकनीकों के बारे में बहुत सारी जानकारी मिलेगी। आप नेटवर्क और प्रोटोकॉल **फ़ज़र्स** जैसे [Mutiny](https://github.com/Cisco-Talos/mutiny-fuzzer), [boofuzz](https://github.com/jtpereyda/boofuzz), और [kitty](https://github.com/cisco-sas/kitty) के साथ उन्हें फज़ करने की कोशिश कर सकते हैं।

आपको देखना चाहिए कि क्या आप **बूटलोडर पर हमला** करके एक रूट शैल प्राप्त कर सकते हैं:

{% content-ref url="bootloader-testing.md" %}
[bootloader-testing.md](bootloader-testing.md)
{% endcontent-ref %}

आपको यह जांचना चाहिए कि क्या उपकरण को कोई भी प्रकार का **फर्मवेयर अखंडता परीक्षण** कर रहा है, यदि नहीं तो यह हमलावर्ती को अखंडित फर्मवेयर प्रदान करने, उन्हें उनके पास रखने या यदि कोई फर्मवेयर अद्यतन संवर्धन है तो उन्हें दूरस्थ तक डिप्लॉय करने की अनुमति देने के लिए हमलाव
## वल्नरबल फर्मवेयर का अभ्यास करने के लिए

फर्मवेयर में खोजने के लिए वल्नरबल फर्मवेयर परियोजनाओं का उपयोग करें, जैसे कि निम्नलिखित वल्नरबल फर्मवेयर परियोजनाएं।

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

## प्रशिक्षण और प्रमाणपत्र

* [https://www.attify-store.com/products/offensive-iot-exploitation](https://www.attify-store.com/products/offensive-iot-exploitation)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की अनुमति चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)

- प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स साझा करें, [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके।**

</details>
