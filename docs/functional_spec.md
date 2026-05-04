# Owca PL — Specyfikacja Funkcjonalna

> Wersja: 2.0 | Data: 2026-05-01  
> Stack: React Native + Expo | WatermelonDB (offline) | Supabase (chmura) | NFC  
> Źródło: 95 screenshotów Figma + dokumenty PZO + analiza systemu Festada (CZ)

---

## Moduł 01: Dashboard

### Cel
Punkt startowy aplikacji. Szybki dostęp do kluczowych modułów i najczęstszych akcji bez konieczności nawigowania przez zakładki.

### Funkcje

**Nawigacja modułów (ekran główny)**
- Kafelki / lista linków do wszystkich modułów: Lista zwierząt, Leczenie, Statystyki, Karencje
- Licznik aktywnych karencji widoczny bezpośrednio na kafelku

**Quick Actions**
- 4 przyciski skrótów: Dodaj zwierzę, Dodaj poród, Stwórz grupę krycia, Dodaj leczenie
- Jeden tap do formularza — bez pośrednich ekranów

---

## Moduł 02: Zarządzanie zwierzętami

### Cel
Centralny rejestr całego stada. Każde zwierzę — owca, tryk, jagnię — to jeden rekord z pełnym pedigree, historią wag, leczenia i rozrodu.

### Funkcje

**Lista zwierząt**
- Wielokryterialne filtrowanie (płeć, status, rasa) i sortowanie
- Wyszukiwanie po numerze PL lub nazwie
- Podsumowanie stada na górze ekranu: liczniki Tryki / Owce / Urodzone
- Plenność widoczna tylko dla owiec (liczba jagniąt per sezon)
- Szybki podgląd kluczowych metryk na karcie listy: numer PL, rasa, waga, status

**Karta zwierzęcia — Certyfikat (tryk)**
- Numer PL, numer kolczyka, chip NFC
- Rasa, linia hodowlana
- Data urodzenia, typ urodzenia (format: 2/1, 3/2 itp.)
- Masa w 10/30/56 dniach + aktualna
- Genotyp PPG i POD-IN (wartości z certyfikatu PZO)
- Numer i typ księgi hodowlanej (G, LR itp.), data licencji
- Hodowca i właściciel

**Karta zwierzęcia — Certyfikat (owca)**
- Identyczne pola jak tryk, bez daty licencji
- Dodatkowa sekcja: **Plenność** — historia miotów (format PZO np. "6/6/6/8"), wskaźnik plenności

**Karta zwierzęcia — Rodowód**
- Wizualizacja drzewa pedigree (min. 2 pokolenia wstecz, rozwijane do 3–4)
- Expandable cards: tap na przodka → inline rozwinięcie ze szczegółami (numer PL, rasa, genotyp)
- Identyczna struktura dla tryka i owcy
- Druk rodowodu (format zgodny z PZO) — przyszłość

**Dodawanie zwierzęcia (wizard)**
- *Krok 1 — Świadectwo:* Formularz z walidacją wymaganych pól
  - Numer identyfikacyjny (PL), typ: Kolczyk / Chip
  - Płeć, Rasa, Typ urodzenia (dropdown 8 opcji)
  - Data urodzenia, Masa urodzeniowa
  - Genotyp PPG + POD-IN
  - Numer księgi, data licencji
  - Walidacja błędów inline (pole czerwone + komunikat)
- *Krok 2 — Rodowód:* Pola ojciec/matka (numer PL lub skanowanie NFC)
  - Opcjonalne dziadki

**Skanowanie NFC**
- Odczyt kolczyka elektronicznego → natychmiastowe otwarcie karty zwierzęcia
- Działa offline
- Dostępne z listy, karty, formularza leczenia, karencji

**Zmiana statusu**
- Statusy: żywy → sprzedany / padł / ubity / zaginiony
- Dostępne z karty zwierzęcia

---

## Moduł 03: Rozród i grupy krycia

### Cel
Planowanie stanówki z uwzględnieniem inbredu. System umożliwia tworzenie grup 4 typów i monitorowanie wyników rozrodu per sezon.

### Funkcje

**4 typy grup krycia:**

| Typ | Opis | Inbred | Kroki wizarda |
|-----|------|--------|--------------|
| Ręczna | Hodowca sam dobiera pary tryk–owca | Tak, per para | 8 |
| Automatyczna | System dobiera optymalnie (min. inbred) | Tak, per para | 2 |
| Inseminacja | Jak ręczna, dawca zewnętrzny | Tak, per para | 6 |
| Towarowe | Wszystkie owce z jednym trykiem | Nie | 4 |

