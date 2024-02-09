# euid, ruid, suid

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित देखना चाहते हैं**? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का एक्सेस चाहिए**? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) की जाँच करें!
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मेरा** **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)** का पालन करें**.
* **अपने हैकिंग ट्रिक्स साझा करें, [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) पर PR जमा करके**।

</details>

### उपयोगकर्ता पहचान चरण

- **`ruid`**: **रियल उपयोगकर्ता आईडी** उस उपयोगकर्ता को दर्शाता है जिसने प्रक्रिया प्रारंभ की है।
- **`euid`**: **प्रभावी उपयोगकर्ता आईडी** के रूप में जाना जाता है, यह उपयोगकर्ता पहचान को स्थापित करने के लिए सिस्टम द्वारा उपयोग किया जाता है। सामान्यत: `euid` `ruid` का प्रतिबिम्ब करता है, सिवाय उदाहरणों के जैसे कि SetUID बाइनरी क्रियान्वयन, जहां `euid` फ़ाइल के मालिक की पहचान लेता है, इसलिए विशेष कार्यान्वयन अनुमतियाँ प्रदान करता है।
- **`suid`**: यह **सहेजी गई उपयोगकर्ता आईडी** महत्वपूर्ण है जब एक उच्च-विशेषाधिकार प्रक्रिया (सामान्यत: जड़) को कुछ कार्य करने के लिए अस्थायी रूप से अपने विशेषाधिकारों से वंचित करने की आवश्यकता होती है, केवल इसके बाद अपनी प्रारंभिक उच्च स्थिति को पुनः प्राप्त करने के लिए।

#### महत्वपूर्ण नोट
एक प्रक्रिया जो रूट के तहत काम नहीं कर रही है, केवल अपना `euid` मौजूदा `ruid`, `euid`, या `suid` के समान बना सकती है।

### set*uid फ़ंक्शनों की समझ

