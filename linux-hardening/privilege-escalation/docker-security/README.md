# Docker Güvenliği

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert)</strong> ile sıfırdan kahramana AWS hacklemeyi öğrenin!</summary>

HackTricks'i desteklemenin diğer yolları:

* **Şirketinizi HackTricks'te reklamını görmek istiyorsanız** veya **HackTricks'i PDF olarak indirmek istiyorsanız** [**ABONELİK PLANLARI**](https://github.com/sponsors/carlospolop)'na göz atın!
* [**Resmi PEASS & HackTricks ürünlerini**](https://peass.creator-spring.com) edinin
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)'yi keşfedin, özel [**NFT'lerimiz**](https://opensea.io/collection/the-peass-family) koleksiyonumuzu görün
* 💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) **katılın** veya **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)'u takip edin.
* **Hacking püf noktalarınızı paylaşarak PR göndererek** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github depolarına katkıda bulunun.

</details>

<figure><img src="../../../.gitbook/assets/image (45).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) kullanarak dünyanın **en gelişmiş topluluk araçları** tarafından desteklenen ve **iş akışlarını otomatikleştiren** bir platform oluşturun.\
Bugün Erişim Alın:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## **Temel Docker Motoru Güvenliği**

**Docker motoru**, konteynerleri izole etmek için Linux çekirdeğinin **Namespaces** ve **Cgroups**'ını kullanır, temel bir güvenlik katmanı sunar. **Yeteneklerin düşürülmesi**, **Seccomp** ve **SELinux/AppArmor** ile ek koruma sağlanır, konteyner izolasyonu artırılır. Bir **auth eklentisi** kullanıcı işlemlerini daha da kısıtlayabilir.

