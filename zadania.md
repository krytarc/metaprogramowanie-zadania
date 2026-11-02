# Metaprogramowanie

Pięć zadań pokazuje różne zastosowania obliczeń i decyzji podejmowanych podczas kompilacji: funkcje `constexpr`, klasyczne szablony, wybór kodu zależnie od typu, generowanie tablic oraz sprawdzanie konfiguracji.

Do każdego zadania przygotowano osobny projekt CMake i plik startowy `main.cpp`: [zadanie 1](zadanie1), [zadanie 2](zadanie2), [zadanie 3](zadanie3), [zadanie 4](zadanie4), [zadanie 5](zadanie5).

## Zanim zaczniesz

**Wersja podstawowa wymaga C++17.** Zadanie 5 ma dodatkowy wariant C++20 z `consteval` i `constinit`; na starszym kompilatorze wykonaj opisany wariant C++17. Nie potrzebujesz C++23, konceptów ani zewnętrznych bibliotek.

Zadania odpowiadają zagadnieniom z notatki: 1 — podstawy `constexpr`; 2 — klasyczne TMP; 3 — `if constexpr`, pakiety parametrów i fold; 4 — tablice w czasie kompilacji; 5 — `consteval` i `constinit`.

W katalogu wybranego zadania wykonaj `cmake -S . -B build`, następnie `cmake --build build`. Na Linuksie uruchom program przez `./build/program`. Bez CMake możesz użyć `g++ -std=c++17 -Wall -Wextra -pedantic main.cpp -o program`.

Dla wariantu C++20 zadania 5 użyj `cmake -S . -B build20 -DUSE_CPP20=ON` i `cmake --build build20` albo kompiluj z `-std=c++20`. Wybierz ten wariant, jeśli Twój kompilator obsługuje oba wymagane słowa kluczowe.

Każde zadanie jest niezależne. Pliki startowe zawierają nagłówki i komentarze TODO, bez rozwiązań. Błędne przykłady kompilacyjne pozostaw zakomentowane i opisz przyczynę błędu.

## 1. „Ile miejsc w sali?”

**Najprostsze.** Organizator ustawia krzesła w prostokącie. Napisz funkcję `constexpr int liczbaMiejsc(int rzedy, int miejscaWRzedzie)`, zwracającą liczbę miejsc. Przyjmij oba argumenty z zakresu `1–100`; obsługa błędnych danych nie jest częścią zadania.

Zapisz wynik dla 3 rzędów po 4 miejsca w zmiennej `constexpr`. Wykorzystaj go jako rozmiar `std::array<bool, ...>`, reprezentującej zajęte miejsca. Potwierdź przez `static_assert`, że wynik i rozmiar tablicy wynoszą `12`, a dla 1 rzędu po 8 miejsc wynik wynosi `8`.

Następnie wczytaj liczbę rzędów i miejsc z klawiatury, wywołaj tę samą funkcję i wypisz wynik. Dla wejścia `5 6` oczekuj `30`. Załóż poprawne wejście.

Dodaj zakomentowaną próbę zapisania wyniku wywołania z danymi z klawiatury do zmiennej `constexpr`. Wyjaśnij, dlaczego jest niedozwolona i dlaczego samo oznaczenie funkcji jako `constexpr` nie wymusza obliczenia każdego wywołania podczas kompilacji.

**Wskazówka:** wynik funkcji może być argumentem szablonu, jeśli jest wyrażeniem stałym.

## 2. „Silnia z samych szablonów”

**Najprostsze.** Oblicz silnię inną techniką niż funkcja z zadania 1. Napisz szablon klasy `Silnia<N>`, udostępniający wynik jako `Silnia<N>::value` typu `unsigned long long`.

Zastosuj rekurencję szablonów i pełną specjalizację dla `N = 0`. Parametr `N` ma mieć typ `unsigned`; przyjmij zakres zadania `0–20`. Nie korzystaj z funkcji obliczającej silnię ani nie wpisuj gotowych wyników dla kolejnych argumentów.

Sprawdź przez `static_assert`:

- `Silnia<0>::value` → `1`;
- `Silnia<1>::value` → `1`;
- `Silnia<5>::value` → `120`;
- `Silnia<10>::value` → `3628800`.

W `main()` wypisz jeden wynik. W komentarzu wskaż warunek kończący rekurencję i wyjaśnij, dlaczego liczba wczytana z klawiatury nie może być argumentem `N`.

**Wskazówka:** odwołanie do `Silnia<N - 1>::value` wymaga utworzenia kolejnej specjalizacji szablonu.

## 3. „Jedna funkcja, różne typy”

**Konkretne.** Przygotuj szablon funkcji `wypisz(const T& wartosc)`, który wypisuje jeden wiersz raportu. Za pomocą `if constexpr` wybierz zachowanie zależnie od typu:

- `bool` → `logiczna: tak` lub `logiczna: nie`;
- pozostałe typy całkowite → `calkowita: wartość`;
- typy zmiennoprzecinkowe → `rzeczywista: wartość`;
- `std::string` → `tekst: wartość`;
- inne typy → czytelny błąd `static_assert` zależny od parametru szablonu.

