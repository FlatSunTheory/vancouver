# Vancouver 2026 — brief projektu

> Dokument bazowy dla wiedzy projektu. Aktualizować po każdym etapie.
> Ostatnia aktualizacja: 18.10.2026 (daty wyjazdu, silnik v5 — światło dzienne)

---

## 1. Cel projektu

Zbudować narzędzie do zaplanowania i kontrolowania dwutygodniowego wyjazdu do Vancouver
na przełomie października i listopada 2026.

Narzędzie ma zawierać:

- **interaktywną mapę Vancouver i okolic** z pinezkami lokalizacji (element główny),
- możliwość **łączenia pinezek w trasy** i przypisywania ich do konkretnych dni,
- **plan dzień po dniu** (20 dni: 24.10–12.11),
- **checklisty** do odhaczania — przed wyjazdem i na miejscu.

---

## 2. Ustalenia

| Pozycja | Status |
|---|---|
| Uczestnicy | Wawrzyniec + partnerka (2 osoby dorosłe) |
| Termin | **wylot z PL sob 24.10.2026 rano → wylot z Vancouver 12.11 (lot w nocy 12/13.11) → lądowanie w PL pt 13.11.** W Vancouver 20 dni kalendarzowych: dzień 1 (24.10) przylot, dzień 20 (12.11) wylot, 18 pełnych dni. Godziny lotów: DO UZUPEŁNIENIA |
| Nocleg | u przyjaciół w Vancouver — **dzielnica: DO UZUPEŁNIENIA** |
| Samochód | prawdopodobnie do pożyczenia od znajomych, niepotwierdzone |
| Wypady poza miasto | 4–5 dni z 18 pełnych |
| Budżet | **DO UZUPEŁNIENIA** (CAD/os./dzień na atrakcje i jedzenie) |
| Dieta | partnerka nie je glutenu ani laktozy — jedzenie planujemy wokół zweryfikowanych lokali |

**Zainteresowania (w kolejności wagi):**

1. Jesień i Halloween — to główny motyw wyjazdu
2. Poznanie miasta i jego klimatu
3. Jedzenie
4. Natura i widoki

---

## 3. Decyzje techniczne

> **Stan z 22.10.2026.** Aplikacja żyje w repo **github.com/FlatSunTheory/vancouver** (publiczne)
> i jest opublikowana na **https://flatsuntheory.github.io/vancouver/**. To jest wersja główna —
> działa na każdym urządzeniu, także na telefonie. Plik otwierany z dysku nie ładuje kafelków OSM
> (blokada OSM dla stron bez nagłówka Referer), dlatego w aplikacji jest przełącznik podkładu:
> OSM (domyślny na stronie), Esri ulice (domyślny z pliku), Esri satelita, OpenTopoMap.
> Oceny i plan dnia żyją w pamięci przeglądarki — między urządzeniami przenosi je Eksport/Import.

> **Zmiana z 22.08.2026.** Pierwotne założenie „Leaflet + kafelki OSM w artifacie,
> trwałość przez `window.storage`" okazało się niewykonalne. Artifact na claude.ai jest
> odcięty od zewnętrznych hostów (CSP) — nie wczyta ani Leafleta z CDN, ani kafelków
> OpenStreetMap. `window.storage` w ogóle nie istnieje w tym środowisku.

**Dwa frontendy, jedne dane:**

| | Wersja lokalna | Wersja artifact |
|---|---|---|
| Plik | `vancouver-silnik.html` | opublikowany artifact |
| Mapa | prawdziwe kafelki OSM, zoom do ulicy | własna geometria wektorowa |
| Leaflet | wbudowany w plik (nie z CDN) | wbudowany w plik |
| Trwałość | `localStorage` | capability `artifact`, zapis do `dane/priorytety.json` |
| Współdzielenie | przez przesłanie pliku / eksport JSON | wspólny stan, działa na telefonie |
| Rola | robocza selekcja przy dużym ekranie | podgląd i selekcja w podróży |

- **Podkład wektorowy** wersji artifact: linia brzegowa 100 m z pakietu npm
  `@geo-maps/earth-lands-100m`, przycięta do prostokąta (−125,30; 48,15)–(−121,55; 50,70),
  uproszczona algorytmem Douglas–Peucker (tolerancja ~35 m), 185 wielokątów / ~6,7 tys.
  punktów / ok. 130 KB. Jeziora zachowane tylko powyżej ~7 km². To realna geometria,
  nie rysunek — pinezki lądują dokładnie na swoich współrzędnych.
