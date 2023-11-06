# macOS IOKit

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks** में विज्ञापित देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड** करने का एक्सेस चाहिए? [**सदस्यता प्लान**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* प्राप्त करें [**PEASS और HackTricks की आधिकारिक स्वैग**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) **Discord समूह** या [**टेलीग्राम समूह**](https://t.me/peass) में या **मुझे ट्विटर पर फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live).
* **अपने हैकिंग ट्रिक्स साझा करें** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को PR भेजकर**।

</details>

## मूलभूत जानकारी

I/O Kit एक ओपन-सोर्स, ऑब्जेक्ट-ओरिएंटेड, **डिवाइस-ड्राइवर फ्रेमवर्क** है जो XNU कर्नल में होता है और **डाइनामिक लोड होने वाले डिवाइस ड्राइवर्स** के जोड़ने और प्रबंधन के लिए जिम्मेदार होता है। ये ड्राइवर्स कर्नल में विभिन्न हार्डवेयर के साथ उपयोग के लिए डाइनामिक रूप से मॉड्यूलर कोड जोड़ने की अनुमति देते हैं।

IOKit ड्राइवर्स मूल रूप से कर्नल से **फ़ंक्शन निर्यात करेंगे**। इन फ़ंक्शन पैरामीटर **प्रकार** पहले से **परिभाषित** होते हैं और सत्यापित होते हैं। इसके अलावा, XPC की तरह, IOKit केवल एक और परत है **Mach संदेशों के ऊपर**।

**IOKit XNU कर्नल कोड** Apple द्वारा [https://github.com/apple-oss-distributions/xnu/tree/main/iokit](https://github.com/apple-oss-distributions/xnu/tree/main/iokit) में ओपन-सोर्स किया गया है। इसके अलावा, यूजर स्पेस IOKit के घटक भी ओपन-सोर्स हैं [https://github.com/opensource-apple/IOKitUser](https://github.com/opensource-apple/IOKitUser)।

हालांकि, **कोई भी IOKit ड्राइवर** ओपन-सोर्स नहीं हैं। फिर भी, समय-समय पर एक ड्राइवर का रिलीज़ सिंबल्स के साथ आ सकता है जो इसे डिबग करना आसान बना देता है। यहां देखें कि [**फर्मवेयर से ड्राइवर एक्सटेंशन कैसे प्राप्त करें**](./#ipsw)**।**

यह **C++** में लिखा गया है। आप डीमैंगल्ड C++ सिंबल्स प्राप्त कर सकते हैं:
```bash
# Get demangled symbols
nm -C com.apple.driver.AppleJPEGDriver

# Demangled symbols from stdin
c++filt
__ZN16IOUserClient202222dispatchExternalMethodEjP31IOExternalMethodArgumentsOpaquePK28IOExternalMethodDispatch2022mP8OSObjectPv
IOUserClient2022::dispatchExternalMethod(unsigned int, IOExternalMethodArgumentsOpaque*, IOExternalMethodDispatch2022 const*, unsigned long, OSObject*, void*)
```
{% hint style="danger" %}
IOKit **उद्घाटन फ़ंक्शन** एक अतिरिक्त सुरक्षा जांच कर सकते हैं जब कोई क्लाइंट एक फ़ंक्शन को कॉल करने की कोशिश करता है, लेकिन ध्यान दें कि ऐप्स आमतौर पर उन IOKit फ़ंक्शनों से प्रतिक्रिया कर सकते हैं जिनके लिए उन्हें सैंडबॉक्स द्वारा सीमित किया जाता है।
{% endhint %}

## ड्राइवर

macOS में वे स्थित होते हैं:

* **`/System/Library/Extensions`**
* ओएस एक्स ऑपरेटिंग सिस्टम में बिल्ट इन केक्स फ़ाइलें।
* **`/Library/Extensions`**
* तृतीय पक्ष सॉफ़्टवेयर द्वारा स्थापित केक्स फ़ाइलें

iOS में वे स्थित होते हैं:

* **`/System/Library/Extensions`**
```bash
#Use kextstat to print the loaded drivers
kextstat
Executing: /usr/bin/kmutil showloaded
No variant specified, falling back to release
Index Refs Address            Size       Wired      Name (Version) UUID <Linked Against>
1  142 0                  0          0          com.apple.kpi.bsd (20.5.0) 52A1E876-863E-38E3-AC80-09BBAB13B752 <>
2   11 0                  0          0          com.apple.kpi.dsep (20.5.0) 52A1E876-863E-38E3-AC80-09BBAB13B752 <>
3  170 0                  0          0          com.apple.kpi.iokit (20.5.0) 52A1E876-863E-38E3-AC80-09BBAB13B752 <>
4    0 0                  0          0          com.apple.kpi.kasan (20.5.0) 52A1E876-863E-38E3-AC80-09BBAB13B752 <>
5  175 0                  0          0          com.apple.kpi.libkern (20.5.0) 52A1E876-863E-38E3-AC80-09BBAB13B752 <>
6  154 0                  0          0          com.apple.kpi.mach (20.5.0) 52A1E876-863E-38E3-AC80-09BBAB13B752 <>
7   88 0                  0          0          com.apple.kpi.private (20.5.0) 52A1E876-863E-38E3-AC80-09BBAB13B752 <>
8  106 0                  0          0          com.apple.kpi.unsupported (20.5.0) 52A1E876-863E-38E3-AC80-09BBAB13B752 <>
9    2 0xffffff8003317000 0xe000     0xe000     com.apple.kec.Libm (1) 6C1342CC-1D74-3D0F-BC43-97D5AD38200A <5>
10   12 0xffffff8003544000 0x92000    0x92000    com.apple.kec.corecrypto (11.1) F5F1255F-6552-3CF4-A9DB-D60EFDEB4A9A <8 7 6 5 3 1>
```
गणितीय संख्या 9 तक सूचीबद्ध ड्राइवर **पते 0 में लोड होते हैं**। इसका अर्थ है कि वे वास्तविक ड्राइवर नहीं हैं, बल्कि **कर्नल का हिस्सा हैं और उन्हें अनलोड नहीं किया जा सकता है**।

विशेष एक्सटेंशन ढूंढ़ने के लिए आप निम्नलिखित का उपयोग कर सकते हैं:
```bash
kextfind -bundle-id com.apple.iokit.IOReportFamily #Search by full bundle-id
kextfind -bundle-id -substring IOR #Search by substring in bundle-id
```
कर्नल एक्सटेंशन्स को लोड और अनलोड करने के लिए निम्नलिखित कार्रवाई करें:
```bash
kextload com.apple.iokit.IOReportFamily
kextunload com.apple.iokit.IOReportFamily
```
## IORegistry

**IORegistry** मैकओएस और आईओएस में IOKit फ्रेमवर्क का एक महत्वपूर्ण हिस्सा है जो सिस्टम के हार्डवेयर कॉन्फ़िगरेशन और स्थिति को प्रतिष्ठान के रूप में प्रदर्शित करने के लिए एक डेटाबेस की तरह कार्य करता है। यह एक **पदात्मक संग्रह है जो सिस्टम पर लोड होने वाले हार्डवेयर और ड्राइवर्स को प्रतिष्ठान करता है, और उनके आपसी संबंधों को प्रतिष्ठान करता है**।

आप कंसोल से इसे जांचने के लिए **`ioreg`** का उपयोग करके IORegistry प्राप्त कर सकते हैं (विशेष रूप से आईओएस के लिए उपयोगी)।
```bash
ioreg -l #List all
ioreg -w 0 #Not cut lines
ioreg -p <plane> #Check other plane
```
आप **Xcode Additional Tools** से **`IORegistryExplorer`** डाउनलोड कर सकते हैं, जिसे [**https://developer.apple.com/download/all/**](https://developer.apple.com/download/all/) से डाउनलोड किया जा सकता है और एक **ग्राफिकल** इंटरफ़ेस के माध्यम से **macOS IORegistry** की जांच कर सकते हैं।

<figure><img src="../../../.gitbook/assets/image (695).png" alt="" width="563"><figcaption></figcaption></figure>

IORegistryExplorer में, "planes" का उपयोग IORegistry में विभिन्न ऑब्जेक्ट के बीच संबंधों को संगठित और प्रदर्शित करने के लिए किया जाता है। प्रत्येक plane एक विशेष प्रकार के संबंध या सिस्टम के हार्डवेयर और ड्राइवर कॉन्फ़िगरेशन के एक विशेष दृश्य को प्रदर्शित करता है। यहां IORegistryExplorer में आपको मिलने वाले कुछ सामान्य planes हैं:

1. **IOService Plane**: यह सबसे सामान्य plane है, जो ड्राइवर और nubs (ड्राइवर के बीच संचार चैनल) को प्रतिष्ठानित करने वाले सेवा ऑब्जेक्ट्स को प्रदर्शित करता है। यह इन ऑब्जेक्ट्स के बीच प्रदाता-ग्राहक संबंधों को दिखाता है।
2. **IODeviceTree Plane**: यह plane सिस्टम में जुड़े उपकरणों के बीच भौतिक संबंधों को प्रदर्शित करता है जब वे सिस्टम से जुड़े होते हैं। यह आमतौर पर USB या PCI जैसे बसों के माध्यम से जुड़े उपकरणों की पदावली को दिखाने के लिए उपयोग किया जाता है।
3. **IOPower Plane**: यह ऑब्जेक्ट्स और उनके संबंधों को शक्ति प्रबंधन के माध्यम से प्रदर्शित करता है। यह दिखा सकता है कि कौन से ऑब्जेक्ट्स दूसरों की शक्ति स्थिति पर प्रभाव डाल रहे हैं, जो शक्ति संबंधित मुद्दों के डीबगिंग के लिए उपयोगी होता है।
4. **IOUSB Plane**: यह विशेष रूप से USB उपकरणों और उनके संबंधों पर ध्यान केंद्रित करता है, जो USB हब्स और जुड़े हुए उपकरणों की पदावली को दिखाता है।
5. **IOAudio Plane**: यह plane सिस्टम के भीतर ऑडियो उपकरणों और उनके संबंधों को प्रदर्शित करने के लिए है।
6. ...

## ड्राइवर संचार कोड उदाहरण

निम्नलिखित कोड "YourServiceNameHere" IOKit सेवा से कनेक्ट करता है और सेलेक्टर 0 के अंदर की फ़ंक्शन को कॉल करता है। इसके लिए:

* पहले यह **`IOServiceMatching`** और **`IOServiceGetMatchingServices`** को कॉल करता है ताकि सेवा प्राप्त कर सके।
* फिर यह **`IOServiceOpen`** को कॉल करके एक कनेक्शन स्थापित करता है।
* और अंत में यह **`IOConnectCallScalarMethod`** के साथ एक फ़ंक्शन को कॉल करता है जिसमें सेलेक्टर 0 को दिखाता है (सेलेक्टर वह संख्या है जिसे आप कॉल करना चाहते हैं वाली फ़ंक्शन को दिया गया है)।
```objectivec
#import <Foundation/Foundation.h>
#import <IOKit/IOKitLib.h>

int main(int argc, const char * argv[]) {
@autoreleasepool {
// Get a reference to the service using its name
CFMutableDictionaryRef matchingDict = IOServiceMatching("YourServiceNameHere");
if (matchingDict == NULL) {
NSLog(@"Failed to create matching dictionary");
return -1;
}

// Obtain an iterator over all matching services
io_iterator_t iter;
kern_return_t kr = IOServiceGetMatchingServices(kIOMasterPortDefault, matchingDict, &iter);
if (kr != KERN_SUCCESS) {
NSLog(@"Failed to get matching services");
return -1;
}

// Get a reference to the first service (assuming it exists)
io_service_t service = IOIteratorNext(iter);
if (!service) {
NSLog(@"No matching service found");
IOObjectRelease(iter);
return -1;
}

// Open a connection to the service
io_connect_t connect;
kr = IOServiceOpen(service, mach_task_self(), 0, &connect);
if (kr != KERN_SUCCESS) {
NSLog(@"Failed to open service");
IOObjectRelease(service);
IOObjectRelease(iter);
return -1;
}

// Call a method on the service
// Assume the method has a selector of 0, and takes no arguments
kr = IOConnectCallScalarMethod(connect, 0, NULL, 0, NULL, NULL);
if (kr != KERN_SUCCESS) {
NSLog(@"Failed to call method");
}

// Cleanup
IOServiceClose(connect);
IOObjectRelease(service);
IOObjectRelease(iter);
}
return 0;
}
```
इसके अलावा **`IOConnectCallScalarMethod`** जैसे IOKit फंक्शन को कॉल करने के लिए अन्य फंक्शन भी हैं जैसे **`IOConnectCallMethod`**, **`IOConnectCallStructMethod`**...

## ड्राइवर एंट्रीपॉइंट को रिवर्स करना

आप इन्हें उदाहरण के लिए [**फर्मवेयर इमेज (ipsw)**](./#ipsw) से प्राप्त कर सकते हैं। फिर, इसे अपने पसंदीदा डीकंपाइलर में लोड करें।

आप **`externalMethod`** फंक्शन का डीकंपाइलिंग शुरू कर सकते हैं क्योंकि यह ड्राइवर फंक्शन है जो कॉल प्राप्त करेगा और सही फंक्शन को कॉल करेगा:

<figure><img src="../../../.gitbook/assets/image (696).png" alt="" width="315"><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (697).png" alt=""><figcaption></figcaption></figure>

वह भयानक कॉल डीमैंगल्ड का अर्थ है:
```cpp
IOUserClient2022::dispatchExternalMethod(unsigned int, IOExternalMethodArgumentsOpaque*, IOExternalMethodDispatch2022 const*, unsigned long, OSObject*, void*)
```
{% endcode %}

पिछले परिभाषण में ध्यान दें कि **`self`** पैरामीटर छूट गया है, अच्छी परिभाषण यह होगी:

{% code overflow="wrap" %}
```cpp
IOUserClient2022::dispatchExternalMethod(self, unsigned int, IOExternalMethodArgumentsOpaque*, IOExternalMethodDispatch2022 const*, unsigned long, OSObject*, void*)
```
{% endcode %}

वास्तव में, आप वास्तविक परिभाषा [https://github.com/apple-oss-distributions/xnu/blob/1031c584a5e37aff177559b9f69dbd3c8c3fd30a/iokit/Kernel/IOUserClient.cpp#L6388](https://github.com/apple-oss-distributions/xnu/blob/1031c584a5e37aff177559b9f69dbd3c8c3fd30a/iokit/Kernel/IOUserClient.cpp#L6388) में पा सकते हैं:
```cpp
IOUserClient2022::dispatchExternalMethod(uint32_t selector, IOExternalMethodArgumentsOpaque *arguments,
const IOExternalMethodDispatch2022 dispatchArray[], size_t dispatchArrayCount,
OSObject * target, void * reference)
```
इस जानकारी के साथ आप Ctrl+Right -> `संपादित करें फ़ंक्शन हस्ताक्षर` को पुनः लिख सकते हैं और नोएड टाइप्स सेट कर सकते हैं:

<figure><img src="../../../.gitbook/assets/image (702).png" alt=""><figcaption></figcaption></figure>

नया डीकंपाइल किया गया कोड इस तरह दिखेगा:

<figure><img src="../../../.gitbook/assets/image (703).png" alt=""><figcaption></figcaption></figure>

अगले कदम के लिए हमें **`IOExternalMethodDispatch2022`** स्ट्रक्ट को परिभाषित करना चाहिए। यह [https://github.com/apple-oss-distributions/xnu/blob/1031c584a5e37aff177559b9f69dbd3c8c3fd30a/iokit/IOKit/IOUserClient.h#L168-L176](https://github.com/apple-oss-distributions/xnu/blob/1031c584a5e37aff177559b9f69dbd3c8c3fd30a/iokit/IOKit/IOUserClient.h#L168-L176) पर ओपनसोर्स है, आप इसे परिभाषित कर सकते हैं:

<figure><img src="../../../.gitbook/assets/image (698).png" alt=""><figcaption></figcaption></figure>

अब, `(IOExternalMethodDispatch2022 *)&sIOExternalMethodArray` के अनुसरण करते हुए आप बहुत सारे डेटा देख सकते हैं:

<figure><img src="../../../.gitbook/assets/image (704).png" alt="" width="563"><figcaption></figcaption></figure>

डेटा प्रकार को **`IOExternalMethodDispatch2022:`** में बदलें

<figure><img src="../../../.gitbook/assets/image (705).png" alt="" width="375"><figcaption></figcaption></figure>

बदलने के बाद:

<figure><img src="../../../.gitbook/assets/image (707).png" alt="" width="563"><figcaption></figcaption></figure>

और जैसा कि हम अब जानते हैं, वहां हमारे पास **7 तत्वों का एक सरणी** है (अंतिम डीकंपाइल किए गए कोड की जांच करें), 7 तत्वों की एक सरणी बनाने के लिए क्लिक करें:

<figure><img src="../../../.gitbook/assets/image (708).png" alt="" width="563"><figcaption></figcaption></figure>

सरणी बनाने के बाद आप सभी निर्यात की गई फ़ंक्शन देख सकते हैं:

<figure><img src="../../../.gitbook/assets/image (709).png" alt=""><figcaption></figcaption></figure>

{% hint style="success" %}
यदि आप याद करें, तो उपयोगकर्ता स्थान से एक्सपोर्ट की गई फ़ंक्शन को **कॉल** करने के लिए हमें फ़ंक्शन के नाम को नहीं बल्कि **सेलेक्टर नंबर** को कॉल करने की आवश्यकता नहीं होती है। यहां आप देख सकते हैं कि सेलेक्टर **0** फ़ंक्शन **`initializeDecoder`** है, सेलेक्टर **1** **`startDecoder`** है, सेलेक्टर **2** **`initializeEncoder`** है...
{% endhint %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम कर रहे हैं? क्या आप अपनी कंपनी को **HackTricks** में विज्ञापित करना चाहते हैं? या क्या आप **PEASS की नवीनतम संस्करण देखना चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**सदस्यता की योजनाओं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**PEASS और HackTricks की आधिकारिक स्वैग**](https://peass.creator-spring.com)
* **डिस्कॉर्ड समूह** या **टेलीग्राम समूह** में **शामिल हों** या मुझे **ट्विटर** पर **फ़ॉलो करें** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live).
* **अपने हैकिंग ट्रिक्स साझा करें** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को PR भेजकर**.

</details>
