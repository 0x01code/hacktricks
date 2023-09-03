# Certificados

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* Você trabalha em uma **empresa de segurança cibernética**? Você quer ver sua **empresa anunciada no HackTricks**? ou você quer ter acesso à **última versão do PEASS ou baixar o HackTricks em PDF**? Verifique os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* Adquira o [**swag oficial do PEASS & HackTricks**](https://peass.creator-spring.com)
* **Junte-se ao** [**💬**](https://emojipedia.org/speech-balloon/) [**grupo Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo telegram**](https://t.me/peass) ou **siga-me** no **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Compartilhe seus truques de hacking enviando PRs para o** [**repositório hacktricks**](https://github.com/carlospolop/hacktricks) **e** [**repositório hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>

<figure><img src="/.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

\
Use [**Trickest**](https://trickest.io/) para construir e **automatizar fluxos de trabalho** com as ferramentas comunitárias mais avançadas do mundo.\
Acesse hoje:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## O que é um Certificado

Em criptografia, um **certificado de chave pública**, também conhecido como **certificado digital** ou **certificado de identidade**, é um documento eletrônico usado para comprovar a propriedade de uma chave pública. O certificado inclui informações sobre a chave, informações sobre a identidade do seu proprietário (chamado de sujeito) e a assinatura digital de uma entidade que verificou o conteúdo do certificado (chamada de emissor). Se a assinatura for válida e o software que examina o certificado confiar no emissor, ele pode usar essa chave para se comunicar de forma segura com o sujeito do certificado.

Em um esquema típico de [infraestrutura de chave pública](https://en.wikipedia.org/wiki/Public-key\_infrastructure) (PKI), o emissor do certificado é uma [autoridade de certificação](https://en.wikipedia.org/wiki/Certificate\_authority) (CA), geralmente uma empresa que cobra dos clientes para emitir certificados para eles. Em contraste, em um esquema de [rede de confiança](https://en.wikipedia.org/wiki/Web\_of\_trust), os indivíduos assinam as chaves uns dos outros diretamente, em um formato que desempenha uma função semelhante a um certificado de chave pública.

O formato mais comum para certificados de chave pública é definido por [X.509](https://en.wikipedia.org/wiki/X.509). Como o X.509 é muito geral, o formato é ainda mais restrito por perfis definidos para determinados casos de uso, como [Infraestrutura de Chave Pública (X.509)](https://en.wikipedia.org/wiki/PKIX) conforme definido no RFC 5280.

## Campos Comuns do x509

* **Número da Versão:** Versão do formato x509.
* **Número Serial**: Usado para identificar unicamente o certificado nos sistemas de uma CA. Em particular, isso é usado para rastrear informações de revogação.
* **Sujeito**: A entidade a qual o certificado pertence: uma máquina, um indivíduo ou uma organização.
* **Nome Comum**: Domínios afetados pelo certificado. Pode ser 1 ou mais e pode conter curingas.
* **País (C)**: País
* **Nome Distinto (DN)**: O sujeito completo: `C=US, ST=California, L=San Francisco, O=Example, Inc., CN=shared.global.example.net`
* **Localidade (L)**: Local
* **Organização (O)**: Nome da organização
* **Unidade Organizacional (OU)**: Divisão de uma organização (como "Recursos Humanos").
* **Estado ou Província (ST, S ou P)**: Lista de nomes de estado ou província
* **Emissor**: A entidade que verificou as informações e assinou o certificado.
* **Nome Comum (CN)**: Nome da autoridade de certificação
* **País (C)**: País da autoridade de certificação
* **Nome Distinto (DN)**: Nome distinto da autoridade de certificação
* **Localidade (L)**: Local onde a organização pode ser encontrada.
* **Organização (O)**: Nome da organização
* **Unidade Organizacional (OU)**: Divisão de uma organização (como "Recursos Humanos").
* **Não Antes**: A data e hora mais cedo em que o certificado é válido. Geralmente definido algumas horas ou dias antes do momento em que o certificado foi emitido, para evitar problemas de [diferença de horário](https://en.wikipedia.org/wiki/Clock\_skew#On\_a\_network).
* **Não Depois**: A data e hora após as quais o certificado não é mais válido.
* **Chave Pública**: Uma chave pública pertencente ao sujeito do certificado. (Esta é uma das partes principais, pois é isso que é assinado pela CA)
* **Algoritmo de Chave Pública**: Algoritmo usado para gerar a chave pública. Como RSA.
* **Curva da Chave Pública**: A curva usada pelo algoritmo de chave pública de curva elíptica (se aplicável). Como nistp521.
* **Expoente da Chave Pública**: Expoente usado para derivar a chave pública (se aplicável). Como 65537.
* **Tamanho da Chave Pública**: O tamanho do espaço da chave pública em bits. Como 2048.
* **Algoritmo de Assinatura**: O algoritmo usado para assinar o certificado de chave pública.
* **Assinatura**: Uma assinatura do corpo do certificado pela chave privada do emissor.
* **Extensões x509v3**
* **Uso da Chave**: Os usos criptográficos válidos da chave pública do certificado. Valores comuns incluem validação de assinatura digital, cifragem de chave e assinatura de certificado.
* Em um certificado da Web, isso aparecerá como uma _extensão X509v3_ e terá o valor `Digital Signature`
* **Uso Estendido da Chave**: As aplicações em que o certificado pode ser usado. Valores comuns incluem autenticação de servidor TLS, proteção de e-mail e assinatura de código.
* Em um certificado da Web, isso aparecerá como uma _extensão X509v3_ e terá o valor `TLS Web Server Authentication`
* **Nome Alternativo do Sujeito:** Permite que os usuários especifiquem nomes adicionais de host para um único **certificado** SSL. O uso da extensão SAN é uma prática padrão para certificados SSL e está substituindo o uso do **nome** comum.
* **Restrição Básica:** Essa extensão descreve se o certificado é um certificado de CA ou um certificado de entidade final. Um certificado de CA é algo que assina certificados de outras pessoas e um certificado de entidade final é o certificado usado em uma página da web, por exemplo (a última parte da cadeia).
* **Identificador da Chave do Sujeito** (SKI): Essa extensão declara um **identificador** único para a **chave** pública no certificado. É necessário em todos os certificados de CA. As CAs propagam seu próprio SKI para a extensão Identificador de Chave do Emissor (AKI) nos certificados emitidos. É o hash da chave pública do sujeito.
* **Identificador de Chave de Autoridade**: Contém um identificador de chave derivado da chave pública no certificado do emissor. É o hash da chave pública do emissor.
* **Acesso à Informação da Autoridade** (AIA): Esta extensão contém no máximo dois tipos de informações:
* Informações sobre **como obter o emissor deste certificado** (método de acesso ao emissor CA)
* Endereço do **responder OCSP de onde a revogação deste certificado** pode ser verificada (método de acesso OCSP).
* **Pontos de Distribuição de CRL**: Esta extensão identifica a localização da CRL da qual a revogação deste certificado pode ser verificada. A aplicação que processa o certificado pode obter a localização da CRL desta extensão, baixar a CRL e então verificar a revogação deste certificado.
* **CT Precertificate SCTs**: Logs de Transparência de Certificado referentes ao certificado

### Diferença entre OCSP e Pontos de Distribuição de CRL

**OCSP** (RFC 2560) é um protocolo padrão que consiste em um **cliente OCSP e um responder OCSP**. Este protocolo **determina o status de revogação de um determinado certificado de chave pública digital** **sem precisar** baixar a **CRL inteira**.\
**CRL** é o **método tradicional** de verificação da validade do certificado. Uma **CRL fornece uma lista de números de série de certificados** que foram revogados ou não são mais válidos. As CRLs permitem que o verificador verifique o status de revogação do certificado apresentado durante a verificação. As CRLs são limitadas a 512 entradas.\
De [aqui](https://www.arubanetworks.com/techdocs/ArubaOS%206\_3\_1\_Web\_Help/Content/ArubaFrameStyles/CertRevocation/About\_OCSP\_and\_CRL.htm).

### O que é Transparência de Certificado

A Transparência de Certificado tem como objetivo remediar ameaças baseadas em certificados, tornando a emissão e a existência de certificados SSL abertas à análise dos proprietários de domínios, CAs e usuários de domínios. Especificamente, a Transparência de Certificado tem três objetivos principais:

* Tornar impossível (ou pelo menos muito difícil) para uma CA **emitir um certificado SSL para um domínio sem que o proprietário** desse domínio **possa visualizar** o certificado.
* Fornecer um sistema de auditoria e monitoramento aberto que permita a qualquer proprietário de domínio ou CA determinar se certificados foram emitidos erroneamente ou maliciosamente.
* **Proteger os usuários** (o máximo possível) de serem enganados por certificados emitidos erroneamente ou maliciosamente.

#### **Logs de Certificado**

Logs de certificado são serviços de rede simples que mantêm registros de certificados **garantidos criptograficamente, auditáveis publicamente e somente para adição**. **Qualquer pessoa pode enviar certificados para um log**, embora as autoridades de certificação provavelmente sejam as principais remetentes. Da mesma forma, qualquer pessoa pode consultar um log para obter uma prova criptográfica, que pode ser usada para verificar se o log está se comportando corretamente ou verificar se um determinado certificado foi registrado. O número de servidores de log não precisa ser grande (digamos, muito menos de mil em todo o mundo), e cada um pode ser operado independentemente por uma CA, um ISP ou qualquer outra parte interessada.

#### Consulta

Você pode consultar os logs de Transparência de Certificado de qualquer domínio em [https://crt.sh/](https://crt.sh).

## Formatos

Existem diferentes formatos que podem ser usados para armazenar um certificado.

#### **Formato PEM**

* É o formato mais comum usado para certificados
* A maioria dos servidores (por exemplo, Apache) espera que os certificados e a chave privada estejam em arquivos separados\
\- Geralmente, eles são arquivos ASCII codificados em Base64\
\- As extensões usadas para certificados PEM são .cer, .crt, .pem, .key\
\- O Apache e servidores similares usam certificados no formato PEM

#### **Formato DER**

* O formato DER é a forma binária do certificado
* Todos os tipos de certificados e chaves privadas podem ser codificados no formato DER
* Certificados formatados em DER não contêm as declarações "BEGIN CERTIFICATE/END CERTIFICATE"
* Certificados formatados em DER geralmente usam as extensões ‘.cer’ e '.der'
* DER é tipicamente usado em plataformas Java

#### **Formato P7B/PKCS#7**

* O formato PKCS#7 ou P7B é armazenado no formato ASCII codificado em Base64 e tem uma extensão de arquivo .p7b ou .p7c
* Um arquivo P7B contém apenas certificados e certificados de cadeia (ACs intermediárias), não a chave privada
* As plataformas mais comuns que suportam arquivos P7B são o Microsoft Windows e o Java Tomcat

#### **Formato PFX/P12/PKCS#12**

* O formato PKCS#12 ou PFX/P12 é um formato binário para armazenar o certificado do servidor, certificados intermediários e a chave privada em um único arquivo criptografável
* Esses arquivos geralmente têm extensões como .pfx e .p12
* Eles são tipicamente usados em máquinas Windows para importar e exportar certificados e chaves privadas

### Conversões de formatos

**Converter x509 para PEM**
```
openssl x509 -in certificatename.cer -outform PEM -out certificatename.pem
```
To convert a PEM (Privacy Enhanced Mail) certificate file to DER (Distinguished Encoding Rules) format, you can use the OpenSSL command-line tool. The following command can be used for the conversion:

```bash
openssl x509 -in certificate.pem -outform der -out certificate.der
```

Replace `certificate.pem` with the path to your PEM certificate file, and `certificate.der` with the desired output file name for the DER format.

This command will read the PEM certificate file and convert it to DER format, saving the output to the specified file.

It's important to note that PEM and DER are two different formats for representing certificates. PEM is a base64-encoded format that includes header and footer lines, while DER is a binary format. Converting a certificate from PEM to DER can be useful in certain scenarios, such as when working with systems that require DER-encoded certificates.
```
openssl x509 -outform der -in certificatename.pem -out certificatename.der
```
**Converter DER para PEM**

Para converter um certificado no formato DER para o formato PEM, você pode usar a ferramenta OpenSSL. O formato PEM é um formato de arquivo base64 codificado que é amplamente suportado por várias aplicações.

Aqui está o comando para converter um certificado DER para PEM usando o OpenSSL:

```bash
openssl x509 -inform der -in certificado.der -out certificado.pem
```

Certifique-se de substituir "certificado.der" pelo nome do arquivo DER que você deseja converter e "certificado.pem" pelo nome do arquivo PEM de saída desejado.

Depois de executar o comando, o certificado DER será convertido para o formato PEM e salvo no arquivo especificado. Agora você pode usar o certificado PEM em suas aplicações que suportam esse formato.
```
openssl x509 -inform der -in certificatename.der -out certificatename.pem
```
**Converter PEM para P7B**

**Nota:** O formato PKCS#7 ou P7B é armazenado em formato ASCII Base64 e tem uma extensão de arquivo .p7b ou .p7c. Um arquivo P7B contém apenas certificados e certificados de cadeia (CAs intermediários), não a chave privada. As plataformas mais comuns que suportam arquivos P7B são o Microsoft Windows e o Java Tomcat.
```
openssl crl2pkcs7 -nocrl -certfile certificatename.pem -out certificatename.p7b -certfile CACert.cer
```
To convert a PKCS7 file to PEM format, you can use the OpenSSL command-line tool. The PKCS7 file contains certificates and/or CRLs (Certificate Revocation Lists) in a binary format, while the PEM format is a base64-encoded ASCII representation of the same data.

Here's the command to convert a PKCS7 file to PEM:

```plaintext
openssl pkcs7 -inform der -in input.p7b -out output.pem -print_certs
```

Replace `input.p7b` with the path to your PKCS7 file, and `output.pem` with the desired name for the PEM file.

This command uses the `pkcs7` command of OpenSSL, with the following options:

- `-inform der`: Specifies that the input file is in DER format (binary).
- `-in input.p7b`: Specifies the input PKCS7 file.
- `-out output.pem`: Specifies the output PEM file.
- `-print_certs`: Prints the certificates in the PKCS7 file.

After running the command, you will have a PEM file containing the certificates extracted from the PKCS7 file.
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

Às vezes, é necessário converter um arquivo no formato PEM (Privacy-Enhanced Mail) para o formato PKCS8 (Public-Key Cryptography Standards #8). O formato PKCS8 é amplamente utilizado para armazenar chaves privadas criptografadas.

Para converter um arquivo PEM para PKCS8, você pode usar a ferramenta OpenSSL. Siga as etapas abaixo:

1. Abra o terminal ou prompt de comando.
2. Execute o seguinte comando para converter o arquivo PEM para PKCS8:

   ```
   openssl pkcs8 -topk8 -inform PEM -outform DER -in chave_privada.pem -out chave_privada.pkcs8 -nocrypt
   ```

   Certifique-se de substituir "chave_privada.pem" pelo caminho e nome do seu arquivo PEM.

3. Após executar o comando, o arquivo PEM será convertido para o formato PKCS8 e salvo como "chave_privada.pkcs8".

Agora você tem um arquivo no formato PKCS8 que pode ser usado em várias aplicações que suportam esse formato.
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

Para converter um arquivo de certificado CER e uma chave privada em um arquivo PFX, você pode usar a ferramenta OpenSSL. Siga as etapas abaixo:

1. Certifique-se de ter o OpenSSL instalado em seu sistema.
2. Abra o terminal ou prompt de comando e navegue até o diretório onde estão localizados o arquivo CER e a chave privada.
3. Execute o seguinte comando para converter o arquivo CER e a chave privada em um arquivo PFX:

```
openssl pkcs12 -export -out certificado.pfx -inkey chave_privada.key -in certificado.cer
```

Certifique-se de substituir "chave_privada.key" pelo nome do arquivo da chave privada e "certificado.cer" pelo nome do arquivo CER.

4. Durante o processo de conversão, você será solicitado a definir uma senha para proteger o arquivo PFX. Digite uma senha segura e lembre-se dela.

Após a conclusão do processo, você terá um arquivo PFX que contém o certificado e a chave privada. Esse arquivo pode ser usado em várias plataformas e aplicativos que suportam o formato PFX para autenticação e criptografia.
```
openssl pkcs12 -export -in certificatename.cer -inkey privateKey.key -out certificatename.pfx -certfile  cacert.cer
```
<figure><img src="/.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

\
Use [**Trickest**](https://trickest.io/) para construir e automatizar facilmente fluxos de trabalho com as ferramentas comunitárias mais avançadas do mundo.\
Acesse hoje mesmo:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* Você trabalha em uma **empresa de cibersegurança**? Gostaria de ver sua **empresa anunciada no HackTricks**? Ou gostaria de ter acesso à **última versão do PEASS ou baixar o HackTricks em PDF**? Confira os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* Adquira o [**swag oficial do PEASS & HackTricks**](https://peass.creator-spring.com)
* **Junte-se ao** [**💬**](https://emojipedia.org/speech-balloon/) [**grupo Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo telegram**](https://t.me/peass) ou **siga-me** no **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Compartilhe seus truques de hacking enviando PRs para o** [**repositório hacktricks**](https://github.com/carlospolop/hacktricks) **e** [**repositório hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
