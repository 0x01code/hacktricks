# macOS Universal binaries & Mach-O Format

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family) का पता लगाएं
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें और PR जमा करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को फ़ॉलो करें।**

</details>

## मूलभूत जानकारी

Mac OS बाइनरी आमतौर पर **यूनिवर्सल बाइनरी** के रूप में कंपाइल होते हैं। एक **यूनिवर्सल बाइनरी** में एक ही फ़ाइल में **एकाधिक आर्किटेक्चर का समर्थन कर सकती है**।

ये बाइनरी **Mach-O संरचना** का पालन करते हैं जो मूल रूप से निम्नलिखित से मिलकर बनी होती है:

* हैडर
* लोड कमांड
* डेटा

![](<../../../.gitbook/assets/image (559).png>)

## फैट हैडर

फ़ाइल को खोजें: `mdfind fat.h | grep -i mach-o | grep -E "fat.h$"`

<pre class="language-c"><code class="lang-c"><strong>#define FAT_MAGIC	0xcafebabe
</strong><strong>#define FAT_CIGAM	0xbebafeca	/* NXSwapLong(FAT_MAGIC) */
</strong>
struct fat_header {
<strong>	uint32_t	magic;		/* FAT_MAGIC or FAT_MAGIC_64 */
</strong><strong>	uint32_t	nfat_arch;	/* number of structs that follow */
</strong>};

struct fat_arch {
cpu_type_t	cputype;	/* cpu specifier (int) */
cpu_subtype_t	cpusubtype;	/* machine specifier (int) */
uint32_t	offset;		/* file offset to this object file */
uint32_t	size;		/* size of this object file */
uint32_t	align;		/* alignment as a power of 2 */
};
</code></pre>

हैडर में **मैजिक** बाइट्स होते हैं जिनके बाद फ़ाइल में मौजूद **आर्किटेक्चर की संख्या** (`nfat_arch`) होती है और प्रत्येक आर्किटेक्चर के पास `fat_arch` संरचना होती है।

इसे निम्नलिखित के साथ जांचें:

<pre class="language-shell-session"><code class="lang-shell-session">% file /bin/ls
/bin/ls: Mach-O universal binary with 2 architectures: [x86_64:Mach-O 64-bit executable x86_64] [arm64e:Mach-O 64-bit executable arm64e]
/bin/ls (for architecture x86_64):	Mach-O 64-bit executable x86_64
/bin/ls (for architecture arm64e):	Mach-O 64-bit executable arm64e

% otool -f -v /bin/ls
Fat headers
fat_magic FAT_MAGIC
<strong>nfat_arch 2
</strong><strong>architecture x86_64
</strong>    cputype CPU_TYPE_X86_64
cpusubtype CPU_SUBTYPE_X86_64_ALL
capabilities 0x0
<strong>    offset 16384
</strong><strong>    size 72896
</strong>    align 2^14 (16384)
<strong>architecture arm64e
</strong>    cputype CPU_TYPE_ARM64
cpusubtype CPU_SUBTYPE_ARM64E
capabilities PTR_AUTH_VERSION USERSPACE 0
<strong>    offset 98304
</strong><strong>    size 88816
</strong>    align 2^14 (16384)
</code></pre>

