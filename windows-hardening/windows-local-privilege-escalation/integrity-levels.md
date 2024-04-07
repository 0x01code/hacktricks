# Poziomy integralności

<details>

<summary><strong>Nauka hakerskiego AWS od zera do bohatera z</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Inne sposoby wsparcia HackTricks:

* Jeśli chcesz zobaczyć swoją **firmę reklamowaną w HackTricks** lub **pobrać HackTricks w formacie PDF**, sprawdź [**PLANY SUBSKRYPCYJNE**](https://github.com/sponsors/carlospolop)!
* Zdobądź [**oficjalne gadżety PEASS & HackTricks**](https://peass.creator-spring.com)
* Odkryj [**Rodzinę PEASS**](https://opensea.io/collection/the-peass-family), naszą kolekcję ekskluzywnych [**NFT**](https://opensea.io/collection/the-peass-family)
* **Dołącz do** 💬 [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegramowej**](https://t.me/peass) lub **śledź** nas na **Twitterze** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podziel się swoimi sztuczkami hakerskimi, przesyłając PR-y do** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) na GitHubie.

</details>

## Poziomy integralności

W systemach Windows Vista i nowszych, wszystkie chronione elementy posiadają etykietę **poziomu integralności**. Domyślnie większość plików i kluczy rejestru otrzymuje "średni" poziom integralności, z wyjątkiem określonych folderów i plików, do których Internet Explorer 7 może zapisywać na niskim poziomie integralności. Domyślne zachowanie polega na nadawaniu procesom uruchamianym przez standardowych użytkowników poziomu integralności średniego, podczas gdy usługi zazwyczaj działają na poziomie integralności systemu. Etykieta wysokiej integralności chroni katalog główny.

Kluczową zasadą jest to, że obiekty nie mogą być modyfikowane przez procesy o niższym poziomie integralności niż poziom obiektu. Poziomy integralności to:

* **Niezaufany**: Ten poziom jest dla procesów z anonimowymi logowaniami. %%%Przykład: Chrome%%%
* **Niski**: Głównie dla interakcji internetowych, zwłaszcza w trybie chronionym Internet Explorera, wpływający na powiązane pliki i procesy oraz niektóre foldery, takie jak **Tymczasowy folder internetowy**. Procesy o niskim poziomie integralności mają znaczne ograniczenia, w tym brak dostępu do zapisu w rejestrze i ograniczony dostęp do zapisu w profilu użytkownika.
* **Średni**: Domyślny poziom dla większości działań, przypisany do standardowych użytkowników i obiektów bez określonych poziomów integralności. Nawet członkowie grupy Administratorzy działają domyślnie na tym poziomie.
* **Wysoki**: Zarezerwowany dla administratorów, umożliwiający im modyfikowanie obiektów na niższych poziomach integralności, w tym tych na poziomie wysokim.
* **Systemowy**: Najwyższy poziom operacyjny dla jądra systemu Windows i podstawowych usług, niedostępny nawet dla administratorów, zapewniający ochronę istotnych funkcji systemowych.
* **Instalatora**: Wyjątkowy poziom stojący ponad wszystkimi innymi, umożliwiający obiektom na tym poziomie odinstalowanie dowolnego innego obiektu.

Możesz uzyskać poziom integralności procesu za pomocą **Process Explorer** z **Sysinternals**, uzyskując dostęp do **właściwości** procesu i przeglądając kartę "**Bezpieczeństwo**":

![](<../../.gitbook/assets/image (821).png>)

Możesz także sprawdzić swoją **bieżącą integralność** za pomocą `whoami /groups`

![](<../../.gitbook/assets/image (322).png>)

### Poziomy integralności w systemie plików

Obiekt w systemie plików może wymagać **minimalnego wymaganego poziomu integralności**, a jeśli proces nie spełnia tego poziomu integralności, nie będzie mógł z nim współdziałać.\
Na przykład, stwórzmy **zwykły plik z konsoli użytkownika i sprawdźmy uprawnienia**:
```
echo asd >asd.txt
icacls asd.txt
asd.txt BUILTIN\Administrators:(I)(F)
DESKTOP-IDJHTKP\user:(I)(F)
NT AUTHORITY\SYSTEM:(I)(F)
NT AUTHORITY\INTERACTIVE:(I)(M,DC)
NT AUTHORITY\SERVICE:(I)(M,DC)
NT AUTHORITY\BATCH:(I)(M,DC)
```
Teraz przypiszmy minimalny poziom integralności **Wysoki** do pliku. To **należy zrobić z konsoli** uruchomionej jako **administrator**, ponieważ **zwykła konsola** działa na poziomie integralności Średni i **nie będzie uprawniona** do przypisania poziomu integralności Wysoki obiektowi:
```
icacls asd.txt /setintegritylevel(oi)(ci) High
processed file: asd.txt
Successfully processed 1 files; Failed processing 0 files

C:\Users\Public>icacls asd.txt
asd.txt BUILTIN\Administrators:(I)(F)
DESKTOP-IDJHTKP\user:(I)(F)
NT AUTHORITY\SYSTEM:(I)(F)
NT AUTHORITY\INTERACTIVE:(I)(M,DC)
NT AUTHORITY\SERVICE:(I)(M,DC)
NT AUTHORITY\BATCH:(I)(M,DC)
Mandatory Label\High Mandatory Level:(NW)
```
To jest moment, w którym rzeczy zaczynają się robić interesujące. Możesz zauważyć, że użytkownik `DESKTOP-IDJHTKP\user` ma **PEŁNE uprawnienia** do pliku (faktycznie to ten użytkownik utworzył plik), jednakże, z powodu zaimplementowanego minimalnego poziomu integralności, nie będzie mógł już modyfikować pliku, chyba że będzie działał na poziomie Wysokiej Integralności (zauważ, że nadal będzie mógł go czytać):
```
echo 1234 > asd.txt
Access is denied.

del asd.txt
C:\Users\Public\asd.txt
Access is denied.
```
{% hint style="info" %}
**Dlatego, gdy plik ma minimalny poziom integralności, aby go zmodyfikować, musisz działać przynajmniej na tym poziomie integralności.**
{% endhint %}

### Poziomy Integralności w Plikach Wykonywalnych

Skopiowałem `cmd.exe` do `C:\Windows\System32\cmd-low.exe` i ustawiłem mu **poziom integralności na niski z konsoli administratora:**
```
icacls C:\Windows\System32\cmd-low.exe
C:\Windows\System32\cmd-low.exe NT AUTHORITY\SYSTEM:(I)(F)
BUILTIN\Administrators:(I)(F)
BUILTIN\Users:(I)(RX)
APPLICATION PACKAGE AUTHORITY\ALL APPLICATION PACKAGES:(I)(RX)
APPLICATION PACKAGE AUTHORITY\ALL RESTRICTED APP PACKAGES:(I)(RX)
Mandatory Label\Low Mandatory Level:(NW)
```
Teraz, gdy uruchomię `cmd-low.exe`, **uruchomi się na niskim poziomie integralności** zamiast na średnim:

![](<../../.gitbook/assets/image (310).png>)

Dla ciekawskich, jeśli przypiszesz wysoki poziom integralności do pliku binarnego (`icacls C:\Windows\System32\cmd-high.exe /setintegritylevel high`), nie będzie on uruchamiany automatycznie z wysokim poziomem integralności (jeśli wywołasz go z poziomu integralności średniego - domyślnie uruchomi się na poziomie integralności średnim).

### Poziomy Integralności w Procesach

Nie wszystkie pliki i foldery mają minimalny poziom integralności, **ale wszystkie procesy działają na określonym poziomie integralności**. Podobnie jak w przypadku systemu plików, **jeśli proces chce pisać w innym procesie, musi mieć co najmniej ten sam poziom integralności**. Oznacza to, że proces o niskim poziomie integralności nie może otworzyć uchwytu z pełnym dostępem do procesu o poziomie integralności średnim.

Ze względu na ograniczenia omówione w tej i poprzedniej sekcji, z punktu widzenia bezpieczeństwa zawsze **zaleca się uruchamianie procesu na jak najniższym poziomie integralności**.