- **`setuid`**: प्राथमिक धारणाओं के विपरीत, `setuid` मुख्य रूप से `ruid` को संशोधित करता है बल्कि `euid`। विशेष रूप से विशेषाधिकारी प्रक्रियाओं के लिए, यह निर्दिष्ट उपयोगकर्ता के साथ `ruid`, `euid`, और `suid` को समान करता है, अक्सर रूट, इन IDs को मजबूत बनाने के कारण। विस्तृत अनुशासन मिल सकता है [setuid man page](https://man7.org/linux/man-pages/man2/setuid.2.html) में।
- **`setreuid`** और **`setresuid`**: ये फ़ंक्शन `ruid`, `euid`, और `suid` को सूक्ष्म रूप से समायोजित करने की अनुमति देते हैं। हालांकि, उनकी क्षमताएँ प्रक्रिया के विशेषाधिकार स्तर पर निर्भर हैं। गैर-रूट प्रक्रियाओं के लिए, संशोधन मौजूदा मूल्यों की सीमित होती है `ruid`, `euid`, और `suid` की। विपरीत, रूट प्रक्रियाएँ या उनके पास `CAP_SETUID` क्षमता हो सकती हैं इन IDs को इसके मान को निर्धारित कर सकती हैं। अधिक जानकारी प्राप्त की जा सकती है [setresuid man page](https://man7.org/linux/man-pages/man2/setresuid.2.html) और [setreuid man page](https://man7.org/linux/man-pages/man2/setreuid.2.html) से।

ये कार्यक्षमताएँ सुरक्षा उपाय के रूप में नहीं डिज़ाइन की गई हैं बल्कि इच्छित कार्यान्वयन फ्लो को सुविधाजनक बनाने के लिए, जैसे जब एक कार्यक्रम अपने प्रभावी उपयोगकर्ता आईडी को संशोधित करके किसी अन्य उपयोगकर्ता की पहचान अपनाता है।

विशेष रूप से, जब `setuid` रूट के लिए विशेषाधिकार उन्नयन के लिए एक सामान्य विकल्प हो सकता है (क्योंकि यह सभी IDs को रूट के साथ समान करता है), तो इन फ़ंक्शनों के बीच भिन्नता को समझने और उपयोगकर्ता आईडी के व्यवहार को विभिन्न परिस्थितियों में कैसे नियंत्रित किया जाए, यह महत्वपूर्ण है।

### लिनक्स में कार्यक्रम निष्पादन तंत्र

#### **`execve` सिस्टम कॉल**
- **कार्यक्षमता**: `execve` पहले तर्क द्वारा निर्धारित एक कार्यक्रम प्रारंभ करता है। यह दो सरणियों, `argv` के लिए तर्क और `envp` के लिए पर्यावरण, लेता है।
- **व्यवहार**: यह कॉलर की मेमोरी स्थान को बनाए रखता है लेकिन स्टैक, हीप, और डेटा सेगमेंट को ताजगी देता है। कार्यक्रम कोड नए कार्यक्रम द्वारा बदल दिया जाता है।
- **उपयोगकर्ता आईडी संरक्षण**:
- `ruid`, `euid`, और सहायक समूह आईडी अपरिवर्तित रहते हैं।
- अगर नया कार्यक्रम SetUID बिट सेट है तो `euid` में सूक्ष्म परिवर्तन हो सकता है।
- `suid` को `euid` से अपडेट किया जाता है `execve` के पश्चात।
- **दस्तावेज़ीकरण**: विस्तृत जानकारी [`execve` man page](https://man7.org/linux/man-pages/man2/execve.2.html) पर उपलब्ध है।

#### **`system` फ़ंक्शन**
- **कार्यक्षमता**: `execve` की विपरीत, `system` `fork` का उपयोग करके एक बच्चा प्रक्रिया बनाता है और उस बच्चा प्रक्रिया में `execl` का उपयोग करके एक कमांड को निष्पादित करता है।
- **कमांड निष्पादन**: `execl("/bin/sh", "sh", "-c", command, (char *) NULL);` के माध्यम से `sh` के माध्यम से कमांड निष्पादित करता है।
- **व्यवहार**: `execl` `execve` का एक रूप होने के बावजूद, यह एक नए बच्चा प्रक्रिया के संदर्भ में समान रूप से काम करता है।
- **दस्तावेज़ीकरण**: अधिक जानकारी [`system` man page](https://man7.org/linux/man-pages/man3/system.3.html) से प्राप्त की जा सकती है।

#### **`bash` और `sh` का SUID के साथ व्यवहार**
- **`bash`**:
- `-p` विकल्प है जो `euid` और `ruid` को कैसे व्यवहारित किया जाता है, इस पर प्रभाव पड़ता है।
- `-p` के बिना, अगर वे पहले से भिन्न हैं, तो `bash` `euid` को `ruid` सेट करता है।
- `-p` के साथ, प्रारंभिक `euid` संरक्षित रहता है।
- अधिक विवरण [`bash` man page](https://linux.die.net/man/1/bash) पर उपलब्ध है।
- **`sh`**:
- `bash` में `-p` के समान एक तंत्र नहीं है।
- उपयोगकर
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

* `ruid` और `euid` पहले 99 (कोई नहीं) और 1000 (फ्रैंक) के रूप में शुरू होते हैं।
* `setuid` दोनों को 1000 के लिए संरेखित करता है।
* `system` `/bin/bash -c id` को निष्पादित करता है क्योंकि sh से bash तक का सिमलिंक है।
* `-p` के बिना `bash` `euid` को `ruid` के समान करने के लिए समायोजित करता है, जिससे दोनों 99 (कोई नहीं) होते हैं।

#### मामला 2: सिस्टम के साथ setreuid का उपयोग करना

**सी कोड**:
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
**व्यावहारिकरण और परिणाम:**
```bash
bash-4.2$ $ ./b
uid=1000(frank) gid=99(nobody) groups=99(nobody) context=system_u:system_r:unconfined_service_t:s0
```
**विश्लेषण:**

* `setreuid` दोनों ruid और euid को 1000 पर सेट करता है।
* `system` बैश को आमंत्रित करता है, जो उनके समानता के कारण उपयोगकर्ता आईडी को बनाए रखता है, असल में frank के रूप में कार्य करता है।

#### मामला 3: execve के साथ setuid का उपयोग
लक्ष्य: setuid और execve के बीच बातचीत का अन्वेषण।
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
**व्यावहारिक और परिणाम:**
```bash
bash-4.2$ $ ./c
uid=99(nobody) gid=99(nobody) euid=1000(frank) groups=99(nobody) context=system_u:system_r:unconfined_service_t:s0
```
**विश्लेषण:**

* `ruid` 99 रहता है, लेकिन euid 1000 पर सेट किया गया है, setuid के प्रभाव के साथ।

**सी कोड उदाहरण 2 (बैश को कॉल करना):**
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
**व्यावहारिकरण और परिणाम:**
```bash
bash-4.2$ $ ./d
bash-4.2$ $ id
uid=99(nobody) gid=99(nobody) groups=99(nobody) context=system_u:system_r:unconfined_service_t:s0
```
**विश्लेषण:**

* यद्यपि `setuid` द्वारा `euid` को 1000 पर सेट किया जाता है, लेकिन `-p` की अनुपस्थिति के कारण `bash` `ruid` (99) पर euid को रीसेट कर देता है।

**सी संकेत उदाहरण 3 (bash -p का उपयोग करके):**
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
**व्यावहारिकरण और परिणाम:**
```bash
bash-4.2$ $ ./e
bash-4.2$ $ id
uid=99(nobody) gid=99(nobody) euid=100
```
## संदर्भ
* [https://0xdf.gitlab.io/2022/05/31/setuid-rabbithole.html#testing-on-jail](https://0xdf.gitlab.io/2022/05/31/setuid-rabbithole.html#testing-on-jail)


<details>

<summary><strong>जीरो से हीरो तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को हैकट्रिक्स में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या हैकट्रिक्स को पीडीएफ में डाउनलोड** करने का एक्सेस चाहिए? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) की जाँच करें!
* [**दी पीएस फैमिली**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**एनएफटी**](https://opensea.io/collection/the-peass-family) संग्रह।
* [**आधिकारिक PEASS और हैकट्रिक्स स्वैग**](https://peass.creator-spring.com) प्राप्त करें।
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर फॉलो करें 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)**।**
* **अपने हैकिंग ट्रिक्स साझा करें, [हैकट्रिक्स रेपो](https://github.com/carlospolop/hacktricks) और [हैकट्रिक्स-क्लाउड रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके**।

</details>
