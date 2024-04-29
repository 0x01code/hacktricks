# macOS GCD - ग्रैंड सेंट्रल डिस्पैच

<details>

<summary><strong>जीरो से हीरो तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

## मूल जानकारी

**ग्रैंड सेंट्रल डिस्पैच (GCD),** जिसे **लिबडिस्पैच** (`libdispatch.dyld`) के रूप में भी जाना जाता है, macOS और iOS दोनों में उपलब्ध है। यह एक प्रौद्योगिकी है जिसे Apple ने बनाया है ताकि एप्लिकेशन समर्थन को अनुक्रमिक (मल्टीथ्रेडेड) क्रियान्वयन के लिए अनुकूलित कर सके मल्टीकोर हार्डवेयर पर।

**GCD** आपके एप्लिकेशन को **ब्लॉक ऑब्जेक्ट** के रूप में **कार्यों को सबमिट** करने के लिए **FIFO कतारें** प्रदान करता है और प्रबंधित करता है। डिस्पैच कतारों में सबमिट किए गए ब्लॉक को सिस्टम द्वारा पूरी तरह से प्रबंधित धागों पर निष्पादित किया जाता है। GCD स्वचालित रूप से धागों को निष्पादित करने के लिए धागों में कार्यों को निष्पादित करने के लिए धागों के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्पादित करने के लिए धागों को निष्प
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
और यह **पैरललिज़्म** का उपयोग करने के लिए एक उदाहरण है **`dispatch_async`** के साथ:
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
## स्विफ्ट

**`libswiftDispatch`** एक लाइब्रेरी है जो ग्रांड सेंट्रल डिस्पैच (GCD) फ्रेमवर्क के लिए **स्विफ्ट बाइंडिंग** प्रदान करती है जो मूल रूप से C में लिखी गई है।\
**`libswiftDispatch`** लाइब्रेरी C GCD APIs को एक और स्विफ्ट-मित्र इंटरफेस में लपेटती है, जिससे स्विफ्ट डेवलपर्स को GCD के साथ काम करना आसान और सहज होता है।

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
## फ्रिडा

निम्नलिखित फ्रिडा स्क्रिप्ट का उपयोग कई `डिस्पैच` फ़ंक्शन में **हुक करने** और कतार का नाम, बैकट्रेस और ब्लॉक निकालने के लिए किया जा सकता है: [**https://github.com/seemoo-lab/frida-scripts/blob/main/scripts/libdispatch.js**](https://github.com/seemoo-lab/frida-scripts/blob/main/scripts/libdispatch.js)
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
## गिडरा

वर्तमान में गिडरा न तो ObjectiveC **`dispatch_block_t`** संरचना को समझता है, न ही **`swift_dispatch_block`** को।

तो यदि आप चाहते हैं कि यह उन्हें समझे, तो आप बस **उन्हें घोषित** कर सकते हैं:

<figure><img src="../../.gitbook/assets/image (1157).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1159).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1160).png" alt="" width="563"><figcaption></figcaption></figure>

फिर, कोड में एक स्थान ढूंढें जहां वे **उपयोग** किए जा रहे हैं:

{% hint style="success" %}
"ब्लॉक" के सभी संदर्भों का ध्यान रखें ताकि आप समझ सकें कि संरचना का उपयोग कैसे किया जा रहा है।
{% endhint %}

<figure><img src="../../.gitbook/assets/image (1161).png" alt="" width="563"><figcaption></figcaption></figure>

माउस दायां क्लिक करें -> परिवर्तन चर का पुनर्निर्धारण करें और इस मामले में **`swift_dispatch_block`** का चयन करें:

<figure><img src="../../.gitbook/assets/image (1162).png" alt="" width="563"><figcaption></figcaption></figure>

गिडरा स्वचालित रूप से सब कुछ लिख देगा:

<figure><img src="../../.gitbook/assets/image (1163).png" alt="" width="563"><figcaption></figcaption></figure>

## संदर्भ

* [**\*OS Internals, Volume I: User Mode. By Jonathan Levin**](https://www.amazon.com/MacOS-iOS-Internals-User-Mode/dp/099105556X)