- **Zasada nadrzędna bez zmian: dane oddzielone od silnika.** Lista miejsc żyje jako
  `claude/vancouver-miejsca.json` w wiedzy projektu. Kod mapy jest wymienny, dane nie.
- **Nie używać** map Google z kluczem API, backendu, bazy danych.

---

## 4. Schemat pinezki

```json
{
  "id": "sp-001",
  "nazwa": "",
  "kategoria": "halloween | natura | miasto | jedzenie | muzeum | widok",
  "lat": 0.0,
  "lng": 0.0,
  "dzielnica": "",
  "czas_zwiedzania_h": 0,
  "indoor": true,
  "swiatlo": "dzien | zachod | zmrok | \"\"",
  "koszt_cad": 0,
  "godziny": "",
  "sezon_do": "",
  "wymaga_auta": false,
  "wymaga_rezerwacji": false,
  "priorytet": "must | nice | skip",
  "link": "",
  "sciezka": [[49.28, -123.11], "..."],
  "dystans_km": 0,
  "przewyzszenie_m": 0,
  "gluten": "dedykowane | opcje | ostrożnie | brak | \"\"",
  "laktoza": "dedykowane | opcje | ostrożnie | \"\"",
  "notatka": ""
}
```

Kategoria `spacer` doszła w wersji 2 danych (piesze wycieczki z przewodnikiem).

`gluten` i `laktoza` wypełniamy tylko dla punktów jedzeniowych. `dedykowane` znaczy
zero ryzyka — cała kuchnia bez glutenu albo cały lokal roślinny. `opcje` znaczy
wiarygodne, oznaczone pozycje w menu. `ostrożnie` znaczy: coś się znajdzie, ale
kontaminacja realna. Pusty string = punkt niejedzeniowy.

`swiatlo` (od wersji 6 danych) mówi silnikowi, jakiego światła miejsce potrzebuje:
`dzien` — tylko za dnia (szlaki, natura, widoki, wypady), `zachod` — najlepsze wokół
zachodu (punkty widokowe, złota/błękitna godzina), `zmrok` — sens ma po ciemku
(Fright Nights, Canyon Frights, parada, spacery z duchami, zdjęcia nocne), pusty — obojętne
(jedzenie, muzea, miasto). Wartości nadane wg kategorii i `foto_pora`, ręcznie poprawione
dla wyjątków — można zmieniać per miejsce.

Pola `indoor`, `sezon_do`, `wymaga_auta` i `swiatlo` są krytyczne — bez nich plan rozsypie się
pierwszego deszczowego dnia albo wyśle nas na atrakcję zamkniętą po sezonie.

---

## 5. Etapy

| # | Etap | Status |
|---|---|---|
| 0 | Brief — daty, dzielnica, budżet, auto, priorytety | W TOKU (2 braki: dzielnica, budżet) — daty ustalone 18.10 |
| 1 | Schemat danych | GOTOWE (sekcja 4) |
| 2a | Rozpoznanie wydarzeń z datami (Halloween, sezonowe) | CZĘŚCIOWO — 6 pozycji zweryfikowanych, sekcja 6 |
| 2b | Rozpoznanie stałych miejsc (~80–100 punktów) | GOTOWE — 80 punktów, 14 ze szlakiem |
| 3 | Kuracja listy przez Wawrzyńca i partnerkę (must/nice/skip) | MOŻLIWA — narzędzie gotowe |
| 4 | Silnik v1 — mapa, pinezki, filtry | GOTOWE |
| 5 | Silnik v2 — dwa głosy, plan 14 dni, tryb telefonu | GOTOWE |
| 5b | Silnik v3 — plan jako panel na mapie, edycja przeciąganiem | GOTOWE |
| 5c | Silnik v4 — szlaki piesze rysowane na mapie | GOTOWE |
| 5d | Silnik v5 — termin 24.10–12.11 (20 dni), światło dzienne w planie dnia | GOTOWE |
| 6 | Układanie 20 dni | MOŻLIWE — daty są, narzędzie gotowe |
| 7 | Zamrożenie, eksport JSON, wersja do druku | — |

**Co umie silnik v1.1:** mapa z 55 pinezkami kolorowanymi kategorią, filtry (kategoria,
pod dachem, bez auta, sezonowe, **bezpieczne GF**, dzielnica, priorytet, szukajka),
oznaczanie must/nice/skip z mapy i z listy, liczniki, podsumowanie wybranych must
(godziny, koszt, ile wymaga auta), plakietki diety, link do strony w dymku,
eksport i import JSON.

