# macOS Dirty NIB

{% hint style="success" %}
Learn & practice AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Learn & practice GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Check the [**subscription plans**](https://github.com/sponsors/carlospolop)!
* **Join the** 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) or the [**telegram group**](https://t.me/peass) or **follow** us on **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Share hacking tricks by submitting PRs to the** [**HackTricks**](https://github.com/carlospolop/hacktricks) and [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}

**तकनीक के बारे में अधिक जानकारी के लिए मूल पोस्ट देखें: [https://blog.xpnsec.com/dirtynib/**](https://blog.xpnsec.com/dirtynib/).** यहाँ एक सारांश है:

NIB फ़ाइलें, जो Apple के विकास पारिस्थितिकी तंत्र का हिस्सा हैं, **UI तत्वों** और उनके इंटरैक्शन को परिभाषित करने के लिए होती हैं। इनमें विंडो और बटन जैसी अनुक्रमित वस्तुएं शामिल होती हैं, और इन्हें रनटाइम पर लोड किया जाता है। इनके निरंतर उपयोग के बावजूद, Apple अब अधिक व्यापक UI प्रवाह दृश्यता के लिए Storyboards की सिफारिश करता है।

### NIB फ़ाइलों के साथ सुरक्षा चिंताएँ
यह ध्यान रखना महत्वपूर्ण है कि **NIB फ़ाइलें सुरक्षा जोखिम हो सकती हैं**। इनमें **मनमाने आदेशों को निष्पादित करने** की क्षमता होती है, और एक ऐप के भीतर NIB फ़ाइलों में परिवर्तन Gatekeeper को ऐप को निष्पादित करने से नहीं रोकते, जो एक महत्वपूर्ण खतरा है।

### Dirty NIB इंजेक्शन प्रक्रिया
#### NIB फ़ाइल बनाना और सेट करना
1. **प्रारंभिक सेटअप**:
- XCode का उपयोग करके एक नई NIB फ़ाइल बनाएं।
- इंटरफ़ेस में एक ऑब्जेक्ट जोड़ें, इसकी कक्षा को `NSAppleScript` पर सेट करें।
- उपयोगकर्ता परिभाषित रनटाइम गुणों के माध्यम से प्रारंभिक `source` संपत्ति को कॉन्फ़िगर करें।

2. **कोड निष्पादन गैजेट**:
- सेटअप AppleScript को मांग पर चलाने की सुविधा प्रदान करता है।
- `Apple Script` ऑब्जेक्ट को सक्रिय करने के लिए एक बटन एकीकृत करें, विशेष रूप से `executeAndReturnError:` चयनकर्ता को ट्रिगर करना।

3. **परीक्षण**:
- परीक्षण उद्देश्यों के लिए एक सरल Apple Script:
```bash
set theDialogText to "PWND"
display dialog theDialogText
```
- XCode डिबगर में चलाकर और बटन पर क्लिक करके परीक्षण करें।

#### एक एप्लिकेशन को लक्षित करना (उदाहरण: Pages)
1. **तैयारी**:
- लक्षित ऐप (जैसे, Pages) को एक अलग निर्देशिका (जैसे, `/tmp/`) में कॉपी करें।
- Gatekeeper समस्याओं से बचने के लिए ऐप को प्रारंभ करें और इसे कैश करें।

2. **NIB फ़ाइल को अधिलेखित करना**:
- एक मौजूदा NIB फ़ाइल (जैसे, About Panel NIB) को तैयार की गई DirtyNIB फ़ाइल से बदलें।

3. **निष्पादन**:
- ऐप के साथ इंटरैक्ट करके निष्पादन को ट्रिगर करें (जैसे, `About` मेनू आइटम का चयन करना)।

#### प्रमाण का सिद्धांत: उपयोगकर्ता डेटा तक पहुँच
- उपयोगकर्ता की सहमति के बिना फ़ोटो जैसी उपयोगकर्ता डेटा तक पहुँचने और निकालने के लिए AppleScript को संशोधित करें।

### कोड नमूना: दुर्भावनापूर्ण .xib फ़ाइल
- मनमाने कोड को निष्पादित करने का प्रदर्शन करने वाली [**दुर्भावनापूर्ण .xib फ़ाइल का एक नमूना**](https://gist.github.com/xpn/16bfbe5a3f64fedfcc1822d0562636b4) देखें और समीक्षा करें।

### लॉन्च प्रतिबंधों का समाधान
- लॉन्च प्रतिबंध अप्रत्याशित स्थानों (जैसे, `/tmp`) से ऐप के निष्पादन में बाधा डालते हैं।
- यह पहचानना संभव है कि कौन से ऐप लॉन्च प्रतिबंधों से सुरक्षित नहीं हैं और उन्हें NIB फ़ाइल इंजेक्शन के लिए लक्षित करें।

### अतिरिक्त macOS सुरक्षा
macOS Sonoma से आगे, ऐप बंडलों के भीतर संशोधन प्रतिबंधित हैं। हालाँकि, पहले के तरीकों में शामिल थे:
1. ऐप को एक अलग स्थान (जैसे, `/tmp/`) में कॉपी करना।
2. प्रारंभिक सुरक्षा को बायपास करने के लिए ऐप बंडल के भीतर निर्देशिकाओं का नाम बदलना।
3. Gatekeeper के साथ पंजीकरण करने के लिए ऐप चलाने के बाद, ऐप बंडल में संशोधन करना (जैसे, MainMenu.nib को Dirty.nib से बदलना)।
4. निर्देशिकाओं का नाम वापस बदलना और इंजेक्ट की गई NIB फ़ाइल को निष्पादित करने के लिए ऐप को फिर से चलाना।

**नोट**: हाल के macOS अपडेट ने Gatekeeper कैशिंग के बाद ऐप बंडलों के भीतर फ़ाइल संशोधनों को रोककर इस शोषण को कम कर दिया है, जिससे यह शोषण अप्रभावी हो गया है।


{% hint style="success" %}
Learn & practice AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Learn & practice GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Check the [**subscription plans**](https://github.com/sponsors/carlospolop)!
* **Join the** 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) or the [**telegram group**](https://t.me/peass) or **follow** us on **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Share hacking tricks by submitting PRs to the** [**HackTricks**](https://github.com/carlospolop/hacktricks) and [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}
