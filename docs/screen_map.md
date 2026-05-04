# Owca PL — Mapa Ekranów i Nawigacji

> Wersja: 2.0 | Data: 2026-05-01  
> Źródło: 95 screenshotów z Figmy  
> Nazwy plików: `screens/{moduł}/{ekran}/{plik}.png`

---

## Struktura nawigacji głównej

Aplikacja używa **bottom tab bar** z 5 zakładkami:

```
┌─────────────────────────────────────────────────────┐
│                   STODOŁA                           │
├──────┬──────────┬──────────┬──────────┬─────────────┤
│ Home │ Zwierzęta│  Rozród  │ Leczenie │  Więcej...  │
└──────┴──────────┴──────────┴──────────┴─────────────┘
```

Zakładka „Więcej" daje dostęp do: Karencje, Kalendarz, Statystyki, Konto.

---

## 01 — Dashboard (Ekran główny)

**Nawigacja modułów** (`0101_ekran_glowny.png`):
- Lista zwierząt, Leczenie, Statystyki, Karencje (z licznikiem aktywnych)

**Quick Actions** (`0101_quick_actions.png`):
- Dodaj zwierzę, Dodaj poród, Stwórz grupę, Dodaj leczenie

```
Dashboard
├── → Lista zwierząt (0201)
├── → Leczenie / Lista (0401)
├── → Karencje w stadzie (0501)
├── → Statystyki rozrodu (0304)
├── Quick: → Dodaj zwierzę (0204)
├── Quick: → Dodaj poród (0601)
├── Quick: → Stwórz grupę krycia (0302)
└── Quick: → Dodaj leczenie (0401 formularz)
```

---

## 02 — Zwierzęta

### 0201 Lista zwierząt

**Funkcje widoczne na ekranie:**
- Wyszukiwanie po numerze / nazwie
- Filtry (płeć, status), sortowanie
- Podsumowanie stada (3 liczniki: Tryki / Owce / Urodzone)
- Karta zwierzęcia: numer PL, płeć, rasa, waga, ikony statusu

```
Lista zwierząt (0201)
├── Tap na kartę → Karta tryka (0202) lub Karta owcy (0203)
├── FAB "+" → Dodaj zwierzę (0204)
└── NFC scan → Karta zwierzęcia (0202/0203)
```

---

### 0202 Karta tryka

**Certyfikat tryka** zawiera:
- Numer PL, numer kolczyka, rasa, linia hodowlana
- Data urodzenia, typ urodzenia
- Masa 10/30/56 dni + aktualna
- Genotyp PPG i POD-IN, numer księgi, data licencji
- Hodowca, właściciel

**Rodowód** — drzewo pedigree:
- 2 pokolenia domyślnie (ojciec + matka)
- Expandable: tap na przodka → rozwinięcie do dziadków

```
Karta tryka (0202)
├── Toggle → Certyfikat / Rodowód
├── Rodowód: tap na przodka → rozwinięcie (inline)
└── ← Wstecz → Lista zwierząt (0201)
```

---

### 0203 Karta owcy (maciorki)

**Różnice vs. karta tryka:**
- Dodatkowa sekcja: **Plenność** (historia miotów, wskaźnik)
- Brak: data licencji

---

### 0204 Dodawanie zwierzęcia — z Dashboardu

> ⚠️ To jest flow **ogólny** — ma pole **Płeć** (Owca / Tryk).  
> Różni się od dodawania tryka (0305) gdzie płeć jest zakładana z góry.

**Pola formularza świadectwa:**
- Numer identyfikacyjny (PL), System identyfikacyjny: Kolczyk / Chip
- **Płeć: Owca / Tryk** ← tylko tutaj
- Rasa, Data urodzenia, Typ urodzenia / odchowu
- Genotyp, Księga, Indeks, Masy 10/30/56 dni

**Formularz rodowodu** (krok 2):
- Ojciec i matka: numer PL lub NFC
- Dziadkowie (opcjonalne)

```
Dashboard → Quick Action "Dodaj zwierzę"
└── 0204: Formularz świadectwa (z Płcią)
    └── Krok 2: Formularz rodowodu
        └── Zapisz → Lista zwierząt (0201)
```

---

## 03 — Rozród

### 0305 Dodawanie tryka — z modułu Rozród

> ⚠️ **Brak pola Płeć** — tryk wiadomy z kontekstu.  
> Przycisk końcowy: "Zapisz tryka do rozrodu" (nie "Zapisz zwierzę").

```
Moduł Rozród → Lista tryków / Tworzenie grupy
└── 0305: Formularz świadectwa tryka (bez Płci)
    └── Krok 2: Formularz rodowodu
        └── "Zapisz tryka do rozrodu" → powrót do Rozrodu
```

---

### 0302 Tworzenie grupy krycia

Cztery typy grup, każdy z osobnym wizard flow:

| Typ | Dobór par | Tryk | Inbred | Kroki |
|-----|-----------|------|--------|-------|
| Ręcznie (`r`) | Hodowca | Własny | Tak, per para | 8 |
| Automatycznie (`a`) | System (min. inbred) | Własny | Tak, auto | 2 |
| Inseminacja (`i`) | Hodowca | Zewnętrzny dawca | Tak | 6 |
| Towarowe (`t`) | Wszystkie owce 1 tryk | Własny | Nie | 4 |

