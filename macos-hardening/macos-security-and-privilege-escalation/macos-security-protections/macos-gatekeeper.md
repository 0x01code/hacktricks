# macOS Gatekeeper / Quarentena / XProtect

<details>

<summary><strong>Aprenda hacking AWS do zero ao herói com</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Você trabalha em uma **empresa de cibersegurança**? Gostaria de ver sua **empresa anunciada no HackTricks**? ou gostaria de ter acesso à **última versão do PEASS ou baixar o HackTricks em PDF**? Confira os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* Adquira o [**oficial PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Junte-se ao** [**💬**](https://emojipedia.org/speech-balloon/) [**grupo Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo telegram**](https://t.me/peass) ou **siga-me** no **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Compartilhe seus truques de hacking enviando PRs para o** [**repositório hacktricks**](https://github.com/carlospolop/hacktricks) **e** [**repositório hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud)
*
* .

</details>

## Gatekeeper

**Gatekeeper** é um recurso de segurança desenvolvido para sistemas operacionais Mac, projetado para garantir que os usuários **executem apenas software confiável** em seus sistemas. Ele funciona **validando o software** que um usuário baixa e tenta abrir de **fontes fora da App Store**, como um aplicativo, um plug-in ou um pacote de instalação.

O mecanismo chave do Gatekeeper reside em seu processo de **verificação**. Ele verifica se o software baixado está **assinado por um desenvolvedor reconhecido**, garantindo a autenticidade do software. Além disso, ele verifica se o software está **notarizado pela Apple**, confirmando que está livre de conteúdo malicioso conhecido e não foi adulterado após a notarização.

Além disso, o Gatekeeper reforça o controle e a segurança do usuário ao **solicitar a aprovação da abertura** do software baixado pela primeira vez. Essa salvaguarda ajuda a evitar que os usuários executem inadvertidamente código executável potencialmente prejudicial que possam ter confundido com um arquivo de dados inofensivo.

### Assinaturas de Aplicativos

As assinaturas de aplicativos, também conhecidas como assinaturas de código, são um componente crítico da infraestrutura de segurança da Apple. Elas são usadas para **verificar a identidade do autor do software** (o desenvolvedor) e garantir que o código não tenha sido adulterado desde a última assinatura.

Veja como funciona:

1. **Assinando o Aplicativo:** Quando um desenvolvedor está pronto para distribuir seu aplicativo, ele **assina o aplicativo usando uma chave privada**. Essa chave privada está associada a um **certificado que a Apple emite para o desenvolvedor** quando ele se inscreve no Programa de Desenvolvedor da Apple. O processo de assinatura envolve a criação de um hash criptográfico de todas as partes do aplicativo e a criptografia desse hash com a chave privada do desenvolvedor.
2. **Distribuindo o Aplicativo:** O aplicativo assinado é então distribuído aos usuários juntamente com o certificado do desenvolvedor, que contém a chave pública correspondente.
3. **Verificando o Aplicativo:** Quando um usuário baixa e tenta executar o aplicativo, seu sistema operacional Mac usa a chave pública do certificado do desenvolvedor para descriptografar o hash. Em seguida, recalcula o hash com base no estado atual do aplicativo e compara isso com o hash descriptografado. Se coincidirem, significa que **o aplicativo não foi modificado** desde que o desenvolvedor o assinou, e o sistema permite a execução do aplicativo.

As assinaturas de aplicativos são uma parte essencial da tecnologia Gatekeeper da Apple. Quando um usuário tenta **abrir um aplicativo baixado da internet**, o Gatekeeper verifica a assinatura do aplicativo. Se estiver assinado com um certificado emitido pela Apple para um desenvolvedor conhecido e o código não foi adulterado, o Gatekeeper permite a execução do aplicativo. Caso contrário, bloqueia o aplicativo e alerta o usuário.

A partir do macOS Catalina, **o Gatekeeper também verifica se o aplicativo foi notarizado** pela Apple, adicionando uma camada extra de segurança. O processo de notarização verifica o aplicativo em busca de problemas de segurança conhecidos e código malicioso, e se essas verificações forem aprovadas, a Apple adiciona um ticket ao aplicativo que o Gatekeeper pode verificar.

#### Verificar Assinaturas

Ao verificar algum **exemplo de malware**, você sempre deve **verificar a assinatura** do binário, pois o **desenvolvedor** que o assinou pode estar **relacionado** a **malware**.
```bash
# Get signer
codesign -vv -d /bin/ls 2>&1 | grep -E "Authority|TeamIdentifier"

# Check if the app’s contents have been modified
codesign --verify --verbose /Applications/Safari.app

# Get entitlements from the binary
codesign -d --entitlements :- /System/Applications/Automator.app # Check the TCC perms

# Check if the signature is valid
spctl --assess --verbose /Applications/Safari.app

# Sign a binary
codesign -s <cert-name-keychain> toolsdemo
```
### Notarização

O processo de notarização da Apple serve como uma salvaguarda adicional para proteger os usuários de software potencialmente prejudicial. Envolve o **desenvolvedor submeter sua aplicação para exame** pelo **Serviço de Notarização da Apple**, que não deve ser confundido com a Revisão de Aplicativos. Este serviço é um **sistema automatizado** que examina o software enviado em busca de **conteúdo malicioso** e possíveis problemas com a assinatura de código.

Se o software **passar** por essa inspeção sem levantar preocupações, o Serviço de Notarização gera um ticket de notarização. O desenvolvedor é então obrigado a **anexar este ticket ao seu software**, um processo conhecido como 'grampeamento'. Além disso, o ticket de notarização também é publicado online, onde o Gatekeeper, a tecnologia de segurança da Apple, pode acessá-lo.

Na primeira instalação ou execução do software pelo usuário, a existência do ticket de notarização - seja grampeado ao executável ou encontrado online - **informa ao Gatekeeper que o software foi notarizado pela Apple**. Como resultado, o Gatekeeper exibe uma mensagem descritiva no diálogo de lançamento inicial, indicando que o software passou por verificações de conteúdo malicioso pela Apple. Esse processo, portanto, aumenta a confiança do usuário na segurança do software que eles instalam ou executam em seus sistemas.

### Enumerando o Gatekeeper

O Gatekeeper é tanto **vários componentes de segurança** que impedem a execução de aplicativos não confiáveis quanto **um dos componentes**.

É possível ver o **status** do Gatekeeper com:
```bash
# Check the status
spctl --status
```
{% hint style="danger" %}
Note que as verificações de assinatura do GateKeeper são realizadas apenas em **arquivos com o atributo de Quarentena**, e não em todos os arquivos.
{% endhint %}

O GateKeeper verificará se, de acordo com as **preferências e a assinatura**, um binário pode ser executado:

<figure><img src="../../../.gitbook/assets/image (678).png" alt=""><figcaption></figcaption></figure>

O banco de dados que mantém essa configuração está localizado em **`/var/db/SystemPolicy`**. Você pode verificar este banco de dados como root com:
```bash
# Open database
sqlite3 /var/db/SystemPolicy

# Get allowed rules
SELECT requirement,allow,disabled,label from authority where label != 'GKE' and disabled=0;
requirement|allow|disabled|label
anchor apple generic and certificate 1[subject.CN] = "Apple Software Update Certification Authority"|1|0|Apple Installer
anchor apple|1|0|Apple System
anchor apple generic and certificate leaf[field.1.2.840.113635.100.6.1.9] exists|1|0|Mac App Store
anchor apple generic and certificate 1[field.1.2.840.113635.100.6.2.6] exists and (certificate leaf[field.1.2.840.113635.100.6.1.14] or certificate leaf[field.1.2.840.113635.100.6.1.13]) and notarized|1|0|Notarized Developer ID
[...]
```
Observe como a primeira regra terminou em "**App Store**" e a segunda em "**Developer ID**" e que na imagem anterior estava **habilitado para executar aplicativos da App Store e desenvolvedores identificados**. Se você **modificar** essa configuração para App Store, as regras de "**Notarized Developer ID" desaparecerão**.

Também existem milhares de regras do **tipo GKE**:
```bash
SELECT requirement,allow,disabled,label from authority where label = 'GKE' limit 5;
cdhash H"b40281d347dc574ae0850682f0fd1173aa2d0a39"|1|0|GKE
cdhash H"5fd63f5342ac0c7c0774ebcbecaf8787367c480f"|1|0|GKE
cdhash H"4317047eefac8125ce4d44cab0eb7b1dff29d19a"|1|0|GKE
cdhash H"0a71962e7a32f0c2b41ddb1fb8403f3420e1d861"|1|0|GKE
cdhash H"8d0d90ff23c3071211646c4c9c607cdb601cb18f"|1|0|GKE
```
Estes são hashes que vêm de **`/var/db/SystemPolicyConfiguration/gke.bundle/Contents/Resources/gke.auth`, `/var/db/gke.bundle/Contents/Resources/gk.db`** e **`/var/db/gkopaque.bundle/Contents/Resources/gkopaque.db`**

Ou você poderia listar as informações anteriores com:
```bash
sudo spctl --list
```
As opções **`--master-disable`** e **`--global-disable`** do **`spctl`** irão **desativar completamente** essas verificações de assinatura:
```bash
# Disable GateKeeper
spctl --global-disable
spctl --master-disable

# Enable it
spctl --global-enable
spctl --master-enable
```
Quando totalmente habilitado, uma nova opção aparecerá:

<figure><img src="../../../.gitbook/assets/image (679).png" alt=""><figcaption></figcaption></figure>

É possível **verificar se um aplicativo será permitido pelo GateKeeper** com:
```bash
spctl --assess -v /Applications/App.app
```
É possível adicionar novas regras no GateKeeper para permitir a execução de determinados aplicativos com:
```bash
# Check if allowed - nop
spctl --assess -v /Applications/App.app
/Applications/App.app: rejected
source=no usable signature

# Add a label and allow this label in GateKeeper
sudo spctl --add --label "whitelist" /Applications/App.app
sudo spctl --enable --label "whitelist"

# Check again - yep
spctl --assess -v /Applications/App.app
/Applications/App.app: accepted
```
### Arquivos em Quarentena

Após **baixar** um aplicativo ou arquivo, **aplicativos específicos do macOS** como navegadores da web ou clientes de e-mail **anexam um atributo de arquivo estendido**, comumente conhecido como "**flag de quarentena**," ao arquivo baixado. Este atributo atua como uma medida de segurança para **marcar o arquivo** como proveniente de uma fonte não confiável (a internet) e potencialmente portando riscos. No entanto, nem todos os aplicativos anexam esse atributo, por exemplo, softwares comuns de cliente BitTorrent geralmente ignoram esse processo.

**A presença de uma flag de quarentena sinaliza o recurso de segurança Gatekeeper do macOS quando um usuário tenta executar o arquivo**.

No caso em que a **flag de quarentena não está presente** (como nos arquivos baixados por alguns clientes BitTorrent), as **verificações do Gatekeeper podem não ser realizadas**. Portanto, os usuários devem ter cautela ao abrir arquivos baixados de fontes menos seguras ou desconhecidas.

{% hint style="info" %}
**Verificar** a **validade** das assinaturas de código é um processo **intensivo em recursos** que inclui a geração de **hashes** criptográficos do código e de todos os seus recursos agrupados. Além disso, verificar a validade do certificado envolve fazer uma **verificação online** nos servidores da Apple para ver se ele foi revogado após ter sido emitido. Por esses motivos, uma verificação completa de assinatura de código e notarização é **impraticável de ser executada toda vez que um aplicativo é lançado**.

Portanto, essas verificações são **executadas apenas ao executar aplicativos com o atributo de quarentena**.
{% endhint %}

{% hint style="warning" %}
Este atributo deve ser **definido pelo aplicativo que cria/baixa** o arquivo.

No entanto, arquivos que estão isolados terão esse atributo definido para cada arquivo que criam. E aplicativos não isolados podem defini-lo por si próprios, ou especificar a chave [**LSFileQuarantineEnabled**](https://developer.apple.com/documentation/bundleresources/information\_property\_list/lsfilequarantineenabled?language=objc) no **Info.plist** que fará o sistema definir o atributo estendido `com.apple.quarantine` nos arquivos criados.
{% endhint %}

É possível **verificar seu status e habilitar/desabilitar** (necessário acesso de root) com:
```bash
spctl --status
assessments enabled

spctl --enable
spctl --disable
#You can also allow nee identifies to execute code using the binary "spctl"
```
Você também pode **verificar se um arquivo possui o atributo de quarentena estendida** com:
```bash
xattr file.png
com.apple.macl
com.apple.quarantine
```
Verifique o **valor** dos **atributos** **estendidos** e descubra o aplicativo que escreveu o atributo de quarentena com:
```bash
xattr -l portada.png
com.apple.macl:
00000000  03 00 53 DA 55 1B AE 4C 4E 88 9D CA B7 5C 50 F3  |..S.U..LN.....P.|
00000010  16 94 03 00 27 63 64 97 98 FB 4F 02 84 F3 D0 DB  |....'cd...O.....|
00000020  89 53 C3 FC 03 00 27 63 64 97 98 FB 4F 02 84 F3  |.S....'cd...O...|
00000030  D0 DB 89 53 C3 FC 00 00 00 00 00 00 00 00 00 00  |...S............|
00000040  00 00 00 00 00 00 00 00                          |........|
00000048
com.apple.quarantine: 00C1;607842eb;Brave;F643CD5F-6071-46AB-83AB-390BA944DEC5
# 00c1 -- It has been allowed to eexcute this file (QTN_FLAG_USER_APPROVED = 0x0040)
# 607842eb -- Timestamp
# Brave -- App
# F643CD5F-6071-46AB-83AB-390BA944DEC5 -- UID assigned to the file downloaded
```
Na verdade, um processo "poderia definir flags de quarentena para os arquivos que cria" (tentei aplicar a flag USER\_APPROVED em um arquivo criado, mas não a aplicou):

<details>

<summary>Código Fonte aplicar flags de quarentena</summary>
```c
#include <stdio.h>
#include <stdlib.h>

enum qtn_flags {
QTN_FLAG_DOWNLOAD = 0x0001,
QTN_FLAG_SANDBOX = 0x0002,
QTN_FLAG_HARD = 0x0004,
QTN_FLAG_USER_APPROVED = 0x0040,
};

#define qtn_proc_alloc _qtn_proc_alloc
#define qtn_proc_apply_to_self _qtn_proc_apply_to_self
#define qtn_proc_free _qtn_proc_free
#define qtn_proc_init _qtn_proc_init
#define qtn_proc_init_with_self _qtn_proc_init_with_self
#define qtn_proc_set_flags _qtn_proc_set_flags
#define qtn_file_alloc _qtn_file_alloc
#define qtn_file_init_with_path _qtn_file_init_with_path
#define qtn_file_free _qtn_file_free
#define qtn_file_apply_to_path _qtn_file_apply_to_path
#define qtn_file_set_flags _qtn_file_set_flags
#define qtn_file_get_flags _qtn_file_get_flags
#define qtn_proc_set_identifier _qtn_proc_set_identifier

typedef struct _qtn_proc *qtn_proc_t;
typedef struct _qtn_file *qtn_file_t;

int qtn_proc_apply_to_self(qtn_proc_t);
void qtn_proc_init(qtn_proc_t);
int qtn_proc_init_with_self(qtn_proc_t);
int qtn_proc_set_flags(qtn_proc_t, uint32_t flags);
qtn_proc_t qtn_proc_alloc();
void qtn_proc_free(qtn_proc_t);
qtn_file_t qtn_file_alloc(void);
void qtn_file_free(qtn_file_t qf);
int qtn_file_set_flags(qtn_file_t qf, uint32_t flags);
uint32_t qtn_file_get_flags(qtn_file_t qf);
int qtn_file_apply_to_path(qtn_file_t qf, const char *path);
int qtn_file_init_with_path(qtn_file_t qf, const char *path);
int qtn_proc_set_identifier(qtn_proc_t qp, const char* bundleid);

int main() {

qtn_proc_t qp = qtn_proc_alloc();
qtn_proc_set_identifier(qp, "xyz.hacktricks.qa");
qtn_proc_set_flags(qp, QTN_FLAG_DOWNLOAD | QTN_FLAG_USER_APPROVED);
qtn_proc_apply_to_self(qp);
qtn_proc_free(qp);

FILE *fp;
fp = fopen("thisisquarantined.txt", "w+");
fprintf(fp, "Hello Quarantine\n");
fclose(fp);

return 0;

}
```
</details>

E **remova** esse atributo com:
```bash
xattr -d com.apple.quarantine portada.png
#You can also remove this attribute from every file with
find . -iname '*' -print0 | xargs -0 xattr -d com.apple.quarantine
```
E encontre todos os arquivos em quarentena com:

{% code overflow="wrap" %}
```bash
find / -exec ls -ld {} \; 2>/dev/null | grep -E "[x\-]@ " | awk '{printf $9; printf "\n"}' | xargs -I {} xattr -lv {} | grep "com.apple.quarantine"
```
{% endcode %}

As informações de quarentena também são armazenadas em um banco de dados central gerenciado pelo LaunchServices em **`~/Library/Preferences/com.apple.LaunchServices.QuarantineEventsV2`**.

#### **Quarantine.kext**

A extensão do kernel está disponível apenas por meio do **cache do kernel no sistema**; no entanto, você _pode_ baixar o **Kernel Debug Kit em https://developer.apple.com/**, que conterá uma versão simbolizada da extensão.

### XProtect

XProtect é um recurso integrado de **anti-malware** no macOS. XProtect **verifica qualquer aplicativo quando é lançado ou modificado pela primeira vez em relação ao seu banco de dados** de malware conhecido e tipos de arquivo inseguros. Quando você baixa um arquivo por meio de aplicativos específicos, como Safari, Mail ou Mensagens, o XProtect verifica automaticamente o arquivo. Se corresponder a algum malware conhecido em seu banco de dados, o XProtect irá **impedir que o arquivo seja executado** e alertá-lo sobre a ameaça.

O banco de dados do XProtect é **atualizado regularmente** pela Apple com novas definições de malware, e essas atualizações são baixadas e instaladas automaticamente em seu Mac. Isso garante que o XProtect esteja sempre atualizado com as últimas ameaças conhecidas.

No entanto, vale ressaltar que o **XProtect não é uma solução antivírus completa**. Ele verifica apenas uma lista específica de ameaças conhecidas e não realiza varreduras de acesso como a maioria dos softwares antivírus.

Você pode obter informações sobre a última atualização do XProtect executando:

{% code overflow="wrap" %}
```bash
system_profiler SPInstallHistoryDataType 2>/dev/null | grep -A 4 "XProtectPlistConfigData" | tail -n 5
```
{% endcode %}

XProtect está localizado em uma localização protegida pelo SIP em **/Library/Apple/System/Library/CoreServices/XProtect.bundle** e dentro do bundle você pode encontrar as informações que o XProtect utiliza:

- **`XProtect.bundle/Contents/Resources/LegacyEntitlementAllowlist.plist`**: Permite que o código com esses cdhashes utilize as permissões legadas.
- **`XProtect.bundle/Contents/Resources/XProtect.meta.plist`**: Lista de plugins e extensões que são proibidos de carregar via BundleID e TeamID ou indicando uma versão mínima.
- **`XProtect.bundle/Contents/Resources/XProtect.yara`**: Regras Yara para detectar malware.
- **`XProtect.bundle/Contents/Resources/gk.db`**: Banco de dados SQLite3 com hashes de aplicativos bloqueados e TeamIDs.

Note que há outro aplicativo em **`/Library/Apple/System/Library/CoreServices/XProtect.app`** relacionado ao XProtect que não está envolvido no processo do Gatekeeper.

### Não é o Gatekeeper

{% hint style="danger" %}
Observe que o Gatekeeper **não é executado toda vez** que você executa um aplicativo, apenas o _**AppleMobileFileIntegrity**_ (AMFI) irá apenas **verificar as assinaturas de código executável** quando você executar um aplicativo que já foi executado e verificado pelo Gatekeeper.
{% endhint %}

Portanto, anteriormente era possível executar um aplicativo para armazená-lo em cache com o Gatekeeper, então **modificar arquivos não executáveis do aplicativo** (como arquivos Electron asar ou NIB) e se nenhuma outra proteção estivesse em vigor, o aplicativo era **executado** com as **adições maliciosas**.

No entanto, agora isso não é mais possível porque o macOS **impede a modificação de arquivos** dentro dos bundles de aplicativos. Portanto, se você tentar o ataque [Dirty NIB](../macos-proces-abuse/macos-dirty-nib.md), você verá que não é mais possível abusar disso porque depois de executar o aplicativo para armazená-lo em cache com o Gatekeeper, você não poderá modificar o bundle. E se você alterar, por exemplo, o nome do diretório Contents para NotCon (como indicado no exploit) e então executar o binário principal do aplicativo para armazená-lo em cache com o Gatekeeper, ele irá disparar um erro e não será executado.

## Bypasses do Gatekeeper

Qualquer forma de burlar o Gatekeeper (conseguir fazer o usuário baixar algo e executá-lo quando o Gatekeeper deveria proibi-lo) é considerado uma vulnerabilidade no macOS. Abaixo estão alguns CVEs atribuídos a técnicas que permitiram burlar o Gatekeeper no passado:

### [CVE-2021-1810](https://labs.withsecure.com/publications/the-discovery-of-cve-2021-1810)

Foi observado que se o **Utilitário de Arquivo** for usado para extração, arquivos com **caminhos superiores a 886 caracteres** não recebem o atributo estendido com.apple.quarantine. Essa situação inadvertidamente permite que esses arquivos **contornem as verificações de segurança do Gatekeeper**.

Confira o [**relatório original**](https://labs.withsecure.com/publications/the-discovery-of-cve-2021-1810) para mais informações.

### [CVE-2021-30990](https://ronmasas.com/posts/bypass-macos-gatekeeper)

Quando um aplicativo é criado com o **Automator**, as informações sobre o que ele precisa para ser executado estão dentro de `application.app/Contents/document.wflow` e não no executável. O executável é apenas um binário genérico do Automator chamado **Automator Application Stub**.

Portanto, você poderia fazer com que `application.app/Contents/MacOS/Automator\ Application\ Stub` **apontasse com um link simbólico para outro Automator Application Stub dentro do sistema** e ele executaria o que está dentro de `document.wflow` (seu script) **sem acionar o Gatekeeper** porque o executável real não possui o atributo de quarentena.

Exemplo da localização esperada: `/System/Library/CoreServices/Automator\ Application\ Stub.app/Contents/MacOS/Automator\ Application\ Stub`

Confira o [**relatório original**](https://ronmasas.com/posts/bypass-macos-gatekeeper) para mais informações.

### [CVE-2022-22616](https://www.jamf.com/blog/jamf-threat-labs-safari-vuln-gatekeeper-bypass/)

Neste bypass, um arquivo zip foi criado com um aplicativo começando a ser comprimido a partir de `application.app/Contents` em vez de `application.app`. Portanto, o **atributo de quarentena** foi aplicado a todos os **arquivos de `application.app/Contents`** mas **não a `application.app`**, que era o que o Gatekeeper estava verificando, então o Gatekeeper foi burlado porque quando `application.app` foi acionado, ele **não tinha o atributo de quarentena**.
```bash
zip -r test.app/Contents test.zip
```
Verifique o [**relatório original**](https://www.jamf.com/blog/jamf-threat-labs-safari-vuln-gatekeeper-bypass/) para mais informações.

### [CVE-2022-32910](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2022-32910)

Mesmo que os componentes sejam diferentes, a exploração dessa vulnerabilidade é muito semelhante à anterior. Neste caso, iremos gerar um Arquivo Apple a partir de **`application.app/Contents`** para que **`application.app` não receba o atributo de quarentena** ao ser descompactado pelo **Archive Utility**.
```bash
aa archive -d test.app/Contents -o test.app.aar
```
Verifique o [**relatório original**](https://www.jamf.com/blog/jamf-threat-labs-macos-archive-utility-vulnerability/) para mais informações.

### [CVE-2022-42821](https://www.microsoft.com/en-us/security/blog/2022/12/19/gatekeepers-achilles-heel-unearthing-a-macos-vulnerability/)

O ACL **`writeextattr`** pode ser usado para impedir que alguém escreva um atributo em um arquivo:
```bash
touch /tmp/no-attr
chmod +a "everyone deny writeextattr" /tmp/no-attr
xattr -w attrname vale /tmp/no-attr
xattr: [Errno 13] Permission denied: '/tmp/no-attr'
```
Além disso, o formato de arquivo **AppleDouble** copia um arquivo incluindo suas ACEs.

No [**código-fonte**](https://opensource.apple.com/source/Libc/Libc-391/darwin/copyfile.c.auto.html) é possível ver que a representação de texto do ACL armazenada dentro do xattr chamado **`com.apple.acl.text`** será definida como ACL no arquivo descompactado. Portanto, se você comprimiu um aplicativo em um arquivo zip com o formato de arquivo **AppleDouble** com um ACL que impede que outros xattrs sejam gravados nele... o xattr de quarentena não foi definido no aplicativo:
```bash
chmod +a "everyone deny write,writeattr,writeextattr" /tmp/test
ditto -c -k test test.zip
python3 -m http.server
# Download the zip from the browser and decompress it, the file should be without a quarantine xattr
```
{% endcode %}

Verifique o [**relatório original**](https://www.microsoft.com/en-us/security/blog/2022/12/19/gatekeepers-achilles-heel-unearthing-a-macos-vulnerability/) para mais informações.

Note que isso também poderia ser explorado com o AppleArchives:
```bash
mkdir app
touch app/test
chmod +a "everyone deny write,writeattr,writeextattr" app/test
aa archive -d app -o test.aar
```
### [CVE-2023-27943](https://blog.f-secure.com/discovery-of-gatekeeper-bypass-cve-2023-27943/)

Foi descoberto que o **Google Chrome não estava definindo o atributo de quarentena** para arquivos baixados devido a alguns problemas internos do macOS.

### [CVE-2023-27951](https://redcanary.com/blog/gatekeeper-bypass-vulnerabilities/)

Os formatos de arquivo AppleDouble armazenam os atributos de um arquivo em um arquivo separado começando por `._`, isso ajuda a copiar atributos de arquivos **entre máquinas macOS**. No entanto, foi observado que após descompactar um arquivo AppleDouble, o arquivo começando com `._` **não recebia o atributo de quarentena**.

{% code overflow="wrap" %}
```bash
mkdir test
echo a > test/a
echo b > test/b
echo ._a > test/._a
aa archive -d test/ -o test.aar

# If you downloaded the resulting test.aar and decompress it, the file test/._a won't have a quarantitne attribute
```
{% endcode %}

Ser capaz de criar um arquivo que não terá o atributo de quarentena definido, era **possível contornar o Gatekeeper.** O truque era **criar um aplicativo de arquivo DMG** usando a convenção de nome AppleDouble (iniciando com `._`) e criar um **arquivo visível como um link simbólico para este arquivo oculto** sem o atributo de quarentena.\
Quando o **arquivo dmg é executado**, como não possui um atributo de quarentena, ele **contornará o Gatekeeper**.
```bash
# Create an app bundle with the backdoor an call it app.app

echo "[+] creating disk image with app"
hdiutil create -srcfolder app.app app.dmg

echo "[+] creating directory and files"
mkdir
mkdir -p s/app
cp app.dmg s/app/._app.dmg
ln -s ._app.dmg s/app/app.dmg

echo "[+] compressing files"
aa archive -d s/ -o app.aar
```
### Prevenir a xattr de quarentena

Em um pacote ".app", se a xattr de quarentena não for adicionada a ele, ao executá-lo **o Gatekeeper não será acionado**.
