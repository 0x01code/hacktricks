# Yazılabilir Sys Yolu +Dll Hijacking Privesc

<details>

<summary><strong>Sıfırdan kahraman olmaya kadar AWS hackleme öğrenin</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Kırmızı Takım Uzmanı)</strong></a><strong> ile!</strong></summary>

HackTricks'ı desteklemenin diğer yolları:

* **Şirketinizi HackTricks'te reklamını görmek istiyorsanız** veya **HackTricks'i PDF olarak indirmek istiyorsanız** [**ABONELİK PLANLARI**]'na(https://github.com/sponsors/carlospolop) göz atın!
* [**Resmi PEASS & HackTricks ürünlerini**](https://peass.creator-spring.com) edinin
* [**PEASS Ailesi'ni**](https://opensea.io/collection/the-peass-family) keşfedin, özel [**NFT'lerimiz**](https://opensea.io/collection/the-peass-family) koleksiyonumuz
* **Katılın** 💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) veya bizi **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)** takip edin.**
* **Hacking hilelerinizi paylaşarak PR göndererek** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github depolarına katkıda bulunun.

</details>

## Giriş

Eğer **Bir Sistem Yolu klasörüne yazabileceğinizi** fark ettiyseniz (unutmayın ki bu bir Kullanıcı Yolu klasörüne yazabiliyorsanız çalışmaz) bu durumda **sistemde ayrıcalıkları yükseltebilirsiniz**.

Bunu yapmak için, **Daha fazla ayrıcalığa sahip bir hizmet veya işlem tarafından yüklenen bir kütüphaneyi ele geçireceğiniz bir Dll Hijacking** kullanabilirsiniz ve çünkü bu hizmet, muhtemelen tüm sistemde mevcut olmayan bir Dll'yi yüklemeye çalışacak, bu Dll'yi yazabileceğiniz Sistem Yolundan yüklemeye çalışacak.

**Dll Hijacking nedir** hakkında daha fazla bilgi için:

{% content-ref url="./" %}
[.](./)
{% endcontent-ref %}

## Dll Hijacking ile Privesc

### Eksik bir Dll bulma

İlk yapmanız gereken şey, **kendi ayrıcalıklarınızdan daha fazla ayrıcalıklara sahip bir işlemi tanımlamak** ve **yazabileceğiniz Sistem Yolundan bir Dll yüklemeye çalışan bir işlemi belirlemektir**.

Bu durumlarda sorun, muhtemelen bu işlemlerin zaten çalışıyor olmasıdır. İhtiyacınız olan Dll'leri bulmak için, gerekli hizmetlerin eksik olan .dll'lerini bulmak için mümkün olan en kısa sürede procmon'u başlatmanız gerekmektedir (işlemler yüklenmeden önce). Yani, eksik .dll'leri bulmak için şunları yapın:

* `C:\privesc_hijacking` klasörünü **oluşturun** ve bu yolu **Sistem Yolu ortam değişkenine** ekleyin. Bunu **manuel olarak** veya **PS** ile yapabilirsiniz:
```powershell
# Set the folder path to create and check events for
$folderPath = "C:\privesc_hijacking"

# Create the folder if it does not exist
if (!(Test-Path $folderPath -PathType Container)) {
New-Item -ItemType Directory -Path $folderPath | Out-Null
}

# Set the folder path in the System environment variable PATH
$envPath = [Environment]::GetEnvironmentVariable("PATH", "Machine")
if ($envPath -notlike "*$folderPath*") {
$newPath = "$envPath;$folderPath"
[Environment]::SetEnvironmentVariable("PATH", $newPath, "Machine")
}
```
* **`procmon`**'u başlatın ve **`Options`** --> **`Enable boot logging`** yolunu izleyin ve ekrandaki **`OK`** düğmesine basın.
* Ardından **sistemi yeniden başlatın**. Bilgisayar yeniden başlatıldığında **`procmon`** olayları hemen kaydetmeye başlayacaktır.
* **Windows** başladığında **`procmon`**'u tekrar **çalıştırın**, çalıştığını ve olayları bir dosyada saklamak isteyip istemediğinizi soracaktır. Olayları bir dosyada **saklamayı kabul edin**.
* **Dosya** oluşturulduktan **sonra**, açık olan **`procmon`** penceresini kapatın ve olaylar dosyasını **açın**.
* Aşağıdaki **filtreleri ekleyin** ve yazılabilir System Path klasöründen yüklenmeye çalışılan tüm Dll'leri bulacaksınız:

