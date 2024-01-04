# macOS यूनिवर्सल बाइनरीज़ और Mach-O फॉर्मेट

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert) के साथ AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) को **फॉलो करें**.
* **HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपोज़ में PRs सबमिट करके अपनी हैकिंग ट्रिक्स शेयर करें.

</details>

## मूल जानकारी

Mac OS बाइनरीज़ आमतौर पर **यूनिवर्सल बाइनरीज़** के रूप में कंपाइल की जाती हैं। एक **यूनिवर्सल बाइनरी** एक ही फाइल में **कई आर्किटेक्चर्स का समर्थन कर सकती है**।

ये बाइनरीज़ **Mach-O संरचना** का पालन करती हैं जो मूल रूप से निम्नलिखित से बनी होती है:

* हेडर
* लोड कमांड्स
* डेटा

![](<../../../.gitbook/assets/image (559).png>)

## फैट हेडर

फाइल को इसके साथ खोजें: `mdfind fat.h | grep -i mach-o | grep -E "fat.h$"`

<pre class="language-c"><code class="lang-c"><strong>#define FAT_MAGIC	0xcafebabe
</strong><strong>#define FAT_CIGAM	0xbebafeca	/* NXSwapLong(FAT_MAGIC) */
</strong>
struct fat_header {
<strong>	uint32_t	magic;		/* FAT_MAGIC or FAT_MAGIC_64 */
</strong><strong>	uint32_t	nfat_arch;	/* इसके बाद आने वाले स्ट्रक्चर्स की संख्या */
</strong>};

struct fat_arch {
cpu_type_t	cputype;	/* cpu विशेषक (int) */
cpu_subtype_t	cpusubtype;	/* मशीन विशेषक (int) */
uint32_t	offset;		/* इस ऑब्जेक्ट फाइल के लिए फाइल ऑफसेट */
uint32_t	size;		/* इस ऑब्जेक्ट फाइल का आकार */
uint32_t	align;		/* 2 की शक्ति के रूप में संरेखण */
};
</code></pre>

हेडर में **मैजिक** बाइट्स के बाद फाइल में **शामिल आर्क्स की संख्या** (`nfat_arch`) होती है और प्रत्येक आर्क के लिए एक `fat_arch` स्ट्रक्चर होगा।

इसे इसके साथ जांचें:

<pre class="language-shell-session"><code class="lang-shell-session">% file /bin/ls
/bin/ls: Mach-O यूनिवर्सल बाइनरी 2 आर्किटेक्चर्स के साथ: [x86_64:Mach-O 64-बिट एक्जीक्यूटेबल x86_64] [arm64e:Mach-O 64-बिट एक्जीक्यूटेबल arm64e]
/bin/ls (for architecture x86_64):	Mach-O 64-बिट एक्जीक्यूटेबल x86_64
/bin/ls (for architecture arm64e):	Mach-O 64-बिट एक्जीक्यूटेबल arm64e

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

जैसा कि आप सोच रहे होंगे, आमतौर पर एक यूनिवर्सल बाइनरी जो 2 आर्किटेक्चर्स के लिए कंपाइल की गई है, उसका आकार उस बाइनरी के दोगुना होता है जो केवल 1 आर्किटेक्चर के लिए कंपाइल की गई है।

## **Mach-O हेडर**

हेडर में फाइल के बारे में मूल जानकारी होती है, जैसे कि इसे Mach-O फाइल के रूप में पहचानने के लिए मैजिक बाइट्स और लक्ष्य आर्किटेक्चर के बारे में जानकारी। आप इसे यहाँ पा सकते हैं: `mdfind loader.h | grep -i mach-o | grep -E "loader.h$"`
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

