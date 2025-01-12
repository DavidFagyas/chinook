# 1. Úvod a Popis Zdrojových Dát

Cieľom semestrálneho projektu je analyzovať dáta týkajúce sa hudby, používateľov a ich nákupného správania. Táto analýza umožňuje identifikovať trendy v hudobných preferenciách používateľov, najpopulárnejšie skladby a albumy, a správanie používateľov pri nákupoch.

Zdrojové dáta pochádzajú z **Chinook databázy**, ktorá obsahuje informácie o hudobných umelcoch, albumoch, skladbách a zákazníkoch. Hlavné tabuľky v tejto databáze sú:

- **Artist** – Obsahuje informácie o hudobných umelcoch.
- **Album** – Ukladá informácie o albumoch a ich priradení k umelcom.
- **Track** – Ukladá informácie o skladbách, vrátane názvu, albumu, typu média, žánru a ceny.
- **Genre** – Obsahuje informácie o hudobných žánroch.
- **MediaType** – Ukladá informácie o typoch médií, ako je napríklad MP3 alebo WMA.
- **Invoice** – Ukladá fakturačné údaje, vrátane informácií o zákazníkoch a dátumoch nákupov.
- **InvoiceLine** – Ukladá informácie o jednotlivých položkách faktúr, ako sú skladby a ich cena.
- **Customer** – Ukladá údaje o zákazníkoch, vrátane kontaktov a preferencií.
- **Employee** – Ukladá informácie o zamestnancoch, ktorí podporujú zákazníkov.

Cieľom projektu je vybudovať multidimenzionálny dátový model, ktorý umožní analýzu správania používateľov v oblasti nákupov hudby, preferencií umelcov a skladieb, a tiež poskytne vizualizácie kľúčových metrík.

## 1.1 Dátová Architektúra

### ERD Diagram

Surové dáta sú usporiadané v relačnom modeli, ktorý je znázornený na entitno-relačnom diagrame (ERD). Tento diagram vizualizuje vzťahy medzi rôznymi entitami, ako sú umelci, albumy, skladby, žánre, zákazníci a faktúry, ktoré sú uložené v rôznych tabuľkách databázy.

Relačný model zobrazuje, ako sú tieto entity prepojené cez primárne a cudzie kľúče, čím umožňuje efektívne vyhľadávanie, manipuláciu a analýzu dát. ERD diagram je základným nástrojom na pochopenie štruktúry databázy a vzorcov dát, ktoré sa používajú na analytické účely.

<p align="center">
  <img src="https://github.com/DavidFagyas/chinook/blob/main/Mysql%20chinook.png" alt="ERD Schema">
  <br>
  <em>Obrázok 1 Entitno-relačná schéma Tracks</em>
</p>

## 2 Dimenzionálny model

Pre analýzu bol navrhnutý hviezdicový model (star schema), kde centrálnym bodom je faktová tabuľka fact_ratings, ktorá je prepojená s nasledujúcimi dimenziami:

dim_albums: Obsahuje informácie o albumoch, ako sú idAlbums a Title. Táto dimenzia poskytuje kontext pre analýzu skladieb na základe ich albumu.

dim_genres: Obsahuje informácie o hudobných žánroch, ako sú idGenres a Name. Pomáha pri kategorizácii skladieb podľa žánrov.

dim_media_types: Obsahuje informácie o typoch médií, ako sú idMediaTypes a Name. Táto dimenzia je dôležitá pre analýzu, aký typ médií (napr. MP3, WAV) bol použitý pri skladbách.

dim_invoices: Obsahuje fakturačné údaje, ako sú idInvoice, InvoiceDate, BillingAddress, BillingCity, BillingCountry, Total. Táto dimenzia sa používa na analýzu transakcií zákazníkov.

dim_customers: Obsahuje demografické údaje o zákazníkoch, ktoré sú získané prostredníctvom faktúr (Invoice). Zahrňuje údaje ako idCustomers, FirstName, LastName, Phone, Address, Country. Pomáha pri analýze správania zákazníkov na základe ich nákupov.

