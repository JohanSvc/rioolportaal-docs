# VS002 — Mockup bugfix-sweep (`erik-werk`) + demo-deploy

#rioolportaal #verbeterlog #VS002

**Status:** Afgehandeld (fixes) + Lopend (demo-omgeving)
**Aangemaakt:** 2026-06-29
**Auteur:** Erik (met Claude Code)
**Branch:** `erik-werk` in [JohanSvc/rioolportaal](https://github.com/JohanSvc/rioolportaal) (38 commits vóór op `main`; `main` blijft Johans referentie)

Terug naar [[Overzicht]] · [[Backlog]]

---

## Focus

Johans mockup (`mockup/`, Express + EJS) draaiend en demonstreerbaar maken op een **Mac mini / Azure SQL Edge** i.p.v. Windows / SQL Server Express, en bij het pagina-per-pagina nakijken de schermen weer werkend maken. Aanleiding: de SQLite→MSSQL/schema-v2-port was scherm-per-scherm gebeurd en niet overal afgemaakt, en er was geen foutafhandeling waardoor één fout de hele server neerhaalde.

> Volledige redenering per fix staat in de commit-berichten + `mockup/WIJZIGINGEN-erik-werk.md` (canoniek levend logboek) en `BUGS-VOOR-JOHAN.md` in de code-repo.

## Inzichten / terugkerende oorzaken

1. **Engine-verschil** SQL Server → Azure SQL Edge: `FORMAT()` is CLR-afhankelijk en staat uit op SQL Edge → query faalt/hangt → `CONVERT(char(7),date,126)`.
2. **Onafgemaakte port** naar schema-v2: queries lazen nog oude inline-kolommen (bv. `geo_lat` i.p.v. `Geotag`-join).
3. **Route ↔ view niet aangesloten**: view verwacht locals die de route niet meegeeft, of een route ontbreekt volledig.
4. **Verkeerde/ontbrekende view-namen.**
5. **Onvolledige rol-takken** in views.
6. **Geen foutafhandeling**: één onafgevangen fout legt op Express 4 de hele server plat.
7. **Implementatie ≠ domein-spec**: gating week af van `rioolportaal-docs` (Planner mocht niet plannen — zie [[Planning en export]]).

## Resultaat — 16 gefixte bugs

Aangemaakt + gesloten als GitHub-issues met fix-commit: [gesloten, label `gefixt`](https://github.com/JohanSvc/rioolportaal/issues?q=is%3Aissue+is%3Aclosed+label%3Agefixt) — **#22 t/m #37**. Kort:

- Crash-vangnet (`unhandledRejection`/`uncaughtException`) — één kapot scherm legde de hele server plat.
- Kaart: oude `geo_lat`-kolommen + veldnaam-mismatch (lege plattegrond + geen markers).
- Route↔view-locals: raamcontracten, beheer, dashboard-grafiek, rapportage.
- Verkeerde/ontbrekende view-namen (admin/ter-validatie); 5 routes renderden onbestaande views.
- Rol-takken: Planning blanco voor Administrator; Planner kon niet plannen (uitgelijnd op de spec).
- Planning-acties: 5 ontbrekende routes (404) + `/toewijzen` verkeerd contract/gate (403).
- `FORMAT()` → `CONVERT` (Azure SQL Edge); admin settings-crash + lege tarieven/nodes.

Geverifieerd: volledige sweep alle hoofdschermen × 6 rollen → enkel 200/403, geen 500/404.

## Openstaande restpunten

8 open issues (label `mockup`) — zie [[Backlog]] en [open issues](https://github.com/JohanSvc/rioolportaal/issues?q=is%3Aissue+is%3Aopen+label%3Amockup): **#38–#45** (planning-filters zonder WHERE, FORMAT()-rest, geen per-route foutafhandeling, twee db-generaties, opdrachttypes-CRUD, rapportage-indiener-aanname, UC-020-doc, opdrachten-filterpaneel).

## Demo-omgeving (Mac mini)

| Onderdeel | Detail |
|---|---|
| Host | Mac mini (`Mac-mini-van-Erik.local`) — zelfde altijd-aan host als de andere ProcesLab-services |
| App | `mockup/server.js` op **:3004**, env-gedreven (`MSSQL_HOST`/`MSSQL_USER`/`MSSQL_PASSWORD`/`MSSQL_DB`/`PORT`) |
| DB | Docker-container `riool-mssql` (Azure SQL Edge, :1433), restart-policy `unless-stopped` |
| Persistentie | LaunchAgent `com.erikborghgraef.agent-os.rioolportaal` (RunAtLoad + KeepAlive) |
| Publiek | cloudflared-tunnel → **https://engineering-leidingen-lab.proceslab.com** (achter Cloudflare Access) |
| Demodata | echte, geanonimiseerde aannemersdata (~5958 opdrachten, 2022-2026) — pipeline in `mockup/` |

> Niet-brekend voor Johan: `db-mssql.js` is env-gedreven gemaakt — zónder `MSSQL_HOST` valt het terug op Johans Windows-ODBC. Zie [[Stack en DB-laag]].

## Zie ook

- [[Backlog]]
- [[Stack en DB-laag]]
- [[Planning en export]]
- [[Overzicht]]
