# macOS पुस्तकालय इंजेक्शन

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert)</strong> के साथ सीखें एडब्ल्यूएस हैकिंग को शून्य से हीरो तक!</summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी कंपनी का विज्ञापन **HackTricks** में देखना चाहते हैं या **PDF में HackTricks डाउनलोड** करना चाहते हैं तो [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह, **The PEASS Family**, खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मेरा** ट्विटर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** का** **पालन** करें।
* **हैकिंग ट्रिक्स साझा करें** द्वारा PRs सबमिट करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में।

</details>

{% hint style="danger" %}
**dyld कोड ओपन सोर्स** है और इसे [https://opensource.apple.com/source/dyld/](https://opensource.apple.com/source/dyld/) पर पाया जा सकता है और एक **URL जैसे** [https://opensource.apple.com/tarballs/dyld/dyld-852.2.tar.gz](https://opensource.apple.com/tarballs/dyld/dyld-852.2.tar.gz) का उपयोग करके डाउनलोड किया जा सकता है।
{% endhint %}

## **DYLD\_INSERT\_LIBRARIES**

यह लिनक्स पर **LD\_PRELOAD** की तरह है। यह एक प्रक्रिया को सूचित करने की अनुमति देता है कि एक विशिष्ट पुस्तकालय को एक मार्ग से लोड करने के लिए चलाया जाएगा (यदि env चर चालू है)

यह तकनीक एक ASEP तकनीक के रूप में भी उपयोग किया जा सकता है क्योंकि हर एप्लिकेशन जिसे स्थापित किया गया है, के पास "Info.plist" नामक एक plist होता है जो `LSEnvironmental` नामक एक कुंजी का उपयोग करके पर्यावरणीय चरों का निरुपण करने की अनुमति देता है।

{% hint style="info" %}
2012 से **Apple ने DYLD_INSERT_LIBRARIES** की **शक्ति को बहुत कम कर दिया है**।

कोड पर जाएं और **`src/dyld.cpp`** की जांच करें। फ़ंक्शन **`pruneEnvironmentVariables`** में आप देख सकते हैं कि **`DYLD_*`** चर हटाए गए हैं।

फ़ंक्शन **`processRestricted`** में प्रतिबंध का कारण सेट किया गया है। उस कोड की जांच करते समय आप देख सकते हैं कि कारण हैं:

* बाइनरी `setuid/setgid` है
* मैचो बाइनरी में `__RESTRICT/__restrict` खंड का मौजूद होना।
* सॉफ़्टवेयर में entitlements (हार्डनेड रनटाइम) हैं बिना [`com.apple.security.cs.allow-dyld-environment-variables`](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_allow-dyld-environment-variables) entitlement
* एक बाइनरी की **entitlements** की जांच करें: `codesign -dv --entitlements :- </path/to/bin>`

अधिक अद्यतित संस्करणों में आप इस तरह की तर्क को **`configureProcessRestrictions`** फ़ंक्शन के दूसरे हिस्से में पा सकते हैं। हालांकि, नए संस्करणों में जो कुछ नया है, वह फ़ंक्शन की **शुरुआती जांचें** हैं (आप iOS या सिमुलेशन से संबंधित ifs को हटा सकते हैं क्योंकि वे macOS में उपयोग नहीं होंगे)।
{% endhint %}

### पुस्तकालय सत्यापन

यदि बाइनरी **`DYLD_INSERT_LIBRARIES`** env चर का उपयोग करने की अनुमति देता है, तो यदि बाइनरी पुस्तकालय के हस्ताक्षर की जांच करता है तो एक अनुकूल वह नहीं लोडेगा।

एक अनुकूल पुस्तकालय लोड करने के लिए बाइनरी के पास निम्नलिखित entitlements होने चाहिए:

* &#x20;[`com.apple.security.cs.disable-library-validation`](../../macos-security-protections/macos-dangerous-entitlements.md#com.apple.security.cs.disable-library-validation)
* [`com.apple.private.security.clear-library-validation`](../../macos-security-protections/macos-dangerous-entitlements.md#com.apple.private.security.clear-library-validation)

या तो बाइनरी के पास **हार्डनेड रनटाइम झंडा** नहीं होना चाहिए या **पुस्तकालय सत्यापन झंडा** नहीं होना चाहिए।

आप यह जांच सकते हैं कि बाइनरी में **हार्डनेड रनटाइम** है या नहीं `codesign --display --verbose <bin>` का उपयोग करके **`CodeDirectory`** में झंडा रनटाइम की जांच करें जैसे: **`CodeDirectory v=20500 size=767 flags=0x10000(runtime) hashes=13+7 location=embedded`**

आप एक पुस्तकालय भी लोड कर सकते हैं यदि यह **बाइनरी के साथ समान प्रमाणपत्र से हस्ताक्षरित** है।

इसका उपयोग कैसे करने के लिए एक उदाहरण खोजें और नियमों की जांच करें:

{% content-ref url="../../macos-dyld-hijacking-and-dyld_insert_libraries.md" %}
[macos-dyld-hijacking-and-dyld\_insert\_libraries.md](../../macos-dyld-hijacking-and-dyld\_insert\_libraries.md)
{% endcontent-ref %}

## Dylib हाइजैकिंग

{% hint style="danger" %}
ध्यान दें कि **पिछले पुस्तकालय सत्यापन प्रतिबंध** भी Dylib हाइजैकिंग हमलों को करने के लिए लागू होते हैं।
{% endhint %}

विंडोज में, मैकओएस में भी आप **dylibs को हाइजैक** कर सकते हैं ताकि **एप्लिकेशन** **कोड** **को** **अर्बिट्रे** **करें** (ठीक है, वास्तव में एक सामान्य उपयोगकर्ता से यह संभव नहीं हो सकता क्योंकि आपको एक TCC अनुमति की आवश्यकता हो सकती है एक `.app` बंडल में लिखने के लिए और एक पुस्तकालय हाइजैक करने के लिए)।\
हालांकि, **मैकओएस** एप्लिकेशन **पुस्तकालयों को लोड** करने का तरीका **विंडोज** से **अधिक प्रतिबंधित** है। इसका अर्थ है कि **मैलवेयर** विकासक इस तकनीक का उपयोग **छुपाव** के लिए कर सकते हैं, लेकिन यहाँ **उच्च स्तर** पर **इसे उच्चारित करने की संभावना कम है**।

सबसे पहले, **अधिक सामान्य** है कि **मैकओएस बाइनरी** लाइब्रेरीज़ के पूरे मार्ग को दिखाते हैं। और दूसरा, **मैकओएस कभी नहीं खोजता** है **$PATH** के फोल्डर में पुस्तकालयों के लिए।

इस कार्यक्षेत्र से संबंधित **मुख्य** हिस्सा **`ImageLoader::recursiveLoadLibraries`** में `ImageLoader.cpp` में है।

मैचो बाइनरी एक पुस्तकालय लोड करने के लिए उपयोग कर सकते हैं **4 विभिन्न हेडर कमांड** हैं:

* **`LC_LOAD_DYLIB`** कमांड एक dylib लोड करने के लिए सामान्य कमांड है।
* **`LC_LOAD_WEAK_DYLIB`** कमांड पिछले के जैसा काम करता है, लेकिन यदि dylib नहीं मिलता है, तो कोई त्रुटि के बिना चलाना जारी रहेगा।
* **`LC_REEXPORT_DYLIB`** कमांड यह प्रॉक्सी करता है (या पुनः निर्यात करता है) विभिन्न पुस्तकालयों से प्रतीकों को।
* **`LC_LOAD_UPWARD_DYLIB`** कमांड जब दो पुस्तकालय एक-दूसरे पर निर्भर होते हैं (इसे _ऊपरी निर्भरता_ कहा जाता है)।

हालांकि, **2 प्रकार की dylib हाइजैकिंग** है:

* **लापता कमजोर लिंक पुस्तकालय**: इसका मतलब है कि एप्लिकेशन एक पुस्तकालय को लोड करने की क
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

यदि एक **विशेषाधिकारी बाइनरी/एप्लिकेशन** (जैसे SUID या कुछ शक्तिशाली entitlements वाला बाइनरी) **एक सांख्यिक मार्ग** पुस्तकालय को लोड कर रहा है (उदाहरण के लिए `@executable_path` या `@loader_path` का उपयोग करके) और **पुस्तकालय सत्यापन अक्षम** है, तो यह संभव है कि हम बाइनरी को एक स्थान पर मूव कर सकते हैं जहां हम आक्रमणकारी सकते हैं और प्रक्रिया पर कोड इंजेक्ट कर सकते हैं।

## `DYLD_*` और `LD_LIBRARY_PATH` env चरों को काटना

फ़ाइल `dyld-dyld-832.7.1/src/dyld2.cpp` में हमें फ़ंक्शन **`pruneEnvironmentVariables`** मिल सकता है, जो किसी भी env चर को हटा देगा जो **`DYLD_`** से शुरू होता है और **`LD_LIBRARY_PATH=`**।

यह भी **suid** और **sgid** बाइनरी के लिए विशेष रूप से env चर **`DYLD_FALLBACK_FRAMEWORK_PATH`** और **`DYLD_FALLBACK_LIBRARY_PATH`** को **null** पर सेट करेगा।

यह फ़ंक्शन यदि OSX को लक्ष्य बनाते हुए फ़ाइल के **`_main`** फ़ंक्शन से बुलाया जाता है:
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
जिसका मतलब है कि यदि बाइनरी **suid** या **sgid** है, या हेडर में **RESTRICT** सेगमेंट है या यह **CS\_RESTRICT** ध्वज के साथ साइन किया गया है, तो **`!gLinkContext.allowEnvVarsPrint && !gLinkContext.allowEnvVarsPath && !gLinkContext.allowEnvVarsSharedCache`** सत्य है और env variables को काट दिया जाता है।

ध्यान दें कि यदि CS\_REQUIRE\_LV सत्य है, तो चरणों को काटा नहीं जाएगा लेकिन पुस्तकालय मान्यता की जांच करेगी कि वे मूल बाइनरी के साथ समान प्रमाणपत्र का उपयोग कर रहे हैं।

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
ध्यान दें कि यदि किसी भी बाइनरी के साइन फ्लैग **`0x0(none)`** के साथ हों, तो जब उन्हें निष्पादित किया जाता है तो वे डायनामिक रूप से **`CS_RESTRICT`** फ्लैग प्राप्त कर सकते हैं और इसलिए यह तकनीक उनमें काम नहीं करेगी।

आप यह जांच सकते हैं कि क्या किसी प्रोसेस में इस फ्लैग के साथ है या नहीं (यहां [**csops यहां**](https://github.com/axelexic/CSOps) से प्राप्त करें):&#x20;
```bash
csops -status <pid>
```
## संदर्भ
* [https://theevilbit.github.io/posts/dyld_insert_libraries_dylib_injection_in_macos_osx_deep_dive/](https://theevilbit.github.io/posts/dyld_insert_libraries_dylib_injection_in_macos_osx_deep_dive/)

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की हमारी विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह की खोज करें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** द्वारा **PRs सबमिट** करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में।

</details>
