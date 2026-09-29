## Scop

Transformă comportamentul analizat în Laboratorul 1 în ținte măsurabile
de calitate, într-o estimare a volumului de citiri pentru trei niveluri
și într-o estimare a stocării datelor despre piață.

# 1. Cerințe de calitate

## Ipoteza privind piața

Pentru prima versiune a Personal Investment Dashboard este considerată
piața Nasdaq Helsinki.

Pentru acțiuni, programul normal de tranzacționare este 10:00–18:30, ora
locală Helsinki, ceea ce reprezintă 8,5 ore de tranzacționare pe zi.

Pentru estimările pe o perioadă de 30 de zile se presupun 22 de zile de
tranzacționare. Aceeași piață va fi utilizată ulterior pentru estimarea
numărului de Stocks și a stocării.

------------------------------------------------------------------------

## 1.1 Latența citirilor

**Măsură:** latența observată de Utilizator, măsurată din momentul în
care Utilizatorul trimite cererea până în momentul în care rezultatul
corect poate fi utilizat.

**Țintă:** p95 ≤ 2 secunde.

**Condiție de operare:** la volumul de lucru țintă, inclusiv în perioada
aglomerată de la deschiderea pieței, pentru citirile Overview, Filter,
Stock price, History, Watchlist și Search.

O citire finalizată în exact 2 secunde respectă ținta. O citire care
depășește 2 secunde nu respectă ținta de latență.

Pentru restul de 5% dintre citiri trebuie definit un timeout; depășirea
limitei de latență nu trebuie să transforme un rezultat lipsă sau
incorect într-un rezultat valid.

------------------------------------------------------------------------

## 1.2 Disponibilitate bazată pe timpul de funcționare

Disponibilitatea este măsurată separat pentru orele de tranzacționare și
pentru restul zilei.

### În timpul orelor de tranzacționare

**Măsură:** proporția din timpul măsurat în care citirile Dashboard-ului
pot returna un rezultat acceptabil.

**Țintă:** 99.99% disponibilitate.

**Condiție de operare:** în timpul programului Nasdaq Helsinki,
10:00–18:30, în cele 22 de zile de tranzacționare presupuse într-o
perioadă de 30 de zile.

Timpul total măsurat:

22 zile × 8,5 ore = 187 ore

Bugetul de indisponibilitate:

187 ore × (1 - 0,9999) = 0,0187 ore ≈ 1,12 minute

Prin urmare, pentru atingerea unei disponibilități de 99.99%,
Dashboard-ul poate fi indisponibil aproximativ **1,12 minute** în timpul
orelor de tranzacționare într-o perioadă de 30 de zile.

### În afara orelor de tranzacționare

**Măsură:** proporția din timpul măsurat în care citirile Dashboard-ului
pot returna un rezultat acceptabil.

**Țintă:** 99.9% disponibilitate.

**Condiție de operare:** toate celelalte ore din aceeași perioadă de 30
de zile.

Timpul total al perioadei:

30 zile × 24 ore = 720 ore

Timpul din afara orelor de tranzacționare:

720 - 187 = 533 ore

Bugetul de indisponibilitate:

533 ore × (1 - 0,999) = 0,533 ore ≈ 31,98 minute

Prin urmare, pentru atingerea unei disponibilități de 99.9%,
Dashboard-ul poate fi indisponibil aproximativ **31,98 minute** în afara
orelor de tranzacționare într-o perioadă de 30 de zile.

Ținta este mai ridicată în timpul orelor de tranzacționare deoarece
informațiile despre piață sunt mai sensibile la timp atunci când piața
este activă.

------------------------------------------------------------------------

## 1.3 Consistența Stock price

**Măsură:** vechimea Stock price, calculată ca diferența dintre momentul
curent și timestamp-ul furnizat de Market Data Provider.

**Țintă:** un Stock price cu o vechime de cel mult 15 minute este
considerat în limita normală a întârzierii și este afișat împreună cu
timestamp-ul furnizorului.

Un Stock price cu o vechime mai mare de 15 minute, dar de cel mult 30 de
minute, poate fi afișat numai dacă este marcat vizibil drept
„întârziat”.

