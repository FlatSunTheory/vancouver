# Vancouver 2026 — mapa i planer wyjazdu

Narzędzie do zaplanowania dwutygodniowego wyjazdu do Vancouver na przełomie
października i listopada 2026. Jeden plik HTML, bez backendu, bez kluczy API.

## Co potrafi

- **Mapa** 190 punktów w Vancouver i okolicach — Halloween, natura, miasto, jedzenie,
  muzea, widoki, spacery z przewodnikiem i szlaki piesze.
- **Filtry** — kategoria, pod dachem, bez auta, sezonowe, bezpieczne bezglutenowo,
  ze szlakiem, dzielnica, priorytet, wyszukiwarka.
- **Dwa głosy** — dwie osoby oceniają niezależnie (must / nice / skip), narzędzie
  pokazuje zgodne „oboje must" i punkty sporne.
- **Plan dzień po dniu (24.10–12.11, 20 dni)** — kalendarz godzinowy jako panel na mapie.
  Liczba dni wynika z dat start/koniec w stopce panelu. Punkty wstawia się
  kliknięciem lub przeciągnięciem, bloki rozciąga za krawędzie, w pustą godzinę
  można wpisać własną notatkę. Punkty wybrane na dany dzień świecą na mapie gwiazdkami.
- **Światło dzienne** — siatka dnia jest zacieniowana (noc / zmierzch / dzień), linia
  zachodu, długość dnia w nagłówku. Każde miejsce ma w danych pole `swiatlo`
  (`dzien` / `zachod` / `zmrok`); blok, który wychodzi poza swoje światło, dostaje
  ostrzeżenie. Szlak ma się kończyć ≥30 min przed zachodem. Przy wstawianiu punktu
  zielony pasek pokazuje okno, w którym trzeba zacząć. Zmiana czasu 1.11 uwzględniona.
- **Szlaki** — 14 tras z dystansem i przewyższeniem. Przycisk „Ślady OSM" wczytuje
  prawdziwy przebieg z OpenStreetMap.
- **Eksport / import JSON** — kopia zapasowa i wymiana ocen między dwiema osobami.

## Uruchomienie

Otwórz `index.html` w przeglądarce. To wszystko — Leaflet jest wbudowany w plik,
z sieci pobierają się tylko kafelki mapy.

Wersja opublikowana przez GitHub Pages działa lepiej niż plik z dysku: kafelki
ładują się pewniej, a przycisk „Ślady OSM" nie ma problemów z zasadami CORS.

## GitHub Pages

Settings → Pages → Source: `Deploy from a branch` → Branch: `main`, katalog `/ (root)`.
Po chwili strona jest pod `https://UZYTKOWNIK.github.io/NAZWA-REPO/`.

## Dane

`dane/vancouver-miejsca.json` to jedyne źródło prawdy o miejscach. Kod mapy jest
wymienny, dane nie. Schemat pinezki i ustalenia merytoryczne opisuje `dane/brief.md`.
Dane są też wbudowane w `index.html` (`<script id="dane">`) — po zmianie JSON-a trzeba
je tam podmienić, inaczej strona pokaże starą listę.

Od wersji 7 danych (22.10.2026, przebiegi researchu A i B) pinezki mogą mieć
dodatkowe pola: `opis`, `dojazd_komunikacja`, `zrodla`, `pewnosc`, `zrodlo_danych`,
a trasy także `start`, `cel`, `typ_trasy`, `punkty`, `trudnosc`, `czas_h`,
`nawierzchnia`, `szlaki_po_drodze`, `warunki_x_xi`, `zwrot_awaryjny`, `alltrails`.
Popup pokazuje opis, dojazd bez auta, trudność, warunki X–XI i ostrzeżenie przy
`pewnosc: "niska"`.

Uwaga: oceny i plan dnia **nie trafiają do repozytorium** — żyją w pamięci
przeglądarki każdej osoby osobno. Publiczne repo ujawnia listę miejsc i kod,
nie ujawnia tego, co zaplanowaliście.

## Licencja

Dane o miejscach i kod — do własnego użytku.
Podkład mapy: © OpenStreetMap contributors (ODbL), kafelki: Esri (domyślnie), OpenTopoMap, OSM — przełącznik w prawym górnym rogu mapy. Biblioteka Leaflet: BSD-2-Clause.
