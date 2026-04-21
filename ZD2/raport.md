---

## Zadanie Domowe 2: Value at Risk – metody historyczna i parametryczna

---

### Cel

Obliczenie VaR dwoma metodami (historyczną i parametryczną) dla dwóch portfeli złożonych z akcji wybranych w Zadaniu 1, przy poziomach ufności 95% i 99%, oraz porównanie wyników.

---

### 1. Definicja portfeli

Zdefiniowano dwa portfele złożone z tych samych 5 spółek sektora finansowego USA (GS, C, AIG, BAC, MS), różniące się wyłącznie strukturą wag:

- **Portfel A – równe wagi (1/N):** każda spółka ma wagę 20%. Klasyczny benchmark dywersyfikacji nieważonej.
- **Portfel B – losowe wagi:** wagi wylosowane przez normalizację wartości bezwzględnych z $\mathcal{N}(0,1)$, co gwarantuje wagi dodatnie sumujące się do 1 (`seed=7` dla powtarzalności).

| Ticker | Spółka | Portfel A | Portfel B |
|--------|--------|-----------|-----------|
| GS | Goldman Sachs | 20,00% | 30,89% |
| C | Citigroup | 20,00% | 8,73% |
| AIG | AIG | 20,00% | 6,67% |
| BAC | Bank of America | 20,00% | 20,77% |
| MS | Morgan Stanley | 20,00% | 32,94% |
| **SUMA** | | **100,00%** | **100,00%** |

Dzienne stopy zwrotu portfeli obliczono jako ważoną sumę log-stóp zwrotu poszczególnych akcji: $r_t^{P} = \sum_i w_i \cdot r_{t,i}$.

---

### 2. Okno 250 dni roboczych

VaR wyznaczono na podstawie **ostatnich 250 dni roboczych** z pełnego okresu próby:

- Okno: **2007-08-03 – 2008-07-30**
- Liczba obserwacji: **250**

Jest to standardowe minimum wymagane przez regulacje Basel II/III dla modeli wewnętrznych banków. Okno to obejmuje fazę narastającego kryzysu finansowego – od sygnału BNP Paribas (09.08.2007) do końca analizowanego okresu.

![Rozkład dziennych stóp zwrotu portfeli](portfele_histogram.png)

Na histogramach widoczna jest wyraźna **dodatnia skośność** obu portfeli – rozkład jest przesunięty w prawo względem krzywej normalnej. Oznacza to, że duże dodatnie zwroty (odbicia rynkowe: marzec 2008 po Bear Stearns, lipiec 2008) są częstsze niż przewiduje symetryczny rozkład normalny, podczas gdy lewy ogon (straty) jest stosunkowo węższy empirycznie niż gaussowski.

---

### 3. Historyczny VaR

VaR historyczny = ujemny kwantyl empiryczny stóp zwrotu, bez żadnych założeń parametrycznych:

$$\text{VaR}_{\alpha}^{\text{hist}} = -Q_{1-\alpha}(r_1, \ldots, r_{250})$$

Przy 95%: 13. najgorszy wynik z 250 obserwacji. Przy 99%: 3. najgorszy wynik z 250.

| Portfel | VaR hist. 95% | VaR hist. 99% |
|---------|--------------|--------------|
| Portfel A | **4,7511%** | **5,9958%** |
| Portfel B | **4,5904%** | **5,8858%** |

![Historyczny VaR](var_historyczny.png)

---

### 4. Parametryczny VaR

Zakładamy $r_t \sim \mathcal{N}(\mu, \sigma^2)$ i wyznaczamy VaR analitycznie:

$$\text{VaR}_{\alpha}^{\text{param}} = -(\mu - z_{\alpha} \cdot \sigma)$$

gdzie $z_{0.05} \approx 1{,}645$ i $z_{0.01} \approx 2{,}326$; $\mu$ i $\sigma$ estymowane z okna 250 dni.

| Portfel | $\mu$ dzienna | $\sigma$ dzienna | VaR param. 95% | VaR param. 99% |
|---------|-------------|----------------|---------------|---------------|
| Portfel A | −0,2004% | 2,9540% | **5,0594%** | **7,0725%** |
| Portfel B | −0,1409% | 2,9784% | **5,0400%** | **7,0698%** |

