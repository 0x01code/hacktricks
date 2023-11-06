# केरबेरोस डबल हॉप समस्या

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल** हों या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)** का पालन करें**.
* **अपने हैकिंग ट्रिक्स को** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **में PR जमा करके अपना योगदान दें।**

</details>

## परिचय

केरबेरोस "डबल हॉप" समस्या उत्पन्न होती है जब एक हमलावर को दो **हॉप** के माध्यम से केरबेरोस प्रमाणीकरण का उपयोग करने की कोशिश करता है, उदाहरण के लिए **PowerShell**/**WinRM** का उपयोग करके।

जब केरबेरोस के माध्यम से **प्रमाणीकरण** होता है, तो **क्रेडेंशियल्स** को **मेमोरी में कैश नहीं किया जाता है।** इसलिए, यदि आप mimikatz चलाते हैं तो आपको मशीन में उपयोगकर्ता के क्रेडेंशियल्स नहीं मिलेंगे, भले ही वह प्रक्रियाएँ चला रहा हो।

यह इसलिए है क्योंकि केरबेरोस के साथ कनेक्ट करते समय ये कदम होते हैं:

1. उपयोगकर्ता 1 क्रेडेंशियल्स प्रदान करता है और **डोमेन कंट्रोलर** उपयोगकर्ता 1 को केरबेरोस **TGT** वापस करता है।
2. उपयोगकर्ता 1 **TGT** का उपयोग करके **सर्विस टिकट** का अनुरोध करता है **सर्वर 1** से कनेक्ट होने के लिए।
3. उपयोगकर्ता 1 **सर्वर 1** से कनेक्ट होता है और **सर्विस टिकट** प्रदान करता है।
4. **सर्वर 1** में उपयोगकर्ता 1 के **क्रेडेंशियल्स** या **TGT** नहीं होते हैं। इसलिए, जब सर्वर 1 से उपयोगकर्ता 1 दूसरे सर्वर में लॉगिन करने की कोशिश करता है, वह **प्रमाणीकरण करने में सक्षम नहीं होता है**।

### असीमित डिलीगेशन

यदि PC में **असीमित डिलीगेशन** सक्षम है, तो ऐसा नहीं होगा क्योंकि **सर्वर** को उसे एक उपयोगकर्ता के **TGT** मिलेगा जो उसे उपयोग कर रहा है। इसके अलावा, यदि असीमित डिलीगेशन का उपयोग किया जाता है, तो आप शायद इससे **डोमेन कंट्रोलर को संक्रमित** कर सकते हैं।\
[**असीमित डिलीगेशन पृष्ठ में अधिक जानकारी**](unconstrained-delegation.md).

### CredSSP

इस समस्या से बचने के लिए **सिस्टम व्यवस्थापकों** के लिए एक और सुझाव दिया जाता है जो [**अत्यंत असुरक्षित**](https://docs.microsoft.com/en-us/powershell/module/microsoft.wsman.management/enable-wsmancredssp?view=powershell-7) है, वह है **Credential Security Support Provider (CredSSP)**। CredSSP को सक्षम करने का समाधान विभिन्न फोरमों पर वर्षों से उल्लेखित किया गया है। माइक्रोसॉफ्ट के अनुसार:

_"CredSSP प्रमाणीकरण स्थानीय कंप्यूटर से दूसरे कंप्यूटर को उपयोगकर्ता क्रेडेंशियल्स अनुप्रेषित करता है। इस अभ्यास से दूरस्थ कंप्यूटर की सुरक्षा जोखिम बढ़ जाता है। यदि दूरस्थ कंप्यूटर प्रभावित हो जाता है, तो जब क्रेडेंशियल्स उसे पास किए जाते हैं, तो क्रेडेंशियल्स का उपयोग नेटवर्क सत्र को नियंत्रित करने के लिए किया जा सकता है।"_

यदि आप प्रोडक्शन सिस्टमों, संवेदनशील नेटवर्कों आदि पर **CredSSP सक्षम** पाते हैं, तो उन्हें अक्षम करना सिफारिश किया जाता है। CredSSP स्थिति की जांच करने का एक त्वरित तरीका है `Get-WSManCredSSP` चलाना। जो दूरस्थता में निष्पादित किया जा सकता है यदि WinRM सक्षम है।
```powershell
Invoke-Command -ComputerName bizintel -Credential ta\redsuit -ScriptBlock {
Get-WSManCredSSP
}
```
## उपाय

### Invoke Command <a href="#invoke-command" id="invoke-command"></a>

यह विधि दोहरी हॉप समस्या के साथ काम करने की तरह है, न कि इसे हल करने की तरह। इसमें कोई कॉन्फ़िगरेशन की आवश्यकता नहीं होती है, और आप इसे आसानी से अपने हमलावर बॉक्स से चला सकते हैं। यह मूल रूप से एक **नेस्टेड `Invoke-Command`** है।

यह **दूसरे सर्वर पर `hostname` चलाएगा:**
```powershell
$cred = Get-Credential ta\redsuit
Invoke-Command -ComputerName bizintel -Credential $cred -ScriptBlock {
Invoke-Command -ComputerName secdev -Credential $cred -ScriptBlock {hostname}
}
```
आपके पास पहले सर्वर के साथ एक **PS-Session** स्थापित किया जा सकता है और बस वहां से `$cred` के साथ **`Invoke-Command`** चला सकते हैं इसे नेस्ट करने की बजाय। हालांकि, इसे अपने हमलावर बॉक्स से चलाने से कार्य केंद्रीकृत होता है:
```powershell
# From the WinRM connection
$pwd = ConvertTo-SecureString 'uiefgyvef$/E3' -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential('DOMAIN\username', $pwd)
# Use "-Credential $cred" option in Powerview commands
```
### पीएससेशन कॉन्फ़िगरेशन पंजीकृत करें

यदि आप **`evil-winrm`** की बजाय **`Enter-PSSession`** कमांडलेट का उपयोग कर सकते हैं, तो आप फिर से जुड़कर डबल हॉप समस्या को टाल सकते हैं। इसके लिए **`Register-PSSessionConfiguration`** का उपयोग करें:
```powershell
# Register a new PS Session configuration
Register-PSSessionConfiguration -Name doublehopsess -RunAsCredential domain_name\username
# Restar WinRM
Restart-Service WinRM
# Get a PSSession
Enter-PSSession -ConfigurationName doublehopsess -ComputerName <pc_name> -Credential domain_name\username
# Check that in this case the TGT was sent and is in memory of the PSSession
klist
# In this session you won't have the double hop problem anymore
```
### पोर्ट फ़ॉरवर्डिंग <a href="#portproxy" id="portproxy"></a>

चूंकि हमारे पास इंटरमीडिएट टारगेट **bizintel: 10.35.8.17** पर स्थानीय प्रशासक है, आप एक पोर्ट फ़ॉरवर्डिंग नियम जोड़कर अपने अनुरोधों को अंतिम/तीसरे सर्वर **secdev: 10.35.8.23** पर भेज सकते हैं।

आप एक वन-लाइनर निकालने और नियम जोड़ने के लिए त्वरित रूप से **netsh** का उपयोग कर सकते हैं।
```bash
netsh interface portproxy add v4tov4 listenport=5446 listenaddress=10.35.8.17 connectport=5985 connectaddress=10.35.8.23
```
तो **पहली सर्वर** पोर्ट 5446 पर सुन रही है और 5446 पर आने वाले अनुरोधों को **दूसरे सर्वर** पोर्ट 5985 (यानी WinRM) पर आगे भेजेगी।

फिर Windows फ़ायरवॉल में एक छेद बनाएं, जिसे एक तेज़ netsh one-liner के साथ भी किया जा सकता है।
```bash
netsh advfirewall firewall add rule name=fwd dir=in action=allow protocol=TCP localport=5446
```
अब सत्र स्थापित करें, जो हमें **पहले सर्वर** के पास आगे भेजेगा।

<figure><img src="../../.gitbook/assets/image (3) (5) (1).png" alt=""><figcaption></figcaption></figure>

#### winrs.exe <a href="#winrsexe" id="winrsexe"></a>

**Portforwarding WinRM** अनुरोधों को भी **`winrs.exe`** का उपयोग करके काम करने लगता है। यदि आपको यह ज्ञात है कि PowerShell का अनुगमन किया जा रहा है, तो यह एक बेहतर विकल्प हो सकता है। नीचे दिए गए कमांड में `hostname` के रूप में "secdev" को परिणाम के रूप में लाता है।
```bash
winrs -r:http://bizintel:5446 -u:ta\redsuit -p:2600leet hostname
```
इसकी तरह, यह आक्रमणकारी सिस्टम कमांड को एक तर्क के रूप में जारी कर सकता है, जैसे `Invoke-Command`। एक सामान्य बैच स्क्रिप्ट उदाहरण _winrm.bat_:

<figure><img src="../../.gitbook/assets/image (2) (6) (2).png" alt=""><figcaption></figcaption></figure>

### OpenSSH <a href="#openssh" id="openssh"></a>

इस तरीके के लिए पहले सर्वर बॉक्स पर [OpenSSH इंस्टॉल करना](https://github.com/PowerShell/Win32-OpenSSH/wiki/Install-Win32-OpenSSH) आवश्यक होता है। विंडोज के लिए OpenSSH इंस्टॉल करना **पूरी तरह से CLI के माध्यम से** किया जा सकता है और यह बहुत कम समय लेता है - और यह मैलवेयर के रूप में नहीं फ्लैग होता है!

बेशक कुछ परिस्थितियों में यह संभव नहीं हो सकता है, यह ज्यादा बोझिल हो सकता है या यह एक सामान्य ऑपसेक जोखिम हो सकता है।

यह तरीका विशेष रूप से एक जंप बॉक्स सेटअप पर उपयोगी हो सकता है - जहां एक अनुप्रयोगी नेटवर्क तक पहुंच होती है। एक बार SSH कनेक्शन स्थापित हो जाता है, उपयोगकर्ता/आक्रमणकारी अनुरोध के रूप में जितने भी `New-PSSession` की आवश्यकता होती है, उन्हें डबल-हॉप समस्या में धमाका नहीं करना पड़ता है।

OpenSSH में **पासवर्ड प्रमाणीकरण** का उपयोग करने पर (कुंजी या केरबेरोस नहीं), **लॉगऑन प्रकार 8** यानी _नेटवर्क क्लियर पाठ लॉगऑन_ होता है। इसका अर्थ यह नहीं है कि आपका पासवर्ड साफ़ रूप में भेजा जाता है - यह वास्तव में SSH द्वारा एन्क्रिप्ट किया जाता है। आपके सत्र के लिए इसके [प्रमाणीकरण पैकेज](https://docs.microsoft.com/en-us/windows/win32/api/winbase/nf-winbase-logonusera?redirectedfrom=MSDN) के माध्यम से यह स्पष्ट पाठ में अनबन्धित हो जाता है ताकि आपकी सत्र और अधिक ताजगी से भरे गए TGT का अनुरोध कर सकें!

इससे आपको बीचकारी सर्वर को आपकी ओर से TGT का अनुरोध करने और प्राथमिक सर्वर पर स्थानीय रूप से संग्रहीत करने की अनुमति मिलती है। आपकी सत्र इस TGT का उपयोग करके अतिरिक्त सर्वरों के प्रतिद्वंद्वीकरण (PS रिमोट) के लिए प्रमाणित कर सकती है।

#### OpenSSH इंस्टॉल स्थिति

अपने आक्रमणकारी बॉक्स पर सबसे नवीनतम [OpenSSH रिलीज़ ज़िप फ़ाइल](https://github.com/PowerShell/Win32-OpenSSH/releases) डाउनलोड करें और इसे स्थानांतरित करें (या इसे सीधे जंप बॉक्स पर डाउनलोड करें)।

जहां चाहें वहां ज़िप फ़ाइल को अनबन्ध करें। फिर, इंस्टॉल स्क्रिप्ट - `Install-sshd.ps1` चलाएं

<figure><img src="../../.gitbook/assets/image (2) (1) (3).png" alt=""><figcaption></figcaption></figure>

अंत में, **पोर्ट 22 खोलने** के लिए एक फ़ायरवॉल नियम जोड़ें। SSH सेवाएं स्थापित होने की पुष्टि करें और उन्हें शुरू करें। इन दोनों सेवाओं को चलाने के लिए SSH की आवश्यकता होगी।

<figure><img src="../../.gitbook/assets/image (1) (7).png" alt=""><figcaption></figcaption></figure>

यदि आपको `Connection reset` त्रुटि मिलती है, तो अनुमतियों को अद्यतन करें ताकि **सभी: पढ़ें और क्रियान्वय करें** रूट OpenSSH निर्देशिका पर अनुमति हो।
```bash
icacls.exe "C:\Users\redsuit\Documents\ssh\OpenSSH-Win64" /grant Everyone:RX /T
```
## संदर्भ

* [https://techcommunity.microsoft.com/t5/ask-the-directory-services-team/understanding-kerberos-double-hop/ba-p/395463?lightbox-message-images-395463=102145i720503211E78AC20](https://techcommunity.microsoft.com/t5/ask-the-directory-services-team/understanding-kerberos-double-hop/ba-p/395463?lightbox-message-images-395463=102145i720503211E78AC20)
* [https://posts.slayerlabs.com/double-hop/](https://posts.slayerlabs.com/double-hop/)
* [https://learn.microsoft.com/en-gb/archive/blogs/sergey\_babkins\_blog/another-solution-to-multi-hop-powershell-remoting](https://learn.microsoft.com/en-gb/archive/blogs/sergey\_babkins\_blog/another-solution-to-multi-hop-powershell-remoting)
* [https://4sysops.com/archives/solve-the-powershell-multi-hop-problem-without-using-credssp/](https://4sysops.com/archives/solve-the-powershell-multi-hop-problem-without-using-credssp/)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **हैकट्रिक्स में विज्ञापित** देखना चाहते हैं? या क्या आप **PEASS के नवीनतम संस्करण का उपयोग करना चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को।**

</details>
