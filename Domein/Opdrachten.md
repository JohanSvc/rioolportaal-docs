# Opdrachten

#rioolportaal #domein

Terug naar [[Overzicht]]

De opdracht is de kern van het Rioolportaal. Elke opdracht is een rioolwerkitem met een adres, een aannemer, een type en een status.

---

## DB-tabel: `Opdrachten`

| Kolom | Type | Beschrijving |
|---|---|---|
| `code` | NVARCHAR | Unieke code (bv. `RP-2026-001`) |
| `opdrachttype_id` | FK → OpdrachtTypes | Type (HA, KH, IP, …) |
| `titel` | NVARCHAR | Leesbare naam |
| `adres / straat / huisnr / postcode / gemeente` | NVARCHAR | Locatie |
| `aannemer_id` | FK → Aannemers | Uitvoerende aannemer |
| `status` | NVARCHAR | Huidige stap in de lifecycle |
| `planningsweek_id` | FK → Planningsweken | Week van uitvoering |
| `plannings_datum` | NVARCHAR | Concrete datum |
| `geotag_id` | FK → Geotag | GPS-coördinaat (geocoded) |

---

## Status-lifecycle

```
Doorgegeven
  ↓  (Planner plant HL)
HLGepland
  ↓  (Planner stelt weekplanning op)
Gepland
  ↓  (Aannemer start werk)
OnderhandenGenomen
  ↓  (Aannemer markeert uitgevoerd)
Uitgevoerd
  ↓  (Werfleider valideert alle secties)
Ter validatie
  ↓  (Planner klaarmaakt voor export)
Te exporteren
  ↓  (Export voltooid)
Afgehandeld
```

Statusovergangen worden gelogd in `OpdrachtStatusLog` (gebruiker + tijdstip).

---

## Importeren

Opdrachten kunnen batch worden geïmporteerd via een Excel-bestand (`.xlsx`).

**Verplichte kolommen:** `OpdrachtCode`, `OpdrachtType`, `Titel`, `Beschrijving`, `Gemeente`, `Postcode`, `Straat`, `Huisnr`, `AannemerCode`

**Gedrag:**
- Bestaande codes worden overgeslagen (idempotent)
- Geocoding loopt asynchroon na de import (blokkeert niet)
- Fouten worden per rij gerapporteerd, import gaat door

Route: `POST /opdrachten/importeren` — rol: Planner, Administrator

---

## Geocoding

Bij import (en handmatige aanmaak) wordt asynchroon een GPS-coördinaat opgezocht via `geocodeOpdracht()` in `geocode.js`. Het resultaat wordt opgeslagen in `Geotag` en gekoppeld via `geotag_id`. Wordt gebruikt op de kaartweergave (`/kaart`).

---

## Standaard opdracht-query (`opQ`)

Server-side helper die de standaard SELECT met joins aanmaakt:

```js
const opQ = where => `
  SELECT o.*, ot.naam as ot_naam, ot.code as ot_code,
         a.titel as aannemer_naam, a.code as aannemer_code,
         pw.jaar as pw_jaar, pw.week as pw_week,
         g.lat as geo_lat, g.lon as geo_lon, g.niveau as geo_niveau
  FROM Opdrachten o
  JOIN OpdrachtTypes ot ON ot.id = o.opdrachttype_id
  JOIN Aannemers a      ON a.id  = o.aannemer_id
  LEFT JOIN Planningsweken pw ON pw.id = o.planningsweek_id
  LEFT JOIN Geotag g ON g.id = o.geotag_id
  ${where}`;
```

---

## Views

| Route | View | Toegang |
|---|---|---|
| `GET /opdrachten` | `opdrachten.ejs` | Alle ingelogde gebruikers (aannemer ziet enkel eigen) |
| `GET /opdrachten/nieuw` | `opdracht_nieuw.ejs` | Administrator, Planner |
| `GET /opdrachten/importeren` | `opdracht_importeren.ejs` | Planner, Administrator |
| `GET /opdrachten/:id` | `opdracht.ejs` | Alle (aannemer: enkel eigen) |

## Zie ook

- [[Secties]]
- [[Servicecatalogus]]
- [[Rollen en accounts]]
