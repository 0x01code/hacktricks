# Binários Universais macOS & Formato Mach-O

<details>

<summary><strong>Aprenda hacking no AWS do zero ao herói com</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Outras formas de apoiar o HackTricks:

* Se você quer ver sua **empresa anunciada no HackTricks** ou **baixar o HackTricks em PDF**, confira os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Adquira o [**material oficial PEASS & HackTricks**](https://peass.creator-spring.com)
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção de [**NFTs exclusivos**](https://opensea.io/collection/the-peass-family)
* **Junte-se ao grupo** 💬 [**Discord**](https://discord.gg/hRep4RUj7f) ou ao grupo [**telegram**](https://t.me/peass) ou **siga-me** no **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **Compartilhe suas técnicas de hacking enviando PRs para os repositórios github** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>

## Informações Básicas

Binários do Mac OS geralmente são compilados como **binários universais**. Um **binário universal** pode **suportar múltiplas arquiteturas no mesmo arquivo**.

Esses binários seguem a **estrutura Mach-O**, que é basicamente composta de:

* Cabeçalho
* Comandos de Carga
* Dados

![](<../../../.gitbook/assets/image (559).png>)

## Cabeçalho Fat

Procure pelo arquivo com: `mdfind fat.h | grep -i mach-o | grep -E "fat.h$"`

<pre class="language-c"><code class="lang-c"><strong>#define FAT_MAGIC	0xcafebabe
</strong><strong>#define FAT_CIGAM	0xbebafeca	/* NXSwapLong(FAT_MAGIC) */
</strong>
struct fat_header {
<strong>	uint32_t	magic;		/* FAT_MAGIC ou FAT_MAGIC_64 */
</strong><strong>	uint32_t	nfat_arch;	/* número de estruturas que seguem */
</strong>};

struct fat_arch {
cpu_type_t	cputype;	/* especificador de cpu (int) */
cpu_subtype_t	cpusubtype;	/* especificador de máquina (int) */
uint32_t	offset;		/* deslocamento no arquivo para este arquivo objeto */
uint32_t	size;		/* tamanho deste arquivo objeto */
uint32_t	align;		/* alinhamento como potência de 2 */
};
</code></pre>

O cabeçalho tem os bytes **magic** seguidos pelo **número** de **archs** que o arquivo **contém** (`nfat_arch`) e cada arch terá uma estrutura `fat_arch`.

Verifique com:

<pre class="language-shell-session"><code class="lang-shell-session">% file /bin/ls
/bin/ls: Mach-O universal binary com 2 arquiteturas: [x86_64:Mach-O executável de 64 bits x86_64] [arm64e:Mach-O executável de 64 bits arm64e]
/bin/ls (para arquitetura x86_64):	Mach-O executável de 64 bits x86_64
/bin/ls (para arquitetura arm64e):	Mach-O executável de 64 bits arm64e

% otool -f -v /bin/ls
Cabeçalhos Fat
fat_magic FAT_MAGIC
<strong>nfat_arch 2
</strong><strong>arquitetura x86_64
</strong>    cputype CPU_TYPE_X86_64
cpusubtype CPU_SUBTYPE_X86_64_ALL
capacidades 0x0
<strong>    deslocamento 16384
</strong><strong>    tamanho 72896
</strong>    alinhamento 2^14 (16384)
<strong>arquitetura arm64e
</strong>    cputype CPU_TYPE_ARM64
cpusubtype CPU_SUBTYPE_ARM64E
capacidades PTR_AUTH_VERSION USERSPACE 0
<strong>    deslocamento 98304
</strong><strong>    tamanho 88816
</strong>    alinhamento 2^14 (16384)
</code></pre>

ou usando a ferramenta [Mach-O View](https://sourceforge.net/projects/machoview/):

<figure><img src="../../../.gitbook/assets/image (5) (1) (1) (3) (1).png" alt=""><figcaption></figcaption></figure>

Como você pode estar pensando, geralmente um binário universal compilado para 2 arquiteturas **dobra o tamanho** de um compilado para apenas 1 arquitetura.

## **Cabeçalho Mach-O**

O cabeçalho contém informações básicas sobre o arquivo, como bytes mágicos para identificá-lo como um arquivo Mach-O e informações sobre a arquitetura alvo. Você pode encontrá-lo em: `mdfind loader.h | grep -i mach-o | grep -E "loader.h$"`
```c
#define	MH_MAGIC	0xfeedface	/* the mach magic number */
#define MH_CIGAM	0xcefaedfe	/* NXSwapInt(MH_MAGIC) */
struct mach_header {
uint32_t	magic;		/* mach magic number identifier */
cpu_type_t	cputype;	/* cpu specifier (e.g. I386) */
cpu_subtype_t	cpusubtype;	/* machine specifier */
uint32_t	filetype;	/* type of file (usage and alignment for the file) */
uint32_t	ncmds;		/* number of load commands */
uint32_t	sizeofcmds;	/* the size of all the load commands */
uint32_t	flags;		/* flags */
};

#define MH_MAGIC_64 0xfeedfacf /* the 64-bit mach magic number */
#define MH_CIGAM_64 0xcffaedfe /* NXSwapInt(MH_MAGIC_64) */
struct mach_header_64 {
uint32_t	magic;		/* mach magic number identifier */
int32_t		cputype;	/* cpu specifier */
int32_t		cpusubtype;	/* machine specifier */
uint32_t	filetype;	/* type of file */
uint32_t	ncmds;		/* number of load commands */
uint32_t	sizeofcmds;	/* the size of all the load commands */
uint32_t	flags;		/* flags */
uint32_t	reserved;	/* reserved */
};
```
**Tipos de Arquivos**:

* MH\_EXECUTE (0x2): Executável Mach-O padrão
* MH\_DYLIB (0x6): Uma biblioteca dinâmica Mach-O (ou seja, .dylib)
* MH\_BUNDLE (0x8): Um pacote Mach-O (ou seja, .bundle)
```bash
# Checking the mac header of a binary
otool -arch arm64e -hv /bin/ls
Mach header
magic  cputype cpusubtype  caps    filetype ncmds sizeofcmds      flags
MH_MAGIC_64    ARM64          E USR00     EXECUTE    19       1728   NOUNDEFS DYLDLINK TWOLEVEL PIE
```
Ou usando [Mach-O View](https://sourceforge.net/projects/machoview/):

<figure><img src="../../../.gitbook/assets/image (4) (1) (4).png" alt=""><figcaption></figcaption></figure>

## **Comandos de Carga Mach-O**

Isso especifica o **layout do arquivo na memória**. Contém a **localização da tabela de símbolos**, o contexto da thread principal no início da execução e quais **bibliotecas compartilhadas** são necessárias.\
Os comandos basicamente instruem o carregador dinâmico **(dyld) como carregar o binário na memória.**

Todos os comandos de carga começam com uma estrutura **load\_command**, definida no **`loader.h`** mencionado anteriormente:
```objectivec
struct load_command {
uint32_t cmd;           /* type of load command */
uint32_t cmdsize;       /* total size of command in bytes */
};
```
Existem cerca de **50 diferentes tipos de comandos de carga** que o sistema lida de maneira diferente. Os mais comuns são: `LC_SEGMENT_64`, `LC_LOAD_DYLINKER`, `LC_MAIN`, `LC_LOAD_DYLIB` e `LC_CODE_SIGNATURE`.

### **LC\_SEGMENT/LC\_SEGMENT\_64**

{% hint style="success" %}
Basicamente, este tipo de Comando de Carga define **como carregar os segmentos \_\_TEXT** (código executável) **e \_\_DATA** (dados para o processo) de acordo com os **deslocamentos indicados na seção de Dados** quando o binário é executado.
{% endhint %}

Esses comandos **definem segmentos** que são **mapeados** no **espaço de memória virtual** de um processo quando ele é executado.

Existem **diferentes tipos** de segmentos, como o segmento **\_\_TEXT**, que contém o código executável de um programa, e o segmento **\_\_DATA**, que contém dados usados pelo processo. Esses **segmentos estão localizados na seção de dados** do arquivo Mach-O.

**Cada segmento** pode ser ainda mais **dividido** em múltiplas **seções**. A **estrutura do comando de carga** contém **informações** sobre **essas seções** dentro do respectivo segmento.

No cabeçalho, primeiro você encontra o **cabeçalho do segmento**:

<pre class="language-c"><code class="lang-c">struct segment_command_64 { /* para arquiteturas de 64 bits */
uint32_t	cmd;		/* LC_SEGMENT_64 */
uint32_t	cmdsize;	/* inclui sizeof section_64 structs */
char		segname[16];	/* nome do segmento */
uint64_t	vmaddr;		/* endereço de memória deste segmento */
uint64_t	vmsize;		/* tamanho de memória deste segmento */
uint64_t	fileoff;	/* deslocamento do arquivo deste segmento */
uint64_t	filesize;	/* quantidade a mapear do arquivo */
int32_t		maxprot;	/* proteção máxima de VM */
int32_t		initprot;	/* proteção inicial de VM */
<strong>	uint32_t	nsects;		/* número de seções no segmento */
</strong>	uint32_t	flags;		/* flags */
};
</code></pre>

Exemplo de cabeçalho de segmento:

<figure><img src="../../../.gitbook/assets/image (2) (2) (1) (1).png" alt=""><figcaption></figcaption></figure>

Este cabeçalho define o **número de seções cujos cabeçalhos aparecem após** ele:
```c
struct section_64 { /* for 64-bit architectures */
char		sectname[16];	/* name of this section */
char		segname[16];	/* segment this section goes in */
uint64_t	addr;		/* memory address of this section */
uint64_t	size;		/* size in bytes of this section */
uint32_t	offset;		/* file offset of this section */
uint32_t	align;		/* section alignment (power of 2) */
uint32_t	reloff;		/* file offset of relocation entries */
uint32_t	nreloc;		/* number of relocation entries */
uint32_t	flags;		/* flags (section type and attributes)*/
uint32_t	reserved1;	/* reserved (for offset or index) */
uint32_t	reserved2;	/* reserved (for count or sizeof) */
uint32_t	reserved3;	/* reserved */
};
```
Exemplo de **cabeçalho de seção**:

<figure><img src="../../../.gitbook/assets/image (6) (2).png" alt=""><figcaption></figcaption></figure>

Se você **adicionar** o **deslocamento da seção** (0x37DC) + o **deslocamento** onde a **arquitetura começa**, neste caso `0x18000` --> `0x37DC + 0x18000 = 0x1B7DC`

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Também é possível obter **informações dos cabeçalhos** a partir da **linha de comando** com:
```bash
otool -lv /bin/ls
```
Segmentos comuns carregados por este cmd:

* **`__PAGEZERO`:** Instrui o kernel a **mapear** o **endereço zero** para que **não possa ser lido, escrito ou executado**. As variáveis maxprot e minprot na estrutura são definidas como zero para indicar que **não há direitos de leitura-escrita-execução nesta página**.
* Esta alocação é importante para **mitigar vulnerabilidades de referência de ponteiro NULL**.
* **`__TEXT`**: Contém **código executável** com permissões de **leitura** e **execução** (não gravável)**.** Seções comuns deste segmento:
* `__text`: Código binário compilado
* `__const`: Dados constantes
* `__cstring`: Constantes de string
* `__stubs` e `__stubs_helper`: Envolvidos durante o processo de carregamento de biblioteca dinâmica
* **`__DATA`**: Contém dados que são **legíveis** e **graváveis** (não executáveis)**.**
* `__data`: Variáveis globais (que foram inicializadas)
* `__bss`: Variáveis estáticas (que não foram inicializadas)
* `__objc_*` (\_\_objc\_classlist, \_\_objc\_protolist, etc): Informações usadas pelo runtime Objective-C
* **`__LINKEDIT`**: Contém informações para o linker (dyld) como, "entradas de tabela de símbolos, strings e realocação."
* **`__OBJC`**: Contém informações usadas pelo runtime Objective-C. Embora essas informações também possam ser encontradas no segmento \_\_DATA, dentro de várias seções \_\_objc\_\*.

### **`LC_MAIN`**

Contém o ponto de entrada no **atributo entryoff.** No momento do carregamento, **dyld** simplesmente **adiciona** este valor à **base (em memória) do binário**, e então **salta** para esta instrução para iniciar a execução do código do binário.

### **LC\_CODE\_SIGNATURE**

Contém informações sobre a **assinatura de código do arquivo Mach-O**. Contém apenas um **deslocamento** que **aponta** para o **blob de assinatura**. Isso geralmente está no final do arquivo.\
No entanto, você pode encontrar algumas informações sobre esta seção neste [**post do blog**](https://davedelong.com/blog/2018/01/10/reading-your-own-entitlements/) e neste [**gists**](https://gist.github.com/carlospolop/ef26f8eb9fafd4bc22e69e1a32b81da4).

### **LC\_LOAD\_DYLINKER**

Contém o **caminho para o executável do linker dinâmico** que mapeia bibliotecas compartilhadas no espaço de endereço do processo. O **valor é sempre definido como `/usr/lib/dyld`**. É importante notar que no macOS, o mapeamento de dylib acontece em **modo usuário**, não em modo kernel.

### **`LC_LOAD_DYLIB`**

Este comando de carregamento descreve uma dependência de **biblioteca dinâmica** que **instrui** o **carregador** (dyld) a **carregar e vincular a referida biblioteca**. Há um comando de carregamento LC\_LOAD\_DYLIB **para cada biblioteca** de que o binário Mach-O necessita.

* Este comando de carregamento é uma estrutura do tipo **`dylib_command`** (que contém uma struct dylib, descrevendo a biblioteca dinâmica dependente real):
```objectivec
struct dylib_command {
uint32_t        cmd;            /* LC_LOAD_{,WEAK_}DYLIB */
uint32_t        cmdsize;        /* includes pathname string */
struct dylib    dylib;          /* the library identification */
};

struct dylib {
union lc_str  name;                 /* library's path name */
uint32_t timestamp;                 /* library's build time stamp */
uint32_t current_version;           /* library's current version number */
uint32_t compatibility_version;     /* library's compatibility vers number*/
};
```
```markdown
Você também pode obter essas informações a partir do cli com:
```
```bash
otool -L /bin/ls
/bin/ls:
/usr/lib/libutil.dylib (compatibility version 1.0.0, current version 1.0.0)
/usr/lib/libncurses.5.4.dylib (compatibility version 5.4.0, current version 5.4.0)
/usr/lib/libSystem.B.dylib (compatibility version 1.0.0, current version 1319.0.0)
```
Algumas bibliotecas potencialmente relacionadas a malware são:

* **DiskArbitration**: Monitoramento de drives USB
* **AVFoundation:** Captura de áudio e vídeo
* **CoreWLAN**: Varreduras Wifi.

{% hint style="info" %}
Um binário Mach-O pode conter um ou **mais** **construtores**, que serão **executados** **antes** do endereço especificado em **LC\_MAIN**.\
Os deslocamentos de quaisquer construtores são mantidos na seção **\_\_mod\_init\_func** do segmento **\_\_DATA\_CONST**.
{% endhint %}

## **Dados Mach-O**

O coração do arquivo é a região final, os dados, que consiste em uma série de segmentos conforme disposto na região de comandos de carga. **Cada segmento pode conter várias seções de dados**. Cada uma dessas seções **contém código ou dados** de um tipo particular.

{% hint style="success" %}
Os dados são basicamente a parte que contém todas as **informações** que são carregadas pelos comandos de carga **LC\_SEGMENTS\_64**
{% endhint %}

![](<../../../.gitbook/assets/image (507) (3).png>)

Isso inclui:

* **Tabela de funções:** Que contém informações sobre as funções do programa.
* **Tabela de símbolos**: Que contém informações sobre a função externa usada pelo binário
* Também pode conter nomes de função interna, nomes de variáveis e mais.

Para verificar, você pode usar a ferramenta [**Mach-O View**](https://sourceforge.net/projects/machoview/):

<figure><img src="../../../.gitbook/assets/image (2) (1) (4).png" alt=""><figcaption></figcaption></figure>

Ou a partir do cli:
```bash
size -m /bin/ls
```
<details>

<summary><strong>Aprenda hacking no AWS do zero ao herói com</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Outras formas de apoiar o HackTricks:

* Se você quer ver sua **empresa anunciada no HackTricks** ou **baixar o HackTricks em PDF**, confira os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Adquira o [**material oficial PEASS & HackTricks**](https://peass.creator-spring.com)
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção de [**NFTs**](https://opensea.io/collection/the-peass-family) exclusivos
* **Junte-se ao grupo** 💬 [**Discord**](https://discord.gg/hRep4RUj7f) ou ao grupo [**telegram**](https://t.me/peass) ou **siga-me** no **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **Compartilhe suas técnicas de hacking enviando PRs para os repositórios github do** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
