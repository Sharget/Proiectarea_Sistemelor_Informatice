# Laboratorul 3 — Desenează limita Dashboard-ului

## Scop și continuitate

Proiectul continuă **Personal Investment Dashboard** din Laboratoarele 1 și 2. În V1, sistemul urmărește acțiuni de pe **Nasdaq Helsinki Main Market**, cu prețuri furnizate de un **Market Data Provider** extern. Nu permite tranzacții financiare. Utilizatorul autentificat poate căuta, filtra și urmări Stocks, prețuri, istoric, propriile investiții și un Watchlist privat.

Din Laboratorul 2 păstrăm: **p95 ≤ 2 s** pentru citirile corecte în perioadele aglomerate; o țintă estimată de capacitate de **10.296 RPS** pentru 30.000 de utilizatori la deschiderea pieței; prețuri de **cel mult 15 min acceptate**, **peste 15 și cel mult 30 min etichetate ca întârziate**, respectiv **peste 30 min indisponibile**. Sincronizarea este o ipoteză de V1: datele de referință zilnic, prețurile și istoricul la 15 minute în timpul tranzacționării.

**Notă de prezentare:** Diagramele folosesc etichete scurte pentru lizibilitate; tabelele descriu responsabilitățile în detaliu. Pentru a adapta automat lățimea diagramelor în Obsidian, se poate activa fișierul CSS oferit separat.

## 1. Diagrama System Context

```mermaid
flowchart TB
    investor["Investitor"]
    dashboard["Personal Investment Dashboard"]
    provider["Market Data Provider"]
    investor -->|"Consulta si gestioneaza date"| dashboard
    dashboard -->|"Solicita date de piata"| provider
    provider -->|"Date sau erori"| dashboard
```

**Explicație:** System Context arată întregul Dashboard ca **un singur sistem** și interacțiunile lui cu Investitorul și cu furnizorul extern; nu prezintă părți interne de execuție.

## 2. Diagrama Container

| Container | Responsabilitate |
| --- | --- |
| Dashboard Web App | Interfața utilizatorului și afișarea rezultatelor |
| Dashboard Application | Verifică identitatea, procesează citirile și modifică datele private |
| Dashboard Database | Stochează conturi, investiții și Watchlist-uri private |
| Market Data Collector | Sincronizează și validează datele furnizorului în fundal |
| Market Data Store | Stochează informațiile de piață acceptate și progresul sincronizării |

```mermaid
flowchart TB
    investor["Investitor"]
    provider["Market Data Provider"]
    subgraph dashboard["Personal Investment Dashboard"]
        direction TB
        web["Dashboard Web App"]
        app["Dashboard Application"]
        db[("Dashboard Database")]
        store[("Market Data Store")]
        collector["Market Data Collector"]
        web -->|"Cereri si raspunsuri"| app
        app -->|"Date private"| db
        app -->|"Citeste date de piata"| store
        collector -->|"Scrie date validate"| store
    end
    investor -->|"Foloseste"| web
    collector -->|"Solicita date"| provider
    provider -->|"Date sau erori"| collector
```

**Decizie de arhitectură:** Sincronizarea este periodică, în fundal. O citire Stock price nu trimite o nouă cerere direct către Market Data Provider. Cele două stocări reprezintă responsabilități logice diferite, fără alegerea obligatorie a două tehnologii.

## 3. Diagrama Component — Dashboard Application

| Componentă | Responsabilitate |
| --- | --- |
| AuthService | Identifică utilizatorul și verifică sesiunea |
| StockService | Search, Filter, Stock price, History și verificarea Stocks acceptate |
| WatchlistService | Acces autorizat și modificări pentru Watchlist-ul privat |
| InvestmentService | Active și cantități înregistrate de investitor |
| DashboardService | Combină datele necesare pentru Overview |

```mermaid
flowchart TB
    web["Dashboard Web App"]
    db[("Dashboard Database")]
    store[("Market Data Store")]
    subgraph app["Dashboard Application"]
        direction TB
        auth["AuthService"]
        stock["StockService"]
        watch["WatchlistService"]
        inv["InvestmentService"]
        overview["DashboardService"]
    end
    web -->|"Sesiune"| auth
    web -->|"Citiri Stocks"| stock
    web -->|"Watchlist"| watch
    web -->|"Investitii"| inv
    web -->|"Overview"| overview
    stock -->|"Identitate"| auth
    watch -->|"Identitate"| auth
    inv -->|"Identitate"| auth
    overview -->|"Identitate"| auth
    auth -->|"Verifica contul"| db
    stock -->|"Citeste piata"| store
    watch -->|"Citeste/scrie lista"| db
    inv -->|"Citeste/scrie investitii"| db
    watch -->|"Valideaza Stock"| stock
    overview -->|"Investitii"| inv
    overview -->|"Preturi"| stock
    overview -->|"Watchlist"| watch
```

