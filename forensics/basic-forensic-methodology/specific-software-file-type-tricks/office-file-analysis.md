# Análise de arquivos do Office

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* Você trabalha em uma **empresa de cibersegurança**? Gostaria de ver sua **empresa anunciada no HackTricks**? Ou gostaria de ter acesso à **última versão do PEASS ou baixar o HackTricks em PDF**? Confira os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* Adquira o [**swag oficial do PEASS & HackTricks**](https://peass.creator-spring.com)
* **Junte-se ao** [**💬**](https://emojipedia.org/speech-balloon/) [**grupo Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo Telegram**](https://t.me/peass) ou **siga-me** no **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Compartilhe seus truques de hacking enviando PRs para o** [**repositório hacktricks**](https://github.com/carlospolop/hacktricks) **e para o** [**repositório hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>

<figure><img src="/.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

\
Use [**Trickest**](https://trickest.io/) para construir e **automatizar fluxos de trabalho** com as ferramentas comunitárias mais avançadas do mundo.\
Acesse hoje mesmo:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## Introdução

A Microsoft criou **dezenas de formatos de arquivos de documentos do Office**, muitos dos quais são populares para a distribuição de ataques de phishing e malware devido à sua capacidade de **incluir macros** (scripts VBA).

De forma geral, existem duas gerações de formatos de arquivos do Office: os formatos **OLE** (extensões de arquivo como RTF, DOC, XLS, PPT) e os formatos "**Office Open XML**" (extensões de arquivo que incluem DOCX, XLSX, PPTX). **Ambos** os formatos são formatos binários de arquivo compostos e estruturados que **permitem conteúdo vinculado ou incorporado** (Objetos). Os arquivos OOXML são contêineres de arquivos zip, o que significa que uma das maneiras mais fáceis de verificar dados ocultos é simplesmente `descompactar` o documento:
```
$ unzip example.docx
Archive:  example.docx
inflating: [Content_Types].xml
inflating: _rels/.rels
inflating: word/_rels/document.xml.rels
inflating: word/document.xml
inflating: word/theme/theme1.xml
extracting: docProps/thumbnail.jpeg
inflating: word/comments.xml
inflating: word/settings.xml
inflating: word/fontTable.xml
inflating: word/styles.xml
inflating: word/stylesWithEffects.xml
inflating: docProps/app.xml
inflating: docProps/core.xml
inflating: word/webSettings.xml
inflating: word/numbering.xml
$ tree
.
├── [Content_Types].xml
├── _rels
├── docProps
│   ├── app.xml
│   ├── core.xml
│   └── thumbnail.jpeg
└── word
├── _rels
│   └── document.xml.rels
├── comments.xml
├── document.xml
├── fontTable.xml
├── numbering.xml
├── settings.xml
├── styles.xml
├── stylesWithEffects.xml
├── theme
│   └── theme1.xml
└── webSettings.xml
```
Como você pode ver, parte da estrutura é criada pelo arquivo e pela hierarquia de pastas. O restante é especificado nos arquivos XML. [_Novas técnicas esteganográficas para o formato de arquivo OOXML_, 2011](http://download.springer.com/static/pdf/713/chp%3A10.1007%2F978-3-642-23300-5\_27.pdf?originUrl=http%3A%2F%2Flink.springer.com%2Fchapter%2F10.1007%2F978-3-642-23300-5\_27\&token2=exp=1497911340\~acl=%2Fstatic%2Fpdf%2F713%2Fchp%25253A10.1007%25252F978-3-642-23300-5\_27.pdf%3ForiginUrl%3Dhttp%253A%252F%252Flink.springer.com%252Fchapter%252F10.1007%252F978-3-642-23300-5\_27\*\~hmac=aca7e2655354b656ca7d699e8e68ceb19a95bcf64e1ac67354d8bca04146fd3d) detalha algumas ideias para técnicas de ocultação de dados, mas os autores de desafios CTF sempre estarão criando novas.

Mais uma vez, existe um conjunto de ferramentas em Python para a análise de documentos OLE e OOXML: [oletools](http://www.decalage.info/python/oletools). Para documentos OOXML em particular, [OfficeDissector](https://www.officedissector.com) é um framework de análise muito poderoso (e uma biblioteca em Python). Este último inclui um [guia rápido sobre o seu uso](https://github.com/grierforensics/officedissector/blob/master/doc/html/\_sources/txt/ANALYZING\_OOXML.txt).

Às vezes, o desafio não é encontrar dados estáticos ocultos, mas **analisar uma macro VBA** para determinar seu comportamento. Esse é um cenário mais realista e algo que os analistas de campo realizam todos os dias. As ferramentas de dissecação mencionadas podem indicar se uma macro está presente e provavelmente extraí-la para você. Uma macro VBA típica em um documento do Office, no Windows, irá baixar um script do PowerShell para %TEMP% e tentar executá-lo, nesse caso, você agora tem uma tarefa de análise de script do PowerShell também. No entanto, as macros VBA maliciosas raramente são complicadas, já que a VBA é [geralmente usada apenas como uma plataforma de partida para a execução de código](https://www.lastline.com/labsblog/party-like-its-1999-comeback-of-vba-malware-downloaders-part-3/). No caso em que você precisa entender uma macro VBA complicada ou se a macro estiver ofuscada e tiver um rotina de descompactação, você não precisa ter uma licença do Microsoft Office para depurar isso. Você pode usar o [Libre Office](http://libreoffice.org): [sua interface](http://www.debugpoint.com/2014/09/debugging-libreoffice-macro-basic-using-breakpoint-and-watch/) será familiar para qualquer pessoa que já tenha depurado um programa; você pode definir pontos de interrupção, criar variáveis de observação e capturar valores depois que eles foram descompactados, mas antes que o comportamento da carga útil seja executado. Você até pode iniciar uma macro de um documento específico a partir da linha de comando:
```
$ soffice path/to/test.docx macro://./standard.module1.mymacro
```
## [oletools](https://github.com/decalage2/oletools)

oletools é uma coleção de scripts Python para análise de arquivos OLE (Object Linking and Embedding). Essas ferramentas podem ser úteis para a análise forense de arquivos do Microsoft Office, como documentos do Word, planilhas do Excel e apresentações do PowerPoint.

### oleid

O script `oleid` é usado para identificar o tipo de arquivo OLE. Ele verifica a estrutura do arquivo e fornece informações sobre os objetos OLE presentes no arquivo. Isso pode ajudar a determinar se um arquivo é realmente um arquivo OLE válido.

### olevba

O script `olevba` é usado para extrair e analisar macros VBA (Visual Basic for Applications) de arquivos do Office. Ele pode ser usado para identificar macros maliciosas ou suspeitas que podem estar presentes em um arquivo.

### oledump

O script `oledump` é uma ferramenta poderosa para análise de arquivos OLE. Ele permite visualizar a estrutura interna de um arquivo OLE e extrair objetos específicos, como macros, streams e objetos embutidos. Isso pode ser útil para identificar conteúdo malicioso oculto em um arquivo.

### olebrowse

O script `olebrowse` é uma ferramenta interativa para explorar arquivos OLE. Ele permite navegar pela estrutura do arquivo, visualizar objetos e streams, e até mesmo editar e salvar modificações no arquivo. Isso pode ser útil para examinar detalhadamente um arquivo OLE e entender sua funcionalidade.

### olemeta

O script `olemeta` é usado para extrair metadados de arquivos OLE. Ele pode fornecer informações sobre o autor, título, assunto e outras propriedades do arquivo. Isso pode ser útil para a análise forense de documentos do Office e a obtenção de informações adicionais sobre um arquivo.

Essas ferramentas do oletools podem ser usadas em conjunto para realizar uma análise forense completa de arquivos do Microsoft Office. Elas podem ajudar a identificar conteúdo malicioso, extrair informações ocultas e entender a estrutura interna de um arquivo OLE.
```bash
sudo pip3 install -U oletools
olevba -c /path/to/document #Extract macros
```
## Execução Automática

Funções de macro como `AutoOpen`, `AutoExec` ou `Document_Open` serão **executadas automaticamente**.

## Referências

* [https://trailofbits.github.io/ctf/forensics/](https://trailofbits.github.io/ctf/forensics/)

<figure><img src="/.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

\
Use [**Trickest**](https://trickest.io/) para construir e **automatizar fluxos de trabalho** com as ferramentas comunitárias mais avançadas do mundo.\
Acesse hoje mesmo:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* Você trabalha em uma **empresa de cibersegurança**? Gostaria de ver sua **empresa anunciada no HackTricks**? Ou gostaria de ter acesso à **última versão do PEASS ou baixar o HackTricks em PDF**? Confira os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* Adquira o [**swag oficial do PEASS & HackTricks**](https://peass.creator-spring.com)
* **Junte-se ao** [**💬**](https://emojipedia.org/speech-balloon/) [**grupo Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo Telegram**](https://t.me/peass) ou **siga-me** no **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Compartilhe suas técnicas de hacking enviando PRs para o** [**repositório hacktricks**](https://github.com/carlospolop/hacktricks) **e para o** [**repositório hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
