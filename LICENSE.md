<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग प्राप्त करें**](https://peass.creator-spring.com)
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* 💬 [**Discord समूह में शामिल हों**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **Twitter पर** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) **का अनुसरण करें.**
* **अपनी हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में.

</details>

<a rel="license" href="https://creativecommons.org/licenses/by-nc/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://licensebuttons.net/l/by-nc/4.0/88x31.png" /></a><br>कॉपीराइट © Carlos Polop 2021. जहां अन्यथा निर्दिष्ट (पुस्तक में कॉपी की गई बाहरी जानकारी मूल लेखकों की है), <a href="https://github.com/carlospolop/hacktricks">HACK TRICKS</a> पर Carlos Polop द्वारा लिखित पाठ <a href="https://creativecommons.org/licenses/by-nc/4.0/">Creative Commons Attribution-NonCommercial 4.0 International (CC BY-NC 4.0)</a> के तहत लाइसेंस प्राप्त है।

लाइसेंस: Attribution-NonCommercial 4.0 International (CC BY-NC 4.0)<br>
मानव पठनीय लाइसेंस: https://creativecommons.org/licenses/by-nc/4.0/<br>
पूर्ण कानूनी शर्तें: https://creativecommons.org/licenses/by-nc/4.0/legalcode<br>
फॉर्मेटिंग: https://github.com/jmatsushita/Creative-Commons-4.0-Markdown/blob/master/licenses/by-nc.markdown<br>

# creative commons

# Attribution-NonCommercial 4.0 International

Creative Commons Corporation (“Creative Commons”) कोई कानूनी फर्म नहीं है और वह कानूनी सेवाएं या कानूनी सलाह प्रदान नहीं करता है। Creative Commons सार्वजनिक लाइसेंसों का वितरण किसी वकील-ग्राहक या अन्य संबंध का निर्माण नहीं करता है। Creative Commons अपने लाइसेंस और संबंधित जानकारी "जैसी है" आधार पर उपलब्ध कराता है। Creative Commons अपने लाइसेंसों, किसी भी सामग्री जो उनकी शर्तों और शर्तों के तहत लाइसेंस प्राप्त है, या किसी भी संबंधित जानकारी के बारे में कोई वारंटी नहीं देता है। Creative Commons उनके उपयोग से होने वाली किसी भी क्षति के लिए सभी दायित्वों से इनकार करता है।

## Creative Commons Public Licenses का उपयोग करना

Creative Commons सार्वजनिक लाइसेंस एक मानक सेट प्रदान करते हैं जिसे रचनाकार और अन्य अधिकार धारक कॉपीराइट और कुछ अन्य अधिकारों द्वारा संरक्षित सामग्री को साझा करने के लिए उपयोग कर सकते हैं। निम्नलिखित विचार केवल सूचनात्मक उद्देश्यों के लिए हैं, समाप्त नहीं हैं, और हमारे लाइसेंसों का हिस्सा नहीं बनते हैं।

* __लाइसेंस देने वालों के लिए विचार:__ हमारे सार्वजनिक लाइसेंस उन लोगों द्वारा उपयोग के लिए इरादा हैं जो सामग्री को कॉपीराइट और कुछ अन्य अधिकारों द्वारा अन्यथा प्रतिबंधित तरीकों में उपयोग करने के लिए सार्वजनिक अनुमति देने के लिए अधिकृत हैं। हमारे लाइसेंस अपरिवर्तनीय हैं। लाइसेंस देने वालों को इसे लागू करने से पहले वे चुने गए लाइसेंस की शर्तों और शर्तों को पढ़ना और समझना चाहिए। लाइसेंस देने वालों को यह भी सुनिश्चित करना चाहिए कि वे हमारे लाइसेंस लागू करने से पहले सभी आवश्यक अधिकारों को सुरक्षित करें ताकि सार्वजनिक रूप से सामग्री का पुन: उपयोग कर सकें। लाइसेंस देने वालों को उस सामग्री को स्पष्ट रूप से चिह्नित करना चाहिए जो लाइसेंस के अधीन नहीं है। इसमें अन्य CC-लाइसेंस प्राप्त सामग्री, या कॉपीराइट के अपवाद या सीमा के तहत उपयोग की गई सामग्री शामिल है। [लाइसेंस देने वालों के लिए अधिक विचार](http://wiki.creativecommons.org/Considerations_for_licensors_and_licensees#Considerations_for_licensors).

* __सार्वजनिक के लिए विचार:__ हमारे सार्वजनिक लाइसेंसों में से एक का उपयोग करके, एक लाइसेंस देने वाला सार्वजनिक को निर्दिष्ट शर्तों और शर्तों के तहत लाइसेंस प्राप्त सामग्री का उपयोग करने की अनुमति देता है। यदि किसी भी कारण से लाइसेंस देने वाले की अनुमति आवश्यक नहीं है–उदाहरण के लिए, कॉपीराइट के किसी भी लागू अपवाद या सीमा के कारण–तो वह उपयोग लाइसेंस द्वारा विनियमित नहीं है। हमारे लाइसेंस केवल कॉपीराइट और कुछ अन्य अधिकारों के तहत अनुमतियां प्रदान करते हैं जिन्हें लाइसेंस देने वाला अनुदान देने का अधिकार रखता है। लाइसेंस प्राप्त सामग्री का उपयोग अन्य कारणों से भी प्रतिबंधित हो सकता है, जिसमें यह भी शामिल है कि अन्य लोगों के पास सामग्री में कॉपीराइट या अन्य अधिकार हो सकते हैं। एक लाइसेंस देने वाला विशेष अनुरोध कर सकता है, जैसे कि सभी परिवर्तनों को चिह्नित या वर्णित करने का अनुरोध करना। हालांकि हमारे लाइसेंसों द्वारा आवश्यक नहीं है, आपको उचित होने पर उन अनुरोधों का सम्मान करने की सलाह दी जाती है। [सार्वजनिक के लिए अधिक विचार](http://wiki.creativecommons.org/Considerations_for_licensors_and_licensees#Considerations_for_licensees).

# Creative Commons Attribution-NonCommercial 4.0 International Public License

लाइसेंस प्राप्त अधिकारों (नीचे परिभाषित) का अभ्यास करके, आप इस Creative Commons Attribution-NonCommercial 4.0 International Public License ("Public License") की शर्तों और शर्तों से बंधे होने के लिए सहमत होते हैं। इस Public License को एक अनुबंध के रूप में व्याख्या की जा सकती है, आपको इन शर्तों और शर्तों की स्वीकृति के विचार में लाइसेंस प्राप्त अधिकार दिए जाते हैं, और लाइसेंस देने वाला आपको इन शर्तों और शर्तों के तहत लाइसेंस प्राप्त सामग्री उपलब्ध कराने से प्राप्त लाभों के विचार में ऐसे अधिकार देता है।

## धारा 1 – परिभाषाएँ।

a. __अनुकूलित सामग्री__ का अर्थ है कॉपीराइट और समान अधिकारों के अधीन सामग्री जो लाइसेंस प्राप्त सामग्री से व्युत्पन्न होती है या उसके आधार पर होती है
```
Creative Commons is not a party to its public licenses. Notwithstanding, Creative Commons may elect to apply one of its public licenses to material it publishes and in those instances will be considered the “Licensor.” Except for the limited purpose of indicating that material is shared under a Creative Commons public license or as otherwise permitted by the Creative Commons policies published at [creativecommons.org/policies](http://creativecommons.org/policies), Creative Commons does not authorize the use of the trademark “Creative Commons” or any other trademark or logo of Creative Commons without its prior written consent including, without limitation, in connection with any unauthorized modifications to any of its public licenses or any other arrangements, understandings, or agreements concerning use of licensed material. For the avoidance of doubt, this paragraph does not form part of the public licenses.

Creative Commons may be contacted at [creativecommons.org](http://creativecommons.org/).
```
<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग प्राप्त करें**](https://peass.creator-spring.com)
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **अपनी हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में.

</details>
