# API-contract

#rioolportaal #architectuur #api

Terug naar [[Overzicht]]

---

## Algemeen

**Base URL (mockup):** `http://localhost:3003`  
**Auth:** JWT via `Authorization: Bearer <token>` of cookie `jwt` (httpOnly)  
**Content-Type:** `application/json` voor API-clients; form-encoded voor browser

---

## Auth

### `GET /login`
Geen auth vereist. Retourneert het loginscherm (browser) of negeer (API-clients).

### `POST /login`
Geen auth vereist.

**Body (form of JSON):**
```json
{ "userId": 1 }
```

**Response (browser):** redirect naar `/dashboard`  
**Response (API — stuur `Accept: application/json`):**
```json
{
  "token": "<jwt>",
  "user": {
    "id": 1,
    "naam": "Admin Portaal",
    "email": "admin@rioolportaal.be",
    "rol": "Administrator",
    "rollen": ["Administrator"],
    "aannemer_id": null
  }
}
```

Token geldigheid: **8 uur**.

### `GET /logout`
Wist sessie en JWT-cookie. Redirect naar `/login`.

---

## Opdrachten

Alle routes: auth vereist.

| Methode | Pad | Rollen | Beschrijving |
|---|---|---|---|
| GET | `/opdrachten` | alle | Lijst met filteropties (`status`, `aannemer`, `type`, `postcode`, `gemeente`) |
| GET | `/opdrachten/nieuw` | Administrator, Planner | Formulier nieuwe opdracht |
| POST | `/opdrachten/nieuw` | Administrator, Planner | Opdracht aanmaken |
| GET | `/opdrachten/importeren` | Planner, Administrator | Importformulier |
| POST | `/opdrachten/importeren` | Planner, Administrator | Excel-import (multipart, veld `bestand`) |
| GET | `/opdrachten/:id` | alle | Opdrachtdetail met alle secties |
| GET | `/opdrachten/:id/meetstaat` | AannemerWerfleider | Meetstaat-overzicht |
| GET | `/opdrachten/:id/serviceitems` | AannemerMedewerker | Service-items invoerscherm |

**Statustransities (POST):**

| Pad | Rollen | Transitie |
|---|---|---|
| `/opdrachten/:id/starten` | AannemerMedewerker | `Toegewezen` → `OnderhandenGenomen` |
| `/opdrachten/:id/uitvoeren` | AannemerMedewerker | `OnderhandenGenomen` → `Uitgevoerd` |
| `/opdrachten/:id/ter-validatie` | AannemerWerfleider | alle secties OK → `Ter validatie` |
| `/opdrachten/:id/meetstaat/indienen` | AannemerWerfleider | `Uitgevoerd`/`Afgekeurd` → `Ter validatie` |

**Sectie-acties (POST):**

| Pad | Rollen | Body |
|---|---|---|
| `/opdrachten/:id/wegenis/bulk` | AannemerMedewerker, AannemerWerfleider | `aantal_<id>: number` per wegenisitem |
| `/opdrachten/:id/sectie-compleet` | AannemerMedewerker, AannemerWerfleider | `sectie`, `reply?`, `verslag?` |
| `/opdrachten/:id/sectie-ter-validatie` | AannemerWerfleider | `sectie`, `reply?` |
| `/opdrachten/:id/sectie-validatie` | Werfleider | `sectie`, `oordeel` (ok/nok/reset), `opmerking?` |
| `/opdrachten/:id/uitvoering-verslag` | AannemerMedewerker, AannemerWerfleider | `verslag` → JSON `{ ok: true }` |
| `/opdrachten/:id/bemating` | AannemerMedewerker, AannemerWerfleider | `maat_1`…`maat_5`, `_compleet?` |
| `/opdrachten/:id/serviceitems` | AannemerMedewerker | `serviceitem_id`, `aantal` of `item_id` + `actie` (delete) |
| `/opdrachten/:id/serviceitems/bulk` | AannemerMedewerker | `aantal_<id>: number` per item |
| `/opdrachten/:id/foto/:sectietype` | AannemerMedewerker, AannemerWerfleider | `data` (base64), `context?`, `geo_lat?`, `geo_lon?`, `geo_acc?` |
| `/opdrachten/:id/foto/:bijlageId/delete` | AannemerMedewerker, AannemerWerfleider | — |

---

## Planning

