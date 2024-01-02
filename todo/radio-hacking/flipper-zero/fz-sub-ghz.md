# FZ - Sub-GHz

<details>

<summary><strong>Aprenda hacking no AWS do zero ao herói com</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Outras formas de apoiar o HackTricks:

* Se você quer ver sua **empresa anunciada no HackTricks** ou **baixar o HackTricks em PDF**, confira os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Adquira o [**material oficial PEASS & HackTricks**](https://peass.creator-spring.com)
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção de [**NFTs**](https://opensea.io/collection/the-peass-family) exclusivos
* **Junte-se ao grupo** 💬 [**Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo do telegram**](https://t.me/peass) ou **siga-me** no **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **Compartilhe suas técnicas de hacking enviando PRs para os repositórios do** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) no github.

</details>

<figure><img src="/.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

Encontre vulnerabilidades que importam mais para que você possa corrigi-las mais rapidamente. Intruder rastreia sua superfície de ataque, executa varreduras proativas de ameaças, encontra problemas em toda a sua pilha tecnológica, de APIs a aplicativos web e sistemas em nuvem. [**Experimente gratuitamente**](https://www.intruder.io/?utm\_source=referral\&utm\_campaign=hacktricks) hoje.

{% embed url="https://www.intruder.io/?utm_campaign=hacktricks&utm_source=referral" %}

***

## Introdução <a href="#kfpn7" id="kfpn7"></a>

O Flipper Zero pode **receber e transmitir frequências de rádio na faixa de 300-928 MHz** com seu módulo embutido, que pode ler, salvar e emular controles remotos. Esses controles são usados para interação com portões, barreiras, fechaduras de rádio, interruptores de controle remoto, campainhas sem fio, luzes inteligentes e mais. O Flipper Zero pode ajudá-lo a aprender se sua segurança está comprometida.

<figure><img src="../../../.gitbook/assets/image (3) (2) (1).png" alt=""><figcaption></figcaption></figure>

## Hardware Sub-GHz <a href="#kfpn7" id="kfpn7"></a>

O Flipper Zero possui um módulo sub-1 GHz embutido baseado em um [chip CC1101](https://www.ti.com/lit/ds/symlink/cc1101.pdf) e uma antena de rádio (o alcance máximo é de 50 metros). Tanto o chip CC1101 quanto a antena são projetados para operar em frequências nas bandas de 300-348 MHz, 387-464 MHz e 779-928 MHz.

<figure><img src="../../../.gitbook/assets/image (1) (8) (1).png" alt=""><figcaption></figcaption></figure>

## Ações

### Analisador de Frequência

{% hint style="info" %}
Como descobrir qual frequência o controle remoto está usando
{% endhint %}

Ao analisar, o Flipper Zero está escaneando a força dos sinais (RSSI) em todas as frequências disponíveis na configuração de frequência. O Flipper Zero exibe a frequência com o valor mais alto de RSSI, com força de sinal superior a -90 [dBm](https://en.wikipedia.org/wiki/DBm).

Para determinar a frequência do controle remoto, faça o seguinte:

1. Coloque o controle remoto bem próximo à esquerda do Flipper Zero.
2. Vá para **Menu Principal** **→ Sub-GHz**.
3. Selecione **Analisador de Frequência**, depois pressione e segure o botão no controle remoto que deseja analisar.
4. Revise o valor da frequência na tela.

### Ler

{% hint style="info" %}
Encontre informações sobre a frequência usada (também outra maneira de descobrir qual frequência é usada)
{% endhint %}

A opção **Ler** **escuta na frequência configurada** na modulação indicada: 433.92 AM por padrão. Se **algo for encontrado** ao ler, **informações são fornecidas** na tela. Essas informações podem ser usadas para replicar o sinal no futuro.

Enquanto a opção Ler está em uso, é possível pressionar o **botão esquerdo** e **configurá-la**.\
No momento, ela tem **4 modulações** (AM270, AM650, FM328 e FM476), e **várias frequências relevantes** armazenadas:

<figure><img src="../../../.gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>

Você pode definir **qualquer uma que lhe interesse**, no entanto, se você **não tem certeza de qual frequência** pode ser a usada pelo controle remoto que possui, **defina Hopping para ON** (Desligado por padrão), e pressione o botão várias vezes até que o Flipper capture e lhe dê as informações necessárias para definir a frequência.

{% hint style="danger" %}
Alternar entre frequências leva algum tempo, portanto sinais transmitidos no momento da troca podem ser perdidos. Para uma melhor recepção de sinal, defina uma frequência fixa determinada pelo Analisador de Frequência.
{% endhint %}

### **Ler Raw**

{% hint style="info" %}
Roubar (e reproduzir) um sinal na frequência configurada
{% endhint %}

A opção **Ler Raw** **grava sinais** enviados na frequência de escuta. Isso pode ser usado para **roubar** um sinal e **repeti-lo**.

Por padrão, **Ler Raw também está em 433.92 em AM650**, mas se com a opção Ler você descobriu que o sinal que lhe interessa está em uma **frequência/modulação diferente, você também pode modificar isso** pressionando à esquerda (enquanto dentro da opção Ler Raw).

### Força Bruta

Se você conhece o protocolo usado, por exemplo, pela porta da garagem, é possível **gerar todos os códigos e enviá-los com o Flipper Zero.** Este é um exemplo que suporta tipos comuns gerais de garagens: [**https://github.com/tobiabocchi/flipperzero-bruteforce**](https://github.com/tobiabocchi/flipperzero-bruteforce)\*\*\*\*

### Adicionar Manualmente

{% hint style="info" %}
Adicionar sinais de uma lista configurada de protocolos
{% endhint %}

#### Lista de [protocolos suportados](https://docs.flipperzero.one/sub-ghz/add-new-remote) <a href="#3iglu" id="3iglu"></a>

| Princeton\_433 (funciona com a maioria dos sistemas de código estático) | 433.92 | Estático  |
| --------------------------------------------------------------- | ------ | ------- |
| Nice Flo 12bit\_433                                             | 433.92 | Estático  |
| Nice Flo 24bit\_433                                             | 433.92 | Estático  |
| CAME 12bit\_433                                                 | 433.92 | Estático  |
| CAME 24bit\_433                                                 | 433.92 | Estático  |
| Linear\_300                                                     | 300.00 | Estático  |
| CAME TWEE                                                       | 433.92 | Estático  |
| Gate TX\_433                                                    | 433.92 | Estático  |
| DoorHan\_315                                                    | 315.00 | Dinâmico |
| DoorHan\_433                                                    | 433.92 | Dinâmico |
| LiftMaster\_315                                                 | 315.00 | Dinâmico |
| LiftMaster\_390                                                 | 390.00 | Dinâmico |
| Security+2.0\_310                                               | 310.00 | Dinâmico |
| Security+2.0\_315                                               | 315.00 | Dinâmico |
| Security+2.0\_390                                               | 390.00 | Dinâmico |

### Fornecedores Sub-GHz suportados

Confira a lista em [https://docs.flipperzero.one/sub-ghz/supported-vendors](https://docs.flipperzero.one/sub-ghz/supported-vendors)

### Frequências Suportadas por Região

Confira a lista em [https://docs.flipperzero.one/sub-ghz/frequencies](https://docs.flipperzero.one/sub-ghz/frequencies)

### Teste

{% hint style="info" %}
Obtenha dBms das frequências salvas
{% endhint %}

## Referência

* [https://docs.flipperzero.one/sub-ghz](https://docs.flipperzero.one/sub-ghz)

<figure><img src="/.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

Encontre vulnerabilidades que importam mais para que você possa corrigi-las mais rapidamente. Intruder rastreia sua superfície de ataque, executa varreduras proativas de ameaças, encontra problemas em toda a sua pilha tecnológica, de APIs a aplicativos web e sistemas em nuvem. [**Experimente gratuitamente**](https://www.intruder.io/?utm\_source=referral\&utm\_campaign=hacktricks) hoje.

{% embed url="https://www.intruder.io/?utm_campaign=hacktricks&utm_source=referral" %}


<details>

<summary><strong>Aprenda hacking no AWS do zero ao herói com</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Outras formas de apoiar o HackTricks:

* Se você quer ver sua **empresa anunciada no HackTricks** ou **baixar o HackTricks em PDF**, confira os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Adquira o [**material oficial PEASS & HackTricks**](https://peass.creator-spring.com)
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção de [**NFTs**](https://opensea.io/collection/the-peass-family) exclusivos
* **Junte-se ao grupo** 💬 [**Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo do telegram**](https://t.me/peass) ou **siga-me** no **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **Compartilhe suas técnicas de hacking enviando PRs para os repositórios do** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) no github.

</details>
