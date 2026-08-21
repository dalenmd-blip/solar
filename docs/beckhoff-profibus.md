# Beckhoff a PROFIBUS

## 1. Miejsce PROFIBUS w ekosystemie Beckhoff

Beckhoff jest przede wszystkim producentem systemów opartych o EtherCAT,
jednak oferuje pełne wsparcie dla integracji PROFIBUS DP jako:

- **magistrali nadrzędnej z perspektywy sterownika** — Beckhoff PLC (CX/CP
  embedded PC lub PC przemysłowy z TwinCAT) pełni rolę mastera klasy 1 wobec
  urządzeń trzecich (falowniki, I/O, czujniki) podłączonych do PROFIBUS DP,
- **mostu EtherCAT ↔ PROFIBUS** — terminal/karta PROFIBUS jest węzłem w sieci
  EtherCAT (proces wymiany danych z urządzeniami PROFIBUS jest „tunelowany”
  przez EtherCAT do reszty systemu I/O),
- opcjonalnie jako **slave PROFIBUS** — gdy sterownik Beckhoff ma być
  podrzędnym urządzeniem w istniejącej instalacji z masterem innego
  producenta (np. Siemens S7).

## 2. Kluczowe produkty sprzętowe

| Produkt | Typ | Rola | Uwagi |
|---|---|---|---|
| **EL6731** | Terminal EtherCAT (IP20) | Master **lub** Slave PROFIBUS (wybór programowy) | Obsługuje PROFIBUS DP, DPV1, DPV2 oraz protokół PROFIdrive-PKW; szybkość transmisji do 12 Mbit/s (DP) |
| **EL6731-0010** | Terminal EtherCAT (IP20) | Wyłącznie Slave PROFIBUS | Wariant „tylko slave” tego samego terminala |
| **FC310x** | Karta PCI/PCIe | Master PROFIBUS dla PC-based Control | Dodatkowo obsługuje S5-FDL (AG-AG) — starszy protokół Siemens |
| **CX/CP z modułem PROFIBUS** | Embedded PC / Panel PC | Master lub slave | Ten sam rdzeń funkcjonalny co EL6731, zintegrowany w obudowie kontrolera |

Wszystkie te urządzenia korzystają z tego samego stosu protokołu PROFIBUS
DP i są konfigurowane w TwinCAT w analogiczny sposób.

## 3. EL6731 / EL6731-0010 — najważniejsze cechy

- **Wybór roli master/slave** ustawiany programowo (w konfiguracji TwinCAT),
  nie przełącznikiem sprzętowym (dotyczy EL6731; EL6731-0010 to wariant
  wyłącznie slave).
- **Obsługiwane protokoły jako master:** PROFIBUS DP, PROFIBUS DPV1,
  PROFIBUS DPV2, PROFIdrive-PKW (protokół parametryzacji napędów), a na
  kartach FC310x dodatkowo S5-FDL (AG-AG).
- **Prędkość transmisji:** do 12 Mbit/s w trybie DP, do 1,5 Mbit/s w trybie
  MC (Multi-Computing/Lightbus-kompatybilnym — starszy tryb).
- **Tryb izochroniczny** o wysokiej precyzji — do sterowania osiami
  (istotne przy napędach wymagających synchronizacji, mniej istotne dla
  prostego sterowania prędkością pompy/silnika).
- **Domyślny adres stacji mastera:** 1 (ustawiany w konfiguracji, nie
  przełącznikiem).
- **Diagnostyka:** terminal posiada diody LED sygnalizujące stan łącza
  PROFIBUS (m.in. run/error magistrali) — dokładne znaczenie diod zależy od
  wariantu (EL6731 vs EL6731-0010) i jest opisane na dedykowanych stronach
  dokumentacji „LED description” dla każdego wariantu.
- Terminal wpina się w standardową szynę EtherCAT (K-bus/E-bus) obok innych
  terminali I/O — z punktu widzenia magistrali EtherCAT jest to zwykły
  slave EtherCAT, wewnątrz którego działa niezależny stos PROFIBUS.

## 4. Konfiguracja w TwinCAT (System Manager / TwinCAT XAE)

Typowy przebieg konfiguracji mastera PROFIBUS na bazie EL6731 (analogicznie
dla FC310x, z tą różnicą że FC310x dodaje się jako karta PCI, a nie terminal
EtherCAT):

1. **Dodanie urządzenia EtherCAT** — prawy klik na „I/O Devices” → „Add New
   Item” → wybór adaptera sieciowego EtherCAT, skan urządzeń podłączonych do
   magistrali (w tym EL6731).
2. **Rozpoznanie EL6731 jako „PROFIBUS Master/Slave EL6731, EtherCAT”** —
   po skanie terminal pojawia się jako osobne urządzenie I/O z własną
   zakładką konfiguracji PROFIBUS.
