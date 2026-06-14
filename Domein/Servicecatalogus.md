# Servicecatalogus

#rioolportaal #domein

Terug naar [[Overzicht]]

De servicecatalogus legt de prijsafspraken vast per raamcontract. Aannemers kunnen enkel service-items aanrekenen die in hun raamcontract zijn opgenomen.

---

## DB-structuur

```
Servicecatalogi (versioned catalogus)
  └── ServiceNodes (hiërarchische boom: 1 → 1.1 → 1.1.1 → ...)
        └── ServiceItems (bladknopen: code, titel, eenheidsprijs, eenheid)
              └── Raamcontracten.id (item is geldig in dit contract)

Raamcontracten (aannemer + geldig_van/tot)
  └── ServiceItems (via raamcontract_id)

DefaultSets (per opdrachttype + catalogus)
  └── DefaultSet_Items (sn_id → aanbevolen ServiceNodes bij aanmaak opdracht)
```

---

## Servicecatalogus-boom

`ServiceNodes` vormt een boom via `parent_id` (zelfverwijzend). Codering is hiërarchisch: `1`, `1.1`, `1.1.1`.

Bladknopen worden gekoppeld aan service-items (`ServiceItems.sn_id`). Elke service-item heeft een eenheidsprijs en is geldig binnen één raamcontract.

---

## Raamcontract

Een raamcontract koppelt een aannemer aan een set prijsafspraken voor een geldigheidsperiode (`geldig_van`, `geldig_tot`). Bij het openen van een opdracht-detail wordt het meest recente geldige raamcontract van de aannemer opgezocht:

```sql
SELECT TOP 1 id FROM Raamcontracten
WHERE aannemer_id=? AND geldig_tot >= CAST(GETDATE() AS DATE)
ORDER BY geldig_tot DESC
```

---

## DefaultSets

Per opdrachttype + catalogus kan een set van standaard `ServiceNodes` worden geconfigureerd (`DefaultSets` + `DefaultSet_Items`). Bij het openen van een opdracht worden deze als startpunt aangeboden in de meetstaat (als suggestie, niet automatisch ingevuld).

---

## Views

| Route | View | Toegang |
|---|---|---|
| `GET /raamcontracten` | `raamcontracten.ejs` | Administrator, AannemerAdministrator |
| `GET /admin/catalogi` | `catalogi.ejs` | Administrator |
| `GET /admin/catalogi/importeren` | `catalogus_importeren.ejs` | Administrator |
| `GET /admin/raamcontracten/opladen` | `rc_opladen.ejs` | Administrator |
| `GET /admin/opdrachttypes/:id` | `opdrachttypes_detail.ejs` | Administrator (incl. defaultsets) |

---

## Eenheden

Tabel `Eenheden` (code + titel) — bv. `m`, `m²`, `st`, `lm`. Gekoppeld aan zowel `ServiceItems` als `Wegenis_Items`.

## Zie ook

- [[Opdrachten]]
- [[Secties]]
- [[Rollen en accounts]]
