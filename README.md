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
