# Owca PL — Braki i Decyzje do Podjęcia

> Wersja: 2.0 | Data: 2026-05-01  
> Aktualizacja: po analizie 95 screenshotów Figma + systemu Festada (CZ)

---

## STATUS: Co mamy, czego brakuje

### ✅ Rozwiązane w tej iteracji (było: "brak w Figmie")

| # | Poprzednia luka | Status |
|---|-----------------|--------|
| L1 | Dashboard brak w Figmie | ✅ Mamy: `0101_ekran_glowny.png` + `0101_quick_actions.png` |
| L2 | Lista zwierząt brak w Figmie | ✅ Mamy: 3 screenshoty (lista, karty, podsumowanie) |
| L3 | Wizard porodu niekompletny | ✅ Mamy: pełny wizard 4 kroki + komponenty |
| L4 | Typy grup krycia niejasne | ✅ Mamy: 4 osobne wizard flows (r/a/i/t), łącznie 20 screenshotów |
| L5 | Komponenty porodu (status jagnięcia, przebieg) | ✅ Mamy: osobne screenshoty komponentów |
| L6 | Widok statystyk rozrodu | ✅ Mamy: 4 screenshoty (kalendarz, lista grup, picker roku, karta sezonu) |

### ❌ Nadal brakuje w Figmie

| # | Brakujący moduł/ekran | Priorytet | Notatka |
|---|-----------------------|-----------|---------|
| B1 | Moduł 08 Konto / Ustawienia | Must Have | Dane hodowcy, IRZ, pula numerów PL |
| B2 | Ekran zmiany statusu zwierzęcia | Should Have | Żywy → padł / sprzedany / ubity / zaginiony |
| B3 | Historia wag / wykres wag | Should Have | Per zwierzę, widok chronologiczny |
| B4 | Ekran/flow importu z IRZ | Could Have | Może być w module Konto |
| B5 | Ekran skanowania świadectwa PZO (OCR) | Could Have | Import ze Związku |
| B6 | Formularz dodania ważenia | Should Have | Typ zdarzenia Ważenie — osobny formularz? |
| B7 | Powiadomienia push (karencje, terminy) | Could Have | UX tych powiadomień nie zaprojektowany |
| B8 | Onboarding / ekran powitalny | Must Have (pre-launch) | Pierwszy raz w aplikacji |

---

## OTWARTE DECYZJE TECHNICZNE

### D1 — Algorytm inbredu: implementacja

**Pytanie:** Jak głęboko liczyć drzewo pedigree przy obliczaniu współczynnika Wrighta?

**Kontekst:** Festada (CZ) importuje gotową macierz inbredu z pliku Excel — nie liczy na bieżąco. Nasz design zakłada obliczanie inline w aplikacji (offline-first).

**Opcje:**
- A) Max 4 pokolenia wstecz (wystarczy dla ~99% praktycznych przypadków)
- B) Pełna głębokość (rekurencja po drzewie do liści)
- C) Cache na poziomie pary tryk–owca (oblicz raz, zapisz w `breeding_group_ewes.inbreeding_coefficient`)

**Rekomendacja:** Opcja A+C — 4 pokolenia + cache per para. Wystarczające dla hodowcy, optymalne na urządzeniu.

**Decyzja: ⬜ do podjęcia**

---

### D2 — Pula numerów PL: skąd pochodzi zakres

**Pytanie:** Czy hodowca ręcznie wpisuje zakres numerów PL, czy integracja z IRZ/ARiMR go dostarcza?

**Kontekst:** Każdy hodowca ma przydzielony zakres numerów przez ARiMR. W teorii można go pobrać przez API IRZ, ale API może być niedostępne offline.

**Opcje:**
- A) Hodowca wpisuje zakres ręcznie (np. "PL10 00278 6000 0" – "PL10 00278 6999 9")
- B) Import z ARiMR API (wymaga internetu przy pierwszej konfiguracji)
- C) Hybryd: import raz przy instalacji, potem offline

