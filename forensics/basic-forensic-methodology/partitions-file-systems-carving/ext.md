<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग प्राप्त करें**](https://peass.creator-spring.com)
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) **का पालन करें**.
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें.

</details>


# Ext - एक्सटेंडेड फाइलसिस्टम

**Ext2** सबसे आम फाइलसिस्टम है **नॉन-जर्नलिंग** पार्टीशन्स के लिए (**पार्टीशन्स जो ज्यादा नहीं बदलते**) जैसे कि बूट पार्टीशन। **Ext3/4** **जर्नलिंग** हैं और आमतौर पर **बाकी पार्टीशन्स** के लिए इस्तेमाल किए जाते हैं।

फाइलसिस्टम में सभी ब्लॉक ग्रुप्स का आकार समान होता है और ये क्रमिक रूप से संग्रहीत होते हैं। इससे कर्नेल को डिस्क में ब्लॉक ग्रुप के स्थान को उसके इंटीजर इंडेक्स से आसानी से निकालने में मदद मिलती है।

प्रत्येक ब्लॉक ग्रुप में निम्नलिखित जानकारी होती है:

* फाइलसिस्टम के सुपरब्लॉक की एक प्रति
* ब्लॉक ग्रुप डिस्क्रिप्टर्स की एक प्रति
* एक डेटा ब्लॉक बिटमैप जो ग्रुप के अंदर मुक्त ब्लॉक्स की पहचान के लिए इस्तेमाल होता है
* एक इनोड बिटमैप, जो ग्रुप के अंदर मुक्त इनोड्स की पहचान के लिए इस्तेमाल होता है
* इनोड टेबल: यह लगातार ब्लॉक्स की एक श्रृंखला होती है, प्रत्येक में एक निर्धारित संख्या के इनोड्स होते हैं। सभी इनोड्स का आकार समान होता है: 128 बाइट्स। एक 1,024 बाइट ब्लॉक में 8 इनोड्स होते हैं, जबकि एक 4,096-बाइट ब्लॉक में 32 इनोड्स होते हैं। ध्यान दें कि Ext2 में, इनोड नंबर और संबंधित ब्लॉक नंबर के बीच डिस्क पर मैपिंग स्टोर करने की आवश्यकता नहीं होती क्योंकि बाद वाला मान ब्लॉक ग्रुप नंबर और इनोड टेबल के अंदर सापेक्ष स्थिति से निकाला जा सकता है। उदाहरण के लिए, मान लीजिए कि प्रत्येक ब्लॉक ग्रुप में 4,096 इनोड्स होते हैं और हमें इनोड 13,021 के डिस्क पर पते को जानना है। इस मामले में, इनोड तीसरे ब्लॉक ग्रुप का हिस्सा है और इसका डिस्क पता संबंधित इनोड टेबल की 733वीं प्रविष्टि में संग्रहीत है। जैसा कि आप देख सकते हैं, इनोड नंबर सिर्फ एक कुंजी है जिसका उपयोग Ext2 रूटीन्स द्वारा डिस्क पर उचित इनोड डिस्क्रिप्टर को जल्दी से पुनः प्राप्त करने के लिए किया जाता है
* डेटा ब्लॉक्स, जिसमें फाइलें होती हैं। कोई भी ब्लॉक जिसमें कोई महत्वपूर्ण जानकारी नहीं होती, उसे मुक्त कहा जाता है।

![](<../../../.gitbook/assets/image (406).png>)

## Ext वैकल्पिक फीचर्स

**फीचर्स प्रभावित करते हैं** डेटा कहाँ स्थित है, **कैसे** डेटा इनोड्स में संग्रहीत होता है और कुछ विश्लेषण के लिए **अतिरिक्त मेटाडेटा** प्रदान कर सकते हैं, इसलिए Ext में फीचर्स महत्वपूर्ण हैं।

Ext में वैकल्पिक फीचर्स होते हैं जिन्हें आपका OS समर्थन कर सकता है या नहीं, इसकी 3 संभावनाएं हैं:

* संगत
* असंगत
* केवल पढ़ने के लिए संगत: इसे माउंट किया जा सकता है लेकिन लिखने के लिए नहीं

यदि **असंगत** फीचर्स हैं तो आप फाइलसिस्टम को माउंट नहीं कर पाएंगे क्योंकि OS को डेटा तक पहुँचने का तरीका पता नहीं होगा।