**Limită:** numai `Dashboard Application` este deschisă în componente. `Market Data Collector` este în continuare un container separat, deci nu apare ca o componentă a acestei aplicații.

## 4A. Secvență — Sincronizează datele de piață

```mermaid
sequenceDiagram
    participant Collector as Market Data Collector
    participant Provider as Market Data Provider
    participant Store as Market Data Store
    participant App as Dashboard Application

    Collector->>Collector: Porneste sincronizarea periodica
    Collector->>Store: Citeste progresul salvat
    Store-->>Collector: Ultima pozitie cunoscuta
    Collector->>Provider: Cere date cu credentile de server

    alt Lot valid
        Provider-->>Collector: Date si timestamp
        Collector->>Collector: Valideaza Stock pret si timestamp
        Collector->>Store: Salveaza atomic date si progres
        Store-->>Collector: Confirmare commit
    else Lot invalid
        Provider-->>Collector: Date cu erori
        Collector->>Collector: Respinge lotul
    else Rate limit
        Provider-->>Collector: Limitare si Retry-After
        Collector->>Collector: Amana urmatoarea incercare
    else Timeout
        Collector->>Collector: Timeout fara date utilizabile
        Collector->>Collector: Programeaza reincercarea
    end

    App->>Store: Citeste date acceptate ulterior
    Store-->>App: Date si timestamp sau eroare
    App->>App: Verifica vechimea conform Lab 2
```

**Ipoteze inițiale:**

- Market Data Collector sincronizează datele independent de cererile utilizatorilor: referințele Stock zilnic înainte de deschidere, latest price și history la 15 minute în timpul tranzacționării.
- Credențialele Market Data Provider rămân în Market Data Collector, pe server, niciodată în Dashboard Web App.
- Datele sunt validate înainte de scriere; un preț negativ ori un timp invalid este respins. Lipsa unui preț nu este echivalentă cu zero.
- Datele și progresul sincronizării sunt confirmate împreună, atomic. O salvare neconfirmată nu este raportată drept succes; o reîncercare trebuie să fie sigură împotriva dublurilor.
- Dacă furnizorul nu poate fi sincronizat, datele acceptate anterior nu se șterg. Citirile le pot afișa numai conform regulilor de vechime din Lab 2. Dacă salvarea datelor eșuează sau este neconfirmată, nu se declară sincronizarea reușită și nu se avansează progresul fără confirmare.

## 4B. Secvență — Citește prețul unui Stock

```mermaid
sequenceDiagram
    actor Investor as Investitor
    participant Web as Dashboard Web App
    participant Stock as StockService
    participant Auth as AuthService
    participant Store as Market Data Store

    Investor->>Web: Cere pret Stock
    Web->>Stock: Citeste pret (sesiune, simbol)
    Stock->>Auth: Verifica identitatea
    alt Identitate invalida
        Auth-->>Stock: Refuz
        Stock-->>Web: Neautentificat
        Web-->>Investor: Cere autentificare
    else Identitate valida
        Auth-->>Stock: Identitate confirmata
        Stock->>Stock: Valideaza simbolul
        alt Format invalid
            Stock-->>Web: Simbol invalid
            Web-->>Investor: Afiseaza eroare
        else Format valid
            Stock->>Store: Citeste Stock si pret
            alt Eroare stocare
                Store-->>Stock: Eroare
                Stock-->>Web: Eroare temporara
                Web-->>Investor: Nu poate afisa pretul
            else Stock nesuportat
                Store-->>Stock: Stock neacceptat
                Stock-->>Web: Stock nesuportat
                Web-->>Investor: Afiseaza mesaj
            else Pret lipsa
                Store-->>Stock: Fara pret
                Stock-->>Web: Indisponibil
                Web-->>Investor: Afiseaza indisponibil
            else Pret gasit
                Store-->>Stock: Pret si timestamp
                Stock->>Stock: Calculeaza vechimea
                alt Vechime maxim 15 minute
                    Stock-->>Web: Pret acceptat si timestamp
                    Web-->>Investor: Afiseaza pret
                else Peste 15 si maxim 30 minute
                    Stock-->>Web: Pret marcat intarziat
                    Web-->>Investor: Afiseaza intarziat
                else Peste 30 minute
                    Stock-->>Web: Indisponibil
                    Web-->>Investor: Afiseaza indisponibil
                end
            end
        end
    end
    Note over Stock,Store: Fara apel nou la Market Data Provider
```

**Ipoteze inițiale:**

