# Owca PL — Roadmapa Projektu

> Wersja: 2.1 | Data: 2026-05-04  
> Stack: React Native + Expo | WatermelonDB | Supabase | NFC  
> Aktualizacja: po ukończeniu setup środowiska dev (Fazy 1–4)

---

## Aktualny stan (2026-05-04)

| Faza | Status |
|------|--------|
| Dokumentacja i design | ✅ gotowa |
| Setup kont (GitHub, Supabase, Expo) | ✅ gotowy |
| CLI + MCPs (Supabase, GitHub, Context7) | ✅ skonfigurowane |
| **M0 — Kod** | 🔜 startujemy |
| M1–M9 | ⏳ zaplanowane |

> Kolejność startowa: schemat SQL (`tenants` → `animals` → ...) przez `/architekt`, potem `/builder` + `supabase db push`.

---

## Filozofia

- **Offline-first zawsze** — każda funkcja działa bez netu
- **Mobile-first** — iOS/Android primary, web secondary
- **Incremental** — każdy milestone to działający, testowalny produkt
- **Hodowca w centrum** — weryfikacja z prawdziwym użytkownikiem po każdym milestone

---

## MILESTONE 0 — Fundament (2–3 tygodnie)

**Cel:** Szkielet techniczny, nic nie jest widoczne dla użytkownika, ale wszystko działa pod spodem.

### Zadania

**Prereqs (✅ gotowe)**
- [x] GitHub repo `owca-app` (private)
- [x] Supabase projekt (Frankfurt, eu-central-1), RLS włączone
- [x] Expo konto + Expo Go na iPhone
- [x] Node.js v25.9.0, Expo CLI, EAS CLI, Supabase CLI
- [x] Supabase MCP + GitHub MCP w `~/.claude.json`

**Infrastruktura (🔜 do zrobienia)**
- [ ] Inicjalizacja projektu Expo (TypeScript, React Native)
- [ ] Migracje SQL wg `database_schema.md` + `supabase db push`
- [ ] Row Level Security (RLS) — policies na `tenant_id`
- [ ] WatermelonDB: lokalna baza SQLite, wszystkie modele JS
- [ ] Sync WatermelonDB ↔ Supabase (pull/push adapter)
- [ ] Podstawowa nawigacja (bottom tabs, stack navigator)
- [ ] Design system: kolory, typografia, podstawowe komponenty (Button, Input, Card)

**Decyzje przed startem**
- [ ] Potwierdź dostępność NFC na iOS (expo-nfc lub natywny moduł)
- [ ] Wybierz bibliotekę PDF (react-native-html-to-pdf vs serwer)
- [ ] Potwierdź API IRZ lub zaplanuj import manualny

**Deliverable:** Pusta aplikacja uruchamia się na iPhone i Android, sync działa, tabele w Supabase gotowe.

---

## MILESTONE 1 — Rejestr zwierząt (3–4 tygodnie)

**Cel:** Hodowca może dodać owcę/tryka, zobaczyć kartę z certyfikatem i rodowód.

### Zadania

**Ekrany**
- [ ] Lista zwierząt (filtry: płeć, status, sortowanie)
- [ ] Karta zwierzęcia — Certyfikat (tryk)
- [ ] Karta zwierzęcia — Certyfikat (owca)
- [ ] Karta zwierzęcia — Rodowód (expandable, 3 pokolenia)
- [ ] Formularz dodawania zwierzęcia
- [ ] Zmiana statusu (żywy → padł / sprzedany / ubity)
- [ ] Skanowanie NFC → otwarcie karty

**Logika**
- [ ] Walidacja numeru PL (format)
- [ ] Self-referencing pedigree (ojciec/matka po numerze PL)
- [ ] Historia wag (wykres + tabela)

**Deliverable:** Demo dla hodowcy. Można dodać całe stado, przejrzeć karty, zobaczyć rodowód.

---

## MILESTONE 2 — Porody i wagi (2–3 tygodnie)

**Cel:** Rejestracja porodów, nadawanie numerów jagniętom, ważenia stada.

### Zadania

**Ekrany**
- [ ] Wizard "Dodaj poród" (4 kroki: Matka → Poród → Jagnięta → Podsumowanie)
  - Krok 1: wybór matki (numer PL lub NFC)
  - Krok 2: data + przebieg (Naturalny/Z asystą/Ciężki) + liczba jagniąt (1–6)
  - Krok 3: dane per jagnię (numer PL + status Żywe/Martwe/Z wadą + waga)
  - Krok 4: podsumowanie + opcja zgłoszenia do IRZ
