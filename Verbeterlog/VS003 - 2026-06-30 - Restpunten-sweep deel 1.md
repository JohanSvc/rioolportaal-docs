# VS003 — Restpunten-sweep deel 1 (`erik-werk`)

#rioolportaal #verbeterlog #VS003

Terug naar [[Overzicht]] · [[Backlog]]

---

_2026-06-30 — vervolg op [[VS002 - 2026-06-29 - Mockup bugfix-sweep en demo-deploy]]._

## Doel

Drie technische restpunten uit de VS002-backlog wegwerken die zonder beslissing van Johan oppakbaar waren: de laatste `FORMAT()`-rest (#39), per-route foutafhandeling (#40) en de overtollige SQLite-DB-generatie (#41).

## Wat is aangepast

| Issue | Probleem | Oplossing |
|---|---|---|
| [#39](https://github.com/JohanSvc/rioolportaal/issues/39) | `FORMAT()` (CLR-afhankelijk) stond nog in de verificatie-queries van `seed-adm.js` en `seed-chart.js` → faalt op Azure SQL Edge | 4× omgezet naar `CONVERT(char(7),date,126)`. Geen `FORMAT()` meer in de codebase. |
| [#40](https://github.com/JohanSvc/rioolportaal/issues/40) | Express 4 stuurt rejected promises uit async-handlers niet naar de error-middleware → een onafgevangen fout liet de request hangen | `express-async-errors` (globale patch, dekt alle `api/*`-routers zonder per-route wrapper) + centrale error-middleware in `server.js` die een nette 500 toont. |
| [#41](https://github.com/JohanSvc/rioolportaal/issues/41) | `db.js` (SQLite, oud Meetstaat-model, dropte tabellen bij start) stond nog naast `db-mssql.js` | `db.js` verwijderd (−422 r.). `better-sqlite3` blijft als dependency: `qea_query.js` leest er het EA-model (`.qea`) mee. |

## Verificatie

- **#39:** `CONVERT(char(7),GETDATE(),126)` geeft `2026-06`; `FORMAT()` faalt op Edge met *"Common Language Runtime (CLR) is not enabled"*.
- **#40:** async-route die gooit → HTTP 500 met nette foutpagina, geen hang/timeout.
- **#41:** geen `./db`-referentie meer; server boot schoon, alle hoofdroutes 302/200.

## Commits (`erik-werk`)

- `bd00178` chore(mockup): oude SQLite db.js verwijderen (#41)
- `75e9a52` fix(mockup): laatste FORMAT() → CONVERT op Azure SQL Edge (#39)
- `9e502a9` fix(mockup): centrale foutafhandeling + async-error-doorgifte (#40)

## Nog open

In de [[Backlog]] blijven staan: #38 (planning-`WHERE` — keuze server-side vs gs-grid), #42 (opdrachttypes-CRUD — keuze read-only vs CRUD), #43 (rapportage-'indiener' — vraag aan Johan), #44 (UC-020-doc), #45 (opdrachten-filterpaneel — UX-keuze).

## Zie ook

- [[VS002 - 2026-06-29 - Mockup bugfix-sweep en demo-deploy]]
- [[Stack en DB-laag]]
- [[Backlog]]
