# Dump de Memória do macOS

<details>

<summary><strong>Aprenda hacking AWS do zero ao herói com</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Outras formas de apoiar o HackTricks:

* Se você deseja ver sua **empresa anunciada no HackTricks** ou **baixar o HackTricks em PDF** Confira os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Adquira o [**swag oficial PEASS & HackTricks**](https://peass.creator-spring.com)
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Junte-se ao** 💬 [**grupo Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo telegram**](https://t.me/peass) ou **siga-nos** no **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Compartilhe seus truques de hacking enviando PRs para o** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repositórios do github.

</details>

### [WhiteIntel](https://whiteintel.io)

<figure><img src="../../../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) é um mecanismo de busca alimentado pela **dark web** que oferece funcionalidades **gratuitas** para verificar se uma empresa ou seus clientes foram **comprometidos** por **malwares de roubo**.

O principal objetivo do WhiteIntel é combater tomadas de conta e ataques de ransomware resultantes de malwares de roubo de informações.

Você pode verificar o site deles e experimentar o mecanismo gratuitamente em:

{% embed url="https://whiteintel.io" %}

***

## Artefatos de Memória

### Arquivos de Troca

Arquivos de troca, como `/private/var/vm/swapfile0`, servem como **caches quando a memória física está cheia**. Quando não há mais espaço na memória física, seus dados são transferidos para um arquivo de troca e depois trazidos de volta para a memória física conforme necessário. Vários arquivos de troca podem estar presentes, com nomes como swapfile0, swapfile1, e assim por diante.

### Imagem de Hibernação

O arquivo localizado em `/private/var/vm/sleepimage` é crucial durante o **modo de hibernação**. **Dados da memória são armazenados neste arquivo quando o OS X hiberna**. Ao acordar o computador, o sistema recupera os dados da memória deste arquivo, permitindo que o usuário continue de onde parou.

Vale ressaltar que em sistemas MacOS modernos, este arquivo é tipicamente criptografado por razões de segurança, tornando a recuperação difícil.

* Para verificar se a criptografia está ativada para o sleepimage, o comando `sysctl vm.swapusage` pode ser executado. Isso mostrará se o arquivo está criptografado.

### Logs de Pressão de Memória

Outro arquivo importante relacionado à memória em sistemas MacOS é o **log de pressão de memória**. Esses logs estão localizados em `/var/log` e contêm informações detalhadas sobre o uso de memória do sistema e eventos de pressão. Eles podem ser particularmente úteis para diagnosticar problemas relacionados à memória ou entender como o sistema gerencia a memória ao longo do tempo.

## Dump de memória com osxpmem

Para fazer o dump de memória em uma máquina MacOS, você pode usar [**osxpmem**](https://github.com/google/rekall/releases/download/v1.5.1/osxpmem-2.1.post4.zip).

**Nota**: As instruções a seguir funcionarão apenas para Macs com arquitetura Intel. Esta ferramenta está arquivada e o último lançamento foi em 2017. O binário baixado usando as instruções abaixo tem como alvo chips Intel, já que o Apple Silicon não existia em 2017. Pode ser possível compilar o binário para a arquitetura arm64, mas você terá que tentar por conta própria.
```bash
#Dump raw format
sudo osxpmem.app/osxpmem --format raw -o /tmp/dump_mem

#Dump aff4 format
sudo osxpmem.app/osxpmem -o /tmp/dump_mem.aff4
```
Se encontrar este erro: `osxpmem.app/MacPmem.kext falhou ao carregar - (libkern/kext) falha de autenticação (propriedade de arquivo/permissões); verifique os logs do sistema/kernel para erros ou tente kextutil(8)` Você pode corrigir fazendo:
```bash
sudo cp -r osxpmem.app/MacPmem.kext "/tmp/"
sudo kextutil "/tmp/MacPmem.kext"
#Allow the kext in "Security & Privacy --> General"
sudo osxpmem.app/osxpmem --format raw -o /tmp/dump_mem
```
**Outros erros** podem ser corrigidos permitindo o carregamento do kext em "Segurança e Privacidade --> Geral", apenas permita.

Você também pode usar este **oneliner** para baixar o aplicativo, carregar o kext e fazer dump da memória:

{% code overflow="wrap" %}
```bash
sudo su
cd /tmp; wget https://github.com/google/rekall/releases/download/v1.5.1/osxpmem-2.1.post4.zip; unzip osxpmem-2.1.post4.zip; chown -R root:wheel osxpmem.app/MacPmem.kext; kextload osxpmem.app/MacPmem.kext; osxpmem.app/osxpmem --format raw -o /tmp/dump_mem
```
{% endcode %}

### [WhiteIntel](https://whiteintel.io)

<figure><img src="../../../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) é um mecanismo de busca alimentado pela **dark web** que oferece funcionalidades **gratuitas** para verificar se uma empresa ou seus clientes foram **comprometidos** por **malwares stealers**.

O principal objetivo do WhiteIntel é combater a apropriação de contas e ataques de ransomware resultantes de malwares que roubam informações.

Você pode acessar o site deles e experimentar o mecanismo gratuitamente em:

{% embed url="https://whiteintel.io" %}

<details>

<summary><strong>Aprenda hacking AWS do zero ao herói com</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Outras maneiras de apoiar o HackTricks:

* Se você deseja ver sua **empresa anunciada no HackTricks** ou **baixar o HackTricks em PDF**, verifique os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Adquira o [**swag oficial PEASS & HackTricks**](https://peass.creator-spring.com)
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Junte-se ao** 💬 [**grupo Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo telegram**](https://t.me/peass) ou **siga-nos** no **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Compartilhe seus truques de hacking enviando PRs para os** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repositórios do github.

</details>
