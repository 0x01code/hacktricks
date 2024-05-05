# Técnicas Anti-Forense

<details>

<summary><strong>Aprenda hacking AWS do zero ao herói com</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Outras formas de apoiar o HackTricks:

* Se você deseja ver sua **empresa anunciada no HackTricks** ou **baixar o HackTricks em PDF** Confira os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Adquira o [**swag oficial PEASS & HackTricks**](https://peass.creator-spring.com)
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Junte-se ao** 💬 [**grupo Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo telegram**](https://t.me/peass) ou **siga-nos** no **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Compartilhe seus truques de hacking enviando PRs para o** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repositórios do github.

</details>

## Carimbos de Tempo

Um atacante pode estar interessado em **alterar os carimbos de tempo dos arquivos** para evitar ser detectado.\
É possível encontrar os carimbos de tempo dentro do MFT nos atributos `$STANDARD_INFORMATION` e `$FILE_NAME`.

Ambos os atributos possuem 4 carimbos de tempo: **Modificação**, **acesso**, **criação** e **modificação do registro MFT** (MACE ou MACB).

O **explorador do Windows** e outras ferramentas mostram as informações de **`$STANDARD_INFORMATION`**.

### TimeStomp - Ferramenta Anti-forense

Esta ferramenta **modifica** as informações de carimbo de tempo dentro de **`$STANDARD_INFORMATION`** **mas não** as informações dentro de **`$FILE_NAME`**. Portanto, é possível **identificar** **atividades suspeitas**.

### Usnjrnl

O **Diário USN** (Update Sequence Number Journal) é um recurso do NTFS (sistema de arquivos do Windows NT) que mantém o controle das alterações no volume. A ferramenta [**UsnJrnl2Csv**](https://github.com/jschicht/UsnJrnl2Csv) permite a análise dessas alterações.

![](<../../.gitbook/assets/image (801).png>)

A imagem anterior é a **saída** mostrada pela **ferramenta** onde pode ser observado que algumas **alterações foram realizadas** no arquivo.

### $LogFile

**Todas as alterações de metadados em um sistema de arquivos são registradas** em um processo conhecido como [write-ahead logging](https://en.wikipedia.org/wiki/Write-ahead\_logging). Os metadados registrados são mantidos em um arquivo chamado `**$LogFile**`, localizado no diretório raiz de um sistema de arquivos NTFS. Ferramentas como [LogFileParser](https://github.com/jschicht/LogFileParser) podem ser usadas para analisar esse arquivo e identificar alterações.

![](<../../.gitbook/assets/image (137).png>)

Novamente, na saída da ferramenta é possível ver que **algumas alterações foram realizadas**.

Usando a mesma ferramenta, é possível identificar a **que horas os carimbos de tempo foram modificados**:

![](<../../.gitbook/assets/image (1089).png>)

* CTIME: Hora de criação do arquivo
* ATIME: Hora de modificação do arquivo
* MTIME: Modificação do registro MFT do arquivo
* RTIME: Hora de acesso do arquivo

### Comparação de `$STANDARD_INFORMATION` e `$FILE_NAME`

Outra forma de identificar arquivos modificados suspeitos seria comparar o tempo em ambos os atributos em busca de **inconsistências**.

### Nanossegundos

Os carimbos de tempo do **NTFS** têm uma **precisão** de **100 nanossegundos**. Portanto, encontrar arquivos com carimbos de tempo como 2010-10-10 10:10:**00.000:0000 é muito suspeito**.

### SetMace - Ferramenta Anti-forense

Esta ferramenta pode modificar ambos os atributos `$STARNDAR_INFORMATION` e `$FILE_NAME`. No entanto, a partir do Windows Vista, é necessário um sistema operacional ao vivo para modificar essas informações.

## Ocultação de Dados

O NTFS usa um cluster e o tamanho mínimo de informação. Isso significa que se um arquivo ocupar um cluster e meio, o **meio restante nunca será usado** até que o arquivo seja excluído. Portanto, é possível **ocultar dados neste espaço ocioso**.

Existem ferramentas como slacker que permitem ocultar dados neste espaço "oculto". No entanto, uma análise do `$logfile` e `$usnjrnl` pode mostrar que alguns dados foram adicionados:

![](<../../.gitbook/assets/image (1060).png>)

Então, é possível recuperar o espaço ocioso usando ferramentas como FTK Imager. Note que esse tipo de ferramenta pode salvar o conteúdo obfuscado ou até mesmo criptografado.

## UsbKill

Esta é uma ferramenta que **desligará o computador se qualquer alteração nas portas USB** for detectada.\
Uma maneira de descobrir isso seria inspecionar os processos em execução e **revisar cada script Python em execução**.

## Distribuições Linux ao Vivo

Essas distribuições são **executadas dentro da memória RAM**. A única maneira de detectá-las é **caso o sistema de arquivos NTFS seja montado com permissões de gravação**. Se for montado apenas com permissões de leitura, não será possível detectar a intrusão.

## Exclusão Segura

[https://github.com/Claudio-C/awesome-data-sanitization](https://github.com/Claudio-C/awesome-data-sanitization)

## Configuração do Windows

É possível desativar vários métodos de registro do Windows para tornar a investigação forense muito mais difícil.

### Desativar Carimbos de Tempo - UserAssist

Esta é uma chave de registro que mantém datas e horas quando cada executável foi executado pelo usuário.

Desativar o UserAssist requer dois passos:

1. Definir duas chaves de registro, `HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Advanced\Start_TrackProgs` e `HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Advanced\Start_TrackEnabled`, ambos para zero para sinalizar que queremos o UserAssist desativado.
2. Limpar os subárvores do registro que se parecem com `HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist\<hash>`.

### Desativar Carimbos de Tempo - Prefetch

Isso salvará informações sobre os aplicativos executados com o objetivo de melhorar o desempenho do sistema Windows. No entanto, isso também pode ser útil para práticas forenses.

* Execute `regedit`
* Selecione o caminho do arquivo `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SessionManager\Memory Management\PrefetchParameters`
* Clique com o botão direito em `EnablePrefetcher` e `EnableSuperfetch`
* Selecione Modificar em cada um deles para alterar o valor de 1 (ou 3) para 0
* Reinicie

### Desativar Carimbos de Tempo - Último Horário de Acesso

Sempre que uma pasta é aberta de um volume NTFS em um servidor Windows NT, o sistema leva tempo para **atualizar um campo de carimbo de tempo em cada pasta listada**, chamado de último horário de acesso. Em um volume NTFS muito utilizado, isso pode afetar o desempenho.

1. Abra o Editor de Registro (Regedit.exe).
2. Navegue até `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\FileSystem`.
3. Procure por `NtfsDisableLastAccessUpdate`. Se não existir, adicione este DWORD e defina seu valor como 1, o que desativará o processo.
4. Feche o Editor de Registro e reinicie o servidor.
### Apagar Histórico USB

Todas as **Entradas de Dispositivos USB** são armazenadas no Registro do Windows sob a chave do registro **USBSTOR** que contém subchaves criadas sempre que você conecta um Dispositivo USB ao seu PC ou Laptop. Você pode encontrar esta chave aqui `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Enum\USBSTOR`. **Apagando isso** você apagará o histórico USB.\
Você também pode usar a ferramenta [**USBDeview**](https://www.nirsoft.net/utils/usb\_devices\_view.html) para ter certeza de que os apagou (e para apagá-los).

Outro arquivo que salva informações sobre os USBs é o arquivo `setupapi.dev.log` dentro de `C:\Windows\INF`. Este arquivo também deve ser apagado.

### Desativar Cópias de Sombra

**Liste** as cópias de sombra com `vssadmin list shadowstorage`\
**Apague** executando `vssadmin delete shadow`

Você também pode apagá-las via GUI seguindo as etapas propostas em [https://www.ubackup.com/windows-10/how-to-delete-shadow-copies-windows-10-5740.html](https://www.ubackup.com/windows-10/how-to-delete-shadow-copies-windows-10-5740.html)

Para desativar as cópias de sombra [passos daqui](https://support.waters.com/KB\_Inf/Other/WKB15560\_How\_to\_disable\_Volume\_Shadow\_Copy\_Service\_VSS\_in\_Windows):

1. Abra o programa Serviços digitando "services" na caixa de pesquisa de texto após clicar no botão iniciar do Windows.
2. Na lista, encontre "Cópia de Sombra de Volume", selecione-a e acesse Propriedades clicando com o botão direito.
3. Escolha Desativado no menu suspenso "Tipo de inicialização" e confirme a alteração clicando em Aplicar e OK.

Também é possível modificar a configuração de quais arquivos serão copiados na cópia de sombra no registro `HKLM\SYSTEM\CurrentControlSet\Control\BackupRestore\FilesNotToSnapshot`

### Sobrescrever arquivos apagados

* Você pode usar uma **ferramenta do Windows**: `cipher /w:C` Isso indicará ao cipher para remover quaisquer dados do espaço em disco não utilizado disponível na unidade C.
* Você também pode usar ferramentas como [**Eraser**](https://eraser.heidi.ie)

### Apagar logs de eventos do Windows

* Windows + R --> eventvwr.msc --> Expandir "Logs do Windows" --> Clique com o botão direito em cada categoria e selecione "Limpar Log"
* `for /F "tokens=*" %1 in ('wevtutil.exe el') DO wevtutil.exe cl "%1"`
* `Get-EventLog -LogName * | ForEach { Clear-EventLog $_.Log }`

### Desativar logs de eventos do Windows

* `reg add 'HKLM\SYSTEM\CurrentControlSet\Services\eventlog' /v Start /t REG_DWORD /d 4 /f`
* Dentro da seção de serviços, desative o serviço "Log de Eventos do Windows"
* `WEvtUtil.exec clear-log` ou `WEvtUtil.exe cl`

### Desativar $UsnJrnl

* `fsutil usn deletejournal /d c:`
