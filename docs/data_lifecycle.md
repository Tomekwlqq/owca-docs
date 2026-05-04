# Owca PL — Cykl Życia Danych (Data Lifecycle)

> Wersja: 1.0 | Data: 2026-05-01  
> Ten dokument definiuje reguły niezmienialności, źródła danych i automaty stanów.  
> **Deweloper powinien przeczytać to przed M0.**

---

## Zasada nadrzędna: Read-Only jako cecha każdej danej

Każda dana w systemie ma naturalną właściwość niezmienialności (ro = read-only po zapisie).  
Nie jest to ograniczenie techniczne — to odzwierciedlenie rzeczywistości hodowlanej.

> Owca urodziła się z konkretnym ojcem. Ważyła 3.2 kg przy urodzeniu. Dostała lek X w dniu Y.  
> Żadnego z tych faktów nie można "cofnąć" — można je co najwyżej uzupełnić lub skorygować nowym faktem.

**Konsekwencja dla bazy danych:**  
- Zdarzenia → zawsze `INSERT`, nigdy `UPDATE`  
- Stany → `UPDATE` tylko na dedykowanych polach stanu z datą zmiany  
- Etykiety → `UPDATE` dozwolony (imię, notatki — nie są faktami biologicznymi)  
- Usunięcie → nigdy `DELETE`, zawsze `is_deleted = TRUE` (soft delete)

---

## Kategorie danych według żywotności

### 🔵 Na zawsze — dane identyfikacyjne (Immutable)

Dane ustalone przy narodzeniu lub rejestracji. Nie zmieniają się nigdy.

| Pole | Tabela | Uwaga |
|------|--------|-------|
| `pl_number` | `animals` | Numer urzędowy ARiMR — niezmienny od urodzenia do śmierci |
| `chip_number` | `animals` | Numer chipa NFC wszywanego raz |
| `birth_date` | `animals` | Data biologiczna — fakt |
| `sex` | `animals` | Płeć biologiczna |
| `father_id`, `mother_id` | `animals` | Rodowód — self-referencing FK, ustalany przy urodzeniu |
| `breed` | `animals` | Rasa — nie zmienia się po wpisaniu do ksiąg |
| `irz_number` | `tenants` | Numer siedziby stada w ARiMR |
| `pzo_breeder_number` | `tenants` | Numer hodowcy w Polskim Związku Owczarskim |

**Reguła implementacyjna:** Te pola nie mają `updated_at`. Jeśli hodowca "poprawia" numer PL — to błąd wpisany przy rejestracji, nie zmiana faktu. Logować jako `correction_log`, nie nadpisywać.

---

### 🟡 Zmiana stanu — ze ścieżką audytu (State Machine)

Dane które się zmieniają, ale każda zmiana jest nieodwracalna i wymaga śladu.

#### Automat stanów zwierzęcia

```
         ┌─────────────────────────────────────────┐
         │              alive (active)              │
         └──┬──────────┬─────────────┬─────────────┘
            │          │             │             │
            ▼          ▼             ▼             ▼
          sold    slaughtered       dead          lost
        (sprzed.)  (ubój)         (padł)       (zaginął)
```

- Przejście jest **nieodwracalne** — brak powrotu do `alive`
- Każda zmiana stanu wymaga: `status_changed_at` (data) + `status_change_reason` (powód)
- Zwierzę w stanie `sold/dead/slaughtered` **pozostaje w bazie** (nie jest kasowane)  
  → Potrzebne w rodowodach potomków, historii leczenia, statystykach sezonu
- Stan `external_archived` — specjalny dla tryków zewnętrznych po zakończeniu stanówki

#### Stany grupy krycia

```
planned → active → closed
```

- `planned` → `active`: po dacie startu stanówki
- `active` → `closed`: ręczne zamknięcie przez hodowcę (lub data końcowa minęła)
- Po `closed`: skład grupy jest immutable — nie można dodać/usunąć owcy

#### Stany porodu (birth)

```
in_progress → closed
```

- `in_progress`: wizard otwarty, można edytować
- `closed`: po zamknięciu — immutable. Korekta = nowe zdarzenie z adnotacją

---

### 🟢 Serie pomiarów — append-only

Dane które przyrastają w czasie. Każdy nowy pomiar to nowy rekord — nigdy nadpisanie poprzedniego.

| Typ | Tabela | Klucz serii |
|-----|--------|-------------|
| Wagi | `animal_weights` | `animal_id` + `weighed_at` |
| Wydajność mięsna 100d | `animal_performance` | `animal_id` + `measured_at` |
| Plenność maciorki | obliczana z `births` + `birth_lambs` | nie przechowywana flat |

**Reguła:** Jeśli hodowca wpisał złą wagę — nie edytuje rekordu. Dodaje nowy z adnotacją `is_correction = TRUE` i `corrects_id = <id błędnego>`. Błędny rekord dostaje flagę `is_superseded = TRUE`.

---

### 🟣 Zdarzenia — append-only, zamknięte