**Co doszło w v2:** osobne głosy Wawrzyńca i Ewy (przełącznik w górnym pasku, licznik
„oboje must" i „sporne"), plan 14 dni jako kalendarz godzinowy 7:00–23:00, data startu
wyjazdu z automatycznym dniem tygodnia, ostrzeżenie przy poniedziałku i przy punkcie
po sezonie, tryb telefonu.

**Co doszło w v3 — jeden widok zamiast dwóch zakładek:** plan dnia to pływający panel
w lewym górnym rogu mapy, więc mapa jest widoczna przez cały czas wybierania. Punkty
wpisane w bieżący dzień zamieniają się na mapie w **gwiazdki** w kolorze kategorii.
Bloki w kalendarzu edytuje się **przeciąganiem**: za dolną krawędź zmienia się długość,
za górną godzinę startu, za środek przesuwa się cały blok (skok 15 minut, działa też
palcem). **Kliknięcie pustej godziny otwiera pole notatki** — wpisujesz tekst i masz
własny wpis w planie. Przycisk „+" przy punkcie na liście i w dymku wstawia go
w wybrany dzień.

**Co doszło w v5 — termin i światło:** liczba dni liczona z dat start/koniec (domyślnie
24.10–12.11 = 20 dni), dzień 1 oznaczony „przylot", ostatni „wylot". Siatka 6:00–24:00
zacieniowana wg słońca dla Vancouver (noc, zmierzch cywilny, dzień), przerywana linia
zachodu, w nagłówku wschód–zachód i długość dnia, w liście dni godzina zachodu.
Bloki z `swiatlo: dzien` kończące się po zachodzie — czerwone ostrzeżenie; szlak kończący
się mniej niż 30 min przed zachodem — „na styk"; `zmrok` przed zachodem — „jeszcze jasno".
Trzymając punkt do wstawienia widać zielone okno startu. Zapisany wcześniej plan jest
przesuwany tak, żeby wpisy zostały przy swoich datach, nie numerach dni.

**Czego jeszcze nie umie:** dodawania i edycji pinezek z poziomu mapy, checklist,
wersji do druku. Tras świadomie nie robimy — plan dnia wystarcza.

---

## 5a. Jak Ewa ocenia na swoim urządzeniu

Ewa nie ma konta Claude, więc pracuje na własnej kopii pliku, nie na wspólnym stanie.

1. Wawrzyniec wysyła jej `vancouver-silnik.html` (albo link, jeśli plik trafi na
   GitHub Pages).
2. Ewa otwiera plik w Chrome lub Safari — **nie w przeglądarce wbudowanej w
   komunikator**, bo tam dane potrafią zniknąć. Przełącza „oceniasz jako" na siebie.
3. Ocenia, potem klika **Eksport** — dostaje `vancouver-oceny-ewa.json`.
4. Wawrzyniec klika **Import** i wskazuje ten plik. Jej głosy lądują w kolumnie Ewy,
   jego własne zostają nietknięte.

Import scala tylko niepuste głosy, więc wymiana w obie strony jest bezpieczna.
Plan dnia przenosi się razem z plikiem, jeśli lokalny plan jest pusty.

---

## 6. Ustalenia merytoryczne z rozpoznania

**Halloween to termin, nie motyw.** Praktycznie wszystko halloweenowe kończy się
31 października i znika z dnia na dzień. Tydzień po Halloween to zupełnie inny wyjazd.

**Co wynika z dat 24.10–12.11:**

- Okno halloweenowe to **dni 1–8 (24–31.10)**, w tym dzień przylotu. 31.10 to **sobota** —
  parada i ostatnia noc Fright Nights wypadają w weekend, bilety na 31.10 kupić od razu.
  Canyon Frights jeszcze w niedzielę 1.11.
- **1.11 (niedz) zmiana czasu** — zachód skacze z ok. 17:55 na 16:53.
- **Sea to Sky Gondola tylko do 8.11 (niedz)** — od dnia 17 nieczynna.
- Poniedziałki: 26.10, 2.11, 9.11 — muzea z poniedziałkową przerwą odpadają.
- 11.11 (śr) — Remembrance Day, święto państwowe; część instytucji krócej lub zamknięta.
- Farmy dyniowe (Maan Farms) i wszystko z `sezon_do: 2026-10-31` — tylko pierwszy tydzień.

**Zweryfikowane daty na 2026:**

- **Fright Nights w Playland (PNE)** — otwarcie 7.10.2026, bilety w sprzedaży, od 39 CAD.
  Noc 31.10 wyprzedaje się pierwsza. Wiek 13+.
