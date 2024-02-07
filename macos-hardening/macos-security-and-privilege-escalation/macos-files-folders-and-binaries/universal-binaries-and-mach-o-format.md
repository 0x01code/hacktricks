# macOS यूनिवर्सल बाइनरीज़ और Mach-O प्रारूप

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert) के साथ जीरो से हीरो तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PR जमा करके।

</details>

## मूल जानकारी

Mac OS बाइनरीज़ आम तौर पर **यूनिवर्सल बाइनरीज़** के रूप में कॉम्पाइल किए जाते हैं। एक **यूनिवर्सल बाइनरी** में **एक ही फ़ाइल में कई आर्किटेक्चर का समर्थन कर सकता है**।

ये बाइनरीज़ **Mach-O संरचना** का पालन करती हैं जो मुख्य रूप से निम्नलिखित से बना होता है:

* हेडर
* लोड कमांड्स
* डेटा

![https://alexdremov.me/content/images/2022/10/6XLCD.gif](<../../../.gitbook/assets/image (559).png>)

## फैट हेडर

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

हेडर में **मैजिक** बाइट्स होते हैं जिनके बाद फ़ाइल में मौजूद **आर्क्स** की संख्या (`nfat_arch`) होती है और प्रत्येक आर्क के पास `fat_arch` स्ट्रक्ट होती है।

इसे चेक करें:

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

जैसा कि आप सोच रहे होंगे, आम तौर पर 2 आर्किटेक्चर के लिए एक यूनिवर्सल बाइनरी का आकार एक के लिए कॉम्पाइल किए जाने वाले के आकार को **दोगुना** कर देता है।

## **Mach-O हेडर**

हेडर में फ़ाइल के बारे में मूल जानकारी होती है, जैसे कि मैजिक बाइट्स ताकि इसे एक Mach-O फ़ाइल के रूप में पहचाना जा सके और लक्षित आर्किटेक्चर के बारे में जानकारी। आप इसे यहाँ पा सकते हैं: `mdfind loader.h | grep -i mach-o | grep -E "loader.h$"`
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

* MH\_EXECUTE (0x2): मानक Mach-O कार्यकारी
* MH\_DYLIB (0x6): एक Mach-O डायनामिक लिंक्ड लाइब्रेरी (अर्थात .dylib)
* MH\_BUNDLE (0x8): एक Mach-O बंडल (अर्थात .bundle)
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

**फ़ाइल का लेआउट मेमोरी में** यहाँ निर्दिष्ट किया गया है, **सिम्बल टेबल का स्थान**, कार्यान्वयन प्रारंभ पर मुख्य धागे का संदर्भ, और आवश्यक **साझा लाइब्रेरी**। निर्देश दिए गए हैं डायनामिक लोडर **(dyld)** को मेमोरी में बाइनरी को लोड करने की प्रक्रिया पर।

उपयोग करता है **load\_command** संरचना, जो उल्लिखित **`loader.h`** में परिभाषित है:
```objectivec
struct load_command {
uint32_t cmd;           /* type of load command */
uint32_t cmdsize;       /* total size of command in bytes */
};
```
व्यवस्था को अलग-अलग ढंग से संभालने वाले लोड कमांड के लगभग **50 विभिन्न प्रकार** हैं। सबसे सामान्य वाले हैं: `LC_SEGMENT_64`, `LC_LOAD_DYLINKER`, `LC_MAIN`, `LC_LOAD_DYLIB`, और `LC_CODE_SIGNATURE`।

### **LC\_SEGMENT/LC\_SEGMENT\_64**

{% hint style="success" %}
मूल रूप से, इस प्रकार के लोड कमांड **कैसे \_\_TEXT** (कार्यात्मक कोड) **और \_\_DATA** (प्रक्रिया के लिए डेटा) **सेगमेंट** को **डेटा खंड में दिखाए गए ऑफसेट के अनुसार** बाइनरी को निष्पादित किया जाता है।
{% endhint %}

ये कमांड **सेगमेंट को परिभाषित** करते हैं जो एक प्रक्रिया के **वर्चुअल मेमोरी स्पेस** में **मैप** किए जाते हैं जब यह निष्पादित होता है।

इनमें **विभिन्न प्रकार** के सेगमेंट होते हैं, जैसे **\_\_TEXT** सेगमेंट, जो किसी प्रोग्राम के कार्यात्मक कोड को धारित करता है, और **\_\_DATA** सेगमेंट, जो प्रक्रिया द्वारा उपयोग किए जाने वाले डेटा को समेतता है। ये **सेगमेंट** मैच-ओ फ़ाइल के डेटा सेक्शन में स्थित होते हैं।

**प्रत्येक सेगमेंट** को और भी अधिक **विभाजित** किया जा सकता है। **लोड कमांड संरचना** में **जानकारी** होती है **इन सेगमेंट के भीतर के अनुभागों** के बारे में।

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

यह हेडर **उन सेक्शनों की संख्या को परिभाषित करता है जिनके हेडर इसके बाद दिखाई देते हैं**:
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
उदाहरण का **खंड शीर्षक**:

<figure><img src="../../../.gitbook/assets/image (6) (2).png" alt=""><figcaption></figcaption></figure>

यदि आप **खंड ऑफसेट** (0x37DC) + **ऑफसेट** जो **आर्क शुरू होता है**, इस मामले में `0x18000` जोड़ें --> `0x37DC + 0x18000 = 0x1B7DC`

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

यह भी संभव है कि आप **कमांड लाइन** से **हेडर्स जानकारी** प्राप्त करें:
```bash
otool -lv /bin/ls
```
इस cmd द्वारा लोड किए जाने वाले सामान्य सेगमेंट:

* **`__PAGEZERO`:** यह कर्नेल को निर्देशित करता है कि **पता शून्य** को **मैप** किया जाए ताकि इसे **पढ़ा नहीं जा सकता, लिखा नहीं जा सकता, या नहीं चलाया जा सकता**। संरचना में maxprot और minprot चर मूल्य शून्य पर सेट किए जाते हैं ताकि इस पृष्ठ पर **कोई पढ़ने-लिखने-चलाने के अधिकार न हों**।
* यह आवंटन NULL प्वाइंटर डिफरेंस वल्नरेबिलिटी को **कम करने** के लिए महत्वपूर्ण है।
* **`__TEXT`**: **पाठ्यक्रमीय** **कोड** को **पढ़ने** और **चलाने** की अनुमति है (कोई लिखने की अनुमति नहीं)**.** इस सेगमेंट के सामान्य खंड:
* `__text`: कंपाइल किया गया बाइनरी कोड
* `__const`: स्थिर डेटा
* `__cstring`: स्ट्रिंग स्थिर
* `__stubs` और `__stubs_helper`: डायनामिक लाइब्रेरी लोडिंग प्रक्रिया के दौरान संलग्न
* **`__DATA`**: डेटा को **पढ़ने** और **लिखने** की अनुमति है (कोई चलाने की अनुमति नहीं)**.**
* `__data`: ग्लोबल वेरिएबल्स (जिन्होंने प्रारंभ किया गया है)
* `__bss`: स्थैतिक वेरिएबल्स (जिन्होंने प्रारंभ नहीं किया गया है)
* `__objc_*` (\_\_objc\_classlist, \_\_objc\_protolist, आदि): ओब्जेक्टिव-सी रनटाइम द्वारा उपयोग की जाने वाली जानकारी
* **`__LINKEDIT`**: लिंकर (dyld) के लिए जानकारी शामिल है, जैसे, "सिम्बल, स्ट्रिंग, और स्थानांतरण सारणियाँ।"
* **`__OBJC`**: ओब्जेक्टिव-सी रनटाइम द्वारा उपयोग की जाने वाली जानकारी। हालांकि, यह जानकारी \_\_DATA सेगमेंट में भी मिल सकती है, विभिन्न \_\_objc\_\* खंडों में।

### **`LC_MAIN`**

**entryoff विशेषता में** प्रवेश बिंदु को शामिल करता है। लोड समय पर, **dyld** बस (इन-मेमोरी) के बाइनरी के **मूल्य** में बस्ता है, फिर इस निर्देशिका पर जाता है ताकि बाइनरी कोड का निष्पादन शुरू हो सके।

### **LC\_CODE\_SIGNATURE**

Macho-O फ़ाइल के **कोड हस्ताक्षर** के बारे में जानकारी शामिल है। यह केवल एक **ओफ़सेट** शामिल करता है जो **हस्ताक्षर ब्लॉब** की ओर **इशारा करता है**। यह आम तौर पर फ़ाइल के अंत में होता है।\
हालांकि, आप इस खंड के बारे में कुछ जानकारी [**इस ब्लॉग पोस्ट**](https://davedelong.com/blog/2018/01/10/reading-your-own-entitlements/) और इस [**गिस्ट**](https://gist.github.com/carlospolop/ef26f8eb9fafd4bc22e69e1a32b81da4) में पा सकते हैं।

### **LC\_LOAD\_DYLINKER**

प्रक्रिया पते में साझा लाइब्रेरी मैप करने वाले डायनामिक लिंकर कार्यक्षम के **पथ** को शामिल करता है। **मान हमेशा `/usr/lib/dyld` पर सेट किया जाता है**। यह महत्वपूर्ण है कि macOS में, dylib मैपिंग **उपयोगकर्ता मोड** में होता है, कर्नेल मोड में नहीं।

### **`LC_LOAD_DYLIB`**

यह लोड कमांड एक **डायनामिक लाइब्रेरी** आवश्यकता का वर्णन करता है जो **लोडर** (dyld) को **उस लाइब्रेरी को लोड और लिंक करने के लिए निर्देशित** करता है। Mach-O बाइनरी के प्रत्येक लाइब्रेरी के लिए एक LC\_LOAD\_DYLIB लोड कमांड होता है।

* यह लोड कमांड एक प्रकार की संरचना है **`dylib_command`** (जिसमें वास्तविक निर्भर डायनामिक लाइब्रेरी का वर्णन करने वाला स्ट्रक्ट डायलिब होता है):
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

आप इस जानकारी को क्लाइंट से भी प्राप्त कर सकते हैं:
```bash
otool -L /bin/ls
/bin/ls:
/usr/lib/libutil.dylib (compatibility version 1.0.0, current version 1.0.0)
/usr/lib/libncurses.5.4.dylib (compatibility version 5.4.0, current version 5.4.0)
/usr/lib/libSystem.B.dylib (compatibility version 1.0.0, current version 1319.0.0)
```
कुछ संभावित मैलवेयर संबंधित लाइब्रेरीज हैं:

* **DiskArbitration**: USB ड्राइव की मॉनिटरिंग
* **AVFoundation:** ऑडियो और वीडियो कैप्चर
* **CoreWLAN**: वाईफाई स्कैन।

{% hint style="info" %}
एक Mach-O बाइनरी में एक या **एक से अधिक निर्माताओं** को शामिल किया जा सकता है, जो **LC\_MAIN** में निर्दिष्ट पते से **पहले** **चलाए जाएंगे**।\
किसी भी निर्माताओं के ऑफसेट **\_\_DATA\_CONST** सेगमेंट के **\_\_mod\_init\_func** खंड में रखे जाते हैं।
{% endhint %}

## **Mach-O डेटा**

फ़ाइल के मूल में डेटा क्षेत्र होता है, जो लोड-कमांड्स क्षेत्र में परिभाषित कई सेगमेंटों से बना होता है। **प्रत्येक सेगमेंट में कई डेटा खंड हो सकते हैं**, हर खंड **कोड या डेटा** विशेष के लिए।

{% hint style="success" %}
डेटा मुख्य रूप से उस सभी **जानकारी** को समेटता है जो लोड कमांड्स **LC\_SEGMENTS\_64** द्वारा लोड की जाती है।
{% endhint %}

![https://www.oreilly.com/api/v2/epubs/9781785883378/files/graphics/B05055_02_38.jpg](<../../../.gitbook/assets/image (507) (3).png>)

इसमें शामिल हैं:

* **कार्य सारणी:** जो कार्य कार्यों के बारे में जानकारी रखती है।
* **प्रतीक सारणी**: जो बाइनरी द्वारा उपयोग की गई बाह्य कार्य के बारे में जानकारी रखती है
* यह आंतरिक कार्य, चर नामों को भी शामिल कर सकता है और अधिक।

इसे जांचने के लिए आप [**Mach-O View**](https://sourceforge.net/projects/machoview/) टूल का उपयोग कर सकते हैं:

<figure><img src="../../../.gitbook/assets/image (2) (1) (4).png" alt=""><figcaption></figcaption></figure>

या CLI से:
```bash
size -m /bin/ls
```
<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

दूसरे तरीके HackTricks का समर्थन करने के लिए:

* अगर आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें** PRs के माध्यम से [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में सबमिट करके।

</details>
