# उपयोगकर्ता पहचान चर

- **`ruid`**: **वास्तविक उपयोगकर्ता आईडी** उस उपयोगकर्ता को दर्शाता है जिसने प्रक्रिया आरंभ की।
- **`euid`**: **प्रभावी उपयोगकर्ता आईडी** के रूप में जाना जाता है, यह प्रणाली द्वारा प्रक्रिया विशेषाधिकारों की पहचान के लिए उपयोग की जाने वाली उपयोगकर्ता पहचान को दर्शाता है। आमतौर पर, `euid` `ruid` के समान होता है, लेकिन SetUID बाइनरी निष्पादन जैसे मामलों में, `euid` फाइल मालिक की पहचान ग्रहण करता है, जिससे विशिष्ट संचालनात्मक अनुमतियां प्रदान की जाती हैं।
- **`suid`**: यह **सहेजा गया उपयोगकर्ता आईडी** तब महत्वपूर्ण होता है जब एक उच्च-विशेषाधिकार प्रक्रिया (आमतौर पर रूट के रूप में चल रही) को कुछ कार्यों को करने के लिए अस्थायी रूप से अपने विशेषाधिकारों को त्यागने की आवश्यकता होती है, केवल बाद में अपनी प्रारंभिक उच्च स्थिति को पुनः प्राप्त करने के लिए।

#### महत्वपूर्ण नोट
रूट के तहत संचालित न होने वाली प्रक्रिया केवल अपने `euid` को मौजूदा `ruid`, `euid`, या `suid` के अनुरूप बदल सकती है।

### set*uid फंक्शन्स को समझना

