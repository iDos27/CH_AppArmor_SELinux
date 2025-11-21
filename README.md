## Slajd 1:
**[Wygląd]**
* **Tytuł:** Zabezpieczenia Systemu Linux
* **Podtytuł:** Od iptablec do AppArmor i SELinux

---

## Slajd 2:
**[Wygląd]**
* **Tytuł:** iptables i Netfilter
* **Tekst:** **iptables** (lub nftables) działa na warstwie sieciowej. Jego zadaniem jest filtrowanie pakietów wchodzących i wychodzących.
* **Lista**
    * Decyduje, kto może się połączyć (IP, Port)
    * Blokuje niechciany ruch sieciowy
    * Pierwsza linia obrony
* **Komenda** `iptables -A INPUT -p tcp --dport 80 -j ACCEPT`
* **Obraz** ![Obraz działania firewalla](https://linuxkamarada.com/files/2019/11/iptables.jpg)

**[Notatki]**
Przez lata bezpieczeństwo kojarzyło się głównie z tym, co wpuszczamy do sieci.
Iptables działa jak strażnik na bramce
Firewall sprawdza nagłówki pakietów: skąd są, dokąd idą. Decyzja jest prosta: wpuścić albo odrzucić

---

## Slajd 3:
**[Wygląd]**
* **Tytuł:** Dlaczego iptables nie wystarcza?
* **Tekst:** Zagrożenie przychodzi legalnym kanałem. Ataki jak SQL Injection lub Remocte Code Execution przechodzą przez otwarte porty i działają na systemie serwera
* **Obraz** ![Koń Trojański](https://i.pinimg.com/564x/3a/f1/63/3af1633b3a210a25820a86b2311d5c39.jpg)

**[Notatki]**
Największym problemem z iptables to fakt, że chronią tylko drogę pakietu.
Jeśli udostępniamy stronę WWW, musimy otworzyć port 80. Iptables wpuszcza wszystko.
Jak strona posiada jakikolwiek błąd to atakujący wchodzi "głównym wejściem". Intruz jest w systemie i może zrobić co tylko chcę.

---

## Slajd 4:
**[Wygląd]**
* **Tytuł:** DAC - Discrretionary Access Control
* **Tekst:** **Standordowe Uprawnienia Linux**
* W modelu DAC właściciel decyduje o dostępie
* **Lista**
    * Opiera się na tożsamości: `user`, `group`, `others`.
    * Uprawnienia: `r`(read), `w`(write), `x`(execute)
* *Problem:* Jeśli proces działa jako root lub użytkownik z dostępem do danych. Atakujący może zrobić z nim co tylko zechcę.
* **Obraz** ![Obraz działania DAC](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F9vcq00kpbmuuahoffegd.png)

**[Notatki]**
Standardowy Linux używa DAC. Właściciel pliku decyduje o dostępie.
Kluczowy problem jest kiedy serwer WWW działa jako użytkownik i zostanie przejęty, wirus ma te same uprawnienia co użytkownik.

---

## Slajd 5:
**[Wygląd]**
* **Tytuł:** MAC: Mandatory Access Control
* **Tekst:** W modelu MAC system operacyjny wymusza politykę bezpieczeństwa zdefiniowaną przez administratora
* **Lista**
    * Użytkownik nie może zmienić polityki dla procesu.
    * Działa na zasadzie "wszystko, co nie jest dozwolone jest zabronione"
    * Ogranicza szkody w przypadku przejęcia
* **Obraz** ![Działanie MAC](https://aimultiple.com/wp-content/uploads/2024/02/mac.png)

**[Notatki]**
MAC to zmiana paradygmatu. Tutaj polityka systemu jest ważniejsza od użytkownika.
Nawet jeśli proces działa jako root, ale polityka MAC mowi, że nie wolno dotykać /etc/shadow to system zablokuje dostęp.
To druga linia obrony. Ona nie przeszkadza we włamaniu ale ogranicza jego skutki

---

## Slajd 6#:
**[Wygląd]**
* **Tytuł:** Czym jest AppArmor
* **Tekst:** AppArmor to moduł bezpieczeństwa jądra Linux, który ogranicza możliwości programów. Domyślnie używany w Debian, Ubuntu i starsze wersje SUSE.
* **Lista**
    * *Prostota:* Łatwiejszy do nauki i konfiguracji niż SELinux
    * *Path-based:* Polityki bezpieczeństwa są przypisywane do ścieżek plików
* **Obraz** ![logo AppArmor](https://upload.wikimedia.org/wikipedia/commons/thumb/b/ba/AppArmor_logo.svg/1200px-AppArmor_logo.svg.png)

**[Notatki]**
Jest to moduł bezpieczeństwa wspierany przez Canonical, czyli twórców Ubuntu.
AppArmor identyfikuje zasoby po ścieżkach w systemie plików.
Profil bezpieczeństwa jest przypięty do konkretnej ścieżki pliku.

---

## Slajd 7:
**[Wygląd]**
* **Tytuł:** Truby pracy AppArmor

| Enforce Mode | Complain Mode | Unconfined |
| :--- | :--- | :--- |
| System blokuje działania niezgodne z profilem i loguje próbe naruszenia. Docelowy dla produkcji | System zezwala na działanie, ale loguje każde naruszenie polityki. Używany do tworzenia nowych profili | Proces nie jest ograniczony przez AppArmor |

**[Notatki]**
Największą siłą AppArmor to tryb Complain.
Uruchamiamy aplikacje, pozwalamy jej działać a następnie jedną komendą na podstawie logów tworzymy profil.


---

## Slajd 8:
**[Wygląd]**
* **Tytuł:** Konfiguracja AppArmor
* **Kolumna Lewa** Zarządzanie profilami
* **Tekst Lewy** Profile znajdują się w `/etc/apparmor.d/`.
Przykładowy profil dla `/usr/bin/nginx`:
```apparmor
/usr/sbin/nginx {
  #include <abstractions/base>
  
  capability dac_override,
  capability net_bind_service,

  /var/www/html/** r,
  /var/log/nginx/*.log rw,
  /run/nginx.pid w,
}
```
* **Kolumna Prawa** Przydatne Komendy
* **Tekst Prawy** `aa-status`: Sprawdza status modułu i załadowane profile.
`aa-enforce <profil>`: Przełącza profil w tryb blokowania.
`aa-complain <profil>`: Przełącza profil w tryb nauki.
`aa-genprof <program>`: Kreator generowania nowego profilu.
`aa-logprof`: Analiza logów i automatyczna aktualizacja profilu.

**[Notatki]**
*#include..* to zestaw podstawowych uprawnień
*dac_override* - Pozwala obchodzić standardowe prawa dostępu do plików. Nginx używa gdy jako root otwiera logi.
*net_bind_service* Pozwala wiązać porty. Nginx używa portu 80 i 443. Bez tego serwis nie mógłby działać
*Reguły dostępu do plików:*
/var/www/html/** r Pozwala czytać pliki i podkatalogi z folderu 
/var/log/nginx/*.log rw Nginx może pisać i czytać pliki logów
/run/nginx.pid w Nginx może zapisywać swój numer PID

---

## Slajd 9:
**[Wygląd]**
* **Tytuł:** Czym jest SELinux?
* **Tekst:** SELinux to system stworzony pierwotnie przez NSA, następnie oddany pod opiekę Red Hat. Jest domyślnie używany w całej rodzinie Red Hat (RHEL, Fedora, CentOS) i SUSE
* **Lista**
    * **Label-based:** Każdy plik, proces i port ma etykiete.
    * **Type Enforcement:** Reguły określają, jak dany typ procesu może oddziaływać na dany typ pliku.
    * **Bardzo szczegółowy:** Oferuje niezwykle precyzyjną kontrolę, ale jest trudniejszy w konfiguracji.
* **Obraz** ![Logo SELinux](https://upload.wikimedia.org/wikipedia/commons/thumb/1/1e/SELinux_logo.svg/1133px-SELinux_logo.svg.png)

**[Notatki]**
SELinux to zupełnie inne myślenie o bezpieczeństwie. Tutaj nie liczy się ścieżka do pliku bądz katalogu a etykieta, która jest trwale zapisana w systemie plików.
W AppArmor jak zmienimy nazwę pliku czy katalog w którym się znajduje to całe bezpieczeństwo idzie do kosza. W SELinux etykieta jest częścią pliku i za nim podąża.
SELinux jest rozwiązaniem dużo bardziej szczegółowym i trudniejszym do obejścia dlatego twórcy SUSE przenieśli się na to rozwiązanie

---

## Slajd 10:
**[Wygląd]**
* **Tytuł:** Etykiety SELinux
* **Kolumna Lewa**:
    * **Struktura etykiety**
    * **user:role:type:level**
    * Przykład dla pliku Nginx:
    * `system_u:object_r:httpd_sys_content_t:s0`
    * Najważniejszy jest **TYPE**(np. `httpd_t` dla procesu Nginx, httpd_sys_content_t dla plików HTML)
* **Kolumna Prawa**:
    * **Jak to działa?**
    * Polityka SELinux mówi:
    * `allow httpd_t httpd_sys_content_t:file { read getattr };`
    * Jeśli proces `httpd_t` spróbuje odczytać plik o typie shadow_t (/etc/shadow), SELinux zablokuje dostęp, bo nie ma na to reguły "allow"

---

## Slajd 11:
**[Wygląd]**
* **Tytuł:** Konfiguracja SELinux
* **Komendy Podstawowe**
    * `sestatus`: Sprawdź stan SELinux
    * `ls -Z` : Pokaż kontekst plików.
    * `ps -Z` : Pokaż kontekst procesów.
    * `setenforce 0|1` : Przełącz tryb tymczasowo.
    * `/etc/selinux/config` : Konfiguracja trwała.
* **Zarządzanie Polityką**
    * `chcon` : Tymczasowa zmiana kontekst pliku
    * `restorecon -Rv /var/www` : Przywraca domyślne konteksty
    * `semanage fcontext` : Definiowanie trwałych reguł kontekstów.
    * `audit2allow` : Generowanie reguł na podstawie logów błędów (audit.log)

---

## Slajd 12:
**[Wygląd]**
* **Tytuł:** SELinux Booleans
* **Przełączniki Funkcji**
    * Booleans to proste przełączniki w polityce SELinux, które pozwalają na zmianę zachowania bez pisania nowych reguł.
    * Przykład: Czy serwer HTTP może łączyć się z bazą danych?
* **Przykład Użycia**
```bash
    # Sprawdzenie dostępnych opcji
    getsebool -a | grep httpd

    # Zezwolenie Nginx na sieć
    setsebool -P httpd_can_network_connect 1

    # Zezwolenie na zapis w katalogach
    setsebool -P httpd_enable_homedirs 1
```

---

## Slajd 13:
**[Wygląd]**
* **Tytuł:** Porównanie: AppArmor vs SELinux

| Cecha | Apparmor | SELinux
| :--- | :--- | :--- |
| Model identyfikacji | Ścieżki | Etykiety |
| Poziom trudności | Niski/Średni | Wysoki |
| Domyślny w | Ubuntu, Debian, Arch | RHEL, Fedora, Android, SUSE |
| Ochrona na zmiane nazwy | Jeśli nazwa pliku/ścieżki się zmieni, reguła przestaje działać | Ochrona "podąża" za plikiem |
| Konfiguracja | Czytelne pliki tekstowe | Skomplikowane moduły wymagające narzędzi jak semanage |
| Zasięg ochrony | Skupiony na konkretnych **programach** | **Cały system** |



## Slajd #:
**[Wygląd]**
* **Tytuł:**
* **Tekst:**
* **Lista**
    * punkt 1
    * punkt 2
* **Komenda** `rm -rf *`
* **Obraz** ![alt txt](linkdojpg)

**[Notatki]**
Tekst notatek

---
