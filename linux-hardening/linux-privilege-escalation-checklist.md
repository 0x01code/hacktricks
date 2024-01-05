# Checklist - Escalação de Privilégios no Linux

<details>

<summary><strong>Aprenda hacking no AWS do zero ao herói com</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Outras formas de apoiar o HackTricks:

* Se você quer ver sua **empresa anunciada no HackTricks** ou **baixar o HackTricks em PDF**, confira os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Adquira o [**material oficial PEASS & HackTricks**](https://peass.creator-spring.com)
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção de [**NFTs**](https://opensea.io/collection/the-peass-family) exclusivos
* **Junte-se ao grupo** 💬 [**Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo do telegram**](https://t.me/peass) ou **siga-me** no **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **Compartilhe suas técnicas de hacking enviando PRs para os repositórios github do** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>

<figure><img src="../.gitbook/assets/image (7) (2).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1) (3) (1).png" alt=""><figcaption></figcaption></figure>

Junte-se ao servidor [**HackenProof Discord**](https://discord.com/invite/N3FrSbmwdy) para se comunicar com hackers experientes e caçadores de recompensas por bugs!

**Insights de Hacking**\
Engaje-se com conteúdo que explora a emoção e os desafios do hacking

**Notícias de Hacking em Tempo Real**\
Mantenha-se atualizado com o mundo acelerado do hacking através de notícias e insights em tempo real

**Últimos Anúncios**\
Fique informado sobre os mais novos programas de recompensa por bugs e atualizações importantes da plataforma

**Junte-se a nós no** [**Discord**](https://discord.com/invite/N3FrSbmwdy) e comece a colaborar com os melhores hackers hoje mesmo!

### **Melhor ferramenta para procurar vetores de escalação de privilégios locais no Linux:** [**LinPEAS**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS)

### [Informações do Sistema](privilege-escalation/#system-information)

* [ ] Obtenha **informações do SO**
* [ ] Verifique o [**PATH**](privilege-escalation/#path), alguma **pasta gravável**?
* [ ] Verifique as [**variáveis de ambiente**](privilege-escalation/#env-info), algum detalhe sensível?
* [ ] Procure por [**exploits do kernel**](privilege-escalation/#kernel-exploits) **usando scripts** (DirtyCow?)
* [ ] **Verifique** se a [**versão do sudo** é vulnerável](privilege-escalation/#sudo-version)
* [ ] [**Verificação de assinatura do Dmesg** falhou](privilege-escalation/#dmesg-signature-verification-failed)
* [ ] Mais enumeração do sistema ([data, estatísticas do sistema, informações da CPU, impressoras](privilege-escalation/#more-system-enumeration))
* [ ] [Enumere mais defesas](privilege-escalation/#enumerate-possible-defenses)

### [Drives](privilege-escalation/#drives)

* [ ] **Liste drives montados**
* [ ] **Algum drive desmontado?**
* [ ] **Alguma credencial no fstab?**

### [**Software Instalado**](privilege-escalation/#installed-software)

* [ ] **Verifique se há** [**software útil**](privilege-escalation/#useful-software) **instalado**
* [ ] **Verifique se há** [**software vulnerável**](privilege-escalation/#vulnerable-software-installed) **instalado**

### [Processos](privilege-escalation/#processes)

* [ ] Há algum **software desconhecido em execução**?
* [ ] Há algum software em execução com **mais privilégios do que deveria**?
* [ ] Procure por **exploits de processos em execução** (especialmente a versão em execução).
* [ ] Você pode **modificar o binário** de algum processo em execução?
* [ ] **Monitore processos** e verifique se algum processo interessante está sendo executado com frequência.
* [ ] Você pode **ler** alguma **memória de processo interessante** (onde senhas podem estar salvas)?

### [Tarefas Agendadas/Cron jobs?](privilege-escalation/#scheduled-jobs)

* [ ] O [**PATH**](privilege-escalation/#cron-path) está sendo modificado por algum cron e você pode **escrever** nele?
* [ ] Algum [**coringa**](privilege-escalation/#cron-using-a-script-with-a-wildcard-wildcard-injection) em um trabalho cron?
* [ ] Algum [**script modificável**](privilege-escalation/#cron-script-overwriting-and-symlink) está sendo **executado** ou está dentro de uma **pasta modificável**?
* [ ] Você detectou que algum **script** poderia ser ou está sendo [**executado muito frequentemente**](privilege-escalation/#frequent-cron-jobs)? (a cada 1, 2 ou 5 minutos)

### [Serviços](privilege-escalation/#services)

* [ ] Algum arquivo **.service gravável**?
* [ ] Algum **binário gravável** executado por um **serviço**?
* [ ] Alguma **pasta gravável no PATH do systemd**?

### [Timers](privilege-escalation/#timers)

* [ ] Algum **timer gravável**?

### [Sockets](privilege-escalation/#sockets)

* [ ] Algum arquivo **.socket gravável**?
* [ ] Você pode **comunicar com algum socket**?
* [ ] **Sockets HTTP** com informações interessantes?

### [D-Bus](privilege-escalation/#d-bus)

* [ ] Você pode **comunicar com algum D-Bus**?

### [Rede](privilege-escalation/#network)

* [ ] Enumere a rede para saber onde você está
* [ ] **Portas abertas que você não podia acessar antes** de obter um shell dentro da máquina?
* [ ] Você pode **farejar o tráfego** usando `tcpdump`?

### [Usuários](privilege-escalation/#users)

* [ ] Enumeração genérica de usuários/grupos
* [ ] Você tem um **UID muito grande**? A **máquina** é **vulnerável**?
* [ ] Você pode [**escalar privilégios graças a um grupo**](privilege-escalation/interesting-groups-linux-pe/) ao qual pertence?
* [ ] **Dados da área de transferência**?
* [ ] Política de Senhas?
* [ ] Tente **usar** cada **senha conhecida** que você descobriu anteriormente para fazer login **com cada** possível **usuário**. Tente fazer login também sem senha.

### [PATH Gravável](privilege-escalation/#writable-path-abuses)

* [ ] Se você tem **privilégios de escrita sobre alguma pasta no PATH**, você pode ser capaz de escalar privilégios

### [Comandos SUDO e SUID](privilege-escalation/#sudo-and-suid)

* [ ] Você pode executar **qualquer comando com sudo**? Você pode usá-lo para LER, ESCREVER ou EXECUTAR algo como root? ([**GTFOBins**](https://gtfobins.github.io))
* [ ] Há algum **binário SUID explorável**? ([**GTFOBins**](https://gtfobins.github.io))
* [ ] Os comandos [**sudo** são **limitados** pelo **caminho**? você pode **burlar** as restrições](privilege-escalation/#sudo-execution-bypassing-paths)?
* [ ] [**Binário Sudo/SUID sem caminho indicado**](privilege-escalation/#sudo-command-suid-binary-without-command-path)?
* [ ] [**Binário SUID especificando caminho**](privilege-escalation/#suid-binary-with-command-path)? Burlar
* [ ] [**Vulnerabilidade LD\_PRELOAD**](privilege-escalation/#ld\_preload)
* [ ] [**Falta de biblioteca .so em binário SUID**](privilege-escalation/#suid-binary-so-injection) de uma pasta gravável?
* [ ] [**Tokens SUDO disponíveis**](privilege-escalation/#reusing-sudo-tokens)? [**Você pode criar um token SUDO**](privilege-escalation/#var-run-sudo-ts-less-than-username-greater-than)?
* [ ] Você pode [**ler ou modificar arquivos sudoers**](privilege-escalation/#etc-sudoers-etc-sudoers-d)?
* [ ] Você pode [**modificar /etc/ld.so.conf.d/**](privilege-escalation/#etc-ld-so-conf-d)?
* [ ] Comando [**OpenBSD DOAS**](privilege-escalation/#doas)

### [Capacidades](privilege-escalation/#capabilities)

* [ ] Algum binário tem alguma **capacidade inesperada**?

### [ACLs](privilege-escalation/#acls)

* [ ] Algum arquivo tem alguma **ACL inesperada**?

### [Sessões de Shell Abertas](privilege-escalation/#open-shell-sessions)

* [ ] **screen**
* [ ] **tmux**

### [SSH](privilege-escalation/#ssh)

* [ ] **Debian** [**OpenSSL PRNG Previsível - CVE-2008-0166**](privilege-escalation/#debian-openssl-predictable-prng-cve-2008-0166)
* [ ] [**Valores de configuração do SSH interessantes**](privilege-escalation/#ssh-interesting-configuration-values)

### [Arquivos Interessantes](privilege-escalation/#interesting-files)

* [ ] **Arquivos de Perfil** - Ler dados sensíveis? Escrever para escalar privilégios?
* [ ] **Arquivos passwd/shadow** - Ler dados sensíveis? Escrever para escalar privilégios?
* [ ] **Verifique pastas comumente interessantes** para dados sensíveis
* [ ] **Arquivos em Localização Estranha/Propriedade**, você pode ter acesso ou alterar arquivos executáveis
* [ ] **Modificados** nos últimos minutos
* [ ] **Arquivos de banco de dados SQLite**
* [ ] **Arquivos ocultos**
* [ ] **Scripts/Binários no PATH**
* [ ] **Arquivos da Web** (senhas?)
* [ ] **Backups**?
* [ ] **Arquivos conhecidos que contêm senhas**: Use **Linpeas** e **LaZagne**
* [ ] **Pesquisa Genérica**

### [**Arquivos Graváveis**](privilege-escalation/#writable-files)

* [ ] **Modificar biblioteca python** para executar comandos arbitrários?
* [ ] Você pode **modificar arquivos de log**? Exploração **Logtotten**
* [ ] Você pode **modificar /etc/sysconfig/network-scripts/**? Exploração Centos/Redhat
* [ ] Você pode [**escrever em arquivos ini, int.d, systemd ou rc.d**](privilege-escalation/#init-init-d-systemd-and-rc-d)?

### [**Outros truques**](privilege-escalation/#other-tricks)

* [ ] Você pode [**abusar do NFS para escalar privilégios**](privilege-escalation/#nfs-privilege-escalation)?
* [ ] Você precisa [**escapar de um shell restritivo**](privilege-escalation/#escaping-from-restricted-shells)?

<figure><img src="../../.gitbook/assets/image (1) (3) (1).png" alt=""><figcaption></figcaption></figure>

Junte-se ao servidor [**HackenProof Discord**](https://discord.com/invite/N3FrSbmwdy) para se comunicar com hackers experientes e caçadores de recompensas por bugs!

**Insights de Hacking**\
Engaje-se com conteúdo que explora a emoção e os desafios do hacking

**Notícias de Hacking em Tempo Real**\
Mantenha-se atualizado com o mundo acelerado do hacking através de notícias e insights em tempo real

**Últimos Anúncios**\
Fique informado sobre os mais novos programas de recompensa por bugs e atualizações importantes da plataforma

**Junte-se a nós no** [**Discord**](https://discord.com/invite/N3FrSbmwdy) e comece a colaborar com os melhores hackers hoje mesmo!

<details>

<summary><strong>Aprenda hacking no AWS do zero ao herói com</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Outras formas de apoiar o HackTricks:

* Se você quer ver sua **empresa anunciada no HackTricks** ou **baixar o HackTricks em PDF**, confira os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Adquira o [**material oficial PEASS & HackTricks**](https://peass.creator-spring.com)
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção de [**NFTs**](https://opensea.io/collection/the-peass-family) exclusivos
* **Junte-se ao grupo** 💬 [**Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo do telegram**](https://t.me/peass) ou **siga-me** no **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **Compartilhe suas técnicas de hacking enviando PRs para os repositórios github do** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
