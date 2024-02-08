# Restrições de Inicialização/Ambiente do macOS & Cache de Confiança

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* Você trabalha em uma **empresa de cibersegurança**? Gostaria de ver sua **empresa anunciada no HackTricks**? ou gostaria de ter acesso à **última versão do PEASS ou baixar o HackTricks em PDF**? Confira os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* Adquira o [**swag oficial PEASS & HackTricks**](https://peass.creator-spring.com)
* **Junte-se ao** [**💬**](https://emojipedia.org/speech-balloon/) [**grupo Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo telegram**](https://t.me/peass) ou **siga-me** no **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Compartilhe seus truques de hacking enviando PRs para o** [**repositório hacktricks**](https://github.com/carlospolop/hacktricks) **e** [**repositório hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud)
*
* .

</details>

## Informações Básicas

As restrições de inicialização no macOS foram introduzidas para aprimorar a segurança ao **regulamentar como, quem e de onde um processo pode ser iniciado**. Iniciadas no macOS Ventura, elas fornecem um framework que categoriza **cada binário do sistema em categorias de restrição distintas**, definidas dentro do **cache de confiança**, uma lista contendo binários do sistema e seus respectivos hashes. Essas restrições se estendem a cada binário executável dentro do sistema, envolvendo um conjunto de **regras** que delineiam os requisitos para **iniciar um determinado binário**. As regras abrangem restrições próprias que um binário deve satisfazer, restrições dos pais que devem ser atendidas pelo processo pai e restrições responsáveis a serem seguidas por outras entidades relevantes.

O mecanismo se estende a aplicativos de terceiros por meio de **Restrições de Ambiente**, iniciando no macOS Sonoma, permitindo que os desenvolvedores protejam seus aplicativos especificando um **conjunto de chaves e valores para restrições de ambiente**.

Você define **restrições de inicialização e biblioteca de ambiente** em dicionários de restrição que você salva em **arquivos de lista de propriedades `launchd`**, ou em **arquivos de lista de propriedades separados** que você usa na assinatura de código.

Existem 4 tipos de restrições:

* **Restrições Próprias**: Restrições aplicadas ao binário **em execução**.
* **Processo Pai**: Restrições aplicadas ao **processo pai do processo** (por exemplo, **`launchd`** executando um serviço XP).
* **Restrições Responsáveis**: Restrições aplicadas ao **processo que chama o serviço** em uma comunicação XPC.
* **Restrições de Carregamento de Biblioteca**: Use restrições de carregamento de biblioteca para descrever seletivamente o código que pode ser carregado.

Portanto, quando um processo tenta iniciar outro processo — chamando `execve(_:_:_:)` ou `posix_spawn(_:_:_:_:_:_:)` — o sistema operacional verifica se o **arquivo executável** satisfaz sua **própria restrição**, verifica se o **arquivo executável do processo pai** satisfaz a **restrição do pai do executável** e se o **arquivo executável do processo responsável** satisfaz a **restrição do processo responsável do executável**. Se alguma dessas restrições de inicialização não for atendida, o sistema operacional não executa o programa.

Se ao carregar uma biblioteca qualquer parte da **restrição da biblioteca não for verdadeira**, seu processo **não carrega** a biblioteca.

## Categorias LC

Um LC é composto por **fatos** e **operações lógicas** (e, ou..) que combinam fatos.

Os [**fatos que um LC pode usar são documentados**](https://developer.apple.com/documentation/security/defining\_launch\_environment\_and\_library\_constraints). Por exemplo:

* is-init-proc: Um valor booleano que indica se o executável deve ser o processo de inicialização do sistema operacional (`launchd`).
* is-sip-protected: Um valor booleano que indica se o executável deve ser um arquivo protegido pelo Sistema de Integridade do Sistema (SIP).
* `on-authorized-authapfs-volume:` Um valor booleano que indica se o sistema operacional carregou o executável de um volume APFS autorizado e autenticado.
* `on-authorized-authapfs-volume`: Um valor booleano que indica se o sistema operacional carregou o executável de um volume APFS autorizado e autenticado.
* Volume Cryptexes
* `on-system-volume:` Um valor booleano que indica se o sistema operacional carregou o executável do volume do sistema atualmente inicializado.
* Dentro de /System...
* ...

Quando um binário da Apple é assinado, ele **é atribuído a uma categoria LC** dentro do **cache de confiança**.

* As **16 categorias LC do iOS** foram [**revertidas e documentadas aqui**](https://gist.github.com/LinusHenze/4cd5d7ef057a144cda7234e2c247c056).
* As atuais **categorias LC (macOS 14** - Somona) foram revertidas e suas [**descrições podem ser encontradas aqui**](https://gist.github.com/theevilbit/a6fef1e0397425a334d064f7b6e1be53).

Por exemplo, a Categoria 1 é:
```
Category 1:
Self Constraint: (on-authorized-authapfs-volume || on-system-volume) && launch-type == 1 && validation-category == 1
Parent Constraint: is-init-proc
```
* `(on-authorized-authapfs-volume || on-system-volume)`: Deve estar no volume do Sistema ou Cryptexes.
* `launch-type == 1`: Deve ser um serviço do sistema (plist em LaunchDaemons).
* `validation-category == 1`: Um executável do sistema operacional.
* `is-init-proc`: Launchd

### Reversão das Categorias LC

Você tem mais informações [**sobre isso aqui**](https://theevilbit.github.io/posts/launch\_constraints\_deep\_dive/#reversing-constraints), mas basicamente, eles são definidos em **AMFI (AppleMobileFileIntegrity)**, então você precisa baixar o Kit de Desenvolvimento do Kernel para obter o **KEXT**. Os símbolos que começam com **`kConstraintCategory`** são os **interessantes**. Extraindo-os, você obterá um fluxo codificado DER (ASN.1) que precisará ser decodificado com [ASN.1 Decoder](https://holtstrom.com/michael/tools/asn1decoder.php) ou a biblioteca python-asn1 e seu script `dump.py`, [andrivet/python-asn1](https://github.com/andrivet/python-asn1/tree/master) que lhe dará uma string mais compreensível.

## Restrições de Ambiente

Essas são as Restrições de Lançamento configuradas em **aplicativos de terceiros**. O desenvolvedor pode selecionar os **fatos** e **operandos lógicos a serem usados** em seu aplicativo para restringir o acesso a si mesmo.

É possível enumerar as Restrições de Ambiente de um aplicativo com:
```bash
codesign -d -vvvv app.app
```
## Caches de Confiança

No **macOS**, existem algumas caches de confiança:

- **`/System/Volumes/Preboot/*/boot/*/usr/standalone/firmware/FUD/BaseSystemTrustCache.img4`**
- **`/System/Volumes/Preboot/*/boot/*/usr/standalone/firmware/FUD/StaticTrustCache.img4`**
- **`/System/Library/Security/OSLaunchPolicyData`**

E no iOS parece estar em **`/usr/standalone/firmware/FUD/StaticTrustCache.img4`**.

{% hint style="warning" %}
No macOS em dispositivos Apple Silicon, se um binário assinado pela Apple não estiver na cache de confiança, o AMFI se recusará a carregá-lo.
{% endhint %}

### Enumerando Caches de Confiança

Os arquivos de cache de confiança anteriores estão no formato **IMG4** e **IM4P**, sendo o IM4P a seção de carga útil de um formato IMG4.

Você pode usar [**pyimg4**](https://github.com/m1stadev/PyIMG4) para extrair a carga útil dos bancos de dados:

{% code overflow="wrap" %}
```bash
# Installation
python3 -m pip install pyimg4

# Extract payloads data
cp /System/Volumes/Preboot/*/boot/*/usr/standalone/firmware/FUD/BaseSystemTrustCache.img4 /tmp
pyimg4 img4 extract -i /tmp/BaseSystemTrustCache.img4 -p /tmp/BaseSystemTrustCache.im4p
pyimg4 im4p extract -i /tmp/BaseSystemTrustCache.im4p -o /tmp/BaseSystemTrustCache.data

cp /System/Volumes/Preboot/*/boot/*/usr/standalone/firmware/FUD/StaticTrustCache.img4 /tmp
pyimg4 img4 extract -i /tmp/StaticTrustCache.img4 -p /tmp/StaticTrustCache.im4p
pyimg4 im4p extract -i /tmp/StaticTrustCache.im4p -o /tmp/StaticTrustCache.data

pyimg4 im4p extract -i /System/Library/Security/OSLaunchPolicyData -o /tmp/OSLaunchPolicyData.data
```
(Outra opção poderia ser usar a ferramenta [**img4tool**](https://github.com/tihmstar/img4tool), que funcionará até mesmo no M1, mesmo que a versão seja antiga e para x86\_64 se você a instalar nos locais apropriados).

Agora você pode usar a ferramenta [**trustcache**](https://github.com/CRKatri/trustcache) para obter as informações em um formato legível:
```bash
# Install
wget https://github.com/CRKatri/trustcache/releases/download/v2.0/trustcache_macos_arm64
sudo mv ./trustcache_macos_arm64 /usr/local/bin/trustcache
xattr -rc /usr/local/bin/trustcache
chmod +x /usr/local/bin/trustcache

# Run
trustcache info /tmp/OSLaunchPolicyData.data | head
trustcache info /tmp/StaticTrustCache.data | head
trustcache info /tmp/BaseSystemTrustCache.data | head

version = 2
uuid = 35EB5284-FD1E-4A5A-9EFB-4F79402BA6C0
entry count = 969
0065fc3204c9f0765049b82022e4aa5b44f3a9c8 [none] [2] [1]
00aab02b28f99a5da9b267910177c09a9bf488a2 [none] [2] [1]
0186a480beeee93050c6c4699520706729b63eff [none] [2] [2]
0191be4c08426793ff3658ee59138e70441fc98a [none] [2] [3]
01b57a71112235fc6241194058cea5c2c7be3eb1 [none] [2] [2]
01e6934cb8833314ea29640c3f633d740fc187f2 [none] [2] [2]
020bf8c388deaef2740d98223f3d2238b08bab56 [none] [2] [3]
```
O cache de confiança segue a seguinte estrutura, então a **categoria LC é a 4ª coluna**.
```c
struct trust_cache_entry2 {
uint8_t cdhash[CS_CDHASH_LEN];
uint8_t hash_type;
uint8_t flags;
uint8_t constraintCategory;
uint8_t reserved0;
} __attribute__((__packed__));
```
Em seguida, você poderia usar um script como [**este**](https://gist.github.com/xpn/66dc3597acd48a4c31f5f77c3cc62f30) para extrair dados.

A partir desses dados, você pode verificar os Apps com um valor de **restrições de lançamento de `0`**, que são aqueles que não estão restritos ([**verifique aqui**](https://gist.github.com/LinusHenze/4cd5d7ef057a144cda7234e2c247c056) para saber o que cada valor representa).

## Mitigações de Ataque

As Restrições de Lançamento teriam mitigado vários ataques antigos ao **garantir que o processo não seja executado em condições inesperadas:** Por exemplo, de locais inesperados ou sendo invocado por um processo pai inesperado (se apenas o launchd deveria iniciá-lo)

Além disso, as Restrições de Lançamento também **mitigam ataques de degradação**.

No entanto, elas **não mitigam abusos comuns de XPC**, injeções de código **Electron** ou **injeções de dylib** sem validação de biblioteca (a menos que os IDs de equipe que podem carregar bibliotecas sejam conhecidos).

### Proteção do Daemon XPC

No lançamento Sonoma, um ponto notável é a **configuração de responsabilidade do serviço XPC do daemon**. O serviço XPC é responsável por si mesmo, ao contrário do cliente conectado ser responsável. Isso está documentado no relatório de feedback FB13206884. Essa configuração pode parecer falha, pois permite certas interações com o serviço XPC:

- **Iniciando o Serviço XPC**: Se for considerado um bug, essa configuração não permite iniciar o serviço XPC por meio de código de atacante.
- **Conectando a um Serviço Ativo**: Se o serviço XPC já estiver em execução (possivelmente ativado por seu aplicativo original), não há barreiras para se conectar a ele.

Embora a implementação de restrições no serviço XPC possa ser benéfica ao **reduzir a janela para ataques potenciais**, isso não aborda a preocupação principal. Garantir a segurança do serviço XPC requer fundamentalmente **validar efetivamente o cliente conectado**. Este ainda é o único método para fortalecer a segurança do serviço. Além disso, vale ressaltar que a configuração de responsabilidade mencionada está atualmente operacional, o que pode não estar alinhado com o design pretendido.


### Proteção do Electron

Mesmo que seja necessário que o aplicativo seja **aberto pelo LaunchService** (nas restrições dos pais). Isso pode ser alcançado usando **`open`** (que pode definir variáveis de ambiente) ou usando a **API de Serviços de Lançamento** (onde as variáveis de ambiente podem ser indicadas).

## Referências

* [https://youtu.be/f1HA5QhLQ7Y?t=24146](https://youtu.be/f1HA5QhLQ7Y?t=24146)
* [https://theevilbit.github.io/posts/launch\_constraints\_deep\_dive/](https://theevilbit.github.io/posts/launch\_constraints\_deep\_dive/)
* [https://eclecticlight.co/2023/06/13/why-wont-a-system-app-or-command-tool-run-launch-constraints-and-trust-caches/](https://eclecticlight.co/2023/06/13/why-wont-a-system-app-or-command-tool-run-launch-constraints-and-trust-caches/)
* [https://developer.apple.com/videos/play/wwdc2023/10266/](https://developer.apple.com/videos/play/wwdc2023/10266/)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* Você trabalha em uma **empresa de cibersegurança**? Gostaria de ver sua **empresa anunciada no HackTricks**? ou gostaria de ter acesso à **última versão do PEASS ou baixar o HackTricks em PDF**? Confira os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* Adquira o [**swag oficial do PEASS & HackTricks**](https://peass.creator-spring.com)
* **Junte-se ao** [**💬**](https://emojipedia.org/speech-balloon/) [**grupo Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo telegram**](https://t.me/peass) ou **siga-me** no **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Compartilhe seus truques de hacking enviando PRs para o** [**repositório hacktricks**](https://github.com/carlospolop/hacktricks) **e** [**repositório hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud)
*
* .

</details>