{% hint style="info" %}
एक संदिग्ध हमलावर के पास गैर-मानक एक्सटेंशन हो सकते हैं
{% endhint %}

**कोई भी उपयोगिता** जो **सुपरब्लॉक** को पढ़ती है, वह एक **Ext फाइलसिस्टम** के **फीचर्स** को इंगित कर सकती है, लेकिन आप `file -sL /dev/sd*` का भी उपयोग कर सकते हैं।

## सुपरब्लॉक

सुपरब्लॉक शुरुआत से पहले 1024 बाइट्स है और यह प्रत्येक ग्रुप के पहले ब्लॉक में दोहराया जाता है और इसमें शामिल हैं:

* ब्लॉक आकार
* कुल ब्लॉक्स
* प्रति ब्लॉक ग्रुप ब्लॉक्स
* पहले ब्लॉक ग्रुप से पहले आरक्षित ब्लॉक्स
* कुल इनोड्स
* प्रति ब्लॉक ग्रुप इनोड्स
* वॉल्यूम नाम
* अंतिम लिखने का समय
* अंतिम माउंट समय
* पथ जहाँ फाइल सिस्टम अंतिम बार माउंट किया गया था
* फाइलसिस्टम की स्थिति (साफ?)

एक Ext फाइलसिस्टम फाइल से इस जानकारी को प्राप्त करना संभव है:
```bash
fsstat -o <offsetstart> /pat/to/filesystem-file.ext
#You can get the <offsetstart> with the "p" command inside fdisk
```
आप मुफ्त GUI एप्लिकेशन का भी उपयोग कर सकते हैं: [https://www.disk-editor.org/index.html](https://www.disk-editor.org/index.html)\
या आप **python** का उपयोग करके सुपरब्लॉक जानकारी प्राप्त कर सकते हैं: [https://pypi.org/project/superblock/](https://pypi.org/project/superblock/)

## inodes

**inodes** में **blocks** की सूची होती है जो एक **file** के वास्तविक **data** को **contains** करती है।\
यदि file बड़ी है, तो एक inode में **other inodes** की ओर संकेत करने वाले pointers हो सकते हैं जो blocks/more inodes की ओर संकेत करते हैं जिसमें file data होता है।

![](<../../../.gitbook/assets/image (416).png>)

**Ext2** और **Ext3** में inodes का आकार **128B** होता है, **Ext4** वर्तमान में **156B** का उपयोग करता है लेकिन भविष्य के विस्तार की अनुमति देने के लिए डिस्क पर **256B** आवंटित करता है।

Inode संरचना:

| Offset | Size | Name              | DescriptionF                                     |
| ------ | ---- | ----------------- | ------------------------------------------------ |
| 0x0    | 2    | File Mode         | File mode और type                                |
| 0x2    | 2    | UID               | Owner ID के Lower 16 bits                        |
| 0x4    | 4    | Size Il           | File size के Lower 32 bits                       |
| 0x8    | 4    | Atime             | Access time in seconds since epoch               |
| 0xC    | 4    | Ctime             | Change time in seconds since epoch               |
| 0x10   | 4    | Mtime             | Modify time in seconds since epoch               |
| 0x14   | 4    | Dtime             | Delete time in seconds since epoch               |
| 0x18   | 2    | GID               | Group ID के Lower 16 bits                        |
| 0x1A   | 2    | Hlink count       | Hard link count                                  |
| 0xC    | 4    | Blocks Io         | Block count के Lower 32 bits                     |
| 0x20   | 4    | Flags             | Flags                                            |
| 0x24   | 4    | Union osd1        | Linux: I version                                 |
| 0x28   | 69   | Block\[15]        | 15 points to data block                         |
| 0x64   | 4    | Version           | NFS के लिए File version                         |
| 0x68   | 4    | File ACL low      | Extended attributes (ACL, etc) के Lower 32 bits  |
| 0x6C   | 4    | File size hi      | File size के Upper 32 bits (ext4 only)           |
| 0x70   | 4    | Obsolete fragment | An obsoleted fragment address                    |
| 0x74   | 12   | Osd 2             | Second operating system dependent union          |
| 0x74   | 2    | Blocks hi         | Block count के Upper 16 bits                     |
| 0x76   | 2    | File ACL hi       | Extended attributes (ACL, etc.) के Upper 16 bits |
| 0x78   | 2    | UID hi            | Owner ID के Upper 16 bits                        |
| 0x7A   | 2    | GID hi            | Group ID के Upper 16 bits                        |
| 0x7C   | 2    | Checksum Io       | Inode checksum के Lower 16 bits                  |

"Modify" वह timestamp है जब file के _content_ में आखिरी बार परिवर्तन किया गया था। इसे अक्सर "_mtime_" कहा जाता है।\
"Change" वह timestamp है जब file के _inode_ में आखिरी बार परिवर्तन किया गया था, जैसे कि permissions, ownership, file name, और hard links की संख्या में बदलाव करना। इसे अक्सर "_ctime_" कहा जाता है।

Inode संरचना विस्तारित (Ext4):

| Offset | Size | Name         | Description                                 |
| ------ | ---- | ------------ | ------------------------------------------- |
| 0x80   | 2    | Extra size   | मानक 128 से कितने bytes अधिक उपयोग किए गए हैं |
| 0x82   | 2    | Checksum hi  | Inode checksum के Upper 16 bits             |
| 0x84   | 4    | Ctime extra  | Change time extra bits                      |
| 0x88   | 4    | Mtime extra  | Modify time extra bits                      |
| 0x8C   | 4    | Atime extra  | Access time extra bits                      |
| 0x90   | 4    | Crtime       | File create time (seconds since epoch)      |
| 0x94   | 4    | Crtime extra | File create time extra bits                 |
| 0x98   | 4    | Version hi   | Version के Upper 32 bits                    |
| 0x9C   |      | Unused       | भविष्य के विस्तार के लिए आरक्षित स्थान        |

विशेष inodes:

| Inode | Special Purpose                                      |
| ----- | ---------------------------------------------------- |
| 0     | कोई ऐसा inode नहीं, नंबरिंग 1 से शुरू होती है         |
| 1     | Defective block list                                 |
| 2     | Root directory                                       |
| 3     | User quotas                                          |
| 4     | Group quotas                                         |
| 5     | Boot loader                                          |
| 6     | Undelete directory                                   |
| 7     | Reserved group descriptors (for resizing filesystem) |
| 8     | Journal                                              |
| 9     | Exclude inode (for snapshots)                        |
| 10    | Replica inode                                        |
| 11    | पहला non-reserved inode (अक्सर lost + found)         |

{% hint style="info" %}
ध्यान दें कि creation time केवल Ext4 में दिखाई देता है।
{% endhint %}

Inode number जानकर आप आसानी से उसका index पा सकते हैं:

* **Block group** जहां एक inode संबंधित है: (Inode number - 1) / (Inodes per group)
* **उसके group के अंदर Index**: (Inode number - 1) mod(Inodes/groups)
* **inode table में Offset**: Inode number \* (Inode size)
* "-1" इसलिए है क्योंकि inode 0 undefined (उपयोग में नहीं) है
```bash
ls -ali /bin | sort -n #Get all inode numbers and sort by them
stat /bin/ls #Get the inode information of a file
istat -o <start offset> /path/to/image.ext 657103 #Get information of that inode inside the given ext file
icat -o <start offset> /path/to/image.ext 657103 #Cat the file
```
File Mode

| Number | Description                                                                                         |
| ------ | --------------------------------------------------------------------------------------------------- |
| **15** | **Reg/Slink-13/Socket-14**                                                                          |
| **14** | **Directory/Block Bit 13**                                                                          |
| **13** | **Char Device/Block Bit 14**                                                                        |
| **12** | **FIFO**                                                                                            |
| 11     | Set UID                                                                                             |
| 10     | Set GID                                                                                             |
| 9      | Sticky Bit (इसके बिना, कोई भी व्यक्ति जिसके पास एक डायरेक्टरी पर Write & exec अनुमतियां हैं, फाइलों को हटा सकता है और नाम बदल सकता है)  |
| 8      | Owner Read                                                                                          |
| 7      | Owner Write                                                                                         |
| 6      | Owner Exec                                                                                          |
| 5      | Group Read                                                                                          |
| 4      | Group Write                                                                                         |
| 3      | Group Exec                                                                                          |
| 2      | Others Read                                                                                         |
| 1      | Others Write                                                                                        |
| 0      | Others Exec                                                                                         |

बोल्ड बिट्स (12, 13, 14, 15) फाइल के प्रकार को दर्शाते हैं (एक डायरेक्टरी, सॉकेट...) केवल एक बोल्ड विकल्प मौजूद हो सकता है।

Directories

| Offset | Size | Name      | Description                                                                                                                                                  |
| ------ | ---- | --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 0x0    | 4    | Inode     |                                                                                                                                                              |
| 0x4    | 2    | Rec len   | Record length                                                                                                                                                |
| 0x6    | 1    | Name len  | Name length                                                                                                                                                  |
| 0x7    | 1    | File type | <p>0x00 अज्ञात<br>0x01 नियमित</p><p>0x02 निर्देशिका</p><p>0x03 Char device</p><p>0x04 Block device</p><p>0x05 FIFO</p><p>0x06 Socket</p><p>0x07 Sym link</p> |
| 0x8    |      | Name      | Name string (255 अक्षरों तक)                                                                                                                               |

**प्रदर्शन बढ़ाने के लिए, Root hash Directory blocks का उपयोग किया जा सकता है।**

**Extended Attributes**

इसे स्टोर किया जा सकता है

* inodes के बीच अतिरिक्त स्थान में (256 - inode size, आमतौर पर = 100)
* एक डेटा ब्लॉक जिसे inode में file_acl द्वारा संकेत किया गया है

इसका उपयोग किसी भी चीज़ को यूजर एट्रिब्यूट के रूप में स्टोर करने के लिए किया जा सकता है अगर नाम "user" से शुरू होता है। इस तरह डेटा छिपाया जा सकता है।

Extended Attributes Entries

| Offset | Size | Name         | Description                                                                                                                                                                                                        |
| ------ | ---- | ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 0x0    | 1    | Name len     | एट्रिब्यूट नाम की लंबाई                                                                                                                                                                                           |
| 0x1    | 1    | Name index   | <p>0x0 = कोई प्रीफिक्स नहीं</p><p>0x1 = user. प्रीफिक्स</p><p>0x2 = system.posix_acl_access</p><p>0x3 = system.posix_acl_default</p><p>0x4 = trusted.</p><p>0x6 = security.</p><p>0x7 = system.</p><p>0x8 = system.richacl</p> |
| 0x2    | 2    | Value offs   | पहले inode प्रविष्टि या ब्लॉक की शुरुआत से ऑफसेट                                                                                                                                                                    |
| 0x4    | 4    | Value blocks | डिस्क ब्लॉक जहां मूल्य संग्रहीत है या इस ब्लॉक के लिए शून्य                                                                                                                                                     |
| 0x8    | 4    | Value size   | मूल्य की लंबाई                                                                                                                                                                                                    |
| 0xC    | 4    | Hash         | ब्लॉक में एट्रिब्यूट्स के लिए हैश या अगर inode में है तो शून्य                                                                                                                                                  |
| 0x10   |      | Name         | एट्रिब्यूट नाम बिना ट्रेलिंग NULL के                                                                                                                                                                             |
```bash
setfattr -n 'user.secret' -v 'This is a secret' file.txt #Save a secret using extended attributes
getfattr file.txt #Get extended attribute names of a file
getdattr -n 'user.secret' file.txt #Get extended attribute called "user.secret"
```
## फाइल सिस्टम दृश्य

फाइल सिस्टम की सामग्री देखने के लिए, आप **निःशुल्क उपकरण का उपयोग कर सकते हैं**: [https://www.disk-editor.org/index.html](https://www.disk-editor.org/index.html)\
या आप इसे अपने लिनक्स में `mount` कमांड का उपयोग करके माउंट कर सकते हैं।

[https://piazza.com/class\_profile/get\_resource/il71xfllx3l16f/inz4wsb2m0w2oz#:\~:text=The%20Ext2%20file%20system%20divides,lower%20average%20disk%20seek%20time.](https://piazza.com/class\_profile/get\_resource/il71xfllx3l16f/inz4wsb2m0w2oz#:\~:text=The%20Ext2%20file%20system%20divides,lower%20average%20disk%20seek%20time.)


<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) **का अनुसरण करें**.
* **अपनी हैकिंग ट्रिक्स साझा करें, HackTricks** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके।

</details>
