# macOS लाइब्रेरी इंजेक्शन

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एकल [**NFT संग्रह**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल हों** या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का पालन करें**.
* **अपने हैकिंग ट्रिक्स को** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में पीआर जमा करके अपना योगदान दें।**

</details>

{% hint style="danger" %}
**dyld का कोड ओपन सोर्स है** और इसे [https://opensource.apple.com/source/dyld/](https://opensource.apple.com/source/dyld/) पर ढूंढा जा सकता है और एक **URL का उपयोग करके एक टार डाउनलोड किया जा सकता है** जैसे कि [https://opensource.apple.com/tarballs/dyld/dyld-852.2.tar.gz](https://opensource.apple.com/tarballs/dyld/dyld-852.2.tar.gz)
{% endhint %}

## **DYLD\_INSERT\_LIBRARIES**

> यह एक कोलन से अलग **डायनामिक लाइब्रेरी की सूची है** जो प्रोग्राम में निर्दिष्ट की गई लाइब्रेरी से पहले लोड होती है। इससे आप पहले से मौजूदा डायनामिक साझा लाइब्रेरी के नए मॉड्यूलों का परीक्षण कर सकते हैं जो फ्लैट-नेमस्पेस इमेजेज़ में उपयोग होते हैं, केवल नए मॉड्यूलों वाली एक अस्थायी डायनामिक साझा लाइब्रेरी लोड करके। ध्यान दें कि यह किसी भी प्रभाव का नहीं होता है जब एक डायनामिक साझा लाइब्रेरी का उपयोग करके दो-स्तरीय नेमस्पेस इमेजेज़ को बनाया जाता है, जब तक DYLD\_FORCE\_FLAT\_NAMESPACE भी उपयोग नहीं किया जाता है।

यह [**LD\_PRELOAD on Linux**](../../../../linux-hardening/privilege-escalation#ld\_preload) की तरह है।

यह तकनीक एक ASEP तकनीक के रूप में भी **उपयोग की जा सकती है** क्योंकि प्रत्येक स्थापित एप्लिकेशन के पास "Info.plist" नामक एक plist होती है जिसमें `LSEnvironmental` नामक एक कुंजी का उपयोग करके पर्यावरणीय चरों का निर्धारण किया जा सकता है।

{% hint style="info" %}
2012 से शुरू होकर **Apple ने DYLD\_INSERT\_LIBRARIES की शक्ति को बहुत कम कर दिया है**।

कोड पर जाएं और **`src/dyld.cpp`** की जांच करें। फ़ंक्शन **`pruneEnvironmentVariables`** में जाकर आप देख सकते हैं कि **`DYLD_*`** चरों को हटा दिया जाता है।

फ़ंक्शन **`processRestricted`** में प्रतिबंध का कारण सेट किया जाता है। उस कोड की जांच करते समय आप देख सकते हैं कि कारण हैं:

* बाइनरी `setuid/setgid` है
* मैचो बाइनरी में `__RESTRICT/__restrict` सेक्शन की मौजूदगी।
* सॉफ़्टवेयर में entitlements (हार्डनेड रनटाइम) हैं जिनमें [`com.apple.security.cs.allow-dyld-environment-variables`](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_allow-dyld-environment-variables) entitlement है
* एक बाइनरी की entitlements की जांच करें: `codesign -dv --entitlements :- </path/to/bin>`

अद्यतित संस्करणों में आप इस तर्क को फ़ंक्शन **`configureProcessRestrictions`** के दूसरे हिस्से में भी ढूंढ सकते हैं। हालांकि, नए संस्करणों में वह क्या निष्पादित करता है, वह तो फ़ंक्शन के **प्रारंभिक जांचों को हटा दिया जाता है** (आप iOS या सिम्युलेशन से संबंधित ifs को हटा सकते हैं क्योंकि वे macOS में उपयोग नहीं होंग
## Dylib हाइजैकिंग

{% hint style="danger" %}
ध्यान दें कि **पिछले लाइब्रेरी मान्यता प्रतिबंध भी लागू होती है** ताकि Dylib हाइजैकिंग हमलों को करने के लिए।
{% endhint %}

विंडोज की तरह, MacOS में आप भी **डायलिब्स को हाइजैक** कर सकते हैं ताकि **एप्लिकेशन** **विचारशील** **कोड** **चला सकें**।\
हालांकि, MacOS एप्लिकेशनों को लाइब्रेरीज़ लोड करने का तरीका विंडोज की तुलना में **अधिक प्रतिबंधित** है। इसका अर्थ है कि **मैलवेयर** विकासक इस तकनीक का उपयोग अभी भी **छिपाने** के लिए कर सकते हैं, लेकिन इसे उन्नति के लिए उपयोग करने की संभावना कम होती है।

सबसे पहले, **अधिक सामान्य है** कि **MacOS बाइनरीज़ पूरा पथ दिखाती हैं** जिसमें लोड करने के लिए लाइब्रेरीज़ की जगह दिखाई जाती है। और दूसरा, **MacOS कभी भी** **$PATH** के फ़ोल्डर में लाइब्रेरीज़ खोजता नहीं है।

इस कार्यक्षेत्र से संबंधित **कोड** का **मुख्य** हिस्सा `ImageLoader.cpp` में `ImageLoader::recursiveLoadLibraries` में है।

मैचो बाइनरी एक लाइब्रेरी लोड करने के लिए उपयोग कर सकते हैं **4 अलग-अलग हेडर कमांड** हैं:

* **`LC_LOAD_DYLIB`** कमांड एक dylib लोड करने के लिए सामान्य कमांड है।
* **`LC_LOAD_WEAK_DYLIB`** कमांड पिछले कमांड की तरह काम करता है, लेकिन यदि dylib नहीं मिलता है, तो बिना किसी त्रुटि के चलाना जारी रखा जाता है।
* **`LC_REEXPORT_DYLIB`** कमांड यह प्रॉक्सी करता है (या पुनः निर्यात करता है) एक अलग लाइब्रेरी से प्रतीक।
* **`LC_LOAD_UPWARD_DYLIB`** कमांड इस्तेमाल किया जाता है जब दो लाइब्रेरीज़ एक-दूसरे पर निर्भर करती हैं (इसे _ऊपरी निर्भरता_ कहा जाता है)।

हालांकि, दो तरह की dylib हाइजैकिंग होती है:

* **हाइजैकिंग लाइब्रेरीज़ की कमी**: इसका अर्थ है कि एप्लिकेशन को एक ऐसी लाइब्रेरी लोड करने की कोशिश करेगा जो कि **LC\_LOAD\_WEAK\_DYLIB** के साथ कॉन्फ़िगर नहीं है। फिर, **यदि हमलावर उस जगह पर एक dylib रखता है जहां लोड होने की उम्मीद है** तो वह लोड हो जाएगी।
* यह तथ्य कि लिंक "कमजोर" है इसका अर्थ है कि एप्लिकेशन चलती रहेगी भले ही लाइब्रेरी नहीं मिलती हो।
* इसके संबंधित **कोड** में `ImageLoaderMachO.cpp` के `ImageLoaderMachO::doGetDependentLibraries` फ़ंक्शन में है जहां `lib->required` केवल तब `false` होता है जब `LC_LOAD_WEAK_DYLIB` सच होता है।
* **बाइनरीज़ में कमजोर लिंक लाइब्रेरीज़ ढूंढें** (आपके पास बाद में हाइजैकिंग लाइब्रेरीज़ बनाने का एक उदाहरण है):
* ```bash
otool -l </path/to/bin> | grep LC_LOAD_WEAK_DYLIB -A 5 cmd LC_LOAD_WEAK_DYLIB
cmdsize 56
name /var/tmp/lib/libUtl.1.dylib (offset 24)
time stamp 2 Wed Jun 21 12:23:31 1969
current version 1.0.0
compatibility version 1.0.0
```
* **@rpath के साथ कॉन्फ़िगर किया गया**: Mach-O बाइनरीज़ में कमांड **`LC_RPATH`** और **`LC_LOAD_DYLIB`** हो सकते हैं। इन कमांडों के **मानों** के आधार पर, लाइब्रेरीज़ **अलग-अलग निर्देशिकाओं** से **लोड** होंगी।
* **`LC_RPATH`** बाइनरी द्वारा लाइब्रेरीज़ लोड करने के लिए उपयोग किए जाने वाले कुछ फ़ोल्डरों के पथ को समेटता है।
* **`LC_LOAD_DYLIB`** विशेष लाइब्रेरीज़ को लोड करने क
* जब पथ **फ्रेमवर्क पथ की तरह दिखता है** (उदा। `/stuff/foo.framework/foo`), अगर **`$DYLD_FRAMEWORK_PATH`** लॉन्च पर सेट किया गया था, तो dyld पहले उस निर्दिष्ट निर्देशिका में फ्रेमवर्क आंशिक पथ (उदा। `foo.framework/foo`) के लिए खोजेगा। अगले, dyld सप्लाइड किया गया पथ को असीमित प्रक्रिया के लिए उपयोग करेगा (संबंधित पथों के लिए मौजूदा काम करने वाली निर्देशिका का उपयोग करते हुए)। अंत में, पुराने बाइनरी के लिए, dyld कुछ फॉलबैक कोशिशें करेगा। अगर लॉन्च पर **`$DYLD_FALLBACK_FRAMEWORK_PATH`** सेट किया गया था, तो dyld उन निर्देशिकाओं में खोजेगा। अन्यथा, यह **`/Library/Frameworks`** (macOS पर अगर प्रक्रिया असीमित है) और फिर **`/System/Library/Frameworks`** में खोजेगा।
1. `$DYLD_FRAMEWORK_PATH`
2. सप्लाइड किया गया पथ (अगर प्रक्रिया असीमित है तो मौजूदा काम करने वाली निर्देशिका का उपयोग करते हुए)
3. `$DYLD_FALLBACK_FRAMEWORK_PATH`
4. `/Library/Frameworks` (अगर प्रक्रिया असीमित है)
5. `/System/Library/Frameworks`

{% hint style="danger" %}
अगर फ्रेमवर्क पथ है, तो इसे हाइजैक करने का तरीका होगा:

* अगर प्रक्रिया **असीमित है**, तो मौजूदा काम करने वाली निर्देशिका से **CWD के अनुपालन का दुरुपयोग** करें (यहां तक कि यदि दस्तावेज़ों में यह कहा नहीं गया है कि प्रक्रिया प्रतिबंधित है तो DYLD\_\* env vars हटा दिए जाते हैं)
{% endhint %}

* जब पथ में **एक स्लैश होता है लेकिन फ्रेमवर्क पथ नहीं होता है** (अर्थात पूरा पथ या dylib के लिए आंशिक पथ), dlopen() पहले (यदि सेट है) **`$DYLD_LIBRARY_PATH`** में खोजेगा (पथ के पते के लिए पत्ता भाग के साथ)। अगले, dyld **सप्लाइड किया गया पथ की कोशिश करेगा** (केवल असीमित प्रक्रियाओं के लिए मौजूदा काम करने वाली निर्देशिका का उपयोग करते हुए)। अंत में, पुराने बाइनरी के लिए, dyld कुछ फॉलबैक कोशिशें करेगा। अगर लॉन्च पर **`$DYLD_FALLBACK_LIBRARY_PATH`** सेट किया गया था, तो dyld उन निर्देशिकाओं में खोजेगा, अन्यथा, dyld **`/usr/local/lib/`** (अगर प्रक्रिया असीमित है) में खोजेगा, और फिर **`/usr/lib/`** में खोजेगा।
1. `$DYLD_LIBRARY_PATH`
2. सप्लाइड किया गया पथ (अगर प्रक्रिया असीमित है तो मौजूदा काम करने वाली निर्देशिका का उपयोग करते हुए)
3. `$DYLD_FALLBACK_LIBRARY_PATH`
4. `/usr/local/lib/` (अगर प्रक्रिया असीमित है)
5. `/usr/lib/`

{% hint style="danger" %}
अगर नाम में स्लैश है और फ्रेमवर्क नहीं है, तो इसे हाइजैक करने का तरीका होगा:

* अगर बाइनरी **असीमित है** और फिर CWD या `/usr/local/lib` से कुछ लोड करना संभव होगा (या उल्लिखित env variables में से किसी का दुरुपयोग करना)
{% endhint %}

{% hint style="info" %}
नोट: **dlopen खोजने को नियंत्रित करने वाले** कोई भी कॉन्फ़िगरेशन फ़ाइलें नहीं हैं।

नोट: अगर मुख्य executable एक **set\[ug]id बाइनरी है या entitlements के साथ codesigned** है, तो **सभी environment variables को अनदेखा किया जाता है**, और केवल पूरा पथ ही उपयोग किया जा सकता है (अधिक विस्तृत जानकारी के लिए [DYLD\_INSERT\_LIBRARIES restrictions की जांच करें](../../macos-dyld-hijacking-and-dyld\_insert\_libraries.md#check-dyld\_insert\_librery-restrictions))

नोट: Apple platforms में 32-बिट और 64-बिट लाइब्रेरीज़ को एकत्रित करने के लिए "universal" फ़ाइलें का उपयोग किया जाता है। इसका मतलब है कि **अलग 32-बिट और 64-बिट खोजने के पथ नहीं होते हैं**।

नोट: Apple platforms पर अधिकांश OS dylibs को dyld कैश में **एकीकृत किया जाता है** और डिस्क पर मौजूद नहीं होते हैं। इसलिए, किसी OS dylib की मौजूदगी की पूर्व-जांच के लिए **`stat()`** को कॉल करना **काम नहीं करेगा**। हालांकि, **`dlopen_preflight()`** एक संगत mach-o फ़ाइल खोजने के लिए **`dlopen()`** के उपयोग करता है।
{% endhint %}

**पथों की जांच करें**

निम्नलिखित कोड के साथ सभी विकल्पों की जांच करें:
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
यदि आप इसे कंपाइल और निष्पादित करते हैं तो आप देख सकते हैं **हर पुस्तकालय की वहाँ असफलतापूर्वक खोजी गई थी**। इसके अलावा, आप **एफएस लॉग को फ़िल्टर कर सकते हैं**:
```bash
sudo fs_usage | grep "dlopentest"
```
## डायल्ड `DYLD_*` और `LD_LIBRARY_PATH` एनवायरनमेंट वेरिएबल को काटें

फ़ाइल `dyld-dyld-832.7.1/src/dyld2.cpp` में हमें फ़ंक्शन **`pruneEnvironmentVariables`** मिल सकता है, जो किसी भी एनवायरनमेंट वेरिएबल को हटा देगा जो **`DYLD_`** और **`LD_LIBRARY_PATH=`** से शुरू होता है।

यह विशेष रूप से **suid** और **sgid** बाइनरी के लिए एनवायरनमेंट वेरिएबल **`DYLD_FALLBACK_FRAMEWORK_PATH`** और **`DYLD_FALLBACK_LIBRARY_PATH`** को **null** पर सेट करेगा।

यदि OSX को लक्ष्य बनाया जाता है, तो यह फ़ंक्शन उसी फ़ाइल के **`_main`** फ़ंक्शन से कॉल किया जाता है:
```cpp
#if TARGET_OS_OSX
if ( !gLinkContext.allowEnvVarsPrint && !gLinkContext.allowEnvVarsPath && !gLinkContext.allowEnvVarsSharedCache ) {
pruneEnvironmentVariables(envp, &apple);
```
और वे बूलियन फ्लैग्स कोड में ही उसी फ़ाइल में सेट किए जाते हैं:
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
जो इसका मतलब है कि यदि बाइनरी **suid** या **sgid** है, या हेडर में **RESTRICT** सेगमेंट है या यह **CS\_RESTRICT** फ्लैग के साथ साइन किया गया है, तो **`!gLinkContext.allowEnvVarsPrint && !gLinkContext.allowEnvVarsPath && !gLinkContext.allowEnvVarsSharedCache`** सत्य होता है और env variables को काट दिया जाता है।

ध्यान दें कि यदि CS\_REQUIRE\_LV सत्य है, तो चर env variables को काटा नहीं जाएगा लेकिन पुस्तकालय सत्यापन यह जांचेगा कि वे मूल बाइनरी के साथ समान प्रमाणपत्र का उपयोग कर रहे हैं।

## प्रतिबंधों की जांच करें

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
### धारा `__RESTRICT` और सेगमेंट `__restrict`
```bash
gcc -sectcreate __RESTRICT __restrict /dev/null hello.c -o hello-restrict
DYLD_INSERT_LIBRARIES=inject.dylib ./hello-restrict
```
### हार्डन्ड रनटाइम

कीचेन में एक नया प्रमाणपत्र बनाएं और इसे बाइनरी को साइन करने के लिए उपयोग करें:

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
ध्यान दें कि यदि किसी बाइनरी के साथ झंडे **`0x0(none)`** के साथ हस्ताक्षर हैं, तो जब उन्हें निष्पादित किया जाता है, तो वे गतिशील रूप से **`CS_RESTRICT`** झंडा प्राप्त कर सकते हैं और इसलिए इस तकनीक का उपयोग उनमें नहीं किया जा सकता है।

आप यह जांच सकते हैं कि क्या किसी प्रोसेस में इस झंडे है या नहीं (यहां [**csops**](https://github.com/axelexic/CSOps) प्राप्त करें):&#x20;
```bash
csops -status <pid>
```
और फिर यह जांचें कि क्या फ्लैग 0x800 सक्षम है।
{% endhint %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने की आवश्यकता** है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एकल [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स को हमें PR के माध्यम से साझा करें** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को सबमिट करके।**

</details>
