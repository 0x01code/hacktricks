# Writable Sys Path +Dll Hijacking Privesc

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग प्राप्त करें**](https://peass.creator-spring.com)
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) को **फॉलो करें**.
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स शेयर करें.

</details>

## परिचय

यदि आपने पाया है कि आप **System Path फोल्डर में लिख सकते हैं** (ध्यान दें कि यह काम नहीं करेगा यदि आप User Path फोल्डर में लिख सकते हैं), तो संभव है कि आप **सिस्टम में विशेषाधिकार बढ़ा सकते हैं**.

इसके लिए आप **Dll Hijacking** का दुरुपयोग कर सकते हैं जहां आप **एक लाइब्रेरी को हाईजैक करने जा रहे हैं** जिसे एक सेवा या प्रक्रिया आपके से **अधिक विशेषाधिकारों** के साथ लोड कर रही है, और चूंकि वह सेवा एक Dll को लोड कर रही है जो शायद पूरे सिस्टम में मौजूद भी नहीं है, वह इसे System Path से लोड करने की कोशिश करेगी जहां आप लिख सकते हैं.

**Dll Hijackig क्या है** इसके बारे में अधिक जानकारी के लिए देखें:

{% content-ref url="../dll-hijacking.md" %}
[dll-hijacking.md](../dll-hijacking.md)
{% endcontent-ref %}

## Dll Hijacking के साथ Privesc

### एक गायब Dll खोजना

पहली चीज जो आपको चाहिए वह है **एक प्रक्रिया की पहचान करना** जो आपके से **अधिक विशेषाधिकारों** के साथ चल रही है और जो System Path से एक Dll को लोड करने की कोशिश कर रही है जिसमें आप लिख सकते हैं.

इस मामले में समस्या यह है कि शायद वे प्रक्रियाएं पहले से ही चल रही हैं। आपको जिन Dlls की कमी है उन सेवाओं को खोजने के लिए आपको जितनी जल्दी हो सके procmon लॉन्च करना होगा (प्रक्रियाओं के लोड होने से पहले)। तो, गायब .dlls को खोजने के लिए करें:

* **फोल्डर बनाएं** `C:\privesc_hijacking` और पथ `C:\privesc_hijacking` को **System Path env variable** में जोड़ें। आप यह **मैन्युअली** या **PS** के साथ कर सकते हैं:
```powershell
# Set the folder path to create and check events for
$folderPath = "C:\privesc_hijacking"

# Create the folder if it does not exist
if (!(Test-Path $folderPath -PathType Container)) {
New-Item -ItemType Directory -Path $folderPath | Out-Null
}

# Set the folder path in the System environment variable PATH
$envPath = [Environment]::GetEnvironmentVariable("PATH", "Machine")
if ($envPath -notlike "*$folderPath*") {
$newPath = "$envPath;$folderPath"
[Environment]::SetEnvironmentVariable("PATH", $newPath, "Machine")
}
```
* **`procmon`** लॉन्च करें और **`Options`** --> **`Enable boot logging`** पर जाएं और प्रॉम्प्ट में **`OK`** दबाएं।
* फिर, **रिबूट** करें। कंप्यूटर रिस्टार्ट होने पर **`procmon`** तुरंत इवेंट्स **रिकॉर्डिंग** शुरू कर देगा।
* एक बार **Windows** शुरू हो जाने पर **`procmon`** को फिर से चलाएं, यह आपको बताएगा कि यह चल रहा है और आपसे **पूछेगा कि क्या आप इवेंट्स को फाइल में स्टोर करना चाहते हैं**। **हां** कहें और **इवेंट्स को फाइल में स्टोर करें**।
* **फाइल** जनरेट होने के **बाद**, खुली **`procmon`** विंडो को **बंद** करें और **इवेंट्स फाइल खोलें**।
* ये **फिल्टर्स** जोड़ें और आपको वे सभी Dlls मिल जाएंगे जिन्हें किसी **प्रोसेस ने लोड करने की कोशिश की** थी लिखने योग्य System Path फोल्डर से:

<figure><img src="../../../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

### Missed Dlls