<figure><img src="../../../.gitbook/assets/image (945).png" alt=""><figcaption></figcaption></figure>

### Eksik Dll'ler

Bu işlemi ücretsiz bir **sanal (vmware) Windows 11 makinesinde** çalıştırdığımda şu sonuçları elde ettim:

<figure><img src="../../../.gitbook/assets/image (607).png" alt=""><figcaption></figcaption></figure>

Bu durumda .exe dosyaları işe yaramaz, onları görmezden gelin, eksik DLL'ler şuradan geldi:

| Servis                         | Dll                | CMD satırı                                                           |
| ------------------------------- | ------------------ | -------------------------------------------------------------------- |
| Görev Zamanlayıcısı (Schedule) | WptsExtensions.dll | `C:\Windows\system32\svchost.exe -k netsvcs -p -s Schedule`          |
| Tanımlayıcı Politika Servisi (DPS) | Unknown.DLL        | `C:\Windows\System32\svchost.exe -k LocalServiceNoNetwork -p -s DPS` |
| ???                             | SharedRes.dll      | `C:\Windows\system32\svchost.exe -k UnistackSvcGroup`                |

Bunu bulduktan sonra, aynı zamanda [**WptsExtensions.dll'yi kötüye kullanmak için nasıl kullanılacağını açıklayan**](https://juggernaut-sec.com/dll-hijacking/#Windows\_10\_Phantom\_DLL\_Hijacking\_-\_WptsExtensionsdll) ilginç bir blog yazısı buldum. Şimdi **yapacağımız şey** budur.

### Sömürü

Yani, **yetkileri yükseltmek** için **WptsExtensions.dll** kütüphanesini ele geçireceğiz. **Yolu** ve **adı** olan kötü niyetli dll'yi oluşturmamız yeterli olacaktır.

[**Bu örneklerden herhangi birini kullanmayı deneyebilirsiniz**](./#creating-and-compiling-dlls). Rev shell alabilir, bir kullanıcı ekleyebilir, bir işaretçi çalıştırabilirsiniz...

{% hint style="warning" %}
**Tüm hizmetlerin** **`NT AUTHORITY\SYSTEM`** ile çalıştırılmadığını unutmayın, bazıları aynı zamanda **`NT AUTHORITY\LOCAL SERVICE`** ile çalıştırılır ki bu daha az ayrıcalığa sahiptir ve bu izinleri kötüye kullanarak yeni bir kullanıcı oluşturamazsınız.\
Ancak, bu kullanıcı **`seImpersonate`** ayrıcalığına sahiptir, bu nedenle [**aşırı ayrıcalıklar için patates paketini kullanabilirsiniz**](../roguepotato-and-printspoofer.md). Bu durumda bir rev shell oluşturmak, bir kullanıcı oluşturmaya çalışmaktan daha iyi bir seçenektir.
{% endhint %}

Bu yazıyı yazdığım sırada **Görev Zamanlayıcısı** hizmeti **Nt AUTHORITY\SYSTEM** ile çalıştırılıyordu.

Kötü niyetli Dll'yi oluşturduktan sonra (_benim durumumda x64 rev shell kullandım ve bir kabuk aldım ancak savunucu onu öldürdü çünkü msfvenom'dan geliyordu_), onu yazılabilir System Path klasörüne **WptsExtensions.dll** adıyla kaydedin ve bilgisayarı yeniden başlatın (veya hizmeti yeniden başlatın veya etkilenen hizmeti/programı yeniden çalıştırmak için gereken herhangi bir işlemi yapın).

Hizmet yeniden başlatıldığında, **dll yüklenmeli ve çalıştırılmalıdır** (kütüphanenin beklenildiği gibi yüklendiğini kontrol etmek için **procmon** hilesini tekrar kullanabilirsiniz).