**Rekomendacja:** C — import raz, cache lokalnie. Przy braku API: fallback do opcji A.

**Decyzja: ⬜ do podjęcia**

---

### D3 — Generowanie PDF certyfikatów: gdzie i jak

**Pytanie:** PDF generowany na urządzeniu czy przez serwer?

**Opcje:**
- A) Na urządzeniu: `react-native-html-to-pdf` (HTML template → PDF)
- B) Na urządzeniu: biblioteka JS (jsPDF, pdfmake)
- C) Na serwerze (Supabase Edge Function lub dedykowany serwis) — wymaga internetu

**Kontekst:** Certyfikat PZO ma ściśle określony format graficzny. Na urządzeniu trudno zachować pixel-perfect układ.

**Rekomendacja:** C dla pierwszej wersji (wymaga netu, ale robi dokładny PDF). A jako fallback dla offline.

**Decyzja: ⬜ do podjęcia**

---

### D4 — Integracja z IRZ/ARiMR: API czy plik

**Pytanie:** Czy aplikacja komunikuje się bezpośrednio z API ARiMR, czy import/eksport przez plik (CSV/Excel)?

**Kontekst:** API ARiMR istnieje, ale wymaga rejestracji jako podmiot zewnętrzny. Alternatywnie: hodowca eksportuje dane ręcznie z portalu IRZ i importuje plik.

**Opcje:**
- A) API ARiMR (automatyczna sync)
- B) Import pliku CSV/Excel (manual, ale niezależny od API)
- C) Oba — API gdy dostępne, plik jako fallback

**Rekomendacja:** B najpierw (szybsza implementacja, nie wymaga umowy z ARiMR). A w późniejszej wersji.

**Decyzja: ⬜ do podjęcia**

---

### D5 — Formularz ważenia: osobny czy część leczenia

**Pytanie:** Czy ważenie (zdarzenie typu "weighing") ma własny uproszczony formularz, czy zawsze przechodzi przez pełny formularz leczenia?

**Kontekst:** Na screenshotach formularz leczenia ma domyślnie zaznaczoną opcję "Ważenie" gdy jest pusty, co sugeruje że ważenie jest podzbiorem leczenia. Ale ważenie nie wymaga pól: lek, seria, dawka, karencje, weterynarz.

**Opcje:**
- A) Jeden formularz — pola dla leków/karencji są ukryte gdy typ = Ważenie
- B) Dwa osobne formularze — uproszczony dla ważenia, pełny dla leczenia/szczepienia

**Rekomendacja:** A — jeden formularz z dynamicznym ukrywaniem pól. Prostszy UX, jeden ekran do utrzymania.

**Decyzja: ⬜ do podjęcia**

---

### D6 — Wizard porodu: krok "zakończ bez zgłaszania" vs "zgłoś do IRZ"

**Pytanie:** Czy zgłoszenie do IRZ jest opcją inline (podczas zamykania porodu) czy osobnym krokiem po fakcie?

**Kontekst:** Screenshot `06013_podsumowanie_zakoncz.png` pokazuje menu z opcjami "Zakończ" i "Zakończ bez zgłaszania" — sugeruje że wybór jest w kroku 3, przed podsumowaniem.

**Decyzja:** ✅ Potwierdzone przez Figmę — wybór opcji IRZ w kroku 3, podsumowanie (krok 4) zawsze następuje przed zapisem.

---

### D7 — Krycie towarowe: inbred nie jest liczony

**Pytanie:** Czy przy kryciu towarowym (jeden tryk + wszystkie owce) w ogóle pokazujemy jakiekolwiek ostrzeżenia o inbredzie?

**Kontekst:** Krycie towarowe z definicji nie jest hodowlane — hodowca świadomie nie przejmuje się doborem genetycznym.

**Rekomendacja:** Nie liczyć inbredu dla krycia towarowego. Brak ostrzeżeń.

**Decyzja: ⬜ potwierdzić z hodowcą**

