# Pcap Inspection

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert)</strong>에서 **제로**부터 **히어로**까지 **AWS 해킹**을 배우세요!</summary>

HackTricks를 지원하는 다른 방법:

* **회사를 HackTricks에서 광고**하거나 **PDF로 HackTricks 다운로드**하려면 [**구독 요금제**](https://github.com/sponsors/carlospolop)를 확인하세요!
* [**공식 PEASS & HackTricks 스왜그**](https://peass.creator-spring.com)를 구매하세요
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)를 발견하세요, 당사의 독점 [**NFTs**](https://opensea.io/collection/the-peass-family) 컬렉션
* **💬 [Discord 그룹](https://discord.gg/hRep4RUj7f)** 또는 [telegram 그룹](https://t.me/peass)에 **가입**하거나 **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)를 **팔로우**하세요.
* **해킹 트릭을 공유하려면** [**HackTricks**](https://github.com/carlospolop/hacktricks) 및 [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github 저장소에 PR을 제출하세요.

</details>

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

[**RootedCON**](https://www.rootedcon.com/)은 **스페인**에서 가장 중요한 사이버 보안 행사 중 하나로 **유럽**에서도 가장 중요한 행사 중 하나입니다. **기술 지식을 촉진**하는 임무를 가진 이 회의는 모든 분야의 기술 및 사이버 보안 전문가들을 위한 뜨거운 만남의 장입니다.

{% embed url="https://www.rootedcon.com/" %}

{% hint style="info" %}
**PCAP** 대 **PCAPNG**에 대한 참고: PCAP 파일 형식에는 두 가지 버전이 있습니다. **PCAPNG은 더 최신이며 모든 도구에서 지원되지 않을 수 있습니다**. 다른 도구에서 작업하기 위해 파일을 Wireshark나 다른 호환 도구를 사용하여 PCAPNG에서 PCAP으로 변환해야 할 수도 있습니다.
{% endhint %}

## Pcap을 위한 온라인 도구

* Pcap의 헤더가 **손상**된 경우 [http://f00l.de/hacking/**pcapfix.php**](http://f00l.de/hacking/pcapfix.php)를 사용하여 **수정**해야 합니다.
* Pcap에서 **정보를 추출**하고 **악성 코드**를 검색하려면 [**PacketTotal**](https://packettotal.com)을 사용하세요.
* [**www.virustotal.com**](https://www.virustotal.com) 및 [**www.hybrid-analysis.com**](https://www.hybrid-analysis.com)을 사용하여 **악성 활동**을 검색하세요.
* 브라우저에서의 **전체 pcap 분석**은 [**https://apackets.com/**](https://apackets.com/)에서 가능합니다.

## 정보 추출

다음 도구는 통계, 파일 등을 추출하는 데 유용합니다.

### Wireshark

{% hint style="info" %}
**PCAP를 분석할 예정이라면 기본적으로 Wireshark 사용 방법을 알아야 합니다**
{% endhint %}

Wireshark 트릭은 다음에서 찾을 수 있습니다:

{% content-ref url="wireshark-tricks.md" %}
[wireshark-tricks.md](wireshark-tricks.md)
{% endcontent-ref %}

### [**https://apackets.com/**](https://apackets.com/)

브라우저에서의 Pcap 분석.

### Xplico Framework

[**Xplico** ](https://github.com/xplico/xplico)_(only linux)_는 **pcap**을 분석하고 정보를 추출할 수 있습니다. 예를 들어 pcap 파일에서 Xplico는 각 이메일 (POP, IMAP 및 SMTP 프로토콜), 모든 HTTP 콘텐츠, 각 VoIP 통화 (SIP), FTP, TFTP 등을 추출합니다.

**설치**
```bash
sudo bash -c 'echo "deb http://repo.xplico.org/ $(lsb_release -s -c) main" /etc/apt/sources.list'
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 791C25CE
sudo apt-get update
sudo apt-get install xplico
```
**실행**
```
/etc/init.d/apache2 restart
/etc/init.d/xplico start
```
_**127.0.0.1:9876**_에 _**xplico:xplico**_ 자격 증명으로 액세스합니다.

그런 다음 **새로운 사례**를 만들고 해당 사례 내에 **새로운 세션**을 만들고 **pcap** 파일을 **업로드**합니다.

### NetworkMiner

Xplico와 마찬가지로 이 도구는 pcap에서 **분석 및 객체 추출**을 위한 도구입니다. 무료 버전을 [**여기**](https://www.netresec.com/?page=NetworkMiner)에서 **다운로드**할 수 있습니다. **Windows**와 호환됩니다.\
이 도구는 또한 **더 빠른** 방식으로 **패킷에서 다른 정보를 분석**하는 데 유용합니다.

### NetWitness Investigator

[**여기**](https://www.rsa.com/en-us/contact-us/netwitness-investigator-freeware)에서 **NetWitness Investigator를 다운로드**할 수 있습니다 **(Windows에서 작동)**.\
이 도구는 **패킷을 분석**하고 정보를 유용한 방식으로 정렬하여 **내부에서 무슨 일이 일어나고 있는지 파악**하는 데 유용합니다.

### [BruteShark](https://github.com/odedshimon/BruteShark)

* 사용자 이름 및 암호 추출 및 인코딩 (HTTP, FTP, Telnet, IMAP, SMTP...)
* 인증 해시 추출 및 Hashcat을 사용하여 해독 (Kerberos, NTLM, CRAM-MD5, HTTP-Digest...)
* 시각적 네트워크 다이어그램 작성 (네트워크 노드 및 사용자)
* DNS 쿼리 추출
* 모든 TCP 및 UDP 세션 재구성
* 파일 조각내기

### Capinfos
```
capinfos capture.pcap
```
### Ngrep

만약 pcap 내부에서 **무언가를** 찾고 싶다면 **ngrep**를 사용할 수 있습니다. 다음은 주요 필터를 사용한 예시입니다:
```bash
ngrep -I packets.pcap "^GET" "port 80 and tcp and host 192.168 and dst host 192.168 and src host 192.168"
```
### 조각 모음

일반적인 조각 모음 기술을 사용하여 pcap에서 파일과 정보를 추출하는 데 유용할 수 있습니다:

{% content-ref url="../partitions-file-systems-carving/file-data-carving-recovery-tools.md" %}
[file-data-carving-recovery-tools.md](../partitions-file-systems-carving/file-data-carving-recovery-tools.md)
{% endcontent-ref %}

### 자격 증명 캡처

[https://github.com/lgandx/PCredz](https://github.com/lgandx/PCredz)와 같은 도구를 사용하여 pcap 또는 라이브 인터페이스에서 자격 증명을 구문 분석할 수 있습니다.

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

[**RootedCON**](https://www.rootedcon.com/)은 **스페인**에서 가장 관련성 높은 사이버 보안 행사 중 하나이며 **유럽**에서 가장 중요한 행사 중 하나입니다. **기술 지식을 촉진하는 데 봉사**하는 이 컨퍼런스는 모든 분야의 기술 및 사이버 보안 전문가들에게 뜨거운 만남의 장입니다.

{% embed url="https://www.rootedcon.com/" %}

## Exploits/Malware 확인

### Suricata

**설치 및 설정**
```
apt-get install suricata
apt-get install oinkmaster
echo "url = http://rules.emergingthreats.net/open/suricata/emerging.rules.tar.gz" >> /etc/oinkmaster.conf
oinkmaster -C /etc/oinkmaster.conf -o /etc/suricata/rules
```
**pcap 확인**
```
suricata -r packets.pcap -c /etc/suricata/suricata.yaml -k none -v -l log
```
### YaraPcap

[**YaraPCAP**](https://github.com/kevthehermit/YaraPcap)은 다음을 수행하는 도구입니다.

* PCAP 파일을 읽고 HTTP 스트림을 추출합니다.
* 모든 압축된 스트림을 gzip으로 해제합니다.
* Yara로 모든 파일을 스캔합니다.
* report.txt를 작성합니다.
* 선택적으로 일치하는 파일을 디렉토리에 저장합니다.

### 악성코드 분석

알려진 악성코드의 지문을 찾을 수 있는지 확인하십시오:

{% content-ref url="../malware-analysis.md" %}
[malware-analysis.md](../malware-analysis.md)
{% endcontent-ref %}

## Zeek

> [Zeek](https://docs.zeek.org/en/master/about.html)는 수동적인 오픈 소스 네트워크 트래픽 분석기입니다. 많은 운영자들이 의심스러운 또는 악의적인 활동의 조사를 지원하기 위해 네트워크 보안 모니터(NSM)로서 Zeek를 사용합니다. Zeek는 보안 도메인을 넘어서 성능 측정 및 문제 해결을 포함한 다양한 트래픽 분석 작업을 지원합니다.

기본적으로 `zeek`에 의해 생성된 로그는 **pcap**이 아닙니다. 따라서 **pcap에 대한 정보**가 있는 로그를 분석하기 위해 **다른 도구**를 사용해야 합니다.
```bash
#Get info about longest connections (add "grep udp" to see only udp traffic)
#The longest connection might be of malware (constant reverse shell?)
cat conn.log | zeek-cut id.orig_h id.orig_p id.resp_h id.resp_p proto service duration | sort -nrk 7 | head -n 10

10.55.100.100   49778   65.52.108.225   443     tcp     -       86222.365445
10.55.100.107   56099   111.221.29.113  443     tcp     -       86220.126151
10.55.100.110   60168   40.77.229.82    443     tcp     -       86160.119664


#Improve the metrics by summing up the total duration time for connections that have the same destination IP and Port.
cat conn.log | zeek-cut id.orig_h id.resp_h id.resp_p proto duration | awk 'BEGIN{ FS="\t" } { arr[$1 FS $2 FS $3 FS $4] += $5 } END{ for (key in arr) printf "%s%s%s\n", key, FS, arr[key] }' | sort -nrk 5 | head -n 10

10.55.100.100   65.52.108.225   443     tcp     86222.4
10.55.100.107   111.221.29.113  443     tcp     86220.1
10.55.100.110   40.77.229.82    443     tcp     86160.1

#Get the number of connections summed up per each line
cat conn.log | zeek-cut id.orig_h id.resp_h duration | awk 'BEGIN{ FS="\t" } { arr[$1 FS $2] += $3; count[$1 FS $2] += 1 } END{ for (key in arr) printf "%s%s%s%s%s\n", key, FS, count[key], FS, arr[key] }' | sort -nrk 4 | head -n 10

10.55.100.100   65.52.108.225   1       86222.4
10.55.100.107   111.221.29.113  1       86220.1
10.55.100.110   40.77.229.82    134       86160.1

#Check if any IP is connecting to 1.1.1.1
cat conn.log | zeek-cut id.orig_h id.resp_h id.resp_p proto service | grep '1.1.1.1' | sort | uniq -c

#Get number of connections per source IP, dest IP and dest Port
cat conn.log | zeek-cut id.orig_h id.resp_h id.resp_p proto | awk 'BEGIN{ FS="\t" } { arr[$1 FS $2 FS $3 FS $4] += 1 } END{ for (key in arr) printf "%s%s%s\n", key, FS, arr[key] }' | sort -nrk 5 | head -n 10


# RITA
#Something similar can be done with the tool rita
rita show-long-connections -H --limit 10 zeek_logs

+---------------+----------------+--------------------------+----------------+
|   SOURCE IP   | DESTINATION IP | DSTPORT:PROTOCOL:SERVICE |    DURATION    |
+---------------+----------------+--------------------------+----------------+
| 10.55.100.100 | 65.52.108.225  | 443:tcp:-                | 23h57m2.3655s  |
| 10.55.100.107 | 111.221.29.113 | 443:tcp:-                | 23h57m0.1262s  |
| 10.55.100.110 | 40.77.229.82   | 443:tcp:-                | 23h56m0.1197s  |

#Get connections info from rita
rita show-beacons zeek_logs | head -n 10
Score,Source IP,Destination IP,Connections,Avg Bytes,Intvl Range,Size Range,Top Intvl,Top Size,Top Intvl Count,Top Size Count,Intvl Skew,Size Skew,Intvl Dispersion,Size Dispersion
1,192.168.88.2,165.227.88.15,108858,197,860,182,1,89,53341,108319,0,0,0,0
1,10.55.100.111,165.227.216.194,20054,92,29,52,1,52,7774,20053,0,0,0,0
0.838,10.55.200.10,205.251.194.64,210,69,29398,4,300,70,109,205,0,0,0,0
```
### DNS 정보
```bash
#Get info about each DNS request performed
cat dns.log | zeek-cut -c id.orig_h query qtype_name answers

#Get the number of times each domain was requested and get the top 10
cat dns.log | zeek-cut query | sort | uniq | rev | cut -d '.' -f 1-2 | rev | sort | uniq -c | sort -nr | head -n 10

#Get all the IPs
cat dns.log | zeek-cut id.orig_h query | grep 'example\.com' | cut -f 1 | sort | uniq -c

#Sort the most common DNS record request (should be A)
cat dns.log | zeek-cut qtype_name | sort | uniq -c | sort -nr

#See top DNS domain requested with rita
rita show-exploded-dns -H --limit 10 zeek_logs
```
## 다른 pcap 분석 트릭

{% content-ref url="dnscat-exfiltration.md" %}
[dnscat-exfiltration.md](dnscat-exfiltration.md)
{% endcontent-ref %}

{% content-ref url="wifi-pcap-analysis.md" %}
[wifi-pcap-analysis.md](wifi-pcap-analysis.md)
{% endcontent-ref %}

{% content-ref url="usb-keystrokes.md" %}
[usb-keystrokes.md](usb-keystrokes.md)
{% endcontent-ref %}

​

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

[**RootedCON**](https://www.rootedcon.com/)은 **스페인**에서 가장 관련성 높은 사이버 보안 이벤트 중 하나이며 **유럽**에서 가장 중요한 행사 중 하나입니다. **기술 지식을 촉진하는 데 봉사하는** 이 컨퍼런스는 모든 분야의 기술 및 사이버 보안 전문가들에게 열정적인 만남의 장입니다.

{% embed url="https://www.rootedcon.com/" %}

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert)</strong>를 통해 **제로부터 영웅까지 AWS 해킹 배우기**!</summary>

HackTricks를 지원하는 다른 방법:

* **회사가 HackTricks에 광고되길 원하거나 PDF로 HackTricks를 다운로드하길 원한다면** [**구독 요금제**](https://github.com/sponsors/carlospolop)를 확인하세요!
* [**공식 PEASS & HackTricks 스왜그**](https://peass.creator-spring.com)를 구매하세요
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)를 발견하세요, 당사의 독점 [**NFTs**](https://opensea.io/collection/the-peass-family) 컬렉션
* 💬 [**디스코드 그룹**](https://discord.gg/hRep4RUj7f) 또는 [**텔레그램 그룹**](https://t.me/peass)에 **가입**하거나 **트위터** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**를 팔로우**하세요.
* **HackTricks** 및 **HackTricks Cloud** 깃허브 저장소에 PR을 제출하여 **해킹 트릭을 공유**하세요.

</details>