![Docker Güvenliği](https://sreeninet.files.wordpress.com/2016/03/dockersec1.png)

### Docker Motoruna Güvenli Erişim

Docker motoruna yerel olarak Unix soketi aracılığıyla veya uzaktan HTTP kullanılarak erişilebilir. Uzaktan erişim için gizlilik, bütünlük ve kimlik doğrulamasını sağlamak için HTTPS ve **TLS** kullanmak önemlidir.

Ubuntu sistemlerinde Docker varsayılan olarak `unix:///var/run/docker.sock` üzerinde Unix soketinde dinler. Docker'ın başlangıç seçenekleri `/etc/default/docker` dosyasında tanımlanmıştır. Docker API ve istemciye uzaktan erişimi etkinleştirmek için Docker daemon'ını HTTP soketi üzerinden açmak için aşağıdaki ayarları ekleyin:
```bash
DOCKER_OPTS="-D -H unix:///var/run/docker.sock -H tcp://192.168.56.101:2376"
sudo service docker restart
```
Ancak Docker daemon'ını HTTP üzerinden açmak güvenlik endişeleri nedeniyle önerilmez. Bağlantıların güvenliğini sağlamak için HTTPS kullanılması tavsiye edilir. Bağlantıyı güvence altına almanın iki temel yaklaşımı vardır:

1. İstemci sunucunun kimliğini doğrular.
2. Hem istemci hem de sunucu birbirlerinin kimliğini karşılıklı olarak doğrular.

Sertifikalar, bir sunucunun kimliğini doğrulamak için kullanılır. Her iki yöntemin detaylı örnekleri için [**bu kılavuza**](https://sreeninet.wordpress.com/2016/03/06/docker-security-part-3engine-access/) başvurun.

### Konteyner Görüntülerinin Güvenliği

Konteyner görüntüleri ya özel ya da genel depolama alanlarında saklanabilir. Docker, konteyner görüntüleri için çeşitli depolama seçenekleri sunar:

* [**Docker Hub**](https://hub.docker.com): Docker'dan genel bir kayıt servisi.
* [**Docker Registry**](https://github.com/docker/distribution): Kullanıcıların kendi kayıtlarını barındırmasına izin veren açık kaynaklı bir proje.
* [**Docker Trusted Registry**](https://www.docker.com/docker-trusted-registry): Rol tabanlı kullanıcı kimlik doğrulaması ve LDAP dizin hizmetleriyle entegrasyon sunan Docker'ın ticari kayıt sunumu.

### Görüntü Tarama

Konteynerler, temel görüntü veya temel görüntü üzerine kurulan yazılım nedeniyle **güvenlik açıklarına** sahip olabilir. Docker, konteynerlerin güvenlik taramasını yaparak güvenlik açıklarını listeler bir proje olan **Nautilus** üzerinde çalışmaktadır. Nautilus, her Konteyner görüntü katmanını güvenlik açıklarını belirlemek için güvenlik açıkları deposuyla karşılaştırarak çalışır.

Daha fazla [**bilgi için burayı okuyun**](https://docs.docker.com/engine/scan/).

* **`docker scan`**

**`docker scan`** komutu, görüntü adını veya kimliğini kullanarak mevcut Docker görüntülerini taramanıza olanak tanır. Örneğin, hello-world görüntüsünü taramak için aşağıdaki komutu çalıştırın:
```bash
docker scan hello-world

Testing hello-world...

Organization:      docker-desktop-test
Package manager:   linux
Project name:      docker-image|hello-world
Docker image:      hello-world
Licenses:          enabled

✓ Tested 0 dependencies for known issues, no vulnerable paths found.

Note that we do not currently have vulnerability data for your image.
```
* [**`trivy`**](https://github.com/aquasecurity/trivy)
```bash
trivy -q -f json <container_name>:<tag>
```
* [**`snyk`**](https://docs.snyk.io/snyk-cli/getting-started-with-the-cli)
```bash
snyk container test <image> --json-file-output=<output file> --severity-threshold=high
```
* [**`clair-scanner`**](https://github.com/arminc/clair-scanner)
```bash
clair-scanner -w example-alpine.yaml --ip YOUR_LOCAL_IP alpine:3.5
```
### Docker İmaj İmzalama

Docker imaj imzalama, konteynerlerde kullanılan imajların güvenliğini ve bütünlüğünü sağlar. İşte özetlenmiş bir açıklama:

- **Docker İçerik Güveni**, imaj imzalama işlemlerini yönetmek için The Update Framework (TUF) üzerine kurulu Notary projesini kullanır. Daha fazla bilgi için [Notary](https://github.com/docker/notary) ve [TUF](https://theupdateframework.github.io) sayfalarına bakabilirsiniz.
- Docker içerik güvenini etkinleştirmek için `export DOCKER_CONTENT_TRUST=1` ayarını yapın. Bu özellik, Docker sürümü 1.10 ve sonrasında varsayılan olarak kapalıdır.
- Bu özellik etkinleştirildiğinde, yalnızca imzalı imajlar indirilebilir. İlk imaj yükleme işlemi, kök ve etiketleme anahtarları için parola belirlemeyi gerektirir ve Docker ayrıca artırılmış güvenlik için Yubikey'i de destekler. Daha fazla ayrıntıya [buradan](https://blog.docker.com/2015/11/docker-content-trust-yubikey/) ulaşabilirsiniz.
- İçerik güveni etkinleştirilmiş bir imzasız imajı çekmeye çalışmak, "No trust data for latest" hatası ile sonuçlanır.
- İlk imaj yükleme işleminden sonra, Docker, imajı imzalamak için depo anahtarının parolasını ister.

Özel anahtarlarınızı yedeklemek için aşağıdaki komutu kullanın:
```bash
tar -zcvf private_keys_backup.tar.gz ~/.docker/trust/private
```
Docker ana bilgisayarlar arasında geçiş yaparken işlemleri sürdürebilmek için kök ve depo anahtarlarını taşımak gereklidir.

***

<figure><img src="../../../.gitbook/assets/image (45).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks) kullanarak dünyanın en gelişmiş topluluk araçları tarafından desteklenen **otomatikleştirilmiş iş akışları** oluşturun ve yönetin.\
Bugün Erişim Alın:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## Konteyner Güvenlik Özellikleri

<details>

<summary>Konteyner Güvenlik Özelliklerinin Özeti</summary>

**Ana İşlem İzolasyon Özellikleri**

Konteynerleştirilmiş ortamlarda, projeleri ve işlemleri izole etmek güvenlik ve kaynak yönetimi için hayati önem taşır. İşte temel kavramların basitleştirilmiş bir açıklaması:

**Ad Alanları (Namespaces)**

* **Amaç**: İşlemler, ağ ve dosya sistemleri gibi kaynakların izolasyonunu sağlamak. Özellikle Docker'da, ad alanları bir konteynerin işlemlerini ana bilgisayardan ve diğer konteynerlerden ayırır.
* **`unshare` Kullanımı**: Yeni ad alanları oluşturmak için `unshare` komutu (veya altta yatan sistem çağrısı) kullanılır, ek bir izolasyon katmanı sağlar. Ancak, Kubernetes bunu doğal olarak engellemezken, Docker engeller.
* **Sınırlama**: Yeni ad alanları oluşturmak, bir işlemin ana bilgisayarın varsayılan ad alanlarına geri dönmesine izin vermez. Ana bilgisayarın ad alanlarına sızabilmek için genellikle ana bilgisayarın `/proc` dizinine erişim sağlamak ve giriş için `nsenter` kullanmak gerekir.

**Kontrol Grupları (CGroups)**

* **Fonksiyon**: İşlemler arasında kaynak tahsisi için başlıca kullanılır.
* **Güvenlik Yönü**: CGroups kendileri izolasyon güvenliği sunmaz, ancak yanlış yapılandırılmışsa `release_agent` özelliği, yetkisiz erişim için potansiyel olarak kötüye kullanılabilir.

**Yetenek Düşürme (Capability Drop)**

* **Önemi**: İşlem izolasyonu için kritik bir güvenlik özelliğidir.
* **İşlevsellik**: Belirli yetenekleri düşürerek kök işlemin gerçekleştirebileceği eylemleri kısıtlar. Bir işlem kök ayrıcalıklarıyla çalışsa bile, gerekli yeteneklere sahip olmaması nedeniyle ayrıcalıklı eylemleri gerçekleştiremez, çünkü sistem çağrıları yetersiz izinler nedeniyle başarısız olur.

Bu, işlem diğerlerini düşürdükten sonra **kalan yeteneklerdir**:

{% code overflow="wrap" %}
```
Current: cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap=ep
```
{% endcode %}

**Seccomp**

Docker'da varsayılan olarak etkindir. **İşlemin çağırabileceği sistem çağrılarını daha da sınırlamaya yardımcı olur**.\
**Varsayılan Docker Seccomp profili**, [https://github.com/moby/moby/blob/master/profiles/seccomp/default.json](https://github.com/moby/moby/blob/master/profiles/seccomp/default.json) adresinde bulunabilir.

**AppArmor**

Docker'ın etkinleştirebileceğiniz bir şablonu vardır: [https://github.com/moby/moby/tree/master/profiles/apparmor](https://github.com/moby/moby/tree/master/profiles/apparmor)

Bu, yetenekleri, sistem çağrılarını, dosya ve klasörlere erişimi azaltmanıza olanak tanır...

</details>

### Ad Alanları

**Ad alanları**, Linux çekirdeğinin bir özelliğidir ve **çekirdek kaynaklarını bölümlendirir**, böylece bir **set işlem**, bir **set kaynak** görürken **başka** bir **set işlem** farklı bir **set kaynak** görür. Bu özellik, aynı ad alanına sahip kaynaklar ve işlemler kümesi oluşturarak çalışır, ancak bu ad alanları farklı kaynaklara işaret eder. Kaynaklar birden fazla alanda bulunabilir.

Docker, Konteyner izolasyonunu sağlamak için aşağıdaki Linux çekirdek Ad Alanlarını kullanır:

* pid ad alanı
* mount ad alanı
* network ad alanı
* ipc ad alanı
* UTS ad alanı

**Ad alanları hakkında daha fazla bilgi** için aşağıdaki sayfaya bakın:

{% content-ref url="namespaces/" %}
[namespaces](namespaces/)
{% endcontent-ref %}

### cgroups

Linux çekirdek özelliği **cgroups**, **cpu, bellek, io, ağ bant genişliği gibi kaynakları sınırlama** yeteneği sağlar. Docker, belirli bir Konteyner için kaynak kontrolü sağlayan cgroup özelliğini kullanarak Konteynerler oluşturmanıza izin verir.\
Aşağıda, kullanıcı alanı belleği 500m, çekirdek belleği 50m, cpu payını 512, blkioweight'ı 400 ile sınırlı bir Konteyner oluşturulmuştur. CPU payı, Konteyner'ın CPU kullanımını kontrol eden bir orandır. Varsayılan değeri 1024'tür ve 0 ile 1024 arasında bir aralığa sahiptir. Üç Konteynerin aynı CPU payına (1024) sahip olduğunu varsayarsak, CPU kaynak çatışması durumunda her Konteyner, CPU'nun %33'üne kadar alabilir. blkio-weight, Konteyner'ın IO'sunu kontrol eden bir orandır. Varsayılan değeri 500'dür ve 10 ile 1000 arasında bir aralığa sahiptir.
```
docker run -it -m 500M --kernel-memory 50M --cpu-shares 512 --blkio-weight 400 --name ubuntu1 ubuntu bash
```
Bir konteynerin cgroup'una erişmek için şunu yapabilirsiniz:
```bash
docker run -dt --rm denial sleep 1234 #Run a large sleep inside a Debian container
ps -ef | grep 1234 #Get info about the sleep process
ls -l /proc/<PID>/ns #Get the Group and the namespaces (some may be uniq to the hosts and some may be shred with it)
```
Daha fazla bilgi için kontrol edin:

{% content-ref url="cgroups.md" %}
[cgroups.md](cgroups.md)
{% endcontent-ref %}

### Yetenekler

Yetenekler, kök kullanıcı için izin verilebilecek yetenekler için daha **inceden kontrol sağlar**. Docker, **Kullanıcı türünden bağımsız olarak bir Konteyner içinde yapılabilen işlemleri sınırlamak** için Linux çekirdek yetenek özelliğini kullanır.

Bir docker konteyneri çalıştırıldığında, **işlem, izolasyondan kaçmak için kullanabileceği hassas yetenekleri bırakır**. Bu, işlemin hassas eylemleri gerçekleştiremeyeceğinden ve kaçamayacağından emin olmaya çalışır:

{% content-ref url="../linux-capabilities.md" %}
[linux-capabilities.md](../linux-capabilities.md)
{% endcontent-ref %}

### Docker'da Seccomp

Bu, Docker'ın konteyner içinde kullanılabilecek **sistem çağrılarını sınırlamasına** olanak tanıyan bir güvenlik özelliğidir:

{% content-ref url="seccomp.md" %}
[seccomp.md](seccomp.md)
{% endcontent-ref %}

### Docker'da AppArmor

**AppArmor**, **konteynerleri** bir **sınırlı** **küme kaynağa** ve **program başına profillere** kapatmak için bir çekirdek geliştirmesidir:

{% content-ref url="apparmor.md" %}
[apparmor.md](apparmor.md)
{% endcontent-ref %}

### Docker'da SELinux

* **Etiketleme Sistemi**: SELinux, her işlem ve dosya sistemi nesnesine benzersiz bir etiket atar.
* **Politika Uygulaması**: Sistem içinde bir işlem etiketinin diğer etiketler üzerinde hangi eylemleri gerçekleştirebileceğini tanımlayan güvenlik politikalarını uygular.
* **Konteyner İşlem Etiketleri**: Konteyner motorları konteyner işlemlerini başlattığında genellikle sınırlı bir SELinux etiketi olan `container_t` atanır.
* **Konteyner İçindeki Dosya Etiketleme**: Konteyner içindeki dosyalar genellikle `container_file_t` olarak etiketlenir.
* **Politika Kuralları**: SELinux politikası, `container_t` etiketine sahip işlemlerin yalnızca `container_file_t` olarak etiketlenmiş dosyalarla etkileşime geçebileceğini sağlar.

Bu mekanizma, bir konteyner içindeki bir işlem bile tehlikeye atılsa, yalnızca karşılık gelen etiketlere sahip nesnelerle etkileşime geçebileceğinden, bu tür tehlikelerden kaynaklanabilecek potansiyel hasarı önemli ölçüde sınırlar.

{% content-ref url="../selinux.md" %}
[selinux.md](../selinux.md)
{% endcontent-ref %}

### AuthZ & AuthN

Docker'da bir yetkilendirme eklentisi, Docker daemonuna yapılan istekleri izin verip engellemenin güvenlik açısından kritik bir rol oynar. Bu karar, iki temel bağlamı inceleyerek verilir:

* **Kimlik Doğrulama Bağlamı**: Bu, kullanıcı hakkında kapsamlı bilgileri içerir, kim oldukları ve nasıl kimlik doğruladıkları gibi.
* **Komut Bağlamı**: Bu, yapılan isteğe ilişkin tüm ilgili verileri içerir.

Bu bağlamlar, yalnızca kimlik doğrulaması yapılmış kullanıcılardan gelen meşru isteklerin işlenmesini sağlayarak Docker işlemlerinin güvenliğini artırır.

{% content-ref url="authz-and-authn-docker-access-authorization-plugin.md" %}
[authz-and-authn-docker-access-authorization-plugin.md](authz-and-authn-docker-access-authorization-plugin.md)
{% endcontent-ref %}

## Bir konteynerden DoS

Bir konteynerin kullanabileceği kaynakları doğru bir şekilde sınırlamazsanız, tehlikeye düşmüş bir konteyner, çalıştığı ana bilgisayarı DoS edebilir.

* CPU DoS
```bash
# stress-ng
sudo apt-get install -y stress-ng && stress-ng --vm 1 --vm-bytes 1G --verify -t 5m

# While loop
docker run -d --name malicious-container -c 512 busybox sh -c 'while true; do :; done'
```
* Bant Genişliği DDoS
```bash
nc -lvp 4444 >/dev/null & while true; do cat /dev/urandom | nc <target IP> 4444; done
```
## İlginç Docker Bayrakları

### --privileged bayrağı

Aşağıdaki sayfada **`--privileged` bayrağının ne anlama geldiğini** öğrenebilirsiniz:

{% content-ref url="docker-privileged.md" %}
[docker-privileged.md](docker-privileged.md)
{% endcontent-ref %}

### --security-opt

#### no-new-privileges

Eğer bir saldırganın düşük ayrıcalıklı bir kullanıcı olarak erişim sağladığı bir konteyner çalıştırıyorsanız ve **hatalı yapılandırılmış bir suid ikili dosyanız** varsa, saldırgan bunu kötüye kullanabilir ve konteyner içinde **ayrıcalıkları yükseltebilir**. Bu da onun kaçmasına izin verebilir.

Konteyneri **`no-new-privileges`** seçeneği etkinleştirilmiş olarak çalıştırmak, bu tür ayrıcalık yükseltmelerini **engelleyecektir**.
```
docker run -it --security-opt=no-new-privileges:true nonewpriv
```
#### Diğer
```bash
#You can manually add/drop capabilities with
--cap-add
--cap-drop

# You can manually disable seccomp in docker with
--security-opt seccomp=unconfined

# You can manually disable seccomp in docker with
--security-opt apparmor=unconfined

# You can manually disable selinux in docker with
--security-opt label:disable
```
Daha fazla **`--security-opt`** seçeneği için kontrol edin: [https://docs.docker.com/engine/reference/run/#security-configuration](https://docs.docker.com/engine/reference/run/#security-configuration)

## Diğer Güvenlik Düşünceleri

### Şifreleri Yönetme: En İyi Uygulamalar

Docker görüntülerine doğrudan şifre gömmek veya çevre değişkenleri kullanmak önemli değildir, çünkü bu yöntemler, `docker inspect` veya `exec` gibi komutlar aracılığıyla konteynıra erişimi olan herkese hassas bilgilerinizi açığa çıkarır.

**Docker birimleri** hassas bilgilere erişim için önerilen daha güvenli bir alternatiftir. Bunlar, riskleri azaltmak için geçici bir dosya sistemi olarak hafızada kullanılabilir ve `docker inspect` ve günlüğe kaydetme ile ilişkili riskleri hafifletir. Ancak, kök kullanıcılar ve konteynıra `exec` erişimi olanlar hala şifrelere erişebilir.

**Docker secrets** hassas bilgileri işlemede daha güvenli bir yöntem sunar. Görüntü oluşturma aşamasında şifreler gerektiren durumlar için, **BuildKit** ek özellikler sunarak görüntü oluşturma hızını artırır ve destek sağlayan etkili bir çözüm sunar.

BuildKit'i kullanmak için üç şekilde etkinleştirilebilir:

1. Bir çevre değişkeni aracılığıyla: `export DOCKER_BUILDKIT=1`
2. Komutlara önek ekleyerek: `DOCKER_BUILDKIT=1 docker build .`
3. Docker yapılandırmasında varsayılan olarak etkinleştirilerek: `{ "features": { "buildkit": true } }`, ardından bir Docker yeniden başlatma işlemi.

BuildKit, `--secret` seçeneği ile yapı zamanı şifrelerin kullanılmasına olanak tanır, bu sayede bu şifrelerin görüntü oluşturma önbelleğine veya nihai görüntüye dahil edilmediğinden emin olunur, şu şekilde bir komut kullanılarak:
```bash
docker build --secret my_key=my_value ,src=path/to/my_secret_file .
```
Çalışan bir konteyner için gerekli olan sırlar için **Docker Compose ve Kubernetes** sağlam çözümler sunar. Docker Compose, gizli dosyaları belirtmek için hizmet tanımında bir `secrets` anahtarı kullanır. Aşağıda bir `docker-compose.yml` örneğinde gösterildiği gibi:
```yaml
version: "3.7"
services:
my_service:
image: centos:7
entrypoint: "cat /run/secrets/my_secret"
secrets:
- my_secret
secrets:
my_secret:
file: ./my_secret_file.txt
```
Bu yapılandırma, Docker Compose ile hizmetleri başlatırken secrets kullanımına izin verir.

Kubernetes ortamlarında, secrets doğal olarak desteklenir ve [Helm-Secrets](https://github.com/futuresimple/helm-secrets) gibi araçlarla daha fazla yönetilebilir. Kubernetes'in Rol Tabanlı Erişim Kontrolleri (RBAC), Docker Enterprise ile benzer şekilde secret yönetimi güvenliğini artırır.

### gVisor

**gVisor**, Go dilinde yazılmış bir uygulama çekirdeğidir ve Linux sistemi yüzeyinin önemli bir kısmını uygular. Uygulama ile ana çekirdek arasında bir **izolasyon sınırı sağlayan** `runsc` adında bir [Open Container Initiative (OCI)](https://www.opencontainers.org) çalışma zamanını içerir. `runsc` çalışma zamanı, Docker ve Kubernetes ile entegre olup, kum havuzlu konteynerlerin çalıştırılmasını kolaylaştırır.

{% embed url="https://github.com/google/gvisor" %}

### Kata Containers

**Kata Containers**, konteynerlere benzer şekilde hissedip performans gösteren hafif sanal makinelerle güvenli bir konteyner çalışma zamanı oluşturmak için çalışan açık kaynak topluluğudur, ancak ikinci bir savunma katmanı olarak donanım sanallaştırma teknolojisini kullanarak **daha güçlü iş yükü izolasyonu sağlar**.

{% embed url="https://katacontainers.io/" %}

### Özet İpuçları

* **`--privileged` bayrağını kullanmayın veya konteyner içinde bir** [**Docker soketi bağlamayın**](https://raesene.github.io/blog/2016/03/06/The-Dangers-Of-Docker.sock/)**.** Docker soketi, konteynerlerin başlatılmasına izin verir, bu nedenle örneğin `--privileged` bayrağı ile başka bir konteyner çalıştırarak ana bilgisayarın tam kontrolünü ele geçirmek kolaydır.
* **Konteyner içinde kök olarak çalıştırmayın.** [**Farklı bir kullanıcı**](https://docs.docker.com/develop/develop-images/dockerfile\_best-practices/#user) **ve** [**kullanıcı ad alanları**](https://docs.docker.com/engine/security/userns-remap/)** kullanın.** Konteynerdeki kök, kullanıcı ad alanları ile yeniden eşlenmediği sürece ana bilgisayarla aynıdır. Yalnızca Linux ad alanları, yetenekler ve cgroups tarafından hafifçe kısıtlanmıştır.
* [**Tüm yetenekleri bırakın**](https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities) **(`--cap-drop=all`) ve yalnızca gerekli olanları etkinleştirin** (`--cap-add=...`). Birçok iş yükü hiçbir yetenek gerektirmez ve bunları eklemek, potansiyel bir saldırı kapsamını artırır.
* [**"no-new-privileges" güvenlik seçeneğini kullanın**](https://raesene.github.io/blog/2019/06/01/docker-capabilities-and-no-new-privs/) **, örneğin suid ikili dosyalar aracılığıyla daha fazla ayrıcalık kazanılmasını önlemek için.**
* [**Konteynere sağlanan kaynakları sınırlayın**](https://docs.docker.com/engine/reference/run/#runtime-constraints-on-resources)**.** Kaynak sınırları, makinenin hizmet reddi saldırılarından korunmasına yardımcı olabilir.
* **[Seccomp](https://docs.docker.com/engine/security/seccomp/)**, [**AppArmor**](https://docs.docker.com/engine/security/apparmor/) **(veya SELinux)** profillerini ayarlayın, konteyner için kullanılabilir eylemleri ve sistem çağrılarını minimuma indirmek için.
* **[Resmi docker görüntülerini](https://docs.docker.com/docker-hub/official\_images/) kullanın ve imzaları gerektirin** veya bunlara dayalı kendi görüntülerinizi oluşturun. Geriye dönük veya [arka kapılı](https://arstechnica.com/information-technology/2018/06/backdoored-images-downloaded-5-million-times-finally-removed-from-docker-hub/) görüntülerden miras almayın veya kullanmayın. Ayrıca kök anahtarları, parola güvenli bir yerde saklayın. Docker, UCP ile anahtarları yönetme planları yapmaktadır.
* **Güvenlik yamalarını uygulamak için düzenli olarak** **görüntülerinizi yeniden oluşturun.**
* **Secret'ları akıllıca yönetin** böylece saldırganın bunlara erişmesi zor olur.
* Docker daemon'ı **HTTPS ile açıklarsanız**, istemci ve sunucu kimlik doğrulaması yapın.
* Dockerfile'ınızda, **ADD yerine COPY'yi tercih edin**. ADD otomatik olarak sıkıştırılmış dosyaları çıkarır ve dosyaları URL'lerden kopyalayabilir. COPY bu yeteneklere sahip değildir. Mümkün olduğunca ADD kullanmaktan kaçının, böylece uzak URL'ler ve Zip dosyaları aracılığıyla yapılan saldırılara karşı savunmasız olmazsınız.
* **Her mikro hizmet için ayrı konteynerler kullanın.**
* **Konteyner görüntülerini daha küçük yapın.**
* **HackTricks'te şirketinizin reklamını görmek istiyorsanız** veya **HackTricks'i PDF olarak indirmek istiyorsanız** [**ABONELİK PLANLARI**](https://github.com/sponsors/carlospolop)'na göz atın!
* [**Resmi PEASS & HackTricks ürünlerini**](https://peass.creator-spring.com) edinin
* [**PEASS Ailesi'ni**](https://opensea.io/collection/the-peass-family) keşfedin, özel [**NFT'lerimiz**](https://opensea.io/collection/the-peass-family) koleksiyonumuz
* **Katılın** 💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) veya bizi **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.** takip edin
* **Hacking püf noktalarınızı paylaşın, PR'lar göndererek** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github depolarına.
