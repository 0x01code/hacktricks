# macOS पुस्तकालय इंजेक्शन

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से नायक तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS रेड टीम विशेषज्ञ)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग प्राप्त करें**](https://peass.creator-spring.com)
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा [**NFTs**](https://opensea.io/collection/the-peass-family) का विशेष संग्रह
* 💬 [**Discord समूह में शामिल हों**](https://discord.gg/hRep4RUj7f) या [**telegram समूह**](https://t.me/peass) या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) **का अनुसरण करें**.
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग तरकीबें साझा करें।

</details>

{% hint style="danger" %}
**dyld का कोड ओपन सोर्स है** और इसे [https://opensource.apple.com/source/dyld/](https://opensource.apple.com/source/dyld/) में पाया जा सकता है और इसे एक **URL का उपयोग करके** एक tar के रूप में डाउनलोड किया जा सकता है [https://opensource.apple.com/tarballs/dyld/dyld-852.2.tar.gz](https://opensource.apple.com/tarballs/dyld/dyld-852.2.tar.gz)
{% endhint %}

## **DYLD\_INSERT\_LIBRARIES**

> यह एक कोलोन से अलग **डायनामिक पुस्तकालयों की सूची** है जिसे **प्रोग्राम में निर्दिष्ट पुस्तकालयों से पहले लोड किया जाता है**। यह आपको फ्लैट-नेमस्पेस इमेजेस में प्रयुक्त डायनामिक शेयर्ड पुस्तकालयों के मौजूदा मॉड्यूल के नए मॉड्यूल का परीक्षण करने की अनुमति देता है, जिसमें केवल नए मॉड्यूल के साथ एक अस्थायी डायनामिक शेयर्ड पुस्तकालय लोड होता है। ध्यान दें कि इसका कोई प्रभाव नहीं होता है उन इमेजेस पर जो दो-स्तरीय नेमस्पेस इमेजेस का उपयोग करते हैं जब तक कि DYLD\_FORCE\_FLAT\_NAMESPACE का भी उपयोग नहीं किया जाता।

यह [**LD\_PRELOAD on Linux**](../../../../linux-hardening/privilege-escalation#ld\_preload) की तरह है।

यह तकनीक **ASEP तकनीक के रूप में भी उपयोग की जा सकती है** क्योंकि हर एप्लिकेशन में एक plist होती है जिसे "Info.plist" कहा जाता है जो `LSEnvironmental` कहे जाने वाले कुंजी का उपयोग करके **पर्यावरणीय चर को असाइन करने की अनुमति देती है**।

{% hint style="info" %}
2012 के बाद से **Apple ने `DYLD_INSERT_LIBRARIES` की शक्ति को काफी कम कर दिया है**।

कोड पर जाएं और **`src/dyld.cpp` की जांच करें**। **`pruneEnvironmentVariables`** फ़ंक्शन में आप देख सकते हैं कि **`DYLD_*`** चर हटा दिए गए हैं।

**`processRestricted`** फ़ंक्शन में प्रतिबंध का कारण सेट किया गया है। उस कोड की जांच करके आप देख सकते हैं कि कारण हैं:

* बाइनरी `setuid/setgid` है
* मैको बाइनरी में `__RESTRICT/__restrict` सेक्शन का अस्तित्व है।
* सॉफ्टवेयर में एंटाइटलमेंट्स (हार्डन्ड रनटाइम) हैं बिना [`com.apple.security.cs.allow-dyld-environment-variables`](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_allow-dyld-environment-variables) एंटाइटलमेंट के
* बाइनरी के **एंटाइटलमेंट्स की जांच करें** `codesign -dv --entitlements :- </path/to/bin>`

अधिक अपडेटेड संस्करणों में आप इस तर्क को फ़ंक्शन **`configureProcessRestrictions`** के दूसरे भाग में पा सकते हैं। हालांकि, नए संस्करणों में जो कुछ भी किया जाता है वह फ़ंक्शन की **शुरुआती जांच है** (आप iOS या सिमुलेशन से संबंधित ifs को हटा सकते हैं क्योंकि वे macOS में उपयोग नहीं किए जाएंगे।
{% endhint %}

### पुस्तकालय मान्यता

यहां तक कि अगर बाइनरी **`DYLD_INSERT_LIBRARIES`** env चर का उपयोग करने की अनुमति देती है, अगर बाइनरी लोड करने के लिए पुस्तकालय के हस्ताक्षर की जांच करती है तो वह कस्टम क्या नहीं लोड करेगी।

कस्टम पुस्तकालय लोड करने के लिए, बाइनरी के पास **निम्नलिखित में से एक एंटाइटलमेंट होना चाहिए**:

* &#x20;[`com.apple.security.cs.disable-library-validation`](../../macos-security-protections/macos-dangerous-entitlements.md#com.apple.security.cs.disable-library-validation)
* [`com.apple.private.security.clear-library-validation`](../../macos-security-protections/macos-dangerous-entitlements.md#com.apple.private.security.clear-library-validation)

या बाइनरी के पास **हार्डन्ड रनटाइम फ्लैग** या **पुस्तकालय मान्यता फ्लैग** नहीं होना चाहिए।

आप जांच सकते हैं कि क्या एक बाइनरी में **हार्डन्ड रनटाइम** है `codesign --display --verbose <bin>` के साथ **`CodeDirectory`** में फ्लैग रनटाइम की जांच करके जैसे: **`CodeDirectory v=20500 size=767 flags=0x10000(runtime) hashes=13+7 location=embedded`**

आप यह भी लोड कर सकते हैं अगर यह **बाइनरी के रूप में ही प्रमाणपत्र के साथ हस्ताक्षरित है**।

इसका उपयोग कैसे करें और प्रतिबंधों की जांच कैसे करें, इसका उदाहरण यहां पाएं:

{% content-ref url="../../macos-dyld-hijacking-and-dyld_insert_libraries.md" %}
[macos-dyld-hijacking-and-dyld\_insert\_libraries.md](../../macos-dyld-hijacking-and-dyld\_insert\_libraries.md)
{% endcontent-ref %}

## Dylib हाइजैकिंग

{% hint style="danger" %}
याद रखें कि **पिछली पुस्तकालय मान्यता प्रतिबंध भी लागू होते हैं** Dylib हाइजैकिंग हमलों को करने के लिए।
{% endhint %}

Windows की तरह, MacOS में भी आप **dylibs को हाइजैक** कर सकते हैं ताकि **एप्लिकेशन** **मनमाने** **कोड को निष्पादित** करें।\
हालांकि, MacOS एप्लिकेशन द्वारा पुस्तकालयों को **लोड** करने का तरीका Windows की तुलना में **अधिक प्रतिबंधित** है। इसका मतलब है कि **मैलवेयर** डेवलपर्स अभी भी इस तकनीक का उपयोग **चुपके** के लिए कर सकते हैं, लेकिन विशेषाधिकार बढ़ाने के लिए इसका दुरुपयोग करने की संभावना बहुत कम है।

सबसे पहले, यह **अधिक सामान्य** है कि MacOS बाइनरी लोड करने के लिए पुस्तकालयों के पूर्ण पथ को इंगित करती है। और दूसरा, **MacOS कभी भी $PATH के फोल्डरों में पुस्तकालयों की खोज नहीं करता है**।

**मुख्य** भाग **कोड** से संबंधित इस कार्यक्षमता का है **`ImageLoader::recursiveLoadLibraries`** `ImageLoader.cpp` में।

मैको बाइनरी द्वारा पुस्तकालयों को लोड करने के लिए **4 अलग-अलग हेडर कमांड्स** हो सकते हैं:

* **`LC_LOAD_DYLIB`** कमांड एक डायलिब को लोड करने के लिए सामान्य कमांड है।
* **`LC_LOAD_WEAK_DYLIB`** कमांड पिछले एक की तरह काम करता है, लेकिन अगर डायलिब नहीं मिलती है, तो निष्पादन किसी भी त्रुटि के बिना जारी रहता है।
* **
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
यदि आप इसे कंपाइल करके निष्पादित करते हैं, तो आप देख सकते हैं **प्रत्येक लाइब्रेरी को असफलतापूर्वक कहाँ खोजा गया था**। साथ ही, आप **FS लॉग्स को फ़िल्टर** कर सकते हैं:
```bash
sudo fs_usage | grep "dlopentest"
```
## सापेक्ष पथ हाइजैकिंग

यदि कोई **विशेषाधिकार प्राप्त बाइनरी/ऐप** (जैसे कि SUID या कुछ बाइनरी जिसमें शक्तिशाली एंटाइटलमेंट्स होते हैं) एक सापेक्ष पथ लाइब्रेरी (उदाहरण के लिए `@executable_path` या `@loader_path` का उपयोग करके) लोड कर रहा है और इसमें **लाइब्रेरी वैलिडेशन अक्षम** है, तो संभव है कि बाइनरी को उस स्थान पर ले जाया जा सकता है जहां हमलावर सापेक्ष पथ लोडेड लाइब्रेरी को **संशोधित कर सकता है**, और प्रक्रिया में कोड इंजेक्ट करने के लिए इसका दुरुपयोग कर सकता है।

## `DYLD_*` और `LD_LIBRARY_PATH` env वेरिएबल्स को प्रून करें

फाइल `dyld-dyld-832.7.1/src/dyld2.cpp` में **`pruneEnvironmentVariables`** नामक फंक्शन को ढूंढा जा सकता है, जो किसी भी env वेरिएबल को हटा देगा जो **`DYLD_` के साथ शुरू होता है** और **`LD_LIBRARY_PATH=`**.

यह **suid** और **sgid** बाइनरीज के लिए विशेष रूप से env वेरिएबल्स **`DYLD_FALLBACK_FRAMEWORK_PATH`** और **`DYLD_FALLBACK_LIBRARY_PATH`** को भी **null** पर सेट कर देगा।

यह फंक्शन उसी फाइल के **`_main`** फंक्शन से बुलाया जाता है यदि OSX को इस तरह टारगेट किया जा रहा हो:
```cpp
#if TARGET_OS_OSX
if ( !gLinkContext.allowEnvVarsPrint && !gLinkContext.allowEnvVarsPath && !gLinkContext.allowEnvVarsSharedCache ) {
pruneEnvironmentVariables(envp, &apple);
```
और वे बूलियन फ्लैग्स उसी फाइल में कोड में सेट किए जाते हैं:
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
## प्रतिबंधों की जाँच करें

### SUID & SGID
```bash
# Make it owned by root and suid
sudo chown root hello
sudo chmod +s hello
# Insert the library
DYLD_INSERT_LIBRARIES=inject.dylib ./hello

# Remove suid
sudo chmod -s hello
```
### अनुभाग `__RESTRICT` सेगमेंट `__restrict` के साथ
```bash
gcc -sectcreate __RESTRICT __restrict /dev/null hello.c -o hello-restrict
DYLD_INSERT_LIBRARIES=inject.dylib ./hello-restrict
```
### हार्डन्ड रनटाइम

कीचेन में एक नया प्रमाणपत्र बनाएं और उसका उपयोग बाइनरी को साइन करने के लिए करें:

{% code overflow="wrap" %}
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
ध्यान दें कि यदि कोई बाइनरी **`0x0(none)`** फ्लैग के साथ साइन की गई है, तो भी उन्हें निष्पादित होने पर डायनामिक रूप से **`CS_RESTRICT`** फ्लैग मिल सकता है और इसलिए यह तकनीक उन पर काम नहीं करेगी।

आप यह जांच सकते हैं कि किसी प्रोसेस में यह फ्लैग है या नहीं (get [**csops here**](https://github.com/axelexic/CSOps)):&#x20;
```bash
csops -status <pid>
```
और फिर जांचें कि क्या 0x800 फ्लैग सक्षम है।
{% endhint %}

<details>

<summary><strong>शून्य से नायक तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) **का पालन करें**।
* **HackTricks** के [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>