- [ ] Historia wag w karcie zwierzęcia

**Logika**
- [ ] Jagnię staje się pełnoprawnym zwierzęciem po porodzie
- [ ] Jagnię "Z wadą" — flaga `has_defect`, wykluczone z rozrodu
- [ ] Aktualizacja wskaźnika plenności matki
- [ ] Pula numerów PL (zarządzanie zakresem z modułu Konto)
- [ ] Blokada zamknięcia porodu bez numeru PL dla każdego jagnięcia

**Deliverable:** Hodowca może zarejestrować wszystkie porody sezonu i nadać numery jagniętom.

---

## MILESTONE 3 — Leczenie i karencje (2 tygodnie)

**Cel:** Rejestr weterynaryjny z automatycznym śledzeniem karencji mięso/mleko.

### Zadania

**Ekrany**
- [ ] Lista zdarzeń weterynaryjnych (leczenie / szczepienie / ważenie)
- [ ] Szczegóły zdarzenia (z karencjami)
- [ ] Formularz "Dodaj leczenie" (z datą, preparatem, karencją, zdjęciem)
- [ ] Ekran "Karencje w stadzie" (tabela z sortowaniem)

**Logika**
- [ ] Automatyczne obliczanie `meat_clear_at` i `milk_clear_at`
- [ ] Wyświetlanie pozostałych dni karencji
- [ ] Powiadomienia push (opcjonalne) o zbliżającym się końcu karencji

**Deliverable:** Weterynarz lub hodowca może zarejestrować każde leczenie i natychmiast widzieć karencje dla całego stada.

---

## MILESTONE 4 — Rozród i inbred (3–4 tygodnie)

**Cel:** Planowanie stanówki z obliczaniem inbredu i śledzeniem wyników.

### Zadania

**Ekrany** (wszystkie zaprojektowane w Figmie ✅)
- [ ] Lista grup krycia + szczegóły grupy (0303)
- [ ] Wizard tworzenia grupy — Ręcznie (8 kroków) (0302r)
- [ ] Wizard tworzenia grupy — Automatycznie (2 kroki) (0302a)
- [ ] Wizard tworzenia grupy — Inseminacja (6 kroków) (0302i)
- [ ] Wizard tworzenia grupy — Krycie towarowe (4 kroki) (0302t)
- [ ] Planowanie stanówki (daty, status, zakończenie) (0303)
- [ ] Statystyki rozrodu per sezon + picker roku (0304)
- [ ] Kalendarz miesięczny (0304)

**Logika**
- [ ] Algorytm inbredu Wrighta (JS, na urządzeniu)
- [ ] Automatyczne dobieranie par (min. inbred, max. par per tryk)
- [ ] Obliczanie planowanego terminu porodów (stanówka + 147 dni)
- [ ] Powiązanie porodów z grupą krycia (retroaktywnie)

**Deliverable:** Hodowca może zaplanować całą stanówkę sezonu, zobaczyć pary z inbredem, śledzić wyniki.

---

## MILESTONE 5 — Dashboard i Konto (1–2 tygodnie)

**Cel:** Dopracowanie głównego ekranu aplikacji i konfiguracja konta hodowcy.

### Zadania

**Ekrany** (Dashboard zaprojektowany w Figmie ✅, Konto ❌ brak w Figmie)
- [ ] Dashboard z kafelkami nawigacji (0101)
- [ ] Dashboard Quick Actions: Dodaj zwierzę / Dodaj poród / Stwórz grupę / Dodaj leczenie (0101)
- [ ] Ekran Konto / Ustawienia (08 — do zaprojektowania)
  - Dane hodowcy i gospodarstwa (IRZ, PZO, adres)
  - Pula numerów PL (zarządzanie zakresem)
  - Ustawienia synchronizacji

**Logika**
- [ ] Licznik aktywnych karencji widoczny na kafelku Dashboardu
- [ ] Pula numerów PL: walidacja dostępności przy nadawaniu jagniętom

**Deliverable:** Hodowca konfiguruje dane gospodarstwa i ma czytelny punkt startowy aplikacji.

---

## MILESTONE 6 — Druk i certyfikaty PZO (2 tygodnie)

**Cel:** Generowanie certyfikatów i rodowodów zgodnych z formatem PZO.

### Zadania