Fakty które się wydarzyły. Po zamknięciu niezmienne — bo mogły już trafić do zewnętrznych rejestrów (IRZ, PZO).

| Zdarzenie | Tabela | Zamknięcie |
|-----------|--------|-----------|
| Poród | `births` + `birth_lambs` | Po kliknięciu "Zakończ" — status `closed` |
| Leczenie / szczepienie | `treatments` | Po zapisie formularza |
| Krycie (przypisanie owcy do grupy) | `breeding_group_ewes` | Po zamknięciu grupy |

**Reguła:** Błąd w leczeniu → nowe zdarzenie z `is_correction = TRUE`, stare zostaje.  
Dlaczego: karencje mogły już zostać przekazane weterynarzowi lub do IRZ.

---

### 🔴 Import zewnętrzny — jednorazowy + delta sync

Dane pochodzące z instytucji zewnętrznych. Nie są "własnością" aplikacji — są kopią stanu zewnętrznego rejestru.

| Źródło | Co dostarcza | Tabela |
|--------|-------------|--------|
| ARiMR (IRZ) | pula numerów PL, potwierdzenia zgłoszeń | `ear_tag_numbers`, `irz_sync_log` |
| PZO / Związek | świadectwo wpisu tryka, zestawienie maciorek | `pzo_certificates` |
| Skan PDF / OCR | świadectwo w formie papierowej | `pzo_certificates` (pending review) |

**Reguła:** Każdy import logowany w `irz_sync_log` z datą, typem i statusem. Przy konflikcie (dane z IRZ różnią się od lokalnych) — wymagana ręczna weryfikacja hodowcy, nie automatyczne nadpisanie.

---

## Źródła danych — kto co tworzy

| Źródło | Tworzy |
|--------|--------|
| 📱 Hodowca (app) | animals, births, breeding_groups, status changes, weights |
| 🩺 Weterynarz / hodowca | treatments, vaccinations, weighing events |
| 🏛️ PZO / Związek | pzo_certificates (import OCR lub CSV) |
| 🏦 ARiMR / IRZ | ear_tag_numbers (pula PL), irz_sync_log (potwierdzenia) |
| ⚙️ System (auto) | meat_clear_at, milk_clear_at, inbreeding_coefficient, planned_birth_date, calendar_events |

---

## Reguły integralności — checklista dla dewelopera

### Przed zapisem zdarzenia
- [ ] Czy `animal_id` istnieje i ma status `alive`?
- [ ] Czy `tenant_id` zgadza się z zalogowanym użytkownikiem?
- [ ] Czy numer PL nie jest już zajęty w tej puli?

### Przy zmianie stanu zwierzęcia
- [ ] Czy podano `status_changed_at`?
- [ ] Czy podano `status_change_reason`?
- [ ] Czy zwierzę nie jest już w stanie końcowym (dead/sold/slaughtered)?

### Przy zamykaniu porodu
- [ ] Czy każde żywe jagnię ma numer PL?
- [ ] Czy `birth_lambs` ma przynajmniej jeden rekord?
- [ ] Czy status porodu to `in_progress` (nie można zamknąć już zamkniętego)?

### Przy planowaniu grupy krycia
- [ ] Czy tryk ma status `alive`?
- [ ] Czy każda owca w grupie ma status `alive`?
- [ ] Czy owca nie jest już przypisana do innej aktywnej grupy w tym sezonie?

### Przy dodawaniu leczenia
- [ ] Czy `meat_withdrawal_days` i `milk_withdrawal_days` są podane przy lekach z karencją?
- [ ] Automatycznie oblicz `meat_clear_at = treated_at + meat_withdrawal_days`
- [ ] Automatycznie oblicz `milk_clear_at = treated_at + milk_withdrawal_days`

### Soft delete — zawsze
- [ ] Nigdy nie używaj `DELETE` na rekordach zdarzeń, zwierząt, porodów, leczenia
- [ ] Zawsze `is_deleted = TRUE` + `last_modified_at = now()`
- [ ] Wyjątek dozwolony: draft/temp rekordy przed pierwszym save

---

## Dane obliczane przez system (nie przechowywane flat)

Poniższe wartości są **zawsze obliczane on-the-fly** lub **cachowane z invalidacją** — nigdy nie są ręcznie wpisywane:

| Wartość | Obliczana z | Gdzie wyświetlana |
|---------|------------|-------------------|
| Współczynnik inbredu (Wright) | `animals.father_id/mother_id` rekurencyjnie | karta grupy krycia, karta pary |
| Planowana data porodu | `breeding_group.mating_date + 147 dni` | karta grupy, kalendarz |
| Plenność maciorki | COUNT z `births` + `birth_lambs` | karta zwierzęcia, statystyki |
| Aktywne karencje | `treatments` gdzie `meat_clear_at > NOW()` | dashboard, tabela karencji |
| Wiek zwierzęcia | `NOW() - animals.birth_date` | karta zwierzęcia |

---

*Data lifecycle v1.0 — dokument żywy, aktualizowany przy zmianach schematu.*