या [Mach-O View](https://sourceforge.net/projects/machoview/) टूल का उपयोग करके:

<figure><img src="../../../.gitbook/assets/image (5) (1) (1) (3) (1).png" alt=""><figcaption></figcaption></figure>

जैसा कि आप सोच रहे होंगे, आमतौर पर 2 आर्किटेक्चर के लिए एक यूनिवर्सल बाइनरी को **एक आर्किटेक्चर के लिए कंपाइल किए जाने वाले बाइनरी के आकार का दोगुना** होता है।

## **Mach-O  Header**

हैडर में फ़ाइल के बारे में मूलभूत जानकारी होती है, जैसे कि मैजिक बाइट्स जो इसे Mach-O फ़ाइल के रूप में पहचानने के लिए उपयोग किए जाते हैं और लक्षित आर्किटेक्चर के बारे में जानकारी। आप इसे यहां ढूंढ सकते हैं: `mdfind loader.h | grep -i mach-o | grep -E "loader.h$"`
```c
#define	MH_MAGIC	0xfeedface	/* the mach magic number */
#define MH_CIGAM	0xcefaedfe	/* NXSwapInt(MH_MAGIC) */
struct mach_header {
uint32_t	magic;		/* mach magic number identifier */
cpu_type_t	cputype;	/* cpu specifier (e.g. I386) */
cpu_subtype_t	cpusubtype;	/* machine specifier */
uint32_t	filetype;	/* type of file (usage and alignment for the file) */
uint32_t	ncmds;		/* number of load commands */
uint32_t	sizeofcmds;	/* the size of all the load commands */
uint32_t	flags;		/* flags */
};

#define MH_MAGIC_64 0xfeedfacf /* the 64-bit mach magic number */
#define MH_CIGAM_64 0xcffaedfe /* NXSwapInt(MH_MAGIC_64) */
struct mach_header_64 {
uint32_t	magic;		/* mach magic number identifier */
int32_t		cputype;	/* cpu specifier */
int32_t		cpusubtype;	/* machine specifier */
uint32_t	filetype;	/* type of file */
uint32_t	ncmds;		/* number of load commands */
uint32_t	sizeofcmds;	/* the size of all the load commands */
uint32_t	flags;		/* flags */
uint32_t	reserved;	/* reserved */
};
```
**फ़ाइल प्रकार**:

* MH\_EXECUTE (0x2): मानक Mach-O एक्जीक्यूटेबल
* MH\_DYLIB (0x6): Mach-O डायनामिक लिंक्ड लाइब्रेरी (अर्थात् .dylib)
* MH\_BUNDLE (0x8): Mach-O बंडल (अर्थात् .bundle)
```bash
# Checking the mac header of a binary
otool -arch arm64e -hv /bin/ls
Mach header
magic  cputype cpusubtype  caps    filetype ncmds sizeofcmds      flags
MH_MAGIC_64    ARM64          E USR00     EXECUTE    19       1728   NOUNDEFS DYLDLINK TWOLEVEL PIE
```
या [Mach-O View](https://sourceforge.net/projects/machoview/) का उपयोग करके:

<figure><img src="../../../.gitbook/assets/image (4) (1) (4).png" alt=""><figcaption></figcaption></figure>

## **Mach-O लोड कमांड्स**

इसमें **मेमोरी में फ़ाइल का लेआउट** निर्दिष्ट किया जाता है। इसमें **सिम्बल टेबल की स्थान**, प्रारंभ में मुख्य धागा संदर्भ, और कौन सी **साझा पुस्तकालयें** आवश्यक हैं, यह सब शामिल होता है।\
ये कमांड्स मूल रूप से डायनामिक लोडर **(dyld) को बताते हैं कि कैसे बाइनरी को मेमोरी में लोड करना है।**

लोड कमांड्स सभी **load\_command** संरचना से शुरू होते हैं, जो पहले से उल्लिखित **`loader.h`** में परिभाषित है:
```objectivec
struct load_command {
uint32_t cmd;           /* type of load command */
uint32_t cmdsize;       /* total size of command in bytes */
};
```
लगभग **50 विभिन्न प्रकार के लोड कमांड** हैं जिन्हें सिस्टम अलग-अलग ढंग से संभालता है। सबसे सामान्य वाले हैं: `LC_SEGMENT_64`, `LC_LOAD_DYLINKER`, `LC_MAIN`, `LC_LOAD_DYLIB`, और `LC_CODE_SIGNATURE`.

### **LC\_SEGMENT/LC\_SEGMENT\_64**

{% hint style="success" %}
मूल रूप से, इस प्रकार के लोड कमांड निर्धारित करते हैं कि जब बाइनरी को निष्पादित किया जाता है, तो **\_\_TEXT** (निष्पादनयोग्य कोड) और **\_\_DATA** (प्रक्रिया के लिए डेटा) सेगमेंट को **डेटा खंड में दिखाए गए ऑफसेट के अनुसार कैसे लोड करें**।
{% endhint %}

ये कमांड्स **सेगमेंट्स को परिचित कराते हैं** जो जब किसी प्रक्रिया को निष्पादित किया जाता है, उसके **वर्चुअल मेमोरी स्पेस में मैप** हो जाते हैं।

इनमें **विभिन्न प्रकार के सेगमेंट्स** होते हैं, जैसे कि **\_\_TEXT** सेगमेंट, जो किसी प्रोग्राम का निष्पादनयोग्य कोड रखता है, और **\_\_DATA** सेगमेंट, जो प्रक्रिया द्वारा उपयोग किए जाने वाले डेटा को संग्रहित करता है। ये **सेगमेंट्स मैच-ओ फ़ाइल के डेटा सेक्शन में स्थित होते हैं**।

**प्रत्येक सेगमेंट** को और भी **अनुभागों में विभाजित** किया जा सकता है। **लोड कमांड संरचना** में **इन अनुभागों** के बारे में **जानकारी** होती है जो संबंधित सेगमेंट के भीतर होते हैं।

हेडर में पहले आपको **सेगमेंट हेडर** मिलता है:

<pre class="language-c"><code class="lang-c">struct segment_command_64 { /* for 64-bit architectures */
uint32_t	cmd;		/* LC_SEGMENT_64 */
uint32_t	cmdsize;	/* includes sizeof section_64 structs */
char		segname[16];	/* segment name */
uint64_t	vmaddr;		/* memory address of this segment */
uint64_t	vmsize;		/* memory size of this segment */
uint64_t	fileoff;	/* file offset of this segment */
uint64_t	filesize;	/* amount to map from the file */
int32_t		maxprot;	/* maximum VM protection */
int32_t		initprot;	/* initial VM protection */
<strong>	uint32_t	nsects;		/* number of sections in segment */
</strong>	uint32_t	flags;		/* flags */
};
</code></pre>

सेगमेंट हेडर का उदाहरण:

<figure><img src="../../../.gitbook/assets/image (2) (2) (1) (1).png" alt=""><figcaption></figcaption></figure>

यह हेडर **इसके बाद आने वाले हेडर्स की संख्या** को परिभाषित करता है:
```c
struct section_64 { /* for 64-bit architectures */
char		sectname[16];	/* name of this section */
char		segname[16];	/* segment this section goes in */
uint64_t	addr;		/* memory address of this section */
uint64_t	size;		/* size in bytes of this section */
uint32_t	offset;		/* file offset of this section */
uint32_t	align;		/* section alignment (power of 2) */
uint32_t	reloff;		/* file offset of relocation entries */
uint32_t	nreloc;		/* number of relocation entries */
uint32_t	flags;		/* flags (section type and attributes)*/
uint32_t	reserved1;	/* reserved (for offset or index) */
uint32_t	reserved2;	/* reserved (for count or sizeof) */
uint32_t	reserved3;	/* reserved */
};
```
एक **खंड शीर्षक** का उदाहरण:

<figure><img src="../../../.gitbook/assets/image (6) (2).png" alt=""><figcaption></figcaption></figure>

यदि आप **खंड ऑफसेट** (0x37DC) को **जोड़ते हैं** जहां **आर्च शुरू होता है**, इस मामले में `0x18000` --> `0x37DC + 0x18000 = 0x1B7DC`

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

यह भी संभव है कि आप **कमांड लाइन** से **हैडर जानकारी** प्राप्त करें:
```bash
otool -lv /bin/ls
```
इस cmd द्वारा लोड किए जाने वाले सामान्य सेगमेंट:

* **`__PAGEZERO`:** यह कर्नल को निर्देशित करता है कि **पता शून्य** को **पढ़ा नहीं जा सकता, लिखा नहीं जा सकता और नहीं चलाया जा सकता**। संरचना में maxprot और minprot चरों को शून्य सेट किया जाता है ताकि इस पृष्ठ पर **कोई पढ़ने-लिखने-चलाने का अधिकार नहीं हो**।
* यह आवंटन **NULL पॉइंटर डेरेफ़ेरेंस संरचनाओं को कम करने** के लिए महत्वपूर्ण है।
* **`__TEXT`**: **पढ़ने और चलाने** की अनुमति वाले **निष्पादन कोड** को संग्रहित करता है (लिखने की अनुमति नहीं)**।** इस सेगमेंट के सामान्य खंड:
* `__text`: कंपाइल किए गए बाइनरी कोड
* `__const`: स्थिर डेटा
* `__cstring`: स्ट्रिंग स्थिरांक
* `__stubs` और `__stubs_helper`: डायनामिक लाइब्रेरी लोडिंग प्रक्रिया के दौरान संलग्न होते हैं
* **`__DATA`**: पढ़ने और लिखने योग्य डेटा को संग्रहित करता है (चलाने की अनुमति नहीं)**।**
* `__data`: वैश्विक चर (जिन्हें प्रारंभित किया गया है)
* `__bss`: स्थिर चर (जिन्हें प्रारंभित नहीं किया गया है)
* `__objc_*` (\_\_objc\_classlist, \_\_objc\_protolist, आदि): Objective-C रनटाइम द्वारा उपयोग की जाने वाली जानकारी
* **`__LINKEDIT`**: लिंकर (dyld) के लिए जानकारी को संग्रहित करता है, जैसे "सिम्बल, स्ट्रिंग और स्थानांतरण तालिका प्रविष्टियाँ"।
* **`__OBJC`**: Objective-C रनटाइम द्वारा उपयोग की जाने वाली जानकारी। हालांकि, इस जानकारी को \_\_DATA सेगमेंट में भी पाया जा सकता है, विभिन्न \_\_objc\_\* खंडों में।

### **`LC_MAIN`**

**entryoff गुणधर्म** में प्रवेशद्वार को संग्रहित करता है। लोड समय पर, **dyld** बस इस मान को (मेमोरी में) बाइनरी के बेस में जोड़ता है, फिर इस निर्देश के पास छलांग लगाता है ताकि बाइनरी के कोड का निष्पादन शुरू हो सके।

### **LC\_CODE\_SIGNATURE**

Macho-O फ़ाइल के **कोड साइनेचर** के बारे में जानकारी संग्रहित करता है। इसमें केवल एक **ऑफ़सेट** होता है जो **साइनेचर ब्लॉब** को इंगित करता है। यह आमतौर पर फ़ाइल के अंतिम हिस्से में होता है।\
हालांकि, आप इस खंड के बारे में कुछ जानकारी [**इस ब्लॉग पोस्ट**](https://davedelong.com/blog/2018/01/10/reading-your-own-entitlements/) और इस [**गिस्ट**](https://gist.github.com/carlospolop/ef26f8eb9fafd4bc22e69e1a32b81da4) में पा सकते हैं।

### **LC\_LOAD\_DYLINKER**

डायनामिक लिंकर एक्सेक्यूटेबल के **पथ** को संग्रहित करता है जो साझा लाइब्रेरी को प्रक्रिया पता स्थान में मैप करता है। **मान हमेशा `/usr/lib/dyld` पर सेट** होता है। यह महत्वपूर्ण है कि macOS में, dylib मैपिंग कर्नल मोड में नहीं, उपयोगकर्ता मोड में होती है।

### **`LC_LOAD_DYLIB`**

यह लोड कमांड एक **डायनामिक** **लाइब्रेरी** आवश्यकता का वर्णन करता है जो **लोडर** (dyld) को उस लाइब्रेरी को **लोड और लिंक करने के लिए निर्देशित** करता है। Mach-O बाइनरी के लिए हर एक लाइब्रेरी के लिए एक LC\_LOAD\_DYLIB लोड कमांड होता है।

* यह लोड कमांड एक **`dylib_command`** प्रकार का संरचना होता है (जिसमें वास्तविक आश्रित डायनामिक लाइब्रेरी का वर्णन करने वाला struct dylib होता है):
```objectivec
struct dylib_command {
uint32_t        cmd;            /* LC_LOAD_{,WEAK_}DYLIB */
uint32_t        cmdsize;        /* includes pathname string */
struct dylib    dylib;          /* the library identification */
};

struct dylib {
union lc_str  name;                 /* library's path name */
uint32_t timestamp;                 /* library's build time stamp */
uint32_t current_version;           /* library's current version number */
uint32_t compatibility_version;     /* library's compatibility vers number*/
};
```
![](<../../../.gitbook/assets/image (558).png>)

आप इस जानकारी को यहां से भी CLI के साथ प्राप्त कर सकते हैं:
```bash
otool -L /bin/ls
/bin/ls:
/usr/lib/libutil.dylib (compatibility version 1.0.0, current version 1.0.0)
/usr/lib/libncurses.5.4.dylib (compatibility version 5.4.0, current version 5.4.0)
/usr/lib/libSystem.B.dylib (compatibility version 1.0.0, current version 1319.0.0)
```
कुछ संभावित मैलवेयर संबंधित लाइब्रेरी हैं:

* **DiskArbitration**: USB ड्राइव की मॉनिटरिंग
* **AVFoundation:** ऑडियो और वीडियो कैप्चर करें
* **CoreWLAN**: वाईफ़ाई स्कैन करें।

{% hint style="info" %}
एक Mach-O बाइनरी में एक या **अधिक** **कंस्ट्रक्टर** हो सकते हैं, जो **LC\_MAIN** में निर्दिष्ट पते से **पहले** **चलाए जाएंगे**।\
किसी भी कंस्ट्रक्टर के ऑफसेट्स **\_\_DATA\_CONST** सेगमेंट के **\_\_mod\_init\_func** सेक्शन में रखे जाते हैं।
{% endhint %}

## **Mach-O डेटा**

फ़ाइल का ह्रदय अंतिम क्षेत्र, डेटा है, जिसमें लोड-कमांड्स क्षेत्र में लगाए गए कई सेगमेंट होते हैं। **प्रत्येक सेगमेंट में कई डेटा सेक्शन हो सकते हैं**। हर एक सेक्शन में एक विशेष प्रकार के कोड या डेटा होता है।

{% hint style="success" %}
डेटा मूल रूप से वह भाग है जिसमें सभी जानकारी होती है जो लोड कमांड्स **LC\_SEGMENTS\_64** द्वारा लोड होती है।
{% endhint %}

![](<../../../.gitbook/assets/image (507) (3).png>)

इसमें शामिल हैं:

* **फ़ंक्शन टेबल:** जो कार्यक्रम के फ़ंक्शन के बारे में जानकारी रखती है।
* **सिम्बल टेबल**: जो बाइनरी द्वारा उपयोग की जाने वाली बाहरी फ़ंक्शन के बारे में जानकारी रखती है
* इसमें आंतरिक फ़ंक्शन, चर, और अधिक भी हो सकते हैं।

इसे जांचने के लिए आप [**Mach-O View**](https://sourceforge.net/projects/machoview/) टूल का उपयोग कर सकते हैं:

<figure><img src="../../../.gitbook/assets/image (2) (1) (4).png" alt=""><figcaption></figcaption></figure>

या cli से:
```bash
size -m /bin/ls
```
<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें और PRs सबमिट करें** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को।**

</details>
