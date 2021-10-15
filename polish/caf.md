# CAF - COMES Archive Format

- **Wersja**: 1 (`jeden`)

COMES Archive Format to nowatorski format archiwizacji plików poprzez pakowanie wielu plików w jednen plik archiwalny.

## Enkodowanie

Pliki CAF enkodowane są enkodowaniem UTF-8. Jako koniec linii uznaje się LF (ASCII 0Ah).

## Liczby

Liczby w CAF zapisywane są słownie, po polsku, wykorzystując znaki diakrytyczne. Minimalna obsługiwana liczba to 0 (zero), a maksymalna obsługiwana liczba to 255 (dwieście pięćdziesiąt pięć). Typ ten jest nazywany `liczba8`.

W niektórych miejscach potrzebne są liczby większe od 255. W tym wypadku liczby koduje się jako wiele `liczba8`, oddzielonych znakami `<<`, np. `trzy<<dwieście pięćdziesiąt pięć<<sto dwadzieścia osiem`. Maksymalna obsługiwana liczba zapisana tym sposobem to `2⁶⁴ - 1`. Typ ten jest nazywany `liczbaZ64` (liczba o zmiennej długości gdzie maksymalna długość to 64 bity).

Liczby z ciągu muszą być zapisane do listy, gdzie pierwszy element to liczba po lewej, a ostatni element to liczba po prawej. Następnie liczba jest dekodowana następującym algorytmem.
```
var liczby = [...] -- odczytane przez parser
var wynik = 0
for i in 0 .. liczby.length
  var fragment = liczby[i]
  wynik = (wynik << 8) | fragment
```
Operator `a << n` przesuwa bity w liczbie `a` o `n` bitów. Operator `|` jest operacją bitową OR.

## Ścieżki

Nazwa pliku w CAF może składać się z dowolnych znaków oprócz ASCII 00h, `/`, oraz końca linii. Nazwa pliku również nie może być znakiem `.` lub ciągiem znaków `..`, z powodów bezpieczeństwa.

Ścieżka jest to lista nazw plików oddzielona znakiem `/`, określająca ścieżkę w hierarchii archiwum. Przykłady ścieżek:
- `plik`
- `mój/folder/mojplik.txt`

## Struktura

Każdy plik CAF jest rozłożony w następujący sposób.
 - nagłówek
 - indeks
 - pliki

### Nagłówek

Nagłówek pliku CAF jest to pierwsza linijka w pliku. Zawiera ona słowo `CAF` oraz wersję formatu (`liczba8`), w tym przypadku `jeden`.
```
CAF jeden
```


### Indeks

Indeks określa informacje o plikach w archiwum. Rozpoczyna się linijką ze słowem `INDEKS`, spacją, oraz ilością wpisów w indeksie (`liczbaZ64`), opisywaną w reszcie dokumentu jako `n_wpisów`.
```
INDEKS pięć
```
Następne `n_wpisów` linijek musi się zaczynać słowem `KATALOG` oraz ścieżką katalogu oddzieloną od słowa kluczowego spacją, lub słowem `PLIK` oraz nazwą pliku, również oddzieloną od słowa `PLIK` spacją.
```
PLIK czytajto.txt
KATALOG muzyka
PLIK piosenka.ogg
KATALOG obrazki
PLIK bob.png
```
Pierwszy podany przykład enkoduje następującą strukturę plików:
```
archiwum.caf
├── czytajto.txt
├── muzyka
│   └── piosenka.ogg
└── obrazki
    └── bob.png
```

Słowo `KATALOG` określa w jakim katalogu mają się znajdować pliki następujące katalog. Każda dyrektywa `KATALOG` specyfikuje ścieżkę od katalogu głównego archiwum, i może być używana do rekurencyjnego tworzenia ścieżek.
```
KATALOG pierwszy
KATALOG pierwszy/drugi
```

### Pliki

Po indeksie znajduje się już tylko lista plików. Plików musi być tyle samo co wpisów w indeksie o typie `PLIK`. Ścieżki plików ustalone są w indeksie, więc same pliki ich nie enkodują.

Każdy plik rozpoczyna się słowem `ROZMIAR` oraz ilością bajtów, które zajmuje plik:
```
ROZMIAR trzysta
```
Ilość bajtów (`n_bajtów`) enkoduje się typem `liczbaZ64`.

Każda linijka enkoduje jedną lub więcej `liczbaZ64`. W związku z tym, zenkodowana ilość bajtów jest podzielna przez 8. Na końcu pliku może być maksymalnie 7 dodatkowych bajtów `zero`, które muszą zostać zignorowane.

Linijki enkodujące bajty mogą mieć dwie formy. Pierwszą z nich jest pojedyncza liczba, i jest nią zwyczajnie pojedyncza `liczbaZ64` na linijce:
```
pięć<<zero
```
Druga forma to _powtórzenie_ bajtów. Powtórzenie określane jest ciągiem znaków ` X ` po liczbie, oraz ilością powtórzeń (`liczbaZ64`):
```
zero X sześćdziesiąt cztery
```

Każda `liczbaZ64` odczytana w ten sposób zapisywana jest w ostatecznym pliku jako cztery bajty w kolejności _big-endian_. Przykłady:

**A.** Bajty zerowe.
```
zero
```
enkoduje się do
```
00 00 00 00 00 00 00 00
```

**B.** Cztery różne bajty.
```
jeden<<dwa<<trzy<<cztery
```
enkoduje się do
```
00 00 00 00 01 02 03 04
```

**C.** Dwa bajty.
```
jeden<<dwa
```
enkoduje się do
```
00 00 00 00 00 00 01 02
```

## Przykład minimalnego pliku CAF

Następujący przykład to minimalny plik CAF z jednym plikiem `czesc.txt` w środku, o tekście `Cześć!`.
```
CAF jeden
INDEKS jeden
PLIK czesc.txt
ROZMIAR osiem
sześćdziesiąt siedem<<sto dwadzieścia dwa<<sto jeden<<sto dziewięćdziesiąt siedem<<sto pięćdziesiąt pięć<<sto dziewięćdziesiąt sześć<<sto trzydzieści pięć<<trzydzieści trzy
```