- [ ] Szablon PDF — Certyfikat tryka (zgodny z wzorem PZO)
- [ ] Szablon PDF — Certyfikat/licencja maciorki
- [ ] Szablon PDF — Rodowód 3 pokolenia
- [ ] AirPrint / udostępnianie PDF
- [ ] Eksport do pliku (share sheet)

**Deliverable:** Hodowca drukuje certyfikat ze swojego iPhone'a na miejscu.

---

## MILESTONE 7 — Import/Sync z IRZ i Związkiem (2–3 tygodnie)

**Cel:** Automatyczny import danych z zewnętrznych rejestrów zamiast ręcznego przepisywania.

### Zadania

- [ ] Import z IRZ (API ARiMR lub plik CSV/Excel)
- [ ] Import świadectwa tryka (skan → OCR lub ręczne zatwierdzenie)
- [ ] Import zbiorczego zestawienia maciorek (skan)
- [ ] Sync IRZ — zgłaszanie porodów/zmian statusu do rejestru
- [ ] Deduplicacja przy imporcie (match po numerze PL)

**Deliverable:** Hodowca importuje całe stado ze Związku jednym skanem, bez ręcznego wpisywania.

---

## MILESTONE 7.5 — Brakujące ekrany (do zaprojektowania w Figmie)

**Cel:** Uzupełnienie designu o ekrany które nie były w pierwotnym zakresie Figmy.

### Do zaprojektowania
- [ ] Moduł 08 Konto / Ustawienia (dane hodowcy, pula numerów PL, IRZ)
- [ ] Ekran zmiany statusu zwierzęcia (modal/sheet)
- [ ] Historia i wykres wag per zwierzę
- [ ] Onboarding / ekran powitalny (pierwszy raz w aplikacji)
- [ ] Ekran skanowania świadectwa PZO (import OCR)

---

## MILESTONE 8 — Beta i testy z hodowcami (ciągły, od M3)

**Cel:** Weryfikacja produktu z prawdziwymi użytkownikami.

- [ ] Rekrutacja 3–5 hodowców testerów
- [ ] TestFlight (iOS) + wewnętrzny track (Android)
- [ ] Zbieranie feedback po każdym milestone
- [ ] Iteracje UX na podstawie realnego użycia

---

## MILESTONE 9 — Launch (po M7)

- [ ] App Store submission (iOS)
- [ ] Google Play submission (Android)
- [ ] Onboarding flow dla nowych użytkowników
- [ ] Cennik / model subskrypcji (SaaS)
- [ ] Landing page

---

## Kolejność priorytetów (MoSCoW)

### Must Have (MVP)
- Rejestr zwierząt z pedigree (M1)
- Dodawanie porodów + numery PL (M2)
- Leczenie + karencje (M3)
- Offline + NFC (M0+M1)

### Should Have
- Planowanie rozrodu + inbred (M4)
- Kalendarz + Dashboard (M5)
- Import IRZ (M7)

### Could Have
- Druk certyfikatów (M6)
- Import ze Związku (M7)
- Statystyki rozrodu (M4)

### Won't Have (v1)
- E-commerce owiec
- Porady żywieniowe
- Baza preparatów weterynaryjnych
- Web app pełna (web = read-only podgląd)

---

## Szacowany czas do MVP

| Milestone | Tygodnie | Kumulatywnie |
|-----------|----------|-------------|
| M0 — Fundament | 2–3 | 3 |
| M1 — Rejestr zwierząt | 3–4 | 7 |
| M2 — Porody i wagi | 2–3 | 10 |
| M3 — Leczenie i karencje | 2 | 12 |
| **MVP gotowy do beta** | | **~12 tygodni** |
| M4 — Rozród | 3–4 | 16 |
| M5 — Kalendarz | 1–2 | 18 |
| M6 — Druk | 2 | 20 |
| M7 — Import | 2–3 | 23 |
| **Launch** | | **~23 tygodnie (~6 miesięcy)** |

> Zakłada 1 developera full-stack. Z 2 deweloperami: skróć o ~30%.

---

## Zależności krytyczne (musi być przed)

```
M0 (infrastruktura)
  └── M1 (zwierzęta)
        ├── M2 (porody) — wymaga listy zwierząt
        ├── M4 (rozród) — wymaga pedigree z M1
        └── M3 (leczenie) — wymaga listy zwierząt
              └── M5 (dashboard) — wymaga M3 dla karencji
M6 (PDF) — może być równolegle z M4/M5
M7 (import) — może być po M1
```

---

*Plik wygenerowany na podstawie UX brief, ekranów Figma i analizy technicznej.*
