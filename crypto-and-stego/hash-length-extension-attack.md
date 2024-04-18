<details>

<summary><strong>Aprenda hacking AWS do zero ao herói com</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Outras maneiras de apoiar o HackTricks:

* Se você deseja ver sua **empresa anunciada no HackTricks** ou **baixar o HackTricks em PDF**, verifique os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Adquira o [**swag oficial PEASS & HackTricks**](https://peass.creator-spring.com)
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Junte-se ao** 💬 [**grupo Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo telegram**](https://t.me/peass) ou **siga-nos** no **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Compartilhe seus truques de hacking enviando PRs para os** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repositórios do github.

</details>

### [WhiteIntel](https://whiteintel.io)

<figure><img src="/.gitbook/assets/image (1224).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) é um mecanismo de busca alimentado pela **dark web** que oferece funcionalidades **gratuitas** para verificar se uma empresa ou seus clientes foram **comprometidos** por **malwares ladrões**.

O principal objetivo do WhiteIntel é combater tomadas de conta e ataques de ransomware resultantes de malwares que roubam informações.

Você pode verificar o site deles e experimentar o mecanismo gratuitamente em:

{% embed url="https://whiteintel.io" %}

---

# Resumo do ataque

Imagine um servidor que está **assinando** alguns **dados** ao **anexar** um **segredo** a alguns dados de texto claro conhecidos e, em seguida, fazendo o hash desses dados. Se você souber:

* **O comprimento do segredo** (isso também pode ser forçado a partir de uma faixa de comprimento fornecida)
* **Os dados de texto claro**
* **O algoritmo (e é vulnerável a este ataque)**
* **O preenchimento é conhecido**
* Geralmente um padrão é usado, então se os outros 3 requisitos forem atendidos, este também é
* O preenchimento varia dependendo do comprimento do segredo+dados, por isso o comprimento do segredo é necessário

Então, é possível para um **atacante** **anexar** **dados** e **gerar** uma assinatura válida para os **dados anteriores + dados anexados**.

## Como?

Basicamente, os algoritmos vulneráveis geram os hashes primeiro **fazendo o hash de um bloco de dados**, e então, **a partir** do **hash previamente** criado (estado), eles **adicionam o próximo bloco de dados** e **fazem o hash dele**.

Então, imagine que o segredo é "secreto" e os dados são "dados", o MD5 de "secretodata" é 6036708eba0d11f6ef52ad44e8b74d5b.\
Se um atacante quiser anexar a string "anexar" ele pode:

* Gerar um MD5 de 64 "A"s
* Alterar o estado do hash previamente inicializado para 6036708eba0d11f6ef52ad44e8b74d5b
* Anexar a string "anexar"
* Finalizar o hash e o hash resultante será um **válido para "secreto" + "dados" + "preenchimento" + "anexar"**

## **Ferramenta**

{% embed url="https://github.com/iagox86/hash_extender" %}

## Referências

Você pode encontrar este ataque bem explicado em [https://blog.skullsecurity.org/2012/everything-you-need-to-know-about-hash-length-extension-attacks](https://blog.skullsecurity.org/2012/everything-you-need-to-know-about-hash-length-extension-attacks)

### [WhiteIntel](https://whiteintel.io)

<figure><img src="/.gitbook/assets/image (1224).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) é um mecanismo de busca alimentado pela **dark web** que oferece funcionalidades **gratuitas** para verificar se uma empresa ou seus clientes foram **comprometidos** por **malwares ladrões**.

O principal objetivo do WhiteIntel é combater tomadas de conta e ataques de ransomware resultantes de malwares que roubam informações.

Você pode verificar o site deles e experimentar o mecanismo gratuitamente em:

{% embed url="https://whiteintel.io" %}

<details>

<summary><strong>Aprenda hacking AWS do zero ao herói com</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Outras maneiras de apoiar o HackTricks:

* Se você deseja ver sua **empresa anunciada no HackTricks** ou **baixar o HackTricks em PDF**, verifique os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Adquira o [**swag oficial PEASS & HackTricks**](https://peass.creator-spring.com)
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Junte-se ao** 💬 [**grupo Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo telegram**](https://t.me/peass) ou **siga-nos** no **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Compartilhe seus truques de hacking enviando PRs para os** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repositórios do github.

</details>