- **Canyon Frights (most Capilano)** — 9.10–1.11.2026, 10:00–20:00, w wybrane dni do 21:00.
  W cenie wstępu do parku. **Jedyna duża impreza halloweenowa czynna jeszcze 1 listopada.**
- **Parade of Lost Souls (Commercial Drive)** — tylko 31.10, darmowa, procesje 19/20/21.
- **Harvest Days w VanDusen** — 26.09–18.10.2026. **Kończy się przed waszym terminem.**
- **Sea to Sky Gondola** — coroczna przerwa techniczna **9–27 listopada**. Wyjazd sięgający
  poza 8.11 nie złapie gondoli.
- **PEAK 2 PEAK w Whistler** — stoi od połowy października do połowy listopada, czyli
  dokładnie w waszym oknie. Whistler bez głównej atrakcji — rozważyć, czy warto 2 h jazdy.
- **Ghost Train w Stanley Parku** — NIE planować. Kolejka nie jeździ od 2025.
- **HowlOver Canada (FlyOver)** — daty 2026 niepotwierdzone, w poprzednich latach do 31.10.

**Do sprawdzenia w dalszym rozpoznaniu:** Fang-Tastic w Britannia Mine (status 2026),
Haunted Village w Burnaby, farmy w Fraser Valley poza Maan Farms (Taves, Chilliwack
Corn Maze), Grouse Mountain między sezonami.

---

## 6a. Jedzenie przy nietolerancji glutenu i laktozy

Vancouver jest pod tym względem jednym z lepszych miast w Ameryce Północnej, ale
bezpieczeństwo bierze się z konkretnych lokali, nie z dzielnic. Dlatego wpisy typu
„Main Street" czy „Alexandra Road" mają w danych flagę `ostrożnie` — same z siebie
nie są bezpieczne.

**Lokale w 100% bezglutenowe (zero kontaminacji):**

- **Saan Saan**, 227 Main St — hongkońska kawiarnia, cała kuchnia GF. Najlepszy
  pierwszy przystanek, w zasięgu spaceru z Gastown i Chinatown.
- **The Gluten Free Epicurean**, 633 E 15th Ave — piekarnia, chleb na zakwasie,
  kanapki, quiche, mrożonki. Tu robi się zapasy.
- **Iki Japanese**, 2756 W Broadway — sushi i tempura, akredytacja stowarzyszenia
  celiakii. Tylko na rezerwację.
- **Bon Macaron**, Granville Island — makaroniki, deser bezpieczny bez pytania.

**Podwójnie bezpieczne (roślinne, więc też bez nabiału):** Virtuous Pie (pizza,
spód GF za dopłatą), Chickpea (kuchnia izraelska).

**Dedykowane frytkownice GF:** Juke Fried Chicken, The Fish Counter — czyli smażone
rzeczy, które normalnie są niedostępne.

**Kawiarnie:** mleko roślinne jest w Vancouver standardem. Jedzenia bezglutenowego
w kawiarniach zwykle nie ma — wyjątki to Matchstick i Aphrodite's (ciasta GF).

**Do zweryfikowania na miejscu:** godziny większości lokali gastronomicznych nie były
potwierdzone na oficjalnych stronach. Przed wyjściem sprawdzić.

---

## 6b. Szlaki piesze — co wybrać

Dziewięć punktów ma narysowany przebieg na mapie (pole `sciezka`). **Linie są
orientacyjne** — pokazują kierunek i rejon, nie serpentyny. Do nawigacji w terenie
mapa szlaków albo AllTrails.

**Na górę:**

- **Stawamus Chief** (Squamish) — 6 km, 600 m w górę, 2,5–4 h. Granitowy monolit nad
  fiordem, drabiny i łańcuchy. Sezon III–XI, czyli łapiecie na styk. Po deszczu granit
  śliski. Parking wspólny z Shannon Falls. **To jest ta góra.**
- **BCMC Trail** — 3 km, 800 m. Ważne: **Grouse Grind jest zamknięty na remont
  od początku października do początku listopada**, BCMC obok zostaje otwarty.
  W dół Skyride za ok. 25 CAD.
- **Dog Mountain** — 5 km, prawie płasko, ale na wysokości 1000 m. Pod koniec
  października potrafi tam już leżeć śnieg. Bez raczków nie wchodzić.

**Po lesie:**

