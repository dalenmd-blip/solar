# Integracja: Beckhoff (master PROFIBUS DP) ↔ KEB COMBIVERT F5 (slave PROFIBUS DP)

Scenariusz: sterownik Beckhoff (CX/CP + TwinCAT, z terminalem **EL6731** lub
kartą **FC310x** jako master PROFIBUS DP) steruje jednym lub wieloma
falownikami **KEB COMBIVERT F5** wyposażonymi w moduł rozszerzeń
**PROFIBUS-DP Operator (nr kat. 00.F5.060-3000)**, np. w instalacji solarnej
sterującej napędami pomp lub nadążników.

## 1. Architektura

```
Beckhoff CX/CP (TwinCAT PLC)
        │  EtherCAT
        ▼
   EL6731 (Master PROFIBUS DP, adres domyślny = 1)
        │  PROFIBUS DP, RS-485, do 12 Mbit/s
        ├── KEB F5 #1  (PROFIBUS-DP Operator, adres np. 3)
        ├── KEB F5 #2  (PROFIBUS-DP Operator, adres np. 4)
        └── ... (max 125 stacji na magistrali/segment, w praktyce mniej
               ze względu na czas cyklu)
```

Rola falownika F5 w sieci PROFIBUS DP to **pasywny użytkownik (slave)** —
odpowiada tylko na zapytania mastera; sam nigdy nie inicjuje transmisji.
Master (Beckhoff) najpierw wykonuje sekwencję **parametryzacji** i
**konfiguracji** slave'a, a dopiero potem rozpoczyna cykliczną wymianę
danych użytkownika (Data Exchange).

## 2. Warstwa fizyczna i adresowanie

- Kabel: ekranowana skrętka RS-485, złącza D-sub 9-pin, topologia liniowa.
- **Terminacja**: rezystory końcowe aktywne na obu skrajnych urządzeniach
  segmentu (typowo przełącznik na złączu D-sub lub na urządzeniu). Brak
  terminacji lub podwójna terminacja to najczęstsza przyczyna błędów typu
  „bus fault” / utraty komunikacji.
- **Adres stacji na module PROFIBUS-DP Operator F5**: ustawiany sprzętowo
  (przełączniki węzłowe na module/falowniku, wartość szesnastkowa) i/lub
  odzwierciedlany w parametrze systemowym falownika **`Sy.06` — adres
  falownika**. Adres musi być unikalny na segmencie i zgodny z adresem
  wprowadzonym w konfiguracji TwinCAT dla danego slave'a.
- **Prędkość transmisji**: PROFIBUS DP zwykle wykrywana automatycznie przez
  slave'a na podstawie ramek mastera; falownik F5 dodatkowo posiada
  parametry `Sy.07` (prędkość magistrali zewnętrznej) i `Sy.11` (prędkość
  magistrali wewnętrznej — HSP5 między modułem Operator a rdzeniem
  falownika) — przy problemach z komunikacją warto je zweryfikować.

## 3. Plik GSD

Beckhoff **nie dostarcza** plików GSD dla urządzeń innych producentów.
Plik GSD dla modułu PROFIBUS-DP Operator F5 trzeba pozyskać **od KEB**
(dokumentacja produktu / support KEB) i:

1. skopiować plik `.gsd`/`.gse` do katalogu GSD instalacji TwinCAT
   (typowo `TwinCAT\3.1\Config\Io\Profibus` lub `TwinCAT\IO\GSD`),
2. w System Managerze/TwinCAT XAE dodać urządzenie pod masterem EL6731 —
   jeśli GSD jest poprawnie umieszczony, falownik KEB powinien pojawić się
   automatycznie na liście urządzeń (pogrupowanych wg producenta); w
   przeciwnym razie użyć opcji ręcznego wskazania pliku GSD („General
   PROFIBUS Box (GSD)”),
3. ustawić w konfiguracji adres stacji zgodny z adresem sprzętowym
   falownika,
4. zmapować dane procesowe (patrz niżej) na zmienne programu PLC.

## 4. Model wymiany danych: PKW + PZD

Falowniki KEB, podobnie jak większość napędów zgodnych z profilem
**PROFIdrive**, rozdzielają komunikację na dwa kanały w ramach jednej ramki
cyklicznej:

- **PKW (Parameter-Kennung-Wert / Parameter ID-Value)** — kanał acykliczny
  „w tunelu cyklicznym”, służący do odczytu/zapisu pojedynczych parametrów
  falownika (np. zmiana rampy, odczyt kodu błędu, zmiana zestawu
  parametrów CP1/CP2/CP3) bez potrzeby przeprogramowywania konfiguracji
  procesowej. Parametry modułu PROFIBUS-DP Operator są adresowane jako
  **PNU zaczynające się od offsetu `XX80h`** względem numeracji
  wewnętrznej falownika.
- **PZD (Prozessdaten / Process Data)** — kanał cykliczny czasu
  rzeczywistego: słowo sterujące/zadana wartość (master → slave) i słowo
  statusowe/wartość rzeczywista (slave → master), wymieniane w każdym
  cyklu magistrali.

