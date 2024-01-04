# Ataque macOS xpc\_connection\_get\_audit\_token

<details>

<summary><strong>Aprenda hacking no AWS do zero ao herói com</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Outras formas de apoiar o HackTricks:

* Se você quer ver sua **empresa anunciada no HackTricks** ou **baixar o HackTricks em PDF**, confira os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Adquira o [**merchandising oficial do PEASS & HackTricks**](https://peass.creator-spring.com)
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção de [**NFTs**](https://opensea.io/collection/the-peass-family) exclusivos
* **Junte-se ao grupo do** 💬 [**Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo do telegram**](https://t.me/peass) ou **siga-me** no **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **Compartilhe suas técnicas de hacking enviando PRs para os repositórios do GitHub** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>

**Esta técnica foi copiada de** [**https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/**](https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/)

## Informações Básicas sobre Mensagens Mach

Se você não sabe o que são Mensagens Mach, comece verificando esta página:

{% content-ref url="../../../../mac-os-architecture/macos-ipc-inter-process-communication/" %}
[macos-ipc-inter-process-communication](../../../../mac-os-architecture/macos-ipc-inter-process-communication/)
{% endcontent-ref %}

Por enquanto, lembre-se de que:
Mensagens Mach são enviadas através de uma _porta mach_, que é um canal de comunicação **com um único receptor e múltiplos remetentes** incorporado ao kernel mach. **Múltiplos processos podem enviar mensagens** para uma porta mach, mas em qualquer momento **apenas um único processo pode lê-la**. Assim como descritores de arquivo e sockets, portas mach são alocadas e gerenciadas pelo kernel e os processos apenas veem um inteiro, que podem usar para indicar ao kernel qual das suas portas mach desejam usar.

## Conexão XPC

Se você não sabe como uma conexão XPC é estabelecida, verifique:

{% content-ref url="../" %}
[..](../)
{% endcontent-ref %}

## Resumo da Vulnerabilidade

O que é interessante para você saber é que a abstração do XPC é uma conexão um-para-um, mas é baseada em uma tecnologia que **pode ter múltiplos remetentes, então:**

* Portas Mach são de receptor único, _**múltiplos remetentes**_.
* O token de auditoria de uma conexão XPC é o token de auditoria _**copiado da mensagem mais recentemente recebida**_.
* Obter o **token de auditoria** de uma conexão XPC é crítico para muitas **verificações de segurança**.

Embora a situação anterior pareça promissora, existem alguns cenários onde isso não vai causar problemas:

* Tokens de auditoria são frequentemente usados para uma verificação de autorização para decidir se aceitam uma conexão. Como isso acontece usando uma mensagem para o serviço de porta, ainda **não há conexão estabelecida**. Mais mensagens nesta porta serão apenas tratadas como solicitações de conexão adicionais. Então, qualquer **verificação antes de aceitar uma conexão não é vulnerável** (isso também significa que dentro de `-listener:shouldAcceptNewConnection:` o token de auditoria é seguro). Portanto, estamos **procurando por conexões XPC que verifiquem ações específicas**.
* Manipuladores de eventos XPC são tratados de forma síncrona. Isso significa que o manipulador de eventos para uma mensagem deve ser concluído antes de chamá-lo para a próxima, mesmo em filas de despacho concorrentes. Então, dentro de um **manipulador de eventos XPC o token de auditoria não pode ser sobrescrito** por outras mensagens normais (não-resposta!).

Isso nos deu a ideia para dois métodos diferentes que isso pode ser possível:

1. Variante1:
* **Exploit** **conecta** ao serviço **A** e serviço **B**
* Serviço **B** pode chamar uma **funcionalidade privilegiada** no serviço A que o usuário não pode
* Serviço **A** chama **`xpc_connection_get_audit_token`** enquanto _**não**_ está dentro do **manipulador de eventos** para uma conexão em um **`dispatch_async`**.
* Então, uma **mensagem diferente** poderia **sobrescrever o Token de Auditoria** porque está sendo despachada de forma assíncrona fora do manipulador de eventos.
* O exploit passa para o **serviço B o direito de ENVIO para o serviço A**.
* Então, o svc **B** estará na verdade **enviando** as **mensagens** para o serviço **A**.
* O **exploit** tenta **chamar** a **ação privilegiada**. Em uma RC, o svc **A** **verifica** a autorização desta **ação** enquanto **o svc B sobrescreveu o Token de Auditoria** (dando ao exploit acesso para chamar a ação privilegiada).
2. Variante 2:
* Serviço **B** pode chamar uma **funcionalidade privilegiada** no serviço A que o usuário não pode
* Exploit se conecta com **serviço A** que **envia** ao exploit uma **mensagem esperando uma resposta** em uma porta de **resposta** específica.
* Exploit envia **serviço** B uma mensagem passando **essa porta de resposta**.
* Quando o serviço **B responde**, ele **envia a mensagem para o serviço A**, **enquanto** o **exploit** envia uma mensagem diferente **para o serviço A** tentando **alcançar uma funcionalidade privilegiada** e esperando que a resposta do serviço B sobrescreva o Token de Auditoria no momento perfeito (Condição de Corrida).

## Variante 1: chamando xpc\_connection\_get\_audit\_token fora de um manipulador de eventos <a href="#variant-1-calling-xpc_connection_get_audit_token-outside-of-an-event-handler" id="variant-1-calling-xpc_connection_get_audit_token-outside-of-an-event-handler"></a>

Cenário:

* Dois serviços mach **A** e **B** aos quais podemos nos conectar (baseado no perfil de sandbox e nas verificações de autorização antes de aceitar a conexão).
* **A** deve ter uma **verificação de autorização** para uma **ação específica que **_**B**_** pode passar** (mas nosso aplicativo não pode).
* Por exemplo, se B tem alguns **direitos** ou está executando como **root**, isso pode permitir que ele peça a A para realizar uma ação privilegiada.
* Para esta verificação de autorização, **A** obtém o token de auditoria de forma assíncrona, por exemplo, chamando `xpc_connection_get_audit_token` de um **`dispatch_async`**.

{% hint style="danger" %}
Neste caso, um atacante poderia desencadear uma **Condição de Corrida** fazendo um **exploit** que **pede a A para realizar uma ação** várias vezes enquanto faz **B enviar mensagens para A**. Quando a RC é **bem-sucedida**, o **token de auditoria** de **B** será copiado na memória **enquanto** a solicitação do nosso **exploit** está sendo **tratada** por A, dando-lhe **acesso à ação privilegiada que apenas B poderia solicitar**.
{% endhint %}

Isso aconteceu com **A** como `smd` e **B** como `diagnosticd`. A função [`SMJobBless`](https://developer.apple.com/documentation/servicemanagement/1431078-smjobbless?language=objc) do smb pode ser usada para instalar uma nova ferramenta de ajuda privilegiada (como **root**). Se um **processo executando como root contatar** **smd**, nenhuma outra verificação será realizada.

Portanto, o serviço **B** é **`diagnosticd`** porque ele executa como **root** e pode ser usado para **monitorar** um processo, então, uma vez que o monitoramento começou, ele enviará **várias mensagens por segundo**.

Para realizar o ataque:

1. Estabelecemos nossa **conexão** com **`smd`** seguindo o protocolo XPC normal.
2. Em seguida, estabelecemos uma **conexão** com **`diagnosticd`**, mas em vez de gerar duas novas portas mach e enviá-las, substituímos o direito de envio da porta do cliente por uma cópia do **direito de envio que temos para a conexão com `smd`**.
3. O que isso significa é que podemos enviar mensagens XPC para `diagnosticd`, mas quaisquer **mensagens que `diagnosticd` envie vão para `smd`**.
* Para `smd`, as mensagens de ambos, nós e `diagnosticd`, parecem chegar na mesma conexão.

<figure><img src="../../../../../../.gitbook/assets/image (1) (1) (1) (1) (1).png" alt="" width="563"><figcaption></figcaption></figure>

4. Pedimos a **`diagnosticd`** para **começar a monitorar** nosso processo (ou qualquer processo ativo) e **enviamos mensagens rotineiras 1004 para `smd`** (para instalar uma ferramenta privilegiada).
5. Isso cria uma condição de corrida que precisa atingir uma janela muito específica em `handle_bless`. Precisamos que a chamada para `xpc_connection_get_pid` retorne o PID do nosso próprio processo, já que a ferramenta de ajuda privilegiada está no nosso pacote de aplicativos. No entanto, a chamada para `xpc_connection_get_audit_token` dentro da função `connection_is_authorized` deve usar o token de auditoria de `diagnosticd`.

## Variante 2: encaminhamento de resposta

Como mencionado anteriormente, o manipulador de eventos em uma conexão XPC nunca é executado várias vezes simultaneamente. No entanto, **mensagens de resposta XPC são tratadas de forma diferente**. Existem duas funções para enviar uma mensagem que espera uma resposta:

* `void xpc_connection_send_message_with_reply(xpc_connection_t connection, xpc_object_t message, dispatch_queue_t replyq, xpc_handler_t handler)`, no qual o caso da mensagem XPC é recebida e analisada na fila especificada.
* `xpc_object_t xpc_connection_send_message_with_reply_sync(xpc_connection_t connection, xpc_object_t message)`, no qual o caso da mensagem XPC é recebida e analisada na fila de despacho atual.

Portanto, **pacotes de resposta XPC podem ser analisados enquanto um manipulador de eventos XPC está em execução**. Embora `_xpc_connection_set_creds` use bloqueio, isso apenas impede a sobrescrita parcial do token de auditoria, não bloqueia o objeto de conexão inteiro, tornando possível **substituir o token de auditoria entre a análise** de um pacote e a execução de seu manipulador de eventos.

Para este cenário, precisaríamos:

* Como antes, dois serviços mach _A_ e _B_ aos quais podemos nos conectar.
* Novamente, _A_ deve ter uma verificação de autorização para uma ação específica que _B_ pode passar (mas nosso aplicativo não pode).
* _A_ nos envia uma mensagem que espera uma resposta.
* Podemos enviar uma mensagem para _B_ que ele responderá.

Esperamos que _A_ nos envie uma mensagem que espera uma resposta (1), em vez de responder, pegamos a porta de resposta e a usamos para uma mensagem que enviamos para _B_ (2). Então, enviamos uma mensagem que usa a ação proibida e esperamos que ela chegue simultaneamente com a resposta de _B_ (3).

<figure><img src="../../../../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1).png" alt="" width="563"><figcaption></figcaption></figure>

## Problemas de Descoberta

Passamos muito tempo tentando encontrar outras instâncias, mas as condições tornaram difícil a busca tanto estática quanto dinamicamente. Para procurar chamadas assíncronas para `xpc_connection_get_audit_token`, usamos o Frida para enganchar nesta função para verificar se o backtrace inclui `_xpc_connection_mach_event` (o que significa que não é chamado de um manipulador de eventos). Mas isso só encontra chamadas no processo que atualmente enganchamos e das ações que estão ativamente sendo usadas. Analisar todos os serviços mach acessíveis no IDA/Ghidra foi muito demorado, especialmente quando chamadas envolviam o cache compartilhado dyld. Tentamos scriptar isso para procurar chamadas para `xpc_connection_get_audit_token` acessíveis a partir de um bloco submetido usando `dispatch_async`, mas analisar blocos e chamadas passando para o cache compartilhado dyld também foi difícil. Depois de passar um tempo nisso, decidimos que seria melhor submeter o que tínhamos.

## A correção <a href="#the-fix" id="the-fix"></a>

No final, relatamos o problema geral e o problema específico no `smd`. A Apple corrigiu apenas no `smd` substituindo a chamada para `xpc_connection_get_audit_token` por `xpc_dictionary_get_audit_token`.

A função `xpc_dictionary_get_audit_token` copia o token de auditoria da mensagem mach na qual esta mensagem XPC foi recebida, o que significa que não é vulnerável. No entanto, assim como `xpc_dictionary_get_audit_token`, isso não faz parte da API pública. Para a API `NSXPCConnection` de nível superior, não existe um método claro para obter o token de auditoria da mensagem atual, pois isso abstrai todas as mensagens em chamadas de método.

Não está claro para nós por que a Apple não aplicou uma correção mais geral, por exemplo, descartando mensagens que não correspondem ao token de auditoria salvo da conexão. Pode haver cenários em que o token de auditoria de um processo muda legitimamente, mas a conexão deve permanecer aberta (por exemplo, chamar `setuid` muda o campo UID), mas mudanças como um PID diferente ou versão do PID são improváveis de serem intencionais.

De qualquer forma, esse problema ainda permanece com o iOS 17 e o macOS 14, então se você quiser ir e procurar por ele, boa sorte!

<details>

<summary><strong>Aprenda hacking no AWS do zero ao herói com</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Outras formas de apoiar o HackTricks:

* Se você quer ver sua **empresa anunciada no HackTricks** ou **baixar o HackTricks em PDF**, confira os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Adquira o [**merchandising oficial do PEASS & HackTricks**](https://peass.creator-spring.com)
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção de [**NFTs**](https://opensea.io/collection/the-peass-family) exclusivos
* **Junte-se ao grupo do** 💬 [**Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo do telegram**](https://t.me/peass) ou **siga-me** no **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **Compartilhe suas técnicas de hacking enviando PRs para os repositórios do GitHub** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
