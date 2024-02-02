# Namespace IPC

<details>

<summary><strong>Aprenda hacking no AWS do zero ao herói com</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Outras formas de apoiar o HackTricks:

* Se você quer ver sua **empresa anunciada no HackTricks** ou **baixar o HackTricks em PDF**, confira os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Adquira o [**material oficial PEASS & HackTricks**](https://peass.creator-spring.com)
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção de [**NFTs**](https://opensea.io/collection/the-peass-family) exclusivos
* **Participe do grupo** 💬 [**Discord**](https://discord.gg/hRep4RUj7f) ou do grupo [**telegram**](https://t.me/peass) ou **siga**-me no **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **Compartilhe suas técnicas de hacking enviando PRs para os repositórios github** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>

## Informações Básicas

Um namespace IPC (Comunicação Inter-Processo) é um recurso do kernel Linux que fornece **isolamento** de objetos IPC do System V, como filas de mensagens, segmentos de memória compartilhada e semáforos. Esse isolamento garante que processos em **diferentes namespaces IPC não possam acessar ou modificar diretamente os objetos IPC uns dos outros**, proporcionando uma camada adicional de segurança e privacidade entre grupos de processos.

### Como funciona:

1. Quando um novo namespace IPC é criado, ele começa com um **conjunto completamente isolado de objetos IPC do System V**. Isso significa que processos executando no novo namespace IPC não podem acessar ou interferir com os objetos IPC em outros namespaces ou no sistema hospedeiro por padrão.
2. Objetos IPC criados dentro de um namespace são visíveis e **acessíveis apenas para processos dentro desse namespace**. Cada objeto IPC é identificado por uma chave única dentro de seu namespace. Embora a chave possa ser idêntica em diferentes namespaces, os próprios objetos são isolados e não podem ser acessados entre namespaces.
3. Processos podem se mover entre namespaces usando a chamada de sistema `setns()` ou criar novos namespaces usando as chamadas de sistema `unshare()` ou `clone()` com a flag `CLONE_NEWIPC`. Quando um processo se move para um novo namespace ou cria um, ele começará a usar os objetos IPC associados com aquele namespace.

## Laboratório:

### Criar diferentes Namespaces

#### CLI
```bash
sudo unshare -i [--mount-proc] /bin/bash
```
Montando uma nova instância do sistema de arquivos `/proc` se você usar o parâmetro `--mount-proc`, você garante que o novo namespace de montagem tenha uma **visão precisa e isolada das informações de processo específicas para aquele namespace**.

<details>

<summary>Erro: bash: fork: Não é possível alocar memória</summary>

Se você executar a linha anterior sem `-f`, você receberá esse erro.\
O erro é causado pelo processo PID 1 que sai no novo namespace.

Após o início do bash, o bash criará vários novos sub-processos para fazer algumas coisas. Se você executar unshare sem -f, o bash terá o mesmo pid que o processo "unshare" atual. O processo "unshare" atual chama a chamada de sistema unshare, cria um novo namespace de pid, mas o processo "unshare" atual não está no novo namespace de pid. É o comportamento desejado do kernel Linux: o processo A cria um novo namespace, o próprio processo A não será colocado no novo namespace, apenas os sub-processos do processo A serão colocados no novo namespace. Então, quando você executa:
</details>
```
unshare -p /bin/bash
```
```markdown
O processo unshare executará /bin/bash, e /bin/bash gera vários sub-processos, o primeiro sub-processo do bash se tornará o PID 1 do novo namespace, e o subprocesso sairá após concluir seu trabalho. Assim, o PID 1 do novo namespace sai.

O processo PID 1 tem uma função especial: deve se tornar o processo pai de todos os processos órfãos. Se o processo PID 1 no namespace raiz sair, o kernel entrará em pânico. Se o processo PID 1 em um sub namespace sair, o kernel linux chamará a função disable_pid_allocation, que limpará a flag PIDNS_HASH_ADDING naquele namespace. Quando o kernel linux cria um novo processo, o kernel chamará a função alloc_pid para alocar um PID em um namespace, e se a flag PIDNS_HASH_ADDING não estiver definida, a função alloc_pid retornará um erro -ENOMEM. É por isso que você recebeu o erro "Cannot allocate memory".

Você pode resolver esse problema usando a opção '-f':
```
```
unshare -fp /bin/bash
```
```markdown
Se você executar unshare com a opção '-f', o unshare irá bifurcar um novo processo após criar o novo namespace de pid. E executar /bin/bash no novo processo. O novo processo será o pid 1 do novo namespace de pid. Então, o bash também irá bifurcar vários sub-processos para realizar algumas tarefas. Como o próprio bash é o pid 1 do novo namespace de pid, seus sub-processos podem sair sem nenhum problema.

Copiado de [https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory](https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory)

</details>

#### Docker
```
```bash
docker run -ti --name ubuntu1 -v /usr:/ubuntu1 ubuntu bash
```
### Verifique em qual namespace seu processo está
```bash
ls -l /proc/self/ns/ipc
lrwxrwxrwx 1 root root 0 Apr  4 20:37 /proc/self/ns/ipc -> 'ipc:[4026531839]'
```
### Encontrar todos os namespaces IPC

{% code overflow="wrap" %}
```bash
sudo find /proc -maxdepth 3 -type l -name ipc -exec readlink {} \; 2>/dev/null | sort -u
# Find the processes with an specific namespace
sudo find /proc -maxdepth 3 -type l -name ipc -exec ls -l  {} \; 2>/dev/null | grep <ns-number>
```
### Entrar em um namespace IPC
```bash
nsenter -i TARGET_PID --pid /bin/bash
```
Também, você só pode **entrar no namespace de outro processo se for root**. E você **não pode** **entrar** em outro namespace **sem um descritor** apontando para ele (como `/proc/self/ns/net`).

### Criar objeto IPC
```bash
# Container
sudo unshare -i /bin/bash
ipcmk -M 100
Shared memory id: 0
ipcs -m

------ Shared Memory Segments --------
key        shmid      owner      perms      bytes      nattch     status
0x2fba9021 0          root       644        100        0

# From the host
ipcs -m # Nothing is seen
```
<details>

<summary><strong>Aprenda hacking no AWS do zero ao herói com</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Outras formas de apoiar o HackTricks:

* Se você quer ver sua **empresa anunciada no HackTricks** ou **baixar o HackTricks em PDF**, confira os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Adquira o [**material oficial PEASS & HackTricks**](https://peass.creator-spring.com)
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção de [**NFTs**](https://opensea.io/collection/the-peass-family) exclusivos
* **Junte-se ao grupo** 💬 [**Discord**](https://discord.gg/hRep4RUj7f) ou ao grupo [**telegram**](https://t.me/peass) ou **siga**-me no **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **Compartilhe suas técnicas de hacking enviando PRs para os repositórios github** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
