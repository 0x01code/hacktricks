# macOS xpc\_connection\_get\_audit\_token Angriff

<details>

<summary><strong>Lernen Sie AWS-Hacking von Null auf Held mit</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Andere Möglichkeiten, HackTricks zu unterstützen:

* Wenn Sie Ihr **Unternehmen in HackTricks beworben sehen möchten** oder **HackTricks in PDF herunterladen möchten**, überprüfen Sie die [**ABONNEMENTPLÄNE**](https://github.com/sponsors/carlospolop)!
* Holen Sie sich das [**offizielle PEASS & HackTricks-Merchandise**](https://peass.creator-spring.com)
* Entdecken Sie [**The PEASS Family**](https://opensea.io/collection/the-peass-family), unsere Sammlung exklusiver [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Treten Sie der** 💬 [**Discord-Gruppe**](https://discord.gg/hRep4RUj7f) oder der [**Telegramm-Gruppe**](https://t.me/peass) bei oder **folgen** Sie uns auf **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Teilen Sie Ihre Hacking-Tricks, indem Sie PRs an die** [**HackTricks**](https://github.com/carlospolop/hacktricks) und [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub-Repositories einreichen.

</details>

**Für weitere Informationen überprüfen Sie den Originalbeitrag:** [**https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/**](https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/). Dies ist eine Zusammenfassung:

## Grundlegende Informationen zu Mach-Nachrichten

Wenn Sie nicht wissen, was Mach-Nachrichten sind, beginnen Sie mit der Überprüfung dieser Seite:

{% content-ref url="../../" %}
[..](../../)
{% endcontent-ref %}

Denken Sie vorerst daran, dass ([Definition von hier](https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing)):\
Mach-Nachrichten werden über einen _Mach-Port_ gesendet, der ein **Kommunikationskanal mit einem einzelnen Empfänger und mehreren Sendern** ist, der in den Mach-Kernel integriert ist. **Mehrere Prozesse können Nachrichten** an einen Mach-Port senden, aber zu jedem Zeitpunkt kann **nur ein einzelner Prozess daraus lesen**. Genau wie Dateideskriptoren und Sockets werden Mach-Ports vom Kernel zugewiesen und verwaltet, und Prozesse sehen nur eine Ganzzahl, die sie verwenden können, um dem Kernel anzuzeigen, welchen ihrer Mach-Ports sie verwenden möchten.

## XPC-Verbindung

Wenn Sie nicht wissen, wie eine XPC-Verbindung hergestellt wird, überprüfen Sie:

{% content-ref url="../" %}
[..](../)
{% endcontent-ref %}

## Schwachstellenzusammenfassung

Was für Sie interessant zu wissen ist, dass die Abstraktion von **XPC eine Eins-zu-Eins-Verbindung** ist, aber sie basiert auf einer Technologie, die **mehrere Sender haben kann, also:**

* Mach-Ports sind einzelne Empfänger, **mehrere Sender**.
* Das Audit-Token einer XPC-Verbindung ist das Audit-Token, das **aus der zuletzt empfangenen Nachricht kopiert wurde**.
* Das Erlangen des **Audit-Tokens** einer XPC-Verbindung ist für viele **Sicherheitsüberprüfungen** entscheidend.

Obwohl die vorherige Situation vielversprechend klingt, gibt es Szenarien, in denen dies keine Probleme verursachen wird ([von hier](https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing)):

* Audit-Token werden häufig für eine Autorisierungsprüfung verwendet, um zu entscheiden, ob eine Verbindung akzeptiert werden soll. Da dies über eine Nachricht an den Dienstport erfolgt, ist **noch keine Verbindung hergestellt**. Weitere Nachrichten an diesem Port werden einfach als zusätzliche Verbindungsanfragen behandelt. Daher sind **Überprüfungen vor der Annahme einer Verbindung nicht gefährdet** (das bedeutet auch, dass das Audit-Token innerhalb von `-listener:shouldAcceptNewConnection:` sicher ist). Wir suchen daher nach XPC-Verbindungen, die spezifische Aktionen überprüfen.
* XPC-Ereignishandler werden synchron behandelt. Das bedeutet, dass der Ereignishandler für eine Nachricht abgeschlossen sein muss, bevor er für die nächste aufgerufen wird, auch auf gleichzeitigen Dispatch-Warteschlangen. Daher kann das Audit-Token innerhalb eines **XPC-Ereignishandlers nicht von anderen normalen (nicht-Antwort-)Nachrichten überschrieben werden**.

Zwei verschiedene Methoden, wie dies ausgenutzt werden könnte:

1. Variante 1:
* Der **Exploit** **verbindet** sich mit Dienst **A** und Dienst **B**.
* Dienst **B** kann eine **privilegierte Funktionalität** in Dienst **A** aufrufen, die der Benutzer nicht kann.
* Dienst **A** ruft **`xpc_connection_get_audit_token`** auf, während es sich _**nicht**_ im **Ereignishandler** für eine Verbindung in einem **`dispatch_async`** befindet.
* Daher könnte eine **andere** Nachricht das **Audit-Token überschreiben**, da sie außerhalb des Ereignishandlers asynchron an den Kernel übermittelt wird.
* Der Exploit übergibt **Dienst B das SEND-Recht an Dienst A**.
* Daher wird svc **B tatsächlich die Nachrichten** an Dienst **A senden**.
* Der **Exploit** versucht, die **privilegierte Aktion aufzurufen**. In einem RC überprüft svc **A** die Autorisierung dieser **Aktion**, während **svc B das Audit-Token überschrieben hat** (was dem Exploit den Zugriff auf das Aufrufen der privilegierten Aktion ermöglicht).
2. Variante 2:
* Dienst **B** kann eine **privilegierte Funktionalität** in Dienst **A** aufrufen, die der Benutzer nicht kann.
* Der Exploit verbindet sich mit **Dienst A**, der dem Exploit eine **Nachricht erwartet, die eine Antwort** in einem bestimmten **Antwort-Port** sendet.
* Der Exploit sendet **Dienst B eine Nachricht**, die **diesen Antwort-Port** übergibt.
* Wenn Dienst **B antwortet**, **sendet er die Nachricht an Dienst A**, **während** der **Exploit** eine andere **Nachricht an Dienst A sendet**, um zu versuchen, eine privilegierte Funktionalität zu erreichen und zu erwarten, dass die Antwort von Dienst B das Audit-Token im perfekten Moment überschreibt (Race Condition).

## Variante 1: Aufruf von xpc\_connection\_get\_audit\_token außerhalb eines Ereignishandlers <a href="#variante-1-aufruf-von-xpc_connection_get_audit_token-outside-of-an-event-handler" id="variante-1-aufruf-von-xpc_connection_get_audit_token-outside-of-an-event-handler"></a>

Szenario:

* Zwei Mach-Dienste **`A`** und **`B`**, zu denen wir beide eine Verbindung herstellen können (basierend auf dem Sandbox-Profil und den Autorisierungsprüfungen vor der Annahme der Verbindung).
* _**A**_ muss eine **Autorisierungsprüfung** für eine spezifische Aktion haben, die **`B`** bestehen kann (aber unsere App nicht).
* Wenn B z.B. einige **Berechtigungen** hat oder als **root** ausgeführt wird, könnte es ihm erlauben, A aufzufordern, eine privilegierte Aktion auszuführen.
* Für diese Autorisierungsprüfung erhält **`A`** das Audit-Token asynchron, z.B. durch Aufruf von `xpc_connection_get_audit_token` aus **`dispatch_async`**.

{% hint style="danger" %}
In diesem Fall könnte ein Angreifer eine **Race Condition** auslösen, indem er einen **Exploit erstellt, der A auffordert, eine Aktion auszuführen**, während **B Nachrichten an `A` sendet**. Wenn die RC **erfolgreich** ist, wird das **Audit-Token** von **B** im Speicher **kopiert, während** die Anfrage unseres **Exploits** von A **bearbeitet** wird, was ihm **Zugriff auf die privilegierte Aktion ermöglicht, die nur B anfordern konnte**.
{% endhint %}

Dies geschah mit **`A`** als `smd` und **`B`** als `diagnosticd`. Die Funktion [`SMJobBless`](https://developer.apple.com/documentation/servicemanagement/1431078-smjobbless?language=objc) von smb kann verwendet werden, um ein neues privilegiertes Hilfsprogramm zu installieren (als **root**). Wenn ein **Prozess als root ausgeführt wird**, wird **smd** kontaktiert, und es werden keine weiteren Überprüfungen durchgeführt.

Daher ist der Dienst **B** **`diagnosticd`**, weil er als **root** ausgeführt wird und dazu verwendet werden kann, einen Prozess zu **überwachen**, sodass, sobald die Überwachung begonnen hat, **mehrere Nachrichten pro Sekunde gesendet werden**.

Um den Angriff durchzuführen:

1. Eine **Verbindung** zum Dienst mit dem Namen `smd` unter Verwendung des Standard-XPC-Protokolls herstellen.
2. Eine sekundäre **Verbindung** zu `diagnosticd` herstellen. Anstatt zwei neue Mach-Ports zu erstellen und zu senden, wird das Client-Port-Senderecht durch eine Kopie des **Senderechts** ersetzt, das mit der Verbindung zu `smd` verbunden ist.
3. Als Ergebnis können XPC-Nachrichten an `diagnosticd` gesendet werden, aber Antworten von `diagnosticd` werden an `smd` umgeleitet. Für `smd` sieht es so aus, als ob die Nachrichten sowohl vom Benutzer als auch von `diagnosticd` von derselben Verbindung stammen.

![Bild, das den Exploit-Prozess darstellt](https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/exploit.png)

4. Der nächste Schritt besteht darin, `diagnosticd` anzuweisen, die Überwachung eines ausgewählten Prozesses zu starten (möglicherweise des Benutzers). Gleichzeitig werden eine Flut von Routine-1004-Nachrichten an `smd` gesendet. Der Zweck hierbei ist die Installation eines Tools mit erhöhten Berechtigungen.
5. Diese Aktion löst eine Race Condition innerhalb der `handle_bless` Funktion aus. Das Timing ist entscheidend: Der Aufruf der Funktion `xpc_connection_get_pid` muss die PID des Benutzerprozesses zurückgeben (da das privilegierte Tool im App-Bundle des Benutzers liegt). Jedoch muss die Funktion `xpc_connection_get_audit_token`, speziell innerhalb der `connection_is_authorized` Unterfunktion, auf das Audit-Token von `diagnosticd` verweisen.

## Variante 2: Antwortweiterleitung

In einer XPC (Cross-Process Communication) Umgebung, obwohl Ereignisbehandler nicht gleichzeitig ausgeführt werden, hat die Verarbeitung von Antwortnachrichten ein einzigartiges Verhalten. Es existieren speziell zwei unterschiedliche Methoden zum Senden von Nachrichten, die eine Antwort erwarten:

1. **`xpc_connection_send_message_with_reply`**: Hier wird die XPC-Nachricht auf einer bestimmten Warteschlange empfangen und verarbeitet.
2. **`xpc_connection_send_message_with_reply_sync`**: Im Gegensatz dazu wird die XPC-Nachricht in dieser Methode auf der aktuellen Dispatch-Warteschlange empfangen und verarbeitet.

Diese Unterscheidung ist entscheidend, da sie die Möglichkeit bietet, dass **Antwortpakete gleichzeitig mit der Ausführung eines XPC-Ereignisbehandlers analysiert werden**. Insbesondere implementiert `_xpc_connection_set_creds` zwar eine Sperrung, um gegen die teilweise Überschreibung des Audit-Tokens abzusichern, erstreckt diesen Schutz jedoch nicht auf das gesamte Verbindungsobjekt. Folglich entsteht eine Schwachstelle, bei der das Audit-Token während des Intervalls zwischen dem Parsen eines Pakets und der Ausführung seines Ereignisbehandlers ersetzt werden kann.

Um diese Schwachstelle auszunutzen, ist folgendes Setup erforderlich:

* Zwei Mach-Services, bezeichnet als **`A`** und **`B`**, die beide eine Verbindung herstellen können.
* Service **`A`** sollte eine Autorisierungsprüfung für eine spezifische Aktion enthalten, die nur **`B`** ausführen kann (die Anwendung des Benutzers nicht).
* Service **`A`** sollte eine Nachricht senden, die eine Antwort erwartet.
* Der Benutzer kann eine Nachricht an **`B`** senden, auf die es antworten wird.

Der Ausbeutungsprozess umfasst die folgenden Schritte:

1. Warten, bis Service **`A`** eine Nachricht sendet, die eine Antwort erwartet.
2. Anstatt direkt an **`A`** zu antworten, wird der Antwort-Port übernommen und verwendet, um eine Nachricht an Service **`B`** zu senden.
3. Anschließend wird eine Nachricht mit der verbotenen Aktion versandt, in der Erwartung, dass sie gleichzeitig mit der Antwort von **`B`** verarbeitet wird.

Nachfolgend ist eine visuelle Darstellung des beschriebenen Angriffsszenarios:

![https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/variant2.png](../../../../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1).png)

<figure><img src="../../../../../../.gitbook/assets/image (33).png" alt="https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/variant2.png" width="563"><figcaption></figcaption></figure>

## Probleme bei der Entdeckung

* **Schwierigkeiten bei der Lokalisierung von Instanzen**: Die Suche nach Instanzen der Verwendung von `xpc_connection_get_audit_token` war sowohl statisch als auch dynamisch herausfordernd.
* **Methodik**: Frida wurde verwendet, um die Funktion `xpc_connection_get_audit_token` zu hooken und Anrufe zu filtern, die nicht aus Ereignisbehandlern stammen. Diese Methode war jedoch auf den gehookten Prozess beschränkt und erforderte aktive Nutzung.
* **Analysetools**: Tools wie IDA/Ghidra wurden verwendet, um erreichbare Mach-Services zu untersuchen, was zeitaufwändig war und durch Aufrufe, die den dyld Shared Cache involvieren, kompliziert wurde.
* **Skripting-Einschränkungen**: Versuche, die Analyse für Anrufe von `xpc_connection_get_audit_token` aus `dispatch_async`-Blöcken zu skripten, wurden durch Komplexitäten beim Parsen von Blöcken und Interaktionen mit dem dyld Shared Cache behindert.

## Die Lösung <a href="#the-fix" id="the-fix"></a>

* **Gemeldete Probleme**: Ein Bericht wurde an Apple eingereicht, der die allgemeinen und spezifischen Probleme in `smd` beschreibt.
* **Antwort von Apple**: Apple hat das Problem in `smd` behoben, indem `xpc_connection_get_audit_token` durch `xpc_dictionary_get_audit_token` ersetzt wurde.
* **Art der Lösung**: Die Funktion `xpc_dictionary_get_audit_token` gilt als sicher, da sie das Audit-Token direkt aus der Mach-Nachricht abruft, die mit der empfangenen XPC-Nachricht verbunden ist. Allerdings ist sie nicht Teil der öffentlichen API, ähnlich wie `xpc_connection_get_audit_token`.
* **Fehlen einer umfassenderen Lösung**: Es bleibt unklar, warum Apple keine umfassendere Lösung implementiert hat, wie beispielsweise das Verwerfen von Nachrichten, die nicht mit dem gespeicherten Audit-Token der Verbindung übereinstimmen. Die Möglichkeit legitimer Änderungen des Audit-Tokens in bestimmten Szenarien (z.B. `setuid`-Verwendung) könnte ein Faktor sein.
* **Aktueller Status**: Das Problem besteht weiterhin in iOS 17 und macOS 14 und stellt eine Herausforderung für diejenigen dar, die es identifizieren und verstehen möchten.
