# macOS Entitlements Perigosos e Permissões TCC

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* Você trabalha em uma **empresa de cibersegurança**? Você quer ver sua **empresa anunciada no HackTricks**? ou você quer ter acesso à **última versão do PEASS ou baixar o HackTricks em PDF**? Verifique os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* Adquira o [**swag oficial do PEASS & HackTricks**](https://peass.creator-spring.com)
* **Junte-se ao** [**💬**](https://emojipedia.org/speech-balloon/) [**grupo Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo telegram**](https://t.me/peass) ou **siga-me** no **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Compartilhe seus truques de hacking enviando PRs para o** [**repositório hacktricks**](https://github.com/carlospolop/hacktricks) **e** [**repositório hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>

{% hint style="warning" %}
Observe que as permissões que começam com **`com.apple`** não estão disponíveis para terceiros, apenas a Apple pode concedê-las.
{% endhint %}

## Alto

### `com.apple.rootless.install.heritable`

A permissão **`com.apple.rootless.install.heritable`** permite **burlar o SIP**. Verifique [isto para mais informações](macos-sip.md#com.apple.rootless.install.heritable).

### **`com.apple.rootless.install`**

A permissão **`com.apple.rootless.install`** permite **burlar o SIP**. Verifique [isto para mais informações](macos-sip.md#com.apple.rootless.install).

### **`com.apple.system-task-ports` (anteriormente chamado de `task_for_pid-allow`)**

Essa permissão permite obter a **porta da tarefa para qualquer** processo, exceto o kernel. Verifique [**isto para mais informações**](../mac-os-architecture/macos-ipc-inter-process-communication/).

### `com.apple.security.get-task-allow`

Essa permissão permite que outros processos com a permissão **`com.apple.security.cs.debugger`** obtenham a porta da tarefa do processo executado pelo binário com essa permissão e **injetem código nele**. Verifique [**isto para mais informações**](../mac-os-architecture/macos-ipc-inter-process-communication/).

### `com.apple.security.cs.debugger`

Aplicativos com a Permissão da Ferramenta de Depuração podem chamar `task_for_pid()` para recuperar uma porta de tarefa válida para aplicativos não assinados e de terceiros com a permissão `Get Task Allow` definida como `true`. No entanto, mesmo com a permissão da ferramenta de depuração, um depurador **não pode obter as portas de tarefa** de processos que **não possuem a permissão `Get Task Allow`**, e que portanto estão protegidos pela Proteção de Integridade do Sistema. Verifique [**isto para mais informações**](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_debugger).

### `com.apple.security.cs.disable-library-validation`

Essa permissão permite **carregar frameworks, plug-ins ou bibliotecas sem serem assinados pela Apple ou assinados com o mesmo ID de equipe** que o executável principal, portanto, um invasor pode abusar de alguma carga de biblioteca arbitrária para injetar código. Verifique [**isto para mais informações**](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_disable-library-validation).

### `com.apple.security.cs.allow-dyld-environment-variables`

Essa permissão permite **usar variáveis de ambiente DYLD** que podem ser usadas para injetar bibliotecas e código. Verifique [**isto para mais informações**](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_allow-dyld-environment-variables).

### `com.apple.private.tcc.manager` e `com.apple.rootless.storage`.`TCC`

[**De acordo com este blog**](https://objective-see.org/blog/blog\_0x4C.html), essas permissões permitem **modificar** o **banco de dados TCC**.

### `com.apple.private.tcc.manager.check-by-audit-token`

TODO: Eu não sei o que isso permite fazer

### `com.apple.private.apfs.revert-to-snapshot`

TODO: No [**este relatório**](https://jhftss.github.io/The-Nightmare-of-Apple-OTA-Update/) **é mencionado que isso poderia ser usado para** atualizar o conteúdo protegido por SSV após uma reinicialização. Se você souber como, envie um PR, por favor!

### `com.apple.private.apfs.create-sealed-snapshot`

TODO: No [**este relatório**](https://jhftss.github.io/The-Nightmare-of-Apple-OTA-Update/) **é mencionado que isso poderia ser usado para** atualizar o conteúdo protegido por SSV após uma reinicialização. Se você souber como, envie um PR, por favor!

### **`kTCCServiceSystemPolicyAllFiles`**

Concede permissões de **Acesso Total ao Disco**, uma das permissões mais altas do TCC que você pode ter.

### **`kTCCServiceAppleEvents`**

Permite que o aplicativo envie eventos para outros aplicativos que são comumente usados para **automatizar tarefas**. Controlando outros aplicativos, ele pode abusar das permissões concedidas a esses outros aplicativos.
### **`kTCCServiceSystemPolicySysAdminFiles`**

Permite **alterar** o atributo **`NFSHomeDirectory`** de um usuário que altera sua pasta inicial e, portanto, permite **burlar o TCC**.

### **`kTCCServiceSystemPolicyAppBundles`**

Permite modificar aplicativos dentro de suas pastas (dentro de app.app), o que é desativado por padrão.

## Médio

### `com.apple.security.cs.allow-jit`

Essa permissão permite **criar memória que pode ser gravada e executada** passando a flag `MAP_JIT` para a função de sistema `mmap()`. Verifique [**aqui para mais informações**](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_allow-jit).

### `com.apple.security.cs.allow-unsigned-executable-memory`

Essa permissão permite **sobrescrever ou corrigir código C**, usar o **`NSCreateObjectFileImageFromMemory`** (que é fundamentalmente inseguro) ou usar o framework **DVDPlayback**. Verifique [**aqui para mais informações**](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_allow-unsigned-executable-memory).

{% hint style="danger" %}
Incluir essa permissão expõe seu aplicativo a vulnerabilidades comuns em linguagens de código inseguras em relação à memória. Considere cuidadosamente se seu aplicativo precisa dessa exceção.
{% endhint %}

### `com.apple.security.cs.disable-executable-page-protection`

Essa permissão permite **modificar seções de seus próprios arquivos executáveis** no disco para sair forçadamente. Verifique [**aqui para mais informações**](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_disable-executable-page-protection).

{% hint style="danger" %}
A permissão de Desabilitar Proteção de Memória Executável é uma permissão extrema que remove uma proteção de segurança fundamental do seu aplicativo, tornando possível que um invasor reescreva o código executável do seu aplicativo sem detecção. Prefira permissões mais restritas, se possível.
{% endhint %}

### `com.apple.security.cs.allow-relative-library-loads`

TODO

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* Você trabalha em uma **empresa de cibersegurança**? Gostaria de ver sua **empresa anunciada no HackTricks**? Ou gostaria de ter acesso à **versão mais recente do PEASS ou baixar o HackTricks em PDF**? Confira os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* Adquira o [**swag oficial do PEASS & HackTricks**](https://peass.creator-spring.com)
* **Junte-se ao** [**💬**](https://emojipedia.org/speech-balloon/) [**grupo Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo telegram**](https://t.me/peass) ou **siga-me** no **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Compartilhe suas técnicas de hacking enviando PRs para o** [**repositório hacktricks**](https://github.com/carlospolop/hacktricks) **e o** [**repositório hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
