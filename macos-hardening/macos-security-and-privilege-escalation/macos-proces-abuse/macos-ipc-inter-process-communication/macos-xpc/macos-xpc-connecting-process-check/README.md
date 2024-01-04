# macOS XPC कनेक्टिंग प्रोसेस चेक

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert) के साथ शून्य से नायक तक AWS हैकिंग सीखें</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) **का अनुसरण करें**.
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें.

</details>

## XPC कनेक्टिंग प्रोसेस चेक

जब XPC सेवा से कनेक्शन स्थापित होता है, सर्वर यह जांचेगा कि कनेक्शन अनुमति है या नहीं। ये आमतौर पर किए जाने वाले चेक हैं:

1. जांचें कि कनेक्टिंग **प्रोसेस Apple-signed** प्रमाणपत्र के साथ हस्ताक्षरित है (केवल Apple द्वारा दिया गया).
* यदि यह **सत्यापित नहीं है**, तो हमलावर किसी भी अन्य चेक से मेल खाने के लिए **नकली प्रमाणपत्र** बना सकता है।
2. जांचें कि कनेक्टिंग प्रोसेस **संगठन के प्रमाणपत्र** के साथ हस्ताक्षरित है, (टीम ID सत्यापन).
* यदि यह **सत्यापित नहीं है**, तो **किसी भी डेवलपर प्रमाणपत्र** का उपयोग Apple से हस्ताक्षरित करने और सेवा से कनेक्ट करने के लिए किया जा सकता है।
3. जांचें कि कनेक्टिंग प्रोसेस में **उचित बंडल ID** है।
* यदि यह **सत्यापित नहीं है**, तो समान संगठन द्वारा हस्ताक्षरित कोई भी टूल XPC सेवा के साथ इंटरैक्ट करने के लिए उपयोग किया जा सकता है।
4. (4 या 5) जांचें कि कनेक्टिंग प्रोसेस में **उचित सॉफ्टवेयर संस्करण संख्या** है।
* यदि यह **सत्यापित नहीं है,** तो पुराने, असुरक्षित क्लाइंट्स, प्रोसेस इंजेक्शन के लिए संवेदनशील हो सकते हैं और अन्य चेक्स के होते हुए भी XPC सेवा से कनेक्ट कर सकते हैं।
5. (4 या 5) जांचें कि कनेक्टिंग प्रोसेस में हार्डन्ड रनटाइम है बिना खतरनाक एंटाइटलमेंट्स के (जैसे कि जो अनियंत्रित लाइब्रेरीज़ लोड करने या DYLD env vars का उपयोग करने की अनुमति देते हैं)
1. यदि यह **सत्यापित नहीं है,** तो क्लाइंट **कोड इंजेक्शन के लिए संवेदनशील हो सकता है**
6. जांचें कि कनेक्टिंग प्रोसेस में एक **एंटाइटलमेंट** है जो इसे सेवा से कनेक्ट करने की अनुमति देता है। यह Apple बाइनरीज़ के लिए लागू होता है।
7. **सत्यापन** को कनेक्टिंग **क्लाइंट के ऑडिट टोकन** पर **आधारित** होना चाहिए **इसके प्रोसेस ID (PID)** के बजाय, क्योंकि पूर्व **PID पुन: उपयोग हमलों** से बचाता है।
* डेवलपर्स **शायद ही कभी ऑडिट टोकन** API कॉल का उपयोग करते हैं क्योंकि यह **निजी** है, इसलिए Apple कभी भी **बदल** सकता है। इसके अलावा, Mac App Store ऐप्स में निजी API का उपयोग अनुमति नहीं है।
* यदि **`processIdentifier`** मेथड का उपयोग किया जाता है, तो यह संवेदनशील हो सकता है
* **`xpc_dictionary_get_audit_token`** का उपयोग **`xpc_connection_get_audit_token`** के बजाय किया जाना चाहिए, क्योंकि नवीनतम भी [कुछ स्थितियों में संवेदनशील हो सकता है](https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/).

### संचार हमले

PID पुन: उपयोग हमले के बारे में अधिक जानकारी के लिए देखें:

{% content-ref url="macos-pid-reuse.md" %}
[macos-pid-reuse.md](macos-pid-reuse.md)
{% endcontent-ref %}

**`xpc_connection_get_audit_token`** हमले के बारे में अधिक जानकारी के लिए देखें:

{% content-ref url="macos-xpc_connection_get_audit_token-attack.md" %}
[macos-xpc\_connection\_get\_audit\_token-attack.md](macos-xpc\_connection\_get\_audit\_token-attack.md)
{% endcontent-ref %}

### Trustcache - डाउनग्रेड हमलों की रोकथाम

Trustcache Apple Silicon मशीनों में पेश की गई एक रक्षात्मक विधि है जो Apple बाइनरीज़ के CDHSAH का एक डेटाबेस संग्रहीत करती है ताकि केवल अनुमति दी गई अपरिवर्तित बाइनरीज़ ही निष्पादित की जा सकें। जो डाउनग्रेड संस्करणों के निष्पादन को रोकता है।

### कोड उदाहरण

सर्वर इस **सत्यापन** को **`shouldAcceptNewConnection`** कहलाने वाले फंक्शन में लागू करेगा।

{% code overflow="wrap" %}
```objectivec
- (BOOL)listener:(NSXPCListener *)listener shouldAcceptNewConnection:(NSXPCConnection *)newConnection {
//Check connection
return YES;
}
```
{% endcode %}

ऑब्जेक्ट NSXPCConnection में एक **निजी** संपत्ति **`auditToken`** (वह जिसका उपयोग किया जाना चाहिए लेकिन बदल सकता है) और एक **सार्वजनिक** संपत्ति **`processIdentifier`** (वह जिसका उपयोग नहीं किया जाना चाहिए) होती है।

कनेक्टिंग प्रोसेस की पुष्टि कुछ इस प्रकार से की जा सकती है:

{% code overflow="wrap" %}
```objectivec
[...]
SecRequirementRef requirementRef = NULL;
NSString requirementString = @"anchor apple generic and identifier \"xyz.hacktricks.service\" and certificate leaf [subject.CN] = \"TEAMID\" and info [CFBundleShortVersionString] >= \"1.0\"";
/* Check:
- Signed by a cert signed by Apple
- Check the bundle ID
- Check the TEAMID of the signing cert
- Check the version used
*/

// Check the requirements with the PID (vulnerable)
SecRequirementCreateWithString(requirementString, kSecCSDefaultFlags, &requirementRef);
SecCodeCheckValidity(code, kSecCSDefaultFlags, requirementRef);

// Check the requirements wuing the auditToken (secure)
SecTaskRef taskRef = SecTaskCreateWithAuditToken(NULL, ((ExtendedNSXPCConnection*)newConnection).auditToken);
SecTaskValidateForRequirement(taskRef, (__bridge CFStringRef)(requirementString))
```
{% endcode %}

यदि एक डेवलपर क्लाइंट के संस्करण की जांच करना नहीं चाहता, तो वह कम से कम यह जांच सकता है कि क्लाइंट प्रोसेस इंजेक्शन के प्रति संवेदनशील नहीं है:

{% code overflow="wrap" %}
```objectivec
[...]
CFDictionaryRef csInfo = NULL;
SecCodeCopySigningInformation(code, kSecCSDynamicInformation, &csInfo);
uint32_t csFlags = [((__bridge NSDictionary *)csInfo)[(__bridge NSString *)kSecCodeInfoStatus] intValue];
const uint32_t cs_hard = 0x100;        // don't load invalid page.
const uint32_t cs_kill = 0x200;        // Kill process if page is invalid
const uint32_t cs_restrict = 0x800;    // Prevent debugging
const uint32_t cs_require_lv = 0x2000; // Library Validation
const uint32_t cs_runtime = 0x10000;   // hardened runtime
if ((csFlags & (cs_hard | cs_require_lv)) {
return Yes; // Accept connection
}
```
<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग प्राप्त करें**](https://peass.creator-spring.com)
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स शेयर करें।

</details>
