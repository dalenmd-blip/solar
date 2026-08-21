# Dokumentacja techniczna: Beckhoff + KEB COMBIVERT F5 (PROFIBUS)

Ten katalog zawiera notatki referencyjne opracowane na potrzeby integracji
sterownika Beckhoff (TwinCAT / EtherCAT) z falownikami KEB COMBIVERT F5 po
magistrali PROFIBUS DP, w kontekście instalacji fotowoltaicznej / solarnej
(np. sterowanie pompami, nadążnikami (trackerami) lub innymi napędami
zasilanymi z PV).

## Zawartość

1. [`beckhoff-profibus.md`](./beckhoff-profibus.md) — sprzęt i oprogramowanie
   Beckhoff do obsługi PROFIBUS (terminale EL6731/EL6731-0010, karty FC310x,
   konfiguracja w TwinCAT, GSD, diagnostyka).
2. [`keb-combivert-f5.md`](./keb-combivert-f5.md) — rodzina falowników KEB
   COMBIVERT F5: budowa, tryby sterowania silnikiem, struktura parametrów,
   terminale sterujące, narzędzie Combivis.
3. [`integration-beckhoff-keb-f5-profibus.md`](./integration-beckhoff-keb-f5-profibus.md) —
   przewodnik integracji: Beckhoff jako master PROFIBUS DP + KEB F5 (moduł
   PROFIBUS-DP Operator 00.F5.060-3000) jako slave — warstwa fizyczna,
   uruchomienie, model wymiany danych (PKW/PZD), słowo sterujące/statusowe,
   typowy scenariusz dla aplikacji solarnej, checklisty i troubleshooting
   (w tym studium przypadku powtarzalnego `physical Bus-Error`).
4. [`beckhoff-bk3120.md`](./beckhoff-bk3120.md) — Bus Coupler BK3120
   (Beckhoff, PROFIBUS DP ↔ K-bus): budowa, dane techniczne, adresacja,
   diagnostyka LED/DPV1 oraz przyczyny powtarzalnych, zlokalizowanych
   błędów PROFIBUS na pojedynczym couplerze.

## Uwaga metodologiczna

Środowisko, w którym powstały te notatki, ma zablokowany bezpośredni dostęp
do stron internetowych (WebFetch) — dostępne było wyłącznie wyszukiwanie
(WebSearch) zwracające fragmenty dokumentów. Fakty potwierdzone przez
wyszukiwanie (numery parametrów, adresy, nazwy protokołów, oznaczenia
modułów) są oznaczone i odsyłają do źródeł w sekcjach „Źródła”. Ogólna wiedza
o architekturze PROFIBUS/PROFIdrive i typowych wzorcach integracji
sterownik–falownik jest przedstawiona na bazie wiedzy inżynierskiej autora.

**Przed wdrożeniem produkcyjnym należy zweryfikować wszystkie liczby
(adresy pinów, domyślne adresy stacji, dokładny układ bitów słowa
sterującego) bezpośrednio w oryginalnych instrukcjach Beckhoff i KEB**,
ponieważ różnią się one między wariantami sprzętowymi i wersjami
oprogramowania układowego.
