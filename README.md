# Rynek Analiza - dodatek do BK-Rynek 3.0 (Minecraft 1.21.4, Fabric)

Osobny mod, który liczy ceny skupu i wpisuje je prosto w listę przedmiotów BK-Rynek.
Nie skompilowany na prawdziwym Minecraft/Fabric (tylko na atrapach API) i nie uruchomiony w grze.

## Jak zbudować
1. Nowe repo/folder. Z projektu AutoRynka skopiuj: `build.gradle`, `gradle.properties`, `settings.gradle`,
   `gradle/`, `gradlew`, `gradlew.bat`. Zmień nazwę archiwum / mod id na `rynek-analiza`.
2. Wrzuć folder `src/` z tego zipa.
3. Zbuduj jar i wrzuć go do `mods/` razem z BK-Rynek.

## Użycie
- `/analiza start` - wszystkie przedmioty z aktywnego profilu BK
- `/analiza start <fragment nazwy>` - tylko wybrane
- `/analiza stop`
- `/analiza lista` - przedmioty, aktualna cena skupu, marża
- `/analiza marza <kwota> <fragment nazwy>` - marża w $ dla przedmiotów (np. 100, 200, 500)
- `/analiza marza_domyslna <kwota>` - marża dla przedmiotów bez własnej (domyślnie 100)
- klawisz F8 (do zmiany w sterowaniu) - start/stop analizy wszystkich

Config: `config/rynek-analiza.json` (marże, własne frazy po `/ah search`, podatek, liczba ofert, opóźnienia).

## Podgląd podczas analizy (do weryfikacji)
- Nad rynkiem pojawia się pasek z ikoną i nazwą analizowanego przedmiotu, numerem (np. 2/7), stroną i liczbą znalezionych ofert.
- Oferty, które w danej chwili są w gronie najtańszych (te idą do średniej), mają żółtą ramkę i żółte tło.
  Strona z takimi ofertami jest trzymana chwilę (`highlightPauseTicks` w configu, 20 = 1 s), żeby dało się je obejrzeć.
- Po policzeniu w czacie jest linia "Użyte oferty: str.1 r1 k1 = 950/szt; ..." (strona, rząd, kolumna, cena za sztukę) - to ostateczny wybór.
- Podświetlenie pokazuje stan na moment skanowania strony. Dla przedmiotów niestackowalnych (hełmy, narzędzia) jest to od razu wynik końcowy,
  dla stackowalnych późniejsza strona może wyprzeć wcześniejszą ofertę (większy stack = niższa cena za sztukę).
- Wyłączenie: `"showOverlay": false` w `config/rynek-analiza.json`.

## Co dzieje się podczas analizy
BK-Rynek jest zatrzymywany (i wznawiany po analizie, jeśli działał). Dla każdego przedmiotu:
`/ah search <nazwa>` -> PPM w repeater dwa razy ("Najniższej ceny") -> skan wszystkich stron ->
cena za sztukę z każdej pasującej oferty -> średnia 3 najtańszych * 0.97 - marża -> `maxPrice` wpisu w BK
-> zapis configu BK i przeładowanie cen.
Dopasowanie ofert = to samo, którego używa auto-kupno BK (`AutoMarketScanner.matchesEntry`) + zgodna liczba komponentów.
Mniej niż 3 oferty = cena nie zostaje ustawiona.

## Do sprawdzenia przy pierwszym teście
1. `/analiza lista` pokazuje przedmioty z BK. Jeśli nie, most do BK nie działa (komunikat błędu w czacie).
2. `/analiza start <jeden przedmiot>` i obserwacja czatu: czy rynek się otwiera, czy sortowanie klika 2x, czy ceny wyglądają sensownie.
3. Fraza po `/ah search` to domyślnie nazwa przedmiotu bez kolorów. Jeśli serwer szuka inaczej, wpisz własną w `search` w configu.
4. Cena z lore: używany jest `lore_regex` z profilu BK, a gdy go brak - pierwsza liczba po słowie "Koszt".
5. BK porównuje cenę za sztukę tylko przy włączonym trybie "Unit" w profilu - sprawdź, że masz go włączony.