Tento hviezdicový model je znázornený v diagrame nižšie. Diagram ukazuje, ako sú jednotlivé dimenzie prepojené s faktovou tabuľkou, čo zjednodušuje pochopenie vzťahov medzi dátami a implementáciu modelu.ie. Diagram ukazuje, ako sú jednotlivé dimenzie prepojené s faktovou tabuľkou, čo zjednodušuje pochopenie vzťahov medzi dátami a implementáciu modelu.

<p align="center">
  <img src="https://github.com/DavidFagyas/chinook/blob/main/MYsql%20star.png" alt="ERD Schema">
  <br>
  <em>Obrázok 2 Dimenzionálny model  Tracks</em>
</p>

## 3. ETL Proces v Snowflake

ETL proces zahŕňa tri hlavné fázy: extrahovanie (Extract), transformácia (Transform) a načítanie (Load). Tento proces bol implementovaný v Snowflake s cieľom pripraviť dáta zo zdrojového datasetu a transformovať ich do viacdimenzionálneho modelu vhodného na analýzu a vizualizáciu.

#### 3.1 Extract (Extrahovanie dát)

Dáta zo zdrojového datasetu vo formáte .csv boli najprv importované do Snowflake cez interné stage úložisko, ktoré bolo nazvané `my_stage`. Stage v Snowflake slúži ako dočasné úložisko na importovanie alebo exportovanie dát. Pre vytvorenie stage sa použil nasledujúci príkaz:

```sql
CREATE OR REPLACE STAGE my_stage;
```
Do stage boli následne nahraté súbory obsahujúce údaje o albumoch, skladbách, žánroch, zákazníkoch a faktúrach. Dáta boli importované do staging tabuliek pomocou príkazu COPY INTO. Pre každú tabuľku sa použil podobný príkaz:
```sql
COPY INTO albums_staging
FROM @my_stage/albums.csv
FILE_FORMAT = (TYPE = 'CSV' SKIP_HEADER = 1);
```

3.2 Transform (Transformácia dát)
V tejto fáze boli dáta zo staging tabuliek vyčistené, transformované a obohatené. Hlavným cieľom bolo pripraviť dimenzie a faktovú tabuľku, ktoré umožnia jednoduchú a efektívnu analýzu.

Dimenzia dim_tracks
Dimenzia dim_tracks obsahuje údaje o skladbách, ako sú názov, dĺžka skladby a album, ku ktorému patrí. Táto dimenzia je typu SCD 0, pretože údaje o skladbách sa považujú za nemenné.

```sql
Kód másolása
CREATE TABLE dim_tracks AS
SELECT DISTINCT
    t.track_id AS dim_track_id,
    t.name AS track_name,
    t.duration AS track_duration,
    a.album_id AS dim_album_id
FROM tracks_staging t
JOIN albums_staging a ON t.album_id = a.album_id;
```
Dimenzia dim_genres
Dimenzia dim_genres uchováva informácie o žánroch, ku ktorým patrí každá skladba. Tento atribút je nemenný, preto je dimenzia typu SCD 0.

```sql
CREATE TABLE dim_genres AS
SELECT DISTINCT
    g.genre_id AS dim_genre_id,
    g.name AS genre_name
FROM genres_staging g;
Dimenzia dim_customers
```
Dimenzia dim_customers obsahuje údaje o zákazníkoch, ako sú ich meno, adresa a krajina. Tieto informácie sú považované za nemenné, preto je táto dimenzia typu SCD 0.
```sql
CREATE TABLE dim_customers AS
SELECT DISTINCT
    c.customer_id AS dim_customer_id,
    c.first_name AS customer_first_name,
    c.last_name AS customer_last_name,
    c.address AS customer_address,
    c.country AS customer_country
FROM customers_staging c;
```
Dimenzia dim_invoices
Dimenzia dim_invoices uchováva informácie o faktúrach, ako je dátum vystavenia faktúry a celková suma. Táto dimenzia je tiež typu SCD 0.