Un Stock price cu o vechime mai mare de 30 de minute nu mai este
considerat acceptabil și Dashboard-ul returnează „indisponibil”.

Limita de 30 de minute este o ipoteză de produs aleasă ca fiind de două
ori întârzierea așteptată de aproximativ 15 minute.

**Condiție de operare:** pentru citirile Stock price bazate pe datele
sincronizate de la Market Data Provider.

Dashboard-ul nu afișează zero pentru un preț lipsă și nu prezintă un
preț întârziat drept preț curent.

------------------------------------------------------------------------

## 1.4 Consistența Watchlist

**Măsură:** rezultatul următoarei citiri a Watchlist-ului efectuate de
același Utilizator după confirmarea unei modificări.

**Țintă:** următoarea citire a Watchlist-ului trebuie să reflecte
modificarea confirmată a aceluiași Utilizator.

**Condiție de operare:** după finalizarea și confirmarea unei adăugări,
eliminări sau alte modificări acceptate asupra Watchlist-ului privat al
Utilizatorului.

------------------------------------------------------------------------

## 1.5 Debit

**Măsură:** numărul de citiri acceptabile finalizate de Dashboard pe
secundă, în timp ce cerințele de latență, disponibilitate și consistență
continuă să fie respectate.

**Țintă:** în perioada de deschidere a pieței, Dashboard-ul trebuie să
poată finaliza cel puțin:

- 103 RPS pentru 300 de Utilizatori concurenți;
- 1.030 RPS pentru 3.000 de Utilizatori concurenți;
- 10.296 RPS pentru 30.000 de Utilizatori concurenți.

**Condiție de operare:** traficul de la deschiderea pieței definit în
estimarea de volum, inclusiv traficul stabil, fluxurile suplimentare
Overview și Watchlist și marja de capacitate de 10%.

O cerere doar pornită nu este considerată debit finalizat. Pentru
această cerință sunt numărate numai citirile acceptabile finalizate.

# 2. Estimări RPS în regim stabil

Pentru fiecare tip de citire se folosește formula:

`RPS = Utilizatori concurenți × proporția participanților × acțiuni per Utilizator / secunde`

Valorile intermediare nu sunt rotunjite.

| Citire                    |                 300 Utilizatori |                3.000 Utilizatori |                 30.000 Utilizatori |
|---------------------------|--------------------------------:|---------------------------------:|-----------------------------------:|
| Overview                  |       `300 × 0,70 / 30 = 7 RPS` |     `3.000 × 0,70 / 30 = 70 RPS` |     `30.000 × 0,70 / 30 = 700 RPS` |
| Filter                    | `300 × 0,50 × 3 / 60 = 7,5 RPS` | `3.000 × 0,50 × 3 / 60 = 75 RPS` | `30.000 × 0,50 × 3 / 60 = 750 RPS` |
| Stock price               |       `300 × 0,20 / 1 = 60 RPS` |     `3.000 × 0,20 / 1 = 600 RPS` |    `30.000 × 0,20 / 1 = 6.000 RPS` |
| History                   |    `300 × 0,20 / 300 = 0,2 RPS` |     `3.000 × 0,20 / 300 = 2 RPS` |     `30.000 × 0,20 / 300 = 20 RPS` |
| Watchlist                 |       `300 × 0,60 / 60 = 3 RPS` |     `3.000 × 0,60 / 60 = 30 RPS` |     `30.000 × 0,60 / 60 = 300 RPS` |
| Search                    | `300 × 0,10 × 3 / 60 = 1,5 RPS` | `3.000 × 0,10 × 3 / 60 = 15 RPS` | `30.000 × 0,10 × 3 / 60 = 150 RPS` |
| **Total în regim stabil** |                    **79,2 RPS** |                      **792 RPS** |                      **7.920 RPS** |

## 3. Estimări RPS la deschiderea pieței

La deschiderea pieței:

- 30% dintre Utilizatori reîmprospătează Overview o dată într-un
  interval de 10 secunde;
