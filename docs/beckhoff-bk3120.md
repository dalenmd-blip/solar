# Beckhoff BK3120 — Bus Coupler PROFIBUS DP

## 1. Co to jest

**BK3120** to **Bus Coupler** ("Economy plus") z rodziny **BK3xx0**, będący
slave'em **PROFIBUS DP**. W odróżnieniu od modułu PROFIBUS-DP Operator w
falowniku KEB F5 (patrz
[`integration-beckhoff-keb-f5-profibus.md`](./integration-beckhoff-keb-f5-profibus.md)),
BK3120 **nie steruje napędem** — jest węzłem zbierającym zwykłe **Bus
Terminale serii KL** (cyfrowe/analogowe wejścia-wyjścia, liczniki, itd.) po
wewnętrznej magistrali **K-bus** i udostępniającym je masterowi PROFIBUS
jako jedną stację. W instalacji solarnej typowo agreguje sygnały polowe:
czujniki poziomu wody, krańcówki, presostaty, wejścia analogowe z czujników
itp.

## 2. Kluczowe dane techniczne

| Parametr | Wartość |
| --- | --- |
| Interfejs | PROFIBUS DP / DPV1, auto-detekcja prędkości transmisji do 12 Mbaud |
| Maks. obraz procesu | 128 B wejść / 128 B wyjść |
| Liczba podłączonych Bus Terminali (K-bus) | do 255 (fizycznie ograniczone też budżetem prądowym) |
| Adresacja stacji | 2 przełączniki obrotowe (rotary) — dolny = dziesiątki adresu, górny = jedynki |
| Zasilanie | 24 VDC (-15%/+20%), pobór własny 70 mA + (prąd K-bus)/4, maks. 500 mA |
| Budżet prądowy K-bus | 1750 mA na wszystkie podłączone terminale KL |
| Diagnostyka LED | `I/O RUN` (zielony — praca bezbłędna) i `I/O ERR` (czerwony — kod mrugania wskazuje pozycję wadliwego terminala na K-bus); osobne diody stanu samego łącza PROFIBUS |
| K-bus reset | konfigurowalny w TwinCAT — automatyczny (wznowienie po ustaniu przyczyny błędu) lub ręczny |
| DPV1 | umożliwia bezpośredni, acykliczny dostęp do rejestrów couplera i parametryzację złożonych terminali KL przez magistralę PROFIBUS |

## 3. Dwa poziomy komunikacji — ważne rozróżnienie diagnostyczne

BK3120 pośredniczy między dwiema odrębnymi magistralami:

- **PROFIBUS DP (na zewnątrz)** — łącze między masterem (np. Beckhoff
  FC310x) a couplerem. Błędy na tym poziomie to np. `Telegram has a
  physical Bus-Error` — dotyczą fizycznego łącza RS-485 do couplera.
- **K-bus (wewnątrz)** — łącze między samym BK3120 a podłączonymi do niego
  terminalami KL. Błędy K-bus sygnalizowane są osobno (dioda `I/O ERR`,
  kod mrugania) i mają zupełnie inne przyczyny (uszkodzony/źle wpięty
  terminal KL, przekroczony budżet prądowy K-bus).

**Błąd `physical Bus-Error` na konkretnym numerze slave'a odpowiadającym
BK3120 dotyczy więc łącza PROFIBUS do tego couplera — nie problemu z
terminalami KL podłączonymi za nim.**

## 4. Typowe przyczyny powtarzalnych, zlokalizowanych błędów PROFIBUS na BK3120

Jeśli błędy `physical Bus-Error` dotyczą **zawsze tego samego** numeru
slave'a odpowiadającego BK3120 (a nie losowych stacji), problem jest
zlokalizowany fizycznie przy tym węźle, nie na całym segmencie. Najbardziej
prawdopodobne przyczyny, w kolejności do sprawdzenia:

1. **Brak/degradacja terminacji na końcu segmentu** — BK3xx0 **nie ma
   wbudowanego przełącznika terminacji** jak niektóre inne urządzenia;
   terminację trzeba zapewnić zewnętrznie (wtyk terminujący/rezystor). Jeśli
   coupler jest ostatnim urządzeniem na trasie kabla, brakujący lub
   degradujący się terminator jest najczęstszą przyczyną.
