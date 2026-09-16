# loudnearby.github.io — instrukcje dla Claude Code

Statyczna strona z zapowiedziami koncertów metal/rock/punk/goth/tribute w Polsce.
Cała logika i dane są w `index.html`: dwie tablice JS (`TRI` = Trójmiasto, `REST` = reszta
Polski), renderowane po stronie klienta.

## Zasady treści — `zasady.txt`

Format wpisu, dozwolone gatunki, obsługa Trójmiasta, festiwali, tras wielomiastowych,
statusu ANULOWANE itd. są opisane w `zasady.txt` w rootcie repo.
**Zawsze przeczytaj go w całości przed edycją danych koncertowych** — poniższe reguły
dotyczą wyłącznie *procesu* researchu i aktualizacji, nie duplikują zasad treściowych.

## Środowisko: ograniczenia sieciowe

- **WebFetch jest zablokowany dla każdej domeny** w standardowym środowisku Claude Code
  używanym do tego repo (potwierdzone dla wszystkich sprawdzanych źródeł, łącznie z
  example.com/wikipedia.org jako kontrolą). Nie trać czasu i tokenów na próby WebFetch —
  od razu używaj WebSearch.
- WebSearch ma limit zapytań na sesję/agenta (ok. 200) — planuj zapytania oszczędnie,
  patrz sekcja „Efektywność” niżej.
- Jeśli WebFetch kiedyś zostanie odblokowany dla konkretnych domen, źródła oznaczone
  ✅ niżej dadzą dużo lepsze pokrycie niż przez WebSearch (paginacja, pełne kalendarze).

## Źródła — priorytet i status

| Źródło | Status | Uwagi |
|---|---|---|
| koncertyw.pl | ✅ Wysoka wydajność | Dobrze zindeksowane strony pojedynczych wydarzeń, `site:koncertyw.pl <fraza>` działa świetnie |
| trojmiasto.pl/imprezy | ✅ Wysoka wydajność | Kategorie `ciezkie-brzmienia` i `rock-punk` to dobry filtr gatunkowy |
| rockmetal.pl | ✅ Wysoka wydajność | Dobre dla tras klubowych i mniej mainstreamowych zespołów |
| stodola.pl, progresja.com, klubkwadrat.pl, klubzascianek.pl, b90.pl, drizzlygrizzly.pl, cka2.pl, alive.wroclaw.pl | 🟡 Średnia | Strony niedostępne przez WebFetch, ale WebSearch po nazwie klubu + miesiącu/roku zwraca wyniki |
| wytwornia.pl | 🟡 Średnia | Warto równolegle szukać „Atlas Arena Łódź” — duże koncerty w Łodzi częściej trafiają tam |
| **mozg.art.pl** | ❌ **Pomiń domyślnie** | Zero trafień w dwóch niezależnych próbach. Sprawdzać najwyżej raz na kilka miesięcy albo na wyraźną prośbę użytkownika |
| **naszemiasto.pl/kalendarz-imprez** | ❌ **Pomiń domyślnie** | Ogólny portal, nic ponad to, co dają źródła ✅ |
| **kalendarzkoncertowy.pl** | ❌ **Pomiń domyślnie** | Jak wyżej |
| klubproxima.pl (Proxima, Warszawa) | 🟡 Słabe bezpośrednio | Szukaj przez rockmetal.pl / koncertyw.pl zamiast wprost po domenie |

Aktualizuj tę tabelę, jeśli kolejna sesja natrafi na inny wynik (np. źródło z ❌ nagle
da trafienia, albo ✅ przestanie działać) — to jest żywy dokument, nie ustalona raz reguła.

„Pomiń domyślnie” to instrukcja dla Claude, nie techniczna blokada sieciowa — środowisko
i tak nie pozwala na WebFetch, więc to działa wyłącznie jako filtr „nie trać na to budżetu
WebSearch”.

## Efektywność (tokeny / liczba zapytań)

1. **Pojedyncze źródło → szukaj bezpośrednio w głównej rozmowie**, nie odpalaj subagenta.
   Subagent ma sens tylko przy naprawdę równoległym przeszukiwaniu 4+ niezależnych źródeł
   naraz.
2. Zapytania **miesiąc po miesiącu / temat po temacie**, nie „sprawdź cały rok” jedną
   szeroką frazą — precyzyjne zapytania dają więcej konkretów przy tej samej liczbie
   wywołań.
3. Gdy zlecasz research subagentowi: każ mu zwracać **tylko dane (blok kodu) + 1–2 zdania
   podsumowania**, bez długich list wykluczeń — to oszczędza kontekst przy odczycie
   raportu w głównej rozmowie.
4. Nie weryfikuj ponownie pozycji już potwierdzonej w więcej niż jednym źródle w tej samej
   sesji.

## Workflow aktualizacji — DOPISUJ, nie podmieniaj całości

Tabela zawiera realne, zweryfikowane dane (od 2026-09-16). **Pełna podmiana `TRI`/`REST`
nie powinna się już powtarzać** — była uzasadniona jednorazowo, żeby zastąpić dane
przykładowe sprzed researchu.

Przy kolejnych aktualizacjach:

1. Szukaj tego, co **nowe od ostatniego sprawdzenia** (punkt odniesienia: `stan na
   DD.MM.RRRR` w nagłówku strony), nie całego roku od nowa.
2. **Dopisuj** nowo znalezione, potwierdzone koncerty do `TRI`/`REST` we właściwym miejscu
   chronologicznym.
3. **Nie usuwaj ręcznie** przeszłych wydarzeń — kod sam je filtruje
   (`daysUntil(iso) < 0`, zasada 15 w `zasady.txt`).
4. Koncert odwołany → dodaj `"CANCELLED"` jako 5. element tablicy, **nie usuwaj wiersza**
   (zasada 6).
5. Sprzeczna/błędna data w istniejącym wpisie → popraw **tylko ten jeden wiersz**, nie
   przepisuj całej tabeli.
6. **Zaktualizuj `stan na DD.MM.RRRR` w nagłówku (`index.html`, `id="ui-asof-prefix"` obok
   `<strong>`) na dzisiejszą datę.** To jest obowiązkowe przy KAŻDEJ aktualizacji danych
   koncertowych, nawet drobnej (np. jeden dopisany wiersz) — nie tylko przy większych
   przebiegach researchu. Traktuj to jako część samej zmiany danych, nie osobny,
   opcjonalny krok.
7. Zawsze waliduj po zmianach (patrz niżej).

## Walidacja przed commitem

- Format każdego wpisu: `[data ISO, nazwa, "Miejsce, Miasto", GATUNEK]` (+ opcjonalnie
  `"CANCELLED"` jako 5. element)
- Gatunek z zamkniętej listy: `METAL, ROCK, PUNK, GOTH, TRIBUTE`
- Brak duplikatów (data + nazwa + miejsce)
- Zgodność miasto → tablica: `TRI` tylko dla Gdańsk/Gdynia/Sopot, `REST` dla reszty
- Daty w oknie czasowym `dziś … dziś+365 dni` (zasada 1)
- **`stan na DD.MM.RRRR` w nagłówku zgadza się z dzisiejszą datą** — jeśli dane koncertowe
  zmieniły się w tym commicie, a ta data nie została zaktualizowana, commit jest
  niekompletny
- `node --check` na wyciągniętym inline `<script>` (dwa bloki `<script>` w pliku — sprawdź
  każdy osobno)
- Renderowanie lokalne (`python3 -m http.server` + Playwright) i porównanie licznika
  koncertów na stronie z liczbą wierszy w danych