Ograniczenia telegramu: **w trybie normalnym pojedyncza ramka PZD przenosi
maks. 8 bajtów danych procesowych** na jedno wewnętrzne PDO; do przesłania
16 bajtów potrzebne są dwa wewnętrzne PDO. Przy planowaniu obrazu procesu
(np. zadana prędkość + słowo sterujące + dodatkowy setpoint) trzeba to
uwzględnić w konfiguracji modułu.

Wewnętrznie moduł Operator komunikuje się z rdzeniem falownika protokołem
własnym KEB **HSP5** — dla integratora jest to transparentne, ale warto o
tym pamiętać przy diagnozowaniu opóźnień (dwa „piętra” komunikacji:
PROFIBUS DP zewnętrznie, HSP5 wewnętrznie).

## 5. Słowo sterujące i statusowe (control word / status word)

KEB implementuje na poziomie mastera obsługę profilu **PROFIdrive-PKW**, co
w praktyce oznacza strukturę zbliżoną do standardowego telegramu
PROFIdrive (np. Telegram 1: STW1/ZSW1 + zadana/rzeczywista prędkość).
Typowy, ogólny wzorzec bitów słowa sterującego (STW1) w profilu
PROFIdrive, który należy **zweryfikować z dokładną tabelą w instrukcji KEB
F5 „Status and Control Word”** (istnieją różnice zależne od wersji
oprogramowania i wybranego trybu telegramu):

| Bit | Nazwa (typowo) | Znaczenie |
|---|---|---|
| 0 | ON / OFF1 | 1 = załącz (start), 0 = zatrzymanie po rampie (OFF1) |
| 1 | OFF2 | 0 = zatrzymanie swobodnym wybiegiem (coast stop) |
| 2 | OFF3 | 0 = szybkie zatrzymanie (quick stop) |
| 3 | Operation enable | 1 = zezwolenie na pracę (impulsy załączone) |
| 4–6 | Ramp generator enable/hold/enable setpoint | sterowanie generatorem rampy |
| 7 | Fault acknowledge | zbocze narastające kasuje aktywny błąd |
| 8–9 | Jog1 / Jog2 | tryb impulsowy (jog) |
| 10 | Control by PLC | 1 = sterowanie z magistrali aktywne |
| 11 | Kierunek / znak zadanej wartości | zależnie od implementacji |
| 12–15 | Bity specyficzne dla producenta | zależnie od KEB |

Analogicznie słowo statusowe (ZSW1) zwraca m.in. bity: „ready to switch
on”, „ready to operate”, „operation enabled”, „fault present”, „coast stop
active” itd. — również do zweryfikowania w dokumentacji KEB.

**To jest wzorzec ogólny profilu PROFIdrive** przywołany dla ułatwienia
zrozumienia architektury — przed uruchomieniem produkcyjnym należy
bezwzględnie sprawdzić rzeczywisty układ bitów w rozdziale „Status and
Control Word” instrukcji aplikacyjnej KEB COMBIVERT F5 (moduł PROFIBUS-DP
Operator), ponieważ część bitów bywa specyficzna dla KEB.

## 6. Typowy scenariusz — instalacja solarna

Przykładowy przypadek użycia dla repozytorium `solar`: sterownik Beckhoff
jako nadrzędny kontroler instalacji PV, sterujący po PROFIBUS DP
falownikami KEB F5 napędzającymi np. pompy w systemie pompowania wody
zasilanym z fotowoltaiki lub siłowniki/napędy nadążników (trackerów):

1. **Pomiar dostępnej mocy PV** (np. przez wejścia analogowe/cyfrowe PLC
   lub magistralę z falownikiem PV) → PLC wylicza dopuszczalną prędkość/moc
   dla napędu.
2. **Zadawanie prędkości/momentu** do KEB F5 przez kanał PZD (setpoint w
   ramce cyklicznej), z uwzględnieniem ograniczeń (np. soft-start,
   ograniczenie prądu przy niskim nasłonecznieniu).
3. **Start/stop i potwierdzanie błędów** przez bity słowa sterującego
   (ON/OFF1, fault acknowledge) — istotne np. przy zabezpieczeniu przed
   pracą na sucho pompy (dry-run) sygnalizowanym z czujnika poziomu wody,
   wpiętego do PLC.
4. **Monitoring stanu napędu** (prędkość rzeczywista, prąd, kody błędów)
   przez słowo statusowe i kanał PKW — archiwizacja/wizualizacja w
   TwinCAT HMI/Scope.
5. **Przełączanie profili pracy** (np. dzień/noc, sezonowe ograniczenia)
   przez zmianę zestawu parametrów `CP1`/`CP2`/`CP3` z poziomu PLC (zapis
   parametru przez kanał PKW).

## 7. Checklist uruchomieniowy

- [ ] Terminacja magistrali PROFIBUS aktywna na obu końcach segmentu.
- [ ] Unikalne adresy stacji dla każdego falownika F5 (zgodne z
      konfiguracją TwinCAT).
