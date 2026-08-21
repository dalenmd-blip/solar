# KEB COMBIVERT F5 — przegląd techniczny

## 1. Rodzina produktów

COMBIVERT F5 to modułowa rodzina przemienników częstotliwości / serwonapędów
niemieckiego producenta KEB Automation, obejmująca warianty:

- **F5-Basic / F5-Compact / F5-General** — wersje podstawowe/uniwersalne,
- **F5-A / F5-E / F5-H** — warianty o rosnącej funkcjonalności (A — Advanced/
  standard, H — high performance),
- **F5-MULTI** — wielosekcyjne/modularne aplikacje,
- **F5-R / F5-LUB** — dedykowane napędy dźwigowe (elevator drive),
- **F5-M** — napędy modularne dużej mocy.

Zakres mocy całej rodziny: **od ok. 0,37 kW do 900 kW**, w klasach napięcia
zasilania **230 / 480 / 690 VAC**. Wszystkie jednostki są uniwersalne dla
sterowania silnikami asynchronicznymi i synchronicznymi, w pętli otwartej
(open-loop) i zamkniętej (closed-loop, z enkoderem sprzężenia zwrotnego,
np. TTL).

## 2. Tryby sterowania silnikiem

- **V/f (skalarne)** — najprostszy tryb, stosunek napięcie/częstotliwość,
  typowy dla prostych aplikacji pompowo-wentylatorowych.
- **A.S.C.L. (Asynchronous Sensorless Closed-Loop)** — flagowy tryb KEB dla
  silników asynchronicznych bez czujnika prędkości, zapewniający wysoką
  dokładność regulacji prędkości i momentu zbliżoną do sterowania z
  enkoderem.
- **Wektorowe sterowanie bezczujnikowe (sensorless vector)**.
- **Sterowanie wektorowe w pętli zamkniętej** (z enkoderem, np. sprzężenie
  TTL) — dla aplikacji wymagających precyzyjnego pozycjonowania/dużego
  momentu przy niskich obrotach.
- Obsługa silników synchronicznych (PMSM) — istotne przy nowoczesnych
  napędach o wysokiej sprawności.

## 3. Budowa i terminale sterujące

- **X2A** — listwa zaciskowa sterująca: wejścia analogowe (np. AN1± —
  zadana wartość 0…±10 VDC, rozdzielczość 11-bit, czas próbkowania ~2 ms),
  wyjście analogowe ANOUT1 (odzwierciedlające np. częstotliwość wyjściową,
  0…±10 VDC / maks. 5 mA), wejścia cyfrowe (zasilane wewnętrznie lub
  zewnętrznie, 13…30 VDC), wyjścia przekaźnikowe (obciążalność maks.
  30 VDC / 1 A).
- **X2B** — terminal bezpieczeństwa STO (Safe Torque Off).
- **X6B** — port diagnostyczny/serwisowy do podłączenia oprogramowania
  narzędziowego **Combivis** (parametryzacja, monitoring, oscyloskop
  parametrów w czasie rzeczywistym).
- **Przełączniki węzłowe (node switches, np. X1/X16)** — służą m.in. do
  ustawiania adresu urządzenia w sieci szeregowej/fieldbus w postaci
  szesnastkowej.
- **Sloty na moduły rozszerzeń fieldbus** — falownik przyjmuje wymienne
  moduły komunikacyjne (tzw. „Operator”), np. PROFIBUS-DP Operator,
  PROFINET Operator, Modbus RTU (COMBICOM), CANopen itd. — patrz
  [`integration-beckhoff-keb-f5-profibus.md`](./integration-beckhoff-keb-f5-profibus.md).

## 4. Struktura parametrów

Parametry F5 są pogrupowane wg prefiksów literowych, m.in.:

- **`Sy.xx` — parametry systemowe**, m.in. adres falownika na magistrali
  (`Sy.06`), prędkość transmisji magistrali zewnętrznej (`Sy.07`) i
  wewnętrznej (`Sy.11`).
- **`CP1` / `CP2` / `CP3` — zestawy parametrów klienta (Customer
  Parameter sets)** — pozwalają zapisać/przełączać kompletne konfiguracje
  parametrów (np. różne profile pracy pompy zależnie od pory dnia/
  nasłonecznienia).
- Dodatkowe grupy dla parametrów operacyjnych (rampy, granice prądu/
  momentu), parametrów silnika (dane znamionowe, autotuning) oraz
  parametrów procesowych dla wymiany danych po magistrali (PZD).

Dokładna numeracja i zakres parametrów różni się między wariantami F5-A/E/H/
MULTI — należy zawsze weryfikować w instrukcji właściwej dla konkretnego
wariantu i wersji oprogramowania.

## 5. Narzędzia uruchomieniowe

- **Combivis** (PC) — podłączenie przez port diagnostyczny (RS-232/USB w
  zależności od wersji) do pełnej parametryzacji, diagnostyki i archiwizacji
  nastaw.
- Wewnętrzny protokół komunikacji między płytą sterującą falownika a
  modułami fieldbus (Operator) nosi nazwę **HSP5** — jest to szeregowy
  protokół własny KEB, na który mapowane są dane z zewnętrznej magistrali
  (PROFIBUS, PROFINET, Modbus itd.).

## Źródła

- [KEB COMBIVERT F5 — Modular Drives 0.37…900 kW (karta katalogowa, PDF)](https://chastotnik.ru/files/KEB-Combivert-F5-eng.pdf)
- [KEB COMBIVERT F5 — Modular Drives (karta katalogowa, PDF, mirror)](https://adegis.com/media/asset/ed9078a792d80615b65f70f63be7c3aedc7dc0f79721ce7ba981285e53b50af6.pdf)
- [KEB Automation — COMBIVERT F5 Drive Controller (strona produktowa)](https://www.keb-automation.com/products/drive-technology/proven-technology/drive-controller-combivert-f5)
- [KEB America — F5 Inverter (strona produktowa)](https://www.kebamerica.com/products/f5-inverter/)
- [KEB COMBIVERT F5-A,-E,-H instrukcja obsługi wer. 4.0 (PDF)](https://www.keb.su/doc/Instrukciya%20F5-A_E_H%20ver.4.0%20(ang.).pdf)
- [KEB COMBIVERT F5 — Instruction Manual (ManualsLib)](https://www.manualslib.com/manual/1279226/Keb-Combivert-F5.html)
- [KEB COMBIVERT F5-A — Applications Manual, str. 505 „Bus Parameter” (ManualsLib)](https://www.manualslib.com/manual/1808036/Keb-Combivert-F5-A.html?page=505)
- [KEB COMBIVERT F5 — Instructions For Use, str. 15 „Diagnostic Interface X6B / Node Switch” (ManualsLib)](https://www.manualslib.com/manual/1426197/Keb-Combivert-F5.html?page=15)

> Uwaga: bezpośredni dostęp (WebFetch) do powyższych domen był zablokowany w
> środowisku przygotowującym tę notatkę — treść opracowano na podstawie
> fragmentów zwróconych przez wyszukiwarkę. **Dane liczbowe (piny, zakresy
> napięć, dokładne oznaczenia parametrów) należy zweryfikować w oryginalnym
> PDF przed użyciem w projekcie.**