```
Stwórz grupę krycia (0302)
├── Wybór typu → właściwy wizard (r/a/i/t)
├── Każdy wizard: wielokrokowy formularz
└── Zapisz → Planowanie stanówki (0303)
```

---

### 0303 Planowanie stanówki

**Funkcje:**
- Lista wszystkich grup krycia (bieżący sezon)
- Status grupy: Planowana / W trakcie / Zakończona
- Licznik „W trakcie: X dni"
- Planowany termin porodów (+147 dni od daty stanówki)
- Przycisk „Zakończ stanówkę"

```
Lista grup krycia (0303)
├── Tap na grupę → Szczegóły grupy
├── FAB "+" → Stwórz grupę (0302)
└── Szczegóły: Zakończ stanówkę → aktualizuje status
```

---

### 0304 Statystyki rozrodu + Kalendarz

**Kalendarz:**
- Widok miesięczny, nawigacja prev/next
- Dni z wydarzeniami podświetlone
- Tap na dzień → lista zdarzeń

**Statystyki rozrodu:**
- Dropdown picker roku (historia od 2020)
- Karta sezonu: łączne urodzenia / żywe / martwe (z podziałem ♂/♀)
- Lista grup z wynikami per sezon

---

## 04 — Leczenie

### 0401 Lista i formularz

**Lista zdarzeń:**
- Typy zdarzeń: 💊 Leczenie | 💉 Szczepienie | ⚖️ Ważenie
- Karta zdarzenia: numer zwierzęcia, typ, data, lek, karencja pozostała

**Formularz dodania zdarzenia:**
- Numer zwierzęcia (wpisz lub NFC)
- Data zdarzenia, Typ (toggle)
- Opis / powód, Weterynarz
- Preparat: nazwa + seria + dawka
- Karencja mięso (godziny) + mleko (dni)
- Zdjęcia dokumentacyjne

```
Lista leczenia (0401)
├── Tap na wpis → Szczegóły wpisu
├── FAB "+" → Formularz dodania
└── Formularz: Zapisz → Lista
```

---

## 05 — Karencje

### 0501 Tabela karencji w stadzie

**Funkcje:**
- Tabela wszystkich zwierząt z **aktywnymi** karencjami
- Kolumny: Numer zwierzęcia | Mięso (dni) | Mleko (dni)
- Sortowanie malejąco (najdłuższa karencja na górze)
- Wyszukiwanie po numerze PL lub NFC scan

```
Karencje (0501)
├── Tap na zwierzę → Karta zwierzęcia (0202/0203)
└── ← Wstecz → Dashboard
```

---

## 06 — Porody

### 0601 Rejestracja porodu (wizard 4-krokowy)

**Krok 1 — Matka** (`06011`):
- Wpisz numer PL matki lub skanuj NFC
- Przycisk „Dalej" nieaktywny dopóki numer nie wpisany

**Krok 2 — Poród** (`06012`):
- Data porodu (datepicker)
- Przebieg: Naturalny / Z asystą / Ciężki (selector)
- Liczba jagniąt: dropdown 1–6

**Krok 3 — Jagnięta** (`06013`):
- Per jagnię: Numer PL + Status (Żywe / Martwe / Z wadą) + Masa urodzeniowa
- Menu „Zakończ": z opcją zgłoszenia do IRZ lub bez

**Krok 4 — Podsumowanie** (`06014`):
- Przegląd wszystkich danych przed zapisem
- Przyciski: Zapisz / Wróć do edycji

```
Rejestracja porodu (0601)
├── Krok 1: Matka (numer PL / NFC)
├── Krok 2: Data + Przebieg + Liczba jagniąt
├── Krok 3: Dane jagniąt per sztuka
└── Krok 4: Podsumowanie → Zapisz
```

---

## Podsumowanie modułów i plików

| Moduł | Ekranów/flows | Plików PNG | Status Figma |
|-------|--------------|------------|--------------|
| 01 Dashboard | 2 | 2 | ✅ |
| 02 Zwierzęta | 4 ekrany | 31 | ✅ |
| 03 Rozród | 4 flows + lista + statystyki | 24 | ✅ |
| 04 Leczenie | Lista + formularz | 11 | ✅ |
| 05 Karencje | Tabela | 3 | ✅ |
| 06 Porody | Wizard 4 kroki | 9 | ✅ |
| 09 Design System | Komponenty | 13 | ✅ |
| 08 Konto | — | 0 | ❌ do zaprojektowania |
| **Łącznie** | | **93+** | |

---

## Globalny diagram nawigacji

```
                    ┌─────────────┐
                    │  DASHBOARD  │ (0101)
                    └──────┬──────┘
          ┌─────────┬───────┼────────┬──────────┐
          ▼         ▼       ▼        ▼          ▼
    ┌──────────┐ ┌──────┐ ┌──────┐ ┌────────┐ ┌──────────┐
    │ Lista    │ │Rozród│ │Porody│ │Leczenie│ │Karencje  │
    │zwierząt  │ │(0303)│ │(0601)│ │ (0401) │ │  (0501)  │
    │  (0201)  │ └──┬───┘ └──┬───┘ └────────┘ └──────────┘
    └────┬─────┘    │        │
         │          │        └──→ Karta zwierzęcia (0202/0203)
    ┌────┴─────┐    │
    │ Karta    │◄───┘
    │ tryka    │
    │ (0202)   │
    └──────────┘
```

---

*Plik wygenerowany na podstawie 95 screenshotów Figma.*  
*Następny krok: zaprojektowanie modułu 08 Konto.*
