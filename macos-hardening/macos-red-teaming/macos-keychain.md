# macOS Keychain

{% hint style="success" %}
Aprenda e pratique Hacking AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**Treinamento HackTricks AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Aprenda e pratique Hacking GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**Treinamento HackTricks GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Apoie o HackTricks</summary>

* Confira os [**planos de assinatura**](https://github.com/sponsors/carlospolop)!
* **Junte-se ao** 💬 [**grupo Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo telegram**](https://t.me/peass) ou **siga-nos** no **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Compartilhe truques de hacking enviando PRs para os repositórios** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
{% endhint %}

### [WhiteIntel](https://whiteintel.io)

<figure><img src="../../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) é um mecanismo de busca alimentado pela **dark web** que oferece funcionalidades **gratuitas** para verificar se uma empresa ou seus clientes foram **comprometidos** por **malwares de roubo**.

O principal objetivo do WhiteIntel é combater a apropriação de contas e ataques de ransomware resultantes de malwares que roubam informações.

Você pode acessar o site deles e experimentar o mecanismo gratuitamente em:

{% embed url="https://whiteintel.io" %}

***

## Principais Cadeias de Chaves

* A **Cadeia de Chaves do Usuário** (`~/Library/Keychains/login.keycahin-db`), que é usada para armazenar **credenciais específicas do usuário** como senhas de aplicativos, senhas de internet, certificados gerados pelo usuário, senhas de rede e chaves públicas/privadas geradas pelo usuário.
* A **Cadeia de Chaves do Sistema** (`/Library/Keychains/System.keychain`), que armazena **credenciais de todo o sistema** como senhas de WiFi, certificados raiz do sistema, chaves privadas do sistema e senhas de aplicativos do sistema.

### Acesso à Cadeia de Chaves de Senha

Esses arquivos, embora não tenham proteção inerente e possam ser **baixados**, são criptografados e exigem a **senha em texto simples do usuário para serem descriptografados**. Uma ferramenta como [**Chainbreaker**](https://github.com/n0fate/chainbreaker) pode ser usada para descriptografar.

## Proteções de Entradas da Cadeia de Chaves

### ACLs

Cada entrada na cadeia de chaves é governada por **Listas de Controle de Acesso (ACLs)** que ditam quem pode realizar várias ações na entrada da cadeia de chaves, incluindo:

* **ACLAuhtorizationExportClear**: Permite ao detentor obter o texto claro do segredo.
* **ACLAuhtorizationExportWrapped**: Permite ao detentor obter o texto claro criptografado com outra senha fornecida.
* **ACLAuhtorizationAny**: Permite ao detentor realizar qualquer ação.

As ACLs são acompanhadas por uma **lista de aplicativos confiáveis** que podem realizar essas ações sem solicitação. Isso poderia ser:

* **N`il`** (nenhuma autorização necessária, **todos são confiáveis**)
* Uma lista **vazia** (ninguém é confiável)
* **Lista** de **aplicativos** específicos.

Além disso, a entrada pode conter a chave **`ACLAuthorizationPartitionID`,** que é usada para identificar o **teamid, apple** e **cdhash.**

* Se o **teamid** for especificado, então para **acessar o valor da entrada** sem um **prompt**, o aplicativo usado deve ter o **mesmo teamid**.
* Se o **apple** for especificado, então o aplicativo precisa ser **assinado** pela **Apple**.
* Se o **cdhash** for indicado, então o **aplicativo** deve ter o **cdhash** específico.

### Criando uma Entrada na Cadeia de Chaves

Quando uma **nova** **entrada** é criada usando o **`Keychain Access.app`**, as seguintes regras se aplicam:

* Todos os aplicativos podem criptografar.
* **Nenhum aplicativo** pode exportar/descriptografar (sem solicitar ao usuário).
* Todos os aplicativos podem ver a verificação de integridade.
* Nenhum aplicativo pode alterar as ACLs.
* O **partitionID** é definido como **`apple`**.

Quando um **aplicativo cria uma entrada na cadeia de chaves**, as regras são ligeiramente diferentes:

* Todos os aplicativos podem criptografar.
* Apenas o **aplicativo criador** (ou quaisquer outros aplicativos adicionados explicitamente) podem exportar/descriptografar (sem solicitar ao usuário).
* Todos os aplicativos podem ver a verificação de integridade.
* Nenhum aplicativo pode alterar as ACLs.
* O **partitionID** é definido como **`teamid:[teamID aqui]`**.

## Acessando a Cadeia de Chaves

### `security`
```bash
# Dump all metadata and decrypted secrets (a lot of pop-ups)
security dump-keychain -a -d

# Find generic password for the "Slack" account and print the secrets
security find-generic-password -a "Slack" -g

# Change the specified entrys PartitionID entry
security set-generic-password-parition-list -s "test service" -a "test acount" -S
```
### APIs

{% hint style="success" %}
A enumeração e dumping do **keychain** de segredos que **não gerará um prompt** pode ser feita com a ferramenta [**LockSmith**](https://github.com/its-a-feature/LockSmith)
{% endhint %}

Liste e obtenha **informações** sobre cada entrada do keychain:

* A API **`SecItemCopyMatching`** fornece informações sobre cada entrada e existem alguns atributos que você pode definir ao usá-la:
* **`kSecReturnData`**: Se verdadeiro, tentará descriptografar os dados (defina como falso para evitar possíveis pop-ups)
* **`kSecReturnRef`**: Obtenha também a referência ao item do keychain (defina como verdadeiro caso depois você veja que pode descriptografar sem pop-up)
* **`kSecReturnAttributes`**: Obtenha metadados sobre as entradas
* **`kSecMatchLimit`**: Quantos resultados retornar
* **`kSecClass`**: Que tipo de entrada do keychain

Obtenha **ACLs** de cada entrada:

* Com a API **`SecAccessCopyACLList`** você pode obter o **ACL para o item do keychain**, e ele retornará uma lista de ACLs (como `ACLAuhtorizationExportClear` e os outros mencionados anteriormente) onde cada lista tem:
* Descrição
* **Lista de Aplicativos Confiáveis**. Isso poderia ser:
* Um aplicativo: /Applications/Slack.app
* Um binário: /usr/libexec/airportd
* Um grupo: group://AirPort

Exporte os dados:

* A API **`SecKeychainItemCopyContent`** obtém o texto simples
* A API **`SecItemExport`** exporta as chaves e certificados, mas pode ser necessário definir senhas para exportar o conteúdo criptografado

E estes são os **requisitos** para poder **exportar um segredo sem um prompt**:

* Se **1+ aplicativos confiáveis** listados:
* Precisa das **autorizações apropriadas** (**`Nil`**, ou fazer **parte** da lista permitida de aplicativos na autorização para acessar as informações secretas)
* Precisa que a assinatura de código corresponda ao **PartitionID**
* Precisa que a assinatura de código corresponda à de um **aplicativo confiável** (ou ser membro do grupo KeychainAccessGroup correto)
* Se **todos os aplicativos forem confiáveis**:
* Precisa das **autorizações apropriadas**
* Precisa que a assinatura de código corresponda ao **PartitionID**
* Se **não houver PartitionID**, então isso não é necessário

{% hint style="danger" %}
Portanto, se houver **1 aplicativo listado**, você precisa **injetar código nesse aplicativo**.

Se **apple** for indicado no **partitionID**, você poderá acessá-lo com **`osascript`** para qualquer coisa que esteja confiando em todos os aplicativos com apple no partitionID. **`Python`** também poderia ser usado para isso.
{% endhint %}

### Dois atributos adicionais

* **Invisível**: É uma sinalização booleana para **ocultar** a entrada do aplicativo **UI** Keychain
* **Geral**: É para armazenar **metadados** (portanto, NÃO É CIFRADO)
* A Microsoft estava armazenando em texto simples todos os tokens de atualização para acessar pontos finais sensíveis.

## Referências

* [**#OBTS v5.0: "Lock Picking the macOS Keychain" - Cody Thomas**](https://www.youtube.com/watch?v=jKE1ZW33JpY)

### [WhiteIntel](https://whiteintel.io)

<figure><img src="../../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) é um mecanismo de busca alimentado pela **dark web** que oferece funcionalidades **gratuitas** para verificar se uma empresa ou seus clientes foram **comprometidos** por **malwares ladrões**.

O objetivo principal do WhiteIntel é combater a apropriação de contas e ataques de ransomware resultantes de malwares que roubam informações.

Você pode verificar o site deles e experimentar o mecanismo gratuitamente em:

{% embed url="https://whiteintel.io" %}

{% hint style="success" %}
Aprenda e pratique Hacking AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Aprenda e pratique Hacking GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Apoie o HackTricks</summary>

* Confira os [**planos de assinatura**](https://github.com/sponsors/carlospolop)!
* **Junte-se ao** 💬 [**grupo Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo telegram**](https://t.me/peass) ou **siga-nos** no **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Compartilhe truques de hacking enviando PRs para os repositórios** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
{% endhint %}
