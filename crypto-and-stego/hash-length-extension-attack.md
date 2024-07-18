# Hash Length Extension Attack

{% hint style="success" %}
Learn & practice AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Learn & practice GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Check the [**subscription plans**](https://github.com/sponsors/carlospolop)!
* **Join the** 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) or the [**telegram group**](https://t.me/peass) or **follow** us on **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Share hacking tricks by submitting PRs to the** [**HackTricks**](https://github.com/carlospolop/hacktricks) and [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}

#### [WhiteIntel](https://whiteintel.io)

<figure><img src="../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) is a **dark-web** fueled search engine that offers **free** functionalities to check if a company or its customers have been **compromised** by **stealer malwares**.

Their primary goal of WhiteIntel is to combat account takeovers and ransomware attacks resulting from information-stealing malware.

You can check their website and try their engine for **free** at:

{% embed url="https://whiteintel.io" %}

***

## Summary of the attack

Уявіть собі сервер, який **підписує** деякі **дані**, **додаючи** **секрет** до деяких відомих відкритих текстових даних, а потім хешуючи ці дані. Якщо ви знаєте:

* **Довжину секрету** (це також можна перебрати з заданого діапазону довжин)
* **Відкриті текстові дані**
* **Алгоритм (і він вразливий до цієї атаки)**
* **Паддінг відомий**
* Зазвичай використовується стандартний, тому якщо виконуються інші 3 вимоги, це також так
* Паддінг змінюється в залежності від довжини секрету + даних, тому довжина секрету потрібна

Тоді зловмисник може **додати** **дані** і **згенерувати** дійсну **підпис** для **попередніх даних + доданих даних**.

### How?

В основному, вразливі алгоритми генерують хеші, спочатку **хешуючи блок даних**, а потім, **з** **раніше** створеного **хешу** (стану), вони **додають наступний блок даних** і **хешують його**.

Тоді уявіть, що секрет - "secret", а дані - "data", MD5 "secretdata" дорівнює 6036708eba0d11f6ef52ad44e8b74d5b.\
Якщо зловмисник хоче додати рядок "append", він може:

* Згенерувати MD5 з 64 "A"
* Змінити стан раніше ініціалізованого хешу на 6036708eba0d11f6ef52ad44e8b74d5b
* Додати рядок "append"
* Завершити хеш, і отриманий хеш буде **дійсним для "secret" + "data" + "padding" + "append"**

### **Tool**

{% embed url="https://github.com/iagox86/hash_extender" %}

### References

Ви можете знайти цю атаку добре поясненою на [https://blog.skullsecurity.org/2012/everything-you-need-to-know-about-hash-length-extension-attacks](https://blog.skullsecurity.org/2012/everything-you-need-to-know-about-hash-length-extension-attacks)

#### [WhiteIntel](https://whiteintel.io)

<figure><img src="../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) is a **dark-web** fueled search engine that offers **free** functionalities to check if a company or its customers have been **compromised** by **stealer malwares**.

Their primary goal of WhiteIntel is to combat account takeovers and ransomware attacks resulting from information-stealing malware.

You can check their website and try their engine for **free** at:

{% embed url="https://whiteintel.io" %}

{% hint style="success" %}
Learn & practice AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Learn & practice GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Check the [**subscription plans**](https://github.com/sponsors/carlospolop)!
* **Join the** 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) or the [**telegram group**](https://t.me/peass) or **follow** us on **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Share hacking tricks by submitting PRs to the** [**HackTricks**](https://github.com/carlospolop/hacktricks) and [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}
