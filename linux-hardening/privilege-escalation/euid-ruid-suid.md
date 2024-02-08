# euid, ruid, suid

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप चाहते हैं कि आपकी **कंपनी को हैकट्रिक्स में विज्ञापित** किया जाए? या क्या आप **PEASS के नवीनतम संस्करण का उपयोग करना चाहते हैं या हैकट्रिक्स को पीडीएफ में डाउनलोड करना चाहते हैं**? [**सब्सक्रिप्शन प्लान**](https://github.com/sponsors/carlospolop) की जांच करें!
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह **The PEASS Family** की खोज करें
* [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मेरा** **ट्विटर** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)** का पालन करें**.
* **हैकिंग ट्रिक्स साझा करें, [हैकट्रिक्स रेपो](https://github.com/carlospolop/hacktricks) और [हैकट्रिक्स-क्लाउड रेपो](https://github.com/carlospolop/hacktricks-cloud)** को PR जमा करके।

</details>

### उपयोगकर्ता पहचान चरण

- **`ruid`**: **वास्तविक उपयोगकर्ता आईडी** उस उपयोगकर्ता को दर्शाता है जिसने प्रक्रिया प्रारंभ की है।
- **`euid`**: **प्रभावी उपयोगकर्ता आईडी** के रूप में जाना जाता है, यह प्रक्रिया विशेषाधिकारों का निर्धारण करने के लिए प्रणाली द्वारा उपयोग किया जाने वाला उपयोगकर्ता पहचान है। सामान्यत: `euid` `ruid` का प्रतिबिम्ब करता है, केवल सेटयूआईडी बाइनरी निष्पादन जैसे स्थितियों को छोड़कर, जहां `euid` फ़ाइल के मालिक की पहचान लेता है, जिससे विशेष कार्यान्वयन अनुमतियाँ प्रदान की जाती है।
- **`suid`**: यह **सहेजी गई उपयोगकर्ता आईडी** महत्वपूर्ण है जब एक उच्च-विशेषाधिकार प्रक्रिया (सामान्यत: जड़) को कुछ कार्य करने के लिए अस्थायी रूप से अपनी विशेषाधिकारों को छोड़ने की आवश्यकता होती है, केवल बाद में अपनी प्रारंभिक उच्च स्थिति को पुनः प्राप्त करने के लिए।

#### महत्वपूर्ण नोट
एक प्रक्रिया जो रूट के तहत काम नहीं कर रही है, केवल अपना `euid` मौजूदा `ruid`, `euid`, या `suid` के समान कर सकती है।

### set*uid फ़ंक्शन को समझना

- **`setuid`**: प्राथमिक धारणाओं के विपरीत, `setuid` मुख्य रूप से `ruid` को संशोधित करता है बल्कि `euid`। विशेष रूप से विशेषाधिकारी प्रक्रियाओं के लिए, यह निर्धारित उपयोगकर्ता के साथ `ruid`, `euid`, और `suid` को समान करता है, जिससे इन आईडी को ओवरराइड करने के कारण ये आईडी स्थिर हो जाते हैं। विस्तृत अनुशासन [`setuid` मैन पेज](https://man7.org/linux/man-pages/man2/setuid.2.html) में पाए जा सकते हैं।
- **`setreuid`** और **`setresuid`**: ये फ़ंक्शन `ruid`, `euid`, और `suid` का सूक्ष्म समायोजन करने की अनुमति देते हैं। हालांकि, उनकी क्षमताएँ प्रक्रिया के विशेषाधिकार स्तर पर निर्भर होती हैं। गैर-रूट प्रक्रियाओं के लिए, संशोधन मौजूदा मूल्यों को सीमित करते हैं `ruid`, `euid`, और `suid`। विपरीत, रूट प्रक्रियाएँ या `CAP_SETUID` क्षमता वाले वे इन आईडी को अपनी इच्छानुसारी मानें दे सकते हैं। अधिक जानकारी [`setresuid` मैन पेज](https://man7.org/linux/man-pages/man2/setresuid.2.html) और [`setreuid` मैन पेज](https://man7.org/linux/man-pages/man2/setreuid.2.html) से प्राप्त की जा सकती है।

ये कार्यक्षमताएँ सुरक्षा उपाय के रूप में नहीं डिज़ाइन की गई हैं बल्कि इच्छित ऑपरेशनल फ्लो को सुविधाजनक बनाने के लिए, जैसे जब एक कार्यक्रम अपने प्रभावी उपयोगकर्ता आईडी को बदलकर किसी अन्य उपयोगकर्ता की पहचान अपनाता है।

विशेष रूप से, जब `setuid` रूट के लिए उच्चता उन्नयन के लिए एक सामान्य विकल्प हो सकता है (क्योंकि यह सभी आईडी को रूट के साथ समान करता है), तो इन फ़ंक्शन के बीच भिन्नता करना उपयोगकर्ता आईडी व्यवहारों को विभिन्न परिदृश्यों में समझने और नियंत्रित करने के लिए महत्वपूर्ण है।

### लिनक्स में कार्यक्रम निष्पादन तंत्र

#### **`execve` सिस्टम कॉल**
- **कार्यक्षमता**: `execve` पहले विधान द्वारा निर्धारित एक कार्यक्रम प्रारंभ करता है। यह दो सरणियों, `argv` के लिए तर्क और `envp` के लिए पर्यावरण लेता है।
- **व्यवहार**: यह कॉलर की मेमोरी स्थान को बनाए रखता है लेकिन स्टैक, हीप, और डेटा सेगमेंट को ताजगी देता है। कार्यक्रम कोड नए कार्यक्रम द्वारा बदल दिया जाता है।
- **उपयोगकर्ता आईडी संरक्षण**:
- `ruid`, `euid`, और सहायक समूह आईडी अपरिवर्तित रहते हैं।
- यदि नया कार्यक्रम SetUID बिट सेट है तो `euid` में सूक्ष्म परिवर्तन हो सकता है।
- `suid` को `euid` से अपडेट किया जाता है पोस्ट-निष्पादन से।
- **दस्तावेज़ीकरण**: विस्तृत जानकारी [`execve` मैन पेज](https://man7.org/linux/man-pages/man2/execve.2.html) पर उपलब्ध है।

#### **`system` फ़ंक्शन**
- **कार्यक्षमता**: `execve` की विपरीत, `system` `fork` का उपयोग करके एक बच्चा प्रक्रिया बनाता है और उस बच्चा प्रक्रिया में `execl` का उपयोग करके एक कमांड को निष्पादित करता है।
- **कमांड निष्पादन**: `execl("/bin/sh", "sh", "-c", command, (char *) NULL);` के माध्यम से कमांड को निष्पादित करता है।
- **व्यवहार**: `execl` `execve` का एक रूप है, यह एक नए बच्चा प्रक्रिया के संदर्भ में समान रूप से काम करता है।
- **दस्तावेज़ीकरण**: अधिक जानकारी [`system` मैन पेज](https://man7.org/linux/man-pages/man3/system.3.html) से प्राप्त की जा सकती है।

#### **`bash` और `sh` का SUID के साथ व्यवहार**
- **`bash`**:
- `-p` विकल्प है जो `euid` और `ruid` को कैसे व्यवहार किया जाता है, पर प्रभाव डालता है।
- `-p` के बिना, `bash` `euid` को `ruid` से समान करता है अगर वे पहले भिन्न ह
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
* `system` बैश को आमंत्रित करता है, जो उनके समानता के कारण उपयोगकर्ता आईडी को बनाए रखता है, असल में फ्रैंक के रूप में कार्य करता है।

#### मामला 3: execve के साथ setuid का उपयोग
उद्देश्य: setuid और execve के बीच बातचीत का अन्वेषण।
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

* `ruid` 99 बना रहता है, लेकिन euid 1000 पर सेट किया जाता है, setuid के प्रभाव के साथ।

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
**क्रियान्वयन और परिणाम:**
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

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ हैकट्रिक्स क्लाउड ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 ट्विटर 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ ट्विच 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 यूट्यूब 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी का हैकट्रिक्स में विज्ञापित करना चाहते हैं**? या क्या आप **PEASS के नवीनतम संस्करण का उपयोग करना चाहते हैं या हैकट्रिक्स को पीडीएफ में डाउनलोड करना चाहते हैं**? [**सब्सक्रिप्शन प्लान**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**द पीएस फैमिली**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**एनएफटी**](https://opensea.io/collection/the-peass-family) संग्रह।
* [**आधिकारिक PEASS और हैकट्रिक्स स्वैग**](https://peass.creator-spring.com) प्राप्त करें।
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)** पर फॉलो करें**।
* **हैकिंग ट्रिक्स साझा करें, [हैकट्रिक्स रेपो](https://github.com/carlospolop/hacktricks) और [हैकट्रिक्स-क्लाउड रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके**।

</details>