Napisz też `wypiszWszystko(const Ts&... wartosci)`, wywołującą `wypisz` dla każdego argumentu za pomocą wyrażenia fold. Pusty pakiet ma być dozwolony i niczego nie wypisywać.

Dla `wypiszWszystko(42, 2.5, true, std::string{"OK"})` oczekuj:

```text
calkowita: 42
rzeczywista: 2.5
logiczna: tak
tekst: OK
```

Sprawdź także `false`, liczbę `-7` i pusty pakiet. Dodaj zakomentowane wywołanie z `std::array<int, 2>`, które ma zostać odrzucone. Obsługa literałów tekstowych zamiast `std::string` nie jest wymagana.

**Wskazówka:** przydadzą się `std::is_same_v`, `std::is_integral_v` i `std::is_floating_point_v`. Sprawdzaj `bool` przed pozostałymi typami całkowitymi. Dla zgodności ze starszymi kompilatorami użyj zależnego fałszu, np. `template<class> inline constexpr bool zawsze_falsz_v = false;`.

## 4. „Liczby pierwsze gotowe przed startem”

**Praktyczne.** Program wielokrotnie sprawdza, czy podana liczba jest pierwsza. Zamiast za każdym razem szukać dzielników, przygotuj tablicę odpowiedzi podczas kompilacji.

Napisz szablon funkcji `constexpr auto przygotujSito<N>()`, zwracający `std::array<bool, N + 1>`. Element pod indeksem `i` ma wskazywać, czy `i` jest liczbą pierwszą. Zastosuj sito Eratostenesa: początkowo oznacz liczby od 2 jako potencjalnie pierwsze, następnie wykreślaj wielokrotności kolejnych liczb pierwszych. Przyjmij `2 <= N <= 10000`.

Utwórz tablicę `constexpr` dla `N = 30`. Nie wpisuj jej zawartości ręcznie i nie generuj jej ponownie w `main()`.

Sprawdź przez `static_assert`, że `0`, `1`, `4`, `25` i `30` nie są pierwsze, a `2`, `3` i `29` są pierwsze.

W programie głównym wczytuj liczby do końca wejścia. Dla każdej liczby z zakresu `0–30` wypisz `pierwsza` albo `zlozona lub mniejsza od 2`. Dla liczby spoza zakresu wypisz `poza zakresem`; nie odczytuj wtedy tablicy. Załóż wejście zawierające liczby całkowite mieszczące się w `int`.

Sprawdź wejście `2 25 29 0 -1 31`. W komentarzu wyjaśnij, co oblicza kompilator, co robi program po uruchomieniu i jaką cenę w pamięci płacimy za gotową tablicę.

**Wskazówka:** funkcja `constexpr` może zawierać pętle. W C++17 można w niej modyfikować elementy lokalnej `std::array`; zainicjalizuj ją przez `{}`.

## 5. „Konfiguracja sprawdzona przed startem”

**Praktyczne.** Urządzenie wysyła pomiary w ustalonym odstępie i ponawia nieudane wysyłki. Przygotuj strukturę `Konfiguracja` z polami `int okresMs` oraz `int proby` i funkcję `przygotujKonfiguracje(int okresMs, int proby)`.

Poprawna konfiguracja wymaga okresu `100–60000` ms oraz liczby prób `1–10`. Funkcja ma zwracać strukturę dla poprawnych danych, a dla błędnych zgłaszać wyjątek, np. `std::invalid_argument`.

**Wariant podstawowy — C++17.** Oznacz funkcję jako `constexpr`. Utwórz globalną konfigurację domyślną `constexpr` z wartościami `1000` i `3`; pola sprawdź przez `static_assert`. Zainicjalizuj nią osobną, modyfikowalną zmienną globalną. W `main()` zmień jej okres na `1500` i wypisz obie konfiguracje, pokazując, że domyślna pozostała bez zmian.

Sprawdź też wywołanie funkcji ze zmiennymi wczytanymi z klawiatury: `100 1` i `60000 10` są poprawne, a `99 3` i `1000 11` mają zostać odrzucone z obsłużonym wyjątkiem. Dodaj zakomentowany przykład inicjalizacji zmiennej `constexpr` błędną konfiguracją — to błąd kompilacji.

**Wariant dodatkowy — C++20.** Zmień funkcję na `consteval`, a modyfikowalną konfigurację globalną oznacz `constinit`. Zachowaj demonstrację zmiany okresu na `1500`. Wywołania funkcji z danymi z klawiatury przenieś do zakomentowanych przykładów: teraz mają być błędem kompilacji. Wybór wariantu w kodzie możesz oprzeć na `#if __cplusplus >= 202002L`.

W komentarzu porównaj gwarancje `constexpr`, `consteval` i `constinit`. Wyjaśnij, dlaczego błędne dane w wyrażeniu stałym uniemożliwiają kompilację, choć wywołanie funkcji `constexpr` podczas działania programu może zgłosić obsługiwany wyjątek.

**Wskazówka:** wyjątek może znajdować się w funkcji `constexpr`, ale wykonanie instrukcji `throw` nie jest dozwolone podczas obliczania wyrażenia stałego. `constinit` wymusza stałą inicjalizację zmiennej, lecz nie czyni jej niemodyfikowalną.