- **Brothers Creek** (West Van) — 7 km, 400 m. Hollyburn Fir: 1100 lat, 44 m, 10 m obwodu.
  Najstarsze drzewo w okolicy.
- **Lighthouse Park** — 3 km. Cedry po 600 lat, Seven Sisters, skaliste wybrzeże.
- **Cypress Falls Park** — 3,5 km. 300-letnie cedry i dwa wodospady, mało turystów.
- **Lynn Loop** — 5,1 km. Czysty las deszczowy, łatwiejszy i pustszy niż Lynn Canyon.

**Płasko i miejsko:** Stanley Park Seawall 9 km (przebieg wycięty z prawdziwej linii
brzegowej, więc ta jedna trasa jest dokładna), Quarry Rock 3,8 km.

**Całodniowe, 10–22 km — wszystkie czynne cały rok i nisko, więc bez śniegu:**

- **Baden-Powell, Deep Cove → Lynn Canyon** — 12 km w jedną stronę, 420 m, 5 h.
  Najlepsza długa trasa **bez auta**: dojazd SeaBus + 239 + 212, powrót 229 z Peters Road.
  Nie jest pętlą. Po drodze Quarry Rock, most nad Seymour, Twin Falls, kładki nad Lynn Creek.
- **Norvan Falls** — 14 km w obie strony, tylko 195 m przewyższenia. Dolina Lynn Creek
  coraz głębiej w las. Ostatnie 2,6 km bywa mocno błotniste.
- **Pełny seawall** — 22 km, płasko, od Coal Harbour przez Stanley Park i False Creek
  do Kitsilano. Jedyna długa trasa wykonalna w ulewę; można przerwać w dowolnym miejscu.
- **Seymour Valley Trailway** — 20 km po asfalcie zamkniętym dla ruchu. Opcja na dzień
  po ulewie, gdy leśne szlaki toną. Da się rowerem w 2 h.
- **Buntzen Lake** — 10,5 km pętla wokół jeziora. Brama parkingu zamyka się wcześnie
  poza sezonem.

---

## 7. Zasady planowania dni

- **2–3 punkty na dzień, nie 6.** Listopad w Vancouver to deszcz i wczesny zmrok.
- **Światło dzienne (Vancouver, liczone przez silnik):**

  | Data | Wschód | Zachód | Ciemno od | Dzień |
  |---|---|---|---|---|
  | 24.10 | 7:49 | 18:07 | 18:40 | 10 h 18 |
  | 31.10 | 8:00 | 17:55 | 18:28 | 9 h 55 |
  | 1.11 (zmiana czasu) | 7:02 | 16:53 | 17:26 | 9 h 51 |
  | 8.11 | 7:13 | 16:42 | 17:16 | 9 h 29 |
  | 12.11 | 7:20 | 16:36 | 17:11 | 9 h 16 |

  Szlaki i wypady autem zaczynamy rano, z zapasem 30 min przed zachodem. Popołudnie po
  zmroku = muzeum, jedzenie, Halloween, zdjęcia nocne. Długie trasy (5–6 h) lepiej
  w październiku, kiedy dzień jest dłuższy o godzinę.
- Każdy dzień ma **plan B pod dachem** na wypadek ulewy (filtr „pod dachem" w silniku).
- Punkty wymagające auta **grupujemy w bloki**, żeby wystarczył wynajem na 4–5 dni
  zamiast na dwa tygodnie — na wypadek gdyby auto od znajomych nie wypaliło.
- Farmy z dyniami w Fraser Valley: tylko do końca października, wymagają auta.
- Zostawić **1–2 dni buforowe** bez planu.
- Uwzględnić dni spędzane z przyjaciółmi i dzień tygodnia (poniedziałki — część
  muzeów zamknięta, m.in. Museum of Anthropology).

---

## 8. Do wklejenia w instrukcje projektu

> Pracujemy nad narzędziem do planowania dwutygodniowego wyjazdu do Vancouver
> (przełom X/XI 2026, 2 osoby, nocleg u przyjaciół). Główny motyw: jesień i Halloween.
> Silnik istnieje w dwóch wersjach: plik lokalny z Leafletem i kafelkami OSM oraz
> artifact z własnym podkładem wektorowym. Dane o miejscach trzymamy jako osobny JSON
> (`claude/vancouver-miejsca.json`) zgodny ze schematem z `vancouver-brief.md` — kod jest
> wymienny, dane nie. Odpowiadaj po polsku, konkretnie, bez lania wody. Przed dodaniem
> miejsca do planu sprawdzaj, czy jest czynne w danym okresie i czy wymaga auta.