- 60% din acest grup reîmprospătează și Watchlist;
- traficul suplimentar este adăugat peste traficul stabil;
- după subtotal se aplică o marjă de capacitate de 10%;
- numai ținta finală este rotunjită în sus.

## Calcule

### Flux suplimentar Overview

Formula:

`Utilizatori × 0,30 / 10 secunde`

- 300 Utilizatori: `300 × 0,30 / 10 = 9 RPS`
- 3.000 Utilizatori: `3.000 × 0,30 / 10 = 90 RPS`
- 30.000 Utilizatori: `30.000 × 0,30 / 10 = 900 RPS`

### Flux suplimentar Watchlist

60% din grupul de 30% reîmprospătează și Watchlist.

Formula:

`Utilizatori × 0,30 × 0,60 / 10 secunde`

- 300 Utilizatori: `300 × 0,30 × 0,60 / 10 = 5,4 RPS`
- 3.000 Utilizatori: `3.000 × 0,30 × 0,60 / 10 = 54 RPS`
- 30.000 Utilizatori: `30.000 × 0,30 × 0,60 / 10 = 540 RPS`

## Tabelul deschiderii pieței

| Calcul la deschiderea pieței                     | 300 Utilizatori | 3.000 Utilizatori | 30.000 Utilizatori |
|--------------------------------------------------|----------------:|------------------:|-------------------:|
| Trafic stabil de citire                          |        79,2 RPS |           792 RPS |          7.920 RPS |
| Flux suplimentar de reîmprospătare Overview      |           9 RPS |            90 RPS |            900 RPS |
| Flux suplimentar de reîmprospătare Watchlist     |         5,4 RPS |            54 RPS |            540 RPS |
| **Subtotal la deschiderea pieței**               |    **93,6 RPS** |       **936 RPS** |      **9.360 RPS** |
| Marjă de capacitate de 10%                       |        9,36 RPS |          93,6 RPS |            936 RPS |
| **Țintă rotunjită în sus la deschiderea pieței** |     **103 RPS** |     **1.030 RPS** |     **10.296 RPS** |

## Verificarea țintei finale

### 300 Utilizatori

`93,6 × 1,10 = 102,96 RPS`

Rotunjit în sus:

**103 RPS**

### 3.000 Utilizatori

`936 × 1,10 = 1.029,6 RPS`

Rotunjit în sus:

**1.030 RPS**

### 30.000 Utilizatori

`9.360 × 1,10 = 10.296 RPS`

Valoarea este deja întreagă:

**10.296 RPS**

# 4. Estimări de stocare

## 4.1 Definirea unui Stock

Nasdaq definește un Stock ca reprezentând o participație de proprietate
într-o corporație, exprimată prin acțiuni.

Pentru prima versiune a Personal Investment Dashboard, un **Stock** este
definit ca seria principală de acțiuni ordinare a unei companii active
listate pe **Nasdaq Helsinki Main Market**.

Prima versiune:

- acceptă Nasdaq Helsinki Main Market;
- include acțiunile ordinare/common shares;
- include o singură serie principală de acțiuni pentru fiecare companie;
- exclude preferred shares, ADR-uri, ETF-uri, funds și derivatives;
- exclude Nasdaq First North;
- nu păstrează ca disponibile instrumentele inactive sau delistate.

Pagina oficială Nasdaq Helsinki raportează **136 companii listate pe
Main Market**, statistică publicată pentru Q4 2025. Pagina a fost
observată la 29.09.2026. Deoarece numărul listărilor se poate modifica,
cele 136 de companii sunt folosite drept estimare pentru acest
laborator.

### Surse