3. **Ustawienie parametrów łącza PROFIBUS** — na karcie urządzenia (np.
   zakładka „FC310x”/„EL6731”) ustawia się prędkość transmisji (domyślnie
   12 Mbit/s) oraz inne parametry warstwy fizycznej.
4. **Dodanie slave'ów PROFIBUS** — prawy klik na master → „Add New Item”:
   - urządzenia, których plik GSD jest już w katalogu
     `TwinCAT\IO\GSD` (lub `TwinCAT\3.1\Config\Io\Profibus`), pojawiają się
     automatycznie na liście, pogrupowane wg producenta;
   - urządzenia nierozpoznane dodaje się jako **„General PROFIBUS Box
     (GSD)”** (sekcja „Miscellaneous”) i wskazuje ręcznie plik `.gsd`/`.gse`
     w oknie wyboru pliku.
5. **Adres stacji slave'a** — ustawiany w konfiguracji TwinCAT musi być
   zgodny z adresem fizycznie ustawionym na urządzeniu (przełącznikami/
   parametrem serwisowym — patrz dokumentacja urządzenia, np. KEB F5).
6. **Mapowanie obrazu procesu** — dane cykliczne (PZD) slave'a są mapowane
   automatycznie na zmienne I/O widoczne w programie PLC (struktury IN/OUT);
   dane acykliczne (parametryzacja, DPV1/PKW) obsługuje się przez osobne
   mechanizmy (ADS/funkcje acykliczne), a nie przez zwykły obraz procesu.
7. **Aktywacja konfiguracji** i przejście w tryb run — TwinCAT automatycznie
   wykonuje sekwencję PROFIBUS: parametryzacja → konfiguracja → wymiana
   danych cyklicznych (Data Exchange).

### Ważne: pliki GSD urządzeń trzecich

Beckhoff **nie dostarcza** plików GSD dla urządzeń innych producentów (np.
falowników KEB). Plik GSD dla konkretnego modelu/wersji oprogramowania
falownika trzeba pobrać od producenta (KEB) i ręcznie skopiować do katalogu
GSD instalacji TwinCAT przed dodaniem urządzenia w konfiguracji.

## 5. Warstwa fizyczna PROFIBUS DP (przypomnienie ogólne)

- Medium: ekranowana skrętka (RS-485), złącza D-sub 9-pin (typowo styk 3 =
  B-line (RxD/TxD-P), styk 8 = A-line (RxD/TxD-N), ekran na obudowie).
- Topologia liniowa (magistrala), z **terminatorami (rezystorami
  końcowymi)** aktywnymi na obu skrajnych urządzeniach segmentu — ich brak
  lub nadmiar to najczęstsza przyczyna błędów transmisji.
- Adresy stacji: 0–125 (0 i adresy zarezerwowane wg konfiguracji mastera nie
  powinny być używane przez slave'y), każdy adres musi być unikalny na
  segmencie.
- Maksymalna długość segmentu zależy od prędkości transmisji — im wyższa
  prędkość (do 12 Mbit/s), tym krótszy dopuszczalny odcinek kabla bez
  repeatera (przy 12 Mbit/s rzędu 100 m, przy niższych prędkościach do
  kilku km).

## Źródła

- [Beckhoff EL6731 / EL6731-0010 — dokumentacja (PDF, mirror)](https://filfar.by/PDF/beckhoff/el6731en.pdf)
- [Beckhoff EL6731 — dokumentacja (ManualsLib)](https://www.manualslib.com/manual/1801471/Beckhoff-El6731.html)
- [Beckhoff Information System — PROFIBUS DP (fc310x)](https://infosys.beckhoff.com/content/1033/fc310x/4381165835.html)
- [Beckhoff Information System — Profibus (tc3_io_intro)](https://infosys.beckhoff.com/content/1033/tc3_io_intro/1718629387.html)
- [Beckhoff Information System — Adding GSD and EDS Boxes](https://infosys.beckhoff.com/content/1033/tcsystemmanager/1087231243.html)
- [Beckhoff Information System — Configuring a fieldbus master (EL6731)](https://infosys.beckhoff.com/content/1033/el6731/2385319435.html)
- [Beckhoff Information System — EL6731 LED description](https://infosys.beckhoff.com/content/1033/el6731/2381577483.html)
- [Beckhoff Information System — EL6731-0010 PROFIBUS slave terminal](https://infosys.beckhoff.com/content/1033/el6731/2381589643.html)
- [Beckhoff EL6731 — dokumentacja (Kempston Controls mirror, PDF)](https://files.kempstoncontrols.com/files/0d67a32511c11ab4f1663947eae3aa55/EL6731.pdf)

> Uwaga: strona `download.beckhoff.com` oraz `infosys.beckhoff.com` były
> zablokowane przez politykę sieciową środowiska, w którym przygotowano tę
> notatkę — powyższe linki pochodzą z wyników wyszukiwania i **należy
> zweryfikować ich aktualną treść bezpośrednio**, najlepiej poprzez
> oficjalny Beckhoff Information System (infosys.beckhoff.com).
