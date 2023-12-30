# Metodologia de Reconhecimento Externo

<details>

<summary><strong>Aprenda hacking no AWS do zero ao herói com</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Outras formas de apoiar o HackTricks:

* Se você quer ver sua **empresa anunciada no HackTricks** ou **baixar o HackTricks em PDF**, confira os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Adquira o [**material oficial PEASS & HackTricks**](https://peass.creator-spring.com)
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção de [**NFTs**](https://opensea.io/collection/the-peass-family) exclusivos
* **Junte-se ao grupo** 💬 [**Discord**](https://discord.gg/hRep4RUj7f) ou ao grupo [**telegram**](https://t.me/peass) ou **siga-me** no **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **Compartilhe suas técnicas de hacking enviando PRs para os repositórios github do** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>

<img src="../../.gitbook/assets/i3.png" alt="" data-size="original">\
**Dica para bug bounty**: **inscreva-se** no **Intigriti**, uma plataforma premium de **bug bounty criada por hackers, para hackers**! Junte-se a nós em [**https://go.intigriti.com/hacktricks**](https://go.intigriti.com/hacktricks) hoje e comece a ganhar recompensas de até **$100,000**!

{% embed url="https://go.intigriti.com/hacktricks" %}

## Descoberta de Ativos

> Então, disseram-lhe que tudo pertencente a alguma empresa está dentro do escopo, e você quer descobrir o que essa empresa realmente possui.

O objetivo desta fase é obter todas as **empresas pertencentes à empresa principal** e, em seguida, todos os **ativos** dessas empresas. Para fazer isso, vamos:

1. Encontrar as aquisições da empresa principal, isso nos dará as empresas dentro do escopo.
2. Encontrar o ASN (se houver) de cada empresa, isso nos dará os intervalos de IP pertencentes a cada empresa.
3. Usar buscas reversas de whois para procurar outras entradas (nomes de organizações, domínios...) relacionadas à primeira (isso pode ser feito recursivamente).
4. Usar outras técnicas como filtros `org` e `ssl` do shodan para procurar outros ativos (o truque `ssl` pode ser feito recursivamente).

### **Aquisições**

Primeiramente, precisamos saber quais **outras empresas são propriedade da empresa principal**.\
Uma opção é visitar [https://www.crunchbase.com/](https://www.crunchbase.com), **pesquisar** pela **empresa principal** e **clicar** em "**aquisições**". Lá você verá outras empresas adquiridas pela principal.\
Outra opção é visitar a página do **Wikipedia** da empresa principal e procurar por **aquisições**.

> Ok, neste ponto você deve saber todas as empresas dentro do escopo. Vamos descobrir como encontrar seus ativos.

### **ASNs**

Um número de sistema autônomo (**ASN**) é um **número único** atribuído a um **sistema autônomo** (AS) pela **Internet Assigned Numbers Authority (IANA)**.\
Um **AS** consiste em **blocos** de **endereços IP** que têm uma política claramente definida para acessar redes externas e são administrados por uma única organização, mas podem ser compostos por vários operadores.

É interessante descobrir se a **empresa possui algum ASN** atribuído para encontrar seus **intervalos de IP**. Será interessante realizar um **teste de vulnerabilidade** contra todos os **hosts** dentro do **escopo** e **procurar por domínios** dentro desses IPs.\
Você pode **pesquisar** pelo **nome da empresa**, por **IP** ou por **domínio** em [**https://bgp.he.net/**](https://bgp.he.net)**.**\
**Dependendo da região da empresa, estes links podem ser úteis para coletar mais dados:** [**AFRINIC**](https://www.afrinic.net) **(África),** [**Arin**](https://www.arin.net/about/welcome/region/)**(América do Norte),** [**APNIC**](https://www.apnic.net) **(Ásia),** [**LACNIC**](https://www.lacnic.net) **(América Latina),** [**RIPE NCC**](https://www.ripe.net) **(Europa). De qualquer forma, provavelmente todas as informações** úteis **(intervalos de IP e Whois)** já aparecem no primeiro link.
```bash
#You can try "automate" this with amass, but it's not very recommended
amass intel -org tesla
amass intel -asn 8911,50313,394161
```
Também, [**BBOT**](https://github.com/blacklanternsecurity/bbot) realiza a enumeração de subdomínios automaticamente agregando e resumindo ASNs ao final da varredura.
```bash
bbot -t tesla.com -f subdomain-enum
...
[INFO] bbot.modules.asn: +----------+---------------------+--------------+----------------+----------------------------+-----------+
[INFO] bbot.modules.asn: | AS394161 | 8.244.131.0/24      | 5            | TESLA          | Tesla Motors, Inc.         | US        |
[INFO] bbot.modules.asn: +----------+---------------------+--------------+----------------+----------------------------+-----------+
[INFO] bbot.modules.asn: | AS16509  | 54.148.0.0/15       | 4            | AMAZON-02      | Amazon.com, Inc.           | US        |
[INFO] bbot.modules.asn: +----------+---------------------+--------------+----------------+----------------------------+-----------+
[INFO] bbot.modules.asn: | AS394161 | 8.45.124.0/24       | 3            | TESLA          | Tesla Motors, Inc.         | US        |
[INFO] bbot.modules.asn: +----------+---------------------+--------------+----------------+----------------------------+-----------+
[INFO] bbot.modules.asn: | AS3356   | 8.32.0.0/12         | 1            | LEVEL3         | Level 3 Parent, LLC        | US        |
[INFO] bbot.modules.asn: +----------+---------------------+--------------+----------------+----------------------------+-----------+
[INFO] bbot.modules.asn: | AS3356   | 8.0.0.0/9           | 1            | LEVEL3         | Level 3 Parent, LLC        | US        |
[INFO] bbot.modules.asn: +----------+---------------------+--------------+----------------+----------------------------+-----------+

```
Você pode encontrar os intervalos de IP de uma organização também usando [http://asnlookup.com/](http://asnlookup.com) (ele possui API gratuita).\
Você pode encontrar o IP e ASN de um domínio usando [http://ipv4info.com/](http://ipv4info.com).

### **Procurando por vulnerabilidades**

Neste ponto, conhecemos **todos os ativos dentro do escopo**, então, se você estiver autorizado, poderia lançar algum **scanner de vulnerabilidade** (Nessus, OpenVAS) sobre todos os hosts.\
Além disso, você poderia realizar alguns [**scans de portas**](../pentesting-network/#discovering-hosts-from-the-outside) **ou usar serviços como** shodan **para encontrar** portas abertas **e, dependendo do que encontrar, você deve** consultar este livro sobre como realizar pentest em vários serviços possíveis em execução.\
**Também vale a pena mencionar que você também pode preparar algumas listas de** nomes de usuário **e** senhas **padrão e tentar** força bruta em serviços com [https://github.com/x90skysn3k/brutespray](https://github.com/x90skysn3k/brutespray).

## Domínios

> Sabemos todas as empresas dentro do escopo e seus ativos, é hora de encontrar os domínios dentro do escopo.

_Por favor, note que nas técnicas propostas a seguir você também pode encontrar subdomínios e essa informação não deve ser subestimada._

Primeiramente, você deve procurar pelo(s) **domínio(s) principal(is)** de cada empresa. Por exemplo, para a _Tesla Inc._ seria _tesla.com_.

### **DNS Reverso**

Como você encontrou todos os intervalos de IP dos domínios, você poderia tentar realizar **consultas de DNS reverso** nesses **IPs para encontrar mais domínios dentro do escopo**. Tente usar algum servidor DNS da vítima ou algum servidor DNS bem conhecido (1.1.1.1, 8.8.8.8)
```bash
dnsrecon -r <DNS Range> -n <IP_DNS>   #DNS reverse of all of the addresses
dnsrecon -d facebook.com -r 157.240.221.35/24 #Using facebooks dns
dnsrecon -r 157.240.221.35/24 -n 1.1.1.1 #Using cloudflares dns
dnsrecon -r 157.240.221.35/24 -n 8.8.8.8 #Using google dns
```
Para que isso funcione, o administrador precisa ativar manualmente o PTR.
Você também pode usar uma ferramenta online para essa informação: [http://ptrarchive.com/](http://ptrarchive.com)

### **Reverse Whois (loop)**

Dentro de um **whois**, você pode encontrar muitas informações interessantes como **nome da organização**, **endereço**, **emails**, números de telefone... Mas o que é ainda mais interessante é que você pode encontrar **mais ativos relacionados à empresa** se realizar **buscas reversas de whois por qualquer um desses campos** (por exemplo, outros registros de whois onde o mesmo email aparece).
Você pode usar ferramentas online como:

* [https://viewdns.info/reversewhois/](https://viewdns.info/reversewhois/) - **Grátis**
* [https://domaineye.com/reverse-whois](https://domaineye.com/reverse-whois) - **Grátis**
* [https://www.reversewhois.io/](https://www.reversewhois.io) - **Grátis**
* [https://www.whoxy.com/](https://www.whoxy.com) - **Grátis** na web, API paga.
* [http://reversewhois.domaintools.com/](http://reversewhois.domaintools.com) - Pago
* [https://drs.whoisxmlapi.com/reverse-whois-search](https://drs.whoisxmlapi.com/reverse-whois-search) - Pago (apenas **100 buscas grátis**)
* [https://www.domainiq.com/](https://www.domainiq.com) - Pago

Você pode automatizar essa tarefa usando [**DomLink**](https://github.com/vysecurity/DomLink) (requer uma chave de API do whoxy).
Você também pode realizar algumas descobertas automáticas de reverse whois com [amass](https://github.com/OWASP/Amass): `amass intel -d tesla.com -whois`

**Note que você pode usar essa técnica para descobrir mais nomes de domínio toda vez que encontrar um novo domínio.**

### **Trackers**

Se encontrar **o mesmo ID do mesmo tracker** em 2 páginas diferentes, você pode supor que **ambas as páginas** são **gerenciadas pela mesma equipe**.
Por exemplo, se você ver o mesmo **ID do Google Analytics** ou o mesmo **ID do Adsense** em várias páginas.

Existem algumas páginas e ferramentas que permitem buscar por esses trackers e mais:

* [**Udon**](https://github.com/dhn/udon)
* [**BuiltWith**](https://builtwith.com)
* [**Sitesleuth**](https://www.sitesleuth.io)
* [**Publicwww**](https://publicwww.com)
* [**SpyOnWeb**](http://spyonweb.com)

### **Favicon**

Você sabia que podemos encontrar domínios e subdomínios relacionados ao nosso alvo procurando pelo mesmo hash do ícone favicon? É exatamente isso que a ferramenta [favihash.py](https://github.com/m4ll0k/Bug-Bounty-Toolz/blob/master/favihash.py) feita por [@m4ll0k2](https://twitter.com/m4ll0k2) faz. Veja como usá-la:
```bash
cat my_targets.txt | xargs -I %% bash -c 'echo "http://%%/favicon.ico"' > targets.txt
python3 favihash.py -f https://target/favicon.ico -t targets.txt -s
```
![favihash - descubra domínios com o mesmo hash de ícone favicon](https://www.infosecmatter.com/wp-content/uploads/2020/07/favihash.jpg)

Simplificando, favihash nos permitirá descobrir domínios que têm o mesmo hash de ícone favicon que nosso alvo.

Além disso, você também pode pesquisar tecnologias usando o hash do favicon conforme explicado neste [**post do blog**](https://medium.com/@Asm0d3us/weaponizing-favicon-ico-for-bugbounties-osint-and-what-not-ace3c214e139). Isso significa que, se você conhece o **hash do favicon de uma versão vulnerável de uma tecnologia web**, você pode pesquisar no shodan e **encontrar mais lugares vulneráveis**:
```bash
shodan search org:"Target" http.favicon.hash:116323821 --fields ip_str,port --separator " " | awk '{print $1":"$2}'
```
Esta é a forma como você pode **calcular o hash do favicon** de um site:
```python
import mmh3
import requests
import codecs

def fav_hash(url):
response = requests.get(url)
favicon = codecs.encode(response.content,"base64")
fhash = mmh3.hash(favicon)
print(f"{url} : {fhash}")
return fhash
```
### **Direitos Autorais / String Única**

Procure dentro das páginas da web **strings que possam ser compartilhadas entre diferentes sites na mesma organização**. A **string de direitos autorais** pode ser um bom exemplo. Em seguida, procure essa string no **Google**, em outros **navegadores** ou até mesmo no **Shodan**: `shodan search http.html:"Copyright string"`

### **Tempo CRT**

É comum ter um trabalho cron como
```bash
# /etc/crontab
37 13 */10 * * certbot renew --post-hook "systemctl reload nginx"
```
para renovar todos os certificados de domínio no servidor. Isso significa que mesmo que a CA utilizada para isso não defina o tempo em que foi gerado no tempo de Validade, é possível **encontrar domínios pertencentes à mesma empresa nos registros de transparência de certificados**.
Confira este [**artigo para mais informações**](https://swarm.ptsecurity.com/discovering-domains-via-a-time-correlation-attack/).

### **Passive Takeover**

Aparentemente é comum que as pessoas atribuam subdomínios a IPs que pertencem a provedores de nuvem e, em algum momento, **percam esse endereço IP mas esqueçam de remover o registro DNS**. Portanto, apenas **iniciando uma VM** em uma nuvem (como Digital Ocean), você estará na verdade **assumindo o controle de alguns subdomínios**.

[**Este post**](https://kmsec.uk/blog/passive-takeover/) explica uma história sobre isso e propõe um script que **inicia uma VM no DigitalOcean**, **obtém** o **IPv4** da nova máquina e **procura no Virustotal por registros de subdomínio** apontando para ele.

### **Outras maneiras**

**Observe que você pode usar esta técnica para descobrir mais nomes de domínio sempre que encontrar um novo domínio.**

**Shodan**

Como você já sabe o nome da organização que possui o espaço IP. Você pode pesquisar por esses dados no shodan usando: `org:"Tesla, Inc."` Verifique os hosts encontrados para novos domínios inesperados no certificado TLS.

Você poderia acessar o **certificado TLS** da página principal da web, obter o **nome da Organização** e depois procurar por esse nome dentro dos **certificados TLS** de todas as páginas da web conhecidas pelo **shodan** com o filtro: `ssl:"Tesla Motors"` ou usar uma ferramenta como [**sslsearch**](https://github.com/HarshVaragiya/sslsearch).

**Assetfinder**

[**Assetfinder**](https://github.com/tomnomnom/assetfinder) é uma ferramenta que procura por **domínios relacionados** com um domínio principal e **subdomínios** deles, bastante incrível.

### **Procurando por vulnerabilidades**

Verifique se há alguma [domínio para assumir](../../pentesting-web/domain-subdomain-takeover.md#domain-takeover). Talvez alguma empresa esteja **usando algum domínio** mas eles **perderam a propriedade**. Basta registrá-lo (se for barato o suficiente) e informar a empresa.

Se você encontrar algum **domínio com um IP diferente** dos que você já encontrou na descoberta de ativos, você deve realizar uma **varredura básica de vulnerabilidade** (usando Nessus ou OpenVAS) e alguma [**varredura de portas**](../pentesting-network/#discovering-hosts-from-the-outside) com **nmap/masscan/shodan**. Dependendo de quais serviços estão em execução, você pode encontrar **neste livro algumas dicas para "atacá-los"**.\
_Note que às vezes o domínio está hospedado dentro de um IP que não é controlado pelo cliente, então não está no escopo, tenha cuidado._

<img src="../../.gitbook/assets/i3.png" alt="" data-size="original">\
**Dica de bug bounty**: **inscreva-se** no **Intigriti**, uma plataforma de bug bounty premium criada por hackers, para hackers! Junte-se a nós em [**https://go.intigriti.com/hacktricks**](https://go.intigriti.com/hacktricks) hoje e comece a ganhar recompensas de até **$100,000**!

{% embed url="https://go.intigriti.com/hacktricks" %}

## Subdomínios

> Nós conhecemos todas as empresas dentro do escopo, todos os ativos de cada empresa e todos os domínios relacionados às empresas.

É hora de encontrar todos os possíveis subdomínios de cada domínio encontrado.

### **DNS**

Vamos tentar obter **subdomínios** a partir dos registros **DNS**. Também devemos tentar por **Transferência de Zona** (Se vulnerável, você deve reportar isso).
```bash
dnsrecon -a -d tesla.com
```
### **OSINT**

A maneira mais rápida de obter muitos subdomínios é pesquisar em fontes externas. As **ferramentas** mais utilizadas são as seguintes (para melhores resultados configure as chaves API):

* [**BBOT**](https://github.com/blacklanternsecurity/bbot)
```bash
# subdomains
bbot -t tesla.com -f subdomain-enum

# subdomains (passive only)
bbot -t tesla.com -f subdomain-enum -rf passive

# subdomains + port scan + web screenshots
bbot -t tesla.com -f subdomain-enum -m naabu gowitness -n my_scan -o .
```
* [**Amass**](https://github.com/OWASP/Amass)
```bash
amass enum [-active] [-ip] -d tesla.com
amass enum -d tesla.com | grep tesla.com # To just list subdomains
```
* [**subfinder**](https://github.com/projectdiscovery/subfinder)
```bash
# Subfinder, use -silent to only have subdomains in the output
./subfinder-linux-amd64 -d tesla.com [-silent]
```
* [**findomain**](https://github.com/Edu4rdSHL/findomain/)
```bash
# findomain, use -silent to only have subdomains in the output
./findomain-linux -t tesla.com [--quiet]
```
* [**OneForAll**](https://github.com/shmilylty/OneForAll/tree/master/docs/en-us)
```bash
python3 oneforall.py --target tesla.com [--dns False] [--req False] [--brute False] run
```
* [**assetfinder**](https://github.com/tomnomnom/assetfinder)
```bash
assetfinder --subs-only <domain>
```
* [**Sudomy**](https://github.com/Screetsec/Sudomy)
```bash
# It requires that you create a sudomy.api file with API keys
sudomy -d tesla.com
```
* [**vita**](https://github.com/junnlikestea/vita)
```
vita -d tesla.com
```
* [**theHarvester**](https://github.com/laramies/theHarvester)
```bash
theHarvester -d tesla.com -b "anubis, baidu, bing, binaryedge, bingapi, bufferoverun, censys, certspotter, crtsh, dnsdumpster, duckduckgo, fullhunt, github-code, google, hackertarget, hunter, intelx, linkedin, linkedin_links, n45ht, omnisint, otx, pentesttools, projectdiscovery, qwant, rapiddns, rocketreach, securityTrails, spyse, sublist3r, threatcrowd, threatminer, trello, twitter, urlscan, virustotal, yahoo, zoomeye"
```
Existem **outras ferramentas/APIs interessantes** que, mesmo não sendo especializadas diretamente na busca de subdomínios, podem ser úteis para encontrar subdomínios, como:

* [**Crobat**](https://github.com/cgboal/sonarsearch)**:** Utiliza a API [https://sonar.omnisint.io](https://sonar.omnisint.io) para obter subdomínios
```bash
# Get list of subdomains in output from the API
## This is the API the crobat tool will use
curl https://sonar.omnisint.io/subdomains/tesla.com | jq -r ".[]"
```
* [**JLDC API gratuita**](https://jldc.me/anubis/subdomains/google.com)
```bash
curl https://jldc.me/anubis/subdomains/tesla.com | jq -r ".[]"
```
* [**RapidDNS**](https://rapiddns.io) API gratuita
```bash
# Get Domains from rapiddns free API
rapiddns(){
curl -s "https://rapiddns.io/subdomain/$1?full=1" \
| grep -oE "[\.a-zA-Z0-9-]+\.$1" \
| sort -u
}
rapiddns tesla.com
```
* [**https://crt.sh/**](https://crt.sh)
```bash
# Get Domains from crt free API
crt(){
curl -s "https://crt.sh/?q=%25.$1" \
| grep -oE "[\.a-zA-Z0-9-]+\.$1" \
| sort -u
}
crt tesla.com
```
* [**gau**](https://github.com/lc/gau)**:** recupera URLs conhecidas do Open Threat Exchange da AlienVault, do Wayback Machine e do Common Crawl para qualquer domínio fornecido.
```bash
# Get subdomains from GAUs found URLs
gau --subs tesla.com | cut -d "/" -f 3 | sort -u
```
* [**SubDomainizer**](https://github.com/nsonaniya2010/SubDomainizer) **&** [**subscraper**](https://github.com/Cillian-Collins/subscraper): Eles vasculham a web em busca de arquivos JS e extraem subdomínios de lá.
```bash
# Get only subdomains from SubDomainizer
python3 SubDomainizer.py -u https://tesla.com | grep tesla.com

# Get only subdomains from subscraper, this already perform recursion over the found results
python subscraper.py -u tesla.com | grep tesla.com | cut -d " " -f
```
* [**Shodan**](https://www.shodan.io/)
```bash
# Get info about the domain
shodan domain <domain>
# Get other pages with links to subdomains
shodan search "http.html:help.domain.com"
```
* [**Censys subdomain finder**](https://github.com/christophetd/censys-subdomain-finder)
```bash
export CENSYS_API_ID=...
export CENSYS_API_SECRET=...
python3 censys-subdomain-finder.py tesla.com
```
* [**DomainTrail.py**](https://github.com/gatete/DomainTrail)
```bash
python3 DomainTrail.py -d example.com
```
* [**securitytrails.com**](https://securitytrails.com/) possui uma API gratuita para pesquisa de subdomínios e histórico de IP
* [**chaos.projectdiscovery.io**](https://chaos.projectdiscovery.io/#/)

Este projeto oferece **gratuitamente todos os subdomínios relacionados a programas de bug-bounty**. Você também pode acessar esses dados usando [chaospy](https://github.com/dr-0x0x/chaospy) ou até mesmo acessar o escopo utilizado por este projeto [https://github.com/projectdiscovery/chaos-public-program-list](https://github.com/projectdiscovery/chaos-public-program-list)

Você pode encontrar uma **comparação** de muitas dessas ferramentas aqui: [https://blog.blacklanternsecurity.com/p/subdomain-enumeration-tool-face-off](https://blog.blacklanternsecurity.com/p/subdomain-enumeration-tool-face-off)

### **DNS Brute force**

Vamos tentar encontrar novos **subdomínios** forçando bruscamente servidores DNS usando nomes de subdomínios possíveis.

Para esta ação, você precisará de algumas **listas de palavras comuns de subdomínios como**:

* [https://gist.github.com/jhaddix/86a06c5dc309d08580a018c66354a056](https://gist.github.com/jhaddix/86a06c5dc309d08580a018c66354a056)
* [https://wordlists-cdn.assetnote.io/data/manual/best-dns-wordlist.txt](https://wordlists-cdn.assetnote.io/data/manual/best-dns-wordlist.txt)
* [https://localdomain.pw/subdomain-bruteforce-list/all.txt.zip](https://localdomain.pw/subdomain-bruteforce-list/all.txt.zip)
* [https://github.com/pentester-io/commonspeak](https://github.com/pentester-io/commonspeak)
* [https://github.com/danielmiessler/SecLists/tree/master/Discovery/DNS](https://github.com/danielmiessler/SecLists/tree/master/Discovery/DNS)

E também IPs de bons resolvedores DNS. Para gerar uma lista de resolvedores DNS confiáveis, você pode baixar os resolvedores de [https://public-dns.info/nameservers-all.txt](https://public-dns.info/nameservers-all.txt) e usar [**dnsvalidator**](https://github.com/vortexau/dnsvalidator) para filtrá-los. Ou você poderia usar: [https://raw.githubusercontent.com/trickest/resolvers/main/resolvers-trusted.txt](https://raw.githubusercontent.com/trickest/resolvers/main/resolvers-trusted.txt)

As ferramentas mais recomendadas para brute-force DNS são:

* [**massdns**](https://github.com/blechschmidt/massdns): Esta foi a primeira ferramenta que realizou um brute-force DNS eficaz. É muito rápida, no entanto, está propensa a falsos positivos.
```bash
sed 's/$/.domain.com/' subdomains.txt > bf-subdomains.txt
./massdns -r resolvers.txt -w /tmp/results.txt bf-subdomains.txt
grep -E "tesla.com. [0-9]+ IN A .+" /tmp/results.txt
```
* [**gobuster**](https://github.com/OJ/gobuster): Este, eu acredito, usa apenas 1 resolvedor
```
gobuster dns -d mysite.com -t 50 -w subdomains.txt
```
* [**shuffledns**](https://github.com/projectdiscovery/shuffledns) é um wrapper em torno do `massdns`, escrito em go, que permite enumerar subdomínios válidos usando bruteforce ativo, bem como resolver subdomínios com tratamento de wildcard e suporte fácil de entrada e saída.
```
shuffledns -d example.com -list example-subdomains.txt -r resolvers.txt
```
* [**puredns**](https://github.com/d3mondev/puredns): Ele também utiliza `massdns`.
```
puredns bruteforce all.txt domain.com
```
* [**aiodnsbrute**](https://github.com/blark/aiodnsbrute) utiliza asyncio para forçar bruscamente nomes de domínio de forma assíncrona.
```
aiodnsbrute -r resolvers -w wordlist.txt -vv -t 1024 domain.com
```
### Segunda Rodada de Brute-Force DNS

Após encontrar subdomínios usando fontes abertas e brute-forcing, você pode gerar alterações dos subdomínios encontrados para tentar descobrir ainda mais. Diversas ferramentas são úteis para esse propósito:

* [**dnsgen**](https://github.com/ProjectAnte/dnsgen)**:** Dado os domínios e subdomínios, gera permutações.
```bash
cat subdomains.txt | dnsgen -
```
* [**goaltdns**](https://github.com/subfinder/goaltdns): Dado os domínios e subdomínios, gera permutações.
* Você pode obter a **wordlist** de permutações do goaltdns **aqui**](https://github.com/subfinder/goaltdns/blob/master/words.txt).
```bash
goaltdns -l subdomains.txt -w /tmp/words-permutations.txt -o /tmp/final-words-s3.txt
```
* [**gotator**](https://github.com/Josue87/gotator)**:** Dado os domínios e subdomínios, gera permutações. Se nenhum arquivo de permutações for indicado, o gotator usará o seu próprio.
```
gotator -sub subdomains.txt -silent [-perm /tmp/words-permutations.txt]
```
* [**altdns**](https://github.com/infosec-au/altdns): Além de gerar permutações de subdomínios, também pode tentar resolvê-los (mas é melhor usar as ferramentas comentadas anteriormente).
* Você pode obter a **wordlist** de permutações do altdns **aqui**](https://github.com/infosec-au/altdns/blob/master/words.txt).
```
altdns -i subdomains.txt -w /tmp/words-permutations.txt -o /tmp/asd3
```
* [**dmut**](https://github.com/bp0lr/dmut): Outra ferramenta para realizar permutações, mutações e alterações de subdomínios. Esta ferramenta executará força bruta no resultado (não suporta coringa dns).
* Você pode obter a lista de palavras de permutações do dmut [**aqui**](https://raw.githubusercontent.com/bp0lr/dmut/main/words.txt).
```bash
cat subdomains.txt | dmut -d /tmp/words-permutations.txt -w 100 \
--dns-errorLimit 10 --use-pb --verbose -s /tmp/resolvers-trusted.txt
```
* [**alterx**](https://github.com/projectdiscovery/alterx)**:** Baseado em um domínio, ele **gera novos possíveis nomes de subdomínios** com base em padrões indicados para tentar descobrir mais subdomínios.

#### Geração inteligente de permutações

* [**regulator**](https://github.com/cramppet/regulator): Para mais informações, leia este [**post**](https://cramppet.github.io/regulator/index.html), mas basicamente ele pega as **partes principais** dos **subdomínios descobertos** e os mistura para encontrar mais subdomínios.
```bash
python3 main.py adobe.com adobe adobe.rules
make_brute_list.sh adobe.rules adobe.brute
puredns resolve adobe.brute --write adobe.valid
```
* [**subzuf**](https://github.com/elceef/subzuf)**:** _subzuf_ é um fuzzer de força bruta de subdomínios acoplado com um algoritmo guiado por resposta DNS extremamente simples, mas eficaz. Utiliza um conjunto de dados de entrada fornecidos, como uma lista de palavras personalizada ou registros históricos de DNS/TLS, para sintetizar com precisão mais nomes de domínio correspondentes e expandi-los ainda mais em um loop baseado nas informações coletadas durante a varredura DNS.
```
echo www | subzuf facebook.com
```
### **Fluxo de Trabalho de Descoberta de Subdomínios**

Confira este post no blog que escrevi sobre como **automatizar a descoberta de subdomínios** de um domínio usando **workflows do Trickest** para que eu não precise iniciar manualmente um monte de ferramentas no meu computador:

{% embed url="https://trickest.com/blog/full-subdomain-discovery-using-workflow/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

{% embed url="https://trickest.com/blog/full-subdomain-brute-force-discovery-using-workflow/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

### **VHosts / Hosts Virtuais**

Se você encontrou um endereço IP contendo **uma ou várias páginas web** pertencentes a subdomínios, você poderia tentar **encontrar outros subdomínios com webs nesse IP** procurando em **fontes OSINT** por domínios em um IP ou por **força bruta nos nomes de domínio VHost nesse IP**.

#### OSINT

Você pode encontrar alguns **VHosts em IPs usando** [**HostHunter**](https://github.com/SpiderLabs/HostHunter) **ou outras APIs**.

**Força Bruta**

Se você suspeita que algum subdomínio pode estar oculto em um servidor web, você poderia tentar forçar a descoberta:
```bash
ffuf -c -w /path/to/wordlist -u http://victim.com -H "Host: FUZZ.victim.com"

gobuster vhost -u https://mysite.com -t 50 -w subdomains.txt

wfuzz -c -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-20000.txt --hc 400,404,403 -H "Host: FUZZ.example.com" -u http://example.com -t 100

#From https://github.com/allyshka/vhostbrute
vhostbrute.py --url="example.com" --remoteip="10.1.1.15" --base="www.example.com" --vhosts="vhosts_full.list"

#https://github.com/codingo/VHostScan
VHostScan -t example.com
```
{% hint style="info" %}
Com esta técnica, você pode até conseguir acessar endpoints internos/ocultos.
{% endhint %}

### **CORS Brute Force**

Às vezes, você encontrará páginas que retornam o cabeçalho _**Access-Control-Allow-Origin**_ apenas quando um domínio/subdomínio válido é definido no cabeçalho _**Origin**_. Nestes cenários, você pode abusar deste comportamento para **descobrir** novos **subdomínios**.
```bash
ffuf -w subdomains-top1million-5000.txt -u http://10.10.10.208 -H 'Origin: http://FUZZ.crossfit.htb' -mr "Access-Control-Allow-Origin" -ignore-body
```
### **Força Bruta em Buckets**

Ao procurar por **subdomínios**, fique atento para ver se está **apontando** para algum tipo de **bucket**, e nesse caso, [**verifique as permissões**](../../network-services-pentesting/pentesting-web/buckets/)**.**\
Além disso, como neste ponto você já conhecerá todos os domínios dentro do escopo, tente [**forçar a bruta de possíveis nomes de buckets e verificar as permissões**](../../network-services-pentesting/pentesting-web/buckets/).

### **Monitoramento**

Você pode **monitorar** se **novos subdomínios** de um domínio são criados monitorando os Logs de **Transparência de Certificados** [**sublert**](https://github.com/yassineaboukir/sublert/blob/master/sublert.py) faz isso.

### **Procurando por vulnerabilidades**

Verifique a possibilidade de [**tomada de subdomínio**](../../pentesting-web/domain-subdomain-takeover.md#subdomain-takeover).\
Se o **subdomínio** estiver apontando para algum **bucket S3**, [**verifique as permissões**](../../network-services-pentesting/pentesting-web/buckets/).

Se você encontrar algum **subdomínio com um IP diferente** dos que você já encontrou na descoberta de ativos, você deve realizar um **scan de vulnerabilidade básico** (usando Nessus ou OpenVAS) e algum [**scan de portas**](../pentesting-network/#discovering-hosts-from-the-outside) com **nmap/masscan/shodan**. Dependendo de quais serviços estão rodando, você pode encontrar **neste livro algumas dicas para "atacá-los"**.\
_Note que às vezes o subdomínio está hospedado dentro de um IP que não é controlado pelo cliente, então não está no escopo, tenha cuidado._

## IPs

Nas etapas iniciais, você pode ter **encontrado alguns intervalos de IP, domínios e subdomínios**.\
É hora de **recolher todos os IPs desses intervalos** e para os **domínios/subdomínios (consultas DNS).**

Usando serviços das seguintes **apis gratuitas**, você também pode encontrar **IPs anteriores usados por domínios e subdomínios**. Esses IPs ainda podem ser de propriedade do cliente (e podem permitir que você encontre [**bypasses do CloudFlare**](../../network-services-pentesting/pentesting-web/uncovering-cloudflare.md))

* [**https://securitytrails.com/**](https://securitytrails.com/)

Você também pode verificar domínios que apontam para um endereço IP específico usando a ferramenta [**hakip2host**](https://github.com/hakluke/hakip2host)

### **Procurando por vulnerabilidades**

**Faça um scan de portas em todos os IPs que não pertencem a CDNs** (já que é muito provável que você não encontre nada interessante lá). Nos serviços em execução descobertos, você pode ser **capaz de encontrar vulnerabilidades**.

**Encontre um** [**guia**](../pentesting-network/) **sobre como escanear hosts.**

## Caça a servidores web

> Encontramos todas as empresas e seus ativos e sabemos os intervalos de IP, domínios e subdomínios dentro do escopo. É hora de procurar por servidores web.

Nas etapas anteriores, você provavelmente já realizou algum **reconhecimento dos IPs e domínios descobertos**, então você pode ter **já encontrado todos os possíveis servidores web**. No entanto, se você não o fez, agora vamos ver algumas **dicas rápidas para procurar servidores web** dentro do escopo.

Por favor, note que isso será **orientado para a descoberta de aplicativos web**, então você também deve **realizar a varredura de vulnerabilidades** e **scan de portas** (**se permitido** pelo escopo).

Um **método rápido** para descobrir **portas abertas** relacionadas a servidores **web** usando [**masscan** pode ser encontrado aqui](../pentesting-network/#http-port-discovery).\
Outra ferramenta amigável para procurar servidores web é [**httprobe**](https://github.com/tomnomnom/httprobe)**,** [**fprobe**](https://github.com/theblackturtle/fprobe) e [**httpx**](https://github.com/projectdiscovery/httpx). Você apenas passa uma lista de domínios e ele tentará se conectar à porta 80 (http) e 443 (https). Além disso, você pode indicar para tentar outras portas:
```bash
cat /tmp/domains.txt | httprobe #Test all domains inside the file for port 80 and 443
cat /tmp/domains.txt | httprobe -p http:8080 -p https:8443 #Check port 80, 443 and 8080 and 8443
```
### **Screenshots**

Agora que você descobriu **todos os servidores web** presentes no escopo (entre os **IPs** da empresa e todos os **domínios** e **subdomínios**), você provavelmente **não sabe por onde começar**. Então, vamos simplificar e começar apenas tirando screenshots de todos eles. Apenas **dando uma olhada** na **página principal**, você pode encontrar **endpoints estranhos** que são mais **propensos** a serem **vulneráveis**.

Para realizar a ideia proposta, você pode usar [**EyeWitness**](https://github.com/FortyNorthSecurity/EyeWitness), [**HttpScreenshot**](https://github.com/breenmachine/httpscreenshot), [**Aquatone**](https://github.com/michenriksen/aquatone), [**Shutter**](https://shutter-project.org/downloads/third-party-packages/) ou [**webscreenshot**](https://github.com/maaaaz/webscreenshot)**.**

Além disso, você poderia então usar [**eyeballer**](https://github.com/BishopFox/eyeballer) para analisar todos os **screenshots** e dizer **o que provavelmente contém vulnerabilidades**, e o que não contém.

## Ativos de Nuvem Pública

Para encontrar possíveis ativos de nuvem pertencentes a uma empresa, você deve **começar com uma lista de palavras-chave que identifiquem essa empresa**. Por exemplo, para uma empresa de criptomoedas, você pode usar palavras como: `"crypto", "wallet", "dao", "<nome_do_domínio>", <"nomes_de_subdomínios">`.

Você também precisará de listas de palavras de **palavras comuns usadas em buckets**:

* [https://raw.githubusercontent.com/cujanovic/goaltdns/master/words.txt](https://raw.githubusercontent.com/cujanovic/goaltdns/master/words.txt)
* [https://raw.githubusercontent.com/infosec-au/altdns/master/words.txt](https://raw.githubusercontent.com/infosec-au/altdns/master/words.txt)
* [https://raw.githubusercontent.com/jordanpotti/AWSBucketDump/master/BucketNames.txt](https://raw.githubusercontent.com/jordanpotti/AWSBucketDump/master/BucketNames.txt)

Então, com essas palavras, você deve gerar **permutações** (confira a [**Segunda Rodada de Força Bruta em DNS**](./#second-dns-bruteforce-round) para mais informações).

Com as listas de palavras resultantes, você poderia usar ferramentas como [**cloud\_enum**](https://github.com/initstring/cloud\_enum)**,** [**CloudScraper**](https://github.com/jordanpotti/CloudScraper)**,** [**cloudlist**](https://github.com/projectdiscovery/cloudlist) **ou** [**S3Scanner**](https://github.com/sa7mon/S3Scanner)**.**

Lembre-se de que, ao procurar por Ativos de Nuvem, você deve **procurar por mais do que apenas buckets na AWS**.

### **Procurando por vulnerabilidades**

Se você encontrar coisas como **buckets abertos ou funções de nuvem expostas**, você deve **acessá-las** e tentar ver o que elas oferecem e se você pode abusar delas.

## Emails

Com os **domínios** e **subdomínios** dentro do escopo, você basicamente tem tudo o que **precisa para começar a procurar por emails**. Estas são as **APIs** e **ferramentas** que funcionaram melhor para mim para encontrar emails de uma empresa:

* [**theHarvester**](https://github.com/laramies/theHarvester) - com APIs
* API de [**https://hunter.io/**](https://hunter.io/) (versão gratuita)
* API de [**https://app.snov.io/**](https://app.snov.io/) (versão gratuita)
* API de [**https://minelead.io/**](https://minelead.io/) (versão gratuita)

### **Procurando por vulnerabilidades**

Emails serão úteis mais tarde para **força bruta em logins web e serviços de autenticação** (como SSH). Além disso, são necessários para **phishings**. Além disso, essas APIs fornecerão ainda mais **informações sobre a pessoa** por trás do email, o que é útil para a campanha de phishing.

## Vazamentos de Credenciais

Com os **domínios**, **subdomínios** e **emails**, você pode começar a procurar por credenciais vazadas no passado pertencentes a esses emails:

* [https://leak-lookup.com](https://leak-lookup.com/account/login)
* [https://www.dehashed.com/](https://www.dehashed.com/)

### **Procurando por vulnerabilidades**

Se você encontrar **credenciais vazadas válidas**, isso é uma vitória muito fácil.

## Vazamentos de Segredos

Vazamentos de credenciais estão relacionados a hacks de empresas onde **informações sensíveis foram vazadas e vendidas**. No entanto, as empresas podem ser afetadas por **outros vazamentos** cujas informações não estão nessas bases de dados:

### Vazamentos no Github

Credenciais e APIs podem ser vazadas nos **repositórios públicos** da **empresa** ou dos **usuários** que trabalham pela empresa no github.\
Você pode usar a **ferramenta** [**Leakos**](https://github.com/carlospolop/Leakos) para **baixar** todos os **repositórios públicos** de uma **organização** e de seus **desenvolvedores** e executar [**gitleaks**](https://github.com/zricethezav/gitleaks) sobre eles automaticamente.

**Leakos** também pode ser usado para executar **gitleaks** contra todos os **textos** fornecidos **URLs passadas** para ele, pois às vezes **páginas da web também contêm segredos**.

#### Dorks do Github

Confira também esta **página** para possíveis **dorks do github** que você também poderia procurar na organização que está atacando:

{% content-ref url="github-leaked-secrets.md" %}
[github-leaked-secrets.md](github-leaked-secrets.md)
{% endcontent-ref %}

### Vazamentos em Pastes

Às vezes, atacantes ou apenas trabalhadores irão **publicar conteúdo da empresa em um site de paste**. Isso pode ou não conter **informações sensíveis**, mas é muito interessante procurar por isso.\
Você pode usar a ferramenta [**Pastos**](https://github.com/carlospolop/Pastos) para pesquisar em mais de 80 sites de paste ao mesmo tempo.

### Dorks do Google

Os antigos, mas eficazes dorks do Google são sempre úteis para encontrar **informações expostas que não deveriam estar lá**. O único problema é que a [**google-hacking-database**](https://www.exploit-db.com/google-hacking-database) contém vários **milhares** de possíveis consultas que você não pode executar manualmente. Então, você pode escolher suas 10 favoritas ou você poderia usar uma **ferramenta como** [**Gorks**](https://github.com/carlospolop/Gorks) **para executá-las todas**.

_Note que as ferramentas que esperam executar toda a base de dados usando o navegador Google regular nunca terminarão, pois o Google irá bloqueá-lo muito em breve._

### **Procurando por vulnerabilidades**

Se você encontrar **credenciais vazadas válidas** ou tokens de API, isso é uma vitória muito fácil.

## Vulnerabilidades em Código Público

Se você descobriu que a empresa tem **código de fonte aberta**, você pode **analisar** e procurar por **vulnerabilidades** nele.

**Dependendo da linguagem** existem diferentes **ferramentas** que você pode usar:

{% content-ref url="../../network-services-pentesting/pentesting-web/code-review-tools.md" %}
[code-review-tools.md](../../network-services-pentesting/pentesting-web/code-review-tools.md)
{% endcontent-ref %}

Também existem serviços gratuitos que permitem **analisar repositórios públicos**, como:

* [**Snyk**](https://app.snyk.io/)

## [**Metodologia de Pentesting Web**](../../network-services-pentesting/pentesting-web/)

A **maioria das vulnerabilidades** encontradas por caçadores de bugs reside dentro de **aplicações web**, então neste ponto eu gostaria de falar sobre uma **metodologia de teste de aplicação web**, e você pode [**encontrar essa informação aqui**](../../network-services-pentesting/pentesting-web/).

Também quero fazer uma menção especial à seção [**Ferramentas de Scanners Automatizados de Web open source**](../../network-services-pentesting/pentesting-web/#automatic-scanners), pois, se você não deve esperar que elas encontrem vulnerabilidades muito sensíveis, elas são úteis para implementá-las em **fluxos de trabalho para ter alguma informação web inicial.**

## Recapitulação

> Parabéns! Neste ponto, você já realizou **toda a enumeração básica**. Sim, é básico porque muito mais enumeração pode ser feita (veremos mais truques mais tarde).

Então você já:

1. Encontrou todas as **empresas** dentro do escopo
2. Encontrou todos os **ativos** pertencentes às empresas (e realizou alguma varredura de vulnerabilidade, se estiver no escopo)
3. Encontrou todos os **domínios** pertencentes às empresas
4. Encontrou todos os **subdomínios** dos domínios (alguma tomada de subdomínio?)
5. Encontrou todos os **IPs** (de e **não de CDNs**) dentro do escopo.
6. Encontrou todos os **servidores web** e tirou um **screenshot** deles (algo estranho que mereça um olhar mais aprofundado?)
7. Encontrou todos os **ativos de nuvem pública** potenciais pertencentes à empresa.
8. **Emails**, **vazamentos de credenciais** e **vazamentos de segredos** que poderiam dar-lhe uma **grande vitória muito facilmente**.
9. **Pentesting em todos os webs que você encontrou**

## **Ferramentas Automáticas de Reconhecimento Completo**

Existem várias ferramentas disponíveis que realizarão parte das ações propostas contra um determinado escopo.

* [**https://github.com/yogeshojha/rengine**](https://github.com/yogeshojha/rengine)
* [**https://github.com/j3ssie/Osmedeus**](https://github.com/j3ssie/Osmedeus)
* [**https://github.com/six2dez/reconftw**](https://github.com/six2dez/reconftw)
* [**https://github.com/hackerspider1/EchoPwn**](https://github.com/hackerspider1/EchoPwn) - Um pouco antigo e não atualizado

## **Referências**

* **Todos os cursos gratuitos de** [**@Jhaddix**](https://twitter.com/Jhaddix) **(como** [**The Bug Hunter's Methodology v4.0 - Recon Edition**](https://www.youtube.com/watch?v=p4JgIu1mceI)**)**

<img src="../../.gitbook/assets/i3.png" alt="" data-size="original">\
**Dica de caça a bugs**: **inscreva-se** no **Intigriti**, uma plataforma premium de **caça a bugs criada por hackers, para hackers**! Junte-se a nós em [**https://go.intigriti.com/hacktricks**](https://go.intigriti.com/hacktricks) hoje e comece a ganhar recompensas de até **$100,000**!

{% embed url="https://go.intigriti.com/hacktricks" %}

<details>

<summary><strong>Aprenda hacking na AWS do zero ao herói com</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Outras maneiras de apoiar o HackTricks:

* Se você quiser ver sua **empresa anunciada no HackTricks** ou **baixar o HackTricks em PDF**, confira os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Obtenha o [**merchandising oficial do PEASS & HackTricks**](https://peass.creator-spring.com)
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção de [**NFTs**](https://opensea.io/collection/the-peass-family) exclusivos
* **Junte-se ao** 💬 [**grupo do Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo do telegram**](https://t.me/peass) ou **siga**-me no **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **Compartilhe suas dicas de hacking enviando PRs para os repositórios do github** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
