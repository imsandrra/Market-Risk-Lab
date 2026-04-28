# Raport: Wycena opcji europejskich – Black-Scholes vs Monte Carlo

**Autor:** Oleksandra Krykun  
**Data:** 28 kwietnia 2026  
**Kurs:** Market Risk Lab – Zadanie Domowe 3

---

## Cel

Wycena dwóch europejskich opcji (call i put) na akcje Goldman Sachs (GS) przy użyciu wzoru Blacka-Scholesa oraz metody Monte Carlo (10 000 i 50 000 ścieżek), a następnie porównanie wyników obu metod.

---

## 1. Dane i parametry

**Źródło danych:** Yahoo Finance (`yfinance`), dzienne ceny zamknięcia skorygowane o dywidendy i splity (`Adj Close`).  
**Okres:** 1 sierpnia 2006 – 30 lipca 2008 (503 dni sesyjne) – ten sam co w Zadaniach 1 i 2.  
**Akcja bazowa:** Goldman Sachs (GS).

Wszystkie parametry wyceny wyznaczono bezpośrednio z danych historycznych:

- **S₀** – ostatnia cena zamknięcia GS w oknie (30.07.2008): **$137,66**
- **σ** – historyczna zmienność roczna z log-stóp zwrotu całego okresu: **36,82%**
- **K** – strike ATM, zaokrąglony do $5: **$140,00** (dla call i put)
- **T** – termin wygaśnięcia: **0,5 roku (6 miesięcy)**
- **r** – stopa wolna od ryzyka: **1,50%** (rentowność 6M US T-Bill, połowa 2008 r. — środowisko agresywnych obniżek stóp Fed po kryzysie Bear Stearns)

| Parametr | Symbol | Wartość |
|----------|--------|---------|
| Cena aktywa bazowego | S₀ | $137,66 |
| Strike (call i put) | K | $140,00 |
| Czas do wygaśnięcia | T | 0,5 roku |
| Stopa wolna od ryzyka | r | 1,50% |
| Zmienność historyczna | σ | 36,82% |

Obie opcje są nieznacznie **out-of-the-money**: cena spot ($137,66) jest poniżej strike'u ($140,00), co oznacza, że call jest OTM, a put jest ITM — co dobrze odzwierciedla kontekst rynkowy połowy 2008 r., gdy akcje GS znajdowały się pod presją, tracąc od szczytu (~$250 w końcu 2007 r.) ponad 40%.

![Wykres: Cena GS i log-stopy zwrotu w oknie analizy (2006–2008)](gs_dane_historyczne.png)

Wykres potwierdza gwałtowny wzrost zmienności po sierpniu 2007 (sygnał BNP Paribas) — skupianie dużych wahań dziennych jest wyraźnie widoczne od Q4 2007. Historyczna zmienność 36,82% jest prawie dwukrotnie wyższa od typowych poziomów sprzed kryzysu (~20%), co bezpośrednio przekłada się na wysokie premie opcyjne.

---

## 2. Wycena Black-Scholes

Wzór analityczny dla europejskiej opcji call i put w modelu Blacka-Scholesa:

$$d_1 = \frac{\ln(S_0/K)+(r+\sigma^2/2)\,T}{\sigma\sqrt{T}}, \qquad d_2 = d_1 - \sigma\sqrt{T}$$

$$C = S_0\,N(d_1) - K\,e^{-rT}\,N(d_2), \qquad P = K\,e^{-rT}\,N(-d_2) - S_0\,N(-d_1)$$

### Wartości pośrednie

| Wielkość | Wartość |
|----------|---------|
| $d_1$ | $+0{,}094145$ |
| $d_2$ | $-0{,}166201$ |
| $N(d_1)$ | $0{,}537503$ |
| $N(d_2)$ | $0{,}433999$ |
| $N(-d_1)$ | $0{,}462497$ |
| $N(-d_2)$ | $0{,}566001$ |

