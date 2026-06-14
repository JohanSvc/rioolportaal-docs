# Secties

#rioolportaal #domein

Terug naar [[Overzicht]]

Elke opdracht heeft meerdere **secties** — afgebakende deelstukken werk die afzonderlijk worden ingevuld, ingediend en gevalideerd. Secties zijn versioned: bij een NOK-oordeel wordt een nieuwe versie aangemaakt.

---

## Sectietypes

| Code | Label | Inhoud |
|---|---|---|
| `wegenis` | Wegenis | Verhardingstypes en oppervlakten (m²) |
| `uitvoering` | Uitvoeringsverslag | Verslag van de uitgevoerde werken |
| `asbuilt` | As-Built | Maatvoering (5 maten) |
| `service` | Meetstaat | Service-items met aantallen en prijzen |
| `voorbereiding` | Voorbereiding | Voorbereiding-bijlagen (foto's, PDF's) |

---

## DB-structuur

```
Secties (hoofd)
  ├── Sectie_Wegenis          (1-op-1 met Secties)
  │     └── Sectie_Wegenis_Items (n rijen per Sectie_Wegenis)
  ├── Sectie_Uitvoeringsverslag (1-op-1, veld: verslag)
  ├── Sectie_AsBuilt          (1-op-1, velden: maat_1..5)
  ├── Sectie_ServiceItems     (1-op-1)
  │     └── Sectie_ServiceItems_Items (n rijen)
  └── Sectie_Bijlagen         (n bijlagen per sectie)
        └── Geotag            (optioneel, voor foto-GPS)
```

`Secties.versie` begint op 1 en verhoogt bij elke herindiening na NOK.

---

## Sectiestatus-lifecycle

```
Draft  →  Compleet  →  Ter validatie  →  OK       →  Afgehandeld
                                      ↘  NOK      →  (aannemer past aan)
                                                  →  nieuwe versie (versie + 1)
```

Statusovergangen worden gelogd in `SectieStatusLog` (sectie_id, status_code, user_id, tijdstip).

---

## Helpers in server.js

| Functie | Beschrijving |
|---|---|
| `getWegenisSectie(opId)` | Laatste versie wegenis-sectie (TOP 1 ORDER BY versie DESC) |
| `getUitvoeringsSectie(opId)` | Laatste versie uitvoering-sectie |
| `getAsBuiltSectie(opId)` | Laatste versie as-built-sectie |
| `getServiceSectie(opId)` | Laatste versie meetstaat-sectie |
| `createSectie(opId, type, userId)` | Maak nieuwe sectie + subtabel-rij aan |
| `getOrCreateSectie(opId, type, userId)` | Haal bestaande Draft op of maak nieuwe aan |

**Patroon:** altijd `getOrCreateSectie()` aanroepen voor acties — nooit zelf de Draft-check doen in routes.

---

## Bijlagen

Bijlagen worden opgeslagen als base64 in `Sectie_Bijlagen.data` (foto's als data-URL, PDF's als base64-PDF). Ze zijn gekoppeld aan een sectie en optioneel aan een `BijlageCategorie`.

Foto's kunnen een GPS-coördinaat meekrijgen via de webcam-flow (`geo_lat`, `geo_lon`, `geo_acc`). De afstand tot de opdracht-geotag wordt berekend bij het ophalen.

---

## Validatie-flow

1. Aannemer vult sectie in → status `Draft`
2. Aannemer dient in → status `Compleet`
3. Werfleider valideert → status `OK` of `NOK`
4. Bij `NOK`: aannemer ontvangt opmerking, kan aanpassen en opnieuw indienen → nieuwe versie
5. Bij `OK`: status → `Afgehandeld` (na alle secties → opdracht naar `Te exporteren`)

---

## Sectie-export check

`checkAfgehandeld(opId)` verifieert of `wegenis`, `asbuilt` en `service` alle `Afgehandeld` zijn — bepaalt of een opdracht naar `Te exporteren` mag.

## Zie ook

- [[Opdrachten]]
- [[Servicecatalogus]]
