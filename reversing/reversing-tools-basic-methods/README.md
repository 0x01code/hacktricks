# Reversing Tools & Basic Methods

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड** करने का उपयोग करना चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को।**

</details>

<figure><img src="/.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

वे संदेश जो महत्वपूर्ण हैं उन्हें खोजें ताकि आप उन्हें तेजी से ठीक कर सकें। Intruder आपकी हमले की सतह का ट्रैक करता है, प्रोएक्टिव धमकी स्कैन चलाता है, आपकी पूरी टेक स्टैक, API से वेब ऐप्स और क्लाउड सिस्टम तक, में समस्याओं को खोजता है। [**इसे नि: शुल्क परीक्षण के लिए प्रयास करें**](https://www.intruder.io/?utm\_source=referral\&utm\_campaign=hacktricks) आज।

{% embed url="https://www.intruder.io/?utm_campaign=hacktricks&utm_source=referral" %}

***

## ImGui आधारित Reversing उपकरण

सॉफ़्टवेयर:

* ReverseKit: [https://github.com/zer0condition/ReverseKit](https://github.com/zer0condition/ReverseKit)

## Wasm डिकंपाइलर / Wat कंपाइलर

ऑनलाइन:

* [https://webassembly.github.io/wabt/demo/wasm2wat/index.html](https://webassembly.github.io/wabt/demo/wasm2wat/index.html) का उपयोग करें वैसम (बाइनरी) से वैट (स्पष्ट पाठ) में **डिकंपाइल** करने के लिए
* [https://webassembly.github.io/wabt/demo/wat2wasm/](https://webassembly.github.io/wabt/demo/wat2wasm/) का उपयोग करें वैट से वैसम में **कंपाइल** करने के लिए
* आप [https://wwwg.github.io/web-wasmdec/](https://wwwg.github.io/web-wasmdec/) का उपयोग भी कर सकते हैं डिकंपाइल करने के लिए

सॉफ़्टवेयर:

* [https://www.pnfsoftware.com/jeb/demo](https://www.pnfsoftware.com/jeb/demo)
* [https://github.com/wwwg/wasmdec](https://github.com/wwwg/wasmdec)

## .Net डिकंपाइलर

### [dotPeek](https://www.jetbrains.com/decompiler/)

dotPeek एक डिकंपाइलर है जो **पुस्तकालय** (.dll), **Windows मेटाडेटा फ़ाइल** (.winmd) और **एक्ज़ीक्यूटेबल** (.exe) सहित कई प्रारूपों को **डिकंपाइल** और **जांचता** है। डिकंपाइल करने के बाद, एक असेंबली को एक विज़ुअल स्टूडियो प्रोजेक्ट (.csproj) के रूप में सहेजा जा सकता है।

यहां महत्व है कि यदि एक खोया हुआ स्रोत कोड एक पुराने असेंबली से पुनर्स्थापित करने की आवश्यकता होती है, तो यह कार्रवाई समय बचा सकती है। इसके अलावा, dotPeek डिकंपाइल कोड के माध्यम से हाथ में आदेश करने की सुविधा प्रदान करता है, जिससे यह **Xamarin एल्गोरिदम विश्लेषण** के लिए एक उत्कृष्ट उपकरण बनाता है।&#x20;

### [.Net Reflector](https://www.red-gate.com/products/reflector/)

एक व्यापक एड-इन मॉडल और एक API के साथ जो आपकी विशेष आवश्यकताओं को पूरा करने के लिए उपकरण का विस्तार करता है, .NET रिफ्लेक्टर समय बचाता है और विकास को सरल बनाता है। आइए इस उपकरण द्वारा प्रदान की जाने वाली पूरी पलटन के प्रतिष्ठित रिवर्स इंजीनियरिंग सेवाओं की ओर देखें:

* एक पुस्तकालय या कंपोनेंट में डेटा कैसे फ्लो होता है का अंदाज़ा देता है
* .NET भाषाओं और फ़्रेमवर्क के अनुरूपीकरण और उपयोग का अंदाज़ा देता है
* उपयोग किए जाने वाले एपीआई और प्रौद
```bash
using System.IO;
path = "C:\\inetpub\\temp\\MyTest2.txt";
File.AppendAllText(path, "Password: " + password + "\n");
```
### DNSpy डीबगिंग

DNSpy का उपयोग करके कोड की डीबगिंग करने के लिए आपको निम्नलिखित कार्रवाई करनी होगी:

पहले, **डीबगिंग** से संबंधित **असेंबली गुण** बदलें:

![](<../../.gitbook/assets/image (278).png>)
```aspnet
[assembly: Debuggable(DebuggableAttribute.DebuggingModes.IgnoreSymbolStoreSequencePoints)]
```
श्रीमान/श्रीमती,

निम्नलिखित है एक हैकिंग किताब से सामग्री जो हैकिंग तकनीकों के बारे में है। निम्नलिखित सामग्री /hive/hacktricks/reversing/reversing-tools-basic-methods/README.md फ़ाइल से है। अंग्रेजी में दिए गए पाठ का हिंदी में अनुवाद करें और अनुवाद को वही मार्कडाउन और HTML सिंटैक्स के साथ वापस लौटाएं। कोड, हैकिंग तकनीक नाम, हैकिंग शब्द, क्लाउड / SaaS प्लेटफ़ॉर्म नाम (जैसे Workspace, aws, gcp ...), शब्द 'leak', पेंटेस्टिंग और मार्कडाउन टैग्स जैसी चीजें अनुवाद न करें। इसके अलावा अतिरिक्त कुछ भी न जोड़ें, केवल अनुवाद और मार्कडाउन सिंटैक्स रखें।

धन्यवाद।

आपका वफादारी,
```
[assembly: Debuggable(DebuggableAttribute.DebuggingModes.Default |
DebuggableAttribute.DebuggingModes.DisableOptimizations |
DebuggableAttribute.DebuggingModes.IgnoreSymbolStoreSequencePoints |
DebuggableAttribute.DebuggingModes.EnableEditAndContinue)]
```
और **कंपाइल** पर क्लिक करें:

![](<../../.gitbook/assets/image (314) (1) (1).png>)

फिर नया फ़ाइल _**फ़ाइल >> मॉड्यूल सहेजें...**_ पर सहेजें:

![](<../../.gitbook/assets/image (279).png>)

यह आवश्यक है क्योंकि यदि आप ऐसा नहीं करते हैं, तो **टाइमर** पर कई **अनुकूलन** कोड पर लागू होंगे और यह संभव है कि डीबगिंग के दौरान कोई **ब्रेक-पॉइंट नहीं हिट होगा** या कुछ **वेरिएबल्स मौजूद नहीं होंगे**।

फिर, यदि आपका .Net एप्लिकेशन **IIS** द्वारा **चलाया** जा रहा है, तो आप इसे **रीस्टार्ट** कर सकते हैं:
```
iisreset /noforce
```
तो, डीबगिंग शुरू करने के लिए आपको सभी खोले गए फ़ाइलों को बंद करना चाहिए और **डीबग टैब** में जाएं और **प्रक्रिया में जुड़ें...** का चयन करें:

![](<../../.gitbook/assets/image (280).png>)

फिर **w3wp.exe** का चयन करें और **IIS सर्वर** से जुड़ने के लिए **जोड़ें** पर क्लिक करें:

![](<../../.gitbook/assets/image (281).png>)

अब जब हम प्रक्रिया को डीबग कर रहे हैं, तो इसे रोकने और सभी मॉड्यूल लोड करने का समय है। पहले _Debug >> Break All_ पर क्लिक करें और फिर _**Debug >> Windows >> Modules**_ पर क्लिक करें:

![](<../../.gitbook/assets/image (286).png>)

![](<../../.gitbook/assets/image (283).png>)

**Modules** पर किसी भी मॉड्यूल पर क्लिक करें और **Open All Modules** का चयन करें:

![](<../../.gitbook/assets/image (284).png>)

**Assembly Explorer** में किसी भी मॉड्यूल पर दायां क्लिक करें और **Sort Assemblies** पर क्लिक करें:

![](<../../.gitbook/assets/image (285).png>)

## जावा डिकंपाइलर

[https://github.com/skylot/jadx](https://github.com/skylot/jadx)\
[https://github.com/java-decompiler/jd-gui/releases](https://github.com/java-decompiler/jd-gui/releases)

## DLLs की डीबगिंग

### IDA का उपयोग करके

* **rundll32 लोड करें** (64 बिट C:\Windows\System32\rundll32.exe और 32 बिट C:\Windows\SysWOW64\rundll32.exe में)
* **Windbg** डीबगर का चयन करें
* "**Suspend on library load/unload**" का चयन करें

![](<../../.gitbook/assets/image (135).png>)

* निष्पादन के **पैरामीटर** को कॉन्फ़िगर करें और **DLL के पथ** और आपको कॉल करना चाहिए वाले फ़ंक्शन को डालें:

![](<../../.gitbook/assets/image (136).png>)

फिर, जब आप डीबगिंग शुरू करते हैं, **प्रत्येक DLL लोड होने पर निष्पादन रुक जाएगा**, फिर, जब rundll32 आपकी DLL लोड करेगा, निष्पादन रुक जाएगा।

लेकिन, कैसे आप लोड हुए DLL के कोड तक पहुंच सकते हैं? इस तरीके का उपयोग करके, मुझे नहीं पता है।

### x64dbg/x32dbg का उपयोग करके

* **rundll32 लोड करें** (64 बिट C:\Windows\System32\rundll32.exe और 32 बिट C:\Windows\SysWOW64\rundll32.exe में)
* **Command Line बदलें** (_File --> Change Command Line_) और dll के पथ और आपको कॉल करना चाहिए वाले फ़ंक्शन का सेट करें, उदाहरण के लिए: "C:\Windows\SysWOW64\rundll32.exe" "Z:\shared\Cybercamp\rev2\\\14.ridii\_2.dll",DLLMain
* _Options --> Settings_ बदलें और "**DLL Entry**" का चयन करें।
* फिर **निष्पादन शुरू करें**, डीबगर हर डीएलएल मेन में रुक जाएगा, किसी बिंदु पर आप **अपने DLL की डीबगिंग करना चाहेंगे**। वहां से, बस उन बिंदुओं की खोज करें जहां आप एक ब्रेकपॉइंट लगाना चाहते हैं।

ध्यान दें कि जब विन64डीबग में किसी कारण से निष्पादन रुक जाता है, तो आप देख सकते हैं कि आप किस कोड में हैं, विन64डीबग विंडो के शीर्ष में देखें:

![](<../../.gitbook/assets/image (137).png>)

फिर, इसे देखकर आप देख सकते हैं कि आप किस DLL में डीबगिंग करने के लिए रुके हुए हैं।

## GUI ऐप्स / वीडियोगेम्स

[**Cheat Engine**](https://www.cheatengine.org/downloads.php) एक उपयोगी प्रोग्राम है जो चल रहे खेल की मेमोरी में महत्वपूर्ण मानों को कहां सहेजे जाते हैं और उन्हें बदलता है। अधिक जानकारी के लिए:

{% content-ref url="cheat-engine.md" %}
[cheat-engine.md](cheat-engine.md)
{% endcontent-ref %}

## ARM और MIPS

{% embed url="https://github.com/nongiach/arm_now" %}

## शेलकोड

### ब्लॉबरनर के साथ शेलकोड की डीबगिंग

[**Blobrunner**](https://github.com/OALabs/BlobRunner) शेलकोड को मेमोरी के एक स्थान में आवंटित करेगा, आपको शेलकोड को आवंटित किए गए मेमोरी पते बताएगा और निष्पादन को रोक देगा।\
फिर, आपको प्रक्रिया में एक डीबगर (Ida या x64dbg) को जोड़ने की आवश्यकता होगी और निर्दिष्ट मेमोरी पते पर एक ब्रेकपॉइंट रखने और निष्पादन को फिर से शुरू करने की आवश्यकता होगी। इस तरह आप शेलकोड की डीबगिंग कर रहे होंगे।

विमुक्ति github पृष्ठ में कंपाइल किए गए रिलीज़ को ज़िप में शामिल किया गया है: [https://github.com/OALabs/BlobRunner/releases/tag/v0.0.5](https://
### शैलकोड को डीओबफस्केट करना और निष्पादित फंक्शन प्राप्त करना

आपको [**scdbg**](http://sandsprite.com/blogs/index.php?uid=7\&pid=152) का प्रयास करना चाहिए।\
यह आपको बताएगा कि शैलकोड कौन से **फंक्शन** का उपयोग कर रहा है और क्या शैलकोड मेमोरी में खुद को **डिकोड** कर रहा है।
```bash
scdbg.exe -f shellcode # Get info
scdbg.exe -f shellcode -r #show analysis report at end of run
scdbg.exe -f shellcode -i -r #enable interactive hooks (file and network) and show analysis report at end of run
scdbg.exe -f shellcode -d #Dump decoded shellcode
scdbg.exe -f shellcode /findsc #Find offset where starts
scdbg.exe -f shellcode /foff 0x0000004D #Start the executing in that offset
```
scDbg एक ग्राफिकल लॉन्चर के साथ भी आता है जहां आप विकल्पों का चयन कर सकते हैं और शेलकोड को निष्पादित कर सकते हैं

![](<../../.gitbook/assets/image (398).png>)

**डंप बनाएं** विकल्प शेलकोड में डाइनामिक रूप से किसी भी परिवर्तन को डंप करेगा (डिकोड किए गए शेलकोड को डाउनलोड करने के लिए उपयोगी होता है)। **स्टार्ट ऑफसेट** शेलकोड को एक विशिष्ट ऑफसेट पर शुरू करने के लिए उपयोगी हो सकता है। **डीबग शेल** विकल्प scDbg टर्मिनल का उपयोग करके शेलकोड को डीबग करने के लिए उपयोगी होता है (हालांकि, मैं इस मामले में पहले बताए गए किसी भी विकल्प का उपयोग करने के लिए Ida या x64dbg का उपयोग करने को बेहतर मानता हूं)।

### साइबरशेफ का उपयोग करके डिसअसेंबल करना

अपने शेलकोड फ़ाइल को इनपुट के रूप में अपलोड करें और इसे डिकॉम्पाइल करने के लिए निम्नलिखित रसीपी का उपयोग करें: [https://gchq.github.io/CyberChef/#recipe=To\_Hex('Space',0)Disassemble\_x86('32','Full%20x86%20architecture',16,0,true,true)](https://gchq.github.io/CyberChef/#recipe=To\_Hex\('Space',0\)Disassemble\_x86\('32','Full%20x86%20architecture',16,0,true,true\))

## [Movfuscator](https://github.com/xoreaxeaxeax/movfuscator)

यह अवशोषक **`mov` के लिए सभी निर्देशों को संशोधित करता है** (हाँ, वास्तव में शानदार)। यह निर्देशों को बदलने के लिए अविरामताएं भी उपयोग करता है। इसके बारे में अधिक जानकारी के लिए:

* [https://www.youtube.com/watch?v=2VF\_wPkiBJY](https://www.youtube.com/watch?v=2VF\_wPkiBJY)
* [https://github.com/xoreaxeaxeax/movfuscator/blob/master/slides/domas\_2015\_the\_movfuscator.pdf](https://github.com/xoreaxeaxeax/movfuscator/blob/master/slides/domas\_2015\_the\_movfuscator.pdf)

यदि आप भाग्यशाली हैं तो [demovfuscator ](https://github.com/kirschju/demovfuscator)बाइनरी को डीऑफस्केट करेगा। इसमें कई आवश्यकताएं होती हैं।
```
apt-get install libcapstone-dev
apt-get install libz3-dev
```
और [keystone को इंस्टॉल करें](https://github.com/keystone-engine/keystone/blob/master/docs/COMPILE-NIX.md) (`apt-get install cmake; mkdir build; cd build; ../make-share.sh; make install`)

यदि आप **CTF खेल रहे हैं, तो यह फ्लैग ढूंढ़ने के लिए उपाय** बहुत उपयोगी हो सकता है: [https://dustri.org/b/defeating-the-recons-movfuscator-crackme.html](https://dustri.org/b/defeating-the-recons-movfuscator-crackme.html)


<figure><img src="/.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

विशेषता को जल्दी ठीक करने के लिए महत्वपूर्ण संरचना की खोज करें। Intruder आपकी हमले की सतह का ट्रैक करता है, प्रोएक्टिव धारणा स्कैन चलाता है, आपकी पूरी टेक स्टैक, एपीआई से वेब ऐप्स और क्लाउड सिस्टम तक, में समस्याएं खोजता है। [**इसे आजमाएं मुफ्त में**](https://www.intruder.io/?utm\_source=referral\&utm\_campaign=hacktricks)।

{% embed url="https://www.intruder.io/?utm_campaign=hacktricks&utm_source=referral" %}

***

## Rust

**एंट्री प्वाइंट** ढूंढ़ने के लिए `::main` के रूप में फंक्शन्स की खोज करें जैसे:

![](<../../.gitbook/assets/image (612).png>)

इस मामले में बाइनरी को authenticator नाम दिया गया था, इसलिए यह काफी स्पष्ट है कि यह दिलचस्प मुख्य फ़ंक्शन है।\
जब फ़ंक्शनों के **नाम** को बुलाया जाता है, तो उन्हें इंटरनेट पर खोजें और उनके **इनपुट** और **आउटपुट** के बारे में जानें।

## **Delphi**

Delphi कंपाइल किए गए बाइनरी के लिए आप [https://github.com/crypto2011/IDR](https://github.com/crypto2011/IDR) का उपयोग कर सकते हैं

यदि आपको एक Delphi बाइनरी को रिवर्स करना है, तो मैं आपको IDA प्लगइन [https://github.com/Coldzer0/IDA-For-Delphi](https://github.com/Coldzer0/IDA-For-Delphi) का उपयोग करने की सलाह दूंगा

बस **ATL+f7** दबाएं (IDA में पायथन प्लगइन आयात करें) और पायथन प्लगइन का चयन करें।

यह प्लगइन बाइनरी को निष्पादित करेगा और डीबगिंग की शुरुआत में फ़ंक्शन नामों को गतिशील रूप से हल करेगा। डीबगिंग शुरू करने के बाद फिर से स्टार्ट बटन (हरी वाला या f9) दबाएं और शुरुआती कोड के शुरुआत में एक ब्रेकपॉइंट हिट होगा।

यह बहुत रोचक भी है क्योंकि यदि आप ग्राफिक एप्लिकेशन में एक बटन दबाते हैं तो डीबगर उस बटन द्वारा निष्पादित फ़ंक्शन में रुक जाएगा।

## Golang

यदि आपको एक Golang बाइनरी को रिवर्स करना है, तो मैं आपको IDA प्लगइन [https://github.com/sibears/IDAGolangHelper](https://github.com/sibears/IDAGolangHelper) का उपयोग करने की सलाह दूंगा

बस **ATL+f7** दबाएं (IDA में पायथन प्लगइन आयात करें) और पायथन प्लगइन का चयन करें।

इससे फ़ंक्शनों के नाम हल हो जाएंगे।

## Compiled Python

इस पेज में आप एक ELF/EXE पायथन कंपाइल किए गए बाइनरी से पायथन कोड कैसे प्राप्त कर सकते हैं, उसकी जानकारी मिलेगी:

{% content-ref url="../../forensics/basic-forensic-methodology/specific-software-file-type-tricks/.pyc.md" %}
[.pyc.md](../../forensics/basic-forensic-methodology/specific-software-file-type-tricks/.pyc.md)
{% endcontent-ref %}

## GBA - Game Body Advance

यदि आपको GBA गेम का **बाइनरी** मिलता है, तो आप इसे **इम्युलेट** और **डीबग** करने के लिए विभिन्न उपकरणों का उपयोग कर सकते हैं:

* [**no$gba**](https://problemkaputt.de/gba.htm) (_डीबग संस्करण डाउनलोड करें_) - इंटरफ़ेस के साथ एक डीबगर शामिल है
* [**mgba** ](https://mgba.io)- CLI डीबगर शामिल है
* [**gba-ghidra-loader**](https://github.com/pudii/gba-ghidra-loader) - Ghidra प्लगइन
* [**GhidraGBA**](https://github.com/SiD3W4y/GhidraGBA) - Ghidra प्लगइन

[**no$gba**](https://problemkaputt.de/gba.htm) में, _**Options --> Emulation Setup --> Controls**_\*\* \*\* में देखें कि कैसे Game Boy Advance **बटन** दबाएं

![](<../../.gitbook/assets/image (578).png>)

दबाए जाने पर, प्रत्येक **कुंजी का मान** उसे पहचानने के लिए होता है:
```
A = 1
B = 2
SELECT = 4
START = 8
RIGHT = 16
LEFT = 32
UP = 64
DOWN = 128
R = 256
L = 256
```
तो, इस प्रकार के प्रोग्राम में, एक रोचक हिस्सा होगा **प्रोग्राम उपयोगकर्ता इनपुट को कैसे व्यवहार करता है**। पते **0x4000130** में आपको आमतौर पर पाए जाने वाले फ़ंक्शन को पाएंगे: **KEYINPUT**।

![](<../../.gitbook/assets/image (579).png>)

पिछली छवि में आप देख सकते हैं कि फ़ंक्शन को **FUN\_080015a8** (पते: _0x080015fa_ और _0x080017ac_) से बुलाया जाता है।

उस फ़ंक्शन में, कुछ प्रारंभिक संचालनों के बाद (जिनका कोई महत्व नहीं है):
```c
void FUN_080015a8(void)

{
ushort uVar1;
undefined4 uVar2;
undefined4 uVar3;
ushort uVar4;
int iVar5;
ushort *puVar6;
undefined *local_2c;

DISPCNT = 0x1140;
FUN_08000a74();
FUN_08000ce4(1);
DISPCNT = 0x404;
FUN_08000dd0(&DAT_02009584,0x6000000,&DAT_030000dc);
FUN_08000354(&DAT_030000dc,0x3c);
uVar4 = DAT_030004d8;
```
यह कोड मिला है:
```c
do {
DAT_030004da = uVar4; //This is the last key pressed
DAT_030004d8 = KEYINPUT | 0xfc00;
puVar6 = &DAT_0200b03c;
uVar4 = DAT_030004d8;
do {
uVar2 = DAT_030004dc;
uVar1 = *puVar6;
if ((uVar1 & DAT_030004da & ~uVar4) != 0) {
```
अंतिम if देख रहा है कि **`uVar4`** **अंतिम कुंजी** में है और यह मौजूदा कुंजी नहीं है, जिसे वह एक बटन छोड़ रहा है (वर्तमान कुंजी **`uVar1`** में संग्रहीत है)।
```c
if (uVar1 == 4) {
DAT_030000d4 = 0;
uVar3 = FUN_08001c24(DAT_030004dc);
FUN_08001868(uVar2,0,uVar3);
DAT_05000000 = 0x1483;
FUN_08001844(&DAT_0200ba18);
FUN_08001844(&DAT_0200ba20,&DAT_0200ba40);
DAT_030000d8 = 0;
uVar4 = DAT_030004d8;
}
else {
if (uVar1 == 8) {
if (DAT_030000d8 == 0xf3) {
DISPCNT = 0x404;
FUN_08000dd0(&DAT_02008aac,0x6000000,&DAT_030000dc);
FUN_08000354(&DAT_030000dc,0x3c);
uVar4 = DAT_030004d8;
}
}
else {
if (DAT_030000d4 < 8) {
DAT_030000d4 = DAT_030000d4 + 1;
FUN_08000864();
if (uVar1 == 0x10) {
DAT_030000d8 = DAT_030000d8 + 0x3a;
```
पिछले कोड में आप देख सकते हैं कि हम **uVar1** (जहां **दबाए गए बटन का मान** होता है) को कुछ मानों के साथ तुलना कर रहे हैं:

* पहले, इसे **मान 4** (**SELECT** बटन) के साथ तुलना की जाती है: चैलेंज में यह बटन स्क्रीन को साफ करता है
* फिर, इसे **मान 8** (**START** बटन) के साथ तुलना की जाती है: चैलेंज में यह जांचता है कि क्या कोड फ़्लैग प्राप्त करने के लिए मान्य है।
* इस मामले में वैर **`DAT_030000d8`** को 0xf3 के साथ तुलना की जाती है और यदि मान समान है तो कुछ कोड चलाया जाता है।
* किसी अन्य मामलों में, कुछ cont (`DAT_030000d4`) की जांच की जाती है। यह एक cont है क्योंकि इसे कोड में प्रवेश करने के बाद एक के बाद 1 जोड़ा जाता है।\
यदि 8 से कम है तो कुछ ऐसा कुछ होता है जो **`DAT_030000d8`** में मानों को जोड़ने से संबंधित है (मूल रूप से यह मानों को इस चर में जोड़ता है जब तक cont 8 से कम होता है)।

तो, इस चुनौती में, बटनों के मानों को जानते हुए, आपको **8 से छोटी लंबाई की एक कॉम्बिनेशन दबानी थी जिसका परिणामस्वरूप जोड़ 0xf3 होता है।**

**इस ट्यूटोरियल के लिए संदर्भ:** [**https://exp.codes/Nostalgia/**](https://exp.codes/Nostalgia/)

## गेम बॉय

{% embed url="https://www.youtube.com/watch?v=VVbRe7wr3G4" %}

## कोर्सेज

* [https://github.com/0xZ0F/Z0FCourse\_ReverseEngineering](https://github.com/0xZ0F/Z0FCourse\_ReverseEngineering)
* [https://github.com/malrev/ABD](https://github.com/malrev/ABD) (बाइनरी डिओबफस्केशन)


<figure><img src="/.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

वे संगठनों को खोजें जिनका महत्वपूर्ण विकर्णता है ताकि आप उन्हें तेजी से ठीक कर सकें। इंट्रूडर आपकी हमला सतह का ट्रैक करता है, प्रोएक्टिव धमकी स्कैन चलाता है, एपीआई से वेब ऐप्स और क्लाउड सिस्टम तक आपकी पूरी टेक स्टैक में मुद्दों को खोजता है। [**इसे नि: शुल्क परीक्षण के लिए आजमाएं**](https://www.intruder.io/?utm\_source=referral\&utm\_campaign=hacktricks)।

{% embed url="https://www.intruder.io/?utm_campaign=hacktricks&utm_source=referral" %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की अनुमति चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह।
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को।**

</details>