| Methode | Pad | Rollen | Beschrijving |
|---|---|---|---|
| GET | `/planning` | Planner, Administrator, AannemerWerfleider | Planningsoverzicht (`opdrachten`, `weken`, `aannemers`) |
| POST | `/planning/toewijzen` | Planner, Administrator | Body: `opdracht_id`, `aannemer_id`, `afspraakdatum?`, `afspraaktekst?` |
| POST | `/planning/status` | Planner, Administrator | Body: `opdracht_id`, `status` (Doorgegeven/HLGepland/Gepland/Toegewezen) |

---

## Validaties

| Methode | Pad | Rollen | Beschrijving |
|---|---|---|---|
| GET | `/validaties` | Werfleider, Administrator | Opdrachten met status `Ter validatie` of `Te exporteren` |
| POST | `/validaties/:id/oordeel` | Werfleider, Administrator | Body: `oordeel` (ok/nok), `opmerking?` |
| GET | `/ter-validatie` | AannemerWerfleider, AannemerAdministrator | Eigen opdrachten `Ter validatie` of `Afgekeurd` |

---

## Uitvoering

| Methode | Pad | Rollen | Beschrijving |
|---|---|---|---|
| GET | `/uitvoering` | AannemerMedewerker, AannemerWerfleider | Opdrachten `Toegewezen`, `OnderhandenGenomen`, `Uitgevoerd` (gefilterd op eigen aannemer) |

---

## Exports

| Methode | Pad | Rollen | Beschrijving |
|---|---|---|---|
| GET | `/exports` | Werfleider, Administrator | Opdrachten `Te exporteren` + meetstaat/wegenis-data |
| POST | `/exports/exporteren` | Werfleider, Administrator | Body: `opdracht_ids[]` → download Excel, zet status op `Afgehandeld` |

---

## Kaart

| Methode | Pad | Rollen | Beschrijving |
|---|---|---|---|
| GET | `/kaart` | alle | Kaartpagina |
| GET | `/kaart/data` | alle | JSON: `[{ id, code, titel, status, lat, lon, gemeente }]` |

---

## Rapportage

| Methode | Pad | Rollen | Beschrijving |
|---|---|---|---|
| GET | `/rapportage` | Administrator, Werfleider, AannemerAdministrator, AannemerWerfleider | Rapportage-pagina |
| GET | `/rapportage/data` | zelfde | JSON: `{ perAannemer, chart }` |

---

## Admin

Alle routes: `Administrator` tenzij anders vermeld.

| Methode | Pad | Extra rollen | Beschrijving |
|---|---|---|---|
| GET | `/admin` | — | Dashboard met statuschart |
| GET | `/admin/accounts` | AannemerAdministrator | Accounts (gefilterd op eigen aannemer) |
| POST | `/admin/accounts/nieuw` | AannemerAdministrator | Body: `naam`, `email`, `rol`, `aannemer_id?` |
| POST | `/admin/accounts/:id/toggle` | — | Account activeren/deactiveren |
| GET | `/admin/opdrachttypes` | — | Lijst opdrachttypes |
| POST | `/admin/opdrachttypes/nieuw` | — | Body: `code`, `naam`, `omschrijving?` |
| GET | `/admin/sectietypes` | — | Lijst sectietypes |
| GET | `/admin/catalogi` | — | Servicecatalogi overzicht |
| GET | `/admin/catalogi/:id` | — | Catalogus detail (nodes, items, defaultsets) |
| POST | `/admin/catalogi/nieuw` | — | Body: `naam`, `code`, `versie?`, `geldig_van?`, `geldig_tot?` |
| GET | `/admin/settings` | — | Systeeminstellingen |
| POST | `/admin/settings` | — | Body: `<sleutel>: <waarde>` per instelling |

---

## Raamcontracten

| Methode | Pad | Rollen | Beschrijving |
|---|---|---|---|
| GET | `/raamcontracten` | Administrator, AannemerAdministrator | Lijst contracten (AannemerAdministrator ziet enkel eigen) |
| GET | `/raamcontracten/:id` | Administrator, AannemerAdministrator | Contract detail met service-items |
| POST | `/raamcontracten/nieuw` | Administrator | Body: `code`, `titel?`, `aannemer_id`, `geldig_van?`, `geldig_tot?` |

---

## Foutcodes

| Code | Betekenis |
|---|---|
| 401 | Niet ingelogd of ongeldige/verlopen JWT |
| 403 | Ingelogd maar onvoldoende rol |
| 500 | Serverfout (zie server-log) |

---

## Zie ook

- [[Presentielaag]] — architectuurbeslissingen (JWT, React Native, Electron)
- [[Stack en DB-laag]] — serverstructuur en route-modules
- [[Rollen en accounts]] — rollenmodel
