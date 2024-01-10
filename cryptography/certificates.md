# Certificados

<details>

<summary><strong>Aprenda hacking no AWS do zero ao herói com</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Outras formas de apoiar o HackTricks:

* Se você quer ver sua **empresa anunciada no HackTricks** ou **baixar o HackTricks em PDF**, confira os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Adquira o [**material oficial PEASS & HackTricks**](https://peass.creator-spring.com)
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção de [**NFTs**](https://opensea.io/collection/the-peass-family) exclusivos
* **Junte-se ao grupo** 💬 [**Discord**](https://discord.gg/hRep4RUj7f) ou ao grupo [**telegram**](https://t.me/peass) ou **siga-me** no **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **Compartilhe suas técnicas de hacking enviando PRs para os repositórios do GitHub** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>

<figure><img src="../.gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Use [**Trickest**](https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks) para construir e **automatizar fluxos de trabalho** com facilidade, utilizando as ferramentas comunitárias **mais avançadas** do mundo.\
Obtenha Acesso Hoje:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## O que é um Certificado

Em criptografia, um **certificado de chave pública**, também conhecido como **certificado digital** ou **certificado de identidade**, é um documento eletrônico usado para comprovar a propriedade de uma chave pública. O certificado inclui informações sobre a chave, informações sobre a identidade do proprietário (chamado de sujeito) e a assinatura digital de uma entidade que verificou o conteúdo do certificado (chamado de emissor). Se a assinatura for válida e o software que examina o certificado confiar no emissor, então ele pode usar essa chave para se comunicar de forma segura com o sujeito do certificado.

Em um esquema típico de [infraestrutura de chave pública](https://en.wikipedia.org/wiki/Public-key_infrastructure) (PKI), o emissor do certificado é uma [autoridade de certificação](https://en.wikipedia.org/wiki/Certificate_authority) (CA), geralmente uma empresa que cobra dos clientes para emitir certificados para eles. Em contraste, em um esquema de [teia de confiança](https://en.wikipedia.org/wiki/Web_of_trust), indivíduos assinam diretamente as chaves uns dos outros, em um formato que desempenha uma função semelhante a um certificado de chave pública.

O formato mais comum para certificados de chave pública é definido por [X.509](https://en.wikipedia.org/wiki/X.509). Como X.509 é muito geral, o formato é ainda mais restrito por perfis definidos para certos casos de uso, como [Infraestrutura de Chave Pública (X.509)](https://en.wikipedia.org/wiki/PKIX) conforme definido no RFC 5280.

## Campos Comuns x509

* **Número da Versão:** Versão do formato x509.
* **Número de Série**: Usado para identificar unicamente o certificado dentro dos sistemas de uma CA. Em particular, isso é usado para rastrear informações de revogação.
* **Sujeito**: A entidade a qual um certificado pertence: uma máquina, um indivíduo ou uma organização.
* **Nome Comum**: Domínios afetados pelo certificado. Pode ser 1 ou mais e pode conter curingas.
* **País (C)**: País
* **Nome Distinto (DN)**: O sujeito completo: `C=US, ST=California, L=San Francisco, O=Example, Inc., CN=shared.global.example.net`
* **Localidade (L)**: Localidade
* **Organização (O)**: Nome da organização
* **Unidade Organizacional (OU)**: Divisão de uma organização (como "Recursos Humanos").
* **Estado ou Província (ST, S ou P)**: Lista de nomes de estados ou províncias
* **Emissor**: A entidade que verificou as informações e assinou o certificado.
* **Nome Comum (CN)**: Nome da autoridade de certificação
* **País (C)**: País da autoridade de certificação
* **Nome Distinto (DN)**: Nome distinto da autoridade de certificação
* **Localidade (L)**: Localidade onde a organização pode ser encontrada.
* **Organização (O)**: Nome da organização
* **Unidade Organizacional (OU)**: Divisão de uma organização (como "Recursos Humanos").
* **Não Antes**: A data e hora mais cedo em que o certificado é válido. Geralmente definido para algumas horas ou dias antes do momento em que o certificado foi emitido, para evitar problemas de [desvio de relógio](https://en.wikipedia.org/wiki/Clock_skew#On_a_network).
* **Não Depois**: A data e hora após a qual o certificado não é mais válido.
* **Chave Pública**: Uma chave pública pertencente ao sujeito do certificado. (Esta é uma das partes principais, pois é o que é assinado pela CA)
* **Algoritmo da Chave Pública**: Algoritmo usado para gerar a chave pública. Como RSA.
* **Curva da Chave Pública**: A curva usada pelo algoritmo de chave pública de curva elíptica (se aplicável). Como nistp521.
* **Expoente da Chave Pública**: Expoente usado para derivar a chave pública (se aplicável). Como 65537.
* **Tamanho da Chave Pública**: O tamanho do espaço da chave pública em bits. Como 2048.
* **Algoritmo de Assinatura**: O algoritmo usado para assinar o certificado de chave pública.
* **Assinatura**: Uma assinatura do corpo do certificado pela chave privada do emissor.
* **extensões x509v3**
* **Uso da Chave**: Os usos criptográficos válidos da chave pública do certificado. Valores comuns incluem validação de assinatura digital, ciframento de chave e assinatura de certificado.
* Em um certificado Web, isso aparecerá como uma _extensão X509v3_ e terá o valor `Assinatura Digital`
* **Uso Estendido da Chave**: As aplicações nas quais o certificado pode ser usado. Valores comuns incluem autenticação de servidor TLS, proteção de e-mail e assinatura de código.
* Em um certificado Web, isso aparecerá como uma _extensão X509v3_ e terá o valor `Autenticação de Servidor Web TLS`
* **Nome Alternativo do Sujeito:** Permite que os usuários especifiquem nomes de host adicionais para um único certificado SSL. O uso da extensão SAN é prática padrão para certificados SSL e está a caminho de substituir o uso do nome comum.
* **Restrição Básica:** Esta extensão descreve se o certificado é um certificado de CA ou um certificado de entidade final. Um certificado de CA é algo que assina certificados de outros e um certificado de entidade final é o certificado usado em uma página da web, por exemplo (a última parte da cadeia).
* **Identificador de Chave do Sujeito** (SKI): Esta extensão declara um identificador único para a chave pública no certificado. É obrigatório em todos os certificados de CA. As CAs propagam seu próprio SKI para a extensão Identificador de Chave do Emissor (AKI) nos certificados emitidos. É o hash da chave pública do sujeito.
* **Identificador de Chave da Autoridade**: Contém um identificador de chave que é derivado da chave pública no certificado do emissor. É o hash da chave pública do emissor.
* **Acesso à Informação da Autoridade** (AIA): Esta extensão contém no máximo dois tipos de informações:
* Informações sobre **como obter o emissor deste certificado** (método de acesso ao emissor da CA)
* Endereço do **responder OCSP de onde a revogação deste certificado** pode ser verificada (método de acesso OCSP).
* **Pontos de Distribuição de CRL**: Esta extensão identifica a localização da CRL de onde a revogação deste certificado pode ser verificada. O aplicativo que processa o certificado pode obter a localização da CRL a partir desta extensão, baixar a CRL e então verificar a revogação deste certificado.
* **CT Precertificate SCTs**: Logs de Transparência de Certificado em relação ao certificado

### Diferença entre OCSP e Pontos de Distribuição de CRL

**OCSP** (RFC 2560) é um protocolo padrão que consiste em um **cliente OCSP e um respondedor OCSP**. Este protocolo **determina o status de revogação de um dado certificado de chave pública digital** **sem** ter que **baixar** a **CRL inteira**.\
**CRL** é o **método tradicional** de verificar a validade do certificado. Uma **CRL fornece uma lista de números de série de certificados** que foram revogados ou não são mais válidos. As CRLs permitem que o verificador verifique o status de revogação do certificado apresentado durante a verificação. As CRLs são limitadas a 512 entradas.\
De [aqui](https://www.arubanetworks.com/techdocs/ArubaOS%206_3_1_Web_Help/Content/ArubaFrameStyles/CertRevocation/About_OCSP_and_CRL.htm).

### O que é Transparência de Certificado

Transparência de Certificado visa remediar ameaças baseadas em certificados **tornando a emissão e existência de certificados SSL abertos ao escrutínio por proprietários de domínios, CAs e usuários de domínios**. Especificamente, a Transparência de Certificado tem três objetivos principais:

* Tornar impossível (ou pelo menos muito difícil) para uma CA **emitir um certificado SSL para um domínio sem que o certificado esteja visível para o proprietário** desse domínio.
* Fornecer um **sistema aberto de auditoria e monitoramento que permite a qualquer proprietário de domínio ou CA determinar se certificados foram emitidos erroneamente ou maliciosamente**.
* **Proteger os usuários** (tanto quanto possível) de serem enganados por certificados que foram emitidos erroneamente ou maliciosamente.

#### **Logs de Certificados**

Logs de certificados são serviços de rede simples que mantêm **registros de certificados assegurados criptograficamente, publicamente auditáveis e apenas de adição**. **Qualquer um pode submeter certificados a um log**, embora as autoridades de certificação provavelmente sejam os principais submissores. Da mesma forma, qualquer um pode consultar um log para uma prova criptográfica, que pode ser usada para verificar se o log está se comportando corretamente ou verificar se um certificado específico foi registrado. O número de servidores de log não precisa ser grande (digamos, muito menos que mil em todo o mundo), e cada um pode ser operado independentemente por uma CA, um ISP ou qualquer outra parte interessada.

#### Consulta

Você pode consultar os logs de Transparência de Certificado de qualquer domínio em [https://crt.sh/](https://crt.sh).

## Formatos

Existem diferentes formatos que podem ser usados para armazenar um certificado.

#### **Formato PEM**

* É o formato mais comum usado para certificados
* A maioria dos servidores (Ex: Apache) espera que os certificados e a chave privada estejam em arquivos separados\
\- Geralmente são arquivos ASCII codificados em Base64\
\- Extensões usadas para certificados PEM são .cer, .crt, .pem, .key\
\- Apache e servidores semelhantes usam certificados no formato PEM

#### **Formato DER**

* O formato DER é a forma binária do certificado
* Todos os tipos de certificados e chaves privadas podem ser codificados no formato DER
* Certificados formatados em DER não contêm as declarações "BEGIN CERTIFICATE/END CERTIFICATE"
* Certificados formatados em DER usam mais frequentemente as extensões ‘.cer’ e '.der'
* DER é tipicamente usado em Plataformas Java

#### **Formato P7B/PKCS#7**

* O formato PKCS#7 ou P7B é armazenado em formato ASCII Base64 e tem uma extensão de arquivo de .p7b ou .p7c
* Um arquivo P7B contém apenas certificados e certificados de cadeia (CAs Intermediárias), não a chave privada
* As plataformas mais comuns que suportam arquivos P7B são Microsoft Windows e Java Tomcat

#### **Formato PFX/P12/PKCS#12**

* O formato PKCS#12 ou PFX/P12 é um formato binário para armazenar o certificado do servidor, certificados intermediários e a chave privada em um único arquivo criptografável
* Esses arquivos geralmente têm extensões como .pfx e .p12
* Eles são tipicamente usados em máquinas Windows para importar e exportar certificados e chaves privadas

### Conversões de formatos

**Converter x509 para PEM**
```
openssl x509 -in certificatename.cer -outform PEM -out certificatename.pem
```
#### **Converter PEM para DER**
```
openssl x509 -outform der -in certificatename.pem -out certificatename.der
```
**Converter DER para PEM**
```
openssl x509 -inform der -in certificatename.der -out certificatename.pem
```
**Converter PEM para P7B**

**Nota:** O formato PKCS#7 ou P7B é armazenado em formato ASCII Base64 e possui uma extensão de arquivo .p7b ou .p7c. Um arquivo P7B contém apenas certificados e cadeias de certificados (CAs Intermediários), não a chave privada. As plataformas mais comuns que suportam arquivos P7B são Microsoft Windows e Java Tomcat.
```
openssl crl2pkcs7 -nocrl -certfile certificatename.pem -out certificatename.p7b -certfile CACert.cer
```
**Converter PKCS7 para PEM**
```
openssl pkcs7 -print_certs -in certificatename.p7b -out certificatename.pem
```
**Converter pfx para PEM**

**Nota:** O formato PKCS#12 ou PFX é um formato binário para armazenar o certificado do servidor, certificados intermediários e a chave privada em um único arquivo criptografável. Arquivos PFX geralmente possuem extensões como .pfx e .p12. Arquivos PFX são tipicamente usados em máquinas Windows para importar e exportar certificados e chaves privadas.
```
openssl pkcs12 -in certificatename.pfx -out certificatename.pem
```
**Converter PFX para PKCS#8**\
**Nota:** Isso requer 2 comandos

**1- Converter PFX para PEM**
```
openssl pkcs12 -in certificatename.pfx -nocerts -nodes -out certificatename.pem
```
**2- Converter PEM para PKCS8**
```
openSSL pkcs8 -in certificatename.pem -topk8 -nocrypt -out certificatename.pk8
```
**Converter P7B para PFX**\
**Nota:** Isso requer 2 comandos

1- **Converter P7B para CER**
```
openssl pkcs7 -print_certs -in certificatename.p7b -out certificatename.cer
```
**2- Converter CER e Chave Privada para PFX**
```
openssl pkcs12 -export -in certificatename.cer -inkey privateKey.key -out certificatename.pfx -certfile  cacert.cer
```
<figure><img src="../.gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Use [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) para construir e **automatizar fluxos de trabalho** com as ferramentas comunitárias **mais avançadas** do mundo.\
Obtenha Acesso Hoje:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><strong>Aprenda a hackear AWS do zero ao herói com</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Outras formas de apoiar o HackTricks:

* Se você quer ver sua **empresa anunciada no HackTricks** ou **baixar o HackTricks em PDF**, confira os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Adquira o [**material oficial PEASS & HackTricks**](https://peass.creator-spring.com)
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção de [**NFTs**](https://opensea.io/collection/the-peass-family) exclusivos
* **Junte-se ao grupo** 💬 [**Discord**](https://discord.gg/hRep4RUj7f) ou ao grupo [**telegram**](https://t.me/peass) ou **siga-me** no **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **Compartilhe suas técnicas de hacking enviando PRs para os repositórios do GitHub** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
