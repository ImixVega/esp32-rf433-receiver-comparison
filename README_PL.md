# Porównanie 4 odbiorników RF433 na jednym ESP32

**ESPHome • 433,92 MHz ASK/OOK • AK-R231101, RX470C-V01 (2 anteny) i MX-RM-5V • pomiary do 110 m**

[English README](README.md) | [Pełny YAML testowy](esphome/esp32-rf433-benchmark.yaml)

## Po co ten projekt?

Chodziło o znalezienie odbiornika 433 MHz z cyfrowym wyjściem **DATA** do sterownika **KinCony KC868-A8 podłączonego Ethernetem** przy bramie. CC1101 wymaga SPI, natomiast moduły DATA można podłączyć do jednego wolnego wejścia GPIO. Drugi ESP32 z Wi-Fi jest w tym miejscu niepraktyczny.

Test nie obejmował **bezpośredniego porównania CC1101**. Wyłącznie antena SMA pochodziła z zestawu CC1101 433 MHz.

Próby wykonano na czterech odbiornikach podłączonych jednocześnie do **jednego klasycznego ESP32**, na płytce stykowej w domowym biurze, gdzie pracuje wiele komputerów, monitorów i innej elektroniki. Nie mierzono osobno wpływu tych zakłóceń. Odbiorniki pozostawały w budynku, pilot był wynoszony coraz dalej na zewnątrz.

**Użyty pilot:** 8-przyciskowy nadajnik 433 MHz z **wysuwaną anteną teleskopową**. To bardzo istotne: wyniki dla małych pilotów-breloków bez takiej anteny mogą być wyraźnie inne.

## Odbiorniki i anteny

| Moduł | Antena |
|---|---|
| AK-R231101 | oryginalna czarna antena zwinięta z drutu |
| RX470C-V01 nr 1 | pionowa, nieosłonięta sprężynka |
| RX470C-V01 nr 2 | antena SMA 433 MHz z zestawu CC1101, przewód koncentryczny 50 Ω |
| MX-RM-5V | najpierw bez dedykowanej anteny; **w drugiej serii dolutowana antena sprężynowa** |

**MX-RM-5V:** w obu seriach był zasilany napięciem 3,3 V. To może ograniczać jego czułość, ponieważ wiele wersji pracuje nominalnie przy 5 V. **Wynik 0% nie jest sprawiedliwą oceną prawidłowo zasilonego MX-RM-5V.** Nie wolno też podać 5 V z wyjścia DATA bezpośrednio na GPIO ESP32; trzeba sprawdzić poziomy i ewentualnie użyć konwertera.

## Seria 1 — 50 naciśnięć, kierunek od budynku

| Odległość | AK-R231101 | RX470C SMA | RX470C sprężynka | MX-RM-5V |
|---|---:|---:|---:|---:|
| 1 m (biurko) | 100% | 100% | 100% | 24% |
| 10 m (2 ściany) **⚠️** | 100% | 46% | 52% | 0% |
| 28 m | 100% | 64% | 76% | 0% |
| 57 m | 82% | 86% | 74% | 0% |
| 75 m | 0% | 16% | **60%** | 0% |

**⚠️ Pomiar 10 m nie jest w pełni porównywalny.** Antena AK była wtedy pionowa, a obie anteny RX470C poziome. Zostawiamy go tylko informacyjnie. W pozostałych próbach terenowych anteny ustawiono pionowo. Szczegóły i liczby ramek: [CSV pierwszej serii](results/range_tests.csv).

## Seria 2 — 25 naciśnięć, drugi kierunek, w stronę ulicy

Przed tą serią MX-RM-5V otrzymał własną sprężynową antenę, ale pozostał przy 3,3 V.

| Odległość | AK-R231101 | RX470C SMA | RX470C sprężynka | MX-RM-5V |
|---|---:|---:|---:|---:|
| 1 m | 100% | 100% | 88% | 0% |
| 10 m | 100% | 100% | 96% | 0% |
| 40 m | 76% | 56% | 56% | 0% |
| 60 m | 100% | 92% | 88% | 0% |
| 90 m | 64% | 88% | 96% | 0% |
| **110 m** | 88% | **100%** | **100%** | 0% |