```sql
CREATE TABLE dim_invoices AS
SELECT DISTINCT
    i.invoice_id AS dim_invoice_id,
    i.invoice_date AS invoice_date,
    i.billing_address AS billing_address,
    i.billing_city AS billing_city,
    i.billing_country AS billing_country,
    i.total AS total_amount
FROM invoices_staging i;
```
Faktová tabuľka fact_sales
Faktová tabuľka fact_sales obsahuje záznamy o predajoch, spojené s dimenziami ako skladba, žáner, zákazník a faktúra.

```sql
CREATE TABLE fact_sales AS
SELECT 
    s.sale_id AS fact_sale_id,
    s.sale_date AS sale_date,
    s.amount AS sale_amount,
    t.dim_track_id AS track_id,
    g.dim_genre_id AS genre_id,
    c.dim_customer_id AS customer_id,
    i.dim_invoice_id AS invoice_id
FROM sales_staging s
JOIN dim_tracks t ON s.track_id = t.dim_track_id
JOIN dim_genres g ON s.genre_id = g.dim_genre_id
JOIN dim_customers c ON s.customer_id = c.dim_customer_id
JOIN dim_invoices i ON s.invoice_id = i.dim_invoice_id;
```
3.3 Load (Načítanie dát)
Po úspešnom vytvorení dimenzií a faktovej tabuľky boli dáta nahraté do finálnej štruktúry. Na záver boli staging tabuľky odstránené, aby sa optimalizovalo využitie úložiska:


```sql
DROP TABLE IF EXISTS tracks_staging;
DROP TABLE IF EXISTS albums_staging;
DROP TABLE IF EXISTS genres_staging;
DROP TABLE IF EXISTS customers_staging;
DROP TABLE IF EXISTS invoices_staging;
DROP TABLE IF EXISTS sales_staging;
```
ETL proces v Snowflake umožnil spracovanie pôvodných dát do viacdimenzionálneho modelu typu hviezda. Tento proces zahŕňal čistenie, obohacovanie a reorganizáciu údajov. Výsledný model umožňuje analýzu predajov skladieb, správania zákazníkov a poskytuje základ pre vizualizácie a reporty.

## 4. Vizualizácia dát
Dashboard obsahuje 6 vizualizácií, ktoré poskytujú základný prehľad o kľúčových metrikách a trendoch týkajúcich sa skladieb, používateľov a hodnotení. Tieto vizualizácie odpovedajú na dôležité otázky a umožňujú lepšie pochopiť správanie používateľov a ich preferencie

### 1. Top 5 najviac ziskových skladieb
Táto vizualizácia zobrazuje počet hodnotení kníh, ktoré boli pridelené v jednotlivých rokoch. Umožňuje sledovať vývoj počtu hodnotení v priebehu času a identifikovať roky s najvyšším nárastom aktivity používateľov.

Táto vizualizácia zobrazuje top 5 skladieb s najvyšším celkovým príjmom, t.j. najziskovejšie skladby podľa ceny za jednotku (UnitPrice). Tento dotaz zobrazuje názvy skladieb a ich celkový výnos, zoradené podľa výnosu zostupne.


```sql
SELECT TrackId, Name, SUM(UnitPrice) AS TotalRevenue
FROM tracks
GROUP BY TrackId, Name
ORDER BY TotalRevenue DESC
LIMIT 5;
```
<p align="center">
  <img src="https://github.com/DavidFagyas/chinook/blob/main/first.png" alt="ERD Schema">
  <br>

</p>
### 2. Top 5 najpopulárnejších albumov
Táto vizualizácia zobrazuje top 5 albumov s najvyšším počtom predaných skladieb. Tento dotaz identifikuje albumy, ktoré boli najviac predávané, na základe počtu predaných jednotiek (Quantity).

