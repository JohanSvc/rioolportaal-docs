# Planning en export

#rioolportaal #domein

Terug naar [[Overzicht]]

De planningsmodule omvat twee lagen: **HL-planning** (week-niveau) en **detailplanning** (concrete datum). Daarna volgt de export-fase na validatie.

---

## Planningsstappen

| Stap | Wie | Status voor → na | Route |
|---|---|---|---|
| HL-week instellen (bulk) | Planner, AannemerWerfleider | Doorgegeven → HLGepland | `POST /planning/hl` |
| Plandatum instellen (bulk) | Planner, AannemerWerfleider | * → Gepland | `POST /planning/detail` |
| Toewijzen | Planner, AannemerWerfleider | Gepland → Toegewezen | `POST /planning/toewijzen` |
| Intrekken | Planner, AannemerWerfleider | Toegewezen → Gepland | `POST /planning/intrekken` |

Bulkacties: meerdere opdracht-id's via `ids[]` in de POST-body.

---

## `Planningsweken`-tabel

Bevat jaar + weeknummer (ISO-week). Bij opstart/seeding worden de huidige week + 25 weken vooruit aangemaakt. Gekoppeld aan `Opdrachten.planningsweek_id`.

---

## Planning-view (`/planning`)

Toont twee tabbladen:
1. **Planning** — opdrachten in statussen `Doorgegeven`, `HLGepland`, `Gepland`, `Toegewezen` (= te plannen)
2. **In & Export** — exporteerbare opdrachten (`Doorgegeven` t/m `OnderhandenGenomen`) met CSV-download

Toegang: Planner, AannemerWerfleider

---

## Planning-export (CSV)

`GET /planning/export` — download CSV met geplande opdrachten (status `Gepland`, `Toegewezen`, `HLGepland`).

Kolommen: `OpdrachtCode`, `Adres`, `Gemeente`, `OpdrachtType`, `Aannemer`, `PlanningsWeek`, `PlanningsDatum`, `Status`

BOM-prefix (`﻿`) voor correcte Excel-weergave.

---

## Uitvoeringsverslagen-export (CSV)

`GET /planning/export-uv` — uitvoeringsverslagen per opdracht inclusief versienummer en verslag-tekst.

Toegang: Planner, Administrator

---

## Exports na validatie

Na goedkeuring door de Werfleider (`POST /validaties/:id/goedkeuren`) gaat de opdracht naar status `Te exporteren`. Daarna kunnen de volgende exports worden gegenereerd:

| Export | Route | Inhoud |
|---|---|---|
| Meetstaten | `GET /exports/meetstaten` | Service-items + aantallen + prijzen |
| Wegenis | `GET /exports/wegenis` | Verhardingstypes + oppervlakten |
| As-Built/Bemating | `GET /exports/bemating` | 5 maatvoeringswaarden |

Alle exports bieden een **Excel-download** via `GET /exports/:type/download` (ExcelJS).

Filters: `van`, `tot` (op plandatum), `aannemer_id`, `type_id` (voor meetstaten).

Enkel secties met status `OK` of `Afgehandeld` worden meegenomen.

---

## Zie ook

- [[Opdrachten]]
- [[Secties]]
- [[Rapportage]]
