# DDexec / EverythingExec

<details>

<summary><strong>Aprenda hacking no AWS do zero ao herói com</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Outras formas de apoiar o HackTricks:

* Se você quer ver sua **empresa anunciada no HackTricks** ou **baixar o HackTricks em PDF**, confira os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Adquira o [**material oficial PEASS & HackTricks**](https://peass.creator-spring.com)
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção de [**NFTs**](https://opensea.io/collection/the-peass-family) exclusivos
* **Junte-se ao grupo** 💬 [**Discord**](https://discord.gg/hRep4RUj7f) ou ao grupo [**telegram**](https://t.me/peass) ou **siga-me** no **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **Compartilhe suas técnicas de hacking enviando PRs para os repositórios github do** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>

## Contexto

No Linux, para executar um programa, ele deve existir como um arquivo e deve ser acessível de alguma forma através da hierarquia do sistema de arquivos (é assim que o `execve()` funciona). Este arquivo pode residir no disco ou na RAM (tmpfs, memfd), mas você precisa de um caminho de arquivo. Isso tornou muito fácil controlar o que é executado em um sistema Linux, facilita a detecção de ameaças e ferramentas de atacantes ou até mesmo impedir que eles tentem executar qualquer coisa deles (por exemplo, não permitir que usuários não privilegiados coloquem arquivos executáveis em qualquer lugar).

Mas esta técnica está aqui para mudar tudo isso. Se você não pode iniciar o processo que deseja... **então você sequestra um já existente**.

Esta técnica permite que você **contorne técnicas comuns de proteção, como somente leitura, noexec, lista de permissões de nomes de arquivos, lista de permissões de hash...**

## Dependências

O script final depende das seguintes ferramentas para funcionar, elas precisam estar acessíveis no sistema que você está atacando (por padrão, você encontrará todas elas em todos os lugares):
```
dd
bash | zsh | ash (busybox)
head
tail
cut
grep
od
readlink
wc
tr
base64
```
## A técnica

Se você consegue modificar arbitrariamente a memória de um processo, então você pode assumir o controle dele. Isso pode ser usado para sequestrar um processo já existente e substituí-lo por outro programa. Podemos alcançar isso usando a syscall `ptrace()` (que exige que você tenha a capacidade de executar syscalls ou tenha o gdb disponível no sistema) ou, mais interessantemente, escrevendo em `/proc/$pid/mem`.

O arquivo `/proc/$pid/mem` é um mapeamento um-para-um de todo o espaço de endereçamento de um processo (_e. g._ de `0x0000000000000000` a `0x7ffffffffffff000` em x86-64). Isso significa que ler ou escrever neste arquivo em um deslocamento `x` é o mesmo que ler ou modificar o conteúdo no endereço virtual `x`.

Agora, temos quatro problemas básicos a enfrentar:

* Em geral, apenas o root e o proprietário do programa do arquivo podem modificá-lo.
* ASLR.
* Se tentarmos ler ou escrever em um endereço não mapeado no espaço de endereçamento do programa, receberemos um erro de E/S.

Esses problemas têm soluções que, embora não sejam perfeitas, são boas:

* A maioria dos interpretadores de shell permite a criação de descritores de arquivo que serão herdados por processos filhos. Podemos criar um fd apontando para o arquivo `mem` do shell com permissões de escrita... assim, processos filhos que usarem esse fd poderão modificar a memória do shell.
* ASLR nem é um problema, podemos verificar o arquivo `maps` do shell ou qualquer outro do procfs para obter informações sobre o espaço de endereçamento do processo.
* Então precisamos usar `lseek()` sobre o arquivo. Do shell, isso não pode ser feito a menos que se use o infame `dd`.

### Em mais detalhes

Os passos são relativamente fáceis e não requerem nenhum tipo de especialização para entendê-los:

* Analisar o binário que queremos executar e o carregador para descobrir quais mapeamentos eles precisam. Em seguida, criar um "shell"code que realizará, em linhas gerais, os mesmos passos que o kernel executa a cada chamada para `execve()`:
* Criar os mapeamentos mencionados.
* Ler os binários para dentro deles.
* Configurar as permissões.
* Finalmente inicializar a pilha com os argumentos para o programa e colocar o vetor auxiliar (necessário pelo carregador).
* Saltar para o carregador e deixá-lo fazer o resto (carregar as bibliotecas necessárias pelo programa).
* Obter do arquivo `syscall` o endereço para o qual o processo retornará após a syscall que está executando.
* Sobrescrever esse local, que será executável, com nosso shellcode (através de `mem` podemos modificar páginas não graváveis).
* Passar o programa que queremos executar para o stdin do processo (será `read()` pelo dito "shell"code).
* A partir deste ponto, cabe ao carregador carregar as bibliotecas necessárias para o nosso programa e saltar para ele.

**Confira a ferramenta em** [**https://github.com/arget13/DDexec**](https://github.com/arget13/DDexec)

## EverythingExec

Desde 12/12/2022, encontrei várias alternativas ao `dd`, uma das quais, `tail`, é atualmente o programa padrão usado para `lseek()` através do arquivo `mem` (que era o único propósito de usar `dd`). As alternativas mencionadas são:
```bash
tail
hexdump
cmp
xxd
```
Definindo a variável `SEEKER`, você pode alterar o seeker utilizado, _por exemplo_:
```bash
SEEKER=cmp bash ddexec.sh ls -l <<< $(base64 -w0 /bin/ls)
```
Se você encontrar outro seeker válido que não esteja implementado no script, ainda pode usá-lo definindo a variável `SEEKER_ARGS`:
```bash
SEEKER=xxd SEEKER_ARGS='-s $offset' zsh ddexec.sh ls -l <<< $(base64 -w0 /bin/ls)
```
Bloqueie isso, EDRs.

<details>

<summary><strong>Aprenda hacking no AWS do zero ao herói com</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Outras formas de apoiar o HackTricks:

* Se você quer ver sua **empresa anunciada no HackTricks** ou **baixar o HackTricks em PDF**, confira os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Adquira o [**material oficial PEASS & HackTricks**](https://peass.creator-spring.com)
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção de [**NFTs**](https://opensea.io/collection/the-peass-family) exclusivos
* **Junte-se ao grupo** 💬 [**Discord**](https://discord.gg/hRep4RUj7f) ou ao grupo [**telegram**](https://t.me/peass) ou **siga-me** no **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **Compartilhe suas técnicas de hacking enviando PRs para os repositórios github do** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
