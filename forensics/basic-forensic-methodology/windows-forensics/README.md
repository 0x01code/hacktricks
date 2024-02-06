# Artefatos do Windows

## Artefatos do Windows

<details>

<summary><strong>Aprenda hacking AWS do zero ao herói com</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Outras maneiras de apoiar o HackTricks:

* Se você deseja ver sua **empresa anunciada no HackTricks** ou **baixar o HackTricks em PDF**, verifique os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Adquira o [**swag oficial PEASS & HackTricks**](https://peass.creator-spring.com)
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Junte-se ao** 💬 [**grupo Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo telegram**](https://t.me/peass) ou **siga-nos** no **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Compartilhe seus truques de hacking enviando PRs para os** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>

## Artefatos Genéricos do Windows

### Notificações do Windows 10

No caminho `\Users\<nome de usuário>\AppData\Local\Microsoft\Windows\Notifications`, você pode encontrar o banco de dados `appdb.dat` (antes do aniversário do Windows) ou `wpndatabase.db` (após o Aniversário do Windows).

Dentro deste banco de dados SQLite, você pode encontrar a tabela `Notification` com todas as notificações (em formato XML) que podem conter dados interessantes.

### Linha do Tempo

A Linha do Tempo é uma característica do Windows que fornece um **histórico cronológico** de páginas da web visitadas, documentos editados e aplicativos executados.

O banco de dados reside no caminho `\Users\<nome de usuário>\AppData\Local\ConnectedDevicesPlatform\<id>\ActivitiesCache.db`. Este banco de dados pode ser aberto com uma ferramenta SQLite ou com a ferramenta [**WxTCmd**](https://github.com/EricZimmerman/WxTCmd) **que gera 2 arquivos que podem ser abertos com a ferramenta** [**TimeLine Explorer**](https://ericzimmerman.github.io/#!index.md).

### ADS (Streams de Dados Alternativos)

Arquivos baixados podem conter a **ADS Zone.Identifier** indicando **como** foi **baixado** da intranet, internet, etc. Alguns softwares (como navegadores) geralmente colocam ainda **mais** **informações** como a **URL** de onde o arquivo foi baixado.

## **Backups de Arquivos**

### Lixeira

No Vista/Win7/Win8/Win10, a **Lixeira** pode ser encontrada na pasta **`$Recycle.bin`** na raiz da unidade (`C:\$Recycle.bin`).\
Quando um arquivo é excluído nesta pasta, 2 arquivos específicos são criados:

* `$I{id}`: Informações do arquivo (data em que foi excluído)
* `$R{id}`: Conteúdo do arquivo

![](<../../../.gitbook/assets/image (486).png>)

Tendo esses arquivos, você pode usar a ferramenta [**Rifiuti**](https://github.com/abelcheung/rifiuti2) para obter o endereço original dos arquivos excluídos e a data em que foram excluídos (use `rifiuti-vista.exe` para Vista – Win10).
```
.\rifiuti-vista.exe C:\Users\student\Desktop\Recycle
```
![](<../../../.gitbook/assets/image (495) (1) (1) (1).png>)

### Cópias de Sombra de Volume

Shadow Copy é uma tecnologia incluída no Microsoft Windows que pode criar **cópias de segurança** ou snapshots de arquivos ou volumes de computador, mesmo quando estão em uso.

Essas cópias de segurança geralmente estão localizadas em `\System Volume Information` a partir da raiz do sistema de arquivos e o nome é composto por **UIDs** mostrados na seguinte imagem:

![](<../../../.gitbook/assets/image (520).png>)

Montando a imagem forense com o **ArsenalImageMounter**, a ferramenta [**ShadowCopyView**](https://www.nirsoft.net/utils/shadow\_copy\_view.html) pode ser usada para inspecionar uma cópia de sombra e até **extrair os arquivos** das cópias de segurança da cópia de sombra.

![](<../../../.gitbook/assets/image (521).png>)

A entrada do registro `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\BackupRestore` contém os arquivos e chaves **para não fazer backup**:

![](<../../../.gitbook/assets/image (522).png>)

O registro `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\VSS` também contém informações de configuração sobre as `Cópias de Sombra de Volume`.

### Arquivos AutoSalvos do Office

Você pode encontrar os arquivos autos salvos do office em: `C:\Usuarios\\AppData\Roaming\Microsoft{Excel|Word|Powerpoint}\`

## Itens de Shell

Um item de shell é um item que contém informações sobre como acessar outro arquivo.

### Documentos Recentes (LNK)

O Windows **automaticamente** **cria** esses **atalhos** quando o usuário **abre, usa ou cria um arquivo** em:

* Win7-Win10: `C:\Users\\AppData\Roaming\Microsoft\Windows\Recent\`
* Office: `C:\Users\\AppData\Roaming\Microsoft\Office\Recent\`

Quando uma pasta é criada, um link para a pasta, para a pasta pai e para a pasta avó também é criado.

Esses arquivos de link criados automaticamente **contêm informações sobre a origem** como se é um **arquivo** **ou** uma **pasta**, **tempos MAC** desse arquivo, **informações de volume** de onde o arquivo está armazenado e **pasta do arquivo de destino**. Essas informações podem ser úteis para recuperar esses arquivos caso tenham sido removidos.

Além disso, a **data de criação do link** é a primeira **vez** que o arquivo original foi **usado** e a **data** **modificada** do arquivo de link é a **última** **vez** que o arquivo de origem foi usado.

Para inspecionar esses arquivos, você pode usar [**LinkParser**](http://4discovery.com/our-tools/).

Nesta ferramenta, você encontrará **2 conjuntos** de carimbos de data/hora:

* **Primeiro Conjunto:**
1. FileModifiedDate
2. FileAccessDate
3. FileCreationDate
* **Segundo Conjunto:**
1. LinkModifiedDate
2. LinkAccessDate
3. LinkCreationDate.

O primeiro conjunto de carimbos de data/hora refere-se aos **carimbos de data/hora do arquivo em si**. O segundo conjunto refere-se aos **carimbos de data/hora do arquivo vinculado**.

Você pode obter as mesmas informações executando a ferramenta de linha de comando do Windows: [**LECmd.exe**](https://github.com/EricZimmerman/LECmd)
```
LECmd.exe -d C:\Users\student\Desktop\LNKs --csv C:\Users\student\Desktop\LNKs
```
### Listas de Salto

Estes são os arquivos recentes indicados por aplicação. É a lista de **arquivos recentes usados por uma aplicação** que você pode acessar em cada aplicação. Eles podem ser criados **automaticamente ou personalizados**.

As **listas de salto** criadas automaticamente são armazenadas em `C:\Users\{username}\AppData\Roaming\Microsoft\Windows\Recent\AutomaticDestinations\`. As listas de salto são nomeadas seguindo o formato `{id}.autmaticDestinations-ms` onde o ID inicial é o ID da aplicação.

As listas de salto personalizadas são armazenadas em `C:\Users\{username}\AppData\Roaming\Microsoft\Windows\Recent\CustomDestination\` e são criadas pela aplicação geralmente porque algo **importante** aconteceu com o arquivo (talvez marcado como favorito).

O **horário de criação** de qualquer lista de salto indica o **primeiro acesso ao arquivo** e o **horário de modificação o último acesso**.

Você pode inspecionar as listas de salto usando [**JumplistExplorer**](https://ericzimmerman.github.io/#!index.md).

![](<../../../.gitbook/assets/image (474).png>)

(_Observe que os carimbos de data e hora fornecidos pelo JumplistExplorer estão relacionados ao arquivo da lista de salto em si_)

### Shellbags

[**Siga este link para aprender o que são as shellbags.**](interesting-windows-registry-keys.md#shellbags)

## Uso de Dispositivos USB do Windows

É possível identificar que um dispositivo USB foi usado graças à criação de:

* Pasta Recente do Windows
* Pasta Recente do Microsoft Office
* Listas de Salto

Observe que alguns arquivos LNK, em vez de apontar para o caminho original, apontam para a pasta WPDNSE:

![](<../../../.gitbook/assets/image (476).png>)

Os arquivos na pasta WPDNSE são uma cópia dos originais, então não sobreviverão a uma reinicialização do PC e o GUID é retirado de uma shellbag.

### Informações do Registro

[Verifique esta página para aprender](interesting-windows-registry-keys.md#usb-information) quais chaves do registro contêm informações interessantes sobre dispositivos USB conectados.

### setupapi

Verifique o arquivo `C:\Windows\inf\setupapi.dev.log` para obter os carimbos de data e hora sobre quando a conexão USB foi produzida (procure por `Section start`).

![](<../../../.gitbook/assets/image (477) (2) (2) (2) (2) (2) (2) (2) (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (14).png>)

### Detetive USB

[**USBDetective**](https://usbdetective.com) pode ser usado para obter informações sobre os dispositivos USB que foram conectados a uma imagem.

![](<../../../.gitbook/assets/image (483).png>)

### Limpeza Plug and Play

A tarefa agendada conhecida como 'Limpeza Plug and Play' é projetada principalmente para a remoção de versões desatualizadas de drivers. Contrariamente ao seu propósito especificado de reter a versão mais recente do pacote de drivers, fontes online sugerem que também visa drivers inativos por 30 dias. Consequentemente, drivers para dispositivos removíveis não conectados nos últimos 30 dias podem estar sujeitos a exclusão.

A tarefa está localizada no seguinte caminho:
`C:\Windows\System32\Tasks\Microsoft\Windows\Plug and Play\Plug and Play Cleanup`.

Uma captura de tela que mostra o conteúdo da tarefa é fornecida:
![](https://2.bp.blogspot.com/-wqYubtuR_W8/W19bV5S9XyI/AAAAAAAANhU/OHsBDEvjqmg9ayzdNwJ4y2DKZnhCdwSMgCLcBGAs/s1600/xml.png)

**Componentes Chave e Configurações da Tarefa:**
- **pnpclean.dll**: Esta DLL é responsável pelo processo de limpeza real.
- **UseUnifiedSchedulingEngine**: Definido como `TRUE`, indicando o uso do mecanismo genérico de agendamento de tarefas.
- **MaintenanceSettings**:
- **Período ('P1M')**: Direciona o Agendador de Tarefas para iniciar a tarefa de limpeza mensalmente durante a manutenção automática regular.
- **Prazo ('P2M')**: Instrui o Agendador de Tarefas, se a tarefa falhar por dois meses consecutivos, a executar a tarefa durante a manutenção automática de emergência.

Esta configuração garante a manutenção regular e a limpeza de drivers, com disposições para tentar novamente a tarefa em caso de falhas consecutivas.

**Para mais informações, consulte:** [**https://blog.1234n6.com/2018/07/windows-plug-and-play-cleanup.html**](https://blog.1234n6.com/2018/07/windows-plug-and-play-cleanup.html)

## Emails

Os emails contêm **2 partes interessantes: Os cabeçalhos e o conteúdo** do email. Nos **cabeçalhos** você pode encontrar informações como:

* **Quem** enviou os emails (endereço de email, IP, servidores de email que redirecionaram o email)
* **Quando** o email foi enviado

Também, nos cabeçalhos `References` e `In-Reply-To` você pode encontrar o ID das mensagens:

![](<../../../.gitbook/assets/image (484).png>)

### Aplicativo de Email do Windows

Este aplicativo salva emails em HTML ou texto. Você pode encontrar os emails dentro de subpastas dentro de `\Users\<username>\AppData\Local\Comms\Unistore\data\3\`. Os emails são salvos com a extensão `.dat`.

Os **metadados** dos emails e os **contatos** podem ser encontrados dentro do banco de dados **EDB**: `\Users\<username>\AppData\Local\Comms\UnistoreDB\store.vol`

**Altere a extensão** do arquivo de `.vol` para `.edb` e você pode usar a ferramenta [ESEDatabaseView](https://www.nirsoft.net/utils/ese\_database\_view.html) para abri-lo. Dentro da tabela `Message` você pode ver os emails.

### Microsoft Outlook

Quando servidores Exchange ou clientes Outlook são usados, haverá alguns cabeçalhos MAPI:

* `Mapi-Client-Submit-Time`: Hora do sistema quando o email foi enviado
* `Mapi-Conversation-Index`: Número de mensagens filhas do tópico e carimbo de data e hora de cada mensagem do tópico
* `Mapi-Entry-ID`: Identificador da mensagem.
* `Mappi-Message-Flags` e `Pr_last_Verb-Executed`: Informações sobre o cliente MAPI (mensagem lida? não lida? respondida? redirecionada? fora do escritório?)

No cliente Microsoft Outlook, todas as mensagens enviadas/recebidas, dados de contatos e dados de calendário são armazenados em um arquivo PST em:

* `%USERPROFILE%\Local Settings\Application Data\Microsoft\Outlook` (WinXP)
* `%USERPROFILE%\AppData\Local\Microsoft\Outlook`

O caminho do registro `HKEY_CURRENT_USER\Software\Microsoft\WindowsNT\CurrentVersion\Windows Messaging Subsystem\Profiles\Outlook` indica o arquivo que está sendo usado.

Você pode abrir o arquivo PST usando a ferramenta [**Kernel PST Viewer**](https://www.nucleustechnologies.com/es/visor-de-pst.html).

![](<../../../.gitbook/assets/image (485).png>)

### Outlook OST

Quando o Microsoft Outlook é configurado **usando** **IMAP** ou usando um servidor **Exchange**, ele gera um arquivo **OST** que armazena quase as mesmas informações que o arquivo PST. Ele mantém o arquivo sincronizado com o servidor pelos **últimos 12 meses**, com um **tamanho máximo de arquivo de 50GB** e na **mesma pasta que o arquivo PST** é salvo. Você pode inspecionar este arquivo usando [**Kernel OST viewer**](https://www.nucleustechnologies.com/ost-viewer.html).

### Recuperando Anexos

Você pode encontrá-los na pasta:

* `%APPDATA%\Local\Microsoft\Windows\Temporary Internet Files\Content.Outlook` -> IE10
* `%APPDATA%\Local\Microsoft\InetCache\Content.Outlook` -> IE11+

### Thunderbird MBOX

**Thunderbird** armazena as informações em **arquivos MBOX** na pasta `\Users\%USERNAME%\AppData\Roaming\Thunderbird\Profiles`

## Miniaturas

Quando um usuário acessa uma pasta e a organiza usando miniaturas, um arquivo `thumbs.db` é criado. Este banco de dados **armazena as miniaturas das imagens** da pasta mesmo que sejam excluídas. No WinXP e Win 8-8.1, este arquivo é criado automaticamente. No Win7/Win10, ele é criado automaticamente se for acessado por um caminho UNC (\IP\pasta...).

É possível ler este arquivo com a ferramenta [**Thumbsviewer**](https://thumbsviewer.github.io).

### Thumbcache

A partir do Windows Vista, **as visualizações em miniatura são armazenadas em um local centralizado no sistema**. Isso fornece ao sistema acesso às imagens independentemente de sua localização e resolve problemas com a localização dos arquivos Thumbs.db. O cache é armazenado em **`%userprofile%\AppData\Local\Microsoft\Windows\Explorer`** como vários arquivos com o rótulo **thumbcache\_xxx.db** (numerados por tamanho); bem como um índice usado para encontrar miniaturas em cada banco de dados de tamanho.

* Thumbcache\_32.db -> pequeno
* Thumbcache\_96.db -> médio
* Thumbcache\_256.db -> grande
* Thumbcache\_1024.db -> extra grande

Você pode ler este arquivo usando [**ThumbCache Viewer**](https://thumbcacheviewer.github.io).

## Registro do Windows

O Registro do Windows contém muitas **informações** sobre o **sistema e as ações dos usuários**.

Os arquivos contendo o registro estão localizados em:

* %windir%\System32\Config\*_SAM\*_: `HKEY_LOCAL_MACHINE`
* %windir%\System32\Config\*_SECURITY\*_: `HKEY_LOCAL_MACHINE`
* %windir%\System32\Config\*_SYSTEM\*_: `HKEY_LOCAL_MACHINE`
* %windir%\System32\Config\*_SOFTWARE\*_: `HKEY_LOCAL_MACHINE`
* %windir%\System32\Config\*_DEFAULT\*_: `HKEY_LOCAL_MACHINE`
* %UserProfile%{User}\*_NTUSER.DAT\*_: `HKEY_CURRENT_USER`

A partir do Windows Vista e do Windows 2008 Server em diante, existem alguns backups dos arquivos de registro `HKEY_LOCAL_MACHINE` em **`%Windir%\System32\Config\RegBack\`**.

Também a partir dessas versões, o arquivo de registro **`%UserProfile%\{User}\AppData\Local\Microsoft\Windows\USERCLASS.DAT`** é criado salvando informações sobre a execução de programas.

### Ferramentas

Algumas ferramentas são úteis para analisar os arquivos de registro:

* **Editor de Registro**: Está instalado no Windows. É uma GUI para navegar pelo registro do Windows da sessão atual.
* [**Explorador de Registro**](https://ericzimmerman.github.io/#!index.md): Permite carregar o arquivo de registro e navegar por eles com uma GUI. Ele também contém Marcadores destacando chaves com informações interessantes.
* [**RegRipper**](https://github.com/keydet89/RegRipper3.0): Novamente, possui uma GUI que permite navegar pelo registro carregado e também contém plugins que destacam informações interessantes dentro do registro carregado.
* [**Recuperação de Registro do Windows**](https://www.mitec.cz/wrr.html): Outra aplicação GUI capaz de extrair as informações importantes do registro carregado.

### Recuperando Elemento Deletado

Quando uma chave é excluída, ela é marcada como tal, mas até que o espaço que ela ocupa seja necessário, ela não será removida. Portanto, usando ferramentas como **Explorador de Registro**, é possível recuperar essas chaves excluídas.

### Último Horário de Escrita

Cada Chave-Valor contém um **carimbo de data e hora** indicando a última vez que foi modificada.

### SAM

O arquivo/hive **SAM** contém os **usuários, grupos e senhas dos usuários** do sistema.

Em `SAM\Domains\Account\Users` você pode obter o nome de usuário, o RID, último login, último login falhado, contador de login, política de senha e quando a conta foi criada. Para obter os **hashes** você também **precisa** do arquivo/hive **SYSTEM**.

### Entradas Interessantes no Registro do Windows

{% content-ref url="interesting-windows-registry-keys.md" %}
[interesting-windows-registry-keys.md](interesting-windows-registry-keys.md)
{% endcontent-ref %}

## Programas Executados

### Processos Básicos do Windows

Na página a seguir, você pode aprender sobre os processos básicos do Windows para detectar comportamentos suspeitos:

{% content-ref url="windows-processes.md" %}
[windows-processes.md](windows-processes.md)
{% endcontent-ref %}

### Aplicativos Recentes do Windows

Dentro do registro `NTUSER.DAT` no caminho `Software\Microsoft\Current Version\Search\RecentApps` você pode encontrar subchaves com informações sobre o **aplicativo executado**, a **última vez** que foi executado e o **número de vezes** que foi iniciado.

### BAM (Moderador de Atividade em Segundo Plano)

Você pode abrir o arquivo `SYSTEM` com um editor de registro e dentro do caminho `SYSTEM\CurrentControlSet\Services\bam\UserSettings\{SID}` você pode encontrar informações sobre os **aplicativos executados por cada usuário** (observe o `{SID}` no caminho) e em **que horário** eles foram executados (o horário está dentro do valor de dados do registro).

### Prefetch do Windows

O prefetching é uma técnica que permite a um computador **buscar silenciosamente os recursos necessários para exibir conteúdo** que um usuário **pode acessar no futuro próximo** para que os recursos possam ser acessados mais rapidamente.

O prefetch do Windows consiste em criar **caches dos programas executados** para poder carregá-los mais rapidamente. Esses caches são criados como arquivos `.pf` dentro do caminho: `C:\Windows\Prefetch`. Há um limite de 128 arquivos no XP/VISTA/WIN7 e 1024 arquivos no Win8/Win10.

O nome do arquivo é criado como `{nome_do_programa}-{hash}.pf` (o hash é baseado no caminho e argumentos do executável). No W10, esses arquivos são comprimidos. Observe que a simples presença do arquivo indica que **o programa foi executado** em algum momento.

O arquivo `C:\Windows\Prefetch\Layout.ini` contém os **nomes das pastas dos arquivos que são prefetchados**. Este arquivo contém **informações sobre o número de execuções**, **datas** da execução e **arquivos** **abertos** pelo programa.

Para inspecionar esses arquivos, você pode usar a ferramenta [**PEcmd.exe**](https://github.com/EricZimmerman/PECmd):
```bash
.\PECmd.exe -d C:\Users\student\Desktop\Prefetch --html "C:\Users\student\Desktop\out_folder"
```
![](<../../../.gitbook/assets/image (487).png>)

### Superprefetch

**Superprefetch** tem o mesmo objetivo que o prefetch, **carregar programas mais rápido** ao prever o que será carregado em seguida. No entanto, não substitui o serviço de prefetch.\
Este serviço irá gerar arquivos de banco de dados em `C:\Windows\Prefetch\Ag*.db`.

Nesses bancos de dados, você pode encontrar o **nome** do **programa**, **número** de **execuções**, **arquivos** **abertos**, **volume** **acessado**, **caminho** **completo**, **intervalos de tempo** e **timestamps**.

Você pode acessar essas informações usando a ferramenta [**CrowdResponse**](https://www.crowdstrike.com/resources/community-tools/crowdresponse/).

### SRUM

**System Resource Usage Monitor** (SRUM) **monitora** os **recursos** **consumidos** **por um processo**. Ele apareceu no W8 e armazena os dados em um banco de dados ESE localizado em `C:\Windows\System32\sru\SRUDB.dat`.

Ele fornece as seguintes informações:

* AppID e Caminho
* Usuário que executou o processo
* Bytes Enviados
* Bytes Recebidos
* Interface de Rede
* Duração da Conexão
* Duração do Processo

Essas informações são atualizadas a cada 60 minutos.

Você pode obter os dados deste arquivo usando a ferramenta [**srum\_dump**](https://github.com/MarkBaggett/srum-dump).
```bash
.\srum_dump.exe -i C:\Users\student\Desktop\SRUDB.dat -t SRUM_TEMPLATE.xlsx -o C:\Users\student\Desktop\srum
```
### AppCompatCache (ShimCache)

**Shimcache**, também conhecido como **AppCompatCache**, é um componente do **Banco de Dados de Compatibilidade de Aplicativos**, que foi criado pela **Microsoft** e usado pelo sistema operacional para identificar problemas de compatibilidade de aplicativos.

O cache armazena vários metadados de arquivos, dependendo do sistema operacional, como:

- Caminho completo do arquivo
- Tamanho do arquivo
- Última hora de modificação do **$Standard\_Information** (SI)
- Hora da última atualização do ShimCache
- Sinalizador de execução do processo

Essas informações podem ser encontradas no registro em:

- `SYSTEM\CurrentControlSet\Control\SessionManager\Appcompatibility\AppcompatCache`
  - XP (96 entradas)
- `SYSTEM\CurrentControlSet\Control\SessionManager\AppcompatCache\AppCompatCache`
  - Server 2003 (512 entradas)
- 2008/2012/2016 Win7/Win8/Win10 (1024 entradas)

Você pode usar a ferramenta [**AppCompatCacheParser**](https://github.com/EricZimmerman/AppCompatCacheParser) para analisar essas informações.

![](<../../../.gitbook/assets/image (488).png>)

### Amcache

O arquivo **Amcache.hve** é um arquivo de registro que armazena as informações de aplicativos executados. Ele está localizado em `C:\Windows\AppCompat\Programas\Amcache.hve`

**Amcache.hve** registra os processos recentes que foram executados e lista o caminho dos arquivos que são executados, os quais podem ser usados para encontrar o programa executado. Também registra o SHA1 do programa.

Você pode analisar essas informações com a ferramenta [**Amcacheparser**](https://github.com/EricZimmerman/AmcacheParser)
```bash
AmcacheParser.exe -f C:\Users\student\Desktop\Amcache.hve --csv C:\Users\student\Desktop\srum
```
O arquivo CVS mais interessante gerado é o `Amcache_Unassociated file entries`.

### RecentFileCache

Este artefato só pode ser encontrado no W7 em `C:\Windows\AppCompat\Programs\RecentFileCache.bcf` e contém informações sobre a execução recente de alguns binários.

Você pode usar a ferramenta [**RecentFileCacheParse**](https://github.com/EricZimmerman/RecentFileCacheParser) para analisar o arquivo.

### Tarefas agendadas

Você pode extrair elas de `C:\Windows\Tasks` ou `C:\Windows\System32\Tasks` e lê-las como XML.

### Serviços

Você pode encontrá-los no registro em `SYSTEM\ControlSet001\Services`. Você pode ver o que será executado e quando.

### **Windows Store**

As aplicações instaladas podem ser encontradas em `\ProgramData\Microsoft\Windows\AppRepository\`\
Este repositório tem um **log** com **cada aplicação instalada** no sistema dentro do banco de dados **`StateRepository-Machine.srd`**.

Dentro da tabela de Aplicativos deste banco de dados, é possível encontrar as colunas: "ID do Aplicativo", "Número do Pacote" e "Nome de Exibição". Essas colunas têm informações sobre aplicativos pré-instalados e instalados e é possível encontrar se alguns aplicativos foram desinstalados porque os IDs dos aplicativos instalados devem ser sequenciais.

Também é possível **encontrar aplicativos instalados** no caminho do registro: `Software\Microsoft\Windows\CurrentVersion\Appx\AppxAllUserStore\Applications\`\
E **desinstalados** **aplicativos** em: `Software\Microsoft\Windows\CurrentVersion\Appx\AppxAllUserStore\Deleted\`

## Eventos do Windows

As informações que aparecem nos eventos do Windows são:

* O que aconteceu
* Carimbo de data/hora (UTC + 0)
* Usuários envolvidos
* Hosts envolvidos (nome do host, IP)
* Ativos acessados (arquivos, pastas, impressoras, serviços)

Os logs estão localizados em `C:\Windows\System32\config` antes do Windows Vista e em `C:\Windows\System32\winevt\Logs` após o Windows Vista. Antes do Windows Vista, os logs de eventos estavam em formato binário e depois disso, estão em formato **XML** e usam a extensão **.evtx**.

A localização dos arquivos de eventos pode ser encontrada no registro do SYSTEM em **`HKLM\SYSTEM\CurrentControlSet\services\EventLog\{Application|System|Security}`**

Eles podem ser visualizados a partir do Visualizador de Eventos do Windows (**`eventvwr.msc`**) ou com outras ferramentas como [**Event Log Explorer**](https://eventlogxp.com) **ou** [**Evtx Explorer/EvtxECmd**](https://ericzimmerman.github.io/#!index.md)**.**

### Segurança

Isso registra os eventos de acesso e fornece informações sobre a configuração de segurança que pode ser encontrada em `C:\Windows\System32\winevt\Security.evtx`.

O **tamanho máximo** do arquivo de evento é configurável e começará a sobrescrever eventos antigos quando o tamanho máximo for atingido.

Eventos que são registrados como:

* Login/Logoff
* Ações do usuário
* Acesso a arquivos, pastas e ativos compartilhados
* Modificação da configuração de segurança

Eventos relacionados à autenticação do usuário:

| EventID   | Descrição                  |
| --------- | ---------------------------- |
| 4624      | Autenticação bem-sucedida    |
| 4625      | Erro de autenticação         |
| 4634/4647 | logoff                      |
| 4672      | Login com permissões de administrador |

Dentro do EventID 4634/4647 existem subtipos interessantes:

* **2 (interativo)**: O login foi interativo usando o teclado ou software como VNC ou `PSexec -U-`
* **3 (rede)**: Conexão a uma pasta compartilhada
* **4 (Lote)**: Processo executado
* **5 (serviço)**: Serviço iniciado pelo Gerenciador de Controle de Serviço
* **6 (proxy):** Login de proxy
* **7 (Desbloquear)**: Tela desbloqueada usando senha
* **8 (texto claro de rede)**: Usuário autenticado enviando senhas em texto claro. Este evento costumava vir do IIS
* **9 (novas credenciais)**: É gerado quando o comando `RunAs` é usado ou o usuário acessa um serviço de rede com credenciais diferentes.
* **10 (interativo remoto)**: Autenticação via Serviços de Terminal ou RDP
* **11 (interativo em cache)**: Acesso usando as últimas credenciais em cache porque não foi possível entrar em contato com o controlador de domínio
* **12 (interativo remoto em cache)**: Login remotamente com credenciais em cache (uma combinação de 10 e 11).
* **13 (desbloqueio em cache)**: Desbloquear uma máquina bloqueada com credenciais em cache.

Neste post, você pode encontrar como imitar todos esses tipos de login e em quais deles você poderá extrair credenciais da memória: [https://www.alteredsecurity.com/post/fantastic-windows-logon-types-and-where-to-find-credentials-in-them](https://www.alteredsecurity.com/post/fantastic-windows-logon-types-and-where-to-find-credentials-in-them)

As informações de status e substatus dos eventos podem indicar mais detalhes sobre as causas do evento. Por exemplo, dê uma olhada nos seguintes Códigos de Status e Substatus do Evento ID 4625:

![](<../../../.gitbook/assets/image (455).png>)

### Recuperando Eventos do Windows

É altamente recomendável desligar o PC suspeito **desconectando-o** para maximizar a probabilidade de recuperar os Eventos do Windows. Caso tenham sido excluídos, uma ferramenta que pode ser útil para tentar recuperá-los é o [**Bulk\_extractor**](../partitions-file-systems-carving/file-data-carving-recovery-tools.md#bulk-extractor) indicando a extensão **evtx**.

## Identificando Ataques Comuns com Eventos do Windows

* [https://redteamrecipe.com/event-codes/](https://redteamrecipe.com/event-codes/)

### Ataque de Força Bruta

Um ataque de força bruta pode ser facilmente identificável porque **vários EventIDs 4625 aparecerão**. Se o ataque foi **bem-sucedido**, após os EventIDs 4625, **um EventID 4624 aparecerá**.

### Mudança de Hora

Isso é terrível para a equipe de forense, pois todos os carimbos de data/hora serão modificados. Este evento é registrado pelo EventID 4616 dentro do log de eventos de Segurança.

### Dispositivos USB

Os seguintes EventIDs do Sistema são úteis:

* 20001 / 20003 / 10000: Primeira vez que foi usado
* 10100: Atualização do driver

O EventID 112 do DeviceSetupManager contém o carimbo de data/hora de cada dispositivo USB inserido.

### Desligar / Ligar

O ID 6005 do serviço "Log de Eventos" indica que o PC foi ligado. O ID 6006 indica que foi desligado.

### Exclusão de Logs

O EventID 1102 de Segurança indica que os logs foram excluídos.
