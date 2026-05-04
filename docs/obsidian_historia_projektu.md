# Stodoła — historia projektu

tags: #owca #stodola #projekt #historia
created: 2026-05-02

---

## Czym jest Stodoła

Aplikacja mobilna do zarządzania stadem owiec dla polskich hodowców. Nazwa robocza **Stodoła**, cel — zastąpienie papierowych ksiąg hodowlanych i arkuszy Excel. Współpraca z oficjalnym rejestrem Polskiego Związku Owczarskiego (IRZ) i Związkiem Hodowców.

Projekt zaczął się od rozmowy o tym, że hodowcy owiec w Polsce nie mają żadnego sensownego narzędzia mobilnego — tylko przestarzałe systemy webowe albo papier.

---

## Stack technologiczny

| Warstwa | Technologia | Dlaczego |
|---|---|---|
| Frontend | React Native + Expo | iOS + Android z jednego kodu |
| Lokalna baza | WatermelonDB (SQLite) | Offline-first — działa w polu bez zasięgu |
| Backend/chmura | Supabase (PostgreSQL) | Open-source, auth, storage, realtime sync |
| Sync | WatermelonDB pull/push | Automatyczny sync gdy pojawi się sieć |
| NFC | expo-nfc | Odczyt elektronicznych kolczyków owiec |

Architektura jest **offline-first** — aplikacja musi działać w polu bez internetu. Sync do chmury odbywa się automatycznie po wejściu w zasięg.

---

## Co zostało zrobione (faza projektowania)

### Maj 2026 — sprint dokumentacyjny

1. **Eksport Figmy** — pobrano ~95 screenshotów z projektu Figma przez Figma MCP
2. **Mapa ekranów** (`screen_map.md`) — pełna mapa nawigacyjna wszystkich modułów
3. **Specyfikacja funkcjonalna** (`functional_spec.md`) — opis każdego modułu
4. **Schemat bazy danych** (`database_schema.xlsx`) — 14 zakładek: 10 tabel, ERD, WatermelonDB schema, legenda
5. **Cykl życia danych** (`data_lifecycle.md`) — zasada "ro": każda dana jest immutable po zapisie, zdarzenia to INSERT never UPDATE
6. **Lista braków** (`gaps_and_decisions.md`) — co wymaga decyzji przed kodowaniem
7. **Roadmapa** (`roadmap.md`) — milestony od MVP do v1.0
8. **HTML canvas** — interaktywna mapa 96 screenshotów (podejście zbiorczy canvas + podejście z menu i podstronami sekcji)
9. **Konwencja nazewnictwa PNG** — `SSFF_NN_opis.png` (SS=sekcja, FF=podsekcja, NN=numer)
10. **SCREENS_INDEX.md** — katalog wszystkich ekranów z opisem zmian

### Kluczowe odkrycie — dwa odrębne flow dodawania zwierzęcia

**0204 — Dodaj zwierzę** (z Dashboardu): ma pole Płeć (Owca/Tryk), CTA "Zapisz zwierzę"
**0305 — Dodaj tryka** (z modułu Rozród): bez pola Płeć, ma Genotyp + Masy (4 wartości), CTA "Zapisz tryka do rozrodu"

To dwa osobne ekrany, osobne flow, osobne komponenty w bazie.

---

## Moduły aplikacji

| Kod | Moduł | Status Figma |
|---|---|---|
| 01 | Dashboard + Quick Actions | ✅ 2 screeny |
| 02 | Zwierzęta (lista, karta owcy, karta tryka, dodaj) | ✅ 26 screenów |
| 03 | Rozród (grupy, stanówka, statystyki, dodaj tryka) | ✅ 24 screeny |
| 04 | Leczenie | ✅ 11 screenów |
| 05 | Karencje | ✅ 4 screeny |
| 06 | Porody | ✅ 15 screenów |
| 07 | Kalendarz | ⚠️ folder pusty |
| 08 | Konto / ustawienia | ⚠️ do zaprojektowania |
| 09 | Design System | ✅ 13 screenów |

---

## Zasada danych ("ro")

Każda dana w systemie jest naturalnie **read-only po zapisie**:
- Zdarzenia → INSERT, never UPDATE
- Stany → tylko dedykowane pola stanu (np. `status: alive → sold`)
- Usunięcie → zawsze soft delete (`is_deleted = TRUE`)
- Korekcja → nowy rekord z `corrects_id` wskazującym na stary

---

## Pliki projektu

```
/Users/Tomek_Main/Documents/Claude/Projects/Owca/
├── docs/
│   ├── screen_map.md
│   ├── functional_spec.md
│   ├── data_lifecycle.md
│   ├── gaps_and_decisions.md
│   ├── roadmap.md
│   └── database_schema.xlsx
├── screens/
│   ├── canvas.html          ← zbiorczy canvas, ~96 screenshotów
│   ├── canvas_v3.html       ← menu → podstrony sekcji
│   ├── sections/            ← 11 podstron HTML
│   ├── 01_dashboard/
│   ├── 02_zwierzeta/
│   ├── 03_rozrod/
│   ├── 04_leczenie/
│   ├── 05_karencje/
│   ├── 06_porody/
│   └── 09_design_system/
├── paczka_wspolniczka/
│   └── STODOŁA - jak aplikacja przechowuje dane.html
└── SCREENS_INDEX.md
```

---

## Następne kroki (faza produkcyjna)

→ szczegóły w [obsidian_plan_dev_setup](obsidian_plan_dev_setup.md)

---

## 2026-05-04 — Ukończenie setup środowiska dev

Pełny setup środowiska (Fazy 1–4) ukończony podczas sesji z Claude Code.

**Faza 1 — Konta:**
- GitHub repo `owca-app` (private)
- Supabase projekt (Frankfurt, eu-central-1), RLS włączone, project URL + klucze zapisane
- Expo konto (`owca-app`), zalogowany przez GitHub

**Faza 2 — Aplikacje:**
- Docker Desktop, TablePlus zainstalowane
- Expo Go na iPhone

**Faza 3 — CLI:**
- Node.js v25.9.0 (bezpośrednia instalacja, bez nvm)
- Expo CLI 6.3.12, EAS CLI 18.9.1
- Supabase CLI 2.95.4
- Git skonfigurowany (Tomek / waleqq@gmail.com)

**Faza 4 — MCPs:**
- Supabase MCP (`@supabase/mcp-server-supabase`) z PAT
- GitHub MCP (`@modelcontextprotocol/server-github`) z PAT
- Oba w `~/.claude.json`

**Stan:** Cały toolstack gotowy. Następny krok: M0 — pierwsze migracje SQL przez `/architekt`.
