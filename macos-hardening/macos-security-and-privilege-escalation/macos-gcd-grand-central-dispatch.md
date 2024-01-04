# macOS GCD - ग्रैंड सेंट्रल डिस्पैच

<details>

<summary><strong>शून्य से लेकर हीरो तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) को **फॉलो करें**.
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>

## मूल जानकारी

**ग्रैंड सेंट्रल डिस्पैच (GCD),** जिसे **libdispatch** भी कहा जाता है, macOS और iOS दोनों में उपलब्ध है। यह Apple द्वारा विकसित एक प्रौद्योगिकी है जो मल्टीकोर हार्डवेयर पर समानांतर (मल्टीथ्रेडेड) निष्पादन के लिए एप्लिकेशन समर्थन को अनुकूलित करती है।

**GCD** आपके एप्लिकेशन को **FIFO कतारें** प्रदान करता है और प्रबंधित करता है जिसमें आप **ब्लॉक ऑब्जेक्ट्स** के रूप में **कार्य सबमिट कर सकते हैं**। डिस्पैच कतारों में सबमिट किए गए ब्लॉक्स को पूरी तरह से सिस्टम द्वारा प्रबंधित थ्रेड्स के पूल पर **निष्पादित किया जाता है**। GCD स्वचालित रूप से डिस्पैच कतारों में कार्यों को निष्पादित करने के लिए थ्रेड्स बनाता है और उपलब्ध कोरों पर उन कार्यों को चलाने के लिए अनुसूचित करता है।

{% hint style="success" %}
संक्षेप में, कोड को **समानांतर रूप से निष्पादित** करने के लिए, प्रक्रियाएं **कोड के ब्लॉक्स को GCD को भेज सकती हैं**, जो उनके निष्पादन का ध्यान रखेगा। इसलिए, प्रक्रियाएं नए थ्रेड्स नहीं बनाती हैं; **GCD अपने थ्रेड्स के पूल के साथ दिए गए कोड को निष्पादित करता है**।
{% endhint %}

यह समानांतर निष्पादन को सफलतापूर्वक प्रबंधित करने के लिए बहुत सहायक है, प्रक्रियाओं द्वारा बनाए गए थ्रेड्स की संख्या को काफी कम करता है और समानांतर निष्पादन को अनुकूलित करता है। यह उन कार्यों के लिए आदर्श है जिन्हें **महान समानांतरता** की आवश्यकता होती है (ब्रूट-फोर्सिंग?) या उन कार्यों के लिए जो मुख्य थ्रेड को ब्लॉक नहीं करना चाहिए: उदाहरण के लिए, iOS पर मुख्य थ्रेड UI इंटरैक्शन्स को संभालता है, इसलिए कोई अन्य कार्यक्षमता जो ऐप को हैंग कर सकती है (खोजना, वेब तक पहुंचना, फाइल पढ़ना...) इस तरह से प्रबंधित की जाती है।

## Objective-C

Objective-C में समानांतर रूप से निष्पादित करने के लिए एक ब्लॉक भेजने के लिए विभिन्न फंक्शन हैं:

* [**dispatch\_async**](https://developer.apple.com/documentation/dispatch/1453057-dispatch\_async): एक डिस्पैच कतार पर एक ब्लॉक को असिंक्रोनस निष्पादन के लिए सबमिट करता है और तुरंत लौटता है।
* [**dispatch\_sync**](https://developer.apple.com/documentation/dispatch/1452870-dispatch\_sync): एक ब्लॉक ऑब्जेक्ट को निष्पादन के लिए सबमिट करता है और उस ब्लॉक के निष्पादित होने के बाद लौटता है।
* [**dispatch\_once**](https://developer.apple.com/documentation/dispatch/1447169-dispatch\_once): एक एप्लिकेशन के जीवनकाल के लिए केवल एक बार एक ब्लॉक ऑब्जेक्ट को निष्पादित करता है।
* [**dispatch\_async\_and\_wait**](https://developer.apple.com/documentation/dispatch/3191901-dispatch\_async\_and\_wait): एक कार्य आइटम को निष्पादन के लिए सबमिट करता है और केवल उसके निष्पादित होने के बाद लौटता है। [**`dispatch_sync`**](https://developer.apple.com/documentation/dispatch/1452870-dispatch\_sync) के विपरीत, यह फंक्शन ब्लॉक को निष्पादित करते समय कतार के सभी गुणों का सम्मान करता है।

इन फंक्शन्स को ये पैरामीटर्स की अपेक्षा होती है: [**`dispatch_queue_t`**](https://developer.apple.com/documentation/dispatch/dispatch\_queue\_t) **`queue,`** [**`dispatch_block_t`**](https://developer.apple.com/documentation/dispatch/dispatch\_block\_t) **`block`**

यह एक **ब्लॉक का स्ट्रक्चर** है:
```c
struct Block {
void *isa; // NSConcreteStackBlock,...
int flags;
int reserved;
void *invoke;
struct BlockDescriptor *descriptor;
// captured variables go here
};
```
और यह **समानांतरता** का प्रयोग **`dispatch_async`** के साथ करने का एक उदाहरण है:
```objectivec
#import <Foundation/Foundation.h>

// Define a block
void (^backgroundTask)(void) = ^{
// Code to be executed in the background
for (int i = 0; i < 10; i++) {
NSLog(@"Background task %d", i);
sleep(1);  // Simulate a long-running task
}
};

int main(int argc, const char * argv[]) {
@autoreleasepool {
// Create a dispatch queue
dispatch_queue_t backgroundQueue = dispatch_queue_create("com.example.backgroundQueue", NULL);

// Submit the block to the queue for asynchronous execution
dispatch_async(backgroundQueue, backgroundTask);

// Continue with other work on the main queue or thread
for (int i = 0; i < 10; i++) {
NSLog(@"Main task %d", i);
sleep(1);  // Simulate a long-running task
}
}
return 0;
}
```
## Swift

**`libswiftDispatch`** एक पुस्तकालय है जो **Swift bindings** प्रदान करता है Grand Central Dispatch (GCD) फ्रेमवर्क के लिए जो मूल रूप से C में लिखा गया है।\
**`libswiftDispatch`** पुस्तकालय C GCD APIs को एक अधिक Swift-अनुकूल इंटरफेस में लपेटता है, जिससे Swift डेवलपर्स के लिए GCD के साथ काम करना आसान और अधिक सहज हो जाता है।

* **`DispatchQueue.global().sync{ ... }`**
* **`DispatchQueue.global().async{ ... }`**
* **`let onceToken = DispatchOnce(); onceToken.perform { ... }`**
* **`async await`**
* **`var (data, response) = await URLSession.shared.data(from: URL(string: "https://api.example.com/getData"))`**

**कोड उदाहरण**:
```swift
import Foundation

// Define a closure (the Swift equivalent of a block)
let backgroundTask: () -> Void = {
for i in 0..<10 {
print("Background task \(i)")
sleep(1)  // Simulate a long-running task
}
}

// Entry point
autoreleasepool {
// Create a dispatch queue
let backgroundQueue = DispatchQueue(label: "com.example.backgroundQueue")

// Submit the closure to the queue for asynchronous execution
backgroundQueue.async(execute: backgroundTask)

// Continue with other work on the main queue
for i in 0..<10 {
print("Main task \(i)")
sleep(1)  // Simulate a long-running task
}
}
```
## Frida

निम्नलिखित Frida स्क्रिप्ट का उपयोग **कई `dispatch`** फंक्शन्स में हुक करने और कतार का नाम, बैकट्रेस और ब्लॉक निकालने के लिए किया जा सकता है: [**https://github.com/seemoo-lab/frida-scripts/blob/main/scripts/libdispatch.js**](https://github.com/seemoo-lab/frida-scripts/blob/main/scripts/libdispatch.js)
```bash
frida -U <prog_name> -l libdispatch.js

dispatch_sync
Calling queue: com.apple.UIKit._UIReusePool.reuseSetAccess
Callback function: 0x19e3a6488 UIKitCore!__26-[_UIReusePool addObject:]_block_invoke
Backtrace:
0x19e3a6460 UIKitCore!-[_UIReusePool addObject:]
0x19e3a5db8 UIKitCore!-[UIGraphicsRenderer _enqueueContextForReuse:]
0x19e3a57fc UIKitCore!+[UIGraphicsRenderer _destroyCGContext:withRenderer:]
[...]
```
## Ghidra

वर्तमान में Ghidra न तो ObjectiveC की **`dispatch_block_t`** संरचना को समझता है और न ही **`swift_dispatch_block`** को।

यदि आप चाहते हैं कि यह उन्हें समझे, तो आप उन्हें **घोषित कर सकते हैं**:

<figure><img src="../../.gitbook/assets/image (688).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (690).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (691).png" alt="" width="563"><figcaption></figcaption></figure>

फिर, कोड में एक जगह ढूंढें जहां उनका **उपयोग किया गया हो**:

{% hint style="success" %}
"block" के सभी संदर्भों को नोट करें ताकि आप समझ सकें कि संरचना का उपयोग कैसे किया जा रहा है।
{% endhint %}

<figure><img src="../../.gitbook/assets/image (692).png" alt="" width="563"><figcaption></figcaption></figure>

वेरिएबल पर राइट क्लिक करें -> Retype Variable और इस मामले में **`swift_dispatch_block`** चुनें:

<figure><img src="../../.gitbook/assets/image (693).png" alt="" width="563"><figcaption></figcaption></figure>

Ghidra स्वचालित रूप से सब कुछ फिर से लिख देगा:

<figure><img src="../../.gitbook/assets/image (694).png" alt="" width="563"><figcaption></figcaption></figure>

<details>

<summary><strong>शून्य से हीरो तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें।
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह।
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) को **फॉलो** करें।
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>