Wartość $d_1 \approx 0{,}094$ wskazuje, że opcja call jest blisko pieniądza — $N(d_1) \approx 0{,}54$, czyli call-delta wynosi ok. 54%, co odpowiada sytuacji lekko OTM przy wysokiej zmienności. Wartość $d_2 \approx -0{,}166$ jest ujemna, co oznacza, że ryzykowo-neutralne prawdopodobieństwo wygaśnięcia call w pieniądzu ($N(d_2) \approx 43\%$) jest poniżej 50%.

### Wyniki

| Typ opcji | Cena Black-Scholes |
|-----------|--------------------|
| **CALL** | **$13,6850** |
| **PUT** | **$14,9822** |

Wyższa cena put niż call jest zgodna z parytetem put-call: opcja put jest głębiej ITM ($S_0 < K$). Weryfikacja parytetu put-call:

$$C - P = S_0 - K \cdot e^{-rT} \implies 13{,}6850 - 14{,}9822 = 137{,}66 - 140 \cdot e^{-0{,}015 \cdot 0{,}5}$$

Błąd numeryczny: $-7{,}11 \times 10^{-15}$ ✓ — parytety spełniony dokładnie.

---

## 3. Wycena Monte Carlo

Symulujemy $N$ realizacji końcowej ceny aktywa według geometrycznego ruchu Browna (GBM):

$$S_T = S_0 \cdot \exp\!\left[\left(r - \frac{\sigma^2}{2}\right)T + \sigma\sqrt{T}\,Z\right], \quad Z\sim\mathcal{N}(0,1)$$

Zdyskontowana średnia wypłat daje estymator ceny:

$$\hat{C}^{MC} = e^{-rT} \cdot \frac{1}{N}\sum_{i=1}^{N}\max(S_T^{(i)}-K,\,0), \qquad \hat{P}^{MC} = e^{-rT} \cdot \frac{1}{N}\sum_{i=1}^{N}\max(K-S_T^{(i)},\,0)$$

Zastosowano **antitetyczne zmienne losowe** — dla każdego wylosowanego $Z$ uwzględniono równocześnie $-Z$, co redukuje wariancję estymatora bez zwiększania liczby wywołań generatora liczb losowych.

### Wyniki

| Typ | N ścieżek | Cena MC | Błąd std. SE |
|-----|-----------|---------|--------------|
| CALL | 10 000 | $13,6056 | ±$0,2398 |
| CALL | 50 000 | $13,6690 | ±$0,1074 |
| PUT | 10 000 | $14,9354 | ±$0,1841 |
| PUT | 50 000 | $14,9746 | ±$0,0825 |

Przy N = 50 000 błąd standardowy spada do ok. $0,10 (call) i $0,08 (put), co odpowiada przedziałowi ufności ±$0,21 i ±$0,17 na poziomie 2σ — czyli cena MC mieści się bardzo blisko wyniku analitycznego BS.

![Wykres: Rozkład cen końcowych GS i wypłat opcji (N = 50 000)](mc_rozklad.png)

Na powyższym wykresie widoczny jest rozkład log-normalny cen końcowych $S_T$ (lewy panel), asymetryczny i z wyraźnie grubym prawym ogonem ze względu na wysoką zmienność (σ = 36,82%). Strike K = $140 leży nieznacznie powyżej S₀ = $137,66, dzieląc rozkład na część OTM (lewo od K — strefa wypłat dla put) i ITM (prawo od K — strefa wypłat dla call). Prawy panel pokazuje rozkłady wypłat warunkowych (tylko ścieżki kończące się ITM) — obydwa mają długi prawy ogon typowy dla opcji europejskich.

---

## 4. Porównanie wyników

| Typ | N ścieżek | BS [$] | MC [$] | SE MC [$] | Różnica [$] | Różnica [%] |
|-----|-----------|--------|--------|-----------|-------------|-------------|
| CALL | 10 000 | 13,6850 | 13,6056 | ±0,2398 | −0,0794 | −0,5800% |
| CALL | 50 000 | 13,6850 | 13,6690 | ±0,1074 | −0,0160 | −0,1168% |
| PUT | 10 000 | 14,9822 | 14,9354 | ±0,1841 | −0,0468 | −0,3126% |
| PUT | 50 000 | 14,9822 | 14,9746 | ±0,0825 | −0,0076 | −0,0509% |