- [Nasdaq — Stock definition](https://www.nasdaq.com/glossary/s/stock)
- [Nasdaq
  Helsinki](https://www.nasdaq.com/products/european-markets/helsinki)

------------------------------------------------------------------------

## 4.2 Datele sincronizate

Dashboard-ul stochează numai datele de la Market Data Provider necesare
fluxurilor definite pentru prima versiune.

| Set de date             | Date stocate                                                       | Nevoie a produsului                                           | Frecvență de sincronizare                                        |
|-------------------------|--------------------------------------------------------------------|---------------------------------------------------------------|------------------------------------------------------------------|
| Date de referință Stock | symbol, numele companiei, ISIN, exchange, currency, sector, active | Search, Filter, Overview și identificarea Stock-ului          | O dată pe zi înainte de deschiderea pieței                       |
| Cel mai recent preț     | symbol, price, provider timestamp, received timestamp, status      | Overview, Stock detail și afișarea vechimii prețului          | La fiecare 15 minute în timpul orelor de tranzacționare          |
| Price history           | symbol, provider timestamp, price                                  | History și analiza evoluției prețului                         | Un punct la fiecare 15 minute în timpul orelor de tranzacționare |
| Alte date               | —                                                                  | Nu sunt necesare alte date ale furnizorului în prima versiune | —                                                                |

Watchlist-ul privat nu este inclus în această estimare deoarece
reprezintă starea Utilizatorului, nu date de piață sincronizate de la
Market Data Provider.

Price history este păstrat timp de **un an**.

Nasdaq Helsinki tranzacționează în mod normal 8,5 ore pe zi. La un
interval de 15 minute:

`8,5 ore × 60 minute / 15 minute = 34 puncte / Stock / zi`

Pentru estimarea anuală se folosește calendarul Nasdaq Helsinki din
2026. Anul 2026 are 261 zile lucrătoare de luni până vineri, iar
calendarul Nasdaq indică 10 zile lucrătoare în care piața Helsinki este
închisă.

`261 - 10 = 251 zile de tranzacționare`

------------------------------------------------------------------------

## 4.3 Estimarea octeților per înregistrare

Pentru estimare au fost create înregistrări JSON reprezentative,
codificate UTF-8. Deoarece toate caracterele din exemplele de mai jos
sunt caractere ASCII, fiecare caracter ocupă 1 byte în UTF-8.
Dimensiunea fiecărei înregistrări poate fi astfel obținută prin
însumarea dimensiunilor câmpurilor, separatorilor și acoladelor JSON.

### Date de referință

``` json
{"symbol":"ABC","name":"Example Oyj","isin":"FI0000000000","exchange":"Nasdaq Helsinki","currency":"EUR","sector":"Industrials","active":true}
```

| Componentă                     | Dimensiune |
|--------------------------------|-----------:|
| `"symbol":"ABC"`               |       14 B |
| `"name":"Example Oyj"`         |       20 B |
| `"isin":"FI0000000000"`        |       21 B |
| `"exchange":"Nasdaq Helsinki"` |       28 B |
| `"currency":"EUR"`             |       16 B |
| `"sector":"Industrials"`       |       22 B |
| `"active":true`                |       13 B |
| 6 separatori `,`               |        6 B |
| Acoladele `{}`                 |        2 B |
| **Total**                      |  **142 B** |

Calcul:

`14 + 20 + 21 + 28 + 16 + 22 + 13 + 6 + 2 = 142 B`

Pentru estimarea stocării, valoarea este rotunjită în sus la **150 bytes
/ record**.

### Cel mai recent preț

``` json
{"symbol":"ABC","price":12.34,"providerTimestamp":"2026-09-29T12:00:00+03:00","receivedAt":"2026-09-29T12:15:00+03:00","status":"delayed"}
```

| Componentă                                        | Dimensiune |
|---------------------------------------------------|-----------:|
| `"symbol":"ABC"`                                  |       14 B |
| `"price":12.34`                                   |       13 B |
| `"providerTimestamp":"2026-09-29T12:00:00+03:00"` |       47 B |
| `"receivedAt":"2026-09-29T12:15:00+03:00"`        |       40 B |
| `"status":"delayed"`                              |       18 B |
| 4 separatori `,`                                  |        4 B |
| Acoladele `{}`                                    |        2 B |
| **Total**                                         |  **138 B** |

Calcul:

`14 + 13 + 47 + 40 + 18 + 4 + 2 = 138 B`

Pentru estimarea stocării, valoarea este rotunjită în sus la **150 bytes
/ record**.

### Price history

``` json
{"symbol":"ABC","providerTimestamp":"2026-09-29T12:00:00+03:00","price":12.34}
```

| Componentă                                        | Dimensiune |
|---------------------------------------------------|-----------:|
| `"symbol":"ABC"`                                  |       14 B |
| `"providerTimestamp":"2026-09-29T12:00:00+03:00"` |       47 B |
| `"price":12.34`                                   |       13 B |
| 2 separatori `,`                                  |        2 B |
| Acoladele `{}`                                    |        2 B |
| **Total**                                         |   **78 B** |

Calcul:

`14 + 47 + 13 + 2 + 2 = 78 B`

Pentru estimarea stocării, valoarea este rotunjită în sus la **80 bytes
/ record**.

------------------------------------------------------------------------

## 4.4 Calculul stocării brute

Formula:

`raw storage = record count × average bytes per record`

Pentru price history:

`history record count = supported Stocks × history points per Stock per day × retained trading days`

### Price history

Înregistrări într-o zi:

`136 × 34 = 4.624 records / trading day`

Înregistrări într-un an:

`136 × 34 × 251 = 1.160.624 records`

| Set de date              | Decizia despre produs și perioada de păstrare                            | Calculul numărului de înregistrări | Octeți per înregistrare |                             Stocare brută |
|--------------------------|--------------------------------------------------------------------------|-----------------------------------:|------------------------:|------------------------------------------:|
| Date de referință Stock  | 136 Stocks active                                                        |                              `136` |                   150 B |                                `20.400 B` |
| Cele mai recente prețuri | Un singur preț curent per Stock; valoarea este înlocuită la sincronizare |                              `136` |                   150 B |                                `20.400 B` |
| Price history            | Interval de 15 minute, păstrat timp de un an                             |       `136 × 34 × 251 = 1.160.624` |                    80 B |                            `92.849.920 B` |
| Alte date selectate      | Nu sunt stocate separat în V1                                            |                                `0` |                       — |                                     `0 B` |
| **Total după un an**     |                                                                          |                                    |                         | **92.890.720 B ≈ 88,59 MiB (≈ 92,89 MB)** |

------------------------------------------------------------------------

## 4.5 Stocare inițială, zilnică și anuală

### Stocare inițială

Date de referință și latest price:

`20.400 + 20.400 = 40.800 B`

`40.800 B ≈ 39,84 KiB`

### Creștere într-o zi de tranzacționare

Price history:

`4.624 × 80 = 369.920 B / zi`

`369.920 B ≈ 361,25 KiB / zi de tranzacționare`

### Stocare după un an

`92.849.920 B history + 40.800 B date curente`

`= 92.890.720 B`

`≈ 88,59 MiB (≈ 92,89 MB)`

Estimarea reprezintă numai date brute. Nu include indexuri, replici,
backup-uri, log-uri, metadata internă sau alte costuri de stocare.

# 5. Posibile blocaje

Un volum mare de RPS nu demonstrează singur existența unui blocaj.
Valorile calculate sunt folosite pentru a identifica punctele care
trebuie investigate și măsurate.

| Calitate        | Posibil blocaj                                                | Dovezi din acest laborator                                                                                                                                                                                                                     | Efect posibil                                                                                                                                                     | Ce trebuie măsurat în continuare                                                                                                                            |
|-----------------|---------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Latență         | Fluxul de citire Stock price                                  | La 30.000 de Utilizatori, Stock price generează aproximativ 6.000 RPS în regim stabil, iar cerința de latență este p95 ≤ 2 secunde.                                                                                                            | Volumul mare de citiri poate crește timpul de răspuns și poate face ca p95 să depășească 2 secunde.                                                               | Latența p50 și p95 pentru Stock price la 6.000 RPS și în timpul țintei de capacitate de la deschiderea pieței.                                              |
| Consistență     | Sincronizarea datelor de piață și actualizarea Watchlist-ului | Stock price este acceptat până la 15 minute, poate fi afișat ca întârziat până la 30 de minute și devine indisponibil peste 30 de minute; următoarea citire a Watchlist-ului trebuie să reflecte modificarea confirmată de același Utilizator. | Datele sincronizate prea lent pot fi afișate drept curente deși sunt învechite, iar o modificare Watchlist poate să nu fie vizibilă imediat la următoarea citire. | Vechimea reală a datelor față de timestamp-ul furnizorului și timpul dintre confirmarea unei modificări Watchlist și vizibilitatea ei la următoarea citire. |
| Debit           | Calea totală de citire, în special Stock price                | Traficul stabil ajunge la 7.920 RPS pentru 30.000 de Utilizatori, iar ținta de capacitate la deschiderea pieței este 10.296 RPS.                                                                                                               | Dashboard-ul poate să nu finalizeze suficiente citiri acceptabile pe secundă, chiar dacă cererile sunt primite.                                                   | Numărul de citiri acceptabile finalizate pe secundă în teste de încărcare la 7.920 RPS și 10.296 RPS, menținând simultan țintele de latență și consistență. |
| Disponibilitate | Market Data Provider                                          | Stock price, History și alte citiri depind de datele sincronizate de la sistemul extern. Dacă datele depășesc limita de vechime, ele trebuie marcate întârziate sau indisponibile.                                                             | O întrerupere prelungită a furnizorului poate face ca prețurile să devină învechite sau indisponibile în timpul orelor de tranzacționare.                         | Disponibilitatea furnizorului, durata întreruperilor, timpul de la ultima sincronizare validă și proporția citirilor care ajung la rezultat indisponibil.   |

## 5.1 Latență

Fluxul Stock price este un posibil punct de presiune pentru latență. La
nivelul de 30.000 de Utilizatori, acesta produce:

`30.000 × 0,20 × 1 / 1 = 6.000 RPS`

Aceasta este cea mai mare contribuție individuală la traficul stabil. În
același timp, cerința de calitate cere ca p95 al citirilor să fie de cel
mult 2 secunde.

Valoarea de 6.000 RPS nu demonstrează că Stock price este un blocaj, dar
justifică măsurarea latenței acestui flux printr-un test de încărcare.

------------------------------------------------------------------------

## 5.2 Consistență

Există două puncte de presiune relevante.

Pentru Stock price, Dashboard-ul trebuie să compare momentul curent cu
timestamp-ul Market Data Provider. Un preț cu o vechime de cel mult 15
minute este acceptat, unul cu o vechime mai mare de 15 minute și de cel
mult 30 de minute poate fi afișat numai ca „întârziat”, iar peste 30 de
minute rezultatul trebuie să devină „indisponibil”. O sincronizare
întârziată poate conduce la afișarea unor date vechi sau indisponibile.

Pentru Watchlist, după confirmarea unei modificări, următoarea citire
efectuată de același Utilizator trebuie să reflecte modificarea. Trebuie
măsurat timpul dintre confirmarea modificării și momentul în care
aceasta devine vizibilă.

------------------------------------------------------------------------

## 5.3 Debit

La 30.000 de Utilizatori, traficul stabil total este:

`7.920 RPS`

La deschiderea pieței:

`7.920 + 900 + 540 = 9.360 RPS`

După aplicarea marjei de capacitate de 10%:

`9.360 × 1,10 = 10.296 RPS`

Prin urmare, calea totală de citire este un posibil punct de presiune
pentru debit.

Ținta nu este doar ca Dashboard-ul să primească 10.296 cereri pe
secundă, ci să finalizeze citiri acceptabile, păstrând în același timp
țintele de latență, consistență și disponibilitate.

------------------------------------------------------------------------

## 5.4 Disponibilitate

Market Data Provider este un posibil punct de presiune pentru
disponibilitate deoarece mai multe fluxuri ale Dashboard-ului depind de
datele furnizate de acesta.

Dacă sincronizarea cu furnizorul este întreruptă, ultima valoare
disponibilă poate deveni mai veche decât limita acceptată. În acest caz
Dashboard-ul trebuie să prezinte rezultatul drept întârziat sau
indisponibil, nu să inventeze un preț curent.

Pentru validarea acestei ipoteze trebuie măsurate:

- disponibilitatea Market Data Provider;
- durata întreruperilor;
- vechimea datelor în timpul unei întreruperi;
- proporția citirilor care ajung la rezultat indisponibil.
