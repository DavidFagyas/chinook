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

![ERD Diagram](./images/erd_diagram.png)


