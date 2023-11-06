# macOS XPC कनेक्टिंग प्रक्रिया जांच

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एकल [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह।
* [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल** हों या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का पालन करें**।**
* **अपने हैकिंग ट्रिक्स को** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में पीआर जमा करके** अपना योगदान दें।

</details>

## XPC कनेक्टिंग प्रक्रिया जांच

जब XPC सेवा के लिए एक कनेक्शन स्थापित किया जाता है, सर्वर यह जांचेगा कि कनेक्शन अनुमति दी जाती है या नहीं। ये वे जांचें हैं जो आमतौर पर किए जाते हैं:

1. जांचें कि कनेक्टिंग **प्रक्रिया Apple-साइन्ड** प्रमाणपत्र के साथ है (केवल Apple द्वारा दिया जाता है)।
* यदि यह **सत्यापित नहीं** होता है, तो हमलावर किसी भी अन्य जांच के साथ मेल खाने के लिए एक **नकली प्रमाणपत्र** बना सकता है।
2. जांचें कि कनेक्टिंग प्रक्रिया **संगठन के प्रमाणपत्र** के साथ हस्ताक्षरित है (टीम ID सत्यापन)।
* यदि यह **सत्यापित नहीं** होता है, तो किसी भी डेवलपर प्रमाणपत्र का उपयोग करके Apple के साथ संपर्क स्थापित करने के लिए उपयोग किया जा सकता है।
3. जांचें कि कनेक्टिंग प्रक्रिया में **एक उचित बंडल ID** है।
* यदि यह **सत्यापित नहीं** होता है, तो किसी भी उपकरण को उपयोग करने के लिए **एक ही संगठन द्वारा हस्ताक्षरित** टूल का उपयोग किया जा सकता है जो XPC सेवा के साथ संवाद करने के लिए उपयोग किया जा सकता है।
4. (4 या 5) जांचें कि कनेक्टिंग प्रक्रिया में **उचित सॉफ़्टवेयर संस्करण नंबर** है।
* यदि यह **सत्यापित नहीं** होता है, तो पुराने, सुरक्षित ग्राहक, प्रक्रिया इंजेक्शन के लिए संक्रमित हो सकते हैं, भले ही अन्य जांचें स्थान पर हों।
5. (4 या 5) जांचें कि कनेक्टिंग प्रक्रिया में खतरनाक entitlements के बिना हार्डन रनटाइम है (जैसे वे जो विचारशील पुस्तकालयों को लोड करने या DYLD env vars का उपयोग करने की अनुमति देते हैं)
1. यदि यह **सत्यापित नहीं** होता है, तो ग्राहक कोड इंजेक्शन के प्रति संक्रमित हो सकता है
6. जांचें कि कनेक्टिंग प्रक्रिया में एक entitlement है जो इसे सेवा से कनेक्ट करने की अनुमति देता है। यह Apple बाइनरी के लिए लागू होता है।
7. **सत्यापन** कनेक्ट करने वाले **क्लाइंट के ऑडिट टोकन** पर आधारित होना चाहिए, न कि इसकी प्रक्रिया ID (**PID**), क्योंकि पिछले को PID पुनः उपयोग करने के हमलों से बचाता है।
* डेवलपर्स आमतौर पर ऑडिट टोकन API कॉल का उपयोग नहीं करते हैं क्योंकि यह **निजी** होता है, इसलिए Apple किसी भी समय इसे **बदल सकता है**। इसके अलावा, Mac App Store ऐप्स में निजी API का उपयोग अनुमति नहीं है।
* **`xpc_dictionary_get_audit_token`** का उपयोग **`xpc_connection_get_audit_token`** के बजाय किया जाना चाहिए, क्योंकि इसका उपयोग विशेष परिस्थितियों में [विकल्पित रूप से संक्रमित हो सकता है](https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/)।

### संचार हमले

PID पुनः उपयो
```objectivec
- (BOOL)listener:(NSXPCListener *)listener shouldAcceptNewConnection:(NSXPCConnection *)newConnection {
//Check connection
return YES;
}
```
{% endcode %}

ऑब्जेक्ट NSXPCConnection में एक **निजी** प्रॉपर्टी **`auditToken`** होती है (जो उपयोग की जानी चाहिए लेकिन बदल सकती है) और एक **सार्वजनिक** प्रॉपर्टी **`processIdentifier`** होती है (जिसे उपयोग नहीं करना चाहिए).

कनेक्टिंग प्रक्रिया को निम्नलिखित तरीके से सत्यापित किया जा सकता है:

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

यदि एक डेवलपर को यह जांचना नहीं है कि क्लाइंट का संस्करण क्या है, तो उसे कम से कम प्रक्रिया इंजेक्शन के प्रति संवेदनशील नहीं होने की जांच कर सकता है:

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
{% endcode %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें और PRs सबमिट करें** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को।**

</details>