* MH\_EXECUTE (0x2): मानक Mach-O निष्पादन योग्य
* MH\_DYLIB (0x6): एक Mach-O डायनामिक लिंक्ड लाइब्रेरी (उदा. .dylib)
* MH\_BUNDLE (0x8): एक Mach-O बंडल (उदा. .bundle)
```bash
# Checking the mac header of a binary
otool -arch arm64e -hv /bin/ls
Mach header
magic  cputype cpusubtype  caps    filetype ncmds sizeofcmds      flags
MH_MAGIC_64    ARM64          E USR00     EXECUTE    19       1728   NOUNDEFS DYLDLINK TWOLEVEL PIE
```
या [Mach-O View](https://sourceforge.net/projects/machoview/) का उपयोग करते हुए:

<figure><img src="../../../.gitbook/assets/image (4) (1) (4).png" alt=""><figcaption></figcaption></figure>

## **Mach-O लोड कमांड्स**

यह **फाइल की मेमोरी में लेआउट** को निर्दिष्ट करता है। इसमें **सिंबल टेबल का स्थान**, निष्पादन की शुरुआत में मुख्य थ्रेड कॉन्टेक्स्ट, और कौन सी **शेयर्ड लाइब्रेरीज** आवश्यक हैं, शामिल हैं।\
कमांड्स मूल रूप से डायनामिक लोडर **(dyld) को यह निर्देश देती हैं कि बाइनरी को मेमोरी में कैसे लोड किया जाए।**

लोड कमांड्स सभी **load\_command** संरचना से शुरू होती हैं, जो पहले उल्लेखित **`loader.h`** में परिभाषित है:
```objectivec
struct load_command {
uint32_t cmd;           /* type of load command */
uint32_t cmdsize;       /* total size of command in bytes */
};
```
लगभग **50 विभिन्न प्रकार के लोड कमांड्स** होते हैं जिन्हें सिस्टम अलग-अलग तरीके से संभालता है। सबसे आम वाले हैं: `LC_SEGMENT_64`, `LC_LOAD_DYLINKER`, `LC_MAIN`, `LC_LOAD_DYLIB`, और `LC_CODE_SIGNATURE`.

### **LC\_SEGMENT/LC\_SEGMENT\_64**

{% hint style="success" %}
मूल रूप से, इस प्रकार का लोड कमांड **कैसे \_\_TEXT** (निष्पादन योग्य कोड) **और \_\_DATA** (प्रक्रिया के लिए डेटा) **सेगमेंट्स को लोड करना है** यह परिभाषित करता है, जो कि **डेटा सेक्शन में दिए गए ऑफसेट्स के अनुसार** होता है जब बाइनरी निष्पादित की जाती है।
{% endhint %}

ये कमांड्स **सेगमेंट्स को परिभाषित करते हैं** जो कि **वर्चुअल मेमोरी स्पेस** में **मैप किए जाते हैं** जब कोई प्रक्रिया निष्पादित की जाती है।

**विभिन्न प्रकार** के सेगमेंट्स होते हैं, जैसे कि **\_\_TEXT** सेगमेंट, जो किसी प्रोग्राम के निष्पादन योग्य कोड को रखता है, और **\_\_DATA** सेगमेंट, जो प्रक्रिया द्वारा उपयोग किए जाने वाले डेटा को समाहित करता है। ये **सेगमेंट्स मच-ओ फाइल के डेटा सेक्शन में स्थित होते हैं**।

**प्रत्येक सेगमेंट** को आगे **विभाजित** किया जा सकता है बहुत सारे **सेक्शन्स** में। **लोड कमांड स्ट्रक्चर** में **इन सेक्शन्स के बारे में जानकारी** होती है जो संबंधित सेगमेंट के भीतर होती है।

हेडर में पहले आपको **सेगमेंट हेडर** मिलता है:

<pre class="language-c"><code class="lang-c">struct segment_command_64 { /* 64-बिट आर्किटेक्चर के लिए */
uint32_t	cmd;		/* LC_SEGMENT_64 */
uint32_t	cmdsize;	/* includes sizeof section_64 structs */
char		segname[16];	/* सेगमेंट का नाम */
uint64_t	vmaddr;		/* इस सेगमेंट का मेमोरी एड्रेस */
uint64_t	vmsize;		/* इस सेगमेंट का मेमोरी साइज */
uint64_t	fileoff;	/* इस सेगमेंट का फाइल ऑफसेट */
uint64_t	filesize;	/* फाइल से मैप करने के लिए राशि */
int32_t		maxprot;	/* अधिकतम VM सुरक्षा */
int32_t		initprot;	/* प्रारंभिक VM सुरक्षा */
<strong>	uint32_t	nsects;		/* सेगमेंट में सेक्शन्स की संख्या */
</strong>	uint32_t	flags;		/* झंडे */
};
</code></pre>

सेगमेंट हेडर का उदाहरण:

<figure><img src="../../../.gitbook/assets/image (2) (2) (1) (1).png" alt=""><figcaption></figcaption></figure>

यह हेडर **उन सेक्शन्स की संख्या को परिभाषित करता है जिनके हेडर्स इसके बाद आते हैं**:
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
उदाहरण के लिए **अनुभाग शीर्षक**:

<figure><img src="../../../.gitbook/assets/image (6) (2).png" alt=""><figcaption></figcaption></figure>

यदि आप **अनुभाग ऑफसेट** (0x37DC) + **ऑफसेट** जहां **आर्किटेक्चर शुरू होता है**, इस मामले में `0x18000` को **जोड़ते हैं** --> `0x37DC + 0x18000 = 0x1B7DC`

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

**कमांड लाइन** के साथ **हेडर्स की जानकारी** प्राप्त करना भी संभव है:
```bash
otool -lv /bin/ls
```
इस cmd द्वारा लोड किए गए सामान्य सेगमेंट:

* **`__PAGEZERO`:** यह कर्नेल को निर्देश देता है कि **address zero** को **map** करें ताकि इसे **पढ़ा, लिखा या निष्पादित** नहीं किया जा सके। संरचना में maxprot और minprot चर को शून्य पर सेट किया गया है जो दर्शाता है कि इस पेज पर **कोई पढ़ने-लिखने-निष्पादित करने के अधिकार नहीं हैं**।
* यह आवंटन **NULL pointer dereference vulnerabilities को कम करने** के लिए महत्वपूर्ण है।
* **`__TEXT`**: **निष्पादन योग्य** **कोड** शामिल होता है जिसमें **पढ़ने** और **निष्पादित** करने की अनुमति होती है (लिखने योग्य नहीं)। इस सेगमेंट के सामान्य अनुभाग:
* `__text`: संकलित बाइनरी कोड
* `__const`: स्थिर डेटा
* `__cstring`: स्ट्रिंग कॉन्स्टेंट्स
* `__stubs` और `__stubs_helper`: डायनामिक लाइब्रेरी लोडिंग प्रक्रिया के दौरान शामिल
* **`__DATA`**: डेटा शामिल होता है जो **पढ़ने योग्य** और **लिखने योग्य** होता है (निष्पादन योग्य नहीं)।
* `__data`: ग्लोबल वेरिएबल्स (जो इनिशियलाइज़ किए गए हैं)
* `__bss`: स्टैटिक वेरिएबल्स (जो इनिशियलाइज़ नहीं किए गए हैं)
* `__objc_*` (\_\_objc\_classlist, \_\_objc\_protolist, आदि): Objective-C रनटाइम द्वारा इस्तेमाल की गई जानकारी
* **`__LINKEDIT`**: लिंकर (dyld) के लिए जानकारी शामिल होती है जैसे, "सिंबल, स्ट्रिंग, और रिलोकेशन टेबल एंट्रीज़।"
* **`__OBJC`**: Objective-C रनटाइम द्वारा इस्तेमाल की गई जानकारी शामिल होती है। हालांकि, यह जानकारी \_\_DATA सेगमेंट में भी मिल सकती है, विभिन्न \_\_objc\_\* अनुभागों के भीतर।

### **`LC_MAIN`**

इसमें **entryoff attribute** में एंट्रीपॉइंट शामिल होता है। लोड समय पर, **dyld** बस इस मान को बाइनरी के (मेमोरी में) **बेस** में **जोड़ता** है, फिर बाइनरी के कोड की निष्पादन शुरू करने के लिए इस निर्देश पर **कूदता** है।

### **LC\_CODE\_SIGNATURE**

Macho-O फाइल के **कोड सिग्नेचर** के बारे में जानकारी शामिल होती है। यह केवल एक **ऑफसेट** शामिल करता है जो **सिग्नेचर ब्लॉब** की ओर **इशारा** करता है। यह आमतौर पर फाइल के बहुत अंत में होता है।\
हालांकि, इस अनुभाग के बारे में कुछ जानकारी आप [**इस ब्लॉग पोस्ट**](https://davedelong.com/blog/2018/01/10/reading-your-own-entitlements/) और इस [**gists**](https://gist.github.com/carlospolop/ef26f8eb9fafd4bc22e69e1a32b81da4) में पा सकते हैं।

### **LC\_LOAD\_DYLINKER**

इसमें **डायनामिक लिंकर एक्जीक्यूटेबल** का पथ शामिल होता है जो शेयर्ड लाइब्रेरीज को प्रोसेस एड्रेस स्पेस में मैप करता है। **मान हमेशा `/usr/lib/dyld` पर सेट किया जाता है**। यह ध्यान देना महत्वपूर्ण है कि macOS में, dylib मैपिंग **यूजर मोड** में होती है, कर्नेल मोड में नहीं।

### **`LC_LOAD_DYLIB`**

यह लोड कमांड एक **डायनामिक** **लाइब्रेरी** निर्भरता का वर्णन करता है जो **लोडर** (dyld) को **संबंधित लाइब्रेरी को लोड और लिंक करने का निर्देश** देता है। Mach-O बाइनरी द्वारा आवश्यक प्रत्येक लाइब्रेरी के लिए एक LC\_LOAD\_DYLIB लोड कमांड होता है।

* यह लोड कमांड **`dylib_command`** प्रकार की संरचना होती है (जिसमें एक struct dylib शामिल होता है, जो वास्तविक निर्भर डायनामिक लाइब्रेरी का वर्णन करता है):
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
आप यह जानकारी cli से भी प्राप्त कर सकते हैं:
```bash
otool -L /bin/ls
/bin/ls:
/usr/lib/libutil.dylib (compatibility version 1.0.0, current version 1.0.0)
/usr/lib/libncurses.5.4.dylib (compatibility version 5.4.0, current version 5.4.0)
/usr/lib/libSystem.B.dylib (compatibility version 1.0.0, current version 1319.0.0)
```
कुछ संभावित मैलवेयर संबंधित लाइब्रेरीज़ हैं:

* **DiskArbitration**: USB ड्राइव्स की निगरानी
* **AVFoundation:** ऑडियो और वीडियो कैप्चर
* **CoreWLAN**: वाईफाई स्कैन।

{% hint style="info" %}
एक Mach-O बाइनरी में एक या **अधिक** **कंस्ट्रक्टर्स** हो सकते हैं, जो **LC\_MAIN** में निर्दिष्ट पते से **पहले** **निष्पादित** किए जाएंगे।\
किसी भी कंस्ट्रक्टर्स के ऑफसेट **\_\_mod\_init\_func** सेक्शन में रखे जाते हैं जो कि **\_\_DATA\_CONST** सेगमेंट का हिस्सा होते हैं।
{% endhint %}

## **Mach-O डेटा**

फाइल का मुख्य भाग अंतिम क्षेत्र, डेटा है, जिसमें लोड-कमांड्स क्षेत्र में निर्धारित कई सेगमेंट्स होते हैं। **प्रत्येक सेगमेंट में कई डेटा सेक्शन्स हो सकते हैं**। प्रत्येक सेक्शन में एक विशेष प्रकार का **कोड या डेटा** होता है।

{% hint style="success" %}
डेटा मूल रूप से वह भाग है जिसमें सभी **जानकारी** होती है जो लोड कमांड्स **LC\_SEGMENTS\_64** द्वारा लोड की जाती है।
{% endhint %}

![](<../../../.gitbook/assets/image (507) (3).png>)

इसमें शामिल हैं:&#x20;

* **फंक्शन टेबल:** जिसमें प्रोग्राम फंक्शन्स के बारे में जानकारी होती है।
* **सिंबल टेबल**: जिसमें बाइनरी द्वारा इस्तेमाल किए गए बाहरी फंक्शन के बारे में जानकारी होती है
* इसमें आंतरिक फंक्शन, वेरिएबल नाम और अधिक भी शामिल हो सकते हैं।

इसे जांचने के लिए आप [**Mach-O View**](https://sourceforge.net/projects/machoview/) टूल का उपयोग कर सकते हैं:

<figure><img src="../../../.gitbook/assets/image (2) (1) (4).png" alt=""><figcaption></figcaption></figure>

या कमांड लाइन से:
```bash
size -m /bin/ls
```
<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग प्राप्त करें**](https://peass.creator-spring.com)
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स शेयर करें।

</details>