- Identitatea este verificată înainte de citirea datelor; simbolul primit este validat, apoi se verifică dacă Stock este acceptat de V1 (Nasdaq Helsinki Main Market).
- Datele provin din Market Data Store, alimentat periodic de Market Data Collector; **citirea nu apelează furnizorul**.
- Vechimea rezultă din timestamp-ul furnizorului: ≤15 min acceptat; >15 min până la 30 min etichetat „întârziat”; >30 min „indisponibil”.
- Un preț lipsă nu este zero. O eroare de stocare nu este echivalentă cu lipsa prețului.

## 4C. Secvență — Modifică un Watchlist privat

```mermaid
sequenceDiagram
    actor Investor as Investitor
    participant Web as Dashboard Web App
    participant Watch as WatchlistService
    participant Auth as AuthService
    participant Stock as StockService
    participant DB as Dashboard Database

    Investor->>Web: Adauga Stock in Watchlist
    Web->>Watch: Adauga (sesiune, lista, simbol)
    Watch->>Auth: Verifica identitatea
    alt Identitate invalida
        Auth-->>Watch: Refuz
        Watch-->>Web: Neautentificat
        Web-->>Investor: Cere autentificare
    else Identitate valida
        Auth-->>Watch: userId verificat
        Watch->>Watch: Valideaza intrarea
        alt Intrare invalida
            Watch-->>Web: Cerere invalida
            Web-->>Investor: Afiseaza eroare
        else Intrare valida
            Watch->>DB: Verifica proprietarul listei
            alt Eroare citire DB
                DB-->>Watch: Eroare
                Watch-->>Web: Operatie indisponibila
                Web-->>Investor: Afiseaza eroare
            else Nu este proprietar
                DB-->>Watch: Proprietar diferit
                Watch-->>Web: Acces interzis
                Web-->>Investor: Modificare respinsa
            else Este proprietar
                DB-->>Watch: Proprietar confirmat
                Watch->>Stock: Verifica Stock acceptat
                alt Stock invalid sau verificare indisponibila
                    Stock-->>Watch: Refuz sau eroare
                    Watch-->>Web: Modificare respinsa
                    Web-->>Investor: Afiseaza problema
                else Stock acceptat
                    Stock-->>Watch: Stock valid
                    Watch->>DB: Salveaza numai daca ownerId corespunde
                    alt Salvare esuata sau incerta
                        DB-->>Watch: Eroare sau confirmare lipsa
                        Watch-->>Web: Succes neconfirmat
                        Web-->>Investor: Afiseaza eroare
                    else Salvare confirmata
                        DB-->>Watch: Commit confirmat
                        Watch-->>Web: Succes
                        Web-->>Investor: Stock adaugat
                        opt Urmatoarea citire a aceluiasi utilizator
                            Investor->>Web: Citeste Watchlist
                            Web->>Watch: Citeste (sesiune, lista)
                            Watch->>Auth: Verifica identitatea
                            Auth-->>Watch: Acelasi userId
                            Watch->>DB: Citeste lista pentru userId
                            DB-->>Watch: Lista actualizata
                            Watch-->>Web: Lista actualizata
                            Web-->>Investor: Arata Stock adaugat
                        end
                    end
                end
            end
        end
    end
```

**Ipoteze inițiale:**

- Utilizatorul solicită adăugarea unui Stock în propriul Watchlist privat. Identitatea este stabilită de AuthService.
- WatchlistService verifică operația, datele de intrare și **proprietarul Watchlist-ului** înainte de scriere. Pentru adăugare, cere StockService să verifice dacă acțiunea este acceptată de V1.
- Dashboard Database păstrează Watchlist-urile private; scrierea impune din nou proprietarul verificat.
- Un rezultat de salvare lipsă sau nesigur **nu este succes**. Succesul se comunică numai după confirmarea stocării.
- După o modificare confirmată, următoarea citire reușită a aceluiași utilizator trebuie să reflecte modificarea (read-your-writes).

## Verificare finală

- [x] O diagramă System Context, una Container și una Component.
- [x] Trei diagrame de secvență: sincronizare, citire Stock price, modificare Watchlist.
- [x] Confirmă afișarea tuturor celor șase blocuri Mermaid în Obsidian (sintaxa a fost verificată separat).
- [x] Component deschide numai `Dashboard Application`.
- [x] Sunt incluse rezultatele normale, acțiunile respinse și defectările cerute.
- [x] Sunt acoperite citirile, datele private, sincronizarea furnizorului și regulile de consistență din Lab 2.

**Notă:** Cele 10.296 RPS reprezintă o **țintă estimată**, nu capacitate demonstrată. Aceasta trebuie validată ulterior prin teste de încărcare.