2. **Cykl termiczny w miejscu montażu** — BK3120 często stoi bliżej
   punktów polowych (np. przy zbiorniku, studni, w terenowej skrzynce) niż
   falowniki w szafie głównej. Nagrzewanie obudowy/kabla/złącza w ciągu dnia
   (nasłonecznienie, słaba wentylacja) potrafi po kilku godzinach pracy
   ujawnić marginalny styk, który rano (na zimno) działał bez zarzutu — stąd
   wzorzec „czysto kilka godzin, potem seria błędów”.
3. **Niestabilne zasilanie 24 VDC couplera** — jeśli BK3120 dzieli linię
   zasilania z czymś, co powoduje chwilowe zapady napięcia (np. cewki
   styczników, rozruch pompy), krótki spadek napięcia może zakłócić
   transceiver RS-485 bez pełnego zresetowania urządzenia — dając pojedyncze
   błędy telegramu skorelowane w czasie z załączeniami innego sprzętu.
4. **Stan złącza D-sub / ekranu kabla przy tym konkretnym couplerze**
   (korozja, poluzowany zacisk, brak pełnego obwodu 360° dla ekranu).
5. **Marginalna długość/jakość kabla przy autodetekcji baudrate** — BK3120
   sam wykrywa prędkość transmisji; przy segmencie na granicy dopuszczalnej
   długości dla danego baudrate nawet niewielka zmiana parametrów kabla
   (temperatura, wilgoć) może okazjonalnie przekraczać margines.

### Test różnicujący

Jeśli to możliwe: **zamień fizyczne miejsce w łańcuchu magistrali** między
tym BK3120 a sąsiednim, zdrowym węzłem (przełóż kabel), albo **podmień sam
coupler** na sprawny.

- Błąd **zostaje w tym samym miejscu fizycznym** → problem w kablu/złączu/
  terminacji w tym punkcie instalacji.
- Błąd **wędruje razem z urządzeniem** → problem w samym BK3120 (np.
  degradacja transceivera RS-485 przy nagrzaniu) lub jego zasilaniu.

### Diagnostyka przez DPV1

BK3120 wspiera odczyt rejestrów diagnostycznych przez PROFIBUS DPV1 —
przez TwinCAT lub KS2000 można odczytać wewnętrzne liczniki/status couplera,
co pozwala potwierdzić charakter błędu bez fizycznego dostępu do szafy.

## Źródła

- [Beckhoff BK3120 | PROFIBUS Economy plus Bus Coupler (strona produktowa)](https://www.beckhoff.com/en-us/products/i-o/bus-terminals/bkxxxx-bus-coupler/bk3120.html)
- [Beckhoff Information System — BK3120](https://infosys.beckhoff.com/content/1033/tc3_io_intro/1718633227.html)
- [Beckhoff Information System — K-bus status (BK3xx0)](https://infosys.beckhoff.com/content/1033/bk3xx0/3028173963.html)
- [Beckhoff Information System — K-bus interruption (BK3xx0)](https://infosys.beckhoff.com/content/1033/bk3xx0/3028183691.html)
- [Dokumentacja BK3xx0 Bus Coupler for PROFIBUS-DP (PDF, mirror)](https://assets.euautomation.com/uploads/parts/datasheet/02/bk3120.pdf)
- [Dokumentacja BK3120 (PDF, mirror)](https://files.kempstoncontrols.com/files/c9460643cc43c4bc8faac953f1f094ea/BK3120.pdf)
- [Beckhoff BK3120 — dokumentacja (ManualsLib)](https://www.manualslib.com/manual/3857537/Beckhoff-Bk3120.html)
- [PROFIBUS Fault Finding Procedures, IDXT0102 Rev. 2.1 (ogólne zasady diagnostyki fizycznej warstwy PROFIBUS)](https://www.idx.co.za/wp-content/uploads/IDXT0102-PROFIBUS-Fault-Finding-Procedures-v2.1.pdf)

> Uwaga: jak w pozostałych dokumentach tego katalogu, treść oparto na
> fragmentach z wyszukiwarki (WebFetch był zablokowany w środowisku, w
> którym powstała ta notatka) i ogólnej wiedzy inżynierskiej o PROFIBUS.
> **Dokładne wartości (rozmieszczenie pinów, domyślne ustawienia, kody
> mrugania LED) należy zweryfikować w oryginalnej instrukcji Beckhoff
> BK3xx0 przed podjęciem działań serwisowych.**
