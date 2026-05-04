# Stodoła — plan startu dev (środowisko)

tags: #owca #stodola #dev #setup #todo
created: 2026-05-02

---

## Faza 1 — Rejestracje (zrób najpierw, bo OAuth łączy serwisy)

- [x] **GitHub** → github.com → nowe repo `owca-app` (private)
- [x] **Supabase** → supabase.com → "Start your project" → zaloguj **przez GitHub** (jedno konto = dwa serwisy)
      - zanotuj: `Project URL`, `anon key` (= publishable key), `service_role key` (= secret key)
- [x] **Expo (EAS)** → expo.dev → konto (`owca-app`) → potem `eas login` z terminala
- [ ] **Apple Developer** → developer.apple.com → 99 USD/rok → **poczekaj na to do TestFlight**

---

## Faza 2 — Aplikacje na Maca

- [x] **Homebrew** 5.1.8 ✓
- [x] **Docker Desktop** ✓
- [x] **TablePlus** ✓
- [x] **Expo Go** → iPhone ✓
- [x] VS Code rozszerzenia: ESLint, Prettier, React Native Tools, GitLens ✓

---

## Faza 3 — CLI

```bash
# Node.js — v25.9.0 już zainstalowany bezpośrednio (nvm pominięty)

# Expo + EAS
npm install -g expo-cli eas-cli

# Supabase CLI
brew install supabase/tap/supabase

# Sprawdź że masz git
brew install git
git config --global user.name "Tomek"
git config --global user.email "waleqq@gmail.com"
```

---

## Faza 4 — MCP w Cowork (żeby Claude mógł pisać do bazy bezpośrednio)

- [x] **Supabase MCP** → zainstalowany lokalnie z Personal Access Token ✓
- [x] **GitHub MCP** → zainstalowany lokalnie z Personal Access Token ✓
- [x] Filesystem/workspace — już masz (folder Owca podmontowany)

---

## Po setup — pierwsza rozmowa z Claude

Jak masz Supabase MCP podpięty, powiedz:

> "Mam Supabase MCP podpięty, zacznijmy tworzyć schemat bazy — zacznij od tabeli animals i events"

Claude napisze SQL migracje bezpośrednio do projektu Supabase.

---

## Kolejność bazy danych (co budujemy pierwsze)

1. `tenants` — multi-tenant fundament
2. `animals` — owce i tryki (core tabela)
3. `animal_events` — append-only log zdarzeń (sprzedaż, śmierć, leczenie)
4. `breeding_groups` — grupy krycia
5. `births` + `lambs` — porody i jagnięta
6. `treatments` — leczenia
7. `withholding_periods` — karencje
8. `sync_metadata` — WatermelonDB sync
