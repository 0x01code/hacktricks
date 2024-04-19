# Reversing Tools & Basic Methods

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert) के साथ</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके।

</details>

**Try Hard Security Group**

<figure><img src="../../.gitbook/assets/telegram-cloud-document-1-5159108904864449420.jpg" alt=""><figcaption></figcaption></figure>

{% embed url="https://discord.gg/tryhardsecurity" %}

***

## ImGui आधारित Reversing टूल्स

सॉफ़्टवेयर:

* ReverseKit: [https://github.com/zer0condition/ReverseKit](https://github.com/zer0condition/ReverseKit)

## Wasm डिकंपाइलर / Wat कंपाइलर

ऑनलाइन:

* [https://webassembly.github.io/wabt/demo/wasm2wat/index.html](https://webassembly.github.io/wabt/demo/wasm2wat/index.html) का उपयोग करें **डिकंपाइल** करने के लिए wasm (बाइनरी) से wat (स्पष्ट पाठ) तक
* [https://webassembly.github.io/wabt/demo/wat2wasm/](https://webassembly.github.io/wabt/demo/wat2wasm/) का उपयोग करें **कंपाइल** करने के लिए wat से wasm तक
* आप [https://wwwg.github.io/web-wasmdec/](https://wwwg.github.io/web-wasmdec/) का उपयोग भी कर सकते हैं डिकंपाइल करने के लिए

सॉफ़्टवेयर:

* [https://www.pnfsoftware.com/jeb/demo](https://www.pnfsoftware.com/jeb/demo)
* [https://github.com/wwwg/wasmdec](https://github.com/wwwg/wasmdec)

## .NET डिकंपाइलर

### [dotPeek](https://www.jetbrains.com/decompiler/)

dotPeek एक डिकंपाइलर है जो **लाइब्रेरी** (.dll), **Windows मेटाडेटा फ़ाइल**s (.winmd), और **एक्जीक्यूटेबल्स** (.exe) सहित कई प्रारूपों को **डिकंपाइल** और जांचता है। एक बार डिकंपाइल कर दिया जाए, एक असेम्बली को एक विजुअल स्टूडियो प्रोजेक्ट (.csproj) के रूप में सहेजा जा सकता है।

यहाँ महत्व है कि यदि किसी खो गए स्रोत कोड की पुनर्स्थापना एक पुराने असेम्बली से की जाती है, तो यह कार्रवाई समय बचा सकती है। इसके अतिरिक्त, dotPeek डिकंपाइल कोड के माध्यम से हाथ में उपयुक्त नेविगेशन प्रदान करता है, जिससे यह **Xamarin एल्गोरिदम विश्लेषण के लिए सबसे उत्तम उपकरणों में से एक बन जाता है।**

### [.NET Reflector](https://www.red-gate.com/products/reflector/)

एक व्यापक एड-इन मॉडल और एपीआई के साथ जो आपकी विशेष आवश्यकताओं को पूरा करने के लिए उपकरण को विस्तारित करता है, .NET रिफ्लेक्टर समय बचाता है और विकास को सरल बनाता है। चलिए इस उपकरण द्वारा प्रदान की जाने वाली पूरी पलटाव की सेवाओं की ओर एक नज़र डालें:

* एक पुस्तकालय या कॉम्पोनेंट के माध्यम से डेटा कैसे फ्लो करता है, इसका अंदाजा लगाता है
* .NET भाषाओं और फ़्रेमवर्क के अंमलन और उपयोग का अंदाजा लगाता है
* उपयोग किए जाने वाले एपीआई और प्रौद्योगिकियों से अधिक प्राप्त करने के लिए अनदोक्यूमेंटेड और अनएक्सपोज़्ड फ़ंक्शनैलिटी प्रदान करता है।
* डिपेंडेंसीज़ और विभिन्न असेम्बलियों को खोजता है
* आपके कोड, थर्ड-पार्टी कॉम्पोनेंट्स, और पुस्तकालयों में त्रुटियों के सटीक स्थान का पता लगाता है।
* आपके साथ काम करने वाले सभी .NET कोड के स्रोत में डीबग करता है।

### [ILSpy](https://github.com/icsharpcode/ILSpy) & [dnSpy](https://github.com/dnSpy/dnSpy/releases)

[ILSpy plugin for Visual Studio Code](https://github.com/icsharpcode/ilspy-vscode): आप इसे किसी भी ओएस में रख सकते हैं (आप इसे वीएसकोड से सीधे इंस्टॉल कर सकते हैं, गिट को डाउनलोड करने की आवश्यकता नहीं है। **एक्सटेंशन्स** पर क्लिक करें और **ILSpy** खोजें)।\
यदि आपको **डिकंपाइल**, **संशोधित** और फिर से **कंपाइल** करने की आवश्यकता है तो आप [**dnSpy**](https://github.com/dnSpy/dnSpy/releases) या इसके सक्रिय रूप से रखरखावित फोर्क, [**dnSpyEx**](https://github.com/dnSpyEx/dnSpy/releases) का उपयोग कर सकते हैं। (**राइट क्लिक -> फ़ंक्शन के भीतर कुछ बदलने के लिए मॉडिफाई मेथड**).

### DNSpy Logging

किसी **फ़ाइल में कुछ जानकारी लॉग करने के लिए DNSpy का उपयोग करने के लिए आप इस स्निपेट का उपयोग कर सकते हैं:
```cs
using System.IO;
path = "C:\\inetpub\\temp\\MyTest2.txt";
File.AppendAllText(path, "Password: " + password + "\n");
```
### DNSpy डीबगिंग

कोड को DNSpy का उपयोग करके डीबग करने के लिए आपको करना होगा:

पहले, **डीबगिंग** से संबंधित **असेंबली गुण** बदलें:

![](<../../.gitbook/assets/image (970).png>)
```aspnet
[assembly: Debuggable(DebuggableAttribute.DebuggingModes.IgnoreSymbolStoreSequencePoints)]
```
## बुनियादी उपकरण और विधियाँ

यह विषय रिवर्सिंग के बुनियादी उपकरण और विधियों पर है।
```
[assembly: Debuggable(DebuggableAttribute.DebuggingModes.Default |
DebuggableAttribute.DebuggingModes.DisableOptimizations |
DebuggableAttribute.DebuggingModes.IgnoreSymbolStoreSequencePoints |
DebuggableAttribute.DebuggingModes.EnableEditAndContinue)]
```
और **कंपाइल** पर क्लिक करें:

![](<../../.gitbook/assets/image (314) (1).png>)

फिर नया फ़ाइल _**File >> Save module...**_ के माध्यम से सहेजें:

![](<../../.gitbook/assets/image (599).png>)

यह आवश्यक है क्योंकि यदि आप ऐसा नहीं करते हैं, तो **रनटाइम** पर कई **ऑप्टिमाइजेशन** को कोड पर लागू किया जाएगा और यह संभावना है कि जब आप **डीबगिंग** कर रहे हों, तो **ब्रेक-पॉइंट कभी हिट नहीं होता** या कुछ **वेरिएबल्स मौजूद नहीं होते** हों।

फिर, यदि आपका .NET एप्लिकेशन **IIS** द्वारा **चलाया** जा रहा है तो आप इसे **रीस्टार्ट** कर सकते हैं:
```
iisreset /noforce
```
फिर, डीबगिंग शुरू करने के लिए आपको सभी खुली फ़ाइलें बंद करनी चाहिए और **डीबग टैब** में जाकर **प्रक्रिया में जुड़ने के लिए चुनें**:

![](<../../.gitbook/assets/image (315).png>)

फिर **w3wp.exe** को चुनें ताकि **IIS सर्वर** से जुड़ सके और **जुड़ें** पर क्लिक करें:

![](<../../.gitbook/assets/image (110).png>)

अब जब हम प्रक्रिया को डीबग कर रहे हैं, तो समय है कि इसे रोकें और सभी मॉड्यूल लोड करें। पहले _Debug >> Break All_ पर क्लिक करें और फिर _**Debug >> Windows >> Modules**_ पर क्लिक करें:

![](<../../.gitbook/assets/image (129).png>)

![](<../../.gitbook/assets/image (831).png>)

**मॉड्यूल** पर किसी भी मॉड्यूल पर क्लिक करें और **सभी मॉड्यूल खोलें** का चयन करें:

![](<../../.gitbook/assets/image (919).png>)

**असेम्बली एक्सप्लोरर** में किसी भी मॉड्यूल पर दायाँ क्लिक करें और **असेम्ब्लीज को क्रमित करें** पर क्लिक करें:

![](<../../.gitbook/assets/image (336).png>)

## जावा डीकंपाइलर

[https://github.com/skylot/jadx](https://github.com/skylot/jadx)\
[https://github.com/java-decompiler/jd-gui/releases](https://github.com/java-decompiler/jd-gui/releases)

## DLLs की डीबगिंग

### IDA का उपयोग

* **rundll32 लोड करें** (64 बिट C:\Windows\System32\rundll32.exe और 32 बिट C:\Windows\SysWOW64\rundll32.exe में)
* **Windbg** डीबगर का चयन करें
* "**लाइब्रेरी लोड/अनलोड पर सस्पेंड करें**" का चयन करें

![](<../../.gitbook/assets/image (865).png>)

* निष्पादन के **पैरामीटर** को कॉन्फ़िगर करें और **DLL के पथ** और वह फ़ंक्शन डालें जिसे आप कॉल करना चाहते हैं:

![](<../../.gitbook/assets/image (701).png>)

फिर, जब आप डीबगिंग शुरू करते हैं, **हर DLL लोड होने पर निष्पादन रुक जाएगा**, फिर, जब rundll32 आपके DLL को लोड करेगा, निष्पादन रुक जाएगा।

लेकिन, जिस DLL को लोड किया गया था, उसके कोड तक कैसे पहुंच सकते हैं? इस तरीके का उपयोग करके, मुझे नहीं पता कि यह कैसे किया जाए।

### x64dbg/x32dbg का उपयोग

* **rundll32 लोड करें** (64 बिट C:\Windows\System32\rundll32.exe और 32 बिट C:\Windows\SysWOW64\rundll32.exe में)
* **कमांड लाइन बदलें** ( _File --> Change Command Line_ ) और dll का पथ और वह फ़ंक्शन सेट करें जिसे आप कॉल करना चाहते हैं, उदाहरण के लिए: "C:\Windows\SysWOW64\rundll32.exe" "Z:\shared\Cybercamp\rev2\\\14.ridii\_2.dll",DLLMain
* _Options --> Settings_ बदलें और "**DLL Entry**" का चयन करें।
* फिर **निष्पादन शुरू करें**, डीबगर हर dll मेन में रुकेगा, किसी समय आपको **अपने dll के dll Entry में रुकना होगा**। वहां से, जहां आप ब्रेकपॉइंट लगाना चाहते हैं, वहां खोजें।

ध्यान दें कि जब विन64dbg में किसी कारण से निष्पादन होता है, तो आप देख सकते हैं कि **आप किस कोड में हैं** विन64dbg विंडो के **शीर्ष में** देख सकते हैं:

![](<../../.gitbook/assets/image (839).png>)

फिर, जब आप उस dll में निष्पादन हुआ है जिसे आप डीबग करना चाहते हैं, तो इसे देख सकते हैं।
```bash
scdbg.exe -f shellcode # Get info
scdbg.exe -f shellcode -r #show analysis report at end of run
scdbg.exe -f shellcode -i -r #enable interactive hooks (file and network) and show analysis report at end of run
scdbg.exe -f shellcode -d #Dump decoded shellcode
scdbg.exe -f shellcode /findsc #Find offset where starts
scdbg.exe -f shellcode /foff 0x0000004D #Start the executing in that offset
```
scDbg के पास एक ग्राफिकल लॉन्चर भी है जहाँ आप चुन सकते हैं जो विकल्प आप चाहते हैं और शेलकोड को निष्पादित कर सकते हैं

![](<../../.gitbook/assets/image (255).png>)

**डंप बनाएं** विकल्प अगर किसी भी परिवर्तन किया जाता है शेलकोड को डायनामिक रूप से मेमोरी में (डिकोड शेलकोड डाउनलोड करने के लिए उपयोगी) तो अंतिम शेलकोड को डंप करेगा। **स्टार्ट ऑफसेट** एक विशेष ऑफसेट पर शेलकोड को शुरू करने के लिए उपयोगी हो सकता है। **डीबग शेल** विकल्प शेलकोड को scDbg टर्मिनल का उपयोग करके डीबग करने के लिए उपयोगी है (हालांकि मैं सोचता हूँ कि इस विषय में पहले विवरणित किए गए किसी भी विकल्प का उपयोग बेहतर होगा क्योंकि आप Ida या x64dbg का उपयोग कर सकेंगे)।

### CyberChef का उपयोग करके डिसएसेम्बलिंग

अपनी शेलकोड फ़ाइल को इनपुट के रूप में अपलोड करें और इसे डिकॉम्पाइल करने के लिए निम्नलिखित रेसिपी का उपयोग करें: [https://gchq.github.io/CyberChef/#recipe=To\_Hex('Space',0)Disassemble\_x86('32','Full%20x86%20architecture',16,0,true,true)](https://gchq.github.io/CyberChef/#recipe=To\_Hex\('Space',0\)Disassemble\_x86\('32','Full%20x86%20architecture',16,0,true,true\))

## [Movfuscator](https://github.com/xoreaxeaxeax/movfuscator)

यह ऑबफस्केटर **सभी निर्देशिकाओं को `mov` के लिए संशोधित करता है** (हाँ, सचमुच कूल)। यह भी निर्वाहन फ्लो को बदलने के लिए बाधाएँ उपयोग करता है। इसके काम करने के बारे में अधिक जानकारी के लिए:

* [https://www.youtube.com/watch?v=2VF\_wPkiBJY](https://www.youtube.com/watch?v=2VF\_wPkiBJY)
* [https://github.com/xoreaxeaxeax/movfuscator/blob/master/slides/domas\_2015\_the\_movfuscator.pdf](https://github.com/xoreaxeaxeax/movfuscator/blob/master/slides/domas\_2015\_the\_movfuscator.pdf)

यदि आप भाग्यशाली हैं तो [demovfuscator](https://github.com/kirschju/demovfuscator) बाइनरी को डीऑफस्केट करेगा। इसमें कई आवश्यकताएँ हैं
```
apt-get install libcapstone-dev
apt-get install libz3-dev
```
और [कीस्टोन इंस्टॉल करें](https://github.com/keystone-engine/keystone/blob/master/docs/COMPILE-NIX.md) (`apt-get install cmake; mkdir build; cd build; ../make-share.sh; make install`)

यदि आप **CTF खेल रहे हैं, तो यह उपाय फ्लैग पाने के लिए** बहुत उपयोगी हो सकता है: [https://dustri.org/b/defeating-the-recons-movfuscator-crackme.html](https://dustri.org/b/defeating-the-recons-movfuscator-crackme.html)

## Rust

**एंट्री पॉइंट** खोजने के लिए `::main` जैसे फ़ंक्शन्स खोजें:

![](<../../.gitbook/assets/image (1077).png>)

इस मामले में बाइनरी को एथेंटिकेटर कहा गया था, इसलिए यह काफी स्पष्ट है कि यह दिलचस्प मुख्य फ़ंक्शन है।\
**फ़ंक्शन्स** के **नाम** के पास होने पर, उन्हें **इंटरनेट** पर खोजें ताकि उनके **इनपुट** और **आउटपुट** के बारे में जान सकें।

## **Delphi**

Delphi संकलित बाइनरी के लिए आप [https://github.com/crypto2011/IDR](https://github.com/crypto2011/IDR) का उपयोग कर सकते हैं।

यदि आपको एक Delphi बाइनरी को रिवर्स करना है तो मैं आपको यह सुझाव दूंगा कि आप IDA प्लगइन [https://github.com/Coldzer0/IDA-For-Delphi](https://github.com/Coldzer0/IDA-For-Delphi) का उपयोग करें।

बस **ATL+f7** दबाएं (IDA में पायथन प्लगइन आयात करें) और पायथन प्लगइन का चयन करें।

यह प्लगइन बाइनरी को निष्पादित करेगा और डीबगिंग की शुरुआत में फ़ंक्शन नामों को गतिशील रूप से हल करेगा। डीबगिंग शुरू करने के बाद फिर से स्टार्ट बटन (हरा वाला या f9) दबाएं और एक ब्रेकपॉइंट वास्तविक कोड की शुरुआत में हिट करेगा।

यह भी बहुत दिलचस्प है क्योंकि यदि आप ग्राफ़िक एप्लिकेशन में एक बटन दबाते हैं तो डीबगर उस बटन द्वारा निष्पादित फ़ंक्शन में रुक जाएगा।

## Golang

यदि आपको एक Golang बाइनरी को रिवर्स करना है तो मैं आपको IDA प्लगइन [https://github.com/sibears/IDAGolangHelper](https://github.com/sibears/IDAGolangHelper) का उपयोग करने की सलाह दूंगा।

बस **ATL+f7** दबाएं (IDA में पायथन प्लगइन आयात करें) और पायथन प्लगइन का चयन करें।

इससे फ़ंक्शनों के नाम हल हो जाएंगे।

## संकलित Python

इस पृष्ठ पर आप एक ELF/EXE Python संकलित बाइनरी से पायथन कोड कैसे प्राप्त करें इसे देख सकते हैं:

{% content-ref url="../../generic-methodologies-and-resources/basic-forensic-methodology/specific-software-file-type-tricks/.pyc.md" %}
[.pyc.md](../../generic-methodologies-and-resources/basic-forensic-methodology/specific-software-file-type-tricks/.pyc.md)
{% endcontent-ref %}

## GBA - गेम बॉडी एडवांस

यदि आपके पास एक GBA गेम का **बाइनरी** है तो आप उसे **इम्युलेट** और **डीबग** करने के लिए विभिन्न उपकरणों का उपयोग कर सकते हैं:

* [**no$gba**](https://problemkaputt.de/gba.htm) (_डीबग संस्करण डाउनलोड करें_) - एक डीबगर सहित
* [**mgba** ](https://mgba.io)- एक CLI डीबगर सहित
* [**gba-ghidra-loader**](https://github.com/pudii/gba-ghidra-loader) - गिडरा प्लगइन
* [**GhidraGBA**](https://github.com/SiD3W4y/GhidraGBA) - गिडरा प्लगइन

[**no$gba**](https://problemkaputt.de/gba.htm) में _**विकल्प --> इम्युलेशन सेटअप --> नियंत्रण**_\*\* \*\* आप देख सकते हैं कि कैसे गेम बॉय एडवांस **बटन्स** दबाए जाते हैं

![](<../../.gitbook/assets/image (578).png>)

जब दबाया जाता है, प्रत्येक **कुंजी का मूल्य** उसे पहचानने के लिए होता है:
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
इस प्रकार के कार्यक्रम में, दिलचस्प हिस्सा **कार्यक्रम उपयोगकर्ता इनपुट को कैसे व्यवहार करता है** पर होगा। पते **0x4000130** पर आपको आमतौर पर पाए जाने वाले फ़ंक्शन मिलेगा: **KEYINPUT**।

![](<../../.gitbook/assets/image (444).png>)

पिछली छवि में आप देख सकते हैं कि फ़ंक्शन **FUN\_080015a8** से कॉल किया जाता है (पते: _0x080015fa_ और _0x080017ac_)।

उस फ़ंक्शन में, कुछ आरंभिक ऑपरेशन के बाद (किसी भी महत्व के बिना):
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
आखिरी if **`uVar4`** की जांच कर रहा है कि क्या वह **आखिरी कुंजी** में है और वर्तमान कुंजी में नहीं है, जिसे छोड़ दिया जाता है (वर्तमान कुंजी **`uVar1`** में संग्रहीत है)।
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
पिछले कोड में आप देख सकते हैं कि हम **uVar1** (जहां **दबाए गए बटन का मान** है) को कुछ मानों के साथ तुलना कर रहे हैं:

* पहले, इसे **मान 4** (**SELECT** बटन) के साथ तुलना की जा रही है: इस चैलेंज में यह बटन स्क्रीन को साफ करता है
* फिर, इसे **मान 8** (**START** बटन) के साथ तुलना की जा रही है: इस चैलेंज में यह जांचता है कि क्या कोड झंडा प्राप्त करने के लिए वैध है।
* इस मामले में वैर **`DAT_030000d8`** को 0xf3 के साथ तुलना की जा रही है और अगर मान समान है तो कुछ कोड कार्यान्वित होता है।
* किसी भी अन्य मामले में, कुछ cont (`DAT_030000d4`) की जांच की जाती है। यह एक cont है क्योंकि इसे कोड में प्रवेश करने के बाद 1 जोड़ दिया जाता है।\
अगर 8 से कम कुछ ऐसा है जिसमें **मानों को जोड़ना** शामिल है \*\*`DAT_030000d8` \*\* (मुख्य रूप से इस चरण में दबाए गए कुंजियों के मानों को इस चरण में जोड़ता है जब तक cont 8 से कम है)।

तो, इस चैलेंज में, बटनों के मानों को जानकर, आपको **8 से छोटी लंबाई वाली एक संयोजन दबाना था जिसका परिणामस्वरूप जोड़ 0xf3 हो।**

**इस ट्यूटोरियल के लिए संदर्भ:** [**https://exp.codes/Nostalgia/**](https://exp.codes/Nostalgia/)

## गेम बॉय

{% embed url="https://www.youtube.com/watch?v=VVbRe7wr3G4" %}

## कोर्सेस

* [https://github.com/0xZ0F/Z0FCourse\_ReverseEngineering](https://github.com/0xZ0F/Z0FCourse\_ReverseEngineering)
* [https://github.com/malrev/ABD](https://github.com/malrev/ABD) (बाइनरी डीओबफस्केशन)

**ट्राई हार्ड सिक्योरिटी ग्रुप**

<figure><img src="../../.gitbook/assets/telegram-cloud-document-1-5159108904864449420.jpg" alt=""><figcaption></figcaption></figure>

{% embed url="https://discord.gg/tryhardsecurity" %}

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert) के साथ शून्य से हीरो तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

हैकट्रिक्स का समर्थन करने के अन्य तरीके:

* यदि आप अपनी कंपनी का विज्ञापन हैकट्रिक्स में देखना चाहते हैं या हैकट्रिक्स को पीडीएफ में डाउनलोड करना चाहते हैं तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक पीईएएस और हैकट्रिक्स स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**एनएफटी**](https://opensea.io/collection/the-peass-family) संग्रह [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें एक PR जमा करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में।

</details>