- [ ] Plik GSD modułu PROFIBUS-DP Operator F5 pobrany od KEB i wgrany do
      katalogu GSD TwinCAT.
- [ ] Prędkość transmisji ustawiona spójnie (zwykle auto-detekcja po
      stronie slave'a — zweryfikować `Sy.07`/`Sy.11` w razie problemów).
- [ ] Rozmiar telegramu PZD dopasowany do liczby wymienianych słów
      procesowych (pamiętać o limicie 8 bajtów/PDO).
- [ ] Zweryfikowany rzeczywisty układ bitów STW/ZSW z instrukcji KEB (nie
      zakładać wprost tabeli ogólnej z punktu 5).
- [ ] Test sekwencji start/stop, OFF2 (coast), OFF3 (quick stop) i
      kasowania błędu przed podłączeniem napędu pod obciążenie.
- [ ] Test zachowania przy zaniku komunikacji PROFIBUS (watchdog/timeout
      po stronie falownika — reakcja typu „stop bezpieczny” skonfigurowana
      świadomie).

## 8. Typowe problemy

| Objaw | Prawdopodobna przyczyna |
|---|---|
| Slave nie pojawia się w konfiguracji TwinCAT | brak/zły plik GSD, niezgodna wersja GSD z wersją firmware modułu Operator |
| Bus fault / ciągłe restarty komunikacji | brak terminacji lub podwójna terminacja, uszkodzony kabel/ekranowanie, konflikt adresów |
| Falownik nie reaguje na polecenia (control word) | brak bitu „control by PLC”/zezwolenia, nieaktywne „operation enable”, niezgodny układ bitów z założeniami |
| Wartości procesowe „poszatkowane”/nielogiczne | zła konfiguracja rozmiaru telegramu PZD (np. założono 16 B, a skonfigurowano jedno PDO 8 B) |
| Komunikacja działa, ale zmiana parametru (CP1 itd.) nie działa | błędny PNU/offset w kanale PKW — zweryfikować numerację względem `XX80h` |

## Źródła

- [KEB COMBIVERT F5-A — Applications Manual, str. 501 „Profibus-DP Operator F5 00.F5.060-3000” (ManualsLib)](https://www.manualslib.com/manual/1808036/Keb-Combivert-F5-A.html?page=501)
- [KEB COMBIVERT F5-A — Applications Manual, str. 505 „Bus Parameter; Inverter Address (Sy.06); Baud Rate” (ManualsLib)](https://www.manualslib.com/manual/1808036/Keb-Combivert-F5-A.html?page=505)
- [KEB COMBIVERT F5-A — Applications Manual, str. 507 „Status and Control Word” (ManualsLib)](https://www.manualslib.com/manual/1808036/Keb-Combivert-F5-A.html?page=507)
- [KEB COMBIVERT F5 MULTI — Applications Manual, „Control and Status Word” (ManualsLib)](https://www.manualslib.com/manual/1930817/Keb-Combivert-F5-Multi.html?page=366)
- [KEB — 00F5060-3000 PROFIBUS-DP Operator Panel for Combivert F5 Series (volt-werk.com)](https://www.volt-werk.com/products/keb-00-f5-060-3000-profibus-dp-operator-panel-for-combivert-f5-series)
- [KEB COMBIVERT F5 Fieldbus Operator Manual (PROFINET Operator, dla porównania struktury) (mans.io)](https://mans.io/files/viewer/1235667/1)
- [KEB COMBICOM ModBus RTU — Instruction Manual (dla porównania architektury modułów Operator) (keb.su)](https://www.keb.su/doc/Instrukciya%20KEB%20F5-ModBus%20RTU%20(engl).pdf)
- [Beckhoff EL6731 — dokumentacja (mirror PDF)](https://filfar.by/PDF/beckhoff/el6731en.pdf)
- [Beckhoff Information System — Configuring a fieldbus master (EL6731)](https://infosys.beckhoff.com/content/1033/el6731/2385319435.html)
- [Beckhoff Information System — Adding GSD and EDS Boxes](https://infosys.beckhoff.com/content/1033/tcsystemmanager/1087231243.html)
- PROFIdrive — ogólny profil aplikacyjny PI (PROFIBUS & PROFINET
  International), przywołany jako punkt odniesienia dla struktury
  STW/ZSW — do zweryfikowania z konkretną implementacją KEB.

> Uwaga: jak w pozostałych dokumentach tego katalogu, treść oparto na
> fragmentach z wyszukiwarki (WebFetch był zablokowany) i ogólnej wiedzy o
> architekturze PROFIBUS/PROFIdrive. **Wartości krytyczne dla bezpieczeństwa
> i poprawności działania (układ bitów STW/ZSW, dokładne offsety PNU,
> limity telegramu) należy potwierdzić w oryginalnych, aktualnych
> instrukcjach KEB i Beckhoff przed uruchomieniem instalacji.**
