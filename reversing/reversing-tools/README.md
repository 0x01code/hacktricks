<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप चाहते हैं कि आपकी **कंपनी HackTricks में विज्ञापित हो** या **HackTricks को PDF में डाउनलोड** करें तो [**सदस्यता योजनाएं देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live) **पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें** द्वारा PRs सबमिट करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos।

</details>

# Wasm Decompilation and Wat Compilation Guide

**WebAssembly** के क्षेत्र में, **डिकंपाइलिंग** और **कंपाइलिंग** के लिए उपकरण डेवलपरों के लिए आवश्यक हैं। यह गाइड कुछ ऑनलाइन संसाधनों और सॉफ्टवेयर पेश करता है जो **Wasm (WebAssembly बाइनरी)** और **Wat (WebAssembly पाठ)** फ़ाइलों को संभालने के लिए हैं।

## ऑनलाइन उपकरण

- **Wasm** को **Wat** में डिकंपाइल करने के लिए, [Wabt का wasm2wat डेमो](https://webassembly.github.io/wabt/demo/wasm2wat/index.html) उपयुक्त है।
- **Wat** को वापस **Wasm** में **कंपाइल** करने के लिए, [Wabt का wat2wasm डेमो](https://webassembly.github.io/wabt/demo/wat2wasm/) उपयुक्त है।
- एक और डिकंपाइलेशन विकल्प [web-wasmdec](https://wwwg.github.io/web-wasmdec/) पर उपलब्ध है।

## सॉफ्टवेयर समाधान

- एक और मजबूत समाधान के लिए, [JEB by PNF Software](https://www.pnfsoftware.com/jeb/demo) व्यापक सुविधाएं प्रदान करता है।
- ओपन-सोर्स परियोजना [wasmdec](https://github.com/wwwg/wasmdec) भी डिकंपाइलेशन कार्यों के लिए उपलब्ध है।

# .Net Decompilation Resources

.Net असेम्ब्लियों की डिकंपाइलेशन को उपकरणों जैसे कर सकते हैं:

- [ILSpy](https://github.com/icsharpcode/ILSpy), जो एक [Visual Studio Code के लिए प्लगइन](https://github.com/icsharpcode/ilspy-vscode) भी प्रदान करता है, जिससे क्रॉस-प्लेटफ़ॉर्म उपयोग संभव होता है।
- **डिकंपाइलेशन**, **संशोधन**, और **पुनर्संशोधन** से संबंधित कार्यों के लिए, [dnSpy](https://github.com/0xd4d/dnSpy/releases) की अत्यधिक सिफारिश की जाती है। **किसी विधि पर दायाँ क्लिक** करके **विधि संशोधित करने** की सुविधा प्राप्त होती है।
- [JetBrains' dotPeek](https://www.jetbrains.com/es-es/decompiler/) .Net असेम्ब्लियों की डिकंपाइलेशन के लिए एक और विकल्प है।

## DNSpy के साथ डीबगिंग और लॉगिंग को बढ़ावा देना

### DNSpy लॉगिंग
DNSpy का उपयोग करके फ़ाइल में जानकारी लॉग करने के लिए, निम्नलिखित .Net कोड स्निपेट को शामिल करें:

%%%cpp
using System.IO;
path = "C:\\inetpub\\temp\\MyTest2.txt";
File.AppendAllText(path, "Password: " + password + "\n");
%%%

### DNSpy डीबगिंग
DNSpy के साथ प्रभावी डीबगिंग के लिए, डीबगिंग के लिए **असेम्बली विशेषताएँ** को समायोजित करने के लिए एक क्रम की सिफारिश की जाती है, जिससे सुनिश्चित हो कि डीबगिंग को बाधित कर सकने वाले अनुकूलनों को अक्षम किया जाता है। इस प्रक्रिया में `DebuggableAttribute` सेटिंग्स को बदलना, असेम्बली को पुनः संकलित करना, और परिवर्तन सहेजना शामिल है।

इसके अतिरिक्त, **IIS** द्वारा चलाया गया एक .Net एप्लिकेशन को डीबग करने के लिए, `iisreset /noforce` का निषेध IIS को पुनः आरंभ करता है। DNSpy को IIS प्रक्रिया से जोड़ने के लिए डीबगिंग सत्र शुरू करने के लिए, मार्गदर्शिका निर्देश देती है कि DNSpy में **w3wp.exe** प्रक्रिया का चयन करना और डीबगिंग सत्र शुरू करना।

डीबगिंग के दौरान लोड किए गए मॉड्यूलों का व्यापक दृश्य प्राप्त करने के लिए, DNSpy में **मॉड्यूल** विंडो तक पहुंचने की सिफारिश की जाती है, जिसके बाद सभी मॉड्यूल खोलने और असेम्ब्लियों को सूचीबद्ध करने के लिए अनुशासन किया जाता है।

यह गाइड WebAssembly और .Net डिकंपाइलेशन की मूल बातों को संग्रहित करता है, डेवलपरों को इन कार्यों को आसानी से संभालने के लिए एक मार्ग प्रदान करता है।

## **Java Decompiler**
Java bytecode को डिकंपाइल करने के लिए ये उपकरण बहुत सहायक हो सकते हैं:
- [jadx](https://github.com/skylot/jadx)
- [JD-GUI](https://github.com/java-decompiler/jd-gui/releases)

## **DLLs का डीबगिंग**
### IDA का उपयोग
- 64-बिट और 32-बिट संस्करणों के लिए **Rundll32** विशेष पथों से लोड किया जाता है।
- **Windbg** को डीबगर के रूप में चयनित किया जाता है जिसमें पुस्तकालय लोड/अनलोड पर रोक लगाने की सुविधा होती है।
- निष्पादन पैरामीटर में DLL पथ और फ़ंक्शन नाम शामिल होते हैं। यह सेटअप प्रत्येक DLL के लोड होने पर निष्पादन को रोक देता है।

### x64dbg/x32dbg का उपयोग
- IDA के समान रूप से, **rundll32** को DLL और फ़ंक्शन को निर्दिष्ट करने के लिए कमांड लाइन संशोधन के साथ लोड किया जाता है।
- सेटिंग्स को बदलकर DLL प्रवेश पाइंट पर विचार ब्रेक करने की सुविधा प्रदान की जाती है।

### छवियाँ
- छवियों के माध्यम से निष्पादन रुकावट बिंदु और विन्यासों को वर्णित किया गया है।

## **ARM & MIPS**
- अनुकरण के लिए, [arm_now](https://github.com/nongiach/arm_now) एक उपयोगी संसाधन है।

## **शैलकोड्स**
### डीबगिंग तकनीकें
- **Blobrunner** और **jmp2it** मेमोरी में शैलकोड्स को आवंटित करने और उन्हें Ida या x64dbg के साथ डीबग करने के लिए उपकरण हैं।
- Blobrunner [रिलीज़](https://github.com/OALabs/BlobRunner/releases/tag/v0.0.5)
- jmp2it [कॉम्पाइल किया गया संस्करण](https://github.com/adamkramer/jmp2it/releases/)
- **Cutter** GUI आधारित शैलकोड अनुकरण और जांच प्रदान करता है, फ़ाइल के रूप में शैलकोड का हैंडलिंग में अंतरों को हाइलाइट करता है।

### डीओबफस्केशन और विश्लेषण
- **scdbg** शैलकोड कार्यों और डीओबफस्केशन क्षमताओं की दृष्टि से प्रकाश डालता है।
%%%bash
scdbg.exe -f shellcode # मूल्यांकन की आवश्यकता
scdbg.exe -f shellcode -r # विश्लेषण रिपोर्ट
scdbg.exe -f shellcode -i -r # इंटरैक
