# macOS पुस्तकालय इंजेक्शन

<details>

<summary><strong>जीरो से हीरो तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी कंपनी का विज्ञापन **HackTricks** में देखना चाहते हैं या **HackTricks को PDF में डाउनलोड** करना चाहते हैं तो [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह "The PEASS Family" की खोज करें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें** द्वारा PR जमा करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में।

</details>

{% hint style="danger" %}
**dyld का कोड ओपन सोर्स** है और इसे [https://opensource.apple.com/source/dyld/](https://opensource.apple.com/source/dyld/) पर पाया जा सकता है और एक **URL जैसे** [https://opensource.apple.com/tarballs/dyld/dyld-852.2.tar.gz](https://opensource.apple.com/tarballs/dyld/dyld-852.2.tar.gz) का उपयोग करके एक टार डाउनलोड किया जा सकता है।
{% endhint %}

## **Dyld प्रक्रिया**

देखें कि Dyld कैसे बाइनरी में पुस्तकालयों को लोड करता है:

{% content-ref url="macos-dyld-process.md" %}
[macos-dyld-process.md](macos-dyld-process.md)
{% endcontent-ref %}

## **DYLD\_INSERT\_LIBRARIES**

यह [**LD\_PRELOAD on Linux**](../../../../linux-hardening/privilege-escalation/#ld\_preload) की तरह है। इसकी अनुमति देता है कि एक प्रक्रिया को सूचित किया जाए कि एक विशिष्ट पुस्तकालय को एक मार्ग से लोड किया जाएगा (यदि env चर सक्षम है)

यह तकनीक एक ASEP तकनीक के रूप में भी **उपयोग किया जा सकता है** क्योंकि हर एप्लिकेशन इंस्टॉल किया गया है उसके पास "Info.plist" नामक एक plist होता है जो `LSEnvironmental` नामक एक कुंजी का उपयोग करके **पर्यावरणीय चरों का निरुपण** करने की अनुमति देता है।

{% hint style="info" %}
2012 से **Apple ने DYLD_INSERT_LIBRARIES** की **शक्ति को बहुत कम कर दिया है**।

कोड पर जाएं और **`src/dyld.cpp`** की जांच करें। फ़ंक्शन **`pruneEnvironmentVariables`** में आप देख सकते हैं कि **`DYLD_*`** चर हटाए गए हैं।

फ़ंक्शन **`processRestricted`** में प्रतिबंध का कारण सेट किया गया है। उस कोड की जांच करके आप देख सकते हैं कि कारण हैं:

* बाइनरी `setuid/setgid` है
* मैचो बाइनरी में `__RESTRICT/__restrict` खंड का मौजूद होना।
* सॉफ़्टवेयर में entitlements हैं (हार्डन्ड रनटाइम) बिना [`com.apple.security.cs.allow-dyld-environment-variables`](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_allow-dyld-environment-variables) entitlement
* एक बाइनरी की **entitlements** की जांच करें: `codesign -dv --entitlements :- </path/to/bin>`

अधिक अद्यतित संस्करणों में आप इस तर्क को **`configureProcessRestrictions`** के फ़ंक्शन के दूसरे हिस्से में पा सकते हैं। हालांकि, नए संस्करणों में वह तर्क का आरंभ करता है जो नए संस्करणों में नहीं उपयोग किया जाएगा (आईओएस या सिम्युलेशन से संबंधित ifs को हटा सकते हैं क्योंकि वे macOS में उपयोग नहीं होंगे)।
{% endhint %}

### पुस्तकालय सत्यापन

यदि बाइनरी **`DYLD_INSERT_LIBRARIES`** env चर का उपयोग करने की अनुमति देता है, तो यदि बाइनरी पुस्तकालय के हस्ताक्षर की जांच करता है तो एक कस्टम क्या नहीं लोड करेगा।

कस्टम पुस्तकालय लोड करने के लिए बाइनरी के पास निम्नलिखित entitlements होने चाहिए:

* [`com.apple.security.cs.disable-library-validation`](../../macos-security-protections/macos-dangerous-entitlements.md#com.apple.security.cs.disable-library-validation)
* [`com.apple.private.security.clear-library-validation`](../../macos-security-protections/macos-dangerous-entitlements.md#com.apple.private.security.clear-library-validation)

या तो बाइनरी के पास **हार्डन्ड रनटाइम झंडा** नहीं होना चाहिए या **पुस्तकालय सत्यापन झंडा** नहीं होना चाहिए।

आप यह जांच सकते हैं कि क्या एक बाइनरी में **हार्डन्ड रनटाइम** है `codesign --display --verbose <bin>` का उपयोग करके **`CodeDirectory`** में झंडा देखकर जैसे: **`CodeDirectory v=20500 size=767 flags=0x10000(runtime) hashes=13+7 location=embedded`**

आप एक पुस्तकालय भी लोड कर सकते हैं यदि वह **बाइनरी के साथ समान प्रमाणपत्र से हस्ताक्षरित** है।

इसे (अब)यह कैसे (अब) उपयोग करने के लिए एक उदाहरण खोजें और प्रतिबंधों की जांच करें:

{% content-ref url="macos-dyld-hijacking-and-dyld_insert_libraries.md" %}
[macos-dyld-hijacking-and-dyld\_insert\_libraries.md](macos-dyld-hijacking-and-dyld\_insert\_libraries.md)
{% endcontent-ref %}

## Dylib हाइजैकिंग

{% hint style="danger" %}
ध्यान दें कि **पिछले पुस्तकालय सत्यापन प्रतिबंध** भी Dylib हाइजैकिंग हमलों को करने के लिए लागू होते हैं।
{% endhint %}

विंडोज में, MacOS में भी आप **डाइलिब्स को हाइजैक** कर सकते हैं ताकि **एप्लिकेशन** **कोड** **को** **अवश्यक** **करें** (ठीक है, वास्तव में एक सामान्य उपयोगकर्ता से यह संभव नहीं है क्योंकि आपको एक `.app` बंडल में लिखने के लिए TCC अनुमति की आवश्यकता हो सकती है और एक पुस्तकालय को हाइजैक करने के लिए)।

हालांकि, **MacOS** एप्लिकेशन **पुस्तकालयों को लोड** करने का **प्रमाण** **विंडोज** से **अधिक प्रतिबंधित** है। इसका अर्थ है कि **मैलवेयर** विकासक इस तकनीक का उपयोग **छिपाने** के लिए कर सकते हैं, लेकिन इसे **विशेषाधिकारों को उन्नत करने के लिए उपयोग करने की संभावना कम है**।

सबसे पहले, **अधिक सामान्य** है कि **MacOS बाइनरी** पुस्तकालयों के लिए **पूरा मार्ग निर्दिष्ट** करते हैं। और दूसरा, **MacOS कभी नहीं खोजता** है **$PATH** के फ़ोल्डर में पुस्तकालयों के लिए।

इस कार्यक्षेत्र से संबंधित **मुख्य** हिस्सा **`ImageLoader::recursiveLoadLibraries`** में `ImageLoader.cpp` में है।

मैचो बाइनरी एक पुस्तकालय लोड करने के लिए ये **4 विभिन्न हेडर कमांड** उपयोग कर सकता है:

* **`LC_LOAD_DYLIB`** कमांड एक dylib लोड करने के लिए सामान्य कमांड है।
* **`LC_LOAD_WEAK_DYLIB`** कमांड पिछले कमांड की तरह काम करता है, लेकिन यदि dylib नहीं मिलता है, तो कोई त्रुटि के बिना कार्यान्वयन जारी रहेगा।
* **`LC_REEXPORT_DYLIB`** कमांड यह प्रॉक्सी करता है (या पुनः निर्यात करता है) विभिन्न पुस्तकालयों से प्रतीकों को।
* **`LC_LOAD_UPWARD_DYLIB`** कमांड जब दो पुस्तकालय एक-दूसरे पर निर्भर होते हैं (इसे _
* **`LC_LOAD_DYLIB`** में विशिष्ट पुस्तकालयों को लोड करने के लिए मार्ग शामिल हैं। ये मार्ग **`@rpath`** भी शामिल कर सकते हैं, जो **`LC_RPATH`** में मौजूद मानों द्वारा **प्रतिस्थापित** किया जाएगा। अगर **`LC_RPATH`** में कई मार्ग हैं तो सभी को पुस्तकालय खोजने के लिए उपयोग किया जाएगा। उदाहरण:
* अगर **`LC_LOAD_DYLIB`** में `@rpath/library.dylib` है और **`LC_RPATH`** में `/application/app.app/Contents/Framework/v1/` और `/application/app.app/Contents/Framework/v2/` है। तो दोनों फोल्डर `library.dylib` को लोड करने के लिए उपयोग किए जाएंगे। अगर पुस्तकालय `[...]/v1/` में मौजूद नहीं है और हमलावर उसे वहां रख सकता है ताकि पुस्तकालय के लोड को `[...]/v2/` में हाइजैक किया जा सके क्योंकि **`LC_LOAD_DYLIB`** में मार्गों का क्रम अनुसरण किया जाता है।
* `otool -l </path/to/binary> | grep -E "LC_RPATH|LC_LOAD_DYLIB" -A 5` के साथ बाइनरी में **rpath मार्ग और पुस्तकालय** खोजें।

{% hint style="info" %}
**`@executable_path`**: **मुख्य कार्यक्रम फ़ाइल** को समेत करने वाले **निर्देशिका** का **मार्ग** है।

**`@loader_path`**: **Mach-O बाइनरी** को समेत करने वाले **निर्देशिका** का **मार्ग** है जिसमें लोड कमांड है।

* जब किसी कार्यक्रम में उपयोग किया जाता है, तो **`@loader_path`** वास्तव में **`@executable_path`** के समान होता है।
* जब किसी **dylib** में उपयोग किया जाता है, तो **`@loader_path`** द्वितीय **dylib** का **मार्ग** देता है।
{% endhint %}

इस कार्यक्षमता का दुरुपयोग करके **वर्चस्व उन्नति** का तरीका वह हो सकता है जब **किसी** **रूट** द्वारा **चलाई जा रही एप्लिकेशन** को किसी **फ़ोल्डर में कुछ पुस्तकालय खोजने की आवश्यकता है जहां हमलावर को लेखन अनुमतियाँ हैं।

{% hint style="success" %}
एक अच्छा **स्कैनर** जो एप्लिकेशन में **गायब पुस्तकालयों** को खोजने के लिए है [**Dylib Hijack Scanner**](https://objective-see.com/products/dhs.html) या एक [**CLI संस्करण**](https://github.com/pandazheng/DylibHijack)।\
इस तकनीक के बारे में एक अच्छी **रिपोर्ट तकनीकी विवरण** यहाँ मिल सकता है [**यहाँ**](https://www.virusbulletin.com/virusbulletin/2015/03/dylib-hijacking-os-x)।
{% endhint %}

**उदाहरण**

{% content-ref url="macos-dyld-hijacking-and-dyld_insert_libraries.md" %}
[macos-dyld-hijacking-and-dyld\_insert\_libraries.md](macos-dyld-hijacking-and-dyld\_insert\_libraries.md)
{% endcontent-ref %}

## Dlopen Hijacking

{% hint style="danger" %}
ध्यान रखें कि **पिछली पुस्तकालय मान्यता प्रतिबंध** भी Dlopen हाइजैकिंग हमलों को करने के लिए लागू होती है।
{% endhint %}

**`man dlopen`** से:

* जब पथ में **कोई स्लैश वाला वर्ण नहीं होता** है (अर्थात केवल एक पत्ता नाम है), तो **dlopen() खोज करेगा**। अगर **`$DYLD_LIBRARY_PATH`** लॉन्च पर सेट किया गया था, तो dyld पहले **उस निर्देशिका में देखेगा**। अगले, यदि कॉलिंग mach-o फ़ाइल या मुख्य कार्यक्रम निर्देशित करते हैं **`LC_RPATH`**, तो dyld **उन निर्देशिकाओं में देखेगा**। अगले, यदि प्रक्रिया **असीमित** है, तो dyld **वर्तमान काम करने वाले निर्देशिका में खोजेगा**। अंततः, पुराने बाइनरी के लिए, dyld कुछ फॉलबैक कोशिश करेगा। यदि **`$DYLD_FALLBACK_LIBRARY_PATH`** लॉन्च पर सेट किया गया था, तो dyld **उन निर्देशिकाओं में देखेगा**, अन्यथा, dyld **`/usr/local/lib/`** में देखेगा (यदि प्रक्रिया असीमित है), और फिर **`/usr/lib/`** में देखेगा।
1. `$DYLD_LIBRARY_PATH`
2. `LC_RPATH`
3. `CWD`(असीमित होने पर)
4. `$DYLD_FALLBACK_LIBRARY_PATH`
5. `/usr/local/lib/` (असीमित होने पर)
6. `/usr/lib/`

{% hint style="danger" %}
नाम में कोई स्लैश नहीं है, तो हाइजैकिंग करने के दो तरीके हो सकते हैं:

* यदि कोई **`LC_RPATH`** **लिखने योग्य** है (लेकिन हस्ताक्षर की जांच होती है, इसलिए इसके लिए आपको बाइनरी को असीमित करने की भी आवश्यकता है)
* यदि बाइनरी **असीमित** है और फिर CWD से कुछ लोड करना संभव है (या उपरोक्त env चरणों में से किसी एक का दुरुपयोग करना)
{% endhint %}

* जब पथ **एक फ्रेमवर्क की तरह दिखता है** (उदाहरण के लिए `/stuff/foo.framework/foo`), अगर **`$DYLD_FRAMEWORK_PATH`** लॉन्च पर सेट किया गया था, तो dyld पहले उस निर्देशिका में देखेगा **फ्रेमवर्क आंशिक पथ** (उदाहरण के लिए `foo.framework/foo`)। अगले, dyld **प्रदान किया गया पथ को जैसा है** (वर्तमान काम करने वाले निर्देशिका का उपयोग करते हुए)। अंततः, पुराने बाइनरी के लिए, dyld कुछ फॉलबैक कोशिश करेगा। यदि **`$DYLD_FALLBACK_FRAMEWORK_PATH`** लॉन्च पर सेट किया गया था, तो dyld **उन निर्देशिकाओं में देखेगा**। अन्यथा, यह **`/Library/Frameworks`** में देखेगा (macOS पर यदि प्रक्रिया असीमित है), फिर **`/System/Library/Frameworks`** में।
1. `$DYLD_FRAMEWORK_PATH`
2. प्रदान किया गया पथ (असीमित होने पर उपयोग करने के लिए वर्तमान काम करने वाले निर्देशिका)
3. `$DYLD_FALLBACK_FRAMEWORK_PATH`
4. `/Library/Frameworks` (असीमित होने पर)
5. `/System/Library/Frameworks`

{% hint style="danger" %}
अगर एक फ्रेमवर्क पथ है, तो उसे हाइजैक करने का तरीका हो सकता है:

* यदि प्रक्रिया **असीमित** है, CWD से **निर्देशित पथ** का दुरुपयोग करना (यहाँ तक कि यदि दस्तावेज़ में नहीं कहा गया है कि प्रक्रिया प्रतिबंधित है तो DYLD\_\* env चरण हटा दिए जाते हैं)
{% endhint %}

* जब पथ **एक स्लैश शामिल करता है लेकिन फ्रेमवर्क पथ नहीं है** (अर्थात एक पूर्ण पथ या एक dylib के लिए आंशिक पथ), dlopen() पहले (यदि सेट किया गया है) **`$DYLD_LIBRARY_PATH`** में देखेगा (पथ के पत्ते के साथ)। अगले, dyld **प्रदान किया गया पथ को जैसा है** (वर्तमान काम करने वाले निर्देशिका का उपयोग करते हुए)। अंततः, पुराने बाइनरी के लिए, dyld कुछ फॉलबैक कोशिश करेगा। यदि **`$DYLD_FALLBACK_LIBRARY_PATH`** लॉन्च पर सेट किया गया था, तो dyld **उन निर्देशिकाओं में देखेगा**, अन्यथा, dyld **`/usr/local/lib/`** में देखेगा (यदि प्रक्रिया असीमित है), और फिर **`/usr/lib/`** में देखेगा।
1. `$DYLD_LIBRARY_PATH`
2. प्रदान किया गया पथ (असीमित होने पर उपयोग करने के लिए वर्तमान काम करने वाले निर्देशिका)
3. `$DYLD_FALLBACK_LIBRARY_PATH`
4. `/usr/local/lib/` (असीमित होने पर)
5. `/usr/lib/`

{% hint style="danger" %}
नाम में स्लैश है और फ्रेमवर्क नहीं है, तो
```c
// gcc dlopentest.c -o dlopentest -Wl,-rpath,/tmp/test
#include <dlfcn.h>
#include <stdio.h>

int main(void)
{
void* handle;

fprintf("--- No slash ---\n");
handle = dlopen("just_name_dlopentest.dylib",1);
if (!handle) {
fprintf(stderr, "Error loading: %s\n\n\n", dlerror());
}

fprintf("--- Relative framework ---\n");
handle = dlopen("a/framework/rel_framework_dlopentest.dylib",1);
if (!handle) {
fprintf(stderr, "Error loading: %s\n\n\n", dlerror());
}

fprintf("--- Abs framework ---\n");
handle = dlopen("/a/abs/framework/abs_framework_dlopentest.dylib",1);
if (!handle) {
fprintf(stderr, "Error loading: %s\n\n\n", dlerror());
}

fprintf("--- Relative Path ---\n");
handle = dlopen("a/folder/rel_folder_dlopentest.dylib",1);
if (!handle) {
fprintf(stderr, "Error loading: %s\n\n\n", dlerror());
}

fprintf("--- Abs Path ---\n");
handle = dlopen("/a/abs/folder/abs_folder_dlopentest.dylib",1);
if (!handle) {
fprintf(stderr, "Error loading: %s\n\n\n", dlerror());
}

return 0;
}
```
यदि आप इसे कंपाइल और निष्पादित करते हैं तो आप **हर पुस्तकालय को असफलतापूर्वक खोजा गया था कहाँ** देख सकते हैं। इसके अलावा, आप **एफएस लॉग को फ़िल्टर कर सकते हैं**:
```bash
sudo fs_usage | grep "dlopentest"
```
## सांख्यिक मार्ग हाइजैकिंग

यदि एक **विशेषाधिकारी बाइनरी/एप्लिकेशन** (जैसे SUID या कुछ शक्तिशाली entitlements वाला बाइनरी) **एक सांख्यिक मार्ग** पुस्तकालय को लोड कर रहा है (उदाहरण के लिए `@executable_path` या `@loader_path` का उपयोग करके) और **पुस्तकालय सत्यापन अक्षम है**, तो हमें बाइनरी को एक स्थान पर मूव करने की संभावना है जहां हमें आक्रमणकारी को **संबंधित मार्ग लोड की गई पुस्तकालय में संशोधित** करने की अनुमति हो, और इसे प्रक्रिया पर कोड इंजेक्ट करने के लिए दुरुपयोग कर सकते हैं।

## `DYLD_*` और `LD_LIBRARY_PATH` env चरों को काटना

फ़ाइल `dyld-dyld-832.7.1/src/dyld2.cpp` में फ़ंक्शन **`pruneEnvironmentVariables`** को खोजना संभव है, जो किसी भी env चर को हटा देगा जो **`DYLD_`** से **शुरू होता है** और **`LD_LIBRARY_PATH=`**।

यह विशेष रूप से **suid** और **sgid** बाइनरी के लिए env चर **`DYLD_FALLBACK_FRAMEWORK_PATH`** और **`DYLD_FALLBACK_LIBRARY_PATH`** को **null** पर सेट करेगा।

यह फ़ंक्शन उसी फ़ाइल के **`_main`** फ़ंक्शन से ओएसएक्स को लक्षित करते हुए बुलाया जाता है:
```cpp
#if TARGET_OS_OSX
if ( !gLinkContext.allowEnvVarsPrint && !gLinkContext.allowEnvVarsPath && !gLinkContext.allowEnvVarsSharedCache ) {
pruneEnvironmentVariables(envp, &apple);
```
और वे बूलियन फ्लैग्स कोड में एक ही फ़ाइल में सेट किए जाते हैं:
```cpp
#if TARGET_OS_OSX
// support chrooting from old kernel
bool isRestricted = false;
bool libraryValidation = false;
// any processes with setuid or setgid bit set or with __RESTRICT segment is restricted
if ( issetugid() || hasRestrictedSegment(mainExecutableMH) ) {
isRestricted = true;
}
bool usingSIP = (csr_check(CSR_ALLOW_TASK_FOR_PID) != 0);
uint32_t flags;
if ( csops(0, CS_OPS_STATUS, &flags, sizeof(flags)) != -1 ) {
// On OS X CS_RESTRICT means the program was signed with entitlements
if ( ((flags & CS_RESTRICT) == CS_RESTRICT) && usingSIP ) {
isRestricted = true;
}
// Library Validation loosens searching but requires everything to be code signed
if ( flags & CS_REQUIRE_LV ) {
isRestricted = false;
libraryValidation = true;
}
}
gLinkContext.allowAtPaths                = !isRestricted;
gLinkContext.allowEnvVarsPrint           = !isRestricted;
gLinkContext.allowEnvVarsPath            = !isRestricted;
gLinkContext.allowEnvVarsSharedCache     = !libraryValidation || !usingSIP;
gLinkContext.allowClassicFallbackPaths   = !isRestricted;
gLinkContext.allowInsertFailures         = false;
gLinkContext.allowInterposing         	 = true;
```
जिसका मतलब है कि यदि बाइनरी **suid** या **sgid** है, या हेडर्स में **RESTRICT** सेगमेंट है या यह **CS\_RESTRICT** फ्लैग के साथ साइन किया गया है, तो **`!gLinkContext.allowEnvVarsPrint && !gLinkContext.allowEnvVarsPath && !gLinkContext.allowEnvVarsSharedCache`** सत्य है और env variables को काट दिया जाता है।

ध्यान दें कि यदि CS\_REQUIRE\_LV सत्य है, तो चरणों को काटा नहीं जाएगा लेकिन पुस्तकालय मान्यता जांचेगी कि वे मूल बाइनरी के साथ समान प्रमाणपत्र का उपयोग कर रहे हैं।

## प्रतिबंधों की जांच

### SUID और SGID
```bash
# Make it owned by root and suid
sudo chown root hello
sudo chmod +s hello
# Insert the library
DYLD_INSERT_LIBRARIES=inject.dylib ./hello

# Remove suid
sudo chmod -s hello
```
### खंड `__RESTRICT` सेगमेंट `__restrict` के साथ
```bash
gcc -sectcreate __RESTRICT __restrict /dev/null hello.c -o hello-restrict
DYLD_INSERT_LIBRARIES=inject.dylib ./hello-restrict
```
### हार्डन्ड रनटाइम

कीचेन में एक नया प्रमाणपत्र बनाएं और इसका उपयोग बाइनरी को साइन करने के लिए करें:
```bash
# Apply runtime proetction
codesign -s <cert-name> --option=runtime ./hello
DYLD_INSERT_LIBRARIES=inject.dylib ./hello #Library won't be injected

# Apply library validation
codesign -f -s <cert-name> --option=library ./hello
DYLD_INSERT_LIBRARIES=inject.dylib ./hello-signed #Will throw an error because signature of binary and library aren't signed by same cert (signs must be from a valid Apple-signed developer certificate)

# Sign it
## If the signature is from an unverified developer the injection will still work
## If it's from a verified developer, it won't
codesign -f -s <cert-name> inject.dylib
DYLD_INSERT_LIBRARIES=inject.dylib ./hello-signed

# Apply CS_RESTRICT protection
codesign -f -s <cert-name> --option=restrict hello-signed
DYLD_INSERT_LIBRARIES=inject.dylib ./hello-signed # Won't work
```
{% endcode %}

{% hint style="danger" %}
ध्यान दें कि यदि किसी भी बाइनरी के साथ झंडे **`0x0(none)`** के साथ हैं, तो जब वे निष्पादित होते हैं तो वे डायनामिक रूप से **`CS_RESTRICT`** झंडा प्राप्त कर सकते हैं और इसलिए यह तकनीक उनमें काम नहीं करेगी।

आप यह झंडा किसी प्रोसेस में है या नहीं इसे चेक कर सकते हैं (यहाँ [**csops यहाँ**](https://github.com/axelexic/CSOps) से प्राप्त करें):
```bash
csops -status <pid>
```
and then check if the flag 0x800 is enabled.
{% endhint %}

## संदर्भ

* [https://theevilbit.github.io/posts/dyld\_insert\_libraries\_dylib\_injection\_in\_macos\_osx\_deep\_dive/](https://theevilbit.github.io/posts/dyld\_insert\_libraries\_dylib\_injection\_in\_macos\_osx\_deep\_dive/)
* [**\*OS Internals, Volume I: User Mode. By Jonathan Levin**](https://www.amazon.com/MacOS-iOS-Internals-User-Mode/dp/099105556X)

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी HackTricks में विज्ञापित हो** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **जुड़ें** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>
