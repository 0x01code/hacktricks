# Hash Length Extension Attack

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

#### [WhiteIntel](https://whiteintel.io)

<figure><img src="../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) एक **डार्क-वेब** द्वारा संचालित सर्च इंजन है जो **मुफ्त** कार्यक्षमताएँ प्रदान करता है ताकि यह जांचा जा सके कि क्या कोई कंपनी या उसके ग्राहक **समझौता** किए गए हैं **चोरी करने वाले मालवेयर** द्वारा।

WhiteIntel का प्राथमिक लक्ष्य जानकारी-चोरी करने वाले मालवेयर के परिणामस्वरूप अकाउंट टेकओवर और रैनसमवेयर हमलों से लड़ना है।

आप उनकी वेबसाइट देख सकते हैं और उनके इंजन को **मुफ्त** में आजमा सकते हैं:

{% embed url="https://whiteintel.io" %}

***

## Summary of the attack

कल्पना करें कि एक सर्वर कुछ **डेटा** को **जोड़कर** एक **गुप्त** को कुछ ज्ञात स्पष्ट पाठ डेटा पर **हैश** कर रहा है। यदि आप जानते हैं:

* **गुप्त की लंबाई** (यह एक दिए गए लंबाई रेंज से भी ब्रूटफोर्स किया जा सकता है)
* **स्पष्ट पाठ डेटा**
* **एल्गोरिदम (और यह इस हमले के प्रति संवेदनशील है)**
* **पैडिंग ज्ञात है**
* आमतौर पर एक डिफ़ॉल्ट का उपयोग किया जाता है, इसलिए यदि अन्य 3 आवश्यकताएँ पूरी होती हैं, तो यह भी है
* पैडिंग गुप्त+डेटा की लंबाई के आधार पर भिन्न होती है, यही कारण है कि गुप्त की लंबाई की आवश्यकता है

तो, एक **हमलावर** के लिए **डेटा** को **जोड़ना** और **पिछले डेटा + जोड़े गए डेटा** के लिए एक वैध **हस्ताक्षर** उत्पन्न करना संभव है।

### How?

बुनियादी रूप से संवेदनशील एल्गोरिदम पहले **डेटा के एक ब्लॉक को हैश** करके हैश उत्पन्न करते हैं, और फिर, **पहले से** बनाए गए **हैश** (राज्य) से, वे **अगले डेटा के ब्लॉक को जोड़ते हैं** और **इसे हैश करते हैं**।

फिर, कल्पना करें कि गुप्त "गुप्त" है और डेटा "डेटा" है, "गुप्तडेटा" का MD5 6036708eba0d11f6ef52ad44e8b74d5b है।\
यदि एक हमलावर "जोड़ें" स्ट्रिंग को जोड़ना चाहता है, तो वह कर सकता है:

* 64 "A"s का MD5 उत्पन्न करें
* पहले से प्रारंभ किए गए हैश की स्थिति को 6036708eba0d11f6ef52ad44e8b74d5b में बदलें
* "जोड़ें" स्ट्रिंग को जोड़ें
* हैश को समाप्त करें और परिणामी हैश "गुप्त" + "डेटा" + "पैडिंग" + "जोड़ें" के लिए एक **वैध** होगा

### **Tool**

{% embed url="https://github.com/iagox86/hash_extender" %}

### References

आप इस हमले को अच्छी तरह से समझा सकते हैं [https://blog.skullsecurity.org/2012/everything-you-need-to-know-about-hash-length-extension-attacks](https://blog.skullsecurity.org/2012/everything-you-need-to-know-about-hash-length-extension-attacks)

#### [WhiteIntel](https://whiteintel.io)

<figure><img src="../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) एक **डार्क-वेब** द्वारा संचालित सर्च इंजन है जो **मुफ्त** कार्यक्षमताएँ प्रदान करता है ताकि यह जांचा जा सके कि क्या कोई कंपनी या उसके ग्राहक **समझौता** किए गए हैं **चोरी करने वाले मालवेयर** द्वारा।

WhiteIntel का प्राथमिक लक्ष्य जानकारी-चोरी करने वाले मालवेयर के परिणामस्वरूप अकाउंट टेकओवर और रैनसमवेयर हमलों से लड़ना है।

आप उनकी वेबसाइट देख सकते हैं और उनके इंजन को **मुफ्त** में आजमा सकते हैं:

{% embed url="https://whiteintel.io" %}

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