```sql

SELECT Albums.AlbumId, Albums.Title, SUM(InvoiceItems.Quantity) AS TotalQuantity
FROM albums
JOIN invoice_items ON albums.AlbumId = invoice_items.AlbumId
GROUP BY Albums.AlbumId, Albums.Title
ORDER BY TotalQuantity DESC
LIMIT 5;
```

<p align="center">
  <img src="https://github.com/DavidFagyas/chinook/blob/main/Track%20.png" alt="ERD Schema">
  <br>
  
  ## 3. Top 5 najpredávanejších žánrov
Táto vizualizácia ukazuje top 5 najpredávanejších žánrov na základe celkových tržieb (TotalRevenue). Tento dotaz identifikuje najobľúbenejšie žánre podľa tržieb.

```sql

SELECT Genres.Name AS Genre, SUM(InvoiceItems.Quantity * InvoiceItems.UnitPrice) AS TotalRevenue
FROM genres
JOIN tracks ON genres.GenreId = tracks.GenreId
JOIN invoice_items ON tracks.TrackId = invoice_items.TrackId
GROUP BY Genres.Name
ORDER BY TotalRevenue DESC
LIMIT 5;
```
<p align="center">
  <img src="https://github.com/DavidFagyas/chinook/blob/main/number.png" alt="ERD Schema">
  <br>
## 4. Top 5 najaktívnejších zákazníkov
Táto vizualizácia zobrazuje top 5 najaktívnejších zákazníkov podľa celkového počtu nákupov. Tento dotaz identifikuje zákazníkov, ktorí najviac nakupovali skladby a albumy.

```sql

SELECT Customers.FirstName, Customers.LastName, COUNT(InvoiceItems.InvoiceId) AS TotalPurchases
FROM customers
JOIN invoices ON customers.CustomerId = invoices.CustomerId
JOIN invoice_items ON invoices.InvoiceId = invoice_items.InvoiceId
GROUP BY Customers.FirstName, Customers.LastName
ORDER BY TotalPurchases DESC
LIMIT 5;
```
<p align="center">
  <img src="https://github.com/DavidFagyas/chinook/blob/main/avg%20long.png" alt="ERD Schema">
  <br>
## 5. Tržby podľa krajiny
Táto vizualizácia ukazuje tržby (TotalRevenue) rozdelené podľa krajiny. Tento dotaz pomáha získať prehľad o tom, z ktorých krajín prichádzajú najväčšie tržby.

```sql
SELECT Customers.Country, SUM(InvoiceItems.Quantity * InvoiceItems.UnitPrice) AS TotalRevenue
FROM customers
JOIN invoices ON customers.CustomerId = invoices.CustomerId
JOIN invoice_items ON invoices.InvoiceId = invoice_items.InvoiceId
GROUP BY Customers.Country
ORDER BY TotalRevenue DESC;
```
<p align="center">
  <img src="https://github.com/DavidFagyas/chinook/blob/main/expensive.png" alt="ERD Schema">
  <br>

## 6. Tržby podľa dátumu
Táto vizualizácia zobrazuje tržby (TotalRevenue) podľa dátumu nákupu, čo pomáha analyzovať sezónne trendy a identifikovať najpredávanejšie dni.

```sql
SELECT CAST(invoices.InvoiceDate AS DATE) AS Date, SUM(InvoiceItems.Quantity * InvoiceItems.UnitPrice) AS TotalRevenue
FROM invoices
JOIN invoice_items ON invoices.InvoiceId = invoice_items.InvoiceId
GROUP BY CAST(invoices.InvoiceDate AS DATE)
ORDER BY Date;
```
<p align="center">
  <img src="https://github.com/DavidFagyas/chinook/blob/main/genres%20types.png" alt="ERD Schema">
  <br>

**Autor:** Dávid Fagyas