[CSV drugiej serii](results/range_tests_series2.csv) | [Ograniczenia pomiarów](results/notes.md)

**Największa niespodzianka:** oba RX470C odebrały 25/25 z 110 m, a z 40 m tylko 14/25. Nie jest to sprzeczność: duży wpływ mogą mieć odbicia, przeszkody, ułożenie pilota i lokalne zaniki sygnału. Dwie serie miały różne kierunki, warunki i liczby naciśnięć — **nie łączymy ich w jedną krzywą zasięgu**.

## Jak podłączyć na płytce ESP32-WROOM-32?

| DATA odbiornika | GPIO ESP32 |
|---|---|
| RX470C sprężynka | GPIO25 |
| RX470C SMA | GPIO32 |
| AK-R231101 | GPIO26 |
| MX-RM-5V | GPIO27 |

Wspólna masa GND, a VCC zgodnie z dokumentacją **konkretnej płytki**. Nie zakładaj zgodności kolejności pinów tylko dlatego, że złącze ma cztery styki. **Poziom DATA na GPIO ESP32 maksymalnie 3,3 V.**

## Uruchomienie

1. Skopiuj [esp32-rf433-benchmark.yaml](esphome/esp32-rf433-benchmark.yaml) do ESPHome.
2. Uzupełnij lokalne `secrets.yaml` (przykład: [secrets.example.yaml](esphome/secrets.example.yaml)); żadnych haseł ani prywatnych kodów pilotów nie umieszczamy w repozytorium.
3. Na początku YAML wpisz ten sam *testowy* kod RCSwitch protokołu 1 w postaci **dziesiętnej i 24-bitowej binarnej**.
4. Skompiluj, wgraj firmware i dodaj ESPHome do Home Assistant.
5. Ustaw liczbę planowanych kliknięć (np. 25 lub 50), wyzeruj liczniki i przeprowadź test.

W HA pojawią się dla każdego odbiornika te same cztery typy wskaźników: **Naciśnięcia**, **Poprawne ramki**, **Skuteczność**, **Ostatni kod**. Nazewnictwo w publicznej wersji YAML jest konsekwentne, po angielsku dla szerszego grona użytkowników.

**To nie są pomiary RSSI.** Program zlicza poprawne zdekodowania. Jeden nacisk pilota zwykle wysyła kilka ramek. Filtr opóźnienia wyłączenia (800 ms) szacuje pojedyncze naciśnięcia; w niektórych warunkach zdarzają się pomyłki, a licznik może nawet przekroczyć liczbę naciśnięć. Użyto tolerancji impulsów 50%, jak w oryginalnej próbie. Ostatni kod może pokazywać także zakłócenia.

## Jak to można wykorzystać w Home Assistant?

**W samym ESP32, w ESPHome, następuje dekodowanie kodu pilota.** Osobne węzły produkcyjne mogą przekazywać do HA zdarzenia `esphome.rf433_received`. Dopiero HA przypisuje je do wybranych akcji i może deduplikować kilka odbiorników słyszących ten sam przycisk, aby nie przełączać światła dwa razy.

**W tym repozytorium jest wyłącznie YAML do testów.** Nie publikujemy programu bramy, automatyzacji oświetlenia ani rzeczywistych kodów pilotów. Stałych kodów RF433 nie należy traktować jako jedynego zabezpieczenia dostępu do posesji.

## Wniosek

RX470C-V01 ze sprężynką nadaje się do **przetestowania w docelowym sterowniku KinCony KC868-A8 podłączonym po Ethernecie**, z anteną pionowo wewnątrz plastikowej obudowy. Zakładany dystans użytkowy to około 20 m; ostateczne sprawdzenie trzeba wykonać przy bramie.

Wersja publikowana jest przystosowanym i zanonimizowanym YAML-em opartym na działającym projekcie, lecz po zmianach nie została osobno przetestowana na sprzęcie. Projekt na licencji [MIT](LICENSE).