![Wykres: Zbieżność estymatora MC do ceny BS (skala logarytmiczna)](zbieznosc_mc.png)

Wykres zbieżności ilustruje, jak estymator MC (wraz z przedziałem ±2σ) zbiega do poziomej linii BS wraz ze wzrostem N. Przy N = 100 oscylacje są duże; przy N = 10 000 przedział ±2σ jest już wąski; przy N = 50 000 estymator jest praktycznie stabilny.

---

## 5. Wnioski

### 5.1 Czy wyniki BS i Monte Carlo są podobne?

**Tak — wyniki są bardzo zbliżone.** Różnica między ceną BS a MC wynosi:

- dla **CALL**: $0,08 przy N = 10 000 (−0,58%) i jedynie $0,02 przy N = 50 000 (−0,12%),
- dla **PUT**: $0,05 przy N = 10 000 (−0,31%) i $0,01 przy N = 50 000 (−0,05%).

Wynik jest zgodny z oczekiwaniami: oba modele opierają się na identycznym założeniu (geometryczny ruch Browna), tyle że Black-Scholes dostarcza rozwiązania w postaci zamkniętego wzoru analitycznego, natomiast Monte Carlo aproksymuje tę samą wartość numerycznie przez uśrednianie zdyskontowanych wypłat po dużej liczbie scenariuszy.

### 5.2 Jak liczba ścieżek wpływa na dokładność?

Błąd standardowy estymatora MC maleje odwrotnie proporcjonalnie do pierwiastka z liczby ścieżek:

$$\text{SE}(N) \propto \frac{1}{\sqrt{N}}$$

Zwiększenie liczby ścieżek z 10 000 do 50 000 (5-krotnie) redukuje błąd standardowy o czynnik $\sqrt{5} \approx 2{,}24$ — co potwierdzają wyniki: SE dla call spada z $0,2398 do $0,1074 (stosunek: 2,23×), a dla put z $0,1841 do $0,0825 (stosunek: 2,23×). Zbieżność jest wyraźnie widoczna na wykresie w skali logarytmicznej.

### 5.3 Kontekst rynkowy — wpływ kryzysu na wycenę

Wysoka zmienność historyczna GS (σ = 36,82%) jest bezpośrednią konsekwencją kryzysu finansowego analizowanego w Zadaniach 1 i 2. Dla porównania: typowa zmienność implied vol dla blue chipów w normalnych warunkach rynkowych wynosi 15–20%. Wzrost σ przekłada się liniowo na wzrost obu premii opcyjnych — dlatego premie rzędu $13–15 przy cenie akcji $138 (stosunek premii do ceny ~10%) są wysokie w porównaniu do typowych rynków ustabilizowanych.

| Kryterium | Black-Scholes | Monte Carlo (N = 10 000) | Monte Carlo (N = 50 000) |
|-----------|:---:|:---:|:---:|
| Szybkość obliczenia | natychmiastowa | szybka | wolniejsza |
| Dokładność (w modelu GBM) | dokładna | dobra (SE ~$0,20) | bardzo dobra (SE ~$0,09) |
| Elastyczność modelu | ograniczona | bardzo wysoka | bardzo wysoka |
| Obsługa egzotycznych payoffów | nie | tak | tak |

Black-Scholes jest optymalny dla standardowych opcji europejskich w modelu GBM — wynik dokładny w ułamku sekundy. Monte Carlo zyskuje przewagę dla opcji egzotycznych (azjatyckich, barierowych, lookback), modeli ze stochastyczną zmiennością (Heston, SABR) oraz gdy wypłata zależy od całej ścieżki cenowej — czyli w przypadkach, gdzie zamknięte rozwiązanie analityczne nie istnieje.

---

*Źródła danych: Yahoo Finance. Teoria: Market Risk Lab – prezentacja „Derivatives" (kwiecień 2026).*