**Obliczanie inbredu (współczynnik Wrighta)**
- Obliczany na bieżąco w aplikacji (offline)
- Wymaga min. 3–4 pokoleń pedigree w bazie
- Progi:
  - `0%` → OK (brak wspólnych przodków)
  - `> 0%` → Uwaga (kolorowanie ostrzegawcze)
  - `> ~10%` → Nieodpowiedni (blokada lub silne ostrzeżenie)

**Planowanie stanówki**
- Lista wszystkich grup krycia bieżącego sezonu
- Status grupy: Planowana / W trakcie / Zakończona
- Planowany termin porodów (data stanówki + ~147 dni)

**Statystyki rozrodu**
- Picker roku (historia od 2020)
- Karta sezonu: łączne Urodzone / Żywe / Martwe (z podziałem ♂/♀)

---

## Moduł 04: Dokumentacja weterynaryjna (Leczenie)

### Cel
Rejestr wszystkich interwencji weterynaryjnych z automatycznym śledzeniem karencji mięso/mleko.

### Funkcje

**Lista zdarzeń**
- Typy: 💊 Leczenie | 💉 Szczepienie | ⚖️ Ważenie
- Karta zdarzenia: numer zwierzęcia, typ, data, lek/preparat, pozostałe dni karencji

**Formularz dodawania zdarzenia leczenia**
- Numer zwierzęcia (wpisz lub NFC)
- Data zdarzenia
- Typ: Leczenie / Szczepienie / Ważenie
- Preparat: nazwa + seria + dawka
- Karencja mięso (godziny) + mleko (dni)
- Zdjęcia (dokumentacja)

**Automatyczne obliczanie karencji**
- `meat_clear_at = data_zdarzenia + karencja_mięso_godziny`
- `milk_clear_at = data_zdarzenia + karencja_mleko_dni`

---

## Moduł 05: Karencje w stadzie

### Cel
Globalny widok wszystkich aktywnych karencji w stadzie — narzędzie bezpieczeństwa żywności.

### Funkcje

**Tabela karencji**
- Tabela wszystkich zwierząt z aktywnymi karencjami
- Kolumny: Numer zwierzęcia | Mięso (dni) | Mleko (dni)
- Sortowanie: malejąco po najdłuższej karencji
- Wyszukiwanie po numerze PL + skanowanie NFC

---

## Moduł 06: Rejestracja porodów

### Cel
Rejestracja porodu jako zdarzenia z przypisaniem jagniąt do matki i nadaniem numerów PL.

### Funkcje

**Wizard 4-krokowy:**

**Krok 1 — Matka:**
- Wpisz numer PL matki lub skanuj NFC

**Krok 2 — Dane porodu:**
- Data porodu, przebieg (Naturalny / Z asystą / Ciężki), liczba jagniąt (1–6)

**Krok 3 — Dane jagniąt:**
- Per każde jagnię: Numer PL + Status (Żywe / Martwe / Z wadą) + Masa urodzeniowa
- Menu „Zakończ": z lub bez zgłoszenia do IRZ

**Krok 4 — Podsumowanie:**
- Przegląd wszystkich danych przed finalnym zapisem

**Walidacja:**
- Każde żywe jagnię MUSI mieć nadany numer PL przed zamknięciem formularza

---

## Moduł 07: Konto i ustawienia

> **Status: do zaprojektowania** — brak screenshotów w Figmie (moduł 08)

### Planowane funkcje
- Dane użytkownika / gospodarstwa (IRZ, PZO, adres)
- Pula numerów PL: zarządzanie zakresem przydzielonym hodowcy
- Zarządzanie synchronizacją (IRZ, Związek)
- Ustawienia aplikacji

---

## Wymagania niefunkcjonalne

| Cecha | Wymaganie |
|-------|-----------|
| Offline | Pełna funkcjonalność bez internetu |
| NFC | Skanowanie kolczyków w polu |
| Sync | Automatyczny po powrocie do sieci |
| Multi-tenant | `tenant_id` na każdej tabeli |
| Platformy | iOS + Android (React Native Expo) |
| Język | Polski (PL) |
| Nazwa w UI | **STODOŁA** |

---

## Podsumowanie zakresu MVP vs pełna wersja

### MVP (Milestony 0–3)
- Rejestr zwierząt z pedigree (M1)
- Dodawanie porodów + numery PL (M2)
- Leczenie + karencje (M3)
- Offline + NFC (M0+M1)

### Pełna wersja (Milestony 4–7)
- Planowanie rozrodu + inbred (M4)
- Kalendarz + Dashboard (M5)
- Druk certyfikatów PDF (M6)
- Import IRZ + Import ze Związku (M7)

---

*Plik wygenerowany na podstawie 95 screenshotów Figma, dokumentów PZO i analizy systemu Festada (CZ).*