---

### D8 — Status jagnięcia "Z wadą": co to znaczy dla systemu

**Pytanie:** Jagnię "Z wadą" — czy jest traktowane jak żywe (ma numer PL, jest w stadzie) czy jak martwe (nie wchodzi do stada)?

**Kontekst:** Figma pokazuje 3 statusy: Żywe / Martwe / Z wadą. Z wadą to prawdopodobnie jagnię żywe, ale z defektem genetycznym/fizycznym — może żyć, ale nie powinno trafić do hodowli.

**Rekomendacja:** Z wadą = żywe w systemie (ma numer PL, rekord w `animals`), ale z flagą `defect = TRUE`. Wykluczone z planowania rozrodu.

**Decyzja: ⬜ do potwierdzenia**

---

## WNIOSKI Z ANALIZY SYSTEMU FESTADA (CZ)

Poniżej lista elementów zidentyfikowanych w czeskim systemie, które warto uwzględnić lub świadomie odrzucić.

### Uwzględnione (już w designie)
- `breeding_line` — linia hodowlana (OSOBNA od rasy)
- `mating_date` — data krycia per jagnię (osobna od daty porodu)
- `in_heat` — ruja podczas krycia (bool)
- `successfully_mated` — wynik krycia (bool)
- `muscle_100d`, `fat_100d`, `weight_100d_kg` — wydajność mięsna 100 dni
- `chip_number` — numer chipa NFC (osobny od kolczyka)
- `cph_date`, `cph_percent` — Kontrola Produkcji Hodowlanej

### Odrzucone świadomie (lepsza architektura)
- **28 flat kolumn pedigree** (Festada: `father_name`, `father_father_name`, …) → zastąpione self-referencing FK w `animals`
- **5 flat kolumn potomka** (Festada: `potomek1` … `potomek5`) → zastąpione tabelą `birth_lambs`
- **Import macierzy inbredu z Excel** → zastąpione algorytmem Wrighta in-app
- **Tylko desktop** → mobile-first React Native

### Do przemyślenia (Festada ma, my nie)
- Moduł żywienia / paszy (Festada ma podstawowy) — **Won't Have v1**
- Baza preparatów weterynaryjnych (Festada ma słownik leków) — **Could Have v2**
- Masowe ważenie stada (ważenie wielu zwierząt jedną sesją) — **Should Have v2**

---

## DECYZJE UX (z Figmy, potwierdzone)

| # | Decyzja | Uzasadnienie |
|---|---------|--------------|
| U1 | Rodowód: expandable inline (nie osobny ekran) | Mniej skoków nawigacyjnych, szybszy przegląd |
| U2 | Lista zwierząt: podsumowanie stada na górze (3 liczniki) | Hodowca widzi od razu ile ma tryków/owiec/jagniąt |
| U3 | Formularz leczenia: typ Ważenie domyślnie zaznaczony | Ważenie jest najczęstszą operacją w formularzu |
| U4 | Wizard porodu: 4 kroki zamiast 6 | Widoczne w Figmie: Matka → Poród → Jagnięta → Podsumowanie |
| U5 | Karencje: tabela (nie lista kart) | Dużo danych numerycznych, tabela czytelniejsza |
| U6 | Krycie towarowe: osobny uproszczony wizard (4 kroki vs 8) | Różna złożoność procesu |
| U7 | Statystyki rozrodu + Kalendarz: jeden moduł (0304) | Naturalnie powiązane — oba dają widok czasowy |

---

## NASTĘPNE KROKI

1. **B1 priorytet** — zaprojektować moduł 08 Konto (dane hodowcy, pula numerów PL)
2. **B2** — ekran zmiany statusu zwierzęcia (prosty modal/sheet)
3. **B3** — ekran historii wag (chart + lista)
4. **D1–D5** — zebrać decyzje techniczne z dev i/lub hodowcą beta

---

*Plik zaktualizowany po analizie 95 screenshotów Figma i systemu Festada (CZ).*