मुफ्त **वर्चुअल (vmware) Windows 11 मशीन** में चलाने पर मुझे ये परिणाम मिले:

<figure><img src="../../../.gitbook/assets/image (253).png" alt=""><figcaption></figcaption></figure>

इस मामले में .exe बेकार हैं इसलिए उन्हें अनदेखा करें, गुम हुए DLLs यहाँ से थे:

| सेवा                             | Dll                | CMD लाइन                                                             |
| ------------------------------- | ------------------ | -------------------------------------------------------------------- |
| टास्क शेड्यूलर (Schedule)       | WptsExtensions.dll | `C:\Windows\system32\svchost.exe -k netsvcs -p -s Schedule`          |
| डायग्नोस्टिक पॉलिसी सेवा (DPS) | Unknown.DLL        | `C:\Windows\System32\svchost.exe -k LocalServiceNoNetwork -p -s DPS` |
| ???                             | SharedRes.dll      | `C:\Windows\system32\svchost.exe -k UnistackSvcGroup`                |

इसे पाने के बाद, मैंने इस दिलचस्प ब्लॉग पोस्ट को पाया जो यह भी समझाता है कि कैसे [**WptsExtensions.dll का दुरुपयोग करके privesc किया जा सकता है**](https://juggernaut-sec.com/dll-hijacking/#Windows\_10\_Phantom\_DLL\_Hijacking\_-\_WptsExtensionsdll)। जो हम **अब करने जा रहे हैं**।

### Exploitation

तो, **अधिकारों को बढ़ाने** के लिए हम **WptsExtensions.dll** लाइब्रेरी को हाइजैक करने जा रहे हैं। पथ और नाम होने के बाद हमें बस **दुष्ट dll जनरेट करने की जरूरत है**।

आप [**इन उदाहरणों में से किसी का भी प्रयास कर सकते हैं**](../dll-hijacking.md#creating-and-compiling-dlls)। आप ऐसे पेलोड चला सकते हैं जैसे: एक रिव शेल प्राप्त करना, एक यूजर जोड़ना, एक बीकन निष्पादित करना...

{% hint style="warning" %}
ध्यान दें कि **सभी सेवाएं नहीं चलाई जाती हैं** **`NT AUTHORITY\SYSTEM`** के साथ, कुछ **`NT AUTHORITY\LOCAL SERVICE`** के साथ भी चलाई जाती हैं जिसके **कम अधिकार होते हैं** और आप **नया यूजर नहीं बना पाएंगे** उसकी अनुमतियों का दुरुपयोग करने के लिए।\
हालांकि, उस यूजर के पास **`seImpersonate`** अधिकार होता है, इसलिए आप [**पोटैटो सूट का उपयोग करके अधिकारों को बढ़ा सकते हैं**](../roguepotato-and-printspoofer.md)। इसलिए, इस मामले में एक रिव शेल एक यूजर बनाने की कोशिश करने से बेहतर विकल्प है।
{% endhint %}

लेखन के समय **टास्क शेड्यूलर** सेवा **Nt AUTHORITY\SYSTEM** के साथ चलाई जा रही है।

**दुष्ट Dll जनरेट करने** के बाद (_मेरे मामले में मैंने x64 रिव शेल का उपयोग किया और मुझे एक शेल वापस मिला लेकिन डिफेंडर ने इसे मार दिया क्योंकि यह msfvenom से था_), इसे लिखने योग्य System Path में **WptsExtensions.dll** के नाम से सेव करें और कंप्यूटर को **रिस्टार्ट** करें (या सेवा को रिस्टार्ट करें या जो भी जरूरी हो प्रभावित सेवा/प्रोग्राम को फिर से चलाने के लिए)।

जब सेवा फिर से शुरू होती है, **dll को लोड और निष्पादित किया जाना चाहिए** (आप **procmon** ट्रिक का **पुन: उपयोग कर सकते हैं** यह जांचने के लिए कि **लाइब्रेरी अपेक्षित रूप से लोड की गई थी या नहीं**).

<details>

<summary><strong>Learn AWS hacking from zero to hero with</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग प्राप्त करें**](https://peass.creator-spring.com)
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में शामिल हों या मुझे **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) पर **फॉलो करें**।
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>