![Parametryczny VaR](var_parametryczny.png)

---

### 5. Porównanie wyników

| Portfel | Poziom ufności | VaR historyczny | VaR parametryczny | Różnica (param − hist) |
|---------|---------------|----------------|------------------|----------------------|
| Portfel A | 95% | 4,7511% | 5,0594% | +0,3082% |
| Portfel A | 99% | 5,9958% | 7,0725% | +1,0767% |
| Portfel B | 95% | 4,5904% | 5,0400% | +0,4495% |
| Portfel B | 99% | 5,8858% | 7,0698% | +1,1840% |

![Porównanie VaR](var_porownanie.png)

Statystyki opisowe portfeli w oknie 250 dni:

| Miara | Portfel A | Portfel B |
|-------|-----------|-----------|
| Średnia dzienna | −0,2004% | −0,1409% |
| Odch. std. dzienna | 2,9540% | 2,9784% |
| Skośność | 0,9266 | 1,0200 |
| Kurtoza (excess) | 2,8097 | 2,9424 |
| Min (najgorszy dzień) | −7,4662% | −6,2229% |

---

### 6. Wnioski

#### Który portfel jest mniej ryzykowny?

**Portfel B (losowe wagi) jest mniej ryzykowny** według VaR historycznego na obu poziomach ufności (4,59% vs 4,75% przy 95%; 5,89% vs 6,00% przy 99%). Wylosowane wagi koncentrują się na GS (30,89%) i MS (32,94%) – bankach inwestycyjnych, które w oknie sierpień 2007 – lipiec 2008 radziły sobie relatywnie lepiej od Citigroup i AIG. Jednocześnie Portfel B ma niższą ekspozycję na AIG (6,67%) i C (8,73%) – spółki z największymi stratami w tym okresie. Potwierdza to niższy najgorszy dzień Portfela B (−6,22% vs −7,47%).

Warto jednak zaznaczyć, że różnice są **małe** (0,1–0,3 pp przy VaR historycznym) ze względu na wysokie korelacje między spółkami sektora finansowego w warunkach kryzysu – dywersyfikacja wewnątrzsektorowa jest ograniczona niezależnie od wag.

#### Metoda historyczna vs. parametryczna – która daje wyższy VaR?

We wszystkich czterech przypadkach **VaR parametryczny jest wyższy** niż historyczny, przy czym różnica rośnie wraz z poziomem ufności (od ~0,3–0,4 pp przy 95% do ~1,1 pp przy 99%).

Przyczyną jest **dodatnia skośność** obu portfeli w analizowanym oknie (0,93 i 1,02). Okno 2007–2008 zawierało nie tylko silne spadki, ale i gwałtowne odbicia rynkowe (marzec 2008 po uratowaniu Bear Stearns, lipiec 2008), które przesuwają empiryczny rozkład w prawo. Model gaussowski zakłada symetrię – przez co jego symetryczny lewy ogon jest grubszy niż faktycznie obserwowany, generując wyższy VaR parametryczny, szczególnie na poziomie 99%.

Kurtoza obu portfeli (~2,8–2,9, nadmiarowa ≈ 0) jest zbliżona do rozkładu normalnego – ogony empiryczne nie są grubsze od gaussowskich, co dodatkowo tłumaczy brak efektu „grubych ogonów" zwiększającego VaR historyczny.

**Ogólny wniosek:** Dobór metody ma istotne znaczenie. W tym konkretnym oknie kryzysowym model parametryczny **przeszacowuje** ryzyko ogonowe ze względu na asymetrię rozkładu. W innych warunkach (silna ujemna skośność, wysoka kurtoza) zależność byłaby odwrotna. Stanowi to argument za stosowaniem obydwu metod równolegle jako wzajemnego sprawdzianu.

---
*Źródła danych: Yahoo Finance. Teoria: Market Risk Lab – prezentacja „Miary at Risk" (kwiecień 2026).*
