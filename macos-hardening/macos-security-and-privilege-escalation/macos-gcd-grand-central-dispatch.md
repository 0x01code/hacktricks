# macOS GCD - ग्रैंड सेंट्रल डिस्पैच

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **हैकट्रिक्स में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एकल [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल हों** या मुझे **ट्विटर** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का** **अनुसरण** करें।**
* **अपने हैकिंग ट्रिक्स को** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में पीआर जमा करके अपना योगदान दें।**

</details>

## मूलभूत जानकारी

**ग्रैंड सेंट्रल डिस्पैच (GCD)**, जिसे **लिबडिस्पैच** भी कहा जाता है, macOS और iOS दोनों में उपलब्ध है। यह एक ऐप्लिकेशन को समय-संचालित (मल्टीथ्रेडेड) निष्पादन के लिए अनुकूलन करने के लिए Apple द्वारा विकसित एक तकनीक है।

**GCD** आपके ऐप्लिकेशन को **ब्लॉक ऑब्जेक्ट के रूप में कार्य** सबमिट करने के लिए **FIFO कतारें** प्रदान करता है और उन्हें पूरी तरह से सिस्टम द्वारा प्रबंधित एक थ्रेड पूल पर निष्पादित करता है। GCD स्वचालित रूप से थ्रेड बनाता है जो डिस्पैच कतारों में कार्यों को निष्पादित करने के लिए निर्मित किए जाते हैं और उपलब्ध कोर्स पर चलाने के लिए उन कार्यों को अनुसूचित करता है।

{% hint style="success" %}
सारांश में, **पारलेल** में कोड को निष्पादित करने के लिए, प्रक्रियाएं **GCD को कोड के ब्लॉक भेज सकती हैं**, जो उनके निष्पादन का ध्यान रखेगा। इसलिए, प्रक्रियाएं नए थ्रेड नहीं बनाती हैं; **GCD अपने थ्रेड पूल के साथ दिए गए कोड को निष्पादित करता है**।
{% endhint %}

यह पारलेल निष्पादन को सफलतापूर्वक प्रबंधित करने के लिए बहुत मददगार है, जो प्रक्रियाओं द्वारा बनाए जाने वाले थ्रेडों की संख्या को कम करता है और पारलेल निष्पादन को अनुकूलित करता है। यह महान पारलेलिज्म की आवश्यकता वाले कार्यों (ब्रूट-फोर्सिंग?) के लिए और उन कार्यों के लिए उपयुक्त है जो मुख्य थ्रेड को ब्लॉक नहीं करना चाहिए: उदाहरण के लिए, iOS पर मुख्य थ्रेड UI इंटरैक्शन को संभालता है, इसलिए ऐप हैंग करा सकती है (खोज, वेब तक पहुंच, फ़ाइल पढ़ना...) इस तरीके से प्रबंधित किया जाता है।

## Objective-C

Objective-C में पारलेल में निष्पादित करने के लिए विभिन्न फ़ंक्शन हैं:

* [**dispatch\_async**](https://developer.apple.com/documentation/dispatch/1453057-dispatch\_async): एक ब्लॉक को एक डिस्पैच कतार पर असिंक्रोनस निष्पादन के लिए सबमिट करता है और तुरंत वापसी करता है।
* [**dispatch\_sync**](https://developer.apple.com/documentation/dispatch/1452870-dispatch\_sync): एक ब्लॉक ऑब्जेक्ट को निष्पादित करने के लिए सबमिट करता है और उस ब्लॉक के निष्पादन के बाद वापसी करता है।
* [**dispatch\_once**](https://developer.apple.com/documentation/dispatch/1447169-dispatch\_once): एक ब्लॉक ऑब्जेक्ट को केवल एक बार एक ऐप्लिकेशन के जीवनकाल के लिए निष्पादित करता है।
* [**dispatch\_async\_and\_wait**](https://developer.apple.com/documentation/dispatch/3191901-dispatch\_async\_and\_wait): एक कार्य आइटम को निष्पादित करने के लिए सबमिट करता है और इसके निष्पादन के बाद ही वापसी करता है। [**`dispatch_sync`**](https://developer.apple.com/documentation/dispatch/1452870-dispatch\_sync) के विपरीत, यह फ़ंक्शन ब्लॉक को निष्पादित करते समय कतार की सभी विशेषताओं का पालन करता है।

इन फ़ंक्शनों की इन पैरामीटर्स की उम्मीद होती हैं: [**`dispatch_queue_t`**](https://developer.apple
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
और यह एक उदाहरण है **पैरलेलिज़्म** का उपयोग करने के लिए **`dispatch_async`** के साथ:
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

**`libswiftDispatch`** एक पुस्तकालय है जो ग्रैंड सेंट्रल डिस्पैच (GCD) फ्रेमवर्क के लिए **स्विफ्ट बाइंडिंग** प्रदान करती है जो मूल रूप से सी में लिखी गई है।\
**`libswiftDispatch`** पुस्तकालय C GCD APIs को एक और स्विफ्ट-मित्री इंटरफ़ेस में लपेटती है, जिससे स्विफ्ट डेवलपर्स को GCD के साथ काम करना आसान और सहज होता है।

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

निम्नलिखित फ्रिडा स्क्रिप्ट का उपयोग करके कई `dispatch` फ़ंक्शन में हुक करने और कतार का नाम, बैकट्रेस और ब्लॉक निकालने के लिए किया जा सकता है: [**https://github.com/seemoo-lab/frida-scripts/blob/main/scripts/libdispatch.js**](https://github.com/seemoo-lab/frida-scripts/blob/main/scripts/libdispatch.js)
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
## गिड्रा

वर्तमान में गिड्रा को ना ही ObjectiveC का **`dispatch_block_t`** संरचना समझता है, ना ही **`swift_dispatch_block`** को।

तो यदि आप चाहते हैं कि इसे समझें, तो आप बस उन्हें **घोषित कर सकते हैं**:

<figure><img src="../../.gitbook/assets/image (688).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (690).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (691).png" alt="" width="563"><figcaption></figcaption></figure>

फिर, कोड में एक ऐसी जगह ढूंढ़ें जहां वे **उपयोग होते हैं**:

{% hint style="success" %}
संरचना का उपयोग हो रहा है उसे समझने के लिए "ब्लॉक" के सभी संदर्भों का ध्यान दें।
{% endhint %}

<figure><img src="../../.gitbook/assets/image (692).png" alt="" width="563"><figcaption></figcaption></figure>

चयनित चर क्लिक करें -> चर का पुनर्निर्धारण करें और इस मामले में **`swift_dispatch_block`** का चयन करें:

<figure><img src="../../.gitbook/assets/image (693).png" alt="" width="563"><figcaption></figcaption></figure>

गिड्रा स्वचालित रूप से सब कुछ दोबारा लिख देगा:

<figure><img src="../../.gitbook/assets/image (694).png" alt="" width="563"><figcaption></figcaption></figure>

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करना है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह
* प्राप्त करें [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें और PR जमा करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **में जमा करें।**

</details>