- **`setuid`**: प्रारंभिक धारणाओं के विपरीत, `setuid` मुख्य रूप से `ruid` के बजाय `euid` को संशोधित करता है। विशेष रूप से, विशेषाधिकार प्रक्रियाओं के लिए, यह निर्दिष्ट उपयोगकर्ता, अक्सर रूट के साथ `ruid`, `euid`, और `suid` को संरेखित करता है, प्रभावी रूप से इन आईडी को अधिलेखित `suid` के कारण स्थिर करता है। विस्तृत जानकारी [setuid man page](https://man7.org/linux/man-pages/man2/setuid.2.html) पर पाई जा सकती है।
- **`setreuid`** और **`setresuid`**: ये फंक्शन्स `ruid`, `euid`, और `suid` के सूक्ष्म समायोजन की अनुमति देते हैं। हालांकि, उनकी क्षमताएं प्रक्रिया के विशेषाधिकार स्तर पर निर्भर करती हैं। गैर-रूट प्रक्रियाओं के लिए, संशोधन मौजूदा `ruid`, `euid`, और `suid` के मूल्यों तक सीमित हैं। इसके विपरीत, रूट प्रक्रियाओं या उन प्रक्रियाओं के लिए जिनके पास `CAP_SETUID` क्षमता है, इन आईडी को मनमाने मूल्य देने की अनुमति है। अधिक जानकारी [setresuid man page](https://man7.org/linux/man-pages/man2/setresuid.2.html) और [setreuid man page](https://man7.org/linux/man-pages/man2/setreuid.2.html) से प्राप्त की जा सकती है।

ये कार्यक्षमताएं सुरक्षा तंत्र के रूप में नहीं बल्कि इच्छित संचालनात्मक प्रवाह को सुविधाजनक बनाने के लिए डिज़ाइन की गई हैं, जैसे कि जब कोई कार्यक्रम अपनी प्रभावी उपयोगकर्ता आईडी को बदलकर दूसरे उपयोगकर्ता की पहचान अपनाता है।

विशेष रूप से, जबकि `setuid` रूट के लिए विशेषाधिकार उन्नयन के लिए एक सामान्य जाने-माने तरीका हो सकता है (क्योंकि यह सभी आईडी को रूट के साथ संरेखित करता है), विभिन्न परिदृश्यों में उपयोगकर्ता आईडी व्यवहारों को समझने और हेरफेर करने के लिए इन फंक्शन्स के बीच अंतर करना महत्वपूर्ण है।

### लिनक्स में कार्यक्रम निष्पादन तंत्र

#### **`execve` सिस्टम कॉल**
- **कार्यक्षमता**: `execve` पहले तर्क द्वारा निर्धारित कार्यक्रम को आरंभ करता है। यह दो सरणी तर्कों को लेता है, `argv` तर्कों के लिए और `envp` पर्यावरण के लिए।
- **व्यवहार**: यह कॉलर के मेमोरी स्पेस को बनाए रखता है लेकिन स्टैक, हीप, और डेटा सेगमेंट्स को ताज़ा करता है। कार्यक्रम का कोड नए कार्यक्रम से बदल दिया जाता है।
- **उपयोगकर्ता आईडी संरक्षण**:
- `ruid`, `euid`, और सहायक समूह आईडी अपरिवर्तित रहते हैं।
- `euid` में सूक्ष्म परिवर्तन हो सकते हैं यदि नया कार्यक्रम SetUID बिट सेट के साथ हो।
- `suid` निष्पादन के बाद `euid` से अपडेट हो जाता है।
- **दस्तावेज़ीकरण**: विस्तृत जानकारी [`execve` man page](https://man7.org/linux/man-pages/man2/execve.2.html) पर पाई जा सकती है।

#### **`system` फंक्शन**
- **कार्यक्षमता**: `execve` के विपरीत, `system` `fork` का उपयोग करके एक बच्चे की प्रक्रिया बनाता है और उस बच्चे की प्रक्रिया में एक कमांड को `execl` का उपयोग करके निष्पादित करता है।
- **कमांड निष्पादन**: `execl("/bin/sh", "sh", "-c", command, (char *) NULL);` के साथ `sh` के माध्यम से कमांड को निष्पादित करता है।
- **व्यवहार**: चूंकि `execl` `execve` का एक रूप है, यह इसी तरह से काम करता है लेकिन एक नए बच्चे की प्रक्रिया के संदर्भ में।
- **दस्तावेज़ीकरण**: आगे की अंतर्दृष्टि [`system` man page](https://man7.org/linux/man-pages/man3/system.3.html) से प्राप्त की जा सकती है।

#### **SUID के साथ `bash` और `sh` का व्यवहार**
- **`bash`**:
- `-p` विकल्प होता है जो `euid` और `ruid` के उपचार को प्रभावित करता है।
- `-p` के बिना, `bash` `euid` को `ruid` में सेट करता है यदि वे प्रारंभ में भिन्न होते हैं।
- `-p` के साथ, प्रारंभिक `euid` संरक्षित रहता है।
- अधिक विवरण [`bash` man page](https://linux.die.net/man/1/bash) पर पाए जा सकते हैं।
- **`sh`**:
- `bash` में `-p` के समान एक तंत्र नहीं होता है।
- उपयोगकर्ता आईडी के संबंध में व्यवहार का विशेष रूप से उल्लेख नहीं किया गया है, सिवाय `-i` विकल्प के तहत, जो `euid` और `ruid` की समानता के संरक्षण पर जोर देता है।
- अतिरिक्त जानकारी [`sh` man page](https://man7.org/linux/man-pages/man1/sh.1p.html) पर उपलब्ध है।

ये तंत्र, अपने संचालन में विशिष्ट, कार्यक्रमों के निष्पादन और संक्रमण के लिए एक बहुमुखी विकल्पों की श्रृंखला प्रदान करते हैं, विशेष रूप से उपयोगकर्ता आईडी के प्रबंधन और संरक्षण में कैसे विशिष्टताएं होती हैं।

### निष्पादन में उपयोगकर्ता आईडी व्यवहारों का परीक्षण

उदाहरण https://0xdf.gitlab.io/2022/05/31/setuid-rabbithole.html#testing-on-jail से लिए गए हैं, अधिक जानकारी के लिए इसे देखें

#### मामला 1: `setuid` का उपयोग `system` के साथ

**उद्देश्य**: `setuid`
```c
#define _GNU_SOURCE
#include <stdlib.h>
#include <unistd.h>

int main(void) {
setuid(1000);
system("id");
return 0;
}
```
**संकलन और अनुमतियाँ:**
```bash
oxdf@hacky$ gcc a.c -o /mnt/nfsshare/a;
oxdf@hacky$ chmod 4755 /mnt/nfsshare/a
```

```bash
bash-4.2$ $ ./a
uid=99(nobody) gid=99(nobody) groups=99(nobody) context=system_u:system_r:unconfined_service_t:s0
```
**विश्लेषण:**

* `ruid` और `euid` क्रमशः 99 (नोबडी) और 1000 (फ्रैंक) के रूप में शुरू होते हैं।
* `setuid` दोनों को 1000 पर संरेखित करता है।
* `system` `/bin/bash -c id` को निष्पादित करता है क्योंकि sh से bash के लिए सिमलिंक होता है।
* `bash`, `-p` के बिना, `euid` को `ruid` के अनुरूप समायोजित करता है, जिससे दोनों 99 (नोबडी) हो जाते हैं।

#### मामला 2: setreuid का उपयोग करते हुए system के साथ

**C कोड**:
```c
#define _GNU_SOURCE
#include <stdlib.h>
#include <unistd.h>

int main(void) {
setreuid(1000, 1000);
system("id");
return 0;
}
```
**संकलन और अनुमतियाँ:**
```bash
oxdf@hacky$ gcc b.c -o /mnt/nfsshare/b; chmod 4755 /mnt/nfsshare/b
```
**निष्पादन और परिणाम:**
```bash
bash-4.2$ $ ./b
uid=1000(frank) gid=99(nobody) groups=99(nobody) context=system_u:system_r:unconfined_service_t:s0
```
**विश्लेषण:**

* `setreuid` ruid और euid दोनों को 1000 पर सेट करता है।
* `system` bash को आमंत्रित करता है, जो उनकी समानता के कारण उपयोगकर्ता आईडी को बनाए रखता है, प्रभावी रूप से frank के रूप में काम करता है।

#### मामला 3: execve के साथ setuid का उपयोग
उद्देश्य: setuid और execve के बीच इंटरैक्शन का पता लगाना।
```bash
#define _GNU_SOURCE
#include <stdlib.h>
#include <unistd.h>

int main(void) {
setuid(1000);
execve("/usr/bin/id", NULL, NULL);
return 0;
}
```
**निष्पादन और परिणाम:**
```bash
bash-4.2$ $ ./c
uid=99(nobody) gid=99(nobody) euid=1000(frank) groups=99(nobody) context=system_u:system_r:unconfined_service_t:s0
```
**विश्लेषण:**

* `ruid` 99 बना रहता है, परन्तु euid 1000 पर सेट हो जाता है, setuid के प्रभाव के अनुसार।

**C कोड उदाहरण 2 (Bash को कॉल करना):**
```bash
#define _GNU_SOURCE
#include <stdlib.h>
#include <unistd.h>

int main(void) {
setuid(1000);
execve("/bin/bash", NULL, NULL);
return 0;
}
```
**निष्पादन और परिणाम:**
```bash
bash-4.2$ $ ./d
bash-4.2$ $ id
uid=99(nobody) gid=99(nobody) groups=99(nobody) context=system_u:system_r:unconfined_service_t:s0
```
**विश्लेषण:**

* हालांकि `setuid` द्वारा `euid` को 1000 पर सेट किया गया है, `-p` की अनुपस्थिति के कारण `bash` `euid` को `ruid` (99) पर रीसेट कर देता है।

**C कोड उदाहरण 3 (bash -p का उपयोग करते हुए):**
```bash
#define _GNU_SOURCE
#include <stdlib.h>
#include <unistd.h>

int main(void) {
char *const paramList[10] = {"/bin/bash", "-p", NULL};
setuid(1000);
execve(paramList[0], paramList, NULL);
return 0;
}
```
**निष्पादन और परिणाम:**
```bash
bash-4.2$ $ ./e
bash-4.2$ $ id
uid=99(nobody) gid=99(nobody) euid=100
```
# संदर्भ
* [https://0xdf.gitlab.io/2022/05/31/setuid-rabbithole.html#testing-on-jail](https://0xdf.gitlab.io/2022/05/31/setuid-rabbithole.html#testing-on-jail)


<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबरसिक्योरिटी कंपनी** में काम करते हैं? क्या आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे**? या क्या आप **PEASS के नवीनतम संस्करण तक पहुँच चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* **[**💬**](https://emojipedia.org/speech-balloon/) [**Discord group**](https://discord.gg/hRep4RUj7f) में शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर मुझे **फॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपनी हैकिंग ट्रिक्स साझा करें, [hacktricks repo](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud repo](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके**.

</details>
